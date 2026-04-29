# 科学调色板与使用指南

## 概述

科学可视化中的色彩选择对可访问性、清晰度和数据准确表达至关重要。本参考提供色盲友好调色板及色彩使用最佳实践。

## 色盲友好调色板

### Okabe-Ito 调色板（推荐用于分类数据）

Okabe-Ito 调色板专为所有类型色盲人群设计，确保可区分性。

```python
# Okabe-Ito 颜色 (RGB值)
okabe_ito = {
    'orange': '#E69F00',      # RGB: (230, 159, 0)
    'sky_blue': '#56B4E9',    # RGB: (86, 180, 233)
    'bluish_green': '#009E73', # RGB: (0, 158, 115)
    'yellow': '#F0E442',      # RGB: (240, 228, 66)
    'blue': '#0072B2',        # RGB: (0, 114, 178)
    'vermillion': '#D55E00',  # RGB: (213, 94, 0)
    'reddish_purple': '#CC79A7', # RGB: (204, 121, 167)
    'black': '#000000'        # RGB: (0, 0, 0)
}
```

**Matplotlib 应用：**
```python
import matplotlib.pyplot as plt

colors = ['#E69F00', '#56B4E9', '#009E73', '#F0E442',
          '#0072B2', '#D55E00', '#CC79A7', '#000000']
plt.rcParams['axes.prop_cycle'] = plt.cycler(color=colors)
```

**Seaborn 应用：**
```python
import seaborn as sns

okabe_ito_palette = ['#E69F00', '#56B4E9', '#009E73', '#F0E442',
                      '#0072B2', '#D55E00', '#CC79A7']
sns.set_palette(okabe_ito_palette)
```

**Plotly 应用：**
```python
import plotly.graph_objects as go

okabe_ito_plotly = ['#E69F00', '#56B4E9', '#009E73', '#F0E442',
                     '#0072B2', '#D55E00', '#CC79A7']
fig = go.Figure()
# 应用于离散色阶
```

### Wong 调色板（分类数据备选方案）

Bang Wong 设计的另一优秀色盲友好调色板（发表于 Nature Methods）。

```python
wong_palette = {
    'black': '#000000',
    'orange': '#E69F00',
    'sky_blue': '#56B4E9',
    'green': '#009E73',
    'yellow': '#F0E442',
    'blue': '#0072B2',
    'vermillion': '#D55E00',
    'purple': '#CC79A7'
}
```

### Paul Tol 调色板

Paul Tol 为不同场景设计了多种科学优化调色板。

**明亮调色板（最多7类）：**
```python
tol_bright = ['#4477AA', '#EE6677', '#228833', '#CCBB44',
              '#66CCEE', '#AA3377', '#BBBBBB']
```

**柔和调色板（最多9类）：**
```python
tol_muted = ['#332288', '#88CCEE', '#44AA99', '#117733',
             '#999933', '#DDCC77', '#CC6677', '#882255', '#AA4499']
```

**高对比度（仅3类）：**
```python
tol_high_contrast = ['#004488', '#DDAA33', '#BB5566']
```

## 顺序色阶（连续数据）

顺序色阶使用单一色调表示从低到高的数值变化。

### 感知均匀色阶

这些色阶在色彩尺度上具有均匀的感知变化。

**Viridis（Matplotlib 默认）：**
- 色盲友好
- 灰度打印效果佳
- 感知均匀
```python
plt.imshow(data, cmap='viridis')
```

**Cividis：**
- 为色盲观众优化
- 专为绿色盲/红色盲设计
```python
plt.imshow(data, cmap='cividis')
```

**Plasma, Inferno, Magma：**
- Viridis 的感知均匀替代方案
- 满足不同审美偏好
```python
plt.imshow(data, cmap='plasma')
```

### 适用场景
- 显示强度的热力图
- 地理高程数据
- 概率分布
- 任何单变量连续数据（低→高）

## 发散色阶（负值到正值）

发散色阶以中性色为中心，两端使用对比色。

### 色盲安全发散色阶

**RdYlBu（红-黄-蓝）：**
```python
plt.imshow(data, cmap='RdYlBu_r')  # _r 反转：蓝(低)到红(高)
```

**PuOr（紫-橙）：**
- 对色盲观众效果极佳
```python
plt.imshow(data, cmap='PuOr')
```

**BrBG（棕-蓝-绿）：**
- 良好的色盲可访问性
```python
plt.imshow(data, cmap='BrBG')
```

### 避免使用的发散色阶
- **RdGn（红-绿）**：红绿色盲无法区分
- **RdYlGn（红-黄-绿）**：存在相同问题

### 适用场景
- 相关性矩阵
- 变化/差异数据（正负值）
- 偏离中心值
- 温度异常

## 专用调色板

### 基因组学/生物信息学

**序列类型识别：**
```python
# DNA/RNA 碱基
nucleotide_colors = {
    'A': '#00CC00',  # 绿色
    'C': '#0000CC',  # 蓝色
    'G': '#FFB300',  # 橙色
    'T': '#CC0000',  # 红色
    'U': '#CC0000'   # 红色 (RNA)
}
```

**基因表达：**
- 表达水平使用顺序色阶（viridis, YlOrRd）
- log2倍数变化使用发散色阶（RdBu）

### 显微成像

