# matchms 常用工作流程

本文档提供了使用 matchms 进行质谱分析的常见工作流程详细示例。

## 工作流程 1：基础谱库匹配

将未知谱图与参考库进行匹配以鉴定化合物。

```python
from matchms.importing import load_from_mgf
from matchms.filtering import default_filters, normalize_intensities
from matchms.filtering import select_by_relative_intensity, require_minimum_number_of_peaks
from matchms import calculate_scores
from matchms.similarity import CosineGreedy

# 加载参考库
print("加载参考库...")
library = list(load_from_mgf("reference_library.mgf"))

# 加载查询谱图（未知物）
print("加载查询谱图...")
queries = list(load_from_mgf("unknown_spectra.mgf"))

# 处理库谱图
print("处理库谱图...")
processed_library = []
for spectrum in library:
    spectrum = default_filters(spectrum)
    spectrum = normalize_intensities(spectrum)
    spectrum = select_by_relative_intensity(spectrum, intensity_from=0.01)
    spectrum = require_minimum_number_of_peaks(spectrum, n_required=5)
    if spectrum is not None:
        processed_library.append(spectrum)

# 处理查询谱图
print("处理查询谱图...")
processed_queries = []
for spectrum in queries:
    spectrum = default_filters(spectrum)
    spectrum = normalize_intensities(spectrum)
    spectrum = select_by_relative_intensity(spectrum, intensity_from=0.01)
    spectrum = require_minimum_number_of_peaks(spectrum, n_required=5)
    if spectrum is not None:
        processed_queries.append(spectrum)

# 计算相似度
print("计算相似度...")
scores = calculate_scores(references=processed_library,
                         queries=processed_queries,
                         similarity_function=CosineGreedy(tolerance=0.1))

# 获取每个查询的前5匹配
print("\n最佳匹配:")
for i, query in enumerate(processed_queries):
    top_matches = scores.scores_by_query(query, sort=True)[:5]

    query_name = query.get("compound_name", f"查询 {i}")
    print(f"\n{query_name}:")

    for ref_idx, score in top_matches:
        ref_spectrum = processed_library[ref_idx]
        ref_name = ref_spectrum.get("compound_name", f"参考 {ref_idx}")
        print(f"  {ref_name}: {score:.4f}")
```

---

## 工作流程 2：质量控制与数据清洗

在分析前过滤和清洗谱图数据。

```python
from matchms.importing import load_from_mgf
from matchms.exporting import save_as_mgf
from matchms.filtering import (default_filters, normalize_intensities,
                               require_precursor_mz, require_minimum_number_of_peaks,
                               require_minimum_number_of_high_peaks,
                               select_by_relative_intensity, remove_peaks_around_precursor_mz)

# 加载谱图
spectra = list(load_from_mgf("raw_data.mgf"))
print(f"已加载 {len(spectra)} 条原始谱图")

# 应用质量过滤器
cleaned_spectra = []
for spectrum in spectra:
    # 统一元数据
    spectrum = default_filters(spectrum)

    # 质量要求
    spectrum = require_precursor_mz(spectrum, minimum_accepted_mz=50.0)
    if spectrum is None:
        continue

    spectrum = require_minimum_number_of_peaks(spectrum, n_required=10)
    if spectrum is None:
        continue

    # 清洗峰
    spectrum = normalize_intensities(spectrum)
    spectrum = remove_peaks_around_precursor_mz(spectrum, mz_tolerance=17)
    spectrum = select_by_relative_intensity(spectrum, intensity_from=0.01)

    # 要求高质量峰
    spectrum = require_minimum_number_of_high_peaks(spectrum,
                                                     n_required=5,
                                                     intensity_threshold=0.05)
    if spectrum is None:
        continue

    cleaned_spectra.append(spectrum)

print(f"保留 {len(cleaned_spectra)} 条高质量谱图")
print(f"移除 {len(spectra) - len(cleaned_spectra)} 条低质量谱图")

# 保存清洗后数据
save_as_mgf(cleaned_spectra, "cleaned_data.mgf")
```

---

## 工作流程 3：多指标相似度评分

结合多种相似度指标实现稳健的化合物鉴定。

