# Polars 数据输入输出指南

使用 Polars 读写多种格式数据的综合指南。

## CSV 文件

### 读取 CSV

**即时模式（加载到内存）：**
```python
import polars as pl

# 基础读取
df = pl.read_csv("data.csv")

# 带选项读取
df = pl.read_csv(
    "data.csv",
    separator=",",
    has_header=True,
    columns=["col1", "col2"],  # 选择特定列
    n_rows=1000,  # 仅读取前1000行
    skip_rows=10,  # 跳过前10行
    dtypes={"col1": pl.Int64, "col2": pl.Utf8},  # 指定类型
    null_values=["NA", "null", ""],  # 定义空值
    encoding="utf-8",
    ignore_errors=False
)
```

**惰性模式（扫描不加载 - 推荐用于大文件）：**
```python
# 扫描 CSV（构建查询计划）
lf = pl.scan_csv("data.csv")

# 应用操作
result = lf.filter(pl.col("age") > 25).select("name", "age")

# 执行并加载
df = result.collect()
```

### 写入 CSV

```python
# 基础写入
df.write_csv("output.csv")

# 带选项写入
df.write_csv(
    "output.csv",
    separator=",",
    include_header=True,
    null_value="",  # 空值表示方式
    quote_char='"',
    line_terminator="\n"
)
```

### 多 CSV 文件处理

**读取多个文件：**
```python
# 读取目录下所有 CSV
lf = pl.scan_csv("data/*.csv")

# 读取特定文件
lf = pl.scan_csv(["file1.csv", "file2.csv", "file3.csv"])
```

## Parquet 文件

Parquet 是推荐的高性能压缩格式。

### 读取 Parquet

**即时模式：**
```python
df = pl.read_parquet("data.parquet")

# 带选项读取
df = pl.read_parquet(
    "data.parquet",
    columns=["col1", "col2"],  # 选择特定列
    n_rows=1000,  # 读取前 N 行
    parallel="auto"  # 控制并行化
)
```

**惰性模式（推荐）：**
```python
lf = pl.scan_parquet("data.parquet")

# 自动谓词和投影下推
result = lf.filter(pl.col("age") > 25).select("name", "age").collect()
```

### 写入 Parquet

```python
# 基础写入
df.write_parquet("output.parquet")

# 带压缩写入
df.write_parquet(
    "output.parquet",
    compression="snappy",  # 选项："snappy", "gzip", "brotli", "lz4", "zstd"
    statistics=True,  # 写入统计信息（启用谓词下推）
    use_pyarrow=False  # 使用 Rust 写入器（更快）
)
```

### 分区式 Parquet（Hive 风格）

**分区写入：**
```python
# 分区写入
df.write_parquet(
    "output_dir",
    partition_by=["year", "month"]  # 创建目录结构
)
# 生成：output_dir/year=2023/month=01/data.parquet
```

**读取分区数据：**
```python
lf = pl.scan_parquet("output_dir/**/*.parquet")

# 自动添加 Hive 分区列
result = lf.filter(pl.col("year") == 2023).collect()
```

## JSON 文件

### 读取 JSON

**NDJSON（换行分隔 JSON）- 推荐：**
```python
df = pl.read_ndjson("data.ndjson")

# 惰性模式
lf = pl.scan_ndjson("data.ndjson")
```

**标准 JSON：**
```python
df = pl.read_json("data.json")

# 从 JSON 字符串读取
df = pl.read_json('{"col1": [1, 2], "col2": ["a", "b"]}')
```

### 写入 JSON

```python
# 写入 NDJSON
df.write_ndjson("output.ndjson")

# 写入标准 JSON
df.write_json("output.json")

# 格式化输出
df.write_json("output.json", pretty=True, row_oriented=False)
```

## Excel 文件

### 读取 Excel

```python
# 读取首张工作表
df = pl.read_excel("data.xlsx")

# 指定工作表
df = pl.read_excel("data.xlsx", sheet_name="Sheet1")
# 或按索引
df = pl.read_excel("data.xlsx", sheet_id=0)

# 带选项读取
df = pl.read_excel(
    "data.xlsx",
    sheet_name="Sheet1",
    columns=["A", "B", "C"],  # Excel 列标识
    n_rows=100,
    skip_rows=5,
    has_header=True
)
```

### 写入 Excel

