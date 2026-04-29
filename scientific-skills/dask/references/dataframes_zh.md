# Dask 数据框

## 概述

Dask 数据框通过将工作分配到多个 pandas 数据框上，实现对大型表格数据的并行处理。如文档所述："Dask 数据框是多个 pandas 数据框的集合"，具有相同的 API，使得从 pandas 迁移变得简单直接。

## 核心概念

Dask 数据框沿索引被分割为多个 pandas 数据框（分区）：
- 每个分区都是常规的 pandas 数据框
- 操作并行应用于每个分区
- 结果自动合并

## 关键能力

### 扩展性
- 在笔记本电脑上处理 100 GiB 数据
- 在集群上处理 100 TiB 数据
- 处理超出可用内存的数据集

### 兼容性
- 实现大部分 pandas API
- 轻松从 pandas 代码迁移
- 支持熟悉的操作方式

## 使用场景

**适用 Dask 的情况**：
- 数据集超出可用内存容量
- 计算耗时且 pandas 优化无效时
- 需要从原型（pandas）扩展到生产环境（更大数据）
- 需同时处理多个关联文件

**建议继续使用 pandas 的情况**：
- 数据可完全放入内存
- 计算在亚秒级完成
- 无需自定义 `.apply()` 的简单操作
- 迭代开发和探索阶段

## 数据读取

Dask 沿用 pandas 的读取语法，并增加多文件支持：

### 单文件读取
```python
import dask.dataframe as dd

# 读取单文件
ddf = dd.read_csv('data.csv')
ddf = dd.read_parquet('data.parquet')
```

### 多文件读取
```python
# 使用通配符读取多文件
ddf = dd.read_csv('data/*.csv')
ddf = dd.read_parquet('s3://mybucket/data/*.parquet')

# 按路径结构读取
ddf = dd.read_parquet('data/year=*/month=*/day=*.parquet')
```

### 优化技巧
```python
# 指定读取列（减少内存）
ddf = dd.read_parquet('data.parquet', columns=['col1', 'col2'])

# 控制分区大小
ddf = dd.read_csv('data.csv', blocksize='64MB')  # 创建64MB分区
```

## 常用操作

所有操作在调用 `.compute()` 前均为惰性执行。

### 数据筛选
```python
# 与pandas语法相同
filtered = ddf[ddf['column'] > 100]
filtered = ddf.query('column > 100')
```

### 列操作
```python
# 添加新列
ddf['new_column'] = ddf['col1'] + ddf['col2']

# 选择列子集
subset = ddf[['col1', 'col2', 'col3']]

# 删除列
ddf = ddf.drop(columns=['unnecessary_col'])
```

### 聚合计算
```python
# 标准聚合操作
mean = ddf['column'].mean().compute()
sum_total = ddf['column'].sum().compute()
counts = ddf['category'].value_counts().compute()
```

### 分组操作
```python
# GroupBy操作（可能需要数据重排）
grouped = ddf.groupby('category')['value'].mean().compute()

# 多重聚合
agg_result = ddf.groupby('category').agg({
    'value': ['mean', 'sum', 'count'],
    'amount': 'sum'
}).compute()
```

### 连接与合并
```python
# 合并数据框
merged = dd.merge(ddf1, ddf2, on='key', how='left')

# 索引连接
joined = ddf1.join(ddf2, on='key')
```

### 数据排序
```python
# 排序（昂贵操作，需数据移动）
sorted_ddf = ddf.sort_values('column')
result = sorted_ddf.compute()
```

## 自定义操作

### 应用函数

**分区级应用（高效）**：
```python
# 对整个分区应用函数
def custom_partition_function(partition_df):
    # partition_df是pandas数据框
    return partition_df.assign(new_col=partition_df['col1'] * 2)

ddf = ddf.map_partitions(custom_partition_function)
```

**行级应用（低效）**：
```python
# 逐行应用（创建大量任务）
ddf['result'] = ddf.apply(lambda row: custom_function(row), axis=1, meta=('result', 'float'))
```

**注意**：优先使用 `map_partitions` 替代逐行 `apply` 以获得更好性能。

### Meta 参数

当 Dask 无法推断输出结构时，需指定 `meta` 参数：
```python
# apply操作示例
ddf['new'] = ddf.apply(func, axis=1, meta=('new', 'float64'))

# map_partitions示例
ddf = ddf.map_partitions(func, meta=pd.DataFrame({
    'col1': pd.Series(dtype='float64'),
    'col2': pd.Series(dtype='int64')
}))
```

## 惰性求值与计算触发

