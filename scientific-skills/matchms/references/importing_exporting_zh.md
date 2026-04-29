# matchms 导入与导出参考指南

本文档详细介绍了 matchms 中用于加载和保存质谱数据的所有文件格式支持。

## 导入光谱数据

Matchms 提供专用函数用于从多种文件格式加载光谱数据。所有导入函数均返回生成器，以实现大文件的内存高效处理。

### 通用导入模式

```python
from matchms.importing import load_from_mgf

# 加载光谱（返回生成器）
spectra_generator = load_from_mgf("spectra.mgf")

# 转换为列表进行处理
spectra = list(spectra_generator)
```

## 支持的导入格式

### MGF (Mascot通用格式)

**函数**: `load_from_mgf(filename, metadata_harmonization=True)`

**描述**: 从MGF文件加载光谱数据，这是质谱数据交换的常用格式。

**参数**:
- `filename` (str): MGF文件路径
- `metadata_harmonization` (bool, 默认=True): 应用自动元数据键名标准化

**示例**:
```python
from matchms.importing import load_from_mgf

# 带元数据标准化加载
spectra = list(load_from_mgf("data.mgf"))

# 不带标准化加载
spectra = list(load_from_mgf("data.mgf", metadata_harmonization=False))
```

**MGF格式**: 基于文本的格式，包含BEGIN IONS/END IONS块，其中包含元数据和峰列表。

---

### MSP (NIST质谱库格式)

**函数**: `load_from_msp(filename, metadata_harmonization=True)`

**描述**: 从MSP文件加载光谱数据，常用于光谱库。

**参数**:
- `filename` (str): MSP文件路径
- `metadata_harmonization` (bool, 默认=True): 应用自动元数据键名标准化

**示例**:
```python
from matchms.importing import load_from_msp

spectra = list(load_from_msp("library.msp"))
```

**MSP格式**: 基于文本的格式，包含Name/MW/Comment字段及峰列表。

---

### mzML (质谱标记语言)

**函数**: `load_from_mzml(filename, ms_level=2, metadata_harmonization=True)`

**描述**: 从mzML文件加载光谱数据，这是原始质谱数据的标准XML格式。

**参数**:
- `filename` (str): mzML文件路径
- `ms_level` (int, 默认=2): 提取的MS级别（1为MS1，2为MS2/串联）
- `metadata_harmonization` (bool, 默认=True): 应用自动元数据键名标准化

**示例**:
```python
from matchms.importing import load_from_mzml

# 加载MS2光谱（默认）
ms2_spectra = list(load_from_mzml("data.mzML"))

# 加载MS1光谱
ms1_spectra = list(load_from_mzml("data.mzML", ms_level=1))
```

**mzML格式**: 基于XML的标准格式，包含原始仪器数据和丰富元数据。

---

### mzXML

**函数**: `load_from_mzxml(filename, ms_level=2, metadata_harmonization=True)`

**描述**: 从mzXML文件加载光谱数据，这是早期的质谱XML格式。

**参数**:
- `filename` (str): mzXML文件路径
- `ms_level` (int, 默认=2): 提取的MS级别
- `metadata_harmonization` (bool, 默认=True): 应用自动元数据键名标准化

**示例**:
```python
from matchms.importing import load_from_mzxml

spectra = list(load_from_mzxml("data.mzXML"))
```

**mzXML格式**: 基于XML的格式，是mzML的前身。

---

### JSON (GNPS格式)

**函数**: `load_from_json(filename, metadata_harmonization=True)`

**描述**: 从JSON文件加载光谱数据，特别是兼容GNPS的JSON格式。

**参数**:
- `filename` (str): JSON文件路径
- `metadata_harmonization` (bool, 默认=True): 应用自动元数据键名标准化

**示例**:
```python
from matchms.importing import load_from_json

spectra = list(load_from_json("spectra.json"))
```

**JSON格式**: 结构化JSON格式，包含光谱元数据和峰数组。

---

### Pickle (Python序列化)

**函数**: `load_from_pickle(filename)`

**描述**: 从pickle文件加载先前保存的matchms光谱对象。用于快速加载预处理后的光谱。

**参数**:
- `filename` (str): pickle文件路径

**示例**:
```python
from matchms.importing import load_from_pickle

spectra = list(load_from_pickle("processed_spectra.pkl"))
```

**使用场景**: 保存和加载预处理光谱以加速后续分析。

---

### USI (通用光谱标识符)

**函数**: `load_from_usi(usi)`

**描述**: 从代谢组学USI引用加载单个光谱。

**参数**:
- `usi` (str): 通用光谱标识符字符串

**示例**:
```python
from matchms.importing import load_from_usi

usi = "mzspec:GNPS:TASK-...:spectrum..."
spectrum = load_from_usi(usi)
```

**USI格式**: 用于从在线存储库访问光谱的标准化标识符。

---

## 导出光谱数据

Matchms提供函数将处理后的光谱保存为多种格式，便于共享和归档。

### MGF导出

**函数**: `save_as_mgf(spectra, filename, write_mode='w')`

**描述**: 将光谱保存为MGF格式。

**参数**:
- `spectra` (list): 要保存的光谱对象列表
- `filename` (str): 输出文件路径
- `write_mode` (str, 默认='w'): 文件写入模式（'w'为覆盖写入，'a'为追加）

**示例**:
```python
from matchms.exporting import save_as_mgf

save_as_mgf(processed_spectra, "output.mgf")
```

---

### MSP导出

**函数**: `save_as_msp(spectra, filename, write_mode='w')`

**描述**: 将光谱保存为MSP格式。

