# 异构图学习 — 完整参考

## 创建 HeteroData

```python
from torch_geometric.data import HeteroData

data = HeteroData()

# 节点特征 — 按节点类型字符串索引
data['paper'].x = ...       # [论文数量, 论文特征维度]
data['author'].x = ...      # [作者数量, 作者特征维度]
data['institution'].x = ... # [机构数量, 机构特征维度]

# 边索引 — 按(源类型, 边类型, 目标类型)三元组索引
data['paper', 'cites', 'paper'].edge_index = ...              # [2, 边数量]
data['author', 'writes', 'paper'].edge_index = ...            # [2, 边数量]
data['author', 'affiliated_with', 'institution'].edge_index = ... # [2, 边数量]

# 边特征（可选）
data['paper', 'cites', 'paper'].edge_attr = ...  # [边数量, 边特征维度]

# 附加节点属性
data['paper'].y = ...           # 标签
data['paper'].train_mask = ...  # 布尔掩码
```

### 数据访问

```python
# 单存储访问
data['paper']                          # 论文节点存储
data['paper', 'cites', 'paper']       # 引用边存储
data['paper', 'paper']                 # 当边类型明确时的简写
data['cites']                          # 当边类型名称唯一时的简写

# 字典访问（模型输入）
data.x_dict                            # {'paper': 张量, 'author': 张量, ...}
data.edge_index_dict                   # {('paper','cites','paper'): 张量, ...}
data.edge_attr_dict

# 元数据
node_types, edge_types = data.metadata()

# 修改
data['paper'].year = ...               # 添加新属性
del data['field_of_study']             # 删除节点类型
del data['has_topic']                  # 删除边类型

# 转换
data.to('cuda:0')                      # 转移到GPU
data.to_homogeneous()                  # 转换为带类型的同构图
```

### HeteroData 变换

```python
import torch_geometric.transforms as T

data = T.ToUndirected()(data)       # 添加反向边类型
data = T.AddSelfLoops()(data)       # 为同类型节点添加自环
data = T.NormalizeFeatures()(data)  # 跨所有类型归一化特征
```

`ToUndirected()` 很重要 — 它会创建反向边类型（例如 `('paper', 'rev_writes', 'author')`），使消息能双向流动。

## 构建异构图神经网络模型

### 方案1：使用 `to_hetero()` 自动转换

编写标准同质GNN后转换：

```python
from torch_geometric.nn import SAGEConv, to_hetero
import torch_geometric.transforms as T
from torch_geometric.datasets import OGB_MAG

dataset = OGB_MAG(root='./data', preprocess='metapath2vec', transform=T.ToUndirected())
data = dataset[0]

class GNN(torch.nn.Module):
    def __init__(self, hidden_channels, out_channels):
        super().__init__()
        # 使用(-1, -1)进行二分图支持的惰性初始化
        self.conv1 = SAGEConv((-1, -1), hidden_channels)
        self.conv2 = SAGEConv((-1, -1), out_channels)

    def forward(self, x, edge_index):
        x = self.conv1(x, edge_index).relu()
        x = self.conv2(x, edge_index)
        return x

model = GNN(64, dataset.num_classes)
model = to_hetero(model, data.metadata(), aggr='sum')

# 初始化惰性模块
with torch.no_grad():
    out = model(data.x_dict, data.edge_index_dict)
```

带跳跃连接（对基于注意力的模型很重要）：

```python
from torch_geometric.nn import GATConv, Linear, to_hetero

class GAT(torch.nn.Module):
    def __init__(self, hidden_channels, out_channels):
        super().__init__()
        self.conv1 = GATConv((-1, -1), hidden_channels, add_self_loops=False)
        self.lin1 = Linear(-1, hidden_channels)
        self.conv2 = GATConv((-1, -1), out_channels, add_self_loops=False)
        self.lin2 = Linear(-1, out_channels)

    def forward(self, x, edge_index):
        # 跳跃连接替代二分图消息传递中的自环
        x = self.conv1(x, edge_index) + self.lin1(x)
        x = x.relu()
        x = self.conv2(x, edge_index) + self.lin2(x)
        return x

model = GAT(64, dataset.num_classes)
model = to_hetero(model, data.metadata(), aggr='sum')
```