```python
from matchms.importing import load_from_mgf
from matchms.filtering import (default_filters, normalize_intensities,
                               derive_inchi_from_smiles, add_fingerprint, add_losses)
from matchms import calculate_scores
from matchms.similarity import (CosineGreedy, ModifiedCosine,
                                NeutralLossesCosine, FingerprintSimilarity)
import numpy as np

# 加载谱图
library = list(load_from_mgf("library.mgf"))
queries = list(load_from_mgf("queries.mgf"))

# 多特征处理流程
def process_for_multimetric(spectrum):
    spectrum = default_filters(spectrum)
    spectrum = normalize_intensities(spectrum)

    # 添加化学指纹
    spectrum = derive_inchi_from_smiles(spectrum)
    spectrum = add_fingerprint(spectrum, fingerprint_type="morgan2", nbits=2048)

    # 添加中性丢失
    spectrum = add_losses(spectrum, loss_mz_from=5.0, loss_mz_to=200.0)

    return spectrum

processed_library = [process_for_multimetric(s) for s in library if s is not None]
processed_queries = [process_for_multimetric(s) for s in queries if s is not None]

# 计算多种相似度分数
print("计算余弦相似度...")
cosine_scores = calculate_scores(processed_library, processed_queries,
                                 CosineGreedy(tolerance=0.1))

print("计算修正余弦相似度...")
modified_cosine_scores = calculate_scores(processed_library, processed_queries,
                                         ModifiedCosine(tolerance=0.1))

print("计算中性丢失相似度...")
neutral_losses_scores = calculate_scores(processed_library, processed_queries,
                                        NeutralLossesCosine(tolerance=0.1))

print("计算指纹相似度...")
fingerprint_scores = calculate_scores(processed_library, processed_queries,
                                      FingerprintSimilarity(similarity_measure="jaccard"))

# 加权组合分数
weights = {
    'cosine': 0.4,
    'modified_cosine': 0.3,
    'neutral_losses': 0.2,
    'fingerprint': 0.1
}

# 获取每个查询的组合分数
for i, query in enumerate(processed_queries):
    query_name = query.get("compound_name", f"查询 {i}")

    combined_scores = []
    for j, ref in enumerate(processed_library):
        combined = (weights['cosine'] * cosine_scores.scores[j, i] +
                   weights['modified_cosine'] * modified_cosine_scores.scores[j, i] +
                   weights['neutral_losses'] * neutral_losses_scores.scores[j, i] +
                   weights['fingerprint'] * fingerprint_scores.scores[j, i])
        combined_scores.append((j, combined))

    # 按组合分数排序
    combined_scores.sort(key=lambda x: x[1], reverse=True)

    print(f"\n{query_name} - 前3匹配:")
    for ref_idx, score in combined_scores[:3]:
        ref_name = processed_library[ref_idx].get("compound_name", f"参考 {ref_idx}")
        print(f"  {ref_name}: {score:.4f}")
```

---

## 工作流程 4：前体离子过滤库搜索

在谱图匹配前通过前体质量预过滤以加速搜索。

```python
from matchms.importing import load_from_mgf
from matchms.filtering import default_filters, normalize_intensities
from matchms import calculate_scores
from matchms.similarity import PrecursorMzMatch, CosineGreedy
import numpy as np

# 加载数据
library = list(load_from_mgf("large_library.mgf"))
queries = list(load_from_mgf("queries.mgf"))

# 处理谱图
processed_library = [normalize_intensities(default_filters(s)) for s in library]
processed_queries = [normalize_intensities(default_filters(s)) for s in queries]

# 步骤1：快速前体质量过滤
print("按前体质量过滤...")
mass_filter = calculate_scores(processed_library, processed_queries,
                               PrecursorMzMatch(tolerance=0.1, tolerance_type="Dalton"))

# 步骤2：仅计算匹配前体的余弦相似度
print("为过滤后的候选计算余弦相似度...")
cosine_scores = calculate_scores(processed_library, processed_queries,
                                CosineGreedy(tolerance=0.1))

# 步骤3：对余弦分数应用质量过滤器
for i, query in enumerate(processed_queries):
    candidates = []

    for j, ref in enumerate(processed_library):
        # 仅考虑前体匹配的情况
        if mass_filter.scores[j, i] > 0:
            cosine_score = cosine_scores.scores[j, i]
            candidates.append((j, cosine_score))

    # 按余弦分数排序
    candidates.sort(key=lambda x: x[1], reverse=True)

    query_name = query.get("compound_name", f"查询 {i}")
    print(f"\n{query_name} - 前5匹配（来自 {len(candidates)} 个候选）:")

    for ref_idx, score in candidates[:5]:
        ref_name = processed_library[ref_idx].get("compound_name", f"参考 {ref_idx}")
        ref_mz = processed_library[ref_idx].get("precursor_mz", "N/A")
        print(f"  {ref_name} (m/z {ref_mz}): {score:.4f}")
```

