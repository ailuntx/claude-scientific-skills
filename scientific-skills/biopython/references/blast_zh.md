# 使用 Bio.Blast 进行 BLAST 操作

## 概述

Bio.Blast 提供了运行 BLAST 搜索（本地和通过 NCBI 网络服务）以及解析多种格式 BLAST 结果的工具。该模块处理了提交查询和解析输出的复杂性。

## 通过 NCBI 网络服务运行 BLAST

### Bio.Blast.NCBIWWW

`qblast()` 函数将序列提交至 NCBI 的在线 BLAST 服务：

```python
from Bio.Blast import NCBIWWW
from Bio import SeqIO

# 从文件读取序列
record = SeqIO.read("sequence.fasta", "fasta")

# 运行 BLAST 搜索
result_handle = NCBIWWW.qblast(
    program="blastn",           # BLAST 程序
    database="nt",              # 搜索数据库
    sequence=str(record.seq)    # 查询序列
)

# 保存结果
with open("blast_results.xml", "w") as out_file:
    out_file.write(result_handle.read())
result_handle.close()
```

### 可用的 BLAST 程序

- **blastn** - 核苷酸 vs 核苷酸
- **blastp** - 蛋白质 vs 蛋白质
- **blastx** - 翻译后的核苷酸 vs 蛋白质
- **tblastn** - 蛋白质 vs 翻译后的核苷酸
- **tblastx** - 翻译后的核苷酸 vs 翻译后的核苷酸

### 常用数据库

**核苷酸数据库：**
- `nt` - 所有 GenBank+EMBL+DDBJ+PDB 序列
- `refseq_rna` - RefSeq RNA 序列

**蛋白质数据库：**
- `nr` - 所有非冗余 GenBank CDS 翻译
- `refseq_protein` - RefSeq 蛋白质序列
- `pdb` - 蛋白质数据银行序列
- `swissprot` - 精选的 UniProtKB/Swiss-Prot

### 高级 qblast 参数

```python
result_handle = NCBIWWW.qblast(
    program="blastn",
    database="nt",
    sequence=str(record.seq),
    expect=0.001,              # E 值阈值
    hitlist_size=50,           # 返回的命中数
    alignments=25,             # 显示的比对数
    word_size=11,              # 初始匹配的字长
    gapcosts="5 2",            # 空位成本（开放 延伸）
    format_type="XML"          # 输出格式（默认）
)
```

### 使用序列文件或 ID

```python
# 使用 FASTA 格式字符串
fasta_string = open("sequence.fasta").read()
result_handle = NCBIWWW.qblast("blastn", "nt", fasta_string)

# 使用 GenBank ID
result_handle = NCBIWWW.qblast("blastn", "nt", "EU490707")

# 使用 GI 编号
result_handle = NCBIWWW.qblast("blastn", "nt", "160418")
```

## 解析 BLAST 结果

### Bio.Blast.NCBIXML

NCBIXML 提供 BLAST XML 输出解析器（推荐格式）：

```python
from Bio.Blast import NCBIXML

# 解析单个 BLAST 结果
with open("blast_results.xml") as result_handle:
    blast_record = NCBIXML.read(result_handle)
```

### 访问 BLAST 记录数据

```python
# 查询信息
print(f"查询: {blast_record.query}")
print(f"查询长度: {blast_record.query_length}")
print(f"数据库: {blast_record.database}")
print(f"数据库序列数量: {blast_record.database_sequences}")

# 遍历比对（命中）
for alignment in blast_record.alignments:
    print(f"\n命中: {alignment.title}")
    print(f"长度: {alignment.length}")
    print(f"访问号: {alignment.accession}")

    # 每个比对可包含多个 HSP（高得分片段对）
    for hsp in alignment.hsps:
        print(f"  E 值: {hsp.expect}")
        print(f"  得分: {hsp.score}")
        print(f"  比特值: {hsp.bits}")
        print(f"  一致性: {hsp.identities}/{hsp.align_length}")
        print(f"  空位: {hsp.gaps}")
        print(f"  查询序列: {hsp.query}")
        print(f"  匹配符号: {hsp.match}")
        print(f"  目标序列: {hsp.sbjct}")
```

### 结果过滤

```python
# 仅显示 E 值 < 0.001 的命中
E_VALUE_THRESH = 0.001

for alignment in blast_record.alignments:
    for hsp in alignment.hsps:
        if hsp.expect < E_VALUE_THRESH:
            print(f"命中: {alignment.title}")
            print(f"E 值: {hsp.expect}")
            print(f"一致性: {hsp.identities}/{hsp.align_length}")
            print()
```

### 多个 BLAST 结果

对于包含多个 BLAST 结果的文件（例如批量搜索）：

