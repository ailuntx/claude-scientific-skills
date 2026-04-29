# 多语言地理空间编程

跨8种语言的地理空间编程：R、Julia、JavaScript、C++、Java、Go、Rust和Python。

## R地理空间

### sf（简单要素）

```r
library(sf)
library(dplyr)
library(ggplot2)

# 读取空间数据
roads <- st_read("roads.shp")
zones <- st_read("zones.geojson")

# 基础操作
st_crs(roads)  # 检查坐标系
roads_utm <- st_transform(roads, 32610)  # 重投影

# 几何操作
roads_buffer <- st_buffer(roads, dist = 100)  # 缓冲区
roads_simplify <- st_simplify(roads, tol = 0.0001)  # 简化
roads_centroid <- st_centroid(roads)  # 质心

# 空间连接
joined <- st_join(roads, zones, join = st_intersects)

# 叠加分析
intersection <- st_intersection(roads, zones)

# 绘图
ggplot() +
  geom_sf(data = zones, fill = NA) +
  geom_sf(data = roads, color = "blue") +
  theme_minimal()

# 计算面积
zones$area <- st_area(zones)  # 使用坐标系单位
zones$area_km2 <- st_area(zones) / 1e6  # 转换为平方公里
```

### terra（栅格处理）

```r
library(terra)

# 加载栅格
r <- rast("elevation.tif")

# 基础信息
r
ext(r)  # 范围
crs(r)  # 坐标系
res(r)  # 分辨率

# 栅格计算
slope <- terrain(r, v = "slope")
aspect <- terrain(r, v = "aspect")

# 多栅格操作
ndvi <- (s2[[8]] - s2[[4]]) / (s2[[8]] + s2[[4]])

# 焦点操作
focal_mean <- focal(r, w = matrix(1, 3, 3), fun = mean)
focal_sd <- focal(r, w = matrix(1, 5, 5), fun = sd)

# 分区统计
zones <- vect("zones.shp")
zonal_mean <- zonal(r, zones, fun = mean)

# 点值提取
points <- vect("points.shp")
values <- extract(r, points)

# 输出结果
writeRaster(slope, "slope.tif", overwrite = TRUE)
```

### R工作流

```r
# 完整土地覆盖分类
library(sf)
library(terra)
library(randomForest)
library(caret)

# 1. 加载数据
training <- st_read("training.shp")
s2 <- rast("sentinel2.tif")

# 2. 提取训练数据
training_points <- st_centroid(training)
values <- extract(s2, training_points)

# 3. 合并标签
df <- data.frame(values)
df$class <- as.factor(training$class_id)

# 4. 训练模型
set.seed(42)
train_index <- createDataPartition(df$class, p = 0.7, list = FALSE)
train_data <- df[train_index, ]
test_data <- df[-train_index, ]

rf_model <- randomForest(class ~ ., data = train_data, ntree = 100)

# 5. 预测
predicted <- predict(s2, model = rf_model)

# 6. 精度评估
conf_matrix <- confusionMatrix(predict(rf_model, test_data), test_data$class)
print(conf_matrix)

# 7. 导出结果
writeRaster(predicted, "classified.tif", overwrite = TRUE)
```

## Julia地理空间

### ArchGDAL.jl

```julia
using ArchGDAL
using GeoInterface

# 注册驱动
ArchGDAL.registerdrivers() do
    # 读取shapefile
    data = ArchGDAL.read("countries.shp") do dataset
        layer = dataset[1]
        features = []
        for feature in layer
            geom = ArchGDAL.getgeom(feature)
            push!(features, geom)
        end
        features
    end
end

# 创建几何对象
using GeoInterface

point = GeoInterface.Point(-122.4, 37.7)
polygon = GeoInterface.Polygon([GeoInterface.LinearRing([
    GeoInterface.Point(-122.5, 37.5),
    GeoInterface.Point(-122.3, 37.5),
    GeoInterface.Point(-122.3, 37.8),
    GeoInterface.Point(-122.5, 37.8),
    GeoInterface.Point(-122.5, 37.5)
])])

# 几何操作
buffered = GeoInterface.buffer(point, 1000)
intersection = GeoInterface.intersection(poly1, poly2)
```

### GeoStats.jl

