# 链接预测 — 完整参考

链接预测是预测图中缺失或未来边的任务。常见应用：社交网络好友推荐、知识图谱补全、药物-靶点相互作用。

## 边分割

使用 `RandomLinkSplit` 将边分割为训练/验证/测试集，同时保持图结构：

```python
import torch_geometric.transforms as T

transform = T.RandomLinkSplit(
    num_val=0.1,              # 10%的边用于验证
    num_test=0.1,             # 10%的边用于测试
    is_undirected=True,       # 无向图设为True
    add_negative_train_samples=False,  # 训练时动态生成负样本
    neg_sampling_ratio=1.0,   # 每个正样本边对应1个负样本
)
train_data, val_data, test_data = transform(data)
```

分割后每个子集包含：
- `edge_index`：消息传递边（仅训练边 — 避免数据泄露）
- `edge_label_index`：监督边 `[2, num_supervision_edges]` — 待预测的边
- `edge_label`：二元标签 — 1表示正样本（真实边），0表示负样本（虚假边）

当 `add_negative_train_samples=False` 时，训练集的 `edge_label_index` 仅含正样本边，负样本在训练时动态采样。验证/测试集始终包含正负样本边。

## 编码器-解码器模式

标准流程：
1. **编码** — 使用GNN通过消息传递边生成节点嵌入
2. **解码** — 利用节点嵌入对候选边进行评分

```python
import torch
import torch.nn.functional as F
from torch_geometric.nn import GCNConv

class LinkEncoder(torch.nn.Module):
    def __init__(self, in_channels, hidden_channels, out_channels):
        super().__init__()
        self.conv1 = GCNConv(in_channels, hidden_channels)
        self.conv2 = GCNConv(hidden_channels, out_channels)

    def forward(self, x, edge_index):
        x = self.conv1(x, edge_index).relu()
        x = self.conv2(x, edge_index)
        return x

def decode(z, edge_label_index):
    """点积解码器：每条边的得分 = z_src . z_dst"""
    src, dst = edge_label_index
    return (z[src] * z[dst]).sum(dim=1)
```

## 全批次训练循环

```python
from torch_geometric.utils import negative_sampling

model = LinkEncoder(data.num_features, 128, 64)
optimizer = torch.optim.Adam(model.parameters(), lr=0.01)

def train(train_data):
    model.train()
    optimizer.zero_grad()

    # 仅使用消息传递边进行编码
    z = model(train_data.x, train_data.edge_index)

    # 为当前批次采样负样本边
    neg_edge_index = negative_sampling(
        edge_index=train_data.edge_index,
        num_nodes=train_data.num_nodes,
        num_neg_samples=train_data.edge_label_index.size(1),
    )

    # 合并正负监督边
    edge_label_index = torch.cat([train_data.edge_label_index, neg_edge_index], dim=1)
    edge_label = torch.cat([
        torch.ones(train_data.edge_label_index.size(1)),
        torch.zeros(neg_edge_index.size(1)),
    ])

    # 解码并计算损失
    pred = decode(z, edge_label_index)
    loss = F.binary_cross_entropy_with_logits(pred, edge_label)
    loss.backward()
    optimizer.step()
    return loss.item()

@torch.no_grad()
def test(data_split):
    model.eval()
    z = model(data_split.x, data_split.edge_index)
    pred = decode(z, data_split.edge_label_index).sigmoid()
    # AUC是链接预测的标准指标
    from sklearn.metrics import roc_auc_score
    return roc_auc_score(data_split.edge_label.cpu(), pred.cpu())
```

## 图自编码器 (GAE / VGAE)

PyG提供 `GAE` 和 `VGAE` 用于无监督链接预测：

