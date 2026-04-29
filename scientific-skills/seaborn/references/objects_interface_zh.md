# Seaborn 对象接口

`seaborn.objects` 接口提供了一套现代化的声明式 API，通过组合方式构建可视化图表。本指南涵盖 seaborn 0.12+ 引入的完整对象接口。

## 核心概念

对象接口将**展示内容**（数据与映射）与**展示方式**（标记、统计与变换）分离。构建图表流程：

1. 创建包含数据和美学映射的 `Plot` 对象
2. 通过 `.add()` 添加结合标记与统计变换的图层
3. 使用 `.scale()`、`.label()`、`.limit()`、`.theme()` 等方法定制
4. 通过 `.show()` 或 `.save()` 渲染输出

## 基础用法

```python
from seaborn import objects as so
import pandas as pd

# 创建包含数据和映射的绘图对象
p = so.Plot(data=df, x='x_var', y='y_var')

# 添加标记（视觉表现形式）
p = p.add(so.Dot())

# 显示图表（Jupyter 环境自动渲染）
p.show()
```

## Plot 类

`Plot` 类是对象接口的基础组件。

### 初始化

```python
so.Plot(data=None, x=None, y=None, color=None, alpha=None,
        fill=None, fillalpha=None, fillcolor=None, marker=None,
        pointsize=None, stroke=None, text=None, **variables)
```

**参数说明：**
- `data` - DataFrame 或数据向量字典
- `x, y` - 位置变量
- `color` - 颜色编码变量
- `alpha` - 透明度变量
- `marker` - 标记形状变量
- `pointsize` - 点尺寸变量
- `stroke` - 线宽变量
- `text` - 文本标签变量
- `**variables` - 使用属性名的附加映射

**示例：**
```python
# 基础映射
so.Plot(df, x='total_bill', y='tip')

# 多重映射
so.Plot(df, x='total_bill', y='tip', color='day', pointsize='size')

# 所有变量在 Plot 中定义
p = so.Plot(df, x='x', y='y', color='cat')
p.add(so.Dot())  # 使用全部映射

# 部分变量在 add() 中定义
p = so.Plot(df, x='x', y='y')
p.add(so.Dot(), color='cat')  # 仅当前图层使用颜色映射
```

### 方法

#### add()

添加包含标记及可选统计/变换的图层。

```python
Plot.add(mark, *transforms, orient=None, legend=True, data=None,
         **variables)
```

**参数说明：**
- `mark` - 定义视觉表现的标记对象
- `*transforms` - 数据转换的统计/变换对象
- `orient` - 方向控制："x"、"y" 或 "v"/"h"
- `legend` - 是否包含在图例中（True/False）
- `data` - 当前图层数据覆盖
- `**variables` - 覆盖或添加变量映射

**示例：**
```python
# 简单标记
p.add(so.Dot())

# 带统计的标记
p.add(so.Line(), so.PolyFit(order=2))

# 带多重变换的标记
p.add(so.Bar(), so.Agg(), so.Dodge())

# 图层专属映射
p.add(so.Dot(), color='category')
p.add(so.Line(), so.Agg(), color='category')

# 图层专属数据
p.add(so.Dot())
p.add(so.Line(), data=summary_df)
```

#### facet()

基于分类变量创建子图。

```python
Plot.facet(col=None, row=None, order=None, wrap=None)
```

**参数说明：**
- `col` - 列分面变量
- `row` - 行分面变量
- `order` - 分面顺序字典（键：变量名）
- `wrap` - 列换行阈值

**示例：**
```python
p.facet(col='time', row='sex')
p.facet(col='category', wrap=3)
p.facet(col='day', order={'day': ['Thur', 'Fri', 'Sat', 'Sun']})
```

#### pair()

为多变量创建成对子图。

```python
Plot.pair(x=None, y=None, wrap=None, cross=True)
```

**参数说明：**
- `x` - X轴配对变量
- `y` - Y轴配对变量（None 时使用 x）
- `wrap` - 列换行阈值
- `cross` - 包含所有 x/y 组合（默认为 True，False 时仅对角线）

