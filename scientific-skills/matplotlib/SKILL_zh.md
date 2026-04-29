---
name: matplotlib
description: 底层绘图库，用于实现完全自定义。当您需要精细控制每个绘图元素、创建新颖图表类型或集成特定科学工作流时使用。可导出为PNG/PDF/SVG格式用于出版。快速统计绘图请使用seaborn；交互式绘图请使用plotly；出版级多面板图表及期刊样式请使用scientific-visualization。
license: https://github.com/matplotlib/matplotlib/tree/main/LICENSE
metadata:
    skill-author: K-Dense Inc.
---

# Matplotlib

## 概述

Matplotlib是Python的基础可视化库，用于创建静态、动态和交互式图表。本技能提供高效使用matplotlib的指南，涵盖pyplot接口（MATLAB风格）和面向对象API（Figure/Axes），以及创建出版级可视化效果的最佳实践。

## 适用场景

本技能适用于：
- 创建各类图表（折线图、散点图、条形图、直方图、热力图、等高线图等）
- 生成科学或统计可视化
- 自定义图表外观（颜色、样式、标签、图例）
- 创建带子图的多面板图表
- 导出可视化结果到多种格式（PNG、PDF、SVG等）
- 构建交互式图表或动画
- 处理3D可视化
- 将图表集成到Jupyter笔记本或GUI应用中

## 核心概念

### Matplotlib层级结构

Matplotlib采用对象层级结构：

1. **Figure** - 所有绘图元素的顶级容器
2. **Axes** - 实际显示数据的绘图区域（一个Figure可包含多个Axes）
3. **Artist** - 图表上所有可见元素（线条、文本、刻度等）
4. **Axis** - 处理刻度和标签的数值轴对象（x轴、y轴）

### 两种接口

**1. pyplot接口（隐式，MATLAB风格）**
```python
import matplotlib.pyplot as plt

plt.plot([1, 2, 3, 4])
plt.ylabel('数值标签')
plt.show()
```
- 适合快速绘制简单图表
- 自动维护状态
- 适用于交互式工作和简单脚本

**2. 面向对象接口（显式）**
```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
ax.plot([1, 2, 3, 4])
ax.set_ylabel('数值标签')
plt.show()
```
- **推荐作为主要使用方式**
- 对图表和坐标轴控制更精确
- 更适合含多个子图的复杂图表
- 更易于维护和调试

## 常用工作流

### 1. 基础绘图

**单图工作流：**
```python
import matplotlib.pyplot as plt
import numpy as np

# 创建图表和坐标轴（面向对象接口 - 推荐）
fig, ax = plt.subplots(figsize=(10, 6))

# 生成并绘制数据
x = np.linspace(0, 2*np.pi, 100)
ax.plot(x, np.sin(x), label='sin(x)')
ax.plot(x, np.cos(x), label='cos(x)')

# 自定义设置
ax.set_xlabel('x轴')
ax.set_ylabel('y轴')
ax.set_title('三角函数')
ax.legend()
ax.grid(True, alpha=0.3)

# 保存/显示
plt.savefig('plot.png', dpi=300, bbox_inches='tight')
plt.show()
```

### 2. 多子图布局

**创建子图布局：**
```python
# 方法1：规则网格
fig, axes = plt.subplots(2, 2, figsize=(12, 10))
axes[0, 0].plot(x, y1)
axes[0, 1].scatter(x, y2)
axes[1, 0].bar(categories, values)
axes[1, 1].hist(data, bins=30)

# 方法2：马赛克布局（更灵活）
fig, axes = plt.subplot_mosaic([['left', 'right_top'],
                                 ['left', 'right_bottom']],
                                figsize=(10, 8))
axes['left'].plot(x, y)
axes['right_top'].scatter(x, y)
axes['right_bottom'].hist(data)

# 方法3：GridSpec（最大控制权）
from matplotlib.gridspec import GridSpec
fig = plt.figure(figsize=(12, 8))
gs = GridSpec(3, 3, figure=fig)
ax1 = fig.add_subplot(gs[0, :])  # 首行全列
ax2 = fig.add_subplot(gs[1:, 0])  # 后两行首列
ax3 = fig.add_subplot(gs[1:, 1:])  # 后两行末两列
```

