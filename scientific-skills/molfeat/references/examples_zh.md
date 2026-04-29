# Molfeat 使用示例

本文档提供了常见 molfeat 使用场景的实用示例。

## 安装

```bash
# 推荐：使用 conda/mamba
mamba install -c conda-forge molfeat

# 替代方案：使用 pip
pip install molfeat

# 安装所有可选依赖
pip install "molfeat[all]"

# 安装特定依赖
pip install "molfeat[dgl]"          # 用于 GNN 模型
pip install "molfeat[graphormer]"   # 用于 Graphormer
pip install "molfeat[transformer]"  # 用于 ChemBERTa, ChemGPT
```

---

## 快速入门

### 基础特征化工作流

```python
import datamol as dm
from molfeat.calc import FPCalculator
from molfeat.trans import MoleculeTransformer

# 加载示例数据
data = dm.data.freesolv().sample(100).smiles.values

# 单分子特征化
calc = FPCalculator("ecfp")
features_single = calc(data[0])
print(f"单分子特征维度: {features_single.shape}")
# 输出: (2048,)

# 批量特征化（并行处理）
transformer = MoleculeTransformer(calc, n_jobs=-1)
features_batch = transformer(data)
print(f"批量特征维度: {features_batch.shape}")
# 输出: (100, 2048)
```

---

## 计算器示例

### 指纹计算器

```python
from molfeat.calc import FPCalculator

# ECFP（扩展连通性指纹）
ecfp = FPCalculator("ecfp", radius=3, fpSize=2048)
fp = ecfp("CCO")  # 乙醇
print(f"ECFP 维度: {fp.shape}")  # (2048,)

# MACCS 键
maccs = FPCalculator("maccs")
fp = maccs("c1ccccc1")  # 苯
print(f"MACCS 维度: {fp.shape}")  # (167,)

# 计数型指纹
ecfp_count = FPCalculator("ecfp-count", radius=3)
fp_count = ecfp_count("CC(C)CC(C)C")  # 非二进制计数

# MAP4 指纹
map4 = FPCalculator("map4")
fp = map4("CC(=O)Oc1ccccc1C(=O)O")  # 阿司匹林
```

### 描述符计算器

```python
from molfeat.calc import RDKitDescriptors2D, MordredDescriptors

# RDKit 2D 描述符（200+ 属性）
desc2d = RDKitDescriptors2D()
descriptors = desc2d("CCO")
print(f"2D 描述符数量: {len(descriptors)}")

# 获取描述符名称
names = desc2d.columns
print(f"前5个描述符: {names[:5]}")

# Mordred 描述符（1800+ 属性）
mordred = MordredDescriptors()
descriptors = mordred("c1ccccc1O")  # 苯酚
print(f"Mordred 描述符数量: {len(descriptors)}")
```

### 药效团计算器

```python
from molfeat.calc import CATSCalculator

# 2D CATS 描述符
cats = CATSCalculator(mode="2D", scale="raw")
descriptors = cats("CC(C)Cc1ccc(C)cc1C")  # 伞花烃
print(f"CATS 描述符维度: {descriptors.shape}")  # (21,)

# 3D CATS 描述符（需要构象）
cats3d = CATSCalculator(mode="3D", scale="num")
```

---

## 转换器示例

### 基础转换器用法

```python
from molfeat.trans import MoleculeTransformer
from molfeat.calc import FPCalculator
import datamol as dm

# 准备数据
smiles_list = [
    "CCO",
    "CC(=O)O",
    "c1ccccc1",
    "CC(C)O",
    "CCCC"
]

# 创建转换器
calc = FPCalculator("ecfp")
transformer = MoleculeTransformer(calc, n_jobs=-1)

# 转换分子
features = transformer(smiles_list)
print(f"特征维度: {features.shape}")  # (5, 2048)
```

### 错误处理

```python
# 优雅处理无效 SMILES
smiles_with_errors = [
    "CCO",           # 有效
    "invalid",       # 无效
    "CC(=O)O",       # 有效
    "xyz123",        # 无效
]

transformer = MoleculeTransformer(
    FPCalculator("ecfp"),
    n_jobs=-1,
    verbose=True,           # 记录错误
    ignore_errors=True      # 出错时继续执行
)

features = transformer(smiles_with_errors)
# 返回：失败分子对应位置为 None 的数组
print(features)  # [array(...), None, array(...), None]
```

### 组合多个特征化器

