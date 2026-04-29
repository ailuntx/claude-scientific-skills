# 核心数据结构

## 概述

PyOpenMS 使用带有 Python 绑定的 C++ 对象。理解这些核心数据结构对于高效的数据操作至关重要。

## 谱图与实验对象

### MSExperiment

用于存储完整 LC-MS 实验数据的容器（包含谱图和色谱图）。

```python
import pyopenms as ms

# 创建实验对象
exp = ms.MSExperiment()

# 从文件加载
ms.MzMLFile().load("data.mzML", exp)

# 访问属性
print(f"谱图数量: {exp.getNrSpectra()}")
print(f"色谱图数量: {exp.getNrChromatograms()}")

# 获取保留时间范围
rts = [spec.getRT() for spec in exp]
print(f"保留时间范围: {min(rts):.1f} - {max(rts):.1f} 秒")

# 访问单个谱图
spec = exp.getSpectrum(0)

# 遍历谱图
for spec in exp:
    if spec.getMSLevel() == 2:
        print(f"在 RT {spec.getRT():.2f} 处的 MS2 谱图")

# 获取元数据
exp_settings = exp.getExperimentalSettings()
instrument = exp_settings.getInstrument()
print(f"仪器: {instrument.getName()}")
```

### MSSpectrum

包含 m/z 和强度数组的单个质谱图。

```python
# 创建空谱图
spec = ms.MSSpectrum()

# 从实验对象获取
exp = ms.MSExperiment()
ms.MzMLFile().load("data.mzML", exp)
spec = exp.getSpectrum(0)

# 基本属性
print(f"MS 级别: {spec.getMSLevel()}")
print(f"保留时间: {spec.getRT():.2f} 秒")
print(f"峰数量: {spec.size()}")

# 以 numpy 数组形式获取峰数据
mz, intensity = spec.get_peaks()
print(f"m/z 范围: {mz.min():.2f} - {mz.max():.2f}")
print(f"最大强度: {intensity.max():.0f}")

# 访问单个峰
for i in range(min(5, spec.size())):  # 前 5 个峰
    print(f"峰 {i}: m/z={mz[i]:.4f}, 强度={intensity[i]:.0f}")

# 前体信息（MS2）
if spec.getMSLevel() == 2:
    precursors = spec.getPrecursors()
    if precursors:
        precursor = precursors[0]
        print(f"前体 m/z: {precursor.getMZ():.4f}")
        print(f"前体电荷: {precursor.getCharge()}")
        print(f"前体强度: {precursor.getIntensity():.0f}")

# 设置峰数据
new_mz = [100.0, 200.0, 300.0]
new_intensity = [1000.0, 2000.0, 1500.0]
spec.set_peaks((new_mz, new_intensity))
```

### MSChromatogram

色谱轨迹（TIC、XIC 或 SRM 跃迁）。

```python
# 从实验对象访问色谱图
for chrom in exp.getChromatograms():
    print(f"色谱图 ID: {chrom.getNativeID()}")

    # 获取数据
    rt, intensity = chrom.get_peaks()

    print(f"  RT 点数: {len(rt)}")
    print(f"  最大强度: {intensity.max():.0f}")

    # 前体信息（XIC）
    precursor = chrom.getPrecursor()
    print(f"  前体 m/z: {precursor.getMZ():.4f}")
```

## 特征对象

### Feature

具有二维空间范围（RT-m/z）的检测到的色谱峰。

```python
# 加载特征
feature_map = ms.FeatureMap()
ms.FeatureXMLFile().load("features.featureXML", feature_map)

# 访问单个特征
feature = feature_map[0]

# 核心属性
print(f"m/z: {feature.getMZ():.4f}")
print(f"保留时间: {feature.getRT():.2f} 秒")
print(f"强度: {feature.getIntensity():.0f}")
print(f"电荷: {feature.getCharge()}")

# 质量指标
print(f"整体质量: {feature.getOverallQuality():.3f}")
print(f"宽度（RT）: {feature.getWidth():.2f}")

# 凸包（空间范围）
hull = feature.getConvexHull()
print(f"凸包点数: {hull.getHullPoints().size()}")

# 边界框
bbox = hull.getBoundingBox()
print(f"RT 范围: {bbox.minPosition()[0]:.2f} - {bbox.maxPosition()[0]:.2f}")
print(f"m/z 范围: {bbox.minPosition()[1]:.4f} - {bbox.maxPosition()[1]:.4f}")

# 子特征（同位素）
subordinates = feature.getSubordinates()
if subordinates:
    print(f"同位素特征: {len(subordinates)}")
    for sub in subordinates:
        print(f"  m/z: {sub.getMZ():.4f}, 强度: {sub.getIntensity():.0f}")

# 元数据值
if feature.metaValueExists("label"):
    label = feature.getMetaValue("label")
    print(f"标签: {label}")
```

