# 广义线性模型（GLM）参考指南

本文档提供关于statsmodels中广义线性模型的全面指导，包括分布族、连接函数和应用场景。

## 概述

GLM通过以下方式将线性回归扩展到非正态响应分布：
1. **分布族**：指定响应的条件分布
2. **连接函数**：将线性预测器转换到均值尺度
3. **方差函数**：建立方差与均值的关联关系

**通用形式**：g(μ) = Xβ，其中g是连接函数，μ = E(Y|X)

## 适用场景

- **二元结果**：逻辑回归（二项分布族 + logit连接）
- **计数数据**：泊松或负二项回归
- **正连续数据**：伽玛或逆高斯分布
- **非正态分布**：当OLS假设不成立时
- **连接函数**：需要预测变量与响应尺度间的非线性关系

## 分布族

### 二项分布族

适用于二元结果（0/1）或比例数据（k/n）。

**适用场景：**
- 二元分类
- 成功/失败结果
- 比例或比率数据

**常用连接函数：**
- Logit（默认）：log(μ/(1-μ))
- Probit：Φ⁻¹(μ)
- Log：log(μ)

```python
import statsmodels.api as sm
import statsmodels.formula.api as smf

# 二元逻辑回归
model = sm.GLM(y, X, family=sm.families.Binomial())
results = model.fit()

# 公式API
results = smf.glm('success ~ x1 + x2', data=df,
                  family=sm.families.Binomial()).fit()

# 获取预测概率
probs = results.predict(X_new)

# 分类（0.5阈值）
predictions = (probs > 0.5).astype(int)
```

**结果解释：**
```python
import numpy as np

# 比值比（logit连接）
odds_ratios = np.exp(results.params)
print("比值比:", odds_ratios)

# x每增加1单位，比值乘以exp(beta)
```

### 泊松分布族

适用于计数数据（非负整数）。

**适用场景：**
- 计数结果（事件发生次数）
- 稀有事件
- 比率建模（需配合偏移量）

**常用连接函数：**
- Log（默认）：log(μ)
- Identity：μ
- Sqrt：√μ

```python
# 泊松回归
model = sm.GLM(y, X, family=sm.families.Poisson())
results = model.fit()

# 带暴露量/偏移量的比率建模
# 若建模比率 = 计数/暴露量
model = sm.GLM(y, X, family=sm.families.Poisson(),
               offset=np.log(exposure))
results = model.fit()

# 解释：exp(beta) = 期望计数的乘数效应
import numpy as np
rate_ratios = np.exp(results.params)
print("比率乘数:", rate_ratios)
```

**过离散检验：**
```python
# 泊松模型的离差/自由度应≈1
overdispersion = results.deviance / results.df_resid
print(f"过离散度: {overdispersion}")

# 若>>1，考虑负二项模型
if overdispersion > 1.5:
    print("存在过离散，建议使用负二项模型")
```

### 负二项分布族

适用于过离散计数数据。

**适用场景：**
- 方差>均值的计数数据
- 零值过多或大方差
- 泊松模型显示过离散

```python
# 负二项GLM
model = sm.GLM(y, X, family=sm.families.NegativeBinomial())
results = model.fit()

# 替代方案：使用带alpha估计的离散选择模型
from statsmodels.discrete.discrete_model import NegativeBinomial
nb_model = NegativeBinomial(y, X)
nb_results = nb_model.fit()

print(f"离散参数alpha: {nb_results.params[-1]}")
```

### 高斯分布族

等价于OLS，但通过IRLS（迭代重加权最小二乘法）拟合。

**适用场景：**
- 需要保持GLM框架一致性
- 需稳健标准误
- 与其他GLM比较

**常用连接函数：**
- Identity（默认）：μ
- Log：log(μ)
- Inverse：1/μ

```python
# 高斯GLM（等价于OLS）
model = sm.GLM(y, X, family=sm.families.Gaussian())
results = model.fit()

# 验证与OLS等价性
ols_results = sm.OLS(y, X).fit()
print("参数一致性:", np.allclose(results.params, ols_results.params))
```

### 伽玛分布族

适用于正连续数据，常呈右偏分布。

**适用场景：**
- 正数结果（保险理赔额、生存时间）
- 右偏分布
- 方差与均值²成比例

**常用连接函数：**
- Inverse（默认）：1/μ
- Log：log(μ)
- Identity：μ

