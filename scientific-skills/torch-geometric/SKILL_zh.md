---
name: torch-geometric
description: "关于使用 PyTorch Geometric (PyG) 构建图神经网络的指南。当用户询问图神经网络、GNN、节点分类、链接预测、图分类、消息传递网络、异构图、邻居采样或任何涉及 torch_geometric/PyG 的任务时使用此技能。当看到导入 torch_geometric 模块、用户提及图卷积（GCN/GAT/GraphSAGE/GIN）、图数据结构或处理关系/网络数据时也应触发。即使用户仅提到'图学习'或'几何深度学习'，也使用此技能。"
---

# PyTorch Geometric (PyG)

PyG 是基于 PyTorch 构建的图神经网络标准库，提供图数据结构、60+ 种 GNN 层实现、可扩展的小批量训练以及异构图支持。

安装：`uv add torch_geometric`（或 `uv pip install torch_geometric`；需预装 PyTorch）。可选加速组件：`pyg-lib`、`torch-scatter`、`torch-sparse`、`torch-cluster`。

## 核心概念

### 图数据：`Data` 与 `HeteroData`

图存储在 `Data` 对象中，关键属性如下：

```python
from torch_geometric.data import Data

data = Data(
    x=节点特征,          # [节点数, 节点特征维度]
    edge_index=边索引,   # [2, 边数量] — COO格式, dtype=torch.long
    edge_attr=边特征,    # [边数量, 边特征维度]
    y=标签,             # 节点级 [节点数, *] 或图级 [1, *]
    pos=位置坐标,        # [节点数, 空间维度] (点云/空间数据)
)
```

**`edge_index` 格式至关重要**：它是 `[2, 边数量]` 张量，其中 `edge_index[0]`=源节点，`edge_index[1]`=目标节点。注意：这不是元组列表。若边对按行存储，需转置并调用 `.contiguous()`：

```python
# 若边格式为 [[源1, 目标1], [源2, 目标2], ...] — 先转置：
edge_index = edge_pairs.t().contiguous()
```

无向图需包含双向边：边 (0,1) 需在 edge_index 中同时存在 `[0,1]` 和 `[1,0]`。

异构图请使用 `HeteroData` — 详见下文异构图章节。

### 数据集

PyG 内置自动下载与预处理的常用数据集：

```python
from torch_geometric.datasets import Planetoid, TUDataset

# 单图节点分类 (Cora, Citeseer, Pubmed)
dataset = Planetoid(root='./data', name='Cora')
data = dataset[0]  # 含 train/val/test 掩码的单图

# 多图分类 (ENZYMES, MUTAG, IMDB-BINARY 等)
dataset = TUDataset(root='./data', name='ENZYMES')
# dataset[0], dataset[1]... 为独立图
```

按任务分类的常用数据集：
- **节点分类**：Planetoid (Cora/Citeseer/Pubmed), OGB (ogbn-arxiv/ogbn-products/ogbn-mag)
- **图分类**：TUDataset (MUTAG/ENZYMES/PROTEINS/IMDB-BINARY), OGB (ogbg-molhiv)
- **链接预测**：OGB (ogbl-collab/ogbl-citation2)
- **分子数据**：QM7/QM9/MoleculeNet
- **点云/网格**：ShapeNet/ModelNet10/40/FAUST

### 数据变换

变换用于预处理或增强图数据，类似 torchvision 的 transforms：

```python
import torch_geometric.transforms as T

# 常用变换
T.NormalizeFeatures()    # 行归一化节点特征（总和为1）
T.ToUndirected()         # 添加反向边使图无向化
T.AddSelfLoops()         # 添加自循环边
T.KNNGraph(k=6)          # 从点云位置构建k近邻图
T.RandomJitter(0.01)     # 位置随机扰动增强
T.Compose([...])         # 组合多个变换

# 作为 pre_transform（单次执行，保存到磁盘）或 transform（每次访问执行）
dataset = ShapeNet(root='./data', pre_transform=T.KNNGraph(k=6),
                   transform=T.RandomJitter(0.01))
```

## 构建 GNN 模型

### 快速入门：使用内置层

最快捷方式 — 堆叠 `torch_geometric.nn` 中的卷积层：

