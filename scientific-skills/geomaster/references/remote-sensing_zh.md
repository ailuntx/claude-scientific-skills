# 遥感参考指南

卫星数据获取、处理与分析综合指南。

## 卫星任务概览

| 卫星 | 运营方 | 分辨率 | 重访周期 | 主要特性 |
|-----------|----------|------------|---------|--------------|
| **Sentinel-2** | 欧空局 | 10-60米 | 5天 | 13个波段，免费访问 |
| **Landsat 8/9** | 美国地质调查局 | 30米 | 16天 | 历史存档（1972年至今） |
| **MODIS** | 美国宇航局 | 250-1000米 | 每日 | 植被指数 |
| **PlanetScope** | Planet公司 | 3米 | 每日 | 商业高分辨率 |
| **WorldView** | Maxar公司 | 0.3米 | 可变 | 超高分辨率 |
| **Sentinel-1** | 欧空局 | 5-40米 | 6-12天 | 合成孔径雷达，全天候 |
| **Envisat** | 欧空局 | 30米 | 35天 | 合成孔径雷达（存档数据） |

## Sentinel-2数据处理

### 获取Sentinel-2数据

```python
import pystac_client
import planetary_computer
import odc.stac
import xarray as xr

# 搜索Sentinel-2数据集
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)

# 定义目标区域和时间范围
bbox = [-122.5, 37.7, -122.3, 37.9]
search = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=bbox,
    datetime="2023-01-01/2023-12-31",
    query={"eo:cloud_cover": {"lt": 20}},
)

items = list(search.get_items())
print(f"找到 {len(items)} 条数据")

# 加载为xarray数据集
data = odc.stac.load(
    [items[0]],
    bands=["B02", "B03", "B04", "B08", "B11"],
    crs="EPSG:32610",
    resolution=10,
)

print(data)
```

### 计算光谱指数

```python
import numpy as np
import rasterio

def ndvi(nir, red):
    """归一化植被指数"""
    return (nir - red) / (nir + red + 1e-8)

def evi(nir, red, blue):
    """增强型植被指数"""
    return 2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)

def savi(nir, red, L=0.5):
    """土壤调节植被指数"""
    return ((nir - red) / (nir + red + L)) * (1 + L)

def ndwi(green, nir):
    """归一化水体指数"""
    return (green - nir) / (green + nir + 1e-8)

def mndwi(green, swir):
    """改进型水体指数（开放水域）"""
    return (green - swir) / (green + swir + 1e-8)

def nbr(nir, swir):
    """归一化燃烧指数"""
    return (nir - swir) / (nir + swir + 1e-8)

def ndbi(swir, nir):
    """归一化建筑指数"""
    return (swir - nir) / (swir + nir + 1e-8)

# 批量处理
with rasterio.open('sentinel2.tif') as src:
    # Sentinel-2波段映射
    B02 = src.read(1).astype(float)  # 蓝光波段(10米)
    B03 = src.read(2).astype(float)  # 绿光波段(10米)
    B04 = src.read(3).astype(float)  # 红光波段(10米)
    B08 = src.read(4).astype(float)  # 近红外波段(10米)
    B11 = src.read(5).astype(float)  # 短波红外1波段(20米，重采样)

    # 计算指数
    NDVI = ndvi(B08, B04)
    EVI = evi(B08, B04, B02)
    SAVI = savi(B08, B04, L=0.5)
    NDWI = ndwi(B03, B08)
    NBR = nbr(B08, B11)
    NDBI = ndbi(B11, B08)
```

## Landsat数据处理

### Landsat Collection 2

