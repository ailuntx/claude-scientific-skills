# 最佳实践

高效使用 AnnData 的指导原则。

## 内存管理

### 对稀疏数据使用稀疏矩阵
```python
import numpy as np
from scipy.sparse import csr_matrix
import anndata as ad

# 检查数据稀疏度
data = np.random.rand(1000, 2000)
sparsity = 1 - np.count_nonzero(data) / data.size
print(f"稀疏度: {sparsity:.2%}")

# 当零值比例>50%时转换为稀疏矩阵
if sparsity > 0.5:
    adata = ad.AnnData(X=csr_matrix(data))
else:
    adata = ad.AnnData(X=data)

# 优势：稀疏基因组数据可减少10-100倍内存占用
```

### 将字符串转换为分类类型
```python
# 低效：字符串列占用大量内存
adata.obs['cell_type'] = ['Type_A', 'Type_B', 'Type_C'] * 333 + ['Type_A']

# 高效：转换为分类类型
adata.obs['cell_type'] = adata.obs['cell_type'].astype('category')

# 转换所有字符串列
adata.strings_to_categoricals()

# 优势：重复字符串可减少10-50倍内存占用
```

### 对大型数据集使用备份模式
```python
# 避免将整个数据集加载到内存
adata = ad.read_h5ad('large_dataset.h5ad', backed='r')

# 操作元数据
filtered = adata[adata.obs['quality'] > 0.8]

# 仅加载过滤后的子集
adata_subset = filtered.to_memory()

# 优势：处理大于内存容量的数据集
```

## 视图与副本

### 理解视图
```python
# 默认情况下子集操作创建视图
subset = adata[0:100, :]
print(subset.is_view)  # True

# 视图不复制数据（内存高效）
# 但修改可能影响原始数据

# 检查对象是否为视图
if adata.is_view:
    adata = adata.copy()  # 创建独立副本
```

### 何时使用视图
```python
# 适用场景：对子集的只读操作
mean_expr = adata[adata.obs['cell_type'] == 'T cell'].X.mean()

# 适用场景：临时分析
temp_subset = adata[:100, :]
result = analyze(temp_subset.X)
```

### 何时使用副本
```python
# 创建独立副本进行修改
adata_filtered = adata[keep_cells, :].copy()

# 可安全修改而不影响原始数据
adata_filtered.obs['new_column'] = values

# 以下情况始终创建副本：
# - 存储子集供后续使用
# - 修改子集数据
# - 传递给会修改数据的函数
```

## 数据存储最佳实践

### 选择合适格式

**H5AD (HDF5) - 默认选择**
```python
adata.write_h5ad('data.h5ad', compression='gzip')
```
- 快速随机访问
- 支持备份模式
- 良好压缩率
- 最佳适用：多数场景

**Zarr - 云端与并行访问**
```python
adata.write_zarr('data.zarr', chunks=(100, 100))
```
- 完美适配云存储(S3, GCS)
- 支持并行I/O
- 良好压缩率
- 最佳适用：大型数据集/云端工作流/并行处理

**CSV - 互操作性**
```python
adata.write_csvs('output_dir/')
```
- 人类可读
- 兼容所有工具
- 文件体积大/速度慢
- 最佳适用：与非Python工具共享/小型数据集

### 优化文件体积
```python
# 保存前优化：

# 1. 如适用转换为稀疏矩阵
from scipy.sparse import csr_matrix, issparse
if not issparse(adata.X):
    density = np.count_nonzero(adata.X) / adata.X.size
    if density < 0.5:
        adata.X = csr_matrix(adata.X)

# 2. 将字符串转为分类类型
adata.strings_to_categoricals()

# 3. 使用压缩
adata.write_h5ad('data.h5ad', compression='gzip', compression_opts=9)

# 典型效果：文件体积缩小5-20倍
```

## 备份模式策略

### 只读分析
```python
# 以只读备份模式打开
adata = ad.read_h5ad('data.h5ad', backed='r')

# 无需加载数据即可执行过滤
high_quality = adata[adata.obs['quality_score'] > 0.8]

# 仅加载过滤后的数据
adata_filtered = high_quality.to_memory()
```

### 读写修改
```python
# 以读写备份模式打开
adata = ad.read_h5ad('data.h5ad', backed='r+')

# 修改元数据（直接写入磁盘）
adata.obs['new_annotation'] = values

# X保留在磁盘上，修改即时保存
```

