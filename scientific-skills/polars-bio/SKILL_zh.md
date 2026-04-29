---
name: polars-bio
description: 基于Polars DataFrame的高性能基因组区间操作与生物信息学文件I/O。支持BED/VCF/BAM/GFF区间的重叠、最近邻、合并、覆盖、互补和差集计算。具备流式处理、云原生特性，是更快的bioframe替代方案。
license: https://github.com/biodatageeks/polars-bio/blob/main/LICENSE
metadata:
    skill-author: K-Dense Inc.
---

# polars-bio

## 概述

polars-bio是基于Polars、Apache Arrow和Apache DataFusion构建的高性能Python库，专注于基因组区间操作和生物信息学文件I/O。它提供熟悉的DataFrame中心化API，支持区间运算（重叠、最近邻、合并、覆盖、互补、差集）及常见生物信息学格式读写（BED、VCF、BAM、CRAM、GFF/GTF、FASTA、FASTQ）。

核心价值：
- 在真实基因组基准测试中比bioframe**快6-38倍**
- 通过DataFusion支持大型基因组的**流式/核外处理**
- 支持谓词下推的**云原生**文件I/O（S3、GCS、Azure）
- **双API风格**：函数式（`pb.overlap(df1, df2)`）与方法链式（`df1.lazy().pb.overlap(df2)`）
- 通过DataFusion SQL引擎提供**基因组数据SQL接口**

## 适用场景

在以下场景使用本技能：
- 执行基因组区间操作（重叠、最近邻、合并、覆盖、互补、差集）
- 读写生物信息学文件格式（BED、VCF、BAM、CRAM、GFF/GTF、FASTA、FASTQ）
- 处理内存无法容纳的大型基因组数据集（流式模式）
- 对基因组数据文件运行SQL查询
- 从bioframe迁移到更快的替代方案
- 基于BAM/CRAM文件计算读取深度/堆积
- 处理包含基因组区间的Polars DataFrame

## 快速入门

### 安装

```bash
pip install polars-bio
# 或
uv pip install polars-bio
```

如需pandas兼容性：
```bash
pip install polars-bio[pandas]
```

### 基础重叠示例

```python
import polars as pl
import polars_bio as pb

# 创建两个区间DataFrame
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

# 函数式API（默认返回LazyFrame）
result = pb.overlap(df1, df2)
result_df = result.collect()

# 直接获取DataFrame
result_df = pb.overlap(df1, df2, output_type="polars.DataFrame")

# 方法链式API（通过LazyFrame的.pb访问器）
result = df1.lazy().pb.overlap(df2)
result_df = result.collect()
```

### 读取BED文件

```python
import polars_bio as pb

# 即时读取（加载整个文件）
df = pb.read_bed("regions.bed")

# 惰性扫描（流式处理大型文件）
lf = pb.scan_bed("regions.bed")
result = lf.collect()
```

## 核心功能

### 1. 基因组区间操作

polars-bio提供8种核心区间运算。所有操作均接受含`chrom`、`start`、`end`列（可配置）的Polars DataFrame，默认返回`LazyFrame`（使用`output_type="polars.DataFrame"`获取即时结果）。

**操作集：**
- `overlap` / `count_overlaps` - 查找或统计两组区间重叠
- `nearest` - 查找最近邻区间（可配置`k`、`overlap`、`distance`参数）
- `merge` - 合并组内重叠/邻接区间
- `cluster` - 为重叠区间分配聚类ID
- `coverage` - 计算每区间覆盖计数（双输入操作）
- `complement` - 查找基因组内区间间隙
- `subtract` - 移除与另一组重叠的区间部分

**示例：**
```python
import polars_bio as pb

# 查找重叠区间（返回LazyFrame）
result = pb.overlap(df1, df2, suffixes=("_1", "_2"))

# 统计每区间重叠数
counts = pb.count_overlaps(df1, df2)

# 合并重叠区间
merged = pb.merge(df1)

# 查找最近邻区间
nearest = pb.nearest(df1, df2)

# 将LazyFrame结果收集为DataFrame
result_df = result.collect()
```

**参考：** 详见`references/interval_operations.md`获取完整操作文档、参数说明、输出模式及性能考量。

### 2. 生物信息学文件I/O

通过`read_*`、`scan_*`、`write_*`和`sink_*`函数读写常见生物信息学格式。支持云存储（S3、GCS、Azure）和压缩（GZIP、BGZF）。

