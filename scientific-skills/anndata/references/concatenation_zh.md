### 翻译要求：
- 保留原始Markdown结构
- 不添加额外解释或注释
- 不翻译URL、文件路径、原始代码
- 翻译代码块内的自然语言注释
- 技术术语或专有名词在翻译可能造成歧义时保留英文

### 翻译策略：
1. **标题与正文**：技术文档采用直译为主，保持术语一致性
   - "Concatenating" 译为"拼接"（生物信息学常用译法）
   - "AnnData" 保留不译（专有名词）
   - "observations/variables" 译为"观测/变量轴"（领域标准译法）

2. **代码注释**：
   - 自然语言注释完整翻译（如"Create multiple AnnData objects"→"创建多个AnnData对象"）
   - 技术术语保留（如"sparse matrices"→"稀疏矩阵"）

3. **技术术语处理**：
   - "inner join"→"内连接"（数据库标准译法）
   - "batch labels"→"批次标签"（生物信息学术语）
   - "block diagonal matrix"→"块对角矩阵"（数学术语）

4. **被动语态转换**：
   - "missing values are filled"→"缺失值被填充"改为主动态"填充缺失值"

5. **长句拆分**：
   - 复合句拆分为短句（如"Control how metadata..."→"通过`merge`参数控制不同对象的元数据如何合并"）

6. **UI元素保留**：
   - Markdown语法符号（#、```）完全保留
   - 代码中的字符串模板（f'Gene_{i}'）保持原格式

### 特别注意：
- "obsm/varm"等AnnData专有字段名不翻译
- "scanpy"等库名保留英文
- 函数参数（如`axis=0`）保持原样
- 数字和标点符号使用全角格式（中文排版规范）
</think>
# 拼接AnnData对象

沿观测轴或变量轴组合多个AnnData对象。

## 基础拼接

### 沿观测轴拼接（堆叠细胞/样本）
```python
import anndata as ad
import numpy as np

# 创建多个AnnData对象
adata1 = ad.AnnData(X=np.random.rand(100, 50))
adata2 = ad.AnnData(X=np.random.rand(150, 50))
adata3 = ad.AnnData(X=np.random.rand(200, 50))

# 沿观测轴拼接（axis=0，默认）
adata_combined = ad.concat([adata1, adata2, adata3], axis=0)

print(adata_combined.shape)  # (450, 50)
```

### 沿变量轴拼接（堆叠基因/特征）
```python
# 创建具有相同观测值、不同变量的对象
adata1 = ad.AnnData(X=np.random.rand(100, 50))
adata2 = ad.AnnData(X=np.random.rand(100, 30))
adata3 = ad.AnnData(X=np.random.rand(100, 70))

# 沿变量轴拼接（axis=1）
adata_combined = ad.concat([adata1, adata2, adata3], axis=1)

print(adata_combined.shape)  # (100, 150)
```

## 连接类型

### 内连接（交集）
仅保留所有对象共有的变量/观测值。

```python
import pandas as pd

# 创建具有不同变量的对象
adata1 = ad.AnnData(
    X=np.random.rand(100, 50),
    var=pd.DataFrame(index=[f'Gene_{i}' for i in range(50)])
)
adata2 = ad.AnnData(
    X=np.random.rand(150, 60),
    var=pd.DataFrame(index=[f'Gene_{i}' for i in range(10, 70)])
)

# 内连接：仅保留10-49号基因（交集）
adata_inner = ad.concat([adata1, adata2], join='inner')
print(adata_inner.n_vars)  # 40个基因（交集）
```

### 外连接（并集）
保留所有变量/观测值，填充缺失值。

```python
# 外连接：保留所有基因
adata_outer = ad.concat([adata1, adata2], join='outer')
print(adata_outer.n_vars)  # 70个基因（并集）

# 缺失值使用适当默认值填充：
# - 稀疏矩阵填充0
# - 密集矩阵填充NaN
```

### 外连接的填充值
```python
# 指定缺失数据的填充值
adata_filled = ad.concat([adata1, adata2], join='outer', fill_value=0)
```

## 追踪数据来源

### 添加批次标签
```python
# 标记每个观测值来自哪个对象
adata_combined = ad.concat(
    [adata1, adata2, adata3],
    label='batch',  # 标签列名
    keys=['batch1', 'batch2', 'batch3']  # 各对象的标签
)

print(adata_combined.obs['batch'].value_counts())
# batch1    100
# batch2    150
# batch3    200
```

