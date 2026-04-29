---
name: etetoolkit
description: 系统发育树工具包（ETE）。支持树操作（Newick/NHX）、进化事件检测、直系同源/旁系同源分析、NCBI分类学、可视化（PDF/SVG），用于系统基因组学研究。
license: GPL-3.0许可证
metadata:
    skill-author: K-Dense Inc.
---

# ETE工具包技能

## 概述

ETE（Environment for Tree Exploration）是用于系统发育和层次树分析的工具包。可操作进化树、分析进化事件、可视化结果，并与生物数据库集成，支持系统基因组学研究和聚类分析。

## 核心功能

### 1. 树操作与分析

加载、操作和分析层次树结构，支持：

- **树输入/输出**：读写Newick、NHX、PhyloXML和NeXML格式
- **树遍历**：使用前序、后序或层级遍历策略导航树结构
- **拓扑修改**：修剪节点、重定根节点、折叠节点、解决多歧分支
- **距离计算**：计算节点间分支长度和拓扑距离
- **树比较**：计算Robinson-Foulds距离并识别拓扑差异

**常用模式：**

```python
from ete3 import Tree

# 从文件加载树
tree = Tree("tree.nw", format=1)

# 基础统计
print(f"叶子节点数: {len(tree)}")
print(f"总节点数: {len(list(tree.traverse()))}")

# 保留目标分类群
taxa_to_keep = ["species1", "species2", "species3"]
tree.prune(taxa_to_keep, preserve_branch_length=True)

# 中点定根
midpoint = tree.get_midpoint_outgroup()
tree.set_outgroup(midpoint)

# 保存修改后的树
tree.write(outfile="rooted_tree.nw")
```

使用`scripts/tree_operations.py`进行命令行树操作：

```bash
# 显示树统计信息
python scripts/tree_operations.py stats tree.nw

# 转换格式
python scripts/tree_operations.py convert tree.nw output.nw --in-format 0 --out-format 1

# 重定根节点
python scripts/tree_operations.py reroot tree.nw rooted.nw --midpoint

# 修剪特定分类群
python scripts/tree_operations.py prune tree.nw pruned.nw --keep-taxa "sp1,sp2,sp3"

# 显示ASCII可视化
python scripts/tree_operations.py ascii tree.nw
```

### 2. 系统发育分析

通过进化事件检测分析基因树：

- **序列比对集成**：将树与多序列比对（FASTA, Phylip）关联
- **物种命名**：从基因名自动或自定义提取物种信息
- **进化事件**：使用物种重叠法或树调和检测复制和分化事件
- **直系同源检测**：基于进化事件识别直系同源和旁系同源基因
- **基因家族分析**：通过复制事件分割树，折叠谱系特异性扩张

**基因树分析工作流：**

```python
from ete3 import PhyloTree

# 加载带比对的基因树
tree = PhyloTree("gene_tree.nw", alignment="alignment.fasta")

# 设置物种命名函数
def get_species(gene_name):
    return gene_name.split("_")[0]

tree.set_species_naming_function(get_species)

# 检测进化事件
events = tree.get_descendant_evol_events()

# 分析事件
for node in tree.traverse():
    if hasattr(node, "evoltype"):
        if node.evoltype == "D":
            print(f"复制事件发生于 {node.name}")
        elif node.evoltype == "S":
            print(f"分化事件发生于 {node.name}")

# 提取直系同源群组
ortho_groups = tree.get_speciation_trees()
for i, ortho_tree in enumerate(ortho_groups):
    ortho_tree.write(outfile=f"ortholog_group_{i}.nw")
```

**查找直系同源和旁系同源基因：**

```python
# 查找查询基因的直系同源
query = tree & "species1_gene1"

orthologs = []
paralogs = []

for event in events:
    if query in event.in_seqs:
        if event.etype == "S":
            orthologs.extend([s for s in event.out_seqs if s != query])
        elif event.etype == "D":
            paralogs.extend([s for s in event.out_seqs if s != query])
```

### 3. NCBI分类学集成

整合NCBI分类学数据库的物种信息：

- **数据库访问**：自动下载并本地缓存NCBI分类学数据（约300MB）
- **Taxid/名称转换**：在分类ID和学名间转换
- **谱系检索**：获取完整进化谱系
- **分类学树构建**：连接指定分类群构建物种树
- **树注释**：自动用分类学信息注释树结构

