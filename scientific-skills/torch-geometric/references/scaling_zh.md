# 扩展图神经网络（GNN）——完整参考指南

适用于无法放入GPU内存的大型图表的GNN训练技术、多GPU训练及性能优化。

## 目录
1. 邻居采样（NeighborLoader）
2. 其他采样策略
3. 多GPU/分布式训练
4. torch.compile支持
5. 性能优化技巧

---

## 1. 邻居采样（NeighborLoader）

大型单图训练的主要方法。递归采样每跳固定数量的邻居，限制计算图规模。

```python
from torch_geometric.loader import NeighborLoader

loader = NeighborLoader(
    data,
    num_neighbors=[15, 10],       # 每跳最大邻居数（跳1:15, 跳2:10）
    batch_size=1024,               # 每批种子节点数
    input_nodes=data.train_mask,   # 采样源节点
    shuffle=True,
    num_workers=4,                 # 并行数据加载
    replace=False,                 # 无放回采样
)
```

**关键参数：**
- `num_neighbors`：每跳最大邻居数列表，长度需匹配GNN深度。使用`-1`表示采样该跳所有邻居。
- `input_nodes`：种子节点——可以是掩码、索引张量或异构图中的`('节点类型', 掩码)`元组。
- `subgraph_type`：`"directional"`（默认）、`"bidirectional"`（添加反向边）或`"induced"`（完整诱导子图）。
- `disjoint`：设为`True`时不融合种子节点间的邻域（占用更多内存但有时必需）。

**训练模式：**
```python
model = GraphSAGE(in_channels, hidden_channels, out_channels, num_layers=2)

for batch in loader:
    batch = batch.to(device)
    out = model(batch.x, batch.edge_index)
    # 关键：仅前batch_size个节点是种子节点
    loss = F.cross_entropy(out[:batch.batch_size], batch.y[:batch.batch_size])
```

**重要细节：**
- 节点已排序：前`batch.batch_size`个节点为种子节点
- `batch.n_id`将局部索引映射回原始节点ID
- 通常无法采样超过2-3跳（邻域指数级增长）
- 保持`len(num_neighbors) == num_gnn_layers`以确保效率

### LinkNeighborLoader（用于链接预测）

围绕监督边采样子图：

```python
from torch_geometric.loader import LinkNeighborLoader

loader = LinkNeighborLoader(
    data,
    num_neighbors=[20, 10],
    edge_label_index=train_data.edge_label_index,
    edge_label=train_data.edge_label,
    batch_size=256,
    neg_sampling_ratio=1.0,
    shuffle=True,
)
```

### HGTLoader（类型感知的异构图采样）

遵循HGT论文，每跳按类型采样固定数量节点：

```python
from torch_geometric.loader import HGTLoader

loader = HGTLoader(
    data,
    num_samples=[512] * 2,        # 每跳每类节点数
    batch_size=128,
    input_nodes=('paper', data['paper'].train_mask),
)
```

## 2. 其他采样策略

### ClusterLoader（ClusterGCN）

将图划分为簇，在完整子图上训练。适用于深层GNN（消息在簇内自由传播）：

```python
from torch_geometric.loader import ClusterData, ClusterLoader

cluster_data = ClusterData(data, num_parts=1500)
loader = ClusterLoader(cluster_data, batch_size=20, shuffle=True)

for batch in loader:
    # batch是完整子图——无需切片
    out = model(batch.x, batch.edge_index)
    loss = F.cross_entropy(out[batch.train_mask], batch.y[batch.train_mask])
```

### GraphSAINTSampler

通过随机游走/节点/边进行重要性归一化子图采样：

```python
from torch_geometric.loader import GraphSAINTRandomWalkSampler

loader = GraphSAINTRandomWalkSampler(
    data, batch_size=6000, walk_length=2, num_steps=5,
)
```

### ShaDowKHopSampler

围绕种子节点提取K跳诱导子图——解耦深度与范围：

```python
from torch_geometric.loader import ShaDowKHopSampler

loader = ShaDowKHopSampler(
    data, depth=2, num_neighbors=5, batch_size=64,
    input_nodes=data.train_mask,
)
```

## 3. 多GPU/分布式训练

### DistributedDataParallel（DDP）

