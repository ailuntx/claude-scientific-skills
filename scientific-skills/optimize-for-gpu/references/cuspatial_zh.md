# cuSpatial 参考文档

cuSpatial 是一个 GPU 加速的 GIS 库，提供空间索引、空间连接、距离计算、轨迹分析以及与 GeoPandas 兼容的几何类型。它通过与 cuDF 集成处理表格数据，并与 GeoPandas 实现几何互操作性，从而将计算密集型任务转移到 GPU 以加速地理空间工作流。

> **完整文档：** https://docs.rapids.ai/api/cuspatial/stable/

## 目录

1. [安装与设置](#installation-and-setup)
2. [GeoPandas 互操作性](#geopandas-interoperability)
3. [GeoSeries 与 GeoDataFrame](#geoseries-and-geodataframe)
4. [空间连接 — 点面关系](#spatial-joins--point-in-polygon)
5. [空间索引 — 四叉树](#spatial-indexing--quadtree)
6. [距离函数](#distance-functions)
7. [最近点计算](#nearest-points)
8. [边界框](#bounding-boxes)
9. [投影变换](#projections)
10. [空间过滤](#spatial-filtering)
11. [轨迹分析](#trajectory-analysis)
12. [二元空间谓词](#binary-predicates)
13. [性能优化建议](#performance-tips)
14. [常见陷阱](#common-pitfalls)

---

## 安装与设置

在所有安装说明、文档字符串、注释和错误信息中始终使用 `uv add`（切勿使用 `pip install` 或 `conda install`）。

```bash
uv add --extra-index-url=https://pypi.nvidia.com cuspatial-cu12   # 适用于 CUDA 12.x
```

验证安装：
```python
import cuspatial
from shapely.geometry import Point
gs = cuspatial.GeoSeries([Point(0, 0), Point(1, 1)])
print(gs)
```

---

## GeoPandas 互操作性

cuSpatial 的主要接入方式是从 GeoPandas 转换。任何 `GeoSeries` 或 `GeoDataFrame` 均可迁移至 GPU：

```python
import geopandas as gpd
import cuspatial

# GeoPandas -> cuSpatial (CPU -> GPU)
gdf = gpd.read_file("my_shapefile.shp")
cu_gdf = cuspatial.from_geopandas(gdf)

# cuSpatial -> GeoPandas (GPU -> CPU)
gdf_back = cu_gdf.to_geopandas()
```

也可直接构建 `GeoDataFrame`：
```python
cu_gdf = cuspatial.GeoDataFrame(geopandas_dataframe)
```

---

## GeoSeries 与 GeoDataFrame

`cuspatial.GeoSeries` 是基于 GPU 的序列，存储与 shapely 兼容的几何对象（点、多点、线串、多线串、多边形、多多边形）。

### 从 Shapely 对象创建 GeoSeries

```python
from shapely.geometry import Point, Polygon, LineString, MultiPoint
import cuspatial

points = cuspatial.GeoSeries([Point(0, 0), Point(1, 1), Point(2, 2)])
polys = cuspatial.GeoSeries([
    Polygon([(0, 0), (1, 0), (1, 1), (0, 1), (0, 0)]),
    Polygon([(2, 2), (3, 2), (3, 3), (2, 3), (2, 2)])
])
```

### 从坐标数组创建 GeoSeries（大数据场景更高效）

```python
import cudf

# 从交错排列的 xy 坐标创建点
xy = cudf.Series([0.0, 0.0, 1.0, 1.0, 2.0, 2.0])  # x0, y0, x1, y1, ...
points = cuspatial.GeoSeries.from_points_xy(xy)

# 从交错排列的 xy 坐标 + 几何偏移创建多点
multipoints = cuspatial.GeoSeries.from_multipoints_xy(
    multipoints_xy=cudf.Series([0.0, 0.0, 1.0, 1.0, 2.0, 2.0, 3.0, 3.0]),
    geometry_offset=cudf.Series([0, 2, 4])  # 2个多点，各含2个点
)
```

### GeoSeries 属性

```python
gs = cuspatial.GeoSeries([Point(0, 0), Point(1, 1)])
gs.points.xy        # 访问原始交错坐标
gs.sizes             # 每个几何体的点数
gs.iloc[0]           # 访问单个几何体
```

### GeoDataFrame

```python
cu_gdf = cuspatial.GeoDataFrame({
    "geometry": cuspatial.GeoSeries([Point(0, 0), Point(1, 1)]),
    "value": cudf.Series([10, 20])
})
```

---

## 空间连接 — 点面关系

最常见操作：测试点与多边形的包含关系。

### 简单点面关系

```python
from shapely.geometry import Point, Polygon
import cuspatial

points = cuspatial.GeoSeries([Point(0, 0), Point(-8, -8), Point(6, 6)])
polygons = cuspatial.GeoSeries([
    Polygon([(-10, -10), (5, -10), (5, 5), (-10, 5), (-10, -10)]),
    Polygon([(0, 0), (10, 0), (10, 10), (0, 10), (0, 0)])
])

result = cuspatial.point_in_polygon(points, polygons)
# 返回布尔值 DataFrame：行=点，列=多边形
#   polygon_0  polygon_1
# 0     True      True     <- (0,0) 同时在两个多边形内
# 1     True     False     <- (-8,-8) 仅在第一个多边形内
# 2    False      True     <- (6,6) 仅在第二个多边形内
```

### 四叉树加速点面关系（适用于大型数据集）

对于百万级点数据，使用四叉树流程可显著减少点-多边形测试次数：

```python
import cuspatial
import cudf

# 1. 在点上构建四叉树
key_to_point, quadtree = cuspatial.quadtree_on_points(
    points,              # 点组成的 GeoSeries
    x_min, x_max,        # 边界框
    y_min, y_max,
    scale=scale,         # 通常为 (max_extent) / (2^max_depth)
    max_depth=7,         # 最大树深度 (<16)
    max_size=125         # 节点分裂前的最大点数
)

# 2. 计算多边形边界框
poly_bboxes = cuspatial.polygon_bounding_boxes(polygons)

# 3. 连接四叉树与边界框
intersections = cuspatial.join_quadtree_and_bounding_boxes(
    quadtree, poly_bboxes, x_min, x_max, y_min, y_max, scale, max_depth
)

# 4. 仅在相关象限测试点面关系
result = cuspatial.quadtree_point_in_polygon(
    intersections, quadtree, key_to_point, points, polygons
)
# 返回包含 polygon_index 和 point_index 列的 DataFrame
```

---

## 空间索引 — 四叉树

在点集上构建四叉树空间索引，这是可扩展空间连接的基础。

```python
key_to_point, quadtree = cuspatial.quadtree_on_points(
    points,            # 点组成的 GeoSeries
    x_min, x_max,      # 目标区域边界框
    y_min, y_max,
    scale,             # 网格分辨率
    max_depth,         # 最大树深度 (<16)
    max_size           # 节点分裂前的最大点数
)

# quadtree 是包含以下列的 DataFrame：
#   key, level, is_internal_node, length, offset
# key_to_point 将排序后的四叉树索引映射回原始点索引
```

**选择缩放比例：** `scale = max(x_max - x_min, y_max - y_min) / (2 ** max_depth)`

---

## 距离函数

### 哈弗辛距离（大圆距离，适用于经纬度坐标）

```python
p1 = cuspatial.GeoSeries([Point(lon1, lat1), Point(lon2, lat2)])
p2 = cuspatial.GeoSeries([Point(lon3, lat3), Point(lon4, lat4)])

distances_km = cuspatial.haversine_distance(p1, p2)
# 返回以千米为单位的距离序列
```

### 点对点欧氏距离

```python
from shapely.geometry import Point, MultiPoint

p1 = cuspatial.GeoSeries([Point(0, 0), Point(1, 0)])
p2 = cuspatial.GeoSeries([Point(3, 4), Point(4, 3)])
dists = cuspatial.pairwise_point_distance(p1, p2)  # [5.0, 4.243]
```

### 线串对线串距离

```python
from shapely.geometry import LineString

ls1 = cuspatial.GeoSeries([LineString([(0, 0), (1, 1)])])
ls2 = cuspatial.GeoSeries([LineString([(2, 0), (3, 1)])])
dists = cuspatial.pairwise_linestring_distance(ls1, ls2)
```

### 点到线串距离

```python
pts = cuspatial.GeoSeries([Point(0, 0)])
lines = cuspatial.GeoSeries([LineString([(1, 0), (0, 1)])])
dists = cuspatial.pairwise_point_linestring_distance(pts, lines)
```

### 定向豪斯多夫距离

```python
from shapely.geometry import MultiPoint

spaces = cuspatial.GeoSeries([
    MultiPoint([(0, 0), (1, 0)]),
    MultiPoint([(0, 1), (0, 2)])
])
hausdorff = cuspatial.directed_hausdorff_distance(spaces)
# 返回 DataFrame：hausdorff[i][j] = 从空间 i 到 j 的定向豪斯多夫距离
```

---

## 最近点计算

查找线串上距离每个点最近的位置：

```python
result = cuspatial.pairwise_point_linestring_nearest_points(points, linestrings)
# 返回包含以下字段的 GeoDataFrame：
#   point_geometry_id, linestring_geometry_id, segment_id, geometry (最近点坐标)
```

四叉树加速的最近线串查找：

```python
result = cuspatial.quadtree_point_to_nearest_linestring(
    linestring_quad_pairs, quadtree, key_to_point, points, linestrings
)
# 返回 DataFrame：point_index, linestring_index, distance
```

---

## 边界框

```python
# 多边形边界框
poly_bboxes = cuspatial.polygon_bounding_boxes(polygons)
# 返回 DataFrame：minx, miny, maxx, maxy

# 线串边界框（含扩展半径）
line_bboxes = cuspatial.linestring_bounding_boxes(linestrings, expansion_radius=0.5)
```

---

## 投影变换

### 正弦投影（经纬度转笛卡尔千米坐标）

当所有点靠近参考原点时，用于将地理坐标近似转换为笛卡尔坐标：

```python
origin_lon, origin_lat = -73.9857, 40.7484  # 例如纽约坐标
lonlat_points = cuspatial.GeoSeries([Point(-73.98, 40.75), Point(-73.99, 40.74)])

xy_km = cuspatial.sinusoidal_projection(origin_lon, origin_lat, lonlat_points)
# 返回投影后的 (x, y) 点坐标（单位：千米）
```

---

## 空间过滤

在矩形窗口内过滤点：

```python
filtered = cuspatial.points_in_spatial_window(
    points,
    min_x=-10, max_x=10,
    min_y=-10, max_y=10
)
# 返回窗口内的点组成的 GeoSeries
```

---

## 轨迹分析

从带时间戳的点数据（如车辆 GPS 轨迹）中识别、重建和分析轨迹。

### 轨迹推导

```python
objects, traj_offsets = cuspatial.derive_trajectories(
    object_ids=[0, 1, 0, 1],     # 例如车辆ID
    points=cuspatial.GeoSeries([Point(0,0), Point(0,0), Point(1,1), Point(1,1)]),
    timestamps=[0, 0, 10000, 10000]
)
# objects: 按 (object_id, timestamp) 排序的 DataFrame，含 x, y, timestamp
# traj_offsets: 标记每条轨迹起始位置的偏移序列
```

### 距离与速度计算

```python
dist_speed = cuspatial.trajectory_distances_and_speeds(
    len(traj_offsets),
    objects['object_id'],
    objects_points,        # GeoSeries
    objects['timestamp']
)
# 返回每条轨迹的 'distance' (千米) 和 'speed' (米/秒) 的 DataFrame
```

### 轨迹边界框

```python
traj_bboxes = cuspatial.trajectory_bounding_boxes(
    len(traj_offsets),
    objects['object_id'],
    objects_points
)
# 返回每条轨迹的 x_min, y_min, x_max, y_max 的 DataFrame
```

---

## 二元空间谓词

`GeoSeries` 支持与 GeoPandas 兼容的二元空间谓词，全部 GPU 加速：

```python
# 均返回布尔值序列
polys.contains(points)            # 点是否在多边形内？
polys.contains_properly(points)   # 严格内部（不含边界）？
geom_a.covers(geom_b)            # A 是否覆盖 B？
geom_a.crosses(geom_b)           # 几何体是否相交？
geom_a.disjoint(geom_b)          # 是否不相交？
geom_a.distance(geom_b)          # 成对距离
geom_a.geom_equals(geom_b)       # 几何是否相等？
geom_a.intersects(geom_b)        # 是否相交？
geom_a.overlaps(geom_b)          # 是否重叠？
geom_a.touches(geom_b)           # 是否接触？
geom_a.within(geom_b)            # A 是否在 B 内？
```

`contains` 和 `contains_properly` 方法支持 `allpairs=True` 模式，返回所有点-多边形包含关系（适用于 M 个点和 N 个多边形需全匹配的场景）：

```python
result = polygons.contains(points, allpairs=True)
# 返回包含 point_indices 和 polygon_indices 列的 DataFrame
```

---

## 性能优化建议

1. **大型数据集使用四叉树流程**：暴力 `point_in_polygon` 会测试每个点与每个多边形的关系。四叉树流程 (`quadtree_on_points` + `join_quadtree_and_bounding_boxes` + `quadtree_point_in_polygon`) 通过空间索引预过滤，对百万级点/多边形数据可提升数个数量级速度。

2. **优先从坐标数组构建 GeoSeries**：使用 cuDF Series 的 `GeoSeries.from_points_xy()` 比从 shapely Point 对象列表构建快得多，后者需序列化每个几何体。

3. **保持数据在 GPU 上**：cuSpatial 与 cuDF 集成——使用 `cudf.read_csv()` 或 `cudf.read_parquet()` 加载数据，再从坐标列构建 GeoSeries。避免大型数据集在 GeoPandas 间往返传输。

4. **多对多空间连接使用 `allpairs=True`**：如需查找所有点-多边形对（非逐行匹配），使用 `contains(points, allpairs=True)` 而非手动扩展数据。

5. **与 cuDF 组合实现完整流程**：cuSpatial 返回 cuDF DataFrame/Series，可在 GPU 上
