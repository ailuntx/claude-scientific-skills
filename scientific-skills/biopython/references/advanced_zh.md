# 高级 Biopython 功能

## 使用 Bio.motifs 处理序列模体

### 创建模体

```python
from Bio import motifs
from Bio.Seq import Seq

# 从实例创建模体
instances = [
    Seq("TACAA"),
    Seq("TACGC"),
    Seq("TACAC"),
    Seq("TACCC"),
    Seq("AACCC"),
    Seq("AATGC"),
    Seq("AATGC"),
]

motif = motifs.create(instances)
```

### 模体共有序列与简并序列

```python
# 获取共有序列
print(motif.counts.consensus)

# 获取简并共有序列（IUPAC模糊代码）
print(motif.counts.degenerate_consensus)

# 访问计数矩阵
print(motif.counts)
```

### 位置权重矩阵 (PWM)

```python
# 创建位置权重矩阵
pwm = motif.counts.normalize(pseudocounts=0.5)
print(pwm)

# 计算信息量
ic = motif.counts.information_content()
print(f"信息量: {ic:.2f} 比特")
```

### 模体搜索

```python
from Bio.Seq import Seq

# 在序列中搜索模体
test_seq = Seq("ATACAGGACAGACATACGCATACAACATTACAC")

# 获取位置特异性评分矩阵 (PSSM)
pssm = pwm.log_odds()

# 搜索序列
for position, score in pssm.search(test_seq, threshold=5.0):
    print(f"位置 {position}: 得分 = {score:.2f}")
```

### 从文件读取模体

```python
# 从JASPAR格式读取模体
with open("motif.jaspar") as handle:
    motif = motifs.read(handle, "jaspar")

# 读取多个模体
with open("motifs.jaspar") as handle:
    for m in motifs.parse(handle, "jaspar"):
        print(m.name)

# 支持格式: jaspar, meme, transfac, pfm
```

### 写入模体

```python
# 以JASPAR格式写入模体
with open("output.jaspar", "w") as handle:
    handle.write(motif.format("jaspar"))
```

## 使用 Bio.PopGen 进行群体遗传学分析

### 处理 GenePop 文件

```python
from Bio.PopGen import GenePop

# 读取GenePop文件
with open("data.gen") as handle:
    record = GenePop.read(handle)

# 访问群体
print(f"群体数量: {len(record.populations)}")
print(f"位点: {record.loci_list}")

# 遍历群体
for pop_idx, pop in enumerate(record.populations):
    print(f"\n群体 {pop_idx + 1}:")
    for individual in pop:
        print(f"  {individual[0]}: {individual[1]}")
```

### 计算群体统计量

```python
from Bio.PopGen.GenePop.Controller import GenePopController

# 创建控制器
ctrl = GenePopController()

# 计算基础统计量
result = ctrl.calc_allele_genotype_freqs("data.gen")

# 计算Fst
fst_result = ctrl.calc_fst_all("data.gen")
print(f"Fst: {fst_result}")

# 哈迪-温伯格平衡检验
hw_result = ctrl.test_hw_pop("data.gen", "probability")
```

## 使用 Bio.SeqUtils 的序列工具

### GC含量

```python
from Bio.SeqUtils import gc_fraction
from Bio.Seq import Seq

seq = Seq("ATCGATCGATCG")
gc = gc_fraction(seq)
print(f"GC含量: {gc:.2%}")
```

### 分子量

```python
from Bio.SeqUtils import molecular_weight

# DNA分子量
dna_seq = Seq("ATCG")
mw = molecular_weight(dna_seq, seq_type="DNA")
print(f"DNA分子量: {mw:.2f} g/mol")

# 蛋白质分子量
protein_seq = Seq("ACDEFGHIKLMNPQRSTVWY")
mw = molecular_weight(protein_seq, seq_type="protein")
print(f"蛋白质分子量: {mw:.2f} Da")
```

### 解链温度

```python
from Bio.SeqUtils import MeltingTemp as mt

# 使用最近邻法计算Tm
seq = Seq("ATCGATCGATCG")
tm = mt.Tm_NN(seq)
print(f"Tm: {tm:.1f}°C")

# 使用不同盐浓度
tm = mt.Tm_NN(seq, Na=50, Mg=1.5)  # 50 mM Na+, 1.5 mM Mg2+

# Wallace规则（用于引物）
tm_wallace = mt.Tm_Wallace(seq)
```

### GC偏移

```python
from Bio.SeqUtils import gc_skew

# 计算GC偏移
seq = Seq("ATCGATCGGGCCCAAATTT")
skew = gc_skew(seq, window=100)
print(f"GC偏移: {skew}")
```

### ProtParam - 蛋白质分析