**示例：**
```python
# 全变量配对
p = so.Plot(df).pair(x=['a', 'b', 'c'])
p.add(so.Dot())

# 矩形网格
p = so.Plot(df).pair(x=['a', 'b'], y=['c', 'd'])
p.add(so.Dot(), alpha=0.5)
```

#### scale()

自定义数据到视觉属性的映射方式。

```python
Plot.scale(**scales)
```

**参数：** 属性名与 Scale 对象的键值对

**示例：**
```python
p.scale(
    x=so.Continuous().tick(every=5),
    y=so.Continuous().label(like='{x:.1f}'),
    color=so.Nominal(['#1f77b4', '#ff7f0e', '#2ca02c']),
    pointsize=(5, 10)  # 范围简写
)
```

#### limit()

设置坐标轴范围。

```python
Plot.limit(x=None, y=None)
```

**参数说明：**
- `x` - X轴范围元组 (最小值, 最大值)
- `y` - Y轴范围元组 (最小值, 最大值)

**示例：**
```python
p.limit(x=(0, 100), y=(0, 50))
```

#### label()

设置坐标轴标签与标题。

```python
Plot.label(x=None, y=None, color=None, title=None, **labels)
```

**参数：** 属性名与标签字符串的键值对

**示例：**
```python
p.label(
    x='总账单 ($)',
    y='小费金额 ($)',
    color='星期',
    title='餐厅小费分析'
)
```

#### theme()

应用 matplotlib 样式设置。

```python
Plot.theme(config, **kwargs)
```

**参数说明：**
- `config` - rcParams 字典或 seaborn 主题字典
- `**kwargs` - 独立 rcParams 参数

**示例：**
```python
# Seaborn 主题
p.theme({**sns.axes_style('whitegrid'), **sns.plotting_context('talk')})

# 自定义 rcParams
p.theme({'axes.facecolor': 'white', 'axes.grid': True})

# 独立参数
p.theme(axes_facecolor='white', font_scale=1.2)
```

#### layout()

配置子图布局。

```python
Plot.layout(size=None, extent=None, engine=None)
```

**参数说明：**
- `size` - 尺寸 (宽度, 高度)（英寸）
- `extent` - 子图范围 (左, 下, 右, 上)
- `engine` - 布局引擎："tight"、"constrained" 或 None

**示例：**
```python
p.layout(size=(10, 6), engine='constrained')
```

#### share()

控制分面间的坐标轴共享。

```python
Plot.share(x=None, y=None)
```

**参数说明：**
- `x` - X轴共享：True、False 或 "col"/"row"
- `y` - Y轴共享：True、False 或 "col"/"row"

**示例：**
```python
p.share(x=True, y=False)  # 全局共享X轴，独立Y轴
p.share(x='col')  # 仅列内共享X轴
```

#### on()

在现有 matplotlib 图形或坐标轴上绘图。

```python
Plot.on(target)
```

**参数说明：**
- `target` - matplotlib Figure 或 Axes 对象

**示例：**
```python
import matplotlib.pyplot as plt

fig, axes = plt.subplots(2, 2, figsize=(10, 10))
so.Plot(df, x='x', y='y').add(so.Dot()).on(axes[0, 0])
so.Plot(df, x='x', y='z').add(so.Line()).on(axes[0, 1])
```

#### show()

渲染并显示图表。

```python
Plot.show(**kwargs)
```

**参数：** 传递给 `matplotlib.pyplot.show()`

#### save()

保存图表到文件。

```python
Plot.save(filename, **kwargs)
```

**参数说明：**
- `filename` - 输出文件名
- `**kwargs` - 传递给 `matplotlib.figure.Figure.savefig()`

**示例：**
```python
p.save('plot.png', dpi=300, bbox_inches='tight')
p.save('plot.pdf')
```

## 标记对象

标记定义数据的视觉表现形式。

### Dot

表示独立观测点的标记。

```python
so.Dot(artist_kws=None, **kwargs)
```

