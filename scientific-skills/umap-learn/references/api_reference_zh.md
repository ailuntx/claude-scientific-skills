# UMAP API 参考

## UMAP 类

`umap.UMAP(n_neighbors=15, n_components=2, metric='euclidean', n_epochs=None, learning_rate=1.0, init='spectral', min_dist=0.1, spread=1.0, low_memory=True, set_op_mix_ratio=1.0, local_connectivity=1.0, repulsion_strength=1.0, negative_sample_rate=5, transform_queue_size=4.0, a=None, b=None, random_state=None, metric_kwds=None, angular_rp_forest=False, target_n_neighbors=-1, target_metric='categorical', target_metric_kwds=None, target_weight=0.5, transform_seed=42, transform_mode='embedding', force_approximation_algorithm=False, verbose=False, unique=False, densmap=False, dens_lambda=2.0, dens_frac=0.3, dens_var_shift=0.1, output_dens=False, disconnection_distance=None, precomputed_knn=(None, None, None))`

寻找近似数据底层流形的低维嵌入。

### 核心参数

#### n_neighbors (int, 默认: 15)
用于流形近似的局部邻域大小。值越大保留的全局结构越多，值越小保留的局部结构越多。通常取值范围为2到100。

**调优指南：**
- 2-5：保留极局部结构
- 10-20：平衡局部/全局结构（典型值）
- 50-200：侧重全局结构

#### n_components (int, 默认: 2)
嵌入空间的维度。与t-SNE不同，UMAP在增加嵌入维度时具有良好的扩展性。

**常用值：**
- 2-3：可视化
- 5-10：聚类预处理
- 10-100：下游机器学习特征工程

#### metric (str 或 callable, 默认: 'euclidean')
使用的距离度量。支持：
- scipy.spatial.distance中的任何度量
- sklearn.metrics中的任何度量
- 自定义可调用距离函数（必须用Numba编译）

**常用度量：**
- `'euclidean'`：标准欧氏距离（默认）
- `'manhattan'`：L1距离
- `'cosine'`：余弦距离（适用于文本/文档向量）
- `'correlation'`：相关距离
- `'hamming'`：汉明距离（用于二进制数据）
- `'jaccard'`：杰卡德距离（用于二进制/集合数据）
- `'dice'`：Dice距离
- `'canberra'`：堪培拉距离
- `'braycurtis'`：Bray-Curtis距离
- `'chebyshev'`：切比雪夫距离
- `'minkowski'`：闵可夫斯基距离（通过metric_kwds指定p）
- `'precomputed'`：使用预计算距离矩阵

#### min_dist (float, 默认: 0.1)
嵌入点之间的有效最小距离。控制点的紧密程度。值越小嵌入越聚集。

**调优指南：**
- 0.0：适用于聚类应用
- 0.1-0.3：可视化（平衡）
- 0.5-0.99：松散结构保留

#### spread (float, 默认: 1.0)
嵌入点的有效尺度。与`min_dist`共同控制聚集与分散程度。决定嵌入空间中簇的分散程度。

### 训练参数

#### n_epochs (int, 默认: None)
训练轮数。若为None，则根据数据集大小自动确定（通常200-500轮）。

**手动调优：**
- 小数据集可能需要500+轮
- 大数据集可能在200轮收敛
- 轮数越多优化越好但训练越慢

#### learning_rate (float, 默认: 1.0)
SGD优化器的初始学习率。值越高收敛越快但可能错过最优解。

#### init (str 或 np.ndarray, 默认: 'spectral')
嵌入初始化方法：
- `'spectral'`：使用谱嵌入（默认，通常最佳）
- `'random'`：随机初始化
- `'pca'`：PCA初始化
- numpy数组：自定义初始化（形状：(n_samples, n_components)）

### 高级结构参数

#### local_connectivity (int, 默认: 1.0)
假设局部连接的最近邻数量。值越高流形连通性越强。

#### set_op_mix_ratio (float, 默认: 1.0)
构建模糊集并集时的并集与交集插值。1.0为纯并集，0.0为纯交集。

#### repulsion_strength (float, 默认: 1.0)
低维嵌入优化中负样本的权重。值越高嵌入点排斥越强。

#### negative_sample_rate (int, 默认: 5)
每个正样本选择的负样本数。值越高点间排斥越强，嵌入越分散，但计算成本增加。

### 监督学习参数

