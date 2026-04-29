---
name: depmap
description: 查询癌症依赖图谱（DepMap）中的癌细胞系基因依赖性评分（CRISPR Chronos）、药物敏感性数据及基因效应谱。用于识别癌症特异性脆弱点、合成致死相互作用及验证肿瘤药物靶点。
license: CC-BY-4.0
metadata:
    skill-author: Kuan-lin Huang
---

# DepMap — 癌症依赖图谱

## 概述

癌症依赖图谱（DepMap）项目由布罗德研究所运营，通过全基因组CRISPR敲除筛选（DepMap CRISPR）、RNA干扰（RNAi）和化合物敏感性检测（PRISM），系统性地表征数百种癌细胞系的遗传依赖性。DepMap数据对以下方面至关重要：
- 识别特定癌症类型中必需的基因
- 发现癌症选择性依赖（治疗靶点）
- 验证肿瘤药物靶点
- 探索合成致死相互作用

**核心资源：**
- DepMap门户：https://depmap.org/portal/
- DepMap数据下载：https://depmap.org/portal/download/all/
- Python包：`depmap`（或通过API/下载访问）
- API：https://depmap.org/portal/api/

## 使用场景

在以下场景使用DepMap：

- **靶点验证**：某基因在特定突变（如KRAS突变）的癌细胞系中是否对生存至关重要？
- **生物标志物发现**：哪些基因组特征能预测基因敲除的敏感性？
- **合成致死**：发现当另一基因突变/缺失时选择性必需的基因
- **药物敏感性**：哪些细胞系特征能预测化合物响应？
- **泛癌必需性**：某基因是否在所有癌症类型中广泛必需（不良靶点）或选择性必需？
- **相关性分析**：哪些基因对的依赖性谱存在相关性（共必需性）？

## 核心概念

### 依赖性评分

| 评分 | 范围 | 含义 |
|-------|-------|---------|
| **Chronos** (CRISPR) | ~ -3 至 0+ | 数值越负表示越必需。常用必需阈值：−1。泛必需基因范围：~−1 至 −2 |
| **RNAi DEMETER2** | ~ -3 至 0+ | 与Chronos尺度相似 |
| **基因效应** | 标准化 | 归一化的Chronos；−1 = 常见必需基因的中位效应 |

**关键阈值：**
- Chronos ≤ −0.5：可能存在依赖性
- Chronos ≤ −1：强依赖性（常见必需范围）

### 细胞系注释

每个细胞系包含：
- `DepMap_ID`：唯一标识符（如`ACH-000001`）
- `cell_line_name`：可读名称
- `primary_disease`：癌症类型
- `lineage`：组织大类
- `lineage_subtype`：具体亚型

## 核心功能

### 1. DepMap API

```python
import requests
import pandas as pd

BASE_URL = "https://depmap.org/portal/api"

def depmap_get(endpoint, params=None):
    url = f"{BASE_URL}/{endpoint}"
    response = requests.get(url, params=params)
    response.raise_for_status()
    return response.json()
```

### 2. 基因依赖性评分

```python
def get_gene_dependency(gene_symbol, dataset="Chronos_Combined"):
    """获取某基因在所有细胞系中的CRISPR依赖性评分"""
    url = f"{BASE_URL}/gene"
    params = {
        "gene_id": gene_symbol,
        "dataset": dataset
    }
    response = requests.get(url, params=params)
    return response.json()

# 或使用/data端点：
def get_dependencies_slice(gene_symbol, dataset_name="CRISPRGeneEffect"):
    """从数据集中获取某基因的依赖性切片"""
    url = f"{BASE_URL}/data/gene_dependency"
    params = {"gene_name": gene_symbol, "dataset_name": dataset_name}
    response = requests.get(url, params=params)
    data = response.json()
    return data
```

### 3. 基于下载的分析（推荐大规模查询）

大规模分析时，下载DepMap数据文件本地处理：

