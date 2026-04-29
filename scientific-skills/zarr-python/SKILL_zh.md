---
name: zarr-python
description: 面向云存储的分块N维数组。支持压缩数组、并行I/O、S3/GCS集成，兼容NumPy/Dask/Xarray，适用于大规模科学计算流水线。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# Zarr Python

## 概述

Zarr是一个用于存储大型N维数组的Python库，支持分块和压缩。应用此技能可实现高效并行I/O、云原生工作流，以及与NumPy、Dask和Xarray的无缝集成。

## 快速入门

### 安装

```bash
uv pip install zarr
```

需要Python 3.11+。如需云存储支持，请安装额外包：
```python
uv pip install s3fs  # 支持S3
uv pip install gcsfs  # 支持Google云存储
```

### 基础数组创建

```python
import zarr
import numpy as np

# 创建带分块和压缩的二维数组
z = zarr.create_array(
    store="data/my_array.zarr",
    shape=(10000, 10000),
    chunks=(1000, 1000),
    dtype="f4"
)

# 使用NumPy风格索引写入数据
z[:, :] = np.random.random((10000, 10000))

# 读取数据
data = z[0:100, 0:100]  # 返回NumPy数组
```

## 核心操作

### 创建数组

Zarr提供多种便捷的数组创建函数：

```python
# 创建空数组
z = zarr.zeros(shape=(10000, 10000), chunks=(1000, 1000), dtype='f4',
               store='data.zarr')

# 创建填充数组
z = zarr.ones((5000, 5000), chunks=(500, 500))
z = zarr.full((1000, 1000), fill_value=42, chunks=(100, 100))

# 从现有数据创建
data = np.arange(10000).reshape(100, 100)
z = zarr.array(data, chunks=(10, 10), store='data.zarr')

# 克隆数组配置
z2 = zarr.zeros_like(z)  # 匹配z的形状/分块/数据类型
```

### 打开现有数组

```python
# 打开数组（默认读写模式）
z = zarr.open_array('data.zarr', mode='r+')

# 只读模式
z = zarr.open_array('data.zarr', mode='r')

# open()函数自动识别数组或组
z = zarr.open('data.zarr')  # 返回Array或Group
```

### 读写数据

Zarr数组支持类NumPy索引：

```python
# 写入整个数组
z[:] = 42

# 写入切片
z[0, :] = np.arange(100)
z[10:20, 50:60] = np.random.random((10, 10))

# 读取数据（返回NumPy数组）
data = z[0:100, 0:100]
row = z[5, :]

# 高级索引
z.vindex[[0, 5, 10], [2, 8, 15]]  # 坐标索引
z.oindex[0:10, [5, 10, 15]]       # 正交索引
z.blocks[0, 0]                     # 块/分块索引
```

### 调整大小与追加

```python
# 调整数组尺寸
z.resize(15000, 15000)  # 扩展或收缩维度

# 沿轴向追加数据
z.append(np.random.random((1000, 10000)), axis=0)  # 添加行
```

## 分块策略

分块对性能至关重要。根据访问模式选择分块尺寸和形状。

### 分块尺寸指南

- **最小分块尺寸**：推荐1MB以获得最佳性能
- **平衡原则**：大分块=元数据操作少；小分块=并行访问更优
- **内存考量**：压缩期间整个分块必须能放入内存

```python
# 配置分块尺寸（目标约1MB/分块）
# 对于float32数据：1MB = 262,144元素 = 512×512数组
z = zarr.zeros(
    shape=(10000, 10000),
    chunks=(512, 512),  # 约1MB分块
    dtype='f4'
)
```

### 分块与访问模式对齐

**关键**：分块形状根据数据访问方式显著影响性能。

```python
# 若频繁访问行（第一维度）
z = zarr.zeros((10000, 10000), chunks=(10, 10000))  # 分块跨列

# 若频繁访问列（第二维度）
z = zarr.zeros((10000, 10000), chunks=(10000, 10))  # 分块跨行

# 混合访问模式（平衡方案）
z = zarr.zeros((10000, 10000), chunks=(1000, 1000))  # 方形分块
```

**性能示例**：对于(200,200,200)数组，沿第一维度读取：
- 使用分块(1,200,200)：约107ms
- 使用分块(200,200,1)：约1.65ms（快65倍！）

