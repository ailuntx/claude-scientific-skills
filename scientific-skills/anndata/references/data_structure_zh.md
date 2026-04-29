# AnnData 对象结构

AnnData 对象存储带有相关注释的数据矩阵，为管理实验数据和元数据提供了灵活的框架。

## 核心组件

### X（数据矩阵）
主数据矩阵，形状为 (n_obs, n_vars)，存储实验测量值。

```python
import anndata as ad
import numpy as np

# 使用密集数组创建
adata = ad.AnnData(X=np.random.rand(100, 2000))

# 使用稀疏矩阵创建（推荐用于大型稀疏数据）
from scipy.sparse import csr_matrix
sparse_data = csr_matrix(np.random.rand(100, 2000))
adata = ad.AnnData(X=sparse_data)
```

访问数据：
```python
# 完整矩阵（大型数据集需谨慎）
full_data = adata.X

# 单个观测样本
obs_data = adata.X[0, :]

# 所有观测样本中的单个变量
var_data = adata.X[:, 0]
```

### obs（观测样本注释）
存储观测样本元数据的 DataFrame。每行对应 X 中的一个观测样本。

```python
import pandas as pd

# 创建带观测元数据的 AnnData
obs_df = pd.DataFrame({
    'cell_type': ['T细胞', 'B细胞', '单核细胞'],
    'treatment': ['对照组', '处理组', '对照组'],
    'timepoint': [0, 24, 24]
}, index=['cell_1', 'cell_2', 'cell_3'])

adata = ad.AnnData(X=np.random.rand(3, 100), obs=obs_df)

# 访问观测元数据
print(adata.obs['cell_type'])
print(adata.obs.loc['cell_1'])
```

### var（变量注释）
存储变量元数据的 DataFrame。每行对应 X 中的一个变量。

```python
# 创建带变量元数据的 AnnData
var_df = pd.DataFrame({
    'gene_name': ['ACTB', 'GAPDH', 'TP53'],
    'chromosome': ['7', '12', '17'],
    'highly_variable': [True, False, True]
}, index=['ENSG00001', 'ENSG00002', 'ENSG00003'])

adata = ad.AnnData(X=np.random.rand(100, 3), var=var_df)

# 访问变量元数据
print(adata.var['gene_name'])
print(adata.var.loc['ENSG00001'])
```

### layers（替代数据表示）
存储与 X 维度相同的替代矩阵的字典。

```python
# 存储原始计数、标准化数据和缩放数据
adata = ad.AnnData(X=np.random.rand(100, 2000))
adata.layers['raw_counts'] = np.random.randint(0, 100, (100, 2000))
adata.layers['normalized'] = adata.X / np.sum(adata.X, axis=1, keepdims=True)
adata.layers['scaled'] = (adata.X - adata.X.mean()) / adata.X.std()

# 访问各层数据
raw_data = adata.layers['raw_counts']
normalized_data = adata.layers['normalized']
```

常用层用途：
- `raw_counts`: 标准化前的原始计数数据
- `normalized`: 对数标准化或 TPM 值
- `scaled`: 用于分析的 Z 分数标准化值
- `imputed`: 插补后的数据

### obsm（多维观测样本注释）
存储与观测样本对齐的多维数组的字典。

```python
# 存储 PCA 坐标和 UMAP 嵌入
adata.obsm['X_pca'] = np.random.rand(100, 50)  # 50个主成分
adata.obsm['X_umap'] = np.random.rand(100, 2)  # 2D UMAP坐标
adata.obsm['X_tsne'] = np.random.rand(100, 2)  # 2D t-SNE坐标

# 访问嵌入坐标
pca_coords = adata.obsm['X_pca']
umap_coords = adata.obsm['X_umap']
```

常用 obsm 用途：
- `X_pca`: 主成分坐标
- `X_umap`: UMAP 嵌入坐标
- `X_tsne`: t-SNE 嵌入坐标
- `X_diffmap`: 扩散映射坐标
- `protein_expression`: 蛋白质丰度测量值（CITE-seq）

### varm（多维变量注释）
存储与变量对齐的多维数组的字典。

```python
# 存储 PCA 载荷
adata.varm['PCs'] = np.random.rand(2000, 50)  # 50个成分的载荷
adata.varm['gene_modules'] = np.random.rand(2000, 10)  # 基因模块评分

# 访问载荷
pc_loadings = adata.varm['PCs']
```

常用 varm 用途：
- `PCs`: 主成分载荷
- `gene_modules`: 基因共表达模块分配

### obsp（观测样本间成对关系）
存储表示观测样本间关系的稀疏矩阵的字典。

```python
from scipy.sparse import csr_matrix

# 存储 k-近邻图
n_obs = 100
knn_graph = csr_matrix(np.random.rand(n_obs, n_obs) > 0.95)
adata.obsp['connectivities'] = knn_graph
adata.obsp['distances'] = csr_matrix(np.random.rand(n_obs, n_obs))

# 访问图数据
knn_connections = adata.obsp['connectivities']
distances = adata.obsp['distances']
```

