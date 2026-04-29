# 时间序列预测

Aeon 提供用于预测未来时间序列值的算法。

## 朴素与基线方法

用于对比的简单预测策略：

- `NaiveForecaster` - 多种策略：末值法、均值法、季节朴素法
  - 参数：`strategy`（"last", "mean", "seasonal"），`sp`（季节周期）
  - **适用场景**：建立基线或处理简单模式时

## 统计模型

经典时间序列预测方法：

### ARIMA
- `ARIMA` - 自回归积分滑动平均模型
  - 参数：`p`（自回归阶数），`d`（差分阶数），`q`（滑动平均阶数）
  - **适用场景**：线性模式、平稳或差分平稳序列

### 指数平滑
- `ETS` - 误差-趋势-季节性分解模型
  - 参数：`error`、`trend`、`seasonal` 类型
  - **适用场景**：存在趋势和季节模式时

### 阈值自回归
- `TAR` - 用于状态转换的阈值自回归模型
- `AutoTAR` - 自动阈值发现
  - **适用场景**：序列在不同状态呈现不同行为时

### Theta 方法
- `Theta` - 经典Theta预测法
  - 参数：分解所需的 `theta` 和 `weights`
  - **适用场景**：需要简单有效的基线时

### 时变参数
- `TVP` - 基于卡尔曼滤波的时变参数模型
  - **适用场景**：参数随时间变化时

## 深度学习预测器

处理复杂时序模式的神经网络：

- `TCNForecaster` - 时序卷积网络
  - 使用扩张卷积获取大感受野
  - **适用场景**：长序列、需要非循环架构时

- `DeepARNetwork` - 基于RNN的概率预测
  - 提供预测区间
  - **适用场景**：需要概率预测和不确定性量化时

## 基于回归的预测

对滞后特征应用回归：

- `RegressionForecaster` - 将回归器封装为预测器
  - 参数：`window_length`、`horizon`
  - **适用场景**：需将任意回归器用作预测器时

## 快速入门

```python
from aeon.forecasting.naive import NaiveForecaster
from aeon.forecasting.arima import ARIMA
import numpy as np

# 创建时间序列
y = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# 朴素基线法
naive = NaiveForecaster(strategy="last")
naive.fit(y)
forecast_naive = naive.predict(fh=[1, 2, 3])

# ARIMA模型
arima = ARIMA(order=(1, 1, 1))
arima.fit(y)
forecast_arima = arima.predict(fh=[1, 2, 3])
```

## 预测时间范围

预测时间范围（`fh`）指定要预测的未来时间点：

```python
# 相对范围（后续3个时间点）
fh = [1, 2, 3]

# 绝对范围（特定时间索引）
from aeon.forecasting.base import ForecastingHorizon
fh = ForecastingHorizon([11, 12, 13], is_relative=False)
```

## 模型选择指南

- **基线**：采用季节策略的NaiveForecaster
- **线性模式**：ARIMA
- **趋势+季节性**：ETS
- **状态转换**：TAR, AutoTAR
- **复杂模式**：TCNForecaster
- **概率预测**：DeepARNetwork
- **长序列**：TCNForecaster
- **短序列**：ARIMA, ETS

## 评估指标

使用标准预测指标：

```python
from aeon.performance_metrics.forecasting import (
    mean_absolute_error,
    mean_squared_error,
    mean_absolute_percentage_error
)

# 计算误差
mae = mean_absolute_error(y_true, y_pred)
mse = mean_squared_error(y_true, y_pred)
mape = mean_absolute_percentage_error(y_true, y_pred)
```

## 外生变量

多数预测器支持外部特征：

```python
# 使用外生变量训练
forecaster.fit(y, X=X_train)

# 预测需提供未来外生变量值
y_pred = forecaster.predict(fh=[1, 2, 3], X=X_test)
```

## 基础类

- `BaseForecaster` - 所有预测器的抽象基类
- `BaseDeepForecaster` - 深度学习预测器基类

可通过扩展这些类实现自定义预测算法。
