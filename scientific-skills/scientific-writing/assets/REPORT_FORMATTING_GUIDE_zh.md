# 科学报告格式指南

`scientific_report.sty` 样式包的快速参考指南。

## 概述

`scientific_report.sty` 包为科学报告、技术文档和白皮书提供专业排版格式，主要特性包括：

- **Helvetica 字体家族**：简洁现代的外观
- **专业配色方案**：蓝色系、绿色系及强调色
- **彩色框体环境**：用于组织不同类型内容
- **精美表格**：交替行颜色与专业表头
- **科学符号命令**：p值、效应量和统计指标
- **专业页眉页脚**：自动生成章节标题

---

## 配色方案

### 主色调（蓝色系）

| 颜色名称 | RGB | Hex | 用途 |
|------------|-----|-----|-------|
| `primaryblue` | (0, 51, 102) | `#003366` | 标题、主元素 |
| `secondaryblue` | (74, 144, 226) | `#4A90E2` | 子章节、次级标题 |
| `lightblue` | (220, 235, 252) | `#DCEBFC` | 关键发现框背景 |
| `accentblue` | (0, 120, 215) | `#0078D7` | 强调元素、假设框 |

### 科学色（绿色系）

| 颜色名称 | RGB | Hex | 用途 |
|------------|-----|-----|-------|
| `sciencegreen` | (0, 168, 150) | `#00A896` | 方法论框、积极发现 |
| `lightgreen` | (220, 245, 240) | `#DCF5F0` | 方法论框背景 |
| `darkgreen` | (0, 128, 96) | `#008060` | 结果框、强证据 |

### 警示色（橙/红色系）

| 颜色名称 | RGB | Hex | 用途 |
|------------|-----|-----|-------|
| `cautionorange` | (255, 140, 66) | `#FF8C42` | 局限性、警告 |
| `lightorange` | (255, 243, 224) | `#FFF3E0` | 局限性框背景 |
| `criticalred` | (198, 40, 40) | `#C62828` | 关键通知、警报 |
| `lightred` | (255, 235, 238) | `#FFEBEE` | 关键通知背景 |

### 建议色

| 颜色名称 | RGB | Hex | 用途 |
|------------|-----|-----|-------|
| `recommendpurple` | (103, 58, 183) | `#673AB7` | 建议框 |
| `lightpurple` | (237, 231, 246) | `#EDE7F6` | 建议框背景 |

### 中性色

| 颜色名称 | RGB | Hex | 用途 |
|------------|-----|-----|-------|
| `darkgray` | (66, 66, 66) | `#424242` | 正文 |
| `mediumgray` | (117, 117, 117) | `#757575` | 次要文本、定义 |
| `lightgray` | (245, 245, 245) | `#F5F5F5` | 背景、定义框 |
| `tablealt` | (248, 250, 252) | `#F8FAFC` | 表格交替行 |

---

## 框体环境

### 关键发现框（蓝色）

用于主要发现、研究成果和重要结论。

```latex
\begin{keyfindings}[自定义标题]
本研究发现治疗方案A显著优于方案B（\pvalue{0.001}, \effectsize{d}{0.75}）。
\end{keyfindings}
```

### 方法论框（绿色）

用于方法、流程和研究设计要点。

```latex
\begin{methodology}[研究设计]
本随机对照试验采用2×2因子设计，包含前后测量和6个月随访。
\end{methodology}
```

### 结果框（蓝绿色）

用于突出具体结果和统计发现。

```latex
\begin{resultsbox}[主要结局]
分析显示显著主效应，F(2, 147) = 12.45, \psig{< 0.001}, $\eta^2$ = 0.145。
\end{resultsbox}
```

### 建议框（紫色）

用于建议、启示和行动项。

```latex
\begin{recommendations}[临床启示]
\begin{enumerate}
    \item 对高风险患者实施筛查方案
    \item 根据生物标志物水平调整治疗剂量
    \item 每3个月随访患者
\end{enumerate}
\end{recommendations}
```

