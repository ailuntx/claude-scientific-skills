# 空间转录组学模型

本文档涵盖 scvi-tools 中用于分析空间分辨转录组数据的模型。

## DestVI（使用变分推断的空间转录组学解卷积）

**目的**：利用单细胞参考数据实现空间转录组学的多分辨率解卷积。

**核心特性**：
- 估计每个空间位置的细胞类型比例
- 使用单细胞RNA-seq参考数据进行解卷积
- 多分辨率分析（全局与局部模式）
- 考虑空间相关性
- 提供不确定性量化

**适用场景**：
- 解卷积 Visium 或类似空间转录组数据
- 拥有带细胞类型标签的scRNA-seq参考数据
- 需要将细胞类型映射到空间位置
- 研究细胞类型的空间组织结构
- 需要细胞类型丰度的概率估计

**数据要求**：
- **空间数据**：Visium 或类似基于点位的测量（目标数据）
- **单细胞参考**：带细胞类型注释的scRNA-seq数据
- 两数据集需共享基因集

**基础用法**：
```python
import scvi

# 步骤1：在单细胞参考上训练scVI
scvi.model.SCVI.setup_anndata(sc_adata, layer="counts")
sc_model = scvi.model.SCVI(sc_adata)
sc_model.train()

# 步骤2：设置空间数据
scvi.model.DESTVI.setup_anndata(
    spatial_adata,
    layer="counts"
)

# 步骤3：使用参考训练DestVI
model = scvi.model.DESTVI.from_rna_model(
    spatial_adata,
    sc_model,
    cell_type_key="cell_type"  # 参考数据中的细胞类型标签
)
model.train(max_epochs=2500)

# 步骤4：获取细胞类型比例
proportions = model.get_proportions()
spatial_adata.obsm["proportions"] = proportions

# 步骤5：获取细胞类型特异性表达
# 每个点位中特定细胞类型的基因表达
ct_expression = model.get_scale_for_ct("T cells")
```

**关键参数**：
- `amortization`：摊销策略（"both"、"latent"、"proportion"）
- `n_latent`：潜在维度（继承自scVI模型）

**输出结果**：
- `get_proportions()`：各点位的细胞类型比例
- `get_scale_for_ct(cell_type)`：细胞类型特异性表达模式
- `get_gamma()`：比例特异性基因表达缩放因子

**可视化**：
```python
import scanpy as sc
import matplotlib.pyplot as plt

# 空间可视化特定细胞类型比例
sc.pl.spatial(
    spatial_adata,
    color="T cells",  # 若比例已添加至.obs
    spot_size=150
)

# 或直接使用obsm
for ct in cell_types:
    plt.figure()
    sc.pl.spatial(
        spatial_adata,
        color=spatial_adata.obsm["proportions"][ct],
        title=f"{ct}比例"
    )
```

## Stereoscope

**目的**：通过概率建模实现空间转录组学的细胞类型解卷积。

**核心特性**：
- 基于参考的解卷积
- 细胞类型比例的概率框架
- 兼容多种空间技术
- 处理基因筛选与标准化

**适用场景**：
- 类似DestVI但方法更简化
- 使用参考数据解卷积空间数据
- 需要快速基础解卷积的替代方案

**基础用法**：
```python
scvi.model.STEREOSCOPE.setup_anndata(
    sc_adata,
    labels_key="cell_type",
    layer="counts"
)

# 在参考数据上训练
ref_model = scvi.model.STEREOSCOPE(sc_adata)
ref_model.train()

# 设置空间数据
scvi.model.STEREOSCOPE.setup_anndata(spatial_adata, layer="counts")

# 迁移至空间数据
spatial_model = scvi.model.STEREOSCOPE.from_reference_model(
    spatial_adata,
    ref_model
)
spatial_model.train()

# 获取比例
proportions = spatial_model.get_proportions()
```

## Tangram

**目的**：将单细胞数据空间映射并整合至空间位置。

