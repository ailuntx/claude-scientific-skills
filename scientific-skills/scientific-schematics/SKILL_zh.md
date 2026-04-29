---
name: scientific-schematics
description: 使用 Nano Banana 2 AI 创建符合出版质量的科学图表，并具备智能迭代优化功能。采用 Gemini 3.1 Pro Preview 进行质量审查。仅在质量低于文档类型阈值时重新生成。专注于神经网络架构、系统图、流程图、生物通路以及复杂的科学可视化。
allowed-tools: Read Write Edit Bash
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# 科学示意图与图表

## 概述

科学示意图与图表将复杂概念转化为清晰的视觉表示，以便出版。**本技能使用 Nano Banana 2 AI 生成图表，并由 Gemini 3.1 Pro Preview 进行质量审查。**

**工作原理：**
- 用自然语言描述您的图表
- Nano Banana 2 自动生成符合出版质量的图像
- **Gemini 3.1 Pro Preview 根据文档类型阈值评估质量**
- **智能迭代**：仅当质量低于阈值时才重新生成
- 数分钟内即可获得可出版输出
- 无需编码、模板或手动绘制

**按文档类型的质量阈值：**
| 文档类型 | 阈值 | 描述 |
|----------|------|------|
| journal | 8.5/10 | 《自然》、《科学》等同行评审期刊 |
| conference | 8.0/10 | 会议论文 |
| thesis | 8.0/10 | 学位论文 |
| grant | 8.0/10 | 基金申请 |
| preprint | 7.5/10 | arXiv、bioRxiv 等预印本 |
| report | 7.5/10 | 技术报告 |
| poster | 7.0/10 | 学术海报 |
| presentation | 6.5/10 | 幻灯片、演讲 |
| default | 7.5/10 | 通用用途 |

**只需描述您想要的，Nano Banana 2 就会创建出来。** 所有图表均存储在 figures/ 子文件夹中，并在论文/海报中引用。

## 快速开始：生成任意图表

通过简单描述即可创建任何科学图表。Nano Banana 2 会自动处理一切，并具备**智能迭代**功能：

```bash
# 为期刊论文生成（最高质量阈值：8.5/10）
python scripts/generate_schematic.py "CONSORT 参与者流程图，显示 500 人筛选，150 人排除，350 人随机分组" -o figures/consort.png --doc-type journal

# 为演示生成（较低阈值：6.5/10 - 速度更快）
python scripts/generate_schematic.py "展示多头注意力的 Transformer 编码器-解码器架构" -o figures/transformer.png --doc-type presentation

# 为海报生成（中等阈值：7.0/10）
python scripts/generate_schematic.py "从 EGFR 到基因转录的 MAPK 信号通路" -o figures/mapk_pathway.png --doc-type poster

# 自定义最大迭代次数（最多 2 次）
python scripts/generate_schematic.py "包含运算放大器、电阻和电容的复杂电路图" -o figures/circuit.png --iterations 2 --doc-type journal
```

**幕后流程：**
1. **首次生成**：Nano Banana 2 根据科学图表最佳实践创建初始图像
2. **首次审查**：**Gemini 3.1 Pro Preview** 根据文档类型阈值评估质量
3. **决策**：若质量 >= 阈值 → **完成**（无需更多迭代！）
4. **若低于阈值**：基于评估改进提示，重新生成
5. **重复**：直至质量满足阈值或达到最大迭代次数

**智能迭代优势：**
- ✅ 若首次生成质量足够，可节省 API 调用
- ✅ 对期刊论文要求更高质量标准
- ✅ 对演示/海报处理更快速
- ✅ 为每种用例提供适当的质量

**输出**：带版本号的图像以及详细的审查日志，包含质量评分、评估和提前停止信息。

### 配置

设置您的 OpenRouter API 密钥：
```bash
export OPENROUTER_API_KEY='your_api_key_here'
```

在此获取 API 密钥：https://openrouter.ai/keys

