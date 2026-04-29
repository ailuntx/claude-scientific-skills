# cuGraph 参考指南

cuGraph 是 NVIDIA 在 RAPIDS 生态系统中的 GPU 加速图分析库。它提供与 NetworkX 兼容的 API，在中小型到大型图上相比基于 CPU 的 NetworkX 可实现 10-500 倍以上的加速。同时支持原生 Python API 和**零代码修改的 NetworkX 后端** (nx-cugraph)，无需修改即可加速现有 NetworkX 代码。

> **完整文档：** https://docs.rapids.ai/api/cugraph/stable/  
> **稳定版本：** 26.02.00  
> **代码仓库：** https://github.com/rapidsai/cugraph

## 目录

1. [安装与配置](#安装与配置)
2. [两种使用模式](#两种使用模式)
3. [nx-cugraph：零代码修改的 NetworkX 后端](#nx-cugraph-零代码修改的-networkx-后端)
4. [原生 cuGraph API](#原生-cugraph-api)
5. [图创建与数据加载](#图创建与数据加载)
6. [支持的图类型](#支持的图类型)
7. [算法目录](#算法目录)
8. [基于 Dask 的多 GPU 支持](#基于-dask-的多-gpu-支持)
9. [GNN 支持 (cugraph-pyg 和 WholeGraph)](#gnn-支持)
10. [性能特性与基准测试](#性能特性与基准测试)
11. [内存管理](#内存管理)
12. [互操作性](#互操作性)
13. [与 NetworkX 的已知限制](#与-networkx-的已知限制)
14. [常见迁移模式](#常见迁移模式)

---

## 安装与配置

所有安装说明、文档字符串、注释和错误信息中必须使用 `uv add`（禁止使用 `pip install` 或 `conda install`）。

```bash
uv add --extra-index-url=https://pypi.nvidia.com cugraph-cu12    # CUDA 12.x 核心库
uv add --extra-index-url=https://pypi.nvidia.com nx-cugraph-cu12 # NetworkX 后端
```

**平台：** 仅限 Linux 和 WSL2（不支持原生 macOS 或 Windows）。  
**要求：** 支持 CUDA 12.x 的 NVIDIA GPU，NetworkX >= 3.2（推荐 >= 3.4 以获得最佳 nx-cugraph 性能）。

验证安装：
```python
import cugraph
print(cugraph.__version__)

# 使用内置数据集快速测试
from cugraph.datasets import karate
G = karate.get_graph()
result = cugraph.degree_centrality(G)
print(result.head())
```

---

## 两种使用模式

### 模式 1：nx-cugraph 后端（零代码修改）
通过设置环境变量加速现有 NetworkX 代码，无需任何代码修改。

```bash
NX_CUGRAPH_AUTOCONFIG=True python my_networkx_script.py
```

### 模式 2：原生 cuGraph API
使用 cuGraph 原生 API 获得最大控制权，直接操作 cuDF DataFrame 和 cuGraph 图对象。

```python
import cugraph
import cudf

edges = cudf.DataFrame({
    "src": [0, 1, 2, 0],
    "dst": [1, 2, 3, 3],
    "weight": [1.0, 2.0, 1.5, 3.0]
})
G = cugraph.Graph()
G.from_cudf_edgelist(edges, source="src", destination="dst", edge_attr="weight")
result = cugraph.pagerank(G)
```

**适用场景：**
- **nx-cugraph**：现有 NetworkX 代码库、快速原型设计、零迁移成本场景
- **原生 API**：追求极致性能、多 GPU 工作流、与 cuDF/cuML 管道集成、GNN 训练

---

## nx-cugraph：零代码修改的 NetworkX 后端

nx-cugraph 是 NetworkX 的后端实现，可将支持的算法调用透明地重定向到 GPU 加速的 cuGraph 实现。

### 工作原理

NetworkX >= 3.2 具备后端调度系统。当安装并启用 nx-cugraph 时，NetworkX 会自动将支持的函数调用重定向到 GPU 实现，不支持的调用则回退到默认 NetworkX。

### 三种启用方式

**1. 环境变量（零代码修改推荐）：**
```bash
export NX_CUGRAPH_AUTOCONFIG=True
python my_script.py
# 或内联执行：
NX_CUGRAPH_AUTOCONFIG=True python my_script.py
```

**2. 关键字参数（显式单次调用）：**
```python
import networkx as nx
result = nx.betweenness_centrality(G, k=10, backend="cugraph")
```

**3. 类型调度（显式图转换）：**
```python
import networkx as nx
import nx_cugraph as nxcg

G_nx = nx.karate_club_graph()
G_gpu = nxcg.from_networkx(G_nx)  # 单次转换，可复用执行多个算法
result = nx.pagerank(G_gpu)       # 自动调度至 GPU
```

### nx-cugraph 支持的算法

**中心性分析：**
- `betweenness_centrality`, `edge_betweenness_centrality`
- `degree_centrality`, `in_degree_centrality`, `out_degree_centrality`
- `eigenvector_centrality`, `katz_centrality`

**社区发现：**
- `louvain_communities`, `leiden_communities`

**连通组件：**
- `connected_components`, `is_connected`, `number_connected_components`
- `node_connected_component`
- `weakly_connected_components`, `is_weakly_connected`, `number_weakly_connected_components`

**聚类分析：**
- `average_clustering`, `clustering`, `transitivity`, `triangles`

**核心结构：**
- `core_number`, `k_truss`

**链接分析：**
- `pagerank`, `hits`

**链接预测：**
- `jaccard_coefficient`

**最短路径（23+ 函数）：**
- `shortest_path`, `shortest_path_length`
- `has_path`, `all_pairs_shortest_path`, `all_pairs_shortest_path_length`
- `dijkstra_path`, `dijkstra_path_length`, `all_pairs_dijkstra`, `all_pairs_dijkstra_path_length`
- `bellman_ford_path`, `bellman_ford_path_length`, `all_pairs_bellman_ford_path_length`
- `single_source_shortest_path`, `single_source_shortest_path_length`
- `single_source_dijkstra`, `single_source_dijkstra_path`, `single_source_dijkstra_path_length`
- `single_source_bellman_ford`, `single_source_bellman_ford_path`, `single_source_bellman_ford_path_length`
- `single_target_shortest_path_length`

**图遍历：**
- `bfs_edges`, `bfs_layers`, `bfs_predecessors`, `bfs_successors`, `bfs_tree`
- `generic_bfs_edges`, `descendants_at_distance`

**有向无环图：**
- `ancestors`, `descendants`

**二分图：**
- `betweenness_centrality` (二分图), `biadjacency_matrix`
- `complete_bipartite_graph`, `from_biadjacency_matrix`

**树结构：**
- `is_arborescence`, `is_branching`, `is_forest`, `is_tree`

**图操作：**
- `complement`, `reverse`

**互惠性：**
- `overall_reciprocity`, `reciprocity`

**孤立节点：**
- `is_isolate`, `isolates`, `number_of_isolates`

**最近公共祖先：**
- `lowest_common_ancestor`

**布局算法：**
- `forceatlas2_layout`

**图生成器：** 支持直接在 GPU 上创建图的各种生成器。

---

## 原生 cuGraph API

### 快速示例

```python
import cugraph
import cudf

# 从 cuDF DataFrame 加载边数据
edges = cudf.DataFrame({
    "source": [0, 1, 2, 3, 0, 2],
    "destination": [1, 2, 3, 4, 4, 1],
    "weight": [1.0, 2.0, 1.0, 3.0, 0.5, 1.5]
})

G = cugraph.Graph(directed=True)
G.from_cudf_edgelist(edges, source="source", destination="destination", edge_attr="weight")

# 执行算法
pr = cugraph.pagerank(G)
bc = cugraph.betweenness_centrality(G)
components = cugraph.weakly_connected_components(G)
```

---

## 图创建与数据加载

### 从 cuDF DataFrame（主要方法）
```python
import cudf, cugraph

df = cudf.DataFrame({"src": [0, 1, 2], "dst": [1, 2, 3], "wt": [1.0, 2.0, 3.0]})

# 无权重图
G = cugraph.Graph()
G.from_cudf_edgelist(df, source="src", destination="dst")

# 带权重图
G = cugraph.Graph()
G.from_cudf_edgelist(df, source="src", destination="dst", edge_attr="wt")

# 有向图
G = cugraph.Graph(directed=True)
G.from_cudf_edgelist(df, source="src", destination="dst")
```

### 从 Pandas DataFrame
```python
import pandas as pd, cugraph

df = pd.DataFrame({"src": [0, 1, 2], "dst": [1, 2, 3]})
G = cugraph.Graph()
G.from_pandas_edgelist(df, source="src", destination="dst")
```

### 从 cuDF 邻接表
```python
G = cugraph.Graph()
G.from_cudf_adjlist(offsets, indices, values)  # CSR 格式
```

### 从 NumPy 数组
```python
import numpy as np
adj_matrix = np.array([[0, 1, 0], [1, 0, 1], [0, 1, 0]])
G = cugraph.Graph()
G.from_numpy_array(adj_matrix)
```

### 从 Pandas 邻接矩阵
```python
G = cugraph.Graph()
G.from_pandas_adjacency(adj_df)
```

### 从 Dask-cuDF（多 GPU）
```python
G = cugraph.Graph()
G.from_dask_cudf_edgelist(dask_cudf_df, source="src", destination="dst")
```

### 从内置数据集
```python
from cugraph.datasets import karate, dolphins, polbooks, netscience
G = karate.get_graph()
```

### 对称化处理（无向图）
```python
# 确保所有边双向存在
sym_df = cugraph.symmetrize_df(df, "src", "dst")

# 或直接对称化图
sym_df = cugraph.symmetrize(source_col, dest_col, weight_col)
```

### 顶点重编号
cuGraph 内部将顶点重编号为从 0 开始的连续整数。使用 `unrenumber()` 映射回原始 ID：
```python
result = cugraph.pagerank(G)
result = G.unrenumber(result, "vertex")  # 将内部 ID 映射回原始 ID
```

---

## 支持的图类型

| 图类型 | cuGraph 类 | 说明 |
|---|---|---|
| **无向图** | `cugraph.Graph()` | 默认类型，边为双向 |
| **有向图** | `cugraph.Graph(directed=True)` | 有向边，部分算法需特定方向性 |
| **带权图** | 在 `from_cudf_edgelist` 中设置 `edge_attr` | 被 SSSP、PageRank、Louvain 等算法使用 |
| **多重图** | `cugraph.MultiGraph()` | 支持顶点间多条边 |
| **二分图** | 通过标准图结构支持 | 无专用类，算法位于 `cugraph.bipartite` |

**重要提示：** cuGraph 使用 CSR（压缩稀疏行）内部表示。图在创建后不可变——调用 `from_cudf_edgelist()` 后无法动态增删单条边。修改图需从新 DataFrame 重建。

---

## 算法目录

### 中心性分析

| 算法 | 单 GPU | 多 GPU | NetworkX 等效函数 |
|---|---|---|---|
| 介数中心性 | `cugraph.betweenness_centrality(G)` | `cugraph.dask.centrality.betweenness_centrality()` | `nx.betweenness_centrality()` |
| 边介数中心性 | `cugraph.edge_betweenness_centrality(G)` | `cugraph.dask.centrality.edge_betweenness_centrality()` | `nx.edge_betweenness_centrality()` |
| 度中心性 | `cugraph.degree_centrality(G)` | -- | `nx.degree_centrality()` |
| 特征向量中心性 | `cugraph.eigenvector_centrality(G)` | `cugraph.dask.centrality.eigenvector_centrality()` | `nx.eigenvector_centrality()` |
| Katz 中心性 | `cugraph.katz_centrality(G)` | `cugraph.dask.centrality.katz_centrality()` | `nx.katz_centrality()` |

### 社区发现

| 算法 | 单 GPU | 多 GPU | NetworkX 等效函数 |
|---|---|---|---|
| Louvain | `cugraph.louvain(G, max_level=, max_iter=, resolution=)` | `cugraph.dask.community.louvain.louvain()` | `nx.community.louvain_communities()` |
| Leiden | `cugraph.leiden(G, max_iter=, resolution=)` | `cugraph.dask.community.leiden.leiden()` | `nx.community.leiden_communities()` |
| ECG | `cugraph.ecg(G, min_weight=)` | `cugraph.dask.community.ecg.ecg()` | -- |
| 谱平衡切割 | `cugraph.spectralBalancedCutClustering(G, num_clusters)` | -- | -- |
| 谱模块化 | `cugraph.spectralModularityMaximizationClustering(G, num_clusters)` | -- | -- |
| 三角计数 | `cugraph.triangle_count(G)` | `cugraph.dask.community.triangle_count()` | `nx.triangles()` |
| K-桁架 | `cugraph.k_truss(G, k)` 或 `cugraph.ktruss_subgraph(G, k)` | `cugraph.dask.community.ktruss_subgraph()` | `nx.k_truss()` |
| EgoNet | `cugraph.ego_graph(G, n, radius=)` | `cugraph.dask.community.egonet()` | `nx.ego_graph()` |
| 诱导子图 | `cugraph.induced_subgraph(G, vertices)` | `cugraph.dask.community.induced_subgraph()` | `G.subgraph(vertices)` |

**聚类分析：**
- `cugraph.analyzeClustering_edge_cut(G, n_clusters, clustering)`
- `cugraph.analyzeClustering_modularity(G, n_clusters, clustering)`
- `cugraph.analyzeClustering_ratio_cut(G, n_clusters, clustering)`

### 图遍历

| 算法 | 单 GPU | 多 GPU | NetworkX 等效函数 |
|---|---|---|---|
| BFS | `cugraph.bfs(G, start=, depth_limit=)` | `cugraph.dask.traversal.bfs.bfs()` | `nx.bfs_edges()` |
| BFS 边遍历 | `cugraph.bfs_edges(G, source)` | -- | `nx.bfs_edges()` |
| 单源最短路径 | `cugraph.sssp(G, source=)` | `cugraph.dask.traversal.sssp.sssp()` | `nx.single_source_dijkstra()` |
| 最短路径 | `cugraph.shortest_path(G, source=)` | -- | `nx.shortest_path()` |
| 最短路径长度 | `cugraph.shortest_path_length(G, source, target=)` | -- | `nx.shortest_path_length()` |
| 过滤不可达节点 | `cugraph.filter_unreachable(df)` | -- | -- |

### 链接分析

| 算法 | 单 GPU | 多 GPU | NetworkX 等效函数 |
|---|---|---|---|
| PageRank | `cugraph.pagerank(G, alpha=)` | `cugraph.dask.link_analysis.p

| Jaccard | `cugraph.jaccard(G, vertex_pair=)` | -- | `nx.jaccard_coefficient()` |
| 余弦相似度 | `cugraph.cosine(G, vertex_pair=)` | -- | -- |
| Overlap | `cugraph.overlap(G, vertex_pair=)` | `cugraph.dask.link_prediction.overlap()` | -- |
| Sorensen | `cugraph.sorensen(G, vertex_pair=)` | `cugraph.dask.link_prediction.sorensen()` | -- |

**NetworkX兼容封装器：** `cugraph.jaccard_coefficient(G, ebunch)`, `cugraph.overlap_coefficient(G, ebunch)`, `cugraph.sorensen_coefficient(G, ebunch)`

### 连通分量

| 算法 | 单GPU | 多GPU | NetworkX等效函数 |
|---|---|---|---|
| 连通分量 | `cugraph.connected_components(G)` | -- | `nx.connected_components()` |
| 弱连通分量 | `cugraph.weakly_connected_components(G)` | `cugraph.dask.components.weakly_connected_components()` | `nx.weakly_connected_components()` |
| 强连通分量 | `cugraph.strongly_connected_components(G)` | -- | `nx.strongly_connected_components()` |

### 核心结构

| 算法 | 单GPU | 多GPU | NetworkX等效函数 |
|---|---|---|---|
| 核心数 | `cugraph.core_number(G, degree_type=)` | `cugraph.dask.cores.core_number()` | `nx.core_number()` |
| K核 | `cugraph.k_core(G, k=, core_number=)` | `cugraph.dask.cores.k_core()` | `nx.k_core()` |

### 采样

| 算法 | 单GPU | 多GPU | 说明 |
|---|---|---|---|
| 偏置随机游走 | `cugraph.biased_random_walks(G, start_vertices)` | `cugraph.dask.sampling.biased_random_walks()` | 加权/偏置遍历 |
| 均匀随机游走 | -- | `cugraph.dask.sampling.uniform_random_walks()` | 结果填充至最大路径长度 |
| 随机游走 | -- | `cugraph.dask.sampling.random_walks()` | 通用随机游走 |
| Node2Vec | -- | `cugraph.dask.sampling.node2vec_random_walks()` | Node2Vec采样框架 |
| 同质邻居采样 | `cugraph.homogeneous_neighbor_sample(G, start_vertices, fanout)` | -- | 可配置每跳采样数 |
| 异质邻居采样 | `cugraph.heterogeneous_neighbor_sample(G, ...)` | -- | 多类型节点/边图 |

### 布局

| 算法 | 单GPU | 多GPU | NetworkX等效函数 |
|---|---|---|---|
| Force Atlas 2 | `cugraph.force_atlas2(G)` | -- | `nx.forceatlas2_layout()` (通过nx-cugraph) |

### 树结构

| 算法 | 单GPU | 多GPU | NetworkX等效函数 |
|---|---|---|---|
| 最小生成树 | `cugraph.minimum_spanning_tree(G)` | -- | `nx.minimum_spanning_tree()` |
| 最大生成树 | `cugraph.maximum_spanning_tree(G)` | -- | `nx.maximum_spanning_tree()` |

### 线性分配

| 算法 | 单GPU | 多GPU |
|---|---|---|
| 匈牙利算法 | `cugraph.hungarian(G, workers, cost)` | -- |

### 工具函数

| 函数 | 用途 |
|---|---|
| `cugraph.symmetrize(src, dst, val)` | 使边双向化（用于无向图） |
| `cugraph.symmetrize_df(df, src, dst)` | 对称化DataFrame |
| `cugraph.symmetrize_ddf(ddf, src, dst)` | 对称化Dask DataFrame |
| `cugraph.NumberMap` | 将外部顶点ID映射至连续内部ID |
| `G.unrenumber(df, col)` | 将内部顶点ID映射回原始ID |

---

## 基于Dask的多GPU支持

cuGraph通过Dask支持多GPU计算，适用于超出单GPU内存或需要加速处理的大规模图。

### 初始化设置
```python
from dask.distributed import Client
from dask_cuda import LocalCUDACluster
import cugraph
import cugraph.dask as dask_cugraph
import dask_cudf

# 初始化多GPU集群
cluster = LocalCUDACluster()
client = Client(cluster)

# 加载分布式边列表
ddf = dask_cudf.read_csv("large_graph.csv", names=["src", "dst", "weight"])

# 创建分布式图
G = cugraph.Graph(directed=True)
G.from_dask_cudf_edgelist(ddf, source="src", destination="dst", edge_attr="weight")

# 运行多GPU算法
pr = dask_cugraph.pagerank(G)
components = dask_cugraph.weakly_connected_components(G)
```

### 支持多GPU的算法

以下算法提供基于Dask的多GPU实现：
- **中心性：** 介数中心性、边介数中心性、特征向量中心性、Katz中心性
- **社区发现：** Louvain、Leiden、ECG、K-Truss、三角计数、EgoNet、诱导子图
- **连通分量：** 弱连通分量
- **核心结构：** 核心数、K核
- **链接分析：** PageRank、HITS
- **链接预测：** Overlap、Sorensen
- **采样：** 随机游走、偏置随机游走、均匀随机游走、Node2Vec、邻居采样
- **遍历：** BFS、SSSP
- **工具：** 重编号、对称化、路径提取、两跳邻居、RMAT生成器

---

## GNN支持

### cugraph-pyg（PyTorch Geometric集成）

自25.06版本起，**cugraph-pyg成为推荐的GNN框架集成方案**（cuGraph-DGL已被移除）。

cugraph-pyg提供PyG核心接口的原生GPU加速实现：

- **GraphStore**：使用cuGraph的CSR表示实现GPU加速图存储
- **FeatureStore**：节点/边特征的GPU常驻存储
- **采样器/加载器**：支持可配置采样数的GPU加速邻居采样

```bash
uv add --extra-index-url=https://pypi.nvidia.com cugraph-pyg-cu12
```

**核心能力：**
- 异构图采样（多节点/边类型）
- 多GPU分布式采样
- 与PyG的`NeighborLoader`和训练循环直接集成
- 在PyG工作流中实现GPU加速的中心性、社区发现等分析

**代码库：** https://github.com/rapidsai/cugraph-gnn

### WholeGraph（面向GNN的分布式GPU内存）

WholeGraph通过**WholeMemory**抽象为大规模GNN训练提供分布式GPU内存管理。

```bash
uv add --extra-index-url=https://pypi.nvidia.com pylibwholegraph-cu12
```

**核心概念：**

- **WholeMemory**：跨多GPU的统一内存视图。每个GPU通过单一抽象访问整个内存空间，即使数据物理分布。
- **WholeMemory通信器**：定义协作GPU组，每GPU对应一个进程。
- **WholeMemory张量**：类似PyTorch张量但分布式；支持一维/二维数据，首维度跨GPU分区。
- **WholeMemory嵌入**：二维张量变体，内置缓存策略和稀疏优化器（SGD、Adam、RMSProp、AdaGrad）。

**内存模式：**
| 模式 | 描述 | 适用场景 |
|---|---|---|
| **连续模式** | 通过硬件点对点实现单一连续地址空间 | NVLink系统（如DGX） |
| **分块模式** | 每GPU数据块支持直接多指针访问 | 具备部分NVLink的多GPU系统 |
| **分布式模式** | 远程访问需显式通信 | 多节点集群 |

**存储位置：** 主机内存（固定）或设备/GPU内存。

**图存储：** CSR格式，ROW_INDEX和COL_INDEX作为WholeMemory张量，实现高效分布式图管理。

**缓存策略：** 设备缓存主机内存、本地缓存全局内存——对处理超出GPU内存的图至关重要。

**目标硬件：** NVLink系统（如DGX A100/H100服务器）以获得最佳性能。

### cuGraph-DGL（已弃用）

**cuGraph-DGL自25.06版本起已移除。** 用户应迁移至cugraph-pyg。cuGraph团队无计划在DGL生态继续开发。

---

## 性能特征与基准测试

### nx-cugraph基准测试（NetworkX后端）

**硬件：** Intel Xeon w9-3495X (56核), NVIDIA RTX 3090 (24GB), 251 GB RAM, CUDA 12.8

**测试数据集：**

| 数据集 | 节点数 | 边数 | 类型 |
|---|---|---|---|
| netscience | 1,461 | 5,484 | 小型 |
| amazon0302 | 262,111 | 1,234,877 | 中型 |
| cit-Patents | 3,774,768 | 16,518,948 | 大型 |
| soc-LiveJournal1 | 4,847,571 | 68,993,773 | 超大型 |

**加速比（GPU vs CPU NetworkX）：**

| 算法 | 中型图 | 大型图 | 超大型图 |
|---|---|---|---|
| `betweenness_centrality` (k=100) | ~20倍 | ~520倍 | ~300倍 |
| `katz_centrality` | ~100倍 | ~5,000倍 | ~24,768倍 |
| `average_clustering` | ~50倍 | ~1,000倍 | ~2,828倍 |
| `transitivity` | ~50倍 | ~1,000倍 | ~2,832倍 |
| `louvain_communities` | ~30倍 | ~273倍 | ~200倍 |
| `pagerank` | ~2倍 | ~50倍 | ~188倍 |
| `eigenvector_centrality` | ~7倍 | ~100倍 | ~376倍 |
| `k_truss` | ~8倍 | ~200倍 | ~540倍 |

**关键发现：** 加速比随图规模显著提升。小型图（<5K边）可能因GPU初始化开销抵消加速优势。对于>100K边的图，多数算法预期获得10-500倍以上加速。

**具体案例：** cit-Patents数据集介数中心性（370万节点，1650万边）：
- CPU NetworkX：7分41秒
- nx-cugraph GPU：5.32秒（约86倍加速）

### 通用性能指南

- **小型图（<10K边）：** GPU开销可能占主导，NetworkX CPU可能更快
- **中型图（100K-1M边）：** 典型加速10-100倍
- **大型图（1M-100M边）：** 典型加速100-1000倍以上
- **超大型图（>100M边）：** 使用多GPU，单GPU内存可能不足
- **首次调用开销：** GPU内核编译和图传输约增加1-3秒；同图后续调用显著加快

---

## 内存管理

### GPU内存考量

- cuGraph以CSR格式在GPU内存存储图数据
- 内存占用约：无权图`(边数*2*4字节)+(顶点数*4字节)`，加权图（float64权重）额外增加`(边数*8字节)`
- 1亿边图约需1.6GB（无权）或2.4GB（加权）
- 算法工作内存各异；部分算法（如介数中心性）需额外O(V)或O(E)临时空间

### 大规模图处理策略

1. **多GPU方案**：通过Dask处理超出单GPU内存的图
2. **WholeGraph方案**：需分布式特征/图存储的GNN工作负载
3. **使用`rmm`**（RAPIDS内存管理器）精细控制GPU内存：
   ```python
   import rmm
   rmm.reinitialize(pool_allocator=True, initial_pool_size=2**30)  # 1GB内存池
   ```
4. **内存监控**：通过`nvidia-smi`或`rmm.get_memory_info()`
5. **显式删除中间结果**：`del result; import gc; gc.collect()`

---

## 互操作性

### 与cuDF集成
cuGraph原生支持cuDF DataFrame输入输出。算法结果以含顶点/边列的cuDF DataFrame返回。

```python
import cudf, cugraph
# 从cuDF创建图
edges = cudf.read_csv("edges.csv")
G = cugraph.Graph()
G.from_cudf_edgelist(edges, source="src", destination="dst")

# 结果以cuDF DataFrame返回
pr = cugraph.pagerank(G)  # 含'vertex'和'pagerank'列的DataFrame
```

### 与cuML集成
将图分析结果输入cuML进行下游机器学习：
```python
import cuml
# 使用图嵌入（如Node2Vec生成）作为cuML特征
# 或使用社区标签作为分类特征
louvain_result = cugraph.louvain(G)
# 将分区标签输入cuML模型
```

### 与CuPy/SciPy集成
```python
# cuGraph支持CuPy和SciPy稀疏矩阵作为输入
import cupy, scipy
```

### 与NetworkX集成
```python
import networkx as nx
import cugraph

# NetworkX转cuGraph
G_nx = nx.karate_club_graph()
G_cu = cugraph.from_networkx(G_nx)  # 部分版本尚未支持

# 或使用nx-cugraph后端透明加速
```

### 与PyTorch Geometric集成
```python
# 通过cugraph-pyg（见GNN支持章节）
from cugraph_pyg.data import CuGraphStore
from cugraph_pyg.loader import CuGraphNeighborLoader
```

### 与Pandas集成
```python
import pandas as pd
df = pd.DataFrame({"src": [0, 1, 2], "dst": [1, 2, 3]})
G = cugraph.Graph()
G.from_pandas_edgelist(df, source="src", destination="dst")
```

---

## 相比NetworkX的已知限制

1. **图不可变**：创建后无法增删单条边，需从DataFrame重建
2. **无图对象属性**：cuGraph仅存储结构，节点/边属性需单独维护（如通过cuDF）。nx-cugraph后端透明处理属性映射
3. **顶点类型**：顶点必须为整数（或内部重编号为整数）。字符串ID自动重编号
4. **非全算法支持**：查看nx-cugraph支持算法列表，未支持调用回退至CPU NetworkX
5. **数值精度**：GPU浮点结果因并行归约顺序可能与CPU结果略有差异
6. **无动态图**：cuGraph设计用于静态图分析，不支持流式/动态图更新
7. **强连通分量**：仅支持单GPU（无多GPU Dask版本）
8. **谱聚类**：仅支持单GPU
9. **最小/最大生成树**：仅支持单GPU
10. **Force Atlas 2布局**：仅支持单GPU
11. **兼容性文档**：26.02版本官方文档中NetworkX兼容性列为"即将推出"

---

## 常见迁移模式

### NetworkX到nx-cugraph（零修改）
```python
# 迁移前（CPU）：
import networkx as nx
G = nx.from_pandas_edgelist(df, "src", "dst")
pr = nx.pagerank(G)

# 迁移后（GPU，无需改代码）：
# 设置环境变量：NX_CUGRAPH_AUTOCONFIG=True
# 相同代码自动在GPU运行

parts, modularity = cugraph.louvain(G, resolution=1.0)
```

### 从 Pandas 到 cuDF + cuGraph 的流程
```python
# 之前：
import pandas as pd
import networkx as nx
df = pd.read_csv("edges.csv")
G = nx.from_pandas_edgelist(df, "source", "target", "weight")
result = nx.pagerank(G)

# 之后：
import cudf
import cugraph
df = cudf.read_csv("edges.csv")
G = cugraph.Graph()
G.from_cudf_edgelist(df, source="source", destination="target", edge_attr="weight")
result = cugraph.pagerank(G)
```

### 为现有 cuGraph 代码添加多 GPU 支持
```python
# 之前（单 GPU）：
import cugraph
G = cugraph.Graph()
G.from_cudf_edgelist(edges, source="src", destination="dst")
result = cugraph.pagerank(G)

# 之后（多 GPU）：
from dask.distributed import Client
from dask_cuda import LocalCUDACluster
import cugraph, cugraph.dask as dcg
import dask_cudf

cluster = LocalCUDACluster()
client = Client(cluster)

ddf = dask_cudf.from_cudf(edges, npartitions=len(cluster.workers))
G = cugraph.Graph()
G.from_dask_cudf_edgelist(ddf, source="src", destination="dst")
result = dcg.pagerank(G)
result_local = result.compute()  # 收集到单个 GPU
```
