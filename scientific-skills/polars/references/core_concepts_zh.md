# Polars 核心概念

## 表达式

表达式是 Polars API 的基础。它们是可组合的单元，用于描述数据转换而不会立即执行。

### 什么是表达式？

表达式描述了对数据的转换操作，仅在特定上下文中才会具体化（执行）：
- `select()` - 选择并转换列
- `with_columns()` - 添加或修改列
- `filter()` - 过滤行
- `group_by().agg()` - 聚合数据

### 表达式语法

**基础列引用：**
```python
pl.col("column_name")
```

**计算表达式：**
```python
# 算术运算
pl.col("height") * 2
pl.col("price") + pl.col("tax")

# 使用别名
(pl.col("weight") / (pl.col("height") ** 2)).alias("bmi")

# 方法链式调用
pl.col("name").str.to_uppercase().str.slice(0, 3)
```

### 表达式上下文

**选择上下文：**
```python
df.select(
    "name",  # 简单列名
    pl.col("age"),  # 表达式
    (pl.col("age") * 12).alias("age_in_months")  # 计算表达式
)
```

**列添加上下文：**
```python
df.with_columns(
    age_doubled=pl.col("age") * 2,
    name_upper=pl.col("name").str.to_uppercase()
)
```

**过滤上下文：**
```python
df.filter(
    pl.col("age") > 25,
    pl.col("city").is_in(["NY", "LA", "SF"])
)
```

**分组上下文：**
```python
df.group_by("department").agg(
    pl.col("salary").mean(),
    pl.col("employee_id").count()
)
```

### 表达式扩展

批量操作多列：

**所有列：**
```python
df.select(pl.all() * 2)
```

**模式匹配：**
```python
# 所有以"_value"结尾的列
df.select(pl.col("^.*_value$") * 100)

# 所有数值列
df.select(pl.col(pl.NUMERIC_DTYPES) + 1)
```

**排除模式：**
```python
df.select(pl.all().exclude("id", "name"))
```

### 表达式组合

表达式可存储并复用：

```python
# 定义可复用表达式
age_expression = pl.col("age") * 12
name_expression = pl.col("name").str.to_uppercase()

# 在多个上下文中使用
df.select(age_expression, name_expression)
df.with_columns(age_months=age_expression)
```

## 数据类型

Polars 拥有基于 Apache Arrow 的严格类型系统。

### 核心数据类型

**数值类型：**
- `Int8`, `Int16`, `Int32`, `Int64` - 有符号整数
- `UInt8`, `UInt16`, `UInt32`, `UInt64` - 无符号整数
- `Float32`, `Float64` - 浮点数

**文本类型：**
- `Utf8` / `String` - UTF-8 编码字符串
- `Categorical` - 分类字符串（低基数）
- `Enum` - 固定字符串值集合

**时间类型：**
- `Date` - 日历日期（不含时间）
- `Datetime` - 日期时间（含可选时区）
- `Time` - 一天中的时间
- `Duration` - 时间间隔/差值

**布尔类型：**
- `Boolean` - 真/假值

**嵌套类型：**
- `List` - 变长列表
- `Array` - 定长数组
- `Struct` - 嵌套记录结构

**其他类型：**
- `Binary` - 二进制数据
- `Object` - Python 对象（生产环境避免使用）
- `Null` - 空类型

### 类型转换

显式转换类型：

```python
# 转换为不同类型
df.select(
    pl.col("age").cast(pl.Float64),
    pl.col("date_string").str.strptime(pl.Date, "%Y-%m-%d"),
    pl.col("id").cast(pl.Utf8)
)
```

### 空值处理

Polars 在所有类型中采用一致的空值处理：

**检查空值：**
```python
df.filter(pl.col("value").is_null())
df.filter(pl.col("value").is_not_null())
```

**填充空值：**
```python
pl.col("value").fill_null(0)
pl.col("value").fill_null(strategy="forward")
pl.col("value").fill_null(strategy="backward")
pl.col("value").fill_null(strategy="mean")
```

**删除空值：**
```python
df.drop_nulls()  # 删除含空值的行
df.drop_nulls(subset=["col1", "col2"])  # 删除指定列含空值的行
```

### 分类数据

对低基数（重复值）字符串列使用分类类型：

```python
# 转换为分类类型
df.with_columns(
    pl.col("category").cast(pl.Categorical)
)

# 优势：
# - 减少内存占用
# - 加速分组和连接操作
# - 保留顺序信息
```

