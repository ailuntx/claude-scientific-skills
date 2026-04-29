# NetworkX 图可视化

## 使用 Matplotlib 进行基本绘图

### 简单可视化
```python
import networkx as nx
import matplotlib.pyplot as plt

# 创建并绘制图形
G = nx.karate_club_graph()
nx.draw(G)
plt.show()

# 保存到文件
nx.draw(G)
plt.savefig('graph.png', dpi=300, bbox_inches='tight')
plt.close()
```

### 带标签的绘图
```python
# 绘制带节点标签
nx.draw(G, with_labels=True)
plt.show()

# 自定义标签
labels = {i: f"节点 {i}" for i in G.nodes()}
nx.draw(G, labels=labels, with_labels=True)
plt.show()
```

## 布局算法

### 弹簧布局（力导向）
```python
# Fruchterman-Reingold 力导向算法
pos = nx.spring_layout(G, seed=42)
nx.draw(G, pos=pos, with_labels=True)
plt.show()

# 带参数配置
pos = nx.spring_layout(G, k=0.5, iterations=50, seed=42)
```

### 环形布局
```python
# 节点环形排列
pos = nx.circular_layout(G)
nx.draw(G, pos=pos, with_labels=True)
plt.show()
```

### 随机布局
```python
# 随机定位
pos = nx.random_layout(G, seed=42)
nx.draw(G, pos=pos, with_labels=True)
plt.show()
```

### 壳层布局
```python
# 同心圆排列
pos = nx.shell_layout(G)
nx.draw(G, pos=pos, with_labels=True)
plt.show()

# 自定义壳层
shells = [[0, 1, 2], [3, 4, 5, 6], [7, 8, 9]]
pos = nx.shell_layout(G, nlist=shells)
```

### 谱布局
```python
# 使用图拉普拉斯矩阵的特征向量
pos = nx.spectral_layout(G)
nx.draw(G, pos=pos, with_labels=True)
plt.show()
```

### Kamada-Kawai 布局
```python
# 基于能量的布局
pos = nx.kamada_kawai_layout(G)
nx.draw(G, pos=pos, with_labels=True)
plt.show()
```

### 平面布局
```python
# 仅适用于平面图
if nx.is_planar(G):
    pos = nx.planar_layout(G)
    nx.draw(G, pos=pos, with_labels=True)
    plt.show()
```

### 树状布局
```python
# 适用于树状图
if nx.is_tree(G):
    pos = nx.nx_agraph.graphviz_layout(G, prog='dot')
    nx.draw(G, pos=pos, with_labels=True)
    plt.show()
```

## 自定义节点外观

### 节点颜色
```python
# 单一颜色
nx.draw(G, node_color='red')

# 按节点分配不同颜色
node_colors = ['red' if G.degree(n) > 5 else 'blue' for n in G.nodes()]
nx.draw(G, node_color=node_colors)

# 按属性着色
colors = [G.nodes[n].get('value', 0) for n in G.nodes()]
nx.draw(G, node_color=colors, cmap=plt.cm.viridis)
plt.colorbar()
plt.show()
```

### 节点尺寸
```python
# 按度值设置尺寸
node_sizes = [100 * G.degree(n) for n in G.nodes()]
nx.draw(G, node_size=node_sizes)

# 按中心性设置尺寸
centrality = nx.degree_centrality(G)
node_sizes = [3000 * centrality[n] for n in G.nodes()]
nx.draw(G, node_size=node_sizes)
```

### 节点形状
```python
# 分别绘制不同形状的节点
pos = nx.spring_layout(G)

# 圆形节点
nx.draw_networkx_nodes(G, pos, nodelist=[0, 1, 2],
                       node_shape='o', node_color='red')

# 方形节点
nx.draw_networkx_nodes(G, pos, nodelist=[3, 4, 5],
                       node_shape='s', node_color='blue')

nx.draw_networkx_edges(G, pos)
nx.draw_networkx_labels(G, pos)
plt.show()
```

### 节点边框
```python
nx.draw(G, pos=pos,
        node_color='lightblue',
        edgecolors='black',  # 节点边框颜色
        linewidths=2)        # 节点边框宽度
plt.show()
```

## 自定义边外观

### 边颜色
```python
# 单一颜色
nx.draw(G, edge_color='gray')

# 按边分配不同颜色
edge_colors = ['red' if G[u][v].get('weight', 1) > 0.5 else 'blue'
               for u, v in G.edges()]
nx.draw(G, edge_color=edge_colors)

# 按权重着色
edges = G.edges()
weights = [G[u][v].get('weight', 1) for u, v in edges]
nx.draw(G, edge_color=weights, edge_cmap=plt.cm.Reds)
```

