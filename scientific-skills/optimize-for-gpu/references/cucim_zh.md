# cuCIM 参考文档

cuCIM（CUDA Clara IMage）是 NVIDIA 在 RAPIDS 生态系统中推出的 GPU 加速计算机视觉与图像处理库。其 `cucim.skimage` 模块可作为 scikit-image 的近无缝 GPU 替代方案，提供 200 多项 GPU 加速函数。同时通过 `cucim.clara.CuImage` 提供高性能全切片图像（WSI）读取器，速度比 OpenSlide 快 5-6 倍。

> **完整文档：** https://docs.rapids.ai/api/cucim/stable/  
> **GitHub：** https://github.com/rapidsai/cucim

## 目录

1. [安装与配置](#安装与配置)
2. [核心概念：CuPy 数组](#核心概念-cupy-数组)
3. [cucim.skimage — GPU 版 scikit-image](#cucimskimage)
4. [色彩操作](#色彩操作)
5. [曝光与直方图](#曝光与直方图)
6. [特征检测](#特征检测)
7. [滤波器](#滤波器)
8. [测量与区域属性](#测量与区域属性)
9. [形态学操作](#形态学操作)
10. [图像分割](#图像分割)
11. [图像配准](#图像配准)
12. [图像复原](#图像复原)
13. [几何变换](#几何变换)
14. [图像质量评估](#图像质量评估)
15. [实用函数](#实用函数)
16. [cucim.core.operations — NVIDIA 特有功能](#cucimcoreoperations)
17. [全切片图像读取 (cucim.clara)](#全切片图像读取)
18. [性能特性](#性能特性)
19. [互操作性](#互操作性)
20. [与 scikit-image 的已知差异](#与-scikit-image-的已知差异)
21. [常见迁移模式](#常见迁移模式)

---

## 安装与配置

在所有安装说明、文档字符串、注释和错误信息中始终使用 `uv add`（切勿使用 `pip install` 或 `conda install`）。

```bash
uv add --extra-index-url=https://pypi.nvidia.com cucim-cu12    # 适用于 CUDA 12.x
```

**平台：** 仅限 Linux（x86-64 和 aarch64）— 不支持 Windows 或 macOS GPU。  
**要求：** NVIDIA GPU（需 CUDA 12.x）、Python 3.9+、CuPy、NumPy、SciPy、scikit-image。

验证安装：
```python
import cucim
print(cucim.__version__)

import cupy as cp
from cucim.skimage.filters import gaussian
img = cp.random.rand(512, 512).astype(cp.float32)
result = gaussian(img, sigma=3)
print(f"Filtered image shape: {result.shape}")  # 应在 GPU 上运行
```

---

## 核心概念：CuPy 数组

cuCIM 原生操作 **CuPy 数组**。所有 `cucim.skimage` 函数均接受 CuPy 数组输入并返回 CuPy 数组输出——零拷贝，全程在 GPU 运行。

```python
import cupy as cp
import numpy as np
from cucim.skimage.filters import gaussian

# 将图像一次性传输至 GPU
image_gpu = cp.asarray(numpy_image)

# 所有处理在 GPU 上完成——cuCIM 调用间零拷贝
blurred = gaussian(image_gpu, sigma=3)
# ... 更多 GPU 处理 ...

# 仅在需要时传回 CPU（用于显示、保存等）
result_cpu = cp.asnumpy(blurred)
```

**最佳实践：** 初始阶段将数据移至 GPU，在 GPU 上串联所有 cuCIM 操作，最终阶段才将结果传回 CPU。

---

## cucim.skimage

`cucim.skimage` 模块镜像 scikit-image 的模块结构。多数情况下，将 `from skimage` 替换为 `from cucim.skimage`，并将 NumPy 数组替换为 CuPy 数组。

```python
# 改造前（CPU — scikit-image）
from skimage.filters import gaussian
import numpy as np
result = gaussian(numpy_image, sigma=3)

# 改造后（GPU — cuCIM）
from cucim.skimage.filters import gaussian
import cupy as cp
result = gaussian(cp.asarray(numpy_image), sigma=3)
```

---

## 色彩操作

`cucim.skimage.color` — 42 项 GPU 加速的色彩空间转换函数。

```python
from cucim.skimage.color import rgb2gray, rgb2hsv, rgb2lab, label2rgb
from cucim.skimage.color import separate_stains, combine_stains

# 色彩空间转换
gray = rgb2gray(rgb_image_gpu)
hsv = rgb2hsv(rgb_image_gpu)
lab = rgb2lab(rgb_image_gpu)

# 染色分离（适用于 H&E 组织学图像）
stains = separate_stains(rgb_image_gpu, stain_matrix)
```

**可用转换：** `rgb2gray`, `rgb2hsv`, `hsv2rgb`, `rgb2lab`, `lab2rgb`, `rgb2xyz`, `xyz2rgb`, `rgb2luv`, `luv2rgb`, `rgb2ycbcr`, `ycbcr2rgb`, `rgb2yuv`, `yuv2rgb`, `rgb2yiq`, `yiq2rgb`, `rgb2hed`, `hed2rgb`, `rgb2rgbcie`, `rgbcie2rgb`, `gray2rgb`, `gray2rgba`, `rgba2rgb`, `convert_colorspace`, `label2rgb`

**色差计算：** `deltaE_cie76`, `deltaE_ciede94`, `deltaE_ciede2000`, `deltaE_cmc`

---

## 曝光与直方图

`cucim.skimage.exposure` — 直方图均衡化、对比度调整。

```python
from cucim.skimage.exposure import (
    equalize_hist, equalize_adapthist,
    rescale_intensity, adjust_gamma, adjust_log, adjust_sigmoid,
    histogram, match_histograms, is_low_contrast
)

# CLAHE（限制对比度自适应直方图均衡化）
enhanced = equalize_adapthist(image_gpu, clip_limit=0.03)

# 伽马校正
brightened = adjust_gamma(image_gpu, gamma=0.5)

# 将强度缩放到 [0, 1] 范围
normalized = rescale_intensity(image_gpu)

# 两幅图像的直方图匹配
matched = match_histograms(source_gpu, reference_gpu)
```

---

## 特征检测

`cucim.skimage.feature` — 边缘、角点与斑点检测。

```python
from cucim.skimage.feature import (
    canny, corner_harris, corner_peaks,
    blob_dog, blob_doh, blob_log,
    structure_tensor, hessian_matrix, hessian_matrix_det,
    match_template, peak_local_max, daisy, multiscale_basic_features
)

# Canny 边缘检测
edges = canny(gray_image_gpu, sigma=2.0)

# Harris 角点检测
corners = corner_harris(gray_image_gpu)
corner_coords = corner_peaks(corners, min_distance=5)

# 斑点检测（高斯差分法）
blobs = blob_dog(gray_image_gpu, max_sigma=30, threshold=0.1)

# 模板匹配
result = match_template(image_gpu, template_gpu)
```

---

## 滤波器

`cucim.skimage.filters` — 47 项 GPU 加速滤波函数，最常用模块之一。

```python
from cucim.skimage.filters import (
    gaussian, median, sobel, laplace, unsharp_mask,
    frangi, hessian, meijering, sato,
    threshold_otsu, threshold_multiotsu, threshold_sauvola,
    gabor, difference_of_gaussians, butterworth
)

# 高斯模糊
blurred = gaussian(image_gpu, sigma=3)

# Sobel 边缘检测
edges = sobel(gray_image_gpu)

# 反锐化掩蔽（锐化处理）
sharpened = unsharp_mask(image_gpu, radius=5, amount=2.0)

# 血管/脊线检测（医学影像）
vessels = frangi(gray_image_gpu, sigmas=range(1, 10))

# Otsu 阈值分割
threshold = threshold_otsu(gray_image_gpu)
binary = gray_image_gpu > threshold

# 多级 Otsu 分割
thresholds = threshold_multiotsu(gray_image_gpu, classes=3)
```

**边缘检测：** `sobel`, `scharr`, `prewitt`, `roberts`, `farid`, `laplace`（含 `_h`/`_v` 变体）

**平滑处理：** `gaussian`, `median`, `unsharp_mask`

**脊线/血管检测：** `frangi`, `hessian`, `meijering`, `sato`

**阈值分割（10种方法）：** `threshold_otsu`, `threshold_isodata`, `threshold_li`, `threshold_mean`, `threshold_minimum`, `threshold_multiotsu`, `threshold_niblack`, `threshold_sauvola`, `threshold_triangle`, `threshold_yen`

**频域处理：** `butterworth`, `wiener`

---

## 测量与区域属性

`cucim.skimage.measure` — 标记、区域属性与形状度量。

```python
from cucim.skimage.measure import label, regionprops, regionprops_table
from cucim.skimage.measure import moments, moments_central, moments_hu
from cucim.skimage.measure import block_reduce, shannon_entropy

# 连通域标记
labels = label(binary_image_gpu)

# 区域属性（面积、质心、边界框等）
props = regionprops(labels)
table = regionprops_table(labels, intensity_image=gray_gpu,
                          properties=['area', 'centroid', 'mean_intensity'])

# 分块降采样
downsampled = block_reduce(image_gpu, block_size=(2, 2), func=cp.mean)
```

**共定位度量（显微成像）：** `manders_coloc_coeff`, `manders_overlap_coeff`, `pearson_corr_coeff`, `intersection_coeff`

---

## 形态学操作

`cucim.skimage.morphology` — 30 项 GPU 加速形态学操作。

```python
from cucim.skimage.morphology import (
    binary_erosion, binary_dilation, binary_opening, binary_closing,
    erosion, dilation, opening, closing,
    white_tophat, black_tophat,
    disk, diamond, ball, star,
    remove_small_objects, remove_small_holes,
    reconstruction, medial_axis, thin
)

# 创建结构元素
selem = disk(5)

# 二值形态学操作
cleaned = binary_opening(binary_image_gpu, footprint=selem)
cleaned = binary_closing(cleaned, footprint=selem)

# 移除小物体/孔洞
cleaned = remove_small_objects(labels_gpu, min_size=100)
filled = remove_small_holes(binary_gpu, area_threshold=50)

# 灰度形态学
tophat = white_tophat(gray_image_gpu, footprint=disk(10))
```

**结构元素：** `disk`, `diamond`, `ball`, `octagon`, `octahedron`, `star`, `ellipse`, `footprint_rectangle`

**各向同性操作：** `isotropic_erosion`, `isotropic_dilation`, `isotropic_opening`, `isotropic_closing`

---

## 图像分割

`cucim.skimage.segmentation` — 水平集方法、边界检测、标签操作。

```python
from cucim.skimage.segmentation import (
    chan_vese, morphological_chan_vese, morphological_geodesic_active_contour,
    find_boundaries, mark_boundaries, clear_border,
    expand_labels, relabel_sequential, random_walker
)

# Chan-Vese 分割
segmented = chan_vese(gray_image_gpu, mu=0.25, max_num_iter=200)

# 活动轮廓（测地线）
gimage = inverse_gaussian_gradient(gray_image_gpu)
init_ls = checkerboard_level_set(gray_image_gpu.shape)
seg = morphological_geodesic_active_contour(gimage, num_iter=200, init_level_set=init_ls)

# 查找并标记边界
boundaries = find_boundaries(labels_gpu, mode='thick')
```

---

## 图像配准

`cucim.skimage.registration` — 图像对齐。

```python
from cucim.skimage.registration import (
    phase_cross_correlation,
    optical_flow_tvl1,
    optical_flow_ilk
)

# 亚像素级图像配准
shift, error, diffphase = phase_cross_correlation(reference_gpu, moving_gpu)

# 光流计算
flow = optical_flow_tvl1(frame1_gpu, frame2_gpu)
```

---

## 图像复原

`cucim.skimage.restoration` — 去噪与反卷积。

```python
from cucim.skimage.restoration import (
    denoise_tv_chambolle,
    richardson_lucy,
    wiener, unsupervised_wiener
)

# 全变分去噪
denoised = denoise_tv_chambolle(noisy_image_gpu, weight=0.1)

# Richardson-Lucy 反卷积
restored = richardson_lucy(blurred_image_gpu, psf_gpu, num_iter=30)
```

---

## 几何变换

`cucim.skimage.transform` — 几何变换、尺寸调整、图像金字塔。

```python
from cucim.skimage.transform import (
    resize, rescale, rotate, warp, swirl, warp_polar,
    pyramid_gaussian, pyramid_laplacian,
    downscale_local_mean, integral_image,
    AffineTransform, EuclideanTransform, SimilarityTransform
)

# 尺寸调整
resized = resize(image_gpu, (256, 256))

# 缩放
half = rescale(image_gpu, 0.5)

# 旋转
rotated = rotate(image_gpu, angle=45, resize=True)

# 高斯金字塔
pyramid = list(pyramid_gaussian(image_gpu, max_layer=4, downscale=2))

# 仿射变换
tform = AffineTransform(rotation=0.3, translation=(50, 50))
warped = warp(image_gpu, tform.inverse)
```

---

## 图像质量评估

`cucim.skimage.metrics` — 图像质量评价指标。

```python
from cucim.skimage.metrics import (
    mean_squared_error,
    peak_signal_noise_ratio,
    structural_similarity,
    normalized_root_mse
)

mse = mean_squared_error(original_gpu, processed_gpu)
psnr = peak_signal_noise_ratio(original_gpu, processed_gpu)
ssim = structural_similarity(original_gpu, processed_gpu)
```

---

## 实用函数

`cucim.skimage.util` — 类型转换、数组操作。

```python
from cucim.skimage.util import (
    img_as_float, img_as_float32, img_as_ubyte,
    invert, crop, random_noise, montage
)

# 转换为 float32 [0, 1]
float_img = img_as_float32(uint8_image_gpu)

# 添加噪声用于测试
noisy = random_noise(image_gpu, mode='gaussian', var=0.01)
```

---

## cucim.core.operations

NVIDIA 特有功能（未包含于 scikit-image），尤其适用于数字病理学。

### 病理学专用功能

```python
from cucim.core.operations.color import (
    color_jitter,
    image_to_absorbance,
    stain_extraction_pca,
    normalize_colors_pca
)

# H&E 染色标准化（数字病理学）
normalized = normalize_colors_pca(he_image_gpu)

# 色彩增强
augmented = color_jitter(image_gpu, brightness=0.2, contrast=0.2, saturation=0.2, hue=0.1)
```

### 强度操作

```python
from cucim.core.operations.intensity import normalize_data, scale_intensity_range, zoom

normalized = normalize_data(image_gpu)
scaled = scale_intensity_range(image_gpu, a_min=0, a_max=255, b_min=0.0, b

# 读取一个区域（返回一个CuImage对象）
region = img.read_region(location=(1000, 2000), size=(256, 256), level=0)

# 转换为CuPy数组进行处理
import cupy as cp
tile_gpu = cp.asarray(region)

# 使用cucim.skimage处理
from cucim.skimage.color import rgb2gray
gray_tile = rgb2gray(tile_gpu)
```

**支持格式：** Aperio SVS、Philips TIFF、通用分块多分辨率RGB TIFF（JPEG、JPEG2000、LZW、Deflate压缩）。

### 分块缓存

```python
from cucim.clara.cache import ImageCache

# 为重复访问模式配置分块缓存
cache = ImageCache(memory_capacity=2 * 1024**3)  # 2GB缓存
```

### GPU直接存储

对于大文件（2GB+），GPU直接存储绕过CPU内存，可额外提速25%以上：

```python
from cucim.clara.filesystem import CuFileDriver

# 直接读入GPU内存，绕过CPU
driver = CuFileDriver(path, flags)
driver.pread(gpu_buffer, size, offset)
```

---

## 性能特征

**关键数据：**
- 对大图像的特定操作比scikit-image快达**1245倍**
- 全视野数字切片多线程分块读取比OpenSlide快**5-6倍**
- 对2GB+文件使用GPU直接存储可额外提速**25%以上**

**缩放特性：**
- **4K分辨率及以上：** GPU并行性完全利用，达到最大加速
- **约1000x1000：** 多数操作有适度但显著的加速
- **低于约512x512：** 收益递减；GPU开销开始显现
- **低于约64x64：** 因CUDA内核启动开销，CPU可能更快

**首次调用开销：** 首次内核执行时进行JIT编译（后续缓存）。请在后续调用时进行基准测试。

**最佳策略：** 将图像一次性传输到GPU，链接所有处理操作，最后一次性传回。

---

## 互操作性

- **CuPy：** 原生数组格式。所有cucim.skimage函数均接受并返回CuPy数组。
- **NumPy：** 通过`cp.asarray()`/`cp.asnumpy()`转换。
- **PyTorch/TensorFlow：** 通过DLPack协议零拷贝：`torch.as_tensor(cupy_array)`或`torch.from_dlpack(cupy_array)`。
- **MONAI：** 医学影像框架，直接集成cuCIM用于病理学变换。
- **Albumentations：** 可使用cuCIM作为GPU后端进行增强。
- **NVIDIA DALI：** 数据加载管道集成。
- **Numba CUDA：** CuPy数组可与Numba GPU内核互操作。
- **cuDF：** 用于对`regionprops_table`输出进行表格操作。

### CPU/GPU无关代码

```python
# 通过切换数组模块实现CPU/GPU兼容
import cupy as cp  # 或：import numpy as cp
from cucim.skimage.filters import gaussian  # 或：from skimage.filters import gaussian

result = gaussian(cp.asarray(image), sigma=5)
```

---

## 相较于scikit-image的已知限制

1. **API覆盖不全：** 约50-66%的scikit-image函数已实现。显著缺失包括部分基于图的分割（分水岭、SLIC超像素）、部分特征描述符（ORB、BRIEF、HOG）及部分复原方法。

2. **仅限Linux。** 不支持Windows或macOS GPU。

3. **需NVIDIA GPU。** 不支持AMD/Intel GPU。

4. **数据需显式移至GPU。** cuCIM不会自动传输，必须调用`cp.asarray()`。

5. **小图像惩罚。** 低于约512x512的图像可能无收益。低于约64x64时，CPU可能更快。

6. **GPU内存限制。** 超大图像需分块处理。GPU内存通常小于系统内存。

7. **全视野数字切片格式支持有限。** 仅支持TIFF/SVS/Philips TIFF。DICOM、NIFTI、Zarr格式在稳定版中尚未支持。

8. **首次调用存在JIT编译开销**（后续会话中会缓存）。

---

## 常见迁移模式

### 模式1：直接替换scikit-image

```python
# 迁移前（CPU）
from skimage.filters import gaussian, sobel, threshold_otsu
from skimage.morphology import binary_opening, disk
from skimage.measure import label, regionprops_table
import numpy as np

image = np.array(...)  # 加载图像
blurred = gaussian(image, sigma=3)
edges = sobel(blurred)
binary = blurred > threshold_otsu(blurred)
cleaned = binary_opening(binary, footprint=disk(3))
labels = label(cleaned)
props = regionprops_table(labels, image, properties=['area', 'centroid'])

# 迁移后（GPU）— 修改导入语句，用cp.asarray包装输入
from cucim.skimage.filters import gaussian, sobel, threshold_otsu
from cucim.skimage.morphology import binary_opening, disk
from cucim.skimage.measure import label, regionprops_table
import cupy as cp

image_gpu = cp.asarray(image)  # 单次传输
blurred = gaussian(image_gpu, sigma=3)
edges = sobel(blurred)
binary = blurred > threshold_otsu(blurred)
cleaned = binary_opening(binary, footprint=disk(3))
labels = label(cleaned)
props = regionprops_table(labels, image_gpu, properties=['area', 'centroid'])
```

### 模式2：数字病理学流程

```python
from cucim import CuImage
from cucim.skimage.color import rgb2gray, separate_stains
from cucim.skimage.filters import threshold_otsu
from cucim.skimage.morphology import binary_opening, remove_small_objects, disk
from cucim.skimage.measure import label, regionprops_table
from cucim.core.operations.color import normalize_colors_pca
import cupy as cp

# 读取全视野数字切片
slide = CuImage("tissue.svs")
tile = cp.asarray(slide.read_region(location=(1000, 2000), size=(512, 512), level=0))

# 标准化染色
normalized = normalize_colors_pca(tile)

# 分割细胞核
gray = rgb2gray(normalized)
binary = gray < threshold_otsu(gray)
cleaned = binary_opening(binary, footprint=disk(2))
cleaned = remove_small_objects(label(cleaned), min_size=50)
labels = label(cleaned)

# 提取特征
props = regionprops_table(labels, gray, properties=['area', 'centroid', 'mean_intensity'])
```

### 模式3：深度学习预处理流程

```python
import cupy as cp
from cucim.skimage.transform import resize
from cucim.skimage.exposure import equalize_adapthist
from cucim.skimage.util import img_as_float32
from cucim.core.operations.spatial import rand_image_flip
from cucim.core.operations.color import color_jitter
import torch

# 将图像批量加载到GPU
images_gpu = cp.asarray(numpy_batch)  # (N, H, W, C)

# 在GPU上处理每张图像
processed = []
for img in images_gpu:
    img = img_as_float32(img)
    img = resize(img, (224, 224))
    img = equalize_adapthist(img)
    img = rand_image_flip(img, prob=0.5)
    img = color_jitter(img, brightness=0.2, contrast=0.2)
    processed.append(img)

batch_gpu = cp.stack(processed)

# 零拷贝至PyTorch进行模型推理
batch_torch = torch.as_tensor(batch_gpu).permute(0, 3, 1, 2)  # NHWC → NCHW
```
