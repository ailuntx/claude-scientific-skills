# Matplotlib 样式指南

全面指导如何设置和自定义 matplotlib 可视化效果。

## 色彩映射

### 色彩映射分类

**1. 感知均匀顺序型**
最适合从低到高有序排列的数据。
- `viridis`（默认，色盲友好）
- `plasma`
- `inferno`
- `magma`
- `cividis`（针对色盲用户优化）

**用法：**
```python
im = ax.imshow(data, cmap='viridis')
scatter = ax.scatter(x, y, c=values, cmap='plasma')
```

**2. 顺序型**
传统的有序数据色彩映射。
- `Blues`, `Greens`, `Reds`, `Oranges`, `Purples`
- `YlOrBr`, `YlOrRd`, `OrRd`, `PuRd`
- `BuPu`, `GnBu`, `PuBu`, `YlGnBu`

**3. 发散型**
最适合具有中心点（如零值、平均值）的数据。
- `coolwarm`（蓝到红）
- `RdBu`（红-蓝）
- `RdYlBu`（红-黄-蓝）
- `RdYlGn`（红-黄-绿）
- `PiYG`, `PRGn`, `BrBG`, `PuOr`, `RdGy`

**用法：**
```python
# 将色彩映射中心设为零点
im = ax.imshow(data, cmap='coolwarm', vmin=-1, vmax=1)
```

**4. 定性型**
最适合无内在顺序的分类/名义数据。
- `tab10`（10种区分色）
- `tab20`（20种区分色）
- `Set1`, `Set2`, `Set3`
- `Pastel1`, `Pastel2`
- `Dark2`, `Accent`, `Paired`

**用法：**
```python
colors = plt.cm.tab10(np.linspace(0, 1, n_categories))
for i, category in enumerate(categories):
    ax.plot(x, y[i], color=colors[i], label=category)
```

**5. 循环型**
最适合周期性数据（如相位、角度）。
- `twilight`
- `twilight_shifted`
- `hsv`

### 色彩映射最佳实践

1. **避免使用 `jet` 色彩映射** - 非感知均匀，易产生误导
2. **使用感知均匀色彩映射** - `viridis`, `plasma`, `cividis`
3. **考虑色盲用户** - 使用 `viridis`, `cividis` 或通过色盲模拟器测试
4. **根据数据类型匹配色彩映射**：
   - 顺序型：递增/递减数据
   - 发散型：含中心点的数据
   - 定性型：分类数据
5. **反转色彩映射** - 添加 `_r` 后缀：`viridis_r`, `coolwarm_r`

### 创建自定义色彩映射

```python
from matplotlib.colors import LinearSegmentedColormap

# 从颜色列表创建
colors = ['blue', 'white', 'red']
n_bins = 100
cmap = LinearSegmentedColormap.from_list('custom', colors, N=n_bins)

# 从RGB值创建
colors = [(0, 0, 1), (1, 1, 1), (1, 0, 0)]  # RGB元组
cmap = LinearSegmentedColormap.from_list('custom', colors)

# 使用自定义色彩映射
ax.imshow(data, cmap=cmap)
```

### 离散色彩映射

```python
import matplotlib.colors as mcolors

# 从连续色彩映射创建离散版本
cmap = plt.cm.viridis
bounds = np.linspace(0, 10, 11)
norm = mcolors.BoundaryNorm(bounds, cmap.N)
im = ax.imshow(data, cmap=cmap, norm=norm)
```

## 样式表

### 使用内置样式

```python
# 列出可用样式
print(plt.style.available)

# 应用样式
plt.style.use('seaborn-v0_8-darkgrid')

# 应用多个样式（后者覆盖前者）
plt.style.use(['seaborn-v0_8-whitegrid', 'seaborn-v0_8-poster'])

# 临时使用样式
with plt.style.context('ggplot'):
    fig, ax = plt.subplots()
    ax.plot(x, y)
```

### 常用内置样式

