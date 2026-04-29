# deepTools 常用工作流程

本文档提供常见 deepTools 分析的完整工作流程示例。

## ChIP-seq 质量控制工作流程

完成 ChIP-seq 实验的全面质量控制评估。

### 步骤 1：初始相关性评估

比较重复样本以验证实验质量：

```bash
# 生成全基因组覆盖矩阵
multiBamSummary bins \
    --bamfiles Input1.bam Input2.bam ChIP1.bam ChIP2.bam \
    --labels Input_rep1 Input_rep2 ChIP_rep1 ChIP_rep2 \
    -o readCounts.npz \
    --numberOfProcessors 8

# 创建相关性热图
plotCorrelation \
    -in readCounts.npz \
    --corMethod pearson \
    --whatToShow heatmap \
    --plotFile correlation_heatmap.png \
    --plotNumbers

# 生成 PCA 图
plotPCA \
    -in readCounts.npz \
    -o PCA_plot.png \
    -T "ChIP-seq 样本 PCA 分析"
```

**预期结果：**
- 重复样本应聚类在一起
- Input 样本应与 ChIP 样本明显区分

---

### 步骤 2：覆盖度与深度评估

```bash
# 检查测序深度和覆盖度
plotCoverage \
    --bamfiles Input1.bam ChIP1.bam ChIP2.bam \
    --labels Input ChIP_rep1 ChIP_rep2 \
    --plotFile coverage.png \
    --ignoreDuplicates \
    --numberOfProcessors 8
```

**解读：** 评估测序深度是否满足下游分析需求。

---

### 步骤 3：片段大小验证（双端测序）

```bash
# 验证预期片段大小
bamPEFragmentSize \
    --bamfiles Input1.bam ChIP1.bam ChIP2.bam \
    --histogram fragmentSizes.png \
    --plotTitle "片段大小分布"
```

**预期结果：** 片段大小应符合建库方案（ChIP-seq 通常为 200-600bp）。

---

### 步骤 4：GC 偏好性检测与校正

```bash
# 计算 GC 偏好性
computeGCBias \
    --bamfile ChIP1.bam \
    --effectiveGenomeSize 2913022398 \
    --genome genome.2bit \
    --fragmentLength 200 \
    --biasPlot GCbias.png \
    --frequenciesFile freq.txt

# 若检测到偏好性则进行校正
correctGCBias \
    --bamfile ChIP1.bam \
    --effectiveGenomeSize 2913022398 \
    --genome genome.2bit \
    --GCbiasFrequenciesFile freq.txt \
    --correctedFile ChIP1_GCcorrected.bam
```

**注意：** 仅当观察到显著偏好性时进行校正。GC 校正后的文件请勿使用 `--ignoreDuplicates` 参数。

---

### 步骤 5：ChIP 信号强度评估

```bash
# 评估 ChIP 富集质量
plotFingerprint \
    --bamfiles Input1.bam ChIP1.bam ChIP2.bam \
    --labels Input ChIP_rep1 ChIP_rep2 \
    --plotFile fingerprint.png \
    --extendReads 200 \
    --ignoreDuplicates \
    --numberOfProcessors 8 \
    --outQualityMetrics fingerprint_metrics.txt
```

**解读：**
- 强 ChIP：累积曲线陡峭上升
- 弱富集：曲线接近对角线（类似 Input）

---

## ChIP-seq 分析工作流程

从 BAM 文件到发表级可视化的完整流程。

### 步骤 1：生成标准化覆盖轨道

```bash
# Input 对照
bamCoverage \
    --bam Input.bam \
    --outFileName Input_coverage.bw \
    --normalizeUsing RPGC \
    --effectiveGenomeSize 2913022398 \
    --binSize 10 \
    --extendReads 200 \
    --ignoreDuplicates \
    --numberOfProcessors 8

# ChIP 样本
bamCoverage \
    --bam ChIP.bam \
    --outFileName ChIP_coverage.bw \
    --normalizeUsing RPGC \
    --effectiveGenomeSize 2913022398 \
    --binSize 10 \
    --extendReads 200 \
    --ignoreDuplicates \
    --numberOfProcessors 8
```

---

### 步骤 2：创建 Log2 比值轨道

