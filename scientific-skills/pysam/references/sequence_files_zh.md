# 处理序列文件（FASTA/FASTQ）

## FASTA文件

### 概述

Pysam提供`FastaFile`类用于对FASTA参考序列进行带索引的随机访问。FASTA文件在使用前必须通过`samtools faidx`建立索引。

### 打开FASTA文件

```python
import pysam

# 打开带索引的FASTA文件
fasta = pysam.FastaFile("reference.fasta")

# 自动查找reference.fasta.fai索引文件
```

### 创建FASTA索引

```python
# 使用pysam创建索引
pysam.faidx("reference.fasta")

# 或使用samtools命令
pysam.samtools.faidx("reference.fasta")
```

这将创建随机访问所需的`.fai`索引文件。

### FastaFile属性

```python
fasta = pysam.FastaFile("reference.fasta")

# 参考序列列表
references = fasta.references
print(f"参考序列: {references}")

# 获取长度
lengths = fasta.lengths
print(f"长度: {lengths}")

# 获取特定序列长度
chr1_length = fasta.get_reference_length("chr1")
```

### 获取序列

#### 按区域获取

使用**0-based半开区间**坐标：

```python
# 获取特定区域
sequence = fasta.fetch("chr1", 1000, 2000)
print(f"序列: {sequence}")  # 返回1000个碱基

# 获取整条染色体
chr1_seq = fasta.fetch("chr1")

# 使用区域字符串获取（1-based）
sequence = fasta.fetch(region="chr1:1001-2000")
```

**重要提示：** 数值参数使用0-based坐标，区域字符串使用1-based坐标（samtools惯例）。

#### 常见用例

```python
# 获取变异位点周围的序列
def get_variant_context(fasta, chrom, pos, window=10):
    """获取变异位点周围的序列上下文（1-based位置）"""
    start = max(0, pos - window - 1)  # 转换为0-based
    end = pos + window
    return fasta.fetch(chrom, start, end)

# 获取基因坐标对应的序列
def get_gene_sequence(fasta, chrom, start, end, strand):
    """获取链特异性基因序列"""
    seq = fasta.fetch(chrom, start, end)

    if strand == "-":
        # 反向互补
        complement = str.maketrans("ATGCatgc", "TACGtacg")
        seq = seq.translate(complement)[::-1]

    return seq

# 验证参考等位基因
def check_ref_allele(fasta, chrom, pos, expected_ref):
    """验证位置上的参考等位基因（1-based位置）"""
    actual = fasta.fetch(chrom, pos-1, pos)  # 转换为0-based
    return actual.upper() == expected_ref.upper()
```

### 提取多个区域

```python
# 高效提取多个区域
regions = [
    ("chr1", 1000, 2000),
    ("chr1", 5000, 6000),
    ("chr2", 10000, 11000)
]

sequences = {}
for chrom, start, end in regions:
    seq_id = f"{chrom}:{start}-{end}"
    sequences[seq_id] = fasta.fetch(chrom, start, end)
```

### 处理模糊碱基

FASTA文件可能包含IUPAC模糊代码：

- N = 任意碱基
- R = A或G（嘌呤）
- Y = C或T（嘧啶）
- S = G或C（强相互作用）
- W = A或T（弱相互作用）
- K = G或T（酮基）
- M = A或C（氨基）
- B = C、G或T（非A）
- D = A、G或T（非C）
- H = A、C或T（非G）
- V = A、C或G（非T）

```python
# 处理模糊碱基
def count_ambiguous(sequence):
    """统计非ATGC碱基数量"""
    return sum(1 for base in sequence.upper() if base not in "ATGC")

# 过滤高N含量区域
def has_quality_sequence(fasta, chrom, start, end, max_n_frac=0.1):
    """检查区域N碱基含量是否可接受"""
    seq = fasta.fetch(chrom, start, end)
    n_count = seq.upper().count('N')
    return (n_count / len(seq)) <= max_n_frac
```

## FASTQ文件

### 概述

Pysam提供`FastxFile`（或`FastqFile`）用于读取包含原始测序读段和质量分数的FASTQ文件。FASTQ文件不支持随机访问——仅支持顺序读取。

