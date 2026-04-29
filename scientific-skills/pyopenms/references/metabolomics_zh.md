# 代谢组学工作流程

## 概述

PyOpenMS 提供专门的非靶向代谢组学分析工具，包括针对小分子优化的特征检测、加合物分组、化合物鉴定以及与代谢组学数据库的集成。

## 非靶向代谢组学流程

### 完整工作流

```python
import pyopenms as ms

def metabolomics_pipeline(input_files, output_dir):
    """
    完整的非靶向代谢组学工作流

    Args:
        input_files: mzML文件路径列表（每个样本一个）
        output_dir: 输出文件目录
    """

    # 步骤1：峰提取与特征检测
    feature_maps = []

    for mzml_file in input_files:
        print(f"处理 {mzml_file}...")

        # 加载数据
        exp = ms.MSExperiment()
        ms.MzMLFile().load(mzml_file, exp)

        # 必要时进行峰提取
        if not exp.getSpectrum(0).isSorted():
            picker = ms.PeakPickerHiRes()
            exp_picked = ms.MSExperiment()
            picker.pickExperiment(exp, exp_picked)
            exp = exp_picked

        # 特征检测
        ff = ms.FeatureFinder()
        params = ff.getParameters("centroided")

        # 代谢组学专用参数
        params.setValue("mass_trace:mz_tolerance", 5.0)  # ppm，代谢物需更严格
        params.setValue("mass_trace:min_spectra", 5)
        params.setValue("isotopic_pattern:charge_low", 1)
        params.setValue("isotopic_pattern:charge_high", 2)  # 主要为单电荷

        features = ms.FeatureMap()
        ff.run("centroided", exp, features, params, ms.FeatureMap())

        features.setPrimaryMSRunPath([mzml_file.encode()])
        feature_maps.append(features)

        print(f"  检测到 {features.size()} 个特征")

    # 步骤2：加合物检测与分组
    print("检测加合物...")
    adduct_grouped_maps = []

    adduct_detector = ms.MetaboliteAdductDecharger()
    params = adduct_detector.getParameters()
    params.setValue("potential_adducts", "[M+H]+,[M+Na]+,[M+K]+,[M+NH4]+,[M-H]-,[M+Cl]-")
    params.setValue("charge_min", 1)
    params.setValue("charge_max", 1)
    adduct_detector.setParameters(params)

    for fm in feature_maps:
        fm_out = ms.FeatureMap()
        adduct_detector.compute(fm, fm_out, ms.ConsensusMap())
        adduct_grouped_maps.append(fm_out)

    # 步骤3：保留时间对齐
    print("对齐保留时间...")
    aligner = ms.MapAlignmentAlgorithmPoseClustering()

    params = aligner.getParameters()
    params.setValue("max_num_peaks_considered", 1000)
    params.setValue("pairfinder:distance_MZ:max_difference", 10.0)
    params.setValue("pairfinder:distance_MZ:unit", "ppm")
    aligner.setParameters(params)

    aligned_maps = []
    transformations = []
    aligner.align(adduct_grouped_maps, aligned_maps, transformations)

    # 步骤4：特征关联
    print("关联特征...")
    grouper = ms.FeatureGroupingAlgorithmQT()

    params = grouper.getParameters()
    params.setValue("distance_RT:max_difference", 60.0)  # 秒
    params.setValue("distance_MZ:max_difference", 5.0)  # ppm
    params.setValue("distance_MZ:unit", "ppm")
    grouper.setParameters(params)

    consensus_map = ms.ConsensusMap()
    grouper.group(aligned_maps, consensus_map)

    print(f"创建 {consensus_map.size()} 个共识特征")

    # 步骤5：缺失值填充
    print("填充缺失值...")
    # Python API 未直接提供缺失值填充功能
    # 需使用 TOPP 工具 FeatureFinderMetaboIdent

    # 步骤6：导出结果
    consensus_file = f"{output_dir}/consensus.consensusXML"
    ms.ConsensusXMLFile().store(consensus_file, consensus_map)

    # 导出CSV供下游分析
    df = consensus_map.get_df()
    csv_file = f"{output_dir}/metabolite_table.csv"
    df.to_csv(csv_file, index=False)

    print(f"结果保存至 {output_dir}")

    return consensus_map

# 运行流程
input_files = ["sample1.mzML", "sample2.mzML", "sample3.mzML"]
consensus = metabolomics_pipeline(input_files, "output")
```

## 加合物检测

### 配置加合物类型

