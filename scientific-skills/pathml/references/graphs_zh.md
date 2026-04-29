# 图构建与空间分析

## 概述

PathML 提供从组织图像构建空间图的工具，用于表示细胞和组织层面的关系。基于图的表示支持复杂的空间分析，包括邻域分析、细胞间相互作用研究和图神经网络应用。这些图捕获形态特征和空间拓扑结构，用于下游计算分析。

## 图类型

PathML 支持构建多种图类型：

### 细胞图
- 节点代表单个细胞
- 边代表空间邻近性或生物相互作用
- 节点特征包括形态、标记物表达、细胞类型
- 适用于单细胞空间分析

### 组织图
- 节点代表组织区域或超像素
- 边代表空间邻接关系
- 节点特征包括组织成分、纹理特征
- 适用于组织级空间模式分析

### 空间转录组图
- 节点代表空间点位或细胞
- 边编码空间关系
- 节点特征包括基因表达谱
- 适用于空间组学分析

## 图构建工作流

### 从分割到图构建

将细胞核或细胞分割结果转换为空间图：

```python
from pathml.graph import CellGraph
from pathml.preprocessing import Pipeline, SegmentMIF
import numpy as np

# 1. 执行细胞分割
pipeline = Pipeline([
    SegmentMIF(
        nuclear_channel='DAPI',
        cytoplasm_channel='CD45',
        model='mesmer'
    )
])
pipeline.run(slide)

# 2. 提取实例分割掩码
inst_map = slide.masks['cell_segmentation']

# 3. 构建细胞图
cell_graph = CellGraph.from_instance_map(
    inst_map,
    image=slide.image,  # 可选：用于提取视觉特征
    connectivity='delaunay',  # 'knn', 'radius' 或 'delaunay'
    k=5,  # knn 的邻居数量
    radius=50  # 半径法的像素距离阈值
)

# 4. 访问图组件
nodes = cell_graph.nodes  # 节点特征
edges = cell_graph.edges  # 边列表
adjacency = cell_graph.adjacency_matrix  # 邻接矩阵
```

### 连接方法

**K近邻 (KNN):**
```python
# 连接每个细胞到其k个最近邻居
graph = CellGraph.from_instance_map(
    inst_map,
    connectivity='knn',
    k=5  # 邻居数量
)
```
- 固定节点度数
- 捕获局部邻域
- 简单可解释

**半径法:**
```python
# 在距离阈值内连接细胞
graph = CellGraph.from_instance_map(
    inst_map,
    connectivity='radius',
    radius=100,  # 像素最大距离
    distance_metric='euclidean'  # 或 'manhattan', 'chebyshev'
)
```
- 基于密度的可变度数
- 生物学驱动（相互作用范围）
- 捕获物理邻近性

**Delaunay三角剖分:**
```python
# 使用Delaunay三角剖分连接细胞
graph = CellGraph.from_instance_map(
    inst_map,
    connectivity='delaunay'
)
```
- 从空间位置创建连通图
- 无孤立节点（在凸包内）
- 捕获空间镶嵌结构

**接触法:**
```python
# 连接边界接触的细胞
graph = CellGraph.from_instance_map(
    inst_map,
    connectivity='contact',
    dilation=2  # 扩张边界以捕获近接触
)
```
- 物理细胞间接触
- 最直接的生物学方法
- 分离细胞的边稀疏

## 节点特征

### 形态特征

提取每个细胞的形状和大小特征：

```python
from pathml.graph import extract_morphology_features

# 计算形态特征
morphology_features = extract_morphology_features(
    inst_map,
    features=[
        'area',  # 像素面积
        'perimeter',  # 周长
        'eccentricity',  # 形状伸长率
        'solidity',  # 凸度测量
        'major_axis_length',
        'minor_axis_length',
        'orientation'  # 细胞方向角
    ]
)

# 添加到图
cell_graph.add_node_features(morphology_features, feature_names=['area', 'perimeter', ...])
```

**可用形态特征:**
- **面积** - 像素数量
- **周长** - 边界长度
- **离心率** - 0（圆形）到1（线形）
- **凸度** - 面积 / 凸包面积
- **圆度** - 4π × 面积 / 周长²
- **长/短轴** - 拟合椭圆轴长度
- **方向** - 长轴角度
- **范围** - 面积 / 边界框面积

