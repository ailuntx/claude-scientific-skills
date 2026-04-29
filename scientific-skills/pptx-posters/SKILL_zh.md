---
name: pptx-posters
description: 使用HTML/CSS创建可导出为PDF或PPTX的研究海报。仅在用户明确要求PowerPoint/PPTX海报格式时使用此技能。标准研究海报请改用latex-posters技能。本技能提供基于现代网页技术的海报设计，支持响应式布局和可视化元素集成。
allowed-tools: Read Write Edit Bash
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# PPTX研究海报（基于HTML）

## 概述

**⚠️ 仅当用户明确要求PPTX/PowerPoint海报格式时使用此技能。**

标准研究海报请改用**latex-posters**技能，该技能提供更优的排版控制能力，是学术会议的默认选择。

本技能通过HTML/CSS创建研究海报，可导出为PDF或转换为PowerPoint格式。基于网页的方案提供：
- 现代化响应式布局
- 轻松集成AI生成的可视化元素
- 浏览器内快速迭代预览
- 通过浏览器打印功能导出PDF
- 按需转换为PPTX格式

## 使用场景

**仅在以下情况使用本技能：**
- 用户明确要求"PPTX海报"、"PowerPoint海报"或"PPT海报"
- 用户特别要求基于HTML的海报
- 用户需要在PowerPoint中编辑海报
- LaTeX不可用或用户要求非LaTeX方案

**禁止使用本技能的情况：**
- 用户仅要求"海报"未指定格式 → 使用latex-posters
- 用户要求"研究海报"或"会议海报" → 使用latex-posters
- 用户提及LaTeX、tikzposter、beamerposter或baposter → 使用latex-posters

## AI驱动的可视化元素生成

**标准工作流：创建HTML海报前必须用AI生成所有主要可视化元素**

推荐流程用于创建视觉冲击力强的海报：
1. 规划所需视觉元素（主视觉图、引言、方法、结果、结论）
2. 使用scientific-schematics或Nano Banana Pro生成每个元素
3. 在HTML模板中组合生成的图像
4. 围绕视觉元素添加文本内容

**目标：60-70%海报区域应为AI生成的可视化元素，30-40%为文本**

---

### 关键要求：海报尺寸字体规范

**⚠️ AI生成可视化元素中的所有文本必须满足海报可读性**

生成海报图形时，必须在每个提示词中包含字体大小要求。海报需在4-6英尺外观看，因此文本必须足够大。

**所有海报图形提示词强制要求：**

```
海报格式要求（严格执行）：
- 每图最多3-4个元素（理想为3个）
- 全图总字数不超过10个
- 禁止5+步骤的复杂流程图（拆分为2-3个简单图形）
- 禁止多层嵌套图表（简化为单层）
- 禁止含多子章节的案例研究（每案例仅展示一个关键点）
- 所有文本使用巨型粗体（标签80pt+，关键数字120pt+）
- 仅使用高对比度（白底黑字或黑底白字，禁止渐变文字）
- 强制要求至少50%留白（图形一半区域应为空白）
- 仅使用粗线条（最小5px+），大图标（最小200px+）
- 每图仅传递单一核心信息（非三个关联信息）
```

**⚠️ 生成前检查提示词：**
- 若描述含5+项 → 停止，拆分为多个图形
- 若工作流含5+阶段 → 停止，仅展示3-4个高层步骤
- 若对比含4+方法 → 停止，仅展示前三名或"我方 vs 最佳基线"

**错误示例（7阶段工作流）：**
```bash
# ❌ 生成无法阅读的小字
python scripts/generate_schematic.py "药物研发流程：阶段1靶点识别、阶段2合成、阶段3筛选、阶段4先导优化、阶段5验证、阶段6临床试验、阶段7FDA审批含指标" -o figures/workflow.png
```

**正确示例（3大阶段）：**
```bash
# ✅ 相同内容简化为可读的海报格式
python scripts/generate_schematic.py "A0海报格式。超简3框工作流：'发现'→'验证'→'批准'。每个词用巨型粗体(120pt+)。粗箭头(10px)。60%留白。仅含这三个词。无子步骤。12英尺外可读" -o figures/workflow_simple.png
```

---

### 关键要求：防止内容溢出

**⚠️ 海报边缘禁止出现文本或内容截断**

**预防规则：**

**1. 限制内容分区（最多5-6区）：**
```
✅ 合理 - 5个分区留白充足：
   - 标题/页眉
   - 引言/问题
   - 方法
   - 结果（1-2个关键发现）
   - 结论

❌ 错误 - 8+分区拥挤堆叠
```

