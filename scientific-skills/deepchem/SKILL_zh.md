---
name: deepchem
description: 提供多样化分子特征化工具和预置数据集的分子机器学习库。适用于需要丰富特征化选项和MoleculeNet基准测试的属性预测（ADMET、毒性），可使用传统机器学习或图神经网络。最适合使用预训练模型进行快速实验和探索多种分子表示。若需图优先的PyTorch工作流请用torchdrug；基准数据集请用pytdc。
license: MIT 许可证
metadata:
    skill-author: K-Dense Inc.
---

# DeepChem

## 概述

DeepChem 是一个全面的 Python 库，用于将机器学习应用于化学、材料科学和生物学领域。通过专用神经网络、分子特征化方法和预训练模型，实现分子属性预测、药物发现、材料设计和生物分子分析。

## 适用场景

在以下情况应使用本技能：
- 加载和处理分子数据（SMILES字符串、SDF文件、蛋白质序列）
- 预测分子属性（溶解度、毒性、结合亲和力、ADMET性质）
- 在化学/生物数据集上训练模型
- 使用 MoleculeNet 基准数据集（Tox21、BBBP、Delaney 等）
- 将分子转换为机器学习就绪特征（指纹、图表示、描述符）
- 实现分子图神经网络（GCN、GAT、MPNN、AttentiveFP）
- 应用预训练模型迁移学习（ChemBERTa、GROVER、MolFormer）
- 预测晶体/材料属性（带隙、形成能）
- 分析蛋白质或DNA序列

## 核心功能

### 1. 分子数据加载与处理

DeepChem 提供针对不同化学数据格式的专用加载器：

```python
import deepchem as dc

# 加载含SMILES的CSV
featurizer = dc.feat.CircularFingerprint(radius=2, size=2048)
loader = dc.data.CSVLoader(
    tasks=['solubility', 'toxicity'],
    feature_field='smiles',
    featurizer=featurizer
)
dataset = loader.create_dataset('molecules.csv')

# 加载SDF文件
loader = dc.data.SDFLoader(tasks=['activity'], featurizer=featurizer)
dataset = loader.create_dataset('compounds.sdf')

# 加载蛋白质序列
loader = dc.data.FASTALoader()
dataset = loader.create_dataset('proteins.fasta')
```

**关键加载器**：
- `CSVLoader`：含分子标识符的表格数据
- `SDFLoader`：分子结构文件
- `FASTALoader`：蛋白质/DNA序列
- `ImageLoader`：分子图像
- `JsonLoader`：JSON格式数据集

### 2. 分子特征化

将分子转换为适合机器学习模型的数值表示。

#### 特征化工具选择决策树

```
是否为图神经网络模型？
├─ 是 → 使用图特征化工具
│   ├─ 标准GNN → MolGraphConvFeaturizer
│   ├─ 消息传递 → DMPNNFeaturizer
│   └─ 预训练模型 → GroverFeaturizer
│
└─ 否 → 模型类型？
    ├─ 传统机器学习（RF、XGBoost、SVM）
    │   ├─ 快速基线 → CircularFingerprint (ECFP)
    │   ├─ 可解释性 → RDKitDescriptors
    │   └─ 最大覆盖 → MordredDescriptors
    │
    ├─ 深度学习（非图结构）
    │   ├─ 密集网络 → CircularFingerprint
    │   └─ 卷积网络 → SmilesToImage
    │
    ├─ 序列模型（LSTM、Transformer）
    │   └─ SmilesToSeq
    │
    └─ 3D结构分析
        └─ CoulombMatrix
```

#### 特征化示例

```python
# 指纹（传统机器学习）
fp = dc.feat.CircularFingerprint(radius=2, size=2048)

# 描述符（可解释模型）
desc = dc.feat.RDKitDescriptors()

# 图特征（图神经网络）
graph_feat = dc.feat.MolGraphConvFeaturizer()

# 应用特征化
features = fp.featurize(['CCO', 'c1ccccc1'])
```

**选择指南**：
- **小型数据集 (<1K)**：CircularFingerprint 或 RDKitDescriptors
- **中型数据集 (1K-100K)**：CircularFingerprint 或图特征化工具
- **大型数据集 (>100K)**：图特征化工具（MolGraphConvFeaturizer, DMPNNFeaturizer）
- **迁移学习**：预训练模型特征化工具（GroverFeaturizer）

完整特征化工具文档见 `references/api_reference.md`

### 3. 数据拆分

**关键提示**：药物发现任务请使用 `ScaffoldSplitter`，防止相似分子结构在训练集和测试集间出现数据泄露。