```python
# 创建加合物检测器
adduct_detector = ms.MetaboliteAdductDecharger()

# 配置常见加合物
params = adduct_detector.getParameters()

# 正离子模式加合物
positive_adducts = [
    "[M+H]+",
    "[M+Na]+",
    "[M+K]+",
    "[M+NH4]+",
    "[2M+H]+",
    "[M+H-H2O]+"
]

# 负离子模式加合物
negative_adducts = [
    "[M-H]-",
    "[M+Cl]-",
    "[M+FA-H]-",  # 甲酸盐
    "[2M-H]-"
]

# 设置为正离子模式
params.setValue("potential_adducts", ",".join(positive_adducts))
params.setValue("charge_min", 1)
params.setValue("charge_max", 1)
params.setValue("max_neutrals", 1)
adduct_detector.setParameters(params)

# 应用加合物检测
feature_map_out = ms.FeatureMap()
adduct_detector.compute(feature_map, feature_map_out, ms.ConsensusMap())
```

### 访问加合物信息

```python
# 检查加合物注释
for feature in feature_map_out:
    # 获取已注释的加合物类型
    if feature.metaValueExists("adduct"):
        adduct = feature.getMetaValue("adduct")
        neutral_mass = feature.getMetaValue("neutral_mass")
        print(f"m/z: {feature.getMZ():.4f}")
        print(f"  加合物: {adduct}")
        print(f"  中性质量: {neutral_mass:.4f}")
```

## 化合物鉴定

### 基于质量的注释

```python
# 使用化合物数据库注释特征
from pyopenms import MassDecomposition

# 加载化合物数据库（示例结构）
# 实际应用中需使用外部数据库如 HMDB, METLIN

compound_db = [
    {"name": "Glucose", "formula": "C6H12O6", "mass": 180.0634},
    {"name": "Citric acid", "formula": "C6H8O7", "mass": 192.0270},
    # ... 更多化合物
]

# 注释特征
mass_tolerance = 5.0  # ppm

for feature in feature_map:
    observed_mz = feature.getMZ()

    # 计算中性质量（假设为 [M+H]+）
    neutral_mass = observed_mz - 1.007276  # 质子质量

    # 数据库检索
    for compound in compound_db:
        mass_error_ppm = abs(neutral_mass - compound["mass"]) / compound["mass"] * 1e6

        if mass_error_ppm <= mass_tolerance:
            print(f"潜在匹配: {compound['name']}")
            print(f"  观测 m/z: {observed_mz:.4f}")
            print(f"  预期质量: {compound['mass']:.4f}")
            print(f"  误差: {mass_error_ppm:.2f} ppm")
```

### 基于MS/MS的鉴定

```python
# 加载MS2数据
exp = ms.MSExperiment()
ms.MzMLFile().load("data_with_ms2.mzML", exp)

# 提取MS2谱图
ms2_spectra = []
for spec in exp:
    if spec.getMSLevel() == 2:
        ms2_spectra.append(spec)

print(f"找到 {len(ms2_spectra)} 个MS2谱图")

# 匹配至谱图库
# (需外部工具或自定义实现)
```

## 数据归一化

### 总离子流(TIC)归一化

```python
import numpy as np

# 加载共识图
consensus_map = ms.ConsensusMap()
ms.ConsensusXMLFile().load("consensus.consensusXML", consensus_map)

# 计算每样本TIC
n_samples = len(consensus_map.getColumnHeaders())
tic_per_sample = np.zeros(n_samples)

for cons_feature in consensus_map:
    for handle in cons_feature.getFeatureList():
        map_idx = handle.getMapIndex()
        tic_per_sample[map_idx] += handle.getIntensity()

print("每样本TIC:", tic_per_sample)

# 归一化至中位TIC
median_tic = np.median(tic_per_sample)
normalization_factors = median_tic / tic_per_sample

print("归一化因子:", normalization_factors)

# 应用归一化
consensus_map_normalized = ms.ConsensusMap(consensus_map)
for cons_feature in consensus_map_normalized:
    feature_list = cons_feature.getFeatureList()
    for handle in feature_list:
        map_idx = handle.getMapIndex()
        normalized_intensity = handle.getIntensity() * normalization_factors[map_idx]
        handle.setIntensity(normalized_intensity)
```

## 质量控制

### 变异系数(CV)过滤

```python
import pandas as pd
import numpy as np

# 导出至pandas
df = consensus_map.get_df()

# 假设质控样本列名含'QC'
qc_cols = [col for col in df.columns if 'QC' in col]

if qc_cols:
    # 计算质控样本中每个特征的CV
    qc_data = df[qc_cols]
    cv = (qc_data.std(axis=1) / qc_data.mean(axis=1)) * 100

    # 过滤质控样本CV<30%的特征
    good_features = df[cv < 30]

    print(f"CV过滤前特征数: {len(df)}")
    print(f"CV过滤后特征数: {len(good_features)}")
```

### 空白样本过滤