### 分块处理
```python
# 分块处理大型数据集
adata = ad.read_h5ad('huge_dataset.h5ad', backed='r')

results = []
chunk_size = 1000

for i in range(0, adata.n_obs, chunk_size):
    chunk = adata[i:i+chunk_size, :].to_memory()
    result = process(chunk)
    results.append(result)

final_result = combine(results)
```

## 性能优化

### 子集操作性能
```python
# 快速：使用数组布尔索引
mask = np.array(adata.obs['quality'] > 0.5)
subset = adata[mask, :]

# 慢速：使用Series布尔索引（创建视图链）
subset = adata[adata.obs['quality'] > 0.5, :]

# 最快：整数索引
indices = np.where(adata.obs['quality'] > 0.5)[0]
subset = adata[indices, :]
```

### 避免重复子集操作
```python
# 低效：多次子集操作
for cell_type in ['A', 'B', 'C']:
    subset = adata[adata.obs['cell_type'] == cell_type]
    process(subset)

# 高效：分组处理
groups = adata.obs.groupby('cell_type').groups
for cell_type, indices in groups.items():
    subset = adata[indices, :]
    process(subset)
```

### 对大型矩阵使用分块操作
```python
# 分块处理X矩阵
for chunk in adata.chunked_X(chunk_size=1000):
    result = compute(chunk)

# 比加载完整X更节省内存
```

## 原始数据处理

### 过滤前存储原始数据
```python
# 包含所有基因的原始数据
adata = ad.AnnData(X=counts)

# 过滤前存储原始数据
adata.raw = adata.copy()

# 过滤保留高变基因
adata = adata[:, adata.var['highly_variable']]

# 后续访问原始数据
original_expression = adata.raw.X
all_genes = adata.raw.var_names
```

### 何时使用原始数据
```python
# 原始数据适用于：
# - 对过滤基因进行差异表达分析
# - 可视化过滤集中未包含的特定基因
# - 标准化后访问原始计数

# 访问原始数据
if adata.raw is not None:
    gene_expr = adata.raw[:, 'GENE_NAME'].X
else:
    gene_expr = adata[:, 'GENE_NAME'].X
```

## 元数据管理

### 命名规范
```python
# 一致的命名提升可用性

# 观测元数据(obs)：
# - cell_id, sample_id
# - cell_type, tissue, condition
# - n_genes, n_counts, percent_mito
# - cluster, leiden, louvain

# 变量元数据(var)：
# - gene_id, gene_name
# - highly_variable, n_cells
# - mean_expression, dispersion

# 嵌入数据(obsm)：
# - X_pca, X_umap, X_tsne
# - X_diffmap, X_draw_graph_fr

# 遵循scanpy/scverse生态系统的约定
```

### 文档化元数据
```python
# 在uns中存储元数据描述
adata.uns['metadata_descriptions'] = {
    'cell_type': '来自自动聚类的细胞类型注释',
    'quality_score': 'scrublet的质量评分(0-1, 越高越好)',
    'batch': '实验批次标识符'
}

# 存储处理历史
adata.uns['processing_steps'] = [
    '从10X加载原始计数',
    '过滤条件：n_genes > 200, n_counts < 50000',
    '标准化为每细胞10000计数',
    '进行log转换'
]
```

## 可复现性

### 设置随机种子
```python
import numpy as np

# 设置种子保证结果可复现
np.random.seed(42)

# 在uns中记录
adata.uns['random_seed'] = 42
```

### 存储参数
```python
# 在uns中存储分析参数
adata.uns['pca'] = {
    'n_comps': 50,
    'svd_solver': 'arpack',
    'random_state': 42
}

adata.uns['neighbors'] = {
    'n_neighbors': 15,
    'n_pcs': 50,
    'metric': 'euclidean',
    'method': 'umap'
}
```

### 版本追踪
```python
import anndata
import scanpy
import numpy

# 存储版本信息
adata.uns['versions'] = {
    'anndata': anndata.__version__,
    'scanpy': scanpy.__version__,
    'numpy': numpy.__version__,
    'python': sys.version
}
```

## 错误处理

### 检查数据有效性
```python
# 验证维度
assert adata.n_obs == len(adata.obs)
assert adata.n_vars == len(adata.var)
assert adata.X.shape == (adata.n_obs, adata.n_vars)

# 检查NaN值
has_nan = np.isnan(adata.X.data).any() if issparse(adata.X) else np.isnan(adata.X).any()
if has_nan:
    print("警告：数据包含NaN值")

# 检查负值（当预期为计数数据时）
has_negative = (adata.X.data < 0).any() if issparse(adata.X) else (adata.X < 0).any()
if has_negative:
    print("警告：数据包含负值")
```

