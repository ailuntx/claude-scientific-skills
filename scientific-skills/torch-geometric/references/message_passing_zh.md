# 自定义消息传递层

通过 `MessagePassing` 基类实现自定义 GNN 层的完整参考指南。

## MessagePassing API

```python
MessagePassing(aggr="add", flow="source_to_target", node_dim=-2)
```

- `aggr`: 聚合方案 — `"add"`（求和）、`"mean"`（平均）或 `"max"`（最大值）
- `flow`: 消息方向 — `"source_to_target"`（默认，源节点到目标节点）或 `"target_to_source"`（目标节点到源节点）
- `node_dim`: 传播操作的维度轴

### 需重写的方法

- `message(...)`: 为每条边构造消息。通过 `_j`/`_i` 后缀访问源/目标节点特征。
- `aggregate(inputs, index)`: 聚合消息（通常由 `aggr` 参数处理）。
- `update(aggr_out, ...)`: 对每个节点进行聚合后变换。
- `propagate(edge_index, size=None, **kwargs)`: 协调整个流程。在 `forward()` 中调用此方法。

传递给 `propagate()` 的任何张量，都可通过在 `message()` 中添加 `_i`（目标节点）或 `_j`（源节点）后缀自动索引。例如，传递 `x=features` 后，即可在消息函数中使用 `x_i` 和 `x_j`。

对于二分图，需向 `propagate()` 传递 `size=(N, M)` 并将特征作为元组提供：`x=(x_src, x_dst)`。

## 示例：从零实现 GCN 层

```python
import torch
from torch.nn import Linear, Parameter
from torch_geometric.nn import MessagePassing
from torch_geometric.utils import add_self_loops, degree

class GCNConv(MessagePassing):
    def __init__(self, in_channels, out_channels):
        super().__init__(aggr='add')
        self.lin = Linear(in_channels, out_channels, bias=False)
        self.bias = Parameter(torch.empty(out_channels))
        self.reset_parameters()

    def reset_parameters(self):
        self.lin.reset_parameters()
        self.bias.data.zero_()

    def forward(self, x, edge_index):
        # 1. 添加自循环边
        edge_index, _ = add_self_loops(edge_index, num_nodes=x.size(0))
        # 2. 线性变换
        x = self.lin(x)
        # 3. 计算归一化系数
        row, col = edge_index
        deg = degree(col, x.size(0), dtype=x.dtype)
        deg_inv_sqrt = deg.pow(-0.5)
        deg_inv_sqrt[deg_inv_sqrt == float('inf')] = 0
        norm = deg_inv_sqrt[row] * deg_inv_sqrt[col]
        # 4-5. 消息传递
        out = self.propagate(edge_index, x=x, norm=norm)
        # 6. 添加偏置项
        return out + self.bias

    def message(self, x_j, norm):
        # x_j: 每条边的源节点特征 [num_edges, out_channels]
        # norm: 归一化系数 [num_edges]
        return norm.view(-1, 1) * x_j
```

## 示例：EdgeConv 层

```python
import torch
from torch.nn import Sequential as Seq, Linear, ReLU
from torch_geometric.nn import MessagePassing

class EdgeConv(MessagePassing):
    def __init__(self, in_channels, out_channels):
        super().__init__(aggr='max')
        self.mlp = Seq(
            Linear(2 * in_channels, out_channels),
            ReLU(),
            Linear(out_channels, out_channels),
        )

    def forward(self, x, edge_index):
        return self.propagate(edge_index, x=x)

    def message(self, x_i, x_j):
        # x_i: 目标节点特征 [num_edges, in_channels]
        # x_j: 源节点特征 [num_edges, in_channels]
        return self.mlp(torch.cat([x_i, x_j - x_i], dim=1))
```

## 示例：动态 EdgeConv（每层重新计算图结构）

```python
from torch_geometric.nn import knn_graph

class DynamicEdgeConv(EdgeConv):
    def __init__(self, in_channels, out_channels, k=6):
        super().__init__(in_channels, out_channels)
        self.k = k

    def forward(self, x, batch=None):
        edge_index = knn_graph(x, self.k, batch, loop=False, flow=self.flow)
        return super().forward(x, edge_index)
```

## 实用函数

```python
from torch_geometric.utils import (
    add_self_loops,      # 添加自循环边
    remove_self_loops,   # 移除自循环边
    degree,              # 计算节点度数
    softmax,             # 在邻域上执行稀疏softmax
    to_dense_adj,        # 将edge_index转为稠密邻接矩阵
    to_undirected,       # 使edge_index变为无向图
    contains_self_loops, # 检查是否存在自循环边
    is_undirected,       # 检查图是否为无向图
    scatter,             # 分散操作（求和、平均、最大值）
)
```
