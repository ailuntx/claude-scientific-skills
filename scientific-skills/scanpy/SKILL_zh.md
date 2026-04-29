---
name: scanpy
description: 标准单细胞RNA测序分析流程。用于质量控制（QC）、标准化、降维（PCA/UMAP/t-SNE）、聚类、差异表达分析和可视化。最适合采用成熟工作流进行探索性单细胞RNA测序分析。深度学习模型请使用scvi-tools；数据格式问题请参考anndata。
license: SD-3-Clause license
metadata:
    skill-author: K-Dense Inc.
---

# Scanpy：单细胞分析工具

## 概述

Scanpy是基于AnnData构建的可扩展Python工具包，用于分析单细胞RNA测序数据。应用此技能可完成完整单细胞分析流程，包括质量控制、标准化、降维、聚类、标记基因识别、可视化和轨迹分析。

## 适用场景

此技能适用于：
- 分析单细胞RNA测序数据（.h5ad、10X、CSV格式）
- 对单细胞RNA测序数据集进行质量控制
- 创建UMAP、t-SNE或PCA可视化图谱
- 识别细胞聚类并发现标记基因
- 基于基因表达注释细胞类型
- 进行轨迹推断或拟时序分析
- 生成出版物级别的单细胞图谱

## 快速入门

### 基础导入与配置

```python
import scanpy as sc
import pandas as pd
import numpy as np

# 配置参数
sc.settings.verbosity = 3
sc.settings.set_figure_params(dpi=80, facecolor='white')
sc.settings.figdir = './figures/'
```

### 数据加载

```python
# 从10X Genomics加载
adata = sc.read_10x_mtx('数据路径/')
adata = sc.read_10x_h5('数据路径/data.h5')

# 从h5ad格式加载（AnnData格式）
adata = sc.read_h5ad('数据路径/data.h5ad')

# 从CSV加载
adata = sc.read_csv('数据路径/data.csv')
```

### 理解AnnData结构

AnnData对象是scanpy的核心数据结构：

```python
adata.X          # 表达矩阵（细胞×基因）
adata.obs        # 细胞元数据（DataFrame）
adata.var        # 基因元数据（DataFrame）
adata.uns        # 非结构化注释（字典）
adata.obsm       # 多维细胞数据（PCA, UMAP）
adata.raw        # 原始数据备份

# 访问细胞和基因名称
adata.obs_names  # 细胞条形码
adata.var_names  # 基因名称
```

## 标准分析流程

### 1. 质量控制

识别并过滤低质量细胞和基因：

```python
# 识别线粒体基因
adata.var['mt'] = adata.var_names.str.startswith('MT-')

# 计算QC指标
sc.pp.calculate_qc_metrics(adata, qc_vars=['mt'], inplace=True)

# 可视化QC指标
sc.pl.violin(adata, ['n_genes_by_counts', 'total_counts', 'pct_counts_mt'],
             jitter=0.4, multi_panel=True)

# 过滤细胞和基因
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_genes(adata, min_cells=3)
adata = adata[adata.obs.pct_counts_mt < 5, :]  # 移除高线粒体占比细胞
```

**使用自动化QC脚本：**
```bash
python scripts/qc_analysis.py input_file.h5ad --output filtered.h5ad
```

### 2. 标准化与预处理

```python
# 按细胞标准化至10,000计数
sc.pp.normalize_total(adata, target_sum=1e4)

# 对数转换
sc.pp.log1p(adata)

# 保存原始计数备用
adata.raw = adata

# 识别高变基因
sc.pp.highly_variable_genes(adata, n_top_genes=2000)
sc.pl.highly_variable_genes(adata)

# 筛选高变基因子集
adata = adata[:, adata.var.highly_variable]

# 回归去除干扰因素
sc.pp.regress_out(adata, ['total_counts', 'pct_counts_mt'])

# 数据缩放
sc.pp.scale(adata, max_value=10)
```

### 3. 降维处理

