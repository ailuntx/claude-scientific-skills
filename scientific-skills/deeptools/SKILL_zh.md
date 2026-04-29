```markdown
---
name: deeptools
description: NGS分析工具包。支持BAM转bigWig格式转换、质量控制（相关性分析、PCA、指纹图谱）、热图/谱线图生成（TSS、峰区域），适用于ChIP-seq、RNA-seq、ATAC-seq数据可视化。
license: BSD许可证
metadata:
    skill-author: K-Dense Inc.
---

# deepTools：高通量测序数据分析工具包

## 概述

deepTools是一套全面的Python命令行工具集，专为处理和分析高通量测序数据而设计。可用于执行质量控制、数据标准化、样本比较，并为ChIP-seq、RNA-seq、ATAC-seq、MNase-seq等NGS实验生成出版级可视化结果。

**核心功能：**
- 将BAM比对文件转换为标准化覆盖度轨道（bigWig/bedGraph）
- 质量控制评估（指纹图谱、相关性分析、覆盖度检测）
- 样本比较与相关性分析
- 围绕基因组特征生成热图和谱线图
- 富集分析与峰区域可视化

## 使用场景

本技能适用于以下场景：

- **文件格式转换**："BAM转bigWig"、"生成覆盖度轨道"、"标准化ChIP-seq数据"
- **质量控制**："检查ChIP质量"、"比较重复样本"、"评估测序深度"、"QC分析"
- **可视化**："在TSS区域创建热图"、"绘制ChIP信号"、"可视化富集结果"、"生成谱线图"
- **样本比较**："处理组与对照组比较"、"样本相关性分析"、"PCA分析"
- **分析流程**："分析ChIP-seq数据"、"RNA-seq覆盖度分析"、"ATAC-seq分析"、"完整工作流"
- **处理特定文件类型**：基因组学场景中的BAM文件、bigWig文件、BED区域文件

## 快速入门

新用户可从文件验证和常用工作流开始：

### 1. 验证输入文件

执行分析前，使用验证脚本检查BAM、bigWig和BED文件：

```bash
python scripts/validate_files.py --bam sample1.bam sample2.bam --bed regions.bed
```

此操作检查文件存在性、BAM索引及格式正确性。

### 2. 生成工作流模板

针对标准分析，使用工作流生成器创建定制脚本：

```bash
# 列出可用工作流
python scripts/workflow_generator.py --list

# 生成ChIP-seq质控工作流
python scripts/workflow_generator.py chipseq_qc -o qc_workflow.sh \
    --input-bam Input.bam --chip-bams "ChIP1.bam ChIP2.bam" \
    --genome-size 2913022398

# 添加执行权限并运行
chmod +x qc_workflow.sh
./qc_workflow.sh
```

### 3. 常用操作指南

查看`assets/quick_reference.md`获取高频命令与参数参考。

## 安装方法

```bash
uv pip install deeptools
```

## 核心工作流

deepTools工作流通常遵循模式：**质控 → 标准化 → 比较/可视化**

### ChIP-seq质量控制工作流

当用户需要ChIP-seq质控或质量评估时：

1. 使用`scripts/workflow_generator.py chipseq_qc`生成工作流脚本
2. **关键质控步骤**：
   - 样本相关性分析（multiBamSummary + plotCorrelation）
   - PCA分析（plotPCA）
   - 覆盖度评估（plotCoverage）
   - 片段大小验证（bamPEFragmentSize）
   - ChIP富集强度检测（plotFingerprint）

**结果解读：**
- **相关性**：重复样本应聚类且相关性>0.9
- **指纹图谱**：强ChIP信号呈陡峭上升；平坦对角线表示富集不足
- **覆盖度**：评估测序深度是否满足分析需求

完整流程详见`references/workflows.md` → "ChIP-seq质量控制工作流"

### ChIP-seq完整分析工作流

实现从BAM到可视化的全流程分析：

1. 标准化生成覆盖度轨道（bamCoverage）
2. 创建比较轨道（bamCompare计算log2比值）
3. 在特征区域计算信号矩阵（computeMatrix）
4. 生成可视化（plotHeatmap, plotProfile）
5. 峰区域富集分析（plotEnrichment）

使用`scripts/workflow_generator.py chipseq_analysis`生成模板。

完整命令序列见`references/workflows.md` → "ChIP-seq分析工作流"

### RNA-seq覆盖度工作流

处理链特异性RNA-seq覆盖度轨道：

使用bamCoverage时添加`--filterRNAstrand`分离正负链。

**重要提示：** RNA-seq切勿使用`--extendReads`（会跨越剪接位点）。

标准化方法：固定分箱用CPM，基因水平分析用RPKM。

模板路径：`scripts/workflow_generator.py rnaseq_coverage`

详情见`references/workflows.md` → "RNA-seq覆盖度工作流"

### ATAC-seq分析工作流

ATAC-seq需进行Tn5偏移校正：

1. 使用alignmentSieve的`--ATACshift`进行读段偏移
2. 通过bamCoverage生成覆盖度
3. 分析片段大小（应出现核小体梯状模式）
4. 在峰区域可视化（若有可用峰文件）

模板：`scripts/workflow_generator.py atacseq`

完整流程见`references/workflows.md` → "ATAC-seq工作流"

## 工具分类与常用任务

### BAM/bigWig处理

**BAM转标准化覆盖度：**
```bash
bamCoverage --bam input.bam --outFileName output.bw \
    --normalizeUsing RPGC --effectiveGenomeSize 2913022398 \
    --binSize 10 --numberOfProcessors 8
