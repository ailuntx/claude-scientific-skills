---
name: torchdrug
description: 基于PyTorch的分子与蛋白质图神经网络工具库。适用于药物研发、蛋白质建模或知识图谱推理领域的自定义GNN架构构建。最适用于定制模型开发、蛋白质属性预测及逆合成分析。预训练模型和多样化特征化工具请使用deepchem；基准数据集请使用pytdc。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# TorchDrug

## 概述

TorchDrug是面向药物研发与分子科学领域的综合性PyTorch机器学习工具箱。应用图神经网络、预训练模型和任务定义处理分子、蛋白质及生物知识图谱，涵盖分子属性预测、蛋白质建模、知识图谱推理、分子生成、逆合成规划等任务，提供40+精选数据集与20+模型架构。

## 适用场景

本技能适用于以下场景：

**数据类型：**
- SMILES字符串或分子结构
- 蛋白质序列或3D结构（PDB文件）
- 化学反应与逆合成
- 生物医学知识图谱
- 药物研发数据集

**任务类型：**
- 预测分子属性（溶解度、毒性、活性）
- 蛋白质功能或结构预测
- 药物-靶点结合预测
- 生成新分子结构
- 规划化学合成路线
- 生物医学知识库链接预测
- 科学数据上的图神经网络训练

**库与集成：**
- TorchDrug为核心库
- 常与RDKit化学信息学工具联用
- 兼容PyTorch与PyTorch Lightning
- 支持AlphaFold和ESM蛋白质工具集成

## 快速入门

### 安装

```bash
uv pip install torchdrug
# 或安装完整依赖
uv pip install torchdrug[full]
```

### 示例代码

```python
from torchdrug import datasets, models, tasks
from torch.utils.data import DataLoader

# 加载分子数据集
dataset = datasets.BBBP("~/molecule-datasets/")
train_set, valid_set, test_set = dataset.split()

# 定义GNN模型
model = models.GIN(
    input_dim=dataset.node_feature_dim,
    hidden_dims=[256, 256, 256],
    edge_input_dim=dataset.edge_feature_dim,
    batch_norm=True,
    readout="mean"
)

# 创建属性预测任务
task = tasks.PropertyPrediction(
    model,
    task=dataset.tasks,
    criterion="bce",
    metric=["auroc", "auprc"]
)

# PyTorch训练流程
optimizer = torch.optim.Adam(task.parameters(), lr=1e-3)
train_loader = DataLoader(train_set, batch_size=32, shuffle=True)

for epoch in range(100):
    for batch in train_loader:
        loss = task(batch)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
```

## 核心功能

### 1. 分子属性预测

基于结构预测分子的化学、物理与生物属性。

**应用场景：**
- 药物相似性与ADMET属性
- 毒性筛查
- 量子化学属性
- 结合亲和力预测

**核心组件：**
- 20+分子数据集（BBBP、HIV、Tox21、QM9等）
- GNN模型（GIN、GAT、SchNet）
- PropertyPrediction与MultipleBinaryClassification任务

**参考：** 查阅`references/molecular_property_prediction.md`获取：
- 完整数据集目录
- 模型选择指南
- 训练流程与最佳实践
- 特征工程细节

### 2. 蛋白质建模

处理蛋白质序列、结构与属性。

**应用场景：**
- 酶功能预测
- 蛋白质稳定性与溶解度
- 亚细胞定位
- 蛋白质相互作用
- 结构预测

**核心组件：**
- 15+蛋白质数据集（EnzymeCommission、GeneOntology、PDBBind等）
- 序列模型（ESM、ProteinBERT、ProteinLSTM）
- 结构模型（GearNet、SchNet）
- 多任务类型支持

**参考：** 查阅`references/protein_modeling.md`获取：
- 蛋白质专用数据集
- 序列模型与结构模型对比
- 预训练策略
- AlphaFold与ESM集成方案

### 3. 知识图谱推理

预测生物知识图谱中的缺失链接与关联关系。

**应用场景：**
- 药物重定位
- 疾病机制发现
- 基因-疾病关联
- 多跳生物医学推理

**核心组件：**
- 通用KG（FB15k、WN18）与生物医学KG（Hetionet）
- 嵌入模型（TransE、RotatE、ComplEx）
- KnowledgeGraphCompletion任务

**参考：** 查阅`references/knowledge_graphs.md`获取：
- 知识图谱数据集（含45k生物医学实体的Hetionet）
- 嵌入模型对比
- 评估指标与协议
- 生物医学应用案例

### 4. 分子生成

