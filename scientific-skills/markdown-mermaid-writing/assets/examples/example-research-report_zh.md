# 基于CRISPR的基因编辑效率分析

_示例研究报告——展示Markdown-Mermaid写作技能标准。所有图表均使用嵌入Markdown的Mermaid作为源格式。_

---

## 📋 概述

本报告分析了三种细胞系模型在不同引导RNA（gRNA）条件下CRISPR-Cas9基因编辑的效率。通过T7E1检测和靶位点新一代测序（NGS）对编辑效率进行量化[^1]。

**主要发现：**

- HEK293T细胞在所有gRNA设计中均显示最高编辑效率（平均78%）
- GC含量在40-65%之间与编辑效率呈正相关（r = 0.82）
- 所有测试条件下脱靶事件发生率均低于0.1%

---

## 🔄 实验流程

CRISPR编辑实验遵循标准化五阶段流程。每个阶段设有明确的通过/终止标准。

```mermaid
flowchart TD
    accTitle: CRISPR编辑实验流程
    accDescr: 从gRNA设计到数据分析的五阶段实验流程，各阶段间设有质量检查点。

    design["🧬 阶段1<br/>gRNA设计<br/>(CRISPRscan + Cas-OFFinder)"]
    synth["⚙️ 阶段2<br/>寡核苷酸合成<br/>& 退火"]
    transfect["🔬 阶段3<br/>细胞转染<br/>(Lipofectamine 3000)"]
    screen["🧪 阶段4<br/>初筛<br/>(T7E1检测)"]
    ngs["📊 阶段5<br/>NGS验证<br/>(150 bp双端测序)"]

    qc1{GC 40-65%?}
    qc2{产量 ≥ 2 µg?}
    qc3{存活率 ≥ 85%?}
    qc4{条带可见?}

    design --> qc1
    qc1 -->|"✅ 通过"| synth
    qc1 -->|"❌ 重新设计"| design
    synth --> qc2
    qc2 -->|"✅ 通过"| transfect
    qc2 -->|"❌ 重新合成"| synth
    transfect --> qc3
    qc3 -->|"✅ 通过"| screen
    qc3 -->|"❌ 优化"| transfect
    screen --> qc4
    qc4 -->|"✅ 通过"| ngs
    qc4 -->|"❌ 重复"| screen

    classDef stage fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    classDef gate fill:#fef9c3,stroke:#ca8a04,stroke-width:2px,color:#713f12
    classDef fail fill:#fee2e2,stroke:#dc2626,stroke-width:2px,color:#7f1d1d

    class design,synth,transfect,screen,ngs stage
    class qc1,qc2,qc3,qc4 gate
```

---

## 🔬 方法

### 细胞系与培养

使用三种细胞系：HEK293T（人胚肾细胞）、K562（慢性髓系白血病细胞）和Jurkat（T淋巴细胞）。所有细胞系均在含10%胎牛血清的RPMI-1640培养基中于37°C/5% CO₂条件下培养[^2]。

### gRNA设计与效率预测

使用CRISPRscan[^3]设计靶向_EMX1_基因座的gRNA，标准如下：

| 标准 | 阈值 | 依据 |
| -------------------- | --------- | ------------------------------------- |
| GC含量 | 40–65% | 最佳Tm值和Cas9结合效率 |
| CRISPRscan评分 | ≥ 0.6 | 预测的靶向活性 |
| 脱靶位点 | ≤ 5 (≤3错配) | 降低脱靶编辑风险 |
| 同聚物序列 | 无(>4 nt) | 防止转录提前终止 |

### 转染方案

按1:1.2摩尔比（Cas9:gRNA）组装RNP复合物，通过脂质体转染递送。转染72小时后收集细胞进行基因组DNA提取。

### 分析流程

```mermaid
sequenceDiagram
    accTitle: NGS数据分析流程
    accDescr: 从原始FASTQ文件到变异检测最终生成效率报告的计算步骤序列

    participant raw as 📥 原始FASTQ
    participant qc as 🔍 FastQC
    participant trim as ✂️ Trimmomatic
    participant align as 🗺️ BWA-MEM2
    participant call as ⚙️ CRISPResso2
    participant report as 📊 报告

    raw->>qc: 碱基质量评分
    qc-->>trim: 标记低质量读段(Q<20)
    trim->>align: 清洁读段
    align->>align: 索引参考基因组(hg38)
    align->>call: BAM + 靶区域BED
    call->>call: 定量插入缺失频率
    call-->>report: 编辑效率(%)
    call-->>report: 脱靶事件
    report-->>report: 统计摘要
```

---

## 📊 结果

### 不同细胞系的编辑效率

| 细胞系 | n (重复数) | 平均效率 (%) | 标准差 (%) | 范围 (%) |
| ---------- | -------------- | ------------------- | ------ | --------- |
| **HEK293T** | 6 | **78.4** | 4.2 | 71.2–84.6 |
| K562 | 6 | 52.1 | 8.7 | 38.4–63.2 |
| Jurkat | 6 | 31.8 | 11.3 | 14.2–47.5 |

