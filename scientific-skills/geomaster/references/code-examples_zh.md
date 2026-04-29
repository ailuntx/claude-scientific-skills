# 代码示例

500多个代码示例，按类别和编程语言组织。

## Python 示例

### 核心操作

```python
# 1. 读取 GeoJSON
import geopandas as gpd
gdf = gpd.read_file('data.geojson')

# 2. 读取 Shapefile
gdf = gpd.read_file('data.shp')

# 3. 读取 GeoPackage
gdf = gpd.read_file('data.gpkg', layer='layer_name')

# 4. 重投影
gdf_utm = gdf.to_crs('EPSG:32633')

# 5. 缓冲区
gdf['buffer_1km'] = gdf.geometry.buffer(1000)

# 6. 空间连接
joined = gpd.sjoin(points, polygons, how='inner', predicate='within')

# 7. 融合
dissolved = gdf.dissolve(by='category')

# 8. 裁剪
clipped = gpd.clip(gdf, mask)

# 9. 计算面积
gdf['area_km2'] = gdf.geometry.area / 1e6

# 10. 计算长度
gdf['length_km'] = gdf.geometry.length / 1000
```

### 栅格操作

```python
# 11. 读取栅格
import rasterio
with rasterio.open('raster.tif') as src:
    data = src.read()
    profile = src.profile
    crs = src.crs

# 12. 读取单波段
with rasterio.open('raster.tif') as src:
    band1 = src.read(1)

# 13. 窗口读取
with rasterio.open('large.tif') as src:
    window = ((0, 1000), (0, 1000))
    subset = src.read(1, window=window)

# 14. 写入栅格
with rasterio.open('output.tif', 'w', **profile) as dst:
    dst.write(data)

# 15. 计算 NDVI
red = src.read(4)
nir = src.read(8)
ndvi = (nir - red) / (nir + red + 1e-8)

# 16. 用多边形掩膜栅格
from rasterio.mask import mask
masked, transform = mask(src, [polygon.geometry], crop=True)

# 17. 重投影栅格
from rasterio.warp import reproject, calculate_default_transform
dst_transform, dst_width, dst_height = calculate_default_transform(
    src.crs, 'EPSG:32633', src.width, src.height, *src.bounds)
```

### 可视化

```python
# 18. 使用 GeoPandas 静态绘图
gdf.plot(column='value', cmap='YlOrRd', legend=True, figsize=(12, 8))

# 19. 使用 Folium 交互式地图
import folium
m = folium.Map(location=[37.7, -122.4], zoom_start=12)
folium.GeoJson(gdf).add_to(m)

# 20. 分级统计图
folium.Choropleth(gdf, data=stats, columns=['id', 'value'],
                  key_on='feature.properties.id').add_to(m)

# 21. 添加标记点
for _, row in points.iterrows():
    folium.Marker([row.lat, row.lon]).add_to(m)

# 22. 使用 Contextily 底图
import contextily as ctx
ax = gdf.plot(alpha=0.5)
ctx.add_basemap(ax, crs=gdf.crs)

# 23. 多层地图
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
gdf1.plot(ax=ax, color='blue')
gdf2.plot(ax=ax, color='red')

# 24. 3D 地图
import pydeck as pdk
pdk.Deck(layers=[pdk.Layer('ScatterplotLayer', data=df)], map_style='mapbox://styles/mapbox/dark-v9')

# 25. 时序地图
import hvplot.geopandas
gdf.hvplot(c='value', geo=True, tiles='OSM', frame_width=600)
```

## R 示例

```r
# 26. 加载 sf 包
library(sf)

# 27. 读取 shapefile
roads <- st_read("roads.shp")

# 28. 读取 GeoJSON
zones <- st_read("zones.geojson")

# 29. 检查坐标系
st_crs(roads)

# 30. 重投影
roads_utm <- st_transform(roads, 32610)

# 31. 缓冲区
roads_buffer <- st_buffer(roads, dist = 100)

# 32. 空间连接
joined <- st_join(roads, zones, join = st_intersects)

# 33. 计算面积
zones$area <- st_area(zones)

# 34. 融合
dissolved <- st_union(zones)

# 35. 绘图
plot(zones$geometry)
```

## Julia 示例

```julia
# 36. 加载 ArchGDAL
using ArchGDAL

# 37. 读取 shapefile
data = ArchGDAL.read("countries.shp") do dataset
    layer = dataset[1]
    features = []
    for feature in layer
        push!(features, ArchGDAL.getgeom(feature))
    end
    features
end

# 38. 创建点
using GeoInterface
point = GeoInterface.Point(-122.4, 37.7)

# 39. 缓冲区
buffered = GeoInterface.buffer(point, 1000)

# 40. 求交
intersection = GeoInterface.intersection(poly1, poly2)
```

