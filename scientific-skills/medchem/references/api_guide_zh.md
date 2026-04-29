# Medchem API 参考

所有 medchem 模块和函数的完整参考。

## 模块：medchem.rules

### 类：RuleFilters

基于多种药物化学规则过滤分子。

**构造函数：**
```python
RuleFilters(rule_list: List[str])
```

**参数：**
- `rule_list`: 要应用的规则名称列表。可用规则见下文。

**方法：**

```python
__call__(mols: List[Chem.Mol], n_jobs: int = 1, progress: bool = False) -> Dict
```
- `mols`: RDKit 分子对象列表
- `n_jobs`: 并行任务数（-1 表示使用所有核心）
- `progress`: 显示进度条
- **返回**: 包含每条规则结果的字典

**示例：**
```python
rfilter = mc.rules.RuleFilters(rule_list=["rule_of_five", "rule_of_cns"])
results = rfilter(mols=mol_list, n_jobs=-1, progress=True)
```

### 模块：medchem.rules.basic_rules

可应用于单个分子的独立规则函数。

#### rule_of_five()

```python
rule_of_five(mol: Union[str, Chem.Mol]) -> bool
```

口服生物利用度的 Lipinski 五规则。

**标准：**
- 分子量 ≤ 500 Da
- LogP ≤ 5
- 氢键供体 ≤ 5
- 氢键受体 ≤ 10

**参数：**
- `mol`: SMILES 字符串或 RDKit 分子对象

**返回：** 若分子通过所有标准则为 True

#### rule_of_three()

```python
rule_of_three(mol: Union[str, Chem.Mol]) -> bool
```

片段筛选库的三规则。

**标准：**
- 分子量 ≤ 300 Da
- LogP ≤ 3
- 氢键供体 ≤ 3
- 氢键受体 ≤ 3
- 可旋转键 ≤ 3
- 极性表面积 ≤ 60 Ų

#### rule_of_oprea()

```python
rule_of_oprea(mol: Union[str, Chem.Mol]) -> bool
```

Oprea 先导化合物优化标准。

**标准：**
- 分子量：200-350 Da
- LogP：-2 至 4
- 可旋转键 ≤ 7
- 环数 ≤ 4

#### rule_of_cns()

```python
rule_of_cns(mol: Union[str, Chem.Mol]) -> bool
```

中枢神经系统药物相似性规则。

**标准：**
- 分子量 ≤ 450 Da
- LogP：-1 至 5
- 氢键供体 ≤ 2
- TPSA ≤ 90 Ų

#### rule_of_leadlike_soft()

```python
rule_of_leadlike_soft(mol: Union[str, Chem.Mol]) -> bool
```

宽松先导化合物标准（更宽松）。

**标准：**
- 分子量：250-450 Da
- LogP：-3 至 4
- 可旋转键 ≤ 10

#### rule_of_leadlike_strict()

```python
rule_of_leadlike_strict(mol: Union[str, Chem.Mol]) -> bool
```

严格先导化合物标准（更严格）。

**标准：**
- 分子量：200-350 Da
- LogP：-2 至 3.5
- 可旋转键 ≤ 7
- 环数：1-3

#### rule_of_veber()

```python
rule_of_veber(mol: Union[str, Chem.Mol]) -> bool
```

Veber 口服生物利用度规则。

**标准：**
- 可旋转键 ≤ 10
- TPSA ≤ 140 Ų

#### rule_of_reos()

```python
rule_of_reos(mol: Union[str, Chem.Mol]) -> bool
```

快速排除无用化合物 (REOS) 过滤器。

**标准：**
- 分子量：200-500 Da
- LogP：-5 至 5
- 氢键供体：0-5
- 氢键受体：0-10

#### rule_of_drug()

```python
rule_of_drug(mol: Union[str, Chem.Mol]) -> bool
```

综合药物相似性标准。

**标准：**
- 通过五规则
- 通过 Veber 规则
- 无 PAINS 子结构

#### golden_triangle()

```python
golden_triangle(mol: Union[str, Chem.Mol]) -> bool
```

药物相似性平衡的黄金三角规则。

**标准：**
- 200 ≤ 分子量 ≤ 50×LogP + 400
- LogP：-2 至 5

#### pains_filter()

```python
pains_filter(mol: Union[str, Chem.Mol]) -> bool
```

泛测定干扰化合物 (PAINS) 过滤器。

**返回：** 若分子不包含 PAINS 子结构则为 True

---

## 模块：medchem.structural

### 类：CommonAlertsFilters

基于 ChEMBL 和文献的常见结构警示过滤器。

**构造函数：**
```python
CommonAlertsFilters()
```

**方法：**

```python
__call__(mols: List[Chem.Mol], n_jobs: int = 1, progress: bool = False) -> List[Dict]
```

