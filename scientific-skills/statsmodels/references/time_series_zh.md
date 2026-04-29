# 时间序列分析参考指南

本文档提供关于statsmodels中时间序列模型的全面指导，包括ARIMA、状态空间模型、VAR、指数平滑及预测方法。

## 概述

Statsmodels提供广泛的时间序列分析能力：
- **单变量模型**：AR、ARIMA、SARIMAX、指数平滑
- **多变量模型**：VAR、VARMAX、动态因子模型
- **状态空间框架**：自定义模型、卡尔曼滤波
- **诊断工具**：ACF、PACF、平稳性检验、残差分析
- **预测**：点预测与预测区间

## 单变量时间序列模型

### 自回归模型（AutoReg）

自回归模型：当前值依赖于历史值。

**适用场景**：
- 单变量时间序列
- 历史值可预测未来
- 平稳序列

**模型**：yₜ = c + φ₁yₜ₋₁ + φ₂yₜ₋₂ + ... + φₚyₜ₋ₚ + εₜ

```python
from statsmodels.tsa.ar_model import AutoReg
import pandas as pd

# 拟合AR(p)模型
model = AutoReg(y, lags=5)  # AR(5)
results = model.fit()

print(results.summary())
```

**含外生回归变量**：
```python
# 带外生变量的AR模型（ARX）
model = AutoReg(y, lags=5, exog=X_exog)
results = model.fit()
```

**季节性AR模型**：
```python
# 季节性滞后（例如含年度季节性的月度数据）
model = AutoReg(y, lags=12, seasonal=True)
results = model.fit()
```

### ARIMA（自回归综合移动平均）

整合AR、差分（I）和MA组件。

**适用场景**：
- 非平稳时间序列（需差分处理）
- 历史值与误差项可预测未来
- 适用于多种时间序列的灵活模型

**模型**：ARIMA(p,d,q)
- p：AR阶数（滞后项）
- d：差分阶数（实现平稳性）
- q：MA阶数（滞后预测误差）

```python
from statsmodels.tsa.arima.model import ARIMA

# 拟合ARIMA(p,d,q)
model = ARIMA(y, order=(1, 1, 1))  # ARIMA(1,1,1)
results = model.fit()

print(results.summary())
```

**选择p,d,q参数**：

1. **确定d（差分阶数）**：
```python
from statsmodels.tsa.stattools import adfuller

# 平稳性ADF检验
def check_stationarity(series):
    result = adfuller(series)
    print(f"ADF统计量: {result[0]:.4f}")
    print(f"p值: {result[1]:.4f}")
    if result[1] <= 0.05:
        print("序列平稳")
        return True
    else:
        print("序列非平稳，需差分处理")
        return False

# 检验原始序列
if not check_stationarity(y):
    # 一阶差分
    y_diff = y.diff().dropna()
    if not check_stationarity(y_diff):
        # 二阶差分
        y_diff2 = y_diff.diff().dropna()
        check_stationarity(y_diff2)
```

2. **确定p和q（ACF/PACF）**：
```python
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
import matplotlib.pyplot as plt

# 差分后达到平稳状态
fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 8))

# ACF：辅助确定q（MA阶数）
plot_acf(y_stationary, lags=40, ax=ax1)
ax1.set_title('自相关函数 (ACF)')

# PACF：辅助确定p（AR阶数）
plot_pacf(y_stationary, lags=40, ax=ax2)
ax2.set_title('偏自相关函数 (PACF)')

plt.tight_layout()
plt.show()

# 经验法则：
# - PACF在滞后p阶截尾 → AR(p)
# - ACF在滞后q阶截尾 → MA(q)
# - 两者均衰减 → ARMA(p,q)
```

3. **模型选择（AIC/BIC）**：
```python
# 在给定d下网格搜索最优(p,q)
import numpy as np

best_aic = np.inf
best_order = None

for p in range(5):
    for q in range(5):
        try:
            model = ARIMA(y, order=(p, d, q))
            results = model.fit()
            if results.aic < best_aic:
                best_aic = results.aic
                best_order = (p, d, q)
        except:
            continue

print(f"最优阶数: {best_order}，AIC值: {best_aic:.2f}")
```

