# GIS软件集成

主要GIS平台集成指南：QGIS、ArcGIS、GRASS GIS和SAGA GIS。

## QGIS / PyQGIS

### 在QGIS中运行Python脚本

```python
# 处理框架脚本
from qgis.core import (QgsProject, QgsVectorLayer, QgsRasterLayer,
                       QgsProcessingAlgorithm, QgsProcessingParameterRasterLayer)

# 加载图层
vector_layer = QgsVectorLayer("path/to/shapefile.shp", "layer_name", "ogr")
raster_layer = QgsRasterLayer("path/to/raster.tif", "raster_name", "gdal")

# 添加到工程
QgsProject.instance().addMapLayer(vector_layer)
QgsProject.instance().addMapLayer(raster_layer)

# 访问要素
for feature in vector_layer.getFeatures():
    geom = feature.geometry()
    attrs = feature.attributes()
```

### 创建QGIS处理脚本

```python
from qgis.PyQt.QtCore import QCoreApplication
from qgis.core import (QgsProcessingAlgorithm, QgsProcessingParameterRasterDestination,
                       QgsProcessingParameterRasterLayer)

class NDVIAlgorithm(QgsProcessingAlgorithm):
    INPUT = 'INPUT'
    OUTPUT = 'OUTPUT'

    def tr(self, string):
        return QCoreApplication.translate('Processing', string)

    def createInstance(self):
        return NDVIAlgorithm()

    def name(self):
        return 'ndvi_calculation'

    def displayName(self):
        return self.tr('计算NDVI')

    def group(self):
        return self.tr('栅格')

    def groupId(self):
        return 'raster'

    def shortHelpString(self):
        return self.tr("基于Sentinel-2影像计算NDVI")

    def initAlgorithm(self, config=None):
        self.addParameter(QgsProcessingParameterRasterLayer(
            self.INPUT, self.tr('输入Sentinel-2栅格')))

        self.addParameter(QgsProcessingParameterRasterDestination(
            self.OUTPUT, self.tr('输出NDVI')))

    def processAlgorithm(self, parameters, context, feedback):
        raster = self.parameterAsRasterLayer(parameters, self.INPUT, context)

        # NDVI计算
        # ...实现...

        return {self.OUTPUT: destination}
```

### 插件开发

```python
# __init__.py
def classFactory(iface):
    from .my_plugin import MyPlugin
    return MyPlugin(iface)

# my_plugin.py
from qgis.PyQt.QtCore import QSettings
from qgis.PyQt.QtWidgets import QAction
from qgis.core import QgsProject

class MyPlugin:
    def __init__(self, iface):
        self.iface = iface

    def initGui(self):
        self.action = QAction("我的插件", self.iface.mainWindow())
        self.action.triggered.connect(self.run)
        self.iface.addPluginToMenu("我的插件", self.action)

    def run(self):
        # 插件逻辑在此
        pass

    def unload(self):
        self.iface.removePluginMenu("我的插件", self.action)
```

## ArcGIS / ArcPy

### 基本ArcPy操作

```python
import arcpy

# 设置工作空间
arcpy.env.workspace = "C:/data"

# 设置输出覆盖
arcpy.env.overwriteOutput = True

# 设置临时工作空间
arcpy.env.scratchWorkspace = "C:/data/scratch"

# 列出要素
feature_classes = arcpy.ListFeatureClasses()
rasters = arcpy.ListRasters()
```

### 地理处理工作流

```python
import arcpy
from arcpy.sa import *

# 检出Spatial Analyst扩展
arcpy.CheckOutExtension("Spatial")

# 设置环境
arcpy.env.workspace = "C:/data"
arcpy.env.cellSize = 10
arcpy.env.extent = "study_area"

# 坡度分析
out_slope = Slope("dem.tif")
out_slope.save("slope.tif")

# 坡向
out_aspect = Aspect("dem.tif")
out_aspect.save("aspect.tif")

# 山体阴影
out_hillshade = Hillshade("dem.tif", azimuth=315, altitude=45)
out_hillshade.save("hillshade.tif")

# 视域分析
out_viewshed = Viewshed("observer_points.shp", "dem.tif", obs_elevation_field="HEIGHT")
out_viewshed.save("viewshed.tif")

# 成本距离
cost_raster = CostDistance("source.shp", "cost.tif")
cost_raster.save("cost_distance.tif")

# 水文：流向
flow_dir = FlowDirection("dem.tif")
flow_dir.save("flowdir.tif")

# 流量累积
flow_acc = FlowAccumulation(flow_dir)
flow_acc.save("flowacc.tif")

# 河流提取
stream = Con(flow_acc > 1000, 1)
stream_raster = StreamOrder(stream, flow_dir)
```

### 矢量分析

