---
name: molfeat
description: 面向机器学习的分子特征化工具（100+种特征化器）。支持ECFP、MACCS、描述符、预训练模型（ChemBERTa），可将SMILES转换为特征，适用于QSAR和分子机器学习。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# Molfeat - 分子特征化中心

## 概述

Molfeat 是一个全面的 Python 分子特征化库，整合了 100 多种预训练嵌入和手工特征化器。可将化学结构（SMILES 字符串或 RDKit 分子）转换为数值表示，用于 QSAR 建模、虚拟筛选、相似性搜索和深度学习等机器学习任务。支持快速并行处理、scikit-learn 兼容的转换器，并内置缓存功能。

## 适用场景

本技能适用于以下场景：
- **分子机器学习**：构建 QSAR/QSPR 模型、属性预测
- **虚拟筛选**：对化合物库进行生物活性排序
- **相似性搜索**：查找结构相似分子
- **化学空间分析**：聚类、可视化、降维
- **深度学习**：在分子数据上训练神经网络
- **特征化流程**：将 SMILES 转换为机器学习就绪的表示形式
- **化学信息学**：任何需要分子特征提取的任务

## 安装

```bash
uv pip install molfeat

# 安装所有可选依赖
uv pip install "molfeat[all]"
```

**特定特征化器的可选依赖：**
- `molfeat[dgl]` - GNN 模型（GIN 变体）
- `molfeat[graphormer]` - Graphormer 模型
- `molfeat[transformer]` - ChemBERTa、ChemGPT、MolT5
- `molfeat[fcd]` - FCD 描述符
- `molfeat[map4]` - MAP4 指纹

## 核心概念

Molfeat 将特征化组织为三个层级类：

### 1. 计算器 (`molfeat.calc`)

可调用对象，将单个分子转换为特征向量。接受 RDKit `Chem.Mol` 对象或 SMILES 字符串。

**适用场景：**
- 单分子特征化
- 自定义处理循环
- 直接特征计算

**示例：**
```python
from molfeat.calc import FPCalculator

calc = FPCalculator("ecfp", radius=3, fpSize=2048)
features = calc("CCO")  # 返回 numpy 数组 (2048,)
```

### 2. 转换器 (`molfeat.trans`)

兼容 scikit-learn 的转换器，封装计算器实现批处理和并行化。

**适用场景：**
- 分子数据集的批量特征化
- 与 scikit-learn 流程集成
- 并行处理（自动利用 CPU）

**示例：**
```python
from molfeat.trans import MoleculeTransformer
from molfeat.calc import FPCalculator

transformer = MoleculeTransformer(FPCalculator("ecfp"), n_jobs=-1)
features = transformer(smiles_list)  # 并行处理
```

### 3. 预训练转换器 (`molfeat.trans.pretrained`)

专为深度学习模型设计的转换器，支持批量推理和缓存。

**适用场景：**
- 最先进的分子嵌入
- 从大型化学数据集迁移学习
- 深度学习特征提取

**示例：**
```python
from molfeat.trans.pretrained import PretrainedMolTransformer

transformer = PretrainedMolTransformer("ChemBERTa-77M-MLM", n_jobs=-1)
embeddings = transformer(smiles_list)  # 深度学习嵌入
```

## 快速入门工作流

### 基础特征化

```python
import datamol as dm
from molfeat.calc import FPCalculator
from molfeat.trans import MoleculeTransformer

# 加载分子数据
smiles = ["CCO", "CC(=O)O", "c1ccccc1", "CC(C)O"]

# 创建计算器和转换器
calc = FPCalculator("ecfp", radius=3)
transformer = MoleculeTransformer(calc, n_jobs=-1)

# 特征化分子
features = transformer(smiles)
print(f"形状: {features.shape}")  # (4, 2048)
```

### 保存与加载配置

```python
# 保存特征化器配置以实现可复现性
transformer.to_state_yaml_file("featurizer_config.yml")

# 重新加载精确配置
loaded = MoleculeTransformer.from_state_yaml_file("featurizer_config.yml")
```

### 优雅处理错误

```python
# 处理可能包含无效 SMILES 的数据集
transformer = MoleculeTransformer(
    calc,
    n_jobs=-1,
    ignore_errors=True,  # 出错时继续执行
    verbose=True          # 记录错误详情
)

features = transformer(smiles_with_errors)
# 失败分子返回 None
```

## 选择合适特征化器

### 传统机器学习（随机森林、SVM、XGBoost）

**从指纹开始：**
```python
# ECFP - 最流行，通用性强
FPCalculator("ecfp", radius=3, fpSize=2048)

# MACCS - 快速，适用于骨架跃迁
FPCalculator("maccs")

# MAP4 - 大规模筛选效率高
FPCalculator("map4")
```

**可解释模型：**
```python
# RDKit 2D 描述符（200+ 命名属性）
from molfeat.calc import RDKitDescriptors2D
RDKitDescriptors2D()

# Mordred（1800+ 全面描述符）
from molfeat.calc import MordredDescriptors
MordredDescriptors()
```

