---
name: seaborn
description: 基于pandas的统计可视化工具。用于快速探索分布、关系和分类比较，提供美观的默认设置。最适用于箱线图、小提琴图、配对图和热力图。构建于matplotlib之上。交互式绘图推荐plotly；出版级样式推荐scientific-visualization。
license: BSD-3-Clause许可证
metadata:
    skill-author: K-Dense公司
---

# Seaborn统计可视化

## 概述

Seaborn是一个用于创建出版级统计图形的Python可视化库。通过此工具可实现面向数据集的绘图、多变量分析、自动统计估算，以及用极简代码构建复杂多面板图表。

## 设计理念

Seaborn遵循以下核心原则：

1. **数据集导向**：直接操作DataFrame和命名变量，而非抽象坐标
2. **语义映射**：自动将数据值转换为视觉属性（颜色/大小/样式）
3. **统计感知**：内置聚合、误差估计和置信区间
4. **美学默认值**：开箱即用的出版级主题与调色板
5. **Matplotlib集成**：完全兼容matplotlib自定义功能

## 快速入门

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

# 加载示例数据集
df = sns.load_dataset('tips')

# 创建简单可视化
sns.scatterplot(data=df, x='total_bill', y='tip', hue='day')
plt.show()
```

## 核心绘图接口

### 函数式接口（传统）

函数式接口按可视化类型提供专用绘图函数。每类包含**轴级**函数（单轴绘图）和**图级**函数（支持分面布局）。

**适用场景：**
- 快速探索性分析
- 单一目的可视化
- 需要特定图表类型时

### 对象式接口（现代）

`seaborn.objects`接口提供类似ggplot2的声明式组合API。通过链式方法指定数据映射、标记、变换和比例尺来构建可视化。

**适用场景：**
- 复杂分层可视化
- 需要精细控制变换过程
- 构建自定义图表类型
- 程序化绘图生成

```python
from seaborn import objects as so

# 声明式语法
(
    so.Plot(data=df, x='total_bill', y='tip')
    .add(so.Dot(), color='day')
    .add(so.Line(), so.PolyFit())
)
```

## 分类绘图函数

### 关系图（变量间关系）

**用途：** 探索两个及以上变量的关联性

- `scatterplot()` - 以点显示独立观测值
- `lineplot()` - 展示趋势变化（自动聚合并计算CI）
- `relplot()` - 支持自动分面的图级接口

**关键参数：**
- `x`, `y` - 主变量
- `hue` - 分类/连续变量的颜色编码
- `size` - 点/线尺寸编码
- `style` - 标记/线型编码
- `col`, `row` - 分面子图（仅图级函数）

```python
# 多语义映射散点图
sns.scatterplot(data=df, x='total_bill', y='tip',
                hue='time', size='size', style='sex')

# 带置信区间的折线图
sns.lineplot(data=timeseries, x='date', y='value', hue='category')

# 分面关系图
sns.relplot(data=df, x='total_bill', y='tip',
            col='time', row='sex', hue='smoker', kind='scatter')
```

### 分布图（单变量与双变量分布）

**用途：** 理解数据分布形态与概率密度

- `histplot()` - 灵活分箱的条形频率分布
- `kdeplot()` - 高斯核密度估计
- `ecdfplot()` - 经验累积分布（无需调参）
- `rugplot()` - 观测值刻度标记
- `displot()` - 单/双变量分布的图级接口
- `jointplot()` - 带边缘分布的双变量图
- `pairplot()` - 数据集变量间成对关系矩阵

**关键参数：**
- `x`, `y` - 变量（单变量可省略y）
- `hue` - 按类别分离分布
- `stat` - 标准化方式："count", "frequency", "probability", "density"
- `bins` / `binwidth` - 直方图分箱控制
- `bw_adjust` - KDE带宽乘数（值越大越平滑）
- `fill` - 填充曲线下区域
- `multiple` - 多类别处理方式："layer", "stack", "dodge", "fill"

```python
# 密度标准化的直方图
sns.histplot(data=df, x='total_bill', hue='time',
             stat='density', multiple='stack')

# 带等高线的双变量KDE
sns.kdeplot(data=df, x='total_bill', y='tip',
            fill=True, levels=5, thresh=0.1)

# 带边缘分布的联合图
sns.jointplot(data=df, x='total_bill', y='tip',
              kind='scatter', hue='time')

