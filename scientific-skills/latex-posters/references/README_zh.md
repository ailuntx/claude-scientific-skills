# LaTeX 研究海报生成技能

使用 LaTeX 创建专业、可直接发表的会议及学术展示研究海报。

## 概述

本技能提供使用三大 LaTeX 包创建研究海报的完整指南：
- **beamerposter**：传统学术海报，熟悉的 Beamer 语法
- **tikzposter**：集成 TikZ 的现代多彩设计
- **baposter**：自动定位的结构化多栏布局

## 快速入门

### 1. 选择模板

浏览 `assets/` 中的模板：
- `beamerposter_template.tex` - 经典学术风格
- `tikzposter_template.tex` - 现代多彩设计
- `baposter_template.tex` - 结构化多栏布局

### 2. 自定义内容

编辑模板添加研究内容：
- 标题、作者、所属机构
- 引言、方法、结果、结论
- 用实际图片替换占位图
- 更新参考文献和致谢

### 3. 全页配置

海报应占满整页并最小化页边距：

```latex
% beamerposter - 全页设置
\documentclass[final,t]{beamer}
\usepackage[size=a0,scale=1.4,orientation=portrait]{beamerposter}
\setbeamersize{text margin left=5mm, text margin right=5mm}
\usepackage[margin=10mm]{geometry}

% tikzposter - 全页设置
\documentclass[25pt,a0paper,portrait,margin=10mm,innermargin=15mm]{tikzposter}

% baposter - 全页设置
\documentclass[a0paper,portrait,fontscale=0.285]{baposter}
```

### 4. 编译

```bash
pdflatex poster.tex

# 或获得更好字体支持：
lualatex poster.tex
xelatex poster.tex
```

### 5. 检查 PDF 质量

**打印前必须执行！**

```bash
# 运行自动检查
./scripts/review_poster.sh poster.pdf

# 人工验证（见下方检查清单）
```

## 核心功能

### 全页覆盖

所有模板均配置为最大化内容区域：
- 最小外页边距 (5-15mm)
- 最佳栏间距 (15-20mm)
- 合理区块内边距确保可读性
- 无多余留白

### PDF 质量控制

**自动检查** (`review_poster.sh`)：
- 页面尺寸验证
- 字体嵌入检查
- 图像分辨率分析
- 文件大小优化

**人工验证** (`assets/poster_quality_checklist.md`)：
- 100% 缩放视觉检查
- 缩印测试 (25%)
- 排版与间距审查
- 内容完整性检查

### 设计原则

所有模板遵循循证海报设计：
- **排版**：标题72pt+，标题48-72pt，正文24-36pt
- **色彩**：高对比度(≥4.5:1)，色盲友好调色板
- **布局**：清晰视觉层次，逻辑流
- **内容**：300-800字上限，40-50%视觉内容

## 常用海报尺寸

模板支持所有标准尺寸：

| 尺寸 | 规格 | 配置 |
|------|------------|---------------|
| A0 | 841 × 1189 mm | `size=a0` 或 `a0paper` |
| A1 | 594 × 841 mm | `size=a1` 或 `a1paper` |
| 36×48" | 914 × 1219 mm | 自定义页面尺寸 |
| 42×56" | 1067 × 1422 mm | 自定义页面尺寸 |

## 文档

### 参考指南

**完整文档** (位于 `references/`)：

1. **`latex_poster_packages.md`** (746行)
   - beamerposter/tikzposter/baposter 详细对比
   - 包特定语法与示例
   - 优势/局限/最佳用例
   - 主题与色彩定制
   - 编译技巧与故障排除

2. **`poster_design_principles.md`** (807行)
   - 视觉层次与留白
   - 排版：字体选择/尺寸/可读性
   - 色彩理论：方案/对比度/无障碍
   - 色盲友好调色板
   - 图标/图形/视觉元素
   - 常见设计错误规避

3. **`poster_layout_design.md`** (650+行)
   - 网格系统(2/3/4栏布局)
   - 视觉流与阅读模式
   - 空间组织策略
   - 留白管理
   - 区块与框体设计
   - 按研究类型分类的布局模式

