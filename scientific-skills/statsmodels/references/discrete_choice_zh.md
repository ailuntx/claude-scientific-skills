# 离散选择模型参考

本文档提供关于statsmodels中离散选择模型的全面指南，包括二元、多项、计数和有序模型。

## 概述

离散选择模型处理以下类型的结果：
- **二元**：0/1，成功/失败
- **多项**：多个无序类别
- **有序**：有序类别
- **计数**：非负整数

所有模型均使用最大似然估计并假设误差项独立同分布。

## 二元模型

### Logit（逻辑回归）

使用逻辑分布处理二元结果。

**适用场景：**
- 二元分类（是/否，成功/失败）
- 二元结果的概率估计
- 可解释的比值比

**模型**：P(Y=1|X) = 1 / (1 + exp(-Xβ))

```python
import statsmodels.api as sm
from statsmodels.discrete.discrete_model import Logit

# 准备数据
X = sm.add_constant(X_data)

# 拟合模型
model = Logit(y, X)
results = model.fit()

print(results.summary())
```

**结果解读：**
```python
import numpy as np

# 比值比
odds_ratios = np.exp(results.params)
print("比值比:", odds_ratios)

# X每增加1单位，事件发生比乘以exp(β)
# OR > 1：增加成功概率
# OR < 1：降低成功概率
# OR = 1：无影响

# 比值比的置信区间
odds_ci = np.exp(results.conf_int())
print("比值比95%置信区间:")
print(odds_ci)
```

**边际效应：**
```python
# 平均边际效应 (AME)
marginal_effects = results.get_margeff(at='mean')
print(marginal_effects.summary())

# 均值处边际效应 (MEM)
marginal_effects_mem = results.get_margeff(at='mean', method='dydx')

# 特定值处边际效应
marginal_effects_custom = results.get_margeff(at='mean',
                                              atexog={'x1': 1, 'x2': 5})
```

**预测：**
```python
# 预测概率
probs = results.predict(X)

# 二元预测 (0.5阈值)
predictions = (probs > 0.5).astype(int)

# 自定义阈值
threshold = 0.3
predictions_custom = (probs > threshold).astype(int)

# 新数据预测
X_new = sm.add_constant(X_new_data)
new_probs = results.predict(X_new)
```

**模型评估：**
```python
from sklearn.metrics import (classification_report, confusion_matrix,
                             roc_auc_score, roc_curve)

# 分类报告
print(classification_report(y, predictions))

# 混淆矩阵
print(confusion_matrix(y, predictions))

# AUC-ROC
auc = roc_auc_score(y, probs)
print(f"AUC: {auc:.4f}")

# 伪R方
print(f"McFadden伪R²: {results.prsquared:.4f}")
```

### Probit

使用正态分布处理二元结果。

**适用场景：**
- 二元结果
- 偏好正态分布假设
- 领域惯例（计量经济学常用probit）

**模型**：P(Y=1|X) = Φ(Xβ)，其中Φ为标准正态累积分布函数

```python
from statsmodels.discrete.discrete_model import Probit

model = Probit(y, X)
results = model.fit()

print(results.summary())
```

**与Logit比较：**
- Probit和Logit通常结果相似
- Probit：基于正态分布，对称
- Logit：尾部略重，更易解释（比值比）
- 系数不可直接比较（尺度差异）

```python
# 边际效应可比较
logit_me = logit_results.get_margeff().margeff
probit_me = probit_results.get_margeff().margeff

print("Logit边际效应:", logit_me)
print("Probit边际效应:", probit_me)
```

## 多项模型

### MNLogit（多项Logit）

处理3个以上无序类别结果。

**适用场景：**
- 多个无序类别（如交通方式、品牌选择）
- 类别间无自然顺序
- 需要各类别概率

**模型**：P(Y=j|X) = exp(Xβⱼ) / Σₖ exp(Xβₖ)

```python
from statsmodels.discrete.discrete_model import MNLogit

# y应为整数0,1,2,...表示类别
model = MNLogit(y, X)
results = model.fit()

print(results.summary())
```

**结果解读：**
```python
# 一个类别作为参照（通常为类别0）
# 系数表示相对于参照的对数发生比

# 类别j vs 参照：
# exp(β_j) = 类别j相对于参照的比值比

# 各类别预测概率
probs = results.predict(X)  # 形状: (n_samples, n_categories)

# 最可能类别
predicted_categories = probs.argmax(axis=1)
```