### 方案2：HeteroConv 包装器（每种边类型不同卷积）

```python
from torch_geometric.nn import HeteroConv, GCNConv, SAGEConv, GATConv, Linear

class HeteroGNN(torch.nn.Module):
    def __init__(self, hidden_channels, out_channels, num_layers):
        super().__init__()

        self.convs = torch.nn.ModuleList()
        for _ in range(num_layers):
            conv = HeteroConv({
                ('paper', 'cites', 'paper'): GCNConv(-1, hidden_channels),
                ('author', 'writes', 'paper'): SAGEConv((-1, -1), hidden_channels),
                ('paper', 'rev_writes', 'author'): GATConv((-1, -1), hidden_channels,
                                                            add_self_loops=False),
            }, aggr='sum')
            self.convs.append(conv)

        self.lin = Linear(hidden_channels, out_channels)

    def forward(self, x_dict, edge_index_dict):
        for conv in self.convs:
            x_dict = conv(x_dict, edge_index_dict)
            x_dict = {key: x.relu() for key, x in x_dict.items()}
        return self.lin(x_dict['paper'])

model = HeteroGNN(64, dataset.num_classes, num_layers=2)
with torch.no_grad():
    out = model(data.x_dict, data.edge_index_dict)
```

### 方案3：HGTConv（原生异构算子）

```python
from torch_geometric.nn import HGTConv, Linear

class HGT(torch.nn.Module):
    def __init__(self, hidden_channels, out_channels, num_heads, num_layers):
        super().__init__()

        self.lin_dict = torch.nn.ModuleDict()
        for node_type in data.node_types:
            self.lin_dict[node_type] = Linear(-1, hidden_channels)

        self.convs = torch.nn.ModuleList()
        for _ in range(num_layers):
            conv = HGTConv(hidden_channels, hidden_channels, data.metadata(),
                           num_heads, group='sum')
            self.convs.append(conv)

        self.lin = Linear(hidden_channels, out_channels)

    def forward(self, x_dict, edge_index_dict):
        for node_type, x in x_dict.items():
            x_dict[node_type] = self.lin_dict[node_type](x).relu_()
        for conv in self.convs:
            x_dict = conv(x_dict, edge_index_dict)
        return self.lin(x_dict['paper'])
```

## 使用 HeteroData 训练

### 全批次训练

```python
def train():
    model.train()
    optimizer.zero_grad()
    out = model(data.x_dict, data.edge_index_dict)
    mask = data['paper'].train_mask
    loss = F.cross_entropy(out['paper'][mask], data['paper'].y[mask])
    loss.backward()
    optimizer.step()
    return float(loss)
```

### 使用 NeighborLoader 的迷你批次训练

```python
from torch_geometric.loader import NeighborLoader

train_loader = NeighborLoader(
    data,
    num_neighbors=[15] * 2,              # 每跳邻居数（应用于所有边类型）
    batch_size=128,
    input_nodes=('paper', data['paper'].train_mask),
)

# 按边类型精细控制邻居数：
# num_neighbors = {key: [15] * 2 for key in data.edge_types}

def train():
    model.train()
    total_examples = total_loss = 0
    for batch in train_loader:
        optimizer.zero_grad()
        batch = batch.to(device)
        batch_size = batch['paper'].batch_size
        out = model(batch.x_dict, batch.edge_index_dict)
        loss = F.cross_entropy(out['paper'][:batch_size],
                               batch['paper'].y[:batch_size])
        loss.backward()
        optimizer.step()
        total_examples += batch_size
        total_loss += float(loss) * batch_size
    return total_loss / total_examples
```

HGTLoader 也支持类型感知采样：

```python
from torch_geometric.loader import HGTLoader

loader = HGTLoader(data, num_samples=[512] * 2, batch_size=128,
                   input_nodes=('paper', data['paper'].train_mask))
```
