# 单细胞RNA测序模型

本文档涵盖scvi-tools中用于分析单细胞RNA测序数据的核心模型。

## scVI（单细胞变分推断）

**用途**：scRNA-seq数据的无监督分析、降维和批次校正。

**核心特性**：
- 基于变分自编码器（VAE）的深度生成模型
- 学习捕获生物变异的低维潜在表征
- 自动校正批次效应和技术协变量
- 支持标准化基因表达量估计
- 支持差异表达分析

**适用场景**：
- scRNA-seq数据集的初步探索与降维
- 整合多批次或多研究数据
- 生成批次校正后的表达矩阵
- 执行概率化差异表达分析

**基础用法**：
```python
import scvi

# 设置数据
scvi.model.SCVI.setup_anndata(
    adata,
    layer="counts",
    batch_key="batch"
)

# 训练模型
model = scvi.model.SCVI(adata, n_latent=30)
model.train()

# 提取结果
latent = model.get_latent_representation()
normalized = model.get_normalized_expression()
```

**关键参数**：
- `n_latent`：潜在空间维度（默认：10）
- `n_layers`：隐藏层数量（默认：1）
- `n_hidden`：每隐藏层节点数（默认：128）
- `dropout_rate`：神经网络丢弃率（默认：0.1）
- `dispersion`：基因特异性或细胞特异性离散度（"gene"或"gene-batch"）
- `gene_likelihood`：数据分布（"zinb"、"nb"、"poisson"）

**输出结果**：
- `get_latent_representation()`：批次校正的低维嵌入
- `get_normalized_expression()`：去噪标准化表达值
- `differential_expression()`：组间概率化差异表达检验
- `get_feature_correlation_matrix()`：基因间相关性估计

## scANVI（基于变分推断的单细胞注释）

**用途**：利用标记与未标记细胞进行半监督细胞类型注释与整合。

**核心特性**：
- 在scVI基础上扩展细胞类型标签功能
- 利用部分标记数据集进行注释迁移
- 同步执行批次校正与细胞类型预测
- 支持查询-参考映射

**适用场景**：
- 使用参考标签注释新数据集
- 从已标注数据集向未标注数据集迁移学习
- 联合分析标记与未标记细胞
- 构建带不确定性量化的细胞类型分类器

**基础用法**：
```python
# 方案1：从头训练
scvi.model.SCANVI.setup_anndata(
    adata,
    layer="counts",
    batch_key="batch",
    labels_key="cell_type",
    unlabeled_category="Unknown"
)
model = scvi.model.SCANVI(adata)
model.train()

# 方案2：基于预训练scVI初始化
scvi_model = scvi.model.SCVI(adata)
scvi_model.train()
scanvi_model = scvi.model.SCANVI.from_scvi_model(
    scvi_model,
    unlabeled_category="Unknown"
)
scanvi_model.train()

# 预测细胞类型
predictions = scanvi_model.predict()
```

**关键参数**：
- `labels_key`：包含细胞类型标签的`adata.obs`列名
- `unlabeled_category`：未标注细胞的标识标签
- 所有scVI参数均适用

**输出结果**：
- `predict()`：所有细胞的细胞类型预测
- `predict_proba()`：预测概率
- `get_latent_representation()`：细胞类型感知的潜在空间

## AUTOZI

**用途**：自动识别并建模scRNA-seq数据中的零膨胀基因。

**核心特性**：
- 区分生物零值与技术性丢失
- 识别具有零膨胀特征的基因
- 提供基因特异性零膨胀概率
- 通过校正丢失提升下游分析

**适用场景**：
- 检测受技术性丢失影响的基因
- 改进稀疏数据集的插补与标准化
- 量化数据中的零膨胀程度

**基础用法**：
```python
scvi.model.AUTOZI.setup_anndata(adata, layer="counts")
model = scvi.model.AUTOZI(adata)
model.train()

# 获取基因零膨胀概率
zi_probs = model.get_alphas_betas()
```

## VeloVI

**用途**：基于变分推断的RNA速率分析。

**核心特性**：
- 联合建模剪接/未剪接RNA计数
- 概率化估计RNA速率
- 校正技术噪音与批次效应
- 提供速率估计的不确定性量化

**适用场景**：
- 推断细胞动态与分化轨迹
- 分析剪接/未剪接计数数据
- 带批次校正的RNA速率分析

**基础用法**：
```python
import scvelo as scv

# 准备速率数据
scv.pp.filter_and_normalize(adata)
scv.pp.moments(adata)

# 训练VeloVI
scvi.model.VELOVI.setup_anndata(adata, spliced_layer="Ms", unspliced_layer="Mu")
model = scvi.model.VELOVI(adata)
model.train()

# 获取速率估计
latent_time = model.get_latent_time()
velocities = model.get_velocity()
```