### SARIMAX（季节性ARIMA含外生变量）

扩展ARIMA模型，支持季节性和外生回归变量。

**适用场景**：
- 季节性模式（月度、季度数据）
- 外部变量影响序列
- 最灵活的单变量模型

**模型**：SARIMAX(p,d,q)(P,D,Q,s)
- (p,d,q)：非季节性ARIMA
- (P,D,Q,s)：周期为s的季节性ARIMA

```python
from statsmodels.tsa.statespace.sarimax import SARIMAX

# 月度数据的季节性ARIMA（s=12）
model = SARIMAX(y,
                order=(1, 1, 1),           # (p,d,q)
                seasonal_order=(1, 1, 1, 12))  # (P,D,Q,s)
results = model.fit()

print(results.summary())
```

**含外生变量**：
```python
# 带外部预测变量的SARIMAX
model = SARIMAX(y,
                exog=X_exog,
                order=(1, 1, 1),
                seasonal_order=(1, 1, 1, 12))
results = model.fit()
```

**示例：含趋势和季节性的月度销售数据**：
```python
# 月度数据典型配置：(p,d,q)(P,D,Q,12)
# 初始尝试(1,1,1)(1,1,1,12)或(0,1,1)(0,1,1,12)

model = SARIMAX(monthly_sales,
                order=(0, 1, 1),
                seasonal_order=(0, 1, 1, 12),
                enforce_stationarity=False,
                enforce_invertibility=False)
results = model.fit()
```

### 指数平滑法

对历史观测值进行指数递减加权的移动平均。

**适用场景**：
- 简单可解释的预测
- 存在趋势和/或季节性
- 无需显式模型设定

**类型**：
- 简单指数平滑：无趋势、无季节性
- Holt方法：含趋势
- Holt-Winters：含趋势和季节性

```python
from statsmodels.tsa.holtwinters import ExponentialSmoothing

# 简单指数平滑
model = ExponentialSmoothing(y, trend=None, seasonal=None)
results = model.fit()

# Holt方法（含趋势）
model = ExponentialSmoothing(y, trend='add', seasonal=None)
results = model.fit()

# Holt-Winters（趋势+季节性）
model = ExponentialSmoothing(y,
                            trend='add',           # 'add'或'mul'
                            seasonal='add',        # 'add'或'mul'
                            seasonal_periods=12)   # 例如月度数据设为12
results = model.fit()

print(results.summary())
```

**加法与乘法模型**：
```python
# 加法模型：季节性波动恒定
# yₜ = 水平项 + 趋势项 + 季节项 + 误差项

# 乘法模型：季节性波动随水平变化
# yₜ = 水平项 × 趋势项 × 季节项 × 误差项

# 选择依据：
# - 加法：季节性波动随时间恒定
# - 乘法：季节性波动随水平增加而增大
```

**创新状态空间（ETS）**：
```python
from statsmodels.tsa.exponential_smoothing.ets import ETSModel

# 更鲁棒的状态空间形式
model = ETSModel(y,
                error='add',           # 'add'或'mul'
                trend='add',           # 'add'、'mul'或None
                seasonal='add',        # 'add'、'mul'或None
                seasonal_periods=12)
results = model.fit()
```

## 多变量时间序列

### VAR（向量自回归）

由多个方程组成的系统，每个变量依赖于所有变量的历史值。

**适用场景**：
- 多个相互关联的时间序列
- 双向因果关系
- 格兰杰因果检验

**模型**：每个变量均为所有变量的自回归：
- y₁ₜ = c₁ + φ₁₁y₁ₜ₋₁ + φ₁₂y₂ₜ₋₁ + ... + ε₁ₜ
- y₂ₜ = c₂ + φ₂₁y₁ₜ₋₁ + φ₂₂y₂ₜ₋₁ + ... + ε₂ₜ