生成具有目标属性的新型分子结构。

**应用场景：**
- 全新药物设计
- 先导化合物优化
- 化学空间探索
- 属性导向生成

**核心组件：**
- 自回归生成
- GCPN（基于策略的生成）
- GraphAutoregressiveFlow
- 属性优化流程

**参考：** 查阅`references/molecular_generation.md`获取：
- 生成策略（无条件/条件/骨架导向）
- 多目标优化
- 验证与过滤
- 属性预测集成

### 5. 逆合成分析

预测从目标分子到起始原料的合成路线。

**应用场景：**
- 合成路线规划
- 路线优化
- 合成可行性评估
- 多步规划

**核心组件：**
- USPTO-50k反应数据集
- CenterIdentification（反应中心预测）
- SynthonCompletion（反应物预测）
- 端到端逆合成流程

**参考：** 查阅`references/retrosynthesis.md`获取：
- 任务分解（中心识别→合成子补全）
- 多步合成规划
- 商业可用性检查
- 逆合成工具集成

### 6. 图神经网络模型

覆盖多数据类型与任务的GNN架构全集。

**可用模型：**
- 通用GNN：GCN、GAT、GIN、RGCN、MPNN
- 3D感知：SchNet、GearNet
- 蛋白质专用：ESM、ProteinBERT、GearNet
- 知识图谱：TransE、RotatE、ComplEx、SimplE
- 生成式：GraphAutoregressiveFlow

**参考：** 查阅`references/models_architectures.md`获取：
- 模型详解
- 按任务与数据集选择指南
- 架构对比
- 实现技巧

### 7. 数据集

40+涵盖化学、生物学与知识图谱的精选数据集。

**分类：**
- 分子属性（药物研发、量子化学）
- 蛋白质属性（功能、结构、相互作用）
- 知识图谱（通用与生物医学）
- 逆合成反应

**参考：** 查阅`references/datasets.md`获取：
- 完整数据集目录（含规模与任务）
- 数据集选择指南
- 加载与预处理
- 划分策略（随机/骨架）

## 典型工作流

### 工作流1：分子属性预测

**场景：** 预测候选药物的血脑屏障穿透性。

**步骤：**
1. 加载数据集：`datasets.BBBP()`
2. 选择模型：分子图用GIN
3. 定义任务：二分类`PropertyPrediction`
4. 采用骨架划分实现真实评估
5. 使用AUROC与AUPRC评估

**导航：** `references/molecular_property_prediction.md` → 数据集选择 → 模型选择 → 训练

### 工作流2：蛋白质功能预测

**场景：** 基于序列预测酶功能。

**步骤：**
1. 加载数据集：`datasets.EnzymeCommission()`
2. 选择模型：ESM（预训练）或GearNet（结构模型）
3. 定义任务：多分类`PropertyPrediction`
4. 微调预训练模型或从头训练
5. 使用准确率与类指标评估

**导航：** `references/protein_modeling.md` → 模型选择（序列vs结构）→ 预训练策略

### 工作流3：基于知识图谱的药物重定位

**场景：** 在Hetionet中发现新疾病疗法。

**步骤：**
1. 加载数据集：`datasets.Hetionet()`
2. 选择模型：RotatE或ComplEx
3. 定义任务：`KnowledgeGraphCompletion`
4. 负采样训练
5. 查询"化合物-治疗-疾病"预测
6. 按合理性与机制过滤

**导航：** `references/knowledge_graphs.md` → Hetionet数据集 → 模型选择 → 生物医学应用

### 工作流4：全新分子生成

**场景：** 生成针对靶点结合优化的类药分子。

**步骤：**
1. 基于活性数据训练属性预测器
2. 选择生成方法：强化学习优化的GCPN
3. 定义结合亲和力/类药性/可合成性的奖励函数
4. 带约束生成候选分子
5. 化学验证与类药性过滤
6. 多目标评分排序

**导航：** `references/molecular_generation.md` → 条件生成 → 多目标优化

### 工作流5：逆合成规划

**场景：** 为目标分子规划合成路线。

**步骤：**
1. 加载数据集：`datasets.USPTO50k()`
2. 训练反应中心识别模型（RGCN）
3. 训练合成子补全模型（GIN）
4. 组合为端到端逆合成流程
5. 递归执行多步规划
6. 检查原料商业可用性

**导航：** `references/retrosynthesis.md` → 任务类型 → 多步规划

## 集成模式

### 与RDKit联用

