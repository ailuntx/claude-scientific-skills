# 图像加载与格式

## 概述

PathML 全面支持从 160 多种专有医学影像格式加载全切片图像（WSI）。该框架通过统一的幻灯片类和接口抽象了供应商特定的复杂性，实现跨不同文件格式无缝访问图像金字塔、元数据和感兴趣区域。

## 支持的格式

PathML 支持以下幻灯片格式：

### 明场显微镜格式
- **Aperio SVS** (`.svs`) - Leica Biosystems
- **Hamamatsu NDPI** (`.ndpi`) - Hamamatsu Photonics
- **Leica SCN** (`.scn`) - Leica Biosystems
- **Zeiss ZVI** (`.zvi`) - Carl Zeiss
- **3DHISTECH** (`.mrxs`) - 3DHISTECH Ltd.
- **Ventana BIF** (`.bif`) - Roche Ventana
- **通用分块 TIFF** (`.tif`, `.tiff`)

### 医学影像标准
- **DICOM** (`.dcm`) - 医学数字成像与通信
- **OME-TIFF** (`.ome.tif`, `.ome.tiff`) - 开放式显微镜环境

### 多参数成像
- **CODEX** - 空间蛋白质组学成像
- **Vectra** (`.qptiff`) - 多重免疫荧光
- **MERFISH** - 多重容错荧光原位杂交

PathML 利用 OpenSlide 和其他专业库自动处理特定格式的细微差别。

## 核心图像加载类

### SlideData

`SlideData` 是 PathML 中表示全切片图像的基础类。

**从文件加载：**
```python
from pathml.core import SlideData

# 加载全切片图像
wsi = SlideData.from_slide("path/to/slide.svs")

# 使用特定后端加载
wsi = SlideData.from_slide("path/to/slide.svs", backend="openslide")

# 从 OME-TIFF 加载
wsi = SlideData.from_slide("path/to/slide.ome.tiff", backend="bioformats")
```

**关键属性：**
- `wsi.slide` - 后端幻灯片对象（OpenSlide、BioFormats 等）
- `wsi.tiles` - 图像分块集合
- `wsi.metadata` - 幻灯片元数据字典
- `wsi.level_dimensions` - 图像金字塔层级尺寸
- `wsi.level_downsamples` - 各金字塔层级的降采样因子

**方法：**
- `wsi.generate_tiles()` - 从幻灯片生成分块
- `wsi.read_region()` - 读取指定层级的具体区域
- `wsi.get_thumbnail()` - 获取缩略图

### SlideType

`SlideType` 是定义支持的后端枚举：

```python
from pathml.core import SlideType

# 可用后端
SlideType.OPENSLIDE  # 适用于多数 WSI 格式（SVS、NDPI 等）
SlideType.BIOFORMATS  # 适用于 OME-TIFF 及其他格式
SlideType.DICOM  # 适用于 DICOM WSI
SlideType.VectraQPTIFF  # 适用于 Vectra 多重免疫荧光
```

### 专用幻灯片类

PathML 为特定成像模式提供专用幻灯片类：

**CODEXSlide：**
```python
from pathml.core import CODEXSlide

# 加载 CODEX 空间蛋白质组学数据
codex_slide = CODEXSlide(
    path="path/to/codex_dir",
    stain="IF",  # 免疫荧光
    backend="bioformats"
)
```

**VectraSlide：**
```python
from pathml.core import types

# 加载 Vectra 多重免疫荧光数据
vectra_slide = SlideData.from_slide(
    "path/to/vectra.qptiff",
    backend=SlideType.VectraQPTIFF
)
```

**MultiparametricSlide：**
```python
from pathml.core import MultiparametricSlide

# 通用多参数成像
mp_slide = MultiparametricSlide(path="path/to/multiparametric_data")
```

## 加载策略

### 基于分块的加载

对于大型 WSI 文件，基于分块的加载可实现内存高效处理：

```python
from pathml.core import SlideData

# 加载幻灯片
wsi = SlideData.from_slide("path/to/slide.svs")

# 在特定放大级别生成分块
wsi.generate_tiles(
    level=0,  # 金字塔层级（0=最高分辨率）
    tile_size=256,  # 分块尺寸（像素）
    stride=256,  # 分块间距（256=无重叠）
    pad=False  # 是否填充边缘分块
)

# 遍历分块
for tile in wsi.tiles:
    image = tile.image  # numpy 数组
    coords = tile.coords  # (x, y) 坐标
    # 处理分块...
```

