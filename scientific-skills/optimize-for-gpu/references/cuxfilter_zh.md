# cuxfilter 参考文档

cuxfilter 是 NVIDIA RAPIDS 生态系统中的 GPU 加速跨过滤仪表盘库。它仅需几行 Python 代码即可在 Jupyter 笔记本中创建交互式多图表探索性数据分析仪表盘。所有过滤、分组和聚合操作都通过 cuDF 在 GPU 上执行，只有可视化结果会发送到浏览器。

> **完整文档：** https://docs.rapids.ai/api/cuxfilter/stable/  
> **稳定版本：** 26.02.00  
> **代码仓库：** https://github.com/rapidsai/cuxfilter

## 目录

1. [安装与设置](#安装与设置)
2. [核心概念](#核心概念)
3. [DataFrame：数据加载](#dataframe-加载数据)
4. [图表](#图表)
5. [控件](#控件)
6. [仪表盘创建](#仪表盘创建)
7. [布局](#布局)
8. [主题](#主题)
9. [仪表盘展示与导出](#仪表盘展示与导出)
10. [图可视化](#图可视化)
11. [使用 Dask-cuDF 实现多 GPU](#使用-dask-cudf-实现多-gpu)
12. [互操作性](#互操作性)
13. [性能优化技巧](#性能优化技巧)
14. [常用模式](#常用模式)

---

## 安装与设置

在所有安装说明、文档字符串、注释和错误信息中始终使用 `uv add`（切勿使用 `pip install` 或 `conda install`）。

```bash
uv add --extra-index-url=https://pypi.nvidia.com cuxfilter-cu12   # 适用于 CUDA 12.x
```

cuxfilter 依赖 cuDF，因此会自动安装 `cudf-cu12`。

**平台：** 仅限 Linux 和 WSL2（不支持原生 macOS 或 Windows）。  
**要求：** 支持 CUDA 12.x 的 NVIDIA GPU。

验证安装：
```python
import cuxfilter
import cudf

df = cudf.DataFrame({"x": [1, 2, 3], "y": [4, 5, 6]})
cux_df = cuxfilter.DataFrame.from_dataframe(df)
print(cux_df.data.head())  # 应打印 GPU 数据框
```

---

## 核心概念

cuxfilter 包含五个核心模块：

1. **`cuxfilter.DataFrame`** — 封装 cuDF DataFrame 用于仪表盘。创建仪表盘的入口点。
2. **`cuxfilter.DashBoard`** — 交互式仪表盘对象。通过 DataFrame 和图表创建。
3. **`cuxfilter.charts`** — 图表工厂函数（柱状图、散点图、折线图、热力图、等值线图、图结构、控件）。
4. **`cuxfilter.layouts`** — 预设和自定义的图表排列布局配置。
5. **`cuxfilter.themes`** — 仪表盘视觉主题（默认、暗黑、RAPIDS、RAPIDS 暗黑）。

工作流始终为：**加载数据 → 创建图表 → 构建仪表盘 → 展示**。

---

## DataFrame：加载数据

`cuxfilter.DataFrame` 是起点，它封装 cuDF 或 dask_cudf DataFrame。

### 从 cuDF DataFrame 加载（最常用）
```python
import cudf
import cuxfilter

cudf_df = cudf.DataFrame({
    "x": [0, 1, 2, 3, 4],
    "y": [10.0, 11.0, 12.0, 13.0, 14.0],
    "category": ["A", "B", "A", "B", "A"]
})
cux_df = cuxfilter.DataFrame.from_dataframe(cudf_df)
```

### 从磁盘加载 Arrow 文件
```python
cux_df = cuxfilter.DataFrame.from_arrow("data/my_dataset.arrow")
```

### 从图结构加载（节点+边）
```python
import cugraph

edges = cudf.DataFrame({"source": [0, 1, 2], "target": [1, 2, 3], "weight": [1.0, 2.0, 3.0]})
G = cugraph.Graph()
G.from_cudf_edgelist(edges, destination="target")
cux_df = cuxfilter.DataFrame.load_graph((G.nodes(), G.edges()))
```

或直接从 cuDF DataFrame 加载：
```python
nodes = cudf.DataFrame({"vertex": [0, 1, 2, 3], "x": [0, 1, 2, 3], "y": [4, 4, 2, 6], "attr": [0, 1, 1, 1]})
edges = cudf.DataFrame({"source": [0, 1, 2], "target": [1, 2, 3], "weight": [1.0, 2.0, 3.0]})
cux_df = cuxfilter.DataFrame.load_graph((nodes, edges))
```

### 访问底层数据
```python
cux_df.data  # cuDF DataFrame
cux_df.data["new_col"] = cux_df.data["x"] * 2  # 在创建仪表盘前添加列
```

---

## 图表

所有图表函数通过 `cuxfilter.charts` 访问。使用顶级简写 — 无需直接导入子模块如 `cuxfilter.charts.bokeh` 或 `cuxfilter.charts.datashader`。

### 柱状图（Bokeh）
```python
chart = cuxfilter.charts.bar(
    x="column_name",           # 必需：x 轴列名
    y=None,                    # 可选：y 轴列名（默认为计数）
    data_points=None,          # 分箱数量（None 表示唯一值数量）
    add_interaction=True,      # 启用跨过滤交互
    aggregate_fn="count",      # 'count' 或 'mean'
    step_size=None,            # 范围滑块的步长
    title="",                  # 图表标题
    autoscaling=True,          # 数据更新时自动缩放 y 轴
)
```

### 折线图（Bokeh）
```python
chart = cuxfilter.charts.line(
    x="x_col",
    y="y_col",
    data_points=100,
    add_interaction=True,
)
```

### 散点图（Datashader — 支持百万级点）
```python
chart = cuxfilter.charts.scatter(
    x="x_col",
    y="y_col",
    aggregate_col=None,              # 颜色聚合列
    aggregate_fn="count",            # 'count', 'mean', 'max', 'min'
    color_palette=None,              # Bokeh 调色板或十六进制颜色列表
    point_size=15,
    pixel_shade_type="eq_hist",      # 'eq_hist', 'linear', 'log', 'cbrt'
    pixel_density=0.5,               # [0, 1]，值越高密度越大
    pixel_spread="dynspread",        # 'dynspread' 或 'spread'
    tile_provider=None,              # 地图瓦片（例如地理数据用 "CartoLight"）
    title="",
    unselected_alpha=0.2,            # 未选点的透明度
)
```

### 热力图（Datashader）
```python
chart = cuxfilter.charts.heatmap(
    x="x_col",
    y="y_col",
    aggregate_col="value_col",
    aggregate_fn="mean",             # 'count', 'mean', 'max', 'min'
    color_palette=None,
    point_size=10,
    point_shape="rect_vertical",     # 'circle', 'square', 'rect_vertical', 'rect_horizontal'
    title="",
)
```

### 堆叠折线图（Datashader）
```python
chart = cuxfilter.charts.stacked_lines(
    x="time_col",
    y=["series_a", "series_b", "series_c"],   # y 列列表
    colors=["red", "green", "blue"],
)
```

### 等值线图（Deck.gl — 2D 和 3D 地图）
```python
chart = cuxfilter.charts.choropleth(
    x="zip_code",
    color_column="metric_col",
    color_aggregate_fn="mean",         # 'count', 'mean', 'sum', 'min', 'max', 'std'
    elevation_column="value_col",      # 设置 3D 等值线图，省略则为 2D
    elevation_factor=0.00001,
    elevation_aggregate_fn="sum",
    geoJSONSource="https://url/to/geojson",
    geo_color_palette=None,            # 默认：Inferno256
    nan_color="#d3d3d3",
    tooltip=True,
    tooltip_include_cols=["zip_code", "metric_col"],
    title="",
)
```

### 图结构（Datashader — 节点-链接图）
```python
chart = cuxfilter.charts.datashader.graph(
    node_x="x",                  # 默认 "x"
    node_y="y",                  # 默认 "y"
    node_id="vertex",            # 默认 "vertex"
    edge_source="source",        # 默认 "source"
    edge_target="target",        # 默认 "target"
    node_aggregate_col=None,
    node_color_palette=None,
    edge_color_palette=["#000000"],
    node_point_size=15,
    node_pixel_shade_type="eq_hist",
    edge_render_type="direct",   # 'direct' 或 'curved'（曲线为实验性）
    edge_transparency=0,         # [0, 1]
    tile_provider=None,
    title="",
    unselected_alpha=0.2,
)
```

---

## 控件

控件提供交互式过滤功能，通常放置在侧边栏。

### 范围滑块
```python
widget = cuxfilter.charts.range_slider("numeric_col", step_size=1)
```

### 日期范围滑块
```python
widget = cuxfilter.charts.date_range_slider("datetime_col")
```

### 浮点滑块
```python
widget = cuxfilter.charts.float_slider("float_col", step_size=0.5)
```

### 整型滑块
```python
widget = cuxfilter.charts.int_slider("int_col", step_size=1)
```

### 下拉菜单
```python
widget = cuxfilter.charts.drop_down("category_col")
```

### 多选控件
```python
widget = cuxfilter.charts.multi_select("category_col")
```

### 数值指示器（KPI）
```python
widget = cuxfilter.charts.number(
    expression="column_name",                # 或计算表达式如 "(x + y) / 2"
    aggregate_fn="mean",                     # 'count', 'mean', 'min', 'max', 'sum', 'std'
    title="平均值",
    format="{value:.2f}",                    # Python 格式化字符串
    colors=[(33, "green"), (66, "gold"), (100, "red")],  # 阈值着色
    font_size="18pt",
)
```

### 卡片（Markdown 内容）
```python
import panel as pn
widget = cuxfilter.charts.card(pn.pane.Markdown("## 我的仪表盘\n描述文本"))
```

---

## 仪表盘创建

在 cuxfilter DataFrame 上调用 `.dashboard()` 创建仪表盘：

```python
# 定义图表和控件
chart1 = cuxfilter.charts.scatter(x="x_col", y="y_col")
chart2 = cuxfilter.charts.bar("category_col")
sidebar_widget = cuxfilter.charts.range_slider("value_col")
number_widget = cuxfilter.charts.number(expression="value_col", aggregate_fn="mean", title="平均值")

# 构建仪表盘
d = cux_df.dashboard(
    charts=[chart1, chart2],               # 主区域图表
    sidebar=[sidebar_widget, number_widget],  # 侧边栏控件
    layout=cuxfilter.layouts.feature_and_base,
    theme=cuxfilter.themes.rapids_dark,
    title="我的仪表盘",
    data_size_widget=True,                 # 显示当前数据计数
)
```

### 创建后添加图表
```python
new_chart = cuxfilter.charts.line("x_col", "y_col")
d.add_charts(charts=[new_chart])
# 或
d.add_charts(sidebar=[cuxfilter.charts.card(pn.pane.Markdown("# 备注"))])
```

---

## 布局

### 预设布局

| 布局 | 描述 | 图表数量 |
|------|------|----------|
| `layouts.single_feature` | 单图表全屏 | 1 |
| `layouts.feature_and_base` | 顶部大图+底部小图（66/33 分割） | 2 |
| `layouts.double_feature` | 并排双图表 | 2 |
| `layouts.left_feature_right_double` | 左侧大图+右侧双堆叠图 | 3 |
| `layouts.triple_feature` | 三图表横向排列 | 3 |
| `layouts.feature_and_double_base` | 顶部大图+底部双图 | 3 |
| `layouts.two_by_two` | 2x2 网格 | 4 |
| `layouts.feature_and_triple_base` | 顶部大图+底部三图 | 4 |
| `layouts.feature_and_quad_base` | 顶部大图+底部四图 | 5 |
| `layouts.feature_and_five_edge` | 中心大图+环绕五图 | 6 |
| `layouts.two_by_three` | 2x3 网格 | 6 |
| `layouts.double_feature_quad_base` | 顶部双大图+底部四图 | 6 |
| `layouts.three_by_three` | 3x3 网格 | 9 |

### 使用 `layout_array` 自定义布局

通过 `layout_array` 实现完全控制。它是列表的列表，每个内部列表代表一行，数字对应图表索引（从 1 开始）：

```python
# 图表 1 占据左上 2x2 区域，图表 2 和 3 在右侧
d = cux_df.dashboard(
    charts_list,
    layout_array=[[1, 1, 2, 2], [1, 1, 3, 4]],
    theme=cuxfilter.themes.rapids_dark,
)
```

规则：
- 每个数字映射一个图表（1=第一个图表，2=第二个，以此类推）
- 跨单元格重复数字使图表跨越多个单元格
- 布局自动缩放以适应屏幕

---

## 主题

四种内置主题：

| 主题 | 描述 |
|------|------|
| `cuxfilter.themes.default` | 浅色主题（默认） |
| `cuxfilter.themes.dark` | 暗黑主题 |
| `cuxfilter.themes.rapids` | RAPIDS 品牌浅色主题 |
| `cuxfilter.themes.rapids_dark` | RAPIDS 品牌暗黑主题 |

```python
d = cux_df.dashboard(charts, theme=cuxfilter.themes.rapids_dark)
```

---

## 仪表盘展示与导出

### 在笔记本中内联展示
```python
d.app(sidebar_width=280, width=1200, height=800)
```

### 作为独立 Web 应用展示（打开新浏览器标签）
```python
d.show()
# 或使用自定义 URL/端口
d.show(notebook_url="http://localhost:8888", port=8050)
```

### JupyterHub 部署
```python
d.show(service_proxy="jupyterhub")
```

### 停止服务器
```python
d.stop()
```

### 导出过滤后数据
与仪表盘交互（选择范围、过滤）后，导出当前过滤状态的 DataFrame：

```python
filtered_df = d.export()  # 返回符合当前过滤状态的 cuDF DataFrame
# 同时打印查询字符串，例如："2 <= key <= 4"
```

### 访问仪表盘图表
```python
d.charts  # 图表对象字典
```

---

## 图可视化

cuxfilter 集成 cuGraph 实现交互式图可视化：

```python
import cuxfilter
import cudf
import cugraph

# 创建图结构
edges = cudf.DataFrame({
    "source": [0, 0, 1, 1, 2],
    "target": [1, 2, 2, 3, 3]
})
G = cugraph.Graph()
G.from_cudf_edgelist(edges, destination="target")

# 加载到 cuxfilter（需要节点位置 — 使用 force_atlas2 或类似布局）
positions = cugraph.force_atlas2(G)
nodes = positions.rename(columns={"vertex": "vertex

cuxfilter 可与 `dask_cudf.DataFrame` 无缝协作——只需用它替代 cuDF DataFrame 即可：

```python
import dask_cudf

ddf = dask_cudf.read_parquet("large_dataset/*.parquet")
cux_df = cuxfilter.DataFrame.from_dataframe(ddf)

# 其余操作完全相同
chart = cuxfilter.charts.scatter(x="x", y="y")
d = cux_df.dashboard([chart])
d.app()
```

在以下场景使用 dask_cudf：
- 数据超出单块 GPU 显存容量
- 需要跨多块 GPU 分布式处理
- 需同时处理多个文件

**dask_cudf 支持的图表类型：**
- bokeh：柱状图、折线图
- datashader：散点图、折线图、堆叠线图、热力图、关系图（有限制的边渲染）
- panel_widgets：所有控件
- deckgl：等值线图（2D 和 3D）

---

## 互操作性

cuxfilter 位于 RAPIDS 生态系统的可视化层：

- **cuDF** —— 数据层。cuxfilter.DataFrame 封装 cuDF DataFrame。
- **cuGraph** —— 图分析。使用 `cuxfilter.DataFrame.load_graph()` 可视化 cuGraph 结果。
- **cuML** —— 运行 cuML 后，通过 cuxfilter 可视化结果（如 UMAP 嵌入、聚类标签）。
- **HoloViz 生态系统** —— 基于 Panel、Bokeh、Datashader 和 HoloViews 构建。
- **Deck.gl** —— 基于 WebGL 的等值线地图。

### 典型 RAPIDS + cuxfilter 流程
```python
import cudf
import cuml
import cuxfilter

# 使用 cuDF 加载与预处理
df = cudf.read_parquet("data.parquet")
df = df.dropna().reset_index(drop=True)

# 通过 cuML 运行机器学习（如 UMAP 降维）
from cuml.manifold import UMAP
umap = UMAP(n_components=2)
embedding = umap.fit_transform(df[["feature1", "feature2", "feature3"]])
df["umap_x"] = embedding[:, 0]
df["umap_y"] = embedding[:, 1]

# 通过 cuxfilter 可视化
cux_df = cuxfilter.DataFrame.from_dataframe(df)
scatter = cuxfilter.charts.scatter(
    x="umap_x", y="umap_y",
    aggregate_col="cluster_label",
    aggregate_fn="mean",
    pixel_shade_type="linear",
)
bar = cuxfilter.charts.bar("cluster_label")
d = cux_df.dashboard([scatter, bar], layout=cuxfilter.layouts.feature_and_base)
d.app()
```

---

## 性能优化建议

1. **保持数据在 GPU 上**。使用 `cudf.read_parquet()` 或 `cudf.read_csv()` 加载，再通过 `cuxfilter.DataFrame.from_dataframe()` 封装。避免与 pandas 相互转换。

2. **根据数据规模选用合适图表：**
   - < 1万点：Bokeh 图表（柱状图、折线图）效果良好
   - 1万–1亿+点：Datashader 图表（散点图、热力图）通过服务端栅格化高效处理大型数据集

3. **限制柱状图数据点**。对高基数列设置 `data_points` 参数分桶（如 `bar("col", data_points=50)`）。

4. **优先使用 `float32`**。32位浮点数可加速 GPU 运算。加载前转换：`df["col"] = df["col"].astype("float32")`。

5. **预计算派生列**。在创建仪表板前完成，而非在图表回调函数内计算。

6. **使用 `layout_array`** 布局复杂仪表板，精确控制各图表位置。

7. **增大 `timeout` 值**。若超大数据集缩放操作卡顿，可延长 datashader 图表响应超时。

---

## 常用模式

### 探索性数据分析仪表板
```python
import cudf
import cuxfilter

df = cudf.read_parquet("dataset.parquet")
cux_df = cuxfilter.DataFrame.from_dataframe(df)

# 概览图表
scatter = cuxfilter.charts.scatter(x="feature1", y="feature2", pixel_shade_type="linear")
hist1 = cuxfilter.charts.bar("feature1", data_points=50)
hist2 = cuxfilter.charts.bar("category")

# 侧边栏筛选器
slider = cuxfilter.charts.range_slider("value_col")
dropdown = cuxfilter.charts.drop_down("category")
kpi = cuxfilter.charts.number(expression="value_col", aggregate_fn="mean", title="均值")

d = cux_df.dashboard(
    [scatter, hist1, hist2],
    sidebar=[slider, dropdown, kpi],
    layout=cuxfilter.layouts.feature_and_double_base,
    theme=cuxfilter.themes.rapids_dark,
    title="数据探索器",
)
d.app()
```

### 带地图瓦片的时空仪表板
```python
chart = cuxfilter.charts.scatter(
    x="longitude",
    y="latitude",
    aggregate_col="value",
    aggregate_fn="mean",
    color_palette=["#3182bd", "#6baed6", "#ff0068"],
    tile_provider="CartoLight",
    pixel_shade_type="linear",
    title="地理散点图",
)
```

### 时间序列仪表板
```python
line_chart = cuxfilter.charts.line("timestamp", "metric")
bar_chart = cuxfilter.charts.bar("hour_of_day")
date_slider = cuxfilter.charts.date_range_slider("timestamp")

d = cux_df.dashboard(
    [line_chart, bar_chart],
    sidebar=[date_slider],
    layout=cuxfilter.layouts.feature_and_base,
)
```

### 导出筛选子集供后续分析
```python
# 用户交互后导出当前筛选结果
d.app()
# ...用户在仪表板中筛选数据...
filtered = d.export()  # 当前可见/选中数据的 cuDF DataFrame
# 通过 cuDF、cuML 等继续分析
```
