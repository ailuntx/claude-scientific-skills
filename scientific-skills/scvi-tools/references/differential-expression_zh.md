# scvi-tools中的差异表达分析

本文档详细介绍了使用scvi-tools概率框架进行差异表达（DE）分析的相关信息。

## 概述

scvi-tools实现了贝叶斯差异表达检验方法，利用学习到的生成模型来估计组间表达差异。相比传统方法具有以下优势：

- **批次校正**：在批次校正后的表征上进行DE检验
- **不确定性量化**：效应大小的概率估计
- **零膨胀处理**：正确建模dropout和零值
- **灵活比较**：支持任意组别或细胞类型间比较
- **多模态支持**：适用于RNA（SCVI）、蛋白质（totalVI）和可及性（PeakVI）

## 核心统计框架

### 问题定义

目标是估计两种条件间表达量的对数倍数变化：

```
对数倍数变化 = log(μ_B) - log(μ_A)
```

其中μ_A和μ_B分别表示条件A和B中的平均表达水平。

### 三阶段流程

**阶段1：估计表达水平**
- 从细胞状态后验分布中采样
- 通过学习到的生成模型生成表达值
- 跨细胞聚合获得群体水平估计

**阶段2：检测相关特征（假设检验）**
- 使用贝叶斯框架检验差异表达
- 提供两种检验模式：
  - **"vanilla"模式**：点零假设（β = 0）
  - **"change"模式**：复合假设（|β| ≤ δ）

**阶段3：控制假阳性**
- 后验期望错误发现比例（FDP）控制
- 选择最大发现数量确保E[FDP] ≤ α

## 基础用法

### 简单双组比较

```python
import scvi

# 训练模型后
model = scvi.model.SCVI(adata)
model.train()

# 比较两种细胞类型
de_results = model.differential_expression(
    groupby="cell_type",
    group1="T细胞",
    group2="B细胞"
)

# 查看顶部DE基因
top_genes = de_results.sort_values("lfc_mean", ascending=False).head(20)
print(top_genes[["lfc_mean", "lfc_std", "bayes_factor", "is_de_fdr_0.05"]])
```

### 单组与其余组比较

```python
# 将单组与所有其他组比较
de_results = model.differential_expression(
    groupby="cell_type",
    group1="T细胞"  # 未指定group2即表示与其余组比较
)
```

### 全配对比较

```python
# 所有细胞类型两两比较
all_comparisons = {}

cell_types = adata.obs["cell_type"].unique()

for ct1 in cell_types:
    for ct2 in cell_types:
        if ct1 != ct2:
            key = f"{ct1}_vs_{ct2}"
            all_comparisons[key] = model.differential_expression(
                groupby="cell_type",
                group1=ct1,
                group2=ct2
            )
```

## 关键参数

### `groupby`（必需）
`adata.obs`中定义比较组的列名

```python
# 必须是分类变量
de_results = model.differential_expression(groupby="cell_type")
```

### `group1`和`group2`
待比较的组别。若`group2`为None，则将`group1`与其余组比较

```python
# 特定比较
de = model.differential_expression(groupby="condition", group1="处理组", group2="对照组")

# 单组对全组
de = model.differential_expression(groupby="cell_type", group1="T细胞")
```

### `mode`（假设检验模式）

**"vanilla"模式**（默认）：点零假设
- 检验β是否精确等于0
- 灵敏度高但可能检测到微小效应

**"change"模式**：复合零假设
- 检验|β|是否≤δ
- 要求具有生物学意义的改变
- 减少微小效应的错误发现

```python
# 带最小效应量的change模式
de = model.differential_expression(
    groupby="cell_type",
    group1="T细胞",
    group2="B细胞",
    mode="change",
    delta=0.25  # 最小对数倍数变化
)
```

### `delta`
"change"模式的最小效应量阈值
- 典型值：0.25, 0.5, 0.7（对数尺度）
- log2(1.5) ≈ 0.58（1.5倍变化）
- log2(2) = 1.0（2倍变化）

