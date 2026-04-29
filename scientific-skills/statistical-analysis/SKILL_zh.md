---
name: statistical-analysis
description: 提供包含检验选择与报告功能的统计分析方法指导。当您需要为数据选择合适的检验方法、进行假设检验、功效分析以及生成APA格式结果时使用。最适合学术研究报告和检验选择指导。如需通过编程实现特定模型，请使用statsmodels。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# 统计分析

## 概述

统计分析是检验假设和量化关系的系统化过程。可执行假设检验（t检验、ANOVA、卡方检验）、回归分析、相关性分析及贝叶斯分析，包含假设检验和APA格式报告。适用于学术研究场景。

## 使用场景

本技能适用于以下情况：
- 执行统计假设检验（t检验、ANOVA、卡方检验）
- 进行回归或相关性分析
- 运行贝叶斯统计分析
- 检验统计假设与诊断
- 计算效应量并进行功效分析
- 以APA格式报告统计结果
- 为研究分析实验或观测数据

---

## 核心功能

### 1. 检验选择与规划
- 根据研究问题和数据特征选择合适的统计检验方法
- 执行先验功效分析确定所需样本量
- 规划包含多重比较校正的分析策略

### 2. 假设检验
- 在运行检验前自动验证所有相关假设
- 提供诊断可视化图表（Q-Q图、残差图、箱线图）
- 当假设违反时推荐补救措施

### 3. 统计检验
- 假设检验：t检验、ANOVA、卡方检验及非参数替代方法
- 回归分析：线性/多元/逻辑回归（含诊断）
- 相关性分析：Pearson/Spearman相关系数（含置信区间）
- 贝叶斯替代方法：贝叶斯t检验/ANOVA/回归（含贝叶斯因子）

### 4. 效应量与解释
- 计算并解释所有分析的适用效应量
- 提供效应估计的置信区间
- 区分统计显著性与实际显著性

### 5. 专业报告
- 生成APA格式统计报告
- 创建可直接发表的图表
- 提供包含完整统计量的结果解读

---

## 工作流决策树

使用此决策树确定分析路径：

```
开始
│
├─ 需要选择统计检验方法？
│  └─ 是 → 参见"检验选择指南"
│  └─ 否 → 继续
│
├─ 准备检验假设？
│  └─ 是 → 参见"假设检验"
│  └─ 否 → 继续
│
├─ 准备运行分析？
│  └─ 是 → 参见"执行统计检验"
│  └─ 否 → 继续
│
└─ 需要报告结果？
   └─ 是 → 参见"结果报告"
```

---

## 检验选择指南

### 快速参考：选择合适检验

完整指南见`references/test_selection_guide.md`。快速参考：

**两组比较：**
- 独立、连续、正态分布 → 独立样本t检验
- 独立、连续、非正态分布 → Mann-Whitney U检验
- 配对、连续、正态分布 → 配对t检验
- 配对、连续、非正态分布 → Wilcoxon符号秩检验
- 二元结果 → 卡方检验或Fisher精确检验

**三组以上比较：**
- 独立、连续、正态分布 → 单因素ANOVA
- 独立、连续、非正态分布 → Kruskal-Wallis检验
- 配对、连续、正态分布 → 重复测量ANOVA
- 配对、连续、非正态分布 → Friedman检验

**变量关系：**
- 两个连续变量 → Pearson（正态）或Spearman相关（非正态）
- 含预测变量的连续结果 → 线性回归
- 含预测变量的二元结果 → 逻辑回归

**贝叶斯替代方法：**
所有检验均有贝叶斯版本，可提供：
- 对假设的直接概率陈述
- 量化证据的贝叶斯因子
- 支持零假设的能力
- 详见`references/bayesian_statistics.md`

---

## 假设检验

### 系统化假设验证

**在解读检验结果前必须验证假设。**

使用`scripts/assumption_checks.py`模块进行自动化检验：