```python
# 伽玛回归（常用于成本数据）
model = sm.GLM(y, X, family=sm.families.Gamma())
results = model.fit()

# Log连接通常更易解释
model = sm.GLM(y, X, family=sm.families.Gamma(link=sm.families.links.Log()))
results = model.fit()

# 使用log连接时，exp(beta)=乘数效应
import numpy as np
effects = np.exp(results.params)
```

### 逆高斯分布族

适用于特定方差结构的正连续数据。

**适用场景：**
- 正偏态结果
- 方差与均值³成比例
- 伽玛分布的替代方案

**常用连接函数：**
- Inverse squared（默认）：1/μ²
- Log：log(μ)

```python
model = sm.GLM(y, X, family=sm.families.InverseGaussian())
results = model.fit()
```

### Tweedie分布族

覆盖多种分布的灵活分布族。

**适用场景：**
- 保险理赔数据（零值与连续值混合）
- 半连续数据
- 需要灵活方差函数

**特例（幂参数p）：**
- p=0：正态分布
- p=1：泊松分布
- p=2：伽玛分布
- p=3：逆高斯分布
- 1<p<2：复合泊松-伽玛分布（保险常用）

```python
# Tweedie模型（power=1.5）
model = sm.GLM(y, X, family=sm.families.Tweedie(link=sm.families.links.Log(),
                                                 var_power=1.5))
results = model.fit()
```

## 连接函数

连接函数将线性预测器与响应均值关联起来。

### 可用连接函数

```python
from statsmodels.genmod import families

# Identity: g(μ) = μ
link = families.links.Identity()

# Log: g(μ) = log(μ)
link = families.links.Log()

# Logit: g(μ) = log(μ/(1-μ))
link = families.links.Logit()

# Probit: g(μ) = Φ⁻¹(μ)
link = families.links.Probit()

# Complementary log-log: g(μ) = log(-log(1-μ))
link = families.links.CLogLog()

# Inverse: g(μ) = 1/μ
link = families.links.InversePower()

# Inverse squared: g(μ) = 1/μ²
link = families.links.InverseSquared()

# Square root: g(μ) = √μ
link = families.links.Sqrt()

# Power: g(μ) = μ^p
link = families.links.Power(power=2)
```

### 选择连接函数

**标准连接**（各分布族默认）：
- 二项分布 → Logit
- 泊松分布 → Log
- 伽玛分布 → Inverse
- 高斯分布 → Identity
- 逆高斯分布 → Inverse squared

**非标准连接适用场景：**
- **二项分布用Log连接**：获取风险比而非比值比
- **Identity连接**：直接加性效应（当合理时）
- **Probit vs Logit**：结果相似，依领域偏好选择
- **CLogLog**：非对称关系，生存分析常用

```python
# 示例：Log-二项模型获取风险比
model = sm.GLM(y, X, family=sm.families.Binomial(link=sm.families.links.Log()))
results = model.fit()

# exp(beta)给出风险比而非比值比
risk_ratios = np.exp(results.params)
```

## 模型拟合与结果

### 基础流程

```python
import statsmodels.api as sm

# 添加常数项
X = sm.add_constant(X_data)

# 指定分布族和连接函数
family = sm.families.Poisson(link=sm.families.links.Log())

# 使用IRLS拟合模型
model = sm.GLM(y, X, family=family)
results = model.fit()

# 结果摘要
print(results.summary())
```

### 结果属性

```python
# 参数与推断
results.params              # 系数
results.bse                 # 标准误
results.tvalues            # Z统计量
results.pvalues            # P值
results.conf_int()         # 置信区间

# 预测
results.fittedvalues       # 拟合值 (μ)
results.predict(X_new)     # 新数据预测

# 模型拟合统计量
results.aic                # 赤池信息准则
results.bic                # 贝叶斯信息准则
results.deviance           # 离差
results.null_deviance      # 零模型离差
results.pearson_chi2       # 皮尔逊卡方统计量
results.df_resid           # 残差自由度
results.llf                # 对数似然值

# 残差
results.resid_response     # 响应残差 (y - μ)
results.resid_pearson      # 皮尔逊残差
results.resid_deviance     # 离差残差
results.resid_anscombe     # 安斯科姆残差
results.resid_working      # 工作残差
```

