```markdown
---
name: infographics
description: "使用 Nano Banana Pro AI 创建专业信息图，支持智能迭代优化。采用 Gemini 3 Pro 进行质量审核，集成研究查询和网络搜索确保数据准确性。支持10种信息图类型、8种行业风格和色盲友好配色方案。"
allowed-tools: Read Write Edit Bash
---

# 信息图

## 概述

信息图是通过视觉化方式呈现信息、数据或知识的工具，旨在快速清晰地展示复杂内容。**本技能使用 Nano Banana Pro AI 生成信息图，通过 Gemini 3 Pro 进行质量审核，并集成 Perplexity Sonar 进行研究。**

**工作原理：**
- (可选) **研究阶段**：使用 Perplexity Sonar 收集准确事实和统计数据
- 用自然语言描述信息图需求
- Nano Banana Pro 自动生成出版级信息图
- **Gemini 3 Pro 根据文档类型阈值审核质量**
- **智能迭代**：仅当质量低于阈值时重新生成
- 数分钟内获得专业级成品
- 无需设计技能

**文档类型质量阈值：**
| 文档类型 | 阈值 | 描述 |
|---------------|-----------|-------------|
| 营销材料 | 8.5/10 | 营销物料 - 必须具有吸引力 |
| 报告 | 8.0/10 | 商业报告 - 专业品质 |
| 演示文稿 | 7.5/10 | 幻灯片/演讲 - 清晰且引人入胜 |
| 社交媒体 | 7.0/10 | 社交媒体内容 |
| 内部使用 | 7.0/10 | 内部文件 |
| 草稿 | 6.5/10 | 工作草案 |
| 默认 | 7.5/10 | 通用场景 |

**只需描述需求，Nano Banana Pro 即可自动创建。**

## 快速入门

通过简单描述生成任意信息图：

```bash
# 生成列表式信息图（默认阈值7.5/10）
python skills/infographics/scripts/generate_infographic.py \
  "定期锻炼的5大益处" \
  -o figures/exercise_benefits.png --type list

# 生成营销材料（最高阈值8.5/10）
python skills/infographics/scripts/generate_infographic.py \
  "产品功能对比" \
  -o figures/product_comparison.png --type comparison --doc-type marketing

# 采用企业风格生成
python skills/infographics/scripts/generate_infographic.py \
  "2010-2025年公司里程碑" \
  -o figures/timeline.png --type timeline --style corporate

# 使用色盲友好配色生成
python skills/infographics/scripts/generate_infographic.py \
  "全球心脏病统计数据" \
  -o figures/health_stats.png --type statistical --palette wong

# 通过研究生成准确最新数据
python skills/infographics/scripts/generate_infographic.py \
  "全球AI市场规模及增长预测" \
  -o figures/ai_market.png --type statistical --research
```

**后台流程：**
1. **(可选) 研究**：Perplexity Sonar 收集准确事实、统计数据
2. **首次生成**：Nano Banana Pro 遵循设计规范创建初始信息图
3. **首次审核**：**Gemini 3 Pro** 根据文档类型阈值评估质量
4. **决策**：若质量≥阈值 → **完成**（无需迭代）
5. **若低于阈值**：根据反馈优化提示词后重新生成
6. **重复**：直至达标或达到最大迭代次数

**智能迭代优势：**
- ✅ 首版达标时节省API调用
- ✅ 营销材料采用更高质量标准
- ✅ 草稿/内部文件更快交付
- ✅ 按场景匹配适当质量

**输出**：版本化图像及详细审核日志（含质量评分、反馈和提前终止原因）。

## 适用场景

在以下情况使用**信息图**技能：
- 以视觉形式呈现数据或统计数据
- 创建项目里程碑或历史时间线
- 解释流程、工作流或分步指南
- 并排对比选项/产品/概念
- 以视觉形式总结核心观点
- 创建地理/地图数据可视化
- 构建层级结构或组织架构图
- 设计社交媒体内容或营销物料

**以下场景请改用科学示意图技能：**
- 技术流程图与电路图
- 生物通路与分子结构图
- 神经网络架构图
- CONSORT/PRISMA方法图

---

## 研究集成

### 自动数据收集 (`--research`)

当信息图需要准确最新数据时，使用`--research`参数通过**Perplexity Sonar Pro**自动收集信息。

```bash
# 研究并生成统计信息图
python skills/infographics/scripts/generate_infographic.py \
  "各国可再生能源采用率" \
  -o figures/renewable_energy.png --type statistical --research

# 为时间线信息图收集研究数据
python skills/infographics/scripts/generate_infographic.py \
  "人工智能突破性进展史" \
  -o figures/ai_history.png --type timeline --research

# 为对比信息图收集研究数据
python skills/infographics/scripts/generate_infographic.py \
  "电动车与氢能源车对比" \
  -o figures/ev_hydrogen.png --type comparison --research
