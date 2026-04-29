# ATAC-seq与染色质可及性模型

本文档涵盖scvi-tools中分析单细胞ATAC-seq和染色质可及性数据的模型。

## PeakVI

**用途**：使用峰计数分析并整合单细胞ATAC-seq数据。

**核心特性**：
- 专为scATAC-seq峰数据设计的变分自编码器
- 学习染色质可及性的低维表征
- 实现跨样本批次校正
- 支持差异可及性检验
- 整合多个ATAC-seq数据集

**适用场景**：
- 分析scATAC-seq峰计数矩阵
- 整合多个ATAC-seq实验
- 染色质可及性数据的批次校正
- ATAC-seq降维处理
- 细胞类型或条件间的差异可及性分析

**数据要求**：
- 峰计数矩阵（细胞×峰）
- 峰可及性的二元或计数数据
- 批次/样本注释（可选，用于批次校正）

**基础用法**：
```python
import scvi

# 准备数据（峰应位于adata.X）
# 可选：过滤低表达峰
sc.pp.filter_genes(adata, min_cells=3)

# 数据设置
scvi.model.PEAKVI.setup_anndata(
    adata,
    batch_key="batch"
)

# 训练模型
model = scvi.model.PEAKVI(adata)
model.train()

# 获取潜在表征（经批次校正）
latent = model.get_latent_representation()
adata.obsm["X_PeakVI"] = latent

# 差异可及性分析
da_results = model.differential_accessibility(
    groupby="cell_type",
    group1="TypeA",
    group2="TypeB"
)
```

**关键参数**：
- `n_latent`：潜在空间维度（默认：10）
- `n_hidden`：每隐藏层节点数（默认：128）
- `n_layers`：隐藏层数量（默认：1）
- `region_factors`：是否学习区域特异性因子（默认：True）
- `latent_distribution`：潜在空间分布（"normal"或"ln"）

**输出结果**：
- `get_latent_representation()`：细胞的低维嵌入
- `get_accessibility_estimates()`：标准化可及性值
- `differential_accessibility()`：差异峰的统计检验
- `get_region_factors()`：峰特异性缩放因子

**最佳实践**：
1. 过滤低质量峰（在极少数细胞中出现）
2. 整合多样本时包含批次信息
3. 使用潜在表征进行聚类和UMAP可视化
4. 技术变异高的数据集建议启用`region_factors=True`
5. 将潜在嵌入存储于`adata.obsm`以便scanpy下游分析

## PoissonVI

**用途**：scATAC-seq片段计数的定量分析（比峰计数更精细）。

**核心特性**：
- 直接建模片段计数（非二元峰数据）
- 计数数据的泊松分布建模
- 捕捉可及性的定量差异
- 实现染色质状态的细粒度分析

**适用场景**：
- 分析片段级ATAC-seq数据
- 需要定量可及性测量
- 分辨率高于二元峰识别
- 研究染色质可及性的渐进变化

**数据要求**：
- 片段计数矩阵（细胞×基因组区域）
- 计数数据（非二元）

**基础用法**：
```python
scvi.model.POISSONVI.setup_anndata(
    adata,
    batch_key="batch"
)

model = scvi.model.POISSONVI(adata)
model.train()

# 获取结果
latent = model.get_latent_representation()
accessibility = model.get_accessibility_estimates()
```

**与PeakVI的关键区别**：
- **PeakVI**：适用于标准峰计数矩阵，速度更快
- **PoissonVI**：适用于定量片段计数，分析更精细

**选择PoissonVI的场景**：
- 使用片段计数而非识别峰
- 需捕捉定量差异
- 拥有高质量、高覆盖度数据
- 关注细微可及性变化

## scBasset

**用途**：通过深度学习分析scATAC-seq，兼具可解释性与基序分析。

**核心特性**：
- 基于序列分析的卷积神经网络（CNN）架构
- 建模原始DNA序列，非仅峰计数
- 支持基序发现与转录因子（TF）结合预测
- 提供可解释的特征重要性
- 实现批次校正

**适用场景**：
- 需整合DNA序列信息
- 关注TF基序分析
- 需要可解释模型（识别驱动可及性的序列）
- 分析调控元件与TF结合位点
- 仅通过序列预测可及性

**数据要求**：
- 峰序列（从基因组提取）
- 峰可及性矩阵
- 基因组参考（用于序列提取）

