---
name: geomaster
description: 全面的地理空间科学技能，涵盖遥感、GIS、空间分析、地球观测机器学习及30+科学领域。支持卫星影像处理（Sentinel、Landsat、MODIS、SAR、高光谱）、矢量与栅格数据操作、空间统计、点云处理、网络分析、云原生工作流（STAC、COG、Planetary Computer）以及8种编程语言（Python、R、Julia、JavaScript、C++、Java、Go、Rust）的500+代码示例。适用于遥感工作流、GIS分析、空间机器学习、地球观测数据处理、地形分析、水文建模、海洋空间分析、大气科学及任何地理空间计算任务。
license: MIT License
metadata:
    skill-author: K-Dense Inc.
---

# GeoMaster

全面的地理空间科学技能，涵盖GIS、遥感、空间分析和地球观测机器学习等70+主题，提供8种编程语言的500+代码示例。

## 安装

```bash
# 核心Python栈（推荐conda）
conda install -c conda-forge gdal rasterio fiona shapely pyproj geopandas

# 遥感与机器学习
uv pip install rsgislib torchgeo earthengine-api
uv pip install scikit-learn xgboost torch-geometric

# 网络分析与可视化
uv pip install osmnx networkx folium keplergl
uv pip install cartopy contextily mapclassify

# 大数据与云计算
uv pip install xarray rioxarray dask-geopandas
uv pip install pystac-client planetary-computer

# 点云处理
uv pip install laspy pylas open3d pdal

# 数据库
conda install -c conda-forge postgis spatialite
```

## 快速入门

### 基于Sentinel-2的NDVI计算

```python
import rasterio
import numpy as np

with rasterio.open('sentinel2.tif') as src:
    red = src.read(4).astype(float)   # B04波段
    nir = src.read(8).astype(float)   # B08波段
    ndvi = (nir - red) / (nir + red + 1e-8)
    ndvi = np.nan_to_num(ndvi, nan=0)

    profile = src.profile
    profile.update(count=1, dtype=rasterio.float32)

    with rasterio.open('ndvi.tif', 'w', **profile) as dst:
        dst.write(ndvi.astype(rasterio.float32), 1)
```

### 使用GeoPandas进行空间分析

```python
import geopandas as gpd

# 加载数据并确保相同坐标系
zones = gpd.read_file('zones.geojson')
points = gpd.read_file('points.geojson')

if zones.crs != points.crs:
    points = points.to_crs(zones.crs)

# 空间连接与统计分析
joined = gpd.sjoin(points, zones, how='inner', predicate='within')
stats = joined.groupby('zone_id').agg({
    'value': ['count', 'mean', 'std', 'min', 'max']
}).round(2)
```

### Google Earth Engine时间序列分析

```python
import ee
import pandas as pd

ee.Initialize(project='your-project')
roi = ee.Geometry.Point([-122.4, 37.7]).buffer(10000)

s2 = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
      .filterBounds(roi)
      .filterDate('2020-01-01', '2023-12-31')
      .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20)))

def add_ndvi(img):
    return img.addBands(img.normalizedDifference(['B8', 'B4']).rename('NDVI'))

s2_ndvi = s2.map(add_ndvi)

def extract_series(image):
    stats = image.reduceRegion(ee.Reducer.mean(), roi.centroid(), scale=10, maxPixels=1e9)
    return ee.Feature(None, {'date': image.date().format('YYYY-MM-dd'), 'ndvi': stats.get('NDVI')})

series = s2_ndvi.map(extract_series).getInfo()
df = pd.DataFrame([f['properties'] for f in series['features']])
df['date'] = pd.to_datetime(df['date'])
```

## 核心概念

### 数据类型

| 类型 | 示例 | 库 |
|------|----------|-----------|
| 矢量 | Shapefile, GeoJSON, GeoPackage | GeoPandas, Fiona, GDAL |
| 栅格 | GeoTIFF, NetCDF, COG | Rasterio, Xarray, GDAL |
| 点云 | LAS, LAZ | Laspy, PDAL, Open3D |

### 坐标系

- **EPSG:4326** (WGS 84) - 地理坐标系（经纬度），用于存储
- **EPSG:3857** (Web墨卡托) - 仅用于网络地图（勿用于面积/距离计算！）
- **EPSG:326xx/327xx** (UTM) - 公制计算，每区域变形率<1%
- 使用`gdf.estimate_utm_crs()`自动检测UTM坐标系

