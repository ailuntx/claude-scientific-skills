# DeepChem 工作流程

本文档提供了常见 DeepChem 用例的详细工作流程。

## 工作流程 1：基于 SMILES 的分子性质预测

**目标**：从 SMILES 字符串预测分子性质（如溶解度、毒性、活性）。

### 分步流程

#### 1. 准备数据
数据应为 CSV 格式，至少包含：
- 包含 SMILES 字符串的列
- 一个或多个包含性质值（目标）的列

示例 CSV 结构：
```csv
smiles,溶解度,毒性
CCO,-0.77,0
CC(=O)OC1=CC=CC=C1C(=O)O,-1.19,1
```

#### 2. 选择特征化器
决策树：
- **小型数据集 (<1K)**：使用 `CircularFingerprint` 或 `RDKitDescriptors`
- **中型数据集 (1K-100K)**：使用 `CircularFingerprint` 或 `MolGraphConvFeaturizer`
- **大型数据集 (>100K)**：使用基于图的特征化器 (`MolGraphConvFeaturizer`, `DMPNNFeaturizer`)
- **迁移学习**：使用预训练模型特征化器 (`GroverFeaturizer`)

#### 3. 加载并特征化数据
```python
import deepchem as dc

# 基于指纹的方法
featurizer = dc.feat.CircularFingerprint(radius=2, size=2048)
# 或基于图的方法
featurizer = dc.feat.MolGraphConvFeaturizer()

loader = dc.data.CSVLoader(
    tasks=['溶解度', '毒性'],  # 要预测的列名
    feature_field='smiles',     # 包含 SMILES 的列
    featurizer=featurizer
)
dataset = loader.create_dataset('data.csv')
```

#### 4. 拆分数据
**关键**：在药物发现中使用 `ScaffoldSplitter` 防止数据泄露。

```python
splitter = dc.splits.ScaffoldSplitter()
train, valid, test = splitter.train_valid_test_split(
    dataset,
    frac_train=0.8,
    frac_valid=0.1,
    frac_test=0.1
)
```

#### 5. 转换数据（可选但推荐）
```python
transformers = [
    dc.trans.NormalizationTransformer(
        transform_y=True,
        dataset=train
    )
]

for transformer in transformers:
    train = transformer.transform(train)
    valid = transformer.transform(valid)
    test = transformer.transform(test)
```

#### 6. 选择并训练模型
```python
# 用于指纹方法
model = dc.models.MultitaskRegressor(
    n_tasks=2,                 # 要预测的性质数量
    n_features=2048,           # 指纹大小
    layer_sizes=[1000, 500],   # 隐藏层大小
    dropouts=0.25,
    learning_rate=0.001
)

# 或用于图方法
model = dc.models.GCNModel(
    n_tasks=2,
    mode='regression',
    batch_size=128,
    learning_rate=0.001
)

# 训练
model.fit(train, nb_epoch=50)
```

#### 7. 评估
```python
metric = dc.metrics.Metric(dc.metrics.r2_score)
train_score = model.evaluate(train, [metric])
valid_score = model.evaluate(valid, [metric])
test_score = model.evaluate(test, [metric])

print(f"训练集 R²: {train_score}")
print(f"验证集 R²: {valid_score}")
print(f"测试集 R²: {test_score}")
```

#### 8. 进行预测
```python
# 对新分子进行预测
new_smiles = ['CCO', 'CC(C)O', 'c1ccccc1']
new_featurizer = dc.feat.CircularFingerprint(radius=2, size=2048)
new_features = new_featurizer.featurize(new_smiles)
new_dataset = dc.data.NumpyDataset(X=new_features)

# 应用相同的转换
for transformer in transformers:
    new_dataset = transformer.transform(new_dataset)

predictions = model.predict(new_dataset)
```

---

## 工作流程 2：使用 MoleculeNet 基准数据集

**目标**：在标准基准上快速训练和评估模型。