4. **`poster_content_guide.md`** (900+行)
   - 内容策略(3-5分钟规则)
   - 各章节字数预算
   - 图文比例(40-50%视觉)
   - 章节写作指南
   - 图表整合与标题
   - 从论文到海报的转化

### 工具与资源

**脚本** (位于 `scripts/`)：
- `review_poster.sh`：自动PDF质量检查
  - 页面尺寸验证
  - 字体嵌入检查
  - 图像分辨率分析
  - 文件大小评估

**检查清单** (位于 `assets/`)：
- `poster_quality_checklist.md`：完整打印前清单
  - 编译前检查项
  - PDF质量验证
  - 视觉检查条目
  - 无障碍检查
  - 同行评审指南
  - 最终打印清单

**模板** (位于 `assets/`)：
- `beamerposter_template.tex`：完整可用模板
- `tikzposter_template.tex`：完整可用模板
- `baposter_template.tex`：完整可用模板

## 工作流程

### 推荐海报创建流程

**1. 规划** (LaTeX前)
- 确定会议要求(尺寸/方向)
- 明确3-5个核心成果
- 创建图表(300+ DPI)
- 起草300-800字内容大纲

**2. 模板选择**
- 按需选择包：
  - **beamerposter**：传统会议/机构品牌
  - **tikzposter**：现代会议/创意领域
  - **baposter**：多章节海报/结构化布局

**3. 内容整合**
- 复制模板并自定义
- 替换占位文本
- 添加高分辨率图表
- 配置色彩匹配品牌

**4. 编译与审查**
- 编译为PDF
- 运行 `review_poster.sh` 自动检查
- 100%缩放视觉审查
- 对照 `poster_quality_checklist.md` 检查

**5. 测试打印**
- **关键步骤！** 25%比例打印
- A0→A4纸，36×48"→Letter纸
- 2-3英尺距离查看(模拟8-12英尺全尺寸效果)
- 验证可读性与色彩

**6. 修订**
- 修复发现的问题
- 仔细校对(错误会被放大！)
- 获取同行反馈
- 最终编译

**7. 打印**
- 验证页面尺寸：`pdfinfo poster.pdf`
- 检查字体嵌入：`pdffonts poster.pdf`
- 截止前2-3天发送专业打印
- 保留备份副本

## 故障排除

### 过大白边

**问题**：海报边缘留白过多

**解决方案**：
```latex
% beamerposter
\setbeamersize{text margin left=5mm, text margin right=5mm}
\usepackage[margin=10mm]{geometry}

% tikzposter
\documentclass[..., margin=5mm, innermargin=10mm]{tikzposter}

% baposter
\documentclass[a0paper, margin=5mm]{baposter}
```

### 内容截断

**问题**：文本或图表超出页面

**解决方案**：
- 检查总宽度：栏宽+间距+页边距=页面宽度
- 减小栏宽或间距
- 启用可视页面边界调试：
```latex
\usepackage{eso-pic}
\AddToShipoutPictureBG{
  \AtPageLowerLeft{
    \put(0,0){\framebox(\LenToUnit{\paperwidth},\LenToUnit{\paperheight}){}}
  }
}
```

### 图像模糊

**问题**：图片像素化或低质量

**解决方案**：
- 优先使用矢量图(PDF/SVG)
- 位图：最终打印尺寸下至少300 DPI
- A0宽度(33.1英寸)：300 DPI=最小9930像素
- 检查命令：`pdfimages -list poster.pdf`

### 字体未嵌入

**问题**：打印机因缺失字体拒绝PDF

**解决方案**：
```bash
# 重新编译嵌入字体
pdflatex -dEmbedAllFonts=true poster.tex

# 验证嵌入
pdffonts poster.pdf
# "emb"列应全为"yes"
```

### 文件过大

**问题**：PDF超过邮件限制(>50MB)

