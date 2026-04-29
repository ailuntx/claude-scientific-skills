# GNN 可解释性 — 完整参考

PyG 提供 `torch_geometric.explain` 模块用于解释 GNN 预测。该模块包含统一的 `Explainer` 接口、多种解释算法、可视化工具及评估指标。

## Explainer 接口

`Explainer` 类是核心入口点，可通过以下配置：
1. 解释**算法**（GNNExplainer, PGExplainer, CaptumExplainer 等）
2. **解释类型**（`"model"`——解释模型预测，或 `"phenomenon"`——解释数据模式）
3. **掩码类型**——需解释的输入部分（节点、边、特征）
4. **后处理**——掩码阈值化方式（top-k, hard 等）

```python
from torch_geometric.explain import Explainer, GNNExplainer

explainer = Explainer(
    model=model,
    algorithm=GNNExplainer(epochs=200),
    explanation_type='model',          # 'model' 或 'phenomenon'
    node_mask_type='attributes',       # 'object', 'common_attributes', 'attributes' 或 None
    edge_mask_type='object',           # 'object' 或 None
    model_config=dict(
        mode='multiclass_classification',  # 'binary_classification', 'multiclass_classification', 'regression'
        task_level='node',                  # 'node', 'edge', 'graph'
        return_type='log_probs',            # 'log_probs', 'probs', 'raw'
    ),
)
```

**掩码类型说明：**
- `'object'`：每个节点/边一个掩码值（哪些节点/边重要？）
- `'attributes'`：每个节点特征维度一个掩码值（哪些特征重要？）
- `'common_attributes'`：所有节点共享相同特征掩码
- `None`：不生成此类型掩码

## 生成解释

### 节点分类

```python
# 解释索引为10的节点预测
explanation = explainer(data.x, data.edge_index, index=10)

print(explanation.node_mask)   # [num_nodes, num_features] — 各节点各特征重要性
print(explanation.edge_mask)   # [num_edges] — 各边重要性
```

### 图分类

```python
explainer = Explainer(
    model=model,
    algorithm=GNNExplainer(epochs=200),
    explanation_type='model',
    edge_mask_type='object',
    model_config=dict(
        mode='multiclass_classification',
        task_level='graph',
        return_type='raw',
    ),
)

explanation = explainer(data.x, data.edge_index)
```

## 可视化

```python
# 可视化最重要特征（条形图）
explanation.visualize_feature_importance(top_k=10)
# 默认保存为 'feature_importance.png'，可通过 path= 指定路径

# 可视化重要子图
explanation.visualize_graph()
# 默认保存为 'graph.png'，可通过 path= 指定路径
```

## 可用算法

### GNNExplainer

通过优化学习软掩码。适用于节点和图级任务，应用最广泛的算法。

```python
from torch_geometric.explain import GNNExplainer

algorithm = GNNExplainer(epochs=200, lr=0.01)
```

### PGExplainer

参数化（可训练）解释器——学习生成边掩码的神经网络。需预训练但可泛化至新图。仅支持边掩码（无节点掩码）。

```python
from torch_geometric.explain import PGExplainer

explainer = Explainer(
    model=model,
    algorithm=PGExplainer(epochs=30, lr=0.003),
    explanation_type='phenomenon',     # PGExplainer 解释现象
    edge_mask_type='object',
    model_config=dict(
        mode='regression',
        task_level='graph',
        return_type='raw',
    ),
    threshold_config=dict(threshold_type='topk', value=10),
)

# 先训练解释器
for epoch in range(30):
    for batch in loader:
        loss = explainer.algorithm.train(
            epoch, model, batch.x, batch.edge_index, target=batch.target
        )

# 再执行解释
explanation = explainer(data.x, data.edge_index)
```

### CaptumExplainer

封装 [Captum](https://captum.ai/) 库，提供基于梯度的归因方法。支持同质图和异质图。

```python
from torch_geometric.explain import CaptumExplainer

# 支持：'IntegratedGradients', 'Saliency', 'Deconvolution',
#       'ShapleyValueSampling', 'GuidedBackprop' 等
algorithm = CaptumExplainer('IntegratedGradients')
```

需通过 `pip install captum`（或 `uv add captum`）安装。

### AttentionExplainer

使用基于注意力的 GNN（GATConv, TransformerConv）中的注意力权重作为边解释。无需训练——直接读取现有注意力分数。

```python
from torch_geometric.explain import AttentionExplainer

algorithm = AttentionExplainer()
```

## 异质图解释

对于异质模型，解释器返回含分类型掩码的 `HeteroExplanation`：

```python
from torch_geometric.explain import Explainer, CaptumExplainer

explainer = Explainer(
    model=hetero_model,
    algorithm=CaptumExplainer('IntegratedGradients'),
    explanation_type='model',
    node_mask_type='attributes',
    edge_mask_type='object',
    model_config=dict(
        mode='multiclass_classification',
        task_level='node',
        return_type='probs',
    ),
)

hetero_explanation = explainer(
    data.x_dict,
    data.edge_index_dict,
    index=torch.tensor([1, 3]),
)

# 访问分类型掩码
hetero_explanation.node_mask_dict    # {'paper': tensor, 'author': tensor, ...}
hetero_explanation.edge_mask_dict    # {('paper','cites','paper'): tensor, ...}
```

## 评估指标

```python
from torch_geometric.explain import unfaithfulness, fidelity, characterization_score

# 不忠实度：解释对预测的改变程度？
# 值越低越好（0=完全忠实）
score = unfaithfulness(explainer, explanation)

# 忠实度：通过正/负忠实度衡量解释质量
pos_fidelity, neg_fidelity = fidelity(explainer, explanation)

# 表征分数：综合指标
char_score = characterization_score(pos_fidelity, neg_fidelity)
```

## 掩码后处理

控制原始掩码值如何转换为最终解释：

```python
explainer = Explainer(
    ...,
    threshold_config=dict(
        threshold_type='topk',    # 'topk', 'hard' 或 None
        value=10,                 # 'topk' 保留前10条边，'hard' 为阈值
    ),
)
```

- `'topk'`：仅保留得分最高的前 k 个元素
- `'hard'`：二值化阈值——高于 `value` 的元素被保留
- `None`：返回原始连续掩码值
