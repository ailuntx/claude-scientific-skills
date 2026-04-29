# Pandas 到 Polars 迁移指南

本指南通过全面的操作映射和关键差异，帮助您从 pandas 迁移到 Polars。

## 核心概念差异

### 1. 无索引系统

**Pandas:** 使用基于行的索引系统
```python
df.loc[0, "column"]
df.iloc[0:5]
df.set_index("id")
```

**Polars:** 仅使用整数位置
```python
df[0, "column"]  # 行位置，列名
df[0:5]  # 行切片
# 无 set_index 等效操作 - 改用 group_by
```

### 2. 内存格式

**Pandas:** 面向行的 NumPy 数组  
**Polars:** 列式 Apache Arrow 格式  

**影响:**  
- Polars 在列操作上更快  
- Polars 内存占用更少  
- Polars 具有更好的数据共享能力  

### 3. 并行处理

**Pandas:** 主要单线程（需 Dask 实现并行）  
**Polars:** 默认使用 Rust 并发机制实现并行  

### 4. 惰性求值

**Pandas:** 仅支持即时求值  
**Polars:** 同时支持即时求值（DataFrame）和惰性求值（LazyFrame）并带查询优化  

### 5. 类型严格性

**Pandas:** 允许隐式类型转换  
**Polars:** 严格类型检查，需显式转换  

**示例:**  
```python
# Pandas: 隐式转换为浮点型
pd_df["int_col"] = [1, 2, None, 4]  # dtype: float64

# Polars: 保留整数类型并包含空值
pl_df = pl.DataFrame({"int_col": [1, 2, None, 4]})  # dtype: Int64
```

## 操作映射

### 数据选择

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 选择列 | `df["col"]` 或 `df.col` | `df.select("col")` 或 `df["col"]` |
| 选择多列 | `df[["a", "b"]]` | `df.select("a", "b")` |
| 按位置选择 | `df.iloc[:, 0:3]` | `df.select(pl.col(df.columns[0:3]))` |
| 按条件选择 | `df[df["age"] > 25]` | `df.filter(pl.col("age") > 25)` |

### 数据过滤

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 单条件过滤 | `df[df["age"] > 25]` | `df.filter(pl.col("age") > 25)` |
| 多条件过滤 | `df[(df["age"] > 25) & (df["city"] == "NY")]` | `df.filter(pl.col("age") > 25, pl.col("city") == "NY")` |
| 查询方法 | `df.query("age > 25")` | `df.filter(pl.col("age") > 25)` |
| 包含检查 | `df[df["city"].isin(["NY", "LA"])]` | `df.filter(pl.col("city").is_in(["NY", "LA"]))` |
| 空值检查 | `df[df["value"].isna()]` | `df.filter(pl.col("value").is_null())` |
| 非空检查 | `df[df["value"].notna()]` | `df.filter(pl.col("value").is_not_null())` |

### 添加/修改列

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 添加列 | `df["new"] = df["old"] * 2` | `df.with_columns(new=pl.col("old") * 2)` |
| 多列添加 | `df.assign(a=..., b=...)` | `df.with_columns(a=..., b=...)` |
| 条件列 | `np.where(condition, a, b)` | `pl.when(condition).then(a).otherwise(b)` |

**重要差异 - 并行执行:**

```python
# Pandas: 顺序执行（lambda 可见前序结果）
df.assign(
    a=lambda df_: df_.value * 10,
    b=lambda df_: df_.value * 100
)

# Polars: 并行执行（同时计算所有列）
df.with_columns(
    a=pl.col("value") * 10,
    b=pl.col("value") * 100
)
```

### 分组聚合

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 分组 | `df.groupby("col")` | `df.group_by("col")` |
| 单聚合 | `df.groupby("col")["val"].mean()` | `df.group_by("col").agg(pl.col("val").mean())` |
| 多聚合 | `df.groupby("col").agg({"val": ["mean", "sum"]})` | `df.group_by("col").agg(pl.col("val").mean(), pl.col("val").sum())` |
| 分组大小 | `df.groupby("col").size()` | `df.group_by("col").agg(pl.len())` |
| 分组计数 | `df.groupby("col").count()` | `df.group_by("col").agg(pl.col("*").count())` |

