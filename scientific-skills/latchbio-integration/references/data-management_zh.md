# 数据管理

## 概述
Latch 通过云存储抽象（LatchFile、LatchDir）和用于组织实验数据的结构化注册表系统提供全面的数据管理功能。

## 云存储：LatchFile 与 LatchDir

### LatchFile

表示 Latch 云存储系统中的文件。

```python
from latch.types import LatchFile

# 创建对现有文件的引用
input_file = LatchFile("latch:///data/sample.fastq")

# 访问文件属性
file_path = input_file.local_path  # 执行时的本地路径
file_remote = input_file.remote_path  # 云存储路径
```

### LatchDir

表示 Latch 云存储系统中的目录。

```python
from latch.types import LatchDir

# 创建目录引用
output_dir = LatchDir("latch:///results/experiment_1")

# 目录操作
all_files = output_dir.glob("*.bam")  # 查找匹配模式的文件
subdirs = output_dir.iterdir()  # 列出内容
```

### 路径格式

Latch 存储使用特殊 URL 方案：
- **Latch 路径**：`latch:///path/to/file`
- **本地路径**：工作流执行时自动解析
- **S3 路径**：配置后可直接使用

### 文件传输

文件在本地执行环境与云存储间自动传输：

```python
from latch import small_task
from latch.types import LatchFile

@small_task
def process_file(input_file: LatchFile) -> LatchFile:
    # 文件自动下载到本地执行环境
    local_path = input_file.local_path

    # 处理文件
    with open(local_path, 'r') as f:
        data = f.read()

    # 写入输出
    output_path = "output.txt"
    with open(output_path, 'w') as f:
        f.write(processed_data)

    # 自动上传回云存储
    return LatchFile(output_path, "latch:///results/output.txt")
```

### 通配符匹配

使用模式匹配查找文件：

```python
from latch.types import LatchDir

data_dir = LatchDir("latch:///data")

# 查找所有 FASTQ 文件
fastq_files = data_dir.glob("**/*.fastq")

# 查找子目录中的文件
bam_files = data_dir.glob("alignments/**/*.bam")

# 多模式匹配
results = data_dir.glob("*.{bam,sam}")
```

## 注册表系统

注册表通过项目、表格和记录提供结构化数据组织。

### 注册表架构

```
账户/工作空间
└── 项目
    └── 表格
        └── 记录
```

### 项目管理

```python
from latch.registry.project import Project

# 获取或创建项目
project = Project.create(
    name="RNA-seq 分析",
    description="批量 RNA-seq 实验"
)

# 列出现有项目
all_projects = Project.list()

# 按 ID 获取项目
project = Project.get(project_id="proj_123")
```

### 表格管理

表格存储结构化数据记录：

```python
from latch.registry.table import Table

# 创建表格
table = Table.create(
    project_id=project.id,
    name="样本",
    columns=[
        {"name": "sample_id", "type": "string"},
        {"name": "condition", "type": "string"},
        {"name": "replicate", "type": "number"},
        {"name": "fastq_file", "type": "file"}
    ]
)

# 列出项目中的表格
tables = Table.list(project_id=project.id)

# 按 ID 获取表格
table = Table.get(table_id="tbl_456")
```

### 列类型

支持的数据类型：
- `string` - 文本数据
- `number` - 数值（整数或浮点数）
- `boolean` - 布尔值
- `date` - 日期值
- `file` - LatchFile 引用
- `directory` - LatchDir 引用
- `link` - 其他表格记录的引用
- `enum` - 预定义列表中的枚举值

### 记录管理

```python
from latch.registry.record import Record

# 创建记录
record = Record.create(
    table_id=table.id,
    values={
        "sample_id": "S001",
        "condition": "处理组",
        "replicate": 1,
        "fastq_file": LatchFile("latch:///data/S001.fastq")
    }
)

# 批量创建记录
records = Record.bulk_create(
    table_id=table.id,
    records=[
        {"sample_id": "S001", "condition": "处理组"},
        {"sample_id": "S002", "condition": "对照组"}
    ]
)

# 查询记录
all_records = Record.list(table_id=table.id)
filtered = Record.list(
    table_id=table.id,
    filter={"condition": "处理组"}
)

# 更新记录
record.update(values={"replicate": 2})

# 删除记录
record.delete()
```

### 关联记录

创建表格间的关系：

