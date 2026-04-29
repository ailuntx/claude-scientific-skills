---
name: scvi-tools
description: 用于单细胞组学的深度生成模型。适用于需要概率批次校正（scVI）、迁移学习、不确定性差异表达或多模态整合（TOTALVI, MultiVI）的场景。擅长高级建模、批次效应处理和多模态数据分析。标准分析流程请使用scanpy。
license: BSD-3-Clause 许可证
metadata:
    skill-author: K-Dense Inc.
---

# scvi-tools

## 概述

scvi-tools 是一个基于 Python 的综合性框架，专注于单细胞基因组学中的概率模型。构建于 PyTorch 和 PyTorch Lightning 之上，通过变分推断提供深度生成模型，用于分析多种单细胞数据模态。

## 适用场景

在以下场景使用本工具：
- 分析单细胞 RNA-seq 数据（降维、批次校正、整合）
- 处理单细胞 ATAC-seq 或染色质可及性数据
- 整合多模态数据（CITE-seq、多组学、配对/非配对数据集）
- 分析空间转录组数据（解卷积、空间映射）
- 执行单细胞数据的差异表达分析
- 进行细胞类型注释或迁移学习任务
- 处理特殊单细胞模态（甲基化、流式细胞术、RNA速率）
- 构建自定义单细胞分析概率模型

## 核心能力

scvi-tools 按数据模态组织模型：

### 1. 单细胞 RNA-seq 分析
核心模型用于表达分析、批次校正和整合。详见 `references/models-scrna-seq.md`：
- **scVI**：无监督降维与批次校正
- **scANVI**：半监督细胞类型注释与整合
- **AUTOZI**：零膨胀检测与建模
- **VeloVI**：RNA速率分析
- **contrastiveVI**：扰动效应分离

### 2. 染色质可及性（ATAC-seq）
单细胞染色质数据分析模型。详见 `references/models-atac-seq.md`：
- **PeakVI**：基于峰值的ATAC-seq分析与整合
- **PoissonVI**：定量片段计数建模
- **scBasset**：结合基序分析的深度学习方法

### 3. 多模态与多组学整合
多数据类型联合分析。详见 `references/models-multimodal.md`：
- **totalVI**：CITE-seq蛋白与RNA联合建模
- **MultiVI**：配对/非配对多组学整合
- **MrVI**：多分辨率跨样本分析

### 4. 空间转录组学
空间分辨转录组分析。详见 `references/models-spatial.md`：
- **DestVI**：多分辨率空间解卷积
- **Stereoscope**：细胞类型解卷积
- **Tangram**：空间映射与整合
- **scVIVA**：细胞-环境关系分析

### 5. 特殊模态
专用分析工具。详见 `references/models-specialized.md`：
- **MethylVI/MethylANVI**：单细胞甲基化分析
- **CytoVI**：流式/质谱流式批次校正
- **Solo**：双联体检测
- **CellAssign**：基于标记的细胞类型注释

## 典型工作流

所有 scvi-tools 模型遵循统一 API 模式：

```python
# 1. 加载并预处理数据（AnnData格式）
import scvi
import scanpy as sc

adata = scvi.data.heart_cell_atlas_subsampled()
sc.pp.filter_genes(adata, min_counts=3)
sc.pp.highly_variable_genes(adata, n_top_genes=1200)

# 2. 向模型注册数据（指定图层、协变量）
scvi.model.SCVI.setup_anndata(
    adata,
    layer="counts",  # 使用原始计数而非对数标准化数据
    batch_key="batch",
    categorical_covariate_keys=["donor"],
    continuous_covariate_keys=["percent_mito"]
)

# 3. 创建并训练模型
model = scvi.model.SCVI(adata)
model.train()

# 4. 提取潜在表示与标准化值
latent = model.get_latent_representation()
normalized = model.get_normalized_expression(library_size=1e4)

# 5. 存储至AnnData供下游分析
adata.obsm["X_scVI"] = latent
adata.layers["scvi_normalized"] = normalized

# 6. 使用scanpy进行下游分析
sc.pp.neighbors(adata, use_rep="X_scVI")
sc.tl.umap(adata)
sc.tl.leiden(adata)
```

**核心设计原则：**
- **需原始计数**：模型需未标准化计数数据以获得最佳性能
- **统一API**：跨模型一致接口（注册→训练→提取）
- **AnnData中心化**：与scanpy生态无缝集成
- **GPU加速**：自动利用可用GPU资源
- **批次校正**：通过协变量注册处理技术变异

## 常见分析任务

### 差异表达
使用学习到的生成模型进行概率差异表达分析：

```python
de_results = model.differential_expression(
    groupby="cell_type",
    group1="TypeA",
    group2="TypeB",
    mode="change",  # 使用复合假设检验
    delta=0.25      # 最小效应量阈值
)
```

详见 `references/differential-expression.md` 获取方法学细节与结果解读。

### 模型持久化
保存与加载训练模型：

```python
# 保存模型
model.save("./model_directory", overwrite=True)

# 加载模型
model = scvi.model.SCVI.load("./model_directory", adata=adata)
```

### 批次校正与整合
跨批次或研究整合数据集：

```python
# 注册批次信息
scvi.model.SCVI.setup_anndata(adata, batch_key="study")

# 模型自动学习批次校正表示
model = scvi.model.SCVI(adata)
model.train()
latent = model.get_latent_representation()  # 批次校正后表示
```

## 理论基础

scvi-tools 构建于：
- **变分推断**：通过近似后验分布实现可扩展贝叶斯推断
- **深度生成模型**：学习复杂数据分布的VAE架构
- **摊销推断**：跨细胞共享神经网络实现高效学习
- **概率建模**：严谨的不确定性量化与统计检验

详见 `references/theoretical-foundations.md` 获取数学框架的详细背景。

## 附加资源

- **工作流**：`references/workflows.md` 包含常用流程、最佳实践、超参调优与GPU优化
- **模型参考**：各模型类别的详细文档位于 `references/` 目录
- **官方文档**：https://docs.scvi-tools.org/en/stable/
- **教程**：https://docs.scvi-tools.org/en/stable/tutorials/index.html
- **API参考**：https://docs.scvi-tools.org/en/stable/api/index.html

## 安装

```bash
uv pip install scvi-tools
# GPU支持版本
uv pip install scvi-tools[cuda]
```

## 最佳实践

1. **使用原始计数**：始终向模型提供未标准化计数数据
2. **基因过滤**：分析前移除低表达基因（如 `min_counts=3`）
3. **注册协变量**：在 `setup_anndata` 中包含已知技术因素（批次、供体等）
4. **特征选择**：使用高变基因提升性能
5. **模型保存**：始终保存训练模型避免重复训练
6. **GPU使用**：大型数据集启用GPU加速（`accelerator="gpu"`）
7. **Scanpy集成**：将输出存储至AnnData对象供下游分析