```python
from Bio.Blast import NCBIXML

with open("batch_blast_results.xml") as result_handle:
    blast_records = NCBIXML.parse(result_handle)

    for blast_record in blast_records:
        print(f"\n查询: {blast_record.query}")
        print(f"命中数: {len(blast_record.alignments)}")

        if blast_record.alignments:
            # 获取最佳命中
            best_alignment = blast_record.alignments[0]
            best_hsp = best_alignment.hsps[0]
            print(f"最佳命中: {best_alignment.title}")
            print(f"E 值: {best_hsp.expect}")
```

## 运行本地 BLAST

### 先决条件

本地 BLAST 需要：
1. 安装 BLAST+ 命令行工具
2. 本地下载 BLAST 数据库

### 使用命令行封装器

```python
from Bio.Blast.Applications import NcbiblastnCommandline

# 设置 BLAST 命令
blastn_cline = NcbiblastnCommandline(
    query="input.fasta",
    db="local_database",
    evalue=0.001,
    outfmt=5,                    # XML 格式
    out="results.xml"
)

# 运行 BLAST
stdout, stderr = blastn_cline()

# 解析结果
from Bio.Blast import NCBIXML
with open("results.xml") as result_handle:
    blast_record = NCBIXML.read(result_handle)
```

### 可用的命令行封装器

- `NcbiblastnCommandline` - BLASTN 封装器
- `NcbiblastpCommandline` - BLASTP 封装器
- `NcbiblastxCommandline` - BLASTX 封装器
- `NcbitblastnCommandline` - TBLASTN 封装器
- `NcbitblastxCommandline` - TBLASTX 封装器

### 创建 BLAST 数据库

```python
from Bio.Blast.Applications import NcbimakeblastdbCommandline

# 创建核苷酸数据库
makedb_cline = NcbimakeblastdbCommandline(
    input_file="sequences.fasta",
    dbtype="nucl",
    out="my_database"
)
stdout, stderr = makedb_cline()
```

## 分析 BLAST 结果

### 提取最佳命中

```python
def get_best_hits(blast_record, num_hits=10, e_value_thresh=0.001):
    """从 BLAST 记录中提取最佳命中"""
    hits = []
    for alignment in blast_record.alignments[:num_hits]:
        for hsp in alignment.hsps:
            if hsp.expect < e_value_thresh:
                hits.append({
                    'title': alignment.title,
                    'accession': alignment.accession,
                    'length': alignment.length,
                    'e_value': hsp.expect,
                    'score': hsp.score,
                    'identities': hsp.identities,
                    'align_length': hsp.align_length,
                    'query_start': hsp.query_start,
                    'query_end': hsp.query_end,
                    'sbjct_start': hsp.sbjct_start,
                    'sbjct_end': hsp.sbjct_end
                })
                break  # 每个比对仅取最佳 HSP
    return hits
```

### 计算一致性百分比

```python
def calculate_percent_identity(hsp):
    """计算 HSP 的一致性百分比"""
    return (hsp.identities / hsp.align_length) * 100

# 使用示例
for alignment in blast_record.alignments:
    for hsp in alignment.hsps:
        if hsp.expect < 0.001:
            identity = calculate_percent_identity(hsp)
            print(f"{alignment.title}: {identity:.2f}% 一致性")
```

### 提取命中序列

```python
from Bio import Entrez, SeqIO

Entrez.email = "your.email@example.com"

def fetch_hit_sequences(blast_record, num_sequences=5):
    """获取 BLAST 最佳命中的序列"""
    sequences = []

    for alignment in blast_record.alignments[:num_sequences]:
        accession = alignment.accession

        # 从 GenBank 获取序列
        handle = Entrez.efetch(
            db="nucleotide",
            id=accession,
            rettype="fasta",
            retmode="text"
        )
        record = SeqIO.read(handle, "fasta")
        handle.close()

        sequences.append(record)

    return sequences
```

## 解析其他 BLAST 格式

### 制表符分隔输出 (outfmt 6/7)

```python
# 运行表格输出格式的 BLAST
blastn_cline = NcbiblastnCommandline(
    query="input.fasta",
    db="database",
    outfmt=6,
    out="results.txt"
)

# 解析表格结果
with open("results.txt") as f:
    for line in f:
        fields = line.strip().split('\t')
        query_id = fields[0]
        subject_id = fields[1]
        percent_identity = float(fields[2])
        align_length = int(fields[3])
        e_value = float(fields[10])
        bit_score = float(fields[11])

        print(f"{query_id} -> {subject_id}: {percent_identity}% 一致性, E={e_value}")
```

### 自定义输出格式

