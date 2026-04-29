# NetworkX 图论基础

## 图类型

NetworkX 支持四种主要图类：

### Graph（无向图）
```python
import networkx as nx
G = nx.Graph()
```
- 节点间具有单条边的无向图
- 不允许平行边
- 边是双向的

### DiGraph（有向图）
```python
G = nx.DiGraph()
```
- 具有单向连接的有向图
- 边方向至关重要：(u, v) ≠ (v, u)
- 用于建模有向关系

### MultiGraph（无向多边图）
```python
G = nx.MultiGraph()
```
- 允许相同节点对之间存在多条边
- 适用于建模多重关系

### MultiDiGraph（有向多边图）
```python
G = nx.MultiDiGraph()
```
- 节点间允许多条有向边的图
- 结合了 DiGraph 和 MultiGraph 的特性

## 创建与添加节点

### 单节点添加
```python
G.add_node(1)
G.add_node("protein_A")
G.add_node((x, y))  # 节点可为任意可哈希类型
```

### 批量节点添加
```python
G.add_nodes_from([2, 3, 4])
G.add_nodes_from(range(100, 110))
```

### 带属性的节点
```python
G.add_node(1, time='5pm', color='red')
G.add_nodes_from([
    (4, {"color": "red"}),
    (5, {"color": "blue", "weight": 1.5})
])
```

### 重要节点特性
- 节点可为任意可哈希Python对象：字符串、元组、数字、自定义对象
- 节点属性以键值对形式存储
- 使用有意义的节点标识符提高可读性

## 创建与添加边

### 单边添加
```python
G.add_edge(1, 2)
G.add_edge('gene_A', 'gene_B')
```

### 批量边添加
```python
G.add_edges_from([(1, 2), (1, 3), (2, 4)])
G.add_edges_from(edge_list)
```

### 带属性的边
```python
G.add_edge(1, 2, weight=4.7, relation='interacts')
G.add_edges_from([
    (1, 2, {'weight': 4.7}),
    (2, 3, {'weight': 8.2, 'color': 'blue'})
])
```

### 从带属性的边列表添加
```python
# 从pandas数据框导入
import pandas as pd
df = pd.DataFrame({'source': [1, 2], 'target': [2, 3], 'weight': [4.7, 8.2]})
G = nx.from_pandas_edgelist(df, 'source', 'target', edge_attr='weight')
```

## 检查图结构

### 基础属性
```python
# 获取集合
G.nodes              # 所有节点的NodeView
G.edges              # 所有边的EdgeView
G.adj                # 邻接关系的AdjacencyView

# 元素计数
G.number_of_nodes()  # 节点总数
G.number_of_edges()  # 边总数
len(G)              # 节点数（简写形式）

# 度信息
G.degree()          # 所有节点度的DegreeView
G.degree(1)         # 特定节点的度
list(G.degree())    # (节点, 度) 元组列表
```

### 存在性检查
```python
# 检查节点存在
1 in G              # 返回True/False
G.has_node(1)

# 检查边存在
G.has_edge(1, 2)
```

### 访问邻居节点
```python
# 获取节点1的邻居
list(G.neighbors(1))
list(G[1])          # 类字典访问方式

# 有向图专用
list(G.predecessors(1))  # 入边邻居
list(G.successors(1))    # 出边邻居
```

### 遍历元素
```python
# 遍历节点
for node in G.nodes:
    print(node, G.nodes[node])  # 访问节点属性

# 遍历边
for u, v in G.edges:
    print(u, v, G[u][v])  # 访问边属性

# 带属性遍历
for node, attrs in G.nodes(data=True):
    print(node, attrs)

for u, v, attrs in G.edges(data=True):
    print(u, v, attrs)
```

## 修改图结构

### 删除元素
```python
# 删除单个节点（同时移除关联边）
G.remove_node(1)

# 批量删除节点
G.remove_nodes_from([1, 2, 3])

# 删除边
G.remove_edge(1, 2)
G.remove_edges_from([(1, 2), (2, 3)])
```

### 清空图
```python
G.clear()           # 移除所有节点和边
G.clear_edges()     # 仅移除边，保留节点
```

## 属性与元数据

### 图级属性
```python
G.graph['name'] = '社交网络'
G.graph['date'] = '2025-01-15'
print(G.graph)
```

### 节点属性
```python
# 创建时设置
G.add_node(1, time='5pm', weight=0.5)

# 创建后设置
G.nodes[1]['time'] = '6pm'
nx.set_node_attributes(G, {1: 'red', 2: 'blue'}, 'color')

# 获取属性
G.nodes[1]
G.nodes[1]['time']
nx.get_node_attributes(G, 'color')
```

### 边属性
```python
# 创建时设置
G.add_edge(1, 2, weight=4.7, color='red')

# 创建后设置
G[1][2]['weight'] = 5.0
nx.set_edge_attributes(G, {(1, 2): 10.5}, 'weight')

# 获取属性
G[1][2]
G[1][2]['weight']
G.edges[1, 2]
nx.get_edge_attributes(G, 'weight')
```

## 子图与视图

### 创建子图
```python
# 从节点列表创建子图
nodes_subset = [1, 2, 3, 4]
H = G.subgraph(nodes_subset)  # 返回视图（引用原图）

# 创建独立副本
H = G.subgraph(nodes_subset).copy()

# 边诱导子图
edge_subset = [(1, 2), (2, 3)]
H = G.edge_subgraph(edge_subset)
```

### 图视图
```python
# 反向视图（用于有向图）
G_reversed = G.reverse()

# 有向/无向图转换
G_undirected = G.to_undirected()
G_directed = G.to_directed()
```

## 图信息与诊断

### 基础信息
```python
print(nx.info(G))   # 图结构摘要

# 密度（实际边数与可能边数之比）
nx.density(G)

# 检查是否为有向图
G.is_directed()

# 检查是否为多重图
G.is_multigraph()
```

### 连通性检查
```python
# 无向图专用
nx.is_connected(G)
nx.number_connected_components(G)

# 有向图专用
nx.is_strongly_connected(G)
nx.is_weakly_connected(G)
```

## 重要注意事项

### 浮点数精度
当图中包含浮点数时，由于精度限制，所有计算结果本质上是近似值。微小的算术误差可能影响算法结果，尤其在最小/最大计算中。

### 内存考量
每次脚本启动时，图数据必须加载到内存中。对于大型数据集，可能导致性能问题。建议：
- 使用高效数据格式（Python对象用pickle）
- 仅加载必要子图
- 超大规模网络使用图数据库

### 节点与边删除行为
删除节点时，所有与该节点关联的边将自动移除。
