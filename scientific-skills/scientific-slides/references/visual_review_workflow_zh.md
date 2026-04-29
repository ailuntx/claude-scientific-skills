# 演示文稿视觉审查工作流程

## 概述

视觉审查是演示文稿质量保证的关键步骤，可在演示前识别并修复布局问题、文本溢出、元素重叠和设计缺陷。本指南涵盖演示文稿转图像方法、系统化视觉检查、常见问题及迭代优化策略。

## ⚠️ 关键规则：切勿直接读取PDF演示文稿

**强制要求：始终先将演示文稿PDF转换为图像，再审查图像。**

### 规则制定原因

- **缓冲区溢出防护**：直接读取演示文稿PDF（尤其是多页文件）会引发"JSON消息超出最大缓冲区"错误
- **视觉准确性**：图像精确呈现观众将看到的效果，包括渲染问题
- **性能优势**：基于图像的审查比PDF文本提取更快更可靠
- **流程一致性**：确保所有演示文稿采用统一审查流程

### 唯一正确的演示文稿处理流程

1. ✅ 从PowerPoint/Beamer源文件生成PDF
2. ✅ 使用pdf_to_images.py脚本**将PDF转换为图像**
3. ✅ **系统化审查图像文件**
4. ✅ 按幻灯片编号记录问题
5. ✅ 在源文件中修复问题
6. ✅ 重新生成PDF并重复流程

### 禁止操作事项

- ❌ 切勿对演示文稿PDF使用read_file工具
- ❌ 切勿尝试以文本形式读取PDF幻灯片
- ❌ 切勿跳过图像转换步骤
- ❌ 切勿假设PDF"足够小"可直接读取

**若正在审查演示文稿但未转换图像，请立即停止并先执行转换。**

## 视觉审查的重要性

### 源文件中不可见的常见问题

**LaTeX Beamer问题**：
- 文本框文本溢出
- 元素重叠（公式覆盖图像）
- 断行不当
- 图形超出幻灯片边界
- 实际分辨率下的字体大小问题

**PowerPoint问题**：
- 文本被形状或边缘截断
- 图像与文本重叠
- 幻灯片间距不一致
- 色彩渲染差异
- 字体替换问题

**投影问题**：
- 笔记本可见内容投影时被裁剪
- 投影仪色彩显示差异
- 低对比度元素不可见
- 细节丢失

### 视觉审查的优势

- **及早发现布局错误**：在打印或演示前修复
- **验证可读性**：确保文本足够大且高对比度
- **检查一致性**：发现跨幻灯片不一致
- **测试无障碍性**：验证色彩对比度和清晰度
- **确认设计效果**：保障专业外观

## 转换：PDF转图像

### 方法1：使用pdf_to_images.py脚本（推荐）

**无需外部依赖**：
脚本使用自包含的PyMuPDF库，无需poppler等系统软件。

**安装**：
```bash
# PyMuPDF已包含在项目依赖中
pip install pymupdf
```

**基础转换**：
```bash
# 将所有幻灯片转为JPEG图像
python skills/scientific-slides/scripts/pdf_to_images.py presentation.pdf slide --dpi 150

# 生成：slide-001.jpg, slide-002.jpg, slide-003.jpg, ...
```

**高分辨率转换**：
```bash
# 高质量转换用于细节检查（300 DPI）
python skills/scientific-slides/scripts/pdf_to_images.py presentation.pdf slide --dpi 300

# PNG格式（无损，文件较大）
python skills/scientific-slides/scripts/pdf_to_images.py presentation.pdf slide --dpi 150 --format png
```

**转换特定幻灯片**：
```bash
# 仅转换5-10页
python skills/scientific-slides/scripts/pdf_to_images.py presentation.pdf slide --dpi 150 --first 5 --last 10

# 单页转换
python skills/scientific-slides/scripts/pdf_to_images.py presentation.pdf slide --dpi 150 --first 3 --last 3
```

**输出选项**：
```bash
# 指定输出目录
python skills/scientific-slides/scripts/pdf_to_images.py presentation.pdf review/slide --dpi 150

# 自定义命名
python skills/scientific-slides/scripts/pdf_to_images.py presentation.pdf output/presentation --dpi 150
```

