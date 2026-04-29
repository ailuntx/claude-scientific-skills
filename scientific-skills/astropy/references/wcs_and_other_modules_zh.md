# WCS 及其他 Astropy 模块

## 世界坐标系系统 (astropy.wcs)

WCS 模块管理图像像素坐标与世界坐标（如天球坐标）之间的转换。

### 从 FITS 读取 WCS

```python
from astropy.wcs import WCS
from astropy.io import fits

# 从 FITS 头读取 WCS
with fits.open('image.fits') as hdul:
    wcs = WCS(hdul[0].header)
```

### 像素到世界坐标转换

```python
# 单像素转世界坐标
world = wcs.pixel_to_world(100, 200)  # 返回 SkyCoord 对象
print(f"赤经: {world.ra}, 赤纬: {world.dec}")

# 像素数组转换
import numpy as np
x_pixels = np.array([100, 200, 300])
y_pixels = np.array([150, 250, 350])
world_coords = wcs.pixel_to_world(x_pixels, y_pixels)
```

### 世界到像素坐标转换

```python
from astropy.coordinates import SkyCoord
import astropy.units as u

# 单坐标转换
coord = SkyCoord(ra=10.5*u.degree, dec=41.2*u.degree)
x, y = wcs.world_to_pixel(coord)

# 坐标数组转换
coords = SkyCoord(ra=[10, 11, 12]*u.degree, dec=[41, 42, 43]*u.degree)
x_pixels, y_pixels = wcs.world_to_pixel(coords)
```

### WCS 信息

```python
# 打印 WCS 详情
print(wcs)

# 访问关键属性
print(wcs.wcs.crpix)  # 参考像素
print(wcs.wcs.crval)  # 参考值（世界坐标）
print(wcs.wcs.cd)     # CD 矩阵
print(wcs.wcs.ctype)  # 坐标类型

# 像素尺度
pixel_scale = wcs.proj_plane_pixel_scales()  # 返回 Quantity 数组
```

### 创建 WCS

```python
from astropy.wcs import WCS

# 创建新 WCS
wcs = WCS(naxis=2)
wcs.wcs.crpix = [512.0, 512.0]  # 参考像素
wcs.wcs.crval = [10.5, 41.2]     # 参考像素处的赤经赤纬
wcs.wcs.ctype = ['RA---TAN', 'DEC--TAN']  # 投影类型
wcs.wcs.cdelt = [-0.0001, 0.0001]  # 像素尺度（度/像素）
wcs.wcs.cunit = ['deg', 'deg']
```

### 图像覆盖范围

```python
# 计算图像覆盖区域（角点坐标）
footprint = wcs.calc_footprint()
# 返回每个角点的 [赤经, 赤纬] 数组
```

## NDData (astropy.nddata)

用于存储带元数据、不确定度和掩码的 n 维数据集容器。

### 创建 NDData

```python
from astropy.nddata import NDData
import numpy as np
import astropy.units as u

# 基础 NDData
data = np.random.random((100, 100))
ndd = NDData(data)

# 带单位
ndd = NDData(data, unit=u.electron/u.s)

# 带不确定度
from astropy.nddata import StdDevUncertainty
uncertainty = StdDevUncertainty(np.sqrt(data))
ndd = NDData(data, uncertainty=uncertainty, unit=u.electron/u.s)

# 带掩码
mask = data < 0.1  # 屏蔽低值
ndd = NDData(data, mask=mask)

# 带 WCS
from astropy.wcs import WCS
ndd = NDData(data, wcs=wcs)
```

### CCDData 处理 CCD 图像

```python
from astropy.nddata import CCDData

# 创建 CCDData
ccd = CCDData(data, unit=u.adu, meta={'object': 'M31'})

# 从 FITS 读取
ccd = CCDData.read('image.fits', unit=u.adu)

# 写入 FITS
ccd.write('output.fits', overwrite=True)
```

## 建模 (astropy.modeling)

用于创建模型并拟合数据的框架。

### 常用模型

```python
from astropy.modeling import models, fitting
import numpy as np

# 一维高斯
gauss = models.Gaussian1D(amplitude=10, mean=5, stddev=1)
x = np.linspace(0, 10, 100)
y = gauss(x)

# 二维高斯
gauss_2d = models.Gaussian2D(amplitude=10, x_mean=50, y_mean=50,
                              x_stddev=5, y_stddev=3)

# 多项式
poly = models.Polynomial1D(degree=3)

# 幂律
power_law = models.PowerLaw1D(amplitude=10, x_0=1, alpha=2)
```

### 模型拟合数据

```python
# 生成含噪数据
true_model = models.Gaussian1D(amplitude=10, mean=5, stddev=1)
x = np.linspace(0, 10, 100)
y_true = true_model(x)
y_noisy = y_true + np.random.normal(0, 0.5, x.shape)

# 拟合模型
fitter = fitting.LevMarLSQFitter()
initial_model = models.Gaussian1D(amplitude=8, mean=4, stddev=1.5)
fitted_model = fitter(initial_model, x, y_noisy)

print(f"拟合振幅: {fitted_model.amplitude.value}")
print(f"拟合均值: {fitted_model.mean.value}")
print(f"拟合标准差: {fitted_model.stddev.value}")
```

