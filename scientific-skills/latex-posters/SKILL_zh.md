---
name: latex-posters
description: "使用 beamerposter、tikzposter 或 baposter 创建专业的研究海报。支持会议演示、学术海报和科学交流。包括布局设计、配色方案、多栏格式、图片整合及针对视觉传达的最佳实践。"
allowed-tools: Read Write Edit Bash
---

# LaTeX 研究海报

## 概述

研究海报是科学交流在会议、研讨会和学术活动中的关键媒介。本技能提供使用 LaTeX 包创建专业、视觉吸引人的研究海报的全面指导。生成具有适当布局、排版、配色方案和视觉层次的出版级海报。

## 何时使用本技能

本技能适用于以下场景：
- 为会议、研讨会或海报展示环节创建研究海报
- 设计用于大学活动或论文答辩的学术海报
- 为公众参与准备研究的视觉摘要
- 将科学论文转换为海报格式
- 为研究小组或部门创建模板海报
- 设计符合特定会议尺寸要求（A0、A1、36×48" 等）的海报
- 构建具有复杂多栏布局的海报
- 在海报格式中整合图片、表格、公式和引用

## 人工智能驱动的视觉元素生成

**标准工作流程：在创建 LaTeX 海报之前，使用 AI 生成所有主要视觉元素。**

这是创建视觉冲击力海报的推荐方法：
1. 规划所有需要的视觉元素（标题、引言、方法、结果、结论）
2. 使用 scientific-schematics 或 Nano Banana Pro 生成每个元素
3. 将生成的图片组装到 LaTeX 模板中
4. 在视觉元素周围添加文本内容

**目标：海报面积的 60-70% 应为 AI 生成的视觉元素，30-40% 为文本。**

---

### 关键：防止内容溢出

**⚠️ 海报不得有文本或内容被边缘切断。**

**常见溢出问题：**
1. **标题/页脚文本超出页面边界**
2. **太多章节挤在可用空间内**
3. **图片放置过于靠近边缘**
4. **文本块超出列宽**

**预防规则：**

**1. 限制内容章节（A0 最多 5-6 个章节）：**
```
✅ 优秀 - 5 个章节，留出呼吸空间：
   - 标题/页眉
   - 引言/问题
   - 方法
   - 结果（1-2 个关键发现）
   - 结论

❌ 糟糕 - 8 个以上章节挤在一起：
   - 概述、引言、背景、方法，
   - 结果 1、结果 2、讨论、结论、未来工作
```

**2. 在 LaTeX 中设置安全边距：**
```latex
% tikzposter - 添加充足的边距
\documentclass[25pt, a0paper, portrait, margin=25mm]{tikzposter}

% baposter - 确保内容不触及边缘
\begin{poster}{
  columns=3,
  colspacing=2em,           % 列间距
  headerheight=0.1\textheight,  % 较小的页眉
  % 底部留出空间
}
```

**3. 图片尺寸 - 永远不要 100% 宽度：**
```latex
% 图片周围保留边距
\includegraphics[width=0.85\linewidth]{figure.png}  % 不要使用 1.0\linewidth
```

**4. 在打印前检查溢出：**
```bash
# 编译并以 100% 缩放检查 PDF
pdflatex poster.tex

# 检查：
# - 任何边缘有文本被截断
# - 内容触及页面边界
# - .log 文件中的 Overfull hbox 警告
grep -i "overfull" poster.log
```

**5. 字数限制：**
- **A0 海报**：最多 300-800 字
- **每个章节**：最多 50-100 字
- **如果有更多内容**：精简或制作 handout

---

### 关键：海报尺寸的字体要求

**⚠️ AI 生成的可视化中所有文本必须在海报上可读。**

为海报生成图形时，必须在每个提示中指定字体大小。海报图形要从 4-6 英尺外观看，因此文本必须很大。

**⚠️ 常见问题：内容溢出和密度**

AI 生成的海报图形头号问题是**内容太多**。这会导致：
- 文本溢出边界
- 小字体无法阅读
- 杂乱、令人透不过气的视觉效果
- 空白区域使用不佳

**解决方案：生成**简单**且**内容最少**的图形。**

**每个海报图形的强制提示要求：**

```
海报格式要求（严格强制）：
- 每个图形绝对最多 3-4 个元素（3 个最理想）
- 整个图形绝对最多 10 个单词
- 不要有 5 步以上的复杂工作流程（拆分成 2-3 个简单图形）
- 不要有多层嵌套图表（扁平化为单层）
- 不要有包含多个子部分的案例研究（每个案例一个关键点）
- 所有文本巨大加粗（标签 80pt+，关键数字 120pt+）
- 仅限高对比度（深色在白底上 或 白色在深底上，不要带有文本的渐变）
- 强制至少 50% 空白区域（图形的一半应为空白）
- 仅使用粗线（最小 5px），大图标（最小 200px）
- 每个图形一条信息（不是 3 条相关信息）
```

**⚠️ 生成前：检查提示并统计元素**
- 如果描述有 5 个以上项目 → 停止。拆分成多个图形
- 如果工作流程有 5 个以上阶段 → 停止。仅显示 3-4 个高级步骤
- 如果比较有 4 种以上方法 → 停止。仅显示前 3 或 我们的 vs 最佳基线

**每种图形类型的内容限制（严格）：**
| 图形类型 | 最大元素数 | 最大单词数 | 拒绝条件 | 优秀示例 |
|-----------|------------|------------|-----------|----------|
| 流程图 | **最多 3-4 个框** | **8 个词** | 5 个以上阶段、嵌套步骤 | "发现 → 验证 → 批准"（3 个词） |
| 关键发现 | **最多 3 项** | **9 个词** | 4 个以上指标、段落 | "95% 准确" "快 2 倍" "FDA 就绪"（6 个词） |
| 比较图 | **最多 3 条柱** | **6 个词** | 4 种以上方法、图例文字 | "我们的：95%" "最佳：85%"（4 个词） |
| 案例研究 | **1 个案例，3 个元素** | **6 个词** | 多个案例、子故事 | 标志 + "18 个月" + "到发现"（2 个词） |
| 时间线 | **最多 3-4 个点** | **8 个词** | 逐年细节 | "2020 开始" "2022 试验" "2024 批准"（6 个词） |

**示例 - 错误的（7 阶段工作流程 - 太复杂）：**
```bash
# ❌ 糟糕 - 这会产生像药物发现海报一样的小而不可读的文本
python scripts/generate_schematic.py "Drug discovery workflow showing: Stage 1 Target Identification, Stage 2 Molecular Synthesis, Stage 3 Virtual Screening, Stage 4 AI Lead Optimization, Stage 5 Clinical Trial Design, Stage 6 FDA Approval. Include success metrics, timelines, and validation steps for each stage." -o figures/workflow.png
# 结果：7 个以上阶段，文本小，从 6 英尺外不可读 - 海报失败
```

**示例 - 正确的（简化为 3 个关键阶段）：**
```bash
# ✅ 优秀 - 相同内容，拆分为一个简单的高级图形
python scripts/generate_schematic.py "POSTER FORMAT for A0. ULTRA-SIMPLE 3-box workflow: 'DISCOVER' → 'VALIDATE' → 'APPROVE'. Each word in GIANT bold (120pt+). Thick arrows (10px). 60% white space. NO substeps, NO details. 3 words total. Readable from 10 feet." -o figures/workflow_overview.png
# 结果：简洁、有冲击力、可读 - 如果需要，可以单独添加详细图形
```

**示例 - 错误的（复杂的多个部分案例研究）：**
```bash
# ❌ 糟糕 - 创建拥挤不可读的部分
python scripts/generate_schematic.py "Case studies: Insilico Medicine (drug candidate, discovery time, clinical trials), Recursion Pharma (platform, methodology, results), Exscientia (drug candidates, FDA status, timeline). Include company logos, metrics, and outcomes." -o figures/cases.png
# 结果：3 个案例研究，每个 4 个以上元素 = 总共 12 个以上元素，文本小
```

