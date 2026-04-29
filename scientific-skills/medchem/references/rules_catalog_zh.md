# 药物化学规则与过滤器目录

收录所有可用的药物化学规则、结构警示和过滤器的综合目录。

## 目录

1. [类药性规则](#drug-likeness-rules)
2. [先导化合物规则](#lead-likeness-rules)
3. [片段规则](#fragment-rules)
4. [中枢神经系统规则](#cns-rules)
5. [结构警示过滤器](#structural-alert-filters)
6. [化学基团模式](#chemical-group-patterns)

---

## 类药性规则

### 五规则（Lipinski）

**参考文献：** Lipinski et al., Adv Drug Deliv Rev (1997) 23:3-25

**目的：** 预测口服生物利用度

**标准：**
- 分子量 ≤ 500 Da
- LogP ≤ 5
- 氢键供体 ≤ 5
- 氢键受体 ≤ 10

**用法：**
```python
mc.rules.basic_rules.rule_of_five(mol)
```

**说明：**
- 药物发现中最广泛使用的过滤器之一
- 约90%的口服活性药物符合这些规则
- 存在例外情况，特别是天然产物和抗生素

---

### Veber规则

**参考文献：** Veber et al., J Med Chem (2002) 45:2615-2623

**目的：** 口服生物利用度的补充标准

**标准：**
- 可旋转键 ≤ 10
- 拓扑极性表面积 (TPSA) ≤ 140 Ų

**用法：**
```python
mc.rules.basic_rules.rule_of_veber(mol)
```

**说明：**
- 对五规则的补充
- TPSA与细胞渗透性相关
- 可旋转键影响分子柔性

---

### 药物规则

**目的：** 综合类药性评估

**标准：**
- 通过五规则
- 通过Veber规则
- 不含PAINS子结构

**用法：**
```python
mc.rules.basic_rules.rule_of_drug(mol)
```

---

### REOS（快速排除无效化合物）

**参考文献：** Walters & Murcko, Adv Drug Deliv Rev (2002) 54:255-271

**目的：** 过滤不可能成为药物的化合物

**标准：**
- 分子量：200-500 Da
- LogP：-5至5
- 氢键供体：0-5
- 氢键受体：0-10

**用法：**
```python
mc.rules.basic_rules.rule_of_reos(mol)
```

---

### 黄金三角

**参考文献：** Johnson et al., J Med Chem (2009) 52:5487-5500

**目的：** 平衡亲脂性与分子量

**标准：**
- 200 ≤ MW ≤ 50 × LogP + 400
- LogP：-2至5

**用法：**
```python
mc.rules.basic_rules.golden_triangle(mol)
```

**说明：**
- 定义最佳理化空间
- 在MW与LogP图中呈现三角形

---

## 先导化合物规则

### Oprea规则

**参考文献：** Oprea et al., J Chem Inf Comput Sci (2001) 41:1308-1315

**目的：** 识别用于优化的先导化合物

**标准：**
- 分子量：200-350 Da
- LogP：-2至4
- 可旋转键 ≤ 7
- 环数量 ≤ 4

**用法：**
```python
mc.rules.basic_rules.rule_of_oprea(mol)
```

**原理：** 先导化合物应在优化过程中留有"扩展空间"

---

### 先导化合物规则（宽松版）

**目的：** 宽松的先导化合物标准

**标准：**
- 分子量：250-450 Da
- LogP：-3至4
- 可旋转键 ≤ 10

**用法：**
```python
mc.rules.basic_rules.rule_of_leadlike_soft(mol)
```

---

### 先导化合物规则（严格版）

**目的：** 严格的先导化合物标准

**标准：**
- 分子量：200-350 Da
- LogP：-2至3.5
- 可旋转键 ≤ 7
- 环数量：1-3

**用法：**
```python
mc.rules.basic_rules.rule_of_leadlike_strict(mol)
```

---

## 片段规则

### 三规则

**参考文献：** Congreve et al., Drug Discov Today (2003) 8:876-877

**目的：** 筛选基于片段的药物发现库

**标准：**
- 分子量 ≤ 300 Da
- LogP ≤ 3
- 氢键供体 ≤ 3
- 氢键受体 ≤ 3
- 可旋转键 ≤ 3
- 极性表面积 ≤ 60 Ų

**用法：**
```python
mc.rules.basic_rules.rule_of_three(mol)
```

**说明：**
- 片段在优化过程中扩展为先导化合物
- 低复杂性提供更多起点

---

## 中枢神经系统规则

### CNS规则

**目的：** 中枢神经系统药物类药性

**标准：**
- 分子量 ≤ 450 Da
- LogP：-1至5
- 氢键供体 ≤ 2
- TPSA ≤ 90 Ų

**用法：**
```python
mc.rules.basic_rules.rule_of_cns(mol)
```

**原理：**
- 血脑屏障穿透需要特定性质
- 低TPSA和HBD计数提升BBB渗透性
- 严格约束反映CNS药物开发挑战

---

## 结构警示过滤器

### PAINS（泛筛选干扰化合物）

**参考文献：** Baell & Holloway, J Med Chem (2010) 53:2719-2740

**目的：** 识别干扰实验的化合物

**类别：**
- 儿茶酚类
- 醌类
- 绕丹宁类
- 羟基苯腙类
- 烷基/芳基醛类
- 迈克尔受体（特定模式）

**用法：**
```python
mc.rules.basic_rules.pains_filter(mol)
# 未发现PAINS时返回True
```

**说明：**
- PAINS化合物通过非特异性机制在多个实验中显示活性
- 筛选活动中常见的假阳性
- 在先导选择中应降级优先级

---

### 常见警示过滤器

**来源：** 基于ChEMBL库和药物化学文献

**目的：** 标记常见问题结构模式

**警示类别：**
1. **反应性基团**
   - 环氧化物
   - 氮丙啶类
   - 酰卤
   - 异氰酸酯

2. **代谢不稳定性**
   - 肼类
   - 硫脲类
   - 苯胺类（特定模式）

3. **聚集诱导物**
   - 多芳香体系
   - 长脂肪链

4. **毒性基团**
   - 硝基芳香物
   - 芳香N-氧化物
   - 特定杂环

**用法：**
```python
alert_filter = mc.structural.CommonAlertsFilters()
has_alerts, details = alert_filter.check_mol(mol)
```

**返回格式：**
```python
{
    "has_alerts": True,
    "alert_details": ["reactive_epoxide", "metabolic_hydrazine"],
    "num_alerts": 2
}
```

---

### NIBR过滤器

**来源：** 诺华生物医学研究所

**目的：** 工业药物化学过滤规则

**特点：**
- 基于诺华经验开发的专有过滤器集
- 平衡类药性与实际药物化学需求
- 包含结构警示和性质过滤器

**用法：**
```python
nibr_filter = mc.structural.NIBRFilters()
results = nibr_filter(mols=mol_list, n_jobs=-1)
```

**返回格式：** 布尔值列表（True=通过）

---

### Lilly缺陷计分过滤器

**参考文献：** 基于礼来公司药物化学规则

**来源：** 18年积累的275个结构模式

**目的：** 识别实验干扰和问题官能团

**机制：**
- 每个匹配模式增加缺陷分
- >100缺陷分的分子被拒绝
- 部分模式增加10-50分，部分增加100+分（直接拒绝）

**缺陷类别：**

1. **高缺陷分(>50)：**
   - 已知毒性基团
   - 高反应性官能团
   - 强金属螯合剂

2. **中缺陷分(20-50)：**
   - 代谢不稳定结构
   - 易聚集结构
   - 高频干扰物

3. **低缺陷分(5-20)：**
   - 次要问题
   - 上下文相关性问题

**用法：**
```python
lilly_filter = mc.structural.LillyDemeritsFilters()
results = lilly_filter(mols=mol_list, n_jobs=-1)
```

**返回格式：**
```python
{
    "demerits": 35,
    "passes": True,  # (缺陷分 ≤ 100)
    "matched_patterns": [
        {"pattern": "phenolic_ester", "demerits": 20},
        {"pattern": "aniline_derivative", "demerits": 15}
    ]
}
```

---

## 化学基团模式

### 铰链结合基团

**目的：** 识别激酶铰链结合基序

**常见模式：**
- 氨基吡啶类
- 氨基嘧啶类
- 吲唑类
- 苯并咪唑类

**用法：**
```python
group = mc.groups.ChemicalGroup(groups=["hinge_binders"])
has_hinge = group.has_match(mol_list)
```

**应用：** 激酶抑制剂设计

---

### 磷酸结合基团

**目的：** 识别磷酸结合基团

**常见模式：**
- 特定几何构型的碱性胺
- 胍基基团
- 精氨酸模拟物

**用法：**
```python
group = mc.groups.ChemicalGroup(groups=["phosphate_binders"])
```

**应用：** 激酶抑制剂、磷酸酶抑制剂

---

### 迈克尔受体

**目的：** 识别亲电性迈克尔受体基团

**常见模式：**
- α,β-不饱和羰基
- α,β-不饱和腈
- 乙烯砜类
- 丙烯酰胺类

**用法：**
```python
group = mc.groups.ChemicalGroup(groups=["michael_acceptors"])
```

**说明：**
- 可用于共价抑制剂设计
- 筛选中常被标记为反应性警示

---

### 反应性基团

**目的：** 识别常见反应性官能团

**常见模式：**
- 环氧化物
- 氮丙啶类
- 酰卤
- 异氰酸酯
- 磺酰氯

**用法：**
```python
group = mc.groups.ChemicalGroup(groups=["reactive_groups"])
```

---

## 自定义SMARTS模式

使用SMARTS定义自定义结构模式：

```python
custom_patterns = {
    "my_warhead": "[C;H0](=O)C(F)(F)F",  # 三氟甲基酮
    "my_scaffold": "c1ccc2c(c1)ncc(n2)N",  # 氨基苯并咪唑
}

group = mc.groups.ChemicalGroup(
    groups=["hinge_binders"],
    custom_smarts=custom_patterns
)
```

---

## 过滤器选择指南

### 初始筛选（高通量）

推荐过滤器：
- 五规则
- PAINS过滤器
- 常见警示（宽松设置）

```python
rfilter = mc.rules.RuleFilters(rule_list=["rule_of_five", "pains_filter"])
alert_filter = mc.structural.CommonAlertsFilters()
```

---

### 苗头化合物到先导化合物

推荐过滤器：
- Oprea规则或先导化合物规则（宽松版）
- NIBR过滤器
- Lilly缺陷计分

```python
rfilter = mc.rules.RuleFilters(rule_list=["rule_of_oprea"])
nibr_filter = mc.structural.NIBRFilters()
lilly_filter = mc.structural.LillyDemeritsFilters()
```

---

### 先导化合物优化

推荐过滤器：
- 药物规则
- 先导化合物规则（严格版）
- 完整结构警示分析
- 复杂性过滤器

```python
rfilter = mc.rules.RuleFilters(rule_list=["rule_of_drug", "rule_of_leadlike_strict"])
alert_filter = mc.structural.CommonAlertsFilters()
complexity_filter = mc.complexity.ComplexityFilter(max_complexity=400)
```

---

### 中枢神经系统靶点

推荐过滤器：
- CNS规则
- 降低PAINS标准（CNS专用）
- BBB渗透性约束

```python
rfilter = mc.rules.RuleFilters(rule_list=["rule_of_cns"])
constraints = mc.constraints.Constraints(
    tpsa_max=90,
    hbd_max=2,
    mw_range=(300, 450)
)
```

---

### 基于片段的药物发现

推荐过滤器：
- 三规则
- 最低复杂性
- 基础反应性基团检查

```python
rfilter = mc.rules.RuleFilters(rule_list=["rule_of_three"])
complexity_filter = mc.complexity.ComplexityFilter(max_complexity=250)
```

---

## 重要注意事项

### 假阳性与假阴性

**过滤器是指导原则，非绝对标准：**

1. **假阳性（优质药物被标记）：**
   - 约10%上市药物未通过五规则
   - 天然产物常违反标准规则
   - 前药设计故意突破规则
   - 抗生素和抗病毒药常不符合

2. **假阴性（劣质化合物通过）：**
   - 通过过滤器不保证成功
   - 未捕获靶点特异性问题
   - 体内性质无法完全预测

### 上下文特异性应用

**不同场景需不同标准：**

- **靶点类别：** 激酶 vs GPCR vs 离子通道有不同优化空间
- **作用模式：** 小分子 vs PROTAC vs 分子胶
- **给药途径：** 口服 vs 静脉注射 vs 局部给药
- **疾病领域：** CNS vs 肿瘤 vs 传染病
- **研发阶段：** 筛选 vs 苗头到先导 vs 先导优化

### 结合机器学习

现代方法将规则与机器学习结合：

```python
# 基于规则的预过滤
rule_results = mc.rules.RuleFilters(rule_list=["rule_of_five"])(mols)
filtered_mols = [mol for mol, r in zip(mols, rule_results) if r["passes"]]

# 对过滤集进行ML模型评分
ml_scores = ml_model.predict(filtered_mols)

# 综合决策
final_candidates = [
    mol for mol, score in zip(filtered_mols, ml_scores)
    if score > threshold
]
```

---

## 参考文献

1. Lipinski CA et al. Adv Drug Deliv Rev (1997) 23:3-25
2. Veber DF et al. J Med Chem (2002) 45:2615-2623
3. Oprea TI et al. J Chem Inf Comput Sci (2001) 41:1308-1315
4. Congreve M et al. Drug Discov Today (2003) 8:876-877
5. Baell JB & Holloway GA. J Med Chem (2010) 53:2719-2740
6. Johnson TW et al. J Med Chem (2009) 52:5487-5500
7. Walters WP & Murcko MA. Adv Drug Deliv Rev (2002) 54:255-271
8. Hann MM & Oprea TI. Curr Opin Chem Biol (2004) 8:255-263
9. Rishton GM. Drug Discov Today (1997) 2:382-384
