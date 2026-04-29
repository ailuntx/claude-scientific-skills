# deepTools 完整工具参考手册

本文档按类别提供所有 deepTools 命令行工具的全面参考指南。

## BAM 与 bigWig 文件处理工具

### multiBamSummary

计算多个 BAM 文件中基因组区域的读段覆盖度，输出压缩的 numpy 数组用于下游相关性和 PCA 分析。

**工作模式：**
- **bins**：使用连续等宽窗口进行全基因组分析（默认 10kb）
- **BED-file**：将分析限制在用户指定的基因组区域

**关键参数：**
- `--bamfiles, -b`：已索引的 BAM 文件（空格分隔，必需）
- `--outFileName, -o`：输出覆盖度矩阵文件（必需）
- `--BED`：区域定义文件（仅限 BED-file 模式）
- `--binSize`：窗口大小（单位：碱基，默认：10,000）
- `--labels`：自定义样本标识符
- `--minMappingQuality`：读段纳入的质量阈值
- `--numberOfProcessors, -p`：并行处理核心数
- `--extendReads`：片段大小延伸
- `--ignoreDuplicates`：移除 PCR 重复
- `--outRawCounts`：导出带坐标列和每样本计数的制表符分隔文件

**输出：** 用于 plotCorrelation 和 plotPCA 的压缩 numpy 数组 (.npz)

**常用用法：**
```bash
# 全基因组比较
multiBamSummary bins --bamfiles sample1.bam sample2.bam -o results.npz

# 峰值区域比较
multiBamSummary BED-file --BED peaks.bed --bamfiles sample1.bam sample2.bam -o results.npz
```

---

### multiBigwigSummary

功能类似 multiBamSummary，但操作对象为 bigWig 文件而非 BAM 文件。用于跨样本比较覆盖度轨迹。

**工作模式：**
- **bins**：全基因组分析
- **BED-file**：区域特异性分析

**关键参数：** 与 multiBamSummary 类似，但接受 bigWig 文件

---

### bamCoverage

将 BAM 比对文件转换为 bigWig 或 bedGraph 格式的标准化覆盖度轨迹。将覆盖度计算为每个窗口的读段数量。

**关键参数：**
- `--bam, -b`：输入 BAM 文件（必需）
- `--outFileName, -o`：输出文件名（必需）
- `--outFileFormat, -of`：输出类型（bigwig 或 bedgraph）
- `--normalizeUsing`：标准化方法
  - **RPKM**：每百万映射读段中每千碱基读段数
  - **CPM**：每百万映射读段计数
  - **BPM**：每百万窗口计数
  - **RPGC**：基因组含量标准化读段数（需 --effectiveGenomeSize）
  - **None**：不标准化（默认）
- `--effectiveGenomeSize`：可映射基因组大小（RPGC 必需）
- `--binSize`：分辨率（单位：碱基对，默认：50）
- `--extendReads, -e`：将读段延伸至片段长度（推荐用于 ChIP-seq，不适用于 RNA-seq）
- `--centerReads`：在片段长度中心对齐读段以获得更锐利信号
- `--ignoreDuplicates`：仅计数唯一读段
- `--minMappingQuality`：过滤低于质量阈值的读段
- `--minFragmentLength / --maxFragmentLength`：片段长度过滤
- `--smoothLength`：窗口平均化降噪
- `--MNase`：分析 MNase-seq 数据的核小体定位
- `--Offset`：位置特异性偏移（适用于 RiboSeq、GROseq）
- `--filterRNAstrand`：分离正/负链读段
- `--ignoreForNormalization`：从标准化中排除染色体（如性染色体）
- `--numberOfProcessors, -p`：并行处理

**重要提示：**
- RNA-seq：请勿使用 --extendReads（会跨越剪接位点延伸）
- ChIP-seq：使用 --extendReads 配合较小窗口尺寸
- GC 偏好性校正后切勿使用 --ignoreDuplicates

**常用用法：**
```bash
# RPKM 标准化的基础覆盖度
bamCoverage --bam input.bam --outFileName coverage.bw --normalizeUsing RPKM

# ChIP-seq 带读段延伸
bamCoverage --bam chip.bam --outFileName chip_coverage.bw \
    --binSize 10 --extendReads 200 --ignoreDuplicates

# 链特异性 RNA-seq
bamCoverage --bam rnaseq.bam --outFileName forward.bw \
    --filterRNAstrand forward
```

---

### bamCompare

通过生成 bigWig 或 bedGraph 文件比较两个 BAM 文件，校正测序深度差异。在等宽窗口中进行处理并执行逐窗口计算。

