# 多参数成像

## 概述

PathML 为多参数成像技术提供专门支持，这些技术可在单细胞分辨率下同时测量多个标记物。这些技术包括 CODEX、Vectra 多重免疫荧光、MERFISH 以及其他空间蛋白质组学和转录组学平台。PathML 处理每种技术特有的数据结构、处理要求和量化工作流程。

## 支持的技术

### CODEX（索引共检测）
- 循环免疫荧光成像
- 同时检测 40+ 种蛋白质标记物
- 单细胞空间蛋白质组学
- 采用抗体条形码的多周期采集

### Vectra Polaris
- 多光谱多重免疫荧光
- 每张玻片 6-8 种标记物
- 光谱解混
- 全玻片扫描

### MERFISH（多重容错荧光原位杂交）
- 空间转录组学
- 检测数百至数千个基因
- 单分子分辨率
- 纠错条形码

### 其他平台
- CycIF（循环免疫荧光）
- IMC（成像质谱流式）
- MIBI（多重离子束成像）

## CODEX 工作流程

### 加载 CODEX 数据

CODEX 数据通常按多周期采集的多通道图像堆栈组织：

```python
from pathml.core import CODEXSlide

# 加载 CODEX 数据集
codex_slide = CODEXSlide(
    path='path/to/codex_directory',
    stain='IF',  # 免疫荧光
    backend='bioformats'
)

# 检查通道和周期
print(f"通道数量: {codex_slide.num_channels}")
print(f"通道名称: {codex_slide.channel_names}")
print(f"周期数量: {codex_slide.num_cycles}")
print(f"图像尺寸: {codex_slide.shape}")
```

**CODEX 目录结构:**
```
codex_directory/
├── cyc001_reg001/
│   ├── 1_00001_Z001_CH1.tif
│   ├── 1_00001_Z001_CH2.tif
│   └── ...
├── cyc002_reg001/
│   └── ...
└── channelnames.txt
```

### CODEX 预处理流程

完整的 CODEX 数据处理流程：

```python
from pathml.preprocessing import Pipeline, CollapseRunsCODEX, SegmentMIF, QuantifyMIF

# 创建 CODEX 专用流程
codex_pipeline = Pipeline([
    # 1. 整合多周期数据
    CollapseRunsCODEX(
        z_slice=2,  # 从 z 堆栈中选择焦平面
        run_order=None,  # 自动周期排序，或指定 [0, 1, 2, ...]
        method='max'  # 跨周期聚合方法：'max'、'mean' 或 'median'
    ),

    # 2. 使用 Mesmer 进行细胞分割
    SegmentMIF(
        nuclear_channel='DAPI',
        cytoplasm_channel='CD45',  # 或其他膜/细胞质标记物
        model='mesmer',
        image_resolution=0.377,  # 微米/像素
        compartment='whole-cell'  # 'nuclear'、'cytoplasm' 或 'whole-cell'
    ),

    # 3. 量化单细胞标记物表达
    QuantifyMIF(
        segmentation_mask_name='cell_segmentation',
        markers=[
            'DAPI', 'CD3', 'CD4', 'CD8', 'CD20', 'CD45',
            'CD68', 'PD1', 'PDL1', 'Ki67', 'panCK'
        ],
        output_format='anndata'
    )
])

# 运行流程
codex_pipeline.run(codex_slide)

# 访问结果
segmentation_mask = codex_slide.masks['cell_segmentation']
cell_data = codex_slide.cell_data  # AnnData 对象
```

### CollapseRunsCODEX

将多周期 CODEX 采集整合为单张多通道图像：

```python
from pathml.preprocessing import CollapseRunsCODEX

transform = CollapseRunsCODEX(
    z_slice=2,  # 选择 z 平面（0 起始索引）
    run_order=[0, 1, 2, 3],  # 采集周期顺序
    method='max',  # 跨周期聚合方法
    background_subtract=True,  # 扣除背景荧光
    channel_mapping=None  # 可选：重映射通道顺序
)
```

