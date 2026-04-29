---
name: matchms
description: 用于代谢组学的谱图相似度与化合物鉴定。用于比较质谱、计算相似度得分（余弦、修正余弦）以及从谱图库中鉴定未知化合物。最适合代谢物鉴定、谱图匹配和库搜索。完整的LC-MS/MS蛋白质组学流程请使用pyopenms。
license: Apache-2.0 许可证
metadata:
    skill-author: K-Dense Inc.
---

# Matchms

## 概述

Matchms 是一个用于质谱数据处理与分析的开源 Python 库。支持从多种格式导入谱图、标准化元数据、过滤峰、计算谱图相似度，并构建可复现的分析工作流。

## 核心功能

### 1. 质谱数据导入与导出

从多种文件格式加载谱图并导出处理后的数据：

```python
from matchms.importing import load_from_mgf, load_from_mzml, load_from_msp, load_from_json
from matchms.exporting import save_as_mgf, save_as_msp, save_as_json

# 导入谱图
spectra = list(load_from_mgf("spectra.mgf"))
spectra = list(load_from_mzml("data.mzML"))
spectra = list(load_from_msp("library.msp"))

# 导出处理后的谱图
save_as_mgf(spectra, "output.mgf")
save_as_json(spectra, "output.json")
```

**支持格式：**
- mzML 和 mzXML（原始质谱格式）
- MGF（Mascot通用格式）
- MSP（谱图库格式）
- JSON（兼容GNPS）
- metabolomics-USI 引用
- Pickle（Python序列化）

详细导入/导出文档请查阅 `references/importing_exporting.md`。

### 2. 谱图过滤与处理

应用全面过滤器标准化元数据并优化峰数据：

```python
from matchms.filtering import default_filters, normalize_intensities
from matchms.filtering import select_by_relative_intensity, require_minimum_number_of_peaks

# 应用默认元数据协调过滤器
spectrum = default_filters(spectrum)

# 标准化峰强度
spectrum = normalize_intensities(spectrum)

# 按相对强度过滤峰
spectrum = select_by_relative_intensity(spectrum, intensity_from=0.01, intensity_to=1.0)

# 要求最小峰数量
spectrum = require_minimum_number_of_peaks(spectrum, n_required=5)
```

**过滤器类别：**
- **元数据处理**：协调化合物名称、推导化学结构、标准化加合物、校正电荷
- **峰过滤**：强度归一化、按m/z或强度筛选、移除前体峰
- **质量控制**：要求最小峰数、验证前体m/z、确保元数据完整性
- **化学注释**：添加指纹、推导InChI/SMILES、修复结构不匹配

Matchms 提供 40+ 过滤器。完整过滤器参考请查阅 `references/filtering.md`。

### 3. 计算谱图相似度

使用多种相似度指标比较谱图：

```python
from matchms import calculate_scores
from matchms.similarity import CosineGreedy, ModifiedCosine, CosineHungarian

# 计算余弦相似度（快速贪婪算法）
scores = calculate_scores(references=library_spectra,
                         queries=query_spectra,
                         similarity_function=CosineGreedy())

# 计算修正余弦（考虑前体m/z差异）
scores = calculate_scores(references=library_spectra,
                         queries=query_spectra,
                         similarity_function=ModifiedCosine(tolerance=0.1))

# 获取最佳匹配
best_matches = scores.scores_by_query(query_spectra[0], sort=True)[:10]
```

**可用相似度函数：**
- **CosineGreedy/CosineHungarian**：基于峰的余弦相似度（不同匹配算法）
- **ModifiedCosine**：考虑前体质量差异的余弦相似度
- **NeutralLossesCosine**：基于中性丢失模式的相似度
- **FingerprintSimilarity**：使用指纹的分子结构相似度
- **MetadataMatch**：比较用户定义元数据字段
- **PrecursorMzMatch/ParentMassMatch**：基于质量的简单过滤

详细相似度函数文档请查阅 `references/similarity.md`。

### 4. 构建处理流程

创建可复现的多步骤分析工作流：

```python
from matchms import SpectrumProcessor
from matchms.filtering import default_filters, normalize_intensities
from matchms.filtering import select_by_relative_intensity, remove_peaks_around_precursor_mz

# 定义处理流程
processor = SpectrumProcessor([
    default_filters,
    normalize_intensities,
    lambda s: select_by_relative_intensity(s, intensity_from=0.01),
    lambda s: remove_peaks_around_precursor_mz(s, mz_tolerance=17)
])

# 应用于所有谱图
processed_spectra = [processor(s) for s in spectra]
```

### 5. 使用谱图对象

核心 `Spectrum` 类包含质谱数据：

```python
from matchms import Spectrum
import numpy as np

# 创建谱图
mz = np.array([100.0, 150.0, 200.0, 250.0])
intensities = np.array([0.1, 0.5, 0.9, 0.3])
metadata = {"precursor_mz": 250.5, "ionmode": "positive"}

spectrum = Spectrum(mz=mz, intensities=intensities, metadata=metadata)

# 访问谱图属性
print(spectrum.peaks.mz)           # m/z值
print(spectrum.peaks.intensities)  # 强度值
print(spectrum.get("precursor_mz")) # 元数据字段

# 可视化谱图
spectrum.plot()
spectrum.plot_against(reference_spectrum)
```

### 6. 元数据管理

标准化与协调谱图元数据：

```python
# 元数据自动协调
spectrum.set("Precursor_mz", 250.5)  # 自动转为小写键名
print(spectrum.get("precursor_mz"))   # 返回250.5

# 推导化学信息
from matchms.filtering import derive_inchi_from_smiles, derive_inchikey_from_inchi
from matchms.filtering import add_fingerprint

spectrum = derive_inchi_from_smiles(spectrum)
spectrum = derive_inchikey_from_inchi(spectrum)
spectrum = add_fingerprint(spectrum, fingerprint_type="morgan", nbits=2048)
```

## 常见工作流

典型质谱分析工作流包括：
- 加载与预处理谱图库
- 将未知谱图与参考库匹配
- 质量过滤与数据清洗
- 大规模相似度比较
- 基于网络的谱图聚类

详细示例请查阅 `references/workflows.md`。

## 安装

```bash
uv pip install matchms
```

分子结构处理（SMILES, InChI）：
```bash
uv pip install matchms[chemistry]
```

## 参考文档

详细参考文档位于 `references/` 目录：
- `filtering.md` - 完整过滤器函数参考及说明
- `similarity.md` - 所有相似度指标及适用场景
- `importing_exporting.md` - 文件格式详情与I/O操作
- `workflows.md` - 常见分析模式与示例

按需查阅这些参考文档以获取 matchms 特定功能的详细信息。
