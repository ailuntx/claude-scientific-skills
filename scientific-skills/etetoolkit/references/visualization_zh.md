# ETE 工具包可视化指南

ETE 工具包树形结构可视化完整指南。

## 目录
1. [渲染基础](#渲染基础)
2. [树样式配置](#树样式配置)
3. [节点样式](#节点样式)
4. [图形元素](#图形元素)
5. [布局函数](#布局函数)
6. [高级可视化](#高级可视化)

---

## 渲染基础

### 输出格式

ETE 支持三种主要输出格式：

```python
from ete3 import Tree

tree = Tree("tree.nw")

# PNG (位图，适合演示)
tree.render("output.png", w=800, h=600, units="px", dpi=300)

# PDF (矢量，适合出版物)
tree.render("output.pdf", w=200, units="mm")

# SVG (矢量，可编辑)
tree.render("output.svg")
```

### 单位与尺寸

```python
# 像素
tree.render("tree.png", w=1200, h=800, units="px")

# 毫米
tree.render("tree.pdf", w=210, h=297, units="mm")  # A4尺寸

# 英寸
tree.render("tree.pdf", w=8.5, h=11, units="in")  # 美国信纸尺寸

# 自动尺寸 (保持宽高比)
tree.render("tree.pdf", w=200, units="mm")  # 高度自动计算
```

### 交互式可视化

```python
from ete3 import Tree

tree = Tree("tree.nw")

# 启动图形界面
# - 鼠标滚轮缩放
# - 拖拽平移
# - Ctrl+F 搜索
# - 菜单导出
# - 编辑节点属性
tree.show()
```

---

## 树样式配置

### 基础树样式选项

```python
from ete3 import Tree, TreeStyle

tree = Tree("tree.nw")
ts = TreeStyle()

# 显示选项
ts.show_leaf_name = True          # 显示叶节点名称
ts.show_branch_length = True      # 显示分支长度
ts.show_branch_support = True     # 显示支持值
ts.show_scale = True              # 显示比例尺

# 分支长度缩放
ts.scale = 50                     # 每单位分支长度的像素数
ts.min_leaf_separation = 10       # 叶节点间最小间距(像素)

# 布局方向
ts.rotation = 0                   # 0=从左到右, 90=从上到下
ts.branch_vertical_margin = 10    # 分支间垂直间距

# 树形
ts.mode = "r"                     # "r"=矩形(默认), "c"=圆形

tree.render("tree.pdf", tree_style=ts)
```

### 圆形树

```python
from ete3 import Tree, TreeStyle

tree = Tree("tree.nw")
ts = TreeStyle()

# 圆形模式
ts.mode = "c"
ts.arc_start = 0      # 起始角度(度)
ts.arc_span = 360     # 角度跨度(度, 360=完整圆形)

# 半圆形设置
ts.arc_start = -180
ts.arc_span = 180

tree.render("circular_tree.pdf", tree_style=ts)
```

### 标题与图例

```python
from ete3 import Tree, TreeStyle, TextFace

tree = Tree("tree.nw")
ts = TreeStyle()

# 添加标题
title = TextFace("物种系统发育树", fsize=20, bold=True)
ts.title.add_face(title, column=0)

# 添加图例
ts.legend.add_face(TextFace("红色节点: 高支持度", fsize=10), column=0)
ts.legend.add_face(TextFace("蓝色节点: 低支持度", fsize=10), column=0)

# 图例位置
ts.legend_position = 1  # 1=右上角, 2=左上角, 3=左下角, 4=右下角

tree.render("tree_with_legend.pdf", tree_style=ts)
```

### 自定义背景

```python
from ete3 import Tree, TreeStyle

tree = Tree("tree.nw")
ts = TreeStyle()

# 背景颜色
ts.bgcolor = "#f0f0f0"  # 浅灰色背景

# 树边框
ts.show_border = True

tree.render("tree_background.pdf", tree_style=ts)
```

---

## 节点样式

### 节点样式属性

```python
from ete3 import Tree, NodeStyle

tree = Tree("tree.nw")

for node in tree.traverse():
    nstyle = NodeStyle()

    # 节点尺寸与形状
    nstyle["size"] = 10                # 节点尺寸(像素)
    nstyle["shape"] = "circle"         # "circle", "square", "sphere"

    # 颜色
    nstyle["fgcolor"] = "blue"         # 前景色(节点本身)
    nstyle["bgcolor"] = "lightblue"    # 背景色(仅sphere形状有效)

    # 分支线样式
    nstyle["hz_line_type"] = 0         # 0=实线, 1=虚线, 2=点线
    nstyle["vt_line_type"] = 0         # 垂直线类型
    nstyle["hz_line_color"] = "black"  # 水平线颜色
    nstyle["vt_line_color"] = "black"  # 垂直线颜色
    nstyle["hz_line_width"] = 2        # 线宽(像素)
    nstyle["vt_line_width"] = 2

    node.set_style(nstyle)

tree.render("styled_tree.pdf")
```

### 条件样式

```python
from ete3 import Tree, NodeStyle

tree = Tree("tree.nw")

# 基于节点属性设置样式
for node in tree.traverse():
    nstyle = NodeStyle()

    if node.is_leaf():
        # 叶节点样式
        nstyle["size"] = 8
        nstyle["fgcolor"] = "darkgreen"
        nstyle["shape"] = "circle"
    else:
        # 内部节点基于支持度设置样式
        if node.support > 0.9:
            nstyle["size"] = 6
            nstyle["fgcolor"] = "red"
            nstyle["shape"] = "sphere"
        else:
            nstyle["size"] = 4
            nstyle["fgcolor"] = "gray"
            nstyle["shape"] = "circle"

    # 基于分支长度设置样式
    if node.dist > 1.0:
        nstyle["hz_line_width"] = 3
        nstyle["hz_line_color"] = "blue"
    else:
        nstyle["hz_line_width"] = 1
        nstyle["hz_line_color"] = "black"

    node.set_style(nstyle)

tree.render("conditional_styled_tree.pdf")
```

### 隐藏节点

```python
from ete3 import Tree, NodeStyle

tree = Tree("tree.nw")

# 隐藏特定节点
for node in tree.traverse():
    if node.support < 0.5:  # 隐藏低支持度节点
        nstyle = NodeStyle()
        nstyle["draw_descendants"] = False  # 不绘制该节点的子树
        nstyle["size"] = 0                   # 使节点不可见
        node.set_style(nstyle)

tree.render("filtered_tree.pdf")
```

---

## 图形元素

图形元素是附加在节点周围特定位置的视觉组件。

### 元素位置

- `"branch-right"`: 分支右侧(节点后)
- `"branch-top"`: 分支上方
- `"branch-bottom"`: 分支下方
- `"aligned"`: 树边缘对齐列(用于叶节点)

### 文本元素

```python
from ete3 import Tree, TreeStyle, TextFace

tree = Tree("tree.nw")

def layout(node):
    if node.is_leaf():
        # 添加物种名称
        name_face = TextFace(node.name, fsize=12, fgcolor="black")
        node.add_face(name_face, column=0, position="branch-right")

        # 添加附加文本
        info_face = TextFace(f"长度: {node.dist:.3f}", fsize=8, fgcolor="gray")
        node.add_face(info_face, column=1, position="branch-right")
    else:
        # 添加支持值
        if node.support:
            support_face = TextFace(f"{node.support:.2f}", fsize=8, fgcolor="red")
            node.add_face(support_face, column=0, position="branch-top")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = False  # 使用自定义名称

tree.render("tree_textfaces.pdf", tree_style=ts)
```

### 属性元素

直接显示节点属性：

```python
from ete3 import Tree, TreeStyle, AttrFace

tree = Tree("tree.nw")

# 添加自定义属性
for leaf in tree:
    leaf.add_feature("habitat", "水生" if "fish" in leaf.name else "陆生")
    leaf.add_feature("temperature", 20)

def layout(node):
    if node.is_leaf():
        # 直接显示属性
        habitat_face = AttrFace("habitat", fsize=10)
        node.add_face(habitat_face, column=0, position="aligned")

        temp_face = AttrFace("temperature", fsize=10)
        node.add_face(temp_face, column=1, position="aligned")

ts = TreeStyle()
ts.layout_fn = layout

tree.render("tree_attrfaces.pdf", tree_style=ts)
```

### 圆形元素

```python
from ete3 import Tree, TreeStyle, CircleFace, TextFace

tree = Tree("tree.nw")

# 添加栖息地注释
for leaf in tree:
    leaf.add_feature("habitat", "海洋" if "fish" in leaf.name else "陆地")

def layout(node):
    if node.is_leaf():
        # 基于栖息地的彩色圆形
        color = "blue" if node.habitat == "海洋" else "green"
        circle = CircleFace(radius=5, color=color, style="circle")
        node.add_face(circle, column=0, position="aligned")

        # 标签
        name = TextFace(node.name, fsize=10)
        node.add_face(name, column=1, position="aligned")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = False

tree.render("tree_circles.pdf", tree_style=ts)
```

### 图像元素

为节点添加图片：

```python
from ete3 import Tree, TreeStyle, ImgFace, TextFace

tree = Tree("tree.nw")

def layout(node):
    if node.is_leaf():
        # 添加物种图片
        img_path = f"images/{node.name}.png"  # 图片路径
        try:
            img_face = ImgFace(img_path, width=50, height=50)
            node.add_face(img_face, column=0, position="aligned")
        except:
            pass  # 图片不存在则跳过

        # 添加名称
        name_face = TextFace(node.name, fsize=10)
        node.add_face(name_face, column=1, position="aligned")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = False

tree.render("tree_images.pdf", tree_style=ts)
```

### 柱状图元素

```python
from ete3 import Tree, TreeStyle, BarChartFace, TextFace

tree = Tree("tree.nw")

# 添加柱状图数据
for leaf in tree:
    leaf.add_feature("values", [1.2, 2.3, 0.5, 1.8])  # 多组数值

def layout(node):
    if node.is_leaf():
        # 添加柱状图
        chart = BarChartFace(
            node.values,
            width=100,
            height=40,
            colors=["red", "blue", "green", "orange"],
            labels=["A", "B", "C", "D"]
        )
        node.add_face(chart, column=0, position="aligned")

        # 添加名称
        name = TextFace(node.name, fsize=10)
        node.add_face(name, column=1, position="aligned")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = False

tree.render("tree_barcharts.pdf", tree_style=ts)
```

### 饼图元素

```python
from ete3 import Tree, TreeStyle, PieChartFace, TextFace

tree = Tree("tree.nw")

# 添加数据
for leaf in tree:
    leaf.add_feature("proportions", [25, 35, 40])  # 百分比

def layout(node):
    if node.is_leaf():
        # 添加饼图
        pie = PieChartFace(
            node.proportions,
            width=30,
            height=30,
            colors=["red", "blue", "green"]
        )
        node.add_face(pie, column=0, position="aligned")

        name = TextFace(node.name, fsize=10)
        node.add_face(name, column=1, position="aligned")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = False

tree.render("tree_piecharts.pdf", tree_style=ts)
```

### 序列元素(用于比对)

```python
from ete3 import PhyloTree, TreeStyle, SeqMotifFace

tree = PhyloTree("tree.nw")
tree.link_to_alignment("alignment.fasta")

def layout(node):
    if node.is_leaf():
        # 显示序列
        seq_face = SeqMotifFace(node.sequence, seq_format="seq")
        node.add_face(seq_face, column=0, position="aligned")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = True

tree.render("tree_alignment.pdf", tree_style=ts)
```

---

## 布局函数

布局函数是在渲染过程中修改节点外观的Python函数。

### 基础布局函数

```python
from ete3 import Tree, TreeStyle, TextFace

tree = Tree("tree.nw")

def my_layout(node):
    """在渲染前为每个节点调用"""

    if node.is_leaf():
        # 为叶节点添加文本
        name_face = TextFace(node.name.upper(), fsize=12, fgcolor="blue")
        node.add_face(name_face, column=0, position="branch-right")
    else:
        # 为内部节点添加支持值
        if node.support:
            support_face = TextFace(f"BS: {node.support:.0f}", fsize=8)
            node.add_face(support_face, column=0, position="branch-top")

# 应用布局函数
ts = TreeStyle()
ts.layout_fn = my_layout
ts.show_leaf_name = False

tree.render("tree_custom_layout.pdf", tree_style=ts)
```

### 动态样式布局

```python
from ete3 import Tree, TreeStyle, NodeStyle, TextFace

tree = Tree("tree.nw")

def layout(node):
    # 动态修改节点样式
    nstyle = NodeStyle()

    # 按分支着色
    if "clade_A" in [l.name for l in node.get_leaves()]:
        nstyle["bgcolor"] = "lightblue"
    elif "clade_B" in [l.name for l in node.get_leaves()]:
        nstyle["bgcolor"] = "lightgreen"

    node.set_style(nstyle)

    # 基于特征添加元素
    if hasattr(node, "annotation"):
        text = TextFace(node.annotation, fsize=8)
        node.add_face(text, column=0, position="branch-top")

ts = TreeStyle()
ts.layout_fn = layout

tree.render("tree_dynamic.pdf", tree_style=ts)
```

### 多列布局

```python
from ete3 import Tree, TreeStyle, TextFace, CircleFace

tree = Tree("tree.nw")

# 添加特征
for leaf in tree:
    leaf.add_feature("habitat", "aquatic")
    leaf.add_feature("temp", 20)
    leaf.add_feature("depth", 100)

def layout(node):
    if node.is_leaf():
        # 列0: 名称
        name = TextFace(node.name, fsize=10)
        node.add_face(name, column=0, position="aligned")

        # 列1: 栖息地指示器
        color = "blue" if node.habitat == "aquatic" else "brown"
        circle = CircleFace(radius=5, color=color)
        node.add_face(circle, column=1, position="aligned")

        # 列2: 温度
        temp = TextFace(f"{node.temp}°C", fsize=8)

tree.render("tree_columns.pdf", tree_style=ts)
```

---

## 高级可视化

### 高亮显示分支

```python
from ete3 import Tree, TreeStyle, NodeStyle, TextFace

tree = Tree("tree.nw")

# 定义需要高亮的分支
clade_members = {
    "分支_A": ["物种1", "物种2", "物种3"],
    "分支_B": ["物种4", "物种5"]
}

def layout(node):
    # 检查节点是否为特定分支的祖先
    node_leaves = set([l.name for l in node.get_leaves()])

    for clade_name, members in clade_members.items():
        if set(members).issubset(node_leaves):
            # 该节点是分支的祖先
            nstyle = NodeStyle()
            nstyle["bgcolor"] = "yellow"
            nstyle["size"] = 0

            # 添加标签
            if set(members) == node_leaves:  # 精确匹配
                label = TextFace(clade_name, fsize=14, bold=True, fgcolor="red")
                node.add_face(label, column=0, position="branch-top")

            node.set_style(nstyle)
            break

ts = TreeStyle()
ts.layout_fn = layout

tree.render("tree_highlighted_clades.pdf", tree_style=ts)
```

### 折叠分支

```python
from ete3 import Tree, TreeStyle, TextFace, NodeStyle

tree = Tree("tree.nw")

# 定义需要折叠的分支
clades_to_collapse = ["分支1_物种1", "分支1_物种2"]

def layout(node):
    if not node.is_leaf():
        node_leaves = [l.name for l in node.get_leaves()]

        # 检查是否为需要折叠的分支
        if all(l in clades_to_collapse for l in node_leaves):
            # 通过隐藏后代实现折叠
            nstyle = NodeStyle()
            nstyle["draw_descendants"] = False
            nstyle["size"] = 20
            nstyle["fgcolor"] = "steelblue"
            nstyle["shape"] = "sphere"
            node.set_style(nstyle)

            # 添加折叠标签
            label = TextFace(f"[{len(node_leaves)} 物种]", fsize=10)
            node.add_face(label, column=0, position="branch-right")

ts = TreeStyle()
ts.layout_fn = layout

tree.render("tree_collapsed.pdf", tree_style=ts)
```

### 热图可视化

```python
from ete3 import Tree, TreeStyle, RectFace, TextFace
import numpy as np

tree = Tree("tree.nw")

# 为热图生成随机数据
for leaf in tree:
    leaf.add_feature("data", np.random.rand(10))  # 10个数据点

def layout(node):
    if node.is_leaf():
        # 添加名称
        name = TextFace(node.name, fsize=8)
        node.add_face(name, column=0, position="aligned")

        # 添加热图单元格
        for i, value in enumerate(node.data):
            # 基于数值设置颜色
            intensity = int(255 * value)
            color = f"#{255-intensity:02x}{intensity:02x}00"  # 红绿渐变

            rect = RectFace(width=20, height=15, fgcolor=color, bgcolor=color)
            node.add_face(rect, column=i+1, position="aligned")

# 添加列标题
ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = False

# 添加表头
for i in range(10):
    header = TextFace(f"C{i+1}", fsize=8, fgcolor="gray")
    ts.aligned_header.add_face(header, column=i+1)

tree.render("tree_heatmap.pdf", tree_style=ts)
```

### 系统发育事件可视化

```python
from ete3 import PhyloTree, TreeStyle, TextFace, NodeStyle

tree = PhyloTree("gene_tree.nw")
tree.set_species_naming_function(lambda x: x.split("_")[0])
tree.get_descendant_evol_events()

def layout(node):
    # 根据进化事件设置样式
    if hasattr(node, "evoltype"):
        nstyle = NodeStyle()

        if node.evoltype == "D":  # 复制事件
            nstyle["fgcolor"] = "red"
            nstyle["size"] = 10
            nstyle["shape"] = "square"

            label = TextFace("复制", fsize=8, fgcolor="red", bold=True)
            node.add_face(label, column=0, position="branch-top")

        elif node.evoltype == "S":  # 物种形成
            nstyle["fgcolor"] = "blue"
            nstyle["size"] = 6
            nstyle["shape"] = "circle"

        node.set_style(nstyle)

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = True

tree.render("gene_tree_events.pdf", tree_style=ts)
```

### 带图例的自定义树

```python
from ete3 import Tree, TreeStyle, TextFace, CircleFace, NodeStyle

tree = Tree("tree.nw")

# 物种分类
for leaf in tree:
    if "fish" in leaf.name.lower():
        leaf.add_feature("category", "fish")
    elif "bird" in leaf.name.lower():
        leaf.add_feature("category", "bird")
    else:
        leaf.add_feature("category", "mammal")

category_colors = {
    "fish": "blue",
    "bird": "green",
    "mammal": "red"
}

def layout(node):
    if node.is_leaf():
        # 按类别着色
        nstyle = NodeStyle()
        nstyle["fgcolor"] = category_colors[node.category]
        nstyle["size"] = 10
        node.set_style(nstyle)

ts = TreeStyle()
ts.layout_fn = layout

# 添加图例
ts.legend.add_face(TextFace("图例:", fsize=12, bold=True), column=0)
for category, color in category_colors.items():
    circle = CircleFace(radius=5, color=color)
    ts.legend.add_face(circle, column=0)
    label = TextFace(f" {category.capitalize()}", fsize=10)
    ts.legend.add_face(label, column=1)

ts.legend_position = 1

tree.render("tree_with_legend.pdf", tree_style=ts)
```

---

## 最佳实践

1. **使用布局函数**处理复杂可视化 - 它们在渲染时被调用
2. **设置 `show_leaf_name = False`** 当使用自定义名称标签时
3. **使用对齐位置**处理叶级列数据
4. **选择合适单位**: 屏幕用像素，打印用毫米/英寸
5. **使用矢量格式 (PDF/SVG)** 用于出版物
6. **尽可能预计算样式** - 布局函数应保持高效
7. **通过 `show()` 交互测试**后再渲染到文件
8. **NodeStyle用于永久修改**，布局函数用于渲染时修改
9. **按列对齐标签**确保整洁有序
10. **添加图例**说明颜色和符号含义
