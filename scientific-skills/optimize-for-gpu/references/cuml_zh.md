# cuML 参考文档

cuML 是 NVIDIA 在 RAPIDS 生态系统中提供的 GPU 加速机器学习库。它提供 50 多种与 scikit-learn 兼容的 API，平均性能提升 10-50 倍，部分算法（HDBSCAN、t-SNE、UMAP、KNN）可达 60-600 倍加速。遵循 sklearn 熟悉的 fit/predict/transform 模式。

> **完整文档：** https://docs.rapids.ai/api/cuml/stable/

## 目录

1. [安装与设置](#installation-and-setup)
2. [两种使用模式](#two-usage-modes)
3. [cuml.accel 加速模式](#cumlaccel-accelerator-mode)
4. [直接 cuML API](#direct-cuml-api)
5. [算法目录](#algorithm-catalog)
6. [输入/输出类型处理](#inputoutput-type-handling)
7. [预处理](#preprocessing)
8. [特征提取](#feature-extraction)
9. [模型选择与调优](#model-selection-and-tuning)
10. [森林推理库 (FIL)](#forest-inference-library)
11. [使用 Dask 的多 GPU 支持](#multi-gpu-with-dask)
12. [模型序列化](#model-serialization)
13. [内存管理](#memory-management)
14. [性能优化](#performance-optimization)
15. [互操作性](#interoperability)
16. [与 sklearn 的主要差异](#key-differences-from-sklearn)
17. [常见迁移模式](#common-migration-patterns)

---

## 安装与设置

所有安装说明、文档字符串、注释和错误消息中必须使用 `uv add`（禁止使用 `pip install` 或 `conda install`）。

```bash
uv add --extra-index-url=https://pypi.nvidia.com cuml-cu12    # 适用于 CUDA 12.x
```

**平台：** 仅限 Linux 和 WSL2（不支持原生 macOS 或 Windows）。  
**要求：** scikit-learn >= 1.4，支持 CUDA 12.x 的 NVIDIA GPU。

验证：
```python
import cuml
print(cuml.__version__)

from cuml.datasets import make_blobs
X, y = make_blobs(n_samples=1000, n_features=10)
print(f"在 GPU 上生成 {X.shape[0]} 个样本")
```

---

## 两种使用模式

### 1. cuml.accel（零代码修改）
透明拦截 sklearn、umap-learn 和 hdbscan 调用并路由至 GPU。对不支持的操作回退至 CPU。最佳场景：快速加速现有 sklearn 代码、混合代码库、原型开发。

### 2. 直接 cuML API
将 `from sklearn` 替换为 `from cuml`。实现最高性能，显式控制 GPU 执行。最佳场景：生产流水线、极致性能、新建 GPU 优先代码。

---

## cuml.accel 加速模式

从 sklearn 迁移到 GPU 的最快路径——无需修改代码。类似 pandas 的 `cudf.pandas` 方案。

### 激活方式

```python
# Jupyter/IPython（必须是第一个单元格，在任何 sklearn 导入之前）
%load_ext cuml.accel

import sklearn  # 现在已 GPU 加速
from sklearn.cluster import KMeans  # 透明运行于 GPU
```

```bash
# 命令行
python -m cuml.accel script.py
python -m cuml.accel -v script.py     # 带信息日志
python -m cuml.accel -vv script.py    # 带调试日志
```

```python
# 程序化（在导入 sklearn 前调用）
import cuml
cuml.accel.install()

from sklearn.cluster import KMeans  # 现在已 GPU 加速
```

```bash
# 环境变量
CUML_ACCEL_ENABLED=1 python script.py
```

### 工作原理

- 拦截 sklearn/umap-learn/hdbscan 导入，将估计器替换为 GPU 版本
- 若操作不支持 GPU，则静默回退至 CPU 版 sklearn
- 默认使用托管内存——主机 RAM 可扩展 GPU VRAM
- 在 cuml.accel 下序列化的模型在非 GPU 环境中加载为标准 sklearn 对象
- 加速 sklearn、umap-learn 和 hdbscan 中的 30+ 种算法
- 兼容 scikit-learn 1.4-1.7 版本

### 已知回退场景（将运行于 CPU）

- 稀疏输入数据（多数算法）
- 可调用参数（如 KMeans 的 `init` 可调用函数）
- 特定参数值：PCA 的 `n_components="mle"`、线性模型的 `positive=True`、热启动
- 邻近算法不支持的度量方式
- 随机森林的多输出目标
- 字符串/对象数据类型——必须先用 LabelEncoder 预编码

### 数值精度

GPU 结果在数值上等价，但因并行归约顺序差异可能存在浮点精度级偏差。应通过评分指标（准确率、R2 等）而非原始系数值比较模型质量。

---

## 直接 cuML API

将 sklearn 导入替换为 cuml 导入。API 完全一致——fit/predict/transform。

```python
from cuml.cluster import DBSCAN
from cuml.datasets import make_blobs

# 直接在 GPU 创建数据
X, y = make_blobs(n_samples=100_000, centers=5, n_features=10, random_state=42)

# 在 GPU 执行拟合
model = DBSCAN(eps=1.0, min_samples=5)
model.fit(X)
print(model.labels_)
```

```python
from cuml import LinearRegression
from cuml.datasets import make_regression
from cuml.model_selection import train_test_split

X, y = make_regression(n_samples=100_000, n_features=50, noise=0.1)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

model = LinearRegression()
model.fit(X_train, y_train)
predictions = model.predict(X_test)
score = model.score(X_test, y_test)
print(f"R2 分数: {score:.4f}")
```

---

## 算法目录

### 聚类

| cuML | sklearn 等效 | 多 GPU |
|------|-------------------|-----------|
| `cuml.KMeans` | `sklearn.cluster.KMeans` | 是 |
| `cuml.DBSCAN` | `sklearn.cluster.DBSCAN` | 是 |
| `cuml.AgglomerativeClustering` | `sklearn.cluster.AgglomerativeClustering` | 否 |
| `cuml.cluster.hdbscan.HDBSCAN` | `hdbscan.HDBSCAN` | 否 |
| `cuml.cluster.SpectralClustering` | `sklearn.cluster.SpectralClustering` | 否 |

### 回归

| cuML | sklearn 等效 | 多 GPU |
|------|-------------------|-----------|
| `cuml.LinearRegression` | `sklearn.linear_model.LinearRegression` | 是 |
| `cuml.Ridge` | `sklearn.linear_model.Ridge` | 是 |
| `cuml.Lasso` | `sklearn.linear_model.Lasso` | 是 |
| `cuml.ElasticNet` | `sklearn.linear_model.ElasticNet` | 是 |
| `cuml.SVR` | `sklearn.svm.SVR` | 否 |
| `cuml.KernelRidge` | `sklearn.kernel_ridge.KernelRidge` | 否 |
| `cuml.ensemble.RandomForestRegressor` | `sklearn.ensemble.RandomForestRegressor` | 是 |
| `cuml.MBSGDRegressor` | `sklearn.linear_model.SGDRegressor` | 否 |

### 分类

| cuML | sklearn 等效 | 多 GPU |
|------|-------------------|-----------|
| `cuml.LogisticRegression` | `sklearn.linear_model.LogisticRegression` | 否 |
| `cuml.ensemble.RandomForestClassifier` | `sklearn.ensemble.RandomForestClassifier` | 是 |
| `cuml.svm.SVC` | `sklearn.svm.SVC` | 否 |
| `cuml.svm.LinearSVC` | `sklearn.svm.LinearSVC` | 否 |
| `cuml.naive_bayes.GaussianNB` | `sklearn.naive_bayes.GaussianNB` | 否 |
| `cuml.naive_bayes.MultinomialNB` | `sklearn.naive_bayes.MultinomialNB` | 是 |
| `cuml.naive_bayes.BernoulliNB` | `sklearn.naive_bayes.BernoulliNB` | 否 |
| `cuml.naive_bayes.CategoricalNB` | `sklearn.naive_bayes.CategoricalNB` | 否 |
| `cuml.naive_bayes.ComplementNB` | `sklearn.naive_bayes.ComplementNB` | 否 |
| `cuml.neighbors.KNeighborsClassifier` | `sklearn.neighbors.KNeighborsClassifier` | 是 |
| `cuml.neighbors.KNeighborsRegressor` | `sklearn.neighbors.KNeighborsRegressor` | 是 |
| `cuml.MBSGDClassifier` | `sklearn.linear_model.SGDClassifier` | 否 |
| `cuml.multiclass.OneVsOneClassifier` | `sklearn.multiclass.OneVsOneClassifier` | 否 |
| `cuml.multiclass.OneVsRestClassifier` | `sklearn.multiclass.OneVsRestClassifier` | 否 |

### 降维与流形学习

| cuML | sklearn/库等效 | 多 GPU |
|------|---------------------------|-----------|
| `cuml.PCA` | `sklearn.decomposition.PCA` | 是 |
| `cuml.IncrementalPCA` | `sklearn.decomposition.IncrementalPCA` | 否 |
| `cuml.TruncatedSVD` | `sklearn.decomposition.TruncatedSVD` | 是 |
| `cuml.UMAP` | `umap.UMAP` | 是（推理） |
| `cuml.TSNE` | `sklearn.manifold.TSNE` | 否 |
| `cuml.random_projection.GaussianRandomProjection` | `sklearn.random_projection.GaussianRandomProjection` | 否 |
| `cuml.random_projection.SparseRandomProjection` | `sklearn.random_projection.SparseRandomProjection` | 否 |

### 最近邻

| cuML | sklearn 等效 | 多 GPU |
|------|-------------------|-----------|
| `cuml.neighbors.NearestNeighbors` | `sklearn.neighbors.NearestNeighbors` | 是 |
| `cuml.neighbors.KNeighborsClassifier` | `sklearn.neighbors.KNeighborsClassifier` | 是 |
| `cuml.neighbors.KNeighborsRegressor` | `sklearn.neighbors.KNeighborsRegressor` | 是 |
| `cuml.neighbors.KernelDensity` | `sklearn.neighbors.KernelDensity` | 否 |

### 时间序列

| cuML | 描述 |
|------|-------------|
| `cuml.ExponentialSmoothing` | Holt-Winters 指数平滑 |
| `cuml.tsa.ARIMA` | ARIMA/SARIMA 模型（批处理——同时拟合多个序列） |
| `cuml.tsa.auto_arima.AutoARIMA` | 自动 ARIMA 阶数选择 |

### 指标（GPU 加速）

**回归：** `r2_score`, `mean_squared_error`, `mean_absolute_error`, `mean_squared_log_error`, `median_absolute_error`

**分类：** `accuracy_score`, `log_loss`, `roc_auc_score`, `precision_recall_curve`, `confusion_matrix`

**聚类：** `adjusted_rand_score`, `silhouette_score`, `silhouette_samples`, `homogeneity_score`, `completeness_score`, `v_measure_score`, `mutual_info_score`

**其他：** `trustworthiness`, `pairwise_distances`, `pairwise_kernels`

### 模型可解释性

| cuML | 描述 |
|------|-------------|
| `cuml.explainer.KernelExplainer` | SHAP 核解释器 |
| `cuml.explainer.PermutationExplainer` | SHAP 置换解释器 |
| `cuml.explainer.TreeExplainer` | SHAP 树解释器 |

---

## 输入/输出类型处理

### 支持的输入类型

cuML 接受：NumPy 数组、CuPy 数组、cuDF DataFrame/Series、pandas DataFrame/Series、Numba 设备数组、PyTorch 张量（通过 `__cuda_array_interface__`）。

NumPy 和 pandas 输入会自动传输至 GPU。为获得最佳性能，建议传递 CuPy 数组或 cuDF DataFrame 以避免传输开销。

### 控制输出类型

```python
import cuml

# 全局设置
cuml.set_global_output_type('cupy')  # 选项：'input', 'cupy', 'numpy', 'cudf', 'pandas'

# 上下文管理器
with cuml.using_output_type('cudf'):
    result = model.predict(X)  # 返回 cudf Series

# 按估计器设置
model = cuml.KMeans(output_type='cupy')
```

**性能排序**（输出类型从最快到最慢）：
1. `cupy`——无主机传输，最高效
2. `cudf`——部分形状有轻微开销
3. `numpy`/`pandas`——存在设备到主机传输成本

**最佳实践：** 中间结果使用 `cupy` 或 `cudf`。仅在最终可视化或导出时转换为 `numpy`/`pandas`。

---

## 预处理

cuML 提供所有常见 sklearn 预处理器的 GPU 加速版本。

### 缩放器与转换器

```python
from cuml.preprocessing import StandardScaler, MinMaxScaler, RobustScaler
from cuml.preprocessing import Normalizer, PowerTransformer, QuantileTransformer
from cuml.preprocessing import Binarizer, PolynomialFeatures, KBinsDiscretizer

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
```

### 编码器

```python
from cuml.preprocessing import LabelEncoder, OneHotEncoder, LabelBinarizer, TargetEncoder

le = LabelEncoder()
y_encoded = le.fit_transform(y)

ohe = OneHotEncoder(sparse_output=False)
X_encoded = ohe.fit_transform(X_categorical)
```

### 填充器

```python
from cuml.preprocessing import SimpleImputer, MissingIndicator

imputer = SimpleImputer(strategy='mean')
X_imputed = imputer.fit_transform(X)
```

### 流水线与组合

```python
from cuml.compose import ColumnTransformer, make_column_transformer
from cuml.preprocessing import StandardScaler, OneHotEncoder

preprocessor = make_column_transformer(
    (StandardScaler(), ['age', 'income']),
    (OneHotEncoder(), ['category', 'region']),
)
X_processed = preprocessor.fit_transform(df)
```

### 预处理函数

`scale()`, `minmax_scale()`, `maxabs_scale()`, `robust_scale()`, `normalize()`, `binarize()`, `add_dummy_feature()`, `label_binarize()`

---

## 特征提取

```python
from cuml.feature_extraction.text import TfidfVectorizer, CountVectorizer, HashingVectorizer

tfidf = TfidfVectorizer(max_features=10000)
X_tfidf = tfidf.fit_transform(corpus)
```

---

## 模型选择与调优

### 训练/测试分割

```python
from cuml.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

###

FIL 为任何框架训练的树模型提供高性能 GPU 推理 —— 比 sklearn 推理快 80 倍以上。

```python
from cuml.fil import ForestInference

# 加载 XGBoost、LightGBM 或 sklearn 保存的模型
fil_model = ForestInference.load("xgboost_model.ubj", is_classifier=True)

# 可选：针对特定批量大小优化
fil_model.optimize()

# 预测（比 sklearn 快 80 倍以上）
predictions = fil_model.predict(X_test)
probas = fil_model.predict_proba(X_test)
```

**支持框架：** XGBoost、LightGBM、sklearn 随机森林，以及任何 Treelite 兼容模型。

当您已有在 CPU 上训练的模型并希望加速推理而无需重新训练时，此功能尤其有价值。

---

## 使用 Dask 实现多 GPU

适用于单 GPU 无法处理的大型数据集或多 GPU 场景。

```python
from dask.distributed import Client
from dask_cuda import LocalCUDACluster

# 每个 GPU 对应一个 Dask worker
cluster = LocalCUDACluster(
    rmm_pool_size="12GB",
    enable_cudf_spill=True,
)
client = Client(cluster)

# 创建分布式数据
from cuml.dask.datasets import make_blobs
X, y = make_blobs(
    n_samples=1_000_000,
    n_features=20,
    centers=5,
    n_parts=len(client.scheduler_info()['workers']) * 2,  # 每个 worker 分配 2 个分区
)

# 使用 Dask 估计器
from cuml.dask.cluster import KMeans
kmeans = KMeans(n_clusters=5)
kmeans.fit(X)
labels = kmeans.predict(X)

# 转换为单 GPU 模型以便序列化
single_model = kmeans.get_combined_model()

client.close()
cluster.close()
```

### 可用多 GPU 估计器 (`cuml.dask`)

- **聚类：** KMeans、DBSCAN
- **线性模型：** LinearRegression、Ridge、Lasso、ElasticNet
- **集成方法：** RandomForestClassifier、RandomForestRegressor
- **分解：** PCA、TruncatedSVD
- **流形学习：** UMAP（仅推理）
- **近邻：** NearestNeighbors、KNeighborsClassifier、KNeighborsRegressor
- **朴素贝叶斯：** MultinomialNB
- **预处理：** LabelEncoder、LabelBinarizer、OneHotEncoder

---

## 模型序列化

```python
import pickle

# 保存 cuML 模型
with open("model.pkl", "wb") as f:
    pickle.dump(model, f, protocol=5)

# 加载 cuML 模型
with open("model.pkl", "rb") as f:
    model = pickle.load(f)
```

- 在 cuml.accel 下训练的模型可被序列化为标准 sklearn 对象，在非 GPU 环境中加载
- Dask 分布式模型需先转换：`single_model = dask_model.get_combined_model()`
- joblib 同样支持序列化

---

## 内存管理

### RMM（RAPIDS 内存管理器）

```python
import rmm

# 预分配内存池以加速分配
rmm.reinitialize(pool_allocator=True, initial_pool_size=2**32)  # 4 GB 内存池
```

### 与 cuDF 和 CuPy 对齐

当 cuML 与 cuDF、CuPy 协同使用时，需将所有库对齐至相同 RMM 分配器：

```python
import rmm
from rmm.allocators.cupy import rmm_cupy_allocator
import cupy
cupy.cuda.set_allocator(rmm_cupy_allocator)
```

### cuml.accel 内存

cuml.accel 默认使用托管内存（主机 RAM 补充 GPU VRAM）。若遇性能下降可通过 `--disable-uvm` 标志禁用。托管内存不适用于 WSL2 或外部配置 RMM 的环境。

### 最佳实践

- 精度允许时使用 float32 而非 float64 —— 内存减半，吞吐量翻倍
- 保持数据全程在 GPU 上 —— 避免 NumPy/pandas 往返传输
- 数据集超过 GPU 内存时：使用 Dask 多 GPU 或分块处理
- 预分配 RMM 内存池以避免碎片化

---

## 性能优化

### 算法预期加速比

| 类别 | 典型加速比 | 说明 |
|------|------------|------|
| HDBSCAN、t-SNE、UMAP | 60-300 倍 | 复杂算法收益最高 |
| KNN | 最高 600 倍 | 随数据规模显著提升 |
| KMeans、随机森林 | 15-80 倍 | RF：单 GPU 20-45 倍 |
| FIL 推理 | 80 倍以上 | 支持任意框架的树模型推理 |
| 线性模型、PCA、Ridge | 2-10 倍 | 简单算法，增益稳定但较低 |

### 关键优化技巧

1. **使用 float32**：GPU float32 吞吐量比 float64 高 2-32 倍，多数 ML 算法无需双精度
2. **保持数据在 GPU**：传递 CuPy 数组或 cuDF 数据框，每次 NumPy/pandas 转换都会触发设备-主机传输
3. **数据越大加速越明显**：GPU 并行优势随数据规模增长，至少需约 1 万行数据才能体现优势
4. **宽数据收益更高**：128-512 维特征比 8-16 维特征加速更显著
5. **首次调用存在 JIT 开销**：基准测试应在后续调用进行
6. **使用 RMM 内存池**：预分配内存池比原始 cudaMalloc 快 1000 倍
7. **使用 dask-ml 调参**：避免 sklearn 的 GridSearchCV 以减少 CPU-GPU 传输
8. **树模型推理用 FIL**：即使模型在 CPU 训练（XGBoost/LightGBM/sklearn RF），FIL 仍可提供 80 倍以上推理加速

---

## 互操作性

- **cuDF**：零拷贝输入，所有估计器直接接受 cuDF 数据框
- **CuPy**：通过 `__cuda_array_interface__` 实现零拷贝，最高效的中间格式
- **NumPy/pandas**：支持输入（自动传输至 GPU），输出类型可配置
- **PyTorch**：通过数组接口接受张量
- **sklearn**：API 兼容，模型可相互转换，cuml.accel 实现透明加速
- **XGBoost/LightGBM**：FIL 为外部训练的树模型提供 GPU 推理
- **Dask**：通过 `cuml.dask` 模块原生支持分布式计算

### 端到端 RAPIDS 流程

```python
import cudf
import cuml
from cuml.preprocessing import StandardScaler
from cuml.ensemble import RandomForestClassifier
from cuml.model_selection import train_test_split

# GPU 加载数据
df = cudf.read_parquet("data.parquet")
X = df.drop("target", axis=1)
y = df["target"]

# 拆分
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# 预处理
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# 训练
model = RandomForestClassifier(n_estimators=100, max_depth=16)
model.fit(X_train, y_train)

# 评估
score = model.score(X_test, y_test)
print(f"准确率: {score:.4f}")
```

从 Parquet 读取到模型评估全程在 GPU 运行 —— 实现零 CPU-GPU 传输。

---

## 与 sklearn 主要差异

1. **平台**：仅支持 Linux 和 WSL2，无原生 macOS 或 Windows 版本
2. **稀疏数据**：多数 cuML 算法不支持稀疏矩阵，cuml.accel 下稀疏输入会回退至 CPU
3. **字符串数据**：必须预编码为数值，估计器不支持原生字符串列
4. **多输出**：随机森林不支持多输出
5. **热启动**：多数算法不支持
6. **忽略部分 sklearn 参数**：`n_jobs`（GPU 处理并行）、`positive=True`、特定求解器选项
7. **数值精度**：结果质量等效但浮点级可能不同，建议比较分数而非原始系数
8. **内存**：受限于 GPU VRAM（通常 8-80GB），超大数据集需用托管内存或 Dask
9. **缺失拟合属性**：部分 sklearn 属性在 cuml.accel 下不计算（如 HDBSCAN 的 `exemplars_`，线性回归的 `rank_`）

---

## 常见迁移模式

### 模式 1：零修改（cuml.accel）

```python
# 在 notebook 顶部添加一行：
%load_ext cuml.accel

from sklearn.cluster import KMeans  # 现为 GPU 加速
from sklearn.decomposition import PCA  # 现为 GPU 加速
# 其余代码完全保持不变
```

### 模式 2：直接替换导入

```python
# 替换前
from sklearn.ensemble import RandomForestClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split

# 替换后
from cuml.ensemble import RandomForestClassifier
from cuml.preprocessing import StandardScaler
from cuml.model_selection import train_test_split
```

### 模式 3：完整 RAPIDS 流程（cuDF + cuML）

```python
import cudf
from cuml.preprocessing import StandardScaler, LabelEncoder
from cuml.ensemble import RandomForestClassifier
from cuml.model_selection import train_test_split

# 全程在 GPU 加载和预处理
df = cudf.read_parquet("data.parquet")
le = LabelEncoder()
df["category_encoded"] = le.fit_transform(df["category"])

X = df[["feature1", "feature2", "category_encoded"]].to_cupy()
y = df["target"].to_cupy()

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

model = RandomForestClassifier(n_estimators=200, max_depth=16)
model.fit(X_train, y_train)
print(f"准确率: {model.score(X_test, y_test):.4f}")
```

### 模式 4：CPU 训练模型的 GPU 推理

```python
from cuml.fil import ForestInference

# 加载 XGBoost/LightGBM/sklearn 模型实现 80 倍以上推理加速
fil_model = ForestInference.load("my_xgboost_model.ubj", is_classifier=True)
predictions = fil_model.predict(X_test)
```