```python
from statsmodels.tsa.api import VAR
import pandas as pd

# 数据应为含多列的DataFrame
# 每列代表一个时间序列
df_multivariate = pd.DataFrame({'series1': y1, 'series2': y2, 'series3': y3})

# 拟合VAR模型
model = VAR(df_multivariate)

# 使用AIC/BIC选择滞后阶数
lag_order_results = model.select_order(maxlags=15)
print(lag_order_results.summary())

# 按最优滞后阶数拟合
results = model.fit(maxlags=5, ic='aic')
print(results.summary())
```

**格兰杰因果检验**：
```python
# 检验series1是否格兰杰引起series2
from statsmodels.tsa.stattools import grangercausalitytests

# 需二维数组[series2, series1]
test_data = df_multivariate[['series2', 'series1']]

# 最大滞后阶数检验
max_lag = 5
results = grangercausalitytests(test_data, max_lag, verbose=True)

# 各滞后阶数p值
for lag in range(1, max_lag + 1):
    p_value = results[lag][0]['ssr_ftest'][1]
    print(f"滞后{lag}阶: p值 = {p_value:.4f}")
```

**脉冲响应函数（IRF）**：
```python
# 追踪冲击在系统中的传导效应
irf = results.irf(10)  # 向前10期

# 绘制脉冲响应
irf.plot(orth=True)  # 正交化（Cholesky分解）
plt.show()

# 累积效应
irf.plot_cum_effects(orth=True)
plt.show()
```

**预测误差方差分解**：
```python
# 各变量对预测误差方差的贡献度
fevd = results.fevd(10)  # 向前10期
fevd.plot()
plt.show()
```

### VARMAX（含移动平均和外生变量的VAR）

扩展VAR模型，增加MA分量和外生变量。

**适用场景**：
- VAR模型不充分（需MA分量）
- 外部变量影响系统
- 更灵活的多变量模型

```python
from statsmodels.tsa.statespace.varmax import VARMAX

# 含外生变量的VARMAX(p,q)模型
model = VARMAX(df_multivariate,
               order=(1, 1),        # (p, q)
               exog=X_exog)
results = model.fit()

print(results.summary())
```

## 状态空间模型

灵活的自定义时间序列建模框架。

**适用场景**：
- 自定义模型设定
- 不可观测成分
- 卡尔曼滤波/平滑
- 缺失数据处理

```python
from statsmodels.tsa.statespace.mlemodel import MLEModel

# 扩展MLEModel实现自定义状态空间模型
# 示例：局部水平模型（随机游走+噪声）
```

**动态因子模型**：
```python
from statsmodels.tsa.statespace.dynamic_factor import DynamicFactor

# 从多变量序列中提取公共因子
model = DynamicFactor(df_multivariate,
                      k_factors=2,          # 因子数量
                      factor_order=2)       # 因子的AR阶数
results = model.fit()

# 估计因子
factors = results.factors.filtered
```

## 预测方法

### 点预测

```python
# ARIMA预测
model = ARIMA(y, order=(1, 1, 1))
results = model.fit()

# 向前预测h步
h = 10
forecast = results.forecast(steps=h)

# 含外生变量（SARIMAX）
model = SARIMAX(y, exog=X, order=(1, 1, 1))
results = model.fit()

# 需提供未来外生变量值
forecast = results.forecast(steps=h, exog=X_future)
```

### 预测区间

```python
# 获取带置信区间的预测
forecast_obj = results.get_forecast(steps=h)
forecast_df = forecast_obj.summary_frame()

print(forecast_df)
# 包含：均值、均值标准误、置信下限、置信上限

# 提取组件
forecast_mean = forecast_df['mean']
forecast_ci_lower = forecast_df['mean_ci_lower']
forecast_ci_upper = forecast_df['mean_ci_upper']

# 可视化
import matplotlib.pyplot as plt

plt.figure(figsize=(12, 6))
plt.plot(y.index, y, label='历史数据')
plt.plot(forecast_df.index, forecast_mean, label='预测值', color='red')
plt.fill_between(forecast_df.index,
                 forecast_ci_lower,
                 forecast_ci_upper,
                 alpha=0.3, color='red', label='95%置信区间')
plt.legend()
plt.title('带预测区间的预测结果')
plt.show()
```

### 动态预测 vs 静态预测

