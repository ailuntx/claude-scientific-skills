---
name: pydeseq2
description: 差异基因表达分析（Python DESeq2）。从bulk RNA-seq计数数据中识别差异表达基因，支持Wald检验、FDR校正、火山图/MA图绘制，用于RNA-seq分析。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# PyDESeq2

## 概述

PyDESeq2是DESeq2的Python实现，用于bulk RNA-seq数据的差异表达分析。支持从数据加载到结果解读的完整工作流，包括单因素和多因素实验设计、带多重检验校正的Wald检验、可选的apeGLM收缩估计，并与pandas和AnnData无缝集成。

## 适用场景

本技能适用于：
- 分析bulk RNA-seq计数数据的差异表达
- 比较实验条件间的基因表达差异（如处理组 vs 对照组）
- 校正批次效应或协变量的多因素实验设计
- 将基于R的DESeq2工作流迁移至Python
- 在Python流程中集成差异表达分析
- 当用户提及"DESeq2"、"差异表达"、"RNA-seq分析"或"PyDESeq2"时

## 快速入门工作流

执行标准差异表达分析：

```python
import pandas as pd
from pydeseq2.dds import DeseqDataSet
from pydeseq2.ds import DeseqStats

# 1. 加载数据
counts_df = pd.read_csv("counts.csv", index_col=0).T  # 转置为样本×基因
metadata = pd.read_csv("metadata.csv", index_col=0)

# 2. 过滤低表达基因
genes_to_keep = counts_df.columns[counts_df.sum(axis=0) >= 10]
counts_df = counts_df[genes_to_keep]

# 3. 初始化并拟合DESeq2
dds = DeseqDataSet(
    counts=counts_df,
    metadata=metadata,
    design="~condition",
    refit_cooks=True
)
dds.deseq2()

# 4. 执行统计检验
ds = DeseqStats(dds, contrast=["condition", "treated", "control"])
ds.summary()

# 5. 获取结果
results = ds.results_df
significant = results[results.padj < 0.05]
print(f"发现 {len(significant)} 个显著差异基因")
```

## 核心工作流步骤

### 步骤1：数据准备

**输入要求：**
- **计数矩阵：** 样本×基因的DataFrame，包含非负整数读段计数
- **元数据：** 样本×变量的DataFrame，包含实验因素

**常用数据加载模式：**

```python
# 从CSV加载（典型格式：基因×样本，需转置）
counts_df = pd.read_csv("counts.csv", index_col=0).T
metadata = pd.read_csv("metadata.csv", index_col=0)

# 从TSV加载
counts_df = pd.read_csv("counts.tsv", sep="\t", index_col=0).T

# 从AnnData加载
import anndata as ad
adata = ad.read_h5ad("data.h5ad")
counts_df = pd.DataFrame(adata.X, index=adata.obs_names, columns=adata.var_names)
metadata = adata.obs
```

**数据过滤：**

```python
# 移除低表达基因
genes_to_keep = counts_df.columns[counts_df.sum(axis=0) >= 10]
counts_df = counts_df[genes_to_keep]

# 移除元数据缺失的样本
samples_to_keep = ~metadata.condition.isna()
counts_df = counts_df.loc[samples_to_keep]
metadata = metadata.loc[samples_to_keep]
```

### 步骤2：实验设计

设计公式指定基因表达模型：

**单因素设计：**
```python
design = "~condition"  # 简单两组比较
```

**多因素设计：**
```python
design = "~batch + condition"  # 校正批次效应
design = "~age + condition"     # 包含连续协变量
design = "~group + condition + group:condition"  # 交互效应
```

**设计公式指南：**
- 使用Wilkinson公式符号（R风格）
- 将校正变量（如批次）置于主要变量前
- 确保变量存在于元数据DataFrame
- 使用合适数据类型（分类变量用category）

### 步骤3：DESeq2拟合

初始化DeseqDataSet并运行完整流程：

