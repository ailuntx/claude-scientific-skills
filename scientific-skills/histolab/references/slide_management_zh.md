# 切片管理

## 概述

`Slide` 类是 histolab 中处理全切片图像（WSI）的主要接口，提供加载、检查和处理各种格式的大型组织病理学图像的方法。

## 初始化

```python
from histolab.slide import Slide

# 使用 WSI 文件路径和输出目录初始化切片
slide = Slide(processed_path="path/to/processed/output",
              slide_path="path/to/slide.svs")
```

**参数说明：**
- `slide_path`：全切片图像文件路径（支持多种格式：SVS、TIFF、NDPI 等）
- `processed_path`：处理结果（切片区块、缩略图等）的保存目录

## 加载样本数据

Histolab 提供来自 TCGA 的内置样本数据集用于测试和演示：

```python
from histolab.data import prostate_tissue, ovarian_tissue, breast_tissue, heart_tissue, kidney_tissue

# 加载前列腺组织样本
prostate_svs, prostate_path = prostate_tissue()
slide = Slide(prostate_path, processed_path="output/")
```

可用样本数据集：
- `prostate_tissue()`：前列腺组织样本
- `ovarian_tissue()`：卵巢组织样本
- `breast_tissue()`：乳腺组织样本
- `heart_tissue()`：心脏组织样本
- `kidney_tissue()`：肾脏组织样本

## 关键属性

### 切片尺寸
```python
# 获取层级0（最高分辨率）的切片尺寸
width, height = slide.dimensions

# 获取特定金字塔层级的尺寸
level_dimensions = slide.level_dimensions
# 返回各层级的（宽度, 高度）元组
```

### 放大倍率信息
```python
# 获取基础放大倍率（如40x, 20x）
base_mag = slide.base_mpp  # 层级0的每像素微米值

# 获取所有可用层级
num_levels = slide.levels  # 金字塔层级数量
```

### 切片属性
```python
# 访问 OpenSlide 属性字典
properties = slide.properties

# 常用属性包括：
# - slide.properties['openslide.objective-power']：物镜倍率
# - slide.properties['openslide.mpp-x']：X轴每像素微米值
# - slide.properties['openslide.mpp-y']：Y轴每像素微米值
# - slide.properties['openslide.vendor']：扫描仪厂商
```

## 缩略图生成

```python
# 获取指定尺寸的缩略图
thumbnail = slide.thumbnail

# 保存缩略图到磁盘
slide.save_thumbnail()  # 保存至 processed_path 目录

# 获取缩放后的缩略图
scaled_thumbnail = slide.scaled_image(scale_factor=32)
```

## 切片可视化

```python
# 使用 matplotlib 显示切片缩略图
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 10))
plt.imshow(slide.thumbnail)
plt.title(f"切片名称: {slide.name}")
plt.axis('off')
plt.show()
```

## 区域提取

```python
# 在指定坐标和层级提取区域
region = slide.extract_region(
    location=(x, y),  # 层级0的左上角坐标
    size=(width, height),  # 区域尺寸
    level=0  # 金字塔层级
)
```

## 金字塔层级操作

WSI 文件采用多分辨率金字塔结构：
- 层级0：最高分辨率（原始扫描分辨率）
- 层级1+：逐级降低分辨率以实现快速访问

```python
# 检查可用层级
for level in range(slide.levels):
    dims = slide.level_dimensions[level]
    downsample = slide.level_downsamples[level]
    print(f"层级 {level}: {dims}, 降采样率: {downsample}x")
```

## 切片名称与路径

```python
# 获取不含扩展名的切片文件名
slide_name = slide.name

# 获取切片文件完整路径
slide_path = slide.scaled_image
```

## 最佳实践

1. **始终指定 processed_path**：在专用目录中组织输出文件
2. **处理前检查尺寸**：大型切片可能超出内存限制
3. **使用合适金字塔层级**：在匹配分析分辨率层级提取切片区块
4. **通过缩略图预览**：在资源密集型处理前使用缩略图快速可视化
5. **监控内存使用**：大型切片层级0操作需要大量内存

## 常见工作流

### 切片检查工作流
```python
from histolab.slide import Slide

# 加载切片
slide = Slide("slide.svs", processed_path="output/")

# 检查属性
print(f"尺寸: {slide.dimensions}")
print(f"层级数: {slide.levels}")
print(f"放大倍率: {slide.properties.get('openslide.objective-power', 'N/A')}")

# 保存缩略图供检查
slide.save_thumbnail()
```

### 多切片批处理
```python
import os
from pathlib import Path

slide_dir = Path("slides/")
output_dir = Path("processed/")

for slide_path in slide_dir.glob("*.svs"):
    slide = Slide(slide_path, processed_path=output_dir / slide_path.stem)
    # 处理每个切片
    print(f"正在处理: {slide.name}")
```
