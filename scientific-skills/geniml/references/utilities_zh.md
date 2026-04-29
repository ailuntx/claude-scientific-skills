# Geniml实用工具与附加工具

## BBClient：BED文件缓存

### 概述
BBClient提供对远程来源BED文件的高效缓存，实现更快的重复访问以及与R工作流的集成。

### 使用场景
在以下情况使用BBClient：
- 需要重复访问远程数据库中的BED文件
- 使用BEDbase存储库时
- 将基因组数据集成到R分析流程中
- 需要本地缓存提升性能时

### Python用法
```python
from geniml.bbclient import BBClient

# 初始化客户端
client = BBClient(cache_folder='~/.bedcache')

# 获取并缓存BED文件
bed_file = client.load_bed(bed_id='GSM123456')

# 访问缓存文件
regions = client.get_regions('GSM123456')
```

### R集成
```r
library(reticulate)
geniml <- import("geniml.bbclient")

# 初始化客户端
client <- geniml$BBClient(cache_folder='~/.bedcache')

# 加载BED文件
bed_file <- client$load_bed(bed_id='GSM123456')
```

### 最佳实践
- 配置具有充足存储空间的缓存目录
- 跨分析使用一致的缓存位置
- 定期清理缓存以移除未使用文件

---

## BEDshift：BED文件随机化

### 概述
BEDshift提供在保留基因组上下文的同时随机化BED文件的工具，这对于生成零分布和统计检验至关重要。

### 使用场景
在以下情况使用BEDshift：
- 创建统计检验的零模型
- 生成对照数据集
- 评估基因组重叠的显著性
- 基准测试分析方法时

### 用法
```python
from geniml.bedshift import bedshift

# 随机化BED文件并保留染色体分布
randomized = bedshift(
    input_bed='peaks.bed',
    genome='hg38',
    preserve_chrom=True,
    n_iterations=100
)
```

### 命令行用法
```bash
geniml bedshift \
  --input peaks.bed \
  --genome hg38 \
  --preserve-chrom \
  --iterations 100 \
  --output randomized_peaks.bed
```

### 随机化策略

**保留染色体分布：**
```python
bedshift(input_bed, genome, preserve_chrom=True)
```
保持区域位于原始染色体上。

**保留距离分布：**
```python
bedshift(input_bed, genome, preserve_distance=True)
```
维持区域间距离关系。

**保留区域大小：**
```python
bedshift(input_bed, genome, preserve_size=True)
```
保持原始区域长度不变。

### 最佳实践
- 选择符合零假设的随机化策略
- 生成多次迭代以获得稳健统计
- 验证随机化输出保持所需特性
- 记录随机化参数确保可复现性

---

## 评估：模型评估工具

### 概述
Geniml提供用于评估嵌入质量和模型性能的评估工具。

### 使用场景
在以下情况使用评估工具：
- 验证训练后的嵌入
- 比较不同模型
- 评估聚类质量
- 发布模型结果时

### 嵌入评估
```python
from geniml.evaluation import evaluate_embeddings

# 评估Region2Vec嵌入
metrics = evaluate_embeddings(
    embeddings_file='region2vec_model/embeddings.npy',
    labels_file='metadata.csv',
    metrics=['silhouette', 'davies_bouldin', 'calinski_harabasz']
)

print(f"轮廓系数: {metrics['silhouette']:.3f}")
print(f"戴维斯-堡丁指数: {metrics['davies_bouldin']:.3f}")
```

### 聚类指标

**轮廓系数：** 衡量聚类内聚性和分离度（-1到1，值越高越好）

**戴维斯-堡丁指数：** 聚类间平均相似度（≥0，值越低越好）

**卡林斯基-哈拉巴斯分数：** 聚类间/内离散度比率（值越高越好）

### scEmbed细胞类型注释评估
```python
from geniml.evaluation import evaluate_annotation

# 评估细胞类型预测
results = evaluate_annotation(
    predicted=adata.obs['predicted_celltype'],
    true=adata.obs['true_celltype'],
    metrics=['accuracy', 'f1', 'confusion_matrix']
)

print(f"准确率: {results['accuracy']:.1%}")
print(f"F1分数: {results['f1']:.3f}")
```

### 最佳实践
- 使用多个互补指标
- 与基线模型比较
- 在保留测试数据上报告指标
- 结合指标可视化嵌入（UMAP/t-SNE）

---

## 标记化：区域标记化工具

