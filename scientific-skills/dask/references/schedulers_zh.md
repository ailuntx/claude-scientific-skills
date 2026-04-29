# Dask 调度器

## 概述

Dask 提供多种任务调度器，分别适用于不同工作负载。调度器决定任务执行方式：顺序执行、并行线程、并行进程或跨集群分布式执行。

## 调度器类型

### 单机调度器

#### 1. 本地线程调度器（默认）

**描述**：线程调度器使用本地 `concurrent.futures.ThreadPoolExecutor` 执行计算。

**适用场景**：
- NumPy、Pandas、scikit-learn 的数值计算
- 能释放 GIL（全局解释器锁）的库
- 受益于共享内存访问的操作
- Dask 数组和 DataFrame 的默认调度器

**特性**：
- 低开销
- 线程间共享内存
- 最适合释放 GIL 的操作
- 不适用于纯 Python 代码（GIL 争用）

**示例**：
```python
import dask.array as da

# 默认使用线程
x = da.random.random((10000, 10000), chunks=(1000, 1000))
result = x.mean().compute()  # 使用线程计算
```

**显式配置**：
```python
import dask

# 全局设置
dask.config.set(scheduler='threads')

# 或按计算设置
result = x.mean().compute(scheduler='threads')
```

#### 2. 本地进程调度器

**描述**：使用 `concurrent.futures.ProcessPoolExecutor` 的多进程调度器。

**适用场景**：
- 存在 GIL 争用的纯 Python 代码
- 文本处理和 Python 集合操作
- 受益于进程隔离的操作
- CPU 密集型 Python 代码

**特性**：
- 规避 GIL 限制
- 进程间数据传输有成本
- 开销高于线程
- 适合输入/输出量小的线性工作流

**示例**：
```python
import dask.bag as db

# 适合处理 Python 对象
bag = db.read_text('data/*.txt')
result = bag.map(complex_python_function).compute(scheduler='processes')
```

**显式配置**：
```python
import dask

# 全局设置
dask.config.set(scheduler='processes')

# 或按计算设置
result = computation.compute(scheduler='processes')
```

**限制**：
- 数据必须可序列化（pickle）
- 进程创建开销
- 数据复制导致内存开销

#### 3. 单线程调度器（同步）

**描述**：单线程同步调度器在本地线程中顺序执行所有计算，无并行性。

**适用场景**：
- 使用 pdb 调试
- 使用标准 Python 工具分析性能
- 详细理解错误
- 开发和测试

**特性**：
- 无并行性
- 易于调试
- 零开销
- 确定性执行

**示例**：
```python
import dask

# 启用调试模式
dask.config.set(scheduler='synchronous')

# 可使用 pdb
result = computation.compute()  # 在单线程中运行
```

**IPython 调试**：
```python
# 在 IPython/Jupyter 中
%pdb on

dask.config.set(scheduler='synchronous')
result = problematic_computation.compute()  # 出错时进入调试器
```

### 分布式调度器

#### 4. 本地分布式调度器

**描述**：尽管名称如此，该调度器可在个人计算机上使用分布式调度器基础设施高效运行。

**适用场景**：
- 需要诊断仪表盘
- 异步 API
- 比多进程更好的数据局部性处理
- 扩展到集群前的开发阶段
- 在单机上使用分布式特性

**特性**：
- 提供监控仪表盘
- 更优的内存管理
- 比线程/进程开销更大
- 后续可扩展到集群

**示例**：
```python
from dask.distributed import Client
import dask.dataframe as dd

# 创建本地集群
client = Client()  # 自动使用所有核心

# 使用分布式调度器
ddf = dd.read_csv('data.csv')
result = ddf.groupby('category').mean().compute()

# 查看仪表盘
print(client.dashboard_link)

# 清理资源
client.close()
```

**配置选项**：
```python
# 控制资源
client = Client(
    n_workers=4,
    threads_per_worker=2,
    memory_limit='4GB'
)
```

#### 5. 集群分布式调度器

**描述**：用于通过分布式调度器跨多台机器扩展计算。

**适用场景**：
- 数据超过单机容量
- 需要超越单机的计算能力
- 生产环境部署
- 集群计算环境（HPC、云平台）

**特性**：
- 可扩展至数百台机器
- 需要集群设置
- 存在网络通信开销
- 支持高级特性（自适应扩缩容、任务优先级）

**Dask-Jobqueue 示例（HPC）**：
```python
from dask_jobqueue import SLURMCluster
from dask.distributed import Client

# 在 SLURM HPC 上创建集群
cluster = SLURMCluster(
    cores=24,
    memory='100GB',
    walltime='02:00:00',
    queue='regular'
)

# 扩展到 10 个作业
cluster.scale(jobs=10)

# 连接客户端
client = Client(cluster)

# 执行计算
result = computation.compute()

client.close()
```

**Kubernetes 示例**：
```python
from dask_kubernetes import KubeCluster
from dask.distributed import Client

cluster = KubeCluster()
cluster.scale(20)  # 20 个工作节点

client = Client(cluster)
result = computation.compute()

client.close()
```

## 调度器配置

### 全局配置

```python
import dask

# 为会话设置全局调度器
dask.config.set(scheduler='threads')
dask.config.set(scheduler='processes')
dask.config.set(scheduler='synchronous')
```

### 上下文管理器

```python
import dask

# 临时切换调度器
with dask.config.set(scheduler='processes'):
    result = computation.compute()

# 恢复默认调度器
result2 = computation2.compute()
```

### 按计算配置