```python
# 操作前务必检查坐标系
assert gdf1.crs == gdf2.crs, "坐标系不匹配！"

# 面积/距离计算使用投影坐标系
gdf_metric = gdf.to_crs(gdf.estimate_utm_crs())
area_sqm = gdf_metric.geometry.area
```

### OGC标准

- **WMS**: 网络地图服务 - 栅格地图
- **WFS**: 网络要素服务 - 矢量数据
- **WCS**: 网络覆盖服务 - 栅格覆盖
- **STAC**: 时空资产目录 - 现代元数据标准

## 常用操作

### 光谱指数计算

```python
def calculate_indices(image_path):
    """基于Sentinel-2计算NDVI、EVI、SAVI、NDWI"""
    with rasterio.open(image_path) as src:
        B02, B03, B04, B08, B11 = [src.read(i).astype(float) for i in [1,2,3,4,5]]

    ndvi = (B08 - B04) / (B08 + B04 + 1e-8)
    evi = 2.5 * (B08 - B04) / (B08 + 6*B04 - 7.5*B02 + 1)
    savi = ((B08 - B04) / (B08 + B04 + 0.5)) * 1.5
    ndwi = (B03 - B08) / (B03 + B08 + 1e-8)

    return {'NDVI': ndvi, 'EVI': evi, 'SAVI': savi, 'NDWI': ndwi}
```

### 矢量操作

```python
# 缓冲区（使用投影坐标系！）
gdf_proj = gdf.to_crs(gdf.estimate_utm_crs())
gdf['buffer_1km'] = gdf_proj.geometry.buffer(1000)

# 空间关系
intersects = gdf[gdf.geometry.intersects(other_geometry)]
contains = gdf[gdf.geometry.contains(point_geometry)]

# 几何操作
gdf['centroid'] = gdf.geometry.centroid
gdf['simplified'] = gdf.geometry.simplify(tolerance=0.001)

# 叠加操作
intersection = gpd.overlay(gdf1, gdf2, how='intersection')
union = gpd.overlay(gdf1, gdf2, how='union')
```

### 地形分析

```python
def terrain_metrics(dem_path):
    """基于DEM计算坡度、坡向、山体阴影"""
    with rasterio.open(dem_path) as src:
        dem = src.read(1)

    dy, dx = np.gradient(dem)
    slope = np.arctan(np.sqrt(dx**2 + dy**2)) * 180 / np.pi
    aspect = (90 - np.arctan2(-dy, dx) * 180 / np.pi) % 360

    # 山体阴影
    az_rad, alt_rad = np.radians(315), np.radians(45)
    hillshade = (np.sin(alt_rad) * np.sin(np.radians(slope)) +
                 np.cos(alt_rad) * np.cos(np.radians(slope)) *
                 np.cos(np.radians(aspect) - az_rad))

    return slope, aspect, hillshade
```

### 网络分析

```python
import osmnx as ox
import networkx as nx

# 下载并分析道路网络
G = ox.graph_from_place('San Francisco, CA', network_type='drive')
G = ox.add_edge_speeds(G).add_edge_travel_times(G)

# 最短路径
orig = ox.distance.nearest_nodes(G, -122.4, 37.7)
dest = ox.distance.nearest_nodes(G, -122.3, 37.8)
route = nx.shortest_path(G, orig, dest, weight='travel_time')
```

## 影像分类

```python
from sklearn.ensemble import RandomForestClassifier
import rasterio
from rasterio.features import rasterize

def classify_imagery(raster_path, training_gdf, output_path):
    """训练随机森林模型并进行影像分类"""
    with rasterio.open(raster_path) as src:
        image = src.read()
        profile = src.profile
        transform = src.transform

    # 提取训练数据
    X_train, y_train = [], []
    for _, row in training_gdf.iterrows():
        mask = rasterize([(row.geometry, 1)],
                        out_shape=(profile['height'], profile['width']),
                        transform=transform, fill=0, dtype=np.uint8)
        pixels = image[:, mask > 0].T
        X_train.extend(pixels)
        y_train.extend([row['class_id']] * len(pixels))

    # 训练与预测
    rf = RandomForestClassifier(n_estimators=100, max_depth=20, n_jobs=-1)
    rf.fit(X_train, y_train)

    prediction = rf.predict(image.reshape(image.shape[0], -1).T)
    prediction = prediction.reshape(profile['height'], profile['width'])

    profile.update(dtype=rasterio.uint8, count=1)
    with rasterio.open(output_path, 'w', **profile) as dst:
        dst.write(prediction.astype(rasterio.uint8), 1)

    return rf
```