**2. 字数限制：**
- **每分区**：最多50-100词
- **全海报**：最多300-800词
- **超量内容**：删减或制作讲义

---

## 核心功能

### 1. HTML/CSS海报设计

HTML模板(`assets/poster_html_template.html`)提供：
- 固定海报尺寸（36×48英寸 = 2592×3456 pt）
- 渐变风格专业页眉
- 三栏内容布局
- 模块化分区现代样式
- 含参考文献和联系信息的页脚

### 2. 海报结构

**标准布局：**
```
┌─────────────────────────────────────────┐
│  页眉：标题、作者、主视觉图              │
├─────────────┬─────────────┬─────────────┤
│  引言       │   结果      │   讨论      │
│             │             │             │
│  方法       │  (图表)     │   结论      │
│             │             │             │
│  (示意图)   │  (数据)     │  (总结)     │
├─────────────┴─────────────┴─────────────┤
│  页脚：参考文献和联系信息               │
└─────────────────────────────────────────┘
```

### 3. 视觉元素集成

每个分区应突出展示AI生成的视觉元素：

**主视觉图（页眉）：**
```html
<img src="figures/hero.png" class="hero-image">
```

**分区图形：**
```html
<div class="block">
  <h2 class="block-title">方法</h2>
  <div class="block-content">
    <img src="figures/workflow.png" class="block-image">
    <ul>
      <li>简要方法说明</li>
    </ul>
  </div>
</div>
```

### 4. 生成视觉元素

**创建HTML前生成所有视觉元素：**

```bash
# 创建图形目录
mkdir -p figures

# 主视觉图 - 简洁有力
python scripts/generate_schematic.py "A0海报格式。主视觉横幅：'[主题]'超大文本(120pt+)。深蓝渐变背景。单一标志性视觉。极简文本。15英尺外可读" -o figures/hero.png

# 引言视觉 - 仅3个元素
python scripts/generate_schematic.py "A0海报格式。仅含3个图标的简易视觉：[图标1]→[图标2]→[图标3]。单字标签(80pt+)。50%留白。8英尺外可读" -o figures/intro.png

# 方法流程图 - 仅4步骤
python scripts/generate_schematic.py "A0海报格式。仅含4个框的简易流程图：步骤1→步骤2→步骤3→步骤4。巨型标签(100pt+)。粗箭头。50%留白。无子步骤" -o figures/workflow.png

# 结果可视化 - 仅3个柱状
python scripts/generate_schematic.py "A0海报格式。仅含3个柱的简易柱状图：基线(70%)、现有(85%)、我方(95%)。柱上显示巨型百分比(120pt+)。无坐标轴/图例。50%留白" -o figures/results.png

# 结论 - 精确3个关键发现
python scripts/generate_schematic.py "A0海报格式。精确3个卡片：'95%'(150pt)'准确率'(60pt)、'2X'(150pt)'更快'(60pt)、勾号'就绪'(60pt)。50%留白。无其他文本" -o figures/conclusions.png
```

---

## PPTX海报创建工作流

### 阶段1：规划

1. **确认用户明确要求PPTX**
2. **确定海报要求：**
   - 尺寸：36×48英寸（最常见）或A0
   - 方向：纵向（最常见）
3. **制定内容大纲：**
   - 明确1-3个核心信息
   - 规划3-5个视觉元素
   - 起草精简文本（总计300-800词）

### 阶段2：生成视觉元素（AI驱动）

**关键：生成内容极简的简易图形**

```bash
mkdir -p figures

# 按海报格式规范生成每个元素
# (参考上文第4节示例)
```

### 阶段3：创建HTML海报

1. **复制模板：**
   ```bash
   cp skills/pptx-posters/assets/poster_html_template.html poster.html
   ```

2. **更新内容：**
   - 替换占位标题和作者
   - 插入AI生成的图像
   - 添加精简辅助文本
   - 更新参考文献和联系信息

3. **浏览器预览：**
   ```bash
   open poster.html  # macOS
   # 或
   xdg-open poster.html  # Linux
   ```

### 阶段4：导出PDF

**浏览器打印法：**
1. 在Chrome/Firefox中打开poster.html
2. 打印(Cmd/Ctrl + P)
3. 选择"另存为PDF"
4. 设置纸张尺寸匹配海报
5. 取消页边距
6. 启用"背景图形"

