# 生物信息学文件输入输出

## 概述

polars-bio 为常见生物信息学格式提供 `read_*`、`scan_*`、`write_*` 和 `sink_*` 函数。`read_*` 将数据立即加载到 DataFrame 中，而 `scan_*` 创建用于流式/核外处理的 LazyFrame。`write_*` 从 DataFrame/LazyFrame 写入并返回行数，而 `sink_*` 从 LazyFrame 流式输出。

## 支持格式

| 格式 | 读取 | 扫描 | 注册(SQL) | 写入 | 流式输出 |
|--------|------|------|-----------------|-------|------|
| BED | `read_bed` | `scan_bed` | `register_bed` | — | — |
| VCF | `read_vcf` | `scan_vcf` | `register_vcf` | `write_vcf` | `sink_vcf` |
| BAM | `read_bam` | `scan_bam` | `register_bam` | `write_bam` | `sink_bam` |
| CRAM | `read_cram` | `scan_cram` | `register_cram` | `write_cram` | `sink_cram` |
| GFF | `read_gff` | `scan_gff` | `register_gff` | — | — |
| GTF | `read_gtf` | `scan_gtf` | `register_gtf` | — | — |
| FASTA | `read_fasta` | `scan_fasta` | — | — | — |
| FASTQ | `read_fastq` | `scan_fastq` | `register_fastq` | `write_fastq` | `sink_fastq` |
| SAM | `read_sam` | `scan_sam` | `register_sam` | `write_sam` | `sink_sam` |
| Hi-C pairs | `read_pairs` | `scan_pairs` | `register_pairs` | — | — |
| 通用表格 | `read_table` | `scan_table` | — | — | — |

## 通用云存储/IO参数

所有 `read_*` 和 `scan_*` 函数共享以下参数（替代单个 `storage_options` 字典）：

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `path` | str | 必填 | 文件路径（本地/S3/GCS/Azure） |
| `chunk_size` | int | `8` | 并行读取的分块数 |
| `concurrent_fetches` | int | `1` | 云存储并发获取数 |
| `allow_anonymous` | bool | `True` | 允许匿名访问云存储 |
| `enable_request_payer` | bool | `False` | 启用云存储请求方付费 |
| `max_retries` | int | `5` | 云操作最大重试次数 |
| `timeout` | int | `300` | 云操作超时时间（秒） |
| `compression_type` | str | `"auto"` | 压缩类型（根据扩展名自动检测） |
| `projection_pushdown` | bool | `True` | 启用投影下推优化 |
| `use_zero_based` | bool | `None` | 设置坐标系元数据（None=使用全局设置） |

并非所有函数都支持全部参数。SAM函数缺少云参数，FASTA/FASTQ缺少`predicate_pushdown`。

## BED格式

### read_bed / scan_bed

读取BED文件。自动检测列数（BED3至BED12）。BED文件使用0-based半开坐标系；polars-bio自动附加坐标系元数据。

```python
import polars_bio as pb

# 立即读取
df = pb.read_bed("regions.bed")

# 惰性扫描
lf = pb.scan_bed("regions.bed")
```

### 列模式（BED3）

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `chrom` | String | 染色体名称 |
| `start` | Int64 | 起始位置 |
| `end` | Int64 | 终止位置 |

扩展BED字段（自动检测）增加：`name`, `score`, `strand`, `thickStart`, `thickEnd`, `itemRgb`, `blockCount`, `blockSizes`, `blockStarts`。

## VCF格式

### read_vcf / scan_vcf

读取VCF/BCF文件。支持 `.vcf`, `.vcf.gz`, `.bcf`。

```python
import polars_bio as pb

# 读取VCF
df = pb.read_vcf("variants.vcf.gz")

# 提取指定INFO和FORMAT字段作为列
df = pb.read_vcf("variants.vcf.gz", info_fields=["AF", "DP"], format_fields=["GT", "GQ"])

# 读取特定样本
df = pb.read_vcf("variants.vcf.gz", samples=["SAMPLE1", "SAMPLE2"])
```

### 附加参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `info_fields` | list[str] | `None` | 提取为列的INFO字段 |
| `format_fields` | list[str] | `None` | 提取为列的FORMAT字段 |
| `samples` | list[str] | `None` | 包含的样本 |
| `predicate_pushdown` | bool | `True` | 启用谓词下推 |