```python
import pandas as pd
import requests, os

def download_depmap_data(url, output_path):
    """下载DepMap数据文件"""
    response = requests.get(url, stream=True)
    with open(output_path, 'wb') as f:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)

# DepMap 24Q4数据文件（按需更新版本）
FILES = {
    "crispr_gene_effect": "https://figshare.com/ndownloader/files/...",
    # 或从https://depmap.org/portal/download/all/下载
    # 可用文件：
    # CRISPRGeneEffect.csv - Chronos基因效应评分
    # OmicsExpressionProteinCodingGenesTPMLogp1.csv - mRNA表达量
    # OmicsSomaticMutationsMatrixDamaging.csv - 突变二元矩阵
    # OmicsCNGene.csv - 拷贝数
    # sample_info.csv - 细胞系元数据
}

def load_depmap_gene_effect(filepath="CRISPRGeneEffect.csv"):
    """
    加载DepMap CRISPR基因效应矩阵
    行=细胞系（DepMap_ID），列=基因（Symbol (EntrezID)）
    """
    df = pd.read_csv(filepath, index_col=0)
    # 将列名简化为基因符号
    df.columns = [col.split(" ")[0] for col in df.columns]
    return df

def load_cell_line_info(filepath="sample_info.csv"):
    """加载细胞系元数据"""
    return pd.read_csv(filepath)
```

### 4. 识别选择性依赖

```python
import numpy as np
import pandas as pd

def find_selective_dependencies(gene_effect_df, cell_line_info, target_gene,
                                 cancer_type=None, threshold=-0.5):
    """识别对某基因存在选择性依赖的细胞系"""

    # 获取目标基因评分
    if target_gene not in gene_effect_df.columns:
        return None

    scores = gene_effect_df[target_gene].dropna()
    dependent = scores[scores <= threshold]

    # 添加细胞系信息
    result = pd.DataFrame({
        "DepMap_ID": dependent.index,
        "gene_effect": dependent.values
    }).merge(cell_line_info[["DepMap_ID", "cell_line_name", "primary_disease", "lineage"]])

    if cancer_type:
        result = result[result["primary_disease"].str.contains(cancer_type, case=False, na=False)]

    return result.sort_values("gene_effect")

# 使用示例（加载数据后）
# df_effect = load_depmap_gene_effect("CRISPRGeneEffect.csv")
# cell_info = load_cell_line_info("sample_info.csv")
# deps = find_selective_dependencies(df_effect, cell_info, "KRAS", cancer_type="Lung")
```

### 5. 生物标志物分析（基因效应 vs. 突变）

```python
import pandas as pd
from scipy import stats

def biomarker_analysis(gene_effect_df, mutation_df, target_gene, biomarker_gene):
    """
    检验生物标志物基因的突变是否预测目标基因的依赖性

    参数：
        gene_effect_df: CRISPR基因效应DataFrame
        mutation_df: 二元突变DataFrame（1=突变）
        target_gene: 待评估依赖性的基因
        biomarker_gene: 其突变可能预测依赖性的基因
    """
    if target_gene not in gene_effect_df.columns or biomarker_gene not in mutation_df.columns:
        return None

    # 对齐细胞系
    common_lines = gene_effect_df.index.intersection(mutation_df.index)
    scores = gene_effect_df.loc[common_lines, target_gene].dropna()
    mutations = mutation_df.loc[scores.index, biomarker_gene]

    mutated = scores[mutations == 1]
    wt = scores[mutations == 0]

    stat, pval = stats.mannwhitneyu(mutated, wt, alternative='less')

    return {
        "target_gene": target_gene,
        "biomarker_gene": biomarker_gene,
        "n_mutated": len(mutated),
        "n_wt": len(wt),
        "mean_effect_mutated": mutated.mean(),
        "mean_effect_wt": wt.mean(),
        "pval": pval,
        "significant": pval < 0.05
    }
```