**荧光通道：**
```python
# 传统荧光染料颜色（谨慎使用）
fluorophore_colors = {
    'DAPI': '#0000FF',      # 蓝色 - DNA
    'GFP': '#00FF00',       # 绿色（色盲不友好）
    'RFP': '#FF0000',       # 红色
    'Cy5': '#FF00FF'        # 品红
}

# 色盲友好替代方案
fluorophore_alt = {
    'Channel1': '#0072B2',  # 蓝色
    'Channel2': '#E69F00',  # 橙色（替代绿色）
    'Channel3': '#D55E00',  # 朱红
    'Channel4': '#CC79A7'   # 品红
}
```

## 色彩使用最佳实践

### 分类数据（定性色彩方案）

**应做：**
- 使用 Okabe-Ito 或 Wong 调色板中鲜明的饱和色
- 单图最多使用 7-8 个类别
- 跨图表保持相同类别色彩一致
- 当色彩不足时添加图案/标记

**避免：**
- 使用红绿组合
- 对分类数据使用彩虹（jet）色阶
- 使用难以区分的相似色调

### 连续数据（顺序/发散方案）

**应做：**
- 使用感知均匀色阶（viridis, plasma, cividis）
- 数据存在中心点时选用发散色阶
- 包含带刻度标签的色条
- 测试灰度显示效果

**避免：**
- 使用彩虹（jet）色阶——非感知均匀
- 使用红绿发散色阶
- 热力图中省略色条

## 色盲可访问性测试

### 在线模拟器
- **Coblis**：https://www.color-blindness.com/coblis-color-blindness-simulator/
- **Color Oracle**：Windows/Mac/Linux 免费工具
- **Sim Daltonism**：Mac 应用

### 色觉缺陷类型
- **绿色盲**（约5%男性）：无法区分绿色
- **红色盲**（约2%男性）：无法区分红色
- **蓝色盲**（<1%）：无法区分蓝色（罕见）

### Python 工具
```python
# 使用 colorspacious 模拟色盲视觉
from colorspacious import cspace_convert

def simulate_deuteranopia(image_rgb):
    from colorspacious import cspace_convert
    # 转换为色盲模拟
    # (实现需 colorspacious 库)
    pass
```

## 实现示例

### 设置全局 Matplotlib 样式
```python
import matplotlib.pyplot as plt
import matplotlib as mpl

# 设置 Okabe-Ito 为默认色彩循环
okabe_ito_colors = ['#E69F00', '#56B4E9', '#009E73', '#F0E442',
                     '#0072B2', '#D55E00', '#CC79A7']
mpl.rcParams['axes.prop_cycle'] = mpl.cycler(color=okabe_ito_colors)

# 设置默认色阶为 viridis
mpl.rcParams['image.cmap'] = 'viridis'
```

### Seaborn 自定义调色板
```python
import seaborn as sns

# 设置 Paul Tol 柔和调色板
tol_muted = ['#332288', '#88CCEE', '#44AA99', '#117733',
             '#999933', '#DDCC77', '#CC6677', '#882255', '#AA4499']
sns.set_palette(tol_muted)

# 热力图应用
sns.heatmap(data, cmap='viridis', annot=True)
```

### Plotly 离散色彩应用
```python
import plotly.express as px

# 分类数据使用 Okabe-Ito
okabe_ito_plotly = ['#E69F00', '#56B4E9', '#009E73', '#F0E442',
                     '#0072B2', '#D55E00', '#CC79A7']

fig = px.scatter(df, x='x', y='y', color='category',
                 color_discrete_sequence=okabe_ito_plotly)
```

## 灰度兼容性

所有图表应在灰度模式下保持可读性。通过灰度转换测试：

```python
# 转换为灰度测试
fig.savefig('figure_gray.png', dpi=300, colormap='gray')
```

**灰度兼容策略：**
1. 使用不同线型（实线、虚线、点线）
2. 使用不同标记形状（圆形、方形、三角形）
3. 柱状图添加填充图案
4. 确保色彩间有足够亮度对比度

## 色彩空间

### RGB vs CMYK
- **RGB**（红绿蓝）：用于数字/屏幕显示
- **CMYK**（青品黄黑）：用于印刷

**重要提示：** 印刷与屏幕显示色彩存在差异。印刷准备时：
1. 转换为 CMYK 色彩空间
2. 检查 CMYK 预览效果
3. 确保保留足够对比度

### Matplotlib 色彩空间处理
```python
# 印刷保存 (CMYK)
# 注意：直接CMYK支持有限；建议保存PDF由出版商转换
fig.savefig('figure.pdf', dpi=300)

# 数字用途 (RGB)
fig.savefig('figure.png', dpi=300)
```

## 常见错误

1. **使用jet/彩虹色阶**：非感知均匀，避免使用
2. **红绿组合**：约8%男性无法区分
3. **色彩过多**：超过7-8种难以区分
4. **色彩含义不一致**：跨图表相同色彩应代表相同含义
5. **缺少色条**：连续数据必须包含
6. **对比度过低**：确保色彩差异足够明显
7. **仅依赖色彩**：补充纹理、图案或标记

## 资源

- **ColorBrewer**：http://colorbrewer2.org/ - 按色盲安全选项选择调色板
- **Paul Tol 调色板**：https://personal.sron.nl/~pault/
- **Okabe-Ito 调色板来源**："Color Universal Design" (Okabe & Ito, 2008)
- **Matplotlib 色阶**：https://matplotlib.org/stable/tutorials/colors/colormaps.html
- **Seaborn 调色板**：https://seaborn.pydata.org/tutorial/color_palettes.html
