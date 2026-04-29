---
name: histolab
description: 轻量级 WSI 切片提取与预处理工具。适用于基础玻片处理、组织检测、切片提取及 H&E 图像染色归一化。最适合简单流程、数据集准备和快速切片分析。如需高级空间蛋白质组学、多重成像或深度学习流程，请使用 pathml。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# Histolab

## 概述

Histolab 是用于处理数字病理学中全切片图像（WSI）的 Python 库。它自动化组织检测，从十亿像素级图像中提取信息切片，并为深度学习流程准备数据集。该库支持多种 WSI 格式，实现复杂的组织分割，并提供灵活的切片提取策略。

## 安装

```bash
uv pip install histolab
```

## 快速入门

从全切片图像提取切片的基础工作流：

```python
from histolab.slide import Slide
from histolab.tiler import RandomTiler

# 加载玻片
slide = Slide("slide.svs", processed_path="output/")

# 配置切片器
tiler = RandomTiler(
    tile_size=(512, 512),
    n_tiles=100,
    level=0,
    seed=42
)

# 预览切片位置
tiler.locate_tiles(slide, n_tiles=20)

# 提取切片
tiler.extract(slide)
```

## 核心功能

### 1. 玻片管理

加载、检查和处理多种格式的全切片图像。

**常用操作：**
- 加载 WSI 文件（SVS, TIFF, NDPI 等）
- 访问玻片元数据（尺寸、放大倍数、属性）
- 生成可视化缩略图
- 处理金字塔图像结构
- 提取特定坐标区域

**核心类：** `Slide`

**参考文档：** `references/slide_management.md` 包含完整文档：
- 玻片初始化与配置
- 内置样本数据集（前列腺、卵巢、乳腺、心脏、肾脏组织）
- 访问玻片属性与元数据
- 缩略图生成与可视化
- 处理金字塔层级
- 多玻片处理工作流

**示例工作流：**
```python
from histolab.slide import Slide
from histolab.data import prostate_tissue

# 加载样本数据
prostate_svs, prostate_path = prostate_tissue()

# 初始化玻片
slide = Slide(prostate_path, processed_path="output/")

# 检查属性
print(f"尺寸: {slide.dimensions}")
print(f"层级: {slide.levels}")
print(f"放大倍数: {slide.properties.get('openslide.objective-power')}")

# 保存缩略图
slide.save_thumbnail()
```

### 2. 组织检测与掩膜

自动识别组织区域并过滤背景/伪影。

**常用操作：**
- 创建二值组织掩膜
- 检测最大组织区域
- 排除背景与伪影
- 自定义组织分割
- 去除笔迹标注

**核心类：** `TissueMask`, `BiggestTissueBoxMask`, `BinaryMask`

**参考文档：** `references/tissue_masks.md` 包含完整文档：
- TissueMask：使用自动过滤器分割所有组织区域
- BiggestTissueBoxMask：返回最大组织区域边界框（默认）
- BinaryMask：自定义掩膜基类
- 使用 `locate_mask()` 可视化掩膜
- 创建自定义矩形及标注排除掩膜
- 掩膜与切片提取的集成
- 最佳实践与故障排除

**示例工作流：**
```python
from histolab.masks import TissueMask, BiggestTissueBoxMask

# 创建全组织区域掩膜
tissue_mask = TissueMask()

# 在玻片上可视化掩膜
slide.locate_mask(tissue_mask)

# 获取掩膜数组
mask_array = tissue_mask(slide)

# 使用最大组织区域（多数提取器默认）
biggest_mask = BiggestTissueBoxMask()
```

**掩膜选用场景：**
- `TissueMask`：多组织区域，综合分析
- `BiggestTissueBoxMask`：单一主组织区域，排除伪影（默认）
- 自定义 `BinaryMask`：特定 ROI，排除标注，自定义分割

### 3. 切片提取

使用不同策略从大型 WSI 提取小区域。

**三种提取策略：**

**RandomTiler：** 提取固定数量的随机位置切片
- 最佳场景：多样化区域采样、探索性分析、训练数据
- 关键参数：`n_tiles`, `seed`（确保可复现性）

**GridTiler：** 以网格模式系统化提取组织切片
- 最佳场景：完整覆盖、空间分析、图像重建
- 关键参数：`pixel_overlap`（滑动窗口重叠）