TorchDrug分子与RDKit互转：
```python
from torchdrug import data
from rdkit import Chem

# SMILES → TorchDrug分子
smiles = "CCO"
mol = data.Molecule.from_smiles(smiles)

# TorchDrug → RDKit
rdkit_mol = mol.to_molecule()

# RDKit → TorchDrug
rdkit_mol = Chem.MolFromSmiles(smiles)
mol = data.Molecule.from_molecule(rdkit_mol)
```

### 与AlphaFold/ESM联用

使用预测结构：
```python
from torchdrug import data

# 加载AlphaFold预测结构
protein = data.Protein.from_pdb("AF-P12345-F1-model_v4.pdb")

# 构建含空间边的图
graph = protein.residue_graph(
    node_position="ca",
    edge_types=["sequential", "radius"],
    radius_cutoff=10.0
)
```

### 与PyTorch Lightning联用

任务封装为Lightning模块：
```python
import pytorch_lightning as pl

class LightningTask(pl.LightningModule):
    def __init__(self, torchdrug_task):
        super().__init__()
        self.task = torchdrug_task

    def training_step(self, batch, batch_idx):
        return self.task(batch)

    def validation_step(self, batch, batch_idx):
        pred = self.task.predict(batch)
        target = self.task.target(batch)
        return {"pred": pred, "target": target}

    def configure_optimizers(self):
        return torch.optim.Adam(self.parameters(), lr=1e-3)
```

## 技术细节

深入理解TorchDrug架构：

**核心概念：** 查阅`references/core_concepts.md`获取：
- 架构哲学（模块化、可配置）
- 数据结构（Graph、Molecule、Protein、PackedGraph）
- 模型接口与前向函数签名
- 任务接口（predict/target/forward/evaluate）
- 训练流程与最佳实践
- 损失函数与评估指标
- 常见陷阱与调试

## 速查表

**选择数据集：**
- 分子属性 → `references/datasets.md` → 分子章节
- 蛋白质任务 → `references/datasets.md` → 蛋白质章节
- 知识图谱 → `references/datasets.md` → 知识图谱章节

**选择模型：**
- 分子 → `references/models_architectures.md` → GNN章节 → GIN/GAT/SchNet
- 蛋白质（序列）→ `references/models_architectures.md` → 蛋白质章节 → ESM
- 蛋白质（结构）→ `references/models_architectures.md` → 蛋白质章节 → GearNet
- 知识图谱 → `references/models_architectures.md` → KG章节 → RotatE/ComplEx

**常用任务：**
- 属性预测 → `references/molecular_property_prediction.md` 或 `references/protein_modeling.md`
- 分子生成 → `references/molecular_generation.md`
- 逆合成 → `references/retrosynthesis.md`
- KG推理 → `references/knowledge_graphs.md`

**理解架构：**
- 数据结构 → `references/core_concepts.md` → 数据结构
- 模型设计 → `references/core_concepts.md` → 模型接口
- 任务设计 → `references/core_concepts.md` → 任务接口

## 常见问题排查

**问题：维度不匹配错误**
→ 检查`model.input_dim`是否匹配`dataset.node_feature_dim`
→ 查阅`references/core_concepts.md` → 核心属性

**问题：分子任务性能不佳**
→ 使用骨架划分替代随机划分
→ 尝试GIN替代GCN
→ 查阅`references/molecular_property_prediction.md` → 最佳实践

**问题：蛋白质模型未收敛**
→ 序列任务使用预训练ESM
→ 结构模型检查边构建
→ 查阅`references/protein_modeling.md` → 训练流程

**问题：大图内存溢出**
→ 减小批大小
→ 使用梯度累积
→ 查阅`references/core_concepts.md` → 内存优化

**问题：生成分子无效**
→ 添加有效性约束
→ 通过RDKit后处理验证
→ 查阅`references/molecular_generation.md` → 验证与过滤

## 资源

**官方文档：** https://torchdrug.ai/docs/
**GitHub：** https://github.com/DeepGraphLearning/torchdrug
**论文：** TorchDrug：面向药物研发的强大灵活机器学习平台

## 导航指南

按任务类型查阅对应参考文档：

1. **分子属性预测** → `molecular_property_prediction.md`
2. **蛋白质建模** → `protein_modeling.md`
3. **知识图谱** → `knowledge_graphs.md`
4. **分子生成** → `molecular_generation.md`
5. **逆合成** → `retrosynthesis.md`
6. **模型选择** → `models_architectures.md`
7. **数据集选择** → `datasets.md`
8. **技术细节** → `core_concepts.md`

每份参考文档均提供领域内的完整覆盖，含示例、最佳实践与典型用例。
