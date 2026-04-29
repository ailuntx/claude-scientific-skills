# Region2Vec：基因组区域嵌入

## 概述

Region2Vec从BED文件生成基因组区域和区域集的无监督嵌入。该方法将基因组区域映射到词汇表，通过连接创建句子，并应用word2vec训练来学习有意义的表示。

## 适用场景

在以下场景中使用Region2Vec：
- 需要降维处理的BED文件集合
- 基因组区域相似性分析
- 需要区域特征向量的下游机器学习任务
- 跨多组基因组数据集的比较分析

## 工作流程

### 步骤1：准备数据

在源文件夹中收集BED文件。可选指定文件列表（默认使用目录内所有文件）。准备universe文件作为标记化的参考词汇表。

### 步骤2：标记化

运行硬标记化将基因组区域转换为标记：

```python
from geniml.tokenization import hard_tokenization

src_folder = '/path/to/raw/bed/files'
dst_folder = '/path/to/tokenized_files'
universe_file = '/path/to/universe_file.bed'

hard_tokenization(src_folder, dst_folder, universe_file, 1e-9)
```

最后一个参数（1e-9）是标记化重叠显著性的p值阈值。

### 步骤3：训练Region2Vec模型

在标记化文件上执行Region2Vec训练：

```python
from geniml.region2vec import region2vec

region2vec(
    token_folder=dst_folder,
    save_dir='./region2vec_model',
    num_shufflings=1000,
    embedding_dim=100,
    context_len=50,
    window_size=5,
    init_lr=0.025
)
```

## 关键参数

| 参数 | 描述 | 典型范围 |
|-----------|-------------|---------------|
| `init_lr` | 初始学习率 | 0.01 - 0.05 |
| `window_size` | 上下文窗口大小 | 3 - 10 |
| `num_shufflings` | 混洗迭代次数 | 500 - 2000 |
| `embedding_dim` | 输出嵌入维度 | 50 - 300 |
| `context_len` | 训练上下文长度 | 30 - 100 |

## 命令行使用

```bash
geniml region2vec --token-folder /path/to/tokens \
  --save-dir ./region2vec_model \
  --num-shuffle 1000 \
  --embed-dim 100 \
  --context-len 50 \
  --window-size 5 \
  --init-lr 0.025
```

## 最佳实践

- **参数调优**：针对特定数据集频繁调整 `init_lr`、`window_size`、`num_shufflings` 和 `embedding_dim` 以获得最佳性能
- **Universe文件**：使用涵盖分析中所有关注区域的全面universe文件
- **验证**：在继续训练前始终验证标记化输出
- **资源管理**：训练可能计算密集；处理大型数据集时监控内存使用

## 输出

训练后的模型保存的嵌入可用于：
- 跨基因组区域的相似性搜索
- 区域集聚类
- 下游机器学习任务的特征向量
- 通过降维技术（t-SNE、UMAP）进行可视化
