```markdown
---
name: umap-learn
description: UMAP降维技术。用于高维数据的快速非线性流形学习，支持2D/3D可视化、聚类预处理（HDBSCAN）、监督/参数化UMAP。
license: BSD-3-Clause许可证
metadata:
    skill-author: K-Dense Inc.
---

# UMAP学习库

## 概述

UMAP（均匀流形逼近与投影）是一种用于可视化和通用非线性降维的技术。应用此技能可实现快速、可扩展的嵌入，保留局部与全局结构，支持监督学习和聚类预处理。

## 快速入门

### 安装

```bash
uv pip install umap-learn
```

### 基础用法

UMAP遵循scikit-learn规范，可作为t-SNE或PCA的替代方案。

```python
import umap
from sklearn.preprocessing import StandardScaler

# 准备数据（标准化至关重要）
scaled_data = StandardScaler().fit_transform(data)

# 方法1：单步操作（拟合与转换）
embedding = umap.UMAP().fit_transform(scaled_data)

# 方法2：分步操作（用于复用训练模型）
reducer = umap.UMAP(random_state=42)
reducer.fit(scaled_data)
embedding = reducer.embedding_  # 访问训练后的嵌入
```

**关键预处理要求：** 应用UMAP前必须将特征标准化至可比尺度，确保各维度权重均衡。

### 典型工作流

```python
import umap
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler

# 1. 数据预处理
scaler = StandardScaler()
scaled_data = scaler.fit_transform(raw_data)

# 2. 创建并拟合UMAP
reducer = umap.UMAP(
    n_neighbors=15,
    min_dist=0.1,
    n_components=2,
    metric='euclidean',
    random_state=42
)
embedding = reducer.fit_transform(scaled_data)

# 3. 可视化
plt.scatter(embedding[:, 0], embedding[:, 1], c=labels, cmap='Spectral', s=5)
plt.colorbar()
plt.title('UMAP嵌入结果')
plt.show()
```

## 参数调优指南

UMAP有四个核心参数控制嵌入行为，理解这些参数对有效使用至关重要。

### n_neighbors（默认：15）

**作用：** 平衡嵌入中的局部与全局结构。

**原理：** 控制UMAP学习流形结构时考察的局部邻域大小。

**取值影响：**
- **低值（2-5）：** 强调局部细节，但可能导致数据碎片化
- **中值（15-20）：** 兼顾局部结构与全局关系（推荐起点）
- **高值（50-200）：** 优先保留宏观拓扑结构，牺牲细节

**建议：** 从15开始调整。增大值强化全局结构，减小值突出局部细节。

### min_dist（默认：0.1）

**作用：** 控制低维空间中点的聚集紧密程度。

**原理：** 设定输出表示中点之间的最小允许距离。

**取值影响：**
- **低值（0.0-0.1）：** 生成密集嵌入，适用于聚类；揭示精细拓扑细节
- **高值（0.5-0.99）：** 避免紧密堆积；强调宏观拓扑保留而非局部结构

**建议：** 聚类应用用0.0，可视化用0.1-0.3，松散结构用0.5+。

### n_components（默认：2）

**作用：** 决定嵌入输出空间的维度。

**关键特性：** 与t-SNE不同，UMAP在嵌入维度上扩展性良好，支持超越可视化的应用。

**常见用途：**
- **2-3维：** 可视化
- **5-10维：** 聚类预处理（比2D更好保留密度）
- **10-50维：** 下游ML模型的特征工程

**建议：** 可视化用2维，聚类用5-10维，ML流水线用更高维度。

### metric（默认：'euclidean'）

**作用：** 指定输入数据点间的距离计算方式。

**支持度量：**
- **闵可夫斯基变体：** euclidean, manhattan, chebyshev
- **空间度量：** canberra, braycurtis, haversine
- **相关性度量：** cosine, correlation（适用于文本/文档嵌入）
- **二元数据度量：** hamming, jaccard, dice, russellrao, kulsinski, rogerstanimoto, sokalmichener, sokalsneath, yule
- **自定义度量：** 通过Numba使用用户定义的距离函数

**建议：** 数值数据用euclidean，文本/文档向量用cosine，二元数据用hamming。

### 参数调优示例

```python
# 强调局部结构的可视化配置
umap.UMAP(n_neighbors=15, min_dist=0.1, n_components=2, metric='euclidean')

# 聚类预处理配置
umap.UMAP(n_neighbors=30, min_dist=0.0, n_components=10, metric='euclidean')

# 文档嵌入配置
umap.UMAP(n_neighbors=15, min_dist=0.1, n_components=2, metric='cosine')