```python
from pydeseq2.dds import DeseqDataSet

dds = DeseqDataSet(
    counts=counts_df,
    metadata=metadata,
    design="~condition",
    refit_cooks=True,  # 剔除离群值后重新拟合
    n_cpus=1           # 并行处理（按需调整）
)

# 执行完整DESeq2流程
dds.deseq2()
```

**`deseq2()`执行流程：**
1. 计算尺寸因子（标准化）
2. 拟合基因特异性离散度
3. 拟合离散度趋势曲线
4. 计算离散度先验
5. 拟合MAP离散度（收缩）
6. 拟合对数倍数变化
7. 计算Cook距离（离群值检测）
8. 检测到离群值时重新拟合（可选）

### 步骤4：统计检验

执行Wald检验识别差异表达基因：

```python
from pydeseq2.ds import DeseqStats

ds = DeseqStats(
    dds,
    contrast=["condition", "treated", "control"],  # 检验处理组 vs 对照组
    alpha=0.05,                # 显著性阈值
    cooks_filter=True,         # 过滤离群值
    independent_filter=True    # 过滤低统计功效检验
)

ds.summary()
```

**对比组设定：**
- 格式：`[变量, 检验组, 参照组]`
- 示例：`["condition", "treated", "control"]`检验处理组 vs 对照组
- 若为`None`，则使用设计矩阵最后一个系数

**结果DataFrame列：**
- `baseMean`：样本间平均标准化计数
- `log2FoldChange`：条件间log2倍数变化
- `lfcSE`：LFC标准误
- `stat`：Wald检验统计量
- `pvalue`：原始p值
- `padj`：校正p值（经Benjamini-Hochberg法FDR校正）

### 步骤5：可选LFC收缩

应用收缩减少倍数变化估计噪声：

```python
ds.lfc_shrink()  # 应用apeGLM收缩
```

**适用场景：**
- 可视化（火山图、热图）
- 按效应量排序基因
- 优先选择后续实验基因

**注意：** 收缩仅影响log2FoldChange值，不改变统计检验结果（p值不变）。可视化使用收缩值，但报告显著性时使用未收缩p值。

### 步骤6：结果导出

保存结果及中间对象：

```python
import pickle

# 导出CSV结果
ds.results_df.to_csv("deseq2_results.csv")

# 仅保存显著基因
significant = ds.results_df[ds.results_df.padj < 0.05]
significant.to_csv("significant_genes.csv")

# 保存DeseqDataSet供后续使用
with open("dds_result.pkl", "wb") as f:
    pickle.dump(dds.to_picklable_anndata(), f)
```

## 常用分析模式

### 两组比较

标准病例-对照比较：

```python
dds = DeseqDataSet(counts=counts_df, metadata=metadata, design="~condition")
dds.deseq2()

ds = DeseqStats(dds, contrast=["condition", "treated", "control"])
ds.summary()

results = ds.results_df
significant = results[results.padj < 0.05]
```

### 多重比较

多处理组与对照组比较：

```python
dds = DeseqDataSet(counts=counts_df, metadata=metadata, design="~condition")
dds.deseq2()

treatments = ["treatment_A", "treatment_B", "treatment_C"]
all_results = {}

for treatment in treatments:
    ds = DeseqStats(dds, contrast=["condition", treatment, "control"])
    ds.summary()
    all_results[treatment] = ds.results_df

    sig_count = len(ds.results_df[ds.results_df.padj < 0.05])
    print(f"{treatment}: {sig_count} 个显著基因")
```

### 校正批次效应

控制技术变异：

```python
# 设计中包含批次
dds = DeseqDataSet(counts=counts_df, metadata=metadata, design="~batch + condition")
dds.deseq2()

# 校正批次后检验条件效应
ds = DeseqStats(dds, contrast=["condition", "treated", "control"])
ds.summary()
```

### 连续协变量

纳入年龄或剂量等连续变量：

```python
# 确保连续变量为数值型
metadata["age"] = pd.to_numeric(metadata["age"])

dds = DeseqDataSet(counts=counts_df, metadata=metadata, design="~age + condition")
dds.deseq2()

ds = DeseqStats(dds, contrast=["condition", "treated", "control"])
ds.summary()
```

## 使用分析脚本

本技能包含标准分析的命令行脚本：

```bash
# 基础用法
python scripts/run_deseq2_analysis.py \
  --counts counts.csv \
  --metadata metadata.csv \
  --design "~condition" \
  --contrast condition treated control \
  --output results/

# 带附加选项
python scripts/run_deseq2_analysis.py \
  --counts counts.csv \
  --metadata metadata.csv \
  --design "~batch + condition" \
  --contrast condition treated control \
  --output results/ \
  --min-counts 10 \
  --alpha 0.05 \
  --n-cpus 4 \
  --plots
```

**脚本功能：**
- 自动数据加载与验证
- 基因和样本过滤
- 完整DESeq2流程执行
- 可定制参数的统计检验
- 结果导出（CSV, pickle）
- 可选可视化（火山图和MA图）

当用户需要独立分析工具或批量处理多数据集时，推荐使用`scripts/run_deseq2_analysis.py`。

## 结果解读

### 识别显著基因

```python
# 按校正p值过滤
significant = ds.results_df[ds.results_df.padj < 0.05]

# 同时过滤显著性和效应量
sig_and_large = ds.results_df[
    (ds.results_df.padj < 0.05) &
    (abs(ds.results_df.log2FoldChange) > 1)
]

# 区分上/下调基因
upregulated = significant[significant.log2FoldChange > 0]
downregulated = significant[significant.log2FoldChange < 0]

print(f"上调基因: {len(upregulated)}")
print(f"下调基因: {len(downregulated)}")
```

### 排序与筛选

```python
# 按校正p值排序
top_by_padj = ds.results_df.sort_values("padj").head(20)

# 按绝对倍数变化排序（使用收缩值）
ds.lfc_shrink()
ds.results_df["abs_lfc"] = abs(ds.results_df.log2FoldChange)
top_by_lfc = ds.results_df.sort_values("abs_lfc", ascending=False).head(20)

# 按综合指标排序
ds.results_df["score"] = -np.log10(ds.results_df.padj) * abs(ds.results_df.log2FoldChange)
top_combined = ds.results_df.sort_values("score", ascending=False).head(20)
```

### 质量评估

```python
# 检查标准化（尺寸因子应接近1）
print("尺寸因子:", dds.obsm["size_factors"])

# 检查离散度估计
import matplotlib.pyplot as plt
plt.hist(dds.varm["dispersions"], bins=50)
plt.xlabel("离散度")
plt.ylabel("频数")
plt.title("离散度分布")
plt.show()

# 检查p值分布（应基本均匀且在0处有峰值）
plt.hist(ds.results_df.pvalue.dropna(), bins=50)
plt.xlabel("P值")
plt.ylabel("频数")
plt.title("P值分布")
plt.show()
```

## 可视化指南

### 火山图

展示显著性 vs 效应量：

```python
import matplotlib.pyplot as plt
import numpy as np

results = ds.results_df.copy()
results["-log10(padj)"] = -np.log10(results.padj)

plt.figure(figsize=(10, 6))
significant = results.padj < 0.05

plt.scatter(
    results.loc[~significant, "log2FoldChange"],
    results.loc[~significant, "-log10(padj)"],
    alpha=0.3, s=10, c='gray', label='不显著'
)
plt.scatter(
    results.loc[significant, "log2FoldChange"],
    results.loc[significant, "-log10(padj)"],
    alpha=0.6, s=10, c='red', label='padj < 0.05'
)

plt.axhline(-np.log10(0.05), color='blue', linestyle='--', alpha=0.5)
plt.xlabel("Log2倍数变化")
plt.ylabel("-Log10(校正P值)")
plt.title("火山图")
plt.legend()
plt.savefig("volcano_plot.png", dpi=300)
```

