# Polars 操作参考

本参考涵盖所有常见 Polars 操作及完整示例。

## 选择操作

### 选择列

**基础选择：**
```python
# 选择特定列
df.select("name", "age", "city")

# 使用表达式
df.select(pl.col("name"), pl.col("age"))
```

**模式匹配选择：**
```python
# 所有以 "sales_" 开头的列
df.select(pl.col("^sales_.*$"))

# 所有数值列
df.select(pl.col(pl.NUMERIC_DTYPES))

# 排除特定列
df.select(pl.all().exclude("id", "timestamp"))
```

**计算列：**
```python
df.select(
    "name",
    (pl.col("age") * 12).alias("age_in_months"),
    (pl.col("salary") * 1.1).alias("salary_after_raise")
)
```

### 列操作（添加/修改）

添加新列或修改现有列，同时保留其他列：

```python
# 添加新列
df.with_columns(
    age_doubled=pl.col("age") * 2,
    full_name=pl.col("first_name") + " " + pl.col("last_name")
)

# 修改现有列
df.with_columns(
    pl.col("name").str.to_uppercase().alias("name"),
    pl.col("salary").cast(pl.Float64).alias("salary")
)

# 并行多操作
df.with_columns(
    pl.col("value") * 10,
    pl.col("value") * 100,
    pl.col("value") * 1000,
)
```

## 筛选操作

### 基础筛选

```python
# 单条件
df.filter(pl.col("age") > 25)

# 多条件（AND）
df.filter(
    pl.col("age") > 25,
    pl.col("city") == "NY"
)

# OR 条件
df.filter(
    (pl.col("age") > 30) | (pl.col("salary") > 100000)
)

# NOT 条件
df.filter(~pl.col("active"))
df.filter(pl.col("city") != "NY")
```

### 高级筛选

**字符串操作：**
```python
# 包含子串
df.filter(pl.col("name").str.contains("John"))

# 开头匹配
df.filter(pl.col("email").str.starts_with("admin"))

# 正则匹配
df.filter(pl.col("phone").str.contains(r"^\d{3}-\d{3}-\d{4}$"))
```

**成员检查：**
```python
# 在列表中
df.filter(pl.col("city").is_in(["NY", "LA", "SF"]))

# 不在列表中
df.filter(~pl.col("status").is_in(["inactive", "deleted"]))
```

**范围筛选：**
```python
# 值区间
df.filter(pl.col("age").is_between(25, 35))

# 日期区间
df.filter(
    pl.col("date") >= pl.date(2023, 1, 1),
    pl.col("date") <= pl.date(2023, 12, 31)
)
```

**空值筛选：**
```python
# 过滤空值
df.filter(pl.col("value").is_not_null())

# 仅保留空值
df.filter(pl.col("value").is_null())
```

## 分组与聚合

### 基础分组

```python
# 单列分组
df.group_by("department").agg(
    pl.col("salary").mean().alias("avg_salary"),
    pl.len().alias("employee_count")
)

# 多列分组
df.group_by("department", "location").agg(
    pl.col("salary").sum()
)

# 保持顺序
df.group_by("category", maintain_order=True).agg(
    pl.col("value").sum()
)
```

### 聚合函数

**计数与长度：**
```python
df.group_by("category").agg(
    pl.len().alias("count"),
    pl.col("id").count().alias("non_null_count"),
    pl.col("id").n_unique().alias("unique_count")
)
```

**统计聚合：**
```python
df.group_by("group").agg(
    pl.col("value").sum().alias("total"),
    pl.col("value").mean().alias("average"),
    pl.col("value").median().alias("median"),
    pl.col("value").std().alias("std_dev"),
    pl.col("value").var().alias("variance"),
    pl.col("value").min().alias("minimum"),
    pl.col("value").max().alias("maximum"),
    pl.col("value").quantile(0.95).alias("p95")
)
```

**首尾值：**
```python
df.group_by("user_id").agg(
    pl.col("timestamp").first().alias("first_seen"),
    pl.col("timestamp").last().alias("last_seen"),
    pl.col("event").first().alias("first_event")
)
```

**列表聚合：**
```python
# 收集值到列表
df.group_by("category").agg(
    pl.col("item").alias("all_items")  # 创建列表列
)
```

### 条件聚合

聚合内筛选：

