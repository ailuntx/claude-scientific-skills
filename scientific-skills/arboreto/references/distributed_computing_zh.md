# 使用 Arboreto 进行分布式计算

Arboreto 利用 Dask 实现并行计算，支持从单机多核处理到多节点集群环境的高效 GRN 推断。

## 计算架构

GRN 推断天然具备可并行性：
- 每个目标基因的回归模型可独立训练
- Arboreto 将计算表示为 Dask 任务图
- 任务被分配到可用计算资源上执行

## 本地多核处理（默认模式）

默认情况下，arboreto 使用本地机器的所有可用 CPU 核心：

```python
from arboreto.algo import grnboost2

# 自动使用所有本地核心
network = grnboost2(expression_data=expression_matrix, tf_names=tf_names)
```

此模式满足大多数需求且无需额外配置。

## 自定义本地 Dask 客户端

如需精细控制本地资源，可创建自定义 Dask 客户端：

```python
from distributed import LocalCluster, Client
from arboreto.algo import grnboost2

if __name__ == '__main__':
    # 配置本地集群
    local_cluster = LocalCluster(
        n_workers=10,              # 工作进程数
        threads_per_worker=1,       # 每个工作进程的线程数
        memory_limit='8GB'          # 每个工作进程的内存限制
    )

    # 创建客户端
    custom_client = Client(local_cluster)

    # 使用自定义客户端运行推断
    network = grnboost2(
        expression_data=expression_matrix,
        tf_names=tf_names,
        client_or_address=custom_client
    )

    # 清理资源
    custom_client.close()
    local_cluster.close()
```

### 自定义客户端的优势
- **资源控制**：限制 CPU 和内存使用
- **多次运行**：复用同一客户端进行不同参数集的推断
- **监控**：访问 Dask 仪表板获取性能洞察

## 使用同一客户端进行多次推断运行

复用单个 Dask 客户端执行不同参数的多次推断：

```python
from distributed import LocalCluster, Client
from arboreto.algo import grnboost2

if __name__ == '__main__':
    # 一次性初始化客户端
    local_cluster = LocalCluster(n_workers=8, threads_per_worker=1)
    client = Client(local_cluster)

    # 运行多次推断
    network_seed1 = grnboost2(
        expression_data=expression_matrix,
        tf_names=tf_names,
        client_or_address=client,
        seed=666
    )

    network_seed2 = grnboost2(
        expression_data=expression_matrix,
        tf_names=tf_names,
        client_or_address=client,
        seed=777
    )

    # 使用同一客户端运行不同算法
    from arboreto.algo import genie3
    network_genie3 = genie3(
        expression_data=expression_matrix,
        tf_names=tf_names,
        client_or_address=client
    )

    # 一次性清理资源
    client.close()
    local_cluster.close()
```

## 分布式集群计算

对于超大规模数据集，可连接远程 Dask 分布式调度器（运行于集群）：

### 步骤 1：设置 Dask 调度器（在集群头节点）
```bash
dask-scheduler
# 输出：Scheduler at tcp://10.118.224.134:8786
```

### 步骤 2：启动 Dask 工作节点（在集群计算节点）
```bash
dask-worker tcp://10.118.224.134:8786
```

### 步骤 3：从客户端连接
```python
from distributed import Client
from arboreto.algo import grnboost2

if __name__ == '__main__':
    # 连接远程调度器
    scheduler_address = 'tcp://10.118.224.134:8786'
    cluster_client = Client(scheduler_address)

    # 在集群上运行推断
    network = grnboost2(
        expression_data=expression_matrix,
        tf_names=tf_names,
        client_or_address=cluster_client
    )

    cluster_client.close()
```

### 集群配置最佳实践

**工作节点配置**：
```bash
dask-worker tcp://scheduler:8786 \
    --nprocs 4 \              # 每个节点的进程数
    --nthreads 1 \            # 每个进程的线程数
    --memory-limit 16GB       # 每个进程的内存限制
```

**大规模推断建议**：
- 采用更多中等内存工作节点，而非少量大内存节点
- 设置 `threads_per_worker=1` 避免 scikit-learn 的 GIL 争用
- 监控内存使用以防工作节点被终止

## 监控与调试

### Dask 仪表板

通过 Dask 仪表板进行实时监控：

```python
from distributed import Client

client = Client()  # 打印仪表板 URL
# 仪表板地址：http://localhost:8787/status
```

仪表板显示：
- **任务进度**：已完成/待处理任务数量
- **资源使用**：各工作节点的 CPU、内存情况
- **任务流**：计算过程的实时可视化
- **性能**：瓶颈定位分析

### 详细输出

启用详细日志记录以跟踪推断进度：

```python
network = grnboost2(
    expression_data=expression_matrix,
    tf_names=tf_names,
    verbose=True
)
```

## 性能优化技巧

### 1. 数据格式
- **优先使用 Pandas DataFrame**：相比 NumPy 在 Dask 操作中更高效
- **缩减数据规模**：推断前过滤低方差基因

### 2. 工作节点配置
- **CPU 密集型任务**：设置 `threads_per_worker=1`，增加 `n_workers`
- **内存密集型任务**：提高每个工作节点的 `memory_limit`

### 3. 集群设置
- **网络**：确保节点间高带宽低延迟连接
- **存储**：大型数据集使用共享文件系统或对象存储
- **调度**：分配专用节点避免资源争用

### 4. 转录因子过滤
- **限制 TF 列表**：提供特定 TF 名称可减少计算量
```python
# 全搜索（较慢）
network = grnboost2(expression_data=matrix)

# 过滤搜索（较快）
network = grnboost2(expression_data=matrix, tf_names=known_tfs)
```

## 示例：大规模单细胞分析

在集群上处理单细胞 RNA-seq 数据的完整工作流：

```python
from distributed import Client
from arboreto.algo import grnboost2
import pandas as pd

if __name__ == '__main__':
    # 连接到集群
    client = Client('tcp://cluster-scheduler:8786')

    # 加载大型单细胞数据集（50,000 个细胞 × 20,000 个基因）
    expression_data = pd.read_csv('scrnaseq_data.tsv', sep='\t')

    # 加载细胞类型特异性转录因子
    tf_names = pd.read_csv('tf_list.txt', header=None)[0].tolist()

    # 运行分布式推断
    network = grnboost2(
        expression_data=expression_data,
        tf_names=tf_names,
        client_or_address=client,
        verbose=True,
        seed=42
    )

    # 保存结果
    network.to_csv('grn_results.tsv', sep='\t', index=False)

    client.close()
```

此方法使得分析单机无法处理的大规模数据集成为可能。