---

## 工作流程 5：构建可复用处理流程

创建标准化流程以实现一致处理。

```python
from matchms import SpectrumProcessor
from matchms.filtering import (default_filters, normalize_intensities,
                               select_by_relative_intensity,
                               remove_peaks_around_precursor_mz,
                               require_minimum_number_of_peaks,
                               derive_inchi_from_smiles, add_fingerprint)
from matchms.importing import load_from_mgf
from matchms.exporting import save_as_pickle

# 定义自定义处理流程
def create_standard_pipeline():
    """创建可复用的处理流程"""
    return SpectrumProcessor([
        default_filters,
        normalize_intensities,
        lambda s: remove_peaks_around_precursor_mz(s, mz_tolerance=17),
        lambda s: select_by_relative_intensity(s, intensity_from=0.01),
        lambda s: require_minimum_number_of_peaks(s, n_required=5),
        derive_inchi_from_smiles,
        lambda s: add_fingerprint(s, fingerprint_type="morgan2")
    ])

# 创建流程实例
pipeline = create_standard_pipeline()

# 使用相同流程处理多个数据集
datasets = ["dataset1.mgf", "dataset2.mgf", "dataset3.mgf"]

for dataset_file in datasets:
    print(f"\n处理 {dataset_file}...")

    # 加载谱图
    spectra = list(load_from_mgf(dataset_file))

    # 应用流程
    processed = []
    for spectrum in spectra:
        result = pipeline(spectrum)
        if result is not None:
            processed.append(result)

    print(f"  已加载: {len(spectra)}")
    print(f"  已处理: {len(processed)}")

    # 保存处理后的数据
    output_file = dataset_file.replace(".mgf", "_processed.pkl")
    save_as_pickle(processed, output_file)
    print(f"  保存至: {output_file}")
```

---

## 工作流程 6：格式转换与标准化

在不同质谱文件格式间转换。

```python
from matchms.importing import load_from_mzml, load_from_mgf
from matchms.exporting import save_as_mgf, save_as_msp, save_as_json
from matchms.filtering import default_filters, normalize_intensities

def convert_and_standardize(input_file, output_format="mgf"):
    """
    加载、标准化并转换质谱数据

    参数:
    -----------
    input_file : str
        输入文件路径（支持 .mzML, .mzXML, .mgf）
    output_format : str
        输出格式 ('mgf', 'msp', 或 'json')
    """
    # 确定输入格式并加载
    if input_file.endswith('.mzML') or input_file.endswith('.mzXML'):
        from matchms.importing import load_from_mzml
        spectra = list(load_from_mzml(input_file, ms_level=2))
    elif input_file.endswith('.mgf'):
        spectra = list(load_from_mgf(input_file))
    else:
        raise ValueError(f"不支持的格式: {input_file}")

    print(f"从 {input_file} 加载 {len(spectra)} 条谱图")

    # 标准化
    processed = []
    for spectrum in spectra:
        spectrum = default_filters(spectrum)
        spectrum = normalize_intensities(spectrum)
        if spectrum is not None:
            processed.append(spectrum)

    print(f"已标准化 {len(processed)} 条谱图")

    # 导出
    output_file = input_file.rsplit('.', 1)[0] + f'_standardized.{output_format}'

    if output_format == 'mgf':
        save_as_mgf(processed, output_file)
    elif output_format == 'msp':
        save_as_msp(processed, output_file)
    elif output_format == 'json':
        save_as_json(processed, output_file)
    else:
        raise ValueError(f"不支持的输出格式: {output_format}")

    print(f"保存至 {output_file}")
    return processed

# 将 mzML 转换为 MGF
convert_and_standardize("raw_data.mzML", output_format="mgf")

# 将 MGF 转换为 MSP 库格式
convert_and_standardize("library.mgf", output_format="msp")
```