### 大规模存储分片

当数组包含数百万小分块时，使用分片将分块分组为更大存储对象：

```python
from zarr.codecs import ShardingCodec, BytesCodec
from zarr.codecs.blosc import BloscCodec

# 创建带分片的数组
z = zarr.create_array(
    store='data.zarr',
    shape=(100000, 100000),
    chunks=(100, 100),  # 小分块便于访问
    shards=(1000, 1000),  # 每分片包含100个分块
    dtype='f4'
)
```

**优势**：
- 减少海量小文件造成的文件系统开销
- 提升云存储性能（减少对象请求）
- 避免文件系统块大小浪费

**重要**：写入前整个分片必须能放入内存。

## 压缩

Zarr对每个分块应用压缩以减少存储空间，同时保持快速访问。

### 配置压缩

```python
from zarr.codecs.blosc import BloscCodec
from zarr.codecs import GzipCodec, ZstdCodec

# 默认：Blosc + Zstandard
z = zarr.zeros((1000, 1000), chunks=(100, 100))  # 使用默认压缩

# 配置Blosc编解码器
z = zarr.create_array(
    store='data.zarr',
    shape=(1000, 1000),
    chunks=(100, 100),
    dtype='f4',
    codecs=[BloscCodec(cname='zstd', clevel=5, shuffle='shuffle')]
)

# 可用Blosc压缩器：'blosclz', 'lz4', 'lz4hc', 'snappy', 'zlib', 'zstd'

# 使用Gzip压缩
z = zarr.create_array(
    store='data.zarr',
    shape=(1000, 1000),
    chunks=(100, 100),
    dtype='f4',
    codecs=[GzipCodec(level=6)]
)

# 禁用压缩
z = zarr.create_array(
    store='data.zarr',
    shape=(1000, 1000),
    chunks=(100, 100),
    dtype='f4',
    codecs=[BytesCodec()]  # 无压缩
)
```

### 压缩性能技巧

- **Blosc**（默认）：压缩/解压快，适合交互式工作
- **Zstandard**：更高压缩率，略慢于LZ4
- **Gzip**：最高压缩率，性能较慢
- **LZ4**：最快压缩，压缩率较低
- **Shuffle**：对数值数据启用shuffle过滤器可提升压缩率

```python
# 数值科学数据最优方案
codecs=[BloscCodec(cname='zstd', clevel=5, shuffle='shuffle')]

# 速度最优方案
codecs=[BloscCodec(cname='lz4', clevel=1)]

# 压缩率最优方案
codecs=[GzipCodec(level=9)]
```

## 存储后端

Zarr通过灵活存储接口支持多种后端。

### 本地文件系统（默认）

```python
from zarr.storage import LocalStore

# 显式创建存储
store = LocalStore('data/my_array.zarr')
z = zarr.open_array(store=store, mode='w', shape=(1000, 1000), chunks=(100, 100))

# 或使用字符串路径（自动创建LocalStore）
z = zarr.open_array('data/my_array.zarr', mode='w', shape=(1000, 1000),
                    chunks=(100, 100))
```

### 内存存储

```python
from zarr.storage import MemoryStore

# 创建内存存储
store = MemoryStore()
z = zarr.open_array(store=store, mode='w', shape=(1000, 1000), chunks=(100, 100))

# 数据仅存于内存，不持久化
```

### ZIP文件存储

```python
from zarr.storage import ZipStore

# 写入ZIP文件
store = ZipStore('data.zip', mode='w')
z = zarr.open_array(store=store, mode='w', shape=(1000, 1000), chunks=(100, 100))
z[:] = np.random.random((1000, 1000))
store.close()  # 重要：必须关闭ZipStore

# 从ZIP文件读取
store = ZipStore('data.zip', mode='r')
z = zarr.open_array(store=store)
data = z[:]
store.close()
```

### 云存储（S3, GCS）

```python
import s3fs
import zarr

# S3存储
s3 = s3fs.S3FileSystem(anon=False)  # 使用凭证
store = s3fs.S3Map(root='my-bucket/path/to/array.zarr', s3=s3)
z = zarr.open_array(store=store, mode='w', shape=(1000, 1000), chunks=(100, 100))
z[:] = data

# Google云存储
import gcsfs
gcs = gcsfs.GCSFileSystem(project='my-project')
store = gcsfs.GCSMap(root='my-bucket/path/to/array.zarr', gcs=gcs)
z = zarr.open_array(store=store, mode='w', shape=(1000, 1000), chunks=(100, 100))
```