**ScoreTiler：** 基于评分函数提取最优切片
- 最佳场景：信息最密集区域、质量驱动选择
- 关键参数：`scorer`（NucleiScorer, CellularityScorer, 自定义）

**通用参数：**
- `tile_size`：切片尺寸（如 (512, 512)）
- `level`：提取的金字塔层级（0=最高分辨率）
- `check_tissue`：按组织内容过滤切片
- `tissue_percent`：最小组织覆盖率（默认 80%）
- `extraction_mask`：定义提取区域的掩膜

**参考文档：** `references/tile_extraction.md` 包含完整文档：
- 各提取策略详解
- 可用评分器（NucleiScorer, CellularityScorer, 自定义）
- 使用 `locate_tiles()` 预览切片
- 提取工作流与报告
- 高级模式（多层级、分层提取）
- 性能优化与故障排除

**示例工作流：**

```python
from histolab.tiler import RandomTiler, GridTiler, ScoreTiler
from histolab.scorer import NucleiScorer

# 随机采样（快速、多样化）
random_tiler = RandomTiler(
    tile_size=(512, 512),
    n_tiles=100,
    level=0,
    seed=42,
    check_tissue=True,
    tissue_percent=80.0
)
random_tiler.extract(slide)

# 网格覆盖（全面性）
grid_tiler = GridTiler(
    tile_size=(512, 512),
    level=0,
    pixel_overlap=0,
    check_tissue=True
)
grid_tiler.extract(slide)

# 评分驱动选择（信息最密集）
score_tiler = ScoreTiler(
    tile_size=(512, 512),
    n_tiles=50,
    scorer=NucleiScorer(),
    level=0
)
score_tiler.extract(slide, report_path="tiles_report.csv")
```

**提取前务必预览：**
```python
# 在缩略图上预览切片位置
tiler.locate_tiles(slide, n_tiles=20)
```

### 4. 过滤器与预处理

应用图像处理过滤器进行组织检测、质量控制与预处理。

**过滤器分类：**

**图像过滤器：** 色彩空间转换、阈值处理、对比度增强
- `RgbToGrayscale`, `RgbToHsv`, `RgbToHed`
- `OtsuThreshold`, `AdaptiveThreshold`
- `StretchContrast`, `HistogramEqualization`

**形态学过滤器：** 二值图像结构操作
- `BinaryDilation`, `BinaryErosion`
- `BinaryOpening`, `BinaryClosing`
- `RemoveSmallObjects`, `RemoveSmallHoles`

**组合：** 串联多个过滤器
- `Compose`：创建过滤器流水线

**参考文档：** `references/filters_preprocessing.md` 包含完整文档：
- 各类过滤器详解
- 过滤器组合与串联
- 常用预处理流水线（组织检测、笔迹去除、细胞核增强）
- 对切片应用过滤器
- 自定义掩膜过滤器
- 质量控制过滤器（模糊检测、组织覆盖率）
- 最佳实践与故障排除

**示例工作流：**

```python
from histolab.filters.compositions import Compose
from histolab.filters.image_filters import RgbToGrayscale, OtsuThreshold
from histolab.filters.morphological_filters import (
    BinaryDilation, RemoveSmallHoles, RemoveSmallObjects
)

# 标准组织检测流水线
tissue_detection = Compose([
    RgbToGrayscale(),
    OtsuThreshold(),
    BinaryDilation(disk_size=5),
    RemoveSmallHoles(area_threshold=1000),
    RemoveSmallObjects(area_threshold=500)
])

# 用于自定义掩膜
from histolab.masks import TissueMask
custom_mask = TissueMask(filters=tissue_detection)

# 对切片应用过滤器
from histolab.tile import Tile
filtered_tile = tile.apply_filters(tissue_detection)
```

### 5. 可视化

可视化玻片、掩膜、切片位置及提取质量。

**常用可视化任务：**
- 显示玻片缩略图
- 可视化组织掩膜
- 预览切片位置
- 评估切片质量
- 生成报告与图表

**参考文档：** `references/visualization.md` 包含完整文档：
- 玻片缩略图显示与保存
- 使用 `locate_mask()` 可视化掩膜
- 使用 `locate_tiles()` 预览切片位置
- 显示提取切片与拼接图
- 质量评估（分数分布、最优/最差切片）
- 多玻片可视化
- 过滤器效果可视化
- 导出高分辨率图表与 PDF 报告
- Jupyter notebook 交互式可视化

**示例工作流：**

