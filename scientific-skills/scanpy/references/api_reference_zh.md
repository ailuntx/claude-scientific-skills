# Scanpy API 快速参考

按模块整理的常用 scanpy 函数快速参考。

## 导入约定

```python
import scanpy as sc
```

## 数据读写 (sc.read_*)

### 读取函数

```python
sc.read_10x_h5(filename)                    # 读取 10X HDF5 文件
sc.read_10x_mtx(path)                       # 读取 10X mtx 目录
sc.read_h5ad(filename)                      # 读取 h5ad (AnnData) 文件
sc.read_csv(filename)                       # 读取 CSV 文件
sc.read_excel(filename)                     # 读取 Excel 文件
sc.read_loom(filename)                      # 读取 loom 文件
sc.read_text(filename)                      # 读取文本文件
sc.read_visium(path)                        # 读取 Visium 空间数据
```

### 写入函数

```python
adata.write_h5ad(filename)                  # 写入 h5ad 格式
adata.write_csvs(dirname)                   # 写入 CSV 文件集
adata.write_loom(filename)                  # 写入 loom 格式
adata.write_zarr(filename)                  # 写入 zarr 格式
```

## 预处理 (sc.pp.*)

### 质量控制

```python
sc.pp.calculate_qc_metrics(adata, qc_vars=['mt'], inplace=True)
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_genes(adata, min_cells=3)
```

### 标准化与转换

```python
sc.pp.normalize_total(adata, target_sum=1e4)    # 标准化至目标总和
sc.pp.log1p(adata)                               # Log(x + 1) 转换
sc.pp.sqrt(adata)                                # 平方根转换
```

### 特征选择

```python
sc.pp.highly_variable_genes(adata, min_mean=0.0125, max_mean=3, min_disp=0.5)
sc.pp.highly_variable_genes(adata, flavor='seurat_v3', n_top_genes=2000)
```

### 缩放与回归

```python
sc.pp.scale(adata, max_value=10)                      # 缩放到单位方差
sc.pp.regress_out(adata, ['total_counts', 'pct_counts_mt'])  # 回归消除干扰变异
```

### 降维（预处理）

```python
sc.pp.pca(adata, n_comps=50)                     # 主成分分析
sc.pp.neighbors(adata, n_neighbors=10, n_pcs=40) # 计算邻域图
```

### 批次校正

```python
sc.pp.combat(adata, key='batch')                 # ComBat 批次校正
```

## 工具 (sc.tl.*)

### 降维

```python
sc.tl.pca(adata, svd_solver='arpack')            # PCA
sc.tl.umap(adata)                                 # UMAP 嵌入
sc.tl.tsne(adata)                                 # t-SNE 嵌入
sc.tl.diffmap(adata)                              # 扩散映射
sc.tl.draw_graph(adata, layout='fa')             # 力导向图
```

### 聚类

```python
sc.tl.leiden(adata, resolution=0.5)              # Leiden 聚类（推荐）
sc.tl.louvain(adata, resolution=0.5)             # Louvain 聚类
sc.tl.kmeans(adata, n_clusters=10)               # K-means 聚类
```

### 标记基因与差异表达

```python
sc.tl.rank_genes_groups(adata, groupby='leiden', method='wilcoxon')
sc.tl.rank_genes_groups(adata, groupby='leiden', method='t-test')
sc.tl.rank_genes_groups(adata, groupby='leiden', method='logreg')

# 获取结果为数据框
sc.get.rank_genes_groups_df(adata, group='0')
```

### 轨迹推断

```python
sc.tl.paga(adata, groups='leiden')               # PAGA 轨迹
sc.tl.dpt(adata)                                  # 扩散伪时间
```

### 基因评分

```python
sc.tl.score_genes(adata, gene_list, score_name='score')
sc.tl.score_genes_cell_cycle(adata, s_genes, g2m_genes)
```

### 嵌入与投影

```python
sc.tl.ingest(adata, adata_ref)                   # 映射到参考集
sc.tl.embedding_density(adata, basis='umap', groupby='leiden')
```