```python
# 骨架拆分（分子数据推荐）
splitter = dc.splits.ScaffoldSplitter()
train, valid, test = splitter.train_valid_test_split(
    dataset,
    frac_train=0.8,
    frac_valid=0.1,
    frac_test=0.1
)

# 随机拆分（非分子数据）
splitter = dc.splits.RandomSplitter()
train, test = splitter.train_test_split(dataset)

# 分层拆分（不平衡分类）
splitter = dc.splits.RandomStratifiedSplitter()
train, test = splitter.train_test_split(dataset)
```

**可用拆分器**：
- `ScaffoldSplitter`：按分子骨架拆分（防止泄露）
- `ButinaSplitter`：基于聚类的分子拆分
- `MaxMinSplitter`：最大化集合间多样性
- `RandomSplitter`：随机拆分
- `RandomStratifiedSplitter`：保持类别分布

### 4. 模型选择与训练

#### 快速模型选择指南

| 数据集规模 | 任务类型 | 推荐模型 | 特征化工具 |
|-------------|------|-------------------|------------|
| < 1K 样本 | 任意 | SklearnModel (RandomForest) | CircularFingerprint |
| 1K-100K | 分类/回归 | GBDTModel 或 MultitaskRegressor | CircularFingerprint |
| > 100K | 分子属性 | GCNModel, AttentiveFPModel, DMPNNModel | MolGraphConvFeaturizer |
| 任意（推荐小型） | 迁移学习 | ChemBERTa, GROVER, MolFormer | 模型专用 |
| 晶体结构 | 材料属性 | CGCNNModel, MEGNetModel | 结构特征 |
| 蛋白质序列 | 蛋白质属性 | ProtBERT | 序列特征 |

#### 示例：传统机器学习
```python
from sklearn.ensemble import RandomForestRegressor

# 封装scikit-learn模型
sklearn_model = RandomForestRegressor(n_estimators=100)
model = dc.models.SklearnModel(model=sklearn_model)
model.fit(train)
```

#### 示例：深度学习
```python
# 多任务回归器（指纹特征）
model = dc.models.MultitaskRegressor(
    n_tasks=2,
    n_features=2048,
    layer_sizes=[1000, 500],
    dropouts=0.25,
    learning_rate=0.001
)
model.fit(train, nb_epoch=50)
```

#### 示例：图神经网络
```python
# 图卷积网络
model = dc.models.GCNModel(
    n_tasks=1,
    mode='regression',
    batch_size=128,
    learning_rate=0.001
)
model.fit(train, nb_epoch=50)

# 图注意力网络
model = dc.models.GATModel(n_tasks=1, mode='classification')
model.fit(train, nb_epoch=50)

# 注意力指纹模型
model = dc.models.AttentiveFPModel(n_tasks=1, mode='regression')
model.fit(train, nb_epoch=50)
```

### 5. MoleculeNet 基准测试

快速访问30+个标准化训练/验证/测试拆分的基准数据集：

```python
# 加载基准数据集
tasks, datasets, transformers = dc.molnet.load_tox21(
    featurizer='GraphConv',  # 或 'ECFP', 'Weave', 'Raw'
    splitter='scaffold',     # 或 'random', 'stratified'
    reload=False
)
train, valid, test = datasets

# 训练与评估
model = dc.models.GCNModel(n_tasks=len(tasks), mode='classification')
model.fit(train, nb_epoch=50)

metric = dc.metrics.Metric(dc.metrics.roc_auc_score)
test_score = model.evaluate(test, [metric])
```

**常用数据集**：
- **分类**：`load_tox21()`, `load_bbbp()`, `load_hiv()`, `load_clintox()`
- **回归**：`load_delaney()`, `load_freesolv()`, `load_lipo()`
- **量子属性**：`load_qm7()`, `load_qm8()`, `load_qm9()`
- **材料**：`load_perovskite()`, `load_bandgap()`, `load_mp_formation_energy()`

完整数据集列表见 `references/api_reference.md`

### 6. 迁移学习

利用预训练模型提升性能，尤其适用于小数据集：

```python
# ChemBERTa（基于7700万分子预训练的BERT）
model = dc.models.HuggingFaceModel(
    model='seyonec/ChemBERTa-zinc-base-v1',
    task='classification',
    n_tasks=1,
    learning_rate=2e-5  # 微调时降低学习率
)
model.fit(train, nb_epoch=10)

# GROVER（基于1000万分子预训练的图变换器）
model = dc.models.GroverModel(
    task='regression',
    n_tasks=1
)
model.fit(train, nb_epoch=20)
```

**迁移学习适用场景**：
- 小型数据集（< 1000样本）
- 新型分子骨架
- 有限计算资源
- 需要快速原型验证

使用 `scripts/transfer_learning.py` 脚本获取引导式迁移学习工作流。

### 7. 模型评估

