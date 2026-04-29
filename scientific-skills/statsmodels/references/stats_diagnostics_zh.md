# 统计检验与诊断参考指南

本文档全面介绍了statsmodels中可用的统计检验、诊断工具和方法。

## 概述

Statsmodels提供广泛的统计检验能力：
- 残差诊断与模型设定检验
- 假设检验（参数与非参数方法）
- 拟合优度检验
- 多重比较与事后检验
- 功效与样本量计算
- 稳健协方差矩阵
- 影响点与异常值检测

## 残差诊断

### 自相关检验

**Ljung-Box检验**：检验残差中的自相关性

```python
from statsmodels.stats.diagnostic import acorr_ljungbox

# 检验残差自相关性
lb_test = acorr_ljungbox(residuals, lags=10, return_df=True)
print(lb_test)

# H0: 滞后k阶内无自相关
# 若p值 < 0.05，则拒绝H0（存在自相关）
```

**Durbin-Watson检验**：检验一阶自相关性

```python
from statsmodels.stats.stattools import durbin_watson

dw_stat = durbin_watson(residuals)
print(f"Durbin-Watson: {dw_stat:.4f}")

# DW ≈ 2：无自相关
# DW < 2：正自相关
# DW > 2：负自相关
# 具体临界值取决于n和k
```

**Breusch-Godfrey检验**：更通用的自相关检验

```python
from statsmodels.stats.diagnostic import acorr_breusch_godfrey

bg_test = acorr_breusch_godfrey(results, nlags=5)
lm_stat, lm_pval, f_stat, f_pval = bg_test

print(f"LM统计量: {lm_stat:.4f}, p值: {lm_pval:.4f}")
# H0: 滞后k阶内无自相关
```

### 异方差性检验

**Breusch-Pagan检验**：检验异方差性

```python
from statsmodels.stats.diagnostic import het_breuschpagan

bp_test = het_breuschpagan(residuals, exog)
lm_stat, lm_pval, f_stat, f_pval = bp_test

print(f"Breusch-Pagan检验p值: {lm_pval:.4f}")
# H0: 同方差性（方差恒定）
# 若p值 < 0.05，则拒绝H0（存在异方差性）
```

**White检验**：更通用的异方差性检验

```python
from statsmodels.stats.diagnostic import het_white

white_test = het_white(residuals, exog)
lm_stat, lm_pval, f_stat, f_pval = white_test

print(f"White检验p值: {lm_pval:.4f}")
# H0: 同方差性
```

**ARCH检验**：检验自回归条件异方差性

```python
from statsmodels.stats.diagnostic import het_arch

arch_test = het_arch(residuals, nlags=5)
lm_stat, lm_pval, f_stat, f_pval = arch_test

print(f"ARCH检验p值: {lm_pval:.4f}")
# H0: 无ARCH效应
# 若显著，考虑使用GARCH模型
```

### 正态性检验

**Jarque-Bera检验**：基于偏度和峰度的正态性检验

```python
from statsmodels.stats.stattools import jarque_bera

jb_stat, jb_pval, skew, kurtosis = jarque_bera(residuals)

print(f"Jarque-Bera统计量: {jb_stat:.4f}")
print(f"p值: {jb_pval:.4f}")
print(f"偏度: {skew:.4f}")
print(f"峰度: {kurtosis:.4f}")

# H0: 残差服从正态分布
# 正态分布：偏度≈0，峰度≈3
```

**Omnibus检验**：另一种正态性检验（同样基于偏度/峰度）

```python
from statsmodels.stats.stattools import omni_normtest

omni_stat, omni_pval = omni_normtest(residuals)
print(f"Omnibus检验p值: {omni_pval:.4f}")
# H0: 正态性
```

**Anderson-Darling检验**：分布拟合检验

```python
from statsmodels.stats.diagnostic import normal_ad

ad_stat, ad_pval = normal_ad(residuals)
print(f"Anderson-Darling检验p值: {ad_pval:.4f}")
```

**Lilliefors检验**：改进的Kolmogorov-Smirnov检验

```python
from statsmodels.stats.diagnostic import lilliefors

lf_stat, lf_pval = lilliefors(residuals, dist='norm')
print(f"Lilliefors检验p值: {lf_pval:.4f}")
```

### 线性与设定检验

**Ramsey RESET检验**：检验函数形式设定错误

