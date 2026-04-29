# 线性回归模型参考指南

本文档详细介绍了statsmodels中的线性回归模型，包括OLS、GLS、WLS、分位数回归及特殊变体。

## 核心模型类

### OLS（普通最小二乘法）

假设误差项独立同分布（Σ=I）。适用于具有同方差误差的标准回归。

**适用场景：**
- 标准回归分析
- 误差项独立且方差恒定
- 无自相关或异方差现象
- 最常用的初始模型

**基础用法：**
```python
import statsmodels.api as sm
import numpy as np

# 准备数据 - 务必添加常数项作为截距
X = sm.add_constant(X_data)  # 添加全1列作为截距项

# 拟合模型
model = sm.OLS(y, X)
results = model.fit()

# 查看结果
print(results.summary())
```

**关键结果属性：**
```python
results.params           # 系数
results.bse              # 标准误
results.tvalues          # T统计量
results.pvalues          # P值
results.rsquared         # R平方
results.rsquared_adj     # 调整后R平方
results.fittedvalues     # 拟合值（训练数据预测）
results.resid            # 残差
results.conf_int()       # 参数置信区间
```

**置信/预测区间预测：**
```python
# 样本内预测
pred = results.get_prediction(X)
pred_summary = pred.summary_frame()
print(pred_summary)  # 包含均值、标准差、置信区间

# 样本外预测
X_new = sm.add_constant(X_new_data)
pred_new = results.get_prediction(X_new)
pred_summary = pred_new.summary_frame()

# 获取区间
mean_ci_lower = pred_summary["mean_ci_lower"]
mean_ci_upper = pred_summary["mean_ci_upper"]
obs_ci_lower = pred_summary["obs_ci_lower"]  # 预测区间
obs_ci_upper = pred_summary["obs_ci_upper"]
```

**公式API（R风格）：**
```python
import statsmodels.formula.api as smf

# 自动处理分类变量和交互项
formula = 'y ~ x1 + x2 + C(category) + x1:x2'
results = smf.ols(formula, data=df).fit()
```

### WLS（加权最小二乘法）

处理异方差误差（对角Σ），即不同观测值具有不同方差的情况。

**适用场景：**
- 已知异方差性（误差方差非恒定）
- 不同观测值可靠性不同
- 权重已知或可估计

**用法：**
```python
# 已知权重（方差的倒数）
weights = 1 / error_variance
model = sm.WLS(y, X, weights=weights)
results = model.fit()

# 常见权重模式：
# - 1/方差：当方差已知时
# - n_i：分组数据的样本量
# - 1/x：当方差与x成比例时
```

**可行WLS（权重估计）：**
```python
# 步骤1：拟合OLS
ols_results = sm.OLS(y, X).fit()

# 步骤2：建模残差平方以估计方差
abs_resid = np.abs(ols_results.resid)
variance_model = sm.OLS(np.log(abs_resid**2), X).fit()

# 步骤3：使用估计方差作为权重
weights = 1 / np.exp(variance_model.fittedvalues)
wls_results = sm.WLS(y, X, weights=weights).fit()
```

### GLS（广义最小二乘法）

处理任意协方差结构（Σ）。其他回归方法的超类。

**适用场景：**
- 已知协方差结构
- 误差项相关
- 比WLS更通用

**用法：**
```python
# 指定协方差结构
# Sigma应为(n x n)协方差矩阵
model = sm.GLS(y, X, sigma=Sigma)
results = model.fit()
```

### GLSAR（自回归误差的GLS）

针对时间序列数据的可行广义最小二乘法，包含AR(p)误差项。

**适用场景：**
- 存在自相关误差的时间序列回归
- 需考虑序列相关性
- 误差独立性假设被违反

**用法：**
```python
# AR(1)误差
model = sm.GLSAR(y, X, rho=1)  # rho=1表示AR(1)，rho=2表示AR(2)等
results = model.iterative_fit()  # 迭代估计AR参数

print(results.summary())
print(f"估计rho值: {results.model.rho}")
```

### RLS（递归最小二乘法）

序列参数估计，适用于自适应或在线学习。

**适用场景：**
- 参数随时间变化
- 在线/流式数据
- 需观察参数演化过程

**用法：**
```python
from statsmodels.regression.recursive_ls import RecursiveLS

model = RecursiveLS(y, X)
results = model.fit()

# 获取时变参数
params_over_time = results.recursive_coefficients
cusum = results.cusum  # 结构断点的CUSUM统计量
```

