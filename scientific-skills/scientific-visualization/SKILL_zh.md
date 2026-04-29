---
name: scientific-visualization
description: 用于生成可发表级别的图表的元技能。在创建需要多面板布局、显著性标注、误差线、色盲友好调色板以及特定期刊格式（Nature, Science, Cell）的投稿图表时使用。通过出版物样式协调 matplotlib/seaborn/plotly。快速探索时请直接使用 seaborn 或 plotly。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# 科学可视化

## 概述

科学可视化将数据转化为清晰、准确的图表以供发表。创建具备多面板布局、误差线、显著性标记和色盲友好调色板的期刊就绪图。使用 matplotlib、seaborn 和 plotly 导出为 PDF/EPS/TIFF 格式用于手稿。

## 何时使用本技能

本技能应在以下情况使用：
- 为科学手稿创建图表或可视化
- 准备提交期刊的图表（Nature, Science, Cell, PLOS 等）
- 确保图表色盲友好且易于访问
- 制作样式统一的多面板图表
- 以正确的分辨率和格式导出图表
- 遵循特定的出版指南
- 改进现有图表以满足出版标准
- 创建需要在彩色和灰度下均能使用的图表

## 快速入门指南

### 基础出版级图表

```python
import matplotlib.pyplot as plt
import numpy as np

# 应用出版样式（来自 scripts/style_presets.py）
from style_presets import apply_publication_style
apply_publication_style('default')

# 创建合适尺寸的图表（单栏 = 3.5 英寸）
fig, ax = plt.subplots(figsize=(3.5, 2.5))

# 绘制数据
x = np.linspace(0, 10, 100)
ax.plot(x, np.sin(x), label='sin(x)')
ax.plot(x, np.cos(x), label='cos(x)')

# 正确标注并带单位
ax.set_xlabel('Time (seconds)')
ax.set_ylabel('Amplitude (mV)')
ax.legend(frameon=False)

# 移除不必要的边框
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

# 以出版格式保存（来自 scripts/figure_export.py）
from figure_export import save_publication_figure
save_publication_figure(fig, 'figure1', formats=['pdf', 'png'], dpi=300)
```

### 使用预配置样式

使用 `assets/` 中的 matplotlib 样式文件应用特定期刊样式：

```python
import matplotlib.pyplot as plt

# 选项 1：直接使用样式文件
plt.style.use('assets/nature.mplstyle')

# 选项 2：使用 style_presets.py 辅助函数
from style_presets import configure_for_journal
configure_for_journal('nature', figure_width='single')

# 现在创建图表——它们将自动符合 Nature 规格
fig, ax = plt.subplots()
# ... 你的绘图代码 ...
```

### 快速开始使用 Seaborn

对于统计图，使用带出版样式的 seaborn：

```python
import seaborn as sns
import matplotlib.pyplot as plt
from style_presets import apply_publication_style

# 应用出版样式
apply_publication_style('default')
sns.set_theme(style='ticks', context='paper', font_scale=1.1)
sns.set_palette('colorblind')

# 创建统计比较图
fig, ax = plt.subplots(figsize=(3.5, 3))
sns.boxplot(data=df, x='treatment', y='response', 
            order=['Control', 'Low', 'High'], palette='Set2', ax=ax)
sns.stripplot(data=df, x='treatment', y='response',
              order=['Control', 'Low', 'High'], 
              color='black', alpha=0.3, size=3, ax=ax)
ax.set_ylabel('Response (μM)')
sns.despine()

# 保存图表
from figure_export import save_publication_figure
save_publication_figure(fig, 'treatment_comparison', formats=['pdf', 'png'], dpi=300)
```

## 核心原则与最佳实践

### 1. 分辨率与文件格式

**关键要求**（详见 `references/publication_guidelines.md`）：
- **光栅图像**（照片、显微图像）：300-600 DPI
- **线条图**（曲线图、图表）：600-1200 DPI 或矢量格式
- **矢量格式**（推荐）：PDF、EPS、SVG
- **光栅格式**：TIFF、PNG（科学数据绝不使用 JPEG）

**实现：**
```python
# 使用 figure_export.py 脚本获取正确设置
from figure_export import save_publication_figure

# 以多种格式及正确 DPI 保存
save_publication_figure(fig, 'myfigure', formats=['pdf', 'png'], dpi=300)

# 或根据特定期刊要求保存
from figure_export import save_for_journal
save_for_journal(fig, 'figure1', journal='nature', figure_type='combination')
```