### 打开FASTQ文件

```python
import pysam

# 打开FASTQ文件
fastq = pysam.FastxFile("reads.fastq")

# 支持压缩文件
fastq_gz = pysam.FastxFile("reads.fastq.gz")
```

### 读取FASTQ记录

```python
fastq = pysam.FastxFile("reads.fastq")

for read in fastq:
    print(f"名称: {read.name}")
    print(f"序列: {read.sequence}")
    print(f"质量值: {read.quality}")
    print(f"注释: {read.comment}")  # 可选的头部注释
```

**FastqProxy属性：**
- `name` - 读段标识符（不含@前缀）
- `sequence` - DNA/RNA序列
- `quality` - ASCII编码的质量字符串
- `comment` - 头部行的可选注释
- `get_quality_array()` - 将质量字符串转换为数值数组

### 质量分数转换

```python
# 将质量字符串转换为数值
for read in fastq:
    qual_array = read.get_quality_array()
    mean_quality = sum(qual_array) / len(qual_array)
    print(f"{read.name}: 平均质量值 Q = {mean_quality:.1f}")
```

质量分数采用Phred标度（通常为Phred+33编码）：
- Q = -10 * log10(错误率)
- ASCII 33 ('!') = Q0
- ASCII 43 ('+') = Q10
- ASCII 63 ('?') = Q30

### 常见FASTQ处理流程

#### 质量过滤

```python
def filter_by_quality(input_fastq, output_fastq, min_mean_quality=20):
    """根据平均质量分数过滤读段"""
    with pysam.FastxFile(input_fastq) as infile:
        with open(output_fastq, 'w') as outfile:
            for read in infile:
                qual_array = read.get_quality_array()
                mean_q = sum(qual_array) / len(qual_array)

                if mean_q >= min_mean_quality:
                    # 以FASTQ格式写入
                    outfile.write(f"@{read.name}\n")
                    outfile.write(f"{read.sequence}\n")
                    outfile.write("+\n")
                    outfile.write(f"{read.quality}\n")
```

#### 长度过滤

```python
def filter_by_length(input_fastq, output_fastq, min_length=50):
    """根据最小长度过滤读段"""
    with pysam.FastxFile(input_fastq) as infile:
        with open(output_fastq, 'w') as outfile:
            kept = 0
            for read in infile:
                if len(read.sequence) >= min_length:
                    outfile.write(f"@{read.name}\n")
                    outfile.write(f"{read.sequence}\n")
                    outfile.write("+\n")
                    outfile.write(f"{read.quality}\n")
                    kept += 1
    print(f"保留读段数: {kept}")
```

#### 计算质量统计

```python
def calculate_fastq_stats(fastq_file):
    """计算FASTQ文件基础统计"""
    total_reads = 0
    total_bases = 0
    quality_sum = 0

    with pysam.FastxFile(fastq_file) as fastq:
        for read in fastq:
            total_reads += 1
            read_length = len(read.sequence)
            total_bases += read_length

            qual_array = read.get_quality_array()
            quality_sum += sum(qual_array)

    return {
        "总读段数": total_reads,
        "总碱基数": total_bases,
        "平均读长": total_bases / total_reads if total_reads > 0 else 0,
        "平均质量值": quality_sum / total_bases if total_bases > 0 else 0
    }
```

#### 按名称提取读段

```python
def extract_reads_by_name(fastq_file, read_names, output_file):
    """按名称提取特定读段"""
    read_set = set(read_names)

    with pysam.FastxFile(fastq_file) as infile:
        with open(output_file, 'w') as outfile:
            for read in infile:
                if read.name in read_set:
                    outfile.write(f"@{read.name}\n")
                    outfile.write(f"{read.sequence}\n")
                    outfile.write("+\n")
                    outfile.write(f"{read.quality}\n")
```

#### FASTQ转FASTA

```python
def fastq_to_fasta(fastq_file, fasta_file):
    """将FASTQ转换为FASTA（丢弃质量分数）"""
    with pysam.FastxFile(fastq_file) as infile:
        with open(fasta_file, 'w') as outfile:
            for read in infile:
                outfile.write(f">{read.name}\n")
                outfile.write(f"{read.sequence}\n")
```

