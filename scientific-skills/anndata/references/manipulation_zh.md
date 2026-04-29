# 数据操作

用于转换、子集化和操作 AnnData 对象的操作。

## 子集化

### 按索引
```python
import anndata as ad
import numpy as np

adata = ad.AnnData(X=np.random.rand(1000, 2000))

# 整数索引
subset = adata[0:100, 0:500]  # 前100个观测值，前500个变量

# 索引列表
obs_indices = [0, 10, 20, 30, 40]
var_indices = [0, 1, 2, 3, 4]
subset = adata[obs_indices, var_indices]

# 单个观测值或变量
single_obs = adata[0, :]
single_var = adata[:, 0]
```

### 按名称
```python
import pandas as pd

# 创建带名称索引
obs_names = [f'cell_{i}' for i in range(1000)]
var_names = [f'gene_{i}' for i in range(2000)]
adata = ad.AnnData(
    X=np.random.rand(1000, 2000),
    obs=pd.DataFrame(index=obs_names),
    var=pd.DataFrame(index=var_names)
)

# 按观测名称子集化
subset = adata[['cell_0', 'cell_1', 'cell_2'], :]

# 按变量名称子集化
subset = adata[:, ['gene_0', 'gene_10', 'gene_20']]

# 双轴子集化
subset = adata[['cell_0', 'cell_1'], ['gene_0', 'gene_1']]
```

### 按布尔掩码
```python
# 创建布尔掩码
high_count_obs = np.random.rand(1000) > 0.5
high_var_genes = np.random.rand(2000) > 0.7

# 使用掩码子集化
subset = adata[high_count_obs, :]
subset = adata[:, high_var_genes]
subset = adata[high_count_obs, high_var_genes]
```

### 按元数据条件
```python
# 添加元数据
adata.obs['cell_type'] = np.random.choice(['A', 'B', 'C'], 1000)
adata.obs['quality_score'] = np.random.rand(1000)
adata.var['highly_variable'] = np.random.rand(2000) > 0.8

# 按细胞类型过滤
t_cells = adata[adata.obs['cell_type'] == 'A']

# 按多条件过滤
high_quality_a_cells = adata[
    (adata.obs['cell_type'] == 'A') &
    (adata.obs['quality_score'] > 0.7)
]

# 按变量元数据过滤
hv_genes = adata[:, adata.var['highly_variable']]

# 复杂条件过滤
filtered = adata[
    (adata.obs['quality_score'] > 0.5) &
    (adata.obs['cell_type'].isin(['A', 'B'])),
    adata.var['highly_variable']
]
```

## 转置

```python
# 转置AnnData对象（交换观测值和变量）
adata_T = adata.T

# 形状变化
print(adata.shape)    # (1000, 2000)
print(adata_T.shape)  # (2000, 1000)

# obs和var互换
print(adata.obs.head())   # 观测元数据
print(adata_T.var.head()) # 相同数据，现作为变量元数据

# 当数据方向相反时非常有用
# 某些文件格式中基因行为行时常见
```

## 复制

### 完整复制
```python
# 创建独立副本
adata_copy = adata.copy()

# 修改副本不影响原始数据
adata_copy.obs['new_column'] = 1
print('new_column' in adata.obs.columns)  # False
```

### 浅复制
```python
# 视图（不复制数据，修改影响原始对象）
adata_view = adata[0:100, :]

# 检查是否为视图
print(adata_view.is_view)  # True

# 将视图转为独立副本
adata_independent = adata_view.copy()
print(adata_independent.is_view)  # False
```

## 重命名

### 重命名观测值和变量
```python
# 重命名所有观测值
adata.obs_names = [f'new_cell_{i}' for i in range(adata.n_obs)]

# 重命名所有变量
adata.var_names = [f'new_gene_{i}' for i in range(adata.n_vars)]

# 使名称唯一（为重复项添加后缀）
adata.obs_names_make_unique()
adata.var_names_make_unique()
```

### 重命名类别
```python
# 创建分类列
adata.obs['cell_type'] = pd.Categorical(['A', 'B', 'C'] * 333 + ['A'])

# 重命名类别
adata.rename_categories('cell_type', ['Type_A', 'Type_B', 'Type_C'])

# 或使用字典
adata.rename_categories('cell_type', {
    'Type_A': 'T_cell',
    'Type_B': 'B_cell',
    'Type_C': 'Monocyte'
})
```

## 类型转换