```python
df.group_by("department").agg(
    # 统计高收入者
    (pl.col("salary") > 100000).sum().alias("high_earners"),

    # 筛选值平均
    pl.col("salary").filter(pl.col("bonus") > 0).mean().alias("avg_with_bonus"),

    # 条件求和
    pl.when(pl.col("active"))
      .then(pl.col("sales"))
      .otherwise(0)
      .sum()
      .alias("active_sales")
)
```

### 多重聚合

高效组合多个聚合：

```python
df.group_by("store_id").agg(
    pl.col("transaction_id").count().alias("num_transactions"),
    pl.col("amount").sum().alias("total_sales"),
    pl.col("amount").mean().alias("avg_transaction"),
    pl.col("customer_id").n_unique().alias("unique_customers"),
    pl.col("amount").max().alias("largest_transaction"),
    pl.col("timestamp").min().alias("first_transaction_date"),
    pl.col("timestamp").max().alias("last_transaction_date")
)
```

## 窗口函数

窗口函数在保留原始行数的同时应用聚合。

### 基础窗口操作

**分组统计：**
```python
# 添加分组均值到每行
df.with_columns(
    avg_age_by_dept=pl.col("age").mean().over("department")
)

# 多分组列
df.with_columns(
    group_avg=pl.col("value").mean().over("category", "region")
)
```

**排名：**
```python
df.with_columns(
    # 组内排名
    rank=pl.col("score").rank().over("team"),

    # 密集排名（无间隔）
    dense_rank=pl.col("score").rank(method="dense").over("team"),

    # 行号
    row_num=pl.col("timestamp").sort().rank(method="ordinal").over("user_id")
)
```

### 窗口映射策略

**group_to_rows（默认）：**
保留原始行序：
```python
df.with_columns(
    group_mean=pl.col("value").mean().over("category", mapping_strategy="group_to_rows")
)
```

**explode：**
更快，集中分组行：
```python
df.with_columns(
    group_mean=pl.col("value").mean().over("category", mapping_strategy="explode")
)
```

**join：**
创建列表列：
```python
df.with_columns(
    group_values=pl.col("value").over("category", mapping_strategy="join")
)
```

### 滚动窗口

**基于时间：**
```python
df.with_columns(
    rolling_avg=pl.col("value").rolling_mean(
        window_size="7d",
        by="date"
    )
)
```

**基于行数：**
```python
df.with_columns(
    rolling_sum=pl.col("value").rolling_sum(window_size=3),
    rolling_max=pl.col("value").rolling_max(window_size=5)
)
```

### 累积操作

```python
df.with_columns(
    cumsum=pl.col("value").cum_sum().over("group"),
    cummax=pl.col("value").cum_max().over("group"),
    cummin=pl.col("value").cum_min().over("group"),
    cumprod=pl.col("value").cum_prod().over("group")
)
```

### 位移与滞后/超前

```python
df.with_columns(
    # 前值（滞后）
    prev_value=pl.col("value").shift(1).over("user_id"),

    # 后值（超前）
    next_value=pl.col("value").shift(-1).over("user_id"),

    # 计算与前值差值
    diff=pl.col("value") - pl.col("value").shift(1).over("user_id")
)
```

## 排序

### 基础排序

```python
# 单列排序
df.sort("age")

# 降序排序
df.sort("age", descending=True)

# 多列排序
df.sort("department", "age")

# 混合排序方向
df.sort(["department", "salary"], descending=[False, True])
```

### 高级排序

**空值处理：**
```python
# 空值在前
df.sort("value", nulls_last=False)

# 空值在后
df.sort("value", nulls_last=True)
```

**表达式排序：**
```python
# 按计算值排序
df.sort(pl.col("first_name").str.len())

# 多表达式排序
df.sort(
    pl.col("last_name").str.to_lowercase(),
    pl.col("age").abs()
)
```

## 条件操作

### When/Then/Otherwise

```python
# 基础条件
df.with_columns(
    status=pl.when(pl.col("age") >= 18)
        .then("adult")
        .otherwise("minor")
)

# 多条件
df.with_columns(
    category=pl.when(pl.col("score") >= 90)
        .then("A")
        .when(pl.col("score") >= 80)
        .then("B")
        .when(pl.col("score") >= 70)
        .then("C")
        .otherwise("F")
)

# 条件计算
df.with_columns(
    adjusted_price=pl.when(pl.col("is_member"))
        .then(pl.col("price") * 0.9)
        .otherwise(pl.col("price"))
)
```

