# Zarr Python 快速参考手册

本参考手册提供了常用 Zarr 函数、参数和模式的简明概览，方便开发过程中快速查阅。

## 数组创建函数

### `zarr.zeros()` / `zarr.ones()` / `zarr.empty()`
```python
zarr.zeros(shape, chunks=None, dtype='f8', store=None, compressor='default',
           fill_value=0, order='C', filters=None)
```
创建填充零值、单位值或未初始化值的数组。

**关键参数：**
- `shape`：定义数组维度的元组（如 `(1000, 1000)`）
- `chunks`：定义分块维度的元组（如 `(100, 100)`），或 `None` 表示不分块
- `dtype`：NumPy 数据类型（如 `'f4'`、`'i8'`、`'bool'`）
- `store`：存储位置（字符串路径、存储对象或 `None` 表示内存存储）
- `compressor`：压缩编解码器或 `None` 表示不压缩

### `zarr.create_array()` / `zarr.create()`
```python
zarr.create_array(store, shape, chunks, dtype='f8', compressor='default',
                  fill_value=0, order='C', filters=None, overwrite=False)
```
通过显式控制所有参数创建新数组。

### `zarr.array()`
```python
zarr.array(data, chunks=None, dtype=None, compressor='default', store=None)
```
从现有数据（NumPy 数组、列表等）创建数组。

**示例：**
```python
import numpy as np
data = np.random.random((1000, 1000))
z = zarr.array(data, chunks=(100, 100), store='data.zarr')
```

### `zarr.open_array()` / `zarr.open()`
```python
zarr.open_array(store, mode='a', shape=None, chunks=None, dtype=None,
                compressor='default', fill_value=0)
```
打开现有数组或创建新数组。

**模式选项：**
- `'r'`：只读
- `'r+'`：读写，文件必须存在
- `'a'`：读写，不存在则创建（默认）
- `'w'`：创建新文件，存在则覆盖
- `'w-'`：创建新文件，存在则报错

## 存储类

### LocalStore（默认）
```python
from zarr.storage import LocalStore

store = LocalStore('path/to/data.zarr')
z = zarr.open_array(store=store, mode='w', shape=(1000, 1000), chunks=(100, 100))
```

### MemoryStore
```python
from zarr.storage import MemoryStore

store = MemoryStore()  # 仅内存存储
z = zarr.open_array(store=store, mode='w', shape=(1000, 1000), chunks=(100, 100))
```

### ZipStore
```python
from zarr.storage import ZipStore

# 写入
store = ZipStore('data.zip', mode='w')
z = zarr.open_array(store=store, mode='w', shape=(1000, 1000), chunks=(100, 100))
z[:] = data
store.close()  # 必须关闭

# 读取
store = ZipStore('data.zip', mode='r')
z = zarr.open_array(store=store)
data = z[:]
store.close()
```

### 云存储（S3/GCS）
```python
# S3
import s3fs
s3 = s3fs.S3FileSystem(anon=False)
store = s3fs.S3Map(root='bucket/path/data.zarr', s3=s3)

# GCS
import gcsfs
gcs = gcsfs.GCSFileSystem(project='my-project')
store = gcsfs.GCSMap(root='bucket/path/data.zarr', gcs=gcs)
```

## 压缩编解码器

### Blosc 编解码器（默认）
```python
from zarr.codecs.blosc import BloscCodec

codec = BloscCodec(
    cname='zstd',      # 压缩器：'blosclz', 'lz4', 'lz4hc', 'snappy', 'zlib', 'zstd'
    clevel=5,          # 压缩级别：0-9
    shuffle='shuffle'  # 洗牌过滤器：'noshuffle', 'shuffle', 'bitshuffle'
)

z = zarr.create_array(store='data.zarr', shape=(1000, 1000), chunks=(100, 100),
                      dtype='f4', codecs=[codec])
```

**Blosc 压缩器特性：**
- `'lz4'`：最快压缩，压缩率较低
- `'zstd'`：均衡型（默认），压缩率和速度俱佳
- `'zlib'`：兼容性好，性能中等
- `'lz4hc'`：压缩率优于 lz4，速度较慢
- `'snappy'`：速度快，压缩率中等
- `'blosclz'`：Blosc 默认压缩器

### 其他编解码器
```python
from zarr.codecs import GzipCodec, ZstdCodec, BytesCodec

# Gzip 压缩（最高压缩率，速度慢）
GzipCodec(level=6)  # 级别 0-9

# Zstandard 压缩
ZstdCodec(level=3)  # 级别 1-22

# 无压缩
BytesCodec()
```

## 数组索引与选择

