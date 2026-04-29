# 核心概念与技术细节

## 概述

本文档涵盖 TorchDrug 的基础架构、设计原则与技术实现细节。

## 架构理念

### 模块化设计

TorchDrug 将功能划分为独立模块：

1. **表示模型** (models.py)：将图结构编码为嵌入向量
2. **任务定义** (tasks.py)：定义学习目标与评估标准
3. **数据处理** (data.py, datasets.py)：图结构与数据集管理
4. **核心组件** (core.py)：基础类与工具集

**优势：**
- 跨任务复用表示模型
- 灵活组合组件
- 便捷实验与原型开发
- 清晰的职责分离

### 可配置系统

所有组件继承自 `core.Configurable`：
- 序列化为配置字典
- 从配置重建实例
- 保存/加载完整流程
- 实验可复现性

## 核心组件

### core.Configurable

所有 TorchDrug 组件的基类。

**关键方法：**
- `config_dict()`：序列化为字典
- `load_config_dict(config)`：从字典加载
- `save(file)`：保存到文件
- `load(file)`：从文件加载

**示例：**
```python
from torchdrug import core, models

model = models.GIN(input_dim=10, hidden_dims=[256, 256])

# 保存配置
config = model.config_dict()
# {'class': 'GIN', 'input_dim': 10, 'hidden_dims': [256, 256], ...}

# 重建模型
model2 = core.Configurable.load_config_dict(config)
```

### core.Registry

用于注册模型、任务和数据集类的装饰器。

**用法：**
```python
from torchdrug import core as core_td

@core_td.register("models.CustomModel")
class CustomModel(nn.Module, core_td.Configurable):
    def __init__(self, input_dim, hidden_dim):
        super().__init__()
        self.linear = nn.Linear(input_dim, hidden_dim)

    def forward(self, graph, input, all_loss, metric):
        # 模型实现
        pass
```

**优势：**
- 模型自动序列化
- 基于字符串的模型指定
- 便捷的模型查找与实例化

## 数据结构

### 图 (Graph)

表示分子或蛋白质图的核心数据结构。

**属性：**
- `num_node`：节点数量
- `num_edge`：边数量
- `node_feature`：节点特征张量 [num_node, feature_dim]
- `edge_feature`：边特征张量 [num_edge, feature_dim]
- `edge_list`：边连接关系 [num_edge, 2 或 3]
- `num_relation`：边类型数量（多关系场景）

**方法：**
- `node_mask(mask)`：选择节点子集
- `edge_mask(mask)`：选择边子集
- `undirected()`：转换为无向图
- `directed()`：转换为有向图

**批处理：**
- 多个图合并为单个不连通图
- DataLoader 自动批处理
- 保留各图的节点/边索引

### 分子 (Molecule，继承自 Graph)

专用于分子的图结构。

**扩展属性：**
- `atom_type`：原子类型（原子序数）
- `bond_type`：键类型（单键、双键、三键、芳香键）
- `formal_charge`：原子形式电荷
- `explicit_hs`：显式氢原子计数

**方法：**
- `from_smiles(smiles)`：从 SMILES 字符串创建
- `from_molecule(mol)`：从 RDKit 分子创建
- `to_smiles()`：转换为 SMILES
- `to_molecule()`：转换为 RDKit 分子
- `ion_to_molecule()`：电荷中和

**示例：**
```python
from torchdrug import data

# 从 SMILES 创建
mol = data.Molecule.from_smiles("CCO")

# 原子特征
print(mol.atom_type)  # [6, 6, 8] (C, C, O)
print(mol.bond_type)  # [1, 1] (单键)
```

### 蛋白质 (Protein，继承自 Graph)

专用于蛋白质的图结构。

**扩展属性：**
- `residue_type`：氨基酸类型
- `atom_name`：原子名称（CA, CB 等）
- `atom_type`：原子序数
- `residue_number`：残基编号
- `chain_id`：链标识符

**方法：**
- `from_pdb(pdb_file)`：从 PDB 文件加载
- `from_sequence(sequence)`：从序列创建
- `to_pdb(pdb_file)`：保存为 PDB 文件

**图构建：**
- 节点通常代表残基（非原子）
- 边类型：序列连接、空间邻近（KNN）或接触关系
- 可配置的边构建策略

**示例：**
```python
from torchdrug import data

# 加载蛋白质
protein = data.Protein.from_pdb("1a3x.pdb")

# 构建多边类型图
graph = protein.residue_graph(
    node_position="ca",  # 使用 Cα 位置
    edge_types=["sequential", "radius"]  # 序列边 + 空间边
)
```

### PackedGraph

异构图的高效批处理结构。

**用途：**
- 批处理不同尺寸的图
- 单次 GPU 内存分配
- 高效并行处理