**参数:**
- `z_slice`: 从 z 堆栈中提取的焦平面（通常为中间层）
- `run_order`: 周期顺序；None 表示自动检测
- `method`: 多周期通道合并方法（'max'、'mean'、'median'）
- `background_subtract`: 是否扣除背景荧光

**输出:** 包含所有标记物的单张多通道图像 (H, W, C)

### 使用 Mesmer 进行细胞分割

DeepCell Mesmer 为多参数成像提供精确的细胞分割：

```python
from pathml.preprocessing import SegmentMIF

transform = SegmentMIF(
    nuclear_channel='DAPI',  # 核标记物（必需）
    cytoplasm_channel='CD45',  # 细胞质/膜标记物（必需）
    model='mesmer',  # DeepCell Mesmer 模型
    image_resolution=0.377,  # 微米/像素（影响精度）
    compartment='whole-cell',  # 分割输出
    min_cell_size=50,  # 最小细胞尺寸（像素）
    max_cell_size=1000  # 最大细胞尺寸（像素）
)
```

**细胞质通道选择:**
- **CD45**: 泛白细胞标记物（适用于免疫富集组织）
- **panCK**: 泛细胞角蛋白（适用于上皮组织）
- **CD298/b2m**: 通用膜标记物
- **组合**: 平均多个膜标记物

**分割区域选项:**
- `'whole-cell'`: 全细胞分割（细胞核+细胞质）
- `'nuclear'`: 仅细胞核分割
- `'cytoplasm'`: 仅细胞质区域

### 远程分割

使用 DeepCell 云 API 实现无本地 GPU 的分割：

```python
from pathml.preprocessing import SegmentMIFRemote

transform = SegmentMIFRemote(
    nuclear_channel='DAPI',
    cytoplasm_channel='CD45',
    model='mesmer',
    api_url='https://deepcell.org/api/predict',
    timeout=300  # 超时时间（秒）
)
```

### 标记物量化

从分割图像中提取单细胞标记物表达：

```python
from pathml.preprocessing import QuantifyMIF

transform = QuantifyMIF(
    segmentation_mask_name='cell_segmentation',
    markers=['DAPI', 'CD3', 'CD4', 'CD8', 'CD20', 'CD68', 'panCK'],
    output_format='anndata',  # 或 'dataframe'
    statistics=['mean', 'median', 'std', 'total'],  # 聚合方法
    compartments=['whole-cell', 'nuclear', 'cytoplasm']  # 多掩膜时使用
)
```

**输出:** AnnData 对象包含：
- `adata.X`: 标记物表达矩阵（细胞 × 标记物）
- `adata.obs`: 细胞元数据（细胞 ID、坐标、面积等）
- `adata.var`: 标记物元数据
- `adata.obsm['spatial']`: 细胞质心坐标

### 与 AnnData 集成

将多个 CODEX 玻片处理为统一 AnnData 对象：

```python
from pathml.core import SlideDataset
import anndata as ad

# 处理多个玻片
slide_paths = ['slide1', 'slide2', 'slide3']
dataset = SlideDataset(
    [CODEXSlide(p, stain='IF') for p in slide_paths]
)

# 在所有玻片上运行流程
dataset.run(codex_pipeline, distributed=True, n_workers=8)

# 合并为单个 AnnData
adatas = []
for slide in dataset:
    adata = slide.cell_data
    adata.obs['slide_id'] = slide.name
    adatas.append(adata)

# 拼接
combined_adata = ad.concat(adatas, join='outer', label='batch', keys=slide_paths)

# 保存用于下游分析
combined_adata.write('codex_dataset.h5ad')
```

## Vectra 工作流程

### 加载 Vectra 数据

Vectra 数据以专有 `.qptiff` 格式存储：

```python
from pathml.core import SlideData, SlideType

# 加载 Vectra 玻片
vectra_slide = SlideData.from_slide(
    'path/to/slide.qptiff',
    backend=SlideType.VectraQPTIFF
)

# 访问光谱通道
print(f"通道: {vectra_slide.channel_names}")
```

