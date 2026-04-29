---
name: statsmodels
description: Python统计模型库。当需要特定模型类（OLS、GLM、混合模型、ARIMA）并需详细诊断、残差分析和统计推断时使用。最适用于计量经济学、时间序列分析及带系数表的严格推断。若需带APA报告的统计检验指导，请使用statistical-analysis。
license: BSD-3-Clause许可证
metadata:
    skill-author: K-Dense公司
---

# Statsmodels：统计建模与计量经济学

## 概述

Statsmodels是Python首屈一指的统计建模库，提供涵盖广泛统计方法的估计、推断和诊断工具。应用此技能可进行从简单线性回归到复杂时间序列模型及计量经济分析的严谨统计分析。

## 使用场景

在以下场景应使用此技能：
- 拟合回归模型（OLS、WLS、GLS、分位数回归）
- 执行广义线性建模（逻辑回归、泊松回归、伽马回归等）
- 分析离散结果（二元、多元、计数、有序）
- 进行时间序列分析（ARIMA、SARIMAX、VAR、预测）
- 运行统计检验与诊断
- 检验模型假设（异方差性、自相关性、正态性）
- 检测离群值与强影响观测点
- 模型比较（AIC/BIC、似然比检验）
- 估计因果效应
- 生成可直接发表的统计表格与推断结果

## 快速入门指南

### 线性回归（OLS）

```python
import statsmodels.api as sm
import numpy as np
import pandas as pd

# 准备数据 - 务必添加常数项作为截距
X = sm.add_constant(X_data)

# 拟合OLS模型
model = sm.OLS(y, X)
results = model.fit()

# 查看完整结果
print(results.summary())

# 关键结果
print(f"R方值: {results.rsquared:.4f}")
print(f"系数:\\n{results.params}")
print(f"P值:\\n{results.pvalues}")

# 带置信区间的预测
predictions = results.get_prediction(X_new)
pred_summary = predictions.summary_frame()
print(pred_summary)  # 包含均值、置信区间、预测区间

# 诊断检验
from statsmodels.stats.diagnostic import het_breuschpagan
bp_test = het_breuschpagan(results.resid, X)
print(f"Breusch-Pagan检验p值: {bp_test[1]:.4f}")

# 可视化残差
import matplotlib.pyplot as plt
plt.scatter(results.fittedvalues, results.resid)
plt.axhline(y=0, color='r', linestyle='--')
plt.xlabel('拟合值')
plt.ylabel('残差')
plt.show()
```

### 逻辑回归（二元结果）

```python
from statsmodels.discrete.discrete_model import Logit

# 添加常数项
X = sm.add_constant(X_data)

# 拟合logit模型
model = Logit(y_binary, X)
results = model.fit()

print(results.summary())

# 比值比
odds_ratios = np.exp(results.params)
print("比值比:\\n", odds_ratios)

# 预测概率
probs = results.predict(X)

# 二元预测（0.5阈值）
predictions = (probs > 0.5).astype(int)

# 模型评估
from sklearn.metrics import classification_report, roc_auc_score

print(classification_report(y_binary, predictions))
print(f"AUC值: {roc_auc_score(y_binary, probs):.4f}")

# 边际效应
marginal = results.get_margeff()
print(marginal.summary())
```

### 时间序列（ARIMA）

```python
from statsmodels.tsa.arima.model import ARIMA
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf

# 检验平稳性
from statsmodels.tsa.stattools import adfuller

adf_result = adfuller(y_series)
print(f"ADF检验p值: {adf_result[1]:.4f}")

if adf_result[1] > 0.05:
    # 序列非平稳，进行差分
    y_diff = y_series.diff().dropna()

# 绘制ACF/PACF确定p,q阶数
fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 8))
plot_acf(y_diff, lags=40, ax=ax1)
plot_pacf(y_diff, lags=40, ax=ax2)
plt.show()

# 拟合ARIMA(p,d,q)
model = ARIMA(y_series, order=(1, 1, 1))
results = model.fit()

print(results.summary())

# 预测
forecast = results.forecast(steps=10)
forecast_obj = results.get_forecast(steps=10)
forecast_df = forecast_obj.summary_frame()

print(forecast_df)  # 包含均值与置信区间

# 残差诊断
results.plot_diagnostics(figsize=(12, 8))
plt.show()
```

