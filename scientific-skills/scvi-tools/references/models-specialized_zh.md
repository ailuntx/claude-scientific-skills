# 专业模态模型

本文档介绍 scvi-tools 中针对专业单细胞数据模态的模型。

## MethylVI / MethylANVI（甲基化分析）

**用途**：分析单细胞亚硫酸氢盐测序（scBS-seq）数据中的DNA甲基化。

**核心特性**：
- 在单细胞分辨率下建模甲基化模式
- 处理甲基化数据的稀疏性
- 甲基化实验的批次校正
- 细胞类型注释的标签迁移（MethylANVI）

**适用场景**：
- 分析 scBS-seq 或类似甲基化数据
- 研究跨细胞类型的DNA甲基化模式
- 整合跨批次的甲基化数据
- 基于甲基化谱的细胞类型注释

**数据要求**：
- 甲基化计数矩阵（每个CpG位点的甲基化读数 vs 总读数）
- 格式：细胞 × CpG位点（含甲基化比率或计数）

### MethylVI（无监督）

**基础用法**：
```python
import scvi

# 设置甲基化数据
scvi.model.METHYLVI.setup_anndata(
    adata,
    layer="methylation_counts",  # 甲基化数据
    batch_key="batch"
)

model = scvi.model.METHYLVI(adata)
model.train()

# 获取潜在表征
latent = model.get_latent_representation()

# 获取标准化甲基化值
normalized_meth = model.get_normalized_methylation()
```

### MethylANVI（半监督细胞类型分析）

**基础用法**：
```python
# 带细胞类型标签的设置
scvi.model.METHYLANVI.setup_anndata(
    adata,
    layer="methylation_counts",
    batch_key="batch",
    labels_key="cell_type",
    unlabeled_category="Unknown"
)

model = scvi.model.METHYLANVI(adata)
model.train()

# 预测细胞类型
predictions = model.predict()
```

**关键参数**：
- `n_latent`：潜在空间维度
- `region_factors`：建模区域特异性效应

**应用场景**：
- 表观遗传异质性分析
- 通过甲基化识别细胞类型
- 与基因表达数据整合（独立分析）
- 差异甲基化分析

## CytoVI（流式与质谱流式分析）

**用途**：流式细胞术和质谱流式（CyTOF）数据的批次校正与整合。

**核心特性**：
- 处理基于抗体的蛋白质测量
- 校正流式数据的批次效应
- 支持跨实验整合
- 专为高维蛋白质面板设计

**适用场景**：
- 分析流式或CyTOF数据
- 跨批次整合流式实验
- 蛋白质面板的批次校正
- 跨研究流式数据整合

**数据要求**：
- 蛋白质表达矩阵（细胞 × 蛋白质）
- 流式或CyTOF测量值
- 批次/实验注释

**基础用法**：
```python
scvi.model.CYTOVI.setup_anndata(
    adata,
    protein_expression_obsm_key="protein_expression",
    batch_key="batch"
)

model = scvi.model.CYTOVI(adata)
model.train()

# 获取批次校正表征
latent = model.get_latent_representation()

# 获取标准化蛋白值
normalized = model.get_normalized_expression()
```

**关键参数**：
- `n_latent`：潜在空间维度
- `n_layers`：网络深度

**典型工作流**：
```python
import scanpy as sc

# 1. 加载流式数据
adata = sc.read_h5ad("cytof_data.h5ad")

# 2. 训练CytoVI
scvi.model.CYTOVI.setup_anndata(
    adata,
    protein_expression_obsm_key="protein",
    batch_key="experiment"
)
model = scvi.model.CYTOVI(adata)
model.train()

# 3. 获取批次校正值
latent = model.get_latent_representation()
adata.obsm["X_CytoVI"] = latent

# 4. 下游分析
sc.pp.neighbors(adata, use_rep="X_CytoVI")
sc.tl.umap(adata)
sc.tl.leiden(adata)

# 5. 可视化批次校正
sc.pl.umap(adata, color=["batch", "leiden"])
```

## SysVI（系统级整合）

**用途**：在保留生物变异的前提下进行批次效应校正。

**核心特性**：
- 专业批次整合方法
- 消除技术效应同时保留生物信号
- 专为大规模整合研究设计

**适用场景**：
- 大规模多批次整合
- 需保留细微生物变异
- 跨多研究的系统级分析

**基础用法**：
```python
scvi.model.SYSVI.setup_anndata(
    adata,
    layer="counts",
    batch_key="batch"
)

model = scvi.model.SYSVI(adata)
model.train()

latent = model.get_latent_representation()
```

## Decipher（轨迹推断）

**用途**：单细胞数据的轨迹推断与拟时序分析。

**核心特性**：
- 学习细胞轨迹与分化路径
- 拟时序估计
- 考虑轨迹结构的不确定性
- 兼容scVI嵌入

**适用场景**：
- 研究细胞分化
- 时序或发育数据集
- 理解细胞状态转换
- 识别发育中的分支点

**基础用法**：
```python
# 通常在scVI嵌入后使用
scvi_model = scvi.model.SCVI(adata)
scvi_model.train()

# Decipher轨迹分析
scvi.model.DECIPHER.setup_anndata(adata)
decipher_model = scvi.model.DECIPHER(adata, scvi_model)
decipher_model.train()

# 获取拟时序
pseudotime = decipher_model.get_pseudotime()
adata.obs["pseudotime"] = pseudotime
```

**可视化**：
```python
import scanpy as sc

# 在UMAP上绘制拟时序
sc.pl.umap(adata, color="pseudotime", cmap="viridis")

# 沿拟时序的基因表达
sc.pl.scatter(adata, x="pseudotime", y="gene_of_interest")
```