- `default` - Matplotlib 默认样式
- `classic` - 经典 matplotlib 样式（2.0之前）
- `seaborn-v0_8-*` - Seaborn 风格样式
  - `seaborn-v0_8-darkgrid`, `seaborn-v0_8-whitegrid`
  - `seaborn-v0_8-dark`, `seaborn-v0_8-white`
  - `seaborn-v0_8-ticks`, `seaborn-v0_8-poster`, `seaborn-v0_8-talk`
- `ggplot` - ggplot2 风格
- `bmh` - Bayesian Methods for Hackers 风格
- `fivethirtyeight` - FiveThirtyEight 风格
- `grayscale` - 灰度风格

### 创建自定义样式表

创建名为 `custom_style.mplstyle` 的文件：

```
# custom_style.mplstyle

# 图形设置
figure.figsize: 10, 6
figure.dpi: 100
figure.facecolor: white

# 字体设置
font.family: sans-serif
font.sans-serif: Arial, Helvetica
font.size: 12

# 坐标轴设置
axes.labelsize: 14
axes.titlesize: 16
axes.facecolor: white
axes.edgecolor: black
axes.linewidth: 1.5
axes.grid: True
axes.axisbelow: True

# 网格设置
grid.color: gray
grid.linestyle: --
grid.linewidth: 0.5
grid.alpha: 0.3

# 线条设置
lines.linewidth: 2
lines.markersize: 8

# 刻度设置
xtick.labelsize: 10
ytick.labelsize: 10
xtick.direction: in
ytick.direction: in
xtick.major.size: 6
ytick.major.size: 6
xtick.minor.size: 3
ytick.minor.size: 3

# 图例设置
legend.fontsize: 12
legend.frameon: True
legend.framealpha: 0.8
legend.fancybox: True

# 保存设置
savefig.dpi: 300
savefig.bbox: tight
savefig.facecolor: white
```

加载使用：
```python
plt.style.use('path/to/custom_style.mplstyle')
```

## rcParams 配置

### 全局配置

```python
import matplotlib.pyplot as plt

# 全局配置
plt.rcParams['figure.figsize'] = (10, 6)
plt.rcParams['font.size'] = 12
plt.rcParams['axes.labelsize'] = 14

# 批量更新
plt.rcParams.update({
    'figure.figsize': (10, 6),
    'font.size': 12,
    'axes.labelsize': 14,
    'axes.titlesize': 16,
    'lines.linewidth': 2
})
```

### 临时配置

```python
# 上下文管理器实现临时修改
with plt.rc_context({'font.size': 14, 'lines.linewidth': 2.5}):
    fig, ax = plt.subplots()
    ax.plot(x, y)
```

### 常用 rcParams

**图形设置：**
```python
plt.rcParams['figure.figsize'] = (10, 6)
plt.rcParams['figure.dpi'] = 100
plt.rcParams['figure.facecolor'] = 'white'
plt.rcParams['figure.edgecolor'] = 'white'
plt.rcParams['figure.autolayout'] = False
plt.rcParams['figure.constrained_layout.use'] = True
```

**字体设置：**
```python
plt.rcParams['font.family'] = 'sans-serif'
plt.rcParams['font.sans-serif'] = ['Arial', 'Helvetica', 'DejaVu Sans']
plt.rcParams['font.size'] = 12
plt.rcParams['font.weight'] = 'normal'
```

**坐标轴设置：**
```python
plt.rcParams['axes.facecolor'] = 'white'
plt.rcParams['axes.edgecolor'] = 'black'
plt.rcParams['axes.linewidth'] = 1.5
plt.rcParams['axes.grid'] = True
plt.rcParams['axes.labelsize'] = 14
plt.rcParams['axes.titlesize'] = 16
plt.rcParams['axes.labelweight'] = 'normal'
plt.rcParams['axes.spines.top'] = True
plt.rcParams['axes.spines.right'] = True
```

**线条设置：**
```python
plt.rcParams['lines.linewidth'] = 2
plt.rcParams['lines.linestyle'] = '-'
plt.rcParams['lines.marker'] = 'None'
plt.rcParams['lines.markersize'] = 6
```