**构建基于分类学的树：**

```python
from ete3 import NCBITaxa

ncbi = NCBITaxa()

# 从物种名构建树
species = ["Homo sapiens", "Pan troglodytes", "Mus musculus"]
name2taxid = ncbi.get_name_translator(species)
taxids = [name2taxid[sp][0] for sp in species]

# 获取连接分类群的最小树
tree = ncbi.get_topology(taxids)

# 用分类学信息注释节点
for node in tree.traverse():
    if hasattr(node, "sci_name"):
        print(f"{node.sci_name} - 分类等级: {node.rank} - TaxID: {node.taxid}")
```

**注释现有树：**

```python
# 为叶子节点获取分类学信息
for leaf in tree:
    species = extract_species_from_name(leaf.name)
    taxid = ncbi.get_name_translator([species])[species][0]

    # 获取谱系
    lineage = ncbi.get_lineage(taxid)
    ranks = ncbi.get_rank(lineage)
    names = ncbi.get_taxid_translator(lineage)

    # 添加到节点
    leaf.add_feature("taxid", taxid)
    leaf.add_feature("lineage", [names[t] for t in lineage])
```

### 4. 树可视化

创建出版级质量的树可视化：

- **输出格式**：支持PNG（位图）、PDF和SVG（矢量）格式
- **布局模式**：矩形和圆形树布局
- **交互式GUI**：通过缩放、平移和搜索交互式探索树
- **自定义样式**：NodeStyle控制节点外观（颜色、形状、大小）
- **Faces功能**：添加图形元素（文本、图像、图表、热图）到节点
- **布局函数**：基于节点属性的动态样式配置

**基础可视化工作流：**

```python
from ete3 import Tree, TreeStyle, NodeStyle

tree = Tree("tree.nw")

# 配置树样式
ts = TreeStyle()
ts.show_leaf_name = True
ts.show_branch_support = True
ts.scale = 50  # 每分支长度单位像素数

# 样式化节点
for node in tree.traverse():
    nstyle = NodeStyle()

    if node.is_leaf():
        nstyle["fgcolor"] = "blue"
        nstyle["size"] = 8
    else:
        # 按支持度着色
        if node.support > 0.9:
            nstyle["fgcolor"] = "darkgreen"
        else:
            nstyle["fgcolor"] = "red"
        nstyle["size"] = 5

    node.set_style(nstyle)

# 渲染到文件
tree.render("tree.pdf", tree_style=ts)
tree.render("tree.png", w=800, h=600, units="px", dpi=300)
```

使用`scripts/quick_visualize.py`快速可视化：

```bash
# 基础可视化
python scripts/quick_visualize.py tree.nw output.pdf

# 圆形布局自定义样式
python scripts/quick_visualize.py tree.nw output.pdf --mode c --color-by-support

# 高分辨率PNG
python scripts/quick_visualize.py tree.nw output.png --width 1200 --height 800 --units px --dpi 300

# 自定义标题和样式
python scripts/quick_visualize.py tree.nw output.pdf --title "物种系统发育" --show-support
```

**使用Faces的高级可视化：**

```python
from ete3 import Tree, TreeStyle, TextFace, CircleFace

tree = Tree("tree.nw")

# 为节点添加特征
for leaf in tree:
    leaf.add_feature("habitat", "marine" if "fish" in leaf.name else "land")

# 布局函数
def layout(node):
    if node.is_leaf():
        # 添加彩色圆形
        color = "blue" if node.habitat == "marine" else "green"
        circle = CircleFace(radius=5, color=color)
        node.add_face(circle, column=0, position="aligned")

        # 添加标签
        label = TextFace(node.name, fsize=10)
        node.add_face(label, column=1, position="aligned")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = False

tree.render("annotated_tree.pdf", tree_style=ts)
```

### 5. 聚类分析

通过数据集成分析层次聚类结果：

- **ClusterTree**：专用于聚类树状图的类
- **数据矩阵链接**：将树叶子节点连接至数值谱
- **聚类指标**：轮廓系数、邓恩指数、簇间/簇内距离
- **验证**：使用不同距离度量测试聚类质量
- **热图可视化**：在树旁展示数据矩阵

**聚类工作流：**