**相对风险比：**
```python
# 对系数取指数得相对风险比
import numpy as np
import pandas as pd

# 获取参数名和值
params_df = pd.DataFrame({
    'coef': results.params,
    'RRR': np.exp(results.params)
})
print(params_df)
```

### 条件Logit

处理备选项具有特征的模型。

**适用场景：**
- 备选项特定回归量（随选择变化）
- 含选择的纵向数据
- 离散选择实验

```python
from statsmodels.discrete.conditional_models import ConditionalLogit

# 数据结构：长格式含选择指示符
model = ConditionalLogit(y_choice, X_alternatives, groups=individual_id)
results = model.fit()
```

## 计数模型

### Poisson

标准计数数据模型。

**适用场景：**
- 计数结果（事件发生次数）
- 稀有事件
- 均值≈方差

**模型**：P(Y=k|X) = exp(-λ) λᵏ / k!，其中log(λ) = Xβ

```python
from statsmodels.discrete.count_model import Poisson

model = Poisson(y_counts, X)
results = model.fit()

print(results.summary())
```

**结果解读：**
```python
# 发生率比
rate_ratios = np.exp(results.params)
print("发生率比:", rate_ratios)

# X每增加1单位，期望计数乘以exp(β)
```

**过离散检验：**
```python
# Poisson要求均值≈方差
print(f"均值: {y_counts.mean():.2f}")
print(f"方差: {y_counts.var():.2f}")

# 正式检验
from statsmodels.stats.stattools import durbin_watson

# 若方差>>均值则存在过离散
# 经验法则：方差/均值>1.5表明过离散
overdispersion_ratio = y_counts.var() / y_counts.mean()
print(f"方差/均值: {overdispersion_ratio:.2f}")

if overdispersion_ratio > 1.5:
    print("建议使用负二项模型")
```

**含偏移项（用于比率）：**
```python
# 当建模具有不同暴露量的比率时
# log(λ) = log(暴露量) + Xβ

model = Poisson(y_counts, X, offset=np.log(exposure))
results = model.fit()
```

### 负二项分布

处理过离散计数数据（方差>均值）。

**适用场景：**
- 存在过离散的计数数据
- Poisson无法解释的额外方差
- 计数的异质性

**模型**：添加离散参数α解释过离散

```python
from statsmodels.discrete.count_model import NegativeBinomial

model = NegativeBinomial(y_counts, X)
results = model.fit()

print(results.summary())
print(f"离散参数alpha: {results.params['alpha']:.4f}")
```

**与Poisson比较：**
```python
# 拟合两个模型
poisson_results = Poisson(y_counts, X).fit()
nb_results = NegativeBinomial(y_counts, X).fit()

# AIC比较（值越低越好）
print(f"Poisson AIC: {poisson_results.aic:.2f}")
print(f"负二项分布 AIC: {nb_results.aic:.2f}")

# 似然比检验（判断NB是否更优）
from scipy import stats
lr_stat = 2 * (nb_results.llf - poisson_results.llf)
lr_pval = 1 - stats.chi2.cdf(lr_stat, df=1)  # 额外参数alpha
print(f"似然比检验p值: {lr_pval:.4f}")

if lr_pval < 0.05:
    print("负二项模型显著更优")
```

### 零膨胀模型

处理含过量零值的计数数据。

**适用场景：**
- 零值数量超过Poisson/NB预期
- 两个过程：零值生成和计数生成
- 示例：就诊次数、保险理赔次数

**模型：**
- ZeroInflatedPoisson (ZIP)
- ZeroInflatedNegativeBinomialP (ZINB)

```python
from statsmodels.discrete.count_model import (ZeroInflatedPoisson,
                                               ZeroInflatedNegativeBinomialP)

# ZIP模型
zip_model = ZeroInflatedPoisson(y_counts, X, exog_infl=X_inflation)
zip_results = zip_model.fit()

# ZINB模型（处理过离散+过量零值）
zinb_model = ZeroInflatedNegativeBinomialP(y_counts, X, exog_infl=X_inflation)
zinb_results = zinb_model.fit()

print(zip_results.summary())
```

**模型的两个部分：**
```python
# 1. 膨胀模型：因膨胀导致Y=0的概率
# 2. 计数模型：计数的分布

# 膨胀概率预测
inflation_probs = zip_results.predict(X, which='prob')

# 计数预测
predicted_counts = zip_results.predict(X, which='mean')
```

### Hurdle模型