# 成对关系图
sns.pairplot(data=df, hue='species', corner=True)
```

### 分类图（跨类别比较）

**用途：** 比较离散类别间的分布或统计量

**分类散点图：**
- `stripplot()` - 带抖动显示所有观测点
- `swarmplot()` - 无重叠点（蜂群算法）

**分布比较：**
- `boxplot()` - 四分位距与离群值
- `violinplot()` - KDE+四分位信息
- `boxenplot()` - 大数据集增强箱线图

**统计估计：**
- `barplot()` - 均值/聚合值带置信区间
- `pointplot()` - 点估计值带连接线
- `countplot()` - 类别观测计数

**图级函数：**
- `catplot()` - 分面分类图（通过`kind`参数指定类型）

**关键参数：**
- `x`, `y` - 变量（通常一个为分类变量）
- `hue` - 附加分类分组
- `order`, `hue_order` - 类别排序控制
- `dodge` - 并排显示不同类别
- `orient` - 方向："v"(垂直)或"h"(水平)
- `kind` - catplot图表类型："strip", "swarm", "box", "violin", "bar", "point"

```python
# 显示所有点的蜂群图
sns.swarmplot(data=df, x='day', y='total_bill', hue='sex')

# 用于比较的分裂小提琴图
sns.violinplot(data=df, x='day', y='total_bill',
               hue='sex', split=True)

# 带误差线的条形图
sns.barplot(data=df, x='day', y='total_bill',
            hue='sex', estimator='mean', errorbar='ci')

# 分面分类图
sns.catplot(data=df, x='day', y='total_bill',
            col='time', kind='box')
```

### 回归图（线性关系）

**用途：** 可视化线性回归与残差

- `regplot()` - 轴级回归图（散点+拟合线）
- `lmplot()` - 支持分面的图级回归
- `residplot()` - 评估模型拟合的残差图

**关键参数：**
- `x`, `y` - 回归变量
- `order` - 多项式回归阶数
- `logistic` - 逻辑回归拟合
- `robust` - 稳健回归（降低离群值影响）
- `ci` - 置信区间宽度（默认95）
- `scatter_kws`, `line_kws` - 自定义散点/线属性

```python
# 简单线性回归
sns.regplot(data=df, x='total_bill', y='tip')

# 带分面的多项式回归
sns.lmplot(data=df, x='total_bill', y='tip',
           col='time', order=2, ci=95)

# 残差检验
sns.residplot(data=df, x='total_bill', y='tip')
```

### 矩阵图（矩形数据）

**用途：** 可视化矩阵、相关性与网格结构数据

- `heatmap()` - 带标注的色阶矩阵
- `clustermap()` - 层次聚类热力图

**关键参数：**
- `data` - 二维矩形数据集（DataFrame或数组）
- `annot` - 单元格内显示数值
- `fmt` - 标注格式（如".2f"）
- `cmap` - 色彩映射名称
- `center` - 色彩映射中心值（发散型色阶）
- `vmin`, `vmax` - 色阶范围
- `square` - 强制方形单元格
- `linewidths` - 单元格间距

```python
# 相关性热力图
corr = df.corr()
sns.heatmap(corr, annot=True, fmt='.2f',
            cmap='coolwarm', center=0, square=True)

# 聚类热力图
sns.clustermap(data, cmap='viridis',
               standard_scale=1, figsize=(10, 10))
```

## 多图网格系统

Seaborn提供网格对象构建复杂多面板图表：

### FacetGrid

基于分类变量创建子图。通常通过图级函数（`relplot`, `displot`, `catplot`）调用，也可直接用于自定义绘图。

```python
g = sns.FacetGrid(df, col='time', row='sex', hue='smoker')
g.map(sns.scatterplot, 'total_bill', 'tip')
g.add_legend()
```

### PairGrid

展示数据集所有变量间的成对关系。

```python
g = sns.PairGrid(df, hue='species')
g.map_upper(sns.scatterplot)
g.map_lower(sns.kdeplot)
g.map_diag(sns.histplot)
g.add_legend()
```

### JointGrid

组合双变量图与边缘分布。

```python
g = sns.JointGrid(data=df, x='total_bill', y='tip')
g.plot_joint(sns.scatterplot)
g.plot_marginals(sns.histplot)
```

## 图级函数与轴级函数

理解此区别对高效使用Seaborn至关重要：

### 轴级函数
- 绘制到单个matplotlib `Axes`对象
- 易于集成到复杂matplotlib图形
- 通过`ax=`参数精确定位
- 返回`Axes`对象
- 示例：`scatterplot`, `histplot`, `boxplot`, `regplot`, `heatmap`

**适用场景：**
- 构建自定义多图布局
- 组合不同图表类型
- 需要matplotlib级控制
- 集成现有matplotlib代码

```python
fig, axes = plt.subplots(2, 2, figsize=(10, 10))
sns.scatterplot(data=df, x='x', y='y', ax=axes[0, 0])
sns.histplot(data=df, x='x', ax=axes[0, 1])
sns.boxplot(data=df, x='cat', y='y', ax=axes[1, 0])
sns.kdeplot(data=df, x='x', y='y', ax=axes[1, 1])
```

### 图级函数
- 管理包含所有子图的完整图形
- 通过`col`和`row`参数内置分面功能
- 返回`FacetGrid`, `JointGrid`或`PairGrid`对象
- 使用`height`和`aspect`控制尺寸（按子图）
- 无法嵌入现有图形
- 示例：`relplot`, `displot`, `catplot`, `lmplot`, `jointplot`, `pairplot`

**适用场景：**
- 分面可视化（小倍数图）
- 快速探索性分析
- 统一的多面板布局
- 无需组合其他图表类型

```python
# 自动分面
sns.relplot(data=df, x='x', y='y', col='category', row='group',
            hue='type', height=3, aspect=1.2)