**属性：**
- `color` - 填充色
- `alpha` - 透明度
- `fillcolor` - 备用颜色属性
- `fillalpha` - 备用透明度属性
- `edgecolor` - 边缘色
- `edgealpha` - 边缘透明度
- `edgewidth` - 边缘线宽
- `marker` - 标记样式
- `pointsize` - 标记尺寸
- `stroke` - 边缘宽度

**示例：**
```python
so.Plot(df, x='x', y='y').add(so.Dot(color='blue', pointsize=10))
so.Plot(df, x='x', y='y', color='cat').add(so.Dot(alpha=0.5))
```

### Line

连接观测点的线条。

```python
so.Line(artist_kws=None, **kwargs)
```

**属性：**
- `color` - 线条颜色
- `alpha` - 透明度
- `linewidth` - 线宽
- `linestyle` - 线型（"-"、"--"、"-."、":"）
- `marker` - 数据点标记
- `pointsize` - 标记尺寸
- `edgecolor` - 标记边缘色
- `edgewidth` - 标记边缘宽度

**示例：**
```python
so.Plot(df, x='x', y='y').add(so.Line())
so.Plot(df, x='x', y='y', color='cat').add(so.Line(linewidth=2))
```

### Path

按数据顺序连接点（不按X轴排序），类似 Line。

```python
so.Path(artist_kws=None, **kwargs)
```

属性同 `Line`。

**示例：**
```python
# 适用于轨迹、环形等场景
so.Plot(trajectory_df, x='x', y='y').add(so.Path())
```

### Bar

矩形柱状图。

```python
so.Bar(artist_kws=None, **kwargs)
```

**属性：**
- `color` - 填充色
- `alpha` - 透明度
- `edgecolor` - 边缘色
- `edgealpha` - 边缘透明度
- `edgewidth` - 边缘线宽
- `width` - 柱宽（数据单位）

**示例：**
```python
so.Plot(df, x='category', y='value').add(so.Bar())
so.Plot(df, x='x', y='y').add(so.Bar(color='#1f77b4', width=0.5))
```

### Bars

多柱状图（用于带误差条的聚合数据）。

```python
so.Bars(artist_kws=None, **kwargs)
```

属性同 `Bar`。需配合 `Agg()` 或 `Est()` 统计使用。

**示例：**
```python
so.Plot(df, x='category', y='value').add(so.Bars(), so.Agg())
```

### Area

线条与基线间的填充区域。

```python
so.Area(artist_kws=None, **kwargs)
```

**属性：**
- `color` - 填充色
- `alpha` - 透明度
- `edgecolor` - 边缘色
- `edgealpha` - 边缘透明度
- `edgewidth` - 边缘线宽
- `baseline` - 基线值（默认：0）

**示例：**
```python
so.Plot(df, x='x', y='y').add(so.Area(alpha=0.3))
so.Plot(df, x='x', y='y', color='cat').add(so.Area())
```

### Band

两条线间的填充带（用于范围/区间）。

```python
so.Band(artist_kws=None, **kwargs)
```

属性同 `Area`。需 `ymin` 和 `ymax` 映射或配合 `Est()` 统计使用。

**示例：**
```python
so.Plot(df, x='x', ymin='lower', ymax='upper').add(so.Band())
so.Plot(df, x='x', y='y').add(so.Band(), so.Est())
```

### Range

带端点标记的线段（用于范围表示）。

```python
so.Range(artist_kws=None, **kwargs)
```

**属性：**
- `color` - 线段与标记颜色
- `alpha` - 透明度
- `linewidth` - 线宽
- `marker` - 端点标记样式
- `pointsize` - 标记尺寸
- `edgewidth` - 标记边缘宽度

**示例：**
```python
so.Plot(df, x='x', y='y').add(so.Range(), so.Est())
```

### Dash

短横/竖线（用于分布标记）。

```python
so.Dash(artist_kws=None, **kwargs)
```

**属性：**
- `color` - 线条颜色
- `alpha` - 透明度
- `linewidth` - 线宽
- `width` - 线段长度（数据单位）

