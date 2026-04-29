# ETE 工具包 API 参考

## 概述

ETE（Environment for Tree Exploration）是一个用于系统发育树操作、分析和可视化的 Python 工具包。本文档涵盖主要类和方法参考。

## 核心类

### TreeNode（别名：Tree）

表示树结构的基础类，采用分层节点组织。

**构造函数：**
```python
from ete3 import Tree
t = Tree(newick=None, format=0, dist=None, support=None, name=None)
```

**参数：**
- `newick`: Newick 字符串或文件路径
- `format`: Newick 格式（0-100）。常用格式：
  - `0`: 支持分支长度和名称的灵活格式
  - `1`: 含内部节点名称
  - `2`: 含自举/支持值
  - `5`: 内部节点名称和分支长度
  - `8`: 全特性（名称、距离、支持值）
  - `9`: 仅叶节点名称
  - `100`: 仅拓扑结构
- `dist`: 到父节点的分支长度（默认：1.0）
- `support`: 自举/置信值（默认：1.0）
- `name`: 节点标识符

### PhyloTree

专用于系统发育分析的特殊类，继承自 TreeNode。

**构造函数：**
```python
from ete3 import PhyloTree
t = PhyloTree(newick=None, alignment=None, alg_format='fasta',
              sp_naming_function=None, format=0)
```

**附加参数：**
- `alignment`: 比对文件路径或比对字符串
- `alg_format`: 'fasta' 或 'phylip'
- `sp_naming_function`: 从节点名提取物种的自定义函数

### ClusterTree

用于层次聚类分析的类。

**构造函数：**
```python
from ete3 import ClusterTree
t = ClusterTree(newick, text_array=None)
```

**参数：**
- `text_array`: 带列标题和行名的制表符分隔矩阵

### NCBITaxa

用于 NCBI 分类数据库操作的类。

**构造函数：**
```python
from ete3 import NCBITaxa
ncbi = NCBITaxa(dbfile=None)
```

首次实例化会下载约 300MB 的 NCBI 分类数据库到 `~/.etetoolkit/taxa.sqlite`。

## 节点属性

### 基础属性

| 属性 | 类型 | 描述 | 默认值 |
|------|------|------|--------|
| `name` | str | 节点标识符 | "NoName" |
| `dist` | float | 到父节点的分支长度 | 1.0 |
| `support` | float | 自举/置信值 | 1.0 |
| `up` | TreeNode | 父节点引用 | None |
| `children` | list | 子节点列表 | [] |

### 自定义特性

添加任意自定义数据到节点：
```python
node.add_feature("custom_name", value)
node.add_features(feature1=value1, feature2=value2)
```

访问特性：
```python
value = node.custom_name
# 或
value = getattr(node, "custom_name", default_value)
```

## 导航与遍历

### 基础导航

```python
# 检查节点类型
node.is_leaf()          # 终端节点返回 True
node.is_root()          # 根节点返回 True
len(node)               # 节点下的叶节点数量

# 获取关联节点
parent = node.up
children = node.children
root = node.get_tree_root()
```

### 遍历策略

```python
# 三种遍历策略
for node in tree.traverse("preorder"):    # 根 → 左 → 右
    print(node.name)

for node in tree.traverse("postorder"):   # 左 → 右 → 根
    print(node.name)

for node in tree.traverse("levelorder"):  # 按层级遍历
    print(node.name)

# 排除根节点
for node in tree.iter_descendants("postorder"):
    print(node.name)
```

### 获取节点

```python
# 获取所有叶节点
leaves = tree.get_leaves()
for leaf in tree:  # 快捷迭代
    print(leaf.name)

# 获取所有后代节点
descendants = tree.get_descendants()

# 获取祖先节点
ancestors = node.get_ancestors()

# 按属性获取特定节点
nodes = tree.search_nodes(name="NodeA")
node = tree & "NodeA"  # 快捷语法

# 按名称获取叶节点
leaves = tree.get_leaves_by_name("LeafA")

# 获取最近共同祖先
ancestor = tree.get_common_ancestor("LeafA", "LeafB", "LeafC")

# 自定义过滤
filtered = [n for n in tree.traverse() if n.dist > 0.5 and n.is_leaf()]
```

### 迭代器方法（内存高效）

```python
# 大型树使用迭代器
for match in tree.iter_search_nodes(name="X"):
    if some_condition:
        break  # 提前终止

for leaf in tree.iter_leaves():
    process(leaf)

for descendant in node.iter_descendants():
    process(descendant)
```

## 树构建与修改

### 从零创建树