```

**样本比较（log2比值）：**
```bash
bamCompare -b1 treatment.bam -b2 control.bam -o ratio.bw \
    --operation log2 --scaleFactorsMethod readCount
```

**核心工具：** bamCoverage, bamCompare, multiBamSummary, multiBigwigSummary, correctGCBias, alignmentSieve

完整参考：`references/tools_reference.md` → "BAM与bigWig处理工具"

### 质量控制

**检测ChIP富集：**
```bash
plotFingerprint -b input.bam chip.bam -o fingerprint.png \
    --extendReads 200 --ignoreDuplicates
```

**样本相关性：**
```bash
multiBamSummary bins --bamfiles *.bam -o counts.npz
plotCorrelation -in counts.npz --corMethod pearson \
    --whatToShow heatmap -o correlation.png
```

**核心工具：** plotFingerprint, plotCoverage, plotCorrelation, plotPCA, bamPEFragmentSize

完整参考：`references/tools_reference.md` → "质控工具"

### 可视化

**在TSS区域创建热图：**
```bash
# 计算矩阵
computeMatrix reference-point -S signal.bw -R genes.bed \
    -b 3000 -a 3000 --referencePoint TSS -o matrix.gz

# 生成热图
plotHeatmap -m matrix.gz -o heatmap.png \
    --colorMap RdBu --kmeans 3
```

**创建谱线图：**
```bash
plotProfile -m matrix.gz -o profile.png \
    --plotType lines --colors blue red