### AI 生成最佳实践

**科学图表的有效提示：**

✓ **优秀提示**（具体、详细）：
- "CONSORT 流程图，显示从筛选（n=500）到随机分组再到最终分析的参与者流动"
- "Transformer 神经网络架构，左侧编码器堆栈，右侧解码器堆栈，显示多头注意力和交叉注意力连接"
- "生物信号级联：EGFR 受体 → RAS → RAF → MEK → ERK → 细胞核，标出磷酸化步骤"
- "物联网系统框图：传感器 → 微控制器 → WiFi 模块 → 云服务器 → 移动应用"

✗ **避免模糊提示**：
- "制作一个流程图"（过于泛泛）
- "神经网络"（哪种类型？包含哪些组件？）
- "通路图"（哪条通路？哪些分子？）

**应包含的关键要素：**
- **类型**：流程图、架构图、通路、电路等
- **组件**：要包含的具体元素
- **流向/方向**：元素如何连接（从左到右、从上到下）
- **标签**：关键注释或要包含的文字
- **样式**：任何特定的视觉要求

**科学质量准则**（自动应用）：
- 干净的白色/浅色背景
- 高对比度以确保可读性
- 清晰、可读的标签（最小 10pt）
- 专业排版（无衬线字体）
- 色盲友好型颜色（Okabe-Ito 调色板）
- 适当间距以防止拥挤
- 在适当位置添加比例尺、图例、坐标轴

## 何时使用本技能

本技能适用于以下情况：
- 创建神经网络架构图（Transformer、CNN、RNN 等）
- 说明系统架构和数据流图
- 绘制研究设计方法论流程图（CONSORT、PRISMA）
- 可视化算法工作流和处理流水线
- 创建电路图和电气原理图
- 描绘生物通路和分子相互作用
- 生成网络拓扑和层次结构
- 说明概念框架和理论模型
- 设计技术论文的框图

## 如何使用本技能

**只需用自然语言描述您的图表。** Nano Banana 2 会自动生成：

```bash
python scripts/generate_schematic.py "您的图表描述" -o output.png
```

**就这样！** AI 负责处理：
- ✓ 布局与构图
- ✓ 标签与注释
- ✓ 颜色与样式
- ✓ 质量审查与优化
- ✓ 可出版输出

**适用于所有图表类型：**
- 流程图（CONSORT、PRISMA 等）
- 神经网络架构
- 生物通路
- 电路图
- 系统架构
- 框图
- 任何科学可视化

**无需编码、模板或手动绘制。**

---

# AI 生成模式（Nano Banana 2 + Gemini 3.1 Pro Preview 审查）

## 智能迭代优化工作流

AI 生成系统使用**智能迭代**——仅在质量低于文档类型阈值时重新生成：

### 智能迭代工作原理

```
┌─────────────────────────────────────────────────────┐
│  1. 使用 Nano Banana 2 生成图像                      │
│                    ↓                                │
│  2. 使用 Gemini 3.1 Pro Preview 审查质量             │
│                    ↓                                │
│  3. 评分 >= 阈值？                                   │
│       YES → 完成！（提前停止）                        │
│       NO  → 改进提示，返回步骤 1                      │
│                    ↓                                │
│  4. 重复直到质量达标或达到最大迭代次数                  │
└─────────────────────────────────────────────────────┘
```

### 迭代 1：初始生成
**提示构造：**
```
科学图表指南 + 用户请求
```

**输出：** `diagram_v1.png`

### Gemini 3.1 Pro Preview 的质量审查

Gemini 3.1 Pro Preview 从以下方面评估图表：
1. **科学准确性**（0-2 分）—— 概念、符号、关系是否正确
2. **清晰度和可读性**（0-2 分）—— 易于理解，层次清晰
3. **标签质量**（0-2 分）—— 标签完整、可读、一致
4. **布局与构图**（0-2 分）—— 逻辑流畅、平衡、无重叠
5. **专业外观**（0-2 分）—— 可出版质量

