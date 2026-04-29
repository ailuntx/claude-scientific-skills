# LaminDB 数据标注与验证

本文档涵盖 LaminDB 中的数据管理、验证、模式管理及标注最佳实践。

## 概述

LaminDB 的数据管理流程通过三个关键步骤确保数据集既经过验证又可查询：

1. **验证**：确认数据集符合预期模式
2. **标准化**：修复拼写错误和同义词映射等不一致问题
3. **标注**：将数据集与元数据实体关联以实现可查询性

## 模式设计

模式定义了预期的数据结构、类型和验证规则。LaminDB 支持三种主要模式方案：

### 1. 灵活模式

仅验证与特征注册表名称匹配的列，允许附加元数据：

```python
import lamindb as ln

# 创建灵活模式
schema = ln.Schema(
    name="valid_features",
    itype=ln.Feature  # 根据特征注册表验证
).save()

# 任何匹配特征名称的列都将被验证
# 允许附加列但不进行验证
```

### 2. 最小必需模式

指定必需列同时允许附加元数据：

```python
# 定义必需特征
required_features = [
    ln.Feature.get(name="cell_type"),
    ln.Feature.get(name="tissue"),
    ln.Feature.get(name="donor_id")
]

# 创建含必需特征的模式
schema = ln.Schema(
    name="minimal_immune_schema",
    features=required_features,
    flexible=True  # 允许附加列
).save()
```

### 3. 严格模式

强制完全控制数据结构：

```python
# 定义所有允许的特征
all_features = [
    ln.Feature.get(name="cell_type"),
    ln.Feature.get(name="tissue"),
    ln.Feature.get(name="donor_id"),
    ln.Feature.get(name="disease")
]

# 创建严格模式
schema = ln.Schema(
    name="strict_immune_schema",
    features=all_features,
    flexible=False  # 禁止附加列
).save()
```

## DataFrame 管理流程

典型管理流程包含六个关键步骤：

### 步骤 1-2：加载数据并建立注册表

```python
import pandas as pd
import lamindb as ln

# 加载数据
df = pd.read_csv("experiment.csv")

# 定义并保存特征
ln.Feature(name="cell_type", dtype=str).save()
ln.Feature(name="tissue", dtype=str).save()
ln.Feature(name="gene_count", dtype=int).save()
ln.Feature(name="experiment_date", dtype="date").save()

# 填充有效值（若使用受控词汇表）
import bionty as bt
bt.CellType.import_source()
bt.Tissue.import_source()
```

### 步骤 3：创建模式

```python
# 将特征关联到模式
features = [
    ln.Feature.get(name="cell_type"),
    ln.Feature.get(name="tissue"),
    ln.Feature.get(name="gene_count"),
    ln.Feature.get(name="experiment_date")
]

schema = ln.Schema(
    name="experiment_schema",
    features=features,
    flexible=True
).save()
```

### 步骤 4：初始化管理工具并验证

```python
# 初始化管理工具
curator = ln.curators.DataFrameCurator(df, schema)

# 验证数据集
validation = curator.validate()

# 检查验证结果
if validation:
    print("✓ 验证通过")
else:
    print("✗ 验证失败")
    curator.non_validated  # 查看问题字段
```

### 步骤 5：修复验证问题

#### 标准化值

```python
# 修复分类列中的拼写错误和同义词
curator.cat.standardize("cell_type")
curator.cat.standardize("tissue")

# 查看标准化映射
curator.cat.inspect_standardize("cell_type")
```

#### 映射到本体

```python
# 将值映射到本体术语
curator.cat.add_ontology("cell_type", bt.CellType)
curator.cat.add_ontology("tissue", bt.Tissue)

# 为未映射术语查找公共本体
curator.cat.lookup(public=True).cell_type  # 交互式查找
```

#### 添加新术语

```python
# 向注册表添加新有效术语
curator.cat.add_new_from("cell_type")

# 或手动创建记录
new_cell_type = bt.CellType(name="my_novel_cell_type").save()
```

#### 重命名列

```python
# 重命名列以匹配特征名称
df = df.rename(columns={"celltype": "cell_type"})

# 使用修复后的DataFrame重新初始化管理工具
curator = ln.curators.DataFrameCurator(df, schema)
```