**示例 - 正确的（一个案例研究，一个关键指标）：**
```bash
# ✅ 优秀 - 显示一个案例和一个关键数字
python scripts/generate_schematic.py "POSTER FORMAT for A0. ONE case study card: Company logo (large), '18 MONTHS' in GIANT text (150pt), 'to discovery' below (60pt). 3 elements total: logo + number + caption. 50% white space. Readable from 10 feet." -o figures/case_single.png
# 结果：清晰、可读、有冲击力。如果需要 3 个案例，制作 3 个单独的图形。
```

**示例 - 错误的（关键发现太复杂）：**
```bash
# BAD - too many items, too much detail
python scripts/generate_schematic.py "Key findings showing 8 metrics: accuracy 95%, precision 92%, recall 94%, F1 0.93, AUC 0.97, training time 2.3 hours, inference 50ms, model size 145MB with comparison to 5 baseline methods" -o figures/findings.png
# Result: Cramped graphic with tiny numbers
```

**示例 - 正确的（关键发现简单）：**
```bash
# GOOD - only 3 key items, giant numbers
python scripts/generate_schematic.py "POSTER FORMAT for A0. KEY FINDINGS with ONLY 3 large cards. Card 1: '95%' in GIANT text (120pt) with 'ACCURACY' below (48pt). Card 2: '2X' in GIANT text with 'FASTER' below. Card 3: checkmark icon with 'VALIDATED' in large text. 50% white space. High contrast colors. NO other text or details." -o figures/findings.png
# Result: Bold, readable impact statement
```

**海报提示中的字体大小参考：**
| 元素 | 最小尺寸 | 提示关键词 |
|---------|--------------|-----------------|
| 主要数字/指标 | 72pt+ | "巨大", "非常大", "巨像", "海报尺寸" |
| 章节标题 | 60pt+ | "大号加粗", "显眼" |
| 标签/说明 | 36pt+ | "从 6 英尺可读", "清晰标签" |
| 正文 | 24pt+ | "海报可读", "大号文本" |

**始终在提示中包含：**
- "POSTER FORMAT" 或 "for A0 poster" 或 "readable from 6 feet"
- "VERY LARGE TEXT" 或 "huge bold fonts"
- 应出现的具体文本（这样它就会嵌入图像中）
- "minimal text, maximum impact"
- "high contrast" 以实现可读性
- "generous margins" 和 "no text near edges"

---

### 关键：AI 生成的图形尺寸

**⚠️ 每个 AI 生成的图形应专注于一个概念，内容最少。**

**问题**：生成包含许多元素的复杂图表会导致文本过小。

**解决方案**：生成**简单**、**元素少**且**文本大**的图形。

**示例 - 错误的（太复杂，文本会很小）：**
```bash
# BAD - too many elements in one graphic
python scripts/generate_schematic.py "Complete ML pipeline showing data collection, 
preprocessing with 5 steps, feature engineering with 8 techniques, model training 
with hyperparameter tuning, validation with cross-validation, and deployment with 
monitoring. Include all labels and descriptions." -o figures/pipeline.png
```

**示例 - 正确的（简单、专注、大文本）：**
```bash
# GOOD - split into multiple simple graphics with large text

# Graphic 1: High-level overview (3-4 elements max)
python scripts/generate_schematic.py "POSTER FORMAT for A0: Simple 4-step pipeline. 
Four large boxes: DATA → PROCESS → MODEL → RESULTS. 
GIANT labels (80pt+), thick arrows, lots of white space. 
Only 4 words total. Readable from 8 feet." -o figures/overview.png

# Graphic 2: Key result (1 metric highlighted)
python scripts/generate_schematic.py "POSTER FORMAT for A0: Single key metric display.
Giant '95%' text (150pt+) with 'ACCURACY' below (60pt+).
Checkmark icon. Minimal design, high contrast.
Readable from 10 feet." -o figures/accuracy.png
```

**AI 生成海报图形的规则：**
| 规则 | 限制 | 原因 |
|------|-------|--------|
| **每个图形的元素数** | 3-5 最多 | 元素越多 = 文本越小 |
| **每个图形的单词数** | 10-15 最多 | 文本越少 = 字体越大 |
| **流程图步骤** | 4-5 最多 | 保持标签可读 |
| **图表类别** | 3-4 最多 | 防止拥挤 |
| **嵌套层级** | 1-2 最多 | 避免复杂性 |

**将复杂内容拆分为多个简单图形：**
```
不要一个包含 12 个元素的复杂图表：
→ 创建 3 个每个 4 个元素的简单图表
→ 每个图形可以有更大的文本
→ 在海报中以清晰的视觉流排列
```

---

### 第 0 步：强制预生成审查（先做这个）

**⚠️ 在生成任何图形之前，审查你的内容计划：**

**对于每个计划中的图形，问以下问题：**
1. **元素计数**：我能否在 3-4 个或更少项目内描述？
   - ❌ 不能 → 简化或拆分为多个图形
   - ✅ 能 → 继续

2. **复杂性检查**：这是多阶段工作流程（5 步以上）还是嵌套图表？
   - ❌ 是 → 仅扁平化为 3-4 个高级步骤
   - ✅ 否 → 继续

3. **字数**：我能否在 10 个单词内描述所有文本？
   - ❌ 不能 → 减少文本，使用单字标签
   - ✅ 能 → 继续

4. **信息清晰度**：这个图形是否传达一条清晰的信息？
   - ❌ 否 → 拆分为多个专注的图形
   - ✅ 是 → 继续生成

**总是失败的常见模式（拒绝这些）：**
- "显示阶段 1 到 7..." → 拆分为高级概述（3 个阶段）+ 详细图形
- "多个案例研究..." → 每个图形一个案例
- "从 2015 到 2024 的时间线，带有年度里程碑..." → 仅显示 3-4 个关键年份
- "6 种方法的比较..." → 仅显示前 3 或我们的方法与最佳基线
- "包含所有层和连接的架构..." → 仅高级（3-4 个组件）

### 第 1 步：规划海报元素

通过预生成审查后，确定所需的视觉元素：

1. **标题块** - 风格化的标题，带有机构品牌（可选 - 可以是 LaTeX 文本）
2. **引言图形** - 概念概述（最多 3 个元素）
3. **方法图** - 高级工作流程（最多 3-4 步）
4. **结果图** - 关键发现（每个图最多 3 个指标，可能需要 2-3 个单独的图）
5. **结论图形** - 摘要视觉（最多 3 个要点）
6. **补充图标** - 简单图标、二维码、标志（最少）

### 第 2 步：生成每个元素（在预生成审查之后）

**⚠️ 关键：在继续之前，请检查第 0 步清单。**

根据元素类型使用适当的工具：

**对于示意图和图表（scientific-schematics）：**
```bash
# 创建图形目录
mkdir -p figures

# 药物发现工作流程 - 仅高级，3 个阶段
# 糟糕："阶段 1：目标识别，阶段 2：分子合成，阶段 3：虚拟筛选，阶段 4：AI 先导优化..."
# 优秀：折叠为 3 个超级阶段
python scripts/generate_schematic.py "POSTER FORMAT for A0. ULTRA-SIMPLE 3-box workflow: 'DISCOVER' (120pt bold) → 'VALIDATE' (120pt bold) → 'APPROVE' (120pt bold). Thick arrows (10px). 60% white space. ONLY these 3 words. NO substeps. Readable from 12 feet." -o figures/workflow_simple.png

# 系统架构 - 最多 3 个组件
python scripts/generate_schematic.py "POSTER FORMAT for A0. ULTRA-SIMPLE 3-component stack: 'DATA' box (120pt) → 'AI MODEL' box (120pt) → 'PREDICTION' box (120pt). Thick vertical arrows. 60% white space. 3 words only. Readable from 12 feet." -o figures/architecture.png

# 时间线 - 仅 3 个关键里程碑（不逐年）
# 糟糕："2018, 2019, 2020, 2021, 2022, 2023, 2024 with events"
# 优秀：仅 3 个突破性时刻
python scripts/generate_schematic.py "POSTER FORMAT for A0. Timeline with ONLY 3 points: '2018' + icon, '2021' + icon, '2024' + icon. GIANT years (120pt). Large icons. 60% white space. NO connecting lines or details. Readable from 12 feet." -o figures/timeline.png

# 案例研究 - 一个案例，一个关键指标
# 糟糕："3 个案例研究：Insilico（细节），Recursion（细节），Exscientia（细节）"
# 优秀：一个案例一个数字
python scripts/generate_schematic.py "POSTER FORMAT for A0. ONE case study: Large logo + '18 MONTHS' (150pt bold) + 'to discovery' (60pt). 3 elements total. 60% white space. Readable from 12 feet." -o figures/case1.png

# 如果需要 3 个案例 → 制作 3 个单独的简单图形（不要一个复杂图形）
```

