# 大数据与云计算

面向地理空间数据的分布式处理、云平台与GPU加速技术。

## 使用Dask进行分布式处理

### Dask-GeoPandas

```python
import dask_geopandas
import geopandas as gpd
import dask.dataframe as dd

# 分块读取大型GeoPackage文件
dask_gdf = dask_geopandas.read_file('large.gpkg', npartitions=10)

# 执行空间运算
dask_gdf['area'] = dask_gdf.geometry.area
dask_gdf['buffer'] = dask_gdf.geometry.buffer(1000)

# 计算结果
result = dask_gdf.compute()

# 分布式空间连接
dask_points = dask_geopandas.read_file('points.gpkg', npartitions=5)
dask_zones = dask_geopandas.read_file('zones.gpkg', npartitions=3)

joined = dask_points.sjoin(dask_zones, how='inner', predicate='within')
result = joined.compute()
```

### 用于栅格处理的Dask

```python
import dask.array as da
import rasterio

# 创建延迟加载的栅格数组
def lazy_raster(path, chunks=(1, 1024, 1024)):
    with rasterio.open(path) as src:
        profile = src.profile
        # 创建dask数组
        raster = da.from_rasterio(src, chunks=chunks)

    return raster, profile

# 处理大型栅格
raster, profile = lazy_raster('very_large.tif')

# 计算NDVI（延迟操作）
ndvi = (raster[3] - raster[2]) / (raster[3] + raster[2] + 1e-8)

# 对每个数据块应用函数
def process_chunk(chunk):
    return (chunk - chunk.min()) / (chunk.max() - chunk.min())

normalized = da.map_blocks(process_chunk, ndvi, dtype=np.float32)

# 计算并保存结果
with rasterio.open('output.tif', 'w', **profile) as dst:
    dst.write(normalized.compute())
```

### Dask分布式集群

```python
from dask.distributed import Client

# 连接集群
client = Client('scheduler-address:8786')

# 或创建本地集群
from dask.distributed import LocalCluster
cluster = LocalCluster(n_workers=4, threads_per_worker=2, memory_limit='4GB')
client = Client(cluster)

# 在集群中使用Dask-GeoPandas
dask_gdf = dask_geopandas.from_geopandas(gdf, npartitions=10)
dask_gdf = dask_gdf.set_index(calculate_spatial_partitions=True)

# 操作实现分布式执行
result = dask_gdf.buffer(1000).compute()
```

## 云平台

### Google Earth Engine

```python
import ee

# 初始化
ee.Initialize(project='your-project')

# 大规模影像合成
def create_annual_composite(year):
    """创建无云年度合成影像"""

    # Sentinel-2数据集
    s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED') \
        .filterBounds(ee.Geometry.Rectangle([-125, 32, -114, 42])) \
        .filterDate(f'{year}-01-01', f'{year}-12-31') \
        .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))

    # 云掩膜处理
    def mask_s2(image):
        qa = image.select('QA60')
        cloud_bit_mask = 1 << 10
        cirrus_bit_mask = 1 << 11
        mask = qa.bitwiseAnd(cloud_bit_mask).eq(0).And(
               qa.bitwiseAnd(cirrus_bit_mask).eq(0))
        return image.updateMask(mask.Not())

    s2_masked = s2.map(mask_s2)

    # 中值合成
    composite = s2_masked.median().clip(roi)

    return composite

# 导出到Google云端硬盘
task = ee.batch.Export.image.toDrive(
    image=composite,
    description='CA_composite_2023',
    scale=10,
    region=roi,
    crs='EPSG:32611',
    maxPixels=1e13
)
task.start()
```

### Planetary Computer（微软）

```python
import pystac_client
import planetary_computer
import odc.stac
import xarray as xr

# 搜索目录
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)

# 搜索NAIP影像
search = catalog.search(
    collections=["naip"],
    bbox=[-125, 32, -114, 42],
    datetime="2020-01-01/2023-12-31",
)

items = list(search.get_items())

# 加载为xarray数据集
data = odc.stac.load(
    items[:100],  # 分批处理
    bands=["image"],
    crs="EPSG:32611",
    resolution=1.0,
    chunkx=1024,
    chunky=1024,
)

# 延迟计算统计量
mean = data.mean().compute()
std = data.std().compute()

# 导出为COG格式
import rioxarray
data.isel(time=0).rio.to_raster('naip_composite.tif', compress='DEFLATE')
```

### Google云存储