**比较方法：**
- **log2**（默认）：样本的对数比值
- **ratio**：直接比值计算
- **subtract**：文件间差值
- **add**：样本总和
- **mean**：样本平均值
- **reciprocal_ratio**：负倒数（适用于比值 <0 的情况）
- **first/second**：输出单个文件的缩放信号

**标准化方法：**
- **readCount**（默认）：补偿测序深度
- **SES**：选择性富集统计
- **RPKM**：每千碱基每百万读段
- **CPM**：每百万计数
- **BPM**：每百万窗口
- **RPGC**：基因组含量标准化读段数（需 --effectiveGenomeSize）

**关键参数：**
- `--bamfile1, -b1`：第一个 BAM 文件（必需）
- `--bamfile2, -b2`：第二个 BAM 文件（必需）
- `--outFileName, -o`：输出文件名（必需）
- `--outFileFormat`：bigwig 或 bedgraph
- `--operation`：比较方法（见上文）
- `--scaleFactorsMethod`：标准化方法（见上文）
- `--binSize`：输出窗口宽度（默认：50bp）
- `--pseudocount`：避免除零错误（默认：1）
- `--extendReads`：将读段延伸至片段长度
- `--ignoreDuplicates`：仅计数唯一读段
- `--minMappingQuality`：质量阈值
- `--numberOfProcessors, -p`：并行处理

**常用用法：**
```bash
# 处理组 vs 对照组的对数比值
bamCompare -b1 treatment.bam -b2 control.bam -o log2ratio.bw

# 从处理组中减去对照组信号
bamCompare -b1 treatment.bam -b2 control.bam -o difference.bw \
    --operation subtract --scaleFactorsMethod readCount
```

---

### correctGCBias / computeGCBias

**computeGCBias：** 识别测序和 PCR 扩增中的 GC 含量偏好性。

**correctGCBias：** 根据 computeGCBias 检测结果校正 BAM 文件的 GC 偏好性。

**关键参数 (computeGCBias)：**
- `--bamfile, -b`：输入 BAM 文件
- `--effectiveGenomeSize`：可映射基因组大小
- `--genome, -g`：2bit 格式参考基因组
- `--fragmentLength, -l`：片段长度（单端测序）
- `--biasPlot`：输出诊断图

**关键参数 (correctGCBias)：**
- `--bamfile, -b`：输入 BAM 文件
- `--effectiveGenomeSize`：可映射基因组大小
- `--genome, -g`：2bit 格式参考基因组
- `--GCbiasFrequenciesFile`：computeGCBias 生成的频率文件
- `--correctedFile, -o`：输出校正后的 BAM

**重要提示：** GC 偏好性校正后切勿使用 --ignoreDuplicates

---

### alignmentSieve

根据多种质量指标动态过滤 BAM 文件。适用于为特定分析创建过滤后的 BAM 文件。

**关键参数：**
- `--bam, -b`：输入 BAM 文件
- `--outFile, -o`：输出 BAM 文件
- `--minMappingQuality`：最低比对质量
- `--ignoreDuplicates`：移除重复读段
- `--minFragmentLength / --maxFragmentLength`：片段长度过滤器
- `--samFlagInclude / --samFlagExclude`：SAM 标志过滤
- `--shift`：读段偏移（如用于 ATACseq Tn5 校正）
- `--ATACshift`：自动偏移 ATAC-seq 数据

---

### computeMatrix

计算每个基因组区域的得分，并为 plotHeatmap 和 plotProfile 准备矩阵。处理 bigWig 得分文件和 BED/GTF 区域文件。

**工作模式：**
- **reference-point**：信号相对于特定位置（TSS、TES 或中心）的分布
- **scale-regions**：在标准化为统一长度的区域内分析信号

**关键参数：**
- `-R`：BED/GTF 格式区域文件（必需）
- `-S`：bigWig 得分文件（必需）
- `-o`：输出矩阵文件（必需）
- `-b`：参考点上游距离
- `-a`：参考点下游距离
- `-m`：区域主体长度（仅限 scale-regions 模式）
- `-bs, --binSize`：得分平均化的窗口尺寸
- `--skipZeros`：跳过全零值区域
- `--minThreshold / --maxThreshold`：按信号强度过滤
- `--sortRegions`：升序、降序、保持原序、不排序
- `--sortUsing`：均值、中位数、最大值、最小值、总和、区域长度
- `-p, --numberOfProcessors`：并行处理
- `--averageTypeBins`：统计方法（均值、中位数、最小值、最大值、总和、标准差）