```python
# 创建空树
t = Tree()

# 添加子节点
child1 = t.add_child(name="A", dist=1.0)
child2 = t.add_child(name="B", dist=2.0)

# 添加同级节点
sister = child1.add_sister(name="C", dist=1.5)

# 随机生成拓扑
t.populate(10)  # 创建10个随机叶节点
t.populate(5, names_library=["A", "B", "C", "D", "E"])
```

### 移除与删除节点

```python
# 分离：移除整个子树
node.detach()
# 或
parent.remove_child(node)

# 删除：移除节点并将子节点连接到父节点
node.delete()
# 或
parent.remove_child(node)
```

### 剪枝

仅保留指定叶节点：
```python
# 仅保留这些叶节点，移除其余部分
tree.prune(["A", "B", "C"])

# 保留原始分支长度
tree.prune(["A", "B", "C"], preserve_branch_length=True)
```

### 树拼接

```python
# 将一棵树作为子树附加
t1 = Tree("(A,(B,C));")
t2 = Tree("((D,E),(F,G));")
A = t1 & "A"
A.add_child(t2)
```

### 树复制

```python
# 四种复制方法
copy1 = tree.copy()  # 默认：cpickle（保留类型）
copy2 = tree.copy("newick")  # 最快：基础拓扑
copy3 = tree.copy("newick-extended")  # 包含文本形式自定义特性
copy4 = tree.copy("deepcopy")  # 最慢：处理复杂对象
```

## 树操作

### 定根

```python
# 设置外群（重定根）
outgroup_node = tree & "OutgroupLeaf"
tree.set_outgroup(outgroup_node)

# 中点定根
midpoint = tree.get_midpoint_outgroup()
tree.set_outgroup(midpoint)

# 取消定根
tree.unroot()
```

### 解决多歧分支

```python
# 将多歧分支转为二叉分支
tree.resolve_polytomy(recursive=False)  # 仅处理单个节点
tree.resolve_polytomy(recursive=True)   # 处理整棵树
```

### 阶梯化排序

```python
# 按子树大小排序分支
tree.ladderize()
tree.ladderize(direction=1)  # 升序排列
```

### 转换为超度量树

```python
# 使所有叶节点到根节点等距
tree.convert_to_ultrametric()
tree.convert_to_ultrametric(tree_length=100)  # 指定总长度
```

## 距离计算与比较

### 距离计算

```python
# 节点间分支长度距离
dist = tree.get_distance("A", "B")
dist = nodeA.get_distance(nodeB)

# 纯拓扑距离（节点计数）
dist = tree.get_distance("A", "B", topology_only=True)

# 最远节点
farthest, distance = node.get_farthest_node()
farthest_leaf, distance = node.get_farthest_leaf()
```

### 单系群检测

```python
# 检测值是否构成单系群
is_mono, clade_type, base_node = tree.check_monophyly(
    values=["A", "B", "C"],
    target_attr="name"
)
# 返回：(布尔值, "单系群"|"并系群"|"多系群", 节点)

# 获取所有单系群
monophyletic_nodes = tree.get_monophyletic(
    values=["A", "B", "C"],
    target_attr="name"
)
```

### 树比较

```python
# Robinson-Foulds 距离
rf, max_rf, common_leaves, parts_t1, parts_t2 = t1.robinson_foulds(t2)
print(f"RF 距离: {rf}/{max_rf}")

# 标准化 RF 距离
result = t1.compare(t2)
norm_rf = result["norm_rf"]  # 0.0 到 1.0
ref_edges = result["ref_edges_in_source"]
```

## 输入/输出

### 读取树

```python
# 从字符串读取
t = Tree("(A:1,(B:1,(C:1,D:1):0.5):0.5);")

# 从文件读取
t = Tree("tree.nw")

# 指定格式
t = Tree("tree.nw", format=1)
```

### 写入树

```python
# 转为字符串
newick = tree.write()
newick = tree.write(format=1)
newick = tree.write(format=1, features=["support", "custom_feature"])

# 写入文件
tree.write(outfile="output.nw")
tree.write(format=5, outfile="output.nw", features=["name", "dist"])

# 自定义叶节点函数（用于折叠）
def is_leaf(node):
    return len(node) <= 3  # 将小型进化枝视为叶节点

newick = tree.write(is_leaf_fn=is_leaf)
```

### 树渲染

```python
# 显示交互式界面
tree.show()

# 渲染到文件（PNG/PDF/SVG）
tree.render("tree.png")
tree.render("tree.pdf", w=200, units="mm")
tree.render("tree.svg", dpi=300)

# ASCII 表示
print(tree)
print(tree.get_ascii(show_internal=True, compact=False))
```

## 性能优化

### 内容缓存

