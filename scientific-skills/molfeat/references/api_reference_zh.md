# Molfeat API 参考

## 核心模块

Molfeat 由多个关键模块组成，提供分子特征化的不同方面：

- **`molfeat.store`** - 管理模型加载、列举和注册
- **`molfeat.calc`** - 提供单分子特征化计算器
- **`molfeat.trans`** - 提供兼容 scikit-learn 的批量处理转换器
- **`molfeat.utils`** - 数据处理实用函数
- **`molfeat.viz`** - 分子特征可视化工具

---

## molfeat.calc - 计算器

计算器是可调用对象，将单个分子转换为特征向量。它们接受 RDKit `Chem.Mol` 对象或 SMILES 字符串作为输入。

### SerializableCalculator (基类)

所有计算器的抽象基类。子类化时必须实现：
- `__call__()` - 特征化必需方法
- `__len__()` - 可选，返回输出长度
- `columns` - 可选属性，返回特征名称
- `batch_compute()` - 可选，用于高效批量处理

**状态管理方法：**
- `to_state_json()` - 将计算器状态保存为 JSON
- `to_state_yaml()` - 将计算器状态保存为 YAML
- `from_state_dict()` - 从状态字典加载计算器
- `to_state_dict()` - 将计算器状态导出为字典

### FPCalculator

计算分子指纹。支持 15+ 种指纹方法。

**支持的指纹类型：**

**结构指纹：**
- `ecfp` - 扩展连通性指纹（圆形）
- `fcfp` - 功能类指纹
- `rdkit` - RDKit 拓扑指纹
- `maccs` - MACCS 键（166 位结构键）
- `avalon` - Avalon 指纹
- `pattern` - 模式指纹
- `layered` - 分层指纹

**基于原子的指纹：**
- `atompair` - 原子对指纹
- `atompair-count` - 计数型原子对
- `topological` - 拓扑扭转指纹
- `topological-count` - 计数型拓扑扭转

**专用指纹：**
- `map4` - 4 键内最小哈希原子对指纹
- `secfp` - SMILES 扩展连通性指纹
- `erg` - 扩展简化图
- `estate` - 电拓扑状态指数

**参数：**
- `method` (str) - 指纹类型名称
- `radius` (int) - 圆形指纹半径（默认：3）
- `fpSize` (int) - 指纹尺寸（默认：2048）
- `includeChirality` (bool) - 包含手性信息
- `counting` (bool) - 使用计数向量替代二进制

**用法：**
```python
from molfeat.calc import FPCalculator

# 创建指纹计算器
calc = FPCalculator("ecfp", radius=3, fpSize=2048)

# 计算单分子指纹
fp = calc("CCO")  # 返回 numpy 数组

# 获取指纹长度
length = len(calc)  # 2048

# 获取特征名称
names = calc.columns
```

**常见指纹维度：**
- MACCS：167 维
- ECFP（默认）：2048 维
- MAP4（默认）：1024 维

### 描述符计算器

**RDKitDescriptors2D**
使用 RDKit 计算 2D 分子描述符。

```python
from molfeat.calc import RDKitDescriptors2D

calc = RDKitDescriptors2D()
descriptors = calc("CCO")  # 返回 200+ 描述符
```

**RDKitDescriptors3D**
计算 3D 分子描述符（需要构象生成）。

**MordredDescriptors**
使用 Mordred 计算 1800+ 分子描述符。

```python
from molfeat.calc import MordredDescriptors

calc = MordredDescriptors()
descriptors = calc("CCO")
```

### 药效团计算器

**Pharmacophore2D**
RDKit 的 2D 药效团指纹生成。

**Pharmacophore3D**
多构象的共识药效团指纹。

**CATSCalculator**
计算化学高级模板搜索（CATS）描述符——药效团点对分布。

**参数：**
- `mode` - "2D" 或 "3D" 距离计算
- `dist_bins` - 点对分布的距离分箱
- `scale` - 缩放模式："raw"、"num" 或 "count"

```python
from molfeat.calc import CATSCalculator

calc = CATSCalculator(mode="2D", scale="raw")
cats = calc("CCO")  # 默认返回 21 个描述符
```

### 形状描述符

**USRDescriptors**
超快形状识别描述符（多种变体）。

**ElectroShapeDescriptors**
结合形状、手性和静电的静电形状描述符。

### 基于图的计算器

**ScaffoldKeyCalculator**
计算 40+ 基于骨架的分子属性。

**AtomCalculator**
用于图神经网络的原子级特征化。

**BondCalculator**
用于图神经网络的键级特征化。

### 实用函数

**get_calculator()**
按名称实例化计算器的工厂函数。

```python
from molfeat.calc import get_calculator

# 按名称实例化任意计算器
calc = get_calculator("ecfp", radius=3)
calc = get_calculator("maccs")
calc = get_calculator("desc2D")
```

遇到不支持的分子特征化器时抛出 `ValueError`。

---

## molfeat.trans - 转换器

转换器将计算器封装为完整的批量特征化流水线。

### MoleculeTransformer

兼容 scikit-learn 的批量分子特征化转换器。

**关键参数：**
- `featurizer` - 使用的计算器或特征化器
- `n_jobs` (int) - 并行任务数（-1 表示使用所有核心）
- `dtype` - 输出数据类型（numpy float32/64，torch 张量）
- `verbose` (bool) - 启用详细日志
- `ignore_errors` (bool) - 出错时继续（对失败分子返回 None）

**核心方法：**
- `transform(mols)` - 处理批次并返回特征表示
- `_transform(mol)` - 处理单分子特征化
- `__call__(mols)` - transform() 的便捷封装
- `preprocess(mol)` - 预处理输入分子（不会自动应用）
- `to_state_yaml_file(path)` - 保存转换器配置
- `from_state_yaml_file(path)` - 加载转换器配置

