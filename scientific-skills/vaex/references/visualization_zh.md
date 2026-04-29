# 数据可视化

本文档涵盖 Vaex 的可视化功能，用于创建大型数据集的图表、热力图和交互式可视化。

## 概述

Vaex 通过高效的分箱和聚合技术，能够可视化数十亿行数据集。其可视化系统直接处理大规模数据而无需采样，提供整个数据集的精确表示。

**核心特性：**
- 交互式可视化十亿行数据集
- 无需采样 - 使用全部数据
- 自动分箱与聚合
- 与 matplotlib 集成
- Jupyter 交互式组件支持

## 基础绘图

### 一维直方图

```python
import vaex
import matplotlib.pyplot as plt

df = vaex.open('data.hdf5')

# 简单直方图
df.plot1d(df.age)

# 自定义参数
df.plot1d(df.age,
          limits=[0, 100],
          shape=50,              # 分箱数量
          figsize=(10, 6),
          xlabel='年龄',
          ylabel='计数')

plt.show()
```

### 二维密度图（热力图）

```python
# 基础二维图
df.plot(df.x, df.y)

# 设置坐标范围
df.plot(df.x, df.y, limits=[[0, 10], [0, 10]])

# 使用百分位自动确定范围
df.plot(df.x, df.y, limits='99.7%')  # 3σ范围

# 自定义分辨率
df.plot(df.x, df.y, shape=(512, 512))

# 对数颜色标度
df.plot(df.x, df.y, f='log')
```

### 散点图（小数据集）

```python
# 适用于小数据集或采样
df_sample = df.sample(n=1000)

df_sample.scatter(df_sample.x, df_sample.y,
                  alpha=0.5,
                  s=10)  # 点大小

plt.show()
```

## 高级可视化选项

### 颜色标度与归一化

```python
# 线性标度（默认）
df.plot(df.x, df.y, f='identity')

# 对数标度
df.plot(df.x, df.y, f='log')
df.plot(df.x, df.y, f='log10')

# 平方根标度
df.plot(df.x, df.y, f='sqrt')

# 自定义颜色映射
df.plot(df.x, df.y, colormap='viridis')
df.plot(df.x, df.y, colormap='plasma')
df.plot(df.x, df.y, colormap='hot')
```

### 范围限制

```python
# 手动设置范围
df.plot(df.x, df.y, limits=[[xmin, xmax], [ymin, ymax]])

# 基于百分位的范围
df.plot(df.x, df.y, limits='99.7%')  # 3σ
df.plot(df.x, df.y, limits='95%')
df.plot(df.x, df.y, limits='minmax')  # 全范围

# 混合范围设置
df.plot(df.x, df.y, limits=[[0, 100], 'minmax'])
```

### 分辨率控制

```python
# 高分辨率（更多分箱）
df.plot(df.x, df.y, shape=(1024, 1024))

# 低分辨率（更快）
df.plot(df.x, df.y, shape=(128, 128))

# 不同轴向分辨率
df.plot(df.x, df.y, shape=(512, 256))
```

## 统计可视化

### 聚合可视化

```python
# 网格均值计算
df.plot(df.x, df.y, what=df.z.mean(),
        limits=[[0, 10], [0, 10]],
        shape=(100, 100),
        colormap='viridis')

# 标准差
df.plot(df.x, df.y, what=df.z.std())

# 求和
df.plot(df.x, df.y, what=df.z.sum())

# 计数（默认）
df.plot(df.x, df.y, what='count')
```

### 多统计量展示

```python
# 创建多子图
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# 计数图
df.plot(df.x, df.y, what='count',
        ax=axes[0, 0], show=False)
axes[0, 0].set_title('计数')

# 均值图
df.plot(df.x, df.y, what=df.z.mean(),
        ax=axes[0, 1], show=False)
axes[0, 1].set_title('z的均值')

# 标准差图
df.plot(df.x, df.y, what=df.z.std(),
        ax=axes[1, 0], show=False)
axes[1, 0].set_title('z的标准差')

# 最小值图
df.plot(df.x, df.y, what=df.z.min(),
        ax=axes[1, 1], show=False)
axes[1, 1].set_title('z的最小值')

plt.tight_layout()
plt.show()
```

## 选择集操作

同时可视化不同数据片段：

```python
import vaex
import matplotlib.pyplot as plt

df = vaex.open('data.hdf5')

# 创建选择集
df.select(df.category == 'A', name='group_a')
df.select(df.category == 'B', name='group_b')

# 绘制双选择集
df.plot1d(df.value, selection='group_a', label='A组')
df.plot1d(df.value, selection='group_b', label='B组')
plt.legend()
plt.show()

# 带选择集的二维图
df.plot(df.x, df.y, selection='group_a')
```

### 叠加多选择集

```python
# 创建基础图
fig, ax = plt.subplots(figsize=(10, 8))

# 不同颜色绘制选择集
df.plot(df.x, df.y, selection='group_a',
        ax=ax, show=False, colormap='Reds', alpha=0.5)
df.plot(df.x, df.y, selection='group_b',
        ax=ax, show=False, colormap='Blues', alpha=0.5)

ax.set_title('叠加选择集')
plt.show()
```