### 方法2：使用PowerPoint缩略图脚本

处理PowerPoint演示文稿时使用pptx技能缩略图工具：

```bash
# 创建缩略图网格
python scripts/thumbnail.py presentation.pptx output --cols 4

# 单页导出
python scripts/thumbnail.py presentation.pptx slides/slide --individual
```

**优势**：
- 专为PowerPoint优化
- 可创建概览网格
- 直接处理.pptx格式
- 布局可定制

### 方法3：使用ImageMagick

**安装**：
```bash
# Ubuntu/Debian
sudo apt-get install imagemagick

# macOS
brew install imagemagick
```

**转换**：
```bash
# PDF转图像
convert -density 150 presentation.pdf slide.jpg

# 高质量转换
convert -density 300 presentation.pdf slide.jpg

# 指定格式
convert -density 150 presentation.pdf slide.png
```

### 方法4：使用Python（编程实现）

```python
import fitz  # PyMuPDF

# 打开PDF
doc = fitz.open('presentation.pdf')

# 逐页转图像
zoom = 200 / 72  # 200 DPI（72为基准DPI）
matrix = fitz.Matrix(zoom, zoom)

for i, page in enumerate(doc, start=1):
    pixmap = page.get_pixmap(matrix=matrix)
    pixmap.save(f'slide-{i:03d}.jpg', output='jpeg')

doc.close()
```

**安装PyMuPDF**：
```bash
pip install pymupdf
# 无需外部依赖！
```

## 系统化视觉检查

### 检查流程

**步骤1：概览审查**
- 快速浏览所有幻灯片
- 记录整体一致性
- 标记明显问题页
- 创建需详细审查的幻灯片列表

**步骤2：细节检查**
- 仔细审查标记页
- 对照问题清单（见下文）
- 按编号记录具体问题
- 备注修复方案

**步骤3：跨页对比**
- 检查相似幻灯片一致性
- 验证统一间距与对齐
- 确保字体大小一致
- 检查配色方案统一性

**步骤4：距离测试**
- 缩小图像模拟投影效果
- 在约6英尺距离检查可读性
- 验证关键元素可见性
- 测试核心信息传达清晰度

### 问题检查清单

逐页检查以下常见问题：

#### 文本问题

**溢出与截断**：
- [ ] 文本在边缘被截断
- [ ] 文本超出文本框
- [ ] 公式超出页边距
- [ ] 标题底部被裁剪
- [ ] 项目符号超出边界

**可读性**：
- [ ] 字体过小（最小18pt可见）
- [ ] 对比度不足（文本与背景）
- [ ] 行间距不足
- [ ] 文本距边缘过近
- [ ] 文本行重叠

#### 元素重叠

**文本重叠**：
- [ ] 文本与图像重叠
- [ ] 文本与形状重叠
- [ ] 多文本框重叠
- [ ] 标签与数据点重叠
- [ ] 标题与内容重叠

**视觉元素重叠**：
- [ ] 图像相互重叠
- [ ] 形状不当重叠
- [ ] 图形超出页边距
- [ ] 图例与图表重叠
- [ ] 水印遮挡内容

#### 布局与间距

**对齐问题**：
- [ ] 文本框未对齐
- [ ] 页边距不均
- [ ] 元素位置不一致
- [ ] 标题未居中
- [ ] 项目符号未对齐

**间距问题**：
- [ ] 内容拥挤（留白不足）
- [ ] 空白过多（空间利用不佳）
- [ ] 元素间距不一致
- [ ] 多列布局间隙不均
- [ ] 内容分布不合理

#### 色彩与对比度

**可见性**：
- [ ] 对比度不足（文本与背景）
- [ ] 色彩过于相似（难以区分）
- [ ] 复杂背景上的文本
- [ ] 浅色文本配浅背景
- [ ] 深色文本配深背景

**一致性**：
- [ ] 幻灯片间配色方案不一致
- [ ] 意外色彩变化
- [ ] 冲突色彩组合
- [ ] 数据可视化配色不当

#### 图形与图表

