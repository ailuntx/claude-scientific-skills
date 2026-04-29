---
name: pysam
description: 基因组文件工具包。用于读取/写入SAM/BAM/CRAM比对文件、VCF/BCF变异文件、FASTA/FASTQ序列，支持区域提取、覆盖度计算，适用于NGS数据处理流程。
license: MIT许可证
metadata:
    skill-author: K-Dense Inc.
---

# Pysam

## 概述

Pysam是一个用于读取、操作和写入基因组数据集的Python模块。通过Pythonic接口调用htslib，可读写SAM/BAM/CRAM比对文件、VCF/BCF变异文件以及FASTA/FASTQ序列。支持查询tabix索引文件、执行覆盖度分析的pileup操作，并可直接运行samtools/bcftools命令。

## 适用场景

该技能适用于：
- 处理测序比对文件（BAM/CRAM）
- 分析遗传变异（VCF/BCF）
- 提取参考序列或基因区域
- 处理原始测序数据（FASTQ）
- 计算覆盖度或测序深度
- 构建生物信息分析流程
- 测序数据质量控制
- 变异检测与注释工作流

## 快速入门

### 安装
```bash
uv pip install pysam
```

### 基础示例

**读取比对文件：**
```python
import pysam

# 打开BAM文件并获取指定区域内的测序片段
samfile = pysam.AlignmentFile("example.bam", "rb")
for read in samfile.fetch("chr1", 1000, 2000):
    print(f"{read.query_name}: {read.reference_start}")
samfile.close()
```

**读取变异文件：**
```python
# 打开VCF文件并遍历变异位点
vcf = pysam.VariantFile("variants.vcf")
for variant in vcf:
    print(f"{variant.chrom}:{variant.pos} {variant.ref}>{variant.alts}")
vcf.close()
```

**查询参考序列：**
```python
# 打开FASTA文件并提取序列
fasta = pysam.FastaFile("reference.fasta")
sequence = fasta.fetch("chr1", 1000, 2000)
print(sequence)
fasta.close()
```

## 核心功能

### 1. 比对文件操作（SAM/BAM/CRAM）

通过`AlignmentFile`类处理比对后的测序片段，适用于分析比对结果、计算覆盖度、提取测序片段或质量控制。

**常用操作：**
- 打开并读取BAM/SAM/CRAM文件
- 获取特定基因组区域的测序片段
- 根据比对质量、标志位等条件筛选片段
- 写入筛选或修改后的比对结果
- 计算覆盖度统计量
- 执行pileup分析（逐碱基覆盖度）
- 访问测序片段序列、质量分数及比对信息

**参考文档：** 详见`references/alignment_files.md`：
- 比对文件的打开与读取
- AlignedSegment属性与方法
- 基于区域的`fetch()`操作
- 覆盖度的pileup分析
- BAM文件写入与创建
- 坐标系统与索引机制
- 性能优化技巧

### 2. 变异文件操作（VCF/BCF）

通过`VariantFile`类处理变异检测流程输出的遗传变异，适用于变异分析、筛选、注释或群体遗传学研究。

**常用操作：**
- 读写VCF/BCF文件
- 查询特定区域的变异位点
- 访问变异信息（位置、等位基因、质量）
- 提取样本的基因型数据
- 根据质量、等位基因频率等条件筛选变异
- 为变异添加注释信息
- 样本或区域子集提取

**参考文档：** 详见`references/variant_files.md`：
- 变异文件的打开与读取
- VariantRecord属性与方法
- INFO与FORMAT字段访问
- 基因型与样本数据处理
- VCF文件创建与写入
- 变异筛选与子集提取
- 多样本VCF操作

### 3. 序列文件操作（FASTA/FASTQ）

使用`FastaFile`随机访问参考序列，`FastxFile`读取原始测序数据，适用于提取基因序列、验证变异位点或处理原始测序片段。

**常用操作：**
- 按基因组坐标查询参考序列
- 提取目标基因或区域序列
- 读取带质量分数的FASTQ文件
- 验证变异位点的参考等位基因
- 计算序列统计量
- 根据质量或长度筛选测序片段
- FASTA与FASTQ格式互转

**参考文档：** 详见`references/sequence_files.md`：
- FASTA文件访问与索引
- 区域序列提取
- 基因反向互补处理
- FASTQ文件顺序读取
- 质量分数转换与筛选
- 处理tabix索引文件（BED/GTF/GFF）
- 常见序列处理模式

### 4. 集成化生物信息工作流

Pysam擅长整合多文件类型实现综合基因组分析，常见工作流结合比对文件、变异文件和参考序列。

**典型工作流：**
- 计算特定区域覆盖度统计量
- 根据比对片段验证变异位点
- 为变异添加覆盖度注释
- 提取变异位点周边序列
- 基于多条件筛选比对或变异数据
- 生成可视化覆盖度轨迹
- 跨数据类型质量控制

