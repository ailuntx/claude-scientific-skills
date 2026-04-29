# BEDspace：联合区域与元数据嵌入

## 概述

BEDspace将StarSpace模型应用于基因组数据，可在共享的低维空间中同时训练区域集及其元数据标签的数值嵌入。这支持跨区域和元数据的丰富查询。

## 适用场景

在以下情况使用BEDspace：
- 处理带关联元数据的区域集（细胞类型、组织、实验条件）
- 需要元数据感知相似性的搜索任务
- 跨模态查询（例如"查找与标签X相似的区域"）
- 基因组内容与实验条件的联合分析

## 工作流程

BEDspace包含四个顺序操作：

### 1. 预处理

格式化基因组区间和元数据以适配StarSpace训练：

```bash
geniml bedspace preprocess \
  --input /路径/到/区域/ \
  --metadata labels.csv \
  --universe universe.bed \
  --labels "cell_type,tissue" \
  --output preprocessed.txt
```

**必需文件：**
- **输入目录**：包含BED文件的文件夹
- **元数据CSV**：必须包含与BED文件名匹配的`file_name`列及元数据列
- **参考文件**：用于标记化的参考BED文件
- **标签**：待使用的元数据列（逗号分隔列表）

预处理步骤会为元数据添加`__label__`前缀，并将区域转换为StarSpace兼容格式。

### 2. 训练

在预处理数据上执行StarSpace模型：

```bash
geniml bedspace train \
  --path-to-starspace /路径/到/starspace \
  --input preprocessed.txt \
  --output model/ \
  --dim 100 \
  --epochs 50 \
  --lr 0.05
```

**关键训练参数：**
- `--dim`：嵌入维度（典型值：50-200）
- `--epochs`：训练轮次（典型值：20-100）
- `--lr`：学习率（典型值：0.01-0.1）

### 3. 距离计算

计算区域集与元数据标签间的距离度量：

```bash
geniml bedspace distances \
  --input model/ \
  --metadata labels.csv \
  --universe universe.bed \
  --output distances.pkl
```

此步骤生成相似性搜索所需的距离矩阵。

### 4. 搜索

支持三种场景的相似项检索：

**区域到标签 (r2l)**：查询区域集 → 检索相似元数据标签
```bash
geniml bedspace search -t r2l -d distances.pkl -q query_regions.bed -n 10
```

**标签到区域 (l2r)**：查询元数据标签 → 检索相似区域集
```bash
geniml bedspace search -t l2r -d distances.pkl -q "T_cell" -n 10
```

**区域到区域 (r2r)**：查询区域集 → 检索相似区域集
```bash
geniml bedspace search -t r2r -d distances.pkl -q query_regions.bed -n 10
```

`-n`参数控制返回结果数量。

## Python API

```python
from geniml.bedspace import BEDSpaceModel

# 加载训练模型
model = BEDSpaceModel.load('model/')

# 查询相似项
results = model.search(
    query="T_cell",
    search_type="l2r",
    top_k=10
)
```

## 最佳实践

- **元数据结构**：确保元数据CSV包含与BED文件名精确匹配的`file_name`列（不含路径）
- **标签选择**：选择能捕获目标生物学变异的元数据列
- **参考文件一致性**：预处理、距离计算及后续分析使用相同参考文件
- **验证**：训练前预处理并检查输出格式
- **StarSpace安装**：需单独安装StarSpace（外部依赖项）

## 结果解读

搜索结果按联合嵌入空间相似度排序返回：
- **r2l**：识别查询区域的元数据特征标签
- **l2r**：查找符合元数据条件的区域集
- **r2r**：发现具有相似基因组内容的区域集

## 环境要求

BEDspace需单独安装StarSpace，下载地址：https://github.com/facebookresearch/StarSpace
