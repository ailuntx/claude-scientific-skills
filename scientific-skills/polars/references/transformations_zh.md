# Polars 数据转换

关于 Polars 中连接、拼接和重塑操作的全面指南。

## 连接操作

连接操作基于公共列合并来自多个 DataFrame 的数据。

### 基本连接类型

**内连接（交集）：**
```python
# 仅保留两个 DataFrame 中匹配的行
result = df1.join(df2, on="id", how="inner")
```

**左连接（保留左侧全部 + 右侧匹配项）：**
```python
# 保留左侧所有行，添加右侧匹配行
result = df1.join(df2, on="id", how="left")
```

**外连接（并集）：**
```python
# 保留两个 DataFrame 的所有行
result = df1.join(df2, on="id", how="outer")
```

**交叉连接（笛卡尔积）：**
```python
# 左侧每行与右侧每行组合
result = df1.join(df2, how="cross")
```

**半连接（过滤左侧）：**
```python
# 仅保留左侧与右侧匹配的行
result = df1.join(df2, on="id", how="semi")
```

**反连接（非匹配左侧）：**
```python
# 仅保留左侧与右侧不匹配的行
result = df1.join(df2, on="id", how="anti")
```

### 连接语法变体

**单列连接：**
```python
df1.join(df2, on="id")
```

**多列连接：**
```python
df1.join(df2, on=["id", "date"])
```

**不同列名连接：**
```python
df1.join(df2, left_on="user_id", right_on="id")
```

**多列不同名连接：**
```python
df1.join(
    df2,
    left_on=["user_id", "date"],
    right_on=["id", "timestamp"]
)
```

### 后缀处理

当两个 DataFrame 存在同名列（非连接键）时：

```python
# 添加后缀区分列
result = df1.join(df2, on="id", suffix="_right")

# 结果：value, value_right（若两表均有"value"列）
```

### 连接示例

**示例1：客户订单**
```python
customers = pl.DataFrame({
    "customer_id": [1, 2, 3, 4],
    "name": ["Alice", "Bob", "Charlie", "David"]
})

orders = pl.DataFrame({
    "order_id": [101, 102, 103],
    "customer_id": [1, 2, 1],
    "amount": [100, 200, 150]
})

# 内连接 - 仅保留有订单的客户
result = customers.join(orders, on="customer_id", how="inner")

# 左连接 - 包含所有客户（即使无订单）
result = customers.join(orders, on="customer_id", how="left")
```

**示例2：时间序列数据**
```python
prices = pl.DataFrame({
    "date": ["2023-01-01", "2023-01-02", "2023-01-03"],
    "stock": ["AAPL", "AAPL", "AAPL"],
    "price": [150, 152, 151]
})

volumes = pl.DataFrame({
    "date": ["2023-01-01", "2023-01-02"],
    "stock": ["AAPL", "AAPL"],
    "volume": [1000000, 1100000]
})

result = prices.join(
    volumes,
    on=["date", "stock"],
    how="left"
)
```

### 模糊连接（最近匹配）

针对时间序列数据，匹配最近时间戳：

```python
# 匹配最近的前置时间戳
quotes = pl.DataFrame({
    "timestamp": [1, 2, 3, 4, 5],
    "stock": ["A", "A", "A", "A", "A"],
    "quote": [100, 101, 102, 103, 104]
})

trades = pl.DataFrame({
    "timestamp": [1.5, 3.5, 4.2],
    "stock": ["A", "A", "A"],
    "trade": [50, 75, 100]
})

result = trades.join_asof(
    quotes,
    on="timestamp",
    by="stock",
    strategy="backward"  # 或 "forward", "nearest"
)
```

## 数据拼接

拼接操作将 DataFrame 堆叠组合。

### 纵向拼接（行堆叠）

```python
df1 = pl.DataFrame({"a": [1, 2], "b": [3, 4]})
df2 = pl.DataFrame({"a": [5, 6], "b": [7, 8]})

# 行堆叠
result = pl.concat([df1, df2], how="vertical")
# 结果：4行，相同列结构
```

**处理模式不匹配：**
```python
df1 = pl.DataFrame({"a": [1, 2], "b": [3, 4]})
df2 = pl.DataFrame({"a": [5, 6], "c": [7, 8]})

# 对角线拼接 - 缺失列填充空值
result = pl.concat([df1, df2], how="diagonal")
# 结果：列a, b, c（缺失位置为null）
```