**支持格式：**
- **BED** - 基因组区间（`read_bed`、`scan_bed`，通用`write_*`）
- **VCF** - 遗传变异（`read_vcf`、`scan_vcf`、`write_vcf`、`sink_vcf`）
- **BAM** - 比对序列（`read_bam`、`scan_bam`、`write_bam`、`sink_bam`）
- **CRAM** - 压缩比对（`read_cram`、`scan_cram`、`write_cram`、`sink_cram`）
- **GFF** - 基因注释（`read_gff`、`scan_gff`）
- **GTF** - 基因注释（`read_gtf`、`scan_gtf`）
- **FASTA** - 参考序列（`read_fasta`、`scan_fasta`）
- **FASTQ** - 测序序列（`read_fastq`、`scan_fastq`、`write_fastq`、`sink_fastq`）
- **SAM** - 文本比对（`read_sam`、`scan_sam`、`write_sam`、`sink_sam`）
- **Hi-C pairs** - 染色质接触（`read_pairs`、`scan_pairs`）

**示例：**
```python
import polars_bio as pb

# 读取VCF文件
variants = pb.read_vcf("samples.vcf.gz")

# 惰性扫描BAM文件（流式）
alignments = pb.scan_bam("aligned.bam")

# 读取GFF注释
genes = pb.read_gff("annotations.gff3")

# 云存储（独立参数，非字典）
df = pb.read_bed("s3://bucket/regions.bed",
                 allow_anonymous=True)
```

**参考：** 详见`references/file_io.md`获取各格式列模式、参数说明、云存储选项及压缩支持。

### 3. SQL数据处理

将生物信息学文件注册为表后使用DataFusion SQL查询，结合SQL能力与polars-bio的基因组感知读取器。

```python
import polars as pl
import polars_bio as pb

# 注册文件为SQL表（路径在前，name=关键字）
pb.register_vcf("samples.vcf.gz", name="variants")
pb.register_bed("target_regions.bed", name="regions")

# 使用SQL查询（返回LazyFrame）
result = pb.sql("SELECT chrom, start, end, ref, alt FROM variants WHERE qual > 30")
result_df = result.collect()

# 将Polars DataFrame注册为SQL表
pb.from_polars("my_intervals", df)
result = pb.sql("SELECT * FROM my_intervals WHERE chrom = 'chr1'").collect()
```

**参考：** 详见`references/sql_processing.md`获取注册函数、SQL语法及示例。

### 4. 堆积操作

基于CIGAR感知深度计算，从BAM/CRAM文件计算每碱基读取深度。

```python
import polars_bio as pb

# 计算BAM文件深度
depth_lf = pb.depth("aligned.bam")
depth_df = depth_lf.collect()

# 带质量过滤
depth_lf = pb.depth("aligned.bam", min_mapping_quality=20)
```

**参考：** 详见`references/pileup_operations.md`获取参数说明与集成模式。

## 关键概念

### 坐标系

polars-bio默认采用**1-based**坐标系（基因组惯例）。可通过全局设置修改：

```python
import polars_bio as pb

# 切换至0-based坐标系
pb.set_option("coordinate_system", "0-based")

# 切换回1-based（默认）
pb.set_option("coordinate_system", "1-based")
```

I/O函数也接受`use_zero_based`参数在结果DataFrame上设置坐标元数据：

```python
# 显式设置0-based元数据读取BED
df = pb.read_bed("regions.bed", use_zero_based=True)
```

**重要提示：** BED文件格式始终为0-based半开区间。polars-bio在读取BED文件时自动处理转换。I/O函数会将坐标元数据附加至DataFrame，并在操作中传递该元数据。

### 双API风格

**函数式API** - 独立函数，显式输入：
```python
result = pb.overlap(df1, df2, suffixes=("_1", "_2"))
merged = pb.merge(df)
```

**方法链式API** - 通过**LazyFrames**的`.pb`访问器（非DataFrame）：
```python
result = df1.lazy().pb.overlap(df2)
merged = df.lazy().pb.merge()
```

**重要提示：** 区间操作的`.pb`访问器仅适用于`LazyFrame`。在`DataFrame`上，`.pb`仅提供写入操作（`write_bam`、`write_vcf`等）。

方法链式支持流畅管道：
```python
# 链式区间操作（注意：overlap输出带后缀列，
# 需在合并前重命名，因merge要求chrom/start/end）
result = (
    df1.lazy()
    .pb.overlap(df2)
    .filter(pl.col("start_2") > 1000)
    .select(
        pl.col("chrom_1").alias("chrom"),
        pl.col("start_1").alias("start"),
        pl.col("end_1").alias("end"),
    )
    .pb.merge()
    .collect()
)
```

### 探针-构建架构

对于双输入操作（重叠、最近邻、计数重叠、覆盖），polars-bio采用探针-构建连接策略：
- **首个**DataFrame为**探针**（被迭代）
- **次个**DataFrame为**构建**（建立索引供查询）

为获得最佳性能，请将较大DataFrame作为首个参数（探针），较小者作为次个参数（构建）。