### 字符串转分类
```python
# 将字符串列转为分类（更节省内存）
adata.obs['cell_type'] = ['TypeA', 'TypeB'] * 500
adata.obs['tissue'] = ['brain', 'liver'] * 500

# 将所有字符串列转为分类
adata.strings_to_categoricals()

print(adata.obs['cell_type'].dtype)  # category
print(adata.obs['tissue'].dtype)     # category
```

### 稀疏与稠密矩阵互转
```python
from scipy.sparse import csr_matrix

# 稠密转稀疏
if not isinstance(adata.X, csr_matrix):
    adata.X = csr_matrix(adata.X)

# 稀疏转稠密
if isinstance(adata.X, csr_matrix):
    adata.X = adata.X.toarray()

# 转换图层
adata.layers['normalized'] = csr_matrix(adata.layers['normalized'])
```

## 分块操作

分块处理大型数据集：

```python
# 分块迭代数据
chunk_size = 100
for chunk in adata.chunked_X(chunk_size):
    # 处理分块
    result = process_chunk(chunk)
```

## 提取向量

### 获取观测向量
```python
# 获取观测元数据数组
cell_types = adata.obs_vector('cell_type')

# 获取基因在观测中的表达量
actb_expression = adata.obs_vector('ACTB')  # 若ACTB在var_names中
```

### 获取变量向量
```python
# 获取变量元数据数组
gene_names = adata.var_vector('gene_name')
```

## 添加/修改数据

### 添加观测值
```python
# 创建新观测值
new_obs = ad.AnnData(X=np.random.rand(100, adata.n_vars))
new_obs.var_names = adata.var_names

# 与现有数据拼接
adata_extended = ad.concat([adata, new_obs], axis=0)
```

### 添加变量
```python
# 创建新变量
new_vars = ad.AnnData(X=np.random.rand(adata.n_obs, 100))
new_vars.obs_names = adata.obs_names

# 与现有数据拼接
adata_extended = ad.concat([adata, new_vars], axis=1)
```

### 添加元数据列
```python
# 添加观测注释
adata.obs['new_score'] = np.random.rand(adata.n_obs)

# 添加变量注释
adata.var['new_label'] = ['label'] * adata.n_vars

# 从外部数据添加
external_data = pd.read_csv('metadata.csv', index_col=0)
adata.obs['external_info'] = external_data.loc[adata.obs_names, 'column']
```

### 添加图层
```python
# 添加新图层
adata.layers['raw_counts'] = np.random.randint(0, 100, adata.shape)
adata.layers['log_transformed'] = np.log1p(adata.X)

# 替换图层
adata.layers['normalized'] = new_normalized_data
```

### 添加嵌入
```python
# 添加PCA
adata.obsm['X_pca'] = np.random.rand(adata.n_obs, 50)

# 添加UMAP
adata.obsm['X_umap'] = np.random.rand(adata.n_obs, 2)

# 添加多个嵌入
adata.obsm['X_tsne'] = np.random.rand(adata.n_obs, 2)
adata.obsm['X_diffmap'] = np.random.rand(adata.n_obs, 10)
```

### 添加成对关系
```python
from scipy.sparse import csr_matrix

# 添加最近邻图
n_obs = adata.n_obs
knn_graph = csr_matrix(np.random.rand(n_obs, n_obs) > 0.95)
adata.obsp['connectivities'] = knn_graph

# 添加距离矩阵
adata.obsp['distances'] = csr_matrix(np.random.rand(n_obs, n_obs))
```

### 添加非结构化数据
```python
# 添加分析参数
adata.uns['pca'] = {
    'variance': [0.2, 0.15, 0.1],
    'variance_ratio': [0.4, 0.3, 0.2],
    'params': {'n_comps': 50}
}

# 添加配色方案
adata.uns['cell_type_colors'] = ['#FF0000', '#00FF00', '#0000FF']
```

## 删除数据

### 删除观测值或变量
```python
# 仅保留特定观测值
keep_obs = adata.obs['quality_score'] > 0.5
adata = adata[keep_obs, :]

# 删除特定变量
remove_vars = adata.var['low_count']
adata = adata[:, ~remove_vars]
```

### 删除元数据列
```python
# 删除观测列
adata.obs.drop('unwanted_column', axis=1, inplace=True)

# 删除变量列
adata.var.drop('unwanted_column', axis=1, inplace=True)
```

### 删除图层
```python
# 删除特定图层
del adata.layers['unwanted_layer']

# 删除所有图层
adata.layers = {}
```

### 删除嵌入
```python
# 删除特定嵌入
del adata.obsm['X_tsne']

# 删除所有嵌入
adata.obsm = {}
```