### 2. 颜色选择 - 色盲可访问性

**始终使用色盲友好调色板**（详见 `references/color_palettes.md`）：

**推荐：Okabe-Ito 调色板**（所有色盲类型均可区分）：
```python
# 选项 1：使用 assets/color_palettes.py
from color_palettes import OKABE_ITO_LIST, apply_palette
apply_palette('okabe_ito')

# 选项 2：手动指定
okabe_ito = ['#E69F00', '#56B4E9', '#009E73', '#F0E442',
             '#0072B2', '#D55E00', '#CC79A7', '#000000']
plt.rcParams['axes.prop_cycle'] = plt.cycler(color=okabe_ito)
```

**对于热图/连续数据：**
- 使用感知均匀的颜色图：`viridis`、`plasma`、`cividis`
- 避免红-绿发散型颜色图（改用 `PuOr`、`RdBu`、`BrBG`）
- 切勿使用 `jet` 或 `rainbow` 颜色图

**始终测试图表在灰度下的可解释性。**

### 3. 排版与文本

**字体指南**（详见 `references/publication_guidelines.md`）：
- 无衬线字体：Arial、Helvetica、Calibri
- **最终打印尺寸**下的最小字号：
  - 轴标签：7-9 pt
  - 刻度标签：6-8 pt
  - 面板标签：8-12 pt（加粗）
- 标签采用句首大写： "Time (hours)" 而非 "TIME (HOURS)"
- 始终在括号内包含单位

**实现：**
```python
# 全局设置字体
import matplotlib as mpl
mpl.rcParams['font.family'] = 'sans-serif'
mpl.rcParams['font.sans-serif'] = ['Arial', 'Helvetica']
mpl.rcParams['font.size'] = 8
mpl.rcParams['axes.labelsize'] = 9
mpl.rcParams['xtick.labelsize'] = 7
mpl.rcParams['ytick.labelsize'] = 7
```

### 4. 图表尺寸

**期刊特定宽度**（详见 `references/journal_requirements.md`）：
- **Nature**：单栏 89 mm，双栏 183 mm
- **Science**：单栏 55 mm，双栏 175 mm
- **Cell**：单栏 85 mm，双栏 178 mm

**检查图表尺寸合规性：**
```python
from figure_export import check_figure_size

fig = plt.figure(figsize=(3.5, 3))  # Nature 的 89 mm
check_figure_size(fig, journal='nature')
```

### 5. 多面板图表

**最佳实践：**
- 用加粗字母标注面板：**A**、**B**、**C**（大多数期刊大写，Nature 小写）
- 保持所有面板样式一致
- 尽可能对齐面板边缘
- 面板间留出足够的空白

**示例实现**（完整代码见 `references/matplotlib_examples.md`）：
```python
from string import ascii_uppercase

fig = plt.figure(figsize=(7, 4))
gs = fig.add_gridspec(2, 2, hspace=0.4, wspace=0.4)

ax1 = fig.add_subplot(gs[0, 0])
ax2 = fig.add_subplot(gs[0, 1])
# ... 创建其他面板 ...

# 添加面板标签
for i, ax in enumerate([ax1, ax2, ...]):
    ax.text(-0.15, 1.05, ascii_uppercase[i], transform=ax.transAxes,
            fontsize=10, fontweight='bold', va='top')
```

## 常见任务

### 任务 1：创建出版就绪的线图

完整代码见 `references/matplotlib_examples.md` 示例 1。

**关键步骤：**
1. 应用出版样式
2. 为目标期刊设置合适的图表尺寸
3. 使用色盲友好颜色
4. 添加正确表示的误差线（SEM、SD 或 CI）
5. 标注轴并带单位
6. 移除不必要的边框
7. 以矢量格式保存

**使用 seaborn 自动计算置信区间：**
```python
import seaborn as sns
fig, ax = plt.subplots(figsize=(5, 3))
sns.lineplot(data=timeseries, x='time', y='measurement',
             hue='treatment', errorbar=('ci', 95), 
             markers=True, ax=ax)
ax.set_xlabel('Time (hours)')
ax.set_ylabel('Measurement (AU)')
sns.despine()
```

### 任务 2：创建多面板图表

完整代码见 `references/matplotlib_examples.md` 示例 2。

**关键步骤：**
1. 使用 `GridSpec` 实现灵活布局
2. 确保各面板样式一致
3. 添加加粗面板标签（A、B、C 等）
4. 对齐相关面板
5. 确认所有文本在最终尺寸下可读

### 任务 3：使用正确颜色图创建热图