**审查输出示例：**
```
评分：8.0

优点：
- 自上而下清晰的流向
- 所有阶段均已正确标注
- 专业排版

问题：
- 参与者数量稍小
- 排除框有轻微重叠

判定：可接受（对于海报，阈值 7.0）
```

### 决策点：继续还是停止？

| 如果评分... | 操作 |
|-------------|------|
| >= 阈值 | **停止**——质量对该文档类型已足够 |
| < 阈值 | 使用改进后的提示继续下一迭代 |

**示例：**
- 对于**海报**（阈值 7.0）：评分 7.5 → **1 次迭代后完成！**
- 对于**期刊**（阈值 8.5）：评分 7.5 → 继续改进

### 后续迭代（仅在需要时）

如果质量低于阈值，系统会：
1. 从 Gemini 3.1 Pro Preview 的审查中提取具体问题
2. 使用改进指令增强提示
3. 使用 Nano Banana 2 重新生成
4. 再次使用 Gemini 3.1 Pro Preview 审查
5. 重复直到达到阈值或最大迭代次数

### 审查日志
所有迭代均保存为 JSON 审查日志，包含提前停止信息：
```json
{
  "user_prompt": "CONSORT 参与者流程图...",
  "doc_type": "poster",
  "quality_threshold": 7.0,
  "iterations": [
    {
      "iteration": 1,
      "image_path": "figures/consort_v1.png",
      "score": 7.5,
      "needs_improvement": false,
      "critique": "评分：7.5\n优点：..."
    }
  ],
  "final_score": 7.5,
  "early_stop": true,
  "early_stop_reason": "质量评分 7.5 满足海报阈值 7.0"
}
```

**注意：** 使用智能迭代时，如果质量提前达成，您可能只看到 1 次迭代而不是完整的 2 次！

## 高级 AI 生成用法

### Python API

```python
from scripts.generate_schematic_ai import ScientificSchematicGenerator

# 初始化生成器
generator = ScientificSchematicGenerator(
    api_key="your_openrouter_key",
    verbose=True
)

# 使用迭代优化生成（最多 2 次迭代）
results = generator.generate_iterative(
    user_prompt="Transformer 架构图",
    output_path="figures/transformer.png",
    iterations=2
)

# 访问结果
print(f"最终评分：{results['final_score']}/10")
print(f"最终图像：{results['final_image']}")

# 查看每次迭代
for iteration in results['iterations']:
    print(f"迭代 {iteration['iteration']}：{iteration['score']}/10")
    print(f"评估：{iteration['critique']}")
```

### 命令行选项

```bash
# 基本用法（默认阈值 7.5/10）
python scripts/generate_schematic.py "diagram description" -o output.png

# 指定文档类型以获得适当的质量阈值
python scripts/generate_schematic.py "diagram" -o out.png --doc-type journal      # 8.5/10
python scripts/generate_schematic.py "diagram" -o out.png --doc-type conference   # 8.0/10
python scripts/generate_schematic.py "diagram" -o out.png --doc-type poster       # 7.0/10
python scripts/generate_schematic.py "diagram" -o out.png --doc-type presentation # 6.5/10

# 自定义最大迭代次数（1-2）
python scripts/generate_schematic.py "complex diagram" -o diagram.png --iterations 2

# 详细输出（查看所有 API 调用和审查）
python scripts/generate_schematic.py "flowchart" -o flow.png -v

# 通过标志提供 API 密钥
python scripts/generate_schematic.py "diagram" -o out.png --api-key "sk-or-v1-..."

# 组合选项
python scripts/generate_schematic.py "neural network" -o nn.png --doc-type journal --iterations 2 -v
```

### 提示工程技巧

**1. 具体说明布局：**
```
✓ "流程图，垂直流向，自上而下"
✓ "架构图，左侧编码器，右侧解码器"
✓ "圆形通路图，顺时针流向"
```

