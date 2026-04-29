# 时间序列聚类

Aeon 提供了专为时序数据优化的聚类算法，包含定制化的距离度量和中心点计算方法。

## 分区算法

适用于时间序列的标准 k-means/k-medoids 变体：

- `TimeSeriesKMeans` - 支持时序距离度量（DTW、欧氏距离等）的 K-means
- `TimeSeriesKMedoids` - 使用实际时间序列作为聚类中心
- `TimeSeriesKShape` - 基于形态的聚类算法
- `TimeSeriesKernelKMeans` - 针对非线性模式的核函数变体

**适用场景**：已知聚类数量，预期为球形聚类分布。

## 大规模数据集方法

针对海量集合的高效聚类：

- `TimeSeriesCLARA` - 基于采样的应用级大规模聚类
- `TimeSeriesCLARANS` - CLARA 的随机搜索变体

**适用场景**：数据集过大无法使用标准 k-medoids，需要可扩展性。

## 弹性距离聚类

专为基于对齐的相似性设计：

- `KASBA` - 具有平移不变弹性平均的 K-means
- `ElasticSOM` - 使用弹性距离的自组织映射

**适用场景**：时间序列存在时移或扭曲。

## 谱方法

基于图论的聚类：

- `KSpectralCentroid` - 带质心计算的谱聚类

**适用场景**：非凸聚类形态，需基于图论的方法。

## 深度学习聚类

基于自编码器的神经网络聚类：

- `AEFCNClusterer` - 全卷积自编码器
- `AEResNetClusterer` - 残差网络自编码器
- `AEDCNNClusterer` - 扩张卷积神经网络自编码器
- `AEDRNNClusterer` - 扩张循环神经网络自编码器
- `AEBiGRUClusterer` - 双向门控循环单元自编码器
- `AEAttentionBiGRUClusterer` - 注意力增强型 BiGRU 自编码器

**适用场景**：大规模数据集，需学习表征或处理复杂模式。

## 特征工程聚类

转换至特征空间后再聚类：

- `Catch22Clusterer` - 基于 22 个标准特征聚类
- `SummaryClusterer` - 使用统计摘要特征
- `TSFreshClusterer` - 自动化 tsfresh 特征工程

**适用场景**：原始时间序列信息不足，需可解释特征。

## 组合构建

创建定制化聚类流程：

- `ClustererPipeline` - 串联转换器与聚类器

## 中心点计算方法

计算时间序列聚类中心：

- `mean_average` - 算术平均
- `ba_average` - 基于 DTW 的重心平均
- `kasba_average` - 平移不变平均
- `shift_invariant_average` - 通用平移不变方法

**适用场景**：需代表性聚类中心用于可视化或初始化。

## 快速入门

```python
from aeon.clustering import TimeSeriesKMeans
from aeon.datasets import load_classification

# 加载数据（使用分类数据进行聚类）
X_train, _ = load_classification("GunPoint", split="train")

# 执行时间序列聚类
clusterer = TimeSeriesKMeans(
    n_clusters=3,
    distance="dtw",  # 使用 DTW 距离
    averaging_method="ba"  # 重心平均法
)
labels = clusterer.fit_predict(X_train)
centers = clusterer.cluster_centers_
```

## 算法选择指南

- **速度优先**：采用欧氏距离的 TimeSeriesKMeans
- **时序对齐**：KASBA 或支持 DTW 的 TimeSeriesKMeans
- **大规模数据**：TimeSeriesCLARA 或 TimeSeriesCLARANS
- **复杂模式**：深度学习聚类器
- **可解释性**：Catch22Clusterer 或 SummaryClusterer
- **非凸聚类**：KSpectralCentroid

## 距离度量

兼容的距离度量包括：
- 欧氏距离、曼哈顿距离、闵可夫斯基距离（步长锁定）
- DTW、DDTW、WDTW（带对齐的弹性距离）
- ERP、EDR、LCSS（基于编辑操作）
- MSM、TWE（专用弹性距离）

## 效果评估

使用 sklearn 或 aeon 的聚类评估指标：
- 轮廓系数
- Davies-Bouldin 指数
- Calinski-Harabasz 指数
