# PyDESeq2 工作流程指南

本文档提供了常见 PyDESeq2 分析模式的详细分步工作流程。

## 目录
1. [完整差异表达分析](#complete-differential-expression-analysis)
2. [数据加载与准备](#data-loading-and-preparation)
3. [单因素分析](#single-factor-analysis)
4. [多因素分析](#multi-factor-analysis)
5. [结果导出与可视化](#result-export-and-visualization)
6. [常见模式与最佳实践](#common-patterns-and-best-practices)
7. [故障排除](#troubleshooting)

---

## 完整差异表达分析

### 概述
标准 PyDESeq2 分析包含两个阶段的 12 个主要步骤：

**阶段 1：读计数建模（步骤 1-7）**
- 标准化与离散度估计
- 对数倍变化拟合
- 离群值检测

**阶段 2：统计分析（步骤 8-12）**
- Wald 检验
- 多重检验校正
- 可选 LFC 收缩

### 完整工作流程代码

```python
import pandas as pd
from pydeseq2.dds import DeseqDataSet
from pydeseq2.ds import DeseqStats

# 加载数据
counts_df = pd.read_csv("counts.csv", index_col=0).T  # 必要时转置
metadata = pd.read_csv("metadata.csv", index_col=0)

# 过滤低表达基因
genes_to_keep = counts_df.columns[counts_df.sum(axis=0) >= 10]
counts_df = counts_df[genes_to_keep]

# 移除元数据缺失的样本
samples_to_keep = ~metadata.condition.isna()
counts_df = counts_df.loc[samples_to_keep]
metadata = metadata.loc[samples_to_keep]

# 初始化 DeseqDataSet
dds = DeseqDataSet(
    counts=counts_df,
    metadata=metadata,
    design="~condition",
    refit_cooks=True
)

# 运行标准化与拟合
dds.deseq2()

# 执行统计检验
ds = DeseqStats(
    dds,
    contrast=["condition", "treated", "control"],
    alpha=0.05,
    cooks_filter=True,
    independent_filter=True
)
ds.summary()

# 可选：应用 LFC 收缩用于可视化
ds.lfc_shrink()

# 访问结果
results = ds.results_df
print(results.head())
```

---

## 数据加载与准备

### 加载 CSV 文件

计数数据通常为基因×样本格式，但需要转置：

```python
import pandas as pd

# 加载计数矩阵（基因×样本）
counts_df = pd.read_csv("counts.csv", index_col=0)

# 转置为样本×基因格式
counts_df = counts_df.T

# 加载元数据（已是样本×变量格式）
metadata = pd.read_csv("metadata.csv", index_col=0)
```

### 从其他格式加载

**从 TSV 加载：**
```python
counts_df = pd.read_csv("counts.tsv", sep="\t", index_col=0).T
metadata = pd.read_csv("metadata.tsv", sep="\t", index_col=0)
```

**从 pickle 加载：**
```python
import pickle

with open("counts.pkl", "rb") as f:
    counts_df = pickle.load(f)

with open("metadata.pkl", "rb") as f:
    metadata = pickle.load(f)
```

**从 AnnData 加载：**
```python
import anndata as ad

adata = ad.read_h5ad("data.h5ad")
counts_df = pd.DataFrame(
    adata.X,
    index=adata.obs_names,
    columns=adata.var_names
)
metadata = adata.obs
```

### 数据过滤

**过滤低表达基因：**
```python
# 移除总读数少于 10 的基因
genes_to_keep = counts_df.columns[counts_df.sum(axis=0) >= 10]
counts_df = counts_df[genes_to_keep]
```

**过滤元数据缺失的样本：**
```python
# 移除'condition'列为 NA 的样本
samples_to_keep = ~metadata.condition.isna()
counts_df = counts_df.loc[samples_to_keep]
metadata = metadata.loc[samples_to_keep]
```

**多条件过滤：**
```python
# 仅保留满足所有条件的样本
mask = (
    ~metadata.condition.isna() &
    (metadata.batch.isin(["batch1", "batch2"])) &
    (metadata.age >= 18)
)
counts_df = counts_df.loc[mask]
metadata = metadata.loc[mask]
```

### 数据验证

**检查数据结构：**
```python
print(f"计数矩阵维度: {counts_df.shape}")  # 应为（样本数, 基因数）
print(f"元数据维度: {metadata.shape}")  # 应为（样本数, 变量数）
print(f"索引匹配: {all(counts_df.index == metadata.index)}")

# 检查负值
assert (counts_df >= 0).all().all(), "计数值必须为非负数"

# 检查非整数值
assert counts_df.applymap(lambda x: x == int(x)).all().all(), "计数值必须为整数"
```

---

## 单因素分析

### 简单双组比较

比较处理组与对照组：

```python
from pydeseq2.dds import DeseqDataSet
from pydeseq2.ds import DeseqStats

# 设计：将表达建模为条件的函数
dds = DeseqDataSet(
    counts=counts_df,
    metadata=metadata,
    design="~condition"
)

dds.deseq2()

# 测试处理组 vs 对照组
ds = DeseqStats(
    dds,
    contrast=["condition", "treated", "control"]
)
ds.summary()

# 结果
results = ds.results_df
significant = results[results.padj < 0.05]
print(f"发现 {len(significant)} 个显著基因")
```

### 多重成对比较

比较多组时：

```python
# 测试每个处理组 vs 对照组
treatments = ["treated_A", "treated_B", "treated_C"]
all_results = {}

for treatment in treatments:
    ds = DeseqStats(
        dds,
        contrast=["condition", treatment, "control"]
    )
    ds.summary()
    all_results[treatment] = ds.results_df

# 跨处理组比较结果
for name, results in all_results.items():
    sig = results[results.padj < 0.05]
    print(f"{name}: {len(sig)} 个显著基因")
```

---

## 多因素分析

### 双因素设计

在测试条件时考虑批次效应：

```python
# 设计包含批次和条件
dds = DeseqDataSet(
    counts=counts_df,
    metadata=metadata,
    design="~batch + condition"
)

dds.deseq2()

# 在控制批次后测试条件效应
ds = DeseqStats(
    dds,
    contrast=["condition", "treated", "control"]
)
ds.summary()
```

### 交互效应

测试处理效应在组间是否不同：

```python
# 设计包含交互项
dds = DeseqDataSet(
    counts=counts_df,
    metadata=metadata,
    design="~group + condition + group:condition"
)

dds.deseq2()

# 测试交互项
ds = DeseqStats(dds, contrast=["group:condition", ...])
ds.summary()
```

### 连续协变量

包含年龄等连续变量：

```python
# 确保元数据中年龄为数值型
metadata["age"] = pd.to_numeric(metadata["age"])

dds = DeseqDataSet(
    counts=counts_df,
    metadata=metadata,
    design="~age + condition"
)

dds.deseq2()
```

---

## 结果导出与可视化

### 保存结果

**导出为 CSV：**
```python
# 保存统计结果
ds.results_df.to_csv("deseq2_results.csv")

# 仅保存显著基因
significant = ds.results_df[ds.results_df.padj < 0.05]
significant.to_csv("significant_genes.csv")

# 保存排序结果
sorted_results = ds.results_df.sort_values("padj")
sorted_results.to_csv("sorted_results.csv")
```

**保存 DeseqDataSet：**
```python
import pickle

# 保存为 AnnData 供后续使用
with open("dds_result.pkl", "wb") as f:
    pickle.dump(dds.to_picklable_anndata(), f)
```

**加载保存结果：**
```python
# 加载结果
results = pd.read_csv("deseq2_results.csv", index_col=0)

# 加载 AnnData
with open("dds_result.pkl", "rb") as f:
    adata = pickle.load(f)
```

### 基础可视化

**火山图：**
```python
import matplotlib.pyplot as plt
import numpy as np

results = ds.results_df.copy()
results["-log10(padj)"] = -np.log10(results.padj)

# 绘图
plt.figure(figsize=(10, 6))
plt.scatter(
    results.log2FoldChange,
    results["-log10(padj)"],
    alpha=0.5,
    s=10
)
plt.axhline(-np.log10(0.05), color='red', linestyle='--', label='padj=0.05')
plt.axvline(1, color='gray', linestyle='--')
plt.axvline(-1, color='gray', linestyle='--')
plt.xlabel("Log2 倍变化值")
plt.ylabel("-Log10(校正P值)")
plt.title("火山图")
plt.legend()
plt.savefig("volcano_plot.png", dpi=300)
```

**MA 图：**
```python
plt.figure(figsize=(10, 6))
plt.scatter(
    np.log10(results.baseMean + 1),
    results.log2FoldChange,
    alpha=0.5,
    s=10,
    c=(results.padj < 0.05),
    cmap='bwr'
)
plt.xlabel("Log10(基础均值 + 1)")
plt.ylabel("Log2 倍变化值")
plt.title("MA 图")
plt.savefig("ma_plot.png", dpi=300)
```

---

## 常见模式与最佳实践

### 1. 数据预处理清单

运行 PyDESeq2 前：
- ✓ 确保计数值为非负整数
- ✓ 验证样本×基因方向
- ✓ 检查计数和元数据的样本名称匹配
- ✓ 移除或处理缺失元数据值
- ✓ 过滤低表达基因（通常总读数 < 10）
- ✓ 验证实验因子编码正确

### 2. 设计公式最佳实践

**顺序重要：** 将调整变量放在目标变量前
```python
# 正确：控制批次后测试条件
design = "~batch + condition"

# 不理想：条件放在首位
design = "~condition + batch"
```

**离散变量使用分类类型：**
```python
# 确保正确数据类型
metadata["condition"] = metadata["condition"].astype("category")
metadata["batch"] = metadata["batch"].astype("category")
```

### 3. 统计检验指南

**设置合适 alpha 值：**
```python
# 标准显著性阈值
ds = DeseqStats(dds, alpha=0.05)

# 探索性分析使用更严格阈值
ds = DeseqStats(dds, alpha=0.01)
```

**使用独立过滤：**
```python
# 推荐：过滤低统计功效检验
ds = DeseqStats(dds, independent_filter=True)

# 仅在特定原因下禁用
ds = DeseqStats(dds, independent_filter=False)
```

### 4. LFC 收缩

**使用场景：**
- 可视化（火山图、热图）
- 按效应大小排序基因
- 优先选择后续验证基因

**避免场景：**
- 报告统计显著性（使用未收缩 p 值）
- 基因集富集分析（通常使用未收缩值）

```python
# 保存两个版本
ds.results_df.to_csv("results_unshrunken.csv")
ds.lfc_shrink()
ds.results_df.to_csv("results_shrunken.csv")
```

### 5. 内存管理

大型数据集处理：
```python
# 使用并行处理
dds = DeseqDataSet(
    counts=counts_df,
    metadata=metadata,
    design="~condition",
    n_cpus=4  # 根据可用核心调整
)

# 需要时分批处理
# （将基因分块，分别分析，合并结果）
```

---

## 故障排除

### 错误：计数与元数据索引不匹配

**问题：** 样本名称不匹配
```
KeyError: 计数和元数据中的样本名称不匹配
```

**解决方案：**
```python
# 检查索引
print("计数样本:", counts_df.index.tolist())
print("元数据样本:", metadata.index.tolist())

# 必要时对齐
common_samples = counts_df.index.intersection(metadata.index)
counts_df = counts_df.loc[common_samples]
metadata = metadata.loc[common_samples]
```

### 错误：所有基因计数为零

**问题：** 数据可能需要转置
```
ValueError: 所有基因总计数为零
```

**解决方案：**
```python
# 检查数据方向
print(f"计数维度: {counts_df.shape}")

# 若基因数 > 样本数，可能需要转置
if counts_df.shape[1] < counts_df.shape[0]:
    counts_df = counts_df.T
```

### 警告：大量基因被过滤

**问题：** 过多低表达基因被移除

**检查：**
```python
# 查看基因计数分布
print(counts_df.sum(axis=0).describe())

# 可视化
import matplotlib.pyplot as plt
plt.hist(counts_df.sum(axis=0), bins=50, log=True)
plt.xlabel("每个基因的总计数")
plt.ylabel("频率")
plt.show()
```

**必要时调整过滤：**
```python
# 尝试更低阈值
genes_to_keep = counts_df.columns[counts_df.sum(axis=0) >= 5]
```

### 错误：设计矩阵非满秩

**问题：** 混杂设计（例如所有处理样本在同一批次）

**解决方案：**
```python
# 检查设计混杂
print(pd.crosstab(metadata.condition, metadata.batch))

# 移除混杂变量或添加交互项
design = "~condition"  # 删除批次
# 或
design = "~condition + batch + condition:batch"  # 添加交互项
```

### 问题：未发现显著基因

**可能原因：**
1. 效应量小
2. 高生物学变异性
3. 样本量不足
4. 技术问题（批次效应、离群值）

**诊断：**
```python
# 检查离散度估计
import matplotlib.pyplot as plt
dispersions = dds.varm["dispersions"]
plt.hist(dispersions, bins=50)
plt.xlabel("离散度")
plt.ylabel("频率")
plt.show()

# 检查尺寸因子（应接近 1）
print("尺寸因子:", dds.obsm["size_factors"])

# 查看最显著基因（即使未达阈值）
top_genes = ds.results_df.nsmallest(20, "pvalue")
print(top_genes)
```

### 大型数据集内存错误

**解决方案：**
```python
# 1. 减少 CPU 核心数（可能有效）
dds = DeseqDataSet(..., n_cpus=1)

# 2. 更严格过滤
genes_to_keep = counts_df.columns[counts_df.sum(axis=0) >= 20]

# 3. 分批处理
# 按基因子集拆分分析并合并结果
```