# 保留全局结构配置
umap.UMAP(n_neighbors=100, min_dist=0.5, n_components=2, metric='euclidean')
```

## 监督与半监督降维

UMAP支持融入标签信息引导嵌入过程，在保留内部结构的同时实现类别分离。

### 监督UMAP

通过`y`参数传递目标标签进行拟合：

```python
# 监督降维
embedding = umap.UMAP().fit_transform(data, y=labels)
```

**核心优势：**
- 实现清晰分离的类别
- 保留各类别内部结构
- 维持类别间的全局关系

**适用场景：** 拥有标注数据且需分离已知类别，同时保持有意义的点嵌入。

### 半监督UMAP

对于部分标注数据，按scikit-learn规范用`-1`标记未标注点：

```python
# 创建半监督标签
semi_labels = labels.copy()
semi_labels[unlabeled_indices] = -1

# 使用部分标签拟合
embedding = umap.UMAP().fit_transform(data, y=semi_labels)
```

**适用场景：** 标注成本高或数据量大于标注量时。

### UMAP度量学习

在标注数据上训练监督嵌入，应用于新未标注数据：

```python
# 在标注数据上训练
mapper = umap.UMAP().fit(train_data, train_labels)

# 转换未标注测试数据
test_embedding = mapper.transform(test_data)

# 作为下游分类器的特征工程
from sklearn.svm import SVC
clf = SVC().fit(mapper.embedding_, train_labels)
predictions = clf.predict(test_embedding)
```

**适用场景：** 机器学习流水线中的监督特征工程。

## UMAP聚类应用

UMAP可作为基于密度的聚类算法（如HDBSCAN）的有效预处理，克服维度灾难。

### 聚类最佳实践

**核心原则：** 为聚类配置的UMAP参数应与可视化不同。

**推荐参数：**
- **n_neighbors：** 增至~30（默认15过于局部化，可能产生人工细粒度聚类）
- **min_dist：** 设为0.0（在聚类内密集排布点以形成清晰边界）
- **n_components：** 使用5-10维（保持性能同时改善密度保留）

### 聚类工作流

```python
import umap
import hdbscan
from sklearn.preprocessing import StandardScaler

# 1. 数据预处理
scaled_data = StandardScaler().fit_transform(data)

# 2. 使用聚类优化参数配置UMAP
reducer = umap.UMAP(
    n_neighbors=30,
    min_dist=0.0,
    n_components=10,  # 高于2维以更好保留密度
    metric='euclidean',
    random_state=42
)
embedding = reducer.fit_transform(scaled_data)

# 3. 应用HDBSCAN聚类
clusterer = hdbscan.HDBSCAN(
    min_cluster_size=15,
    min_samples=5,
    metric='euclidean'
)
labels = clusterer.fit_predict(embedding)

# 4. 评估
from sklearn.metrics import adjusted_rand_score
score = adjusted_rand_score(true_labels, labels)
print(f"调整兰德指数: {score:.3f}")
print(f"聚类数量: {len(set(labels)) - (1 if -1 in labels else 0)}")
print(f"噪声点数量: {sum(labels == -1)}")
```

### 聚类后可视化

```python
# 创建独立于聚类的2D可视化嵌入
vis_reducer = umap.UMAP(n_neighbors=15, min_dist=0.1, n_components=2, random_state=42)
vis_embedding = vis_reducer.fit_transform(scaled_data)

# 带聚类标签的可视化
import matplotlib.pyplot as plt
plt.scatter(vis_embedding[:, 0], vis_embedding[:, 1], c=labels, cmap='Spectral', s=5)
plt.colorbar()
plt.title('带HDBSCAN聚类的UMAP可视化')
plt.show()
```

**重要提示：** UMAP不能完全保留密度，可能产生人工聚类划分。务必验证并探索结果聚类。

## 新数据转换

通过`transform()`方法，UMAP可将新数据投影到已学习的嵌入空间。

### 基础转换用法

```python
# 在训练数据上训练
trans = umap.UMAP(n_neighbors=15, random_state=42).fit(X_train)

# 转换测试数据
test_embedding = trans.transform(X_test)
```

### 机器学习流水线集成

```python
from sklearn.svm import SVC
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
import umap

# 数据分割
X_train, X_test, y_train, y_test = train_test_split(data, labels, test_size=0.2)

# 预处理
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 训练UMAP
reducer = umap.UMAP(n_components=10, random_state=42)
X_train_embedded = reducer.fit_transform(X_train_scaled)
X_test_embedded = reducer.transform(X_test_scaled)

# 在嵌入上训练分类器
clf = SVC()
clf.fit(X_train_embedded, y_train)
accuracy = clf.score(X_test_embedded, y_test)
print(f"测试准确率: {accuracy:.3f}")
```

### 重要注意事项

**数据一致性：** transform方法假设高维空间中训练/测试数据的整体分布一致。若不成立，考虑改用参数化UMAP。

**性能：** 转换操作高效（通常<1秒），但首次调用可能因Numba JIT编译较慢。

**scikit-learn兼容性：** UMAP遵循标准sklearn规范，可无缝集成流水线：

```python
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('umap', umap.UMAP(n_components=10)),
    ('classifier', SVC())
])