**云存储最佳实践**：
- 使用`zarr.consolidate_metadata(store)`合并元数据以减少延迟
- 分块尺寸对齐云存储对象大小（通常5-100MB最优）
- 使用Dask启用大规模数据并行写入
- 考虑分片以减少对象数量

## 组与层级结构

组可分层组织多个数组，类似目录或HDF5组。

### 创建与使用组

```python
# 创建根组
root = zarr.group(store='data/hierarchy.zarr')

# 创建子组
temperature = root.create_group('temperature')
precipitation = root.create_group('precipitation')

# 在组内创建数组
temp_array = temperature.create_array(
    name='t2m',
    shape=(365, 720, 1440),
    chunks=(1, 720, 1440),
    dtype='f4'
)

precip_array = precipitation.create_array(
    name='prcp',
    shape=(365, 720, 1440),
    chunks=(1, 720, 1440),
    dtype='f4'
)

# 通过路径访问
array = root['temperature/t2m']

# 可视化层级
print(root.tree())
# 输出：
# /
#  ├── temperature
#  │   └── t2m (365, 720, 1440) f4
#  └── precipitation
#      └── prcp (365, 720, 1440) f4
```

### H5py兼容API

Zarr提供h5py兼容接口方便HDF5用户：

```python
# 使用h5py风格方法创建组
root = zarr.group('data.zarr')
dataset = root.create_dataset('my_data', shape=(1000, 1000), chunks=(100, 100),
                              dtype='f4')

# 类h5py访问
grp = root.require_group('subgroup')
arr = grp.require_dataset('array', shape=(500, 500), chunks=(50, 50), dtype='i4')
```

## 属性与元数据

通过属性附加自定义元数据到数组和组：

```python
# 添加数组属性
z = zarr.zeros((1000, 1000), chunks=(100, 100))
z.attrs['description'] = '开尔文温度数据'
z.attrs['units'] = 'K'
z.attrs['created'] = '2024-01-15'
z.attrs['processing_version'] = 2.1

# 属性以JSON格式存储
print(z.attrs['units'])  # 输出：K

# 添加组属性
root = zarr.group('data.zarr')
root.attrs['project'] = '气候分析'
root.attrs['institution'] = '研究院'

# 属性随数组/组持久化
z2 = zarr.open('data.zarr')
print(z2.attrs['description'])
```

**重要**：属性必须为JSON可序列化类型（字符串/数字/列表/字典/布尔值/null）。

## 与NumPy、Dask和Xarray集成

### NumPy集成

Zarr数组实现NumPy数组接口：

```python
import numpy as np
import zarr

z = zarr.zeros((1000, 1000), chunks=(100, 100))

# 直接使用NumPy函数
result = np.sum(z, axis=0)  # NumPy操作Zarr数组
mean = np.mean(z[:100, :100])

# 转为NumPy数组
numpy_array = z[:]  # 将整个数组加载到内存
```

### Dask集成

Dask提供对Zarr数组的惰性并行计算：

```python
import dask.array as da
import zarr

# 创建大型Zarr数组
z = zarr.open('data.zarr', mode='w', shape=(100000, 100000),
              chunks=(1000, 1000), dtype='f4')

# 加载为Dask数组（惰性，不加载数据）
dask_array = da.from_zarr('data.zarr')

# 执行计算（并行，核外）
result = dask_array.mean(axis=0).compute()  # 并行计算

# 将Dask数组写入Zarr
large_array = da.random.random((100000, 100000), chunks=(1000, 1000))
da.to_zarr(large_array, 'output.zarr')
```

**优势**：
- 处理大于内存的数据集
- 跨分块自动并行计算
- 分块存储实现高效I/O

### Xarray集成

Xarray提供带Zarr后端的带标签多维数组：

