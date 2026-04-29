# DepMap 依赖性分析指南

## 理解 Chronos 分数

Chronos 是当前（v5+版本）用于从 CRISPR 筛选数据计算基因依赖性分数的算法。它解决了以下系统性偏差：
- 拷贝数效应（高拷贝基因因 DNA 切割而呈现必需性）
- 向导 RNA 效率差异
- 细胞系生长速率

### 分数解读

| 分数范围 | 解释说明 |
|------------|----------------|
| > 0 | 敲除后可能促进生长（存在一定噪音） |
| 0 至 −0.3 | 非必需基因：对适应性影响极小 |
| −0.3 至 −0.5 | 轻度依赖性 |
| −0.5 至 −1.0 | 显著依赖性 |
| < −1.0 | 强依赖性（常见必需基因范围） |
| ≈ −1.0 | 泛必需基因中位值（如蛋白酶体亚基） |

### 常见必需基因（对照）

在几乎所有细胞系中都必需的基因（分数约−1至−2）：
- 核糖体蛋白：RPL...，RPS...
- 蛋白酶体：PSMA...，PSMB...
- 剪接体：SNRPD1，SNRNP70
- DNA 复制：MCM2，PCNA
- 转录：POLR2A，TAF...

这些可作为筛选质量的阳性对照。

### 非必需对照

对适应性影响可忽略的基因（分数约0）：
- 不表达基因（组织特异性）
- 安全港基因座

## 选择性评估

判断依赖性是否具有癌症特异性：

```python
import pandas as pd
import numpy as np

def compute_selectivity(gene_effect_df, target_gene, cancer_lineage):
    """计算特定癌症谱系的选择性分数。"""
    scores = gene_effect_df[target_gene].dropna()

    # 获取细胞系元数据
    from depmap_utils import load_cell_line_info
    cell_info = load_cell_line_info()
    scores_df = scores.reset_index()
    scores_df.columns = ["DepMap_ID", "score"]
    scores_df = scores_df.merge(cell_info[["DepMap_ID", "lineage"]])

    cancer_scores = scores_df[scores_df["lineage"] == cancer_lineage]["score"]
    other_scores = scores_df[scores_df["lineage"] != cancer_lineage]["score"]

    # 选择性：癌症谱系均值低于其他谱系
    selectivity = other_scores.mean() - cancer_scores.mean()
    return {
        "target_gene": target_gene,
        "cancer_lineage": cancer_lineage,
        "cancer_mean": cancer_scores.mean(),
        "other_mean": other_scores.mean(),
        "selectivity_score": selectivity,
        "n_cancer": len(cancer_scores),
        "fraction_dependent": (cancer_scores < -0.5).mean()
    }
```

## CRISPR 数据集版本

| 数据集 | 描述 | 推荐状态 |
|---------|-------------|-------------|
| `CRISPRGeneEffect` | Chronos 校正的基因效应 | 推荐（当前版本） |
| `Achilles_gene_effect` | 旧版 CERES 算法 | 仅历史参考 |
| `RNAi_merged` | DEMETER2 RNAi 数据 | 用于交叉验证 |

## 质量指标

DepMap 提供每项筛选的质量控制指标：
- **偏度**：泛必需基因应呈负偏态分布
- **AUC**：泛必需基因与非必需对照的 ROC 曲线下面积

优质筛选标准：偏度 < −1，AUC > 0.85

## 癌症谱系代码

`sample_info.csv` 中 `lineage` 字段常用值：

| 谱系代码 | 描述 |
|---------|-------------|
| `lung` | 肺癌 |
| `breast` | 乳腺癌 |
| `colorectal` | 结直肠癌 |
| `brain_cancer` | 脑癌（胶质母细胞瘤等） |
| `leukemia` | 白血病 |
| `lymphoma` | 淋巴瘤 |
| `prostate` | 前列腺癌 |
| `ovarian` | 卵巢癌 |
| `pancreatic` | 胰腺癌 |
| `skin` | 黑色素瘤及其他皮肤癌 |
| `liver` | 肝癌 |
| `kidney` | 肾癌 |

## 合成致死性分析

```python
import pandas as pd
import numpy as np
from scipy import stats

def find_synthetic_lethal(gene_effect_df, mutation_df, biomarker_gene,
                           fdr_threshold=0.1):
    """
    寻找功能缺失突变的合成致死配对基因。

    针对每个基因，检测携带 biomarker_gene 突变的细胞系
    是否比野生型细胞系更依赖该基因。
    """
    if biomarker_gene not in mutation_df.columns:
        return pd.DataFrame()

    # 区分突变型与野生型细胞系
    common = gene_effect_df.index.intersection(mutation_df.index)
    is_mutant = mutation_df.loc[common, biomarker_gene] == 1

    mutant_lines = common[is_mutant]
    wt_lines = common[~is_mutant]

    results = []
    for gene in gene_effect_df.columns:
        mut_scores = gene_effect_df.loc[mutant_lines, gene].dropna()
        wt_scores = gene_effect_df.loc[wt_lines, gene].dropna()

        if len(mut_scores) < 5 or len(wt_scores) < 10:
            continue

        stat, pval = stats.mannwhitneyu(mut_scores, wt_scores, alternative='less')
        results.append({
            "gene": gene,
            "mean_mutant": mut_scores.mean(),
            "mean_wt": wt_scores.mean(),
            "effect_size": wt_scores.mean() - mut_scores.mean(),
            "pval": pval,
            "n_mutant": len(mut_scores),
            "n_wt": len(wt_scores)
        })

    df = pd.DataFrame(results)
    # FDR校正
    from scipy.stats import false_discovery_control
    df["qval"] = false_discovery_control(df["pval"], method="bh")
    df = df[df["qval"] < fdr_threshold].sort_values("effect_size", ascending=False)
    return df
```

## 药物敏感性（PRISM）

DepMap 还包含来自 PRISM 实验的化合物敏感性数据：

```python
import pandas as pd

def load_prism_data(filepath="primary-screen-replicate-collapsed-logfold-change.csv"):
    """
    加载 PRISM 药物敏感性数据。
    行 = 细胞系，列 = 化合物（broad_id::名称::剂量）
    数值 = log2 倍数变化（越负表示越敏感）
    """
    return pd.read_csv(filepath, index_col=0)

# 可用数据集：
# primary-screen：单剂量下的 4,518 种化合物
# secondary-screen：约 8,000 种化合物的多剂量数据（提供 AUC）
```