### 边宽度
```python
# 按权重设置宽度
edge_widths = [3 * G[u][v].get('weight', 1) for u, v in G.edges()]
nx.draw(G, width=edge_widths)

# 按介数中心性设置宽度
edge_betweenness = nx.edge_betweenness_centrality(G)
edge_widths = [5 * edge_betweenness[(u, v)] for u, v in G.edges()]
nx.draw(G, width=edge_widths)
```

### 边样式
```python
# 虚线边
nx.draw(G, style='dashed')

# 按边分配不同样式
pos = nx.spring_layout(G)
strong_edges = [(u, v) for u, v in G.edges() if G[u][v].get('weight', 0) > 0.5]
weak_edges = [(u, v) for u, v in G.edges() if G[u][v].get('weight', 0) <= 0.5]

nx.draw_networkx_nodes(G, pos)
nx.draw_networkx_edges(G, pos, edgelist=strong_edges, style='solid', width=2)
nx.draw_networkx_edges(G, pos, edgelist=weak_edges, style='dashed', width=1)
plt.show()
```

### 有向图（箭头）
```python
# 绘制带箭头的有向图
G_directed = nx.DiGraph([(1, 2), (2, 3), (3, 1)])
pos = nx.spring_layout(G_directed)

nx.draw(G_directed, pos=pos, with_labels=True,
        arrows=True,
        arrowsize=20,
        arrowstyle='->',
        connectionstyle='arc3,rad=0.1')
plt.show()
```

## 标签与标注

### 节点标签
```python
pos = nx.spring_layout(G)

# 自定义标签
labels = {n: f"N{n}" for n in G.nodes()}
nx.draw_networkx_labels(G, pos, labels=labels, font_size=12, font_color='white')

# 字体定制
nx.draw_networkx_labels(G, pos,
                       font_size=10,
                       font_family='serif',
                       font_weight='bold')
```

### 边标签
```python
pos = nx.spring_layout(G)
nx.draw_networkx_nodes(G, pos)
nx.draw_networkx_edges(G, pos)

# 从属性获取边标签
edge_labels = nx.get_edge_attributes(G, 'weight')
nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels)
plt.show()

# 自定义边标签
edge_labels = {(u, v): f"{u}-{v}" for u, v in G.edges()}
nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels)
```

## 高级绘图技巧

### 组合绘图函数
```python
# 通过分离组件实现完全控制
pos = nx.spring_layout(G, seed=42)

# 绘制边
nx.draw_networkx_edges(G, pos, alpha=0.3, width=1)

# 绘制节点
nx.draw_networkx_nodes(G, pos,
                       node_color='lightblue',
                       node_size=500,
                       edgecolors='black')

# 绘制标签
nx.draw_networkx_labels(G, pos, font_size=10)

# 移除坐标轴
plt.axis('off')
plt.tight_layout()
plt.show()
```

### 子图高亮
```python
pos = nx.spring_layout(G)

# 标识要高亮的子图
subgraph_nodes = [1, 2, 3, 4]
subgraph = G.subgraph(subgraph_nodes)

# 绘制主图
nx.draw_networkx_nodes(G, pos, node_color='lightgray', node_size=300)
nx.draw_networkx_edges(G, pos, alpha=0.2)

# 高亮子图
nx.draw_networkx_nodes(subgraph, pos, node_color='red', node_size=500)
nx.draw_networkx_edges(subgraph, pos, edge_color='red', width=2)

nx.draw_networkx_labels(G, pos)
plt.axis('off')
plt.show()
```

### 社区着色
```python
from networkx.algorithms import community

# 检测社区
communities = community.greedy_modularity_communities(G)

# 分配颜色
color_map = {}
colors = ['red', 'blue', 'green', 'yellow', 'purple', 'orange']
for i, comm in enumerate(communities):
    for node in comm:
        color_map[node] = colors[i % len(colors)]

node_colors = [color_map[n] for n in G.nodes()]

pos = nx.spring_layout(G)
nx.draw(G, pos=pos, node_color=node_colors, with_labels=True)
plt.show()
```

## 创建出版级图表

### 高分辨率导出
```python
plt.figure(figsize=(12, 8))
pos = nx.spring_layout(G, seed=42)

nx.draw(G, pos=pos,
        node_color='lightblue',
        node_size=500,
        edge_color='gray',
        width=1,
        with_labels=True,
        font_size=10)

plt.title('图可视化', fontsize=16)
plt.axis('off')
plt.tight_layout()
plt.savefig('publication_graph.png', dpi=300, bbox_inches='tight')
plt.savefig('publication_graph.pdf', bbox_inches='tight')  # 矢量格式
plt.close()
```