**质量**：
- [ ] 图像模糊或像素化
- [ ] 低分辨率图形
- [ ] 宽高比失真
- [ ] 截图质量差
- [ ] 图形边缘锯齿

**布局**：
- [ ] 图形过小无法阅读
- [ ] 坐标轴标签过小
- [ ] 图例文字不清
- [ ] 复杂图形缺少说明
- [ ] 图形未居中或对齐

#### 技术问题

**渲染**：
- [ ] 字体缺失（被替换）
- [ ] 特殊字符未显示
- [ ] 公式渲染错误
- [ ] 图像损坏或缺失
- [ ] 色彩偏差（RGB与CMYK）

**一致性**：
- [ ] 页码错误或缺失
- [ ] 页眉/页脚不一致
- [ ] 导航元素损坏
- [ ] 超链接失效（交互测试时）

## 文档记录模板

### 问题日志格式

创建电子表格或文档跟踪所有问题：

```
页码 | 问题类型 | 描述 | 严重性 | 状态
------|---------|------|--------|-----
3     | 文本溢出 | 项目符号4超出文本框 | 高 | 已修复
7     | 元素重叠 | 图表与标题重叠 | 高 | 已修复
12    | 字体大小 | 坐标轴标签过小 | 中 | 已修复
15    | 对齐问题 | 标题未居中 | 低 | 已修复
22    | 对比度   | 白底黄字 | 高 | 已修复
```

**严重性等级**：
- **严重**：导致幻灯片无法使用或不专业
- **高**：显著影响可读性或外观
- **中**：可察觉但不妨碍理解
- **低**：轻微外观问题

### 问题记录示例

**规范记录**：
```
第8页：文本溢出问题
- 描述：末项项目符号"...实现细节"超出文本框右边界约0.5英寸
- 原因：项目文本过长超出可用宽度
- 修复：缩减文本至"...实现"或增加文本框宽度
- 验证：检查相邻幻灯片是否存在类似问题
```

**不规范记录**：
```
第8页：文本问题
- 修复：缩小文本
```

## 常见问题与解决方案

### 问题1：文本溢出

**现象**：文本超出边界

**识别特征**：
- 边缘可见截断文本
- 文本侵入页边距
- 部分字符可见

**解决方案**：

**LaTeX Beamer**：
```latex
% 精简文本
\begin{frame}{标题}
  \begin{itemize}
    \item 缩短过长项目文本
    % 或
    \item 使用缩写或首字母缩略词
    % 或
    \item<alert@1> 拆分为多个项目
  \end{itemize}
\end{frame}

% 调整页边距
\newgeometry{margin=1.5cm}
\begin{frame}
  宽边距内容
\end{frame}
\restoregeometry

% 特定元素缩小字体
{\small
  需适配的长文本
}
```

**PowerPoint**：
- 缩小元素字体
- 精简文本内容
- 扩大文本框
- 谨慎使用文本框自动适配
- 拆分为多张幻灯片

### 问题2：元素重叠

**现象**：元素不当重叠

**识别特征**：
- 图像遮挡文本
- 形状覆盖文字
- 图形相互重叠

**解决方案**：

**LaTeX Beamer**：
```latex
% 分栏布局隔离元素
\begin{columns}
  \begin{column}{0.5\textwidth}
    文本内容
  \end{column}
  \begin{column}{0.5\textwidth}
    \includegraphics[width=\textwidth]{figure.pdf}
  \end{column}
\end{columns}

% 增加间距
\vspace{0.5cm}

% 调整图形尺寸
\includegraphics[width=0.7\textwidth]{figure.pdf}
```

**PowerPoint**：
- 使用参考线重定位
- 缩小元素尺寸
- 采用双栏布局
- 调整元素前后层叠顺序
- 增加元素间距

### 问题3：对比度不足

**现象**：色彩选择导致文本难以阅读

**识别特征**：
- 需眯眼阅读文本
- 文本融入背景
- 色彩过于相近

**解决方案**：

**LaTeX Beamer**：
```latex
% 增强对比度
\setbeamercolor{frametitle}{fg=black,bg=white}
\setbeamercolor{normal text}{fg=black,bg=white}

% 使用深色系
\definecolor{darkblue}{RGB}{0,50,100}
\setbeamercolor{structure}{fg=darkblue}

% 灰度测试
\usepackage{xcolor}
\selectcolormodel{gray}  % 临时测试用
```