```python
# 要求至少1.5倍变化
de = model.differential_expression(
    groupby="condition",
    group1="疾病组",
    group2="健康组",
    mode="change",
    delta=0.58  # log2(1.5)
)
```

### `fdr_target`
错误发现率阈值（默认：0.05）

```python
# 更严格的FDR控制
de = model.differential_expression(
    groupby="cell_type",
    group1="T细胞",
    fdr_target=0.01
)
```

### `batch_correction`
是否在DE检验中执行批次校正（默认：True）

```python
# 在特定批次内检验
de = model.differential_expression(
    groupby="cell_type",
    group1="T细胞",
    group2="B细胞",
    batch_correction=False
)
```

### `n_samples`
用于估计的后验采样数（默认：5000）
- 采样数越多越精确但速度越慢
- 减少采样可提速，增加采样可提精度

```python
# 高精度分析
de = model.differential_expression(
    groupby="cell_type",
    group1="T细胞",
    n_samples=10000
)
```

## 结果解读

### 输出列说明

结果DataFrame包含以下重要列：

**效应量估计**：
- `lfc_mean`：对数倍数变化均值
- `lfc_median`：对数倍数变化中位数
- `lfc_std`：对数倍数变化标准差
- `lfc_min`：效应量下限
- `lfc_max`：效应量上限

**统计显著性**：
- `bayes_factor`：差异表达的贝叶斯因子
  - 值越大证据越强
  - >3通常认为有意义
- `is_de_fdr_0.05`：布尔值指示FDR 0.05下是否为DE基因
- `is_de_fdr_0.1`：布尔值指示FDR 0.1下是否为DE基因

**表达水平**：
- `mean1`：组1的平均表达
- `mean2`：组2的平均表达
- `non_zeros_proportion1`：组1中非零细胞比例
- `non_zeros_proportion2`：组2中非零细胞比例

### 示例解读

```python
de_results = model.differential_expression(
    groupby="cell_type",
    group1="T细胞",
    group2="B细胞"
)

# 找出T细胞中显著上调基因
upreg_tcells = de_results[
    (de_results["is_de_fdr_0.05"]) &
    (de_results["lfc_mean"] > 0)
].sort_values("lfc_mean", ascending=False)

print(f"T细胞中上调基因数: {len(upreg_tcells)}")
print(upreg_tcells.head(10))

# 找出大效应量基因
large_effect = de_results[
    (de_results["is_de_fdr_0.05"]) &
    (abs(de_results["lfc_mean"]) > 1)  # 2倍变化
]
```

## 高级用法

### 特定细胞子集内DE分析

```python
# 仅在部分细胞子集内检验DE
subset_indices = adata.obs["tissue"] == "肺组织"

de = model.differential_expression(
    idx1=adata.obs["cell_type"] == "T细胞" & subset_indices,
    idx2=adata.obs["cell_type"] == "B细胞" & subset_indices
)
```

### 批次特异性DE分析

```python
# 在每个批次内分别检验DE
batches = adata.obs["batch"].unique()

batch_de_results = {}
for batch in batches:
    batch_idx = adata.obs["batch"] == batch
    batch_de_results[batch] = model.differential_expression(
        idx1=(adata.obs["condition"] == "处理组") & batch_idx,
        idx2=(adata.obs["condition"] == "对照组") & batch_idx
    )
```

### 伪批量DE分析

```python
# DE检验前聚合细胞
# 适用于组内细胞数较少的情况

de = model.differential_expression(
    groupby="cell_type",
    group1="稀有细胞类型",
    group2="常见细胞类型",
    n_samples=10000,  # 增加采样保证稳定性
    batch_correction=True
)
```

## 可视化

### 火山图

```python
import matplotlib.pyplot as plt
import numpy as np

de = model.differential_expression(
    groupby="condition",
    group1="处理组",
    group2="对照组"
)

# 火山图
plt.figure(figsize=(10, 6))
plt.scatter(
    de["lfc_mean"],
    -np.log10(1 / (de["bayes_factor"] + 1)),
    c=de["is_de_fdr_0.05"],
    cmap="coolwarm",
    alpha=0.5
)
plt.xlabel("对数倍数变化")
plt.ylabel("-log10(1/贝叶斯因子)")
plt.title("火山图：处理组 vs 对照组")
plt.axvline(x=0, color='k', linestyle='--', linewidth=0.5)
plt.show()
```

