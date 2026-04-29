# 文件输入输出与数据格式

## 概述

PyOpenMS 支持多种质谱文件格式的读写操作。本指南涵盖文件处理策略及特定格式的操作方法。

## 支持的格式

### 谱图数据格式

- **mzML**: 基于XML的质谱数据标准格式
- **mzXML**: 早期基于XML的格式
- **mzData**: XML格式（已弃用但仍支持）

### 鉴定结果格式

- **idXML**: OpenMS原生鉴定格式
- **mzIdentML**: 鉴定数据的标准XML格式
- **pepXML**: X! Tandem格式
- **protXML**: 蛋白质鉴定格式

### 特征与定量格式

- **featureXML**: OpenMS特征检测格式
- **consensusXML**: 跨样本一致性特征格式
- **mzTab**: 报告用制表符分隔格式

### 序列与库格式

- **FASTA**: 蛋白质/多肽序列
- **TraML**: 靶向实验的转换列表

## 读取mzML文件

### 内存加载

将整个文件加载到内存（适用于小文件）：

```python
import pyopenms as ms

# 创建实验容器
exp = ms.MSExperiment()

# 加载文件
ms.MzMLFile().load("sample.mzML", exp)

# 访问数据
print(f"谱图数量: {exp.getNrSpectra()}")
print(f"色谱图数量: {exp.getNrChromatograms()}")
```

### 索引访问

大文件的高效随机访问：

```python
# 创建索引访问
indexed_mzml = ms.IndexedMzMLFileLoader()
indexed_mzml.load("large_file.mzML")

# 通过索引获取特定谱图
spec = indexed_mzml.getSpectrumById(100)

# 通过原生ID访问
spec = indexed_mzml.getSpectrumByNativeId("scan=5000")
```

### 流式访问

超大文件的内存高效处理：

```python
# 定义消费者函数
class SpectrumProcessor(ms.MSExperimentConsumer):
    def __init__(self):
        super().__init__()
        self.count = 0

    def consumeSpectrum(self, spec):
        # 处理谱图
        if spec.getMSLevel() == 2:
            self.count += 1

# 流式处理文件
consumer = SpectrumProcessor()
ms.MzMLFile().transform("large.mzML", consumer)
print(f"已处理 {consumer.count} 个MS2谱图")
```

### 缓存访问

内存与速度的平衡方案：

```python
# 使用磁盘缓存
options = ms.CachedmzML()
options.setMetaDataOnly(False)

exp = ms.MSExperiment()
ms.CachedmzMLHandler().load("sample.mzML", exp, options)
```

## 写入mzML文件

### 基础写入

```python
# 创建或修改实验
exp = ms.MSExperiment()
# ... 添加谱图 ...

# 写入文件
ms.MzMLFile().store("output.mzML", exp)
```

### 压缩选项

```python
# 配置压缩
file_handler = ms.MzMLFile()

options = ms.PeakFileOptions()
options.setCompression(True)  # 启用压缩
file_handler.setOptions(options)

file_handler.store("compressed.mzML", exp)
```

## 读取鉴定数据

### idXML格式

```python
# 加载鉴定结果
protein_ids = []
peptide_ids = []

ms.IdXMLFile().load("identifications.idXML", protein_ids, peptide_ids)

# 访问多肽鉴定
for peptide_id in peptide_ids:
    print(f"保留时间: {peptide_id.getRT()}")
    print(f"质荷比: {peptide_id.getMZ()}")

    # 获取多肽匹配
    for hit in peptide_id.getHits():
        print(f"  序列: {hit.getSequence().toString()}")
        print(f"  得分: {hit.getScore()}")
        print(f"  电荷: {hit.getCharge()}")
```

### mzIdentML格式

```python
# 读取mzIdentML
protein_ids = []
peptide_ids = []

ms.MzIdentMLFile().load("results.mzid", protein_ids, peptide_ids)
```

### pepXML格式

```python
# 加载pepXML
protein_ids = []
peptide_ids = []

ms.PepXMLFile().load("results.pep.xml", protein_ids, peptide_ids)
```

## 读取特征数据

### featureXML

