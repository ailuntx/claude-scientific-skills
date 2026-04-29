# 常见查询模式与最佳实践

## 查询模式分类

### 1. 探索性查询（仅元数据）
在不加载表达矩阵时探索可用数据。

**模式：获取组织中唯一细胞类型**
```python
import cellxgene_census

with cellxgene_census.open_soma() as census:
    cell_metadata = cellxgene_census.get_obs(
        census,
        "homo_sapiens",
        value_filter="tissue_general == 'brain' and is_primary_data == True",
        column_names=["cell_type"]
    )
    unique_cell_types = cell_metadata["cell_type"].unique()
    print(f"发现 {len(unique_cell_types)} 种唯一细胞类型")
```

**模式：按条件统计细胞数量**
```python
cell_metadata = cellxgene_census.get_obs(
    census,
    "homo_sapiens",
    value_filter="disease != 'normal' and is_primary_data == True",
    column_names=["disease", "tissue_general"]
)
counts = cell_metadata.groupby(["disease", "tissue_general"]).size()
```

**模式：探索数据集信息**
```python
# 访问数据集表
datasets = census["census_info"]["datasets"].read().concat().to_pandas()

# 按特定条件筛选
covid_datasets = datasets[datasets["disease"].str.contains("COVID", na=False)]
```

### 2. 中小型查询（AnnData）
当结果可放入内存时使用（通常 < 10万细胞）。

**模式：组织特异性细胞类型查询**
```python
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",
    obs_value_filter="cell_type == 'B cell' and tissue_general == 'lung' and is_primary_data == True",
    obs_column_names=["assay", "disease", "sex", "donor_id"],
)
```

**模式：多基因特异性查询**
```python
marker_genes = ["CD4", "CD8A", "CD19", "FOXP3"]

# 首先获取基因ID
gene_metadata = cellxgene_census.get_var(
    census, "homo_sapiens",
    value_filter=f"feature_name in {marker_genes}",
    column_names=["feature_id", "feature_name"]
)
gene_ids = gene_metadata["feature_id"].tolist()

# 使用基因过滤器查询
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",
    var_value_filter=f"feature_id in {gene_ids}",
    obs_value_filter="cell_type == 'T cell' and is_primary_data == True",
)
```

**模式：多组织查询**
```python
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",
    obs_value_filter="tissue_general in ['lung', 'liver', 'kidney'] and is_primary_data == True",
    obs_column_names=["cell_type", "tissue_general", "dataset_id"],
)
```

**模式：疾病特异性查询**
```python
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",
    obs_value_filter="disease == 'COVID-19' and tissue_general == 'lung' and is_primary_data == True",
)
```

### 3. 大型查询（核外处理）
当查询超出可用内存时使用`axis_query()`。

**模式：迭代处理**
```python
import pyarrow as pa

# 创建查询
query = census["census_data"]["homo_sapiens"].axis_query(
    measurement_name="RNA",
    obs_query=soma.AxisQuery(
        value_filter="tissue_general == 'brain' and is_primary_data == True"
    ),
    var_query=soma.AxisQuery(
        value_filter="feature_name in ['FOXP2', 'TBR1', 'SATB2']"
    )
)

# 分块迭代X矩阵
iterator = query.X("raw").tables()
for batch in iterator:
    # 处理批次（pyarrow.Table对象）
    # batch包含列：soma_data, soma_dim_0, soma_dim_1
    process_batch(batch)
```

**模式：增量统计（均值/方差）**
```python
# 使用Welford在线算法
n = 0
mean = 0
M2 = 0

iterator = query.X("raw").tables()
for batch in iterator:
    values = batch["soma_data"].to_numpy()
    for x in values:
        n += 1
        delta = x - mean
        mean += delta / n
        delta2 = x - mean
        M2 += delta * delta2

variance = M2 / (n - 1) if n > 1 else 0
```

### 4. PyTorch集成（机器学习）
使用`experiment_dataloader()`训练模型。

**模式：创建训练数据加载器**
```python
from cellxgene_census.experimental.ml import experiment_dataloader
import torch

with cellxgene_census.open_soma() as census:
    # 创建数据加载器
    dataloader = experiment_dataloader(
        census["census_data"]["homo_sapiens"],
        measurement_name="RNA",
        X_name="raw",
        obs_value_filter="tissue_general == 'liver' and is_primary_data == True",
        obs_column_names=["cell_type"],
        batch_size=128,
        shuffle=True,
    )

    # 训练循环
    for epoch in range(num_epochs):
        for batch in dataloader:
            X = batch["X"]  # 基因表达
            labels = batch["obs"]["cell_type"]  # 细胞类型标签
            # 训练模型...
```