**重叠分块：**
```python
# 生成 50% 重叠的分块
wsi.generate_tiles(
    level=0,
    tile_size=256,
    stride=128  # 50% 重叠
)
```

### 基于区域的加载

直接提取特定感兴趣区域：

```python
# 在指定位置和层级读取区域
region = wsi.read_region(
    location=(10000, 15000),  # 层级 0 坐标中的 (x, y)
    level=1,  # 金字塔层级
    size=(512, 512)  # 宽高（像素）
)

# 返回 numpy 数组
```

### 金字塔层级选择

全切片图像以多分辨率金字塔存储。根据所需放大倍数选择合适的层级：

```python
# 检查可用层级
print(wsi.level_dimensions)  # [(宽度0, 高度0), (宽度1, 高度1), ...]
print(wsi.level_downsamples)  # [1.0, 4.0, 16.0, ...]

# 在较低分辨率加载以加速处理
wsi.generate_tiles(level=2, tile_size=256)  # 使用层级 2（16 倍降采样）
```

**常见金字塔层级：**
- 层级 0：全分辨率（如 40 倍放大）
- 层级 1：4 倍降采样（如 10 倍放大）
- 层级 2：16 倍降采样（如 2.5 倍放大）
- 层级 3：64 倍降采样（缩略图）

### 缩略图加载

生成低分辨率缩略图用于可视化和质量控制：

```python
# 获取缩略图
thumbnail = wsi.get_thumbnail(size=(1024, 1024))

# 使用 matplotlib 显示
import matplotlib.pyplot as plt
plt.imshow(thumbnail)
plt.axis('off')
plt.show()
```

## 使用 SlideDataset 批量加载

高效处理多个幻灯片：

```python
from pathml.core import SlideDataset
import glob

# 从多个幻灯片创建数据集
slide_paths = glob.glob("data/*.svs")
dataset = SlideDataset(
    slide_paths,
    tile_size=256,
    stride=256,
    level=0
)

# 遍历所有幻灯片的所有分块
for tile in dataset:
    image = tile.image
    slide_id = tile.slide_id
    # 处理分块...
```

**使用预处理流水线：**
```python
from pathml.preprocessing import Pipeline, StainNormalizationHE

# 创建流水线
pipeline = Pipeline([
    StainNormalizationHE(target='normalize')
])

# 应用于整个数据集
dataset = SlideDataset(slide_paths)
dataset.run(pipeline, distributed=True, n_workers=8)
```

## 元数据访问

提取幻灯片元数据，包括采集参数、放大倍数和供应商特定信息：

```python
# 访问元数据
metadata = wsi.metadata

# 常见元数据字段
print(metadata.get('openslide.objective-power'))  # 放大倍数
print(metadata.get('openslide.mpp-x'))  # X 轴每像素微米数
print(metadata.get('openslide.mpp-y'))  # Y 轴每像素微米数
print(metadata.get('openslide.vendor'))  # 扫描仪供应商

# 幻灯片尺寸
print(wsi.level_dimensions[0])  # 层级 0 的 (宽度, 高度)
```

## 处理 DICOM 幻灯片

PathML 通过专用处理支持 DICOM WSI：

```python
from pathml.core import SlideData, SlideType

# 加载 DICOM WSI
dicom_slide = SlideData.from_slide(
    "path/to/slide.dcm",
    backend=SlideType.DICOM
)

# DICOM 特定元数据
print(dicom_slide.metadata.get('PatientID'))
print(dicom_slide.metadata.get('StudyDate'))
```

## 处理 OME-TIFF

OME-TIFF 为多维成像提供开放标准：

```python
from pathml.core import SlideData

# 加载 OME-TIFF
ome_slide = SlideData.from_slide(
    "path/to/slide.ome.tiff",
    backend="bioformats"
)

# 访问多通道图像的通道信息
n_channels = ome_slide.shape[2]  # 通道数
```

## 性能考量

### 内存管理

对于大型 WSI 文件（通常 >1GB），使用基于分块的加载避免内存耗尽：

```python
# 高效：基于分块的处理
wsi.generate_tiles(level=1, tile_size=256)
for tile in wsi.tiles:
    process_tile(tile)  # 逐个处理分块

# 低效：将整个幻灯片加载到内存
full_image = wsi.read_region((0, 0), level=0, wsi.level_dimensions[0])  # 可能导致崩溃
```

