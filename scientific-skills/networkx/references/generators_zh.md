# NetworkX 图生成器

## 经典图

### 完全图
```python
# 完全图（所有节点相互连接）
G = nx.complete_graph(n=10)

# 完全二分图
G = nx.complete_bipartite_graph(n1=5, n2=7)

# 完全多分图
G = nx.complete_multipartite_graph(3, 4, 5)  # 三个分区
```

### 环图与路径图
```python
# 环图（节点环形排列）
G = nx.cycle_graph(n=20)

# 路径图（线性链）
G = nx.path_graph(n=15)

# 环形阶梯图
G = nx.circular_ladder_graph(n=10)
```

### 正则图
```python
# 空图（无边）
G = nx.empty_graph(n=10)

# 零图（无节点）
G = nx.null_graph()

# 星形图（中心节点连接所有其他节点）
G = nx.star_graph(n=19)  # 创建20节点星形图

# 轮形图（带中心枢纽的环）
G = nx.wheel_graph(n=10)
```

### 特殊命名图
```python
# 牛头图
G = nx.bull_graph()

# Chvatal图
G = nx.chvatal_graph()

# 立方体图
G = nx.cubical_graph()

# 钻石图
G = nx.diamond_graph()

# 十二面体图
G = nx.dodecahedral_graph()

# Heawood图
G = nx.heawood_graph()

# 房屋图
G = nx.house_graph()

# Petersen图
G = nx.petersen_graph()

# 空手道俱乐部图（经典社交网络）
G = nx.karate_club_graph()
```

## 随机图

### Erdős-Rényi图
```python
# G(n, p)模型：n个节点，边概率p
G = nx.erdos_renyi_graph(n=100, p=0.1, seed=42)

# G(n, m)模型：n个节点，精确m条边
G = nx.gnm_random_graph(n=100, m=500, seed=42)

# 快速版本（适用于大型稀疏图）
G = nx.fast_gnp_random_graph(n=10000, p=0.0001, seed=42)
```

### Watts-Strogatz小世界网络
```python
# 带重连的小世界网络
# n个节点，k个最近邻居，重连概率p
G = nx.watts_strogatz_graph(n=100, k=6, p=0.1, seed=42)

# 连通版本（保证连通性）
G = nx.connected_watts_strogatz_graph(n=100, k=6, p=0.1, tries=100, seed=42)
```

### Barabási-Albert偏好依附
```python
# 无标度网络（幂律度分布）
# n个节点，新节点附加m条边
G = nx.barabasi_albert_graph(n=100, m=3, seed=42)

# 带参数的扩展版本
G = nx.extended_barabasi_albert_graph(n=100, m=3, p=0.5, q=0.2, seed=42)
```

### 幂律度序列
```python
# 幂律聚类图
G = nx.powerlaw_cluster_graph(n=100, m=3, p=0.1, seed=42)

# 随机幂律树
G = nx.random_powerlaw_tree(n=100, gamma=3, seed=42, tries=1000)
```

### 配置模型
```python
# 指定度序列的图
degree_sequence = [3, 3, 3, 3, 2, 2, 2, 1, 1, 1]
G = nx.configuration_model(degree_sequence, seed=42)

# 移除自环和平行边
G = nx.Graph(G)
G.remove_edges_from(nx.selfloop_edges(G))
```

### 随机几何图
```python
# 单位正方形内节点，距离小于半径则连边
G = nx.random_geometric_graph(n=100, radius=0.2, seed=42)

# 带位置信息
pos = nx.get_node_attributes(G, 'pos')
```

### 随机正则图
```python
# 每个节点恰好有d个邻居
G = nx.random_regular_graph(d=3, n=100, seed=42)
```

### 随机块模型
```python
# 社区结构模型
sizes = [50, 50, 50]  # 三个社区
probs = [[0.25, 0.05, 0.02],  # 社区内/间连接概率
         [0.05, 0.35, 0.07],
         [0.02, 0.07, 0.40]]
G = nx.stochastic_block_model(sizes, probs, seed=42)
```

## 格点与网格图

### 网格图
```python
# 二维网格
G = nx.grid_2d_graph(m=5, n=7)  # 5x7网格

# 三维网格
G = nx.grid_graph(dim=[5, 7, 3])  # 5x7x3网格

# 六边形晶格
G = nx.hexagonal_lattice_graph(m=5, n=7)

# 三角形晶格
G = nx.triangular_lattice_graph(m=5, n=7)
```

### 超立方体
```python
# n维超立方体
G = nx.hypercube_graph(n=4)
```

## 树图

### 随机树
```python
# 含n个节点的随机树
G = nx.random_tree(n=100, seed=42)

# 前缀树（字典树）
G = nx.prefix_tree([[0, 1, 2], [0, 1, 3], [0, 4]])
```