```python
from molfeat.trans import FeatConcat, MoleculeTransformer
from molfeat.calc import FPCalculator

# 组合 MACCS (167) + ECFP (2048) = 2215 维
concat_calc = FeatConcat([
    FPCalculator("maccs"),
    FPCalculator("ecfp", radius=3, fpSize=2048)
])

transformer = MoleculeTransformer(concat_calc, n_jobs=-1)
features = transformer(smiles_list)
print(f"组合特征维度: {features.shape}")  # (n, 2215)

# 三重组合
triple_concat = FeatConcat([
    FPCalculator("maccs"),
    FPCalculator("ecfp"),
    FPCalculator("rdkit")
])
```

### 保存和加载配置

```python
from molfeat.trans import MoleculeTransformer
from molfeat.calc import FPCalculator

# 创建并保存转换器
transformer = MoleculeTransformer(
    FPCalculator("ecfp", radius=3, fpSize=2048),
    n_jobs=-1
)

# 保存为 YAML
transformer.to_state_yaml_file("my_featurizer.yml")

# 保存为 JSON
transformer.to_state_json_file("my_featurizer.json")

# 从保存状态加载
loaded_transformer = MoleculeTransformer.from_state_yaml_file("my_featurizer.yml")

# 使用加载的转换器
features = loaded_transformer(smiles_list)
```

---

## 预训练模型示例

### 使用模型仓库

```python
from molfeat.store.modelstore import ModelStore

# 初始化模型仓库
store = ModelStore()

# 列出所有可用模型
print(f"可用模型总数: {len(store.available_models)}")

# 搜索特定模型
chemberta_models = store.search(name="ChemBERTa")
for model in chemberta_models:
    print(f"- {model.name}: {model.description}")

# 获取模型信息
model_card = store.search(name="ChemBERTa-77M-MLM")[0]
print(f"模型: {model_card.name}")
print(f"版本: {model_card.version}")
print(f"作者: {model_card.authors}")

# 查看使用说明
model_card.usage()

# 直接加载模型
transformer = store.load("ChemBERTa-77M-MLM")
```

### ChemBERTa 嵌入

```python
from molfeat.trans.pretrained import PretrainedMolTransformer

# 加载 ChemBERTa 模型
chemberta = PretrainedMolTransformer("ChemBERTa-77M-MLM", n_jobs=-1)

# 生成嵌入向量
smiles = ["CCO", "CC(=O)O", "c1ccccc1"]
embeddings = chemberta(smiles)
print(f"ChemBERTa 嵌入维度: {embeddings.shape}")
# 输出: (3, 768) - 768 维嵌入向量

# 在机器学习流程中使用
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    embeddings, labels, test_size=0.2
)

clf = RandomForestClassifier()
clf.fit(X_train, y_train)
predictions = clf.predict(X_test)
```

### ChemGPT 模型

```python
# 小模型 (4.7M 参数)
chemgpt_small = PretrainedMolTransformer("ChemGPT-4.7M", n_jobs=-1)

# 中模型 (19M 参数)
chemgpt_medium = PretrainedMolTransformer("ChemGPT-19M", n_jobs=-1)

# 大模型 (1.2B 参数)
chemgpt_large = PretrainedMolTransformer("ChemGPT-1.2B", n_jobs=-1)

# 生成嵌入向量
embeddings = chemgpt_small(smiles)
```

### 图神经网络模型

```python
# 不同预训练目标的 GIN 模型
gin_masking = PretrainedMolTransformer("gin-supervised-masking", n_jobs=-1)
gin_infomax = PretrainedMolTransformer("gin-supervised-infomax", n_jobs=-1)
gin_edgepred = PretrainedMolTransformer("gin-supervised-edgepred", n_jobs=-1)

# 生成图嵌入向量
embeddings = gin_masking(smiles)
print(f"GIN 嵌入维度: {embeddings.shape}")

# Graphormer（用于量子化学）
graphormer = PretrainedMolTransformer("Graphormer-pcqm4mv2", n_jobs=-1)
embeddings = graphormer(smiles)
```

---

## 机器学习集成

### Scikit-learn 流程

```python
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score
from molfeat.trans import MoleculeTransformer
from molfeat.calc import FPCalculator

# 创建机器学习流程
pipeline = Pipeline([
    ('featurizer', MoleculeTransformer(FPCalculator("ecfp"), n_jobs=-1)),
    ('classifier', RandomForestClassifier(n_estimators=100))
])

# 训练和评估
pipeline.fit(smiles_train, y_train)
predictions = pipeline.predict(smiles_test)

# 交叉验证
scores = cross_val_score(pipeline, smiles_all, y_all, cv=5)
print(f"交叉验证分数: {scores.mean():.3f} (+/- {scores.std():.3f})")
```

### 超参数网格搜索