### 3. 图表类型与应用场景

**折线图** - 时间序列、连续数据、趋势分析
```python
ax.plot(x, y, linewidth=2, linestyle='--', marker='o', color='blue')
```

**散点图** - 变量关系、相关性
```python
ax.scatter(x, y, s=sizes, c=colors, alpha=0.6, cmap='viridis')
```

**条形图** - 分类比较
```python
ax.bar(categories, values, color='steelblue', edgecolor='black')
# 横向条形图：
ax.barh(categories, values)
```

**直方图** - 数据分布
```python
ax.hist(data, bins=30, edgecolor='black', alpha=0.7)
```

**热力图** - 矩阵数据、相关性
```python
im = ax.imshow(matrix, cmap='coolwarm', aspect='auto')
plt.colorbar(im, ax=ax)
```

**等高线图** - 二维平面上的三维数据
```python
contour = ax.contour(X, Y, Z, levels=10)
ax.clabel(contour, inline=True, fontsize=8)
```

**箱线图** - 统计分布
```python
ax.boxplot([data1, data2, data3], labels=['A', 'B', 'C'])
```

**小提琴图** - 分布密度
```python
ax.violinplot([data1, data2, data3], positions=[1, 2, 3])
```

完整图表类型示例请参考`references/plot_types.md`。

### 4. 样式与自定义

**颜色指定方法：**
- 命名颜色：`'red'`, `'blue'`, `'steelblue'`
- 十六进制代码：`'#FF5733'`
- RGB元组：`(0.1, 0.2, 0.3)`
- 颜色映射：`cmap='viridis'`, `cmap='plasma'`, `cmap='coolwarm'`

**使用样式表：**
```python
plt.style.use('seaborn-v0_8-darkgrid')  # 应用预定义样式
# 可用样式：'ggplot', 'bmh', 'fivethirtyeight'等
print(plt.style.available)  # 列出所有可用样式
```

**通过rcParams自定义：**
```python
plt.rcParams['font.size'] = 12
plt.rcParams['axes.labelsize'] = 14
plt.rcParams['axes.titlesize'] = 16
plt.rcParams['xtick.labelsize'] = 10
plt.rcParams['ytick.labelsize'] = 10
plt.rcParams['legend.fontsize'] = 12
plt.rcParams['figure.titlesize'] = 18
```

**文本与标注：**
```python
ax.text(x, y, '标注文本', fontsize=12, ha='center')
ax.annotate('关键点', xy=(x, y), xytext=(x+1, y+1),
            arrowprops=dict(arrowstyle='->', color='red'))
```

详细样式选项参考`references/styling_guide.md`。

### 5. 图表保存

**导出多种格式：**
```python
# 高分辨率PNG（用于演示/论文）
plt.savefig('figure.png', dpi=300, bbox_inches='tight', facecolor='white')

# 矢量格式（用于出版）
plt.savefig('figure.pdf', bbox_inches='tight')
plt.savefig('figure.svg', bbox_inches='tight')

# 透明背景
plt.savefig('figure.png', dpi=300, bbox_inches='tight', transparent=True)
```

**关键参数：**
- `dpi`：分辨率（出版300dpi，网页150dpi，屏幕72dpi）
- `bbox_inches='tight'`：去除多余空白
- `facecolor='white'`：确保白色背景（透明主题时适用）
- `transparent=True`：透明背景

### 6. 3D绘图

```python
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')

# 曲面图
ax.plot_surface(X, Y, Z, cmap='viridis')

# 3D散点图
ax.scatter(x, y, z, c=colors, marker='o')

# 3D折线图
ax.plot(x, y, z, linewidth=2)

# 坐标轴标签
ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_zlabel('Z轴')
```

## 最佳实践

