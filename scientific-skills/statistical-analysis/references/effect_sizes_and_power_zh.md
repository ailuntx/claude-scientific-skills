# 效应量与功效分析

本文档提供关于计算、解释和报告效应量的指导，以及进行研究规划时的功效分析。

## 为何效应量至关重要

1. **统计显著性 ≠ 实际显著性**：p值仅说明效应是否存在，而非效应大小
2. **样本量依赖性**：大样本下微小效应也会变得"显著"
3. **可解释性**：效应量提供效应幅度与实际重要性
4. **元分析**：效应量支持跨研究结果整合
5. **功效分析**：样本量确定的必备依据

**黄金准则**：务必同时报告效应量与p值。

---

## 按分析类型划分的效应量

### T检验与均值差异

#### Cohen's d（标准化均值差）

**公式**：
- 独立组：d = (M₁ - M₂) / SD_pooled
- 配对组：d = M_diff / SD_diff

**解释准则**（Cohen, 1988）：
- 小效应：|d| = 0.20
- 中效应：|d| = 0.50
- 大效应：|d| = 0.80

**情境化解读**：
- 教育领域：d=0.40代表成功干预的典型效果
- 心理学领域：d=0.40被视为有意义
- 医学领域：小效应量可能具有临床重要性

**Python计算**：
```python
import pingouin as pg
import numpy as np

# 含效应量的独立t检验
result = pg.ttest(group1, group2, correction=False)
cohens_d = result['cohen-d'].values[0]

# 手动计算
mean_diff = np.mean(group1) - np.mean(group2)
pooled_std = np.sqrt((np.var(group1, ddof=1) + np.var(group2, ddof=1)) / 2)
cohens_d = mean_diff / pooled_std

# 配对t检验
result = pg.ttest(pre, post, paired=True)
cohens_d = result['cohen-d'].values[0]
```

**d的置信区间**：
```python
from pingouin import compute_effsize_from_t

d, ci = compute_effsize_from_t(t_statistic, nx=n1, ny=n2, eftype='cohen')
```

---

#### Hedges' g（偏差校正d）

**使用原因**：小样本时(n < 20)Cohen's d存在轻微高估

**公式**：g = d × 校正因子，其中校正因子 = 1 - 3/(4df - 1)

**Python计算**：
```python
result = pg.ttest(group1, group2, correction=False)
hedges_g = result['hedges'].values[0]
```

**适用场景**：
- 小样本研究(每组n < 20)
- 进行元分析(元分析标准指标)

---

#### Glass's Δ (Delta)

**适用场景**：当对照组具有已知变异性时

**公式**：Δ = (M₁ - M₂) / SD_control

**应用案例**：
- 临床试验(使用对照组SD)
- 处理影响变异性的情况

---

### 方差分析(ANOVA)

#### Eta平方(η²)

**测量内容**：因子解释的总方差比例

**公式**：η² = SS_effect / SS_total

**解释准则**：
- 小效应：η² = 0.01（解释1%方差）
- 中效应：η² = 0.06（解释6%方差）
- 大效应：η² = 0.14（解释14%方差）

**局限**：多因子分析存在偏差(总和>1.0)

**Python计算**：
```python
import pingouin as pg

# 单因素ANOVA
aov = pg.anova(dv='value', between='group', data=df)
eta_squared = aov['SS'][0] / aov['SS'].sum()

# 直接使用pingouin
aov = pg.anova(dv='value', between='group', data=df, detailed=True)
eta_squared = aov['np2'][0]  # 注意：pingouin默认报告偏eta平方
```

---

#### 偏Eta平方(η²_p)

**测量内容**：排除其他因子后，某因子解释的方差比例

**公式**：η²_p = SS_effect / (SS_effect + SS_error)

**解释准则**：与η²相同

**适用场景**：多因素ANOVA(因子设计标准指标)

**Python计算**：
```python
aov = pg.anova(dv='value', between=['factor1', 'factor2'], data=df)
# pingouin默认报告偏eta平方
partial_eta_sq = aov['np2']
```

---

#### Omega平方(ω²)