常用 obsp 用途：
- `connectivities`: 细胞-细胞邻域图
- `distances`: 细胞间成对距离

### varp（变量间成对关系）
存储表示变量间关系的稀疏矩阵的字典。

```python
# 存储基因-基因相关矩阵
n_vars = 2000
gene_corr = csr_matrix(np.random.rand(n_vars, n_vars) > 0.99)
adata.varp['correlations'] = gene_corr

# 访问相关性
gene_correlations = adata.varp['correlations']
```

### uns（非结构化注释）
存储任意非结构化元数据的字典。

```python
# 存储分析参数和结果
adata.uns['experiment_date'] = '2025-11-03'
adata.uns['pca'] = {
    'variance_ratio': [0.15, 0.10, 0.08],
    'params': {'n_comps': 50}
}
adata.uns['neighbors'] = {
    'params': {'n_neighbors': 15, 'method': 'umap'},
    'connectivities_key': 'connectivities'
}

# 访问非结构化数据
exp_date = adata.uns['experiment_date']
pca_params = adata.uns['pca']['params']
```

常用 uns 用途：
- 分析参数和设置
- 绘图调色板
- 聚类信息
- 工具特定元数据

### raw（原始数据快照）
在过滤前保留原始数据矩阵和变量注释的可选属性。

```python
# 创建 AnnData 并存储原始状态
adata = ad.AnnData(X=np.random.rand(100, 5000))
adata.var['gene_name'] = [f'Gene_{i}' for i in range(5000)]

# 在过滤前存储原始状态
adata.raw = adata.copy()

# 筛选高变基因
highly_variable_mask = np.random.rand(5000) > 0.5
adata = adata[:, highly_variable_mask]

# 访问原始数据
original_matrix = adata.raw.X
original_var = adata.raw.var
```

## 对象属性

```python
# 维度信息
n_observations = adata.n_obs
n_variables = adata.n_vars
shape = adata.shape  # (n_obs, n_vars)

# 索引信息
obs_names = adata.obs_names  # 观测样本标识符
var_names = adata.var_names  # 变量标识符

# 存储模式
is_view = adata.is_view  # 是否为其他对象的视图
is_backed = adata.isbacked  # 是否基于磁盘存储
filename = adata.filename  # 后备文件路径（若基于磁盘）
```

## 创建 AnnData 对象

### 从数组和 DataFrame 创建
```python
import anndata as ad
import numpy as np
import pandas as pd

# 最小化创建
X = np.random.rand(100, 2000)
adata = ad.AnnData(X)

# 带元数据创建
obs = pd.DataFrame({'cell_type': ['A', 'B'] * 50}, index=[f'cell_{i}' for i in range(100)])
var = pd.DataFrame({'gene_name': [f'Gene_{i}' for i in range(2000)]}, index=[f'ENSG{i:05d}' for i in range(2000)])
adata = ad.AnnData(X=X, obs=obs, var=var)

# 包含所有组件
adata = ad.AnnData(
    X=X,
    obs=obs,
    var=var,
    layers={'raw': np.random.randint(0, 100, (100, 2000))},
    obsm={'X_pca': np.random.rand(100, 50)},
    uns={'experiment': 'test'}
)
```

### 从 DataFrame 创建
```python
# 从 pandas DataFrame 创建（列为基因，行为细胞）
df = pd.DataFrame(
    np.random.rand(100, 50),
    columns=[f'Gene_{i}' for i in range(50)],
    index=[f'Cell_{i}' for i in range(100)]
)
adata = ad.AnnData(df)
```

## 数据访问模式

### 向量提取
```python
# 获取观测注释为数组
cell_types = adata.obs_vector('cell_type')

# 获取跨观测样本的变量值
gene_expression = adata.obs_vector('ACTB')  # 若 ACTB 在 var_names 中

# 获取变量注释为数组
gene_names = adata.var_vector('gene_name')
```

### 子集提取
```python
# 按索引提取
subset = adata[0:10, 0:100]  # 前10个观测样本，前100个变量

# 按名称提取
subset = adata[['cell_1', 'cell_2'], ['ACTB', 'GAPDH']]

# 按布尔掩码提取
high_count_cells = adata.obs['total_counts'] > 1000
subset = adata[high_count_cells, :]

# 按观测元数据提取
t_cells = adata[adata.obs['cell_type'] == 'T细胞']
```

## 内存考量

AnnData 结构专为内存效率设计：
- 稀疏矩阵减少稀疏数据内存占用
- 视图模式避免不必要的数据复制
- 后备模式支持处理大于内存的数据
- 分类注释减少离散值内存占用

```python
# 将字符串转为分类（更节省内存）
adata.obs['cell_type'] = adata.obs['cell_type'].astype('category')
adata.strings_to_categoricals()

# 检查是否为视图（不持有数据）
if adata.is_view:
    adata = adata.copy()  # 创建独立副本
```
