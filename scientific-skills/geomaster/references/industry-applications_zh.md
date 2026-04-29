# 行业应用

跨行业的真实世界地理空间工作流：城市规划、灾害管理、公共事业等。

## 城市规划

### 土地利用分类

```python
def classify_urban_land_use(sentinel2_path, training_data_path):
    """
    城市土地利用分类工作流。
    类别：居住区、商业区、工业区、绿地、水域
    """
    from sklearn.ensemble import RandomForestClassifier
    import geopandas as gpd
    import rasterio

    # 1. 加载训练数据
    training = gpd.read_file(training_data_path)

    # 2. 提取光谱与纹理特征
    features = extract_features(sentinel2_path, training)

    # 3. 训练分类器
    rf = RandomForestClassifier(n_estimators=100, max_depth=20)
    rf.fit(features['X'], features['y'])

    # 4. 全图分类
    classified = classify_image(sentinel2_path, rf)

    # 5. 后处理
    cleaned = remove_small_objects(classified, min_size=100)
    smoothed = majority_filter(cleaned, size=3)

    # 6. 计算统计指标
    stats = calculate_class_statistics(cleaned)

    return cleaned, stats

def extract_features(image_path, training_gdf):
    """提取光谱与纹理特征"""
    with rasterio.open(image_path) as src:
        image = src.read()
        profile = src.profile

    # 光谱特征
    features = {
        'NDVI': (image[7] - image[3]) / (image[7] + image[3] + 1e-8),
        'NDWI': (image[2] - image[7]) / (image[2] + image[7] + 1e-8),
        'NDBI': (image[10] - image[7]) / (image[10] + image[7] + 1e-8),
        'UI': (image[10] + image[3]) / (image[7] + image[2] + 1e-8)  # 城市指数
    }

    # 纹理特征 (GLCM)
    from skimage.feature import graycomatrix, graycoprops

    textures = {}
    for band_idx in [3, 7, 10]:  # 红波段、近红外、短波红外
        band = image[band_idx]
        band_8bit = ((band - band.min()) / (band.max() - band.min()) * 255).astype(np.uint8)

        glcm = graycomatrix(band_8bit, distances=[1], angles=[0], levels=256, symmetric=True)
        contrast = graycoprops(glcm, 'contrast')[0, 0]
        homogeneity = graycoprops(glcm, 'homogeneity')[0, 0]

        textures[f'contrast_{band_idx}'] = contrast
        textures[f'homogeneity_{band_idx}'] = homogeneity

    # 合并所有特征
    # ... (实现细节)

    return features
```

### 人口估算

```python
def dasymetric_population(population_raster, land_use_classified):
    """
    密度表面人口重分配
    """
    # 1. 识别可居住区域
    inhabitable_mask = (
        (land_use_classified != 0) &  # 水域
        (land_use_classified != 4) &  # 工业区
        (land_use_classified != 5)    # 道路
    )

    # 2. 按土地利用类型分配权重
    weights = np.zeros_like(land_use_classified, dtype=float)
    weights[land_use_classified == 1] = 1.0  # 居住区
    weights[land_use_classified == 2] = 0.3  # 商业区
    weights[land_use_classified == 3] = 0.5  # 绿地

    # 3. 计算权重层
    weighting_layer = weights * inhabitable_mask
    total_weight = np.sum(weighting_layer)

    # 4. 重分配人口
    total_population = np.sum(population_raster)
    redistributed = population_raster * (weighting_layer / total_weight) * total_population

    return redistributed
```

## 灾害管理

### 洪水风险评估

