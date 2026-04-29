# 使用 Bio.Phylo 进行系统发育分析

## 概述

Bio.Phylo 提供了一套统一的工具包，用于读取、写入、分析和可视化系统发育树。它支持多种文件格式，包括 Newick、NEXUS、phyloXML、NeXML 和 CDAO。

## 支持的文件格式

- **Newick** - 简洁的树表示（最常用）
- **NEXUS** - 包含附加数据的扩展格式
- **phyloXML** - 支持丰富注释的 XML 格式
- **NeXML** - 现代 XML 格式
- **CDAO** - 比较数据分析本体

## 读取与写入树结构

### 读取树结构

```python
from Bio import Phylo

# 从文件读取树
tree = Phylo.read("tree.nwk", "newick")

# 从文件解析多棵树
trees = list(Phylo.parse("trees.nwk", "newick"))
print(f"找到 {len(trees)} 棵树")
```

### 写入树结构

```python
# 将树写入文件
Phylo.write(tree, "output.nwk", "newick")

# 写入多棵树
Phylo.write(trees, "output.nex", "nexus")
```

### 格式转换

```python
# 格式间转换
count = Phylo.convert("input.nwk", "newick", "output.xml", "phyloxml")
print(f"已转换 {count} 棵树")
```

## 树结构与遍历

### 基本树组件

树包含：
- **分支（Clade）** - 树中的节点（内部或末端）
- **末端分支** - 叶子节点（分类单元）
- **内部分支** - 内部节点
- **分支长度** - 进化距离

### 访问树属性

```python
# 树根节点
root = tree.root

# 末端节点（叶子）
terminals = tree.get_terminals()
print(f"分类单元数量: {len(terminals)}")

# 非末端节点
nonterminals = tree.get_nonterminals()
print(f"内部节点数量: {len(nonterminals)}")

# 所有分支
all_clades = list(tree.find_clades())
print(f"总分枝数: {len(all_clades)}")
```

### 遍历树结构

```python
# 遍历所有分支
for clade in tree.find_clades():
    if clade.name:
        print(f"分支: {clade.name}, 分支长度: {clade.branch_length}")

# 仅遍历末端节点
for terminal in tree.get_terminals():
    print(f"分类单元: {terminal.name}")

# 深度优先遍历
for clade in tree.find_clades(order="preorder"):
    print(clade.name)

# 层级遍历（广度优先）
for clade in tree.find_clades(order="level"):
    print(clade.name)
```

### 查找特定分支

```python
# 按名称查找分支
clade = tree.find_any(name="Species_A")

# 查找符合条件的分支
def is_long_branch(clade):
    return clade.branch_length and clade.branch_length > 0.5

long_branches = tree.find_clades(is_long_branch)
```

## 树分析

### 树统计

```python
# 总分支长度
total_length = tree.total_branch_length()
print(f"树总长度: {total_length:.3f}")

# 树深度（根到最远叶节点）
depths = tree.depths()
max_depth = max(depths.values())
print(f"最大深度: {max_depth:.3f}")

# 末端节点计数
terminal_count = tree.count_terminals()
print(f"分类单元数量: {terminal_count}")
```

### 距离计算

```python
# 两个分类单元间距离
distance = tree.distance("Species_A", "Species_B")
print(f"距离: {distance:.3f}")

# 创建距离矩阵
from Bio import Phylo

terminals = tree.get_terminals()
taxa_names = [t.name for t in terminals]

print("距离矩阵:")
for taxon1 in taxa_names:
    row = []
    for taxon2 in taxa_names:
        if taxon1 == taxon2:
            row.append(0)
        else:
            dist = tree.distance(taxon1, taxon2)
            row.append(dist)
    print(f"{taxon1}: {row}")
```

### 共同祖先

```python
# 查找两个分支的共同祖先
clade1 = tree.find_any(name="Species_A")
clade2 = tree.find_any(name="Species_B")
ancestor = tree.common_ancestor(clade1, clade2)
print(f"共同祖先: {ancestor.name}")

# 查找多个分支的共同祖先
clades = [tree.find_any(name=n) for n in ["Species_A", "Species_B", "Species_C"]]
ancestor = tree.common_ancestor(*clades)
```

### 树比较

```python
# 比较树拓扑结构
def compare_trees(tree1, tree2):
    """比较两棵树"""
    # 获取末端名称
    taxa1 = set(t.name for t in tree1.get_terminals())
    taxa2 = set(t.name for t in tree2.get_terminals())

    # 检查分类单元是否相同
    if taxa1 != taxa2:
        return False, "分类单元不同"

    # 比较距离
    differences = []
    for taxon1 in taxa1:
        for taxon2 in taxa1:
            if taxon1 < taxon2:
                dist1 = tree1.distance(taxon1, taxon2)
                dist2 = tree2.distance(taxon1, taxon2)
                if abs(dist1 - dist2) > 0.01:
                    differences.append((taxon1, taxon2, dist1, dist2))

    return len(differences) == 0, differences
```

