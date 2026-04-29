# deepTools 快速参考指南

## 最常用命令

### BAM转bigWig（标准化）
```bash
bamCoverage --bam input.bam --outFileName output.bw \
    --normalizeUsing RPGC --effectiveGenomeSize 2913022398 \
    --binSize 10 --numberOfProcessors 8
```

### 比较两个BAM文件
```bash
bamCompare -b1 treatment.bam -b2 control.bam -o ratio.bw \
    --operation log2 --scaleFactorsMethod readCount
```

### 相关性热图
```bash
multiBamSummary bins --bamfiles *.bam -o counts.npz
plotCorrelation -in counts.npz --corMethod pearson \
    --whatToShow heatmap -o correlation.png
```

### TSS附近热图
```bash
computeMatrix reference-point -S signal.bw -R genes.bed \
    -b 3000 -a 3000 --referencePoint TSS -o matrix.gz

plotHeatmap -m matrix.gz -o heatmap.png
```

### ChIP富集检测
```bash
plotFingerprint -b input.bam chip.bam -o fingerprint.png \
    --extendReads 200 --ignoreDuplicates
```

## 有效基因组大小

| 物种 | 基因组版本 | 大小 |
|----------|----------|------|
| 人类 | hg38 | 2913022398 |
| 小鼠 | mm10 | 2652783500 |
| 果蝇 | dm6 | 142573017 |

## 常用标准化方法

- **RPGC**：1×基因组覆盖度（需--effectiveGenomeSize参数）
- **CPM**：每百万计数（适用于固定分箱）
- **RPKM**：每千碱基每百万读数（适用于基因）

## 典型工作流程

1. **质控**：plotFingerprint, plotCorrelation
2. **覆盖度**：使用标准化的bamCoverage
3. **比较**：通过bamCompare处理实验组vs对照组
4. **可视化**：computeMatrix → plotHeatmap/plotProfile
