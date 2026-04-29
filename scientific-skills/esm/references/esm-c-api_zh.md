# ESM C API 参考

## 概述

ESM C（寒武纪）是一系列专为表征学习和高效嵌入生成优化的蛋白质语言模型。作为 ESM2 的直接替代品，ESM C 在所有模型尺寸上均实现了速度和质量的显著提升。

## 模型架构

**ESM C 系列模型：**

| 模型 ID | 参数量 | 层数 | 最佳适用场景 |
|----------|-----------|--------|----------|
| `esmc-300m` | 300M | 30 | 快速推理，轻量级应用 |
| `esmc-600m` | 600M | 36 | 性能与质量平衡 |
| `esmc-6b` | 6B | 80 | 最高表征质量 |

**核心特性：**
- 推理速度比 ESM2 快 3 倍
- 改进的困惑度与嵌入质量
- 面向生产部署的高效架构
- 兼容 ESM2 工作流（直接替代）
- 支持长序列（最高效处理 1024 个残基）

**相对 ESM2 的架构改进：**
- 优化的注意力机制
- 更优的标记表征
- 增强的训练流程
- 减少的内存占用

## 核心 API 组件

### ESMC 类

ESM C 模型的主接口。

**模型加载：**

```python
from esm.models.esmc import ESMC
from esm.sdk.api import ESMProtein

# 自动设备分配加载模型
model = ESMC.from_pretrained("esmc-300m").to("cuda")

# 或显式指定设备
model = ESMC.from_pretrained("esmc-600m").to("cpu")

# 追求最高质量
model = ESMC.from_pretrained("esmc-6b").to("cuda")
```

**模型选择标准：**

- **esmc-300m**：开发环境、实时应用、多序列批量处理
- **esmc-600m**：生产部署、优质性能平衡
- **esmc-6b**：科研场景、下游任务最高精度

### 基础嵌入生成

**单序列处理：**

```python
from esm.models.esmc import ESMC
from esm.sdk.api import ESMProtein

# 加载模型
model = ESMC.from_pretrained("esmc-600m").to("cuda")

# 创建蛋白质对象
protein = ESMProtein(sequence="MPRTKEINDAGLIVHSPQWFYK")

# 编码为张量
protein_tensor = model.encode(protein)

# 生成嵌入
embeddings = model.forward(protein_tensor)

# 获取 logits（逐位置预测）
logits = model.logits(embeddings)

print(f"嵌入维度: {embeddings.shape}")
print(f"Logits 维度: {logits.shape}")
```

**输出维度：**

对于长度为 L 的序列：
- `embeddings.shape`：`(1, L, hidden_dim)`，其中 hidden_dim 取决于模型
  - esmc-300m: hidden_dim = 960
  - esmc-600m: hidden_dim = 1152
  - esmc-6b: hidden_dim = 2560
- `logits.shape`：`(1, L, 64)` - 氨基酸逐位置预测

### 批量处理

高效处理多序列：

```python
import torch

# 多蛋白质序列
sequences = [
    "MPRTKEINDAGLIVHSP",
    "AGKWFYLTQSNHERVPM",
    "DEIFKRNAVWGSLTPQY"
]

proteins = [ESMProtein(sequence=seq) for seq in sequences]

# 统一编码
protein_tensors = [model.encode(p) for p in proteins]

# 批量处理（等长序列）
# 变长序列需单独处理或填充
embeddings_list = []
for tensor in protein_tensors:
    embedding = model.forward(tensor)
    embeddings_list.append(embedding)

print(f"已处理 {len(embeddings_list)} 条蛋白质")
```

**变长序列高效批处理：**

```python
def batch_encode_variable_length(model, sequences, max_batch_size=32):
    """
    高效批处理变长序列。
    按相似长度分组以提升效率。
    """
    # 按长度排序
    sorted_seqs = sorted(enumerate(sequences), key=lambda x: len(x[1]))

    results = [None] * len(sequences)
    batch = []
    batch_indices = []

    for idx, seq in sorted_seqs:
        batch.append(seq)
        batch_indices.append(idx)

        # 当批次满载或长度显著变化时处理
        if (len(batch) >= max_batch_size or
            (len(batch) > 0 and abs(len(seq) - len(batch[0])) > 10)):

            # 处理当前批次
            proteins = [ESMProtein(sequence=s) for s in batch]
            embeddings = [model.forward(model.encode(p)) for p in proteins]

            # 存储结果
            for i, emb in zip(batch_indices, embeddings):
                results[i] = emb

            batch = []
            batch_indices = []

    # 处理剩余序列
    if batch:
        proteins = [ESMProtein(sequence=s) for s in batch]
        embeddings = [model.forward(model.encode(p)) for p in proteins]
        for i, emb in zip(batch_indices, embeddings):
            results[i] = emb

    return results
```

## 典型应用场景

### 1. 序列相似性分析

使用嵌入计算蛋白质相似度：

