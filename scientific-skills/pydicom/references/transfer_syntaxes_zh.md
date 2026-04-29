# DICOM 传输语法参考

本文档提供 DICOM 传输语法和压缩格式的全面参考。传输语法定义了 DICOM 数据的编码方式，包括字节顺序、压缩方法及其他编码规则。

## 概述

传输语法 UID 指定：
1. **字节顺序**：小端序或大端序
2. **值表示法 (VR)**：隐式或显式
3. **压缩**：无压缩或特定压缩算法

## 未压缩传输语法

### 隐式 VR 小端序 (1.2.840.10008.1.2)
- **默认**传输语法
- 值表示法为隐式（不显式编码）
- 小端序字节顺序
- **Pydicom 常量**：`pydicom.uid.ImplicitVRLittleEndian`

**用法：**
```python
import pydicom
ds.file_meta.TransferSyntaxUID = pydicom.uid.ImplicitVRLittleEndian
```

### 显式 VR 小端序 (1.2.840.10008.1.2.1)
- **最常用**传输语法
- 值表示法为显式
- 小端序字节顺序
- **Pydicom 常量**：`pydicom.uid.ExplicitVRLittleEndian`

**用法：**
```python
ds.file_meta.TransferSyntaxUID = pydicom.uid.ExplicitVRLittleEndian
```

### 显式 VR 大端序 (1.2.840.10008.1.2.2) - 已弃用
- 值表示法为显式
- 大端序字节顺序
- **已弃用** - 不建议新实现使用
- **Pydicom 常量**：`pydicom.uid.ExplicitVRBigEndian`

## JPEG 压缩

### JPEG 基线 (Process 1) (1.2.840.10008.1.2.4.50)
- **有损**压缩
- 仅支持 8 位样本
- 最广泛支持的 JPEG 格式
- **Pydicom 常量**：`pydicom.uid.JPEGBaseline8Bit`

**依赖项：** 需要 `pylibjpeg` 或 `pillow`

**用法：**
```python
# 压缩
ds.compress(pydicom.uid.JPEGBaseline8Bit)

# 解压缩
ds.decompress()
```

### JPEG 扩展 (Process 2 & 4) (1.2.840.10008.1.2.4.51)
- **有损**压缩
- 支持 8 位和 12 位样本
- **Pydicom 常量**：`pydicom.uid.JPEGExtended12Bit`

### JPEG 无损, 非分层 (Process 14) (1.2.840.10008.1.2.4.57)
- **无损**压缩
- 一阶预测
- **Pydicom 常量**：`pydicom.uid.JPEGLossless`

**依赖项：** 需要 `pylibjpeg-libjpeg` 或 `gdcm`

### JPEG 无损, 非分层, 一阶预测 (1.2.840.10008.1.2.4.70)
- **无损**压缩
- 使用 Process 14 选择值 1
- **Pydicom 常量**：`pydicom.uid.JPEGLosslessSV1`

**用法：**
```python
# 压缩为 JPEG 无损格式
ds.compress(pydicom.uid.JPEGLossless)
```

### JPEG-LS 无损 (1.2.840.10008.1.2.4.80)
- **无损**压缩
- 低复杂度，良好压缩率
- **Pydicom 常量**：`pydicom.uid.JPEGLSLossless`

**依赖项：** 需要 `pylibjpeg-libjpeg` 或 `gdcm`

### JPEG-LS 有损 (近无损) (1.2.840.10008.1.2.4.81)
- **近无损**压缩
- 允许受控精度损失
- **Pydicom 常量**：`pydicom.uid.JPEGLSNearLossless`

## JPEG 2000 压缩

### JPEG 2000 纯无损 (1.2.840.10008.1.2.4.90)
- **无损**压缩
- 基于小波的压缩
- 压缩率优于 JPEG 无损
- **Pydicom 常量**：`pydicom.uid.JPEG2000Lossless`

**依赖项：** 需要 `pylibjpeg-openjpeg`, `gdcm` 或 `pillow`

**用法：**
```python
# 压缩为 JPEG 2000 无损格式
ds.compress(pydicom.uid.JPEG2000Lossless)
```

### JPEG 2000 (1.2.840.10008.1.2.4.91)
- **有损或无损**压缩
- 基于小波的压缩
- 低比特率下高质量
- **Pydicom 常量**：`pydicom.uid.JPEG2000`

**依赖项：** 需要 `pylibjpeg-openjpeg`, `gdcm` 或 `pillow`

### JPEG 2000 Part 2 多分量无损 (1.2.840.10008.1.2.4.92)
- **无损**压缩
- 支持多分量图像
- **Pydicom 常量**：`pydicom.uid.JPEG2000MCLossless`

### JPEG 2000 Part 2 多分量 (1.2.840.10008.1.2.4.93)
- **有损或无损**压缩
- 支持多分量图像
- **Pydicom 常量**：`pydicom.uid.JPEG2000MC`

## RLE 压缩

### RLE 无损 (1.2.840.10008.1.2.5)
- **无损**压缩
- 游程编码
- 简单快速算法
- 适用于重复值图像
- **Pydicom 常量**：`pydicom.uid.RLELossless`

**依赖项：** 内置于 pydicom（无需额外包）

**用法：**
```python
# 使用 RLE 压缩
ds.compress(pydicom.uid.RLELossless)

# 解压缩
ds.decompress()
```

## 紧缩传输语法

### 紧缩显式 VR 小端序 (1.2.840.10008.1.2.1.99)
- 对整个数据集使用 ZLIB 压缩
- 不常用
- **Pydicom 常量**：`pydicom.uid.DeflatedExplicitVRLittleEndian`

## MPEG 压缩