### 滚动回归

通过移动窗口计算估计值，用于检测时变参数。

**适用场景：**
- 参数随时间变化
- 需检测结构变化
- 关系随时间演变的时间序列

**用法：**
```python
from statsmodels.regression.rolling import RollingOLS, RollingWLS

# 60期窗口的滚动OLS
rolling_model = RollingOLS(y, X, window=60)
rolling_results = rolling_model.fit()

# 提取时变参数
rolling_params = rolling_results.params  # 随时间变化的参数DataFrame
rolling_rsquared = rolling_results.rsquared

# 绘制参数演化图
import matplotlib.pyplot as plt
rolling_params.plot()
plt.title('时变系数')
plt.show()
```

### 分位数回归

分析条件分位数而非条件均值。

**适用场景：**
- 关注分位数（中位数、90分位点等）
- 对异常值稳健（中位数回归）
- 分位数间的分布效应
- 异质性效应

**用法：**
```python
from statsmodels.regression.quantile_regression import QuantReg

# 中位数回归（50分位点）
model = QuantReg(y, X)
results_median = model.fit(q=0.5)

# 多分位数分析
quantiles = [0.1, 0.25, 0.5, 0.75, 0.9]
results_dict = {}
for q in quantiles:
    results_dict[q] = model.fit(q=q)

# 绘制分位数效应变化图
import matplotlib.pyplot as plt
coef_dict = {q: res.params for q, res in results_dict.items()}
coef_df = pd.DataFrame(coef_dict).T
coef_df.plot()
plt.xlabel('分位数')
plt.ylabel('系数')
plt.show()
```

## 混合效应模型

适用于具有随机效应的分层/嵌套数据。

**适用场景：**
- 聚类/分组数据（学校中的学生、医院中的患者）
- 重复测量数据
- 需考虑分组随机效应

**用法：**
```python
from statsmodels.regression.mixed_linear_model import MixedLM

# 随机截距模型
model = MixedLM(y, X, groups=group_ids)
results = model.fit()

# 随机截距和斜率模型
model = MixedLM(y, X, groups=group_ids, exog_re=X_random)
results = model.fit()

print(results.summary())
```

## 诊断与模型评估

### 残差分析

```python
# 基础残差图
import matplotlib.pyplot as plt

# 残差 vs 拟合值
plt.scatter(results.fittedvalues, results.resid)
plt.xlabel('拟合值')
plt.ylabel('残差')
plt.axhline(y=0, color='r', linestyle='--')
plt.title('残差 vs 拟合值')
plt.show()

# 正态性Q-Q图
from statsmodels.graphics.gofplots import qqplot
qqplot(results.resid, line='s')
plt.show()

# 残差直方图
plt.hist(results.resid, bins=30, edgecolor='black')
plt.xlabel('残差')
plt.ylabel('频数')
plt.title('残差分布')
plt.show()
```

### 设定检验

```python
from statsmodels.stats.diagnostic import het_breuschpagan, het_white
from statsmodels.stats.stattools import durbin_watson, jarque_bera

# 异方差检验
lm_stat, lm_pval, f_stat, f_pval = het_breuschpagan(results.resid, X)
print(f"Breusch-Pagan检验p值: {lm_pval}")

# White检验
white_test = het_white(results.resid, X)
print(f"White检验p值: {white_test[1]}")

# 自相关检验
dw_stat = durbin_watson(results.resid)
print(f"Durbin-Watson统计量: {dw_stat}")
# DW ≈ 2 表示无自相关
# DW < 2 暗示正自相关
# DW > 2 暗示负自相关

# 正态性检验
jb_stat, jb_pval, skew, kurtosis = jarque_bera(results.resid)
print(f"Jarque-Bera检验p值: {jb_pval}")
```

### 多重共线性

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor

# 计算各变量VIF
vif_data = pd.DataFrame()
vif_data["变量"] = X.columns
vif_data["VIF"] = [variance_inflation_factor(X.values, i) for i in range(X.shape[1])]

print(vif_data)
# VIF > 10 表示存在多重共线性问题
# VIF > 5 暗示中度多重共线性

# 条件数（来自摘要）
print(f"条件数: {results.condition_number}")
# 条件数 > 20 暗示多重共线性
# 条件数 > 30 表示严重问题
```

### 影响统计量

```python
from statsmodels.stats.outliers_influence import OLSInfluence

