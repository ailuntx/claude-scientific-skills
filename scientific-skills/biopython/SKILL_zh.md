---
name: biopython
description: 全面的分子生物学工具包。用于序列操作、文件解析（FASTA/GenBank/PDB）、系统发育分析以及程序化访问NCBI/PubMed（Bio.Entrez）。最适合批处理、定制生物信息学流程和BLAST自动化。快速查询推荐gget；多服务集成推荐bioservices。
license: 未知
metadata:
    skill-author: K-Dense Inc.
---

# Biopython：Python中的计算分子生物学

## 概述

Biopython是一套全面且免费提供的生物计算Python工具集。它提供序列操作、文件I/O、数据库访问、结构生物信息学、系统发育学及其他众多生物信息学任务的功能。当前版本为**Biopython 1.85**（2025年1月发布），支持Python 3并依赖NumPy。

## 使用场景

在以下场景使用本技能：

- 处理生物序列（DNA、RNA或蛋白质）
- 读写或转换生物文件格式（FASTA、GenBank、FASTQ、PDB、mmCIF等）
- 通过Entrez访问NCBI数据库（GenBank、PubMed、Protein、Gene等）
- 运行BLAST搜索或解析BLAST结果
- 执行序列比对（双序列或多序列比对）
- 从PDB文件分析蛋白质结构
- 创建、操作或可视化系统发育树
- 查找序列模体或分析模体模式
- 计算序列统计量（GC含量、分子量、解链温度等）
- 执行结构生物信息学任务
- 处理群体遗传学数据
- 其他任何计算分子生物学任务

## 核心功能

Biopython采用模块化子包设计，覆盖特定生物信息学领域：

1. **序列处理** - Bio.Seq和Bio.SeqIO用于序列操作与文件I/O
2. **比对分析** - Bio.Align和Bio.AlignIO用于双序列与多序列比对
3. **数据库访问** - Bio.Entrez用于程序化访问NCBI数据库
4. **BLAST操作** - Bio.Blast用于运行和解析BLAST搜索
5. **结构生物信息学** - Bio.PDB用于处理3D蛋白质结构
6. **系统发育学** - Bio.Phylo用于系统发育树操作与可视化
7. **高级功能** - 模体分析、群体遗传学、序列工具等

## 安装配置

通过pip安装Biopython（需Python 3和NumPy）：

```python
uv pip install biopython
```

访问NCBI数据库时需设置邮箱（NCBI强制要求）：

```python
from Bio import Entrez
Entrez.email = "your.email@example.com"

# 可选：API密钥可提升请求速率（从3次/秒升至10次/秒）
Entrez.api_key = "your_api_key_here"
```

## 使用指南

本技能提供按功能分类的完整文档。执行任务时请查阅对应参考文档：

### 1. 序列处理（Bio.Seq & Bio.SeqIO）

**参考文档：** `references/sequence_io.md`

适用场景：
- 创建与操作生物序列
- 读写序列文件（FASTA、GenBank、FASTQ等）
- 文件格式转换
- 从大型文件提取序列
- 序列翻译、转录与反向互补
- 使用SeqRecord对象

**快速示例：**
```python
from Bio import SeqIO

# 从FASTA文件读取序列
for record in SeqIO.parse("sequences.fasta", "fasta"):
    print(f"{record.id}: {len(record.seq)} bp")

# 将GenBank转为FASTA
SeqIO.convert("input.gb", "genbank", "output.fasta", "fasta")
```

### 2. 比对分析（Bio.Align & Bio.AlignIO）

**参考文档：** `references/alignment.md`

适用场景：
- 双序列比对（全局与局部）
- 读写多序列比对
- 使用替换矩阵（BLOSUM、PAM）
- 计算比对统计量
- 自定义比对参数

**快速示例：**
```python
from Bio import Align

# 双序列比对
aligner = Align.PairwiseAligner()
aligner.mode = 'global'
alignments = aligner.align("ACCGGT", "ACGGT")
print(alignments[0])
```

### 3. 数据库访问（Bio.Entrez）

**参考文档：** `references/databases.md`

适用场景：
- 搜索NCBI数据库（PubMed、GenBank、Protein、Gene等）
- 下载序列与记录
- 获取文献信息
- 跨数据库查找关联记录
- 带速率限制的批量下载

**快速示例：**
```python
from Bio import Entrez
Entrez.email = "your.email@example.com"

# 搜索PubMed
handle = Entrez.esearch(db="pubmed", term="biopython", retmax=10)
results = Entrez.read(handle)
handle.close()
print(f"找到 {results['Count']} 条结果")
```