**示例：**
```python
so.Plot(df, x='category', y='value').add(so.Dash())
```

### Text

数据点处的文本标签。

```python
so.Text(artist_kws=None, **kwargs)
```

**属性：**
- `color` - 文本颜色
- `alpha` - 透明度
- `fontsize` - 字体大小
- `halign` - 水平对齐："left"、"center"、"right"
- `valign` - 垂直对齐："bottom"、"center"、"top"
- `offset` - 相对点的 (x, y) 偏移量

需配合 `text` 映射使用。

**示例：**
```python
so.Plot(df, x='x', y='y', text='label').add(so.Text())
so.Plot(df, x='x', y='y', text='value').add(so.Text(fontsize=10, offset=(0, 5)))
```

## 统计对象

统计对象在渲染前转换数据。通过 `.add()` 与标记组合使用。

### Agg

按组聚合观测值。

```python
so.Agg(func='mean')
```

**参数说明：**
- `func` - 聚合函数："mean"、"median"、"sum"、"min"、"max"、"count" 或可调用对象

**示例：**
```python
so.Plot(df, x='category', y='value').add(so.Bar(), so.Agg('mean'))
so.Plot(df, x='x', y='y', color='group').add(so.Line(), so.Agg('median'))
```

### Est

带误差区间的中心趋势估计。

```python
so.Est(func='mean', errorbar=('ci', 95), n_boot=1000, seed=None)
```

**参数说明：**
- `func` - 估计器："mean"、"median"、"sum" 或可调用对象
- `errorbar` - 误差表示方式：
  - `("ci", level)` - 置信区间（自助法）
  - `("pi", level)` - 百分位区间
  - `("se", scale)` - 标准误差（缩放因子）
  - `"sd"` - 标准差
- `n_boot` - 自助法迭代次数
- `seed` - 随机种子

**示例：**
```python
so.Plot(df, x='category', y='value').add(so.Bar(), so.Est())
so.Plot(df, x='x', y='y').add(so.Line(), so.Est(errorbar='sd'))
so.Plot(df, x='x', y='y').add(so.Line(), so.Est(errorbar=('ci', 95)))
so.Plot(df, x='x', y='y').add(so.Band(), so.Est())
```

### Hist

分箱观测值并计数/聚合。

```python
so.Hist(stat='count', bins='auto', binwidth=None, binrange=None,
        common_norm=True, common_bins=True, cumulative=False)
```

**参数说明：**
- `stat` - 统计量："count"、"density"、"probability"、"percent"、"frequency"
- `bins` - 分箱数量、分箱方法或边界
- `binwidth` - 分箱宽度

- `binrange` - 用于分箱的 (min, max) 范围
- `common_norm` - 跨组统一归一化
- `common_bins` - 所有组使用相同分箱
- `cumulative` - 累积直方图

**示例：**
```python
so.Plot(df, x='value').add(so.Bars(), so.Hist())
so.Plot(df, x='value').add(so.Bars(), so.Hist(bins=20, stat='density'))
so.Plot(df, x='value', color='group').add(so.Area(), so.Hist(cumulative=True))
```

### KDE

核密度估计。

```python
so.KDE(bw_method='scott', bw_adjust=1, gridsize=200,
       cut=3, cumulative=False)
```

**参数：**
- `bw_method` - 带宽方法："scott"、"silverman" 或标量值
- `bw_adjust` - 带宽乘数
- `gridsize` - 密度曲线分辨率
- `cut` - 数据范围外延量（带宽单位）
- `cumulative` - 累积密度

**示例：**
```python
so.Plot(df, x='value').add(so.Line(), so.KDE())
so.Plot(df, x='value', color='group').add(so.Area(alpha=0.5), so.KDE())
so.Plot(df, x='x', y='y').add(so.Line(), so.KDE(bw_adjust=0.5))
```

### Count

按组统计观测值数量。

```python
so.Count()
```

**示例：**
```python
so.Plot(df, x='category').add(so.Bar(), so.Count())
```

### PolyFit

多项式回归拟合。

```python
so.PolyFit(order=1)
```

