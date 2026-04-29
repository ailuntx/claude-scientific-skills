# 相似性搜索

Aeon 提供了在时间序列内部及跨时间序列查找相似模式的工具，包括子序列搜索、模体发现和近似最近邻。

## 子序列最近邻 (SNN)

在单个时间序列中查找最相似的子序列。

### MASS 算法
- `MassSNN` - Mueen 相似性搜索算法
  - 快速归一化互相关计算相似度
  - 高效计算距离轮廓
  - **使用场景**：需要精确最近邻距离、处理大型序列时

### 基于 STOMP 的模体发现
- `StompMotif` - 发现重复模式（模体）
  - 查找前 k 个最相似的子序列对
  - 基于矩阵轮廓计算
  - **使用场景**：需要发现重复模式时

### 暴力搜索基线
- `DummySNN` - 穷举距离计算
  - 计算所有成对距离
  - **使用场景**：小型序列、需要精确基准时

## 集合级搜索

跨时间序列集合查找相似序列。

### 近似最近邻 (ANN)
- `RandomProjectionIndexANN` - 局部敏感哈希
  - 使用余弦相似度的随机投影
  - 构建索引实现快速近似搜索
  - **使用场景**：大型集合、速度优先于精确度时

## 快速入门：模体发现

```python
from aeon.similarity_search import StompMotif
import numpy as np

# 创建带有重复模式的时间序列
pattern = np.sin(np.linspace(0, 2*np.pi, 50))
y = np.concatenate([
    pattern + np.random.normal(0, 0.1, 50),
    np.random.normal(0, 1, 100),
    pattern + np.random.normal(0, 0.1, 50),
    np.random.normal(0, 1, 100)
])

# 查找前 3 个模体
motif_finder = StompMotif(window_size=50, k=3)
motifs = motif_finder.fit_predict(y)

# motifs 包含模体出现位置的索引
for i, (idx1, idx2) in enumerate(motifs):
    print(f"模体 {i+1} 出现在位置 {idx1} 和 {idx2}")
```

## 快速入门：子序列搜索

```python
from aeon.similarity_search import MassSNN
import numpy as np

# 待搜索的时间序列
y = np.sin(np.linspace(0, 20, 500))

# 查询子序列
query = np.sin(np.linspace(0, 2, 50))

# 查找最近子序列
searcher = MassSNN()
distances = searcher.fit_transform(y, query)

# 查找最佳匹配
best_match_idx = np.argmin(distances)
print(f"最佳匹配位置索引 {best_match_idx}")
```

## 快速入门：集合上的近似最近邻

```python
from aeon.similarity_search import RandomProjectionIndexANN
from aeon.datasets import load_classification

# 加载时间序列集合
X_train, _ = load_classification("GunPoint", split="train")

# 构建索引
ann = RandomProjectionIndexANN(n_projections=8, n_bits=4)
ann.fit(X_train)

# 查找近似最近邻
query = X_train[0]
neighbors, distances = ann.kneighbors(query, k=5)
```

## 矩阵轮廓

矩阵轮廓是多种相似性搜索任务的基础数据结构：

- **距离轮廓**：查询到所有子序列的距离
- **矩阵轮廓**：每个子序列到其他子序列的最小距离
- **模体**：具有最小距离的子序列对
- **异常点**：具有最大最小距离的子序列（异常）

```python
from aeon.similarity_search import StompMotif

# 计算矩阵轮廓并查找模体/异常点
mp = StompMotif(window_size=50)
mp.fit(y)

# 访问矩阵轮廓
profile = mp.matrix_profile_
profile_indices = mp.matrix_profile_index_

# 查找异常点（异常）
discord_idx = np.argmax(profile)
```

## 算法选择

- **精确子序列搜索**：MassSNN
- **模体发现**：StompMotif
- **异常检测**：矩阵轮廓（参见 anomaly_detection.md）
- **快速近似搜索**：RandomProjectionIndexANN
- **小型数据**：DummySNN（获取精确结果）

## 使用案例

### 模式匹配
在长序列中查找模式出现位置：

```python
# 在 ECG 数据中查找心跳模式
searcher = MassSNN()
distances = searcher.fit_transform(ecg_data, heartbeat_pattern)
occurrences = np.where(distances < threshold)[0]
```

### 模体发现
识别重复模式：

```python
# 查找重复的行为模式
motif_finder = StompMotif(window_size=100, k=5)
motifs = motif_finder.fit_predict(activity_data)
```

### 时间序列检索
在数据库中查找相似时间序列：

```python
# 构建可搜索索引
ann = RandomProjectionIndexANN()
ann.fit(time_series_database)

# 查询相似序列
neighbors = ann.kneighbors(query_series, k=10)
```

## 最佳实践

1. **窗口大小**：子序列方法的关键参数
   - 过小：捕获噪声
   - 过大：遗漏细粒度模式
   - 经验法则：序列长度的 10-20%

2. **归一化**：多数方法假设 z-归一化子序列
   - 处理振幅变化
   - 聚焦形状相似性

3. **距离度量**：不同需求选用不同度量
   - 欧氏距离：快速、基于形状
   - 动态时间规整 (DTW)：处理时间扭曲
   - 余弦相似度：尺度不变

4. **排除区域**：模体发现中排除平凡匹配
   - 通常设为窗口大小的 0.5-1.0 倍
   - 避免找到重叠出现

5. **性能**：
   - MASS 复杂度 O(n log n)，暴力搜索 O(n²)
   - ANN 以精度换取速度
   - 部分方法支持 GPU 加速
