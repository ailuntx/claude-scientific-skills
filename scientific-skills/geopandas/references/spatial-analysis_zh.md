# 空间分析

## 属性连接

基于公共变量使用标准 pandas 合并方法组合数据集：

```python
# 按公共列合并
result = gdf.merge(df, on='common_column')

# 左连接
result = gdf.merge(df, on='common_column', how='left')

# 重要：在GeoDataFrame上调用merge以保留几何属性
# 正确方式：gdf.merge(df, ...)
# 错误方式：df.merge(gdf, ...) # 返回DataFrame而非GeoDataFrame
```

## 空间连接

基于空间关系组合数据集。

### 二元谓词连接 (sjoin)

基于几何谓词进行连接：

```python
# 相交（默认）
joined = gpd.sjoin(gdf1, gdf2, how='inner', predicate='intersects')

# 可用谓词
joined = gpd.sjoin(gdf1, gdf2, predicate='contains')
joined = gpd.sjoin(gdf1, gdf2, predicate='within')
joined = gpd.sjoin(gdf1, gdf2, predicate='touches')
joined = gpd.sjoin(gdf1, gdf2, predicate='crosses')
joined = gpd.sjoin(gdf1, gdf2, predicate='overlaps')

# 连接类型
joined = gpd.sjoin(gdf1, gdf2, how='left')   # 保留左侧所有要素
joined = gpd.sjoin(gdf1, gdf2, how='right')  # 保留右侧所有要素
joined = gpd.sjoin(gdf1, gdf2, how='inner')  # 仅保留交集要素
```

`how` 参数决定保留哪些几何要素：
- **left**：保留左侧GeoDataFrame的索引和几何
- **right**：保留右侧GeoDataFrame的索引和几何
- **inner**：使用索引交集，保留左侧几何

### 最近邻连接 (sjoin_nearest)

连接至最近要素：

```python
# 查找最近邻
nearest = gpd.sjoin_nearest(gdf1, gdf2)

# 添加距离列
nearest = gpd.sjoin_nearest(gdf1, gdf2, distance_col='distance')

# 限制搜索半径（显著提升性能）
nearest = gpd.sjoin_nearest(gdf1, gdf2, max_distance=1000)

# 查找k个最近邻
nearest = gpd.sjoin_nearest(gdf1, gdf2, k=5)
```

## 叠加操作

对两个GeoDataFrame的几何执行集合运算：

```python
# 交集 - 保留重叠区域
intersection = gpd.overlay(gdf1, gdf2, how='intersection')

# 并集 - 合并所有区域
union = gpd.overlay(gdf1, gdf2, how='union')

# 差集 - 保留第一个数据集独有的区域
difference = gpd.overlay(gdf1, gdf2, how='difference')

# 对称差集 - 保留仅出现在一个数据集中的区域
sym_diff = gpd.overlay(gdf1, gdf2, how='symmetric_difference')

# 标识 - 交集与差集的组合
identity = gpd.overlay(gdf1, gdf2, how='identity')
```

结果包含两个输入GeoDataFrame的属性。

## 融合（聚合）

基于属性值聚合几何要素：

```python
# 按属性融合
dissolved = gdf.dissolve(by='region')

# 带聚合函数的融合
dissolved = gdf.dissolve(by='region', aggfunc='sum')
dissolved = gdf.dissolve(by='region', aggfunc={'population': 'sum', 'area': 'mean'})

# 融合为单一几何
dissolved = gdf.dissolve()

# 保留内部边界
dissolved = gdf.dissolve(by='region', as_index=False)
```

## 裁剪

将几何要素裁剪至另一几何边界：

```python
# 裁剪至多边形边界
clipped = gpd.clip(gdf, boundary_polygon)

# 裁剪至另一GeoDataFrame
clipped = gpd.clip(gdf, boundary_gdf)
```

## 追加

组合多个GeoDataFrame：

```python
import pandas as pd

# 拼接GeoDataFrame（坐标系必须匹配）
combined = pd.concat([gdf1, gdf2], ignore_index=True)

# 添加标识键
combined = pd.concat([gdf1, gdf2], keys=['source1', 'source2'])
```

## 空间索引

提升空间操作性能：

```python
# GeoPandas在多数操作中自动使用空间索引
# 直接访问空间索引
sindex = gdf.sindex

# 查询与边界框相交的几何
possible_matches_index = list(sindex.intersection((xmin, ymin, xmax, ymax)))
possible_matches = gdf.iloc[possible_matches_index]

# 查询与多边形相交的几何
possible_matches_index = list(sindex.query(polygon_geometry))
possible_matches = gdf.iloc[possible_matches_index]
```

空间索引显著加速以下操作：
- 空间连接
- 叠加操作
- 几何谓词查询

## 距离计算

```python
# 几何要素间距离
distances = gdf1.geometry.distance(gdf2.geometry)

# 到单一几何的距离
distances = gdf.geometry.distance(single_point)

# 到最近要素的最小距离
min_dist = gdf.geometry.distance(point).min()
```

## 面积与长度计算

确保使用合适的坐标系以获取精确测量值：

```python
# 重投影至适当投影坐标系进行面积/长度计算
gdf_projected = gdf.to_crs(epsg=3857)  # 或合适的UTM分区

# 计算面积（使用坐标系单位，通常为平方米）
areas = gdf_projected.geometry.area

# 计算长度/周长（使用坐标系单位）
lengths = gdf_projected.geometry.length
```