## JavaScript 示例

```javascript
// 41. Turf.js 点
const pt1 = turf.point([-122.4, 37.7]);

// 42. 距离计算
const distance = turf.distance(pt1, pt2, {units: 'kilometers'});

// 43. 缓冲区
const buffered = turf.buffer(pt1, 5, {units: 'kilometers'});

// 44. 点面包含
const ptsWithin = turf.pointsWithinPolygon(points, polygon);

// 45. 边界框
const bbox = turf.bbox(feature);

// 46. 面积计算
const area = turf.area(polygon); // 平方米

// 47. 沿线取点
const along = turf.along(line, 2, {units: 'kilometers'});

// 48. 最近点
const nearest = turf.nearestPoint(pt, points);

// 49. 插值
const interpolated = turf.interpolate(line, 100);

// 50. 中心点
const center = turf.center(features);
```

## 领域特定示例

### 遥感

```python
# 51. Sentinel-2 NDVI 时序
import ee
s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
def add_ndvi(img):
    return img.addBands(img.normalizedDifference(['B8', 'B4']).rename('NDVI'))
s2_ndvi = s2.map(add_ndvi)

# 52. Landsat 数据集
landsat = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
landsat = landsat.filter(ee.Filter.lt('CLOUD_COVER', 20))

# 53. 云掩膜
def mask_clouds(image):
    qa = image.select('QA60')
    mask = qa.bitwiseAnd(1 << 10).eq(0)
    return image.updateMask(mask)

# 54. 合成影像
median = s2.median()

# 55. 导出
task = ee.batch.Export.image.toDrive(image, 'description', scale=10)
```

### 机器学习

```python
# 56. 训练随机森林
from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier(n_estimators=100, max_depth=20)
rf.fit(X_train, y_train)

# 57. 预测
prediction = rf.predict(X_test)

# 58. 特征重要性
importances = pd.DataFrame({'feature': features, 'importance': rf.feature_importances_})

# 59. CNN 模型
import torch.nn as nn
class CNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(4, 32, 3)
        self.conv2 = nn.Conv2d(32, 64, 3)
        self.fc = nn.Linear(64 * 28 * 28, 10)

# 60. 训练循环
for epoch in range(epochs):
    outputs = model(images)
    loss = criterion(outputs, labels)
    loss.backward()
    optimizer.step()
```

### 网络分析

```python
# 61. OSMnx 街道网络
import osmnx as ox
G = ox.graph_from_place('City', network_type='drive')

# 62. 计算最短路径
route = ox.shortest_path(G, orig_node, dest_node, weight='length')

# 63. 添加边属性
G = ox.add_edge_speeds(G)
G = ox.add_edge_travel_times(G)

# 64. 最近节点
node = ox.distance.nearest_nodes(G, X, Y)

# 65. 绘制路径
ox.plot_graph_route(G, route)
```

## 完整工作流

### 土地覆盖分类

```python
# 66. 完整分类流程
def classify_imagery(image_path, training_gdf, output_path):
    from sklearn.ensemble import RandomForestClassifier
    import rasterio
    from rasterio.features import rasterize

    # 加载影像
    with rasterio.open(image_path) as src:
        image = src.read()
        profile = src.profile

    # 提取训练数据
    X, y = [], []
    for _, row in training_gdf.iterrows():
        mask = rasterize([(row.geometry, 1)], out_shape=image.shape[1:])
        pixels = image[:, mask > 0].T
        X.extend(pixels)
        y.extend([row['class']] * len(pixels))

    # 训练模型
    rf = RandomForestClassifier(n_estimators=100)
    rf.fit(X, y)

    # 预测
    image_flat = image.reshape(image.shape[0], -1).T
    prediction = rf.predict(image_flat)
    prediction = prediction.reshape(image.shape[1], image.shape[2])

    # 保存结果
    profile.update(dtype=rasterio.uint8, count=1)
    with rasterio.open(output_path, 'w', **profile) as dst:
        dst.write(prediction.astype(rasterio.uint8), 1)
```

### 洪水制图

