# 市场研究报告可视化生成指南

用于市场研究报告的可视化生成完整提示与指导。

---

## 概述

市场研究报告应首先创建**5-6个核心可视化图表**建立分析框架。撰写具体章节时可按需生成额外图表。本指南提供适用于`scientific-schematics`和`generate-image`工具的即用型提示模板。

### 核心图表（优先生成 - 优先级1-6）

每份市场报告应首先生成以下5-6个核心图表：

1. **市场增长轨迹图** - 展示市场规模趋势
2. **TAM/SAM/SOM示意图** - 市场机会分解
3. **波特五力图** - 竞争动态框架
4. **竞争定位矩阵** - 战略定位分析
5. **风险热力图** - 风险评估可视化
6. **执行摘要信息图**（可选） - 报告概览

### 扩展图表（按需生成 - 优先级7+）

撰写特定章节需要视觉支持时可生成补充图表：
- 区域细分图
- 细分市场分析
- 客户旅程地图
- 技术路线图
- 监管时间轴
- 财务预测图
- 实施时间线

### 工具选择

| 图表类型 | 工具 | 选用依据 |
|-------------|------|-----------|
| 图表（柱状/折线/饼图） | scientific-schematics | 精确数据呈现 |
| 示意图（流程/结构） | scientific-schematics | 清晰技术布局 |
| 矩阵（2x2/定位） | scientific-schematics | 战略框架展示 |
| 时间轴 | scientific-schematics | 序列信息表达 |
| 信息图 | generate-image | 创意视觉整合 |
| 概念插图 | generate-image | 抽象概念表达 |

---

## 可视化命名规范

### 核心图表（优先生成）
```
figures/
├── 01_market_growth_trajectory.png      # 优先级1
├── 02_tam_sam_som.png                   # 优先级2
├── 03_porters_five_forces.png           # 优先级3
├── 04_competitive_positioning.png       # 优先级4
├── 05_risk_heatmap.png                  # 优先级5
└── 06_exec_summary_infographic.png      # 优先级6 (可选)
```

### 扩展图表（按需生成）
```
figures/
├── 07_industry_ecosystem.png
├── 08_regional_breakdown.png
├── 09_segment_growth.png
├── 10_driver_impact_matrix.png
├── 11_pestle_analysis.png
├── 12_trends_timeline.png
├── 13_market_share.png
├── 14_strategic_groups.png
├── 15_customer_segments.png
├── 16_segment_attractiveness.png
├── 17_customer_journey.png
├── 18_technology_roadmap.png
├── 19_innovation_curve.png
├── 20_regulatory_timeline.png
├── 21_risk_mitigation.png
├── 22_opportunity_matrix.png
├── 23_recommendation_priority.png
├── 24_implementation_timeline.png
├── 25_milestone_tracker.png
├── 26_financial_projections.png
└── 27_scenario_analysis.png
```

---

## 核心图表（优先级1-6） - 首先生成

### 优先级1：市场增长轨迹图

**工具：** scientific-schematics

**用途：** 展示历史与预测市场规模的基础图表

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Bar chart market growth 2020 to 2034. Historical bars 2020-2024 in dark blue, projected bars 2025-2034 in light blue. Y-axis billions USD, X-axis years. CAGR annotation. Data labels on each bar. Vertical dashed line between 2024 and 2025. Title: Market Growth Trajectory. Professional white background" \
  -o figures/01_market_growth_trajectory.png --doc-type report
```

---

### 优先级2：TAM/SAM/SOM示意图

**工具：** scientific-schematics

**用途：** 市场机会规模可视化

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "TAM SAM SOM concentric circles. Outer circle TAM Total Addressable Market. Middle circle SAM Serviceable Addressable Market. Inner circle SOM Serviceable Obtainable Market. Each labeled with acronym, full name, placeholder for dollar value. Arrows pointing to each with descriptions. Blue gradient darkest outer to lightest inner. White background professional appearance" \
  -o figures/02_tam_sam_som.png --doc-type report
```