### 删除非结构化数据
```python
# 删除特定键
del adata.uns['unwanted_key']

# 删除所有非结构化数据
adata.uns = {}
```

## 重排序

### 排序观测值
```python
# 按观测元数据排序
adata = adata[adata.obs.sort_values('quality_score').index, :]

# 按观测名称排序
adata = adata[sorted(adata.obs_names), :]
```

### 排序变量
```python
# 按变量元数据排序
adata = adata[:, adata.var.sort_values('gene_name').index]

# 按变量名称排序
adata = adata[:, sorted(adata.var_names)]
```

### 匹配外部列表重排序
```python
# 按外部列表重排观测值
desired_order = ['cell_10', 'cell_5', 'cell_20', ...]
adata = adata[desired_order, :]

# 重排变量
desired_genes = ['TP53', 'ACTB', 'GAPDH', ...]
adata = adata[:, desired_genes]
```

## 数据转换

### 标准化
```python
# 总量标准化（类CPM/TPM）
total_counts = adata.X.sum(axis=1)
adata.layers['normalized'] = adata.X / total_counts[:, np.newaxis] * 1e6

# 对数转换
adata.layers['log1p'] = np.log1p(adata.X)

# Z-score标准化
mean = adata.X.mean(axis=0)
std = adata.X.std(axis=0)
adata.layers['scaled'] = (adata.X - mean) / std
```

### 过滤
```python
# 按总计数过滤细胞
total_counts = np.array(adata.X.sum(axis=1)).flatten()
adata.obs['total_counts'] = total_counts
adata = adata[adata.obs['total_counts'] > 1000, :]

# 按检出率过滤基因
detection_rate = (adata.X > 0).sum(axis=0) / adata.n_obs
adata.var['detection_rate'] = np.array(detection_rate).flatten()
adata = adata[:, adata.var['detection_rate'] > 0.01]
```

## 视图操作

视图是数据子集的轻量级引用，不复制底层矩阵：

```python
# 创建视图
view = adata[0:100, 0:500]
print(view.is_view)  # True

# 视图允许读取访问
data = view.X

# 修改视图数据会影响原始数据
# （需谨慎操作！）

# 将视图转为独立副本
independent = view.copy()

# 强制AnnData转为副本而非视图
adata = adata.copy()
```

## 合并元数据

```python
# 合并外部元数据
external_metadata = pd.read_csv('additional_metadata.csv', index_col=0)

# 连接元数据（索引内连接）
adata.obs = adata.obs.join(external_metadata)

# 左连接（保留所有adata观测值）
adata.obs = adata.obs.merge(
    external_metadata,
    left_index=True,
    right_index=True,
    how='left'
)
```

## 常用操作模式

### 质量控制过滤
```python
# 计算QC指标
adata.obs['n_genes'] = (adata.X > 0).sum(axis=1)
adata.obs['total_counts'] = adata.X.sum(axis=1)
adata.var['n_cells'] = (adata.X > 0).sum(axis=0)

# 过滤低质量细胞
adata = adata[adata.obs['n_genes'] > 200, :]
adata = adata[adata.obs['total_counts'] < 50000, :]

# 过滤低检出基因
adata = adata[:, adata.var['n_cells'] >= 3]
```

### 选择高变基因
```python
# 标记高变基因
gene_variance = np.var(adata.X, axis=0)
adata.var['variance'] = np.array(gene_variance).flatten()
adata.var['highly_variable'] = adata.var['variance'] > np.percentile(gene_variance, 90)

# 子集化为高变基因
adata_hvg = adata[:, adata.var['highly_variable']].copy()
```

### 下采样
```python
# 随机抽样观测值
np.random.seed(42)
n_sample = 500
sample_indices = np.random.choice(adata.n_obs, n_sample, replace=False)
adata_downsampled = adata[sample_indices, :].copy()

# 按细胞类型分层抽样
from sklearn.model_selection import train_test_split
train_idx, test_idx = train_test_split(
    range(adata.n_obs),
    test_size=0.2,
    stratify=adata.obs['cell_type']
)
adata_train = adata[train_idx, :].copy()
adata_test = adata[test_idx, :].copy()
```

### 拆分训练/测试集
```python
# 随机训练/测试集拆分
np.random.seed(42)
n_obs = adata.n_obs
train_size = int(0.8 * n_obs)
indices = np.random.permutation(n_obs)
train_indices = indices[:train_size]
test_indices = indices[train_size:]

adata_train = adata[train_indices, :].copy()
adata_test = adata[test_indices, :].copy()
```
