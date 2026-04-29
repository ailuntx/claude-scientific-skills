# 距离度量

Aeon 提供了专门用于衡量时间序列相似性的距离函数，兼容 aeon 和 scikit-learn 评估器。

## 距离分类

### 弹性距离

支持序列间的灵活时间对齐：

**动态时间规整系列：**
- `dtw` - 经典动态时间规整
- `ddtw` - 导数动态时间规整（比较导数）
- `wdtw` - 加权动态时间规整（按位置惩罚规整）
- `wddtw` - 加权导数动态时间规整
- `shape_dtw` - 基于形状的动态时间规整

**基于编辑的距离：**
- `erp` - 带实际惩罚的编辑距离
- `edr` - 实数序列编辑距离
- `lcss` - 最长公共子序列
- `twe` - 时间规整编辑距离

**专用距离：**
- `msm` - 移动-分割-合并距离
- `adtw` - 罚金动态时间规整
- `sbd` - 基于形状的距离

**适用场景**：时间序列可能存在时间偏移、速度变化或相位差异时。

### 锁步距离

无需对齐，逐点比较时间序列：

- `euclidean` - 欧氏距离（L2范数）
- `manhattan` - 曼哈顿距离（L1范数）
- `minkowski` - 广义闵可夫斯基距离（Lp范数）
- `squared` - 平方欧氏距离

**适用场景**：序列已对齐、需要计算速度或预期无时间规整时。

## 使用模式

### 计算单点距离

```python
from aeon.distances import dtw_distance

# 两个时间序列间的距离
distance = dtw_distance(x, y)

# 带窗口约束（Sakoe-Chiba带）
distance = dtw_distance(x, y, window=0.1)
```

### 成对距离矩阵

```python
from aeon.distances import dtw_pairwise_distance

# 集合内所有成对距离
X = [series1, series2, series3, series4]
distance_matrix = dtw_pairwise_distance(X)

# 跨集合距离
distance_matrix = dtw_pairwise_distance(X_train, X_test)
```

### 成本矩阵与对齐路径

```python
from aeon.distances import dtw_cost_matrix, dtw_alignment_path

# 获取完整成本矩阵
cost_matrix = dtw_cost_matrix(x, y)

# 获取最优对齐路径
path = dtw_alignment_path(x, y)
# 返回索引：[(0,0), (1,1), (2,1), (2,2), ...]
```

### 与评估器配合使用

```python
from aeon.classification.distance_based import KNeighborsTimeSeriesClassifier

# 在分类器中使用DTW距离
clf = KNeighborsTimeSeriesClassifier(
    n_neighbors=5,
    distance="dtw",
    distance_params={"window": 0.2}
)
clf.fit(X_train, y_train)
```

## 距离参数

### 窗口约束

限制规整路径偏差（提升速度并防止病态规整）：

```python
# Sakoe-Chiba带：窗口设为序列长度比例
dtw_distance(x, y, window=0.1)  # 允许10%偏差

# Itakura平行四边形：斜率约束路径
dtw_distance(x, y, itakura_max_slope=2.0)
```

### 归一化

控制是否在距离计算前进行z归一化：

```python
# 多数弹性距离支持归一化
distance = dtw_distance(x, y, normalize=True)
```

### 距离特定参数

```python
# ERP：间隙惩罚
distance = erp_distance(x, y, g=0.5)

# TWE：刚度和惩罚参数
distance = twe_distance(x, y, nu=0.001, lmbda=1.0)

# LCSS：匹配阈值
distance = lcss_distance(x, y, epsilon=0.5)
```

## 算法选择

### 按使用场景：

**时间错位**：DTW, DDTW, WDTW  
**速度变化**：带窗口约束的DTW  
**形状相似性**：Shape DTW, SBD  
**编辑操作**：ERP, EDR, LCSS  
**导数匹配**：DDTW  
**计算速度**：欧氏距离, 曼哈顿距离  
**异常值鲁棒性**：曼哈顿距离, LCSS  

### 按计算成本：

**最快**：欧氏距离 (O(n))  
**快速**：约束DTW (O(nw)，w为窗口)  
**中等**：完整DTW (O(n²))  
**较慢**：复杂弹性距离 (ERP, TWE, MSM)  

## 速查表

| 距离 | 对齐方式 | 速度 | 鲁棒性 | 可解释性 |
|------|----------|------|--------|----------|
| 欧氏距离 | 锁步 | 极快 | 低 | 高 |
| DTW | 弹性 | 中等 | 中等 | 中等 |
| DDTW | 弹性 | 中等 | 高 | 中等 |
| WDTW | 弹性 | 中等 | 中等 | 中等 |
| ERP | 基于编辑 | 慢 | 高 | 低 |
| LCSS | 基于编辑 | 慢 | 极高 | 低 |
| Shape DTW | 弹性 | 中等 | 中等 | 高 |

## 最佳实践

### 1. 归一化

多数距离对尺度敏感，适时归一化：

```python
from aeon.transformations.collection import Normalizer

normalizer = Normalizer()
X_normalized = normalizer.fit_transform(X)
```

### 2. 窗口约束

对DTW变体使用窗口约束以提升速度和泛化能力：

```python
# 初始使用10-20%窗口
distance = dtw_distance(x, y, window=0.1)
```

### 3. 序列长度

- 需等长：多数锁步距离
- 支持不等长：弹性距离（DTW, ERP等）

### 4. 多变量序列

多数距离支持多变量时间序列：

```python
# x.shape = (n_channels, n_timepoints)
distance = dtw_distance(x_multivariate, y_multivariate)
```

### 5. 性能优化

- 使用numba编译实现（aeon默认）
- 若无需对齐则考虑锁步距离
- 使用窗口化DTW替代完整DTW
- 预计算距离矩阵供重复使用

### 6. 选择合适距离

```python
# 快速决策树：
if 序列已对齐:
    使用距离 = "euclidean"
elif 需要速度:
    使用距离 = "dtw"  # 带窗口约束
elif 预期时间偏移:
    使用距离 = "dtw" 或 "shape_dtw"
elif 存在异常值:
    使用距离 = "lcss" 或 "manhattan"
elif 需考虑导数:
    使用距离 = "ddtw" 或 "wddtw"
```

## 与scikit-learn集成

Aeon距离可与sklearn评估器协同工作：

```python
from sklearn.neighbors import KNeighborsClassifier
from aeon.distances import dtw_pairwise_distance

# 预计算距离矩阵
X_train_distances = dtw_pairwise_distance(X_train)

# 在sklearn中使用
clf = KNeighborsClassifier(metric='precomputed')
clf.fit(X_train_distances, y_train)
```

## 可用距离函数

获取所有可用距离列表：

```python
from aeon.distances import get_distance_function_names

print(get_distance_function_names())
# ['dtw', 'ddtw', 'wdtw', 'euclidean', 'erp', 'edr', ...]
```

获取特定距离函数：

```python
from aeon.distances import get_distance_function

distance_func = get_distance_function("dtw")
result = distance_func(x, y, window=0.1)
```
