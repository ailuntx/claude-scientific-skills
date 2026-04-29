# 科学演示文稿的PowerPoint设计指南

## 概述

本指南提供使用PowerPoint创建专业科学演示文稿的全面指导，重点涵盖与pptx技能的编程化集成方法及科学内容的最佳实践。

**关键提示**：避免枯燥、文字密集的演示。科学幻灯片应具备：
- **视觉吸引力**：每张幻灯片都需包含高质量图片、图表和示意图
- **研究支撑**：通过research-lookup引用文献增强可信度（至少8-15篇论文）
- **现代设计**：采用当代配色方案，避免默认主题
- **精简文本**：每页3-4个要点，每点4-6个词，让视觉元素主导呈现
- **专业润色**：布局统一而富有变化，留白充足

**反面模式警告**：白底黑字的纯要点幻灯片 = 即刻引发倦怠并被遗忘的科学内容。

## 使用PPTX技能

### 参考文档

完整PowerPoint创建技术文档请参阅：
- **主文档**：`document-skills/pptx/SKILL.md`
- **HTML转PPT工作流**：详见`pptx/html2pptx.md`
- **OOXML编辑**：高级编辑指南见`pptx/ooxml.md`

### 两种PowerPoint创建方法

#### 1. 编程化创建 (html2pptx)

**最适用场景**：从零开始创建自定义设计和数据可视化的演示文稿。

**工作流程**：
1. 完整阅读`document-skills/pptx/SKILL.md`
2. 按标准尺寸设计HTML幻灯片（16:9比例使用720pt×405pt）
3. 使用`html2pptx()`函数创建JavaScript文件
4. 通过PptxGenJS API添加图表和表格
5. 生成缩略图并进行视觉验证
6. 根据视觉检查结果迭代优化

**示例结构**：
```javascript
const pptx = new PptxGenJS();

// 添加标题幻灯片
const slide1 = pptx.addSlide();
slide1.addText("您的标题", {
  x: 1, y: 2, w: 8, h: 1,
  fontSize: 44, bold: true, align: "center"
});

// 添加带图表的幻灯片
const slide2 = pptx.addSlide();
slide2.addText("结果", { x: 0.5, y: 0.5, fontSize: 32 });
slide2.addImage({ path: "figure.png", x: 1, y: 1.5, w: 8, h: 4 });

pptx.writeFile({ fileName: "presentation.pptx" });
```

#### 2. 模板化创建

**最适用场景**：使用现有PowerPoint模板或编辑已有演示文稿。

**工作流程**：
1. 从template.pptx开始
2. 使用`scripts/rearrange.py`复制/重排幻灯片
3. 使用`scripts/inventory.py`提取文本
4. 生成替换文本JSON
5. 使用`scripts/replace.py`更新内容
6. 通过缩略图网格验证

**核心脚本**：
- `rearrange.py`：复制和重排幻灯片
- `inventory.py`：提取所有文本形状
- `replace.py`：应用文本替换
- `thumbnail.py`：视觉验证

## 科学演示设计原则

### 1. 布局与结构

**幻灯片母版设置**：
- 创建统一的母版幻灯片
- 定义4-5种布局类型（标题页、内容页、图表页、双栏页、结束页）
- 设置默认字体、颜色和间距
- 包含标识和页脚占位符

**标准布局**：

**标题页**：
```
┌─────────────────────────┐
│                         │
│   演示文稿标题          │
│   演讲者姓名            │
│   所属机构              │
│   日期/会议名称         │
│                         │
└─────────────────────────┘
```

**内容页**：
```
┌─────────────────────────┐
│ 幻灯片标题              │
├─────────────────────────┤
│ • 要点1                 │
│ • 要点2                 │
│ • 要点3                 │
│                         │
│ [可选图表]              │
└─────────────────────────┘
```

**双栏页**：
```
┌─────────────────────────┐
│ 幻灯片标题              │
├───────────┬─────────────┤
│           │             │
│ 文本内容  │   图表      │
│           │   或        │
│           │   数据      │
└───────────┴─────────────┘
```

**全图表页**：
```
┌─────────────────────────┐
│ 图表标题（小字号）      │
├─────────────────────────┤
│                         │
│    大型图表或           │
│    可视化元素           │
│                         │
└─────────────────────────┘
```

### 2. 排版

**字体选择**：
- **首选**：无衬线体（Arial, Calibri, Helvetica）
- **备选**：Verdana, Tahoma, Trebuchet MS
- **避免**：衬线字体（屏幕阅读困难）、装饰性字体

**字号规范**：
- 标题页主标题：44-54pt
- 幻灯片标题：32-40pt
- 正文：24-28pt（最小18pt）
- 图表标注：16-20pt
- 页脚：10-12pt