**对于风格化区块和图形（Nano Banana Pro）：**
```bash
# 标题块 - 简单
python scripts/generate_schematic.py "POSTER FORMAT for A0. Title block: 'ML FOR DRUG DISCOVERY' in HUGE bold text (120pt+). Dark blue background. ONE subtle icon. NO other text. 40% white space. Readable from 15 feet." -o figures/title_block.png

# 引言视觉 - 简单，仅 3 个元素
python scripts/generate_schematic.py "POSTER FORMAT for A0. SIMPLE problem visual with ONLY 3 icons: drug icon, arrow, target icon. ONE label per icon (80pt+). 50% white space. NO detailed text. Readable from 8 feet." -o figures/intro_visual.png

# 结论/摘要 - 仅 3 项，巨大数字
python scripts/generate_schematic.py "POSTER FORMAT for A0. KEY FINDINGS with EXACTLY 3 cards only. Card 1: '95%' (150pt font) with 'ACCURACY' (60pt). Card 2: '2X' (150pt) with 'FASTER' (60pt). Card 3: checkmark icon with 'READY' (60pt). 50% white space. NO other text. Readable from 10 feet." -o figures/conclusions_graphic.png

# 背景视觉 - 简单，仅 3 个图标
python scripts/generate_schematic.py "POSTER FORMAT for A0. SIMPLE visual with ONLY 3 large icons in a row: problem icon → challenge icon → impact icon. ONE word label each (80pt+). 50% white space. NO detailed text. Readable from 8 feet." -o figures/background_visual.png
```

**对于数据可视化 - 简单，最多 3 条柱：**
```bash
# 简单图表，仅 3 条柱，巨大标签
python scripts/generate_schematic.py "POSTER FORMAT for A0. SIMPLE bar chart with ONLY 3 bars: BASELINE (70%), EXISTING (85%), OURS (95%). GIANT percentage labels ON the bars (100pt+). NO axis labels, NO legend, NO gridlines. Our bar highlighted in different color. 40% white space. Readable from 8 feet." -o figures/comparison_chart.png
```

### 第 2b 步：强制生成后审查（在组装前）

**⚠️ 关键：在添加到海报之前，审查每个生成的图形。**

**对于每个生成的图片，以 25% 缩放打开并检查：**

1. **✅ 通过标准（必须全部为真）：**
   - 在 25% 缩放时能清晰阅读所有文本
   - 元素计数：3-4 个或更少
   - 空白区域：图像 50% 以上为空
   - 足够简单，2 秒内能理解
   - 不是 5 步以上的复杂工作流程
   - 不是多个嵌套部分

2. **❌ 失败标准（如果任意为真则重新生成）：**
   - 文本在 25% 缩放时小或难以阅读 → 用 "150pt+" 字体重新生成
   - 超过 4 个元素 → 用 "ONLY 3 elements" 重新生成
   - 空白区域少于 50% → 用 "60% white space" 重新生成
   - 复杂多阶段工作流程 → 拆分为 2-3 个简单图形
   - 多个案例研究挤在一起 → 拆分为单独图形
   - 需要超过 3 秒才能理解 → 简化并重新生成

**常见失败和修复：**
- "7 阶段工作流程，文本小" → 重新生成为 "仅 3 个高级阶段"
- "一个图形中 3 个案例研究" → 生成 3 个单独的简单图形
- "时间线有 8 年" → 用 "ONLY 3 key milestones" 重新生成
- "5 种方法比较" → 用 "ONLY Our method vs Best baseline (2 bars)" 重新生成

**如果任何图形未通过上述检查，不要继续进行组装。**

### 第 3 步：在 LaTeX 模板中组装

所有图片通过生成后审查后，包含到海报模板中：

**tikzposter 示例：**
```latex
\documentclass[25pt, a0paper, portrait]{tikzposter}

\begin{document}

\maketitle

\begin{columns}
\column{0.5}

\block{引言}{
  \centering
  \includegraphics[width=0.85\linewidth]{figures/intro_visual.png}
  
  \vspace{0.5em}
  简要背景文本（最多 2-3 句）。
}

\block{方法}{
  \centering
  \includegraphics[width=0.9\linewidth]{figures/methods_flowchart.png}
}

\column{0.5}

\block{结果}{
  \begin{minipage}{0.48\linewidth}
    \centering
    \includegraphics[width=\linewidth]{figures/result_1.png}
  \end{minipage}
  \hfill
  \begin{minipage}{0.48\linewidth}
    \centering
    \includegraphics[width=\linewidth]{figures/result_2.png}
  \end{minipage}
  
  \vspace{0.5em}
  3-4 个要点的关键发现。
}

\block{结论}{
  \centering
  \includegraphics[width=0.8\linewidth]{figures/conclusions_graphic.png}
}

\end{columns}

\end{document}
```

**baposter 示例：**
```latex
\headerbox{方法}{name=methods,column=0,row=0}{
  \centering
  \includegraphics[width=0.95\linewidth]{figures/methods_flowchart.png}
}

\headerbox{结果}{name=results,column=1,row=0}{
  \includegraphics[width=\linewidth]{figures/comparison_chart.png}
  \vspace{0.3em}
  
  关键发现：我们的方法达到 92% 准确率。
}
```

### 示例：完整的海报生成工作流程

**包含所有质量检查的完整工作流程：**

```bash
# 第 0 步：预生成审查（强制）
# 内容计划：药物发现海报
# - 工作流程：7 个阶段 → ❌ 太多 → 减少为 3 个超级阶段 ✅
# - 3 个案例研究 → ❌ 太多 → 每个图形一个案例（制作 3 个图形） ✅
# - 时间线 2018-2024 → ❌ 太详细 → 仅 3 个关键年份 ✅

# 第 1 步：创建图形目录
mkdir -p figures

# 第 2 步：生成超简单图形，严格限制

# 工作流程 - 仅高级（从 7 个阶段折叠为 3 个）
python scripts/generate_schematic.py "POSTER FORMAT for A0. ULTRA-SIMPLE 3-box workflow: 'DISCOVER' → 'VALIDATE' → 'APPROVE'. Each word 120pt+ bold. Thick arrows (10px). 60% white space. ONLY 3 words total. Readable from 12 feet." -o figures/workflow.png

# 案例研究 1 - 一个案例，一个指标（将制作 3 个单独的图形）
python scripts/generate_schematic.py "POSTER FORMAT for A0. ONE case: Company logo + '18 MONTHS' (150pt bold) + 'to drug discovery' (60pt). 3 elements only. 60% white space. Readable from 12 feet." -o figures/case1.png

python scripts/generate_schematic.py "POSTER FORMAT for A0. ONE case: Company logo + '95% SUCCESS' (150pt bold) + 'in trials' (60pt). 3 elements only. 60% white space." -o figures/case2.png

python scripts/generate_schematic.py "POSTER FORMAT for A0. ONE case: Company logo + 'FDA APPROVED' (150pt bold) + '2024' (60pt). 3 elements only. 60% white space." -o figures/case3.png

# 时间线 - 仅 3 个关键年份（不是 7 年）
python scripts/generate_schematic.py "POSTER FORMAT for A0. ONLY 3 years: '2018' (150pt) + icon, '2021' (150pt) + icon, '2024' (150pt) + icon. Large icons. 60% white space. NO lines or details. Readable from 12 feet." -o figures/timeline.png

# 结果 - 仅 2 条柱（我们的方法与最佳基线，不是 5 种方法）
python scripts/generate_schematic.py "POSTER FORMAT for A0. TWO bars only: 'BASELINE 70%' and 'OURS 95%' (highlighted). GIANT percentages (150pt) ON bars. NO axis, NO legend. 60% white space. Readable from 12 feet." -o figures/results.png

# 第 2b 步：生成后审查（强制）
# 以 25% 缩放打开每个图片：
# ✅ workflow.png: 3 个元素，文本可读，60% 空白 - 通过
# ✅ case1.png: 3 个元素，巨大数字，干净 - 通过
# ✅ case2.png: 3 个元素，巨大数字，干净 - 通过  
# ✅ case3.png: 3 个元素，巨大数字，干净 - 通过
# ✅ timeline.png: 3 个元素，可读，简单 - 通过
# ✅ results.png: 2 条柱，巨大百分比，清晰 - 通过
# 全部通过 → 继续组装

# 第 3 步：编译 LaTeX 海报
pdflatex poster.tex

# 第 4 步：PDF 溢出检查（见第 11 节）
grep "Overfull" poster.log
# 以 100% 打开并检查所有 4 条边
```