### 基础索引（NumPy 风格）
```python
z = zarr.zeros((1000, 1000), chunks=(100, 100))

# 读取
row = z[0, :]           # 单行
col = z[:, 0]           # 单列
block = z[10:20, 50:60] # 切片
element = z[5, 10]      # 单元素

# 写入
z[0, :] = 42
z[10:20, 50:60] = np.random.random((10, 10))
```

### 高级索引
```python
# 坐标索引（点选择）
z.vindex[[0, 5, 10], [2, 8, 15]]  # 特定坐标点

# 正交索引（外积）
z.oindex[0:10, [5, 10, 15]]  # 0-9行，5/10/15列

# 分块索引
z.blocks[0, 0]  # 第一个分块
z.blocks[0:2, 0:2]  # 前四个分块
```

## 组与层级结构

### 创建组
```python
# 创建根组
root = zarr.group(store='data.zarr')

# 创建嵌套组
grp1 = root.create_group('group1')
grp2 = grp1.create_group('subgroup')

# 在组内创建数组
arr = grp1.create_array(name='data', shape=(1000, 1000),
                        chunks=(100, 100), dtype='f4')

# 通过路径访问
arr2 = root['group1/data']
```

### 组方法
```python
root = zarr.group('data.zarr')

# h5py 兼容方法
dataset = root.create_dataset('data', shape=(1000, 1000), chunks=(100, 100))
subgrp = root.require_group('subgroup')  # 不存在则创建

# 可视化结构
print(root.tree())

# 列出内容
print(list(root.keys()))
print(list(root.groups()))
print(list(root.arrays()))
```

## 数组属性与元数据

### 操作属性
```python
z = zarr.zeros((1000, 1000), chunks=(100, 100))

# 设置属性
z.attrs['units'] = '米'
z.attrs['description'] = '温度数据'
z.attrs['created'] = '2024-01-15'
z.attrs['version'] = 1.2
z.attrs['tags'] = ['气候', '温度']

# 读取属性
print(z.attrs['units'])
print(dict(z.attrs))  # 所有属性转为字典

# 更新/删除
z.attrs['version'] = 2.0
del z.attrs['tags']
```

**注意：** 属性必须可 JSON 序列化。

## 数组属性与方法

### 属性
```python
z = zarr.zeros((1000, 1000), chunks=(100, 100), dtype='f4')

z.shape          # (1000, 1000)
z.chunks         # (100, 100)
z.dtype          # dtype('float32')
z.size           # 1000000
z.nbytes         # 4000000 (未压缩字节数)
z.nbytes_stored  # 磁盘实际压缩大小
z.nchunks        # 100 (分块数量)
z.cdata_shape    # 分块网格形状：(10, 10)
```

### 方法
```python
# 信息查看
print(z.info)  # 数组详细信息
print(z.info_items())  # 信息转为元组列表

# 调整尺寸
z.resize(1500, 1500)  # 改变维度

# 追加数据
z.append(new_data, axis=0)  # 沿轴向添加数据

# 复制
z2 = z.copy(store='new_location.zarr')
```

## 分块策略指南

### 分块大小计算
```python
# 对于 float32（每元素4字节）：
# 1 MB = 262,144 元素
# 10 MB = 2,621,440 元素

# 1 MB 分块示例：
(512, 512)      # 二维：512 × 512 × 4 = 1,048,576 字节
(128, 128, 128) # 三维：128 × 128 × 128 × 4 = 8,388,608 字节 ≈ 8 MB
(64, 256, 256)  # 三维：64 × 256 × 256 × 4 = 16,777,216 字节 ≈ 16 MB
```

### 按访问模式的分块策略

**时间序列（沿第一维顺序访问）：**
```python
chunks=(1, 720, 1440)  # 每个分块一个时间步
```

**行访问：**
```python
chunks=(10, 10000)  # 小行尺寸，跨列
```

**列访问：**
```python
chunks=(10000, 10)  # 跨行，小列尺寸
```

**随机访问：**
```python
chunks=(500, 500)  # 均衡的方形分块
```

**三维体数据：**
```python
chunks=(64, 64, 64)  # 立方体分块，各向同性访问
```

## 集成 API

### NumPy 集成
```python
import numpy as np

z = zarr.zeros((1000, 1000), chunks=(100, 100))

# 使用 NumPy 函数
result = np.sum(z, axis=0)
mean = np.mean(z)
std = np.std(z)

# 转为 NumPy
arr = z[:]  # 将整个数组加载到内存
```

### Dask 集成
```python
import dask.array as da

# 将 Zarr 加载为 Dask 数组
dask_array = da.from_zarr('data.zarr')

# 并行计算操作
result = dask_array.mean(axis=0).compute()

# 将 Dask 数组写入 Zarr
large_array = da.random.random((100000, 100000), chunks=(1000, 1000))
da.to_zarr(large_array, 'output.zarr')
```

