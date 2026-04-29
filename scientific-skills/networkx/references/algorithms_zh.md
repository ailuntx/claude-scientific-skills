# NetworkX 图算法

## 最短路径

### 单源最短路径
```python
# Dijkstra 算法（加权图）
path = nx.shortest_path(G, source=1, target=5, weight='weight')
length = nx.shortest_path_length(G, source=1, target=5, weight='weight')

# 源点到所有节点的最短路径
paths = nx.single_source_shortest_path(G, source=1)
lengths = nx.single_source_shortest_path_length(G, source=1)

# Bellman-Ford（支持负权重）
path = nx.bellman_ford_path(G, source=1, target=5, weight='weight')
```

### 全节点对最短路径
```python
# 全节点对（返回迭代器）
for source, paths in nx.all_pairs_shortest_path(G):
    print(f"从 {source} 出发: {paths}")

# Floyd-Warshall 算法
lengths = dict(nx.all_pairs_shortest_path_length(G))
```

### 专用最短路径算法
```python
# A* 算法（带启发函数）
def heuristic(u, v):
    # 自定义启发函数
    return abs(u - v)

path = nx.astar_path(G, source=1, target=5, heuristic=heuristic, weight='weight')

# 平均最短路径长度
avg_length = nx.average_shortest_path_length(G)
```

## 连通性

### 连通分量（无向图）
```python
# 检查连通性
is_connected = nx.is_connected(G)

# 连通分量数量
num_components = nx.number_connected_components(G)

# 获取所有分量（返回集合迭代器）
components = list(nx.connected_components(G))
largest_component = max(components, key=len)

# 获取包含特定节点的分量
component = nx.node_connected_component(G, node=1)
```

### 强/弱连通性（有向图）
```python
# 强连通性（双向可达）
is_strongly_connected = nx.is_strongly_connected(G)
strong_components = list(nx.strongly_connected_components(G))
largest_scc = max(strong_components, key=len)

# 弱连通性（忽略方向）
is_weakly_connected = nx.is_weakly_connected(G)
weak_components = list(nx.weakly_connected_components(G))

# 凝聚图（强连通分量构成的DAG）
condensed = nx.condensation(G)
```

### 割集与连通度
```python
# 最小节点/边割集
min_node_cut = nx.minimum_node_cut(G, s=1, t=5)
min_edge_cut = nx.minimum_edge_cut(G, s=1, t=5)

# 节点/边连通度
node_connectivity = nx.node_connectivity(G)
edge_connectivity = nx.edge_connectivity(G)
```

## 中心性度量

### 度中心性
```python
# 节点连接比例
degree_cent = nx.degree_centrality(G)

# 有向图专用
in_degree_cent = nx.in_degree_centrality(G)
out_degree_cent = nx.out_degree_centrality(G)
```

### 介数中心性
```python
# 节点在最短路径中的出现比例
betweenness = nx.betweenness_centrality(G, weight='weight')

# 边介数中心性
edge_betweenness = nx.edge_betweenness_centrality(G, weight='weight')

# 大型图近似计算
approx_betweenness = nx.betweenness_centrality(G, k=100)  # 采样100个节点
```

### 接近中心性
```python
# 平均最短路径长度的倒数
closeness = nx.closeness_centrality(G)

# 非连通图专用
closeness = nx.closeness_centrality(G, wf_improved=True)
```

### 特征向量中心性
```python
# 基于高中心性节点的连接关系
eigenvector = nx.eigenvector_centrality(G, max_iter=1000)

# Katz中心性（带衰减因子）
katz = nx.katz_centrality(G, alpha=0.1, beta=1.0)
```

### PageRank
```python
# Google PageRank算法
pagerank = nx.pagerank(G, alpha=0.85)

# 个性化PageRank
personalization = {node: 1.0 if node in [1, 2] else 0.0 for node in G}
ppr = nx.pagerank(G, personalization=personalization)
```

## 聚类分析

### 聚类系数
```python
# 节点聚类系数
clustering = nx.clustering(G)

# 平均聚类系数
avg_clustering = nx.average_clustering(G)

# 加权聚类系数
weighted_clustering = nx.clustering(G, weight='weight')
```

### 传递性
```python
# 整体聚类度（三角形与三元组比例）
transitivity = nx.transitivity(G)
```

### 三角形计数
```python
# 节点三角形数量
triangles = nx.triangles(G)

# 三角形总数
total_triangles = sum(triangles.values()) // 3
```

## 社区发现

### 模块度优化
```python
from networkx.algorithms import community

# 贪婪模块度最大化
communities = community.greedy_modularity_communities(G)

# 计算模块度
modularity = community.modularity(G, communities)
```

### 标签传播
```python
# 快速社区发现
communities = community.label_propagation_communities(G)
```

