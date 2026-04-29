# LaTeX Beamer 科学演示文稿制作指南

## 概述

Beamer 是用于创建具有专业统一格式的演示文稿的 LaTeX 文档类。它特别适合包含公式、代码、算法和引用的科学演示。本指南涵盖 Beamer 基础、主题、自定义和高级功能，助您打造高效的科学演讲。

## 为何选择 Beamer？

### 优势

**专业品质**：
- 统一精美的外观
- 优雅的排版（尤其数学公式）
- 出版级输出质量
- 专业主题与模板

**科学内容支持**：
- 原生公式支持（LaTeX 数学）
- 语法高亮的代码清单
- 算法环境
- 参考文献集成
- 交叉引用

**可复现性**：
- 纯文本源文件（适合版本控制）
- 程序化图表生成
- 跨演示文稿的样式一致性
- 易于维护更新

**高效性**：
- 跨演示文稿复用内容
- 一次模板，永久使用
- 自动化元素（页码、导航）
- 无需手动排版

### 劣势

**学习曲线**：
- 需要 LaTeX 知识
- 编译耗时
- 调试较复杂
- 不如 PowerPoint 所见即所得

**灵活性**：
- 复杂布局需额外工作
- 图像编辑需外部工具
- 部分设计在 PowerPoint 中更简便
- 动画功能较有限

**协作性**：
- 不适合非 LaTeX 用户
- 可能存在版本冲突
- 需安装 LaTeX 环境

## Beamer 基础文档结构

### 最小示例

```latex
\documentclass{beamer}

% 主题
\usetheme{Madrid}
\usecolortheme{beaver}

% 标题信息
\title{演示文稿标题}
\subtitle{可选副标题}
\author{您的姓名}
\institute{您的机构}
\date{\today}

\begin{document}

% 标题页
\begin{frame}
  \titlepage
\end{frame}

% 内容页
\begin{frame}{幻灯片标题}
  内容区域
\end{frame}

\end{document}
```

### 核心宏包

```latex
\documentclass{beamer}

% 编码与字体
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}

% 图形
\usepackage{graphicx}
\graphicspath{{./figures/}}

% 数学
\usepackage{amsmath, amssymb, amsthm}

% 表格
\usepackage{booktabs}
\usepackage{multirow}

% 颜色
\usepackage{xcolor}

% 算法
\usepackage{algorithm}
\usepackage{algorithmic}

% 代码清单
\usepackage{listings}

% 参考文献
\usepackage[style=authoryear,backend=biber]{biblatex}
\addbibresource{references.bib}
```

### 框架基础

```latex
% 基础框架
\begin{frame}{标题}
  内容
\end{frame}

% 带副标题框架
\begin{frame}{标题}{副标题}
  内容
\end{frame}

% 无标题框架
\begin{frame}
  内容
\end{frame}

% 脆弱框架（用于逐字/代码）
\begin{frame}[fragile]{代码示例}
  \begin{verbatim}
  def hello():
      print("Hello")
  \end{verbatim}
\end{frame}

% 纯净框架（无页眉页脚）
\begin{frame}[plain]
  全幅内容
\end{frame}
```

## 主题与外观

### 演示主题

Beamer 包含多种控制整体布局的内置主题：

**经典主题**：
```latex
\usetheme{Berlin}      % 页眉显示章节
\usetheme{Copenhagen}  % 极简风格
\usetheme{Madrid}      % 专业圆角
\usetheme{Boadilla}    % 简洁页脚
\usetheme{AnnArbor}    % 垂直导航
```

**现代主题**：
```latex
\usetheme{CambridgeUS}  % 蓝色主题
\usetheme{Singapore}    % 极简主义
\usetheme{Rochester}    % 极度简洁
\usetheme{Antibes}      % 树状导航
```

**科学领域常用**：
```latex
% 简洁极简
\usetheme{default}
\usetheme{Copenhagen}

% 带导航的专业风格
\usetheme{Madrid}
\usetheme{Berlin}

% 传统学术风格
\usetheme{Pittsburgh}
\usetheme{Boadilla}
```

### 颜色主题

```latex
% 蓝色系
\usecolortheme{default}      % 蓝色
\usecolortheme{dolphin}      % 青蓝色
\usecolortheme{seagull}      % 灰度

% 暖色系
\usecolortheme{beaver}       % 红棕色
\usecolortheme{rose}         % 粉红色

% 自然系
\usecolortheme{orchid}       % 紫色
\usecolortheme{crane}        % 橙黄色

% 专业系
\usecolortheme{albatross}    % 灰蓝色
```