### 4. BLAST操作（Bio.Blast）

**参考文档：** `references/blast.md`

适用场景：
- 通过NCBI网络服务运行BLAST搜索
- 运行本地BLAST搜索
- 解析BLAST XML输出
- 按E值或相似度过滤结果
- 提取命中序列

**快速示例：**
```python
from Bio.Blast import NCBIWWW, NCBIXML

# 运行BLAST搜索
result_handle = NCBIWWW.qblast("blastn", "nt", "ATCGATCGATCG")
blast_record = NCBIXML.read(result_handle)

# 显示前5个命中
for alignment in blast_record.alignments[:5]:
    print(f"{alignment.title}: E值={alignment.hsps[0].expect}")
```

### 5. 结构生物信息学（Bio.PDB）

**参考文档：** `references/structure.md`

适用场景：
- 解析PDB与mmCIF结构文件
- 导航蛋白质结构层级（SMCRA：结构/模型/链/残基/原子）
- 计算距离、角度和二面角
- 二级结构分配（DSSP）
- 结构叠合与RMSD计算
- 从结构提取序列

**快速示例：**
```python
from Bio.PDB import PDBParser

# 解析结构
parser = PDBParser(QUIET=True)
structure = parser.get_structure("1crn", "1crn.pdb")

# 计算α碳原子间距
chain = structure[0]["A"]
distance = chain[10]["CA"] - chain[20]["CA"]
print(f"距离: {distance:.2f} Å")
```

### 6. 系统发育学（Bio.Phylo）

**参考文档：** `references/phylogenetics.md`

适用场景：
- 读写系统发育树（Newick、NEXUS、phyloXML）
- 基于距离矩阵或比对构建树
- 树操作（剪枝、重定根、阶梯化）
- 计算系统发育距离
- 创建共识树
- 可视化树结构

**快速示例：**
```python
from Bio import Phylo

# 读取并可视化树
tree = Phylo.read("tree.nwk", "newick")
Phylo.draw_ascii(tree)

# 计算距离
distance = tree.distance("Species_A", "Species_B")
print(f"距离: {distance:.3f}")
```

### 7. 高级功能

**参考文档：** `references/advanced.md`

适用场景：
- **序列模体**（Bio.motifs） - 查找与分析模体模式
- **群体遗传学**（Bio.PopGen） - GenePop文件、Fst计算、哈迪-温伯格检验
- **序列工具**（Bio.SeqUtils） - GC含量、解链温度、分子量、蛋白质分析
- **限制性分析**（Bio.Restriction） - 查找限制性酶切位点
- **聚类分析**（Bio.Cluster） - K均值与层次聚类
- **基因组图谱**（GenomeDiagram） - 可视化基因组特征

**快速示例：**
```python
from Bio.SeqUtils import gc_fraction, molecular_weight
from Bio.Seq import Seq

seq = Seq("ATCGATCGATCG")
print(f"GC含量: {gc_fraction(seq):.2%}")
print(f"分子量: {molecular_weight(seq, seq_type='DNA'):.2f} g/mol")
```

## 通用工作流程指南

### 文档查阅

处理用户特定Biopython任务时：

1. **根据任务描述确定相关模块**
2. **使用Read工具阅读对应参考文件**
3. **提取代码模式并适配用户需求**
4. **复杂任务需组合多个模块**

参考文件搜索模式示例：
```bash
# 查找特定函数信息
grep -n "SeqIO.parse" references/sequence_io.md

# 查找任务示例
grep -n "BLAST" references/blast.md

# 查找概念说明
grep -n "alignment" references/alignment.md
```

### 编写Biopython代码

遵循以下原则：

1. **显式导入模块**
   ```python
   from Bio import SeqIO, Entrez
   from Bio.Seq import Seq
   ```

2. **使用NCBI数据库时设置邮箱**
   ```python
   Entrez.email = "your.email@example.com"
   ```

3. **选用合适文件格式**
   ```python
   # 常用格式: "fasta", "genbank", "fastq", "clustal", "phylip"
   ```

4. **规范文件处理** - 使用后关闭句柄或上下文管理器
   ```python
   with open("file.fasta") as handle:
       records = SeqIO.parse(handle, "fasta")
   ```

5. **大文件使用迭代器** - 避免内存过载
   ```python
   for record in SeqIO.parse("large_file.fasta", "fasta"):
       # 逐条处理记录
   ```

6. **优雅处理错误** - 网络操作与文件解析可能失败
   ```python
   try:
       handle = Entrez.efetch(db="nucleotide", id=accession)
   except HTTPError as e:
       print(f"错误: {e}")
   ```