### 快速开始
```python
import deepchem as dc

# 加载基准数据集
tasks, datasets, transformers = dc.molnet.load_tox21(
    featurizer='GraphConv',
    splitter='scaffold'
)
train, valid, test = datasets

# 训练模型
model = dc.models.GCNModel(
    n_tasks=len(tasks),
    mode='classification'
)
model.fit(train, nb_epoch=50)

# 评估
metric = dc.metrics.Metric(dc.metrics.roc_auc_score)
test_score = model.evaluate(test, [metric])
print(f"测试集 ROC-AUC: {test_score}")
```

### 可用特征化器选项
调用 `load_*()` 函数时：
- `'ECFP'`：扩展连通性指纹（圆形指纹）
- `'GraphConv'`：图卷积特征
- `'Weave'`：编织特征
- `'Raw'`：原始 SMILES 字符串
- `'smiles2img'`：2D 分子图像

### 可用拆分器选项
- `'scaffold'`：基于骨架的拆分（推荐用于药物发现）
- `'random'`：随机拆分
- `'stratified'`：分层拆分（保留类别分布）
- `'butina'`：基于 Butina 聚类的拆分

---

## 工作流程 3：超参数优化

**目标**：系统性地寻找最优模型超参数。

### 使用 GridHyperparamOpt
```python
import deepchem as dc
import numpy as np

# 加载数据
tasks, datasets, transformers = dc.molnet.load_bbbp(
    featurizer='ECFP',
    splitter='scaffold'
)
train, valid, test = datasets

# 定义参数网格
params_dict = {
    'layer_sizes': [[1000], [1000, 500], [1000, 1000]],
    'dropouts': [0.0, 0.25, 0.5],
    'learning_rate': [0.001, 0.0001]
}

# 定义模型构建函数
def model_builder(model_params, model_dir):
    return dc.models.MultitaskClassifier(
        n_tasks=len(tasks),
        n_features=1024,
        **model_params
    )

# 设置优化器
metric = dc.metrics.Metric(dc.metrics.roc_auc_score)
optimizer = dc.hyper.GridHyperparamOpt(model_builder)

# 运行优化
best_model, best_params, all_results = optimizer.hyperparam_search(
    params_dict,
    train,
    valid,
    metric,
    transformers=transformers
)

print(f"最优参数: {best_params}")
print(f"最佳验证分数: {all_results['best_validation_score']}")
```

---

## 工作流程 4：使用预训练模型进行迁移学习

**目标**：利用预训练模型提升小数据集性能。

### 使用 ChemBERTa
```python
import deepchem as dc
from transformers import AutoTokenizer

# 加载数据
loader = dc.data.CSVLoader(
    tasks=['活性'],
    feature_field='smiles',
    featurizer=dc.feat.DummyFeaturizer()  # ChemBERTa 处理特征化
)
dataset = loader.create_dataset('data.csv')

# 拆分数据
splitter = dc.splits.ScaffoldSplitter()
train, test = splitter.train_test_split(dataset)

# 加载预训练 ChemBERTa
model = dc.models.HuggingFaceModel(
    model='seyonec/ChemBERTa-zinc-base-v1',
    task='regression',
    n_tasks=1
)

# 微调
model.fit(train, nb_epoch=10)

# 评估
predictions = model.predict(test)
```

### 使用 GROVER
```python
# GROVER：在分子图上预训练
model = dc.models.GroverModel(
    task='classification',
    n_tasks=1,
    model_dir='./grover_model'
)

# 在您的数据上微调
model.fit(train_dataset, nb_epoch=20)
```

---

## 工作流程 5：使用 GAN 生成分子

**目标**：生成具有所需性质的新分子。

### 基础 MolGAN
```python
import deepchem as dc

# 加载训练数据（生成器学习的分子）
tasks, datasets, _ = dc.molnet.load_qm9(
    featurizer='GraphConv',
    splitter='random'
)
train, _, _ = datasets

# 创建并训练 MolGAN
gan = dc.models.BasicMolGANModel(
    learning_rate=0.001,
    vertices=9,  # 分子最大原子数
    edges=5,     # 最大键数
    nodes=[128, 256, 512]
)

# 训练
gan.fit_gan(
    train,
    nb_epoch=100,
    generator_steps=0.2,
    checkpoint_interval=10
)

# 生成新分子
generated_molecules = gan.predict_gan_generator(1000)
```

