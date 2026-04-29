# deepTools 标准化方法

本文档详细介绍了 deepTools 中可用的各种标准化方法及其适用场景。

## 为何需要标准化？

标准化对于以下情况至关重要：
1. **比较测序深度不同的样本**
2. **校正文库大小差异**
3. **使覆盖度数值在跨实验中可解释**
4. **实现不同条件间的公平比较**

若不进行标准化，即使生物信号完全相同，一个具有1亿条读段的样本也会显得比5000万条读段的样本覆盖度更高。

---

## 可用标准化方法

### 1. RPKM（每千碱基每百万映射读段数）

**公式：** `(读段数量) / (区域长度（千碱基）× 总映射读段数（百万）)`

**适用场景：**
- 比较同一样本内的不同基因组区域
- 同时校正测序深度和区域长度
- RNA-seq基因表达分析

**支持工具：** `bamCoverage`

**示例：**
```bash
bamCoverage --bam input.bam --outFileName output.bw \
    --normalizeUsing RPKM
```

**解读：** RPKM值为10表示每千碱基特征区域每百万映射读段对应10条读段。

**优点：**
- 同时考虑区域长度和文库大小
- 在基因组学中广泛使用

**缺点：**
- 当样本间总RNA含量不同时不适用
- 比较组成差异大的样本时可能产生误导

---

### 2. CPM（每百万映射读段计数）

**公式：** `(读段数量) / (总映射读段数（百万）)`

**别称：** RPM（每百万读段数）

**适用场景：**
- 比较不同样本的相同基因组区域
- 区域长度恒定或不相关时
- ChIP-seq、ATAC-seq、DNase-seq分析

**支持工具：** `bamCoverage`, `bamCompare`

**示例：**
```bash
bamCoverage --bam input.bam --outFileName output.bw \
    --normalizeUsing CPM
```

**解读：** CPM值为5表示该区间每百万映射读段对应5条读段。

**优点：**
- 简单直观
- 适用于比较不同测序深度的样本
- 适合固定大小区间的比较

**缺点：**
- 未考虑区域长度
- 受高丰度区域影响（如RNA-seq中的rRNA）

---

### 3. BPM（每百万区间读段数）

**公式：** `(区间内读段数) / (所有区间读段总和（百万）)`

**与CPM关键区别：** 仅考虑分析区间内的读段，而非全部映射读段。

**适用场景：**
- 类似CPM，但需排除分析区域外的读段
- 比较特定基因组区域时忽略背景信号

**支持工具：** `bamCoverage`, `bamCompare`

**示例：**
```bash
bamCoverage --bam input.bam --outFileName output.bw \
    --normalizeUsing BPM
```

**解读：** BPM仅基于分箱区域内的读段进行标准化。

**优点：**
- 聚焦于分析区域的标准化
- 较少受未分析区域读段影响

**缺点：**
- 使用较少，与公开数据比较时可能受限

---

### 4. RPGC（基因组含量每读段数）

**公式：** `(读段数 × 缩放因子) / 有效基因组大小`

**缩放因子：** 计算实现1×基因组覆盖度（每个碱基1条读段）

**适用场景：**
- 需要跨样本可比的覆盖度数值
- 需要可解释的绝对覆盖度值
- 比较总读段数差异大的样本
- 含spike-in校正的ChIP-seq分析

**支持工具：** `bamCoverage`, `bamCompare`

**必选参数：** `--effectiveGenomeSize`

**示例：**
```bash
bamCoverage --bam input.bam --outFileName output.bw \
    --normalizeUsing RPGC \
    --effectiveGenomeSize 2913022398
```

**解读：** 信号值近似覆盖深度（如值2≈2×覆盖度）。

**优点：**
- 生成1×标准化覆盖度
- 可解释为基因组覆盖度
- 适合比较不同测序深度的样本

**缺点：**
- 需已知有效基因组大小
- 假设覆盖均匀（不适用于含峰值的ChIP-seq）

---

### 5. None（无标准化）

**公式：** 原始读段计数

**适用场景：**
- 初步分析
- 样本文库大小完全相同时（罕见）
- 下游工具将执行标准化时
- 调试或质量控制

**支持工具：** 所有工具（通常为默认）

**示例：**
```bash
bamCoverage --bam input.bam --outFileName output.bw \
    --normalizeUsing None
```

**解读：** 每个区间的原始读段计数。

**优点：**
- 无预设假设
- 便于查看原始数据
- 计算速度最快

**缺点：**
- 无法公平比较不同测序深度的样本
- 不适用于发表级图表

---

### 6. SES（选择性富集统计）

**方法：** 信号提取缩放——更先进的ChIP与对照比较方法

**适用场景：**
- 使用bamCompare进行ChIP-seq分析
- 需要复杂背景校正
- 替代简单读段计数缩放

**支持工具：** 仅限`bamCompare`

**示例：**
```bash
bamCompare -b1 chip.bam -b2 input.bam -o output.bw \
    --scaleFactorsMethod SES
```

**注意：** SES专为ChIP-seq数据设计，在噪声数据中表现优于简单读段计数缩放。

---

### 7. readCount（读段计数缩放）

**方法：** 按样本间总读段数比例缩放

**适用场景：**
- `bamCompare`的默认方法
- 在比较中补偿测序深度差异
- 确信总读段数反映文库大小时

**支持工具：** `bamCompare`

**示例：**
```bash
bamCompare -b1 treatment.bam -b2 control.bam -o output.bw \
    --scaleFactorsMethod readCount
```

**原理：** 若样本1有1亿读段，样本2有5000万读段，则样本2在比较前缩放2倍。

---

## 标准化方法选择指南

### ChIP-seq覆盖度轨道

