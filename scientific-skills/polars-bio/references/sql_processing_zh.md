# SQL 数据处理

## 概述

polars-bio 集成了 Apache DataFusion 的 SQL 引擎，支持对生物信息学文件和 Polars 数据框执行 SQL 查询。将文件注册为表后即可使用标准 SQL 语法进行查询。所有查询均返回 **LazyFrame**——调用 `.collect()` 可物化结果。

## 注册函数

将生物信息学文件注册为 SQL 表。**路径为首个参数**，表名通过关键字参数指定：

```python
import polars_bio as pb

# 注册不同文件格式（路径在前，name=为关键字参数）
pb.register_vcf("samples.vcf.gz", name="variants")
pb.register_bed("target_regions.bed", name="regions")
pb.register_bam("aligned.bam", name="alignments")
pb.register_cram("aligned.cram", name="cram_alignments")
pb.register_gff("genes.gff3", name="annotations")
pb.register_gtf("genes.gtf", name="gtf_annotations")
pb.register_fastq("sample.fastq.gz", name="reads")
pb.register_sam("alignments.sam", name="sam_alignments")
pb.register_pairs("contacts.pairs", name="hic_contacts")
```

### 参数

所有 `register_*` 函数共享以下参数：

| 参数               | 类型    | 默认值       | 描述                     |
|--------------------|---------|--------------|--------------------------|
| `path`             | str     | 必填（首位） | 文件路径（本地或云端）   |
| `name`             | str     | `None`       | SQL 查询表名（省略时自动生成） |
| `chunk_size`       | int     | `64`         | 读取块大小               |
| `concurrent_fetches` | int     | `8`          | 云端并发获取数           |
| `allow_anonymous`  | bool    | `True`       | 允许匿名云端访问         |
| `max_retries`      | int     | `5`          | 云端重试次数             |
| `timeout`          | int     | `300`        | 云端超时秒数             |
| `enable_request_payer` | bool | `False`      | 请求方支付云端费用       |
| `compression_type` | str     | `"auto"`     | 压缩类型                 |

部分注册函数具有格式特定参数（如 `register_vcf` 的 `info_fields`）。

**注意：** 不存在 `register_fasta` 函数，可通过 `scan_fasta` + `from_polars` 变通实现。

## from_polars

将现有 Polars 数据框注册为可 SQL 查询的表：

```python
import polars as pl
import polars_bio as pb

df = pl.DataFrame({
    "chrom": ["chr1", "chr1", "chr2"],
    "start": [100, 500, 200],
    "end":   [200, 600, 400],
    "name":  ["peak1", "peak2", "peak3"],
})

pb.from_polars("my_peaks", df)

# 使用 SQL 查询
result = pb.sql("SELECT * FROM my_peaks WHERE chrom = 'chr1'").collect()
```

**重要提示：** `register_view` 接收 SQL 查询字符串而非数据框。注册数据框请使用 `from_polars`。

## register_view

通过查询字符串创建 SQL 视图：

```python
import polars_bio as pb

# 通过 SQL 查询创建视图
pb.register_view("chr1_variants", "SELECT * FROM variants WHERE chrom = 'chr1'")

# 查询视图
result = pb.sql("SELECT * FROM chr1_variants WHERE qual > 30").collect()
```

### 参数

| 参数    | 类型 | 描述               |
|---------|------|--------------------|
| `name`  | str  | 视图名称           |
| `query` | str  | 定义视图的 SQL 查询字符串 |

## pb.sql()

使用 DataFusion SQL 语法执行查询。**返回 LazyFrame**——调用 `.collect()` 获取数据框。

```python
import polars_bio as pb

# 简单查询
result = pb.sql("SELECT chrom, start, end FROM regions WHERE chrom = 'chr1'").collect()

# 聚合操作
result = pb.sql("""
    SELECT chrom, COUNT(*) as variant_count, AVG(qual) as avg_qual
    FROM variants
    GROUP BY chrom
    ORDER BY variant_count DESC
""").collect()

# 表连接
result = pb.sql("""
    SELECT v.chrom, v.start, v.end, v.ref, v.alt, r.name
    FROM variants v
    JOIN regions r ON v.chrom = r.chrom
        AND v.start >= r.start
        AND v.end <= r.end
""").collect()
```

## DataFusion SQL 语法

polars-bio 采用 Apache DataFusion 的 SQL 方言。核心特性：

### 过滤

```sql
SELECT * FROM variants WHERE qual > 30 AND filter = 'PASS'
```

### 聚合

```sql
SELECT chrom, COUNT(*) as n, MIN(start) as min_pos, MAX(end) as max_pos
FROM regions
GROUP BY chrom
HAVING COUNT(*) > 100
```

### 窗口函数

```sql
SELECT chrom, start, end,
    ROW_NUMBER() OVER (PARTITION BY chrom ORDER BY start) as row_num,
    LAG(end) OVER (PARTITION BY chrom ORDER BY start) as prev_end
FROM regions
```

### 子查询

```sql
SELECT * FROM variants
WHERE chrom IN (SELECT DISTINCT chrom FROM regions)
```

### 公用表表达式 (CTE)

```sql
WITH filtered_variants AS (
    SELECT * FROM variants WHERE qual > 30
),
chr1_regions AS (
    SELECT * FROM regions WHERE chrom = 'chr1'
)
SELECT f.chrom, f.start, f.ref, f.alt
FROM filtered_variants f
JOIN chr1_regions r ON f.start BETWEEN r.start AND r.end
```

## 结合区间操作

SQL 查询返回的 LazyFrame 可直接用于 polars-bio 的区间操作：

```python
import polars_bio as pb

# 注册文件
pb.register_vcf("samples.vcf.gz", name="variants")
pb.register_bed("target_regions.bed", name="targets")

# SQL 过滤（返回 LazyFrame）
high_qual = pb.sql("SELECT chrom, start, end FROM variants WHERE qual > 30").collect()
targets = pb.sql("SELECT chrom, start, end FROM targets WHERE chrom = 'chr1'").collect()

# 对 SQL 结果执行区间操作
overlapping = pb.overlap(high_qual, targets).collect()
```

## 示例工作流

### 变异密度分析

```python
import polars_bio as pb

pb.register_vcf("cohort.vcf.gz", name="variants")
pb.register_bed("genome_windows_1mb.bed", name="windows")

# 使用 SQL 统计窗口内变异数
result = pb.sql("""
    SELECT w.chrom, w.start, w.end, COUNT(v.start) as variant_count
    FROM windows w
    LEFT JOIN variants v ON w.chrom = v.chrom
        AND v.start >= w.start
        AND v.start < w.end
    GROUP BY w.chrom, w.start, w.end
    ORDER BY variant_count DESC
""").collect()
```

### 基因注释查询

```python
import polars_bio as pb

pb.register_gff("gencode.gff3", name="genes")

# 查找 1 号染色体所有蛋白质编码基因
coding_genes = pb.sql("""
    SELECT chrom, start, end, attributes
    FROM genes
    WHERE type = 'gene'
        AND chrom = 'chr1'
        AND attributes LIKE '%protein_coding%'
    ORDER BY start
""").collect()
```