## 树操作

### 修剪树

```python
# 修剪（移除）特定分类单元
tree_copy = tree.copy()
tree_copy.prune("Species_A")

# 仅保留特定分类单元
taxa_to_keep = ["Species_B", "Species_C", "Species_D"]
terminals = tree_copy.get_terminals()
for terminal in terminals:
    if terminal.name not in taxa_to_keep:
        tree_copy.prune(terminal)
```

### 折叠短分支

```python
# 折叠短于阈值的分支
def collapse_short_branches(tree, threshold=0.01):
    """折叠短于阈值的分支"""
    for clade in tree.find_clades():
        if clade.branch_length and clade.branch_length < threshold:
            clade.branch_length = 0
    return tree
```

### 梯形化树

```python
# 梯形化树（按分支大小排序）
tree.ladderize()  # 升序排列
tree.ladderize(reverse=True)  # 降序排列
```

### 重定根节点

```python
# 中点重定根
tree.root_at_midpoint()

# 使用外群重定根
outgroup = tree.find_any(name="Outgroup_Species")
tree.root_with_outgroup(outgroup)

# 在内部节点重定根
internal = tree.get_nonterminals()[0]
tree.root_with_outgroup(internal)
```

## 树可视化

### 基础 ASCII 绘图

```python
# 在控制台绘制树
Phylo.draw_ascii(tree)

# 自定义格式绘制
Phylo.draw_ascii(tree, column_width=80)
```

### Matplotlib 可视化

```python
import matplotlib.pyplot as plt
from Bio import Phylo

# 简单绘图
fig = plt.figure(figsize=(10, 8))
axes = fig.add_subplot(1, 1, 1)
Phylo.draw(tree, axes=axes)
plt.show()

# 自定义绘图
fig = plt.figure(figsize=(10, 8))
axes = fig.add_subplot(1, 1, 1)
Phylo.draw(tree, axes=axes, do_show=False)
axes.set_title("系统发育树")
plt.tight_layout()
plt.savefig("tree.png", dpi=300)
```

### 高级可视化选项

```python
# 径向（圆形）树
Phylo.draw(tree, branch_labels=lambda c: c.branch_length)

# 显示分支支持值
Phylo.draw(tree, label_func=lambda n: str(n.confidence) if n.confidence else "")

# 分支着色
def color_by_length(clade):
    if clade.branch_length:
        if clade.branch_length > 0.5:
            return "red"
        elif clade.branch_length > 0.2:
            return "orange"
    return "black"

# 注意：直接分支着色需要自定义 matplotlib 代码
```

## 构建树

### 从距离矩阵构建

```python
from Bio.Phylo.TreeConstruction import DistanceTreeConstructor, DistanceMatrix

# 创建距离矩阵
dm = DistanceMatrix(
    names=["Alpha", "Beta", "Gamma", "Delta"],
    matrix=[
        [],
        [0.23],
        [0.45, 0.34],
        [0.67, 0.58, 0.29]
    ]
)

# 使用 UPGMA 构建树
constructor = DistanceTreeConstructor()
tree = constructor.upgma(dm)
Phylo.draw_ascii(tree)

# 使用邻接法构建树
tree = constructor.nj(dm)
```

### 从多序列比对构建

```python
from Bio import AlignIO, Phylo
from Bio.Phylo.TreeConstruction import DistanceCalculator, DistanceTreeConstructor

# 读取比对结果
alignment = AlignIO.read("alignment.fasta", "fasta")

# 计算距离矩阵
calculator = DistanceCalculator("identity")
distance_matrix = calculator.get_distance(alignment)

# 构建树
constructor = DistanceTreeConstructor()
tree = constructor.upgma(distance_matrix)

# 写入树
Phylo.write(tree, "output_tree.nwk", "newick")
```

### 距离模型

可用距离计算模型：
- **identity** - 简单一致性
- **blastn** - BLASTN 一致性
- **trans** - 转换/颠换比率
- **blosum62** - BLOSUM62 矩阵
- **pam250** - PAM250 矩阵

```python
# 使用不同模型
calculator = DistanceCalculator("blosum62")
dm = calculator.get_distance(alignment)
```

## 共识树

```python
from Bio.Phylo.Consensus import majority_consensus, strict_consensus

# 读取多棵树
trees = list(Phylo.parse("bootstrap_trees.nwk", "newick"))

# 多数规则共识树
consensus = majority_consensus(trees, cutoff=0.5)

# 严格共识树
strict_cons = strict_consensus(trees)

# 写入共识树
Phylo.write(consensus, "consensus.nwk", "newick")
```

## PhyloXML 特性

PhyloXML 格式支持丰富注释：