```python
# 加载特征
feature_map = ms.FeatureMap()
ms.FeatureXMLFile().load("features.featureXML", feature_map)

# 访问特征
for feature in feature_map:
    print(f"保留时间: {feature.getRT()}")
    print(f"质荷比: {feature.getMZ()}")
    print(f"强度: {feature.getIntensity()}")
    print(f"质量: {feature.getOverallQuality()}")
```

### consensusXML

```python
# 加载一致性特征
consensus_map = ms.ConsensusMap()
ms.ConsensusXMLFile().load("consensus.consensusXML", consensus_map)

# 访问一致性特征
for consensus_feature in consensus_map:
    print(f"保留时间: {consensus_feature.getRT()}")
    print(f"质荷比: {consensus_feature.getMZ()}")

    # 获取特征句柄（来自不同图谱的子特征）
    for handle in consensus_feature.getFeatureList():
        map_index = handle.getMapIndex()
        intensity = handle.getIntensity()
        print(f"  图谱 {map_index}: {intensity}")
```

## 读取FASTA文件

```python
# 加载蛋白质序列
fasta_entries = []
ms.FASTAFile().load("database.fasta", fasta_entries)

for entry in fasta_entries:
    print(f"标识符: {entry.identifier}")
    print(f"描述: {entry.description}")
    print(f"序列: {entry.sequence}")
```

## 读取TraML文件

```python
# 加载靶向实验的转换列表
targeted_exp = ms.TargetedExperiment()
ms.TraMLFile().load("transitions.TraML", targeted_exp)

# 访问转换
for transition in targeted_exp.getTransitions():
    print(f"前体质荷比: {transition.getPrecursorMZ()}")
    print(f"产物质荷比: {transition.getProductMZ()}")
```

## 写入mzTab文件

```python
# 创建报告用mzTab
mztab = ms.MzTab()

# 添加元数据
metadata = mztab.getMetaData()
metadata.mz_tab_version.set("1.0.0")
metadata.title.set("蛋白质组学分析结果")

# 添加蛋白质数据
protein_section = mztab.getProteinSectionRows()
# ... 填充蛋白质数据 ...

# 写入文件
ms.MzTabFile().store("report.mzTab", mztab)
```

## 格式转换

### mzXML转mzML

```python
# 读取mzXML
exp = ms.MSExperiment()
ms.MzXMLFile().load("data.mzXML", exp)

# 写入为mzML
ms.MzMLFile().store("data.mzML", exp)
```

### 从mzML提取色谱图

```python
# 加载实验
exp = ms.MSExperiment()
ms.MzMLFile().load("data.mzML", exp)

# 提取特定色谱图
for chrom in exp.getChromatograms():
    if chrom.getNativeID() == "TIC":
        rt, intensity = chrom.get_peaks()
        print(f"TIC包含 {len(rt)} 个数据点")
```

## 文件元数据

### 访问mzML元数据

```python
# 加载文件
exp = ms.MSExperiment()
ms.MzMLFile().load("sample.mzML", exp)

# 获取实验设置
exp_settings = exp.getExperimentalSettings()

# 仪器信息
instrument = exp_settings.getInstrument()
print(f"仪器: {instrument.getName()}")
print(f"型号: {instrument.getModel()}")

# 样品信息
sample = exp_settings.getSample()
print(f"样品名称: {sample.getName()}")

# 源文件
for source_file in exp_settings.getSourceFiles():
    print(f"源文件: {source_file.getNameOfFile()}")
```

## 最佳实践

### 内存管理

处理大文件时：
1. 使用索引或流式访问替代全内存加载
2. 分块处理数据
3. 及时清理不再使用的数据结构

```python
# 适用于大文件
indexed_mzml = ms.IndexedMzMLFileLoader()
indexed_mzml.load("huge_file.mzML")

# 逐个处理谱图
for i in range(indexed_mzml.getNrSpectra()):
    spec = indexed_mzml.getSpectrumById(i)
    # 处理谱图
    # 处理完成后谱图自动清理
```

### 错误处理

```python
try:
    exp = ms.MSExperiment()
    ms.MzMLFile().load("data.mzML", exp)
except Exception as e:
    print(f"文件加载失败: {e}")
```

### 文件验证

```python
# 检查文件是否存在且可读
import os

if os.path.exists("data.mzML") and os.path.isfile("data.mzML"):
    exp = ms.MSExperiment()
    ms.MzMLFile().load("data.mzML", exp)
else:
    print("未找到文件")
```
