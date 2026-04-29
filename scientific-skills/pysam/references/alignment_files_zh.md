# 处理比对文件 (SAM/BAM/CRAM)

## 概述

Pysam 提供了 `AlignmentFile` 类用于读写包含比对序列数据的 SAM/BAM/CRAM 格式文件。BAM/CRAM 文件支持通过索引进行压缩和随机访问。

## 打开比对文件

通过模式限定符指定格式：
- `"rb"` - 读取 BAM（二进制）
- `"r"` - 读取 SAM（文本）
- `"rc"` - 读取 CRAM（压缩格式）
- `"wb"` - 写入 BAM
- `"w"` - 写入 SAM
- `"wc"` - 写入 CRAM

```python
import pysam

# 读取
samfile = pysam.AlignmentFile("example.bam", "rb")

# 写入（需要模板或头部信息）
outfile = pysam.AlignmentFile("output.bam", "wb", template=samfile)
```

### 流处理

使用 `"-"` 作为文件名进行 stdin/stdout 操作：

```python
# 从 stdin 读取
infile = pysam.AlignmentFile('-', 'rb')

# 写入 stdout
outfile = pysam.AlignmentFile('-', 'w', template=infile)
```

**重要提示：** Pysam 不支持从真正的 Python 文件对象读写——仅支持 stdin/stdout 流。

## AlignmentFile 属性

**头部信息：**
- `references` - 染色体/重叠群名称列表
- `lengths` - 每个参考序列对应的长度
- `header` - 完整的头部字典

```python
samfile = pysam.AlignmentFile("example.bam", "rb")
print(f"References: {samfile.references}")
print(f"Lengths: {samfile.lengths}")
```

## 读取序列

### fetch() - 基于区域的检索

使用**0起始坐标**检索与指定基因组区域重叠的序列。

```python
# 获取特定区域
for read in samfile.fetch("chr1", 1000, 2000):
    print(read.query_name, read.reference_start)

# 获取整个重叠群
for read in samfile.fetch("chr1"):
    print(read.query_name)

# 无索引读取（顺序读取）
for read in samfile.fetch(until_eof=True):
    print(read.query_name)
```

**重要说明：**
- 随机访问需要索引文件 (.bai/.crai)
- 返回**重叠**该区域的序列（可能超出边界）
- 对无索引文件或顺序读取使用 `until_eof=True`
- 默认仅返回已比对的序列
- 获取未比对序列使用 `fetch("*")` 或 `until_eof=True`

### 多迭代器

在同一文件上使用多个迭代器时：

```python
samfile = pysam.AlignmentFile("example.bam", "rb", multiple_iterators=True)
iter1 = samfile.fetch("chr1", 1000, 2000)
iter2 = samfile.fetch("chr2", 5000, 6000)
```

若未设置 `multiple_iterators=True`，新的 fetch() 调用会重置文件指针并中断现有迭代器。

### count() - 统计区域内的序列数

```python
# 统计所有序列
num_reads = samfile.count("chr1", 1000, 2000)

# 带质量过滤统计
num_quality_reads = samfile.count("chr1", 1000, 2000, quality=20)
```

### count_coverage() - 每碱基覆盖度

返回四个数组 (A, C, G, T) 表示每碱基覆盖度：

```python
coverage = samfile.count_coverage("chr1", 1000, 2000)
a_counts, c_counts, g_counts, t_counts = coverage
```

## AlignedSegment 对象

每个序列表示为 `AlignedSegment` 对象，包含以下关键属性：

### 序列信息
- `query_name` - 序列名称/ID
- `query_sequence` - 序列碱基
- `query_qualities` - 碱基质量分数（ASCII编码）
- `query_length` - 序列长度

### 比对信息
- `reference_name` - 染色体/重叠群名称
- `reference_start` - 起始位置（0起始，包含）
- `reference_end` - 结束位置（0起始，不包含）
- `mapping_quality` - MAPQ 分数
- `cigarstring` - CIGAR 字符串（如 "100M"）
- `cigartuples` - CIGAR 元组列表 (操作类型, 长度)

**重要提示：** `cigartuples` 格式与 SAM 规范不同。操作类型为整数：
- 0 = M（匹配/错配）
- 1 = I（插入）
- 2 = D（缺失）
- 3 = N（跳过参考序列）
- 4 = S（软裁剪）
- 5 = H（硬裁剪）
- 6 = P（填充）
- 7 = =（序列匹配）
- 8 = X（序列错配）

### 标志位与状态
- `flag` - SAM 标志位整数
- `is_paired` - 是否为成对序列？
- `is_proper_pair` - 是否形成正确配对？
- `is_unmapped` - 是否未比对？
- `mate_is_unmapped` - 配对序列是否未比对？
- `is_reverse` - 是否在反向链？
- `mate_is_reverse` - 配对序列是否在反向链？
- `is_read1` - 是否为 read1？
- `is_read2` - 是否为 read2？
- `is_secondary` - 是否为次要比对？
- `is_qcfail` - 是否未通过质控？
- `is_duplicate` - 是否为重复序列？
- `is_supplementary` - 是否为补充比对？