### 字体主题

```latex
\usefonttheme{default}              % 标准
\usefonttheme{serif}                % 衬线字体
\usefonttheme{structurebold}        % 加粗结构
\usefonttheme{structureitalicserif} % 斜体衬线
\usefonttheme{professionalfonts}    % 专业字体
```

### 自定义颜色

```latex
% 定义自定义颜色
\definecolor{myblue}{RGB}{0,115,178}
\definecolor{myred}{RGB}{214,40,40}

% 应用于主题元素
\setbeamercolor{structure}{fg=myblue}
\setbeamercolor{title}{fg=myred}
\setbeamercolor{frametitle}{fg=myblue,bg=white}
\setbeamercolor{block title}{fg=white,bg=myblue}
```

### 极简自定义主题

```latex
% 移除导航符号
\setbeamertemplate{navigation symbols}{}

% 页码设置
\setbeamertemplate{footline}[frame number]

% 简洁项目符号
\setbeamertemplate{itemize items}[circle]

% 纯净区块
\setbeamertemplate{blocks}[rounded][shadow=false]

% 配色方案
\setbeamercolor{structure}{fg=blue!70!black}
\setbeamercolor{title}{fg=black}
\setbeamercolor{frametitle}{fg=blue!70!black}
```

## 内容元素

### 列表

**项目符号**：
```latex
\begin{frame}{要点列表}
  \begin{itemize}
    \item 第一要点
    \item 第二要点
      \begin{itemize}
        \item 嵌套要点
      \end{itemize}
    \item 第三要点
  \end{itemize}
\end{frame}
```

**编号列表**：
```latex
\begin{frame}{编号列表}
  \begin{enumerate}
    \item 第一项
    \item 第二项
    \item 第三项
  \end{enumerate}
\end{frame}
```

**描述列表**：
```latex
\begin{frame}{术语定义}
  \begin{description}
    \item[术语1] 术语1的定义
    \item[术语2] 术语2的定义
  \end{description}
\end{frame}
```

### 分栏

```latex
\begin{frame}{双栏布局}
  \begin{columns}
    
    % 左栏
    \begin{column}{0.5\textwidth}
      \begin{itemize}
        \item 要点1
        \item 要点2
      \end{itemize}
    \end{column}
    
    % 右栏
    \begin{column}{0.5\textwidth}
      \includegraphics[width=\textwidth]{figure.png}
    \end{column}
    
  \end{columns}
\end{frame}
```

**三栏布局**：
```latex
\begin{columns}[T] % 顶部对齐
  \begin{column}{0.32\textwidth}
    内容A
  \end{column}
  \begin{column}{0.32\textwidth}
    内容B
  \end{column}
  \begin{column}{0.32\textwidth}
    内容C
  \end{column}
\end{columns}
```

### 图表

```latex
\begin{frame}{图表示例}
  \begin{figure}
    \centering
    \includegraphics[width=0.8\textwidth]{figure.pdf}
    \caption{图表说明文字}
  \end{figure}
\end{frame}
```

**并排图表**：
```latex
\begin{frame}{对比展示}
  \begin{columns}
    \begin{column}{0.5\textwidth}
      \includegraphics[width=\textwidth]{fig1.pdf}
      \caption{条件A}
    \end{column}
    \begin{column}{0.5\textwidth}
      \includegraphics[width=\textwidth]{fig2.pdf}
      \caption{条件B}
    \end{column}
  \end{columns}
\end{frame}
```

**子图表**：
```latex
\usepackage{subcaption}

\begin{frame}{多面板图表}
  \begin{figure}
    \centering
    \begin{subfigure}{0.45\textwidth}
      \includegraphics[width=\textwidth]{fig1.pdf}
      \caption{面板A}
    \end{subfigure}
    \hfill
    \begin{subfigure}{0.45\textwidth}
      \includegraphics[width=\textwidth]{fig2.pdf}
      \caption{面板B}
    \end{subfigure}
    \caption{整体图表说明}
  \end{figure}
\end{frame}
```

### 表格

