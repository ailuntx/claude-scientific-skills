# 使用 Bio.Align 和 Bio.AlignIO 进行序列比对

## 概述

Bio.Align 提供使用多种算法进行双序列比对的工具，而 Bio.AlignIO 则处理多种格式的多序列比对文件的读写。

## 使用 Bio.Align 进行双序列比对

### PairwiseAligner 类

`PairwiseAligner` 类使用 Needleman-Wunsch（全局）、Smith-Waterman（局部）、Gotoh（三状态）和 Waterman-Smith-Beyer 算法执行双序列比对。系统会根据空位得分参数自动选择合适的算法。

### 创建比对器

```python
from Bio import Align

# 使用默认参数创建比对器
aligner = Align.PairwiseAligner()

# 默认分数（自 Biopython 1.85+ 起）：
# - 匹配得分：+1.0
# - 错配得分：0.0
# - 所有空位得分：-1.0
```

### 自定义比对参数

```python
# 设置评分参数
aligner.match_score = 2.0
aligner.mismatch_score = -1.0
aligner.gap_score = -0.5

# 或分别设置空位开放/延伸罚分
aligner.open_gap_score = -2.0
aligner.extend_gap_score = -0.5

# 单独设置内部空位分数
aligner.internal_open_gap_score = -2.0
aligner.internal_extend_gap_score = -0.5

# 设置末端空位分数（用于半全局比对）
aligner.left_open_gap_score = 0.0
aligner.left_extend_gap_score = 0.0
aligner.right_open_gap_score = 0.0
aligner.right_extend_gap_score = 0.0
```

### 比对模式

```python
# 全局比对（默认）
aligner.mode = 'global'

# 局部比对
aligner.mode = 'local'
```

### 执行比对

```python
from Bio.Seq import Seq

seq1 = Seq("ACCGGT")
seq2 = Seq("ACGGT")

# 获取所有最优比对
alignments = aligner.align(seq1, seq2)

# 遍历比对结果
for alignment in alignments:
    print(alignment)
    print(f"得分: {alignment.score}")

# 仅获取得分
score = aligner.score(seq1, seq2)
```

### 使用替换矩阵

```python
from Bio.Align import substitution_matrices

# 加载替换矩阵
matrix = substitution_matrices.load("BLOSUM62")
aligner.substitution_matrix = matrix

# 比对蛋白质序列
protein1 = Seq("KEVLA")
protein2 = Seq("KSVLA")
alignments = aligner.align(protein1, protein2)
```

### 可用替换矩阵

常用矩阵包括：
- **BLOSUM** 系列 (BLOSUM45, BLOSUM50, BLOSUM62, BLOSUM80, BLOSUM90)
- **PAM** 系列 (PAM30, PAM70, PAM250)
- **MATCH** - 简单匹配/错配矩阵

```python
# 列出可用矩阵
available = substitution_matrices.load()
print(available)
```

## 使用 Bio.AlignIO 处理多序列比对

### 读取比对文件

Bio.AlignIO 提供类似 Bio.SeqIO 的 API，但用于比对文件：

```python
from Bio import AlignIO

# 读取单个比对文件
alignment = AlignIO.read("alignment.aln", "clustal")

# 从文件解析多个比对
for alignment in AlignIO.parse("alignments.aln", "clustal"):
    print(f"包含 {len(alignment)} 条序列的比对")
    print(f"比对长度: {alignment.get_alignment_length()}")
```

### 支持的比对格式

常用格式包括：
- **clustal** - Clustal 格式
- **phylip** - PHYLIP 格式
- **phylip-relaxed** - 宽松 PHYLIP（支持长名称）
- **stockholm** - Stockholm 格式
- **fasta** - FASTA 格式（已比对）
- **nexus** - NEXUS 格式
- **emboss** - EMBOSS 比对格式
- **msf** - MSF 格式
- **maf** - 多序列比对格式

### 写入比对文件

```python
# 将比对写入文件
AlignIO.write(alignment, "output.aln", "clustal")

# 格式转换
count = AlignIO.convert("input.aln", "clustal", "output.phy", "phylip")
```

### 操作比对对象

```python
from Bio import AlignIO

alignment = AlignIO.read("alignment.aln", "clustal")

# 获取比对属性
print(f"序列数量: {len(alignment)}")
print(f"比对长度: {alignment.get_alignment_length()}")

# 访问单条序列
for record in alignment:
    print(f"{record.id}: {record.seq}")

# 获取比对列
column = alignment[:, 0]  # 第一列

# 获取比对切片
sub_alignment = alignment[:, 10:20]  # 第10-20位

# 获取特定序列
seq_record = alignment[0]  # 第一条序列
```

### 比对分析

```python
# 计算比对统计信息
from Bio.Align import AlignInfo

summary = AlignInfo.SummaryInfo(alignment)

# 获取共有序列
consensus = summary.gap_consensus(threshold=0.7)

# 位置特异性评分矩阵 (PSSM)
pssm = summary.pos_specific_score_matrix(consensus)

# 计算信息量
from Bio import motifs
motif = motifs.create([record.seq for record in alignment])
information = motif.counts.information_content()
```