```python
import xarray as xr
import zarr

# 以Xarray Dataset打开Zarr存储（惰性加载）
ds = xr.open_zarr('data.zarr')

# 数据集包含坐标和元数据
print(ds)

# 访问变量
temperature = ds['temperature']

# 执行带标签操作
subset = ds.sel(time='2024-01', lat=slice(30, 60))

# 将Xarray Dataset写入Zarr
ds.to_zarr('output.zarr')

# 带坐标创建数据集
ds = xr.Dataset(
    {
        'temperature': (['time', 'lat', 'lon'], data),
        'precipitation': (['time', 'lat', 'lon'], data2)
    },
    coords={
        'time': pd.date_range('2024-01-01', periods=365),
        'lat': np.arange(-90, 91, 1),
        'lon': np.arange(-180, 180, 1)
    }
)
ds.to_zarr('climate_data.zarr')
```

**优势**：
- 命名维度和坐标
- 基于标签的索引和选择
- 与pandas集成处理时间序列
- 气候/地理空间科学家熟悉的NetCDF式接口

## 并行计算与同步

### 线程安全操作

```python
from zarr import ThreadSynchronizer
import zarr

# 多线程写入
synchronizer = ThreadSynchronizer()
z = zarr.open_array('data.zarr', mode='r+', shape=(10000, 10000),
                    chunks=(1000

z = zarr.open_array('data.zarr', mode='r+', shape=(10000, 10000),
                    chunks=(1000, 1000), synchronizer=synchronizer)

# 支持多进程并发写入
```

**注意**：
- 并发读取无需同步
- 仅当写入操作可能跨越分块边界时才需同步
- 每个进程/线程写入独立分块时无需同步

## 元数据整合

对于包含大量数组的分层存储，将元数据整合到单个文件以减少 I/O 操作：

```python
import zarr

# 创建数组/组后
root = zarr.group('data.zarr')
# ... 创建多个数组/组 ...

# 整合元数据
zarr.consolidate_metadata('data.zarr')

# 使用整合元数据打开（速度更快，尤其在云存储上）
root = zarr.open_consolidated('data.zarr')
```

**优势**：
- 将元数据读取操作从 N（每个数组一次）减少到 1
- 对云存储至关重要（降低延迟）
- 加速 `tree()` 操作和组遍历

**注意事项**：
- 若数组更新后未重新整合，元数据可能过时
- 不适用于频繁更新的数据集
- 多写入器场景可能导致读取不一致

## 性能优化

### 最佳性能检查清单

1. **分块大小**：目标为每块 1-10 MB
   ```python
   # float32 类型：1MB = 262,144 个元素
   chunks = (512, 512)  # 512×512×4 字节 ≈ 1MB
   ```

2. **分块形状**：与访问模式对齐
   ```python
   # 行访问 → 分块跨列：(小, 大)
   # 列访问 → 分块跨行：(大, 小)
   # 随机访问 → 均衡：(中, 中)
   ```

3. **压缩**：根据工作负载选择
   ```python
   # 交互/快速：BloscCodec(cname='lz4')
   # 均衡：BloscCodec(cname='zstd', clevel=5)
   # 最大压缩：GzipCodec(level=9)
   ```

4. **存储后端**：与环境匹配
   ```python
   # 本地：LocalStore（默认）
   # 云端：S3Map/GCSMap（需整合元数据）
   # 临时：MemoryStore
   ```

5. **分片**：用于大规模数据集
   ```python
   # 当存在数百万个小分块时
   shards=(10*chunk_size, 10*chunk_size)
   ```

6. **并行 I/O**：大型操作使用 Dask
   ```python
   import dask.array as da
   dask_array = da.from_zarr('data.zarr')
   result = dask_array.compute(scheduler='threads', num_workers=8)
   ```

### 性能分析与调试

```python
# 打印详细数组信息
print(z.info)

# 输出包含：
# - 类型、形状、分块、数据类型
# - 压缩编解码器及级别
# - 存储大小（压缩 vs 未压缩）
# - 存储位置

# 检查存储大小
print(f"压缩大小: {z.nbytes_stored / 1e6:.2f} MB")
print(f"未压缩大小: {z.nbytes / 1e6:.2f} MB")
print(f"压缩率: {z.nbytes / z.nbytes_stored:.2f}x")
```

## 常用模式与最佳实践

### 模式：时间序列数据

