# Matplotlib 常见问题与解决方案

针对频繁遇到的 matplotlib 问题的故障排除指南。

## 显示与后端问题

### 问题：图表未显示

**问题描述：** `plt.show()` 未显示任何内容

**解决方案：**
```python
# 1. 检查后端是否设置正确（用于交互式环境）
import matplotlib
print(matplotlib.get_backend())

# 2. 尝试不同后端
matplotlib.use('TkAgg')  # 或 'Qt5Agg', 'MacOSX'
import matplotlib.pyplot as plt

# 3. 在 Jupyter 笔记本中使用魔术命令
%matplotlib inline  # 静态图像
# 或
%matplotlib widget  # 交互式图表

# 4. 确保调用 plt.show()
plt.plot([1, 2, 3])
plt.show()
```

### 问题："RuntimeError: main thread is not in main loop"

**问题描述：** 线程交互模式问题

**解决方案：**
```python
# 切换到非交互式后端
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

# 或关闭交互模式
plt.ioff()
```

### 问题：图表未实时更新

**问题描述：** 交互窗口未反映变更

**解决方案：**
```python
# 启用交互模式
plt.ion()

# 每次变更后重绘
plt.plot(x, y)
plt.draw()
plt.pause(0.001)  # 短暂暂停以更新显示
```

## 布局与间距问题

### 问题：标签与标题重叠

**问题描述：** 标签、标题或刻度标签重叠或被截断

**解决方案：**
```python
# 方案1：使用约束布局（推荐）
fig, ax = plt.subplots(constrained_layout=True)

# 方案2：紧凑布局
fig, ax = plt.subplots()
plt.tight_layout()

# 方案3：手动调整边距
plt.subplots_adjust(left=0.15, right=0.95, top=0.95, bottom=0.15)

# 方案4：保存时使用 bbox_inches='tight'
plt.savefig('figure.png', bbox_inches='tight')

# 方案5：旋转长刻度标签
ax.set_xticklabels(labels, rotation=45, ha='right')
```

### 问题：颜色条影响子图尺寸

**问题描述：** 添加颜色条导致图表缩小

**解决方案：**
```python
# 方案1：使用约束布局
fig, ax = plt.subplots(constrained_layout=True)
im = ax.imshow(data)
plt.colorbar(im, ax=ax)

# 方案2：手动指定颜色条尺寸
from mpl_toolkits.axes_grid1 import make_axes_locatable
divider = make_axes_locatable(ax)
cax = divider.append_axes("right", size="5%", pad=0.05)
plt.colorbar(im, cax=cax)

# 方案3：多子图共享颜色条
fig, axes = plt.subplots(1, 3, figsize=(15, 4))
for ax in axes:
    im = ax.imshow(data)
fig.colorbar(im, ax=axes.ravel().tolist(), shrink=0.95)
```

### 问题：子图间距过小

**问题描述：** 多个子图相互重叠

**解决方案：**
```python
# 方案1：使用约束布局
fig, axes = plt.subplots(2, 2, constrained_layout=True)

# 方案2：用 subplots_adjust 调整间距
fig, axes = plt.subplots(2, 2)
plt.subplots_adjust(hspace=0.4, wspace=0.4)

# 方案3：在 tight_layout 中指定间距
plt.tight_layout(h_pad=2.0, w_pad=2.0)
```

## 内存与性能问题

### 问题：多图表内存泄漏

**问题描述：** 创建多个图表时内存占用持续增长

**解决方案：**
```python
# 显式关闭图表
fig, ax = plt.subplots()
ax.plot(x, y)
plt.savefig('plot.png')
plt.close(fig)  # 或 plt.close('all')

# 清除当前图表但不关闭
plt.clf()

# 清除当前坐标轴
plt.cla()
```

### 问题：文件体积过大

**问题描述：** 保存的图表文件过大

**解决方案：**
```python
# 方案1：降低 DPI
plt.savefig('figure.png', dpi=150)  # 替代默认的 300

# 方案2：复杂图表使用栅格化
ax.plot(x, y, rasterized=True)

# 方案3：简单图表使用矢量格式
plt.savefig('figure.pdf')  # 或 .svg

# 方案4：压缩 PNG
plt.savefig('figure.png', dpi=300, optimize=True)
```

### 问题：大数据集绘图缓慢

**问题描述：** 数据点过多导致绘图耗时过长

**解决方案：**
```python
# 方案1：降采样数据
from scipy.signal import decimate
y_downsampled = decimate(y, 10)  # 每10个点保留1个

# 方案2：使用栅格化
ax.plot(x, y, rasterized=True)

# 方案3：使用线条简化
ax.plot(x, y)
for line in ax.get_lines():
    line.set_rasterized(True)

# 方案4：散点图考虑 hexbin 或二维直方图
ax.hexbin(x, y, gridsize=50, cmap='viridis')
```

## 字体与文本问题