完整代码见 `references/matplotlib_examples.md` 示例 4。

**关键步骤：**
1. 使用感知均匀的颜色图（`viridis`、`plasma`、`cividis`）
2. 包含带标签的颜色条
3. 对于发散数据，使用色盲安全的发散颜色图（`RdBu_r`、`PuOr`）
4. 为发散颜色图设置合适的中心值
5. 测试灰度下的外观

**使用 seaborn 创建相关矩阵：**
```python
import seaborn as sns
fig, ax = plt.subplots(figsize=(5, 4))
corr = df.corr()
mask = np.triu(np.ones_like(corr, dtype=bool))
sns.heatmap(corr, mask=mask, annot=True, fmt='.2f',
            cmap='RdBu_r', center=0, square=True,
            linewidths=1, cbar_kws={'shrink': 0.8}, ax=ax)
```

### 任务 4：为特定期刊准备图表

**工作流程：**
1. 查阅期刊要求：`references/journal_requirements.md`
2. 为期刊配置 matplotlib：
   ```python
   from style_presets import configure_for_journal
   configure_for_journal('nature', figure_width='single')
   ```
3. 创建图表（将自动调整正确大小）
4. 按期刊规范导出：
   ```python
   from figure_export import save_for_journal
   save_for_journal(fig, 'figure1', journal='nature', figure_type='line_art')
   ```

### 任务 5：改进现有图表以满足出版标准

**清单方法**（完整清单见 `references/publication_guidelines.md`）：

1. **检查分辨率**：确认 DPI 满足期刊要求
2. **检查文件格式**：图表使用矢量，图像使用 TIFF/PNG
3. **检查颜色**：确保色盲友好
4. **检查字体**：最终尺寸下最小 6-7 pt，无衬线字体
5. **检查标签**：所有轴带单位标签
6. **检查尺寸**：符合期刊栏宽
7. **测试灰度**：图表在无颜色时可解释
8. **移除图表垃圾**：无多余网格、3D 效果、阴影

### 任务 6：创建色盲友好的可视化

**策略：**
1. 使用 `assets/color_palettes.py` 中的认可调色板
2. 添加冗余编码（线型、标记、图案）
3. 使用色盲模拟器测试
4. 确保灰度兼容性

**示例：**
```python
from color_palettes import apply_palette
import matplotlib.pyplot as plt

apply_palette('okabe_ito')

# 在颜色之外添加冗余编码
line_styles = ['-', '--', '-.', ':']
markers = ['o', 's', '^', 'v']

for i, (data, label) in enumerate(datasets):
    plt.plot(x, data, linestyle=line_styles[i % 4],
             marker=markers[i % 4], label=label)
```

## 统计严谨性

**始终包含：**
- 误差线（SD、SEM 或 CI——在图注中说明）
- 样本量 (n) 在图表或图注中
- 统计显著性标记 (*, **, ***)
- 尽可能显示单个数据点（不仅仅是汇总统计）

**带统计的示例：**
```python
# 显示带汇总统计的单个点
ax.scatter(x_jittered, individual_points, alpha=0.4, s=8)
ax.errorbar(x, means, yerr=sems, fmt='o', capsize=3)

# 标记显著性
ax.text(1.5, max_y * 1.1, '***', ha='center', fontsize=8)
```

## 使用不同绘图库

### Matplotlib
- 对出版细节控制最多
- 最适合复杂的多面板图表
- 使用提供的样式文件保持格式一致
- 大量示例见 `references/matplotlib_examples.md`

### Seaborn

Seaborn 提供了一个高层级、面向数据集的统计图形接口，基于 matplotlib 构建。它擅长以最少的代码创建出版质量的统计可视化，同时保持与 matplotlib 自定义的完全兼容。

**科学可视化的关键优势：**
- 自动统计估计和置信区间
- 内置多面板图表支持（分面）
- 默认提供色盲友好调色板
- 基于 pandas DataFrame 的数据集导向 API
- 将变量语义映射到视觉属性

#### 快速开始使用出版样式

始终先应用 matplotlib 出版样式，再配置 seaborn：

```python
import seaborn as sns
import matplotlib.pyplot as plt
from style_presets import apply_publication_style

# 应用出版样式
apply_publication_style('default')

# 为出版配置 seaborn
sns.set_theme(style='ticks', context='paper', font_scale=1.1)
sns.set_palette('colorblind')  # 使用色盲安全调色板

# 创建图表
fig, ax = plt.subplots(figsize=(3.5, 2.5))
sns.scatterplot(data=df, x='time', y='response', 
                hue='treatment', style='condition', ax=ax)
sns.despine()  # 移除顶部和右侧边框
```