### 自动批次标签
```python
# 未提供keys时使用整数索引
adata_combined = ad.concat(
    [adata1, adata2, adata3],
    label='dataset'
)
# dataset列包含：0, 1, 2
```

## 合并策略

通过`merge`参数控制不同对象的元数据如何合并。

### merge=None（观测值默认）
排除非拼接轴上的元数据。

```python
# 拼接观测值时，var元数据必须匹配
adata1.var['gene_type'] = 'protein_coding'
adata2.var['gene_type'] = 'protein_coding'

# 仅在所有对象相同时保留var
adata_combined = ad.concat([adata1, adata2], merge=None)
```

### merge='same'
保留所有对象完全相同的元数据。

```python
adata1.var['chromosome'] = ['chr1'] * 25 + ['chr2'] * 25
adata2.var['chromosome'] = ['chr1'] * 25 + ['chr2'] * 25
adata1.var['type'] = 'protein_coding'
adata2.var['type'] = 'lncRNA'  # 不同

# 保留'chromosome'（相同），排除'type'（不同）
adata_combined = ad.concat([adata1, adata2], merge='same')
```

### merge='unique'
保留每个键具有唯一值的元数据列。

```python
adata1.var['gene_id'] = [f'ENSG{i:05d}' for i in range(50)]
adata2.var['gene_id'] = [f'ENSG{i:05d}' for i in range(50)]

# 保留gene_id（每个键的值唯一）
adata_combined = ad.concat([adata1, adata2], merge='unique')
```

### merge='first'
采用首个包含该键的对象的值。

```python
adata1.var['description'] = ['Desc1'] * 50
adata2.var['description'] = ['Desc2'] * 50

# 使用adata1的描述
adata_combined = ad.concat([adata1, adata2], merge='first')
```

### merge='only'
保留仅出现在单个对象中的元数据。

```python
adata1.var['adata1_specific'] = [1] * 50
adata2.var['adata2_specific'] = [2] * 50

# 保留两个元数据列
adata_combined = ad.concat([adata1, adata2], merge='only')
```

## 处理索引冲突

### 使索引唯一
```python
import pandas as pd

# 创建具有重叠观测名称的对象
adata1 = ad.AnnData(
    X=np.random.rand(3, 10),
    obs=pd.DataFrame(index=['cell_1', 'cell_2', 'cell_3'])
)
adata2 = ad.AnnData(
    X=np.random.rand(3, 10),
    obs=pd.DataFrame(index=['cell_1', 'cell_2', 'cell_3'])
)

# 通过追加批次键使索引唯一
adata_combined = ad.concat(
    [adata1, adata2],
    label='batch',
    keys=['batch1', 'batch2'],
    index_unique='_'  # 索引唯一化分隔符
)

print(adata_combined.obs_names)
# ['cell_1_batch1', 'cell_2_batch1', 'cell_3_batch1',
#  'cell_1_batch2', 'cell_2_batch2', 'cell_3_batch2']
```

## 拼接图层

```python
# 包含图层的对象
adata1 = ad.AnnData(X=np.random.rand(100, 50))
adata1.layers['normalized'] = np.random.rand(100, 50)
adata1.layers['scaled'] = np.random.rand(100, 50)

adata2 = ad.AnnData(X=np.random.rand(150, 50))
adata2.layers['normalized'] = np.random.rand(150, 50)
adata2.layers['scaled'] = np.random.rand(150, 50)

# 若所有对象均包含图层则自动拼接
adata_combined = ad.concat([adata1, adata2])

print(adata_combined.layers.keys())
# dict_keys(['normalized', 'scaled'])
```

## 拼接多维注释

### obsm/varm
```python
# 包含嵌入的对象
adata1.obsm['X_pca'] = np.random.rand(100, 50)
adata2.obsm['X_pca'] = np.random.rand(150, 50)

# obsm沿观测轴拼接
adata_combined = ad.concat([adata1, adata2])
print(adata_combined.obsm['X_pca'].shape)  # (250, 50)
```

### obsp/varp（成对注释）
```python
from scipy.sparse import csr_matrix

# 成对矩阵
adata1.obsp['connectivities'] = csr_matrix((100, 100))
adata2.obsp['connectivities'] = csr_matrix((150, 150))

# 默认不拼接obsp（设置pairwise=True以包含）
adata_combined = ad.concat([adata1, adata2])
# adata_combined.obsp为空

# 包含成对数据（创建块对角矩阵）
adata_combined = ad.concat([adata1, adata2], pairwise=True)
print(adata_combined.obsp['connectivities'].shape)  # (250, 250)
```

