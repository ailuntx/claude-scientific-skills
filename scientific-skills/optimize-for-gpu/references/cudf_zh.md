# cuDF 参考指南

cuDF 是一个 GPU 数据帧库，提供类似 pandas 的 API，可在 GPU 上完全实现表格数据的加载、连接、聚合、过滤和操作。它是 NVIDIA RAPIDS 生态系统的一部分，基于 Apache Arrow 列式内存格式构建。

> **完整文档：** https://docs.rapids.ai/api/cudf/stable/

## 目录

1. [安装与设置](#installation-and-setup)
2. [两种使用模式](#two-usage-modes)
3. [cudf.pandas 加速模式](#cudfpandas-accelerator-mode)
4. [核心 API：DataFrame 与 Series](#core-api)
5. [IO 操作](#io-operations)
6. [GroupBy 分组操作](#groupby-operations)
7. [字符串操作](#string-operations)
8. [用户定义函数 (UDFs)](#user-defined-functions)
9. [缺失数据处理](#missing-data-handling)
10. [数据类型](#data-types)
11. [内存管理](#memory-management)
12. [互操作性](#interoperability)
13. [Dask-cuDF 多 GPU 支持](#multi-gpu-with-dask-cudf)
14. [性能优化](#performance-optimization)
15. [与 pandas 的主要差异](#key-differences-from-pandas)
16. [常见迁移模式](#common-migration-patterns)

---

## 安装与设置

在所有安装说明、文档字符串、注释和错误信息中始终使用 `uv add`（切勿使用 `pip install` 或 `conda install`）。

```bash
uv add --extra-index-url=https://pypi.nvidia.com cudf-cu12    # 适用于 CUDA 12.x
```

验证安装：
```python
import cudf
print(cudf.Series([1, 2, 3]))  # 应打印 GPU 序列
```

---

## 两种使用模式

cuDF 提供两种加速 pandas 代码的方式：

### 1. cudf.pandas（零代码修改）
直接替换的加速方案，自动加速 pandas。对不支持的操作回退到 CPU。适用于：快速加速现有代码、混合代码库、原型开发。

### 2. 直接 cuDF API
将 `import pandas` 替换为 `import cudf`。可获得最大性能且无代理开销，但需适配 cuDF API（与 pandas 存在行为差异）。适用于：生产流水线、追求极致性能、新建 GPU 优先的代码。

---

## cudf.pandas 加速模式

从 pandas 迁移到 GPU 的最快路径——无需修改代码。

### 激活方式

```python
# Jupyter/IPython（必须在导入 pandas 前执行）
%load_ext cudf.pandas
import pandas as pd  # 现在已启用 GPU 加速

# 命令行
# python -m cudf.pandas your_script.py
# python -m cudf.pandas --profile your_script.py  # 带性能分析

# 编程方式
import cudf.pandas
cudf.pandas.install()
import pandas as pd  # 现在已启用 GPU 加速
```

**关键提示：** 若会话中已导入 pandas，必须重启内核/进程。

### 工作原理

- `import pandas` 返回代理模块，封装 cuDF 和 pandas
- 所有操作优先在 GPU (cuDF) 尝试，失败时自动回退到 CPU (pandas)
- 仅在必要时进行 GPU 与 CPU 间的数据传输
- 默认使用托管内存——可处理超过 GPU 内存的数据集
- 当前通过 **pandas 187,000+ 单元测试的 93%**

### 分析 GPU 与 CPU 执行

```python
%%cudf.pandas.profile        # 按单元格显示 GPU/CPU 操作分解
%%cudf.pandas.line_profile   # 按行显示 GPU/CPU 耗时
```

### 访问底层对象

```python
proxy_df.as_gpu_object()  # 获取 cuDF DataFrame 对象
proxy_df.as_cpu_object()  # 获取 pandas DataFrame 对象
```

注意：提取底层对象后自动回退功能将失效。

### 兼容的第三方库
cuGraph, cuML, Hvplot, Holoview, Ibis, NumPy, Matplotlib, Plotly, PyTorch, Seaborn, Scikit-Learn, SciPy, TensorFlow, XGBoost。

**不兼容：** Joblib。分布式任务请改用 Dask-cuDF。

### 限制

- Join 操作不保证 pandas 的行顺序（为性能优化）
- 同一会话中不能同时使用 `import cudf` 和 cudf.pandas
- Pickle 对象在常规 pandas 和 cudf.pandas 间不互通
- 代理数组继承 `numpy.ndarray`，可能导致设备到主机的急切传输
- 强制仅使用 CPU：设置 `CUDF_PANDAS_FALLBACK_MODE=1`

---

## 核心 API

### 创建 DataFrame 和 Series

```python
import cudf

# 从字典创建
df = cudf.DataFrame({"a": [1, 2, 3], "b": [4.0, 5.0, 6.0], "c": ["x", "y", "z"]})

# 从 pandas 转换
import pandas as pd
gdf = cudf.DataFrame.from_pandas(pd.DataFrame({"a": [1, 2, 3]}))
# 或
gdf = cudf.DataFrame(pandas_df)

# Series
s = cudf.Series([1, 2, 3, None, 5])

# 转回 pandas
pdf = gdf.to_pandas()
```

### 常用操作（与 pandas 相同）

```python
df.head(10)
df.tail(5)
df.describe()
df.info()
df.dtypes
df.columns
df.shape

# 选择
df["a"]                     # 列 → Series
df[["a", "b"]]             # 多列 → DataFrame
df.loc[2:5, ["a", "b"]]   # 基于标签索引
df.iloc[0:3]               # 基于整数索引

# 过滤
df[df["a"] > 2]
df.query("a > 2 and b < 6")  # 支持 @var 引用局部变量

# 排序
df.sort_values("a", ascending=False)
df.sort_index()

# 缺失数据处理
df.fillna(0)
df.dropna()
df.isna()

# 聚合
df["a"].sum()
df["a"].mean()
df["a"].std()
df["a"].value_counts()

# 转换
df["a"].clip(lower=1, upper=5)
df["a"].apply(lambda x: x * 2)  # JIT 编译

# 合并
cudf.concat([df1, df2])
df1.merge(df2, on="key")
df1.merge(df2, on="key", how="left")  # left, right, inner, outer

# Arrow 互操作（零拷贝）
arrow_table = df.to_arrow()
df = cudf.DataFrame.from_arrow(arrow_table)
```

---

## IO 操作

GPU 加速的文件读写——处理大文件时通常比 pandas 快得多。

### Parquet（推荐用于高性能场景）

```python
# 读取
df = cudf.read_parquet("data.parquet")
df = cudf.read_parquet("data.parquet", columns=["a", "b"])  # 仅读取指定列

# 写入
df.to_parquet("output.parquet")

# 元数据检查（无需加载数据）
cudf.io.parquet.read_parquet_metadata("data.parquet")

# 增量写入
writer = cudf.io.parquet.ParquetDatasetWriter("output_dir/", partition_cols=["year"])
writer.write_table(df)
writer.close()
```

### CSV

```python
df = cudf.read_csv("data.csv")
df = cudf.read_csv("data.csv", usecols=["a", "b"], dtype={"a": "int32"})
df.to_csv("output.csv", index=False)
```

### JSON

```python
df = cudf.read_json("data.json")
df = cudf.read_json("data.json", lines=True)  # JSON Lines 格式
df.to_json("output.json")
```

### ORC

```python
df = cudf.read_orc("data.orc")
df.to_orc("output.orc")
```

### 其他格式

| 格式 | 读取 | 写入 | GPU 加速 |
|------|------|------|-----------|
| Avro | `cudf.read_avro()` | N/A | 是（仅读取） |
| 文本 | `cudf.read_text()` | N/A | 是（仅读取） |
| HDF5 | `cudf.read_hdf()` | `df.to_hdf()` | 否（使用 pandas） |
| Feather | `cudf.read_feather()` | `df.to_feather()` | 否（使用 pandas） |

**优先选择 Parquet 而非 CSV**——列式格式在 GPU 上读取更快，支持谓词下推，且压缩效果好。

---

## GroupBy 分组操作

### 基础 GroupBy

```python
df.groupby("category").sum()
df.groupby(["category", "subcategory"]).mean()
df.groupby("category").agg({"value": "sum", "count": "max"})
df.groupby("category").agg({"value": ["sum", "min", "max"], "count": "mean"})
```

### 支持的聚合操作

**通用：** `count`, `size`, `nunique`, `nth`, `collect`, `unique`
**数值：** `sum`, `mean`, `var`, `std`, `median`, `idxmin`, `idxmax`, `min`, `max`, `quantile`
**特殊：** `corr`, `cov`

### GroupBy 转换

```python
df.groupby("category").transform("max")  # 将结果广播到组内所有行
```

### GroupBy 应用

```python
df.groupby("category").apply(lambda x: x.max() - x.min())
```

**警告：** Apply 按组顺序执行函数——小分组过多时可能变慢。尽可能使用向量化聚合。

### JIT 编译 GroupBy（用户定义聚合）

```python
def custom_agg(df):
    return df["value"].max() - df["value"].min() / 2

result = df.groupby("category").apply(custom_agg, engine="jit")
```

JIT 限制：不支持空值、仅支持 int32/64 和 float32/64、不能返回新列。

### 重要：排序行为

cuDF 默认使用 `sort=False`（不同于 pandas 默认排序）。要匹配 pandas 行为：
```python
df.groupby("category", sort=True).sum()
# 或全局设置：
cudf.set_option("mode.pandas_compatible", True)
```

---

## 字符串操作

cuDF 通过 `.str` 访问器提供 GPU 加速的字符串操作——API 与 pandas 完全相同。

```python
s = cudf.Series(["Hello World", "foo bar", "RAPIDS GPU", None])

# 大小写转换
s.str.lower()
s.str.upper()
s.str.title()
s.str.capitalize()

# 模式匹配
s.str.contains("World")
s.str.startswith("Hello")
s.str.endswith("GPU")
s.str.match(r"^[A-Z]")

# 提取与替换
s.str.extract(r"(\w+)\s(\w+)")
s.str.replace("World", "GPU")
s.str.slice(0, 5)

# 拆分与连接
s.str.split(" ")
s.str.cat(sep=", ")

# 信息获取
s.str.len()
s.str.isalpha()
s.str.isdigit()

# cuDF 专属操作（pandas 不支持）
s.str.normalize_spaces()   # 压缩空白符
s.str.tokenize()           # 分词
s.str.ngrams(2)            # 生成 n-gram
s.str.edit_distance(other) # 编辑距离（Levenshtein）
s.str.url_encode()
s.str.url_decode()
```

---

## 用户定义函数

### Series.apply() —— JIT 编译

```python
s = cudf.Series([1, 2, 3, 4, 5])

def square_plus_one(x):
    return x ** 2 + 1

s.apply(square_plus_one)  # 通过 Numba 编译为 GPU 内核
```

带参数：
```python
def add_constant(x, c):
    return x + c

s.apply(add_constant, args=(42,))
```

### DataFrame.apply() —— 按行操作 (axis=1)

```python
def row_func(row):
    return row["a"] + row["b"] * 2

df.apply(row_func, axis=1)  # 通过类字典语法按列名访问
```

### UDF 中的空值处理

空值自动传播：
```python
s = cudf.Series([1, cudf.NA, 3])
def f(x):
    return x + 1
s.apply(f)  # 返回 [2, <NA>, 4]
```

显式空值检查：
```python
def f(x):
    if x is cudf.NA:
        return 0
    return x + 1
```

### 字符串 UDF

UDF 内支持的字符串操作：`==`, `!=`, `>=`, `<=`, `startswith()`, `endswith()`, `find()`, `rfind()`, `count()`, `in`, `strip/lstrip/rstrip()`, `upper/lower()`, `replace()`, `+`（连接）, `len()`, 布尔检查。

创建中间字符串的字符串 UDF 需分配堆内存：
```python
from cudf.core.udf.utils import set_malloc_heap_size
set_malloc_heap_size(int(2e9))  # 2 GB
```

### 滚动窗口 UDF

```python
import math

s = cudf.Series([16, 25, 36, 49, 64, 81], dtype="float64")

def max_sqrt(window):
    result = 0
    for val in window:
        result = max(result, math.sqrt(val))
    return result

s.rolling(window=3, min_periods=3).apply(max_sqrt)
```

**限制：** 滚动窗口 UDF 不支持空值。

### 在 cuDF 列上编写自定义 Numba CUDA 内核

为获得最大控制权，可直接操作 cuDF 列编写 CUDA 内核：

```python
from numba import cuda

@cuda.jit
def gpu_multiply(in_col, out_col, multiplier):
    i = cuda.grid(1)
    if i < in_col.size:
        out_col[i] = in_col[i] * multiplier

df["result"] = 0.0
gpu_multiply.forall(len(df))(df["a"], df["result"], 10.0)
```

### UDF 限制

- 仅数值非十进制类型完全支持；字符串部分支持
- 不支持 `**kwargs`
- UDF 中未实现位运算
- GroupBy JIT：不支持空值、仅支持 int32/64 和 float32/64、不能返回新列
- 滚动窗口 UDF：不支持空值

---

## 缺失数据处理

- 缺失值表示为 `<NA>`（非 NaN）—— cuDF 使用独立空值掩码，而非 NaN 标记
- 所有数据类型均可为空（包括整数——缺失整数值不会强制转为浮点型）
- 插入整数列的 `np.nan` 会转为 `<NA>` 而不转为浮点型

```python
s = cudf.Series([1, None, 3, None, 5])

s.isna()                # 布尔掩码
s.notna()
s.fillna(0)             # 标量填充
s.fillna({"a": 0, "b": 1})  # 按列字典填充
s.dropna()

# 聚合操作默认跳过 NA
s.sum()                 # skipna=True（默认）
s.sum(skipna=False)     # 传播 NA

# GroupBy 默认排除 NA 分组
df.groupby("a", dropna=False).sum()  # 包含 NA 分组
```

---

## 数据类型

| 类别 | 类型 |
|------|------|
| 整数 | `int8`, `int16`, `int32`, `int64`, `uint32`, `uint64` |
| 浮点 | `float32`, `float64` |
| 日期时间 | `datetime64[s/ms/us/ns]` |
| 时间差 | `timedelta[s/ms/us/ns]` |
| 分类 | `CategoricalDtype` |
| 字符串 | `object` / `string` |
| 十进制 | `Decimal32Dtype`, `Decimal64Dtype`, `Decimal128Dtype` |
| 列表 | `ListDtype`（嵌套列表） |
| 结构体 | `StructDtype`（类字典结构） |

所有类型均可为空。列表列有 `.list` 访问器（`get()`, `len()`, `contains

### RMM（RAPIDS 内存管理器）

cuDF 使用 RMM 进行 GPU 内存分配。根据工作负载配置：

```python
import rmm

# 池分配器（生产环境推荐——避免每次分配时的 cudaMalloc 开销）
pool = rmm.mr.PoolMemoryResource(
    rmm.mr.CudaMemoryResource(),
    initial_pool_size="1GiB",
    maximum_pool_size="4GiB"
)
rmm.mr.set_current_device_resource(pool)

# 托管内存（支持大于 GPU 内存的数据集）
rmm.mr.set_current_device_resource(rmm.mr.ManagedMemoryResource())

# 托管内存 + 池（两者优势结合）
pool = rmm.mr.PoolMemoryResource(
    rmm.mr.ManagedMemoryResource(),
    initial_pool_size="1GiB"
)
rmm.mr.set_current_device_resource(pool)
```

### 对齐 CuPy 和 Numba 的 RMM 配置

当 cuDF 与 CuPy 或 Numba 联用时，需统一内存分配器以避免内存碎片：

```python
# CuPy
from rmm.allocators.cupy import rmm_cupy_allocator
import cupy
cupy.cuda.set_allocator(rmm_cupy_allocator)

# Numba
from rmm.allocators.numba import RMMNumbaManager
from numba import cuda
cuda.set_memory_manager(RMMNumbaManager)
```

### 写时复制

```python
cudf.set_option("copy_on_write", True)
# 或：export CUDF_COPY_ON_WRITE=1
```

切片、`.head()`、浅拷贝及视图生成方法共享内存，直到其中一个被修改。显著减少含派生 DataFrame 工作流的内存占用。

### 内存分析

```python
rmm.statistics.enable_statistics()
stats = rmm.statistics.get_statistics()
# 返回：current_bytes, current_count, peak_bytes, peak_count, total_bytes, total_count
```

---

## 互操作性

### CuPy（零拷贝）

```python
import cupy as cp

# cuDF → CuPy
arr = df.to_cupy()             # DataFrame → 二维 CuPy 数组
arr = cp.asarray(df["col"])    # Series → 一维 CuPy 数组
arr = df["col"].values         # Series → 一维 CuPy 数组

# CuPy → cuDF
df = cudf.DataFrame(cupy_2d_array)
s = cudf.Series(cupy_1d_array)

# 通过 DLPack
df = cudf.from_dlpack(cupy_array.__dlpack__())
```

### Arrow（零拷贝）

```python
arrow_table = df.to_arrow()
df = cudf.DataFrame.from_arrow(arrow_table)
```

### RAPIDS 生态系统

- **cuML:** 直接接受 cuDF DataFrame 进行机器学习流程。
- **cuGraph:** 接受 cuDF DataFrame 进行图分析。
- **Dask-cuDF:** 分布式 GPU DataFrame（见下文）。

### CUDA 数组接口

cuDF Series 暴露 `__cuda_array_interface__`，支持与兼容库（CuPy、Numba、PyTorch 等）零拷贝共享。

---

## 使用 Dask-cuDF 实现多 GPU

适用于超过单 GPU 内存的数据集或多 GPU 并行场景：

```python
import dask_cudf
from dask.distributed import Client
from dask_cuda import LocalCUDACluster

# 每个 GPU 一个工作进程
cluster = LocalCUDACluster()
client = Client(cluster)

# 从文件读取
ddf = dask_cudf.read_csv("path/*.csv")
ddf = dask_cudf.read_parquet("path/")

# 从 cuDF DataFrame 创建
ddf = dask_cudf.from_cudf(df, npartitions=16)

# 操作（惰性执行——需调用 .compute() 触发）
result = ddf.groupby("a").sum().compute()

# 持久化到 GPU 内存供重复访问
ddf = ddf.persist()
```

与 cuDF 主要差异：不支持 `.iloc`，必须调用 `.compute()` 实现物化，未实现转置操作。

---

## 性能优化

1. **从 cudf.pandas 开始**——零代码修改，自动 GPU/CPU 回退。
2. **切换至 cuDF 原生 API 获得最大性能**——避免代理开销和回退复制成本。
3. **优先选择 Parquet 而非 CSV**——列式存储格式，GPU 读取更快，谓词下推，压缩率更高。
4. **通过 RMM 使用池分配器**——避免每次分配的 `cudaMalloc` 开销。
5. **启用写时复制**——`cudf.set_option("copy_on_write", True)` 减少切片和视图的内存占用。
6. **将数据重塑为长格式**（更多行，更少列）——GPU 按行并行化。
7. **禁止迭代**——仅使用向量化操作。`for row in df.iterrows()` 会抵消 GPU 加速优势。
8. **最小数据集规模**：GPU 在 **10,000-100,000+ 行**时性能显著，小数据集 CPU 可能更快。
9. **使用向量化字符串操作**（`.str.` 访问器）替代逐行字符串 UDF。
10. **对 cuDF 原生不支持的逐行数学运算使用 CuPy**。
11. **对复杂逐元素操作使用 Numba CUDA 内核**。
12. **所有 RAPIDS 库统一使用相同 RMM 分配器**——避免内存碎片。
13. **分布式工作流中**，使用 Dask-cuDF 的 `persist()` 保持数据在 GPU 内存。

---

## 与 pandas 的主要差异

1. **结果顺序默认非确定性**（groupby、连接等）。需显式设置 `sort=True` 或 `cudf.set_option("mode.pandas_compatible", True)`。
2. **所有类型可为空**。缺失值用 `<NA>` 表示而非 NaN。含缺失值的整型列保持整型（不强制转为浮点）。
3. **不支持迭代**。`for val in series` 不可用，必须迭代时可转为 pandas。
4. **列名必须唯一**。禁止重复列名。
5. **不支持任意 Python 对象**。`object` 类型仅存储字符串。
6. **`.apply()` 使用 Numba JIT**——UDF 仅支持 Python 子集（禁用任意 Python 对象和外部库调用）。
7. **浮点结果可能因 GPU 并行顺序存在细微差异**，建议使用容差比较。
8. **GroupBy 默认 `sort=False`**（pandas 默认为 `sort=True`）。
9. **不支持 pandas 的 ExtensionDtype**。

---

## 常见迁移模式

### 模式 1：零修改（cudf.pandas）
```python
%load_ext cudf.pandas
import pandas as pd
# 其余代码完全不变
```

### 模式 2：直接导入替换
```python
# 迁移前
import pandas as pd
df = pd.read_csv("data.csv")
result = df.groupby("col").mean()

# 迁移后
import cudf
df = cudf.read_csv("data.csv")
result = df.groupby("col").mean()
```

### 模式 3：用向量化操作替代迭代
```python
# 迁移前（pandas——CPU 上也慢）
for idx, row in df.iterrows():
    df.at[idx, "c"] = row["a"] + row["b"]

# 迁移后（cuDF）
df["c"] = df["a"] + df["b"]
```

### 模式 4：用向量化替代 apply()
```python
# 迁移前
df["result"] = df.apply(lambda row: row["a"] ** 2 + row["b"], axis=1)

# 迁移后（向量化——显著加速）
df["result"] = df["a"] ** 2 + df["b"]
```

### 模式 5：GPU 处理，边界转 CPU
```python
# GPU 加载处理
gdf = cudf.read_parquet("data.parquet")
result = gdf.groupby("key").agg({"val": "sum"})

# 仅在需要时转 pandas（绘图/导出等）
pdf = result.to_pandas()
pdf.plot()
```

### 模式 6：用 CuPy 处理不支持的数学运算
```python
import cupy as cp

# 将数据转 CuPy 处理 cuDF 不支持的运算
arr = df[["x", "y", "z"]].to_cupy()
norms = cp.linalg.norm(arr, axis=1)
df["norm"] = cudf.Series(norms)
```

---

## 配置

```python
cudf.set_option("copy_on_write", True)            # 启用写时复制
cudf.set_option("mode.pandas_compatible", True)    # 匹配 pandas 行为
cudf.describe_option()                             # 列出所有选项
```

| 环境变量 | 用途 |
|---------------------|---------|
| `CUDF_COPY_ON_WRITE=1` | 启用写时复制 |
| `CUDF_PANDAS_RMM_MODE` | 控制 cudf.pandas 的内存分配器 |
| `CUDF_PANDAS_FALLBACK_MODE=1` | 强制 cudf.pandas 仅使用 CPU |
