# NetworkX 输入/输出

## 从文件读取图

### 邻接表格式
```python
# 读取邻接表（简单文本格式）
G = nx.read_adjlist('graph.adjlist')

# 带节点类型转换
G = nx.read_adjlist('graph.adjlist', nodetype=int)

# 用于有向图
G = nx.read_adjlist('graph.adjlist', create_using=nx.DiGraph())

# 写入邻接表
nx.write_adjlist(G, 'graph.adjlist')
```

邻接表示例格式：
```
# 节点 邻居
0 1 2
1 0 3 4
2 0 3
3 1 2 4
4 1 3
```

### 边列表格式
```python
# 读取边列表
G = nx.read_edgelist('graph.edgelist')

# 带节点类型和边数据
G = nx.read_edgelist('graph.edgelist',
                     nodetype=int,
                     data=(('weight', float),))

# 读取带权边列表
G = nx.read_weighted_edgelist('weighted.edgelist')

# 写入边列表
nx.write_edgelist(G, 'graph.edgelist')

# 写入带权边列表
nx.write_weighted_edgelist(G, 'weighted.edgelist')
```

边列表示例格式：
```
# 源节点 目标节点
0 1
1 2
2 3
3 0
```

带权边列表示例：
```
# 源节点 目标节点 权重
0 1 0.5
1 2 1.0
2 3 0.75
```

### GML（图建模语言）
```python
# 读取GML（保留所有属性）
G = nx.read_gml('graph.gml')

# 写入GML
nx.write_gml(G, 'graph.gml')
```

### GraphML格式
```python
# 读取GraphML（基于XML的格式）
G = nx.read_graphml('graph.graphml')

# 写入GraphML
nx.write_graphml(G, 'graph.graphml')

# 指定编码
nx.write_graphml(G, 'graph.graphml', encoding='utf-8')
```

### GEXF（图交换XML格式）
```python
# 读取GEXF
G = nx.read_gexf('graph.gexf')

# 写入GEXF
nx.write_gexf(G, 'graph.gexf')
```

### Pajek格式
```python
# 读取Pajek .net文件
G = nx.read_pajek('graph.net')

# 写入Pajek格式
nx.write_pajek(G, 'graph.net')
```

### LEDA格式
```python
# 读取LEDA格式
G = nx.read_leda('graph.leda')

# 写入LEDA格式
nx.write_leda(G, 'graph.leda')
```

## 与Pandas集成

### 从Pandas数据框创建
```python
import pandas as pd

# 从边列表数据框创建图
df = pd.DataFrame({
    'source': [1, 2, 3, 4],
    'target': [2, 3, 4, 1],
    'weight': [0.5, 1.0, 0.75, 0.25]
})

# 创建图
G = nx.from_pandas_edgelist(df,
                            source='source',
                            target='target',
                            edge_attr='weight')

# 带多个边属性
G = nx.from_pandas_edgelist(df,
                            source='source',
                            target='target',
                            edge_attr=['weight', 'color', 'type'])

# 创建有向图
G = nx.from_pandas_edgelist(df,
                            source='source',
                            target='target',
                            create_using=nx.DiGraph())
```

### 转换为Pandas数据框
```python
# 将图转换为边列表数据框
df = nx.to_pandas_edgelist(G)

# 指定边属性
df = nx.to_pandas_edgelist(G, source='node1', target='node2')
```

### 使用Pandas的邻接矩阵
```python
# 从邻接矩阵创建数据框
df = nx.to_pandas_adjacency(G, dtype=int)

# 从邻接数据框创建图
G = nx.from_pandas_adjacency(df)

# 用于有向图
G = nx.from_pandas_adjacency(df, create_using=nx.DiGraph())
```

## NumPy与SciPy集成

### 邻接矩阵
```python
import numpy as np

# 转换为NumPy邻接矩阵
A = nx.to_numpy_array(G, dtype=int)

# 指定节点顺序
nodelist = [1, 2, 3, 4, 5]
A = nx.to_numpy_array(G, nodelist=nodelist)

# 从NumPy数组创建图
G = nx.from_numpy_array(A)

# 用于有向图
G = nx.from_numpy_array(A, create_using=nx.DiGraph())
```

### 稀疏矩阵（SciPy）
```python
from scipy import sparse

# 转换为稀疏矩阵
A = nx.to_scipy_sparse_array(G)

# 指定格式（csr, csc, coo等）
A_csr = nx.to_scipy_sparse_array(G, format='csr')

# 从稀疏矩阵创建图
G = nx.from_scipy_sparse_array(A)
```

## JSON格式

### 节点-链接格式
```python
import json

# 转换为节点-链接格式（适用于d3.js）
data = nx.node_link_data(G)
with open('graph.json', 'w') as f:
    json.dump(data, f)

# 从节点-链接格式创建图
with open('graph.json', 'r') as f:
    data = json.load(f)
G = nx.node_link_graph(data)
```

### 邻接数据格式
```python
# 转换为邻接格式
data = nx.adjacency_data(G)
with open('graph.json', 'w') as f:
    json.dump(data, f)

# 从邻接格式创建图
with open('graph.json', 'r') as f:
    data = json.load(f)
G = nx.adjacency_graph(data)
```