```python
from statsmodels.stats.diagnostic import linear_reset

reset_test = linear_reset(results, power=2)
f_stat, f_pval = reset_test

print(f"RESET检验p值: {f_pval:.4f}")
# H0: 模型设定正确（线性）
# 若拒绝，可能需要多项式项或变换
```

**Harvey-Collier检验**：检验线性关系

```python
from statsmodels.stats.diagnostic import linear_harvey_collier

hc_stat, hc_pval = linear_harvey_collier(results)
print(f"Harvey-Collier检验p值: {hc_pval:.4f}")
# H0: 线性设定正确
```

## 多重共线性检测

**方差膨胀因子(VIF)**：

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor
import pandas as pd

# 计算每个变量的VIF
vif_data = pd.DataFrame()
vif_data["变量"] = X.columns
vif_data["VIF"] = [variance_inflation_factor(X.values, i)
                   for i in range(X.shape[1])]

print(vif_data.sort_values('VIF', ascending=False))

# 解释：
# VIF = 1：与其他预测变量无相关性
# VIF > 5：中度多重共线性
# VIF > 10：严重多重共线性问题
# VIF > 20：强烈多重共线性（考虑移除变量）
```

**条件数**：来自回归结果

```python
print(f"条件数: {results.condition_number:.2f}")

# 解释：
# < 10：无多重共线性担忧
# 10-30：中度多重共线性
# > 30：强多重共线性
# > 100：严重多重共线性
```

## 影响点与异常值检测

### 杠杆值

高杠杆点具有极端的预测变量值。

```python
from statsmodels.stats.outliers_influence import OLSInfluence

influence = results.get_influence()

# 帽子值（杠杆值）
leverage = influence.hat_matrix_diag

# 经验法则：杠杆值 > 2*p/n 或 3*p/n 为高杠杆
# p = 参数数量, n = 样本量
threshold = 2 * len(results.params) / len(y)
high_leverage = np.where(leverage > threshold)[0]

print(f"高杠杆观测点: {high_leverage}")
```

### Cook距离

衡量每个观测点的整体影响力。

```python
# Cook距离
cooks_d = influence.cooks_distance[0]

# 经验法则：Cook距离 > 4/n 为强影响点
threshold = 4 / len(y)
influential = np.where(cooks_d > threshold)[0]

print(f"强影响观测点(Cook距离): {influential}")

# 绘图
import matplotlib.pyplot as plt
plt.stem(range(len(cooks_d)), cooks_d)
plt.axhline(y=threshold, color='r', linestyle='--', label=f'阈值(4/n)')
plt.xlabel('观测点')
plt.ylabel("Cook距离")
plt.legend()
plt.show()
```

### DFFITS

衡量对拟合值的影响。

```python
# DFFITS
dffits = influence.dffits[0]

# 经验法则：|DFFITS| > 2*sqrt(p/n) 为强影响点
p = len(results.params)
n = len(y)
threshold = 2 * np.sqrt(p / n)

influential_dffits = np.where(np.abs(dffits) > threshold)[0]
print(f"强影响观测点(DFFITS): {influential_dffits}")
```

### DFBETAs

衡量对每个系数的影响。

```python
# DFBETAs（每个参数对应一个）
dfbetas = influence.dfbetas

# 经验法则：|DFBETA| > 2/sqrt(n)
threshold = 2 / np.sqrt(n)

for i, param_name in enumerate(results.params.index):
    influential = np.where(np.abs(dfbetas[:, i]) > threshold)[0]
    if len(influential) > 0:
        print(f"对{param_name}有影响的观测点: {influential}")
```

### 影响图

```python
from statsmodels.graphics.regressionplots import influence_plot

fig, ax = plt.subplots(figsize=(12, 8))
influence_plot(results, ax=ax, criterion='cooks')
plt.show()

# 综合杠杆值、残差和Cook距离
# 大圆圈 = 高Cook距离
# 远离x=0 = 高杠杆值
# 远离y=0 = 大残差
```

### 学生化残差

```python
# 学生化残差（异常值）
student_resid = influence.resid_studentized_internal

# 外部学生化残差（更保守）
student_resid_external = influence.resid_studentized_external

# 异常值：|学生化残差| > 3（或 > 2.5）
outliers = np.where(np.abs(student_resid_external) > 3)[0]
print(f"异常值: {outliers}")
```

## 假设检验

### t检验

**单样本t检验**：检验均值是否等于特定值

```python
from scipy import stats

# H0: 总体均值 = mu_0
t_stat, p_value = stats.ttest_1samp(data, popmean=mu_0)

