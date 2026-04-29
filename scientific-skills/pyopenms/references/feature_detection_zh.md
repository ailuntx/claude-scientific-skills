# 特征检测与关联

## 概述

特征检测用于识别LC-MS数据中的持久信号（色谱峰）。特征关联将多个样本中的特征进行组合，实现定量比较。

## 特征检测基础

一个特征代表一个色谱峰，其特征包括：
- m/z值（质荷比）
- 保留时间（RT）
- 强度值
- 质量评分
- 凸包（在RT-m/z空间中的空间范围）

## 特征查找

### 多重特征查找器（FFM）

在中心化数据中进行特征检测的标准算法：

```python
import pyopenms as ms

# 加载中心化数据
exp = ms.MSExperiment()
ms.MzMLFile().load("centroided.mzML", exp)

# 创建特征查找器
ff = ms.FeatureFinder()

# 获取默认参数
params = ff.getParameters("centroided")

# 修改关键参数
params.setValue("mass_trace:mz_tolerance", 10.0)  # ppm单位
params.setValue("mass_trace:min_spectra", 7)  # 每个特征最小扫描次数
params.setValue("isotopic_pattern:charge_low", 1)
params.setValue("isotopic_pattern:charge_high", 4)

# 执行特征检测
features = ms.FeatureMap()
ff.run("centroided", exp, features, params, ms.FeatureMap())

print(f"检测到 {features.size()} 个特征")

# 保存特征
ms.FeatureXMLFile().store("features.featureXML", features)
```

### 代谢组学特征查找器

针对小分子优化：

```python
# 创建代谢组学特征查找器
ff = ms.FeatureFinder()

# 获取代谢组学专用参数
params = ff.getParameters("centroided")

# 配置代谢组学参数
params.setValue("mass_trace:mz_tolerance", 5.0)  # 更低容差
params.setValue("mass_trace:min_spectra", 5)
params.setValue("isotopic_pattern:charge_low", 1)  # 主要单电荷
params.setValue("isotopic_pattern:charge_high", 2)

# 执行检测
features = ms.FeatureMap()
ff.run("centroided", exp, features, params, ms.FeatureMap())
```

## 访问特征数据

### 遍历特征

```python
# 加载特征
feature_map = ms.FeatureMap()
ms.FeatureXMLFile().load("features.featureXML", feature_map)

# 访问单个特征
for feature in feature_map:
    print(f"m/z: {feature.getMZ():.4f}")
    print(f"RT: {feature.getRT():.2f}")
    print(f"强度: {feature.getIntensity():.0f}")
    print(f"电荷: {feature.getCharge()}")
    print(f"质量: {feature.getOverallQuality():.3f}")
    print(f"宽度(RT): {feature.getWidth():.2f}")

    # 获取凸包
    hull = feature.getConvexHull()
    print(f"凸包点数: {hull.getHullPoints().size()}")
```

### 特征子项（同位素模式）

```python
# 访问同位素模式
for feature in feature_map:
    # 获取子特征（同位素）
    subordinates = feature.getSubordinates()

    if subordinates:
        print(f"主特征 m/z: {feature.getMZ():.4f}")
        for sub in subordinates:
            print(f"  同位素 m/z: {sub.getMZ():.4f}")
            print(f"  同位素强度: {sub.getIntensity():.0f}")
```

### 导出到Pandas

```python
import pandas as pd

# 转换为DataFrame
df = feature_map.get_df()

print(df.columns)
# 典型列名: RT, mz, intensity, charge, quality

# 分析特征
print(f"平均强度: {df['intensity'].mean()}")
print(f"RT范围: {df['RT'].min():.1f} - {df['RT'].max():.1f}")
```

## 特征关联

### 图谱对齐

关联前对齐保留时间：

```python
# 加载多个特征图谱
fm1 = ms.FeatureMap()
fm2 = ms.FeatureMap()
ms.FeatureXMLFile().load("sample1.featureXML", fm1)
ms.FeatureXMLFile().load("sample2.featureXML", fm2)

# 创建对齐器
aligner = ms.MapAlignmentAlgorithmPoseClustering()

# 对齐图谱
fm_aligned = []
transformations = []
aligner.align([fm1, fm2], fm_aligned, transformations)
```

### 特征关联算法

跨样本关联特征：

```python
# 创建特征分组算法
grouper = ms.FeatureGroupingAlgorithmQT()

# 配置参数
params = grouper.getParameters()
params.setValue("distance_RT:max_difference", 30.0)  # 最大RT差异(秒)
params.setValue("distance_MZ:max_difference", 10.0)  # 最大m/z差异(ppm)
params.setValue("distance_MZ:unit", "ppm")
grouper.setParameters(params)

# 准备特征图谱
feature_maps = [fm1, fm2, fm3]

# 创建共识图谱
consensus_map = ms.ConsensusMap()

# 关联特征
grouper.group(feature_maps, consensus_map)

print(f"创建 {consensus_map.size()} 个共识特征")

# 保存共识图谱
ms.ConsensusXMLFile().store("consensus.consensusXML", consensus_map)
```

## 共识特征

### 访问共识数据

```python
# 加载共识图谱
consensus_map = ms.ConsensusMap()
ms.ConsensusXMLFile().load("consensus.consensusXML", consensus_map)

# 遍历共识特征
for cons_feature in consensus_map:
    print(f"共识 m/z: {cons_feature.getMZ():.4f}")
    print(f"共识 RT: {cons_feature.getRT():.2f}")

    # 获取各图谱中的特征
    for handle in cons_feature.getFeatureList():
        map_idx = handle.getMapIndex()
        intensity = handle.getIntensity()
        print(f"  样本 {map_idx}: 强度 {intensity:.0f}")
```

### 共识图谱元数据