### FeatureMap

来自单个 LC-MS 运行的特征集合。

```python
# 创建特征图
feature_map = ms.FeatureMap()

# 从文件加载
ms.FeatureXMLFile().load("features.featureXML", feature_map)

# 访问属性
print(f"特征数量: {feature_map.size()}")

# 获取唯一特征
print(f"唯一特征: {feature_map.getUniqueId()}")

# 元数据
primary_path = feature_map.getPrimaryMSRunPath()
if primary_path:
    print(f"源文件: {primary_path[0].decode()}")

# 遍历特征
for feature in feature_map:
    print(f"特征: m/z={feature.getMZ():.4f}, RT={feature.getRT():.2f}")

# 添加新特征
new_feature = ms.Feature()
new_feature.setMZ(500.0)
new_feature.setRT(300.0)
new_feature.setIntensity(10000.0)
feature_map.push_back(new_feature)

# 排序特征
feature_map.sortByRT()  # 或 sortByMZ(), sortByIntensity()

# 导出到 pandas
df = feature_map.get_df()
print(df.head())
```

### ConsensusFeature

跨多个样本关联的特征。

```python
# 加载共识图
consensus_map = ms.ConsensusMap()
ms.ConsensusXMLFile().load("consensus.consensusXML", consensus_map)

# 访问共识特征
cons_feature = consensus_map[0]

# 共识属性
print(f"共识 m/z: {cons_feature.getMZ():.4f}")
print(f"共识 RT: {cons_feature.getRT():.2f}")
print(f"共识强度: {cons_feature.getIntensity():.0f}")

# 获取特征句柄（各图谱中的特征）
feature_list = cons_feature.getFeatureList()
print(f"存在于 {len(feature_list)} 个图谱中")

for handle in feature_list:
    map_idx = handle.getMapIndex()
    intensity = handle.getIntensity()
    mz = handle.getMZ()
    rt = handle.getRT()

    print(f"  图谱 {map_idx}: m/z={mz:.4f}, RT={rt:.2f}, 强度={intensity:.0f}")

# 获取源图谱中的唯一 ID
for handle in feature_list:
    unique_id = handle.getUniqueId()
    print(f"唯一 ID: {unique_id}")
```

### ConsensusMap

跨样本的共识特征集合。

```python
# 创建共识图
consensus_map = ms.ConsensusMap()

# 从文件加载
ms.ConsensusXMLFile().load("consensus.consensusXML", consensus_map)

# 访问属性
print(f"共识特征数量: {consensus_map.size()}")

# 列头（文件描述）
headers = consensus_map.getColumnHeaders()
print(f"文件数量: {len(headers)}")

for map_idx, description in headers.items():
    print(f"图谱 {map_idx}:")
    print(f"  文件名: {description.filename}")
    print(f"  标签: {description.label}")
    print(f"  大小: {description.size}")

# 遍历共识特征
for cons_feature in consensus_map:
    print(f"共识特征: m/z={cons_feature.getMZ():.4f}")

# 导出到 DataFrame
df = consensus_map.get_df()
```

## 鉴定对象

### PeptideIdentification

单个谱图的鉴定结果。

```python
# 加载鉴定结果
protein_ids = []
peptide_ids = []
ms.IdXMLFile().load("identifications.idXML", protein_ids, peptide_ids)

# 访问肽段鉴定
peptide_id = peptide_ids[0]

# 谱图元数据
print(f"保留时间: {peptide_id.getRT():.2f}")
print(f"m/z: {peptide_id.getMZ():.4f}")

# 鉴定元数据
print(f"标识符: {peptide_id.getIdentifier()}")
print(f"分数类型: {peptide_id.getScoreType()}")
print(f"分数越高越好: {peptide_id.isHigherScoreBetter()}")

# 获取肽段匹配
hits = peptide_id.getHits()
print(f"匹配数量: {len(hits)}")

for hit in hits:
    print(f"  序列: {hit.getSequence().toString()}")
    print(f"  分数: {hit.getScore()}")
    print(f"  电荷: {hit.getCharge()}")
```

### PeptideHit

单个肽段与谱图的匹配结果。

```python
# 访问匹配项
hit = peptide_id.getHits()[0]

# 序列信息
sequence = hit.getSequence()
print(f"序列: {sequence.toString()}")
print(f"质量: {sequence.getMonoWeight():.4f}")

# 分数与排名
print(f"分数: {hit.getScore()}")
print(f"排名: {hit.getRank()}")

# 电荷状态
print(f"电荷: {hit.getCharge()}")

# 蛋白质登录号
accessions = hit.extractProteinAccessionsSet()
for acc in accessions:
    print(f"蛋白质: {acc.decode()}")

# 元数据值（附加分数、误差）
if hit.metaValueExists("MS:1002252"):  # 质量误差
    mass_error = hit.getMetaValue("MS:1002252")
    print(f"质量误差: {mass_error:.4f} ppm")
```