print(f"t统计量: {t_stat:.4f}")
print(f"p值: {p_value:.4f}")
```

**双样本t检验**：比较两组均值

```python
# H0: 均值1 = 均值2（等方差）
t_stat, p_value = stats.ttest_ind(group1, group2)

# Welch t检验（异方差）
t_stat, p_value = stats.ttest_ind(group1, group2, equal_var=False)

print(f"t统计量: {t_stat:.4f}")
print(f"p值: {p_value:.4f}")
```

**配对t检验**：比较配对观测值

```python
# H0: 均值差 = 0
t_stat, p_value = stats.ttest_rel(before, after)

print(f"t统计量: {t_stat:.4f}")
print(f"p值: {p_value:.4f}")
```

### 比例检验

**单比例检验**：

```python
from statsmodels.stats.proportion import proportions_ztest

# H0: 比例 = p0
count = 45  # 成功次数
nobs = 100  # 总观测数
p0 = 0.5    # 假设比例

z_stat, p_value = proportions_ztest(count, nobs, value=p0)

print(f"z统计量: {z_stat:.4f}")
print(f"p值: {p_value:.4f}")
```

**双比例检验**：

```python
# H0: 比例1 = 比例2
counts = [45, 60]
nobs = [100, 120]

z_stat, p_value = proportions_ztest(counts, nobs)
print(f"z统计量: {z_stat:.4f}")
print(f"p值: {p_value:.4f}")
```

### 卡方检验

**卡方独立性检验**：

```python
from scipy.stats import chi2_contingency

# 列联表
contingency_table = pd.crosstab(variable1, variable2)

chi2, p_value, dof, expected = chi2_contingency(contingency_table)

print(f"卡方统计量: {chi2:.4f}")
print(f"p值: {p_value:.4f}")
print(f"自由度: {dof}")

# H0: 变量相互独立
```

**卡方拟合优度检验**：

```python
from scipy.stats import chisquare

# 观测频数
observed = [20, 30, 25, 25]

# 期望频数（默认相等）
expected = [25, 25, 25, 25]

chi2, p_value = chisquare(observed, expected)

print(f"卡方统计量: {chi2:.4f}")
print(f"p值: {p_value:.4f}")

# H0: 数据符合期望分布
```

### 非参数检验

**Mann-Whitney U检验**（独立样本）：

```python
from scipy.stats import mannwhitneyu

# H0: 分布相同
u_stat, p_value = mannwhitneyu(group1, group2, alternative='two-sided')

print(f"U统计量: {u_stat:.4f}")
print(f"p值: {p_value:.4f}")
```

**Wilcoxon符号秩检验**（配对样本）：

```python
from scipy.stats import wilcoxon

# H0: 中位数差 = 0
w_stat, p_value = wilcoxon(before, after)

print(f"W统计量: {w_stat:.4f}")
print(f"p值: {p_value:.4f}")
```

**Kruskal-Wallis H检验**（>2组）：

```python
from scipy.stats import kruskal

# H0: 所有组分布相同
h_stat, p_value = kruskal(group1, group2, group3)

print(f"H统计量: {h_stat:.4f}")
print(f"p值: {p_value:.4f}")
```

**符号检验**：

```python
from statsmodels.stats.descriptivestats import sign_test

# H0: 中位数 = m0
result = sign_test(data, m0=0)
print(result)
```

### 方差分析(ANOVA)

**单因素方差分析**：

```python
from scipy.stats import f_oneway

# H0: 所有组均值相等
f_stat, p_value = f_oneway(group1, group2, group3)

print(f"F统计量: {f_stat:.4f}")
print(f"p值: {p_value:.4f}")
```

**双因素方差分析**（使用statsmodels）：

```python
from statsmodels.formula.api import ols
from statsmodels.stats.anova import anova_lm

# 拟合模型
model = ols('响应变量 ~ C(因子1) + C(因子2) + C(因子1):C(因子2)',
            data=df).fit()

# 方差分析表
anova_table = anova_lm(model, typ=2)
print(anova_table)
```

**重复测量方差分析**：

```python
from statsmodels.stats.anova import AnovaRM

# 需要长格式数据
aovrm = AnovaRM(df, depvar='分数', subject='受试者ID', within=['时间'])
results = aovrm.fit()

print(results.summary())
```

## 多重比较

### 事后检验

**Tukey HSD**（真实显著差异）：

```python
from statsmodels.stats.multicomp import pairwise_tukeyhsd

# 执行Tukey HSD检验
tukey = pairwise_tukeyhsd(data, groups, alpha=0.05)

