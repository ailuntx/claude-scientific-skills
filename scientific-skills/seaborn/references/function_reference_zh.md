# Seaborn 函数参考

本文档按类别组织，提供所有主要 seaborn 函数的综合参考。

## 关系图

### scatterplot()

**用途：** 创建散点图，点代表独立观测值。

**关键参数：**
- `data` - DataFrame、数组或数组字典
- `x, y` - x轴和y轴的变量
- `hue` - 颜色编码的分组变量
- `size` - 尺寸编码的分组变量
- `style` - 标记样式的分组变量
- `palette` - 调色板名称或列表
- `hue_order` - 分类色调级别的顺序
- `hue_norm` - 数值色调的归一化（元组或Normalize对象）
- `sizes` - 尺寸编码的范围（元组或字典）
- `size_order` - 分类尺寸级别的顺序
- `size_norm` - 数值尺寸的归一化
- `markers` - 标记样式（字符串、列表或字典）
- `style_order` - 分类样式级别的顺序
- `legend` - 图例绘制方式："auto"、"brief"、"full"或False
- `ax` - 绘图的Matplotlib坐标轴

**示例：**
```python
sns.scatterplot(data=df, x='height', y='weight',
                hue='gender', size='age', style='smoker',
                palette='Set2', sizes=(20, 200))
```

### lineplot()

**用途：** 绘制折线图，自动聚合重复测量值并计算置信区间。

**关键参数：**
- `data` - DataFrame、数组或数组字典
- `x, y` - x轴和y轴的变量
- `hue` - 颜色编码的分组变量
- `size` - 线宽的分组变量
- `style` - 线型（虚线）的分组变量
- `units` - 抽样单位的分组变量（单位内不聚合）
- `estimator` - 跨观测值的聚合函数（默认：均值）
- `errorbar` - 误差条方法："sd"、"se"、"pi"、("ci", level)、("pi", level)或None
- `n_boot` - CI计算的自举迭代次数
- `seed` - 可复现自举的随机种子
- `sort` - 绘图前排序数据
- `err_style` - 误差表示形式："band"或"bars"
- `err_kws` - 误差表示的附加参数
- `markers` - 强调数据点的标记样式
- `dashes` - 线条的虚线样式
- `legend` - 图例绘制方式
- `ax` - 绘图的Matplotlib坐标轴

**示例：**
```python
sns.lineplot(data=timeseries, x='time', y='signal',
             hue='condition', style='subject',
             errorbar=('ci', 95), markers=True)
```

### relplot()

**用途：** 在FacetGrid上绘制关系图（散点或折线）的图形级接口。

**关键参数：**
包含`scatterplot()`和`lineplot()`的所有参数，另加：
- `kind` - "scatter"或"line"
- `col` - 列分面的分类变量
- `row` - 行分面的分类变量
- `col_wrap` - 分列显示的列数阈值
- `col_order` - 列分面级别的顺序
- `row_order` - 行分面级别的顺序
- `height` - 每个分面的高度（英寸）
- `aspect` - 宽高比（宽度 = 高度 * 宽高比）
- `facet_kws` - FacetGrid的附加参数

**示例：**
```python
sns.relplot(data=df, x='time', y='measurement',
            hue='treatment', style='batch',
            col='cell_line', row='timepoint',
            kind='line', height=3, aspect=1.5)
```

## 分布图

### histplot()

**用途：** 绘制灵活分箱的单变量或双变量直方图。

**关键参数：**
- `data` - DataFrame、数组或字典
- `x, y` - 变量（双变量时y可选）
- `hue` - 分组变量
- `weights` - 观测值加权变量
- `stat` - 聚合统计量："count"、"frequency"、"probability"、"percent"、"density"
- `bins` - 分箱数、分箱边界或方法（"auto"、"fd"、"doane"、"scott"、"stone"、"rice"、"sturges"、"sqrt"）
- `binwidth` - 分箱宽度（覆盖bins）
- `binrange` - 分箱范围（元组）
- `discrete` - 将x视为离散值（柱条居中显示）
- `cumulative` - 计算累积分布
- `common_bins` - 所有色调级别使用相同分箱
- `common_norm` - 跨色调级别归一化
- `multiple` - 处理色调的方式："layer"、"dodge"、"stack"、"fill"
- `element` - 视觉元素："bars"、"step"、"poly"
- `fill` - 填充柱条/元素
- `shrink` - 柱条宽度缩放（multiple="dodge"时）
- `kde` - 叠加KDE估计
- `kde_kws` - KDE参数
- `line_kws` - step/poly元素的参数
- `thresh` - 分箱最小计数阈值
- `pthresh` - 最小概率阈值
- `pmax` - 颜色缩放的最大概率
- `log_scale` - 坐标轴对数刻度（布尔值或底数）
- `legend` - 是否显示图例
- `ax` - Matplotlib坐标轴

