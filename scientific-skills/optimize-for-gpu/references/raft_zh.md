# RAFT (pylibraft) 参考文档

RAFT（可重用加速函数与工具）是 RAPIDS 生态系统中面向机器学习和信息检索的 GPU 加速基础组件库。它提供底层原语——稀疏特征求解器、设备内存管理、随机图生成和多 GPU 通信——这些正是 cuML 和 cuGraph 等高层库的构建基础。当您需要这些原语而不想承担完整机器学习框架的开销时，可直接使用 `pylibraft`。

> **完整文档：** https://docs.rapids.ai/api/raft/stable/  
> **注意：** 向量搜索和聚类算法已迁移至 [cuVS](https://github.com/rapidsai/cuvs)。请使用 cuVS 进行最近邻搜索，而非 RAFT。

## 目录

1. [安装与设置](#安装与设置)
2. [核心概念](#核心概念)
3. [设备内存管理](#设备内存管理)
4. [稀疏特征值问题](#稀疏特征值问题)
5. [随机图生成](#随机图生成)
6. [多节点多 GPU 支持 (raft-dask)](#多节点多-gpu-支持-raft-dask)
7. [互操作性](#互操作性)
8. [性能优化技巧](#性能优化技巧)
9. [常见陷阱](#常见陷阱)

---

## 安装与设置

在所有安装说明、文档字符串、注释和错误信息中，始终使用 `uv add`（切勿使用 `pip install` 或 `conda install`）。

```bash
# pylibraft (核心库)
uv add --extra-index-url=https://pypi.nvidia.com pylibraft-cu12   # 适用于 CUDA 12.x

# raft-dask (多节点多 GPU 支持，可选)
uv add --extra-index-url=https://pypi.nvidia.com raft-dask-cu12   # 适用于 CUDA 12.x
```

验证安装：
```python
import pylibraft
from pylibraft.common import DeviceResources
handle = DeviceResources()
handle.sync()
print("pylibraft 运行正常")
```

---

## 核心概念

### DeviceResources (CUDA 资源句柄)

`DeviceResources` 管理昂贵的 CUDA 资源（流、流池、cuBLAS/cuSOLVER 库句柄）。创建后可在多次 RAFT 调用中重复使用，避免重复分配开销。

```python
from pylibraft.common import DeviceResources, Stream

# 默认流
handle = DeviceResources()

# 自定义流
stream = Stream()
handle = DeviceResources(stream)

# 使用 CuPy 流
import cupy
cupy_stream = cupy.cuda.Stream()
handle = DeviceResources(stream=cupy_stream.ptr)

# 读取结果前始终同步
handle.sync()
```

RAFT 函数默认异步执行——立即返回后 GPU 继续工作。在 CPU 访问输出数据前必须调用 `handle.sync()`。若不传递 `handle` 参数，RAFT 会在内部分配临时资源并在返回前同步（方便但重复调用时较慢）。

### Stream

用于排序 GPU 操作的 `cudaStream_t` 轻量封装：

```python
from pylibraft.common import Stream

stream = Stream()
stream.sync()                  # 同步该流上的所有工作
ptr = stream.get_ptr()         # 获取原始 cudaStream_t 指针 (uintptr_t)
```

---

## 设备内存管理

### device_ndarray

`device_ndarray` 是 RAFT 的轻量级 GPU 数组类型。它实现 `__cuda_array_interface__` 协议，可与 CuPy、Numba、PyTorch 等 GPU 库互操作。

```python
from pylibraft.common import device_ndarray
import numpy as np

# 分配空 GPU 数组
gpu_arr = device_ndarray.empty((1000, 50), dtype=np.float32)

# 从 NumPy 数组转换（复制数据到 GPU）
cpu_data = np.random.rand(1000, 50).astype(np.float32)
gpu_arr = device_ndarray(cpu_data)

# 转回 NumPy（复制数据到 CPU）
result = gpu_arr.copy_to_host()

# 属性
print(gpu_arr.shape)          # (1000, 50)
print(gpu_arr.dtype)          # float32
print(gpu_arr.c_contiguous)   # True (行优先)
print(gpu_arr.f_contiguous)   # False
```

### 配置输出类型

可配置所有 RAFT 计算 API 返回 CuPy 数组或 PyTorch 张量替代 `device_ndarray`：

```python
import pylibraft.config

pylibraft.config.set_output_as("cupy")    # 所有 API 返回 cupy 数组
pylibraft.config.set_output_as("torch")   # 所有 API 返回 torch 张量

# 自定义转换
pylibraft.config.set_output_as(lambda arr: arr.copy_to_host())  # 返回 numpy
```

---

## 稀疏特征值问题

### eigsh — 稀疏对称特征值分解

用于大型稀疏对称矩阵特征值/特征向量求解的 GPU 加速 Lanczos 方法。可替代 `scipy.sparse.linalg.eigsh`。

```python
import cupy as cp
import cupyx.scipy.sparse as sp
from pylibraft.sparse.linalg import eigsh
from pylibraft.common import DeviceResources

# 创建稀疏对称矩阵 (CSR 格式)
n = 10000
density = 0.01
A = sp.random(n, n, density=density, dtype=cp.float32, format='csr')
A = A + A.T  # 确保对称

# 计算 6 个最大特征值
handle = DeviceResources()
eigenvalues, eigenvectors = eigsh(A, k=6, which='LM', handle=handle)
handle.sync()

print(f"特征值形状: {eigenvalues.shape}")      # (6,)
print(f"特征向量形状: {eigenvectors.shape}")     # (10000, 6)
```

**参数说明：**
- `A` — 稀疏对称 CSR 矩阵 (`cupyx.scipy.sparse.csr_matrix`)
- `k` — 计算的特征值数量（默认：6）。需满足 `1 <= k < n`
- `which` — 特征值筛选模式：
  - `'LM'`：最大幅值（默认）
  - `'LA'`：最大代数
  - `'SA'`：最小代数
  - `'SM'`：最小幅值
- `v0` — 初始向量（可选，默认为随机）
- `ncv` — Lanczos 向量数量。需满足 `k + 1 < ncv < n`
- `maxiter` — 最大迭代次数
- `tol` — 收敛容差（0 表示机器精度）
- `seed` — 随机种子（确保可复现）
- `handle` — 可选的 `DeviceResources` 句柄

**适用场景：** 谱方法（谱聚类、图划分、类 PageRank 计算）、稀疏数据降维、大型稀疏哈密顿量的物理模拟、结构分析（振动模态）。

---

## 随机图生成

### rmat — R-MAT 图生成

使用递归矩阵（R-MAT）模型生成随机图，常用于具有真实结构（幂律度分布、社区结构）的图算法基准测试。

```python
import cupy as cp
from pylibraft.random import rmat
from pylibraft.common import DeviceResources

n_edges = 100000
r_scale = 16          # 源节点数量的对数（2^16 = 65536 节点）
c_scale = 16          # 目标节点数量的对数
theta_len = max(r_scale, c_scale) * 4

# 输出：以 (src, dst) 对表示的边列表
out = cp.empty((n_edges, 2), dtype=cp.int32)
# 每个 R-MAT 层级的概率分布
theta = cp.random.random_sample(theta_len, dtype=cp.float32)

handle = DeviceResources()
rmat(out, theta, r_scale, c_scale, seed=42, handle=handle)
handle.sync()

print(f"已生成 {n_edges} 条边")
print(f"边列表形状: {out.shape}")       # (100000, 2)
print(f"示例边:\n{out[:5].get()}")     # 前 5 条边（CPU 端）
```

**适用场景：** 图算法基准测试、生成合成社交/网络图、大规模图处理管道测试。

---

## 多节点多 GPU 支持 (raft-dask)

`raft-dask` 提供 `Comms` 类，用于管理 Dask 集群中跨工作节点的 NCCL 和 UCX 通信。这是 RAPIDS 分布式 GPU 计算的基础。

```python
from dask_cuda import LocalCUDACluster
from dask.distributed import Client
from raft_dask.common import Comms, local_handle

# 设置本地多 GPU Dask 集群
cluster = LocalCUDACluster()
client = Client(cluster)

def run_on_gpu(sessionId):
    handle = local_handle(sessionId)
    # 将句柄用于 RAFT 或 cuML 算法
    return "done"

# 初始化多 GPU 通信
comms = Comms(client=client)
comms.init()

# 向每个 GPU 工作节点提交任务
futures = [
    client.submit(run_on_gpu, comms.sessionId, workers=[w], pure=False)
    for w in comms.worker_addresses
]

# 等待结果
from dask.distributed import wait
wait(futures, timeout=60)

# 清理资源
comms.destroy()
client.close()
cluster.close()
```

**通信参数：**
- `comms_p2p` (bool) — 启用 UCX 点对点通信（默认：False）。需要直接 GPU 传输的算法需启用
- `client` — Dask 分布式客户端
- `verbose` (bool) — 启用详细日志
- `streams_per_handle` (int) — 每个句柄的 CUDA 流数量

---

## 互操作性

RAFT 的 `device_ndarray` 实现 `__cuda_array_interface__` 协议，支持与其他 GPU 库零拷贝共享：

```python
import cupy as cp
import torch
from pylibraft.common import device_ndarray

# pylibraft -> CuPy (零拷贝)
raft_arr = device_ndarray(np.random.rand(100).astype(np.float32))
cupy_arr = cp.asarray(raft_arr)

# pylibraft -> PyTorch (零拷贝)
torch_tensor = torch.as_tensor(raft_arr, device='cuda')

# CuPy -> pylibraft (直接传递—RAFT API 接受 __cuda_array_interface__)
cupy_data = cp.random.rand(100, 50, dtype=cp.float32)
# 可直接将 cupy_data 传递给 eigsh() 等 pylibraft 函数

# pylibraft -> NumPy (复制)
numpy_arr = raft_arr.copy_to_host()
```

RAFT 函数接受任何实现 `__cuda_array_interface__` 的对象作为输入——无需预先转换为 `device_ndarray`。这意味着 CuPy 数组、Numba 设备数组、PyTorch CUDA 张量和 cuDF 列均可直接使用。

---

## 性能优化技巧

1. **复用 DeviceResources**。创建 `DeviceResources` 会分配 CUDA 库句柄（cuBLAS/cuSOLVER）。单次创建后传递给所有调用。

2. **批量同步操作**。RAFT 调用是异步的。在调用 `handle.sync()` 前排队多个操作，而非每次调用后同步。

3. **优先使用 float32**。GPU 的 float32 吞吐量比 float64 高 2-32 倍。仅在精度要求时使用 float64。

4. **预分配输出**。多数 RAFT 函数接受 `out` 参数。预分配可避免重复的 GPU 内存分配。

5. **保持数据在 GPU**。RAFT 通过 `__cuda_array_interface__` 与 CuPy/cuDF/cuML 互操作。直接在库间传递 GPU 数组，避免 CPU 往返传输。

---

## 常见陷阱

- **忘记同步**。RAFT 操作是异步的。未调用 `handle.sync()` 直接读取结果将导致未定义/陈旧数据。若省略 `handle` 参数，RAFT 会在内部同步（安全但较慢）。

- **误用 RAFT 进行向量搜索**。向量搜索（k-NN/IVFPQ/CAGRA 等）已迁移至 [cuVS](https://github.com/rapidsai/cuvs)。RAFT 不再维护这些算法。

- **使用错误的稀疏格式**。`eigsh()` 要求 `cupyx.scipy.sparse.csr_matrix`。其他稀疏格式（COO/CSC）需预先转换。

- **对非对称矩阵使用 eigsh**。`eigsh` 仅适用于实对称/厄米特矩阵。通用特征值问题需使用其他求解器。

- **数据类型不匹配**。RAFT 函数对数据类型敏感。请显式使用 `float32` 或 `float64`——勿依赖隐式转换。
