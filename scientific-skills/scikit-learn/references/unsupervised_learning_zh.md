# 无监督学习参考指南

## 概述

无监督学习通过聚类、降维和密度估计在未标记数据中发现模式。

## 聚类

### K均值算法

**K均值 (`sklearn.cluster.KMeans`)**
- 基于划分的聚类方法，形成K个簇
- 关键参数：
  - `n_clusters`: 需要形成的簇数量
  - `init`: 初始化方法 ('k-means++', 'random')
  - `n_init`: 初始化次数 (默认=10)
  - `max_iter`: 最大迭代次数
- 适用场景：已知簇数量，簇呈球形分布
- 快速且可扩展
- 示例：
```python
from sklearn.cluster import KMeans

model = KMeans(n_clusters=3, init='k-means++', n_init=10, random_state=42)
labels = model.fit_predict(X)
centers = model.cluster_centers_

# 惯性（样本到最近中心的平方距离之和）
print(f"Inertia: {model.inertia_}")
```

**小批量K均值**
- 使用小批量加速的K均值算法
- 适用场景：大型数据集，需要快速训练
- 精度略低于标准K均值
- 示例：
```python
from sklearn.cluster import MiniBatchKMeans

model = MiniBatchKMeans(n_clusters=3, batch_size=100, random_state=42)
labels = model.fit_predict(X)
```

### 基于密度的聚类

**DBSCAN (`sklearn.cluster.DBSCAN`)**
- 基于密度的空间聚类
- 关键参数：
  - `eps`: 两个样本成为邻居的最大距离
  - `min_samples`: 形成核心点的最小邻域样本数
  - `metric`: 距离度量标准
- 适用场景：任意形状簇，存在噪声/离群点
- 自动确定簇数量
- 将噪声点标记为-1
- 示例：
```python
from sklearn.cluster import DBSCAN

model = DBSCAN(eps=0.5, min_samples=5, metric='euclidean')
labels = model.fit_predict(X)

# 簇数量（不含噪声）
n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
n_noise = list(labels).count(-1)
print(f"Clusters: {n_clusters}, Noise points: {n_noise}")
```

**HDBSCAN (`sklearn.cluster.HDBSCAN`)**
- 具有自适应epsilon的分层DBSCAN
- 比DBSCAN更鲁棒
- 关键参数：`min_cluster_size`
- 适用场景：密度变化的簇
- 示例：
```python
from sklearn.cluster import HDBSCAN

model = HDBSCAN(min_cluster_size=10, min_samples=5)
labels = model.fit_predict(X)
```

**OPTICS (`sklearn.cluster.OPTICS`)**
- 通过排序点识别聚类结构
- 类似DBSCAN但无需eps参数
- 关键参数：`min_samples`, `max_eps`
- 适用场景：密度变化，探索性分析
- 示例：
```python
from sklearn.cluster import OPTICS

model = OPTICS(min_samples=5, max_eps=0.5)
labels = model.fit_predict(X)
```

### 分层聚类

**凝聚聚类**
- 自底向上的分层聚类
- 关键参数：
  - `n_clusters`: 簇数量（或使用`distance_threshold`）
  - `linkage`: 连接方式 ('ward', 'complete', 'average', 'single')
  - `metric`: 距离度量标准
- 适用场景：需要树状图，层级结构重要
- 示例：
```python
from sklearn.cluster import AgglomerativeClustering

model = AgglomerativeClustering(n_clusters=3, linkage='ward')
labels = model.fit_predict(X)

# 使用scipy创建树状图
from scipy.cluster.hierarchy import dendrogram, linkage
Z = linkage(X, method='ward')
dendrogram(Z)
```

### 其他聚类方法

**均值漂移**
- 通过向密度模式移动点来发现簇
- 自动确定簇数量
- 关键参数：`bandwidth`
- 适用场景：未知簇数量，任意形状
- 示例：
```python
from sklearn.cluster import MeanShift, estimate_bandwidth

# 估计带宽
bandwidth = estimate_bandwidth(X, quantile=0.2, n_samples=500)
model = MeanShift(bandwidth=bandwidth)
labels = model.fit_predict(X)
```

**谱聚类**
- 基于特征值的图方法
- 关键参数：`n_clusters`, `affinity` ('rbf', 'nearest_neighbors')
- 适用场景：非凸簇，图结构
- 示例：
```python
from sklearn.cluster import SpectralClustering

model = SpectralClustering(n_clusters=3, affinity='rbf', random_state=42)
labels = model.fit_predict(X)
```

**亲和传播**
- 通过消息传递寻找代表点
- 自动确定簇数量
- 关键参数：`damping`, `preference`
- 适用场景：未知簇数量
- 示例：
```python
from sklearn.cluster import AffinityPropagation

model = AffinityPropagation(damping=0.9, random_state=42)
labels = model.fit_predict(X)
n_clusters = len(model.cluster_centers_indices_)
```