**示例：**
```python
sns.histplot(data=df, x='measurement', hue='condition',
             stat='density', bins=30, kde=True,
             multiple='layer', alpha=0.5)
```

### kdeplot()

**用途：** 绘制单变量或双变量核密度估计图。

**关键参数：**
- `data` - DataFrame、数组或字典
- `x, y` - 变量（双变量时y可选）
- `hue` - 分组变量
- `weights` - 观测值加权变量
- `palette` - 调色板
- `hue_order` - 色调级别顺序
- `hue_norm` - 数值色调归一化
- `multiple` - 处理色调的方式："layer"、"stack"、"fill"
- `common_norm` - 跨色调级别归一化
- `common_grid` - 所有色调级别使用相同网格
- `cumulative` - 计算累积分布
- `bw_method` - 带宽方法："scott"、"silverman"或标量
- `bw_adjust` - 带宽乘数（值越大越平滑）
- `log_scale` - 坐标轴对数刻度
- `levels` - 等高线层级数或值（双变量）
- `thresh` - 等高线最小密度阈值
- `gridsize` - 网格分辨率
- `cut` - 超出数据极值的扩展（带宽单位）
- `clip` - 曲线数据范围（元组）
- `fill` - 填充曲线/等高线下区域
- `legend` - 是否显示图例
- `ax` - Matplotlib坐标轴

**示例：**
```python
# 单变量
sns.kdeplot(data=df, x='measurement', hue='condition',
            fill=True, common_norm=False, bw_adjust=1.5)

# 双变量
sns.kdeplot(data=df, x='var1', y='var2',
            fill=True, levels=10, thresh=0.05)
```

### ecdfplot()

**用途：** 绘制经验累积分布函数图。

**关键参数：**
- `data` - DataFrame、数组或字典
- `x, y` - 变量（指定一个）
- `hue` - 分组变量
- `weights` - 观测值加权变量
- `stat` - "proportion"或"count"
- `complementary` - 绘制互补CDF（1 - ECDF）
- `palette` - 调色板
- `hue_order` - 色调级别顺序
- `hue_norm` - 数值色调归一化
- `log_scale` - 坐标轴对数刻度
- `legend` - 是否显示图例
- `ax` - Matplotlib坐标轴

**示例：**
```python
sns.ecdfplot(data=df, x='response_time', hue='treatment',
             stat='proportion', complementary=False)
```

### rugplot()

**用途：** 沿坐标轴绘制显示独立观测值的刻度标记。

**关键参数：**
- `data` - DataFrame、数组或字典
- `x, y` - 变量（指定一个）
- `hue` - 分组变量
- `height` - 刻度高度（坐标轴比例）
- `expand_margins` - 为刻度添加边距空间
- `palette` - 调色板
- `hue_order` - 色调级别顺序
- `hue_norm` - 数值色调归一化
- `legend` - 是否显示图例
- `ax` - Matplotlib坐标轴

**示例：**
```python
sns.rugplot(data=df, x='value', hue='category', height=0.05)
```

### displot()

**用途：** 在FacetGrid上绘制分布图的图形级接口。

**关键参数：**
包含`histplot()`、`kdeplot()`和`ecdfplot()`的所有参数，另加：
- `kind` - "hist"、"kde"、"ecdf"
- `rug` - 在边缘坐标轴添加地毯图
- `rug_kws` - 地毯图参数
- `col` - 列分面的分类变量
- `row` - 行分面的分类变量
- `col_wrap` - 分列显示
- `col_order` - 列分面顺序
- `row_order` - 行分面顺序
- `height` - 每个分面的高度
- `aspect` - 宽高比
- `facet_kws` - FacetGrid的附加参数

**示例：**
```python
sns.displot(data=df, x='measurement', hue='treatment',
            col='timepoint', kind='kde', fill=True,
            height=3, aspect=1.5, rug=True)
```

### jointplot()

**用途：** 绘制带边缘单变量图的双变量图。