**用法：**
```python
from molfeat.calc import FPCalculator
from molfeat.trans import MoleculeTransformer
import datamol as dm

# 加载分子
smiles = dm.data.freesolv().sample(100).smiles.values

# 创建转换器
calc = FPCalculator("ecfp")
transformer = MoleculeTransformer(calc, n_jobs=-1)

# 批量特征化
features = transformer(smiles)  # 返回 numpy 数组 (100, 2048)

# 保存配置
transformer.to_state_yaml_file("ecfp_config.yml")

# 重新加载
transformer = MoleculeTransformer.from_state_yaml_file("ecfp_config.yml")
```

**性能：** 在 642 个分子上的测试显示，使用 4 个并行任务比单线程处理快 3.4 倍。

### FeatConcat

将多个特征化器拼接为统一表示。

```python
from molfeat.trans import FeatConcat
from molfeat.calc import FPCalculator

# 组合多个指纹
concat = FeatConcat([
    FPCalculator("maccs"),      # 167 维
    FPCalculator("ecfp")         # 2048 维
])

# 结果：2167 维特征
transformer = MoleculeTransformer(concat, n_jobs=-1)
features = transformer(smiles)
```

### PretrainedMolTransformer

用于预训练深度学习模型的 `MoleculeTransformer` 子类。

**独特功能：**
- `_embed()` - 神经网络的批量推理
- `_convert()` - 将 SMILES/分子转换为模型兼容格式
  - 语言模型的 SELFIES 字符串
  - 图神经网络的 DGL 图
- 集成缓存系统提高存储效率

**用法：**
```python
from molfeat.trans.pretrained import PretrainedMolTransformer

# 加载预训练模型
transformer = PretrainedMolTransformer("ChemBERTa-77M-MLM", n_jobs=-1)

# 生成嵌入向量
embeddings = transformer(smiles)
```

### PrecomputedMolTransformer

用于缓存/预计算特征的转换器。

---

## molfeat.store - 模型存储库

管理特征化器的发现、加载和注册。

### ModelStore

访问可用特征化器的中心枢纽。

**核心方法：**
- `available_models` - 列出所有可用特征化器的属性
- `search(name=None, **kwargs)` - 搜索特定特征化器
- `load(name, **kwargs)` - 按名称加载特征化器
- `register(name, card)` - 注册自定义特征化器

**用法：**
```python
from molfeat.store.modelstore import ModelStore

# 初始化存储库
store = ModelStore()

# 列出所有可用模型
all_models = store.available_models
print(f"找到 {len(all_models)} 个特征化器")

# 搜索特定模型
results = store.search(name="ChemBERTa-77M-MLM")
if results:
    model_card = results[0]

    # 查看使用信息
    model_card.usage()

    # 加载模型
    transformer = model_card.load()

# 直接加载
transformer = store.load("ChemBERTa-77M-MLM")
```

**ModelCard 属性：**
- `name` - 模型标识符
- `description` - 模型描述
- `version` - 模型版本
- `authors` - 模型作者
- `tags` - 分类标签
- `usage()` - 显示用法示例
- `load(**kwargs)` - 加载模型

---

## 通用模式

### 错误处理

```python
# 启用容错
featurizer = MoleculeTransformer(
    calc,
    n_jobs=-1,
    verbose=True,
    ignore_errors=True
)

# 失败分子返回 None
features = featurizer(smiles_with_errors)
```

### 数据类型控制

```python
# NumPy float32 (默认)
features = transformer(smiles, enforce_dtype=True)

# PyTorch 张量
import torch
transformer = MoleculeTransformer(calc, dtype=torch.float32)
features = transformer(smiles)
```

### 持久化与可复现性

```python
# 保存转换器状态
transformer.to_state_yaml_file("config.yml")
transformer.to_state_json_file("config.json")

# 从保存状态加载
transformer = MoleculeTransformer.from_state_yaml_file("config.yml")
transformer = MoleculeTransformer.from_state_json_file("config.json")
```

### 预处理

```python
# 手动预处理
mol = transformer.preprocess("CCO")

# 带预处理的转换
features = transformer.transform(smiles_list)
```

---

## 集成示例

### Scikit-learn 流水线

```python
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier
from molfeat.trans import MoleculeTransformer
from molfeat.calc import FPCalculator

# 创建流水线
pipeline = Pipeline([
    ('featurizer', MoleculeTransformer(FPCalculator("ecfp"))),
    ('classifier', RandomForestClassifier())
])

# 训练和预测
pipeline.fit(smiles_train, y_train)
predictions = pipeline.predict(smiles_test)
```

### PyTorch 集成

```python
import torch
from torch.utils.data import Dataset, DataLoader
from molfeat.trans import MoleculeTransformer

class MoleculeDataset(Dataset):
    def __init__(self, smiles, labels, transformer):
        self.smiles = smiles
        self.labels = labels
        self.transformer = transformer

    def __len__(self):
        return len(self.smiles)

    def __getitem__(self, idx):
        features = self.transformer(self.smiles[idx])
        return torch.tensor(features), torch.tensor(self.labels[idx])

# 创建数据集和数据加载器
transformer = MoleculeTransformer(FPCalculator("ecfp"))
dataset = MoleculeDataset(smiles, labels, transformer)
loader = DataLoader(dataset, batch_size=32)
```

---

## 性能优化建议

1. **并行化**：使用 `n_jobs=-1` 利用所有 CPU 核心
2. **批量处理**：批量处理分子而非循环处理
3. **缓存**：利用预训练模型的内置缓存
4. **数据类型**：精度允许时使用 float32 替代 float64
5. **错误处理**：对含潜在无效分子的大数据集设置 `ignore_errors=True`