### 列模式

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `chrom` | String | 染色体 |
| `start` | UInt32 | 起始位置 |
| `end` | UInt32 | 终止位置 |
| `id` | String | 变异ID |
| `ref` | String | 参考等位基因 |
| `alt` | String | 替代等位基因 |
| `qual` | Float32 | 质量分数 |
| `filter` | String | 过滤状态 |
| `info` | String | INFO字段（原始值，除非指定`info_fields`） |

### write_vcf / sink_vcf

```python
import polars_bio as pb

# 将DataFrame写入VCF
rows_written = pb.write_vcf(df, "output.vcf")

# 将LazyFrame流式写入VCF
pb.sink_vcf(lf, "output.vcf")
```

## BAM格式

### read_bam / scan_bam

从BAM文件读取比对序列。需要 `.bai` 索引文件。

```python
import polars_bio as pb

# 读取BAM
df = pb.read_bam("aligned.bam")

# 扫描BAM（流式）
lf = pb.scan_bam("aligned.bam")

# 读取指定标签
df = pb.read_bam("aligned.bam", tag_fields=["NM", "MD"])
```

### 附加参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `tag_fields` | list[str] | `None` | 提取为列的SAM标签 |
| `predicate_pushdown` | bool | `True` | 启用谓词下推 |
| `infer_tag_types` | bool | `True` | 根据数据推断标签列类型 |
| `infer_tag_sample_size` | int | `100` | 类型推断采样记录数 |
| `tag_type_hints` | list[str] | `None` | 标签的显式类型提示 |

### 列模式

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `chrom` | String | 参考序列名称 |
| `start` | Int64 | 比对起始位置 |
| `end` | Int64 | 比对终止位置 |
| `name` | String | 读段名称 |
| `flags` | UInt32 | SAM标志位 |
| `mapping_quality` | UInt32 | 比对质量 |
| `cigar` | String | CIGAR字符串 |
| `sequence` | String | 读段序列 |
| `quality_scores` | String | 碱基质量字符串 |
| `mate_chrom` | String | 配对读段参考名称 |
| `mate_start` | Int64 | 配对读段起始位置 |
| `template_length` | Int64 | 模板长度 |

### write_bam / sink_bam

```python
rows_written = pb.write_bam(df, "output.bam")
rows_written = pb.write_bam(df, "output.bam", sort_on_write=True)

pb.sink_bam(lf, "output.bam")
pb.sink_bam(lf, "output.bam", sort_on_write=True)
```

## CRAM格式

### read_cram / scan_cram

CRAM文件有**独立于BAM的函数**。需要参考FASTA和 `.crai` 索引。

```python
import polars_bio as pb

# 读取CRAM（需参考序列）
df = pb.read_cram("aligned.cram", reference_path="reference.fasta")

# 扫描CRAM（流式）
lf = pb.scan_cram("aligned.cram", reference_path="reference.fasta")
```

附加参数和列模式与BAM相同，增加：

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `reference_path` | str | `None` | 参考FASTA路径 |

### write_cram / sink_cram

```python
rows_written = pb.write_cram(df, "output.cram", reference_path="reference.fasta")
pb.sink_cram(lf, "output.cram", reference_path="reference.fasta")
```

## GFF/GTF格式

### read_gff / scan_gff / read_gtf / scan_gtf

GFF3和GTF有独立函数。

```python
import polars_bio as pb

# 读取GFF3
df = pb.read_gff("annotations.gff3")

# 读取GTF
df = pb.read_gtf("genes.gtf")

# 提取特定属性作为列
df = pb.read_gff("annotations.gff3", attr_fields=["gene_id", "gene_name"])
```

### 附加参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `attr_fields` | list[str] | `None` | 提取为列的属性字段 |
| `predicate_pushdown` | bool | `True` | 启用谓词下推 |

### 列模式

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `chrom` | String | 序列名称 |
| `source` | String | 特征来源 |
| `type` | String | 特征类型（基因/外显子等） |
| `start` | Int64 | 起始位置 |
| `end` | Int64 | 终止位置 |
| `score` | Float32 | 分数 |
| `strand` | String | 链方向（+/-/.） |
| `phase` | UInt32 | 相位（0/1/2） |
| `attributes` | String | 属性字符串 |