## 拼接uns（非结构化）

非结构化元数据递归合并：

```python
adata1.uns['experiment'] = {'date': '2025-01-01', 'batch': 'A'}
adata2.uns['experiment'] = {'date': '2025-01-01', 'batch': 'B'}

# 对uns使用merge='unique'
adata_combined = ad.concat([adata1, adata2], uns_merge='unique')
# 保留'date'（相同值），可能排除'batch'（不同值）
```

## 惰性拼接（AnnCollection）

处理超大数据集时，使用不加载全部数据的惰性拼接：

```python
from anndata.experimental import AnnCollection

# 从文件路径创建集合（不加载数据）
files = ['data1.h5ad', 'data2.h5ad', 'data3.h5ad']
collection = AnnCollection(
    files,
    join_obs='outer',
    join_vars='inner',
    label='dataset',
    keys=['dataset1', 'dataset2', 'dataset3']
)

# 惰性访问数据
print(collection.n_obs)  # 总观测数
print(collection.obs.head())  # 加载元数据，不加载X

# 需要时转换为常规AnnData（加载全部数据）
adata = collection.to_adata()
```

### 使用AnnCollection
```python
# 不加载数据即可子集化
subset = collection[collection.obs['cell_type'] == 'T cell']

# 遍历数据集
for adata in collection:
    print(adata.shape)

# 访问特定数据集
first_dataset = collection[0]
```

## 磁盘拼接

处理内存不足的超大数据集时，直接在磁盘拼接：

```python
from anndata.experimental import concat_on_disk

# 不加载到内存直接拼接
concat_on_disk(
    ['data1.h5ad', 'data2.h5ad', 'data3.h5ad'],
    'combined.h5ad',
    join='outer'
)

# 以备份模式加载结果
adata = ad.read_h5ad('combined.h5ad', backed='r')
```

## 常见拼接模式

### 合并技术重复
```python
# 同一样本的多次运行
replicates = [adata_run1, adata_run2, adata_run3]
adata_combined = ad.concat(
    replicates,
    label='technical_replicate',
    keys=['rep1', 'rep2', 'rep3'],
    join='inner'  # 仅保留所有运行共有的基因
)
```

### 合并实验批次
```python
# 不同实验批次
batches = [adata_batch1, adata_batch2, adata_batch3]
adata_combined = ad.concat(
    batches,
    label='batch',
    keys=['batch1', 'batch2', 'batch3'],
    join='outer'  # 保留所有基因
)

# 后续：应用批次校正
```

### 合并多模态数据
```python
# 不同测量模态（如RNA+蛋白质）
adata_rna = ad.AnnData(X=np.random.rand(100, 2000))
adata_protein = ad.AnnData(X=np.random.rand(100, 50))

# 沿变量轴拼接以合并模态
adata_multimodal = ad.concat([adata_rna, adata_protein], axis=1)

# 添加标签区分模态
adata_multimodal.var['modality'] = ['RNA'] * 2000 + ['protein'] * 50
```

## 最佳实践

1. **拼接前检查兼容性**
```python
# 验证形状兼容
print([adata.n_vars for adata in [adata1, adata2, adata3]])

# 检查变量名称匹配
print([set(adata.var_names) for adata in [adata1, adata2, adata3]])
```

2. **选用合适的连接类型**
- `inner`：需要所有样本具有相同特征时（最严格）
- `outer`：需保留所有特征时（最包容）

3. **追踪数据来源**
始终使用`label`和`keys`标记观测值来源。

4. **考虑内存使用**
- 大数据集使用`AnnCollection`或`concat_on_disk`
- 结果考虑备份模式

5. **处理批次效应**
拼接仅组合数据，不校正批次效应。拼接后需应用批次校正：
```python
# 拼接后应用批次校正
import scanpy as sc
sc.pp.combat(adata_combined, key='batch')
```

6. **验证结果**
```python
# 检查维度
print(adata_combined.shape)

# 检查批次分布
print(adata_combined.obs['batch'].value_counts())

# 验证元数据完整性
print(adata_combined.var.head())
print(adata_combined.obs.head())
```
