# 输入/输出操作

AnnData 提供了全面的 I/O 功能，支持读写多种格式的数据。

## 原生格式

### H5AD (基于 HDF5)
AnnData 对象的推荐原生格式，提供高效存储和快速访问。

#### 写入 H5AD 文件
```python
import anndata as ad

# 写入文件
adata.write_h5ad('data.h5ad')

# 带压缩写入
adata.write_h5ad('data.h5ad', compression='gzip')

# 指定压缩级别写入 (0-9，数值越高压缩率越大)
adata.write_h5ad('data.h5ad', compression='gzip', compression_opts=9)
```

#### 读取 H5AD 文件
```python
# 完整读入内存
adata = ad.read_h5ad('data.h5ad')

# 备份模式读取（大文件惰性加载）
adata = ad.read_h5ad('data.h5ad', backed='r')  # 只读模式
adata = ad.read_h5ad('data.h5ad', backed='r+')  # 读写模式

# 备份模式支持处理大于内存的数据集
# 仅访问的数据会加载到内存
```

#### 备份模式操作
```python
# 以备份模式打开
adata = ad.read_h5ad('large_dataset.h5ad', backed='r')

# 访问元数据而不加载X矩阵
print(adata.obs.head())
print(adata.var.head())

# 子集操作创建视图
subset = adata[:100, :500]  # 视图，不加载数据

# 加载特定数据到内存
X_subset = subset.X[:]  # 加载此子集

# 将整个备份对象转为内存对象
adata_memory = adata.to_memory()
```

### Zarr
分层数组存储格式，针对云存储和并行 I/O 优化。

#### 写入 Zarr
```python
# 写入Zarr存储
adata.write_zarr('data.zarr')

# 指定分块写入（对性能至关重要）
adata.write_zarr('data.zarr', chunks=(100, 100))
```

#### 读取 Zarr
```python
# 读取Zarr存储
adata = ad.read_zarr('data.zarr')
```

#### 远程 Zarr 访问
```python
import fsspec

# 从S3访问Zarr
store = fsspec.get_mapper('s3://bucket-name/data.zarr')
adata = ad.read_zarr(store)

# 从URL访问Zarr
store = fsspec.get_mapper('https://example.com/data.zarr')
adata = ad.read_zarr(store)
```

## 替代输入格式

### CSV/TSV
```python
# 读取CSV（基因列，细胞行）
adata = ad.read_csv('data.csv')

# 自定义分隔符读取
adata = ad.read_csv('data.tsv', delimiter='\t')

# 指定首列为行名
adata = ad.read_csv('data.csv', first_column_names=True)
```

### Excel
```python
# 读取Excel文件
adata = ad.read_excel('data.xlsx')

# 读取特定工作表
adata = ad.read_excel('data.xlsx', sheet='Sheet1')
```

### Matrix Market (MTX)
基因组学中稀疏矩阵的常用格式。

```python
# 读取MTX及相关文件
# 需要：matrix.mtx, genes.tsv, barcodes.tsv
adata = ad.read_mtx('matrix.mtx')

# 自定义基因和条形码文件读取
adata = ad.read_mtx(
    'matrix.mtx',
    var_names='genes.tsv',
    obs_names='barcodes.tsv'
)

# 必要时转置（MTX通常基因在行）
adata = adata.T
```

### 10X Genomics 格式
```python
# 读取10X h5格式
adata = ad.read_10x_h5('filtered_feature_bc_matrix.h5')

# 读取10X MTX目录
adata = ad.read_10x_mtx('filtered_feature_bc_matrix/')

# 指定基因组（当存在多个时）
adata = ad.read_10x_h5('data.h5', genome='GRCh38')
```

### Loom
```python
# 读取Loom文件
adata = ad.read_loom('data.loom')

# 指定观测值和变量注释读取
adata = ad.read_loom(
    'data.loom',
    obs_names='CellID',
    var_names='Gene'
)
```

### 文本文件
```python
# 读取通用文本文件
adata = ad.read_text('data.txt', delimiter='\t')

# 自定义参数读取
adata = ad.read_text(
    'data.txt',
    delimiter=',',
    first_column_names=True,
    dtype='float32'
)
```

### UMI 工具
```python
# 读取UMI工具格式
adata = ad.read_umi_tools('counts.tsv')
```

### HDF5 (通用)
```python
# 从HDF5文件读取（非h5ad格式）
adata = ad.read_hdf('data.h5', key='dataset')
```

## 替代输出格式

### CSV
```python
# 写入CSV文件（生成多个文件）
adata.write_csvs('output_dir/')

# 生成文件：
# - output_dir/X.csv (表达矩阵)
# - output_dir/obs.csv (观测注释)
# - output_dir/var.csv (变量注释)
# - output_dir/uns.csv (非结构化注释，如适用)

# 跳过特定组件
adata.write_csvs('output_dir/', skip_data=True)  # 跳过X矩阵
```

### Loom
```python
# 写入Loom格式
adata.write_loom('output.loom')
```

## 读取特定元素

通过细粒度控制从存储中读取特定元素：

```python
from anndata import read_elem

# 仅读取观测注释
obs = read_elem('data.h5ad/obs')

# 读取特定图层
layer = read_elem('data.h5ad/layers/normalized')

# 读取非结构化数据元素
params = read_elem('data.h5ad/uns/pca_params')
```