**核心特性**：
- 将单细胞映射至空间坐标
- 学习单细胞与空间数据间的最优传输
- 空间位置基因填补
- 细胞类型映射

**适用场景**：
- 将scRNA-seq细胞映射至空间位置
- 填补空间数据中未测量的基因
- 单细胞分辨率下理解空间组织结构
- 整合scRNA-seq与空间转录组数据

**数据要求**：
- 带注释的单细胞RNA-seq数据
- 空间转录组数据
- 模态间共享基因集

**基础用法**：
```python
import tangram as tg

# 将细胞映射至空间位置
ad_map = tg.map_cells_to_space(
    adata_sc=sc_adata,
    adata_sp=spatial_adata,
    mode="cells",  # 或使用"clusters"进行细胞类型映射
    density_prior="rna_count_based"
)

# 获取映射矩阵（细胞×点位）
mapping = ad_map.X

# 将细胞注释投影至空间
tg.project_cell_annotations(
    ad_map,
    spatial_adata,
    annotation="cell_type"
)

# 在空间数据中填补基因
genes_to_impute = ["CD3D", "CD8A", "CD4"]
tg.project_genes(ad_map, spatial_adata, genes=genes_to_impute)
```

**可视化**：
```python
# 可视化细胞类型映射
sc.pl.spatial(
    spatial_adata,
    color="cell_type_projected",
    spot_size=100
)
```

## gimVI（高斯恒等多模态插补模型）

**目的**：空间与单细胞数据间的跨模态插补。

**核心特性**：
- 空间与单细胞数据的联合建模
- 填补空间数据中缺失基因
- 支持跨数据集查询
- 学习共享表征

**适用场景**：
- 填补空间数据中未测量的基因
- 空间与单细胞数据联合分析
- 模态间映射

**基础用法**：
```python
# 合并数据集
combined_adata = sc.concat([sc_adata, spatial_adata])

scvi.model.GIMVI.setup_anndata(
    combined_adata,
    layer="counts"
)

model = scvi.model.GIMVI(combined_adata)
model.train()

# 在空间数据中插补基因
imputed = model.get_imputed_values(spatial_indices)
```

## scVIVA（空间变分自编码器变异分析）

**目的**：分析空间数据中的细胞-环境关系。

**核心特性**：
- 建模细胞邻域与微环境
- 识别环境相关基因表达
- 考虑空间相关结构
- 细胞间相互作用分析

**适用场景**：
- 理解空间环境如何影响细胞
- 识别生态位特异性基因程序
- 细胞间相互作用研究
- 微环境分析

**数据要求**：
- 带坐标的空间转录组数据
- 细胞类型注释（可选）

**基础用法**：
```python
scvi.model.SCVIVA.setup_anndata(
    spatial_adata,
    layer="counts",
    spatial_key="spatial"  # .obsm中的坐标
)

model = scvi.model.SCVIVA(spatial_adata)
model.train()

# 获取环境表征
env_latent = model.get_environment_representation()

# 识别环境相关基因
env_genes = model.get_environment_specific_genes()
```

## ResolVI

**目的**：通过分辨率感知建模解决空间转录组噪声问题。

**核心特性**：
- 考虑空间分辨率效应
- 空间数据降噪
- 多尺度分析
- 提升下游分析质量

**适用场景**：
- 噪声较多的空间数据
- 多空间分辨率场景
- 分析前需降噪处理
- 提升数据质量

**基础用法**：
```python
scvi.model.RESOLVI.setup_anndata(
    spatial_adata,
    layer="counts",
    spatial_key="spatial"
)

model = scvi.model.RESOLVI(spatial_adata)
model.train()

# 获取降噪后表达
denoised = model.get_denoised_expression()
```

## 空间转录组学模型选择指南

### DestVI
**选择时机**：
- 需要基于参考的精细解卷积
- 拥有高质量scRNA-seq参考
- 需多分辨率分析
- 需要不确定性量化