```python
# 以时间作为第一维度存储时间序列
# 便于高效追加新时间步
z = zarr.open('timeseries.zarr', mode='a',
              shape=(0, 720, 1440),  # 初始 0 个时间步
              chunks=(1, 720, 1440),  # 每个分块一个时间步
              dtype='f4')

# 追加新时间步
new_data = np.random.random((1, 720, 1440))
z.append(new_data, axis=0)
```

### 模式：大型矩阵运算

```python
import dask.array as da

# 在 Zarr 中创建大型矩阵
z = zarr.open('matrix.zarr', mode='w',
              shape=(100000, 100000),
              chunks=(1000, 1000),
              dtype='f8')

# 使用 Dask 并行计算
dask_z = da.from_zarr('matrix.zarr')
result = (dask_z @ dask_z.T).compute()  # 并行矩阵乘法
```

### 模式：云原生工作流

```python
import s3fs
import zarr

# 写入 S3
s3 = s3fs.S3FileSystem()
store = s3fs.S3Map(root='s3://my-bucket/data.zarr', s3=s3)

# 创建适合云存储的分块数组
z = zarr.open_array(store=store, mode='w',
                    shape=(10000, 10000),
                    chunks=(500, 500),  # ≈1MB 分块
                    dtype='f4')
z[:] = data

# 整合元数据加速读取
zarr.consolidate_metadata(store)

# 从 S3 读取（随时随地）
store_read = s3fs.S3Map(root='s3://my-bucket/data.zarr', s3=s3)
z_read = zarr.open_consolidated(store_read)
subset = z_read[0:100, 0:100]
```

### 模式：格式转换

```python
# HDF5 转 Zarr
import h5py
import zarr

with h5py.File('data.h5', 'r') as h5:
    dataset = h5['dataset_name']
    z = zarr.array(dataset[:],
                   chunks=(1000, 1000),
                   store='data.zarr')

# NumPy 转 Zarr
import numpy as np
data = np.load('data.npy')
z = zarr.array(data, chunks='auto', store='data.zarr')

# Zarr 转 NetCDF（通过 Xarray）
import xarray as xr
ds = xr.open_zarr('data.zarr')
ds.to_netcdf('data.nc')
```

## 常见问题与解决方案

### 问题：性能缓慢

**诊断**：检查分块大小与对齐
```python
print(z.chunks)  # 分块大小是否合适？
print(z.info)    # 检查压缩率
```

**解决方案**：
- 将分块增至 1-10 MB
- 使分块与访问模式对齐
- 尝试不同压缩编解码器
- 使用 Dask 并行操作

### 问题：内存占用过高

**原因**：加载整个数组或大分块至内存

**解决方案**：
```python
# 避免加载整个数组
# 错误：data = z[:]
# 正确：分块处理
for i in range(0, z.shape[0], 1000):
    chunk = z[i:i+1000, :]
    process(chunk)

# 或使用 Dask 自动分块
import dask.array as da
dask_z = da.from_zarr('data.zarr')
result = dask_z.mean().compute()  # 分块处理
```

### 问题：云存储延迟

**解决方案**：
```python
# 1. 整合元数据
zarr.consolidate_metadata(store)
z = zarr.open_consolidated(store)

# 2. 使用合适分块大小（云存储建议 5-100 MB）
chunks = (2000, 2000)  # 云存储使用更大分块

# 3. 启用分片
shards = (10000, 10000)  # 聚合多个分块
```

### 问题：并发写入冲突

**解决方案**：使用同步器或确保非重叠写入
```python
from zarr import ProcessSynchronizer

sync = ProcessSynchronizer('sync.sync')
z = zarr.open_array('data.zarr', mode='r+', synchronizer=sync)

# 或设计工作流使各进程写入独立分块
```

## 扩展资源

详细 API 文档、高级用法及最新更新：

- **官方文档**：https://zarr.readthedocs.io/
- **Zarr 规范**：https://zarr-specs.readthedocs.io/
- **GitHub 仓库**：https://github.com/zarr-developers/zarr-python
- **社区聊天**：https://gitter.im/zarr-developers/community

**相关库**：
- **Xarray**：https://docs.xarray.dev/（带标签数组）
- **Dask**：https://docs.dask.org/（并行计算）
- **NumCodecs**：https://numcodecs.readthedocs.io/（压缩编解码器）