### 强度特征

提取标记物表达或强度统计值：

```python
from pathml.graph import extract_intensity_features

# 提取每个细胞的平均标记强度
intensity_features = extract_intensity_features(
    inst_map,
    image=multichannel_image,  # 形状: (H, W, C)
    channel_names=['DAPI', 'CD3', 'CD4', 'CD8', 'CD20'],
    statistics=['mean', 'std', 'median', 'max']
)

# 添加到图
cell_graph.add_node_features(
    intensity_features,
    feature_names=['DAPI_mean', 'CD3_mean', ...]
)
```

**可用统计量:**
- **平均值** - 平均强度
- **中位数** - 中位强度
- **标准差** - 标准偏差
- **最大值** - 最大强度
- **最小值** - 最小强度
- **分位数_25/75** - 四分位数

### 纹理特征

计算每个细胞区域的纹理描述符：

```python
from pathml.graph import extract_texture_features

# Haralick纹理特征
texture_features = extract_texture_features(
    inst_map,
    image=grayscale_image,
    features='haralick',  # 或 'lbp', 'gabor'
    distance=1,
    angles=[0, np.pi/4, np.pi/2, 3*np.pi/4]
)

cell_graph.add_node_features(texture_features)
```

### 细胞类型标注

添加分类得到的细胞类型标签：

```python
# 来自ML模型预测
cell_types = hovernet_type_predictions  # 细胞类型ID数组

cell_graph.add_node_features(
    cell_types,
    feature_names=['cell_type']
)

# 细胞类型独热编码
cell_type_onehot = one_hot_encode(cell_types, num_classes=5)
cell_graph.add_node_features(
    cell_type_onehot,
    feature_names=['type_epithelial', 'type_inflammatory', ...]
)
```

## 边特征

### 空间距离

基于空间关系计算边特征：

```python
from pathml.graph import compute_edge_distances

# 添加成对距离作为边特征
distances = compute_edge_distances(
    cell_graph,
    metric='euclidean'  # 或 'manhattan', 'chebyshev'
)

cell_graph.add_edge_features(distances, feature_names=['distance'])
```

### 相互作用特征

建模细胞类型间的生物相互作用：

```python
from pathml.graph import compute_interaction_features

# 沿边的细胞类型共现特征
interaction_features = compute_interaction_features(
    cell_graph,
    cell_types=cell_type_labels,
    interaction_type='categorical'  # 或 'numerical'
)

cell_graph.add_edge_features(interaction_features)
```

## 图级特征

聚合整个图的特征：

```python
from pathml.graph import compute_graph_features

# 拓扑特征
graph_features = compute_graph_features(
    cell_graph,
    features=[
        'num_nodes',
        'num_edges',
        'average_degree',
        'clustering_coefficient',
        'average_path_length',
        'diameter'
    ]
)

# 细胞组成特征
composition = cell_graph.compute_cell_type_composition(
    cell_type_labels,
    normalize=True  # 比例形式
)
```

## 空间分析

### 邻域分析

分析细胞邻域和微环境：

```python
from pathml.graph import analyze_neighborhoods

# 表征每个细胞周围的邻域
neighborhoods = analyze_neighborhoods(
    cell_graph,
    cell_types=cell_type_labels,
    radius=100,  # 邻域半径
    metrics=['diversity', 'density', 'composition']
)

# 邻域多样性（香农熵）
diversity = neighborhoods['diversity']

# 每个邻域的细胞类型组成
composition = neighborhoods['composition']  # (n_cells, n_cell_types)
```

### 空间聚类

识别细胞类型的空间簇：

```python
from pathml.graph import spatial_clustering
import matplotlib.pyplot as plt

# 检测空间簇
clusters = spatial_clustering(
    cell_graph,
    cell_positions,
    method='dbscan',  # 或 'kmeans', 'hierarchical'
    eps=50,  # DBSCAN邻域半径
    min_samples=10  # DBSCAN最小簇大小
)

# 可视化簇
plt.scatter(
    cell_positions[:, 0],
    cell_positions[:, 1],
    c=clusters,
    cmap='tab20'
)
plt.title('空间聚类')
plt.show()
```

### 细胞间相互作用分析

检测细胞类型相互作用的富集或缺失：

