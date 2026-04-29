# 自定义数据集 — 完整参考指南

如何创建自己的图数据集并从原始数据源（CSV、pandas、numpy等）加载图数据。

## 快速方法：无需数据集类

对于合成数据或一次性图数据，可跳过数据集机制——直接创建 `Data` 对象并传递给 `DataLoader`：

```python
from torch_geometric.data import Data
from torch_geometric.loader import DataLoader

data_list = [Data(x=..., edge_index=..., y=...) for _ in range(100)]
loader = DataLoader(data_list, batch_size=32)
```

## InMemoryDataset（内存数据集）

适用于可复用且能放入CPU内存的数据集。需重写4个方法：

```python
from torch_geometric.data import InMemoryDataset, download_url

class MyDataset(InMemoryDataset):
    def __init__(self, root, transform=None, pre_transform=None, pre_filter=None):
        super().__init__(root, transform, pre_transform, pre_filter)
        self.load(self.processed_paths[0])

    @property
    def raw_file_names(self):
        # 必须存在于raw_dir中的文件才能跳过download()
        return ['data.csv']

    @property
    def processed_file_names(self):
        # 必须存在于processed_dir中的文件才能跳过process()
        return ['data.pt']

    def download(self):
        # 将原始文件下载到self.raw_dir
        download_url('https://example.com/data.csv', self.raw_dir)

    def process(self):
        # 读取原始数据并创建Data对象列表
        data_list = [...]

        if self.pre_filter is not None:
            data_list = [d for d in data_list if self.pre_filter(d)]
        if self.pre_transform is not None:
            data_list = [self.pre_transform(d) for d in data_list]

        # save()将列表合并为一个Data对象+切片字典后保存
        self.save(data_list, self.processed_paths[0])
```

**自动创建的目录结构：**
```
root/
├── raw/          # raw_dir — 下载的原始文件存放位置
│   └── data.csv
└── processed/    # processed_dir — 处理后的.pt文件存放位置
    └── data.pt
```

**关键行为：**
- 仅当`raw_dir`中缺少`raw_file_names`指定的文件时才会运行`download()`
- 仅当`processed_dir`中缺少`processed_file_names`指定的文件时才会运行`process()`
- 若修改`pre_transform`，需删除`processed/`目录以重新处理

## Dataset（超内存数据集）

对于超大型数据集，单独保存每个图：

```python
import os.path as osp
import torch
from torch_geometric.data import Dataset, download_url

class LargeDataset(Dataset):
    def __init__(self, root, transform=None, pre_transform=None):
        super().__init__(root, transform, pre_transform)

    @property
    def raw_file_names(self):
        return ['graph_data.csv']

    @property
    def processed_file_names(self):
        return [f'data_{i}.pt' for i in range(1000)]

    def download(self):
        download_url('...', self.raw_dir)

    def process(self):
        for idx in range(1000):
            data = Data(...)  # 从原始数据构建图
            if self.pre_filter is not None and not self.pre_filter(data):
                continue
            if self.pre_transform is not None:
                data = self.pre_transform(data)
            torch.save(data, osp.join(self.processed_dir, f'data_{idx}.pt'))

    def len(self):
        return 1000

    def get(self, idx):
        return torch.load(osp.join(self.processed_dir, f'data_{idx}.pt'))
```

## 从CSV加载图数据

通用模式：将节点/边数据从CSV文件加载到HeteroData对象中。

### 步骤1：加载节点特征

```python
import pandas as pd
import torch

def load_node_csv(path, index_col, encoders=None):
    df = pd.read_csv(path, index_col=index_col)
    # 将原始ID映射到连续的0..N-1索引
    mapping = {idx: i for i, idx in enumerate(df.index.unique())}

    x = None
    if encoders is not None:
        xs = [encoder(df[col]) for col, encoder in encoders.items()]
        x = torch.cat(xs, dim=-1)

    return x, mapping
```

### 步骤2：加载边数据

```python
def load_edge_csv(path, src_index_col, src_mapping, dst_index_col, dst_mapping,
                  encoders=None):
    df = pd.read_csv(path)
    src = [src_mapping[idx] for idx in df[src_index_col]]
    dst = [dst_mapping[idx] for idx in df[dst_index_col]]
    edge_index = torch.tensor([src, dst])

    edge_attr = None
    if encoders is not None:
        edge_attrs = [encoder(df[col]) for col, encoder in encoders.items()]
        edge_attr = torch.cat(edge_attrs, dim=-1)

    return edge_index, edge_attr
```

### 步骤3：组装HeteroData对象

```python
from torch_geometric.data import HeteroData

# 加载节点
movie_x, movie_mapping = load_node_csv('movies.csv', 'movieId',
    encoders={'genres': GenresEncoder()})
_, user_mapping = load_node_csv('ratings.csv', 'userId')

# 加载边
edge_index, edge_label = load_edge_csv('ratings.csv',
    src_index_col='userId', src_mapping=user_mapping,
    dst_index_col='movieId', dst_mapping=movie_mapping,
    encoders={'rating': IdentityEncoder(dtype=torch.long)})

# 构建HeteroData
data = HeteroData()
data['user'].num_nodes = len(user_mapping)
data['movie'].x = movie_x
data['user', 'rates', 'movie'].edge_index = edge_index
data['user', 'rates', 'movie'].edge_label = edge_label
```

### 常用编码器

```python
class IdentityEncoder:
    """将数值列原样编码"""
    def __init__(self, dtype=None):
        self.dtype = dtype
    def __call__(self, df):
        return torch.from_numpy(df.values).view(-1, 1).to(self.dtype)

class GenresEncoder:
    """对竖线分隔的分类列进行多热编码"""
    def __init__(self, sep='|'):
        self.sep = sep
    def __call__(self, df):
        genres = set(g for col in df.values for g in col.split(self.sep))
        mapping = {genre: i for i, genre in enumerate(genres)}
        x = torch.zeros(len(df), len(mapping))
        for i, col in enumerate(df.values):
            for genre in col.split(self.sep):
                x[i, mapping[genre]] = 1
        return x
```

对于文本特征，使用sentence-transformers：

```python
from sentence_transformers import SentenceTransformer

class SequenceEncoder:
    def __init__(self, model_name='all-MiniLM-L6-v2'):
        self.model = SentenceTransformer(model_name)
    @torch.no_grad()
    def __call__(self, df):
        return self.model.encode(df.values, convert_to_tensor=True).cpu()
```

## 从NetworkX导入

```python
from torch_geometric.utils import from_networkx
import networkx as nx

G = nx.karate_club_graph()
data = from_networkx(G)
# 节点属性转为data.x，边属性转为data.edge_attr
```

## 从scipy稀疏邻接矩阵导入

```python
from torch_geometric.utils import from_scipy_sparse_matrix

edge_index, edge_attr = from_scipy_sparse_matrix(adj_matrix)
data = Data(x=features, edge_index=edge_index)
```

## 无特征节点处理

若节点无特征，常用方案：
- 使用`torch.nn.Embedding`在训练中学习特征
- 设置`data['node_type'].num_nodes = N`（针对HeteroData）
- 使用结构特征：度、聚类系数等
- 使用`data.x = torch.eye(num_nodes)`（独热编码，仅适用于小型图）
