---
name: dask
description: 用于处理超出内存容量的pandas/NumPy工作流的分布式计算。当需要将现有pandas/NumPy代码扩展到内存限制之外或跨集群运行时使用。最适合并行文件处理、分布式机器学习、与现有pandas代码集成。单机内存外分析请使用vaex；追求内存计算速度请使用polars。
license: BSD-3-Clause license
metadata:
    skill-author: K-Dense Inc.
---

# Dask

## 概述

Dask是一个用于并行和分布式计算的Python库，提供三大核心能力：
- **超内存执行**：在单机上处理超出可用RAM的数据
- **并行处理**：跨多核提升计算速度
- **分布式计算**：支持跨多台机器处理TB级数据集

Dask可从小型笔记本（约100 GiB）扩展到集群（约100 TiB），同时保持熟悉的Python API接口。

## 适用场景

在以下情况应使用本技能：
- 处理超出可用内存的数据集
- 将pandas或NumPy操作扩展到更大数据集
- 通过并行化提升计算性能
- 高效处理多文件（CSV、Parquet、JSON、文本日志）
- 构建带任务依赖的自定义并行工作流
- 在多核或多机间分配工作负载

## 核心能力

Dask提供五大组件，分别适用于不同场景：

### 1. 数据框 - 并行化Pandas操作

**用途**：通过并行处理将pandas操作扩展到更大数据集。

**适用场景**：
- 表格数据超出可用内存
- 需同时处理多个CSV/Parquet文件
- pandas操作缓慢需并行化
- 从pandas原型扩展到生产环境

**参考文档**：完整Dask数据框指南请参阅`references/dataframes.md`，包含：
- 数据读取（单文件、多文件、通配符匹配）
- 常用操作（过滤、分组、连接、聚合）
- 使用`map_partitions`的自定义操作
- 性能优化技巧
- 常用模式（ETL、时间序列、多文件处理）

**快速示例**：
```python
import dask.dataframe as dd

# 将多文件读取为单一数据框
ddf = dd.read_csv('data/2024-*.csv')

# 操作延迟执行直到compute()
filtered = ddf[ddf['value'] > 100]
result = filtered.groupby('category').mean().compute()
```

**关键要点**：
- 操作延迟构建任务图直到调用`.compute()`
- 使用`map_partitions`实现高效自定义操作
- 处理结构化数据时尽早转换为DataFrame

### 2. 数组 - 并行化NumPy操作

**用途**：通过分块算法将NumPy能力扩展到超内存数据集。

**适用场景**：
- 数组超出可用内存
- NumPy操作需并行化
- 处理科学数据集（HDF5、Zarr、NetCDF）
- 需并行线性代数或数组运算

**参考文档**：完整Dask数组指南请参阅`references/arrays.md`，包含：
- 数组创建（从NumPy、随机生成、磁盘加载）
- 分块策略与优化
- 常用操作（算术、归约、线性代数）
- 使用`map_blocks`的自定义操作
- 与HDF5、Zarr和XArray的集成

**快速示例**：
```python
import dask.array as da

# 创建带分块的大数组
x = da.random.random((100000, 100000), chunks=(10000, 10000))

# 操作延迟执行
y = x + 100
z = y.mean(axis=0)

# 计算结果
result = z.compute()
```

**关键要点**：
- 分块大小至关重要（目标约100MB/块）
- 操作在分块上并行执行
- 必要时重新分块以提升效率
- 使用`map_blocks`处理Dask未内置的操作

### 3. 包 - 非结构化数据并行处理

**用途**：通过函数式操作处理非结构化/半结构化数据（文本、JSON、日志）。

**适用场景**：
- 处理文本文件、日志或JSON记录
- 结构化分析前的数据清洗与ETL
- 处理不适合数组/数据框格式的Python对象
- 需内存高效的流式处理

**参考文档**：完整Dask包指南请参阅`references/bags.md`，包含：
- 文本与JSON文件读取
- 函数式操作（映射、过滤、折叠、分组）
- 转换为数据框
- 常用模式（日志分析、JSON处理、文本处理）
- 性能考量

**快速示例**：
```python
import dask.bag as db
import json

# 读取并解析JSON文件
bag = db.read_text('logs/*.json').map(json.loads)

# 过滤与转换
valid = bag.filter(lambda x: x['status'] == 'valid')
processed = valid.map(lambda x: {'id': x['id'], 'value': x['value']})

# 转换为数据框进行分析
ddf = processed.to_dataframe()
```

