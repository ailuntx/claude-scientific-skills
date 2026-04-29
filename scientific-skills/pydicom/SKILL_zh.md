---
name: pydicom
description: 用于处理DICOM（医学数字成像与通信）文件的Python库。当需要读取、写入或修改DICOM格式的医学影像数据，从医学图像（CT、MRI、X光、超声）中提取像素数据，对DICOM文件进行匿名化处理，操作DICOM元数据和标签，将DICOM图像转换为其他格式，处理压缩的DICOM数据，或处理医学影像数据集时使用此技能。适用于涉及医学图像分析、PACS系统、放射学工作流和医疗影像应用的任务。
license: https://github.com/pydicom/pydicom/blob/main/LICENSE
metadata:
    skill-author: K-Dense Inc.
---

# Pydicom

## 概述

Pydicom是一个纯Python包，用于处理医学影像数据的标准格式DICOM文件。本技能提供关于读取、写入和操作DICOM文件的指导，包括处理像素数据、元数据及各种压缩格式。

## 使用场景

在以下场景中使用本技能：
- 处理医学影像文件（CT、MRI、X光、超声、PET等）
- 需要提取或修改元数据的DICOM数据集
- 从医学扫描中提取像素数据并进行图像处理
- 为研究或数据共享进行DICOM匿名化
- 将DICOM文件转换为标准图像格式
- 需要解压缩的压缩DICOM数据
- DICOM序列和结构化报告
- 多切片体积重建
- PACS（影像归档与通信系统）集成

## 安装

安装pydicom及常用依赖：

```bash
uv pip install pydicom
uv pip install pillow  # 用于图像格式转换
uv pip install numpy   # 用于像素数组操作
uv pip install matplotlib  # 用于可视化
```

处理压缩DICOM文件可能需要额外安装：

```bash
uv pip install pylibjpeg pylibjpeg-libjpeg pylibjpeg-openjpeg  # JPEG压缩支持
uv pip install python-gdcm  # 替代压缩处理器
```

## 核心工作流

### 读取DICOM文件

使用`pydicom.dcmread()`读取DICOM文件：

```python
import pydicom

# 读取DICOM文件
ds = pydicom.dcmread('path/to/file.dcm')

# 访问元数据
print(f"患者姓名: {ds.PatientName}")
print(f"检查日期: {ds.StudyDate}")
print(f"影像模态: {ds.Modality}")

# 显示所有元素
print(ds)
```

**关键点：**
- `dcmread()`返回`Dataset`对象
- 使用属性表示法（如`ds.PatientName`）或标签表示法（如`ds[0x0010, 0x0010]`）访问数据元素
- 通过`ds.file_meta`访问传输语法UID等文件元数据
- 使用`getattr(ds, '属性名', 默认值)`或`hasattr(ds, '属性名')`处理缺失属性

### 处理像素数据

提取并操作DICOM文件中的图像数据：

```python
import pydicom
import numpy as np
import matplotlib.pyplot as plt

# 读取DICOM文件
ds = pydicom.dcmread('image.dcm')

# 获取像素数组（需numpy支持）
pixel_array = ds.pixel_array

# 图像信息
print(f"形状: {pixel_array.shape}")
print(f"数据类型: {pixel_array.dtype}")
print(f"行数: {ds.Rows}, 列数: {ds.Columns}")

# 应用窗宽窗位显示（CT/MRI）
if hasattr(ds, 'WindowCenter') and hasattr(ds, 'WindowWidth'):
    from pydicom.pixel_data_handlers.util import apply_voi_lut
    windowed_image = apply_voi_lut(pixel_array, ds)
else:
    windowed_image = pixel_array

# 显示图像
plt.imshow(windowed_image, cmap='gray')
plt.title(f"{ds.Modality} - {ds.StudyDescription}")
plt.axis('off')
plt.show()
```

**处理彩色图像：**

```python
# RGB图像形状为(行, 列, 3)
if ds.PhotometricInterpretation == 'RGB':
    rgb_image = ds.pixel_array
    plt.imshow(rgb_image)
elif ds.PhotometricInterpretation == 'YBR_FULL':
    from pydicom.pixel_data_handlers.util import convert_color_space
    rgb_image = convert_color_space(ds.pixel_array, 'YBR_FULL', 'RGB')
    plt.imshow(rgb_image)
```

**多帧图像（视频/序列）：**

```python
# 处理多帧DICOM文件
if hasattr(ds, 'NumberOfFrames') and ds.NumberOfFrames > 1:
    frames = ds.pixel_array  # 形状: (帧数, 行, 列)
    print(f"帧数: {frames.shape[0]}")

    # 显示特定帧
    plt.imshow(frames[0], cmap='gray')
```

### 转换DICOM为图像格式

使用提供的`dicom_to_image.py`脚本或手动转换：

