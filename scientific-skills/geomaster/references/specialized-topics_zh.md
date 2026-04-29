# 专题研究

高级专题：地统计学、优化、伦理与最佳实践。

## 地统计学

### 变异函数分析

```python
import numpy as np
from scipy.spatial.distance import pdist, squareform
import matplotlib.pyplot as plt

def empirical_variogram(points, values, max_lag=None, n_lags=15):
    """
    计算经验变异函数。
    """
    n = len(points)

    # 距离矩阵
    dist_matrix = squareform(pdist(points))

    if max_lag is None:
        max_lag = np.max(dist_matrix) / 2

    # 计算半方差
    semivariance = []
    mean_distances = []

    for lag in np.linspace(0, max_lag, n_lags):
        # 点对选择
        mask = (dist_matrix >= lag) & (dist_matrix < lag + max_lag/n_lags)

        if np.sum(mask) == 0:
            continue

        # 半方差公式: (1/2n) * sum(z_i - z_j)^2
        diff_squared = (values[:, None] - values) ** 2
        gamma = 0.5 * np.mean(diff_squared[mask])

        semivariance.append(gamma)
        mean_distances.append(lag + max_lag/(2*n_lags))

    return np.array(mean_distances), np.array(semivariance)

# 拟合变异函数模型
def fit_variogram_model(lags, gammas, model='spherical'):
    """
    拟合理论变异函数模型。
    """
    from scipy.optimize import curve_fit

    def spherical(h, nugget, sill, range_):
        """球状模型"""
        h = np.asarray(h)
        gamma = np.where(h < range_,
                        nugget + sill * (1.5 * h/range_ - 0.5 * (h/range_)**3),
                        nugget + sill)
        return gamma

    def exponential(h, nugget, sill, range_):
        """指数模型"""
        return nugget + sill * (1 - np.exp(-3 * h / range_))

    def gaussian(h, nugget, sill, range_):
        """高斯模型"""
        return nugget + sill * (1 - np.exp(-3 * (h/range_)**2))

    models = {
        'spherical': spherical,
        'exponential': exponential,
        'gaussian': gaussian
    }

    # 模型拟合
    popt, _ = curve_fit(models[model], lags, gammas,
                        p0=[np.min(gammas), np.max(gammas), np.max(lags)/2],
                        bounds=(0, np.inf))

    return popt, models[model]
```

### 克里金插值法

```python
from pykrige.ok import OrdinaryKriging
import numpy as np

def ordinary_kriging(x, y, z, grid_resolution=100):
    """
    执行普通克里金插值。
    """
    # 创建网格
    gridx = np.linspace(x.min(), x.max(), grid_resolution)
    gridy = np.linspace(y.min(), y.max(), grid_resolution)

    # 拟合变异函数
    OK = OrdinaryKriging(
        x, y, z,
        variogram_model='spherical',
        verbose=False,
        enable_plotting=False,
        coordinates_type='euclidean',
    )

    # 插值计算
    zinterp, sigmasq = OK.execute('grid', gridx, gridy)

    return zinterp, sigmasq, gridx, gridy

# 交叉验证
def kriging_cross_validation(x, y, z, n_folds=5):
    """
    执行克里金法的k折交叉验证。
    """
    from sklearn.model_selection import KFold

    kf = KFold(n_splits=n_folds)
    errors = []

    for train_idx, test_idx in kf.split(z):
        # 训练
        OK = OrdinaryKriging(
            x[train_idx], y[train_idx], z[train_idx],
            variogram_model='spherical',
            verbose=False
        )

        # 在测试点预测
        predictions, _ = OK.execute('points',
                                    x[test_idx], y[test_idx])

        # 计算误差
        rmse = np.sqrt(np.mean((predictions - z[test_idx])**2))
        errors.append(rmse)

    return np.mean(errors), np.std(errors)
```

## 空间优化

### 位置分配问题

```python
from scipy.optimize import minimize
import numpy as np

def facility_location(demand_points, n_facilities=5):
    """
    解决p中位数设施选址问题。
    """

    n_demand = len(demand_points)

    # 距离矩阵
    dist_matrix = np.zeros((n_demand, n_demand))
    for i, p1 in enumerate(demand_points):
        for j, p2 in enumerate(demand_points):
            dist_matrix[i, j] = np.sqrt((p1[0]-p2[0])**2 + (p1[1]-p2[1])**2)

    # 决策变量：哪些需求点设置设施
    def objective(x):
        """最小化总加权距离"""
        # x是设施位置的二进制数组
        facility_indices = np.where(x > 0.5)[0]

        # 分配每个需求点到最近设施
        total_distance = 0
        for i in range(n_demand):
            min_dist = np.min([dist_matrix[i, f] for f in facility_indices])
            total_distance += min_dist

        return total_distance

    # 约束：恰好n_facilities个设施
    constraints = {'type': 'eq', 'fun': lambda x: np.sum(x) - n_facilities}

    # 边界：二进制
    bounds = [(0, 1)] * n_demand

    # 初始猜测：随机位置
    x0 = np.zeros(n_demand)
    x0[:n_facilities] = 1

    # 求解
    result = minimize(
        objective, x0,
        method='SLSQP',
        bounds=bounds,
        constraints=constraints
    )

    facility_indices = np.where(result.x > 0.5)[0]
    return demand_points[facility_indices]
```

### 路径优化