#### target_n_neighbors (int, 默认: -1)
构建目标单纯集时使用的最近邻数量。若为-1，则使用n_neighbors值。

#### target_metric (str, 默认: 'categorical')
目标值（标签）的距离度量：
- `'categorical'`：分类任务
- 其他度量用于回归任务

#### target_weight (float, 默认: 0.5)
目标信息与数据结构的权重平衡。范围0.0到1.0：
- 0.0：纯无监督嵌入（忽略标签）
- 0.5：平衡（默认）
- 1.0：纯监督嵌入（仅考虑标签）

### 变换参数

#### transform_queue_size (float, 默认: 4.0)
变换操作的最近邻搜索队列大小。值越大变换精度越高，但内存和计算时间增加。

#### transform_seed (int, 默认: 42)
变换操作的随机种子。确保变换结果可复现。

#### transform_mode (str, 默认: 'embedding')
新数据变换方法：
- `'embedding'`：标准方法（默认）
- `'graph'`：使用最近邻图

### 性能参数

#### low_memory (bool, 默认: True)
是否使用内存高效实现。仅在内存不受限且需更快性能时设为False。

#### verbose (bool, 默认: False)
是否在拟合过程中打印进度信息。

#### unique (bool, 默认: False)
是否仅考虑唯一数据点。当数据存在大量重复时可设为True以提升性能。

#### force_approximation_algorithm (bool, 默认: False)
即使小数据集也强制使用近似最近邻搜索。可提升大数据集性能。

#### angular_rp_forest (bool, 默认: False)
是否使用角度随机投影森林进行最近邻搜索。可提升高维归一化数据的性能。

### DensMAP 参数

DensMAP是保留局部密度信息的变体。

#### densmap (bool, 默认: False)
是否使用DensMAP算法替代标准UMAP。除拓扑结构外还保留局部密度。

#### dens_lambda (float, 默认: 2.0)
DensMAP优化中密度保留项的权重。值越高越强调密度保留。

#### dens_frac (float, 默认: 0.3)
DensMAP中用于密度估计的数据集比例。

#### dens_var_shift (float, 默认: 0.1)
DensMAP密度估计的正则化参数。

#### output_dens (bool, 默认: False)
是否额外输出局部密度估计。结果存储在`rad_orig_`和`rad_emb_`属性中。

### 其他参数

#### a (float, 默认: None)
控制嵌入的参数。若为None，则根据min_dist和spread自动确定。

#### b (float, 默认: None)
控制嵌入的参数。若为None，则根据min_dist和spread自动确定。

#### random_state (int, RandomState实例或None, 默认: None)
可复现性的随机状态。设为整数以获得可复现结果。

#### metric_kwds (dict, 默认: None)
距离度量的额外关键字参数。

#### disconnection_distance (float, 默认: None)
判定点断开的距离阈值。若为None，则使用图中的最大距离。

#### precomputed_knn (tuple, 默认: (None, None, None))
预计算的k近邻，格式为(knn_indices, knn_dists, knn_search_index)。适用于复用昂贵计算。

## 方法

### fit(X, y=None)
将UMAP模型拟合到数据。

**参数：**
- `X`：类数组，形状(n_samples, n_features) - 训练数据
- `y`：类数组，形状(n_samples,)，可选 - 监督降维的目标值

**返回：**
- `self`：拟合的UMAP对象

**设置属性：**
- `embedding_`：训练数据的嵌入表示
- `graph_`：流形的模糊单纯集近似
- `_raw_data`：训练数据副本
- `_small_data`：数据集是否被视为小型
- `_metric_kwds`：处理的度量关键字参数
- `_n_neighbors`：实际使用的n_neighbors
- `_initial_alpha`：初始学习率
- `_a`, `_b`：曲线参数

### fit_transform(X, y=None)
拟合模型并返回嵌入表示。

**参数：**
- `X`：类数组，形状(n_samples, n_features) - 训练数据
- `y`：类数组，形状(n_samples,)，可选 - 监督降维的目标值

**返回：**
- `X_new`：数组，形状(n_samples, n_components) - 嵌入数据

### transform(X)
将新数据变换到现有嵌入空间。

**参数：**
- `X`：类数组，形状(n_samples, n_features) - 待变换的新数据

**返回：**
- `X_new`：数组，形状(n_samples, n_components) - 新数据的嵌入表示

**重要说明：**
- 调用transform前必须拟合模型
- 变换质量取决于训练与测试分布的相似性
- 对于显著不同的数据分布，考虑使用Parametric UMAP