```python
import torch
import torch.nn.functional as F
from torch_geometric.nn import GCNConv

class GCN(torch.nn.Module):
    def __init__(self, 输入通道, 隐藏通道, 输出通道):
        super().__init__()
        self.conv1 = GCNConv(输入通道, 隐藏通道)
        self.conv2 = GCNConv(隐藏通道, 输出通道)

    def forward(self, x, edge_index):
        x = self.conv1(x, edge_index).relu()
        x = F.dropout(x, p=0.5, training=self.training)
        x = self.conv2(x, edge_index)
        return x
```

**重要**：PyG 卷积层不包含激活函数 — 需在每层后手动添加。此设计为保持灵活性。

### 选择卷积层

根据任务和图结构选择：

| 层 | 适用场景 | 核心思想 |
|-------|----------|----------|
| `GCNConv` | 同构图/半监督节点分类 | 谱图理论启发，度归一化聚合 |
| `GATConv` / `GATv2Conv` | 邻居重要性差异大时 | 注意力加权消息传递 |
| `SAGEConv` | 大图/归纳式场景 | 支持采样，可学习聚合 |
| `GINConv` | 图分类/最大化表达能力 | 与WL测试同等强大 |
| `TransformerConv` | 丰富边特征/复杂交互 | 含边特征的多头注意力 |
| `EdgeConv` | 点云/动态图 | 基于边特征 (x_i, x_j - x_i) 的MLP |
| `RGCNConv` | 多关系类型异构图 | 关系特定权重矩阵 |
| `HGTConv` | 异构图 | 类型特定注意力机制 |

所有卷积层至少接受 `(x, edge_index)`。多数支持 `edge_attr` 处理边特征。

### 惰性初始化

使用 `-1` 作为输入通道数可让 PyG 自动推断维度 — 尤其适用于异构图模型：

```python
conv = SAGEConv((-1, -1), 64)  # 输入维度在首次前向传播时推断
# 惰性模块初始化：
with torch.no_grad():
    out = model(data.x, data.edge_index)
```

### 高层模型 API

PyG 为常见架构提供开箱即用的模型类：

```python
from torch_geometric.nn import GraphSAGE, GCN, GAT, GIN

model = GraphSAGE(
    in_channels=dataset.num_features,
    hidden_channels=64,
    out_channels=dataset.num_classes,
    num_layers=2,
)
```

### 通过 MessagePassing 自定义层

实现新型 GNN 层需继承 `MessagePassing`，框架如下：

1. `propagate()` 协调消息传递
2. `message()` 定义沿边传递的信息（phi函数）
3. `aggregate()` 聚合节点消息（sum/mean/max）
4. `update()` 转换聚合结果（gamma函数）

```python
from torch_geometric.nn import MessagePassing
from torch_geometric.utils import add_self_loops, degree

class MyConv(MessagePassing):
    def __init__(self, 输入通道, 输出通道):
        super().__init__(aggr='add')  # "add"/"mean"/"max"
        self.lin = torch.nn.Linear(输入通道, 输出通道)

    def forward(self, x, edge_index):
        # 消息传递前预处理
        x = self.lin(x)
        # 启动消息传递
        return self.propagate(edge_index, x=x)

    def message(self, x_j):
        # x_j: 每条边源节点特征 [边数量, 特征维度]
        # _j 后缀自动索引源节点，_i 索引目标节点
        return x_j
```

**`_i` / `_j` 命名约定**：传递给 `propagate()` 的张量可通过在 `message()` 签名中添加 `_i`（目标/中心节点）或 `_j`（源/邻居节点）自动索引。例如传递 `x=...` 后，可在 message() 中访问 `x_i` 和 `x_j`。

完整实现示例见 `references/message_passing.md` 中的 GCN 和 EdgeConv。

## 任务特定模式

### 节点分类

```python
# 单图全批量训练 (如 Cora)
model.train()
for epoch in range(200):
    optimizer.zero_grad()
    out = model(data.x, data.edge_index)
    loss = F.cross_entropy(out[data.train_mask], data.y[data.train_mask])
    loss.backward()
    optimizer.step()

# 评估
model.eval()
pred = model(data.x, data.edge_index).argmax(dim=1)
acc = (pred[data.test_mask] == data.y[data.test_mask]).float().mean()
```