### 顶部DE基因热图

```python
import seaborn as sns

# 获取顶部DE基因
top_genes = de.sort_values("lfc_mean", ascending=False).head(50).index

# 获取标准化表达
norm_expr = model.get_normalized_expression(
    adata,
    indices=adata.obs["condition"].isin(["处理组", "对照组"]),
    gene_list=top_genes
)

# 绘制热图
plt.figure(figsize=(12, 10))
sns.heatmap(
    norm_expr.T,
    cmap="viridis",
    xticklabels=False,
    yticklabels=top_genes
)
plt.title("前50个DE基因")
plt.show()
```

### 基因排序图

```python
# 按效应量排序绘制基因
de_sorted = de.sort_values("lfc_mean", ascending=False)

plt.figure(figsize=(12, 6))
plt.plot(range(len(de_sorted)), de_sorted["lfc_mean"].values)
plt.axhline(y=0, color='r', linestyle='--')
plt.xlabel("基因排序")
plt.ylabel("对数倍数变化")
plt.title("按效应量排序的基因")
plt.show()
```

## 与传统方法比较

### scvi-tools vs. Wilcoxon检验

```python
import scanpy as sc

# 传统Wilcoxon检验
sc.tl.rank_genes_groups(
    adata,
    groupby="cell_type",
    method="wilcoxon",
    key_added="wilcoxon"
)

# scvi-tools DE分析
de_scvi = model.differential_expression(
    groupby="cell_type",
    group1="T细胞"
)

# 结果比较
wilcox_results = sc.get.rank_genes_groups_df(adata, group="T细胞", key="wilcoxon")
```

**scvi-tools优势**：
- 自动校正批次效应
- 正确处理零膨胀
- 提供不确定性量化
- 无需人工设定伪计数
- 更优的统计特性

**Wilcoxon适用场景**：
- 快速探索性分析
- 与使用Wilcoxon的已发表结果比较

## 多模态DE分析

### 蛋白质DE（totalVI）

```python
# 在CITE-seq数据上训练totalVI
totalvi_model = scvi.model.TOTALVI(adata)
totalvi_model.train()

# RNA差异表达
rna_de = totalvi_model.differential_expression(
    groupby="cell_type",
    group1="T细胞",
    group2="B细胞",
    protein_expression=False  # 默认
)

# 蛋白质差异表达
protein_de = totalvi_model.differential_expression(
    groupby="cell_type",
    group1="T细胞",
    group2="B细胞",
    protein_expression=True
)

print(f"DE基因数: {rna_de['is_de_fdr_0.05'].sum()}")
print(f"DE蛋白数: {protein_de['is_de_fdr_0.05'].sum()}")
```

### 差异可及性（PeakVI）

```python
# 在ATAC-seq数据上训练PeakVI
peakvi_model = scvi.model.PEAKVI(atac_adata)
peakvi_model.train()

# 差异可及性分析
da = peakvi_model.differential_accessibility(
    groupby="cell_type",
    group1="T细胞",
    group2="B细胞"
)

# 解读方式与DE相同
```

## 特殊案例处理

### 低细胞数组

```python
# 增加后验采样保证稳定性
de = model.differential_expression(
    groupby="cell_type",
    group1="稀有类型",  # 例如50个细胞
    group2="常见类型",  # 例如5000个细胞
    n_samples=10000
)
```

### 不平衡比较

```python
# 当组间规模差异极大时
# 使用change模式避免微小效应

de = model.differential_expression(
    groupby="condition",
    group1="稀有条件",
    group2="常见条件",
    mode="change",
    delta=0.5
)
```

### 多重检验校正

