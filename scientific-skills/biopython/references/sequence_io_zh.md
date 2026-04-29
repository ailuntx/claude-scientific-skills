# 使用 Bio.Seq 和 Bio.SeqIO 处理序列

## 概述

Bio.Seq 提供具有专用方法的 `Seq` 对象用于处理生物序列，而 Bio.SeqIO 则为跨多种格式读取、写入和转换序列文件提供了统一接口。

## Seq 对象

### 创建序列

```python
from Bio.Seq import Seq

# 创建基础序列
my_seq = Seq("AGTACACTGGT")

# 序列支持类字符串操作
print(len(my_seq))  # 长度
print(my_seq[0:5])  # 切片
```

### 核心序列操作

```python
# 互补链与反向互补链
complement = my_seq.complement()
rev_comp = my_seq.reverse_complement()

# 转录（DNA 转 RNA）
rna = my_seq.transcribe()

# 翻译（转蛋白质）
protein = my_seq.translate()

# 反向转录（RNA 转 DNA）
dna = rna_seq.back_transcribe()
```

### 序列方法

- `complement()` - 返回互补链
- `reverse_complement()` - 返回反向互补链
- `transcribe()` - DNA 到 RNA 的转录
- `back_transcribe()` - RNA 到 DNA 的转换
- `translate()` - 翻译为蛋白质序列
- `translate(table=N)` - 使用特定遗传密码表
- `translate(to_stop=True)` - 在第一个终止密码子处停止

## Bio.SeqIO：序列文件输入输出

### 核心函数

**Bio.SeqIO.parse()**：读取序列文件的主要工具，返回 `SeqRecord` 对象的迭代器。

```python
from Bio import SeqIO

# 解析 FASTA 文件
for record in SeqIO.parse("sequences.fasta", "fasta"):
    print(record.id)
    print(record.seq)
    print(len(record))
```

**Bio.SeqIO.read()**：用于单记录文件（验证仅存在一条记录）。

```python
record = SeqIO.read("single.fasta", "fasta")
```

**Bio.SeqIO.write()**：将 SeqRecord 对象输出到文件。

```python
# 将记录写入文件
count = SeqIO.write(seq_records, "output.fasta", "fasta")
print(f"写入 {count} 条记录")
```

**Bio.SeqIO.convert()**：简化的格式转换工具。

```python
# 格式间转换
count = SeqIO.convert("input.gbk", "genbank", "output.fasta", "fasta")
```

### 支持的文件格式

常见格式包括：
- **fasta** - FASTA 格式
- **fastq** - FASTQ 格式（含质量分数）
- **genbank** 或 **gb** - GenBank 格式
- **embl** - EMBL 格式
- **swiss** - SwissProt 格式
- **fasta-2line** - 单行序列的 FASTA
- **tab** - 简单制表符分隔格式

### SeqRecord 对象

`SeqRecord` 对象整合序列数据与注释信息：

```python
record.id          # 主标识符
record.name        # 短名称
record.description # 描述行
record.seq         # 实际序列（Seq 对象）
record.annotations # 附加信息字典
record.features    # SeqFeature 对象列表
record.letter_annotations  # 逐字符注释（如质量分数）
```

### 修改记录

```python
# 修改记录属性
record.id = "new_id"
record.description = "新描述"

# 提取子序列
sub_record = record[10:30]  # 切片操作保留注释

# 修改序列
record.seq = record.seq.reverse_complement()
```

## 处理大文件

### 内存高效解析

使用迭代器避免将整个文件加载到内存：

```python
# 适用于大文件
for record in SeqIO.parse("large_file.fasta", "fasta"):
    if len(record.seq) > 1000:
        print(record.id)
```

### 基于字典的访问

三种随机访问方法：

**1. Bio.SeqIO.to_dict()** - 将所有记录加载到内存：

```python
seq_dict = SeqIO.to_dict(SeqIO.parse("sequences.fasta", "fasta"))
record = seq_dict["sequence_id"]
```

