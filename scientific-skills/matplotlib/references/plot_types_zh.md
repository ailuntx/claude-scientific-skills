# Matplotlib 绘图类型指南

全面介绍 matplotlib 中的不同绘图类型，包含示例和使用场景。

## 1. 折线图

**使用场景：** 时间序列、连续数据、趋势分析、函数可视化

### 基础折线图
```python
fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(x, y, linewidth=2, label='数据')
ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.legend()
```

### 多线折线图
```python
ax.plot(x, y1, label='数据集1', linewidth=2)
ax.plot(x, y2, label='数据集2', linewidth=2, linestyle='--')
ax.plot(x, y3, label='数据集3', linewidth=2, linestyle=':')
ax.legend()
```

### 带标记点的折线图
```python
ax.plot(x, y, marker='o', markersize=8, linestyle='-',
        linewidth=2, markerfacecolor='red', markeredgecolor='black')
```

### 阶梯图
```python
ax.step(x, y, where='mid', linewidth=2, label='阶梯函数')
# where 选项: 'pre', 'post', 'mid'
```

### 误差线图
```python
ax.errorbar(x, y, yerr=error, fmt='o-', linewidth=2,
            capsize=5, capthick=2, label='含不确定性')
```

## 2. 散点图

**使用场景：** 相关性分析、变量关系、聚类识别、异常值检测

### 基础散点图
```python
ax.scatter(x, y, s=50, alpha=0.6)
```

### 尺寸与颜色散点图
```python
scatter = ax.scatter(x, y, s=sizes*100, c=colors,
                     cmap='viridis', alpha=0.6, edgecolors='black')
plt.colorbar(scatter, ax=ax, label='颜色变量')
```

### 分类散点图
```python
for category in categories:
    mask = data['category'] == category
    ax.scatter(data[mask]['x'], data[mask]['y'],
               label=category, s=50, alpha=0.7)
ax.legend()
```

## 3. 柱状图

**使用场景：** 分类比较、离散数据、计数统计

### 垂直柱状图
```python
ax.bar(categories, values, color='steelblue',
       edgecolor='black', linewidth=1.5)
ax.set_ylabel('数值')
```

### 水平柱状图
```python
ax.barh(categories, values, color='coral',
        edgecolor='black', linewidth=1.5)
ax.set_xlabel('数值')
```

### 分组柱状图
```python
x = np.arange(len(categories))
width = 0.35

ax.bar(x - width/2, values1, width, label='组1')
ax.bar(x + width/2, values2, width, label='组2')
ax.set_xticks(x)
ax.set_xticklabels(categories)
ax.legend()
```

### 堆叠柱状图
```python
ax.bar(categories, values1, label='部分1')
ax.bar(categories, values2, bottom=values1, label='部分2')
ax.bar(categories, values3, bottom=values1+values2, label='部分3')
ax.legend()
```

### 带误差线的柱状图
```python
ax.bar(categories, values, yerr=errors, capsize=5,
       color='steelblue', edgecolor='black')
```

### 带图案的柱状图
```python
bars1 = ax.bar(x - width/2, values1, width, label='组1',
               color='white', edgecolor='black', hatch='//')
bars2 = ax.bar(x + width/2, values2, width, label='组2',
               color='white', edgecolor='black', hatch='\\\\')
```

## 4. 直方图

**使用场景：** 分布分析、频率统计

### 基础直方图
```python
ax.hist(data, bins=30, edgecolor='black', alpha=0.7)
ax.set_xlabel('数值')
ax.set_ylabel('频率')
```

### 多重叠加直方图
```python
ax.hist(data1, bins=30, alpha=0.5, label='数据集1')
ax.hist(data2, bins=30, alpha=0.5, label='数据集2')
ax.legend()
```

### 归一化直方图（密度图）
```python
ax.hist(data, bins=30, density=True, alpha=0.7,
        edgecolor='black', label='经验分布')

# 叠加理论分布
from scipy.stats import norm
x = np.linspace(data.min(), data.max(), 100)
ax.plot(x, norm.pdf(x, data.mean(), data.std()),
        'r-', linewidth=2, label='正态拟合')
ax.legend()
```

### 二维直方图（六边形分箱）
```python
hexbin = ax.hexbin(x, y, gridsize=30, cmap='Blues')
plt.colorbar(hexbin, ax=ax, label='计数')
```