**BIRCH**
- 使用层次结构的平衡迭代规约聚类
- 内存高效，适用于大型数据集
- 关键参数：`n_clusters`, `threshold`, `branching_factor`
- 适用场景：超大规模数据集
- 示例：
```python
from sklearn.cluster import Birch

model = Birch(n_clusters=3, threshold=0.5)
labels = model.fit_predict(X)
```

### 聚类评估

**已知真实标签的指标：**
```python
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score
from sklearn.metrics import adjusted_mutual_info_score, fowlkes_mallows_score

# 比较预测标签与真实标签
ari = adjusted_rand_score(y_true, y_pred)
nmi = normalized_mutual_info_score(y_true, y_pred)
ami = adjusted_mutual_info_score(y_true, y_pred)
fmi = fowlkes_mallows_score(y_true, y_pred)
```

**无真实标签的指标：**
```python
from sklearn.metrics import silhouette_score, calinski_harabasz_score
from sklearn.metrics import davies_bouldin_score

# 轮廓系数：[-1, 1]，值越大越好
silhouette = silhouette_score(X, labels)

# Calinski-Harabasz指数：值越大越好
ch_score = calinski_harabasz_score(X, labels)

# Davies-Bouldin指数：值越小越好
db_score = davies_bouldin_score(X, labels)
```

**K均值的肘部法则：**
```python
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

inertias = []
K_range = range(2, 11)
for k in K_range:
    model = KMeans(n_clusters=k, random_state=42)
    model.fit(X)
    inertias.append(model.inertia_)

plt.plot(K_range, inertias, 'bo-')
plt.xlabel('簇数量')
plt.ylabel('惯性')
plt.title('肘部法则')
```

## 降维

### 主成分分析 (PCA)

**PCA (`sklearn.decomposition.PCA`)**
- 使用SVD的线性降维
- 关键参数：
  - `n_components`: 主成分数量（整数或解释方差的浮点数）
  - `whiten`: 白化成分至单位方差
- 适用场景：线性关系，需要解释方差
- 示例：
```python
from sklearn.decomposition import PCA

# 保留解释95%方差的成分
pca = PCA(n_components=0.95)
X_reduced = pca.fit_transform(X)

print(f"原始维度: {X.shape[1]}")
print(f"降维后维度: {X_reduced.shape[1]}")
print(f"解释方差比: {pca.explained_variance_ratio_}")
print(f"总解释方差: {pca.explained_variance_ratio_.sum()}")

# 或指定确切成分数量
pca = PCA(n_components=2)
X_2d = pca.fit_transform(X)
```

**增量PCA**
- 适用于内存无法容纳的大型数据集
- 分批次处理数据
- 关键参数：`n_components`, `batch_size`
- 示例：
```python
from sklearn.decomposition import IncrementalPCA

pca = IncrementalPCA(n_components=50, batch_size=100)
X_reduced = pca.fit_transform(X)
```

**核PCA**
- 使用核方法的非线性降维
- 关键参数：`n_components`, `kernel` ('linear', 'poly', 'rbf', 'sigmoid')
- 适用场景：非线性关系
- 示例：
```python
from sklearn.decomposition import KernelPCA

pca = KernelPCA(n_components=2, kernel='rbf', gamma=0.1)
X_reduced = pca.fit_transform(X)
```

### 流形学习

**t-SNE (`sklearn.manifold.TSNE`)**
- t分布随机邻域嵌入
- 优秀的2D/3D可视化工具
- 关键参数：
  - `n_components`: 通常为2或3
  - `perplexity`: 局部与全局结构的平衡 (5-50)
  - `learning_rate`: 通常10-1000
  - `n_iter`: 迭代次数 (最小250)
- 适用场景：高维数据可视化
- 注意：大型数据集速度慢，无transform()方法
- 示例：
```python
from sklearn.manifold import TSNE

tsne = TSNE(n_components=2, perplexity=30, learning_rate=200, n_iter=1000, random_state=42)
X_embedded = tsne.fit_transform(X)

# 可视化
import matplotlib.pyplot as plt
plt.scatter(X_embedded[:, 0], X_embedded[:, 1], c=labels, cmap='viridis')
plt.title('t-SNE可视化')
```

**UMAP (不在scikit-learn中，但兼容)**
- 均匀流形逼近与投影
- 比t-SNE更快，更好保留全局结构
- 安装：`uv pip install umap-learn`
- 示例：
```python
from umap import UMAP

reducer = UMAP(n_components=2, n_neighbors=15, min_dist=0.1, random_state=42)
X_embedded = reducer.fit_transform(X)
```

**等距映射**
- 保持测地距离
- 关键参数：`n_components`, `n_neighbors`
- 适用场景：非线性流形
- 示例：
```python
from sklearn.manifold import Isomap

isomap = Isomap(n_components=2, n_neighbors=5)
X_embedded = isomap.fit_transform(X)
```