```python
# 已通过FDP控制实现
# 但可应用额外校正

from statsmodels.stats.multitest import multipletests

# Bonferroni校正（非常保守）
_, pvals_corrected, _, _ = multipletests(
    1 / (de["bayes_factor"] + 1),
    method="bonferroni"
)
```

## 性能考量

### 速度优化

```python
# 大型数据集加速DE检验
de = model.differential_expression(
    groupby="cell_type",
    group1="T细胞",
    n_samples=1000,  # 减少采样
    batch_size=512    # 增大批次大小
)
```

### 内存管理

```python
# 超大型数据集处理
# 逐次比较而非全配对

cell_types = adata.obs["cell_type"].unique()
for ct in cell_types:
    de = model.differential_expression(
        groupby="cell_type",
        group1=ct
    )
    # 保存结果
    de.to_csv(f"de_results_{ct}.csv")
```

## 最佳实践

1. **使用"change"模式**：获得生物学可解释结果
2. **设置合适delta**：基于生物学意义
3. **检查表达水平**：过滤低表达基因
4. **验证发现**：检查标记基因进行合理性确认
5. **可视化结果**：始终绘制顶部DE基因
6. **记录参数**：注明使用的模式、delta和FDR
7. **考虑批次效应**：使用batch_correction=True
8. **多重比较**：注意多组检验问题
9. **样本量**：确保每组足够细胞（建议>50）
10. **生物学验证**：通过功能实验跟进

## 示例：完整DE分析流程

```python
import scvi
import scanpy as sc
import matplotlib.pyplot as plt

# 1. 训练模型
scvi.model.SCVI.setup_anndata(adata, layer="counts", batch_key="batch")
model = scvi.model.SCVI(adata)
model.train()

# 2. 执行DE分析
de_results = model.differential_expression(
    groupby="cell_type",
    group1="疾病_T细胞",
    group2="健康_T细胞",
    mode="change",
    delta=0.5,
    fdr_target=0.05
)

# 3. 过滤与分析
sig_genes = de_results[de_results["is_de_fdr_0.05"]]
upreg = sig_genes[sig_genes["lfc_mean"] > 0].sort_values("lfc_mean", ascending=False)
downreg = sig_genes[sig_genes["lfc_mean"] < 0].sort_values("lfc_mean")

print(f"显著基因数: {len(sig_genes)}")
print(f"上调基因: {len(upreg)}")
print(f"下调基因: {len(downreg)}")

# 4. 可视化顶部基因
```