### 条件生成
```python
# 用于目标性质生成
from deepchem.models.optimizers import ExponentialDecay

gan = dc.models.BasicMolGANModel(
    learning_rate=ExponentialDecay(0.001, 0.9, 1000),
    conditional=True  # 启用条件生成
)

# 带性质训练
gan.fit_gan(train, nb_epoch=100)

# 生成具有目标性质的分子
target_properties = np.array([[5.0, 300.0]])  # 例如 [logP, 分子量]
molecules = gan.predict_gan_generator(
    1000,
    conditional_inputs=target_properties
)
```

---

## 工作流程 6：材料性质预测

**目标**：预测晶体材料性质。

### 使用晶体图卷积网络
```python
import deepchem as dc

# 加载材料数据（CIF 格式的结构文件）
loader = dc.data.CIFLoader()
dataset = loader.create_dataset('materials.csv')

# 拆分数据
splitter = dc.splits.RandomSplitter()
train, test = splitter.train_test_split(dataset)

# 创建 CGCNN 模型
model = dc.models.CGCNNModel(
    n_tasks=1,
    mode='regression',
    batch_size=32,
    learning_rate=0.001
)

# 训练
model.fit(train, nb_epoch=100)

# 评估
metric = dc.metrics.Metric(dc.metrics.mae_score)
test_score = model.evaluate(test, [metric])
```

---

## 工作流程 7：蛋白质序列分析

**目标**：从序列预测蛋白质性质。

### 使用 ProtBERT
```python
import deepchem as dc

# 加载蛋白质序列数据
loader = dc.data.FASTALoader()
dataset = loader.create_dataset('proteins.fasta')

# 使用 ProtBERT
model = dc.models.HuggingFaceModel(
    model='Rostlab/prot_bert',
    task='classification',
    n_tasks=1
)

# 拆分并训练
splitter = dc.splits.RandomSplitter()
train, test = splitter.train_test_split(dataset)
model.fit(train, nb_epoch=5)

# 预测
predictions = model.predict(test)
```

---

## 工作流程 8：自定义模型集成

**目标**：在 DeepChem 中使用自定义 PyTorch/scikit-learn 模型。

### 封装 Scikit-Learn 模型
```python
from sklearn.ensemble import RandomForestRegressor
import deepchem as dc

# 创建 scikit-learn 模型
sklearn_model = RandomForestRegressor(
    n_estimators=100,
    max_depth=10,
    random_state=42
)

# 封装到 DeepChem
model = dc.models.SklearnModel(model=sklearn_model)

# 与 DeepChem 数据集配合使用
model.fit(train)
predictions = model.predict(test)

# 评估
metric = dc.metrics.Metric(dc.metrics.r2_score)
score = model.evaluate(test, [metric])
```

### 创建自定义 PyTorch 模型
```python
import torch
import torch.nn as nn
import deepchem as dc

class CustomNetwork(nn.Module):
    def __init__(self, n_features, n_tasks):
        super().__init__()
        self.fc1 = nn.Linear(n_features, 512)
        self.fc2 = nn.Linear(512, 256)
        self.fc3 = nn.Linear(256, n_tasks)
        self.relu = nn.ReLU()
        self.dropout = nn.Dropout(0.2)

    def forward(self, x):
        x = self.relu(self.fc1(x))
        x = self.dropout(x)
        x = self.relu(self.fc2(x))
        x = self.dropout(x)
        return self.fc3(x)

# 封装到 DeepChem TorchModel
model = dc.models.TorchModel(
    model=CustomNetwork(n_features=2048, n_tasks=1),
    loss=nn.MSELoss(),
    output_types=['prediction']
)

# 训练
model.fit(train, nb_epoch=50)
```

---

## 常见陷阱与解决方案

### 问题 1：药物发现中的数据泄露
**问题**：随机拆分导致相似分子同时出现在训练集
