# 已验证工作流

## 概述
Latch 已验证工作流是由 Latch 工程师开发维护、可直接投入生产的预制生物信息学流程。这些工作流被顶尖制药公司和生物技术企业用于研究与发现。

## Python SDK 集成

`latch.verified` 模块提供从 Python 代码访问已验证工作流的编程接口。

### 导入已验证工作流

```python
from latch.verified import (
    bulk_rnaseq,
    deseq2,
    mafft,
    trim_galore,
    alphafold,
    colabfold
)
```

## 核心已验证工作流

### 批量RNA测序分析

**比对与定量：**
```python
from latch.verified import bulk_rnaseq
from latch.types import LatchFile

# 运行批量RNA测序流程
results = bulk_rnaseq(
    fastq_r1=LatchFile("latch:///data/sample_R1.fastq.gz"),
    fastq_r2=LatchFile("latch:///data/sample_R2.fastq.gz"),
    reference_genome="hg38",
    output_dir="latch:///results/rnaseq"
)
```

**功能特性：**
- 使用FastQC进行读段质量控制
- 接头序列修剪
- 使用STAR或HISAT2进行比对
- 通过featureCounts实现基因水平定量
- 生成MultiQC报告

### 差异表达分析

**DESeq2：**
```python
from latch.verified import deseq2
from latch.types import LatchFile

# 运行差异表达分析
results = deseq2(
    count_matrix=LatchFile("latch:///data/counts.csv"),
    sample_metadata=LatchFile("latch:///data/metadata.csv"),
    design_formula="~ condition",
    output_dir="latch:///results/deseq2"
)
```

**功能特性：**
- 标准化与方差稳定化
- 差异表达检验
- MA图与火山图
- PCA可视化
- 带注释的结果表格

### 通路分析

**富集分析：**
```python
from latch.verified import pathway_enrichment

results = pathway_enrichment(
    gene_list=LatchFile("latch:///data/deg_list.txt"),
    organism="human",
    databases=["GO_Biological_Process", "KEGG", "Reactome"],
    output_dir="latch:///results/pathways"
)
```

**支持数据库：**
- 基因本体(GO)
- KEGG通路
- Reactome
- WikiPathways
- MSigDB集合

### 序列比对

**MAFFT多序列比对：**
```python
from latch.verified import mafft
from latch.types import LatchFile

aligned = mafft(
    input_fasta=LatchFile("latch:///data/sequences.fasta"),
    algorithm="auto",
    output_format="fasta"
)
```

**功能特性：**
- 多种比对算法(FFT-NS-1, FFT-NS-2, G-INS-i, L-INS-i)
- 自动算法选择
- 支持大规模比对
- 多种输出格式

### 接头与质量修剪

**Trim Galore：**
```python
from latch.verified import trim_galore

trimmed = trim_galore(
    fastq_r1=LatchFile("latch:///data/sample_R1.fastq.gz"),
    fastq_r2=LatchFile("latch:///data/sample_R2.fastq.gz"),
    quality_threshold=20,
    adapter_auto_detect=True
)
```

**功能特性：**
- 自动接头检测
- 质量修剪
- FastQC集成
- 支持单端与双端测序

## 蛋白质结构预测

### AlphaFold

**标准AlphaFold：**
```python
from latch.verified import alphafold
from latch.types import LatchFile

structure = alphafold(
    sequence_fasta=LatchFile("latch:///data/protein.fasta"),
    model_preset="monomer",
    use_templates=True,
    output_dir="latch:///results/alphafold"
)
```

**功能特性：**
- 单体与多聚体预测
- 基于模板的建模选项
- MSA生成
- 置信度指标(pLDDT, PAE)
- PDB结构输出

**模型预设：**
- `monomer`: 单蛋白链
- `monomer_casp14`: CASP14竞赛版本
- `monomer_ptm`: 带pTM置信度
- `multimer`: 蛋白质复合物

### ColabFold

**优化的AlphaFold替代方案：**
```python
from latch.verified import colabfold

structure = colabfold(
    sequence_fasta=LatchFile("latch:///data/protein.fasta"),
    num_models=5,
    use_amber_relax=True,
    output_dir="latch:///results/colabfold"
)
```

**功能特性：**
- 比标准AlphaFold更快
- 基于MMseqs2的MSA生成
- 多模型预测
- Amber弛豫优化
- 按置信度排序

**优势：**
- MSA生成速度快3-5倍
- 更低计算成本
- 与AlphaFold相当的准确度

## 单细胞分析

### ArchR (scATAC-seq)

**染色质可及性分析：**
```python
from latch.verified import archr

results = archr(
    fragments_file=LatchFile("latch:///data/fragments.tsv.gz"),
    genome="hg38",
    output_dir="latch:///results/archr"
)
```

**功能特性：**
- Arrow文件生成
- 质量控制指标
- 降维处理
- 聚类分析
- 峰识别
- 基序富集

### scVelo (RNA速率)

**RNA速率分析：**
```python
from latch.verified import scvelo

results = scvelo(
    adata_file=LatchFile("latch:///data/adata.h5ad"),
    mode="dynamical",
    output_dir="latch:///results/scvelo"
)
```

**功能特性：**
- 剪接/未剪接定量
- 速率估计
- 动态建模
- 轨迹推断
- 可视化

### emptyDropsR (细胞识别)

