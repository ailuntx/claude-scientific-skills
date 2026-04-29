# 假设生成报告 - 格式速查指南

## 概述

本指南提供使用假设生成LaTeX模板和样式包的快速参考。完整文档请参阅`SKILL.md`。

## 快速入门

```latex
% !TEX program = xelatex
\documentclass[11pt,letterpaper]{article}
\usepackage{hypothesis_generation}
\usepackage{natbib}

\title{您的现象名称}
\begin{document}
\maketitle
% 您的内容
\end{document}
```

**编译说明：** 推荐使用XeLaTeX或LuaLaTeX
```bash
xelatex your_document.tex
bibtex your_document
xelatex your_document.tex
xelatex your_document.tex
```

## 配色方案参考

### 假设颜色
- **假设1**：深蓝色 (RGB: 0, 102, 153) - 用于第一个假设
- **假设2**：森林绿 (RGB: 0, 128, 96) - 用于第二个假设
- **假设3**：皇家紫 (RGB: 102, 51, 153) - 用于第三个假设
- **假设4**：青蓝色 (RGB: 0, 128, 128) - 用于第四个假设（如需要）
- **假设5**：焦橙色 (RGB: 204, 85, 0) - 用于第五个假设（如需要）

### 功能颜色
- **预测**：琥珀色 (RGB: 255, 191, 0) - 用于可检验预测
- **证据**：浅蓝色 (RGB: 102, 178, 204) - 用于支持性证据
- **对比**：钢灰色 (RGB: 108, 117, 125) - 用于关键比较
- **局限性**：珊瑚红 (RGB: 220, 53, 69) - 用于局限性/挑战

## 自定义框环境

### 1. 执行摘要框

```latex
\begin{summarybox}[执行摘要]
  内容在此
\end{summarybox}
```

**用途：** 文档开头的高层概述

---

### 2. 假设框（5种变体）

```latex
\begin{hypothesisbox1}[假设1: 标题]
  \textbf{机制解释:}
  [2-3段解释HOW和WHY]
  
  \textbf{关键支持证据:}
  \begin{itemize}
    \item 证据点1 \citep{ref1}
    \item 证据点2 \citep{ref2}
  \end{itemize}
  
  \textbf{核心假设:}
  \begin{enumerate}
    \item 假设1
    \item 假设2
  \end{enumerate}
\end{hypothesisbox1}
```

**可用框体：** `hypothesisbox1`, `hypothesisbox2`, `hypothesisbox3`, `hypothesisbox4`, `hypothesisbox5`

**用途：** 呈现每个竞争性假设及其机制、证据和前提

**4页正文最佳实践：**
- 机制解释保持1-2个简短段落（最多6-10句）
- 包含2-3个最核心证据点（含引用）
- 列出1-2个最关键前提
- 确保每个假设具有实质性差异
- 所有详细解释移至附录A
- **每个假设框前使用`\newpage`防止溢出**
- 每个完整假设框应≤0.6页

---

### 3. 预测框

```latex
\begin{predictionbox}[预测: 假设1]
  \textbf{预测1.1:} [具体预测]
  \begin{itemize}
    \item \textbf{条件:} 适用场景
    \item \textbf{预期结果:} 具体可测量结果
    \item \textbf{证伪条件:} 可证伪该预测的情况
  \end{itemize}
\end{predictionbox}
```

**用途：** 展示从每个假设衍生的可检验预测

**4页正文最佳实践：**
- 预测应具体且尽可能量化
- 明确说明预测成立的条件
- 必须指定证伪标准
- 正文中每个假设仅包含1-2个最关键预测
- 额外预测移至附录

---

### 4. 证据框

```latex
\begin{evidencebox}[支持证据]
  讨论支持证据的内容
\end{evidencebox}
```

**用途：** 突出关键支持证据或文献综述

**最佳实践：**
- 正文中谨慎使用（详细证据见附录A）
- 所有证据需包含引用
- 聚焦最具说服力的证据

---

### 5. 对比框

```latex
\begin{comparisonbox}[H1 vs. H2: 关键区别]
  \textbf{根本差异:}
  [核心差异描述]
  
  \textbf{判别性实验:}
  [实验描述]
  
  \textbf{结果解读:}
  \begin{itemize}
    \item \textbf{若[结果A]:} 支持H1
    \item \textbf{若[结果B]:} 支持H2
  \end{itemize}
\end{comparisonbox}
```

**用途：** 解释如何区分竞争性假设

**最佳实践：**
- 聚焦根本机制差异
- 提出清晰可行的判别性实验
- 明确具体结果解读
- 为所有主要假设对创建比较

---

### 6. 局限框

```latex
\begin{limitationbox}[局限性与挑战]
  局限性讨论
\end{limitationbox}
```

**用途：** 强调重要局限性或挑战