**关键要点**：
- 用于初始数据清洗，再转为数据框/数组
- 使用`foldby`替代`groupby`提升性能
- 操作支持流式处理且内存高效
- 复杂操作应转为结构化格式（数据框）

### 4. 期货 - 基于任务的并行化

**用途**：通过细粒度任务执行控制构建自定义并行工作流。

**适用场景**：
- 构建动态演进的工作流
- 需即时任务执行（非延迟）
- 计算依赖运行时条件
- 实现自定义并行算法
- 需有状态计算

**参考文档**：完整Dask期货指南请参阅`references/futures.md`，包含：
- 分布式客户端设置
- 任务提交与期货操作
- 任务依赖与数据移动
- 高级协调（队列、锁、事件、执行器）
- 常用模式（参数扫描、动态任务、迭代算法）

**快速示例**：
```python
from dask.distributed import Client

client = Client()  # 创建本地集群

# 提交任务（立即执行）
def process(x):
    return x ** 2

futures = client.map(process, range(100))

# 收集结果
results = client.gather(futures)

client.close()
```

**关键要点**：
- 需分布式客户端（单机也适用）
- 任务提交后立即执行
- 预先分发大数据避免重复传输
- 每任务约1ms开销（不适用百万级微任务）
- 有状态工作流使用执行器

### 5. 调度器 - 执行后端

**用途**：控制Dask任务执行方式与位置（线程、进程、分布式）。

**调度器选择场景**：
- **线程**（默认）：NumPy/Pandas操作、释放GIL的库、共享内存优势
- **进程**：纯Python代码、文本处理、GIL限制操作
- **同步**：使用pdb调试、性能剖析、错误诊断
- **分布式**：需监控面板、多机集群、高级功能

**参考文档**：完整调度器指南请参阅`references/schedulers.md`，包含：
- 调度器特性详解
- 配置方法（全局、上下文管理器、按计算指定）
- 性能考量与开销
- 常用模式与故障排除
- 线程优化配置

**快速示例**：
```python
import dask
import dask.dataframe as dd

# 数据框使用线程（默认，适合数值计算）
ddf = dd.read_csv('data.csv')
result1 = ddf.mean().compute()  # 使用线程

# Python密集型工作使用进程
import dask.bag as db
bag = db.read_text('logs/*.txt')
result2 = bag.map(python_function).compute(scheduler='processes')

# 调试使用同步模式
dask.config.set(scheduler='synchronous')
result3 = problematic_computation.compute()  # 可使用pdb

# 监控与扩展使用分布式
from dask.distributed import Client
client = Client()
result4 = computation.compute()  # 使用带监控面板的分布式
```

**关键要点**：
- 线程：最低开销（约10µs/任务），最适合数值计算
- 进程：规避GIL（约10ms/任务），最适合Python计算
- 分布式：带监控面板（约1ms/任务），可扩展至集群
- 可按计算或全局切换调度器

## 最佳实践

完整性能优化指南、内存管理策略及常见陷阱请参阅`references/best-practices.md`。核心原则包括：

### 优先简单方案
使用Dask前考虑：
- 更优算法
- 高效文件格式（用Parquet替代CSV）
- 编译代码（Numba、Cython）
- 数据采样

### 关键性能准则

**1. 避免本地加载后转交Dask**
```python
# 错误：先全量加载到内存
import pandas as pd
df = pd.read_csv('large.csv')
ddf = dd.from_pandas(df, npartitions=10)

# 正确：由Dask直接加载
import dask.dataframe as dd
ddf = dd.read_csv('large.csv')
```

**2. 避免重复compute()调用**
```python
# 错误：每次compute独立执行
for item in items:
    result = dask_computation(item).compute()

# 正确：单次compute批量执行
computations = [dask_computation(item) for item in items]
results = dask.compute(*computations)
```

**3. 避免构建超大任务图**
- 任务超百万时增大分块
- 使用`map_partitions`/`map_blocks`融合操作
- 检查任务图大小：`len(ddf.__dask_graph__())`

**4. 选择合适分块大小**
- 目标：约100MB/块（或工作内存中每核10块）
- 过大：内存溢出
- 过小：调度开销

**5. 使用监控面板**
```python
from dask.distributed import Client
client = Client()
print(client.dashboard_link)  # 监控性能，定位瓶颈
```