#### 用于出版的常见图表类型

**统计比较：**
```python
# 带单个点的箱线图以提高透明度
fig, ax = plt.subplots(figsize=(3.5, 3))
sns.boxplot(data=df, x='treatment', y='response', 
            order=['Control', 'Low', 'High'], palette='Set2', ax=ax)
sns.stripplot(data=df, x='treatment', y='response',
              order=['Control', 'Low', 'High'], 
              color='black', alpha=0.3, size=3, ax=ax)
ax.set_ylabel('Response (μM)')
sns.despine()
```

**分布分析：**
```python
# 带分割比较的提琴图
fig, ax = plt.subplots(figsize=(4, 3))
sns.violinplot(data=df, x='timepoint', y='expression',
               hue='treatment', split=True, inner='quartile', ax=ax)
ax.set_ylabel('Gene Expression (AU)')
sns.despine()
```

**相关矩阵：**
```python
# 使用正确颜色图和标注的热图
fig, ax = plt.subplots(figsize=(5, 4))
corr = df.corr()
mask = np.triu(np.ones_like(corr, dtype=bool))  # 仅显示下三角
sns.heatmap(corr, mask=mask, annot=True, fmt='.2f',
            cmap='RdBu_r', center=0, square=True,
            linewidths=1, cbar_kws={'shrink': 0.8}, ax=ax)
plt.tight_layout()
```

**带置信带的时间序列：**
```python
# 带自动 CI 计算的线图
fig, ax = plt.subplots(figsize=(5, 3))
sns.lineplot(data=timeseries, x='time', y='measurement',
             hue='treatment', style='replicate',
             errorbar=('ci', 95), markers=True, dashes=False, ax=ax)
ax.set_xlabel('Time (hours)')
ax.set_ylabel('Measurement (AU)')
sns.despine()
```

#### 使用 Seaborn 的多面板图表

**使用 FacetGrid 自动分面：**
```python
# 创建分面图
g = sns.relplot(data=df, x='dose', y='response',
                hue='treatment', col='cell_line', row='timepoint',
                kind='line', height=2.5, aspect=1.2,
                errorbar=('ci', 95), markers=True)
g.set_axis_labels('Dose (μM)', 'Response (AU)')
g.set_titles('{row_name} | {col_name}')
sns.despine()

# 以正确 DPI 保存
from figure_export import save_publication_figure
save_publication_figure(g.figure, 'figure_facets', 
                       formats=['pdf', 'png'], dpi=300)
```

**结合 seaborn 与 matplotlib 子图：**
```python
# 创建自定义多面板布局
fig, axes = plt.subplots(2, 2, figsize=(7, 6))

# 面板 A：带回归的散点图
sns.regplot(data=df, x='predictor', y='response', ax=axes[0, 0])
axes[0, 0].text(-0.15, 1.05, 'A', transform=axes[0, 0].transAxes,
                fontsize=10, fontweight='bold')

# 面板 B：分布比较
sns.violinplot(data=df, x='group', y='value', ax=axes[0, 1])
axes[0, 1].text(-0.15, 1.05, 'B', transform=axes[0, 1].transAxes,
                fontsize=10, fontweight='bold')

# 面板 C：热图
sns.heatmap(correlation_data, cmap='viridis', ax=axes[1, 0])
axes[1, 0].text(-0.15, 1.05, 'C', transform=axes[1, 0].transAxes,
                fontsize=10, fontweight='bold')

# 面板 D：时间序列
sns.lineplot(data=timeseries, x='time', y='signal', 
             hue='condition', ax=axes[1, 1])
axes[1, 1].text(-0.15, 1.05, 'D', transform=axes[1, 1].transAxes,
                fontsize=10, fontweight='bold')

plt.tight_layout()
sns.despine()
```

#### 用于出版的调色板

Seaborn 包含多个色盲安全调色板：

```python
# 使用内置色盲调色板（推荐）
sns.set_palette('colorblind')

# 或指定自定义色盲安全颜色（Okabe-Ito）
okabe_ito = ['#E69F00', '#56B4E9', '#009E73', '#F0E442',
             '#0072B2', '#D55E00', '#CC79A7', '#000000']
sns.set_palette(okabe_ito)

# 对于热图和连续数据
sns.heatmap(data, cmap='viridis')  # 感知均匀
sns.heatmap(corr, cmap='RdBu_r', center=0)  # 发散，居中
```