**PowerPoint**：
- 选择高对比度配色
- 采用深色文本配浅背景或反之
- 避免文本使用浅色系
- 使用WebAIM对比度检查器
- 必要时添加文本背景框

### 问题4：字体过小

**现象**：远距离无法阅读文本

**识别特征**：
- 3英尺外无法辨识
- 正常观看时坐标标签消失
- 标题文字模糊

**解决方案**：

**LaTeX Beamer**：
```latex
% 增大基准字号
\documentclass[14pt]{beamer}  % 替代默认11pt

% 用大号字体重制图形
% Matplotlib中：
plt.rcParams['font.size'] = 18
plt.rcParams['axes.labelsize'] = 20

% R/ggplot2中：
theme_set(theme_minimal(base_size = 16))
```

**PowerPoint**：
- 正文最小18pt（推荐24pt）
- 用大标签重制图形
- 采用直接标注替代图例
- 简化复杂图形
- 拆分密集内容至多页

### 问题5：未对齐

**现象**：元素未正确对齐

**识别特征**：
- 页边距不均
- 标题位置不一致
- 间距不规则

**解决方案**：

**LaTeX Beamer**：
```latex
% 使用统一模板
\setbeamertemplate{frametitle}[default][center]

% 顶部对齐分栏
\begin{columns}[T]  % T=顶部对齐
  \begin{column}{0.5\textwidth}
    内容
  \end{column}
  \begin{column}{0.5\textwidth}
    内容
  \end{column}
\end{columns}

% 居中图形
\begin{center}
  \includegraphics[width=0.8\textwidth]{figure.pdf}
\end{center}
```

**PowerPoint**：
- 使用对齐工具（左/中/右对齐）
- 启用网格线和参考线
- 使用贴齐网格
- 均匀分布对象
- 创建统一版式的母版

## 迭代优化流程

### 工作流循环

```
1. 生成PDF
    ↓
2. 转换为图像
    ↓
3. 系统化视觉检查
    ↓
4. 记录问题
    ↓
5. 优先级排序
    ↓
6. 在源文件中修复
    ↓
7. 重新生成PDF
    ↓
8. 再次检查（返回第2步）
    ↓
9. 无严重问题时完成
```

### 优先级策略

**立即修复**（阻碍演示）：
- 导致内容无法阅读的文本溢出
- 遮挡数据的关键元素重叠
- 图形损坏或内容缺失
- 严重对比度问题

**演示前修复**：
- 字体过小
- 中度对齐问题
- 间距不一致
- 中度对比度问题

**时间允许时修复**：
- 轻微未对齐
- 小范围间距不一致
- 外观优化
- 非关键色彩调整

### 终止标准

**最低标准**：
- [ ] 无文本溢出或截断
- [ ] 无遮挡内容的元素重叠
- [ ] 所有文本满足最小18pt等效可读
- [ ] 足够对比度（最小4.5:1比率）
- [ ] 图形图像正确显示
- [ ] 幻灯片结构统一

**理想标准**：
- [ ] 整体呈现专业外观
- [ ] 对齐与间距统一
- [ ] 高对比

```python
img = Image.open(image_path).convert('L')  # 转换为灰度图像
    arr = np.array(img)
    
    # 检查边缘（10像素边界）
    left_edge = arr[:, :threshold]
    right_edge = arr[:, -threshold:]
    top_edge = arr[:threshold, :]
    bottom_edge = arr[-threshold:, :]
    
    # 检测非白色像素（内容）
    white_threshold = 240
    
    issues = []
    if np.any(left_edge < white_threshold):
        issues.append("左侧边缘")
    if np.any(right_edge < white_threshold):
        issues.append("右侧边缘")
    if np.any(top_edge < white_threshold):
        issues.append("顶部边缘")
    if np.any(bottom_edge < white_threshold):
        issues.append("底部边缘")
    
    return issues

# 使用示例
for slide_num in range(1, 26):
    issues = detect_edge_content(f'slide-{slide_num}.jpg')
    if issues:
        print(f"幻灯片 {slide_num}: 内容靠近 {', '.join(issues)}")
```