### Girvan-Newman
```python
# 基于边介数的层次化社区发现
comp = community.girvan_newman(G)
limited = itertools.takewhile(lambda c: len(c) <= 10, comp)
for communities in limited:
    print(tuple(sorted(c) for c in communities))
```

## 匹配与覆盖

### 最大匹配
```python
# 最大基数匹配
matching = nx.max_weight_matching(G)

# 验证匹配有效性
is_matching = nx.is_matching(G, matching)
is_perfect = nx.is_perfect_matching(G, matching)
```

### 最小顶点/边覆盖
```python
# 覆盖所有边的最小节点集
min_vertex_cover = nx.approximation.min_weighted_vertex_cover(G)

# 最小边支配集
min_edge_dom = nx.approximation.min_edge_dominating_set(G)
```

## 树算法

### 最小生成树
```python
# Kruskal或Prim算法
mst = nx.minimum_spanning_tree(G, weight='weight')

# 最大生成树
mst_max = nx.maximum_spanning_tree(G, weight='weight')

# 枚举所有生成树
all_spanning = nx.all_spanning_trees(G)
```

### 树属性
```python
# 检查是否为树
is_tree = nx.is_tree(G)
is_forest = nx.is_forest(G)

# 有向图专用
is_arborescence = nx.is_arborescence(G)
```

## 流与容量

### 最大流
```python
# 最大流值
flow_value = nx.maximum_flow_value(G, s=1, t=5, capacity='capacity')

# 带流字典的最大流
flow_value, flow_dict = nx.maximum_flow(G, s=1, t=5, capacity='capacity')

# 最小割
cut_value, partition = nx.minimum_cut(G, s=1, t=5, capacity='capacity')
```

### 最小费用流
```python
# 最小费用流
flow_dict = nx.min_cost_flow(G, demand='demand', capacity='capacity', weight='weight')
cost = nx.cost_of_flow(G, flow_dict, weight='weight')
```

## 环路检测

### 环路查找
```python
# 简单环路（有向图）
cycles = list(nx.simple_cycles(G))

# 环路基（无向图）
basis = nx.cycle_basis(G)

# 检查无环性
is_dag = nx.is_directed_acyclic_graph(G)
```

### 拓扑排序
```python
# 仅适用于DAG
try:
    topo_order = list(nx.topological_sort(G))
except nx.NetworkXError:
    print("图包含环路")

# 所有拓扑排序
all_topo = nx.all_topological_sorts(G)
```

## 团分析

### 团查找
```python
# 所有极大团
cliques = list(nx.find_cliques(G))

# 最大团（NP完全问题，近似解）
max_clique = nx.approximation.max_clique(G)

# 团数
clique_number = nx.graph_clique_number(G)

# 包含节点的极大团数量
clique_counts = nx.node_clique_number(G)
```

## 图着色

### 节点着色
```python
# 贪婪着色
coloring = nx.greedy_color(G, strategy='largest_first')

# 不同策略: 'largest_first', 'smallest_last', 'random_sequential'
coloring = nx.greedy_color(G, strategy='smallest_last')
```

## 同构分析

### 图同构
```python
# 检查图同构
is_isomorphic = nx.is_isomorphic(G1, G2)

# 获取同构映射
from networkx.algorithms import isomorphism
GM = isomorphism.GraphMatcher(G1, G2)
if GM.is_isomorphic():
    mapping = GM.mapping
```

### 子图同构
```python
# 检查G1是否与G2的子图同构
is_subgraph_iso = nx.is_isomorphic(G1, G2.subgraph(nodes))
```

## 遍历算法

### 深度优先搜索 (DFS)
```python
# DFS边序列
dfs_edges = list(nx.dfs_edges(G, source=1))

# DFS树
dfs_tree = nx.dfs_tree(G, source=1)

# DFS前驱节点
dfs_pred = nx.dfs_predecessors(G, source=1)

# 前序与后序遍历
preorder = list(nx.dfs_preorder_nodes(G, source=1))
postorder = list(nx.dfs_postorder_nodes(G, source=1))
```

### 广度优先搜索 (BFS)
```python
# BFS边序列
bfs_edges = list(nx.bfs_edges(G, source=1))

# BFS树
bfs_tree = nx.bfs_tree(G, source=1)

# BFS前驱与后继节点
bfs_pred = nx.bfs_predecessors(G, source=1)
bfs_succ = nx.bfs_successors(G, source=1)
```

## 效率考量

### 算法复杂度
- 多数算法提供参数控制计算时间
- 大型图建议使用近似算法
- 中心性计算使用`k`参数采样节点
- 迭代算法设置`max_iter`参数

### 内存使用
- 迭代器函数（如`nx.simple_cycles()`）节省内存
- 仅在必要时转换为列表
- 大型结果集使用生成器

### 数值精度
使用浮点数的加权算法结果为近似值，需注意：
- 尽可能使用整数权重
- 设置合适的容差参数
- 警惕迭代算法的累积舍入误差