### 图分类

多图场景 — 使用 `DataLoader` 小批量加载，通过全局池化获取图级表示：

```python
from torch_geometric.loader import DataLoader
from torch_geometric.nn import GCNConv, global_mean_pool

loader = DataLoader(dataset, batch_size=32, shuffle=True)

class GraphClassifier(torch.nn.Module):
    def __init__(self, 输入通道, 隐藏通道, 输出通道):
        super().__init__()
        self.conv1 = GCNConv(输入通道, 隐藏通道)
        self.conv2 = GCNConv(隐藏通道, 隐藏通道)
        self.lin = torch.nn.Linear(隐藏通道, 输出通道)

    def forward(self, x, edge_index, batch):
        x = self.conv1(x, edge_index).relu()
        x = self.conv2(x, edge_index).relu()
        x = global_mean_pool(x, batch)  # [批次图数量, 隐藏通道]
        return self.lin(x)

# 训练循环
for data in loader:
    out = model(data.x, data.edge_index, data.batch)
    loss = F.cross_entropy(out, data.y)
```

PyG 的 `DataLoader` 通过创建块对角邻接矩阵批量处理多图。`batch` 张量将节点映射到所属图索引。池化操作（`global_mean_pool`/`global_max_pool`/`global_add_pool`）据此聚合图级特征。

### 链接预测

划分边为训练/验证/测试集，使用负采样：

```python
from torch_geometric.transforms import RandomLinkSplit

transform = RandomLinkSplit(
    num_val=0.1,
    num_test=0.1,
    is_undirected=True,
    add_negative_train_samples=False,
)
train_data, val_data, test_data = transform(data)

# 编码节点后计算边分数
z = model.encode(train_data.x, train_data.edge_index)
# 正边分数
pos_score = (z[train_data.edge_label_index[0]] * z[train_data.edge_label_index[1]]).sum(dim=1)
```

完整指南见 `references/link_prediction.md`：GAE/VGAE 自编码器、完整训练循环、大图专用 LinkNeighborLoader、异构图链接预测及评估指标。

## 大图扩展

当图数据超出 GPU 显存时，使用 `NeighborLoader` 进行邻居采样：

```python
from torch_geometric.loader import NeighborLoader

train_loader = NeighborLoader(
    data,
    num_neighbors=[15, 10],     # 第一跳采样15邻居，第二跳10邻居
    batch_size=128,              # 每批种子节点数
    input_nodes=data.train_mask, # 采样源节点
    shuffle=True,
)

for batch in train_loader:
    batch = batch.to(device)
    out = model(batch.x, batch.edge_index)
    # 仅使用前 batch_size 个节点计算损失（种子节点）
    loss = F.cross_entropy(out[:batch.batch_size], batch.y[:batch.batch_size])
```

**NeighborLoader 要点**：
- `num_neighbors` 列表长度需匹配 GNN 深度（消息传递层数）
- 输出中前 `batch.batch_size` 个节点始终为种子节点
- `batch.n_id` 将重标索引映射回原始节点ID
- 支持 `Data` 和 `HeteroData`
- 链接预测需改用 `LinkNeighborLoader`
- 超过2-3跳的采样通常不可行（指数级膨胀）

其他扩展方案：`ClusterLoader` (ClusterGCN)、`GraphSAINTSampler`、`ShaDowKHopSampler`。多GPU训练、DDP、PyTorch Lightning 集成及 `torch.compile` 支持详见 `references/scaling.md`。

## 异构图

处理多节点/边类型图（社交网络/知识图谱/推荐系统）：

```python
from torch_geometric.data import HeteroData

data = HeteroData()

# 节点特征 — 按节点类型字符串索引
data['user'].x = torch.randn(1000, 64)
data['movie'].x = torch.randn(500, 128)

# 边索引 — 按 (源类型, 边类型, 目标类型) 三元组索引
data['user', 'rates', 'movie'].edge_index = torch.randint(0, 500, (2, 3000))
data['user', 'follows', 'user'].edge_index = torch.randint(0, 1000, (2, 5000))

# 访问便捷字典
data.x_dict        # {'user': 张量, 'movie': 张量}
data.edge_index_dict  # {('user','rates','movie'): 张量, ...}
data.metadata()    # ([节点类型列表], [边类型列表])
```

