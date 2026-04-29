---
name: gget
description: "快速通过CLI/Python查询20多个生物信息学数据库。适用于快速查找：基因信息、BLAST搜索、AlphaFold结构、富集分析。最适合交互式探索和简单查询。批量处理或高级BLAST请使用biopython；多数据库Python工作流请使用bioservices。"
license: BSD-2-Clause license
metadata:
    skill-author: K-Dense Inc.
---

# gget

## 概述

gget 是一个命令行生物信息学工具和Python包，提供对20多个基因组数据库和分析方法的统一访问。通过一致的接口查询基因信息、序列分析、蛋白质结构、表达数据和疾病关联。所有gget模块均可作为命令行工具和Python函数使用。

**重要提示**：gget查询的数据库持续更新，有时会改变其结构。gget模块每两周自动测试一次，必要时更新以匹配新的数据库结构。

## 安装

在干净的虚拟环境中安装gget以避免冲突：

```bash
# 使用uv（推荐）
uv pip install gget

# 或使用pip
pip install --upgrade gget

# 在Python/Jupyter中
import gget
```

## 快速入门

所有模块的基本使用模式：

```bash
# 命令行
gget <模块名> [参数] [选项]

# Python
gget.模块名(参数, 选项)
```

大多数模块返回：
- **命令行**：JSON（默认）或使用`-csv`标志返回CSV
- **Python**：DataFrame或字典

模块通用标志：
- `-o/--out`：保存结果到文件
- `-q/--quiet`：抑制进度信息
- `-csv`：返回CSV格式（仅命令行）

## 模块分类

### 1. 参考与基因信息

#### gget ref - 参考基因组下载

获取Ensembl参考基因组的下载链接和元数据。

**参数**：
- `species`：属_种格式（如'homo_sapiens', 'mus_musculus'）。快捷方式：'human', 'mouse'
- `-w/--which`：指定返回类型（gtf, cdna, dna, cds, cdrna, pep）。默认：全部
- `-r/--release`：Ensembl版本号（默认：最新）
- `-l/--list_species`：列出可用脊椎动物物种
- `-liv/--list_iv_species`：列出可用无脊椎动物物种
- `-ftp`：仅返回FTP链接
- `-d/--download`：下载文件（需要curl）

**示例**：
```bash
# 列出可用物种
gget ref --list_species

# 获取人类所有参考文件
gget ref homo_sapiens

# 仅下载小鼠的GTF注释
gget ref -w gtf -d mouse
```

```python
# Python
gget.ref("homo_sapiens")
gget.ref("mus_musculus", which="gtf", download=True)
```

#### gget search - 基因搜索

通过名称或描述跨物种定位基因。

**参数**：
- `searchwords`：一个或多个搜索词（不区分大小写）
- `-s/--species`：目标物种（如'homo_sapiens', 'mouse'）
- `-r/--release`：Ensembl版本号
- `-t/--id_type`：返回'gene'（默认）或'transcript'
- `-ao/--andor`：'or'（默认）匹配任意搜索词；'and'要求匹配所有
- `-l/--limit`：最大返回结果数

**返回**：ensembl_id, gene_name, ensembl_description, ext_ref_description, biotype, URL

**示例**：
```bash
# 在人类中搜索GABA相关基因
gget search -s human gaba gamma-aminobutyric

# 查找特定基因，要求匹配所有术语
gget search -s mouse -ao and pax7 transcription
```

```python
# Python
gget.search(["gaba", "gamma-aminobutyric"], species="homo_sapiens")
```

#### gget info - 基因/转录本信息

从Ensembl、UniProt和NCBI获取全面的基因和转录本元数据。

**参数**：
- `ens_ids`：一个或多个Ensembl ID（支持WormBase、Flybase ID）。限制：约1000个ID
- `-n/--ncbi`：禁用NCBI数据获取
- `-u/--uniprot`：禁用UniProt数据获取
- `-pdb`：包含PDB标识符（增加运行时间）