**组合多个特征化器：**
```python
from molfeat.trans import FeatConcat

concat = FeatConcat([
    FPCalculator("maccs"),      # 167 维
    FPCalculator("ecfp")         # 2048 维
])  # 结果：2215 维组合特征
```

### 深度学习

**基于 Transformer 的嵌入：**
```python
# ChemBERTa - 在 7700 万 PubChem 化合物上预训练
PretrainedMolTransformer("ChemBERTa-77M-MLM")

# ChemGPT - 自回归语言模型
PretrainedMolTransformer("ChemGPT-1.2B")
```

**图神经网络：**
```python
# 不同预训练目标的 GIN 模型
PretrainedMolTransformer("gin-supervised-masking")
PretrainedMolTransformer("gin-supervised-infomax")

# 量子化学 Graphormer
PretrainedMolTransformer("Graphormer-pcqm4mv2")
```

### 相似性搜索

```python
# ECFP - 通用性强，应用最广
FPCalculator("ecfp")

# MACCS - 快速，基于骨架相似性
FPCalculator("maccs")

# MAP4 - 大型数据库效率高
FPCalculator("map4")

# USR/USRCAT - 3D 形状相似性
from molfeat.calc import USRDescriptors
USRDescriptors()
```

### 基于药效团的方法

```python
# FCFP - 基于官能团
FPCalculator("fcfp")

# CATS - 药效团对分布
from molfeat.calc import CATSCalculator
CATSCalculator(mode="2D")

# Gobbi - 显式药效团特征
FPCalculator("gobbi2D")
```

## 常见工作流

### 构建 QSAR 模型

```python
from molfeat.trans import MoleculeTransformer
from molfeat.calc import FPCalculator
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import cross_val_score

# 特征化分子
transformer = MoleculeTransformer(FPCalculator("ecfp"), n_jobs=-1)
X = transformer(smiles_train)

# 训练模型
model = RandomForestRegressor(n_estimators=100)
scores = cross_val_score(model, X, y_train, cv=5)
print(f"R² = {scores.mean():.3f}")

# 保存配置用于部署
transformer.to_state_yaml_file("production_featurizer.yml")
```

### 虚拟筛选流程

```python
from sklearn.ensemble import RandomForestClassifier

# 在已知活性/非活性数据上训练
transformer = MoleculeTransformer(FPCalculator("ecfp"), n_jobs=-1)
X_train = transformer(train_smiles)
clf = RandomForestClassifier(n_estimators=500)
clf.fit(X_train, train_labels)

# 筛选大型化合物库
X_screen = transformer(screening_library)  # 例如 100 万化合物
predictions = clf.predict_proba(X_screen)[:, 1]

# 排序并选择前 1000 个候选分子
top_indices = predictions.argsort()[::-1][:1000]
top_hits = [screening_library[i] for i in top_indices]
```

### 相似性搜索

```python
from sklearn.metrics.pairwise import cosine_similarity

# 查询分子
calc = FPCalculator("ecfp")
query_fp = calc(query_smiles).reshape(1, -1)

# 数据库指纹
transformer = MoleculeTransformer(calc, n_jobs=-1)
database_fps = transformer(database_smiles)

# 计算相似度
similarities = cosine_similarity(query_fp, database_fps)[0]
top_similar = similarities.argsort()[-10:][::-1]
```

### Scikit-learn 流程集成

```python
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier

# 创建端到端流程
pipeline = Pipeline([
    ('featurizer', MoleculeTransformer(FPCalculator("ecfp"), n_jobs=-1)),
    ('classifier', RandomForestClassifier(n_estimators=100))
])

# 直接在 SMILES 上训练和预测
pipeline.fit(smiles_train, y_train)
predictions = pipeline.predict(smiles_test)
```

### 比较多个特征化器

```python
featurizers = {
    'ECFP': FPCalculator("ecfp"),
    'MACCS': FPCalculator("maccs"),
    'Descriptors': RDKitDescriptors2D(),
    'ChemBERTa': PretrainedMolTransformer("ChemBERTa-77M-MLM")
}

results = {}
for name, feat in featurizers.items():
    transformer = MoleculeTransformer(feat, n_jobs=-1)
    X = transformer(smiles)
    # 用您的 ML 模型评估
    score = evaluate_model(X, y)
    results[name] = score
```

## 探索可用特征化器

使用 ModelStore 浏览所有可用特征化器：

```python
from molfeat.store.modelstore import ModelStore

store = ModelStore()

# 列出所有可用模型
all_models = store.available_models
print(f"特征化器总数: {len(all_models)}")

# 搜索特定模型
chemberta_models = store.search(name="ChemBERTa")
for model in chemberta_models:
    print(f"- {model.name}: {model.description}")

# 获取使用信息
model_card = store.search(name="ChemBERTa-77M-MLM")[0]
model_card.usage()  # 显示使用示例

# 加载模型
transformer = store.load("ChemBERTa-77M-MLM")
```

## 高级功能

### 自定义预处理