```python
import matplotlib.pyplot as plt
from histolab.masks import TissueMask

# 显示玻片缩略图
plt.figure(figsize=(10, 10))
plt.imshow(slide.thumbnail)
plt.title(f"玻片: {slide.name}")
plt.axis('off')
plt.show()

# 可视化组织掩膜
tissue_mask = TissueMask()
slide.locate_mask(tissue_mask)

# 预览切片位置
tiler = RandomTiler(tile_size=(512, 512), n_tiles=50)
tiler.locate_tiles(slide, n_tiles=20)

# 以网格显示提取切片
from pathlib import Path
from PIL import Image

tile_paths = list(Path("output/tiles/").glob("*.png"))[:16]
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

## 典型工作流

### 工作流 1：探索性切片提取

快速采样多样化组织区域进行初步分析。

```python
from histolab.slide import Slide
from histolab.tiler import RandomTiler
import logging

# 启用日志记录跟踪进度
logging.basicConfig(level=logging.INFO)

# 加载玻片
slide = Slide("slide.svs", processed_path="output/random_tiles/")

# 检查玻片
print(f"尺寸: {slide.dimensions}")
print(f"层级: {slide.levels}")
slide.save_thumbnail()

# 配置随机切片器
random_tiler = RandomTiler(
    tile_size=(512, 512),
    n_tiles=100,
    level=0,
    seed=42,
    check_tissue=True,
    tissue_percent=80.0
)

# 预览位置
random_tiler.locate_tiles(slide, n_tiles=20)

# 提取切片
random_tiler.extract(slide)
```

### 工作流 2：全面网格提取

完整组织覆盖的全玻片分析。

```python
from histolab.slide import Slide
from histolab.tiler import GridTiler
from histolab.masks import TissueMask

# 加载玻片
slide = Slide("slide.svs", processed_path="output/grid_tiles/")

# 使用 TissueMask 处理多组织区域
tissue_mask = TissueMask()
slide.locate_mask(tissue_mask)

# 配置网格切片器
grid_tiler = GridTiler(
    tile_size=(512, 512),
    level=1,  # 使用层级 1 加速提取
    pixel_overlap=0,
    check_tissue=True,
    tissue_percent=70.0
)

# 预览网格
grid_tiler.locate_tiles(slide)

# 提取全部切片
grid_tiler.extract(slide, extraction_mask=tissue_mask)
```

### 工作流 3：质量驱动切片选择

基于细胞核密度提取信息最密集切片。

```python
from histolab.slide import Slide
from histolab.tiler import ScoreTiler
from histolab.scorer import NucleiScorer
import pandas as pd
import matplotlib.pyplot as plt

# 加载玻片
slide = Slide("slide.svs", processed_path="output/scored_tiles/")

# 配置评分切片器
score_tiler = ScoreTiler(
    tile_size=(512, 512),
    n_tiles=50,
    level=0,
    scorer=NucleiScorer(),
    check_tissue=True
)

# 预览最优切片
score_tiler.locate_tiles(slide, n_tiles=15)

# 提取并生成报告
score_tiler.extract(slide, report_path="tiles_report.csv")

# 分析分数
report_df = pd.read_csv("tiles_report.csv")
plt.hist(report_df['score'], bins=20, edgecolor='black')
plt.xlabel('切片分数')
plt.ylabel('频次')
plt.title('切片分数分布')
plt.show()
```

### 工作流 4：多玻片处理流水线

使用统一参数处理整个玻片库。

```python
from pathlib import Path
from histolab.slide import Slide
from histolab.tiler import RandomTiler
import logging

logging.basicConfig(level=logging.INFO)

# 一次性配置切片器
tiler = RandomTiler(
    tile_size=(512, 512),
    n_tiles=50,
    level=0,
    seed=42,
    check_tissue=True
)

# 处理所有玻片
slide_dir = Path("slides/")
output_base = Path("output/")

for slide_path in slide_dir.glob("*.svs"):
    print(f"\n处理中: {slide_path.name}")

    # 创建玻片专属输出目录
    output_dir = output_base / slide_path.stem
    output_dir.mkdir(parents=True, exist_ok=True)

    # 加载并处理玻片
    slide = Slide(slide_path, processed_path=output_dir)

    # 保存缩略图供复查
    slide.save_thumbnail()

    # 提取切片
    tiler.extract(slide)

    print(f"已完成: {slide_path.name}")