**返回**：UniProt ID、NCBI基因ID、主要基因名、别名、蛋白质名称、描述、生物类型、典型转录本

**示例**：
```bash
# 获取多个基因信息
gget info ENSG00000034713 ENSG00000104853 ENSG00000170296

# 包含PDB ID
gget info ENSG00000034713 -pdb
```

```python
# Python
gget.info(["ENSG00000034713", "ENSG00000104853"], pdb=True)
```

#### gget seq - 序列获取

获取基因和转录本的核苷酸或氨基酸序列。

**参数**：
- `ens_ids`：一个或多个Ensembl标识符
- `-t/--translate`：获取氨基酸序列而非核苷酸
- `-iso/--isoforms`：返回所有转录变体（仅限基因ID）

**返回**：FASTA格式序列

**示例**：
```bash
# 获取核苷酸序列
gget seq ENSG00000034713 ENSG00000104853

# 获取所有蛋白质异构体
gget seq -t -iso ENSG00000034713
```

```python
# Python
gget.seq(["ENSG00000034713"], translate=True, isoforms=True)
```

### 2. 序列分析与比对

#### gget blast - BLAST搜索

对标准数据库进行核苷酸或氨基酸序列的BLAST搜索。

**参数**：
- `sequence`：序列字符串或FASTA/.txt文件路径
- `-p/--program`：blastn, blastp, blastx, tblastn, tblastx（自动检测）
- `-db/--database`：
  - 核苷酸：nt, refseq_rna, pdbnt
  - 蛋白质：nr, swissprot, pdbaa, refseq_protein
- `-l/--limit`：最大命中数（默认：50）
- `-e/--expect`：E值阈值（默认：10.0）
- `-lcf/--low_comp_filt`：启用低复杂度过滤
- `-mbo/--megablast_off`：禁用MegaBLAST（仅blastn）

**示例**：
```bash
# BLAST蛋白质序列
gget blast MKWMFKEDHSLEHRCVESAKIRAKYPDRVPVIVEKVSGSQIVDIDKRKYLVPSDITVAQFMWIIRKRIQLPSEKAIFLFVDKTVPQSR

# 从文件BLAST并指定数据库
gget blast sequence.fasta -db swissprot -l 10
```

```python
# Python
gget.blast("MKWMFK...", database="swissprot", limit=10)
```

#### gget blat - BLAT搜索

使用UCSC BLAT定位序列的基因组位置。

**参数**：
- `sequence`：序列字符串或FASTA/.txt文件路径
- `-st/--seqtype`：'DNA', 'protein', 'translated%20RNA', 'translated%20DNA'（自动检测）
- `-a/--assembly`：目标基因组组装（默认：'human'/hg38；选项：'mouse'/mm39, 'zebrafinch'/taeGut2等）

**返回**：基因组、查询大小、比对位置、匹配数、错配数、比对百分比

**示例**：
```bash
# 在人类基因组中定位
gget blat ATCGATCGATCGATCG

# 在其他组装中搜索
gget blat -a mm39 ATCGATCGATCGATCG
```

```python
# Python
gget.blat("ATCGATCGATCGATCG", assembly="mouse")
```

#### gget muscle - 多序列比对

使用Muscle5比对多个核苷酸或氨基酸序列。

**参数**：
- `fasta`：序列或FASTA/.txt文件路径
- `-s5/--super5`：使用Super5算法加速处理（大型数据集）

**返回**：ClustalW格式或对齐的FASTA（.afa）

**示例**：
```bash
# 从文件比对序列
gget muscle sequences.fasta -o aligned.afa

# 大型数据集使用Super5
gget muscle large_dataset.fasta -s5
```

```python
# Python
gget.muscle("sequences.fasta", save=True)
```

#### gget diamond - 局部序列比对

使用DIAMOND进行快速的蛋白质或翻译DNA局部比对。

