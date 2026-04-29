# GeoMaster 故障排除指南

常见地理空间问题解决方案与调试策略。

## 安装问题

### GDAL 安装问题

```bash
# 问题：提示"gdal-config not found"或rasterio安装失败

# 方案1：使用conda（推荐）
conda install -c conda-forge gdal rasterio

# 方案2：系统包安装（Ubuntu/Debian）
sudo apt-get install gdal-bin libgdal-dev
export CPLUS_INCLUDE_PATH=/usr/include/gdal
export C_INCLUDE_PATH=/usr/include/gdal
pip install rasterio

# 方案3：Wheel文件安装
pip install rasterio --find-links=https://gis.wheelwrights.com/

# 验证安装
python -c "from osgeo import gdal; print(gdal.__version__)"
python -c "import rasterio; print(rasterio.__version__)"
```

### Python 绑定问题

```bash
# 问题：Windows系统出现"DLL load failed"
# 解决方案：用conda重新安装
conda install -c conda-forge --force-reinstall gdal rasterio fiona

# 问题：macOS系统出现"Symbol not found"
# 解决方案：源码编译或使用conda
brew install gdal
pip install rasterio --no-binary rasterio

# 问题：GEOS错误
brew install geos
pip install shapely --no-binary shapely
```

## 运行时错误

### CRS 转换错误

```python
# 问题："Invalid projection"或"CRS mismatch"
import geopandas as gpd

# 检查CRS
print(f"CRS: {gdf.crs}")

# 若未设置则指定
if gdf.crs is None:
    gdf.set_crs("EPSG:4326", inplace=True)

# 若未知则尝试自动检测
gdf = gdf.to_crs(gdf.estimate_utm_crs())
```

### 大型栅格内存错误

```python
# 问题：读取大文件时出现"MemoryError"
# 解决方案：分块读取或使用窗口

import rasterio
from rasterio.windows import Window

# 窗口读取
with rasterio.open('large.tif') as src:
    window = Window(0, 0, 1000, 1000)  # (列偏移, 行偏移, 宽度, 高度)
    subset = src.read(1, window=window)

# 分块处理
with rasterio.open('large.tif') as src:
    for i, window in src.block_windows(1):
        block = src.read(1, window=window)
        # 处理区块...

# 超大文件使用Dask
import dask.array as da
dask_array = da.from_rasterio('large.tif', chunks=(1, 1024, 1024))
```

### 几何验证错误

```python
# 问题："TopologyException"或"Self-intersection"
import geopandas as gpd
from shapely.validation import make_valid

# 检查无效几何体
invalid = gdf[~gdf.is_valid]
print(f"无效几何体数量: {len(invalid)}")

# 修复无效几何体
gdf['geometry'] = gdf.geometry.make_valid()

# 缓冲修复法（替代方案）
gdf['geometry'] = gdf.geometry.buffer(0)
```

### 坐标顺序混淆

```python
# 问题：点位位置错误（经纬度颠倒）
from pyproj import Transformer

# 常见错误：lon/lat与lat/lon混淆
# 始终明确指定坐标轴顺序
transformer = Transformer.from_crs(
    "EPSG:4326",
    "EPSG:32610",
    always_xy=True  # 输入：x=经度, y=纬度（非y=纬度, x=经度）
)

# 正确用法
x, y = transformer.transform(lon, lat)  # 非lat, lon
```

## 性能问题

### 空间连接缓慢

```python
# 问题：sjoin耗时过长
import geopandas as gpd

# 解决方案：使用空间索引
gdf1.sindex  # 自动创建
gdf2.sindex

# 邻近分析使用专用函数
result = gpd.sjoin_nearest(gdf1, gdf2, max_distance=1000)

# 使用'within'谓词（点要素比'intersects'更快）
result = gpd.sjoin(points, polygons, predicate='within')

# 先裁剪到边界框
bbox = gdf1.total_bounds
gdf2_clipped = gdf2.cx[bbox[0]:bbox[2], bbox[1]:bbox[3]]
result = gpd.sjoin(gdf1, gdf2_clipped, predicate='intersects')
```

### 栅格读写缓慢

```python
# 问题：栅格读写速度慢
import rasterio

# 方案1：写入时启用压缩
profile.update(
    compress='DEFLATE',
    predictor=2,
    zlevel=4
)

# 方案2：分块输出
profile.update(
    tiled=True,
    blockxsize=256,
    blockysize=256
)

# 方案3：启用缓存
from osgeo import gdal
gdal.SetCacheMax(2**30)  # 1GB

# 方案4：使用COG格式云端访问
from rio_cogeo.cogeo import cog_translate
cog_translate('input.tif', 'output.tif', profile)
```