两阶段模型：是否发生计数，然后计数多少。

**适用场景：**
- 过量零值
- 零值与正计数的生成机制不同
- 零值在结构上不同于正值

```python
from statsmodels.discrete.count_model import HurdleCountModel

# 指定计数分布和零值膨胀
model = HurdleCountModel(y_counts, X,
                         exog_infl=X_hurdle,
                         dist='poisson')  # 或'negbin'
results = model.fit()

print(results.summary())
```

## 有序模型

### 有序Logit/Probit

处理有序类别结果。

**适用场景：**
- 有序类别（如低/中/高，1-5级评分）
- 自然顺序重要
- 需保持序数结构

**模型**：含切点的累积概率模型

```python
from statsmodels.miscmodels.ordinal_model import OrderedModel

# y应为有序整数：0,1,2,...
model = OrderedModel(y_ordered, X, distr='logit')  # 或'probit'
results = model.fit(method='bfgs')

print(results.summary())
```

**结果解读：**
```python
# 切点（类别间阈值）
cutpoints = results.params[-n_categories+1:]
print("切点:", cutpoints)

# 系数
coefficients = results.params[:-n_categories+1]
print("系数:", coefficients)

# 各类别预测概率
probs = results.predict(X)  # 形状: (n_samples, n_categories)

# 最可能类别
predicted_categories = probs.argmax(axis=1)
```

**比例优势假设：**
```python
# 检验各切点系数是否相同
# (Brant检验 - 需手动实现或检查残差)

# 检查方法：分别建模各切点并比较系数
```

## 模型诊断

### 拟合优度

```python
# 伪R方 (McFadden)
print(f"伪R²: {results.prsquared:.4f}")

# 模型比较的AIC/BIC
print(f"AIC: {results.aic:.2f}")
print(f"BIC: {results.bic:.2f}")

# 对数似然值
print(f"对数似然值: {results.llf:.2f}")

# 与零模型的似然比检验
lr_stat = 2 * (results.llf - results.llnull)
from scipy import stats
lr_pval = 1 - stats.chi2.cdf(lr_stat, results.df_model)
print(f"似然比检验p值: {lr_pval}")
```

### 分类指标（二元）

```python
from sklearn.metrics import (accuracy_score, precision_score, recall_score,
                             f1_score, roc_auc_score)

# 预测
probs = results.predict(X)
predictions = (probs > 0.5).astype(int)

# 指标
print(f"准确率: {accuracy_score(y, predictions):.4f}")
print(f"精确率: {precision_score(y, predictions):.4f}")
print(f"召回率: {recall_score(y, predictions):.4f}")
print(f"F1值: {f1_score(y, predictions):.4f}")
print(f"AUC: {roc_auc_score(y, probs):.4f}")
```

### 分类指标（多项）

```python
from sklearn.metrics import accuracy_score, classification_report, log_loss

# 预测类别
probs = results.predict(X)
predictions = probs.argmax(axis=1)

# 准确率
accuracy = accuracy_score(y, predictions)
print(f"准确率: {accuracy:.4f}")

# 分类报告
print(classification_report(y, predictions))

# 对数损失
logloss = log_loss(y, probs)
print(f"对数损失: {logloss:.4f}")
```

### 计数模型诊断

```python
# 观测与预测频数
observed = pd.Series(y_counts).value_counts().sort_index()
predicted = results.predict(X)
predicted_counts = pd.Series(np.round(predicted)).value_counts().sort_index()

# 分布比较
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
observed.plot(kind='bar', alpha=0.5, label='观测值', ax=ax)
predicted_counts.plot(kind='bar', alpha=0.5, label='预测值', ax=ax)
ax.legend()
ax.set_xlabel('计数值')
ax.set_ylabel('频数')
plt.show()

# 根方图（更佳可视化）
from statsmodels.graphics.agreement import mean_diff_plot
# 需自定义根方图实现
```

### 影响点与异常值

```python
# 标准化残差
std_resid = (y - results.predict(X)) / np.sqrt(results.predict(X))

# 异常值检测 (|std_resid| > 2)
outliers = np.where(np.abs(std_resid) > 2)[0]
print(f"异常值数量: {len(outliers)}")

# 杠杆值（帽子矩阵）- 用于logit/probit
# from statsmodels.stats.outliers_influence
```

## 假设检验

