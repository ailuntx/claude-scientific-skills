# 地理空间数据源

卫星影像、矢量数据和地理空间分析API的综合目录。

## 卫星数据源

### 哨兵任务（ESA）

| 平台 | 分辨率 | 覆盖范围 | 访问地址 |
|----------|------------|----------|--------|
| **Sentinel-2** | 10-60米 | 全球 | https://scihub.copernicus.eu/ |
| **Sentinel-1** | 5-40米 (SAR) | 全球 | https://scihub.copernicus.eu/ |
| **Sentinel-3** | 300米-1公里 | 全球 | https://scihub.copernicus.eu/ |
| **Sentinel-5P** | 多种 | 全球 | https://scihub.copernicus.eu/ |

```python
# 通过Sentinelsat访问
from sentinelsat import SentinelAPI, read_geojson, geojson_to_wkt

api = SentinelAPI('用户名', '密码', 'https://scihub.copernicus.eu/dhus')

# 搜索
products = api.query(geojson_to_wkt(aoi_geojson),
                     date=('20230101', '20231231'),
                     platformname='Sentinel-2',
                     cloudcoverpercentage=(0, 20))

# 下载
api.download_all(products)
```

### 陆地卫星（USGS/NASA）

| 平台 | 分辨率 | 覆盖范围 | 访问地址 |
|----------|------------|----------|--------|
| **Landsat 9** | 30米 | 全球 | https://earthexplorer.usgs.gov/ |
| **Landsat 8** | 30米 | 全球 | https://earthexplorer.usgs.gov/ |
| **Landsat 7** | 15-60米 | 全球 | https://earthexplorer.usgs.gov/ |
| **Landsat 5-7** | 30-60米 | 全球 | https://earthexplorer.usgs.gov/ |

### 商业卫星数据

| 供应商 | 平台 | 分辨率 | API |
|----------|----------|------------|-----|
| **Planet** | PlanetScope, SkySat | 0.5-3米 | planet.com |
| **Maxar** | WorldView, GeoEye | 0.3-1.2米 | maxar.com |
| **Airbus** | Pleiades, SPOT | 0.5-2米 | airbus.com |
| **Capella** | Capella-2 (SAR) | 0.5-1米 | capellaspace.com |

## 高程数据

| 数据集 | 分辨率 | 覆盖范围 | 来源 |
|---------|------------|----------|--------|
| **AW3D30** | 30米 | 全球 | https://www.eorc.jaxa.jp/ALOS/en/aw3d30/ |
| **SRTM** | 30米 | 南纬56°-北纬60° | https://www.usgs.gov/ |
| **ASTER GDEM** | 30米 | 南纬83°-北纬83° | https://asterweb.jpl.nasa.gov/ |
| **Copernicus DEM** | 30米 | 全球 | https://copernicus.eu/ |
| **ArcticDEM** | 2-10米 | 北极 | https://www.pgc.umn.edu/ |

```python
# 通过API下载SRTM
import elevation

# 下载SRTM 1弧秒数据(30米)
elevation.clip(bounds=(-122.5, 37.7, -122.3, 37.9), output='srtm.tif')

# 清理并填补空隙
elevation.clean('srtm.tif', 'srtm_filled.tif')
```

## 土地覆盖数据

| 数据集 | 分辨率 | 分类 | 来源 |
|---------|------------|---------|--------|
| **ESA WorldCover** | 10米 | 11类 | https://worldcover2021.esa.int/ |
| **ESRI Land Cover** | 10米 | 10类 | https://www.esri.com/ |
| **Copernicus Global** | 100米 | 23类 | https://land.copernicus.eu/ |
| **MODIS MCD12Q1** | 500米 | 17类 | https://lpdaac.usgs.gov/ |
| **NLCD (美国)** | 30米 | 20类 | https://www.mrlc.gov/ |

## 气候与气象数据

### 再分析数据

| 数据集 | 分辨率 | 时间频率 | 访问地址 |
|---------|------------|----------|--------|
| **ERA5** | 31公里 | 每小时(1979+) | https://cds.climate.copernicus.eu/ |
| **MERRA-2** | 50公里 | 每小时(1980+) | https://gmao.gsfc.nasa.gov/ |
| **JRA-55** | 55公里 | 每3小时(1958+) | https://jra.kishou.go.jp/ |

```python
# 通过CDS API下载ERA5数据
import cdsapi

c = cdsapi.Client()

c.retrieve(
    'reanalysis-era5-single-levels',
    {
        'product_type': 'reanalysis',
        'variable': '2m_temperature',
        'year': '2023',
        'month': '01',
        'day': '01',
        'time': '12:00',
        'area': [37.9, -122.5, 37.7, -122.3],
        'format': 'netcdf'
    },
    'era5_temp.nc'
)
```

## OpenStreetMap数据

### 访问方法

```python
# 通过OSMnx
import osmnx as ox

# 下载区域边界
gdf = ox.geocode_to_gdf('旧金山, 加利福尼亚州')

# 下载道路网络
G = ox.graph_from_place('旧金山, 加利福尼亚州', network_type='drive')

# 下载建筑轮廓
buildings = ox.geometries_from_place('旧金山, 加利福尼亚州', tags={'building': True})

# 通过Overpass API
import requests

overpass_url = "http://overpass-api.de/api/interpreter"
query = """
    [out:json];
    way["highway"](37.7,-122.5,37.9,-122.3);
    out geom;
"""

response = requests.get(overpass_url, params={'data': query})
data = response.json()
```