```python
from sklearn.model_selection import GridSearchCV
from sklearn.svm import SVC

# 定义流程
pipeline = Pipeline([
    ('featurizer', MoleculeTransformer(FPCalculator("ecfp"), n_jobs=-1)),
    ('classifier', SVC())
])

# 定义参数网格
param_grid = {
    'classifier__C': [0.1, 1, 10],
    'classifier__kernel': ['rbf', 'linear'],
    'classifier__gamma': ['scale', 'auto']
}

# 网格搜索
grid_search = GridSearchCV(pipeline, param_grid, cv=5, n_jobs=-1)
grid_search.fit(smiles_train, y_train)

print(f"最佳参数: {grid_search.best_params_}")
print(f"最佳分数: {grid_search.best_score_:.3f}")
```

### 多特征化器比较

```python
from sklearn.metrics import roc_auc_score

# 测试不同特征化器
featurizers = {
    'ECFP': FPCalculator("ecfp"),
    'MACCS': FPCalculator("maccs"),
    'RDKit': FPCalculator("rdkit"),
    'Descriptors': RDKitDescriptors2D(),
    'Combined': FeatConcat([
        FPCalculator("maccs"),
        FPCalculator("ecfp")
    ])
}

results = {}
for name, calc in featurizers.items():
    transformer = MoleculeTransformer(calc, n_jobs=-1)
    X_train = transformer(smiles_train)
    X_test = transformer(smiles_test)

    clf = RandomForestClassifier(n_estimators=100)
    clf.fit(X_train, y_train)

    y_pred = clf.predict_proba(X_test)[:, 1]
    auc = roc_auc_score(y_test, y_pred)
    results[name] = auc

    print(f"{name}: AUC = {auc:.3f}")
```

### PyTorch 深度学习

```python
import torch
import torch.nn as nn
from torch.utils.data import Dataset, DataLoader
from molfeat.trans import MoleculeTransformer
from molfeat.calc import FPCalculator

# 自定义数据集
class MoleculeDataset(Dataset):
    def __init__(self, smiles, labels, transformer):
        self.features = transformer(smiles)
        self.labels = torch.tensor(labels, dtype=torch.float32)

    def __len__(self):
        return len(self.labels)

    def __getitem__(self, idx):
        return (
            torch.tensor(self.features[idx], dtype=torch.float32),
            self.labels[idx]
        )

# 准备数据
transformer = MoleculeTransformer(FPCalculator("ecfp"), n_jobs=-1)
train_dataset = MoleculeDataset(smiles_train, y_train, transformer)
train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)

# 简单神经网络
class MoleculeClassifier(nn.Module):
    def __init__(self, input_dim):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(input_dim, 512),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(512, 256),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(256, 1),
            nn.Sigmoid()
        )

    def forward(self, x):
        return self.network(x)

# 训练模型
model = MoleculeClassifier(input_dim=2048)
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
criterion = nn.BCELoss()

for epoch in range(10):
    for batch_features, batch_labels in train_loader:
        optimizer.zero_grad()
        outputs = model(batch_features).squeeze()
        loss = criterion(outputs, batch_labels)
        loss.backward()
        optimizer.step()
```

---

## 高级使用模式

### 自定义预处理

```python
from molfeat.trans import MoleculeTransformer
import datamol as dm

class CustomTransformer(MoleculeTransformer):
    def preprocess(self, mol):
        """自定义预处理：标准化分子"""
        if isinstance(mol, str):
            mol = dm.to_mol(mol)

        # 标准化
        mol = dm.standardize_mol(mol)

        # 去除盐
        mol = dm.remove_salts(mol)

        return mol

# 使用自定义转换器
transformer = CustomTransformer(FPCalculator("ecfp"), n_jobs=-1)
features = transformer(smiles_list)
```

### 含构象的特征化

```python
import datamol as dm
from molfeat.calc import RDKitDescriptors3D

# 生成构象
def prepare_3d_mol(smiles):
    mol = dm.to_mol(smiles)
    mol = dm.add_hs(mol)
    mol = dm.conform.generate_conformers(mol, n_confs=1)
    return mol

# 3D 描述符
calc_3d = RDKitDescriptors3D()

smiles = "CC(C)Cc1ccc(C)cc1C"
mol_3d = prepare_3d_mol(smiles)
descriptors_3d = calc_3d(mol_3d)
```

### 并行批处理

```python
from molfeat.trans import MoleculeTransformer
from molfeat.calc import FPCalculator
import time

# 大型数据集
smiles_large = load_large_dataset()  # 例如 100,000 个分子

# 测试不同并行级别
for n_jobs in [1, 2, 4, -1]:
    transformer = MoleculeTransformer(
        FPCalculator("ecfp"),
        n_jobs=n_jobs
    )

    start = time.time()
    features = transformer(smiles_large)
    elapsed = time.time() - start

    print(f"n_jobs={n_jobs}: {elapsed:.2f}s")
```

### 昂贵操作缓存

