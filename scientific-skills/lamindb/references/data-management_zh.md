# LaminDB 数据管理

本文档涵盖 LaminDB 中的数据查询、搜索、筛选和流式处理，以及组织和访问数据集的最佳实践。

## 注册表概览

查看可用注册表及其内容：

```python
import lamindb as ln

# 查看跨模块的所有注册表
ln.view()

# 查看最新的100个工件
ln.Artifact.to_dataframe()

# 查看其他注册表
ln.Transform.to_dataframe()
ln.Run.to_dataframe()
ln.User.to_dataframe()
```

## 快速访问查询

对于记录数少于10万的注册表，`Lookup`对象支持便捷的自动补全：

```python
# 创建查询对象
records = ln.Record.lookup()

# 按名称访问（IDE中启用自动补全）
experiment_1 = records.experiment_1
sample_a = records.sample_a

# 同样适用于生物本体
import bionty as bt
cell_types = bt.CellType.lookup()
t_cell = cell_types.t_cell
```

## 检索单条记录

### 使用 get()

精确检索单条记录（无匹配或多条匹配时报错）：

```python
# 通过UID
artifact = ln.Artifact.get("aRt1Fact0uid000")

# 通过字段
artifact = ln.Artifact.get(key="data/experiment.h5ad")
user = ln.User.get(handle="researcher123")

# 通过本体ID（bionty）
cell_type = bt.CellType.get(ontology_id="CL:0000084")
```

### 使用 one() 和 one_or_none()

```python
# 从QuerySet获取唯一记录（0条或多条时报错）
artifact = ln.Artifact.filter(key="data.csv").one()

# 获取单条或None（多条时报错）
artifact = ln.Artifact.filter(key="maybe_data.csv").one_or_none()

# 获取首条匹配记录
artifact = ln.Artifact.filter(suffix=".h5ad").first()
```

## 数据筛选

`filter()`方法返回支持灵活检索的QuerySet：

```python
# 基础筛选
artifacts = ln.Artifact.filter(suffix=".h5ad")
artifacts.to_dataframe()

# 多条件筛选（AND逻辑）
artifacts = ln.Artifact.filter(
    suffix=".h5ad",
    created_by=user
)

# 比较运算符
ln.Artifact.filter(size__gt=1e6).to_dataframe()           # 大于
ln.Artifact.filter(size__gte=1e6).to_dataframe()          # 大于等于
ln.Artifact.filter(size__lt=1e9).to_dataframe()           # 小于
ln.Artifact.filter(size__lte=1e9).to_dataframe()          # 小于等于

# 范围查询
ln.Artifact.filter(size__gte=1e6, size__lte=1e9).to_dataframe()
```

## 文本与字符串查询

```python
# 精确匹配
ln.Artifact.filter(description="实验1").to_dataframe()

# 包含（区分大小写）
ln.Artifact.filter(description__contains="RNA").to_dataframe()

# 不区分大小写包含
ln.Artifact.filter(description__icontains="rna").to_dataframe()

# 开头匹配
ln.Artifact.filter(key__startswith="experiments/").to_dataframe()

# 结尾匹配
ln.Artifact.filter(key__endswith=".csv").to_dataframe()

# IN列表查询
ln.Artifact.filter(suffix__in=[".h5ad", ".csv", ".parquet"]).to_dataframe()
```

## 基于特征的查询

通过标注特征查询工件：

```python
# 按特征值筛选
ln.Artifact.filter(cell_type="T细胞").to_dataframe()
ln.Artifact.filter(treatment="DMSO").to_dataframe()

# 输出包含特征
ln.Artifact.filter(treatment="DMSO").to_dataframe(include="features")

# 嵌套字典访问
ln.Artifact.filter(study_metadata__assay="RNA测序").to_dataframe()
ln.Artifact.filter(study_metadata__detail1="123").to_dataframe()

# 检查标注状态
ln.Artifact.filter(cell_type__isnull=False).to_dataframe()  # 存在标注
ln.Artifact.filter(treatment__isnull=True).to_dataframe()    # 缺失标注
```

