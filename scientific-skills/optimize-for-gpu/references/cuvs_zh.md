# cuVS 参考文档

cuVS 是 NVIDIA 推出的 GPU 加速向量搜索与聚类库，属于 RAPIDS 生态系统。它提供最先进的近似最近邻（ANN）搜索算法 GPU 实现，相比基于 CPU 的库（如 Faiss CPU 模式、Annoy 和 scikit-learn 的 NearestNeighbors），在高维向量搜索中可实现数量级的速度提升。

> **完整文档：** https://docs.rapids.ai/api/cuvs/stable/

## 目录

1. [安装与设置](#安装与设置)
2. [适用场景](#适用场景)
3. [索引选择指南](#索引选择指南)
4. [CAGRA — 基于图的索引](#cagra)
5. [IVF-Flat — 倒排文件索引](#ivf-flat)
6. [IVF-PQ — 压缩倒排文件索引](#ivf-pq)
7. [暴力搜索 — 精确搜索](#暴力搜索)
8. [HNSW — 基于 GPU 索引的 CPU 搜索](#hnsw)
9. [距离度量](#距离度量)
10. [结果过滤](#结果过滤)
11. [多 GPU 支持](#多-gpu-支持)
12. [内存与性能](#内存与性能)
13. [互操作性](#互操作性)
14. [常用模式](#常用模式)

---

## 安装与设置

在所有安装说明、文档字符串、注释和错误信息中，始终使用 `uv add`（切勿使用 `pip install` 或 `conda install`）。

```bash
uv add --extra-index-url=https://pypi.nvidia.com cuvs-cu12   # 适用于 CUDA 12.x
```

**平台：** 仅限 Linux 和 WSL2（不支持原生 macOS 或 Windows）。  
**要求：** 支持 CUDA 12.x 的 NVIDIA GPU，推荐安装 CuPy 处理 GPU 数组。

验证安装：
```python
from cuvs.neighbors import cagra
import cupy as cp

dataset = cp.random.rand(1000, 128, dtype=cp.float32)
index = cagra.build(cagra.IndexParams(), dataset)
print("cuVS 运行正常 — 已构建 CAGRA 索引")
```

---

## 适用场景

cuVS 适用于以下需求：
- 高维向量（嵌入）的**最近邻搜索**
- 面向 RAG、推荐系统、图像/文本/音频检索的**相似性搜索**
- 用于聚类或可视化流程的**k-NN 图构建**
- **向量数据库后端** — cuVS 为 Milvus、Lucene、Kinetica 和 Faiss GPU 提供搜索支持
- **替代 Faiss、Annoy、ScaNN 或 sklearn NearestNeighbors** 并实现 GPU 加速

cuVS 不适用于：
- 通用机器学习（请改用 cuML）
- 低维数据（< ~16 维）且数据集较小（< 10K 向量）
- 无 GPU 的纯 CPU 环境

---

## 索引选择指南

| 索引 | 最佳场景 | 构建速度 | 搜索速度 | 内存占用 | 准确度 |
|-------|----------|-------------|--------------|--------|----------|
| **CAGRA** | 默认选择 — 构建与搜索均快 | 快 | 最快 | 中等 | 高 |
| **IVF-Flat** | 需要精确距离的场景 | 中等 | 快 | 高（存储完整向量） | 极高 |
| **IVF-PQ** | GPU 内存不足的大型数据集 | 中等 | 快 | 低（压缩存储） | 良好 |
| **暴力搜索** | 小型数据集或生成基准真值 | 不适用 | 大规模时慢 | 高 | 精确 |
| **HNSW** | 基于 GPU 构建索引的 CPU 端搜索 | 慢 | 快（CPU） | 中等 | 高 |

**优先选择 CAGRA**，除非有特殊原因。它是速度最快的 GPU 原生算法，适用于多数场景。内存紧张时选用 IVF-PQ，需要更高准确度时选用 IVF-Flat，小型数据集或验证场景使用暴力搜索。

---

## CAGRA

CAGRA（CUDA 加速图算法）是专为 GPU 优化的基于图的 ANN 索引，在多数工作负载中速度最快。

### 构建

```python
import cupy as cp
from cuvs.neighbors import cagra

n_samples = 1_000_000
n_features = 128
dataset = cp.random.rand(n_samples, n_features, dtype=cp.float32)

# 默认参数适用于多数场景
index_params = cagra.IndexParams(
    metric="sqeuclidean",             # "sqeuclidean", "inner_product", "cosine"
    intermediate_graph_degree=128,     # 值越高质量越好，构建越慢
    graph_degree=64,                   # 最终图度数（值越低内存占用越少）
    build_algo="ivf_pq",              # "ivf_pq", "nn_descent" 或 "ace"
)

index = cagra.build(index_params, dataset)
```

### 搜索

```python
from cuvs.common import Resources

queries = cp.random.rand(1000, n_features, dtype=cp.float32)

search_params = cagra.SearchParams(
    itopk_size=64,       # 中间 top-k 值（值越高越精确，速度越慢）
    search_width=1,      # 每次迭代的起始节点数
    max_iterations=0,    # 0 表示自动选择
    algo="auto",         # "auto", "single_cta", "multi_cta", "multi_kernel"
)

resources = Resources()
distances, neighbors = cagra.search(
    search_params, index, queries, k=10, resources=resources
)
resources.sync()

# distances: 形状 (1000, 10) — 平方欧氏距离
# neighbors: 形状 (1000, 10) — 原始数据集中的索引
```

### 保存/加载

```python
cagra.save("my_index.cagra", index)
loaded_index = cagra.load("my_index.cagra")
```

### 扩展

```python
new_data = cp.random.rand(10_000, n_features, dtype=cp.float32)
extended_index = cagra.extend(cagra.ExtendParams(), index, new_data)
```

### 压缩支持（大型数据集）

```python
from cuvs.neighbors.cagra import CompressionParams

index_params = cagra.IndexParams(
    compression=CompressionParams(
        pq_bits=8,
        pq_dim=64,
    )
)
index = cagra.build(index_params, dataset)
```

---

## IVF-Flat

IVF-Flat 将数据集分区为簇（倒排文件）并存储完整向量。比 IVF-PQ 准确度更高，但内存占用更大。

### 构建

```python
from cuvs.neighbors import ivf_flat

build_params = ivf_flat.IndexParams(
    n_lists=1024,                    # 簇数量（建议从 sqrt(n_samples) 开始）
    metric="sqeuclidean",            # "sqeuclidean", "euclidean", "inner_product", "cosine"
    kmeans_trainset_fraction=0.5,    # 用于 k-means 训练的数据比例
    kmeans_n_iters=20,               # K-means 迭代次数
    add_data_on_build=True,          # 构建时添加向量（而非后续扩展）
)

index = ivf_flat.build(build_params, dataset)
```

### 搜索

```python
search_params = ivf_flat.SearchParams(
    n_probes=50,    # 搜索的簇数量（值越高越精确，速度越慢）
)

distances, neighbors = ivf_flat.search(
    search_params, index, queries, k=10
)
```

### 保存/加载/扩展

```python
ivf_flat.save("my_index.ivf_flat", index)
loaded_index = ivf_flat.load("my_index.ivf_flat")

# 扩展新数据
import numpy as np
new_vectors = cp.random.rand(5000, n_features, dtype=cp.float32)
new_indices = cp.arange(n_samples, n_samples + 5000, dtype=cp.int64)
ivf_flat.extend(index, new_vectors, new_indices)
```

---

## IVF-PQ

IVF-PQ 通过乘积量化压缩向量，显著降低内存占用。最适合无法在 GPU 内存中完整存储的大型数据集。

### 构建

```python
from cuvs.neighbors import ivf_pq

build_params = ivf_pq.IndexParams(
    n_lists=1024,                # 簇数量
    metric="sqeuclidean",        # "sqeuclidean", "inner_product"
    pq_bits=8,                   # 子量化器位数（4 或 8）
    pq_dim=0,                    # PQ 维度（0 表示自动，通常为 dim/4）
    codebook_kind="subspace",    # "subspace" 或 "cluster"
    kmeans_n_iters=20,
    add_data_on_build=True,
)

index = ivf_pq.build(build_params, dataset)
```

### 搜索

```python
search_params = ivf_pq.SearchParams(
    n_probes=50,                     # 搜索的簇数量
    lut_dtype="float32",             # 查找表精度
    internal_distance_dtype="float32",
)

distances, neighbors = ivf_pq.search(
    search_params, index, queries, k=10
)
```

### 保存/加载/扩展

```python
ivf_pq.save("my_index.ivf_pq", index)
loaded_index = ivf_pq.load("my_index.ivf_pq")

# 扩展
new_vectors = cp.random.rand(5000, n_features, dtype=cp.float32)
new_indices = cp.arange(n_samples, n_samples + 5000, dtype=cp.int64)
ivf_pq.extend(index, new_vectors, new_indices)
```

---

## 暴力搜索

精确 k-NN 搜索 — 计算所有距离。适用于小型数据集（< 50K 向量）或生成基准真值以评估近似索引。

```python
from cuvs.neighbors import brute_force

# 构建（仅存储数据集）
index = brute_force.build(dataset, metric="sqeuclidean")

# 搜索
distances, neighbors = brute_force.search(index, queries, k=10)

# 保存/加载
brute_force.save("bf_index.bin", index)
loaded = brute_force.load("bf_index.bin")
```

---

## HNSW

cuVS 提供 HNSW 实现用于 CPU 端搜索。典型工作流：在 GPU 构建 CAGRA 索引，转换为 HNSW 用于 CPU 服务。利用 GPU 加速构建，同时在 CPU 执行搜索（适用于查询时无 GPU 的场景）。

```python
from cuvs.neighbors import cagra, hnsw
import numpy as np

# 在 GPU 构建 CAGRA
dataset_gpu = cp.random.rand(100_000, 128, dtype=cp.float32)
cagra_index = cagra.build(cagra.IndexParams(), dataset_gpu)

# 转换为 HNSW 用于 CPU 搜索
hnsw_index = hnsw.from_cagra(hnsw.IndexParams(), cagra_index)

# 使用 numpy 查询在 CPU 搜索
queries_cpu = np.random.rand(100, 128).astype(np.float32)
search_params = hnsw.SearchParams(
    ef=200,           # 搜索深度（值越高越精确，速度越慢）
    num_threads=0,    # 0 表示自动（使用所有可用线程）
)
distances, neighbors = hnsw.search(search_params, hnsw_index, queries_cpu, k=10)

# 保存/加载
hnsw.save("my_index.hnsw", hnsw_index)
loaded = hnsw.load(hnsw.IndexParams(), "my_index.hnsw", dim=128,
                    dtype=np.float32, metric="sqeuclidean")
```

### 可扩展 HNSW

构建后添加向量需设置 `hierarchy="cpu"`：

```python
hnsw_index = hnsw.from_cagra(hnsw.IndexParams(hierarchy="cpu"), cagra_index)

new_data = np.random.rand(5000, 128).astype(np.float32)
hnsw.extend(hnsw.ExtendParams(), hnsw_index, new_data)
```

---

## 距离度量

| 度量标准 | 字符串标识 | 说明 |
|--------|--------|-------|
| 平方欧氏距离 | `"sqeuclidean"` | 默认选项。最快 — 避免开方运算。 |
| 欧氏距离 | `"euclidean"` | L2 距离 |
| 内积 | `"inner_product"` | 适用于归一化嵌入（通过点积计算余弦相似度） |
| 余弦相似度 | `"cosine"` | CAGRA 和 IVF-Flat 支持 |

使用 IVF-PQ 计算余弦相似度时，需将向量归一化为单位长度并使用 `"inner_product"`。

---

## 结果过滤

cuVS 支持使用位图或位集进行预过滤，排除特定向量。

```python
from cuvs.neighbors import brute_force
import cupy as cp

# 位集过滤：从所有查询中排除指定索引
# 1 = 排除，0 = 包含
n_samples = 100_000
bitset = cp.zeros(n_samples, dtype=cp.uint8)
bitset[0:1000] = 1  # 排除前 1000 个向量

distances, neighbors = brute_force.search(
    index, queries, k=10, prefilter=bitset
)
```

CAGRA 也支持通过 `cagra.search()` 的 `filter` 参数进行过滤。

---

## 多 GPU 支持

对于单 GPU 无法容纳的大型数据集，使用多 GPU API：

```python
from cuvs.neighbors.mg import cagra as mg_cagra

# 在所有可用 GPU 上构建
build_params = mg_cagra.IndexParams(
    intermediate_graph_degree=64,
    graph_degree=32,
)
index = mg_cagra.build(build_params, dataset)

# 跨 GPU 搜索
search_params = mg_cagra.SearchParams()
distances, neighbors = mg_cagra.search(search_params, index, queries, k=10)
```

通过 `cuvs.neighbors.mg` 也可为 IVF-Flat 和 IVF-PQ 提供多 GPU 支持。

---

## 内存与性能

### 支持的数据类型

所有索引类型均支持：`float32`、`float16`、`int8`、`uint8`。

当不需要完整 float32 精度时（常见于嵌入场景），使用 `float16` 可减半内存占用并加速构建和搜索。

### 性能优化建议

1. **使用 CuPy 数组输入**。NumPy 数组可用但会触发 CPU-GPU 传输。若向量已在 GPU（来自模型或流程），请直接传递。

2. **调整搜索参数而非仅构建参数**。准确度/速度的最大权衡在搜索阶段：
   - CAGRA：增加 `itopk_size`（默认 64）
   - IVF-Flat/IVF-PQ：增加 `n_probes`（默认 20）
   - HNSW：增加 `ef`（默认 200）

3. **嵌入向量使用 float16**。多数嵌入模型输出 float32，但相似性搜索很少需要额外精度。转换为 float16 可提升一倍吞吐量。

4. **IVF 索引的 n_lists 调优**。建议起始值 `sqrt(n_samples)`。列表过少导致搜索慢，过多则召回率低。

5. **批量查询**。GPU 吞吐量随批量增大而提升。单次搜索 1000 个查询比 1000 次独立搜索高效得多。

6. **复用 Resources 句柄**。创建单个 `Resources()` 对象并传递给所有构建/搜索调用 — 它管理 CUDA 流和内存。

### 内存估算

- **暴力搜索：** `n_samples * dim * dtype_size`（完整数据集）
- **IVF-Flat：** 近似暴力搜索 + 簇开销
- **IVF-PQ：** `n_samples * pq_dim * pq_bits / 8`（高度压缩）
- **CAGRA：** `n_samples * (dim * dtype_size + graph_degree * 4)`（数据集 + 图结构）

---

## 互操作性

- **CuPy：** 原生支持 — 通过 `__cuda_array_interface__` 零拷贝传输
- **NumPy：** 可作为输入（GPU 索引自动传输至 GPU，HNSW 直接使用）
- **PyTorch/TensorFlow：** 通过 CUDA 数组接口支持张量 — 无需复制
- **cuDF：** 通过 `.values` 将列转换为 CuPy 后传入 cuVS
- **Faiss：** cuVS 是 Faiss GPU 的底层引擎；直接使用 cuVS 可获更高控制权
- **向量数据库：** cuVS 已集成至 Milvus、Luc

```markdown
search_params = cagra.SearchParams(itopk_size=128)
distances, neighbors = cagra.search(search_params, index, query_embedding, k=20)

# neighbors[0] 包含前20个最相似文档的索引
top_doc_ids = neighbors[0].get()  # 传输到CPU
```

---

## 常见模式

### 模式1：快速近似最近邻搜索 (CAGRA)

```python
import cupy as cp
from cuvs.neighbors import cagra

dataset = cp.random.rand(500_000, 128, dtype=cp.float32)
queries = cp.random.rand(1000, 128, dtype=cp.float32)

index = cagra.build(cagra.IndexParams(), dataset)
distances, neighbors = cagra.search(cagra.SearchParams(), index, queries, k=10)
```

### 模式2：内存高效搜索 (IVF-PQ)

```python
import cupy as cp
from cuvs.neighbors import ivf_pq

dataset = cp.random.rand(10_000_000, 256, dtype=cp.float32)

# PQ压缩向量 —— 内存使用量比暴力搜索减少约32倍
params = ivf_pq.IndexParams(n_lists=4096, pq_bits=8, pq_dim=64)
index = ivf_pq.build(params, dataset)

search_params = ivf_pq.SearchParams(n_probes=100)
distances, neighbors = ivf_pq.search(search_params, index, queries, k=10)
```

### 模式3：GPU构建，CPU服务 (CAGRA → HNSW)

```python
import cupy as cp
import numpy as np
from cuvs.neighbors import cagra, hnsw

# 在GPU上构建（快速）
dataset = cp.random.rand(1_000_000, 128, dtype=cp.float32)
gpu_index = cagra.build(cagra.IndexParams(), dataset)

# 转换为HNSW用于CPU服务
cpu_index = hnsw.from_cagra(hnsw.IndexParams(), gpu_index)
hnsw.save("serving_index.hnsw", cpu_index)

# 服务阶段（无需GPU）
loaded = hnsw.load(hnsw.IndexParams(), "serving_index.hnsw",
                    dim=128, dtype=np.float32)
queries = np.random.rand(100, 128).astype(np.float32)
distances, neighbors = hnsw.search(
    hnsw.SearchParams(ef=200), loaded, queries, k=10
)
```

### 模式4：通过暴力搜索验证

```python
from cuvs.neighbors import brute_force, cagra

# 基准真值
bf_index = brute_force.build(dataset)
gt_distances, gt_neighbors = brute_force.search(bf_index, queries, k=10)

# 近似搜索
cagra_index = cagra.build(cagra.IndexParams(), dataset)
approx_distances, approx_neighbors = cagra.search(
    cagra.SearchParams(), cagra_index, queries, k=10
)

# 计算召回率
recall = sum(
    len(set(gt_neighbors[i].get()) & set(approx_neighbors[i].get())) / 10
    for i in range(len(queries))
) / len(queries)
print(f"Recall@10: {recall:.4f}")
```

### 模式5：余弦相似度搜索

```python
import cupy as cp
from cuvs.neighbors import cagra

# 将嵌入向量归一化为单位长度
embeddings = cp.random.rand(100_000, 768, dtype=cp.float32)
norms = cp.linalg.norm(embeddings, axis=1, keepdims=True)
embeddings_normalized = embeddings / norms

# 在归一化向量上使用内积 = 余弦相似度
index = cagra.build(
    cagra.IndexParams(metric="inner_product"),
    embeddings_normalized,
)

query = cp.random.rand(1, 768, dtype=cp.float32)
query_normalized = query / cp.linalg.norm(query)

distances, neighbors = cagra.search(
    cagra.SearchParams(), index, query_normalized, k=10
)
```

---

## 超越近邻搜索：聚类、距离计算与预处理

cuVS 还提供GPU加速的聚类、成对距离计算和量化功能——这些是向量搜索流水线中的重要构建模块。

### K均值聚类

```python
import cupy as cp
from cuvs.cluster.kmeans import fit, predict, KMeansParams

X = cp.random.rand(100_000, 128, dtype=cp.float32)

params = KMeansParams(
    n_clusters=256,
    init_method="KMeansPlusPlus",   # 或 "Random", "Array"
    max_iter=300,
    tol=1e-4,
)
centroids, inertia, n_iter = fit(params, X)
labels, inertia = predict(params, X, centroids)
```

对于超出GPU内存的数据集，使用`streaming_batch_size`传递NumPy数组：

```python
import numpy as np
from cuvs.cluster.kmeans import fit, KMeansParams

X_host = np.random.rand(10_000_000, 128).astype(np.float32)
params = KMeansParams(n_clusters=1000, streaming_batch_size=1_000_000)
centroids, inertia, n_iter = fit(params, X_host)
```

### 成对距离计算

```python
from cuvs.distance import pairwise_distance

# 支持: euclidean, l2, l1, inner_product, cosine, chebyshev,
# canberra, hellinger, jensenshannon, kl_divergence, correlation, minkowski
output = pairwise_distance(X, Y, metric="euclidean")
```

### 量化（预处理）

量化在索引前压缩向量，减少内存占用并通常提升搜索吞吐量。

**标量量化** (float32 → int8):
```python
from cuvs.preprocessing.quantize import scalar

params = scalar.QuantizerParams(quantile=0.99)
quantizer = scalar.train(params, dataset)
transformed = scalar.transform(quantizer, dataset)       # int8
reconstructed = scalar.inverse_transform(quantizer, transformed)
```

**二值量化** (float32 → uint8位压缩):
```python
from cuvs.preprocessing.quantize import binary

transformed = binary.transform(dataset)  # uint8
# 配合 metric="bitwise_hamming" 使用
```

**乘积量化**:
```python
from cuvs.preprocessing.quantize import pq

params = pq.QuantizerParams(pq_bits=8, pq_dim=16)
quantizer = pq.build(params, dataset)
transformed, _ = pq.transform(quantizer, dataset)        # uint8
reconstructed = pq.inverse_transform(quantizer, transformed)
```

### NN-Descent (k-NN图构建)

构建全连接k近邻图——可作为UMAP、t-SNE或基于图的聚类算法的输入。

```python
import cupy as cp
from cuvs.neighbors import nn_descent

dataset = cp.random.rand(100_000, 128, dtype=cp.float32)

build_params = nn_descent.IndexParams(
    metric="sqeuclidean",
    graph_degree=64,
    intermediate_graph_degree=96,   # >= 1.5 * graph_degree
    max_iterations=20,
)
index = nn_descent.build(build_params, dataset)
graph = index.graph  # (n_samples, graph_degree) —— k近邻图
```