```python
def flood_risk_assessment(dem_path, river_path, return_period_years=100):
    """
    综合洪水风险评估
    """

    # 1. 水文建模
    flow_accumulation = calculate_flow_accumulation(dem_path)
    flow_direction = calculate_flow_direction(dem_path)
    watershed = delineate_watershed(dem_path, flow_direction)

    # 2. 洪水范围估算
    flood_depth = estimate_flood_extent(dem_path, river_path, return_period_years)

    # 3. 暴露分析
    settlements = gpd.read_file('settlements.shp')
    roads = gpd.read_file('roads.shp')
    infrastructure = gpd.read_file('infrastructure.shp')

    exposed_settlements = gpd.clip(settlements, flood_extent_polygon)
    exposed_roads = gpd.clip(roads, flood_extent_polygon)

    # 4. 脆弱性评估
    vulnerability = assess_vulnerability(exposed_settlements)

    # 5. 风险计算
    risk = flood_depth * vulnerability  # 风险 = 危险性 × 脆弱性

    # 6. 生成风险地图
    create_risk_map(risk, settlements, output_path='flood_risk.tif')

    return {
        'flood_extent': flood_extent_polygon,
        'exposed_population': calculate_exposed_population(exposed_settlements),
        'risk_zones': risk
    }

def estimate_flood_extent(dem_path, river_path, return_period):
    """
    使用曼宁公式和水力模型估算洪水范围
    """
    # 1. 获取河道横截面
    # 2. 计算重现期流量
    # 3. 应用曼宁公式计算水深
    # 4. 创建洪水栅格

    # 简化版：固定水位
    with rasterio.open(dem_path) as src:
        dem = src.read(1)
        profile = src.profile

    # 基于重现期的水位
    water_levels = {10: 5, 50: 8, 100: 10, 500: 12}
    water_level = water_levels.get(return_period, 10)

    # 洪水范围
    flood_extent = dem < water_level

    return flood_extent
```

### 野火风险建模

```python
def wildfire_risk_assessment(vegetation_path, dem_path, weather_data, infrastructure_path):
    """
    综合多因素的野火风险评估
    """

    # 1. 可燃物载量（来自植被）
    with rasterio.open(vegetation_path) as src:
        vegetation = src.read(1)

    # 可燃物类型: 0=无燃料, 1=低, 2=中, 3=高
    fuel_load = vegetation.map_classes({1: 0.2, 2: 0.5, 3: 0.8, 4: 1.0})

    # 2. 坡度（火势上坡蔓延更快）
    with rasterio.open(dem_path) as src:
        dem = src.read(1)

    slope = calculate_slope(dem)
    slope_factor = 1 + (slope / 90) * 0.5  # 最高增加50%

    # 3. 风力影响
    wind_speed = weather_data['wind_speed']
    wind_direction = weather_data['wind_direction']
    wind_factor = 1 + (wind_speed / 50) * 0.3

    # 4. 植被干燥度（基于NDWI异常值）
    dryness = calculate_vegetation_dryness(vegetation_path)
    dryness_factor = 1 + dryness * 0.4

    # 5. 综合因子
    risk = fuel_load * slope_factor * wind_factor * dryness_factor

    # 6. 识别高风险资产
    infrastructure = gpd.read_file(infrastructure_path)
    risk_at_infrastructure = extract_raster_values_at_points(risk, infrastructure)

    infrastructure['risk_level'] = risk_at_infrastructure
    high_risk_assets = infrastructure[infrastructure['risk_level'] > 0.7]

    return risk, high_risk_assets
```

## 公共事业与基础设施

### 电力线路廊道测绘

```python
def power_line_corridor_analysis(power_lines_path, vegetation_height_path, buffer_distance=50):
    """
    分析植被对电力线路廊道的侵占情况
    """

    # 1. 加载电力线路
    power_lines = gpd.read_file(power_lines_path)

    # 2. 创建廊道缓冲区
    corridor = power_lines.buffer(buffer_distance)

    # 3. 加载植被高度
    with rasterio.open(vegetation_height_path) as src:
        veg_height = src.read(1)
        profile = src.profile

    # 4. 提取廊道内植被高度
    veg_within_corridor = rasterio.mask.mask(veg_height, corridor.geometry, crop=True)[0]

    # 5. 识别侵占区域（植被超过安全高度）
    safe_height = 10  # 米
    encroachment = veg_within_corridor > safe_height

    # 6. 划分风险区域
    high_risk = encroachment & (veg_within_corridor > safe_height * 1.5)
    medium_risk = encroachment & ~high_risk

    # 7. 生成维护优先级地图
    priority = np.zeros_like(veg_within_corridor)
    priority[high_risk] = 3  # 紧急
    priority[medium_risk] = 2  # 监控
    priority[~encroachment] = 1  # 畅通

    # 8. 创建工单点位
    from scipy import ndimage
    labeled, num_features = ndimage.label(high_risk)

    work_orders = []
    for i in range(1, num_features + 1):
        mask = labeled == i
        centroid = ndimage.center_of_mass(mask)
        work_orders.append({
            'location': centroid,
            'area_ha': np.sum(mask) * 0.0001,  # 假设1米分辨率
            'priority': '紧急'
        })

    return priority, work_orders
```

### 管线路径优化