```python
# 缓冲区分析
arcpy.Buffer_analysis("roads.shp", "roads_buffer.shp", "100 meters")

# 空间连接
arcpy.SpatialJoin_analysis("points.shp", "zones.shp", "points_joined.shp",
                           join_operation="JOIN_ONE_TO_ONE",
                           match_option="HAVE_THEIR_CENTER_IN")

# 融合
arcpy.Dissolve_management("parcels.shp", "parcels_dissolved.shp",
                          dissolve_field="OWNER_ID")

# 相交
arcpy.Intersect_analysis(["layer1.shp", "layer2.shp"], "intersection.shp")

# 裁剪
arcpy.Clip_analysis("input.shp", "clip_boundary.shp", "output.shp")

# 按位置选择
arcpy.SelectLayerByLocation_management("points_layer", "HAVE_THEIR_CENTER_IN",
                                      "polygon_layer")

# 要素转栅格
arcpy.FeatureToRaster_conversion("landuse.shp", "LU_CODE", "landuse.tif", 10)
```

### ArcGIS Pro笔记本

```python
# ArcGIS Pro Jupyter笔记本
import arcpy
import pandas as pd
import matplotlib.pyplot as plt

# 使用当前工程的地图
aprx = arcpy.mp.ArcGISProject("CURRENT")
m = aprx.listMaps()[0]

# 获取图层
layer = m.listLayers("Parcels")[0]

# 导出至空间数据框
sdf = pd.DataFrame.spatial.from_layer(layer)

# 绘图
sdf.plot(column='VALUE', cmap='YlOrRd', legend=True)
plt.show()

# 地理编码地址
locator = "C:/data/locators/composite.locator"
results = arcpy.geocoding.GeocodeAddresses(
    "addresses.csv", locator, "Address Address",
    None, "geocoded_results.gdb"
)
```

## GRASS GIS

### GRASS的Python API

```python
import grass.script as gscript
import grass.script.array as garray

# 初始化GRASS会话
gscript.run_command('g.gisenv', set='GISDBASE=/grassdata')
gscript.run_command('g.gisenv', set='LOCATION_NAME=nc_spm_08')
gscript.run_command('g.gisenv', set='MAPSET=user1')

# 导入栅格
gscript.run_command('r.in.gdal', input='elevation.tif', output='elevation')

# 导入矢量
gscript.run_command('v.in.ogr', input='roads.shp', output='roads')

# 获取栅格信息
info = gscript.raster_info('elevation')
print(info)

# 坡度分析
gscript.run_command('r.slope.aspect', elevation='elevation',
                    slope='slope', aspect='aspect')

# 缓冲区
gscript.run_command('v.buffer', input='roads', output='roads_buffer',
                    distance=100)

# 叠加
gscript.run_command('v.overlay', ainput='zones', binput='roads',
                    operator='and', output='zones_roads')

# 计算统计量
stats = gscript.parse_command('r.univar', map='elevation', flags='g')
```

## SAGA GIS

### 通过命令行使用SAGA

```python
import subprocess
import os

# SAGA路径
saga_cmd = "/usr/local/saga/saga_cmd"

# 栅格计算
def saga_grid_calculus(input1, input2, output, formula):
    cmd = [
        saga_cmd, "grid_calculus", "GridCalculator",
        f"-GRIDS={input1};{input2}",
        f"-RESULT={output}",
        f"-FORMULA={formula}"
    ]
    subprocess.run(cmd)

# 坡度分析
def saga_slope(dem, output_slope):
    cmd = [
        saga_cmd, "ta_morphometry", "SlopeAspectCurvature",
        f"-ELEVATION={dem}",
        f"-SLOPE={output_slope}"
    ]
    subprocess.run(cmd)

# 形态测量特征
def saga_morphometry(dem):
    cmd = [
        saga_cmd, "ta_morphometry", "MorphometricFeatures",
        f"-DEM={dem}",
        f"-SLOPE=slope.sgrd",
        f"-ASPECT=aspect.sgrd",
        f"-CURVATURE=curvature.sgrd"
    ]
    subprocess.run(cmd)

# 河网提取
def saga_channels(dem, threshold=1000):
    cmd = [
        saga_cmd, "ta_channels", "ChannelNetworkAndDrainageBasins",
        f"-ELEVATION={dem}",
        f"-CHANNELS=channels.shp",
        f"-BASINS=basins.shp",
        f"-THRESHOLD={threshold}"
    ]
    subprocess.run(cmd)
```

## 跨平台工作流

### 从QGIS导出至ArcGIS

```python
import geopandas as gpd

# 读取在QGIS中处理的数据
gdf = gpd.read_file('qgis_output.geojson')

# 确保坐标参考系统
gdf = gdf.to_crs('EPSG:32633')

# 导出至ArcGIS（文件地理数据库）
gdf.to_file('arcgis_input.gpkg', driver='GPKG')
# ArcGIS可直接读取GPKG

# 或导出为shapefile
gdf.to_file('arcgis_input.shp')
```

### 批处理

```python
import geopandas as
