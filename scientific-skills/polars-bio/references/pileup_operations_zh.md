# 深度叠加操作

## 概述

polars-bio 提供 `pb.depth()` 函数用于从 BAM/CRAM 文件计算每碱基或每区块的测序深度。该函数采用 CIGAR 感知的深度计算方式，能精确处理插入、缺失和剪切操作。默认返回 **LazyFrame**。

## pb.depth()

从比对文件计算测序深度。

### 基础用法

```python
import polars_bio as pb

# 计算整个BAM文件的深度（返回LazyFrame）
depth_lf = pb.depth("aligned.bam")
depth_df = depth_lf.collect()

# 直接获取DataFrame
depth_df = pb.depth("aligned.bam", output_type="polars.DataFrame")
```

### 参数说明

| 参数名 | 类型 | 默认值 | 描述 |
|-----------|------|---------|-------------|
| `path` | str | 必填 | BAM/CRAM文件路径 |
| `filter_flag` | int | `1796` | SAM标志过滤器（默认排除未比对、次要对齐、重复序列、QC失败） |
| `min_mapping_quality` | int | `0` | 纳入计算的最小比对质量值 |
| `binary_cigar` | bool | `True` | 使用二进制CIGAR提升处理速度 |
| `dense_mode` | str | `"auto"` | 密集输出模式 |
| `use_zero_based` | bool | `None` | 坐标系统（None=使用全局设置） |
| `per_base` | bool | `False` | 每碱基深度(True) vs 区块深度(False) |
| `output_type` | str | `"polars.LazyFrame"` | 输出格式：`"polars.LazyFrame"`, `"polars.DataFrame"`, `"pandas.DataFrame"` |

### 输出结构（区块模式，默认）

当 `per_base=False`（默认）时，相同深度的相邻位置被合并为区块：

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `contig` | String | 染色体/contig名称 |
| `pos_start` | Int64 | 区块起始位置 |
| `pos_end` | Int64 | 区块终止位置 |
| `coverage` | Int16 | 测序深度 |

### 输出结构（每碱基模式）

当 `per_base=True` 时，每个位置单独报告：

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `contig` | String | 染色体/contig名称 |
| `pos` | Int64 | 位置坐标 |
| `coverage` | Int16 | 该位置测序深度 |

### filter_flag 详解

默认 `filter_flag=1796` 会排除具有以下SAM标志的读段：
- 4: 未比对
- 256: 次要对齐
- 512: 质控失败
- 1024: PCR/光学重复

### CIGAR感知计算

`pb.depth()` 正确处理CIGAR操作：
- **M/X/=** (匹配/错配)：计入覆盖度
- **D** (缺失)：计入覆盖度（读段跨越缺失区域）
- **N** (跳过区域)：不计入（如剪接比对）
- **I** (插入)：不在参考位置计数
- **S/H** (软/硬剪切)：不计入

## 应用示例

### 全基因组深度分析

```python
import polars_bio as pb
import polars as pl

# 计算全基因组深度（区块模式）
depth = pb.depth("sample.bam", output_type="polars.DataFrame")

# 统计摘要
depth.select(
    pl.col("coverage").cast(pl.Int64).mean().alias("mean_depth"),
    pl.col("coverage").cast(pl.Int64).median().alias("median_depth"),
    pl.col("coverage").cast(pl.Int64).max().alias("max_depth"),
)
```

### 每碱基深度分析

```python
import polars_bio as pb

# 每碱基深度（每个位置一行）
depth = pb.depth("sample.bam", per_base=True, output_type="polars.DataFrame")
```

### 带质量过滤的深度分析

```python
import polars_bio as pb

# 仅统计高质量比对读段
depth = pb.depth(
    "sample.bam",
    min_mapping_quality=20,
    output_type="polars.DataFrame",
)
```

### 自定义标志过滤

```python
import polars_bio as pb

# 仅排除未比对(4)和重复(1024)读段
depth = pb.depth(
    "sample.bam",
    filter_flag=4 + 1024,
    output_type="polars.DataFrame",
)
```

## 与区间操作集成

深度计算结果可与polars-bio区间操作结合使用。注意深度输出使用`contig`/`pos_start`/`pos_end`列名，需通过`cols`参数进行映射：

```python
import polars_bio as pb
import polars as pl

# 计算深度
depth = pb.depth("sample.bam", output_type="polars.DataFrame")

# 重命名列以匹配区间操作规范
depth_intervals = depth.rename({
    "contig": "chrom",
    "pos_start": "start",
    "pos_end": "end",
})

# 筛选满足覆盖度的区域
adequate = depth_intervals.filter(pl.col("coverage") >= 30)

# 合并相邻满足条件的区块
merged = pb.merge(adequate, output_type="polars.DataFrame")

# 查找覆盖度缺口（补集）
genome = pl.DataFrame({
    "chrom": ["chr1"],
    "start": [0],
    "end": [248956422],
})
gaps = pb.complement(adequate, view_df=genome, output_type="polars.DataFrame")
```

### 使用cols参数替代重命名

```python
import polars_bio as pb

depth = pb.depth("sample.bam", output_type="polars.DataFrame")
targets = pb.read_bed("targets.bed")

# 通过cols1指定深度数据列名
overlapping = pb.overlap(
    depth, targets,
    cols1=["contig", "pos_start", "pos_end"],
    output_type="polars.DataFrame",
)
```
