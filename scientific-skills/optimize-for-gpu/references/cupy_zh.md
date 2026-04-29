# CuPy 参考指南

CuPy 是一个兼容 NumPy/SciPy 的 GPU 加速数组计算库。它封装了 NVIDIA 的优化库（cuBLAS、cuFFT、cuSOLVER、cuSPARSE、cuRAND），因此标准数组操作已高度优化。大多数 NumPy 代码只需更改导入即可运行。

> **完整文档：** https://docs.cupy.dev/en/stable/

## 目录

1. [安装与设置](#安装与设置)
2. [无缝替换模式](#无缝替换模式)
3. [核心 API：cupy.ndarray](#核心-api)
4. [支持的操作](#支持的操作)
5. [自定义内核](#自定义内核)
6. [内核融合](#内核融合)
7. [内存管理](#内存管理)
8. [流与异步操作](#流与异步操作)
9. [多 GPU](#多-gpu)
10. [性能优化](#性能优化)
11. [互操作性](#互操作性)
12. [与 NumPy 的主要差异](#与-numpy-的主要差异)
13. [常见陷阱](#常见陷阱)
14. [环境变量](#环境变量)

---

## 安装与设置

在所有安装说明、文档字符串、注释和错误消息中始终使用 `uv add`（切勿使用 `pip install` 或 `conda install`）。

```bash
uv add cupy-cuda12x    # 适用于 CUDA 12.x（最常用）
```

验证安装：
```python
import cupy as cp
print(cp.cuda.runtime.getDeviceCount())  # >=1 表示 GPU 可用
print(cp.show_config())                  # 完整环境信息
```

---

## 无缝替换模式

加速 NumPy 代码的最快方式：更改导入语句。

```python
# 之前（CPU）
import numpy as np
a = np.random.rand(10_000_000)
b = np.fft.fft(a)
c = np.sort(b.real)

# 之后（GPU）
import cupy as cp
a = cp.random.rand(10_000_000)
b = cp.fft.fft(a)
c = cp.sort(b.real)
```

### CPU 与 GPU 间的数据传输

```python
# NumPy → CuPy (CPU → GPU)
gpu_array = cp.asarray(numpy_array)     # 若已在当前设备则为零拷贝
gpu_array = cp.array(numpy_array)       # 始终复制数据

# CuPy → NumPy (GPU → CPU)
cpu_array = cp.asnumpy(gpu_array)       # 复制到 CPU
cpu_array = gpu_array.get()             # 效果相同
```

### 编写 CPU/GPU 通用代码

```python
def normalize(x):
    xp = cp.get_array_module(x)  # 根据输入返回 cupy 或 numpy
    return x / xp.linalg.norm(x)

# 同时兼容 NumPy 和 CuPy 数组
normalize(numpy_array)   # 在 CPU 上运行
normalize(cupy_array)    # 在 GPU 上运行
```

CuPy 数组实现了 `__array_ufunc__` 和 `__array_function__`，因此当传入 CuPy 数组时（NumPy >= 1.17），NumPy 函数会自动调度到 CuPy。

---

## 核心 API

`cupy.ndarray` 镜像 `numpy.ndarray` —— 相同属性（`shape`、`dtype`、`ndim`、`size`、`strides`、`T`），额外增加 `device`（数组所在的 GPU 设备）。

**重要提示：** `cupy.ndarray` 和 `numpy.ndarray` 不能隐式转换。每次转换都会引发主机-设备数据传输。

### 数组创建

```python
cp.empty((1000, 1000), dtype=cp.float32)
cp.zeros((1000,), dtype=cp.float64)
cp.ones((512, 512), dtype=cp.float32)
cp.full((100,), fill_value=3.14, dtype=cp.float32)
cp.arange(0, 100, 0.1)
cp.linspace(0, 1, 1000)
cp.eye(100)
cp.random.rand(1000, 1000)                    # 均匀分布 [0, 1)
cp.random.randn(1000, 1000)                   # 标准正态分布
cp.random.default_rng(42).normal(0, 1, 1000)  # 生成器 API
```

CuPy 的随机函数支持 `dtype` 参数（float32/float64）—— 不同于 NumPy 始终返回 float64。当不需要双精度时使用 `dtype=cp.float32`。

---

## 支持的操作

CuPy 实现了大多数 NumPy 和大量 SciPy 功能。所有操作均支持 GPU 加速。

### 数组数学与逐元素操作
`sin`, `cos`, `tan`, `exp`, `log`, `log2`, `log10`, `sqrt`, `square`, `abs`, `power`, `add`, `subtract`, `multiply`, `divide`, `mod`, `clip`, `sign`, `ceil`, `floor`, `round`, `maximum`, `minimum`

### 归约操作
`sum`, `prod`, `mean`, `std`, `var`, `min`, `max`, `argmin`, `argmax`, `cumsum`, `cumprod`, `any`, `all`, `nansum`, `nanmean`, `nanstd`, `nanvar`

### 线性代数（`cupy.linalg` — 基于 cuBLAS/cuSOLVER）
`dot`, `matmul`, `@` 运算符, `tensordot`, `einsum`, `inner`, `outer`, `cholesky`, `qr`, `svd`, `eig`, `eigh`, `eigvalsh`, `norm`, `solve`, `inv`, `pinv`, `lstsq`, `det`, `slogdet`, `matrix_rank`, `matrix_power`

### FFT（`cupy.fft` — 基于 cuFFT）
`fft`, `ifft`, `fft2`, `ifft2`, `fftn`, `ifftn`, `rfft`, `irfft`, `rfft2`, `irfft2`, `rfftn`, `irfftn`, `fftfreq`, `rfftfreq`, `fftshift`, `ifftshift`

### 排序与搜索
`sort`, `argsort`, `partition`, `argpartition`, `argmin`, `argmax`, `where`, `nonzero`, `unique`, `searchsorted`

### 数组操作
`reshape`, `ravel`, `flatten`, `transpose`, `swapaxes`, `concatenate`, `stack`, `vstack`, `hstack`, `dstack`, `split`, `hsplit`, `vsplit`, `tile`, `repeat`, `pad`, `flip`, `fliplr`, `flipud`, `roll`, `rot90`, `broadcast_to`, `expand_dims`, `squeeze`

### 稀疏矩阵（`cupyx.scipy.sparse`）
CSR、CSC、COO 格式。矩阵-向量乘法、矩阵-矩阵乘法、格式转换。基于 cuSPARSE。

### 信号处理（`cupyx.scipy.signal`）
卷积、相关、滤波、窗函数。

### 特殊函数（`cupyx.scipy.special`）
贝塞尔函数、误差函数、伽马函数等。

### 统计
`mean`, `median`, `std`, `var`, `percentile`, `quantile`, `corrcoef`, `cov`, `histogram`, `bincount`, `digitize`

---

## 自定义内核

当内置操作不满足需求时，CuPy 提供多种编写自定义 GPU 代码的方式，按复杂度从低到高排序。

### ElementwiseKernel — 自定义逐元素操作

CuPy 自动处理索引和广播。只需用 C++ 编写逐元素逻辑。

```python
squared_diff = cp.ElementwiseKernel(
    'float32 x, float32 y',   # 输入参数
    'float32 z',               # 输出参数
    'z = (x - y) * (x - y)',  # 逐元素操作（C++ 代码）
    'squared_diff'             # 内核名称
)

result = squared_diff(a, b)  # 自动支持广播
```

**泛型内核：** 使用单字母类型占位符。相同字母表示相同类型，在调用时根据参数解析。

```python
generic_squared_diff = cp.ElementwiseKernel(
    'T x, T y', 'T z',
    'z = (x - y) * (x - y)',
    'generic_squared_diff'
)
# 支持 float32、float64 等 — 类型根据输入推断
```

**原始索引：** 添加 `raw` 前缀禁用自动索引。使用 `i` 作为循环索引。

```python
# 访问相邻元素 — raw 禁用自动索引以便手动索引
stencil = cp.ElementwiseKernel(
    'raw T x', 'T y',
    'y = (x[i > 0 ? i-1 : 0] + x[i] + x[i < _ind.size()-1 ? i+1 : _ind.size()-1]) / 3',
    'stencil_1d'
)
```

### ReductionKernel — 自定义归约操作

四步归约：映射每个元素、成对归约、结果后处理。

```python
l2norm = cp.ReductionKernel(
    'T x',           # 输入
    'T y',           # 输出
    'x * x',         # 映射：对每个元素平方
    'a + b',         # 归约：成对求和（a, b 为二元操作数）
    'y = sqrt(a)',   # 后映射：对最终和取平方根
    '0',             # 单位元
    'l2norm'         # 内核名称
)

norm = l2norm(array)        # 完全归约 → 标量
norms = l2norm(matrix, axis=1)  # 沿轴归约 → 向量
```

### RawKernel — 完整 CUDA C/C++ 控制

完全控制网格、线程块、共享内存 — 编写原生 CUDA 代码。

```python
kernel_code = r'''
extern "C" __global__
void vector_add(const float* a, const float* b, float* c, int n) {
    int tid = blockDim.x * blockIdx.x + threadIdx.x;
    if (tid < n) {
        c[tid] = a[tid] + b[tid];
    }
}
'''
vector_add = cp.RawKernel(kernel_code, 'vector_add')

n = 1_000_000
a = cp.random.rand(n, dtype=cp.float32)
b = cp.random.rand(n, dtype=cp.float32)
c = cp.zeros(n, dtype=cp.float32)

threads = 256
blocks = (n + threads - 1) // threads
vector_add((blocks,), (threads,), (a, b, c, n))  # (网格, 线程块, 参数)
```

**RawKernel 重要注意事项：**
- 忽略数组视图/步幅 — `matrix.T` 被视为 `matrix`。需自行处理步幅。
- 使用 `extern "C"` 避免 C++ 名称修饰。
- 对于复数，包含 `<cupy/complex.cuh>`。
- 编译后的二进制文件缓存于 `~/.cupy/kernel_cache`。

**CuPy 数据类型到 CUDA 类型映射：**

| CuPy 类型 | CUDA 类型 |
|-----------|-----------|
| `float16` | `half` |
| `float32` | `float` |
| `float64` | `double` |
| `int32` | `int` |
| `int64` | `long long` |
| `complex64` | `complex<float>` |
| `complex128` | `complex<double>` |

### RawModule — 大型 CUDA 代码库

适用于多内核 CUDA 文件或预编译二进制文件：

```python
module = cp.RawModule(code=cuda_source)       # 从源码字符串
module = cp.RawModule(path='kernels.cu')      # 从文件
module = cp.RawModule(path='kernels.cubin')   # 从预编译文件

kernel = module.get_function('my_kernel')
kernel((blocks,), (threads,), (args...))
```

### JIT 内核（cupyx.jit.rawkernel）— Python 语法的 CUDA 内核

使用 Python 语法编写 CUDA 风格的内核。

```python
@cupyx.jit.rawkernel()
def my_kernel(x, y, size):
    tid = cupyx.jit.grid(1)
    if tid < size:
        y[tid] = x[tid] * 2.0

my_kernel[blocks, threads](x, y, n)
```

可用的 JIT 原语：
- `cupyx.jit.threadIdx`, `blockIdx`, `blockDim`, `gridDim`
- `cupyx.jit.grid(ndim)`, `gridsize(ndim)`
- `cupyx.jit.syncthreads()`, `syncwarp()`
- `cupyx.jit.shared_memory(dtype, size)`
- `cupyx.jit.atomic_add/min/max/and/or/xor(array, index, value)`
- Warp 洗牌：`shfl_sync`, `shfl_up_sync`, `shfl_down_sync`, `shfl_xor_sync`

**限制：** 不能在 Python REPL 中使用（需要源码访问）。请在 .py 文件中使用。

---

## 内核融合

将多个逐元素操作合并为单个内核启动 — 消除中间数组并减少内核启动开销。

```python
@cp.fuse()
def fused_op(x, y):
    return cp.sqrt((x - y) ** 2 + 1.0)

# 编译为单个内核而非多个
result = fused_op(a, b)
```

**限制：** 仅融合逐元素和简单归约操作。不支持 `matmul`、`reshape`、索引等操作。

---

## 内存管理

### 内存池（默认行为）

CuPy 默认使用内存池 — 这对性能至关重要。该池缓存已释放的 GPU 内存以供重用，避免昂贵的 `cudaMalloc`/`cudaFree` 调用和隐式同步。

**关键洞察：** 当数组超出作用域时，内存不会释放到操作系统 — 而是返回内存池。这是预期行为（在 `nvidia-smi` 中显示为仍占用状态）。

```python
mempool = cp.get_default_memory_pool()
mempool.used_bytes()        # CuPy 数组当前分配量
mempool.total_bytes()       # 池持有的总量（含空闲块）
mempool.free_all_blocks()   # 释放所有未用内存到操作系统

pinned_mempool = cp.get_default_pinned_memory_pool()
pinned_mempool.free_all_blocks()
```

### 限制 GPU 内存

```python
mempool = cp.get_default_memory_pool()
with cp.cuda.Device(0):
    mempool.set_limit(size=4 * 1024**3)  # GPU 0 限制为 4 GiB
```

或通过环境变量（在 `import cupy` 前设置）：
```bash
export CUPY_GPU_MEMORY_LIMIT="50%"     # GPU 总内存百分比
export CUPY_GPU_MEMORY_LIMIT="4294967296"  # 字节数
```

### 托管（统一）内存

数据在 CPU 和 GPU 间自动迁移。适用于数据超出 GPU 内存容量的场景。

```python
cp.cuda.set_allocator(cp.cuda.MemoryPool(cp.cuda.malloc_managed).malloc)
```

### 固定内存加速传输

```python
# 高级 API
pinned_array = cupyx.empty_pinned((1000,), dtype=np.float32)
pinned_array = cupyx.zeros_pinned((1000,), dtype=np.float32)

# 这些是由页锁定内存支持的 NumPy 数组 — 传输到 GPU 更快
```

### 禁用内存池

```python
cp.cuda.set_allocator(None)                    # 禁用设备内存池
cp.cuda.set_pinned_memory_allocator(None)      # 禁用固定内存池
```

必须在任何 CuPy 操作前执行。

### 使用 RMM（RAPIDS 内存管理器）

当 CuPy 与 cuDF/RAPIDS 协同使用时，统一分配器：

```python
import rmm
rmm.reinitialize(pool_allocator=True)
cp.cuda.set_allocator(rmm.rmm_cupy_allocator)
```

---

## 流与异步操作

流支持计算与数据传输重叠，以及并发执行多个操作。

```python
stream = cp.cuda.Stream()

# 上下文管理器模式
with stream:
    d_data = cp.asarray(host_data)     # 在此流上执行 H→D 传输
    result = cp

```markdown
export CUPY_CUDA_PER_THREAD_DEFAULT_STREAM=1
```

启用每线程默认流，提升多线程应用程序的并发性能。

---

## 多 GPU 支持

```python
# 设置当前设备
cp.cuda.Device(0).use()

# 上下文管理器
with cp.cuda.Device(1):
    x = cp.array([1, 2, 3])  # 在 GPU 1 上分配

# 检查数组所在设备
print(x.device)  # Device 1
```

若 GPU 拓扑支持，跨设备操作可通过 P2P（点对点）内存访问实现。使用 `cp.asarray()` 在设备间显式传输数组。

### 每设备内存限制

```python
mempool = cp.get_default_memory_pool()
with cp.cuda.Device(0):
    mempool.set_limit(size=4 * 1024**3)
with cp.cuda.Device(1):
    mempool.set_limit(size=4 * 1024**3)
```

---

## 性能优化

### 基准测试（关键第一步）

**切勿在 GPU 代码中使用 `time.perf_counter()` 或 `%timeit`** —— 它们仅测量 CPU 时间而非 GPU 执行时间。CuPy 操作是异步的。

```python
from cupyx.profiler import benchmark

result = benchmark(my_function, (arg1, arg2), n_repeat=100, n_warmup=10)
print(result)  # 显示带统计数据的 CPU/GPU 耗时
```

在 IPython/Jupyter 中：
```python
%load_ext cupy
%gpu_timeit my_function(args)
```

### 一次性开销

- **上下文初始化：** 首次调用 CuPy 可能耗时 1-5 秒（创建 CUDA 上下文），此开销仅出现一次。
- **内核即时编译：** 首次调用任何操作会触发即时内核编译。编译结果缓存于 `~/.cupy/kernel_cache`，在 CI/CD 流程中需保留此目录。

### CUB 与 cuTENSOR 加速

```bash
# CuPy v11+ 默认启用 CUB
export CUPY_ACCELERATORS=cub          # 仅 CUB（默认）
export CUPY_ACCELERATORS=cub,cutensor # 同时启用（需安装 cuTENSOR）
```

CUB 加速：归约操作（`sum`, `prod`, `amin`, `amax`, `argmin`, `argmax`）、包含性扫描（`cumsum`）、直方图、稀疏矩阵向量乘法及 `ReductionKernel`。对归约操作可带来约 100 倍加速。

cuTENSOR 加速：二元逐元素 ufuncs、归约操作、张量收缩。

### 核心优化策略

1. **优先使用 float32 而非 float64。** 消费级 GPU 的 float32 吞吐量高 2-32 倍。精度允许时使用 `dtype=cp.float32`。
2. **最小化 CPU-GPU 数据传输。** 每次 `cp.asnumpy()` / `.get()` 都会触发同步和 PCI-e 传输。尽量将数据保留在 GPU 上。
3. **使用内核融合。** `@cp.fuse()` 将多个逐元素操作合并为单一内核，消除中间数组。
4. **批量操作。** 少量大型操作优于多次小型操作（每次内核启动开销约 5-20μs）。
5. **预分配输出数组。** 在 ufuncs 中使用 `out=` 参数避免重复分配：
   ```python
   cp.add(a, b, out=result)  # 写入现有数组
   ```
6. **使用原地操作。** `a += b` 避免分配新数组。
7. **使用流** 重叠计算与数据传输。
8. **通过 NVTX 标记进行性能分析**（用于 Nsight Systems）：
   ```python
   with cupyx.profiler.time_range('my_operation', color_id=0):
       result = heavy_computation()
   ```

### 决策树：选择内核方案

1. **能否用 NumPy 操作实现？** → 使用内置 CuPy 函数（开发最快，通常性能最佳）
2. **多个链式逐元素操作？** → 使用 `@cp.fuse()`
3. **需广播的自定义逐元素操作？** → 使用 `ElementwiseKernel`
4. **自定义归约操作？** → 使用 `ReductionKernel`
5. **需完全控制网格/块/共享内存？** → 使用 `RawKernel` 或 `cupyx.jit.rawkernel`
6. **大型 CUDA 代码库？** → 使用 `RawModule`

---

## 互操作性

CuPy 通过 CUDA 数组接口和 DLPack 协议与其他 GPU 库互操作——两者均支持零拷贝数据共享。

### NumPy

```python
# NumPy 函数自动调度至 CuPy (NumPy >= 1.17)
import numpy as np
result = np.sum(cupy_array)  # 调度至 CuPy，返回 CuPy 数组
```

### Numba

```python
from numba import cuda

@cuda.jit
def numba_kernel(x, y):
    i = cuda.grid(1)
    if i < x.shape[0]:
        y[i] = x[i] * 2

# CuPy 数组直接传递至 Numba 内核——零拷贝
a = cp.arange(1000, dtype=cp.float32)
b = cp.zeros_like(a)
numba_kernel[4, 256](a, b)
```

### PyTorch

```python
import torch

# CuPy → PyTorch (通过 CUDA 数组接口零拷贝)
cupy_array = cp.array([1.0, 2.0, 3.0], dtype=cp.float32)
torch_tensor = torch.as_tensor(cupy_array, device='cuda')

# PyTorch → CuPy (零拷贝)
cupy_array = cp.asarray(torch_tensor)

# 通过 DLPack (同样零拷贝)
cupy_array = cp.from_dlpack(torch_tensor)
torch_tensor = torch.from_dlpack(cupy_array)
```

### cuDF

```python
import cudf

# cuDF → CuPy
arr = df.to_cupy()
arr = cp.asarray(df['column'])

# CuPy → cuDF
df = cudf.DataFrame(cupy_array)
s = cudf.Series(cupy_array)
```

### 原始指针互操作

```python
# 导出指针
ptr = cupy_array.data.ptr  # 原始设备指针（整数形式）

# 导入外部指针
mem = cp.cuda.UnownedMemory(ptr, size_bytes, owner=owner_obj)
memptr = cp.cuda.MemoryPointer(mem, offset=0)
arr = cp.ndarray(shape, dtype, memptr=memptr)
```

---

## 与 NumPy 的关键差异

以下行为差异可能导致未察觉的 bug。

1. **归约操作返回 0 维数组而非标量。** `cp.sum(a)` 返回 0 维 `cupy.ndarray` 而非 Python 浮点数。这避免了隐式 GPU-CPU 同步。需标量时使用 `.item()`。
2. **越界索引静默回绕。** NumPy 会抛出 `IndexError`；CuPy 无错误回绕。
3. **重复索引赋值结果未定义。** `a[[0, 0]] = [1, 2]` —— NumPy 存储末值；CuPy 存储未定义值（GPU 竞态条件）。
4. **浮点到整型的边界转换不同。** 负浮点数转无符号整型或无穷大转整型的结果与 NumPy 不同。
5. **不支持字符串/对象数据类型。** CuPy 仅支持数值类型。无含字符串字段的结构化数组。
6. **CuPy ufuncs 要求 CuPy 数组。** 与 NumPy ufuncs 不同，CuPy ufuncs 不接受列表或 NumPy 数组——需预先转换。
7. **随机种子数组被哈希处理。** 数组种子产生的熵值低于 NumPy 方法。

---

## 常见陷阱

1. **使用 CPU 计时器测量。** GPU 操作是异步的。`time.perf_counter()` 仅测量操作入队时间而非执行时间。始终使用 `cupyx.profiler.benchmark()`。
2. **不必要的往返传输。** 每次 `cp.asnumpy()` / `.get()` 都会同步 GPU 并跨 PCI-e 复制数据。重构代码以保持数据在 GPU 上。
3. **内存池导致的"内存泄漏"。** 内存池缓存已释放块。`nvidia-smi` 将其显示为已分配。使用 `mempool.free_all_blocks()` 释放。
4. **首次调用延迟。** CUDA 上下文初始化 + 内核即时编译。基准测试前需预热。
5. **混合使用设备。** 未显式传输就在 GPU 1 上使用 GPU 0 的数组可能导致失败或性能下降。
6. **RawKernel 忽略视图。** 传递给 RawKernel 的转置或切片数组被视为原始连续布局，必须手动处理步幅。
7. **读取结果前忘记 `synchronize()`。** 若将数据传回 CPU 或用于非 CuPy 代码，需确保 GPU 已完成计算。

---

## 环境变量

| 变量 | 用途 |
|----------|---------|
| `CUPY_ACCELERATORS` | 后端列表：`cub`, `cutensor` (v11+ 默认 `cub`) |
| `CUPY_CACHE_DIR` | 内核缓存目录 (默认：`~/.cupy/kernel_cache`) |
| `CUPY_GPU_MEMORY_LIMIT` | GPU 内存限制（字节或 `"50%"`） |
| `CUPY_CACHE_SAVE_CUDA_SOURCE` | 设为 `1` 可导出内核源码用于性能分析 |
| `CUPY_CUDA_PER_THREAD_DEFAULT_STREAM` | 设为 `1` 启用每线程默认流 |
```