```python
# 静态预测（单步预测，使用实际值）
static_forecast = results.get_prediction(start=split_point, end=len(y)-1)

# 动态预测（多步预测，使用预测值）
dynamic_forecast = results.get_prediction(start=split_point,
                                          end=len(y)-1,
                                          dynamic=True)

# 对比可视化
fig, ax = plt.subplots(figsize=(12, 6))
y.plot(ax=ax, label='实际值')
static_forecast.predicted_mean.plot(ax=ax, label='静态预测')
dynamic_forecast.predicted_mean.plot(ax=ax, label='动态预测')
ax.legend()
plt.show()
```

## 诊断检验

### 平稳性检验

```python
from statsmodels.tsa.stattools import adfuller, kpss

# 增广迪基-富勒（ADF）检验
# H0：存在单位根（非平稳）
adf_result = adfuller(y, autolag='AIC')
print(f"ADF统计量: {adf_result[0]:.4f}")
print(f"p值: {adf_result[1]:.4f}")
if adf_result[1] <= 0.05:
    print("拒绝H0：序列平稳")
else:
    print("未拒绝H0：序列非平稳")

# KPSS检验
# H0：序列平稳（与ADF相反）
kpss_result = kpss(y, regression='c', nlags='auto')
print(f"KPSS统计量: {kpss_result[0]:.4f}")
print(f"p值: {kpss_result[1]:.4f}")
if kpss_result[1] <= 0.05:
    print("拒绝H0：序列非平稳")
else:
    print("未拒绝H0：序列平稳")
```

### 残差诊断

```python
# 残差自相关的Ljung-Box检验
from statsmodels.stats.diagnostic import acorr_ljungbox

lb_test = acorr_ljungbox(results.resid, lags=10, return_df=True)
print(lb_test)
# P值>0.05表示无显著自相关（良好）

# 绘制残差诊断图
results.plot_diagnostics(figsize=(12, 8))
plt.show()

# 包含组件：
# 1. 标准化残差时序图
# 2. 残差直方图+KDE
# 3. 正态Q-Q图
# 4. 残差自相关图（ACF）
```

### 异方差性检验

```python
from statsmodels.stats.diagnostic import het_arch

# ARCH异方差检验
arch_test = het_arch(results.resid, nlags=10)
print(f"ARCH检验统计量: {arch_test[0]:.4f}")
print(f"p值: {arch_test[1]:.4f}")

# 若显著，建议采用GARCH模型
```

## 季节性分解

```python

```python
from statsmodels.tsa.seasonal import seasonal_decompose

# 分解为趋势项、季节项和残差项
decomposition = seasonal_decompose(y,
                                   model='additive',  # 或 'multiplicative'
                                   period=12)         # 季节周期

# 绘制各成分
fig = decomposition.plot()
fig.set_size_inches(12, 8)
plt.show()

# 获取各成分
trend = decomposition.trend
seasonal = decomposition.seasonal
residual = decomposition.resid

# STL分解（鲁棒性更强）
from statsmodels.tsa.seasonal import STL

stl = STL(y, seasonal=13)  # 季节周期必须为奇数
stl_result = stl.fit()

fig = stl_result.plot()
plt.show()
```

## 模型评估

### 样本内指标

```python
# 从结果对象获取
print(f"AIC: {results.aic:.2f}")
print(f"BIC: {results.bic:.2f}")
print(f"对数似然值: {results.llf:.2f}")

# 训练数据的均方误差
from sklearn.metrics import mean_squared_error

mse = mean_squared_error(y, results.fittedvalues)
rmse = np.sqrt(mse)
print(f"RMSE: {rmse:.4f}")

# 平均绝对误差
from sklearn.metrics import mean_absolute_error
mae = mean_absolute_error(y, results.fittedvalues)
print(f"MAE: {mae:.4f}")
```

### 样本外评估

```python
# 时间序列训练测试分割（禁止洗牌！）
train_size = int(0.8 * len(y))
y_train = y[:train_size]
y_test = y[train_size:]

# 在训练数据上拟合
model = ARIMA(y_train, order=(1, 1, 1))
results = model.fit()