print(tukey.summary())

# 绘制置信区间
tukey.plot_simultaneous()
plt.show()
```

**Bonferroni校正**：

```python
from statsmodels.stats.multitest import multipletests

# 多重检验的p值
p_values = [0.01, 0.03, 0.04, 0.15, 0.001]

# 应用校正
reject, pvals_corrected, alphac_sidak, alphac_bonf = multipletests(
    p_values,
    alpha=0.05,
    method='bonferroni'
)

print("拒绝假设:", reject)
print("校正后p值:", pvals_corrected)
```

**错误发现率(FDR)**：

```python

# FDR校正（比Bonferroni方法更不保守）
reject, pvals_corrected, alphac_sidak, alphac_bonf = multipletests(
    p_values,
    alpha=0.05,
    method='fdr_bh'  # Benjamini-Hochberg方法
)

print("拒绝假设:", reject)
print("校正后p值:", pvals_corrected)
```

## 稳健协方差矩阵

### 异方差一致性（HC）标准误

```python
# OLS拟合后
results = sm.OLS(y, X).fit()

# HC0 (White异方差一致性标准误)
results_hc0 = results.get_robustcov_results(cov_type='HC0')

# HC1 (自由度调整)
results_hc1 = results.get_robustcov_results(cov_type='HC1')

# HC2 (杠杆调整)
results_hc2 = results.get_robustcov_results(cov_type='HC2')

# HC3 (最保守，推荐小样本使用)
results_hc3 = results.get_robustcov_results(cov_type='HC3')

print("标准OLS标准误:", results.bse)
print("稳健HC3标准误:", results_hc3.bse)
```

### HAC（异方差和自相关一致性）

**Newey-West标准误**:

```python
# 适用于存在自相关和异方差的时间序列
results_hac = results.get_robustcov_results(cov_type='HAC', maxlags=4)

print("HAC (Newey-West)标准误:", results_hac.bse)
print(results_hac.summary())
```

### 聚类稳健标准误

```python
# 适用于聚类/分组数据
results_cluster = results.get_robustcov_results(
    cov_type='cluster',
    groups=cluster_ids
)

print("聚类稳健标准误:", results_cluster.bse)
```

## 描述性统计

**基本描述性统计**:

```python
from statsmodels.stats.api import DescrStatsW

# 综合描述统计
desc = DescrStatsW(data)

print("均值:", desc.mean)
print("标准差:", desc.std)
print("方差:", desc.var)
print("置信区间:", desc.tconfint_mean())

# 分位数
print("中位数:", desc.quantile(0.5))
print("四分位距:", desc.quantile([0.25, 0.75]))
```

**加权统计**:

```python
# 带权重计算
desc_weighted = DescrStatsW(data, weights=weights)

print("加权均值:", desc_weighted.mean)
print("加权标准差:", desc_weighted.std)
```

**两组比较**:

```python
from statsmodels.stats.weightstats import CompareMeans

# 创建比较对象
cm = CompareMeans(DescrStatsW(group1), DescrStatsW(group2))

# t检验
print("t检验结果:", cm.ttest_ind())

# 差异置信区间
print("差异置信区间:", cm.tconfint_diff())

# 等方差检验
print("等方差检验:", cm.test_equal_var())
```

## 功效分析与样本量

**t检验功效**:

```python
from statsmodels.stats.power import tt_ind_solve_power

# 求解样本量
effect_size = 0.5  # Cohen's d效应量
alpha = 0.05
power = 0.8

n = tt_ind_solve_power(effect_size=effect_size,
                        alpha=alpha,
                        power=power,
                        alternative='two-sided')

print(f"每组所需样本量: {n:.0f}")

# 给定样本量求解功效
power = tt_ind_solve_power(effect_size=0.5,
                           nobs1=50,
                           alpha=0.05,
                           alternative='two-sided')

print(f"功效: {power:.4f}")
```

**比例检验功效**:

```python
from statsmodels.stats.power import zt_ind_solve_power

# 比例检验(z检验)
effect_size = 0.3  # 比例差异
alpha = 0.05
power = 0.8

n = zt_ind_solve_power(effect_size=effect_size,
                        alpha=alpha,
                        power=power,
                        alternative='two-sided')

print(f"每组所需样本量: {n:.0f}")
```

**功效曲线**:

```python
from statsmodels.stats.power import TTestIndPower
import matplotlib.pyplot as plt

# 创建功效分析对象
analysis = TTestIndPower()

