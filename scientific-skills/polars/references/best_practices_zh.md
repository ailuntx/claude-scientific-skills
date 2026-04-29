# Polars 最佳实践与性能指南

全面指导如何编写高效的 Polars 代码并避免常见陷阱。

## 性能优化

### 1. 使用惰性求值

**处理大型数据集时始终优先使用惰性模式：**

```python
# 错误：立即加载所有数据的急切模式
df = pl.read_csv("large_file.csv")
result = df.filter(pl.col("age") > 25).select("name", "age")

# 正确：先优化再执行的惰性模式
lf = pl.scan_csv("large_file.csv")
result = lf.filter(pl.col("age") > 25).select("name", "age").collect()
```

**惰性求值的优势：**
- 谓词下推（在源头过滤）
- 投影下推（仅读取所需列）
- 查询优化
- 并行执行规划

### 2. 尽早过滤和选择列

在流水线中尽早应用过滤和列选择操作：

```python
# 错误：先处理所有数据再过滤选择
result = (
    lf.group_by("category")
    .agg(pl.col("value").mean())
    .join(other, on="category")
    .filter(pl.col("value") > 100)
    .select("category", "value")
)

# 正确：尽早过滤和选择
result = (
    lf.select("category", "value")  # 仅选择所需列
    .filter(pl.col("value") > 100)  # 提前过滤
    .group_by("category")
    .agg(pl.col("value").mean())
    .join(other.select("category", "other_col"), on="category")
)
```

### 3. 避免使用 Python 函数

保持在表达式 API 内以维持并行化能力：

```python
# 错误：Python 函数会禁用并行化
df = df.with_columns(
    result=pl.col("value").map_elements(lambda x: x * 2, return_dtype=pl.Float64)
)

# 正确：使用原生表达式（可并行化）
df = df.with_columns(result=pl.col("value") * 2)
```

**必须使用自定义函数时：**
```python
# 如确实需要，请显式声明
df = df.with_columns(
    result=pl.col("value").map_elements(
        custom_function,
        return_dtype=pl.Float64,
        skip_nulls=True  # 优化空值处理
    )
)
```

### 4. 超大数据使用流式处理

处理超过内存容量的数据集时启用流式模式：

```python
# 流式模式分块处理数据
lf = pl.scan_parquet("very_large.parquet")
result = lf.filter(pl.col("value") > 100).collect(streaming=True)

# 或使用 sink 直接流式写入
lf.filter(pl.col("value") > 100).sink_parquet("output.parquet")
```

### 5. 优化数据类型

选择合适的数据类型以减少内存占用并提升性能：

```python
# 错误：默认类型可能浪费资源
df = pl.read_csv("data.csv")

# 正确：指定最优类型
df = pl.read_csv(
    "data.csv",
    dtypes={
        "id": pl.UInt32,  # 若值域匹配则替代 Int64
        "category": pl.Categorical,  # 用于低基数字符串
        "date": pl.Date,  # 替代字符串类型
        "small_int": pl.Int16,  # 替代 Int64
    }
)
```

**数据类型优化准则：**
- 使用能容纳数据的最小整数类型
- 对低基数（<50% 唯一值）字符串使用 `Categorical`
- 不需要时间精度时用 `Date` 替代 `Datetime`
- 对二元标志使用 `Boolean` 而非整数

### 6. 并行操作

结构化代码以最大化并行化：

```python
# 错误：顺序管道操作会禁用并行化
df = (
    df.pipe(operation1)
    .pipe(operation2)
    .pipe(operation3)
)

# 正确：组合操作可启用并行化
df = df.with_columns(
    result1=operation1_expr(),
    result2=operation2_expr(),
    result3=operation3_expr()
)
```

### 7. 连接后重分块

```python
# 连接操作可能导致数据碎片化
combined = pl.concat([df1, df2, df3])

# 重分块以提升后续操作性能
combined = pl.concat([df1, df2, df3], rechunk=True)
```

## 表达式模式

### 条件逻辑

**简单条件：**
```python
df.with_columns(
    status=pl.when(pl.col("age") >= 18)
        .then("成年")
        .otherwise("未成年")
)
```

**多重条件：**
```python
df.with_columns(
    grade=pl.when(pl.col("score") >= 90)
        .then("A")
        .when(pl.col("score") >= 80)
        .then("B")
        .when(pl.col("score") >= 70)
        .then("C")
        .when(pl.col("score") >= 60)
        .then("D")
        .otherwise("F")
)
```

**复杂条件：**
```python
df.with_columns(
    category=pl.when(
        (pl.col("revenue") > 1000000) & (pl.col("customers") > 100)
    )
    .then("企业级")
    .when(
        (pl.col("revenue") > 100000) | (pl.col("customers") > 50)
    )
    .then("商业级")
    .otherwise("入门级")
)
```

### 空值处理

**检查空值：**
```python
df.filter(pl.col("value").is_null())
df.filter(pl.col("value").is_not_null())
```

