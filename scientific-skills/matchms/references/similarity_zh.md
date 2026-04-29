# Matchms 相似度函数参考

本文档提供 matchms 中所有可用相似度评分方法的详细信息。

## 概述

Matchms 提供多种相似度函数用于比较质谱图。使用 `calculate_scores()` 计算参考谱图集合与查询谱图集合之间的成对相似度。

```python
from matchms import calculate_scores
from matchms.similarity import CosineGreedy

scores = calculate_scores(references=library_spectra,
                         queries=query_spectra,
                         similarity_function=CosineGreedy())
```

## 基于峰值的相似度函数

这些函数根据质谱图的峰模式（m/z 值和强度值）进行比较。

### CosineGreedy

**描述**：使用快速贪婪匹配算法计算两个谱图间的余弦相似度。峰值在指定容差范围内匹配，相似度基于匹配峰强度计算。

**适用场景**：
- 大型数据集的快速相似度计算
- 通用谱图匹配
- 当速度优先于数学最优匹配时

**参数**：
- `tolerance` (浮点数, 默认=0.1)：峰值匹配的最大 m/z 差值（道尔顿）
- `mz_power` (浮点数, 默认=0.0)：m/z 加权指数（0 表示无加权）
- `intensity_power` (浮点数, 默认=1.0)：强度加权指数

**示例**：
```python
from matchms.similarity import CosineGreedy

similarity_func = CosineGreedy(tolerance=0.1, mz_power=0.0, intensity_power=1.0)
scores = calculate_scores(references, queries, similarity_func)
```

**输出**：0.0 到 1.0 之间的相似度分数，以及匹配峰数量。

---

### CosineHungarian

**描述**：使用匈牙利算法计算余弦相似度，实现最优峰值匹配。提供数学最优的峰值分配，但速度慢于 CosineGreedy。

**适用场景**：
- 需要最优峰值匹配时
- 高质量参考库比较
- 要求可重现、数学严谨结果的研究

**参数**：
- `tolerance` (浮点数, 默认=0.1)：峰值匹配的最大 m/z 差值
- `mz_power` (浮点数, 默认=0.0)：m/z 加权指数
- `intensity_power` (浮点数, 默认=1.0)：强度加权指数

**示例**：
```python
from matchms.similarity import CosineHungarian

similarity_func = CosineHungarian(tolerance=0.1)
scores = calculate_scores(references, queries, similarity_func)
```

**输出**：0.0 到 1.0 之间的最优相似度分数，以及匹配峰信息。

**注意**：速度慢于 CosineGreedy；建议用于小型数据集或精度要求高的场景。

---

### ModifiedCosine

**描述**：通过考虑前体 m/z 差异扩展余弦相似度。允许根据前体质量差异进行质量偏移后匹配峰值，适用于比较相关化合物谱图（同位素、加合物、类似物）。

**适用场景**：
- 比较不同前体质量的谱图
- 识别结构类似物或衍生物
- 跨电离模式比较
- 当前体质量差异具有意义时

**参数**：
- `tolerance` (浮点数, 默认=0.1)：偏移后峰值匹配的最大 m/z 差值
- `mz_power` (浮点数, 默认=0.0)：m/z 加权指数
- `intensity_power` (浮点数, 默认=1.0)：强度加权指数

**示例**：
```python
from matchms.similarity import ModifiedCosine

similarity_func = ModifiedCosine(tolerance=0.1)
scores = calculate_scores(references, queries, similarity_func)
```

**要求**：两个谱图必须具有有效 precursor_mz 元数据。

---

### NeutralLossesCosine

**描述**：基于中性丢失模式而非碎片 m/z 值计算相似度。中性丢失通过碎片 m/z 减去前体 m/z 获得，特别适用于识别具有相似碎裂模式的化合物。

**适用场景**：
- 比较不同前体质量的碎裂模式
- 识别具有相似中性丢失特征的化合物
- 作为常规余弦评分的补充
- 代谢物鉴定与分类

**参数**：
- `tolerance` (浮点数, 默认=0.1)：中性丢失匹配的最大差值
- `mz_power` (浮点数, 默认=0.0)：丢失值加权指数
- `intensity_power` (浮点数, 默认=1.0)：强度加权指数