```python
# PCA分析
sc.tl.pca(adata, svd_solver='arpack')
sc.pl.pca_variance_ratio(adata, log=True)  # 检查肘部图

# 构建邻域图
sc.pp.neighbors(adata, n_neighbors=10, n_pcs=40)

# UMAP可视化
sc.tl.umap(adata)
sc.pl.umap(adata, color='leiden')

# 替代方案：t-SNE
sc.tl.tsne(adata)
```

### 4. 聚类分析

```python
# Leiden聚类（推荐）
sc.tl.leiden(adata, resolution=0.5)
sc.pl.umap(adata, color='leiden', legend_loc='on data')

# 尝试多分辨率寻找最优粒度
for res in [0.3, 0.5, 0.8, 1.0]:
    sc.tl.leiden(adata, resolution=res, key_added=f'leiden_{res}')
```

### 5. 标记基因识别

```python
# 为每个聚类寻找标记基因
sc.tl.rank_genes_groups(adata, 'leiden', method='wilcoxon')

# 结果可视化
sc.pl.rank_genes_groups(adata, n_genes=25, sharey=False)
sc.pl.rank_genes_groups_heatmap(adata, n_genes=10)
sc.pl.rank_genes_groups_dotplot(adata, n_genes=5)

# 获取DataFrame格式结果
markers = sc.get.rank_genes_groups_df(adata, group='0')
```

### 6. 细胞类型注释

```python
# 定义已知细胞类型的标记基因
marker_genes = ['CD3D', 'CD14', 'MS4A1', 'NKG7', 'FCGR3A']

# 可视化标记基因
sc.pl.umap(adata, color=marker_genes, use_raw=True)
sc.pl.dotplot(adata, var_names=marker_genes, groupby='leiden')

# 手动注释
cluster_to_celltype = {
    '0': 'CD4 T细胞',
    '1': 'CD14+单核细胞',
    '2': 'B细胞',
    '3': 'CD8 T细胞',
}
adata.obs['cell_type'] = adata.obs['leiden'].map(cluster_to_celltype)

# 可视化注释结果
sc.pl.umap(adata, color='cell_type', legend_loc='on data')
```

### 7. 结果保存

```python
# 保存处理后的数据
adata.write('results/processed_data.h5ad')

# 导出元数据
adata.obs.to_csv('results/cell_metadata.csv')
adata.var.to_csv('results/gene_metadata.csv')
```

## 常见任务

### 创建出版物级图谱

```python
# 设置高质量默认参数
sc.settings.set_figure_params(dpi=300, frameon=False, figsize=(5, 5))
sc.settings.file_format_figs = 'pdf'

# 自定义样式UMAP
sc.pl.umap(adata, color='cell_type',
           palette='Set2',
           legend_loc='on data',
           legend_fontsize=12,
           legend_fontoutline=2,
           frameon=False,
           save='_publication.pdf')

# 标记基因热图
sc.pl.heatmap(adata, var_names=genes, groupby='cell_type',
              swap_axes=True, show_gene_labels=True,
              save='_markers.pdf')

# 点图
sc.pl.dotplot(adata, var_names=genes, groupby='cell_type',
              save='_dotplot.pdf')
```

完整可视化示例请参考`references/plotting_guide.md`。

### 轨迹推断

```python
# PAGA（基于分区的图抽象）
sc.tl.paga(adata, groups='leiden')
sc.pl.paga(adata, color='leiden')

# 扩散拟时序
adata.uns['iroot'] = np.flatnonzero(adata.obs['leiden'] == '0')[0]
sc.tl.dpt(adata)
sc.pl.umap(adata, color='dpt_pseudotime')
```

### 条件间差异表达分析

```python
# 在特定细胞类型内比较处理组与对照组
adata_subset = adata[adata.obs['cell_type'] == 'T细胞']
sc.tl.rank_genes_groups(adata_subset, groupby='condition',
                         groups=['treated'], reference='control')
sc.pl.rank_genes_groups(adata_subset, groups=['treated'])
```

### 基因集评分

```python
# 计算细胞基因集表达评分
gene_set = ['CD3D', 'CD3E', 'CD3G']
sc.tl.score_genes(adata, gene_set, score_name='T_cell_score')
sc.pl.umap(adata, color='T_cell_score')
```