### 伪R方

```python
# McFadden伪R方
pseudo_r2 = 1 - (results.deviance / results.null_deviance)
print(f"伪R²: {pseudo_r2:.4f}")

# 调整伪R方
n = len(y)
k = len(results.params)
adj_pseudo_r2 = 1 - ((n-1)/(n-k)) * (results.deviance / results.null_deviance)
print(f"调整伪R²: {adj_pseudo_r2:.4f}")
```

## 模型诊断

### 拟合优度

```python
# 离差应近似服从自由度为df_resid的χ²分布
from scipy import stats

deviance_pval = 1 - stats.chi2.cdf(results.deviance, results.df_resid)
print(f"离差检验p值: {deviance_pval}")

# 皮尔逊卡方检验
pearson_pval = 1 - stats.chi2.cdf(results.pearson_chi2, results.df_resid)
print(f"皮尔逊卡方检验p值: {pearson_pval}")

# 检查过离散/欠离散
dispersion = results.pearson_chi2 / results.df_resid
print(f"离散度: {dispersion}")
# 应≈1；>1提示过离散，<1提示欠离散
```

### 残差分析

```python
import matplotlib.pyplot as plt

# 离差残差 vs 拟合值
plt.figure(figsize=(10, 6))
plt.scatter(results.fittedvalues, results.resid_deviance, alpha=0.5)
plt.xlabel('拟合值')
plt.ylabel('离差残差')
plt.axhline(y=0, color='r', linestyle='--')
plt.title('离差残差 vs 拟合值')
plt.show()

# 离差残差Q-Q图
from statsmodels.graphics.gofplots import qqplot
qqplot(results.resid_deviance, line='s')
plt.title('离差残差Q-Q图')
plt.show()

# 二元结果：分组残差图
if isinstance(results.model.family, sm.families.Binomial):
    from statsmodels.graphics.gofplots import qqplot
    # 分组预测并计算平均残差
    # (需自定义实现)
    pass
```

### 影响点与异常值

```python
from statsmodels.stats.outliers_influence import GLMInfluence

influence = GLMInfluence(results)

# 杠杆值
leverage = influence.hat_matrix_diag

# Cook距离
cooks_d = influence.cooks_distance[0]

# DFFITS统计量
dffits = influence.dffits[0]

# 识别强影响点
influential = np.where(cooks_d > 4/len(y))[0]
print(f"强影响观测点: {influential}")
```

## 假设检验

```python
# 单参数Wald检验（结果摘要中自动包含）

# 嵌套模型似然比检验
# 拟合简化模型
model_reduced = sm.GLM(y, X_reduced, family=family).fit()
model_full = sm.GLM(y, X_full, family=family).fit()

# LR统计量
lr_stat = 2 * (model_full.llf - model_reduced.llf)
df = model_full.df_model - model_reduced.df_model

from scipy import stats
lr_pval = 1 - stats.chi2.cdf(lr_stat, df)
print(f"LR检验p值: {lr_pval}")

# 多参数Wald检验
# 检验beta_1 = beta_2 = 0
R = [[0, 1, 0, 0], [0, 0, 1, 0]]
wald_test = results.wald_test(R)
print(wald_test)
```

## 稳健标准误

```python
# 异方差稳健标准误（三明治估计量）
results_robust = results.get_robustcov_results(cov_type='HC0')

# 聚类稳健标准误
results_cluster = results.get_robustcov_results(cov_type='cluster',
                                                groups=cluster_ids)

# 比较标准误
print("常规标准误:", results.bse)
print("稳健标准误:", results_robust.bse)
```

## 模型比较

```python
# 非嵌套模型的AIC/BIC
models = [model1_results, model2_results, model3_results]
for i, res in enumerate(models, 1):
    print(f"模型 {i}: AIC={res.aic:.2f}, BIC={res.bic:.2f}")

# 嵌套模型似然比检验（如前所示）

# 预测性能交叉验证
from sklearn.model_selection import KFold
from sklearn.metrics import log_loss

kf = KFold(n_splits=5, shuffle=True, random_state=42)
cv_scores = []

for train_idx, val_idx in kf.split(X):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]

    model_cv = sm.GLM(y_train, X_train, family=family).fit()
    pred_probs = model_cv.predict(X_val)

    score = log_loss(y_val, pred_probs)
    cv_scores.append(score)

print(f"交叉验证对数损失: {np.mean(cv_scores):.4f} ± {np.std(cv_scores):.4f}")
```

