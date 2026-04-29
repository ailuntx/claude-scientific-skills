# Dask 数组

## 概述

Dask 数组通过分块算法实现了 NumPy 的 ndarray 接口。它将多个 NumPy 数组协调排列成网格结构，利用多核并行能力，支持处理超出内存限制的大型数据集。

## 核心概念

Dask 数组被划分为多个分块（block）：
- 每个分块是常规的 NumPy 数组
- 操作并行应用于每个分块
- 结果自动合并
- 支持核外计算（处理超出内存的数据）

## 核心功能

### Dask 数组支持的操作

**数学运算**：
- 算术运算（+, -, *, /）
- 标量函数（指数、对数、三角函数）
- 逐元素操作

**归约运算**：
- `sum()`, `mean()`, `std()`, `var()`
- 沿指定轴的归约
- `min()`, `max()`, `argmin()`, `argmax()`

**线性代数**：
- 张量收缩
- 点积与矩阵乘法
- 部分分解（SVD, QR）

**数据操作**：
- 转置
- 切片（标准索引与花式索引）
- 重塑形状
- 拼接与堆叠

**数组协议**：
- 通用函数（ufuncs）
- NumPy 互操作协议

## 使用场景

**适用 Dask 数组的情况**：
- 数组超出可用内存容量
- 计算可跨分块并行化
- 需执行 NumPy 风格的数值运算
- 需要将 NumPy 代码扩展到更大数据集

**建议继续使用 NumPy 的情况**：
- 数组完全容纳在内存中
- 操作需要全局数据视图
- 使用 Dask 未实现的特殊函数
- NumPy 单独使用已满足性能需求

## 重要限制

Dask 数组有意未实现部分 NumPy 功能：

**未实现的功能**：
- 多数 `np.linalg` 函数（仅支持基础操作）
- 难以并行化的操作（如全局排序）
- 内存低效操作（转换为列表、循环遍历）
- 许多特殊函数（根据社区需求逐步添加）

**替代方案**：对于不支持的操作，可考虑使用 `map_blocks` 配合自定义 NumPy 代码实现。

## 创建 Dask 数组

### 从 NumPy 数组转换
```python
import dask.array as da
import numpy as np

# 指定分块大小从 NumPy 数组创建
x = np.arange(10000)
dx = da.from_array(x, chunks=1000)  # 创建10个分块，每块1000个元素
```

### 随机数组
```python
# 创建指定分块的随机数组
x = da.random.random((10000, 10000), chunks=(1000, 1000))

# 其他随机函数
x = da.random.normal(10, 0.1, size=(10000, 10000), chunks=(1000, 1000))
```

### 全零/全一/空数组
```python
# 创建常量填充数组
zeros = da.zeros((10000, 10000), chunks=(1000, 1000))
ones = da.ones((10000, 10000), chunks=(1000, 1000))
empty = da.empty((10000, 10000), chunks=(1000, 1000))
```

### 通过函数创建
```python
# 通过函数创建数组
def create_block(block_id):
    return np.random.random((1000, 1000)) * block_id[0]

x = da.from_delayed(
    [[dask.delayed(create_block)((i, j)) for j in range(10)] for i in range(10)],
    shape=(10000, 10000),
    dtype=float
)
```

### 从磁盘加载
```python
# 从 HDF5 加载
import h5py
f = h5py.File('myfile.hdf5', mode='r')
x = da.from_array(f['/data'], chunks=(1000, 1000))

# 从 Zarr 加载
import zarr
z = zarr.open('myfile.zarr', mode='r')
x = da.from_array(z, chunks=(1000, 1000))
```

## 常用操作

### 算术运算
```python
import dask.array as da

x = da.random.random((10000, 10000), chunks=(1000, 1000))
y = da.random.random((10000, 10000), chunks=(1000, 1000))

# 逐元素运算（惰性执行）
z = x + y
z = x * y
z = da.exp(x)
z = da.log(y)

# 计算结果
result = z.compute()
```

### 归约运算
```python
# 沿轴归约
total = x.sum().compute()
mean = x.mean().compute()
std = x.std().compute()

# 沿特定轴归约
row_means = x.mean(axis=1).compute()
col_sums = x.sum(axis=0).compute()
```