```

## 数据结构要求

### 长格式数据（推荐）

每列代表一个变量，每行代表一个观测值。这种"整洁"格式提供最大灵活性：

```python
# 长格式结构
   受试者  条件      测量值
0      1   对照组     10.5
1      1   实验组     12.3
2      2   对照组      9.8
3      2   实验组     13.1
```

**优势：**
- 兼容所有seaborn函数
- 便于变量重映射到视觉属性
- 支持任意复杂度
- 天然适合DataFrame操作

### 宽格式数据

变量分散在多列中。适用于简单矩形数据：

```python
# 宽格式结构
   对照组  实验组
0   10.5  12.3
1    9.8  13.1
```

**适用场景：**
- 简单时间序列
- 相关矩阵
- 热力图
- 数组数据快速绘图

**宽转长方法：**
```python
df_long = df.melt(var_name='条件', value_name='测量值')
```

## 调色板

Seaborn提供针对不同数据类型的精心设计调色板：

### 定性调色板（分类数据）

通过色调区分类别：
- `"deep"` - 默认鲜艳色系
- `"muted"` - 柔和低饱和度
- `"pastel"` - 浅色低饱和度
- `"bright"` - 高饱和度
- `"dark"` - 深色系
- `"colorblind"` - 色盲友好

```python
sns.set_palette("colorblind")
sns.color_palette("Set2")
```

### 顺序调色板（有序数据）

展示从低到高的数值变化：
- `"rocket"`, `"mako"` - 宽亮度范围（适合热力图）
- `"flare"`, `"crest"` - 受限亮度（适合点/线图）
- `"viridis"`, `"magma"`, `"plasma"` - Matplotlib感知均匀色系

```python
sns.heatmap(data, cmap='rocket')
sns.kdeplot(data=df, x='x', y='y', cmap='mako', fill=True)
```

### 发散调色板（中心化数据）

强调与中点的偏差：
- `"vlag"` - 蓝到红渐变
- `"icefire"` - 蓝到橙渐变
- `"coolwarm"` - 冷到暖渐变
- `"Spectral"` - 彩虹发散色系

```python
sns.heatmap(correlation_matrix, cmap='vlag', center=0)
```

### 自定义调色板

```python
# 创建自定义调色板
custom = sns.color_palette("husl", 8)

# 浅到深渐变
palette = sns.light_palette("seagreen", as_cmap=True)

# 双色调发散调色板
palette = sns.diverging_palette(250, 10, as_cmap=True)
```

## 主题与美学

### 设置主题

`set_theme()`控制整体外观：

```python
# 设置完整主题
sns.set_theme(style='whitegrid', palette='pastel', font='sans-serif')

# 重置默认值
sns.set_theme()
```

### 样式

控制背景与网格外观：
- `"darkgrid"` - 灰底白网格（默认）
- `"whitegrid"` - 白底灰网格
- `"dark"` - 灰底无网格
- `"white"` - 白底无网格
- `"ticks"` - 白底带坐标轴刻度

```python
sns.set_style("whitegrid")

# 移除边框
sns.despine(left=False, bottom=False, offset=10, trim=True)

# 临时样式
with sns.axes_style("white"):
    sns.scatterplot(data=df, x='x', y='y')
```

### 上下文

按使用场景缩放元素：
- `"paper"` - 最小尺寸（默认）
- `"notebook"` - 稍大尺寸
- `"talk"` - 演示文稿尺寸
- `"poster"` - 大幅面尺寸

```python
sns.set_context("talk", font_scale=1.2)

# 临时上下文
with sns.plotting_context("poster"):
    sns.barplot(data=df, x='category', y='value')
```

## 最佳实践

### 1. 数据预处理

始终使用结构良好的DataFrame并赋予有意义的列名：

```python
# 推荐：DataFrame含命名列
df = pd.DataFrame({'账单': bills, '小费': tips, '日期': days})
sns.scatterplot(data=df, x='账单', y='小费', hue='日期')

# 避免：未命名数组

**相关性/矩阵：** `heatmap`、`clustermap`  
**成对关系：** `pairplot`、`jointplot`  

### 3. 使用图形级函数实现分面  