```python
from ete3 import ClusterTree

# 加载带数据矩阵的树
matrix = """#Names\tSample1\tSample2\tSample3
Gene1\t1.5\t2.3\t0.8
Gene2\t0.9\t1.1\t1.8
Gene3\t2.1\t2.5\t0.5"""

tree = ClusterTree("((Gene1,Gene2),Gene3);", text_array=matrix)

# 评估聚类质量
for node in tree.traverse():
    if not node.is_leaf():
        silhouette = node.get_silhouette()
        dunn = node.get_dunn()

        print(f"聚类: {node.name}")
        print(f"  轮廓系数: {silhouette:.3f}")
        print(f"  邓恩指数: {dunn:.3f}")

# 带热图可视化
tree.show("heatmap")
```

### 6. 树比较

量化树间拓扑差异：

- **Robinson-Foulds距离**：标准树比较指标
- **标准化RF**：尺度不变距离（0.0至1.0）
- **分区分析**：识别独特和共享二分体
- **共识树**：分析多棵树的支持度
- **批量比较**：多棵树两两比较

**比较两棵树：**

```python
from ete3 import Tree

tree1 = Tree("tree1.nw")
tree2 = Tree("tree2.nw")

# 计算RF距离
rf, max_rf, common_leaves, parts_t1, parts_t2 = tree1.robinson_foulds(tree2)

print(f"RF距离: {rf}/{max_rf}")
print(f"标准化RF: {rf/max_rf:.3f}")
print(f"共有叶子节点: {len(common_leaves)}")

# 查找独特分区
unique_t1 = parts_t1 - parts_t2
unique_t2 = parts_t2 - parts_t1

print(f"tree1独有: {len(unique_t1)}")
print(f"tree2独有: {len(unique_t2)}")
```

**比较多棵树：**

```python
import numpy as np

trees = [Tree(f"tree{i}.nw") for i in range(4)]

# 创建距离矩阵
n = len(trees)
dist_matrix = np.zeros((n, n))

for i in range(n):
    for j in range(i+1, n):
        rf, max_rf, _, _, _ = trees[i].robinson_foulds(trees[j])
        norm_rf = rf / max_rf if max_rf > 0 else 0
        dist_matrix[i, j] = norm_rf
        dist_matrix[j, i] = norm_rf
```

## 安装与配置

安装ETE工具包：

```bash
# 基础安装
uv pip install ete3

# 带渲染外部依赖（推荐）
# macOS系统:
brew install qt@5

# Ubuntu/Debian系统:
sudo apt-get install python3-pyqt5 python3-pyqt5.qtsvg

# 完整功能（含GUI）
uv pip install ete3[gui]
```

**首次NCBI分类学配置：**

首次实例化NCBITaxa时，自动下载NCBI分类学数据库（约300MB）至`~/.etetoolkit/taxa.sqlite`，仅发生一次：

```python
from ete3 import NCBITaxa
ncbi = NCBITaxa()  # 首次运行时下载数据库
```

更新分类学数据库：

```python
ncbi.update_taxonomy_database()  # 下载最新NCBI数据
```

## 典型用例

### 用例1：系统基因组学流程

从基因树到直系同源识别的完整工作流：

```python
from ete3 import PhyloTree, NCBITaxa

# 1. 加载带比对的基因树
tree = PhyloTree("gene_tree.nw", alignment="alignment.fasta")

# 2. 配置物种命名
tree.set_species_naming_function(lambda x: x.split("_")[0])

# 3. 检测进化事件
tree.get_descendant_evol_events()

# 4. 分类学注释
ncbi = NCBITaxa()
for leaf in tree:
    if leaf.species in species_to_taxid:
        taxid = species_to_taxid[leaf.species]
        lineage = ncbi.get_lineage(taxid)
        leaf.add_feature("lineage", lineage)

# 5. 提取直系同源群组
ortho_groups = tree.get_speciation_trees()

# 6. 保存与可视化
for i, ortho in enumerate(ortho_groups):
    ortho.write(outfile=f"ortho_{i}.nw")
```

### 用例2：树预处理与格式化

批量处理树进行分析：

```bash
# 格式转换
python scripts/tree_operations.py convert input.nw output.nw --in-format 0 --out-format 1

# 中点定根
python scripts/tree_operations.py reroot input.nw rooted.nw --midpoint

# 修剪至目标分类群
python scripts/tree_operations.py prune rooted.nw pruned.nw --keep-taxa taxa_list.txt

# 获取统计信息
python scripts/tree_operations.py stats pruned.nw
```