### 步骤 6：保存管理后的数据对象

```python
# 保存并关联模式
artifact = curator.save_artifact(
    key="experiments/curated_data.parquet",
    description="已验证标注的实验数据"
)

# 验证数据对象是否关联模式
artifact.schema  # 返回模式对象
artifact.describe()  # 显示验证状态
```

## AnnData 管理

对于复合结构如 AnnData，使用"槽位"验证不同组件：

### 定义 AnnData 模式

```python
# 为不同槽位创建模式
obs_schema = ln.Schema(
    name="cell_metadata",
    features=[
        ln.Feature.get(name="cell_type"),
        ln.Feature.get(name="tissue"),
        ln.Feature.get(name="donor_id")
    ]
).save()

var_schema = ln.Schema(
    name="gene_ids",
    features=[ln.Feature.get(name="ensembl_gene_id")]
).save()

# 创建复合AnnData模式
anndata_schema = ln.Schema(
    name="scrna_schema",
    otype="AnnData",
    slots={
        "obs": obs_schema,
        "var.T": var_schema  # .T 表示转置
    }
).save()
```

### 管理 AnnData 对象

```python
import anndata as ad

# 加载AnnData
adata = ad.read_h5ad("data.h5ad")

# 初始化管理工具
curator = ln.curators.AnnDataCurator(adata, anndata_schema)

# 验证所有槽位
validation = curator.validate()

# 按槽位修复问题
curator.cat.standardize("obs", "cell_type")
curator.cat.add_ontology("obs", "cell_type", bt.CellType)
curator.cat.standardize("var.T", "ensembl_gene_id")

# 保存管理后的数据对象
artifact = curator.save_artifact(
    key="scrna/validated_data.h5ad",
    description="管理后的单细胞RNA-seq数据"
)
```

## MuData 管理

MuData 通过模态特定槽位支持多模态数据：

```python
# 为每个模态定义模式
rna_obs_schema = ln.Schema(name="rna_obs_schema", features=[...]).save()
protein_obs_schema = ln.Schema(name="protein_obs_schema", features=[...]).save()

# 创建MuData模式
mudata_schema = ln.Schema(
    name="multimodal_schema",
    otype="MuData",
    slots={
        "rna:obs": rna_obs_schema,
        "protein:obs": protein_obs_schema
    }
).save()

# 管理流程
curator = ln.curators.MuDataCurator(mdata, mudata_schema)
curator.validate()
```

## SpatialData 管理

空间转录组数据管理：

```python
# 定义空间模式
spatial_schema = ln.Schema(
    name="spatial_schema",
    otype="SpatialData",
    slots={
        "tables:cell_metadata.obs": cell_schema,
        "attrs:bio": bio_metadata_schema
    }
).save()

# 管理流程
curator = ln.curators.SpatialDataCurator(sdata, spatial_schema)
curator.validate()
```

## TileDB-SOMA 管理

可扩展的阵列支持数据管理：

```python
# 定义SOMA模式
soma_schema = ln.Schema(
    name="soma_schema",
    otype="tiledbsoma",
    slots={
        "obs": obs_schema,
        "ms:RNA.T": var_schema  # 测量:模态.T
    }
).save()

# 管理流程
curator = ln.curators.TileDBSOMACurator(soma_exp, soma_schema)
curator.validate()
```

## 特征验证

### 数据类型验证

```python
# 定义类型化特征
ln.Feature(name="age", dtype=int).save()
ln.Feature(name="weight", dtype=float).save()
ln.Feature(name="is_treated", dtype=bool).save()
ln.Feature(name="collection_date", dtype="date").save()

# 验证期间强制类型转换
ln.Feature(name="age_str", dtype=int, coerce_dtype=True).save()  # 自动将字符串转为整数
```

### 值验证

```python
# 根据允许值验证
cell_type_feature = ln.Feature(name="cell_type", dtype=str).save()

# 链接到受控词汇表注册表
cell_type_feature.link_to_registry(bt.CellType)

# 现在验证会检查CellType注册表
curator = ln.curators.DataFrameCurator(df, schema)
curator.validate()  # 若cell_type值不在注册表中则报错
```

## 标准化策略

### 使用公共本体