### 广义线性模型（GLM）

```python
import statsmodels.api as sm

# 针对计数数据的泊松回归
X = sm.add_constant(X_data)
model = sm.GLM(y_counts, X, family=sm.families.Poisson())
results = model.fit()

print(results.summary())

# 比率比（泊松对数连接）
rate_ratios = np.exp(results.params)
print("比率比:\\n", rate_ratios)

# 检验过离散
overdispersion = results.pearson_chi2 / results.df_resid
print(f"过离散系数: {overdispersion:.2f}")

if overdispersion > 1.5:
    # 改用负二项分布
    from statsmodels.discrete.count_model import NegativeBinomial
    nb_model = NegativeBinomial(y_counts, X)
    nb_results = nb_model.fit()
    print(nb_results.summary())
```

## 核心统计建模能力

### 1. 线性回归模型

针对连续型结果的完整线性模型套件，支持多种误差结构。

**可用模型：**
- **OLS**：带独立同分布误差的标准线性回归
- **WLS**：处理异方差误差的加权最小二乘法
- **GLS**：处理任意协方差结构的广义最小二乘法
- **GLSAR**：带自回归误差的时间序列GLS
- **分位数回归**：条件分位数估计（对离群值稳健）
- **混合效应**：含随机效应的分层/多水平模型
- **递归/滚动回归**：时变参数估计

**关键特性：**
- 全面诊断检验
- 稳健标准误（HC、HAC、聚类稳健）
- 影响统计量（库克距离、杠杆值、DFFITS）
- 假设检验（F检验、Wald检验）
- 模型比较（AIC、BIC、似然比检验）
- 带置信区间和预测区间的预测

**使用场景：** 连续型结果变量，需系数推断与诊断

**参考：** 模型选择、诊断与最佳实践详见`references/linear_models.md`

### 2. 广义线性模型（GLM）

将线性模型扩展至非正态分布的灵活框架。

**分布族：**
- **二项分布**：二元结果或比例（逻辑回归）
- **泊松分布**：计数数据
- **负二项分布**：过离散计数
- **伽马分布**：右偏的正连续数据
- **逆高斯分布**：特定方差结构的正连续数据
- **高斯分布**：等价于OLS
- **Tweedie分布**：半连续数据的灵活分布族

**连接函数：**
- Logit、Probit、Log、Identity、Inverse、Sqrt、CLogLog、Power
- 根据解释需求与模型拟合选择

**关键特性：**
- 通过IRLS进行最大似然估计
- 偏差与皮尔逊残差
- 拟合优度统计量
- 伪R方度量
- 稳健标准误

**使用场景：** 非正态结果，需灵活方差与连接函数设定

**参考：** 分布族选择、连接函数、解释与诊断详见`references/glm.md`

### 3. 离散选择模型

处理分类与计数结果的模型。

**二元模型：**
- **Logit**：逻辑回归（比值比）
- **Probit**：Probit回归（正态分布）

**多元模型：**
- **MNLogit**：无序分类（3+类别）
- **条件Logit**：含替代特定变量的选择模型
- **有序模型**：有序结果（等级分类）

**计数模型：**
- **泊松模型**：标准计数模型
- **负二项模型**：过离散计数
- **零膨胀模型**：过量零值（ZIP、ZINB）
- **障碍模型**：针对零值密集数据的两阶段模型

**关键特性：**
- 最大似然估计
- 均值处边际效应或平均边际效应
- 通过AIC/BIC进行模型比较
- 预测概率与分类
- 拟合优度检验

**使用场景：** 二元、分类或计数结果

**参考：** 模型选择、解释与评估详见`references/discrete_choice.md`