### 概述
标记化使用参考宇宙将基因组区域转换为离散标记，实现word2vec风格的训练。

### 使用场景
标记化是以下场景的必要预处理步骤：
- Region2Vec训练
- scEmbed模型训练
- 任何需要离散标记的嵌入方法

### 硬标记化
基于严格重叠的标记化：
```python
from geniml.tokenization import hard_tokenization

hard_tokenization(
    src_folder='bed_files/',
    dst_folder='tokenized/',
    universe_file='universe.bed',
    p_value_threshold=1e-9
)
```

**参数：**
- `p_value_threshold`：重叠显著性水平（通常为1e-9或1e-6）

### 软标记化
允许部分匹配的概率标记化：
```python
from geniml.tokenization import soft_tokenization

soft_tokenization(
    src_folder='bed_files/',
    dst_folder='tokenized/',
    universe_file='universe.bed',
    overlap_threshold=0.5
)
```

**参数：**
- `overlap_threshold`：最小重叠比例（0-1）

### 基于宇宙的标记化
使用自定义参数将区域映射到宇宙标记：
```python
from geniml.tokenization import universe_tokenization

universe_tokenization(
    bed_file='peaks.bed',
    universe_file='universe.bed',
    output_file='tokens.txt',
    method='hard',
    threshold=1e-9
)
```

### 最佳实践
- **宇宙质量**：使用全面构建的优质宇宙
- **阈值选择**：更高置信度需更严格阈值（更低p值）
- **验证**：检查标记化覆盖率（区域被标记化的百分比）
- **一致性**：相关分析使用相同宇宙和参数

### 标记化覆盖率
检查区域标记化效果：
```python
from geniml.tokenization import check_coverage

coverage = check_coverage(
    bed_file='peaks.bed',
    universe_file='universe.bed',
    threshold=1e-9
)

print(f"标记化覆盖率: {coverage:.1%}")
```
可靠训练需>80%覆盖率。

---

## Text2BedNN：搜索后端

### 概述
Text2BedNN创建基于神经网络的搜索后端，支持使用自然语言或元数据查询基因组区域。

### 使用场景
在以下情况使用Text2BedNN：
- 构建基因组数据库搜索接口
- 支持对BED文件的自然语言查询
- 创建元数据感知搜索系统
- 部署交互式基因组搜索应用

### 工作流程

**步骤1：准备嵌入**
使用元数据训练BEDspace或Region2Vec模型。

**步骤2：构建搜索索引**
```python
from geniml.search import build_search_index

build_search_index(
    embeddings_file='bedspace_model/embeddings.npy',
    metadata_file='metadata.csv',
    output_dir='search_backend/'
)
```

**步骤3：查询索引**
```python
from geniml.search import SearchBackend

backend = SearchBackend.load('search_backend/')

# 自然语言查询
results = backend.query(
    text="T细胞调控区域",
    top_k=10
)

# 元数据查询
results = backend.query(
    metadata={'cell_type': 'T_cell', 'tissue': 'blood'},
    top_k=10
)
```

### 最佳实践
- 使用丰富元数据训练嵌入以提升搜索质量
- 索引大型集合确保全面覆盖
- 在已知查询上验证搜索相关性
- 通过API部署实现交互式应用

---

## 附加工具

### I/O工具
```python
from geniml.io import read_bed, write_bed, load_universe

# 读取BED文件
regions = read_bed('peaks.bed')

# 写入BED文件
write_bed(regions, 'output.bed')

# 加载宇宙
universe = load_universe('universe.bed')
```

### 模型工具
```python
from geniml.models import save_model, load_model

# 保存训练好的模型
save_model(model, 'my_model/')

# 加载模型
model = load_model('my_model/')
```

### 通用模式

**流程工作流：**
```python
# 1. 构建宇宙
universe = build_universe(coverage_folder='coverage/', method='cc', cutoff=5)

# 2. 标记化
hard_tokenization(src_folder='beds/', dst_folder='tokens/',
                   universe_file='universe.bed', p_value_threshold=1e-9)

# 3. 训练嵌入
region2vec(token_folder='tokens/', save_dir='model/', num_shufflings=1000)

# 4. 评估
metrics = evaluate_embeddings(embeddings_file='model/embeddings.npy',
                               labels_file='metadata.csv')
```

这种模块化设计允许灵活组合geniml工具，满足多样化的基因组机器学习工作流需求。