```python
def optimize_pipeline_route(origin, destination, constraints_path, cost_surface_path):
    """
    使用最小成本路径分析优化管线路由
    """

    # 1. 加载成本表面
    with rasterio.open(cost_surface_path) as src:
        cost = src.read(1)
        profile = src.profile

    # 2. 应用约束条件
    constraints = gpd.read_file(constraints_path)
    no_go_zones = constraints[constraints['type'] == 'no_go']

    # 为禁行区设置极高成本
    for _, zone in no_go_zones.iterrows():
        mask = rasterize_features(zone.geometry, profile['shape'])
        cost[mask > 0] = 999999

    # 3. 最小成本路径 (Dijkstra算法)
    from scipy.sparse import csr_matrix
    from scipy.sparse.csgraph import shortest_path

    # 转换为图结构 (8邻接)
    graph = create_graph_from_raster(cost)

    # 起点与终点节点
    orig_node = coord_to_node(origin, profile)
    dest_node = coord_to_node(destination, profile)

    # 寻找路径
    _, predecessors = shortest_path(csgraph=graph,
                                   directed=True,
                                   indices=orig_node,
                                   return_predecessors=True)

    # 重建路径
    path = reconstruct_path(predecessors, dest_node)

    # 4. 转换为坐标
    route_coords = [node_to_coord(node, profile) for node in path]
    route = LineString(route_coords)

    return route

def create_graph_from_raster(cost_raster):
    """从成本栅格创建最小成本路径图"""
    # 8邻接成本计算
    # 具体实现取决于库选择
    pass
```

## 交通运输

### 交通分析

```python
def traffic_analysis(roads_gdf, traffic_counts_path):
    """
    分析交通模式与拥堵情况
    """

    # 1. 加载交通流量数据
    counts = gpd.read_file(traffic_counts_path)

    # 2. 向全路网插值
    import networkx as nx

    # 创建道路网络
    G = nx.Graph()
    for _, road in roads_gdf.iterrows():
        coords = list(road.geometry.coords)
        for i in range(len(coords) - 1):
            G.add_edge(coords[i], coords[i+1],
                      length=road.geometry.length,
                      road_id=road.id)

    # 3. 流量空间插值
    from sklearn.neighbors import KNeighborsRegressor

    count_coords = np.array([[p.x, p.y] for p in counts.geometry])
    count_values = counts['AADT'].values

    knn = KNeighborsRegressor(n_neighbors=5, weights='distance')
    knn.fit(count_coords, count_values)

    # 4. 预测所有路段流量
    all_coords = np.array([[n[0], n[1]] for n in G.nodes()])
    predicted_traffic = knn.predict(all_coords)

    # 5. 识别拥堵路段
    for i, (u, v) in enumerate(G.edges()):
        avg_traffic = (predicted_traffic[list(G.nodes()).index(u)] +
                      predicted_traffic[list(G.nodes()).index(v)]) / 2
        capacity = G[u][v]['capacity']  # 需提供通行能力数据

        G[u][v]['v_c_ratio'] = avg_traffic / capacity

    # 6. 拥堵热点
    congested_edges = [(u, v) for u, v, d in G.edges(data=True)
                      if d.get('v_c_ratio', 0) > 0.9]

    return G, congested_edges
```

### 公交服务区分析

```python
def transit_service_area(stops_gdf, max_walk_distance=800, max_time=30):
    """
    考虑步行距离与出行时间的公交服务区计算
    """

    # 1. 站点周边步行可达区域
    walk_buffer = stops_gdf.buffer(max_walk_distance)

    # 2. 加载步行时间路网
    roads = gpd.read_file('roads.shp')
    G = osmnx.graph_from_gdf(roads)

    # 3. 计算各站点步行时间可达区域
    service_areas = []

    for _, stop in stops_gdf.iterrows():
        # 查找最近节点
        stop_node = ox.distance.nearest_nodes(G, stop.geometry.x, stop.geometry.y)

        # 获取步行时间可达子图
        walk_speed = 5 / 3.6  # 公里/小时转米/秒
        max_nodes = int(max_time * 60 * walk_speed / 20)  # 假设每边约20米

        subgraph = nx.ego_graph(G, stop_node, radius=max_nodes)

        # 从可达节点创建多边形
        reachable_nodes = ox.graph_to_gdfs(subgraph, edges=False)
        service_area = reachable_nodes.geometry.unary_union.convex_hull

        service_areas.append({
            'stop_id': stop.stop_id,
            'service_area': service_area,
            'area_km2': service_area.area / 1e6
        })

    return service_areas
```

更多行业专属工作流详见 [code-examples.md](code-examples.md)。
