# 高级GIS专题

高级空间分析技术：三维GIS、时空分析、拓扑与网络分析。

## 三维GIS

### 三维矢量操作

```python
import geopandas as gpd
from shapely.geometry import Point, LineString, Polygon
import pyproj
import numpy as np

# 创建三维几何体（带Z坐标）
point_3d = Point(0, 0, 100)  # x, y, 高程
line_3d = LineString([(0, 0, 0), (100, 100, 50)])

# 加载三维数据
gdf_3d = gpd.read_file('buildings_3d.geojson')

# 访问Z坐标
gdf_3d['height'] = gdf_3d.geometry.apply(lambda g: g.coords[0][2] if g.has_z else None)

# 三维缓冲区（圆柱体）
def buffer_3d(point, radius, height):
    """创建三维圆柱形缓冲区"""
    base = Point(point.x, point.y).buffer(radius)
    # 拉伸至三维（概念性）
    return base, point.z, point.z + height

# 三维距离（三维空间欧几里得距离）
def distance_3d(point1, point2):
    """计算三维欧几里得距离"""
    dx = point2.x - point1.x
    dy = point2.y - point1.y
    dz = point2.z - point1.z
    return np.sqrt(dx**2 + dy**2 + dz**2)
```

### 三维栅格分析

```python
import rasterio
import numpy as np

# 体素分析
def voxel_analysis(dem_path, dsm_path):
    """分析DEM与DSM之间的体积"""
    with rasterio.open(dem_path) as src_dem:
        dem = src_dem.read(1)
        transform = src_dem.transform

    with rasterio.open(dsm_path) as src_dsm:
        dsm = src_dsm.read(1)

    # 高度差
    height = dsm - dem

    # 体积计算
    pixel_area = transform[0] * transform[4]  # 通常为负值
    volume = np.sum(height[height > 0]) * abs(pixel_area)

    # 按高度等级统计体积
    height_bins = [0, 5, 10, 20, 50, 100]
    volume_by_class = {}

    for i in range(len(height_bins) - 1):
        mask = (height >= height_bins[i]) & (height < height_bins[i + 1])
        volume_by_class[f'{height_bins[i]}-{height_bins[i+1]}m'] = \
            np.sum(height[mask]) * abs(pixel_area)

    return volume, volume_by_class
```

### 视域分析

```python
def viewshed(dem, observer_x, observer_y, observer_height=1.7, max_distance=5000):
    """
    使用视线算法计算视域
    """

    # 将观察点转换为栅格坐标
    observer_row = int((observer_y - dem_origin_y) / cell_size)
    observer_col = int((observer_x - dem_origin_x) / cell_size)

    rows, cols = dem.shape
    viewshed = np.zeros_like(dem, dtype=bool)

    observer_z = dem[observer_row, observer_col] + observer_height

    # 遍历每个方向
    for angle in np.linspace(0, 2*np.pi, 360):
        # 投射射线
        for r in range(1, int(max_distance / cell_size)):
            row = observer_row + int(r * np.sin(angle))
            col = observer_col + int(r * np.cos(angle))

            if row < 0 or row >= rows or col < 0 or col >= cols:
                break

            target_z = dem[row, col]

            # 视线计算
            dist = r * cell_size
            line_height = observer_z + (target_z - observer_z) * (dist / max_distance)

            if target_z > line_height:
                viewshed[row, col] = False
            else:
                viewshed[row, col] = True

    return viewshed
```

## 时空分析

### 轨迹分析

```python
import movingpandas as mpd
import geopandas as gpd
import pandas as pd

# 从点数据创建轨迹
gdf = gpd.read_file('gps_points.gpkg')

# 转换为轨迹
traj_collection = mpd.TrajectoryCollection(gdf, 'track_id', t='timestamp')

# 分割轨迹（例如按时间间隔）
traj_collection = mpd.SplitByObservationGap(traj_collection, gap=pd.Timedelta('1 hour'))

# 轨迹统计
for traj in traj_collection:
    print(f"轨迹 {traj.id}:")
    print(f"  长度: {traj.get_length() / 1000:.2f} 公里")
    print(f"  时长: {traj.get_duration()}")
    print(f"  速度: {traj.get_speed() * 3.6:.2f} 公里/小时")

# 停留点检测
stops = mpd.stop_detection(
    traj_collection,
    max_diameter=100,  # 米
    min_duration=pd.Timedelta('5 minutes')
)

# 轨迹简化
traj_generalized = mpd.DouglasPeuckerGeneralizer(traj_collection, tolerance=10).generalize()

# 按停留点分割
traj_moving, stops = mpd.StopSplitter(traj_collection).split()
```

### 时空立方体

```python
def create_space_time_cube(gdf, time_column='timestamp', grid_size=100, time_step='1H'):
    """
    创建用于热点分析的三维时空立方体
    """

    # 1. 空间分箱
    gdf['x_bin'] = (gdf.geometry.x // grid_size).astype(int)
    gdf['y_bin'] = (gdf.geometry.y // grid_size).astype(int)

    # 2. 时间分箱
    gdf['t_bin'] = gdf[time_column].dt.floor(time_step)

    # 3. 创建立方体（x, y, 时间）
    cube = gdf.groupby(['x_bin', 'y_bin', 't_bin']).size().unstack(fill_value=0)

    return cube

def emerging_hot_spot_analysis(cube, k=8):
    """
    新兴热点分析（ArcGIS实现方式简化版）
    使用Getis-Ord Gi*统计量
    """
    from esda.getisord import G_Local

    # 计算每个时间步的Gi*统计量
    hotspots = {}
    for timestep in cube.columns:
        data = cube[timestep].values.reshape(-1, 1)
        g_local = G_Local(data, k=k)
        hotspots[timestep] = g_local.p_sim < 0.05  # 显著热点

    return hotspots
```

