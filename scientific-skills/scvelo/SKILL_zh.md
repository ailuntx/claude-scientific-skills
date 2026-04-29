---
name: scvelo
description: 使用 scVelo 进行 RNA 速率分析。通过未剪接/已剪接 mRNA 动态估计细胞状态转换，推断轨迹方向，计算潜在时间，并在单细胞 RNA 测序数据中识别驱动基因。作为 Scanpy/scVI-tools 轨迹推断的补充工具。
license: BSD-3-Clause
metadata:
    skill-author: Kuan-lin Huang
---

# scVelo — RNA 速率分析

## 概述

scVelo 是单细胞 RNA 测序数据中 RNA 速率分析的领先 Python 工具包。它通过模拟 mRNA 剪接动力学推断细胞状态转换——利用未剪接（前体 mRNA）与已剪接（成熟 mRNA）丰度的比例，确定每个细胞中基因的上调或下调状态。这可在无需时间序列数据的情况下重建发育轨迹并识别细胞命运决定。

**安装命令：** `pip install scvelo`

**核心资源：**
- 文档：https://scvelo.readthedocs.io/
- GitHub：https://github.com/theislab/scvelo
- 论文：Bergen 等人 (2020) Nature Biotechnology. PMID: 32747759

## 适用场景

在以下情况使用 scVelo：

- **基于静态数据的轨迹推断**：确定细胞分化方向
- **细胞命运预测**：识别祖细胞及其下游命运
- **驱动基因识别**：发现最能解释观测轨迹的动态基因
- **发育生物学研究**：模拟造血、神经发生、上皮-间质转化
- **潜在时间估计**：根据剪接动态将细胞沿伪时间排序
- **Scanpy 补充**：为 UMAP 嵌入添加方向信息

## 先决条件

scVelo 需要 **未剪接** 和 **已剪接** RNA 的计数矩阵。生成方式包括：
1. **STARsolo** 或 **kallisto|bustools** 的 `lamanno` 模式
2. **velocyto** 命令行：`velocyto run10x` / `velocyto run`
3. **alevin-fry** / **simpleaf** 支持未剪接/已剪接输出

数据存储在包含 `layers["spliced"]` 和 `layers["unspliced"]` 的 `AnnData` 对象中。

## 标准 RNA 速率工作流

### 1. 环境配置与数据加载

```python
import scvelo as scv
import scanpy as sc
import numpy as np
import matplotlib.pyplot as plt

# 配置参数
scv.settings.verbosity = 3       # 显示计算步骤
scv.settings.presenter_view = True
scv.settings.set_figure_params('scvelo')

# 加载数据（需含未剪接/已剪接层的 AnnData）
# 方案 A：从 loom 文件加载（velocyto 输出）
adata = scv.read("cellranger_output.loom", cache=True)

# 方案 B：合并 velocyto loom 与 Scanpy 处理的 AnnData
adata_processed = sc.read_h5ad("processed.h5ad")  # 含 UMAP、聚类信息
adata_velocity = scv.read("velocyto.loom")
adata = scv.utils.merge(adata_processed, adata_velocity)

# 验证数据层
print(adata)
# obs × var: N × G
# layers: 'spliced', 'unspliced' (必需)
# obsm['X_umap'] (可视化必需)
```

### 2. 预处理

```python
# 过滤与标准化（遵循 Scanpy 规范）
scv.pp.filter_and_normalize(
    adata,
    min_shared_counts=20,   # 未剪接+已剪接最小计数
    n_top_genes=2000        # 高变基因数量
)

# 计算一阶和二阶矩（均值与方差）
# 需先计算 knn_connectivities
sc.pp.neighbors(adata, n_neighbors=30, n_pcs=30)
scv.pp.moments(
    adata,
    n_pcs=30,
    n_neighbors=30
)
```

### 3. 速率估计 — 随机模型

随机模型速度快，适合探索性分析：