**示例**：
```python
from matchms.similarity import NeutralLossesCosine
from matchms.filtering import add_losses

# 首先为谱图添加丢失信息
spectra_with_losses = [add_losses(s) for s in spectra]

similarity_func = NeutralLossesCosine(tolerance=0.1)
scores = calculate_scores(references_with_losses, queries_with_losses, similarity_func)
```

**要求**：
- 两个谱图必须具有有效 precursor_mz 元数据
- 评分前需使用 `add_losses()` 过滤器计算中性丢失

---

## 结构相似度函数

这些函数比较分子结构而非谱图峰值。

### FingerprintSimilarity

**描述**：计算化学结构（SMILES 或 InChI）衍生的分子指纹相似度。支持多种指纹类型和相似度度量标准。

**适用场景**：
- 无谱图数据的结构相似度计算
- 结合结构与谱图相似度
- 谱图匹配前的候选预过滤
- 构效关系研究

**参数**：
- `fingerprint_type` (字符串, 默认="daylight")：指纹类型
  - `"daylight"`：Daylight 指纹
  - `"morgan1"`, `"morgan2"`, `"morgan3"`：半径为 1, 2 或 3 的 Morgan 指纹
- `similarity_measure` (字符串, 默认="jaccard")：相似度度量标准
  - `"jaccard"`：Jaccard 指数（交集 / 并集）
  - `"dice"`：Dice 系数（2 * 交集 / (大小1 + 大小2)）
  - `"cosine"`：余弦相似度

**示例**：
```python
from matchms.similarity import FingerprintSimilarity
from matchms.filtering import add_fingerprint

# 为谱图添加指纹
spectra_with_fps = [add_fingerprint(s, fingerprint_type="morgan2", nbits=2048)
                    for s in spectra]

similarity_func = FingerprintSimilarity(similarity_measure="jaccard")
scores = calculate_scores(references_with_fps, queries_with_fps, similarity_func)
```

**要求**：
- 谱图必须具有有效 SMILES 或 InChI 元数据
- 需使用 `add_fingerprint()` 过滤器计算指纹
- 需要 rdkit 库

---

## 基于元数据的相似度函数

这些函数比较元数据字段而非谱图或结构数据。

### MetadataMatch

**描述**：比较谱图间用户定义的元数据字段。支持分类数据的精确匹配和数值数据的容差匹配。

**适用场景**：
- 按实验条件过滤（碰撞能量、保留时间）
- 仪器特异性匹配
- 结合元数据约束与谱图相似度
- 自定义元数据过滤

**参数**：
- `field` (字符串)：要比较的元数据字段名
- `matching_type` (字符串, 默认="exact")：匹配方法
  - `"exact"`：精确字符串/值匹配
  - `"difference"`：数值的绝对差值
  - `"relative_difference"`：数值的相对差值
- `tolerance` (浮点数, 可选)：数值匹配的最大差值

**示例（精确匹配）**：
```python
from matchms.similarity import MetadataMatch

# 按仪器类型匹配
similarity_func = MetadataMatch(field="instrument_type", matching_type="exact")
scores = calculate_scores(references, queries, similarity_func)
```

**示例（数值匹配）**：
```python
# 在 0.5 分钟内匹配保留时间
similarity_func = MetadataMatch(field="retention_time",
                                matching_type="difference",
                                tolerance=0.5)
scores = calculate_scores(references, queries, similarity_func)
```

**输出**：精确匹配返回 1.0（匹配）或 0.0（不匹配）。数值匹配根据差值返回相似度分数。

---

### PrecursorMzMatch

**描述**：基于前体 m/z 值的二元匹配。根据前体质量是否在指定容差内匹配返回 True/False。

**适用场景**：
- 按前体质量预过滤谱图库
- 基于质量的快速候选选择
- 与其他相似度指标结合使用
- 同量异位化合物识别

**参数**：
- `tolerance` (浮点数, 默认=0.1)：匹配的最大 m/z 差值
- `tolerance_type` (字符串, 默认="Dalton")：容差单位
  - `"Dalton"`：绝对质量差
  - `"ppm"`：百万分率（相对）