**模式：训练/测试集分割**
```python
from cellxgene_census.experimental.ml import ExperimentDataset

# 从查询创建数据集
dataset = ExperimentDataset(
    experiment_axis_query,
    layer_name="raw",
    obs_column_names=["cell_type"],
    batch_size=128,
)

# 分割数据
train_dataset, test_dataset = dataset.random_split(
    split=[0.8, 0.2],
    seed=42
)

# 创建加载器
train_loader = experiment_dataloader(train_dataset)
test_loader = experiment_dataloader(test_dataset)
```

### 5. 集成工作流

**模式：Scanpy集成**
```python
import scanpy as sc

# 加载数据
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",
    obs_value_filter="cell_type == 'neuron' and is_primary_data == True",
)

# 标准Scanpy工作流
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata)
sc.pp.pca(adata)
sc.pp.neighbors(adata)
sc.tl.umap(adata)
sc.pl.umap(adata, color=["cell_type", "tissue_general"])
```

**模式：多数据集集成**
```python
# 分别查询多个数据集
datasets_to_integrate = ["dataset_id_1", "dataset_id_2", "dataset_id_3"]

adatas = []
for dataset_id in datasets_to_integrate:
    adata = cellxgene_census.get_anndata(
        census=census,
        organism="Homo sapiens",
        obs_value_filter=f"dataset_id == '{dataset_id}' and is_primary_data == True",
    )
    adatas.append(adata)

# 使用scanorama/harmony等工具集成
import scanpy.external as sce
sce.pp.scanorama_integrate(adatas)
```

## 最佳实践

### 1. 始终筛选主数据
除非专门分析重复数据，否则始终包含`is_primary_data == True`：
```python
obs_value_filter="cell_type == 'B cell' and is_primary_data == True"
```

### 2. 指定Census版本
为保证可复现性，始终指定版本：
```python
census = cellxgene_census.open_soma(census_version="2023-07-25")
```

### 3. 使用上下文管理器
始终使用上下文管理器确保资源清理：
```python
with cellxgene_census.open_soma() as census:
    # 在此编写代码
```

### 4. 仅选择所需列
通过仅选择必要元数据列减少数据传输：
```python
obs_column_names=["cell_type", "tissue_general", "disease"]  # 非全部列
```

### 5. 检查基因查询的数据集存在性
分析特定基因时，检查哪些数据集测量了它们：
```python
presence = cellxgene_census.get_presence_matrix(
    census,
    "homo_sapiens",
    var_value_filter="feature_name in ['CD4', 'CD8A']"
)
```

### 6. 使用tissue_general进行广泛查询
`tissue_general`提供比`tissue`更粗粒度的分组，适用于跨组织分析：
```python
# 更适合广泛查询
obs_value_filter="tissue_general == 'immune system'"

# 需要时使用特定组织
obs_value_filter="tissue == 'peripheral blood mononuclear cell'"
```

### 7. 结合元数据探索与表达查询
先探索元数据了解可用数据，再查询表达：
```python
# 步骤1：探索
metadata = cellxgene_census.get_obs(
    census, "homo_sapiens",
    value_filter="disease == 'COVID-19'",
    column_names=["cell_type", "tissue_general"]
)
print(metadata.value_counts())

# 步骤2：基于发现进行查询
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",
    obs_value_filter="disease == 'COVID-19' and cell_type == 'T cell' and is_primary_data == True",
)
```

### 8. 大型查询的内存管理
加载前检查预估大小：
```python
# 先获取细胞计数
metadata = cellxgene_census.get_obs(
    census, "homo_sapiens",
    value_filter="tissue_general == 'brain' and is_primary_data == True",
    column_names=["soma_joinid"]
)
n_cells = len(metadata)
print(f"查询将返回 {n_cells} 个细胞")

# 如果过大，使用核外处理或进一步筛选
```

### 9. 利用本体术语保证一致性
尽可能使用本体术语ID而非自由文本：
```python
# 比跨数据集的cell_type == 'B cell'更可靠
obs_value_filter="cell_type_ontology_term_id == 'CL:0000236'"
```

### 10. 批处理模式
跨多条件系统分析：
```python
tissues = ["lung", "liver", "kidney", "heart"]
results = {}

for tissue in tissues:
    adata = cellxgene_census.get_anndata(
        census=census,
        organism="Homo sapiens",
        obs_value_filter=f"tissue_general == '{tissue}' and is_primary_data == True",
    )
    # 执行分析
    results[tissue] = analyze(adata)
```

## 常见陷阱避免

1. **未筛选is_primary_data**：导致重复细胞计数
2. **加载过多数据**：先用元数据查询预估大小
3. **未使用上下文管理器**：可能引起资源泄漏
4. **版本不一致**：未指定版本导致结果不可复现
5. **查询范围过广**：从聚焦查询开始，按需扩展
6. **忽略数据集存在性**：部分基因未在所有数据集测量
7. **错误计数标准化**：注意UMI与read count差异