```python
from torch_geometric.nn import GAE, VGAE, GCNConv

class Encoder(torch.nn.Module):
    def __init__(self, in_channels, out_channels):
        super().__init__()
        self.conv1 = GCNConv(in_channels, 2 * out_channels)
        self.conv2 = GCNConv(2 * out_channels, out_channels)
        # VGAE需额外定义conv_mu和conv_logstd

    def forward(self, x, edge_index):
        x = self.conv1(x, edge_index).relu()
        return self.conv2(x, edge_index)

# GAE封装编码器并提供训练/测试方法
model = GAE(Encoder(data.num_features, 64))
optimizer = torch.optim.Adam(model.parameters(), lr=0.01)

def train():
    model.train()
    optimizer.zero_grad()
    z = model.encode(train_data.x, train_data.edge_index)
    loss = model.recon_loss(z, train_data.edge_label_index)
    # VGAE需添加KL散度：
    # loss = loss + (1 / data.num_nodes) * model.kl_loss()
    loss.backward()
    optimizer.step()
    return loss.item()

@torch.no_grad()
def test(data_split):
    model.eval()
    z = model.encode(data_split.x, data_split.edge_index)
    return model.test(z, data_split.edge_label_index[0],  # 正样本边
                         data_split.edge_label_index[1])   # 负样本边
```

VGAE编码器需返回 `mu` 和 `logstd` 而非单一嵌入。使用VGAE专用编码模式：

```python
class VariationalEncoder(torch.nn.Module):
    def __init__(self, in_channels, out_channels):
        super().__init__()
        self.conv1 = GCNConv(in_channels, 2 * out_channels)
        self.conv_mu = GCNConv(2 * out_channels, out_channels)
        self.conv_logstd = GCNConv(2 * out_channels, out_channels)

    def forward(self, x, edge_index):
        x = self.conv1(x, edge_index).relu()
        return self.conv_mu(x, edge_index), self.conv_logstd(x, edge_index)

model = VGAE(VariationalEncoder(data.num_features, 64))
```

## 使用 LinkNeighborLoader 进行小批次链接预测

大型图可使用 `LinkNeighborLoader` — 它在监督边周围采样子图：

```python
from torch_geometric.loader import LinkNeighborLoader

train_loader = LinkNeighborLoader(
    data=train_data,
    num_neighbors=[20, 10],         # 每跳采样邻居数
    edge_label_index=train_data.edge_label_index,
    edge_label=train_data.edge_label,
    batch_size=128,                  # 每批次监督边数量
    neg_sampling_ratio=1.0,          # 正负样本1:1
    shuffle=True,
)

for batch in train_loader:
    # batch.edge_label_index: 监督边（正+负）
    # batch.edge_label: 1为正样本，0为负样本
    # batch.edge_index: 消息传递边（来自邻居采样）
    z = model(batch.x, batch.edge_index)
    pred = decode(z, batch.edge_label_index)
    loss = F.binary_cross_entropy_with_logits(pred, batch.edge_label)
```

## 异构图链接预测

异构图场景（如用户-物品推荐）：

```python
transform = T.RandomLinkSplit(
    num_val=0.1,
    num_test=0.1,
    neg_sampling_ratio=1.0,
    add_negative_train_samples=False,
    edge_types=('user', 'rates', 'movie'),              # 待预测的边类型
    rev_edge_types=('movie', 'rev_rates', 'user'),       # 对应的反向边
)
train_data, val_data, test_data = transform(data)

# 监督边位于：
# train_data['user', 'rates', 'movie'].edge_label_index
# train_data['user', 'rates', 'movie'].edge_label
```

## 评估指标

- **AUC-ROC**：标准指标 — ROC曲线下面积
- **平均精度 (AP)**：精确率-召回率曲线下面积
- **Hits@K**：正样本边排名前K的比例（知识图谱常用）
- **MRR**：正样本边的平均倒数排名

```python
from sklearn.metrics import roc_auc_score, average_precision_score

auc = roc_auc_score(edge_label.cpu(), pred.cpu())
ap = average_precision_score(edge_label.cpu(), pred.cpu())
```

## 常见陷阱

1. **数据泄露**：训练时切勿在消息传递图中包含验证/测试边。`RandomLinkSplit` 已正确处理 — `train_data` 中的 `edge_index` 仅含训练边。
2. **负采样质量**：随机负样本虽标准但过于简单。可使用二跳邻居生成更难负样本。
3. **无向图处理**：在 `RandomLinkSplit` 中设置 `is_undirected=True` — 否则会独立处理方向导致信息泄露。
4. **解码策略**：点积解码最简单但非最优。异构图/知识图谱可考虑MLP解码器或DistMult。