**参数：**
- `order` - 多项式阶数（1=线性，2=二次，依此类推）

**示例：**
```python
so.Plot(df, x='x', y='y').add(so.Dot())
so.Plot(df, x='x', y='y').add(so.Line(), so.PolyFit(order=2))
```

### Perc

计算百分位数。

```python
so.Perc(k=5, method='linear')
```

**参数：**
- `k` - 百分位区间数量
- `method` - 插值方法

**示例：**
```python
so.Plot(df, x='x', y='y').add(so.Band(), so.Perc())
```

## 移动对象

通过调整位置解决重叠或创建特定布局。

### Dodge

水平错位排列。

```python
so.Dodge(empty='keep', gap=0)
```

**参数：**
- `empty` - 空组处理方式："keep"（保留）、"drop"（丢弃）、"fill"（填充）
- `gap` - 错位元素间距（比例值）

**示例：**
```python
so.Plot(df, x='category', y='value', color='group').add(so.Bar(), so.Dodge())
so.Plot(df, x='cat', y='val', color='hue').add(so.Dot(), so.Dodge(gap=0.1))
```

### Stack

垂直堆叠标记。

```python
so.Stack()
```

**示例：**
```python
so.Plot(df, x='x', y='y', color='category').add(so.Bar(), so.Stack())
so.Plot(df, x='x', y='y', color='group').add(so.Area(), so.Stack())
```

### Jitter

添加随机位置扰动。

```python
so.Jitter(width=None, height=None, seed=None)
```

**参数：**
- `width` - X方向扰动幅度（数据单位或比例）
- `height` - Y方向扰动幅度
- `seed` - 随机种子

**示例：**
```python
so.Plot(df, x='category', y='value').add(so.Dot(), so.Jitter())
so.Plot(df, x='cat', y='val').add(so.Dot(), so.Jitter(width=0.2))
```

### Shift

按固定量平移位置。

```python
so.Shift(x=0, y=0)
```

**参数：**
- `x` - X方向平移量（数据单位）
- `y` - Y方向平移量

**示例：**
```python
so.Plot(df, x='x', y='y').add(so.Dot(), so.Shift(x=1))
```

### Norm

数值归一化。

```python
so.Norm(func='max', where=None, by=None, percent=False)
```

**参数：**
- `func` - 归一化方法："max"、"sum"、"area" 或可调用函数
- `where` - 应用轴："x"、"y" 或 None
- `by` - 分组变量（独立归一化）
- `percent` - 显示为百分比

**示例：**
```python
so.Plot(df, x='x', y='y', color='group').add(so.Area(), so.Norm())
```

## 比例对象

控制数据值到视觉属性的映射关系。

### Continuous

适用于数值型数据。

```python
so.Continuous(values=None, norm=None, trans=None)
```

**方法：**
- `.tick(at=None, every=None, between=None, minor=None)` - 配置刻度
- `.label(like=None, base=None, unit=None)` - 格式化标签

**参数：**
- `values` - 显式值范围 (min, max)
- `norm` - 归一化函数
- `trans` - 变换方法："log"、"sqrt"、"symlog"、"logit"、"pow10" 或可调用函数

**示例：**
```python
p.scale(
    x=so.Continuous().tick(every=10),
    y=so.Continuous(trans='log').tick(at=[1, 10, 100]),
    color=so.Continuous(values=(0, 1)),
    pointsize=(5, 20)  # Continuous范围的简写形式
)
```

### Nominal

适用于分类型数据。

```python
so.Nominal(values=None, order=None)
```

**参数：**
- `values` - 显式值（如颜色、标记）
- `order` - 类别排序

**示例：**
```python
p.scale(
    color=so.Nominal(['#1f77b4', '#ff7f0e', '#2ca02c']),
    marker=so.Nominal(['o', 's', '^']),
    x=so.Nominal(order=['Low', 'Medium', 'High'])
)
```

### Temporal

适用于时间数据。

```python
so.Temporal(values=None, trans=None)
```

**方法：**
- `.tick(every=None, between=None)` - 配置刻度
- `.label(concise=False)` - 格式化标签

