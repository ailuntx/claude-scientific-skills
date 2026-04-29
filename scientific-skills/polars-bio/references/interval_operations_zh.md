# 基因组区间操作

## 概述

polars-bio 提供 8 种核心基因组区间运算操作。所有操作均作用于包含基因组区间的 Polars DataFrames 或 LazyFrames（默认列：`chrom`, `start`, `end`），默认返回 **LazyFrame**。如需即时计算结果，请传递 `output_type="polars.DataFrame"` 参数。

## 操作概览

| 操作 | 输入 | 描述 |
|-----------|--------|-------------|
| `overlap` | 两个 DataFrame | 查找重叠区间对 |
| `count_overlaps` | 两个 DataFrame | 统计第一组中每个区间的重叠次数 |
| `nearest` | 两个 DataFrame | 查找两组间最近的区间 |
| `merge` | 一个 DataFrame | 合并重叠/首尾相接的区间 |
| `cluster` | 一个 DataFrame | 为重叠区间分配聚类 ID |
| `coverage` | 两个 DataFrame | 计算每个区间的覆盖计数 |
| `complement` | 一个 DataFrame + 基因组 | 查找区间间的空隙 |
| `subtract` | 两个 DataFrame | 移除重叠部分 |

## overlap

查找两个 DataFrame 间重叠的区间对。

### 函数式 API

```python
import polars as pl
import polars_bio as pb

df1 = pl.DataFrame({
    "chrom": ["chr1", "chr1", "chr1"],
    "start": [1, 5, 22],
    "end":   [6, 9, 30],
})

df2 = pl.DataFrame({
    "chrom": ["chr1", "chr1"],
    "start": [3, 25],
    "end":   [8, 28],
})

# 默认返回 LazyFrame
result_lf = pb.overlap(df1, df2, suffixes=("_1", "_2"))
result_df = result_lf.collect()

# 或直接获取 DataFrame
result_df = pb.overlap(df1, df2, suffixes=("_1", "_2"), output_type="polars.DataFrame")
```

### 链式调用 API（仅限 LazyFrame）

```python
result = df1.lazy().pb.overlap(df2, suffixes=("_1", "_2")).collect()
```

### 参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `df1` | DataFrame/LazyFrame/str | 必需 | 第一组（探针）区间集 |
| `df2` | DataFrame/LazyFrame/str | 必需 | 第二组（构建）区间集 |
| `suffixes` | tuple[str, str] | `("_1", "_2")` | 重叠列名的后缀 |
| `on_cols` | list[str] | `None` | 额外连接列（除基因组坐标外） |
| `cols1` | list[str] | `["chrom", "start", "end"]` | df1 中的列名 |
| `cols2` | list[str] | `["chrom", "start", "end"]` | df2 中的列名 |
| `algorithm` | str | `"Coitrees"` | 区间算法 |
| `low_memory` | bool | `False` | 低内存模式 |
| `output_type` | str | `"polars.LazyFrame"` | 输出格式：`"polars.LazyFrame"`, `"polars.DataFrame"`, `"pandas.DataFrame"` |
| `projection_pushdown` | bool | `True` | 启用投影下推优化 |

### 输出结构

返回带后缀的输入列：
- `chrom_1`, `start_1`, `end_1`（来自 df1）
- `chrom_2`, `start_2`, `end_2`（来自 df2）
- df1 和 df2 的所有额外列

染色体列类型为 `String`，起止列类型为 `Int64`。

## count_overlaps

统计 df1 中每个区间与 df2 的重叠次数。

```python
# 函数式
counts = pb.count_overlaps(df1, df2)

# 链式调用（LazyFrame）
counts = df1.lazy().pb.count_overlaps(df2)
```

### 参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `df1` | DataFrame/LazyFrame/str | 必需 | 查询区间集 |
| `df2` | DataFrame/LazyFrame/str | 必需 | 目标区间集 |
| `suffixes` | tuple[str, str] | `("", "_")` | 列名后缀 |
| `cols1` | list[str] | `["chrom", "start", "end"]` | df1 中的列名 |
| `cols2` | list[str] | `["chrom", "start", "end"]` | df2 中的列名 |
| `on_cols` | list[str] | `None` | 额外连接列 |
| `output_type` | str | `"polars.LazyFrame"` | 输出格式 |
| `naive_query` | bool | `True` | 使用简单查询策略 |
| `projection_pushdown` | bool | `True` | 启用投影下推 |