#### 选择轴级函数与图级函数

**轴级函数**（例如 `scatterplot`、`boxplot`、`heatmap`）：
- 在构建自定义多面板布局时使用
- 接受 `ax=` 参数以便精确定位
- 与 matplotlib 子图更好地集成
- 对图表组成有更多控制

```python
fig, ax = plt.subplots(figsize=(3.5, 2.5))
sns.scatterplot(data=df, x='x', y='y', hue='group', ax=ax)
```

**图级函数**（例如 `relplot`、`catplot`、`displot`）：
- 用于按分类变量自动分面
- 创建样式一致的完整图表
- 非常适合探索性分析
- 使用 `height` 和 `aspect` 控制尺寸

```python
g = sns.relplot(data=df, x='x', y='y', col='category', kind='scatter')
```

#### 使用 Seaborn 的统计严谨性

Seaborn 自动计算并显示不确定性：

```python
# 线图：默认显示均值 ± 95% CI
sns.lineplot(data=df, x='time', y='value', hue='treatment',
             errorbar=('ci', 95))  # 可改为 'sd', 'se' 等

# 条形图：显示均值及自助法 CI
sns.barplot(data=df, x='treatment', y='response',
            errorbar=('ci', 95), capsize=0.1)

# 始终在图注中说明误差类型：
# "Error bars represent 95% confidence intervals"
```

#### 出版就绪 Seaborn 图表的最佳实践

1. **始终先设置出版主题：**
   ```python
   sns.set_theme(style='ticks', context='paper', font_scale=1.1)
   ```

2. **使用色盲安全调色板：**
   ```python
   sns.set_palette('colorblind')
   ```

3. **移除不必要的元素：**
   ```python
   sns.despine()  # 移除顶部和右侧边框
   ```

4. **适当控制图表尺寸：**
   ```python
   # 轴级：使用 matplotlib figsize
   fig, ax = plt.subplots(figsize=(3.5, 2.5))
   
   # 图级：使用 height 和 aspect
   g = sns.relplot(..., height=3, aspect=1.2)
   ```

5. **尽可能显示单个数据点：**
   ```python
   sns.boxplot(...)  # 汇总统计
   sns.stripplot(..., alpha=0.3)  # 单个点
   ```

6. **包含带单位的正确标签：**
   ```python
   ax.set_xlabel('Time (hours)')
   ax.set_ylabel('Expression (AU)')
   ```

7. **以正确分辨率导出：**
   ```python
   from figure_export import save_publication_figure
   save_publication_figure(fig, 'figure_name', 
                          formats=['pdf', 'png'], dpi=300)
   ```

#### 高级 Seaborn 技巧

**成对关系用于探索性分析：**
```python
# 快速概览所有关系
g = sns.pairplot(data=df, hue='condition', 
                 vars=['gene1', 'gene2', 'gene3'],
                 corner=True, diag_kind='kde', height=2)
```

**层次聚类热图：**
```python
# 对样本和特征聚类
g = sns.clustermap(expression_data, method='ward', 
                   metric='euclidean', z_score=0,
                   cmap='RdBu_r', center=0, 
                   figsize=(10, 8), 
                   row_colors=condition_colors,
                   cbar_kws={'label': 'Z-score'})
```

**带边缘分布的联合分布：**
```python
# 带背景的双变量分布
g = sns.jointplot(data=df, x='gene1', y='gene2',
                  hue='treatment', kind='scatter',
                  height=6, ratio=4, marginal_kws={'kde': True})
```

#### 常见 Seaborn 问题及解决方案

**问题：图例位于绘图区域外**
```python
g = sns.relplot(...)
g._legend.set_bbox_to_anchor((0.9, 0.5))
```

**问题：标签重叠**
```python
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
```

**问题：最终尺寸下文本太小**
```python
sns.set_context('paper', font_scale=1.2)  # 如需要可增大
```

#### 其他资源

更多关于 seaborn 的详细信息，请参阅：
- `scientific-skills/seaborn/SKILL.md` - 全面的 seaborn 文档
- `scientific-skills/seaborn/references/examples.md` - 实际用例
- `scientific-skills/seaborn/references/function_reference.md` - 完整 API 参考
- `scientific-skills/seaborn/references/objects_interface.md` - 现代声明式 API

