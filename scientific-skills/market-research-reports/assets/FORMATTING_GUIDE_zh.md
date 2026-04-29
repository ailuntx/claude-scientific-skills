# 市场研究报告格式指南

`market_research.sty`样式包的快速参考指南。

## 调色板

### 主色调
| 颜色名称 | RGB | Hex | 用途 |
|------------|-----|-----|-------|
| `primaryblue` | (0, 51, 102) | `#003366` | 标题、主标题、链接 |
| `secondaryblue` | (51, 102, 153) | `#336699` | 子章节、次要元素 |
| `lightblue` | (173, 216, 230) | `#ADD8E6` | 关键洞察框背景 |
| `accentblue` | (0, 120, 215) | `#0078D7` | 强调高亮、机会框 |

### 辅助色
| 颜色名称 | RGB | Hex | 用途 |
|------------|-----|-----|-------|
| `accentgreen` | (0, 128, 96) | `#008060` | 市场数据框、积极指标 |
| `lightgreen` | (200, 230, 201) | `#C8E6C9` | 市场数据框背景 |
| `warningorange` | (255, 140, 0) | `#FF8C00` | 风险框、警告 |
| `alertred` | (198, 40, 40) | `#C62828` | 重大风险 |
| `recommendpurple` | (103, 58, 183) | `#673AB7` | 建议框 |

### 中性色
| 颜色名称 | RGB | Hex | 用途 |
|------------|-----|-----|-------|
| `darkgray` | (66, 66, 66) | `#424242` | 正文文本 |
| `mediumgray` | (117, 117, 117) | `#757575` | 次要文本 |
| `lightgray` | (240, 240, 240) | `#F0F0F0` | 背景、标注框 |
| `tablealt` | (245, 247, 250) | `#F5F7FA` | 表格交替行 |

---

## 框体环境

### 关键洞察框（蓝色）
用于主要发现、洞察和重要结论。

```latex
\begin{keyinsightbox}[自定义标题]
预计到2030年市场将以15.3%的年复合增长率扩张，主要驱动因素是企业采用率提升和有利的监管环境。
\end{keyinsightbox}
```

### 市场数据框（绿色）
用于市场统计、指标和数据亮点。

```latex
\begin{marketdatabox}[市场概览]
\begin{itemize}
    \item \textbf{市场规模(2024年):} \marketsize{452亿}
    \item \textbf{预测规模(2030年):} \marketsize{987亿}
    \item \textbf{年复合增长率:} \growthrate{15.3}
\end{itemize}
\end{marketdatabox}
```

### 风险框（橙色/警告）
用于风险因素、警告和注意事项。

```latex
\begin{riskbox}[市场风险]
欧盟的监管变化可能在18个月内影响40%的市场参与者。
\end{riskbox}
```

### 重大风险框（红色）
用于高严重性或重大风险。

```latex
\begin{criticalriskbox}[重大：供应链中断]
主要供应链中断可能导致6-12个月延误和30%的成本增长。
\end{criticalriskbox}
```

### 建议框（紫色）
用于战略建议和行动项。

```latex
\begin{recommendationbox}[战略建议]
\begin{enumerate}
    \item 优先进入亚太市场
    \item 与本地分销商建立战略合作
    \item 投资产品本地化
\end{enumerate}
\end{recommendationbox}
```

### 标注框（灰色）
用于定义、注释和补充信息。

```latex
\begin{calloutbox}[定义：TAM]
总可寻址市场(TAM)代表获得100%市场份额时的总收入机会。
\end{calloutbox}
```

### 执行摘要框
执行摘要的特殊样式。

```latex
\begin{executivesummarybox}[执行摘要]
报告的核心发现与亮点...
\end{executivesummarybox}
```

### 机会框（青色/强调蓝）
用于机会和积极发现。

```latex
\begin{opportunitybox}[增长机会]
亚太市场代表150亿美元机会，年复合增长率达22%。
\end{opportunitybox}
```