### 批次校正

```python
# ComBat批次校正
sc.pp.combat(adata, key='batch')

# 替代方案：使用Harmony或scVI（需单独安装）
```

## 关键参数调整

### 质量控制
- `min_genes`：每个细胞最少基因数（通常200-500）
- `min_cells`：每个基因最少细胞数（通常3-10）
- `pct_counts_mt`：线粒体基因阈值（通常5-20%）

### 标准化
- `target_sum`：目标细胞计数（默认1e4）

### 特征选择
- `n_top_genes`：高变基因数量（通常2000-3000）
- `min_mean`, `max_mean`, `min_disp`：高变基因筛选参数

### 降维处理
- `n_pcs`：主成分数量（参考方差比图）
- `n_neighbors`：邻域数量（通常10-30）

### 聚类分析
- `resolution`：聚类粒度（0.4-1.2，值越大聚类越多）

## 常见陷阱与最佳实践

1. **始终保存原始计数**：基因过滤前执行`adata.raw = adata`
2. **仔细检查QC图谱**：根据数据集质量调整阈值
3. **优先使用Leiden算法**：比Louvain更高效且效果更好
4. **尝试多分辨率聚类**：寻找最优粒度
5. **验证细胞类型注释**：使用多个标记基因交叉验证
6. **基因表达图谱使用`use_raw=True`**：显示原始计数
7. **检查PCA方差比**：确定最佳主成分数量
8. **保存中间结果**：长流程可能中途失败

## 内置资源

### scripts/qc_analysis.py
自动化质量控制脚本，计算指标、生成图谱并过滤数据：

```bash
python scripts/qc_analysis.py input.h5ad --output filtered.h5ad \
    --mt-threshold 5 --min-genes 200 --min-cells 3
```

### references/standard_workflow.md
完整分步工作流指南，包含详细解释和代码示例：
- 数据加载与配置
- 可视化质量控制
- 标准化与缩放
- 特征选择
- 降维处理（PCA, UMAP, t-SNE）
- 聚类分析（Leiden, Louvain）
- 标记基因识别
- 细胞类型注释
- 轨迹推断
- 差异表达分析

从零开始完整分析时请参考此文档。

### references/api_reference.md
Scanpy函数速查手册（按模块分类）：
- 数据读写（`sc.read_*`, `adata.write_*`）
- 预处理（`sc.pp.*`）
- 工具函数（`sc.tl.*`）
- 可视化（`sc.pl.*`）
- AnnData结构与操作
- 设置与工具

用于快速查询函数签名和常用参数。

### references/plotting_guide.md
综合可视化指南包含：
- 质量控制图谱
- 降维可视化
- 聚类可视化
- 标记基因图谱（热图、点图、小提琴图）
- 轨迹与拟时序图谱
- 出版物级定制
- 多面板图表
- 调色板与样式配置

创建出版物级图表时参考此文档。

### assets/analysis_template.py
完整分析模板，提供从数据加载到细胞注释的全流程：

```bash
cp assets/analysis_template.py my_analysis.py
# 编辑参数后运行
python my_analysis.py
```

模板包含所有标准步骤，含可配置参数和详细注释。

## 扩展资源

- **官方文档**：https://scanpy.readthedocs.io/
- **教程资源**：https://scanpy-tutorials.readthedocs.io/
- **scverse生态系统**：https://scverse.org/（相关工具：squidpy, scvi-tools, cellrank）
- **最佳实践**：Luecken & Theis (2019) "单细胞RNA-seq当前最佳实践"

## 高效分析技巧

1. **从模板开始**：使用`assets/analysis_template.py`作为起点
2. **优先运行QC脚本**：使用`scripts/qc_analysis.py`进行初始过滤
3. **按需查阅参考文档**：加载工作流和API参考到分析环境
4. **迭代优化聚类**：尝试多分辨率和可视化方法
5. **生物学验证**：检查标记基因是否符合预期细胞类型
6. **记录参数**：保存QC阈值和分析设置
7. **设置检查点**：关键步骤保存中间结果