### ProteinIdentification

蛋白质级别的鉴定信息。

```python
# 访问蛋白质鉴定
protein_id = protein_ids[0]

# 搜索引擎信息
print(f"搜索引擎: {protein_id.getSearchEngine()}")
print(f"搜索引擎版本: {protein_id.getSearchEngineVersion()}")

# 搜索参数
search_params = protein_id.getSearchParameters()
print(f"数据库: {search_params.db}")
print(f"酶解酶: {search_params.digestion_enzyme.getName()}")
print(f"漏切位点: {search_params.missed_cleavages}")
print(f"前体容差: {search_params.precursor_mass_tolerance}")

# 蛋白质匹配
hits = protein_id.getHits()
for hit in hits:
    print(f"登录号: {hit.getAccession()}")
    print(f"分数: {hit.getScore()}")
    print(f"覆盖率: {hit.getCoverage():.1f}%")
```

### ProteinHit

单个蛋白质鉴定结果。

```python
# 访问蛋白质匹配
protein_hit = protein_id.getHits()[0]

# 蛋白质信息
print(f"登录号: {protein_hit.getAccession()}")
print(f"描述: {protein_hit.getDescription()}")
print(f"序列: {protein_hit.getSequence()}")

# 评分
print(f"分数: {protein_hit.getScore()}")
print(f"覆盖率: {protein_hit.getCoverage():.1f}%")

# 排名
print(f"排名: {protein_hit.getRank()}")
```

## 序列对象

### AASequence

带有修饰的氨基酸序列。

```python
# 从字符串创建序列
seq = ms.AASequence.fromString("PEPTIDE")

# 基本属性
print(f"序列: {seq.toString()}")
print(f"长度: {seq.size()}")
print(f"单同位素质量: {seq.getMonoWeight():.4f}")
print(f"平均质量: {seq.getAverageWeight():.4f}")

# 单个残基
for i in range(seq.size()):
    residue = seq.getResidue(i)
    print(f"位置 {i}: {residue.getOneLetterCode()}")
    print(f"  质量: {residue.getMonoWeight():.4f}")
    print(f"  分子式: {residue.getFormula().toString()}")

# 修饰序列
mod_seq = ms.AASequence.fromString("PEPTIDEM(Oxidation)K")
print(f"已修饰: {mod_seq.isModified()}")

# 检查修饰
for i in range(mod_seq.size()):
    residue = mod_seq.getResidue(i)
    if residue.isModified():
        print(f"位置 {i} 的修饰: {residue.getModificationName()}")

# N端和C端修饰
term_mod_seq = ms.AASequence.fromString("(Acetyl)PEPTIDE(Amidated)")
```

### EmpiricalFormula

分子式表示。

```python
# 创建分子式
formula = ms.EmpiricalFormula("C6H12O6")  # 葡萄糖

# 属性
print(f"分子式: {formula.toString()}")
print(f"单同位素质量: {formula.getMonoWeight():.4f}")
print(f"平均质量: {formula.getAverageWeight():.4f}")

# 元素组成
print(f"碳原子数: {formula.getNumberOf(b'C')}")
print(f"氢原子数: {formula.getNumberOf(b'H')}")
print(f"氧原子数: {formula.getNumberOf(b'O')}")

# 算术运算
formula2 = ms.EmpiricalFormula("H2O")
combined = formula + formula2  # 添加水分子
print(f"组合式: {combined.toString()}")
```

## 参数对象

### Param

算法使用的通用参数容器。

```python
# 获取算法参数
algo = ms.GaussFilter()
params = algo.getParameters()

# 列出所有参数
for key in params.keys():
    value = params.getValue(key)
    print(f"{key}: {value}")

# 获取特定参数
gaussian_width = params.getValue("gaussian_width")
print(f"高斯宽度: {gaussian_width}")

# 设置参数
params.setValue("gaussian_width", 0.2)

# 应用修改后的参数
algo.setParameters(params)

# 复制参数
params_copy = ms.Param(params)
```

## 最佳实践

### 内存管理

```python
# 处理大文件时使用索引访问而非全量加载
indexed_mzml = ms.IndexedMzMLFileLoader()
indexed_mzml.load("large_file.mzML")

# 无需加载整个文件即可访问特定谱图
spec = indexed_mzml.getSpectrumById(100)
```

### 类型转换

```python
# 将峰数组转换为 numpy
import numpy as np

mz, intensity = spec.get_peaks()
# 这些已经是 numpy 数组

# 可执行 numpy 操作
filtered_mz = mz[intensity > 1000]
```

### 对象复制

```python
# 创建深拷贝
exp_copy = ms.MSExperiment(exp)

# 对拷贝的修改不影响原始对象
```
