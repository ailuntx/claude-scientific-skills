# 过滤器和预处理

## 概述

Histolab 提供了一套全面的过滤器，用于预处理整张切片图像和图块。这些过滤器可应用于图像以实现可视化、质量控制、组织检测和伪影去除。它们具有可组合性，能够链接起来构建复杂的预处理流程。

## 过滤器分类

### 图像过滤器
色彩空间转换、阈值处理和强度调整

### 形态学过滤器
膨胀、腐蚀、开运算和闭运算等结构操作

### 组合过滤器
用于合并多个过滤器的实用工具

## 图像过滤器

### RgbToGrayscale

将 RGB 图像转换为灰度图像。

```python
from histolab.filters.image_filters import RgbToGrayscale

gray_filter = RgbToGrayscale()
gray_image = gray_filter(rgb_image)
```

**应用场景：**
- 基于强度操作的前处理
- 简化色彩复杂度
- 形态学操作的输入

### RgbToHsv

将 RGB 转换为 HSV（色相、饱和度、明度）色彩空间。

```python
from histolab.filters.image_filters import RgbToHsv

hsv_filter = RgbToHsv()
hsv_image = hsv_filter(rgb_image)
```

**应用场景：**
- 基于颜色的组织分割
- 通过色相检测笔迹标记
- 分离彩色与无彩色内容

### RgbToHed

将 RGB 转换为 HED（苏木精-伊红-DAB）色彩空间以实现染色分离。

```python
from histolab.filters.image_filters import RgbToHed

hed_filter = RgbToHed()
hed_image = hed_filter(rgb_image)
```

**应用场景：**
- 分离 H&E 染色成分
- 分析细胞核（苏木精）与细胞质（伊红）染色
- 量化染色强度

### OtsuThreshold

应用大津自动阈值法创建二值图像。

```python
from histolab.filters.image_filters import OtsuThreshold

otsu_filter = OtsuThreshold()
binary_image = otsu_filter(grayscale_image)
```

**工作原理：**
- 自动确定最佳阈值
- 分离前景与背景
- 最小化类内方差

**应用场景：**
- 组织检测
- 细胞核分割
- 二值掩模创建

### AdaptiveThreshold

应用自适应阈值处理以应对局部强度变化。

```python
from histolab.filters.image_filters import AdaptiveThreshold

adaptive_filter = AdaptiveThreshold(
    block_size=11,      # 局部邻域大小
    offset=2            # 从均值中减去的常数
)
binary_image = adaptive_filter(grayscale_image)
```

**应用场景：**
- 非均匀光照
- 局部对比度增强
- 处理可变染色强度

### Invert

反转图像强度值。

```python
from histolab.filters.image_filters import Invert

invert_filter = Invert()
inverted_image = invert_filter(image)
```

**应用场景：**
- 特定分割算法的预处理
- 可视化调整

### StretchContrast

通过拉伸强度范围增强图像对比度。

```python
from histolab.filters.image_filters import StretchContrast

contrast_filter = StretchContrast()
enhanced_image = contrast_filter(image)
```

**应用场景：**
- 提升低对比度特征的可见性
- 可视化预处理
- 增强微弱结构

### HistogramEqualization

通过直方图均衡化增强对比度。

```python
from histolab.filters.image_filters import HistogramEqualization

hist_eq_filter = HistogramEqualization()
equalized_image = hist_eq_filter(grayscale_image)
```

**应用场景：**
- 标准化图像对比度
- 揭示隐藏细节
- 特征提取的预处理

## 形态学过滤器

### BinaryDilation

在二值图像中扩展白色区域。

```python
from histolab.filters.morphological_filters import BinaryDilation

dilation_filter = BinaryDilation(disk_size=5)
dilated_image = dilation_filter(binary_image)
```

**参数：**
- `disk_size`：结构元素大小（默认：5）

**应用场景：**
- 连接邻近组织区域
- 填充小间隙
- 扩展组织掩模

### BinaryErosion

在二值图像中收缩白色区域。

```python
from histolab.filters.morphological_filters import BinaryErosion

erosion_filter = BinaryErosion(disk_size=5)
eroded_image = erosion_filter(binary_image)
```

**应用场景：**
- 去除细小突起
- 分离连接对象
- 收缩组织边界

### BinaryOpening

先腐蚀后膨胀（去除小对象）。