**填充空值：**
```python
# 常量填充
df.with_columns(pl.col("value").fill_null(0))

# 前向填充
df.with_columns(pl.col("value").fill_null(strategy="forward"))

# 后向填充
df.with_columns(pl.col("value").fill_null(strategy="backward"))

# 均值填充
df.with_columns(pl.col("value").fill_null(strategy="mean"))

# 分组填充
df.with_columns(
    pl.col("value").fill_null(pl.col("value").mean()).over("group")
)
```

**合并列（取首个非空值）：**
```python
df.with_columns(
    combined=pl.coalesce(["col1", "col2", "col3"])
)
```

### 列选择模式

**按名称选择：**
```python
df.select("col1", "col2", "col3")
```

**按模式选择：**
```python
# 正则表达式
df.select(pl.col("^sales_.*$"))

# 开头匹配
df.select(pl.col("^sales"))

# 结尾匹配
df.select(pl.col("_total$"))

# 包含匹配
df.select(pl.col(".*revenue.*"))
```

**按类型选择：**
```python
# 所有数值列
df.select(pl.col(pl.NUMERIC_DTYPES))

# 所有字符串列
df.select(pl.col(pl.Utf8))

# 多类型选择
df.select(pl.col(pl.NUMERIC_DTYPES, pl.Boolean))
```

**排除列：**
```python
df.select(pl.all().exclude("id", "timestamp"))
```

**多列转换：**
```python
# 对多列应用相同操作
df.select(
    pl.col("^sales_.*$") * 1.1  # 所有销售列增加10%
)
```

### 聚合模式

**多重聚合：**
```python
df.group_by("category").agg(
    pl.col("value").sum().alias("总和"),
    pl.col("value").mean().alias("平均值"),
    pl.col("value").std().alias("标准差"),
    pl.col("id").count().alias("计数"),
    pl.col("id").n_unique().alias("唯一值计数"),
    pl.col("value").min().alias("最小值"),
    pl.col("value").max().alias("最大值"),
    pl.col("value").quantile(0.5).alias("中位数"),
    pl.col("value").quantile(0.95).alias("95分位数")
)
```

**条件聚合：**
```python
df.group_by("category").agg(
    # 高值计数
    (pl.col("value") > 100).sum().alias("高值计数"),

    # 过滤值的平均值
    pl.col("value").filter(pl.col("active")).mean().alias("活跃平均值"),

    # 条件求和
    pl.when(pl.col("status") == "completed")
        .then(pl.col("amount"))
        .otherwise(0)
        .sum()
        .alias("完成总额")
)
```

**分组转换：**
```python
df.with_columns(
    # 分组统计量
    group_mean=pl.col("value").mean().over("category"),
    group_std=pl.col("value").std().over("category"),

    # 组内排名
    rank=pl.col("value").rank().over("category"),

    # 占组总值的百分比
    pct_of_group=(pl.col("value") / pl.col("value").sum().over("category")) * 100
)
```

## 常见陷阱与反模式

### 陷阱1：逐行迭代

```python
# 错误：切勿逐行迭代
for row in df.iter_rows():
    # 处理单行
    result = row[0] * 2

# 正确：使用向量化操作
df = df.with_columns(result=pl.col("value") * 2)
```

### 陷阱2：原地修改

```python
# 错误：Polars 不可变，此操作不符合预期
df["new_col"] = df["old_col"] * 2  # 可能有效但不推荐

# 正确：函数式风格
df = df.with_columns(new_col=pl.col("old_col") * 2)
```

### 陷阱3：未使用表达式

```python
# 错误：基于字符串的操作
df.select("value * 2")  # 无效操作

# 正确：基于表达式
df.select(pl.col("value") * 2)
```

### 陷阱4：低效连接

```python
# 错误：未过滤直接连接大表
result = large_df1.join(large_df2, on="id")

# 正确：连接前先过滤
result = (
    large_df1.filter(pl.col("active"))
    .join(
        large_df2.filter(pl.col("status") == "valid"),
        on="id"
    )
)
```

### 陷阱5：未指定类型

```python
# 错误：完全依赖 Polars 推断
df = pl.read_csv("data.csv")

# 正确：为正确性和性能指定类型
df = pl.read_csv(
    "data.csv",
    dtypes={"id": pl.Int64, "date": pl.Date, "category": pl.Categorical}
)
```

### 陷阱6：创建过多小型 DataFrame

```python
# 错误：多次操作产生中间 DataFrame
df1 = df.filter(pl.col("age") > 25)
df2 = df1.select("name", "age")
df3 = df2.sort("age")
result = df3.head(10)

# 正确：链式操作
result = (
    df.filter(pl.col("age") > 25)
    .select("name", "age")
    .sort("age")
    .head(10)
)

# 更佳：使用惰性模式
result = (
    df.lazy()
    .filter(pl.col("age") > 25)
    .select("name", "age")
    .sort("age")
    .head(10)
    .collect()
)
```

## 内存管理

### 监控内存使用

```python
# 检查 DataFrame 大小
print(f"预估大小: {df.estimated_size('mb'):.2f} MB")

# 操作中分析内存
lf = pl.scan_csv("large.csv")
print(lf.explain())  # 查看查询计划
```

### 减少内存占用