## 常用工作流模式

### ETL流程
```python
import dask.dataframe as dd

# 抽取：读取数据
ddf = dd.read_csv('raw_data/*.csv')

# 转换：清洗处理
ddf = ddf[ddf['status'] == 'valid']
ddf['amount'] = ddf['amount'].astype('float64')
ddf = ddf.dropna(subset=['important_col'])

# 加载：聚合存储
summary = ddf.groupby('category').agg({'amount': ['sum', 'mean']})
summary.to_parquet('output/summary.parquet')
```

### 非结构化转结构化流程
```python
import dask.bag as db
import json

# 从包开始处理非结构化数据
bag = db.read_text('logs/*.json').map(json.loads)
bag = bag.filter(lambda x: x['status'] == 'valid')

# 转为数据框进行结构化分析
ddf = bag.to_dataframe()
result = ddf.groupby('category').mean().compute()
```

### 大规模数组计算
```python
import dask.array as da

# 加载或创建大数组
x = da.from_zarr('large_dataset.zarr')

# 分块处理
normalized = (x - x.mean()) / x.std()

# 保存结果
da.to_zarr(normalized, 'normalized.zarr')
```

### 自定义并行工作流
```python
from dask.distributed import Client

client = Client()

# 一次性分发大数据集
data = client.scatter(large_dataset)

# 带依赖的并行处理
futures = []
for param in parameters:
    future = client.submit(process, data, param)
    futures.append(future)

# 汇总结果
results = client.gather(futures)
```

## 组件选择指南

根据数据类型选择Dask组件：

**数据类型**：
- 表格数据 → **数据框**
- 数值数组 → **数组**
- 文本/JSON/日志 → **包**（再转数据框）
- 自定义Python对象 → **包**或**期货**

**操作类型**：
- 标准pandas操作 → **数据框**
- 标准NumPy操作 → **数组**
- 自定义并行任务 → **期货**
- 文本处理/ETL → **包**

**控制层级**：
- 高层自动 → **数据框/数组**
- 底层手动 → **期货**

**工作流类型**：
- 静态计算图 → **数据框/数组/包**
- 动态演进 → **期货**

## 集成考量

### 文件格式
- **高效格式**：Parquet、HDF5、Zarr（列式存储、压缩、并行友好）
- **兼容但低效**：CSV（仅用于初始摄取）
- **数组专用**：HDF5、Zarr、NetCDF

### 集合间转换
```python
# 包 → 数据框
ddf = bag.to_dataframe()

# 数据框 → 数组（数值数据）
arr = ddf.to_dask_array(lengths=True)

# 数组 → 数据框
ddf = dd.from_dask_array(arr, columns=['col1', 'col2'])
```

### 与其他库集成
- **XArray**：为Dask数组添加维度标签（地理空间、影像）
- **Dask-ML**：兼容scikit-learn API的机器学习
- **Distributed**：高级集群管理与监控

## 调试与开发

### 迭代开发流程

1. **同步调度器小数据测试**：
```python
dask.config.set(scheduler='synchronous')
result = computation.compute()  # 可使用pdb调试
```

2. **样本数据线程验证**：
```python
sample = ddf.head(1000)  # 小样本
# 验证逻辑后扩展到全量
```

3. **分布式监控扩展**：
```python
from dask.distributed import Client
client = Client()
print(client.dashboard_link)  # 监控性能
result = computation.compute()
```

### 常见问题

**内存错误**：
- 减小分块大小
- 策略性使用`persist()`并及时删除
- 检查自定义函数内存泄漏

**启动缓慢**：
- 任务图过大（增大分块）
- 使用`map_partitions`或`map_blocks`减少任务数

**并行效率低**：
- 分块过大（增加分区数）
- Python代码误用线程（切换进程）
- 数据依赖阻碍并行化

## 参考文件

所有参考文档可按需查阅：

- `references/dataframes.md` - Dask数据框完整指南
- `references/arrays.md` - Dask数组完整指南
- `references/bags.md` - Dask包完整指南
- `references/futures.md` - Dask期货与分布式计算完整指南
- `references/schedulers.md` - 调度器选择与配置完整指南
- `references/best-practices.md` - 综合性能优化与故障排除

当用户需要超出本文快速指南的详细组件说明、操作指南或模式解析时，请加载对应参考文件。