### 复合模型

```python
# 模型相加
double_gauss = models.Gaussian1D(amp=5, mean=3, stddev=1) + \
               models.Gaussian1D(amp=8, mean=7, stddev=1.5)

# 模型组合
composite = models.Gaussian1D(amp=10, mean=5, stddev=1) | \
            models.Scale(factor=2)  # 缩放输出
```

## 可视化 (astropy.visualization)

天文图像与数据的可视化工具。

### 图像归一化

```python
from astropy.visualization import simple_norm
import matplotlib.pyplot as plt

# 加载图像
from astropy.io import fits
data = fits.getdata('image.fits')

# 归一化显示
norm = simple_norm(data, 'sqrt', percent=99)

# 显示图像
plt.imshow(data, norm=norm, cmap='gray', origin='lower')
plt.colorbar()
plt.show()
```

### 拉伸与区间调整

```python
from astropy.visualization import (MinMaxInterval, AsinhStretch,
                                    ImageNormalize, ZScaleInterval)

# Z 尺度区间
interval = ZScaleInterval()
vmin, vmax = interval.get_limits(data)

# 反双曲正弦拉伸
stretch = AsinhStretch()
norm = ImageNormalize(data, interval=interval, stretch=stretch)

plt.imshow(data, norm=norm, cmap='gray', origin='lower')
```

### 百分位区间

```python
from astropy.visualization import PercentileInterval

# 显示 5% 到 95% 分位的数据
interval = PercentileInterval(90)  # 90% 数据范围
vmin, vmax = interval.get_limits(data)

plt.imshow(data, vmin=vmin, vmax=vmax, cmap='gray', origin='lower')
```

## 常量 (astropy.constants)

带单位的物理与天文常量。

```python
from astropy import constants as const

# 光速
c = const.c
print(f"光速 = {c}")
print(f"光速 (km/s) = {c.to(u.km/u.s)}")

# 引力常数
G = const.G

# 天文常量
M_sun = const.M_sun     # 太阳质量
R_sun = const.R_sun     # 太阳半径
L_sun = const.L_sun     # 太阳光度
au = const.au           # 天文单位
pc = const.pc           # 秒差距

# 基本常量
h = const.h             # 普朗克常数
hbar = const.hbar       # 约化普朗克常数
k_B = const.k_B         # 玻尔兹曼常数
m_e = const.m_e         # 电子质量
m_p = const.m_p         # 质子质量
e = const.e             # 基本电荷
N_A = const.N_A         # 阿伏伽德罗常数
```

### 在计算中使用常量

```python
# 计算史瓦西半径
M = 10 * const.M_sun
r_s = 2 * const.G * M / const.c**2
print(f"史瓦西半径: {r_s.to(u.km)}")

# 计算逃逸速度
M = const.M_earth
R = const.R_earth
v_esc = np.sqrt(2 * const.G * M / R)
print(f"地球逃逸速度: {v_esc.to(u.km/u.s)}")
```

## 卷积 (astropy.convolution)

图像处理的卷积核。

```python
from astropy.convolution import Gaussian2DKernel, convolve

# 创建高斯核
kernel = Gaussian2DKernel(x_stddev=2)

# 卷积图像
smoothed_image = convolve(data, kernel)

# 处理 NaN
from astropy.convolution import convolve_fft
smoothed = convolve_fft(data, kernel, nan_treatment='interpolate')
```

## 统计 (astropy.stats)

天文数据的统计函数。

```python
from astropy.stats import sigma_clip, sigma_clipped_stats

# 西格玛截断
clipped_data = sigma_clip(data, sigma=3, maxiters=5)

# 获取西格玛截断统计量
mean, median, std = sigma_clipped_stats(data, sigma=3.0)

# 稳健统计
from astropy.stats import mad_std, biweight_location, biweight_scale
robust_std = mad_std(data)
robust_mean = biweight_location(data)
robust_scale = biweight_scale(data)
```

## 实用工具

### 数据下载

```python
from astropy.utils.data import download_file

# 下载文件（本地缓存）
url = 'https://example.com/data.fits'
local_file = download_file(url, cache=True)
```

### 进度条

```python
from astropy.utils.console import ProgressBar

with ProgressBar(len(data_list)) as bar:
    for item in data_list:
        # 处理项目
        bar.update()
```

## SAMP（简单应用消息协议）

与其他天文工具的互操作性。

```python
from astropy.samp import SAMPIntegratedClient

# 连接 SAMP 中心
client = SAMPIntegratedClient()
client.connect()

# 向其他应用广播表格
message = {
    'samp.mtype': 'table.load.votable',
    'samp.params': {
        'url': 'file:///path/to/table.xml',
        'table-id': 'my_table',
        'name': 'My Catalog'
    }
}
client.notify_all(message)

# 断开连接
client.disconnect()
```
