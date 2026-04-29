# Seaborn 常用场景与示例

本文档提供使用 seaborn 进行数据可视化的实用案例。

## 探索性数据分析

### 快速数据集概览

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

# 加载数据
df = pd.read_csv('data.csv')

# 所有数值变量的成对关系
sns.pairplot(df, hue='target_variable', corner=True, diag_kind='kde')
plt.suptitle('数据集概览', y=1.01)
plt.savefig('overview.png', dpi=300, bbox_inches='tight')
```

### 分布探索

```python
# 跨类别的多重分布
g = sns.displot(
    data=df,
    x='measurement',
    hue='condition',
    col='timepoint',
    kind='kde',
    fill=True,
    height=3,
    aspect=1.5,
    col_wrap=3,
    common_norm=False
)
g.set_axis_labels('测量值', '密度')
g.set_titles('{col_name}')
```

### 相关性分析

```python
# 计算相关矩阵
corr = df.select_dtypes(include='number').corr()

# 创建上三角掩码
mask = np.triu(np.ones_like(corr, dtype=bool))

# 绘制热力图
fig, ax = plt.subplots(figsize=(10, 8))
sns.heatmap(
    corr,
    mask=mask,
    annot=True,
    fmt='.2f',
    cmap='coolwarm',
    center=0,
    square=True,
    linewidths=1,
    cbar_kws={'shrink': 0.8}
)
plt.title('相关矩阵')
plt.tight_layout()
```

## 科研论文图表

### 多面板组合图表

```python
# 设置论文样式
sns.set_theme(style='ticks', context='paper', font_scale=1.1)
sns.set_palette('colorblind')

# 创建自定义布局图表
fig = plt.figure(figsize=(12, 8))
gs = fig.add_gridspec(2, 3, hspace=0.3, wspace=0.3)

# 面板A：时间序列
ax1 = fig.add_subplot(gs[0, :2])
sns.lineplot(
    data=timeseries_df,
    x='time',
    y='expression',
    hue='gene',
    style='treatment',
    markers=True,
    dashes=False,
    ax=ax1
)
ax1.set_title('A. 基因表达随时间变化', loc='left', fontweight='bold')
ax1.set_xlabel('时间（小时）')
ax1.set_ylabel('表达水平（AU）')

# 面板B：分布比较
ax2 = fig.add_subplot(gs[0, 2])
sns.violinplot(
    data=expression_df,
    x='treatment',
    y='expression',
    inner='box',
    ax=ax2
)
ax2.set_title('B. 表达分布', loc='left', fontweight='bold')
ax2.set_xlabel('处理条件')
ax2.set_ylabel('')

# 面板C：相关性
ax3 = fig.add_subplot(gs[1, 0])
sns.scatterplot(
    data=correlation_df,
    x='gene1',
    y='gene2',
    hue='cell_type',
    alpha=0.6,
    ax=ax3
)
sns.regplot(
    data=correlation_df,
    x='gene1',
    y='gene2',
    scatter=False,
    color='black',
    ax=ax3
)
ax3.set_title('C. 基因相关性', loc='left', fontweight='bold')
ax3.set_xlabel('基因1表达量')
ax3.set_ylabel('基因2表达量')

# 面板D：热力图
ax4 = fig.add_subplot(gs[1, 1:])
sns.heatmap(
    sample_matrix,
    cmap='RdBu_r',
    center=0,
    annot=True,
    fmt='.1f',
    cbar_kws={'label': 'Log2倍数变化'},
    ax=ax4
)
ax4.set_title('D. 处理效应', loc='left', fontweight='bold')
ax4.set_xlabel('样本')
ax4.set_ylabel('基因')

# 清理图表
sns.despine()
plt.savefig('figure.pdf', dpi=300, bbox_inches='tight')
plt.savefig('figure.png', dpi=300, bbox_inches='tight')
```

### 带显著性标注的箱线图

```python
import numpy as np
from scipy import stats

# 创建图表
fig, ax = plt.subplots(figsize=(8, 6))
sns.boxplot(
    data=df,
    x='treatment',
    y='response',
    order=['Control', 'Low', 'Medium', 'High'],
    palette='Set2',
    ax=ax
)

