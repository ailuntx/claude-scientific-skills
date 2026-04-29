# 多模态与多组学整合模型

本文档涵盖 scvi-tools 中用于多数据模态联合分析的模型。

## totalVI（总体变分推断）

**用途**：联合分析 CITE-seq 数据（同一细胞的 RNA 与蛋白质同步测量）。

**核心特性**：
- 联合建模基因表达与蛋白质丰度
- 学习共享的低维表征
- 支持从 RNA 数据推算蛋白质
- 执行双模态差异表达分析
- 处理 RNA 和蛋白质层的批次效应

**适用场景**：
- 分析 CITE-seq 或 REAP-seq 数据
- RNA + 表面蛋白联合测量
- 推算缺失蛋白质
- 整合蛋白质与 RNA 信息
- 多批次 CITE-seq 整合

**数据要求**：
- AnnData 对象：基因表达数据位于 `.X` 或某图层
- 蛋白质测量数据位于 `.obsm["protein_expression"]`
- 双模态需测量相同细胞

**基础用法**：
```python
import scvi

# 数据准备 - 指定 RNA 和蛋白质图层
scvi.model.TOTALVI.setup_anndata(
    adata,
    layer="counts",  # RNA 计数
    protein_expression_obsm_key="protein_expression",  # 蛋白质计数
    batch_key="batch"
)

# 训练模型
model = scvi.model.TOTALVI(adata)
model.train()

# 获取联合潜在表征
latent = model.get_latent_representation()

# 获取双模态标准化值
rna_normalized = model.get_normalized_expression()
protein_normalized = model.get_normalized_expression(
    transform_batch="batch1",
    protein_expression=True
)

# 差异表达（支持 RNA 和蛋白质）
rna_de = model.differential_expression(groupby="cell_type")
protein_de = model.differential_expression(
    groupby="cell_type",
    protein_expression=True
)
```

**关键参数**：
- `n_latent`：潜在空间维度（默认：20）
- `n_layers_encoder`：编码器层数（默认：1）
- `n_layers_decoder`：解码器层数（默认：1）
- `protein_dispersion`：蛋白质离散度处理（"protein" 或 "protein-batch"）
- `empirical_protein_background_prior`：使用蛋白质经验背景先验

**高级功能**：

**蛋白质推算**：
```python
# 为仅含 RNA 的细胞推算缺失蛋白质
# （适用于将 RNA-seq 映射到 CITE-seq 参考）
protein_foreground = model.get_protein_foreground_probability()
imputed_proteins = model.get_normalized_expression(
    protein_expression=True,
    n_samples=25
)
```

**去噪处理**：
```python
# 获取双模态去噪计数
denoised_rna = model.get_normalized_expression(n_samples=25)
denoised_protein = model.get_normalized_expression(
    protein_expression=True,
    n_samples=25
)
```

**最佳实践**：
1. 存在环境蛋白质时使用经验背景先验
2. 异质蛋白质数据考虑蛋白质特异性离散度
3. 使用联合潜在空间进行聚类（优于单独 RNA）
4. 通过已知标记物验证蛋白质推算
5. 训练前检查蛋白质质控指标

## MultiVI（多模态变分推断）

**用途**：整合配对与非配对多组学数据（如 RNA + ATAC，含配对与非配对细胞）。

**核心特性**：
- 处理配对数据（相同细胞）与非配对数据（不同细胞）
- 整合多模态：RNA、ATAC、蛋白质等
- 缺失模态推算
- 学习跨模态共享表征
- 灵活的整合策略

**适用场景**：
- 10x Multiome 数据（配对 RNA + ATAC）
- 整合独立 RNA-seq 与 ATAC-seq 实验
- 部分细胞含双模态，部分仅含单模态
- 跨模态推算任务

**数据要求**：
- 含多模态的 AnnData 对象
- 模态标识（标记每个细胞的测量类型）
- 支持：
  - 全配对细胞（双模态）
  - 混合配对与非配对细胞
  - 完全非配对数据集

**基础用法**：
```python
# 准备含模态信息的数据
# adata.X 应包含所有特征（基因+峰）
# adata.var["modality"] 标识 "Gene" 或 "Peak"
# adata.obs["modality"] 标识每个细胞的模态

scvi.model.MULTIVI.setup_anndata(
    adata,
    batch_key="batch",
    modality_key="modality"  # 标识细胞模态的列
)

model = scvi.model.MULTIVI(adata)
model.train()

# 获取联合潜在表征
latent = model.get_latent_representation()

# 推算缺失模态
# 例如：为仅含 RNA 的细胞预测 ATAC
imputed_accessibility = model.get_accessibility_estimates(
    indices=rna_only_indices
)

# 获取标准化表达/可及性
rna_normalized = model.get_normalized_expression()
atac_normalized = model.get_accessibility_estimates()
```

**关键参数**：
- `n_genes`：基因特征数量
- `n_regions`：可及性区域数量
- `n_latent`：潜在维度（默认：20）

**整合场景**：

**场景 1：全配对（10x Multiome）**：
```python
# 所有细胞含 RNA 和 ATAC
# 单一模态标识："paired"
adata.obs["modality"] = "paired"
```

**场景 2：部分配对**：
```python
# 部分细胞双模态，部分仅 RNA，部分仅 ATAC
adata.obs["modality"] = ["RNA+ATAC", "RNA", "ATAC", ...]
```