标准PyTorch DDP可与PyG协同工作。每个GPU处理部分种子节点：

```python
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel
import torch.multiprocessing as mp

def run(rank, world_size, dataset):
    # 初始化进程组
    os.environ['MASTER_ADDR'] = 'localhost'
    os.environ['MASTER_PORT'] = '12345'
    dist.init_process_group('nccl', rank=rank, world_size=world_size)

    data = dataset[0]

    # 跨GPU分配训练节点
    train_idx = data.train_mask.nonzero().view(-1)
    train_idx = train_idx.split(train_idx.size(0) // world_size)[rank]

    loader = NeighborLoader(
        data,
        input_nodes=train_idx,
        num_neighbors=[25, 10],
        batch_size=1024,
        num_workers=4,
        shuffle=True,
    )

    # 用DDP封装模型
    model = GraphSAGE(...).to(rank)
    model = DistributedDataParallel(model, device_ids=[rank])
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

    for epoch in range(10):
        model.train()
        for batch in loader:
            batch = batch.to(rank)
            optimizer.zero_grad()
            out = model(batch.x, batch.edge_index)[:batch.batch_size]
            loss = F.cross_entropy(out, batch.y[:batch.batch_size])
            loss.backward()
            optimizer.step()

        # 评估前同步
        dist.barrier()

        if rank == 0:
            # 仅在rank 0上评估
            ...

    dist.destroy_process_group()

# 启动
if __name__ == '__main__':
    dataset = Reddit('./data/Reddit')
    world_size = torch.cuda.device_count()
    mp.spawn(run, args=(world_size, dataset), nprocs=world_size, join=True)
```

**关键点：**
- 在`mp.spawn()`前初始化数据集——数据自动移至共享内存
- 每个rank创建独立的NeighborLoader处理种子节点子集
- 评估前调用`dist.barrier()`同步
- 为简化仅在rank 0上评估
- 用`dist.destroy_process_group()`清理资源

### PyTorch Lightning集成

PyG提供Lightning封装减少样板代码：

```python
from torch_geometric.data import LightningNodeData

datamodule = LightningNodeData(
    data,
    input_train_nodes=data.train_mask,
    input_val_nodes=data.val_mask,
    input_test_nodes=data.test_mask,
    loader='neighbor',
    num_neighbors=[25, 10],
    batch_size=1024,
)

# 兼容任意Lightning Trainer
trainer = L.Trainer(devices=4, accelerator='gpu', strategy='ddp')
trainer.fit(model, datamodule)
```

另有：链接预测用`LightningLinkData`，图级任务用`LightningDataset`。

## 4. torch.compile支持

PyG支持`torch.compile`加速执行：

```python
model = GCN(...)
model = torch.compile(model)

# 兼容标准训练循环
out = model(data.x, data.edge_index)
```

**支持范围：**
- 多数GNN层（GCNConv, SAGEConv, GATConv等）
- 标准训练/推理流程
- CPU和CUDA后端

**限制：**
- 动态形状（每批图尺寸变化）可能触发重编译
- 部分特殊层或自定义MessagePassing子类可能不兼容
- 若批次图尺寸差异大，使用`torch.compile(model, dynamic=True)`

## 5. 性能优化技巧

- **num_workers**：在数据加载器中设置`num_workers=4`（或更高）实现CPU端并行
- **pin_memory**：加载器中使用`pin_memory=True`加速CPU到GPU传输
- **稀疏张量**：在部分层使用`torch_sparse`的`SparseTensor`替代`edge_index`加速消息传递
- **性能剖析**：使用`torch_geometric.profile`测量单层时间和内存
- **混合精度**：标准PyTorch AMP与PyG兼容：
  ```python
  from torch.amp import autocast, GradScaler
  scaler = GradScaler()
  with autocast('cuda'):
      out = model(batch.x, batch.edge_index)
      loss = F.cross_entropy(out[:batch.batch_size], batch.y[:batch.batch_size])
  scaler.scale(loss).backward()
  scaler.step(optimizer)
  scaler.update()
  ```
- **减少采样**：每跳邻居数越少越快但噪声越大。双层GNN建议从`[15, 10]`开始
- **避免冗余计算**：使用NeighborLoader时仅前`batch_size`个输出有效——不要在采样节点上计算指标