**关键参数：**
- `data` - DataFrame
- `x, y` - x轴和y轴的变量
- `hue` - 分组变量
- `kind` - "scatter"、"kde"、"hist"、"hex"、"reg"、"resid"
- `height` - 图形尺寸（正方形）
- `ratio` - 主图与边缘图的比例
- `space` - 主图与边缘图的间距
- `dropna` - 删除缺失值
- `xlim, ylim` - 坐标轴范围（元组）
- `marginal_ticks` - 边缘坐标轴显示刻度
- `joint_kws` - 主图参数
- `marginal_kws` - 边缘图参数
- `hue_order` - 色调级别顺序
- `palette` - 调色板

**示例：**
```python
sns.jointplot(data=df, x='var1', y='var2', hue='group',
              kind='scatter', height=6, ratio=4,
              joint_kws={'alpha': 0.5})
```

### pairplot()

**用途：** 绘制数据集中的成对关系图。

**关键参数：**
- `data` - DataFrame
- `hue` - 颜色编码的分组变量
- `hue_order` - 色调级别顺序
- `palette` - 调色板
- `vars` - 要绘制的变量（默认：所有数值型）
- `x_vars, y_vars` - x轴和y轴的变量（非方形网格）
- `kind` - "scatter"、"kde"、"hist"、"reg"
- `diag_kind` - "auto"、"hist"、"kde"、None
- `markers` - 标记样式
- `height` - 每个分面的高度
- `aspect` - 宽高比
- `corner` - 仅绘制下三角区域
- `dropna` - 删除缺失值
- `plot_kws` - 非对角线图的参数
- `diag_kws` - 对角线图的参数
- `grid_kws` - PairGrid参数

**示例：**
```python
sns.pairplot(data=df, hue='species', palette='Set2',
             vars=['sepal_length', 'sepal_width', 'petal_length'],
             corner=True, height=2.5)
```

## 分类图

### stripplot()

**用途：** 绘制带抖动点的分类散点图。

**关键参数：**
- `data` - DataFrame、数组或字典
- `x, y` - 变量（一个分类，一个连续）
- `hue` - 分组变量
- `order` - 分类级别顺序
- `hue_order` - 色调级别顺序
- `jitter` - 抖动量：True、浮点数或False
- `dodge` - 并排分离色调级别
- `orient` - "v"或"h"（通常自动推断）
- `color` - 所有元素的统一颜色
- `palette` - 调色板
- `size` - 标记尺寸
- `edgecolor` - 标记边缘颜色
- `linewidth` - 标记边缘宽度
- `native_scale` - 分类轴使用数值刻度
- `formatter` - 分类轴格式化器
- `legend` - 是否显示图例
- `ax` - Matplotlib坐标轴

**示例：**
```python
sns.stripplot(data=df, x='day', y='total_bill',
              hue='sex', dodge=True, jitter=0.2)
```

### swarmplot()

**用途：** 绘制无重叠点的分类散点图。

**关键参数：**
与`stripplot()`相同，但：
- 无`jitter`参数
- `size` - 标记尺寸（避免重叠的关键）
- `warn_thresh` - 点数过多警告阈值（默认：0.05）

**注意：** 大数据集计算密集。超过1000个点时请使用stripplot。

**示例：**
```python
sns.swarmplot(data=df, x='day', y='total_bill',
              hue='time', dodge=True, size=5)
```

### boxplot()

**用途：** 绘制显示四分位数和异常值的箱线图。

**关键参数：**
- `data` - DataFrame、数组或字典
- `x, y` - 变量（一个分类，一个连续）
- `hue` - 分组变量
- `order` - 分类级别顺序
- `hue_order` - 色调级别顺序
- `orient` - "v"或"h"
- `color` - 箱体的统一颜色
- `palette` - 调色板
- `saturation` - 颜色饱和度
- `width` - 箱体宽度
- `dodge` - 并排分离色调级别
- `fliersize` - 异常值标记尺寸
- `linewidth` - 箱体线宽
- `whis` - 须线的IQR乘数（默认：1.5）
- `notch` - 绘制缺口箱体
- `showcaps` - 显示须线端帽
- `showmeans` - 显示均值
- `meanprops` - 均值标记属性
- `boxprops` - 箱体属性
- `whiskerprops` - 须线属性
- `capprops` - 端帽属性
- `flierprops` - 异常值属性
- `medianprops` - 中位线属性
- `native_scale` - 使用数值刻度
- `formatter` - 分类轴格式化器
- `legend` - 是否显示图例
- `ax` - Matplotlib坐标轴

