# 核心 DataFrame 与数据加载

本文档涵盖 Vaex DataFrame 基础、从多种来源加载数据以及理解 DataFrame 结构。

## DataFrame 基础

Vaex DataFrame 是处理大型表格数据集的中心数据结构。与 pandas 不同，Vaex DataFrame：
- 采用**惰性求值** - 操作仅在需要时执行
- **支持核外计算** - 数据无需完全载入内存
- 支持**虚拟列** - 无内存开销的计算列
- 通过优化的 C++ 后端实现**每秒十亿行**处理能力

## 打开现有文件

### 主要方法：`vaex.open()`

加载数据的常用方式：

```python
import vaex

# 支持多种格式
df = vaex.open('data.hdf5')     # HDF5（推荐）
df = vaex.open('data.arrow')    # Apache Arrow（推荐）
df = vaex.open('data.parquet')  # Parquet
df = vaex.open('data.csv')      # CSV（大文件处理较慢）
df = vaex.open('data.fits')     # FITS（天文数据）

# 可将多个文件作为单个 DataFrame 打开
df = vaex.open('data_*.hdf5')   # 支持通配符
```

**关键特性：**
- **HDF5/Arrow 即时加载** - 内存映射文件，无加载时间
- **处理大型 CSV** - 自动分块处理大 CSV 文件
- **立即返回** - 惰性求值意味着需要时才执行计算

### 格式特定的加载器

```python
# 带选项的 CSV
df = vaex.from_csv(
    'large_file.csv',
    chunk_size=5_000_000,      # 分块处理
    convert=True,               # 自动转换为 HDF5
    copy_index=False            # 不复制 pandas 索引（若存在）
)

# Apache Arrow
df = vaex.open('data.arrow')    # 原生支持，速度极快

# HDF5（最优格式）
df = vaex.open('data.hdf5')     # 通过内存映射即时加载
```

## 从其他来源创建 DataFrame

### 从 Pandas 转换

```python
import pandas as pd
import vaex

# 转换 pandas DataFrame
pdf = pd.read_csv('data.csv')
df = vaex.from_pandas(pdf, copy_index=False)

# 警告：此操作会将整个 pandas DataFrame 载入内存
# 处理大数据时，建议直接使用 vaex.from_csv()
```

### 从 NumPy 数组创建

```python
import numpy as np
import vaex

# 单数组
x = np.random.rand(1_000_000)
df = vaex.from_arrays(x=x)

# 多数组
x = np.random.rand(1_000_000)
y = np.random.rand(1_000_000)
df = vaex.from_arrays(x=x, y=y)
```

### 从字典创建

```python
import vaex

# 字典包含列表/数组
data = {
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'salary': [50000, 60000, 70000]
}
df = vaex.from_dict(data)
```

### 从 Arrow 表创建

```python
import pyarrow as pa
import vaex

# 从 Arrow 表转换
arrow_table = pa.table({
    'x': [1, 2, 3],
    'y': [4, 5, 6]
})
df = vaex.from_arrow_table(arrow_table)
```

## 示例数据集

Vaex 提供内置测试数据集：

```python
import vaex

# 纽约出租车数据集（约 1GB，1100 万行）
df = vaex.example()

# 小型数据集
df = vaex.datasets.titanic()
df = vaex.datasets.iris()
```

## 检查 DataFrame

### 基本信息

```python
# 显示首尾行
print(df)

# 维度（行数，列数）
print(df.shape)  # 返回 (行数, 列数)
print(len(df))   # 行数

# 列名
print(df.columns)
print(df.column_names)

# 数据类型
print(df.dtypes)

# 内存使用（针对实体化列）
df.byte_size()
```

### 统计摘要

```python
# 所有数值列的快速统计
df.describe()

# 单列统计
df.x.mean()
df.x.std()
df.x.min()
df.x.max()
df.x.sum()
df.x.count()

# 分位数
df.x.quantile(0.5)   # 中位数
df.x.quantile([0.25, 0.5, 0.75])  # 多分位数
```

### 查看数据

```python
# 首尾行（返回 pandas DataFrame）
df.head(10)
df.tail(10)

# 随机抽样
df.sample(n=100)

# 转换为 pandas（注意大数据量！）
pdf = df.to_pandas_df()

# 仅转换特定列
pdf = df[['x', 'y']].to_pandas_df()
```