```python
import ee

# 初始化Earth Engine
ee.Initialize(project='your-project')

# Landsat 8 Collection 2 Level 2
landsat = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2') \
    .filterBounds(ee.Geometry.Point([-122.4, 37.7])) \
    .filterDate('2020-01-01', '2023-12-31') \
    .filter(ee.Filter.lt('CLOUD_COVER', 20))

# 应用缩放因子（Collection 2）
def apply_scale_factors(image):
    optical = image.select('SR_B.').multiply(0.0000275).add(-0.2)
    thermal = image.select('ST_B10').multiply(0.00341802).add(149.0)
    return image.addBands(optical, None, True).addBands(thermal, None, True)

landsat_scaled = landsat.map(apply_scale_factors)

# 计算NDVI
def add_ndvi(image):
    ndvi = image.normalizedDifference(['SR_B5', 'SR_B4']).rename('NDVI')
    return image.addBands(ndvi)

landsat_ndvi = landsat_scaled.map(add_ndvi)

# 获取合成影像
composite = landsat_ndvi.median()
```

### 地表温度反演

```python
def land_surface_temperature(image):
    """从Landsat 8计算地表温度"""
    # 亮温
    Tb = image.select('ST_B10')

    # 发射率计算用NDVI
    ndvi = image.normalizedDifference(['SR_B5', 'SR_B4'])
    pv = ((ndvi - 0.2) / (0.5 - 0.2)) ** 2  # 植被比例

    # 发射率
    em = 0.004 * pv + 0.986

    # 开尔文温度
    lst = Tb.divide(1 + (0.00115 * Tb / 1.4388) * np.log(em))

    # 转换为摄氏度
    lst_c = lst.subtract(273.15).rename('LST')

    return image.addBands(lst_c)

landsat_lst = landsat_scaled.map(land_surface_temperature)
```

## SAR处理（合成孔径雷达）

### Sentinel-1 GRD数据处理

```python
import rasterio
from scipy.ndimage import gaussian_filter
import numpy as np

def process_sentinel1_grd(input_path, output_path):
    """处理Sentinel-1 GRD数据"""
    with rasterio.open(input_path) as src:
        # 读取VV和VH波段
        vv = src.read(1).astype(float)
        vh = src.read(2).astype(float)

        # 转换为分贝值
        vv_db = 10 * np.log10(vv + 1e-8)
        vh_db = 10 * np.log10(vh + 1e-8)

        # 斑点滤波（Lee滤波近似）
        def lee_filter(img, size=3):
            """简化的Lee斑点滤波"""
            # 局部均值
            mean = gaussian_filter(img, size)
            # 局部方差
            sq_mean = gaussian_filter(img**2, size)
            variance = sq_mean - mean**2
            # 噪声方差
            noise_var = np.var(img) * 0.5
            # Lee滤波公式
            return mean + (variance - noise_var) / (variance) * (img - mean)

        vv_filtered = lee_filter(vv_db)
        vh_filtered = lee_filter(vh_db)

        # 计算比值
        ratio = vv_db - vh_db  # 分贝单位：差值即比值

        # 保存结果
        profile = src.profile
        profile.update(dtype=rasterio.float32, count=3)

        with rasterio.open(output_path, 'w', **profile) as dst:
            dst.write(vv_filtered.astype(np.float32), 1)
            dst.write(vh_filtered.astype(np.float32), 2)
            dst.write(ratio.astype(np.float32), 3)

# 使用示例
process_sentinel1_grd('S1A_IW_GRDH.tif', 'S1A_processed.tif')
```

### SAR极化指数

```python
def calculate_sar_indices(vv, vh):
    """计算SAR衍生指数"""
    # 分贝后向散射比值
    ratio_db = 10 * np.log10(vv / (vh + 1e-8) + 1e-8)

    # 雷达植被指数
    rvi = (4 * vh) / (vv + vh + 1e-8)

    # 土壤湿度指数（近似）
    smi = vv / (vv + vh + 1e-8)

    return ratio_db, rvi, smi
```

## 高光谱成像

### 高光谱数据处理