**最佳实践：**
- 在局限性特别重要时使用
- 坦诚面对挑战
- 建议如何解决局限性

---

## 文档结构

### 正文（最多4页 - 高度精简）

1. **执行摘要** (0.5-1页)
   - 使用`summarybox`
   - 现象简要概述
   - 用1句话列出所有假设
   - 建议研究路径

2. **竞争性假设** (2-2.5页)
   - 使用`hypothesisbox1`, `hypothesisbox2`等
   - 每个假设对应一个框
   - 简要机制解释(1-2段)+核心证据(2-3点)+关键前提(1-2个)
   - 目标：3-5个假设
   - 保持高度精简 - 细节移至附录

3. **可检验预测** (0.5-1页)
   - 每个假设使用`predictionbox`
   - 每个假设仅包含1-2个最关键预测
   - 非常简洁 - 完整预测见附录

4. **关键比较** (0.5-1页)
   - 使用`comparisonbox`展示最高优先级比较
   - 展示如何区分主要假设
   - 额外比较移至附录

**正文总计：最多4页 - 严格筛选内容**

### 附录（全面详细）

**附录A：综合文献综述**
- 详细背景（大量引用）
- 当前认知水平
- 各假设证据（详细版）
- 矛盾发现
- 知识缺口
- **目标：40-60+篇引用**

**附录B：详细实验设计**
- 每个假设的完整方案
- 方法、对照、样本量
- 统计方法
- 可行性评估
- 时间线与资源需求

**附录C：质量评估**
- 详细评估表格
- 优劣势分析
- 对比评分
- 建议方案

**附录D：补充证据**
- 类似机制
- 初步数据
- 理论框架
- 历史背景

**参考文献**
- **目标：50+篇总引用**

## 引用最佳实践

### 正文中
- 引用15-20篇关键论文
- 括号引用使用`\citep{author2023}`
- 文本引用使用`\citet{author2023}`
- 聚焦最重要/最新证据

### 附录中
- 总计引用40-60+篇论文
- 全面覆盖相关文献
- 包含综述、原创研究、理论论文
- 每个主张和证据均需引用

### 引用密度指南
- 主要假设框：每框2-3篇引用（仅最核心）
- 正文总计：最多10-15篇引用（保持简洁）
- 附录A文献部分：每小节8-15篇引用
- 实验设计：方法参考引用2-5篇
- 质量评估：按需引用评估标准
- 文档总计：50+篇引用（绝大多数在附录）

## 表格

### 专业表格格式

```latex
\begin{hypotable}{标题}
\begin{tabular}{|l|l|l|}
\hline
\tableheadercolor
\textcolor{white}{\textbf{表头1}} & \textcolor{white}{\textbf{表头2}} \\
\hline
数据行1 & 数据 \\
\hline
\tablerowcolor  % 交替灰色背景
数据行2 & 数据 \\
\hline
\end{tabular}
\caption{您的标题}
\end{hypotable}
```

**最佳实践：**
- 表头行使用`\tableheadercolor`
- 超过3行的表格使用`\tablerowcolor`交替背景
- 保持表格可读性（避免过宽）
- 用于质量评估和比较

## 常用格式模式

### 假设章节模式

```latex
% 使用\newpage防止溢出
\newpage
\subsection*{假设N: [简洁标题]}

\begin{hypothesisboxN}[假设N: [标题]]

\textbf{机制解释:}

[1-2个简短段落 - 最多6-10句]

\vspace{0.3cm}

\textbf{关键支持证据:}
\begin{itemize}
  \item [证据1] \citep{ref1}
  \item [证据2] \citep{ref2}
  \item [证据3] \citep{ref3}
\end{itemize}

\vspace{0.3cm}

\textbf{核心前提:}
\begin{enumerate}
  \item [前提1]
  \item [前提2]
\end{enumerate}

\end{hypothesisboxN}

\vspace{0.5cm}
```

**注意：** 假设框前的`\newpage`确保从新页开始，防止溢出。当框内包含大量内容时尤为重要。

### 预测章节模式

```latex
\subsection*{假设N的预测}

\begin{predictionbox}[预测: 假设N]

\textbf{预测N.1:} [陈述]
\begin{itemize}
  \item \textbf{条件:} [适用条件]
  \item \textbf{预期结果:} [具体结果]
  \item \textbf{证伪条件:} [证伪情况]
\end{itemize}

\vspace{0.2cm}

\textbf{预测N.2:} [陈述]
[... 继续 ...]

\end{predictionbox}
```

### 比较章节模式

```latex
\subsection*{区分假设X与假设Y}

\begin{comparisonbox}[HX vs. HY: 关键区别]

\textbf{根本差异:}

[核心差异描述]

\vspace{0.3cm}

\textbf{判别性实验:}

[实验描述]

\vspace{0.3cm}

\textbf{结果解读:}
\begin{itemize}
  \item \textbf{若[结果A]:} 支持HX
  \item \textbf{若[结果B]:} 支持HY
  \item \textbf{若[结果C]:} 均支持/均不支持
\end{itemize}

\end{comparisonbox}
```

