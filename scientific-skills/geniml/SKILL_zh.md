---
name: geniml
description: 该技能适用于处理基因组区间数据（BED文件）的机器学习任务。可用于训练区域嵌入（Region2Vec、BEDspace）、单细胞ATAC-seq分析（scEmbed）、构建共识峰（universes）或任何基于机器学习的基因组区域分析。适用于BED文件集合、scATAC-seq数据、染色质可及性数据集及基于区域的基因组特征学习。
license: BSD-2-Clause license
metadata:
    skill-author: K-Dense Inc.
---

# Geniml：基因组区间机器学习

## 概述

Geniml是一个用于在BED文件基因组区间数据上构建机器学习模型的Python包。它提供无监督方法学习基因组区域、单细胞和元数据标签的嵌入表示，支持相似性搜索、聚类及下游机器学习任务。

## 安装

使用uv安装geniml：

```bash
uv uv pip install geniml
```

安装机器学习依赖项（PyTorch等）：

```bash
uv uv pip install 'geniml[ml]'
```

从GitHub安装开发版：

```bash
uv uv pip install git+https://github.com/databio/geniml.git
```

## 核心功能

Geniml提供五大核心功能，各功能在专属参考文件中详细说明：

### 1. Region2Vec：基因组区域嵌入

使用word2vec风格学习训练基因组区域的无监督嵌入表示。

**适用场景：** BED文件降维、区域相似性分析、下游机器学习特征向量。

**工作流程：**
1. 使用参考宇宙（universe）对BED文件进行标记化
2. 在标记上训练Region2Vec模型
3. 生成区域嵌入向量

**参考：** 详细工作流、参数及示例见`references/region2vec.md`

### 2. BEDspace：联合区域与元数据嵌入

使用StarSpace训练区域集与元数据标签的共享嵌入空间。

**适用场景：** 元数据感知搜索、跨模态查询（区域→标签或标签→区域）、基因组内容与实验条件的联合分析。

**工作流程：**
1. 预处理区域和元数据
2. 训练BEDspace模型
3. 计算距离
4. 跨区域和标签执行查询

**参考：** 详细工作流、搜索类型及示例见`references/bedspace.md`

### 3. scEmbed：单细胞染色质可及性嵌入

在单细胞ATAC-seq数据上训练Region2Vec模型，生成细胞级嵌入表示。

**适用场景：** scATAC-seq聚类、细胞类型注释、单细胞降维、与scanpy工作流集成。

**工作流程：**
1. 准备包含峰坐标的AnnData对象
2. 预标记化细胞
3. 训练scEmbed模型
4. 生成细胞嵌入向量
5. 使用scanpy进行聚类和可视化

**参考：** 详细工作流、参数及示例见`references/scembed.md`

### 4. Consensus Peaks：宇宙构建

通过多种统计方法从BED文件集合构建参考峰集（宇宙）。

**适用场景：** 创建标记化参考、跨数据集标准化区域、定义具有统计严谨性的共识特征。

**工作流程：**
1. 合并BED文件
2. 生成覆盖度轨迹
3. 使用CC/CCF/ML/HMM方法构建宇宙

**方法：**
- **CC（覆盖度阈值）**：基于简单阈值
- **CCF（灵活覆盖度阈值）**：边界置信区间
- **ML（最大似然）**：位置概率建模
- **HMM（隐马尔可夫模型）**：复杂状态建模

**参考：** 方法对比、参数及示例见`references/consensus_peaks.md`

### 5. Utilities：支持工具

提供缓存、随机化、评估和搜索等辅助工具。

**可用工具：**
- **BBClient**：BED文件缓存（支持重复访问）
- **BEDshift**：保持基因组背景的随机化
- **Evaluation**：嵌入质量评估指标（轮廓系数、Davies-Bouldin指数等）
- **Tokenization**：区域标记化工具（硬标记、软标记、基于宇宙）
- **Text2BedNN**：基因组查询的神经搜索后端

**参考：** 各工具详细用法见`references/utilities.md`

## 常用工作流

### 基础区域嵌入流程

```python
from geniml.tokenization import hard_tokenization
from geniml.region2vec import region2vec
from geniml.evaluation import evaluate_embeddings

# 步骤1：标记化BED文件
hard_tokenization(
    src_folder='bed_files/',
    dst_folder='tokens/',
    universe_file='universe.bed',
    p_value_threshold=1e-9
)

# 步骤2：训练Region2Vec
region2vec(
    token_folder='tokens/',
    save_dir='model/',
    num_shufflings=1000,
    embedding_dim=100
)

# 步骤3：评估
metrics = evaluate_embeddings(
    embeddings_file='model/embeddings.npy',
    labels_file='metadata.csv'
)
```

### scATAC-seq分析流程