```python
# 移除空白样本中存在的特征
blank_cols = [col for col in df.columns if 'Blank' in col]
sample_cols = [col for col in df.columns if 'Sample' in col]

if blank_cols and sample_cols:
    # 计算空白与样本平均强度
    blank_mean = df[blank_cols].mean(axis=1)
    sample_mean = df[sample_cols].mean(axis=1)

    # 保留样本强度高于空白3倍的特征
    ratio = sample_mean / (blank_mean + 1)  # 加1避免除零
    filtered_df = df[ratio > 3]

    print(f"空白过滤前特征数: {len(df)}")
    print(f"空白过滤后特征数: {len(filtered_df)}")
```

## 缺失值填补

```python
import pandas as pd
import numpy as np

# 加载数据
df = consensus_map.get_df()

# 将零替换为NaN
df = df.replace(0, np.nan)

# 统计缺失值
missing_per_feature = df.isnull().sum(axis=1)
print(f">50%缺失的特征数: {sum(missing_per_feature > len(df.columns)/2)}")

# 简单填补：用最小值的一半替换
for col in df.columns:
    if df[col].dtype in [np.float64, np.int64]:
        min_val = df[col].min() / 2  # 最小值的一半
        df[col].fillna(min_val, inplace=True)
```

## 代谢物表格导出

### 创建分析就绪表格

```python
import pandas as pd

def create_metabolite_table(consensus_map, output_file):
    """
    创建用于统计分析的代谢物定量表格
    """

    # 获取列头（文件描述）
    headers = consensus_map.getColumnHeaders()

    # 初始化数据结构
    data = {
        'mz': [],
        'rt': [],
        'feature_id': []
    }

    # 添加样本列
    for map_idx, header in headers.items():
        sample_name = header.label or f"Sample_{map_idx}"
        data[sample_name] = []

    # 提取特征数据
    for idx, cons_feature in enumerate(consensus_map):
        data['mz'].append(cons_feature.getMZ())
        data['rt'].append(cons_feature.getRT())
        data['feature_id'].append(f"F{idx:06d}")

        # 初始化强度值
        intensities = {map_idx: 0.0 for map_idx in headers.keys()}

        # 填充测量强度
        for handle in cons_feature.getFeatureList():
            map_idx = handle.getMapIndex()
            intensities[map_idx] = handle.getIntensity()

        # 加入数据结构
        for map_idx, header in headers.items():
            sample_name = header.label or f"Sample_{map_idx}"
            data[sample_name].append(intensities[map_idx])

    # 创建DataFrame
    df = pd.DataFrame(data)

    # 按保留时间排序
    df = df.sort_values('rt')

    # 保存CSV
    df.to_csv(output_file, index=False)

    print(f"包含 {len(df)} 个特征的代谢物表格保存至 {output_file}")

    return df

# 创建表格
df = create_metabolite_table(consensus_map, "metabolite_table.csv")
```

## 外部工具集成

### 导出至MetaboAnalyst

```python
def export_for_metaboanalyst(df, output_file):
    """
    格式化数据供MetaboAnalyst输入

    要求列名为样本名，行为特征
    """

    # 转置DataFrame
    # 移除元数据列
    sample_cols = [col for col in df.columns if col not in ['mz', 'rt', 'feature_id']]

    # 提取样本数据
    sample_data = df[sample_cols]

    # 转置（样本为行，特征为列）
    df_transposed = sample_data.T

    # 添加特征标识符作为列名
    df_transposed.columns = df['feature_id']

    # 保存
    df_transposed.to_csv(output_file)

    print(f"MetaboAnalyst格式保存至 {output_file}")

# 导出
export_for_metaboanalyst(df, "for_metaboanalyst.csv")
```

## 最佳实践

### 样本量与重复

- 每5-10次进样插入质控样本（混合样本）
- 运行空白样本识别污染
- 每组至少3个生物学重复
- 随机化样本进样顺序

### 参数优化

在混合质控样本上测试参数：

```python
# 测试不同质量轨迹参数
mz_tolerances = [3.0, 5.0, 10.0]
min_spectra_values = [3, 5, 7]

for tol in mz_tolerances:
    for min_spec in min_spectra_values:
        ff = ms.FeatureFinder()
        params = ff.getParameters("centroided")
        params.setValue("mass_trace:mz_tolerance", tol)
        params.setValue("mass_trace:min_spectra", min_spec)

        features = ms.FeatureMap()
        ff.run("centroided", exp, features, params, ms.FeatureMap())

        print(f"tol={tol}, min_spec={min_spec}: {features.size()} 个特征")
```

### 保留时间窗口

根据色谱方法调整：

```python
# 10分钟LC梯度
params.setValue("distance_RT:max_difference", 30.0)  # 30秒

# 60分钟LC梯度
params.setValue("distance_RT:max_difference", 90.0)  # 90秒
```