**示例：**
```python
sns.boxplot(data=df, x='day', y='total_bill',
            hue='smoker', palette='Set3',
            showmeans=True, notch=True)
```

### violinplot()

**用途：** 结合箱线图和KDE的小提琴图。

**关键参数：**
与`boxplot()`相同，另加：
- `bw_method` - KDE带宽方法
- `bw_adjust` - KDE带宽乘数
- `cut` - KDE超出极值的扩展
- `density_norm` - "area"、"count"、"width"
- `inner` - "box"、"quartile"、"point"、"stick"、None
- `split

- `showfliers` - 显示离群点

**示例：**
```python
sns.boxenplot(data=df, x='day', y='total_bill',
              hue='time', palette='Set2')
```

### barplot()

**功能：** 绘制带误差条的条形图以展示统计估计值。

**关键参数：**
- `data` - 数据框、数组或字典
- `x, y` - 变量（一个分类变量，一个连续变量）
- `hue` - 分组变量
- `order` - 分类层级顺序
- `hue_order` - 分组层级顺序
- `estimator` - 聚合函数（默认：均值）
- `errorbar` - 误差表示方式："sd"、"se"、"pi"、("ci", 水平值)、("pi", 水平值) 或 None
- `n_boot` - 自助法迭代次数
- `seed` - 随机种子
- `units` - 抽样单元标识符
- `weights` - 观测权重
- `orient` - 方向："v"（垂直）或 "h"（水平）
- `color` - 统一条形颜色
- `palette` - 调色板
- `saturation` - 颜色饱和度
- `width` - 条形宽度
- `dodge` - 并排显示分组层级
- `errcolor` - 误差条颜色
- `errwidth` - 误差条线宽
- `capsize` - 误差条端帽宽度
- `native_scale` - 使用数值尺度
- `formatter` - 分类轴格式化器
- `legend` - 是否显示图例
- `ax` - Matplotlib 坐标轴

**示例：**
```python
sns.barplot(data=df, x='day', y='total_bill',
            hue='sex', estimator='median',
            errorbar=('ci', 95), capsize=0.1)
```

### countplot()

**功能：** 显示每个分类区间的观测计数。

**关键参数：**
与 `barplot()` 相同，但：
- 仅需指定 x 或 y 之一（分类变量）
- 无 estimator 或 errorbar 参数（直接显示计数）
- `stat` - 统计方式："count"（计数）或 "percent"（百分比）

**示例：**
```python
sns.countplot(data=df, x='day', hue='time',
              palette='pastel', dodge=True)
```

### pointplot()

**功能：** 通过连接线展示点估计值及置信区间。

**关键参数：**
与 `barplot()` 相同，额外增加：
- `markers` - 标记样式
- `linestyles` - 线条样式
- `scale` - 标记缩放比例
- `join` - 用线条连接点
- `capsize` - 误差条端帽宽度

**示例：**
```python
sns.pointplot(data=df, x='time', y='total_bill',
              hue='sex', markers=['o', 's'],
              linestyles=['-', '--'], capsize=0.1)
```

### catplot()

**功能：** 在分面网格上绘制分类图的顶层图形接口。

**关键参数：**
包含所有分类图参数，额外增加：
- `kind` - 图形类型："strip"、"swarm"、"box"、"violin"、"boxen"、"bar"、"point"、"count"
- `col` - 列分面分类变量
- `row` - 行分面分类变量
- `col_wrap` - 列分面换行数
- `col_order` - 列分面顺序
- `row_order` - 行分面顺序
- `height` - 分面高度
- `aspect` - 分面宽高比
- `sharex, sharey` - 共享坐标轴
- `legend` - 是否显示图例
- `legend_out` - 图例置于图形外
- `facet_kws` - 额外 FacetGrid 参数

**示例：**
```python
sns.catplot(data=df, x='day', y='total_bill',
            hue='smoker', col='time',
            kind='violin', split=True,
            height=4, aspect=0.8)