## contrastiveVI

**用途**：从背景生物变异中分离扰动特异性变异。

**核心特性**：
- 区分共享变异（跨条件）与目标特异性变异
- 适用于扰动研究（药物处理、遗传扰动）
- 识别条件特异性基因程序
- 支持发现处理特异性效应

**适用场景**：
- 分析扰动实验（药物筛选、CRISPR等）
- 识别特异性响应处理的基因
- 分离处理效应与背景变异
- 比较对照与扰动条件

**基础用法**：
```python
scvi.model.CONTRASTIVEVI.setup_anndata(
    adata,
    layer="counts",
    batch_key="batch",
    categorical_covariate_keys=["condition"]  # 对照 vs 处理
)

model = scvi.model.CONTRASTIVEVI(
    adata,
    n_latent=10,        # 共享变异
    n_latent_target=5   # 目标特异性变异
)
model.train()

# 提取表征
shared = model.get_latent_representation(representation="shared")
target_specific = model.get_latent_representation(representation="target")
```

## CellAssign

**用途**：基于已知标记基因的细胞类型注释。

**核心特性**：
- 利用细胞类型标记基因先验知识
- 概率化分配细胞类型
- 处理标记基因重叠与模糊性
- 提供带不确定性的软分配

**适用场景**：
- 使用已知标记基因注释细胞
- 利用现有生物知识进行分类
- 无参考数据集但可获得标记基因列表的场景

**基础用法**：
```python
# 创建标记基因矩阵（细胞类型×基因）
marker_gene_mat = pd.DataFrame({
    "CD4 T细胞": [1, 1, 0, 0],  # CD3D, CD4, CD8A, CD19
    "CD8 T细胞": [1, 0, 1, 0],
    "B细胞": [0, 0, 0, 1]
}, index=["CD3D", "CD4", "CD8A", "CD19"])

scvi.model.CELLASSIGN.setup_anndata(adata, layer="counts")
model = scvi.model.CELLASSIGN(adata, marker_gene_mat)
model.train()

predictions = model.predict()
```

## Solo（双联体检测）

**用途**：识别scRNA-seq数据中的双联体（包含两个及以上细胞的液滴）。

**核心特性**：
- 基于scVI嵌入的半监督双联体检测
- 生成人工双联体进行训练
- 提供双联体概率评分
- 可应用于任意scVI模型

**适用场景**：
- scRNA-seq数据集质量控制
- 下游分析前移除双联体
- 评估数据中的双联体比例

**基础用法**：
```python
# 先训练scVI模型
scvi.model.SCVI.setup_anndata(adata, layer="counts")
scvi_model = scvi.model.SCVI(adata)
scvi_model.train()

# 训练Solo双联体检测
solo_model = scvi.external.SOLO.from_scvi_model(scvi_model)
solo_model.train()

# 预测双联体
predictions = solo_model.predict()
doublet_scores = predictions["doublet"]
adata.obs["doublet_score"] = doublet_scores
```

## Amortized LDA（主题建模）

**用途**：基于隐狄利克雷分配（LDA）的基因表达主题建模。

**核心特性**：
- 发现基因表达程序（主题）
- 摊销变分推断实现可扩展性
- 每个细胞是主题的混合
- 每个主题是基因的分布

**适用场景**：
- 发现基因程序或表达模块
- 理解表达的组成结构
- 替代性降维方法
- 可解释的表达模式分解

**基础用法**：
```python
scvi.model.AMORTIZEDLDA.setup_anndata(adata, layer="counts")
model = scvi.model.AMORTIZEDLDA(adata, n_topics=10)
model.train()

# 获取细胞主题组成
topic_proportions = model.get_latent_representation()

# 获取主题基因载荷
topic_gene_loadings = model.get_topic_distribution()
```

## 模型选择指南

**选择scVI当**：
- 需进行无监督分析
- 需要批次校正与整合
- 需要标准化表达与差异表达分析

**选择scANVI当**：
- 拥有部分标记细胞用于训练
- 需进行细胞类型注释
- 需将标签从参考集迁移到查询集

**选择AUTOZI当**：
- 关注技术性丢失问题
- 需识别零膨胀基因
- 处理极稀疏数据集

**选择VeloVI当**：
- 拥有剪接/未剪接计数数据
- 关注细胞动态过程
- 需带批次校正的RNA速率分析

**选择contrastiveVI当**：
- 分析扰动实验
- 需分离处理效应
- 需识别条件特异性程序

**选择CellAssign当**：
- 可获得标记基因列表
- 需基于标记的概率化注释
- 无可用参考数据集

**选择Solo当**：
- 需双联体检测
- 已使用scVI进行分析
- 需概率化双联体评分