```python
# 替代手动创建子图的方式
sns.relplot(data=df, x='x', y='y', col='category', col_wrap=3)

# 而非：为简单分面手动创建子图
```

### 4. 利用语义映射  

通过 `hue`、`size` 和 `style` 编码额外维度：  

```python
sns.scatterplot(data=df, x='x', y='y',
                hue='category',      # 按类别着色
                size='importance',   # 按连续变量调整大小
                style='type')        # 按类型设置标记样式
```

### 5. 控制统计估计  

多数函数自动计算统计量，需理解并定制：  

```python
# 线图默认计算均值及95%置信区间
sns.lineplot(data=df, x='time', y='value',
             errorbar='sd')  # 改用标准差

# 柱状图默认计算均值
sns.barplot(data=df, x='category', y='value',
            estimator='median',     # 改用中位数
            errorbar=('ci', 95))    # 使用自助法置信区间
```

### 6. 与 Matplotlib 结合  

Seaborn 可与 matplotlib 无缝集成进行精细调整：  

```python
ax = sns.scatterplot(data=df, x='x', y='y')
ax.set(xlabel='自定义X标签', ylabel='自定义Y标签',
       title='自定义标题')
ax.axhline(y=0, color='r', linestyle='--')
plt.tight_layout()
```

### 7. 保存高质量图形  

```python
fig = sns.relplot(data=df, x='x', y='y', col='group')
fig.savefig('figure.png', dpi=300, bbox_inches='tight')
fig.savefig('figure.pdf')  # 矢量格式适用于出版
```

## 常用模式  

### 探索性数据分析  

```python
# 快速查看所有关系
sns.pairplot(data=df, hue='target', corner=True)

# 分布探索
sns.displot(data=df, x='variable', hue='group',
            kind='kde', fill=True, col='category')

# 相关性分析
corr = df.corr()
sns.heatmap(corr, annot=True, cmap='coolwarm', center=0)
```

### 出版级图形  

```python
sns.set_theme(style='ticks', context='paper', font_scale=1.1)

g = sns.catplot(data=df, x='treatment', y='response',
                col='cell_line', kind='box', height=3, aspect=1.2)
g.set_axis_labels('处理条件', '响应值(μM)')
g.set_titles('{col_name}')
sns.despine(trim=True)

g.savefig('figure.pdf', dpi=300, bbox_inches='tight')
```

### 复杂多面板图形  

```python
# 结合 matplotlib 子图与 seaborn
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

sns.scatterplot(data=df, x='x1', y='y', hue='group', ax=axes[0, 0])
sns.histplot(data=df, x='x1', hue='group', ax=axes[0, 1])
sns.violinplot(data=df, x='group', y='y', ax=axes[1, 0])
sns.heatmap(df.pivot_table(values='y', index='x1', columns='x2'),
            ax=axes[1, 1], cmap='viridis')

plt.tight_layout()
```

### 带置信带的时间序列  

```python
# 线图自动聚合并显示置信区间
sns.lineplot(data=timeseries, x='date', y='measurement',
             hue='sensor', style='location', errorbar='sd')

# 更精细控制
g = sns.relplot(data=timeseries, x='date', y='measurement',
                col='location', hue='sensor', kind='line',
                height=4, aspect=1.5, errorbar=('ci', 95))
g.set_axis_labels('日期', '测量值(单位)')
```

## 问题排查  

### 问题：图例超出绘图区域  

图形级函数默认将图例置于外部，调整位置：  

```python
g = sns.relplot(data=df, x='x', y='y', hue='category')
g._legend.set_bbox_to_anchor((0.9, 0.5))  # 调整坐标位置
```

### 问题：标签重叠  

```python
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
```

### 问题：图形过小  

图形级函数：  
```python
sns.relplot(data=df, x='x', y='y', height=6, aspect=1.5)
```

坐标级函数：  
```python
fig, ax = plt.subplots(figsize=(10, 6))
sns.scatterplot(data=df, x='x', y='y', ax=ax)
```

### 问题：颜色区分度不足  

```python
# 更换调色板
sns.set_palette("bright")

# 或指定颜色数量
palette = sns.color_palette("husl", n_colors=len(df['category'].unique()))
sns.scatterplot(data=df, x='x', y='y', hue='category', palette=palette)
```

### 问题：KDE 过平滑或锯齿  

```python
# 调整带宽
sns.kdeplot(data=df, x='x', bw_adjust=0.5)  # 降低平滑度
sns.kdeplot(data=df, x='x', bw_adjust=2)    # 增强平滑度
```

## 资源  

本技能包含深度探索的参考资料：  

### references/  

- `function_reference.md` - 含参数和示例的 seaborn 函数全集  
- `objects_interface.md` - seaborn.objects 现代 API 详解  
- `examples.md` - 不同分析场景的常见用例与代码模式  

按需加载参考文件以获取详细函数签名、高级参数或特定示例。