# 添加数据点
sns.stripplot(
    data=df,
    x='treatment',
    y='response',
    order=['Control', 'Low', 'Medium', 'High'],
    color='black',
    alpha=0.3,
    size=3,
    ax=ax
)

# 添加显著性标记
def add_significance_bar(ax, x1, x2, y, h, text):
    ax.plot([x1, x1, x2, x2], [y, y+h, y+h, y], 'k-', lw=1.5)
    ax.text((x1+x2)/2, y+h, text, ha='center', va='bottom')

y_max = df['response'].max()
add_significance_bar(ax, 0, 3, y_max + 1, 0.5, '***')
add_significance_bar(ax, 0, 1, y_max + 3, 0.5, 'ns')

ax.set_ylabel('响应值（μM）')
ax.set_xlabel('处理条件')
ax.set_title('处理响应分析')
sns.despine()
```

## 时间序列分析

### 带置信区间的多重时间序列

```python
# 自动聚合绘图
fig, ax = plt.subplots(figsize=(10, 6))
sns.lineplot(
    data=timeseries_df,
    x='timestamp',
    y='value',
    hue='sensor',
    style='location',
    markers=True,
    dashes=False,
    errorbar=('ci', 95),
    ax=ax
)

# 自定义设置
ax.set_xlabel('日期')
ax.set_ylabel('测量值（单位）')
ax.set_title('传感器时间序列测量')
ax.legend(title='传感器 & 位置', bbox_to_anchor=(1.05, 1), loc='upper left')

# 日期格式设置
import matplotlib.dates as mdates
ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
ax.xaxis.set_major_locator(mdates.DayLocator(interval=7))
plt.xticks(rotation=45, ha='right')

plt.tight_layout()
```

### 分面时间序列

```python
# 创建分面时间序列
g = sns.relplot(
    data=long_timeseries,
    x='date',
    y='measurement',
    hue='device',
    col='location',
    row='metric',
    kind='line',
    height=3,
    aspect=2,
    errorbar='sd',
    facet_kws={'sharex': True, 'sharey': False}
)

# 自定义分面标题
g.set_titles('{row_name} - {col_name}')
g.set_axis_labels('日期', '数值')

# 旋转X轴标签
for ax in g.axes.flat:
    ax.tick_params(axis='x', rotation=45)

g.tight_layout()
```

## 分类变量比较

### 嵌套分类变量

```python
# 创建图表
fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 左面板：分组条形图
sns.barplot(
    data=df,
    x='category',
    y='value',
    hue='subcategory',
    errorbar=('ci', 95),
    capsize=0.1,
    ax=axes[0]
)
axes[0].set_title('均值与95%置信区间')
axes[0].set_ylabel('数值（单位）')
axes[0].legend(title='子类别')

# 右面板：小提琴+散点组合图
sns.violinplot(
    data=df,
    x='category',
    y='value',
    hue='subcategory',
    inner=None,
    alpha=0.3,
    ax=axes[1]
)
sns.stripplot(
    data=df,
    x='category',
    y='value',
    hue='subcategory',
    dodge=True,
    size=3,
    alpha=0.6,
    ax=axes[1]
)
axes[1].set_title('个体值分布')
axes[1].set_ylabel('')
axes[1].get_legend().remove()

plt.tight_layout()
```

### 趋势点图

```python
# 展示跨类别数值变化
sns.pointplot(
    data=df,
    x='timepoint',
    y='score',
    hue='treatment',
    markers=['o', 's', '^'],
    linestyles=['-', '--', '-.'],
    dodge=0.3,
    capsize=0.1,
    errorbar=('ci', 95)
)

plt.xlabel('时间点')
plt.ylabel('性能得分')
plt.title('随时间变化的处理效应')
plt.legend(title='处理条件', bbox_to_anchor=(1.05, 1), loc='upper left')
sns.despine()
plt.tight_layout()
```

## 回归与关系分析

### 分面线性回归

```python
# 为每个类别分别拟合回归
g = sns.lmplot(
    data=df,
    x='predictor',
    y='response',
    hue='treatment',
    col='cell_line',
    height=4,
    aspect=1.2,
    scatter_kws={'alpha': 0.5, 's': 50},
    ci=95,
    palette='Set2'
)