```python
from scripts.assumption_checks import comprehensive_assumption_check

# 带可视化的综合检验
results = comprehensive_assumption_check(
    data=df,
    value_col='score',
    group_col='group',  # 可选：用于组间比较
    alpha=0.05
)
```

功能包含：
1. **异常值检测**（IQR和z分数法）
2. **正态性检验**（Shapiro-Wilk检验 + Q-Q图）
3. **方差齐性检验**（Levene检验 + 箱线图）
4. **结果解读与建议**

### 独立假设检验

针对性检验使用独立函数：

```python
from scripts.assumption_checks import (
    check_normality,
    check_normality_per_group,
    check_homogeneity_of_variance,
    check_linearity,
    detect_outliers
)

# 示例：带可视化的正态性检验
result = check_normality(
    data=df['score'],
    name='测试分数',
    alpha=0.05,
    plot=True
)
print(result['interpretation'])
print(result['recommendation'])
```

### 假设违反时的处理方案

**正态性违反：**
- 轻微违反 + 每组n>30 → 继续参数检验（稳健）
- 中度违反 → 使用非参数替代方法
- 严重违反 → 数据转换或非参数检验

**方差齐性违反：**
- t检验 → 使用Welch t检验
- ANOVA → 使用Welch ANOVA或Brown-Forsythe ANOVA
- 回归 → 使用稳健标准误或加权最小二乘法

**线性违反（回归）：**
- 添加多项式项
- 变量转换
- 使用非线性模型或GAM

完整指南见`references/assumptions_and_diagnostics.md`

---

## 执行统计检验

### Python库

主要统计分析库：
- **scipy.stats**：核心统计检验
- **statsmodels**：高级回归与诊断
- **pingouin**：带效应量的用户友好型统计检验
- **pymc**：贝叶斯统计建模
- **arviz**：贝叶斯可视化与诊断

### 分析示例

#### 带完整报告的T检验

```python
import pingouin as pg
import numpy as np

# 执行独立样本t检验
result = pg.ttest(group_a, group_b, correction='auto')

# 提取结果
t_stat = result['T'].values[0]
df = result['dof'].values[0]
p_value = result['p-val'].values[0]
cohens_d = result['cohen-d'].values[0]
ci_lower = result['CI95%'].values[0][0]
ci_upper = result['CI95%'].values[0][1]

# 报告
print(f"t({df:.0f}) = {t_stat:.2f}, p = {p_value:.3f}")
print(f"Cohen's d = {cohens_d:.2f}, 95% CI [{ci_lower:.2f}, {ci_upper:.2f}]")
```

#### 含事后检验的ANOVA

```python
import pingouin as pg

# 单因素ANOVA
aov = pg.anova(dv='score', between='group', data=df, detailed=True)
print(aov)

# 若显著则进行事后检验
if aov['p-unc'].values[0] < 0.05:
    posthoc = pg.pairwise_tukey(dv='score', between='group', data=df)
    print(posthoc)

# 效应量
eta_squared = aov['np2'].values[0]  # 偏eta方
print(f"偏η² = {eta_squared:.3f}")
```

#### 带诊断的线性回归

```python
import statsmodels.api as sm
from statsmodels.stats.outliers_influence import variance_inflation_factor

# 拟合模型
X = sm.add_constant(X_predictors)  # 添加截距项
model = sm.OLS(y, X).fit()

# 摘要
print(model.summary())

# 检查多重共线性（VIF）
vif_data = pd.DataFrame()
vif_data["变量"] = X.columns
vif_data["VIF"] = [variance_inflation_factor(X.values, i) for i in range(X.shape[1])]
print(vif_data)

# 检验假设
residuals = model.resid
fitted = model.fittedvalues

# 残差图
import matplotlib.pyplot as plt
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# 残差 vs 拟合值
axes[0, 0].scatter(fitted, residuals, alpha=0.6)
axes[0, 0].axhline(y=0, color='r', linestyle='--')
axes[0, 0].set_xlabel('拟合值')
axes[0, 0].set_ylabel('残差')
axes[0, 0].set_title('残差 vs 拟合值')

# Q-Q图
from scipy import stats
stats.probplot(residuals, dist="norm", plot=axes[0, 1])
axes[0, 1].set_title('正态Q-Q图')

# 尺度-位置图
axes[1, 0].scatter(fitted, np.sqrt(np.abs(residuals / residuals.std())), alpha=0.6)
axes[1, 0].set_xlabel('拟合值')
axes[1, 0].set_ylabel('√|标准化残差|')
axes[1, 0].set_title('尺度-位置图')

# 残差直方图
axes[1, 1].hist(residuals, bins=20, edgecolor='black', alpha=0.7)
axes[1, 1].set_xlabel('残差')
axes[1, 1].set_ylabel('频数')
axes[1, 1].set_title('残差直方图')

plt.tight_layout()
plt.show()
```