通过单因素方差分析及Tukey事后检验，HEK293T细胞编辑效率显著高于K562（p < 0.001）和Jurkat（p < 0.001）。

### GC含量对效率的影响

GC含量在40-65%区间与编辑效率呈强相关性（Pearson r = 0.82, p < 0.0001, n = 48 gRNA）。

```mermaid
xychart-beta
    accTitle: 编辑效率 vs gRNA GC含量
    accDescr: 按GC含量分组展示平均编辑效率的柱状图，证明40-65%区间性能最优

    title "按GC含量分组的平均编辑效率(HEK293T)"
    x-axis ["< 30%", "30–40%", "40–50%", "50–65%", "> 65%"]
    y-axis "编辑效率 (%)" 0 --> 100
    bar [18, 42, 76, 81, 38]
```

### 关键实验里程碑时间线

```mermaid
timeline
    accTitle: 实验时间线——CRISPR效率研究
    accDescr: 六个月内从研究设计到论文提交的时序里程碑

    section 第1月
        研究设计与gRNA文库设计 ： 3个靶基因座共48个gRNA
        细胞系鉴定 ： STR谱分析确认全部三系
    section 第2月
        gRNA合成与质控 ： 46/48 gRNA达到产量阈值
        预转染(HEK293T) ： 优化脂转染条件
    section 第3月
        全量转染系列 ： 3种细胞系×46 gRNA×6重复
        T7E1初筛 ： 全部条件通过终止检查
    section 第4月
        NGS文库制备 ： 处理276个样本
        测序运行(NovaSeq) ： 150 bp双端测序，平均50k读段/样本
    section 第5月
        生物信息分析 ： CRISPResso2流程
        统计分析 ： 方差分析、相关性、回归
    section 第6月
        论文撰写 ： 本报告
```

---

## 🔍 讨论

### HEK293T优于悬浮细胞系的原因

HEK293T相对于K562和Jurkat的编辑效率优势可能源于三个因素[^4]：

1. **贴壁形态**——确保更均匀的脂转染接触
2. **高转染许可性**——HEK293T表达SV40大T抗原，可能促进核内转运
3. **细胞周期分布**——S/G2期比例更高，利于同源定向修复(HDR)

<details>
<summary><strong>🔧 技术细节——脱靶分析</strong></summary>

通过GUIDE-seq评估5个最高活性gRNA的脱靶编辑。未检测到超过0.1%编辑频率的脱靶位点。Cas-OFFinder标记的3个潜在位点（≤2错配）分别显示0.00%、0.02%和0.04%的插入缺失频率——均低于0.05%的检测噪声阈值。

完整GUIDE-seq数据见补充资料包（GEO编号待定）。

</details>

---

### 与已发表基准的比较

_比较三种CRISPR递送方法在五个性能维度的雷达图。注：雷达图不支持`accTitle`/`accDescr`——描述见上文。_

```mermaid
radar-beta
title 与已发表方法的性能对比
axis eff["效率"], spec["特异性"], del["递送便捷性"], cost["成本"], viab["细胞存活率"]
curve this_study["本研究(RNP+脂转)"]{78, 95, 80, 85, 90}
curve plasmid["质粒Cas9(文献)"]{55, 70, 90, 95, 75}
curve electroporation["电穿孔RNP(文献)"]{88, 96, 50, 60, 65}
max 100
graticule polygon
ticks 5
showLegend true
```

---

## 🎯 结论

1. RNP-脂转染在HEK293T中实现>75%的CRISPR编辑效率——与电穿孔法相当且无细胞活性损失
2. gRNA的GC含量是本数据集中最强的编辑效率预测因子（r = 0.82）
3. 本方案不能直接应用于悬浮细胞系；K562和Jurkat需采用电穿孔或病毒递送才能获得相当效率

---

## 🔗 参考文献

[^1]: Ran, F.A. et al. (2013). "Genome engineering using the CRISPR-Cas9 system." _Nature Protocols_, 8(11), 2281–2308. https://doi.org/10.1038/nprot.2013.143

[^2]: ATCC. (2024). "Cell Line Authentication and Quality Control." https://www.atcc.org/resources/technical-documents/cell-line-authentication

[^3]: Moreno-Mateos, M.A. et al. (2015). "CRISPRscan: designing highly efficient sgRNAs for CRISPR-Cas9 targeting in vivo." _Nature Methods_, 12(10), 982–988. https://doi.org/10.1038/nmeth.3543

[^4]: Molla, K.A. & Yang, Y. (2019). "CRISPR/Cas-Mediated Base Editing: Technical Considerations and Practical Applications." _Trends in Biotechnology_, 37(10), 1121–1142. https://doi.org/10.1016/j.tibtech.2019.03.008