### MPEG2 主配置 @ 主级别 (1.2.840.10008.1.2.4.100)
- **有损**视频压缩
- 适用于多帧图像/视频
- **Pydicom 常量**：`pydicom.uid.MPEG2MPML`

### MPEG2 主配置 @ 高级别 (1.2.840.10008.1.2.4.101)
- **有损**视频压缩
- 分辨率高于 MPML
- **Pydicom 常量**：`pydicom.uid.MPEG2MPHL`

### MPEG-4 AVC/H.264 高级配置 (1.2.840.10008.1.2.4.102-106)
- **有损**视频压缩
- 多种级别 (BD, 2D, 3D, Stereo)
- 现代视频编解码器

## 检查传输语法

### 识别当前传输语法
```python
import pydicom

ds = pydicom.dcmread('image.dcm')

# 获取传输语法 UID
ts_uid = ds.file_meta.TransferSyntaxUID
print(f"传输语法 UID: {ts_uid}")

# 获取人类可读名称
print(f"传输语法名称: {ts_uid.name}")

# 检查是否压缩
print(f"是否压缩: {ts_uid.is_compressed}")
```

### 常用检查
```python
# 检查是否小端序
if ts_uid.is_little_endian:
    print("小端序")

# 检查是否隐式 VR
if ts_uid.is_implicit_VR:
    print("隐式 VR")

# 检查压缩类型
if 'JPEG' in ts_uid.name:
    print("JPEG 压缩")
elif 'JPEG2000' in ts_uid.name:
    print("JPEG 2000 压缩")
elif 'RLE' in ts_uid.name:
    print("RLE 压缩")
```

## 解压缩

### 自动解压缩
访问 `pixel_array` 时 Pydicom 会自动解压缩像素数据：
```python
import pydicom

# 读取压缩的 DICOM
ds = pydicom.dcmread('compressed.dcm')

# 像素数据自动解压缩
pixel_array = ds.pixel_array  # 需要时自动解压缩
```

### 手动解压缩
```python
import pydicom

ds = pydicom.dcmread('compressed.dcm')

# 原地解压缩
ds.decompress()

# 保存为未压缩格式
ds.save_as('uncompressed.dcm', write_like_original=False)
```

## 压缩

### 压缩 DICOM 文件
```python
import pydicom

ds = pydicom.dcmread('uncompressed.dcm')

# 使用 JPEG 2000 无损压缩
ds.compress(pydicom.uid.JPEG2000Lossless)
ds.save_as('compressed_j2k.dcm')

# 使用 RLE 无损压缩（无需额外依赖）
ds.compress(pydicom.uid.RLELossless)
ds.save_as('compressed_rle.dcm')

# 使用 JPEG 基线压缩（有损）
ds.compress(pydicom.uid.JPEGBaseline8Bit)
ds.save_as('compressed_jpeg.dcm')
```

### 使用自定义编码参数压缩
```python
import pydicom
from pydicom.encoders import JPEGLSLosslessEncoder

ds = pydicom.dcmread('uncompressed.dcm')

# 使用自定义参数压缩
ds.compress(pydicom.uid.JPEGLSLossless, encoding_plugin='pylibjpeg')
```

## 安装压缩处理器

不同传输语法需要不同的 Python 包：

### JPEG 基线/扩展
```bash
pip install pylibjpeg pylibjpeg-libjpeg
# 或
pip install pillow
```

### JPEG 无损/JPEG-LS
```bash
pip install pylibjpeg pylibjpeg-libjpeg
# 或
pip install python-gdcm
```

### JPEG 2000
```bash
pip install pylibjpeg pylibjpeg-openjpeg
# 或
pip install python-gdcm
# 或
pip install pillow
```

### RLE
无需额外包 - 内置于 pydicom

### 综合安装
```bash
# 安装所有常用处理器
pip install pylibjpeg pylibjpeg-libjpeg pylibjpeg-openjpeg python-gdcm
```

## 检查可用处理器

```python
import pydicom

# 列出可用像素数据处理器
from pydicom.pixel_data_handlers.util import get_pixel_data_handlers
handlers = get_pixel_data_handlers()

print("可用处理器:")
for handler in handlers:
    print(f"  - {handler.__name__}")
```

## 最佳实践

1. 创建新文件时**使用显式 VR 小端序**以获得最大兼容性
2. **使用 JPEG 2000 无损**实现高质量无损压缩
3. **使用 RLE 无损**当无法安装额外依赖时
4. 处理前**检查传输语法**确保安装正确处理器
5. 部署前**测试解压缩**确保所有必需包已安装
6. 尽可能**保留原始传输语法**（使用 `write_like_original=True`）
7. 选择有损压缩时**权衡文件大小与质量**
8. 诊断图像**使用无损压缩**以保持临床质量

## 常见问题

### 问题："无法解码像素数据"
**原因：** 缺少压缩处理器
**解决方案：** 安装对应包（见上方"安装压缩处理器"）

### 问题："不支持的传输语法"
**原因：** 罕见或不支持的压缩格式
**解决方案：** 尝试安装支持更多格式的 `python-gdcm`

### 问题："像素数据已解压但显示异常"
**原因：** 可能需要应用 VOI LUT 或重缩放
**解决方案：** 使用 `apply_voi_lut()` 或应用 `RescaleSlope`/`RescaleIntercept`

## 参考

- DICOM 标准第 5 部分（数据结构与编码）：https://dicom.nema.org/medical/dicom/current/output/chtml/part05/PS3.5.html
- Pydicom 传输语法文档：https://pydicom.github.io/pydicom/stable/guides/user/transfer_syntaxes.html
- Pydicom 压缩指南：https://pydicom.github.io/pydicom/stable/old/image_data_compression.html