## 编程创建比对

### 从 SeqRecord 对象创建

```python
from Bio.Align import MultipleSeqAlignment
from Bio.SeqRecord import SeqRecord
from Bio.Seq import Seq

# 创建记录
records = [
    SeqRecord(Seq("ACTGCTAGCTAG"), id="seq1"),
    SeqRecord(Seq("ACT-CTAGCTAG"), id="seq2"),
    SeqRecord(Seq("ACTGCTA-CTAG"), id="seq3"),
]

# 创建比对
alignment = MultipleSeqAlignment(records)
```

### 向比对添加序列

```python
# 从空比对开始
alignment = MultipleSeqAlignment([])

# 添加序列（必须等长）
alignment.append(SeqRecord(Seq("ACTG"), id="seq1"))
alignment.append(SeqRecord(Seq("ACTG"), id="seq2"))

# 扩展其他比对
alignment.extend(other_alignment)
```

## 高级比对操作

### 去除空位

```python
# 移除全空位列
from Bio.Align import AlignInfo

no_gaps = []
for i in range(alignment.get_alignment_length()):
    column = alignment[:, i]
    if set(column) != {'-'}:  # 非全空位
        no_gaps.append(column)
```

### 比对排序

```python
# 按序列ID排序
sorted_alignment = sorted(alignment, key=lambda x: x.id)
alignment = MultipleSeqAlignment(sorted_alignment)
```

### 计算成对一致性

```python
def pairwise_identity(seq1, seq2):
    """计算两条序列之间的百分比一致性"""
    matches = sum(a == b for a, b in zip(seq1, seq2) if a != '-' and b != '-')
    length = sum(1 for a, b in zip(seq1, seq2) if a != '-' and b != '-')
    return matches / length if length > 0 else 0

# 计算所有成对一致性
for i, record1 in enumerate(alignment):
    for record2 in alignment[i+1:]:
        identity = pairwise_identity(record1.seq, record2.seq)
        print(f"{record1.id} 对比 {record2.id}: {identity:.2%}")
```

## 运行外部比对工具

### Clustal Omega（命令行方式）

```python
from Bio.Align.Applications import ClustalOmegaCommandline

# 设置命令
clustal_cmd = ClustalOmegaCommandline(
    infile="sequences.fasta",
    outfile="alignment.aln",
    verbose=True,
    auto=True
)

# 运行比对
stdout, stderr = clustal_cmd()

# 读取结果
alignment = AlignIO.read("alignment.aln", "clustal")
```

### MUSCLE（命令行方式）

```python
from Bio.Align.Applications import MuscleCommandline

muscle_cmd = MuscleCommandline(
    input="sequences.fasta",
    out="alignment.aln"
)
stdout, stderr = muscle_cmd()
```

## 最佳实践

1. **选择合适的评分方案** - 蛋白质使用 BLOSUM62，DNA 使用自定义分数
2. **考虑比对模式** - 相似长度序列用全局比对，寻找保守区域用局部比对
3. **谨慎设置空位罚分** - 高罚分产生更少、更长的空位
4. **使用合适格式** - 简单比对用 FASTA，丰富注释用 Stockholm
5. **验证比对质量** - 检查保守区域和百分比一致性
6. **谨慎处理大型比对** - 使用切片和迭代提高内存效率
7. **保留元数据** - 在比对操作中保持 SeqRecord ID 和注释

## 常见用例

### 寻找最佳局部比对

```python
from Bio.Align import PairwiseAligner
from Bio.Seq import Seq

aligner = PairwiseAligner()
aligner.mode = 'local'
aligner.match_score = 2
aligner.mismatch_score = -1

seq1 = Seq("AGCTTAGCTAGCTAGC")
seq2 = Seq("CTAGCTAGC")

alignments = aligner.align(seq1, seq2)
print(alignments[0])
```

### 蛋白质序列比对

```python
from Bio.Align import PairwiseAligner, substitution_matrices

aligner = PairwiseAligner()
aligner.substitution_matrix = substitution_matrices.load("BLOSUM62")
aligner.open_gap_score = -10
aligner.extend_gap_score = -0.5

protein1 = Seq("KEVLA")
protein2 = Seq("KEVLAEQP")
alignments = aligner.align(protein1, protein2)
```

### 提取保守区域

```python
from Bio import AlignIO

alignment = AlignIO.read("alignment.aln", "clustal")

# 寻找一致性>80%的列
conserved_positions = []
for i in range(alignment.get_alignment_length()):
    column = alignment[:, i]
    most_common = max(set(column), key=column.count)
    if column.count(most_common) / len(column) > 0.8:
        conserved_positions.append(i)

print(f"保守位置: {conserved_positions}")
```