**保存设置：**
```python
plt.rcParams['savefig.dpi'] = 300
plt.rcParams['savefig.format'] = 'png'
plt.rcParams['savefig.bbox'] = 'tight'
plt.rcParams['savefig.pad_inches'] = 0.1
plt.rcParams['savefig.transparent'] = False
```

## 调色板

### 命名颜色集

```python
# Tableau 颜色
tableau_colors = plt.cm.tab10.colors

# CSS4 颜色（子集）
css_colors = ['steelblue', 'coral', 'teal', 'goldenrod', 'crimson']

# 手动定义
custom_colors = ['#1f77b4', '#ff7f0e', '#2ca02c', '#d62728', '#9467bd']
```

### 颜色循环

```python
# 设置默认颜色循环
from cycler import cycler
colors = ['#1f77b4', '#ff7f0e', '#2ca02c', '#d62728']
plt.rcParams['axes.prop_cycle'] = cycler(color=colors)

# 组合颜色与线型
plt.rcParams['axes.prop_cycle'] = cycler(color=colors) + cycler(linestyle=['-', '--', ':', '-.'])
```

### 调色板生成

```python
# 从色彩映射中均匀取色
n_colors = 5
colors = plt.cm.viridis(np.linspace(0, 1, n_colors))

# 在绘图中使用
for i, (x, y) in enumerate(data):
    ax.plot(x, y, color=colors[i])
```

## 排版

### 字体配置

```python
# 设置衬线字体
plt.rcParams['font.family'] = 'serif'
plt.rcParams['font.serif'] = ['Times New Roman', 'DejaVu Serif']

# 设置无衬线字体
plt.rcParams['font.family'] = 'sans-serif'
plt.rcParams['font.sans-serif'] = ['Arial', 'Helvetica']

# 设置等宽字体
plt.rcParams['font.family'] = 'monospace'
plt.rcParams['font.monospace'] = ['Courier New', 'DejaVu Sans Mono']
```

### 文本字体属性

```python
from matplotlib import font_manager

# 指定字体属性
ax.text(x, y, '文本',
        fontsize=14,
        fontweight='bold',  # 'normal', 'bold', 'heavy', 'light'
        fontstyle='italic',  # 'normal', 'italic', 'oblique'
        fontfamily='serif')

# 使用特定字体文件
prop = font_manager.FontProperties(fname='path/to/font.ttf')
ax.text(x, y, '文本', fontproperties=prop)
```

### 数学文本

```python
# LaTeX 风格数学公式
ax.set_title(r'$\alpha > \beta$')
ax.set_xlabel(r'$\mu \pm \sigma$')
ax.text(x, y, r'$\int_0^\infty e^{-x} dx = 1$')

# 上下标
ax.set_ylabel(r'$y = x^2 + 2x + 1$')
ax.text(x, y, r'$x_1, x_2, \ldots, x_n$')

# 希腊字母
ax.text(x, y, r'$\alpha, \beta, \gamma, \delta, \epsilon$')
```

### 使用完整 LaTeX

```python
# 启用完整 LaTeX 渲染（需安装 LaTeX）
plt.rcParams['text.usetex'] = True
plt.rcParams['text.latex.preamble'] = r'\usepackage{amsmath}'

ax.set_title(r'\textbf{加粗标题}')
ax.set_xlabel(r'时间 $t$ (秒)')
```

## 坐标轴与网格

### 坐标轴定制

```python
# 隐藏特定坐标轴
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

# 移动坐标轴位置
ax.spines['left'].set_position(('outward', 10))
ax.spines['bottom'].set_position(('data', 0))

# 修改坐标轴颜色和宽度
ax.spines['left'].set_color('red')
ax.spines['bottom'].set_linewidth(2)
```

### 网格定制

```python
# 基础网格
ax.grid(True)

# 自定义网格
ax.grid(True, which='major', linestyle='--', linewidth=0.8, alpha=0.3)
ax.grid(True, which='minor', linestyle=':', linewidth=0.5, alpha=0.2)

# 单轴网格
ax.grid(True, axis='x')  # 仅垂直线
ax.grid(True, axis='y')  # 仅水平线

# 网格与数据层叠顺序
ax.set_axisbelow(True)  # 网格在数据下方
```