### 输出结构

返回 df1 的列，并添加 `count` 列（Int64 类型）。

## nearest

为 df1 中每个区间查找 df2 中最近的区间。

```python
# 查找最近（默认：k=1，任意方向）
nearest = pb.nearest(df1, df2, output_type="polars.DataFrame")

# 查找 k 个最近
nearest = pb.nearest(df1, df2, k=3)

# 排除结果中的重叠区间
nearest = pb.nearest(df1, df2, overlap=False)

# 不包含距离列
nearest = pb.nearest(df1, df2, distance=False)
```

### 参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `df1` | DataFrame/LazyFrame/str | 必需 | 查询区间集 |
| `df2` | DataFrame/LazyFrame/str | 必需 | 目标区间集 |
| `suffixes` | tuple[str, str] | `("_1", "_2")` | 列名后缀 |
| `on_cols` | list[str] | `None` | 额外连接列 |
| `cols1` | list[str] | `["chrom", "start", "end"]` | df1 中的列名 |
| `cols2` | list[str] | `["chrom", "start", "end"]` | df2 中的列名 |
| `k` | int | `1` | 要查找的最近邻数量 |
| `overlap` | bool | `True` | 结果中包含重叠区间 |
| `distance` | bool | `True` | 输出中包含距离列 |
| `output_type` | str | `"polars.LazyFrame"` | 输出格式 |
| `projection_pushdown` | bool | `True` | 启用投影下推 |

### 输出结构

返回两个 DataFrame 的列（带后缀），并添加 `distance` 列（Int64 类型）表示到最近区间的距离（重叠时为 0）。若 `distance=False` 则省略距离列。

## merge

合并单个 DataFrame 中重叠和首尾相接的区间。

```python
import polars as pl
import polars_bio as pb

df = pl.DataFrame({
    "chrom": ["chr1", "chr1", "chr1", "chr2"],
    "start": [1, 4, 20, 1],
    "end":   [6, 9, 30, 10],
})

# 函数式
merged = pb.merge(df, output_type="polars.DataFrame")

# 链式调用（LazyFrame）
merged = df.lazy().pb.merge().collect()

# 在最小距离内合并区间
merged = pb.merge(df, min_dist=10)
```

### 参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `df` | DataFrame/LazyFrame/str | 必需 | 待合并区间集 |
| `min_dist` | int | `0` | 合并区间的最小距离（0=必须重叠或首尾相接） |
| `cols` | list[str] | `["chrom", "start", "end"]` | 列名 |
| `on_cols` | list[str] | `None` | 额外分组列 |
| `output_type` | str | `"polars.LazyFrame"` | 输出格式 |
| `projection_pushdown` | bool | `True` | 启用投影下推 |

### 输出结构

| 列 | 类型 | 描述 |
|--------|------|-------------|
| `chrom` | String | 染色体 |
| `start` | Int64 | 合并后区间起始 |
| `end` | Int64 | 合并后区间终止 |
| `n_intervals` | Int64 | 合并的区间数量 |

## cluster

为重叠区间分配聚类 ID。相互重叠的区间分配相同聚类 ID。

```python
# 函数式
clustered = pb.cluster(df, output_type="polars.DataFrame")

# 链式调用（LazyFrame）
clustered = df.lazy().pb.cluster().collect()

# 带最小距离
clustered = pb.cluster(df, min_dist=5)
```

### 参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `df` | DataFrame/LazyFrame/str | 必需 | 区间集 |
| `min_dist` | int | `0` | 聚类最小距离 |
| `cols` | list[str] | `["chrom", "start", "end"]` | 列名 |
| `output_type` | str | `"polars.LazyFrame"` | 输出格式 |
| `projection_pushdown` | bool | `True` | 启用投影下推 |

### 输出结构

返回原始列，并添加：

| 列 | 类型 | 描述 |
|--------|------|-------------|
| `cluster` | Int64 | 聚类 ID（相同聚类的区间重叠） |
| `cluster_start` | Int64 | 聚类范围起始 |
| `cluster_end` | Int64 | 聚类范围终止 |

## coverage

计算每个区间的覆盖计数。此为**双输入**操作：针对 df1 的每个区间，统计 df2 的覆盖情况。