```

### 工作流 5：自定义组织检测与过滤

处理含伪影、标注或特殊染色的玻片。

```python
from histolab.slide import Slide
from histolab.masks import TissueMask
from histolab.tiler import RandomTiler
from histolab.filters.compositions import Compose
from histolab.filters.image_filters import RgbToGrayscale, OtsuThreshold

### 全切片分析
- 使用 GridTiler 实现完整组织覆盖
- 在多个金字塔层级提取数据以进行分层分析
- 通过网格坐标保持空间关系
- 采用 `pixel_overlap` 实现滑动窗口处理

### 组织表征
- 使用 RandomTiler 采样多样化区域
- 通过掩膜量化组织覆盖率
- 利用 HED 分解提取染色特异性信息
- 跨切片比较组织模式

### 质量评估
- 使用 ScoreTiler 识别最佳聚焦区域
- 通过自定义掩膜和过滤器检测伪影
- 评估整批切片的染色质量
- 标记问题切片供人工复核

### 数据集构建
- 使用 ScoreTiler 优先选择信息量丰富的切片
- 按组织百分比筛选切片
- 生成包含切片评分和元数据的报告
- 创建跨切片和组织类型的分层数据集

## 故障排除

### 未提取到切片
- 降低 `tissue_percent` 阈值
- 确认切片包含组织（检查缩略图）
- 确保 extraction_mask 覆盖组织区域
- 检查 tile_size 是否符合切片分辨率

### 背景切片过多
- 启用 `check_tissue=True`
- 提高 `tissue_percent` 阈值
- 选用合适掩膜（TissueMask 对比 BiggestTissueBoxMask）
- 自定义掩膜过滤器以优化组织检测

### 提取速度过慢
- 在较低金字塔层级提取（level=1 或 2）
- 减少 RandomTiler/ScoreTiler 的 `n_tiles` 数量
- 使用 RandomTiler 替代 GridTiler 进行采样
- 采用 BiggestTissueBoxMask 替代 TissueMask

### 切片存在伪影
- 实施自定义注释排除掩膜
- 调整伪影去除的过滤器参数
- 提高小物体移除阈值
- 应用提取后质量过滤

### 跨切片结果不一致
- 为 RandomTiler 设置相同随机种子
- 使用预处理过滤器标准化染色
- 根据染色质量调整 `tissue_percent`
- 实施切片特异性掩膜定制

## 资源文档

本技能包含 `references/` 目录下的详细参考文档：

### references/slide_management.md
全切片图像加载、检查与处理的综合指南：
- 切片初始化与配置
- 内置样本数据集
- 切片属性与元数据
- 缩略图生成与可视化
- 金字塔层级操作
- 多切片处理流程
- 最佳实践与常见模式

### references/tissue_masks.md
组织检测与掩膜技术完整文档：
- TissueMask、BiggestTissueBoxMask、BinaryMask 类
- 组织检测过滤器原理
- 使用过滤器链定制掩膜
- 掩膜可视化
- 创建自定义矩形及注释排除掩膜
- 与切片提取的集成
- 最佳实践与故障排除

### references/tile_extraction.md
切片提取策略详解：
- RandomTiler、GridTiler、ScoreTiler 对比
- 可用评分器（NucleiScorer、CellularityScorer、自定义）
- 通用及策略特定参数
- 通过 locate_tiles() 预览切片位置
- 提取流程与 CSV 报告
- 高级模式（多层级、分层提取）
- 性能优化
- 常见问题排查

### references/filters_preprocessing.md
完整过滤器参考与预处理指南：
- 图像过滤器（色彩转换、阈值分割、对比度）
- 形态学过滤器（膨胀、腐蚀、开闭运算）
- 过滤器组合与链式调用
- 常用预处理流程
- 在切片上应用过滤器
- 自定义掩膜过滤器
- 质量控制过滤器
- 最佳实践与故障排除

### references/visualization.md
综合可视化指南：
- 切片缩略图显示与保存
- 掩膜可视化技术
- 切片位置预览
- 显示提取切片并创建马赛克图
- 质量评估可视化
- 多切片对比
- 过滤器效果可视化
- 导出高分辨率图表与 PDF
- Jupyter 笔记本交互式可视化

**使用模式：** 参考文件包含支持本主技能文档工作流的深度信息。可根据需要加载特定参考文件获取详细实施指南、故障排除或高级功能说明。