**基础用法**：
```python
# scBasset需序列信息
# 首先提取峰序列
from scbasset import utils
sequences = utils.fetch_sequences(adata, genome="hg38")

# 设置与训练
scvi.model.SCBASSET.setup_anndata(
    adata,
    batch_key="batch"
)

model = scvi.model.SCBASSET(adata, sequences=sequences)
model.train()

# 获取潜在表征
latent = model.get_latent_representation()

# 模型解释：识别重要序列/基序
importance_scores = model.get_feature_importance()
```

**关键参数**：
- `n_latent`：潜在空间维度
- `conv_layers`：卷积层数量
- `n_filters`：每卷积层滤波器数
- `filter_size`：卷积滤波器尺寸

**高级功能**：
- **计算机突变分析**：预测序列变化对可及性的影响
- **基序富集**：识别可及区域的富集TF基序
- **批次校正**：类似其他scvi-tools模型
- **迁移学习**：在新数据集上微调

**可解释性工具**：
```python
# 获取序列重要性评分
importance = model.get_sequence_importance(region_indices=[0, 1, 2])

# 预测新序列可及性
predictions = model.predict_accessibility(new_sequences)
```

## ATAC-seq模型选择指南

### PeakVI
**适用场景**：
- 标准scATAC-seq分析流程
- 拥有峰计数矩阵（最常见格式）
- 需快速高效的批次校正
- 需要直接差异可及性分析
- 优先考虑计算效率

**优势**：
- 训练与推理速度快
- 经scATAC-seq验证的可靠性
- 轻松集成scanpy工作流
- 稳健的批次校正

### PoissonVI
**适用场景**：
- 拥有片段级计数数据
- 需要定量可及性测量
- 关注细微差异
- 拥有高覆盖度、高质量数据

**优势**：
- 提供更精细的定量信息
- 更擅长梯度变化分析
- 适用于计数数据的统计模型

### scBasset
**适用场景**：
- 需整合DNA序列信息
- 需要生物学解释（基序、TF）
- 关注调控机制
- 具备CNN训练的计算资源
- 需预测新序列可及性

**优势**：
- 基于序列，具有生物学可解释性
- 内置基序与TF分析
- 具备预测建模能力
- 支持计算机扰动实验

## 工作流示例：完整ATAC-seq分析

```python
import scvi
import scanpy as sc

# 1. 加载并预处理ATAC-seq数据
adata = sc.read_h5ad("atac_data.h5ad")

# 2. 过滤低质量峰
sc.pp.filter_genes(adata, min_cells=10)

# 3. 设置并训练PeakVI
scvi.model.PEAKVI.setup_anndata(
    adata,
    batch_key="sample"
)

model = scvi.model.PEAKVI(adata, n_latent=20)
model.train(max_epochs=400)

# 4. 提取潜在表征
latent = model.get_latent_representation()
adata.obsm["X_PeakVI"] = latent

# 5. 下游分析
sc.pp.neighbors(adata, use_rep="X_PeakVI")
sc.tl.umap(adata)
sc.tl.leiden(adata, key_added="clusters")

# 6. 差异可及性分析
da_results = model.differential_accessibility(
    groupby="clusters",
    group1="0",
    group2="1"
)

# 7. 保存模型
model.save("peakvi_model")
```

## 与基因表达整合（RNA+ATAC）

对于配对多模态数据（同细胞RNA+ATAC），改用**MultiVI**：

```python
# 适用于10x Multiome等配对数据
scvi.model.MULTIVI.setup_anndata(
    adata,
    batch_key="sample",
    modality_key="modality"  # "RNA"或"ATAC"
)

model = scvi.model.MULTIVI(adata)
model.train()

# 获取联合潜在空间
latent = model.get_latent_representation()
```

详见`models-multimodal.md`获取多模态整合细节。

## ATAC-seq分析最佳实践

1. **质量控制**：
   - 过滤峰计数极低/极高的细胞
   - 移除在极少数细胞中出现的峰
   - 必要时过滤线粒体和性染色体峰

2. **批次校正**：
   - 整合多样本时务必包含`batch_key`
   - 考虑技术协变量（测序深度、TSS富集度）

3. **特征选择**：
   - 与RNA-seq不同，通常使用所有峰
   - 为提升效率可过滤极罕见峰

4. **潜在维度**：
   - 初始设置`n_latent=10-30`（依数据集复杂度调整）
   - 异质性高的数据集需更大值

5. **下游分析**：
   - 使用潜在表征进行聚类和可视化
   - 将峰关联至基因以进行调控分析
   - 对簇特异性峰执行基序富集

6. **计算考量**：
   - ATAC-seq矩阵通常极大（大量峰）
   - 初始探索可考虑下采样峰
   - 大型数据集使用GPU加速
