# GeoMaster 地理空间科学技能

## 概述

GeoMaster 是一项全面的地理空间科学技能，涵盖：
- **70+ 章节** 地理空间科学主题
- **500+ 代码示例** 覆盖 7 种编程语言
- **300+ 地理空间库** 与工具
- 遥感、GIS、空间统计、地球观测中的机器学习/人工智能

## 内容目录

### 核心文档
- **SKILL.md** - 主技能文档，包含安装指南、快速入门、核心概念、常用操作和工作流程

### 参考文档
1. **core-libraries.md** - GDAL, Rasterio, Fiona, Shapely, PyProj, GeoPandas
2. **remote-sensing.md** - 卫星任务、光学/SAR/高光谱分析、影像处理
3. **gis-software.md** - QGIS/PyQGIS, ArcGIS/ArcPy, GRASS GIS, SAGA GIS 集成
4. **scientific-domains.md** - 海洋、大气、水文、农业、林业应用
5. **advanced-gis.md** - 3D GIS、时空分析、拓扑学、网络分析
6. **programming-languages.md** - R, Julia, JavaScript, C++, Java, Go 地理空间工具
7. **machine-learning.md** - 遥感深度学习、空间机器学习、图神经网络、地理空间可解释AI
8. **big-data.md** - 分布式处理、云平台、GPU加速
9. **industry-applications.md** - 城市规划、灾害管理、公共事业、交通运输
10. **specialized-topics.md** - 地统计学、优化方法、伦理规范、最佳实践
11. **data-sources.md** - 卫星数据目录、开放数据仓库、API访问
12. **code-examples.md** - 7种编程语言的500+代码示例

## 核心主题覆盖

### 遥感技术
- Sentinel-1/2/3, Landsat, MODIS, Planet, Maxar
- SAR、高光谱、激光雷达、热成像
- 光谱指数、分类、变化检测

### GIS 操作
- 矢量数据（点、线、面）
- 栅格数据处理
- 坐标参考系统
- 空间分析与统计

### 机器学习
- 随机森林、支持向量机、卷积神经网络、U-Net
- 空间统计、地统计学
- 图神经网络
- 可解释人工智能

### 编程语言
- **Python** - GDAL, Rasterio, GeoPandas, TorchGeo, RSGISLib
- **R** - sf, terra, raster, stars
- **Julia** - ArchGDAL, GeoStats.jl
- **JavaScript** - Turf.js, Leaflet
- **C++** - GDAL C++ API
- **Java** - GeoTools
- **Go** - Simple Features Go

## 安装指南

详见 [SKILL.md](SKILL.md) 获取详细安装说明。

### 核心 Python 工具栈
```bash
conda install -c conda-forge gdal rasterio fiona shapely pyproj geopandas
```

### 遥感工具
```bash
pip install rsgislib torchgeo earthengine-api
```

## 快速示例

### 基于 Sentinel-2 计算 NDVI
```python
import rasterio
import numpy as np

with rasterio.open('sentinel2.tif') as src:
    red = src.read(4)
    nir = src.read(8)
    ndvi = (nir - red) / (nir + red + 1e-8)
```

### 使用 GeoPandas 进行空间分析
```python
import geopandas as gpd

zones = gpd.read_file('zones.geojson')
points = gpd.read_file('points.geojson')
joined = gpd.sjoin(points, zones, predicate='within')
```

## 许可证

MIT 许可证

## 作者

K-Dense Inc.

## 贡献指南

本技能属于 K-Dense-AI/scientific-agent-skills 代码库。
贡献流程请参考主代码库指南。
