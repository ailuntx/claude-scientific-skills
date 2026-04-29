# 重叠检测与IGD

overlaprs模块利用集成基因组数据库（IGD）数据结构，提供高效的基因组区间重叠检测功能。

## IGD索引

IGD（集成基因组数据库）是一种专为快速基因组区间查询和重叠检测设计的专用数据结构。

### 构建IGD索引

从基因组区域文件创建索引：

```python
import gtars

# 从BED文件构建IGD索引
igd = gtars.igd.build_index("regions.bed")

# 保存索引以便复用
igd.save("regions.igd")

# 加载现有索引
igd = gtars.igd.load_index("regions.igd")
```

### 查询重叠区域

高效查找重叠区域：

```python
# 查询单个区域
overlaps = igd.query("chr1", 1000, 2000)

# 查询多个区域
results = []
for chrom, start, end in query_regions:
    overlaps = igd.query(chrom, start, end)
    results.append(overlaps)

# 仅获取重叠计数
count = igd.count_overlaps("chr1", 1000, 2000)
```

## 命令行工具使用

overlaprs命令行工具提供重叠检测功能：

```bash
# 查找两个BED文件间的重叠区域
gtars overlaprs query --index regions.bed --query query_regions.bed

# 统计重叠数量
gtars overlaprs count --index regions.bed --query query_regions.bed

# 输出重叠区域
gtars overlaprs overlap --index regions.bed --query query_regions.bed --output overlaps.bed
```

### IGD命令行指令

构建和查询IGD索引：

```bash
# 构建IGD索引
gtars igd build --input regions.bed --output regions.igd

# 查询IGD索引
gtars igd query --index regions.igd --region "chr1:1000-2000"

# 从文件批量查询
gtars igd query --index regions.igd --query-file queries.bed --output results.bed
```

## Python API

### 重叠检测

计算区域集之间的重叠：

```python
import gtars

# 加载两个区域集
set_a = gtars.RegionSet.from_bed("regions_a.bed")
set_b = gtars.RegionSet.from_bed("regions_b.bed")

# 查找重叠区域
overlaps = set_a.overlap(set_b)

# 获取A中与B重叠的区域
overlapping_a = set_a.filter_overlapping(set_b)

# 获取A中不与B重叠的区域
non_overlapping_a = set_a.filter_non_overlapping(set_b)
```

### 重叠统计

计算重叠指标：

```python
# 统计重叠数量
overlap_count = set_a.count_overlaps(set_b)

# 计算重叠比例
overlap_fraction = set_a.overlap_fraction(set_b)

# 获取重叠覆盖率
coverage = set_a.overlap_coverage(set_b)
```

## 性能特征

IGD提供高效查询：
- **索引构建**：O(n log n)，其中n为区域数量
- **查询时间**：O(k + log n)，其中k为重叠数量
- **内存高效**：基因组区间的紧凑表示

## 应用场景

### 调控元件分析

识别基因组特征间的重叠：

```python
# 查找与启动子重叠的转录因子结合位点
tfbs = gtars.RegionSet.from_bed("chip_seq_peaks.bed")
promoters = gtars.RegionSet.from_bed("promoters.bed")

overlapping_tfbs = tfbs.filter_overlapping(promoters)
print(f"在启动子区域发现{len(overlapping_tfbs)}个TFBS")
```

### 变异注释

用重叠特征注释变异：

```python
# 检查哪些变异与编码区重叠
variants = gtars.RegionSet.from_bed("variants.bed")
cds = gtars.RegionSet.from_bed("coding_sequences.bed")

coding_variants = variants.filter_overlapping(cds)
```

### 染色质状态分析

比较样本间的染色质状态：

```python
# 查找染色质状态一致的区域
sample1 = gtars.RegionSet.from_bed("sample1_peaks.bed")
sample2 = gtars.RegionSet.from_bed("sample2_peaks.bed")

consistent_regions = sample1.overlap(sample2)
```