## 预测

```python
# 点预测
predictions = results.predict(X_new)

# 分类任务：获取概率并转换
if isinstance(family, sm.families.Binomial):
    probs = predictions
    class_predictions = (probs > 0.5).astype(int)

# 计数数据：预测值为期望计数
if isinstance(family, sm.families.Poisson):
    expected_counts = predictions

# 自助法预测区间
n_boot = 1000
boot_preds = np.zeros((n_boot, len(X_new)))

for i in range(n_boot):
    # 自助法重抽样
    boot_idx = np.random.choice(len(y), size=len(y), replace=True)
    X_boot, y_boot = X[boot_idx], y[boot_idx]

    # 拟合并预测
    boot_model = sm.GLM(y_boot, X_boot, family=family).fit()
    boot_preds[i] = boot_model.predict(X_new)

# 95%预测区间
pred_lower = np.percentile(boot_preds, 2.5, axis=0)
pred_upper = np.percentile(boot_preds, 97.5, axis=0)
```

## 典型应用

### 逻辑回归（二元分类）

```python
import statsmodels.api as sm

# 拟合逻辑回归
X = sm.add_constant(X_data)
model = sm.GLM(y, X, family=sm.families.Binomial())
results = model.fit()

# 比值比
odds_ratios = np

```python
odds_ci = np.exp(results.conf_int())

# 分类指标
from sklearn.metrics import classification_report, roc_auc_score

probs = results.predict(X)
predictions = (probs > 0.5).astype(int)

print(classification_report(y, predictions))
print(f"AUC: {roc_auc_score(y, probs):.4f}")

# ROC曲线
from sklearn.metrics import roc_curve
import matplotlib.pyplot as plt

fpr, tpr, thresholds = roc_curve(y, probs)
plt.plot(fpr, tpr)
plt.plot([0, 1], [0, 1], 'k--')
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve')
plt.show()
```

### 泊松回归（计数数据）

```python
# 拟合泊松模型
X = sm.add_constant(X_data)
model = sm.GLM(y_counts, X, family=sm.families.Poisson())
results = model.fit()

# 比率比
rate_ratios = np.exp(results.params)
print("Rate ratios:", rate_ratios)

# 检查过度离散
dispersion = results.pearson_chi2 / results.df_resid
if dispersion > 1.5:
    print(f"Overdispersion detected ({dispersion:.2f}). Consider Negative Binomial.")
```

### Gamma回归（成本/时长数据）

```python
# 使用对数链接拟合Gamma模型
X = sm.add_constant(X_data)
model = sm.GLM(y_cost, X,
               family=sm.families.Gamma(link=sm.families.links.Log()))
results = model.fit()

# 乘性效应
effects = np.exp(results.params)
print("Multiplicative effects on mean:", effects)
```

## 最佳实践

1. **检查分布假设**：绘制响应变量的直方图和Q-Q图
2. **验证链接函数**：除非有特殊原因，否则使用规范链接
3. **检验残差**：偏差残差应近似服从正态分布
4. **测试过度离散**：特别是针对泊松模型
5. **正确使用偏移量**：用于不同暴露条件下的速率建模
6. **采用稳健标准误**：当方差假设存疑时
7. **模型比较**：非嵌套模型用AIC/BIC，嵌套模型用似然比检验
8. **原始尺度解释**：转换系数（如对数链接取指数）
9. **检查强影响点**：使用库克距离
10. **验证预测效果**：使用交叉验证或保留数据集

## 常见陷阱

1. **遗漏常数项**：未添加截距项
2. **选错分布族**：未检查响应变量的分布
3. **忽视过度离散**：应使用负二项回归替代泊松回归
4. **错误解释系数**：忽略链接函数的转换作用
5. **未检查收敛性**：IRLS算法可能未收敛，需检查警告
6. **逻辑回归完全分离**：某些类别完美预测结果
7. **对有界结果使用恒等链接**：可能导致超出有效范围的预测
8. **不同样本的模型比较**：应使用相同观测数据
9. **速率模型遗漏偏移量**：必须使用log(暴露量)作为偏移量
10. **忽略替代方案**：复杂数据应考虑混合模型、零膨胀模型等