### MA图

展示倍数变化 vs 平均表达量：

```python
plt.figure(figsize=(10, 6))

plt.scatter(
    np.log10(results.loc[~significant, "baseMean"] + 1),
    results.loc[~significant, "log2FoldChange"],
    alpha=0.3, s=10, c='gray'
)
plt.scatter(
    np.log10(results.loc[significant, "baseMean"] + 1),
    results.loc[significant, "log2FoldChange"],
    alpha=0.6, s=10, c='red'
)

plt.axhline(0, color='blue', linestyle='--', alpha=0.5)
plt.xlabel("Log10(基础均值 + 1)")
plt.ylabel("Log2倍数变化")
plt.title("MA图")
plt.savefig("ma_plot.png", dpi=300)
```

## 常见问题排查

### 数据格式问题

**问题：** "计数矩阵与元数据的索引不匹配"

**解决方案：** 确保样本名称完全一致
```python
print("计数样本:", counts_df.index.tolist())
print("元数据样本:", metadata.index.tolist())

# 必要时取交集
common = counts_df.index.intersection(metadata.index)
counts_df = counts_df.loc[common]
metadata = metadata.loc[common]
```

**问题：** "所有基因计数为零"

**解决方案：** 检查是否需要转置
```python
print(f"计数矩阵维度: {counts_df.shape}")
# 若基因数 > 样本数，需转置
if counts_df.shape[1] < counts_df.shape[0]:
    counts_df = counts_df.T
```

### 设计矩阵问题

**问题：** "设计矩阵非满秩"

**原因：** 变量混淆（如所有处理样本在同一批次）

**解决方案：** 移除混淆变量或添加交互项
```python
# 检查混淆
print(pd.crosstab(metadata.condition, metadata.batch))

# 简化设计或添加交互项
design = "~condition"  # 移除批次
# 或
design = "~condition + batch + condition:batch"  # 建模交互效应
```

### 无显著基因

**诊断方法：**
```python
# 检查离散度分布
plt.hist(dds.varm["dispersions"], bins=50)
plt.show()

# 检查尺寸因子
print(dds.obsm["size_factors"])

# 查看原始p值最小的基因
print(ds.results_df.nsmallest(20, "pvalue"))
```

**可能原因：**
- 效应量过小
- 生物变异性高
- 样本量不足
- 技术问题（批次效应、离群值）

3. **基因过滤：** 过滤低计数基因（例如总读数<10），以提高统计功效并减少计算时间。

4. **设计公式顺序：** 将调整变量置于目标变量之前（例如 `"~batch + condition"`，而非 `"~condition + batch"`）。

5. **LFC收缩时机：** 在统计检验后应用收缩，且仅用于可视化/排序目的。P值仍基于未收缩估计值。

6. **结果解读：** 使用 `padj < 0.05` 作为显著性标准（非原始p值）。Benjamini-Hochberg 程序控制错误发现率。

7. **对比设定：** 格式为 `[变量, 测试水平, 参考水平]`，其中测试水平与参考水平进行比较。

8. **保存中间对象：** 使用 pickle 保存 DeseqDataSet 对象，便于后续使用或附加分析，避免重新运行耗时的拟合步骤。

## 安装与要求

```bash
uv pip install pydeseq2
```

**系统要求：**
- Python 3.10-3.11
- pandas 1.4.3+
- numpy 1.23.0+
- scipy 1.11.0+
- scikit-learn 1.1.1+
- anndata 0.8.0+

**可视化可选依赖：**
- matplotlib
- seaborn

## 附加资源

- **官方文档：** https://pydeseq2.readthedocs.io
- **GitHub 仓库：** https://github.com/owkin/PyDESeq2
- **文献：** Muzellec et al. (2023) Bioinformatics, DOI: 10.1093/bioinformatics/btad547
- **原始 DESeq2 (R)：** Love et al. (2014) Genome Biology, DOI: 10.1186/s13059-014-0550-8