**解决方案**：
```bash
# 数字分享压缩
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 \
   -dPDFSETTINGS=/printer -dNOPAUSE -dQUIET -dBATCH \
   -sOutputFile=poster_compressed.pdf poster.pdf

# 保留原始未压缩版本用于打印
```

## 常见错误规避

### 内容
- ❌ 文字过多(>1000字)
- ❌ 字号过小(正文<24pt)
- ❌ 无明确核心信息
- ✅ 300-800字，30pt+正文，1-3个核心发现

### 设计
- ❌ 低色彩对比度(<4.5:1)
- ❌ 红绿配色(色盲问题)
- ❌ 布局混乱无留白
- ✅ 高对比度/无障碍色彩/充足间距

### 技术
- ❌ 错误海报尺寸
- ❌ 低分辨率图像(<300 DPI)
- ❌ 字体未嵌入
- ✅ 验证规格/高分辨率图/嵌入字体

## 包对比

快速选择参考：

| 特性 | beamerposter | tikzposter | baposter |
|---------|--------------|------------|----------|
| **学习曲线** | 简单(Beamer用户) | 中等 | 中等 |
| **美观度** | 传统 | 现代 | 专业 |
| **定制性** | 中等 | 高(TikZ) | 结构化 |
| **编译速度** | 快 | 较慢 | 中快 |
| **最佳适用** | 学术会议 | 创意设计 | 多栏布局 |

**推荐**：
- 首次制作：**beamerposter**(熟悉/简单)
- 现代会议：**tikzposter**(美观/灵活)
- 复杂布局：**baposter**(自动定位)

## 使用示例

### 科学写作 CLI 中

```
> 为NeurIPS会议创建关于Transformer注意力的研究海报

助手将：
1. 询问海报尺寸与方向
2. 生成完整LaTeX海报
3. 配置全页覆盖
4. 提供编译指南
5. 对生成PDF执行质量检查
```

### 手动创建

```bash
# 1. 复制模板
cp assets/tikzposter_template.tex my_poster.tex

# 2. 编辑内容
vim my_poster.tex

# 3. 编译
pdflatex my_poster.tex

# 4. 审查
./scripts/review_poster.sh my_poster.pdf

# 5. 25%比例测试打印
# (A0印于A4纸)

# 6. 最终打印
```

## 成功要诀

### 内容策略
1. **单一核心信息**：观众应记住什么？
2. **3-5个关键图表**：视觉内容主导
3. **300-800字**：少即是多
4. **要点列表**：比段落更易浏览

### 设计策略
1. **高对比度**：深底浅字或浅底深字
2. **大字号**：30pt+正文确保远距可读
3. **留白**：海报30-40%应为空白
4. **视觉层次**：显著尺寸变化(标题3倍正文)

### 技术策略
1. **及早测试**：最终打印前先缩印
2. **矢量图形**：优先使用PDF/SVG
3. **验证规格**：检查页面尺寸/字体/分辨率
4. **获取反馈**：打印前请同行评审

## 附加资源

### 在线工具
- **色彩对比检测**：https://webaim.org/resources/contrastchecker/
- **色盲模拟器**：https://www.color-blindness.com/coblis-color-blindness-simulator/
- **调色板生成器**：https://coolors.co/

### LaTeX 包
- `beamerposter`：扩展Beamer支持海报尺寸
- `tikzposter`：基于TikZ的现代海报创建
- `baposter`：基于框体的自动海报布局
- `qrcode`：LaTeX内生成二维码
- `graphicx`：图像导入
- `tcolorbox`：彩色框体与框架

### 延伸阅读
- `references/` 目录内所有文档
- 质量清单：`assets/poster_quality_checklist.md`
- 包对比：`references/latex_poster_packages.md`

## 支持

问题咨询：
- 查阅 `references/` 参考文档
- 参考上方故障排除章节
- 运行自动审查：`./scripts/review_poster.sh`
- 使用质量清单：`assets/poster_quality_checklist.md`

## 版本

LaTeX 海报技能 v1.0
兼容：beamerposter/tikzposter/baposter
最后更新：2025年1月
