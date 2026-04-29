---
name: scikit-bio
description: 生物数据处理工具包。支持序列分析、比对、系统发育树、多样性指标（α/β多样性、UniFrac）、排序分析（PCoA）、PERMANOVA、FASTA/Newick输入输出，用于微生物组分析。
license: BSD-3-Clause 许可证
metadata:
    skill-author: K-Dense Inc.
---

# scikit-bio

## 概述

scikit-bio 是一个用于处理生物数据的综合性 Python 库。该技能适用于生物信息学分析，涵盖序列操作、比对、系统发育学、微生物生态学及多元统计分析。

## 适用场景

当用户遇到以下情况时应使用此技能：
- 处理生物序列（DNA、RNA、蛋白质）
- 需读写生物文件格式（FASTA、FASTQ、GenBank、Newick、BIOM 等）
- 执行序列比对或模体搜索
- 构建或分析系统发育树
- 计算多样性指标（α/β多样性、UniFrac距离）
- 执行排序分析（PCoA、CCA、RDA）
- 对生物/生态数据进行统计检验（PERMANOVA、ANOSIM、Mantel）
- 分析微生物组或群落生态数据
- 处理语言模型生成的蛋白质嵌入向量
- 需操作生物数据表格

## 核心功能

### 1. 序列操作

使用 DNA、RNA 和蛋白质专用类处理生物序列。

**关键操作：**
- 从 FASTA、FASTQ、GenBank、EMBL 格式读写序列
- 序列切片、拼接与搜索
- 反向互补、转录（DNA→RNA）及翻译（RNA→蛋白质）
- 使用正则表达式查找模体
- 计算距离（汉明距离、基于k-mer）
- 处理序列质量分数与元数据

**常用模式：**
```python
import skbio

# 从文件读取序列
seq = skbio.DNA.read('input.fasta')

# 序列操作
rc = seq.reverse_complement()
rna = seq.transcribe()
protein = rna.translate()

# 查找模体
motif_positions = seq.find_with_regex('ATG[ACGT]{3}')

# 检查序列属性
has_degens = seq.has_degenerates()
seq_no_gaps = seq.degap()
```

**重要提示：**
- 使用 `DNA`、`RNA`、`Protein` 类处理带语法验证的序列
- 使用 `Sequence` 类处理无字母表限制的通用序列
- FASTQ 文件的质量分数自动加载为位置元数据
- 元数据类型：序列级（ID、描述）、位置级（每碱基）、区间级（区域/特征）

### 2. 序列比对

通过动态规划算法执行成对与多重序列比对。

**核心能力：**
- 全局比对（Needleman-Wunsch，含半全局变体）
- 局部比对（Smith-Waterman）
- 可配置计分方案（匹配/错配、空位罚分、替换矩阵）
- CIGAR 字符串转换
- 通过 `TabularMSA` 存储和操作多重序列比对

**常用模式：**
```python
from skbio.alignment import local_pairwise_align_ssw, TabularMSA

# 成对比对
alignment = local_pairwise_align_ssw(seq1, seq2)

# 访问比对序列
msa = alignment.aligned_sequences

# 从文件读取多重比对
msa = TabularMSA.read('alignment.fasta', constructor=skbio.DNA)

# 计算共有序列
consensus = msa.consensus()
```

**重要提示：**
- 使用 `local_pairwise_align_ssw` 进行局部比对（更快，基于SSW）
- 使用 `StripedSmithWaterman` 处理蛋白质比对
- 推荐对生物序列使用仿射空位罚分
- 支持 scikit-bio、BioPython 和 Biotite 比对格式互转

### 3. 系统发育树

构建、操作和分析表示进化关系的系统发育树。

**核心能力：**
- 基于距离矩阵建树（UPGMA、WPGMA、邻接法、GME、BME）
- 树操作（剪枝、重定根、遍历）
- 距离计算（支系距离、同表型距离、Robinson-Foulds距离）
- ASCII 可视化
- Newick 格式输入输出

**常用模式：**
```python
from skbio import TreeNode
from skbio.tree import nj

# 从文件读取树
tree = TreeNode.read('tree.nwk')

# 基于距离矩阵建树
tree = nj(distance_matrix)

# 树操作
subtree = tree.shear(['taxon1', 'taxon2', 'taxon3'])
tips = [node for node in tree.tips()]
lca = tree.lowest_common_ancestor(['taxon1', 'taxon2'])

# 计算距离
patristic_dist = tree.find('taxon1').distance(tree.find('taxon2'))
cophenetic_matrix = tree.cophenetic_matrix()

# 比较树结构
rf_distance = tree.robinson_foulds(other_tree)
```

**重要提示：**
- 使用 `nj()` 执行邻接法（经典系统发育方法）
- 使用 `upgma()` 执行 UPGMA（假设分子钟）
- GME 和 BME 适用于大型树的高效构建
- 树可设为有根或无根；部分指标需特定根节点