**如果任何图形未通过第 2b 步审查：**
- 元素太多 → 用 "ONLY 3 elements" 重新生成
- 文本太小 → 用 "150pt+" 或 "GIANT BOLD (150pt+)" 重新生成
- 杂乱 → 用 "60% white space" 和 "ULTRA-SIMPLE" 重新生成
- 复杂工作流程 → 拆分为多个简单的 3 元素图形

### 视觉元素指南

**⚠️ 关键：每个图形必须有一个信息，最多 3-4 个元素。**

**绝对限制 - 这些不是指南，而是硬限制：**
- **每个图形最多 3-4 个元素**（3 个最理想）
- **每个图形最多 10 个单词**
- **最低 50% 空白区域**（60% 更好）
- **关键数字/指标最低 120pt**
- **标签最低 80pt**

**对于每个海报部分 - 严格要求：**

| 部分 | 最大元素数 | 最大单词数 | 示例提示（必需模式） |
|---------|--------------|-----------|-------------------------------------|
| **引言** | 3 个图标 | 6 个词 | "POSTER FORMAT for A0: ULTRA-SIMPLE 3 icons: [icon1] [icon2] [icon3]. ONE WORD labels (100pt bold). 60% white space. 3 words total." |
| **方法** | 3 个框 | 6 个词 | "POSTER FORMAT for A0: ULTRA-SIMPLE 3-box workflow: 'STEP1' → 'STEP2' → 'STEP3'. GIANT labels (120pt+). 60% white space. 3 words only." |
| **结果** | 2-3 条柱 | 6 个词 | "POSTER FORMAT for A0: TWO bars: 'BASELINE 70%' 'OURS 95%'. GIANT percentages (150pt+) ON bars. NO axis. 60% white space." |
| **结论** | 3 张卡片 | 9 个词 | "POSTER FORMAT for A0: THREE cards: '95%' (150pt) 'ACCURATE', '2X' (150pt) 'FASTER', checkmark 'READY'. 60% white space." |
| **案例研究** | 3 个元素 | 5 个词 | "POSTER FORMAT for A0: ONE case: logo + '18 MONTHS' (150pt) + 'to discovery' (60pt). 60% white space." |
| **时间线** | 3 个点 | 3 个词 | "POSTER FORMAT for A0: THREE years only: '2018' '2021' '2024' (150pt each). Large icons. 60% white space. NO details." |

**强制提示元素（全部必需，无例外）：**
1. **"POSTER FORMAT for A0"** - 必须放在首位
2. **"ULTRA-SIMPLE"** 或 **"ONLY X elements"** - 内容限制
3. **"GIANT (120pt+)"** 或特定字体大小 - 可读性
4. **"60% white space"** - 强制留白
5. **"readable from 10-12 feet"** - 观看距离
6. **单词/元素的准确计数** - "3 words total" 或 "ONLY 3 icons"

**总是失败的模式（立即拒绝）：**
- ❌ "7 阶段药物发现工作流程" → 拆分为 "3 个超级阶段"
- ❌ "2015-2024 时间线，逐年更新" → "ONLY 3 key years"
- ❌ "3 个案例研究，包含细节" → 制作 3 个单独的简单图形
- ❌ "5 种方法比较，包含指标" → "ONLY 2: ours vs best"
- ❌ "显示所有层的完整架构" → "3 components only"
- ❌ "显示阶段 1,2,3,4,5,6" → "3 high-level stages"

**成功的模式：**
- ✅ "从 7 个阶段折叠为 3 个超级阶段" → 适当简化
- ✅ "一个案例一个指标" → 如果需要将制作多个
- ✅ "仅 3 个里程碑" → 选择性、专注
- ✅ "2 条柱：我们的 vs 基线" → 直接比较
- ✅ "3 组件高级视图" → 适当简化

---

## 科学示意图集成

有关创建示意图的详细指南，请参考 **scientific-schematics** 技能文档。

**关键能力：**
- Nano Banana Pro 自动生成、审查和优化图表
- 创建具有适当格式的出版级图像
- 确保可访问性（色盲友好、高对比度）
- 支持复杂图表的迭代优化

---

## 核心能力

### 1. LaTeX 海报包

支持三个主要的 LaTeX 海报包，各有独特优势。有关详细比较和包特定指南，请参考 `references/latex_poster_packages.md`。

**beamerposter**：
- Beamer 演示类的扩展
- 对 Beamer 用户熟悉的语法
- 出色的主题支持和定制
- 最适合：传统学术海报、机构品牌

**tikzposter**：
- 现代、灵活的设计，集成了 TikZ
- 内置颜色主题和布局模板
- 通过 TikZ 命令进行广泛定制
- 最适合：色彩丰富、现代的设计、自定义图形

**baposter**：
- 基于框的布局系统
- 自动间距和定位
- 专业外观的默认样式
- 最适合：多栏布局、一致间距

### 2. 海报布局与结构

创建遵循视觉传达原则的有效海报布局。有关全面布局指导，请参考 `references/poster_layout_design.md`。

**常见海报部分**：
- **页眉/标题**：标题、作者、单位、标志
- **引言/背景**：研究背景和动机
- **方法/途径**：方法论和实验设计
- **结果**：关键发现，包括图片和数据可视化
- **结论**：主要收获和意义
- **参考文献**：关键引用（通常精简）
- **致谢**：资金、合作者、机构

**布局策略**：
- **基于列的布局**：2 列、3 列或 4 列网格
- **基于块的布局**：内容块的灵活排列
- **Z 形流**：引导读者逻辑地阅读内容
- **视觉层次**：使用大小、颜色和间距来强调关键点

### 3. 研究海报设计原则

应用循证设计原则以获得最大效果。有关详细设计指导，请参考 `references/poster_design_principles.md`。

**排版**：
- 标题：72-120pt，确保远处可见
- 章节标题：48-72pt
- 正文：最小 24-36pt，确保 4-6 英尺可读
- 使用无衬线字体（Arial、Helvetica、Calibri）以获得清晰度
- 最多使用 2-3 种字体族

**颜色与对比度**：
- 使用高对比度配色方案提高可读性
- 机构配色方案用于品牌
- 色盲友好调色板（避免红绿组合）
- 空白区域是活动空间——不要过度拥挤

**视觉元素**：
- 高分辨率图片（打印至少 300 DPI）
- 所有图片上大而清晰的标签
- 整个海报一致的图片样式
- 有策略地使用图标和图形
- 平衡文本与视觉内容（建议视觉内容占 40-50%）

**内容指南**：
- **少即是多**：建议总字数 300-800
- 使用项目符号而非段落，便于扫描
- 清晰、简洁的信息传递
- 自解释的图片，文本说明最少
- 用于补充材料或在线资源的二维码

### 4. 标准海报尺寸

