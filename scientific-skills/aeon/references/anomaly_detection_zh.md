# 异常检测

Aeon 提供了在序列级别和集合级别识别时间序列异常模式的异常检测方法。

## 集合异常检测器

检测集合中的异常时间序列：

- `ClassificationAdapter` - 适配分类器用于异常检测
  - 在正常数据上训练，预测时标记异常值
  - **适用场景**：拥有标记的正常数据，需要基于分类的方法

- `OutlierDetectionAdapter` - 封装 sklearn 异常检测器
  - 兼容 IsolationForest、LOF、OneClassSVM
  - **适用场景**：需在集合上使用 sklearn 异常检测器

## 序列异常检测器

检测单个时间序列中的异常点或子序列。

### 基于距离的方法

使用相似性度量识别异常：

- `CBLOF` - 基于聚类的局部离群因子
  - 聚类数据，根据聚类属性识别异常
  - **适用场景**：异常形成稀疏聚类

- `KMeansAD` - 基于 K 均值的异常检测
  - 到最近聚类中心的距离指示异常
  - **适用场景**：正常模式聚类良好

- `LeftSTAMPi` - 增量式左 STAMP
  - 矩阵剖面用于在线异常检测
  - **适用场景**：流式数据，需要在线检测

- `STOMP` - 可扩展时序有序搜索矩阵剖面
  - 计算子序列异常的矩阵剖面
  - **适用场景**：不一致性发现，模式检测

- `MERLIN` - 基于矩阵剖面的方法
  - 高效矩阵剖面计算
  - **适用场景**：大型时间序列，需要可扩展性

- `LOF` - 适配时间序列的局部离群因子
  - 基于密度的异常检测
  - **适用场景**：低密度区域的异常

- `ROCKAD` - 基于 ROCKET 的半监督检测
  - 使用 ROCKET 特征识别异常
  - **适用场景**：拥有部分标记数据，需要基于特征的方法

### 基于分布的方法

分析统计分布：

- `COPOD` - 基于 Copula 的异常检测
  - 建模边缘分布和联合分布
  - **适用场景**：多维时间序列，复杂依赖关系

- `DWT_MLEAD` - 离散小波变换多级异常检测
  - 将序列分解为频带
  - **适用场景**：特定频率的异常

### 基于隔离的方法

使用隔离原理：

- `IsolationForest` - 基于随机森林的隔离
  - 异常点比正常点更易被隔离
  - **适用场景**：高维数据，无分布假设

- `OneClassSVM` - 用于新颖性检测的支持向量机
  - 学习正常数据的边界
  - **适用场景**：正常区域定义明确，需要鲁棒边界

- `STRAY` - 流式鲁棒异常检测
  - 对数据分布变化具有鲁棒性
  - **适用场景**：流式数据，存在分布漂移

### 外部库集成

- `PyODAdapter` - 桥接 PyOD 库至 aeon
  - 支持 40+ PyOD 异常检测器
  - **适用场景**：需要特定 PyOD 算法

## 快速入门

```python
from aeon.anomaly_detection import STOMP
import numpy as np

# 创建含异常的时间序列
y = np.concatenate([
    np.sin(np.linspace(0, 10, 100)),
    [5.0],  # 异常尖峰
    np.sin(np.linspace(10, 20, 100))
])

# 检测异常
detector = STOMP(window_size=10)
anomaly_scores = detector.fit_predict(y)

# 高分值表示更异常的点
threshold = np.percentile(anomaly_scores, 95)
anomalies = anomaly_scores > threshold
```

## 点异常与子序列异常

- **点异常**：单个异常值
  - 推荐方法：COPOD, DWT_MLEAD, IsolationForest

- **子序列异常**（不一致性）：异常模式
  - 推荐方法：STOMP, LeftSTAMPi, MERLIN

- **集体异常**：形成异常模式的点群
  - 推荐方法：矩阵剖面方法，基于聚类的方法

## 评估指标

专用异常检测评估指标：

```python
from aeon.benchmarking.metrics.anomaly_detection import (
    range_precision,
    range_recall,
    range_f_score,
    roc_auc_score
)

# 基于范围的指标考虑窗口检测
precision = range_precision(y_true, y_pred, alpha=0.5)
recall = range_recall(y_true, y_pred, alpha=0.5)
f1 = range_f_score(y_true, y_pred, alpha=0.5)
```

## 算法选择指南

- **速度优先**：KMeansAD, IsolationForest
- **精度优先**：STOMP, COPOD
- **流式数据**：LeftSTAMPi, STRAY
- **不一致性发现**：STOMP, MERLIN
- **多维数据**：COPOD, PyODAdapter
- **半监督场景**：ROCKAD, OneClassSVM
- **无训练数据**：IsolationForest, STOMP

## 最佳实践

1. **数据归一化**：多数方法对尺度敏感
2. **选择窗口大小**：矩阵剖面方法中窗口大小至关重要
3. **设置阈值**：使用基于百分位或领域特定阈值
4. **验证结果**：可视化检测结果确保有效性
5. **处理季节性**：检测前进行去趋势/去季节化处理