对分子列表应用常见警示过滤器。

**返回：** 包含以下键的字典列表：
- `has_alerts`: 指示分子是否存在警示的布尔值
- `alert_details`: 匹配的警示模式列表
- `num_alerts`: 发现的警示数量

```python
check_mol(mol: Chem.Mol) -> Tuple[bool, List[str]]
```

检查单个分子的结构警示。

**返回：** (has_alerts, alert_names) 元组

### 类：NIBRFilters

诺华 NIBR 药物化学过滤器。

**构造函数：**
```python
NIBRFilters()
```

**方法：**

```python
__call__(mols: List[Chem.Mol], n_jobs: int = 1, progress: bool = False) -> List[bool]
```

对分子应用 NIBR 过滤器。

**返回：** 布尔值列表（通过过滤则为 True）

### 类：LillyDemeritsFilters

礼来基于缺陷分的结构警示系统（275 条规则）。

**构造函数：**
```python
LillyDemeritsFilters()
```

**方法：**

```python
__call__(mols: List[Chem.Mol], n_jobs: int = 1, progress: bool = False) -> List[Dict]
```

计算分子的礼来缺陷分。

**返回：** 包含以下键的字典列表：
- `demerits`: 总缺陷分
- `passes`: 布尔值（缺陷分 ≤ 100 则为 True）
- `matched_patterns`: 带分数的匹配模式列表

---

## 模块：medchem.functional

常见操作的高级函数式 API。

### nibr_filter()

```python
nibr_filter(mols: List[Chem.Mol], n_jobs: int = 1) -> List[bool]
```

使用函数式 API 应用 NIBR 过滤器。

**参数：**
- `mols`: 分子列表
- `n_jobs`: 并行级别

**返回：** 通过/失败布尔值列表

### common_alerts_filter()

```python
common_alerts_filter(mols: List[Chem.Mol], n_jobs: int = 1) -> List[Dict]
```

使用函数式 API 应用常见警示过滤器。

**返回：** 结果字典列表

### lilly_demerits_filter()

```python
lilly_demerits_filter(mols: List[Chem.Mol], n_jobs: int = 1) -> List[Dict]
```

使用函数式 API 计算礼来缺陷分。

---

## 模块：medchem.groups

### 类：ChemicalGroup

检测分子中的特定化学基团。

**构造函数：**
```python
ChemicalGroup(groups: List[str], custom_smarts: Optional[Dict[str, str]] = None)
```

**参数：**
- `groups`: 预定义基团名称列表
- `custom_smarts`: 自定义基团名称到 SMARTS 模式的映射字典

**预定义基团：**
- `"hinge_binders"`: 激酶铰链结合基序
- `"phosphate_binders"`: 磷酸盐结合基团
- `"michael_acceptors"`: 迈克尔受体亲电体
- `"reactive_groups"`: 通用反应性官能团

**方法：**

```python
has_match(mols: List[Chem.Mol]) -> List[bool]
```

检查分子是否包含指定基团。

```python
get_matches(mol: Chem.Mol) -> Dict[str, List[Tuple]]
```

获取单个分子的详细匹配信息。

**返回：** 基团名称到原子索引列表的映射字典

```python
get_all_matches(mols: List[Chem.Mol]) -> List[Dict]
```

获取所有分子的匹配信息。

**示例：**
```python
group = mc.groups.ChemicalGroup(groups=["hinge_binders", "phosphate_binders"])
matches = group.get_all_matches(mol_list)
```

---

## 模块：medchem.catalogs

### 类：NamedCatalogs

访问精选化学目录。

**可用目录：**
- `"functional_groups"`: 常见官能团
- `"protecting_groups"`: 保护基结构
- `"reagents"`: 常见试剂
- `"fragments"`: 标准片段

**用法：**
```python
catalog = mc.catalogs.NamedCatalogs.get("functional_groups")
matches = catalog.get_matches(mol)
```

---

## 模块：medchem.complexity

计算分子复杂度指标。

### calculate_complexity()

```python
calculate_complexity(mol: Chem.Mol, method: str = "bertz") -> float
```

计算分子复杂度分数。

**参数：**
- `mol`: RDKit 分子
- `method`: 复杂度度量方法（"bertz", "whitlock", "barone"）

**返回：** 复杂度分数（值越高越复杂）

### 类：ComplexityFilter

按复杂度阈值过滤分子。

**构造函数：**
```python
ComplexityFilter(max_complexity: float, method: str = "bertz")
```

**方法：**

```python
__call__(mols: List[Chem.Mol], n_jobs: int = 1) -> List[bool]
```

过滤超过复杂度阈值的分子。

---

## 模块：medchem.constraints

### 类：Constraints

应用基于属性的自定义约束。