支持国际和会议特定的海报尺寸：

**国际标准**：
- A0（841 × 1189 毫米 / 33.1 × 46.8 英寸） - 最常见的欧洲标准
- A1（594 × 841 毫米 / 23.4 × 33.1 英寸） - 较小格式
- A2（420 × 594 毫米 / 16.5 × 23.4 英寸） - 紧凑型海报

**北美标准**：
- 36 × 48 英寸（914 × 1219 毫米） - 常见美国会议尺寸
- 42 × 56 英寸（1067 × 1422 毫米） - 大幅面
- 48 × 72 英寸（1219 × 1829 毫米） - 超大尺寸

**方向**：
- 纵向（垂直） - 最常见，传统
- 横向（水平） - 更适合宽内容、时间线

### 5. 包特定模板

为每个主要包提供即用型模板。模板位于 `assets/` 目录。

**beamerposter 模板**：
- `beamerposter_template.tex` - 可定制的 beamerposter 模板

**tikzposter 模板**：
- `tikzposter_template.tex` - 可定制的 tikzposter 模板

**baposter 模板**：
- `baposter_template.tex` - 可定制的 baposter 模板

### 6. 图片和图像集成

优化海报演示的视觉内容：

**最佳实践**：
- 尽可能使用矢量图形（PDF、SVG）以获得可伸缩性
- 光栅图像：最终打印尺寸至少 300 DPI
- 一致的图像样式（边框、说明、大小）
- 将相关图片分组
- 使用子图进行比较

**LaTeX 图片命令**：
```latex
% 包含图形包
\usepackage{graphicx}

% 简单图片
\includegraphics[width=0.8\linewidth]{figure.pdf}

% 在 tikzposter 中带有说明的图片
\block{结果}{
  \begin{tikzfigure}
    \includegraphics[width=0.9\linewidth]{results.png}
  \end{tikzfigure}
}

% 多个子图
\usepackage{subcaption}
\begin{figure}
  \begin{subfigure}{0.48\linewidth}
    \includegraphics[width=\linewidth]{fig1.pdf}
    \caption{条件 A}
  \end{subfigure}
  \begin{subfigure}{0.48\linewidth}
    \includegraphics[width=\linewidth]{fig2.pdf}
    \caption{条件 B}
  \end{subfigure}
\end{figure}
```

### 7. 配色方案和主题

为各种上下文提供专业调色板：

**学术机构颜色**：
- 匹配大学或院系品牌
- 使用官方颜色代码（RGB、CMYK 或 LaTeX 颜色定义）

**科学调色板**（色盲友好）：
- Viridis：从紫色到黄色的专业渐变
- ColorBrewer：经过研究测试的数据可视化调色板
- IBM Color Blind Safe：可访问的企业调色板

**包特定主题选择**：

**beamerposter**：
```latex
\usetheme{Berlin}
\usecolortheme{beaver}
```

**tikzposter**：
```latex
\usetheme{Rays}
\usecolorstyle{Denmark}
```

**baposter**：
```latex
\begin{poster}{
  background=plain,
  bgColorOne=white,
  headerColorOne=blue!70,
  textborder=rounded
}
```

### 8. 排版和文本格式

确保可读性和视觉吸引力：

**字体选择**：
```latex
% 推荐用于海报的无衬线字体
\usepackage{helvet}      % Helvetica
\usepackage{avant}       % Avant Garde
\usepackage{sfmath}      % 无衬线数学字体

% 设置默认无衬线字体
\renewcommand{\familydefault}{\sfdefault}
```

**文本大小**：
```latex
% 调整文本大小以提高可见性
\setbeamerfont{title}{size=\VeryHuge}
\setbeamerfont{author}{size=\Large}
\setbeamerfont{institute}{size=\normalsize}
```

**强调和高亮**：
- 使用加粗表示关键术语：`\textbf{important}`
- 谨慎使用颜色高亮：`\textcolor{blue}{highlight}`
- 关键信息使用框
- 避免斜体（从远处更难阅读）

### 9. 二维码和交互元素

为现代会议增强海报交互性：

**二维码集成**：
```latex
\usepackage{qrcode}

% 链接到论文、代码仓库或补充材料
\qrcode[height=2cm]{https://github.com/username/project}

% 带有说明的二维码
\begin{center}
  \qrcode[height=3cm]{https://doi.org/10.1234/paper}\\
  \small 扫描查看完整论文
\end{center}
```

**数字增强**：
- 链接到 GitHub 仓库（代码）
- 链接到视频演示或演示
- 链接到交互式网络可视化
- 链接到补充数据或附录

### 10. 编译与输出

生成高质量的 PDF 输出，用于打印或数字显示：

**编译命令**：
```bash
# 基本编译
pdflatex poster.tex

# 带参考文献
pdflatex poster.tex
bibtex poster
pdflatex poster.tex
pdflatex poster.tex

# 对于基于 beamer 的海报
lualatex poster.tex  # 更好的字体支持
xelatex poster.tex   # Unicode 和现代字体
```

**确保全页覆盖**：

海报应使用整个页面，避免过多边距。正确配置包：

**beamerposter - 全页设置**：
```latex
\documentclass[final,t]{beamer}
\usepackage[size=a0,scale=1.4,orientation=portrait]{beamerposter}

% 移除默认的 beamer 边距
\setbeamersize{text margin left=0mm, text margin right=0mm}

% 使用 geometry 实现精确控制
\usepackage[margin=10mm]{geometry}  % 四周 10mm 边距

% 移除导航符号
\setbeamertemplate{navigation symbols}{}

% 如果不需要，移除页脚和页眉
\setbeamertemplate{footline}{}
\setbeamertemplate{headline}{}
```

**tikzposter - 全页设置**：
```latex
\documentclass[
  25pt,                      % 字体缩放
  a0paper,                   % 纸张尺寸
  portrait,                  % 方向
  margin=10mm,               % 外边距（最小）
  innermargin=15mm,          % 块内空间
  blockverticalspace=15mm,   % 块间垂直间距
  colspace=15mm,             % 列间距
  subcolspace=8mm            % 子列间距
]{tikzposter}

% 这确保内容填满页面
```

**baposter - 全页设置**：
```latex
\documentclass[a0paper,portrait,fontscale=0.285]{baposter}

\begin{poster}{
  grid=false,
  columns=3,
  colspacing=1.5em,          % 列间距
  eyecatcher=true,
  background=plain,
  bgColorOne=white,
  borderColor=blue!50,
  headerheight=0.12\textheight,  % 12% 用于页眉
  textborder=roundedleft,
  headerborder=closed,
  boxheaderheight=2em        % 一致的框标题高度
}
% 内容在此
\end{poster}
```

**常见问题及修复**：

**问题**：海报周围有大量白色边距
```latex
% 修复 beamerposter
\setbeamersize{text margin left=5mm, text margin right=5mm}

% 修复 tikzposter
\documentclass[..., margin=5mm, innermargin=10mm]{tikzposter}

% 修复 baposter - 在文档类中调整
\documentclass[a0paper, margin=5mm]{baposter}
```

**问题**：内容未填满垂直空间
```latex
% 在部分之间使用 \vfill 分配空间
\block{引言}{...}
\vfill
\block{方法}{...}
\vfill
\block{结果}{...}

% 或手动调整块间距
\vspace{1cm}  % 在特定块之间添加空间
```

**问题**：海报超出页面边界
```latex
% 检查总宽度计算
% 对于 3 列带间距：
% 总宽度 = 3×列宽 + 2×列间距 + 2×边距
% 确保等于 \paperwidth

% 通过添加可见页面边界进行调试
\usepackage{eso-pic}
\AddToShipoutPictureBG{
  \AtPageLowerLeft{
    \put(0,0){\framebox(\LenToUnit{\paperwidth},\LenToUnit{\paperheight}){}}
  }
}
```

**打印准备**：
- 为专业打印生成 PDF/X-1a
- 嵌入所有字体
- 如果需要，将颜色转换为 CMYK
- 检查所有图像的分辨率（最低 300 DPI）
- 如果打印机要求，添加出血区域（通常 3-5mm）
- 验证页面尺寸与要求完全匹配

