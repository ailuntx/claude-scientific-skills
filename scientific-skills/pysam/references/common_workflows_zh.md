# 使用 Pysam 的常见生物信息学工作流程

## 概述

本文档提供使用 pysam 的常见生物信息学工作流程实用示例，展示如何组合不同文件类型和操作。

## 质量控制工作流程

### 计算 BAM 文件统计信息

```python
import pysam

def calculate_bam_stats(bam_file):
    """计算 BAM 文件的基础统计信息"""
    samfile = pysam.AlignmentFile(bam_file, "rb")

    stats = {
        "total_reads": 0,
        "mapped_reads": 0,
        "unmapped_reads": 0,
        "paired_reads": 0,
        "proper_pairs": 0,
        "duplicates": 0,
        "total_bases": 0,
        "mapped_bases": 0
    }

    for read in samfile.fetch(until_eof=True):
        stats["total_reads"] += 1

        if read.is_unmapped:
            stats["unmapped_reads"] += 1
        else:
            stats["mapped_reads"] += 1
            stats["mapped_bases"] += read.query_alignment_length

        if read.is_paired:
            stats["paired_reads"] += 1
            if read.is_proper_pair:
                stats["proper_pairs"] += 1

        if read.is_duplicate:
            stats["duplicates"] += 1

        stats["total_bases"] += read.query_length

    samfile.close()

    # 计算衍生统计量
    stats["mapping_rate"] = stats["mapped_reads"] / stats["total_reads"] if stats["total_reads"] > 0 else 0
    stats["duplication_rate"] = stats["duplicates"] / stats["total_reads"] if stats["total_reads"] > 0 else 0

    return stats
```

### 检查参考序列一致性

```python
def check_bam_reference_consistency(bam_file, fasta_file):
    """验证 BAM 读数是否匹配参考基因组"""
    samfile = pysam.AlignmentFile(bam_file, "rb")
    fasta = pysam.FastaFile(fasta_file)

    mismatches = 0
    total_checked = 0

    for read in samfile.fetch():
        if read.is_unmapped:
            continue

        # 获取比对区域的参考序列
        ref_seq = fasta.fetch(
            read.reference_name,
            read.reference_start,
            read.reference_end
        )

        # 获取与参考序列比对的读数序列
        aligned_pairs = read.get_aligned_pairs(with_seq=True)

        for query_pos, ref_pos, ref_base in aligned_pairs:
            if query_pos is not None and ref_pos is not None and ref_base is not None:
                read_base = read.query_sequence[query_pos]
                if read_base.upper() != ref_base.upper():
                    mismatches += 1
                total_checked += 1

        if total_checked >= 10000:  # 采样前1万个位置
            break

    samfile.close()
    fasta.close()

    error_rate = mismatches / total_checked if total_checked > 0 else 0
    return {
        "positions_checked": total_checked,
        "mismatches": mismatches,
        "error_rate": error_rate
    }
```

## 覆盖度分析

### 计算单碱基覆盖度

```python
def calculate_coverage(bam_file, chrom, start, end):
    """计算区域内每个位置的覆盖度"""
    samfile = pysam.AlignmentFile(bam_file, "rb")

    # 初始化覆盖度数组
    length = end - start
    coverage = [0] * length

    # 统计每个位置的覆盖度
    for pileupcolumn in samfile.pileup(chrom, start, end):
        if start <= pileupcolumn.pos < end:
            coverage[pileupcolumn.pos - start] = pileupcolumn.nsegments

    samfile.close()

    return coverage
```

### 识别低覆盖区域

```python
def find_low_coverage_regions(bam_file, chrom, start, end, min_coverage=10):
    """查找覆盖度低于阈值的区域"""
    samfile = pysam.AlignmentFile(bam_file, "rb")

    low_coverage_regions = []
    in_low_region = False
    region_start = None

    for pileupcolumn in samfile.pileup(chrom, start, end):
        pos = pileupcolumn.pos
        if pos < start or pos >= end:
            continue

        coverage = pileupcolumn.nsegments

        if coverage < min_coverage:
            if not in_low_region:
                region_start = pos
                in_low_region = True
        else:
            if in_low_region:
                low_coverage_regions.append((region_start, pos))
                in_low_region = False

    # 处理未闭合的低覆盖区域
    if in_low_region:
        low_coverage_regions.append((region_start, end))

    samfile.close()

    return low_coverage_regions
```

### 计算覆盖度统计信息

```python
def coverage_statistics(bam_file, chrom, start, end):
    """计算区域的覆盖度统计信息"""
    samfile = pysam.AlignmentFile(bam_file, "rb")

    coverages = []

    for pileupcolumn in samfile.pileup(chrom, start, end):
        if start <= pileupcolumn.pos < end:
            coverages.append(pileupcolumn.nsegments)

    samfile.close()

    if not coverages:
        return None

    coverages.sort()
    n = len(coverages)

    return {
        "mean": sum(coverages) / n,
        "median": coverages[n // 2],
        "min": coverages[0],
        "max": coverages[-1],
        "positions": n
    }
```