### Vectra 预处理

```python
from pathml.preprocessing import Pipeline, CollapseRunsVectra, SegmentMIF, QuantifyMIF

vectra_pipeline = Pipeline([
    # 1. 处理 Vectra 多通道数据
    CollapseRunsVectra(
        wavelengths=[520, 540, 570, 620, 670, 780],  # 发射波长
        unmix=True,  # 应用光谱解混
        autofluorescence_correction=True
    ),

    # 2. 细胞分割
    SegmentMIF(
        nuclear_channel='DAPI',
        cytoplasm_channel='FITC',
        model='mesmer',
        image_resolution=0.5
    ),

    # 3. 量化
    QuantifyMIF(
        segmentation_mask_name='cell_segmentation',
        markers=['DAPI', 'CD3', 'CD8', 'PD1', 'PDL1', 'panCK'],
        output_format='anndata'
    )
])

vectra_pipeline.run(vectra_slide)
```

## 下游分析

### 细胞类型注释

基于标记物表达注释细胞类型：

```python
import anndata as ad
import numpy as np

# 加载量化数据
adata = ad.read_h5ad('codex_dataset.h5ad')

# 通过标记物阈值定义细胞类型
def annotate_cell_types(adata, thresholds):
    cell_types = np.full(adata.n_obs, 'Unknown', dtype=object)

    # T 细胞: CD3+
    cd3_pos = adata[:, 'CD3'].X.flatten() > thresholds['CD3']
    cell_types[cd3_pos] = 'T 细胞'

    # CD4 T 细胞: CD3+ CD4+ CD8-
    cd4_tcells = (
        (adata[:, 'CD3'].X.flatten() > thresholds['CD3']) &
        (adata[:, 'CD4'].X.flatten() > thresholds['CD4']) &
        (adata[:, 'CD8'].X.flatten() < thresholds['CD8'])
    )
    cell_types[cd4_tcells] = 'CD4 T 细胞'

    # CD8 T 细胞: CD3+ CD8+ CD4-
    cd8_tcells = (
        (adata[:, 'CD3'].X.flatten() > thresholds['CD3']) &
        (adata[:, 'CD8'].X.flatten() > thresholds['CD8']) &
        (adata[:, 'CD4'].X.flatten() < thresholds['CD4'])
    )
    cell_types[cd8_tcells] = 'CD8 T 细胞'

    # B 细胞: CD20+
    b_cells = adata[:, 'CD20'].X.flatten() > thresholds['CD20']
    cell_types[b_cells] = 'B 细胞'

    # 巨噬细胞: CD68+
    macrophages = adata[:, 'CD68'].X.flatten() > thresholds['CD68']
    cell_types[macrophages] = '巨噬细胞'

    # 肿瘤细胞: panCK+
    tumor = adata[:, 'panCK'].X.flatten() > thresholds['panCK']
    cell_types[tumor] = '肿瘤细胞'

    return cell_types

# 应用注释
thresholds = {
    'CD3': 0.5,
    'CD4': 0.4,
    'CD8': 0.4,
    'CD20': 0.3,
    'CD68': 0.3,
    'panCK': 0.5
}

adata.obs['cell_type'] = annotate_cell_types(adata, thresholds)

# 可视化细胞类型组成
import matplotlib.pyplot as plt
cell_type_counts = adata.obs['cell_type'].value_counts()
plt.figure(figsize=(10, 6))
cell_type_counts.plot(kind='bar')
plt.xlabel('细胞类型')
plt.ylabel('数量')
plt.title('细胞类型组成')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### 聚类分析

无监督聚类识别细胞群体：

```python
import scanpy as sc

# 聚类预处理
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.scale(adata, max_value=10)

# PCA
sc.tl.pca(adata, n_comps=50)

# 邻域图
sc.pp.neighbors(adata, n_neighbors=15, n_pcs=30)