**文本格式**：
- **加粗**：用于强调（节制使用）
- **颜色**：用于突出（保持含义一致性）
- **字号**：建立层次结构
- **对齐**：正文左对齐，标题居中

**6×6法则**：
- 每页最多6个要点
- 每要点最多6个词
- 更佳方案：3-4个要点，每点4-8词

### 3. 配色方案

**选色原则**：

根据主题和受众选择：
- **学术/专业**：藏青、灰、白搭配少量强调色
- **生物医学**：蓝绿色调（避免红绿组合）
- **科技领域**：现代色系（青绿、橙、紫）
- **临床医学**：保守配色（蓝、灰、柔和的绿）

**示例方案**：

**经典科学风**：
- 背景：白 (#FFFFFF)
- 标题：藏青 (#1C3D5A)
- 文本：深灰 (#2D3748)
- 强调色：橙 (#E67E22)

**现代研究风**：
- 背景：浅灰 (#F7FAFC)
- 标题：青绿 (#0A9396)
- 文本：炭黑 (#2C2C2C)
- 强调色：珊瑚红 (#EE6C4D)

**高对比度**（适合大型会场）：
- 背景：白 (#FFFFFF)
- 标题：黑 (#000000)
- 文本：深灰 (#1A1A1A)
- 强调色：亮蓝 (#0066CC)

**无障碍指南**：
- 最小对比度：4.5:1（正文）
- 推荐对比度：7:1（AAA标准）
- 避免红绿组合（8%男性有色盲）
- 数据展示需辅以图案或形状

### 4. 视觉元素

**图表与图片**：
- **分辨率**：印刷至少300 DPI，投影至少150 DPI
- **格式**：截图用PNG，矢量图用PDF/SVG
- **尺寸**：确保后排观众可清晰阅读
- **位置**：居中或采用双栏布局

**数据可视化**：
- **简化**期刊图表（减少分块，增大文字）
- **字号**：坐标轴标签18-24pt
- **线宽**：2-4pt粗细
- **颜色**：高对比度，色盲友好
- **标注**：直接标记优于图例

**图标与形状**：
- 用于增强视觉吸引力和组织性
- 风格统一（全轮廓或全填充）
- 尺寸适中（避免过大过小）
- 限制颜色数量（匹配主题）

### 5. 动画与过渡

**适用场景**：
- ✅ 要点渐进式展开
- ✅ 复杂图表分步构建
- ✅ 强调关键发现
- ✅ 展示流程步骤

**避免场景**：
- ❌ 装饰或娱乐目的
- ❌ 每页幻灯片都用
- ❌ 干扰性效果（飞入、弹跳、旋转）

**推荐动画**：
- **出现**：简洁专业
- **淡入**：柔和过渡
- **擦除**：定向呈现
- **时长**：快速（0.2-0.3秒）
- **触发**：点击触发（非自动）

**页面过渡**：
- 全程使用统一过渡效果（或无效果）
- 推荐：无、淡入或推进
- 避免：3D旋转、复杂特效
- 时长：极快（0.3-0.5秒）

## 使用PPTX技能创建演示文稿

### 设计优先工作流

**步骤0：根据主题选择现代配色**

**关键提示**：选择反映主题的配色，而非通用默认方案。

**主题配色示例**：
- **生物技术/生命科学**：青绿 (#0A9396)、珊瑚红 (#EE6C4D)、米白 (#F4F1DE)
- **神经科学/脑研究**：深紫 (#722880)、品红 (#D72D51)、白
- **机器学习/人工智能**：亮红 (#E74C3C)、橙 (#F39C12)、深灰 (#2C2C2C)
- **物理/工程**：藏青 (#1C3D5A)、橙 (#E67E22)、浅灰 (#F7FAFC)
- **医学/健康**：青绿 (#5EA8A7)、珊瑚红 (#FE4447)、白 (#FFFFFF)
- **环境科学**：鼠尾草绿 (#87A96B)、陶土红 (#E07A5F)、米白 (#F4F1DE)

完整配色方案见pptx技能SKILL.md（76-94行）。

**步骤1：规划设计系统**（采用现代配色）
```javascript
// 使用现代配色定义设计常量（非默认值）
const DESIGN = {
  colors: {
    primary: "0A9396",    // 青绿（现代、吸睛）
    accent: "EE6C4D",     // 珊瑚红（醒目）
    text: "2C2C2C",       // 炭黑（易读）
    background: "FFFFFF"  // 白（纯净）
  },
  fonts: {
    title: { size: 40, bold: true, face: "Arial" },
    heading: { size: 28, bold: true, face: "Arial" },
    body: { size: 24, face: "Arial" },
    caption: { size: 16, face: "Arial" }
  },
  layout: {
    margin: 0.5,
    titleY: 0.5,
    contentY: 1.5
  }
};
```

**步骤2：创建复用函数**
```javascript
function addTitleSlide(pptx, title, subtitle, author) {
  const slide = pptx.addSlide();
  slide.background = { color: DESIGN.colors.primary };
  
  slide.addText(title, {
    x: 1, y: 2, w: 8, h: 1,
    fontSize: 44, bold: true, color: "FFFFFF",
    align: "center"
  });
  
  slide.addText(subtitle, {
    x: 1, y: 3.2, w: 8, h: 0.5,
    fontSize: 24, color: "FFFFFF",
    align: "center"
  });
  
  slide.addText(author, {
    x: 1, y: 4, w: 8, h: 0.4,
    fontSize: 18, color: "FFFFFF",
    align: "center"
  });
  
  return slide;
}

function addContentSlide(pptx, title, bullets) {
  const slide = pptx.addSlide();
  
  slide.addText(title, {
    x: DESIGN.layout.margin,
    y: DESIGN.layout.titleY,
    w: 9,
    h: 0.5,
    ...DESIGN.fonts.heading,
    color: DESIGN.colors.primary
  });
  
  slide.addText(bullets, {
    x: DESIGN.layout.margin,
    y: DESIGN.layout.contentY,
    w: 9,
    h: 3,
    ...DESIGN.fonts.body,
    bullet: true
  });
  
  return slide;
}
```

**步骤3：构建演示文稿**（视觉优先策略）
```javascript
const pptx = new PptxGenJS();
pptx.layout = "LAYOUT_16x9";

// 带背景图的标题页
const titleSlide = pptx.addSlide();
titleSlide.background = { color: DESIGN.colors.primary }; // 纯色背景
addTitleSlide(
  pptx,
  "研究标题",
  "副标题或会议名称",
  "姓名 • 机构 • 日期"
);

// 带概念图的引言页
const introSlide = pptx.addSlide();
introSlide.addImage({
  path: "concept_image.png",  // 概念可视化图
  x: 5, y: 1.5, w: 4, h: 3
});
introSlide.addText("背景", { x: 0.5, y: 0.5, fontSize: 36, bold: true });
introSlide.addText([
  "核心背景点1 (AuthorA, 2023)",
  "核心背景点2 (AuthorB, 2022)",
  "研究缺口 (AuthorC, 2021)"
], {
  x: 0.5, y: 1.5, w: 4, h: 2,
  fontSize: 24, bullet: true
});

// 图表主导的结果页
const resultsSlide = pptx.addSlide();
resultsSlide.addText("主要发现", { x: 0.5, y: 0.5, fontSize: 32, bold: true });
resultsSlide.addImage({
  path: "results_figure.png",  // 清晰的大幅图表
  x: 0.5, y: 1.5, w: 9, h: 4   // 几乎占满页面
});
// 最小化文字标注
resultsSlide.addText("34%提升 (p < 0.001)", {
  x: 7, y: 1, fontSize: 20, color: DESIGN.colors.accent, bold: true
});

// 保存
pptx.writeFile({ fileName: "presentation.pptx" });
```

**对比枯燥演示的关键改进**：
- 标题页采用纯色背景（非纯白）
- 引言页包含相关图片（非纯文字）
- 结果页以图表为主导（非文字主导）
- 要点中包含文献引用
- 文字精简辅助，视觉元素为主

### 添加科学内容

**公式**（转为图片）：
```javascript
// 先将公式渲染为PNG（使用LaTeX或在线工具）
// 再添加到幻灯片
slide.addImage({
  path: "equation.png",
  x: 2, y: 3, w: 6, h: 1
});
```

**表格**：
```javascript
slide.addTable([
  [
    { text: "方法", options: { bold: true } },
    { text: "准确率", options: { bold: true } },
    { text: "耗时(秒)", options: { bold: true } }
  ],
  ["方法A", "0.85", "10"],
  ["方法

```bash
python scripts/thumbnail.py presentation.pptx review/thumbnails --cols 4

# 或针对单张幻灯片
python scripts/thumbnail.py presentation.pptx review/slide
```

### 检查清单

每张幻灯片需检查：
- [ ] 文本清晰可读（无截断或过小）
- [ ] 无元素重叠
- [ ] 颜色与字体统一
- [ ] 留白充足
- [ ] 图表清晰且尺寸适当
- [ ] 对齐正确

### 常见问题

**文本溢出**：
- 缩小字号或缩短文本
- 扩大文本框
- 拆分为多张幻灯片

**元素重叠**：
- 采用双栏布局
- 缩小元素尺寸
- 调整位置

**对比度不足**：
- 选择高对比度配色
- 浅底深字原则
- 使用对比度检测工具

## 模板与示例

### 基于模板创建

若已有模板：

1. **提取模板结构**：
```bash
python scripts/inventory.py template.pptx inventory.json
```

2. **创建缩略图网格**：
```bash
python scripts/thumbnail.py template.pptx template_review
```

3. **分析版式**并记录需使用的幻灯片

4. **重排幻灯片顺序**：
```bash
python scripts/rearrange.py template.pptx working.pptx 0,5,5,12,18,22
```

5. **替换内容**：
```bash
python scripts/replace.py working.pptx replacements.json output.pptx
```

## 最佳实践摘要

### 推荐做法（提升演示吸引力）

- ✅ 通过research-lookup引用8-15篇文献
- ✅ 每张幻灯片添加高质量视觉元素（图表/图片/图示/图标）
- ✅ 选用符合主题的现代配色（避免默认模板）
- ✅ 精简文本（3-4个要点，每点4-6词）
- ✅ 使用大字号（正文24-28pt，标题36-44pt）
- ✅ 多样化版式（全图/双栏/视觉叠加）
- ✅ 保持高对比度（推荐7:1）
- ✅ 充足留白（占幻灯片40-50%）
- ✅ 引言和讨论部分引用文献（建立可信度）
- ✅ 测试远距离可读性
- ✅ 演示前进行视觉验证

### 避免做法（防止枯燥演示）

- ❌ 勿创建纯文本幻灯片（每页需含视觉元素）
- ❌ 勿直接套用默认主题（按主题定制）
- ❌ 勿全用要点列表（多样化版式）
- ❌ 勿跳过文献检索（演示同样需要引证）
- ❌ 勿在单页堆砌过多文本
- ❌ 勿使用过小字号（正文<24pt）
- ❌ 勿仅依赖颜色传达信息
- ❌ 勿使用复杂动画
- ❌ 勿混用过多字体样式
- ❌ 勿忽视无障碍设计
- ❌ 勿跳过视觉验证

## 无障碍设计要点

**颜色对比**：
- 使用WebAIM对比检测工具
- 常规文本最低4.5:1
- 推荐7:1实现最佳可读性

**色盲适配**：
- 通过Coblis模拟器测试
- 结合图案/形状与颜色
- 避免红绿组合

**可读性**：
- 仅使用无衬线字体
- 最小18pt，推荐24pt+
- 清晰的视觉层次
- 合理间距

## 与其他技能整合

**结合科技写作**：
- 将论文内容转为幻灯片
- 简化密集文本
- 提取核心发现
- 创建视觉摘要

**结合数据可视化**：
- 简化期刊图表
- 用大标签重制图表
- 采用渐进式呈现
- 突出关键结果

**结合文献检索**：
- 查找相关论文
- 提取关键引文
- 构建背景脉络
- 用证据支持论点

## 资源

**PowerPoint教程**：
- Microsoft官方文档
- PowerPoint设计模板
- 科技演示案例

**设计工具**：
- 配色生成器（Coolors.co）
- 对比检测器（WebAIM）
- 图标库（Noun Project）
- 图像编辑（PPT内置工具/外部软件）

**PPTX技能文档**：
- `document-skills/pptx/SKILL.md`：主文档
- `document-skills/pptx/html2pptx.md`：HTML转PPTX流程
- `document-skills/pptx/ooxml.md`：高级编辑指南
- `document-skills/pptx/scripts/`：工具脚本集

## 速查指南

### 常用幻灯片尺寸

- **16:9比例**：10英寸×5.625英寸（720pt×405pt）
- **4:3比例**：10英寸×7.5英寸（720pt×540pt）

### 度量单位

- PowerPoint使用英寸制
- 72点=1英寸
- 坐标(x,y)从左上角计算
- 尺寸(w,h)表示宽高

### 字号规范

| 元素       | 最小值 | 推荐值     |
|------------|--------|------------|
| 标题页     | 40pt   | 44-54pt    |
| 幻灯片标题 | 28pt   | 32-40pt    |
| 正文       | 18pt   | 24-28pt    |
| 图注       | 14pt   | 16-20pt    |
| 页脚       | 10pt   | 10-12pt    |

### 颜色使用

- **背景**：白色或浅色系
- **文本**：深色（黑/深灰）配浅底，或白字配深底
- **强调色**：最多使用1-2种
- **数据色**：色盲安全配色（蓝橙系）

## 故障排除

**问题**：文本显示不全  
- **解决**：扩大文本框或缩小字号

**问题**：图片模糊  
- **解决**：使用更高分辨率（300 DPI）

**问题**：投影时色彩失真  
- **解决**：提前投影测试，采用高对比度

**问题**：文件过大  
- **解决**：压缩图片，降低分辨率

**问题**：动画失效  
- **解决**：检查PowerPoint版本兼容性

## 结语

优质科研演示需具备：
1. 简洁清晰的设计
2. 可读文本（正文24pt+）
3. 高质量图表
4. 统一格式
5. 视觉验证
6. 无障碍考量

使用pptx技能进行程序化创建，并通过视觉审查流程确保演示前的专业质量。