**参数**：
- Query：序列（字符串/列表）或FASTA文件路径
- `--reference`：参考序列（字符串/列表）或FASTA文件路径（必需）
- `--sensitivity`：fast, mid-sensitive, sensitive, more-sensitive, very-sensitive（默认）, ultra-sensitive
- `--threads`：CPU线程数（默认：1）
- `--diamond_db`：保存数据库以供重用
- `--translated`：启用核苷酸到氨基酸的比对

**返回**：一致性百分比、序列长度、匹配位置、缺口开放数、E值、比特分数

**示例**：
```bash
# 比对参考序列
gget diamond GGETISAWESQME -ref reference.fasta --threads 4

# 保存数据库供重用
gget diamond query.fasta -ref ref.fasta --diamond_db my_db.dmnd
```

```python
# Python
gget.diamond("GGETISAWESQME", reference="reference.fasta", threads=4)
```

### 3. 结构与蛋白质分析

#### gget pdb - 蛋白质结构

查询RCSB蛋白质数据库获取结构和元数据。

**参数**：
- `pdb_id`：PDB标识符（如'7S7U'）
- `-r/--resource`：数据类型（pdb, entry, pubmed, assembly, entity types）
- `-i/--identifier`：组装、实体或链ID

**返回**：PDB格式（结构）或JSON（元数据）

**示例**：
```bash
# 下载PDB结构
gget pdb 7S7U -o 7S7U.pdb

# 获取元数据
gget pdb 7S7U -r entry
```

```python
# Python
gget.pdb("7S7U", save=True)
```

#### gget alphafold - 蛋白质结构预测

使用简化的AlphaFold2预测3D蛋白质结构。

**安装要求**：
```bash
# 先安装OpenMM
pip install openmm

# 然后设置AlphaFold
gget setup alphafold
```

**参数**：
- `sequence`：氨基酸序列（字符串）、多序列（列表）或FASTA文件。多序列触发多聚体建模
- `-mr/--multimer_recycles`：循环迭代次数（默认：3；推荐20以提高精度）
- `-mfm/--multimer_for_monomer`：对单体应用多聚体模型
- `-r/--relax`：对顶级模型进行AMBER松弛
- `plot`：仅Python；生成交互式3D可视化（默认：True）
- `show_sidechains`：仅Python；包含侧链（默认：True）

**返回**：PDB结构文件、JSON比对误差数据、可选的3D可视化

**示例**：
```bash
# 预测单蛋白结构
gget alphafold MKWMFKEDHSLEHRCVESAKIRAKYPDRVPVIVEKVSGSQIVDIDKRKYLVPSDITVAQFMWIIRKRIQLPSEKAIFLFVDKTVPQSR

# 高精度预测多聚体
gget alphafold sequence1.fasta -mr 20 -r
```

```python
# Python带可视化
gget.alphafold("MKWMFK...", plot=True, show_sidechains=True)

# 多聚体预测
gget.alphafold(["sequence1", "sequence2"], multimer_recycles=20)
```

#### gget elm - 真核生物线性模体

预测蛋白质序列中的真核生物线性模体。

**安装要求**：
```bash
gget setup elm
```

**参数**：
- `sequence`：氨基酸序列或UniProt编号
- `-u/--uniprot`：指示序列为UniProt编号
- `-e/--expand`：包含蛋白质名称、生物体、参考文献
- `-s/--sensitivity`：DIAMOND比对敏感度（默认："very-sensitive"）
- `-t/--threads`：线程数（默认：1）

**返回**：两个输出：
1. **ortholog_df**：来自直系同源蛋白的线性模体
2. **regex_df**：输入序列中直接匹配的模体

**示例**：
```bash
# 从序列预测模体
gget elm LIAQSIGQASFV -o results

# 使用UniProt编号并扩展信息
gget elm --uniprot Q02410 -e
```

```python
# Python
ortholog_df, regex_df = gget.elm("LIAQSIGQASFV")
```

### 4. 表达与疾病数据

#### gget archs4 - 基因相关性及组织表达