```python
from histolab.filters.morphological_filters import BinaryOpening

opening_filter = BinaryOpening(disk_size=3)
opened_image = opening_filter(binary_image)
```

**应用场景：**
- 去除小伪影
- 平滑对象边界
- 降噪

### BinaryClosing

先膨胀后腐蚀（填充小孔洞）。

```python
from histolab.filters.morphological_filters import BinaryClosing

closing_filter = BinaryClosing(disk_size=5)
closed_image = closing_filter(binary_image)
```

**应用场景：**
- 填充组织区域的小孔洞
- 连接邻近对象
- 平滑内部边界

### RemoveSmallObjects

移除小于阈值的连通区域。

```python
from histolab.filters.morphological_filters import RemoveSmallObjects

remove_small_filter = RemoveSmallObjects(
    area_threshold=500  # 最小像素面积
)
cleaned_image = remove_small_filter(binary_image)
```

**应用场景：**
- 去除灰尘和伪影
- 过滤噪声
- 清理组织掩模

### RemoveSmallHoles

填充小于阈值的孔洞。

```python
from histolab.filters.morphological_filters import RemoveSmallHoles

fill_holes_filter = RemoveSmallHoles(
    area_threshold=1000  # 待填充孔洞的最大尺寸
)
filled_image = fill_holes_filter(binary_image)
```

**应用场景：**
- 填充组织中的小间隙
- 创建连续组织区域
- 去除内部伪影

## 过滤器组合

### 链式过滤器

按顺序组合多个过滤器：

```python
from histolab.filters.image_filters import RgbToGrayscale, OtsuThreshold
from histolab.filters.morphological_filters import BinaryDilation, RemoveSmallObjects
from histolab.filters.compositions import Compose

# 创建过滤器流程
tissue_detection_pipeline = Compose([
    RgbToGrayscale(),
    OtsuThreshold(),
    BinaryDilation(disk_size=5),
    RemoveSmallHoles(area_threshold=1000),
    RemoveSmallObjects(area_threshold=500)
])

# 应用流程
result = tissue_detection_pipeline(rgb_image)
```

### Lambda 过滤器

内联创建自定义过滤器：

```python
from histolab.filters.image_filters import Lambda
import numpy as np

# 自定义亮度调整
brightness_filter = Lambda(lambda img: np.clip(img * 1.2, 0, 255).astype(np.uint8))

# 自定义颜色通道提取
red_channel_filter = Lambda(lambda img: img[:, :, 0])
```

## 常用预处理流程

### 标准组织检测

```python
from histolab.filters.compositions import Compose
from histolab.filters.image_filters import RgbToGrayscale, OtsuThreshold
from histolab.filters.morphological_filters import (
    BinaryDilation, RemoveSmallHoles, RemoveSmallObjects
)

tissue_detection = Compose([
    RgbToGrayscale(),
    OtsuThreshold(),
    BinaryDilation(disk_size=5),
    RemoveSmallHoles(area_threshold=1000),
    RemoveSmallObjects(area_threshold=500)
])
```

### 笔迹去除

```python
from histolab.filters.image_filters import RgbToHsv, Lambda
import numpy as np

def remove_pen_marks(hsv_image):
    """去除蓝/绿色笔迹标记"""
    h, s, v = hsv_image[:, :, 0], hsv_image[:, :, 1], hsv_image[:, :, 2]
    # 蓝/绿色相掩模（常见笔迹颜色）
    pen_mask = ((h > 0.45) & (h < 0.7) & (s > 0.3))
    # 将笔迹区域设为白色
    hsv_image[pen_mask] = [0, 0, 1]
    return hsv_image

pen_removal = Compose([
    RgbToHsv(),
    Lambda(remove_pen_marks)
])
```

### 细胞核增强

```python
from histolab.filters.image_filters import RgbToHed, HistogramEqualization
from histolab.filters.compositions import Compose

nuclei_enhancement = Compose([
    RgbToHed(),
    Lambda(lambda hed: hed[:, :, 0]),  # 提取苏木精通道
    HistogramEqualization()
])
```

### 对比度标准化

```python
from histolab.filters.image_filters import StretchContrast, HistogramEqualization

contrast_normalization = Compose([
    RgbToGrayscale(),
    StretchContrast(),
    HistogramEqualization()
])
```

## 对图块应用过滤器

过滤器可应用于单个图块：