**空液滴检测：**
```python
from latch.verified import emptydrops

filtered_matrix = emptydrops(
    raw_matrix_dir=LatchDir("latch:///data/raw_feature_bc_matrix"),
    fdr_threshold=0.01
)
```

**功能特性：**
- 区分细胞与空液滴
- 基于FDR的阈值设定
- 环境RNA去除
- 兼容10X数据

## 基因编辑分析

### CRISPResso2

**CRISPR编辑评估：**
```python
from latch.verified import crispresso2

results = crispresso2(
    fastq_r1=LatchFile("latch:///data/sample_R1.fastq.gz"),
    amplicon_sequence="AGCTAGCTAG...",
    guide_rna="GCTAGCTAGC",
    output_dir="latch:///results/crispresso"
)
```

**功能特性：**
- 插入缺失定量
- 碱基编辑分析
- 先导编辑分析
- HDR定量
- 等位基因频率图谱

## 系统发育学

### 系统发育树构建

```python
from latch.verified import phylogenetics

tree = phylogenetics(
    alignment_file=LatchFile("latch:///data/aligned.fasta"),
    method="maximum_likelihood",
    bootstrap_replicates=1000,
    output_dir="latch:///results/phylo"
)
```

**功能特性：**
- 多种建树方法
- Bootstrap支持度
- 树可视化
- 模型选择

## 工作流集成

### 在自定义流程中使用已验证工作流

```python
from latch import workflow, small_task
from latch.verified import bulk_rnaseq, deseq2
from latch.types import LatchFile, LatchDir

@workflow
def complete_rnaseq_analysis(
    fastq_files: List[LatchFile],
    metadata: LatchFile,
    output_dir: LatchDir
) -> LatchFile:
    """
    使用已验证工作流实现完整RNA-seq分析流程
    """
    # 对每个样本进行比对
    aligned_samples = []
    for fastq in fastq_files:
        result = bulk_rnaseq(
            fastq_r1=fastq,
            reference_genome="hg38",
            output_dir=output_dir
        )
        aligned_samples.append(result)

    # 聚合计数并运行差异表达
    count_matrix = aggregate_counts(aligned_samples)
    deseq_results = deseq2(
        count_matrix=count_matrix,
        sample_metadata=metadata,
        design_formula="~ condition"
    )

    return deseq_results
```

## 最佳实践

### 何时使用已验证工作流

**适用场景：**
1. 标准化分析流程
2. 成熟分析方法
3. 生产级分析
4. 可重复研究
5. 经过验证的生物信息学工具

**需构建自定义工作流场景：**
1. 新型分析方法
2. 定制化预处理步骤
3. 与专有工具集成
4. 实验性流程
5. 高度专业化工作流

### 组合使用已验证与自定义工作流

```python
from latch import workflow, small_task
from latch.verified import alphafold
from latch.types import LatchFile

@small_task
def preprocess_sequence(raw_fasta: LatchFile) -> LatchFile:
    """自定义预处理"""
    # 此处添加定制逻辑
    return processed_fasta

@small_task
def postprocess_structure(pdb_file: LatchFile) -> LatchFile:
    """自定义后处理"""
    # 此处添加定制分析
    return analysis_results

@workflow
def custom_structure_pipeline(input_fasta: LatchFile) -> LatchFile:
    """
    将自定义步骤与已验证AlphaFold结合
    """
    # 自定义预处理
    processed = preprocess_sequence(raw_fasta=input_fasta)

    # 使用已验证AlphaFold
    structure = alphafold(
        sequence_fasta=processed,
        model_preset="monomer_ptm"
    )

    # 自定义后处理
    results = postprocess_structure(pdb_file=structure)

    return results
```

## 访问工作流文档

### 平台内文档

每个已验证工作流包含：
- 参数说明
- 输入/输出规范
- 方法详情
- 引用信息
- 使用示例

### 查看可用工作流

```python
from latch.verified import list_workflows

# 列出所有可用已验证工作流
workflows = list_workflows()

for workflow in workflows:
    print(f"{workflow.name}: {workflow.description}")
```

## 版本管理

### 工作流版本

已验证工作流具备版本维护机制：
- 漏洞修复与改进
- 新增功能
- 保持向后兼容性
- 支持版本锁定

### 使用特定版本

```python
from latch.verified import bulk_rnaseq

# 使用指定版本
results = bulk_rnaseq(
    fastq_r1=input_file,
    reference_genome="hg38",
    workflow_version="2.1.0"
)
```

## 支持与更新

### 获取帮助

- **文档中心**：https://docs.latch.bio
- **Slack社区**：Latch SDK工作区
- **技术支持**：support@latch.bio
- **GitHub Issues**：报告问题与功能请求

### 工作流更新

已验证工作流定期更新：
- 工具版本升级
- 性能优化
- 漏洞修复
- 新增功能

订阅发布说明获取更新通知。

## 典型应用场景

### 完整RNA-seq研究

```python
# 1. 质量控制与比对
aligned = bulk_rnaseq(fastq=samples)

# 2. 差异表达分析
deg = deseq2(counts=aligned)

# 3. 通路富集
pathways = pathway_enrichment(genes=deg)
```

### 蛋白质结构分析

```python
# 1. 结构预测
structure = alphafold(sequence=protein_seq)

# 2. 定制化分析
results = analyze_structure(pdb=structure)
```

### 单细胞工作流

```python
# 1. 细胞过滤
filtered = emptydrops(matrix=raw_counts)

# 2. RNA速率分析
velocity = scvelo(adata=filtered)
```
