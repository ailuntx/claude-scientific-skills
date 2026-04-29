---
name: astropy
description: 用于天文学和天体物理学的综合性Python库。当处理天文数据时应使用此技能，包括天体坐标、物理单位、FITS文件、宇宙学计算、时间系统、表格、世界坐标系（WCS）和天文数据分析。适用于坐标转换、单位换算、FITS文件操作、宇宙学距离计算、时间尺度转换或天文数据处理等任务。
license: BSD-3-Clause许可证
metadata:
    skill-author: K-Dense公司
---

# Astropy

## 概述

Astropy是天文领域的核心Python包，为天文研究和数据分析提供基础功能。可用于坐标转换、单位和量值计算、FITS文件操作、宇宙学计算、精确时间处理、表格数据处理以及天文图像处理。

## 何时使用此技能

在以下任务中使用astropy：
- 天体坐标系转换（ICRS、银河系、FK5、地平坐标等）
- 处理物理单位和量值（如将Jy转换为mJy、秒差距转换为千米）
- 读写或操作FITS文件（图像或表格）
- 宇宙学计算（光度距离、回溯时间、哈勃参数）
- 不同时间尺度（UTC、TAI、TT、TDB）和格式（JD、MJD、ISO）的精确时间处理
- 表格操作（读取星表、交叉匹配、筛选、连接）
- 像素坐标与世界坐标系（WCS）间的转换
- 天文常数与计算

## 快速入门

```python
import astropy.units as u
from astropy.coordinates import SkyCoord
from astropy.time import Time
from astropy.io import fits
from astropy.table import Table
from astropy.cosmology import Planck18

# 单位与量值
distance = 100 * u.pc
distance_km = distance.to(u.km)

# 坐标
coord = SkyCoord(ra=10.5*u.degree, dec=41.2*u.degree, frame='icrs')
coord_galactic = coord.galactic

# 时间
t = Time('2023-01-15 12:30:00')
jd = t.jd  # 儒略日

# FITS文件
data = fits.getdata('image.fits')
header = fits.getheader('image.fits')

# 表格
table = Table.read('catalog.fits')

# 宇宙学
d_L = Planck18.luminosity_distance(z=1.0)
```

## 核心功能

### 1. 单位与量值 (`astropy.units`)

处理带单位的物理量，执行单位转换，确保计算中的量纲一致性。

**关键操作：**
- 通过数值乘以单位创建量值
- 使用`.to()`方法进行单位转换
- 执行自动单位处理的算术运算
- 使用等效关系进行领域特定转换（光谱、多普勒、视差）
- 处理对数单位（星等、分贝）

**参见：** `references/units.md` 获取完整文档（单位系统、等效关系、性能优化和单位运算）

### 2. 坐标系 (`astropy.coordinates`)

表示天体位置并在不同坐标框架间转换。

**关键操作：**
- 用`SkyCoord`创建任意框架坐标（ICRS、银河系、FK5、地平坐标等）
- 坐标系间转换
- 计算角距和位置角
- 与星表坐标匹配
- 包含距离的三维坐标操作
- 处理自行和径向速度
- 从在线数据库查询命名天体

**参见：** `references/coordinates.md` 获取详细坐标框架说明、转换方法、观测者相关框架（地平坐标）、星表匹配和性能优化建议

### 3. 宇宙学计算 (`astropy.cosmology`)

使用标准宇宙学模型执行计算。

**关键操作：**
- 使用内置宇宙学模型（Planck18、WMAP9等）
- 创建自定义宇宙学模型
- 计算距离（光度距离、共动距离、角直径距离）
- 计算宇宙年龄和回溯时间
- 获取任意红移处的哈勃参数
- 计算密度参数和体积
- 执行逆向计算（根据距离求红移）

**参见：** `references/cosmology.md` 获取可用模型、距离计算、时间计算、密度参数和中微子效应说明

### 4. FITS文件处理 (`astropy.io.fits`)

读写和操作FITS（灵活图像传输系统）文件。

**关键操作：**
- 使用上下文管理器打开FITS文件
- 通过索引或名称访问HDU（头数据单元）
- 读写修改头文件（关键字、注释、历史记录）
- 处理图像数据（NumPy数组）
- 操作表格数据（二进制和ASCII表）
- 创建新FITS文件（单扩展或多扩展）
- 对大文件使用内存映射
- 访问远程FITS文件（S3、HTTP）

**参见：** `references/fits.md` 获取完整文件操作、头文件处理、图像表格操作、多扩展文件处理和性能考量

### 5. 表格操作 (`astropy.table`)

处理表格数据，支持单位、元数据和多种文件格式。

**关键操作：**
- 从数组、列表或字典创建表格
- 多格式读写表格（FITS、CSV、HDF5、VOTable）
- 访问修改行列
- 排序、筛选和索引表格
- 执行类数据库操作（连接、分组、聚合）
- 堆叠和拼接表格
- 处理带单位的列（QTable）
- 用掩码处理缺失数据

**参见：** `references/tables.md` 获取表格创建、I/O操作、数据处理、排序筛选、连接分组和性能建议

### 6. 时间处理 (`astropy.time`)

精确表示时间并在时间尺度和格式间转换。

**关键操作：**
- 用多种格式创建时间对象（ISO、JD、MJD、Unix等）
- 时间尺度转换（UTC、TAI、TT、TDB等）
- 用TimeDelta执行时间算术
- 计算观测者所在地的恒星时
- 计算光行时校正（质心、日心）
- 高效处理时间数组
- 处理掩码（缺失）时间