#### FASTQ子采样

```python
import random

def subsample_fastq(input_fastq, output_fastq, fraction=0.1, seed=42):
    """从FASTQ文件中随机子采样读段"""
    random.seed(seed)

    with pysam.FastxFile(input_fastq) as infile:
        with open(output_fastq, 'w') as outfile:
            for read in infile:
                if random.random() < fraction:
                    outfile.write(f"@{read.name}\n")
                    outfile.write(f"{read.sequence}\n")
                    outfile.write("+\n")
                    outfile.write(f"{read.quality}\n")
```

## Tabix索引文件

### 概述

Pysam提供`TabixFile`用于访问带tabix索引的基因组数据文件（BED、GFF、GTF、通用制表符分隔文件）。

### 打开Tabix文件

```python
import pysam

# 打开带tabix索引的文件
tabix = pysam.TabixFile("annotations.bed.gz")

# 文件必须经过bgzip压缩并建立tabix索引
```

### 创建Tabix索引

```python
# 为文件建立索引
pysam.tabix_index("annotations.bed", preset="bed", force=True)
# 生成annotations.bed.gz和annotations.bed.gz.tbi

# 可用预设：bed, gff, vcf
```

### 获取记录

```python
tabix = pysam.TabixFile("annotations.bed.gz")

# 获取区域
for row in tabix.fetch("chr1", 1000000, 2000000):
    print(row)  # 返回制表符分隔的字符串

# 使用特定解析器
for row in tabix.fetch("chr1", 1000000, 2000000, parser=pysam.asBed()):
    print(f"区间: {row.contig}:{row.start}-{row.end}")

# 可用解析器：asBed(), asGTF(), asVCF(), asTuple()
```

### 处理BED文件

```python
bed = pysam.TabixFile("regions.bed.gz")

# 按字段名访问BED数据
for interval in bed.fetch("chr1", 1000000, 2000000, parser=pysam.asBed()):
    print(f"区域: {interval.contig}:{interval.start}-{interval.end}")
    print(f"名称: {interval.name}")
    print(f"分数: {interval.score}")
    print(f"链: {interval.strand}")
```

### 处理GTF/GFF文件

```python
gtf = pysam.TabixFile("annotations.gtf.gz")

# 访问GTF字段
for feature in gtf.fetch("chr1", 1000000, 2000000, parser=pysam.asGTF()):
    print(f"特征类型: {feature.feature}")
    print(f"基因ID: {feature.gene_id}")
    print(f"转录本ID: {feature.transcript_id}")
    print(f"坐标: {feature.start}-{feature.end}")
```

## 性能优化建议

### FASTA
1. **始终使用带索引的FASTA**文件（用samtools faidx创建.fai）
2. **批量执行提取操作**处理多个区域时
3. **缓存频繁访问的序列**到内存中
4. **使用合适的窗口大小**避免加载过量序列数据

### FASTQ
1. **流式处理** - FASTQ文件顺序读取，实时处理
2. **使用压缩的FASTQ.gz**节省磁盘空间（pysam自动处理）
3. **避免全文件加载内存**——逐条处理读段
4. **大文件处理**可考虑文件分片并行处理

### Tabix
1. **必须使用bgzip压缩并建立tabix索引**
2. **创建索引时选用合适预设**
3. **指定解析器**实现字段名访问
4. **批量查询**相同文件避免重复打开

## 常见陷阱

1. **FASTA坐标系统：** fetch()使用0-based坐标，区域字符串使用1-based
2. **缺失索引：** FASTA随机访问需要.fai索引文件
3. **FASTQ仅顺序访问：** 不支持随机访问或区域查询
4. **质量编码：** 除非特别说明，默认使用Phred+33编码
5. **Tabix压缩要求：** 必须使用bgzip而非普通gzip
6. **解析器要求：** TabixFile需要显式解析器才能按字段名访问
7. **大小写敏感：** FASTA序列保留大小写——使用.upper()或.lower()确保一致性
