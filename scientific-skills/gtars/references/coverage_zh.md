# 使用Uniwig进行覆盖度分析

uniwig模块可从测序数据生成覆盖度轨道，高效地将基因组区间转换为覆盖度图谱。

## 覆盖度轨道生成

从BED文件创建覆盖度轨道：

```python
import gtars

# 从BED文件生成覆盖度
coverage = gtars.uniwig.coverage_from_bed("fragments.bed")

# 按指定分辨率生成覆盖度
coverage = gtars.uniwig.coverage_from_bed("fragments.bed", resolution=10)

# 生成链特异性覆盖度
fwd_coverage = gtars.uniwig.coverage_from_bed("fragments.bed", strand="+")
rev_coverage = gtars.uniwig.coverage_from_bed("fragments.bed", strand="-")
```

## 命令行使用

通过命令行生成覆盖度轨道：

```bash
# 生成覆盖度轨道
gtars uniwig generate --input fragments.bed --output coverage.wig

# 指定分辨率
gtars uniwig generate --input fragments.bed --output coverage.wig --resolution 10

# 生成BigWig格式
gtars uniwig generate --input fragments.bed --output coverage.bw --format bigwig

# 链特异性覆盖度
gtars uniwig generate --input fragments.bed --output forward.wig --strand +
gtars uniwig generate --input fragments.bed --output reverse.wig --strand -
```

## 处理覆盖度数据

### 访问覆盖度值

查询特定位置的覆盖度：

```python
# 获取单点覆盖度
cov = coverage.get_coverage("chr1", 1000)

# 获取区间覆盖度
cov_array = coverage.get_coverage_range("chr1", 1000, 2000)

# 获取覆盖度统计
mean_cov = coverage.mean_coverage("chr1", 1000, 2000)
max_cov = coverage.max_coverage("chr1", 1000, 2000)
```

### 覆盖度操作

对覆盖度轨道执行运算：

```python
# 标准化覆盖度
normalized = coverage.normalize()

# 平滑覆盖度
smoothed = coverage.smooth(window_size=10)

# 合并覆盖度轨道
combined = coverage1.add(coverage2)

# 计算覆盖度差异
diff = coverage1.subtract(coverage2)
```

## 输出格式

Uniwig支持多种输出格式：

### WIG格式

标准wiggle格式：
```
fixedStep chrom=chr1 start=1000 step=1
12
15
18
22
...
```

### BigWig格式

高效存储的二进制格式：
```bash
# 生成BigWig
gtars uniwig generate --input fragments.bed --output coverage.bw --format bigwig
```

### BedGraph格式

可变覆盖度的灵活格式：
```
chr1    1000    1001    12
chr1    1001    1002    15
chr1    1002    1003    18
```

## 应用场景

### ATAC-seq分析

生成染色质可及性图谱：

```python
# 生成ATAC-seq覆盖度
atac_fragments = gtars.RegionSet.from_bed("atac_fragments.bed")
coverage = gtars.uniwig.coverage_from_bed("atac_fragments.bed", resolution=1)

# 识别可及区域
peaks = coverage.call_peaks(threshold=10)
```

### ChIP-seq峰可视化

为ChIP-seq数据创建覆盖度轨道：

```bash
# 生成可视化覆盖度
gtars uniwig generate --input chip_seq_fragments.bed \
                      --output chip_coverage.bw \
                      --format bigwig
```

### RNA-seq覆盖度

计算RNA-seq读段覆盖度：

```python
# 生成链特异性RNA-seq覆盖度
fwd = gtars.uniwig.coverage_from_bed("rnaseq.bed", strand="+")
rev = gtars.uniwig.coverage_from_bed("rnaseq.bed", strand="-")

# 导出至IGV
fwd.to_bigwig("rnaseq_fwd.bw")
rev.to_bigwig("rnaseq_rev.bw")
```

### 差异覆盖度分析

比较样本间覆盖度：

```python
# 为两个样本生成覆盖度
control = gtars.uniwig.coverage_from_bed("control.bed")
treatment = gtars.uniwig.coverage_from_bed("treatment.bed")

# 计算倍数变化
fold_change = treatment.divide(control)

# 寻找差异区域
diff_regions = fold_change.find_regions(threshold=2.0)
```

## 性能优化

- 根据数据规模选择合适分辨率
- 大型数据集推荐使用BigWig格式
- 支持多染色体并行处理
- 大文件内存高效流式处理
