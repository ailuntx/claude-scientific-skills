# LaminDB 本体管理

本文档介绍通过 Bionty 插件在 LaminDB 中管理生物本体的方法，包括访问、搜索和使用标准化生物术语进行数据标注。

## 概述

LaminDB 集成 `bionty` 插件管理标准化生物本体，实现跨研究项目的一致元数据管理和数据标注。Bionty 提供 20+ 个精选生物本体，涵盖基因、蛋白质、细胞类型、组织、疾病等领域。

## 可用本体

LaminDB 提供多个精选本体源：

| 注册表 | 本体来源 | 描述 |
|----------|----------------|-------------|
| **基因** | Ensembl | 跨物种基因（人类、小鼠等） |
| **蛋白质** | UniProt | 蛋白质序列与注释 |
| **细胞类型** | 细胞本体 (CL) | 标准化细胞类型分类 |
| **细胞系** | 细胞系本体 (CLO) | 细胞系注释 |
| **组织** | Uberon | 解剖结构与组织 |
| **疾病** | Mondo, DOID | 疾病分类 |
| **表型** | 人类表型本体 (HPO) | 表型异常 |
| **通路** | 基因本体 (GO) | 生物通路与过程 |
| **实验因子** | 实验因子本体 (EFO) | 实验变量 |
| **发育阶段** | 多源 | 跨物种发育阶段 |
| **种族** | HANCESTRO | 人类血统本体 |
| **药物** | DrugBank | 药物化合物 |
| **生物体** | NCBItaxon | 分类学分类 |

## 安装与导入

```python
# 安装 bionty（包含在 lamindb 中）
pip install lamindb

# 导入
import lamindb as ln
import bionty as bt
```

## 导入公共本体

将公共本体源填充到注册表：

```python
# 导入细胞本体
bt.CellType.import_source()

# 导入物种特异性基因
bt.Gene.import_source(organism="human")
bt.Gene.import_source(organism="mouse")

# 导入组织
bt.Tissue.import_source()

# 导入疾病
bt.Disease.import_source(source="mondo")  # Mondo 疾病本体
bt.Disease.import_source(source="doid")   # 疾病本体
```

## 搜索与访问记录

### 关键词搜索

```python
# 搜索细胞类型
bt.CellType.search("T cell").to_dataframe()
bt.CellType.search("gamma-delta").to_dataframe()

# 搜索基因
bt.Gene.search("CD8").to_dataframe()
bt.Gene.search("TP53").to_dataframe()

# 搜索疾病
bt.Disease.search("cancer").to_dataframe()

# 搜索组织
bt.Tissue.search("brain").to_dataframe()
```

### 自动补全查询

适用于记录数少于 10 万的注册表：

```python
# 创建查询对象
cell_types = bt.CellType.lookup()

# 按名称访问（IDE 中支持自动补全）
t_cell = cell_types.t_cell
hsc = cell_types.hematopoietic_stem_cell

# 其他注册表同理
genes = bt.Gene.lookup()
cd8a = genes.cd8a
```

### 精确字段匹配

```python
# 通过本体 ID
cell_type = bt.CellType.get(ontology_id="CL:0000798")
disease = bt.Disease.get(ontology_id="MONDO:0004992")

# 通过名称
cell_type = bt.CellType.get(name="T cell")
gene = bt.Gene.get(symbol="CD8A")

# 通过 Ensembl ID
gene = bt.Gene.get(ensembl_gene_id="ENSG00000153563")
```

## 本体层次结构

### 探索关系

```python
# 获取细胞类型
gdt_cell = bt.CellType.get(ontology_id="CL:0000798")  # γδ T 细胞

# 查看直接父节点
gdt_cell.parents.to_dataframe()

# 递归查看所有祖先
ancestors = []
current = gdt_cell
while current.parents.exists():
    parent = current.parents.first()
    ancestors.append(parent)
    current = parent

# 查看直接子节点
gdt_cell.children.to_dataframe()

# 递归查看所有后代
gdt_cell.query_children().to_dataframe()
```

### 可视化层次结构

```python
# 可视化父系层次
gdt_cell.view_parents()

# 包含子节点可视化
gdt_cell.view_parents(with_children=True)

# 获取所有相关术语进行可视化
t_cell = bt.CellType.get(name="T cell")
t_cell.view_parents(with_children=True)  # 显示 T 细胞亚型
```

## 标准化与验证数据

### 验证

检查术语是否存在于本体中：

```python
# 验证细胞类型
bt.CellType.validate(["T cell", "B cell", "invalid_cell"])
# 返回: [True, True, False]

# 验证基因
bt.Gene.validate(["CD8A", "TP53", "FAKEGENE"], organism="human")
# 返回: [True, True, False]

# 识别无效术语
terms = ["T cell", "fat cell", "neuron", "invalid_term"]
invalid = [t for t, valid in zip(terms, bt.CellType.validate(terms)) if not valid]
print(f"无效术语: {invalid}")
```