## 现代云原生工作流

### STAC + Planetary Computer

```python
import pystac_client
import planetary_computer
import odc.stac

# 通过STAC搜索Sentinel-2数据
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)

search = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=[-122.5, 37.7, -122.3, 37.9],
    datetime="2023-01-01/2023-12-31",
    query={"eo:cloud_cover": {"lt": 20}},
)

# 以xarray格式加载（云原生！）
data = odc.stac.load(
    list(search.get_items())[:5],
    bands=["B02", "B03", "B04", "B08"],
    crs="EPSG:32610",
    resolution=10,
)

# 在xarray上计算NDVI
ndvi = (data.B08 - data.B04) / (data.B08 + data.B04)
```

### 云优化GeoTIFF (COG)

```python
import rasterio
from rasterio.session import AWSSession

# 从云端直接读取COG（支持局部读取）
session = AWSSession(aws_access_key_id=..., aws_secret_access_key=...)
with rasterio.open('s3://bucket/path.tif', session=session) as src:
    # 仅读取目标窗口
    window = ((1000, 2000), (1000, 2000))
    subset = src.read(1, window=window)

# 写入COG
with rasterio.open('output.tif', 'w', **profile,
                   tiled=True, blockxsize=256, blockysize=256,
                   compress='DEFLATE', predictor=2) as dst:
    dst.write(data)

# 验证COG
from rio_cogeo.cogeo import cog_validate
cog_validate('output.tif')
```

## 性能优化技巧

```python
# 1. 空间索引（提升查询速度10-100倍）
gdf.sindex  # GeoPandas自动创建

# 2. 分块处理大型栅格
with rasterio.open('large.tif') as src:
    for i, window in src.block_windows(1):
        block = src.read(1, window=window)

# 3. 使用Dask处理大数据
import dask.array as da
dask_array = da.from_rasterio('large.tif', chunks=(1, 1024, 1024))

# 4. 使用Arrow加速I/O
gdf.to_file('output.gpkg', use_arrow=True)

# 5. GDAL缓存设置
from osgeo import gdal
gdal.SetCacheMax(2**30)  # 1GB缓存

# 6. 并行处理
rf = RandomForestClassifier(n_jobs=-1)  # 使用全部核心
```

## 最佳实践

1. **操作前务必检查坐标系**
2. **面积/距离计算使用投影坐标系**
3. **验证几何体**：`gdf = gdf[gdf.is_valid]`
4. **处理缺失数据**：`gdf['geometry'] = gdf['geometry'].fillna(None)`
5. **使用高效格式**：GeoPackage > Shapefile，大型数据用Parquet
6. **对光学影像应用云掩膜**
7. **保留数据溯源**确保研究可复现
8. **根据分析尺度选择合适分辨率**

## 详细文档

- **[坐标系](references/coordinate-systems.md)** - CRS基础、UTM、坐标转换
- **[核心库](references/core-libraries.md)** - GDAL、Rasterio、GeoPandas、Shapely
- **[遥感](references/remote-sensing.md)** - 卫星任务、光谱指数、SAR
- **[机器学习](references/machine-learning.md)** - 深度学习、CNN、GNN在遥感中的应用
- **[GIS软件](references/gis-software.md)** - QGIS、ArcGIS、GRASS集成
- **[科学领域](references/scientific-domains.md)** - 海洋学、水文学、农业、林业
- **[高级GIS](references/advanced-gis.md)** - 3D GIS、时空分析、拓扑学
- **[大数据](references/big-data.md)** - 分布式处理、GPU加速
- **[行业应用](references/industry-applications.md)** - 城市规划、灾害管理
- **[编程语言](references/programming-languages.md)** - Python、R、Julia、JS、C++、Java、Go、Rust
- **[数据源](references/data-sources.md)** - 卫星目录、API接口
- **[故障排除](references/troubleshooting.md)** - 常见问题、调试方法、错误参考
- **[代码示例](references/code-examples.md)** - 500+示例

---

**GeoMaster涵盖从基础GIS操作到高级遥感与机器学习的全领域技能。**