### 横向拼接（列堆叠）

```python
df1 = pl.DataFrame({"a": [1, 2, 3]})
df2 = pl.DataFrame({"b": [4, 5, 6]})

# 列堆叠
result = pl.concat([df1, df2], how="horizontal")
# 结果：3行，列a和b
```

**注意：** 横向拼接要求行数相同。

### 拼接选项

```python
# 拼接后重新分块（提升后续操作性能）
result = pl.concat([df1, df2], rechunk=True)

# 并行执行
result = pl.concat([df1, df2], parallel=True)
```

### 应用场景

**合并多源数据：**
```python
# 读取多个文件后拼接
files = ["data_2023.csv", "data_2024.csv", "data_2025.csv"]
dfs = [pl.read_csv(f) for f in files]
combined = pl.concat(dfs, how="vertical")
```

**添加计算列：**
```python
base = pl.DataFrame({"value": [1, 2, 3]})
computed = pl.DataFrame({"doubled": [2, 4, 6]})
result = pl.concat([base, computed], how="horizontal")
```

## 数据透视（宽格式）

将列的唯一值转换为多列结构。

### 基础透视

```python
df = pl.DataFrame({
    "date": ["2023-01", "2023-01", "2023-02", "2023-02"],
    "product": ["A", "B", "A", "B"],
    "sales": [100, 150, 120, 160]
})

# 透视：产品转为列名
pivoted = df.pivot(
    values="sales",
    index="date",
    columns="product"
)
# 结果：
# date     | A   | B
# 2023-01  | 100 | 150
# 2023-02  | 120 | 160
```

### 带聚合的透视

存在重复组合时进行聚合：

```python
df = pl.DataFrame({
    "date": ["2023-01", "2023-01", "2023-01"],
    "product": ["A", "A", "B"],
    "sales": [100, 110, 150]
})

# 聚合重复项
pivoted = df.pivot(
    values="sales",
    index="date",
    columns="product",
    aggregate_function="sum"  # 或 "mean", "max", "min" 等
)
```

### 多索引列透视

```python
df = pl.DataFrame({
    "region": ["North", "North", "South", "South"],
    "date": ["2023-01", "2023-01", "2023-01", "2023-01"],
    "product": ["A", "B", "A", "B"],
    "sales": [100, 150, 120, 160]
})

pivoted = df.pivot(
    values="sales",
    index=["region", "date"],
    columns="product"
)
```

## 逆透视/融化（长格式）

将多列转换为行（透视的逆操作）。

### 基础逆透视

```python
df = pl.DataFrame({
    "date": ["2023-01", "2023-02"],
    "product_A": [100, 120],
    "product_B": [150, 160]
})

# 逆透视：列转行
unpivoted = df.unpivot(
    index="date",
    on=["product_A", "product_B"]
)
# 结果：
# date     | variable   | value
# 2023-01  | product_A  | 100
# 2023-01  | product_B  | 150
# 2023-02  | product_A  | 120
# 2023-02  | product_B  | 160
```

### 自定义列名

```python
unpivoted = df.unpivot(
    index="date",
    on=["product_A", "product_B"],
    variable_name="product",
    value_name="sales"
)
```

### 模式匹配逆透视

```python
# 逆透视匹配模式的所有列
df = pl.DataFrame({
    "id": [1, 2],
    "sales_Q1": [100, 200],
    "sales_Q2": [150, 250],
    "sales_Q3": [120, 220],
    "revenue_Q1": [1000, 2000]
})

# 逆透视所有销售列
unpivoted = df.unpivot(
    index="id",
    on=pl.col("^sales_.*$")
)
```

## 数据展开（解嵌套列表）

将列表列转换为多行。

### 基础展开

```python
df = pl.DataFrame({
    "id": [1, 2],
    "values": [[1, 2, 3], [4, 5]]
})

# 将列表展开为行
exploded = df.explode("values")
# 结果：
# id | values
# 1  | 1
# 1  | 2
# 1  | 3
# 2  | 4
# 2  | 5
```

### 多列展开

