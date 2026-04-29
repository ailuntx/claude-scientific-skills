# RDKit 分子描述符参考

RDKit `Descriptors` 模块中可用分子描述符的完整参考。

## 使用方法

```python
from rdkit import Chem
from rdkit.Chem import Descriptors

mol = Chem.MolFromSmiles('CCO')

# 计算单个描述符
mw = Descriptors.MolWt(mol)

# 一次性计算所有描述符
all_desc = Descriptors.CalcMolDescriptors(mol)
```

## 分子量与质量

### MolWt
分子的平均分子量。
```python
Descriptors.MolWt(mol)
```

### ExactMolWt
使用同位素组成的精确分子量。
```python
Descriptors.ExactMolWt(mol)
```

### HeavyAtomMolWt
忽略氢原子的平均分子量。
```python
Descriptors.HeavyAtomMolWt(mol)
```

## 亲脂性

### MolLogP
Wildman-Crippen LogP（辛醇-水分配系数）。
```python
Descriptors.MolLogP(mol)
```

### MolMR
Wildman-Crippen 摩尔折射率。
```python
Descriptors.MolMR(mol)
```

## 极性表面积

### TPSA
基于片段贡献的拓扑极性表面积（TPSA）。
```python
Descriptors.TPSA(mol)
```

### LabuteASA
Labute 近似表面积（ASA）。
```python
Descriptors.LabuteASA(mol)
```

## 氢键作用

### NumHDonors
氢键供体数量（N-H 和 O-H）。
```python
Descriptors.NumHDonors(mol)
```

### NumHAcceptors
氢键受体数量（N 和 O 原子）。
```python
Descriptors.NumHAcceptors(mol)
```

### NOCount
氮氧原子总数。
```python
Descriptors.NOCount(mol)
```

### NHOHCount
N-H 和 O-H 键总数。
```python
Descriptors.NHOHCount(mol)
```

## 原子计数

### HeavyAtomCount
重原子数量（非氢原子）。
```python
Descriptors.HeavyAtomCount(mol)
```

### NumHeteroatoms
杂原子数量（非碳非氢原子）。
```python
Descriptors.NumHeteroatoms(mol)
```

### NumValenceElectrons
价电子总数。
```python
Descriptors.NumValenceElectrons(mol)
```

### NumRadicalElectrons
自由基电子数量。
```python
Descriptors.NumRadicalElectrons(mol)
```

## 环系描述符

### RingCount
环总数。
```python
Descriptors.RingCount(mol)
```

### NumAromaticRings
芳香环数量。
```python
Descriptors.NumAromaticRings(mol)
```

### NumSaturatedRings
饱和环数量。
```python
Descriptors.NumSaturatedRings(mol)
```

### NumAliphaticRings
脂肪环数量（非芳香环）。
```python
Descriptors.NumAliphaticRings(mol)
```

### NumAromaticCarbocycles
芳香碳环数量（仅含碳原子的环）。
```python
Descriptors.NumAromaticCarbocycles(mol)
```

### NumAromaticHeterocycles
芳香杂环数量（含杂原子的环）。
```python
Descriptors.NumAromaticHeterocycles(mol)
```

### NumSaturatedCarbocycles
饱和碳环数量。
```python
Descriptors.NumSaturatedCarbocycles(mol)
```

### NumSaturatedHeterocycles
饱和杂环数量。
```python
Descriptors.NumSaturatedHeterocycles(mol)
```

### NumAliphaticCarbocycles
脂肪碳环数量。
```python
Descriptors.NumAliphaticCarbocycles(mol)
```

### NumAliphaticHeterocycles
脂肪杂环数量。
```python
Descriptors.NumAliphaticHeterocycles(mol)
```

## 可旋转键

### NumRotatableBonds
可旋转键数量（分子柔性指标）。
```python
Descriptors.NumRotatableBonds(mol)
```

## 芳香原子

### NumAromaticAtoms
芳香原子总数。
```python
Descriptors.NumAromaticAtoms(mol)
```

## 比例描述符

### FractionCsp3
sp3杂化碳原子占比。
```python
Descriptors.FractionCsp3(mol)
```

## 复杂度描述符

### BertzCT
Bertz 复杂度指数。
```python
Descriptors.BertzCT(mol)
```

### Ipc
信息熵（复杂度度量）。
```python
Descriptors.Ipc(mol)
```

## Kappa 形状指数

基于图不变量的分子形状描述符。

### Kappa1
第一 Kappa 形状指数。
```python
Descriptors.Kappa1(mol)
```

### Kappa2
第二 Kappa 形状指数。
```python
Descriptors.Kappa2(mol)
```

