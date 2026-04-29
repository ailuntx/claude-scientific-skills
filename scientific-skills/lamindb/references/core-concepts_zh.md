# LaminDB 核心概念

本文档涵盖 LaminDB 的基本概念和构建模块：工件（Artifacts）、记录（Records）、运行（Runs）、转换（Transforms）、特征（Features）以及数据谱系跟踪。

## 工件（Artifacts）

工件代表各种格式的数据集（DataFrame、AnnData、SpatialData、Parquet、Zarr 等）。它们是 LaminDB 中的主要数据对象。

### 创建与保存工件

**从文件创建：**
```python
import lamindb as ln

# 将文件保存为工件
ln.Artifact("sample.fasta", key="sample.fasta").save()

# 添加描述信息
artifact = ln.Artifact(
    "data/analysis.h5ad",
    key="experiments/scrna_batch1.h5ad",
    description="单细胞RNA-seq批次1"
).save()
```

**从DataFrame创建：**
```python
import pandas as pd

df = pd.read_csv("data.csv")
artifact = ln.Artifact.from_dataframe(
    df,
    key="datasets/processed_data.parquet",
    description="处理后的实验数据"
).save()
```

**从AnnData创建：**
```python
import anndata as ad

adata = ad.read_h5ad("data.h5ad")
artifact = ln.Artifact.from_anndata(
    adata,
    key="scrna/experiment1.h5ad",
    description="经过质控的scRNA-seq数据"
).save()
```

### 检索工件

```python
# 通过键名检索
artifact = ln.Artifact.get(key="sample.fasta")

# 通过UID检索
artifact = ln.Artifact.get("aRt1Fact0uid000")

# 通过过滤器检索
artifact = ln.Artifact.filter(suffix=".h5ad").first()
```

### 访问工件内容

```python
# 获取缓存的本地路径
local_path = artifact.cache()

# 加载到内存
data = artifact.load()  # 返回DataFrame、AnnData等对象

# 流式访问（适用于大文件）
with artifact.open() as f:
    # 增量读取
    chunk = f.read(1000)
```

### 工件元数据

```python
# 查看所有元数据
artifact.describe()

# 访问特定元数据
artifact.size          # 文件大小（字节）
artifact.suffix        # 文件扩展名
artifact.created_at    # 创建时间戳
artifact.created_by    # 创建者
artifact.run          # 关联的运行
artifact.transform    # 关联的转换
artifact.version      # 版本字符串
```

## 记录（Records）

记录代表实验实体：样本、处理条件、仪器、细胞系等任何元数据实体。通过类型定义支持层次化关系。

### 创建记录

```python
# 定义类型
sample_type = ln.Record(name="Sample", is_type=True).save()

# 创建该类型的实例
ln.Record(name="P53突变体1", type=sample_type).save()
ln.Record(name="P53突变体2", type=sample_type).save()
ln.Record(name="野生型对照", type=sample_type).save()
```

### 搜索记录

```python
# 文本搜索
ln.Record.search("p53").to_dataframe()

# 按字段过滤
ln.Record.filter(type=sample_type).to_dataframe()

# 获取特定记录
record = ln.Record.get(name="P53突变体1")
```

### 层次化关系

```python
# 建立父子关系
parent_record = ln.Record.get(name="P53突变体1")
child_record = ln.Record(name="P53突变体1-重复1", type=sample_type).save()
child_record.parents.add(parent_record)

# 查询关系
parent_record.children.to_dataframe()
child_record.parents.to_dataframe()
```

## 运行（Runs）与转换（Transforms）

这些模块捕获计算谱系。**转换（Transform）** 代表可复用的分析步骤（笔记本、脚本或函数），而**运行（Run）** 记录具体的执行实例。

### 基础跟踪流程

```python
import lamindb as ln

# 开始跟踪（在笔记本/脚本开头）
ln.track()

# 分析代码
data = ln.Artifact.get(key="input.csv").load()
# ...执行分析...
result.to_csv("output.csv")
artifact = ln.Artifact("output.csv", key="output.csv").save()

# 结束跟踪（在笔记本/脚本结尾）
ln.finish()
```

### 带参数的跟踪

```python
ln.track(params={
    "learning_rate": 0.01,
    "batch_size": 32,
    "epochs": 100,
    "downsample": True
})

# 按参数查询运行
ln.Run.filter(params__learning_rate=0.01).to_dataframe()
ln.Run.filter(params__downsample=True).to_dataframe()
```

### 带项目的跟踪

```python
# 关联项目
ln.track(project="2025年抗癌药物筛选")

# 按项目查询
project = ln.Project.get(name="2025年抗癌药物筛选")
ln.Artifact.filter(projects=project).to_dataframe()
ln.Run.filter(project=project).to_dataframe()
```

### 函数级跟踪

使用 `@ln.tracked()` 装饰器实现细粒度谱系跟踪：