```python
df = pl.DataFrame({
    "id": [1, 2],
    "letters": [["a", "b"], ["c", "d"]],
    "numbers": [[1, 2], [3, 4]]
})

# 展开多列（长度必须相同）
exploded = df.explode("letters", "numbers")
```

## 数据转置

行列互换：

```python
df = pl.DataFrame({
    "metric": ["sales", "costs", "profit"],
    "Q1": [100, 60, 40],
    "Q2": [150, 80, 70]
})

# 转置操作
transposed = df.transpose(
    include_header=True,
    header_name="quarter",
    column_names="metric"
)
# 结果：季度为行，指标为列
```

## 重塑模式

### 模式1：宽表转长表再转宽表

```python
# 初始宽表
wide = pl.DataFrame({
    "id": [1, 2],
    "A": [10, 20],
    "B": [30, 40]
})

# 转为长表
long = wide.unpivot(index="id", on=["A", "B"])

# 转回宽表（可含转换）
wide_again = long.pivot(values="value", index="id", columns="variable")
```

### 模式2：嵌套转扁平

```python
# 嵌套数据
df = pl.DataFrame({
    "user": [1, 2],
    "purchases": [
        [{"item": "A", "qty": 2}, {"item": "B", "qty": 1}],
        [{"item": "C", "qty": 3}]
    ]
})

# 展开并解嵌套
flat = (
    df.explode("purchases")
    .unnest("purchases")
)
```

### 模式3：聚合后透视

```python
# 原始数据
sales = pl.DataFrame({
    "date": ["2023-01", "2023-01", "2023-02"],
    "product": ["A", "B", "A"],
    "sales": [100, 150, 120]
})

# 聚合后透视
result = (
    sales
    .group_by("date", "product")
    .agg(pl.col("sales").sum())
    .pivot(values="sales", index="date", columns="product")
)
```

## 高级转换

### 条件重塑

```python
# 仅透视特定值
df.filter(pl.col("year") >= 2020).pivot(...)

# 带过滤的逆透视
df.unpivot(index="id", on=pl.col("^sales.*$"))
```

### 多级转换

```python
# 复杂重塑流程
result = (
    df
    .unpivot(index="id", on=pl.col("^Q[0-9]_.*$"))
    .with_columns(
        quarter=pl.col("variable").str.extract(r"Q([0-9])", 1),
        metric=pl.col("variable").str.extract(r"Q[0-9]_(.*)", 1)
    )
    .drop("variable")
    .pivot(values="value", index=["id", "quarter"], columns="metric")
)
```

## 性能考量

### 连接性能优化

```python
# 1. 优先在索引/排序列上连接
df1_sorted = df1.sort("id")
df2_sorted = df2.sort("id")
result = df1_sorted.join(df2_sorted, on="id")

# 2. 选用合适连接类型
# 半/反连接优于内连接+过滤
matches = df1.join(df2, on="id", how="semi")  # 优于内连接后过滤

# 3. 连接前预过滤
df1_filtered = df1.filter(pl.col("active"))
result = df1_filtered.join(df2, on="id")  # 缩小连接范围
```

### 拼接性能优化

```python
# 1. 拼接后重新分块
result = pl.concat(dfs, rechunk=True)

# 2. 大文件拼接使用惰性模式
lf1 = pl.scan_parquet("file1.parquet")
lf2 = pl.scan_parquet("file2.parquet")
result = pl.concat([lf1, lf2]).collect()
```

### 透视性能优化

```python
# 1. 透视前过滤
pivoted = df.filter(pl.col("year") == 2023).pivot(...)

# 2. 显式指定聚合函数
pivoted = df.pivot(..., aggregate_function="first")  # 单值场景比"sum"更快
```

## 典型应用场景

### 时间序列对齐

```python
# 对齐不同时间戳的两组序列
ts1.join_asof(ts2, on="timestamp", strategy="backward")
```

### 特征工程

```python
# 创建滞后特征
df.with_columns(
    pl.col("value").shift(1).over("user_id").alias("prev_value"),
    pl.col("value").shift(2).over("user_id").alias("prev_prev_value")
)
```

### 数据反规范化

```python
# 合并规范化表
orders.join(customers, on="customer_id").join(products, on="product_id")
```

### 报表生成

```python
# 透视生成报表
sales.pivot(values="amount", index="month", columns="product")
```