---

## 工作流程 7：元数据增强与验证

用化学结构信息增强谱图并验证注释。

```python
from matchms.importing import load_from_mgf
from matchms.exporting import save_as_mgf
from matchms.filtering import (default_filters, derive_inchi_from_smiles,
                               derive_inchikey_from_inchi, derive_smiles_from_inchi,
                               add_fingerprint, repair_not_matching_annotation,
                               require_valid_annotation)

# 加载谱图
spectra = list(load_from_mgf("spectra.mgf"))

# 增强与验证
enriched_spectra = []
validation_failures = []

for i, spectrum in enumerate(spectra):
    # 基础统一
    spectrum = default_filters(spectrum)

    # 推导化学结构
    spectrum = derive_inchi_from_smiles(spectrum)
    spectrum = derive_inchikey_from_inchi(spectrum)
    spectrum = derive_smiles_from_inchi(spectrum)

    # 修复不匹配注释
    spectrum = repair_not_matching_annotation(spectrum)

    # 添加分子指纹
    spectrum = add_fingerprint(spectrum, fingerprint_type="morgan2", nbits=2048)

    # 验证
    validated = require_valid_annotation(spectrum)

    if validated is not None:
        enriched_spectra.append(validated)
    else:
        validation_failures.append(i)

print(f"成功增强: {len(enriched_spectra)}")
print(f"验证失败: {len(validation_failures)}")

# 保存增强数据
save_as_mgf(enriched_spectra, "enriched_spectra.mgf")

# 报告失败
if validation_failures:
    print("\n验证失败的谱图:")
    for idx in validation_failures[:10]:  # 显示前10条
        original = spectra[idx]
        name = original.get("compound_name", f"谱图 {idx}")
        print(f"  - {name}")
```

---

## 工作流程 8：大规模谱库比对

高效比对两个大型谱图库。

```python
from matchms.importing import load_from_mgf
from matchms.filtering import default_filters, normalize_intensities
from matchms import calculate_scores
from matchms.similarity import CosineGreedy
import numpy as

```markdown
'lib2_idx': j,
                'lib1_name': spec1.get("compound_name", f"L1_{i}"),
                'lib2_name': spec2.get("compound_name", f"L2_{j}"),
                'similarity': score
            })

# 按相似度排序
similar_pairs.sort(key=lambda x: x['similarity'], reverse=True)

print(f"\n找到 {len(similar_pairs)} 对相似度 >= {threshold} 的化合物")
print("\n前10个最相似配对：")
for pair in similar_pairs[:10]:
    print(f"{pair['lib1_name']} <-> {pair['lib2_name']}: {pair['similarity']:.4f}")

# 导出为CSV
import pandas as pd
df = pd.DataFrame(similar_pairs)
df.to_csv("library_comparison.csv", index=False)
print("\n完整结果已保存至 library_comparison.csv")
```

---

## 工作流9：离子模式特异性处理

分别处理正负离子模式质谱数据。

```python
from matchms.importing import load_from_mgf
from matchms.filtering import (default_filters, normalize_intensities,
                               require_correct_ionmode, derive_ionmode)
from matchms.exporting import save_as_mgf

# 加载混合模式质谱数据
spectra = list(load_from_mgf("mixed_modes.mgf"))

# 按离子模式分离
positive_spectra = []
negative_spectra = []
unknown_mode = []

for spectrum in spectra:
    # 标准化处理并推导离子模式
    spectrum = default_filters(spectrum)
    spectrum = derive_ionmode(spectrum)

    # 按模式分离
    ionmode = spectrum.get("ionmode")

    if ionmode == "positive":
        spectrum = normalize_intensities(spectrum)
        positive_spectra.append(spectrum)
    elif ionmode == "negative":
        spectrum = normalize_intensities(spectrum)
        negative_spectra.append(spectrum)
    else:
        unknown_mode.append(spectrum)

print(f"正离子模式: {len(positive_spectra)}")
print(f"负离子模式: {len(negative_spectra)}")
print(f"未知模式: {len(unknown_mode)}")