### 局限性框（橙色）

用于局限性、注意事项和说明。

```latex
\begin{limitations}[研究局限]
\begin{itemize}
    \item 样本局限于城市人群
    \item 横断面设计无法推断因果关系
    \item 自我报告可能引入偏差
\end{itemize}
\end{limitations}
```

### 关键通知框（红色）

用于关键警告、重要通知或安全信息。

```latex
\begin{criticalnotice}[安全警告]
存在禁忌症X的患者禁用本疗法。实施前请咨询专科医师。
\end{criticalnotice}
```

### 定义框（灰色）

用于定义、注释和补充信息。

```latex
\begin{definition}[关键术语]
\textbf{效应量}指现象强度的量化度量，与样本量无关。
\end{definition}
```

### 执行摘要框（特殊）

用于带增强样式和阴影效果的执行摘要。

```latex
\begin{executivesummary}[报告概览]
本报告呈现关于[主题]的综合分析结果。关键发现表明...
\end{executivesummary}
```

### 假设框（浅蓝色）

用于陈述研究假设。

```latex
\begin{hypothesis}[主要假设]
我们假设干预措施X将显著改善结局Y（相较于对照组）。
\end{hypothesis}
```

---

## 引用框

用于突出重要引述或声明。

```latex
\begin{pullquote}
"这些发现标志着我们对底层机制理解的范式转变。"
\end{pullquote}
```

---

## 统计框

用于突出关键统计量（每行3个）。

```latex
\begin{center}
\statbox{n = 500}{参与者}
\statbox{p < 0.001}{显著性}
\statbox{d = 0.75}{效应量}
\end{center}
```

---

## 科学符号命令

### P值

```latex
\pvalue{0.023}          % 输出: p = 0.023
\psig{< 0.001}          % 输出: p = < 0.001 (显著时加粗)
```

### 置信区间

```latex
\CI{0.45}{0.72}         % 输出: 95% CI [0.45, 0.72]
```

### 效应量

```latex
\effectsize{d}{0.75}    % 输出: d = 0.75
\effectsize{r}{0.42}    % 输出: r = 0.42
\effectsize{F(2, 97)}{12.45}  % 输出: F(2, 97) = 12.45
```

### 样本量

```latex
\samplesize{250}        % 输出: n = 250
```

### 均值与标准差

```latex
\meansd{42.5}{8.3}      % 输出: 42.5 ± 8.3
```

### 显著性标记（用于表格）

```latex
结果\sigone           % * 表示 p < 0.05
结果\sigtwo           % ** 表示 p < 0.01
结果\sigthree         % *** 表示 p < 0.001
结果\signs            % ns 表示不显著

% 表格脚注图例:
\siglegend              % 输出: *p < 0.05; **p < 0.01; ***p < 0.001; ns 不显著
```

### 质量/证据等级标记

```latex
\qualityhigh            % 高（绿色）
\qualitymedium          % 中（橙色）
\qualitylow             % 低（红色）

\evidencestrong         % 强（绿色）
\evidencemoderate       % 中（橙色）
\evidenceweak           % 弱（红色）
```

### 趋势标记

```latex
\trendup                % 绿色上三角 ▲
\trenddown              % 红色下三角 ▼
\trendflat              % 灰色右箭头 →
```

### 文本高亮

```latex
\highlight{重要文本}  % 蓝色粗体文本
```

---

## 表格格式

### 标准表格（交替行）