## DataFrame 结构

### 列操作

```python
# 以表达式形式访问列
x_column = df.x
y_column = df['y']

# 列操作返回表达式（惰性）
sum_column = df.x + df.y    # 尚未计算

# 列出所有列
print(df.get_column_names())

# 检查列类型
print(df.dtypes)

# 虚拟列与实体化列
print(df.get_column_names(virtual=False))  # 仅实体化列
print(df.get_column_names(virtual=True))   # 所有列
```

### 行操作

```python
# 行计数
row_count = len(df)
row_count = df.count()

# 单行访问（返回字典）
row = df.row(0)
print(row['column_name'])

# 注意：Vaex 中不建议遍历行
# 应使用向量化操作替代
```

## 使用表达式

表达式是 Vaex 表示尚未执行的计算的方式：

```python
# 创建表达式（不执行计算）
expr = df.x ** 2 + df.y

# 表达式可用于多种场景
mean_of_expr = expr.mean()          # 仍为惰性
df['new_col'] = expr                # 虚拟列
filtered = df[expr > 10]            # 筛选

# 强制求值
result = expr.values  # 返回 NumPy 数组（谨慎使用！）
```

## DataFrame 操作

### 复制

```python
# 浅拷贝（共享数据）
df_copy = df.copy()

# 深拷贝（独立数据）
df_deep = df.copy(deep=True)
```

### 裁剪/切片

```python
# 选择行范围
df_subset = df[1000:2000]      # 1000-2000 行
df_subset = df[:1000]          # 前 1000 行
df_subset = df[-1000:]         # 后 1000 行

# 注意：此操作创建视图而非副本（高效）
```

### 连接

```python
# 纵向连接（合并行）
df_combined = vaex.concat([df1, df2, df3])

# 横向连接（合并列）
# 使用 join 或直接分配列
df['new_col'] = other_df.some_column
```

## 最佳实践

1. **优先使用 HDF5 或 Arrow 格式** - 即时加载，性能最优
2. **将大型 CSV 转为 HDF5** - 单次转换可重复使用
3. **避免对大数据使用 `.to_pandas_df()`** - 违背 Vaex 设计初衷
4. **使用表达式替代 `.values`** - 保持操作惰性
5. **检查数据类型** - 确保数值列非字符串类型
6. **使用虚拟列** - 派生数据零内存开销

## 常见模式

### 模式：一次性 CSV 转 HDF5

```python
# 初始转换（执行一次）
df = vaex.from_csv('large_data.csv', convert='large_data.hdf5')

# 后续加载（即时）
df = vaex.open('large_data.hdf5')
```

### 模式：检查大型数据集

```python
import vaex

df = vaex.open('large_file.hdf5')

# 快速概览
print(df)                    # 首尾行
print(df.shape)             # 维度
print(df.describe())        # 统计信息

# 抽样详细检查
sample = df.sample(1000).to_pandas_df()
print(sample.head())
```

### 模式：加载多个文件

```python
# 将多个文件作为单个 DataFrame 加载
df = vaex.open('data_part*.hdf5')

# 或显式连接
df1 = vaex.open('data_2020.hdf5')
df2 = vaex.open('data_2021.hdf5')
df_all = vaex.concat([df1, df2])
```

## 常见问题与解决方案

### 问题：CSV 加载缓慢

```python
# 解决方案：先转为 HDF5
df = vaex.from_csv('large.csv', convert='large.hdf5')
# 后续加载：df = vaex.open('large.hdf5')
```

### 问题：列显示为字符串类型

```python
# 检查类型
print(df.dtypes)

# 转为数值类型（创建虚拟列）
df['age_numeric'] = df.age.astype('int64')
```

### 问题：小操作导致内存不足

```python
# 可能使用了 .values 或 .to_pandas_df()
# 解决方案：使用惰性操作

# 错误做法（载入内存）
array = df.x.values

# 正确做法（保持惰性）
mean = df.x.mean()
filtered = df[df.x > 10]
```

## 相关资源

- 数据操作与筛选：参见 `data_processing.md`
- 性能优化：参见 `performance.md`
- 文件格式详情：参见 `io_operations.md`