influence = results.get_influence()

# 杠杆值（帽子矩阵对角线）
leverage = influence.hat_matrix_diag
# 高杠杆：> 2*p/n (p=预测变量数, n=观测数)

# Cook距离
cooks_d = influence.cooks_distance[0]
# 强影响点：Cook距离 > 4/n

# DFFITS统计量
dffits = influence.dffits[0]
# 强影响点：|DFFITS| > 2*sqrt(p/n)

# 绘制影响图
from statsmodels.graphics.regressionplots import influence_plot
fig, ax = plt.subplots(figsize=(12, 8))
influence_plot(results, ax=ax)
plt.show()
```

### 假设检验

```python
# 单系数检验
# H0: beta_i = 0 (摘要中自动包含)

# 多重约束F检验
# 示例：检验 beta_1 = beta_2 = 0
R = [[0, 1, 0, 0], [0, 0, 1, 0]]  # 约束矩阵
f_test = results.f_test(R)
print(f_test)

# 基于公式的假设检验
f_test = results.f_test("x1 = x2 = 0")
print(f_test)

# 线性组合检验：beta_1 + beta_2 = 1
r_matrix = [[0, 1, 1, 0]]
q_matrix = [1]  # 右侧值
f_test = results.f_test((r_matrix, q_matrix))
print(f_test)

# Wald检验（线性约束下等价于F检验）
wald_test = results.wald_test(R)
print(wald_test)
```

## 模型比较

```python
# 嵌套模型似然比检验（使用MLE时）
from statsmodels.stats.anova import anova_lm

# 拟合约束与非约束模型
model_restricted = sm.OLS(y, X_restricted).fit()
model_full = sm.OLS(y, X_full).fit()

# 模型比较的ANOVA表
anova_results = anova_lm(model_restricted, model_full)
print(anova_results)

# 非嵌套模型的AIC/BIC比较
print(f"模型1 AIC: {model1.aic}, BIC: {model1.bic}")
print(f"模型2 AIC: {model2.aic}, BIC: {model2.bic}")
# AIC/BIC值越低模型越好
```

## 稳健标准误

无需重新加权即可处理异方差或聚类问题。

```python
# 异方差稳健标准误（HC）
results_hc = results.get_robustcov_results(cov_type='HC0')  # White方法
results_hc1 = results.get_robustcov_results(cov_type='HC1')
results_hc2 = results.get_robustcov_results(cov_type='HC2')
results_hc3 = results.get_robustcov_results(cov_type='HC3')  # 最保守

# Newey-West HAC（异方差和自相关一致）
results_hac = results.get_robustcov_results(cov_type='HAC', maxlags=4)

# 聚类稳健标准误
results_cluster = results.get_robustcov_results(cov_type='cluster',
                                                groups=cluster_ids)

# 查看稳健结果
print(results_hc3.summary())
```

## 最佳实践

1. **始终添加常数项**：除非明确排除截距，否则使用`sm.add_constant()`
2. **检验假设**：执行诊断检验（异方差、自相关、正态性）
3. **使用公式API处理分类变量**：`smf.ols()`自动处理分类变量
4. **稳健标准误**：当检测到异方差但模型设定正确时使用
5. **模型选择**：非嵌套模型用AIC/BIC，嵌套模型用F检验/似然比检验
6. **异常值与影响点**：务必检查Cook距离和杠杆值
7. **多重共线性**：解释前检查VIF和条件数
8. **时间序列**：存在自相关误差时使用`GLSAR`或稳健HAC标准误
9. **分组数据**：考虑混合效应模型或聚类稳健标准误
10. **分位数回归**：用于稳健估计或关注分布效应时

## 常见陷阱

1. **遗漏常数项**：导致无截距模型
2. **忽略异方差**：应使用WLS或稳健标准误
3. **在自相关误差中使用OLS**：应使用GLSAR或HAC标准误
4. **多重共线性下过度解释**：先检查VIF
5. **未检查残差**：务必绘制残差 vs 拟合值图
6. **使用t-SNE/PCA残差**：残差应来自原始空间
7. **混淆预测区间与置信区间**：预测区间更宽
8. **未正确处理分类变量**：使用公式API或手动虚拟编码
9. **比较不同样本量的模型**：确保使用相同观测值
10. **忽略强影响点**：检查Cook距离和DFFITS