```latex
\begin{frame}{表格示例}
  \begin{table}
    \centering
    \begin{tabular}{lcc}
      \toprule
      方法 & 准确率 & 耗时 \\
      \midrule
      方法A & 0.85 & 10秒 \\
      方法B & 0.92 & 25秒 \\
      方法C & 0.88 & 15秒 \\
      \bottomrule
    \end{tabular}
    \caption{性能对比}
  \end{table}
\end{frame}
```

### 区块

**标准区块**：
```latex
\begin{frame}{区块示例}
  
  % 标准区块
  \begin{block}{区块标题}
    区块内容
  \end{block}
  
  % 警示区块（红色）
  \begin{alertblock}{重要提示}
    警告或关键信息
  \end{alertblock}
  
  % 示例区块（绿色）
  \begin{exampleblock}{示例}
    示例内容
  \end{exampleblock}
  
\end{frame}
```

**定理环境**：
```latex
\begin{frame}{数学结果}
  
  \begin{theorem}
    定理表述
  \end{theorem}
  
  \begin{proof}
    证明过程
  \end{proof}
  
  \begin{definition}
    定义文本
  \end{definition}
  
  \begin{lemma}
    引理表述
  \end{lemma}
  
\end{frame}
```

## 覆盖与动画

### 渐进显示（\pause）

```latex
\begin{frame}{内容渐进}
  第一点立即显示
  
  \pause
  
  第二点点击后显示
  
  \pause
  
  第三点再次点击后显示
\end{frame}
```

### 覆盖规范

**带覆盖的项目列表**：
```latex
\begin{frame}{顺序显示要点}
  \begin{itemize}
    \item<1-> 幻灯片1出现并保留
    \item<2-> 幻灯片2出现并保留
    \item<3-> 幻灯片3出现并保留
  \end{itemize}
\end{frame}
```

**替代语法**：
```latex
\begin{frame}{顺序显示要点}
  \begin{itemize}[<+->]  % 自动顺序显示
    \item 第一要点
    \item 第二要点
    \item 第三要点
  \end{itemize}
\end{frame}
```

### 覆盖高亮

**特定幻灯片警示**：
```latex
\begin{frame}{高亮显示}
  \begin{itemize}
    \item 常规文本
    \item<2-| alert@2> 幻灯片2高亮文本
    \item 常规文本
  \end{itemize}
\end{frame}
```

**临时显示**：
```latex
\begin{frame}{出现与消失}
  所有幻灯片可见
  
  \only<2>{仅幻灯片2可见}
  
  \uncover<3->{幻灯片3出现并保留}
  
  \visible<4->{幻灯片4出现并保留（保留空间）}
\end{frame}
```

### 构建复杂图表

```latex
\begin{frame}{构建图表}
  \begin{tikzpicture}
    % 基础元素（始终可见）
    \draw (0,0) rectangle (4,3);
    
    % 幻灯片2+添加
    \draw<2-> (1,1) circle (0.5);
    
    % 幻灯片3+添加
    \draw<3->[->, thick] (2,1.5) -- (3,2);
    
    % 幻灯片4高亮
    \node<4>[red,thick] at (2,1.5) {结果};
  \end{tikzpicture}
\end{frame}
```

## 数学内容

### 公式

**行内公式**：
```latex
\begin{frame}{行内公式}
  著名公式 $E = mc^2$
  
  也可写作 $\alpha + \beta = \gamma$
\end{frame}
```

**显示公式**：
```latex
\begin{frame}{显示公式}
  单行公式：
  \begin{equation}
    f(x) = \int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
  \end{equation}
  
  多行公式：
  \begin{align}
    E &= mc^2 \\
    F &= ma \\
    V &= IR
  \end{align}
\end{frame}
```

**方程组**：
```latex
\begin{frame}{方程组}
  \begin{equation}
    \begin{cases}
      \dot{x} = f(x,y) \\
      \dot{y} = g(x,y)
    \end{cases}
  \end{equation}
\end{frame}
```

### 矩阵

```latex
\begin{frame}{矩阵示例}
  \begin{equation}
    A = \begin{bmatrix}
      a_{11} & a_{12} & a_{13} \\
      a_{21} & a_{22} & a_{23} \\
      a_{31} & a_{32} & a_{33}
    \end{bmatrix}
  \end{equation}
\end{frame}
```

## 代码与算法

### 代码清单

```latex
\begin{frame}[fragile]{Python代码}
  \begin{lstlisting}[language=Python]
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
  \end{lstlisting}
\end{frame}
```