---

### 优先级3：波特五力图

**工具：** scientific-schematics

**用途：** 竞争动态框架展示

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Porter's Five Forces diagram. Center box Competitive Rivalry with rating. Four surrounding boxes with arrows to center: Top Threat of New Entrants, Left Bargaining Power Suppliers, Right Bargaining Power Buyers, Bottom Threat of Substitutes. Color code HIGH red, MEDIUM yellow, LOW green. Include 2-3 key factors per box. Professional appearance" \
  -o figures/03_porters_five_forces.png --doc-type report
```

---

### 优先级4：竞争定位矩阵

**工具：** scientific-schematics

**用途：** 关键市场参与者战略定位

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "2x2 competitive positioning matrix. X-axis Market Focus Niche to Broad. Y-axis Solution Approach Product to Platform. Quadrants: Upper-right Platform Leaders, Upper-left Niche Platforms, Lower-right Product Leaders, Lower-left Specialists. Plot 8-10 company circles with names. Circle size = market share. Legend for sizes. Professional appearance" \
  -o figures/04_competitive_positioning.png --doc-type report
```

---

### 优先级5：风险热力图

**工具：** scientific-schematics

**用途：** 可视化风险评估矩阵

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Risk heatmap matrix. X-axis Impact Low Medium High Critical. Y-axis Probability Unlikely Possible Likely Very Likely. Cell colors: Green low risk, Yellow medium, Orange high, Red critical. Plot 10-12 numbered risks R1 R2 etc as labeled points. Legend with risk names. Professional clear" \
  -o figures/05_risk_heatmap.png --doc-type report
```

---

### 优先级6：执行摘要信息图（可选）

**工具：** generate-image

**用途：** 用于封面或执行摘要的高阶视觉整合

**命令：**
```bash
python skills/generate-image/scripts/generate_image.py \
  "Executive summary infographic for market research, one page layout, central large metric showing market size, four quadrants showing growth rate key players top segments regional leaders, modern flat design, professional blue and green color scheme, clean white background, corporate business aesthetic" \
  --output figures/06_exec_summary_infographic.png
```

---

## 扩展图表 - 撰写过程中按需生成

以下图表可在撰写特定章节需要时生成。

---

## 前页图表

### 扩展：封面主视觉图

**工具：** generate-image

**提示：**
```
Professional executive summary infographic for [MARKET NAME] market research report. 
Modern data visualization style showing key metrics: market size, growth rate, key players.
Blue and green color scheme matching corporate design.
Clean minimalist design with icons.
High resolution, publication quality.
No text overlays, image only.
```

**命令：**
```bash
python skills/generate-image/scripts/generate_image.py \
  "Professional executive summary infographic for [MARKET] market research report, modern data visualization style, key metrics display, blue and green corporate color scheme, clean minimalist design with icons, high resolution publication quality" \
  --output figures/01_cover_image.png
