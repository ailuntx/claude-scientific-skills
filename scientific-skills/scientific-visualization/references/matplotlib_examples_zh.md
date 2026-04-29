# 可直接用于发表的 Matplotlib 示例

## 概述

本参考提供了使用 Matplotlib、Seaborn 和 Plotly 创建可直接用于发表的科学图表的实用代码示例。所有示例均遵循 `publication_guidelines.md` 中的最佳实践，并使用 `color_palettes.md` 中的色盲友好调色板。

## 设置与配置

### 符合发表质量的 Matplotlib 配置

```python
import matplotlib.pyplot as plt
import matplotlib as mpl
import numpy as np

# 设置发表质量参数
mpl.rcParams['figure.dpi'] = 300
mpl.rcParams['savefig.dpi'] = 300
mpl.rcParams['font.size'] = 8
mpl.rcParams['font.family'] = 'sans-serif'
mpl.rcParams['font.sans-serif'] = ['Arial', 'Helvetica']
mpl.rcParams['axes.labelsize'] = 9
mpl.rcParams['axes.titlesize'] = 9
mpl.rcParams['xtick.labelsize'] = 7
mpl.rcParams['ytick.labelsize'] = 7
mpl.rcParams['legend.fontsize'] = 7
mpl.rcParams['axes.linewidth'] = 0.5
mpl.rcParams['xtick.major.width'] = 0.5
mpl.rcParams['ytick.major.width'] = 0.5
mpl.rcParams['lines.linewidth'] = 1.5

# 使用色盲友好颜色（Okabe-Ito 调

borderwidth=0.5
    )
)

# 保存为静态图像（需安装kaleido）
fig.write_image('plotly_scatter.png', width=500, height=400, scale=3)  # scale=3对应约300 DPI
fig.write_html('plotly_scatter.html')  # 交互式版本

fig.show()
```

## 示例9：带显著性标记的分组柱状图

```python
import matplotlib.pyplot as plt
import numpy as np

# 数据
categories = ['WT', 'Mutant A', 'Mutant B']
control_means = [100, 85, 70]
control_sem = [5, 6, 5]
treatment_means = [100, 120, 140]
treatment_sem = [6, 8, 9]

x = np.arange(len(categories))
width = 0.35

fig, ax = plt.subplots(figsize=(3.5, 3))

# 创建柱状图
bars1 = ax.bar(x - width/2, control_means, width, yerr=control_sem,
               capsize=3, label='Control', color='#0072B2', alpha=0.8)
bars2 = ax.bar(x + width/2, treatment_means, width, yerr=treatment_sem,
               capsize=3, label='Treatment', color='#E69F00', alpha=0.8)

# 添加显著性标记
def add_significance_bar(ax, x1, x2, y, h, text):
    """在两组柱状图间添加显著性标记"""
    ax.plot([x1, x1, x2, x2], [y, y+h, y+h, y], linewidth=0.8, c='black')
    ax.text((x1+x2)/2, y+h, text, ha='center', va='bottom', fontsize=7)

# 标记显著差异
add_significance_bar(ax, x[1]-width/2, x[1]+width/2, 135, 3, '***')
add_significance_bar(ax, x[2]-width/2, x[2]+width/2, 155, 3, '***')

# 自定义设置
ax.set_ylabel('活性（% WT对照组）')
ax.set_xticks(x)
ax.set_xticklabels(categories)
ax.legend(frameon=False, loc='upper left')
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.set_ylim(0, 180)

# 添加显著性说明
ax.text(0.98, 0.02, '*** p < 0.001', transform=ax.transAxes,
        ha='right', va='bottom', fontsize=6)

fig.tight_layout()
save_publication_figure(fig, 'grouped_bar_significance')
plt.show()
```

## 示例10：符合Nature期刊标准的出版级图表