### 异构图 GNN 三种构建方式

**1. `to_hetero()` 自动转换** — 先写同构图模型，再自动转换：

```python
from torch_geometric.nn import SAGEConv, to_hetero

class GNN(torch.nn.Module):
    def __init__(self, 隐藏通道, 输出通道):
        super().__init__()
        self.conv1 = SAGEConv((-1, -1), 隐藏通道)
        self.conv2 = SAGEConv((-1, -1), 输出通道)

    def forward(self, x, edge_index):
        x = self.conv1(x, edge_index).relu()
        x = self.conv2(x, edge_index)
        return x

model = GNN(64, dataset.num_classes)
model = to_hetero(model, data.metadata(), aggr='sum')

# 转换后接受字典输入：
out = model(data.x_dict, data.edge_index_dict)
```

使用 `(-1, -1)` 处理二分图输入（源/目标维度可能不同）。惰性初始化自动处理维度推断。

**2. `HeteroConv` 包装器** — 为每种边类型指定不同卷积：

```python
from torch_geometric.nn import HeteroConv, GCNConv, SAGEConv, GATConv

conv = HeteroConv({
    ('paper', 'cites', 'paper'): GCNConv(-1, 64),
    ('author', 'writes', 'paper'): SAGEConv((-1, -1), 64),
    ('paper', 'rev_writes', 'author'): GATConv((-1, -1), 64, add_self_loops=False),
}, aggr='sum')
```

**3. 原生异构算子** 如 `

- **来自 NetworkX**：`from_networkx(G)` 直接转换 NetworkX 图
- **来自 scipy sparse**：`from_scipy_sparse_matrix(adj)` 提取 edge_index

阅读 `references/custom_datasets.md` 查看完整示例，包括所有模式、使用编码器的 CSV 加载以及 MovieLens 教程。

## 可解释性

PyG 提供 `torch_geometric.explain` 用于解释 GNN 预测：

```python
from torch_geometric.explain import Explainer, GNNExplainer

explainer = Explainer(
    model=model,
    algorithm=GNNExplainer(epochs=200),
    explanation_type='model',
    node_mask_type='attributes',
    edge_mask_type='object',
    model_config=dict(
        mode='multiclass_classification',
        task_level='node',
        return_type='log_probs',
    ),
)

explanation = explainer(data.x, data.edge_index, index=10)
explanation.visualize_graph()           # 重要子图
explanation.visualize_feature_importance(top_k=10)  # 特征重要性
```

可用算法：`GNNExplainer`（基于优化）、`PGExplainer`（参数化训练）、`CaptumExplainer`（通过 Captum 的梯度方法）、`AttentionExplainer`（注意力权重）。适用于同构图和异构图。

阅读 `references/explainability.md` 查看所有算法、异构图解释、评估指标及 PGExplainer 训练。

## 常见陷阱

1. **edge_index 形状**：必须是 `[2, num_edges]` 而非 `[num_edges, 2]`，必要时进行转置。
2. **忘记激活函数**：卷积层不含 ReLU 等激活函数——需手动添加。
3. **异构图二分中的自环**：当源节点与目标节点类型不同时，勿用 `add_self_loops=True`，改用跳跃连接。
4. **NeighborLoader 切片**：仅前 `batch.batch_size` 个节点是种子节点，需相应切片预测和标签。
5. **无向图处理**：若为无向图，需在 `edge_index` 包含双向边，或使用 `T.ToUndirected()`。
6. **延迟初始化**：含 `-1` 输入通道的模型需在训练前用 `torch.no_grad()` 执行一次前向传播以初始化参数。
7. **图任务的全局池化**：使用 `global_mean_pool(x, batch)`（而非手动重塑）聚合节点特征至图级别。
8. **num_neighbors 对齐**：保持 `len(num_neighbors)` 与 GNN 层数一致。跳数多于层数浪费算力；少于则浪费模型容量。