```python
# 单参数检验（摘要中自动提供）

# 多参数：Wald检验
# 检验H0: β₁ = β₂ = 0
R = [[0, 1, 0, 0], [0, 0, 1, 0]]
wald_test = results.wald_test(R)
print(wald_test)

# 嵌套模型似然比检验
model_reduced = Logit(y, X_reduced).fit()
model_full = Logit(y, X_full).fit()

lr_stat = 2 * (model_full.llf - model_reduced.llf)
df = model_full.df_model - model_reduced.df_model
from scipy import stats
lr_pval = 1 - stats.chi2.cdf(lr_stat, df)
print(f"似然比检验p值: {lr_pval:.4f}")
```

## 模型选择与比较

```python
# 拟合多个模型
models = {
    'Logit': Logit(y, X).fit(),
    'Probit': Probit(y, X).fit(),
    # 添加更多模型
}

# 比较AIC/BIC
comparison = pd.DataFrame({
    'AIC': {name: model.aic for name, model in models.items()},
    'BIC': {name: model.bic for name, model in models.items()},
    '伪R²': {name: model.prsquared for name, model in models.items()}
})
print(comparison.sort_values('AIC'))

# 预测性能的交叉验证
from sklearn.model_selection import cross_val_score
from sklearn.linear_model import LogisticRegression

# 使用sklearn包装器或手动CV
```

## 公式API

使用R风格公式简化模型设定。

```python
import statsmodels.formula.api as smf

# 使用公式的Logit回归
formula = 'y ~ x1 + x2 + C(category) + x1:x2'
results = smf.logit(formula, data=df).fit()

# 使用公式的MNLogit回归
results = smf.mnlogit(formula, data=df).fit()

# 使用公式的泊松回归
results = smf.poisson(formula, data=df).fit()

# 使用公式的负二项回归
results = smf.negativebinomial(formula, data=df).fit()
```

## 常见应用场景

### 二元分类（营销响应）

```python
# 预测客户购买概率
X = sm.add_constant(customer_features)
model = Logit(purchased, X)
results = model.fit()

# 目标定位：选择前20%最可能购买的用户
probs = results.predict(X)
top_20_pct_idx = np.argsort(probs)[-int(0.2*len(probs)):]
```

### 多项选择（交通方式）

```python
# 预测交通方式选择
model = MNLogit(mode_choice, X)
results = model.fit()

# 新通勤者的预测交通方式
new_commuter = sm.add_constant(new_features)
mode_probs = results.predict(new_commuter)
predicted_mode = mode_probs.argmax(axis=1)
```

### 计数数据（就诊次数）

```python
# 医疗资源使用建模
model = NegativeBinomial(num_visits, X)
results = model.fit()

# 新患者的预期就诊次数
expected_visits = results.predict(new_patient_X)
```

### 零膨胀模型（保险理赔）

```python
# 多数人零理赔
# 零膨胀：部分人从不理赔
# 计数过程：可能理赔的人群

zip_model = ZeroInflatedPoisson(claims, X_count, exog_infl=X_inflation)
results = zip_model.fit()

# P(从不理赔)
never_claim_prob = results.predict(X, which='prob-zero')

# 预期理赔次数
expected_claims = results.predict(X, which='mean')
```

## 最佳实践

1. **检查数据类型**：确保响应变量匹配模型（二元/计数/分类）
2. **添加常数项**：除非不需要截距，否则始终使用`sm.add_constant()`
3. **标准化连续预测变量**：提升收敛性和可解释性
4. **检查收敛性**：关注收敛警告
5. **使用公式API**：便于处理分类变量和交互项
6. **边际效应**：报告边际效应而非仅系数
7. **模型比较**：使用AIC/BIC和交叉验证
8. **验证**：预测模型需保留验证集或交叉验证
9. **检查过度离散**：计数模型需检验泊松假设
10. **考虑替代模型**：零膨胀/跨栏模型处理过量零值

## 常见陷阱

1. **遗漏常数项**：缺失截距项
2. **完全分离**：Logit/probit模型可能不收敛
3. **过度离散时使用泊松回归**：应改用负二项回归
4. **错误解释系数**：系数处于对数几率/对数尺度
5. **忽略收敛检查**：优化可能静默失败
6. **错误分布假设**：模型需匹配数据类型（二元/计数/分类）
7. **忽视过量零值**：适用时采用ZIP/ZINB模型
8. **未验证预测**：必须检查样本外表现
9. **比较非嵌套模型**：使用AIC/BIC而非似然比检验
10. **将有序变量视为无序**：有序类别应使用OrderedModel```