**命令行（需安装Chrome）：**
```bash
# Chrome无头模式导出PDF
google-chrome --headless --print-to-pdf=poster.pdf \
  --print-to-pdf-no-header \
  --no-margins \
  poster.html
```

### 阶段5：转换为PPTX（如需）

**方案1：PDF转PPTX**
```bash
# 使用LibreOffice
libreoffice --headless --convert-to pptx poster.pdf

# 或使用在线转换工具处理简单情况
```

**方案2：python-pptx直接创建**
```python
from pptx import Presentation
from pptx.util import Inches, Pt

prs = Presentation()
prs.slide_width = Inches(48)
prs.slide_height = Inches(36)

slide = prs.slides.add_slide(prs.slide_layouts[6])  # 空白布局

# 从figures/添加图像
slide.shapes.add_picture("figures/hero.png", Inches(0), Inches(0), width=Inches(48))
# ...添加其他元素

prs.save("poster.pptx")
```

---

## HTML模板结构

模板文件(`assets/poster_html_template.html`)包含：

### 可定制的CSS变量

```css
/* 海报尺寸 */
body {
  width: 2592pt;   /* 36英寸 */
  height: 3456pt;  /* 48英寸 */
}

/* 配色方案 - 可自定义 */
.header {
  background: linear-gradient(135deg, #1a365d 0%, #2b6cb0 50%, #3182ce 100%);
}

/* 排版 */
.poster-title { font-size: 108pt; }
.authors { font-size: 48pt; }
.block-title { font-size: 52pt; }
.block-content { font-size: 34pt; }
```

### 关键样式类

| 类名 | 用途 | 字号 |
|-------|---------|-----------|
| `.poster-title` | 主标题 | 108pt |
| `.authors` | 作者姓名 | 48pt |
| `.affiliations` | 机构名称 | 38pt |
| `.block-title` | 分区标题 | 52pt |
| `.block-content` | 正文 | 34pt |
| `.key-finding` | 高亮框 | 36pt |

---

## 质量检查清单

### 步骤0：生成前审查（强制）

**对每个规划图形验证：**
- [ ] 能否用3-4个元素描述？（非5+）
- [ ] 是否为简易工作流（3-4步，非7+）？
- [ ] 所有文本能否在10词内描述？
- [ ] 是否传达单一信息（非多个）？

**拒绝以下模式：**
- ❌ "7阶段工作流" → 简化为"3大阶段"
- ❌ "多案例研究" → 每图仅一个案例
- ❌ "2015-2024年度时间线" → "仅3个关键年份"
- ❌ "对比6种方法" → "仅2种：我方 vs 最佳方案"

### 步骤2b：生成后审查（强制）

**在25%缩放比例下检查每个生成图形：**

**✅ 通过标准（需全部满足）：**
- [ ] 所有文本清晰可读
- [ ] 元素数量≤3-4个
- [ ] 留白≥50%
- [ ] 2秒内可理解
- [ ] 非5+阶段的复杂工作流
- [ ] 无多层嵌套分区

**❌ 失败标准（任意一项即需重新生成）：**
- [ ] 文本过小难读 → 用"150pt+"重新生成
- [ ] 超4个元素 → 用"仅3元素"重新生成
- [ ] 留白不足50% → 用"60%留白"重新生成
- [ ] 复杂多阶段 → 拆分为2-3个图形
- [ ] 多案例拥挤 → 拆分为独立图形

### 导出后检查

- [ ] 四边无任何内容截断（仔细检查）
- [ ] 所有图像正常显示
- [ ] 色彩渲染符合预期
- [ ] 25%缩放下文本可读
- [ ] 图形保持简洁（非复杂7阶段工作流样式）

---

## 常见陷阱规避

**AI生成图形错误：**
- ❌ 元素过多（10+项） → 限制在3-5项
- ❌ 文本过小 → 提示词中指定"巨型(100pt+)"
- ❌ 无留白 → 每个提示词添加"50%留白"
- ❌ 复杂流程图（8+步骤） → 限制在4-5步

**HTML/导出错误：**
- ❌ 内容超出海报尺寸 → 检查浏览器溢出
- ❌ PDF缺失背景图形 → 打印设置中启用
- ❌ PDF纸张尺寸错误 → 精确匹配海报尺寸
- ❌ 图像分辨率过低 → 最低300 DPI

**内容错误：**
- ❌ 文本过多（超1000词） → 精简至300-800词
- ❌ 分区过多（7+） → 整合为5-6区
- ❌ 缺乏视觉层次 → 突出关键发现

---

## 与其他技能的协同

本技能可与