```

**核心工具：** computeMatrix, plotHeatmap, plotProfile, plotEnrichment

完整参考：`references/tools_reference.md` → "可视化工具"

## 标准化方法

选择正确的标准化方法对有效比较至关重要。查阅`references/normalization_methods.md`获取完整指南。

**快速选择指南：**

- **ChIP-seq覆盖度**：使用RPGC或CPM
- **ChIP-seq比较**：使用bamCompare配合log2和readCount
- **RNA-seq分箱**：使用CPM
- **RNA-seq基因**：使用RPKM（考虑基因长度）
- **ATAC-seq**：使用RPGC或CPM

**标准化方法：**
- **RPGC**：1×基因组覆盖度（需--effectiveGenomeSize）
- **CPM**：每百万映射读段计数
- **RPKM**：每千碱基每百万读段（考虑区域长度）
- **BPM**：每百万分箱
- **None**：原始计数（不推荐用于比较）

完整说明：`references/normalization_methods.md`

## 有效基因组大小

RPGC标准化需提供有效基因组大小。常用数值：

| 物种 | 基因组版本 | 大小 | 用法 |
|------|------------|------|------|
| 人 | GRCh38/hg38 | 2,913,022,398 | `--effectiveGenomeSize 2913022398` |
| 小鼠 | GRCm38/mm10 | 2,652,783,500 | `--effectiveGenomeSize 2652783500` |
| 斑马鱼 | GRCz11 | 1,368,780,147 | `--effectiveGenomeSize 1368780147` |
| 果蝇 | dm6 | 142,573,017 | `--effectiveGenomeSize 142573017` |
| 线虫 | ce10/ce11 | 100,286,401 | `--effectiveGenomeSize 100286401` |

含读长特异性数值的完整表格：`references/effective_genome_sizes.md`

## 通用参数

多数deepTools命令共享以下选项：

**性能优化：**
- `--numberOfProcessors, -p`：启用并行处理（建议使用可用核心数）
- `--region`：在特定区域测试（如`chr1:1-1000000`）

**读段过滤：**
- `--ignoreDuplicates`：移除PCR重复（多数分析推荐）
- `--minMappingQuality`：按比对质量过滤（如`--minMappingQuality 10`）
- `--minFragmentLength` / `--maxFragmentLength`：片段长度范围
- `--samFlagInclude` / `--samFlagExclude`：SAM标志过滤

**读段处理：**
- `--extendReads`：延伸至片段长度（ChIP-seq：推荐，RNA-seq：禁用）
- `--centerReads`：在片段中点居中以获得锐利信号

## 最佳实践

### 文件验证
**始终优先验证文件**：使用`scripts/validate_files.py`检查：
- 文件存在性与可读性
- BAM索引存在性（.bai文件）
- BED格式正确性
- 文件大小合理性

### 分析策略

1. **从质控开始**：执行相关性、覆盖度和指纹分析
2. **小区域测试**：使用`--region chr1:1-10000000`测试参数
3. **记录命令**：保存完整命令行确保可复现性
4. **标准化一致性**：比较样本时采用相同标准化方法
5. **验证基因组版本**：确保BAM与BED文件使用匹配的基因组版本

### ChIP-seq专项

- **始终延伸读段**：`--extendReads 200`
- **移除重复**：多数情况使用`--ignoreDuplicates`
- **优先检测富集**：详细分析前运行plotFingerprint
- **GC校正**：仅在检测到显著偏差时应用；GC校正后禁用`--ignoreDuplicates`

### RNA-seq专项

- **禁止延伸读段**（避免跨越剪接位点）
- **链特异性**：链特异性文库使用`--filterRNAstrand forward/reverse`
- **标准化**：分箱用CPM，基因水平用RPKM

### ATAC-seq专项

- **应用Tn5校正**：使用alignmentSieve的`--ATACshift`
- **片段过滤**：设置合理的片段长度范围
- **检查核小体模式**：片段大小图应呈梯状分布

### 性能优化

1. **多核并行**：`--numberOfProcessors 8`（或可用核心数）
2. **增大分箱尺寸**：加速处理并减小文件体积
3. **分染色体处理**：适用于内存受限系统
4. **预过滤BAM**：使用alignmentSieve创建可复用过滤文件
5. **优先bigWig格式**：压缩格式处理更快

## 故障排除

### 常见问题

**BAM索引缺失：**
```bash
samtools index input.bam
```

**内存不足：**
使用`--region`分染色体处理：
```bash
bamCoverage --bam input.bam -o chr1.bw --region chr1
```

**处理缓慢：**
增加`--numberOfProcessors`和/或增大`--binSize`

**bigWig文件过大：**
增大分箱尺寸：`--binSize 50`或更高

### 验证错误

运行验证脚本定位问题：
```bash
python scripts/validate_files.py --bam *.bam --bed regions.bed
```

常见错误及解决方案详见脚本输出。

## 参考文档

本技能包含完整参考文档：

### references/tools_reference.md
按类别组织的完整工具文档：
- BAM与bigWig处理工具（9项）
- 质控工具（6项）
- 可视化工具（3项）
- 辅助工具（2项）

每项工具包含：
- 功能概述
- 关键参数说明
- 使用示例
- 注意事项与最佳实践

**适用场景：** 用户咨询特定工具、参数或详细用法时。

### references/workflows.md
常用分析完整工作流示例：
- ChIP-seq质控工作流
- ChIP-seq全分析工作流
- RNA-seq覆盖度工作流
- ATAC-seq分析工作流
- 多样本比较工作流
- 峰区域分析工作流
- 故障排除与性能优化

**适用场景：** 用户需要完整分析流程或工作流示例时。

### references/normalization_methods.md
标准化方法综合指南：
- 各方法详解（RPGC、CPM、RPKM、BPM等）
- 适用场景
- 计算公式与解读
- 按实验类型的选择指南
- 常见陷阱与解决方案
- 速查表

**适用场景：** 用户咨询标准化方法、样本比较或方法选择时。

### references/effective_genome_sizes.md
有效基因组大小参考值：
- 常见物种数值（人、小鼠、果蝇、线虫、斑马鱼）
- 读长特异性数值
- 计算方法
- 命令中的使用规范
- 自定义基因组计算指南

**适用场景：** 用户需要RPGC标准化或GC校正的基因组大小时。

## 辅助脚本

### scripts/validate_files.py

验证BAM、bigWig和BED文件是否符合deepTools分析要求。检查文件存在性、索引和格式。

**用法：**
```bash
python scripts/validate_files.py --bam sample1.bam sample2.bam \
    --bed peaks.bed --bigwig signal.bw
