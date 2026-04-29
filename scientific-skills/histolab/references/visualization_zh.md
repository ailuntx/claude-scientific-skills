# 可视化

## 概述

Histolab 提供多种内置可视化方法，用于检查玻片、预览分块位置、可视化掩膜及评估提取质量。恰当的可视化对于验证预处理流程、调试提取问题和展示结果至关重要。

## 玻片可视化

### 缩略图展示

```python
from histolab.slide import Slide
import matplotlib.pyplot as plt

slide = Slide("slide.svs", processed_path="output/")

# 展示缩略图
plt.figure(figsize=(10, 10))
plt.imshow(slide.thumbnail)
plt.title(f"玻片: {slide.name}")
plt.axis('off')
plt.show()
```

### 保存缩略图至磁盘

```python
# 将缩略图保存为图像文件
slide.save_thumbnail()
# 保存路径：processed_path/thumbnails/slide_name_thumb.png
```

### 缩放图像

```python
# 获取特定下采样倍率的玻片缩放图像
scaled_img = slide.scaled_image(scale_factor=32)

plt.imshow(scaled_img)
plt.title(f"32倍下采样玻片")
plt.show()
```

## 掩膜可视化

### 使用 locate_mask()

```python
from histolab.masks import TissueMask, BiggestTissueBoxMask

# 可视化组织掩膜
tissue_mask = TissueMask()
slide.locate_mask(tissue_mask)

# 可视化最大组织框掩膜
biggest_mask = BiggestTissueBoxMask()
slide.locate_mask(biggest_mask)
```

此操作将在玻片缩略图上以红色叠加显示掩膜边界。

### 手动掩膜可视化

```python
import matplotlib.pyplot as plt
from histolab.masks import TissueMask

slide = Slide("slide.svs", processed_path="output/")
mask = TissueMask()

# 生成掩膜
mask_array = mask(slide)

# 创建并排对比图
fig, axes = plt.subplots(1, 3, figsize=(20, 7))

# 原始缩略图
axes[0].imshow(slide.thumbnail)
axes[0].set_title("原始玻片")
axes[0].axis('off')

# 二值掩膜
axes[1].imshow(mask_array, cmap='gray')
axes[1].set_title("组织掩膜")
axes[1].axis('off')

# 缩略图叠加掩膜
from matplotlib.colors import ListedColormap
overlay = slide.thumbnail.copy()
axes[2].imshow(overlay)
axes[2].imshow(mask_array, cmap=ListedColormap(['none', 'red']), alpha=0.3)
axes[2].set_title("掩膜叠加")
axes[2].axis('off')

plt.tight_layout()
plt.show()
```

### 多掩膜对比

```python
from histolab.masks import TissueMask, BiggestTissueBoxMask

masks = {
    'TissueMask': TissueMask(),
    'BiggestTissueBoxMask': BiggestTissueBoxMask()
}

fig, axes = plt.subplots(1, len(masks) + 1, figsize=(20, 6))

# 原始图像
axes[0].imshow(slide.thumbnail)
axes[0].set_title("原始图像")
axes[0].axis('off')

# 各掩膜展示
for idx, (name, mask) in enumerate(masks.items(), 1):
    mask_array = mask(slide)
    axes[idx].imshow(mask_array, cmap='gray')
    axes[idx].set_title(name)
    axes[idx].axis('off')

plt.tight_layout()
plt.show()
```

## 分块位置预览

### 使用 locate_tiles()

提取前预览分块位置：

```python
from histolab.tiler import RandomTiler, GridTiler, ScoreTiler
from histolab.scorer import NucleiScorer

# RandomTiler 预览
random_tiler = RandomTiler(
    tile_size=(512, 512),
    n_tiles=50,
    level=0,
    seed=42
)
random_tiler.locate_tiles(slide, n_tiles=20)

# GridTiler 预览
grid_tiler = GridTiler(
    tile_size=(512, 512),
    level=0
)
grid_tiler.locate_tiles(slide)

# ScoreTiler 预览
score_tiler = ScoreTiler(
    tile_size=(512, 512),
    n_tiles=30,
    scorer=NucleiScorer()
)
score_tiler.locate_tiles(slide, n_tiles=15)
```

此操作将在玻片缩略图上以彩色矩形标记待提取分块位置。

### 自定义分块位置可视化

```python
import matplotlib.pyplot as plt
import matplotlib.patches as patches
from histolab.tiler import RandomTiler

slide = Slide("slide.svs", processed_path="output/")
tiler = RandomTiler(tile_size=(512, 512), n_tiles=30, seed=42)

# 获取缩略图及缩放因子
thumbnail = slide.thumbnail
scale_factor = slide.dimensions[0] / thumbnail.size[0]

# 生成分块坐标（不实际提取）
fig, ax = plt.subplots(figsize=(12, 12))
ax.imshow(thumbnail)
ax.set_title("分块位置预览")
ax.axis('off')

# 手动添加每个分块位置的矩形
# 注：此为概念演示 - 实际实现需从分块器中获取坐标
tile_coords = []  # 应由分块器逻辑填充
for coord in tile_coords:
    x, y = coord[0] / scale_factor, coord[1] / scale_factor
    w, h = 512 / scale_factor, 512 / scale_factor
    rect = patches.Rectangle((x, y), w, h,
                             linewidth=2, edgecolor='red',
                             facecolor='none')
    ax.add_patch(rect)

plt.show()
```