### Kappa3
第三 Kappa 形状指数。
```python
Descriptors.Kappa3(mol)
```

## Chi 连接性指数

分子连接性指数。

### Chi0, Chi1, Chi2, Chi3, Chi4
简单 Chi 连接性指数。
```python
Descriptors.Chi0(mol)
Descriptors.Chi1(mol)
Descriptors.Chi2(mol)
Descriptors.Chi3(mol)
Descriptors.Chi4(mol)
```

### Chi0n, Chi1n, Chi2n, Chi3n, Chi4n
价态修正 Chi 连接性指数。
```python
Descriptors.Chi0n(mol)
Descriptors.Chi1n(mol)
Descriptors.Chi2n(mol)
Descriptors.Chi3n(mol)
Descriptors.Chi4n(mol)
```

### Chi0v, Chi1v, Chi2v, Chi3v, Chi4v
价态 Chi 连接性指数。
```python
Descriptors.Chi0v(mol)
Descriptors.Chi1v(mol)
Descriptors.Chi2v(mol)
Descriptors.Chi3v(mol)
Descriptors.Chi4v(mol)
```

## Hall-Kier Alpha 值

### HallKierAlpha
Hall-Kier alpha 值（分子柔性指标）。
```python
Descriptors.HallKierAlpha(mol)
```

## Balaban J 指数

### BalabanJ
Balaban J 指数（分支度描述符）。
```python
Descriptors.BalabanJ(mol)
```

## EState 指数

电拓扑态指数。

### MaxEStateIndex
最大 E-state 值。
```python
Descriptors.MaxEStateIndex(mol)
```

### MinEStateIndex
最小 E-state 值。
```python
Descriptors.MinEStateIndex(mol)
```

### MaxAbsEStateIndex
最大绝对 E-state 值。
```python
Descriptors.MaxAbsEStateIndex(mol)
```

### MinAbsEStateIndex
最小绝对 E-state 值。
```python
Descriptors.MinAbsEStateIndex(mol)
```

## 部分电荷

### MaxPartialCharge
最大部分电荷值。
```python
Descriptors.MaxPartialCharge(mol)
```

### MinPartialCharge
最小部分电荷值。
```python
Descriptors.MinPartialCharge(mol)
```

### MaxAbsPartialCharge
最大绝对部分电荷值。
```python
Descriptors.MaxAbsPartialCharge(mol)
```

### MinAbsPartialCharge
最小绝对部分电荷值。
```python
Descriptors.MinAbsPartialCharge(mol)
```

## 指纹密度

分子指纹密度度量。

### FpDensityMorgan1
半径1的Morgan指纹密度。
```python
Descriptors.FpDensityMorgan1(mol)
```

### FpDensityMorgan2
半径2的Morgan指纹密度。
```python
Descriptors.FpDensityMorgan2(mol)
```

### FpDensityMorgan3
半径3的Morgan指纹密度。
```python
Descriptors.FpDensityMorgan3(mol)
```

## PEOE VSA 描述符

轨道电负性部分均衡（PEOE）VSA描述符。

### PEOE_VSA1 至 PEOE_VSA14
使用部分电荷和表面积贡献的MOE型描述符。
```python
Descriptors.PEOE_VSA1(mol)
# ... 至 PEOE_VSA14
```

## SMR VSA 描述符

分子折射率VSA描述符。

### SMR_VSA1 至 SMR_VSA10
使用MR贡献和表面积的MOE型描述符。
```python
Descriptors.SMR_VSA1(mol)
# ... 至 SMR_VSA10
```

## SLogP VSA 描述符

LogP VSA描述符。

### SLogP_VSA1 至 SLogP_VSA12
使用LogP贡献和表面积的MOE型描述符。
```python
Descriptors.SLogP_VSA1(mol)
# ... 至 SLogP_VSA12
```

## EState VSA 描述符

### EState_VSA1 至 EState_VSA11
使用E-state指数和表面积的MOE型描述符。
```python
Descriptors.EState_VSA1(mol)
# ... 至 EState_VSA11
```

## VSA 描述符

范德华表面积描述符。

### VSA_EState1 至 VSA_EState10
EState VSA描述符。
```python
Descriptors.VSA_EState1(mol)
# ... 至 VSA_EState10
```

## BCUT 描述符

Burden-CAS-德克萨斯大学特征值描述符。

### BCUT2D_MWHI
分子量加权的Burden矩阵最高特征值。
```python
Descriptors.BCUT2D_MWHI(mol)
```

### BCUT2D_MWLOW
分子量加权的Burden矩阵最低特征值。
```python
Descriptors.BCUT2D_MWLOW(mol)
```