**数字显示**：
- 针对屏幕显示使用 RGB 颜色空间
- 优化文件大小以适合电子邮件/网络
- 测试在不同屏幕上的可读性

### 11. PDF 审查与质量控制

**关键**：在打印或展示前，始终检查生成的 PDF。使用此系统性检查清单：

**第 1 步：页面尺寸验证**
```bash
# 检查 PDF 尺寸（应完全匹配海报尺寸）
pdfinfo poster.pdf | grep "Page size"

# 预期输出：
# A0: 2384 x 3370 points (841 x 1189 mm)
# 36x48": 2592 x 3456 points
# A1: 1684 x 2384 points (594 x 841 mm)
```

**第 2 步：溢出检查（关键）——编译后立即执行**

**⚠️ 这是海报失败的第一大原因。继续前务必检查。**

**第 2a 步：检查 LaTeX 日志文件**
```bash
# 检查溢出警告（这些是错误，不是建议）
grep -i "overfull\|underfull\|badbox" poster.log

# 任何 "Overfull" 警告 = 内容被截断或超出边界
# 继续前修复所有这些
```

**常见溢出警告及其含义：**
- `Overfull \hbox (15.2pt too wide)` → 文本或图形比列宽 15.2pt
- `Overfull \vbox (23.5pt too high)` → 内容比可用空间高 23.5pt
- `Badbox` → LaTeX 难以将内容适配在边界内

**第 2b 步：视觉边缘检查（PDF 阅读器中以 100% 缩放）**

**系统性检查所有四个边缘：**

1. **顶部边缘：**
   - [ ] 标题完全可见（未被截断）
   - [ ] 作者姓名完全可见
   - [ ] 没有图形触及上边距
   - [ ] 页眉内容在安全区内

2. **底部边缘：**
   - [ ] 参考文献完全可见（未被截断）
   - [ ] 致谢完整
   - [ ] 联系信息可读
   - [ ] 没有图形在底部被截断

3. **左侧边缘：**
   - [ ] 没有文本触及左边距
   - [ ] 所有项目符号完全可见
   - [ ] 图形有左边距（未溢出）
   - [ ] 列内容在边界内

4. **右侧边缘：**
   - [ ] 没有文本超出右边距
   - [ ] 图形在右侧未被截断
   - [ ] 列内容保持在边界内
   - [ ] 二维码完全可见

5. **列之间：**
   - [ ] 内容保持在各自的列内
   - [ ] 没有文本渗入相邻列
   - [ ] 图片尊重列边界

**如果任何检查失败，则存在溢出。在继续前立即修复：**

**修复层次（按顺序尝试）：**
1. **首先检查 AI 生成的图形：**
   - 它们是否太复杂（5 个以上元素）？→ 重新生成更简单的
   - 它们是否有小文本？→ 用 "150pt+" 字体重新生成
   - 是否太多？→ 减少图片数量

2. **减少部分：**
   - 超过 5-6 个部分？→ 合并或移除
   - 例如：将 "讨论" 合并到 "结论"

3. **减少文本内容：**
   - 总字数超过 800？→ 减少到 300-500
   - 每个部分超过 100 字？→ 减少到 50-80

4. **调整图片大小：**
   - 使用 `width=\linewidth`？→ 改为 `width=0.85\linewidth`
   - 使用 `width=1.0\columnwidth`？→ 改为 `width=0.9\columnwidth`

5. **增加边距（最后手段）：**
   ```latex
   \documentclass[25pt, a0paper, portrait, margin=25mm]{tikzposter}
   ```

**如果存在任何溢出，不要继续第 3 步。**

**第 3 步：视觉检查清单**

以 100% 缩放打开 PDF 并检查：

**布局和间距**：
- [ ] 内容填满整个页面（没有大的白色边距）
- [ ] 列之间间距一致
- [ ] 块/部分之间间距一致
- [ ] 所有元素正确对齐（使用标尺工具）
- [ ] 没有文本或图片重叠
- [ ] 空白区域均匀分布

**排版**：
- [ ] 标题清晰可见且大（72pt+）
- [ ] 章节标题可读（48-72pt）
- [ ] 正文在 100% 缩放时可读（最小 24-36pt）
- [ ] 没有文本被截断或溢出边缘
- [ ] 整个海报字体使用一致
- [ ] 所有特殊字符正确渲染（符号、希腊字母）

**视觉元素**：
- [ ] 所有图片正确显示
- [ ] 没有像素化或模糊的图像
- [ ] 图片说明存在且可读
- [ ] 颜色按预期渲染（不显褪色或太暗）
- [ ] 标志显示清晰
- [ ] 二维码可见且可扫描

**内容完整性**：
- [ ] 标题和作者完整
- [ ] 所有部分都存在（引言、方法、结果、结论）
- [ ] 参考文献已包含
- [ ] 联系信息可见
- [ ] 致谢（如适用）
- [ ] 没有剩余的占位符文本（Lorem ipsum、TODO 等）

**技术质量**：
- [ ] 重要区域没有 LaTeX 编译警告
- [ ] 所有引用已解决（没有 [?] 标记）
- [ ] 所有交叉引用工作正常
- [ ] 页面边界正确（没有内容被截断）

**第 4 步：缩小比例打印测试**

**必要的打印前测试**：
```bash
# 创建缩小尺寸的测试打印（最终尺寸的 25%）
# 这模拟从约 8-10 英尺观看完整海报

# 对于 A0 海报，在 A4 纸上打印（24.7% 比例）
# 对于 36x48" 海报，在信纸上打印（约 25% 比例）
```

**打印测试清单**：
- [ ] 标题从 6 英尺外可读
- [ ] 章节标题从 4 英尺外可读
- [ ] 正文从 2 英尺外可读
- [ ] 图片清晰且可理解
- [ ] 颜色打印准确
- [ ] 没有明显设计缺陷

**第 5 步：数字质量检查**

**字体嵌入验证**：
```bash
# 检查所有字体是否已嵌入（打印必需）
pdffonts poster.pdf

# 所有字体应在 "emb" 列显示 "yes"
# 如果任何显示 "no"，重新编译：
pdflatex -dEmbedAllFonts=true poster.tex
```

**图像分辨率检查**：
```bash
# 提取图像信息
pdfimages -list poster.pdf

# 检查所有图像至少为 300 DPI
# 公式：DPI = 像素 / （海报英寸数）
# 对于 A0 宽度（33.1"）：300 DPI = 至少 9930 像素
```

**文件大小优化**：
```bash
# 对于电子邮件/网络，如果需要则压缩（>50MB）
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 \
   -dPDFSETTINGS=/printer -dNOPAUSE -dQUIET -dBATCH \
   -sOutputFile=poster_compressed.pdf poster.pdf

# 对于打印，保留原始文件（不压缩）
```

**第 6 步：可访问性检查**

**颜色对比度验证**：
- [ ] 文本-背景对比度 ≥ 4.5:1（WCAG AA）
- [ ] 重要元素对比度 ≥ 7:1（WCAG AAA）
- 在线测试：https://webaim.org/resources/contrastchecker/

**色盲模拟**：
- [ ] 通过色盲模拟器查看 PDF
- [ ] 信息不会因红绿模拟而丢失
- [ ] 使用 Coblis（color-blindness.com）或类似工具

**第 7 步：内容校对**

**系统性审查**：
- [ ] 拼写检查所有文本
- [ ] 验证所有作者姓名和单位
- [ ] 检查所有数字和统计数据的准确性
- [ ] 确认所有引用正确
- [ ] 审查图片标签和说明
- [ ] 检查标题和标题中的拼写错误

**同行审查**：
- [ ] 请同事审查海报
- [ ] 30 秒测试：他们能否识别主要信息？
- [ ] 5 分钟审查：他们是否理解结论？
- [ ] 记录任何令人困惑的元素

**第 8 步：技术验证**