### 4. 时间序列分析

全面的时间序列建模与预测能力。

**单变量模型：**
- **自回归模型（AR）**
- **ARIMA**：自回归综合移动平均
- **SARIMAX**：含外生变量的季节性ARIMA
- **指数平滑**：简单指数、Holt、Holt-Winters
- **ETS**：创新状态空间模型

**多变量模型：**
- **VAR**：向量自回归
- **VARMAX**：含MA项和外生变量的VAR
- **动态因子模型**：提取公共因子
- **VECM**：向量误差修正模型（协整）

**高级模型：**
- **状态空间**：卡尔曼滤波、自定义设定
- **体制转换**：马尔可夫转换模型
- **ARDL**：自回归分布滞后

**关键特性：**
- 通过ACF/PACF分析识别模型
- 平稳性检验（ADF、KPSS）
- 带预测区间的预测
- 残差诊断（Ljung-Box、异方差性）
- 格兰杰因果检验
- 脉冲响应函数（IRF）
- 预测误差方差分解（FEVD）

**使用场景：** 时间序列数据、预测、理解时序动态

**参考：** 模型选择、诊断与预测方法详见`references/time_series.md`

### 5. 统计检验与诊断

丰富的模型验证测试与诊断能力。

**残差诊断：**
- 自相关检验（Ljung-Box、Durbin-Watson、Breusch-Godfrey）
- 异方差性检验（Breusch-Pagan、White、ARCH）
- 正态性检验（Jarque-Bera、Omnibus、Anderson-Darling、Lilliefors）
- 设定检验（RESET、Harvey-Collier）

**影响与离群值：**
- 杠杆值（帽子矩阵）
- 库克距离
- DFFITS与DFBETAs
- 学生化残差
- 影响图

**假设检验：**
- t检验（单样本、双样本、配对）
- 比例检验
- 卡方检验
- 非参数检验（Mann-Whitney、Wilcoxon、Kruskal-Wallis）
- 方差分析（单因素、双因素、重复测量）

**多重比较：**
- Tukey HSD法
- Bonferroni校正
- 错误发现率（FDR）

**效应量与功效：**
- Cohen's d、eta方
- t检验与比例检验的功效分析
- 样本量计算

**稳健推断：**
- 异方差稳健标准误（HC0-HC3）
- HAC标准误（Newey-West）
- 聚类稳健标准误

**使用场景：** 验证假设、检测问题、确保稳健推断

**参考：** 完整测试与诊断流程详见`references/stats_diagnostics.md`

## 公式API（R语言风格）

Statsmodels支持R语言风格公式进行直观模型设定：

```python
import statsmodels.formula.api as smf

# 公式法OLS
results = smf.ols('y ~ x1 + x2 + x1:x2', data=df).fit()

# 分类变量（自动虚拟编码）
results = smf.ols('y ~ x1 + C(category)', data=df).fit()

# 交互项
results = smf.ols('y ~ x1 * x2', data=df).fit()  # 等价于 x1 + x2 + x1:x2

# 多项式项
results = smf.ols('y ~ x + I(x**2)', data=df).fit()

# Logit模型
results = smf.logit('y ~ x1 + x2 + C(group)', data=df).fit()

# 泊松模型
results = smf.poisson('count ~ x1 + x2', data=df).fit()

# ARIMA（不支持公式，需用常规API）
```

## 模型选择与比较

### 信息准则

```python
# 使用AIC/BIC比较模型
models = {
    '模型1': model1_results,
    '模型2': model2_results,
    '模型3': model3_results
}

comparison = pd.DataFrame({
    'AIC': {name: res.aic for name, res in models.items()},
    'BIC': {name: res.bic for name, res in models.items()},
    '对数似然值': {name: res.llf for name, res in models.items()}
})

print(comparison.sort_values('AIC'))
# AIC/BIC值越低模型越优
```

### 似然比检验（嵌套模型）