**参数**:
- `spectra` (list): 要保存的光谱对象列表
- `filename` (str): 输出文件路径
- `write_mode` (str, 默认='w'): 文件写入模式

**示例**:
```python
from matchms.exporting import save_as_msp

save_as_msp(library_spectra, "library.msp")
```

---

### JSON导出

**函数**: `save_as_json(spectra, filename, write_mode='w')`

**描述**: 将光谱保存为JSON格式（兼容GNPS）。

**参数**:
- `spectra` (list): 要保存的光谱对象列表
- `filename` (str): 输出文件路径
- `write_mode` (str, 默认='w'): 文件写入模式

**示例**:
```python
from matchms.exporting import save_as_json

save_as_json(spectra, "spectra.json")
```

---

### Pickle导出

**函数**: `save_as_pickle(spectra, filename)`

**描述**: 将光谱保存为Python pickle文件。保留所有光谱属性且加载速度最快。

**参数**:
- `spectra` (list): 要保存的光谱对象列表
- `filename` (str): 输出文件路径

**示例**:
```python
from matchms.exporting import save_as_pickle

save_as_pickle(processed_spectra, "processed.pkl")
```

**优势**:
- 快速保存和加载
- 保留精确的光谱状态
- 无格式转换开销

**劣势**:
- 非人类可读
- 特定于Python（不可跨语言移植）
- pickle格式可能不兼容不同Python版本

---

## 完整导入/导出工作流

### 预处理与保存流程

```python
from matchms.importing import load_from_mgf
from matchms.exporting import save_as_mgf, save_as_pickle
from matchms.filtering import default_filters, normalize_intensities
from matchms.filtering import select_by_relative_intensity

# 加载原始光谱
spectra = list(load_from_mgf("raw_data.mgf"))

# 处理光谱
processed = []
for spectrum in spectra:
    spectrum = default_filters(spectrum)
    spectrum = normalize_intensities(spectrum)
    spectrum = select_by_relative_intensity(spectrum, intensity_from=0.01)
    if spectrum is not None:
        processed.append(spectrum)

# 保存处理后的光谱（MGF格式用于共享）
save_as_mgf(processed, "processed_data.mgf")

# 保存为pickle以便快速重载
save_as_pickle(processed, "processed_data.pkl")
```

### 格式转换

```python
from matchms.importing import load_from_mzml
from matchms.exporting import save_as_mgf, save_as_msp

# 将mzML转换为MGF
spectra = list(load_from_mzml("data.mzML", ms_level=2))
save_as_mgf(spectra, "data.mgf")

# 转换为MSP库格式
save_as_msp(spectra, "data.msp")
```

### 从多文件加载

```python
from matchms.importing import load_from_mgf
import glob

# 加载目录中所有MGF文件
all_spectra = []
for mgf_file in glob.glob("data/*.mgf"):
    spectra = list(load_from_mgf(mgf_file))
    all_spectra.extend(spectra)

print(f"从多个文件加载了 {len(all_spectra)} 个光谱")
```

### 内存高效处理

```python
from matchms.importing import load_from_mgf
from matchms.exporting import save_as_mgf
from matchms.filtering import default_filters, normalize_intensities

# 处理大文件时不全部加载到内存
def process_spectrum(spectrum):
    spectrum = default_filters(spectrum)
    spectrum = normalize_intensities(spectrum)
    return spectrum

# 流式处理
with open("output.mgf", 'w') as outfile:
    for spectrum in load_from_mgf("large_file.mgf"):
        processed = process_spectrum(spectrum)
        if processed is not None:
            # 立即写入而不存储在内存中
            save_as_mgf([processed], outfile, write_mode='a')
```

## 格式选择指南

**MGF**:
- ✓ 广泛支持
- ✓ 人类可读
- ✓ 适合数据共享
- ✓ 中等文件大小
- 最佳场景：数据交换、GNPS上传、发表数据

**MSP**:
- ✓ 光谱库标准
- ✓ 人类可读
- ✓ 良好元数据支持
- 最佳场景：参考库、NIST格式兼容

**JSON**:
- ✓ 结构化格式
- ✓ GNPS兼容
- ✓ 易于程序解析
- 最佳场景：Web应用、GNPS集成、结构化数据

**Pickle**:
- ✓ 最快保存/加载
- ✓ 保留精确状态
- ✗ 不可跨语言移植
- ✗ 非人类可读
- 最佳场景：中间处理、纯Python工作流

**mzML/mzXML**:
- ✓ 原始仪器数据
- ✓ 丰富元数据
- ✓ 行业标准
- ✗ 大文件尺寸
- ✗ 解析较慢
- 最佳场景：原始数据归档、多级MS数据

## 元数据标准化

大多数导入函数中的`metadata_harmonization`参数可自动标准化元数据键名：

```python
# 无标准化
spectrum = load_from_mgf("data.mgf", metadata_harmonization=False)
# 可能包含: "PRECURSOR_MZ", "Precursor_mz", "precursormz"

# 带标准化（默认）
spectrum = load_from_mgf("data.mgf", metadata_harmonization=True)
# 标准化为: "precursor_mz"
```

**推荐**: 保持标准化启用（默认），以确保跨不同数据源的元数据访问一致性。

## 文件格式规范

详细格式规范：
- **MGF**: http://www.matrixscience.com/help/data_file_help.html
- **MSP**: https://chemdata.nist.gov/mass-spc/ms-search/
- **mzML**: http://www.psidev.info/mzML
- **GNPS JSON**: https://gnps.ucsd.edu/

## 扩展阅读

完整API文档：
https://matchms.readthedocs.io/en/latest/api/matchms.importing.html
https://matchms.readthedocs.io/en/latest/api/matchms.exporting.html