```

### 2. 执行摘要信息图

**工具：** generate-image

**提示：**
```
One-page executive summary infographic showing:
- Large central metric: $XX billion market size
- Four quadrants with: Growth Rate, Key Players, Top Segments, Regional Leaders
- Modern flat design with data visualization elements
- Professional blue (#003366) and green (#008060) color scheme
- Clean white background
- Business/corporate aesthetic
```

**命令：**
```bash
python skills/generate-image/scripts/generate_image.py \
  "Executive summary infographic for market research, one page layout, central large metric showing market size, four quadrants showing growth rate key players top segments regional leaders, modern flat design, professional blue and green color scheme, clean white background, corporate business aesthetic" \
  --output figures/02_exec_summary_infographic.png
```

---

## 第1章：市场概览图表

### 3. 产业生态示意图

**工具：** scientific-schematics

**提示：**
```
Industry ecosystem value chain diagram showing horizontal flow from left to right:
[Suppliers/Inputs] → [Manufacturers/Processors] → [Distributors/Channels] → [End Users/Customers]

At each stage, show 3-4 example player types in smaller boxes below.
Use arrows to show product/service flow (solid) and money flow (dashed).
Include regulatory bodies as oversight layer above the chain.
Professional blue color scheme.
Clean white background.
All text clearly readable.
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Industry ecosystem value chain diagram. Horizontal flow left to right: Suppliers box → Manufacturers box → Distributors box → End Users box. Below each main box show 3-4 smaller boxes with example player types. Solid arrows for product flow, dashed arrows for money flow. Regulatory oversight layer above. Professional blue color scheme, white background, clear labels" \
  -o figures/03_industry_ecosystem.png --doc-type report
```

### 4. 市场结构图

**工具：** scientific-schematics

**提示：**
```
Market structure diagram showing concentric rectangles:
- Center: Core Market (labeled with market name)
- Second layer: Adjacent Markets (labeled with 4-5 adjacent market names)
- Third layer: Enabling Technologies (labeled with key technologies)
- Outer layer: Regulatory Framework

Use different shades of blue for each layer.
Include small icons or labels for key elements.
Professional appearance.
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Market structure diagram with concentric rectangles. Center: Core Market [MARKET NAME]. Second layer: Adjacent Markets with 4-5 labels. Third layer: Enabling Technologies with key tech labels. Outer layer: Regulatory Framework. Different blue shades for each layer, professional appearance, clear labels" \
  -o figures/03b_market_structure.png --doc-type report
```

---

## 第2章：市场规模与增长图表

### 5. 市场增长轨迹图

**工具：** scientific-schematics

**提示：**
```
Bar chart showing market growth from 2020 to 2034.
Historical years (2020-2024): Dark blue bars
Projected years (2025-2034): Light blue bars
Y-axis: Market size in billions USD (0 to $XXX)
X-axis: Years
Include CAGR annotation showing "XX.X% CAGR (2024-2034)"
Data labels on top of each bar
Vertical dashed line separating historical from projected
Title: "[MARKET NAME] Market Growth Trajectory"
Professional appearance, white background
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Bar chart market growth 2020 to 2034. Historical bars 2020-2024 in dark blue, projected bars 2025-2034 in light blue. Y-axis billions USD, X-axis years. CAGR annotation XX.X% (2024-2034). Data labels on each bar. Vertical dashed line between 2024 and 2025. Title: Market Growth Trajectory. Professional white background" \
  -o figures/04_market_growth_trajectory.png --doc-type report
```

### 6. TAM/SAM/SOM示意图

**工具：** scientific-schematics

**提示：**
```
TAM SAM SOM concentric circles diagram:
- Outer circle: TAM (Total Addressable Market) - $XXX billion
- Middle circle: SAM (Serviceable Addressable Market) - $XX billion  
- Inner circle: SOM (Serviceable Obtainable Market) - $X billion

Each circle labeled with:
- Acronym in bold
- Full name
- Dollar value

Arrows pointing to each circle with descriptions
Use blue color gradient (darkest for TAM, lightest for SOM)
Professional appearance
White background
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "TAM SAM SOM concentric circles. Outer circle TAM Total Addressable Market [VALUE]B. Middle circle SAM Serviceable Addressable Market [VALUE]B. Inner circle SOM Serviceable Obtainable Market [VALUE]B. Each labeled with acronym, full name, dollar value. Arrows pointing to each with descriptions. Blue gradient darkest outer to lightest inner. White background professional" \
  -o figures/05_tam_sam_som.png --doc-type report
```

### 7. 区域市场细分图

**工具：** scientific-schematics

**提示：**
```
Pie chart OR treemap showing regional market breakdown:
- North America: XX% ($X.XB) - Dark blue
- Europe: XX% ($X.XB) - Medium blue
- Asia-Pacific: XX% ($X.XB) - Teal
- Latin America: X% ($X.XB) - Light blue
- Middle East & Africa: X% ($X.XB) - Gray blue

Include both percentage and dollar value for each region
Legend on right side
Title: "Market Size by Region (2024)"
Professional appearance
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Pie chart regional market breakdown. North America XX% dark blue, Europe XX% medium blue, Asia-Pacific XX% teal, Latin America XX% light blue, Middle East Africa XX% gray blue. Show percentage and dollar value for each slice. Legend on right. Title: Market Size by Region 2024. Professional appearance" \
  -o figures/06_regional_breakdown.png --doc-type report
```

### 8. 细分市场增长对比

**工具：** scientific-schematics

**提示：**
```
Horizontal bar chart comparing segment growth rates:
- Y-axis: Segment names (5-7 segments)
- X-axis: CAGR percentage (0% to 30%)
- Bars colored by growth rate: Green (highest) to blue (lowest)
- Data labels showing exact percentage on each bar
- Sort segments from highest to lowest growth
- Title: "Segment Growth Rate Comparison (CAGR 2024-2034)"
- Include average line or marker
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Horizontal bar chart segment growth comparison. Y-axis 5-7 segment names, X-axis CAGR percentage 0-30%. Bars colored green highest to blue lowest. Data labels with exact percentages. Sorted highest to lowest. Title: Segment Growth Rate Comparison CAGR 2024-2034. Include market average line" \
  -o figures/07_segment_growth.png --doc-type report
```

---

## 第3章：行业驱动与趋势图表

### 9. 驱动因素影响矩阵

**工具：** scientific-schematics

**提示：**
```
2x2 matrix for market driver assessment:
- X-axis: Impact on Market (Low → High)
- Y-axis: Probability of Occurrence (Low → High)
- Upper-right quadrant: "CRITICAL DRIVERS" (red/orange background)
- Upper-left quadrant: "MONITOR" (yellow background)
- Lower-right quadrant: "WATCH CAREFULLY" (yellow background)
- Lower-left quadrant: "LOWER PRIORITY" (green background)

Plot 8-10 drivers as labeled circles:
- Size of circle represents current market impact
- Position based on ratings

Include legend for circle sizes
Professional appearance with clear labels
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "2x2 matrix driver impact assessment. X-axis Impact Low to High, Y-axis Probability Low to High. Quadrants: Upper-right CRITICAL DRIVERS red, Upper-left MONITOR yellow, Lower-right WATCH CAREFULLY yellow, Lower-left LOWER PRIORITY green. Plot 8-10 labeled driver circles at appropriate positions. Circle size indicates current impact. Professional clear labels" \
  -o figures/08_driver_impact_matrix.png --doc-type report
```

### 10. PESTLE分析图

**工具：** scientific-schematics

**提示：**
```
PESTLE analysis hexagonal diagram:
- Center hexagon: "[MARKET NAME]" 
- Six surrounding hexagons connected to center:
  - Political (red/orange)
  - Economic (blue)
  - Social (green)
  - Technological (orange)
  - Legal (purple)
  - Environmental (teal)
```

每个外部六边形包含2-3个关键要点  
中心与外部六边形之间的连接线  
专业外观  
每个六边形内文字清晰可读  

```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "PESTLE六边形图。中心六边形标记为MARKET。周围六个六边形：Political红色，Economic蓝色，Social绿色，Technological橙色，Legal紫色，Environmental青色。每个外部六边形包含2-3个关键要点的项目符号。中心与每个外部六边形之间的连接线。专业外观，文字清晰可读" \
  -o figures/09_pestle_analysis.png --doc-type report
```

### 11. 行业趋势时间轴

**工具：** scientific-schematics

**提示：**
```
展示2024至2030年新兴趋势的水平时间轴：
- 带年份标记的主水平轴
- 在时间轴不同位置绘制6-8个趋势
- 每个趋势包含：
  - 图标或符号
  - 趋势名称
  - 下方简短描述（3-5词）

按趋势类别颜色编码：
- 技术趋势：蓝色
- 市场趋势：绿色
- 监管趋势：橙色

在2024年标注"当前"标记
专业外观，标签清晰
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "2024至2030年水平时间轴。在不同年份绘制6-8个新兴趋势。每个趋势含图标、名称、简短描述。颜色编码：技术趋势蓝色，市场趋势绿色，监管趋势橙色。2024年标注当前标记。专业清晰标签" \
  -o figures/10_trends_timeline.png --doc-type report
```

---

## 第四章：竞争格局可视化

### 12. 波特五力图

**工具：** scientific-schematics

**提示：**
```
波特五力图含中心框和四个周边框：

中心框："竞争强度" 标注评级 [高/中/低]

带箭头连接的周边框：
- 顶部："新进入者威胁" [评级]
- 左侧："供应商议价能力" [评级]
- 右侧："买方议价能力" [评级]
- 底部："替代品威胁" [评级]

评级颜色编码：
- 高：红/橙色背景
- 中：黄色背景
- 低：绿色背景

箭头指向中心框
每个框内包含关键因素要点
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "波特五力图。中心框竞争强度[评级]。四个周边框带箭头指向中心：顶部新进入者威胁[评级]，左侧供应商议价能力[评级]，右侧买方议价能力[评级]，底部替代品威胁[评级]。颜色编码：高红色，中黄色，低绿色。每个框含2-3个关键因素。专业外观" \
  -o figures/11_porters_five_forces.png --doc-type report
```

### 13. 市场份额图表

**工具：** scientific-schematics

**提示：**
```
展示市场份额的饼图或环形图：
- 前10家公司使用不同颜色
- 公司A：XX%（最大区块，深蓝）
- 公司B：XX%（中蓝）
- [继续列出前10家公司]
- 其他：XX%（灰色）

包含：
- 每个区块的百分比标签
- 图例或区块上的公司名称
- 总市场规模标注
- 标题："企业市场份额（2024年）"

专业外观
色盲友好配色
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "前10家企业市场份额饼图。公司A XX%深蓝，公司B XX%中蓝，[列出公司及份额]，其他XX%灰色。区块显示百分比标签。图例标注公司名称。总市场规模说明。标题：2024年企业市场份额。色盲友好配色，专业外观" \
  -o figures/12_market_share.png --doc-type report
```

### 14. 竞争定位矩阵

**工具：** scientific-schematics

**提示：**
```
2x2竞争定位矩阵：
- X轴：市场聚焦（细分←→广泛）
- Y轴：解决方案模式（产品←→平台）

象限标注：
- 右上："平台领导者"
- 左上："细分平台商"
- 右下："产品领导者"
- 左下："专业服务商"

绘制8-10家公司的标注圆圈：
- 圆圈大小代表市场份额
- 位置基于战略定位

包含圆圈大小图例
公司名称标签
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "2x2竞争定位矩阵。X轴市场聚焦（细分到广泛），Y轴解决方案模式（产品到平台）。象限：右上平台领导者，左上细分平台商，右下产品领导者，左下专业服务商。绘制8-10家带名称的公司圆圈。圆圈大小=市场份额。尺寸图例。专业外观" \
  -o figures/13_competitive_positioning.png --doc-type report
```

### 15. 战略群组图

**工具：** scientific-schematics

**提示：**
```
展示竞争者集群的战略群组图：
- X轴：地域范围（区域←→全球）
- Y轴：产品广度（狭窄←→广泛）

绘制4-5个代表战略群组的椭圆形"气泡"：
- 每个气泡包含2-4家公司名称
- 气泡大小代表该群组集体市场份额
- 不同战略群组使用不同颜色

标注每个战略群组：
- "全球综合型"
- "区域专业型"
- "聚焦创新者"
- 等

专业外观
清晰的公司名称标签
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "战略群组图。X轴地域范围（区域到全球），Y轴产品广度（狭窄到广泛）。绘制4-5个战略群组椭圆形气泡。每个气泡含2-4家公司名称。气泡大小=集体市场份额。群组标注：全球综合型，区域专业型，聚焦创新者等。不同群组不同颜色。专业清晰标签" \
  -o figures/14_strategic_groups.png --doc-type report
```

---

## 第五章：客户分析可视化

### 16. 客户细分构成

**工具：** scientific-schematics

**提示：**
```
展示客户细分的树形图或饼图：
- 大型企业：XX%（深蓝）
- 中型市场：XX%（中蓝）
- 中小企业：XX%（浅蓝）
- 个人消费者：XX%（青色）

尺寸代表市场份额
每个细分包含：
- 细分名称
- 百分比
- 美元价值

标题："按市场份额划分的客户细分"
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "客户细分树形图。大型企业XX%深蓝，中型市场XX%中蓝，中小企业XX%浅蓝，个人消费者XX%青色。每个细分显示名称、百分比、美元价值。标题：按市场份额划分的客户细分。专业外观" \
  -o figures/15_customer_segments.png --doc-type report
```

### 17. 细分吸引力矩阵

**工具：** scientific-schematics

**提示：**
```
2x2细分吸引力矩阵：
- X轴：细分规模（小←→大）
- Y轴：增长率（低←→高）

象限标注与行动：
- 右上："优先领域 - 重点投入"
- 左上："投资培育"
- 右下："收割维持"
- 左下："降低优先级"

绘制标注的客户细分圆圈
圆圈大小代表盈利能力
不同细分使用不同颜色
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "2x2细分吸引力矩阵。X轴细分规模（小到大），Y轴增长率（低到高）。象限：右上优先领域重点投入，左上投资培育，右下收割维持，左下降低优先级。绘制客户细分圆圈。圆圈大小=盈利能力。不同颜色。专业外观" \
  -o figures/16_segment_attractiveness.png --doc-type report
```

### 18. 客户旅程地图

**工具：** scientific-schematics

**提示：**
```
展示5-6阶段的水平流程图客户旅程：
认知→考虑→决策→实施→使用→推荐

每个阶段显示三行信息：
1. 关键行为（客户行动）
2. 痛点（面临挑战）
3. 触点（互动方式）

各阶段使用图标
随旅程推进采用浅到深的颜色渐变
专业外观
清晰标签
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "客户旅程水平流程图。5阶段从左至右：认知，考虑，决策，实施，使用，推荐。每阶段下方分行显示关键行为、痛点、触点。各阶段配图标。浅至深颜色渐变。专业清晰标签" \
  -o figures/17_customer_journey.png --doc-type report
```

---

## 第六章：技术格局可视化

### 19. 技术路线图

**工具：** scientific-schematics

**提示：**
```
2024至2030年技术路线图：
三条平行水平轨道：
1. 核心技术（蓝色）- 当前基础
2. 新兴技术（绿色）- 发展中的能力
3. 使能技术（橙色）- 基础设施/支持

每条轨道标注里程碑和技术引入点
垂直线连接跨轨道的相关技术
每年时间标记
技术名称在引入点标注
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "2024至2030年技术路线图。三条平行轨道：核心技术蓝色，新兴技术绿色，使能技术橙色。各轨道标注里程碑和技术引入点。垂直线连接相关技术。年度标记。技术名称标注。专业外观" \
  -o figures/18_technology_roadmap.png --doc-type report
```

### 20. 创新/采用曲线

**工具：** scientific-schematics

**提示：**
```
高德纳技术成熟度曲线或技术采用曲线：
从左至右五个阶段：
1. 技术萌芽期（上升）
2. 期望膨胀期（峰值）
3. 泡沫破裂期（谷底）
4. 稳步爬升期（回升）
5. 生产成熟期（平稳）

在曲线上标注6-8项不同位置的技术
每项技术标注名称
按技术类别颜色编码
专业外观
清晰坐标轴标签
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "高德纳技术成熟度曲线。五阶段：技术萌芽期上升，期望膨胀期峰值，泡沫破裂期谷底，稳步爬升期回升，生产成熟期平稳。曲线上标注6-8项带标签技术。按类别颜色编码。专业清晰标签" \
  -o figures/19_innovation_curve.png --doc-type report
```

---

## 第七章：监管环境可视化

### 21. 监管时间轴

**工具：** scientific-schematics

**提示：**
```
2020至2028年监管时间轴：
带年份标记的水平时间轴
标注关键监管事件：
- 历史法规（深蓝标记，实线）
- 现行法规（当前年份绿色标记）
- 未来法规（浅蓝标记，虚线）

每个标记显示：
- 法规名称
- 生效日期
- 简述（5-7词）

当前年份（2024）垂直线标注"当前"
多辖区需按区域分组
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "2020至2028年监管时间轴。历史法规深蓝实线标记，现行法规绿色标记，未来法规浅蓝虚线标记。每个标记显示法规名称、日期、简述。2024年垂直线标注当前。专业外观清晰标签" \
  -o figures/20_regulatory_timeline.png --doc-type report
```

---

## 第八章：风险分析可视化

### 22. 风险热力图

**工具：** scientific-schematics

**提示：**
```
风险评估热力图/矩阵：
- X轴：影响程度（低→中→高→严重）
- Y轴：发生概率（罕见→可能→较可能→极可能）

单元格颜色梯度：
- 绿色：低风险（低概率，低影响）
- 黄色：中风险
- 橙色：高风险
- 红色：严重风险（高概率，高影响）

在对应单元格绘制10-12个标注风险点/圆圈
风险标签清晰可读
标注风险编号（R1, R2等）
图例关联编号与风险名称
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "风险热力图矩阵。X轴影响程度（低中高严重），Y轴发生概率（罕见可能较可能极可能）。单元格颜色：绿色低风险，黄色中风险，橙色高风险，红色严重风险。绘制10-12个编号风险点R1 R2等。带风险名称的图例。专业清晰" \
  -o figures/21_risk_heatmap.png --doc-type report
```

### 23. 风险应对框架

**工具：** scientific-schematics

**提示：**
```
展示风险与应对措施的关系图：
左列：风险（红/橙色框）
右列：应对策略（绿/蓝色框）

用箭头连接每个风险与对应措施
按类别分组风险（市场、监管、技术等）
包含预防与响应策略

风险严重度通过框体颜色深度表示
专业外观
清晰标签
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "风险应对关系图。左列红/橙色框标注风险。右列绿/蓝色框标注应对策略。箭头连接风险与措施。按类别分组。颜色深度表示风险严重度。包含预防与响应策略。专业清晰标签" \
  -o figures/22_risk_mitigation.png --doc-type report
```

---

## 第九章：战略建议可视化

### 24. 机会矩阵

**工具：** scientific-schematics

**提示：**
```
2x2机会评估矩阵：
- X轴：市场吸引力（低←→高）
- Y轴：制胜能力（低←→高）

象限标注与策略：
- 右上："全力推进"（绿色）
- 左上："构建能力"（黄色）
- 右下："选择性投入"（黄色）
- 左下："规避/剥离"（红色）

绘制6-8

### 26. 实施时间线/甘特图

**工具：** scientific-schematics

**提示：**
```
甘特图风格的24个月实施时间线：
四个阶段显示为水平条形：
- 阶段1：基础建设（第1-6月）- 深蓝色
- 阶段2：构建（第4-12月）- 中蓝色
- 阶段3：扩展（第10-18月）- 青绿色
- 阶段4：优化（第16-24月）- 浅蓝色

阶段按日期显示重叠
关键里程碑在时间线上用菱形标记
X轴显示月份标记
Y轴显示阶段名称
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Gantt chart implementation 24 months. Phase 1 Foundation months 1-6 dark blue. Phase 2 Build months 4-12 medium blue. Phase 3 Scale months 10-18 teal. Phase 4 Optimize months 16-24 light blue. Overlapping bars. Key milestones as diamonds. Month markers X-axis. Professional" \
  -o figures/25_implementation_timeline.png --doc-type report
```

### 27. 里程碑跟踪器

**工具：** scientific-schematics

**提示：**
```
水平时间线上显示8-10个关键里程碑的跟踪器：
每个里程碑包含：
- 日期/月份
- 里程碑名称
- 状态指示器：
  - 已完成：绿色对勾 ✓
  - 进行中：黄色圆圈 ○
  - 待开始：灰色圆圈 ○

按阶段分组里程碑
用时间线连接里程碑
时间线上方标注阶段标签
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Milestone tracker horizontal timeline 8-10 milestones. Each shows date, name, status: Completed green check, In Progress yellow circle, Upcoming gray circle. Group by phase. Phase labels above. Connected timeline line. Professional" \
  -o figures/26_milestone_tracker.png --doc-type report
```

---

## 第11章：投资主题视觉呈现

### 28. 财务预测图表

**工具：** scientific-schematics

**提示：**
```
组合柱状图与折线图显示5年财务预测：
- 柱状图：年度收入（主Y轴，单位：百万美元）
- 折线图：增长率叠加（次Y轴，单位：%）

显示三种情景：
- 保守：灰色柱
- 基准：蓝色柱
- 乐观：绿色柱

X轴：第1年至第5年
柱体添加数据标签
情景与增长线的图例
标题："财务预测（5年期）"
专业外观
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Combined bar and line chart 5-year projections. Bar chart revenue primary Y-axis dollars. Line chart growth rate secondary Y-axis percent. Three scenarios: Conservative gray, Base Case blue, Optimistic green. X-axis Year 1-5. Data labels. Legend. Title Financial Projections 5-Year. Professional" \
  -o figures/27_financial_projections.png --doc-type report
```

### 29. 情景分析对比

**工具：** scientific-schematics

**提示：**
```
分组柱状图对比关键指标在三种情景下的表现：
X轴：指标（第5年收入、第5年EBITDA、市场份额、投资回报率）
Y轴：数值（按指标适配刻度）

每个指标三组柱：
- 保守：灰色
- 基准：蓝色
- 乐观：绿色

每个柱体添加数据标签
情景图例
标题："情景分析对比"
专业外观
清晰的指标标签
```

**命令：**
```bash
python skills/scientific-schematics/scripts/generate_schematic.py \
  "Grouped bar chart scenario comparison. X-axis metrics: Revenue Y5, EBITDA Y5, Market Share, ROI. Three bars per metric: Conservative gray, Base Case blue, Optimistic green. Data labels. Legend. Title Scenario Analysis Comparison. Professional clear labels" \
  -o figures/28_scenario_analysis.png --doc-type report
```

---

## 批量生成脚本

使用 `generate_market_visuals.py` 脚本批量生成视觉内容：

```bash
# 仅生成核心5-6个视觉内容（报告初始推荐）
python skills/market-research-reports/scripts/generate_market_visuals.py \
  --topic "电动汽车充电基础设施" \
  --output-dir figures/

# 生成全部27个视觉内容（核心+扩展，全面覆盖）
python skills/market-research-reports/scripts/generate_market_visuals.py \
  --topic "电动汽车充电基础设施" \
  --output-dir figures/ \
  --all

# 跳过已生成文件
python skills/market-research-reports/scripts/generate_market_visuals.py \
  --topic "您的市场" \
  --output-dir figures/ \
  --skip-existing
```

**默认行为**：仅生成5-6个核心优先视觉内容。如需完整覆盖所有章节，请使用 `--all` 参数。

---

## 质量检查清单

将视觉内容纳入报告前需验证：

- [ ] 所有文字在预期尺寸下清晰可读
- [ ] 所有视觉内容配色一致
- [ ] 配色方案对色盲友好
- [ ] 数据标签准确无误
- [ ] 图例清晰完整
- [ ] 标题描述性强
- [ ] 适用处标注数据来源
- [ ] 分辨率达300 DPI或更高
- [ ] 文件格式为PNG
- [ ] 遵循命名规范