**测量内容**：总体方差解释量的低偏估计

**使用原因**：η²高估效应量；ω²提供更优总体估计

**公式**：ω² = (SS_effect - df_effect × MS_error) / (SS_total + MS_error)

**解释准则**：与η²基准相同，但数值通常更小

**Python计算**：
```python
def omega_squared(aov_table):
    ss_effect = aov_table.loc[0, 'SS']
    ss_total = aov_table['SS'].sum()
    ms_error = aov_table.loc[aov_table.index[-1], 'MS']  # 残差MS
    df_effect = aov_table.loc[0, 'DF']

    omega_sq = (ss_effect - df_effect * ms_error) / (ss_total + ms_error)
    return omega_sq
```

---

#### Cohen's f

**测量内容**：ANOVA效应量(类比Cohen's d)

**公式**：f = √(η² / (1 - η²))

**解释准则**：
- 小效应：f = 0.10
- 中效应：f = 0.25
- 大效应：f = 0.40

**Python计算**：
```python
eta_squared = 0.06  # 来自ANOVA
cohens_f = np.sqrt(eta_squared / (1 - eta_squared))
```

**功效分析应用**：ANOVA功效计算必需指标

---

### 相关性分析

#### Pearson's r / Spearman's ρ

**解释准则**：
- 小效应：|r| = 0.10
- 中效应：|r| = 0.30
- 大效应：|r| = 0.50

**重要提示**：
- r² = 决定系数(解释方差比例)
- r=0.30表示9%共享方差(0.30²=0.09)
- 需考虑方向(正/负)和研究背景

**Python计算**：
```python
import pingouin as pg

# 含CI的Pearson相关
result = pg.corr(x, y, method='pearson')
r = result['r'].values[0]
ci = [result['CI95%'][0][0], result['CI95%'][0][1]]

# Spearman相关
result = pg.corr(x, y, method='spearman')
rho = result['r'].values[0]
```

---

### 回归分析

#### R²（决定系数）

**测量内容**：模型解释的Y变量方差比例

**解释准则**：
- 小效应：R² = 0.02
- 中效应：R² = 0.13
- 大效应：R² = 0.26

**情境差异**：
- 物理科学：期望R² > 0.90
- 社会科学：R² > 0.30即属良好
- 行为预测：R² > 0.10即具意义

**Python计算**：
```python
from sklearn.metrics import r2_score
from statsmodels.api import OLS

# 使用statsmodels
model = OLS(y, X).fit()
r_squared = model.rsquared
adjusted_r_squared = model.rsquared_adj

# 手动计算
r_squared = 1 - (SS_residual / SS_total)
```

---

#### 调整R²

**使用原因**：添加预测变量会人为增加R²；调整R²惩罚模型复杂度

**公式**：R²_adj = 1 - (1 - R²) × (n - 1) / (n - k - 1)

**适用场景**：多元回归中始终与R²同时报告

---

#### 标准化回归系数(β)

**测量内容**：预测变量单标准差变化对结果的影响(SD单位)

**解释准则**：类似Cohen's d
- 小效应：|β| = 0.10
- 中效应：|β| = 0.30
- 大效应：|β| = 0.50

**Python计算**：
```python
from scipy import stats

# 先标准化变量
X_std = (X - X.mean()) / X.std()
y_std = (y - y.mean()) / y.std()

model = OLS(y_std, X_std).fit()
beta = model.params
```

---

#### f²（回归的Cohen's f平方）

**测量内容**：单个预测变量或模型比较的效应量

**公式**：f² = (R²_AB - R²_A) / (1 - R²_AB)

其中：
- R²_AB = 含预测变量的完整模型R²
- R²_A = 不含预测变量的简化模型R²

**解释准则**：
- 小效应：f² = 0.02
- 中效应：f² = 0.15
- 大效应：f² = 0.35

**Python计算**：
```python
# 比较两个嵌套模型
model_full = OLS(y, X_full).fit()
model_reduced = OLS(y, X_reduced).fit()

r2_full = model_full.rsquared
r2_reduced = model_reduced.rsquared

f_squared = (r2_full - r2_reduced) / (1 - r2_full)
```

---

### 分类数据分析

#### Cramér's V

**测量内容**：χ²检验的关联强度(适用于任意维度列联表)

**公式**：V = √(χ² / (n × (k - 1)))

其中k = min(行数, 列数)

**解释准则**（k > 2时）：
- 小效应：V = 0.07
- 中效应：V = 0.21
- 大效应：V = 0.35

**2×2列联表**：使用phi系数(φ)

**Python计算**：
```python
from scipy.stats.contingency import association

# Cramér's V
cramers_v = association(contingency_table, method='cramer')

# Phi系数(2×2表)
phi = association(contingency_table, method='pearson')
```

---

#### 比值比(OR)与风险比(RR)

**2×2列联表**：

|           | 阳性结果 | 阴性结果 |
|-----------|-----------|-----------|
| 暴露组   | a         | b         |
| 非暴露组 | c         | d         |

**比值比**：OR = (a/b) / (c/d) = ad / bc

**解释**：
- OR = 1：无关联
- OR > 1：正关联（风险增加）
- OR < 1：负关联（风险降低）
- OR = 2：风险翻倍
- OR = 0.5：风险减半

**风险比**：RR = (a/(a+b)) / (c/(c+d))

**适用场景**：
- 队列研究：使用RR（更易解释）
- 病例对照研究：使用OR（无法计算RR）
- 逻辑回归：OR是自然输出

**Python计算**：
```python
import statsmodels.api as sm

# 通过列联表计算
odds_ratio = (a * d) / (b * c)

# 置信区间
table = np.array([[a, b], [c, d]])
oddsratio, pvalue = stats.fisher_exact(table)

# 通过逻辑回归
model = sm.Logit(y, X).fit()
odds_ratios = np.exp(model.params)  # 指数化系数
ci = np.exp(model.conf_int())  # 指数化置信区间
```

---

### 贝叶斯效应量

#### 贝叶斯因子(BF)

**测量内容**：备择假设与零假设的证据比

**解释**：
- BF₁₀ = 1：H₁与H₀证据相当
- BF₁₀ = 3：H₁可能性是H₀的3倍（中等证据）
- BF₁₀ = 10：H₁可能性是H₀的10倍（强证据）
- BF₁₀ = 100：H₁可能性是H₀的100倍（决定性证据）
- BF₁₀ = 0.33：H₀可能性是H₁的3倍
- BF₁₀ = 0.10：H₀可能性是H₁的10倍

**分级标准**（Jeffreys, 1961）：
- 1-3：微弱证据
- 3-10：中等证据
- 10-30：强证据
- 30-100：极强证据
- >100：决定性证据

**Python计算**：
```python
import pingouin as pg

# 贝叶斯t检验
result = pg.ttest(group1, group2, correction=False)
# 注意：pingouin不包含BF；需使用其他包

# 通过rpy2使用JASP或BayesFactor(R)
# 或通过数值积分实现
```

---

## 功效分析

### 基本概念

**统计功效**：真实效应存在时被检出的概率(1 - β)

**常规标准**：
- 功效 = 0.80（80%检出概率）
- α = 0.05（5%第一类错误率）

**四个互相关联的参数**（已知3个可求解第4个）：
1. 样本量(n)
2. 效应量(d, f等)
3. 显著性水平(α)
4. 功效(1 - β)

---

### 先验功效分析（研究规划）

**目的**：研究开始前确定所需样本量

**步骤**：
1. 指定预期效应量（来自文献/预实验/最小有意义效应）
2. 设定α水平（通常0.05）
3. 设定期望功效（通常0.80）
4. 计算所需样本量n

**Python实现**：
```python
from statsmodels.stats.power import (
    tt_ind_solve_power,
    zt_ind_solve_power,
    FTestAnovaPower,
    NormalIndPower
)

# T检验功效分析
n_required = tt_ind_solve_power(
    effect_size=0.5,  # Cohen's d
    alpha=0.05,
    power=0.80,
    ratio=1.0,  # 等量分组
    alternative='two-sided'
)

# ANOVA功效分析
anova_power = FTestAnovaPower()
n_per_group = anova_power.solve_power(
    effect_size=0.25,  # Cohen's f
    ngroups=3,
    alpha=0.05,
    power=0.80
)

# 相关性功效分析
from pingouin import power_corr
n_required = power_corr(r=0.30, power=0.80, alpha=0.05)
```

---

### 事后功效分析（研究完成后）

**⚠️ 警告**：事后功效存在争议，通常不推荐

**问题根源**：
- 观测功效是p值的直接函数
- 若p > 0.05，功效必然较低
- 无法提供超越p值的额外信息
- 可能产生误导

**可接受场景**：
- 为后续研究进行规划
- 使用多研究综合效应量（非单一研究）
- 明确以重复实验样本量为目标

**更优替代方案**：
- 报告效应量的置信区间
- 进行敏感性分析
- 报告最小可检测效应量

---

### 敏感性分析

**目的**：在给定研究参数下确定最小可检测效应量

**适用场景**：研究完成后评估研究检测能力

**Python实现**：
```python
# 当每组n=50时可检测的效应量？
detectable_effect = tt_ind_solve_power(
    effect_size=None,  # 求解此项
    nobs1=50,
    alpha=0.05,
    power=0.80,
    ratio=1.0,
    alternative='two-sided'
)```

print(f"当每组样本量 n=50 时，我们能够检测到 d ≥ {detectable_effect:.2f}")
```

---

## 效应量报告规范

### APA格式指南

**T检验示例**:
> "A组（M = 75.2, SD = 8.5）得分显著高于B组（M = 68.3, SD = 9.2），t(98) = 3.82, p < .001, d = 0.77, 95% CI [0.36, 1.18]"

**方差分析示例**:
> "处理条件对测试分数存在显著主效应，F(2, 87) = 8.45, p < .001, η²p = .16。采用Tukey HSD的事后检验表明..."

**相关分析示例**:
> "学习时间与考试成绩呈中度正相关，r(148) = .42, p < .001, 95% CI [.27, .55]"

**回归分析示例**:
> "回归模型对考试成绩的预测显著，F(3, 146) = 45.2, p < .001, R² = .48。学习时长（β = .52, p < .001）和先前GPA（β = .31, p < .001）是显著预测因子"

**贝叶斯分析示例**:
> "贝叶斯独立样本t检验为组间差异提供了强有力证据，BF₁₀ = 23.5，表明数据在H₁假设下的可能性是H₀假设下的23.5倍"

---

## 效应量常见误区

1. **勿仅依赖基准值**：情境至关重要；微小效应可能具有实际意义
2. **报告置信区间**：CI反映效应量估计的精确度
3. **区分统计显著与实际显著**：大样本可能使微小效应"显著"
4. **考虑成本效益**：低成本的干预即使效应量小也可能有价值
5. **多重结果变量**：不同结果的效应量存在差异；需全部报告
6. **避免选择性报告**：报告所有预设分析的结果
7. **发表偏倚**：已发表效应量常被高估

---

## 效应量速查表

| 分析方法 | 效应量指标 | 小效应 | 中等效应 | 大效应 |
|----------|-------------|-------|--------|-------|
| T检验 | Cohen's d | 0.20 | 0.50 | 0.80 |
| 方差分析 | η², ω² | 0.01 | 0.06 | 0.14 |
| 方差分析 | Cohen's f | 0.10 | 0.25 | 0.40 |
| 相关分析 | r, ρ | 0.10 | 0.30 | 0.50 |
| 回归分析 | R² | 0.02 | 0.13 | 0.26 |
| 回归分析 | f² | 0.02 | 0.15 | 0.35 |
| 卡方检验 | Cramér's V | 0.07 | 0.21 | 0.35 |
| 卡方检验(2×2) | φ | 0.10 | 0.30 | 0.50 |

---

## 参考文献

- Cohen, J. (1988). *行为科学统计功效分析* (第2版)
- Lakens, D. (2013). 效应量的计算与报告
- Ellis, P. D. (2010). *效应量指南精要*