# 绘制不同样本量的功效曲线
sample_sizes = range(10, 200, 10)
effect_sizes = [0.2, 0.5, 0.8]  # 小、中、大效应量

fig, ax = plt.subplots(figsize=(10, 6))

for es in effect_sizes:
    power = [analysis.solve_power(effect_size=es, nobs1=n, alpha=0.05)
             for n in sample_sizes]
    ax.plot(sample_sizes, power, label=f'效应量 = {es}')

ax.axhline(y=0.8, color='r', linestyle='--', label='功效=0.8')
ax.set_xlabel('每组样本量')
ax.set_ylabel('功效')
ax.set_title('双样本t检验功效曲线')
ax.legend()
ax.grid(True, alpha=0.3)
plt.show()
```

## 效应量

**Cohen's d** (标准化均值差异):

```python
def cohens_d(group1, group2):
    \"\"\"计算独立样本的Cohen's d\"\"\"
    n1, n2 = len(group1), len(group2)
    var1, var2 = np.var(group1, ddof=1), np.var(group2, ddof=1)

    # 合并标准差
    pooled_std = np.sqrt(((n1-1)*var1 + (n2-1)*var2) / (n1+n2-2))

    # Cohen's d
    d = (np.mean(group1) - np.mean(group2)) / pooled_std

    return d

d = cohens_d(group1, group2)
print(f"Cohen's d值: {d:.4f}")

# 解释标准:
# |d| < 0.2: 可忽略
# |d| ~ 0.2: 小效应
# |d| ~ 0.5: 中等效应
# |d| ~ 0.8: 大效应
```

**Eta平方** (用于ANOVA):

```python
# 从ANOVA表中计算
# η² = 组间平方和 / 总平方和

def eta_squared(anova_table):
    return anova_table['sum_sq'][0] / anova_table['sum_sq'].sum()

# 运行ANOVA后
eta_sq = eta_squared(anova_table)
print(f"Eta平方值: {eta_sq:.4f}")

# 解释标准:
# 0.01: 小效应
# 0.06: 中等效应
# 0.14: 大效应
```

## 列联表与关联性

**McNemar检验** (配对二元数据):

```python
from statsmodels.stats.contingency_tables import mcnemar

# 2x2列联表
table = [[a, b],
         [c, d]]

result = mcnemar(table, exact=True)  # 大样本可用exact=False
print(f"p值: {result.pvalue:.4f}")

# 原假设: 边际概率相等
```

**Cochran-Mantel-Haenszel检验**:

```python
from statsmodels.stats.contingency_tables import StratifiedTable

# 分层2x2表
strat_table = StratifiedTable(tables_list)
result = strat_table.test_null_odds()

print(f"p值: {result.pvalue:.4f}")
```

## 处理效应与因果推断

**倾向得分匹配**:

```python
from statsmodels.treatment import propensity_score

# 估计倾向得分
ps_model = sm.Logit(treatment, X).fit()
propensity_scores = ps_model.predict(X)

# 用于匹配或加权
# (需手动实现匹配过程)
```

**双重差分法**:

```python
# DID公式: outcome ~ treatment * post
model = ols('outcome ~ treatment + post + treatment:post', data=df).fit()

# DID估计值为交互项系数
did_estimate = model.params['treatment:post']
print(f"DID估计值: {did_estimate:.4f}")
```

## 最佳实践

1. **始终检验假设**: 在解释结果前进行检验
2. **报告效应量**: 不仅报告p值
3. **选用适当检验**: 匹配数据类型和分布
4. **多重比较校正**: 当进行多次检验时
5. **检查样本量**: 确保足够功效
6. **可视化检查**: 检验前先绘图
7. **报告置信区间**: 与点估计值一起报告
8. **考虑替代方法**: 当假设违反时使用非参数方法
9. **稳健标准误**: 存在异方差/自相关时使用
10. **记录决策过程**: 说明使用的检验及原因

## 常见陷阱

1. **未检验假设**: 可能导致结果无效
2. **未校正的多重检验**: 增加I类错误
3. **在非正态数据上使用参数检验**: 考虑非参数方法
4. **忽略异方差性**: 应使用稳健标准误
5. **混淆统计显著性与实际意义**: 检查效应量
6. **未报告置信区间**: 仅p值不足够
7. **选用错误检验**: 需匹配研究问题
8. **功效不足**: II类错误风险(假阴性)
9. **p值操纵**: 不断尝试直到显著
10. **过度解读p值**: 记住零假设显著性检验的局限性
