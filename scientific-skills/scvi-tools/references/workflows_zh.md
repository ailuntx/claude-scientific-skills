# 常用工作流程与最佳实践

本文档涵盖 scvi-tools 的常用工作流程、最佳实践和高级使用模式。

## 标准分析流程

### 1. 数据加载与准备

```python
import scvi
import scanpy as sc
import numpy as np

# 加载数据（需AnnData格式）
adata = sc.read_h5ad("data.h5ad")
# 或从其他格式加载
# adata = sc.read_10x_mtx("filtered_feature_bc_matrix/")
# adata = sc.read_csv("counts.csv")

# 基础QC指标
sc.pp.calculate_qc_metrics(adata, inplace=True)
adata.var['mt'] = adata.var_names.str.startswith('MT-')
sc.pp.calculate_qc_metrics(adata, qc_vars=['mt'], inplace=True)
```

### 2. 质量控制

```python
# 过滤细胞
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_cells(adata, max_genes=5000)

# 过滤基因
sc.pp.filter_genes(adata, min_cells=3)

# 按线粒体含量过滤
adata = adata[adata.obs['pct_counts_mt'] < 20, :]

# 去除双细胞（可选，训练前执行）
sc.external.pp.scrublet(adata)
adata = adata[~adata.obs['predicted_doublet'], :]
```

### 3. scvi-tools预处理

```python
# 重要：scvi-tools需要原始计数
# 若已标准化，请使用原始层或重新加载数据

# 保存原始计数（如尚未保存）
if 'counts' not in adata.layers:
    adata.layers['counts'] = adata.X.copy()

# 特征选择（可选但推荐）
sc.pp.highly_variable_genes(
    adata,
    n_top_genes=4000,
    subset=False,  # 保留所有基因，仅标记HVG
    batch_key="batch"  # 多批次时使用
)

# 筛选HVG（可选）
# adata = adata[:, adata.var['highly_variable']]
```

### 4. 向scvi-tools注册数据

```python
# 为scvi-tools设置AnnData
scvi.model.SCVI.setup_anndata(
    adata,
    layer="counts",  # 使用原始计数
    batch_key="batch",  # 技术批次
    categorical_covariate_keys=["donor", "condition"],
    continuous_covariate_keys=["percent_mito", "n_counts"]
)

# 检查注册状态
adata.uns['_scvi']['summary_stats']
```

### 5. 模型训练

```python
# 创建模型
model = scvi.model.SCVI(
    adata,
    n_latent=30,  # 潜在维度
    n_layers=2,   # 网络深度
    n_hidden=128, # 隐藏层大小
    dropout_rate=0.1,
    gene_likelihood="zinb"  # 零膨胀负二项分布
)

# 训练模型
model.train(
    max_epochs=400,
    batch_size=128,
    train_size=0.9,
    early_stopping=True,
    check_val_every_n_epoch=10
)

# 查看训练历史
train_history = model.history["elbo_train"]
val_history = model.history["elbo_validation"]
```

### 6. 提取结果

```python
# 获取潜在表示
latent = model.get_latent_representation()
adata.obsm["X_scVI"] = latent

# 获取标准化表达
normalized = model.get_normalized_expression(
    adata,
    library_size=1e4,
    n_samples=25  # 蒙特卡洛采样
)
adata.layers["scvi_normalized"] = normalized
```

### 7. 下游分析

```python
# 在scVI潜在空间聚类
sc.pp.neighbors(adata, use_rep="X_scVI", n_neighbors=15)
sc.tl.umap(adata, min_dist=0.3)
sc.tl.leiden(adata, resolution=0.8, key_added="leiden")

# 可视化
sc.pl.umap(adata, color=["leiden", "batch", "cell_type"])

# 差异表达分析
de_results = model.differential_expression(
    groupby="leiden",
    group1="0",
    group2="1",
    mode="change",
    delta=0.25
)
```

### 8. 模型持久化

```python
# 保存模型
model_dir = "./scvi_model/"
model.save(model_dir, overwrite=True)

# 保存含结果的AnnData
adata.write("analyzed_data.h5ad")

# 后续加载模型
model = scvi.model.SCVI.load(model_dir, adata=adata)
```

## 超参数调优

### 关键超参数

**架构**:
- `n_latent`: 潜在空间维度 (10-50)
  - 复杂异构数据集用较大值
  - 简单数据集或防过拟合用较小值
- `n_layers`: 隐藏层数量 (1-3)
  - 复杂数据用更多层，但收益递减
- `n_hidden`: 隐藏层节点数 (64-256)
  - 随数据集规模和复杂度调整