# 保存分离数据
save_as_mgf(positive_spectra, "positive_mode.mgf")
save_as_mgf(negative_spectra, "negative_mode.mgf")

# 执行模式特异性分析
from matchms import calculate_scores
from matchms.similarity import CosineGreedy

if len(positive_spectra) > 1:
    print("\n计算正离子模式相似度...")
    pos_scores = calculate_scores(positive_spectra, positive_spectra,
                                  CosineGreedy(tolerance=0.1))

if len(negative_spectra) > 1:
    print("计算负离子模式相似度...")
    neg_scores = calculate_scores(negative_spectra, negative_spectra,
                                  CosineGreedy(tolerance=0.1))
```

---

## 工作流10：自动化化合物鉴定报告

生成详细化合物鉴定报告。

```python
from matchms.importing import load_from_mgf
from matchms.filtering import default_filters, normalize_intensities
from matchms import calculate_scores
from matchms.similarity import CosineGreedy, ModifiedCosine
import pandas as pd

def identify_compounds(query_file, library_file, output_csv="identification_report.csv"):
    """
    生成详细报告的自动化化合物鉴定流程
    """
    # 加载数据
    print("加载数据中...")
    queries = list(load_from_mgf(query_file))
    library = list(load_from_mgf(library_file))

    # 数据处理
    proc_queries = [normalize_intensities(default_filters(s)) for s in queries]
    proc_library = [normalize_intensities(default_filters(s)) for s in library]

    # 计算相似度
    print("计算相似度中...")
    cosine_scores = calculate_scores(proc_library, proc_queries, CosineGreedy())
    modified_scores = calculate_scores(proc_library, proc_queries, ModifiedCosine())

    # 生成报告
    results = []
    for i, query in enumerate(proc_queries):
        query_name = query.get("compound_name", f"Unknown_{i}")
        query_mz = query.get("precursor_mz", "N/A")

        # 获取前5个匹配项
        cosine_matches = cosine_scores.scores_by_query(query, sort=True)[:5]

        for rank, (lib_idx, cos_score) in enumerate(cosine_matches, 1):
            ref = proc_library[lib_idx]
            mod_score = modified_scores.scores[lib_idx, i]

            results.append({
                'Query': query_name,
                'Query_mz': query_mz,
                'Rank': rank,
                'Match': ref.get("compound_name", f"Ref_{lib_idx}"),
                'Match_mz': ref.get("precursor_mz", "N/A"),
                'Cosine_Score': cos_score,
                'Modified_Cosine': mod_score,
                'InChIKey': ref.get("inchikey", "N/A")
            })

    # 创建数据框并保存
    df = pd.DataFrame(results)
    df.to_csv(output_csv, index=False)
    print(f"\n报告已保存至 {output_csv}")

    # 统计摘要
    print("\n统计摘要：")
    high_confidence = len(df[df['Cosine_Score'] >= 0.8])
    medium_confidence = len(df[(df['Cosine_Score'] >= 0.6) & (df['Cosine_Score'] < 0.8)])
    low_confidence = len(df[df['Cosine_Score'] < 0.6])

    print(f"  高置信度 (≥0.8): {high_confidence}")
    print(f"  中置信度 (0.6-0.8): {medium_confidence}")
    print(f"  低置信度 (<0.6): {low_confidence}")

    return df

# 执行鉴定流程
report = identify_compounds("unknowns.mgf", "reference_library.mgf")
```

---

## 最佳实践

1. **统一处理查询谱和参考谱**：应用相同过滤器确保比较一致性
2. **保存中间结果**：使用pickle格式快速重载处理后的质谱数据
3. **监控内存使用**：处理大文件时使用生成器替代全量加载
4. **验证数据质量**：相似度计算前应用质量过滤器
5. **选择合适的相似度指标**：CosineGreedy追求速度，ModifiedCosine适合关联化合物
6. **组合多种指标**：使用多重相似度评分提升鉴定鲁棒性
7. **先按前体质量过滤**：显著加速大型库搜索
8. **记录处理流程**：保存处理参数确保结果可复现

## 扩展资源

- matchms文档：https://matchms.readthedocs.io
- GNPS平台：https://gnps.ucsd.edu
- matchms GitHub：https://github.com/matchms/matchms
```