### inverse_transform(X)
将数据从嵌入空间逆变换回原始数据空间。

**参数：**
- `X`：类数组，形状(n_samples, n_components) - 嵌入数据点

**返回：**
- `X_new`：数组，形状(n_samples, n_features) - 原始空间的重构数据

**重要说明：**
- 计算开销大的操作
- 在训练嵌入凸包外效果不佳
- 重构质量因区域而异

### update(X)
用新数据更新模型。支持增量拟合。

**参数：**
- `X`：类数组，形状(n_samples, n_features) - 待整合的新数据

**返回：**
- `self`：更新的UMAP对象

**注意：** 实验性功能，可能无法保留批量训练的所有特性。

## 属性

### embedding_
数组，形状(n_samples, n_components) - 训练数据的嵌入表示。

### graph_
scipy.sparse.csr_matrix - 流形模糊单纯集近似的加权邻接矩阵。

### _raw_data
数组 - 原始训练数据的副本。

### _sparse_data
bool - 训练数据是否为稀疏格式。

### _small_data
bool - 数据集是否被视为小型（对小型数据集使用不同算法）。

### _input_hash
str - 输入数据的哈希值（用于缓存）。

### _knn_indices
数组 - 每个训练点的k近邻索引。

### _knn_dists
数组 - 每个训练点到k近邻的距离。

### _rp_forest
列表 - 用于近似最近邻搜索的随机投影森林。

## ParametricUMAP 类

`umap.ParametricUMAP(encoder=None, decoder=None, parametric_reconstruction=False, autoencoder_loss=False, reconstruction_validation=None, dims=None, batch_size=None, n_training_epochs=1, loss_report_frequency=10, optimizer=None, keras_fit_kwargs={}, **kwargs)`

使用神经网络学习嵌入函数的参数化UMAP。

### 额外参数（相对于UMAP）

#### encoder (tensorflow.keras.Model, 默认: None)
将数据编码为嵌入的Keras模型。若为None，则使用默认的3层架构（每层100个神经元）。

#### decoder (tensorflow.keras.Model, 默认: None)
将嵌入解码回数据空间的Keras模型。仅在parametric_reconstruction=True时使用。

#### parametric_reconstruction (bool, 默认: False)
是否使用参数化重构。需要解码器模型。

#### autoencoder_loss (bool, 默认: False)
是否在优化中包含重构损失。需要解码器模型。

#### reconstruction_validation (tuple, 默认: None)
用于训练期间监控重构损失的验证数据(X_val, y_val)。

#### dims (tuple, 默认: None)
编码器网络的输入维度。提供自定义编码器时必需。

#### batch_size (int, 默认: None)
神经网络训练的批次大小。若为None则自动确定。

#### n_training_epochs (int, 默认: 1)
神经网络的训练轮数。轮数越多质量越高但训练时间越长。

#### loss_report_frequency (int, 默认: 10)
训练期间报告损失的频率。

#### optimizer (tensorflow.keras.optimizers.Optimizer, 默认: None)
训练的Keras优化器。若为None，则使用带learning_rate参数的Adam。

#### keras_fit_kwargs (dict, 默认: {})
传递给Keras fit()方法的额外关键字参数。

### 方法
与UMAP类相同，但transform()和inverse_transform()使用学习到的神经网络实现更快推理。

## 实用函数

### umap.nearest_neighbors(X, n_neighbors, metric, metric_kwds={}, angular=False, random_state=None)
计算数据的k近邻。

**返回：** (knn_indices, knn_dists, rp_forest)

### umap.fuzzy_simplicial_set(X, n_neighbors, random_state, metric, metric_kwds={}, knn_indices=None, knn_dists=None, angular=False, set_op_mix_ratio=1.0, local_connectivity=1.0, apply_set_operations=True, verbose=False, return_dists=None)
构建数据的模糊单纯集表示。

**返回：** 稀疏矩阵形式的模糊单纯集

### umap.simplicial_set_embedding(data, graph, n_components, initial_alpha, a, b, gamma, negative_sample_rate, n_epochs, init, random_state, metric, metric_kwds, densmap, densmap_kwds, output_dens, output_metric, output_metric_kwds, euclidean_output, parallel=False, verbose=False)
执行优化以寻找低维嵌入。

**返回：** 嵌入数组

