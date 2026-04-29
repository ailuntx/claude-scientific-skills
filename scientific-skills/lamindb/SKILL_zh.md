---
name: lamindb
description: 本技能适用于操作LaminDB——一个为生物学设计的开源数据框架，旨在使数据可查询、可追溯、可复现且符合FAIR原则。适用于管理生物数据集（scRNA-seq、空间组学、流式细胞术等）、追踪计算工作流、使用生物本体论管理和验证数据、构建数据湖仓、或确保生物研究中的数据溯源与可复现性。涵盖数据管理、注释、本体论（基因、细胞类型、疾病、组织）、模式验证、与工作流管理器（Nextflow、Snakemake）及MLOps平台（W&B、MLflow）的集成，以及部署策略。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# LaminDB

## 概述

LaminDB 是一个为生物学设计的开源数据框架，旨在使数据可查询、可追溯、可复现且符合FAIR（可发现、可访问、可互操作、可重用）原则。它通过单一Python API提供统一平台，整合了湖仓架构、溯源追踪、特征存储、生物本体论、LIMS（实验室信息管理系统）和ELN（电子实验记录本）功能。

**核心价值主张：**
- **可查询性**：通过元数据、特征和本体术语搜索过滤数据集
- **可追溯性**：从原始数据到分析结果的自动溯源追踪
- **可复现性**：数据、代码和环境的版本控制
- **FAIR合规性**：使用生物本体论进行标准化注释

## 适用场景

在以下场景使用本技能：

- **管理生物数据集**：scRNA-seq、bulk RNA-seq、空间转录组、流式细胞术、多模态数据、EHR数据
- **追踪计算工作流**：笔记本、脚本、流程执行（Nextflow、Snakemake、Redun）
- **管理和验证数据**：模式验证、标准化、基于本体论的注释
- **操作生物本体论**：基因、蛋白质、细胞类型、组织、疾病、通路（通过Bionty）
- **构建数据湖仓**：跨多数据集的统一查询接口
- **确保可复现性**：自动版本控制、溯源追踪、环境捕获
- **集成ML流程**：连接Weights & Biases、MLflow、HuggingFace、scVI-tools
- **部署数据基础设施**：搭建本地或云端数据管理系统
- **数据集协作**：共享带标准化元数据的标注数据集

## 核心能力

LaminDB提供六大互相关联的能力模块，详细文档见references文件夹。

### 1. 核心概念与数据溯源

**核心实体：**
- **Artifacts（数据对象）**：版本化数据集（DataFrame、AnnData、Parquet、Zarr等）
- **Records（实验记录）**：实验实体（样本、干预措施、仪器）
- **Runs & Transforms（运行与转换）**：计算过程溯源（代码与产出数据的关联）
- **Features（特征）**：用于注释和查询的类型化元数据字段

**关键工作流：**
- 从文件或Python对象创建版本化数据对象
- 使用`ln.track()`和`ln.finish()`追踪笔记本/脚本执行
- 用类型化特征标注数据对象
- 通过`artifact.view_lineage()`可视化数据溯源图
- 基于来源查询（查找特定代码/输入的所有输出）

**参考文档：** `references/core-concepts.md` - 阅读此文件了解数据对象、记录、运行、转换、特征、版本控制和溯源的详细信息。

### 2. 数据管理与查询

**查询能力：**
- 带自动补全的注册表浏览与查找
- 通过`get()`、`one()`、`one_or_none()`获取单条记录
- 比较运算符过滤（`__gt`、`__lte`、`__contains`、`__startswith`）
- 基于特征的查询（通过标注元数据查询）
- 双下划线语法跨注册表遍历
- 全注册表全文搜索
- 使用Q对象的高级逻辑查询（AND、OR、NOT）
- 免内存加载的大数据集流式处理

**关键工作流：**
- 带过滤和排序的数据对象浏览
- 按特征、创建日期、创建者、大小等查询
- 分块或数组切片流式处理大文件
- 用层级键名组织数据
- 将数据对象分组为集合

**参考文档：** `references/data-management.md` - 阅读此文件获取完整查询模式、过滤示例、流式处理策略及数据组织最佳实践。

### 3. 注释与验证

**管理流程：**
1. **验证**：确认数据集符合目标模式
2. **标准化**：修正拼写错误，将同义词映射至标准术语
3. **注释**：关联数据集与元数据实体以实现可查询

