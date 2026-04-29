# 坐标系参考系统（CRS）

坐标系参考系统定义了坐标如何关联到地球上的位置。

## 理解CRS

CRS信息以`pyproj.CRS`对象存储：

```python
# 检查CRS
print(gdf.crs)

# 检查是否设置了CRS
if gdf.crs is None:
    print("未定义CRS")
```

## 设置坐标系与重投影

### 设置坐标系

当坐标正确但缺少CRS元数据时使用`set_crs()`：

```python
# 设置CRS（不转换坐标）
gdf = gdf.set_crs("EPSG:4326")
gdf = gdf.set_crs(4326)
```

**警告**：仅在缺少CRS元数据时使用。此操作不会转换坐标。

### 重投影

使用`to_crs()`在不同坐标系间转换坐标：

```python
# 重投影到不同CRS
gdf_projected = gdf.to_crs("EPSG:3857")  # 网络墨卡托
gdf_projected = gdf.to_crs(3857)

# 重投影以匹配另一个GeoDataFrame
gdf1_reprojected = gdf1.to_crs(gdf2.crs)
```

## CRS格式

GeoPandas通过`pyproj.CRS.from_user_input()`支持多种格式：

```python
# EPSG代码（整数）
gdf.to_crs(4326)

# 授权字符串
gdf.to_crs("EPSG:4326")
gdf.to_crs("ESRI:102003")

# WKT字符串（Well-Known Text）
gdf.to_crs("GEOGCS[...]")

# PROJ字符串
gdf.to_crs("+proj=longlat +datum=WGS84")

# pyproj.CRS对象
from pyproj import CRS
crs_obj = CRS.from_epsg(4326)
gdf.to_crs(crs_obj)
```

**最佳实践**：使用WKT2或授权字符串（EPSG）以保留完整CRS信息。

## 常用EPSG代码

### 地理坐标系

```python
# WGS 84（纬度/经度）
gdf.to_crs("EPSG:4326")

# NAD83
gdf.to_crs("EPSG:4269")
```

### 投影坐标系

```python
# 网络墨卡托（网络地图使用）
gdf.to_crs("EPSG:3857")

# UTM分区（示例：UTM 33N区）
gdf.to_crs("EPSG:32633")

# UTM分区（南半球，示例：UTM 33S区）
gdf.to_crs("EPSG:32733")

# 美国国家地图等积投影
gdf.to_crs("ESRI:102003")

# 阿尔伯斯等积圆锥投影（北美）
gdf.to_crs("EPSG:5070")
```

## 空间操作对CRS的要求

### 需要匹配CRS的操作

以下操作要求CRS必须一致：

```python
# 空间连接
gpd.sjoin(gdf1, gdf2, ...)  # CRS必须匹配

# 叠加操作
gpd.overlay(gdf1, gdf2, ...)  # CRS必须匹配

# 数据拼接
pd.concat([gdf1, gdf2])  # CRS必须匹配

# 需要时先重投影
gdf2_reprojected = gdf2.to_crs(gdf1.crs)
result = gpd.sjoin(gdf1, gdf2_reprojected)
```

### 建议在投影坐标系中执行的操作

面积和距离计算应使用投影坐标系：

```python
# 错误：以度为单位计算面积（无意义）
areas_degrees = gdf.geometry.area  # 若CRS为EPSG:4326

# 正确：先重投影到合适的投影坐标系
gdf_projected = gdf.to_crs("EPSG:3857")
areas_meters = gdf_projected.geometry.area  # 平方米

# 更佳：使用本地UTM分区提高精度
gdf_utm = gdf.to_crs("EPSG:32633")  # UTM 33N区
accurate_areas = gdf_utm.geometry.area
```

## 选择合适的CRS

### 面积/距离计算

使用等积投影：

```python
# 阿尔伯斯等积圆锥投影（北美）
gdf.to_crs("EPSG:5070")

# 兰伯特方位等积投影
gdf.to_crs("EPSG:3035")  # 欧洲

# UTM分区（适用于局部区域）
gdf.to_crs("EPSG:32633")  # 合适的UTM分区
```

### 保持距离（导航用途）

使用等距投影：

```python
# 方位等距投影
gdf.to_crs("ESRI:54032")
```

### 保持形状（角度）

使用等角投影：

```python
# 网络墨卡托（等角但会扭曲面积）
gdf.to_crs("EPSG:3857")

# UTM分区（局部区域等角）
gdf.to_crs("EPSG:32633")
```

### 网络地图应用

```python
# 网络墨卡托（网络地图标准）
gdf.to_crs("EPSG:3857")
```

## 估算UTM分区

```python
# 根据数据估算合适的UTM CRS
utm_crs = gdf.estimate_utm_crs()
gdf_utm = gdf.to_crs(utm_crs)
```

## 多几何列不同CRS支持

GeoPandas 0.8+支持不同几何列使用不同CRS：

```python
# 为特定几何列设置CRS
gdf = gdf.set_crs("EPSG:4326", allow_override=True)

# 活动几何列决定操作
gdf = gdf.set_geometry('other_geom_column')

# 检查CRS不匹配
try:
    result = gdf1.overlay(gdf2)
except ValueError as e:
    print("CRS不匹配:", e)
```

## CRS信息获取

```python
# 获取完整CRS详情
print(gdf.crs)

# 获取EPSG代码（若存在）
print(gdf.crs.to_epsg())

# 获取WKT表示
print(gdf.crs.to_wkt())

# 获取PROJ字符串
print(gdf.crs.to_proj4())

# 检查是否为地理CRS（经纬度）
print(gdf.crs.is_geographic)

# 检查是否为投影CRS
print(gdf.crs.is_projected)
```

## 转换单个几何体

```python
from pyproj import Transformer

# 创建转换器
transformer = Transformer.from_crs("EPSG:4326", "EPSG:3857", always_xy=True)

# 转换点坐标
x_new, y_new = transformer.transform(x, y)
```