**属性：**
- `num_nodes`：各图的节点数列表
- `num_edges`：各图的边数列表
- `graph_ind`：节点的归属图索引

**应用场景：**
- DataLoader 自动批处理
- 自定义批处理策略
- 多图联合操作

## 模型接口

### 前向函数签名

所有 TorchDrug 模型遵循标准化接口：

```python
def forward(self, graph, input, all_loss=None, metric=None):
    """
    参数:
        graph (Graph): 批处理图结构
        input (Tensor): 节点输入特征
        all_loss (Tensor, 可选): 损失累加器
        metric (dict, 可选): 指标字典

    返回:
        dict: 包含表示键的输出字典
    """
    # 模型计算
    output = self.layers(graph, input)

    return {
        "node_feature": output,
        "graph_feature": graph_pooling(output)
    }
```

**关键点：**
- `graph`：批处理图结构
- `input`：节点特征 [num_node, input_dim]
- `all_loss`：累计损失（多任务场景）
- `metric`：共享指标字典
- 返回包含表示类型的字典

### 必需属性

**所有模型必须定义：**
- `input_dim`：输入特征维度
- `output_dim`：输出表示维度

**目的：**
- 自动维度检查
- 模型管道化组合
- 错误检查与验证

**示例：**
```python
class CustomModel(nn.Module):
    def __init__(self, input_dim, hidden_dim):
        super().__init__()
        self.input_dim = input_dim
        self.output_dim = hidden_dim
        # ... 层定义 ...
```

## 任务接口

### 核心任务方法

所有任务实现以下方法：

```python
class CustomTask(tasks.Task):
    def preprocess(self, train_set, valid_set, test_set):
        """数据集预处理（可选）"""
        pass

    def predict(self, batch):
        """生成批次预测"""
        graph, label = batch
        output = self.model(graph, graph.node_feature)
        pred = self.mlp(output["graph_feature"])
        return pred

    def target(self, batch):
        """提取真实标签"""
        graph, label = batch
        return label

    def forward(self, batch):
        """计算训练损失"""
        pred = self.predict(batch)
        target = self.target(batch)
        loss = self.criterion(pred, target)
        return loss

    def evaluate(self, pred, target):
        """计算评估指标"""
        metrics = {}
        metrics["auroc"] = compute_auroc(pred, target)
        metrics["auprc"] = compute_auprc(pred, target)
        return metrics
```

### 任务组件

**典型任务结构：**
1. **表示模型**：将图编码为嵌入向量
2. **读出/预测头**：将嵌入映射为预测结果
3. **损失函数**：训练目标函数
4. **评估指标**：性能度量标准

**示例：**
```python
from torchdrug import tasks, models

# 表示模型
model = models.GIN(input_dim=10, hidden_dims=[256, 256])

# 任务封装模型与预测头
task = tasks.PropertyPrediction(
    model=model,
    task=["task1", "task2"],  # 多任务
    criterion="bce",
    metric=["auroc", "auprc"],
    num_mlp_layer=2
)
```

## 训练流程

### 标准训练循环

```python
import torch
from torch.utils.data import DataLoader
from torchdrug import core, models, tasks, datasets

# 1. 加载数据集
dataset = datasets.BBBP("~/datasets/")
train_set, valid_set, test_set = dataset.split()

# 2. 创建数据加载器
train_loader = DataLoader(train_set, batch_size=32, shuffle=True)
valid_loader = DataLoader(valid_set, batch_size=32)

# 3. 定义模型与任务
model = models.GIN(input_dim=dataset.node_feature_dim,
                   hidden_dims=[256, 256, 256])
task = tasks.PropertyPrediction(model, task=dataset.tasks,
                                 criterion="bce", metric=["auroc", "auprc"])

# 4. 设置优化器
optimizer = torch.optim.Adam(task.parameters(), lr=1e-3)

# 5. 训练循环
for epoch in range(100):
    # 训练阶段
    task.train()
    for batch in train_loader:
        loss = task(batch)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

    # 验证阶段
    task.eval()
    preds, targets = [], []
    for batch in valid_loader:
        pred = task.predict(batch)
        target = task.target(batch)
        preds.append(pred)
        targets.append(target)

    preds = torch.cat(preds)
    targets = torch.cat(targets)
    metrics = task.evaluate(preds, targets)
    print(f"Epoch {epoch}: {metrics}")
```

### PyTorch Lightning 集成

TorchDrug 任务兼容 PyTorch Lightning：