```python
import spectral.io.envi as envi
import numpy as np
import matplotlib.pyplot as plt

# 加载高光谱立方体
hdr_path = 'hyperspectral.hdr'
img = envi.open(hdr_path)
data = img.load()

print(f"数据维度: {data.shape}")  # (行, 列, 波段)

# 提取单像素光谱特征
pixel_signature = data[100, 100, :]
plt.plot(img.bands.centers, pixel_signature)
plt.xlabel('波长(nm)')
plt.ylabel('反射率')
plt.show()

# 高光谱指数计算
def calculate_ndi(hyper_data, band1_idx, band2_idx):
    """任意双波段归一化差值指数"""
    band1 = hyper_data[:, :, band1_idx]
    band2 = hyper_data[:, :, band2_idx]
    return (band1 - band2) / (band1 + band2 + 1e-8)

# 红边位置计算
def red_edge_position(hyper_data, wavelengths):
    """计算红边位置"""
    # 在红边区域(680-750nm)寻找最大斜率波长
    red_edge_idx = np.where((wavelengths >= 680) & (wavelengths <= 750))[0]

    first_derivative = np.diff(hyper_data, axis=2)
    rep_idx = np.argmax(first_derivative[:, :, red_edge_idx], axis=2)
    return wavelengths[red_edge_idx][rep_idx]
```

## 图像预处理

### 大气校正

```python
# 使用6S（通过Py6S）
from Py6S import *

# 创建6S实例
s = SixS()

# 设置大气条件
s.atmos_profile = AtmosProfile.PredefinedType(AtmosProfile.MidlatitudeSummer)
s.aero_profile = AeroProfile.PredefinedType(AeroProfile.Continental)

# 设置几何参数
s.geometry = Geometry.User()
s.geometry.month = 6
s.geometry.day = 15
s.geometry.solar_z = 30
s.geometry.solar_a = 180

# 运行模拟
s.run()

# 获取校正系数
xa, xb, xc = s.outputs.coef_xa, s.outputs.coef_xb, s.outputs.coef_xc

def atmospheric_correction(dn, xa, xb, xc):
    """应用6S大气校正"""
    y = xa * dn - xb
    y = y / (1 + xc * y)
    return y
```

### 云掩膜

```python
def sentinel2_cloud_mask(s2_image):
    """生成Sentinel-2云掩膜"""
    # 使用光谱测试的简单云检测
    scl = s2_image.select('SCL')  # 场景分类层

    # 云类别: 8=云, 9=中云, 10=高云
    cloud_mask = scl.gt(7).And(scl.lt(11))

    # 附加测试: 亮度阈值
    brightness = s2_image.select(['B02','B03','B04','B08']).mean()

    return cloud_mask.Or(brightness.gt(0.4))

# 应用掩膜
def apply_mask(image):
    mask = sentinel2_cloud_mask(image)
    return image.updateMask(mask.Not())
```

### 全色锐化

```python
import cv2
import numpy as np

def gram_schmidt_pansharpen(ms, pan):
    """Gram-Schmidt全色锐化"""
    # 多光谱: (高, 宽, 波段)
    # 全色: (高, 宽)

    # 1. 将多光谱上采样至全色分辨率
    ms_up = cv2.resize(ms, (pan.shape[1], pan.shape[0]),
                       interpolation=cv2.INTER_CUBIC)

    # 2. 从多光谱模拟全色影像
    weights = np.array([0.25, 0.25, 0.25, 0.25])  # 等权重
    simulated = np.sum(ms_up * weights.reshape(1, 1, -1), axis=2)

    # 3. Gram-Schmidt正交化
    # (简化版本)
    for i in range(ms_up.shape[2]):
        band = ms_up[:, :, i].astype(float)
        mean_sim = np.mean(simulated)
        mean_band = np.mean(band)
        diff = band - mean_band
        sim_diff = simulated - mean_sim

        # 调整
        ms_up[:, :, i] = band + diff * (pan - simulated) / (np.std(sim_diff) + 1e-8)

    return ms_up
```

更多代码示例请参见[code-examples.md](code-examples.md)。
