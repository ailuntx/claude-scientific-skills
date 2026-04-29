---
name: cellxgene-census
description: 通过编程方式查询CELLxGENE Census（6100万+细胞）。当您需要从全球最大规模精选单细胞图谱中获取跨组织、疾病或细胞类型的表达数据时使用。最适合群体规模查询、参考图谱比对。分析自有数据请使用scanpy或scvi-tools。
license: 未知
metadata:
    skill-author: K-Dense Inc.
---

# CZ CELLxGENE Census

## 概述

CZ CELLxGENE Census 提供对CZ CELLxGENE Discover中标准化单细胞基因组学数据的程序化访问，这是一个经过版本控制的综合数据集。该技能支持高效查询和分析数千个数据集中的数百万细胞。

Census包含：
- **6100万+细胞**（人类与小鼠）
- **标准化元数据**（细胞类型、组织、疾病、供体）
- **原始基因表达**矩阵
- **预计算嵌入**与统计量
- **与PyTorch、scanpy等分析工具的集成**

## 适用场景

该技能适用于：
- 按细胞类型、组织或疾病查询单细胞表达数据
- 探索可用单细胞数据集及元数据
- 在单细胞数据上训练机器学习模型
- 执行大规模跨数据集分析
- 将Census数据与scanpy等分析框架集成
- 计算数百万细胞的统计量
- 访问预计算嵌入或模型预测结果

## 安装与配置

安装Census API：
```bash
uv pip install cellxgene-census
```

机器学习工作流需额外依赖：
```bash
uv pip install cellxgene-census[experimental]
```

## 核心工作模式

### 1. 打开Census

始终使用上下文管理器确保资源清理：
```python
import cellxgene_census

# 打开最新稳定版
with cellxgene_census.open_soma() as census:
    # 操作census数据

# 指定版本确保可复现性
with cellxgene_census.open_soma(census_version="2023-07-25") as census:
    # 操作census数据
```

**关键点：**
- 使用上下文管理器（`with`语句）自动清理资源
- 指定`census_version`确保分析可复现
- 默认打开最新"稳定"版本

### 2. 探索Census信息

查询表达数据前，先探索可用数据集和元数据。

**获取摘要信息：**
```python
# 读取统计摘要
summary = census["census_info"]["summary"].read().concat().to_pandas()
print(f"总细胞数: {summary['total_cell_count'][0]}")

# 获取所有数据集
datasets = census["census_info"]["datasets"].read().concat().to_pandas()

# 按条件筛选数据集
covid_datasets = datasets[datasets["disease"].str.contains("COVID", na=False)]
```

**查询细胞元数据理解数据结构：**
```python
# 获取组织中唯一细胞类型
cell_metadata = cellxgene_census.get_obs(
    census,
    "homo_sapiens",
    value_filter="tissue_general == 'brain' and is_primary_data == True",
    column_names=["cell_type"]
)
unique_cell_types = cell_metadata["cell_type"].unique()
print(f"在脑组织中发现{len(unique_cell_types)}种细胞类型")

# 按组织统计细胞数量
tissue_counts = cell_metadata.groupby("tissue_general").size()
```

**重要提示：** 除非专门分析重复细胞，否则始终添加`is_primary_data == True`过滤条件避免重复计数。

### 3. 查询表达数据（中小规模）

对于返回<10万细胞且内存可容纳的查询，使用`get_anndata()`：
```python
# 基础查询（细胞类型+组织过滤）
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",  # 或"Mus musculus"
    obs_value_filter="cell_type == 'B cell' and tissue_general == 'lung' and is_primary_data == True",
    obs_column_names=["assay", "disease", "sex", "donor_id"],
)

# 多条件基因查询
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",
    var_value_filter="feature_name in ['CD4', 'CD8A', 'CD19', 'FOXP3']",
    obs_value_filter="cell_type == 'T cell' and disease == 'COVID-19' and is_primary_data == True",
    obs_column_names=["cell_type", "tissue_general", "donor_id"],
)
```

**过滤语法：**
- `obs_value_filter`用于细胞过滤
- `var_value_filter`用于基因过滤
- 用`and`/`or`组合条件
- 多值查询用`in`：`tissue in ['lung', 'liver']`
- 通过`obs_column_names`选择所需列

**单独获取元数据：**
```python
# 查询细胞元数据
cell_metadata = cellxgene_census.get_obs(
    census, "homo_sapiens",
    value_filter="disease == 'COVID-19' and is_primary_data == True",
    column_names=["cell_type", "tissue_general", "donor_id"]
)

# 查询基因元数据
gene_metadata = cellxgene_census.get_var(
    census, "homo_sapiens",
    value_filter="feature_name in ['CD4', 'CD8A']",
    column_names=["feature_id", "feature_name", "feature_length"]
)
```

