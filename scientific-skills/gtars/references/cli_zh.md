# 命令行界面

Gtars 提供了全面的命令行界面，可直接在终端进行基因组区间分析。

## 安装

```bash
# 安装所有功能
cargo install gtars-cli --features "uniwig overlaprs igd bbcache scoring fragsplit"

# 仅安装特定功能
cargo install gtars-cli --features "uniwig overlaprs"
```

## 全局选项

```bash
# 显示帮助
gtars --help

# 显示版本
gtars --version

# 详细输出
gtars --verbose <命令>

# 静默模式
gtars --quiet <命令>
```

## IGD 命令

构建和查询用于重叠检测的 IGD 索引：

```bash
# 构建 IGD 索引
gtars igd build --input regions.bed --output regions.igd

# 查询单个区域
gtars igd query --index regions.igd --region "chr1:1000-2000"

# 从文件查询
gtars igd query --index regions.igd --query-file queries.bed --output results.bed

# 统计重叠
gtars igd count --index regions.igd --query-file queries.bed
```

## 重叠命令

计算基因组区域集之间的重叠：

```bash
# 查找重叠区域
gtars overlaprs overlap --set-a regions_a.bed --set-b regions_b.bed --output overlaps.bed

# 统计重叠
gtars overlaprs count --set-a regions_a.bed --set-b regions_b.bed

# 按重叠过滤区域
gtars overlaprs filter --input regions.bed --filter overlapping.bed --output filtered.bed

# 区域相减
gtars overlaprs subtract --set-a regions_a.bed --set-b regions_b.bed --output difference.bed
```

## Uniwig 命令

从基因组区间生成覆盖度轨道：

```bash
# 生成覆盖度轨道
gtars uniwig generate --input fragments.bed --output coverage.wig

# 指定分辨率
gtars uniwig generate --input fragments.bed --output coverage.wig --resolution 10

# 生成 BigWig
gtars uniwig generate --input fragments.bed --output coverage.bw --format bigwig

# 链特异性覆盖
gtars uniwig generate --input fragments.bed --output forward.wig --strand +
```

## BBCache 命令

缓存和管理来自 BEDbase.org 的 BED 文件：

```bash
# 从 bedbase 缓存 BED 文件
gtars bbcache fetch --id <bedbase_id> --output cached.bed

# 列出缓存文件
gtars bbcache list

# 清除缓存
gtars bbcache clear

# 更新缓存
gtars bbcache update
```

## 评分命令

针对参考数据集对片段重叠进行评分：

```bash
# 片段评分
gtars scoring score --fragments fragments.bed --reference reference.bed --output scores.txt

# 批量评分
gtars scoring batch --fragments-dir ./fragments/ --reference reference.bed --output-dir ./scores/

# 带权重评分
gtars scoring score --fragments fragments.bed --reference reference.bed --weights weights.txt --output scores.txt
```

## FragSplit 命令

按细胞条形码或聚类拆分片段文件：

```bash
# 按条形码拆分
gtars fragsplit split --input fragments.tsv --barcodes barcodes.txt --output-dir ./split/

# 按聚类拆分
gtars fragsplit cluster-split --input fragments.tsv --clusters clusters.txt --output-dir ./clustered/

# 过滤片段
gtars fragsplit filter --input fragments.tsv --min-fragments 100 --output filtered.tsv
```

## 常用工作流

### 工作流 1：重叠分析流程

```bash
# 步骤 1：为参考构建 IGD 索引
gtars igd build --input reference_regions.bed --output reference.igd

# 步骤 2：用实验数据查询
gtars igd query --index reference.igd --query-file experimental.bed --output overlaps.bed

# 步骤 3：生成统计
gtars overlaprs count --set-a experimental.bed --set-b reference_regions.bed
```

### 工作流 2：覆盖度轨道生成

```bash
# 步骤 1：生成覆盖度
gtars uniwig generate --input fragments.bed --output coverage.wig --resolution 10

# 步骤 2：转换为 BigWig
gtars uniwig generate --input fragments.bed --output coverage.bw --format bigwig
```

### 工作流 3：片段处理

```bash
# 步骤 1：过滤片段
gtars fragsplit filter --input raw_fragments.tsv --min-fragments 100 --output filtered.tsv

# 步骤 2：按聚类拆分
gtars fragsplit cluster-split --input filtered.tsv --clusters clusters.txt --output-dir ./by_cluster/

# 步骤 3：对照参考评分
gtars scoring batch --fragments-dir ./by_cluster/ --reference reference.bed --output-dir ./scores/
```

## 输入/输出格式

### BED 格式
标准三列或扩展 BED 格式：
```
chr1    1000    2000
chr1    3000    4000
chr2    5000    6000
```

### 片段格式 (TSV)
单细胞片段的制表符分隔格式：
```
chr1    1000    2000    BARCODE1
chr1    3000    4000    BARCODE2
chr2    5000    6000    BARCODE1
```

### WIG 格式
覆盖度轨道的 Wiggle 格式：
```
fixedStep chrom=chr1 start=1000 step=10
12
15
18
```

## 性能选项

```bash
# 设置线程数
gtars --threads 8 <命令>

# 内存限制
gtars --memory-limit 4G <命令>

# 缓冲区大小
gtars --buffer-size 10000 <命令>
```

## 错误处理

```bash
# 出错时继续
gtars --continue-on-error <命令>

# 严格模式（警告即失败）
gtars --strict <命令>

# 日志输出到文件
gtars --log-file output.log <命令>
```