**输出选项：**
- `--outFileNameMatrix`：导出制表符分隔数据
- `--outFileSortedRegions`：保存过滤/排序后的 BED 文件

**常用用法：**
```bash
# TSS 分析
computeMatrix reference-point -S signal.bw -R genes.bed \
    -o matrix.gz -b 2000 -a 2000 --referencePoint TSS

# 标准化基因主体分析
computeMatrix scale-regions -S signal.bw -R genes.bed \
    -o matrix.gz -b 1000 -a 1000 -m 3000
```

---

## 质量控制工具

### plotFingerprint

主要用于 ChIP-seq 实验的质量控制工具。评估抗体富集是否成功。生成累积读段覆盖度曲线以区分信号与噪音。

**关键参数：**
- `--bamfiles, -b`：已索引的 BAM 文件（必需）
- `--plotFile, -plot, -o`：输出图像文件名（必需）
- `--extendReads, -e`：将读段延伸至片段长度
- `--ignoreDuplicates`：仅计数唯一读段
- `--minMappingQuality`：比对质量过滤器
- `--centerReads`：在片段长度中心对齐读段
- `--minFragmentLength / --maxFragmentLength`：片段过滤器
- `--outRawCounts`：保存每窗口读段计数
- `--outQualityMetrics`：输出 QC 指标（Jensen-Shannon 距离）
- `--labels`：自定义样本名称
- `--numberOfProcessors, -p`：并行处理

**结果解读：**
- 理想对照组：直线对角线
- 强 ChIP 信号：曲线陡升至最高秩（读段集中在少数窗口）
- 弱富集：曲线趋近对角线的扁平形态

**常用用法：**
```bash
plotFingerprint -b input.bam chip1.bam chip2.bam \
    --labels Input ChIP1 ChIP2 -o fingerprint.png \
    --extendReads 200 --ignoreDuplicates
```

---

### plotCoverage

可视化全基因组平均读段分布。展示基因组覆盖度，帮助判断测序深度是否充足。

**关键参数：**
- `--bamfiles, -b`：待分析的 BAM 文件（必需）
- `--plotFile, -o`：输出图像文件名（必需）
- `--ignoreDuplicates`：移除 PCR 重复
- `--minMappingQuality`：质量阈值
- `--outRawCounts`：保存原始数据
- `--labels`：样本名称
- `--numberOfSamples`：采样位置数（默认：1,000,000）

---

### bamPEFragmentSize

确定双端测序数据的片段长度分布。验证文库制备预期片段大小的关键 QC 步骤。

**关键参数：**
- `--bamfiles, -b`：BAM 文件（必需）
- `--histogram, -hist`：输出直方图文件名（必需）
- `--plotTitle, -T`：图像标题
- `--maxFragmentLength`：考虑的最大长度（默认：1000）
- `--logScale`：使用对数 Y 轴
- `--outRawFragmentLengths`：保存原始片段长度

---

### plotCorrelation

分析 multiBamSummary 或 multiBigwigSummary 输出的样本相关性。展示不同样本间的相似度。

**相关性方法：**
- **Pearson**：度量数值差异；对离群值敏感；适用于正态分布数据
- **Spearman**：基于秩次；受离群值影响小；更适用于非正态分布数据

**可视化选项：**
- **heatmap**：带层次聚类（完全连接）的色阶热图
- **scatterplot**：带相关系数的成对散点图

**关键参数：**
- `--corData, -in`：multiBamSummary/multiBigwigSummary 的输入矩阵（必需）
- `--corMethod`：pearson 或 spearman（必需）
- `--whatToShow`：heatmap 或 scatterplot（必需）
- `--plotFile, -o`：输出文件名（必需）
- `--skipZeros`：排除零值区域
- `--removeOutliers`：使用中位数绝对偏差（MAD）过滤
- `--outFileCorMatrix`：导出相关矩阵
- `--labels`：自定义样本名称
- `--plotTitle`：图像标题
- `--colorMap`：配色方案（50+ 选项）
- `--plotNumbers`：在热图上显示相关值

**常用用法：**
```bash
# Pearson 相关性热图
plotCorrelation -in readCounts.npz --corMethod pearson \
    --whatToShow heatmap -o correlation_heatmap.png --plotNumbers

# Spearman 相关性散点图
plotCorrelation -in readCounts.npz --corMethod spearman \
    --whatToShow scatterplot -o correlation_scatter.png
```

---

### plotPCA

根据 multiBamSummary 或 multiBigwigSummary 输出生成主成分分析图。在降维空间中展示样本关系。

