# 单细胞分析的标准 Scanpy 工作流程

本文档概述了使用 scanpy 分析单细胞 RNA-seq 数据的标准工作流程。

## 完整分析流程

### 1. 数据加载与初始设置

```python
import scanpy as sc
import pandas as pd
import numpy as np

# 配置 scanpy 设置
sc.settings.verbosity = 3  # 详细程度：错误(0), 警告(1), 信息(2), 提示(3)
sc.settings.set_figure_params(dpi=80, facecolor='white')

# 加载数据（多种格式）
adata = sc.read_10x_mtx('path/to/data/')  # 10X 数据
# adata = sc.read_h5ad('path/to/data.h5ad')  # h5ad 格式
# adata = sc.read_csv('path/to/data.csv')  # CSV 格式
```

### 2. 质量控制（QC）

```python
# 计算 QC 指标
sc.pp.calculate_qc_metrics(adata, qc_vars=['mt'], percent_top=None, log1p=False, inplace=True)

# 常用过滤阈值（根据数据集调整）
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_genes(adata, min_cells=3)

# 移除高线粒体含量的细胞
adata = adata[adata.obs.pct_counts_mt < 5, :]

# 可视化 QC 指标
sc.pl.violin(adata, ['n_genes_by_counts', 'total_counts', 'pct_counts_mt'],
             jitter=0.4, multi_panel=True)
sc.pl.scatter(adata, x='total_counts', y='pct_counts_mt')
sc.pl.scatter(adata, x='total_counts', y='n_genes_by_counts')
```

### 3. 归一化处理

```python
# 按细胞标准化至 10,000 计数
sc.pp.normalize_total(adata, target_sum=1e4)

# 对数转换数据
sc.pp.log1p(adata)

# 存储归一化数据供后续使用
adata.raw = adata
```

### 4. 特征选择

```python
# 识别高变基因
sc.pp.highly_variable_genes(adata, min_mean=0.0125, max_mean=3, min_disp=0.5)

# 可视化高变基因
sc.pl.highly_variable_genes(adata)

# 筛选高变基因子集
adata = adata[:, adata.var.highly_variable]
```

### 5. 数据缩放与回归

```python
# 回归去除细胞总计数和线粒体基因百分比的影响
sc.pp.regress_out(adata, ['total_counts', 'pct_counts_mt'])

# 缩放数据至单位方差和零均值
sc.pp.scale(adata, max_value=10)
```

### 6. 降维分析

```python
# 主成分分析（PCA）
sc.tl.pca(adata, svd_solver='arpack')

# 可视化 PCA 结果
sc.pl.pca(adata, color='CST3')
sc.pl.pca_variance_ratio(adata, log=True)

# 计算邻域图
sc.pp.neighbors(adata, n_neighbors=10, n_pcs=40)

# UMAP 可视化
sc.tl.umap(adata)

# t-SNE（UMAP替代方案）
# sc.tl.tsne(adata)
```

### 7. 聚类分析

```python
# Leiden 聚类（推荐）
sc.tl.leiden(adata, resolution=0.5)

# 替代方案：Louvain 聚类
# sc.tl.louvain(adata, resolution=0.5)

# 可视化聚类结果
sc.pl.umap(adata, color=['leiden'], legend_loc='on data')
```

### 8. 标记基因识别

```python
# 为每个聚类寻找标记基因
sc.tl.rank_genes_groups(adata, 'leiden', method='wilcoxon')

# 可视化顶级标记基因
sc.pl.rank_genes_groups(adata, n_genes=25, sharey=False)

# 获取标记基因数据框
marker_genes = sc.get.rank_genes_groups_df(adata, group='0')

# 可视化特定标记物
sc.pl.umap(adata, color=['leiden', 'CST3', 'NKG7'])
```

### 9. 细胞类型注释

```python
# 基于标记基因的手动注释
cluster_annotations = {
    '0': 'CD4 T 细胞',
    '1': 'CD14+ 单核细胞',
    '2': 'B 细胞',
    '3': 'CD8 T 细胞',
    # ... 添加更多注释
}
adata.obs['cell_type'] = adata.obs['leiden'].map(cluster_annotations)

# 可视化注释的细胞类型
sc.pl.umap(adata, color='cell_type', legend_loc='on data')
```

### 10. 结果保存

```python
# 保存处理后的 AnnData 对象
adata.write('results/processed_data.h5ad')

# 导出结果至 CSV
adata.obs.to_csv('results/cell_metadata.csv')
adata.var.to_csv('results/gene_metadata.csv')
```

## 附加分析选项

### 轨迹推断

```python
# PAGA（基于分区的图抽象）
sc.tl.paga(adata, groups='leiden')
sc.pl.paga(adata, color=['leiden'])

# 扩散伪时间（DPT）
adata.uns['iroot'] = np.flatnonzero(adata.obs['leiden'] == '0')[0]
sc.tl.dpt(adata)
sc.pl.umap(adata, color=['dpt_pseudotime'])
```

### 条件间差异表达

```python
# 在细胞类型内比较条件
sc.tl.rank_genes_groups(adata, groupby='condition', groups=['treated'],
                         reference='control', method='wilcoxon')
sc.pl.rank_genes_groups(adata, groups=['treated'])
```

### 基因集评分

```python
# 计算细胞基因集表达评分
gene_set = ['CD3D', 'CD3E', 'CD3G']
sc.tl.score_genes(adata, gene_set, score_name='T_cell_score')
sc.pl.umap(adata, color='T_cell_score')
```

## 常用参数调整

- **QC 阈值**：`min_genes`, `min_cells`, `pct_counts_mt` - 取决于数据集质量
- **归一化目标**：通常为 1e4，可调整
- **高变基因参数**：影响特征选择严格度
- **PCA 成分数**：通过方差比图确定最优数量
- **聚类分辨率**：值越高聚类越多（通常 0.4-1.2）
- **n_neighbors**：影响 UMAP 和聚类的粒度（通常 10-30）

## 最佳实践

1. 过滤前始终可视化 QC 指标
2. 归一化前保存原始计数（`adata.raw = adata`）
3. 使用 Leiden 替代 Louvain 聚类（效率更高）
4. 尝试多种聚类分辨率寻找最优粒度
5. 用已知标记基因验证细胞类型注释
6. 关键步骤保存中间结果