```

**适用时机：** 开始分析前或排查错误时。

### scripts/workflow_generator.py

为常用deepTools工作流生成可定制的bash脚本模板。

**可用工作流：**
- `chipseq_qc`：ChIP-seq质控
- `chipseq_analysis`：ChIP-seq全分析
- `rnaseq_coverage`：链特异性RNA-seq覆盖度
- `atacseq`：含Tn5校正的ATAC-seq

**用法：**
```bash
# 列出工作流
python scripts/workflow_generator.py --list

# 生成工作流
python scripts/workflow_generator.py chipseq_qc -o qc.sh \
    --input-bam Input.bam --chip-bams "ChIP1.bam ChIP2.bam" \
    --genome-size 2913022398 --threads 8

# 运行生成的工作流
chmod +x qc.sh
./qc.sh
```

**适用时机：** 用户需要标准工作流或定制脚本模板时。

## 资源文件

### assets/quick_reference.md

速查卡片包含高频命令、有效基因组大小和典型工作流模式。

**适用时机：** 用户需要快速命令参考而无需详细文档时。

## 用户请求处理指南

### 新用户引导

1. 从安装验证开始
2. 使用`scripts/validate_files.py`验证输入文件
3. 根据实验类型推荐合适工作流
4. 用`scripts/workflow_generator.py`生成工作流模板
5. 指导定制与执行

### 资深用户支持

1. 提供特定工具命令
2. 指引查阅`references/tools_reference.md`相关章节
3. 建议优化方案与最佳实践
4. 提供故障排查支持

### 专项任务处理

**"BAM转bigWig"：**
- 使用bamCoverage配合适当标准化
- 根据场景推荐RPGC或CPM
- 提供对应物种有效基因组大小
- 建议相关参数（extendReads, ignoreDuplicates, binSize）

**"检查ChIP质量"：**
- 运行完整质控流程或单独使用plotFingerprint
- 解释结果判读方法
- 根据结果建议后续操作

**"创建热图"：

当用户需要详细信息时：
- **工具详情**：指向 `references/tools_reference.md` 中的特定章节
- **工作流程**：使用 `references/workflows.md` 获取完整分析流程
- **标准化方法**：查阅 `references/normalization_methods.md` 选择方法
- **基因组大小**：参考 `references/effective_genome_sizes.md`

使用 grep 模式搜索参考文档：
```bash
# 查找工具文档
grep -A 20 "^### 工具名" references/tools_reference.md

# 查找工作流程
grep -A 50 "^## 工作流程名称" references/workflows.md

# 查找标准化方法
grep -A 15 "^### 方法名称" references/normalization_methods.md
```

## 交互示例

**用户："我需要分析 ChIP-seq 数据"**

响应策略：
1. 询问可用文件（BAM 文件、峰图、基因）
2. 使用验证脚本校验文件
3. 生成 chipseq_analysis 工作流程模板
4. 根据具体文件和生物体定制
5. 在脚本运行时解释每个步骤

**用户："该用哪种标准化方法？"**

响应策略：
1. 询问实验类型（ChIP-seq、RNA-seq 等）
2. 询问比较目标（样本内或样本间）
3. 查阅 `references/normalization_methods.md` 选择指南
4. 推荐合适方法并说明理由
5. 提供带参数的命令示例

**用户："创建 TSS 附近的热图"**

响应策略：
1. 确认 bigWig 和基因 BED 文件可用
2. 在 TSS 处使用 reference-point 模式的 computeMatrix
3. 用合适可视化参数生成 plotHeatmap
4. 大型数据集建议聚类
5. 补充提供剖面图选项

## 关键提醒

- **先验证文件**：分析前务必校验输入文件
- **标准化至关重要**：根据比较类型选择合适方法
- **谨慎延伸读段**：ChIP-seq 选 YES，RNA-seq 选 NO
- **使用所有核心**：设置 `--numberOfProcessors` 为可用核心数
- **区域测试参数**：使用 `--region` 进行参数测试
- **先检查质控**：详细分析前运行质量控制
- **全程记录**：保存命令确保可复现性
- **参考文档**：通过完整参考文档获取详细指导
