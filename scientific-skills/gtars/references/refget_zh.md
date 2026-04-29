# 参考序列管理

refget 模块负责参考序列检索和摘要计算，遵循 refget 协议进行序列标识。

## RefgetStore

RefgetStore 管理参考序列及其摘要：

```python
import gtars

# 创建 RefgetStore
store = gtars.RefgetStore()

# 添加序列
store.add_sequence("chr1", sequence_data)

# 检索序列
seq = store.get_sequence("chr1")

# 获取序列摘要
digest = store.get_digest("chr1")
```

## 序列摘要

计算并验证序列摘要：

```python
# 计算序列摘要
from gtars.refget import compute_digest

digest = compute_digest(sequence_data)

# 验证摘要匹配
is_valid = store.verify_digest("chr1", expected_digest)
```

## 与参考基因组集成

处理标准参考基因组：

```python
# 加载参考基因组
store = gtars.RefgetStore.from_fasta("hg38.fa")

# 获取染色体序列
chr1 = store.get_sequence("chr1")
chr2 = store.get_sequence("chr2")

# 获取子序列
region_seq = store.get_subsequence("chr1", 1000, 2000)
```

## 命令行界面使用

通过命令行管理参考序列：

```bash
# 计算 FASTA 文件摘要
gtars refget digest --input genome.fa --output digests.txt

# 验证序列摘要
gtars refget verify --sequence sequence.fa --digest expected_digest
```

## Refget 协议合规性

refget 模块遵循 GA4GH refget 协议：

### 摘要计算

摘要使用 SHA-512 算法计算并截断至 48 字节：

```python
# 计算符合 refget 协议的摘要
digest = gtars.refget.compute_digest(sequence)
# 返回格式: "SQ.abc123..."
```

### 序列检索

通过摘要检索序列：

```python
# 通过 refget 摘要获取序列
seq = store.get_sequence_by_digest("SQ.abc123...")
```

## 使用案例

### 参考基因组验证

验证参考基因组完整性：

```python
# 计算参考基因组摘要
store = gtars.RefgetStore.from_fasta("reference.fa")
digests = {chrom: store.get_digest(chrom) for chrom in store.chromosomes}

# 与预期摘要比较
for chrom, expected in expected_digests.items():
    actual = digests[chrom]
    if actual != expected:
        print(f"{chrom} 摘要不匹配: {actual} != {expected}")
```

### 序列提取

提取特定基因组区域：

```python
# 提取目标区域
store = gtars.RefgetStore.from_fasta("hg38.fa")

regions = [
    ("chr1", 1000, 2000),
    ("chr2", 5000, 6000),
    ("chr3", 10000, 11000)
]

sequences = [store.get_subsequence(c, s, e) for c, s, e in regions]
```

### 跨参考基因组比较

比较不同参考版本间的序列：

```python
# 加载两个参考版本
hg19 = gtars.RefgetStore.from_fasta("hg19.fa")
hg38 = gtars.RefgetStore.from_fasta("hg38.fa")

# 比较摘要
for chrom in hg19.chromosomes:
    digest_19 = hg19.get_digest(chrom)
    digest_38 = hg38.get_digest(chrom)
    if digest_19 != digest_38:
        print(f"{chrom} 在 hg19 和 hg38 中存在差异")
```

## 性能说明

- 按需加载序列
- 摘要计算后缓存
- 高效子序列提取
- 支持大型基因组的内存映射文件访问