```python
import matplotlib.pyplot as plt
import numpy as np
from string import ascii_lowercase

# Nature规范：89mm单栏宽度
inch_per_mm = 0.0393701
width_mm = 89
height_mm = 110
figsize = (width_mm * inch_per_mm, height_mm * inch_per_mm)

fig = plt.figure(figsize=figsize)
gs = fig.add_gridspec(3, 2, hspace=0.5, wspace=0.4,
                      left=0.12, right=0.95, top=0.96, bottom=0.08)

# 面板a：时间序列
ax_a = fig.add_subplot(gs[0, :])
time = np.linspace(0, 48, 100)
for i, label in enumerate(['Control', 'Treatment']):
    y = (1 + i*0.5) * np.exp(-time/20) * (1 + 0.3*np.sin(time/5))
    ax_a.plot(time, y, linewidth=1.2, label=label)
ax_a.set_xlabel('时间 (h)', fontsize=7)
ax_a.set_ylabel('生长 (OD$_{600}$)', fontsize=7)
ax_a.legend(frameon=False, fontsize=6)
ax_a.tick_params(labelsize=6)
ax_a.spines['top'].set_visible(False)
ax_a.spines['right'].set_visible(False)

# 面板b：柱状图
ax_b = fig.add_subplot(gs[1, 0])
categories = ['A', 'B', 'C']
values = [1.0, 1.5, 2.2]
errors = [0.1, 0.15, 0.2]
ax_b.bar(categories, values, yerr=errors, capsize=2, width=0.6,
         color='#0072B2', alpha=0.8)
ax_b.set_ylabel('倍数变化', fontsize=7)
ax_b.tick_params(labelsize=6)
ax_b.spines['top'].set_visible(False)
ax_b.spines['right'].set_visible(False)

# 面板c：热力图
ax_c = fig.add_subplot(gs[1, 1])
data = np.random.randn(8, 6)
im = ax_c.imshow(data, cmap='viridis', aspect='auto')
ax_c.set_xlabel('样本', fontsize=7)
ax_c.set_ylabel('基因', fontsize=7)
ax_c.tick_params(labelsize=6)

# 面板d：散点图
ax_d = fig.add_subplot(gs[2, :])
x = np.random.randn(50)
y = 2*x + np.random.randn(50)*0.5
ax_d.scatter(x, y, s=8, alpha=0.6, color='#E69F00')
ax_d.set_xlabel('基因X表达量', fontsize=7)
ax_d.set_ylabel('基因Y表达量', fontsize=7)
ax_d.tick_params(labelsize=6)
ax_d.spines['top'].set_visible(False)
ax_d.spines['right'].set_visible(False)

# 添加小写字母面板标签（Nature风格）
for i, ax in enumerate([ax_a, ax_b, ax_c, ax_d]):
    ax.text(-0.2, 1.1, f'{ascii_lowercase[i]}', transform=ax.transAxes,
            fontsize=9, fontweight='bold', va='top')

# 保存为Nature推荐格式
fig.savefig('nature_figure.pdf', dpi=1000, bbox_inches='tight',
           facecolor='white', edgecolor='none')
fig.savefig('nature_figure.png', dpi=300, bbox_inches='tight',
           facecolor='white', edgecolor='none')

plt.show()
```

## 各库使用技巧

### Matplotlib
- 使用`fig.tight_layout()`或`constrained_layout=True`防止元素重叠
- 设置DPI为300-600满足出版要求
- 线图优先使用矢量格式（PDF/EPS）
- PDF/EPS文件中需嵌入字体

### Seaborn
- 基于matplotlib，所有matplotlib定制方法均适用
- 使用`sns.set_style('ticks')`或`'whitegrid'`保持简洁外观
- `sns.despine()`移除顶部和右侧边框
- 通过`sns.set_palette()`设置自定义调色板

### Plotly
- 适用于交互式探索分析
- 通过`fig.write_image()`导出静态图像（需kaleido包）
- 用`scale`参数控制DPI（scale=3≈300 DPI）
- 深度调整布局参数满足出版质量要求

## 通用工作流程

1. **使用默认设置进行探索**
2. **应用出版级配置**（参考设置章节）
3. **创建符合尺寸要求的图表**（查阅期刊规范）
4. **定制颜色方案**（使用色盲友好调色板）
5. **调整字体和线宽**（确保最终尺寸下可读）
6. **移除冗余元素**（顶部/右侧边框、过度网格线）
7. **添加带单位的清晰标签**
8. **进行灰度测试**
9. **保存多种格式**（矢量图用PDF，栅格图用PNG）
10. **在最终环境中验证**（导入文稿检查尺寸）

## 资源

- Matplotlib文档：https://matplotlib.org/
- Seaborn图库：https://seaborn.pydata.org/examples/index.html
- Plotly文档：https://plotly.com/python/
- Nature Methods观点专栏：数据可视化存档