### 问题：字体警告

**问题描述：** "findfont: Font family [...] not found"

**解决方案：**
```python
# 方案1：使用可用字体
from matplotlib.font_manager import findfont, FontProperties
print(findfont(FontProperties(family='sans-serif')))

# 方案2：重建字体缓存
import matplotlib.font_manager
matplotlib.font_manager._rebuild()

# 方案3：忽略警告
import warnings
warnings.filterwarnings("ignore", category=UserWarning)

# 方案4：指定备用字体
plt.rcParams['font.sans-serif'] = ['Arial', 'DejaVu Sans', 'sans-serif']
```

### 问题：LaTeX 渲染错误

**问题描述：** 数学公式渲染异常

**解决方案：**
```python
# 方案1：使用 r 前缀的原始字符串
ax.set_xlabel(r'$\alpha$')  # 不要用 '\alpha'

# 方案2：普通字符串中转义反斜杠
ax.set_xlabel('$\\alpha$')

# 方案3：未安装 LaTeX 时禁用
plt.rcParams['text.usetex'] = False

# 方案4：使用 mathtext 替代完整 LaTeX
# Mathtext 无需安装，始终可用
ax.text(x, y, r'$\int_0^\infty e^{-x} dx$')
```

### 问题：文本被截断或超出边界

**问题描述：** 标签或注释显示在图表区域外

**解决方案：**
```python
# 方案1：使用 bbox_inches='tight'
plt.savefig('figure.png', bbox_inches='tight')

# 方案2：调整图表边界
plt.subplots_adjust(left=0.15, right=0.85, top=0.85, bottom=0.15)

# 方案3：将文本裁剪至坐标轴内
ax.text(x, y, 'text', clip_on=True)

# 方案4：使用约束布局
fig, ax = plt.subplots(constrained_layout=True)
```

## 颜色与色谱问题

### 问题：颜色条与图表不匹配

**问题描述：** 颜色条显示范围与数据不一致

**解决方案：**
```python
# 显式设置 vmin 和 vmax
im = ax.imshow(data, vmin=0, vmax=1, cmap='viridis')
plt.colorbar(im, ax=ax)

# 或多图表使用相同归一化
import matplotlib.colors as mcolors
norm = mcolors.Normalize(vmin=data.min(), vmax=data.max())
im1 = ax1.imshow(data1, norm=norm, cmap='viridis')
im2 = ax2.imshow(data2, norm=norm, cmap='viridis')
```

### 问题：颜色显示异常

**问题描述：** 图表中出现意外颜色

**解决方案：**
```python
# 方案1：检查颜色格式
ax.plot(x, y, color='blue')  # 正确
ax.plot(x, y, color=(0, 0, 1))  # 正确 RGB
ax.plot(x, y, color='#0000FF')  # 正确十六进制

# 方案2：验证色谱是否存在
print(plt.colormaps())  # 列出可用色谱

# 方案3：散点图确保 c 形状匹配
ax.scatter(x, y, c=colors)  # colors 长度应与 x, y 相同

# 方案4：检查透明度设置
ax.plot(x, y, alpha=1.0)  # 0=透明, 1=不透明
```

### 问题：色谱方向颠倒

**问题描述：** 色谱方向反向

**解决方案：**
```python
# 添加 _r 后缀反转色谱
ax.imshow(data, cmap='viridis_r')
```

## 坐标轴与比例问题

### 问题：坐标轴范围设置无效

**问题描述：** `set_xlim` 或 `set_ylim` 未生效

**解决方案：**
```python
# 方案1：在绘图后设置
ax.plot(x, y)
ax.set_xlim(0, 10)
ax.set_ylim(-1, 1)

# 方案2：禁用自动缩放
ax.autoscale(False)
ax.set_xlim(0, 10)

# 方案3：使用 axis 方法
ax.axis([xmin, xmax, ymin, ymax])
```

### 问题：对数坐标含零或负值

**问题描述：** 数据含 ≤0 值时使用对数比例报错

**解决方案：**
```python
# 方案1：过滤非正值
mask = (data > 0)
ax.plot(x[mask], data[mask])
ax.set_yscale('log')

# 方案2：正负值数据使用 symlog
ax.set_yscale('symlog')

# 方案3：添加微小偏移
ax.plot(x, data + 1e-10)
ax.set_yscale('log')
```

### 问题：日期显示异常

**问题描述：** 日期坐标轴显示数字而非日期

**解决方案：**
```python
import matplotlib.dates as mdates
import pandas as pd

# 必要时转换为日期时间格式
dates = pd.to_datetime(date_strings)

ax.plot(dates, values)

# 格式化日期坐标轴
ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
ax.xaxis.set_major_locator(mdates.DayLocator(interval=7))
plt.xticks(rotation=45)
```

## 图例问题

### 问题：图例遮挡数据

**问题描述：** 图例遮盖图表关键区域