```

### 研究阶段产出

研究阶段自动完成：
1. **收集关键事实**：5-8条相关事实与统计数据
2. **提供背景**：确保准确呈现的背景信息
3. **定位数据点**：具体数值、百分比及日期
4. **引用来源**：标注主要研究或数据源
5. **时效优先**：聚焦2023-2026年最新信息

### 研究功能适用场景

**启用研究 (`--research`) 的场景：**
- 需要精确数值的统计信息图
- 市场数据/行业统计/趋势分析
- 科学或医疗信息
- 时事或近期动态
- 任何对准确性要求高的主题

**无需研究的场景：**
- 简单概念性信息图
- 内部流程文档
- 提示词已包含完整数据时
- 速度优先的生成任务

### 研究输出

启用研究时额外生成：
- `{名称}_research.json` - 原始研究数据及来源
- 研究内容自动融入信息图提示词

---

## 信息图类型

### 1. 统计/数据驱动型 (`--type statistical`)

适用场景：展示数值、百分比、调查结果等定量数据。

**核心元素：** 图表（柱状/饼状/折线/环形图）、突出数值、数据对比、趋势指示器。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "2025年全球网络使用情况：55亿用户（占人口68%），\
   亚太53%，欧洲15%，美洲20%，非洲12%" \
  -o figures/internet_stats.png --type statistical --style technology
```

---

### 2. 时间线型 (`--type timeline`)

适用场景：历史事件、项目里程碑、公司发展史、概念演进。

**核心元素：** 时间轴、日期标记、事件节点、连接线。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "AI发展史：1950图灵测试，1956达特茅斯会议，\
   1997深蓝，2016 AlphaGo，2022 ChatGPT" \
  -o figures/ai_history.png --type timeline --style technology
```

---

### 3. 流程/操作指南型 (`--type process`)

适用场景：分步说明、工作流、操作流程、教程。

**核心元素：** 步骤编号、方向箭头、操作图标、清晰流向。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "创建播客步骤：1.选择领域，2.规划内容，\
   3.准备设备，4.录制节目，5.发布推广" \
  -o figures/podcast_process.png --type process --style marketing
```

---

### 4. 对比型 (`--type comparison`)

适用场景：产品对比、优劣分析、前后对照、方案评估。

**核心元素：** 并排布局、匹配分类、对勾/叉号标识。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "电动车vs燃油车：燃料成本(低vs高)，\
   维护(少vs多)，续航(提升中vs稳定)" \
  -o figures/ev_comparison.png --type comparison --style nature
```

---

### 5. 列表/信息型 (`--type list`)

适用场景：技巧清单、关键事实、要点总结、速查指南。

**核心元素：** 编号/项目符号、图标、清晰层级。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "高效人士的7个习惯：积极主动、\
   以终为始、要事第一、双赢思维、\
   知彼解己、统合综效、不断更新" \
  -o figures/habits.png --type list --style corporate
```

---

### 6. 地理型 (`--type geographic`)

适用场景：区域数据、人口统计、基于位置的数据、全球趋势。

**核心元素：** 地图可视化、色块编码、数据叠加、图例。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "地区可再生能源采用率：冰岛100%，挪威98%，\
   德国50%，美国22%，印度20%" \
  -o figures/renewable_map.png --type geographic --style nature
```

---

### 7. 层级/金字塔型 (`--type hierarchical`)

适用场景：组织结构、优先级划分、重要性排序。

**核心元素：** 金字塔/树状结构、明确层级、尺寸递进。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "马斯洛需求层次：生理需求、安全需求、\
   社交需求、尊重需求、自我实现" \
  -o figures/maslow.png --type hierarchical --style education
```

---

### 8. 解剖/视觉隐喻型 (`--type anatomical`)

适用场景：通过熟悉视觉隐喻解释复杂系统。

**核心元素：** 核心隐喻图像、标注部件、连接线。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "企业人体模型：大脑=领导力，心脏=文化，\
   手臂=销售，双腿=运营，骨架=系统" \
  -o figures/business_body.png --type anatomical --style corporate
```

---

### 9. 简历/专业型 (`--type resume`)

适用场景：个人品牌、简历、作品集亮点、职业成就。

**核心元素：** 照片区、技能可视化、时间线、联系信息。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "UX设计师简历：技能-用户研究95%，线框图90%，\
   原型设计85%。经历-2020-2022初级，2022-2025高级" \
  -o figures/resume.png --type resume --style technology
```

---

### 10. 社交媒体型 (`--type social`)

适用场景：Instagram、LinkedIn、Twitter/X 帖子、可分享图文。

**核心元素：** 醒目标题、精简文字、强视觉冲击、鲜明色彩。

