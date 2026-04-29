# Matplotlib API 参考

本文档提供最常用 matplotlib 类和方法的快速参考。

## 核心类

### Figure 图形

所有绘图元素的顶级容器。

**创建方式：**
```python
fig = plt.figure(figsize=(10, 6), dpi=100, facecolor='white')
fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(10, 6))
fig, axes = plt.subplots(2, 2, figsize=(12, 10))
```

**关键方法：**
- `fig.add_subplot(nrows, ncols, index)` - 添加子图
- `fig.add_axes([left, bottom, width, height])` - 在指定位置添加坐标轴
- `fig.savefig(filename, dpi=300, bbox_inches='tight')` - 保存图形
- `fig.tight_layout()` - 调整间距防止重叠
- `fig.suptitle(title)` - 设置图形标题
- `fig.legend()` - 创建图形级图例
- `fig.colorbar(mappable)` - 添加颜色条
- `plt.close(fig)` - 关闭图形释放内存

**关键属性：**
- `fig.axes` - 图形中所有坐标轴列表
- `fig.dpi` - 分辨率（每英寸点数）
- `fig.figsize` - 图形尺寸（英寸，宽×高）

### Axes 坐标轴

实际绘制数据的区域。

**创建方式：**
```python
fig, ax = plt.subplots()  # 单坐标轴
ax = fig.add_subplot(111)  # 替代方法
```

**绘图方法：**

**折线图：**
- `ax.plot(x, y, **kwargs)` - 折线图
- `ax.step(x, y, where='pre'/'mid'/'post')` - 阶梯图
- `ax.errorbar(x, y, yerr, xerr)` - 误差线

**散点图：**
- `ax.scatter(x, y, s=size, c=color, marker='o', alpha=0.5)` - 散点图

**条形图：**
- `ax.bar(x, height, width=0.8, align='center')` - 垂直条形图
- `ax.barh(y, width)` - 水平条形图

**统计图：**
- `ax.hist(data, bins=10, density=False)` - 直方图
- `ax.boxplot(data, labels=None)` - 箱线图
- `ax.violinplot(data)` - 小提琴图

**二维图：**
- `ax.imshow(array, cmap='viridis', aspect='auto')` - 显示图像/矩阵
- `ax.contour(X, Y, Z, levels=10)` - 等高线
- `ax.contourf(X, Y, Z, levels=10)` - 填充等高线
- `ax.pcolormesh(X, Y, Z)` - 伪彩色图

**填充：**
- `ax.fill_between(x, y1, y2, alpha=0.3)` - 曲线间填充
- `ax.fill_betweenx(y, x1, x2)` - 垂直曲线间填充

**文本与标注：**
- `ax.text(x, y, text, fontsize=12)` - 添加文本
- `ax.annotate(text, xy=(x, y), xytext=(x2, y2), arrowprops={})` - 带箭头标注

**自定义方法：**

**标签与标题：**
- `ax.set_xlabel(label, fontsize=12)` - 设置x轴标签
- `ax.set_ylabel(label, fontsize=12)` - 设置y轴标签
- `ax.set_title(title, fontsize=14)` - 设置坐标轴标题

**范围与比例：**
- `ax.set_xlim(left, right)` - 设置x轴范围
- `ax.set_ylim(bottom, top)` - 设置y轴范围
- `ax.set_xscale('linear'/'log'/'symlog')` - 设置x轴比例
- `ax.set_yscale('linear'/'log'/'symlog')` - 设置y轴比例

**刻度：**
- `ax.set_xticks(positions)` - 设置x刻度位置
- `ax.set_xticklabels(labels)` - 设置x刻度标签
- `ax.tick_params(axis='both', labelsize=10)` - 自定义刻度外观

**网格与边框：**
- `ax.grid(True, alpha=0.3, linestyle='--')` - 添加网格
- `ax.spines['top'].set_visible(False)` - 隐藏顶部边框
- `ax.spines['right'].set_visible(False)` - 隐藏右侧边框

**图例：**
- `ax.legend(loc='best', fontsize=10, frameon=True)` - 添加图例
- `ax.legend(handles, labels)` - 自定义图例

**比例与布局：**
- `ax.set_aspect('equal'/'auto'/ratio)` - 设置宽高比
- `ax.invert_xaxis()` - 反转x轴
- `ax.invert_yaxis()` - 反转y轴

### pyplot 模块

快速绘图的高级接口。