```

## 回归图

### regplot()

**功能：** 绘制数据及线性回归拟合线。

**关键参数：**
- `data` - 数据框
- `x, y` - 变量或数据向量
- `x_estimator` - 对 x 分箱应用聚合函数
- `x_bins` - x 分箱数
- `x_ci` - 分箱估计值的置信区间
- `scatter` - 显示散点
- `fit_reg` - 绘制回归线
- `ci` - 回归估计置信区间（整数或 None）
- `n_boot` - 置信区间自助法迭代次数
- `units` - 抽样单元标识符
- `seed` - 随机种子
- `order` - 多项式回归阶数
- `logistic` - 拟合逻辑回归
- `lowess` - 拟合局部加权回归
- `robust` - 拟合稳健回归
- `logx` - x 轴对数变换
- `x_partial, y_partial` - 偏回归（剔除变量影响）
- `truncate` - 将回归线限制在数据范围内
- `dropna` - 删除缺失值
- `x_jitter, y_jitter` - 添加数据抖动
- `label` - 图例标签
- `color` - 统一元素颜色
- `marker` - 标记样式
- `scatter_kws` - 散点图参数
- `line_kws` - 回归线参数
- `ax` - Matplotlib 坐标轴

**示例：**
```python
sns.regplot(data=df, x='total_bill', y='tip',
            order=2, robust=True, ci=95,
            scatter_kws={'alpha': 0.5})
```

### lmplot()

**功能：** 在分面网格上绘制回归图的顶层图形接口。

**关键参数：**
包含所有 `regplot()` 参数，额外增加：
- `hue` - 分组变量
- `col` - 列分面
- `row` - 行分面
- `palette` - 调色板
- `col_wrap` - 列分面换行数
- `height` - 分面高度
- `aspect` - 分面宽高比
- `markers` - 标记样式
- `sharex, sharey` - 共享坐标轴
- `hue_order` - 分组层级顺序
- `col_order` - 列分面顺序
- `row_order` - 行分面顺序
- `legend` - 是否显示图例
- `legend_out` - 图例置于图形外
- `facet_kws` - FacetGrid 参数

**示例：**
```python
sns.lmplot(data=df, x='total_bill', y='tip',
           hue='smoker', col='time', row='sex',
           height=3, aspect=1.2, ci=None)
```

### residplot()

**功能：** 绘制回归残差图。

**关键参数：**
与 `regplot()` 相同，但：
- 始终绘制残差（y - 预测值）与 x 的关系
- 添加 y=0 水平参考线
- `lowess` - 对残差拟合局部加权回归

**示例：**
```python
sns.residplot(data=df, x='x', y='y', lowess=True,
              scatter_kws={'alpha': 0.5})
```

## 矩阵图

### heatmap()

**功能：** 将矩形数据绘制为颜色编码矩阵。

**关键参数：**
- `data` - 二维类数组数据
- `vmin, vmax` - 色彩映射锚点值
- `cmap` - 色彩映射名称或对象
- `center` - 色彩映射中心值
- `robust` - 使用鲁棒分位数确定色彩范围
- `annot` - 单元格标注：True、False 或数组
- `fmt` - 标注格式字符串（如 ".2f"）
- `annot_kws` - 标注参数
- `linewidths` - 单元格边框宽度
- `linecolor` - 单元格边框颜色
- `cbar` - 显示颜色条
- `cbar_kws` - 颜色条参数
- `cbar_ax` - 颜色条坐标轴
- `square` - 强制单元格为正方形
- `xticklabels, yticklabels` - 刻度标签（True、False、整数或列表）
- `mask` - 掩码单元格的布尔数组
- `ax` - Matplotlib 坐标轴

**示例：**
```python
# 相关性矩阵
corr = df.corr()
mask = np.triu(np.ones_like(corr, dtype=bool))
sns.heatmap(corr, mask=mask, annot=True, fmt='.2f',
            cmap='coolwarm', center=0, square=True,
            linewidths=1, cbar_kws={'shrink': 0.8})
```

### clustermap()

**功能：** 绘制层次聚类热力图。

**关键参数：**
包含所有 `heatmap()` 参数，额外增加：
- `pivot_kws` - 数据透视参数（如需要）
- `method` - 聚类方法："single"、"complete"、"average"、"weighted"、"centroid"、"median"、"ward"
- `metric` - 聚类距离度量
- `standard_scale` - 标准化数据：0（行）、1（列）或 None
- `z_score` - Z 分数标准化：0（行）、1（列）或 None
- `row_cluster, col_cluster` - 是否聚类行/列
- `row_linkage, col_linkage` - 预计算聚类链接矩阵
- `row_colors, col_colors` - 附加颜色标注
- `dendrogram_ratio` - 树状图与热力图比例
- `colors_ratio` - 颜色标注与热力图比例
- `cbar_pos` - 颜色条位置（元组：x, y, 宽, 高）
- `tree_kws` - 树状图参数
- `figsize` - 图形尺寸

**示例：**
```python
sns.clustermap(data, method='average', metric='euclidean',
               z_score=0, cmap='viridis',
               row_colors=row_colors, col_colors=col_colors,
               figsize=(12, 12), dendrogram_ratio=0.1)