<think>
We are translating a markdown content that includes a mix of code and comments.
 The instruction: 
   - Preserve the original markdown structure exactly.
   - Do not add explanations, notes, or markdown fences.
   - Do not translate URLs, file paths, or raw code.
   - Translate comments inside code blocks when they are natural language.
   - Keep technical terms or proper nouns in English when translation would be harmful.

 Steps:
 1. The code lines should remain as they are (including variable names, function names, etc.)
 2. The comments (lines starting with #) should be translated to Chinese, but note that some comments might be code (like #5) which is actually a markdown header? Let's see:

 Observation:
   The content has:
     # 5. Functional enrichment (using external tools)
     # E.g., g:Profiler, DAVID, or gprofiler-official Python package
     # Perform pathway analysis...

   And then:
     # 6. Save results

   These are actually markdown headers and comments? But note they are inside a code block? Actually, the entire content is in a code block? 

   Wait, the input markdown content is:

   top_genes = ... 
   sc.pl.violin(...)
   # 5. Functional enrichment ...
   upreg_genes = ...
   # Perform pathway analysis...
   # 6. Save results
   de_results.to_csv(...)

   And ends with a triple backtick.

   However, the instruction says: "Return only the translated markdown", and we are to preserve the structure.

   Important: The entire content is inside a code block? Actually, the input markdown content is provided without any fences? 

   But note: the input says "Markdown content", and it includes code and comments. And the output should be the same structure.

   Actually, the input does not have triple backticks? It starts with:

   top_genes = ... 

   and ends with:

   downreg.to_csv("downregulated_genes.csv")

   and then a triple backtick? But wait, the triple backtick is at the very end? 

   The input:

   ```
   top_genes = upreg.head(10).index.tolist() + downreg.head(10).index.tolist()

   sc.pl.violin(
       adata[adata.obs["cell_type"].isin(["Disease_T_cells", "Healthy_T_cells"])],
       keys=top_genes,
       groupby="cell_type",
       rotation=90
   )

   # 5. Functional enrichment (using external tools)
   # E.g., g:Profiler, DAVID, or gprofiler-official Python package
   upreg_genes = upreg.head(100).index.tolist()
   # Perform pathway analysis...

   # 6. Save results
   de_results.to_csv("de_results_disease_vs_healthy.csv")
   upreg.to_csv("upregulated_genes.csv")
   downreg.to_csv("downregulated_genes.csv")
   ```

   But note: the triple backtick at the end of the input is actually the end of the markdown code block? Or is it part of the content?

   The instruction says: the markdown content is provided, and we are to translate and return the translated markdown.

   However, the input we are given ends with a triple backtick? That might be the end of the code block in the original markdown? But our task is to translate the content inside.

   Actually, the user's input is:

   "Markdown content:

   top_genes = ... 

   ... 

   downreg.to_csv("downregulated_genes.csv")
   ```"

   So the triple backtick is the end of the code block? But note that the content we are to translate is the code block? 

   However, the instruction says: preserve the original markdown structure. So we should output the same code block? But without the triple backticks? 

   But wait, the example output format: we are to return only the translated markdown content? And the input markdown content was the code block without the fences? 

   Actually, the user says: "Return only the translated markdown." and the markdown content is the code and comments? 

   How to handle:

   We are to translate the natural language comments inside the code block? And leave the code as is.

   So:

   - The lines that are code: do not change.
   - The lines that are comments (starting with #) and are in natural language: translate to Chinese.

   But note: the first two lines of the content are code? Then a blank line? Then a function call? Then comments?

   Specifically:

   Line: "# 5. Functional enrichment (using external tools)" -> This is a comment and should be translated.
   Line: "# E.g., g:Profiler, DAVID, or gprofiler-official Python package" -> translate.
   Line: "# Perform pathway analysis..." -> translate.
   Line: "# 6. Save results" -> translate.

   However, note that the numbers 5 and 6 are part of the comment? They are section headers? We should translate the entire comment including the number? 

   But the number is part of the text? So we translate the text after the number? And keep the number? 

   Example: 
      # 5. Functional enrichment (using external tools) 
      becomes: 
      # 5. 功能富集（使用外部工具）

   Similarly, # 6. Save results -> # 6. 保存结果

   Also, note that the example includes a code line: `upreg_genes = upreg.head(100).index.tolist()` - this is code and remains.

   And the save commands: they are code and remain.

   Important: the triple backtick at the end of the input? It is not part of the content? Because the user wrote:

   "Markdown content:

   [the code block without fences?]"

   But then they included a triple backtick? Actually, the triple backtick is the end of the markdown code block that they are showing? 

   Since we are only to return the translated markdown content (which is the inside of the code block), we do not include the triple backtick? 

   However, the user's input ends with:

   downreg.to_csv("downregulated_genes.csv")
   ```

   So the triple backtick is the last line? But note: the triple backtick is not part of the code block content? It is the markdown fence? 

   Therefore, we should not include it in the translation? 

   But the instruction says: preserve the original markdown structure exactly. And the original content we are translating is the code block without the fences? 

   Actually, the user says: "Translate the following markdown content", and then they provided:

   top_genes = ... 
   ... 
   downreg.to_csv("downregulated_genes.csv")
   ```

   Wait, the triple backtick is at the end? So the markdown content they are providing is a code block? But we are to translate the content of the code block? 

   However, the instruction says: "Return only the translated markdown", meaning we are to output the same structure? 

   But note: the markdown content we are translating is the code block? So we should output the same code block? But with the comments translated? 

   However, the user's example input does not have the starting triple backtick? It starts with the code? 

   Actually, the user input:

   "Markdown content:

   top_genes = ..."

   So we are to translate the lines from "top_genes = ..." to the last line of code? And the triple backtick is the end of the markdown code block? But we are not to include the triple backtick? 

   Since the instruction says: "Return only the translated markdown", and the markdown content we are given is the code block without the fences? 

   How the user provided:

   They wrote:

   ```
   Markdown content:

   top_genes = ... 
   ... 
   downreg.to_csv("downregulated_genes.csv")
   ```

   So the triple backtick at the end is actually the end of the entire markdown block? But we are only concerned with the content between the two triple backticks? 

   Actually, the user's input is:

   "Translate the following markdown content from English to Chinese.

   Return only the translated markdown.

   Markdown content:

   top_genes = ... 
   ... 
   downreg.to_csv("downregulated_genes.csv")
   ```"

   So the triple backtick is the last line? And it is part of the markdown content? 

   But note: in markdown, a code block is started and ended with triple backticks. So if the content we are translating is a code block, then the triple backtick is the end of the code block? 

   However, the instruction says: preserve the original markdown structure. So we must include the triple backtick? 

   But wait, the triple backtick is not inside the code block? It is the fence? 

   Actually, the content we are translating is:

   top_genes = ... 
   ... 
   downreg.to_csv("downregulated_genes.csv")

   and then the triple backtick is the end of the code block? But we are not translating the fence? 

   The instruction: "Translate the following markdown content" and then they provided the code block? 

   How to proceed:

   We are to translate the content that is inside the code block? And then output the same code block? But without the fences? 

   But the user says: "Return only the translated markdown", meaning we are to output the same as the input? But translated? 

   However, the input markdown content is the code block? So we must output the code block? 

   But note: the user's input does not have the starting triple backtick? It starts with the code? 

   Actually, the user's input for the markdown content is:

   top_genes = ... 
   ... 
   downreg.to_csv("downregulated_genes.csv")
   ```

   So the last line is a triple backtick? 

   Therefore, we must translate the lines that are not the triple backtick? And then include the triple backtick? 

   But the triple backtick is not a comment? It is markdown syntax? 

   However, the instruction says: preserve the original markdown structure exactly. So we must include the triple backtick? 

   But note: the triple backtick is not part of the code? It is the fence? 

   Alternatively, the user might have intended the content to be:

   The code block without the fences? 

   Since the instruction is ambiguous, and the example input ends with triple backtick, we will assume that the triple backtick is part of the markdown content? 

   However, looking at the input, the triple backtick is on a separate line? 

   So we break down:

   - The first line: "top_genes = ..."
   - ... 
   - The last two lines: "downreg.to_csv("downregulated_genes.csv")" and then "```"

   But note: the triple backtick is not inside the code? It is the end of the code block? 

   Therefore, we should not translate the triple backtick? And it is not part of the code block content? 

   How the user wrote:

   They wrote:

   Markdown content:

   [code lines]
   ```

   So the triple backtick is the end of the markdown code block? But we are only to translate the content of the code block? 

   Therefore, we will translate the code block content (leaving code as is, translating comments) and then we do not include the triple backtick? 

   But the instruction says: preserve the original markdown structure. The original structure is a code block? 

   However, the user's request: "Return only the translated markdown" and the markdown content they provided is the code block without the starting fence? But with the ending fence? 

   Actually, the entire markdown content they provided is:

   top_genes = ... 
   ... 
   downreg.to_csv("downregulated_genes.csv")
   ```

   So we must translate the entire thing? But the triple backtick is not translatable? 

   Alternatively, the triple backtick might be a mistake? 

   Given the confusion, I will assume that the triple backtick is not part of the content? Because the user said "Markdown content" and then listed the lines? And the triple backtick is the end of the code block in the message? 

   Therefore, we will translate the content without the triple backtick? 

   But note: the input ends with:

   downreg.to_csv("downregulated_genes.csv")
   ```

   So the last line is the triple backtick? 

   We are to translate the markdown content? And the markdown content includes the triple backtick? 

   However, the triple backtick is not English? So we leave it? 

   But the instruction: "Do not translate URLs, file paths, or raw code." and the triple backtick is raw markdown? 

   So we leave it? 

   However, the triple backtick is not inside the code block? It is the fence? 

   Actually, in the context of the entire markdown document, the code block is:

   \```python
   ... code ...
   \```

   But the user only provided the inside? 

   Since the instruction is to preserve the structure, and the input we are given ends with triple backtick, we will output:

   [translated code lines]
   ```

   But wait, the triple backtick is not translated? 

   However, the user's input for the markdown content ends with triple backtick? So we must include it? 

   But note: the triple backtick is not a natural language? So we leave it as is.

   Therefore, we will:

   - Translate the comments (lines starting with #) that are in natural language.
   - Leave the code lines as they are.
   - Leave the triple backtick as it is.

   However, the triple backtick is on a separate line? 

   Steps:

   We split the input by lines:

   Line 1: top_genes = upreg.head(10).index.tolist() + downreg.head(10).index.tolist()
   Line 2: (empty)
   Line 3: sc.pl.violin(
   Line 4: adata[adata.obs["cell_type"].isin(["Disease_T_cells", "Healthy_T_cells"])],
   Line 5: keys=top_genes,
   Line 6: groupby="cell_type",
   Line 7: rotation=90
   Line 8: )
   Line 9: (empty)
   Line 10: # 5. Functional enrichment (using external tools)
   Line 11: # E.g., g:Profiler, DAVID, or gprofiler-official Python package
   Line 12: upreg_genes = upreg.head(100).index.tolist()
   Line 13: # Perform pathway analysis...
   Line 14: (empty)
   Line 15: # 6. Save results
   Line 16: de_results.to_csv("de_results_disease_vs_healthy.csv")
   Line 17: upreg.to_csv("upregulated_genes.csv")
   Line 18: downreg.to_csv("downregulated_genes.csv")
   Line 19: ```

   But wait, the input ends with triple backtick? So we have 19 lines? 

   However, the user input does not have a starting triple backtick? So the code block is not started? 

   Actually, the user input for the markdown content starts at "top_genes" and ends with triple backtick? 

   Therefore, we will translate the comments in lines 10, 11, 13, 15? 

   Translation:

   Line 10: # 5. Functional enrichment (using external tools) 
        -> # 5. 功能富集（使用外部工具）

   Line 11: # E.g., g:Profiler, DAVID, or gprofiler-official Python package
        -> # 例如：g:Profiler、DAVID 或 gprofiler-official Python 包

   Line 13: # Perform pathway analysis...
        -> # 执行通路分析...

   Line 15: # 6. Save results
        -> # 6. 保存结果

   The rest of the lines (code and the triple backtick) remain unchanged.

   But note: the triple backtick is not a comment? So we leave it? 

   However, the triple backtick is on a line by itself? And it is not a comment? So we don't touch it.

   Therefore, the translated markdown content is:

   top_genes = upreg.head(10).index.tolist() + downreg.head(10).index.tolist()

   sc.pl.violin(
       adata[adata.obs["cell_type"].isin(["Disease_T_cells", "Healthy_T_cells"])],
       keys=top_genes,
       groupby="cell_type",
       rotation=90
   )

   # 5. 功能富集（使用外部工具）
   # 例如：g:Profiler、DAVID 或 gprofiler-official Python 包
   upreg_genes = upreg.head(100).index.tolist()
   # 执行通路分析...

   # 6. 保存结果
   de_results.to_csv("de_results_disease_vs_healthy.csv")
   upreg.to_csv("upregulated_genes.csv")
   downreg.to_csv("downregulated_genes.csv")
   ```

   But note: the triple backtick is included? 

   However, the instruction says: preserve the original structure. So we output exactly the same lines, with the same line breaks, but with the comments translated.

   But wait, the original input had two empty lines? We must preserve them? 

   We are preserving every line? 

   Therefore, we output:

   [line1] top_genes = ... 
   [line2] (empty)
   [line3] sc.pl.violin( ...