```python
# 针对嵌套模型（一个模型是另一个的子集）
from scipy import stats

lr_stat = 2 * (full_model.llf - reduced_model.llf)
df = full_model.df_model - reduced_model.df_model
p_value = 1 - stats.chi2.cdf(lr_stat, df)

print(f"似然比统计量: {lr_stat:.4f}")
print(f"p值: {p_value:.4f}")

if p_value < 0.05:
    print("完整模型显著更优")
else:
    print("建议选择简化模型（简约性）")
```

### 交叉验证

```python
from sklearn.model_selection import KFold
from sklearn.metrics import mean_squared_error

kf = KFold(n_splits=5, shuffle=True, random_state=42)
cv_scores = []

for train_idx, val_idx in kf.split(X):
    X_train, X_val = X.iloc[train_idx], X.iloc[val_idx]
    y_train, y_val = y.iloc[train_idx], y.iloc[val_idx]

    # 拟合模型
    model = sm.OLS(y_train, X_train).fit()

    # 预测
    y_pred = model.predict(X_val)

    # 评分
    rmse = np.sqrt(mean_squared_error(y_val, y_pred))
    cv_scores.append(rmse)

print(f"交叉验证RMSE: {np.mean(cv_scores):.4f} ± {np.std(cv_scores):.4f}")
```

## 最佳实践

### 数据准备

1. **始终添加常数项**：除非排除截距，否则使用`sm.add_constant()`
2. **处理缺失值**：拟合前需处理或填补
3. **必要时标准化**：提升收敛速度与解释性（树模型无需）
4. **编码分类变量**：使用公式API或手动虚拟编码

### 模型构建

1. **从简开始**：先构建基础模型，逐步增加复杂度
2. **检验假设**：测试残差、异方差性、自相关性
3. **选用适当模型**：匹配结果类型（二元→Logit，计数→泊松）
4. **考虑替代方案**：若假设被违反，改用稳健方法或不同模型

### 统计推断

1. **报告效应量**：不仅报告p值
2. **使用稳健标准误**：存在异方差或聚类时
3. **多重比较校正**：检验多个假设时需校正
4. **置信区间**：始终与点估计值同时报告

### 模型评估

1. **检验残差**：绘制残差-拟合值图、Q-Q图
2. **影响诊断**：识别并检查强影响观测点
3. **样本外验证**：在保留集或交叉验证中测试
4. **模型比较**：非嵌套模型用AIC/BIC，嵌套模型用似然比检验

### 结果报告

1. **完整摘要**：使用`.summary()`输出详细信息
2. **记录决策**：注明数据转换、排除的观测点
3. **谨慎解释**：考虑连接函数（如对数连接的exp(β)）
4. **可视化**：绘制预测值、置信区间、诊断图

## 典型工作流

### 工作流1：线性回归分析

1. 探索数据（绘图、描述统计）
2. 拟合初始OLS模型
3. 检查残差诊断
4. 检验异方差性与自相关性
5. 检查多重共线性（VIF）
6. 识别强影响观测点
7. 必要时用稳健标准误重新拟合

8. 解释系数并进行推断  
9. 在保留集或通过交叉验证进行验证  

### 工作流程2：二分类问题  

1. 拟合逻辑回归（Logit）  
2. 检查收敛问题  
3. 解释比值比（Odds Ratios）  
4. 计算边际效应  
5. 评估分类性能（AUC、混淆矩阵）  
6. 检查强影响观测点  
7. 与替代模型比较（如Probit）  
8. 在测试集上验证预测结果  

### 工作流程3：计数数据分析  

1. 拟合泊松回归  
2. 检查过度离散性  
3. 若存在过度离散，拟合负二项回归  
4. 检查零膨胀（考虑ZIP/ZINB模型）  
5. 解释比率比（Rate Ratios）  
6. 评估模型拟合优度  
7. 通过AIC比较模型  
8. 验证预测结果  

### 工作流程4：时间序列预测  

