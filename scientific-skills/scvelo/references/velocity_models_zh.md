# scVelo 速度模型参考

## 数学框架

RNA速度基于转录动力学模型：

```
dx_s/dt = β·x_u - γ·x_s   (剪接动态)
dx_u/dt = α(t) - β·x_u    (未剪接动态)
```

其中：
- `x_s`：剪接mRNA丰度
- `x_u`：未剪接（前体mRNA）丰度
- `α(t)`：转录速率（随时间变化）
- `β`：剪接速率
- `γ`：降解速率

**速度**定义为：`v = dx_s/dt = β·x_u - γ·x_s`

- **v > 0**：基因表达上调（未剪接mRNA高于稳态预期）
- **v < 0**：基因表达下调（未剪接mRNA低于稳态预期）

## 模型比较

### 稳态模型（Velocyto，原始版）

- 假设恒定α（转录速率）
- 通过线性回归拟合稳态细胞的γ值
- **局限**：需可识别稳态细胞；假设转录恒定

```python
# 用于scVelo向后兼容
scv.tl.velocity(adata, mode='steady_state')
```

### 随机模型（scVelo v1）

- 扩展稳态模型，引入方差/协方差项
- 建模细胞间mRNA计数的变异性
- 比稳态模型更抗噪

```python
scv.tl.velocity(adata, mode='stochastic')
```

### 动力学模型（scVelo v2，推荐）

- 联合估计所有动力学参数（α, β, γ）和细胞特异性潜时间
- 不假设稳态
- 识别诱导期与抑制期
- 计算基因级fit_likelihood（质量指标）

```python
scv.tl.recover_dynamics(adata, n_jobs=4)
scv.tl.velocity(adata, mode='dynamical')
```

**动力学模型识别的状态：**

| 状态 | 描述 |
|-------|-------------|
| 诱导期 | α > 0，x_u上升 |
| 开启稳态 | α > 0，持续高表达 |
| 抑制期 | α = 0，x_u下降 |
| 关闭稳态 | α = 0，持续低表达 |

## 速度图

速度图基于细胞速度与邻近细胞状态的相似性建立连接：

```python
scv.tl.velocity_graph(adata)
# 存储于adata.uns['velocity_graph']
# 元素[i,j] = 细胞i向细胞j迁移的概率
```

**参数说明：**
- `n_neighbors`：考虑的邻近细胞数
- `sqrt_transform`：应用平方根变换（剪接数据默认False）
- `approx`：使用近似最近邻搜索（加速大规模数据集）

## 潜时间解读

基因级潜时间τ ∈ [0, 1] 表示：
- τ = 0：基因处于诱导起始点
- τ = 0.5：基因处于诱导峰值（完整周期）
- τ = 1：基因回归关闭稳态

**共享潜时间**通过所有速度基因的fit_likelihood加权平均计算。

## 质量指标

### 基因级
- `fit_likelihood`：动力学模型拟合优度（0-1；值越高越好）
  - 筛选驱动基因：`adata.var[adata.var['fit_likelihood'] > 0.1]`
- `fit_alpha`：诱导期转录速率
- `fit_gamma`：mRNA降解速率
- `fit_r2`：动力学拟合R²值

### 细胞级
- `velocity_length`：速度向量模长（细胞迁移速度）
- `velocity_confidence`：与邻近细胞的速度一致性（0-1）

### 数据集级
```python
# 评估整体速度质量
scv.pl.proportions(adata)  # 细胞剪接/未剪接比例
scv.pl.velocity_confidence(adata, groupby='leiden')
```

## 参数调优指南

| 参数 | 功能 | 默认值 | 调整场景 |
|-----------|----------|---------|----------------|
| `min_shared_counts` | 基因过滤 | 20 | 深度测序时增大；浅层测序时减小 |
| `n_top_genes` | 高变基因选择 | 2000 | 复杂数据集时增大 |
| `n_neighbors` | kNN图构建 | 30 | 小数据集时减小；高噪声时增大 |
| `n_pcs` | PCA维度 | 30 | 匹配碎石图拐点 |
| `t_max_rank` | 潜时间约束 | None | 已知发育方向时设置 |

## 工具集成

### CellRank（命运预测）

```python
import cellrank as cr
from cellrank.kernels import VelocityKernel, ConnectivityKernel

# 结合速度和连接性核
vk = VelocityKernel(adata).compute_transition_matrix()
ck = ConnectivityKernel(adata).compute_transition_matrix()
combined = 0.8 * vk + 0.2 * ck

# 计算宏状态（终末与初始状态）
g = cr.estimators.GPCCA(combined)
g.compute_macrostates(n_states=4, cluster_key='leiden')
g.plot_macrostates(which="all")

# 计算命运概率
g.compute_fate_probabilities()
g.plot_fate_probabilities()
```

### Scanpy集成

scVelo原生支持Scanpy的AnnData：

```python
import scanpy as sc
import scvelo as scv

# 先运行标准Scanpy流程
sc.pp.normalize_total(adata)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata)
sc.pp.pca(adata)
sc.pp.neighbors(adata)
sc.tl.umap(adata)
sc.tl.leiden(adata)

# 叠加速度分析
scv.pp.moments(adata)
scv.tl.recover_dynamics(adata)
scv.tl.velocity(adata, mode='dynamical')
scv.tl.velocity_graph(adata)
scv.tl.latent_time(adata)
```