```python
from molfeat.trans.pretrained import PretrainedMolTransformer
import pickle

# 加载昂贵的预训练模型
transformer = PretrainedMolTransformer("ChemBERTa-77M-MLM", n_jobs=-1)

# 缓存嵌入向量以便复用
cache_file = "embeddings_cache.pkl"

try:
    # 尝试加载缓存的嵌入向量
    with open(cache_file, "rb") as f:
        embeddings = pickle.load(f)
    print("已加载缓存的嵌入向量")
except FileNotFoundError:
    # 计算并缓存
    embeddings = transformer(sm

# 1. 准备训练数据（已知活性/非活性分子）
train_smiles = load_training_data()
train_labels = load_training_labels()  # 1=活性, 0=非活性

# 2. 特征化训练集
transformer = MoleculeTransformer(FPCalculator("ecfp"), n_jobs=-1)
X_train = transformer(train_smiles)

# 3. 训练分类器
clf = RandomForestClassifier(n_estimators=500, n_jobs=-1)
clf.fit(X_train, train_labels)

# 4. 特征化筛选库
screening_smiles = load_screening_library()  # 例如：100万化合物
X_screen = transformer(screening_smiles)

# 5. 预测并排序
predictions = clf.predict_proba(X_screen)[:, 1]
ranked_indices = predictions.argsort()[::-1]

# 6. 获取命中分子
top_n = 1000
top_hits = [screening_smiles[i] for i in ranked_indices[:top_n]]

### QSAR 模型构建

```python
from molfeat.calc import RDKitDescriptors2D
from sklearn.linear_model import Ridge
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import cross_val_score
import numpy as np

# 加载 QSAR 数据集
smiles = load_molecules()
y = load_activity_values()  # 例如：IC50, logP

# 使用可解释描述符进行特征化
transformer = MoleculeTransformer(RDKitDescriptors2D(), n_jobs=-1)
X = transformer(smiles)

# 标准化特征
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 构建线性模型
model = Ridge(alpha=1.0)
scores = cross_val_score(model, X_scaled, y, cv=5, scoring='r2')
print(f"R² = {scores.mean():.3f} (+/- {scores.std():.3f})")

# 拟合最终模型
model.fit(X_scaled, y)

# 解释特征重要性
feature_names = transformer.featurizer.columns
importance = np.abs(model.coef_)
top_features_idx = importance.argsort()[-10:][::-1]

print("Top 10 important features:")
for idx in top_features_idx:
    print(f"  {feature_names[idx]}: {model.coef_[idx]:.3f}")
```

### 相似性搜索

```python
from molfeat.calc import FPCalculator
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

# 查询分子
query_smiles = "CC(=O)Oc1ccccc1C(=O)O"  # 阿司匹林

# 分子数据库
database_smiles = load_molecule_database()  # 大型化合物库

# 计算指纹
calc = FPCalculator("ecfp")
query_fp = calc(query_smiles).reshape(1, -1)

transformer = MoleculeTransformer(calc, n_jobs=-1)
database_fps = transformer(database_smiles)

# 计算相似性
similarities = cosine_similarity(query_fp, database_fps)[0]

# 查找最相似分子
top_k = 10
top_indices = similarities.argsort()[-top_k:][::-1]

print(f"Top {top_k} similar molecules:")
for i, idx in enumerate(top_indices, 1):
    print(f"{i}. {database_smiles[idx]} (相似度: {similarities[idx]:.3f})")
```

---

## 故障排除

### 处理无效分子

```python
# 使用 ignore_errors 跳过无效分子
transformer = MoleculeTransformer(
    FPCalculator("ecfp"),
    ignore_errors=True,
    verbose=True
)

# 转换后过滤 None 值
features = transformer(smiles_list)
valid_mask = [f is not None for f in features]
valid_features = [f for f in features if f is not None]
valid_smiles = [s for s, m in zip(smiles_list, valid_mask) if m]
```

### 大数据集的内存管理

```python
# 对超大数据集分块处理
def featurize_in_chunks(smiles_list, transformer, chunk_size=10000):
    all_features = []

    for i in range(0, len(smiles_list), chunk_size):
        chunk = smiles_list[i:i+chunk_size]
        features = transformer(chunk)
        all_features.append(features)
        print(f"Processed {i+len(chunk)}/{len(smiles_list)}")

    return np.vstack(all_features)

# 用于大型数据集
features = featurize_in_chunks(large_smiles_list, transformer)
```

### 可复现性

```python
import random
import numpy as np
import torch

# 设置所有随机种子
def set_seed(seed=42):
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)

set_seed(42)

# 保存精确配置
transformer.to_state_yaml_file("config.yml")

# 记录版本
import molfeat
print(f"molfeat version: {molfeat.__version__}")
```