```python
import scanpy as sc
from geniml.scembed import ScEmbed
from geniml.io import tokenize_cells

# 步骤1：加载数据
adata = sc.read_h5ad('scatac_data.h5ad')

# 步骤2：标记化细胞
tokenize_cells(
    adata='scatac_data.h5ad',
    universe_file='universe.bed',
    output='tokens.parquet'
)

# 步骤3：训练scEmbed
model = ScEmbed(embedding_dim=100)
model.train(dataset='tokens.parquet', epochs=100)

# 步骤4：生成嵌入向量
embeddings = model.encode(adata)
adata.obsm['scembed_X'] = embeddings

# 步骤5：使用scanpy聚类
sc.pp.neighbors(adata, use_rep='scembed_X')
sc.tl.leiden(adata)
sc.tl.umap(adata)
```

### 宇宙构建与评估

```bash
# 生成覆盖度
cat bed_files/*.bed > combined.bed
uniwig -m 25 combined.bed chrom.sizes coverage/

# 使用覆盖度阈值构建宇宙
geniml universe build cc \
  --coverage-folder coverage/ \
  --output-file universe.bed \
  --cutoff 5 \
  --merge 100 \
  --filter-size 50

# 评估宇宙质量
geniml universe evaluate \
  --universe universe.bed \
  --coverage-folder coverage/ \
  --bed-folder bed_files/
```

## CLI参考

Geniml为主要操作提供命令行接口：

```bash
# Region2Vec训练
geniml region2vec --token-folder tokens/ --save-dir model/ --num-shuffle 1000

# BEDspace预处理
geniml bedspace preprocess --input regions/ --metadata labels.csv --universe universe.bed

# BEDspace训练
geniml bedspace train --input preprocessed.txt --output model/ --dim 100

# BEDspace搜索
geniml bedspace search -t r2l -d distances.pkl -q query.bed -n 10

# 宇宙构建
geniml universe build cc --coverage-folder coverage/ --output universe.bed --cutoff 5

# BEDshift随机化
geniml bedshift --input peaks.bed --genome hg38 --preserve-chrom --iterations 100
```

## 工具选择指南

**使用Region2Vec当：**
- 处理批量基因组数据（ChIP-seq、ATAC-seq等）
- 需要无元数据的无监督嵌入
- 跨实验比较区域集
- 为下游监督学习构建特征

**使用BEDspace当：**
- 存在元数据标签（细胞类型、组织、条件）
- 需按元数据查询区域或反向查询
- 需要区域与标签的联合嵌入空间
- 构建可搜索基因组数据库

**使用scEmbed当：**
- 分析单细胞ATAC-seq数据
- 按染色质可及性聚类细胞
- 从scATAC-seq注释细胞类型
- 需与scanpy集成

**使用宇宙构建当：**
- 需要标记化参考峰集
- 将多实验合并为共识集
- 需要统计严谨的区域定义
- 为项目构建标准参考

**使用Utilities当：**
- 需缓存远程BED文件（BBClient）
- 生成统计零模型（BEDshift）
- 评估嵌入质量（Evaluation）
- 构建搜索接口（Text2BedNN）

## 最佳实践

### 通用准则

- **宇宙质量至关重要**：投入时间构建全面优质的宇宙
- **标记化验证**：训练前检查覆盖度（理想值>80%）
- **参数调优**：尝试不同嵌入维度、学习率和训练周期
- **评估**：始终通过多指标和可视化验证嵌入效果
- **文档记录**：保存参数和随机种子确保可复现性

### 性能考量

- **预标记化**：scEmbed场景下始终预标记化细胞以加速训练
- **内存管理**：大数据集需分批处理或降采样
- **计算资源**：ML/HMM宇宙构建方法计算密集
- **模型缓存**：使用BBClient避免重复下载

### 集成模式

- **与scanpy**：scEmbed嵌入可无缝集成至`adata.obsm`
- **与BEDbase**：使用BBClient访问远程BED存储库
- **与Hugging Face**：导出训练模型以便共享和复现
- **与R**：通过reticulate实现R集成（见工具参考）

## 关联项目

Geniml属于BEDbase生态系统：

- **BEDbase**：基因组区域统一平台
- **BEDboss**：BED文件处理流程
- **Gtars**：基因组工具集
- **BBClient**：BEDbase存储库客户端

## 资源

- **文档**：https://docs.bedbase.org/geniml/
- **GitHub**：https://github.com/databio/geniml
- **预训练模型**：Hugging Face平台（databio组织）
- **出版物**：文档中引用方法学细节

## 故障排除

**"标记化覆盖度过低"：**
- 检查宇宙质量和完整性
- 调整p值阈值（尝试1e-6替代1e-9）
- 确保宇宙匹配基因组组装版本

**"训练未收敛"：**
- 调整学习率（尝试0.01-0.05范围）
- 增加训练周期
- 检查数据质量和预处理

**"内存溢出错误"：**
- 减小scEmbed批次大小
- 分块处理数据
- 单细胞数据使用预标记化

**"未找到StarSpace"（BEDspace）：**
- 单独安装StarSpace：https://github.com/facebookresearch/StarSpace
- 正确设置`--path-to-starspace`参数

详细故障排除及方法特定问题请查阅对应参考文件。