### 用例3：出版级图表

创建样式化可视化：

```python
from ete3 import Tree, TreeStyle, NodeStyle, TextFace

tree = Tree("tree.nw")

# 定义进化枝颜色
clade_colors = {
    "Mammals": "red",
    "Birds": "blue",
    "Fish": "green"
}

def layout(node):
    # 高亮进化枝
    if node.is_leaf():
        for clade, color in clade_colors.items():
            if clade in node.name:
                nstyle = NodeStyle()
                nstyle["fgcolor"] = color
                nstyle["size"] = 8
                node.set_style(nstyle)
    else:
        # 添加支持度值
        if node.support > 0.95:
            support = TextFace(f"{node.support:.2f}", fsize=8)
            node.add_face(support, column=0, position="branch-top")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_scale = True

# 出版级渲染
tree.render("figure.pdf", w=200, units="mm", tree

如需全面的API文档、代码示例和详细指南，请查阅`references/`目录中的以下资源：

- **`api_reference.md`**：所有ETE类与方法的完整API文档（Tree、PhyloTree、ClusterTree、NCBITaxa），包含参数说明、返回类型及代码示例
- **`workflows.md`**：按任务分类的常用工作流模式（树操作、系统发育分析、树比较、分类学集成、聚类分析）
- **`visualization.md`**：完整可视化指南，涵盖TreeStyle、NodeStyle、Faces、布局函数及高级可视化技术

需要详细信息时加载这些参考文档：

```python
# 使用API参考
# 阅读 references/api_reference.md 获取完整方法签名和参数说明

# 实现工作流
# 阅读 references/workflows.md 获取分步工作流示例

# 创建可视化
# 阅读 references/visualization.md 获取样式配置和渲染选项
```

## 故障排除

**导入错误：**

```bash
# 若出现 "ModuleNotFoundError: No module named 'ete3'"
uv pip install ete3

# 解决GUI和渲染问题
uv pip install ete3[gui]
```

**渲染问题：**

若`tree.render()`或`tree.show()`因Qt相关错误失败，请安装系统依赖：

```bash
# macOS
brew install qt@5

# Ubuntu/Debian
sudo apt-get install python3-pyqt5 python3-pyqt5.qtsvg
```

**NCBI分类数据库：**

若数据库下载失败或损坏：

```python
from ete3 import NCBITaxa
ncbi = NCBITaxa()
ncbi.update_taxonomy_database()  # 重新下载数据库
```

**大型树内存问题：**

处理超大型树（>10,000叶子节点）时使用迭代器：

```python
# 内存高效迭代
for leaf in tree.iter_leaves():
    process(leaf)

# 避免使用
for leaf in tree.get_leaves():  # 会加载全部到内存
    process(leaf)
```

## Newick格式参考

ETE支持多种Newick格式规范（0-100）：

- **格式0**：灵活支持分支长度（默认）
- **格式1**：包含内部节点名称
- **格式2**：包含自举/支持值
- **格式5**：内部节点名称+分支长度
- **格式8**：全特性（名称、距离、支持值）
- **格式9**：仅叶子节点名称
- **格式100**：仅拓扑结构

读写时指定格式：

```python
tree = Tree("tree.nw", format=1)
tree.write(outfile="output.nw", format=5)
```

NHX（New Hampshire扩展）格式保留自定义特性：

```python
tree.write(outfile="tree.nhx", features=["habitat", "temperature", "depth"])
```

## 最佳实践

1. **保留分支长度**：系统发育分析剪枝时使用`preserve_branch_length=True`
2. **缓存内容**：大型树重复访问节点内容时使用`get_cached_content()`
3. **使用迭代器**：大型树处理采用`iter_*`方法实现内存高效操作
4. **选择遍历方式**：自底向上分析用后序，自顶向下用前序
5. **验证单系性**：始终检查返回的进化支类型（单系/并系/多系）
6. **出版级矢量图**：发表用图选择PDF或SVG格式（可缩放/可编辑）
7. **交互测试**：文件渲染前用`tree.show()`测试可视化效果
8. **系统发育专用**：基因树和进化分析使用PhyloTree类
9. **复制方法选择**：速度选"newick"，保真选"cpickle"，复杂对象选"deepcopy"
10. **NCBI查询缓存**：存储分类查询结果避免重复访问数据库