## 常用模式

### 模式1：从GenBank获取序列

```python
from Bio import Entrez, SeqIO

Entrez.email = "your.email@example.com"

# 获取序列
handle = Entrez.efetch(db="nucleotide", id="EU490707", rettype="gb", retmode="text")
record = SeqIO.read(handle, "genbank")
handle.close()

print(f"描述: {record.description}")
print(f"序列长度: {len(record.seq)}")
```

### 模式2：序列分析流程

```python
from Bio import SeqIO
from Bio.SeqUtils import gc_fraction

for record in SeqIO.parse("sequences.fasta", "fasta"):
    # 计算统计量
    gc = gc_fraction(record.seq)
    length = len(record.seq)

    # 查找ORF、翻译等
    protein = record.seq.translate()

    print(f"{record.id}: {length} bp, GC={gc:.2%}")
```

### 模式3：BLAST搜索并获取前位命中

```python
from Bio.Blast import NCBIWWW, NCBIXML
from Bio import Entrez, SeqIO

Entrez.email = "your.email@example.com"

# 运行BLAST
result_handle = NCBIWWW.qblast("blastn", "nt", sequence)
blast_record = NCBIXML.read(result_handle)

# 获取前5位命中编号
accessions = [aln.accession for aln in blast_record.alignments[:5]]

# 获取序列
for acc in accessions:
    handle = Entrez.efetch(db="nucleotide", id=acc, rettype="fasta", retmode="text")
    record = SeqIO.read(handle, "fasta")
    handle.close()
    print(f">{record.description}")
```

### 模式4：基于序列构建系统发育树

```python
from Bio import AlignIO, Phylo
from Bio.Phylo.TreeConstruction import DistanceCalculator, DistanceTreeConstructor

# 读取比对
alignment = AlignIO.read("alignment.fasta", "fasta")

# 计算距离
calculator = DistanceCalculator("identity")
dm = calculator.get_distance(alignment)

# 构建树
constructor = DistanceTreeConstructor()
tree = constructor.nj(dm)

# 可视化
Phylo.draw_ascii(tree)
```

## 最佳实践

1. **编码前必读相关参考文档**
2. **使用grep搜索参考文件**中的函数或示例
3. **解析前验证文件格式**
4. **优雅处理缺失数据** - 非所有记录均含全字段
5. **缓存下载数据** - 避免重复下载相同序列
6. **遵守NCBI速率限制** - 使用API密钥与适当延迟
7. **先用小数据集测试**再处理大文件
8. **保持Biopython更新**以获取新功能与修复
9. **翻译时选用正确密码子表**
10. **记录分析参数**确保可复现性

## 常见问题排查

### 问题："No handlers could be found for logger 'Bio.Entrez'"
**解决：** 仅为警告，设置Entrez.email即可消除。

### 问题：NCBI返回"HTTP Error 400"
**解决：** 检查ID/编号是否有效且格式正确。

### 问题：解析文件时出现"ValueError: EOF"
**解决：** 确认文件格式与声明格式匹配。

### 问题：比对失败提示"sequences are not the same length"
**解决：** 使用AlignIO或多序列比对前确保序列已对齐。

### 问题：BLAST搜索缓慢
**解决：** 大规模搜索使用本地BLAST或缓存结果。

### 问题：PDB解析器警告
**解决：** 使用`PDBParser(QUIET=True)`屏蔽警告，或检查结构质量。

## 附加资源

- **官方文档**: https://biopython.org/docs/latest/
- **教程**: https://biopython.org/docs/latest/Tutorial/
- **进阶指南**: https://biopython.org/docs/latest/Tutorial/ (高级示例)
- **GitHub**: https://github.com/biopython/biopython
- **邮件列表**: biopython@biopython.org

## 速查指南

在参考文件中定位信息：

```bash
# 搜索特定函数
grep -n "function_name" references/*.md

# 查找任务示例
grep -n "example" references/sequence_io.md

# 查找模块所有引用
grep -n "Bio.Seq" references/*.md
```

## 总结

Biopython提供全面的计算分子生物学工具。使用本技能时：

1. **确定任务领域**（序列、比对、数据库、BLAST、结构、系统发育或高级功能）
2. **查阅`references/`目录对应参考文件**
3. **调整代码示例**适配具体场景
4. **复杂流程组合多个模块**
5. **遵循最佳实践**处理文件、错误检查与数据管理

模块化参考文档确保为每个Biopython核心功能提供可搜索的详细信息。