# 预测测试期
forecast = results.forecast(steps=len(y_test))

# 评估指标
from sklearn.metrics import mean_squared_error, mean_absolute_error

rmse = np.sqrt(mean_squared_error(y_test, forecast))
mae = mean_absolute_error(y_test, forecast)
mape = np.mean(np.abs((y_test - forecast) / y_test)) * 100

print(f"测试集RMSE: {rmse:.4f}")
print(f"测试集MAE: {mae:.4f}")
print(f"测试集MAPE: {mape:.2f}%")
```

### 滚动预测

```python
# 更真实的评估：滚动一步预测
forecasts = []

for t in range(len(y_test)):
    # 用新观测值重新拟合或更新模型
    y_current = y[:train_size + t]
    model = ARIMA(y_current, order=(1, 1, 1))
    fit = model.fit()

    # 单步预测
    fc = fit.forecast(steps=1)[0]
    forecasts.append(fc)

forecasts = np.array(forecasts)

rmse = np.sqrt(mean_squared_error(y_test, forecasts))
print(f"滚动预测RMSE: {rmse:.4f}")
```

### 交叉验证

```python
# 时间序列交叉验证（扩展窗口）
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
rmse_scores = []

for train_idx, test_idx in tscv.split(y):
    y_train_cv = y.iloc[train_idx]
    y_test_cv = y.iloc[test_idx]

    model = ARIMA(y_train_cv, order=(1, 1, 1))
    results = model.fit()

    forecast = results.forecast(steps=len(test_idx))
    rmse = np.sqrt(mean_squared_error(y_test_cv, forecast))
    rmse_scores.append(rmse)

print(f"交叉验证RMSE: {np.mean(rmse_scores):.4f} ± {np.std(rmse_scores):.4f}")
```

## 高级主题

### ARDL（自回归分布滞后模型）

连接单变量与多变量时间序列分析。

```python
from statsmodels.tsa.ardl import ARDL

# ARDL(p, q)模型
# y取决于自身滞后项和X的滞后项
model = ARDL(y, lags=2, exog=X, exog_lags=2)
results = model.fit()
```

### 误差修正模型

适用于协整序列。

```python
from statsmodels.tsa.vector_ar.vecm import coint_johansen

# 协整检验
johansen_test = coint_johansen(df_multivariate, det_order=0, k_ar_diff=1)

# 若存在协整则拟合VECM
from statsmodels.tsa.vector_ar.vecm import VECM

model = VECM(df_multivariate, k_ar_diff=1, coint_rank=1)
results = model.fit()
```

### 状态转换模型

处理结构突变和状态转换。

```python
from statsmodels.tsa.regime_switching.markov_regression import MarkovRegression

# 马尔可夫转换模型
model = MarkovRegression(y, k_regimes=2, order=1)
results = model.fit()

# 状态平滑概率
regime_probs = results.smoothed_marginal_probabilities
```

## 最佳实践

1. **检验平稳性**：必要时差分，用ADF/KPSS检验验证
2. **绘制数据**：建模前务必可视化
3. **识别季节性**：使用合适季节模型（SARIMAX, Holt-Winters）
4. **模型选择**：结合AIC/BIC和样本外验证
5. **残差诊断**：检查自相关、正态性、异方差性
6. **预测评估**：采用滚动预测和时序交叉验证
7. **避免过拟合**：优先选择简单模型，使用信息准则
8. **记录假设**：注明数据变换（对数化、差分等）
9. **预测区间**：始终提供不确定性估计
10. **定期重拟合**：随新数据更新模型

## 常见陷阱

1. **未检验平稳性**：在非平稳数据上拟合ARIMA
2. **数据泄露**：在变换中使用未来数据
3. **错误季节周期**：季度数据S=4，月度数据S=12
4. **过拟合**：参数过多导致数据过载
5. **忽略残差自相关**：模型不充分
6. **使用不当指标**：MAPE在零值或负值场景失效
7. **未处理缺失值**：影响模型估计
8. **外推外生变量**：SARIMAX需要未来X值
9. **混淆静态与动态预测**：多步预测中动态更真实
10. **未验证预测**：必须检查样本外表现