**训练**:
- `max_epochs`: 训练迭代次数 (200-500)
  - 使用早停防止过拟合
- `batch_size`: 每批样本数 (64-256)
  - 大数据集用较大值，内存有限用较小值
- `lr`: 学习率 (默认0.001，通常适用)

**模型特定**:
- `gene_likelihood`: 分布类型 ("zinb", "nb", "poisson")
  - "zinb"适用于零膨胀稀疏数据
  - "nb"适用于稀疏度较低数据
- `dispersion`: 基因或基因-批次特异性
  - "gene"用于简单情况，"gene-batch"用于复杂批次效应

### 超参数搜索示例

```python
from scvi.model import SCVI

# 定义搜索空间
latent_dims = [10, 20, 30]
n_layers_options = [1, 2]

best_score = float('-inf')
best_params = None

for n_latent in latent_dims:
    for n_layers in n_layers_options:
        model = SCVI(
            adata,
            n_latent=n_latent,
            n_layers=n_layers
        )
        model.train(max_epochs=200)

        # 在验证集评估
        val_elbo = model.history["elbo_validation"][-1]

        if val_elbo > best_score:
            best_score = val_elbo
            best_params = {"n_latent": n_latent, "n_layers": n_layers}

print(f"最佳参数: {best_params}")
```

### 使用Optuna进行超参数优化

```python
import optuna

def objective(trial):
    n_latent = trial.suggest_int("n_latent", 10, 50)
    n_layers = trial.suggest_int("n_layers", 1, 3)
    n_hidden = trial.suggest_categorical("n_hidden", [64, 128, 256])

    model = scvi.model.SCVI(
        adata,
        n_latent=n_latent,
        n_layers=n_layers,
        n_hidden=n_hidden
    )

    model.train(max_epochs=200, early_stopping=True)
    return model.history["elbo_validation"][-1]

study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=20)

print(f"最佳参数: {study.best_params}")
```

## GPU加速

### 启用GPU训练

```python
# 自动检测GPU
model = scvi.model.SCVI(adata)
model.train(accelerator="auto")  # 可用时自动使用GPU

# 强制使用GPU
model.train(accelerator="gpu")

# 多GPU
model.train(accelerator="gpu", devices=2)

# 检查GPU使用状态
import torch
print(f"CUDA可用: {torch.cuda.is_available()}")
print(f"GPU数量: {torch.cuda.device_count()}")
```

### GPU内存管理

```python
# 内存不足时减小批次大小
model.train(batch_size=64)  # 替代默认值128

# 混合精度训练（节省内存）
model.train(precision=16)

# 运行间清空缓存
import torch
torch.cuda.empty_cache()
```

## 批次整合策略

### 策略1：简单批次键

```python
# 标准批次校正
scvi.model.SCVI.setup_anndata(adata, batch_key="batch")
model = scvi.model.SCVI(adata)
```

### 策略2：多协变量

```python
# 校正多技术因素
scvi.model.SCVI.setup_anndata(
    adata,
    batch_key="sequencing_batch",
    categorical_covariate_keys=["donor", "tissue"],
    continuous_covariate_keys=["percent_mito"]
)
```

### 策略3：层次化批次

```python
# 批次存在层级结构时
# 例如：研究内的样本
adata.obs["batch_hierarchy"] = (
    adata.obs["study"].astype(str) + "_" +
    adata.obs["sample"].astype(str)
)

scvi.model.SCVI.setup_anndata(adata, batch_key="batch_hierarchy")
```

## 参考映射（scArches）

### 训练参考模型

```python
# 在参考数据集训练
scvi.model.SCVI.setup_anndata(ref_adata, batch_key="batch")
ref_model = scvi.model.SCVI(ref_adata)
ref_model.train()

# 保存参考模型
ref_model.save("reference_model")
```

### 将查询数据映射到参考

```python
# 加载参考模型
ref_model = scvi.model.SCVI.load("reference_model", adata=ref_adata)

# 使用相同参数设置查询数据
scvi.model.SCVI.setup_anndata(query_adata, batch_key="batch")

# 迁移学习
query_model = scvi.model.SCVI.load_query_data(
    query_adata,
    "reference_model"
)

# 在查询数据上微调（可选）
query_model.train(max_epochs=200)

# 获取查询嵌入
query_latent = query_model.get_latent_representation()

# 使用KNN迁移标签
from sklearn.neighbors import KNeighborsClassifier

knn = KNeighborsClassifier(n_neighbors=15)
knn.fit(ref_model.get_latent_representation(), ref_adata.obs["cell_type"])
query_adata.obs["predicted_cell_type"] = knn.predict(query_latent)
```