#### 贝叶斯T检验

```python
import pymc as pm
import arviz as az
import numpy as np

with pm.Model() as model:
    # 先验分布
    mu1 = pm.Normal('mu_group1', mu=0, sigma=10)
    mu2 = pm.Normal('mu_group2', mu=0, sigma=10)
    sigma = pm.HalfNormal('sigma', sigma=10)

    # 似然函数
    y1 = pm.Normal('y1', mu=mu1, sigma=sigma, observed=group_a)
    y2 = pm.Normal('y2', mu=mu2, sigma=sigma, observed=group_b)

    # 派生量
    diff = pm.Deterministic('difference', mu1 - mu2)

    # 采样
    trace = pm.sample(2000, tune=1000, return_inferencedata=True)

# 结果摘要
print(az.summary(trace, var_names=['difference']))

# 组1>组2的概率
prob_greater = np.mean(trace.posterior['difference'].values > 0)
print(f"P(μ₁ > μ₂ | 数据) = {prob_greater:.3f}")

# 后验分布图
az.plot_posterior(trace, var_names=['difference'], ref_val=0)
```

---

## 效应量

### 必须计算效应量

**效应量量化效应强度，而p值仅表明效应存在。**

完整指南见`references/effect_sizes_and_power.md`

### 快速参考：常用效应量

| 检验 | 效应量 | 小 | 中 | 大 |
|------|-------------|-------|--------|-------|
| T检验 | Cohen's d | 0.20 | 0.50 | 0.80 |
| ANOVA | η²_p | 0.01 | 0.06 | 0.14 |
| 相关 | r | 0.10 | 0.30 | 0.50 |
| 回归 | R² | 0.02 | 0.13 | 0.26 |
| 卡方 | Cramér's V | 0.07 | 0.21 | 0.35 |

**注意**：基准值仅供参考，需结合具体情境！

### 效应量计算

pingouin自动计算多数效应量：

```python
# T检验返回Cohen's d
result = pg.ttest(x, y)
d = result['cohen-d'].values[0]

# ANOVA返回偏eta方
aov = pg.anova(dv='score', between='group', data=df)
eta_p2 = aov['np2'].values[0]

# 相关：r本身即为效应量
corr = pg.corr(x, y)
r = corr['r'].values[0]
```

### 效应量置信区间

始终报告CI以展示精度：

```python
from pingouin import compute_effsize_from_t

# T检验效应量
d, ci = compute_effsize_from_t(
    t_statistic,
    nx=len(group1),
    ny=len(group2),
    eftype='cohen'
)
print(f"d = {d:.2f}, 95% CI [{ci[0]:.2f}, {ci[1]:.2f}]")
```

---

## 功效分析

### 先验功效分析（研究规划）

数据收集前确定所需样本量：

```python
from statsmodels.stats.power import (
    tt_ind_solve_power,
    FTestAnovaPower
)

# T检验：检测d=0.5需要多少样本量？
n_required = tt_ind_solve_power(
    effect_size=0.5,
    alpha=0.05,
    power=0.80,
    ratio=1.0,
    alternative='two-sided'
)
print(f"每组所需样本量: {n_required:.0f}")

# ANOVA：检测f=0.25需要多少样本量？
anova_power = FTestAnovaPower()
n_per_group = anova_power.solve_power(
    effect_size=0.25,
    ngroups=3,
    alpha=0.05,
    power=0.80
)
print(f"每组所需样本量: {n_per_group:.0f}")
```