```python
# 1. 使用惰性模式
lf = pl.scan_parquet("data.parquet")

# 2. 流式处理结果
result = lf.collect(streaming=True)

# 3. 仅选择所需列
lf = lf.select("col1", "col2")

# 4. 优化数据类型
df = df.with_columns(
    pl.col("int_col").cast(pl.Int32),  # 可能时向下转型
    pl.col("category").cast(pl.Categorical)  # 低基数处理
)

# 5. 删除不需要的列
df = df.drop("large_text_col", "unused_col")
```

## 测试与调试

### 检查查询计划

```python
lf = pl.scan_csv("data.csv")
query = lf.filter(pl.col("age") > 25).select("name", "age")

# 查看优化后的查询计划
print(query.explain())

# 查看详细查询计划
print(query.explain(optimized=True))
```

### 开发阶段采样数据

```python
# 使用 n_rows 测试
df = pl.read_csv("large.csv", n_rows=1000)

# 或读取后采样
df_sample = df.sample(n=1000, seed=42)
```

### 验证数据模式

```python
# 检查模式
print(df.schema)

# 确保模式符合预期
expected_schema = {
    "id": pl.Int64,
    "name": pl.Utf8,
    "date": pl.Date
}

assert df.schema == expected_schema
```

### 性能分析

```python
import time

# 计时操作
start = time.time()
result = lf.collect()
print(f"执行时间: {time.time() - start:.2f}s")

# 对比急切模式与惰性模式
start = time.time()
df_eager = pl.read_csv("data.csv").filter(pl.col("age") > 25)
eager_time = time.time() - start

start = time.time()
df_lazy = pl.scan_csv("data.csv").filter(pl.col("age") > 25).collect()
lazy_time = time.time() - start

print(f"急切模式: {eager_time:.2f}s, 惰性模式: {lazy_time:.2f}s")
```

## 文件格式最佳实践

### 选择合适的格式

**Parquet：**
- 适用场景：大型数据集、归档、数据湖
- 优点：卓越压缩率、列式存储、快速读取
- 缺点：非人类可读

**CSV：**
- 适用场景：小型数据集、人工检查、遗留系统
- 优点：通用性强、人类可读
- 缺点：速度慢、文件体积大、不保留类型

**Arrow IPC：**
- 适用场景：进程间通信、临时存储
- 优点：最快速度、零拷贝、保留所有类型
- 缺点：压缩率低于 Parquet

### 文件读取最佳实践

```python
# 1. 使用惰性读取
lf = pl.scan_parquet("data.parquet")  # 非 read_parquet

# 2. 高效读取多文件
lf = pl.scan_parquet("data/*.parquet")  # 并行读取

# 3. 已知模式时显式声明
lf = pl.scan_csv(
    "data.csv",
    dtypes={"id": pl.Int64, "date": pl.Date}
)

# 4. 使用谓词下推
result = lf.filter(pl.col("date") >= "2023-01-01").collect()
```

### 文件写入最佳实践

```python
# 1. 大型数据使用 Parquet
df.write_parquet("output.parquet", compression="zstd")

# 2. 分区存储大型数据集
df.write_parquet("output", partition_by=["year", "month"])

# 3. 超大数据使用流式写入
lf.sink_parquet("output.parquet")  # 流式写入

# 4. 优化压缩设置
df.write_parquet(
    "output.parquet",
    compression="snappy",  # 快速压缩
    statistics=True  # 启用读取时的谓词下推
)
```

## 代码组织

### 可复用表达式

```python
# 定义可复用表达式
age_group = (
    pl.when(pl.col("age") < 18)
    .then("未成年")
    .when(pl.col("age") < 65)
    .then("成年")
    .otherwise("老年")
)

revenue_per_customer = pl.col("revenue") / pl.col("customer_count")

# 在多个上下文中使用
df = df.with_columns(
    age_group=age_group,
    rpc=revenue_per_customer
)

# 在过滤中复用
df = df.filter(revenue_per_customer > 100)
```

### 流水线函数

```python
def clean_data(lf: pl.LazyFrame) -> pl.LazyFrame:
    """数据清洗与标准化"""
    return lf.with_columns(
        pl.col("name").str.to_uppercase(),
        pl.col("date").str.strptime(pl.Date, "%Y-%m-%d"),

year=pl.col("date").dt.year(),
        amount_log=pl.col("amount").log()
    )

# 组合管道
result = (
    pl.scan_csv("data.csv")
    .pipe(clean_data)
    .pipe(add_features)
    .filter(pl.col("year") == 2023)
    .collect()
)
```

## 文档

始终为复杂的表达式和转换添加文档：

```python
# 良好实践：记录意图
df = df.with_columns(
    # 计算客户生命周期价值：总购买金额
    # 除以自首次购买以来的月数
    clv=(
        pl.col("total_purchases") /
        ((pl.col("last_purchase_date") - pl.col("first_purchase_date"))
         .dt.total_days() / 30)
    )
)
```

## 版本兼容性

```python
# 检查Polars版本
import polars as pl
print(pl.__version__)

# 功能可用性因版本而异
# 为生产代码记录版本要求
```
