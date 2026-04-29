# Dask 最佳实践

## 性能优化原则

### 优先采用简单方案

在实施 Dask 并行计算前，请先尝试以下替代方案：
- 针对特定问题的更优算法
- 高效文件格式（用 Parquet、HDF5、Zarr 替代 CSV）
- 通过 Numba 或 Cython 编译代码
- 开发测试阶段采用数据采样

这些方案通常比分布式系统收益更高，应在扩展并行计算前充分尝试。

### 分块大小策略

**核心原则**：分块应足够小，确保单个工作节点内存可同时容纳多个分块。

**推荐目标**：按每个核心可处理 10 个分块且不超出内存的标准设置分块大小。

**重要性**：
- 分块过大：导致内存溢出且并行效率低下
- 分块过小：产生过量调度开销

**计算示例**：
- 8 核 CPU 配 32 GB 内存
- 目标分块大小：约 400 MB（32 GB / 8 核 / 10 分块）

### 利用仪表板监控

Dask 仪表板提供关键运行洞察：
- 工作节点状态与资源利用率
- 任务进度与瓶颈点
- 内存使用模式
- 性能特征

通过仪表板可精准定位并行工作负载中的性能瓶颈，避免盲目优化。

## 关键陷阱规避

### 1. 避免在 Dask 前创建大型本地对象

**错误做法**：
```python
import pandas as pd
import dask.dataframe as dd

# 先将整个数据集载入内存
df = pd.read_csv('large_file.csv')
ddf = dd.from_pandas(df, npartitions=10)
```

**正确做法**：
```python
import dask.dataframe as dd

# 让 Dask 控制加载过程
ddf = dd.read_csv('large_file.csv')
```

**原因**：先用 pandas 或 NumPy 加载数据会迫使调度器序列化对象并嵌入任务图，违背并行计算初衷。

**核心原则**：始终使用 Dask 原生方法加载数据并控制结果。

### 2. 避免重复调用 compute()

**错误做法**：
```python
results = []
for item in items:
    result = dask_computation(item).compute()  # 每次单独计算
    results.append(result)
```

**正确做法**：
```python
computations = [dask_computation(item) for item in items]
results = dask.compute(*computations)  # 单次批量计算
```

**原因**：循环调用 compute 会阻止 Dask：
- 并行执行不同计算
- 共享中间结果
- 优化整体任务图

### 3. 避免构建过大规模任务图

**典型症状**：
- 单次计算包含数百万任务
- 严重调度开销
- 计算启动前长时间延迟

**解决方案**：
- 增大分块尺寸以减少任务数
- 使用 `map_partitions` 或 `map_blocks` 融合操作
- 通过中间持久化拆分计算过程
- 评估问题是否真正需要分布式计算

**map_partitions 应用示例**：
```python
# 避免逐行应用函数
ddf['result'] = ddf.apply(complex_function, axis=1)  # 产生大量任务

# 对整个分块批量处理
ddf = ddf.map_partitions(lambda df: df.assign(result=complex_function(df)))
```

## 基础设施考量

### 调度器选择

**适用线程的场景**：
- 使用释放 GIL 的数值计算库（NumPy、Pandas、scikit-learn）
- 受益于共享内存的操作
- 单机数组/数据框运算

**适用进程的场景**：
- 文本处理与 Python 集合操作
- 受 GIL 限制的纯 Python 代码
- 需要进程隔离的操作

**适用分布式调度器的场景**：
- 多机集群环境
- 需要诊断仪表板
- 异步 API 需求
- 需优化数据本地性处理

### 线程配置

**推荐方案**：数值计算负载建议每个进程配置约 4 线程。

**依据**：
- 平衡并行性与开销
- 高效利用 CPU 核心
- 降低上下文切换成本

### 内存管理

**策略性持久化**：
```python
# 持久化需复用的中间结果
intermediate = expensive_computation(data).persist()
result1 = intermediate.operation1().compute()
result2 = intermediate.operation2().compute()
```

**及时清理内存**：
```python
# 显式删除大型对象
del intermediate
```

## 数据加载最佳实践

### 选用合适文件格式

**表格数据**：
- Parquet：列式存储、压缩、快速过滤
- CSV：仅适用于小数据或初始导入

**数组数据**：
- HDF5：适用于数值数组
- Zarr：云原生、并行友好
- NetCDF：带元数据的科学数据

### 优化数据摄取

**高效读取多文件**：
```python
# 使用通配符并行读取多个文件
ddf = dd.read_parquet('data/year=2024/month=*/day=*.parquet')
```

**提前指定必要列**：
```python
# 仅读取所需列
ddf = dd.read_parquet('data.parquet', columns=['col1', 'col2', 'col3'])
```

## 常见模式与解决方案

### 模式：高度并行问题

独立计算任务建议使用 Futures：
```python
from dask.distributed import Client

client = Client()
futures = [client.submit(func, arg) for arg in args]
results = client.gather(futures)
```

### 模式：数据预处理流水线

使用 Bags 进行初始 ETL，再转为结构化格式：
```python
import dask.bag as db

# 处理原始 JSON
bag = db.read_text('logs/*.json').map(json.loads)
bag = bag.filter(lambda x: x['status'] == 'success')

# 转为 DataFrame 进行分析
ddf = bag.to_dataframe()
```

### 模式：迭代算法

在迭代间持久化数据：
```python
data = dd.read_parquet('data.parquet')
data = data.persist()  # 跨迭代保留内存

for iteration in range(num_iterations):
    data = update_function(data)
    data = data.persist()  # 持久化更新版本
```

## 调试技巧

### 使用单线程调度器

配合 pdb 或详细错误检查：
```python
import dask

dask.config.set(scheduler='synchronous')
result = computation.compute()  # 单线程运行便于调试
```

### 检查任务图规模

计算前验证任务数量：
```python
print(len(ddf.__dask_graph__()))  # 应保持合理规模（非百万级）
```

### 小规模数据验证

扩展前在小数据集测试逻辑：
```python
# 在首个分块测试
sample = ddf.head(1000)
# 验证结果
# 再扩展到完整数据集
```

## 性能故障排除

### 症状：计算启动缓慢

**可能原因**：任务图规模过大
**解决方案**：增大分块尺寸或使用 map_partitions

### 症状：内存错误

**可能原因**：
- 分块过大
- 中间结果过多
- 用户函数内存泄漏

**解决方案**：
- 减小分块尺寸
- 策略性使用 persist() 并及时清理
- 分析用户函数内存问题

### 症状：并行效率低下

**可能原因**：
- 数据依赖阻碍并行
- 分块过大（任务数不足）
- 线程模式中 Python 代码的 GIL 争用

**解决方案**：
- 重构计算逻辑减少依赖
- 增加分区数量
- 对 Python 代码切换多进程调度器
