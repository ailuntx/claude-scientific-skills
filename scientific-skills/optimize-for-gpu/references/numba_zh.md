# Numba CUDA 参考指南

Numba 将 Python 直接编译为 CUDA 内核，让您完全掌控 GPU 线程、块、共享内存和同步机制。当算法需要无法通过标准数组操作实现的自定义 GPU 逻辑时，请使用 Numba。

> **完整文档：** https://numba.readthedocs.io/en/stable/cuda/index.html

## 目录

1. [安装与配置](#installation-and-setup)
2. [核心概念：内核、线程、块、网格](#core-concepts)
3. [编写 CUDA 内核](#writing-cuda-kernels)
4. [线程定位](#thread-positioning)
5. [内存管理](#memory-management)
6. [共享内存](#shared-memory)
7. [设备函数](#device-functions)
8. [原子操作](#atomic-operations)
9. [GPU 通用函数：@vectorize 与 @guvectorize](#gpu-ufuncs)
10. [GPU 归约运算](#gpu-reductions)
11. [流与异步操作](#streams)
12. [随机数生成](#random-number-generation)
13. [协作组](#cooperative-groups)
14. [科学计算常用模式](#common-patterns)
15. [性能优化](#performance-optimization)
16. [调试](#debugging)
17. [互操作性](#interoperability)
18. [常见陷阱](#common-pitfalls)

---

## 安装与配置

在所有安装说明、文档字符串、注释和错误信息中，始终使用 `uv add`（切勿使用 `pip install` 或 `conda install`）。

```bash
uv add numba numba-cuda
```

`numba-cuda` 是官方维护的 NVIDIA 实现包。它在 `numba.cuda` 命名空间下实现功能——相比旧版内置目标无需修改代码。

**要求：** CUDA Toolkit ≥ 11.2，GPU 计算能力 ≥ 5.0（Maxwell 或更新架构）。

```python
from numba import cuda

# 验证 GPU 可用性
print(cuda.is_available())   # CUDA 可用时返回 True
cuda.detect()                # 打印 GPU 详细信息
```

---

## 核心概念

CUDA 采用层级结构组织并行执行：

```
网格（由块组成）→ 块（由线程组成）→ 线程
```

- **线程**：最小执行单元，每个线程运行内核函数
- **块**：可共享快速片上内存并相互同步的线程组，每块最多 1024 线程
- **网格**：内核启动时所有块的集合

**内核**是在 GPU 上运行的函数，由 CPU 启动。**设备函数**在 GPU 上运行，但由其他 GPU 代码调用（非 CPU 调用）。

---

## 编写 CUDA 内核

### @cuda.jit 装饰器

```python
from numba import cuda

@cuda.jit
def my_kernel(input_array, output_array):
    i = cuda.grid(1)                    # 获取当前线程全局索引
    if i < input_array.size:            # 边界检查——必须执行
        output_array[i] = input_array[i] * 2.0
```

**@cuda.jit 关键参数：**

| 参数 | 用途 |
|-----------|---------|
| `device=True` | 声明为设备函数（仅 GPU 可调用，可返回值） |
| `fastmath=True` | 启用快速数学（快速 sqrt/除法/FMA/三角/指数/对数浮点近似） |
| `max_registers=N` | 限制每线程寄存器数以提升占用率 |
| `cache=True` | 缓存编译后内核到磁盘 |
| `debug=True` | 启用异常检查（性能低，仅调试时配合 `opt=False` 使用） |
| `lineinfo=True` | 为性能分析提供源码行信息（无完整调试开销） |

### 启动内核

```python
import numpy as np
from numba import cuda

data = np.random.rand(1_000_000).astype(np.float32)
out = np.zeros_like(data)

# 传输到 GPU
d_data = cuda.to_device(data)
d_out = cuda.device_array_like(out)

# 计算启动配置
threads_per_block = 256
blocks_per_grid = (data.size + threads_per_block - 1) // threads_per_block

# 启动内核
my_kernel[blocks_per_grid, threads_per_block](d_data, d_out)

# 取回结果
result = d_out.copy_to_host()
```

**启动语法：** `kernel[网格维度, 块维度, 流, 动态共享内存字节数](...参数)`

第 3、4 参数可选（流对象和动态共享内存字节数）。

### 二维启动配置

```python
@cuda.jit
def kernel_2d(matrix, output):
    x, y = cuda.grid(2)
    if x < matrix.shape[0] and y < matrix.shape[1]:
        output[x, y] = matrix[x, y] * 2.0

threads = (16, 16)
blocks = (
    (matrix.shape[0] + threads[0] - 1) // threads[0],
    (matrix.shape[1] + threads[1] - 1) // threads[1],
)
kernel_2d[blocks, threads](d_matrix, d_output)
```

### 便捷方法：.forall() 一维处理

```python
# 自动计算一维网格维度
my_kernel.forall(len(data))(d_data, d_out)
```

### 内核关键规则

1. **内核不可返回值**，所有输出必须写入参数传递的数组
2. **必须检查数组边界**，网格尺寸 > 数组尺寸时越界线程会静默破坏内存
3. **内核启动是异步的**，CPU 读取结果前需调用 `cuda.synchronize()`

---

## 线程定位

### 内置函数

| 内置函数 | 描述 |
|-----------|-------------|
| `cuda.threadIdx.x/y/z` | 线程在块内的索引 |
| `cuda.blockIdx.x/y/z` | 块在网格中的索引 |
| `cuda.blockDim.x/y/z` | 每块线程数 |
| `cuda.gridDim.x/y/z` | 网格中的块数 |
| `cuda.grid(ndim)` | 网格中的绝对位置（1D→整数，2D/3D→元组） |
| `cuda.gridsize(ndim)` | 网格中线程总数 |

### 网格步进循环模式

处理大于网格的数据时，使用网格步进循环。解耦网格尺寸与问题规模，对复用 RNG 状态至关重要。

```python
@cuda.jit
def process_large(data, out):
    start = cuda.grid(1)
    stride = cuda.gridsize(1)
    for i in range(start, data.shape[0], stride):
        out[i] = data[i] * 2.0
```

---

## 内存管理

### 数据传输

```python
# 主机→设备
d_array = cuda.to_device(numpy_array)                    # 同步复制
d_array = cuda.to_device(numpy_array, stream=stream)     # 异步复制

# 设备端分配（无复制）
d_array = cuda.device_array(shape=(1000,), dtype=np.float32)
d_array = cuda.device_array_like(numpy_array)

# 设备→主机
host_array = d_array.copy_to_host()                      # 创建新数组
d_array.copy_to_host(existing_array)                     # 写入预分配数组
d_array.copy_to_host(stream=stream)                      # 异步复制
```

### 内存类型

| 类型 | API | 使用场景 |
|------|-----|----------|
| **设备内存** | `cuda.device_array()`, `cuda.to_device()` | 标准 GPU 内存 |
| **锁页内存** | `cuda.pinned_array()`, `cuda.pinned()` 上下文管理器 | 页锁定主机内存——传输更快 |
| **映射内存** | `cuda.mapped_array()` | 主机和设备均可访问 |
| **统一内存** | `cuda.managed_array()` | 自动在主机/设备间迁移（推荐 Linux/x86） |
| **常量内存** | `cuda.const.array_like(arr)` | 只读缓存，由主机设置 |

### 锁页内存加速传输

```python
# 分配锁页主机内存（页锁定——加速 PCI-e 传输）
with cuda.pinned(host_array):
    d_array = cuda.to_device(host_array, stream=stream)
    # 传输更快，因 OS 无法换出此内存

# 或直接分配
pinned = cuda.pinned_array(shape=(1000,), dtype=np.float32)
```

### 延迟释放控制

```python
with cuda.defer_cleanup():
    # 此区域内延迟所有 GPU 内存释放——避免隐式同步
    # 在性能关键区域使用
    run_many_kernels()
# 此处执行清理
```

---

## 共享内存

共享内存是块内共享的快速片上内存（带宽达 TB/s 级）。它是实现高性能内核的关键——用于缓存块内多线程将访问的数据。

### 静态共享内存（编译时已知尺寸）

```python
from numba import cuda, float32

@cuda.jit
def kernel_with_shared(data, output):
    # 分配共享内存——当前块所有线程可见
    shared = cuda.shared.array(256, dtype=float32)

    tid = cuda.threadIdx.x
    i = cuda.grid(1)

    # 每个线程加载一个元素到共享内存
    if i < data.size:
        shared[tid] = data[i]

    # 屏障：等待块内所有线程完成加载
    cuda.syncthreads()

    # 此时可安全读取 shared[] 中任意元素
    if i < data.size and tid > 0:
        output[i] = shared[tid] + shared[tid - 1]
```

### 动态共享内存（启动时设置尺寸）

```python
@cuda.jit
def kernel_dynamic_shared(data):
    # size=0 表示使用动态共享内存
    dyn = cuda.shared.array(0, dtype=float32)
    tid = cuda.threadIdx.x
    dyn[tid] = data[cuda.grid(1)]
    cuda.syncthreads()
    # ...

# 启动时指定尺寸（第4参数=字节数）
kernel_dynamic_shared[blocks, threads, stream, 1024](data)  # 1024字节共享内存
```

**重要提示：** 同一内核中所有 `cuda.shared.array(0, ...)` 调用共享相同内存区域。需手动划分切片以使用多个动态共享数组。

### 本地内存（线程私有暂存区）

```python
@cuda.jit
def kernel_with_local(data):
    # 每个线程拥有私有数组
    local_buf = cuda.local.array(10, dtype=float32)
    i = cuda.grid(1)
    for j in range(10):
        local_buf[j] = data[i * 10 + j]
    # 处理 local_buf...
```

---

## 设备函数

设备函数在 GPU 上运行，由内核或其他设备函数调用。与内核不同，它们**可返回值**。

```python
@cuda.jit(device=True)
def compute_distance(x1, y1, x2, y2):
    return math.sqrt((x2 - x1)**2 + (y2 - y1)**2)

@cuda.jit
def kernel(points, distances):
    i = cuda.grid(1)
    if i < points.shape[0] - 1:
        distances[i] = compute_distance(
            points[i, 0], points[i, 1],
            points[i+1, 0], points[i+1, 1]
        )
```

**交叉编译说明：** 被 `@numba.jit`（CPU JIT）装饰的函数也可从 CUDA 内核调用——适用于在 CPU/GPU 代码路径间共享逻辑。

---

## 原子操作

原子操作确保线程安全更新共享数据，所有操作返回**旧值**。

```python
cuda.atomic.add(array, index, value)       # +=   (int32, float32, float64)
cuda.atomic.sub(array, index, value)       # -=   (int32, float32, float64)
cuda.atomic.max(array, index, value)       # max  (int/uint 32/64, float 32/64)
cuda.atomic.min(array, index, value)       # min  (相同类型)
cuda.atomic.nanmax(array, index, value)    # 忽略 NaN 取最大值
cuda.atomic.nanmin(array, index, value)    # 忽略 NaN 取最小值
cuda.atomic.and_(array, index, value)      # &=   (int/uint 32/64)
cuda.atomic.or_(array, index, value)       # |=   (int/uint 32/64)
cuda.atomic.xor(array, index, value)       # ^=   (int/uint 32/64)
cuda.atomic.exch(array, index, value)      # 交换
cuda.atomic.cas(array, index, old, value)  # 比较并交换
```

多维索引通过元组实现：`cuda.atomic.add(result, (row, col), value)`

### 示例：直方图

```python
@cuda.jit
def histogram(data, bins):
    i = cuda.grid(1)
    if i < data.size:
        bin_idx = int(data[i] * len(bins))
        if 0 <= bin_idx < len(bins):
            cuda.atomic.add(bins, bin_idx, 1)
```

---

## GPU 通用函数

### @vectorize —— GPU 元素级操作

在 GPU 上运行元素级操作的最简方式。编写标量函数后，Numba 自动在数组上广播执行。

```python
from numba import vectorize, float32, float64
import math

@vectorize([float32(float32, float32),
            float64(float64, float64)],
           target='cuda')
def gpu_hypot(x, y):
    return math.sqrt(x**2 + y**2)

# 用法——像 NumPy 通用函数调用
result = gpu_hypot(array_x, array_y)

# 传入设备数组避免传输
d_x = cuda.to_device(x)
d_y = cuda.to_device(y)
d_result = gpu_hypot(d_x, d_y)
```

### @guvectorize —— 广义通用函数

用于子数组操作（非标量）。采用 NumPy 的广义通用函数签名。

```python
from numba import guvectorize, float32

@guvectorize([float32[:,:], float32[:,:], float32[:,:]],
             '(m,n),(n,p)->(m,p)', target='cuda')
def gpu_matmul(A, B, C):
    for i in range(A.shape[0]):
        for j in range(B.shape[1]):
            total = 0.0
            for k in range(A.shape[1]):
                total += A[i, k] * B[k, j]
            C[i, j] = total
```

---

## GPU 归约运算

```python
from numba import cuda

# 定义归约操作
sum_reduce = cuda.reduce(lambda a, b: a + b)

# 使用示例
result = sum_reduce(array)                    # 完整归约
result = sum_reduce(array, init=0)            # 带初始值
sum_reduce(array, res=device_result)          # 写入设备数组（无设备→主机复制）
sum_reduce(array, stream=stream)              # 异步执行
```

自定义归约：

```python
@cuda.reduce
def max_reduce(a, b):
    return a if a > b else b

maximum = max_reduce(data_array)
```

---

## 流操作

流支持重叠计算与数据传输，以及并发运行多个内核。

```python
stream = cuda.stream()

# 异步传输 → 内核 → 回传
d_data = cuda.to_device(host_data, stream=stream)
my_kernel[blocks, threads, stream](d_data, d_out)
result = d_out.copy_to_host(stream=stream)
stream.synchronize()  # 等待此流所有操作

# 自动同步的上下文管理器
with stream.auto_synchronize():
    d_data = cuda.to_device(host_data, stream=stream)
    my_kernel[blocks, threads, stream](d_data, d_out)
    result = d_out.copy_to_host(stream=stream)
# 此处自动同步
```

### 流水线模式（重叠传输与计算）

```python
stream1 = cuda.stream()
stream2 = cuda.stream()

# 数据块1：在流1传输
d_chunk1 = cuda.to_device(data[:half], stream=stream1)
# 数据块2：在流2传输（与流1传输重叠）
d_chunk2 = cuda.to_device(data[half:], stream=stream2)

# 在流1处理数据块1
kernel[blocks, threads, stream1](d_chunk1, d_out1)
# 在流2处理数据块2（与流1计算重叠）
kernel[blocks, threads, stream

## 随机数生成

Numba 使用 xoroshiro128+ 算法提供 GPU 原生的随机数生成功能。

```python
from numba import cuda
from numba.cuda.random import (
    create_xoroshiro128p_states,
    xoroshiro128p_uniform_float32,
    xoroshiro128p_uniform_float64,
    xoroshiro128p_normal_float32,
    xoroshiro128p_normal_float64,
)

# 创建 RNG 状态——每个线程一个
n_threads = 256 * 128
rng_states = create_xoroshiro128p_states(n_threads, seed=42)

@cuda.jit
def monte_carlo_pi(rng_states, iterations, out):
    gid = cuda.grid(1)
    if gid < out.size:
        inside = 0
        for _ in range(iterations):
            x = xoroshiro128p_uniform_float32(rng_states, gid)
            y = xoroshiro128p_uniform_float32(rng_states, gid)
            if x**2 + y**2 <= 1.0:
                inside += 1
        out[gid] = inside / iterations * 4.0

monte_carlo_pi[128, 256](rng_states, 10000, d_out)
```

**提示：** RNG 状态消耗的内存与线程数成正比。对于大规模问题，使用网格步长循环来限制所需状态数量。

---

## 协作组

适用于需要在网格中所有块（而不仅仅是单个块内）进行同步的算法。

```python
@cuda.jit
def iterative_kernel(M):
    col = cuda.grid(1)
    g = cuda.cg.this_grid()  # 获取网格组

    for row in range(1, M.shape[0]):
        M[row, col] = M[row - 1, col] + 1
        g.sync()  # 全局屏障——所有块在此等待

# 查询协作启动的最大网格尺寸
overload = iterative_kernel.overloads[signature]
max_blocks = overload.max_cooperative_grid_blocks(block_dim)
```

当检测到 `g.sync()` 时，协作启动会自动触发。网格大小不得超过 `max_cooperative_grid_blocks()`。

---

## 常见模式

### 使用共享内存的分块矩阵乘法

这是共享内存优化的经典示例——将 A 和 B 的块加载到快速的共享内存中，以减少对慢速全局内存的访问。

```python
from numba import cuda, float32
import numpy as np

TPB = 16  # 分块/块大小

@cuda.jit
def matmul_shared(A, B, C):
    sA = cuda.shared.array((TPB, TPB), dtype=float32)
    sB = cuda.shared.array((TPB, TPB), dtype=float32)

    x, y = cuda.grid(2)
    tx, ty = cuda.threadIdx.x, cuda.threadIdx.y

    tmp = float32(0.0)
    for tile in range(cuda.gridDim.x):
        # 将分块加载到共享内存中（带边界检查）
        col = tx + tile * TPB
        row = ty + tile * TPB
        sA[ty, tx] = A[y, col] if (y < A.shape[0] and col < A.shape[1]) else 0
        sB[ty, tx] = B[row, x] if (x < B.shape[1] and row < B.shape[0]) else 0
        cuda.syncthreads()

        # 从此分块计算部分点积
        for k in range(TPB):
            tmp += sA[ty, k] * sB[k, tx]
        cuda.syncthreads()

    if y < C.shape[0] and x < C.shape[1]:
        C[y, x] = tmp
```

### 并行前缀和（扫描）

```python
@cuda.jit
def inclusive_scan(data, output):
    shared = cuda.shared.array(256, dtype=float32)
    tid = cuda.threadIdx.x
    i = cuda.grid(1)

    shared[tid] = data[i] if i < data.size else 0
    cuda.syncthreads()

    # 上扫
    offset = 1
    while offset < cuda.blockDim.x:
        if tid >= offset:
            shared[tid] += shared[tid - offset]
        offset *= 2
        cuda.syncthreads()

    if i < data.size:
        output[i] = shared[tid]
```

### 共享内存归约

```python
@cuda.jit
def block_reduce_sum(data, partial_sums):
    shared = cuda.shared.array(256, dtype=float32)
    tid = cuda.threadIdx.x
    i = cuda.grid(1)

    shared[tid] = data[i] if i < data.size else 0.0
    cuda.syncthreads()

    # 在共享内存中进行树形归约
    s = cuda.blockDim.x // 2
    while s > 0:
        if tid < s:
            shared[tid] += shared[tid + s]
        s //= 2
        cuda.syncthreads()

    # 每个块的线程 0 写入该块的和
    if tid == 0:
        partial_sums[cuda.blockIdx.x] = shared[0]
```

### 模板/邻域访问模式

```python
@cuda.jit
def stencil_1d(data, output, radius):
    shared = cuda.shared.array(288, dtype=float32)  # 块维度 + 2*半径
    tid = cuda.threadIdx.x
    i = cuda.grid(1)

    # 将中心区域和光晕区域加载到共享内存中
    shared[tid + radius] = data[i] if i < data.size else 0
    if tid < radius:
        shared[tid] = data[i - radius] if i >= radius else 0
        shared[tid + cuda.blockDim.x + radius] = (
            data[i + cuda.blockDim.x] if i + cuda.blockDim.x < data.size else 0
        )
    cuda.syncthreads()

    if i < data.size:
        total = float32(0.0)
        for j in range(-radius, radius + 1):
            total += shared[tid + radius + j]
        output[i] = total / (2 * radius + 1)
```

---

## 性能优化

### GPU 特定技巧

1. **最小化主机与设备之间的传输**。使用 `cuda.to_device()` 并在多个内核调用之间将数据保留在 GPU 上。每次 PCI-e 传输都很昂贵（约 12 GB/s），而 GPU 内存带宽则高得多（约 900+ GB/s）。

2. **使用共享内存**存储块内线程间复用的数据。共享内存带宽比全局内存高约 10-100 倍。

3. **合并内存访问**。相邻线程（连续的 `threadIdx.x`）应访问相邻的内存位置。这使硬件可以将访问合并为更少的宽事务。

4. **为占用率选择合适的块大小**。对于一维情况，每个块 128-256 个线程；对于二维情况，使用 (16,16) 或 (32,32)。线程太少会导致 GPU 利用率不足；太多则可能限制每个线程的寄存器/共享内存。

5. **当不需要严格的 IEEE-754 标准时，使用 `fastmath=True`**。启用 FMA、快速平方根/除法，以及更快的三角函数/指数/对数（针对 float32）。

6. **在精度允许的情况下，优先使用 float32 而非 float64**。GPU 的 float32 吞吐量比 float64 高 2 到 32 倍（具体取决于 GPU，消费级 GPU 对 float64 的惩罚尤为严重）。

7. **使用流**来重叠数据传输与计算。

8. **在性能关键部分使用 `cuda.defer_cleanup()`**，以防止内存释放导致的隐式同步。

9. **当占用率是瓶颈时，使用 `max_registers` 参数限制寄存器使用**。

10. **使用网格步长循环**将网格大小与问题规模解耦，提高灵活性。

### 避免事项

- 不要在内核中使用 Python 对象、字符串或动态内存分配——Numba CUDA 仅支持受限的 Python 子集。
- 不要在分支条件不同的地方使用 `syncthreads()`——如果块内线程通过屏障的路径不同，行为将未定义（死锁或数据损坏）。
- 在 CPU 读取结果前不要忘记 `cuda.synchronize()`——内核启动是异步的。
- 不要为微小数据量启动内核——小数组场景下内核启动开销（约 5-20μs）会占主导地位。

---

## 调试

### CUDA 模拟器

在 CPU 上运行 CUDA 代码进行调试——支持在内核中使用 `print()` 和 `pdb`。

```bash
export NUMBA_ENABLE_CUDASIM=1
python your_script.py
```

模拟器每次运行一个块，为每个 CUDA 线程生成一个线程。支持共享/本地/常量内存、原子操作和 `syncthreads()`。

### 调试特定线程

```python