```python
# 访问文件描述（图谱元数据）
file_descriptions = consensus_map.getColumnHeaders()

for map_idx, description in file_descriptions.items():
    print(f"图谱 {map_idx}:")
    print(f"  文件名: {description.filename}")
    print(f"  标签: {description.label}")
    print(f"  大小: {description.size}")
```

## 加合物检测

识别同一分子的不同电离形态：

```python
# 创建加合物检测器
adduct_detector = ms.MetaboliteAdductDecharger()

# 配置参数
params = adduct_detector.getParameters()
params.setValue("potential_adducts", "[M+H]+,[M+Na]+,[M+K]+,[M-H]-")
params.setValue("charge_min", 1)
params.setValue("charge_max", 1)
params.setValue("max_neutrals", 1)
adduct_detector.setParameters(params)

# 检测加合物
feature_map_out = ms.FeatureMap()
adduct_detector.compute(feature_map, feature_map_out, ms.ConsensusMap())
```

## 完整特征检测流程

### 端到端示例

```python
import pyopenms as ms

def feature_detection_workflow(input_files, output_consensus):
    """
    完整流程：跨样本的特征检测与关联

    参数:
        input_files: mzML文件路径列表
        output_consensus: 输出consensusXML文件路径
    """

    feature_maps = []

    # 步骤1：检测每个文件中的特征
    for mzml_file in input_files:
        print(f"处理 {mzml_file}...")

        # 加载实验数据
        exp = ms.MSExperiment()
        ms.MzMLFile().load(mzml_file, exp)

        # 查找特征
        ff = ms.FeatureFinder()
        params = ff.getParameters("centroided")
        params.setValue("mass_trace:mz_tolerance", 10.0)
        params.setValue("mass_trace:min_spectra", 7)

        features = ms.FeatureMap()
        ff.run("centroided", exp, features, params, ms.FeatureMap())

        # 在特征图中存储文件名
        features.setPrimaryMSRunPath([mzml_file.encode()])

        feature_maps.append(features)
        print(f"  发现 {features.size()} 个特征")

    # 步骤2：对齐保留时间
    print("对齐保留时间...")
    aligner = ms.MapAlignmentAlgorithmPoseClustering()
    aligned_maps = []
    transformations = []
    aligner.align(feature_maps, aligned_maps, transformations)

    # 步骤3：关联特征
    print("跨样本关联特征...")
    grouper = ms.FeatureGroupingAlgorithmQT()
    params = grouper.getParameters()
    params.setValue("distance_RT:max_difference", 30.0)
    params.setValue("distance_MZ:max_difference", 10.0)
    params.setValue("distance_MZ:unit", "ppm")
    grouper.setParameters(params)

    consensus_map = ms.ConsensusMap()
    grouper.group(aligned_maps, consensus_map)

    # 保存结果
    ms.ConsensusXMLFile().store(output_consensus, consensus_map)

    print(f"创建 {consensus_map.size()} 个共识特征")
    print(f"结果保存至 {output_consensus}")

    return consensus_map

# 执行流程
input_files = ["sample1.mzML", "sample2.mzML", "sample3.mzML"]
consensus = feature_detection_workflow(input_files, "consensus.consensusXML")
```

## 特征过滤

### 按质量过滤

```python
# 根据质量评分过滤特征
filtered_features = ms.FeatureMap()

for feature in feature_map:
    if feature.getOverallQuality() > 0.5:  # 质量阈值
        filtered_features.push_back(feature)

print(f"保留 {filtered_features.size()} 个高质量特征")
```

### 按强度过滤

```python
# 仅保留高强度特征
min_intensity = 10000

filtered_features = ms.FeatureMap()
for feature in feature_map:
    if feature.getIntensity() >= min_intensity:
        filtered_features.push_back(feature)
```

### 按m/z范围过滤

```python
# 提取特定m/z范围内的特征
mz_min = 200.0
mz_max = 800.0

filtered_features = ms.FeatureMap()
for feature in feature_map:
    mz = feature.getMZ()
    if mz_min <= mz <= mz_max:
        filtered_features.push_back(feature)
```

## 特征注释

### 添加鉴定信息

```python
# 用肽段鉴定信息注释特征
# 加载鉴定结果
protein_ids = []
peptide_ids = []
ms.IdXMLFile().load("identifications.idXML", protein_ids, peptide_ids)

# 创建ID映射器
mapper = ms.IDMapper()

# 将ID映射到特征
mapper.annotate(feature_map, peptide_ids, protein_ids)

# 检查注释
for feature in feature_map:
    peptide_ids_for_feature = feature.getPeptideIdentifications()
    if peptide_ids_for_feature:
        print(f"在 {feature.getMZ():.4f} m/z 处鉴定到特征")
```

## 最佳实践

### 参数优化

根据数据类型优化参数：

```python
# 测试不同容差值
mz_tolerances = [5.0, 10.0, 20.0]  # ppm单位

for tol in mz_tolerances:
    ff = ms.FeatureFinder()
    params = ff.getParameters("centroided")
    params.setValue("mass_trace:mz_tolerance", tol)

    features = ms.FeatureMap()
    ff.run("centroided", exp, features, params, ms.FeatureMap())

    print(f"容差 {tol} ppm: 检测到 {features.size()} 个特征")
```

### 可视化检查

导出特征进行可视化：

```python
# 转换为DataFrame用于绘图
df = feature_map.get_df()

import matplotlib.pyplot as plt

plt.figure(figsize=(10, 6))
plt.scatter(df['RT'], df['mz'], s=df['intensity']/1000, alpha=0.5)
plt.xlabel('保留时间 (秒)')
plt.ylabel('m/z')
plt.title('特征图谱')
plt.colorbar(label='强度 (缩放后)')
plt.show()
```