### 使用同义词标准化

将非标准术语转换为验证名称：

```python
# 标准化细胞类型名称
bt.CellType.standardize(["fat cell", "blood forming stem cell"])
# 返回: ['adipocyte', 'hematopoietic stem cell']

# 标准化基因
bt.Gene.standardize(["BRCA-1", "p53"], organism="human")
# 返回: ['BRCA1', 'TP53']

# 处理混合有效/无效术语
terms = ["T cell", "T lymphocyte", "invalid"]
standardized = bt.CellType.standardize(terms)
# 返回可能的标准化名称
```

### 加载已验证记录

```python
# 从值加载记录（包含同义词）
records = bt.CellType.from_values(["fat cell", "blood forming stem cell"])

# 返回 CellType 记录列表
for record in records:
    print(record.name, record.ontology_id)

# 使用基因符号
genes = bt.Gene.from_values(["CD8A", "CD8B"], organism="human")
```

## 标注数据集

### 标注 AnnData

```python
import anndata as ad
import lamindb as ln

# 加载示例数据
adata = ad.read_h5ad("data.h5ad")

# 验证并获取匹配记录
cell_types = bt.CellType.from_values(adata.obs.cell_type)

# 创建带标注的数据对象
artifact = ln.Artifact.from_anndata(
    adata,
    key="scrna/annotated_data.h5ad",
    description="带验证细胞类型标注的单细胞RNA-seq数据"
).save()

# 将本体记录链接到数据对象
artifact.feature_sets.add_ontology(cell_types)
```

### 标注 DataFrame

```python
import pandas as pd

# 创建含生物实体的 DataFrame
df = pd.DataFrame({
    "cell_type": ["T cell", "B cell", "NK cell"],
    "tissue": ["blood", "spleen", "liver"],
    "disease": ["healthy", "lymphoma", "healthy"]
})

# 验证与标准化
df["cell_type"] = bt.CellType.standardize(df["cell_type"])
df["tissue"] = bt.Tissue.standardize(df["tissue"])

# 创建数据对象
artifact = ln.Artifact.from_dataframe(
    df,
    key="metadata/sample_info.parquet"
).save()

# 链接本体记录
cell_type_records = bt.CellType.from_values(df["cell_type"])
tissue_records = bt.Tissue.from_values(df["tissue"])

artifact.feature_sets.add_ontology(cell_type_records)
artifact.feature_sets.add_ontology(tissue_records)
```

## 管理自定义术语与层次结构

### 添加自定义术语

```python
# 注册公共本体中不存在的新术语
my_celltype = bt.CellType(name="my_novel_T_cell_subtype").save()

# 建立父子关系
parent = bt.CellType.get(name="T cell")
my_celltype.parents.add(parent)

# 验证关系
my_celltype.parents.to_dataframe()
parent.children.to_dataframe()  # 应包含 my_celltype
```

### 添加同义词

```python
# 添加标准化同义词
hsc = bt.CellType.get(name="hematopoietic stem cell")
hsc.add_synonym("HSC")
hsc.add_synonym("blood stem cell")
hsc.add_synonym("hematopoietic progenitor")

# 设置缩写
hsc.set_abbr("HSC")

# 同义词标准化生效
bt.CellType.standardize(["HSC", "blood stem cell"])
# 返回: ['hematopoietic stem cell', 'hematopoietic stem cell']
```

### 创建自定义层次结构

```python
# 构建自定义细胞类型层次
immune_cell = bt.CellType.get(name="immune cell")

# 添加自定义亚型
my_subtype1 = bt.CellType(name="custom_immune_subtype_1").save()
my_subtype2 = bt.CellType(name="custom_immune_subtype_2").save()

# 链接到父节点
my_subtype1.parents.add(immune_cell)
my_subtype2.parents.add(immune_cell)

# 创建子亚型
my_subsubtype = bt.CellType(name="custom_sub_subtype").save()
my_subsubtype.parents.add(my_subtype1)

# 可视化自定义层次
immune_cell.view_parents(with_children=True)
```

## 多物种支持

适用于基因等物种感知注册表：

```python
# 设置全局物种
bt.settings.organism = "human"

# 验证人类基因
bt.Gene.validate(["TCF7", "CD8A"], organism="human")

# 加载特定物种基因
human_genes = bt.Gene.from_values(["CD8A", "TP53"], organism="human")
mouse_genes = bt.Gene.from_values(["Cd8a", "Trp53"], organism="mouse")

# 搜索物种特异性基因
bt.Gene.search("CD8", organism="human").to_dataframe()
bt.Gene.search("Cd8", organism="mouse").to_dataframe()

# 切换物种上下文
bt.settings.organism = "mouse"
genes = bt.Gene.from_source(symbol="Ap5b1")
```