```python
import torch
import torch.nn.functional as F

def get_sequence_embedding(model, sequence):
    """获取平均池化的序列嵌入"""
    protein = ESMProtein(sequence=sequence)
    tensor = model.encode(protein)
    embedding = model.forward(tensor)

    # 序列长度平均池化
    return embedding.mean(dim=1)

# 获取嵌入
seq1_emb = get_sequence_embedding(model, "MPRTKEINDAGLIVHSP")
seq2_emb = get_sequence_embedding(model, "MPRTKEINDAGLIVHSQ")  # 相似序列
seq3_emb = get_sequence_embedding(model, "WWWWWWWWWWWWWWWWW")  # 差异序列

# 计算余弦相似度
sim_1_2 = F.cosine_similarity(seq1_emb, seq2_emb)
sim_1_3 = F.cosine_similarity(seq1_emb, seq3_emb)

print(f"相似度 (1,2): {sim_1_2.item():.4f}")
print(f"相似度 (1,3): {sim_1_3.item():.4f}")
```

### 2. 蛋白质分类

将嵌入作为分类特征：

```python
import numpy as np
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split

# 为训练集生成嵌入
def embed_dataset(model, sequences):
    embeddings = []
    for seq in sequences:
        protein = ESMProtein(sequence=seq)
        tensor = model.encode(protein)
        emb = model.forward(tensor).mean(dim=1)  # 平均池化
        embeddings.append(emb.cpu().detach().numpy().flatten())
    return np.array(embeddings)

# 示例：按功能分类蛋白质
train_sequences = [...]  # 输入序列
train_labels = [...]      # 对应标签

embeddings = embed_dataset(model, train_sequences)

# 训练分类器
X_train, X_test, y_train, y_test = train_test_split(
    embeddings, train_labels, test_size=0.2
)

classifier = LogisticRegression(max_iter=1000)
classifier.fit(X_train, y_train)

# 评估
accuracy = classifier.score(X_test, y_test)
print(f"分类准确率: {accuracy:.4f}")
```

### 3. 蛋白质聚类

基于序列相似性聚类：

```python
from sklearn.cluster import KMeans
import numpy as np

# 生成嵌入
sequences = [...]  # 蛋白质序列
embeddings = embed_dataset(model, sequences)

# 聚类
n_clusters = 5
kmeans = KMeans(n_clusters=n_clusters, random_state=42)
cluster_labels = kmeans.fit_predict(embeddings)

# 分析聚类结果
for i in range(n_clusters):
    cluster_seqs = [seq for seq, label in zip(sequences, cluster_labels) if label == i]
    print(f"聚类 {i}: {len(cluster_seqs)} 条序列")
```

### 4. 序列搜索与检索

在数据库中查找相似序列：

```python
import torch
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

def build_sequence_index(model, database_sequences):
    """构建可搜索的序列嵌入索引"""
    embeddings = []
    for seq in database_sequences:
        emb = get_sequence_embedding(model, seq)
        embeddings.append(emb.cpu().detach().numpy().flatten())
    return np.array(embeddings)

def search_similar_sequences(model, query_seq, database_embeddings,
                            database_sequences, top_k=10):
    """查找 top-k 最相似序列"""
    query_emb = get_sequence_embedding(model, query_seq)
    query_emb_np = query_emb.cpu().detach().numpy().flatten().reshape(1, -1)

    # 计算相似度
    similarities = cosine_similarity(query_emb_np, database_embeddings)[0]

    # 获取 top-k
    top_indices = np.argsort(similarities)[-top_k:][::-1]

    results = [
        (database_sequences[idx], similarities[idx])
        for idx in top_indices
    ]
    return results

# 使用示例
database_seqs = [...]  # 大型序列数据库
index = build_sequence_index(model, database_seqs)

query = "MPRTKEINDAGLIVHSP"
similar = search_similar_sequences(model, query, index, database_seqs, top_k=5)

for seq, score in similar:
    print(f"相似度: {score:.4f} - {seq[:30]}...")
```

### 5. 下游模型特征提取

将 ESM C 嵌入作为自定义神经网络输入：

```python
import torch.nn as nn

class ProteinPropertyPredictor(nn.Module):
    """示例：基于 ESM C 嵌入预测蛋白质属性"""

    def __init__(self, embedding_dim, hidden_dim, output_dim):
        super().__init__()
        self.fc1 = nn.Linear(embedding_dim, hidden_dim)
        self.fc2 = nn.Linear(hidden_dim, hidden_dim)
        self.fc3 = nn.Linear(hidden_dim, output_dim)
        self.relu = nn.ReLU()
        self.dropout = nn.Dropout(0.3)

    def forward(self, embeddings):
        # embeddings: (batch, seq_len, embedding_dim)
        # 序列平均池化
        x = embeddings.mean(dim=1)

        x = self.relu(self.fc1(x))
        x = self.dropout(x)
        x = self.relu(self.fc2(x))
        x = self.dropout(x)
        x = self.fc3(x)
        return x

# 使用 ESM C 作为冻结特征提取器
esm_model = ESMC.from_pretrained("esmc-600m").to("cuda")
esm_model.eval()  # 冻结参数

# 创建任务特定模型
predictor = ProteinPropertyPredictor(
    embedding_dim=1152,  # esmc-600m 维度
    hidden_dim=512,
    output_dim=1  # 例如稳定性评分
).to("cuda")

# 训练循环
for sequence, target in dataloader:
    protein = ESMProtein(sequence=sequence)
    with torch.no_grad():
        embeddings = esm_model.forward(esm_model.encode(protein))

    prediction = predictor(embeddings)
    loss = criterion(prediction, target)
    # ... 仅通过预测器反向传播
```