**示例：**
```python
p.scale(x=so.Temporal().tick(every=('month', 1)).label(concise=True))
```

## 完整示例

### 带统计量的分层绘图

```python
(
    so.Plot(df, x='total_bill', y='tip', color='time')
    .add(so.Dot(), alpha=0.5)
    .add(so.Line(), so.PolyFit(order=2))
    .scale(color=so.Nominal(['#1f77b4', '#ff7f0e']))
    .label(x='总账单 ($)', y='小费 ($)', title='小费分析')
    .theme({**sns.axes_style('whitegrid')})
)
```

### 分面分布图

```python
(
    so.Plot(df, x='measurement', color='treatment')
    .facet(col='timepoint', wrap=3)
    .add(so.Area(alpha=0.5), so.KDE())
    .add(so.Dot(), so.Jitter(width=0.1), y=0)
    .scale(x=so.Continuous().tick(every=5))
    .label(x='测量值 (单位)', title='随时间变化的处理效果')
    .share(x=True, y=False)
)
```

### 分组条形图

```python
(
    so.Plot(df, x='category', y='value', color='group')
    .add(so.Bar(), so.Agg('mean'), so.Dodge())
    .add(so.Range(), so.Est(errorbar='se'), so.Dodge())
    .scale(color=so.Nominal(order=['A', 'B', 'C']))
    .label(y='均值', title='按类别和分组比较')
)
```

### 复杂多层绘图

```python
(
    so.Plot(df, x='date', y='value')
    .add(so.Dot(color='gray', pointsize=3), alpha=0.3)
    .add(so.Line(color='blue', linewidth=2), so.Agg('mean'))
    .add(so.Band(color='blue', alpha=0.2), so.Est(errorbar=('ci', 95)))
    .facet(col='sensor', row='location')
    .scale(
        x=so.Temporal().label(concise=True),
        y=so.Continuous().tick(every=10)
    )
    .label(
        x='日期',
        y='测量值',
        title='按位置划分的传感器测量'
    )
    .layout(size=(12, 8), engine='constrained')
)
```

## 从函数接口迁移

### 散点图

**函数接口：**
```python
sns.scatterplot(data=df, x='x', y='y', hue='category', size='value')
```

**对象接口：**
```python
so.Plot(df, x='x', y='y', color='category', pointsize='value').add(so.Dot())
```

### 带置信区间的折线图

**函数接口：**
```python
sns.lineplot(data=df, x='time', y='measurement', hue='group', errorbar='ci')
```

**对象接口：**
```python
(
    so.Plot(df, x='time', y='measurement', color='group')
    .add(so.Line(), so.Est())
)
```

### 直方图

**函数接口：**
```python
sns.histplot(data=df, x='value', hue='category', stat='density', kde=True)
```

**对象接口：**
```python
(
    so.Plot(df, x='value', color='category')
    .add(so.Bars(), so.Hist(stat='density'))
    .add(so.Line(), so.KDE())
)
```

### 带误差条的条形图

**函数接口：**
```python
sns.barplot(data=df, x='category', y='value', hue='group', errorbar='ci')
```

**对象接口：**
```python
(
    so.Plot(df, x='category', y='value', color='group')
    .add(so.Bar(), so.Agg(), so.Dodge())
    .add(so.Range(), so.Est(), so.Dodge())
)
```

## 技巧与最佳实践

1. **方法链**：每个方法返回新Plot对象，支持流式链式调用
2. **图层组合**：通过多个`.add()`调用叠加不同标记
3. **变换顺序**：在`.add(mark, stat, move)`中，先应用stat再应用move
4. **变量优先级**：图层级映射覆盖Plot级映射
5. **比例简写**：使用元组定义简单范围：`color=(min, max)` 替代完整Scale对象
6. **Jupyter渲染**：返回Plot对象时自动渲染，否则使用`.show()`
7. **保存图像**：使用`.save()`而非`plt.savefig()`确保正确处理
8. **Matplotlib访问**：通过`.on(ax)`集成到matplotlib图形