**图形创建：**
- `plt.figure()` - 创建新图形
- `plt.subplots()` - 创建图形和坐标轴
- `plt.subplot()` - 添加子图到当前图形

**绘图（使用当前坐标轴）：**
- `plt.plot()` - 折线图
- `plt.scatter()` - 散点图
- `plt.bar()` - 条形图
- `plt.hist()` - 直方图
- （所有坐标轴方法均可用）

**显示与保存：**
- `plt.show()` - 显示图形
- `plt.savefig()` - 保存图形
- `plt.close()` - 关闭图形

**样式：**
- `plt.style.use(style_name)` - 应用样式表
- `plt.style.available` - 列出可用样式

**状态管理：**
- `plt.gca()` - 获取当前坐标轴
- `plt.gcf()` - 获取当前图形
- `plt.sca(ax)` - 设置当前坐标轴
- `plt.clf()` - 清除当前图形
- `plt.cla()` - 清除当前坐标轴

## 线条与标记样式

### 线条样式
- `'-'` 或 `'solid'` - 实线
- `'--'` 或 `'dashed'` - 虚线
- `'-.'` 或 `'dashdot'` - 点划线
- `':'` 或 `'dotted'` - 点线
- `''` 或 `' '` 或 `'None'` - 无线条

### 标记样式
- `'.'` - 点标记
- `'o'` - 圆形标记
- `'v'`, `'^'`, `'<'`, `'>'` - 三角形标记
- `'s'` - 方形标记
- `'p'` - 五边形标记
- `'*'` - 星形标记
- `'h'`, `'H'` - 六边形标记
- `'+'` - 加号标记
- `'x'` - X标记
- `'D'`, `'d'` - 菱形标记

### 颜色规范

**单字符简写：**
- `'b'` - 蓝色
- `'g'` - 绿色
- `'r'` - 红色
- `'c'` - 青色
- `'m'` - 品红
- `'y'` - 黄色
- `'k'` - 黑色
- `'w'` - 白色

**命名颜色：**
- `'steelblue'`, `'coral'`, `'teal'` 等
- 完整列表：https://matplotlib.org/stable/gallery/color/named_colors.html

**其他格式：**
- 十六进制：`'#FF5733'`
- RGB元组：`(0.1, 0.2, 0.3)`
- RGBA元组：`(0.1, 0.2, 0.3, 0.5)`

## 常用参数

### 绘图函数参数

```python
ax.plot(x, y,
    color='blue',           # 线条颜色
    linewidth=2,            # 线条宽度
    linestyle='--',         # 线条样式
    marker='o',             # 标记样式
    markersize=8,           # 标记尺寸
    markerfacecolor='red',  # 标记填充色
    markeredgecolor='black',# 标记边缘色
    markeredgewidth=1,      # 标记边缘宽度
    alpha=0.7,              # 透明度 (0-1)
    label='data',           # 图例标签
    zorder=2,               # 绘制顺序
    rasterized=True         # 栅格化减小文件尺寸
)
```

### 散点图函数参数

```python
ax.scatter(x, y,
    s=50,                   # 尺寸 (标量或数组)
    c='blue',               # 颜色 (标量/数组/序列)
    marker='o',             # 标记样式
    cmap='viridis',         # 颜色映射 (当c为数值时)
    alpha=0.5,              # 透明度
    edgecolors='black',     # 边缘颜色
    linewidths=1,           # 边缘宽度
    vmin=0, vmax=1,         # 颜色范围限制
    label='data'            # 图例标签
)
```

### 文本参数

```python
ax.text(x, y, text,
    fontsize=12,            # 字体大小
    fontweight='normal',    # 'normal', 'bold', 'heavy', 'light'
    fontstyle='normal',     # 'normal', 'italic', 'oblique'
    fontfamily='sans-serif',# 字体族
    color='black',          # 文本颜色
    alpha=1.0,              # 透明度
    ha='center',            # 水平对齐: 'left', 'center', 'right'
    va='center',            # 垂直对齐: 'top', 'center', 'bottom', 'baseline'
    rotation=0,             # 旋转角度 (度)
    bbox=dict(              # 背景框
        facecolor='white',
        edgecolor='black',
        boxstyle='round'
    )
)
```

## rcParams 配置

全局自定义常用 rcParams 设置：

```python
# 字体设置
plt.rcParams['font.family'] = 'sans-serif'
plt.rcParams['font.sans-serif'] = ['Arial', 'Helvetica']
plt.rcParams['font.size'] = 12

# 图形设置
plt.rcParams['figure.figsize'] = (