```python
# 写入 Excel
df.write_excel("output.xlsx")

# 多工作表写入
with pl.ExcelWriter("output.xlsx") as writer:
    df1.write_excel(writer, worksheet="Sheet1")
    df2.write_excel(writer, worksheet="Sheet2")
```

## 数据库连接

### 从数据库读取

```python
import polars as pl

# 读取整表
df = pl.read_database("SELECT * FROM users", connection_uri="postgresql://...")

# 使用 connectorx 提升性能
df = pl.read_database_uri(
    "SELECT * FROM users WHERE age > 25",
    uri="postgresql://user:pass@localhost/db"
)
```

### 写入数据库

```python
# 使用 SQLAlchemy
from sqlalchemy import create_engine

engine = create_engine("postgresql://user:pass@localhost/db")
df.write_database("table_name", connection=engine)

# 带选项写入
df.write_database(
    "table_name",
    connection=engine,
    if_exists="replace",  # 或 "append", "fail"
)
```

### 常用数据库连接器

**PostgreSQL：**
```python
uri = "postgresql://username:password@localhost:5432/database"
df = pl.read_database_uri("SELECT * FROM table", uri=uri)
```

**MySQL：**
```python
uri = "mysql://username:password@localhost:3306/database"
df = pl.read_database_uri("SELECT * FROM table", uri=uri)
```

**SQLite：**
```python
uri = "sqlite:///path/to/database.db"
df = pl.read_database_uri("SELECT * FROM table", uri=uri)
```

## 云存储

### AWS S3

```python
# 从 S3 读取
df = pl.read_parquet("s3://bucket/path/to/file.parquet")
lf = pl.scan_parquet("s3://bucket/path/*.parquet")

# 写入 S3
df.write_parquet("s3://bucket/path/output.parquet")

# 带凭证访问
import os
os.environ["AWS_ACCESS_KEY_ID"] = "your_key"
os.environ["AWS_SECRET_ACCESS_KEY"] = "your_secret"
os.environ["AWS_REGION"] = "us-west-2"

df = pl.read_parquet("s3://bucket/file.parquet")
```

### Azure Blob 存储

```python
# 从 Azure 读取
df = pl.read_parquet("az://container/path/file.parquet")

# 写入 Azure
df.write_parquet("az://container/path/output.parquet")

# 带凭证访问
os.environ["AZURE_STORAGE_ACCOUNT_NAME"] = "account"
os.environ["AZURE_STORAGE_ACCOUNT_KEY"] = "key"
```

### Google 云存储 (GCS)

```python
# 从 GCS 读取
df = pl.read_parquet("gs://bucket/path/file.parquet")

# 写入 GCS
df.write_parquet("gs://bucket/path/output.parquet")

# 带凭证访问
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/path/to/credentials.json"
```

## Google BigQuery

```python
# 从 BigQuery 读取
df = pl.read_database(
    "SELECT * FROM project.dataset.table",
    connection_uri="bigquery://project"
)

# 或使用 Google Cloud SDK
from google.cloud import bigquery
client = bigquery.Client()

query = "SELECT * FROM project.dataset.table WHERE date > '2023-01-01'"
df = pl.from_pandas(client.query(query).to_dataframe())
```

## Apache Arrow

### IPC/Feather 格式

**读取：**
```python
df = pl.read_ipc("data.arrow")
lf = pl.scan_ipc("data.arrow")
```

**写入：**
```python
df.write_ipc("output.arrow")

# 压缩写入
df.write_ipc("output.arrow", compression="zstd")
```

### Arrow 流式处理

```python
# 写入流式格式
df.write_ipc("output.arrows", compression="zstd")

# 读取流式数据
df = pl.read_ipc("output.arrows")
```

### Arrow 转换

```python
import pyarrow as pa

# 从 Arrow 表转换
arrow_table = pa.table({"col": [1, 2, 3]})
df = pl.from_arrow(arrow_table)

# 转为 Arrow 表
arrow_table = df.to_arrow()
```

## 内存格式

### Python 字典

```python
# 从字典创建
df = pl.DataFrame({
    "col1": [1, 2, 3],
    "col2": ["a", "b", "c"]
})

# 转为字典
data_dict = df.to_dict()  # 列导向
data_dict = df.to_dict(as_series=False)  # 列表替代 Series
```

### NumPy 数组