## 拓扑

### 拓扑关系

```python
from shapely.geometry import Point, LineString, Polygon
from shapely.ops import unary_union

# 平面图构建
def build_planar_graph(lines_gdf):
    """从线要素构建平面图"""
    import networkx as nx

    G = nx.Graph()

    # 在交点处添加节点
    for i, line1 in lines_gdf.iterrows():
        for j, line2 in lines_gdf.iterrows():
            if i < j:
                if line1.geometry.intersects(line2.geometry):
                    intersection = line1.geometry.intersection(line2.geometry)
                    G.add_node((intersection.x, intersection.y))

    # 添加边
    for _, line in lines_gdf.iterrows():
        coords = list(line.geometry.coords)
        G.add_edge(coords[0], coords[-1],
                   weight=line.geometry.length,
                   geometry=line.geometry)

    return G

# 拓扑验证
def validate_topology(gdf):
    """检查拓扑错误"""

    errors = []

    # 1. 检查缝隙
    if gdf.geom_type.iloc[0] == 'Polygon':
        dissolved = unary_union(gdf.geometry)
        for i, geom in enumerate(gdf.geometry):
            if not geom.touches(dissolved - geom):
                errors.append(f"要素{i}检测到缝隙")

    # 2. 检查重叠
    for i, geom1 in enumerate(gdf.geometry):
        for j, geom2 in enumerate(gdf.geometry):
            if i < j and geom1.overlaps(geom2):
                errors.append(f"要素{i}与{j}存在重叠")

    # 3. 检查自相交
    for i, geom in enumerate(gdf.geometry):
        if not geom.is_valid:
            errors.append(f"要素{i}存在自相交: {geom.is_valid}")

    return errors
```

## 网络分析

### 高级路径分析

```python
import osmnx as ox
import networkx as nx

# 下载并准备网络
G = ox.graph_from_place('Portland, Maine, USA', network_type='drive')
G = ox.add_edge_speeds(G)
G = ox.add_edge_travel_times(G)

# 多准则路径规划
def multi_criteria_routing(G, orig, dest, weights=['length', 'travel_time']):
    """
    基于多准则优化路径
    """
    # 归一化权重
    for w in weights:
        values = [G.edges[e][w] for e in G.edges]
        min_val, max_val = min(values), max(values)
        for e in G.edges:
            G.edges[e][f'{w}_norm'] = (G.edges[e][w] - min_val) / (max_val - min_val)

    # 组合权重
    for e in G.edges:
        G.edges[e]['combined'] = sum(G.edges[e][f'{w}_norm'] for w in weights) / len(weights)

    # 查找路径
    route = nx.shortest_path(G, orig, dest, weight='combined')
    return route

# 等时线（可达区域）
def isochrone(G, center_node, time_limit=600):
    """
    计算时间限制内的可达区域
    """
    # 获取可达节点子图
    subgraph = nx.ego_graph(G, center_node,
                            radius=time_limit,
                            distance='travel_time')

    # 获取节点几何
    nodes = ox.graph_to_gdfs(subgraph, edges=False)

    # 创建可达区域多边形
    from shapely.geometry import MultiPoint
    points = MultiPoint(nodes.geometry.tolist())
    isochrone_polygon = points.convex_hull

    return isochrone_polygon, subgraph

# 中介中心性（节点重要性）
def calculate_centrality(G):
    """
    计算网络中介中心性
    """
    centrality = nx.betweenness_centrality(G, weight='length')

    # 添加到节点属性
    for node, value in centrality.items():
        G.nodes[node]['betweenness'] = value

    return centrality
```

### 服务区分析

```python
def service_area(G, facilities, max_distance=1000):
    """
    计算设施服务区
    """

    service_areas = []

    for facility in facilities:
        # 查找最近节点
        node = ox.distance.nearest_nodes(G, facility.x, facility.y)

        # 获取距离内节点
        subgraph = nx.ego_graph(G, node, radius=max_distance, distance='length')

        # 创建凸包
        nodes = ox.graph_to_gdfs(subgraph, edges=False)
        service_area = nodes.geometry.unary_union.convex_hull

        service_areas.append({
            'facility': facility,
            'area': service_area,
            'nodes_served': len(subgraph.nodes())
        })

    return service_areas

# 区位分配（设施选址）
def location_allocation(demand_points, candidate_sites, n_facilities=5):
    """
    解决设施选址问题（p中位数）
    """
    from scipy.spatial.distance import cdist

    # 距离矩阵
    coords_demand = [[p.x, p.y] for p in demand_points]
    coords_sites = [[s.x, s.y] for s in candidate_sites]
    distances = cdist(coords_demand, coords_sites)

    # 简单启发式：K均值聚类
    from sklearn.cluster import KMeans

    kmeans = KMeans(n_clusters=n_facilities, random_state=42)
    labels = kmeans.fit_predict(coords_demand)

    # 为每个聚类中心寻找最近候选点
    facilities = []
    for i in range(n_facilities):
        cluster_center = kmeans.cluster_centers_[i]
        nearest_site_idx = np.argmin(cdist([cluster_center], coords_sites))
        facilities.append(candidate_sites[nearest_site_idx])

    return facilities
```

更多高级示例见[code-examples.md](code-examples.md)。