```python
@ln.tracked()
def preprocess_data(input_key: str, output_key: str, normalize: bool = True) -> None:
    """预处理原始数据并保存结果"""
    # 加载输入（自动跟踪）
    artifact = ln.Artifact.get(key=input_key)
    data = artifact.load()

    # 处理过程
    if normalize:
        data = (data - data.mean()) / data.std()

    # 保存输出（自动跟踪）
    ln.Artifact.from_dataframe(data, key=output_key).save()

# 每次调用创建独立的转换和运行
preprocess_data("raw/batch1.csv", "processed/batch1.csv", normalize=True)
preprocess_data("raw/batch2.csv", "processed/batch2.csv", normalize=False)
```

### 访问谱系信息

```python
# 从工件追溯到运行
artifact = ln.Artifact.get(key="output.csv")
run = artifact.run
transform = run.transform

# 查看详情
run.describe()          # 运行元数据
transform.describe()    # 转换元数据

# 访问输入项
run.inputs.to_dataframe()

# 可视化谱系图
artifact.view_lineage()
```

## 特征（Features）

特征定义类型化的元数据字段，用于验证和查询。支持结构化标注和搜索。

### 定义特征

```python
from datetime import date

# 数值型特征
ln.Feature(name="gc_content", dtype=float).save()
ln.Feature(name="read_count", dtype=int).save()

# 日期型特征
ln.Feature(name="experiment_date", dtype=date).save()

# 分类型特征
ln.Feature(name="cell_type", dtype=str).save()
ln.Feature(name="treatment", dtype=str).save()
```

### 为工件标注特征

```python
# 单值标注
artifact.features.add_values({
    "gc_content": 0.55,
    "experiment_date": "2025-10-31"
})

# 使用特征注册记录
gc_content_feature = ln.Feature.get(name="gc_content")
artifact.features.add(gc_content_feature)
```

### 按特征查询

```python
# 按特征值过滤
ln.Artifact.filter(gc_content=0.55).to_dataframe()
ln.Artifact.filter(experiment_date="2025-10-31").to_dataframe()

# 比较运算符
ln.Artifact.filter(read_count__gt=1000000).to_dataframe()
ln.Artifact.filter(gc_content__gte=0.5, gc_content__lte=0.6).to_dataframe()

# 检查是否存在标注
ln.Artifact.filter(cell_type__isnull=False).to_dataframe()

# 在输出中包含特征
ln.Artifact.filter(treatment="DMSO").to_dataframe(include="features")
```

### 嵌套字典特征

适用于存储复杂元数据的字典结构：

```python
# 访问嵌套值
ln.Artifact.filter(study_metadata__detail1="123").to_dataframe()
ln.Artifact.filter(study_metadata__assay__type="RNA-seq").to_dataframe()
```

## 数据谱系跟踪

LaminDB 自动捕获执行上下文及数据、代码与运行间的关系。

### 跟踪内容

- **源代码**：脚本/笔记本内容及git提交记录
- **环境**：Python包及版本
- **输入工件**：执行期间加载的数据
- **输出工件**：执行期间创建的数据
- **执行元数据**：时间戳、用户、参数
- **计算依赖**：转换关系

### 查看谱系

```python
# 可视化完整谱系图
artifact.view_lineage()

# 查看捕获的元数据
artifact.describe()

# 访问相关实体
artifact.run              # 创建该工件的运行
artifact.run.transform    # 使用的转换（代码）
artifact.run.inputs       # 输入工件
artifact.run.report       # 执行报告
```

### 谱系查询

```python
# 查找转换的所有输出
transform = ln.Transform.get(name="preprocessing.py")
ln.Artifact.filter(transform=transform).to_dataframe()

# 查找特定用户的所有工件
user = ln.User.get(handle="researcher123")
ln.Artifact.filter(created_by=user).to_dataframe()

# 查找使用特定输入的工件
input_artifact = ln.Artifact.get(key="raw/data.csv")
runs = ln.Run.filter(inputs=input_artifact)
ln.Artifact.filter(run__in=runs).to_dataframe()
```

## 版本控制

当源数据或代码变更时，LaminDB 自动管理工件版本。

### 自动版本控制

```python
# 第一版
artifact_v1 = ln.Artifact("data.csv", key="experiment/data.csv").save()

# 修改后再次保存 - 创建新版本
# （修改data.csv）
artifact_v2 = ln.Artifact("data.csv", key="experiment/data.csv").save()
```

### 版本操作

```python
# 获取最新版本（默认）
artifact = ln.Artifact.get(key="experiment/data.csv")

# 查看所有版本
artifact.versions.to_dataframe()

# 获取特定版本
artifact_v1 = artifact.versions.filter(version="1").first()

# 比较版本
v1_data = artifact_v1.load()
v2_data = artifact.load()
```

## 最佳实践

1. **使用有意义的键名**：采用层次化结构（如 `项目/实验/样本.h5ad`）
2. **添加描述信息**：帮助未来用户理解工件内容
3. **保持跟踪一致性**：在每次分析开始时调用 `ln.track()`
4. **预先定义特征**：在标注前创建特征注册表
5. **使用类型化特征**：指定数据类型以提升验证效果
6. **利用版本控制**：微小变更时不创建新键名
7. **记录转换过程**：为跟踪函数添加文档字符串
8. **设置项目分组**：组织相关工作以便管理和访问控制
9. **高效查询**：加载大型数据集前先使用过滤器
10. **可视化谱系**：使用 `view_lineage()` 理解数据来源