g.set_axis_labels('预测变量', '响应变量')
g.set_titles('{col_name}')
g.tight_layout()
```

### 多项式回归

```python
fig, axes = plt.subplots(1, 3, figsize=(15, 5))

for idx, order in enumerate([1, 2, 3]):
    sns.regplot(
        data=df,
        x='x',
        y='y',
        order=order,
        scatter_kws={'alpha': 0.5},
        line_kws={'color': 'red'},
        ci=95,
        ax=axes[idx]
    )
    axes[idx].set_title(f'{order}阶多项式拟合')
    axes[idx].set_xlabel('X变量')
    axes[idx].set_ylabel('Y变量')

plt.tight_layout()
```

### 残差分析

```python
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# 主回归图
sns.regplot(data=df, x='x', y='y', ax=axes[0, 0])
axes[0, 0].set_title('回归拟合')

# 残差 vs 拟合值
sns.residplot(data=df, x='x', y='y', lowess=True,
              scatter_kws={'alpha': 0.5},
              line_kws={'color': 'red', 'lw': 2},
              ax=axes[0, 1])
axes[0, 1].set_title('残差 vs 拟合值')
axes[0, 1].axhline(0, ls='--', color='gray')

# Q-Q图（使用scipy）
from scipy import stats as sp_stats
residuals = df['y'] - np.poly1d(np.polyfit(df['x'], df['y'], 1))(df['x'])
sp_stats.probplot(residuals, dist="norm", plot=axes[1, 0])
axes[1, 0].set_title('Q-Q图')

# 残差直方图
sns.histplot(residuals, kde=True, ax=axes[1, 1])
axes[1, 1].set_title('残差分布')
axes[1, 1].set_xlabel('残差值')

plt.tight_layout()
```

## 双变量与联合分布

### 多形式联合分布图

```python
# 带边缘分布的散点图
g = sns.jointplot(
    data=df,
    x='var1',
    y='var2',
    hue='category',
    kind='scatter',
    height=8,
    ratio=4,
    space=0.1,
    joint_kws={'alpha': 0.5, 's': 50},
    marginal_kws={'kde': True, 'bins': 30}
)

# 添加参考线
g.ax_joint.axline((0, 0), slope=1, color='r', ls='--', alpha=0.5, label='y=x')
g.ax_joint.legend()

g.set_axis_labels('变量1', '变量2', fontsize=12)
```

### KDE等高线图

```python
fig, ax = plt.subplots(figsize=(8, 8))

# 填充等高线双变量KDE
sns.kdeplot(
    data=df,
    x='x',
    y='y',
    fill=True,
    levels=10,
    cmap='viridis',
    thresh=0.05,
    ax=ax
)

# 叠加散点
sns.scatterplot(
    data=df,
    x='x',
    y='y',
    color='white',
    edgecolor='black',
    s=50,
    alpha=0.6,
    ax=ax
)

ax.set_xlabel('X变量')
ax.set_ylabel('Y变量')
ax.set_title('双变量分布')
```

### 六边形分箱边缘图

```python
# 适用于大型数据集
g = sns.jointplot(
    data=large_df,
    x='x',
    y='y',
    kind='hex',
    height=8,
    ratio=5,
    space=0.1,
    joint_kws={'gridsize': 30, 'cmap': 'viridis'},
    marginal_kws={'bins': 50, 'color': 'skyblue'}
)

g.set_axis_labels('X变量', 'Y变量')
```

## 矩阵与热力图可视化

### 层次聚类热力图

```python
# 准备数据（样本×特征）
data_matrix = df.set_index('sample_id')[feature_columns]

# 创建颜色标注
row_colors = df.set_index('sample_id')['condition'].map({
    'control': '#1f77b4',
    'treatment': '#ff7f0e'
})

col_colors = pd.Series(['#2ca02c' if 'gene' in col else '#d62728'
                        for col in data_matrix.columns])

