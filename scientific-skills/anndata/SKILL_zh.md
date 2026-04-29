---
name: anndata
description: 用于单细胞分析中带注释矩阵的数据结构。适用于处理.h5ad文件或与scverse生态系统集成。此为数据格式技能——分析工作流请使用scanpy；概率模型请使用scvi-tools；群体规模查询请使用cellxgene-census。
license: BSD-3-Clause许可证
metadata:
    skill-author: K-Dense公司
---

# AnnData

## 概述

AnnData是一个用于处理带注释数据矩阵的Python包，它将实验测量值（X）与观测元数据（obs）、变量元数据（var）以及多维注释（obsm、varm、obsp、varp、uns）共同存储。最初为Scanpy设计的单细胞基因组学工具，现已成为通用框架，适用于任何需要高效存储、操作和分析的带注释数据。

## 何时使用此技能

在以下场景使用此技能：
- 创建、读取或写入AnnData对象
- 处理h5ad、zarr或其他基因组学数据格式
- 执行单细胞RNA-seq分析
- 管理包含稀疏矩阵或备份模式的大型数据集
- 拼接多个数据集或实验批次
- 子集化、过滤或转换带注释数据
- 与scanpy、scvi-tools或其他scverse生态系统工具集成

## 安装

```bash
uv pip install anndata

# 安装可选依赖
uv pip install anndata[dev,test,doc]
```

## 快速入门

### 创建AnnData对象
```python
import anndata as ad
import numpy as np
import pandas as pd

# 最小化创建
X = np.random.rand(100, 2000)  # 100个细胞 × 2000个基因
adata = ad.AnnData(X)

# 包含元数据
obs = pd.DataFrame({
    'cell_type': ['T细胞', 'B细胞'] * 50,
    'sample': ['A', 'B'] * 50
}, index=[f'cell_{i}' for i in range(100)])

var = pd.DataFrame({
    'gene_name': [f'Gene_{i}' for i in range(2000)]
}, index=[f'ENSG{i:05d}' for i in range(2000)])

adata = ad.AnnData(X=X, obs=obs, var=var)
```

### 读取数据
```python
# 读取h5ad文件
adata = ad.read_h5ad('data.h5ad')

# 以备份模式读取（适用于大文件）
adata = ad.read_h5ad('large_data.h5ad', backed='r')

# 读取其他格式
adata = ad.read_csv('data.csv')
adata = ad.read_loom('data.loom')
adata = ad.read_10x_h5('filtered_feature_bc_matrix.h5')
```

### 写入数据
```python
# 写入h5ad文件
adata.write_h5ad('output.h5ad')

# 带压缩写入
adata.write_h5ad('output.h5ad', compression='gzip')

# 写入其他格式
adata.write_zarr('output.zarr')
adata.write_csvs('output_dir/')
```

### 基础操作
```python
# 按条件子集化
t_cells = adata[adata.obs['cell_type'] == 'T细胞']

# 按索引子集化
subset = adata[0:50, 0:100]

# 添加元数据
adata.obs['quality_score'] = np.random.rand(adata.n_obs)
adata.var['highly_variable'] = np.random.rand(adata.n_vars) > 0.8

# 访问维度
print(f"{adata.n_obs}个观测值 × {adata.n_vars}个变量")
```

## 核心功能

### 1. 数据结构

理解AnnData对象结构，包括X、obs、var、layers、obsm、varm、obsp、varp、uns和raw组件。

**参见**：`references/data_structure.md`获取完整信息：
- 核心组件（X, obs, var, layers, obsm, varm, obsp, varp, uns, raw）
- 从不同来源创建AnnData对象
- 访问和操作数据组件
- 内存优化实践

### 2. 输入/输出操作

支持压缩、备份模式和云存储，读写多种格式数据。

**参见**：`references/io_operations.md`获取详情：
- 原生格式（h5ad, zarr）
- 替代格式（CSV, MTX, Loom, 10X, Excel）
- 大型数据集备份模式
- 远程数据访问
- 格式转换
- 性能优化

常用命令：
```python
# 读写h5ad
adata = ad.read_h5ad('data.h5ad', backed='r')
adata.write_h5ad('output.h5ad', compression='gzip')

# 读取10X数据
adata = ad.read_10x_h5('filtered_feature_bc_matrix.h5')

# 读取MTX格式
adata = ad.read_mtx('matrix.mtx').T
```

### 3. 数据拼接

沿观测值或变量轴灵活拼接多个AnnData对象。

**参见**：`references/concatenation.md`获取完整指南：
- 基础拼接（axis=0沿观测值，axis=1沿变量）
- 连接类型（inner, outer）
- 合并策略（same, unique, first, only）
- 使用标签追踪数据来源
- 延迟拼接（AnnCollection）
- 大型数据集磁盘拼接

常用命令：
```python
# 沿观测值拼接（合并样本）
adata = ad.concat(
    [adata1, adata2, adata3],
    axis=0,
    join='inner',
    label='batch',
    keys=['batch1', 'batch2', 'batch3']
)

# 沿变量拼接（合并模态）
adata = ad.concat([adata_rna, adata_protein], axis=1)

# 延迟拼接
from anndata.experimental import AnnCollection
collection = AnnCollection(
    ['data1.h5ad', 'data2.h5ad'],
    join_obs='outer',
    label='dataset'
)
```

### 4. 数据操作

高效转换、子集化、过滤和重组数据。