```python
# 定义带链接列的表格
results_table = Table.create(
    project_id=project.id,
    name="结果",
    columns=[
        {"name": "sample", "type": "link", "target_table": samples_table.id},
        {"name": "alignment_bam", "type": "file"},
        {"name": "gene_counts", "type": "file"}
    ]
)

# 创建带链接的记录
result_record = Record.create(
    table_id=results_table.id,
    values={
        "sample": sample_record.id,  # 链接到样本记录
        "alignment_bam": LatchFile("latch:///results/aligned.bam"),
        "gene_counts": LatchFile("latch:///results/counts.tsv")
    }
)

# 访问关联数据
sample_data = result_record.values["sample"].resolve()
```

### 枚举列

定义含预定义值的列：

```python
table = Table.create(
    project_id=project.id,
    name="实验",
    columns=[
        {
            "name": "status",
            "type": "enum",
            "options": ["待处理", "运行中", "已完成", "失败"]
        }
    ]
)
```

### 事务与批量更新

高效更新多条记录：

```python
from latch.registry.transaction import Transaction

# 启动事务
with Transaction() as txn:
    for record in records:
        record.update(values={"status": "已处理"}, transaction=txn)
    # 退出上下文时提交更改
```

## 工作流集成

### 在工作流中使用注册表

```python
from latch import workflow, small_task
from latch.types import LatchFile
from latch.registry.table import Table
from latch.registry.record import Record

@small_task
def process_and_save(sample_id: str, table_id: str) -> str:
    # 从注册表获取样本
    table = Table.get(table_id=table_id)
    records = Record.list(
        table_id=table_id,
        filter={"sample_id": sample_id}
    )
    sample = records[0]

    # 处理文件
    input_file = sample.values["fastq_file"]
    # ... 处理逻辑 ...

    # 将结果保存回注册表
    sample.update(values={
        "status": "已完成",
        "results_file": output_file
    })

    return "成功"

@workflow
def registry_workflow(sample_id: str, table_id: str):
    """与注册表集成的工作流"""
    return process_and_save(sample_id=sample_id, table_id=table_id)
```

### 数据自动触发工作流

配置工作流在数据添加到注册表文件夹时自动运行：

```python
from latch.resources.launch_plan import LaunchPlan

# 创建监控文件夹的启动计划
launch_plan = LaunchPlan.create(
    workflow_name="rnaseq_pipeline",
    name="auto_process",
    trigger_folder="latch:///incoming_data",
    default_inputs={
        "output_dir": "latch:///results"
    }
)
```

## 账户与工作空间管理

### 账户信息

```python
from latch.account import Account

# 获取当前账户
account = Account.current()

# 账户属性
workspace_id = account.id
workspace_name = account.name
```

### 团队工作空间

访问共享团队工作空间：

```python
# 列出可用工作空间
workspaces = Account.list()

# 切换工作空间
Account.set_current(workspace_id="ws_789")
```

## 数据操作函数

### 数据连接

`latch.functions` 模块提供数据操作工具：

```python
from latch.functions import left_join, inner_join, outer_join, right_join

# 连接表格
combined = left_join(
    left_table=table1,
    right_table=table2,
    on="sample_id"
)
```

### 数据过滤

```python
from latch.functions import filter_records

# 过滤记录
filtered = filter_records(
    table=table,
    condition=lambda record: record["replicate"] > 1
)
```

### 密钥管理

安全存储和检索密钥：

```python
from latch.functions import get_secret

# 在工作流中获取密钥
api_key = get_secret("api_key")
```

## 最佳实践

1. **路径组织**：使用一致的文件夹结构（如 `/data`, `/results`, `/logs`）
2. **注册表模式**：在批量录入数据前定义表格模式
3. **关联记录**：使用链接维护实验间关系
4. **批量操作**：使用事务更新多条记录
5. **文件命名**：采用一致且描述性的文件命名规范
6. **元数据**：在注册表中存储实验元数据以确保可追溯性
7. **验证**：创建记录时验证数据类型
8. **清理**：定期归档或删除未使用数据

## 常用模式

### 样本追踪

```python
# 创建样本表
samples = Table.create(
    project_id=project.id,
    name="样本",
    columns=[
        {"name": "sample_id", "type": "string"},
        {"name": "collection_date", "type": "date"},
        {"name": "raw_fastq_r1", "type": "file"},
        {"name": "raw_fastq_r2", "type": "file"},
        {"name": "status", "type": "enum", "options": ["待处理", "处理中", "完成"]}
    ]
)
```

### 结果组织

```python
# 创建与样本关联的结果表
results = Table.create(
    project_id=project.id,
    name="分析结果",
    columns=[
        {"name": "sample", "type": "link", "target_table": samples.id},
        {"name": "alignment_bam", "type": "file"},
        {"name": "variants_vcf", "type": "file"},
        {"name": "qc_metrics", "type": "file"}
    ]
)
```
