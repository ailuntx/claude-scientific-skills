# 科学文档专业报告格式规范

本参考指南涵盖科学报告、技术文档和白皮书的专业格式规范。使用`scientific_report.sty` LaTeX样式包可确保输出格式统一且专业。

---

## 何时使用专业报告格式

### 适用场景：

- **研究报告** - 内外部研究摘要
- **技术报告** - 详细技术文档与分析
- **白皮书** - 立场文件与思想领导力文档
- **资助报告** - 进展报告与结题报告
- **政策简报** - 基于研究的政策建议
- **行业报告** - 面向行业受众的技术报告
- **内部研究摘要** - 团队与利益相关方沟通
- **可行性研究** - 技术与研究可行性评估
- **项目文档** - 研究项目交付成果

### 不适用场景：

- **期刊稿件** → 使用`venue-templates`技能满足期刊特定格式
- **会议论文** → 使用`venue-templates`技能满足会议要求
- **学位论文** → 使用机构模板
- **同行评审投稿** → 遵循期刊作者指南

**核心区别**：专业报告格式优先考虑普通读者的视觉吸引力和可读性，而期刊稿件必须遵循严格的出版商要求。

---

## scientific_report.sty 概览

`scientific_report.sty` 包提供以下功能：

| 功能 | 描述 |
|---------|-------------|
| 排版 | Helvetica字体家族呈现现代专业外观 |
| 配色方案 | 协调的蓝色、绿色、橙色与紫色系 |
| 框体环境 | 彩色框体用于内容分类 |
| 表格 | 交替行色的专业样式 |
| 图表 | 一致的标题格式 |
| 页眉/页脚 | 专业页眉页脚设计 |
| 科学命令 | p值、效应量、统计数据的快捷命令 |

### 基础文档设置

```latex
\documentclass[11pt,letterpaper]{report}
\usepackage{scientific_report}

\begin{document}
% 您的内容在此
\end{document}
```

**编译说明**：使用XeLaTeX或LuaLaTeX确保Helvetica字体正确渲染：
```bash
xelatex document.tex
```

---

## 内容组织的框体环境

### 目的与用法

彩色框体帮助读者快速识别内容类型。策略性使用以突出关键信息。

### 可用框体环境

| 环境 | 颜色 | 用途 |
|-------------|-------|---------|
| `keyfindings` | 蓝色 | 主要发现、关键结论 |
| `methodology` | 绿色 | 方法、流程、研究设计 |
| `resultsbox` | 蓝绿色 | 统计结果、数据亮点 |
| `recommendations` | 紫色 | 建议、行动项、启示 |
| `limitations` | 橙色 | 局限、注意事项 |
| `criticalnotice` | 红色 | 关键警告、安全提示 |
| `definition` | 灰色 | 定义、注释、补充信息 |
| `executivesummary` | 蓝色(阴影) | 执行摘要 |
| `hypothesis` | 浅蓝 | 研究假设 |

### 关键发现框

用于突出主要发现和重要成果：

```latex
\begin{keyfindings}[研究亮点]
分析揭示三项重要发现：
\begin{enumerate}
    \item 治疗方案A比对照组有效40% (\pvalue{0.001})
    \item 效应量具有临床意义 (\effectsize{d}{0.82})
    \item 效益在12个月随访中持续存在
\end{enumerate}
\end{keyfindings}
```

**最佳实践：**
- 节制使用（每章最多1-3个）
- 仅用于真正重要的发现
- 包含具体数值和统计数据
- 表述简洁

### 方法框

用于强调方法和流程：

```latex
\begin{methodology}[研究设计]
本双盲随机对照试验采用2×2析因设计。受试者(\samplesize{450})随机分配至四组：
(1)治疗方案A, (2)治疗方案B, (3)A+B联合方案, (4)安慰剂对照组。
\end{methodology}
```

**最佳实践：**
- 概括关键方法特征
- 在方法章节开头使用
- 包含样本量和设计类型
- 保持技术性但易懂

### 结果框

用于突出特定统计结果：