## 惰性求值与即时求值

Polars 支持两种执行模式：即时（DataFrame）和惰性（LazyFrame）。

### 即时求值（DataFrame）

操作立即执行：

```python
import polars as pl

# DataFrame 操作立即执行
df = pl.read_csv("data.csv")  # 立即读取文件
result = df.filter(pl.col("age") > 25)  # 立即过滤
final = result.select("name", "age")  # 立即选择
```

**适用场景：**
- 可放入内存的小型数据集
- 在笔记本中交互式探索
- 简单的一次性操作
- 需要即时反馈

### 惰性求值（LazyFrame）

操作构建查询计划，优化后执行：

```python
import polars as pl

# LazyFrame 操作构建查询计划
lf = pl.scan_csv("data.csv")  # 尚未读取
lf2 = lf.filter(pl.col("age") > 25)  # 添加到计划
lf3 = lf2.select("name", "age")  # 添加到计划
df = lf3.collect()  # 此时执行优化后的计划
```

**适用场景：**
- 大型数据集
- 复杂查询流程
- 仅需数据子集
- 性能要求高
- 需要流式处理

### 查询优化

Polars 自动优化惰性查询：

**谓词下推：**
尽可能将过滤操作推送到数据源：
```python
# 从 CSV 中仅读取 age > 25 的行
lf = pl.scan_csv("data.csv")
result = lf.filter(pl.col("age") > 25).collect()
```

**投影下推：**
仅从数据源读取所需列：
```python
# 从 CSV 中仅读取 "name" 和 "age" 列
lf = pl.scan_csv("data.csv")
result = lf.select("name", "age").collect()
```

**查询计划检查：**
```python
# 查看优化后的查询计划
lf = pl.scan_csv("data.csv")
result = lf.filter(pl.col("age") > 25).select("name", "age")
print(result.explain())  # 显示优化计划
```

### 流式模式

处理超出内存容量的数据：

```python
# 为超大数据集启用流式处理
lf = pl.scan_csv("very_large.csv")
result = lf.filter(pl.col("age") > 25).collect(streaming=True)
```

**流式优势：**
- 处理超过 RAM 的数据
- 降低峰值内存使用
- 基于分块处理
- 自动内存管理

**流式限制：**
- 并非所有操作都支持流式
- 小数据可能较慢
- 部分操作需物化整个数据集

### 即时与惰性转换

**即时转惰性：**
```python
df = pl.read_csv("data.csv")
lf = df.lazy()  # 转为 LazyFrame
```

**惰性转即时：**
```python
lf = pl.scan_csv("data.csv")
df = lf.collect()  # 执行并返回 DataFrame
```

## 内存格式

Polars 使用 Apache Arrow 列式内存格式：

**优势：**
- 与其他 Arrow 库零拷贝共享数据
- 高效的列式操作
- SIMD 向量化
- 减少内存开销
- 快速序列化

**影响：**
- 数据按列存储而非按行
- 列操作极快
- 随机行访问慢于 pandas
- 最适合分析型工作负载

## 并行处理

Polars 使用 Rust 并发自动并行化操作：

**可并行化操作：**
- 分组内的聚合
- 窗口函数
- 多数表达式求值
- 文件读取（多文件）
- 连接操作

**需避免并行化的操作：**
- Python 用户定义函数（UDF）
- `.map_elements()` 中的 lambda 函数
- 顺序执行的 `.pipe()` 链

**最佳实践：**
```python
# 良好：保持在表达式 API 内（可并行）
df.with_columns(
    pl.col("value") * 10,
    pl.col("value").log(),
    pl.col("value").sqrt()
)

# 不佳：使用 Python 函数（顺序执行）
df.with_columns(
    pl.col("value").map_elements(lambda x: x * 10)
)
```

## 严格类型系统

Polars 强制执行严格类型：

**无隐式转换：**
```python
# 这将报错——不能混合类型
# df.with_columns(pl.col("int_col") + "string")

# 必须显式转换
df.with_columns(
    pl.col("int_col").cast(pl.Utf8) + "_suffix"
)
```

**优势：**
- 防止静默错误
- 行为可预测
- 更优性能
- 代码意图更清晰

**整数空值：**
与 pandas 不同，整数列可包含空值而无需转为浮点：
```python
# pandas：含空值的整数列转为浮点
# polars：含空值的整数列保持整数类型（含空值）
df = pl.DataFrame({"int_col": [1, 2, None, 4]})
# 数据类型：Int64（非 Float64）
```
