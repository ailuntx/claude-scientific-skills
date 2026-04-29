# 空间数据读写

## 读取文件

使用 `geopandas.read_file()` 导入矢量空间数据：

```python
import geopandas as gpd

# 从文件读取
gdf = gpd.read_file("data.shp")
gdf = gpd.read_file("data.geojson")
gdf = gpd.read_file("data.gpkg")

# 从URL读取
gdf = gpd.read_file("https://example.com/data.geojson")

# 从ZIP压缩包读取
gdf = gpd.read_file("data.zip")
```

### 性能：Arrow加速

使用Arrow实现2-4倍读取加速：

```python
gdf = gpd.read_file("data.gpkg", use_arrow=True)
```

需要安装PyArrow：`uv pip install pyarrow`

### 读取时过滤

预过滤数据仅加载所需内容：

```python
# 加载特定行
gdf = gpd.read_file("data.gpkg", rows=100)  # 前100行
gdf = gpd.read_file("data.gpkg", rows=slice(10, 20))  # 第10-20行

# 加载特定列
gdf = gpd.read_file("data.gpkg", columns=['name', 'population'])

# 通过边界框空间过滤
gdf = gpd.read_file("data.gpkg", bbox=(xmin, ymin, xmax, ymax))

# 通过几何掩膜空间过滤
gdf = gpd.read_file("data.gpkg", mask=polygon_geometry)

# SQL WHERE子句（需Fiona 1.9+或Pyogrio）
gdf = gpd.read_file("data.gpkg", where="population > 1000000")

# 跳过几何（返回pandas DataFrame）
df = gpd.read_file("data.gpkg", ignore_geometry=True)
```

## 写入文件

使用 `to_file()` 导出数据：

```python
# 写入Shapefile
gdf.to_file("output.shp")

# 写入GeoJSON
gdf.to_file("output.geojson", driver='GeoJSON')

# 写入GeoPackage（支持多图层）
gdf.to_file("output.gpkg", layer='layer1', driver="GPKG")

# Arrow加速写入
gdf.to_file("output.gpkg", use_arrow=True)
```

### 支持格式

列出所有可用驱动：

```python
import pyogrio
pyogrio.list_drivers()
```

常用格式：Shapefile、GeoJSON、GeoPackage (GPKG)、KML、MapInfo File、CSV（含WKT几何）

## Parquet与Feather

保留空间信息的列式格式，支持多几何列：

```python
# 写入
gdf.to_parquet("data.parquet")
gdf.to_feather("data.feather")

# 读取
gdf = gpd.read_parquet("data.parquet")
gdf = gpd.read_feather("data.feather")
```

优势：
- 比传统格式更快的I/O
- 更好的压缩率
- 保留多几何列
- 支持模式版本控制

## PostGIS数据库

### 从PostGIS读取

```python
from sqlalchemy import create_engine

engine = create_engine('postgresql://user:password@host:port/database')

# 读取整表
gdf = gpd.read_postgis("SELECT * FROM table_name", con=engine, geom_col='geometry')

# 通过SQL查询读取
gdf = gpd.read_postgis("SELECT * FROM table WHERE population > 100000", con=engine, geom_col='geometry')
```

### 写入PostGIS

```python
# 创建或替换表
gdf.to_postgis("table_name", con=engine, if_exists='replace')

# 追加到现有表
gdf.to_postgis("table_name", con=engine, if_exists='append')

# 表存在时报错
gdf.to_postgis("table_name", con=engine, if_exists='fail')
```

需要安装：`uv pip install psycopg2` 或 `uv pip install psycopg` 以及 `uv pip install geoalchemy2`

## 类文件对象

从文件句柄或内存缓冲区读取：

```python
# 从文件句柄读取
with open('data.geojson', 'r') as f:
    gdf = gpd.read_file(f)

# 从StringIO读取
from io import StringIO
geojson_string = '{"type": "FeatureCollection", ...}'
gdf = gpd.read_file(StringIO(geojson_string))
```

## 远程存储（fsspec）

从云存储访问数据：

```python
# S3
gdf = gpd.read_file("s3://bucket/data.gpkg")

# Azure Blob存储
gdf = gpd.read_file("az://container/data.gpkg")

# HTTP/HTTPS
gdf = gpd.read_file("https://example.com/data.geojson")
```
