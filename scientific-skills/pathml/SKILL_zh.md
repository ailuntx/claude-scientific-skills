---
name: pathml
description: 功能全面的计算病理学工具包。适用于高级全切片图像（WSI）分析，包括多重免疫荧光（CODEX、Vectra）、细胞核分割、组织图构建以及病理数据上的机器学习模型训练。支持160多种切片格式。若仅需从H&E切片提取简单图块，histolab可能更简便。
license: GPL-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# PathML

## 概述

PathML 是一个用于计算病理学工作流的综合性 Python 工具包，旨在促进全切片病理图像的机器学习和图像分析。该框架提供模块化、可组合的工具，用于加载多种切片格式、预处理图像、构建空间图、训练深度学习模型，以及分析来自 CODEX 和多重免疫荧光等多参数成像技术的数据。

## 适用场景

在以下场景应用此技能：
- 加载和处理各种专有格式的全切片图像（WSI）
- 通过染色归一化预处理 H&E 染色组织图像
- 细胞核检测、分割和分类工作流
- 构建用于空间分析的细胞与组织图
- 在病理数据上训练或部署机器学习模型（HoVer-Net、HACTNet）
- 分析多参数成像（CODEX、Vectra、MERFISH）以进行空间蛋白质组学研究
- 量化多重免疫荧光的标记物表达
- 使用 HDF5 存储管理大规模病理数据集
- 基于图块的分析与拼接操作

## 核心能力

PathML 提供六大核心能力模块，详细文档见参考文件：

### 1. 图像加载与格式支持

加载160多种专有格式的全切片图像，包括 Aperio SVS、Hamamatsu NDPI、Leica SCN、Zeiss ZVI、DICOM 和 OME-TIFF。PathML 自动处理厂商特定格式，并提供统一接口访问图像金字塔、元数据和感兴趣区域。

**参见：** `references/image_loading.md` 获取支持的格式、加载策略及不同切片类型的操作方法。

### 2. 预处理流水线

通过组合图像处理、质量控制、染色归一化、组织检测和掩膜操作等转换模块，构建模块化预处理流水线。PathML 的流水线架构支持跨大规模数据集的可复现、可扩展预处理。

**关键转换模块：**
- `StainNormalizationHE` - Macenko/Vahadane 染色归一化
- `TissueDetectionHE`, `NucleusDetectionHE` - 组织/细胞核分割
- `MedianBlur`, `GaussianBlur` - 降噪处理
- `LabelArtifactTileHE` - 伪影质量控制

**参见：** `references/preprocessing.md` 获取完整转换目录、流水线构建及预处理工作流。

### 3. 图结构构建

构建表示细胞和组织层级关系的空间图。从分割对象中提取特征，创建适用于图神经网络和空间分析的图结构表示。

**参见：** `references/graphs.md` 获取图构建方法、特征提取和空间分析工作流。

### 4. 机器学习

训练和部署用于细胞核检测、分割及分类的深度学习模型。PathML 集成 PyTorch，提供预建模型（HoVer-Net、HACTNet）、定制 DataLoader 及 ONNX 推理支持。

**关键模型：**
- **HoVer-Net** - 同步实现细胞核分割与分类
- **HACTNet** - 分层细胞类型分类

**参见：** `references/machine_learning.md` 获取模型训练、评估、推理工作流及公共数据集使用方法。

### 5. 多参数成像分析

分析来自 CODEX、Vectra、MERFISH 等多重成像平台的空间蛋白质组学和基因表达数据。PathML 提供专用切片类和转换模块，用于处理多参数数据、通过 Mesmer 进行细胞分割及量化工作流。

**参见：** `references/multiparametric.md` 获取 CODEX/Vectra 工作流、细胞分割、标记物量化及 AnnData 集成方法。

### 6. 数据管理

使用 HDF5 格式高效存储和管理大规模病理数据集。PathML 在统一存储结构中处理图块、掩膜、元数据和提取特征，并针对机器学习工作流进行优化。

**参见：** `references/data_management.md` 获取 HDF5 集成、图块管理、数据集组织和批处理策略。

## 快速入门

### 安装

```bash
# 安装 PathML
uv pip install pathml

# 安装支持全部功能的可选依赖
uv pip install pathml[all]
```

### 基础工作流示例

```python
from pathml.core import SlideData
from pathml.preprocessing import Pipeline, StainNormalizationHE, TissueDetectionHE

# 加载全切片图像
wsi = SlideData.from_slide("path/to/slide.svs")

# 创建预处理流水线
pipeline = Pipeline([
    TissueDetectionHE(),
    StainNormalizationHE(target='normalize', stain_estimation_method='macenko')
])

# 运行流水线
pipeline.run(wsi)

# 访问处理后的图块
for tile in wsi.tiles:
    processed_image = tile.image
    tissue_mask = tile.masks['tissue']
```

### 典型工作流

**H&E 图像分析：**
1. 使用对应切片类加载 WSI
2. 应用组织检测与染色归一化
3. 执行细胞核检测或训练分割模型
4. 提取特征并构建空间图
5. 开展下游分析

**多参数成像（CODEX）：**
1. 通过 `CODEXSlide` 加载 CODEX 切片
2. 合并多轮通道数据
3. 使用 Mesmer 模型分割细胞
4. 量化标记物表达
5. 导出至 AnnData 进行单细胞分析

**训练机器学习模型：**
1. 使用公共病理数据准备数据集
2. 通过 PathML 数据集创建 PyTorch DataLoader
3. 训练 HoVer-Net 或定制模型
4. 在预留测试集上评估
5. 通过 ONNX 部署推理

## 详细文档索引

执行特定任务时，请参阅对应参考文件获取完整信息：

- **图像加载：** `references/image_loading.md`
- **预处理工作流：** `references/preprocessing.md`
- **空间分析：** `references/graphs.md`
- **模型训练：** `references/machine_learning.md`
- **CODEX/多重免疫荧光：** `references/multiparametric.md`
- **数据存储：** `references/data_management.md`

## 资源

本技能包含按功能模块组织的完整参考文档。每个参考文件均提供详细的 API 信息、工作流示例、最佳实践及针对特定 PathML 功能的故障排除指南。

### references/

深度解析 PathML 功能的文档文件：

- `image_loading.md` - 全切片图像格式、加载策略、切片类
- `preprocessing.md` - 完整转换目录、流水线构建、预处理工作流
- `graphs.md` - 图构建方法、特征提取、空间分析
- `machine_learning.md` - 模型架构、训练工作流、评估、推理
- `multiparametric.md` - CODEX、Vectra、多重免疫荧光分析、细胞分割、量化
- `data_management.md` - HDF5 存储、图块管理、批处理、数据集组织

执行具体计算病理学任务时，按需加载这些参考文档。