### 6. 残基级分析

提取残基级表征进行精细分析：

```python
def get_per_residue_embeddings(model, sequence):
    """获取每个残基的嵌入"""
    protein = ESMProtein(sequence=sequence)
    tensor = model.encode(protein)
    embeddings = model.forward(tensor)

    # 嵌入维度: (1, seq_len, hidden_dim)
    return embeddings.squeeze(0)  # (seq_len, hidden_dim)

# 分析特定位点
sequence = "MPRTKEINDAGLIVHSPQWFYK"
residue_embeddings = get_per_residue_embeddings(model, sequence)

# 提取第 10 位特征
position_10_features = residue_embeddings[10]
print(f"第 10 位残基 {sequence[10]} 的特征:")
print(f"维度: {position_10_features.shape}")

# 比较残基表征
pos_5 = residue_embeddings[5]
pos_15 = residue_embeddings[15]
similarity = F.cosine_similarity(pos_5, pos_15, dim=0)
print(f"残基相似度: {similarity.item():.4f}")
```

## 性能优化

### 内存管理

```python
import torch

# 使用半精度节省内存
model = ESMC.from_pretrained("esmc-600m").to("cuda").half()

# 混合精度处理
with torch.cuda.amp.autocast():
    embeddings = model.forward(model.encode(protein))

# 批次间清空缓存
torch.cuda.empty_cache()
```

### 批处理最佳实践

```python
def efficient_batch_processing(model, sequences, batch_size=32):
    """优化批次处理序列"""
    results = []

    for i in range(0, len(sequences), batch_size):
        batch = sequences[i:i + batch_size]

        # 处理批次
        batch_embeddings = []
        for seq in batch:
            protein = ESMProtein(sequence=seq)
            emb = model.forward(model.encode(protein))
            batch_embeddings.append(emb)

        results.extend(batch_embeddings)

        # 定期清空缓存
        if i % (batch_size * 10) == 0:
            torch.cuda.empty_cache()

    return results
```

### 嵌入缓存

```python
import pickle
import hashlib

def get_cache_key(sequence):
    """生成序列缓存键"""
    return hashlib.md5(sequence.encode()).hexdigest()

class EmbeddingCache:
    """蛋白质嵌入缓存系统"""

    def __init__(self, cache_file="embeddings_cache.pkl"):
        self.cache_file = cache_file
        try:
            with open(cache_file, 'rb') as f:
                self.cache = pickle.load(f)
        except FileNotFoundError:
            self.cache = {}

    def get(self, sequence):
        key = get_cache_key(sequence)
        return self.cache.get(key)

    def set(self, sequence, embedding):
        key = get_cache_key(sequence)
        self.cache[key] = embedding

    def save(self):
        with open(self.cache_file, 'wb') as f:
            pickle.dump(self.cache, f)

# 使用示例
cache = EmbeddingCache()

def get_embedding_cached(model, sequence):
    cached = cache.get(sequence)
    if cached is not None:
        return cached

    # 计算嵌入
    protein = ESMProtein(sequence=sequence)
    embedding = model.forward(model.encode(protein))
    cache.set(sequence, embedding)

    return embedding

# 保存缓存
cache.save()
```

## 与 ESM2 对比

**性能提升：**

| 指标 | ESM2-650M | ESM C-600M | 提升幅度 |
|--------|-----------|------------|-------------|
| 推理速度 | 1.0x | 3.0x | 3 倍加速 |
|

"""从模型中提取注意力权重。"""
    protein = ESMProtein(sequence=sequence)
    tensor = model.encode(protein)

    # 前向传播并输出注意力
    output = model.forward(tensor, output_attentions=True)

    return output.attentions  # 每层的注意力张量列表

# 可视化注意力
attentions = get_attention_weights(model, "MPRTKEINDAGLIVHSP")
# 处理并可视化注意力模式
```

## 引用

如果在研究中使用 ESM C，请引用：

```
ESM Cambrian: https://www.evolutionaryscale.ai/blog/esm-cambrian
EvolutionaryScale (2024)
```

## 附加资源

- ESM C 博客文章：https://www.evolutionaryscale.ai/blog/esm-cambrian
- 模型权重：HuggingFace EvolutionaryScale 组织
- 比较基准：详见博客文章中的详细性能对比
