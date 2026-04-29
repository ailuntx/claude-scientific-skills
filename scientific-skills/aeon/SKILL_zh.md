---
name: aeon
description: 该技能应用于时间序列机器学习任务，包括分类、回归、聚类、预测、异常检测、分割和相似性搜索。适用于处理时序数据、序列模式或需要超越标准机器学习方法的专用算法的时间索引观测。特别适合通过scikit-learn兼容API进行单变量和多变量时间序列分析。
license: BSD-3-Clause许可证
metadata:
    skill-author: K-Dense Inc.
---

# Aeon时间序列机器学习

## 概述

Aeon是一个兼容scikit-learn的Python时间序列机器学习工具包，提供最先进的分类、回归、聚类、预测、异常检测、分割和相似性搜索算法。

## 使用场景

在以下场景应用此技能：
- 对时间序列数据进行分类或预测
- 检测时序序列中的异常点或变化点
- 聚类相似的时间序列模式
- 预测未来值
- 发现重复模式（motifs）或异常子序列（discords）
- 使用专用距离度量比较时间序列
- 从时序数据中提取特征

## 安装

```bash
uv pip install aeon
```

## 核心功能

### 1. 时间序列分类

将时间序列分类到预定义类别。完整算法目录见`references/classification.md`。

**快速入门：**
```python
from aeon.classification.convolution_based import RocketClassifier
from aeon.datasets import load_classification

# 加载数据
X_train, y_train = load_classification("GunPoint", split="train")
X_test, y_test = load_classification("GunPoint", split="test")

# 训练分类器
clf = RocketClassifier(n_kernels=10000)
clf.fit(X_train, y_train)
accuracy = clf.score(X_test, y_test)
```

**算法选择：**
- **速度+性能**：`MiniRocketClassifier`, `Arsenal`
- **最高精度**：`HIVECOTEV2`, `InceptionTimeClassifier`
- **可解释性**：`ShapeletTransformClassifier`, `Catch22Classifier`
- **小数据集**：带DTW距离的`KNeighborsTimeSeriesClassifier`

### 2. 时间序列回归

从时间序列预测连续值。算法见`references/regression.md`。

**快速入门：**
```python
from aeon.regression.convolution_based import RocketRegressor
from aeon.datasets import load_regression

X_train, y_train = load_regression("Covid3Month", split="train")
X_test, y_test = load_regression("Covid3Month", split="test")

reg = RocketRegressor()
reg.fit(X_train, y_train)
predictions = reg.predict(X_test)
```

### 3. 时间序列聚类

无标签分组相似时间序列。方法见`references/clustering.md`。

**快速入门：**
```python
from aeon.clustering import TimeSeriesKMeans

clusterer = TimeSeriesKMeans(
    n_clusters=3,
    distance="dtw",
    averaging_method="ba"
)
labels = clusterer.fit_predict(X_train)
centers = clusterer.cluster_centers_
```

### 4. 时间序列预测

预测未来时间序列值。预测器见`references/forecasting.md`。

**快速入门：**
```python
from aeon.forecasting.arima import ARIMA

forecaster = ARIMA(order=(1, 1, 1))
forecaster.fit(y_train)
y_pred = forecaster.predict(fh=[1, 2, 3, 4, 5])
```

### 5. 异常检测

识别异常模式或离群点。检测器见`references/anomaly_detection.md`。

**快速入门：**
```python
from aeon.anomaly_detection import STOMP

detector = STOMP(window_size=50)
anomaly_scores = detector.fit_predict(y)

# 高分值表示异常
threshold = np.percentile(anomaly_scores, 95)
anomalies = anomaly_scores > threshold
```

### 6. 序列分割

将时间序列划分为带变化点的区域。见`references/segmentation.md`。

**快速入门：**
```python
from aeon.segmentation import ClaSPSegmenter

segmenter = ClaSPSegmenter()
change_points = segmenter.fit_predict(y)
```

### 7. 相似性搜索

在时间序列内或跨序列查找相似模式。见`references/similarity_search.md`。

**快速入门：**
```python
from aeon.similarity_search import StompMotif

# 查找重复模式
motif_finder = StompMotif(window_size=50, k=3)
motifs = motif_finder.fit_predict(y)
```

## 特征提取与转换

用于特征工程的时间序列转换。见`references/transformations.md`。

**ROCKET特征：**
```python
from aeon.transformations.collection.convolution_based import RocketTransformer

rocket = RocketTransformer()
X_features = rocket.fit_transform(X_train)

# 特征可用于任意sklearn分类器
from sklearn.ensemble import RandomForestClassifier
clf = RandomForestClassifier()
clf.fit(X_features, y_train)
```

**统计特征：**
```python
from aeon.transformations.collection.feature_based import Catch22

catch22 = Catch22()
X_features = catch22.fit_transform(X_train)
```

**预处理：**
```python
from aeon.transformations.collection import MinMaxScaler, Normalizer

scaler = Normalizer()  # Z标准化
X_normalized = scaler.fit_transform(X_train)
```

## 距离度量

专用时序距离测量方法。完整目录见`references/distances.md`。

**用法：**
```python
from aeon.distances import dtw_distance, dtw_pairwise_distance

# 单序列距离
distance = dtw_distance(x, y, window=0.1)

# 成对距离
distance_matrix = dtw_pairwise_distance(X_train)

# 用于分类器
from aeon.classification.distance_based import KNeighborsTimeSeriesClassifier

clf = KNeighborsTimeSeriesClassifier(
    n_neighbors=5,
    distance="dtw",
    distance_params={"window": 0.2}
)
```