## 绘图 (sc.pl.*)

### 基础嵌入图

```python
sc.pl.umap(adata, color='leiden')                # UMAP 图
sc.pl.tsne(adata, color='gene_name')             # t-SNE 图
sc.pl.pca(adata, color='leiden')                 # PCA 图
sc.pl.diffmap(adata, color='leiden')             # 扩散映射图
```

### 热图与点图

```python
sc.pl.heatmap(adata, var_names=genes, groupby='leiden')
sc.pl.dotplot(adata, var_names=genes, groupby='leiden')
sc.pl.matrixplot(adata, var_names=genes, groupby='leiden')
sc.pl.stacked_violin(adata, var_names=genes, groupby='leiden')
```

### 小提琴图与散点图

```python
sc.pl.violin(adata, keys=['gene1', 'gene2'], groupby='leiden')
sc.pl.scatter(adata, x='gene1', y='gene2', color='leiden')
```

### 标记基因可视化

```python
sc.pl.rank_genes_groups(adata, n_genes=25, sharey=False)
sc.pl.rank_genes_groups_violin(adata, groups='0')
sc.pl.rank_genes_groups_heatmap(adata, n_genes=10)
sc.pl.rank_genes_groups_dotplot(adata, n_genes=5)
```

### 轨迹可视化

```python
sc.pl.paga(adata, color='leiden')                # PAGA 图
sc.pl.dpt_timeseries(adata)                      # DPT 时间序列
```

### 质控图

```python
sc.pl.highest_expr_genes(adata, n_top=20)
sc.pl.violin(adata, ['n_genes_by_counts', 'total_counts', 'pct_counts_mt'])
sc.pl.scatter(adata, x='total_counts', y='n_genes_by_counts')
```

### 高级绘图

```python
sc.pl.dendrogram(adata, groupby='leiden')
sc.pl.correlation_matrix(adata, groupby='leiden')
sc.pl.tracksplot(adata, var_names=genes, groupby='leiden')
```

## 通用参数

### 颜色参数
- `color`: 着色变量（基因名/观测列）
- `use_raw`: 使用 adata 的 `.raw` 属性
- `palette`: 使用的调色板
- `vmin`, `vmax`: 颜色标尺范围

### 布局参数
- `basis`: 嵌入基础（'umap'、'tsne'、'pca' 等）
- `legend_loc`: 图例位置（'on data'、'right margin' 等）
- `size`: 点大小
- `alpha`: 点透明度

### 保存参数
- `save`: 保存文件名
- `show`: 是否显示图像

## AnnData 结构

```python
adata.X                    # 表达矩阵（细胞×基因）
adata.obs                  # 细胞注释（数据框）
adata.var                  # 基因注释（数据框）
adata.uns                  # 非结构化注释（字典）
adata.obsm                 # 多维细胞注释（如 PCA、UMAP）
adata.varm                 # 多维基因注释
adata.layers               # 附加数据层
adata.raw                  # 原始数据备份

# 访问
adata.obs_names            # 细胞条形码
adata.var_names            # 基因名称
adata.shape                # (细胞数, 基因数)

# 切片
adata[cell_indices, gene_indices]
adata[:, adata.var_names.isin(gene_list)]
adata[adata.obs['leiden'] == '0', :]
```

## 设置

```python
sc.settings.verbosity = 3              # 0=错误, 1=警告, 2=信息, 3=提示
sc.settings.set_figure_params(dpi=80, facecolor='white')
sc.settings.autoshow = False           # 不自动显示图像
sc.settings.autosave = True            # 自动保存图像
sc.settings.figdir = './figures/'      # 图像目录
sc.settings.cachedir = './cache/'      # 缓存目录
sc.settings.n_jobs = 8                 # 并行任务数
```

## 实用工具

```python
sc.logging.print_versions()            # 打印版本信息
sc.logging.print_memory_usage()        # 打印内存使用
adata.copy()                           # 创建 AnnData 对象副本
adata.concatenate([adata1, adata2])    # 拼接 AnnData 对象
```