## 间距与布局

### 垂直间距
- `\vspace{0.3cm}` - 框内元素间距
- `\vspace{0.5cm}` - 主要章节或框体间距
- `\vspace{1cm}` - 标题后/正文前间距

### 分页与溢出预防

**关键：防止内容溢出**

LaTeX框体（tcolorbox环境）不会自动跨页。超出页面剩余空间的内容将导致格式问题。遵循以下准则：

1. **长框体前策略性分页：**
```latex
\newpage  % 长框体从新页开始
\begin{hypothesisbox1}[假设1: 标题]
  % 大量内容在此
\end{hypothesisbox1}
```

2. **监控框体内容长度：**
   - 每个假设框最多≤0.7页
   - 若机制解释+证据+前提超过≈0.6页，内容过长
   - 解决方案：详细内容移至附录，框内仅保留核心

3. **何时使用`\newpage`：**
   - 包含>3小节或>15行内容的假设框前
   - 含详细实验描述的对比框前
   - 附录主要章节之间
   - 当前页面剩余空间<0.6页时开始新框体

4. **正文内容长度指南：**
   - 执行摘要框：最多0.5-0.8页
   - 每个假设框：最多0.4-0.6页
   - 每个预测框：最多0.3-0.5页
   - 每个对比框：最多0.4-0.6页

5. **拆分长内容：**
   ```latex
   % 正确：精简正文+分页
   \newpage
   \begin{hypothesisbox1}[假设1: 简短标题]
   \textbf{机制解释:}
   1-2段简要概述（6-10句）。
   
   \textbf{关键支持证据:}
   \begin{itemize}
     \item 证据1 \citep{ref1}
     \item 证据2 \citep{ref2}
   \end{itemize}
   
   \textbf{核心前提:}
   \begin{enumerate}
     \item 前提1
   \end{enumerate}
   
   详细机制和完整证据见附录A。
   \end{hypothesisbox1}
   ```

   ```latex
   % 错误：过长内容将导致溢出
   \begin{hypothesisbox1}[假设1]
   \subsection{冗长章节}
   多个段落...
   \subsection{另一冗长章节}
   更多段落...
   \subsection{额外内容}
   [内容超出页面边界 → 溢出!]
   \end{hypothesisbox1}
   ```

6. **分页命令：**
   - `\newpage` - 强制分页（长框体前推荐）
   - `\clearpage` - 强制分页并清除浮动体（附录前使用）

### 章节间距
样式包已处理，但可手动调整：
```latex
\vspace{0.5cm}  % 按需添加额外间距
```

## 故障排除

### 常见问题

**问题："File hypothesis_generation.sty not found"**
- 解决方案：确保.sty文件与.tex文件同目录或在LaTeX路径中

**问题：框体无颜色**
- 解决方案：使用XeLaTeX或LuaLaTeX编译，非pdfLaTeX
- 命令：`xelatex yourfile.tex`

**问题：引用显示为[?]**
- 解决方案：首次xelatex编译后运行bibtex
```bash
xelatex yourfile.tex
bibtex yourfile
xelatex yourfile.tex
xelatex yourfile.tex
```

**问题：字体未找到**
- 解决方案：若未安装自定义字体，注释.sty文件中的字体行
- 需注释行：`\setmainfont{...}`和`\setsansfont{...}`

**问题：框标题与内容重叠**
- 解决方案：标题后添加`\vspace{0.3cm}`增加垂直间距

**问题：表格过宽**
- 解决方案：在tabular前使用`\small`或`\footnotesize`，或使用`p{宽度}`列格式

**问题：内容溢出页面**
- **原因：** 框体（tcolorbox环境）超出页面剩余空间
- **解决方案1：** 框体前添加`\newpage`从新页开始

- **解决方案2：** 精简框内内容——将详细信息移至附录  
- **解决方案3：** 将内容拆分为多个小框  
- **预防措施：** 每个假设框控制在0.4-0.6页以内；内容较多的框前自由使用`\newpage`  

**问题：正文超过4页**  
- **原因：** 框内包含过多细节信息  
- **解决方案：** 主动将内容移至附录——正文框仅保留：  
  - 简要机制概述（1-2段）  
  - 2-3个关键证据要点  
  - 1-2个核心假设  
- 所有详细解释、补充证据和完整讨论归入附录A  

### 必要宏包  

确保安装以下宏包：  
- `tcolorbox`（带`most`选项）  
- `xcolor`  
- `fontspec`（用于XeLaTeX/LuaLaTeX）  
- `fancyhdr`  
- `titlesec`  
- `enumitem`  
- `booktabs`  
- `natbib`  