# 绘图
g = sns.clustermap(
    data_matrix,
    method='ward',
    metric='euclidean',
    z_score=0,  # 行标准化
    cmap='RdBu_r',
    center=0,
    row_colors=row_colors,
    col_colors=col_colors,
    figsize=(12, 10),
    dendrogram_ratio=(0.1, 0.1),
    cbar_pos=(0.02, 0.8, 0.03, 0.15),
    linewidths=0.5
)

g.ax_heatmap.set_xlabel('特征')
g.ax_heatmap.set_ylabel('样本')
plt.savefig('clustermap.png', dpi=300, bbox_inches='tight')
```

### 带自定义色标的注释热力图

```python
# 透视数据准备
pivot_data = df.pivot(index='row_var', columns='col_var', values='value')

# 创建热力图
fig, ax = plt.subplots(figsize=(10, 8))
sns.heatmap(
    pivot_data,
    annot=True,
    fmt='.1f',
    cmap='RdYlGn',
    center=pivot_data.mean().mean(),
    vmin=pivot_data.min().min(),
    vmax=pivot_data.max().max(),
    linewidths=0.5,
    linecolor='gray',
    cbar_kws={
        'label': '数值（单位）',
        'orientation': 'vertical',
        'shrink': 0.8,
        'aspect': 20
    },
    ax=ax
)

ax.set_title('变量关系', fontsize=14, pad=20)
ax.set_xlabel('列变量', fontsize=12)
ax.set_ylabel('行变量', fontsize=12)

plt.xticks(rotation=45, ha='right')
plt.yticks(rotation=0

id_vars='subject',
    value_vars=['before', 'after'],
    var_name='timepoint',
    value_name='measurement'
)

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# 左图：个体轨迹
for subject in df_paired['subject'].unique():
    subject_data = df_paired[df_paired['subject'] == subject]
    axes[0].plot(subject_data['timepoint'], subject_data['measurement'],
                 'o-', alpha=0.3, color='gray')

sns.pointplot(
    data=df_paired,
    x='timepoint',
    y='measurement',
    color='red',
    markers='D',
    scale=1.5,
    errorbar=('ci', 95),
    capsize=0.2,
    ax=axes[0]
)
axes[0].set_title('Individual Changes')
axes[0].set_ylabel('Measurement')

# 右图：分布比较
sns.violinplot(
    data=df_paired,
    x='timepoint',
    y='measurement',
    inner='box',
    ax=axes[1]
)
sns.swarmplot(
    data=df_paired,
    x='timepoint',
    y='measurement',
    color='black',
    alpha=0.5,
    size=3,
    ax=axes[1]
)
axes[1].set_title('Distribution Comparison')
axes[1].set_ylabel('')

plt.tight_layout()
```

### 剂量-反应曲线

```python
# 创建剂量-反应图
fig, ax = plt.subplots(figsize=(8, 6))

# 绘制个体数据点
sns.stripplot(
    data=dose_df,
    x='dose',
    y='response',
    order=sorted(dose_df['dose'].unique()),
    color='gray',
    alpha=0.3,
    jitter=0.2,
    ax=ax
)

# 叠加均值及置信区间
sns.pointplot(
    data=dose_df,
    x='dose',
    y='response',
    order=sorted(dose_df['dose'].unique()),
    color='blue',
    markers='o',
    scale=1.2,
    errorbar=('ci', 95),
    capsize=0.1,
    ax=ax
)

# 拟合S型曲线
from scipy.optimize import curve_fit

def sigmoid(x, bottom, top, ec50, hill):
    return bottom + (top - bottom) / (1 + (ec50 / x) ** hill)

doses_numeric = dose_df['dose'].astype(float)
params, _ = curve_fit(sigmoid, doses_numeric, dose_df['response'])

x_smooth = np.logspace(np.log10(doses_numeric.min()),
                       np.log10(doses_numeric.max()), 100)
y_smooth = sigmoid(x_smooth, *params)

ax.plot(range(len(sorted(dose_df['dose'].unique()))),
        sigmoid(sorted(doses_numeric.unique()), *params),
        'r-', linewidth=2, label='Sigmoid Fit')

