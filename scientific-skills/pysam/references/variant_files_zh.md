# 处理变异文件（VCF/BCF）

## 概述

Pysam 提供了 `VariantFile` 类用于读写 VCF（变异调用格式）和 BCF（二进制 VCF）文件。这些文件包含遗传变异信息，包括 SNP、插入缺失和结构变异。

## 打开变异文件

```python
import pysam

# 读取 VCF
vcf = pysam.VariantFile("example.vcf")

# 读取 BCF（二进制压缩格式）
bcf = pysam.VariantFile("example.bcf")

# 读取压缩 VCF
vcf_gz = pysam.VariantFile("example.vcf.gz")

# 写入文件
outvcf = pysam.VariantFile("output.vcf", "w", header=vcf.header)
```

## VariantFile 属性

**头信息：**
- `header` - 包含元数据的完整 VCF 头
- `header.contigs` - 染色体/重叠群字典
- `header.samples` - 样本名称列表
- `header.filters` - FILTER 定义字典
- `header.info` - INFO 字段定义字典
- `header.formats` - FORMAT 字段定义字典

```python
vcf = pysam.VariantFile("example.vcf")

# 列出样本
print(f"样本: {list(vcf.header.samples)}")

# 列出染色体
for contig in vcf.header.contigs:
    print(f"{contig}: 长度={vcf.header.contigs[contig].length}")

# 列出 INFO 字段
for info in vcf.header.info:
    print(f"{info}: {vcf.header.info[info].description}")
```

## 读取变异记录

### 遍历所有变异

```python
for variant in vcf:
    print(f"{variant.chrom}:{variant.pos} {variant.ref}>{variant.alts}")
```

### 获取特定区域

需要 VCF.gz 的 tabix 索引 (.tbi) 或 BCF 的索引：

```python
# 获取区域内的变异（区域字符串使用 1-based 坐标）
for variant in vcf.fetch("chr1", 1000000, 2000000):
    print(f"{variant.chrom}:{variant.pos} {variant.id}")

# 使用区域字符串 (1-based)
for variant in vcf.fetch("chr1:1000000-2000000"):
    print(variant.pos)
```

**注意：** `fetch()` 调用使用 **1-based 坐标** 以符合 VCF 规范。

## VariantRecord 对象

每个变异表示为 `VariantRecord` 对象：

### 位置信息
- `chrom` - 染色体/重叠群名称
- `pos` - 位置 (1-based)
- `start` - 起始位置 (0-based)
- `stop` - 终止位置 (0-based, 不包含)
- `id` - 变异 ID (如 rsID)

### 等位基因信息
- `ref` - 参考等位基因
- `alts` - 替代等位基因元组
- `alleles` - 所有等位基因元组 (ref + alts)

### 质量和过滤
- `qual` - 质量分数 (QUAL 字段)
- `filter` - 过滤状态

### INFO 字段

以字典形式访问 INFO 字段：

```python
for variant in vcf:
    # 检查字段是否存在
    if "DP" in variant.info:
        depth = variant.info["DP"]
        print(f"深度: {depth}")

    # 获取所有 INFO 键
    print(f"INFO 字段: {variant.info.keys()}")

    # 访问特定字段
    if "AF" in variant.info:
        allele_freq = variant.info["AF"]
        print(f"等位基因频率: {allele_freq}")
```

### 样本基因型数据

通过 `samples` 字典访问样本数据：

```python
for variant in vcf:
    for sample_name in variant.samples:
        sample = variant.samples[sample_name]

        # 基因型 (GT 字段)
        gt = sample["GT"]
        print(f"{sample_name} 基因型: {gt}")

        # 其他 FORMAT 字段
        if "DP" in sample:
            print(f"{sample_name} 深度: {sample['DP']}")
        if "GQ" in sample:
            print(f"{sample_name} 质量: {sample['GQ']}")

        # 该基因型的等位基因
        alleles = sample.alleles
        print(f"{sample_name} 等位基因: {alleles}")

        # 相位信息
        if sample.phased:
            print(f"{sample_name} 已定相")
```

**基因型表示：**
- `(0, 0)` - 纯合参考
- `(0, 1)` - 杂合
- `(1, 1)` - 纯合替代
- `(None, None)` - 缺失基因型
- 定相：`(0|1)` vs 未定相：`(0/1)`

## 写入变异文件

### 创建头信息

```python
header = pysam.VariantHeader()

# 添加染色体
header.contigs.add("chr1", length=248956422)
header.contigs.add("chr2", length=242193529)

# 添加 INFO 字段
header.add_line('##INFO=<ID=DP,Number=1,Type=Integer,Description="总深度">')
header.add_line('##INFO=<ID=AF,Number=A,Type=Float,Description="等位基因频率">')

# 添加 FORMAT 字段
header.add_line('##FORMAT=<ID=GT,Number=1,Type=String,Description="基因型">')
header.add_line('##FORMAT=<ID=DP,Number=1,Type=Integer,Description="读取深度">')

# 添加样本
header.add_sample("sample1")
header.add_sample("sample2")

# 创建输出文件
outvcf = pysam.VariantFile("output.vcf", "w", header=header)
```

### 创建变异记录

```python
# 创建新变异记录
record = outvcf.new_record()
record.chrom = "chr1"
record.pos = 100000
record.id = "rs123456"
record.ref = "A"
record.alts = ("G",)
record.qual = 30
record.filter.add("PASS")

# 设置 INFO 字段
record.info["DP"] = 100
record.info["AF"] = (0.25,)

# 设置基因型数据
record.samples["sample1"]["GT"] = (0, 1)
record.samples["sample1"]["DP"] = 50
record.samples["sample2"]["GT"] = (0, 0)
record.samples["sample2"]["DP"] = 50

# 写入文件
outvcf.write(record)
```

## 变异过滤

### 基础过滤

```python
# 按质量过滤
for variant in vcf:
    if variant.qual >= 30:
        print(f"高质量变异: {variant.chrom}:{variant.pos}")

# 按深度过滤
for variant in vcf:
    if "DP" in variant.info and variant.info["DP"] >= 20:
        print(f"高深度变异: {variant.chrom}:{variant.pos}")

# 按等位基因频率过滤
for variant in vcf:
    if "AF" in variant.info:
        for af in variant.info["AF"]:
            if af >= 0.01:
                print(f"常见变异: {variant.chrom}:{variant.pos}")
```

### 按基因型过滤

```python
# 查找样本含替代等位基因的变异
for variant in vcf:
    sample = variant.samples["sample1"]
    gt = sample["GT"]

    # 检查是否含替代等位基因
    if gt and any(allele and allele > 0 for allele in gt):
        print(f"样本含替代等位基因: {variant.chrom}:{variant.pos}")

    # 检查是否纯合替代
    if gt == (1, 1):
        print(f"纯合替代: {variant.chrom}:{variant.pos}")
```

### 过滤字段

```python
# 检查 FILTER 状态
for variant in vcf:
    if "PASS" in variant.filter or len(variant.filter) == 0:
        print(f"通过过滤: {variant.chrom}:{variant.pos}")
    else:
        print(f"未通过: {variant.filter.keys()}")
```

## 索引 VCF 文件

为压缩 VCF 创建 tabix 索引：

```python
# 压缩并创建索引
pysam.tabix_index("example.vcf", preset="vcf", force=True)
# 生成 example.vcf.gz 和 example.vcf.gz.tbi
```

或对 BCF 使用 bcftools：

```python
pysam.bcftools.index("example.bcf")
```

## 常见工作流

### 提取特定样本的变异

```python
invcf = pysam.VariantFile("input.vcf")
samples_to_keep = ["sample1", "sample3"]

# 创建包含样本子集的新头信息
new_header = invcf.header.copy()
new_header.samples.clear()
for sample in samples_to_keep:
    new_header.samples.add(sample)

outvcf = pysam.VariantFile("output.vcf", "w", header=new_header)

for variant in invcf:
    # 创建新记录
    new_record = outvcf.new_record(
        contig=variant.chrom,
        start=variant.start,
        stop=variant.stop,
        alleles=variant.alleles,
        id=variant.id,
        qual=variant.qual,
        filter=variant.filter,
        info=variant.info
    )

    # 复制选定样本的基因型数据
    for sample in samples_to_keep:
        new_record.samples[sample].update(variant.samples[sample])

    outvcf.write(new_record)
```

### 计算等位基因频率

```python
vcf = pysam.VariantFile("example.vcf")

for variant in vcf:
    total_alleles = 0
    alt_alleles = 0

    for sample_name in variant.samples:
        gt = variant.samples[sample_name]["GT"]
        if gt and None not in gt:
            total_alleles += 2
            alt_alleles += sum(1 for allele in gt if allele > 0)

    if total_alleles > 0:
        af = alt_alleles / total_alleles
        print(f"{variant.chrom}:{variant.pos} 等位基因频率={af:.4f}")
```

### 将 VCF 转换为汇总表

```python
import csv

vcf = pysam.VariantFile("example.vcf")

with open("variants.csv", "w", newline="") as csvfile:
    writer = csv.writer(csvfile)
    writer.writerow(["染色体", "位置", "ID", "参考", "替代", "质量", "深度"])

    for variant in vcf:
        writer.writerow([
            variant.chrom,
            variant.pos,
            variant.id or ".",
            variant.ref,
            ",".join(variant.alts) if variant.alts else ".",
            variant.qual or ".",
            variant.info.get("DP", ".")
        ])
```

## 性能优化建议

1. **使用 BCF 格式**：比 VCF 压缩率更高，访问更快
2. **建立索引文件**：使用 tabix 实现高效区域查询
3. **提前过滤**：减少无关变异处理
4. **高效使用 INFO 字段**：访问前检查是否存在
5. **批量写入操作**：创建 VCF 文件时使用

## 常见陷阱

1. **坐标系统**：VCF 使用 1-based 坐标，但 VariantRecord.start 是 0-based
2. **缺失数据**：访问 INFO/FORMAT 字段前务必检查是否存在
3. **基因型元组**：基因型是元组而非列表——需处理缺失数据的 None 值
4. **等位基因索引**：基因型 (0, 1) 中，0=参考，1=第一个替代，2=第二个替代等
5. **索引要求**：基于区域的 `fetch()` 需要 VCF.gz 的 tabix 索引
6. **头信息修改**：子集化样本时需正确更新头信息并复制 FORMAT 字段
