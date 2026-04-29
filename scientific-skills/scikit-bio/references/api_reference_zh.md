# scikit-bio API 参考文档

本文档提供详细的 scikit-bio API 信息、高级示例和故障排除指南。

## 目录
1. [序列类](#sequence-classes)
2. [比对方法](#alignment-methods)
3. [系统发育树](#phylogenetic-trees)
4. [多样性指标](#diversity-metrics)
5. [排序分析](#ordination)
6. [统计检验](#statistical-tests)
7. [距离矩阵](#distance-matrices)
8. [文件输入输出](#file-io)
9. [故障排除](#troubleshooting)

## 序列类

### DNA、RNA 和蛋白质类

```python
from skbio import DNA, RNA, Protein, Sequence

# 创建序列
dna = DNA('ATCGATCG', metadata={'id': 'seq1', 'description': '示例'})
rna = RNA('AUCGAUCG')
protein = Protein('ACDEFGHIKLMNPQRSTVWY')

# 序列操作
dna_rc = dna.reverse_complement()  # 反向互补
rna = dna.transcribe()  # DNA -> RNA
protein = rna.translate()  # RNA -> 蛋白质

# 使用遗传密码表
protein = rna.translate(genetic_code=11)  # 细菌密码表
```

### 序列搜索与模式匹配

```python
# 使用正则表达式查找基序
dna = DNA('ATGCGATCGATGCATCG')
motif_locs = dna.find_with_regex('ATG.{3}')  # 起始密码子

# 查找所有位置
import re
for match in re.finditer('ATG', str(dna)):
    print(f"在位置 {match.start()} 发现 ATG")

# k-mer 计数
from skbio.sequence import _motifs
kmers = dna.kmer_frequencies(k=3)
```

### 处理序列元数据

```python
# 序列级元数据
dna = DNA('ATCG', metadata={'id': 'seq1', 'source': '大肠杆菌'})
print(dna.metadata['id'])

# 位置元数据（FASTQ 中的每碱基质量分数）
from skbio import DNA
seqs = DNA.read('reads.fastq', format='fastq', phred_offset=33)
quality_scores = seqs.positional_metadata['quality']

# 区间元数据（特征/注释）
dna.interval_metadata.add([(5, 15)], metadata={'type': '基因', 'name': '基因A'})
```

### 距离计算

```python
from skbio import DNA

seq1 = DNA('ATCGATCG')
seq2 = DNA('ATCG--CG')

# 汉明距离（默认）
dist = seq1.distance(seq2)

# 自定义距离函数
from skbio.sequence.distance import kmer_distance
dist = seq1.distance(seq2, metric=kmer_distance)
```

## 比对方法

### 双序列比对

```python
from skbio.alignment import local_pairwise_align_ssw, global_pairwise_align
from skbio import DNA, Protein

# 局部比对（基于 SSW 的 Smith-Waterman）
seq1 = DNA('ATCGATCGATCG')
seq2 = DNA('ATCGGGGATCG')
alignment = local_pairwise_align_ssw(seq1, seq2)

# 访问比对详情
print(f"得分: {alignment.score}")
print(f"起始位置: {alignment.target_begin}")
aligned_seqs = alignment.aligned_sequences

# 自定义评分的全局比对
from skbio.alignment import AlignScorer

scorer = AlignScorer(
    match_score=2,
    mismatch_score=-3,
    gap_open_penalty=5,
    gap_extend_penalty=2
)

alignment = global_pairwise_align(seq1, seq2, scorer=scorer)

# 使用替换矩阵的蛋白质比对
from skbio.alignment import StripedSmithWaterman

protein_query = Protein('ACDEFGHIKLMNPQRSTVWY')
protein_target = Protein('ACDEFMNPQRSTVWY')

aligner = StripedSmithWaterman(
    str(protein_query),
    gap_open_penalty=11,
    gap_extend_penalty=1,
    substitution_matrix='blosum62'
)
alignment = aligner(str(protein_target))
```

### 多序列比对

```python
from skbio.alignment import TabularMSA
from skbio import DNA

# 从文件读取 MSA
msa = TabularMSA.read('alignment.fasta', constructor=DNA)

# 手动创建 MSA
seqs = [
    DNA('ATCG--'),
    DNA('ATGG--'),
    DNA('ATCGAT')
]
msa = TabularMSA(seqs)

# MSA 操作
consensus = msa.consensus()
majority_consensus = msa.majority_consensus()

# 计算保守性
conservation = msa.conservation()

# 访问序列
first_seq = msa[0]
column = msa[:, 2]  # 第三列

# 过滤空位
degapped_msa = msa.omit_gap_positions(maximum_gap_frequency=0.5)

# 计算位置特异性分数
position_entropies = msa.position_entropies()
```

### CIGAR 字符串处理

```python
from skbio.alignment import AlignPath

# 解析 CIGAR 字符串
cigar = "10M2I5M3D10M"
align_path = AlignPath.from_cigar(cigar, target_length=100, query_length=50)

# 将比对转换为 CIGAR
alignment = local_pairwise_align_ssw(seq1, seq2)
cigar_string = alignment.to_cigar()
```

## 系统发育树

### 树构建

```python
from skbio import TreeNode, DistanceMatrix
from skbio.tree import nj, upgma

# 距离矩阵
dm = DistanceMatrix([[0, 5, 9, 9],
                     [5, 0, 10, 10],
                     [9, 10, 0, 8],
                     [9, 10, 8, 0]],
                    ids=['A', 'B', 'C', 'D'])

# 邻接法
nj_tree = nj(dm)

# UPGMA（假设分子钟）
upgma_tree = upgma(dm)

# 平衡最小进化（适用于大型树）
from skbio.tree import bme
bme_tree = bme(dm)
```

### 树操作

```python
from skbio import TreeNode

# 读取树文件
tree = TreeNode.read('tree.nwk', format='newick')

# 遍历
for node in tree.traverse():
    print(node.name)

# 前序、后序、层序遍历
for node in tree.preorder():
    print(node.name)

# 仅获取末端节点
tips = list(tree.tips())

# 查找特定节点
node = tree.find('分类单元名称')

# 在中心点定根
rooted_tree = tree.root_at_midpoint()

# 修剪树至特定分类单元
pruned = tree.shear(['分类单元1', '分类单元2', '分类单元3'])

# 获取子树
lca = tree.lowest_common_ancestor(['分类单元1', '分类单元2'])
subtree = lca.copy()

# 添加/删除节点
parent = tree.find('父节点名称')
child = TreeNode(name='新子节点', length=0.5)
parent.append(child)

# 删除节点
node_to_remove = tree.find('待删除节点')
node_to_remove.parent.remove(node_to_remove)
```

### 树距离与比较

```python
# 谱系距离（分支长度距离）
node1 = tree.find('分类单元1')
node2 = tree.find('分类单元2')
patristic = node1.distance(node2)

# 同表型矩阵（所有成对距离）
cophenetic_dm = tree.cophenetic_matrix()

# Robinson-Foulds 距离（拓扑比较）
rf_dist = tree.robinson_foulds(other_tree)

# 非加权 RF 比较
rf_dist, max_rf = tree.robinson_foulds(other_tree, proportion=False)

# 末端节点间距离
tip_distances = tree.tip_tip_distances()
```

### 树可视化

```python
# ASCII 艺术可视化
print(tree.ascii_art())

# 高级可视化需导出到外部工具
tree.write('tree.nwk', format='newick')

# 使用 ete3/toytree/ggtree 生成出版级图表
```

## 多样性指标

### Alpha 多样性

```python
from skbio.diversity import alpha_diversity, get_alpha_diversity_metrics
import numpy as np

# 样本计数数据（样本 x 特征）
counts = np.array([
    [10, 5, 0, 3],
    [2, 0, 8, 4],
    [5, 5, 5, 5]
])
sample_ids = ['样本1', '样本2', '样本3']

# 列出可用指标
print(get_alpha_diversity_metrics())

# 计算多种 alpha 多样性指标
shannon = alpha_diversity('shannon', counts, ids=sample_ids)
simpson = alpha_diversity('simpson', counts, ids=sample_ids)
observed_otus = alpha_diversity('observed_otus', counts, ids=sample_ids)
chao1 = alpha_diversity('chao1', counts, ids=sample_ids)

# 系统发育 alpha 多样性（需树结构）
from skbio import TreeNode

tree = TreeNode.read('tree.nwk')
feature_ids = ['OTU1', 'OTU2', 'OTU3', 'OTU4']

faith_pd = alpha_diversity('faith_pd', counts, ids=sample_ids,
                          tree=tree, otu_ids=feature_ids)
```

### Beta 多样性

```python
from skbio.diversity import beta_diversity, partial_beta_diversity

# Beta 多样性（所有成对比较）
bc_dm = beta_diversity('braycurtis', counts, ids=sample_ids)

# Jaccard（存在/缺失）
jaccard_dm = beta_diversity('jaccard', counts, ids=sample_ids)

# 系统发育 beta 多样性
unifrac_dm = beta_diversity('unweighted_unifrac', counts,
                           ids=sample_ids,
                           tree=tree,
                           otu_ids=feature_ids)

weighted_unifrac_dm = beta_diversity('weighted_unifrac', counts,
                                    ids=sample_ids,
                                    tree=tree,
                                    otu_ids=feature_ids)

# 仅计算特定对（更高效）
pairs = [('样本1', '样本2'), ('样本1', '样本3')]
partial_dm = partial_beta_diversity('braycurtis', counts,
                                   ids=sample_ids,
                                   id_pairs=pairs)
```

### 稀疏化与子采样

```python
from skbio.diversity import subsample_counts

# 稀疏至最小深度
min_depth = counts.min(axis=1).max()
rarefied = [subsample_counts(row, n=min_depth) for row in counts]

# 多次稀疏化计算置信区间
import numpy as np
rarefactions = []
for i in range(100):
    rarefied_counts = np.array([subsample_counts(row, n=1000) for row in counts])
    shannon_rare = alpha_diversity('shannon', rarefied_counts)
    rarefactions.append(shannon_rare)

# 计算均值与标准差
mean_shannon = np.mean(rarefactions, axis=0)
std_shannon = np.std(rarefactions, axis=0)
```

## 排序分析

### 主坐标分析 (PCoA)

```python
from skbio.stats.ordination import pcoa
from skbio import DistanceMatrix
import numpy as np

# 基于距离矩阵的 PCoA
dm = DistanceMatrix(...)
pcoa_results = pcoa(dm)

# 访问坐标
pc1 = pcoa_results.samples['PC1']
pc2 = pcoa_results.samples['PC2']

# 解释比例
prop_explained = pcoa_results.proportion_explained

# 特征值
eigenvalues = pcoa_results.eigvals

# 保存结果
pcoa_results.write('pcoa_results.txt')

# 使用 matplotlib 绘图
import matplotlib.pyplot as plt
plt.scatter(pc1, pc2)
plt.xlabel(f'PC1 ({prop_explained[0]*100:.1f}%)')
plt.ylabel(f'PC2 ({prop_explained[1]*100:.1f}%)')
```

### 典范对应分析 (CCA)

```python
from skbio.stats.ordination import cca
import pandas as pd
import numpy as np

# 物种丰度矩阵（样本 x 物种）
species = np.array([
    [10, 5, 3],
    [2, 8, 4],
    [5, 5, 5]
])

# 环境变量（样本 x 变量）
env = pd.DataFrame({
    'pH': [6.5, 7.0, 6.8],
    '温度': [20, 25, 22],
    '深度': [10, 15, 12]
})

# CCA
cca_results = cca(species, env,
                 sample_ids=['站点1', '站点2', '站点3'],
                 species_ids=['物种A', '物种B', '物种C'])

# 访问约束轴
cca1 = cca_results.samples['CCA1']
cca2 = cca_results.samples['CCA2']

# 环境变量的双标图得分
env_scores = cca_results.biplot_scores
```

### 冗余分析 (RDA)

```python
from skbio.stats.ordination import rda

# 类似 CCA，但用于线性关系
rda_results = rda(species, env,
                 sample_ids=['站点1', '站点2', '站点3'],
                 species_ids=['物种A', '物种B', '物种C'])
```

## 统计检验

### PERMANOVA

```python
from skbio.stats.distance import permanova
from skbio import DistanceMatrix
import numpy as np

# 距离矩阵
dm = DistanceMatrix(...)

# 分组变量
grouping = ['组1', '组1', '组2', '组2', '组3', '组3']

# 执行 PERMANOVA
results = permanova(dm, grouping, permutations=999)

print(f"检验统计量: {results['test statistic']}")
print(f"p值: {results['p-value']}")
print(f"样本量: {results['sample size']}")
print(f"组数: {results['number of groups']}")
```

### ANOSIM

```python
from skbio.stats.distance import anosim

# ANOSIM 检验
results = anosim(dm, grouping, permutations=999)

print(f"R 统计量: {results['test statistic']}")
print(f"p值: {results['p-value']}")
```

### PERMDISP

```python
from skbio.stats.distance import permdisp

# 检验离散度同质性
results = permdisp(dm, grouping, permutations=999)

print(f"F 统计量: {results['test statistic']}")
print(f"p值: {results['p-value']}")
```

### Mantel 检验

```python
from skbio.stats.distance import mantel
from skbio import DistanceMatrix

# 待比较的两个距离矩阵
dm1 = DistanceMatrix(...)  # 例如遗传距离
dm2 = DistanceMatrix(...)  # 例如地理距离

# Mantel 检验
r, p_value, n = mantel(dm1, dm2, method='pearson', permutations=999)

print(f"相关性: {r}")
print(f"p值: {p_value}")
print(f"样本量: {n}")

# Spearman 相关性
r_spearman, p, n = mantel(dm1, dm2, method='spearman', permutations=999)
```

### 偏 Mantel 检验

```python
from skbio.stats.distance import mantel

# 控制第三个矩阵
dm3 = DistanceMatrix(...)  # 控制变量

r_partial, p_value, n = mantel(dm1, dm2, method='pearson',
                               permutations=999, alternative='two-sided')
```

## 距离矩阵

### 创建与操作距离矩阵

```python
from skbio import DistanceMatrix, DissimilarityMatrix
import numpy as np

# 从数组创建
data = np.array([[0, 1, 2],
                 [1, 0, 3],
                 [2, 3, 0]])
dm = DistanceMatrix(data, ids=['A', 'B', 'C'])

# 访问元素
dist_ab = dm['A', 'B']
row

```markdown
dm.write('distances.txt')
dm2 = DistanceMatrix.read('distances.txt')

# 转换为压缩格式（供scipy使用）
condensed = dm.condensed_form()

# 转换为数据框
df = dm.to_data_frame()
```

## 文件输入/输出

### 读取序列

```python
import skbio

# 读取单条序列
dna = skbio.DNA.read('sequence.fasta', format='fasta')

# 读取多条序列（生成器）
for seq in skbio.io.read('sequences.fasta', format='fasta', constructor=skbio.DNA):
    print(seq.metadata['id'], len(seq))

# 读取到列表
sequences = list(skbio.io.read('sequences.fasta', format='fasta',
                               constructor=skbio.DNA))

# 读取带质量分数的FASTQ
for seq in skbio.io.read('reads.fastq', format='fastq', constructor=skbio.DNA):
    quality = seq.positional_metadata['quality']
    print(f"平均质量: {quality.mean()}")
```

### 写入序列

```python
# 写入单条序列
dna.write('output.fasta', format='fasta')

# 写入多条序列
sequences = [dna1, dna2, dna3]
skbio.io.write(sequences, format='fasta', into='output.fasta')

# 自定义换行写入
dna.write('output.fasta', format='fasta', max_width=60)
```

### BIOM表格

```python
from skbio import Table

# 读取BIOM表格
table = Table.read('table.biom', format='hdf5')

# 访问数据
sample_ids = table.ids(axis='sample')
feature_ids = table.ids(axis='observation')
matrix = table.matrix_data.toarray()  # 稀疏矩阵时使用

# 过滤样本
abundant_samples = table.filter(lambda row, id_, md: row.sum() > 1000, axis='sample')

# 过滤特征（OTUs/ASVs）
prevalent_features = table.filter(lambda col, id_, md: (col > 0).sum() >= 3,
                                 axis='observation')

# 标准化
relative_abundance = table.norm(axis='sample', inplace=False)

# 写入
table.write('filtered_table.biom', format='hdf5')
```

### 格式转换

```python
# FASTQ转FASTA
seqs = skbio.io.read('input.fastq', format='fastq', constructor=skbio.DNA)
skbio.io.write(seqs, format='fasta', into='output.fasta')

# GenBank转FASTA
seqs = skbio.io.read('genes.gb', format='genbank', constructor=skbio.DNA)
skbio.io.write(seqs, format='fasta', into='genes.fasta')
```

## 故障排除

### 常见问题与解决方案

#### 问题："ValueError: Ids must be unique"
```python
# 问题：序列ID重复
# 解决方案：确保ID唯一或过滤重复项
seen = set()
unique_seqs = []
for seq in sequences:
    if seq.metadata['id'] not in seen:
        unique_seqs.append(seq)
        seen.add(seq.metadata['id'])
```

#### 问题："ValueError: Counts must be integers"
```python
# 问题：使用了相对丰度而非整数计数
# 解决方案：转换为整数计数或使用适当指标
counts_int = (abundance_table * 1000).astype(int)
```

#### 问题：大文件导致内存错误
```python
# 问题：尝试将整个文件加载到内存
# 解决方案：使用生成器
for seq in skbio.io.read('huge.fasta', format='fasta', constructor=skbio.DNA):
    # 逐条处理
    process(seq)
```

#### 问题：树末端节点与OTU ID不匹配
```python
# 问题：树末端名称与特征ID不一致
# 解决方案：验证并对齐ID
tree_tips = {tip.name for tip in tree.tips()}
feature_ids = set(feature_ids)
missing_in_tree = feature_ids - tree_tips
missing_in_table = tree_tips - feature_ids

# 修剪树以匹配表格
tree_pruned = tree.shear(feature_ids)
```

#### 问题：不同长度序列导致比对失败
```python
# 问题：尝试比对已预对齐的序列
# 解决方案：先去除间隔或确保序列未对齐
seq1_degapped = seq1.degap()
seq2_degapped = seq2.degap()
alignment = local_pairwise_align_ssw(seq1_degapped, seq2_degapped)
```

### 性能优化建议

1. **使用合适的数据结构**：大型表格用BIOM HDF5，大序列文件用生成器
2. **并行处理**：对可并行化的子集计算使用`partial_beta_diversity()`
3. **大型数据集抽样**：探索性分析时先处理抽样数据
4. **缓存结果**：保存距离矩阵和排序结果避免重复计算

### 集成示例

#### 与pandas集成
```python
import pandas as pd
from skbio import DistanceMatrix

# 距离矩阵转DataFrame
dm = DistanceMatrix(...)
df = dm.to_data_frame()

# Alpha多样性转DataFrame
alpha = alpha_diversity('shannon', counts, ids=sample_ids)
alpha_df = pd.DataFrame({'shannon': alpha})
```

#### 与matplotlib/seaborn集成
```python
import matplotlib.pyplot as plt
import seaborn as sns

# PCoA图
fig, ax = plt.subplots()
scatter = ax.scatter(pc1, pc2, c=grouping, cmap='viridis')
ax.set_xlabel(f'PC1 ({prop_explained[0]*100:.1f}%)')
ax.set_ylabel(f'PC2 ({prop_explained[1]*100:.1f}%)')
plt.colorbar(scatter)

# 距离矩阵热力图
sns.heatmap(dm.to_data_frame(), cmap='viridis')
```

#### 与QIIME 2集成
```python
# scikit-bio对象兼容QIIME 2
# 从QIIME 2导出
# qiime tools export --input-path table.qza --output-path exported/

# 在scikit-bio中读取
table = Table.read('exported/feature-table.biom')

# 使用scikit-bio处理
# ...

# 如需导回QIIME 2
table.write('processed-table.biom')
# qiime tools import --input-path processed-table.biom --output-path processed.qza
```