## FASTA格式

### read_fasta / scan_fasta

从FASTA文件读取参考序列。

```python
import polars_bio as pb

df = pb.read_fasta("reference.fasta")
```

### 列模式

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `name` | String | 序列名称 |
| `description` | String | 描述行 |
| `sequence` | String | 核苷酸序列 |

## FASTQ格式

### read_fastq / scan_fastq

读取带质量分数的原始测序读段。

```python
import polars_bio as pb

df = pb.read_fastq("reads.fastq.gz")
```

### 列模式

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `name` | String | 读段名称 |
| `description` | String | 描述行 |
| `sequence` | String | 核苷酸序列 |
| `quality` | String | 质量字符串（Phred+33编码） |

### write_fastq / sink_fastq

```python
rows_written = pb.write_fastq(df, "output.fastq")
pb.sink_fastq(lf, "output.fastq")
```

## SAM格式

### read_sam / scan_sam

读取文本格式比对文件。列模式与BAM相同。无云参数。

```python
import polars_bio as pb

df = pb.read_sam("alignments.sam")
```

### 附加参数

| 参数 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `tag_fields` | list[str] | `None` | 提取的SAM标签 |
| `infer_tag_types` | bool | `True` | 推断标签类型 |
| `infer_tag_sample_size` | int | `100` | 类型推断采样大小 |
| `tag_type_hints` | list[str] | `None` | 显式类型提示 |

### write_sam / sink_sam

```python
rows_written = pb.write_sam(df, "output.sam")
pb.sink_sam(lf, "output.sam", sort_on_write=True)
```

## Hi-C Pairs格式

### read_pairs / scan_pairs

读取Hi-C pairs格式的染色质接触数据。

```python
import polars_bio as pb

df = pb.read_pairs("contacts.pairs")
lf = pb.scan_pairs("contacts.pairs")
```

### 列模式

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `readID` | String | 读段标识符 |
| `chrom1` | String | 第一个接触点染色体 |
| `pos1` | Int32 | 第一个接触点位置 |
| `chrom2` | String | 第二个接触点染色体 |
| `pos2` | Int32 | 第二个接触点位置 |
| `strand1` | String | 第一个接触点链方向 |
| `strand2` | String | 第二个接触点链方向 |

## 通用表格读取器

### read_table / scan_table

读取带自定义模式的制表符分隔文件。适用于非标准格式或兼容bioframe的表格。

```python
import polars_bio as pb

df = pb.read_table("custom.tsv", schema={"chrom": str, "start": int, "end": int, "name": str})
lf = pb.scan_table("custom.tsv", schema={"chrom": str, "start": int, "end": int})
```

## 云存储支持

所有 `read_*` 和 `scan_*` 函数通过独立参数支持云存储：

### Amazon S3

```python
df = pb.read_bed(
    "s3://bucket/regions.bed",
    allow_anonymous=False,
    max_retries=10,
    timeout=600,
)
```

### Google云存储

```python
df = pb.read_vcf("gs://bucket/variants.vcf.gz", allow_anonymous=True)
```

### Azure Blob存储

```python
df = pb.read_bam("az://container/aligned.bam", allow_anonymous=False)
```

**注意：** 认证访问需通过环境变量或云SDK配置（如 `AWS_ACCESS_KEY_ID`, `GOOGLE_APPLICATION_CREDENTIALS`）。

## 压缩支持

polars-bio 透明处理压缩文件：

| 压缩类型 | 扩展名 | 并行解压 |
|-------------|-----------|----------------------|
| GZIP | `.gz` | 否 |
| BGZF | `.gz` (含BGZF块) | 是 |
| 未压缩 | (无) | 不适用 |

**建议：** 大文件使用BGZF压缩（例如用 `bgzip` 创建）。BGZF支持并行块解压，相比普通GZIP显著提升读取性能。

## 描述函数

无需完整读取即可检查文件结构：

```python
import polars_bio as pb

# 描述文件模式和元数据
schema_df = pb.describe_vcf("samples.vcf.gz")
schema_df = pb.describe_bam("aligned.bam")
schema_df = pb.describe_sam("alignments.sam")
schema_df = pb.describe_cram("aligned.cram", reference_path="ref.fasta")
```
