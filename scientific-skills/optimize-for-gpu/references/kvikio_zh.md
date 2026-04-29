# KvikIO 参考指南 —— 高性能 GPU 文件 I/O

KvikIO 是一个用于高性能文件 I/O 的 Python 和 C++ 库。它提供对 NVIDIA cuFile 的绑定，支持 GPUDirect Storage (GDS) —— 直接在存储设备和 GPU 内存之间读写数据，完全绕过 CPU 内存。当 GDS 不可用时，KvikIO 会优雅地回退到 POSIX I/O，同时仍能无缝处理主机和设备数据。

KvikIO 是 RAPIDS 生态系统的一部分，可与 CuPy、cuDF、Numba 及其他 GPU 库互操作。

## 目录

1. [安装](#installation)
2. [何时使用 KvikIO](#when-to-use-kvikio)
3. [CuFile —— 本地文件 I/O](#cufile--local-file-io)
4. [RemoteFile —— S3、HTTP、WebHDFS](#remotefile--s3-http-webhdfs)
5. [Zarr 集成](#zarr-integration)
6. [内存映射文件](#memory-mapped-files)
7. [运行时设置](#runtime-settings)
8. [性能优化](#performance-optimization)
9. [互操作性](#interoperability)
10. [常用模式](#common-patterns)
11. [常见陷阱](#common-pitfalls)

---

## 安装

```bash
# CUDA 12.x
uv add kvikio-cu12

# CUDA 13.x
uv add kvikio-cu13

# 支持 Zarr（可选）
uv add zarr
```

验证安装：

```python
import kvikio
# 检查 GDS 是否可用
import kvikio.cufile_driver
print(kvikio.cufile_driver.get("is_gds_available"))  # 若 GDS 已配置则返回 True
```

---

## 何时使用 KvikIO

在以下场景使用 KvikIO：
- **将大型二进制数据直接加载到 GPU** —— 避免标准 `open()` 或 NumPy 的 `fromfile()` 所需的 CPU 内存拷贝
- **将 GPU 数组写入磁盘** —— 直接从设备内存保存，无需先复制到主机
- **从远程存储（S3、HTTP、WebHDFS）读取到 GPU 内存** —— 跳过主机内存中转步骤
- **在 GPU 上处理 Zarr 数组** —— GDSStore 后端将数据块直接读入 CuPy 数组
- **I/O 成为性能瓶颈** —— GDS 可实现接近原生 NVMe 的带宽（每驱动器 6-7 GB/s），而标准 I/O 受限于 CPU 内存带宽
- **重叠 I/O 与计算** —— 非阻塞读写允许将数据加载与 GPU 计算流水线化

KvikIO 不适用于：
- 数据量小（< 1 MB）—— 内核启动和 GDS 开销占主导
- 读取结构化格式（CSV、Parquet、JSON）—— 应使用 cuDF 的优化读取器
- 仅需主机内存 —— 标准 Python I/O 更简单

---

## CuFile —— 本地文件 I/O

`kvikio.CuFile` 是本地文件 I/O 的主要接口，替代 Python 的 `open()` 用于 GPU 工作负载。

### 基础用法

```python
import cupy as cp
import kvikio

# 将 GPU 数组写入磁盘
a = cp.arange(1_000_000, dtype=cp.float32)
with kvikio.CuFile("data.bin", "w") as f:
    f.write(a)

# 读回数据
b = cp.empty(1_000_000, dtype=cp.float32)
with kvikio.CuFile("data.bin", "r") as f:
    f.read(b)

assert cp.all(a == b)
```

### API 方法

| 方法 | 阻塞 | 描述 |
|--------|----------|-------------|
| `read(buf, size, file_offset)` | 是 | 读取到设备或主机缓冲区 |
| `write(buf, size, file_offset)` | 是 | 从设备或主机缓冲区写入 |
| `pread(buf, size, file_offset)` | 否 | 非阻塞并行读，返回 `IOFuture` |
| `pwrite(buf, size, file_offset)` | 否 | 非阻塞并行写，返回 `IOFuture` |
| `raw_read(buf, size, file_offset)` | 是 | 底层单线程读（仅设备） |
| `raw_write(buf, size, file_offset)` | 是 | 底层单线程写（仅设备） |
| `raw_read_async(buf, stream, size, file_offset)` | 否 | CUDA 流异步读（仅设备） |
| `raw_write_async(buf, stream, size, file_offset)` | 否 | CUDA 流异步写（仅设备） |

文件模式：`"r"`（读）、`"w"`（写/截断）、`"a"`（追加）、`"+"`（读写）。

### 基于 Future 的非阻塞 I/O

`pread` 和 `pwrite` 将操作拆分为在线程池执行的任务，并返回 `IOFuture`：

```python
import cupy as cp
import kvikio

data = cp.empty(10_000_000, dtype=cp.float32)

with kvikio.CuFile("data.bin", "r") as f:
    # 启动两个非阻塞读取不同区段
    future1 = f.pread(data[:5_000_000])
    future2 = f.pread(data[5_000_000:], file_offset=5_000_000 * 4)

    # I/O 期间执行其他工作...

    # 等待完成
    bytes_read1 = future1.get()
    bytes_read2 = future2.get()
```

### 部分读写

```python
import cupy as cp
import kvikio

# 仅读取文件部分内容
buf = cp.empty(1000, dtype=cp.float32)
with kvikio.CuFile("data.bin", "r") as f:
    # 从字节偏移 4000 处读取 1000 个浮点数
    f.read(buf, size=4000, file_offset=4000)
```

### 主机内存支持

KvikIO 透明处理主机内存 —— 无需特殊 API：

```python
import numpy as np
import kvikio

# 从主机内存写入
a = np.arange(1_000_000, dtype=np.float32)
with kvikio.CuFile("data.bin", "w") as f:
    f.write(a)

# 读取到主机内存
b = np.empty_like(a)
with kvikio.CuFile("data.bin", "r") as f:
    f.read(b)
```

### GDS 对齐

GDS 在页面对齐的 I/O 下表现最佳。GPU 页大小为 4 KiB（4096 字节）：
- **文件偏移**：应为 4096 的倍数
- **传输大小**：应为 4096 的倍数

KvikIO 能正确处理未对齐 I/O，但会将其拆分为对齐和未对齐部分，因此对齐 I/O 更快。

---

## RemoteFile —— S3、HTTP、WebHDFS

`kvikio.RemoteFile` 将远程文件直接读入 GPU 或主机内存。

### HTTP/HTTPS

```python
import cupy as cp
import kvikio

buf = cp.empty(1_000_000, dtype=cp.float32)
with kvikio.RemoteFile.open_http("https://example.com/data.bin") as f:
    print(f.nbytes())  # 文件大小
    f.read(buf)
```

### AWS S3

```python
import cupy as cp
import kvikio

# 使用桶名 + 对象名（需 AWS 环境变量或显式凭证）
with kvikio.RemoteFile.open_s3("my-bucket", "data/file.bin") as f:
    buf = cp.empty(f.nbytes(), dtype=cp.uint8)
    f.read(buf)

# 使用 S3 URL
with kvikio.RemoteFile.open_s3_url("s3://my-bucket/data/file.bin") as f:
    buf = cp.empty(f.nbytes(), dtype=cp.uint8)
    f.read(buf)

# 公开 S3（无需凭证）
with kvikio.RemoteFile.open_s3_public("s3://public-bucket/data.bin") as f:
    buf = cp.empty(f.nbytes(), dtype=cp.uint8)
    f.read(buf)

# 预签名 URL
with kvikio.RemoteFile.open_s3_presigned_url(presigned_url) as f:
    buf = cp.empty(f.nbytes(), dtype=cp.uint8)
    f.read(buf)
```

AWS 凭证来自环境变量（`AWS_DEFAULT_REGION`、`AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`）或可通过关键字参数传递。

### 自动检测端点类型

```python
import kvikio

# KvikIO 根据 URL 自动推断协议
with kvikio.RemoteFile.open("s3://bucket/object") as f:
    ...

with kvikio.RemoteFile.open("https://example.com/file.bin") as f:
    ...
```

### WebHDFS

```python
import kvikio

with kvikio.RemoteFile.open_webhdfs("http://namenode:9870/path/to/file") as f:
    buf = cp.empty(f.nbytes(), dtype=cp.uint8)
    f.read(buf)
```

### RemoteFile 的主机内存支持

RemoteFile 同样轻松支持主机内存：

```python
import numpy as np
import kvikio

with kvikio.RemoteFile.open_http("https://example.com/data.bin") as f:
    buf = np.empty(f.nbytes(), dtype=np.uint8)
    f.read(buf)
```

---

## Zarr 集成

KvikIO 为 Zarr（3.x 版本）提供 GPU 存储后端，支持通过 GDS 直接在 GPU 内存中读写分块 N 维数组。

```python
import zarr
from kvikio.zarr import GDSStore

# 在 Zarr 中启用 GPU 支持
zarr.config.enable_gpu()

# 创建 GDS 支持的存储
store = GDSStore(root="data.zarr")

# 创建并写入 Zarr 数组（数据保留在 GPU）
z = zarr.create_array(
    store=store,
    shape=(1000, 1000),
    chunks=(100, 100),
    dtype="float32",
    overwrite=True,
)

# 读取返回 CuPy 数组
chunk = z[:100, :100]  # 返回 cupy.ndarray
```

Zarr + KvikIO 适用于：
- 气候/气象数据（大型多维数组）
- 生物信息学（基因组数组）
- 任何需要 GPU 处理的块状数组工作负载

需额外安装：`uv add zarr`。

---

## 内存映射文件

`kvikio.mmap.Mmap` 提供内存映射文件访问，支持主机和设备目标：

```python
from kvikio.mmap import Mmap
import cupy as cp

# 映射文件用于读取
with Mmap("data.bin", flags="r") as m:
    print(m.file_size())

    # 顺序读取到设备内存
    buf = cp.empty(1000, dtype=cp.float32)
    m.read(buf, size=4000, offset=0)

    # 并行读取（返回 IOFuture）
    future = m.pread(buf, size=4000, offset=0)
    future.get()
```

---

## 运行时设置

KvikIO 行为通过环境变量或 `kvikio.defaults` API 控制。

### 关键设置

| 设置项 | 环境变量 | 默认值 | 描述 |
|---------|-------------|---------|-------------|
| 兼容模式 | `KVIKIO_COMPAT_MODE` | `AUTO` | `ON`：仅 POSIX，`OFF`：仅 GDS，`AUTO`：尝试 GDS 并回退 |
| 线程池大小 | `KVIKIO_NTHREADS` | 1 | `pread`/`pwrite` 的 I/O 线程数 |
| 任务大小 | `KVIKIO_TASK_SIZE` | 4 MiB | 并行 I/O 任务的最大尺寸 |
| GDS 阈值 | `KVIKIO_GDS_THRESHOLD` | 16 KiB | 使用 GDS 的最小尺寸（更小则用 POSIX） |
| 弹跳缓冲区大小 | `KVIKIO_BOUNCE_BUFFER_SIZE` | 16 MiB | 每个线程的中间主机缓冲区大小 |
| 直接 I/O 读 | `KVIKIO_AUTO_DIRECT_IO_READ` | 关闭 | 为读取启用机会性 O_DIRECT |
| 直接 I/O 写 | `KVIKIO_AUTO_DIRECT_IO_WRITE` | 开启 | 为写入启用机会性 O_DIRECT |

### 编程式配置

```python
import kvikio.defaults

# 查询设置
print(kvikio.defaults.get("compat_mode"))
print(kvikio.defaults.get("num_threads"))

# 运行时修改设置
kvikio.defaults.set({"num_threads": 16, "task_size": 8 * 1024 * 1024})

# 启用读取直接 I/O
kvikio.defaults.set({"auto_direct_io_read": True})
```

### 兼容模式

当 GDS 不可用时（缺少 `libcufile.so`、在 WSL 中运行、Docker 无 `/run/udev`），`AUTO` 模式自动回退到 POSIX I/O。这意味着 KvikIO 代码可在任何环境运行 —— 仅在有 GDS 时更快。

```python
import kvikio.cufile_driver

# 检查是否实际使用 GDS
print(kvikio.cufile_driver.get("is_gds_available"))
```

### cuFile 驱动配置

```python
import kvikio.cufile_driver

# 查询驱动属性
print(kvikio.cufile_driver.get("is_gds_available"))
print(kvikio.cufile_driver.get("major_version"))

# 配置可设置属性
kvikio.cufile_driver.set("max_device_cache_size", 1024)

# 作为上下文管理器使用（退出时自动恢复）
with kvikio.cufile_driver.set({"poll_mode": True}):
    # 此处轮询模式激活
    ...
# 轮询模式已恢复
```

---

## 性能优化

### 1. 增加线程池大小

默认 1 线程较保守。对大文件应增加：

```python
import kvikio.defaults
kvikio.defaults.set({"num_threads": 16})
```

### 2. 使用非阻塞 I/O 实现流水线

通过 `pread`/`pwrite` 重叠 I/O 与计算：

```python
import cupy as cp
import kvikio

# 流水线：读取第 N 块时处理第 N-1 块
chunk_size = 10_000_000
buf_a = cp.empty(chunk_size, dtype=cp.float32)
buf_b = cp.empty(chunk_size, dtype=cp.float32)

with kvikio.CuFile("large_data.bin", "r") as f:
    # 启动首次读取
    future = f.pread(buf_a)
    future.get()

    for offset in range(chunk_size * 4, file_size, chunk_size * 4):
        # 处理当前块时启动下次读取
        next_future = f.pread(buf_b, file_offset=offset)

        # 在 GPU 上处理 buf_a（与 I/O 重叠）
        result = cp.fft.fft(buf_a)

        next_future.get()
        buf_a, buf_b = buf_b, buf_a  # 交换缓冲区
```

### 3. 对齐 I/O 到页面边界

GDS 在 4 KiB 对齐的偏移和尺寸下性能最佳：

```python
# 良好：对齐的偏移和尺寸
f.read(buf, size=4096 * 1000, file_offset=4096 * 10)

# 较慢：未对齐（KvikIO 能处理，但拆分为对齐+未对齐部分）
f.read(buf, size=5000, file_offset=100)
```

### 4. 启用直接 I/O

对于顺序写入和冷读取，直接 I/O（绕过 OS 页缓存）可能有帮助：

```python
import kvikio.defaults
kvikio.defaults.set({
    "auto_direct_io_read": True,
    "auto_direct_io_write": True,

# 数据现已转换为 CuPy 数组，可进行 GPU 计算

### 使用 Numba CUDA

KvikIO 支持任何符合 CUDA 数组接口的缓冲区：

```python
from numba import cuda
import kvikio

d_arr = cuda.device_array(1_000_000, dtype="float32")
with kvikio.CuFile("data.bin", "r") as f:
    f.read(d_arr)
```

### 使用 cuDF

对于非表格格式的原始二进制数据，先用 KvikIO 加载再转换：

```python
import cupy as cp
import cudf
import kvikio

# 加载原始浮点数组，封装为 cuDF Series
buf = cp.empty(1_000_000, dtype=cp.float32)
with kvikio.CuFile("signal.bin", "r") as f:
    f.read(buf)
signal = cudf.Series(buf)
```

对于表格格式（CSV、Parquet、JSON、ORC），请使用 cuDF 原生读取器——它们已针对这些格式优化。

### 使用 NumPy（主机内存）

KvikIO 无缝处理主机内存：

```python
import numpy as np
import kvikio

arr = np.empty(1_000_000, dtype=np.float32)
with kvikio.CuFile("data.bin", "r") as f:
    f.read(arr)
```

---

## 常用模式

### 保存与加载 GPU 模型检查点

```python
import cupy as cp
import kvikio

def save_checkpoint(arrays: dict[str, cp.ndarray], path: str):
    """将多个 GPU 数组保存至单个文件"""
    with kvikio.CuFile(path, "w") as f:
        offset = 0
        for arr in arrays.values():
            f.write(arr, file_offset=offset)
            offset += arr.nbytes

def load_checkpoint(shapes_dtypes: dict, path: str) -> dict[str, cp.ndarray]:
    """从检查点文件加载 GPU 数组"""
    arrays = {}
    with kvikio.CuFile(path, "r") as f:
        offset = 0
        for name, (shape, dtype) in shapes_dtypes.items():
            arr = cp.empty(shape, dtype=dtype)
            f.read(arr, file_offset=offset)
            offset += arr.nbytes
            arrays[name] = arr
    return arrays
```

### 从 S3 流式传输数据至 GPU 处理

```python
import cupy as cp
import kvikio

with kvikio.RemoteFile.open_s3("my-bucket", "large-dataset.bin") as f:
    total_bytes = f.nbytes()
    chunk_size = 100 * 1024 * 1024  # 100 MB 分块
    buf = cp.empty(chunk_size // 4, dtype=cp.float32)

    for offset in range(0, total_bytes, chunk_size):
        size = min(chunk_size, total_bytes - offset)
        f.read(buf[:size // 4], size=size, file_offset=offset)
        # 在 GPU 上处理分块数据
        result = cp.mean(buf[:size // 4])
```

### 替代 Python open() 处理 GPU 工作负载

```python
# 之前：CPU 受限的文件 IO
import numpy as np
data = np.fromfile("data.bin", dtype=np.float32)
import cupy as cp
gpu_data = cp.asarray(data)  # 额外复制：磁盘 → CPU → GPU

# 之后：直连 GPU
import cupy as cp
import kvikio
gpu_data = cp.empty(1_000_000, dtype=cp.float32)
with kvikio.CuFile("data.bin", "r") as f:
    f.read(gpu_data)  # 磁盘 → GPU 直连（通过 GDS）
```

---

## 常见陷阱

1. **忘记设置线程池大小**——默认仅 1 线程。处理大文件时，`kvikio.defaults.set({"num_threads": 16})` 可显著提升吞吐量。

2. **对结构化格式使用 KvikIO**——请勿用 KvikIO 读取 CSV/Parquet/JSON。应使用 `cudf.read_csv()`, `cudf.read_parquet()` 等。KvikIO 仅适用于原始二进制数据。

3. **未检查 GDS 可用性**——无 GDS 时代码仍可运行（回退至 POSIX），但无法获得全带宽优势。通过 `kvikio.cufile_driver.get("is_gds_available")` 检查。

4. **性能关键路径中的未对齐 IO**——使用 4 KiB 对齐的偏移量和大小以获得最佳 GDS 性能。

5. **未使用上下文管理器**——始终通过 `with kvikio.CuFile(...)` 确保文件正确关闭和注销。

6. **尝试 RemoteFile 写入**——`RemoteFile` 为只读。写入远程存储时，请先本地写入再通过对应 SDK 上传（如 S3 使用 boto3）。

7. **未配置 GDS 的 Docker 环境**——在 Docker 中需以只读方式挂载 `/run/udev`（`--volume /run/udev:/run/udev:ro`）才能启用 GDS。否则 KvikIO 将静默回退至 POSIX。