```python
# 从公共源查找标准化术语
curator.cat.lookup(public=True).cell_type

# 返回含公共本体术语的自动补全对象
# 用户可交互式选择正确术语
```

### 同义词映射

```python
# 向记录添加同义词
t_cell = bt.CellType.get(name="T cell")
t_cell.add_synonym("T lymphocyte")
t_cell.add_synonym("T-cell")

# 现在标准化自动映射同义词
curator.cat.standardize("cell_type")
# "T lymphocyte" → "T cell"
# "T-cell" → "T cell"
```

### 自定义标准化

```python
# 手动映射
mapping = {
    "TCell": "T cell",
    "t cell": "T cell",
    "T-cells": "T cell"
}

# 应用映射
df["cell_type"] = df["cell_type"].map(lambda x: mapping.get(x, x))
```

## 处理验证错误

### 常见问题及解决方案

**问题：列不在模式中**
```python
# 方案1：重命名列
df = df.rename(columns={"old_name": "feature_name"})

# 方案2：向模式添加特征
new_feature = ln.Feature(name="new_column", dtype=str).save()
schema.features.add(new_feature)
```

**问题：无效值**
```python
# 方案1：标准化
curator.cat.standardize("column_name")

# 方案2：添加新有效值
curator.cat.add_new_from("column_name")

# 方案3：映射到本体
curator.cat.add_ontology("column_name", bt.Registry)
```

**问题：数据类型不匹配**
```python
# 方案1：转换数据类型
df["column"] = df["column"].astype(int)

# 方案2：在特征中启用强制转换
feature = ln.Feature.get(name="column")
feature.coerce_dtype = True
feature.save()
```

## 模式版本控制

模式可像其他记录一样进行版本控制：

```python
# 创建初始模式
schema_v1 = ln.Schema(name="experiment_schema", features=[...]).save()

# 更新模式添加新特征
schema_v2 = ln.Schema(
    name="experiment_schema",
    features=[...],  # 更新后的列表
    version="2"
).save()

# 将数据对象关联到特定模式版本
artifact.schema = schema_v2
artifact.save()
```

## 查询已验证数据

数据经验证标注后即可查询：

```python
# 查找所有已验证数据对象
ln.Artifact.filter(is_valid=True).to_dataframe()

# 查找含特定模式的数据对象
ln.Artifact.filter(schema=schema).to_dataframe()

# 按标注特征查询
ln.Artifact.filter(cell_type="T cell", tissue="blood").to_dataframe()

# 在结果中包含特征
ln.Artifact.filter(is_valid=True).to_dataframe(include="features")
```

## 最佳实践

1. **先定义特征**：在管理前创建特征注册表
2. **使用公共本体**：利用 bt.lookup(public=True) 进行标准化
3. **从灵活开始**：初期使用灵活模式，随理解深入逐步收紧
4. **记录槽位**：在复合模式中明确指定转置(.T)
5. **尽早标准化**：在验证前修复拼写错误和同义词
6. **增量验证**：对复合结构分别验证每个槽位
7. **版本化模式**：跟踪模式随时间的变化
8. **添加同义词**：注册常见变体以简化后续管理
9. **谨慎强制类型**：仅在安全时启用dtype强制转换
10. **样本测试**：全数据集管理前验证小样本

## 高级：自定义验证器

创建自定义验证逻辑：

```python
def validate_gene_expression(df):
    """基因表达值的自定义验证器"""
    # 检查非负性
    if (df < 0).any().any():
        return False, "发现负表达值"

    # 检查合理范围
    if (df > 1e6).any().any():
        return False, "检测到过高表达值"

    return True, "验证通过"

# 在管理过程中应用
is_valid, message = validate_gene_expression(df)
if not is_valid:
    print(f"验证失败: {message}")
```

## 跟踪管理溯源

```python
# 管理后的数据对象跟踪管理谱系
ln.track()  # 开始跟踪

# 执行管理流程
curator = ln.curators.DataFrameCurator(df, schema)
curator.validate()
curator.cat.standardize("cell_type")
artifact = curator.save_artifact(key="curated.parquet")

ln.finish()  # 完成跟踪

# 查看管理谱系
artifact.run.describe()  # 显示管理转换
artifact.view_lineage()  # 可视化管理流程
```