## peRegLM（峰值调控线性模型）

**用途**：建立染色质可及性与基因表达的调控关联。

**核心特性**：
- 关联ATAC-seq峰值与基因表达
- 识别调控关系
- 适用于配对多组学数据

**适用场景**：
- 多组学数据（同细胞RNA+ATAC）
- 理解基因调控机制
- 关联峰值与靶基因
- 构建调控网络

**基础用法**：
```python
# 需配对RNA+ATAC数据
scvi.model.PEREGLM.setup_anndata(
    multiome_adata,
    rna_layer="counts",
    atac_layer="atac_counts"
)

model = scvi.model.PEREGLM(multiome_adata)
model.train()

# 获取峰值-基因关联
peak_gene_links = model.get_regulatory_links()
```

## 模型专属最佳实践

### MethylVI/MethylANVI
1. **稀疏性**：甲基化数据天然稀疏；模型已考虑此特性
2. **CpG筛选**：过滤覆盖度极低的CpG位点
3. **生物学解释**：考虑基因组上下文（启动子、增强子）
4. **整合**：多组学分析需先独立处理再整合结果

### CytoVI
1. **蛋白质质控**：移除低质量或无信息蛋白
2. **补偿校正**：分析前确保光谱补偿正确
3. **批次设计**：包含生物和技术重复
4. **对照**：使用对照样本验证批次校正

### SysVI
1. **样本量**：专为大规模整合设计
2. **批次定义**：精确定义批次结构
3. **生物学验证**：确认生物信号保留

### Decipher
1. **起点**：若已知轨迹起点需明确定义
2. **分支**：指定预期分支数量
3. **验证**：使用已知标记基因验证拟时序
4. **整合**：与scVI嵌入协同效果佳

## 与其他模型整合

专业模型常组合使用：

**甲基化+表达**：
```python
# 分别分析后整合
methylvi_model = scvi.model.METHYLVI(meth_adata)
scvi_model = scvi.model.SCVI(rna_adata)

# 在分析层整合结果
# 例如：关联甲基化与表达模式
```

**流式+CITE-seq**：
```python
# CytoVI处理流式/CyTOF
cyto_model = scvi.model.CYTOVI(cyto_adata)

# totalVI处理CITE-seq
cite_model = scvi.model.TOTALVI(cite_adata)

# 跨平台比较蛋白测量
```

**ATAC+RNA（多组学）**：
```python
# MultiVI联合分析
multivi_model = scvi.model.MULTIVI(multiome_adata)

# peRegLM分析调控关联
pereglm_model = scvi.model.PEREGLM(multiome_adata)
```

## 选择专业模型

### 决策树

1. **数据类型？**
   - 甲基化 → MethylVI/MethylANVI
   - 流式/CyTOF → CytoVI
   - 轨迹 → Decipher
   - 多批次整合 → SysVI
   - 调控关联 → peRegLM

2. **有标签数据？**
   - 有 → MethylANVI（甲基化）
   - 无 → MethylVI（甲基化）

3. **主要目标？**
   - 批次校正 → CytoVI, SysVI
   - 轨迹/拟时序 → Decipher
   - 峰值-基因关联 → peRegLM
   - 甲基化模式 → MethylVI/ANVI

## 示例：完整甲基化分析流程

```python
import scvi
import scanpy as sc

# 1. 加载甲基化数据
meth_adata = sc.read_h5ad("methylation_data.h5ad")

# 2. 质控：过滤低覆盖CpG位点
sc.pp.filter_genes(meth_adata, min_cells=10)

# 3. 设置MethylVI
scvi.model.METHYLVI.setup_anndata(
    meth_adata,
    layer="methylation",
    batch_key="batch"
)

# 4. 训练模型
model = scvi.model.METHYLVI(meth_adata, n_latent=15)
model.train(max_epochs=400)

# 5. 获取潜在表征
latent = model.get_latent_representation()
meth_adata.obsm["X_MethylVI"] = latent

# 6. 聚类分析
sc.pp.neighbors(meth_adata, use_rep="X_MethylVI")
sc.tl.umap(meth_adata)
sc.tl.leiden(meth_adata)

# 7. 差异甲基化分析
dm_results = model.differential_methylation(
    groupby="leiden",
    group1="0",
    group2="1"
)

# 8. 保存结果
model.save("methylvi_model")
meth_adata.write("methylation_analyzed.h5ad")
```

## 外部工具整合

部分专业模型通过外部包提供：

**SOLO**（双联体检测）：
```python
from scvi.external import SOLO

solo = SOLO.from_scvi_model(scvi_model)
solo.train()
doublets = solo.predict()
```

**scArches**（参考映射）：
```python
from scvi.external import SCARCHES

# 用于迁移学习及查询-参考映射
```

这些外部工具扩展了 scvi-tools 的特定场景功能。

## 模型摘要表

| 模型 | 数据类型 | 主要用途 | 监督类型 |
|-------|-----------|-------------|-------------|
| MethylVI | 甲基化 | 无监督分析 | 无 |
| MethylANVI | 甲基化 | 细胞类型注释 | 半监督 |
| CytoVI | 流式 | 批次校正 | 无 |
| SysVI | scRNA-seq | 大规模整合 | 无 |
| Decipher | scRNA-seq | 轨迹推断 | 无 |
| peRegLM | 多组学 | 峰值-基因关联 | 无 |
| SOLO | scRNA-seq | 双联体检测 | 半监督 |