**参见：** `references/time.md` 获取时间格式、时间尺度、转换方法、算术运算、观测特性和精度处理

### 7. 世界坐标系 (`astropy.wcs`)

在图像像素坐标与世界坐标间转换。

**关键操作：**
- 从FITS头文件读取WCS
- 像素坐标与世界坐标互转
- 计算图像覆盖区域
- 访问WCS参数（参考像素、投影、比例）
- 创建自定义WCS对象

**参见：** `references/wcs_and_other_modules.md` 获取WCS操作和转换方法

## 附加功能

`references/wcs_and_other_modules.md` 文件还涵盖：

### NDData与CCDData
包含n维数据集的容器，支持元数据、不确定性、掩码和WCS信息。

### 建模
为天文数据创建和拟合数学模型的框架。

### 可视化
天文图像显示工具，支持适当拉伸和缩放。

### 常数
带单位的物理和天文常数（光速、太阳质量、普朗克常数等）。

### 卷积
用于平滑和滤波的图像处理核函数。

### 统计
稳健统计函数，包括西格玛截断和离群值剔除。

## 安装

```bash
# 安装astropy
uv pip install astropy

# 安装完整功能所需依赖
uv pip install astropy[all]
```

## 常用工作流

### 坐标系间转换

```python
from astropy.coordinates import SkyCoord
import astropy.units as u

# 创建坐标
c = SkyCoord(ra='05h23m34.5s', dec='-69d45m22s', frame='icrs')

# 转换为银河系坐标
c_gal = c.galactic
print(f"银经l={c_gal.l.deg}, 银纬b={c_gal.b.deg}")

# 转换为地平坐标（需时间和位置）
from astropy.time import Time
from astropy.coordinates import EarthLocation, AltAz

observing_time = Time('2023-06-15 23:00:00')
observing_location = EarthLocation(lat=40*u.deg, lon=-120*u.deg)
aa_frame = AltAz(obstime=observing_time, location=observing_location)
c_altaz = c.transform_to(aa_frame)
print(f"高度角={c_altaz.alt.deg}, 方位角={c_altaz.az.deg}")
```

### 读取分析FITS文件

```python
from astropy.io import fits
import numpy as np

# 打开FITS文件
with fits.open('observation.fits') as hdul:
    # 显示结构
    hdul.info()

    # 获取图像数据和头文件
    data = hdul[1].data
    header = hdul[1].header

    # 访问头文件值
    exptime = header['EXPTIME']
    filter_name = header['FILTER']

    # 分析数据
    mean = np.mean(data)
    median = np.median(data)
    print(f"均值: {mean}, 中位数: {median}")
```

### 宇宙学距离计算

```python
from astropy.cosmology import Planck18
import astropy.units as u
import numpy as np

# 计算z=1.5处的距离
z = 1.5
d_L = Planck18.luminosity_distance(z)
d_A = Planck18.angular_diameter_distance(z)

print(f"光度距离: {d_L}")
print(f"角直径距离: {d_A}")

# 该红移处的宇宙年龄
age = Planck18.age(z)
print(f"z={z}处的年龄: {age.to(u.Gyr)}")

# 回溯时间
t_lookback = Planck18.lookback_time(z)
print(f"回溯时间: {t_lookback.to(u.Gyr)}")
```

### 星表交叉匹配

```python
from astropy.table import Table
from astropy.coordinates import SkyCoord, match_coordinates_sky
import astropy.units as u

# 读取星表
cat1 = Table.read('catalog1.fits')
cat2 = Table.read('catalog2.fits')

# 创建坐标对象
coords1 = SkyCoord(ra=cat1['RA']*u.degree, dec=cat1['DEC']*u.degree)
coords2 = SkyCoord(ra=cat2['RA']*u.degree, dec=cat2['DEC']*u.degree)

# 查找匹配
idx, sep, _ = coords1.match_to_catalog_sky(coords2)

# 按角距阈值筛选
max_sep = 1 * u.arcsec
matches = sep < max_sep

# 创建匹配星表
cat1_matched = cat1[matches]
cat2_matched = cat2[idx[matches]]
print(f"找到{len(cat1_matched)}个匹配")
```

## 最佳实践

1. **始终使用单位**：为量值附加单位以避免错误并确保量纲一致
2. **FITS文件使用上下文管理器**：确保文件正确关闭
3. **优先使用数组而非循环**：以数组形式批量处理坐标/时间提升性能
4. **检查坐标框架**：转换前验证框架类型
5. **选用合适宇宙学模型**：为分析选择正确模型
6. **处理缺失数据**：对含缺失值的表格使用掩码列
7. **明确时间尺度**：精确计时需指定时间尺度（UTC、TT、TDB）
8. **单位感知表格使用QTable**：当表格列包含单位时
9. **验证WCS有效性**：使用转换前检查WCS
10. **缓存常用值**：可缓存昂贵计算（如宇宙学距离）

## 文档与资源

- 官方文档：https://docs.astropy.org/en/stable/
- 教程：https://learn.astropy.org/
- GitHub：https://github.com/astropy/astropy

## 参考文件

各模块详细说明：
- `references/units.md` - 单位、量值、转换和等效关系
- `references/coordinates.md` - 坐标系、转换和星表匹配
- `references/cosmology.md` - 宇宙学模型与计算
- `references/fits.md` - FITS文件操作与处理
- `references/tables.md` - 表格创建、I/O和操作
- `references/time.md` - 时间格式、尺度和计算
- `references/wcs_and_other_modules.md` - WCS、NDData、建模、可视化、常数和工具