### 分布式处理

使用 Dask 实现跨多工作节点的并行处理：

```python
from pathml.core import SlideDataset
from dask.distributed import Client

# 启动 Dask 客户端
client = Client(n_workers=8, threads_per_worker=2)

# 并行处理数据集
dataset = SlideDataset(slide_paths)
dataset.run(pipeline, distributed=True, client=client)
```

### 层级选择

通过选择适当的金字塔层级平衡分辨率和性能：

- **层级 0：** 用于需要最高细节的最终分析
- **层级 1-2：** 用于大多数预处理和模型训练
- **层级 3+：** 用于缩略图、质量控制和快速探索

## 常见问题与解决方案

**问题：幻灯片加载失败**
- 确认文件格式受支持
- 检查文件权限和路径
- 尝试不同后端：`backend="bioformats"` 或 `backend="openslide"`

**问题：内存不足错误**
- 使用基于分块的加载替代全幻灯片加载
- 在较低金字塔层级处理（如 level=1 或 level=2）
- 减小 tile_size 参数
- 启用 Dask 分布式处理

**问题：幻灯片间颜色不一致**
- 应用染色归一化预处理（参见 `preprocessing.md`）
- 检查扫描仪元数据中的校准信息
- 在预处理流水线中使用 `StainNormalizationHE` 变换

**问题：元数据缺失或不正确**
- 不同供应商将元数据存储在不同位置
- 使用 `wsi.metadata` 检查可用字段
- 某些格式可能元数据支持有限

## 最佳实践

1. **始终在处理前检查金字塔结构**：通过 `level_dimensions` 和 `level_downsamples` 了解可用分辨率

2. **使用适当的金字塔层级**：大多数任务在层级 1-2 处理；保留层级 0 用于最终高分辨率分析

3. **分割任务使用重叠分块**：设置 stride < tile_size 避免边缘伪影

4. **验证放大倍数一致性**：组合不同来源幻灯片时检查 `openslide.objective-power` 元数据

5. **处理供应商特定格式**：对多参数数据使用专用幻灯片类（CODEXSlide、VectraSlide）

6. **实施质量控制**：处理前生成缩略图检查伪影

7. **大型数据集使用分布式处理**：利用 Dask 跨多工作节点并行处理

## 示例工作流程

### 加载并检查新幻灯片

```python
from pathml.core import SlideData
import matplotlib.pyplot as plt

# 加载幻灯片
wsi = SlideData.from_slide("path/to/slide.svs")

# 检查属性
print(f"尺寸：{wsi.level_dimensions}")
print(f"降采样率：{wsi.level_downsamples}")
print(f"放大倍数：{wsi.metadata.get('openslide.objective-power')}")

# 生成缩略图进行质量控制
thumbnail = wsi.get_thumbnail(size=(1024, 1024))
plt.imshow(thumbnail)
plt.title(f"幻灯片：{wsi.name}")
plt.axis('off')
plt.show()
```

### 处理多个幻灯片

```python
from pathml.core import SlideDataset
from pathml.preprocessing import Pipeline, TissueDetectionHE
import glob

# 查找所有幻灯片
slide_paths = glob.glob("data/slides/*.svs")

# 创建流水线
pipeline = Pipeline([TissueDetectionHE()])

# 处理所有幻灯片
dataset = SlideDataset(
    slide_paths,
    tile_size=512,
    stride=512,
    level=1
)

# 分布式运行流水线
dataset.run(pipeline, distributed=True, n_workers=8)

# 保存处理后的数据
dataset.to_hdf5("processed_dataset.h5")
```

### 加载 CODEX 多参数数据

```python
from pathml.core import CODEXSlide
from pathml.preprocessing import Pipeline, CollapseRunsCODEX, SegmentMIF

# 加载 CODEX 幻灯片
codex = CODEXSlide("path/to/codex_dir", stain="IF")

# 创建 CODEX 专用流水线
pipeline = Pipeline([
    CollapseRunsCODEX(z_slice=2),  # 选择 z 切片
    SegmentMIF(
        nuclear_channel='DAPI',
        cytoplasm_channel='CD45',
        model='mesmer'
    )
])

# 处理
pipeline.run(codex)
```

## 附加资源

- **PathML 文档：** https://pathml.readthed
