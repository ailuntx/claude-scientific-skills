# scEmbed：单细胞嵌入生成

## 概述

scEmbed在单细胞ATAC-seq数据集上训练Region2Vec模型，生成用于聚类和分析的细胞嵌入。该框架提供无监督机器学习方法，用于在低维空间中表示和分析scATAC-seq数据。

## 适用场景

在以下场景中使用scEmbed：
- 需要聚类的单细胞ATAC-seq（scATAC-seq）数据
- 细胞类型注释任务
- 单细胞染色质可及性的降维处理
- 与scanpy工作流集成进行下游分析

## 工作流程

### 步骤1：数据准备

输入数据必须为AnnData格式，且`.var`属性需包含峰值的`chr`、`start`和`end`值。

**从原始数据开始**（barcodes.txt, peaks.bed, matrix.mtx）：

```python
import scanpy as sc
import pandas as pd
import scipy.io
import anndata

# 加载数据
barcodes = pd.read_csv('barcodes.txt', header=None, names=['barcode'])
peaks = pd.read_csv('peaks.bed', sep='\t', header=None,
                    names=['chr', 'start', 'end'])
matrix = scipy.io.mmread('matrix.mtx').tocsr()

# 创建AnnData对象
adata = anndata.AnnData(X=matrix.T, obs=barcodes, var=peaks)
adata.write('scatac_data.h5ad')
```

### 步骤2：预标记化

使用gtars工具将基因组区域转换为标记，创建可加速训练的parquet格式标记化细胞文件：

```python
from geniml.io import tokenize_cells

tokenize_cells(
    adata='scatac_data.h5ad',
    universe_file='universe.bed',
    output='tokenized_cells.parquet'
```

**预标记化优势：**
- 加速训练迭代
- 降低内存需求
- 可复用标记化数据进行多次训练

### 步骤3：模型训练

使用标记化数据训练scEmbed模型：

```python
from geniml.scembed import ScEmbed
from geniml.region2vec import Region2VecDataset

# 加载标记化数据集
dataset = Region2VecDataset('tokenized_cells.parquet')

# 初始化并训练模型
model = ScEmbed(
    embedding_dim=100,
    window_size=5,
    negative_samples=5
)

model.train(
    dataset=dataset,
    epochs=100,
    batch_size=256,
    learning_rate=0.025
)

# 保存模型
model.save('scembed_model/')
```

### 步骤4：生成细胞嵌入

使用训练好的模型生成细胞嵌入：

```python
from geniml.scembed import ScEmbed

# 加载预训练模型
model = ScEmbed.from_pretrained('scembed_model/')

# 为AnnData对象生成嵌入
embeddings = model.encode(adata)

# 添加至AnnData用于下游分析
adata.obsm['scembed_X'] = embeddings
```

### 步骤5：下游分析

与scanpy集成进行聚类和可视化：

```python
import scanpy as sc

# 使用scEmbed嵌入构建邻域图
sc.pp.neighbors(adata, use_rep='scembed_X')

# 细胞聚类
sc.tl.leiden(adata, resolution=0.5)

# 计算UMAP可视化
sc.tl.umap(adata)

# 绘制结果
sc.pl.umap(adata, color='leiden')
```

## 关键参数

### 训练参数

| 参数 | 描述 | 典型范围 |
|-----------|-------------|---------------|
| `embedding_dim` | 细胞嵌入维度 | 50 - 200 |
| `window_size` | 训练上下文窗口大小 | 3 - 10 |
| `negative_samples` | 负样本数量 | 5 - 20 |
| `epochs` | 训练轮数 | 50 - 200 |
| `batch_size` | 训练批大小 | 128 - 512 |
| `learning_rate` | 初始学习率 | 0.01 - 0.05 |

### 标记化参数

- **参考文件**：定义基因组词汇表的参考BED文件
- **重叠阈值**：峰值与参考基因组匹配的最小重叠（通常为1e-9）

## 预训练模型

Hugging Face提供常用参考数据集的预训练scEmbed模型，加载方式：

```python
from geniml.scembed import ScEmbed

# 加载预训练模型
model = ScEmbed.from_pretrained('databio/scembed-pbmc-10k')

# 生成嵌入
embeddings = model.encode(adata)
```

## 最佳实践

- **数据质量**：使用过滤后的峰值-条形码矩阵，而非原始计数
- **预标记化**：始终预标记化以提升训练效率
- **参数调优**：根据数据集大小调整`embedding_dim`和训练轮数
- **验证**：使用已知细胞类型标记验证聚类质量
- **集成**：结合scanpy进行全面的单细胞分析
- **模型共享**：将训练模型导出至Hugging Face确保可复现性

## 示例数据集

10x Genomics PBMC 10k数据集（10,000个外周血单核细胞）作为标准基准：
- 包含多种免疫细胞类型
- 具有明确特征的细胞群体
- 可从10x Genomics官网获取

## 细胞类型注释

聚类后使用参考数据集通过K近邻（KNN）进行细胞类型注释：

```python
from geniml.scembed import annotate_celltypes

# 使用参考数据集注释
annotations = annotate_celltypes(
    query_adata=adata,
    reference_adata=reference,
    embedding_key='scembed_X',
    k=10
)

adata.obs['cell_type'] = annotations
```

## 输出

scEmbed生成：
- 低维细胞嵌入（存储在`adata.obsm`中）
- 可复用的训练模型文件
- 兼容scanpy下游分析的格式
- 可选导出至Hugging Face实现共享
