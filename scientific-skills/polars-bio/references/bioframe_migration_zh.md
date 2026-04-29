# 从 bioframe 迁移到 polars-bio

## 概述

polars-bio 是 bioframe 核心区间操作的即插即用替代方案，在真实基因组基准测试中提供 6.5-38 倍的加速。主要差异包括：使用 Polars DataFrame 替代 pandas、采用 Rust/DataFusion 后端替代纯 Python、支持大型基因组的流式处理，以及默认返回 LazyFrame。

## 操作映射

| bioframe | polars-bio | 说明 |
|----------|------------|-------|
| `bioframe.overlap(df1, df2)` | `pb.overlap(df1, df2)` | 返回 LazyFrame；使用 `.collect()` 获取 DataFrame |
| `bioframe.closest(df1, df2)` | `pb.nearest(df1, df2)` | 重命名；使用 `k`, `overlap`, `distance` 参数 |
| `bioframe.count_overlaps(df1, df2)` | `pb.count_overlaps(df1, df2)` | 默认后缀不同：`("", "_")` 对比 bioframe |
| `bioframe.merge(df)` | `pb.merge(df)` | 输出包含 `n_intervals` 列 |
| `bioframe.cluster(df)` | `pb.cluster(df)` | 输出列：`cluster`, `cluster_start`, `cluster_end` |
| `bioframe.coverage(df1, df2)` | `pb.coverage(df1, df2)` | 两个库均支持双输入 |
| `bioframe.complement(df, chromsizes)` | `pb.complement(df, view_df=genome)` | 基因组需为 DataFrame（非 Series） |
| `bioframe.subtract(df1, df2)` | `pb.subtract(df1, df2)` | 语义相同 |

## 关键 API 差异

### DataFrame：pandas 与 Polars

**bioframe (pandas):**
```python
import bioframe
import pandas as pd

df1 = pd.DataFrame({
    "chrom": ["chr1", "chr1"],
    "start": [1, 10],
    "end":   [5, 20],
})

result = bioframe.overlap(df1, df2)
# result 是 pandas DataFrame
result["start_1"]  # pandas 列访问方式
```

**polars-bio (Polars):**
```python
import polars_bio as pb
import polars as pl

df1 = pl.DataFrame({
    "chrom": ["chr1", "chr1"],
    "start": [1, 10],
    "end":   [5, 20],
})

result = pb.overlap(df1, df2)  # 返回 LazyFrame
result_df = result.collect()   # 物化为 DataFrame
result_df.select("start_1")   # Polars 列访问方式
```

### 返回类型：默认 LazyFrame

所有 polars-bio 操作默认返回 **LazyFrame**。使用 `.collect()` 或 `output_type="polars.DataFrame"`：

```python
# bioframe：始终返回 DataFrame
result = bioframe.overlap(df1, df2)

# polars-bio：返回 LazyFrame，collect 获取 DataFrame
result_lf = pb.overlap(df1, df2)
result_df = result_lf.collect()

# 或直接获取 DataFrame
result_df = pb.overlap(df1, df2, output_type="polars.DataFrame")
```

### 基因组/染色体尺寸

**bioframe:**
```python
chromsizes = bioframe.fetch_chromsizes("hg38")  # 返回 pandas Series
complement = bioframe.complement(df, chromsizes)
```

**polars-bio:**
```python
genome = pl.DataFrame({
    "chrom": ["chr1", "chr2"],
    "start": [0, 0],
    "end":   [248956422, 242193529],
})
complement = pb.complement(df, view_df=genome)
```

### closest 与 nearest

**bioframe:**
```python
result = bioframe.closest(df1, df2)
```

**polars-bio:**
```python
# 基础最近邻查询
result = pb.nearest(df1, df2)

# 查找 k 个最近邻
result = pb.nearest(df1, df2, k=3)

# 排除重叠区间
result = pb.nearest(df1, df2, overlap=False)

# 不包含距离列
result = pb.nearest(df1, df2, distance=False)
```

### 方法链式调用（仅 polars-bio）

polars-bio 在 **LazyFrame** 上添加了 `.pb` 访问器以支持链式调用：

```python
# bioframe：顺序函数调用
merged = bioframe.merge(bioframe.overlap(df1, df2))

# polars-bio：流式管道（必须使用 LazyFrame）
# 注意：overlap 添加后缀，需在 merge 前重命名
merged = (
    df1.lazy()
    .pb.overlap(df2)
    .select(
        pl.col("chrom_1").alias("chrom"),
        pl.col("start_1").alias("start"),
        pl.col("end_1").alias("end"),
    )
    .pb.merge()
    .collect()
)
```