### 框架分析框
用于战略分析框架。

```latex
% SWOT分析
\begin{swotbox}[SWOT分析摘要]
内容...
\end{swotbox}

% 波特五力模型
\begin{porterbox}[波特五力分析]
内容...
\end{porterbox}
```

---

## 引用框

用于突出重要数据或引述。

```latex
\begin{pullquote}
"人工智能与医疗健康的融合代表到2034年1990亿美元的机会。"
\end{pullquote}
```

---

## 数据框

用于突出关键统计数据（每行3个）。

```latex
\begin{center}
\statbox{\$452亿}{2024年市场规模}
\statbox{15.3\%}{2024-2030年复合增长率}
\statbox{23\%}{市场领导者份额}
\end{center}
```

---

## 自定义命令

### 文本高亮
```latex
\highlight{重要文本}  % 蓝色粗体
```

### 市场规模格式化
```latex
\marketsize{452亿}   % 输出：$452亿（绿色）
```

### 增长率格式化
```latex
\growthrate{15.3}    % 输出：15.3%（绿色）
```

### 风险指示器
```latex
\riskhigh{}     % 输出：高（红色）
\riskmedium{}   % 输出：中（橙色）
\risklow{}      % 输出：低（绿色）
```

### 星级评分（1-5）
```latex
\rating{4}      % 输出：★★★★☆
```

### 趋势指示器
```latex
\trendup{}      % 绿色上升三角
\trenddown{}    % 红色下降三角
\trendflat{}    % 灰色右向箭头
```

---

## 表格格式化

### 带交替行的标准表格
```latex
\begin{table}[htbp]
\centering
\caption{分区域市场规模}
\begin{tabular}{@{}lrrr@{}}
\toprule
\textbf{区域} & \textbf{规模} & \textbf{份额} & \textbf{复合增长率} \\
\midrule
北美 & \$182亿 & 40.3\% & 12.5\% \\
\rowcolor{tablealt} 欧洲 & \$121亿 & 26.8\% & 14.2\% \\
亚太 & \$105亿 & 23.2\% & 18.7\% \\
\rowcolor{tablealt} 其他地区 & \$44亿 & 9.7\% & 11.3\% \\
\midrule
\textbf{总计} & \textbf{\$452亿} & \textbf{100\%} & \textbf{15.3\%} \\
\bottomrule
\end{tabular}
\label{tab:regional}
\end{table}
```

### 带趋势指示器的表格
```latex
\begin{tabular}{@{}lrrl@{}}
\toprule
\textbf{公司} & \textbf{收入} & \textbf{份额} & \textbf{趋势} \\
\midrule
公司A & \$52亿 & 15.3\% & \trendup{} +12\% \\
公司B & \$48亿 & 14.1\% & \trenddown{} -3\% \\
公司C & \$42亿 & 12.4\% & \trendflat{} +1\% \\
\bottomrule
\end{tabular}
```

---

## 图表格式化

### 标准图表
```latex
\begin{figure}[htbp]
\centering
\includegraphics[width=0.9\textwidth]{../figures/market_growth.png}
\caption{市场增长轨迹(2020-2030)}
\label{fig:growth}
\end{figure}
```

### 带来源标注的图表
```latex
\begin{figure}[htbp]
\centering
\includegraphics[width=0.85\textwidth]{../figures/market_share.png}
\caption{市场份额分布(2024年)}
\figuresource{公司年报、行业分析}
\label{fig:market_share}
\end{figure}
```

---

## 列表格式化

### 项目符号列表
```latex
\begin{itemize}
    \item 带自动蓝色符号的首个项目
    \item 第二个项目
    \item 第三个项目
\end{itemize}
```

### 编号列表
```latex
\begin{enumerate}
    \item 带蓝色编号的首个项目
    \item 第二个项目
    \item 第三个项目
\end{enumerate}
```

