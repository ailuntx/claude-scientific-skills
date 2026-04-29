# 时间序列回归

Aeon 提供了 9 个类别的时间序列回归器，用于根据时间序列预测连续值。

## 基于卷积的回归器

应用卷积核进行特征提取：

- `HydraRegressor` - 多分辨率空洞卷积
- `RocketRegressor` - 随机卷积核
- `MiniRocketRegressor` - 简化版 ROCKET 加速运算
- `MultiRocketRegressor` - 组合型 ROCKET 变体
- `MultiRocketHydraRegressor` - 融合 ROCKET 和 Hydra 方法

**适用场景**：需要快速回归且具备强基线性能时。

## 深度学习回归器

端到端时序回归的神经网络架构：

- `FCNRegressor` - 全卷积网络
- `ResNetRegressor` - 带跳跃连接的残差块
- `InceptionTimeRegressor` - 多尺度初始模块
- `TimeCNNRegressor` - 标准 CNN 架构
- `RecurrentRegressor` - RNN/LSTM/GRU 变体
- `MLPRegressor` - 多层感知机
- `EncoderRegressor` - 通用编码器封装
- `LITERegressor` - 轻量级初始时间集成
- `DisjointCNNRegressor` - 专用 CNN 架构

**适用场景**：大型数据集、复杂模式或需要特征学习时。

## 基于距离的回归器

采用时序距离度量的 k 近邻算法：

- `KNeighborsTimeSeriesRegressor` - 支持 DTW、LCSS、ERP 等距离度量的 k-NN

**适用场景**：小型数据集、局部相似性模式或需要可解释预测时。

## 基于特征的回归器

回归前提取统计特征：

- `Catch22Regressor` - 22 项时序特征规范集
- `FreshPRINCERegressor` - 多特征提取器组合流程
- `SummaryRegressor` - 统计摘要特征
- `TSFreshRegressor` - 自动化 tsfresh 特征提取

**适用场景**：需要可解释特征或领域特定特征工程时。

## 混合回归器

融合多种方法：

- `RISTRegressor` - 随机区间-shapelet 变换

**适用场景**：需结合区间和 shapelet 方法优势时。

## 基于区间的回归器

从时间区间提取特征：

- `CanonicalIntervalForestRegressor` - 决策树随机区间
- `DrCIFRegressor` - 多样化表征 CIF
- `TimeSeriesForestRegressor` - 随机区间集成
- `RandomIntervalRegressor` - 基础区间方法
- `RandomIntervalSpectralEnsembleRegressor` - 频谱区间特征
- `QUANTRegressor` - 基于分位数的区间特征

**适用场景**：预测模式出现在特定时间窗口时。

## 基于 Shapelet 的回归器

使用判别子序列进行预测：

- `RDSTRegressor` - 随机扩张 Shapelet 变换

**适用场景**：需要相位不变的判别模式时。

## 组合工具

构建自定义回归流程：

- `RegressorPipeline` - 转换器与回归器链式组合
- `RegressorEnsemble` - 带可学习权重的加权集成
- `SklearnRegressorWrapper` - 适配 sklearn 回归器用于时序

## 实用工具

- `DummyRegressor` - 基线策略（均值/中位数）
- `BaseRegressor` - 自定义回归器抽象基类
- `BaseDeepRegressor` - 深度学习回归器基类

## 快速入门

```python
from aeon.regression.convolution_based import RocketRegressor
from aeon.datasets import load_regression

# 加载数据
X_train, y_train = load_regression("Covid3Month", split="train")
X_test, y_test = load_regression("Covid3Month", split="test")

# 训练与预测
reg = RocketRegressor()
reg.fit(X_train, y_train)
predictions = reg.predict(X_test)
```

## 算法选择指南

- **速度优先**：MiniRocketRegressor
- **精度优先**：InceptionTimeRegressor, MultiRocketHydraRegressor
- **可解释性**：Catch22Regressor, SummaryRegressor
- **小数据集**：KNeighborsTimeSeriesRegressor
- **大数据集**：深度学习回归器, ROCKET 变体
- **区间模式**：DrCIFRegressor, CanonicalIntervalForestRegressor