# UMAP 嵌入
sc.tl.umap(adata)

# Leiden 聚类
sc.tl.leiden(adata, resolution=0.5)

# 可视化
sc.pl.umap(adata, color=['leiden', 'CD3', 'CD8', 'CD20', 'panCK'])
```

### 空间可视化

在空间背景下可视化细胞：

```python
import matplotlib.pyplot as plt

# 空间散点图
fig, ax = plt.subplots(figsize=(15, 15))

# 按细胞类型着色
cell_types = adata.obs['cell_type'].unique()
colors = plt.cm.tab10(np.linspace(0, 1, len(cell_types)))

for i, cell_type in enumerate(cell_types):
    mask = adata.obs['cell_type'] == cell_type
    coords = adata.obsm['spatial'][mask]
    ax.scatter(
        coords[:, 0],
        coords[:, 1],
        c=[colors[i]],
        label=cell_type,
        s=5,
        alpha=0.7
    )

ax.legend(markerscale=2)
ax.set_xlabel('X (像素)')
ax.set_ylabel('Y (像素)')
ax.set_title('空间细胞类型分布')
ax.axis('equal')
plt.tight_layout()
plt.show()
```

### 空间邻域分析

分析细胞邻域和相互作用：

```python
import squidpy as sq

# 计算空间邻域富集
sq.gr.spatial_neighbors(adata, coord_type='generic', spatial_key='spatial')

# 邻域富集检验
sq.gr.nhood_enrichment(adata, cluster_key='cell_type')

# 可视化相互作用矩阵
sq.pl.nhood_enrichment(adata, cluster_key='cell_type')

# 共现分数
sq.gr.co_occurrence(adata, cluster_key='cell_type')
sq.pl.co_occurrence(
    adata,
    cluster_key='cell_type',
    clusters=['CD8 T 细胞', '肿瘤细胞'],
    figsize=(8, 8)
)
```

### 空间自相关

检测标记物的空间聚类：

```python
# Moran's I 空间自相关
sq.gr.spatial_autocorr(
    adata,
    mode='moran',
    genes=['CD3', 'CD8', 'PD1', 'PDL1', 'panCK']
)

# 可视化
results = adata.uns['moranI']
print(results.head())
```

## MERFISH 工作流程

### 加载 MERFISH 数据

```python
from pathml.core import MERFISHSlide

# 加载 MERFISH 数据集
merfish_slide = MERFISHSlide(
    path='path/to/merfish_data',
    fov_size=2048,  # 视野尺寸
    microns_per_pixel=0.108
)
```

### MERFISH 处理

```python
from pathml.preprocessing import Pipeline, DecodeMERFISH, SegmentMIF