```python
from pathml.graph import cell_interaction_analysis

# 检验显著相互作用
interaction_results = cell_interaction_analysis(
    cell_graph,
    cell_types=cell_type_labels,
    method='permutation',  # 或 'expected'
    n_permutations=1000,
    significance_level=0.05
)

# 相互作用分数（正值=吸引，负值=排斥）
interaction_matrix = interaction_results['scores']

# 热力图可视化
import seaborn as sns
sns.heatmap(
    interaction_matrix,
    cmap='RdBu_r',
    center=0,
    xticklabels=cell_type_names,
    yticklabels=cell_type_names
)
plt.title('细胞间相互作用分数')
plt.show()
```

### 空间统计

计算空间统计量和模式：

```python
from pathml.graph import spatial_statistics

# 空间点模式的Ripley's K函数
ripleys_k = spatial_statistics(
    cell_positions,
    cell_types=cell_type_labels,
    statistic='ripleys_k',
    radii=np.linspace(0, 200, 50)
)

# 最近邻距离
nn_distances = spatial_statistics(
    cell_positions,
    statistic='nearest_neighbor',
    by_cell_type=True
)
```

## 图神经网络集成

### 转换为PyTorch Geometric格式

```python
from pathml.graph import to_pyg
import torch
from torch_geometric.data import Data

# 转换为PyTorch Geometric数据对象
pyg_data = cell_graph.to_pyg()

# 访问组件
x = pyg_data.x  # 节点特征 (n_nodes, n_features)
edge_index = pyg_data.edge_index  # 边连接 (2, n_edges)
edge_attr = pyg_data.edge_attr  # 边特征 (n_edges, n_edge_features)
y = pyg_data.y  # 图级标签
pos = pyg_data.pos  # 节点位置 (n_nodes, 2)

# 与PyTorch Geometric集成
from torch_geometric.nn import GCNConv

class GNN(torch.nn.Module):
    def __init__(self, in_channels, hidden_channels, out_channels):
        super().__init__()
        self.conv1 = GCNConv(in_channels, hidden_channels)
        self.conv2 = GCNConv(hidden_channels, out_channels)

    def forward(self, data):
        x, edge_index = data.x, data.edge_index
        x = self.conv1(x, edge_index).relu()
        x = self.conv2(x, edge_index)
        return x

model = GNN(in_channels=pyg_data.num_features, hidden_channels=64, out_channels=5)
output = model(pyg_data)
```

### 多切片图数据集

```python
from pathml.graph import GraphDataset
from torch_geometric.loader import DataLoader

# 创建多切片图数据集
graphs = []
for slide in slides:
    # 为每个切片构建图
    cell_graph = CellGraph.from_instance_map(slide.inst_map, ...)
    pyg_graph = cell_graph.to_pyg()
    graphs.append(pyg_graph)

# 创建DataLoader
loader = DataLoader(graphs, batch_size=32, shuffle=True)

# 训练GNN
for batch in loader:
    output = model(batch)
    loss = criterion(output, batch.y)
    loss.backward()
    optimizer.step()
```

## 可视化

### 图可视化

```python
import matplotlib.pyplot as plt
import networkx as nx

# 转换为NetworkX图
nx_graph = cell_graph.to_networkx()

# 使用细胞位置作为布局绘图
pos = {i: cell_graph.positions[i] for i in range(len(cell_graph.nodes))}

plt.figure(figsize=(12, 12))
nx.draw_networkx(
    nx_graph,
    pos=pos,
    node_color=cell_type_labels,
    node_size=50,
    cmap='tab10',
    with_labels=False,
    alpha=0.8
)
plt.axis('equal')
plt.title('细胞图')
plt.show()
```

### 组织图像叠加

```python
from pathml.graph import visualize_graph_on_image

# 在组织图像上叠加可视化图
fig, ax = plt.subplots(figsize=(15, 15))
ax.imshow(tissue_image)

# 绘制边
for edge in cell_graph.edges:
    node1, node2 = edge
    pos1 = cell_graph.positions[node1]
    pos2 = cell_graph.positions[node2]
    ax.plot([pos1[0], pos2[0]], [pos1[1], pos2[1]], 'b-', alpha=0.3, linewidth=0.5)

# 按类型着色绘制节点
for cell_type in np.unique(cell_type_labels):
    mask = cell_type_labels == cell_type
    positions = cell_graph.positions[mask]
    ax.scatter(positions[:, 0], positions[:, 1], label=f'类型 {cell_type}', s=20)

ax.legend()
ax.axis('off')
plt.title('组织上的细胞图')
plt.show()
```