## 变异分析

### 提取区域内的变异

```python
def extract_variants_in_genes(vcf_file, bed_file):
    """提取与基因区域重叠的变异"""
    vcf = pysam.VariantFile(vcf_file)
    bed = pysam.TabixFile(bed_file)

    variants_by_gene = {}

    for gene in bed.fetch(parser=pysam.asBed()):
        gene_name = gene.name
        variants_by_gene[gene_name] = []

        # 查找基因区域内的变异
        for variant in vcf.fetch(gene.contig, gene.start, gene.end):
            variant_info = {
                "chrom": variant.chrom,
                "pos": variant.pos,
                "ref": variant.ref,
                "alt": variant.alts,
                "qual": variant.qual
            }
            variants_by_gene[gene_name].append(variant_info)

    vcf.close()
    bed.close()

    return variants_by_gene
```

### 为变异添加覆盖度注释

```python
def annotate_variants_with_coverage(vcf_file, bam_file, output_file):
    """为变异添加覆盖度信息"""
    vcf = pysam.VariantFile(vcf_file)
    samfile = pysam.AlignmentFile(bam_file, "rb")

    # 若不存在则添加DP字段到头部
    if "DP" not in vcf.header.info:
        vcf.header.info.add("DP", "1", "Integer", "来自BAM的总深度")

    outvcf = pysam.VariantFile(output_file, "w", header=vcf.header)

    for variant in vcf:
        # 获取变异位置的覆盖度
        coverage = samfile.count(
            variant.chrom,
            variant.pos - 1,  # 转换为0-based坐标
            variant.pos
        )

        # 添加到INFO字段
        variant.info["DP"] = coverage

        outvcf.write(variant)

    vcf.close()
    samfile.close()
    outvcf.close()
```

### 按读数支持过滤变异

```python
def filter_variants_by_support(vcf_file, bam_file, output_file, min_alt_reads=3):
    """按最低等位基因支持数过滤变异"""
    vcf = pysam.VariantFile(vcf_file)
    samfile = pysam.AlignmentFile(bam_file, "rb")
    outvcf = pysam.VariantFile(output_file, "w", header=vcf.header)

    for variant in vcf:
        # 统计支持每个等位基因的读数
        allele_counts = {variant.ref: 0}
        for alt in variant.alts:
            allele_counts[alt] = 0

        # 在变异位置进行堆叠分析
        for pileupcolumn in samfile.pileup(
            variant.chrom,
            variant.pos - 1,
            variant.pos
        ):
            if pileupcolumn.pos == variant.pos - 1:  # 0-based坐标
                for pileupread in pileupcolumn.pileups:
                    if not pileupread.is_del and not pileupread.is_refskip:
                        base = pileupread.alignment.query_sequence[
                            pileupread.query_position
                        ]
                        if base in allele_counts:
                            allele_counts[base] += 1

        # 检查是否有替代等位基因达到支持阈值
        has_support = any(
            allele_counts.get(alt, 0) >= min_alt_reads
            for alt in variant.alts
        )

        if has_support:
            outvcf.write(variant)

    vcf.close()
    samfile.close()
    outvcf.close()
```

## 序列提取

### 提取变异位点周边序列

```python
def extract_variant_contexts(vcf_file, fasta_file, output_file, window=50):
    """提取变异位点周边的参考序列"""
    vcf = pysam.VariantFile(vcf_file)
    fasta = pysam.FastaFile(fasta_file)

    with open(output_file, 'w') as out:
        for variant in vcf:
            # 获取序列上下文
            start = max(0, variant.pos - window - 1)  # 转换为0-based坐标
            end = variant.pos + window

            context = fasta.fetch(variant.chrom, start, end)

            # 标记变异位置
            var_pos_in_context = variant.pos - 1 - start

            out.write(f">{variant.chrom}:{variant.pos} {variant.ref}>{variant.alts}\n")
            out.write(context[:var_pos_in_context].lower())
            out.write(context[var_pos_in_context:var_pos_in_context+len(variant.ref)].upper())
            out.write(context[var_pos_in_context+len(variant.ref):].lower())
            out.write("\n")

    vcf.close()
    fasta.close()
```

### 提取基因序列

```python
def extract_gene_sequences(bed_file, fasta_file, output_fasta):
    """从BED文件中提取基因序列"""
    bed = pysam.TabixFile(bed_file)
    fasta = pysam.FastaFile(fasta_file)

    with open(output_fasta, 'w') as out:
        for gene in bed.fetch(parser=pysam.asBed()):
            sequence = fasta.fetch(gene.contig, gene.start, gene.end)

            # 处理链方向
            if hasattr(gene, 'strand') and gene.strand == '-':
                # 反向互补
                complement = str.maketrans("ATGCatgcNn", "TACGtacgNn")
                sequence = sequence.translate(complement)[::-1]

            out.write(f">{gene.name} {gene.contig}:{gene.start}-{gene.end}\n")

            # 按每行60字符写入序列
            for i in range(0, len(sequence), 60):
                out.write(sequence[i:i+60] + "\n")

    bed.close()
    fasta.close()
```

