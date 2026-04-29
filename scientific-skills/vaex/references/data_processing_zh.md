# 数据处理与操作

本文档涵盖 Vaex 中的筛选、选择、虚拟列、表达式、聚合、分组操作及数据转换。

## 筛选与选择

Vaex 使用布尔表达式高效筛选数据而无需复制：

### 基础筛选

```python
# 简单筛选
df_filtered = df[df.age > 25]

# 多条件筛选
df_filtered = df[(df.age > 25) & (df.salary > 50000)]
df_filtered = df[(df.category == 'A') | (df.category == 'B')]

# 取反筛选
df_filtered = df[~(df.age < 18)]
```

### 选择对象

Vaex 可同时维护多个命名选择：

```python
# 创建命名选择
df.select(df.age > 30, name='adults')
df.select(df.salary > 100000, name='high_earners')

# 在操作中使用选择
mean_age_adults = df.mean(df.age, selection='adults')
count_high_earners = df.count(selection='high_earners')

# 组合选择
df.select((df.age > 30) & (df.salary > 100000), name='adult_high_earners')

# 列出所有选择
print(df.selection_names())

# 删除选择
df.select_drop('adults')
```

### 高级筛选

```python
# 字符串匹配
df_filtered = df[df.name.str.contains('John')]
df_filtered = df[df.name.str.startswith('A')]
df_filtered = df[df.email.str.endswith('@gmail.com')]

# 空值/缺失值筛选
df_filtered = df[df.age.isna()]      # 保留缺失值
df_filtered = df[df.age.notna()]     # 移除缺失值

# 值成员判断
df_filtered = df[df.category.isin(['A', 'B', 'C'])]

# 范围筛选
df_filtered = df[df.age.between(25, 65)]
```

## 虚拟列与表达式

虚拟列实时计算且零内存开销：

### 创建虚拟列

```python
# 算术运算
df['total'] = df.price * df.quantity
df['price_squared'] = df.price ** 2

# 数学函数
df['log_price'] = df.price.log()
df['sqrt_value'] = df.value.sqrt()
df['abs_diff'] = (df.x - df.y).abs()

# 条件逻辑
df['is_adult'] = df.age >= 18
df['category'] = (df.score > 80).where('A', 'B')  # 条件选择
```

### 表达式方法

```python
# 数学运算
df.x.abs()          # 绝对值
df.x.sqrt()         # 平方根
df.x.log()          # 自然对数
df.x.log10()        # 常用对数
df.x.exp()          # 指数

# 三角函数
df.angle.sin()
df.angle.cos()
df.angle.tan()
df.angle.arcsin()

# 舍入
df.x.round(2)       # 保留两位小数
df.x.floor()        # 向下取整
df.x.ceil()         # 向上取整

# 类型转换
df.x.astype('int64')
df.x.astype('float32')
df.x.astype('str')
```

### 条件表达式

```python
# where()方法: condition.where(true_value, false_value)
df['status'] = (df.age >= 18).where('adult', 'minor')

# 嵌套where实现多条件
df['grade'] = (df.score >= 90).where('A',
              (df.score >= 80).where('B',
              (df.score >= 70).where('C', 'F')))

# 使用searchsorted分箱
bins = [0, 18, 65, 100]
labels = ['minor', 'adult', 'senior']
df['age_group'] = df.age.searchsorted(bins).where(...)
```

## 字符串操作

通过 `.str` 访问器调用字符串方法：

### 基础字符串方法

```python
# 大小写转换
df['upper_name'] = df.name.str.upper()
df['lower_name'] = df.name.str.lower()
df['title_name'] = df.name.str.title()

# 修剪
df['trimmed'] = df.text.str.strip()
df['ltrimmed'] = df.text.str.lstrip()
df['rtrimmed'] = df.text.str.rstrip()

# 搜索
df['has_john'] = df.name.str.contains('John')
df['starts_with_a'] = df.name.str.startswith('A')
df['ends_with_com'] = df.email.str.endswith('.com')

# 切片
df['first_char'] = df.name.str.slice(0, 1)
df['last_three'] = df.name.str.slice(-3, None)

# 长度
df['name_length'] = df.name.str.len()
```

### 高级字符串操作

```python
# 替换
df['clean_text'] = df.text.str.replace('bad', 'good')

# 拆分（返回第一部分）
df['first_name'] = df.full_name.str.split(' ')[0]

# 拼接
df['full_name'] = df.first_name + ' ' + df.last_name

# 填充
df['padded'] = df.code.str.pad(10, '0', 'left')  # 左侧零填充
```

## 日期时间操作