频繁访问节点内容时：
```python
# 缓存所有节点内容
node2content = tree.get_cached_content()

# 快速查询
for node in tree.traverse():
    leaves = node2content[node]
    print(f"节点包含 {len(leaves)} 个叶节点")
```

### 预计算距离

```python
# 用于多次距离查询
node2dist = {}
for node in tree.traverse():
    node2dist[node] = node.get_distance(tree)
```

## PhyloTree 特有方法

### 序列比对

```python
# 关联比对文件
tree.link_to_alignment("alignment.fasta", alg_format="fasta")

# 访问序列
for leaf in tree:
    print(f"{leaf.name}: {leaf.sequence}")
```

### 物种命名

```python
# 默认：取前3个字母
# 自定义函数
def get_species(node_name):
    return node_name.split("_")[0]

tree.set_species_naming_function(get_species)

# 手动设置
for leaf in tree:
    leaf.species = extract_species(leaf.name)
```

### 进化事件检测

```python
# 检测复制/分化事件
events = tree.get_descendant_evol_events()

for node in tree.traverse():
    if hasattr(node, "evoltype"):
        print(f"{node.name}: {node.evoltype}")  # "D" 或 "S"

# 使用物种树
species_tree = Tree("(human, (chimp, gorilla));")
events = tree.get_descendant_evol_events(species_tree=species_tree)
```

### 基因树操作

```python
# 从重复基因家族获取物种树
species_trees = tree.get_speciation_trees()

# 按复制事件分割
subtrees = tree.split_by_dups()

# 折叠谱系特异性扩张
tree.collapse_lineage_specific_expansions()
```

## NCBITaxa 方法

### 数据库操作

```python
from ete3 import NCBITaxa
ncbi = NCBITaxa()

# 更新数据库
ncbi.update_taxonomy_database()
```

### 分类查询

```python
# 从名称获取分类ID
taxid = ncbi.get_name_translator(["Homo sapiens"])
# 返回：{'Homo sapiens': [9606]}

# 从分类ID获取名称
names = ncbi.get_taxid_translator([9606, 9598])
# 返回：{9606: 'Homo sapiens', 9598: 'Pan troglodytes'}

# 获取分类层级
rank = ncbi.get_rank([9606])
# 返回：{9606: 'species'}

# 获取谱系
lineage = ncbi.get_lineage(9606)
# 返回：[1, 131567, 2759, ..., 9606]

# 获取后代分类单元
descendants = ncbi.get_descendant_taxa("Primates")
descendants = ncbi.get_descendant_taxa("Primates", collapse_subspecies=True)
```

### 构建分类树

```python
# 获取连接分类单元的最小树
tree = ncbi.get_topology([9606, 9598, 9593])  # 人、黑猩猩、大猩猩

# 用分类信息注释树
tree.annotate_ncbi_taxa()

# 访问分类信息
for node in tree.traverse():
    print(f"{node.sci_name} ({node.taxid}) - 层级: {node.rank}")
```

## ClusterTree 方法

### 关联数据

```python
# 将矩阵关联到树
tree.link_to_arraytable(matrix_string)

# 访问数据剖面
for leaf in tree:
    print(leaf.profile)  # 数值数组
```

### 聚类指标

```python
# 获取轮廓系数
silhouette = tree.get_silhouette()

# 获取邓恩指数
dunn = tree.get_dunn()

# 簇间/簇内距离
inter = node.intercluster_dist
intra = node.intracluster_dist

# 标准差
dev = node.deviation
```

### 距离度量

支持度量方法：
- `"euclidean"`: 欧氏距离
- `"pearson"`: 皮尔逊相关系数
- `"spearman"`: 斯皮尔曼秩相关系数

```python
tree.dist_to(node2, metric="pearson")
```

## 常见错误处理

```python
# 检查树是否为空
if tree.children:
    print("树包含子节点")

# 检查节点是否存在
nodes = tree.search_nodes(name="X")
if nodes:
    node = nodes[0]

# 安全特性访问
value = getattr(node, "feature_name", default_value)

# 检查格式兼容性
try:
    tree.write(format=1)
except:
    print("树缺少内部节点名称")
```

## 最佳实践

1. **选择合适遍历方式**：自底向上用后序，自顶向下用前序
2. **重复访问使用缓存**：频繁查询时使用 `get_cached_content()`
3. **大型树使用迭代器**：内存高效处理
4. **剪枝时保留分支长度**：使用 `preserve_branch_length=True`
5. **明智选择复制方法**：速度选 "newick"，保真选 "cpickle"
6. **验证单系性**：检查返回的进化枝类型（单系/并系/多系）
7. **系统发育分析用 PhyloTree**：专为进化分析设计
8. **缓存 NCBI 查询**：存储结果避免重复访问