## 读数过滤与子集提取

### 按区域和质量过滤 BAM

```python
def filter_bam(input_bam, output_bam, chrom, start, end, min_mapq=20):
    """按区域和比对质量过滤BAM文件"""
    infile = pysam.AlignmentFile(input_bam, "rb")
    outfile = pysam.AlignmentFile(output_bam, "wb", template=infile)

    for read in infile.fetch(chrom, start, end):
        if read.mapping_quality >= min_mapq and not read.is_duplicate:
            outfile.write(read)

    infile.close()
    outfile.close()

    # 创建索引
    pysam.index(output_bam)
```

### 提取特定变异位点的读数

```python
def extract_reads_at_variants(bam_file, vcf_file, output_bam, window=100):
    """提取重叠变异位置的读数"""
    samfile = pysam.AlignmentFile(bam_file, "rb")
    vcf = pysam.VariantFile(vcf_file)
    outfile = pysam.AlignmentFile(output_bam, "wb", template=samfile)

    # 收集所有读数（使用集合避免重复）
    reads_to_keep = set()

    for variant in vcf:
        start = max(0, variant.pos - window - 1)
        end = variant.pos + window

        for read in samfile.fetch(variant.chrom, start, end):
            reads_to_keep.add(read.query_name)

    # 写入所有读数
    samfile.close()
    samfile = pysam.AlignmentFile(bam_file, "rb")

    for read in samfile.fetch(until_eof=True):
        if read.query_name in reads_to_keep:
            outfile.write(read)

    samfile.close()
    vcf.close()
    outfile.close()

    pysam.index(output_bam)
```

## 整合工作流程

### 从 BAM 创建覆盖度轨迹

```python
def create_coverage_bedgraph(bam_file, output_file, chrom=None):
    """从BAM创建bedGraph格式的覆盖度轨迹"""
    samfile = pysam.AlignmentFile(bam_file, "rb")

    chroms = [chrom] if chrom else samfile.references

    with open(output_file, 'w') as out:
        out.write("track type=bedGraph name=\"Coverage\"\n")

        for chrom in chroms:
            current_cov = None
            region_start = None

            for pileupcolumn in samfile.pileup(chrom):
                pos = pileupcolumn.pos
                cov = pileupcolumn.nsegments

                if cov != current_cov:
                    # 写入前一个区域
                    if current_cov is not None:
                        out.write(f"{chrom}\t{region_start}\t{pos}\t{current_cov}\n")

                    # 开始新区域
                    current_cov = cov
                    region_start = pos

            # 写入最终区域
            if current_cov is not None:
                out.write(f"{chrom}\t{region_start}\t{pos+1}\t{current_cov}\n")

    samfile.close()
```

### 合并多个 VCF 文件

```python
def merge_vcf_samples(vcf_files, output_file):
    """合并多个单样本VCF文件"""
    # 打开所有输入文件
    vcf_readers = [pysam.VariantFile(f) for f in vcf_files]

    # 创建合并后的头部
    merged_header = vcf_readers[0].header.copy()
    for vcf in vcf_readers[1:]:
        for sample in vcf.header.samples:
            merged_header.samples.add(sample)

    outvcf = pysam.VariantFile(output_file, "w", header=merged_header)

    # 获取所有变异位置
    all_variants = {}
    for vcf in vcf_readers:
        for variant in vcf:
            key = (variant.chrom, variant.pos, variant.ref, variant.alts)
            if key not in all_variants:
                all_variants[key] = []
            all_variants[key].append(variant)

    # 写入合并后的变异
    for key, variants in sorted(all_variants.items()):
        # 从首个变异创建合并记录
        merged = outvcf.new_record(
            contig=variants[0].chrom,
            start=variants[0].start,
            stop=variants[0].stop,
            alleles=variants[0].alleles
        )

        # 添加所有样本的基因型
        for variant in variants:
            for sample in variant.samples:
                merged.samples[sample].update(variant.samples[sample])

        outvcf.write(merged)

    # 关闭所有文件
    for vcf in vcf_readers:
        vcf.close()
    outvcf.close()
```

## 工作流程性能优化建议

1. **使用索引文件**进行所有随机访问操作
2. **并行处理区域**当分析多个独立区域时
3. **尽可能流式处理数据** - 避免将整个文件加载到内存
4. **显式关闭文件**以释放资源
5. **使用 `until_eof=True`** 进行整个文件的顺序处理
6. **批量操作**同一文件以最小化I/O
7. **注意内存使用**高覆盖区域的堆叠操作
8. **需要计数时优先使用 count() 而非 pileup()**

## 常见整合模式

1. **BAM + 参考序列**：验证比对，提取比对序列
2. **BAM + VCF**：验证变异，计算等位基因频率
3. **VCF + BED**：用基因/区域

6. **多份BAM文件**：比较不同样本间的覆盖度或变异情况  
7. **BAM + FASTQ**：提取未比对的读段进行重新比对
