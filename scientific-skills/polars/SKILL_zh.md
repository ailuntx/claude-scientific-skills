---
name: polars
description: 适用于内存数据集的高速内存DataFrame库。当pandas过慢但数据仍能放入内存时使用。支持惰性求值、并行执行，后端基于Apache Arrow。最适合1-100GB数据集、ETL管道及替代pandas加速场景。超内存数据请使用dask或vaex。
license: https://github.com/pola-rs/polars/blob/main/LICENSE
metadata:
    skill-author: K-Dense Inc.
---

# Polars

## 概述

Polars是基于Apache Arrow构建的Python/Rust超高速DataFrame库。通过其表达式API、惰性求值框架和高效数据操作能力，可进行高效数据处理、pandas迁移和数据管道优化。

## 快速入门

### 安装与基础用法

安装Polars：
```python
uv pip install polars
```

基础DataFrame创建与操作：
```python
import polars as pl

# 创建DataFrame
df = pl.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "age": [25, 30, 35],
    "city": ["NY", "LA", "SF"]
})

# 选择列
df.select("name", "age")

# 过滤行
df.filter(pl.col("age") > 25)

# 添加计算列
df.with_columns(
    age_plus_10=pl.col("age") + 10
)
```

## 核心概念

### 表达式

表达式是Polars操作的基本构建块，用于描述数据转换逻辑，支持组合、复用和优化。

**核心原则：**
- 使用`pl.col("列名")`引用列
- 链式方法构建复杂转换
- 表达式在上下文（select/with_columns/filter/group_by）中惰性执行

**示例：**
```python
# 基于表达式的计算
df.select(
    pl.col("name"),
    (pl.col("age") * 12).alias("月龄")
)
```

### 惰性求值 vs 即时求值

**即时求值（DataFrame）：** 操作立即执行
```python
df = pl.read_csv("file.csv")  # 立即读取
result = df.filter(pl.col("age") > 25)  # 立即执行
```

**惰性求值（LazyFrame）：** 构建查询计划，优化后执行
```python
lf = pl.scan_csv("file.csv")  # 暂不读取
result = lf.filter(pl.col("age") > 25).select("name", "age")
df = result.collect()  # 执行优化后的查询
```

**适用惰性场景：**
- 处理大型数据集
- 复杂查询管道
- 仅需部分列/行时
- 性能关键场景

**惰性求值优势：**
- 自动查询优化
- 谓词下推
- 投影下推
- 并行执行

详细概念请加载`references/core_concepts.md`。

## 常用操作

### 列选择
选择并操作列：
```python
# 选择特定列
df.select("name", "age")

# 表达式选择
df.select(
    pl.col("name"),
    (pl.col("age") * 2).alias("双倍年龄")
)

# 按模式匹配列
df.select(pl.col("^.*_id$"))
```

### 行过滤
按条件过滤行：
```python
# 单条件过滤
df.filter(pl.col("age") > 25)

# 多条件（比&更清晰）
df.filter(
    pl.col("age") > 25,
    pl.col("city") == "NY"
)

# 复杂条件
df.filter(
    (pl.col("age") > 25) | (pl.col("city") == "LA")
)
```

### 列扩展
保留现有列的同时添加/修改列：
```python
# 添加新列
df.with_columns(
    age_plus_10=pl.col("age") + 10,
    name_upper=pl.col("name").str.to_uppercase()
)

# 并行计算（所有列同时计算）
df.with_columns(
    pl.col("value") * 10,
    pl.col("value") * 100,
)
```

### 分组聚合
分组数据并计算聚合：
```python
# 基础分组
df.group_by("city").agg(
    pl.col("age").mean().alias("平均年龄"),
    pl.len().alias("计数")
)

# 多分组键
df.group_by("city", "department").agg(
    pl.col("salary").sum()
)

# 条件聚合
df.group_by("city").agg(
    (pl.col("age") > 30).sum().alias("超30岁人数")
)
```

详细操作模式请加载`references/operations.md`。

## 聚合与窗口函数

### 聚合函数
`group_by`上下文中的常用聚合：
- `pl.len()` - 行计数
- `pl.col("x").sum()` - 求和
- `pl.col("x").mean()` - 平均值
- `pl.col("x").min()` / `pl.col("x").max()` - 极值
- `pl.first()` / `pl.last()` - 首尾值

### 窗口函数（`over()`）
保留行数的聚合操作：
```python
# 为每行添加分组统计
df.with_columns(
    avg_age_by_city=pl.col("age").mean().over("city"),
    rank_in_city=pl.col("salary").rank().over("city")
)

# 多分组列
df.with_columns(
    group_avg=pl.col("value").mean().over("category", "region")
)
```

**映射策略：**
- `group_to_rows`（默认）：保留原始行序
- `explode`：更快但行分组排列
- `join`：创建列表列

## 数据读写

### 支持格式
支持读写：
- CSV, Parquet, JSON, Excel
- 数据库（通过连接器）
- 云存储（S3/Azure/GCS）
- Google BigQuery
- 多文件/分区文件

### 常用读写操作

**CSV：**
```python
# 即时模式
df = pl.read_csv("file.csv")
df.write_csv("output.csv")

# 惰性模式（大文件首选）
lf = pl.scan_csv("file.csv")
result = lf.filter(...).select(...).collect()
```

**Parquet（性能推荐）：**
```python
df = pl.read_parquet("file.parquet")
df.write_parquet("output.parquet")
```

**JSON：**
```python
df = pl.read_json("file.json")
df.write_json("output.json")
```

完整I/O文档请加载`references/io_guide.md`。

## 数据转换

### 连接
合并DataFrame：
```python
# 内连接
df1.join(df2, on="id", how="inner")

# 左连接
df1.join(df2, on="id", how="left")

# 异名列连接
df1.join(df2, left_on="user_id", right_on="id")
```

### 拼接
堆叠DataFrame：
```python
# 纵向（行堆叠）
pl.concat([df1, df2], how="vertical")

# 横向（列扩展）
pl.concat([df1, df2], how="horizontal")

# 对角线（模式合并）
pl.concat([df1, df2], how="diagonal")
```

### 透视与逆透视
重塑数据：
```python
# 透视（宽格式）
df.pivot(values="sales", index="date", columns="product")

# 逆透视（长格式）
df.unpivot(index="id", on=["col1", "col2"])
```

详细转换示例请加载`references/transformations.md`。

## pandas迁移指南

Polars在保持API简洁的同时显著提升性能。主要差异：

### 概念差异
- **无索引**：仅使用整数位置
- **严格类型**：无隐式类型转换
- **惰性求值**：通过LazyFrame实现
- **默认并行**：操作自动并行化

### 操作对照表

| 操作 | pandas | Polars |
|-----------|--------|--------|
| 选择列 | `df["col"]` | `df.select("col")` |
| 过滤 | `df[df["col"] > 10]` | `df.filter(pl.col("col") > 10)` |
| 添加列 | `df.assign(x=...)` | `df.with_columns(x=...)` |
| 分组 | `df.groupby("col").agg(...)` | `df.group_by("col").agg(...)` |
| 窗口 | `df.groupby("col").transform(...)` | `df.with_columns(...).over("col")` |

### 关键语法模式

**pandas顺序执行（较慢）：**
```python
df.assign(
    col_a=lambda df_: df_.value * 10,
    col_b=lambda df_: df_.value * 100
)
```

**Polars并行执行（较快）：**
```python
df.with_columns(
    col_a=pl.col("value") * 10,
    col_b=pl.col("value") * 100,
)
```

完整迁移指南请加载`references/pandas_migration.md`。

## 最佳实践

### 性能优化

1. **大数据集使用惰性求值：**
   ```python
   lf = pl.scan_csv("large.csv")  # 避免read_csv
   result = lf.filter(...).select(...).collect()
   ```

2. **避免热点路径使用Python函数：**
   - 优先使用表达式API实现并行
   - 仅在必要时使用`.map_elements()`
   - 首选原生Polars操作

3. **超大数据使用流式处理：**
   ```python
   lf.collect(streaming=True)
   ```

4. **尽早选择所需列：**
   ```python
   # 优：先选列
   lf.select("col1", "col2").filter(...)

   # 劣：先过滤全列
   lf.filter(...).select("col1", "col2")
   ```

5. **使用合适数据类型：**
   - 低基数字符串用Categorical
   - 选择合适整数类型（i32 vs i64）
   - 时间数据用Date类型

### 表达式模式

**条件操作：**
```python
pl.when(condition).then(value).otherwise(other_value)
```

**跨列操作：**
```python
df.select(pl.col("^.*_value$") * 2)  # 正则匹配
```

**空值处理：**
```python
pl.col("x").fill_null(0)
pl.col("x").is_null()
pl.col("x").drop_nulls()
```

更多实践请加载`references/best_practices.md`。

## 资源

本技能包含完整参考文档：

### references/
- `core_concepts.md` - 表达式/惰性求值/类型系统详解
- `operations.md` - 常用操作完整指南及示例
- `pandas_migration.md` - pandas到Polars迁移手册
- `io_guide.md` - 所有支持格式的读写操作
- `transformations.md` - 连接/拼接/透视/重塑操作
- `best_practices.md` - 性能优化技巧与常用模式

用户需要特定主题详细信息时加载对应参考文档。