```python
# 定义评估指标
classification_metrics = [
    dc.metrics.Metric(dc.metrics.roc_auc_score, name='ROC-AUC'),
    dc.metrics.Metric(dc.metrics.accuracy_score, name='准确率'),
    dc.metrics.Metric(dc.metrics.f1_score, name='F1值')
]

regression_metrics = [
    dc.metrics.Metric(dc.metrics.r2_score, name='R²'),
    dc.metrics.Metric(dc.metrics.mean_absolute_error, name='平均绝对误差'),
    dc.metrics.Metric(dc.metrics.root_mean_squared_error, name='均方根误差')
]

# 评估模型
train_scores = model.evaluate(train, classification_metrics)
test_scores = model.evaluate(test, classification_metrics)
```

### 8. 预测应用

```python
# 在测试集预测
predictions = model.predict(test)

# 预测新分子
new_smiles = ['CCO', 'c1ccccc1', 'CC(C)O']
new_features = featurizer.featurize(new_smiles)
new_dataset = dc.data.NumpyDataset(X=new_features)

# 应用与训练相同的转换器
for transformer in transformers:
    new_dataset = transformer.transform(new_dataset)

predictions = model.predict(new_dataset)
```

## 典型工作流

### 工作流A：快速基准评估

标准基准测试模型评估流程：

```python
import deepchem as dc

# 1. 加载基准数据
tasks, datasets, _ = dc.molnet.load_bbbp(
    featurizer='GraphConv',
    splitter='scaffold'
)
train, valid, test = datasets

# 2. 训练模型
model = dc.models.GCNModel(n_tasks=len(tasks), mode='classification')
model.fit(train, nb_epoch=50)

# 3. 评估
metric = dc.metrics.Metric(dc.metrics.roc_auc_score)
test_score = model.evaluate(test, [metric])
print(f"测试集 ROC-AUC: {test_score}")
```

### 工作流B：自定义数据预测

自定义分子数据集训练流程：

```python
import deepchem as dc

# 1. 加载并特征化数据
featurizer = dc.feat.CircularFingerprint(radius=2, size=2048)
loader = dc.data.CSVLoader(
    tasks=['activity'],
    feature_field='smiles',
    featurizer=featurizer
)
dataset = loader.create_dataset('my_molecules.csv')

# 2. 拆分数据（分子数据务必用ScaffoldSplitter!）
splitter = dc.splits.ScaffoldSplitter()
train, valid, test = splitter.train_valid_test_split(dataset)

# 3. 标准化（推荐步骤）
transformers = [dc.trans.NormalizationTransformer(
    transform_y=True, dataset=train
)]
for transformer in transformers:
    train = transformer.transform(train)
    valid = transformer.transform(valid)
    test = transformer.transform(test)

# 4. 训练模型
model = dc.models.MultitaskRegressor(
    n_tasks=1,
    n_features=2048,
    layer_sizes=[1000, 500],
    dropouts=0.25
)
model.fit(train, nb_epoch=50)

# 5. 评估
metric = dc.metrics.Metric(dc.metrics.r2_score)
test_score = model.evaluate(test, [metric])
```

### 工作流C：小数据集迁移学习

预训练模型微调流程：

```python
import deepchem as dc

# 1. 加载数据（预训练模型通常需要原始SMILES）
loader = dc.data.CSVLoader(
    tasks=['activity'],
    feature_field='smiles',
    featurizer=dc.feat.DummyFeaturizer()  # 模型自行处理特征化
)
dataset = loader.create_dataset('small_dataset.csv')

# 2. 拆分数据
splitter = dc.splits.ScaffoldSplitter()
train, test = splitter.train_test_split(dataset)

# 3. 加载预训练模型
model = dc.models.HuggingFaceModel(
    model='seyonec/ChemBERTa-zinc-base-v1',
    task='classification',
    n_tasks=1,
    learning_rate=2e-5
)

# 4. 微调
model.fit(train, nb_epoch=10)

# 5. 评估
predictions = model.predict(test)
```

更多工作流示例（含分子生成、材料科学、蛋白质分析）见 `references/workflows.md`

## 示例脚本

本技能包含 `scripts/` 目录下三个生产就绪脚本：

### 1. `predict_solubility.py`
训练并评估溶解度预测模型，支持Delaney基准或自定义CSV数据。

```bash
# 使用Delaney基准
python scripts/predict_solubility.py

# 使用自定义数据
python scripts/predict_solubility.py \
    --data my_data.csv \
    --smiles-col smiles \
    --target-col solubility \
    --predict "CCO" "c1ccccc1"
```

### 2. `graph_neural_network.py`
在分子数据上训练各类图神经网络架构。