```python
# 随机速率模型（快速但精度较低）
scv.tl.velocity(adata, mode='stochastic')
scv.tl.velocity_graph(adata)

# 可视化
scv.pl.velocity_embedding_stream(
    adata,
    basis='umap',
    color='leiden',
    title="RNA 速率（随机模型）"
)
```

### 4. 速率估计 — 动力学模型（推荐）

动力学模型拟合完整剪接过程，精度更高：

```python
# 恢复动力学（计算密集；10K细胞约需10-30分钟）
scv.tl.recover_dynamics(adata, n_jobs=4)

# 基于动力学模型计算速率
scv.tl.velocity(adata, mode='dynamical')
scv.tl.velocity_graph(adata)
```

### 5. 潜在时间

动力学模型支持计算共享潜在时间（伪时间）：

```python
# 计算潜在时间
scv.tl.latent_time(adata)

# 在UMAP上可视化潜在时间
scv.pl.scatter(
    adata,
    color='latent_time',
    color_map='gnuplot',
    size=80,
    title='潜在时间'
)

# 按潜在时间排序识别关键基因
top_genes = adata.var['fit_likelihood'].sort_values(ascending=False).index[:300]
scv.pl.heatmap(
    adata,
    var_names=top_genes,
    sortby='latent_time',
    col_color='leiden',
    n_convolve=100
)
```

### 6. 驱动基因分析

```python
# 识别速率拟合度最高的基因
scv.tl.rank_velocity_genes(adata, groupby='leiden', min_corr=0.3)
df = scv.DataFrame(adata.uns['rank_velocity_genes']['names'])
print(df.head(10))

# 速率长度与置信度
scv.tl.velocity_confidence(adata)
scv.pl.scatter(
    adata,
    c=['velocity_length', 'velocity_confidence'],
    cmap='coolwarm',
    perc=[5, 95]
)

# 特定基因的相位图
scv.pl.velocity(adata, ['Cpe', 'Gnao1', 'Ins2'],
               ncols=3, figsize=(16, 4))
```

### 7. 速率箭头与伪时间

```python
# UMAP上的箭头图
scv.pl.velocity_embedding(
    adata,
    arrow_length=3,
    arrow_size=2,
    color='leiden',
    basis='umap'
)

# 流线图（更清晰的可视化）
scv.pl.velocity_embedding_stream(
    adata,
    basis='umap',
    color='leiden',
    smooth=0.8,
    min_mass=4
)

# 速率伪时间（替代潜在时间）
scv.tl.velocity_pseudotime(adata)
scv.pl.scatter(adata, color='velocity_pseudotime', cmap='gnuplot')
```

### 8. PAGA 轨迹图

```python
# 基于速率信息的PAGA轨迹图
scv.tl.paga(adata, groups='leiden')
df = scv.get_df(adata, 'paga/transitions_confidence', precision=2).T
df.style.background_gradient(cmap='Blues').format('{:.2g}')

# 结合速率的PAGA图
scv.pl.paga(
    adata,
    basis='umap',
    size=50,
    alpha=0.1,
    min_edge_width=2,
    node_size_scale=1.5
)
```

## 完整工作流脚本

```python
import scvelo as scv
import scanpy as sc

def run_rna_velocity(adata, n_top_genes=2000, mode='dynamical', n_jobs=4):
    """
    完整RNA速率工作流

    参数：
        adata: 含未剪接/已剪接层和UMAP的AnnData
        n_top_genes: 用于速率分析的高变基因数量
        mode: 'stochastic'（快速）或'dynamical'（精确）
        n_jobs: 动力学模型的并行任务数

    返回：
        包含速率信息的处理后AnnData
    """
    scv.settings.verbosity = 2

    # 1. 预处理
    scv.pp.filter_and_normalize(adata, min_shared_counts=20, n_top_genes=n_top_genes)

    if 'neighbors' not in adata.uns:
        sc.pp.neighbors(adata, n_neighbors=30)

    scv.pp.moments(adata, n_pcs=30, n_neighbors=30)

    # 2. 速率估计
    if mode == 'dynamical':
        scv.tl.recover_dynamics(adata, n_jobs=n_jobs)

    scv.tl.velocity(adata, mode=mode)
    scv.tl.velocity_graph(adata)

    # 3. 下游分析
    if mode == 'dynamical':
        scv.tl.latent_time(adata)
        scv.tl.rank_velocity_genes(adata, groupby='leiden', min_corr=0.3)

    scv.tl.velocity_confidence(adata)
    scv.tl.velocity_pseudotime(adata)

    return adata
```

