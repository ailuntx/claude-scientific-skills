# Scanpy 绘图指南

使用 scanpy 创建出版级质量可视化的综合指南。

## 通用绘图原则

所有 scanpy 绘图函数遵循一致模式：
- `sc.pl.*` 中的函数对应 `sc.tl.*` 中的分析函数
- 多数函数接受 `color` 参数用于基因名或元数据列
- 通过 `save` 参数保存结果
- 单次调用可生成多个图表

## 核心质量控制图表

### 可视化 QC 指标

```python
# 绘制QC指标的小提琴图
sc.pl.violin(adata, ['n_genes_by_counts', 'total_counts', 'pct_counts_mt'],
             jitter=0.4, multi_panel=True, save='_qc_violin.pdf')

# 识别异常值的散点图
sc.pl.scatter(adata, x='total_counts', y='pct_counts_mt', save='_qc_mt.pdf')
sc.pl.scatter(adata, x='total_counts', y='n_genes_by_counts', save='_qc_genes.pdf')

# 高表达基因展示
sc.pl.highest_expr_genes(adata, n_top=20, save='_highest_expr.pdf')
```

### 过滤后 QC 验证

```python
# 比较过滤前后数据
sc.pl.violin(adata, ['n_genes_by_counts', 'total_counts'],
             groupby='sample', save='_post_filter.pdf')
```

## 降维可视化

### PCA 图表

```python
# 基础PCA图
sc.pl.pca(adata, color='leiden', save='_pca.pdf')

# 按基因表达着色的PCA
sc.pl.pca(adata, color=['gene1', 'gene2', 'gene3'], save='_pca_genes.pdf')

# 方差比图表（肘部图）
sc.pl.pca_variance_ratio(adata, log=True, n_pcs=50, save='_variance.pdf')

# PCA载荷可视化
sc.pl.pca_loadings(adata, components=[1, 2, 3], save='_loadings.pdf')
```

### UMAP 图表

```python
# 带聚类的基础UMAP
sc.pl.umap(adata, color='leiden', legend_loc='on data', save='_umap_leiden.pdf')

# 多变量着色的UMAP
sc.pl.umap(adata, color=['leiden', 'cell_type', 'batch'],
           save='_umap_multi.pdf')

# 基因表达UMAP
sc.pl.umap(adata, color=['CD3D', 'CD14', 'MS4A1'],
           use_raw=False, save='_umap_genes.pdf')

# 自定义外观
sc.pl.umap(adata, color='leiden',
           palette='Set2',
           size=50,
           alpha=0.8,
           frameon=False,
           title='细胞类型',
           save='_umap_custom.pdf')
```

### t-SNE 图表

```python
# 带聚类的t-SNE
sc.pl.tsne(adata, color='leiden', legend_loc='right margin', save='_tsne.pdf')

# 多困惑度t-SNE（如已计算）
sc.pl.tsne(adata, color='leiden', save='_tsne_default.pdf')
```

## 聚类可视化

### 基础聚类图

```python
# 带聚类标注的UMAP
sc.pl.umap(adata, color='leiden', add_outline=True,
           legend_loc='on data', legend_fontsize=12,
           legend_fontoutline=2, frameon=False,
           save='_clusters.pdf')

# 展示聚类比例
sc.pl.umap(adata, color='leiden', size=50, edges=True,
           edges_width=0.1, save='_clusters_edges.pdf')
```

### 聚类比较

```python
# 比较聚类结果
sc.pl.umap(adata, color=['leiden', 'louvain'],
           save='_cluster_comparison.pdf')

# 聚类树状图
sc.tl.dendrogram(adata, groupby='leiden')
sc.pl.dendrogram(adata, groupby='leiden', save='_dendrogram.pdf')
```

## 标记基因可视化

### 排序标记基因

```python
# 各聚类Top标记基因概览
sc.pl.rank_genes_groups(adata, n_genes=25, sharey=False,
                        save='_marker_overview.pdf')

# Top标记基因热图
sc.pl.rank_genes_groups_heatmap(adata, n_genes=10, groupby='leiden',
                                 show_gene_labels=True,
                                 save='_marker_heatmap.pdf')

# 标记基因点图
sc.pl.rank_genes_groups_dotplot(adata, n_genes=5,
                                 save='_marker_dotplot.pdf')

# 堆叠小提琴图
sc.pl.rank_genes_groups_stacked_violin(adata, n_genes=5,
                                        save='_marker_violin.pdf')

# 矩阵图
sc.pl.rank_genes_groups_matrixplot(adata, n_genes=5,
                                    save='_marker_matrix.pdf')
```

### 特定基因表达

```python
# 特定基因小提琴图
marker_genes = ['CD3D', 'CD14', 'MS4A1', 'NKG7', 'FCGR3A']
sc.pl.violin(adata, keys=marker_genes, groupby='leiden',
             save='_markers_violin.pdf')

# 定制标记基因点图
sc.pl.dotplot(adata, var_names=marker_genes, groupby='leiden',
              save='_markers_dotplot.pdf')

# 特定基因热图
sc.pl.heatmap(adata, var_names=marker_genes, groupby='leiden',
              swap_axes=True, save='_markers_heatmap.pdf')

# 基因集堆叠小提琴图
sc.pl.stacked_violin(adata, var_names=marker_genes, groupby='leiden',
                     save='_markers_stacked.pdf')
```

### 嵌入空间基因表达