## 分块可视化

### 展示提取的分块

```python
from pathlib import Path
from PIL import Image
import matplotlib.pyplot as plt

tile_dir = Path("output/tiles/")
tile_paths = list(tile_dir.glob("*.png"))[:16]  # 前16个分块

fig, axes = plt.subplots(4, 4, figsize=(12, 12))
axes = axes.ravel()

for idx, tile_path in enumerate(tile_paths):
    tile_img = Image.open(tile_path)
    axes[idx].imshow(tile_img)
    axes[idx].set_title(tile_path.stem, fontsize=8)
    axes[idx].axis('off')

plt.tight_layout()
plt.show()
```

### 分块网格拼图

```python
def create_tile_mosaic(tile_dir, grid_size=(4, 4)):
    """创建分块拼图"""
    tile_paths = list(Path(tile_dir).glob("*.png"))[:grid_size[0] * grid_size[1]]

    fig, axes = plt.subplots(grid_size[0], grid_size[1], figsize=(16, 16))

    for idx, tile_path in enumerate(tile_paths):
        row = idx // grid_size[1]
        col = idx % grid_size[1]
        tile_img = Image.open(tile_path)
        axes[row, col].imshow(tile_img)
        axes[row, col].axis('off')

    plt.tight_layout()
    plt.savefig("tile_mosaic.png", dpi=150, bbox_inches='tight')
    plt.show()

create_tile_mosaic("output/tiles/", grid_size=(5, 5))
```

### 带组织掩膜的分块展示

```python
from histolab.tile import Tile
import matplotlib.pyplot as plt

# 假设已有分块对象
tile = Tile(image=pil_image, coords=(x, y))

# 计算组织掩膜
tile.calculate_tissue_mask()

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# 原始分块
axes[0].imshow(tile.image)
axes[0].set_title("原始分块")
axes[0].axis('off')

# 组织掩膜
axes[1].imshow(tile.tissue_mask, cmap='gray')
axes[1].set_title(f"组织掩膜 (组织占比: {tile.tissue_ratio:.1%})")
axes[1].axis('off')

# 叠加效果
axes[2].imshow(tile.image)
axes[2].imshow(tile.tissue_mask, cmap='Reds', alpha=0.3)
axes[2].set_title("叠加效果")
axes[2].axis('off')

plt.tight_layout()
plt.show()
```

## 质量评估可视化

### 分块分数分布

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# 从 ScoreTiler 加载分块报告
report_df = pd.read_csv("tiles_report.csv")

# 分数分布直方图
plt.figure(figsize=(10, 6))
plt.hist(report_df['score'], bins=30, edgecolor='black', alpha=0.7)
plt.xlabel('分块分数')
plt.ylabel('频次')
plt.title('分块分数分布')
plt.grid(axis='y', alpha=0.3)
plt.show()

# 分数与组织占比散点图
plt.figure(figsize=(10, 6))
plt.scatter(report_df['tissue_percent'], report_df['score'], alpha=0.5)
plt.xlabel('组织百分比')
plt.ylabel('分块分数')
plt.title('分块分数与组织覆盖率关系')
plt.grid(alpha=0.3)
plt.show()
```

### 最高分与最低分分块对比

```python
import pandas as pd
from PIL import Image
import matplotlib.pyplot as plt

# 加载分块报告
report_df = pd.read_csv("tiles_report.csv")
report_df = report_df.sort_values('score', ascending=False)

# 最高8个分块
top_tiles = report_df.head(8)
# 最低8个分块
bottom_tiles = report_df.tail(8)

fig, axes = plt.subplots(2, 8, figsize=(20, 6))

# 展示高分分块
for idx, (_, row) in enumerate(top_tiles.iterrows()):
    tile_img = Image.open(f"output/tiles/{row['tile_name']}")
    axes[0, idx].imshow(tile_img)
    axes[0, idx].set_title(f"分数: {row['score']:.3f}", fontsize=8)
    axes[0, idx].axis('off')

# 展示低分分块
for idx, (_, row) in enumerate(bottom_tiles.iterrows()):
    tile_img = Image.open(f"output/tiles/{row['tile_name']}")
    axes[1, idx].imshow(tile_img)
    axes[1, idx].set_title(f"分数: {row['score']:.3f}", fontsize=8)
    axes[1, idx].axis('off')

axes[0, 0].set_ylabel('高分分块', fontsize=12)
axes[1, 0].set_ylabel('低分分块', fontsize=12)