### 平衡树
```python
# 高度h的平衡r叉树
G = nx.balanced_tree(r=2, h=5)  # 二叉树，高度5

# 含n个节点的完全r叉树
G = nx.full_rary_tree(r=3, n=100)  # 三叉树
```

### 杠铃图与棒棒糖图
```python
# 通过路径连接的两个完全图
G = nx.barbell_graph(m1=5, m2=3)  # 两个K_5图通过3节点路径连接

# 完全图连接路径
G = nx.lollipop_graph(m=7, n=5)  # K_7连接5节点路径
```

## 社交网络模型

### 空手道俱乐部
```python
# Zachary空手道俱乐部（经典社交网络）
G = nx.karate_club_graph()
```

### Davis南方女性网络
```python
# 二分社交网络
G = nx.davis_southern_women_graph()
```

### 佛罗伦萨家族网络
```python
# 历史婚姻与商业网络
G = nx.florentine_families_graph()
```

### 悲惨世界网络
```python
# 角色共现网络
G = nx.les_miserables_graph()
```

## 有向图生成器

### 随机有向图
```python
# 有向Erdős-Rényi图
G = nx.gnp_random_graph(n=100, p=0.1, directed=True, seed=42)

# 无标度有向图
G = nx.scale_free_graph(n=100, seed=42)
```

### DAG（有向无环图）
```python
# 随机DAG
G = nx.gnp_random_graph(n=20, p=0.2, directed=True, seed=42)
G = nx.DiGraph([(u, v) for (u, v) in G.edges() if u < v])  # 移除反向边
```

### 竞赛图
```python
# 随机竞赛图（完全有向图）
G = nx.random_tournament(n=10, seed=42)
```

## 复制-发散模型

### 复制发散图
```python
# 生物网络模型（蛋白质相互作用网络）
G = nx.duplication_divergence_graph(n=100, p=0.5, seed=42)
```

## 度序列生成器

### 有效度序列
```python
# 检查度序列是否有效（可图形化）
sequence = [3, 3, 3, 3, 2, 2, 2, 1, 1, 1]
is_valid = nx.is_graphical(sequence)

# 有向图版本
in_sequence = [2, 2, 2, 1, 1]
out_sequence = [2, 2, 1, 2, 1]
is_valid = nx.is_digraphical(in_sequence, out_sequence)
```

### 从度序列创建
```python
# Havel-Hakimi算法
G = nx.havel_hakimi_graph(degree_sequence)

# 配置模型（允许多重边/自环）
G = nx.configuration_model(degree_sequence)

# 有向配置模型
G = nx.directed_configuration_model(in_degree_sequence, out_degree_sequence)
```

## 二分图

### 随机二分图
```python
# 含两个节点集的随机二分图
G = nx.bipartite.random_graph(n=50, m=30, p=0.1, seed=42)

# 二分图配置模型
G = nx.bipartite.configuration_model(deg1=[3, 3, 2], deg2=[2, 2, 2, 2], seed=42)
```

### 二分图生成器
```python
# 完全二分图
G = nx.complete_bipartite_graph(n1=5, n2=7)

# Gnmk随机二分图（n, m个节点，k条边）
G = nx.bipartite.gnmk_random_graph(n=10, m=8, k=20, seed=42)
```

## 图操作符

### 图运算
```python
# 并集
G = nx.union(G1, G2)

# 不相交并集
G = nx.disjoint_union(G1, G2)

# 复合（叠加）
G = nx.compose(G1, G2)

# 补图
G = nx.complement(G1)

# 笛卡尔积
G = nx.cartesian_product(G1, G2)

# 张量（Kronecker）积
G = nx.tensor_product(G1, G2)

# 强积
G = nx.strong_product(G1, G2)
```

## 定制与种子设置

### 设置随机种子
始终设置种子以确保可复现性：
```python
G = nx.erdos_renyi_graph(n=100, p=0.1, seed=42)
```

### 转换图类型
```python
# 转换为特定类型
G_directed = G.to_directed()
G_undirected = G.to_undirected()
G_multi = nx.MultiGraph(G)
```

## 性能考量

### 快速生成器
大型图使用优化生成器：
```python
# 快速ER图（稀疏）
G = nx.fast_gnp_random_graph(n=10000, p=0.0001, seed=42)
```

### 内存效率
部分生成器采用增量创建节省内存。超大图建议：
- 使用稀疏表示
- 按需生成子图
- 使用邻接表或边表替代完整图结构

## 验证与属性

### 检查生成图
```python
# 验证属性
print(f"节点数: {G.number_of_nodes()}")
print(f"边数: {G.number_of_edges()}")
print(f"密度: {nx.density(G)}")
print(f"连通性: {nx.is_connected(G)}")

# 度分布
degree_sequence = sorted([d for n, d in G.degree()], reverse=True)
```