## 矢量数据源

### Natural Earth

```python
import geopandas as gpd

# 行政边界 (比例尺: 10m, 50m, 110m)
countries = gpd.read_file('https://naturalearth.s3.amazonaws.com/10m_cultural/ne_10m_admin_0_countries.zip')
urban_areas = gpd.read_file('https://naturalearth.s3.amazonaws.com/10m_cultural/ne_10m_urban_areas.zip')
ports = gpd.read_file('https://naturalearth.s3.amazonaws.com/10m_cultural/ne_10m_ports.zip')
```

### 其他来源

| 数据集 | 类型 | 访问地址 |
|---------|------|--------|
| **GADM** | 行政边界 | https://gadm.org/ |
| **HydroSHEDS** | 河流、流域 | https://www.hydrosheds.org/ |
| **Global Power Plant** | 发电厂 | https://datasets.wri.org/ |
| **WorldPop** | 人口 | https://www.worldpop.org/ |
| **GPW** | 人口 | https://sedac.ciesin.columbia.edu/ |
| **HDX** | 人道主义数据 | https://data.humdata.org/ |

## API接口

### 谷歌地图平台

```python
import requests

# 地理编码
url = "https://maps.googleapis.com/maps/api/geocode/json"
params = {
    'address': '金门大桥',
    'key': YOUR_API_KEY
}

response = requests.get(url, params=params)
data = response.json()
location = data['results'][0]['geometry']['location']
```

### Mapbox

```python
# 地理编码
import requests

url = "https://api.mapbox.com/geocoding/v5/mapbox.places/Golden%20Gate%20Bridge.json"
params = {'access_token': YOUR_ACCESS_TOKEN}

response = requests.get(url, params=params)
data = response.json()
```

### OpenWeatherMap

```python
# 实时天气
url = "https://api.openweathermap.org/data/2.5/weather"
params = {
    'lat': 37.7,
    'lon': -122.4,
    'appid': YOUR_API_KEY
}

response = requests.get(url, params=params)
weather = response.json()
```

## Python数据API

### STAC（时空资产目录）

```python
import pystac_client

# 连接STAC目录
catalog = pystac_client.Client.open("https://earth-search.aws.element84.com/v1")

# 搜索
search = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=[-122.5, 37.7, -122.3, 37.9],
    datetime="2023-01-01/2023-12-31",
    query={"eo:cloud_cover": {"lt": 20}}
)

items = search.get_all_items()
```

### Planetary Computer

```python
import planetary_computer
import pystac_client

catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace
)

# 搜索并签名项目
items = catalog.search(...)
signed_items = [planetary_computer.sign(item) for item in items]
```

## 下载脚本

### 自动化下载脚本

```python
from sentinelsat import SentinelAPI
import rasterio
from rasterio.warp import calculate_default_transform, reproject, Resampling
import os

def download_and_process_sentinel2(aoi, date_range, output_dir):
    """
    下载并处理Sentinel-2影像
    """
    # 初始化API
    api = SentinelAPI('用户名', '密码', 'https://scihub.copernicus.eu/dhus')

    # 搜索
    products = api.query(
        aoi,
        date=date_range,
        platformname='Sentinel-2',
        processinglevel='Level-2A',
        cloudcoverpercentage=(0, 20)
    )

    # 下载
    api.download_all(products, directory_path=output_dir)

    # 处理每个产品
    for product in products:
        product_path = f"{output_dir}/{product['identifier']}.SAFE"
        processed = process_sentinel2_product(product_path)
        save_rgb_composite(processed, f"{output_dir}/{product['identifier']}_rgb.tif")

def process_sentinel2_product(product_path):
    """处理Sentinel-2 L2A产品"""
    # 查找10米波段(B02, B03, B04, B08)
    bands = {}
    for band_id in ['B02', 'B03', 'B04', 'B08']:
        band_path = find_band_file(product_path, band_id, resolution='10m')
        with rasterio.open(band_path) as src:
            bands[band_id] = src.read(1)
            profile = src.profile

    # 堆叠波段
    stacked = np.stack([bands['B04'], bands['B03'], bands['B02']])  # RGB

    return stacked, profile
```

## 数据质量评估

```python
def assess_data_quality(raster_path):
    """
    评估地理空间栅格数据质量
    """
    import rasterio
    import numpy as np

    with rasterio.open(raster_path) as src:
        data = src.read()
        profile = src.profile

    quality_report = {
        'nodata_percentage': np.sum(data == src.nodata) / data.size * 100,
        'data_range': (data.min(), data.max()),
        'mean': np.mean(data),
        'std': np.std(data),
        'has_gaps': np.any(data == src.nodata),
        'projection': profile['crs'],
        'resolution': (profile['transform'][0], abs(profile['transform'][4]))
    }

    return quality_report
```

数据访问代码示例详见[code-examples.md](code-examples.md)。