**模式类型：**
- **灵活模式**：仅验证已知列，允许额外元数据
- **最小必需模式**：指定必要列，允许扩展
- **严格模式**：完全控制结构与取值

**支持数据类型：**
- DataFrame（Parquet、CSV）
- AnnData（单细胞基因组学）
- MuData（多模态）
- SpatialData（空间转录组）
- TileDB-SOMA（可扩展数组）

**关键工作流：**
- 定义数据验证的特征与模式
- 使用`DataFrameCurator`或`AnnDataCurator`验证
- 通过`.cat.standardize()`标准化取值
- 用`.cat.add_ontology()`映射至本体论
- 保存带模式关联的已管理数据对象
- 通过特征查询已验证数据集

**参考文档：** `references/annotation-validation.md` - 阅读此文件了解详细管理流程、模式设计模式、验证错误处理及最佳实践。

### 4. 生物本体论

**可用本体论（通过Bionty）：**
- 基因（Ensembl）、蛋白质（UniProt）
- 细胞类型（CL）、细胞系（CLO）
- 组织（Uberon）、疾病（Mondo、DOID）
- 表型（HPO）、通路（GO）
- 实验因子（EFO）、发育阶段
- 生物体（NCBItaxon）、药物（DrugBank）

**关键工作流：**
- 通过`bt.CellType.import_source()`导入公共本体
- 关键词或精确匹配搜索本体
- 使用同义词映射标准化术语
- 探索层级关系（父级、子级、祖先）
- 根据本体术语验证数据
- 用本体记录标注数据集
- 创建自定义术语与层级
- 处理多生物体上下文（人、鼠等）

**参考文档：** `references/ontologies.md` - 阅读此文件获取完整本体操作、标准化策略、层级导航及注释流程。

### 5. 集成扩展

**工作流管理器：**
- Nextflow：追踪流程与输出
- Snakemake：集成至Snakemake规则
- Redun：结合Redun任务追踪

**MLOps平台：**
- Weights & Biases：关联实验与数据对象
- MLflow：追踪模型与实验
- HuggingFace：追踪模型微调
- scVI-tools：单细胞分析工作流

**存储系统：**
- 本地文件系统、AWS S3、Google云存储
- S3兼容存储（MinIO、Cloudflare R2）
- HTTP/HTTPS端点（只读）
- HuggingFace数据集

**数组存储：**
- TileDB-SOMA（支持cellxgene）
- DuckDB用于Parquet文件的SQL查询

**可视化：**
- Vitessce交互式空间/单细胞可视化

**版本控制：**
- Git源代码追踪集成

**参考文档：** `references/integrations.md` - 阅读此文件获取第三方系统集成模式、代码示例及故障排除。

### 6. 安装与部署

**安装：**
- 基础版：`uv pip install lamindb`
- 扩展版：`uv pip install 'lamindb[gcp,zarr,fcs]'`
- 模块：bionty、wetlab、clinical

**实例类型：**
- 本地SQLite（开发环境）
- 云存储+SQLite（小团队）
- 云存储+PostgreSQL（生产环境）

**存储选项：**
- 本地文件系统
- 支持区域与权限配置的AWS S3
- Google云存储
- S3兼容端点（MinIO、Cloudflare R2）

**配置：**
- 云文件缓存管理
- 多用户系统配置
- Git仓库同步
- 环境变量

**部署模式：**
- 本地开发 → 云端生产迁移
- 多区域部署
- 共享存储与个人实例

**参考文档：** `references/setup-deployment.md` - 阅读此文件获取详细安装、配置、存储设置、数据库管理、安全实践及故障排除。

## 典型应用场景

### 场景1：带本体验证的单细胞RNA-seq分析

```python
import lamindb as ln
import bionty as bt
import anndata as ad

# 开始追踪
ln.track(params={"analysis": "scRNA-seq QC and annotation"})

# 导入细胞类型本体
bt.CellType.import_source()

# 加载数据
adata = ad.read_h5ad("raw_counts.h5ad")

# 验证并标准化细胞类型
adata.obs["cell_type"] = bt.CellType.standardize(adata.obs["cell_type"])

# 按模式管理数据
curator = ln.curators.AnnDataCurator(adata, schema)
curator.validate()
artifact = curator.save_artifact(key="scrna/validated.h5ad")

# 关联本体注释
cell_types = bt.CellType.from_values(adata.obs.cell_type)
artifact.feature_sets.add_ontology(cell_types)

ln.finish()
```