**LaTeX 编译日志审查**：
```bash
# 检查 .log 文件中的警告
grep -i "warning\|error\|overfull\|underfull" poster.log

# 需要修复的常见问题：
# - Overfull hbox：文本超出边距
# - Underfull hbox：间距过大
# - Missing references：引用未解析
# - Missing figures：图像文件未找到
```

**修复常见警告**：
```latex
% Overfull hbox（文本太宽）
\usepackage{microtype}  % 更好的间距
\sloppy  % 允许稍微宽松的间距
\hyphenation{long-word}  % 手动断字

% 缺少字体
\usepackage[T1]{fontenc}  % 更好的字体编码

% 图像未找到
% 确保路径正确且文件存在
\graphicspath{{./figures/}{./images/}}
```

**第 9 步：最终打印前检查清单**

**发送到打印机前**：
- [ ] PDF 尺寸完全符合要求（用 pdfinfo 检查）
- [ ] 所有字体已嵌入（用 pdffonts 检查）
- [ ] 颜色模式正确（屏幕用 RGB，如果需要打印用 CMYK）
- [ ] 如果需要，已添加出血区域（通常 3-5mm）
- [ ] 如果需要，裁切标记可见
- [ ] 测试打印已完成并审查
- [ ] 文件名清晰：[LastName]_[Conference]_Poster.pdf
- [ ] 已保存备用副本

**确认打印规格**：
- [ ] 纸张类型（哑光 vs. 光面）
- [ ] 打印方法（喷墨、大幅面、织物）
- [ ] 颜色配置文件（如果需要，提供给打印机）
- [ ] 交付截止日期和送货地址
- [ ] 管状或扁平包装偏好

**数字演示检查清单**：
- [ ] PDF 大小已优化（电子邮件 <10MB）
- [ ] 在多个 PDF 阅读器上测试（Adobe、Preview 等）
- [ ] 在不同屏幕上正确显示
- [ ] 二维码已测试且功能正常
- [ ] 已准备替代格式（社交媒体用 PNG）

**审查脚本**（位于 `scripts/review_poster.sh`）：
```bash
#!/bin/bash
# 自动海报 PDF 审查脚本

echo "海报 PDF 质量检查"
echo "======================="

# 检查文件是否存在
if [ ! -f "$1" ]; then
    echo "错误：文件未找到"
    exit 1
fi

echo "文件：$1"
echo ""

# 检查页面尺寸
echo "1. 页面尺寸："
pdfinfo "$1" | grep "Page size"
echo ""

# 检查字体
echo "2. 字体嵌入："
pdffonts "$1" | head -20
echo ""

# 检查文件大小
echo "3. 文件大小："
ls -lh "$1" | awk '{print $5}'
echo ""

# 计算页数（海报应为 1 页）
echo "4. 页数："
pdfinfo "$1" | grep "Pages"
echo ""

echo "需要手动检查："
echo "- 以 100% 缩放进行视觉检查"
echo "- 缩小比例打印测试（25%）"
echo "- 颜色对比度验证"
echo "- 拼写错误校对"
```

**常见 PDF 问题及解决方案**：

| 问题 | 原因 | 解决方案 |
|------|-------|----------|
| 大量白色边距 | 边距设置不正确 | 在文档类中减少边距 |
| 内容被截断 | 超出页面边界 | 检查总宽度/高度计算 |
| 图像模糊 | 分辨率低（<300 DPI） | 替换为更高分辨率图像 |
| 缺少字体 | 字体未嵌入 | 用 -dEmbedAllFonts=true 编译 |
| 页面尺寸错误 | 纸张尺寸设置不正确 | 验证文档类的纸张尺寸 |
| 颜色看起来不对 | RGB 与 CMYK 不匹配 | 为打印转换颜色空间 |
| 文件太大（>50MB） | 未压缩的图像 | 优化图像或压缩 PDF |
| 二维码不起作用 | 太小或分辨率低 | 最小 2×2cm，高对比度 |

### 11. 常见的海报内容模式

针对不同研究类型的有效内容组织：

**实验研究海报**：
1. 标题和作者
2. 引言：问题和假设
3. 方法：实验设计（带图）
4. 结果：关键发现（2-4 个主要图片）
5. 结论：主要收获（3-5 个要点）
6. 未来工作（可选）
7. 参考文献和致谢

**计算/建模海报**：
1. 标题和作者
2. 动机：问题陈述
3. 方法：算法或模型（带流程图）
4. 实现：技术细节
5. 结果：性能指标和比较
6. 应用：使用案例
7. 代码可用性（GitHub 二维码）
8. 参考文献

**综述/调查海报**：
1. 标题和作者
2. 范围：主题概述
3. 方法：文献检索策略
4. 关键发现：主要主题（按类别组织）
5. 趋势：出版模式可视化
6. 空白：已确定的研究需求
7. 结论：总结和意义
8. 参考文献

### 12. 可访问性和包容性设计

设计面向不同受众的海报：

**色盲考虑**：
- 避免红绿组合（最常见的色盲）
- 除了颜色外，使用图案或形状
- 使用色盲模拟器测试
- 提供高对比度（WCAG AA 标准：最低 4.5:1）

**视觉障碍适应**：
- 大而清晰的字体（正文最小 24pt）
- 高对比度的文本和背景
- 清晰的视觉层次
- 避免背景中复杂的纹理或图案

**语言和内容**：
- 清晰、简洁的语言
- 定义首字母缩写词和术语
- 考虑国际受众
- 考虑为国际会议提供多语言二维码选项

### 13. 海报展示最佳实践

超越 LaTeX 的有效海报环节指导：

**内容策略**：
- 讲故事，不要只罗列事实
- 专注于 1-3 条主要信息
- 使用视觉摘要或图形摘要
- 留出对话空间（不要过度解释）

**实体展示技巧**：
- 携带打印的手册或带有二维码的名片
- 准备 30 秒、2 分钟和 5 分钟的口头摘要
- 站在一侧，不要挡住海报
- 用开放式问题吸引观众

**数字备份**：
- 在移动设备上保存 PDF 版本的海报
- 准备用于电子邮件分享的数字版本
- 创建适合社交媒体的图像版本
- 准备好备用打印副本或数字显示选项

## 海报创建工作流程

### 第一阶段：规划与内容开发

1. **确定海报要求**：
   - 会议尺寸规格（A0、36×48" 等）
   - 方向（纵向 vs. 横向）
   - 提交截止日期和格式要求

2. **开发内容大纲**：
   - 确定 1-3 条核心信息
   - 选择关键图片（通常 3-6 个主要视觉）
   - 为每个部分起草简洁文本（首选项目符号）
   - 总字数目标 300-800

3. **选择 LaTeX 包**：
   - beamerposter：如果熟悉 Beamer，需要机构主题
   - tikzposter：用于现代、色彩丰富的设计，灵活性高
   - baposter：用于结构化、专业的多栏布局

### 第二阶段：生成视觉元素（人工智能驱动）

**关键：生成简单的图片，内容最少。每个图形 = 一条信息。**

**内容限制：**
- 每个图形最多 4-5 个元素
- 每个图形最多 15 个单词
- 最低 50% 空白区域
- 巨大字体（标签 80pt+，关键数字 120pt+）

1. **创建图形目录**：
   ```bash
   mkdir -p figures
   ```

2. **生成简单的视觉元素**：
   ```bash
   # 引言 - 仅 3 个图标/元素
   python scripts/generate_schematic.py "POSTER FORMAT for A0. SIMPLE visual with ONLY 3 elements: [icon1] [icon2] [icon3]. ONE word labels (80pt+). 50% white space. Readable from 8 feet." -o figures/intro.png
   
   # 方法 - 最多 4 步
   python scripts/generate_schematic.py "POSTER FORMAT for A0. SIMPLE flowchart with ONLY 4 boxes: STEP1 → STEP2 → STEP3 → STEP4. GIANT labels (100pt+). 50% white space. NO sub-steps." -o figures/methods.png
   
   # 结果 - 仅 3 条柱/比较
   python scripts/generate_schematic.py "POSTER FORMAT for A0. SIMPLE chart with ONLY 3 bars. GIANT percentages ON bars (120pt+). NO axis, NO legend. 50% white space." -o figures/results.png
   
   # 结论 - 恰好 3 项，巨大数字
   python scripts/generate_schematic.py "POSTER FORMAT for A0. EXACTLY 3 key findings: '[NUMBER]' (150pt) '[LABEL]' (60pt) for each. 50% white space. NO other text." -o figures/conclusions.png
   ```