```latex
\begin{table}[htbp]
\centering
\caption{分组描述性统计}
\label{tab:descriptives}
\begin{tabular}{@{}lccc@{}}
\toprule
\textbf{变量} & \textbf{组A} & \textbf{组B} & \textbf{p} \\
\midrule
年龄(岁) & \meansd{42.5}{8.3} & \meansd{43.1}{7.9} & .58 \\
\rowcolor{tablealt} 得分1 & \meansd{15.2}{3.4} & \meansd{18.7}{4.1} & <.001\sigthree \\
得分2 & \meansd{22.8}{5.1} & \meansd{23.4}{4.8} & .42 \\
\rowcolor{tablealt} 得分3 & \meansd{8.9}{2.2} & \meansd{7.2}{2.5} & .003\sigtwo \\
\bottomrule
\end{tabular}

\vspace{0.5em}
{\small \siglegend}
\end{table}
```

### 含质量标记的表格

```latex
\begin{tabular}{@{}llcc@{}}
\toprule
\textbf{研究} & \textbf{设计} & \textbf{质量} & \textbf{证据} \\
\midrule
Smith et al. (2023) & RCT & \qualityhigh & \evidencestrong \\
\rowcolor{tablealt} Jones et al. (2022) & 队列研究 & \qualitymedium & \evidencemoderate \\
Lee et al. (2021) & 横断面研究 & \qualitylow & \evidenceweak \\
\bottomrule
\end{tabular}
```

### 含趋势标记的表格

```latex
\begin{tabular}{@{}lrrl@{}}
\toprule
\textbf{指标} & \textbf{基线} & \textbf{随访} & \textbf{变化} \\
\midrule
得分A & 42.5 & 58.3 & \trendup +37\% \\
\rowcolor{tablealt} 得分B & 18.2 & 15.1 & \trenddown -17\% \\
得分C & 7.8 & 7.9 & \trendflat +1\% \\
\bottomrule
\end{tabular}
```

---

## 图表格式

### 标准图表

```latex
\begin{figure}[htbp]
\centering
\includegraphics[width=0.9\textwidth]{../figures/results_chart.png}
\caption{不同治疗条件下的结局得分比较}
\label{fig:results}
\end{figure}
```

### 带来源标注的图表

```latex
\begin{figure}[htbp]
\centering
\includegraphics[width=0.85\textwidth]{../figures/data_visualization.png}
\caption{参与者回答的类别分布}
\figuresource{研究数据，采集于2024年1-3月}
\label{fig:distribution}
\end{figure}
```

### 带注释的图表

```latex
\begin{figure}[htbp]
\centering
\includegraphics[width=0.8\textwidth]{../figures/model_diagram.png}
\caption{变量关系的概念模型}
\figurenote{实线箭头表示直接影响；虚线箭头表示调节效应。}
\label{fig:model}
\end{figure}
```

---

## 标题页

### 标准标题页

```latex
\makereporttitle
    {研究报告标题}              % 标题
    {综合分析}                 % 副标题
    {作者姓名, 博士}            % 作者
    {研究机构}                 % 机构
    {2025年1月}                % 日期
```

### 带封面图的标题页

```latex
\makereporttitlewithimage
    {研究报告标题}              % 标题
    {综合分析}                 % 副标题
    {../figures/cover_image.png} % 图片路径
    {作者姓名, 博士}            % 作者
    {研究机构}                 % 机构
    {2025年1月}                % 日期
```

---

## 列表格式

列表自动使用蓝色项目符号/编号。

### 项目符号列表

```latex
\begin{itemize}
    \item 带蓝色符号的首个项目
    \item 第二项
    \item 第三项
\end{itemize}
```

### 编号列表

```latex
\begin{enumerate}
    \item 带蓝色编号的首个项目
    \item 第二项
    \item 第三项
\end{enumerate}
```

---

## 附录章节

```latex
\appendix

\chapter{补充材料}

\appendixsection{附加表格}
% 内容将出现在目录中

\appendixsection{测量工具}
% 附加附录内容
```

---

## 编译指南

使用 XeLaTeX 或 LuaLaTeX 获得最佳字体渲染效果：

```bash
# 使用 XeLaTeX
xelatex report.tex
bibtex report        # 若使用 BibTeX
xelatex report.tex
xelatex report.tex

# 使用 latexmk（推荐）
latexmk -xelatex report.tex

# 使用 LuaLaTeX
lualatex report.tex
```