### 窗口函数

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 转换 | `df.groupby("col").transform("mean")` | `df.with_columns(pl.col("val").mean().over("col"))` |
| 排名 | `df.groupby("col")["val"].rank()` | `df.with_columns(pl.col("val").rank().over("col"))` |
| 位移 | `df.groupby("col")["val"].shift(1)` | `df.with_columns(pl.col("val").shift(1).over("col"))` |
| 累加 | `df.groupby("col")["val"].cumsum()` | `df.with_columns(pl.col("val").cum_sum().over("col"))` |

### 连接操作

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 内连接 | `df1.merge(df2, on="id")` | `df1.join(df2, on="id", how="inner")` |
| 左连接 | `df1.merge(df2, on="id", how="left")` | `df1.join(df2, on="id", how="left")` |
| 不同键名 | `df1.merge(df2, left_on="a", right_on="b")` | `df1.join(df2, left_on="a", right_on="b")` |

### 数据拼接

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 垂直拼接 | `pd.concat([df1, df2], axis=0)` | `pl.concat([df1, df2], how="vertical")` |
| 水平拼接 | `pd.concat([df1, df2], axis=1)` | `pl.concat([df1, df2], how="horizontal")` |

### 排序

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 按列排序 | `df.sort_values("col")` | `df.sort("col")` |
| 降序 | `df.sort_values("col", ascending=False)` | `df.sort("col", descending=True)` |
| 多列排序 | `df.sort_values(["a", "b"])` | `df.sort("a", "b")` |

### 数据重塑

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 透视 | `df.pivot(index="a", columns="b", values="c")` | `df.pivot(values="c", index="a", columns="b")` |
| 逆透视 | `df.melt(id_vars="id")` | `df.unpivot(index="id")` |

### I/O 操作

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 读取 CSV | `pd.read_csv("file.csv")` | `pl.read_csv("file.csv")` 或 `pl.scan_csv()` |
| 写入 CSV | `df.to_csv("file.csv")` | `df.write_csv("file.csv")` |
| 读取 Parquet | `pd.read_parquet("file.parquet")` | `pl.read_parquet("file.parquet")` |
| 写入 Parquet | `df.to_parquet("file.parquet")` | `df.write_parquet("file.parquet")` |
| 读取 Excel | `pd.read_excel("file.xlsx")` | `pl.read_excel("file.xlsx")` |

### 字符串操作

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 大写 | `df["col"].str.upper()` | `df.select(pl.col("col").str.to_uppercase())` |
| 小写 | `df["col"].str.lower()` | `df.select(pl.col("col").str.to_lowercase())` |
| 包含 | `df["col"].str.contains("pattern")` | `df.filter(pl.col("col").str.contains("pattern"))` |
| 替换 | `df["col"].str.replace("old", "new")` | `df.select(pl.col("col").str.replace("old", "new"))` |
| 分割 | `df["col"].str.split(" ")` | `df.select(pl.col("col").str.split(" "))` |

### 日期时间操作

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 解析日期 | `pd.to_datetime(df["col"])` | `df.select(pl.col("col").str.strptime(pl.Date, "%Y-%m-%d"))` |
| 年份 | `df["date"].dt.year` | `df.select(pl.col("date").dt.year())` |
| 月份 | `df["date"].dt.month` | `df.select(pl.col("date").dt.month())` |
| 日期 | `df["date"].dt.day` | `df.select(pl.col("date").dt.day())` |

### 缺失数据处理

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 删除空值 | `df.dropna()` | `df.drop_nulls()` |
| 填充空值 | `df.fillna(0)` | `df.fill_null(0)` |
| 检查空值 | `df["col"].isna()` | `df.select(pl.col("col").is_null())` |
| 前向填充 | `df.fillna(method="ffill")` | `df.select(pl.col("col").fill_null(strategy="forward"))` |

### 其他操作

| 操作 | Pandas | Polars |
|-----------|--------|--------|
| 唯一值 | `df["col"].unique()` | `df["col"].unique()` |
| 值计数 | `df["col"].value_counts()` | `df["col"].value_counts()` |
| 描述统计 | `df.describe()` | `df.describe()` |
| 采样 | `df.sample(n=100)` | `df.sample(n=100)` |
| 头部 | `df.head()` | `df.head()` |
| 尾部 | `df.tail()` | `df.tail()` |

## 常见迁移模式

### 模式 1：链式操作

**Pandas:**
```python
result = (df
    .assign(new_col=lambda x: x["old_col"] * 2)
    .query("new_col > 10")
    .groupby("category")
    .agg({"value": "sum"})
    .reset_index()
)
```

