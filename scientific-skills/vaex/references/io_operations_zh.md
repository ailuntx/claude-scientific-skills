# I/O 操作

本文档涵盖 Vaex 中的文件输入/输出操作、格式转换、导出策略以及处理各类数据格式的方法。

## 概述

Vaex 支持多种文件格式，不同格式具有不同的性能特征。格式选择会显著影响加载速度、内存占用和整体性能。

**格式推荐：**
- **HDF5** - 适用于大多数场景（即时加载，内存映射）
- **Apache Arrow** - 最佳互操作性（即时加载，列式存储）
- **Parquet** - 适用于分布式系统（压缩存储，列式存储）
- **CSV** - 避免用于大型数据集（加载缓慢，非内存映射）

## 数据读取

### HDF5 文件（推荐）

```python
import vaex

# 打开 HDF5 文件（即时内存映射）
df = vaex.open('data.hdf5')

# 多文件合并为单个 DataFrame
df = vaex.open('data_part*.hdf5')
df = vaex.open(['data_2020.hdf5', 'data_2021.hdf5', 'data_2022.hdf5'])
```

**优势：**
- 即时加载（内存映射，不读入 RAM）
- Vaex 操作性能最优
- 支持压缩
- 支持随机访问模式

### Apache Arrow 文件

```python
# 打开 Arrow 文件（即时内存映射）
df = vaex.open('data.arrow')
df = vaex.open('data.feather')  # Feather 是 Arrow 格式

# 多 Arrow 文件合并
df = vaex.open('data_*.arrow')
```

**优势：**
- 即时加载（内存映射）
- 语言无关格式
- 极佳的数据共享能力
- 与 Arrow 生态零拷贝集成

### Parquet 文件

```python
# 打开 Parquet 文件
df = vaex.open('data.parquet')

# 多 Parquet 文件合并
df = vaex.open('data_*.parquet')

# 从云存储读取
df = vaex.open('s3://bucket/data.parquet')
df = vaex.open('gs://bucket/data.parquet')
```

**优势：**
- 默认压缩
- 列式存储格式
- 广泛的生态支持
- 适合分布式系统

**注意事项：**
- 本地文件读取慢于 HDF5/Arrow
- 部分操作需读取完整文件

### CSV 文件

```python
# 简单 CSV 读取
df = vaex.from_csv('data.csv')

# 自动分块读取大型 CSV
df = vaex.from_csv('large_data.csv', chunk_size=5_000_000)

# CSV 转 HDF5
df = vaex.from_csv('large_data.csv', convert='large_data.hdf5')
# 生成 HDF5 文件供后续快速加载

# 带选项读取 CSV
df = vaex.from_csv(
    'data.csv',
    sep=',',
    header=0,
    names=['col1', 'col2', 'col3'],
    dtype={'col1': 'int64', 'col2': 'float64'},
    usecols=['col1', 'col2'],  # 仅加载指定列
    nrows=100000  # 限制行数
)
```

**建议：**
- **大型 CSV 务必转为 HDF5** 供重复使用
- 使用 `convert` 参数自动创建 HDF5
- 大型 CSV 加载耗时显著

### FITS 文件（天文数据）

```python
# 打开 FITS 文件
df = vaex.open('astronomical_data.fits')

# 多 FITS 文件合并
df = vaex.open('survey_*.fits')
```

## 数据写入/导出

### 导出为 HDF5

```python
# 导出为 HDF5（Vaex 推荐格式）
df.export_hdf5('output.hdf5')

# 带进度条导出
df.export_hdf5('output.hdf5', progress=True)

# 导出指定列子集
df[['col1', 'col2', 'col3']].export_hdf5('subset.hdf5')

# 压缩导出
df.export_hdf5('compressed.hdf5', compression='gzip')
```

### 导出为 Arrow

```python
# 导出为 Arrow 格式
df.export_arrow('output.arrow')

# 导出为 Feather（Arrow 格式）
df.export_feather('output.feather')
```

### 导出为 Parquet

```python
# 导出为 Parquet
df.export_parquet('output.parquet')

# 压缩导出
df.export_parquet('output.parquet', compression='snappy')
df.export_parquet('output.parquet', compression='gzip')
```

### 导出为 CSV

