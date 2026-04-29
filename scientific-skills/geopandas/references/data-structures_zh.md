# GeoPandas 数据结构

## GeoSeries

GeoSeries 是一个向量，其中每个条目是一组对应于一个观测的形状（类似于 pandas 的 Series，但包含几何数据）。

```python
import geopandas as gpd
from shapely.geometry import Point, Polygon

# 从几何对象创建 GeoSeries
points = gpd.GeoSeries([Point(1, 1), Point(2, 2), Point(3, 3)])

# 访问几何属性
points.area
points.length
points.bounds
```

## GeoDataFrame

GeoDataFrame 是一种表格型数据结构，包含一个 GeoSeries（类似于 pandas 的 DataFrame，但包含地理数据）。

```python
# 从字典创建
gdf = gpd.GeoDataFrame({
    'name': ['Point A', 'Point B'],
    'value': [100, 200],
    'geometry': [Point(1, 1), Point(2, 2)]
})

# 从带有坐标的 pandas DataFrame 创建
import pandas as pd
df = pd.DataFrame({'x': [1, 2, 3], 'y': [1, 2, 3], 'name': ['A', 'B', 'C']})
gdf = gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df.x, df.y))
```

## 关键属性

- **geometry**：活动几何列（可以有多个几何列）
- **crs**：坐标系参考系统
- **bounds**：所有几何图形的边界框
- **total_bounds**：整体边界框

## 设置活动几何列

当 GeoDataFrame 有多个几何列时：

```python
# 设置活动几何列
gdf = gdf.set_geometry('other_geom_column')

# 检查活动几何列
gdf.geometry.name
```

## 索引与选择

对空间数据使用标准的 pandas 索引：

```python
# 按标签选择
gdf.loc[0]

# 布尔索引
large_areas = gdf[gdf.area > 100]

# 选择列
gdf[['name', 'geometry']]
```