**示例**：
```python
from matchms.similarity import PrecursorMzMatch

# 在 0.1 Da 内匹配前体
similarity_func = PrecursorMzMatch(tolerance=0.1, tolerance_type="Dalton")
scores = calculate_scores(references, queries, similarity_func)

# 在 10 ppm 内匹配前体
similarity_func = PrecursorMzMatch(tolerance=10, tolerance_type="ppm")
scores = calculate_scores(references, queries, similarity_func)
```

**输出**：1.0（匹配）或 0.0（不匹配）

**要求**：两个谱图必须具有有效 precursor_mz 元数据。

---

### ParentMassMatch

**描述**：基于母体质量（中性质量）的二元匹配。类似 PrecursorMzMatch，但使用计算的母体质量而非前体 m/z。

**适用场景**：
- 比较不同电离模式的谱图
- 不依赖加合物的匹配
- 基于中性质量的库搜索

**参数**：
- `tolerance` (浮点数, 默认=0.1)：匹配的最大质量差值
- `tolerance_type` (字符串, 默认="Dalton")：容差单位（"Dalton" 或 "ppm"）

**示例**：
```python
from matchms.similarity import ParentMassMatch

similarity_func = ParentMassMatch(tolerance=0.1, tolerance_type="Dalton")
scores = calculate_scores(references, queries, similarity_func)
```

**输出**：1.0（匹配）或 0.0（不匹配）

**要求**：两个谱图必须具有有效 parent_mass 元数据。

---

## 组合多个相似度函数

组合多个相似度指标以实现稳健的化合物识别：

```python
from matchms import calculate_scores
from matchms.similarity import CosineGreedy, ModifiedCosine, FingerprintSimilarity

# 计算多个相似度分数
cosine_scores = calculate_scores(refs, queries, CosineGreedy())
modified_cosine_scores = calculate_scores(refs, queries, ModifiedCosine())
fingerprint_scores = calculate_scores(refs, queries, FingerprintSimilarity())

# 加权组合分数
for i, query in enumerate(queries):
    for j, ref in enumerate(refs):
        combined_score = (0.5 * cosine_scores.scores[j, i] +
                         0.3 * modified_cosine_scores.scores[j, i] +
                         0.2 * fingerprint_scores.scores[j, i])
```

## 访问评分结果

`Scores` 对象提供多种访问结果的方法：

```python
# 获取查询的最佳匹配
best_matches = scores.scores_by_query(query_spectrum, sort=True)[:10]

# 获取 numpy 数组格式的分数
score_array = scores.scores

# 获取 pandas DataFrame 格式的分数
import pandas as pd
df = scores.to_dataframe()

# 按阈值过滤
high_scores = [(i, j, score) for i, j, score in scores.to_list() if score > 0.7]

# 保存分数
scores.to_json("scores.json")
scores.to_pickle("scores.pkl")
```

## 性能考量

**快速方法**（大型数据集）：
- CosineGreedy
- PrecursorMzMatch
- ParentMassMatch

**慢速方法**（小型数据集或高精度需求）：
- CosineHungarian
- ModifiedCosine（慢于 CosineGreedy）
- NeutralLossesCosine
- FingerprintSimilarity（需计算指纹）

**建议**：对于大规模库搜索，先用 PrecursorMzMatch 预过滤候选，再对过滤结果应用 CosineGreedy 或 ModifiedCosine。

## 常见相似度工作流程

### 标准库匹配
```python
from matchms.similarity import CosineGreedy

scores = calculate_scores(library_spectra, query_spectra,
                         CosineGreedy(tolerance=0.1))
```

### 多指标匹配
```python
from matchms.similarity import CosineGreedy, ModifiedCosine, FingerprintSimilarity

# 谱图相似度
cosine = calculate_scores(refs, queries, CosineGreedy())
modified = calculate_scores(refs, queries, ModifiedCosine())

# 结构相似度
fingerprint = calculate_scores(refs, queries, FingerprintSimilarity())
```

### 前体过滤匹配
```python
from matchms.similarity import PrecursorMzMatch, CosineGreedy

# 首先按前体质量过滤
mass_filter = calculate_scores(refs, queries, PrecursorMzMatch(tolerance=0.1))

# 仅对匹配前体计算余弦相似度
cosine_scores = calculate_scores(refs, queries, CosineGreedy())
```

## 延伸阅读

详细 API 文档、参数说明和数学公式请参阅：
https://matchms.readthedocs.io/en/latest/api/matchms.similarity.html