### Xarray 集成
```python
import xarray as xr

# 将 Zarr 作为 Xarray 数据集打开
ds = xr.open_zarr('data.zarr')

# 将 Xarray 写入 Zarr
ds.to_zarr('output.zarr')

# 带坐标创建
ds = xr.Dataset(
    {'temperature': (['time', 'lat', 'lon'], data)},
    coords={
        'time': pd.date_range('2024-01-01', periods=365),
        'lat': np.arange(-90, 91, 1),
        'lon': np.arange(-180, 180, 1)
    }
)
ds.to_zarr('climate.zarr')
```

## 并行计算

### 同步器
```python
from zarr import ThreadSynchronizer, ProcessSynchronizer

# 多线程写入
sync = ThreadSynchronizer()
z = zarr.open_array('data.zarr', mode='r+', synchronizer=sync)

# 多进程写入
sync = ProcessSynchronizer('sync.sync')
z = zarr.open_array('data.zarr', mode='r+', synchronizer=sync)
```

**注意：** 同步仅在以下情况需要：
- 跨分块的并发写入
- 读取无需同步（始终安全）
- 各进程写入独立分块时无需同步

## 元数据合并

```python
# 合并元数据（创建所有数组/组后执行）
zarr.consolidate_metadata('data.zarr')

# 使用合并元数据打开（速度更快，尤其在云存储）
root = zarr.open_consolidated('data.zarr')
```

**优势：**
- 将 I/O 操作从 N 次减少到 1 次
- 对云存储至关重要（降低延迟）
- 加速层级遍历

**注意事项：**
- 数据更新后可能失效
- 修改后需重新合并
- 不适用于频繁更新的数据集

## 常用模式

### 动态增长的时间序列
```python
# 初始第一维为空
z = zarr.open('timeseries.zarr', mode='a',
              shape=(0, 720, 1440),
              chunks=(1, 720, 1440),
              dtype='f4')

# 追加新时间步
for new_timestep in data_stream:
    z.append(new_timestep, axis=0)
```

### 分块处理大型数组
```python
z = zarr.open('large_data.zarr', mode='r')

# 无需加载整个数组的处理
for i in range(0, z.shape[0], 1000):
    chunk = z[i:i+1000, :]
    result = process(chunk)
    save(result)
```

### 格式转换流程
```python
# HDF5 → Zarr
import h5py
with h5py.File('data.h5', 'r') as h5:
    z = zarr.array(h5['dataset'][:], chunks=(1000, 1000), store='data.zarr')

# Zarr → NumPy 文件
z = zarr.open('data.zarr', mode='r')
np.save('data.npy', z[:])

# Zarr → NetCDF（通过 Xarray）
ds = xr.open_zarr('data.zarr')
ds.to_netcdf('data.nc')
```

## 性能优化速查表

1. **分块大小**：每块 1-10 MB
2. **分块形状**：与访问模式对齐
3. **压缩策略**：
   - 快速：`BloscCodec(cname='lz4', clevel=1)`
   - 均衡：`BloscCodec(cname='zstd', clevel=5)`
   - 最高压缩：`GzipCodec(level=9)`
4. **云存储优化**：
   - 使用较大分块（5-100 MB）
   - 合并元数据
   - 考虑分片存储
5. **并行 I/O**：大型操作使用 Dask
6. **内存管理**：分块处理，避免加载整个数组

## 调试与性能分析

```python
z = zarr.open('data.zarr', mode='r')

# 详细信息
print(z.info)

# 大小统计
print(f"未压缩：{z.nbytes / 1e6:.2f} MB")
print(f"压缩后：{z.nbytes_stored / 1e6:.2f} MB")
print(f"压缩比：{z.nbytes / z.nbytes_stored:.1f}x")

# 分块信息
print(f"分块尺寸：{z.chunks}")
print(f"分块数量：{z.nchunks}")
print(f"分块网格：{z.cdata_shape}")
```

## 常用数据类型

```python
# 整型
'i1', 'i2', 'i4', 'i8'  # 有符号：8/16/32/64位
'u1', 'u2', 'u4', 'u8'  # 无符号：8/16/32/64位

# 浮点型
'f2', 'f4', 'f8'  # 16/32/64位（半精度/单精度/双精度）

# 其他类型
'bool'     # 布尔型
'c8', 'c16'  # 复数：64/128位
'S10'      # 定长字符串（10字节）
'U10'      # Unicode 字符串（10字符）
```

## 版本兼容性

Zarr-Python 3.x 版本同时支持：
- **Zarr

```python
# 以只读模式打开数组
try:
    z = zarr.open_array('data.zarr', mode='r')
except zarr.errors.PathNotFoundError:
    print("数组不存在")
except zarr.errors.ReadOnlyError:
    print("无法写入只读数组")
except Exception as e:
    print(f"意外错误：{e}")
```