**2. 包含量化细节：**
```
✓ "神经网络，输入层（784 个节点），隐藏层（128 个节点），输出层（10 个节点）"
✓ "流程图显示 n=500 筛选，n=150 排除，n=350 随机分组"
✓ "电路包含 1kΩ 电阻、10µF 电容、5V 电源"
```

**3. 指定视觉风格：**
```
✓ "极简框图，线条干净利落"
✓ "详细的生物通路，包含蛋白质结构"
✓ "带有工程符号的技术原理图"
```

**4. 请求特定标签：**
```
✓ "在所有箭头上标注激活/抑制"
✓ "在每个框中包含层尺寸"
✓ "用时间戳显示时间进程"
```

**5. 提及颜色要求：**
```
✓ "使用色盲友好型颜色"
✓ "灰度兼容设计"
✓ "按功能颜色编码：蓝色用于输入，绿色用于处理，红色用于输出"
```

## AI 生成示例

### 示例 1：CONSORT 流程图
```bash
python scripts/generate_schematic.py \
  "用于随机对照试验的 CONSORT 参与者流程图。\
   顶部以'评估资格（n=500）'开始。\
   显示'排除（n=150）'，并注明原因：年龄<18（n=80）、拒绝（n=50）、其他（n=20）。\
   然后'随机分组（n=350）'分为两个组：\
   '治疗组（n=175）'和'对照组（n=175）'。\
   每个组显示'失访'（n=15 和 n=10）。\
   以'分析'（n=160 和 n=165）结束。\
   流程步骤使用蓝色框，排除用橙色框，最终分析用绿色框。" \
  -o figures/consort.png
```

### 示例 2：神经网络架构
```bash
python scripts/generate_schematic.py \
  "Transformer 编码器-解码器架构图。\
   左侧：编码器堆栈，包含输入嵌入、位置编码、\
   多头自注意力、加与归一化、前馈、加与归一化。\
   右侧：解码器堆栈，包含输出嵌入、位置编码、\
   掩码自注意力、加与归一化、交叉注意力（从编码器接收）、\
   加与归一化、前馈、加与归一化、线性与 Softmax。\
   使用虚线显示从编码器到解码器的交叉注意力连接。\
   编码器使用浅蓝色，解码器使用浅红色。\
   清晰标注所有组件。" \
  -o figures/transformer.png --iterations 2
```

### 示例 3：生物通路
```bash
python scripts/generate_schematic.py \
  "MAPK 信号通路图。\
   从细胞膜（顶部）的 EGFR 受体开始。\
   箭头向下至 RAS（标注 GTP）。\
   箭头至 RAF 激酶。\
   箭头至 MEK 激酶。\
   箭头至 ERK 激酶。\
   最终箭头指向细胞核，显示基因转录。\
   在每个箭头上标注'磷酸化'或'激活'。\
   蛋白质使用圆角矩形，每种颜色不同。\
   顶部包含细胞膜边界线。" \
  -o figures/mapk_pathway.png
```

### 示例 4：系统架构
```bash
python scripts/generate_schematic.py \
  "物联网系统架构框图。\
   底层：传感器（温度、湿度、运动），使用绿色框。\
   中间层：微控制器（ESP32），使用蓝色框。\
   连接至 WiFi 模块（橙色框）和显示器（紫色框）。\
   顶层：云服务器（灰色框）连接至移动应用（浅蓝色框）。\
   显示所有组件之间的数据流箭头。\
   用协议标注连接：I2C、UART、WiFi、HTTPS。" \
  -o figures/iot_architecture.png
```

---

## 命令行用法

生成科学示意图的主要入口点：

```bash
# 基本用法
python scripts/generate_schematic.py "diagram description" -o output.png

# 自定义迭代次数（最多 2 次）
python scripts/generate_schematic.py "complex diagram" -o diagram.png --iterations 2

# 详细模式
python scripts/generate_schematic.py "diagram" -o out.png -v
```

