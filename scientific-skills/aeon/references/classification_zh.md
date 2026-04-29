# 时间序列分类

Aeon 提供 13 类兼容 scikit-learn API 的时间序列分类器。

## 基于卷积的分类器

应用随机卷积变换实现高效特征提取：

- `Arsenal` - 使用多种核函数的 ROCKET 分类器集成
- `HydraClassifier` - 带膨胀操作的多分辨率卷积
- `RocketClassifier` - 随机卷积核配合岭回归
- `MiniRocketClassifier` - 速度优化的简化版 ROCKET
- `MultiRocketClassifier` - 融合多种 ROCKET 变体

**适用场景**：需要跨数据集快速、可扩展且性能强劲的分类。

## 深度学习分类器

针对时序序列优化的神经网络架构：

- `FCNClassifier` - 全卷积网络
- `ResNetClassifier` - 带跳跃连接的残差网络
- `InceptionTimeClassifier` - 多尺度初始模块
- `TimeCNNClassifier` - 标准时间序列 CNN
- `MLPClassifier` - 多层感知机基线模型
- `EncoderClassifier` - 通用编码器封装
- `DisjointCNNClassifier` - 专注 shapelet 的架构

**适用场景**：拥有大型数据集、需要端到端学习或处理复杂时序模式。

## 基于字典的分类器

将时间序列转换为符号化表示：

- `BOSSEnsemble` - 基于 SFA 符号的词袋模型集成投票
- `TemporalDictionaryEnsemble` - 多字典方法组合
- `WEASEL` - 时间序列分类词提取算法
- `MrSEQLClassifier` - 多符号序列学习

**适用场景**：需要可解释模型、稀疏模式或符号推理。

## 基于距离的分类器

利用专业时序距离度量：

- `KNeighborsTimeSeriesClassifier` - 使用时序距离的 k 近邻 (DTW, LCSS, ERP 等)
- `ElasticEnsemble` - 多弹性距离度量组合
- `ProximityForest` - 基于距离划分的树集成

**适用场景**：小型数据集、基于相似性的分类或需要可解释决策。

## 基于特征提取的分类器

先提取统计特征再进行分类：

- `Catch22Classifier` - 22 个核心时序特征
- `TSFreshClassifier` - 通过 tsfresh 自动特征提取
- `SignatureClassifier` - 路径签名变换
- `SummaryClassifier` - 统计摘要提取
- `FreshPRINCEClassifier` - 多特征提取器组合

**适用场景**：需要可解释特征、具备领域知识或采用特征工程方法。

## 基于区间的分类器

从随机或监督区间提取特征：

- `CanonicalIntervalForestClassifier` - 随机区间特征配合决策树
- `DrCIFClassifier` - 融合 catch22 特征的多表征 CIF
- `TimeSeriesForestClassifier` - 基于统计摘要的随机区间
- `RandomIntervalClassifier` - 简单区间方法
- `RandomIntervalSpectralEnsembleClassifier` - 区间谱特征
- `SupervisedTimeSeriesForest` - 监督式区间选择

**适用场景**：判别性模式出现在特定时间窗口。

## 基于 Shapelet 的分类器

识别判别性子序列 (shapelet)：

- `ShapeletTransformClassifier` - 发现并利用判别性 shapelet
- `LearningShapeletClassifier` - 通过梯度下降学习 shapelet
- `SASTClassifier` - 可扩展近似 shapelet 变换
- `RDSTClassifier` - 随机膨胀 shapelet 变换

**适用场景**：需要可解释的判别模式或相位不变特征。

## 混合分类器

融合多种分类范式：

- `HIVECOTEV1` - 基于变换集成的分层投票组合 (版本1)
- `HIVECOTEV2` - 组件升级的增强版本

**适用场景**：追求最高精度且具备计算资源。

## 早期分类

在观测完整序列前进行预测：

- `TEASER` - 双层早期精准序列分类器
- `ProbabilityThresholdEarlyClassifier` - 置信度超阈值时预测

**适用场景**：需要实时决策或观测成本较高时。

## 序数分类

处理有序类别标签：

- `OrdinalTDE` - 面向序数输出的时序字典集成

**适用场景**：类别存在自然顺序 (如严重等级)。

## 组合工具

构建自定义流水线和集成模型：

- `ClassifierPipeline` - 串联转换器与分类器
- `WeightedEnsembleClassifier` - 加权组合分类器
- `SklearnClassifierWrapper` - 适配 sklearn 分类器处理时序数据

## 快速入门

```python
from aeon.classification.convolution_based import RocketClassifier
from aeon.datasets import load_classification

# 加载数据
X_train, y_train = load_classification("GunPoint", split="train")
X_test, y_test = load_classification("GunPoint", split="test")

# 训练与预测
clf = RocketClassifier()
clf.fit(X_train, y_train)
accuracy = clf.score(X_test, y_test)
```

## 算法选择指南

- **速度优先**：MiniRocketClassifier, Arsenal
- **精度优先**：HIVECOTEV2, InceptionTimeClassifier
- **可解释性**：ShapeletTransformClassifier, Catch22Classifier
- **小数据**：KNeighborsTimeSeriesClassifier, 基于距离的方法
- **大数据**：深度学习分类器, ROCKET 变体