## 完整工作流示例

```python
from pathml.core import SlideData, CODEXSlide
from pathml.preprocessing import Pipeline, CollapseRunsCODEX, SegmentMIF
from pathml.graph import CellGraph, extract_morphology_features, extract_intensity_features
import matplotlib.pyplot as plt

# 1. 加载并预处理切片
slide = CODEXSlide('path/to/codex', stain='IF')

pipeline = Pipeline([
    CollapseRunsCODEX(z_slice=2),
    SegmentMIF(
        nuclear_channel='DAPI',
        cytoplasm_channel='CD45',
        model='mesmer'
    )
])
pipeline.run(slide)

# 2. 构建细胞图
inst_map = slide.masks['cell_segmentation']
cell_graph = CellGraph.from_instance_map(
    inst_map,
    image=slide.image,
    connectivity='knn',
    k=6
)

# 3. 提取特征
# 形态特征
morph_features = extract_morphology_features(
    inst_map,
    features=['area', 'perimeter', 'eccentricity', 'solidity']
)
cell_graph.add_node_features(morph_features)

# 强度特征（标记物表达）
intensity_features = extract_intensity_features(
    inst_map,
    image=slide.image,
    channel_names=['DAPI', 'CD3', 'CD4', 'CD8', 'CD20'],
    statistics=['mean', 'std']
)
cell_graph.add_node_features(intensity_features)

# 4. 空间分析
from pathml.graph import analyze_neighborhoods

neighborhoods = analyze_neighborhoods(
    cell_graph,
    cell_types=cell_type_predictions,
    radius=100,
    metrics=['diversity', 'composition']
)

# 5. 导出到GNN
pyg_data = cell_graph.to_pyg()

# 6. 可视化
plt.figure(figsize=(15, 15))
plt.imshow(slide.image)

# 叠加图
nx_graph = cell_graph.to_networkx()
pos = {i: cell_graph.positions[i] for i in range(cell_graph.num_nodes)}
nx.draw_networkx(
    nx_graph,
    pos=pos,
    node_color=cell_type_predictions,
    cmap='tab10',
    node_size=30,
    with_labels=False
)
plt.axis('off')
plt.title('带空间邻域的细胞图')
plt.show()
```

## 性能考量

**大组织切片:**
- 分块构建图后合并
- 使用稀疏邻接矩阵
- 利用GPU加速特征提取

**内存效率:**
- 仅存储必要的边特征
- 使用int32/float32替代int64/float64
- 批量处理多个切片

**计算效率:**
- 跨细胞并行化特征提取
- 使用KNN加速邻居查询
- 缓存已计算特征

## 最佳实践

1. **选择合适的连通性：** 均匀分析用KNN，物理相互作用用半径法，细胞间直接通讯用接触法  
2. **特征归一化：** 缩放形态和强度特征以适应GNN处理  
3. **处理边界效应：** 排除边界细胞或使用组织掩膜定义有效区域  
4. **验证图构建：** 大规模处理前在小区域可视化图结构  
5. **融合多类型特征：** 形态+强度+纹理特征可提供丰富表征  
6. **考虑组织背景：** 组织类型影响图参数选择（连通性、半径）  

## 常见问题与解决方案  

**问题：边过多/过少**  
- 调整k值（KNN）或半径参数（基于半径）  
- 验证像素到微米的转换是否符合生物学意义  

**问题：大图内存错误**  
- 分块处理并合并图结构  
- 使用稀疏矩阵表示  
- 仅保留必要的边特征  

**问题：组织边界细胞缺失**  
- 应用edge_correction参数  
- 使用组织掩膜排除无效区域  

**问题：特征尺度不一致**  
- 特征归一化：`(x - 均值) / 标准差`  
- 对离群值使用鲁棒缩放  

## 扩展资源  

- **PathML图API：** https://pathml.readthedocs.io/en/latest/api_graph_reference.html  
- **PyTorch Geometric：** https://pytorch-geometric.readthedocs.io/  
- **NetworkX：** https://networkx.org/  
- **空间统计学：** Baddeley et al., 《空间点模式：R语言方法与应用》(Spatial Point Patterns: Methodology and Applications with R)