```

## 多图网格

### FacetGrid

**功能：** 用于绘制条件关系的多图网格。

**初始化：**
```python
g = sns.FacetGrid(data, row=None, col=None, hue=None,
                  col_wrap=None, sharex=True, sharey=True,
                  height=3, aspect=1, palette=None,
                  row_order=None, col_order=None, hue_order=None,
                  hue_kws=None, dropna=False, legend_out=True,
                  despine=True, margin_titles=False,
                  xlim=None, ylim=None, subplot_kws=None,
                  gridspec_kws=None)
```

**方法：**
- `map(func, *args, **kwargs)` - 对每个分面应用函数
- `map_dataframe(func, *args, **kwargs)` - 应用函数至完整数据框
- `set_axis_labels(x_var, y_var)` - 设置坐标轴标签
- `set_titles(template, **kwargs)` - 设置子图标题
- `set(kwargs)` - 设置所有坐标轴属性
- `add_legend(legend_data, title, label_order, **kwargs)` - 添加图例
- `savefig(*args, **kwargs)` - 保存图形

**示例：**
```python
g = sns.FacetGrid(df, col='time', row='sex', hue='smoker',
                  height=3, aspect=1.5, margin_titles=True)
g.map(sns.scatterplot, 'total_bill', 'tip', alpha=0.7)
g.add_legend()
g.set_axis_labels('总账单 ($)', '小费 ($)')
g.set_titles('{col_name} | {row_name}')
```

### PairGrid

**功能：** 用于绘制数据集中成对关系的网格。

**初始化：**
```python
g = sns.PairGrid(data, hue=None, vars=None,
                 x_vars=None, y_vars=None,
                 hue_order=None, palette=None,
                 hue_kws=None, corner=False,
                 diag_sharey=True, height=2.5,
                 aspect=1, layout_pad=0.5,
                 despine=True, dropna=False)
```

**方法：**
- `map(func, **kwargs)` - 对所有子图应用函数
- `map_diag(func, **kwargs)` - 对角线应用函数
- `map_offdiag(func, **kwargs)` - 非对角线应用函数
- `map_upper(func, **kwargs)` - 上三角应用函数
- `map_lower(func, **kwargs)` - 下三角应用函数
- `add_legend(legend_data, **kwargs)` - 添加图例
- `savefig(*args, **kwargs)` - 保存图形

**示例：**
```python
g = sns.PairGrid(df, hue='species', vars=['a', 'b', 'c', 'd'],
                 corner=True, height=2.5)
g.map_upper(sns.scatterplot, alpha=0.5)
g.map_lower(sns.kdeplot)
g.map_diag(sns.histplot, kde=True)
g.add_legend()
```

### JointGrid

**功能：** 双变量图网格（含边缘单变量图）。

**初始化：**
```python
g = sns.JointGrid(data=None, x=None, y=None, hue=None,
                  height=6, ratio=5, space=0.2,
                  dropna=False, xlim=None, ylim=None,
                  marginal_ticks=False, hue_order=None,
                  palette=None)
```

**方法：**
- `plot(joint_func, marginal_func, **kwargs)` - 绘制联合分布与边缘分布
- `plot_joint(func, **kwargs)` - 绘制联合分布
- `plot_marginals(func, **kwargs)` - 绘制边缘分布
- `refline(x, y, **kwargs)` - 添加参考线
- `set_axis_labels(xlabel, ylabel, **kwargs)` - 设置坐标轴标签
- `savefig(*args, **kwargs)` - 保存图形

**示例：**
```python
g = sns.JointGrid(data=df, x='x', y='y', hue='group',
                  height=6, ratio=5, space=0.2)
g.plot_joint(sns.scatterplot, alpha=0.5)
g.plot_marginals(sns.histplot, kde=True)
g.set_axis_labels('变量 X', '变量 Y')
```