通过 `.dt` 访问器调用日期时间方法：

### 日期时间属性

```python
# 字符串转日期时间
df['date_parsed'] = df.date_string.astype('datetime64')

# 提取时间分量
df['year'] = df.timestamp.dt.year
df['month'] = df.timestamp.dt.month
df['day'] = df.timestamp.dt.day
df['hour'] = df.timestamp.dt.hour
df['minute'] = df.timestamp.dt.minute
df['second'] = df.timestamp.dt.second

# 星期几
df['weekday'] = df.timestamp.dt.dayofweek  # 0=周一
df['day_name'] = df.timestamp.dt.day_name  # 'Monday', 'Tuesday', ...

# 日期运算
df['tomorrow'] = df.date + pd.Timedelta(days=1)
df['next_week'] = df.date + pd.Timedelta(weeks=1)
```

## 聚合操作

Vaex 可高效处理数十亿行数据的聚合：

### 基础聚合

```python
# 单列聚合
mean_age = df.age.mean()
std_age = df.age.std()
min_age = df.age.min()
max_age = df.age.max()
sum_sales = df.sales.sum()
count_rows = df.count()

# 带选择集的聚合
mean_adult_age = df.age.mean(selection='adults')

# 延迟批量聚合
mean = df.age.mean(delay=True)
std = df.age.std(delay=True)
results = vaex.execute([mean, std])
```

### 可用聚合函数

```python
# 集中趋势
df.x.mean()
df.x.median_approx()  # 近似中位数（快速）

# 离散度
df.x.std()           # 标准差
df.x.var()           # 方差
df.x.min()
df.x.max()
df.x.minmax()        # 同时获取最小最大值

# 计数
df.count()           # 总行数
df.x.count()         # 非缺失值计数

# 求和与乘积
df.x.sum()
df.x.prod()

# 分位数
df.x.quantile(0.5)           # 中位数
df.x.quantile([0.25, 0.75])  # 四分位数

# 相关性
df.correlation(df.x, df.y)
df.covar(df.x, df.y)

# 高阶矩
df.x.kurtosis()
df.x.skew()

# 唯一值
df.x.nunique()       # 唯一值计数
df.x.unique()        # 获取唯一值（返回数组）
```

## 分组操作

按组聚合数据：

### 基础分组

```python
# 单列分组
grouped = df.groupby('category')

# 聚合
result = grouped.agg({'sales': 'sum'})
result = grouped.agg({'sales': 'sum', 'quantity': 'mean'})

# 单列多聚合
result = grouped.agg({
    'sales': ['sum', 'mean', 'std'],
    'quantity': 'sum'
})
```

### 高级分组

```python
# 多列分组
result = df.groupby(['category', 'region']).agg({
    'sales': 'sum',
    'quantity': 'mean'
})

# 自定义聚合函数
result = df.groupby('category').agg({
    'sales': lambda x: x.max() - x.min()
})

# 可用聚合函数
# 'sum', 'mean', 'std', 'min', 'max', 'count', 'first', 'last'
```

### 分箱分组

```python
# 对连续变量分箱后聚合
result = df.groupby(vaex.vrange(0, 100, 10)).agg({
    'sales': 'sum'
})

# 日期分箱
result = df.groupby(df.timestamp.dt.year).agg({
    'sales': 'sum'
})
```

## 分箱与离散化

将连续变量分箱：

### 简单分箱

```python
# 创建分箱
df['age_bin'] = df.age.digitize([18, 30, 50, 65, 100])

# 带标签分箱
bins = [0, 18, 30, 50, 65, 100]
labels = ['child', 'young_adult', 'adult', 'middle_age', 'senior']
df['age_group'] = df.age.digitize(bins)
# 注意：需使用where()或映射应用标签
```

### 统计分箱

```python
# 等宽分箱
df['value_bin'] = df.value.digitize(
    vaex.vrange(df.value.min(), df.value.max(), 10)
)

# 分位数分箱
quantiles = df.value.quantile([0.25, 0.5, 0.75])
df['value_quartile'] = df.value.digitize(quantiles)
```

## 多维聚合

在网格上计算统计量：

```python
# 二维直方图/热力图数据
counts = df.count(binby=[df.x, df.y], limits=[[0, 10], [0, 10]], shape=(100, 100))

# 网格均值
mean_z = df.mean(df.z, binby=[df.x, df.y], limits=[[0, 10], [0, 10]], shape=(50, 50))

# 网格多统计量
stats = df.mean(df.z, binby=[df.x, df.y], shape=(50, 50), delay=True)
counts = df.count(binby=[df.x, df.y], shape=(50, 50), delay=True)
results = vaex.execute([stats, counts])
```

