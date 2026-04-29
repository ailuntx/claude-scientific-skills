---
name: vaex
description: 当处理和分析超出可用内存的大型表格数据集（数十亿行）时使用此技能。Vaex擅长核外DataFrame操作、惰性求值、快速聚合、大数据高效可视化以及大型数据集上的机器学习。适用于用户需要处理大型CSV/HDF5/Arrow/Parquet文件、对海量数据集执行快速统计、创建大数据可视化或构建内存无法容纳的机器学习流水线场景。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# Vaex

## 概述

Vaex是一个高性能Python库，专为惰性核外DataFrame设计，用于处理和可视化因过大而无法载入内存的表格数据集。Vaex每秒可处理超过十亿行数据，支持对数十亿行数据集进行交互式数据探索与分析。

## 适用场景

在以下场景使用Vaex：
- 处理超过可用内存的表格数据集（GB至TB级）
- 对海量数据集执行快速统计聚合
- 创建大型数据集的可视化图表和热力图
- 构建大数据的机器学习流水线
- 转换数据格式（CSV、HDF5、Arrow、Parquet）
- 需要惰性求值和虚拟列以避免内存开销
- 处理天文数据、金融时间序列或其他大规模科学数据集

## 核心能力

Vaex提供六大核心能力领域，详细文档见references目录：

### 1. DataFrame与数据加载

从多种来源加载和创建Vaex DataFrame，包括文件（HDF5、CSV、Arrow、Parquet）、pandas DataFrame、NumPy数组和字典。参考`references/core_dataframes.md`了解：
- 高效打开大型文件
- 从pandas/NumPy/Arrow转换
- 使用示例数据集
- 理解DataFrame结构

### 2. 数据处理与操作

执行过滤、创建虚拟列、使用表达式以及聚合数据，无需将所有数据载入内存。参考`references/data_processing.md`了解：
- 过滤与选择
- 虚拟列与表达式
- 分组操作与聚合
- 字符串操作与日期时间处理
- 缺失数据处理

### 3. 性能与优化

利用Vaex的惰性求值、缓存策略和内存高效操作。参考`references/performance.md`了解：
- 理解惰性求值
- 使用`delay=True`进行批处理操作
- 按需物化列
- 缓存策略
- 异步操作

### 4. 数据可视化

创建大型数据集的交互式可视化，包括热力图、直方图和散点图。参考`references/visualization.md`了解：
- 创建一维和二维图表
- 热力图可视化
- 使用选择集
- 自定义图表与子图

### 5. 机器学习集成

构建包含转换器、编码器的ML流水线，并与scikit-learn、XGBoost等框架集成。参考`references/machine_learning.md`了解：
- 特征缩放与编码
- PCA与降维
- K均值聚类
- 与scikit-learn/XGBoost/CatBoost集成
- 模型序列化与部署

### 6. I/O操作

以最优性能高效读写多种格式数据。参考`references/io_operations.md`了解：
- 文件格式推荐
- 导出策略
- Apache Arrow操作
- 大型CSV文件处理
- 服务器与远程数据访问

## 快速入门模式

大多数Vaex任务遵循以下模式：

```python
import vaex

# 1. 打开或创建DataFrame
df = vaex.open('large_file.hdf5')  # 或.csv/.arrow/.parquet
# 或
df = vaex.from_pandas(pandas_df)

# 2. 探索数据
print(df)  # 显示首尾行及列信息
df.describe()  # 统计摘要

# 3. 创建虚拟列（无内存开销）
df['new_column'] = df.x ** 2 + df.y

# 4. 使用选择集过滤
df_filtered = df[df.age > 25]

# 5. 计算统计量（快速惰性求值）
mean_val = df.x.mean()
stats = df.groupby('category').agg({'value': 'sum'})

# 6. 可视化
df.plot1d(df.x, limits=[0, 100])
df.plot(df.x, df.y, limits='99.7%')

# 7. 按需导出
df.export_hdf5('output.hdf5')
```

## 参考文档使用

参考文件包含各能力领域的详细信息。根据具体任务加载对应文档：

- **基础操作**：从`references/core_dataframes.md`和`references/data_processing.md`开始
- **性能问题**：查阅`references/performance.md`
- **可视化任务**：使用`references/visualization.md`
- **ML流水线**：参考`references/machine_learning.md`
- **文件I/O**：咨询`references/io_operations.md`

## 最佳实践

1. **使用HDF5或Apache Arrow格式**实现大型数据集最优性能
2. **利用虚拟列**替代物化数据以节省内存
3. **使用`delay=True`批处理操作**执行多重计算
4. **导出为高效格式**而非保留CSV数据
5. **使用表达式**执行复杂计算无需中间存储
6. **通过`df.stat()`分析**了解内存使用并优化操作

## 常用模式

### 模式：大型CSV转HDF5
```python
import vaex

# 打开大型CSV（自动分块处理）
df = vaex.from_csv('large_file.csv')

# 导出为HDF5以便快速后续访问
df.export_hdf5('large_file.hdf5')

# 后续加载即时完成
df = vaex.open('large_file.hdf5')
```

### 模式：高效聚合
```python
# 使用delay=True批处理多个操作
mean_x = df.x.mean(delay=True)
std_y = df.y.std(delay=True)
sum_z = df.z.sum(delay=True)

# 一次性执行所有操作
results = vaex.execute([mean_x, std_y, sum_z])
```

### 模式：特征工程虚拟列
```python
# 无内存开销 - 动态计算
df['age_squared'] = df.age ** 2
df['full_name'] = df.first_name + ' ' + df.last_name
df['is_adult'] = df.age >= 18
```

## 资源

本技能包含`references/`目录下的参考文档：

- `core_dataframes.md` - DataFrame创建、加载与基础结构
- `data_processing.md` - 过滤、表达式、聚合与转换
- `performance.md` - 优化策略与惰性求值
- `visualization.md` - 绘图与交互式可视化
- `machine_learning.md` - ML流水线与模型集成
- `io_operations.md` - 文件格式与数据导入导出