### 4. 大规模查询（核外处理）

当查询超出内存容量时，使用`axis_query()`进行迭代处理：
```python
import tiledbsoma as soma

# 创建轴向查询
query = census["census_data"]["homo_sapiens"].axis_query(
    measurement_name="RNA",
    obs_query=soma.AxisQuery(
        value_filter="tissue_general == 'brain' and is_primary_data == True"
    ),
    var_query=soma.AxisQuery(
        value_filter="feature_name in ['FOXP2', 'TBR1', 'SATB2']"
    )
)

# 分块迭代表达矩阵
iterator = query.X("raw").tables()
for batch in iterator:
    # batch是pyarrow.Table，包含列：
    # - soma_data: 表达值
    # - soma_dim_0: 细胞（obs）坐标
    # - soma_dim_1: 基因（var）坐标
    process_batch(batch)
```

**增量统计计算：**
```python
# 示例：计算平均表达量
n_observations = 0
sum_values = 0.0

iterator = query.X("raw").tables()
for batch in iterator:
    values = batch["soma_data"].to_numpy()
    n_observations += len(values)
    sum_values += values.sum()

mean_expression = sum_values / n_observations
```

### 5. 使用PyTorch进行机器学习

训练模型时使用实验性PyTorch集成：
```python
from cellxgene_census.experimental.ml import experiment_dataloader

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
            X = batch["X"]  # 基因表达张量
            labels = batch["obs"]["cell_type"]  # 细胞类型标签

            # 前向传播
            outputs = model(X)
            loss = criterion(outputs, labels)

            # 反向传播
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
```

**训练/测试集划分：**
```python
from cellxgene_census.experimental.ml import ExperimentDataset

# 从实验创建数据集
dataset = ExperimentDataset(
    experiment_axis_query,
    layer_name="raw",
    obs_column_names=["cell_type"],
    batch_size=128,
)

# 划分训练/测试集
train_dataset, test_dataset = dataset.random_split(
    split=[0.8, 0.2],
    seed=42
)
```

### 6. 与Scanpy集成

无缝衔接Census数据与scanpy工作流：
```python
import scanpy as sc

# 从Census加载数据
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",
    obs_value_filter="cell_type == 'neuron' and tissue_general == 'cortex' and is_primary_data == True",
)

# 标准scanpy流程
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata, n_top_genes=2000)

# 降维处理
sc.pp.pca(adata, n_comps=50)
sc.pp.neighbors(adata)
sc.tl.umap(adata)

# 可视化
sc.pl.umap(adata, color=["cell_type", "tissue", "disease"])
```

### 7. 多数据集集成

查询并集成多个数据集：
```python
# 策略1：分别查询不同组织
tissues = ["lung", "liver", "kidney"]
adatas = []

for tissue in tissues:
    adata = cellxgene_census.get_anndata(
        census=census,
        organism="Homo sapiens",
        obs_value_filter=f"tissue_general == '{tissue}' and is_primary_data == True",
    )
    adata.obs["tissue"] = tissue
    adatas.append(adata)

# 合并数据集
combined = adatas[0].concatenate(adatas[1:])

# 策略2：直接查询多数据集
adata = cellxgene_census.get_anndata(
    census=census,
    organism="Homo sapiens",
    obs_value_filter="tissue_general in ['lung', 'liver', 'kidney'] and is_primary_data == True",
)
```

## 核心概念与最佳实践

### 始终过滤主数据
除非分析重复细胞，否则查询中始终包含`is_primary_data == True`避免重复计数：
```python
obs_value_filter="cell_type == 'B cell' and is_primary_data == True"
```

### 指定Census版本确保可复现性
生产分析中始终指定Census版本：
```python
census = cellxgene_census.open_soma(census_version="2023-07-25")
```

### 加载前预估查询规模
大型查询前先检查细胞数量避免内存问题：
```python
# 获取细胞计数
metadata = cellxgene_census.get_obs(
    census, "homo_sapiens",
    value_filter="tissue_general == 'brain' and is_primary_data == True",
    column_names=["soma_joinid"]
)
n_cells = len(metadata)
print(f"查询将返回{n_cells:,}个细胞")

# 若规模过大(>10万)，使用核外处理
```

### 使用tissue_general进行粗粒度分组
`tissue_general`字段提供比`tissue`更宽泛的分类，适用于跨组织分析：
```python
# 粗粒度分组
obs_value_filter="tissue_general == 'immune system'"

# 特定组织
obs_value_filter="tissue == 'peripheral blood mononuclear cell'"
```