```latex
\begin{resultsbox}[主要结果分析]
混合效应回归显示显著的治疗×时间交互效应，\effectsize{F(3, 446)}{8.72}, \psig{< 0.001},
$\eta^2_p$ = 0.055，表明不同治疗方案在研究期间改善程度存在差异。
\end{resultsbox}
```

**最佳实践：**
- 报告完整统计信息
- 使用科学记法命令
- 同时呈现效应量和p值
- 每个主要分析使用一个框体

### 建议框

用于建议和启示：

```latex
\begin{recommendations}[临床实践指南]
基于研究发现，我们建议：
\begin{enumerate}
    \item \textbf{首要建议：} 对高风险人群实施筛查方案
    \item \textbf{次要建议：} 根据基线严重程度调整治疗强度
    \item \textbf{监测：} 每3个月重新评估
\end{enumerate}
\end{recommendations}
```

**最佳实践：**
- 建议需具体且可操作
- 通过标签明确优先级
- 关联支持证据
- 包含实施指导

### 局限框

用于说明局限性和注意事项：

```latex
\begin{limitations}[研究局限]
需考虑以下局限：
\begin{itemize}
    \item \textbf{样本：} 受试者来自学术医疗中心，限制社区环境普适性
    \item \textbf{设计：} 观察性设计无法推断治疗效果的因果关系
    \item \textbf{流失：} 15%脱落率可能引入偏倚
\end{itemize}
\end{limitations}
```

**最佳实践：**
- 诚实且全面
- 说明每个局限的影响
- 建议未来研究如何解决
- 避免过度弱化发现

### 关键提示框

用于关键警告或安全信息：

```latex
\begin{criticalnotice}[安全警告]
\textbf{禁忌症：} 本干预措施禁用于[特定病症]患者。监测[不良反应]，若出现[症状]立即停药。严重不良事件报告至[联系方式]。
\end{criticalnotice}
```

**最佳实践：**
- 仅用于真正关键信息
- 表述清晰直接
- 包含具体应对措施
- 必要时提供联系方式

### 定义框

用于术语定义和解释说明：

```latex
\begin{definition}[效应量]
\textbf{效应量}是量化现象规模的指标。与显著性检验不同，效应量独立于样本量，支持跨研究比较。常用指标包括均值差异的Cohen's \textit{d}和相关的Pearson's \textit{r}。
\end{definition}
```

**最佳实践：**
- 首次出现技术术语时定义
- 定义保持简洁
- 包含实际解释指南
- 用于受众适用的术语

---

## 专业表格格式

### 设计原则

1. **简洁外观**：使用`booktabs`线型(`\toprule`, `\midrule`, `\bottomrule`)
2. **交替行色**：隔行应用`\rowcolor{tablealt}`
3. **清晰表头**：加粗列标题
4. **恰当精度**：统计数据保留合适小数位
5. **完整信息**：包含样本量、单位和注释

### 标准数据表

```latex
\begin{table}[htbp]
\centering
\caption{治疗组人口学特征}
\label{tab:demographics}
\begin{tabular}{@{}lcc@{}}
\toprule
\textbf{特征} & \textbf{治疗组} & \textbf{对照组} \\
 & (\samplesize{225}) & (\samplesize{225}) \\
\midrule
年龄(岁), \meansd{M}{SD} & \meansd{42.3}{12.5} & \meansd{43.1}{11.8} \\
\rowcolor{tablealt} 女性, n (\%) & 128 (56.9) & 121 (53.8) \\
教育年限, \meansd{M}{SD} & \meansd{14.2}{2.8} & \meansd{14.5}{2.6} \\
\rowcolor{tablealt} 基线评分, \meansd{M}{SD} & \meansd{52.4}{15.3} & \meansd{51.8}{14.9} \\
\bottomrule
\end{tabular}
\figurenote{组间基线无显著差异(所有\textit{p} > .10)}
\end{table}
```

### 含显著性标记的结果表

