# 科学领域应用

跨学科地理空间应用：海洋、大气、水文等领域。

## 海洋与海岸地理信息系统

### 海岸脆弱性评估

```python
import geopandas as gpd
import rasterio
import numpy as np

def coastal_vulnerability_index(dem_path, shoreline_path, output_path):
    """计算海岸脆弱性指数。"""

    # 1. 加载高程数据
    with rasterio.open(dem_path) as src:
        dem = src.read(1)
        transform = src.transform

    # 2. 计算至海岸线距离
    shoreline = gpd.read_file(shoreline_path)
    # ... 计算距离栅格 ...

    # 3. 脆弱性指标（0-1范围）
    elevation_vuln = 1 - np.clip(dem / 10, 0, 1)  # 高程越低越脆弱
    slope_vuln = 1 - np.clip(slope / 10, 0, 1)

    # 4. 加权叠加分析
    weights = {
        'elevation': 0.3,
        'slope': 0.2,
        'distance_to_shore': 0.2,
        'wave_height': 0.2,
        'sea_level_trend': 0.1
    }

    cvi = sum(vuln * w for vuln, w in zip(
        [elevation_vuln, slope_vuln, distance_vuln, wave_vuln, slr_vuln],
        weights.values()
    ))

    return cvi
```

### 海洋生境制图

```python
# 底栖生境分类
def classify_benthic_habitat(bathymetry, backscatter, derived_layers):
    """
    使用以下要素进行底栖生境分类：
    - 水深数据
    - 后向散射强度
    - 衍生地形特征
    """

    # 1. 提取特征
    features = {
        'depth': bathymetry,
        'backscatter': backscatter,
        'slope': calculate_slope(bathymetry),
        'rugosity': calculate_rugosity(bathymetry),
        'curvature': calculate_curvature(bathymetry)
    }

    # 2. 分类规则
    habitat_classes = {}

    # 珊瑚礁：浅水区、高粗糙度、中后向散射
    coral_mask = (
        (features['depth'] > -30) &
        (features['depth'] < -5) &
        (features['rugosity'] > 2) &
        (features['backscatter'] > -15)
    )
    habitat_classes[coral_mask] = 1  # 珊瑚

    # 海草：极浅水区、低后向散射
    seagrass_mask = (
        (features['depth'] > -15) &
        (features['depth'] < -2) &
        (features['backscatter'] < -20)
    )
    habitat_classes[seagrass_mask] = 2  # 海草

    # 沙质海底：低粗糙度
    sand_mask = (
        (features['rugosity'] < 1.5) &
        (features['slope'] < 5)
    )
    habitat_classes[sand_mask] = 3  # 沙地

    return habitat_classes
```

## 大气科学

### 气象数据处理

```python
import xarray as xr
import rioxarray

# 打开NetCDF气象数据
ds = xr.open_dataset('era5_data.nc')

# 选择变量和时间
temperature = ds.t2m  # 2米气温
precipitation = ds.tp  # 总降水量

# 空间子集提取
roi = ds.sel(latitude=slice(20, 30), longitude=slice(65, 75))

# 时间聚合
monthly = roi.resample(time='1M').mean()
daily = roi.resample(time='1D').sum()

# 导出为GeoTIFF
temperature.rio.to_raster('temperature.tif')

# 计算气候指数
def calculate_spi(precip, scale=3):
    """标准化降水指数。"""
    # 拟合伽马分布
    from scipy import stats
    # ... SPI计算过程 ...
    return spi
```

### 空气质量分析

```python
# PM2.5空间插值
def interpolate_pm25(sensor_gdf, grid_resolution=1000):
    """
    基于传感器网络进行PM2.5插值。
    使用反距离加权或克里金法。
    """
    from pykrige.ok import OrdinaryKriging
    import numpy as np

    # 提取坐标和数值
    lon = sensor_gdf.geometry.x.values
    lat = sensor_gdf.geometry.y.values
    values = sensor_gdf['PM25'].values

    # 创建网格
    grid_lon = np.arange(lon.min(), lon.max(), grid_resolution)
    grid_lat = np.arange(lat.min(), lat.max(), grid_resolution)

    # 普通克里金法
    OK = OrdinaryKriging(lon, lat, values,
                        variogram_model='exponential',
                        verbose=False,
                        enable_plotting=False)

    # 插值计算
    z, ss = OK.execute('grid', grid_lon, grid_lat)

    return z, grid_lon, grid_lat
```

## 水文学

### 流域划分

```python
import rasterio
import numpy as np
from scipy import ndimage

def delineate_watershed(dem_path, outlet_point):
    """
    基于数字高程模型和出水口划分流域。
    """

    # 1. 加载DEM数据
    with rasterio.open(dem_path) as src:
        dem = src.read(1)
        transform = src.transform

    # 2. 填充洼地
    filled = fill_sinks(dem)

    # 3. 计算流向（D8算法）
    flow_dir = calculate_flow_direction_d8(filled)

    # 4. 计算汇流量
    flow_acc = calculate_flow_accumulation(flow_dir)

    # 5. 划分流域
    # 将出水点转换为栅格坐标
    row, col = ~transform * (outlet_point.x, outlet_point.y)
    row, col = int(row), int(col)

    # 向上游追踪
    watershed = trace_upstream(flow_dir, row, col)

    return watershed, flow_acc, flow_dir

def calculate_flow_direction_d8(dem):
    """D8流向算法。"""
    # 用2的幂次编码方向
    # 32 64 128
    # 16  0   1
    # 8   4   2

    rows, cols = dem.shape
    flow_dir = np.zeros_like(dem, dtype=np.uint8)

    directions = [
        (-1, 0, 64), (-1, 1, 128), (0, 1, 1), (1, 1, 2),
        (1, 0, 4), (1, -1, 8), (0, -1, 16), (-1, -1, 32)
    ]

    for i in range(1, rows - 1):
        for j in range(1, cols - 1):
            max_drop = -np.inf
            steepest_dir = 0

            for di, dj, code in directions:
                ni, nj = i + di, j + dj
                drop = dem[i, j] - dem[ni, nj]

                if drop > max_drop and drop > 0:
                    max_drop = drop
                    steepest_dir = code

            flow_dir[i, j] = steepest_dir

    return flow_dir
```