**最佳适用**：Visium等点位技术

### Stereoscope
**选择时机**：
- 需要更简单快速解卷积
- 基础细胞类型比例估计
- 计算资源有限

**最佳适用**：快速解卷积任务

### Tangram
**选择时机**：
- 需要单细胞分辨率映射
- 需填补大量基因
- 关注细胞定位
- 倾向最优传输方法

**最佳适用**：精细空间映射

### gimVI
**选择时机**：
- 需要双向插补
- 空间与单细胞联合建模
- 跨数据集查询

**最佳适用**：整合与插补

### scVIVA
**选择时机**：
- 研究细胞微环境
- 细胞间相互作用分析
- 邻域效应研究

**最佳适用**：微环境研究

### ResolVI
**选择时机**：
- 数据质量较差
- 需降噪处理
- 多尺度分析

**最佳适用**：噪声数据预处理

## 完整工作流：使用DestVI进行空间解卷积

```python
import scvi
import scanpy as sc
import squidpy as sq

# ===== 第一部分：准备单细胞参考 =====
# 加载并处理scRNA-seq参考
sc_adata = sc.read_h5ad("reference_scrna.h5ad")

# 质控与过滤
sc.pp.filter_genes(sc_adata, min_cells=10)
sc.pp.highly_variable_genes(sc_adata, n_top_genes=4000)

# 在参考上训练scVI
scvi.model.SCVI.setup_anndata(
    sc_adata,
    layer="counts",
    batch_key="batch"
)

sc_model = scvi.model.SCVI(sc_adata)
sc_model.train(max_epochs=400)

# ===== 第二部分：加载空间数据 =====
spatial_adata = sc.read_visium("path/to/visium")
spatial_adata.var_names_make_unique()

# 空间数据质控
sc.pp.filter_genes(spatial_adata, min_cells=10)

# ===== 第三部分：运行DestVI =====
scvi.model.DESTVI.setup_anndata(
    spatial_adata,
    layer="counts"
)

destvi_model = scvi.model.DESTVI.from_rna_model(
    spatial_adata,
    sc_model,
    cell_type_key="cell_type"
)

destvi_model.train(max_epochs=2500)

# ===== 第四部分：提取结果 =====
# 获取比例
proportions = destvi_model.get_proportions()
spatial_adata.obsm["proportions"] = proportions

# 将比例添加至.obs便于绘图
for i, ct in enumerate(sc_model.adata.obs["cell_type"].cat.categories):
    spatial_adata.obs[f"prop_{ct}"] = proportions[:, i]

# ===== 第五部分：可视化 =====
# 绘制特定细胞类型
cell_types = ["T cells", "B cells", "Macrophages"]

for ct in cell_types:
    sc.pl.spatial(
        spatial_adata,
        color=f"prop_{ct}",
        title=f"{ct}比例",
        spot_size=150,
        cmap="viridis"
    )

# ===== 第六部分：空间分析 =====
# 计算空间邻域
sq.gr.spatial_neighbors(spatial_adata)

# 细胞类型的空间自相关
for ct in cell_types:
    sq.gr.spatial_autocorr(
        spatial_adata,
        attr="obs",
        mode="moran",
        genes=[f"prop_{ct}"]
    )

# ===== 第七部分：保存结果 =====
destvi_model.save("destvi_model")
spatial_adata.write("spatial_deconvolved.h5ad")
```

## 空间分析最佳实践

1. **参考质量**：使用高质量、注释完善的scRNA-seq参考
2. **基因重叠**：确保参考与空间数据有足够共享基因
3. **空间坐标**：在`.obsm["spatial"]`中正确注册空间坐标
4. **验证**：使用已知标记基因验证解卷积结果
5. **可视化**：始终空间可视化结果以检查生物学合理性
6. **细胞类型粒度**：考虑合适的细胞类型分辨率
7. **计算资源**：空间模型可能消耗大量内存
8. **质控**：分析前过滤低质量点位