### 场景2：构建可查询数据湖仓

```python
import lamindb as ln

# 注册多组实验
for i, file in enumerate(data_files):
    artifact = ln.Artifact.from_anndata(
        ad.read_h5ad(file),
        key=f"scrna/batch_{i}.h5ad",
        description=f"scRNA-seq batch {i}"
    ).save()

    # 添加特征注释
    artifact.features.add_values({
        "batch": i,
        "tissue": tissues[i],
        "condition": conditions[i]
    })

# 跨实验查询
immune_datasets = ln.Artifact.filter(
    key__startswith="scrna/",
    tissue="PBMC",
    condition="treated"
).to_dataframe()

# 加载特定数据集
for artifact in immune_datasets:
    adata = artifact.load()
    # 执行分析
```

### 场景3：集成W&B的ML流程

```python
import lamindb as ln
import wandb

# 初始化系统
wandb.init(project="drug-response", name="exp-42")
ln.track(params={"model": "random_forest", "n_estimators": 100})

# 从LaminDB加载训练数据
train_artifact = ln.Artifact.get(key="datasets/train.parquet")
train_data = train_artifact.load()

# 训练模型
model = train_model(train_data)

# 记录至W&B
wandb.log({"accuracy": 0.95})

# 保存模型并关联W&B
import joblib
joblib.dump(model, "model.pkl")
model_artifact = ln.Artifact("model.pkl", key="models/exp-42.pkl").save()
model_artifact.features.add_values({"wandb_run_id": wandb.run.id})

ln.finish()
wandb.finish()
```

### 场景4：Nextflow流程集成

```python
# Nextflow流程脚本中
import lamindb as ln

ln.track()

# 加载输入数据对象
input_artifact = ln.Artifact.get(key="raw/batch_${batch_id}.fastq.gz")
input_path = input_artifact.cache()

# 执行处理（比对、定量等）
# ... Nextflow流程逻辑 ...

# 保存输出
output_artifact = ln.Artifact(
    "counts.csv",
    key="processed/batch_${batch_id}_counts.csv"
).save()

ln.finish()
```

## 快速入门清单

高效使用LaminDB的步骤：

1. **安装与配置** (`references/setup-deployment.md`)
   - 安装LaminDB及所需扩展
   - 通过`lamin login`认证
   - 使用`lamin init --storage ...`初始化实例

2. **学习核心概念** (`references/core-concepts.md`)
   - 理解数据对象、记录、运行、转换
   - 练习创建和检索数据对象
   - 在工作流中实现`ln.track()`和`ln.finish()`

3. **掌握查询** (`references/data-management.md`)
   - 练习注册表过滤与搜索
   - 学习基于特征的查询
   - 尝试大文件流式处理

4. **设置验证** (`references/annotation-validation.md`)
   - 定义研究领域相关特征
   - 为数据类型创建模式
   - 练习数据管理工作流

5. **集成本体论** (`references/ontologies.md`)
   - 导入相关生物本体（基因、细胞类型等）
   - 验证现有注释
   - 用本体术语标准化元数据

6. **连接工具** (`references/integrations.md`)
   - 集成现有工作流管理器
   - 关联ML平台实现实验追踪
   - 配置云存储与计算资源

## 核心原则

使用LaminDB时遵循以下原则：

1. **全量追踪**：每个分析开始时使用`ln.track()`自动捕获溯源
2. **及早验证**：在深入分析前定义模式并验证数据
3. **善用本体**：利用公共生物本体实现标准化注释
4. **键名组织**：使用层级结构组织数据对象键名（如`项目/实验/批次/文件.h5ad`）
5. **元数据优先**：加载大文件前先过滤搜索
6. **版本替代复制**：使用内置版本控制而非为新修改创建新键名
7. **特征注释**：定义类型化特征实现可查询元数据
8. **详尽文档**：为数据对象、模式和转换添加描述
9. **溯源溯源**：通过`view_lineage()`理解数据来源
10. **本地启程云端扩展**：用SQLite本地开发，通过PostgreSQL部署至云端

## 参考文档

本技能包含按能力模块组织的完整参考文档：

- **`references/core-concepts.md`** - 数据对象、记录、运行、转换、特征、版本控制、溯源
- **`references/data-management.md`** - 查询、过滤、搜索、流式处理、数据组织
- **`references/annotation-validation.md`** -