```python
from histolab.tile import Tile
from histolab.filters.image_filters import RgbToGrayscale

# 加载或提取图块
tile = Tile(image=pil_image, coords=(x, y))

# 应用过滤器
gray_filter = RgbToGrayscale()
filtered_tile = tile.apply_filters(gray_filter)

# 链式多过滤器
from histolab.filters.compositions import Compose
from histolab.filters.image_filters import StretchContrast

filter_chain = Compose([
    RgbToGrayscale(),
    StretchContrast()
])
processed_tile = tile.apply_filters(filter_chain)
```

## 自定义掩模过滤器

将自定义过滤器与组织掩模集成：

```python
from histolab.masks import TissueMask
from histolab.filters.compositions import Compose
from histolab.filters.image_filters import RgbToGrayscale, OtsuThreshold
from histolab.filters.morphological_filters import BinaryDilation

# 自定义激进组织检测
aggressive_filters = Compose([
    RgbToGrayscale(),
    OtsuThreshold(),
    BinaryDilation(disk_size=10),  # 更大膨胀
    RemoveSmallObjects(area_threshold=5000)  # 仅移除大伪影
])

# 使用自定义过滤器创建掩模
custom_mask = TissueMask(filters=aggressive_filters)
```

## 染色标准化

虽然 histolab 未内置染色标准化功能，但可通过过滤器实现基础标准化：

```python
from histolab.filters.image_filters import RgbToHed, Lambda
import numpy as np

def normalize_hed(hed_image, target_means=[0.65, 0.70], target_stds=[0.15, 0.13]):
    """简易 H&E 标准化"""
    h_channel = hed_image[:, :, 0]
    e_channel = hed_image[:, :, 1]

    # 标准化苏木精
    h_normalized = (h_channel - h_channel.mean()) / h_channel.std()
    h_normalized = h_normalized * target_stds[0] + target_means[0]

    # 标准化伊红
    e_normalized = (e_channel - e_channel.mean()) / e_channel.std()
    e_normalized = e_normalized * target_stds[1] + target_means[1]

    hed_image[:, :, 0] = h_normalized
    hed_image[:, :, 1] = e_normalized

    return hed_image

normalization_pipeline = Compose([
    RgbToHed(),
    Lambda(normalize_hed)
    # 如需可转回 RGB
])
```

## 最佳实践

1. **预览过滤器**：在应用到图块前通过缩略图可视化输出
2. **高效链式组合**：按逻辑顺序排列过滤器（如先色彩转换后阈值处理）
3. **参数调优**：针对特定组织调整阈值和结构元素尺寸
4. **使用组合**：通过 `Compose` 构建可复用流程
5. **考虑性能**：复杂过滤器链会增加处理时间
6. **多样化验证**：在不同扫描仪、染色剂和组织类型上测试
7. **记录自定义过滤器**：明确描述自定义流程的目的和参数

## 质量控制过滤器

### 模糊检测

```python
from histolab.filters.image_filters import Lambda
import cv2
import numpy as np

def laplacian_blur_score(gray_image):
    """计算拉普拉斯方差（模糊度指标）"""
    return cv2.Laplacian(np.array(gray_image), cv2.CV_64F).var()

blur_detector = Lambda(lambda img: laplacian_blur_score(
    RgbToGrayscale()(img)
))
```

### 组织覆盖率

```python
from histolab.filters.image_filters import RgbToGrayscale, OtsuThreshold
from histolab.filters.compositions import Compose

def tissue_coverage(image):
    """计算图像中组织占比"""
    tissue_mask = Compose([
        RgbToGrayscale(),
        OtsuThreshold()
    ])(image)
    return tissue_mask.sum() / tissue_mask.size * 100

coverage_filter = Lambda(tissue_coverage)
```

## 故障排除

### 问题：组织检测遗漏有效组织
**解决方案：**
- 降低 `RemoveSmallObjects` 的 `area_threshold`
- 减小腐蚀/开运算的 disk_size
- 尝试用自适应阈值替代大津法

### 问题：包含过多伪影
**解决方案：**
- 提高 `RemoveSmallObjects` 的 `area_threshold`
- 添加开/闭运算
- 对特定伪影使用基于颜色的自定义过滤

### 问题：组织边界过于粗糙
**解决方案：**
- 添加 `BinaryClosing` 或 `BinaryOpening` 进行平滑
- 调整形态学操作的 disk_size

### 问题：染色质量不稳定
**解决方案：**
- 应用直方图均衡化
- 使用自适应阈值
- 实施染色标准化流程
