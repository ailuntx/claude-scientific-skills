# Datamol 片段与骨架参考指南

## 骨架模块 (`datamol.scaffold`)

骨架代表分子的核心结构，可用于识别结构家族和分析构效关系（SAR）。

### Murcko 骨架

#### `dm.to_scaffold_murcko(mol)`
提取 Bemis-Murcko 骨架（分子框架）。
- **方法**：去除侧链，保留环系和连接基
- **返回**：代表骨架的分子对象
- **应用场景**：识别化合物系列中的核心结构
- **示例**：
  ```python
  mol = dm.to_mol("c1ccc(cc1)CCN")  # 苯乙胺
  scaffold = dm.to_scaffold_murcko(mol)
  scaffold_smiles = dm.to_smiles(scaffold)
  # 返回：'c1ccccc1CC'（苯环 + 乙基连接基）
  ```

**骨架分析工作流**：
```python
# 从化合物库提取骨架
scaffolds = [dm.to_scaffold_murcko(mol) for mol in mols]
scaffold_smiles = [dm.to_smiles(s) for s in scaffolds]

# 统计骨架频率
from collections import Counter
scaffold_counts = Counter(scaffold_smiles)
most_common = scaffold_counts.most_common(10)
```

### 模糊骨架

#### `dm.scaffold.fuzzy_scaffolding(mol, ...)`
生成包含强制保留基团的模糊骨架。
- **目的**：提供比 Murcko 规则更灵活的自定义骨架定义
- **应用场景**：超越 Murcko 规则的自定义骨架需求

### 应用场景

**基于骨架的数据集划分**（用于机器学习模型验证）：
```python
# 按骨架分组化合物
scaffold_to_mols = {}
for mol, scaffold in zip(mols, scaffolds):
    smi = dm.to_smiles(scaffold)
    if smi not in scaffold_to_mols:
        scaffold_to_mols[smi] = []
    scaffold_to_mols[smi].append(mol)

# 确保训练集/测试集具有不同骨架
```

**构效关系分析**：
```python
# 按骨架分组并分析活性
for scaffold_smi, molecules in scaffold_to_mols.items():
    activities = [get_activity(mol) for mol in molecules]
    print(f"骨架: {scaffold_smi}, 平均活性: {np.mean(activities)}")
```

---

## 片段模块 (`datamol.fragment`)

分子片段化基于化学规则将分子拆解为更小单元，适用于基于片段的药物设计和子结构分析。

### BRICS 片段化

#### `dm.fragment.brics(mol, ...)`
使用 BRICS（逆合成关键化学子结构断裂）方法进行片段化。
- **方法**：基于 16 种化学意义键类型进行切割
- **考量因素**：考虑化学环境及周围子结构
- **返回**：片段 SMILES 字符串集合
- **应用场景**：逆合成分析、基于片段的设计
- **示例**：
  ```python
  mol = dm.to_mol("c1ccccc1CCN")
  fragments = dm.fragment.brics(mol)
  # 返回片段如: '[1*]CCN', '[1*]c1ccccc1' 等
  # [1*] 表示连接点
  ```

### RECAP 片段化

#### `dm.fragment.recap(mol, ...)`
使用 RECAP（逆合成组合分析流程）方法进行片段化。
- **方法**：基于 11 种预定义键类型切割
- **规则**：
  - 保留小于 5 个碳的烷基链
  - 保持环状键完整
- **返回**：片段 SMILES 字符串集合
- **应用场景**：组合库设计
- **示例**：
  ```python
  mol = dm.to_mol("CCCCCc1ccccc1")
  fragments = dm.fragment.recap(mol)
  ```

### MMPA 片段化

#### `dm.fragment.mmpa_frag(mol, ...)`
生成适用于匹配分子对分析的片段。
- **目的**：生成适合识别分子对的片段
- **应用场景**：分析微小结构变化对性质的影响
- **示例**：
  ```python
  fragments = dm.fragment.mmpa_frag(mol)
  # 用于发现仅存在单一结构差异的分子对
  ```

### 方法对比

| 方法   | 键类型数 | 保留环系 | 最佳适用场景               |
|--------|----------|----------|----------------------------|
| BRICS  | 16       | 是       | 逆合成分析、片段重组       |
| RECAP  | 11       | 是       | 组合库设计                 |
| MMPA   | 可变     | 视情况   | 构效关系分析               |

### 片段化工作流

```python
import datamol as dm

# 1. 片段化分子
mol = dm.to_mol("CC(=O)Oc1ccccc1C(=O)O")  # 阿司匹林
brics_frags = dm.fragment.brics(mol)
recap_frags = dm.fragment.recap(mol)

# 2. 统计库中片段频率
all_fragments = []
for mol in molecule_library:
    frags = dm.fragment.brics(mol)
    all_fragments.extend(frags)

# 3. 识别常见片段
from collections import Counter
fragment_counts = Counter(all_fragments)
common_fragments = fragment_counts.most_common(20)

# 4. 将片段还原为分子（移除连接点）
def clean_fragment(frag_smiles):
    # 移除 [1*], [2*] 等连接点标记
    clean = frag_smiles.replace('[1*]', '[H]')
    return dm.to_mol(clean)
```

### 进阶：基于片段的虚拟筛选

```python
# 从已知活性物构建片段库
active_fragments = set()
for active_mol in active_compounds:
    frags = dm.fragment.brics(active_mol)
    active_fragments.update(frags)

# 筛选含活性片段的化合物
def score_by_fragments(mol, fragment_set):
    mol_frags = dm.fragment.brics(mol)
    overlap = mol_frags.intersection(fragment_set)
    return len(overlap) / len(mol_frags)

# 计算筛选库得分
scores = [score_by_fragments(mol, active_fragments) for mol in screening_lib]
```

### 核心概念

- **连接点**：在片段 SMILES 中以 [1*], [2*] 等标记
- **逆合成导向**：片段化模拟合成中的断键过程
- **化学意义**：在典型合成键位点断裂
- **重组能力**：片段理论上可重组为有效分子
