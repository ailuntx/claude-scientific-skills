# 组织掩膜

## 概述

组织掩膜是用于识别全切片图像中组织区域的二值化表示。在提取图块时，它们对于过滤背景、伪影和非组织区域至关重要。Histolab 提供了多种掩膜类以满足不同的组织分割需求。

## 掩膜类别

### BinaryMask

**用途：** 创建自定义二值掩膜的通用基类。

```python
from histolab.masks import BinaryMask

class CustomMask(BinaryMask):
    def _mask(self, obj):
        # 实现自定义掩膜逻辑
        # 返回二值化 numpy 数组
        pass
```

**适用场景：**
- 自定义组织分割算法
- 特定区域分析（如排除标注区域）
- 与外部分割模型集成

### TissueMask

**用途：** 使用自动化过滤器分割切片中的所有组织区域。

```python
from histolab.masks import TissueMask

# 创建组织掩膜
tissue_mask = TissueMask()

# 应用于切片
mask_array = tissue_mask(slide)
```

**工作原理：**
1. 将图像转换为灰度
2. 应用大津阈值法分离组织与背景
3. 执行二值膨胀以连接邻近组织区域
4. 移除组织区域内的小孔洞
5. 过滤小物体（伪影）

**返回：** 二值化 NumPy 数组，其中：
- `True` (或 1)：组织像素
- `False` (或 0)：背景像素

**最佳适用场景：**
- 包含多个独立组织区域的切片
- 全面的组织分析
- 当所有组织区域都重要时

### BiggestTissueBoxMask（默认）

**用途：** 识别并返回最大连通组织区域的边界框。

```python
from histolab.masks import BiggestTissueBoxMask

# 创建最大组织区域掩膜
biggest_mask = BiggestTissueBoxMask()

# 应用于切片
mask_array = biggest_mask(slide)
```

**工作原理：**
1. 应用与 TissueMask 相同的过滤流程
2. 识别所有连通组织成分
3. 选择最大连通成分
4. 返回包含该区域的边界框

**最佳适用场景：**
- 包含单个主要组织区域的切片
- 排除小伪影或组织碎片
- 聚焦主要组织区域（多数分块器的默认选择）

## 使用过滤器自定义掩膜

掩膜可接受自定义过滤器链实现专业组织检测：

```python
from histolab.masks import TissueMask
from histolab.filters.image_filters import RgbToGrayscale, OtsuThreshold
from histolab.filters.morphological_filters import BinaryDilation, RemoveSmallHoles

# 定义自定义过滤器组合
custom_mask = TissueMask(
    filters=[
        RgbToGrayscale(),
        OtsuThreshold(),
        BinaryDilation(disk_size=5),
        RemoveSmallHoles(area_threshold=500)
    ]
)
```

## 可视化掩膜

### 使用 locate_mask()

```python
from histolab.slide import Slide
from histolab.masks import TissueMask

slide = Slide("slide.svs", processed_path="output/")
mask = TissueMask()

# 在缩略图上可视化掩膜边界
slide.locate_mask(mask)
```

该方法会显示切片缩略图，并以对比色叠加掩膜边界。

### 手动可视化

```python
import matplotlib.pyplot as plt
from histolab.masks import TissueMask

slide = Slide("slide.svs", processed_path="output/")
tissue_mask = TissueMask()

# 生成掩膜
mask_array = tissue_mask(slide)

# 并排绘制
fig, axes = plt.subplots(1, 2, figsize=(15, 7))

axes[0].imshow(slide.thumbnail)
axes[0].set_title("原始切片")
axes[0].axis('off')

axes[1].imshow(mask_array, cmap='gray')
axes[1].set_title("组织掩膜")
axes[1].axis('off')

plt.show()
```

## 创建自定义矩形掩膜

定义特定关注区域：

```python
from histolab.masks import BinaryMask
import numpy as np

class RectangularMask(BinaryMask):
    def __init__(self, x_start, y_start, width, height):
        self.x_start = x_start
        self.y_start = y_start
        self.width = width
        self.height = height

    def _mask(self, obj):
        # 创建指定矩形区域的掩膜
        thumb = obj.thumbnail
        mask = np.zeros(thumb.shape[:2], dtype=bool)
        mask[self.y_start:self.y_start+self.height,
             self.x_start:self.x_start+self.width] = True
        return mask

# 使用自定义掩膜
roi_mask = RectangularMask(x_start=1000, y_start=500, width=2000, height=1500)
```

## 排除标注区域

病理切片常包含笔迹标记或数字标注，可通过自定义掩膜排除：

```python
from histolab.masks import TissueMask
from histolab.filters.image_filters import RgbToGrayscale, OtsuThreshold
from histolab.filters.morphological_filters import BinaryDilation

class AnnotationExclusionMask(BinaryMask):
    def _mask(self, obj):
        thumb = obj.thumbnail

        # 转换为HSV空间检测笔迹标记（通常为蓝/绿色）
        hsv = cv2.cvtColor(np.array(thumb), cv2.COLOR_RGB2HSV)

        # 定义笔迹标记的颜色范围
        lower_blue = np.array([100, 50, 50])
        upper_blue = np.array([130, 255, 255])

        # 创建排除笔迹的掩膜
        pen_mask = cv2.inRange(hsv, lower_blue, upper_blue)

        # 应用标准组织检测
        tissue_mask = TissueMask()(obj)

        # 组合：保留组织区域，排除笔迹标记
        final_mask = tissue_mask & ~pen_mask.astype(bool)

        return final_mask
```

## 与图块提取集成

通过 `extraction_mask` 参数实现掩膜与分块器的无缝集成：

```python
from histolab.tiler import RandomTiler
from histolab.masks import TissueMask, BiggestTissueBoxMask

# 使用 TissueMask 从所有组织区域提取
random_tiler = RandomTiler(
    tile_size=(512, 512),
    n_tiles=100,
    level=0,
    extraction_mask=TissueMask()  # 从所有组织区域提取
)

# 或使用默认的 BiggestTissueBoxMask
random_tiler = RandomTiler(
    tile_size=(512, 512),
    n_tiles=100,
    level=0,
    extraction_mask=BiggestTissueBoxMask()  # 默认行为
)
```

## 最佳实践

1. **提取前预览掩膜**：使用 `locate_mask()` 或手动可视化验证掩膜质量
2. **选择合适的掩膜类型**：多组织区域用 `TissueMask`，单一主区域用 `BiggestTissueBoxMask`
3. **针对特定染色定制**：不同染色（H&E、IHC）可能需要调整阈值参数
4. **处理伪影**：使用自定义过滤器或掩膜排除笔迹、气泡或褶皱
5. **在多样化切片上测试**：在不同质量和伪影的切片上验证掩膜性能
6. **考虑计算成本**：`TissueMask` 更全面但计算强度高于 `BiggestTissueBoxMask`

## 常见问题与解决方案

### 问题：掩膜包含过多背景
**解决方案：** 调整大津阈值或增加小物体移除阈值

### 问题：掩膜排除了有效组织
**解决方案：** 降低小物体移除阈值或修改膨胀参数

### 问题：存在多个组织区域但仅捕获最大区域
**解决方案：** 从 `BiggestTissueBoxMask` 切换至 `TissueMask`

### 问题：笔迹标注被包含在掩膜中
**解决方案：** 实现自定义标注排除掩膜（参考前文示例）
