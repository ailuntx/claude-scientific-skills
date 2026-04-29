# 坐标参考系统 (CRS)

地理空间数据的坐标系、投影与转换完全指南

## 目录

1. [基础概念](#fundamentals)
2. [常用CRS代码](#common-crs-codes)
3. [投影坐标系与地理坐标系](#projected-vs-geographic)
4. [UTM分区](#utm-zones)
5. [坐标转换](#transformations)
6. [最佳实践](#best-practices)

## 基础概念

### 什么是CRS？

坐标参考系统定义坐标如何对应地球上的位置：

- **地理坐标系**：使用经纬度（度）
- **投影坐标系**：使用笛卡尔坐标（米、英尺）
- **垂直坐标系**：定义高度/深度（如椭球高）

### 核心组件

1. **基准面**：地球形状的数学模型
   - WGS 84 (EPSG:4326) - 全球GPS系统
   - NAD 83 (EPSG:4269) - 北美地区
   - ETRS89 (EPSG:4258) - 欧洲地区

2. **投影方式**：从曲面到平面的转换方法
   - 圆柱投影（墨卡托）
   - 圆锥投影（兰勃特等角）
   - 方位投影（极球面立体）

3. **单位**：度、米、英尺等

## 常用CRS代码

### 地理坐标系（经纬度）

| EPSG | 名称 | 区域 | 备注 |
|------|------|------|-------|
| 4326 | WGS 84 | 全球 | GPS默认坐标系，建议存储使用 |
| 4269 | NAD83 | 北美 | 美国地质调查局数据，与WGS84略有差异 |
| 4258 | ETRS89 | 欧洲 | 欧洲参考框架 |
| 4612 | GDA94 | 澳大利亚 | 澳大利亚基准面 |

### 投影坐标系（米制）

| EPSG | 名称 | 区域 | 形变率 | 备注 |
|------|------|------|------------|-------|
| 3857 | 网络墨卡托 | 全球（85°S-85°N） | 极区高度形变 | 网络地图（谷歌、OSM） |
| 32601-32660 | UTM北半球分区 | 全球（1°分带） | 每带<1% | 米制计算 |
| 32701-32760 | UTM南半球分区 | 全球（1°分带） | 每带<1% | 南半球 |
| 3395 | 墨卡托 | 世界 | 中等 | 世界地图 |
| 5070 | 美国本土阿尔伯斯 | 美国本土 | 低 | 美国国家制图 |
| 2154 | 兰勃特93 | 法国 | 极低 | 法国国家投影 |

### 区域投影系统

**美国：**
- EPSG:5070 - 美国国家图集等积投影（本土）
- EPSG:6350 - 美国国家图集（阿拉斯加）
- EPSG:102003 - 美国本土阿尔伯斯等积投影
- EPSG:2227 - 加利福尼亚3区（美制英尺）

**欧洲：**
- EPSG:3035 - 欧洲等积投影2001
- EPSG:3857 - 网络墨卡托（网络地图）
- EPSG:2154 - 兰勃特93（法国）
- EPSG:25832-25836 - UTM分区（ETRS89基准）

**其他地区：**
- EPSG:3112 - GDA94 / MGA 52区（澳大利亚）
- EPSG:2056 - CH1903+ / LV95（瑞士）
- EPSG:4326 - WGS 84（全球默认）

## 投影坐标系与地理坐标系

### 何时使用地理坐标系（EPSG:4326）

✅ 数据存储（数据库、文件）
✅ 全球数据集
✅ 网络API（GeoJSON, KML）
✅ 经纬度查询
✅ GPS坐标

```python
# 错误：在地理坐标系中计算距离
gpd.geographic_crs = "EPSG:4326"
distance = gdf.geometry.length  # 错误！返回单位为度而非米

# 正确：在投影坐标系中计算距离
gdf_projected = gdf.to_crs("EPSG:32633")  # UTM 33N分区
distance_m = gdf_projected.geometry.length  # 正确：单位为米
```

### 何时使用投影坐标系

✅ 面积/距离计算
✅ 缓冲区操作
✅ 空间分析
✅ 高精度制图
✅ 工程应用

```python
import geopandas as gpd

# 投影至合适的UTM分区
gdf = gpd.to_crs(gdf.estimate_utm_crs())

# 此时面积和距离计算精确
area_sqm = gdf.geometry.area
buffer_1km = gdf.geometry.buffer(1000)  # 1000米缓冲区
```

### 网络墨卡特警告

⚠️ **EPSG:3857（网络墨卡托）仅适用于可视化**

```python
# 切勿用网络墨卡托计算面积
gdf_web = gdf.to_crs("EPSG:3857")
area = gdf_web.geometry.area  # 错误！存在显著形变

# 应使用合适投影
gdf_utm = gdf.to_crs("EPSG:32633")  # 或estimate_utm_crs()
area = gdf_utm.geometry.area  # 正确
```

## UTM分区

### UTM分区原理

地球划分为60个分区（每6°经度一个分区）：
- 分区1-60：自西向东排列
- 每个分区分为北半球(326xx)和南半球(327xx)

### 确定UTM分区

```python
def get_utm_zone(longitude, latitude):
    """根据坐标获取UTM分区EPSG代码"""
    import math

    zone = math.floor((longitude + 180) / 6) + 1

    if latitude >= 0:
        epsg = 32600 + zone  # 北半球
    else:
        epsg = 32700 + zone  # 南半球

    return f"EPSG:{epsg}"

# 示例
get_utm_zone(-122.4, 37.7)  # 返回'EPSG:32610'（10N分区）
```

### 使用GeoPandas自动检测UTM分区

```python
import geopandas as gpd

# 加载数据
gdf = gpd.read_file('data.geojson')

# 估算最佳UTM分区
utm_crs = gdf.estimate_utm_crs()
print(f"最佳UTM坐标系: {utm_crs}")

# 重投影
gdf_projected = gdf.to_crs(utm_crs)
```

### 特殊UTM案例

**UPS（通用极球面投影）：**
- EPSG:5041 - 北极UPS投影
- EPSG:5042 - 南极UPS投影

**非标准UTM：**
- EPSG:31466-31469 - 德国高斯-克吕格分区
- EPSG:2056 - 瑞士LV95（基于UTM原理）

## 坐标转换

### 基础转换

```python
from pyproj import Transformer

# 创建转换器
transformer = Transformer.from_crs(
    "EPSG:4326",  # WGS 84 (经纬度)
    "EPSG:32633", # UTM 33N分区 (米制)
    always_xy=True  # 输入格式: x=经度, y=纬度 (非y=纬度, x=经度)
)

# 转换单点坐标
lon, lat = -122.4, 37.7
x, y = transformer.transform(lon, lat)
print(f"东向坐标: {x:.2f}, 北向坐标: {y:.2f}")
```

### 批量转换

```python
import numpy as np
from pyproj import Transformer

# 坐标数组
lon_array = [-122.4, -122.3]
lat_array = [37.7, 37.8]

transformer = Transformer.from_crs("EPSG:4326", "EPSG:32610", always_xy=True)
xs, ys = transformer.transform(lon_array, lat_array)
```

### 使用PyProj CRS对象转换

```python
from pyproj import CRS

# 获取CRS信息
crs = CRS.from_epsg(32633)

print(f"名称: {crs.name}")
print(f"类型: {crs.type_name}")
print(f"适用范围: {crs.area_of_use.name}")
print(f"基准面: {crs.datum.name}")
print(f"椭球体: {crs.ellipsoid_name}")
```

## 最佳实践

### 1. 始终明确CRS信息

```python
import geopandas as gpd

gdf = gpd.read_file('data.geojson')

# 立即检查CRS
print(f"坐标系: {gdf.crs}")  # 绝不应为None!

# 若未定义则设置
if gdf.crs is None:
    gdf.set_crs("EPSG:4326", inplace=True)
```

### 2. 空间操作前验证CRS

```python
def ensure_same_crs(gdf1, gdf2):
    """确保两个地理数据框使用相同CRS"""
    if gdf1.crs != gdf2.crs:
        gdf2 = gdf2.to_crs(gdf1.crs)
        print(f"已将gdf2重投影至{gdf1.crs}")
    return gdf1, gdf2

# 空间操作前使用
zones, points = ensure_same_crs(zones_gdf, points_gdf)
result = gpd.sjoin(points, zones, predicate='within')
```

### 3. 选用合适投影

```python
# 局部分析（<500公里范围）
gdf_local = gdf.to_crs(gdf.estimate_utm_crs())

# 国家/区域级分析
gdf_us = gdf.to_crs("EPSG:5070")  # 美国国家图集等积投影
gdf_eu = gdf.to_crs("EPSG:3035")  # 欧洲等积投影

# 网络可视化
gdf_web = gdf.to_crs("EPSG:3857")  # 网络墨卡托
```

### 4. 保留原始CRS

```python
# 备份原始坐标系
gdf_original = gdf.copy()
original_crs = gdf.crs

# 在投影坐标系中执行分析
gdf_projected = gdf.to_crs(gdf.estimate_utm_crs())
result = gdf_projected.geometry.buffer(1000)

# 必要时转换回原坐标系
result = result.to_crs(original_crs)
```

## 常见误区

### 错误1：使用度制单位计算面积

```python
# 错误：计算平方度面积
gdf = gpd.read_file('data.geojson')
area = gdf.geometry.area  # 错误！

# 正确：使用投影坐标系
gdf_proj = gdf.to_crs(gdf.estimate_utm_crs())
area_sqm = gdf_proj.geometry.area
area_sqkm = area_sqm / 1_000_000
```

### 错误2：在地理坐标系创建缓冲区

```python
# 错误：创建1000度缓冲区
gdf['buffer'] = gdf.geometry.buffer(1000)

# 正确：先投影转换
gdf_proj = gdf.to_crs("EPSG:32610")
gdf_proj['buffer_km'] = gdf_proj.geometry.buffer(1000)  # 1000米缓冲区
```

### 错误3：混合坐标系操作

```python
# 错误：未检查CRS的空间连接
result = gpd.sjoin(gdf1, gdf2, predicate='intersects')

# 正确：确保CRS一致
if gdf1.crs != gdf2.crs:
    gdf2 = gdf2.to_crs(gdf1.crs)
result = gpd.sjoin(gdf1, gdf2, predicate='intersects')
```

## 速查手册

```python
# 常用操作

# 检查CRS
gdf.crs
rasterio.open('file.tif').crs

# 重投影
gdf.to_crs("EPSG:32633")

# 自动检测UTM
gdf.estimate_utm_crs()

# 单点坐标转换
from pyproj import Transformer
tx = Transformer.from_crs("EPSG:4326", "EPSG:32610", always_xy=True)
x, y = tx.transform(lon, lat)

# 创建自定义CRS
from pyproj import CRS
custom_crs = CRS.from_proj4(
    "+proj=utm +zone=10 +ellps=WGS84 +datum=WGS84 +units=m +no_defs"
)
```

更多信息请访问：
- [EPSG注册库](https://epsg.org/)
- [PROJ文档](https://proj.org/)
- [pyproj文档](https://pyproj4.github.io/pyproj/)