### 敏感性分析（研究后）

确定可检测的效应量：

```python
# 每组n=50时能检测多大效应？
detectable_d = tt_ind_solve_power(
    effect_size=None,  # 求解此项
    nobs1=50,
    alpha=0.05,
    power=0.80,
    ratio=1.0,
    alternative='two-sided'
)
print(f"研究可检测d ≥ {detectable_d:.2f}")
```

**注意**：通常不推荐事后功效分析（研究后计算功效），建议改用敏感性分析。

完整指南见`references/effect_sizes_and_power.md`

---

## 结果报告

### APA格式统计报告

遵循`references/reporting_standards.md`指南

### 核心报告要素

1. **描述性统计**：所有组别/变量的均值(M)、标准差(SD)、样本量(n)
2. **检验统计量**：检验名称、统计值、自由度、精确p值
3. **效应量**：含置信区间
4. **假设检验**：执行的检验、结果、采取的措施
5. **所有计划分析**：包含不显著的结果

### 报告模板示例

#### 独立样本T检验

```
A组（n=48, M=75.2, SD=8.5）得分显著高于B组（n=52, M=68.3, SD=9.2），
t(98)=3.82, p<.001, d=0.77, 95% CI [0.36, 1.18]，双尾检验。
正态性假设（Shapiro-Wilk: A组 W=0.97, p=.18; B组 W=0.96, p=.12）
和方差齐性假设（Levene检验 F(1,98)=1.23, p=.27）均满足。
```

#### 单因素ANOVA