**自定义代码样式**：
```latex
\lstset{
  language=Python,
  basicstyle=\ttfamily\small,
  keywordstyle=\color{blue},
  commentstyle=\color{green!60!black},
  stringstyle=\color{orange},
  numbers=left,
  numberstyle=\tiny,
  frame=single,
  breaklines=true
}

\begin{frame}[fragile]{样式化代码}
  \begin{lstlisting}
  # 这是注释
  def hello(name):
      """问候函数"""
      print(f"Hello, {name}")
  \end{lstlisting}
\end{frame}
```

### 算法

```latex
\begin{frame}{算法示例}
  \begin{algorithm}[H]
    \caption{快速排序}
```

\begin{algorithmic}[1]
      \REQUIRE 数组 $A$，索引 $low$, $high$
      \ENSURE 排序后的数组
      \IF{$low < high$}
        \STATE $pivot \gets partition(A, low, high)$
        \STATE $quicksort(A, low, pivot-1)$
        \STATE $quicksort(A, pivot+1, high)$
      \ENDIF
    \end{algorithmic}
  \end{algorithm}
\end{frame}
```

## 引用与参考文献

### 行内引用

```latex
\begin{frame}{背景}
  先前工作 \cite{smith2020} 表明...
  
  多项研究 \cite{jones2019,brown2021} 发现...
  
  根据 \textcite{davis2022}，该方法通过...
\end{frame}
```

### 参考文献页

```latex
% 演示文稿末尾
\begin{frame}[allowframebreaks]{参考文献}
  \printbibliography
\end{frame}
```

### 自定义参考文献样式

```latex
% 导言区
\usepackage[style=authoryear,maxbibnames=2,maxcitenames=2]{biblatex}
\addbibresource{references.bib}

% 参考文献小字号
\renewcommand*{\bibfont}{\scriptsize}
```

## 高级功能

### 章节组织

```latex
\section{引言}
\begin{frame}{引言}
  内容
\end{frame}

\section{方法}
\begin{frame}{方法}
  内容
\end{frame}

% 自动生成目录
\begin{frame}{目录}
  \tableofcontents
\end{frame}

% 每章节起始显示目录
\AtBeginSection{
  \begin{frame}{目录}
    \tableofcontents[currentsection]
  \end{frame}
}
```

### 备用幻灯片

```latex
% 主演示结束
\begin{frame}{致谢}
  问题？
\end{frame}

% 备用幻灯片（不计入页码）
\appendix

\begin{frame}{补充数据}
  答疑用附加分析
\end{frame}

\begin{frame}{详细方法}
  更多方法细节
\end{frame}
```

### 超链接

```latex
% 定义标签
\begin{frame}{主要结果}
  \label{mainresult}
  这是核心发现
\end{frame}

% 链接到标签页
\begin{frame}{参考}
  如\hyperlink{mainresult}{主要结果}所示...
\end{frame}