### 验证元数据
```python
# 检查缺失值
missing_obs = adata.obs.isnull().sum()
if missing_obs.any():
    print("obs中存在缺失值：")
    print(missing_obs[missing_obs > 0])

# 验证索引唯一性
assert adata.obs_names.is_unique, "观测名称不唯一"
assert adata.var_names.is_unique, "变量名称不唯一"

# 检查元数据对齐
assert len(adata.obs) == adata.n_obs
assert len(adata.var) == adata.n_vars
```

## 与其他工具集成

### Scanpy集成
```python
import scanpy as sc

# AnnData是scanpy的本地格式
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_genes(adata, min_cells=3)
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata)
sc.pp.pca(adata)
sc.pp.neighbors(adata)
sc.tl.umap(adata)
```

### Pandas集成
```python
import pandas as pd

# 转换为DataFrame
df = adata.to_df()

# 从DataFrame创建
adata = ad.AnnData(df)

# 将元数据作为DataFrame操作
adata.obs = adata.obs.merge(external_metadata, left_index=True, right_index=True)
```

### PyTorch集成
```python
from anndata.experimental import AnnLoader

# 创建PyTorch DataLoader
dataloader = AnnLoader(adata, batch_size=128, shuffle=True)

# 在训练循环中迭代
for batch in dataloader:
    X = batch.X
    # 在批次上训练模型
```

## 常见陷阱

### 陷阱1：修改视图
```python
# 错误：修改视图可能影响原始数据
subset = adata[:100, :]
subset.X = new_data  # 可能修改adata.X!

# 正确：修改前创建副本
subset = adata[:100, :].copy()
subset.X = new_data  # 独立副本
```

### 陷阱2：索引错位
```python
# 错误：假设顺序匹配
external_data = pd.read_csv('data.csv')
adata.obs['new_col'] = external_data['values']  # 可能错位!

# 正确：按索引对齐
adata.obs['new_col'] = external_data.set_index('cell_id').loc[adata.obs_names, 'values']
```

### 陷阱3：混合稀疏与稠密矩阵
```python
# 错误：稀疏转稠密消耗巨大内存
result = adata.X + 1  # 将稀疏矩阵转为稠密!

# 正确：使用稀疏操作
from scipy.sparse import issparse
if issparse(adata.X):
    result = adata.X.copy()
    result.data += 1
```

### 陷阱4：未正确处理视图
```python
# 错误：假设子集独立
subset = adata[mask, :]
del adata  # subset可能失效!

# 正确：需要时创建副本
subset = adata[mask, :].copy()
del adata  # subset保持有效
```

### 陷阱5：忽略内存限制
```python
# 错误：将超大数据集加载到内存
adata = ad.read_h5ad('100GB_file.h5ad')  # 内存溢出错误!

# 正确：使用备份模式
adata = ad.read_h5ad('100GB_file.h5ad', backed='r')
subset = adata[adata.obs['keep']].to_memory()
```

## 工作流示例

完整的最佳实践工作流：

```python
import anndata as ad
import numpy as np
from scipy.sparse import csr_matrix

# 1. 大型数据集使用备份模式加载
adata = ad.read_h5ad('data.h5ad', backed='r')

# 2. 不加载数据快速检查元数据
print(f"数据集: {adata.n_obs}细胞 × {adata.n_vars}基因")

# 3. 基于元数据过滤
high_quality = adata[adata.obs['quality_score'] > 0.8]

# 4. 将过滤子集加载到内存
adata = high_quality.to_memory()

# 5. 转换为最优存储类型
adata.strings_to_categoricals()
if not issparse(adata.X):
    density = np.count_nonzero(adata.X) / adata.X.size
    if density < 0.5:
        adata.X = csr_matrix(adata.X)

# 6. 过滤基因前存储原始数据
adata.raw = adata.copy()

# 7. 过滤保留高变基因
adata = adata[:, adata.var['highly_variable']].copy()

# 8. 记录处理过程
adata.uns['processing'] = {
    'filtered': 'quality_score > 0.8',
    'n_hvg': adata.n_vars,
    'date': '2025-11-03'
}

# 9. 优化保存
adata.write_h5ad('processed.h5ad', compression='gzip')
```
