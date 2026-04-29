---
name: medchem
description: 药物化学过滤器。应用类药性规则（Lipinski、Veber）、PAINS过滤器、结构警示、复杂度指标，用于化合物优先级排序和库筛选。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# Medchem

## 概述

Medchem 是一个用于药物发现工作流程中分子过滤和优先级排序的 Python 库。应用数百种成熟及新颖的分子过滤器、结构警示和药物化学规则，可高效大规模筛选和优先排序化合物库。规则和过滤器具有场景特异性——需结合领域专业知识作为指导原则使用。

## 使用场景

本技能适用于：
- 对化合物库应用类药性规则（Lipinski、Veber 等）
- 通过结构警示或 PAINS 模式过滤分子
- 为先导化合物优化进行优先级排序
- 评估化合物质量和药物化学性质
- 检测反应性或问题官能团
- 计算分子复杂度指标

## 安装

```bash
uv pip install medchem
```

## 核心功能

### 1. 药物化学规则

通过 `medchem.rules` 模块应用成熟的类药性规则。

**可用规则：**
- 五规则（Lipinski）
- Oprea 规则
- CNS 规则
- 先导化合物规则（宽松版与严格版）
- 三规则
- Reos 规则
- 药物规则
- Veber 规则
- 黄金三角规则
- PAINS 过滤器

**单规则应用：**

```python
import medchem as mc

# 对 SMILES 字符串应用五规则
smiles = "CC(=O)OC1=CC=CC=C1C(=O)O"  # 阿司匹林
passes = mc.rules.basic_rules.rule_of_five(smiles)
# 返回：True

# 检查特定规则
passes_oprea = mc.rules.basic_rules.rule_of_oprea(smiles)
passes_cns = mc.rules.basic_rules.rule_of_cns(smiles)
```

**多规则 RuleFilters 应用：**

```python
import datamol as dm
import medchem as mc

# 加载分子
mols = [dm.to_mol(smiles) for smiles in smiles_list]

# 创建包含多个规则的过滤器
rfilter = mc.rules.RuleFilters(
    rule_list=[
        "rule_of_five",
        "rule_of_oprea",
        "rule_of_cns",
        "rule_of_leadlike_soft"
    ]
)

# 并行化应用过滤器
results = rfilter(
    mols=mols,
    n_jobs=-1,  # 使用所有 CPU 核心
    progress=True
)
```

**结果格式：**
返回结果为字典形式，包含每条规则的通过/失败状态及详细信息。

### 2. 结构警示过滤器

通过 `medchem.structural` 模块检测潜在问题结构模式。

**可用过滤器：**

1. **通用警示** - 源自 ChEMBL 数据整理和文献的结构警示
2. **NIBR 过滤器** - 诺华生物医学研究所过滤器集
3. **Lilly 扣分系统** - 礼来公司扣分系统（275条规则，扣分>100的分子被拒绝）

**通用警示：**

```python
import medchem as mc

# 创建过滤器
alert_filter = mc.structural.CommonAlertsFilters()

# 检查单个分子
mol = dm.to_mol("c1ccccc1")
has_alerts, details = alert_filter.check_mol(mol)

# 并行化批量过滤
results = alert_filter(
    mols=mol_list,
    n_jobs=-1,
    progress=True
)
```

**NIBR 过滤器：**

```python
import medchem as mc

# 应用 NIBR 过滤器
nibr_filter = mc.structural.NIBRFilters()
results = nibr_filter(mols=mol_list, n_jobs=-1)
```

**Lilly 扣分系统：**

```python
import medchem as mc

# 计算 Lilly 扣分
lilly = mc.structural.LillyDemeritsFilters()
results = lilly(mols=mol_list, n_jobs=-1)

# 每个结果包含扣分值和是否通过（≤100分）
```

### 3. 高阶操作函数式接口

`medchem.functional` 模块为常见工作流提供便捷函数。

**快速过滤：**

```python
import medchem as mc

# 对列表应用 NIBR 过滤器
filter_ok = mc.functional.nibr_filter(
    mols=mol_list,
    n_jobs=-1
)

# 应用通用警示
alert_results = mc.functional.common_alerts_filter(
    mols=mol_list,
    n_jobs=-1
)
```

### 4. 化学基团检测

通过 `medchem.groups` 识别特定化学基团和官能团。

**可用基团：**
- 铰链结合物
- 磷酸盐结合物
- 迈克尔受体
- 反应性基团
- 自定义 SMARTS 模式

**用法：**

```python
import medchem as mc

# 创建基团检测器
group = mc.groups.ChemicalGroup(groups=["hinge_binders"])

# 检查匹配
has_matches = group.has_match(mol_list)

# 获取详细匹配信息
matches = group.get_matches(mol)
```

### 5. 命名目录

通过 `medchem.catalogs` 访问精选化学结构集合。

**可用目录：**
- 官能团
- 保护基团
- 常用试剂
- 标准片段

**用法：**

```python
import medchem as mc

# 访问命名目录
catalogs = mc.catalogs.NamedCatalogs

# 使用目录进行匹配
catalog = catalogs.get("functional_groups")
matches = catalog.get_matches(mol)
```

### 6. 分子复杂度