## 写入特定元素

```python
from anndata import write_elem
import h5py

# 写入元素到现有文件
with h5py.File('data.h5ad', 'a') as f:
    write_elem(f, 'new_layer', adata.X.copy())
```

## 惰性操作

处理超大数据集时使用惰性读取避免加载完整数据集：

```python
from anndata.experimental import read_elem_lazy

# 惰性读取（返回dask数组等）
X_lazy = read_elem_lazy('large_data.h5ad/X')

# 仅在需要时计算
subset = X_lazy[:100, :100].compute()
```

## 常用 I/O 模式

### 格式间转换
```python
# MTX转H5AD
adata = ad.read_mtx('matrix.mtx').T
adata.write_h5ad('data.h5ad')

# CSV转H5AD
adata = ad.read_csv('data.csv')
adata.write_h5ad('data.h5ad')

# H5AD转Zarr
adata = ad.read_h5ad('data.h5ad')
adata.write_zarr('data.zarr')
```

### 不加载数据读取元数据
```python
# 备份模式允许在不加载X的情况下检查元数据
adata = ad.read_h5ad('large_file.h5ad', backed='r')
print(f"数据集包含 {adata.n_obs} 个观测值和 {adata.n_vars} 个变量")
print(adata.obs.columns)
print(adata.var.columns)
# X矩阵未加载到内存
```

### 追加到现有文件
```python
# 以读写模式打开
adata = ad.read_h5ad('data.h5ad', backed='r+')

# 修改元数据
adata.obs['new_column'] = values

# 变更将写入磁盘
```

### 从 URL 下载
```python
import anndata as ad

# 直接从URL读取（h5ad文件）
url = 'https://example.com/data.h5ad'
adata = ad.read_h5ad(url, backed='r')  # 流式访问

# 其他格式需先下载
import urllib.request
urllib.request.urlretrieve(url, 'local_file.h5ad')
adata = ad.read_h5ad('local_file.h5ad')
```

## 性能优化建议

### 读取
- 仅需查询的大文件使用 `backed='r'`
- 需修改元数据但不加载全部数据时使用 `backed='r+'`
- H5AD 格式通常随机访问最快
- Zarr 更适合云存储和并行访问
- 考虑压缩存储，但注意可能降低读取速度

### 写入
- 长期存储使用压缩：`compression='gzip'` 或 `compression='lzf'`
- LZF 压缩更快但压缩率低于 GZIP
- 对于 Zarr，根据访问模式调整分块大小：
  - 顺序读取使用较大分块
  - 随机访问使用较小分块
- 写入前将字符串列转为分类类型（减小文件）

### 内存管理
```python
# 字符串转分类（减小文件大小和内存占用）
adata.strings_to_categoricals()
adata.write_h5ad('data.h5ad')

# 对稀疏数据使用稀疏矩阵
from scipy.sparse import csr_matrix
if isinstance(adata.X, np.ndarray):
    density = np.count_nonzero(adata.X) / adata.X.size
    if density < 0.5:  # 零值超过50%时
        adata.X = csr_matrix(adata.X)
```

## 处理大型数据集

### 策略 1：备份模式
```python
# 处理大于内存的数据集
adata = ad.read_h5ad('100GB_file.h5ad', backed='r')

# 基于元数据筛选（快速，不加载数据）
filtered = adata[adata.obs['quality_score'] > 0.8]

# 将筛选子集加载到内存
adata_memory = filtered.to_memory()
```

### 策略 2：分块处理
```python
# 分块处理数据
adata = ad.read_h5ad('large_file.h5ad', backed='r')

chunk_size = 1000
results = []

for i in range(0, adata.n_obs, chunk_size):
    chunk = adata[i:i+chunk_size, :].to_memory()
    # 处理分块
    result = process(chunk)
    results.append(result)
```

### 策略 3：使用 AnnCollection
```python
from anndata.experimental import AnnCollection

# 创建不加载数据的集合
adatas = [f'dataset_{i}.h5ad' for i in range(10)]
collection = AnnCollection(
    adatas,
    join_obs='inner',
    join_vars='inner'
)

# 惰性处理集合
# 仅在访问时加载数据
```

## 常见问题与解决方案

### 问题：读取时内存不足
**解决方案**：使用备份模式或分块读取
```python
adata = ad.read_h5ad('file.h5ad', backed='r')
```

### 问题：云存储读取缓慢
**解决方案**：使用 Zarr 格式并合理分块
```python
adata.write_zarr('data.zarr', chunks=(1000, 1000))
```

### 问题：文件过大
**解决方案**：使用压缩并转为稀疏/分类类型
```python
adata.strings_to_categoricals()
from scipy.sparse import csr_matrix
adata.X = csr_matrix(adata.X)
adata.write_h5ad('compressed.h5ad', compression='gzip')
```

### 问题：无法修改备份对象
**解决方案**：加载到内存或使用 'r+' 模式打开
```python
# 方案1：加载到内存
adata = adata.to_memory()

# 方案2：以读写模式打开
adata = ad.read_h5ad('file.h5ad', backed='r+')
```