**可用距离：**
- **弹性距离**：DTW, DDTW, WDTW, ERP, EDR, LCSS, TWE, MSM
- **锁步距离**：欧氏距离、曼哈顿距离、闵可夫斯基距离
- **形状距离**：Shape DTW, SBD

## 深度学习网络

时间序列神经网络架构。见`references/networks.md`。

**架构：**
- 卷积网络：`FCNClassifier`, `ResNetClassifier`, `InceptionTimeClassifier`
- 循环网络：`RecurrentNetwork`, `TCNNetwork`
- 自编码器：`AEFCNClusterer`, `AEResNetClusterer`

**用法：**
```python
from aeon.classification.deep_learning import InceptionTimeClassifier

clf = InceptionTimeClassifier(n_epochs=100, batch_size=32)
clf.fit(X_train, y_train)
predictions = clf.predict(X_test)
```

## 数据集与基准测试

加载标准基准并评估性能。见`references/datasets_benchmarking.md`。

**加载数据集：**
```python
from aeon.datasets import load_classification, load_regression

# 分类任务
X_train, y_train = load_classification("ArrowHead", split="train")

# 回归任务
X_train, y_train = load_regression("Covid3Month", split="train")
```

**基准测试：**
```python
from aeon.benchmarking import get_estimator_results

# 与已发布结果对比
published = get_estimator_results("ROCKET", "GunPoint")
```

## 常用工作流

### 分类流程

```python
from aeon.transformations.collection import Normalizer
from aeon.classification.convolution_based import RocketClassifier
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ('normalize', Normalizer()),
    ('classify', RocketClassifier())
])

pipeline.fit(X_train, y_train)
accuracy = pipeline.score(X_test, y_test)
```

### 特征提取+传统机器学习

```python
from aeon.transformations.collection import RocketTransformer
from sklearn.ensemble import GradientBoostingClassifier

# 提取特征
rocket = RocketTransformer()
X_train_features = rocket.fit_transform(X_train)
X_test_features = rocket.transform(X_test)

# 训练传统ML模型
clf = GradientBoostingClassifier()
clf.fit(X_train_features, y_train)
predictions = clf.predict(X_test_features)
```

### 带可视化的异常检测

```python
from aeon.anomaly_detection import STOMP
import matplotlib.pyplot as plt

detector = STOMP(window_size=50)
scores = detector.fit_predict(y)

plt.figure(figsize=(15, 5))
plt.subplot(2, 1, 1)
plt.plot(y, label='时间序列')
plt.subplot(2, 1, 2)
plt.plot(scores, label='异常分值', color='red')
plt.axhline(np.percentile(scores, 95), color='k', linestyle='--')
plt.show()
```

## 最佳实践

### 数据准备

1. **标准化**：多数算法受益于Z标准化
   ```python
   from aeon.transformations.collection import Normalizer
   normalizer = Normalizer()
   X_train = normalizer.fit_transform(X_train)
   X_test = normalizer.transform(X_test)
   ```

2. **处理缺失值**：分析前进行填补
   ```python
   from aeon.transformations.collection import SimpleImputer
   imputer = SimpleImputer(strategy='mean')
   X_train = imputer.fit_transform(X_train)
   ```

3. **检查数据格式**：Aeon要求形状为`(n_samples, n_channels, n_timepoints)`

### 模型选择

1. **从简开始**：先尝试ROCKET变体再转向深度学习
2. **使用验证集**：划分训练数据进行超参数调优
3. **对比基线**：测试简单方法（1-NN欧氏距离、朴素方法）
4. **考虑资源**：追求速度用ROCKET，有GPU可用深度学习

### 算法选择指南

**快速原型设计：**
- 分类：`MiniRocketClassifier`
- 回归：`MiniRocketRegressor`
- 聚类：带欧氏距离的`TimeSeriesKMeans`

**追求最高精度：**
- 分类：`HIVECOTEV2`, `InceptionTimeClassifier`
- 回归：`InceptionTimeRegressor`
- 预测：`ARIMA`, `TCNForecaster`

**追求可解释性：**
- 分类：`ShapeletTransformClassifier`, `Catch22Classifier`
- 特征：`Catch22`, `TSFresh`

**小数据集场景：**
- 基于距离：带DTW的`KNeighborsTimeSeriesClassifier`
- 避免：深度学习（需大数据量）

## 参考文档

详细文档见`references/`目录：
- `classification.md` - 所有分类算法
- `regression.md` - 回归方法
- `clustering.md` - 聚类算法
- `forecasting.md` - 预测方法
- `anomaly_detection.md` - 异常检测方法
- `segmentation.md` - 分割算法
- `similarity_search.md` - 模式匹配与motif发现
- `transformations.md` - 特征提取与预处理
- `distances.md` - 时间序列距离度量
- `networks.md` - 深度学习架构
- `datasets_benchmarking.md` - 数据加载与评估工具

## 附加资源

- 文档：https://www.aeon-toolkit.org/
- GitHub：https://github.com/aeon-toolkit/aeon
- 示例：https://www.aeon-toolkit.org/en/stable/examples.html
- API参考：https://www.aeon-toolkit.org/en/stable/api_reference.html
