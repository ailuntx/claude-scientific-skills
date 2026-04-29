---
name: geopandas
description: 用于处理地理空间矢量数据的Python库，支持Shapefile、GeoJSON和GeoPackage文件格式。适用于地理数据的空间分析、几何操作、坐标转换、空间连接、叠加操作、等值区域制图，以及任何涉及读取/写入/分析矢量地理数据的任务。支持PostGIS数据库、交互式地图，并能与matplotlib/folium/cartopy集成。可用于缓冲区分析、数据集间空间连接、边界融合、数据裁剪、面积/距离计算、坐标系重投影、地图创建或空间文件格式转换等任务。
license: BSD-3-Clause license
metadata:
    skill-author: K-Dense Inc.
---

# GeoPandas

GeoPandas扩展了pandas库，使其支持几何类型的空间操作。它结合了pandas和shapely的地理空间数据分析能力。

## 安装

```bash
uv pip install geopandas
```

### 可选依赖项

```bash
# 用于交互式地图
uv pip install folium

# 用于地图分类方案
uv pip install mapclassify

# 加速I/O操作（2-4倍性能提升）
uv pip install pyarrow

# 支持PostGIS数据库
uv pip install psycopg2
uv pip install geoalchemy2

# 用于底图
uv pip install contextily

# 用于地图投影
uv pip install cartopy
```

## 快速入门

```python
import geopandas as gpd

# 读取空间数据
gdf = gpd.read_file("data.geojson")

# 基础探索
print(gdf.head())
print(gdf.crs)
print(gdf.geometry.geom_type)

# 简单绘图
gdf.plot()

# 重投影到不同坐标系
gdf_projected = gdf.to_crs("EPSG:3857")

# 计算面积（使用投影坐标系确保精度）
gdf_projected['area'] = gdf_projected.geometry.area

# 保存文件
gdf.to_file("output.gpkg")
```

## 核心概念

### 数据结构

- **GeoSeries**: 支持空间操作的几何向量
- **GeoDataFrame**: 包含几何列的表结构数据

详见[数据结构文档](references/data-structures.md)。

### 数据读写

支持多种格式：Shapefile、GeoJSON、GeoPackage、PostGIS、Parquet。

```python
# 带过滤条件的读取
gdf = gpd.read_file("data.gpkg", bbox=(xmin, ymin, xmax, ymax))

# 使用Arrow加速写入
gdf.to_file("output.gpkg", use_arrow=True)
```

完整I/O操作见[数据输入输出文档](references/data-io.md)。

### 坐标系管理

始终检查并管理坐标系以确保空间操作精度：

```python
# 检查坐标系
print(gdf.crs)

# 重投影（坐标转换）
gdf_projected = gdf.to_crs("EPSG:3857")

# 设置坐标系（仅当元数据缺失时）
gdf = gdf.set_crs("EPSG:4326")
```

坐标系操作详见[坐标系管理文档](references/crs-management.md)。

## 常用操作

### 几何操作

缓冲区、简化、质心、凸包、仿射变换：

```python
# 创建10单位缓冲区
buffered = gdf.geometry.buffer(10)

# 带容差的几何简化
simplified = gdf.geometry.simplify(tolerance=5, preserve_topology=True)

# 获取几何质心
centroids = gdf.geometry.centroid
```

完整操作见[几何操作文档](references/geometric-operations.md)。

### 空间分析

空间连接、叠加操作、融合：

```python
# 空间连接（相交关系）
joined = gpd.sjoin(gdf1, gdf2, predicate='intersects')

# 最近邻连接
nearest = gpd.sjoin_nearest(gdf1, gdf2, max_distance=1000)

# 叠加求交
intersection = gpd.overlay(gdf1, gdf2, how='intersection')

# 按属性融合
dissolved = gdf.dissolve(by='region', aggfunc='sum')
```

分析操作详见[空间分析文档](references/spatial-analysis.md)。

### 可视化

创建静态与交互式地图：

```python
# 等值区域图
gdf.plot(column='population', cmap='YlOrRd', legend=True)

# 交互式地图
gdf.explore(column='population', legend=True).save('map.html')

# 多图层地图
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
gdf1.plot(ax=ax, color='blue')
gdf2.plot(ax=ax, color='red')
```

制图技巧见[可视化文档](references/visualization.md)。

## 详细文档

- **[数据结构](references/data-structures.md)** - GeoSeries与GeoDataFrame基础
- **[数据输入输出](references/data-io.md)** - 文件读写、PostGIS、Parquet
- **[几何操作](references/geometric-operations.md)** - 缓冲区、简化、仿射变换
- **[空间分析](references/spatial-analysis.md)** - 连接、叠加、融合、裁剪
- **[可视化](references/visualization.md)** - 绘图、等值区域图、交互式地图
- **[坐标系管理](references/crs-management.md)** - 坐标系与投影系统

## 典型工作流

### 加载→转换→分析→导出

```python
# 1. 加载数据
gdf = gpd.read_file("data.shp")

# 2. 检查并转换坐标系
print(gdf.crs)
gdf = gdf.to_crs("EPSG:3857")

# 3. 执行分析
gdf['area'] = gdf.geometry.area
buffered = gdf.copy()
buffered['geometry'] = gdf.geometry.buffer(100)

# 4. 导出结果
gdf.to_file("results.gpkg", layer='原始数据')
buffered.to_file("results.gpkg", layer='缓冲区')
```

### 空间连接与聚合

```python
# 点要素与面要素连接
points_in_polygons = gpd.sjoin(points_gdf, polygons_gdf, predicate='within')

# 按面要素聚合
aggregated = points_in_polygons.groupby('index_right').agg({
    'value': 'sum',
    'count': 'size'
})

# 合并回面要素
result = polygons_gdf.merge(aggregated, left_index=True, right_index=True)
```

### 多源数据整合

```python
# 从不同来源读取
roads = gpd.read_file("roads.shp")
buildings = gpd.read_file("buildings.geojson")
parcels = gpd.read_postgis("SELECT * FROM parcels", con=engine, geom_col='geom')

# 确保坐标系一致
buildings = buildings.to_crs(roads.crs)
parcels = parcels.to_crs(roads.crs)

# 执行空间操作
buildings_near_roads = buildings[buildings.geometry.distance(roads.union_all()) < 50]
```

## 性能优化建议

1. **启用空间索引**：GeoPandas在多数操作中自动创建空间索引
2. **读取时过滤**：使用`bbox`、`mask`或`where`参数仅加载所需数据
3. **使用Arrow加速I/O**：添加`use_arrow=True`可提升2-4倍读写速度
4. **简化几何结构**：精度要求不高时使用`.simplify()`降低复杂度
5. **批量操作**：矢量化操作远快于逐行迭代
6. **选用合适坐标系**：面积/距离计算用投影坐标系，可视化用地理坐标系

## 最佳实践

1. **空间操作前必查坐标系**
2. **面积距离计算使用投影坐标系**
3. **空间连接或叠加前确保坐标系匹配**
4. **操作前用`.is_valid`验证几何有效性**
5. **修改几何列时使用`.copy()`避免副作用**
6. **分析时简化几何需保持拓扑结构**
7. **优先选用GeoPackage格式（优于Shapefile）**
8. **sjoin_nearest中设置max_distance提升性能**