```latex
\begin{table}[htbp]
\centering
\caption{主要与次要结果的治疗效应}
\label{tab:results}
\begin{tabular}{@{}lcccc@{}}
\toprule
\textbf{结果指标} & \textbf{治疗组} & \textbf{对照组} & \textbf{效应量} & \textbf{p} \\
 & \meansd{M}{SD} & \meansd{M}{SD} & \textbf{(d)} & \\
\midrule
主要结果 & \meansd{68.4}{14.2} & \meansd{54.1}{15.8} & 0.95\sigthree & <.001 \\
\rowcolor{tablealt} 次要指标A & \meansd{4.2}{1.1} & \meansd{3.5}{1.2} & 0.61\sigtwo & .003 \\
次要指标B & \meansd{22.8}{5.4} & \meansd{21.2}{5.1} & 0.31\sigone & .042 \\
\rowcolor{tablealt} 次要指标C & \meansd{8.9}{2.3} & \meansd{8.5}{2.4} & 0.17\signs & .285 \\
\bottomrule
\end{tabular}

\vspace{0.5em}
{\small \siglegend}
\end{table}
```

### 含质量评级的对比表

```latex
\begin{table}[htbp]
\centering
\caption{研究证据摘要}
\label{tab:evidence}
\begin{tabular}{@{}llccc@{}}
\toprule
\textbf{研究} & \textbf{设计} & \textbf{N} & \textbf{质量} & \textbf{证据强度} \\
\midrule
Smith et al. (2024) & RCT & 450 & \qualityhigh & \evidencestrong \\
\rowcolor{tablealt} Jones et al. (2023) & 队列研究 & 1,250 & \qualitymedium & \evidencemoderate \\
Chen et al. (2023) & 病例对照 & 320 & \qualitymedium & \evidencemoderate \\
\rowcolor{tablealt} Lee et al. (2022) & 横断面 & 890 & \qualitylow & \evidenceweak \\
\bottomrule
\end{tabular}
\end{table}
```

---

## 图表与标题样式

### 标题格式

样式包自动实现标题格式：
- 蓝色加粗图表标签
- 灰色描述文本
- 居中带边距对齐

### 标准图表

```latex
\begin{figure}[htbp]
\centering
\includegraphics[width=0.9\textwidth]{../figures/results_comparison.png}
\caption{不同治疗方案及时点的结果评分对比}
\label{fig:results}
\end{figure}
```

### 含来源标注的图表

```latex
\begin{figure}[htbp]
\centering
\includegraphics[width=0.85\textwidth]{../figures/trend_analysis.png}
\caption{研究期间关键指标趋势}
\figuresource{2024年1月至12月收集的研究数据}
\label{fig:trends}
\end{figure}
```

### 含说明注释的图表

```latex
\begin{figure}[htbp]
\centering
\includegraphics[width=0.8\textwidth]{../figures/conceptual_model.png}
\caption{假设关系的概念模型}
\figurenote{实线箭头表示主要路径；虚线箭头表示调节关系。数字代表标准化系数}
\label{fig:model}
\end{figure}
```

---

## 配色方案与视觉层次

### 颜色使用指南

| 颜色 | 适用场景 | 避免场景 |
|-------|---------|-----------------|
| 主蓝色 | 标题、重要发现 | 警告、注意事项 |
| 科学绿 | 方法、积极结果 | 负面发现 |
| 橙色 | 注意事项、局限 | 积极发现 |
| 红色 | 关键警告 | 常规内容 |
| 紫色 | 建议 | 发现、方法 |
| 灰色 | 定义、注释 | 关键发现 |

### 视觉层次

1. **执行摘要框**（阴影效果）- 最突出
2. **彩色内容框** - 关键内容高显性
3. **彩色表格** - 数据中等显性
4. **正文文本** - 标准显性
5. **定义框** - 补充信息低显性

### 无障碍考量

- 配色方案确保常见色觉缺陷可区分
- 所有框体兼具颜色和结构标识（边框、背景）
- 文本保持足够对比度
- 避免仅靠颜色传递信息

---

## 排版指南

### 字体规范