## 缺失数据处理

处理缺失值、空值和 NaN：

### 检测缺失数据

```python
# 检查缺失值
df['age_missing'] = df.age.isna()
df['age_present'] = df.age.notna()

# 缺失值计数
missing_count = df.age.isna().sum()
missing_pct = df.age.isna().mean() * 100
```

### 处理缺失数据

```python
# 过滤缺失值
df_clean = df[df.age.notna()]

# 填充缺失值
df['age_filled'] = df.age.fillna(0)
df['age_filled'] = df.age.fillna(df.age.mean())

# 前向/后向填充（时间序列）
df['age_ffill'] = df.age.fillna(method='ffill')
df['age_bfill'] = df.age.fillna(method='bfill')
```

### Vaex 中的缺失数据类型

Vaex 区分三种类型：
- **NaN** - IEEE 浮点数非数值
- **NA** - Arrow 空类型
- **Missing** - 缺失数据统称

```python
# 检查缺失类型
df.is_masked('column_name')  # 使用Arrow空类型(NA)时返回True

# 类型转换
df['col_masked'] = df.col.as_masked()  # 转换为NA表示
```

## 排序

```python
# 单列排序
df_sorted = df.sort('age')
df_sorted = df.sort('age', ascending=False)

# 多列排序
df_sorted = df.sort(['category', 'age'])

# 注意：排序会物化新索引列
# 超大数据集需谨慎使用排序
```

## 连接数据框

基于键合并数据框：

```python
# 内连接
df_joined = df1.join(df2, on='key_column')

# 左连接
df_joined = df1.join(df2, on='key_column', how='left')

# 异名列连接
df_joined = df1.join(
    df2,
    left_on='id',
    right_on='user_id',
    how='left'
)

# 多键连接
df_joined = df1.join(df2, on=['key1', 'key2'])
```

## 添加与删除列

### 添加列

```python
# 虚拟列（无内存开销）
df['new_col'] = df.x + df.y

# 从外部数组添加（需长度匹配）
import numpy as np
new_data = np.random.rand(len(df))
df['random'] = new_data

# 常量列
df['constant'] = 42
```

### 删除列

```python
# 删除单列
df = df.drop('column_name')

# 删除多列
df = df.drop(['col1', 'col2', 'col3'])

# 选择特定列（删除其他列）
df = df[['col1', 'col2', 'col3']]
```

### 重命名列

```python
# 重命名单列
df = df.rename('old_name', 'new_name')

# 批量重命名
df = df.rename({
    'old_name1': 'new_name1',
    'old_name2': 'new_name2'
})
```

## 常用模式

### 模式：复杂特征工程

```python
# 多衍生特征
df['log_price'] = df.price.log()
df['price_per_unit'] = df.price / df.quantity
df['is_discount'] = df.discount > 0
df['price_category'] = (df.price > 100).where('expensive', 'affordable')
df['revenue'] = df.price * df.quantity * (1 - df.discount)
```

### 模式：文本清洗

```python
# 文本清洗标准化
df['email_clean'] = df.email.str.lower().str.strip()
df['has_valid_email'] = df.email_clean.str.contains('@')
df['domain'] = df.email_clean.str.split('@')[1]
```

### 模式：时间序列分析

```python
# 提取时间特征
df['year'] = df.timestamp.dt.year
df['month'] = df.timestamp.dt.month
df['day_of_week'] = df.timestamp.dt.dayofweek
df['is_weekend'] = df.day_of_week >= 5
df['quarter'] = ((df.month - 1) // 3) + 1
```

### 模式：分组统计

```python
# 按组计算统计量
monthly_sales = df.groupby(df.timestamp.dt.month).agg({
    'revenue': ['sum', 'mean', 'count'],
    'quantity': 'sum'
})

# 多级分组
category_region_sales = df.groupby(['category', 'region']).agg({
    'sales': 'sum',
    'profit': 'mean'
})
```

## 性能优化建议

1. **使用虚拟列** - 实时计算无内存开销
2. **批量操作 delay=True** - 一次性计算多个聚合
3. **避免 `.values` 或 `.to_pandas_df()`** - 尽量保持惰性计算
4. **使用选择集** - 命名选择比创建新数据框更高效
5. **利用表达式** - 支持查询优化
6. **减少排序操作** - 大数据集排序成本高昂

## 相关资源

- 数据框创建：参见 `core_dataframes.md`
- 性能优化：参见 `performance.md`
- 可视化：参见 `visualization.md`
- 机器学习流程：参见 `machine_learning.md`