```python
from PIL import Image
import pydicom
import numpy as np

ds = pydicom.dcmread('input.dcm')
pixel_array = ds.pixel_array

# 归一化到0-255范围
if pixel_array.dtype != np.uint8:
    pixel_array = ((pixel_array - pixel_array.min()) /
                   (pixel_array.max() - pixel_array.min()) * 255).astype(np.uint8)

# 保存为PNG
image = Image.fromarray(pixel_array)
image.save('output.png')
```

使用脚本：`python scripts/dicom_to_image.py input.dcm output.png`

### 修改元数据

修改DICOM数据元素：

```python
import pydicom
from datetime import datetime

ds = pydicom.dcmread('input.dcm')

# 修改现有元素
ds.PatientName = "Doe^John"
ds.StudyDate = datetime.now().strftime('%Y%m%d')
ds.StudyDescription = "修改后的检查"

# 添加新元素
ds.SeriesNumber = 1
ds.SeriesDescription = "新序列"

# 删除元素
if hasattr(ds, 'PatientComments'):
    delattr(ds, 'PatientComments')
# 或使用del
if 'PatientComments' in ds:
    del ds.PatientComments

# 保存修改后的文件
ds.save_as('modified.dcm')
```

### DICOM文件匿名化

移除或替换患者身份信息：

```python
import pydicom
from datetime import datetime

ds = pydicom.dcmread('input.dcm')

# 包含PHI（受保护健康信息）的常见标签
tags_to_anonymize = [
    'PatientName', 'PatientID', 'PatientBirthDate',
    'PatientSex', 'PatientAge', 'PatientAddress',
    'InstitutionName', 'InstitutionAddress',
    'ReferringPhysicianName', 'PerformingPhysicianName',
    'OperatorsName', 'StudyDescription', 'SeriesDescription',
]

# 移除或替换敏感数据
for tag in tags_to_anonymize:
    if hasattr(ds, tag):
        if tag in ['PatientName', 'PatientID']:
            setattr(ds, tag, '匿名')
        elif tag == 'PatientBirthDate':
            setattr(ds, tag, '19000101')
        else:
            delattr(ds, tag)

# 更新日期以保持时间关系
if hasattr(ds, 'StudyDate'):
    # 随机偏移日期
    ds.StudyDate = '20000101'

# 保持像素数据完整
ds.save_as('anonymized.dcm')
```

使用脚本：`python scripts/anonymize_dicom.py input.dcm output.dcm`

### 创建DICOM文件

从头创建DICOM文件：

```python
import pydicom
from pydicom.dataset import Dataset, FileDataset
from datetime import datetime
import numpy as np

# 创建文件元信息
file_meta = Dataset()
file_meta.MediaStorageSOPClassUID = pydicom.uid.generate_uid()
file_meta.MediaStorageSOPInstanceUID = pydicom.uid.generate_uid()
file_meta.TransferSyntaxUID = pydicom.uid.ExplicitVRLittleEndian

# 创建FileDataset实例
ds = FileDataset('new_dicom.dcm', {}, file_meta=file_meta, preamble=b"\0" * 128)

# 添加必需DICOM元素
ds.PatientName = "Test^Patient"
ds.PatientID = "123456"
ds.Modality = "CT"
ds.StudyDate = datetime.now().strftime('%Y%m%d')
ds.StudyTime = datetime.now().strftime('%H%M%S')
ds.ContentDate = ds.StudyDate
ds.ContentTime = ds.StudyTime

# 添加图像特定元素
ds.SamplesPerPixel = 1
ds.PhotometricInterpretation = "MONOCHROME2"
ds.Rows = 512
ds.Columns = 512
ds.BitsAllocated = 16
ds.BitsStored = 16
ds.HighBit = 15
ds.PixelRepresentation = 0

# 创建像素数据
pixel_array = np.random.randint(0, 4096, (512, 512), dtype=np.uint16)
ds.PixelData = pixel_array.tobytes()

# 添加必需UID
ds.SOPClassUID = pydicom.uid.CTImageStorage
ds.SOPInstanceUID = file_meta.MediaStorageSOPInstanceUID
ds.SeriesInstanceUID = pydicom.uid.generate_uid()
ds.StudyInstanceUID = pydicom.uid.generate_uid()

# 保存文件
ds.save_as('new_dicom.dcm')
```

### 压缩与解压缩

处理压缩DICOM文件：

```python
import pydicom

# 读取压缩DICOM文件
ds = pydicom.dcmread('compressed.dcm')

# 检查传输语法
print(f"传输语法: {ds.file_meta.TransferSyntaxUID}")
print(f"传输语法名称: {ds.file_meta.TransferSyntaxUID.name}")

# 解压缩并保存为非压缩格式
ds.decompress()
ds.save_as('uncompressed.dcm', write_like_original=False)

# 或在保存时压缩（需相应编码器）
ds_uncompressed = pydicom.dcmread('uncompressed.dcm')
ds_uncompressed.compress(pydicom.uid.JPEGBaseline8Bit)
ds_uncompressed.save_as('compressed_jpeg.dcm')
```