1. 绘制序列图，检查趋势/季节性  
2. 平稳性检验（ADF、KPSS）  
3. 非平稳时进行差分处理  
4. 通过ACF/PACF确定p、q阶数  
5. 拟合ARIMA或SARIMAX模型  
6. 检查残差诊断（Ljung-Box检验）  
7. 生成带置信区间的预测  
8. 在测试集评估预测准确性  

## 参考文档  

本技能包含详细指导的完整参考文件：  

### references/linear_models.md  
线性回归模型全面覆盖：  
- OLS、WLS、GLS、GLSAR、分位数回归  
- 混合效应模型  
- 递归与滚动回归  
- 综合诊断（异方差性、自相关、多重共线性）  
- 影响统计量与离群值检测  
- 稳健标准误（HC、HAC、聚类）  
- 假设检验与模型比较  

### references/glm.md  
广义线性模型完整指南：  
- 所有分布族（二项、泊松、伽马等）  
- 连接函数及适用场景  
- 模型拟合与解释  
- 伪R方与拟合优度  
- 诊断与残差分析  
- 应用案例（逻辑、泊松、伽马回归）  

### references/discrete_choice.md  
离散选择模型综合指南：  
- 二分类模型（Logit、Probit）  
- 多分类模型（MNLogit、条件Logit）  
- 计数模型（泊松、负二项、零膨胀、Hurdle）  
- 有序模型  
- 边际效应与解释  
- 模型诊断与比较  

### references/time_series.md  
深度时间序列分析指南：  
- 单变量模型（AR、ARIMA、SARIMAX、指数平滑）  
- 多变量模型（VAR、VARMAX、动态因子）  
- 状态空间模型  
- 平稳性检验与诊断  
- 预测方法与评估  
- 格兰杰因果、脉冲响应、方差分解  

### references/stats_diagnostics.md  
统计检验与诊断综合指南：  
- 残差诊断（自相关、异方差性、正态性）  
- 影响度与离群值检测  
- 假设检验（参数与非参数）  
- 方差分析与事后检验  
- 多重比较校正  
- 稳健协方差矩阵  
- 功效分析与效应量  

**何时查阅：**  
- 需要详细参数说明时  
- 在相似模型间抉择时  
- 排查收敛或诊断问题时  
- 理解特定检验统计量时  
- 查找高级功能代码示例时  

**搜索模式：**  
```bash
# 查找特定模型信息  
grep -r "Quantile Regression" references/  

# 查找诊断检验  
grep -r "Breusch-Pagan" references/stats_diagnostics.md  

# 查找时间序列指导  
grep -r "SARIMAX" references/time_series.md  
```  

## 常见陷阱规避  

1. **遗漏常数项**：除非无需截距，否则始终使用 `sm.add_constant()`  
2. **忽略假设检验**：必须检查残差、异方差性、自相关性  
3. **误用模型类型**：二分类→Logit/Probit，计数→泊松/负二项，勿用OLS  
4. **未检查收敛性**：警惕优化警告  
5. **错误解释系数**：注意连接函数（log、logit等）  
6. **泊松模型处理过度离散**：检查离散度，必要时使用负二项回归  
7. **未采用稳健标准误**：存在异方差或聚类时必需  
8. **过拟合**：参数过多导致样本量不足  
9. **数据泄露**：在测试集上拟合或使用未来信息  
10. **未验证预测**：必须检查样本外性能  
11. **比较非嵌套模型**：使用AIC/BIC而非似然比检验  
12. **忽视强影响点**：检查库克距离与杠杆值  
13. **多重检验**：多假设检验时校正p值  
14. **未差分时间序列**：对非平稳数据直接拟合ARIMA  
15. **混淆预测区间与置信区间**：预测区间范围更宽  

## 获取帮助  

详细文档与示例：  
- 官方文档：https://www.statsmodels.org/stable/  
- 用户指南：https://www.statsmodels.org/stable/user-guide.html  
- 示例库：https://www.statsmodels.org/stable/examples/index.html  
- API参考：https://www.statsmodels.org/stable/api.html