**注意：** Nano Banana 2 AI 生成系统在其迭代优化过程中包含自动质量审查。每次迭代都会评估科学准确性、清晰度和可访问性。

## 最佳实践总结

### 设计原则

1. **清晰优于复杂** —— 简化，移除不必要的元素
2. **统一风格** —— 使用模板和样式文件
3. **色盲友好** —— 使用 Okabe-Ito 调色板，冗余编码
4. **适当排版** —— 无衬线字体，最小 7-8 pt
5. **矢量格式** —— 出版时始终使用 PDF/SVG

### 技术要求

1. **分辨率** —— 首选矢量，或 300+ DPI 的位图
2. **文件格式** —— LaTeX 用 PDF，网页用 SVG，PNG 作为后备
3. **色彩空间** —— 数字用 RGB，印刷用 CMYK（必要时转换）
4. **线条粗细** —— 最小 0.5 pt，通常 1-2 pt
5. **字号** —— 最终尺寸下最小 7-8 pt

### 集成指南

1. **在 LaTeX 中包含** —— 使用 `\includegraphics{}` 引用生成的图像
2. **完整标题** —— 描述所有元素和缩写
3. **在正文中引用** —— 在叙述流程中解释图表
4. **保持一致性** —— 论文中所有图形风格统一
5. **版本控制** —— 将提示和生成的图像保存在仓库中

## 常见问题排查

### AI 生成问题

**问题**：文本或元素重叠
- **解决方案**：AI 生成自动处理间距
- **解决方案**：增加迭代次数：`--iterations 2` 以获得更好的优化

**问题**：元素连接不正确
- **解决方案**：使提示更具体，关于连接和布局
- **解决方案**：增加迭代次数以更好地优化

### 图像质量问题

**问题**：导出质量差
- **解决方案**：AI 生成自动生成高质量图像
- **解决方案**：增加迭代次数以获得更好结果：`--iterations 2`

**问题**：生成后元素重叠
- **解决方案**：AI 生成自动处理间距
- **解决方案**：增加迭代次数：`--iterations 2` 以获得更好的优化
- **解决方案**：使提示更具体，关于布局和间距要求

### 质量检查问题

**问题**：重叠检测假阳性
- **解决方案**：调整阈值：`detect_overlaps(image_path, threshold=0.98)`
- **解决方案**：在可视化报告中手动检查标记区域

**问题**：生成图像质量低
- **解决方案**：AI 生成默认生成高质量图像
- **解决方案**：增加迭代次数以获得更好结果：`--iterations 2`

**问题**：色盲模拟显示对比度差
- **解决方案**：在代码中明确切换到 Okabe-Ito 调色板
- **解决方案**：添加冗余编码（形状、图案、线条样式）
- **解决方案**：增加颜色饱和度和明度差异

**问题**：检测到高严重性重叠
- **解决方案**：查看 overlap_report.json 了解确切位置
- **解决方案**：增加这些特定区域的间距
- **解决方案**：重新运行调整后的参数并再次验证

**问题**：可视化报告生成失败
- **解决方案**：检查 Pillow 和 matplotlib 安装
- **解决方案**：确保图像文件可读：`Image.open(path).verify()`
- **解决方案**：检查足够磁盘空间用于报告生成

### 可访问性问题

**问题**：灰度下颜色无法区分
- **解决方案**：运行可访问性检查器：`verify_accessibility(image_path)`
- **解决方案**：添加图案、形状或线条样式以实现冗余
- **解决方案**：增加相邻元素之间的对比度

**问题**：打印时文本太小
- **解决方案**：运行分辨率验证器：`validate_resolution(image_path)`
- **解决方案**：按最终尺寸设计，使用最小 7-8 pt 字体
- **解决方案**：在分辨率报告中检查物理尺寸

