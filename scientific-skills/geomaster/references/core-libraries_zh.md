# 核心地理空间库

本文档涵盖用于地理空间数据处理的基础Python库。

## GDAL（地理空间数据抽象库）

GDAL是Python中地理空间I/O操作的基础。

```python
from osgeo import gdal

# 打开栅格文件
ds = gdal.Open('raster.tif')
band = ds.GetRasterBand(1)
data = band.ReadAsArray()

# 获取地理变换参数
geotransform = ds.GetGeoTransform()
origin_x = geotransform[0]
pixel_width = geotransform[1]

# 获取投影信息
proj = ds.GetProjection()
```

## Rasterio

Rasterio为GDAL提供了更简洁的接口。

```python
import rasterio
import numpy as np

# 基础读取
with rasterio.open('raster.tif') as src:
    data = src.read()           # 所有波段
    band1 = src.read(1)         # 单波段
    profile = src.profile       # 元数据

# 分块读取（内存高效）
with rasterio.open('large.tif') as src:
    window = ((0, 100), (0, 100))
    subset = src.read(1, window=window)

# 写入
with rasterio.open('output.tif', 'w',
                   driver='GTiff',
                   height=data.shape[0],
                   width=data.shape[1],
                   count=1,
                   dtype=data.dtype,
                   crs=src.crs,
                   transform=src.transform) as dst:
    dst.write(data, 1)

# 掩膜处理
with rasterio.open('raster.tif') as src:
    masked_data, mask = rasterio.mask.mask(src, shapes=[polygon], crop=True)
```

## Fiona

Fiona处理矢量数据I/O操作。

```python
import fiona

# 读取要素
with fiona.open('data.geojson') as src:
    for feature in src:
        geom = feature['geometry']
        props = feature['properties']

# 获取模式与坐标系
with fiona.open('data.shp') as src:
    schema = src.schema
    crs = src.crs

# 写入数据
schema = {'geometry': 'Point', 'properties': {'name': 'str'}}
with fiona.open('output.geojson', 'w', driver='GeoJSON',
                schema=schema, crs='EPSG:4326') as dst:
    dst.write({
        'geometry': {'type': 'Point', 'coordinates': [0, 0]},
        'properties': {'name': 'Origin'}
    })
```

## Shapely

Shapely提供几何运算功能。

```python
from shapely.geometry import Point, LineString, Polygon
from shapely.ops import unary_union

# 创建几何对象
point = Point(0, 0)
line = LineString([(0, 0), (1, 1)])
poly = Polygon([(0, 0), (1, 0), (1, 1), (0, 1)])

# 几何运算
buffered = point.buffer(1)              # 缓冲区
simplified = poly.simplify(0.01)        # 简化
centroid = poly.centroid                 # 质心
intersection = poly1.intersection(poly2) # 交集

# 空间关系
point.within(poly)      # 点在多边形内时为True
poly1.intersects(poly2) # 几何体相交时为True
poly1.contains(poly2)   # poly2在poly1内时为True

# 单一合并
combined = unary_union([poly1, poly2, poly3])

# 不同连接方式的缓冲区
buffer_round = point.buffer(1, quad_segs=16)
buffer_mitre = point.buffer(1, mitre_limit=1, join_style=2)
```

## PyProj

PyProj处理坐标转换。

```python
from pyproj import Transformer, CRS

# 坐标转换
transformer = Transformer.from_crs('EPSG:4326', 'EPSG:32633')
x, y = transformer.transform(lat, lon)
x_inv, y_inv = transformer.transform(x, y, direction='INVERSE')

# 批量转换
lon_array = [-122.4, -122.3]
lat_array = [37.7, 37.8]
x_array, y_array = transformer.transform(lon_array, lat_array)

# 始终包含z/高度值
transformer_always_z = Transformer.from_crs(
    'EPSG:4326', 'EPSG:32633', always_z=True
)

# 获取CRS信息
crs = CRS.from_epsg(4326)
print(crs.name)  # WGS 84
print(crs.axis_info)  # 坐标轴信息

# 自定义转换
transformer = Transformer.from_pipeline(
    'proj=pipeline step inv proj=utm zone=32 ellps=WGS84 step proj=unitconvert xy_in=rad xy_out=deg'
)
```

## GeoPandas

GeoPandas将pandas与地理空间功能相结合。

```python
import geopandas as gpd

# 读取数据
gdf = gpd.read_file('data.geojson')
gdf = gpd.read_file('data.shp', encoding='utf-8')
gdf = gpd.read_postgis('SELECT * FROM data', con=engine)

# 写入数据
gdf.to_file('output.geojson', driver='GeoJSON')
gdf.to_file('output.gpkg', layer='data', use_arrow=True)

# CRS操作
gdf.crs  # 获取CRS
gdf = gdf.to_crs('EPSG:32633')  # 重投影
gdf = gdf.set_crs('EPSG:4326')  # 设置CRS

# 几何运算
gdf['area'] = gdf.geometry.area
gdf['length'] = gdf.geometry.length
gdf['buffer'] = gdf.geometry.buffer(100)
gdf['centroid'] = gdf.geometry.centroid

# 空间连接
joined = gpd.sjoin(gdf1, gdf2, how='inner', predicate='intersects')
joined = gpd.sjoin_nearest(gdf1, gdf2, max_distance=1000)

# 叠加操作
intersection = gpd.overlay(gdf1, gdf2, how='intersection')
union = gpd.overlay(gdf1, gdf2, how='union')
difference = gpd.overlay(gdf1, gdf2, how='difference')

# 融合
dissolved = gdf.dissolve(by='region', aggfunc='sum')

# 裁剪
clipped = gpd.clip(gdf, mask_gdf)

# 空间索引（提升性能）
idx = gdf.sindex
possible_matches = idx.intersection(polygon.bounds)
```

## 常见工作流

### 批量重投影

```python
import geopandas as gpd
from pathlib import Path

input_dir = Path('input')
output_dir = Path('output')

for shp in input_dir.glob('*.shp'):
    gdf = gpd.read_file(shp)
    gdf = gdf.to_crs('EPSG:32633')
    gdf.to_file(output_dir / shp.name)
```

### 栅格转矢量

```python
import rasterio.features
import geopandas as gpd
from shapely.geometry import shape

with rasterio.open('raster.tif') as src:
    image = src.read(1)
    results = (
        {'properties': {'value': v}, 'geometry': s}
        for s, v in rasterio.features.shapes(image, transform=src.transform)
    )

geoms = list(results)
gdf = gpd.GeoDataFrame.from_features(geoms, crs=src.crs)
```

### 矢量转栅格

```python
from rasterio.features import rasterize
import geopandas as gpd

gdf = gpd.read_file('polygons.gpkg')
shapes = ((geom, 1) for geom in gdf.geometry)

raster = rasterize(
    shapes,
    out_shape=(height, width),
    transform=transform,
    fill=0,
    dtype=np.uint8
)
```

### 合并多幅栅格

```python
import rasterio.merge
import rasterio as rio

files = ['tile1.tif', 'tile2.tif', 'tile3.tif']
datasets = [rio.open(f) for f in files]

merged, transform = rasterio.merge.merge(datasets)

# 保存
profile = datasets[0].profile
profile.update(transform=transform, height=merged.shape[1], width=merged.shape[2])

with rio.open('merged.tif', 'w', **profile) as dst:
    dst.write(merged)
```

更多详细示例请参阅[code-examples.md](code-examples.md)。