### 二维直方图（hist2d）
```python
h = ax.hist2d(x, y, bins=30, cmap='Blues')
plt.colorbar(h[3], ax=ax, label='计数')
```

## 5. 箱线图与小提琴图

**使用场景：** 统计分布、异常值检测、分布比较

### 箱线图
```python
ax.boxplot([data1, data2, data3],
           labels=['组A', '组B', '组C'],
           showmeans=True, meanline=True)
ax.set_ylabel('数值')
```

### 水平箱线图
```python
ax.boxplot([data1, data2, data3], vert=False,
           labels=['组A', '组B', '组C'])
ax.set_xlabel('数值')
```

### 小提琴图
```python
parts = ax.violinplot([data1, data2, data3],
                      positions=[1, 2, 3],
                      showmeans=True, showmedians=True)
ax.set_xticks([1, 2, 3])
ax.set_xticklabels(['组A', '组B', '组C'])
```

## 6. 热力图

**使用场景：** 矩阵数据、相关性分析、强度映射

### 基础热力图
```python
im = ax.imshow(matrix, cmap='coolwarm', aspect='auto')
plt.colorbar(im, ax=ax, label='数值')
ax.set_xlabel('X')
ax.set_ylabel('Y')
```

### 带标注的热力图
```python
im = ax.imshow(matrix, cmap='coolwarm')
plt.colorbar(im, ax=ax)

# 添加文本标注
for i in range(matrix.shape[0]):
    for j in range(matrix.shape[1]):
        text = ax.text(j, i, f'{matrix[i, j]:.2f}',
                       ha='center', va='center', color='black')
```

### 相关矩阵图
```python
corr = data.corr()
im = ax.imshow(corr, cmap='RdBu_r', vmin=-1, vmax=1)
plt.colorbar(im, ax=ax, label='相关性')

# 设置刻度标签
ax.set_xticks(range(len(corr)))
ax.set_yticks(range(len(corr)))
ax.set_xticklabels(corr.columns, rotation=45, ha='right')
ax.set_yticklabels(corr.columns)
```

## 7. 等高线图

**使用场景：** 二维平面上的三维数据、地形图、函数可视化

### 等高线
```python
contour = ax.contour(X, Y, Z, levels=10, cmap='viridis')
ax.clabel(contour, inline=True, fontsize=8)
plt.colorbar(contour, ax=ax)
```

### 填充等高线
```python
contourf = ax.contourf(X, Y, Z, levels=20, cmap='viridis')
plt.colorbar(contourf, ax=ax)
```

### 组合等高线
```python
contourf = ax.contourf(X, Y, Z, levels=20, cmap='viridis', alpha=0.8)
contour = ax.contour(X, Y, Z, levels=10, colors='black',
                     linewidths=0.5, alpha=0.4)
ax.clabel(contour, inline=True, fontsize=8)
plt.colorbar(contourf, ax=ax)
```

## 8. 饼图

**使用场景：** 比例展示、百分比（谨慎使用）

### 基础饼图
```python
ax.pie(sizes, labels=labels, autopct='%1.1f%%',
       startangle=90, colors=colors)
ax.axis('equal')  # 等比例确保圆形
```

### 突出饼图
```python
explode = (0.1, 0, 0, 0)  # 突出第一块
ax.pie(sizes, explode=explode, labels=labels,
       autopct='%1.1f%%', shadow=True, startangle=90)
ax.axis('equal')
```

### 环形图
```python
ax.pie(sizes, labels=labels, autopct='%1.1f%%',
       wedgeprops=dict(width=0.5), startangle=90)
ax.axis('equal')
```

## 9. 极坐标图

**使用场景：** 周期性数据、方向性数据、雷达图

### 基础极坐标图
```python
theta = np.linspace(0, 2*np.pi, 100)
r = np.abs(np.sin(2*theta))

ax = plt.subplot(111, projection='polar')
ax.plot(theta, r, linewidth=2)
```

### 雷达图
```python
categories = ['A', 'B', 'C', 'D', 'E']
values = [4, 3, 5, 2, 4]

# 添加首值形成闭合多边形
angles = np.linspace(0, 2*np.pi, len(categories), endpoint=False)
values_closed = np.concatenate((values, [values[0]]))
angles_closed = np.concatenate((angles, [angles[0]]))

ax = plt.subplot(111, projection='polar')
ax.plot(angles_closed, values_closed, 'o-', linewidth=2)
ax.fill(angles_closed, values_closed, alpha=0.25)
ax.set_xticks(angles)
ax.set_xticklabels(categories)
```