```
单因素ANOVA显示处理条件对测试分数存在显著主效应，
F(2,147)=8.45, p<.001, η²_p=.10。采用Tukey HSD进行事后比较表明，
条件A（M=78.2, SD=7.3）得分显著高于条件B（M=71.5, SD=8.1, p=.002, d=0.87）
和条件C（M=70.1, SD=7.9, p<.001, d=1.07）。条件B与C无显著差异（p=.52, d=0.18

F(3, 146) = 45.2, p < .001, R² = .48, 调整后 R² = .47。学习时长  
(B = 1.80, SE = 0.31, β = .35, t = 5.78, p < .001, 95% CI [1.18, 2.42])  
与先前 GPA (B = 8.52, SE = 1.95, β = .28, t = 4.37, p < .001,  
95% CI [4.66, 12.38]) 是显著的预测因子，而出勤率则不是  
(B = 0.15, SE = 0.12, β = .08, t = 1.25, p = .21, 95% CI [-0.09, 0.39])。  
多重共线性不是问题（所有 VIF < 1.5）。

#### 贝叶斯分析

```
使用弱信息先验（均值差服从 Normal(0, 1)）进行贝叶斯独立样本 t 检验。后验  
分布显示 A 组得分高于 B 组（M_diff = 6.8, 95% 可信区间 [3.2, 10.4]）。  
贝叶斯因子 BF₁₀ = 45.3 为组间差异提供了极强证据，A 组均值高于 B 组的  
后验概率达 99.8%。收敛诊断结果良好（所有 R̂ < 1.01, ESS > 1000）。
```

---

## 贝叶斯统计

### 何时使用贝叶斯方法

在以下情况考虑贝叶斯方法：  
- 需整合先验信息时  
- 需要直接获得假设的概率陈述时  
- 样本量较小或计划序贯数据收集时  
- 需量化零假设证据时  
- 模型复杂时（分层模型、缺失数据）

完整指南请参阅 `references/bayesian_statistics.md`：  
- 贝叶斯定理与解释  
- 先验设定（信息性、弱信息性、无信息性）  
- 基于贝叶斯因子的假设检验  
- 可信区间 vs 置信区间  
- 贝叶斯 t 检验、方差分析、回归与分层模型  
- 模型收敛诊断与后验预测检验

### 核心优势

1. **直观解释**："给定数据，参数有 95% 概率位于此区间内"  
2. **支持零假设**：可量化无效应证据  
3. **灵活性**：无 p 值操纵风险；支持实时数据分析  
4. **不确定性量化**：提供完整的后验分布  

---

## 资源

本技能包含完整参考资料：

### 参考文献目录

- **test_selection_guide.md**：统计检验选择决策树  
- **assumptions_and_diagnostics.md**：假设检验与处理的详细指南  
- **effect_sizes_and_power.md**：效应量计算、解释与报告；功效分析  
- **bayesian_statistics.md**：贝叶斯分析方法完整指南  
- **reporting_standards.md**：APA 格式报告规范及示例  

### 脚本目录

- **assumption_checks.py**：带可视化的自动化假设检验  
  - `comprehensive_assumption_check()`：完整工作流  
  - `check_normality()`：含 Q-Q 图的正态性检验  
  - `check_homogeneity_of_variance()`：含箱形图的 Levene 方差齐性检验  
  - `check_linearity()`：回归线性检验  
  - `detect_outliers()`：IQR 与 z 分数离群值检测  

---

## 最佳实践

1. **尽可能预注册分析**：区分验证性与探索性分析  
2. **结果解释前必做假设检验**  
3. **报告效应量及置信区间**  
4. **报告所有计划分析**（含不显著结果）  
5. **区分统计显著性与实际意义**  
6. **分析前后均需数据可视化**  
7. **回归/方差分析需诊断检验**（残差图、VIF 等）  
8. **进行敏感性分析**评估稳健性  
9. **共享数据与代码**确保可复现性  
10. **透明公开**违规、数据转换及决策过程  

---

## 常见误区规避

1. **P 值操纵**：避免反复测试直至显著  
2. **HARKing**：勿将探索性结果伪装成验证性结论  
3. **忽视假设检验**：必须检验并报告违规情况  
4. **混淆显著性与重要性**：p < .05 ≠ 实际意义  
5. **遗漏效应量报告**：效应量是解释核心  
6. **选择性报告**：需呈现所有计划分析  
7. **误解 p 值**：p 值非假设成立概率  
8. **多重比较**：必要时校正家族误差  
9. **忽略缺失数据**：需识别缺失机制（MCAR/MAR/MNAR）  
10. **过度解读不显著结果**：无证据 ≠ 证据不存在  

---

## 分析启动清单

开展统计分析前：  

- [ ] 明确研究问题与假设  
- [ ] 确定合适统计检验（参考 test_selection_guide.md）  
- [ ] 功效分析确定样本量  
- [ ] 加载并检查数据  
- [ ] 检测缺失值与离群值  
- [ ] 用 assumption_checks.py 验证假设  
- [ ] 执行主分析  
- [ ] 计算效应量及置信区间  
- [ ] 按需进行事后检验（含校正）  
- [ ] 创建可视化图表  
- [ ] 按 reporting_standards.md 撰写结果  
- [ ] 执行敏感性分析  
- [ ] 共享数据与代码  

---

## 支持与延伸阅读

问题咨询指南：  
- **检验选择**：见 references/test_selection_guide.md  
- **假设检验**：见 references/assumptions_and_diagnostics.md  
- **效应量**：见 references/effect_sizes_and_power.md  
- **贝叶斯方法**：见 references/bayesian_statistics.md  
- **报告规范**：见 references/reporting_standards.md  

**核心教材**：  
- Cohen, J. (1988). *行为科学统计功效分析*  
- Field, A. (2013). *基于 IBM SPSS 的统计学发现*  
- Gelman, A., & Hill, J. (2006). *回归与多级/分层模型数据分析*  
- Kruschke, J. K. (2014). *贝叶斯数据分析实践*  

**在线资源**：  
- APA 格式指南：https://apastyle.apa.org/  
- 统计咨询：Cross Validated (stats.stackexchange.com)