查询ARCHS4数据库获取相关基因或组织表达数据。

**参数**：
- `gene`：基因符号或Ensembl ID（需`--ensembl`标志）
- `-w/--which`：'correlation'（默认，返回100个最相关基因）或'tissue'（表达图谱）
- `-s/--species`：'human'（默认）或'mouse'（仅组织数据）
- `-e/--ensembl`：输入为Ensembl ID

**返回**：
- **相关性模式**：基因符号、皮尔逊相关系数
- **组织模式**：组织标识符、最小/Q1/中位数/Q3/最大表达值

**示例**：
```bash
# 获取相关基因
gget archs4 ACE2

# 获取组织表达
gget archs4 -w tissue ACE2
```

```python
# Python
gget.archs4("ACE2", which="tissue")
```

#### gget cellxgene - 单细胞RNA-seq数据

查询CZ CELLxGENE Discover Census获取单细胞数据。

**安装要求**：
```bash
gget setup cellxgene
```

**参数**：
- `--gene` (-g)：基因名或Ensembl ID（区分大小写！人类用'PAX7'，小鼠用'Pax7'）
- `--tissue`：组织类型
- `--cell_type`：特定细胞类型
- `--species` (-s)：'homo_sapiens'（默认）或'mus_musculus'
- `--census_version` (-cv)：版本（"stable", "latest"或日期）
- `--ensembl` (-e)：使用Ensembl ID
- `--meta_only` (-mo)：仅返回元数据
- 附加筛选器：disease, development_stage, sex, assay, dataset_id, donor_id, ethnicity, suspension_type

**返回**：包含计数矩阵和元数据的AnnData对象（或仅元数据的数据框）

**示例**：
```bash
# 获取特定基因和细胞类型的单细胞数据
gget cellxgene --gene ACE2 ABCA1 --tissue lung --cell_type "mucus secreting cell" -o lung_data.h5ad

# 仅元数据
gget cellxgene --gene PAX7 --tissue muscle --meta_only -o metadata.csv
```

```python
# Python
adata = gget.cellxgene(gene=["ACE2", "ABCA1"], tissue="lung", cell_type="mucus secreting cell")
```

#### gget enrichr - 富集分析

使用Enrichr对基因列表进行本体富集分析。

**参数**：
- `genes`：基因符号或Ensembl ID
- `-db/--database`：参考数据库（支持快捷方式：'pathway', 'transcription', 'ontology', 'diseases_drugs', 'celltypes'）
- `-s/--species`：human（默认）, mouse, fly, yeast, worm, fish
- `-bkg_l/--background_list`：用于比较的背景基因
- `-ko/--kegg_out`：保存带高亮基因的KEGG通路图像
- `plot`：仅Python；生成图形结果

**数据库快捷方式**：
- 'pathway' → KEGG_2021_Human
- 'transcription' → ChEA_2016
- 'ontology' → GO_Biological_Process_2021
- 'diseases_drugs' → GWAS_Catalog_2019

# 保存KEGG通路
gget enrichr -db pathway ACE2 AGT AGTR1 -ko ./kegg_images/
```

```python
# Python绘图模式
gget.enrichr(["ACE2", "AGT", "AGTR1"], database="ontology", plot=True)
```

#### gget bgee - 同源性与表达数据

从Bgee数据库检索同源性和基因表达数据。

**参数**:
- `ens_id`: Ensembl基因ID或NCBI基因ID（非Ensembl物种）。当`type=expression`时支持多个ID
- `-t/--type`: 'orthologs'（默认）或'expression'

**返回**:
- **同源性模式**: 跨物种匹配基因的ID、名称、分类信息
- **表达模式**: 解剖实体、置信度分数、表达状态

**示例**:
```bash
# 获取同源基因
gget bgee ENSG00000169194

# 获取表达数据
gget bgee ENSG00000169194 -t expression