### 6. 共必需性分析

```python
import pandas as pd

def co_essentiality(gene_effect_df, target_gene, top_n=20):
    """发现依赖性谱相关性最高的基因（共必需伙伴）"""
    if target_gene not in gene_effect_df.columns:
        return None

    target_scores = gene_effect_df[target_gene].dropna()

    correlations = {}
    for gene in gene_effect_df.columns:
        if gene == target_gene:
            continue
        other_scores = gene_effect_df[gene].dropna()
        common = target_scores.index.intersection(other_scores.index)
        if len(common) < 50:
            continue
        r = target_scores[common].corr(other_scores[common])
        if not pd.isna(r):
            correlations[gene] = r

    corr_series = pd.Series(correlations).sort_values(ascending=False)
    return corr_series.head(top_n)

# 共必需基因通常共享生物复合体或通路
```

## 查询工作流

### 工作流1：癌症类型的靶点验证

1. 下载`CRISPRGeneEffect.csv`和`sample_info.csv`
2. 按癌症类型筛选细胞系
3. 计算目标基因在特定癌症类型 vs. 其他类型中的平均基因效应
4. 评估选择性：依赖性在目标癌症类型中的特异性
5. 结合突变、表达或拷贝数数据作为生物标志物交叉验证

### 工作流2：合成致死筛选

1. 识别目标基因突变/缺失的细胞系（如BRCA1突变型）
2. 计算突变型 vs. 野生型细胞系中所有基因的效应评分
3. 识别在突变系中显著更必需的基因（合成致死伙伴）
4. 按选择性和效应大小筛选

### 工作流3：化合物敏感性分析

1. 下载PRISM化合物敏感性数据（`primary-screen-replicate-treatment-info.csv`）
2. 将化合物AUC/log2(倍数变化)与基因组特征关联
3. 识别化合物敏感性的预测性生物标志物

## DepMap数据文件参考

| 文件 | 描述 |
|------|-------------|
| `CRISPRGeneEffect.csv` | CRISPR Chronos基因效应（核心依赖性数据） |
| `CRISPRGeneEffectUnscaled.csv` | 未标准化的CRISPR评分 |
| `RNAi_merged.csv` | DEMETER2 RNAi依赖性 |
| `sample_info.csv` | 细胞系元数据（谱系、疾病等） |
| `OmicsExpressionProteinCodingGenesTPMLogp1.csv` | mRNA表达量 |
| `OmicsSomaticMutationsMatrixDamaging.csv` | 有害体细胞突变（二元） |
| `OmicsCNGene.csv` | 基因拷贝数 |
| `PRISM_Repurposing_Primary_Screens_Data.csv` | 药物敏感性（重定位化合物库） |

所有文件下载地址：https://depmap.org/portal/download/all/

## 最佳实践

- **优先使用Chronos评分**（非DEMETER2）进行CRISPR分析——切割效率控制更佳
- **区分泛必需与癌症选择性**：低变异度基因（所有细胞系均必需）是劣质药物靶点
- **用表达数据验证**：细胞系中未表达的基因无论实际功能如何均显示非必需
- **使用DepMap ID识别细胞系**——cell_line_name可能模糊
- **考虑拷贝数影响**：扩增基因可能因拷贝数效应显示为必需（垃圾DNA假说）
- **多重检验校正**：全基因组生物标志物关联分析时应用FDR校正

## 扩展资源

- **DepMap门户**：https://depmap.org/portal/
- **数据下载**：https://depmap.org/portal/download/all/
- **DepMap论文**：Behan FM et al. (2019) Nature. PMID: 30971826
- **Chronos论文**：Dempster JM et al. (2021) Nature Methods. PMID: 34349281
- **GitHub**：https://github.com/broadinstitute/depmap-portal
- **Figshare**：https://figshare.com/articles/dataset/DepMap_24Q4_Public/27993966