## 图例定制

### 图例定位

```python
# 定位字符串
ax.legend(loc='best')  # 自动最佳位置
ax.legend(loc='upper right')
ax.legend(loc='upper left')
ax.legend(loc='lower right')
ax.legend(loc='lower left')
ax.legend(loc='center')
ax.legend(loc='upper center')
ax.legend(loc='lower center')
ax.legend(loc='center left')
ax.legend(loc='center right')

# 精确定位 (bbox_to_anchor)
ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left')  # 绘图区外
ax.legend(bbox_to_anchor=(0.5, -0.15), loc='upper center', ncol=3)  # 绘图区下方
```

### 图例样式

```python
ax.legend(
    fontsize=12,
    frameon=True,           # 显示边框
    framealpha=0.9,         # 边框透明度
    fancybox=True,          # 圆角边框
    shadow=True,            # 阴影效果
    ncol=2,                 # 列数
    title='图例标题',       # 图例标题
    title_fontsize=14,      # 标题字号
    edgecolor='black',      # 边框颜色
    facecolor='white'       # 背景颜色
)
```

### 自定义图例项

```python
from matplotlib.lines import Line2D

# 创建自定义图例句柄
custom_lines = [Line2D([0], [0], color='red', lw=2),
                Line2D([0], [0], color='blue', lw=2, linestyle='--'),
                Line2D([0], [0], marker='o', color='w', markerfacecolor='green', markersize=10)]

ax.legend(custom_lines, ['标签1', '标签2', '标签3'])
```

## 布局与间距

### 约束布局

```python
# 首选方法（自动调整）
fig, axes = plt.subplots(2, 2, constrained_layout=True)
```

### 紧凑布局

```python
# 替代方法
fig, axes = plt.subplots(2, 2)
plt.tight_layout(pad=1.5, h_pad=2.0, w_pad=2.0)
```

### 手动调整

```python
# 精细控制
plt.subplots_adjust(left=0.1, right=0.9, top=0.9, bottom=0.1,
                    hspace=0.3, wspace=0.4)
```

## 专业出版样式

学术出版级图表配置示例：

```python
# 出版级样式配置
plt.rcParams.update({
    # 图形设置
    'figure.figsize': (8, 6),
    'figure.dpi': 100,
    'savefig.dpi': 300,
    '

'lines.markersize': 8,

    # 刻度线
    'xtick.labelsize': 10,
    'ytick.labelsize': 10,
    'xtick.major.size': 6,
    'ytick.major.size': 6,
    'xtick.major.width': 1.5,
    'ytick.major.width': 1.5,
    'xtick.direction': 'in',
    'ytick.direction': 'in',

    # 图例
    'legend.fontsize': 10,
    'legend.frameon': True,
    'legend.framealpha': 1.0,
    'legend.edgecolor': 'black'
})

## 深色主题

```python
# 深色背景样式
plt.style.use('dark_background')

# 或手动配置
plt.rcParams.update({
    'figure.facecolor': '#1e1e1e',
    'axes.facecolor': '#1e1e1e',
    'axes.edgecolor': 'white',
    'axes.labelcolor': 'white',
    'text.color': 'white',
    'xtick.color': 'white',
    'ytick.color': 'white',
    'grid.color': 'gray',
    'legend.facecolor': '#1e1e1e',
    'legend.edgecolor': 'white'
})
```

## 颜色可访问性

### 色盲友好调色板

```python
# 使用色盲友好的颜色映射
colorblind_friendly = ['viridis', 'plasma', 'cividis']

# 色盲友好的离散颜色
cb_colors = ['#0173B2', '#DE8F05', '#029E73', '#CC78BC',
             '#CA9161', '#949494', '#ECE133', '#56B4E9']

# 使用模拟工具测试或采用这些经过验证的调色板
```

### 高对比度

```python
# 确保足够的对比度
plt.rcParams['axes.edgecolor'] = 'black'
plt.rcParams['axes.linewidth'] = 2
plt.rcParams['xtick.major.width'] = 2
plt.rcParams['ytick.major.width'] = 2
```