**解决方案：**
```python
# 方案1：使用 'best' 位置
ax.legend(loc='best')

# 方案2：放置到图表区域外
ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left')

# 方案3：设置半透明图例
ax.legend(framealpha=0.7)

# 方案4：将图例置于图表下方
ax.legend(bbox_to_anchor=(0.5, -0.15), loc='upper center', ncol=3)
```

### 问题：图例条目过多

**问题描述：** 图例因条目过多而混乱

**解决方案：**
```python
# 方案1：仅标记选定条目
for i, (x, y) in enumerate(data):
    label = f'数据 {i}' if i % 5 == 0 else None
    ax.plot(x, y, label=label)

# 方案2：使用多列布局
ax.legend(ncol=3)

# 方案3：创建精简自定义图例
from matplotlib.lines import Line2D
custom_lines = [Line2D([0], [0], color='r'),
                Line2D([0], [0], color='b')]
ax.legend(custom_lines, ['类别 A', '类别 B'])

# 方案4：使用独立图例图表
fig_leg = plt.figure(figsize=(3, 2))
ax_leg = fig_leg.add_subplot(111)
ax_leg.legend(*ax.get_legend_handles_labels(), loc='center')
ax_leg.axis('off')
```

## 3D 绘图问题

### 问题：3D 图表显示扁平

**问题描述：** 3D 图表深度感知困难

**解决方案：**
```python
# 方案1：调整视角
ax.view_init(elev=30, azim=45)

# 方案2：添加网格线
ax.grid(True)

# 方案3：用颜色表示深度
scatter = ax.scatter(x, y, z, c=z, cmap='viridis')

# 方案4：交互式旋转（使用交互后端时）
# 用户可点击拖拽旋转
```

### 问题：3D 坐标轴标签被截断

**问题描述：** 3D 坐标轴标签显示在图表外

**解决方案：**
```python
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')
ax.plot_surface(X, Y, Z)

# 增加内边距
fig.tight_layout(pad=3.0)

# 或保存时使用紧凑边界框
plt.savefig('3d_plot.png', bbox_inches='tight', pad_inches=0.5)
```

## 图像与颜色条问题

### 问题：图像方向颠倒

**问题描述：** 图像显示方向错误

**解决方案：**
```python
# 设置 origin 参数
ax.imshow(img, origin='lower')  # 或 'upper'（默认）

# 或翻转数组
ax.imshow(np.flipud(img))
```

### 问题：图像显示像素化

**问题描述：** 缩放时图像出现块状

**解决方案：**
```python
# 方案1：使用插值
ax.imshow(img, interpolation='bilinear')
# 选项: 'nearest', 'bilinear', 'bicubic', 'spline16', 'spline36' 等

# 方案2：保存时提高 DPI
plt.savefig('figure.png', dpi=300)

# 方案3：适用时使用矢量格式
plt.savefig('figure.pdf')
```

## 常见错误与修复

### "TypeError: 'AxesSubplot' object is not subscriptable"

**问题描述：** 尝试索引单个坐标轴
```python
# 错误
fig, ax = plt.subplots()
ax[0].plot(x, y)  # 报错!

# 正确
fig, ax = plt.subplots()
ax.plot(x, y)
```

### "ValueError: x and y must have same first dimension"

**问题描述：** 数据数组长度不匹配
```python
# 检查形状
print(f"x 形状: {x.shape}, y 形状: {y.shape}")

# 确保长度匹配
assert len(x) == len(y), "x 和 y 长度必须相同"
```

### "AttributeError: 'numpy.ndarray' object has no attribute 'plot'"

**问题描述：** 在数组而非坐标轴上调用 plot
```python
# 错误
data.plot(x, y)

# 正确
ax.plot(x, y)
# 或 pandas 方式
data.plot(ax=ax)
```

## 避免问题的最佳实践

1. **始终使用面向对象接口** - 避免 pyplot 状态机
   ```python
   fig, ax = plt.subplots()  # 推荐
   ax.plot(x, y)
   ```

2. **使用约束布局** - 防止重叠问题
   ```python
   fig, ax = plt.subplots(constrained_layout=True)
   ```

3. **显式关闭图表** - 防止内存泄漏
   ```python
   plt.close(fig)
   ```

4. **创建时设置图表尺寸** - 优于后期调整
   ```python
   fig, ax = plt.subplots(figsize=(10, 6))
   ```

5. **数学文本使用原始字符串** - 避免转义问题
   ```python
   ax.set_xlabel(r'$\alpha$')
   ```

6. **绘图前检查数据形状** - 提前捕获尺寸不匹配
   ```python
   assert len(x) == len(y)
   ```

7. **使用合适 DPI** - 印刷用 300，网页用 150
   ```python
   plt.savefig('figure.png', dpi=300)
   ```

8. **测试不同后端** - 出现显示问题时
   ```python
   import matplotlib
   matplotlib.use('TkAgg')
   ```