### BCUT2D_CHGHI
部分电荷加权的最高特征值。
```python
Descriptors.BCUT2D_CHGHI(mol)
```

### BCUT2D_CHGLO
部分电荷加权的最低特征值。
```python
Descriptors.BCUT2D_CHGLO(mol)
```

### BCUT2D_LOGPHI
LogP加权的最高特征值。
```python
Descriptors.BCUT2D_LOGPHI(mol)
```

### BCUT2D_LOGPLOW
LogP加权的最低特征值。
```python
Descriptors.BCUT2D_LOGPLOW(mol)
```

### BCUT2D_MRHI
摩尔折射率加权的最高特征值。
```python
Descriptors.BCUT2D_MRHI(mol)
```

### BCUT2D_MRLOW
摩尔折射率加权的最低特征值。
```python
Descriptors.BCUT2D_MRLOW(mol)
```

## 自相关描述符

### AUTOCORR2D
二维自相关描述符（若启用）。
测量属性空间分布的各种自相关指数。

## MQN 描述符

分子量子数 - 42个简单描述符。

### mqn1 至 mqn42
统计分子特征的整数描述符。
```python
# 通过CalcMolDescriptors访问
desc = Descriptors.CalcMolDescriptors(mol)
mqns = {k: v for k, v in desc.items() if k.startswith('mqn')}
```

## QED

### qed
类药性定量评估。
```python
Descriptors.qed(mol)
```

## Lipinski 五规则

使用Lipinski标准检查类药性：

```python
def lipinski_rule_of_five(mol):
    mw = Descriptors.MolWt(mol) <= 500
    logp = Descriptors.MolLogP(mol) <= 5
    hbd = Descriptors.NumHDonors(mol) <= 5
    hba = Descriptors.NumHAcceptors(mol) <= 10
    return mw and logp and hbd and hba
```

## 批量描述符计算

一次性计算所有描述符：

```python
from rdkit import Chem
from rdkit.Chem import Descriptors

mol = Chem.MolFromSmiles('CCO')

# 获取字典形式的所有描述符
all_descriptors = Descriptors.CalcMolDescriptors(mol)

# 访问特定描述符
mw = all_descriptors['MolWt']
logp = all_descriptors['MolLogP']

# 获取可用描述符名称列表
from rdkit.Chem import Descriptors
descriptor_names = [desc[0] for desc in Descriptors._descList]
```

## 描述符类别摘要

1. **物理化学性质**：MolWt, MolLogP, MolMR, TPSA  
2. **拓扑结构**：BertzCT, BalabanJ, Kappa指数  
3. **电子性质**：部分电荷, E-state指数  
4. **形状特征**：Kappa指数, BCUT描述符  
5. **连接性**：Chi指数  
6. **二维指纹**：FpDensity描述符  
7. **原子计数**：重原子, 杂原子, 环系  
8. **类药性**：QED, Lipinski参数  
9. **柔性指标**：NumRotatableBonds, HallKierAlpha  
10. **表面积**：VSA系列描述符  

## 典型应用场景

### 类药性筛选

```python
def screen_druglikeness(mol):
    return {
        'MW': Descriptors.MolWt(mol),
        'LogP': Descriptors.MolLogP(mol),
        'HBD': Descriptors.NumHDonors(mol),
        'HBA': Descriptors.NumHAcceptors(mol),
        'TPSA': Descriptors.TPSA(mol),
        'RotBonds': Descriptors.NumRotatableBonds(mol),
        'AromaticRings': Descriptors.NumAromaticRings(mol),
        'QED': Descriptors.qed(mol)
    }
```

### 先导化合物过滤

```python
def is_leadlike(mol):
    mw = 250 <= Descriptors.MolWt(mol) <= 350
    logp = Descriptors.MolLogP(mol) <= 3.5
    rot_bonds = Descriptors.NumRotatableBonds(mol) <= 7
    return mw and logp and rot_bonds
```

### 多样性分析

```python
def molecular_complexity(mol):
    return {
        'BertzCT': Descriptors.BertzCT(mol),
        'NumRings': Descriptors.RingCount(mol),
        'NumRotBonds': Descriptors.NumRotatableBonds(mol),
        'FractionCsp3': Descriptors.FractionCsp3(mol),
        'NumAromaticRings': Descriptors.NumAromaticRings(mol)
    }
```

## 使用建议

1. **批量计算**多个描述符以避免重复计算  
2. **检查空值** - 无效分子可能返回None  
3. **标准化处理** - 机器学习应用中需归一化  
4. **选择相关描述符** - 200+描述符并非全部适用  
5. **考虑3D描述符** - 需单独处理（依赖3D坐标）  
6. **验证数值范围** - 确保描述符值在预期区间内