**Polars:**
```python
result = (df
    .with_columns(new_col=pl.col("old_col") * 2)
    .filter(pl.col("new_col") > 10)
    .group_by("category")
    .agg(pl.col("value").sum())
)
# 无需 reset_index - Polars 无索引系统
```

### 模式 2：应用函数

**Pandas:**
```python
# Polars 中应避免 - 破坏并行性
df["result"] = df["value"].apply(lambda x: x * 2)
```

**Polars:**
```python
# 改用表达式
df = df.with_columns(result=pl.col("value") * 2)

# 如需自定义函数
df = df.with_columns(
    result=pl.col("value").map_elements(lambda x: x * 2, return_dtype=pl.Float64)
)
```

### 模式 3：条件列创建

**Pandas:**
```python
df["category"] = np.where(
    df["value"] > 100,
    "high",
    np.where(df["value"] > 50, "medium", "low")
)
```

**Polars:**
```python
df = df.with_columns(
    category=pl.when(pl.col("value") > 100)
        .then("high")
        .when(pl.col("value") > 50)
        .then("medium")
        .otherwise("low")
)
```

### 模式 4：分组转换

**Pandas:**
```python
df["group_mean"] = df.groupby("category")["value"].transform("mean")
```

**Polars:**
```python
df = df.with_columns(
    group_mean=pl.col("value").mean().over("category")
)
```

### 模式 5：多重聚合

**Pandas:**
```python
result = df.groupby("category").agg({
    "value": ["mean", "sum", "count"],
    "price": ["min", "max"]
})
```

**Polars:**
```python
result = df.group_by("category").agg(
    pl.col("value").mean().alias("value_mean"),
    pl.col("value").sum().alias("value_sum"),
    pl.col("value").count().alias("value_count"),
    pl.col("price").min().alias("price_min"),
    pl.col("price").max().alias("price_max")
)
```

## 需避免的性能反模式

### 反模式 1：顺序管道操作

**错误（禁用并行）:**
```python
df = df.pipe(function1).pipe(function2).pipe(function3)
```

**正确（启用并行）:**
```python
df = df.with_columns(
    function1_result(),
    function2_result(),
    function3_result()
)
```

### 反模式 2：在热点路径使用 Python 函数

**错误:**
```python
df = df.with_columns(
    result=pl.col("value").map_elements(lambda x: x * 2)
)
```

**正确:**
```python
df = df.with_columns(result=pl.col("value") * 2)
```

### 反模式 3：对大文件使用即时读取

**错误:**
```python
df = pl.read_csv("large_file.csv")
result = df.filter(pl.col("age") > 25).select("name", "age")
```

**正确:**
```python
lf = pl.scan_csv("large_file.csv")
result = lf.filter(pl.col("age") > 25).select("name", "age").collect()
```

### 反模式 4：行迭代

**错误:**
```python
for row in df.iter_rows():
    # 处理行
    pass
```

**正确:**
```python
# 使用向量化操作
df = df.with_columns(
    # 向量化计算
)
```

## 迁移检查清单

从 pandas 迁移到 Polars 时：

1. **移除索引操作** - 使用整数位置或 group_by
2. **替换 apply/map 为表达式** - 使用 Polars 原生操作
3. **更新列赋值方式** - 使用 `with_columns()` 替代直接赋值
4. **将 groupby.transform 改为 .over()** - 窗口函数工作方式不同
5. **更新字符串操作** - 使用 `.str.to_uppercase()` 替代 `.str.upper()`
6. **添加显式类型转换** - Polars 不会隐式转换类型
7. **考虑惰性求值** - 大数据集使用 `scan_*` 替代 `read_*`
8. **更新聚合语法** - Polars 语法更显式
9. **移除 reset_index 调用** - Polars 中不需要
10. **更新条件逻辑** - 使用 `when().then().otherwise()` 模式

## 兼容层

如需逐步迁移，可同时使用两个库：

```python
import pandas as pd
import polars as pl

# 将 pandas 转为 Polars
pl_df = pl.from_pandas(pd_df)

# 将 Polars 转为 pandas
pd_df = pl_df.to_pandas()

# 使用 Arrow 实现零拷贝（可能时）
pl_df = pl.from_arrow(pd_df)
pd_df = pl_df.to_arrow().to_pandas()
```

## 何时继续使用 Pandas

以下情况建议继续使用 pandas：
- 处理需要复杂索引操作的时间序列
- 需要广泛的生态支持（部分库仅支持 pandas）
-