**2. Bio.SeqIO.index()** - 惰性加载字典（内存高效）：

```python
seq_index = SeqIO.index("sequences.fasta", "fasta")
record = seq_index["sequence_id"]
seq_index.close()
```

**3. Bio.SeqIO.index_db()** - 基于 SQLite 的索引，适用于超大文件：

```python
seq_index = SeqIO.index_db("index.idx", "sequences.fasta", "fasta")
record = seq_index["sequence_id"]
seq_index.close()
```

### 高性能底层解析器

处理高通量测序数据时，使用返回元组而非对象的底层解析器：

```python
from Bio.SeqIO.FastaIO import SimpleFastaParser

with open("sequences.fasta") as handle:
    for title, sequence in SimpleFastaParser(handle):
        print(title, len(sequence))

from Bio.SeqIO.QualityIO import FastqGeneralIterator

with open("reads.fastq") as handle:
    for title, sequence, quality in FastqGeneralIterator(handle):
        print(title)
```

## 压缩文件处理

Bio.SeqIO 自动处理压缩文件：

```python
# 支持 gzip 压缩
for record in SeqIO.parse("sequences.fasta.gz", "fasta"):
    print(record.id)

# 支持 BGZF 格式随机访问
from Bio import bgzf
with bgzf.open("sequences.fasta.bgz", "r") as handle:
    records = SeqIO.parse(handle, "fasta")
```

## 数据提取模式

### 提取特定信息

```python
# 获取所有 ID
ids = [record.id for record in SeqIO.parse("file.fasta", "fasta")]

# 获取长度超过阈值的序列
long_seqs = [record for record in SeqIO.parse("file.fasta", "fasta")
             if len(record.seq) > 500]

# 从 GenBank 提取生物体信息
for record in SeqIO.parse("file.gbk", "genbank"):
    organism = record.annotations.get("organism", "未知")
    print(f"{record.id}: {organism}")
```

### 筛选与写入

```python
# 按条件筛选序列
long_sequences = (record for record in SeqIO.parse("input.fasta", "fasta")
                  if len(record) > 500)
SeqIO.write(long_sequences, "filtered.fasta", "fasta")
```

## 最佳实践

1. **使用迭代器**处理大文件，避免全量加载到内存
2. **优先选择 index()** 实现大文件的重复随机访问
3. **使用 index_db()** 处理数百万记录或多文件场景
4. **采用底层解析器**处理高通量数据以提升速度
5. **本地下载后复用**，避免重复网络访问
6. **显式关闭索引文件**或使用上下文管理器
7. **写入前验证输入**（通过 SeqIO.write()）
8. **使用正确格式字符串** - 始终小写（如 "fasta" 而非 "FASTA"）

## 常见用例

### 格式转换

```python
# GenBank 转 FASTA
SeqIO.convert("input.gbk", "genbank", "output.fasta", "fasta")

# 多格式批量转换
for fmt in ["fasta", "genbank", "embl"]:
    SeqIO.convert("input.fasta", "fasta", f"output.{fmt}", fmt)
```

### 质量筛选（FASTQ）

```python
from Bio import SeqIO

good_reads = (record for record in SeqIO.parse("reads.fastq", "fastq")
              if min(record.letter_annotations["phred_quality"]) >= 20)
count = SeqIO.write(good_reads, "filtered.fastq", "fastq")
```

### 序列统计

```python
from Bio.SeqUtils import gc_fraction

for record in SeqIO.parse("sequences.fasta", "fasta"):
    gc = gc_fraction(record.seq)
    print(f"{record.id}: GC含量={gc:.2%}, 长度={len(record)}")
```

### 编程创建记录

```python
from Bio.Seq import Seq
from Bio.SeqRecord import SeqRecord

# 创建新记录
new_record = SeqRecord(
    Seq("ATGCGATCGATCG"),
    id="seq001",
    name="MySequence",
    description="测试序列"
)

# 写入文件
SeqIO.write([new_record], "new.fasta", "fasta")
```