```python
import pytorch_lightning as pl

class LightningWrapper(pl.LightningModule):
    def __init__(self, task):
        super().__init__()
        self.task = task

    def training_step(self, batch, batch_idx):
        loss = self.task(batch)
        return loss

    def validation_step(self, batch, batch_idx):
        pred = self.task.predict(batch)
        target = self.task.target(batch)
        return {"pred": pred, "target": target}

    def validation_epoch_end(self, outputs):
        preds = torch.cat([o["pred"] for o in outputs])
        targets = torch.cat([o["target"] for o in outputs])
        metrics = self.task.evaluate(preds, targets)
        self.log_dict(metrics)

    def configure_optimizers(self):
        return torch.optim.Adam(self.parameters(), lr=1e-3)
```

## 损失函数

### 内置损失类型

**分类任务：**
- `"bce"`：二元交叉熵
- `"ce"`：交叉熵（多分类）

**回归任务：**
- `"mse"`：均方误差
- `"mae"`：平均绝对误差

**知识图谱：**
- `"bce"`：三元组二元分类
- `"ce"`：排序交叉熵
- `"margin"`：基于间隔的排序损失

### 自定义损失

```python
class CustomTask(tasks.Task):
    def forward(self, batch):
        pred = self.predict(batch)
        target = self.target(batch)

        # 自定义损失计算
        loss = custom_loss_function(pred, target)

        return loss
```

## 评估指标

### 常用指标

**分类任务：**
- **AUROC**：ROC 曲线下面积
- **AUPRC**：精确率-召回率曲线下面积
- **Accuracy**：准确率
- **F1**：精确率与召回率的调和平均

**回归任务：**
- **MAE**：平均绝对误差
- **RMSE**：均方根误差
- **R²**：决定系数
- **Pearson**：皮尔逊相关系数

**排序任务（知识图谱）：**
- **MR**：平均排名
- **MRR**：平均倒数排名
- **Hits@K**：前 K 命中率

### 多任务指标

多标签/多任务场景：
- 按任务分别计算指标
- 跨任务宏平均
- 支持任务重要性加权

## 数据变换

### 分子变换

```python
from torchdrug import transforms

# 添加连接所有原子的虚拟节点
transform1 = transforms.VirtualNode()

# 添加虚拟边
transform2 = transforms.VirtualEdge()

# 组合变换
transform = transforms.Compose([transform1, transform2])

dataset = datasets.BBBP("~/datasets/", transform=transform)
```

### 蛋白质变换

```python
# 基于空间邻近添加边
transform = transforms.TruncateProtein(max_length=500)

dataset = datasets.Fold("~/datasets/", transform=transform)
```

## 最佳实践

### 内存优化

1. **梯度累积**：适用于大模型
2. **混合精度**：FP16 训练
3. **批尺寸调优**：平衡速度与内存
4. **数据加载**：多进程 I/O 优化

### 可复现性

1. **设置随机种子**：PyTorch/NumPy/Python
2. **确定性操作**：`torch.use_deterministic_algorithms(True)`
3. **保存配置**：使用 `core.Configurable`
4. **版本控制**：记录 TorchDrug 版本

### 调试技巧

1. **维度检查**：验证 `input_dim` 和 `output_dim`
2. **批处理验证**：打印批次统计信息
3. **梯度监控**：检测梯度消失/爆炸
4. **小批量过拟合**：验证模型容量

### 性能优化

1. **GPU 利用率**：通过 `nvidia-smi` 监控
2. **代码剖析**：使用 PyTorch profiler
3. **数据加载优化**：预取/内存锁定
4. **模型编译**：适用时使用 TorchScript

## 高级主题

### 多任务学习

单模型处理多任务：
```python
task = tasks.PropertyPrediction(
    model,
    task=["task1", "task2", "task3"],
    criterion="bce",
    metric=["auroc"],
    task_weight=[1.0, 1.0, 2.0]  # 任务3权重更高
)
```

### 迁移学习

1. 大型数据集预训练
2. 目标数据集微调
3. 可选冻结底层参数

### 自监督预训练

预训练任务类型：
- `AttributeMasking`：掩码节点特征
- `EdgePrediction`：预测边存在性
- `ContextPrediction`：对比学习

### 自定义层

扩展 TorchDrug 的 GNN 层：
```python
from torchdrug import layers

class CustomConv(layers.MessagePassingBase):
    def message(self, graph, input):
        # 自定义消息函数
        pass

    def aggregate(self, graph, message):
        # 自定义聚合操作
        pass

    def combine(self, input, update):
        # 自定义组合函数
        pass
```

## 常见陷阱

1. **忽略 `input_dim/output_dim`**：导致模型无法组合
2. **批处理不当**：变长图需用 PackedGraph
3. **数据泄露**：注意骨架划分与预训练
4. **忽视边特征**：键/空间信息可能至关重要
5. **指标选择错误**：匹配任务特性（如不平衡数据用 AUROC）
6. **正则化不足**：使用 dropout/权重衰减/早停
7. **未验证化学有效性**：生成分子必须有效
8. **小数据集过拟合**：采用预训练或简化模型