**关键参数：**
- `--corData, -in`：multiBamSummary/multiBigwigSummary 的覆盖度文件（必需）
- `--plotFile, -o`：输出图像（png, eps, pdf, svg）（必需）
- `--out

- `--outFileName, -o`：输出图像（png、eps、pdf、svg）（必需）
- `--plotType`：线条图、填充图、标准误、标准差、重叠线图、热图
- `--colors`：调色板（名称或十六进制代码）
- `--plotHeight / --plotWidth`：尺寸（厘米）
- `--yMin / --yMax`：Y轴范围
- `--averageType`：均值、中位数、最小值、最大值、标准差、总和

**聚类分析：**
- `--kmeans`：k均值聚类
- `--hclust`：层次聚类
- `--silhouette`：聚类质量指标

**标签设置：**
- `--plotTitle`：主标题
- `--regionsLabel`：区域集标识符
- `--samplesLabel`：样本名称
- `--startLabel / --endLabel`：区域边界标签（scale-regions模式）

**输出选项：**
- `--outFileNameData`：导出制表符分隔数据
- `--outFileSortedRegions`：保存过滤/排序区域为BED格式

**常用示例：**
```bash
# 线图绘制
plotProfile -m matrix.gz -o profile.png --plotType lines

# 带标准误阴影
plotProfile -m matrix.gz -o profile.png --plotType se \
    --colors blue red green
```

---

### plotEnrichment

计算并可视化基因组区域的信号富集程度。测量比对序列在区域组中的重叠百分比，适用于FRiP（峰值片段占比）评分。

**核心参数：**
- `--bamfiles, -b`：已索引的BAM文件（必需）
- `--BED`：BED/GTF格式区域文件（必需）
- `--plotFile, -o`：输出可视化文件（png/pdf/eps/svg）
- `--labels, -l`：自定义样本标识
- `--outRawCounts`：导出原始数值
- `--perSample`：按样本分组（默认）
- `--regionLabels`：自定义区域名称

**读段处理：**
- `--minFragmentLength / --maxFragmentLength`：片段长度过滤
- `--minMappingQuality`：质量阈值
- `--samFlagInclude / --samFlagExclude`：SAM标志过滤
- `--ignoreDuplicates`：去除重复读段
- `--centerReads`：中心化读段以锐化信号

**常用示例：**
```bash
plotEnrichment -b Input.bam H3K4me3.bam \
    --BED peaks_up.bed peaks_down.bed \
    --regionLabels "上调区域" "下调区域" \
    -o enrichment.png
```

---

## 其他工具

### computeMatrixOperations

高级矩阵操作工具，用于合并或截取computeMatrix生成的矩阵。支持复杂多样本、多区域分析。

**操作类型：**
- `cbind`：列向合并矩阵
- `rbind`：行向合并矩阵
- `subset`：提取特定样本或区域
- `filterStrand`：保留特定链区域
- `filterValues`：信号强度过滤
- `sort`：按多种标准排序区域
- `dataRange`：输出最小/最大值

**常用示例：**
```bash
# 合并矩阵
computeMatrixOperations cbind -m matrix1.gz matrix2.gz -o combined.gz

# 提取特定样本
computeMatrixOperations subset -m matrix.gz --samples 0 2 -o subset.gz
```

---

### estimateReadFiltering

预测不同过滤参数的影响而无需实际过滤。帮助在完整分析前优化过滤策略。

**核心参数：**
- `--bamfiles, -b`：待分析BAM文件
- `--sampleSize`：采样读段数（默认：100,000）
- `--binSize`：分析区间大小
- `--distanceBetweenBins`：采样区间间距

**待测试过滤选项：**
- `--minMappingQuality`：测试质量阈值
- `--ignoreDuplicates`：评估去重影响
- `--minFragmentLength / --maxFragmentLength`：测试片段长度过滤

---

## 通用参数

多数deepTools工具共享以下过滤和性能参数：

**读段过滤：**
- `--ignoreDuplicates`：去除PCR重复
- `--minMappingQuality`：比对置信度过滤
- `--samFlagInclude / --samFlagExclude`：SAM格式过滤
- `--minFragmentLength / --maxFragmentLength`：片段长度限制

**性能优化：**
- `--numberOfProcessors, -p`：启用并行处理
- `--region`：处理特定基因组区域（chr:起止位置）

**读段处理：**
- `--extendReads`：延伸至片段长度
- `--centerReads`：中心化至片段中点
- `--ignoreDuplicates`：仅统计唯一读段
