# ETE 工具包常用工作流程

本文档提供了使用 ETE 工具包执行常见任务的完整工作流程。

## 目录
1. [基础树操作](#basic-tree-operations)
2. [系统发育分析](#phylogenetic-analysis)
3. [树比较](#tree-comparison)
4. [分类学整合](#taxonomy-integration)
5. [聚类分析](#clustering-analysis)
6. [树可视化](#tree-visualization)

---

## 基础树操作

### 加载与探索树结构

```python
from ete3 import Tree

# 从文件加载树
tree = Tree("my_tree.nw", format=1)

# 显示ASCII表示
print(tree.get_ascii(show_internal=True))

# 获取基础统计信息
print(f"叶子节点数: {len(tree)}")
print(f"总节点数: {len(list(tree.traverse()))}")
print(f"树深度: {tree.get_farthest_leaf()[1]}")

# 列出所有叶子名称
for leaf in tree:
    print(leaf.name)
```

### 提取与保存子树

```python
from ete3 import Tree

tree = Tree("full_tree.nw")

# 获取特定节点为根的子树
node = tree.search_nodes(name="MyNode")[0]
subtree = node.copy()

# 保存子树到文件
subtree.write(outfile="subtree.nw", format=1)

# 提取单系分支
species_of_interest = ["species1", "species2", "species3"]
ancestor = tree.get_common_ancestor(species_of_interest)
clade = ancestor.copy()
clade.write(outfile="clade.nw")
```

### 修剪树至特定分类单元

```python
from ete3 import Tree

tree = Tree("large_tree.nw")

# 仅保留目标分类单元
taxa_to_keep = ["taxon1", "taxon2", "taxon3", "taxon4"]
tree.prune(taxa_to_keep, preserve_branch_length=True)

# 保存修剪后的树
tree.write(outfile="pruned_tree.nw")
```

### 树的重根操作

```python
from ete3 import Tree

tree = Tree("unrooted_tree.nw")

# 方法1：通过外群定根
outgroup = tree & "Outgroup_species"
tree.set_outgroup(outgroup)

# 方法2：中点定根
midpoint = tree.get_midpoint_outgroup()
tree.set_outgroup(midpoint)

# 保存定根后的树
tree.write(outfile="rooted_tree.nw")
```

### 使用自定义数据注释节点

```python
from ete3 import Tree

tree = Tree("tree.nw")

# 基于元数据添加节点特征
metadata = {
    "species1": {"habitat": "marine", "temperature": 20},
    "species2": {"habitat": "freshwater", "temperature": 15},
}

for leaf in tree:
    if leaf.name in metadata:
        leaf.add_features(**metadata[leaf.name])

# 查询注释特征
for leaf in tree:
    if hasattr(leaf, "habitat"):
        print(f"{leaf.name}: {leaf.habitat}, {leaf.temperature}°C")

# 保存带自定义特征的树(NHX格式)
tree.write(outfile="annotated_tree.nhx", features=["habitat", "temperature"])
```

### 修改树拓扑结构

```python
from ete3 import Tree

tree = Tree("tree.nw")

# 移除分支
node_to_remove = tree & "unwanted_clade"
node_to_remove.detach()

# 折叠节点（删除节点但保留子节点）
node_to_collapse = tree & "low_support_node"
node_to_collapse.delete()

# 向现有分支添加新物种
target_clade = tree & "target_node"
new_leaf = target_clade.add_child(name="new_species", dist=0.5)

# 解决多歧分支
tree.resolve_polytomy(recursive=True)

# 保存修改后的树
tree.write(outfile="modified_tree.nw")
```

---

## 系统发育分析

### 包含比对的完整基因树分析

```python
from ete3 import PhyloTree

# 加载基因树并关联比对序列
tree = PhyloTree("gene_tree.nw", format=1)
tree.link_to_alignment("alignment.fasta", alg_format="fasta")

# 设置物种命名函数（例如gene_species格式）
def extract_species(node_name):
    return node_name.split("_")[0]

tree.set_species_naming_function(extract_species)

# 访问序列
for leaf in tree:
    print(f"{leaf.name} ({leaf.species})")
    print(f"序列: {leaf.sequence[:50]}...")
```

### 检测复制与分化事件

```python
from ete3 import PhyloTree, Tree

# 加载基因树
gene_tree = PhyloTree("gene_tree.nw")

# 设置物种命名
gene_tree.set_species_naming_function(lambda x: x.split("_")[0])

# 选项1：物种重叠算法（无需物种树）
events = gene_tree.get_descendant_evol_events()

# 选项2：树协调（需要物种树）
species_tree = Tree("species_tree.nw")
events = gene_tree.get_descendant_evol_events(species_tree=species_tree)

# 分析事件
duplications = 0
speciations = 0

for node in gene_tree.traverse():
    if hasattr(node, "evoltype"):
        if node.evoltype == "D":
            duplications += 1
            print(f"节点 {node.name} 发生复制事件")
        elif node.evoltype == "S":
            speciations += 1

print(f"\n总复制事件: {duplications}")
print(f"总分化事件: {speciations}")
```

### 提取直系同源与旁系同源基因

```python
from ete3 import PhyloTree

gene_tree = PhyloTree("gene_tree.nw")
gene_tree.set_species_naming_function(lambda x: x.split("_")[0])

# 检测进化事件
events = gene_tree.get_descendant_evol_events()

# 查找查询基因的所有直系同源
query_gene = gene_tree & "species1_gene1"

orthologs = []
paralogs = []

for event in events:
    if query_gene in event.in_seqs:
        if event.etype == "S":  # 分化事件
            orthologs.extend([s for s in event.out_seqs if s != query_gene])
        elif event.etype == "D":  # 复制事件
            paralogs.extend([s for s in event.out_seqs if s != query_gene])

print(f"{query_gene.name} 的直系同源基因:")
for ortholog in set(orthologs):
    print(f"  {ortholog.name}")

print(f"\n{query_gene.name} 的旁系同源基因:")
for paralog in set(paralogs):
    print(f"  {paralog.name}")
```

### 按复制事件分割基因家族

```python
from ete3 import PhyloTree

gene_tree = PhyloTree("gene_family.nw")
gene_tree.set_species_naming_function(lambda x: x.split("_")[0])
gene_tree.get_descendant_evol_events()

# 分割为独立基因家族
subfamilies = gene_tree.split_by_dups()

print(f"基因家族被分割为 {len(subfamilies)} 个子家族")

for i, subtree in enumerate(subfamilies):
    subtree.write(outfile=f"subfamily_{i}.nw")
    species = set([leaf.species for leaf in subtree])
    print(f"子家族 {i}: 包含 {len(subtree)} 个基因，来自 {len(species)} 个物种")
```

### 折叠谱系特异性扩张

```python
from ete3 import PhyloTree

gene_tree = PhyloTree("expanded_tree.nw")
gene_tree.set_species_naming_function(lambda x: x.split("_")[0])

# 折叠谱系特异性复制
gene_tree.collapse_lineage_specific_expansions()

print("折叠扩张后:")
print(gene_tree.get_ascii())

gene_tree.write(outfile="collapsed_tree.nw")
```

### 检验单系性

```python
from ete3 import Tree

tree = Tree("tree.nw")

# 检验目标类群是否单系
target_species = ["species1", "species2", "species3"]
is_mono, clade_type, base_node = tree.check_monophyly(
    values=target_species,
    target_attr="name"
)

if is_mono:
    print(f"该组为单系群")
    print(f"最近共同祖先: {base_node.name}")
elif clade_type == "paraphyletic":
    print(f"该组为并系群")
elif clade_type == "polyphyletic":
    print(f"该组为多系群")

# 获取特定类型的所有单系分支
# 先注释叶子节点
for leaf in tree:
    if leaf.name.startswith("species"):
        leaf.add_feature("type", "typeA")
    else:
        leaf.add_feature("type", "typeB")

mono_clades = tree.get_monophyletic(values=["typeA"], target_attr="type")
print(f"找到 {len(mono_clades)} 个 typeA 单系分支")
```

---

## 树比较

### 计算Robinson-Foulds距离

```python
from ete3 import Tree

tree1 = Tree("tree1.nw")
tree2 = Tree("tree2.nw")

# 计算RF距离
rf, max_rf, common_leaves, parts_t1, parts_t2 = tree1.robinson_foulds(tree2)

print(f"Robinson-Foulds距离: {rf}")
print(f"最大RF距离: {max_rf}")
print(f"标准化RF: {rf/max_rf:.3f}")
print(f"共有叶子节点: {len(common_leaves)}")

# 查找独特分区
unique_in_t1 = parts_t1 - parts_t2
unique_in_t2 = parts_t2 - parts_t1

print(f"\n树1独有分区: {len(unique_in_t1)}")
print(f"树2独有分区: {len(unique_in_t2)}")
```

### 比较多棵树

```python
from ete3 import Tree
import numpy as np

# 加载多棵树
tree_files = ["tree1.nw", "tree2.nw", "tree3.nw", "tree4.nw"]
trees = [Tree(f) for f in tree_files]

# 创建距离矩阵
n = len(trees)
dist_matrix = np.zeros((n, n))

for i in range(n):
    for j in range(i+1, n):
        rf, max_rf, _, _, _ = trees[i].robinson_foulds(trees[j])
        norm_rf = rf / max_rf if max_rf > 0 else 0
        dist_matrix[i, j] = norm_rf
        dist_matrix[j, i] = norm_rf

print("标准化RF距离矩阵:")
print(dist_matrix)

# 查找最相似树对
min_dist = float('inf')
best_pair = None

for i in range(n):
    for j in range(i+1, n):
        if dist_matrix[i, j] < min_dist:
            min_dist = dist_matrix[i, j]
            best_pair = (i, j)

print(f"\n最相似树: {tree_files[best_pair[0]]} 和 {tree_files[best_pair[1]]}")
print(f"距离: {min_dist:.3f}")
```

### 寻找一致拓扑结构

```python
from ete3 import Tree

# 加载多棵bootstrap树
bootstrap_trees = [Tree(f"bootstrap_{i}.nw") for i in range(100)]

# 获取参考树（第一棵树）
ref_tree = bootstrap_trees[0].copy()

# 统计二分频次
bipartition_counts = {}

for tree in bootstrap_trees:
    rf, max_rf, common, parts_ref, parts_tree = ref_tree.robinson_foulds(tree)
    for partition in parts_tree:
        bipartition_counts[partition] = bipartition_counts.get(partition, 0) + 1

# 按支持度阈值过滤
threshold = 70  # 70%支持度
supported_bipartitions = {
    k: v for k, v in bipartition_counts.items()
    if (v / len(bootstrap_trees)) * 100 >= threshold
}

print(f"支持度>{threshold}%的二分结构: {len(supported_bipartitions)}")
```

---

## 分类学整合

### 基于NCBI分类学构建物种树

```python
from ete3 import NCBITaxa

ncbi = NCBITaxa()

# 定义目标物种
species = ["Homo sapiens", "Pan troglodytes", "Gorilla gorilla",
           "Mus musculus", "Rattus norvegicus"]

# 获取分类ID
name2taxid = ncbi.get_name_translator(species)
taxids = [name2taxid[sp][0] for sp in species]

# 构建树
tree = ncbi.get_topology(taxids)

# 添加分类学注释
for node in tree.traverse():
    if hasattr(node, "sci_name"):
        print(f"{node.sci_name} - 分类等级: {node.rank} - 分类ID: {node.taxid}")

# 保存树
tree.write(outfile="species_tree.nw")
```

### 使用NCBI分类学注释现有树

```python
from ete3 import Tree, NCBITaxa

tree = Tree("species_tree.nw")
ncbi = NCBITaxa()

# 映射叶子名称到物种名（按需调整）
leaf_to_species = {
    "Hsap_gene1": "Homo sapiens",
    "Ptro_gene1": "Pan troglodytes",
    "Mmur_gene1": "Microcebus murinus",
}

# 获取分类ID
all_species = list(set(leaf_to_species.values()))
name2taxid = ncbi.get_name_translator(all_species)

# 注释叶子节点
for leaf in tree:
    if leaf.name in leaf_to_species:
        species_name = leaf_to_species[leaf.name]
        taxid = name2taxid[species_name][0]

        # 添加分类学信息
        leaf.add_feature("species", species_name)
        leaf.add_feature("taxid", taxid)

        # 获取完整谱系
        lineage = ncbi.get_lineage(taxid)
        names = ncbi.get_taxid_translator(lineage)
        leaf.add_feature("lineage", [names[t] for t in lineage])

        print(f"{leaf.name}: {species_name} (分类ID: {taxid})")
```

### 查询NCBI分类学数据库

```python
from ete3 import NCBITaxa

ncbi = NCBITaxa()

# 获取所有灵长类
primates_taxid = ncbi.get_name_translator(["Primates"])["Primates"][0]
all_primates = ncbi.get_descendant_taxa(primates_taxid, collapse_subspecies=True)

print(f"灵长类物种总数: {len(all_primates)}")

# 获取子集名称
taxid2name = ncbi.get_taxid_translator(all_primates[:10])
for taxid, name in taxid2name.items():
    rank = ncbi.get_rank([taxid])[taxid]
    print(f"{name} ({rank})")

# 获取特定物种谱系
human_taxid = 9606
lineage = ncbi.get_lineage(human_taxid)
ranks = ncbi.get_rank(lineage)
names = ncbi.get_taxid_translator(lineage)

print("\n人类谱系:")
for taxid in lineage:
    print(f"{ranks[taxid]:15s} {names[taxid]}")
```

---

## 聚类分析

### 分析层次聚类结果

```python
from ete3 import ClusterTree

# 加载带数据矩阵的聚类树
matrix = """#Names\tSample1\tSample2\tSample3\tSample4
Gene1\t1.5\t2.3\t0.8\t1.2
Gene2\t0.9\t1.1\t1.8\t2.1
Gene3\t2.1\t2.5\t0.5\t0.9
Gene4\t0.

```python
print(f"  轮廓系数: {silhouette:.3f}")
        print(f"  邓恩指数: {dunn:.3f}")
        print(f"  簇间距离: {inter:.3f}")
        print(f"  簇内距离: {intra:.3f}")
```

### 验证聚类结果

```python
from ete3 import ClusterTree

matrix = """#Names\tCol1\tCol2\tCol3
ItemA\t1.2\t0.5\t0.8
ItemB\t1.3\t0.6\t0.9
ItemC\t0.1\t2.5\t2.3
ItemD\t0.2\t2.6\t2.4"""

tree = ClusterTree("((ItemA,ItemB),(ItemC,ItemD));", text_array=matrix)

# 测试不同距离度量标准
metrics = ["euclidean", "pearson", "spearman"]

for metric in metrics:
    print(f"\n使用 {metric} 距离:")

    for node in tree.traverse():
        if not node.is_leaf():
            silhouette = node.get_silhouette(distance=metric)

            # 正轮廓系数 = 良好聚类
            # 负轮廓系数 = 较差聚类
            quality = "良好" if silhouette > 0 else "较差"

            print(f"  聚类 {node.name}: {silhouette:.3f} ({quality})")
```

---

## 树形可视化

### 基础树形渲染

```python
from ete3 import Tree, TreeStyle

tree = Tree("tree.nw")

# 创建树样式
ts = TreeStyle()
ts.show_leaf_name = True
ts.show_branch_length = True
ts.show_branch_support = True
ts.scale = 50  # 每单位分支长度的像素值

# 渲染到文件
tree.render("tree_output.pdf", tree_style=ts)
tree.render("tree_output.png", tree_style=ts, w=800, h=600, units="px")
tree.render("tree_output.svg", tree_style=ts)
```

### 自定义节点外观

```python
from ete3 import Tree, TreeStyle, NodeStyle

tree = Tree("tree.nw")

# 定义节点样式
for node in tree.traverse():
    nstyle = NodeStyle()

    if node.is_leaf():
        nstyle["fgcolor"] = "blue"
        nstyle["size"] = 10
    else:
        nstyle["fgcolor"] = "red"
        nstyle["size"] = 5

    if node.support > 0.9:
        nstyle["shape"] = "sphere"
    else:
        nstyle["shape"] = "circle"

    node.set_style(nstyle)

# 渲染
ts = TreeStyle()
tree.render("styled_tree.pdf", tree_style=ts)
```

### 添加节点标签

```python
from ete3 import Tree, TreeStyle, TextFace, CircleFace, AttrFace

tree = Tree("tree.nw")

# 为节点添加特征
for leaf in tree:
    leaf.add_feature("habitat", "marine" if "fish" in leaf.name else "terrestrial")
    leaf.add_feature("temp", 20)

# 布局函数添加标签
def layout(node):
    if node.is_leaf():
        # 添加文本标签
        name_face = TextFace(node.name, fsize=10)
        node.add_face(name_face, column=0, position="branch-right")

        # 根据栖息地添加彩色圆点
        color = "blue" if node.habitat == "marine" else "green"
        circle_face = CircleFace(radius=5, color=color)
        node.add_face(circle_face, column=1, position="branch-right")

        # 添加属性标签
        temp_face = AttrFace("temp", fsize=8)
        node.add_face(temp_face, column=2, position="branch-right")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = False  # 已添加自定义名称

tree.render("tree_with_faces.pdf", tree_style=ts)
```

### 环形树布局

```python
from ete3 import Tree, TreeStyle

tree = Tree("tree.nw")

ts = TreeStyle()
ts.mode = "c"  # 环形模式
ts.arc_start = 0  # 起始角度
ts.arc_span = 360  # 完整圆周
ts.show_leaf_name = True

tree.render("circular_tree.pdf", tree_style=ts)
```

### 交互式探索

```python
from ete3 import Tree

tree = Tree("tree.nw")

# 启动图形界面（支持缩放、搜索、修改）
# 关闭后修改仍会保留
tree.show()

# 可保存图形界面中的修改
tree.write(outfile="modified_tree.nw")
```

---

## 高级工作流

### 完整系统发育分析流程

```python
from ete3 import PhyloTree, NCBITaxa, TreeStyle

# 1. 加载基因树
gene_tree = PhyloTree("gene_tree.nw", alignment="alignment.fasta")

# 2. 设置物种命名规则
gene_tree.set_species_naming_function(lambda x: x.split("_")[0])

# 3. 检测进化事件
gene_tree.get_descendant_evol_events()

# 4. 使用NCBI分类法注释
ncbi = NCBITaxa()
species_set = set([leaf.species for leaf in gene_tree])
name2taxid = ncbi.get_name_translator(list(species_set))

for leaf in gene_tree:
    if leaf.species in name2taxid:
        taxid = name2taxid[leaf.species][0]
        lineage = ncbi.get_lineage(taxid)
        names = ncbi.get_taxid_translator(lineage)
        leaf.add_feature("lineage", [names[t] for t in lineage])

# 5. 识别并保存直系同源组
ortho_groups = gene_tree.get_speciation_trees()

for i, ortho_tree in enumerate(ortho_groups):
    ortho_tree.write(outfile=f"ortholog_group_{i}.nw")

# 6. 可视化标记进化事件
def layout(node):
    from ete3 import TextFace
    if hasattr(node, "evoltype"):
        if node.evoltype == "D":
            dup_face = TextFace("复制事件", fsize=8, fgcolor="red")
            node.add_face(dup_face, column=0, position="branch-top")

ts = TreeStyle()
ts.layout_fn = layout
ts.show_leaf_name = True
gene_tree.render("annotated_gene_tree.pdf", tree_style=ts)

print(f"流程完成。发现 {len(ortho_groups)} 个直系同源组。")
```

### 批量处理多棵树

```python
from ete3 import Tree
import os

input_dir = "input_trees"
output_dir = "processed_trees"
os.makedirs(output_dir, exist_ok=True)

for filename in os.listdir(input_dir):
    if filename.endswith(".nw"):
        # 加载树
        tree = Tree(os.path.join(input_dir, filename))

        # 处理：定根、修剪、注释
        midpoint = tree.get_midpoint_outgroup()
        tree.set_outgroup(midpoint)

        # 按分支长度过滤
        to_remove = []
        for node in tree.traverse():
            if node.dist < 0.001 and not node.is_root():
                to_remove.append(node)

        for node in to_remove:
            node.delete()

        # 保存处理后的树
        output_file = os.path.join(output_dir, f"processed_{filename}")
        tree.write(outfile=output_file)

        print(f"已处理 {filename}")
```