```python
import networkx as nx

def traveling_salesman(G, start_node):
    """
    使用启发式方法解决旅行商问题。
    """
    unvisited = set(G.nodes())
    unvisited.remove(start_node)

    route = [start_node]
    current = start_node

    while unvisited:
        # 寻找最近的未访问节点
        nearest = min(unvisited,
                     key=lambda n: G[current][n].get('weight', 1))
        route.append(nearest)
        unvisited.remove(nearest)
        current = nearest

    # 返回起点
    route.append(start_node)

    return route

# 车辆路径问题
def vehicle_routing(G, depot, customers, n_vehicles=3, capacity=100):
    """
    使用启发式方法解决车辆路径问题（先聚类后规划）。
    """
    from sklearn.cluster import KMeans

    # 1. 客户聚类
    coords = np.array([[G.nodes[n]['x'], G.nodes[n]['y']] for n in customers])
    kmeans = KMeans(n_clusters=n_vehicles, random_state=42)
    labels = kmeans.fit_predict(coords)

    # 2. 为每个聚类规划路径
    routes = []
    for i in range(n_vehicles):
        cluster_customers = [customers[j] for j in range(len(customers)) if labels[j] == i]
        route = traveling_salesman(G.subgraph(cluster_customers + [depot]), depot)
        routes.append(route)

    return routes
```

## 伦理与隐私

### 保护隐私的地理空间分析

```python
# 空间数据的差分隐私
def add_dp_noise(locations, epsilon=1.0, radius=100):
    """
    为位置数据添加差分隐私噪声。
    """
    import numpy as np

    noisy_locations = []
    for lon, lat in locations:
        # 计算噪声（拉普拉斯机制）
        sensitivity = radius
        scale = sensitivity / epsilon

        noise_lon = np.random.laplace(0, scale)
        noise_lat = np.random.laplace(0, scale)

        noisy_locations.append((lon + noise_lon, lat + noise_lat))

    return noisy_locations

# 轨迹数据的k-匿名化
def k_anonymize_trajectory(trajectory, k=5):
    """
    对轨迹应用k-匿名化。
    """
    # 1. 分割为子段
    # 2. 寻找k-1条相似轨迹
    # 3. 使用泛化替换子段

    # 简化版：空间泛化
    from shapely.geometry import LineString

    simplified = LineString(trajectory).simplify(0.01)
    return list(simplified.coords)
```

### 数据溯源

```python
# 追踪地理空间数据谱系
class DataLineage:
    def __init__(self):
        self.history = []

    def record_transformation(self, input_data, operation, output_data, params):
        """记录数据转换过程"""
        record = {
            'timestamp': pd.Timestamp.now(),
            'input': input_data,
            'operation': operation,
            'output': output_data,
            'parameters': params
        }
        self.history.append(record)

    def get_lineage(self, data_id):
        """获取数据集的完整谱系"""
        lineage = []
        for record in reversed(self.history):
            if record['output'] == data_id:
                lineage.append(record)
                lineage.extend(self.get_lineage(record['input']))
        return lineage
```

## 最佳实践

### 可重复性研究

```python
# 使用environment.yml管理依赖
# environment.yml:
"""
name: geomaster
dependencies:
  - python=3.11
  - geopandas
  - rasterio
  - scikit-learn
  - pip
  - pip:
    - torchgeo
"""

# 捕获环境信息
def capture_environment():
    """记录软件和数据版本信息"""
    import platform
    import geopandas as gpd
    import rasterio
    import numpy as np
    import pandas as pd

    info = {
        'os': platform.platform(),
        'python': platform.python_version(),
        'geopandas': gpd.__version__,
        'rasterio': rasterio.__version__,
        'numpy': np.__version__,
        'pandas': pd.__version__,
        'timestamp': pd.Timestamp.now()
    }

    return info

# 保存环境信息
import json
with open('processing_info.json', 'w') as f:
    json.dump(capture_environment(), f, indent=2, default=str)
```

### 代码组织

```python
# 项目结构
"""
project/
├── data/
│   ├── raw/
│   ├── processed/
│   └── external/
├── notebooks/
├── src/
│   ├── __init__.py
│   ├── data_loading.py
│   ├── preprocessing.py
│   ├── analysis.py
│   └── visualization.py
├── tests/
├── config.yaml
└── README.md
"""

# 配置管理
import yaml

with open('config.yaml') as f:
    config = yaml.safe_load(f)

# 访问参数
crs = config['projection']['output_crs']
resolution = config['data']['resolution']
```

### 性能优化

```python
# 内存分析
import memory_profiler

@memory_profiler.profile
def process_large_dataset(data_path):
    """分析内存使用情况"""
    data = load_data(data_path)
    result = process(data)
    return result

# 向量化 vs 循环
# 差: 逐行迭代
for idx, row in gdf.iterrows():
    gdf.loc[idx, 'buffer'] = row.geometry.buffer(100)

# 优: 向量化操作
gdf['buffer'] = gdf.geometry.buffer(100)

# 分块处理
def process_in_chunks(gdf, func, chunk_size=1000):
    """分块处理地理数据框"""
    results = []
    for i in range(0, len(gdf), chunk_size):
        chunk = gdf.iloc[i:i+chunk_size]
        result = func(chunk)
        results.append(result)
    return pd.concat(results)
```

更多代码示例请参见[code-examples.md](code-examples.md)。