**局部线性嵌入 (LLE)**
- 保持局部邻域结构
- 关键参数：`n_components`, `n_neighbors`
- 示例：
```python
from sklearn.manifold import LocallyLinearEmbedding

lle = LocallyLinearEmbedding(n_components=2, n_neighbors=10)
X_embedded = lle.fit_transform(X)
```

**多维缩放 (MDS)**
- 保持成对距离
- 关键参数：`n_components`, `metric` (True/False)
- 示例：
```python
from sklearn.manifold import MDS

mds = MDS(n_components=2, metric=True, random_state=42)
X_embedded = mds.fit_transform(X)
```

### 矩阵分解

**非负矩阵分解 (NMF)**
- 分解为非负矩阵
- 关键参数：`n_components`, `init` ('nndsvd', 'random')
- 适用场景：非负数据（图像、文本）
- 可解释的成分
- 示例：
```python
from sklearn.decomposition import NMF

nmf = NMF(n_components=10, init='nndsvd', random_state=42)
W = nmf.fit_transform(X)  # 文档-主题矩阵
H = nmf.components_  # 主题-词矩阵
```

**截断SVD**
- 稀疏矩阵的SVD
- 类似PCA但适用于稀疏数据
- 适用场景：文本数据，稀疏矩阵
- 示例：
```python
from sklearn.decomposition import TruncatedSVD

svd = TruncatedSVD(n_components=100, random_state=42)
X_reduced = svd.fit_transform(X_sparse)
print(f"解释方差: {svd.explained_variance_ratio_.sum()}")
```

**快速独立成分分析**
- 将多元信号分离为独立成分
- 关键参数：`n_components`
- 适用场景：信号分离（如音频、脑电图）
- 示例：
```python
from sklearn.decomposition import FastICA

ica = FastICA(n_components=10, random_state=42)
S = ica.fit_transform(X)  # 独立源
A = ica.mixing_  # 混合矩阵
```

**潜在狄利克雷分配 (LDA)**
- 文本数据的主题建模
- 关键参数：`n_components` (主题数), `learning_method` ('batch', 'online')
- 适用场景：主题建模，文档聚类
- 示例：
```python
from sklearn.decomposition import LatentDirichletAllocation

lda = LatentDirichletAllocation(n_components=10, random_state=42)
doc_topics = lda.fit_transform(X_counts)  # 文档-主题分布

# 获取每个主题的关键词
feature_names = vectorizer.get_feature_names_out()
for topic_idx, topic in enumerate(lda.components_):
    top_words = [feature_names[i] for i in topic.argsort()[-10:]]
    print(f"Topic {topic_idx}: {', '.join(top_words)}")
```

## 离群点与新颖性检测

### 离群点检测

**隔离森林**
- 使用随机树隔离异常点
- 关键参数：
  - `contamination`: 预期离群点比例
  - `n_estimators`: 树的数量
- 适用场景：高维数据，效率优先
- 示例：
```python
from sklearn.ensemble import IsolationForest

model = IsolationForest(contamination=0.1, random_state=42)
predictions = model.fit_predict(X)  # -1表示离群点，1表示正常点
```

**局部离群因子**
- 测量局部密度偏差
- 关键参数：`n_neighbors`, `contamination`
- 适用场景：密度变化区域
- 示例：
```python
from sklearn.neighbors import LocalOutlierFactor

lof = LocalOutlierFactor(n_neighbors=20, contamination=0.1)
predictions = lof.fit_predict(X)  # -1表示离群点，1表示正常点
outlier_scores = lof.negative_outlier_factor_
```

**单类支持向量机**
- 围绕正常数据学习决策边界
- 关键参数：`nu` (离群点上界), `kernel`, `gamma`
- 适用场景：少量正常数据训练集
- 示例：
```python
from sklearn.svm import OneClassSVM

model = OneClassSVM(nu=0.1, kernel='rbf', gamma='auto')
model.fit(X_train)
predictions = model.predict(X_test)  # -1表示离群点，1表示正常点
```

**椭圆包络**
- 假设高斯分布
- 关键参数：`contamination`
- 适用场景：数据呈高斯分布
- 示例：
```python
from sklearn.covariance import EllipticEnvelope

model = EllipticEnvelope(contamination=0.1, random_state=42)
predictions = model.fit_predict(X)
```

## 高斯混合模型

**高斯混合模型**
- 使用高斯混合的概率聚类
- 关键参数：
  - `n_components`: 混合成分数量
  - `covariance_type`: 协方差类型 ('full', 'tied', 'diag', 'spherical')
- 适用场景：软聚类，需要概率估计
- 示例：
```python
from sklearn.mixture import GaussianMixture

gmm = GaussianMixture(n_components=3, covariance_type='full', random_state=42)
gmm.fit(X)

- **变化密度**：LocalOutlierFactor
- **高斯数据**：EllipticEnvelope