## 子图与布局

### 创建多图

```python
import matplotlib.pyplot as plt

# 创建子图网格
fig, axes = plt.subplots(2, 3, figsize=(15, 10))

# 绘制不同变量
variables = ['x', 'y', 'z', 'a', 'b', 'c']
for idx, var in enumerate(variables):
    row = idx // 3
    col = idx % 3
    df.plot1d(df[var], ax=axes[row, col], show=False)
    axes[row, col].set_title(f'{var}的分布')

plt.tight_layout()
plt.show()
```

### 分面绘图

```python
# 按类别绘图
categories = df.category.unique()

fig, axes = plt.subplots(1, len(categories), figsize=(15, 5))

for idx, cat in enumerate(categories):
    df_cat = df[df.category == cat]
    df_cat.plot(df_cat.x, df_cat.y,
                ax=axes[idx], show=False)
    axes[idx].set_title(f'{cat}类别')

plt.tight_layout()
plt.show()
```

## 交互式组件（Jupyter）

在 Jupyter 笔记本中创建交互式可视化：

### 选择集组件

```python
# 交互式选择
df.widget.selection_expression()
```

### 直方图组件

```python
# 带选择集的交互式直方图
df.plot_widget(df.x, df.y)
```

### 散点图组件

```python
# 交互式散点图
df.scatter_widget(df.x, df.y)
```

## 自定义设置

### 样式定制

```python
import matplotlib.pyplot as plt

# 创建自定义样式图
fig, ax = plt.subplots(figsize=(12, 8))

df.plot(df.x, df.y,
        limits='99%',
        shape=(256, 256),
        colormap='plasma',
        ax=ax,
        show=False)

# 坐标轴定制
ax.set_xlabel('X变量', fontsize=14, fontweight='bold')
ax.set_ylabel('Y变量', fontsize=14, fontweight='bold')
ax.set_title('定制密度图', fontsize=16, fontweight='bold')
ax.grid(alpha=0.3)

# 添加颜色条
plt.colorbar(ax.collections[0], ax=ax, label='密度')

plt.tight_layout()
plt.show()
```

### 图尺寸与DPI

```python
# 高分辨率绘图
df.plot(df.x, df.y,
        figsize=(12, 10),
        dpi=300)
```

## 特殊可视化

### 六边形分箱图

```python
# 热力图替代方案（六边形分箱）
plt.figure(figsize=(10, 8))
plt.hexbin(df.x.values[:100000], df.y.values[:100000],
           gridsize=50, cmap='viridis')
plt.colorbar(label='计数')
plt.xlabel('X')
plt.ylabel('Y')
plt.show()
```

### 等高线图

```python
import numpy as np

# 获取二维直方图数据
counts = df.count(binby=[df.x, df.y],
                  limits=[[0, 10], [0, 10]],
                  shape=(100, 100))

# 创建等高线图
x = np.linspace(0, 10, 100)
y = np.linspace(0, 10, 100)
plt.contourf(x, y, counts.T, levels=20, cmap='viridis')
plt.colorbar(label='计数')
plt.xlabel('X')
plt.ylabel('Y')
plt.title('等高线图')
plt.show()
```

### 矢量场叠加

```python
# 计算网格平均矢量
mean_vx = df.mean(df.vx, binby=[df.x, df.y],
                  limits=[[0, 10], [0, 10]],
                  shape=(20, 20))
mean_vy = df.mean(df.vy, binby=[df.x, df.y],
                  limits=[[0, 10], [0, 10]],
                  shape=(20, 20))

# 创建网格
x = np.linspace(0, 10, 20)
y = np.linspace(0, 10, 20)
X, Y = np.meshgrid(x, y)

# 绘图
fig, ax = plt.subplots(figsize=(10, 8))

# 基础热力图
df.plot(df.x, df.y, ax=ax, show=False)

# 矢量场叠加
ax.quiver(X, Y, mean_vx.T, mean_vy.T, alpha=0.7, color='white')

plt.show()
```

## 性能考量

### 大型可视化优化

```python
# 超大数据集降低分辨率
df.plot(df.x, df.y, shape=(256, 256))  # 快速

# 出版级质量
df.plot(df.x, df.y, shape=(1024, 1024))  # 高质量

# 平衡质量与性能
df.plot(df.x, df.y, shape=(512, 512))  # 良好平衡
```

### 可视化数据缓存

```python
# 单次计算多次绘图
counts = df.count(binby=[df.x, df.y],
                  limits=[[0, 10], [0, 10]],
                  shape=(512, 512))

# 多场景复用
plt.figure()
plt.imshow(counts.T, origin='lower', cmap='viridis')
plt.colorbar()
plt.show()

plt.figure()
plt.imshow(np.log10(counts.T + 1), origin='lower', cmap='plasma')
plt.colorbar()
plt.show()
```

## 导出与保存

### 保存图像

```python
# 保存为PNG
df.plot(df.x, df.y)
plt.savefig('plot.png', dpi=300, bbox_inches='tight')

# 保存为PDF（矢量）
plt.savefig('plot.pdf', bbox_inches='tight')

# 保存为SVG
plt.savefig('plot.svg', bbox_inches='tight')
```