## 跨关联注册表查询

使用Django双下划线语法实现跨表查询：

```python
# 按创建者句柄查找工件
ln.Artifact.filter(created_by__handle="researcher123").to_dataframe()
ln.Artifact.filter(created_by__handle__startswith="test").to_dataframe()

# 按转换名称查找工件
ln.Artifact.filter(transform__name="preprocess.py").to_dataframe()

# 查找测量特定基因的工件
ln.Artifact.filter(feature_sets__genes__symbol="CD8A").to_dataframe()
ln.Artifact.filter(feature_sets__genes__ensembl_gene_id="ENSG00000153563").to_dataframe()

# 查找含特定参数的运行
ln.Run.filter(params__learning_rate=0.01).to_dataframe()
ln.Run.filter(params__downsample=True).to_dataframe()

# 查找特定项目中的工件
project = ln.Project.get(name="癌症研究")
ln.Artifact.filter(projects=project).to_dataframe()
```

## 结果排序

```python
# 按字段升序排序
ln.Artifact.filter(suffix=".h5ad").order_by("created_at").to_dataframe()

# 降序排序
ln.Artifact.filter(suffix=".h5ad").order_by("-created_at").to_dataframe()

# 多字段排序
ln.Artifact.order_by("-created_at", "size").to_dataframe()
```

## 高级逻辑查询

### OR逻辑

```python
from lamindb import Q

# OR条件
artifacts = ln.Artifact.filter(
    Q(suffix=".jpg") | Q(suffix=".png")
).to_dataframe()

# 含多条件的复杂OR
artifacts = ln.Artifact.filter(
    Q(suffix=".h5ad", size__gt=1e6) | Q(suffix=".csv", size__lt=1e3)
).to_dataframe()
```

### NOT逻辑

```python
# 排除条件
artifacts = ln.Artifact.filter(
    ~Q(suffix=".tmp")
).to_dataframe()

# 复杂排除
artifacts = ln.Artifact.filter(
    ~Q(created_by__handle="testuser")
).to_dataframe()
```

### 组合AND/OR/NOT

```python
# 复杂查询
artifacts = ln.Artifact.filter(
    (Q(suffix=".h5ad") | Q(suffix=".csv")) &
    Q(size__gt=1e6) &
    ~Q(created_by__handle__startswith="test")
).to_dataframe()
```

## 搜索功能

跨注册表字段的全文搜索：

```python
# 基础搜索
ln.Artifact.search("iris").to_dataframe()
ln.User.search("smith").to_dataframe()

# 特定注册表内搜索
bt.CellType.search("T细胞").to_dataframe()
bt.Gene.search("CD8").to_dataframe()
```

## 使用QuerySet

QuerySet具有惰性特性——仅在求值时访问数据库：

```python
# 创建查询（不访问数据库）
qs = ln.Artifact.filter(suffix=".h5ad")

# 多种求值方式
df = qs.to_dataframe()        # 转为pandas DataFrame
list_records = list(qs)       # 转为Python列表
count = qs.count()            # 仅计数
exists = qs.exists()          # 布尔检查

# 迭代访问
for artifact in qs:
    print(artifact.key, artifact.size)

# 切片访问
first_10 = qs[:10]
next_10 = qs[10:20]
```

## 链式筛选

```python
# 增量构建查询
qs = ln.Artifact.filter(suffix=".h5ad")
qs = qs.filter(size__gt=1e6)
qs = qs.filter(created_at__year=2025)
qs = qs.order_by("-created_at")

# 执行查询
results = qs.to_dataframe()
```

## 流式处理大型数据集

对于内存无法容纳的大型数据集，使用流式访问：

### 文件流处理

```python
# 打开文件流
artifact = ln.Artifact.get(key="large_file.csv")

with artifact.open() as f:
    # 分块读取
    chunk = f.read(10000)  # 读取10KB
    # 处理数据块
```

### 数组切片