### 重投影缓慢

```python
# 问题：to_crs()执行极慢
import geopandas as gpd

# 方案1：先简化几何体
gdf_simple = gdf.geometry.simplify(tolerance=0.0001)
gdf_reproj = gdf_simple.to_crs(target_crs)

# 方案2：显示用低精度
gdf_reproj = gdf.to_crs(target_crs, geometry_precision=2)

# 方案3：并行重投影
import multiprocessing as mp
from functools import partial

def reproj_chunk(chunk, target_crs):
    return chunk.to_crs(target_crs)

chunks = np.array_split(gdf, mp.cpu_count())
with mp.Pool() as pool:
    results = pool.map(partial(reproj_chunk, target_crs=target_crs), chunks)
gdf_reproj = gpd.GeoDataFrame(pd.concat(results))
```

## 常见陷阱

### 以度为单位的面积计算

```python
# 错误：平方度单位面积
gdf = gpd.read_file('data.geojson')
area = gdf.geometry.area  # 错误！

# 正确：使用投影坐标系
gdf_proj = gdf.to_crs(gdf.estimate_utm_crs())
area_sqm = gdf_proj.geometry.area
area_sqkm = area_sqm / 1_000_000
```

### 地理坐标系缓冲

```python
# 错误：1000度缓冲
gdf['buffer'] = gdf.geometry.buffer(1000)

# 正确：先投影转换
gdf_proj = gdf.to_crs("EPSG:32610")
gdf_proj['buffer_km'] = gdf_proj.geometry.buffer(1000)  # 1000米缓冲
```

### Web墨卡托投影变形

```python
# 错误：Web墨卡托投影下计算面积
gdf = gdf.to_crs("EPSG:3857")
area = gdf.geometry.area  # 严重变形！

# 正确：使用合适投影
gdf = gdf.to_crs(gdf.estimate_utm_crs())
area = gdf.geometry.area  # 精确结果
```

### 混合坐标系

```python
# 错误：未检查CRS的空间连接
result = gpd.sjoin(gdf1, gdf2, predicate='intersects')

# 正确：确保坐标系一致
if gdf1.crs != gdf2.crs:
    gdf2 = gdf2.to_crs(gdf1.crs)
result = gpd.sjoin(gdf1, gdf2, predicate='intersects')
```

## 数据问题

### 坐标系缺失

```python
# 问题：CRS为None
gdf = gpd.read_file('data.geojson')
if gdf.crs is None:
    # 尝试通过数据范围推断
    lon_min, lat_min, lon_max, lat_max = gdf.total_bounds

    if -180 <= lon_min <= 180 and -90 <= lat_min <= 90:
        gdf.set_crs("EPSG:4326", inplace=True)
        print("已设为WGS 84 (EPSG:4326)")
    else:
        gdf.set_crs(gdf.estimate_utm_crs(), inplace=True)
        print("已估算UTM投影")
```

### 无效坐标

```python
# 问题：坐标超出有效范围
gdf = gpd.read_file('data.geojson')

# 检查无效坐标
invalid_lon = (gdf.geometry.x < -180) | (gdf.geometry.x > 180)
invalid_lat = (gdf.geometry.y < -90) | (gdf.geometry.y > 90)

if invalid_lon.any() or invalid_lat.any():
    print("警告：发现无效坐标")
    gdf = gdf[~invalid_lon & ~invalid_lat]
```

### 空几何体

```python
# 问题：空几何体导致处理失败
# 移除空几何体
gdf = gdf[~gdf.geometry.is_empty]

# 或用None填充
gdf.loc[gdf.geometry.is_empty, 'geometry'] = None

# 操作前检查
if gdf.geometry.is_empty.any():
    print(f"警告：存在{gdf.geometry.is_empty.sum()}个空几何体")
```

## 遥感问题

### Sentinel-2 波段顺序

```python
# 问题：波段索引错误
# Sentinel-2 L2A SAFE结构：
# B01 (60m), B02 (10m), B03 (10m), B04 (10m), B05 (20m),
# B06 (20m), B07 (20m), B08 (10m), B08A (20m), B09 (60m),
# B11 (20m), B12 (20m)

# 重采样至10米的Sentinel-2：
# B02=1, B03=2, B04=3, B05=4, B06=5, B07=6, B08=7, B8A=8, B11=9, B12=10

# 仅10米波段：
blue = src.read(1)   # B02
green = src.read(2)  # B03
red = src.read(3)    # B04
nir = src.read(4)    # B08
```