# 多基因查询
gget bgee ENSBTAG00000047356 ENSBTAG00000018317 -t expression
```

```python
# Python
gget.bgee("ENSG00000169194", type="orthologs")
```

#### gget opentargets - 疾病与药物关联

从OpenTargets检索疾病和药物关联信息。

**参数**:
- Ensembl基因ID（必需）
- `-r/--resource`: diseases（默认）、drugs、tractability、pharmacogenetics、expression、depmap、interactions
- `-l/--limit`: 结果数量上限
- 筛选参数（因资源类型而异）:
  - drugs: `--filter_disease`
  - pharmacogenetics: `--filter_drug`
  - expression/depmap: `--filter_tissue`, `--filter_anat_sys`, `--filter_organ`
  - interactions: `--filter_protein_a`, `--filter_protein_b`, `--filter_gene_b`

**示例**:
```bash
# 获取关联疾病
gget opentargets ENSG00000169194 -r diseases -l 5

# 获取关联药物
gget opentargets ENSG00000169194 -r drugs -l 10

# 获取组织表达
gget opentargets ENSG00000169194 -r expression --filter_tissue brain
```

```python
# Python
gget.opentargets("ENSG00000169194", resource="diseases", limit=5)
```

#### gget cbio - cBioPortal癌症基因组学

使用cBioPortal数据绘制癌症基因组学热图。

**两个子命令**:

**search** - 查找研究ID:
```bash
gget cbio search breast lung
```

**plot** - 生成热图:

**参数**:
- `-s/--study_ids`: 空格分隔的cBioPortal研究ID（必需）
- `-g/--genes`: 空格分隔的基因名或Ensembl ID（必需）
- `-st/--stratification`: 数据组织列（tissue, cancer_type, cancer_type_detailed, study_id, sample）
- `-vt/--variation_type`: 数据类型（mutation_occurrences, cna_nonbinary, sv_occurrences, cna_occurrences, Consequence）
- `-f/--filter`: 按列值筛选（如'study_id:msk_impact_2017'）
- `-dd/--data_dir`: 缓存目录（默认：./gget_cbio_cache）
- `-fd/--figure_dir`: 输出目录（默认：./gget_cbio_figures）
- `-dpi`: 分辨率（默认：100）
- `-sh/--show`: 在窗口中显示图表
- `-nc/--no_confirm`: 跳过下载确认

**示例**:
```bash
# 搜索研究
gget cbio search esophag ovary

# 创建热图
gget cbio plot -s msk_impact_2017 -g AKT1 ALK BRAF -st tissue -vt mutation_occurrences
```

```python
# Python
gget.cbio_search(["esophag", "ovary"])
gget.cbio_plot(["msk_impact_2017"], ["AKT1", "ALK"], stratification="tissue")
```

#### gget cosmic - COSMIC数据库

搜索COSMIC（体细胞突变目录）数据库。

**重要**: 商业用途需支付许可费。需要COSMIC账户凭证。

**参数**:
- `searchterm`: 基因名、Ensembl ID、突变符号或样本ID
- `-ctp/--cosmic_tsv_path`: COSMIC TSV文件路径（查询必需）
- `-l/--limit`: 最大结果数（默认：100）

**数据库下载标志**:
- `-d/--download_cosmic`: 激活下载模式
- `-gm/--gget_mutate`: 创建gget mutate专用版本
- `-cp/--cosmic_project`: 数据库类型（cancer, census, cell_line, resistance, genome_screen, targeted_screen）
- `-cv/--cosmic_version`: COSMIC版本
- `-gv/--grch_version`: 人类参考基因组（37或38）
- `--email`, `--password`: COSMIC凭证

**示例**:
```bash
# 先下载数据库
gget cosmic -d --email user@example.com --password xxx -cp cancer