```bash
# 在Tox21上训练GCN
python scripts/graph_neural_network.py --model gcn --dataset tox21

# 在自定义数据上训练AttentiveFP
python scripts/graph_neural_network.py \
    --model attentivefp \
    --data molecules.csv \
    --task-type regression \
    --targets activity \
    --epochs 100
```

### 3. `transfer_learning.py`
在分子属性预测任务上微调预训练模型（ChemBERTa、GROVER）。

```bash
# 在BBBP上微调ChemBERTa
python scripts/transfer_learning.py --model chemberta --dataset bbbp

# 在自定义数据上微调GROVER
python scripts/transfer_learning.py \
    --model grover \
    --data small_dataset.csv \
    --target activity \
    --task-type classification \
    --epochs 20
```

## 通用模式与最佳实践

### 模式1：分子数据始终使用骨架拆分
```python
# 正确：防止数据泄露
splitter = dc.splits.ScaffoldSplitter()
train, test = splitter.train_test_split(dataset)

# 错误：相似分子可能同时出现在训练集和测试集
splitter = dc.splits.RandomSplitter()
train, test = splitter.train_test_split(dataset)
```

### 模式2：标准化特征和目标值
```python
transformers = [
    dc.trans.NormalizationTransformer(
        transform_y=True,  # 同时标准化目标值
        dataset=train
    )
]
for transformer in transformers:
    train = transformer.transform(train)
    test = transformer.transform(test)
```

### 模式3：从简单模型开始逐步扩展
1. 从随机森林+分子指纹开始（快速基线）
2. 若效果良好尝试XGBoost/LightGBM
3. 样本量>5K时转向深度学习（MultitaskRegressor）
4. 样本量>10K时尝试图神经网络
5. 小数据集或新型骨架使用

# 选项二：使用平衡指标
metric = dc.metrics.Metric(dc.metrics.balanced_accuracy_score)
```

### 模式五：避免内存问题
```python
# 大型数据集使用DiskDataset
dataset = dc.data.DiskDataset.from_numpy(X, y, w, ids)

# 使用更小的批处理大小
model = dc.models.GCNModel(batch_size=32)  # 替代128
```

## 常见陷阱

### 问题一：药物发现中的数据泄露
**问题**：随机划分会导致训练集/测试集中出现相似分子。  
**解决方案**：分子数据集始终使用`ScaffoldSplitter`。

### 问题二：图神经网络性能低于分子指纹
**问题**：图神经网络表现不如简单的分子指纹方法。  
**解决方案**：
- 确保数据集足够大（通常需>10K样本）
- 增加训练轮次（50-100轮）
- 尝试不同架构（用AttentiveFP/DMPNN替代GCN）
- 使用预训练模型（GROVER）

### 问题三：小数据集过拟合
**问题**：模型过度记忆训练数据。  
**解决方案**：
- 加强正则化（dropout提升至0.5）
- 使用更简单模型（用随机森林替代深度学习）
- 应用迁移学习（ChemBERTa, GROVER）
- 收集更多数据

### 问题四：导入错误
**问题**：模块未找到错误。  
**解决方案**：确保安装DeepChem及必要依赖：
```bash
uv pip install deepchem
# PyTorch模型支持
uv pip install deepchem[torch]
# 完整功能支持
uv pip install deepchem[all]
```

## 参考文档

本技能包含完整参考文档：

### `references/api_reference.md`
完整API文档包含：
- 所有数据加载器及其用例
- 数据集类及适用场景
- 完整特征化工具目录与选择指南
- 按类别组织的模型目录（50+模型）
- MoleculeNet数据集说明
- 指标与评估函数
- 通用代码模式

**参考时机**：需查询具体API细节、参数名称或探索可用选项时。

### `references/workflows.md`
八个端到端详细工作流：
1. 基于SMILES的分子属性预测
2. 使用MoleculeNet基准测试
3. 超参数优化
4. 预训练模型迁移学习
5. GAN分子生成
6. 材料属性预测
7. 蛋白质序列分析
8. 自定义模型集成

**参考时机**：实现完整解决方案时作为模板使用。

## 安装说明

基础安装：
```bash
uv pip install deepchem
```

PyTorch模型支持（GCN, GAT等）：
```bash
uv pip install deepchem[torch]
```

完整功能支持：
```bash
uv pip install deepchem[all]
```

若遇导入错误，用户可能需要特定依赖。详细安装说明请查阅DeepChem文档。

## 附加资源

- 官方文档：https://deepchem.readthedocs.io/
- GitHub仓库：https://github.com/deepchem/deepchem
- 教程：https://deepchem.readthedocs.io/en/latest/get_started/tutorials.html
- 论文：《MoleculeNet: A Benchmark for Molecular Machine Learning》