## AnnData 关键输出字段

工作流完成后将添加以下字段：

| 位置 | 字段 | 描述 |
|----------|-----|-------------|
| `adata.layers` | `velocity` | 单细胞单基因RNA速率 |
| `adata.layers` | `fit_t` | 单细胞单基因拟合潜在时间 |
| `adata.obsm` | `velocity_umap` | UMAP二维速率向量 |
| `adata.obs` | `velocity_pseudotime` | 速率伪时间 |
| `adata.obs` | `latent_time` | 动力学模型潜在时间 |
| `adata.obs` | `velocity_length` | 细胞速率大小 |
| `adata.obs` | `velocity_confidence` | 细胞置信度评分 |
| `adata.var` | `fit_likelihood` | 基因模型拟合质量 |
| `adata.var` | `fit_alpha` | 转录速率 |
| `adata.var` | `fit_beta` | 剪接速率 |
| `adata.var` | `fit_gamma` | 降解速率 |
| `adata.uns` | `velocity_graph` | 细胞间转移概率矩阵 |

## 速率模型对比

| 模型 | 速度 | 精度 | 适用场景 |
|-------|-------|----------|-------------|
| `stochastic` | 快 | 中等 | 探索性分析；大型数据集 |
| `deterministic` | 中等 | 中等 | 简单线性动力学 |
| `dynamical` | 慢 | 高 | 发表级分析；驱动基因识别 |

## 最佳实践

- **先用随机模型探索**，最终分析切换至动力学模型
- **需保证未剪接读段覆盖度**：短读段（<100 bp）可能丢失内含子覆盖
- **最少 2,000 个细胞**：细胞量过少会导致噪声显著
- **速率应具生物学一致性**：箭头方向需符合已知生物学；随机指向表明存在问题
- **k-NN 带宽影响显著**：邻域过小→速率噪声大；过大→过度平滑
- **完整性检查**：根细胞（祖细胞）的标记基因应具高未剪接/已剪接比例
- **动力学模型需明确动力学状态**：在清晰的分化过程中效果最佳

## 故障排除

| 问题 | 解决方案 |
|---------|---------|
| 缺少未剪接层 | 重运行 velocyto 或使用 `--soloFeatures Gene Velocyto` 的 STARsolo |
| 有效速率基因过少 | 降低 `min_shared_counts`；检查测序深度 |
| 箭头随机分布 | 调整 `n_neighbors` 或更换速率模型 |
| 动力学模型内存错误 | 设置 `n_jobs=1`；减少 `n_top_genes` |
| 全局负速率 | 检查未剪接/已剪接层是否颠倒 |

## 扩展资源

- **scVelo 文档**：https://scvelo.readthedocs.io/
- **教程笔记本**：https://scvelo.readthedocs.io/tutorials/
- **GitHub**：https://github.com/theislab/scvelo
- **论文**：Bergen V 等人 (2020) Nature Biotechnology. PMID: 32747759
- **velocyto**（预处理）：http://velocyto.org/
- **CellRank**（命运预测，scVelo扩展）：https://cellrank.readthedocs.io/
- **dynamo**（代谢标记替代方案）：https://dynamo-release.readthedocs.io/
