# Datamol 反应与数据模块参考

## 反应模块 (`datamol.reactions`)

该反应模块支持使用 SMARTS 反应模式进行程序化化学转化。

### 应用化学反应

#### `dm.reactions.apply_reaction(rxn, reactants, as_smiles=False, sanitize=True, single_product_group=True, rm_attach=True, product_index=0)`
对反应物分子应用化学反应。
- **参数**：
  - `rxn`：反应对象（来自 SMARTS 模式）
  - `reactants`：反应物分子元组
  - `as_smiles`：返回 SMILES 字符串（True）或分子对象（False）
  - `sanitize`：对产物分子进行净化处理
  - `single_product_group`：返回单一产物（True）或所有产物组（False）
  - `rm_attach`：移除连接点标记
  - `product_index`：从反应中返回的产物索引
- **返回**：产物分子或 SMILES
- **示例**：
  ```python
  from rdkit import Chem

  # 定义反应：醇 + 羧酸 → 酯
  rxn = Chem.rdChemReactions.ReactionFromSmarts(
      '[C:1][OH:2].[C:3](=[O:4])[OH:5]>>[C:1][O:2][C:3](=[O:4])'
  )

  # 应用于反应物
  alcohol = dm.to_mol("CCO")
  acid = dm.to_mol("CC(=O)O")
  product = dm.reactions.apply_reaction(rxn, (alcohol, acid))
  ```

### 创建反应

通常使用 RDKit 从 SMARTS 模式创建反应：
```python
from rdkit.Chem import rdChemReactions

# 反应模式：[反应物1].[反应物2]>>[产物]
rxn = rdChemReactions.ReactionFromSmarts(
    '[1*][*:1].[1*][*:2]>>[*:1][*:2]'
)
```

### 验证函数

该模块包含以下功能：
- **检查分子是否为反应物**：验证分子是否匹配反应物模式
- **验证反应**：检查反应在合成上是否合理
- **处理反应文件**：从文件或数据库加载反应

### 常见反应模式

**酰胺形成**：
```python
# 胺 + 羧酸 → 酰胺
amide_rxn = rdChemReactions.ReactionFromSmarts(
    '[N:1].[C:2](=[O:3])[OH]>>[N:1][C:2](=[O:3])'
)
```

**Suzuki 偶联**：
```python
# 芳基卤 + 硼酸 → 联芳基
suzuki_rxn = rdChemReactions.ReactionFromSmarts(
    '[c:1][Br].[c:2][B]([OH])[OH]>>[c:1][c:2]'
)
```

**官能团转化**：
```python
# 醇 → 酯
esterification = rdChemReactions.ReactionFromSmarts(
    '[C:1][OH:2].[C:3](=[O:4])[Cl]>>[C:1][O:2][C:3](=[O:4])'
)
```

### 工作流示例

```python
import datamol as dm
from rdkit.Chem import rdChemReactions

# 1. 定义反应
rxn_smarts = '[C:1](=[O:2])[OH:3]>>[C:1](=[O:2])[Cl:3]'  # 酸 → 酰氯
rxn = rdChemReactions.ReactionFromSmarts(rxn_smarts)

# 2. 应用于分子库
acids = [dm.to_mol(smi) for smi in acid_smiles_list]
acid_chlorides = []

for acid in acids:
    try:
        product = dm.reactions.apply_reaction(
            rxn,
            (acid,),  # 单反应物需转为元组
            sanitize=True
        )
        acid_chlorides.append(product)
    except Exception as e:
        print(f"反应失败: {e}")

# 3. 验证产物
valid_products = [p for p in acid_chlorides if p is not None]
```

### 关键概念

- **SMARTS**：SMiles 任意目标规范 - 用于描述反应的模式语言
- **原子映射**：如 [C:1] 的数字标识在反应中保持原子身份
- **连接点**：[1*] 表示通用连接位置
- **反应验证**：并非所有 SMARTS 反应在化学上都是合理的

---

## 数据模块 (`datamol.data`)

该数据模块提供便捷访问精选分子数据集的功能，用于测试和学习。

### 可用数据集

#### `dm.data.cdk2(as_df=True, mol_column='mol')`
RDKit CDK2 数据集 - 激酶抑制剂数据。
- **参数**：
  - `as_df`：返回 DataFrame（True）或分子列表（False）
  - `mol_column`：分子列名称
- **返回**：包含分子结构和活性数据的数据集
- **用例**：用于算法测试的小型数据集
- **示例**：
  ```python
  cdk2_df = dm.data.cdk2(as_df=True)
  print(cdk2_df.shape)
  print(cdk2_df.columns)
  ```

#### `dm.data.freesolv()`
FreeSolv 数据集 - 实验和计算的溶剂化自由能。
- **内容**：642 个分子，包含：
  - IUPAC 名称
  - SMILES 字符串
  - 实验溶剂化自由能值
  - 计算值
- **警告**："仅适用于教学和测试目的"
- **不适用于**：基准测试或生产模型训练
- **示例**：
  ```python
  freesolv_df = dm.data.freesolv()
  # 列名：iupac, smiles, expt (kcal/mol), calc (kcal/mol)
  ```

#### `dm.data.solubility(as_df=True, mol_column='mol')`
RDKit 溶解度数据集（含训练/测试划分）。
- **内容**：包含预定义划分的水溶性数据
- **列**：包含标识'train'或'test'的'split'列
- **用例**：测试具有正确训练/测试分离的机器学习工作流
- **示例**：
  ```python
  sol_df = dm.data.solubility(as_df=True)

  # 划分训练/测试集
  train_df = sol_df[sol_df['split'] == 'train']
  test_df = sol_df[sol_df['split'] == 'test']

  # 用于模型开发
  X_train = dm.to_fp(train_df[mol_column])
  y_train = train_df['solubility']
  ```

### 使用指南

**测试和教程**：
```python
# 快速获取测试数据集
df = dm.data.cdk2()
mols = df['mol'].tolist()

# 测试描述符计算
descriptors_df = dm.descriptors.batch_compute_many_descriptors(mols)

# 测试聚类
clusters = dm.cluster_mols(mols, cutoff=0.3)
```

**学习工作流**：
```python
# 完整机器学习流程示例
sol_df = dm.data.solubility()

# 预处理
train = sol_df[sol_df['split'] == 'train']
test = sol_df[sol_df['split'] == 'test']

# 特征化
X_train = dm.to_fp(train['mol'])
X_test = dm.to_fp(test['mol'])

# 模型训练（示例）
from sklearn.ensemble import RandomForestRegressor
model = RandomForestRegressor()
model.fit(X_train, train['solubility'])
predictions = model.predict(X_test)
```

### 重要说明

- **玩具数据集**：专为教学目的设计，不可用于生产环境
- **小规模**：化合物数量有限，适合快速测试
- **预处理**：数据已完成清洗和格式化
- **引用**：若用于发表，请查阅数据集文档以正确标注来源

### 最佳实践

1. **仅用于开发**：勿从玩具数据集中得出科学结论
2. **使用真实数据验证**：生产代码务必在实际项目数据上测试
3. **规范引用**：若在出版物中使用请引用原始数据源
4. **了解局限性**：明确每个数据集的范围和质量