**问题**：可访问性检查始终失败
- **解决方案**：查看 accessibility_report.json 了解具体失败原因
- **解决方案**：将颜色对比度提高至少 20%
- **解决方案**：定稿前使用实际灰度转换进行测试

## 资源与参考

### 详细参考

加载以下文件以获取特定主题的全面信息：

- **`references/best_practices.md`** —— 出版标准和可访问性指南

### 外部资源

**Python 库**
- Schemdraw 文档：https://schemdraw.readthedocs.io/
- NetworkX 文档：https://networkx.org/documentation/
- Matplotlib 文档：https://matplotlib.org/

**出版标准**
- Nature 图形指南：https://www.nature.com/nature/for-authors/final-submission
- Science 图形指南：https://www.science.org/content/page/instructions-preparing-initial-manuscript
- CONSORT 图：http://www.consort-statement.org/consort-statement/flow-diagram

## 与其他技能的集成

本技能与以下技能协同工作：

- **科学写作** —— 图表遵循图形最佳实践
- **科学可视化** —— 共享调色板和样式
- **LaTeX 海报** —— 为海报展示生成图表
- **研究基金** —— 用于提案的方法论图
- **同行评审** —— 评估图表清晰度和可访问性

## 快速参考检查清单

提交图表前，请验证：

### 视觉质量
- [ ] 高质量图像格式（AI 生成的 PNG）
- [ ] 无重叠元素（AI 自动处理）
- [ ] 所有组件间距充足（AI 优化）
- [ ] 清晰、专业的对齐
- [ ] 所有箭头正确连接到目标

### 可访问性
- [ ] 使用色盲友好调色板（Okabe-Ito）
- [ ] 灰度下可工作（使用可访问性检查器测试）
- [ ] 元素间对比度充足（已验证）
- [ ] 在适当位置使用冗余编码（形状 + 颜色）
- [ ] 色盲模拟通过所有检查

### 排版与可读性
- [ ] 最终尺寸下文本最小 7-8 pt
- [ ] 所有元素标注清晰完整
- [ ] 一致的字体族和大小
- [ ] 无文本重叠或截断
- [ ] 在适用处包含单位

### 出版标准
- [ ] 与手稿中其他图形风格一致
- [ ] 撰写完整的标题，定义所有缩写
- [ ] 在手稿正文中适当引用
- [ ] 满足期刊特定的尺寸要求
- [ ] 以期刊要求的格式导出（PDF/EPS/TIFF）

### 质量验证（必需）
- [ ] 运行 `run_quality_checks()` 并获得 PASS 状态
- [ ] 查看重叠检测报告（无高严重性重叠）
- [ ] 通过可访问性验证（灰度和色盲）
- [ ] 分辨率在目标 DPI 下验证（印刷 300+）
- [ ] 生成并审查视觉质量报告
- [ ] 所有质量报告与图形文件一起保存

### 文档与版本控制
- [ ] 源文件（.tex, .py）保存以备将来修改
- [ ] 质量报告归档在 `quality_reports/` 目录
- [ ] 配置参数已记录（颜色、间距、尺寸）
- [ ] Git 提交包含源文件、输出和质量报告
- [ ] README 或注释说明如何重新生成图形

### 最终集成检查
- [ ] 图形在编译后的手稿中显示正确
- [ ] 交叉引用工作正常（`\ref{}` 指向正确的图形）
- [ ] 图形编号与正文引用匹配
- [ ] 标题出现在图形附近的正确页面
- [ ] 与图形相关的编译无警告或错误

## 环境设置

```bash
# 必需
export OPENROUTER_API_KEY='your_api_key_here'

# 在此获取密钥：https://openrouter.ai/keys
```

## 快速入门

**最简单的用法：**
```bash
python scripts/generate_schematic.py "your diagram description" -o output.png
```

---

使用本技能创建清晰、可访问、符合出版质量的图表，有效传达复杂的科学概念。AI 驱动的工作流配合迭代优化，确保图表符合专业标准。