**参见**：`references/manipulation.md`获取详细指导：
- 子集化（按索引、名称、布尔掩码、元数据条件）
- 转置
- 复制（完整副本与视图）
- 重命名（观测值、变量、类别）
- 类型转换（字符串转分类、稀疏/稠密）
- 添加/删除数据组件
- 重排序
- 质量控制过滤

常用命令：
```python
# 按元数据子集化
filtered = adata[adata.obs['quality_score'] > 0.8]
hv_genes = adata[:, adata.var['highly_variable']]

# 转置
adata_T = adata.T

# 复制与视图
view = adata[0:100, :]  # 视图（轻量引用）
copy = adata[0:100, :].copy()  # 独立副本

# 字符串转分类
adata.strings_to_categoricals()
```

### 5. 最佳实践

遵循内存效率、性能和可复现性推荐模式。

**参见**：`references/best_practices.md`获取指南：
- 内存管理（稀疏矩阵、分类、备份模式）
- 视图与副本
- 数据存储优化
- 性能优化
- 原始数据处理
- 元数据管理
- 可复现性
- 错误处理
- 与其他工具集成
- 常见陷阱与解决方案

关键建议：
```python
# 对稀疏数据使用稀疏矩阵
from scipy.sparse import csr_matrix
adata.X = csr_matrix(adata.X)

# 字符串转分类
adata.strings_to_categoricals()

# 大文件使用备份模式
adata = ad.read_h5ad('large.h5ad', backed='r')

# 过滤前存储原始数据
adata.raw = adata.copy()
adata = adata[:, adata.var['highly_variable']]
```

## 与Scverse生态系统集成

AnnData是scverse生态系统的基石数据结构：

### Scanpy（单细胞分析）
```python
import scanpy as sc

# 预处理
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata, n_top_genes=2000)

# 降维
sc.pp.pca(adata, n_comps=50)
sc.pp.neighbors(adata, n_neighbors=15)
sc.tl.umap(adata)
sc.tl.leiden(adata)

# 可视化
sc.pl.umap(adata, color=['cell_type', 'leiden'])
```

### Muon（多模态数据）
```python
import muon as mu

# 合并RNA与蛋白质数据
mdata = mu.MuData({'rna': adata_rna, 'protein': adata_protein})
```

### PyTorch集成
```python
from anndata.experimental import AnnLoader

# 创建深度学习DataLoader
dataloader = AnnLoader(adata, batch_size=128, shuffle=True)

for batch in dataloader:
    X = batch.X
    # 训练模型
```

## 常见工作流

### 单细胞RNA-seq分析
```python
import anndata as ad
import scanpy as sc

# 1. 加载数据
adata = ad.read_10x_h5('filtered_feature_bc_matrix.h5')

# 2. 质量控制
adata.obs['n_genes'] = (adata.X > 0).sum(axis=1)
adata.obs['n_counts'] = adata.X.sum(axis=1)
adata = adata[adata.obs['n_genes'] > 200]
adata = adata[adata.obs['n_counts'] < 50000]

# 3. 存储原始数据
adata.raw = adata.copy()

# 4. 标准化与过滤
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata, n_top_genes=2000)
adata = adata[:, adata.var['highly_variable']]

# 5. 保存处理数据
adata.write_h5ad('processed.h5ad')
```

### 批次整合
```python
# 加载多个批次
adata1 = ad.read_h5ad('batch1.h5ad')
adata2 = ad.read_h5ad('batch2.h5ad')
adata3 = ad.read_h5ad('batch3.h5ad')

# 带批次标签拼接
adata = ad.concat(
    [adata1, adata2, adata3],
    label='batch',
    keys=['batch1', 'batch2', 'batch3'],
    join='inner'
)

# 应用批次校正
import scanpy as sc
sc.pp.combat(adata, key='batch')

# 继续分析
sc.pp.pca(adata)
sc.pp.neighbors(adata)
sc.tl.umap(adata)
```

### 处理大型数据集
```python
# 以备份模式打开
adata = ad.read_h5ad('100GB_dataset.h5ad', backed='r')

# 基于元数据过滤（不加载数据）
high_quality = adata[adata.obs['quality_score'] > 0.8]

# 加载过滤子集
adata_subset = high_quality.to_memory()

# 处理子集
process(adata_subset)

# 或分块处理
chunk_size = 1000
for i in range(0, adata.n_obs, chunk_size):
    chunk = adata[i:i+chunk_size, :].to_memory()
    process(chunk)
```

## 故障排除

### 内存溢出错误
使用备份模式或转为稀疏矩阵：
```python
# 备份模式
adata = ad.read_h5ad('file.h5ad', backed='r')

# 稀疏矩阵
from scipy.sparse import csr_matrix
adata.X = csr_matrix(adata.X)
```

### 文件读取缓慢
使用压缩和合适格式：
```python
# 存储优化
adata.strings_to_categoricals()
adata.write_h5ad('file.h5ad', compression='gzip')

# 云存储使用Zarr
adata.write_zarr('file.zarr', chunks=(1000, 1000))
```

### 索引对齐问题
始终按索引对齐外部数据：
```python
# 错误方式
adata.obs['new_col'] = external_data['values']

# 正确方式
adata.obs['new_col'] = external_data.set_index('cell_id').loc[adata.obs_names, 'values']
```

## 附加资源

- **官方文档**：https://anndata.readthedocs.io/
- **Scanpy教程**：https://scanpy.readthedocs.io/
- **Scverse生态系统**：https://scverse.org/
- **GitHub仓库**：https://github.com/scverse/anndata