| 元素 | 字体 | 字号 | 颜色 |
|---------|------|------|-------|
| 正文 | Helvetica | 11pt | 深灰(#424242) |
| 章节标题 | Helvetica Bold | Huge | 主蓝(#003366) |
| 节标题 | Helvetica Bold | Large | 主蓝(#003366) |
| 小节 | Helvetica Bold | large | 辅蓝(#4A90E2) |
| 子小节 | Helvetica Bold | normalsize | 深灰(#424242) |

### 间距

- 行距：1.15（增强可读性）
- 段落间距：0.5em
- 页边距：四边各1英寸

### 排版最佳实践

1. **一致性**：同类元素统一格式
2. **层次性**：通过视觉权重区分重要性
3. **可读性**：充足间距与对比度
4. **专业性**：避免混用字体或过度修饰

---

## 科学记法命令参考

### 统计报告

| 命令 | 输出 | 使用场景 |
|---------|--------|-------------|
| `\pvalue{0.023}` | *p* = 0.023 | 报告p值 |
| `\psig{< 0.001}` | ***p*** = < 0.001 | 显著p值（加粗） |
| `\CI{0.45}{0.72}` | 95% CI [0.45, 0.72] | 置信区间 |
| `\effectsize{d}{0.75}` | d = 0.75 | 效应量 |
| `\samplesize{250}` | *n* = 250 | 样本量 |
| `\meansd{42.5}{8.3}` | 42.5 ± 8.3 | 均值±标准差 |

### 显著性标记

| 命令 | 输出 | 含义 |
|---------|--------|---------|
| `\sigone` | * | p < 0.05 |
| `\sigtwo` | ** | p < 0.01 |
| `\sigthree` | *** | p < 0.001 |
| `\signs` | ns | 不显著 |
| `\siglegend` | 完整图例 | 表格脚注 |

### 质量与证据评级

| 命令 | 输出 | 含义 |
|---------|--------|---------|
| `\qualityhigh` | **高**(绿色) | 高质量 |
| `\qualitymedium` | **中**(橙色) | 中等质量 |
| `\qualitylow` | **低**(红色) | 低质量 |
| `\evidencestrong` | **强**(绿色) | 强证据 |
| `\evidencemoderate` | **中**(橙色) | 中等证据 |
| `\evidenceweak` | **弱**(红色) | 弱证据 |

### 趋势指示符

| 命令 | 符号 | 含义 |
|---------|--------|---------|
| `\trendup` | ▲ (绿色) | 上升趋势 |
| `\trenddown` | ▼ (红色) | 下降趋势 |

| `\trendflat` | → (灰色) | 稳定/无变化 |

---

## 完整 LaTeX 示例

### 执行摘要示例

```latex
\chapter*{执行摘要}
\addcontentsline{toc}{chapter}{执行摘要}

\begin{executivesummary}[报告亮点]
本报告展示了对[主题]的综合研究结果，涵盖12个研究站点的\samplesize{450}名参与者。
该研究采用[方法论]探讨了[核心问题]。
\end{executivesummary}

\subsection*{主要发现}

\begin{keyfindings}
\begin{enumerate}
    \item 主要干预措施效果显著
          (\effectsize{d}{0.82}, \psig{< 0.001})。
    \item 效益在12个月随访期内持续存在。
    \item 成本效益分析支持实施方案。
\end{enumerate}
\end{keyfindings}

\subsection*{建议}

\begin{recommendations}
基于上述发现，我们建议：
\begin{enumerate}
    \item 在[具体场景]中实施干预措施。
    \item 采用标准化方案培训从业人员。
    \item 使用验证工具监测结果。
\end{enumerate}
\end{recommendations}
```

### 方法部分示例

```latex
\chapter{方法}

\begin{methodology}[研究概述]
本随机对照试验采用平行组设计，按1:1比例分配至干预组或对照组。
研究于2023年1月至2024年12月在12个站点开展。
\end{methodology}

\section{参与者}

共纳入\samplesize{450}名参与者。入选标准为：

\begin{itemize}
    \item 年龄18-65岁
    \item 符合[标准]的[病症]诊断
    \item 无[干预措施]禁忌症
\end{itemize}

表~\ref{tab:participants}呈现参与者特征。

\begin{limitations}[招募挑战]
由于[原因]，招募进度慢于预期。最终样本量低于目标10%，
可能影响次要分析的统计效力。
\end{limitations}
```

### 结果部分示例

```latex
\chapter{结果}

\section{主要结局}

\begin{resultsbox}[主要分析]
混合效应回归显示显著的治疗效果，
\effectsize{F(1, 448)}{42.18}, \psig{< 0.001}，效应量较大
(\effectsize{d}{0.82})。治疗组改善程度显著更高
(\meansd{16.4}{5.2}分) 对照组(\meansd{8.1}{4.8}分)。
\end{resultsbox}

图~\ref{fig:primary}展示随时间变化的治疗效果。

\begin{figure}[htbp]
\centering
\includegraphics[width=0.9\textwidth]{../figures/primary_outcome.png}
\caption{不同治疗组和时间点的主要结局得分}
\figurenote{误差线代表95\%置信区间。}
\label{fig:primary}
\end{figure}

\section{次要结局}

次要结局结果见表~\ref{tab:secondary}。
```

### 讨论部分示例

```latex
\chapter{讨论}

\section{发现总结}

\begin{keyfindings}[主要结论]
\begin{enumerate}
    \item 干预措施效果显著（主要假设\highlight{成立}）
    \item 效应具有临床意义且持久
    \item 证据强度：\evidencestrong
\end{enumerate}
\end{keyfindings}

\section{局限性}

\begin{limitations}
需考虑以下局限：
\begin{itemize}
    \item 样本以[人群]为主，限制普适性
    \item 对照组脱落率更高（18\% vs. 12\%）
    \item 自我报告测量可能存在应答偏倚
\end{itemize}
\end{limitations}

\section{启示}

\begin{recommendations}[研究启示]
\begin{enumerate}
    \item 在多样化人群中复现研究
    \item 探索作用机制
    \item 测试实施策略
\end{enumerate}
\end{recommendations}

\begin{recommendations}[实践启示]
\begin{enumerate}
    \item 在[场景]中采用干预措施
    \item 使用标准化方案培训服务提供者
    \item 监测实施保真度与结果
\end{enumerate}
\end{recommendations}
```

---

## 清单：专业报告质量

定稿前请核查：

### 格式规范
- [ ] 使用`scientific_report.sty`包
- [ ] 通过XeLaTeX或LuaLaTeX编译
- [ ] Helvetica字体渲染正常
- [ ] 色彩显示正确

### 内容组织
- [ ] 执行摘要完整存在
- [ ] 核心发现用框体突出
- [ ] 方法描述清晰
- [ ] 结果含规范统计格式
- [ ] 局限性已说明
- [ ] 建议具体且可操作

### 表格
- [ ] 所有表格含标题和标签
- [ ] 应用交替行配色
- [ ] 显著性标识已解释
- [ ] 数字格式统一

### 图表
- [ ] 所有图表含标题和标签
- [ ] 来源标注完整
- [ ] 分辨率满足印刷要求（300 DPI）
- [ ] 正文中已引用

### 统计报告
- [ ] P值报告规范
- [ ] 包含效应量
- [ ] 相关处提供置信区间
- [ ] 声明样本量

### 专业呈现
- [ ] 全文格式统一
- [ ] 无孤立标题或段落
- [ ] 分页位置恰当
- [ ] 参考文献完整规范

---

## 资源

### 本技能文件

- `assets/scientific_report.sty` - LaTeX样式包
- `assets/scientific_report_template.tex` - 完整报告模板
- `assets/REPORT_FORMATTING_GUIDE.md` - 速查指南

### 相关技能

- `venue-templates` - 期刊稿件与会议论文
- `scientific-schematics` - 图表生成
- `generate-image` - 插画与图形创作

### 外部资源

- [LaTeX维基手册](https://en.wikibooks.org/wiki/LaTeX) - 通用LaTeX参考
- [Booktabs包文档](https://ctan.org/pkg/booktabs) - 专业表格样式
- [tcolorbox包文档](https://ctan.org/pkg/tcolorbox) - 彩色框体环境