merfish_pipeline = Pipeline([
    # 1. 将条形码解码为基因
    DecodeMERFISH(
        codebook='path/to/codebook.csv',
        error_correction=True,
        distance_threshold=0.5
    ),

    # 2. 细胞分割
    SegmentMIF(
        nuclear_channel='DAPI',
        cytoplasm_channel='polyT',  # poly(T) 染色标记细胞边界
        model='mesmer'
    ),

    # 3. 将转录本分配至细胞
    AssignTranscripts(
        segmentation_mask_name='cell_se

```python
import matplotlib.pyplot as plt
plt.hist(qc_metrics['cell_sizes'], bins=50)
plt.xlabel('细胞大小（像素）')
plt.ylabel('频率')
plt.title('细胞大小分布')
plt.show()
```

### 标记物表达质量控制

```python
import scanpy as sc

# 加载AnnData对象
adata = ad.read_h5ad('codex_dataset.h5ad')

# 计算质控指标
adata.obs['total_intensity'] = adata.X.sum(axis=1)
adata.obs['n_markers_detected'] = (adata.X > 0).sum(axis=1)

# 过滤低质量细胞
adata = adata[adata.obs['total_intensity'] > 100, :]
adata = adata[adata.obs['n_markers_detected'] >= 3, :]

# 可视化
sc.pl.violin(adata, ['total_intensity', 'n_markers_detected'], multi_panel=True)
```

## 批量处理

高效处理大型多参数数据集：

```python
from pathml.core import SlideDataset
from pathml.preprocessing import Pipeline
from dask.distributed import Client
import glob

# 启动Dask集群
client = Client(n_workers=16, threads_per_worker=2, memory_limit='8GB')

# 查找所有CODEX玻片
slide_dirs = glob.glob('data/codex_slides/*/')

# 创建数据集
codex_slides = [CODEXSlide(d, stain='IF') for d in slide_dirs]
dataset = SlideDataset(codex_slides)

# 并行运行处理流程
dataset.run(
    codex_pipeline,
    distributed=True,
    client=client,
    scheduler='distributed'
)

# 保存处理后的数据
for i, slide in enumerate(dataset):
    slide.cell_data.write(f'processed/slide_{i}.h5ad')

client.close()
```

## 与其他工具集成

### 导出至空间分析工具

```python
# 导出到Giotto
def export_to_giotto(adata, output_dir):
    import os
    os.makedirs(output_dir, exist_ok=True)

    # 表达矩阵
    pd.DataFrame(
        adata.X.T,
        index=adata.var_names,
        columns=adata.obs_names
    ).to_csv(f'{output_dir}/expression.csv')

    # 细胞坐标
    pd.DataFrame(
        adata.obsm['spatial'],
        columns=['x', 'y'],
        index=adata.obs_names
    ).to_csv(f'{output_dir}/spatial_locs.csv')

# 导出到Seurat
def export_to_seurat(adata, output_file):
    adata.write_h5ad(output_file)
    # 在R中读取: library(Seurat); ReadH5AD(output_file)
```

## 最佳实践

1. **分割通道选择：**
   - 使用最亮、最稳定的核标记物（通常为DAPI）
   - 根据组织类型选择膜/胞浆标记物
   - 测试多种选项以优化分割效果

2. **背景扣除：**
   - 在定量分析前应用以减少自发荧光
   - 使用空白/对照图像建模背景

3. **质量控制：**
   - 在样本区域可视化分割结果
   - 检查细胞大小分布中的异常值
   - 验证标记物表达范围

4. **细胞类型注释：**
   - 从经典标记物开始（CD3, CD20, panCK）
   - 使用多个标记物确保稳健分类
   - 考虑无监督聚类发现新细胞群体

5. **空间分析：**
   - 考虑组织结构（上皮、间质等）
   - 解释相互作用时评估局部密度
   - 使用置换检验确定统计显著性

6. **批次效应处理：**
   - 在AnnData.obs中记录批次信息
   - 合并多组实验时应用批次校正
   - 通过UMAP着色可视化批次效应

## 常见问题与解决方案

**问题：分割质量差**
- 确认核通道和胞浆通道设置正确
- 调整image_resolution参数匹配实际分辨率
- 尝试不同胞浆标记物
- 手动调节细胞最小/最大尺寸参数

**问题：标记物信号弱**
- 检查背景扣除伪影
- 确认通道名称与实际通道匹配
- 检查原始图像技术问题（焦距、曝光）

**问题：细胞类型注释不符预期**
- 调整标记物阈值（过高/过低）
- 可视化标记物分布以设置数据驱动阈值
- 检查抗体特异性问题

**问题：空间分析未显示显著相互作用**
- 增加邻域半径
- 确保各类细胞数量充足
- 验证空间坐标缩放正确

## 附加资源

- **PathML多参数API:** https://pathml.readthedocs.io/en/latest/api_multiparametric_reference.html
- **CODEX:** https://www.akoyabio.com/codex/
- **Vectra:** https://www.akoyabio.com/vectra/
- **DeepCell Mesmer:** https://www.deepcell.org/
- **Scanpy:** https://scanpy.readthedocs.io/ (单细胞分析)
- **Squidpy:** https://squidpy.readthedocs.io/ (空间组学分析)