**推荐：** RPGC或CPM

```bash
bamCoverage --bam chip.bam --outFileName chip.bw \
    --normalizeUsing RPGC \
    --effectiveGenomeSize 2913022398 \
    --extendReads 200 \
    --ignoreDuplicates
```

**依据：** 校正测序深度差异；RPGC提供可解释的覆盖度值。

---

### ChIP-seq比较（处理组 vs 对照组）

**推荐：** 采用readCount或SES缩放的log2比值

```bash
bamCompare -b1 chip.bam -b2 input.bam -o ratio.bw \
    --operation log2 \
    --scaleFactorsMethod readCount \
    --extendReads 200 \
    --ignoreDuplicates
```

**依据：** log2比值显示富集（正值）和缺失（负值）；readCount校正深度差异。

---

### RNA-seq覆盖度轨道

**推荐：** CPM或RPKM

```bash
# 链特异性正向链
bamCoverage --bam rnaseq.bam --outFileName forward.bw \
    --normalizeUsing CPM \
    --filterRNAstrand forward

# 基因水平：RPKM校正基因长度
bamCoverage --bam rnaseq.bam --outFileName output.bw \
    --normalizeUsing RPKM
```

**依据：** CPM用于固定宽度区间比较；RPKM用于基因（校正长度）。

---

### ATAC-seq分析

**推荐：** RPGC或CPM

```bash
bamCoverage --bam atac_shifted.bam --outFileName atac.bw \
    --normalizeUsing RPGC \
    --effectiveGenomeSize 2913022398
```

**依据：** 类似ChIP-seq；需要跨样本可比的覆盖度。

---

### 样本相关性分析

**推荐：** CPM或RPGC

```bash
multiBamSummary bins \
    --bamfiles sample1.bam sample2.bam sample3.bam \
    -o readCounts.npz

plotCorrelation -in readCounts.npz \
    --corMethod pearson \
    --whatToShow heatmap \
    -o correlation.png
```

**注意：** `multiBamSummary`不显式标准化，但相关性分析对缩放稳健。对于文库大小差异极大的情况，建议先标准化BAM文件或使用CPM标准化的bigWig文件配合`multiBigwigSummary`。

---

## 高级标准化考量

### Spike-in标准化

含spike-in对照的实验（如ChIP-seq中使用果蝇染色质spike-in）：
1. 基于spike-in读段计算缩放因子
2. 使用`--scaleFactor`参数应用自定义缩放因子

```bash
# 计算spike-in因子（示例：0.8）
SCALE_FACTOR=0.8

bamCoverage --bam chip.bam --outFileName chip_spikenorm.bw \
    --scaleFactor ${SCALE_FACTOR} \
    --extendReads 200
```

---

### 手动缩放因子

可应用自定义缩放因子：

```bash
# 应用2倍缩放
bamCoverage --bam input.bam --outFileName output.bw \
    --scaleFactor 2.0
```

---

### 染色体排除

从标准化计算中排除特定染色体：

```bash
bamCoverage --bam input.bam --outFileName output.bw \
    --normalizeUsing RPGC \
    --effectiveGenomeSize 2913022398 \
    --ignoreForNormalization chrX chrY chrM
```

**适用场景：** 混合性别样本中的性染色体、线粒体DNA或覆盖度异常的染色体。

---

## 常见陷阱

### 1. 在分箱数据中使用RPKM
**问题：** RPKM校正区域长度，但所有分箱大小相同
**解决方案：** 改用CPM或RPGC

### 2. 比较未标准化样本
**问题：** 测序深度2倍的样本显示2倍信号
**解决方案：** 比较样本时务必标准化

### 3. 错误的有效基因组大小
**问题：** 在hg38数据中使用hg19基因组大小
**解决方案：** 复核基因组版本并使用正确大小

### 4. GC校正后忽略重复读段
**问题：** 可能引入偏差
**解决方案：** 在`correctGCBias`后切勿使用`--ignoreDuplicates`

### 5. 使用RPGC未指定有效基因组大小
**问题：** 命令执行失败
**解决方案：** 使用RPGC时务必指定`--effectiveGenomeSize`

---

## 不同比较场景的标准化选择

### 样本内比较（不同区域）
**采用：** RPKM（校正区域长度）

### 样本间比较（相同区域）
**采用：** CPM、RPGC或BPM（校正文库大小）

### 处理组 vs 对照组
**采用：** bamCompare配合log2比值及readCount/SES缩放

### 多样本相关性
**采用：** CPM或RPGC标准化的bigWig文件，配合multiBigwigSummary

---

## 速查表

| 方法 | 校正深度 | 校正长度 | 最佳场景 | 命令 |
|--------|-------------------|---------------------|----------|---------|
| RPKM | ✓ | ✓ | RNA-seq基因 | `--normalizeUsing RPKM` |
| CPM | ✓ | ✗ | 固定大小分箱 | `--normalizeUsing CPM` |
| BPM | ✓ | ✗ | 特定区域 | `--normalizeUsing BPM` |
| RPGC | ✓ | ✗ | 可解释覆盖度 | `--normalizeUsing RPGC --effectiveGenomeSize X` |
| None | ✗ | ✗ | 原始数据 | `--normalizeUsing None` |
| SES | ✓ | ✗ | ChIP比较 | `bamCompare --scaleFactorsMethod SES` |
| readCount | ✓ | ✗ | ChIP比较 | `bamCompare --scaleFactorsMethod readCount` |

---

## 延伸阅读

关于标准化理论和最佳实践的更多细节：
- deepTools文档：https://deeptools.readthedocs.io/
- ENCODE ChIP-seq分析指南
- RNA-seq标准化相关论文（DESeq2、TMM方法）