### 标签与可选字段
- `get_tag(tag)` - 获取可选字段值
- `set_tag(tag, value)` - 设置可选字段
- `has_tag(tag)` - 检查标签是否存在
- `get_tags()` - 获取所有标签（元组列表）

```python
for read in samfile.fetch("chr1", 1000, 2000):
    if read.has_tag("NM"):
        edit_distance = read.get_tag("NM")
        print(f"{read.query_name}: NM={edit_distance}")
```

## 写入比对文件

### 创建头部

```python
header = {
    'HD': {'VN': '1.0'},
    'SQ': [
        {'LN': 1575, 'SN': 'chr1'},
        {'LN': 1584, 'SN': 'chr2'}
    ]
}

outfile = pysam.AlignmentFile("output.bam", "wb", header=header)
```

### 创建 AlignedSegment 对象

```python
# 创建新序列
a = pysam.AlignedSegment()
a.query_name = "read001"
a.query_sequence = "AGCTTAGCTAGCTACCTATATCTTGGTCTTGGCCG"
a.flag = 0
a.reference_id = 0  # 指向 header['SQ'] 的索引
a.reference_start = 100
a.mapping_quality = 20
a.cigar = [(0, 35)]  # 35M
a.query_qualities = pysam.qualitystring_to_array("IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII")

# 写入文件
outfile.write(a)
```

### 格式转换

```python
# BAM 转 SAM
infile = pysam.AlignmentFile("input.bam", "rb")
outfile = pysam.AlignmentFile("output.sam", "w", template=infile)
for read in infile:
    outfile.write(read)
infile.close()
outfile.close()
```

## Pileup 分析

`pileup()` 方法提供跨区域的**列式**（逐位置）分析：

```python
for pileupcolumn in samfile.pileup("chr1", 1000, 2000):
    print(f"位置 {pileupcolumn.pos}: 覆盖度 = {pileupcolumn.nsegments}")

    for pileupread in pileupcolumn.pileups:
        if not pileupread.is_del and not pileupread.is_refskip:
            # 查询位置是序列中的位置
            base = pileupread.alignment.query_sequence[pileupread.query_position]
            print(f"  {pileupread.alignment.query_name}: {base}")
```

**关键属性：**
- `pileupcolumn.pos` - 0起始参考位置
- `pileupcolumn.nsegments` - 覆盖该位置的序列数
- `pileupread.alignment` - AlignedSegment 对象
- `pileupread.query_position` - 序列中的位置（缺失时为 None）
- `pileupread.is_del` - 是否为缺失？
- `pileupread.is_refskip` - 是否为参考跳过（CIGAR 中的 N）？

**重要提示：** 保持迭代器引用有效。当迭代器过早超出作用域时会出现 "PileupProxy accessed after iterator finished" 错误。

## 坐标系统

**关键说明：** Pysam 使用 **0起始半开区间**坐标（Python 惯例）：
- `reference_start` 为 0起始（首个碱基为 0）
- `reference_end` 为不包含（不在范围内）
- 区域 1000-2000 包含碱基 1000-1999

**例外情况：** `fetch()` 和 `pileup()` 中的区域字符串遵循 samtools 惯例（1起始）：
```python
# 以下两种方式等价：
samfile.fetch("chr1", 999, 2000)  # Python 风格：0起始
samfile.fetch("chr1:1000-2000")   # samtools 风格：1起始
```

## 索引

创建 BAM 索引：
```python
pysam.index("example.bam")
```

或使用命令行接口：
```python
pysam.samtools.index("example.bam")
```

## 性能提示

1. **重复查询特定区域时使用索引访问**
2. **列式分析优先使用 `pileup()`** 而非重复 fetch 操作
3. **无索引文件顺序读取使用 `fetch(until_eof=True)`**
4. **避免使用多迭代器**（除非必要，有性能开销）
5. **简单计数使用 `count()`** 而非手动迭代计数

## 常见陷阱

1. **部分重叠：** `fetch()` 返回跨越区域边界的序列——如需精确边界需显式过滤
2. **质量分数编辑：** 修改 `query_sequence` 后无法原地编辑 `query_qualities`。需先创建副本：`quals = read.query_qualities`
3. **缺失索引：** 未使用 `until_eof=True` 的 `fetch()` 需要索引文件
4. **线程安全：** 虽然 pysam 在 I/O 期间释放 GIL，但未完全验证线程安全性
5. **迭代器作用域：** 保持 pileup 迭代器引用有效，避免 "PileupProxy accessed after iterator finished" 错误