# 再查询
gget cosmic EGFR -ctp cosmic_data.tsv -l 10
```

```python
# Python
gget.cosmic("EGFR", cosmic_tsv_path="cosmic_data.tsv", limit=10)
```

### 5. 附加工具

#### gget mutate - 生成突变序列

根据突变注释生成突变核苷酸序列。

**参数**:
- `sequences`: FASTA文件路径或直接序列输入（字符串/列表）
- `-m/--mutations`: 包含突变数据的CSV/TSV文件或DataFrame（必需）
- `-mc/--mut_column`: 突变列名（默认：'mutation'）
- `-sic/--seq_id_column`: 序列ID列（默认：'seq_ID'）
- `-mic/--mut_id_column`: 突变ID列
- `-k/--k`: 侧翼序列长度（默认：30个核苷酸）

**返回**: FASTA格式的突变序列

**示例**:
```bash
# 单突变
gget mutate ATCGCTAAGCT -m "c.4G>T"

# 从文件读取多序列突变
gget mutate sequences.fasta -m mutations.csv -o mutated.fasta
```

```python
# Python
import pandas as pd
mutations_df = pd.DataFrame({"seq_ID": ["seq1"], "mutation": ["c.4G>T"]})
gget.mutate(["ATCGCTAAGCT"], mutations=mutations_df)
```

#### gget gpt - OpenAI文本生成

使用OpenAI API生成自然语言文本。

**必需设置**:
```bash
gget setup gpt
```

**重要**: 免费层级限账户创建后3个月。请设置月度账单限额。

**参数**:
- `prompt`: 生成用文本输入（必需）
- `api_key`: OpenAI认证密钥（必需）
- 模型配置: temperature, top_p, max_tokens, frequency_penalty, presence_penalty
- 默认模型: gpt-3.5-turbo（可配置）

**示例**:
```bash
gget gpt "解释CRISPR技术" --api_key your_key_here
```

```python
# Python
gget.gpt("解释CRISPR技术", api_key="your_key_here")
```

#### gget setup - 安装依赖

为特定模块安装/下载第三方依赖。

**参数**:
- `module`: 需要安装依赖的模块名
- `-o/--out`: 输出文件夹路径（仅elm模块）

**需要设置的模块**:
- `alphafold` - 下载约4GB模型参数
- `cellxgene` - 安装cellxgene-census（可能不支持最新Python）
- `elm` - 下载本地ELM数据库
- `gpt` - 配置OpenAI集成

**示例**:
```bash
# 设置AlphaFold
gget setup alphafold

# 自定义目录设置ELM
gget setup elm -o /path/to/elm_data
```

```python
# Python
gget.setup("alphafold")
```

## 常用工作流

### 工作流1: 基因发现到序列分析

查找并分析目标基因:

```python
# 1. 搜索基因
results = gget.search(["GABA", "receptor"], species="homo_sapiens")

# 2. 获取详细信息
gene_ids = results["ensembl_id"].tolist()
info = gget.info(gene_ids[:5])

# 3. 检索序列
sequences = gget.seq(gene_ids[:5], translate=True)
```

### 工作流2: 序列比对与结构预测

比对序列并预测结构:

```python
# 1. 多序列比对
alignment = gget.muscle("sequences.fasta")

# 2. 查找相似序列
blast_results = gget.blast(my_sequence, database="swissprot", limit=10)

# 3. 预测结构
structure = gget.alphafold(my_sequence, plot=True)

# 4. 寻找线性模体
ortholog_df, regex_df = gget.elm(my_sequence)
```

### 工作流3: 基因表达与富集分析

分析表达模式与功能富集:

```python
# 1. 获取组织表达
tissue_expr = gget.archs4("ACE2", which="tissue")

# 2. 查找相关基因
correlated = gget.archs4("ACE2", which="correlation")

# 3. 获取单细胞数据
adata = gget.cellxgene(gene=["ACE2"], tissue="lung", cell_type="epithelial cell")

# 4. 执行富集分析
gene_list = correlated["gene_symbol"].tolist()[:50]
enrichment = gget.enrichr(gene_list, database="ontology", plot=True)
```

### 工作流4: 疾病与药物分析

研究疾病关联与治疗靶点:

```python
# 1. 搜索基因
genes = gget.search(["breast cancer"], species="homo_sapiens")