```python
# 导出为 CSV（不推荐大型数据）
df.export_csv('output.csv')

# 带选项导出
df.export_csv(
    'output.csv',
    sep=',',
    header=True,
    index=False,
    chunk_size=1_000_000
)

# 导出筛选结果
df[df.age > 25].export_csv('filtered_output.csv')
```

## 格式转换

### CSV 转 HDF5（最常用）

```python
import vaex

# 方法1：读取时自动转换
df = vaex.from_csv('large.csv', convert='large.hdf5')
# 创建 large.hdf5 并返回指向它的 DataFrame

# 方法2：显式转换
df = vaex.from_csv('large.csv')
df.export_hdf5('large.hdf5')

# 后续加载（即时完成）
df = vaex.open('large.hdf5')
```

### HDF5 转 Arrow

```python
# 加载 HDF5
df = vaex.open('data.hdf5')

# 导出为 Arrow
df.export_arrow('data.arrow')
```

### Parquet 转 HDF5

```python
# 加载 Parquet
df = vaex.open('data.parquet')

# 导出为 HDF5
df.export_hdf5('data.hdf5')
```

### 多 CSV 合并为单 HDF5

```python
import vaex
import glob

# 查找所有 CSV 文件
csv_files = glob.glob('data_*.csv')

# 加载并拼接
dfs = [vaex.from_csv(f) for f in csv_files]
df_combined = vaex.concat(dfs)

# 导出为单个 HDF5
df_combined.export_hdf5('combined_data.hdf5')
```

## 增量/分块 I/O

### 分块处理大型 CSV

```python
import vaex

# 分块处理 CSV
chunk_size = 1_000_000
output_file = 'processed.hdf5'

for i, df_chunk in enumerate(vaex.from_csv_chunked('huge.csv', chunk_size=chunk_size)):
    # 处理分块数据
    df_chunk['new_col'] = df_chunk.x + df_chunk.y

    # 追加到 HDF5
    if i == 0:
        df_chunk.export_hdf5(output_file)
    else:
        df_chunk.export_hdf5(output_file, mode='a')  # 追加模式

# 加载最终结果
df = vaex.open(output_file)
```

### 分块导出

```python
# 分块导出大型 DataFrame（CSV 适用）
chunk_size = 1_000_000

for i in range(0, len(df), chunk_size):
    df_chunk = df[i:i+chunk_size]
    mode = 'w' if i == 0 else 'a'
    df_chunk.export_csv('large_output.csv', mode=mode, header=(i == 0))
```

## Pandas 集成

### Pandas 转 Vaex

```python
import pandas as pd
import vaex

# 用 pandas 读取
pdf = pd.read_csv('data.csv')

# 转为 Vaex
df = vaex.from_pandas(pdf, copy_index=False)

# 更优方案：直接使用 Vaex
df = vaex.from_csv('data.csv')  # 推荐方式
```

### Vaex 转 Pandas

```python
# 完整转换（大型数据需谨慎！）
pdf = df.to_pandas_df()

# 转换子集
pdf = df[['col1', 'col2']].to_pandas_df()
pdf = df[:10000].to_pandas_df()  # 前 1 万行
pdf = df[df.age > 25].to_pandas_df()  # 筛选结果

# 采样探索
pdf_sample = df.sample(n=10000).to_pandas_df()
```

## Arrow 集成

### Arrow 转 Vaex

```python
import pyarrow as pa
import vaex

# 从 Arrow Table 转换
arrow_table = pa.table({
    'a': [1, 2, 3],
    'b': [4, 5, 6]
})
df = vaex.from_arrow_table(arrow_table)

# 从 Arrow 文件转换
arrow_table = pa.ipc.open_file('data.arrow').read_all()
df = vaex.from_arrow_table(arrow_table)
```

### Vaex 转 Arrow

```python
# 转为 Arrow Table
arrow_table = df.to_arrow_table()

# 写入 Arrow 文件
import pyarrow as pa
with pa.ipc.new_file('output.arrow', arrow_table.schema) as writer:
    writer.write_table(arrow_table)

# 或使用 Vaex 导出
df.export_arrow('output.arrow')
```

## 远程与云存储

### 从 S3 读取

```python
import vaex

# 从 S3 读取（需 s3fs）
df = vaex.open('s3://bucket-name/data.parquet')
df = vaex.open('s3://bucket-name/data.hdf5')

# 带凭证访问
import s3fs
fs = s3fs.S3FileSystem(key='access_key', secret='secret_key')
df = vaex.open('s3://bucket-name/data.parquet', fs=fs)
```

### 从 Google 云存储读取

```python
# 从 GCS 读取（需 gcsfs）
df = vaex.open('gs://bucket-name/data.parquet')

# 带凭证访问
import gcsfs
fs = gcsfs.GCSFileSystem(token='path/to/credentials.json')
df = vaex.open('gs://bucket-name/data.parquet', fs=fs)
```

### 从 Azure 读取

```python
# 从 Azure Blob 存储读取（需 adlfs）
df = vaex.open('az://container-name/data.parquet')
```

### 写入云存储

```python
# 导出到 S3
df.export_parquet('s3://bucket-name/output.parquet')
df.export_hdf5('s3://bucket-name/output.hdf5')

# 导出到 GCS
df.export_parquet('gs://bucket-name/output.parquet')
```

## 数据库集成

### 从 SQL 数据库读取

```python
import vaex
import pandas as pd
from sqlalchemy import create_engine

# 通过 pandas 中转读取
engine = create_engine('postgresql://user:password@host:port/database')
pdf = pd.read_sql('SELECT * FROM table', engine)
df = vaex.from_pandas(pdf)

# 大型表分块读取
chunks = []
for chunk in pd.read_sql('SELECT * FROM large_table', engine, chunksize=100000):
    chunks.append(vaex.from_pandas(chunk))
df = vaex.concat(chunks)

# 更优方案：从数据库导出 CSV/Parquet，再用 Vaex 加载
```

### 写入 SQL 数据库

```python
# 转为 pandas 后写入
pdf = df.to_pandas_df()
pdf.to_sql('table_name', engine, if_exists='replace', index=False)

# 大型数据分块写入
chunk_size = 100000
for i in range(0, len(df), chunk_size):
    chunk = df[i:i+chunk_size].to_pandas_df()
    chunk.to_sql('table_name', engine,
                 if_exists='append' if i > 0 else 'replace',
                 index=False)
```

## 内存映射文件

### 理解内存映射

```python
# HDF5 和 Arrow 文件默认内存映射
df = vaex.open('data.hdf5')  # 数据不加载到 RAM

# 按需从磁盘读取数据
mean = df.x.mean()  # 流式处理数据，内存占用极低

# 检查列是否内存映射
print(df.is_local('column_name'))  # False 表示内存映射
```

### 强制数据载入内存

```python
# 必要时将数据载入内存
df_in_memory = df.copy()
for col in df.get_column_names():
    df_in_memory[col] = df[col].values  # 在内存中实体化
```

## 文件压缩

### HDF5 压缩

```python
# 压缩导出
df.export_hdf5('compressed.hdf5', compression='gzip')
df.export_hdf5('compressed.hdf5', compression='lzf')
df.export_hdf5('compressed.hdf5', compression='blosc')

# 权衡：文件更小，I/O 稍慢
```

### Parquet 压缩

```python
# Parquet 默认压缩
df.export_parquet('data.parquet', compression='snappy')  # 快速
df.export_parquet('data.parquet', compression='gzip')    # 更高压缩率
df.export_parquet('data.parquet', compression='brotli')  # 最佳压缩率
```

## Vaex 服务端（远程数据）

### 启动 Vaex 服务

```bash
# 启动服务
vaex-server data.hdf5 --host 0.0.0.0 --port 9000
```

### 连接远程服务

```python
import vaex

# 连接远程 Vaex 服务
df = vaex.open('ws://hostname:9000/data')

# 操作透明执行
mean = df.x.mean()  # 在服务端计算
```

## 状态文件

### 保存 DataFrame 状态

```python
# 保存状态（含虚拟列、筛选器等）
df.state_write('state.json')

# 包含内容：
# - 虚拟列定义
# - 活动筛选器
# - 变量
# - 转换操作（缩放器、编码器、模型）
```

### 加载 DataFrame 状态

```python
# 加载数据
df = vaex.open('data.hdf5')

# 应用保存的状态
df.state_load('state.json')

# 所有虚拟列、筛选器和转换操作均恢复
```

## 最佳实践

### 1. 选择正确格式

```python
# 本地工作：HDF5
df.export_hdf5('data.hdf5')

# 共享/互操作：Arrow
df.export_arrow('data.arrow')

# 分布式系统：Parquet
df.export_parquet('data.parquet')

# 避免用 CSV 处理大型数据
```

### 2. CSV 单次转换

```python
# 一次性转换
df = vaex.from_csv('large.csv', convert='large.hdf5')

# 后续所有加载
df = vaex.open('large.hdf5')  # 即时完成！
```

### 3. 导出前实体化

```python
# 若 DataFrame 含大量虚拟列
df_materialized = df.materialize()
df_materialized.export_hdf5('output.hdf5')

# 提升导出及后续加载速度
```

### 4. 合理使用压缩

```python
# 归档或低频访问数据
df.export_hdf5('archived.hdf5', compression='gzip')

# 活跃工作数据（更快 I/O）
df.export_hdf5('working.hdf5')  # 无压缩
```

### 5. 长流程设置检查点

```python
# 完成高成本预处理后
df_preprocessed = preprocess(df)
df_preprocessed.export_hdf5('checkpoint_preprocessed.hdf5')

# 完成特征工程后
df_features = engineer_features(df_preprocessed)
df_features.export_hdf5('checkpoint_features.hdf5')

# 支持从检查点恢复
```

## 性能对比

### 格式加载速度测试

```python
import time
import vaex

# CSV（最慢）
start = time.time()
df_csv = vaex.from_csv('data.csv')
csv_time = time.time() - start

# HDF5（即时）
start = time.time()
df_hdf5 = vaex.open('data.hdf5')
hdf5_time = time.time() - start

# Arrow（即时）
start = time.time()
df_arrow = vaex.open('data.arrow')
arrow_time = time.time() - start

print(f"CSV: {csv_time:.2f}s")
print(f"HDF5: {hdf5_time:.4f}s")
print(f"Arrow: {arrow_time:.4f}s")
```

## 常用模式

### 模式：生产数据管道

```python
import vaex

# 从源头读取（CSV、数据库导出等）
df = vaex.from_csv('raw_data.csv')

# 处理流程
df['cleaned'] = clean(df.raw_column)
df['feature']

### 模式：压缩归档

```python
# 使用压缩归档旧数据
df_2020 = vaex.open('data_2020.hdf5')
df_2020.export_hdf5('archive_2020.hdf5', compression='gzip')

# 删除未压缩的原始文件
import os
os.remove('data_2020.hdf5')
```

### 模式：多源数据加载

```python
import vaex

# 从多源加载数据
df_csv = vaex.from_csv('data.csv')
df_hdf5 = vaex.open('data.hdf5')
df_parquet = vaex.open('data.parquet')

# 合并数据
df_all = vaex.concat([df_csv, df_hdf5, df_parquet])

# 导出统一格式
df_all.export_hdf5('unified.hdf5')
```

## 故障排除

### 问题：CSV加载过慢

```python
# 解决方案：转换为HDF5格式
df = vaex.from_csv('large.csv', convert='large.hdf5')
# 后续操作：df = vaex.open('large.hdf5')
```

### 问题：导出时内存不足

```python
# 解决方案：分块导出或先物化数据
df_materialized = df.materialize()
df_materialized.export_hdf5('output.hdf5')
```

### 问题：无法读取云端文件

```python
# 安装所需库
# pip install s3fs gcsfs adlfs

# 验证凭据
import s3fs
fs = s3fs.S3FileSystem()
fs.ls('s3://bucket-name/')
```

## 格式特性矩阵

| 特性 | HDF5 | Arrow | Parquet | CSV |
|---------|------|-------|---------|-----|
| 加载速度 | 即时 | 即时 | 快速 | 慢速 |
| 内存映射 | 是 | 是 | 否 | 否 |
| 压缩支持 | 可选 | 否 | 是 | 否 |
| 列式存储 | 是 | 是 | 是 | 否 |
| 可移植性 | 良好 | 优秀 | 优秀 | 优秀 |
| 文件大小 | 中等 | 中等 | 小 | 大 |
| 最佳适用场景 | Vaex工作流 | 互操作性 | 分布式 | 数据交换 |

## 相关资源

- 创建DataFrame：参见 `core_dataframes.md`
- 性能优化：参见 `performance.md`
- 数据处理：参见 `data_processing.md`