### 切片与索引
```python
# 标准切片（返回 Dask 数组）
subset = x[1000:5000, 2000:8000]

# 花式索引
indices = [0, 5, 10, 15]
selected = x[indices, :]

# 布尔索引
mask = x > 0.5
filtered = x[mask]
```

### 矩阵运算
```python
# 矩阵乘法
A = da.random.random((10000, 5000), chunks=(1000, 1000))
B = da.random.random((5000, 8000), chunks=(1000, 1000))
C = da.matmul(A, B)
result = C.compute()

# 点积
dot_product = da.dot(A, B)

# 转置
AT = A.T
```

### 线性代数
```python
# SVD（奇异值分解）
U, s, Vt = da.linalg.svd(A)
U_computed, s_computed, Vt_computed = dask.compute(U, s, Vt)

# QR 分解
Q, R = da.linalg.qr(A)
Q_computed, R_computed = dask.compute(Q, R)

# 注意：仅部分线性代数操作可用
```

### 重塑与操作
```python
# 重塑形状
x = da.random.random((10000, 10000), chunks=(1000, 1000))
reshaped = x.reshape(5000, 20000)

# 转置
transposed = x.T

# 拼接
x1 = da.random.random((5000, 10000), chunks=(1000, 1000))
x2 = da.random.random((5000, 10000), chunks=(1000, 1000))
combined = da.concatenate([x1, x2], axis=0)

# 堆叠
stacked = da.stack([x1, x2], axis=0)
```

## 分块策略

分块设计对 Dask 数组性能至关重要。

### 分块大小指南

**理想分块大小**：
- 每块约 10-100 MB（压缩后）
- 数值数据每块约 100 万元素
- 平衡并行性与调度开销

**示例计算**：
```python
# 对于 float64 数据（每元素8字节）
# 目标100MB分块：100 MB / 8 字节 = 1250万元素

# 二维数组 (10000, 10000)：
x = da.random.random((10000, 10000), chunks=(1000, 1000))  # 每块约8MB
```

### 查看分块结构
```python
# 检查分块
print(x.chunks)  # ((1000, 1000, ...), (1000, 1000, ...))

# 分块数量
print(x.npartitions)

# 分块字节大小
print(x.nbytes / x.npartitions)
```

### 重分块
```python
# 更改分块大小
x = da.random.random((10000, 10000), chunks=(500, 500))
x_rechunked = x.rechunk((2000, 2000))

# 重分块特定维度
x_rechunked = x.rechunk({0: 2000, 1: 'auto'})
```

## 使用 map_blocks 自定义操作

对于 Dask 未实现的操作，使用 `map_blocks`：

```python
import dask.array as da
import numpy as np

def custom_function(block):
    # 应用自定义 NumPy 操作
    return np.fft.fft2(block)

x = da.random.random((10000, 10000), chunks=(1000, 1000))
result = da.map_blocks(custom_function, x, dtype=x.dtype)

# 计算结果
output = result.compute()
```

### 输出形状变化的 map_blocks
```python
def reduction_function(block):
    # 为每个分块返回标量
    return np.array([block.mean()])

result = da.map_blocks(
    reduction_function,
    x,
    dtype='float64',
    drop_axis=[0, 1],  # 输出不保留输入轴
    new_axis=0,        # 输出新增维度
    chunks=(1,)        # 每块输出一个元素
)
```

## 惰性求值与计算

### 惰性操作
```python
# 所有操作均为惰性（即时构建，不实际计算）
x = da.random.random((10000, 10000), chunks=(1000, 1000))
y = x + 100
z = y.mean(axis=0)
result = z * 2

# 尚未执行计算，仅构建任务图
```

### 触发计算
```python
# 计算单个结果
final = result.compute()

# 高效计算多个结果
result1, result2 = dask.compute(operation1, operation2)
```

### 持久化到内存
```python
# 将中间结果保留在内存
x_cached = x.persist()

# 复用缓存结果
y1 = (x_cached + 10).compute()
y2 = (x_cached * 2).compute()
```

## 结果保存

### 转为 NumPy
```python
# 转换为 NumPy 数组（加载全部数据到内存）
numpy_array = dask_array.compute()
```