pipeline.fit(X_train, y_train)
predictions = pipeline.predict(X_test)
```

## 高级功能

### 参数化UMAP

参数化UMAP通过神经网络映射函数替代直接嵌入优化。

**与标准UMAP的关键差异：**
- 使用TensorFlow/Keras训练编码器网络
- 支持高效转换新数据
- 通过解码器网络支持重建（逆变换）
- 允许自定义架构（图像用CNN，序列用RNN）

**安装：**
```bash
uv pip install umap-learn[parametric_umap]
# 需TensorFlow 2.x
```

**基础用法：**
```python
from umap.parametric_umap import ParametricUMAP

# 默认架构（3层100神经元全连接网络）
embedder = ParametricUMAP()
embedding = embedder.fit_transform(data)

# 高效转换新数据
new_embedding = embedder.transform(new_data)
```

**自定义架构：**
```python
import tensorflow as tf

# 定义自定义编码器
encoder = tf.keras.Sequential([
    tf.keras.layers.InputLayer(input_shape=(input_dim,)),
    tf.keras.layers.Dense(128, activation='relu'),
    tf.keras.layers.Dense(64, activation='relu'),
    tf.keras.layers.Dense(2)  # 输出维度
])

embedder = ParametricUMAP(encoder=encoder, dims=(input_dim,))
embedding = embedder.fit_transform(data)
```

**参数化UMAP适用场景：**
- 训练后需高效转换新数据
- 需要重建能力（逆变换）
- 需将UMAP与自编码器结合
- 处理复杂数据类型（图像/序列）需专用架构

**标准UMAP适用场景：**
- 追求简单性与快速原型设计
- 数据集小且计算效率非关键
- 无需对未来数据应用学习转换

### 逆变换

逆变换支持从低维嵌入重建高维数据。

**基础用法：**
```python
reducer = umap.UMAP()
embedding = reducer.fit_transform(data)

# 从嵌入坐标重建高维数据
reconstructed = reducer.inverse_transform(embedding)
```

**重要限制：**
- 计算开销大
- 在嵌入凸包外效果差
- 聚类间隙区域精度下降

**应用场景：**
- 理解嵌入数据结构
- 可视化聚类间平滑过渡
- 探索数据点间插值
- 在嵌入空间生成合成样本

**示例：探索嵌入空间**
```python
import numpy as np

# 在嵌入空间创建点阵网格
x = np.linspace(embedding[:, 0].min(), embedding[:, 0].max(), 10)
y = np.linspace(embedding[:, 1].min(), embedding[:, 1].max(), 10)
xx, yy = np.meshgrid(x, y)
grid_points = np.c_[xx.ravel(), yy.ravel()]

# 从网格重建样本
reconstructed_samples = reducer.inverse_transform(grid_points)
```

### AlignedUMAP

用于分析时序或相关数据集（如时间序列实验、批次数据）：

```python
from umap import AlignedUMAP

# 相关数据集列表
datasets = [day1_data, day2_data, day3_data]

# 创建对齐嵌入
mapper = AlignedUMAP().fit(datasets)
aligned_embeddings = mapper.embeddings_  # 嵌入列表
```

**适用场景：** 跨相关数据集比较嵌入，同时保持坐标系一致。

## 结果复现

为确保结果可复现，务必设置`random_state`参数：

```python
reducer = umap.UMAP(random_state=42)
```

UMAP使用随机优化，未固定随机状态时不同运行结果会有轻微差异。

## 常见问题与解决方案

**问题：** 不连通组件或碎片化聚类
- **解决方案：** 增大`n_neighbors`以强化全局结构

**问题：** 聚类过度分散或分离不佳
- **解决方案：** 减小`min_dist`以允许更紧密堆积

**问题：** 聚类结果差
- **解决方案：** 使用聚类专用参数（n_neighbors=30, min_dist=0.0, n_components=5-10）

**问题：** 转换结果与训练差异显著
- **解决方案：** 确保测试数据分布匹配训练数据，或改用参数化UMAP

**问题：** 大数据集性能慢
- **解决方案：** 设置`low_memory=True`（默认），或先用PCA降维

**问题：** 所有点坍缩至单一聚类
- **解决方案：** 检查数据预处理（确保正确标准化），增大`min_dist`

## 资源

### references/

包含详细API文档：
- `api_reference.md`：完整UMAP类参数与方法

需要详细参数信息或高级方法用法时加载这些参考文档。
```