% 外部链接
\begin{frame}{资源}
  访问 \url{https://example.com} 获取更多信息
  
  \href{https://github.com/user/repo}{GitHub仓库}
\end{frame}
```

### 二维码

```latex
\usepackage{qrcode}

\begin{frame}{扫描获取论文}
  \begin{center}
    \qrcode[height=3cm]{https://doi.org/10.1234/paper}
    
    \vspace{0.5cm}
    扫描查看全文
  \end{center}
\end{frame}
```

### 多媒体

```latex
\usepackage{multimedia}

\begin{frame}{视频}
  \movie[width=8cm,height=6cm]{点击播放}{video.mp4}
\end{frame}
```

**注意**：多媒体支持因PDF阅读器而异

## TikZ绘图

### 基础形状

```latex
\usepackage{tikz}

\begin{frame}{TikZ示例}
  \begin{tikzpicture}
    % 矩形
    \draw (0,0) rectangle (2,1);
    
    % 圆形
    \draw (3,0.5) circle (0.5);
    
    % 带箭头线段
    \draw[->, thick] (0,0) -- (3,2);
    
    % 文本节点
    \node at (1.5,2) {标签};
  \end{tikzpicture}
\end{frame}
```

### 流程图

```latex
\usetikzlibrary{shapes,arrows,positioning}

\begin{frame}{工作流}
  \begin{tikzpicture}[node distance=2cm]
    \node[rectangle,draw] (start) {开始};
    \node[rectangle,draw,right=of start] (process) {处理};
    \node[rectangle,draw,right=of process] (end) {结束};
    
    \draw[->,thick] (start) -- (process);
    \draw[->,thick] (process) -- (end);
  \end{tikzpicture}
\end{frame}
```

### 数据图

```latex
\usepackage{pgfplots}
\pgfplotsset{compat=1.18}

\begin{frame}{数据图}
  \begin{tikzpicture}
    \begin{axis}[
      xlabel={$x$},
      ylabel={$y$},
      width=8cm,
      height=6cm
    ]
    \addplot[blue,thick] coordinates {
      (0,0) (1,1) (2,4) (3,9)
    };
    \addplot[red,dashed] {x};
    \end{axis}
  \end{tikzpicture}
\end{frame}
```

## 编译

### 基础编译

```bash
# 标准编译
pdflatex presentation.tex

# 含参考文献
pdflatex presentation.tex
biber presentation
pdflatex presentation.tex
pdflatex presentation.tex
```

### 现代编译（推荐）

```bash
# 使用latexmk（自动化）
latexmk -pdf presentation.tex

# 实时预览
latexmk -pdf -pvc presentation.tex
```

### 编译选项

```bash
# 快速编译（草稿模式）
pdflatex -draftmode presentation.tex

# 指定引擎
lualatex presentation.tex    # 更好的Unicode支持
xelatex presentation.tex     # 系统字体

# 输出目录
pdflatex -output-directory=build presentation.tex
```

## 讲义与备注

### 创建讲义

```latex
% 导言区
\documentclass[handout]{beamer}

% 移除动画效果，每页单帧显示
```

### 演讲者备注

```latex
\usepackage{pgfpages}
\setbeameroption{show notes on second screen=right}

\begin{frame}{幻灯片标题}
  观众可见内容
  
  \note{
    仅演讲者可见备注：
    - 强调X点
    - 提及与Y的合作
    - 预计关于Z的提问
  }
\end{frame}
```

### 含备注的讲义

```latex
\documentclass[handout]{beamer}
\usepackage{pgfpages}
\pgfpagesuselayout{2 on 1}[a4paper,border shrink=5mm]
```

## 最佳实践

### 建议事项

- ✅ 全程使用统一主题
- ✅ 保持公式简洁醒目
- ✅ 使用渐进展开(\pause, 动画层)
- ✅ 包含页码
- ✅ 图形使用矢量格式(PDF)
- ✅ 尽早并频繁测试编译
- ✅ 使用有意义的章节名
- ✅ 备用幻灯片置于附录

### 避免事项

- ❌ 避免使用过多字体或颜色
- ❌ 避免堆砌密集文本
- ❌ 避免使用过小字号
- ❌ 避免复杂动画（支持有限）
- ❌ 勿忘代码页添加[fragile]
- ❌ 避免主题混用不一致
- ❌ 勿忽略编译警告

## 故障排除

### 常见问题

**缺失Fragile选项**：
```
错误：帧内逐字环境
解决方案：为帧添加[fragile]选项
```

**包冲突**：
```
错误：包X的选项冲突
解决方案：仅在导言区加载一次
```

**图片未找到**：
```
错误：未找到`figure.pdf`文件
解决方案：检查路径，使用\graphicspath，确认文件存在
```

**动画异常**：
```
问题：动画效果不符预期
解决方案：检查<n->/<n-m>语法，测试增量编译
```

### 调试技巧

```latex
% 显示帧标签
\usepackage[notref,notcite]{showkeys}

% 草稿模式（更快，显示占位框）
\documentclass[draft]{beamer}

% 详细错误信息
\errorcontextlines=999
```

## 模板与示例

### 最小工作示例

完整可定制会议模板见：`assets/beamer_template_conference.tex`

### 资源

- Beamer用户指南：`texdoc beamer`
- 主题库：https://deic.uab.cat/~iblanes/beamer_gallery/
- TikZ示例：https://texample.net/tikz/

## 总结

Beamer优势：
- 数学公式支持
- 专业统一排版
- 可复现演示文稿
- 版本控制友好
- 引用与交叉引用

适用场景：
- 含大量数学公式
- 重视版本控制与纯文本
- 需要统一风格
- 熟悉LaTeX

考虑PowerPoint当：
- 需复杂自定义图形
- 与非LaTeX用户协作
- 需要复杂动画
- 需快速原型设计