### Plotly
- 用于探索的交互式图表
- 导出静态图像用于出版
- 配置为出版质量：
```python
fig.update_layout(
    font=dict(family='Arial, sans-serif', size=10),
    plot_bgcolor='white',
    # ... 见 matplotlib_examples.md 示例 8
)
fig.write_image('figure.png', scale=3)  # scale=3 大约得到 300 DPI
```

## 资源

### 参考文档目录

**需要时加载以获取详细信息：**

- **`publication_guidelines.md`**：全面的最佳实践
  - 分辨率和文件格式要求
  - 排版指南
  - 布局与构成规则
  - 统计严谨性要求
  - 完整出版检查清单

- **`color_palettes.md`**：颜色使用指南
  - 包含 RGB 值的色盲友好调色板规范
  - 顺序和发散颜色图建议
  - 可访问性测试流程
  - 特定领域调色板（基因组学、显微镜）

- **`journal_requirements.md`**：期刊特定规范
  - 按出版商的 technical 要求
  - 文件格式和 DPI 规范
  - 图表尺寸要求
  - 快速参考表

- **`matplotlib_examples.md`**：实用代码示例
  - 10 个完整工作示例
  - 线图、条形图、热图、多面板图表
  - 期刊特定图表示例
  - 各库的使用技巧（matplotlib、seaborn、plotly）

### 脚本目录

**使用这些辅助脚本实现自动化：**

- **`figure_export.py`**：导出工具
  - `save_publication_figure()`：以多种格式及正确 DPI 保存
  - `save_for_journal()`：自动使用期刊特定要求
  - `check_figure_size()`：验证尺寸是否符合期刊规范
  - 直接运行：`python scripts/figure_export.py` 以查看示例

- **`style_presets.py`**：预配置样式
  - `apply_publication_style()`：应用预设样式（default、nature、science、cell）
  - `set_color_palette()`：快速切换调色板
  - `configure_for_journal()`：一键期刊配置
  - 直接运行：`python scripts/style_presets.py` 以查看示例

### 资源目录

**在图表中使用这些文件：**

- **`color_palettes.py`**：可导入的颜色定义
  - 所有推荐调色板作为 Python 常量
  - `apply_palette()` 辅助函数
  - 可直接导入到笔记本/脚本中

- **Matplotlib 样式文件**：使用 `plt.style.use()`
  - `publication.mplstyle`：通用出版质量
  - `nature.mplstyle`：Nature 期刊规范
  - `presentation.mplstyle`：海报/幻灯片用大字号

## 工作流程总结

**推荐创建出版图表的工作流程：**

1. **计划**：确定目标期刊、图表类型和内容
2. **配置**：应用适合期刊的样式
   ```python
   from style_presets import configure_for_journal
   configure_for_journal('nature', 'single')
   ```
3. **创建**：用正确的标签、颜色、统计构建图表
4. **验证**：检查尺寸、字体、颜色、可访问性
   ```python
   from figure_export import check_figure_size
   check_figure_size(fig, journal='nature')
   ```
5. **导出**：以所需格式保存
   ```python
   from figure_export import save_for_journal
   save_for_journal(fig, 'figure1', 'nature', 'combination')
   ```
6. **审阅**：在手稿上下文中以最终尺寸查看

## 应避免的常见陷阱

1. **字体过小**：打印为最终尺寸时文本不可读
2. **JPEG 格式**：切勿对图形/图表使用 JPEG（产生伪影）
3. **红绿色**：约 8% 的男性无法区分
4. **低分辨率**：出版时图表像素化
5. **缺少单位**：始终为轴标注单位
6. **3D 效果**：扭曲感知，完全避免
7. **图表垃圾**：移除多余网格线、装饰
8. **截断轴**：条形图从零开始，除非有科学依据
9. **样式不一致**：同一手稿中不同图表使用不同字体/颜色
10. **无误差线**：始终显示不确定性

## 最终检查清单

提交图表前，验证：

- [ ] 分辨率满足期刊要求（300+ DPI）
- [ ] 文件格式正确（图表用矢量，图像用 TIFF）
- [ ] 图表尺寸符合期刊规范
- [ ] 所有文本在最终尺寸下可读（≥6 pt）
- [ ] 颜色色盲友好
- [ ] 图表在灰度下可用
- [ ] 所有轴标注了单位
- [ ] 存在误差线并在图注中定义
- [ ] 面板标签存在且一致
- [ ] 无图表垃圾或 3D 效果
- [ ] 所有图表字体一致
- [ ] 统计显著性清晰标注
- [ ] 图例清晰完整

使用本技能确保科学图表达到最高出版标准，同时保持对所有读者的可访问性。