```python
# 函数式
cov = pb.coverage(df1, df2, output_type="polars.DataFrame")

# 链式调用（LazyFrame）
cov = df1.lazy().pb.coverage(df2).collect()
```

### 参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `df1` | DataFrame/LazyFrame/str | 必需 | 查询区间 |
| `df2` | DataFrame/LazyFrame/str | 必需 | 覆盖源区间 |
| `suffixes` | tuple[str, str] | `("_1", "_2")` | 列名后缀 |
| `on_cols` | list[str] | `None` | 额外连接列 |
| `cols1` | list[str] | `["chrom", "start", "end"]` | df1 中的列名 |
| `cols2` | list[str] | `["chrom", "start", "end"]` | df2 中的列名 |
| `output_type` | str | `"polars.LazyFrame"` | 输出格式 |
| `projection_pushdown` | bool | `True` | 启用投影下推 |

### 输出结构

返回 df1 的列，并添加 `coverage` 列（Int64 类型）。

## complement

查找基因组内区间间的空隙。需要指定染色体大小的基因组定义。

```python
import polars as pl
import polars_bio as pb

df = pl.DataFrame({
    "chrom": ["chr1", "chr1"],
    "start": [100, 500],
    "end":   [200, 600],
})

genome = pl.DataFrame({
    "chrom": ["chr1"],
    "start": [0],
    "end":   [1000],
})

# 函数式
gaps = pb.complement(df, view_df=genome, output_type="polars.DataFrame")

# 链式调用（LazyFrame）
gaps = df.lazy().pb.complement(genome).collect()
```

### 参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `df` | DataFrame/LazyFrame/str | 必需 | 区间集 |
| `view_df` | DataFrame/LazyFrame | `None` | 定义染色体范围的基因组（含 chrom, start, end） |
| `cols` | list[str] | `["chrom", "start", "end"]` | df 中的列名 |
| `view_cols` | list[str] | `None` | view_df 中的列名 |
| `output_type` | str | `"polars.LazyFrame"` | 输出格式 |
| `projection_pushdown` | bool | `True` | 启用投影下推 |

### 输出结构

返回包含 `chrom` (String), `start` (Int64), `end` (Int64) 列的 DataFrame，表示区间间的空隙。

## subtract

移除 df1 中与 df2 重叠的部分区间。

```python
# 函数式
result = pb.subtract(df1, df2, output_type="polars.DataFrame")

# 链式调用（LazyFrame）
result = df1.lazy().pb.subtract(df2).collect()
```

### 参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `df1` | DataFrame/LazyFrame/str | 必需 | 被减区间集 |
| `df2` | DataFrame/LazyFrame/str | 必需 | 减去的区间集 |
| `cols1` | list[str] | `["chrom", "start", "end"]` | df1 中的列名 |
| `cols2` | list[str] | `["chrom", "start", "end"]` | df2 中的列名 |
| `output_type` | str | `"polars.LazyFrame"` | 输出格式 |
| `projection_pushdown` | bool | `True` | 启用投影下推 |

### 输出结构

返回 `chrom` (String), `start` (Int64), `end` (Int64) 列，表示 df1 区间被减去后的剩余部分。

## 性能考量

### 探针-构建架构

双输入操作（`overlap`, `nearest`, `count_overlaps`, `coverage`, `subtract`）采用探针-构建连接：
- **探针**（第一个 DataFrame）：逐行迭代
- **构建**（第二个 DataFrame）：索引化为区间树以快速查找

为获得最佳性能，应将**较大**的 DataFrame 作为探针（第一个参数），**较小**的作为构建（第二个参数）。

### 并行处理

默认情况下 polars-bio 使用单执行分区。处理大型数据集时启用并行执行：

```python
import os
import polars_bio as pb

pb.set_option("datafusion.execution.target_partitions", os.cpu_count())
```

### 流式执行

默认启用 DataFusion 流式处理。数据按批次处理，支持超出可用内存的大型数据集进行核外计算。

### 惰性求值适用场景

在以下情况使用 `scan_*` 函数和惰性 DataFrame：
- 文件大小超过可用内存
- 仅需部分结果时
- 可优化中间结果的管道操作

```python
# 惰性管道
lf1 = pb.scan_bed("large1.bed")
lf2 = pb.scan_bed("large2.bed")
result = pb.overlap(lf1, lf2).collect()
```