```julia
using GeoStats
using GeoStatsBase
using Variography

# 加载点数据
data = georef((value = [1.0, 2.0, 3.0],),
              [Point(0.0, 0.0), Point(1.0, 0.0), Point(0.5, 1.0)])

# 实验变异函数
γ = variogram(EmpiricalVariogram, data, :value, maxlag = 1.0)

# 拟合理论变异函数
γfit = fit(EmpiricalVariogram, γ, SphericalVariogram)

# 普通克里金法
problem = OrdinaryKriging(data, :value, γfit)
solution = solve(problem)

# 模拟
simulation = SimulationProblem(data, :value, SphericalVariogram, 100)
result = solve(simulation)
```

## JavaScript（Node.js与浏览器）

### Turf.js（浏览器/Node）

```javascript
// npm install @turf/turf
const turf = require('@turf/turf');

// 创建要素
const pt1 = turf.point([-122.4, 37.7]);
const pt2 = turf.point([-122.3, 37.8]);

// 距离计算（公里）
const distance = turf.distance(pt1, pt2, { units: 'kilometers' });

// 缓冲区
const buffered = turf.buffer(pt1, 5, { units: 'kilometers' });

// 边界框
const bbox = turf.bbox(buffered);

// 沿线定位
const line = turf.lineString([[-122.4, 37.7], [-122.3, 37.8]]);
const along = turf.along(line, 2, { units: 'kilometers' });

// 点面包含
const points = turf.points([
  [-122.4, 37.7],
  [-122.35, 37.75],
  [-122.3, 37.8]
]);
const polygon = turf.polygon([[[-122.4, 37.7], [-122.3, 37.7], [-122.3, 37.8], [-122.4, 37.8], [-122.4, 37.7]]]);
const ptsWithin = turf.pointsWithinPolygon(points, polygon);

// 最近点
const nearest = turf.nearestPoint(pt1, points);

// 面积计算
const area = turf.area(polygon); // 平方米

```

### Leaflet（网络地图）

```javascript
// 初始化地图
const map = L.map('map').setView([37.7, -122.4], 13);

// 添加瓦片图层
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
  attribution: '© OpenStreetMap contributors'
}).addTo(map);

// 添加GeoJSON图层
fetch('data.geojson')
  .then(response => response.json())
  .then(data => {
    L.geoJSON(data, {
      style: function(feature) {
        return { color: feature.properties.color };
      },
      onEachFeature: function(feature, layer) {
        layer.bindPopup(feature.properties.name);
      }
    }).addTo(map);
  });

// 添加标记
const marker = L.marker([37.7, -122.4]).addTo(map);
marker.bindPopup("你好！").openPopup();

// 绘制圆形
const circle = L.circle([37.7, -122.4], {
  color: 'red',
  fillColor: '#f03',
  fillOpacity: 0.5,
  radius: 500
}).addTo(map);
```

## C++地理空间

### GDAL C++ API

```cpp
#include "gdal_priv.h"
#include "ogr_api.h"
#include "ogr_spatialref.h"

// 打开栅格
GDALDataset *poDataset = (GDALDataset *) GDALOpen("input.tif", GA_ReadOnly);

// 获取波段
GDALRasterBand *poBand = poDataset->GetRasterBand(1);

// 读取数据
int nXSize = poBand->GetXSize();
int nYSize = poBand->GetYSize();
float *pafScanline = (float *) CPLMalloc(sizeof(float) * nXSize);
poBand->RasterIO(GF_Read, 0, 0, nXSize, 1,
                 pafScanline, nXSize, 1, GDT_Float32, 0, 0);

// 矢量数据
GDALDataset *poDS = (GDALDataset *) GDALOpenEx("roads.shp",
    GDAL_OF_VECTOR, NULL, NULL, NULL);
OGRLayer *poLayer = poDS->GetLayer(0);

OGRFeature *poFeature;
poLayer->ResetReading();
while ((poFeature = poLayer->GetNextFeature()) != NULL) {
    OGRGeometry *poGeometry = poFeature->GetGeometryRef();
    // 处理几何对象
    OGRFeature::DestroyFeature(poFeature);
}

GDALClose(poDS);
```

## Java地理空间

### GeoTools