### 列命名约定

默认期望列名为`chrom`、`start`、`end`。可通过列表指定自定义列名：

```python
result = pb.overlap(
    df1, df2,
    cols1=["chromosome", "begin", "finish"],
    cols2=["chr", "pos_start", "pos_end"],
)
```

### 返回类型与结果收集

所有区间操作和`pb.sql()`默认返回**LazyFrame**。使用`.collect()`物化结果，或传递`output_type="polars.DataFrame"`进行即时求值：

```python
# 惰性（默认）- 按需收集
result_lf = pb.overlap(df1, df2)
result_df = result_lf.collect()

# 即时- 直接获取DataFrame
result_df = pb.overlap(df1, df2, output_type="polars.DataFrame")
```

### 流式与核外处理

对于超过可用内存的数据集，使用`scan_*`函数和流式执行：

```python
# 惰性扫描文件
lf = pb.scan_bed("large_intervals.bed")

# 流式处理
result = lf.collect(streaming=True)
```

DataFusion流式处理在区间操作中默认启用，通过分批处理数据避免全量加载。

## 常见陷阱

1. **`.pb`访问器在DataFrame与LazyFrame的区别**：区间操作（重叠、合并等）仅适用于`LazyFrame.pb`。`DataFrame.pb`仅含写入方法。执行链式区间操作前使用`.lazy()`转换。

2. **LazyFrame返回值**：所有区间操作和`pb.sql()`默认返回`LazyFrame`。勿忘`.collect()`或使用`output_type="polars.DataFrame"`。

3. **列名不匹配**：polars-bio默认期望`chrom`、`start`、`end`。若列名不同，请使用`cols1`/`cols2`参数（以列表形式）。

4. **坐标系元数据**：手动构建DataFrame时（非通过`read_*`/`scan_*`），polars-bio会警告缺失坐标元数据。使用`pb.set_option("coordinate_system", "0-based")`全局设置，或使用自动设置元数据的I/O函数。

5. **探针-构建顺序影响**：对于重叠、最近邻和覆盖操作，首个DataFrame被探针化处理。交换参数将改变输出列中区间的左右位置，并影响性能。

6. **INT32位置限制**：基因组位置以32位整数存储，坐标上限约21亿。满足所有已知基因组需求，但在自定义坐标空间中可能受限。

7. **BAM索引要求**：`read_bam`和`scan_bam`需BAM旁存在`.bai`索引文件。若缺失请用`samtools index`创建。

8. **默认禁用并行执行**：DataFusion并行度默认为1分区。处理大型数据集时启用：
   ```python
   pb.set_option("datafusion.execution.target_partitions", 8)
   ```

9. **CRAM有独立函数**：CRAM文件请使用`read_cram`/`scan_cram`/`register_cram`（非`read_bam`）。CRAM函数需`reference_path`参数。

## 最佳实践

1. **大型文件使用`scan_*`**：处理超过可用内存的文件时，优先选用`scan_bed`、`scan_vcf`等而非`read_*`。扫描函数支持流式处理和谓词下推。

2. **大型数据集配置并行度**：
   ```python
   import os
   pb.set_option("datafusion.execution.target_partitions", os.cpu_count())
   ```

3. **使用BGZF压缩**：BGZF压缩文件（`.bed.gz`、`.vcf.gz`）支持并行块解压，显著快于普通GZIP。

4. **提前选择列**：仅需特定列时提前筛选以降低内存占用：
   ```python
   df = pb.read_vcf("large.vcf.gz").select("chrom", "start", "end", "ref", "alt")
   ```

5. **直接使用云路径**：将S3/GCS/Azure URI直接传入read/scan函数，无需预先下载：
   ```python
   df = pb.read_bed("s3://my-bucket/regions.bed", allow_anonymous=True)
   ```

6. **单操作用函数式API，管道操作用方法链式**：单次操作使用`pb.overlap()`，多步骤管道使用`.lazy().pb.overlap()`。

## 资源

### references/目录

各核心功能详细文档：

- **interval_operations.md** - 全部8种区间操作的参数、示例、输出模式及性能技巧。基因组范围运算核心参考。

- **file_io.md** - 支持格式表、各格式列模式、云存储配置、压缩支持及通用参数。

- **sql_processing.md** - 注册函数、DataFusion SQL语法、SQL与区间操作结合及查询示例。

- **pileup_operations.md** - 基于BAM/CRAM文件的每碱基读取深度计算、参数说明及与区间操作的集成。

- **configuration.md** - 全局设置（并行度、坐标系、流模式）、日志记录及元数据管理。

- **bioframe_migration.md** - 操作映射表、API差异、性能对比、迁移代码示例及p