```bash
# 比较 ChIP 与 Input
bamCompare \
    --bamfile1 ChIP.bam \
    --bamfile2 Input.bam \
    --outFileName ChIP_vs_Input_log2ratio.bw \
    --operation log2 \
    --scaleFactorsMethod readCount \
    --binSize 10 \
    --extendReads 200 \
    --ignoreDuplicates \
    --numberOfProcessors 8
```

**结果：** 显示富集（正值）和缺失（负值）的 Log2 比值轨道。

---

### 步骤 3：计算 TSS 周边矩阵

```bash
# 准备转录起始位点周边热图/谱图数据
computeMatrix reference-point \
    --referencePoint TSS \
    --scoreFileName ChIP_coverage.bw \
    --regionsFileName genes.bed \
    --beforeRegionStartLength 3000 \
    --afterRegionStartLength 3000 \
    --binSize 10 \
    --sortRegions descend \
    --sortUsing mean \
    --outFileName matrix_TSS.gz \
    --outFileNameMatrix matrix_TSS.tab \
    --numberOfProcessors 8
```

---

### 步骤 4：生成热图

```bash
# 创建 TSS 周边热图
plotHeatmap \
    --matrixFile matrix_TSS.gz \
    --outFileName heatmap_TSS.png \
    --colorMap RdBu \
    --whatToShow 'plot, heatmap and colorbar' \
    --zMin -3 --zMax 3 \
    --yAxisLabel "基因" \
    --xAxisLabel "距 TSS 距离 (bp)" \
    --refPointLabel "TSS" \
    --heatmapHeight 15 \
    --kmeans 3
```

---

### 步骤 5：生成谱线图

```bash
# 创建 TSS 周边元谱图
plotProfile \
    --matrixFile matrix_TSS.gz \
    --outFileName profile_TSS.png \
    --plotType lines \
    --perGroup \
    --colors blue \
    --plotTitle "TSS 周边 ChIP-seq 信号" \
    --yAxisLabel "平均信号" \
    --xAxisLabel "距 TSS 距离 (bp)" \
    --refPointLabel "TSS"
```

---

### 步骤 6：峰区域富集分析

```bash
# 计算峰区域富集度
plotEnrichment \
    --bamfiles Input.bam ChIP.bam \
    --BED peaks.bed \
    --labels Input ChIP \
    --plotFile enrichment.png \
    --outRawCounts enrichment_counts.tab \
    --extendReads 200 \
    --ignoreDuplicates
```

---

## RNA-seq 覆盖度工作流程

生成 RNA-seq 数据的链特异性覆盖轨道。

### 正向链

```bash
bamCoverage \
    --bam rnaseq.bam \
    --outFileName forward_coverage.bw \
    --filterRNAstrand forward \
    --normalizeUsing CPM \
    --binSize 1 \
    --numberOfProcessors 8
```

### 反向链

```bash
bamCoverage \
    --bam rnaseq.bam \
    --outFileName reverse_coverage.bw \
    --filterRNAstrand reverse \
    --normalizeUsing CPM \
    --binSize 1 \
    --numberOfProcessors 8
```

**重要提示：** RNA-seq 请勿使用 `--extendReads`（会跨越剪接位点延伸）。

---

## 多样本比较工作流程

比较多个 ChIP-seq 样本（如不同条件或时间点）。

### 步骤 1：生成覆盖文件

```bash
# 为每个样本执行
for sample in Control_ChIP Treated_ChIP; do
    bamCoverage \
        --bam ${sample}.bam \
        --outFileName ${sample}.bw \
        --normalizeUsing RPGC \
        --effectiveGenomeSize 2913022398 \
        --binSize 10 \
        --extendReads 200 \
        --ignoreDuplicates \
        --numberOfProcessors 8
done
```

---

### 步骤 2：计算多样本矩阵

```bash
computeMatrix scale-regions \
    --scoreFileName Control_ChIP.bw Treated_ChIP.bw \
    --regionsFileName genes.bed \
    --beforeRegionStartLength 1000 \
    --afterRegionStartLength 1000 \
    --regionBodyLength 3000 \
    --binSize 10 \
    --sortRegions descend \
    --sortUsing mean \
    --outFileName matrix_multi.gz \
    --numberOfProcessors 8
```

---

### 步骤 3：多样本热图

