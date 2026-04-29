# 几何操作

GeoPandas 通过集成 Shapely 提供了丰富的几何操作功能。

## 构造性操作

基于现有几何创建新几何体：

### 缓冲区

创建表示指定距离内所有点的几何体：

```python
# 固定距离创建缓冲区
buffered = gdf.geometry.buffer(10)

# 负缓冲区（侵蚀）
eroded = gdf.geometry.buffer(-5)

# 带分辨率参数的缓冲区
smooth_buffer = gdf.geometry.buffer(10, resolution=16)
```

### 边界

获取低维度边界：

```python
# 多边形 -> 线串，线串 -> 多点集
boundaries = gdf.geometry.boundary
```

### 质心

获取每个几何体的中心点：

```python
centroids = gdf.geometry.centroid
```

### 凸包

包含所有点的最小凸多边形：

```python
hulls = gdf.geometry.convex_hull
```

### 凹包

包含所有点的最小凹多边形：

```python
# ratio 参数控制凹度（0=凸包，1=最凹）
concave_hulls = gdf.geometry.concave_hull(ratio=0.5)
```

### 包络矩形

最小轴对齐矩形：

```python
envelopes = gdf.geometry.envelope
```

### 简化

降低几何复杂度：

```python
# 带容差的 Douglas-Peucker 算法
simplified = gdf.geometry.simplify(tolerance=10)

# 保持拓扑结构（防止自相交）
simplified = gdf.geometry.simplify(tolerance=10, preserve_topology=True)
```

### 分段化

为线段添加顶点：

```python
# 按最大段长添加顶点
segmented = gdf.geometry.segmentize(max_segment_length=5)
```

### 整体并集

将所有几何体合并为单一几何体：

```python
# 合并所有要素
unified = gdf.geometry.union_all()
```

## 仿射变换

坐标的数学变换：

### 旋转

```python
# 绕原点 (0, 0) 旋转指定角度（度）
rotated = gdf.geometry.rotate(angle=45, origin='center')

# 绕自定义点旋转
rotated = gdf.geometry.rotate(angle=45, origin=(100, 100))
```

### 缩放

```python
# 均匀缩放
scaled = gdf.geometry.scale(xfact=2.0, yfact=2.0)

# 指定原点缩放
scaled = gdf.geometry.scale(xfact=2.0, yfact=2.0, origin='center')
```

### 平移

```python
# 坐标偏移
translated = gdf.geometry.translate(xoff=100, yoff=50)
```

### 倾斜

```python
# 剪切变换
skewed = gdf.geometry.skew(xs=15, ys=0, origin='center')
```

### 自定义仿射变换

```python
from shapely import affinity

# 应用六参数仿射变换矩阵
# [a, b, d, e, xoff, yoff]
transformed = gdf.geometry.affine_transform([1, 0, 0, 1, 100, 50])
```

## 几何属性

访问几何属性（返回 pandas Series）：

```python
# 面积
areas = gdf.geometry.area

# 长度/周长
lengths = gdf.geometry.length

# 边界框坐标
bounds = gdf.geometry.bounds  # 返回含 minx, miny, maxx, maxy 的 DataFrame

# 整个 GeoSeries 的总边界
total_bounds = gdf.geometry.total_bounds  # 返回数组 [minx, miny, maxx, maxy]

# 检查几何类型
geom_types = gdf.geometry.geom_type

# 检查有效性
is_valid = gdf.geometry.is_valid

# 检查是否为空
is_empty = gdf.geometry.is_empty
```

## 几何关系

二元谓词关系测试：

```python
# 内部关系
gdf1.geometry.within(gdf2.geometry)

# 包含关系
gdf1.geometry.contains(gdf2.geometry)

# 相交关系
gdf1.geometry.intersects(gdf2.geometry)

# 接触关系
gdf1.geometry.touches(gdf2.geometry)

# 穿越关系
gdf1.geometry.crosses(gdf2.geometry)

# 重叠关系
gdf1.geometry.overlaps(gdf2.geometry)

# 覆盖关系
gdf1.geometry.covers(gdf2.geometry)

# 被覆盖关系
gdf1.geometry.covered_by(gdf2.geometry)
```

## 点提取

从几何体中提取特定点：

```python
# 代表性点（保证在几何体内）
rep_points = gdf.geometry.representative_point()

# 沿线段按距离插值取点
points = line_gdf.geometry.interpolate(distance=10)

# 按归一化距离插值取点（0 到 1）
midpoints = line_gdf.geometry.interpolate(distance=0.5, normalized=True)
```

## 德劳内三角剖分

```python
# 创建三角网
triangles = gdf.geometry.delaunay_triangles()
```