## 性能对比

真实基因组数据集基准测试（源自 polars-bio 论文，Bioinformatics 2025）：

| 操作 | bioframe | polars-bio | 加速比 |
|-----------|----------|------------|---------|
| overlap | 1.0x | 6.5x | 6.5x |
| nearest | 1.0x | 38x | 38x |
| merge | 1.0x | 8.2x | 8.2x |
| coverage | 1.0x | 12x | 12x |

加速源于：
- 基于 Rust 的区间树实现
- Apache DataFusion 查询引擎
- Apache Arrow 列式内存格式
- 并行执行（需配置）
- 流式/核外支持

## 迁移代码示例

### 示例 1：基础重叠分析流程

**迁移前 (bioframe):**
```python
import bioframe
import pandas as pd

df1 = pd.read_csv("peaks.bed", sep="\t", names=["chrom", "start", "end"])
df2 = pd.read_csv("genes.bed", sep="\t", names=["chrom", "start", "end", "name"])

overlaps = bioframe.overlap(df1, df2, suffixes=("_peak", "_gene"))
filtered = overlaps[overlaps["start_gene"] > 10000]
merged = bioframe.merge(filtered[["chrom_peak", "start_peak", "end_peak"]]
    .rename(columns={"chrom_peak": "chrom", "start_peak": "start", "end_peak": "end"}))
```

**迁移后 (polars-bio):**
```python
import polars_bio as pb
import polars as pl

df1 = pb.read_bed("peaks.bed")
df2 = pb.read_bed("genes.bed")

overlaps = pb.overlap(df1, df2, suffixes=("_peak", "_gene"), output_type="polars.DataFrame")
filtered = overlaps.filter(pl.col("start_gene") > 10000)
merged = pb.merge(
    filtered.select(
        pl.col("chrom_peak").alias("chrom"),
        pl.col("start_peak").alias("start"),
        pl.col("end_peak").alias("end"),
    ),
    output_type="polars.DataFrame",
)
```

### 示例 2：大规模流式处理

**迁移前 (bioframe) — 受限于内存：**
```python
import bioframe
import pandas as pd

# 需将整个文件加载到内存
df1 = pd.read_csv("huge_intervals.bed", sep="\t", names=["chrom", "start", "end"])
result = bioframe.merge(df1)  # 受内存限制
```

**迁移后 (polars-bio) — 流式处理：**
```python
import polars_bio as pb

# 惰性扫描，流式执行
lf = pb.scan_bed("huge_intervals.bed")
result = pb.merge(lf).collect(streaming=True)
```

## pandas 兼容模式

为逐步迁移，可安装 pandas 支持：

```bash
pip install polars-bio[pandas]
```

此模式支持 pandas 与 Polars DataFrame 互转：

```python
import polars_bio as pb
import polars as pl

# 将 pandas DataFrame 转为 Polars 供 polars-bio 使用
polars_df = pl.from_pandas(pandas_df)
result = pb.overlap(polars_df, other_df).collect()

# 如需转回 pandas
pandas_result = result.to_pandas()

# 或直接请求 pandas 输出
pandas_result = pb.overlap(polars_df, other_df, output_type="pandas.DataFrame")
```

## 迁移检查清单

1. 将 `import bioframe` 替换为 `import polars_bio as pb`
2. 将 `import pandas as pd` 替换为 `import polars as pl`
3. 将 DataFrame 创建从 `pd.DataFrame` 改为 `pl.DataFrame`
4. 将 `bioframe.closest` 替换为 `pb.nearest`
5. 在操作后添加 `.collect()`（默认返回 LazyFrame）
6. 将列访问 `df["col"]` 更新为 `df.select("col")` 或 `pl.col("col")`
7. 将 pandas 过滤 `df[df["col"] > x]` 替换为 `df.filter(pl.col("col") > x)`
8. 将 chromsizes 从 Series 转为含 `chrom`, `start`, `end` 的 DataFrame；以 `view_df=` 传递
9. 添加 `pb.set_option("datafusion.execution.target_partitions", N)` 启用并行
10. 将 BED 文件的 `pd.read_csv` 替换为 `pb.read_bed` 或 `pb.scan_bed`
11. 注意 `cluster` 输出列为 `cluster`（非 `cluster_id`）及 `cluster_start`, `cluster_end`
12. 注意 `merge` 输出包含 `n_intervals` 列