```python
import numpy as np

# 从 NumPy 创建
arr = np.array([[1, 2], [3, 4], [5, 6]])
df = pl.DataFrame(arr, schema=["col1", "col2"])

# 转为 NumPy
arr = df.to_numpy()
```

### Pandas 数据框

```python
import pandas as pd

# 从 Pandas 转换
pd_df = pd.DataFrame({"col": [1, 2, 3]})
pl_df = pl.from_pandas(pd_df)

# 转为 Pandas
pd_df = pl_df.to_pandas()

# 可能时零拷贝转换
pl_df = pl.from_arrow(pd_df)
```

### 行列表

```python
# 从字典列表创建
data = [
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 30}
]
df = pl.DataFrame(data)

# 转为字典列表
rows = df.to_dicts()

# 从元组列表创建
data = [("Alice", 25), ("Bob", 30)]
df = pl.DataFrame(data, schema=["name", "age"])
```

## 大文件流式处理

处理超过内存的数据集时使用惰性流式模式：

```python
# 流式模式
lf = pl.scan_csv("very_large.csv")
result = lf.filter(pl.col("value") > 100).collect(streaming=True)

# 多文件流式处理
lf = pl.scan_parquet("data/*.parquet")
result = lf.group_by("category").agg(pl.col("value").sum()).collect(streaming=True)
```

## 最佳实践

### 格式选择

**使用 Parquet 当：**
- 需要压缩（比 CSV 小 10 倍）
- 需要快速读写
- 需保留数据类型
- 处理大型数据集
- 需要谓词下推

**使用 CSV 当：**
- 需要人类可读格式
- 对接遗留系统
- 数据量小
- 需要通用兼容性

**使用 JSON 当：**
- 处理嵌套/分层数据
- 需要 Web API 兼容
- 数据结构灵活

**使用 Arrow IPC 当：**
- 需要零拷贝数据共享
- 要求最快序列化
- 在 Arrow 兼容系统间工作

### 大文件读取技巧

```python
# 1. 始终使用惰性模式
lf = pl.scan_csv("large.csv")  # 非 read_csv

# 2. 尽早筛选和选择（下推优化）
result = (
    lf
    .select("col1", "col2", "col3")  # 仅需列
    .filter(pl.col("date") > "2023-01-01")  # 尽早筛选
    .collect()
)

# 3. 超大文件使用流式处理
result = lf.filter(...).select(...).collect(streaming=True)

# 4. 开发时仅读取所需行
df = pl.read_csv("large.csv", n_rows=10000)  # 测试采样
```

### 大文件写入技巧

```python
# 1. 使用带压缩的 Parquet
df.write_parquet("output.parquet", compression="zstd")

# 2. 超大数据集使用分区
df.write_parquet("output", partition_by=["year", "month"])

# 3. 流式写入
lf = pl.scan_csv("input.csv")
lf.sink_parquet("output.parquet")  # 流式写入
```

### 性能优化建议

```python
# 1. 读取 CSV 时指定类型
df = pl.read_csv(
    "data.csv",
    dtypes={"id": pl.Int64, "name": pl.Utf8}  # 避免类型推断
)

# 2. 选择合适的压缩算法
df.write_parquet("output.parquet", compression="snappy")  # 快速
df.write_parquet("output.parquet", compression="zstd")    # 更高压缩率

# 3. 并行读取
df = pl.read_csv("data.csv", parallel="auto")

# 4. 并行读取多文件
lf = pl.scan_parquet("data/*.parquet")  # 自动并行读取
```

## 错误处理

```python
try:
    df = pl.read_csv("data.csv")
except pl.exceptions.ComputeError as e:
    print(f"CSV 读取错误: {e}")

# 解析时忽略错误
df = pl.read_csv("messy.csv", ignore_errors=True)

# 处理缺失文件
from pathlib import Path
if Path("data.csv").exists():
    df = pl.read_csv("data.csv")
else:
    print("文件未找到")
```

## 模式管理

```python
# 从样本推断模式
schema = pl.read_csv("data.csv", n_rows=1000).schema

# 使用推断模式完整读取
df = pl.read_csv("data.csv", dtypes=schema)

# 显式定义模式
schema = {
    "id": pl.Int64,
    "name": pl.Utf8,
    "date": pl.Date,
    "value": pl.Float64
}
df = pl.read_csv("data.csv", dtypes=schema)
```