```python
from Bio.SeqUtils.ProtParam import ProteinAnalysis

protein_seq = "ACDEFGHIKLMNPQRSTVWY"
analyzed_seq = ProteinAnalysis(protein_seq)

# 分子量
print(f"分子量: {analyzed_seq.molecular_weight():.2f} Da")

# 等电点
print(f"等电点: {analyzed_seq.isoelectric_point():.2f}")

# 氨基酸组成
print(f"组成: {analyzed_seq.get_amino_acids_percent()}")

# 不稳定指数
print(f"不稳定指数: {analyzed_seq.instability_index():.2f}")

# 芳香性
print(f"芳香性: {analyzed_seq.aromaticity():.2f}")

# 二级结构比例
ss = analyzed_seq.secondary_structure_fraction()
print(f"螺旋: {ss[0]:.2%}, 转角: {ss[1]:.2%}, 折叠: {ss[2]:.2%}")

# 消光系数（假设半胱氨酸还原，无二硫键）
print(f"消光系数: {analyzed_seq.molar_extinction_coefficient()}")

# GRAVY（亲水性总平均值）
print(f"GRAVY: {analyzed_seq.gravy():.3f}")
```

## 使用 Bio.Restriction 进行限制性分析

```python
from Bio import Restriction
from Bio.Seq import Seq

# 分析序列的限制性酶切位点
seq = Seq("GAATTCATCGATCGATGAATTC")

# 使用特定酶
ecori = Restriction.EcoRI
sites = ecori.search(seq)
print(f"EcoRI位点位于: {sites}")

# 使用多个酶
rb = Restriction.RestrictionBatch(["EcoRI", "BamHI", "PstI"])
results = rb.search(seq)
for enzyme, sites in results.items():
    if sites:
        print(f"{enzyme}: {sites}")

# 获取所有可切割序列的酶
all_enzymes = Restriction.Analysis(rb, seq)
print(f"切割酶: {all_enzymes.with_sites()}")
```

## 序列翻译表

```python
from Bio.Data import CodonTable

# 标准遗传密码
standard_table = CodonTable.unambiguous_dna_by_id[1]
print(standard_table)

# 线粒体密码
mito_table = CodonTable.unambiguous_dna_by_id[2]

# 获取特定密码子
print(f"ATG编码: {standard_table.forward_table['ATG']}")

# 获取终止密码子
print(f"终止密码子: {standard_table.stop_codons}")

# 获取起始密码子
print(f"起始密码子: {standard_table.start_codons}")
```

## 使用 Bio.Cluster 进行聚类分析

```python
from Bio.Cluster import kcluster
import numpy as np

# 样本数据矩阵（基因×条件）
data = np.array([
    [1.2, 0.8, 0.5, 1.5],
    [0.9, 1.1, 0.7, 1.3],
    [0.2, 0.3, 2.1, 2.5],
    [0.1, 0.4, 2.3, 2.2],
])

# 执行k均值聚类
clusterid, error, nfound = kcluster(data, nclusters=2)
print(f"聚类分配: {clusterid}")
print(f"误差: {error}")
```

## 使用 GenomeDiagram 绘制基因组图谱

```python
from Bio.Graphics import GenomeDiagram
from Bio.SeqFeature import SeqFeature, FeatureLocation
from Bio import SeqIO
from reportlab.lib import colors

# 读取GenBank文件
record = SeqIO.read("sequence.gb", "genbank")

# 创建图谱
gd_diagram = GenomeDiagram.Diagram("基因组图谱")
gd_track = gd_diagram.new_track(1, greytrack=True)
gd_feature_set = gd_track.new_set()

# 添加特征
for feature in record.features:
    if feature.type == "CDS":
        color = colors.blue
    elif feature.type == "gene":
        color = colors.lightblue
    else:
        color = colors.grey

    gd_feature_set.add_feature(
        feature,
        color=color,
        label=True,
        label_size=6,
        label_angle=45
    )

# 绘制并保存
gd_diagram.draw(format="linear", pagesize="A4", fragments=1)
gd_diagram.write("genome_diagram.pdf", "PDF")
```

## 使用 Bio.pairwise2 进行序列比对

**注意**: Bio.pairwise2 已弃用。请改用 Bio.Align.PairwiseAligner（参见 alignment.md）。

对于遗留代码：

```python
from Bio import pairwise2
from Bio.pairwise2 import format_alignment

# 全局比对
alignments = pairwise2.align.globalxx("ACCGT", "ACGT")

# 打印前三个比对结果
for alignment in alignments[:3]:
    print(format_alignment(*alignment))
```

## 使用 PubChem

```python
from Bio import Entrez

Entrez.email = "your.email@example.com"

# 搜索PubChem
handle = Entrez.esearch(db="pccompound", term="aspirin")
result = Entrez.read(handle)
handle.close()

compound_id = result["IdList"][0]

# 获取化合物信息
handle = Entrez.efetch(db="pccompound", id=compound_id, retmode="xml")
compound_data = handle.read()
handle.close()
```

## 使用 Bio.SeqFeature 处理序列特征

```python
from Bio.SeqFeature import SeqFeature, FeatureLocation
from Bio.Seq import Seq
from Bio.SeqRecord import SeqRecord

# 创建特征
feature = SeqFeature(
    location=FeatureLocation(start=10, end=50),
    type="CDS",
    strand=1,
    qualifiers={"gene": ["ABC1"], "product": ["ABC蛋白"]}
)

# 将特征添加到记录
record = SeqRecord(Seq("ATCG" * 20), id="seq1")
record.features.append(feature)

# 提取特征序列
feature_seq = feature.extract(record.seq)
print(feature_seq)
```