### 对比度检查

```python
from PIL import Image
import numpy as np

def check_contrast(image_path):
    """
    估算图像对比度。
    简易版：比较最亮和最暗区域。
    """
    img = Image.open(image_path).convert('L')
    arr = np.array(img)
    
    # 获取亮度值
    bright = np.percentile(arr, 95)
    dark = np.percentile(arr, 5)
    
    # 粗略对比度计算
    contrast = (bright + 0.05) / (dark + 0.05)
    
    if contrast < 4.5:
        return f"低对比度: {contrast:.1f}:1 (最低要求 4.5:1)"
    return f"正常: {contrast:.1f}:1"

# 使用示例
for slide_num in range(1, 26):
    result = check_contrast(f'slide-{slide_num}.jpg')
    print(f"幻灯片 {slide_num}: {result}")
```

## 人工审查最佳实践

### 审查环境

**设置要求**:
- 大尺寸显示器或双显示器
- 良好光照（避免过亮或过暗）
- 无干扰环境
- 支持缩放功能的图像查看器
- 用于记录问题的记事本或电子表格

**查看选项**:
- 100%比例查看细节
- 50%比例模拟远距离观看
- 顺序查看确保一致性
- 并排对比相似幻灯片

### 审查技巧

**保持新鲜视角**:
- 每15-20张幻灯片休息一次
- 在不同时段进行审查
- 请同事交叉审查
- 隔日进行最终复核

**系统化方法**:
- 按顺序审查（从第1张至末尾）
- 每次专注一类问题
- 使用检查清单确保全面性
- 实时记录而非凭记忆

**常见疏漏**:
- 备份幻灯片（这些也要查！）
- 标题页（第一印象至关重要）
- 致谢页（常被遗忘）
- 末页（问答环节持续显示）

## 工具与资源

### 推荐软件

**PDF转图像工具**:
- **PyMuPDF** (Python): 快速，无外部依赖（推荐）
- **pdf_to_images.py脚本**: 命令行简易封装
- **ImageMagick**: 功能灵活（可选）

**图像查看工具**:
- **IrfanView** (Windows): 快速，多格式支持
- **预览** (macOS): 系统内置
- **Eye of GNOME** (Linux): 轻量级
- **XnView**: 跨平台，支持批量操作

**问题追踪工具**:
- **电子表格** (Excel, Google Sheets): 简单灵活
- **Markdown文件**: 兼容版本控制
- **问题追踪系统** (GitHub, Jira): 团队协作适用
- **检查清单应用**: 移动端审查

### 对比度检测工具

- **WebAIM对比度检查器**: https://webaim.org/resources/contrastchecker/
- **色彩对比分析器**: 桌面应用
- **Chrome开发者工具**: 内置对比度检查

### 色盲模拟工具

- **Coblis**: https://www.color-blindness.com/coblis-color-blindness-simulator/
- **Color Oracle**: 免费桌面应用
- **Photoshop/GIMP**: 内置色盲滤镜

## 最终检查清单

定稿前确认：

**转换环节**:
- [ ] PDF以足够分辨率转图像（150-300 DPI）
- [ ] 包含所有幻灯片（含备份页）
- [ ] 图像保存至有序目录

**视觉审查**:
- [ ] 系统化审查所有幻灯片
- [ ] 每页完成问题检查清单
- [ ] 标注问题及对应页码
- [ ] 为每个问题分配严重等级

**问题修复**:
- [ ] 紧急问题已修正
- [ ] 高优先级问题已处理
- [ ] 更新源文件（非仅PDF）
- [ ] 重新生成并二次审查

**最终验证**:
- [ ] 无文本溢出或截断
- [ ] 无元素异常重叠
- [ ] 整体对比度达标
- [ ] 布局与间距统一
- [ ] 呈现专业外观
- [ ] 满足投影/分发要求

**测试环节**:
- [ ] 实际投影测试（如可行）
- [ ] 模拟后排距离查看
- [ ] 多光照环境测试
- [ ] 备份副本已保存