### 1. 接口选择
- **生产代码使用面向对象接口**（fig, ax = plt.subplots()）
- pyplot接口仅用于快速交互探索
- 始终显式创建图表而非依赖隐式状态

### 2. 图表尺寸与DPI
- 创建时设置figsize：`fig, ax = plt.subplots(figsize=(10, 6))`
- 按输出媒介选择合适DPI：
  - 屏幕/笔记本：72-100 dpi
  - 网页：150 dpi
  - 印刷/出版：300 dpi

### 3. 布局管理
- 使用`constrained_layout=True`或`tight_layout()`避免元素重叠
- 推荐`fig, ax = plt.subplots(constrained_layout=True)`自动调整间距

### 4. 颜色映射选择
- **顺序型**（viridis, plasma, inferno）：有序数据连续变化
- **发散型**（coolwarm, RdBu）：含中心点数据（如零点）
- **定性型**（tab10, Set3）：分类/名义数据
- 避免彩虹色系（jet）——不符合感知均匀性

### 5. 可访问性
- 使用色盲友好色系（viridis, cividis）
- 条形图添加图案/纹理辅助颜色区分
- 确保元素间足够对比度
- 包含描述性标签和图例

### 6. 性能优化
- 大数据集使用`rasterized=True`减小文件体积
- 绘图前适当降采样（如密集时间序列）
- 动画使用blitting提升性能

### 7. 代码组织
```python
# 良好实践：清晰结构
def create_analysis_plot(data, title):
    """创建标准化分析图表"""
    fig, ax = plt.subplots(figsize=(10, 6), constrained_layout=True)

    # 绘制数据
    ax.plot(data['x'], data['y'], linewidth=2)

    # 自定义设置
    ax.set_xlabel('X轴标签', fontsize=12)
    ax.set_ylabel('Y轴标签', fontsize=12)
    ax.set_title(title, fontsize=14, fontweight='bold')
    ax.grid(True, alpha=0.3)

    return fig, ax

# 调用函数
fig, ax = create_analysis_plot(my_data, '分析图表')
plt.savefig('analysis.png', dpi=300, bbox_inches='tight')
```

## 快速参考脚本

本技能包含`scripts/`目录下的辅助脚本：

### `plot_template.py`
演示各类图表最佳实践的模板脚本，可作为新可视化项目的起点。

**用法：**
```bash
python scripts/plot_template.py
```

### `style_configurator.py`
交互式工具，用于配置matplotlib样式偏好并生成自定义样式表。

**用法：**
```bash
python scripts/style_configurator.py
```

## 详细参考文档

完整信息请查阅参考文档：
- **`references/plot_types.md`** - 图表类型全集（含代码示例及应用场景）
- **`references/styling_guide.md`** - 样式选项、颜色映射与自定义详解
- **`references/api_reference.md`** - 核心类与方法参考
- **`references/common_issues.md`** - 常见问题排查指南

## 与其他工具集成

Matplotlib可与以下工具无缝协作：
- **NumPy/Pandas** - 直接绘制数组和DataFrame数据
- **Seaborn** - 基于matplotlib的高级统计可视化
- **Jupyter** - 通过`%matplotlib inline`或`%matplotlib widget`实现交互式绘图
- **GUI框架** - 嵌入Tkinter、Qt、wxPython应用

## 常见陷阱

1. **元素重叠**：使用`constrained_layout=True`或`tight_layout()`
2. **状态混淆**：采用面向对象接口避免pyplot状态机问题
3. **多图表内存泄漏**：显式关闭图表`plt.close(fig)`
4. **字体警告**：安装字体或通过`plt.rcParams['font.sans-serif']`屏蔽警告
5. **DPI误解**：figsize单位为英寸，非像素（像素数 = dpi × 英寸数）

## 扩展资源

- 官方文档：https://matplotlib.org/
- 示例库：https://matplotlib.org/stable/gallery/index.html
- 速查表：https://matplotlib.org/cheatsheets/
- 教程：https://matplotlib.org/stable/tutorials/index.html