```python
class CustomTransformer(MoleculeTransformer):
    def preprocess(self, mol):
        """自定义预处理流程"""
        if isinstance(mol, str):
            mol = dm.to_mol(mol)
        mol = dm.standardize_mol(mol)
        mol = dm.remove_salts(mol)
        return mol

transformer = CustomTransformer(FPCalculator("ecfp"), n_jobs=-1)
```

### 大数据集分批处理

```python
def featurize_in_chunks(smiles_list, transformer, chunk_size=10000):
    """分批处理大型数据集以管理内存"""
    all_features = []
    for i in range(0, len(smiles_list), chunk_size):
        chunk = smiles_list[i:i+chunk_size]
        features = transformer(chunk)
        all_features.append(features)
    return np.vstack(all_features)
```

### 缓存高开销嵌入

```python
import pickle

cache_file = "embeddings_cache.pkl"
transformer = PretrainedMolTransformer("ChemBERTa-77M-MLM", n_jobs=-1)

try:
    with open(cache_file, "rb") as f:
        embeddings = pickle.load(f)
except FileNotFoundError:
    embeddings = transformer(smiles_list)
    with open(cache_file, "wb") as f:
        pickle.dump(embeddings, f)
```

## 性能优化建议

1. **使用并行化**：设置 `n_jobs=-1` 利用所有 CPU 核心
2. **批处理**：批量处理分子而非循环
3. **选择合适的特征化器**：指纹比深度学习模型更快
4. **缓存预训练模型**：利用内置缓存实现重复使用
5. **使用 float32**：精度允许时设置 `dtype=np.float32`
6. **高效处理错误**：大型数据集使用 `ignore_errors=True`

## 常用特征化器参考

**常用特征化器速查表：**

| 特征化器 | 类型 | 维度 | 速度 | 适用场景 |
|------------|------|------------|-------|----------|
| `ecfp` | 指纹 | 2048 | 快 | 通用场景 |
| `maccs` | 指纹 | 167 | 极快 | 骨架相似性 |
| `desc2D` | 描述符 | 200+ | 快 | 可解释模型 |
| `mordred` | 描述符 | 1800+ | 中等 | 全面特征 |
| `map4` | 指纹 | 1024 | 快 | 大规模筛选 |
| `ChemBERTa-77M-MLM` | 深度学习 | 768 | 慢* | 迁移学习 |
| `gin-supervised-masking` | GNN | 可变 | 慢* | 图模型 |

*首次运行慢；后续运行受益于缓存

## 资源

本技能包含全面的参考文档：

### references/api_reference.md
完整 API 文档涵盖：
- `molfeat.calc` - 所有计算器类及参数
- `molfeat.trans` - 转换器类与方法
- `molfeat.store` - ModelStore 使用指南
- 常用模式与集成示例
- 性能优化技巧

**加载时机**：实现特定计算器、理解转换器参数或集成 scikit-learn/PyTorch 时参考。

### references/available_featurizers.md
按类别组织的 100+ 特征化器完整目录：
- Transformer 语言模型（ChemBERTa, ChemGPT）
- 图神经网络（GIN, Graphormer）
- 分子描述符（RDKit, Mordred）
- 指纹（ECFP, MACCS, MAP4 等 15+ 种）
- 药效团描述符（CATS, Gobbi）
- 形状描述符（USR, ElectroShape）
- 骨架描述符

**加载时机**：为特定任务选择最优特征化器、探索可用选项或理解特征化器特性时参考。

**搜索技巧**：使用 grep 查找特定类型特征化器：
```bash
grep -i "chembert" references/available_featurizers.md
grep -i "pharmacophore" references/available_featurizers.md
```

### references/examples.md
常见场景的实用代码示例：
- 安装与快速入门
- 计算器与转换器示例
- 预训练模型使用
- Scikit-learn 和 PyTorch 集成
- 虚拟筛选工作流
- QSAR 模型构建
- 相似性搜索
- 故障排除与最佳实践

**加载时机**：实现特定工作流、排查问题或学习 molfeat 模式时参考。

## 故障排除

### 无效分子处理
启用错误处理跳过无效 SMILES：
```python
transformer = MoleculeTransformer(
    calc,
    ignore_errors=True,
    verbose=True
)
```

### 大数据集内存问题
处理超过 10 万分子时采用分批或流式处理。

### 预训练模型依赖
部分模型需额外安装包：
```bash
uv pip install "molfeat[transformer]"  # ChemBERTa/ChemGPT
uv pip install "molfeat[dgl]"          # GIN 模型
```

### 可复现性
保存精确配置并记录版本：
```python
transformer.to_state_yaml_file("config.yml")
import molfeat
print(f"molfeat 版本: {molfeat.__version__}")
```

## 附加资源

- **官方文档**：https://molfeat-docs.datamol.io/
- **GitHub 仓库**：https://github.com/datamol-io/molfeat
- **PyPI 包**：https://pypi.org/project/molfeat/
- **教程**：https://portal.valencelabs.com/datamol/post/types-of-featurizers-b1e8HHrbFMkbun6