```python
# 指定自定义列（outfmt 6 自定义字段）
blastn_cline = NcbiblastnCommandline(
    query="input.fasta",
    db="database",
    outfmt="6 qseqid sseqid pident length evalue bitscore qseq sseq",
    out="results.txt"
)
```

## 最佳实践

1. **使用 XML 格式**进行解析 (outfmt 5) - 最可靠完整
2. **保存 BLAST 结果** - 避免不必要的重复搜索
3. **设置合适的 E 值阈值** - 默认值为 10，但 0.001-0.01 通常更佳
4. **处理速率限制** - NCBI 限制请求频率
5. **使用本地 BLAST** - 适用于大规模搜索或重复查询
6. **缓存结果** - 保存解析数据避免重复解析
7. **检查空结果** - 优雅处理无命中的情况
8. **考虑替代方案** - 大型数据集可考虑 DIAMOND 等快速比对工具
9. **批量搜索** - 尽可能同时提交多个序列
10. **按一致性过滤** - 仅靠 E 值可能不足

## 常见用例

### 基础 BLAST 搜索与解析

```python
from Bio.Blast import NCBIWWW, NCBIXML
from Bio import SeqIO

# 读取查询序列
record = SeqIO.read("query.fasta", "fasta")

# 运行 BLAST
print("正在运行 BLAST 搜索...")
result_handle = NCBIWWW.qblast("blastn", "nt", str(record.seq))

# 解析结果
blast_record = NCBIXML.read(result_handle)

# 显示前 5 个命中
print(f"\n{blast_record.query} 的前 5 个命中:")
for i, alignment in enumerate(blast_record.alignments[:5], 1):
    hsp = alignment.hsps[0]
    identity = (hsp.identities / hsp.align_length) * 100
    print(f"{i}. {alignment.title}")
    print(f"   E 值: {hsp.expect}, 一致性: {identity:.1f}%")
```

### 寻找直系同源物

```python
from Bio.Blast import NCBIWWW, NCBIXML
from Bio import Entrez, SeqIO

Entrez.email = "your.email@example.com"

# 查询基因序列
query_record = SeqIO.read("gene.fasta", "fasta")

# 针对特定生物进行 BLAST
result_handle = NCBIWWW.qblast(
    "blastn",
    "nt",
    str(query_record.seq),
    entrez_query="Mus musculus[Organism]"  # 限定为小鼠
)

blast_record = NCBIXML.read(result_handle)

# 寻找最佳命中
if blast_record.alignments:
    best_hit = blast_record.alignments[0]
    print(f"潜在直系同源物: {best_hit.title}")
    print(f"访问号: {best_hit.accession}")
```

### 批量 BLAST 多个序列

```python
from Bio.Blast import NCBIWWW, NCBIXML
from Bio import SeqIO

# 读取多个序列
sequences = list(SeqIO.parse("queries.fasta", "fasta"))

# 创建批量结果文件
with open("batch_results.xml", "w") as out_file:
    for seq_record in sequences:
        print(f"正在搜索 {seq_record.id}...")

        result_handle = NCBIWWW.qblast("blastn", "nt", str(seq_record.seq))
        out_file.write(result_handle.read())
        result_handle.close()

# 解析批量结果
with open("batch_results.xml") as result_handle:
    for blast_record in NCBIXML.parse(result_handle):
        print(f"\n{blast_record.query}: {len(blast_record.alignments)} 个命中")
```

### 互惠最佳命中

```python
def reciprocal_best_hit(seq1_id, seq2_id, database="nr", program="blastp"):
    """检查两个序列是否为互惠最佳命中"""
    from Bio.Blast import NCBIWWW, NCBIXML
    from Bio import Entrez

    Entrez.email = "your.email@example.com"

    # 正向 BLAST
    result1 = NCBIWWW.qblast(program, database, seq1_id)
    record1 = NCBIXML.read(result1)
    best_hit1 = record1.alignments[0].accession if record1.alignments else None

    # 反向 BLAST
    result2 = NCBIWWW.qblast(program, database, seq2_id)
    record2 = NCBIXML.read(result2)
    best_hit2 = record2.alignments[0].accession if record2.alignments else None

    # 检查互惠性
    return best_hit1 == seq2_id and best_hit2 == seq1_id
```

## 错误处理

```python
from Bio.Blast import NCBIWWW, NCBIXML
from urllib.error import HTTPError

try:
    result_handle = NCBIWWW.qblast("blastn", "nt", "ATCGATCGATCG")
    blast_record = NCBIXML.read(result_handle)
    result_handle.close()
except HTTPError as e:
    print(f"HTTP 错误: {e.code}")
except Exception as e:
    print(f"运行 BLAST 出错: {e}")
```