```bash
python skills/infographics/scripts/generate_infographic.py \
  "节约用水，守护生命：22亿人缺乏安全饮用水。\
   建议：缩短淋浴、修复漏水、满载洗涤" \
  -o figures/water_social.png --type social --style marketing
```

---

## 风格预设

### 行业风格 (`--style`)

| 风格 | 配色 | 适用场景 |
|-------|--------|----------|
| `corporate` | 藏青/钢蓝/金色 | 商业报告/金融 |
| `healthcare` | 医疗蓝/青蓝/浅青 | 医疗/健康 |
| `technology` | 科技蓝/石板色/紫罗兰 | 软件/数据/AI |
| `nature` | 森林绿/薄荷色/大地棕 | 环保/有机 |
| `education` | 学术蓝/浅蓝/珊瑚色 | 教育/学术 |
| `marketing` | 珊瑚色/青绿/明黄 | 社交媒体/营销活动 |
| `finance` | 藏青/金色/红绿色 | 投资/银行 |
| `nonprofit` | 暖橙/鼠尾草绿/沙色 | 公益事业/慈善 |

```bash
# 企业风格
python skills/infographics/scripts/generate_infographic.py \
  "第四季度业绩" -o q4.png --type statistical --style corporate

# 医疗风格
python skills/infographics/scripts/generate_infographic.py \
  "患者旅程" -o journey.png --type process --style healthcare
```

---

## 色盲友好配色

### 可用配色 (`--palette`)

| 配色方案 | 颜色 | 描述 |
|---------|--------|-------------|
| `wong` | 橙/天蓝/绿/蓝/朱红 | 最广泛推荐方案 |
| `ibm` | 群青/靛蓝/品红/橙/金 | IBM无障碍配色 |
| `tol` | 12色扩展方案 | 多类别适用 |

```bash
# 采用Wong色盲友好配色
python skills/infographics/scripts/generate_infographic.py \
  "分类调查结果" -o survey.png --type statistical --palette wong
```

---

## 智能迭代优化

### 工作原理

```
┌─────────────────────────────────────────────────────┐
│  1. 使用Nano Banana Pro生成信息图                   │
│                    ↓                                │
│  2. 通过Gemini 3 Pro审核质量                        │
│                    ↓                                │
│  3. 评分≥阈值？                                     │
│       是 → 完成! (提前终止)                         │
│       否 → 优化提示词，返回步骤1                     │
│                    ↓                                │
│  4. 循环直至达标或达最大迭代次数                     │
└─────────────────────────────────────────────────────┘
```

### 质量审核标准

Gemini 3 Pro 从五个维度评估（每项0-2分）：
1. **视觉层次与布局**
   - 清晰的视觉层次
   - 符合逻辑的阅读流
   - 均衡的构图

2. **排版与可读性**
   - 文本易读性
   - 标题突出性
   - 无重叠遮挡

3. **数据可视化**
   - 数值显著性
   -

"市场规模从100亿美元（2020年）增长至450亿美元（2025年），年复合增长率35%"
```

✗ **模糊描述**：
```
"市场正在增长"
```

### 明确视觉元素

✓ **优秀示例**：
```
"时间轴展示5个里程碑事件，每个事件配图标"
```

---

## 参考文件

详细指南请加载以下参考文件：

- **`references/infographic_types.md`**：包含10+种信息图类型的扩展模板
- **`references/design_principles.md`**：视觉层次、布局、排版原则
- **`references/color_palettes.md`**：完整配色方案规范

---

## 故障排除

### 常见问题

**问题**：信息图文字无法阅读
- **解决方案**：精简文字内容；使用--type指定布局类型

**问题**：色彩冲突或存在可访问性问题
- **解决方案**：使用`--palette wong`获取色盲友好配色

**问题**：质量评分过低
- **解决方案**：通过`--iterations 3`增加迭代次数；使用更具体的提示词

**问题**：生成的信息图类型错误
- **解决方案**：始终使用`--type`参数确保结果一致

---

## 与其他技能协同

本技能可与以下功能协同工作：

- **scientific-schematics**：用于技术图表和流程图
- **market-research-reports**：商业报告信息图
- **scientific-slides**：演示文稿中的信息图元素
- **generate-image**：非信息图类视觉内容

---

## 速查清单

生成前检查：
- [ ] 内容描述清晰具体
- [ ] 选定信息图类型（`--type`）
- [ ] 风格符合受众（`--style`）
- [ ] 指定输出路径（`-o`）
- [ ] 已配置API密钥

生成后检查：
- [ ] 审查生成图像
- [ ] 查看审核日志中的评分
- [ ] 如需改进，使用更具体提示词重新生成

---

通过Nano Banana Pro AI的智能质量审核，运用本技能可创建专业、易用且视觉震撼的信息图。