```bash
plotHeatmap \
    --matrixFile matrix_multi.gz \
    --outFileName heatmap_comparison.png \
    --colorMap Blues \
    --whatToShow 'plot, heatmap and colorbar' \
    --samplesLabel Control Treated \
    --yAxisLabel "基因" \
    --heatmapHeight 15 \
    --kmeans 4
```

---

### 步骤 4：多样本谱线图

```bash
plotProfile \
    --matrixFile matrix_multi.gz \
    --outFileName profile_comparison.png \
    --plotType lines \
    --perGroup \
    --colors blue red \
    --samplesLabel Control Treated \
    --plotTitle "ChIP-seq 信号比较" \
    --startLabel "TSS" \
    --endLabel "TES"
```

---

## ATAC-seq 工作流程

含 Tn5 偏移校正的 ATAC-seq 专用流程。

### 步骤 1：Tn5 校正读段偏移

```bash
alignmentSieve \
    --bam atacseq.bam \
    --outFile atacseq_shifted.bam \
    --ATACshift \
    --minFragmentLength 38 \
    --maxFragmentLength 2000 \
    --ignoreDuplicates
```

---

### 步骤 2：生成覆盖轨道

```bash
bamCoverage \
    --bam atacseq_shifted.bam \
    --outFileName atacseq_coverage.bw \
    --normalizeUsing RPGC \
    --effectiveGenomeSize 2913022398 \
    --binSize 1 \
    --numberOfProcessors 8
```

---

### 步骤 3：片段大小分析

```bash
bamPEFragmentSize \
    --bamfiles atacseq.bam \
    --histogram fragmentSizes_atac.png \
    --maxFragmentLength 1000
```

**预期模式：** 核小体梯状分布（~50bp 无核小体区，~200bp 单核小体，~400bp 双核小体）。

---

## 峰区域分析工作流程

在 ChIP-seq 峰区域进行特异性信号分析。

### 步骤 1：峰区域矩阵计算

```bash
computeMatrix reference-point \
    --referencePoint center \
    --scoreFileName ChIP_coverage.bw \
    --regionsFileName peaks.bed \
    --beforeRegionStartLength 2000 \
    --afterRegionStartLength 2000 \
    --binSize 10 \
    --outFileName matrix_peaks.gz \
    --numberOfProcessors 8
```

---

### 步骤 2：峰区域热图

```bash
plotHeatmap \
    --matrixFile matrix_peaks.gz \
    --outFileName heatmap_peaks.png \
    --colorMap YlOrRd \
    --refPointLabel "峰中心" \
    --heatmapHeight 15 \
    --sortUsing max
```

---

## 常见问题排查

### 问题：内存不足
**解决方案：** 使用 `--region` 参数分染色体处理：
```bash
bamCoverage --bam input.bam -o chr1.bw --region chr1
```

### 问题：BAM 索引缺失
**解决方案：** 运行 deepTools 前先索引 BAM 文件：
```bash
samtools index input.bam
```

### 问题：处理速度慢
**解决方案：** 增加 `--numberOfProcessors`：
```bash
# 使用 8 核替代默认值
--numberOfProcessors 8
```

### 问题：bigWig 文件过大
**解决方案：** 增大分箱尺寸：
```bash
--binSize 50  # 或更大（默认为 10-50）
```

---

## 性能优化建议

1. **使用多核处理：** 始终将 `--numberOfProcessors` 设为可用核心数
2. **分区域处理：** 测试或内存受限时使用 `--region` 参数
3. **调整分箱尺寸：** 分箱越大 = 处理越快 + 文件越小
4. **预过滤 BAM 文件：** 用 `alignmentSieve` 创建过滤后 BAM 文件并复用
5. **优先选择 bigWig：** bigWig 格式压缩率更高且处理更快

---

## 最佳实践

1. **先做质量控制：** 进行相关性、覆盖度和指纹分析后再继续
2. **记录参数：** 保存命令行确保可重复性
3. **统一标准化方法：** 比较样本时采用相同标准化方法
4. **验证参考基因组匹配：** 确保 BAM 和区域文件使用相同基因组版本
5. **检查链方向：** RNA-seq 需验证链特异性方向
6. **小区域测试：** 使用 `--region chr1:1-1000000` 测试参数
7. **保留中间文件：** 保存矩阵以便调整参数后重新生成图表