3. **审查生成的图片 - 检查溢出：**
   - **以 25% 缩放查看**：所有文本仍然可读？
   - **计数元素**：超过 5 个？→ 重新生成更简单的
   - **检查空白区域**：少于 40%？→ 在提示中添加 "60% white space"
   - **字体太小？**：添加 "EVEN LARGER" 或增加 pt 尺寸
   - **仍然溢出？**：将元素从 4-5 减少到 3

### 第三阶段：设计与布局

1. **选择或创建模板**：
   - 从 `assets/` 中的提供的模板开始
   - 自定义配色方案以匹配品牌
   - 配置页面尺寸和方向

2. **设计布局结构**：
   - 规划列结构（2、3 或 4 列）
   - 映射内容流（通常从左到右、从上到下）
   - 为标题分配空间（10-15%）、内容（70-80%）、页脚（5-10%）

3. **设置排版**：
   - 为不同层级配置字体大小
   - 确保正文最小 24pt
   - 从 4-6 英尺距离测试可读性

### 第四阶段：内容集成

1. **创建海报页眉**：
   - 标题（简洁、描述性、10-15 个单词）
   - 作者和单位
   - 机构标志（高分辨率）
   - 如果需要，会议标志

2. **集成 AI 生成的图片**：
   - 将第二阶段的所有图片添加到适当部分
   - 使用 `\includegraphics` 并设置正确大小
   - 确保图片在每个部分占主导地位（视觉优先，文本次之）
   - 将图片在块内居中以实现视觉冲击

3. **添加最少的支持性文本**：
   - 保持文本最少且可扫描（总字数 300-800）
   - 使用项目符号，而不是段落
   - 使用主动语态
   - 文本应补充图片，而不是重复

4. **添加补充元素**：
   - 用于补充材料的二维码
   - 参考文献（仅引用关键论文，通常 5-10 篇）
   - 联系信息和致谢

### 第五阶段：优化与测试

1. **审查和迭代**：
   - 检查拼写和错误
   - 验证所有图片为高分辨率
   - 确保格式一致
   - 确认配色方案协调

2. **测试可读性**：
   - 以 25% 比例打印，从 2-3 英尺阅读（模拟从 8-12 英尺观看海报）
   - 在不同显示器上检查颜色
   - 验证二维码功能正常
   - 请同事审查

3. **优化打印**：
   - 在 PDF 中嵌入所有字体
   - 验证图像分辨率
   - 检查 PDF 尺寸要求
   - 如果需要，包含出血区域

### 第六阶段：编译与交付

1. **编译最终 PDF**：
   ```bash
   pdflatex poster.tex
   # 或者为了更好的字体支持：
   lualatex poster.tex
   ```

2. **验证输出质量**：
   - 检查所有元素是否可见且位置正确
   - 放大到 100% 并检查图片质量
   - 验证颜色与预期一致
   - 确认 PDF 在不同阅读器中能正确打开

3. **准备打印**：
   - 如果需要，导出为 PDF/X-1a
   - 保存备用副本
   - 先在普通纸上进行测试打印
   - 在截止日期前 2-3 天订购专业打印

4. **创建补充材料**：
   - 为社交媒体保存 PNG/JPG 版本
   - 创建 handout 版本（8.5×11" 摘要）
   - 准备用于电子邮件分享的数字版本

## 与其他技能的集成

本技能与以下技能配合使用：
- **科学示意图**：关键 - 用于生成所有海报图表和流程图
- **生成图像 / Nano Banana Pro**：用于风格化图形、概念插图和摘要视觉
- **科学写作**：用于从论文开发海报内容
- **文献综述**：用于背景研究
- **数据分析**：用于创建结果图和图表

**推荐工作流程**：在创建 LaTeX 海报之前，始终先使用科学示意图和生成图像技能来生成所有视觉元素。

## 需要避免的常见陷阱

**AI 生成图形的错误（最常见）：**
- ❌ 一个图形中元素太多（10 个以上）→ 保持最多 3-5 个
- ❌ AI 图形中文本太小 → 指定 "GIANT (100pt+)" 或 "HUGE (150pt+)"
- ❌ 提示中细节太多 → 使用 "SIMPLE" 和 "ONLY X elements"
- ❌ 未指定空白区域 → 在每个提示中添加 "50% white space"
- ❌ 包含 8 步以上的复杂流程图 → 最多限制为 4-5 步
- ❌ 包含 6 个以上项目的比较图 → 最多限制为 3 个项目
- ❌ 包含 5 个以上指标的关键发现 → 仅显示前 3 个

**修复 AI 图形中的溢出：**
如果 AI 生成的图形溢出或文本太小：
1. 在提示中添加 "SIMPLER" 或 "ONLY 3 elements"
2. 增加字体大小：用 "150pt+" 代替 "80pt+"
3. 用 "60% white space" 代替 "50%"
4. 移除子细节： "NO sub-steps"、 "NO axis labels"、 "NO legend"
5. 用更少的元素重新生成

**设计错误**：
- ❌ 文本太多（超过 1000 字）
- ❌ 字体大小太小（正文小于 24pt）
- ❌ 低对比度颜色组合
- ❌ 布局拥挤，没有空白区域
- ❌ 各部分样式不一致
- ❌ 低质量或像素化的图像

**内容错误**：
- ❌ 没有清晰的叙述或信息
- ❌ 太多研究问题或目标
- ❌ 过度使用术语而不加定义
- ❌ 结果没有上下文或解释
- ❌ 缺少作者联系信息

**技术错误**：
- ❌ 海报尺寸不符合会议要求
- ❌ RGB 颜色发送到 CMYK 打印机（颜色偏移）
- ❌ PDF 中字体未嵌入
- ❌ 文件大小对于提交门户来说太大
- ❌ 二维码太小或未测试

**最佳实践**：
- ✅ 生成简单的 AI 图形，最多 3-5 个元素
- ✅ 在图形中对关键数字使用巨大字体（100pt+）
- ✅ 在每个 AI 提示中指定 "50% white space"
- ✅ 严格遵循会议尺寸规格
- ✅ 在最终打印前以缩小比例进行测试打印
- ✅ 使用高对比度、可访问的配色方案
- ✅ 保持文本最少且高度可扫描
- ✅ 包含清晰的联系信息和二维码
- ✅ 仔细校对（错误在海报上会被放大！）

## 包安装

确保安装了所需的 LaTeX 包：

```bash
# 对于 TeX Live（Linux/Mac）
tlmgr install beamerposter tikzposter baposter

# 对于 MiKTeX（Windows）
# 包通常在首次使用时自动安装

# 其他推荐包
tlmgr install qrcode graphics xcolor tcolorbox subcaption
```

## 脚本与自动化

`scripts/` 目录中提供的辅助脚本：

- `review_poster.sh`：海报审查和验证
- `generate_schematic.py`：生成科学图表和示意图

## 参考文献

用于详细指导的全面参考文件：

- `references/latex_poster_packages.md`：beamerposter、tikzposter 和 baposter 的详细比较及示例
- `references/poster_layout_design.md`：布局原则、网格系统和视觉流
- `references/poster_design_principles.md`：排版、颜色理论、视觉层次和可访问性
- `references/poster_content_guide.md`：内容组织、写作风格和部分特定指导

## 模板

`assets/` 目录中提供的即用型海报模板：

- beamerposter 模板（经典、现代、多彩）
- tikzposter 模板（默认、射线、波浪、信封）
- baposter 模板（纵向、横向、极简）
- 来自各个科学学科的示例海报
- 配色方案定义和机构模板

加载这些模板并根据你的特定研究和会议要求进行自定义。