## 公共本体查询

无需导入即可访问公共本体术语：

```python
# 在公共源中交互查询
cell_types_public = bt.CellType.lookup(public=True)

# 访问公共术语
hepatocyte = cell_types_public.hepatocyte

# 导入特定术语
hepatocyte_local = bt.CellType.from_source(name="hepatocyte")

# 或通过本体 ID 导入
specific_cell = bt.CellType.from_source(ontology_id="CL:0000182")
```

## 版本追踪

LaminDB 自动追踪本体版本：

```python
# 查看当前源版本
bt.Source.filter(currently_used=True).to_dataframe()

# 检查记录来源
cell_type = bt.CellType.get(name="hepatocyte")
cell_type.source  # 返回源元数据

# 查看源详情
source = cell_type.source
print(source.name)        # 如 "cl"
print(source.version)     # 如 "2023-05-18"
print(source.url)         # 本体 URL
```

## 本体集成工作流

### 工作流 1：验证现有数据

```python
# 加载含生物标注的数据
adata = ad.read_h5ad("uncurated_data.h5ad")

# 验证细胞类型
validation = bt.CellType.validate(adata.obs.cell_type)

# 识别无效术语
invalid_idx = [i for i, v in enumerate(validation) if not v]
invalid_terms = adata.obs.cell_type.iloc[invalid_idx].unique()
print(f"无效细胞类型: {invalid_terms}")

# 手动或通过标准化修复
adata.obs["cell_type"] = bt.CellType.standardize(adata.obs.cell_type)

# 重新验证
validation = bt.CellType.validate(adata.obs.cell_type)
assert all(validation), "所有术语应已有效"
```

### 工作流 2：管理与标注

```python
import lamindb as ln

ln.track()  # 开始追踪

# 加载数据
df = pd.read_csv("experimental_data.csv")

# 使用本体标准化
df["cell_type"] = bt.CellType.standardize(df["cell_type"])
df["tissue"] = bt.Tissue.standardize(df["tissue"])

# 创建管理后的数据对象
artifact = ln.Artifact.from_dataframe(
    df,
    key="curated/experiment_2025_10.parquet",
    description="带本体验证标注的管理后实验数据"
).save()

# 链接本体记录
artifact.feature_sets.add_ontology(bt.CellType.from_values(df["cell_type"]))
artifact.feature_sets.add_ontology(bt.Tissue.from_values(df["tissue"]))

ln.finish()  # 完成追踪
```

### 工作流 3：跨物种基因映射

```python
# 获取人类基因
human_genes = ["CD8A", "CD8B", "TP53"]
human_records = bt.Gene.from_values(human_genes, organism="human")

# 查找小鼠直系同源基因（需外部映射）
# LaminDB 不提供内置直系同源映射
# 使用 Ensembl BioMart 或 homologene 等工具

mouse_orthologs = ["Cd8a", "Cd8b", "Trp53"]
mouse_records = bt.Gene.from_values(mouse_orthologs, organism="mouse")
```

## 查询本体标注数据

```python
# 查找含特定细胞类型的所有数据集
t_cell = bt.CellType.get(name="T cell")
ln.Artifact.filter(feature_sets__cell_types=t_cell).to_dataframe()

# 查找测量特定基因的数据集
cd8a = bt.Gene.get(symbol="CD8A", organism="human")
ln.Artifact.filter(feature_sets__genes=cd8a).to_dataframe()

# 跨本体层次查询
# 查找含 T 细胞或其亚型的所有数据集
t_cell_subtypes = t_cell.query_children()
ln.Artifact.filter(
    feature_sets__cell_types__in=t_cell_subtypes
).to_dataframe()
```

## 最佳实践

1. **优先导入本体**：在验证前调用 `import_source()`
2. **使用标准化**：利用同义词映射处理变体
3. **及早验证**：创建数据对象前检查术语
4. **设置物种上下文**：基因相关查询指定物种
5. **添加自定义同义词**：注册领域内常用变体
6. **使用公共查询**：通过 `lookup(public=True)` 探索术语
7. **追踪版本**：监控本体源版本确保可复现性
8. **构建层次结构**：将自定义术语链接到现有本体结构
9. **层次化查询**：使用 `query_children()` 进行综合搜索
10. **记录映射关系**：追踪自定义术语添加和关联

## 常用本体操作

```python
# 检查术语是否存在
exists = bt.CellType.filter(name="T cell").exists()

# 统计注册表术语数
n_cell_types = bt.CellType.filter().count()

# 获取特定父节点的所有术语
immune_cells = bt.CellType.filter(parents__name="immune cell")

# 查找孤立术语（无父节点）
orphans = bt.CellType.filter(parents__isnull=True)

# 获取最近添加的术语
from datetime import datetime, timedelta
recent = bt.CellType.filter(
    created_at__gte=datetime.now() - timedelta(days=7)
)
```