```java
import org.geotools.data.FileDataStore;
import org.geotools.data.FileDataStoreFinder;
import org.geotools.data.simple.SimpleFeatureCollection;
import org.geotools.data.simple.SimpleFeatureIterator;
import org.geotools.data.simple.SimpleFeatureSource;
import org.geotools.geometry.jts.JTS;
import org.geotools.referencing.CRS;
import org.opengis.feature.simple.SimpleFeature;
import org.opengis.referencing.crs.CoordinateReferenceSystem;

import org.locationtech.jts.geom.Coordinate;
import org.locationtech.jts.geom.GeometryFactory;
import org.locationtech.jts.geom.Point;

// 加载shapefile
File file = new File("roads.shp");
FileDataStore store = FileDataStoreFinder.getDataStore(file);
SimpleFeatureSource featureSource = store.getFeatureSource();

// 读取要素
SimpleFeatureCollection collection = featureSource.getFeatures();
try (SimpleFeatureIterator iterator = collection.features()) {
    while (iterator.hasNext()) {
        SimpleFeature feature = iterator.next();
        Geometry geom = (Geometry) feature.getDefaultGeometryProperty().getValue();
        // 处理几何对象
    }
}

// 创建点
GeometryFactory gf = new GeometryFactory();
Point point = gf.createPoint(new Coordinate(-122.4, 37.7));

// 重投影
CoordinateReferenceSystem sourceCRS = CRS.decode("EPSG:4326");
CoordinateReferenceSystem targetCRS = CRS.decode("EPSG:32633");
MathTransform transform = CRS.findMathTransform(sourceCRS, targetCRS);
Geometry reprojected = JTS.transform(point, transform);
```

## Go地理空间

### Simple Features Go

```go
package main

import (
    "fmt"
    "github.com/paulmach/orb"
    "github.com/paulmach/orb/geojson"
    "github.com/paulmach/orb/planar"
)

func main() {
    // 创建点
    point := orb.Point{122.4, 37.7}

    // 创建线
    line := orb.LineString{
        {122.4, 37.7},
        {122.3, 37.8},
    }

    // 创建面
    polygon := orb.Polygon{
        {{122.4, 37.7}, {122.3, 37.7}, {122.3, 37.8}, {122.4, 37.8}, {122.4, 37.7}},
    }

    // GeoJSON要素
    feature := geojson.NewFeature(polygon)
    feature.Properties["name"] = "Zone 1"

    // 距离计算（平面）
    distance := planar.Distance(point, orb.Point{122.3, 37.8})

    // 面积计算
    area := planar.Area(polygon)

    fmt.Printf("距离: %.2f 米\n", distance)
    fmt.Printf("面积: %.2f 平方米\n", area)
}
```

更多语言代码示例请参见[code-examples.md](code-examples.md)。

## Rust地理空间

### GeoRust（地理空间Rust）

Rust地理空间生态包含几何操作、投影转换和文件I/O的crate。

\`\`\`rust
// Cargo.toml依赖项:
// geo = "0.28"
// geo-types = "0.7"
// proj = "0.27"
// shapefile = "0.5"

use geo::{Coord, Point, LineString, Polygon, Geometry};
use geo::prelude::*;
use proj::Proj;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 创建点
    let point = Point::new(-122.4_f64, 37.7_f64);

    // 创建线
    let linestring = LineString::new(vec![
        Coord { x: -122.4, y: 37.7 },
        Coord { x: -122.3, y: 37.8 },
        Coord { x: -122.2, y: 37.9 },
    ]);

    // 创建面
    let polygon = Polygon::new(
        LineString::new(vec![
            Coord { x: -122.4, y: 37.7 },
            Coord { x: -122.3, y:  37.7 },
            Coord { x: -122.3, y: 37.8 },
            Coord { x: -122.4, y: 37.8 },
            Coord { x: -2.4, y: 37.7 }, // 闭合环
        ]),
        vec![], // 无内环
    );

    // 几何操作
    let buffered = polygon.buffer(1000.0); // 缓冲区（坐标系单位）
    let centroid = polygon.centroid();
    let convex_hull = polygon.convex_hull();
    let simplified = polygon.simplify(&1.0); // 容差

    // 空间关系
    let point_within = point.within(&polygon);
    let line_intersects = linestring.intersects(&polygon);

    // 坐标转换
    let from = "EPSG:4326";
    let to = "EPSG:32610";
    let proj = Proj::new_known_crs(from, to, None)?;
    let transformed = proj.convert(point)?;

    println!("点坐标: {:?}", point);
    println!("是否在面内: {}", point_within);

    Ok(())
}
\`\`\`