```python
from Bio.Phylo.PhyloXML import Phylogeny, Clade

# 创建 PhyloXML 树
tree = Phylogeny(rooted=True)
tree.name = "示例树"
tree.description = "样本系统发育树"

# 添加带注释的分支
clade = Clade(branch_length=0.5)
clade.name = "Species_A"
clade.color = "red"
clade.width = 2.0

# 添加分类信息
from Bio.Phylo.PhyloXML import Taxonomy
taxonomy = Taxonomy(scientific_name="Homo sapiens", common_name="人类")
clade.taxonomies.append(taxonomy)
```

## 自举支持率

```python
# 为树添加自举支持率
def add_bootstrap_support(tree, support_values):
    """为内部节点添加自举支持率"""
    internal_nodes = tree.get_nonterminals()
    for node, support in zip(internal_nodes, support_values):
        node.confidence = support
    return tree

# 示例
support_values = [95, 87, 76, 92]
tree_with_support = add_bootstrap_support(tree, support_values)
```

## 最佳实践

1. **选择合适的文件格式** - Newick 用于简单树，phyloXML 用于带注释的树
2. **验证树拓扑结构** - 检查多歧分支和负分支长度
3. **合理确定根节点** - 使用中点或外群定根
4. **处理自举值** - 存储为分支置信度
5. **考虑树规模** - 大型树需特殊处理
6. **使用树副本** - 修改前调用 `.copy()`
7. **导出出版级图像** - 使用 matplotlib 生成高质量输出
8. **记录建树过程** - 保存比对参数和构建参数
9. **比较多棵树** - 对自举树使用共识方法
10. **验证分类单元名称** - 确保跨文件命名一致性

## 常见用例

### 从序列构建树

```python
from Bio import AlignIO, Phylo
from Bio.Phylo.TreeConstruction import DistanceCalculator, DistanceTreeConstructor

# 读取比对序列
alignment = AlignIO.read("sequences.aln", "clustal")

# 计算距离
calculator = DistanceCalculator("identity")
dm = calculator.get_distance(alignment)

# 构建邻接树
constructor = DistanceTreeConstructor()
tree = constructor.nj(dm)

# 中点定根
tree.root_at_midpoint()

# 保存树
Phylo.write(tree, "tree.nwk", "newick")

# 可视化
import matplotlib.pyplot as plt
fig = plt.figure(figsize=(10, 8))
Phylo.draw(tree)
plt.show()
```

### 提取子树

```python
def extract_subtree(tree, taxa_list):
    """提取包含特定分类单元的子树"""
    # 创建副本
    subtree = tree.copy()

    # 获取所有末端节点
    all_terminals = subtree.get_terminals()

    # 修剪不在列表中的分类单元
    for terminal in all_terminals:
        if terminal.name not in taxa_list:
            subtree.prune(terminal)

    return subtree

# 使用示例
subtree = extract_subtree(tree, ["Species_A", "Species_B", "Species_C"])
Phylo.write(subtree, "subtree.nwk", "newick")
```

### 计算系统发育多样性

```python
def phylogenetic_diversity(tree, taxa_subset=None):
    """计算系统发育多样性（分支长度总和）"""
    if taxa_subset:
        # 修剪至子集
        tree = extract_subtree(tree, taxa_subset)

    # 累加所有分支长度
    total = 0
    for clade in tree.find_clades():
        if clade.branch_length:
            total += clade.branch_length

    return total

# 计算全部分类单元多样性
pd_all = phylogenetic_diversity(tree)
print(f"总系统发育多样性: {pd_all:.3f}")

# 计算子集多样性
pd_subset = phylogenetic_diversity(tree, ["Species_A", "Species_B"])
print(f"子集系统发育多样性: {pd_subset:.3f}")
```

### 使用外部数据注释树

```python
def annotate_tree_from_csv(tree, csv_file):
    """用 CSV 数据注释树末端节点"""
    import csv

    # 读取注释数据
    annotations = {}
    with open(csv_file) as f:
        reader = csv.DictReader(f)
        for row in reader:
            annotations[row["species"]] = row

    # 注释树
    for terminal in tree.get_terminals():
        if terminal.name in annotations:
            # 添加自定义属性
            for key, value in annotations[terminal.name].items():
                setattr(terminal, key, value)

    return tree
```

### 比较树拓扑结构

```python
def robinson_foulds_distance(tree1, tree2):
    """计算两棵树间的 Robinson-Foulds 距离"""
    # 获取每棵树的二分结构
    def get_bipartitions(tree):
        bipartitions = set()
        for clade in tree.get_nonterminals():
            terminals = frozenset(t.name for t in clade.get_terminals())
            bipartitions.add(terminals)
        return bipartitions

    bp1 = get_bipartitions(tree1)
    bp2 = get_bipartitions(tree2)

    # 对称差集
    diff = len(bp1.symmetric_difference(bp2))
    return diff

# 使用示例
tree1 = Phylo.read("tree1.nwk", "newick")
tree2 = Phylo.read("tree2.nwk", "newick")
rf_dist = robinson_foulds_distance(tree1, tree2)
print(f"Robinson-Foulds 距离:
