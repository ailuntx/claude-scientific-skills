# 数据管理与存储

## 概述

PathML 通过 HDF5 存储、切片管理策略和优化的批处理工作流，为处理大规模病理数据集提供高效的数据管理解决方案。该框架支持以机器学习流程和下游分析优化的格式无缝存储和检索图像、掩码、特征及元数据。

## HDF5 集成

HDF5（分层数据格式）是 PathML 处理数据的主要存储格式，具备以下特性：
- 高效压缩与分块存储
- 快速随机访问数据子集
- 支持任意规模数据集
- 异构数据类型的层级组织
- 跨平台兼容性

### 保存至 HDF5

**单张切片：**
```python
from pathml.core import SlideData

# 加载并处理切片
wsi = SlideData.from_slide("slide.svs")
wsi.generate_tiles(level=1, tile_size=256, stride=256)

# 运行预处理流程
pipeline.run(wsi)

# 保存为 HDF5
wsi.to_hdf5("processed_slide.h5")
```

**多张切片（SlideDataset）：**
```python
from pathml.core import SlideDataset
import glob

# 创建数据集
slide_paths = glob.glob("data/*.svs")
dataset = SlideDataset(slide_paths, tile_size=256, stride=256, level=1)

# 处理
dataset.run(pipeline, distributed=True, n_workers=8)

# 保存完整数据集
dataset.to_hdf5("processed_dataset.h5")
```

### HDF5 文件结构

PathML HDF5 文件采用层级组织：

```
processed_dataset.h5
├── slide_0/
│   ├── metadata/
│   │   ├── name
│   │   ├── level
│   │   ├── dimensions
│   │   └── ...
│   ├── tiles/
│   │   ├── tile_0/
│   │   │   ├── image  (H, W, C) 数组
│   │   │   ├── coords  (x, y) 坐标
│   │   │   └── masks/
│   │   │       ├── tissue 组织
│   │   │       ├── nucleus 细胞核
│   │   │       └── ...
│   │   ├── tile_1/
│   │   └── ...
│   └── features/
│       ├── tile_features  (n_tiles, n_features) 切片特征
│       └── feature_names 特征名称
├── slide_1/
└── ...
```

### 从 HDF5 加载

**加载完整切片：**
```python
from pathml.core import SlideData

# 从 HDF5 加载
wsi = SlideData.from_hdf5("processed_slide.h5")

# 访问切片
for tile in wsi.tiles:
    image = tile.image
    masks = tile.masks
    # 处理切片...
```

**加载特定切片：**
```python
# 仅加载指定索引的切片
tile_indices = [0, 10, 20, 30]
tiles = wsi.load_tiles_from_hdf5("processed_slide.h5", indices=tile_indices)

for tile in tiles:
    # 处理子集...
    pass
```

**内存映射访问：**
```python
import h5py

# 不加载到内存直接打开 HDF5 文件
with h5py.File("processed_dataset.h5", 'r') as f:
    # 访问特定数据
    tile_0_image = f['slide_0/tiles/tile_0/image'][:]
    tissue_mask = f['slide_0/tiles/tile_0/masks/tissue'][:]

    # 高效遍历切片
    for tile_key in f['slide_0/tiles'].keys():
        tile_image = f[f'slide_0/tiles/{tile_key}/image'][:]
        # 无需加载全部切片即可处理...
```

## 切片管理

### 切片生成策略

**固定尺寸无重叠切片：**
```python
wsi.generate_tiles(
    level=1,
    tile_size=256,
    stride=256,  # 步长=切片尺寸 → 无重叠
    pad=False  # 不填充边缘切片
)
```
- **适用场景：** 标准切片处理、分类任务
- **优点：** 简单无冗余、处理快速
- **缺点：** 切片边界存在边缘效应

**重叠切片：**
```python
wsi.generate_tiles(
    level=1,
    tile_size=256,
    stride=128,  # 50% 重叠
    pad=False
)
```
- **适用场景：** 分割、检测任务（减少边界伪影）
- **优点：** 边界处理更优、拼接更平滑
- **缺点：** 切片数量增多、存在冗余计算

**基于组织内容的自适应切片：**
```python
from pathml.utils import adaptive_tile_generation

# 仅在组织区域生成切片
wsi.generate_tiles(level=1, tile_size=256, stride=256)

# 过滤保留含有效组织的切片
tissue_tiles = []
for tile in wsi.tiles:
    if tile.masks.get('tissue') is not None:
        tissue_coverage = tile.masks['tissue'].sum() / (tile_size**2)
        if tissue_coverage > 0.5:  # 保留组织覆盖率>50%的切片
            tissue_tiles.append(tile)

wsi.tiles = tissue_tiles
```
- **适用场景：** 稀疏组织样本、效率优先
- **优点：** 减少背景切片处理
- **缺点：** 需预执行组织检测步骤

### 切片拼接

将处理后的切片重建为完整图像：

```python
from pathml.utils import stitch_tiles

# 处理切片
for tile in wsi.tiles:
    tile.prediction = model.predict(tile.image)

# 将预测结果拼接至全分辨率
full_prediction_map = stitch_tiles(
    wsi.tiles,
    output_shape=wsi.level_dimensions[1],  # 使用层级1的尺寸
    tile_size=256,
    stride=256,
    method='average'  # 'average'、'max' 或 'first'
)

# 可视化
import matplotlib.pyplot as plt
plt.figure(figsize=(15, 15))
plt.imshow(full_prediction_map)
plt.title('拼接预测图')
plt.axis('off')
plt.show()
```

**拼接方法：**
- `'average'`：重叠区域取平均值（平滑过渡）
- `'max'`：重叠区域取最大值
- `'first'`：保留首个切片值（无混合）
- `'weighted'`：距离加权混合实现平滑边界

### 切片缓存

缓存频繁访问的切片以加速迭代：

```python
from pathml.utils import TileCache

# 创建缓存
cache = TileCache(max_size_gb=10)

# 首次迭代时缓存切片
for i, tile in enumerate(wsi.tiles):
    cache.add(f'tile_{i}', tile.image)
    # 处理切片...

# 后续迭代使用缓存数据
for i in range(len(wsi.tiles)):
    cached_image = cache.get(f'tile_{i}')
    # 快速访问...
```

## 数据集组织

### 大型项目目录结构

采用统一结构组织病理项目：

```
project/
├── raw_slides/ 原始切片
│   ├── cohort1/ 队列1
│   │   ├── slide001.svs
│   │   ├── slide002.svs
│   │   └── ...
│   └── cohort2/ 队列2
│       └── ...
├── processed/ 处理结果
│   ├── cohort1/
│   │   ├── slide001.h5
│   │   ├── slide002.h5
│   │   └── ...
│   └── cohort2/
│       └── ...
├── features/ 特征
│   ├── cohort1_features.h5
│   └── cohort2_features.h5
├── models/ 模型
│   ├── hovernet_checkpoint.pth
│   └── classifier.onnx
├── results/ 结果
│   ├── predictions/ 预测结果
│   ├── visualizations/ 可视化
│   └── metrics.csv 指标
└── metadata/ 元数据
    ├── clinical_data.csv 临床数据
    └── slide_manifest.csv 切片清单
```

### 元数据管理

存储切片级和队列级元数据：

```python
import pandas as pd

# 切片清单
manifest = pd.DataFrame({
    'slide_id': ['slide001', 'slide002', 'slide003'],
    'path': ['raw_slides/cohort1/slide001.svs', ...],
    'cohort': ['cohort1', 'cohort1', 'cohort2'],
    'tissue_type': ['breast', 'breast', 'lung'],  # 组织类型
    'scanner': ['Aperio', 'Hamamatsu', 'Aperio'],  # 扫描仪
    'magnification': [40, 40, 20],  # 放大倍数
    'staining': ['H&E', 'H&E', 'H&E']  # 染色类型
})

manifest.to_csv('metadata/slide_manifest.csv', index=False)

# 临床数据
clinical = pd.DataFrame({
    'slide_id': ['slide001', 'slide002', 'slide003'],
    'patient_id': ['P001', 'P002', 'P003'],  # 患者ID
    'age': [55, 62, 48],  # 年龄
    'diagnosis': ['invasive', 'in_situ', 'invasive'],  # 诊断结果
    'stage': ['II', 'I', 'III'],  # 分期
    'outcome': ['favorable', 'favorable', 'poor']  # 预后
})

clinical.to_csv('metadata/clinical_data.csv', index=False)

# 加载并合并
manifest = pd.read_csv('metadata/slide_manifest.csv')
clinical = pd.read_csv('metadata/clinical_data.csv')
data = manifest.merge(clinical, on='slide_id')
```

## 批处理策略

### 顺序处理

单张切片顺序处理（内存高效）：

```python
import glob
from pathml.core import SlideData
from pathml.preprocessing import Pipeline

slide_paths = glob.glob('raw_slides/**/*.svs', recursive=True)

for slide_path in slide_paths:
    # 加载切片
    wsi = SlideData.from_slide(slide_path)
    wsi.generate_tiles(level=1, tile_size=256, stride=256)

    # 处理
    pipeline.run(wsi)

    # 保存
    output_path = slide_path.replace('raw_slides', 'processed').replace('.svs', '.h5')
    wsi.to_hdf5(output_path)

    print(f"已处理: {slide_path}")
```

### 基于 Dask 的并行处理

并行处理多张切片：

```python
from pathml.core import SlideDataset
from dask.distributed import Client, LocalCluster
from pathml.preprocessing import Pipeline

# 启动 Dask 集群
cluster = LocalCluster(
    n_workers=8,
    threads_per_worker=2,
    memory_limit='8GB',
    dashboard_address=':8787'  # 在 localhost:8787 查看进度
)
client = Client(cluster)

# 创建数据集
slide_paths = glob.glob('raw_slides/**/*.svs', recursive=True)
dataset = SlideDataset(slide_paths, tile_size=256, stride=256, level=1)

# 分布式处理
dataset.run(
    pipeline,
    distributed=True,
    client=client,
    scheduler='distributed'
)

# 保存结果
for i, slide in enumerate(dataset):
    output_path = slide_paths[i].replace('raw_slides', 'processed').replace('.svs', '.h5')
    slide.to_hdf5(output_path)

client.close()
cluster.close()
```

### 基于任务阵列的批处理

适用于 HPC 集群（SLURM, PBS）：

```python
# submit_jobs.py
import os
import glob

slide_paths = glob.glob('raw_slides/**/*.svs', recursive=True)

# 写入切片列表
with open('slide_list.txt', 'w') as f:
    for path in slide_paths:
        f.write(path + '\n')

# 创建 SLURM 任务脚本
slurm_script = """#!/bin/bash
#SBATCH --array=1-{n_slides}
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=4:00:00
#SBATCH --output=logs/slide_%A_%a.out

# 获取当前阵列任务对应的切片路径
SLIDE_PATH=$(sed -n "${{SLURM_ARRAY_TASK_ID}}p" slide_list.txt)

# 执行处理
python process_slide.py --slide_path $SLIDE_PATH
""".format(n_slides=len(slide_paths))

with open('submit_jobs.sh', 'w') as f:
    f.write(slurm_script)

# 提交任务: sbatch submit_jobs.sh
```

```python
# process_slide.py
import argparse
from pathml.core import SlideData
from pathml.preprocessing import Pipeline

parser = argparse.ArgumentParser()
parser.add_argument('--slide_path', type=str, required=True)
args = parser.parse_args()

# 加载并处理
wsi = SlideData.from_slide(args.slide_path)
wsi.generate_tiles(level=1, tile_size=256, stride=256)

pipeline = Pipeline([...])
pipeline.run(wsi)

# 保存
output_path = args.slide_path.replace('raw_slides', 'processed').replace('.svs', '.h5')
wsi.to_hdf5(output_path)

print(f"已处理: {args.slide_path}")
```

## 特征提取与存储

### 特征提取

```python
from pathml.core import SlideData
import torch
import numpy as np

# 加载预训练特征提取模型
model = torch.load('models/feature_extractor.pth')
model.eval()
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = model.to(device)

# 加载处理后的切片
wsi = SlideData.from_hdf5('processed/slide001.h5')

# 为每个切片提取特征
features = []
coords = []

for tile in wsi.tiles:
    # 预处理切片
    tile_tensor = torch.from_numpy(tile.image).permute(2, 0, 1).unsqueeze(0).float()
    tile_tensor = tile_tensor.to(device)

    # 提取特征
    with torch.no_grad():
        feature_vec = model(tile_tensor).cpu().numpy().flatten()

    features.append(feature_vec)
    coords.append(tile.coords)

features = np.array(features)  # 形状: (n_tiles, feature_dim)
coords = np.array(coords)  # 形状: (n_tiles, 2)
```

### 在 HDF5 中存储特征

```python
import h5py

# 保存特征
with h5py.File('features/slide001_features.h5', 'w') as f:
    f.create_dataset('features', data=features, compression='gzip')
    f.create_dataset('coords', data=coords)
    f.attrs['feature_dim'] = features.shape[1]
    f.attrs['num_tiles'] = features.shape[0]
    f.attrs['model'] = 'resnet50'

# 加载特征
with h5py.File('features/slide001_features.h5', 'r') as f:
    features = f['features'][:]
    coords = f['coords'][:]
    feature_dim = f.attrs['feature_dim']
```

### 多切片特征数据库

```python
# 创建统一特征数据库
import h5py
import glob

feature_files = glob.glob('features/*_features.h5')

with h5py.File('features/all_features.h5', 'w') as out_f:
    for i, feature_file in enumerate(feature_files):
        slide_name = feature_file.split('/')[-1].replace('_features.h5', '')

        with h5py.File(feature_file, 'r') as in_f:
            features = in_f['features'][:]
            coords = in_f['coords'][:]

            # 存入统一文件
            grp = out_f.create_group(f'slide_{i}')
            grp.create_dataset('features', data=features, compression='gzip')
            grp.create_dataset('coords', data=coords)
            grp.attrs['slide_name'] = slide_name

# 从所有切片查询特征
with h5py.File('features/all_features.h5', 'r') as f:
    for slide_key in f.keys():
        slide_name = f[slide_key].attrs['slide_name']
        features = f[f'{slide_key}/features'][:]
        # 处理...
```

## 数据版本控制

### 使用 DVC

```markdown
for chunk in iter(lambda: f.read(4096), b""):
            hash_md5.update(chunk)
    return hash_md5.hexdigest()

# 创建校验和清单
slide_paths = glob.glob('raw_slides/**/*.svs', recursive=True)
checksums = []

for slide_path in slide_paths:
    checksum = compute_checksum(slide_path)
    checksums.append({
        'path': slide_path,
        'checksum': checksum,
        'size_mb': os.path.getsize(slide_path) / 1e6
    })

checksum_df = pd.DataFrame(checksums)
checksum_df.to_csv('metadata/checksums.csv', index=False)

# 验证文件
def validate_files(manifest_path):
    manifest = pd.read_csv(manifest_path)
    for _, row in manifest.iterrows():
        current_checksum = compute_checksum(row['path'])
        if current_checksum != row['checksum']:
            print(f"ERROR: Checksum mismatch for {row['path']}")
        else:
            print(f"OK: {row['path']}")

validate_files('metadata/checksums.csv')
```

## 性能优化

### 压缩设置

优化 HDF5 压缩的速度与体积权衡：

```python
import h5py

# 快速压缩（CPU占用低，文件较大）
with h5py.File('output.h5', 'w') as f:
    f.create_dataset(
        'images',
        data=images,
        compression='gzip',
        compression_opts=1  # 级别1-9，数值越低越快
    )

# 最大压缩（CPU占用高，文件较小）
with h5py.File('output.h5', 'w') as f:
    f.create_dataset(
        'images',
        data=images,
        compression='gzip',
        compression_opts=9
    )

# 平衡模式（推荐）
with h5py.File('output.h5', 'w') as f:
    f.create_dataset(
        'images',
        data=images,
        compression='gzip',
        compression_opts=4,
        chunks=True  # 启用分块以优化I/O
    )
```

### 分块策略

根据访问模式优化分块存储：

```python
# 适用于基于瓦片的访问（每次访问单个瓦片）
with h5py.File('tiles.h5', 'w') as f:
    f.create_dataset(
        'tiles',
        shape=(n_tiles, 256, 256, 3),
        dtype='uint8',
        chunks=(1, 256, 256, 3),  # 每个分块包含一个瓦片
        compression='gzip'
    )

# 适用于基于通道的访问（访问某通道的所有瓦片）
with h5py.File('tiles.h5', 'w') as f:
    f.create_dataset(
        'tiles',
        shape=(n_tiles, 256, 256, 3),
        dtype='uint8',
        chunks=(n_tiles, 256, 256, 1),  # 包含某通道的所有瓦片
        compression='gzip'
    )
```

### 内存映射数组

对大数组使用内存映射：

```python
import numpy as np

# 保存为内存映射文件
features_mmap = np.memmap(
    'features/features.mmap',
    dtype='float32',
    mode='w+',
    shape=(n_tiles, feature_dim)
)

# 填充数据
for i, tile in enumerate(wsi.tiles):
    features_mmap[i] = extract_features(tile)

# 刷新到磁盘
features_mmap.flush()

# 无需载入内存即可读取
features_mmap = np.memmap(
    'features/features.mmap',
    dtype='float32',
    mode='r',
    shape=(n_tiles, feature_dim)
)

# 高效访问子集
subset = features_mmap[1000:2000]  # 仅加载所需行
```

## 最佳实践

1. **使用HDF5存储处理后的数据**：将预处理瓦片和特征保存至HDF5以实现快速访问
2. **分离原始数据与处理结果**：原始切片应与处理输出分开存储
3. **维护元数据**：跟踪切片来源、处理参数和临床标注
4. **实施校验和**：尤其在传输后验证数据完整性
5. **数据集版本控制**：使用DVC等工具管理大型数据集版本
6. **优化存储**：平衡压缩级别与I/O性能
7. **按队列组织**：按研究队列构建目录结构以保持清晰
8. **定期备份**：将数据和元数据备份至远程存储
9. **记录处理过程**：保存处理步骤、参数和版本的日志
10. **监控磁盘使用**：跟踪数据集增长时的存储消耗

## 常见问题与解决方案

**问题：HDF5文件过大**
- 提高压缩级别：`compression_opts=9`
- 仅存储必要数据（避免冗余副本）
- 使用合适数据类型（图像用uint8而非float64）

**问题：HDF5读写缓慢**
- 根据访问模式优化分块大小
- 降低压缩级别以提升I/O速度
- 使用SSD替代HDD存储
- 通过MPI启用并行HDF5

**问题：磁盘空间不足**
- 处理完成后删除中间文件
- 压缩非活跃数据集
- 将旧数据移至归档存储
- 对低频访问数据使用云存储

**问题：数据损坏或丢失**
- 实施定期备份
- 使用RAID实现冗余
- 传输后验证校验和
- 使用版本控制系统（如DVC）

## 扩展资源

- **HDF5文档：** https://www.hdfgroup.org/solutions/hdf5/
- **h5py：** https://docs.h5py.org/
- **DVC（数据版本控制）：** https://dvc.org/
- **Dask：** https://docs.dask.org/
- **PathML数据管理API：** https://pathml.readthedocs.io/en/latest/api_data_reference.html
```
