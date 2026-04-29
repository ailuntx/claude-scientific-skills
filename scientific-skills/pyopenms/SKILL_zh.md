---
name: pyopenms
description: 完整的质谱分析平台。用于蛋白质组学工作流程，包括特征检测、肽段鉴定、蛋白质定量和复杂LC-MS/MS流程。支持广泛的文件格式和算法。最适合蛋白质组学及综合性质谱数据处理。若需简单谱图比对和代谢物鉴定，请使用matchms。
license: 三条款BSD许可证
metadata:
    skill-author: K-Dense Inc.
---

# PyOpenMS

## 概述

PyOpenMS为计算质谱学库OpenMS提供Python绑定，支持分析蛋白质组学和代谢组学数据。可用于处理质谱文件格式、处理谱图数据、检测特征、鉴定肽段/蛋白质以及执行定量分析。

## 安装

使用uv安装：

```bash
uv uv pip install pyopenms
```

验证安装：

```python
import pyopenms
print(pyopenms.__version__)
```

## 核心功能

PyOpenMS的功能分为以下领域：

### 1. 文件I/O与数据格式

处理质谱文件格式并在不同表示形式间转换。

**支持格式**：mzML, mzXML, TraML, mzTab, FASTA, pepXML, protXML, mzIdentML, featureXML, consensusXML, idXML

基础文件读取：

```python
import pyopenms as ms

# 读取mzML文件
exp = ms.MSExperiment()
ms.MzMLFile().load("data.mzML", exp)

# 访问谱图
for spectrum in exp:
    mz, intensity = spectrum.get_peaks()
    print(f"谱图包含 {len(mz)} 个峰")
```

**详细文件处理**：参见`references/file_io.md`

### 2. 信号处理

通过平滑、滤波、质心化和归一化处理原始谱图数据。

基础谱图处理：

```python
# 使用高斯滤波器平滑谱图
gaussian = ms.GaussFilter()
params = gaussian.getParameters()
params.setValue("gaussian_width", 0.1)
gaussian.setParameters(params)
gaussian.filterExperiment(exp)
```

**算法详情**：参见`references/signal_processing.md`

### 3. 特征检测

跨谱图与样本检测并关联特征以进行定量分析。

```python
# 检测特征
ff = ms.FeatureFinder()
ff.run("centroided", exp, features, params, ms.FeatureMap())
```

**完整工作流**：参见`references/feature_detection.md`

### 4. 肽段与蛋白质鉴定

整合搜索引擎并处理鉴定结果。

**支持引擎**：Comet, Mascot, MSGFPlus, XTandem, OMSSA, Myrimatch

基础鉴定工作流：

```python
# 加载鉴定数据
protein_ids = []
peptide_ids = []
ms.IdXMLFile().load("identifications.idXML", protein_ids, peptide_ids)

# 应用FDR过滤
fdr = ms.FalseDiscoveryRate()
fdr.apply(peptide_ids)
```

**详细工作流**：参见`references/identification.md`

### 5. 代谢组学分析

执行非靶向代谢组学预处理与分析。

典型工作流：
1. 加载并处理原始数据
2. 检测特征
3. 跨样本对齐保留时间
4. 将特征关联至共识图谱
5. 使用化合物数据库注释

**完整代谢组学工作流**：参见`references/metabolomics.md`

## 数据结构

PyOpenMS使用以下主要对象：

- **MSExperiment**：谱图与色谱图的集合
- **MSSpectrum**：包含m/z与强度对的单张质谱图
- **MSChromatogram**：色谱轨迹
- **Feature**：带有质量指标的检测色谱峰
- **FeatureMap**：特征集合
- **PeptideIdentification**：肽段搜索结果
- **ProteinIdentification**：蛋白质搜索结果

**详细文档**：参见`references/data_structures.md`

## 常用工作流

### 快速入门：加载与探索数据

```python
import pyopenms as ms

# 加载mzML文件
exp = ms.MSExperiment()
ms.MzMLFile().load("sample.mzML", exp)

# 获取基础统计
print(f"谱图数量: {exp.getNrSpectra()}")
print(f"色谱图数量: {exp.getNrChromatograms()}")

# 检查首张谱图
spec = exp.getSpectrum(0)
print(f"MS层级: {spec.getMSLevel()}")
print(f"保留时间: {spec.getRT()}")
mz, intensity = spec.get_peaks()
print(f"峰数量: {len(mz)}")
```

### 参数管理

多数算法使用参数系统：

```python
# 获取算法参数
algo = ms.GaussFilter()
params = algo.getParameters()

# 查看可用参数
for param in params.keys():
    print(f"{param}: {params.getValue(param)}")

# 修改参数
params.setValue("gaussian_width", 0.2)
algo.setParameters(params)
```

### 导出至Pandas

将数据转换为pandas DataFrame进行分析：

```python
import pyopenms as ms
import pandas as pd

# 加载特征图谱
fm = ms.FeatureMap()
ms.FeatureXMLFile().load("features.featureXML", fm)

# 转换为DataFrame
df = fm.get_df()
print(df.head())
```

## 与其他工具集成

PyOpenMS可与以下工具集成：
- **Pandas**：导出数据至DataFrame
- **NumPy**：处理峰值数组
- **Scikit-learn**：质谱数据机器学习
- **Matplotlib/Seaborn**：可视化
- **R**：通过rpy2桥接

## 资源

- **官方文档**：https://pyopenms.readthedocs.io
- **OpenMS文档**：https://www.openms.org
- **GitHub**：https://github.com/OpenMS/OpenMS

## 参考文档

- `references/file_io.md` - 文件格式综合处理
- `references/signal_processing.md` - 信号处理算法
- `references/feature_detection.md` - 特征检测与关联
- `references/identification.md` - 肽段与蛋白质鉴定
- `references/metabolomics.md` - 代谢组学专用工作流
- `references/data_structures.md` - 核心对象与数据结构