---

## 常用模式

### 结果章节示例

```latex
\section{主要结局}

\begin{resultsbox}[主要发现]
干预组得分显著高于对照组，\effectsize{t(98)}{3.45}, \psig{< 0.001}, 
\effectsize{d}{0.69}, \CI{0.42}{0.96}。
\end{resultsbox}

表~\ref{tab:outcomes} 呈现所有结局指标的完整结果。

\begin{table}[htbp]
\centering
\caption{治疗条件下的结局指标}
\label{tab:outcomes}
\begin{tabular}{@{}lcccc@{}}
\toprule
\textbf{指标} & \textbf{对照组} & \textbf{干预组} & \textbf{d} & \textbf{p} \\
\midrule
主要 & \meansd{42.1}{8.2} & \meansd{51.3}{9.1} & 0.69\sigthree & <.001 \\
\rowcolor{tablealt} 次要 & \meansd{3.2}{1.1} & \meansd{4.1}{1.3} & 0.52\sigtwo & .004 \\
三级 & \meansd{18.5}{4.2} & \meansd{19.2}{4.5} & 0.16\signs & .328 \\
\bottomrule
\end{tabular}
\end{table}
```

### 讨论章节示例

```latex
\section{结果解读}

\begin{keyfindings}[总结]
\begin{enumerate}
    \item 主要假设获 \highlight{支持}（大效应量）
    \item 次要假设部分支持
    \item 证据质量: \evidencestrong
\end{enumerate}
\end{keyfindings}

\begin{limitations}
本研究存在若干需考量的局限性...
\end{limitations}

\begin{recommendations}[未来研究]
未来研究应关注以下方向：
\begin{enumerate}
    \item 在不同人群中复现结果
    \item 延长随访期评估长期效应
    \item 探索调节变量
\end{enumerate}
\end{recommendations}
```

---

## 故障排除

### 框体溢出
若框内内容超出页面范围：
```latex
\newpage
\begin{keyfindings}[续...]
```

### 图形定位
使用 `[htbp]` 实现灵活定位，或使用 `[H]`（需加载 `float` 宏包）精确定位：
```latex
\usepackage{float}
\begin{figure}[H]
```

### 表格过宽
使用 `\resizebox` 或缩小字号：
```latex
\resizebox{\textwidth}{!}{
\begin{tabular}{...}
...
\end{tabular}
}
```

### 字体问题
若 Helvetica 无法渲染，请确保使用 XeLaTeX 或 LuaLaTeX：
```bash
xelatex report.tex   # 禁用 pdflatex
```

---

## 速查表

| 用途 | 命令/环境 |
|---------|---------------------|
| 核心发现 | `\begin{keyfindings}...\end{keyfindings}` |
| 研究方法 | `\begin{methodology}...\end{methodology}` |
| 结果 | `\begin{resultsbox}...\end{resultsbox}` |
| 建议 | `\begin{recommendations}...\end{recommendations}` |
| 局限性 | `\begin{limitations}...\end{limitations}` |
| 警告 | `\begin{criticalnotice}...\end{criticalnotice}` |
| 定义 | `\begin{definition}...\end{definition}` |
| 执行摘要 | `\begin{executivesummary}...\end{executivesummary}` |
| 假设 | `\begin{hypothesis}...\end{hypothesis}` |
| P值 | `\pvalue{0.05}` 或 `\psig{< 0.001}` |
| 效应量 | `\effectsize{d}{0.75}` |
| 样本量 | `\samplesize{250}` |
| 均值±标准差 | `\meansd{42.5}{8.3}` |
| 置信区间 | `\CI{0.38}{0.72}` |
| 高亮文本 | `\highlight{text}` |
| 交替行色 | `\rowcolor{tablealt}` |
| 显著性标记 | `\sigone`, `\sigtwo`, `\sigthree`, `\signs` |
