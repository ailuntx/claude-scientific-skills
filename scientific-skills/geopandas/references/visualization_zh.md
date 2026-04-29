# 地图绘制与可视化

GeoPandas 通过集成 matplotlib 提供绘图功能。

## 基础绘图

```python
# 简单绘图
gdf.plot()

# 自定义图形尺寸
gdf.plot(figsize=(10, 10))

# 设置颜色
gdf.plot(color='blue', edgecolor='black')

# 控制线宽
gdf.plot(edgecolor='black', linewidth=0.5)
```

## 分级统计图

根据数据值着色要素：

```python
# 基础分级统计图
gdf.plot(column='population', legend=True)

# 指定色彩映射
gdf.plot(column='population', cmap='OrRd', legend=True)

# 其他色彩映射：'viridis', 'plasma', 'inferno', 'YlOrRd', 'Blues', 'Greens'
```

### 分类方案

需要安装：`uv pip install mapclassify`

```python
# 分位数法
gdf.plot(column='population', scheme='quantiles', k=5, legend=True)

# 等间距法
gdf.plot(column='population', scheme='equal_interval', k=5, legend=True)

# 自然间断法 (Fisher-Jenks)
gdf.plot(column='population', scheme='fisher_jenks', k=5, legend=True)

# 其他方案：'box_plot', 'headtail_breaks', 'max_breaks', 'std_mean'

# 向分类传递参数
gdf.plot(column='population', scheme='quantiles', k=7,
         classification_kwds={'pct': [10, 20, 30, 40, 50, 60, 70, 80, 90]})
```

### 图例定制

```python
# 将图例置于绘图区外
gdf.plot(column='population', legend=True,
         legend_kwds={'loc': 'upper left', 'bbox_to_anchor': (1, 1)})

# 水平图例
gdf.plot(column='population', legend=True,
         legend_kwds={'orientation': 'horizontal'})

# 自定义图例标签
gdf.plot(column='population', legend=True,
         legend_kwds={'label': '人口数量'})

# 为色条使用独立坐标轴
import matplotlib.pyplot as plt
fig, ax = plt.subplots(1, 1, figsize=(10, 6))
divider = make_axes_locatable(ax)
cax = divider.append_axes("right", size="5%", pad=0.1)
gdf.plot(column='population', ax=ax, legend=True, cax=cax)
```

## 处理缺失数据

```python
# 设置缺失值样式
gdf.plot(column='population',
         missing_kwds={'color': 'lightgrey', 'edgecolor': 'red', 'hatch': '///',
                      'label': '缺失数据'})
```

## 多层地图

组合多个地理数据框：

```python
import matplotlib.pyplot as plt

# 创建基础绘图
fig, ax = plt.subplots(figsize=(10, 10))

# 添加图层
gdf1.plot(ax=ax, color='lightblue', edgecolor='black')
gdf2.plot(ax=ax, color='red', markersize=5)
gdf3.plot(ax=ax, color='green', alpha=0.5)

plt.show()

# 通过zorder控制图层顺序（数值越大越靠上）
gdf1.plot(ax=ax, zorder=1)
gdf2.plot(ax=ax, zorder=2)
```

## 样式选项

```python
# 透明度
gdf.plot(alpha=0.5)

# 点要素标记样式
points.plot(marker='o', markersize=50)
points.plot(marker='^', markersize=100, color='red')

# 线样式
lines.plot(linestyle='--', linewidth=2)
lines.plot(linestyle=':', color='blue')

# 分类着色
gdf.plot(column='category', categorical=True, legend=True)

# 按列值变化标记大小
gdf.plot(markersize=gdf['value']/1000)
```

## 地图增强

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(12, 8))
gdf.plot(ax=ax, column='population', legend=True)

# 添加标题
ax.set_title('区域人口分布', fontsize=16)

# 添加坐标轴标签
ax.set_xlabel('经度')
ax.set_ylabel('纬度')

# 隐藏坐标轴
ax.set_axis_off()

# 添加指北针和比例尺（需额外安装包）
# 参见geopandas-plot或contextily获取相关功能

plt.tight_layout()
plt.show()
```

## 交互式地图

需要安装：`uv pip install folium`

```python
# 创建交互式地图
m = gdf.explore(column='population', cmap='YlOrRd', legend=True)
m.save('map.html')

# 自定义底图
m = gdf.explore(tiles='OpenStreetMap', legend=True)
m = gdf.explore(tiles='CartoDB positron', legend=True)

# 添加悬停提示
m = gdf.explore(column='population', tooltip=['name', 'population'], legend=True)

# 样式选项
m = gdf.explore(color='red', style_kwds={'fillOpacity': 0.5, 'weight': 2})

# 多层叠加
m = gdf1.explore(color='blue', name='图层1')
gdf2.explore(m=m, color='red', name='图层2')
folium.LayerControl().add_to(m)
```

## 与其他图表类型集成

GeoPandas 支持 pandas 的绘图类型：

```python
# 属性直方图
gdf['population'].plot.hist(bins=20)

# 散点图
gdf.plot.scatter(x='income', y='population')

# 箱线图
gdf.boxplot(column='population', by='region')
```

## 使用 Contextily 添加底图

需要安装：`uv pip install contextily`

```python
import contextily as ctx

# 重投影至Web Mercator坐标系以兼容底图
gdf_webmercator = gdf.to_crs(epsg=3857)

fig, ax = plt.subplots(figsize=(10, 10))
gdf_webmercator.plot(ax=ax, alpha=0.5, edgecolor='k')

# 添加底图
ctx.add_basemap(ax, source=ctx.providers.OpenStreetMap.Mapnik)
# 其他来源：ctx.providers.CartoDB.Positron, ctx.providers.Stamen.Terrain

plt.show()
```

## 使用 CartoPy 实现地图投影

需要安装：`uv pip install cartopy`

```python
import cartopy.crs as ccrs

# 创建特定投影的地图
fig, ax = plt.subplots(subplot_kw={'projection': ccrs.Robinson()}, figsize=(15, 10))

gdf.plot(ax=ax, transform=ccrs.PlateCarree(), column='population', legend=True)

ax.coastlines()
ax.gridlines(draw_labels=True)

plt.show()
```

## 保存图形

```python
# 保存至文件
ax = gdf.plot()
fig = ax.get_figure()
fig.savefig('map.png', dpi=300, bbox_inches='tight')
fig.savefig('map.pdf')
fig.savefig('map.svg')
```