**参考文档：** 详见`references/common_workflows.md`：
- 质量控制工作流（BAM统计、参考一致性验证）
- 覆盖度分析（单碱基覆盖度、低覆盖区域检测）
- 变异分析（注释、基于测序支持度筛选）
- 序列提取（变异上下文、基因序列）
- 测序片段筛选与子集提取
- 整合模式（BAM+VCF、VCF+BED等）
- 复杂工作流的性能优化

## 关键概念

### 坐标系统

**重要：** Pysam采用**0基半开区间**坐标（Python惯例）：
- 起始位置为0基（首个碱基位置为0）
- 终止位置为开区间（不包含在范围内）
- 区域1000-2000包含碱基1000-1999（共1000个碱基）

**例外：** `fetch()`中的区域字符串遵循samtools惯例（1基）：
```python
samfile.fetch("chr1", 999, 2000)      # 0基：位置999-1999
samfile.fetch("chr1:1000-2000")       # 1基字符串：位置1000-2000
```

**VCF文件：** 文件格式使用1基坐标，但`VariantRecord.start`为0基。

### 索引要求

随机访问特定基因组区域需索引文件：
- **BAM文件**：需`.bai`索引（通过`pysam.index()`创建）
- **CRAM文件**：需`.crai`索引
- **FASTA文件**：需`.fai`索引（通过`pysam.faidx()`创建）
- **VCF.gz文件**：需`.tbi` tabix索引（通过`pysam.tabix_index()`创建）
- **BCF文件**：需`.csi`索引

无索引时，使用`fetch(until_eof=True)`进行顺序读取。

### 文件模式

打开文件时指定格式：
- `"rb"` - 读取BAM（二进制）
- `"r"` - 读取SAM（文本）
- `"rc"` - 读取CRAM
- `"wb"` - 写入BAM
- `"w"` - 写入SAM
- `"wc"` - 写入CRAM

### 性能优化

1. **随机访问必须使用索引文件**
2. **列式分析优先使用`pileup()`** 而非重复fetch操作
3. **计数操作使用`count()`** 避免手动迭代计数
4. **独立基因组区域采用并行处理**
5. **显式关闭文件**释放资源
6. **无索引时使用`until_eof=True`** 进行顺序处理
7. **避免多重迭代器**（必要时使用`multiple_iterators=True`）

## 常见问题

1. **坐标混淆：** 注意不同场景下的0基与1基系统差异
2. **缺失索引：** 多数操作需索引文件——请预先创建
3. **部分重叠：** `fetch()`返回与区域边界重叠的片段，不仅限于完全包含
4. **迭代器作用域：** 保持pileup迭代器引用避免"PileupProxy accessed after iterator finished"错误
5. **质量分数编辑：** 修改`query_sequence`后不可直接修改`query_qualities`——需先创建副本
6. **流限制：** 仅支持stdin/stdout流式处理，不适用任意Python文件对象
7. **线程安全：** I/O期间会释放GIL，但未完全验证全面线程安全性

## 命令行工具

Pysam提供samtools和bcftools命令接口：

```python
# 排序BAM文件
pysam.samtools.sort("-o", "sorted.bam", "input.bam")

# 建立BAM索引
pysam.samtools.index("sorted.bam")

# 查看特定区域
pysam.samtools.view("-b", "-o", "region.bam", "input.bam", "chr1:1000-2000")

# BCF工具
pysam.bcftools.view("-O", "z", "-o", "output.vcf.gz", "input.vcf")
```

**错误处理：**
```python
try:
    pysam.samtools.sort("-o", "output.bam", "input.bam")
except pysam.SamtoolsError as e:
    print(f"错误: {e}")
```

## 资源

### references/

各核心功能详细文档：

- **alignment_files.md** - SAM/BAM/CRAM操作完整指南，含AlignmentFile类、AlignedSegment属性、fetch操作、pileup分析及比对文件写入

- **variant_files.md** - VCF/BCF操作完整指南，含VariantFile类、VariantRecord属性、基因型处理、INFO/FORMAT字段及多样本操作

- **sequence_files.md** - FASTA/FASTQ操作完整指南，含FastaFile与FastxFile类、序列提取、质量分数处理及tabix索引文件访问

- **common_workflows.md** - 整合多文件类型的生物信息工作流实例，含质量控制、覆盖度分析、变异验证及序列提取

## 获取帮助

具体操作详见对应参考文档：
- BAM文件处理或覆盖度计算 → `alignment_files.md`
- 变异或基因型分析 → `variant_files.md`
- 序列提取或FASTQ处理 → `sequence_files.md`
- 多文件类型整合工作流 → `common_workflows.md`

官方文档：https://pysam.readthedocs.io/