### 嵌套列表
```latex
\begin{itemize}
    \item 主要观点
    \begin{itemize}
        \item 子观点A
        \item 子观点B
    \end{itemize}
    \item 其他主要观点
\end{itemize}
```

---

## 标题页

### 使用自定义标题命令
```latex
\makemarketreporttitle
    {市场标题}              % 报告标题
    {副标题}                % 副标题
    {../figures/cover.png}  % 主图（留空则不显示）
    {2025年1月}            % 日期
    {市场情报团队}          % 作者/编制者
```

### 手动标题页
完整代码请参考模板。

---

## 附录章节

```latex
\appendix

\chapter{方法论}

\appendixsection{数据来源}
将出现在目录中的内容...
```

---

## 常用模式

### 市场概览章节
```latex
\begin{marketdatabox}[市场概览]
\begin{itemize}
    \item \textbf{当前市场规模:} \marketsize{452亿}
    \item \textbf{预测规模(2030年):} \marketsize{987亿}
    \item \textbf{复合增长率:} \growthrate{15.3}
    \item \textbf{最大细分市场:} 企业级(42\%份额)
    \item \textbf{增长最快区域:} 亚太(\growthrate{22.1}复合增长率)
\end{itemize}
\end{marketdatabox}
```

### 风险登记摘要
```latex
\begin{table}[htbp]
\centering
\caption{风险评估摘要}
\begin{tabular}{@{}llccl@{}}
\toprule
\textbf{风险} & \textbf{类别} & \textbf{概率} & \textbf{影响} & \textbf{评级} \\
\midrule
市场中断 & 市场 & 高 & 高 & \riskhigh{} \\
\rowcolor{tablealt} 监管变化 & 监管 & 中 & 高 & \riskhigh{} \\
新进入者 & 竞争 & 中 & 中 & \riskmedium{} \\
\rowcolor{tablealt} 技术淘汰 & 技术 & 低 & 高 & \riskmedium{} \\
汇率波动 & 财务 & 中 & 低 & \risklow{} \\
\bottomrule
\end{tabular}
\end{table}
```

### 竞争对比表
```latex
\begin{table}[htbp]
\centering
\caption{竞争对比}
\begin{tabular}{@{}lccccc@{}}
\toprule
\textbf{要素} & \textbf{A公司} & \textbf{B公司} & \textbf{C公司} & \textbf{D公司} \\
\midrule
市场份额 & \rating{5} & \rating{4} & \rating{3} & \rating{2} \\
\rowcolor{tablealt} 产品质量 & \rating{4} & \rating{5} & \rating{3} & \rating{4} \\
价格竞争力 & \rating{3} & \rating{3} & \rating{5} & \rating{4} \\
\rowcolor{tablealt} 创新能力 & \rating{5} & \rating{4} & \rating{2} & \rating{3} \\
客户服务 & \rating{4} & \rating{4} & \rating{4} & \rating{5} \\
\bottomrule
\end{tabular}
\end{table}
```

---

## 故障排除

### 框体溢出
若内容超出页面，请分多个框体或使用分页符：
```latex
\newpage
\begin{keyinsightbox}[续...]
```

### 图表定位
使用`[htbp]`实现灵活定位，或使用`[H]`（需`float`包）精确定位：
```latex
\begin{figure}[H]  % 需\usepackage{float}
```

### 表格过宽
使用`\resizebox`或`adjustbox`：
```latex
\resizebox{\textwidth}{!}{
\begin{tabular}{...}
...
\end{tabular}
}
```

### 颜色未显示
确保`xcolor`包加载了`[table]`选项（样式文件已包含）。

---

## 编译

推荐使用XeLaTeX编译：
```bash
xelatex report.tex
bibtex report
xelatex report.tex
xelatex report.tex
```

或使用latexmk：
```bash
latexmk -xelatex report.tex
```
