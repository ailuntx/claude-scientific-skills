# 专业治疗方案样式指南 - 快速参考

## 文件位置
`medical_treatment_plan.sty` - 位于资源目录中

## 快速开始

```latex
% !TEX program = xelatex
\documentclass[11pt,letterpaper]{article}
\usepackage{medical_treatment_plan}
\usepackage{natbib}

\begin{document}
\maketitle
% 您的内容
\end{document}
```

## 自定义盒子环境

### 1. 信息盒（蓝色）- 常规信息
```latex
\begin{infobox}[标题]
  内容
\end{infobox}
```
**适用场景：** 临床评估、监测计划、剂量调整方案

### 2. 警告盒（黄/红色）- 关键警报
```latex
\begin{warningbox}[标题]
  关键信息
\end{warningbox}
```
**适用场景：** 安全规程、决策点、禁忌症

### 3. 目标盒（绿色）- 治疗目标
```latex
\begin{goalbox}[标题]
  目标与指标
\end{goalbox}
```
**适用场景：** SMART目标、预期结果、成功指标

### 4. 要点盒（浅蓝色）- 核心摘要
```latex
\begin{keybox}[标题]
  重要摘要
\end{keybox}
```
**适用场景：** 执行摘要、关键结论、核心建议

### 5. 紧急盒（红色）- 应急信息
```latex
\begin{emergencybox}
  紧急联系人
\end{emergencybox}
```
**适用场景：** 紧急联系人、应急预案

### 6. 患者信息盒（白/蓝色）- 人口统计
```latex
\begin{patientinfo}
  患者信息
\end{patientinfo}
```
**适用场景：** 患者人口统计与基线数据

## 专业表格

```latex
\begin{medtable}{标题}
\begin{tabular}{|l|l|l|}
\hline
\tableheadercolor
\textcolor{white}{\textbf{表头1}} & \textcolor{white}{\textbf{表头2}} \\
\hline
数据行1 \\
\hline
\tablerowcolor  % 交替灰色行
数据行2 \\
\hline
\end{tabular}
\caption{表格说明}
\end{medtable}
```

## 配色方案

- **主蓝** (0, 102, 153)：标题与页眉
- **辅蓝** (102, 178, 204)：浅色背景
- **强调蓝** (0, 153, 204)：链接与高亮
- **成功绿** (0, 153, 76)：目标
- **警告红** (204, 0, 0)：警示信息

## 编译流程

```bash
xelatex document.tex
bibtex document
xelatex document.tex
xelatex document.tex
```

## 最佳实践

1. **匹配盒子类型：** 绿色用于目标，红/黄色用于警告
2. **避免过度使用：** 仅用于关键信息
3. **保持色彩一致：** 遵循既定配色方案
4. **合理留白：** 主章节间添加 `\vspace{0.5cm}`
5. **表格隔行着色：** 使用 `\tablerowcolor` 提升可读性

## 安装方法

**选项1：** 复制 `assets/medical_treatment_plan.sty` 到文档目录

**选项2：** 安装到用户TeX目录
```bash
mkdir -p ~/texmf/tex/latex/medical_treatment_plan
cp assets/medical_treatment_plan.sty ~/texmf/tex/latex/medical_treatment_plan/
texhash ~/texmf
```

## 依赖宏包
样式文件自动加载：
- tcolorbox, tikz, xcolor
- fancyhdr, titlesec, enumitem
- booktabs, longtable, array, colortbl
- hyperref, natbib, fontspec

## 示例结构

```latex
\maketitle

\section*{患者信息}
\begin{patientinfo}
  人口统计数据
\end{patientinfo}

\section{执行摘要}
\begin{keybox}[方案概览]
  核心要点
\end{keybox}

\section{治疗目标}
\begin{goalbox}[SMART目标]
  目标清单
\end{goalbox}

\section{用药方案}
\begin{infobox}[剂量说明]
  用药指导
\end{infobox}

\begin{warningbox}[安全警示]
  警告事项
\end{warningbox}

\section{紧急情况}
\begin{emergencybox}
  联系人信息
\end{emergencybox}
```

## 故障排除

**宏包缺失：**
```bash
sudo tlmgr install tcolorbox tikz pgf
```

**特殊字符显示异常：**
- 使用XeLaTeX替代PDFLaTeX
- 或使用LaTeX命令：`$\checkmark$`, `$\geq$`

**页眉警告：**
- 样式文件已预设为22pt
- 按需调整

---

完整文档请参阅SKILL.md中的"专业文档样式"章节