## 字符串操作

### 常用字符串方法

```python
df.with_columns(
    # 大小写转换
    upper=pl.col("name").str.to_uppercase(),
    lower=pl.col("name").str.to_lowercase(),
    title=pl.col("name").str.to_titlecase(),

    # 修剪
    trimmed=pl.col("text").str.strip_chars(),

    # 子串
    first_3=pl.col("name").str.slice(0, 3),

    # 替换
    cleaned=pl.col("text").str.replace("old", "new"),
    cleaned_all=pl.col("text").str.replace_all("old", "new"),

    # 分割
    parts=pl.col("full_name").str.split(" "),

    # 长度
    name_length=pl.col("name").str.len_chars()
)
```

### 字符串筛选

```python
# 包含
df.filter(pl.col("email").str.contains("@gmail.com"))

# 开头/结尾匹配
df.filter(pl.col("name").str.starts_with("A"))
df.filter(pl.col("file").str.ends_with(".csv"))

# 正则匹配
df.filter(pl.col("phone").str.contains(r"^\d{3}-\d{4}$"))
```

## 日期时间操作

### 日期解析

```python
# 字符串转日期
df.with_columns(
    date=pl.col("date_str").str.strptime(pl.Date, "%Y-%m-%d"),
    datetime=pl.col("dt_str").str.strptime(pl.Datetime, "%Y-%m-%d %H:%M:%S")
)
```

### 日期组件

```python
df.with_columns(
    year=pl.col("date").dt.year(),
    month=pl.col("date").dt.month(),
    day=pl.col("date").dt.day(),
    weekday=pl.col("date").dt.weekday(),
    hour=pl.col("datetime").dt.hour(),
    minute=pl.col("datetime").dt.minute()
)
```

### 日期运算

```python
# 添加时长
df.with_columns(
    next_week=pl.col("date") + pl.duration(weeks=1),
    next_month=pl.col("date") + pl.duration(months=1)
)

# 日期差值
df.with_columns(
    days_diff=(pl.col("end_date") - pl.col("start_date")).dt.total_days()
)
```

### 日期筛选

```python
# 按日期范围筛选
df.filter(
    pl.col("date").is_between(pl.date(2023, 1, 1), pl.date(2023, 12, 31))
)

# 按年份筛选
df.filter(pl.col("date").dt.year() == 2023)

# 按月份筛选
df.filter(pl.col("date").dt.month().is_in([6, 7, 8]))  # 夏季月份
```

## 列表操作

### 列表列处理

```python
# 创建列表列
df.with_columns(
    items_list=pl.col("item1", "item2", "item3").to_list()
)

# 列表操作
df.with_columns(
    list_len=pl.col("items").list.len(),
    first_item=pl.col("items").list.first(),
    last_item=pl.col("items").list.last(),
    unique_items=pl.col("items").list.unique(),
    sorted_items=pl.col("items").list.sort()
)

# 展开列表为行
df.explode("items")

# 筛选列表元素
df.with_columns(
    filtered=pl.col("items").list.eval(pl.element() > 10)
)
```

## 结构体操作

### 嵌套结构处理

```python
# 创建结构体列
df.with_columns(
    address=pl.struct(["street", "city", "zip"])
)

# 访问结构体字段
df.with_columns(
    city=pl.col("address").struct.field("city")
)

# 展开结构体为列
df.unnest("address")
```

## 唯一值与重复项操作

```python
# 获取唯一行
df.unique()

# 指定列唯一
df.unique(subset=["name", "email"])

# 保留首个/末个重复项
df.unique(subset=["id"], keep="first")
df.unique(subset=["id"], keep="last")

# 标识重复项
df.with_columns(
    is_duplicate=pl.col("id").is_duplicated()
)

# 统计重复项
df.group_by("email").agg(
    pl.len().alias("count")
).filter(pl.col("count") > 1)
```

## 采样

```python
# 随机采样
df.sample(n=100)

# 比例采样
df.sample(fraction=0.1)

# 可复现采样
df.sample(n=100, seed=42)
```

## 列重命名

```python
# 重命名特定列
df.rename({"old_name": "new_name", "age": "years"})

# 表达式重命名
df.select(pl.col("*").name.suffix("_renamed"))
df.select(pl.col("*").name.prefix("data_"))
df.select(pl.col("*").name.to_uppercase())
```