plt.tight_layout()
plt.savefig("score_comparison.png", dpi=150, bbox_inches='tight')
plt.show()
```

## 多玻片可视化

### 玻片集合缩略图

```python
from pathlib import Path
from histolab.slide import Slide
import matplotlib.pyplot as plt

slide_dir = Path("slides/")
slide_paths = list(slide_dir.glob("*.svs"))[:9]

fig, axes = plt.subplots(3, 3, figsize=(15, 15))
axes = axes.ravel()

for idx, slide_path in enumerate(slide_paths):
    slide = Slide(slide_path, processed_path="output/")
    axes[idx].imshow(slide.thumbnail)
    axes[idx].set_title(slide.name, fontsize=10)
    axes[idx].axis('off')

plt.tight_layout()
plt.savefig("slide_collection.png", dpi=150, bbox_inches='tight')
plt.show()
```

### 组织覆盖率对比

```python
from pathlib import Path
from histolab.slide import Slide
from histolab.masks import TissueMask
import matplotlib.pyplot as plt
import numpy as np

slide_paths = list(Path("slides/").glob("*.svs"))
tissue_percentages = []
slide_names = []

for slide_path in slide_paths:
    slide = Slide(slide_path, processed_path="output/")
    mask = TissueMask()(slide)
    tissue_pct = mask.sum() / mask.size * 100
    tissue_percentages.append(tissue_pct)
    slide_names.append(slide.name)

# 柱状图
plt.figure(figsize=(12, 6))
plt.bar(range(len(slide_names)), tissue_percentages)
plt.xticks(range(len(slide_names)), slide_names, rotation=45, ha='right')
plt.ylabel('组织覆盖率 (%)')
plt.title('不同玻片的组织覆盖率对比')
plt.grid(axis='y', alpha=0.3)
plt.tight_layout()
plt.show()
```

## 滤镜效果可视化

### 处理前后对比

```python
from histolab.filters.image_filters import RgbToGrayscale, HistogramEqualization
from histolab.filters.compositions import Compose

# 定义滤镜流程
filter_pipeline = Compose([
    RgbToGrayscale(),
    HistogramEqualization()
])

# 原始与处理后对比
fig, axes = plt.subplots(1, 2, figsize=(12, 6))

axes[0].imshow(slide.thumbnail)
axes[0].set_title("原始图像")
axes[0].axis('off')

filtered = filter_pipeline(slide.thumbnail)
axes[1].imshow(filtered, cmap='gray')
axes[1].set_title("滤镜处理后")
axes[1].axis('off')

plt.tight_layout()
plt.show()
```

### 多步骤滤镜可视化

```python
from histolab.filters.image_filters import RgbToGrayscale, OtsuThreshold
from histolab.filters.morphological_filters import BinaryDilation, RemoveSmallObjects

# 分步滤镜处理
steps = [
    ("原始图像", None),
    ("灰度化", RgbToGrayscale()),
    ("Otsu阈值", Compose([RgbToGrayscale(), OtsuThreshold()])),
    ("膨胀处理", Compose([RgbToGrayscale(), OtsuThreshold(), BinaryDilation(disk_size=5)])),
    ("移除小对象", Compose([RgbToGrayscale(), OtsuThreshold(), BinaryDilation(disk_size=5), RemoveSmallObjects(area_threshold=500)]))
]

fig, axes = plt.subplots(1, len(steps), figsize=(20, 4))

for idx, (title, filter_fn) in enumerate(steps):
    if filter_fn is None:
        axes[idx].imshow(slide.thumbnail)
    else:
        result = filter_fn(slide.thumbnail)
        axes[idx].imshow(result, cmap='gray')
    axes[idx].set_title(title, fontsize=10)
    axes[idx].axis('off')

plt.tight_layout()
plt.show()
```

## 可视化导出

### 高分辨率导出

```python
# 导出高分辨率图像
fig, ax = plt.subplots(figsize=(20, 20))
ax.imshow(slide.thumbnail)
ax.axis('off')
plt.savefig("slide_high_res.png", dpi=300, bbox_inches='tight', pad_inches=0)
plt.close()
```

### PDF报告生成

```python
from matplotlib.backends.backend_pdf import PdfPages

# 创建多页PDF报告
with PdfPages('slide_report.pdf') as pdf:
    # 第1页：玻片缩略图
    fig1, ax1 = plt.subplots(figsize=(10, 10))
    ax1.imshow(slide.thumbnail)
    ax1.set_title(f"玻片: {slide.name}")
    ax1.axis('off')
    pdf.savefig(fig1, bbox_inches='tight')
    plt.close()

    # 第2页：组织掩膜
    fig2, ax2 = plt.subplots(figsize=(10, 10))
    mask = TissueMask()(slide)
    ax2.imshow(mask, cmap='gray')
    ax2.set_title("组织掩膜")
    ax2.axis('off')
    pdf.savefig(fig2, bbox_inches='tight')
    plt.close()

    # 第

6. **恰当使用颜色映射**：二值掩码用 'gray'，热力图用 'viridis'  
7. **创建可复用的可视化函数**：实现跨项目的报告标准化