## 模型精简

减小模型尺寸以加速推理：

```python
# 训练完整模型
model = scvi.model.SCVI(adata)
model.train()

# 精简部署版本
minified = model.minify_adata(adata)

# 保存精简版本
minified.write("minified_data.h5ad")
model.save("minified_model")

# 加载使用（速度显著提升）
mini_model = scvi.model.SCVI.load("minified_model", adata=minified)
```

## 内存高效数据加载

### 使用AnnDataLoader

```python
from scvi.data import AnnDataLoader

# 处理超大规模数据集
dataloader = AnnDataLoader(
    adata,
    batch_size=128,
    shuffle=True,
    drop_last=False
)

# 自定义训练循环（高级）
for batch in dataloader:
    # 处理批次数据
    pass
```

### 使用磁盘模式AnnData

```python
# 处理内存无法容纳的数据
adata = sc.read_h5ad("huge_dataset.h5ad", backed='r')

# scvi-tools支持磁盘模式
scvi.model.SCVI.setup_anndata(adata)
model = scvi.model.SCVI(adata)
model.train()
```

## 模型解释

### 使用SHAP的特征重要性

```python
import shap

# 获取可解释性SHAP值
explainer = shap.DeepExplainer(model.module, background_data)
shap_values = explainer.shap_values(test_data)

# 可视化
shap.summary_plot(shap_values, feature_names=adata.var_names)
```

### 基因相关性分析

```python
# 获取基因-基因相关矩阵
correlation = model.get_feature_correlation_matrix(
    adata,
    transform_batch="batch1"
)

# 可视化高相关基因
import seaborn as sns
sns.heatmap(correlation[:50, :50], cmap="coolwarm")
```

## 常见问题排查

### 问题：训练中出现NaN损失

**原因**:
- 学习率过高
- 输入未标准化（必须使用原始计数）
- 数据质量问题

**解决方案**:
```python
# 降低学习率
model.train(lr=0.0001)

# 检查数据
assert adata.X.min() >= 0  # 无负值
assert np.isnan(adata.X).sum() == 0  # 无NaN值

# 使用更稳定的似然函数
model = scvi.model.SCVI(adata, gene_likelihood="nb")
```

### 问题：批次校正效果差

**解决方案**:
```python
# 增强批次效应建模
model = scvi.model.SCVI(
    adata,
    encode_covariates=True,  # 在编码器中注入批次
    deeply_inject_covariates=False
)

# 或尝试相反策略
model = scvi.model.SCVI(adata, deeply_inject_covariates=True)

# 增加潜在维度
model = scvi.model.SCVI(adata, n_latent=50)
```

### 问题：模型未训练（ELBO未下降）

**解决方案**:
```python
# 提高学习率
model.train(lr=0.005)

# 增加网络容量
model = scvi.model.SCVI(adata, n_hidden=256, n_layers=2)

# 延长训练时间
model.train(max_epochs=500)
```

### 问题：内存不足（OOM）

**解决方案**:
```python
# 减小批次大小
model.train(batch_size=64)

# 使用混合精度
model.train(precision=16)

# 减小模型尺寸
model = scvi.model.SCVI(adata, n_latent=10, n_hidden=64)

# 使用磁盘模式AnnData
adata = sc.read_h5ad("data.h5ad", backed='r')
```

## 性能基准测试

```python
import time

# 训练耗时
start = time.time()
model.train(max_epochs=400)
training_time = time.time() - start
print(f"训练耗时: {training_time:.2f}s")

# 推理耗时
start = time.time()
latent = model.get_latent_representation()
inference_time = time.time() - start
print(f"推理耗时: {inference_time:.2f}s")

# 内存使用
import psutil
import os
process = psutil.Process(os.getpid())
memory_gb = process.memory_info().rss / 1024**3
print(f"内存使用: {memory_gb:.2f} GB")
```

## 最佳实践总结

1. **始终使用原始计数**：切勿在scvi-tools前进行log标准化
2. **特征选择**：使用高变基因提升效率
3. **批次校正**：注册所有已知技术协变量
4. **早停机制**：使用验证集防止过拟合
5. **模型保存**：始终保存训练好的模型
6. **GPU使用**：大型数据集(>10k细胞)使用GPU
7. **超参数调优**：从默认值开始，按需调整
8. **验证**：通过可视化检查批次校正效果（UMAP按批次着色）
9. **文档记录**：跟踪预处理步骤
10. **可复现性**：设置随机种子 (`scvi.settings.seed = 0`)