```python
# 在计算调用中指定调度器
result = computation.compute(scheduler='threads')
result = computation.compute(scheduler='processes')
result = computation.compute(scheduler='synchronous')
```

### 分布式客户端

```python
from dask.distributed import Client

# 使用客户端自动启用分布式调度器
client = Client()

# 所有计算使用分布式调度器
result = computation.compute()

client.close()
```

## 选择合适调度器

### 决策矩阵

| 工作负载类型 | 推荐调度器 | 依据 |
|--------------|----------------------|-----------|
| NumPy/Pandas 操作 | 线程（默认） | 释放 GIL，共享内存 |
| 纯 Python 对象 | 进程 | 避免 GIL 争用 |
| 文本/日志处理 | 进程 | Python 密集型操作 |
| 调试 | 同步 | 易于调试，确定性 |
| 需要仪表盘 | 本地分布式 | 监控和诊断 |
| 多机环境 | 集群分布式 | 超越单机容量 |
| 小数据快速任务 | 线程 | 最低开销 |
| 大数据单机处理 | 本地分布式 | 更优内存管理 |

### 性能考量

**线程调度器**：
- 开销：约 10 µs/任务
- 最佳场景：数值运算
- 内存：共享
- GIL：受影响

**进程调度器**：
- 开销：约 10 ms/任务
- 最佳场景：Python 操作
- 内存：进程间复制
- GIL：不受影响

**同步调度器**：
- 开销：约 1 µs/任务
- 最佳场景：调试
- 内存：无并行
- GIL：不相关

**分布式调度器**：
- 开销：约 1 ms/任务
- 最佳场景：复杂工作流，监控
- 内存：由调度器管理
- GIL：工作节点可使用线程/进程

## 分布式调度器的线程配置

### 设置线程数

```python
from dask.distributed import Client

# 控制工作节点配置
client = Client(
    n_workers=4,           # 工作进程数
    threads_per_worker=2   # 每个进程的线程数
)
```

### 推荐配置

**数值计算工作负载**：
- 每个进程约 4 个线程
- 平衡并行性和开销
- 示例：8 核 → 2 个工作进程 × 4 线程

**Python 工作负载**：
- 使用更多工作进程（单线程）
- 示例：8 核 → 8 个工作进程 × 1 线程

### 环境变量

```bash
# 通过环境变量设置
export DASK_NUM_WORKERS=4
export DASK_THREADS_PER_WORKER=2

# 或通过配置文件设置
```

## 常用模式

### 从开发到生产

```python
# 开发：使用本地分布式测试
from dask.distributed import Client
client = Client(processes=False)  # 进程内调试

# 生产：扩展到集群
from dask.distributed import Client
client = Client('scheduler-address:8786')
```

### 混合工作负载

```python
import dask
import dask.dataframe as dd

# DataFrame 操作使用线程
ddf = dd.read_parquet('data.parquet')
result1 = ddf.mean().compute(scheduler='threads')

# Python 代码使用进程
import dask.bag as db
bag = db.read_text('logs/*.txt')
result2 = bag.map(parse_log).compute(scheduler='processes')
```

### 调试工作流

```python
import dask

# 步骤1：用同步调度器调试
dask.config.set(scheduler='synchronous')
result = problematic_computation.compute()

# 步骤2：用线程测试
dask.config.set(scheduler='threads')
result = computation.compute()

# 步骤3：用分布式扩展
from dask.distributed import Client
client = Client()
result = computation.compute()
```

## 监控与诊断

### 仪表盘访问（仅分布式）

```python
from dask.distributed import Client

client = Client()

# 获取仪表盘链接
print(client.dashboard_link)
# 浏览器中显示：
# - 任务进度
# - 工作节点状态
# - 内存使用
# - 任务流
# - 资源利用率
```

### 性能分析

```python
# 分析计算过程
from dask.distributed import Client

client = Client()
result = computation.compute()

# 获取性能报告
client.profile(filename='profile.html')
```

### 资源监控

```python
# 检查工作节点信息
client.scheduler_info()

# 获取当前任务
client.who_has()

# 内存使用
client.run(lambda: psutil.virtual_memory().percent)
```

## 高级配置

### 自定义执行器

```python
from concurrent.futures import ThreadPoolExecutor
import dask

# 使用自定义线程池
with ThreadPoolExecutor(max_workers=4) as executor:
    dask.config.set(pool=executor)
    result = computation.compute(scheduler='threads')
```

### 自适应扩缩容（分布式）

```python
from dask.distributed import Client

client = Client()

# 启用自适应扩缩容
client.cluster.adapt(minimum=2, maximum=10)

# 集群根据负载自动扩缩
result = computation.compute()
```

### 工作节点插件

```python
from dask.distributed import Client, WorkerPlugin

class CustomPlugin(WorkerPlugin):
    def setup(self, worker):
        # 初始化工作节点专属资源
        worker.custom_resource = initialize_resource()

client = Client()
client.register_worker_plugin(CustomPlugin())
```

## 故障排除

### 线程调度器性能低下
**问题**：纯 Python 代码在线程调度器中运行缓慢  
**解决方案**：切换到进程或分布式调度器

### 进程调度器内存错误
**问题**：数据过大无法在进程间 pickle/复制  
**解决方案**：使用线程或分布式调度器

### 调试困难
**问题**：并行调度器无法使用 pdb  
**解决方案**：使用同步调度器调试

### 任务开销过高
**问题**：大量微任务导致开销  
**解决方案**：使用线程调度器（最低开销）或增大分块大小