### 云影掩膜

```python
# 问题：云层和阴影掩膜不完整
def improved_cloud_mask(scl):
    """
    使用SCL图层增强云掩膜。
    类别：0=无数据, 1=饱和, 2=暗区, 3=云影,
    4=植被, 5=裸土, 6=水体, 7=低概率云,
    8=中概率云, 9=高概率云, 10=薄卷云
    """
    # 掩膜：云层、云影、饱和区
    mask = scl.isin([0, 1, 3, 8, 9, 10])
    return mask

# 应用
scl = s2_image.select('SCL')
cloud_mask = improved_cloud_mask(scl)
image_clean = s2_image.updateMask(cloud_mask.Not())
```

## 错误信息参考表

| 错误 | 原因 | 解决方案 |
|-------|-------|----------|
| `CRS mismatch` | 坐标系不一致 | `gdf2 = gdf2.to_crs(gdf1.crs)` |
| `TopologyException` | 无效/自相交几何体 | `gdf.geometry = gdf.geometry.make_valid()` |
| `MemoryError` | 数据集过大 | 使用Dask或分块读取 |
| `Invalid projection` | 未知CRS代码 | 检查EPSG代码，使用`CRS.from_user_input()` |
| `Empty geometry` | 空几何体 | `gdf = gdf[~gdf.geometry.is_empty]` |
| `Bounds error` | 坐标越界 | 过滤无效坐标 |
| `DLL load failed` | GDAL未安装 | 使用conda：`conda install gdal` |
| `Symbol not found` | 库链接问题 | 用二进制wheel或conda重装 |
| `Self-intersection` | 无效多边形 | Buffer(0)或make_valid() |

## 调试策略

### 1. 数据完整性检查

```python
def check_geodataframe(gdf):
    """GeoDataFrame全面健康检查"""
    print(f"数据维度: {gdf.shape}")
    print(f"坐标系: {gdf.crs}")
    print(f"边界范围: {gdf.total_bounds}")
    print(f"无效几何体: {(~gdf.is_valid).sum()}")
    print(f"空几何体: {gdf.geometry.is_empty.sum()}")
    print(f"空值几何体: {gdf.geometry.isna().sum()}")
    print(f"重复几何体: {gdf.geometry.duplicated().sum()}")
    print("\n几何类型分布:")
    print(gdf.geometry.type.value_counts())
    print("\n坐标范围:")
    print(f"  X轴: {gdf.geometry.x.min():.2f} 至 {gdf.geometry.x.max():.2f}")
    print(f"  Y轴: {gdf.geometry.y.min():.2f} 至 {gdf.geometry.y.max():.2f}")

check_geodataframe(gdf)
```

### 2. 转换测试

```python
def test_reprojection(gdf, target_crs):
    """测试重投影是否可逆"""
    original = gdf.copy()
    gdf_proj = gdf.to_crs(target_crs)
    gdf_back = gdf_proj.to_crs(gdf.crs)

    diff = original.geometry.distance(gdf_back.geometry).max()
    if diff > 1:  # 误差超过1米
        print(f"警告：最大误差: {diff:.2f}米")
        return False
    return True
```

### 3. 代码性能分析

```python
import time

def time_operation(func, *args, **kwargs):
    """计时地理空间操作"""
    start = time.time()
    result = func(*args, **kwargs)
    elapsed = time.time() - start
    print(f"{func.__name__}: 耗时{elapsed:.2f}秒")
    return result

# 使用示例
time_operation(gdf.to_crs, "EPSG:32610")
```

## 获取帮助

### 检查版本

```python
import sys
import geopandas as gpd
import rasterio
from osgeo import gdal

print(f"Python版本: {sys.version}")
print(f"GeoPandas版本: {gpd.__version__}")
print(f"Rasterio版本: {rasterio.__version__}")
print(f"GDAL版本: {gdal.__version__}")
print(f"PROJ版本: {gdal.VersionInfo('PROJ')}")
```

### 实用资源

- **GeoPandas文档**: https://geopandas.org/
- **Rasterio文档**: https://rasterio.readthedocs.io/
- **PROJ数据库**: https://epsg.org/
- **Stack Overflow**: 使用`gis`和`python`标签
- **GIS Stack Exchange**: https://gis.stackexchange.com/