### umap.find_ab_params(spread, min_dist)
根据spread和min_dist拟合UMAP曲线的a,b参数。

**返回：** (a, b)元组

## AlignedUMAP 类

`umap.AlignedUMAP(n_neighbors=15, n_components=2, metric='euclidean', alignment_regularisation=1e-2, alignment_window_size=3, **kwargs)`

用于对齐多个相关数据集的UMAP变体。

### 额外参数

#### alignment_regularisation (float, 默认: 1e-2)
数据集间对齐正则化的强度。

#### alignment_window_size (int, 默认: 3)
对齐的相邻数据集数量。

### 方法

#### fit(X)
将模型拟合到多个数据集。

**参数：**
- `X`：数组列表 - 待对齐的数据集列表

**返回：**
- `self`：拟合的模型

### 属性

#### embeddings_
数组列表 - 对齐后的嵌入列表（每个输入数据集一个）。

## 使用示例

### 包含所有常用参数的基础用法

```python
import umap

# 标准二维可视化嵌入
reducer = umap.UMAP(
    n_neighbors=15,          # 平衡局部/全局结构
    n_components=2,          # 输出维度
    metric='euclidean',      # 距离度量
    min_dist=0.1,            # 点间最小距离
    spread=1.0,              # 嵌入点尺度
    random_state=42,         # 可复现性
    n_epochs=200,

```python
embedding = reducer.fit_transform(data)
```

### 自定义距离度量

```python
from numba import njit

@njit()
def custom_distance(x, y):
    """自定义距离函数（必须与Numba兼容）"""
    result = 0.0
    for i in range(x.shape[0]):
        result += abs(x[i] - y[i])
    return result

reducer = umap.UMAP(metric=custom_distance)
embedding = reducer.fit_transform(data)
```

### 使用自定义架构的参数化UMAP

```python
import tensorflow as tf
from umap.parametric_umap import ParametricUMAP

# 定义自定义编码器
encoder = tf.keras.Sequential([
    tf.keras.layers.InputLayer(input_shape=(input_dim,)),
    tf.keras.layers.Dense(256, activation='relu'),
    tf.keras.layers.Dropout(0.3),
    tf.keras.layers.Dense(128, activation='relu'),
    tf.keras.layers.Dropout(0.3),
    tf.keras.layers.Dense(2)  # 输出维度
])

# 定义用于重建的解码器
decoder = tf.keras.Sequential([
    tf.keras.layers.InputLayer(input_shape=(2,)),
    tf.keras.layers.Dense(128, activation='relu'),
    tf.keras.layers.Dense(256, activation='relu'),
    tf.keras.layers.Dense(input_dim)
])

# 使用自动编码器训练参数化UMAP
embedder = ParametricUMAP(
    encoder=encoder,
    decoder=decoder,
    dims=(input_dim,),
    parametric_reconstruction=True,
    autoencoder_loss=True,
    n_training_epochs=10,
    batch_size=128,
    n_neighbors=15,
    min_dist=0.1,
    random_state=42
)

embedding = embedder.fit_transform(data)
new_embedding = embedder.transform(new_data)
reconstructed = embedder.inverse_transform(embedding)
```

### 用于密度保持的DensMAP

```python
# 保留局部密度信息
reducer = umap.UMAP(
    densmap=True,           # 启用DensMAP
    dens_lambda=2.0,       # 密度保持的权重
    dens_frac=0.3,         # 用于密度估计的分数
    output_dens=True,      # 输出密度估计值
    n_neighbors=15,
    min_dist=0.1,
    random_state=42
)

embedding = reducer.fit_transform(data)

# 访问密度估计值
original_density = reducer.rad_orig_  # 原始空间中的密度
embedded_density = reducer.rad_emb_   # 嵌入空间中的密度
```

### 用于时间序列的对齐UMAP

```python
from umap import AlignedUMAP

# 多个相关数据集（例如，不同时间点）
datasets = [day1_data, day2_data, day3_data, day4_data]

# 对齐嵌入
mapper = AlignedUMAP(
    n_neighbors=15,
    alignment_regularisation=1e-2,  # 对齐强度
    alignment_window_size=2,        # 与相邻数据集对齐
    n_components=2,
    random_state=42
)

mapper.fit(datasets)

# 访问对齐的嵌入
aligned_embeddings = mapper.embeddings_
# aligned_embeddings[0] 是第1天的嵌入
# aligned_embeddings[1] 是第2天的嵌入，以此类推
```