**构造函数：**
```python
Constraints(
    mw_range: Optional[Tuple[float, float]] = None,
    logp_range: Optional[Tuple[float, float]] = None,
    tpsa_max: Optional[float] = None,
    tpsa_range: Optional[Tuple[float, float]] = None,
    hbd_max: Optional[int] = None,
    hba_max: Optional[int] = None,
    rotatable_bonds_max: Optional[int] = None,
    rings_range: Optional[Tuple[int, int]] = None,
    aromatic_rings_max: Optional[int] = None,
)
```

**参数：** 所有参数均为可选。仅指定所需约束。

**方法：**

```python
__call__(mols: List[Chem.Mol], n_jobs: int = 1) -> List[Dict]
```

对分子应用约束。

**返回：** 包含以下键的字典列表：
- `passes`: 布尔值（所有约束通过则为 True）
- `violations`: 未通过约束的名称列表

**示例：**
```python
constraints = mc.constraints.Constraints(
    mw_range=(200, 500),
    logp_range=(-2, 5),
    tpsa_max=140
)
results = constraints(mols=mol_list, n_jobs=-1)
```

---

## 模块：medchem.query

复杂过滤的查询语言。

### parse()

```python
parse(query: str) -> Query
```

将 medchem 查询字符串解析为 Query 对象。

**查询语法：**
- 运算符：`AND`, `OR`, `NOT`
- 比较符：`<`, `>`, `<=`, `>=`, `==`, `!=`
- 属性：`complexity`, `lilly_demerits`, `mw`, `logp`, `tpsa`
- 规则：`rule_of_five`, `rule_of_cns` 等
- 过滤器：`common_alerts`, `nibr_filter`, `pains_filter`

**示例查询：**
```python
"rule_of_five AND NOT common_alerts"
"rule_of_cns AND complexity < 400"
"mw > 200 AND mw < 500 AND logp < 5"
"(rule_of_five OR rule_of_oprea) AND NOT pains_filter"
```

### 类：Query

**方法：**

```python
apply(mols: List[Chem.Mol], n_jobs: int = 1) -> List[bool]
```

将解析后的查询应用于分子。

**示例：**
```python
query = mc.query.parse("rule_of_five AND NOT common_alerts")
results = query.apply(mols=mol_list, n_jobs=-1)
passing_mols = [mol for mol, passes in zip(mol_list, results) if passes]
```

---

## 模块：medchem.utils

分子处理的实用函数。

### batch_process()

```python
batch_process(
    mols: List[Chem.Mol],
    func: Callable,
    n_jobs: int = 1,
    progress: bool = False,
    batch_size: Optional[int] = None
) -> List
```

以并行批次处理分子。

**参数：**
- `mols`: 分子列表
- `func`: 应用于每个分子的函数
- `n_jobs`: 并行工作线程数
- `progress`: 显示进度条
- `batch_size`: 处理批次大小

### standardize_mol()

```python
standardize_mol(mol: Chem.Mol) -> Chem.Mol
```

标准化分子表示（净化、中和电荷等）。

---

## 常用模式

### 模式：并行处理

所有过滤器均支持并行化：

```python
# 使用所有 CPU 核心
results = filter_object(mols=mol_list, n_jobs=-1, progress=True)

# 使用指定核心数
results = filter_object(mols=mol_list, n_jobs=4, progress=True)
```

### 模式：组合多个过滤器

```python
import medchem as mc

# 应用多个过滤器
rule_filter = mc.rules.RuleFilters(rule_list=["rule_of_five"])
alert_filter = mc.structural.CommonAlertsFilters()
lilly_filter = mc.structural.LillyDemeritsFilters()

# 获取结果
rule_results = rule_filter(mols=mol_list, n_jobs=-1)
alert_results = alert_filter(mols=mol_list, n_jobs=-1)
lilly_results = lilly_filter(mols=mol_list, n_jobs=-1)

# 组合标准
passing_mols = [
    mol for i, mol in enumerate(mol_list)
    if rule_results[i]["passes"]
    and not alert_results[i]["has_alerts"]
    and lilly_results[i]["passes"]
]
```

### 模式：与 DataFrame 协同工作

```python
import pandas as pd
import datamol as dm
import medchem as mc

# 加载数据
df = pd.read_csv("molecules.csv")
df["mol"] = df["smiles"].apply(dm.to_mol)

# 应用过滤器
rfilter = mc.rules.RuleFilters(rule_list=["rule_of_five", "rule_of_cns"])
results = rfilter(mols=df["mol"].tolist(), n_jobs=-1)

# 将结果加入 DataFrame
df["passes_ro5"] = [r["rule_of_five"] for r in results]
df["passes_cns"] = [r["rule_of_cns"] for r in results]

# 过滤 DataFrame
filtered_df = df[df["passes_ro5"] & df["passes_cns"]]
```