# 2. 获取疾病关联
diseases = gget.opentargets("ENSG00000169194", resource="diseases")

# 3. 获取药物关联
drugs = gget.opentargets("ENSG00000169194", resource="drugs")

# 4. 查询癌症基因组数据
study_ids = gget.cbio_search(["breast"])
gget.cbio_plot(study_ids[:2], ["BRCA1", "BRCA2"], stratification="cancer_type")

# 5. COSMIC突变搜索
cosmic_results = gget.cosmic("BRCA1", cosmic_tsv_path="cosmic.tsv")
```

### 工作流5: 比较基因组学

跨物种比较蛋白质:

```python
# 1. 获取同源基因
orthologs = gget.bgee("ENSG00000169194", type="orthologs")

# 2. 获取比对序列
human_seq = gget.seq("ENSG00000169194", translate=True)
mouse_seq = gget.seq("ENSMUSG00000026091", translate=True)

# 3. 序列比对
alignment = gget.muscle([human_seq, mouse_seq])

# 4. 比较结构
human_structure = gget.pdb("7S7U")
mouse_structure = gget.alphafold(mouse_seq)
```

### 工作流6: 构建参考索引

为下游分析准备参考数据（如kallisto|bustools）:

```bash
# 1. 列出可用物种
gget ref --list_species

# 2. 下载参考文件
gget ref -w gtf -w cdna -d homo_sapiens

# 3. 构建kallisto索引
kallisto index -i transcriptome.idx transcriptome.fasta

# 4. 下载比对用基因组
gget ref -w dna -d homo_sapiens
```

## 最佳实践

### 数据检索
- 使用`--limit`控制大型查询结果规模
- 用`-o/--out`保存结果确保可复现性
- 检查数据库版本/发布确保分析一致性
- 生产脚本中使用`--quiet`减少输出

### 序列分析
- BLAST/BLAT从默认参数开始，再调整灵敏度
- 使用`gget diamond`搭配`--threads`加速本地比对
- 用`--diamond_db`保存DIAMOND数据库供重复查询
- 大型数据集多序列比对使用`-s5/--super5`

### 表达与疾病数据
- cellxgene中基因符号区分大小写（如'PAX7' vs 'Pax7'）
- 首次使用alphafold/cellxgene/elm/gpt前运行`gget setup`
- 富集分析使用数据库快捷方式提高效率
- 用`-dd`缓存cBioPortal数据避免重复下载

### 结构预测
- AlphaFold多聚体预测：用`-mr 20`提高准确度
- 最终结构优化使用`-r`标志进行AMBER弛豫
- Python中用`plot=True`可视化结果
- 运行AlphaFold前先检查PDB数据库

### 错误处理
- 数据库结构会变更：定期更新gget `uv pip install --upgrade gget`
- gget info单次处理最多约1000个Ensembl ID
- 大规模分析需实现API查询速率限制
- 使用虚拟环境避免依赖冲突

## 输出格式

### 命令行
- 默认：JSON
- CSV：添加`-csv`标志
- FASTA：gget seq, gget mutate
- PDB：gget pdb, gget alphafold
- PNG：gget cbio plot

### Python
- 默认：DataFrame或字典
- JSON：添加`json=True`参数
- 保存文件：添加`save=True`或指定`out="文件名"`
- AnnData：gget cellxgene

## 资源

本技能包含详细模块参考文档：

### references/
- `module_reference.md` - 所有模块的完整参数参考
- `database_info.md` - 数据库信息及更新频率
- `workflows.md` - 扩展工作流示例与用例

更多帮助：
- 官方文档：https://pachterlab.github.io/gget/
- GitHub问题：https://github.com/pachterlab/gget/issues
- 引用文献：Luebbert, L. & Pachter, L. (2023). Efficient querying of genomic reference databases with gget. Bioinformatics. https://doi.org/10.1093/bioinformatics/btac836