## 10. 流场与箭头图

**使用场景：** 矢量场、流动可视化

### 箭头图（矢量场）
```python
ax.quiver(X, Y, U, V, alpha=0.8)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_aspect('equal')
```

### 流线图
```python
ax.streamplot(X, Y, U, V, density=1.5, color='k', linewidth=1)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_aspect('equal')
```

## 11. 区域填充图

**使用场景：** 不确定性边界、置信区间、曲线下面积

### 双曲线间填充
```python
ax.plot(x, y, 'k-', linewidth=2, label='均值')
ax.fill_between(x, y - std, y + std, alpha=0.3,
                label='±1标准差')
ax.legend()
```

### 条件填充
```python
ax.plot(x, y1, label='线1')
ax.plot(x, y2, label='线2')
ax.fill_between(x, y1, y2, where=(y2 >= y1),
                alpha=0.3, label='y2 > y1', interpolate=True)
ax.legend()
```

## 12. 三维图

**使用场景：** 三维数据可视化

### 三维散点图
```python
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')
scatter = ax.scatter(x, y, z, c=colors, cmap='viridis',
                     marker='o', s=50)
plt.colorbar(scatter, ax=ax)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
```

### 三维曲面图
```python
fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')
surf = ax.plot_surface(X, Y, Z, cmap='viridis',
                       edgecolor='none', alpha=0.9)
plt.colorbar(surf, ax=ax)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
```

### 三维线框图
```python
fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')
ax.plot_wireframe(X, Y, Z, color='black', linewidth=0.5)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
```

### 三维等高线图
```python
fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')
ax.contour(X, Y, Z, levels=15, cmap='viridis')
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
```

## 13. 特殊绘图

### 茎叶图
```python
ax.stem(x, y, linefmt='C0-', markerfmt='C0o', basefmt='k-')
ax.set_xlabel('X')
ax.set_ylabel('Y')
```

### 填充多边形
```python
vertices = [(0, 0), (1, 0), (1, 1), (0, 1)]
from matplotlib.patches import Polygon
polygon = Polygon(vertices, closed=True, edgecolor='black',
                  facecolor='lightblue', alpha=0.5)
ax.add_patch(polygon)
ax.set_xlim(-0.5, 1.5)
ax.set_ylim(-0.5, 1.5)
```

### 阶梯状图
```python
ax.stairs(values, edges, fill=True, alpha=0.5)
```

### 水平断条图（甘特图式）
```python
ax.broken_barh([(10, 50), (100, 20), (130, 10)], (10, 9),
               facecolors='tab:blue')
ax.broken_barh([(10, 20), (50, 50), (120, 30)], (20, 9),
               facecolors='tab:orange')
ax.set_ylim(5, 35)
ax.set_xlim(0, 200)
ax.set_xlabel('时间')
ax.set_yticks([15, 25])
ax.set_yticklabels(['任务1', '任务2'])
```

## 14. 时间序列图

### 基础时间序列
```python
import pandas as pd
import matplotlib.dates as mdates

ax.plot(dates, values, linewidth=2)
ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
ax.xaxis.set_major_locator(mdates.DayLocator(interval=7))
plt.xticks(rotation=45)
ax.set_xlabel('日期')
ax.set_ylabel('数值')
```

### 带阴影区域的时间序列
```python
ax.plot(dates, values, linewidth=2)
# 标记周末或特定时段
ax.axvspan(start_date, end_date, alpha=0.2, color='gray')
```

## 绘图选择指南

| 数据类型 | 推荐绘图 | 备选方案 |
|-----------|-----------------|---------------------|
| 单连续变量 | 直方图, KDE图 | 箱线图, 小提琴图 |
| 双连续变量 | 散点图 | 六边形分箱图, 二维直方图 |
| 时间序列 | 折线图 | 面积图, 阶梯图 |
| 分类变量 vs 连续变量 | 柱状图, 箱线图 | 小提琴图, 带状图 |
| 双分类变量 | 热力图 | 分组柱状图 |
| 三连续变量 | 三维散点图, 等高线图 | 颜色编码散点图 |
| 比例数据 | 柱状图 | 饼