**场景 3：完全非配对**：
```python
# 独立的 RNA 和 ATAC 实验
adata.obs["modality"] = ["RNA"] * n_rna + ["ATAC"] * n_atac
```

**高级用例**：

**跨模态预测**：
```python
# 从基因表达预测峰
accessibility_from_rna = model.get_accessibility_estimates(
    indices=rna_only_cells
)

# 从可及性预测基因
expression_from_atac = model.get_normalized_expression(
    indices=atac_only_cells
)
```

**模态特异性分析**：
```python
# 按模态分别分析
rna_subset = adata[adata.obs["modality"].str.contains("RNA")]
atac_subset = adata[adata.obs["modality"].str.contains("ATAC")]
```

## MrVI（多分辨率变分推断）

**用途**：考虑样本特异性和共享变异的多样本分析。

**核心特性**：
- 同步分析多个样本/条件
- 将变异分解为：
  - 共享变异（跨样本共有）
  - 样本特异性变异
- 支持样本间比较
- 识别样本特异性细胞状态

**适用场景**：
- 比较多个生物样本或条件
- 区分样本特异性与共享细胞状态
- 疾病 vs 健康样本比较
- 理解样本间异质性
- 多供体研究

**基础用法**：
```python
scvi.model.MRVI.setup_anndata(
    adata,
    layer="counts",
    batch_key="batch",
    sample_key="sample"  # 关键：定义生物样本
)

model = scvi.model.MRVI(adata, n_latent=10, n_latent_sample=5)
model.train()

# 获取表征
shared_latent = model.get_latent_representation()  # 跨样本共享
sample_specific = model.get_sample_specific_representation()

# 样本距离矩阵
sample_distances = model.get_sample_distances()
```

**关键参数**：
- `n_latent`：共享潜在空间维度
- `n_latent_sample`：样本特异性空间维度
- `sample_key`：定义生物样本的列

**分析流程**：
```python
# 1. 识别跨样本共享细胞类型
sc.pp.neighbors(adata, use_rep="X_MrVI_shared")
sc.tl.umap(adata)
sc.tl.leiden(adata, key_added="shared_clusters")

# 2. 分析样本特异性变异
sample_repr = model.get_sample_specific_representation()

# 3. 比较样本
distances = model.get_sample_distances()

# 4. 寻找样本富集基因
de_results = model.differential_expression(
    groupby="sample",
    group1="Disease",
    group2="Healthy"
)
```

**应用场景**：
- **多供体研究**：分离供体效应与细胞类型变异
- **疾病研究**：区分疾病特异性与共享生物学
- **时间序列**：分离时序变异与稳定变异
- **批次+生物学**：解耦技术与生物变异

## totalVI vs. MultiVI vs. MrVI：如何选择？

### totalVI
**适用**：CITE-seq（RNA + 蛋白质，相同细胞）
- 配对测量
- 单模态特征类型
- 重点：蛋白质推算、联合分析

### MultiVI
**适用**：多模态（RNA + ATAC 等）
- 配对/非配对/混合
- 不同特征类型
- 重点：跨模态整合与推算

### MrVI
**适用**：多样本 RNA-seq
- 单模态（RNA）
- 多个生物样本
- 重点：样本级变异分解

## 整合最佳实践

### CITE-seq（totalVI）
1. **蛋白质质控**：剔除低质量抗体
2. **背景扣除**：使用经验背景先验
3. **联合聚类**：使用联合潜在空间而非单独 RNA
4. **验证**：检查双模态已知标记物

### Multiome/多模态（MultiVI）
1. **特征过滤**：独立过滤基因与峰
2. **平衡模态**：确保各模态合理代表性
3. **模态权重**：考虑主导模态影响
4. **推算验证**：谨慎验证推算值

### 多样本（MrVI）
1. **样本定义**：明确定义生物样本
2. **样本规模**：需足够细胞/样本
3. **协变量处理**：区分批次与样本效应
4. **结果解读**：辨别技术与生物变异

## 完整示例：使用 totalVI 分析 CITE-seq

```python
import scvi
import scanpy as sc

# 1. 加载 CITE-seq 数据
adata = sc.read_h5ad("cite_seq.h5ad")

# 2. 质控与过滤
sc.pp.filter_genes(adata, min_cells=3)
sc.pp.highly_variable_genes(adata, n_top_genes=4000)

# 蛋白质质控
protein_counts = adata.obsm["protein_expression"]
# 剔除低质量蛋白质

# 3. 配置 totalVI
scvi.model.TOTALVI.setup_anndata(
    adata,
    layer="counts",
    protein_expression_obsm_key="protein_expression",
    batch_key="batch"
)

# 4. 训练
model = scvi.model.TOTALVI(adata, n_latent=20)
model.train(max_epochs=400)

# 5. 提取联合表征
latent = model.get_latent_representation()
adata.obsm["X_totalVI"] = latent

# 6. 基于联合空间聚类
sc.pp.neighbors(adata, use_rep="X_totalVI")
sc.tl.umap(adata)
sc.tl.leiden(adata, resolution=0.5)

# 7. 双模态差异表达
rna_de = model.differential_expression(
    groupby="leiden",
    group1="0",
    group2="1"
)

protein_de = model.differential_expression(
    groupby="leiden",
    group1="0",
    group2="1",
    protein_expression=True
)

# 8. 保存模型
model.save("totalvi_model")
```