## 序列模糊性处理

```python
from Bio.Data import IUPACData

# DNA模糊代码
print(IUPACData.ambiguous_dna_letters)

# 蛋白质模糊代码
print(IUPACData.ambiguous_protein_letters)

# 解析模糊碱基
print(IUPACData.ambiguous_dna_values["N"])  # 任意碱基
print(IUPACData.ambiguous_dna_values["R"])  # A或G
```

## 质量分数 (FASTQ)

```python
from Bio import SeqIO

# 读取带质量分数的FASTQ文件
for record in SeqIO.parse("reads.fastq", "fastq"):
    print(f"ID: {record.id}")
    print(f"序列: {record.seq}")
    print(f"质量: {record.letter_annotations['phred_quality']}")

    # 计算平均质量
    avg_quality = sum(record.letter_annotations['phred_quality']) / len(record)
    print(f"平均质量: {avg_quality:.2f}")

    # 按质量过滤
    min_quality = min(record.letter_annotations['phred_quality'])
    if min_quality >= 20:
        print("高质量读段")
```

## 最佳实践

1. **使用合适模块** - 为分析选择正确工具
2. **处理伪计数** - 对模体分析至关重要
3. **验证输入数据** - 检查文件格式和数据质量
4. **考虑性能** - 某些操作可能计算密集
5. **缓存结果** - 大型分析中存储中间结果
6. **使用正确遗传密码** - 选择合适的翻译表
7. **记录参数** - 记录使用的阈值和设置
8. **验证统计结果** - 理解检验的局限性
9. **处理边界情况** - 检查空结果或无效输入
10. **组合模块** - 协同使用多个Biopython工具

## 常见用例

### 寻找开放阅读框

```python
from Bio import SeqIO
from Bio.SeqUtils import gc_fraction

def find_orfs(seq, min_length=100):
    """在序列中查找所有开放阅读框"""
    orfs = []

    for strand, nuc in [(+1, seq), (-1, seq.reverse_complement())]:
        for frame in range(3):
            trans = nuc[frame:].translate()
            trans_len = len(trans)

            aa_start = 0
            while aa_start < trans_len:
                aa_end = trans.find("*", aa_start)
                if aa_end == -1:
                    aa_end = trans_len

                if aa_end - aa_start >= min_length // 3:
                    start = frame + aa_start * 3
                    end = frame + aa_end * 3
                    orfs.append({
                        'start': start,
                        'end': end,
                        'strand': strand,
                        'frame': frame,
                        'length': end - start,
                        'sequence': nuc[start:end]
                    })

                aa_start = aa_end + 1

    return orfs

# 使用示例
record = SeqIO.read("sequence.fasta", "fasta")
orfs = find_orfs(record.seq, min_length=300)
for orf in orfs:
    print(f"ORF: {orf['start']}-{orf['end']}, 链={orf['strand']}, 长度={orf['length']}")
```

### 分析密码子使用频率

```python
from Bio import SeqIO
from Bio.SeqUtils import CodonUsage

def analyze_codon_usage(fasta_file):
    """分析编码序列中的密码子使用频率"""
    codon_counts = {}

    for record in SeqIO.parse(fasta_file, "fasta"):
        # 确保序列长度为3的倍数
        seq = record.seq[:len(record.seq) - len(record.seq) % 3]

        # 计数密码子
        for i in range(0, len(seq), 3):
            codon = str(seq[i:i+3])
            codon_counts[codon] = codon_counts.get(codon, 0) + 1

    # 计算频率
    total = sum(codon_counts.values())
    codon_freq = {k: v/total for k, v in codon_counts.items()}

    return codon_freq
```

### 计算序列复杂度

```python
def sequence_complexity(seq, k=2):
    """计算k-mer复杂度（香农熵）"""
    import math
    from collections import Counter

    # 生成k-mer
    kmers = [str(seq[i:i+k]) for i in range(len(seq) - k + 1)]

    # 计数k-mer
    counts = Counter(kmers)
    total = len(kmers)

    # 计算熵值
    entropy = 0
    for count in counts.values():
        freq = count / total
        entropy -= freq * math.log2(freq)

    # 按最大可能熵归一化
    max_entropy = math.log2(4 ** k)  # 对于DNA序列

    return entropy / max_entropy if max_entropy > 0 else 0

# 使用示例
from Bio.Seq import Seq
seq = Seq("ATCGATCGATCGATCG")
complexity = sequence_complexity(seq, k=2)
print(f"序列复杂度: {complexity:.3f}")
```

### 提取启动子区域

```python
def extract_promoters(genbank_file, upstream=500):
    """提取基因上游的启动子区域"""
    from Bio import SeqIO

    record = SeqIO.read(genbank_file, "genbank")
    promoters =

promoter_seq = record.seq[start:end]
            if feature.strand == -1:
                promoter_seq = promoter_seq.reverse_complement()

            promoters.append({
                'gene': feature.qualifiers.get('gene', ['未知'])[0],
                'sequence': promoter_seq,
                'start': start,
                'end': end
            })

    return promoters
```