适用于基于数组的格式（Zarr, HDF5, AnnData）：

```python
# 获取底层文件而不加载
artifact = ln.Artifact.get(key="large_data.h5ad")
adata = artifact.backed()  # 返回backed AnnData

# 切片特定部分
subset = adata[:1000, :]  # 前1000个细胞
genes_of_interest = adata[:, ["CD4", "CD8A", "CD8B"]]

# 流式批处理
for i in range(0, adata.n_obs, 1000):
    batch = adata[i:i+1000, :]
    # 处理批次数据
```

### 迭代器访问

```python
# 增量处理大型集合
artifacts = ln.Artifact.filter(suffix=".fastq.gz")

for artifact in artifacts.iterator(chunk_size=10):
    # 每次处理10个
    path = artifact.cache()
    # 分析文件
```

## 聚合与统计

```python
# 记录计数
ln.Artifact.filter(suffix=".h5ad").count()

# 去重值
ln.Artifact.values_list("suffix", flat=True).distinct()

# 聚合（需Django ORM知识）
from django.db.models import Sum, Avg, Max, Min

# 所有工件的总大小
ln.Artifact.aggregate(Sum("size"))

# 按后缀统计平均工件大小
ln.Artifact.values("suffix").annotate(avg_size=Avg("size"))
```

## 缓存与性能优化

```python
# 检查缓存位置
ln.settings.cache_dir

# 配置缓存
lamin cache set /path/to/cache

# 清除特定工件缓存
artifact.delete_cache()

# 获取缓存路径（需要时下载）
path = artifact.cache()

# 检查是否已缓存
if artifact.is_cached():
    path = artifact.cache()
```

## 键名组织数据

键名结构的最佳实践：

```python
# 层级化组织
ln.Artifact("data.h5ad", key="project/experiment/batch1/data.h5ad").save()
ln.Artifact("data.h5ad", key="scrna/2025/oct/sample_001.h5ad").save()

# 按前缀浏览
ln.Artifact.filter(key__startswith="scrna/2025/oct/").to_dataframe()

# 键名包含版本（替代内置版本控制）
ln.Artifact("data.h5ad", key="data/processed/v1/final.h5ad").save()
ln.Artifact("data.h5ad", key="data/processed/v2/final.h5ad").save()
```

## 集合

将相关工件分组为集合：

```python
# 创建集合
collection = ln.Collection(
    [artifact1, artifact2, artifact3],
    name="scRNA-seq批次1-3",
    description="跨三个批次的完整数据集"
).save()

# 访问集合成员
for artifact in collection.artifacts:
    print(artifact.key)

# 查询集合
ln.Collection.filter(name__contains="batch").to_dataframe()
```

## 最佳实践

1. **加载前先筛选**：访问文件内容前先查询元数据
2. **善用QuerySet**：增量构建复杂条件查询
3. **流式处理大文件**：避免不必要地全量加载数据集
4. **层级化组织键名**：便于浏览和筛选
5. **利用搜索功能**：当不确定具体字段值时使用
6. **策略性缓存**：根据存储容量配置缓存位置
7. **建立特征索引**：预先定义特征以实现高效查询
8. **使用集合**：分组相关工件进行数据集级操作
9. **结果排序**：按创建日期或其他字段排序保证检索一致性
10. **检查存在性**：使用`exists()`或`one_or_none()`避免错误

## 常用查询模式

```python
# 近期工件
ln.Artifact.order_by("-created_at")[:10].to_dataframe()

# 我的工件
me = ln.setup.settings.user
ln.Artifact.filter(created_by=me).to_dataframe()

# 大文件
ln.Artifact.filter(size__gt=1e9).order_by("-size").to_dataframe()

# 本月数据
from datetime import datetime
ln.Artifact.filter(
    created_at__year=2025,
    created_at__month=10
).to_dataframe()

# 含特定特征的已验证数据集
ln.Artifact.filter(
    is_valid=True,
    cell_type__isnull=False
).to_dataframe(include="features")
```