### 保存到磁盘
```python
# 保存到 HDF5
import h5py
with h5py.File('output.hdf5', mode='w') as f:
    dset = f.create_dataset('/data', shape=x.shape, dtype=x.dtype)
    da.store(x, dset)

# 保存到 Zarr
import zarr
z = zarr.open('output.zarr', mode='w', shape=x.shape, dtype=x.dtype, chunks=x.chunks)
da.store(x, z)
```

## 性能考量

### 高效操作
- 逐元素运算：极高效率
- 可并行化的归约运算：高效
- 沿分块边界的切片：高效
- 分块对齐的矩阵运算：高效

### 低效操作
- 跨多分块的切片：需数据迁移
- 需全局排序的操作：支持有限
- 极不规则访问模式：性能低下
- 分块未对齐的操作：需重分块

### 优化建议

**1. 选择合适分块大小**
```python
# 追求分块均衡
# 推荐：每块约100MB
x = da.random.random((100000, 10000), chunks=(10000, 10000))
```

**2. 对齐操作的分块**
```python
# 确保操作的分块对齐
x = da.random.random((10000, 10000), chunks=(1000, 1000))
y = da.random.random((10000, 10000), chunks=(1000, 1000))  # 对齐
z = x + y  # 高效执行
```

**3. 选用合适调度器**
```python
# 数组操作默认适配线程调度器
# 共享内存访问效率高
result = x.compute()  # 默认使用线程
```

**4. 最小化数据传输**
```python
# 推荐：在各分块计算后传输结果
means = x.mean(axis=1).compute()  # 传输数据量少

# 不推荐：先传输全部数据再计算
x_numpy = x.compute()
means = x_numpy.mean(axis=1)  # 传输数据量大
```

## 常用模式

### 图像处理
```python
import dask.array as da

# 加载大型图像堆栈
images = da.from_zarr('images.zarr')

# 应用滤波
def apply_gaussian(block):
    from scipy.ndimage import gaussian_filter
    return gaussian_filter(block, sigma=2)

filtered = da.map_blocks(apply_gaussian, images, dtype=images.dtype)

# 计算统计量
mean_intensity = filtered.mean().compute()
```

### 科学计算
```python
# 大规模数值模拟
x = da.random.random((100000, 100000), chunks=(10000, 10000))

# 迭代计算
for i in range(num_iterations):
    x = da.exp(-x) * da.sin(x)
    x = x.persist()  # 保留内存供下次迭代

# 最终结果
result = x.compute()
```

### 数据分析
```python
# 加载大型数据集
data = da.from_zarr('measurements.zarr')

# 计算统计量
mean = data.mean(axis=0)
std = data.std(axis=0)
normalized = (data - mean) / std

# 保存标准化数据
da.to_zarr(normalized, 'normalized.zarr')
```

## 与其他工具集成

### XArray
```python
import xarray as xr
import dask.array as da

# XArray 用带标签维度封装 Dask 数组
data = da.random.random((1000, 2000, 3000), chunks=(100, 200, 300))
dataset = xr.DataArray(
    data,
    dims=['time', 'y', 'x'],
    coords={'time': range(1000), 'y': range(2000), 'x': range(3000)}
)
```

### Scikit-learn（通过 Dask-ML）
```python
# 部分 scikit-learn 兼容操作
from dask_ml.preprocessing import StandardScaler

X = da.random.random((10000, 100), chunks=(1000, 100))
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
```

## 调试技巧

### 可视化任务图
```python
# 可视化计算图（适用于小型数组）
x = da.random.random((100, 100), chunks=(10, 10))
y = x + 1
y.visualize(filename='graph.png')
```

### 检查数组属性
```python
# 计算前检查属性
print(f"形状: {x.shape}")
print(f"数据类型: {x.dtype}")
print(f"分块结构: {x.chunks}")
print(f"任务数量: {len(x.__dask_graph__())}")
```

### 先在小数组测试
```python
# 在小型数组测试逻辑
small_x = da.random.random((100, 100), chunks=(50, 50))
result_small = computation(small_x).compute()

# 验证后扩展
large_x = da.random.random((100000, 100000), chunks=(10000, 10000))
result_large = computation(large_x).compute()
```