### 4. 多样性分析

计算微生物生态学与群落分析的α和β多样性指标。

**核心能力：**
- α多样性：丰富度、香农熵、辛普森指数、Faith系统发育多样性、Pielou均匀度
- β多样性：Bray-Curtis、Jaccard、加权/非加权UniFrac、欧氏距离
- 系统发育多样性指标（需输入树）
- 稀疏抽样与子采样
- 与排序分析和统计检验集成

**常用模式：**
```python
from skbio.diversity import alpha_diversity, beta_diversity
import skbio

# α多样性
alpha = alpha_diversity('shannon', counts_matrix, ids=sample_ids)
faith_pd = alpha_diversity('faith_pd', counts_matrix, ids=sample_ids,
                          tree=tree, otu_ids=feature_ids)

# β多样性
bc_dm = beta_diversity('braycurtis', counts_matrix, ids=sample_ids)
unifrac_dm = beta_diversity('unweighted_unifrac', counts_matrix,
                           ids=sample_ids, tree=tree, otu_ids=feature_ids)

# 获取可用指标
from skbio.diversity import get_alpha_diversity_metrics
print(get_alpha_diversity_metrics())
```

**重要提示：**
- 计数必须为表示丰度的整数，而非相对频率
- 系统发育指标（Faith's PD、UniFrac）需树结构和OTU ID映射
- 使用 `partial_beta_diversity()` 计算特定样本对
- α多样性返回Series，β多样性返回DistanceMatrix

### 5. 排序分析

将高维生物数据降维至可可视化空间。

**核心能力：**
- 基于距离矩阵的主坐标分析（PCoA）
- 列联表的对应分析（CA）
- 带环境约束的典范对应分析（CCA）
- 线性关系的冗余分析（RDA）
- 特征解释的双标图投影

**常用模式：**
```python
from skbio.stats.ordination import pcoa, cca

# 基于距离矩阵的PCoA
pcoa_results = pcoa(distance_matrix)
pc1 = pcoa_results.samples['PC1']
pc2 = pcoa_results.samples['PC2']

# 带环境变量的CCA
cca_results = cca(species_matrix, environmental_matrix)

# 保存/加载排序结果
pcoa_results.write('ordination.txt')
results = skbio.OrdinationResults.read('ordination.txt')
```

**重要提示：**
- PCoA 适用于任意距离/相异矩阵
- CCA 揭示群落组成的环境驱动因素
- 排序结果包含特征值、解释比例及样本/特征坐标
- 结果可集成绘图库（matplotlib、seaborn、plotly）

### 6. 统计检验

执行生态与生物数据专用假设检验。

**核心能力：**
- PERMANOVA：基于距离矩阵检验组间差异
- ANOSIM：组间差异替代检验法
- PERMDISP：检验组离散同质性
- Mantel检验：距离矩阵间相关性
- Bioenv：寻找与距离相关的环境变量

**常用模式：**
```python
from skbio.stats.distance import permanova, anosim, mantel

# 检验组间差异显著性
permanova_results = permanova(distance_matrix, grouping, permutations=999)
print(f"p值: {permanova_results['p-value']}")

# ANOSIM检验
anosim_results = anosim(distance_matrix, grouping, permutations=999)

# 两个距离矩阵的Mantel检验
mantel_results = mantel(dm1, dm2, method='pearson', permutations=999)
print(f"相关性: {mantel_results[0]}, p值: {mantel_results[1]}")
```

**重要提示：**
- 置换检验提供非参数显著性检验
- 使用999+次置换确保p值稳健性
- PERMANOVA对离散差异敏感，需配合PERMDISP使用
- Mantel检验评估矩阵相关性（如地理距离与遗传距离）

### 7. 文件输入输出与格式转换

支持19+种生物文件格式的自动检测读写。

**支持格式：**
- 序列：FASTA、FASTQ、GenBank、EMBL、QSeq
- 比对：Clustal、PHYLIP、Stockholm
- 树结构：Newick
- 表格：BIOM（HDF5 和 JSON）
- 距离：带分隔符的方阵
- 分析：BLAST+6/7、GFF3、排序结果
- 元数据：带验证的TSV/CSV

**常用模式：**
```python
import skbio

# 自动检测格式读取
seq = skbio.DNA.read('file.fasta', format='fasta')
tree = skbio.TreeNode.read('tree.nwk')

# 写入文件
seq.write('output.fasta', format='fasta')

# 大文件生成器（内存高效）
for seq in skbio.io.read('large.fasta', format='fasta', constructor=skbio.DNA):
    process(seq)

# 格式转换
seqs = list(skbio.io.read('input.fastq', format='fastq', constructor=skbio.DNA))
skbio.io.write(seqs, format='fasta', into='output.fasta')
```