### 多面板图表
```python
fig, axes = plt.subplots(1, 3, figsize=(18, 6))

# 不同布局
layouts = [nx.circular_layout(G), nx.spring_layout(G), nx.spectral_layout(G)]
titles = ['环形布局', '弹簧布局', '谱布局']

for ax, pos, title in zip(axes, layouts, titles):
    nx.draw(G, pos=pos, ax=ax, with_labels=True, node_color='lightblue')
    ax.set_title(title)
    ax.axis('off')

plt.tight_layout()
plt.savefig('layouts_comparison.png', dpi=300)
plt.close()
```

## 交互式可视化库

### Plotly（交互式）
```python
import plotly.graph_objects as go

# 创建位置
pos = nx.spring_layout(G)

# 边轨迹
edge_x = []
edge_y = []
for edge in G.edges():
    x0, y0 = pos[edge[0]]
    x1, y1 = pos[edge[1]]
    edge_x.extend([x0, x1, None])
    edge_y.extend([y0, y1, None])

edge_trace = go.Scatter(
    x=edge_x, y=edge_y,
    line=dict(width=0.5, color='#888'),
    hoverinfo='none',
    mode='lines')

# 节点轨迹
node_x = [pos[node][0] for node in G.nodes()]
node_y = [pos[node][1] for node in G.nodes()]

node_trace = go.Scatter(
    x=node_x, y=node_y,
    mode='markers',
    hoverinfo='text',
    marker=dict(
        showscale=True,
        colorscale='YlGnBu',
        size=10,
        colorbar=dict(thickness=15, title='节点连接数'),
        line_width=2))

# 按度值着色
node_adjacencies = [len(list(G.neighbors(node))) for node in G.nodes()]
node_trace.marker.color = node_adjacencies

fig = go.Figure(data=[edge_trace, node_trace],
                layout=go.Layout(
                    showlegend=False,
                    hovermode='closest',
                    margin=dict(b=0, l=0, r=0, t=0)))

fig.show()
```

### PyVis（交互式 HTML）
```python
from pyvis.network import Network

# 创建网络
net = Network(notebook=True, height='750px', width='100%')

# 从 NetworkX 添加节点和边
net.from_nx(G)

# 自定义
net.show_buttons(filter_=['physics'])

# 保存
net.show('graph.html')
```

### Graphviz（通过 pydot）
```python
# 需要 graphviz 和 pydot
from networkx.drawing.nx_pydot import graphviz_layout

pos = graphviz_layout(G, prog='neato')  # neato, dot, fdp, sfdp, circo, twopi
nx.draw(G, pos=pos, with_labels=True)
plt.show()

# 导出到 graphviz
nx.drawing.nx_pydot.write_dot(G, 'graph.dot')
```

## 二分图可视化

### 双集合布局
```python
from networkx.algorithms import bipartite

# 创建二分图
B = nx.Graph()
B.add_nodes_from([1, 2, 3, 4], bipartite=0)
B.add_nodes_from(['a', 'b', 'c', 'd', 'e'], bipartite=1)
B.add_edges_from([(1, 'a'), (1, 'b'), (2, 'b'), (2, 'c'), (3, 'd'), (4, 'e')])

# 双列布局
pos = {}
top_nodes = [n for n, d in B.nodes(data=True) if d['bipartite'] == 0]
bottom_nodes = [n for n, d in B.nodes(data=True) if d['bipartite'] == 1]

pos.update({node: (0, i) for i, node in enumerate(top_nodes)})
pos.update({node: (1, i) for i, node in enumerate(bottom_nodes)})

nx.draw(B, pos=pos, with_labels=True,
        node_color=['lightblue' if B.nodes[n]['bipartite'] == 0 else 'lightgreen'
                   for n in B.nodes()])
plt.show()
```

## 3D 可视化

### 3D 网络图
```python
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

# 3D 弹簧布局
pos = nx.spring_layout(G, dim=3, seed=42)

# 提取坐标
node_xyz = np.array([pos[v] for v in G.nodes()])
edge_xyz = np.array([(pos[u], pos[v]) for u, v in G.edges()])

# 创建图表
fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')

# 绘制边
for vizedge in edge_xyz:
    ax.plot(*vizedge.T, color='gray', alpha=0.5)

# 绘制节点
ax.scatter(*node_xyz.T, s=100, c='lightblue', edgecolors='black')

# 标签
for i, (x, y, z) in enumerate(node_xyz):
    ax.text(x, y, z, str(i))

ax.set_axis_off()
plt.show()
```

## 最佳实践

### 性能优化
- 大型图（>1000 节点）使用简单布局（环形、随机）
- 使用 `alpha` 参数增强密集边的可见性
- 对超大规模网络考虑降采样或显示子图

### 视觉美学
- 使用一致的配色方案
- 按意义缩放节点尺寸（如按度值或重要性）
- 保持标签可读性（调整字体大小和位置）
- 有效利用留白（调整图表尺寸）

### 可复现性
- 布局始终设置随机种子：`nx.spring_layout(G, seed=42)`
- 保存布局位置确保多图