### 惰性操作
```python
# 以下操作均为惰性（瞬时完成，无实际计算）
filtered = ddf[ddf['value'] > 100]
aggregated = filtered.groupby('category').mean()
final = aggregated[aggregated['value'] < 500]

# 此时尚未执行计算
```

### 触发计算
```python
# 计算单个结果
result = final.compute()

# 高效计算多个结果
result1, result2, result3 = dask.compute(
    operation1,
    operation2,
    operation3
)
```

### 内存持久化
```python
# 将结果保留在分布式内存中复用
ddf_cached = ddf.persist()

# 后续操作不会重复计算
result1 = ddf_cached.mean().compute()
result2 = ddf_cached.sum().compute()
```

## 索引管理

### 设置索引
```python
# 设置索引（高效连接和特定操作必需）
ddf = ddf.set_index('timestamp', sorted=True)
```

### 索引特性
- 有序索引支持高效筛选和连接
- 索引决定数据分区
- 合适的索引可提升操作性能

## 结果输出

### 写入文件
```python
# 写入多文件（每个分区一个文件）
ddf.to_parquet('output/data.parquet')
ddf.to_csv('output/data-*.csv')

# 写入单文件（强制计算并合并）
ddf.compute().to_csv('output/single_file.csv')
```

### 转存内存（Pandas）
```python
# 转换为pandas（加载全部数据到内存）
pdf = ddf.compute()
```

## 性能优化

### 高效操作
- 列选择和筛选：极高效率
- 简单聚合（求和/均值/计数）：高效
- 分区级行操作：使用 `map_partitions` 高效

### 昂贵操作
- 排序：需跨工作节点数据重排
- 多分组GroupBy：可能需要重排
- 复杂连接：依赖数据分布
- 逐行应用：产生大量任务

### 优化建议

**1. 提前选择列**
```python
# 推荐：仅读取所需列
ddf = dd.read_parquet('data.parquet', columns=['col1', 'col2'])
```

**2. 聚合前筛选**
```python
# 推荐：在昂贵操作前缩减数据
result = ddf[ddf['year'] == 2024].groupby('category').sum().compute()
```

**3. 使用高效文件格式**
```python
# 用Parquet替代CSV提升性能
ddf.to_parquet('data.parquet')  # 更快/更小/列式存储
```

**4. 合理重分区**
```python
# 分区过小时
ddf = ddf.repartition(npartitions=10)

# 分区过大时
ddf = ddf.repartition(partition_size='100MB')
```

## 常用模式

### ETL流程
```python
import dask.dataframe as dd

# 读取数据
ddf = dd.read_csv('raw_data/*.csv')

# 转换
ddf = ddf[ddf['status'] == 'valid']
ddf['amount'] = ddf['amount'].astype('float64')
ddf = ddf.dropna(subset=['important_col'])

# 聚合
summary = ddf.groupby('category').agg({
    'amount': ['sum', 'mean'],
    'quantity': 'count'
})

# 输出结果
summary.to_parquet('output/summary.parquet')
```

### 时间序列分析
```python
# 读取时间序列数据
ddf = dd.read_parquet('timeseries/*.parquet')

# 设置时间戳索引
ddf = ddf.set_index('timestamp', sorted=True)

# 重采样（取决于Dask版本支持）
hourly = ddf.resample('1H').mean()

# 计算统计量
result = hourly.compute()
```

### 多文件合并处理
```python
# 读取多文件为单一数据框
ddf = dd.read_csv('data/2024-*.csv')

# 处理合并数据
result = ddf.groupby('category')['value'].sum().compute()
```

## 与Pandas的差异及限制

### 未完全实现Pandas功能
部分pandas操作在Dask中不可用：
- 某些字符串方法
- 特定窗口函数
- 部分专业统计函数

### 分区影响显著
- 分区内操作高效
- 跨分区操作可能昂贵
- 基于索引的操作受益于有序索引

### 惰性求值特性
- 操作在 `.compute()` 前不执行
- 需注意计算触发时机
- 无法在不计算的情况下检查中间结果

## 调试技巧

### 检查分区
```python
# 获取分区数量
print(ddf.npartitions)

# 计算单个分区
first_partition = ddf.get_partition(0).compute()

# 查看前几行（计算首个分区）
print(ddf.head())
```

### 小数据验证
```python
# 先在小样本测试
sample = ddf.head(1000)
# 验证逻辑正确性
# 再扩展到全量数据
result = ddf.compute()
```

### 校验数据类型
```python
# 确认数据类型正确
print(ddf.dtypes)
```