### 仅选择必要列
通过指定元数据列减少数据传输：
```python
obs_column_names=["cell_type", "tissue_general", "disease"]  # 非全量列
```

### 检查基因特异性查询的数据集覆盖度
分析特定基因时，验证哪些数据集包含该基因：
```python
presence = cellxgene_census.get_presence_matrix(
    census,
    "homo_sapiens",
    var_value_filter="feature_name in ['CD4', 'CD8A']"
)
```

### 两步工作流：先探索后查询
先探索元数据了解数据分布，再查询表达：
```python
# 步骤1：探索可用数据
metadata = cellxgene_census.get_obs(
    census, "homo_sapiens",
    value_filter="disease == 'COVID-19' and is_primary_data == True",
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

## 可用元数据字段

### 细胞元数据（obs）
关键过滤字段：
- `cell_type`, `cell_type_ontology_term_id`
- `tissue`, `tissue_general`, `tissue_ontology_term_id`
- `disease`, `disease_ontology_term_id`
- `assay`, `assay_ontology_term_id`
- `donor_id`, `sex`, `self_reported_ethnicity`
- `development_stage`, `development_stage_ontology_term_id`
- `dataset_id`
- `is_primary_data`（布尔值：True=唯一细胞）

### 基因元数据（var）
- `feature_id`（Ensembl基因ID，如"ENSG00000161798"）
- `feature_name`（基因符号，如"FOXP2"）
- `feature_length`（基因长度，单位碱基对）

## 参考文档

本技能包含详细参考文档：

### references/census_schema.md
完整文档包含：
- Census数据结构与组织方式
- 所有可用元数据字段
- 值过滤语法与运算符
- SOMA对象类型
- 数据纳入标准

**适用场景：** 需要详细模式信息、元数据字段全集或复杂过滤语法时

### references/common_patterns.md
示例与模式：
- 探索性查询（仅元数据）
- 中小规模查询（AnnData）
- 大规模查询（核外处理）
- PyTorch集成
- Scanpy集成工作流
- 多数据集集成
- 最佳实践与常见陷阱

**适用场景：** 实现特定查询模式、查找代码示例或排查常见问题时

## 典型用例

### 用例1：探索组织中的细胞类型
```python
with cellxgene_census.open_soma() as census:
    cells = cellxgene_census.get_obs(
        census, "homo_sapiens",
        value_filter="tissue_general == 'lung' and is_primary_data == True",
        column_names=["cell_type"]
    )
    print(cells["cell_type"].value_counts())
```

### 用例2：查询标记基因表达
```python
with cellxgene_census.open_soma() as census:
    adata = cellxgene_census.get_anndata(
        census=census,
        organism="Homo sapiens",
        var_value_filter="feature_name in ['CD4', 'CD8A', 'CD19']",
        obs_value_filter="cell_type in ['T cell', 'B cell'] and is_primary_data == True",
    )
```

### 用例3：训练细胞类型分类器
```python
from cellxgene_census.experimental.ml import experiment_dataloader

with cellxgene_census.open_soma() as census:
    dataloader = experiment_dataloader(
        census["census_data"]["homo_sapiens"],
        measurement_name="RNA",
        X_name="raw",
        obs_value_filter="is_primary_data == True",
        obs_column_names=["cell_type"],
        batch_size=128,
        shuffle=True,
    )

    # 训练模型
    for epoch in range(epochs):
        for batch in dataloader:
            # 训练逻辑
            pass
```

### 用例4：跨组织分析
```python
with cellxgene_census.open_soma() as census:
    adata = cellxgene_census.get_anndata(
        census=census,
        organism="Homo sapiens",
        obs_value_filter="cell_type == 'macrophage' and tissue_general in ['lung', 'liver', 'brain'] and is_primary_data == True",
    )

    # 分析巨噬细胞跨组织差异
    sc.tl.rank_genes_groups(adata, groupby="tissue_general")
```

## 故障排查

### 查询返回过多细胞
- 添加更具体的过滤条件缩小范围
- 使用`tissue`替代`tissue_general`获取更细粒度
- 若已知特定`dataset_id`则进行过滤
- 对大型查询切换至核外处理

### 内存错误
-

- 尝试使用 `feature_id` 而非 `feature_name` 输入 Ensembl ID
- 检查数据集存在矩阵以确认基因是否已被测量
- 在构建 Census 过程中，部分基因可能已被过滤

### 版本不一致问题
- 始终明确指定 `census_version`
- 在所有分析中使用相同版本
- 查看版本发布说明以了解版本特定的更改
