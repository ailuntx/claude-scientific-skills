# 基因组标记器

标记器将基因组区域转换为离散标记，适用于机器学习应用，特别有助于训练基因组深度学习模型。

## Python 接口

### 创建标记器

从多种来源加载标记器配置：

```python
import gtars

# 从 BED 文件
tokenizer = gtars.tokenizers.TreeTokenizer.from_bed_file("regions.bed")

# 从配置文件
tokenizer = gtars.tokenizers.TreeTokenizer.from_config("tokenizer_config.yaml")

# 从区域字符串
tokenizer = gtars.tokenizers.TreeTokenizer.from_region_string("chr1:1000-2000")
```

### 基因组区域标记化

将基因组坐标转换为标记：

```python
# 标记单个区域
token = tokenizer.tokenize("chr1", 1000, 2000)

# 标记多个区域
tokens = []
for chrom, start, end in regions:
    token = tokenizer.tokenize(chrom, start, end)
    tokens.append(token)
```

### 标记属性

访问标记信息：

```python
# 获取标记ID
token_id = token.id

# 获取基因组坐标
chrom = token.chromosome
start = token.start
end = token.end

# 获取标记元数据
metadata = token.metadata
```

## 使用案例

### 机器学习预处理

标记器对准备ML模型所需的基因组数据至关重要：

1. **序列建模**：将基因组区间转换为离散标记以供Transformer模型使用
2. **位置编码**：跨数据集创建一致的位置编码
3. **数据增强**：生成用于训练的替代标记化方案

### 与 geniml 集成

标记器模块与基因组机器学习库 geniml 无缝集成：

```python
# 为 geniml 标记区域
from gtars.tokenizers import TreeTokenizer
import geniml

tokenizer = TreeTokenizer.from_bed_file("training_regions.bed")
tokens = [tokenizer.tokenize(r.chrom, r.start, r.end) for r in regions]

# 在 geniml 模型中使用标记
model = geniml.Model(vocab_size=tokenizer.vocab_size)
```

## 配置格式

标记器配置文件支持 YAML 格式：

```yaml
# tokenizer_config.yaml
type: tree
resolution: 1000  # 以碱基对为单位的标记分辨率
chromosomes:
  - chr1
  - chr2
  - chr3
options:
  overlap_handling: merge
  gap_threshold: 100
```

## 性能注意事项

- TreeTokenizer 使用高效数据结构实现快速标记化
- 处理大型数据集时建议采用批量标记化
- 预加载标记器可减少重复操作的开销