### 批量绘图

```python
# 生成多图
variables = ['x', 'y', 'z']

for var in variables:
    plt.figure(figsize=(10, 6))
    df.plot1d(df[var])
    plt.title(f'{var}的分布')
    plt.savefig(f'plot_{var}.png', dpi=300, bbox_inches='tight')
    plt.close()
```

## 常用模式

### 模式：探索性数据分析

```python
import matplotlib.pyplot as plt

# 创建综合可视化
fig = plt.figure(figsize=(16, 12))

# 一维直方图
ax1 = plt.subplot(3, 3, 1)
df.plot1d(df.x, ax=ax1, show=False)
ax1.set_title('X分布')

ax2 = plt.subplot(3, 3, 2)
df.plot1d(df.y, ax=ax2, show=False)
ax2.set_title('Y分布')

ax3 = plt.subplot(3, 3, 3)
df.plot1d(df.z, ax=ax3, show=False)
ax3.set_title('Z分布')

# 二维图
ax4 = plt.subplot(3, 3, 4)
df.plot(df.x, df.y, ax=ax4, show=False)
ax4.set_title('X vs Y')

ax5 = plt.subplot(3, 3, 5)
df.plot(df.x, df.z, ax=ax5, show=False)
ax5.set_title('X vs Z')

ax6 = plt.subplot(3, 3, 6)
df.plot(df.y, df.z, ax=ax6, show=False)
ax6.set_title('Y vs Z')

# 网格统计
ax7 = plt.subplot(3, 3, 7)
df.plot(df.x, df.y, what=df.z.mean(), ax=ax7, show=False)
ax7.set_title('X-Y网格上的Z均值')

plt.tight_layout()
plt.savefig('eda_summary.png', dpi=300, bbox_inches='tight')
plt.show()
```

### 模式：跨组比较

```python
# 按类别比较分布
categories = df.category.unique()

fig, axes = plt.subplots(len(categories), 2,
                         figsize=(12, 4 * len(categories)))

for idx, cat in enumerate(categories):
    df.select(df.category == cat, name=f'cat_{cat}')

    # 一维直方图
    df.plot1d(df.value, selection=f'cat_{cat}',
              ax=axes[idx, 0], show=False)
    axes[idx, 0].set_title(f'{cat}类别 - 分布')

    # 二维图
    df.plot(df.x, df.y, selection=f'cat_{cat}',
            ax=axes[idx, 1], show=False)
    axes[idx, 1].set_title(f'{cat}类别 - X vs Y')

plt.tight_layout()
plt.show()
```

### 模式：时间序列可视化

```python
# 按时间分箱聚合
df['year'] = df.timestamp.dt.year
df['month'] = df.timestamp.dt.month

# 绘制时间序列
monthly_sales = df.groupby(['year', 'month']).agg({'sales': 'sum'})

plt.figure(figsize=(14, 6))
plt.plot(range(len(monthly_sales)), monthly_sales['sales'])
plt.xlabel('时间段')
plt.ylabel('销售额')
plt.title('销售额时序变化')
plt.grid(alpha=0.3)
plt.show()
```

## 与其他库集成

### Plotly 交互支持

```python
import plotly.graph_objects as go

# 从Vaex获取数据
counts = df.count(binby=[df.x, df.y], shape=(100, 100))

# 创建plotly图表
fig = go.Figure(data=go.Heatmap(z=counts.T))
fig.update_layout(title='交互式热力图')
fig.show()
```

### Seaborn 风格

```python
import seaborn as sns
import matplotlib.pyplot as plt

# 使用seaborn样式
sns.set_style('darkgrid')
sns.set_palette('husl')

df.plot1d(df.value)
plt.show()
```

## 最佳实践

1. **合理设置分辨率** - 平衡分辨率与性能（探索用256-512，出版用1024+）
2. **应用有效范围** - 使用百分位范围（'99%', '99.7%'）处理异常值
3. **明智选择颜色标度** - 宽范围计数用对数标度，均匀数据用线性标度
4. **利用选择集** - 无需创建新DataFrame即可比较子集
5. **缓存聚合结果** - 创建相似图时单次计算复用
6. **出版使用矢量格式** - 保存PDF/SVG获取可缩放图像
7. **避免采样** - Vaex可视化使用全量数据

## 故障排除

### 问题：空图或稀疏图

```python
#

df.plot(df.x, df.y, limits='99%')
```

### 问题：绘图速度过慢

```python
# 问题：分辨率过高
df.plot(df.x, df.y, shape=(2048, 2048))

# 解决方案：降低分辨率
df.plot(df.x, df.y, shape=(512, 512))
```

### 问题：无法观察低密度区域

```python
# 问题：线性比例被高密度区域掩盖
df.plot(df.x, df.y, f='identity')

# 解决方案：使用对数比例
df.plot(df.x, df.y, f='log')
```

## 相关资源

- 数据聚合方法：参见 `data_processing.md`
- 性能优化指南：参见 `performance.md`
- DataFrame 基础操作：参见 `core_dataframes.md`