通过 `medchem.complexity` 计算近似合成难易度的复杂度指标。

**常用指标：**
- Bertz 复杂度
- Whitlock 复杂度
- Barone 复杂度

**用法：**

```python
import medchem as mc

# 计算复杂度
complexity_score = mc.complexity.calculate_complexity(mol)

# 按复杂度阈值过滤
complex_filter = mc.complexity.ComplexityFilter(max_complexity=500)
results = complex_filter(mols=mol_list)
```

### 7. 约束条件过滤

通过 `medchem.constraints` 应用自定义属性约束。

**约束示例：**
- 分子量范围
- LogP 边界
- TPSA 上限
- 可旋转键数量

**用法：**

```python
import medchem as mc

# 定义约束
constraints = mc.constraints.Constraints(
    mw_range=(200, 500),
    logp_range=(-2, 5),
    tpsa_max=140,
    rotatable_bonds_max=10
)

# 应用约束
results = constraints(mols=mol_list, n_jobs=-1)
```

### 8. 药物化学查询语言

使用专用查询语言实现复杂过滤条件。

**查询示例：**
```
# 通过五规则且无通用警示的分子
"rule_of_five AND NOT common_alerts"

# 低复杂度的 CNS 类分子
"rule_of_cns AND complexity < 400"

# 无 Lilly 扣分的先导化合物类分子
"rule_of_leadlike AND lilly_demerits == 0"
```

**用法：**

```python
import medchem as mc

# 解析并应用查询
query = mc.query.parse("rule_of_five AND NOT common_alerts")
results = query.apply(mols=mol_list, n_jobs=-1)
```

## 工作流模式

### 模式 1：化合物库初筛

过滤大型化合物集合以识别类药候选物。

```python
import datamol as dm
import medchem as mc
import pandas as pd

# 加载化合物库
df = pd.read_csv("compounds.csv")
mols = [dm.to_mol(smi) for smi in df["smiles"]]

# 应用主过滤器
rule_filter = mc.rules.RuleFilters(rule_list=["rule_of_five", "rule_of_veber"])
rule_results = rule_filter(mols=mols, n_jobs=-1, progress=True)

# 应用结构警示
alert_filter = mc.structural.CommonAlertsFilters()
alert_results = alert_filter(mols=mols, n_jobs=-1, progress=True)

# 合并结果
df["passes_rules"] = rule_results["pass"]
df["has_alerts"] = alert_results["has_alerts"]
df["drug_like"] = df["passes_rules"] & ~df["has_alerts"]

# 保存过滤后化合物
filtered_df = df[df["drug_like"]]
filtered_df.to_csv("filtered_compounds.csv", index=False)
```

### 模式 2：先导化合物优化过滤

在先导化合物优化阶段应用更严格标准。

```python
import medchem as mc

# 创建综合过滤器
filters = {
    "rules": mc.rules.RuleFilters(rule_list=["rule_of_leadlike_strict"]),
    "alerts": mc.structural.NIBRFilters(),
    "lilly": mc.structural.LillyDemeritsFilters(),
    "complexity": mc.complexity.ComplexityFilter(max_complexity=400)
}

# 应用所有过滤器
results = {}
for name, filt in filters.items():
    results[name] = filt(mols=candidate_mols, n_jobs=-1)

# 识别通过所有过滤器的化合物
passes_all = all(r["pass"] for r in results.values())
```

### 模式 3：识别特定化学基团

查找含特定官能团或骨架的分子。

```python
import medchem as mc

# 创建多基团检测器
group_detector = mc.groups.ChemicalGroup(
    groups=["hinge_binders", "phosphate_binders"]
)

# 筛选化合物库
matches = group_detector.get_all_matches(mol_list)

# 过滤含目标基团的分子
mol_with_groups = [mol for mol, match in zip(mol_list, matches) if match]
```

## 最佳实践

1. **场景适配**：勿盲目应用过滤器。需理解生物靶点和化学空间。

2. **组合多过滤器**：结合规则、结构警示和领域知识做出更优决策。

3. **并行化处理**：处理大型数据集（>1000分子）时，始终使用 `n_jobs=-1` 进行并行处理。

4. **迭代优化**：从宽泛过滤器（五规则）开始，再按需应用更严格标准（CNS规则、先导化合物规则）。

5. **记录过滤决策**：追踪被过滤分子及其原因以保证可复现性。

6. **验证结果**：注意上市药物常违反标准过滤器——这些规则应视为指导而非绝对标准。

7. **考虑前药设计**：设计为前药的分子可能有意违反标准药物化学规则。

## 资源

### references/api_guide.md
涵盖所有 medchem 模块的完整 API 参考，包含详细函数签名、参数和返回类型。

### references/rules_catalog.md
可用规则、过滤器和警示的完整目录，含描述、阈值和文献引用。

### scripts/filter_molecules.py
生产级批处理工作流脚本。支持多输入格式（CSV、SDF、SMILES）、可配置过滤器组合和详细报告。

**用法：**
```bash
python scripts/filter_molecules.py input.csv --rules rule_of_five,rule_of_cns --alerts nibr --output filtered.csv
```

## 文档

官方文档：https://medchem-docs.datamol.io/
GitHub 仓库：https://github.com/datamol-io/medchem