```python
from google.cloud import storage
import rasterio
from rasterio.session import GSSession

# 上传至GCS
client = storage.Client()
bucket = client.bucket('my-bucket')
blob = bucket.blob('geospatial/data.tif')
blob.upload_from_filename('local_data.tif')

# 直接从GCS读取
with rasterio.open(
    'gs://my-bucket/geospatial/data.tif',
    session=GSSession()
) as src:
    data = src.read()

# 与Rioxarray配合使用
import rioxarray
da = rioxarray.open_rasterio('gs://my-bucket/geospatial/data.tif')
```

## GPU加速

### 使用CuPy处理栅格数据

```python
import cupy as cp
import numpy as np

def gpu_ndvi(nir, red):
    """在GPU上计算NDVI"""
    # 传输至GPU
    nir_gpu = cp.asarray(nir)
    red_gpu = cp.asarray(red)

    # GPU端计算
    ndvi_gpu = (nir_gpu - red_gpu) / (nir_gpu + red_gpu + 1e-8)

    # 传回CPU
    return cp.asnumpy(ndvi_gpu)

# 批量处理
def batch_process_gpu(raster_path):
    with rasterio.open(raster_path) as src:
        data = src.read()  # (波段, 高度, 宽度)

    data_gpu = cp.asarray(data)

    # 处理所有波段
    for i in range(data.shape[0]):
        data_gpu[i] = (data_gpu[i] - data_gpu[i].min()) / \
                      (data_gpu[i].max() - data_gpu[i].min())

    return cp.asnumpy(data_gpu)
```

### 使用RAPIDS进行空间分析

```python
import cudf
import cuspatial

# 将数据加载至GPU
gdf_gpu = cuspatial.from_geopandas(gdf)

# GPU端空间连接
points_gpu = cuspatial.from_geopandas(points_gdf)
polygons_gpu = cuspatial.from_geopandas(polygons_gdf)

joined = cuspatial.join_polygon_points(
    polygons_gpu,
    points_gpu
)

# 转换回CPU
result = joined.to_pandas()
```

### 使用PyTorch进行地理空间深度学习

```python
import torch
from torch.utils.data import DataLoader

# 自定义数据集
class SatelliteDataset(torch.utils.data.Dataset):
    def __init__(self, image_paths, label_paths):
        self.image_paths = image_paths
        self.label_paths = label_paths

    def __getitem__(self, idx):
        with rasterio.open(self.image_paths[idx]) as src:
            image = src.read().astype(np.float32)

        with rasterio.open(self.label_paths[idx]) as src:
            label = src.read(1).astype(np.int64)

        return torch.from_numpy(image), torch.from_numpy(label)

# 支持GPU预取的数据加载器
dataset = SatelliteDataset(images, labels)
loader = DataLoader(
    dataset,
    batch_size=16,
    shuffle=True,
    num_workers=4,
    pin_memory=True,  # 加速向GPU传输
)

# 混合精度训练
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()

for images, labels in loader:
    images, labels = images.to('cuda'), labels.to('cuda')

    with autocast():
        outputs = model(images)
        loss = criterion(outputs, labels)

    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

## 高效数据格式

### 云优化GeoTIFF（COG）

```python
from rio_cogeo.cogeo import cog_translate

# 转换为COG格式
cog_translate(
    src_path='input.tif',
    dst_path='output_cog.tif',
    dst_kwds={'compress': 'DEFLATE', 'predictor': 2},
    overview_level=5,
    overview_resampling='average',
    config={'GDAL_TIFF_INTERNAL_MASK': True}
)

# 创建概览图加速访问
with rasterio.open('output.tif', 'r+') as src:
    src.build_overviews([2, 4, 8, 16], resampling='average')
    src.update_tags(ns='rio_overview', resampling='average')
```

### 多维数组存储格式Zarr

```python
import xarray as xr
import zarr

# 创建Zarr存储
store = zarr.DirectoryStore('data.zarr')

# 将数据立方体保存至Zarr
ds.to_zarr(store, consolidated=True)

# 高效读取
ds = xr.open_zarr('data.zarr', consolidated=True)

# 高效提取子集
subset = ds.sel(time='2023-01', latitude=slice(30, 40))
```

### 矢量数据存储格式Parquet

```python
import geopandas as gpd

# 写入Parquet（含空间索引）
gdf.to_parquet('data.parquet', compression='snappy', index=True)

# 高效读取
gdf = gpd.read_parquet('data.parquet')

# 带过滤条件读取子集
import pyarrow.parquet as pq
table = pq.read_table('data.parquet', filters=[('column', '==', 'value')])
```

更多大数据示例请参见[code-examples.md](code-examples.md)。