**重要提示：**
- 大文件使用生成器避免内存问题
- 指定 `into` 参数时可自动检测格式
- 部分对象支持多格式输出
- 通过 `verify=False` 支持 stdin/stdout 管道

### 8. 距离矩阵

创建并操作距离/相异矩阵的统计方法。

**核心能力：**
- 存储对称（DistanceMatrix）或非对称（DissimilarityMatrix）数据
- 基于ID的索引与切片
- 与多样性、排序和统计检验集成
- 读写带分隔符的文本格式

**常用模式：**
```python
from skbio import DistanceMatrix
import numpy as np

# 从数组创建
data = np.array([[0, 1, 2], [1, 0, 3], [2, 3, 0]])
dm = DistanceMatrix(data, ids=['A', 'B', 'C'])

# 访问距离
dist_ab = dm['A', 'B']
row_a = dm['A']

# 从文件读取
dm = DistanceMatrix.read('distances.txt')

# 用于下游分析
pcoa_results = pcoa(dm)
permanova_results = permanova(dm, grouping)
```

**重要提示：**
- DistanceMatrix 强制对称性与零对角线
- DissimilarityMatrix 允许非对称值
- ID机制支持元数据与生物知识集成
- 兼容 pandas、numpy 和 scikit-learn

### 9. 生物表格

处理微生物组研究中常见的特征表（OTU/ASV表）。

**核心能力：**
- BIOM 格式输入输出（HDF5 和 JSON）
- 与 pandas、polars、AnnData、numpy 集成
- 数据增强技术（phylomix、mixup、组合方法）
- 样本/特征过滤与标准化
- 元数据集成

**常用模式：**
```python
from skbio import Table

# 读取BIOM表格
table = Table.read('table.biom')

# 访问数据
sample_ids = table.ids(axis='sample')
feature_ids = table.ids(axis='observation')
counts = table.matrix_data

# 过滤
filtered = table.filter(sample_ids_to_keep, axis='sample')

# 与pandas互转
df = table.to_dataframe()
table = Table.from_dataframe(df)
```

**重要提示：**
- BIOM 表格是 QIIME 2 的标准格式
- 行通常代表样本，列代表特征（OTU/ASV）
- 支持稀疏与稠密表示
- 输出格式可配置（pandas/polars/numpy）

### 10. 蛋白质嵌入

处理蛋白质语言模型嵌入以进行下游分析。

**核心能力：**
- 存储蛋白质语言模型嵌入（ESM、ProtTrans等）
- 将嵌入向量转为距离矩阵
- 生成用于可视化的排序对象
- 导出至 numpy/pandas 供机器学习流程使用

**常用模式：**
```python
from skbio.embedding import ProteinEmbedding, ProteinVector

# 从数组创建嵌入
embedding = ProteinEmbedding(embedding_array, sequence_ids)

# 转为距离矩阵分析
dm = embedding.to_distances(metric='euclidean')

# 嵌入空间的PCoA可视化
pcoa_results = embedding.to_ordination(metric='euclidean', method='pcoa')

# 导出供机器学习使用
array = embedding.to_array()
df = embedding.to_dataframe()
```

**重要提示：**
- 嵌入向量连接蛋白质语言模型与传统生物信息学
- 兼容 scikit-bio 的距离/排序/统计生态系统
- SequenceEmbedding 和 ProteinEmbedding 提供专用功能
- 适用于序列聚类、分类与可视化

## 最佳实践

### 安装
```bash
uv pip install scikit-bio
```

### 性能考量
- 大序列文件使用生成器降低内存占用
- 超大树结构优先选用 GME 或 BME 而非邻接法
- β多样性计算可通过 `partial_beta_diversity()` 并行化
- BIOM格式（HDF5）比JSON更高效处理大表格

### 生态集成
- 序列通过标准格式与 Biopython 互操作
- 表格与 pandas、polars 和 AnnData 集成
- 距离矩阵兼容 scikit-learn
- 排序结果可通过 matplotlib/seaborn/plotly 可视化
- 无缝对接 QIIME 2 组件（BIOM、树结构、距离矩阵）

### 典型工作流
1. **微生物组多样性分析**：读取BIOM表 → 计算α/β多样性 → 排序分析（PCoA） → 统计检验（PERMANOVA）
2. **系统发育分析**：读取序列 → 比对 → 构建距离矩阵 → 建树 → 计算系统发育距离
3. **序列处理**：读取FASTQ → 质量过滤 → 修剪/清洗 → 查找模体 → 翻译 → 输出FASTA
4. **比较基因组学**：读取序列 → 成对比对 → 计算距离 → 建树 → 分析进化枝

## 参考文档

详细API信息、参数说明及高级用例请参阅 `references/api_reference.md`，包含