### 洪水淹没模拟

```python
def flood_inundation(dem, flood_level, roughness=0.03):
    """
    简易洪水淹没模拟。
    """

    # 1. 识别淹没单元
    flooded_mask = dem < flood_level

    # 2. 计算洪水深度
    flood_depth = np.where(flood_mask, flood_level - dem, 0)

    # 3. 移除孤立像元（连通域分析）
    labeled, num_features = ndimage.label(flooded_mask)

    # 仅保留大型连通域（湖泊级，非像元级）
    component_sizes = np.bincount(labeled.ravel())
    large_components = component_sizes > 100  # 阈值

    mask_indices = large_components[labeled]
    final_flooded = flooded_mask & mask_indices

    # 4. 计算淹没范围面积
    cell_area = 30 * 30  # 假设30米分辨率
    flooded_area = np.sum(final_flooded) * cell_area

    return flood_depth, final_flooded, flooded_area
```

## 农业领域

### 作物长势监测

```python
def crop_condition_indices(ndvi_time_series):
    """
    利用NDVI时间序列监测作物长势。
    """

    # 1. 计算生长季指标
    max_ndvi = np.max(ndvi_time_series)
    time_to_peak = np.argmax(ndvi_time_series)

    # 2. 与历史基准对比
    baseline_max = 0.8  # 历史数据基准
    condition = (max_ndvi / baseline_max) * 100

    # 3. 长势分级
    if condition > 90:
        status = "优"
    elif condition > 75:
        status = "良"
    elif condition > 60:
        status = "中"
    else:
        status = "差"

    # 4. 产量估算（简化模型）
    yield_potential = condition * 0.5  # 吨/公顷

    return {
        'condition': condition,
        'status': status,
        'yield_potential': yield_potential
    }
```

### 精准农业

```python
def prescription_map(soil_data, yield_data, nutrient_data):
    """
    生成变量施肥处方图。
    """

    # 1. 网格化分析
    # 划分农田管理分区
    from sklearn.cluster import KMeans

    features = np.column_stack([
        soil_data['organic_matter'],
        soil_data['ph'],
        yield_data['yield_t'],
        nutrient_data['nitrogen']
    ])

    # 聚类为3-4个分区
    kmeans = KMeans(n_clusters=3, random_state=42)
    zones = kmeans.fit_predict(features)

    # 2. 分区制定施肥方案
    prescriptions = {}
    for zone_id in range(3):
        zone_mask = zones == zone_id
        avg_yield = np.mean(yield_data['yield_t'][zone_mask])

        # 高产区域需更高养分
        nitrogen_rate = avg_yield * 0.02  # 千克氮/千克产量
        prescriptions[zone_id] = {
            'nitrogen': nitrogen_rate,
            'phosphorus': nitrogen_rate * 0.3,
            'potassium': nitrogen_rate * 0.4
        }

    return zones, prescriptions
```

## 林业领域

### 森林资源调查分析

```python
def estimate_biomass_from_lidar(chm_path, plot_data):
    """
    基于激光雷达冠层高度模型估算地上生物量。
    """

    # 1. 加载冠层高度模型
    with rasterio.open(chm_path) as src:
        chm = src.read(1)

    # 2. 按样地提取指标
    metrics = {}
    for plot_id, geom in plot_data.geometry.items():
        # 提取样地CHM值
        # ... (掩膜提取过程)

        plot_metrics = {
            'height_max': np.max(plot_chm),
            'height_mean': np.mean(plot_chm),
            'height_std': np.std(plot_chm),
            'height_p95': np.percentile(plot_chm, 95),
            'canopy_cover': np.sum(plot_chm > 2) / plot_chm.size
        }

        # 3. 生物量异速生长方程
        # 生物量 = a * (树高^b) * (冠盖度^c)
        biomass = 0.2 * (plot_metrics['height_mean'] ** 1.5) * \
                  (plot_metrics['canopy_cover'] ** 0.8)

        metrics[plot_id] = {
            **plot_metrics,
            'biomass_tonnes': biomass
        }

    return metrics
```

### 森林砍伐监测

```python
def detect_deforestation(image1_path, image2_path, threshold=0.3):
    """
    双时相森林砍伐检测。
    """

    # 1. 加载NDVI影像
    with rasterio.open(image1_path) as src:
        ndvi1 = src.read(1)
    with rasterio.open(image2_path) as src:
        ndvi2 = src.read(1)

    # 2. 计算NDVI变化量
    ndvi_diff = ndvi2 - ndvi1

    # 3. 检测砍伐区域（显著NDVI下降）
    deforestation = ndvi_diff < -threshold

    # 4. 移除小斑块
    deforestation_cleaned = remove_small_objects(deforestation, min_size=100)

    # 5. 计算面积
    pixel_area = 900  # 平方米（30米分辨率）
    deforested_area = np.sum(deforestation_cleaned) * pixel_area

    return deforestation_cleaned, deforested_area
```

更多领域应用示例详见[code-examples.md](code-examples.md)。