**常见传输语法：**
- `ExplicitVRLittleEndian` - 未压缩，最常用
- `JPEGBaseline8Bit` - JPEG有损压缩
- `JPEGLossless` - JPEG无损压缩
- `JPEG2000Lossless` - JPEG 2000无损
- `RLELossless` - 游程编码无损

完整列表见`references/transfer_syntaxes.md`。

### 处理DICOM序列

操作嵌套数据结构：

```python
import pydicom

ds = pydicom.dcmread('file.dcm')

# 访问序列
if 'ReferencedStudySequence' in ds:
    for item in ds.ReferencedStudySequence:
        print(f"引用的SOP实例UID: {item.ReferencedSOPInstanceUID}")

# 创建序列
from pydicom.sequence import Sequence

sequence_item = Dataset()
sequence_item.ReferencedSOPClassUID = pydicom.uid.CTImageStorage
sequence_item.ReferencedSOPInstanceUID = pydicom.uid.generate_uid()

ds.ReferencedImageSequence = Sequence([sequence_item])
```

### 处理DICOM序列

处理多个相关DICOM文件：

```python
import pydicom
import numpy as np
from pathlib import Path

# 读取目录下所有DICOM文件
dicom_dir = Path('dicom_series/')
slices = []

for file_path in dicom_dir.glob('*.dcm'):
    ds = pydicom.dcmread(file_path)
    slices.append(ds)

# 按切片位置或实例号排序
slices.sort(key=lambda x: float(x.ImagePositionPatient[2]))
# 或: slices.sort(key=lambda x: int(x.InstanceNumber))

# 创建3D体积
volume = np.stack([s.pixel_array for s in slices])
print(f"体积形状: {volume.shape}")  # (切片数, 行, 列)

# 获取间距信息以正确缩放
pixel_spacing = slices[0].PixelSpacing  # [行间距, 列间距]
slice_thickness = slices[0].SliceThickness
print(f"体素尺寸: {pixel_spacing[0]}x{pixel_spacing[1]}x{slice_thickness} mm")
```

## 辅助脚本

本技能在`scripts/`目录提供实用脚本：

### anonymize_dicom.py
通过移除或替换受保护健康信息（PHI）实现DICOM匿名化。

```bash
python scripts/anonymize_dicom.py input.dcm output.dcm
```

### dicom_to_image.py
将DICOM文件转换为常见图像格式（PNG、JPEG、TIFF）。

```bash
python scripts/dicom_to_image.py input.dcm output.png
python scripts/dicom_to_image.py input.dcm output.jpg --format JPEG
```

### extract_metadata.py
提取并以可读格式显示DICOM元数据。

```bash
python scripts/extract_metadata.py file.dcm
python scripts/extract_metadata.py file.dcm --output metadata.txt
```

## 参考资料

`references/`目录包含详细参考信息：

- **common_tags.md**：按类别（患者、检查、序列、图像等）整理的常用DICOM标签完整列表
- **transfer_syntaxes.md**：DICOM传输语法和压缩格式完整参考

## 常见问题与解决方案

**问题："无法解码像素数据"**
- 解决方案：安装额外压缩处理器：`uv pip install pylibjpeg pylibjpeg-libjpeg python-gdcm`

**问题：访问标签时出现"AttributeError"**
- 解决方案：使用`hasattr(ds, '属性名')`检查属性是否存在，或使用`ds.get('属性名', 默认值)`

**问题：图像显示异常（过暗/过亮）**
- 解决方案：应用VOI LUT窗宽窗位：`apply_voi_lut(pixel_array, ds)`，或手动调整`WindowCenter`和`WindowWidth`

**问题：大型序列内存不足**
- 解决方案：迭代处理文件，使用内存映射数组，或对图像降采样

## 最佳实践

1. **始终检查必需属性**：使用`hasattr()`或`get()`访问前验证属性存在
2. **保留文件元数据**：修改文件时使用`save_as(write_like_original=True)`
3. **理解传输语法UID**：处理像素数据前确认压缩格式
4. **处理异常**：读取不可信来源文件时捕获异常
5. **正确应用窗宽窗位**：医学影像可视化必须使用VOI LUT
6. **保持空间信息**：处理3D体积时保留像素间距和切片厚度
7. **严格验证匿名化**：共享医学数据前彻底检查
8. **正确使用UID**：创建新实例时生成新UID，修改时保留原UID

## 文档

官方pydicom文档：https://pydicom.github.io/pydicom/dev/
- 用户指南：https://pydicom.github.io/pydicom/dev/guides/user/index.html
- 教程：https://pydicom.github.io/pydicom/dev/tutorials/index.html
- API参考：https://pydicom.github.io/pydicom/dev/reference/index.html
- 示例：https://pydicom.github.io/pydicom/dev/auto_examples/index.html