```python
# UMAP多基因表达
genes = ['CD3D', 'CD14', 'MS4A1', 'NKG7']
sc.pl.umap(adata, color=genes, cmap='viridis',
           save='_umap_markers.pdf')

# 自定义色图的基因表达
sc.pl.umap(adata, color='CD3D', cmap='Reds',
           vmin=0, vmax=3, save='_umap_cd3d.pdf')
```

## 轨迹与伪时序可视化

### PAGA 图表

```python
# PAGA图
sc.pl.paga(adata, color='leiden', save='_paga.pdf')

# 带基因表达的PAGA
sc.pl.paga(adata, color=['leiden', 'dpt_pseudotime'],
           save='_paga_pseudotime.pdf')

# UMAP叠加PAGA
sc.pl.umap(adata, color='leiden', save='_umap_with_paga.pdf',
           edges=True, edges_color='gray')
```

### 伪时序图表

```python
# UMAP展示DPT伪时序
sc.pl.umap(adata, color='dpt_pseudotime', save='_umap_dpt.pdf')

# 伪时序基因表达趋势
sc.pl.dpt_timeseries(adata, save='_dpt_timeseries.pdf')

# 伪时序排序热图
sc.pl.heatmap(adata, var_names=genes, groupby='leiden',
              use_raw=False, show_gene_labels=True,
              save='_pseudotime_heatmap.pdf')
```

## 高级可视化

### 轨迹图（基因表达趋势）

```python
# 展示跨细胞类型的基因表达
sc.pl.tracksplot(adata, var_names=marker_genes, groupby='leiden',
                 save='_tracks.pdf')
```

### 相关性矩阵

```python
# 聚类间相关性
sc.pl.correlation_matrix(adata, groupby='leiden',
                         save='_correlation.pdf')
```

### 嵌入密度

```python
# UMAP细胞密度
sc.tl.embedding_density(adata, basis='umap', groupby='cell_type')
sc.pl.embedding_density(adata, basis='umap', key='umap_density_cell_type',
                        save='_density.pdf')
```

## 多面板图表

### 创建组合图表

```python
import matplotlib.pyplot as plt

# 创建多面板图表
fig, axes = plt.subplots(2, 2, figsize=(12, 12))

# 在指定坐标轴绘图
sc.pl.umap(adata, color='leiden', ax=axes[0, 0], show=False)
sc.pl.umap(adata, color='CD3D', ax=axes[0, 1], show=False)
sc.pl.umap(adata, color='CD14', ax=axes[1, 0], show=False)
sc.pl.umap(adata, color='MS4A1', ax=axes[1, 1], show=False)

plt.tight_layout()
plt.savefig('figures/multi_panel.pdf')
plt.show()
```

## 出版级定制化

### 高质量设置

```python
# 设置出版级默认参数
sc.settings.set_figure_params(dpi=300, frameon=False, figsize=(5, 5),
                               facecolor='white')

# 矢量图形输出
sc.settings.figdir = './figures/'
sc.settings.file_format_figs = 'pdf'  # 或'svg'
```

### 自定义调色板

```python
# 使用自定义颜色
custom_colors = ['#1f77b4', '#ff7f0e', '#2ca02c', '#d62728']
sc.pl.umap(adata, color='leiden', palette=custom_colors,
           save='_custom_colors.pdf')

# 连续色图
sc.pl.umap(adata, color='CD3D', cmap='viridis', save='_viridis.pdf')
sc.pl.umap(adata, color='CD3D', cmap='RdBu_r', save='_rdbu.pdf')
```

### 移除坐标轴与边框

```python
# 无坐标轴的简洁图
sc.pl.umap(adata, color='leiden', frameon=False,
           save='_clean.pdf')

# 无图例
sc.pl.umap(adata, color='leiden', legend_loc=None,
           save='_no_legend.pdf')
```

## 图表导出

### 保存单个图表

```python
# 通过save参数自动保存
sc.pl.umap(adata, color='leiden', save='_leiden.pdf')
# 保存路径: sc.settings.figdir + 'umap_leiden.pdf'

# 手动保存
import matplotlib.pyplot as plt
fig = sc.pl.umap(adata, color='leiden', show=False, return_fig=True)
fig.savefig('figures/my_umap.pdf', dpi=300, bbox_inches='tight')
```

### 批量导出

```python
# 保存多版本图表
for gene in ['CD3D', 'CD14', 'MS4A1']:
    sc.pl.umap(adata, color=gene, save=f'_{gene}.pdf')
```

## 常用定制参数

### 布局参数
- `figsize`: 图表尺寸 (宽, 高)
- `frameon`: 显示图表边框
- `title`: 图表标题
- `legend_loc`: 图例位置 ('right margin', 'on data', 'best' 或 None)
- `legend_fontsize`: 图例字体大小
- `size`: 点大小

### 颜色参数
- `color`: 着色变量
- `palette`: 调色板 (如 'Set1', 'viridis')
- `cmap`: 连续变量色图
- `vmin`, `vmax`: 颜色标尺范围
- `use_raw`: 使用原始计数计算基因表达

### 保存参数
- `save`: 保存文件后缀
- `show`: 是否显示图表
- `dpi`: 栅格格式分辨率

## 出版图表技巧

1. **使用矢量格式**: PDF或SVG实现无损缩放
2. **高分辨率**: 栅格图像设置dpi=300+
3. **风格统一**: 跨图表使用相同调色板
4. **清晰标注**: 确保基因名和细胞类型可读
5. **白色背景**: 出版级图表使用 `facecolor='white'`
6. **简化元素**: `frameon=False` 获得更简洁外观
7. **图例布局**: 'on data' 位置节省空间
8. **色盲友好**: 选用 'colorblind' 或 'Set2' 等调色板