### 树数据格式
```python
# 用于树图
data = nx.tree_data(G, root=0)
with open('tree.json', 'w') as f:
    json.dump(data, f)

# 从树格式创建图
with open('tree.json', 'r') as f:
    data = json.load(f)
G = nx.tree_graph(data)
```

## Pickle格式

### 二进制Pickle
```python
import pickle

# 写入pickle（保留所有Python对象）
with open('graph.pkl', 'wb') as f:
    pickle.dump(G, f)

# 读取pickle
with open('graph.pkl', 'rb') as f:
    G = pickle.load(f)

# NetworkX便捷函数
nx.write_gpickle(G, 'graph.gpickle')
G = nx.read_gpickle('graph.gpickle')
```

## CSV文件

### 自定义CSV读取
```python
import csv

# 从CSV读取边
G = nx.Graph()
with open('edges.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        G.add_edge(row['source'], row['target'], weight=float(row['weight']))

# 将边写入CSV
with open('edges.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['source', 'target', 'weight'])
    for u, v, data in G.edges(data=True):
        writer.writerow([u, v, data.get('weight', 1.0)])
```

## 数据库集成

### SQL数据库
```python
import sqlite3
import pandas as pd

# 通过pandas从SQL数据库读取
conn = sqlite3.connect('network.db')
df = pd.read_sql_query("SELECT source, target, weight FROM edges", conn)
G = nx.from_pandas_edgelist(df, 'source', 'target', edge_attr='weight')
conn.close()

# 写入SQL数据库
df = nx.to_pandas_edgelist(G)
conn = sqlite3.connect('network.db')
df.to_sql('edges', conn, if_exists='replace', index=False)
conn.close()
```

## 可视化图格式

### DOT格式（Graphviz）
```python
# 为Graphviz写入DOT文件
nx.drawing.nx_pydot.write_dot(G, 'graph.dot')

# 读取DOT文件
G = nx.drawing.nx_pydot.read_dot('graph.dot')

# 直接生成图像（需要Graphviz）
from networkx.drawing.nx_pydot import to_pydot
pydot_graph = to_pydot(G)
pydot_graph.write_png('graph.png')
```

## Cytoscape集成

### Cytoscape JSON
```python
# 导出到Cytoscape
data = nx.cytoscape_data(G)
with open('cytoscape.json', 'w') as f:
    json.dump(data, f)

# 从Cytoscape导入
with open('cytoscape.json', 'r') as f:
    data = json.load(f)
G = nx.cytoscape_graph(data)
```

## 专用格式

### Matrix Market格式
```python
from scipy.io import mmread, mmwrite

# 读取Matrix Market
A = mmread('graph.mtx')
G = nx.from_scipy_sparse_array(A)

# 写入Matrix Market
A = nx.to_scipy_sparse_array(G)
mmwrite('graph.mtx', A)
```

### Shapefile（地理网络）
```python
# 需要pyshp库
# 从shapefile读取地理网络
G = nx.read_shp('roads.shp')

# 写入shapefile
nx.write_shp(G, 'network')
```

## 格式选择指南

### 根据需求选择

**邻接表** - 简单、可读性强、不支持属性
- 最佳场景：简单无权图、快速查看

**边列表** - 简单、支持权重、可读性强
- 最佳场景：带权图、数据导入/导出

**GML/GraphML** - 完整属性保留、基于XML
- 最佳场景：包含所有元数据的完整图序列化

**JSON** - 兼容Web、JavaScript集成
- 最佳场景：Web应用、d3.js可视化

**Pickle** - 快速、保留Python对象、二进制
- 最佳场景：纯Python存储、复杂属性

**Pandas** - 数据分析集成、数据框操作
- 最佳场景：数据处理流水线、统计分析

**NumPy/SciPy** - 数值计算、稀疏矩阵
- 最佳场景：矩阵运算、科学计算

**DOT** - 可视化、Graphviz集成
- 最佳场景：创建可视化图表

## 性能考量

### 大型图处理
处理大型图时考虑：
```python
# 使用压缩格式
import gzip
with gzip.open('graph.adjlist.gz', 'wt') as f:
    nx.write_adjlist(G, f)

with gzip.open('graph.adjlist.gz', 'rt') as f:
    G = nx.read_adjlist(f)

# 使用二进制格式（更快）
nx.write_gpickle(G, 'graph.gpickle')  # 比文本格式更快

# 使用稀疏矩阵表示邻接
A = nx.to_scipy_sparse_array(G, format='csr')  # 内存高效
```

### 增量加载
处理超大型图：
```python
# 从边列表增量加载图
G = nx.Graph()
with open('huge_graph.edgelist') as f:
    for line in f:
        u, v = line.strip().split()
        G.add_edge(u, v)

        # 分块处理
        if G.number_of_edges() % 100000 == 0:
            print(f"已加载 {G.number_of_edges()} 条边")
```

## 错误处理

### 健壮的文件读取
```python
try:
    G = nx.read_graphml('graph.graphml')
except nx.NetworkXError as e:
    print(f"读取GraphML出错: {e}")
except FileNotFoundError:
    print("文件未找到")
    G = nx.Graph()

# 检查文件格式是否支持
if os.path.exists('graph.txt'):
    with open('graph.txt') as f:
        first_line = f.readline()
        # 检测格式并相应读取
```