安装缺失宏包：  
```bash  
# TeX Live用户  
tlmgr install tcolorbox xcolor fontspec fancyhdr titlesec enumitem booktabs natbib  

# MiKTeX用户（Windows）  
# 使用MiKTeX包管理器GUI  
```  

## 样式统一建议  

1. **色彩规范**  
   - 全文对同一假设使用固定颜色  
   - H1=蓝, H2=绿, H3=紫等  
   - 避免同一假设混用颜色  

2. **框体使用**  
   - 正文：假设框、预测框、对比框  
   - 附录：按需使用证据框、局限框  
   - 避免滥用——仅用于关键内容  

3. **引用格式**  
   - 全文保持统一引用格式  
   - 主要使用`\citep{}`  
   - 多文献合并引用：`\citep{ref1, ref2, ref3}`  

4. **假设编号**  
   - 统一编号（H1, H2, H3等）  
   - 预测编号对应假设（H1的预测为P1.1, P1.2）  
   - 对比框使用相同编号（如H1 vs. H2）  

5. **语言表达**  
   - 精准明确  
   - 避免模糊表述（"可能"、"或许"）  
   - 优先使用主动语态  
   - 预测尽量量化  

## 终稿检查清单  

提交前确认：  
- [ ] 标题页标注现象名称  
- [ ] **正文不超过4页**  
- [ ] 执行摘要精炼（0.5-1页）  
- [ ] 每个假设使用独立彩色框  
- [ ] 提出3-5个假设（勿超）  
- [ ] 每个假设含1-2段简要机制说明  
- [ ] 每个假设列2-3个核心证据点（附引用）  
- [ ] 每个假设含1-2个关键前提  
- [ ] 预测框包含每个假设的1-2个核心预测  
- [ ] 正文含优先级对比框（其他对比放附录）  
- [ ] 明确优先实验方案  
- [ ] **长框前使用分页符（`\newpage`）防溢出**  
- [ ] **无内容超出页面边界（仔细检查PDF）**  
- [ ] **每个假设框≤0.6页（过长则移细节至附录）**  
- [ ] 附录A含完整文献综述（含详细证据）  
- [ ] 附录B含详细实验方案  
- [ ] 附录C含质量评估表  
- [ ] 附录D含补充证据  
- [ ] 正文引用10-15篇（精选）  
- [ ] 全文总引用≥50篇  
- [ ] 所有框体颜色正确  
- [ ] 文档编译无报错  
- [ ] 参考文献格式正确  
- [ ] **逐页检查PDF防溢出问题**  

## 最小示例文档  

```latex  
% !TEX program = xelatex  
\documentclass[11pt,letterpaper]{article}  
\usepackage{hypothesis_generation}  
\usepackage{natbib}  

\title{X在Y中的作用}  

\begin{document}  
\maketitle  

\section*{执行摘要}  
\begin{summarybox}[执行摘要]  
现象与假设的简要概述。  
\end{summarybox}  

\section{竞争性假设}  

% 每个假设框前用\newpage防溢出  
\newpage  
\subsection*{假设1：标题}  
\begin{hypothesisbox1}[假设1：标题]  
\textbf{机制解释：}  
1-2段简要说明。  

\textbf{核心证据：}  
\begin{itemize}  
  \item 证据点 \citep{ref1}  
\end{itemize}  
\end{hypothesisbox1}  

\newpage  
\subsection*{假设2：标题}  
\begin{hypothesisbox2}[假设2：标题]  
\textbf{机制解释：}  
1-2段简要说明。  

\textbf{核心证据：}  
\begin{itemize}  
  \item 证据点 \citep{ref2}  
\end{itemize}  
\end{hypothesisbox2}  

\section{可检验预测}  

\subsection*{假设1的预测}  
\begin{predictionbox}[预测：假设1]  
预测内容。  
\end{predictionbox}  

\section{关键对比}  

\subsection*{H1 vs. H2}  
\begin{comparisonbox}[H1 vs. H2]  
对比内容。  
\end{comparisonbox}  

% 附录前强制分页  
\appendix  
\newpage  
\appendixsection{附录A：文献综述}  
详细文献综述。  

\newpage  
\bibliographystyle{plainnat}  
\bibliography{references}  

\end{document}  
```  

**要点说明：**  
- 用`\newpage`确保每个假设框从新页开始  
- 有效防止内容溢出  
- 正文框保持简洁（1-2段+要点）  
- 细节内容移至附录  

## 附加资源  

- 完整模板：`hypothesis_report_template.tex`  
- 工作流指南：`SKILL.md`  
- 评估框架：`references/hypothesis_quality_criteria.md`  
- 实验设计指南：`references/experimental_design_patterns.md`  
- LaTeX样式示例：参见treatment-plans技能库