ax.set_xlabel('Dose')
ax.set_ylabel('Response')
ax.set_title('Dose-Response Analysis')
ax.legend()
sns.despine()
```

## 自定义样式

### 使用十六进制代码自定义调色板

```python
# 定义自定义调色板
custom_palette = ['#E64B35', '#4DBBD5', '#00A087', '#3C5488', '#F39B7F']
sns.set_palette(custom_palette)

# 或用于特定绘图
sns.scatterplot(
    data=df,
    x='x',
    y='y',
    hue='category',
    palette=custom_palette
)
```

### 符合出版要求的主题

```python
# 设置综合主题
sns.set_theme(
    context='paper',
    style='ticks',
    palette='colorblind',
    font='Arial',
    font_scale=1.1,
    rc={
        'figure.dpi': 300,
        'savefig.dpi': 300,
        'savefig.format': 'pdf',
        'axes.linewidth': 1.0,
        'axes.labelweight': 'bold',
        'xtick.major.width': 1.0,
        'ytick.major.width': 1.0,
        'xtick.direction': 'out',
        'ytick.direction': 'out',
        'legend.frameon': False,
        'pdf.fonttype': 42,  # True Type字体用于PDF
    }
)
```

### 以零为中心的发散色彩映射

```python
# 适用于具有意义零点（如对数倍数变化）的数据
from matplotlib.colors import TwoSlopeNorm

# 获取数据范围
vmin, vmax = df['value'].min(), df['value'].max()
vcenter = 0

# 创建归一化对象
norm = TwoSlopeNorm(vmin=vmin, vcenter=vcenter, vmax=vmax)

# 绘图
sns.heatmap(
    pivot_data,
    cmap='RdBu_r',
    norm=norm,
    center=0,
    annot=True,
    fmt='.2f'
)
```

## 大型数据集

### 降采样策略

```python
# 对于超大数据集，进行智能采样
def smart_sample(df, target_size=10000, category_col=None):
    if len(df) <= target_size:
        return df

    if category_col:
        # 分层采样
        return df.groupby(category_col, group_keys=False).apply(
            lambda x: x.sample(min(len(x), target_size // df[category_col].nunique()))
        )
    else:
        # 简单随机采样
        return df.sample(target_size)

# 使用采样数据进行可视化
df_sampled = smart_sample(large_df, target_size=5000, category_col='category')

sns.scatterplot(data=df_sampled, x='x', y='y', hue='category', alpha=0.5)
```

### 针对密集散点图的六边形分箱

```python
# 适用于数百万个点
fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 常规散点图（较慢）
axes[0].scatter(df['x'], df['y'], alpha=0.1, s=1)
axes[0].set_title('Scatter (all points)')

# 六边形分箱（较快）
hb = axes[1].hexbin(df['x'], df['y'], gridsize=50, cmap='viridis', mincnt=1)
axes[1].set_title('Hexbin Aggregation')
plt.colorbar(hb, ax=axes[1], label='Count')

plt.tight_layout()
```

## 笔记本交互元素

### 可调参数

```python
from ipywidgets import interact, FloatSlider

@interact(bandwidth=FloatSlider(min=0.1, max=3.0, step=0.1, value=1.0))
def plot_kde(bandwidth):
    plt.figure(figsize=(10, 6))
    sns.kdeplot(data=df, x='value', hue='category',
                bw_adjust=bandwidth, fill=True)
    plt.title(f'KDE with bandwidth adjustment = {bandwidth}')
    plt.show()
```

### 动态筛选

```python
from ipywidgets import interact, SelectMultiple

categories = df['category'].unique().tolist()

@interact(selected=SelectMultiple(options=categories, value=[categories[0]]))
def filtered_plot(selected):
    filtered_df = df[df['category'].isin(selected)]

    fig, ax = plt.subplots(figsize=(10, 6))
    sns.violinplot(data=filtered_df, x='category', y='value', ax=ax)
    ax.set_title(f'Showing {len(selected)} categories')
    plt.show()
```