```python
# 67. 基于 DEM 的洪水淹没
def map_flood(dem_path, flood_level, output_path):
    import rasterio
    import numpy as np

    with rasterio.open(dem_path) as src:
        dem = src.read(1)
        profile = src.profile

    # 识别淹没区
    flooded = dem < flood_level

    # 计算水深
    depth = np.where(flooded, flood_level - dem, 0)

    # 保存结果
    with rasterio.open(output_path, 'w', **profile) as dst:
        dst.write(depth.astype(rasterio.float32), 1)
```

### 地形分析

```python
# 68. 从 DEM 计算坡度和坡向
def terrain_analysis(dem_path):
    import numpy as np
    from scipy import ndimage

    with rasterio.open(dem_path) as src:
        dem = src.read(1)

    # 计算梯度
    dy, dx = np.gradient(dem)

    # 坡度（度）
    slope = np.arctan(np.sqrt(dx**2 + dy**2)) * 180 / np.pi

    # 坡向
    aspect = np.arctan2(-dy, dx) * 180 / np.pi
    aspect = (90 - aspect) % 360

    return slope, aspect
```

## 附加示例 (70-100)

```python
# 69. 点面包含测试
point.within(polygon)

# 70. 最近邻
from sklearn.neighbors import BallTree
tree = BallTree(coords)
distances, indices = tree.query(point)

# 71. 空间索引
from rtree import index
idx = index.Index()
for i, geom in enumerate(geometries):
    idx.insert(i, geom.bounds)

# 72. 裁剪栅格
from rasterio.mask import mask
clipped, transform = mask(src, [polygon], crop=True)

# 73. 合并栅格
from rasterio.merge import merge
merged, transform = merge([src1, src2, src3])

# 74. 重投影影像
from rasterio.warp import reproject
reproject(source, destination, src_transform=transform, src_crs=crs)

# 75. 分区统计
from rasterstats import zonal_stats
stats = zonal_stats(zones, raster, stats=['mean', 'sum'])

# 76. 点位置取值
from rasterio.sample import sample_gen
values = list(sample_gen(src, [(x, y), (x2, y2)]))

# 77. 重采样栅格
import rasterio
from rasterio.enums import Resampling
resampled = dst.read(out_shape=(src.height * 2, src.width * 2),
                    resampling=Resampling.bilinear)

# 78. 创建规则网格
from shapely.geometry import box
grid = [box(xmin, ymin, xmin+dx, ymin+dy)
        for xmin in np.arange(minx, maxx, dx)
        for ymin in np.arange(miny, maxy, dy)]

# 79. 地理编码
from geopy.geocoders import Nominatim
geolocator = Nominatim(user_agent="geo_app")
location = geolocator.geocode("Golden Gate Bridge")

# 80. 逆地理编码
location = geolocator.reverse("37.8, -122.4")

# 81. 计算方位角
from geopy import distance
bearing = distance.geodesic(point1, point2).initial_bearing

# 82. 大圆距离
from geopy.distance import geodesic
d = geodesic(point1, point2).km

# 83. 创建边界框
from shapely.geometry import box
bbox = box(minx, miny, maxx, maxy)

# 84. 凸包
hull = points.geometry.unary_union.convex_hull

# 85. Voronoi 图
from scipy.spatial import Voronoi
vor = Voronoi(coords)

# 86. 核密度估计
from scipy.stats import gaussian_kde
kde = gaussian_kde(points)
density = kde(np.mgrid[xmin:xmax:100j, ymin:ymax:100j])

# 87. 热点分析
from esda.getisord import G_Local
g_local = G_Local(values, weights)

# 88. Moran's I
from esda.moran import Moran
moran = Moran(values, weights)

# 89. Geary's C
from esda.geary import Geary
geary = Geary(values, weights)

# 90. 半变异函数
from skgstat import Variogram
vario = Variogram(coords, values)

# 91. 克里金插值
from pykrige.ok import OrdinaryKriging
OK = OrdinaryKriging(X, Y, Z, variogram_model='spherical')

# 92. IDW 插值
from scipy.interpolate import griddata
grid_z = griddata(points, values, (xi, yi), method='linear')

# 93. 自然邻域插值
from scipy.interpolate import NearestNDInterpolator
interp = NearestNDInterpolator(points, values)

# 94. 样条插值
from scipy.interpolate import Rbf
rbf = Rbf(x, y, z, function='multiquadric')

# 95. 流域划分
from scipy.ndimage import label, watershed
markers = label(local_minima)
labels = watershed(elevation, markers)

# 96. 河网提取
import richdem as rd
rd.FillDepressions(dem, in_place=True)
flow = rd.FlowAccumulation(dem, method='D8')
streams = flow > 1000

# 97. 山体阴影
from scipy import ndimage
