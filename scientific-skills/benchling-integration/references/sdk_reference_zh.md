# Benchling Python SDK 参考

## 安装与设置

### 安装

```bash
# 稳定版
pip install benchling-sdk

# 使用 Poetry
poetry add benchling-sdk

# 预发布/预览版本（不建议用于生产环境）
pip install benchling-sdk --pre
poetry add benchling-sdk --allow-prereleases
```

### 要求
- Python 3.7 或更高版本
- 在您的 Benchling 租户上启用 API 访问

### 基本初始化

```python
from benchling_sdk.benchling import Benchling
from benchling_sdk.auth.api_key_auth import ApiKeyAuth

benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=ApiKeyAuth("your_api_key")
)
```

## SDK 架构

### 主要类

**Benchling 客户端：**
`benchling_sdk.benchling.Benchling` 类是所有 SDK 交互的根。它提供对所有资源端点的访问：

```python
benchling.dna_sequences      # DNA 序列操作
benchling.rna_sequences      # RNA 序列操作
benchling.aa_sequences       # 氨基酸序列操作
benchling.custom_entities    # 自定义实体操作
benchling.mixtures           # 混合物操作
benchling.containers         # 容器操作
benchling.boxes              # 盒子操作
benchling.locations          # 位置操作
benchling.plates             # 微孔板操作
benchling.entries            # 笔记本条目操作
benchling.workflow_tasks     # 工作流任务操作
benchling.requests           # 请求操作
benchling.folders            # 文件夹操作
benchling.projects           # 项目操作
benchling.users              # 用户操作
benchling.teams              # 团队操作
```

### 资源模式

所有资源都遵循一致的 CRUD 模式：

```python
# 创建
resource.create(CreateModel(...))

# 读取（单个）
resource.get(id="resource_id")

# 读取（列表）
resource.list(optional_filters...)

# 更新
resource.update(id="resource_id", UpdateModel(...))

# 归档/删除
resource.archive(id="resource_id")
```

## 实体管理

### DNA 序列

**创建：**
```python
from benchling_sdk.models import DnaSequenceCreate

sequence = benchling.dna_sequences.create(
    DnaSequenceCreate(
        name="pET28a-GFP",
        bases="ATCGATCGATCG",
        is_circular=True,
        folder_id="fld_abc123",
        schema_id="ts_abc123",
        fields=benchling.models.fields({
            "gene_name": "GFP",
            "resistance": "Kanamycin",
            "copy_number": "High"
        })
    )
)
```

**读取：**
```python
# 按 ID 获取
seq = benchling.dna_sequences.get(sequence_id="seq_abc123")
print(f"{seq.name}: {len(seq.bases)} bp")

# 带过滤器的列表
sequences = benchling.dna_sequences.list(
    folder_id="fld_abc123",
    schema_id="ts_abc123",
    name="pET28a"  # 按名称筛选
)

for page in sequences:
    for seq in page:
        print(f"{seq.id}: {seq.name}")
```

**更新：**
```python
from benchling_sdk.models import DnaSequenceUpdate

updated = benchling.dna_sequences.update(
    sequence_id="seq_abc123",
    dna_sequence=DnaSequenceUpdate(
        name="pET28a-GFP-v2",
        fields=benchling.models.fields({
            "gene_name": "eGFP",
            "notes": "Codon optimized"
        })
    )
)
```

**归档：**
```python
benchling.dna_sequences.archive(
    sequence_id="seq_abc123",
    reason="Deprecated construct"
)
```

### RNA 序列

与 DNA 序列类似：

```python
from benchling_sdk.models import RnaSequenceCreate, RnaSequenceUpdate

# 创建
rna = benchling.rna_sequences.create(
    RnaSequenceCreate(
        name="gRNA-target1",
        bases="AUCGAUCGAUCG",
        folder_id="fld_abc123",
        fields=benchling.models.fields({
            "target_gene": "TP53",
            "off_target_score": "95"
        })
    )
)

# 更新
updated_rna = benchling.rna_sequences.update(
    rna_sequence_id=rna.id,
    rna_sequence=RnaSequenceUpdate(
        fields=benchling.models.fields({
            "validated": "Yes"
        })
    )
)
```

### 氨基酸（蛋白质）序列

```python
from benchling_sdk.models import AaSequenceCreate

protein = benchling.aa_sequences.create(
    AaSequenceCreate(
        name="Green Fluorescent Protein",
        amino_acids="MSKGEELFTGVVPILVELDGDVNGHKFSVSGEGEGDATYGKLTLKF",
        folder_id="fld_abc123",
        fields=benchling.models.fields({
            "molecular_weight": "27000",
            "extinction_coefficient": "21000"
        })
    )
)
```

### 自定义实体

自定义实体由您的租户的模式定义：

```python
from benchling_sdk.models import CustomEntityCreate, CustomEntityUpdate

# 创建
cell_line = benchling.custom_entities.create(
    CustomEntityCreate(
        name="HEK293T-Clone5",
        schema_id="ts_cellline_abc123",
        folder_id="fld_abc123",
        fields=benchling.models.fields({
            "passage_number": "15",
            "mycoplasma_test": "Negative",
            "freezing_date": "2025-10-15"
        })
    )
)

# 更新
updated_cell_line = benchling.custom_entities.update(
    entity_id=cell_line.id,
    custom_entity=CustomEntityUpdate(
        fields=benchling.models.fields({
            "passage_number": "16",
            "notes": "Expanded for experiment"
        })
    )
)
```

### 混合物

混合物结合了多种组分：

```python
from benchling_sdk.models import MixtureCreate, IngredientCreate

mixture = benchling.mixtures.create(
    MixtureCreate(
        name="LB-Amp Media",
        folder_id="fld_abc123",
        schema_id="ts_mixture_abc123",
        ingredients=[
            IngredientCreate(
                component_entity_id="ent_lb_base",
                amount="1000 mL"
            ),
            IngredientCreate(
                component_entity_id="ent_ampicillin",
                amount="100 mg"
            )
        ],
        fields=benchling.models.fields({
            "pH": "7.0",
            "sterilized": "Yes"
        })
    )
)
```

### 注册表操作

**直接注册表注册：**
```python
# 创建时注册实体
registered_seq = benchling.dna_sequences.create(
    DnaSequenceCreate(
        name="Construct-001",
        bases="ATCG",
        is_circular=True,
        folder_id="fld_abc123",
        entity_registry_id="src_abc123",
        naming_strategy="NEW_IDS"  # or "IDS_FROM_NAMES"
    )
)
print(f"Registry ID: {registered_seq.registry_id}")
```

**命名策略：**
- `NEW_IDS`：Benchling 生成新的注册 ID
- `IDS_FROM_NAMES`：使用实体名称作为注册 ID（名称必须唯一）

## 库存管理

### 容器

```python
from benchling_sdk.models import ContainerCreate, ContainerUpdate

# 创建
container = benchling.containers.create(
    ContainerCreate(
        name="Sample-001-Tube",
        schema_id="cont_schema_abc123",
        barcode="CONT001",
        parent_storage_id="box_abc123",  # 放入盒子
        fields=benchling.models.fields({
            "concentration": "100 ng/μL",
            "volume": "50 μL",
            "sample_type": "gDNA"
        })
    )
)

# 更新位置
benchling.containers.transfer(
    container_id=container.id,
    destination_id="box_xyz789"
)

# 更新属性
updated = benchling.containers.update(
    container_id=container.id,
    container=ContainerUpdate(
        fields=benchling.models.fields({
            "volume": "45 μL",
            "notes": "Used 5 μL for PCR"
        })
    )
)

# 检出
benchling.containers.check_out(
    container_id=container.id,
    comment="Taking to bench"
)

# 检入
benchling.containers.check_in(
    container_id=container.id,
    location_id="bench_location_abc"
)
```

### 盒子

```python
from benchling_sdk.models import BoxCreate

box = benchling.boxes.create(
    BoxCreate(
        name="Freezer-A-Box-01",
        schema_id="box_schema_abc123",
        parent_storage_id="loc_freezer_a",
        barcode="BOX001",
        fields=benchling.models.fields({
            "box_type": "81-place",
            "temperature": "-80C"
        })
    )
)

# 列出盒子中的容器
containers = benchling.containers.list(
    parent_storage_id=box.id
)
```

### 位置

```python
from benchling_sdk.models import LocationCreate

location = benchling.locations.create(
    LocationCreate(
        name="Freezer A - Shelf 2",
        parent_storage_id="loc_freezer_a",
        barcode="LOC-A-S2"
    )
)
```

### 微孔板

```python
from benchling_sdk.models import PlateCreate, WellCreate

# 创建 96 孔板
plate = benchling.plates.create(
    PlateCreate(
        name="PCR-Plate-001",
        schema_id="plate_schema_abc123",
        barcode="PLATE001",
        wells=[
            WellCreate(
                position="A1",
                entity_id="sample_entity_abc"
            ),
            WellCreate(
                position="A2",
                entity_id="sample_entity_xyz"
            )
            # ... 更多孔
        ]
    )
)
```

## 笔记本操作

### 条目

```python
from benchling_sdk.models import EntryCreate, EntryUpdate

# 创建条目
entry = benchling.entries.create(
    EntryCreate(
        name="Cloning Experiment 2025-10-20",
        folder_id="fld_abc123",
        schema_id="entry_schema_abc123",
        fields=benchling.models.fields({
            "objective": "Clone GFP into pET28a",
            "date": "2025-10-20",
            "experiment_type": "Molecular Biology"
        })
    )
)

# 更新条目
updated_entry = benchling.entries.update(
    entry_id=entry.id,
    entry=EntryUpdate(
        fields=benchling.models.fields({
            "results": "Successful cloning, 10 colonies",
            "notes": "Colony 5 shows best fluorescence"
        })
    )
)
```

### 将实体链接到条目

```python
# 将 DNA 序列链接到条目
link = benchling.entry_links.create(
    entry_id="entry_abc123",
    entity_id="seq_xyz789"
)

# 列出条目的链接
links = benchling.entry_links.list(entry_id="entry_abc123")
```

## 工作流管理

### 任务

```python
from benchling_sdk.models import WorkflowTaskCreate, WorkflowTaskUpdate

# 创建任务
task = benchling.workflow_tasks.create(
    WorkflowTaskCreate(
        name="PCR Amplification",
        workflow_id="wf_abc123",
        assignee_id="user_abc123",
        schema_id="task_schema_abc123",
        fields=benchling.models.fields({
            "template": "seq_abc123",
            "primers": "Forward: ATCG, Reverse: CGAT",
            "priority": "High"
        })
    )
)

# 更新状态
completed_task = benchling.workflow_tasks.update(
    task_id=task.id,
    workflow_task=WorkflowTaskUpdate(
        status_id="status_complete_abc123",
        fields=benchling.models.fields({
            "completion_date": "2025-10-20",
            "yield": "500 ng"
        })
    )
)

# 列出任务
tasks = benchling.workflow_tasks.list(
    workflow_id="wf_abc123",
    status_ids=["status_pending", "status_in_progress"]
)
```

## 高级功能

### 分页

SDK 使用生成器实现内存高效的分页：

```python
# 自动分页
sequences = benchling.dna_sequences.list()

# 获取预估总数
total = sequences.estimated_count()
print(f"Total sequences: {total}")

# 遍历所有页面
for page in sequences:
    for seq in page:
        process(seq)

# 手动控制页面大小
sequences = benchling.dna_sequences.list(page_size=50)
```

### 异步任务处理

一些操作是异步的并返回任务 ID：

```python
from benchling_sdk.helpers.tasks import wait_for_task
from benchling_sdk.errors import WaitForTaskExpiredError

# 启动异步操作
response = benchling.some_bulk_operation(...)
task_id = response.task_id

# 等待完成
try:
    result = wait_for_task(
        benchling,
        task_id=task_id,
        interval_wait_seconds=2,  # 每 2 秒轮询一次
        max_wait_seconds=600       # 10 分钟后超时
    )
    print("任务成功完成")
except WaitForTaskExpiredError:
    print("任务超时")
```

### 错误处理

```python
from benchling_sdk.errors import (
    BenchlingError,
    NotFoundError,
    ValidationError,
    UnauthorizedError
)

try:
    sequence = benchling.dna_sequences.get(sequence_id="seq_invalid")
except NotFoundError:
    print("序列未找到")
except UnauthorizedError:
    print("权限不足")
except ValidationError as e:
    print(f"无效数据: {e}")
except BenchlingError as e:
    print(f"一般 Benchling 错误: {e}")
```

### 重试策略

自定义重试行为：

```python
from benchling_sdk.benchling import Benchling
from benchling_sdk.auth.api_key_auth import ApiKeyAuth
from benchling_sdk.retry import RetryStrategy

# 自定义重试配置
retry_strategy = RetryStrategy(
    max_retries=3,
    backoff_factor=0.5,
    status_codes_to_retry=[429, 502, 503, 504]
)

benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=ApiKeyAuth("your_api_key"),
    retry_strategy=retry_strategy
)

# 禁用重试
benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=ApiKeyAuth("your_api_key"),
    retry_strategy=RetryStrategy(max_retries=0)
)
```

### 自定义 API 调用

对于不支持的端点：

```python
# 带模型解析的 GET 请求
from benchling_sdk.models import DnaSequence

response = benchling.api.get_modeled(
    path="/api/v2/dna-sequences/seq_abc123",
    response_type=DnaSequence
)

# POST 请求
from benchling_sdk.models import DnaSequenceCreate

response = benchling.api.post_modeled(
    path="/api/v2/dna-sequences",
    request_body=DnaSequenceCreate(...),
    response_type=DnaSequence
)

# 原始请求
raw_response = benchling.api.get(
    path="/api/v2/custom-endpoint",
    params={"key": "value"}
)
```

### 批量操作

高效处理多个项目：

```python
# 批量创建
from benchling_sdk.models import DnaSequenceCreate

sequences_to_create = [
    DnaSequenceCreate(name=f"Seq-{i}", bases="ATCG", folder_id="fld_abc")
    for i in range(100)
]

# 分批创建
batch_size = 10
for i in range(0, len(sequences_to_create), batch_size):
    batch = sequences_to_create[i:i+batch_size]
    for seq in batch:
        benchling.dna_sequences.create(seq)
```

### 模式字段助手

将字典转换为 Fields 对象：

```python
# 使用字段助手
fields_dict = {
    "concentration": "100 ng/μL",
    "volume": "50 μL",
    "quality_score": "8.5",
    "date_prepared": "2025-10-20"
}

fields = benchling.models.fields(fields_dict)

# 在创建/更新中使用
container = benchling.containers.create(
    ContainerCreate(
        name="Sample-001",
        schema_id="schema_abc",
        fields=fields
    )
)
```

### 前向兼容性

SDK 优雅地处理未知 API 值：

```python
# 未知枚举值被保留
entity = benchling.dna_sequences.get("seq_abc")
# 即使 API 返回 SDK 中没有的新枚举值，它也会被保留

# 未知多态类型返回 UnknownType
from benchling_sdk.models import UnknownType

if isinstance(entity, UnknownType):
    print(f"未知类型: {entity.type}")
    # 仍可访问原始数据
    print(entity.raw_data)
```

## 最佳实践

### 使用类型提示

```python
from benchling_sdk.models import DnaSequence, DnaSequenceCreate
from typing import List

def create_sequences(names: List[str], folder_id: str) -> List[DnaSequence]:
    sequences = []
    for name in names:
        seq = benchling.dna_sequences.create(
            DnaSequenceCreate(
                name=name,
                bases="ATCG",
                folder_id=folder_id
            )
        )
        sequences.append(seq)
    return sequences
```

### 高效过滤

使用 API 过滤器而不是客户端过滤：

```python
# 良好 - 在服务器上过滤
sequences = benchling.dna_sequences.list(
    folder_id="fld_abc123",
    schema_id="ts_abc123"
)

# 不佳 - 加载所有内容然后过滤
all_sequences = benchling.dna_sequences.list()
filtered = [s for page in all_sequences for s in page if s.folder_id == "fld_abc123"]
```

### 资源清理

```python
# 归档旧实体
cutoff_date = "2024-01-01"
sequences = benchling.dna_sequences.list()

for page in sequences:
    for seq in page:
        if seq.created_at < cutoff_date:
            benchling.dna_sequences.archive(
                sequence_id=seq.id,
                reason="Archiving old sequences"
            )
```

## 故障排除

### 常见问题

**导入错误：**
```python
# 错误
from benchling_sdk import Benchling  # ImportError

# 正确
from benchling_sdk.benchling import Benchling
```

**字段验证：**
```python
# 字段必须与模式匹配
# 在 Benchling UI 中检查模式字段类型
fields = benchling.models.fields({
    "numeric_field": "123",    # 即使是数字也应为字符串
    "date_field": "2025-10-20", # 格式：YYYY-MM-DD
    "dropdown_field": "Option1" # 必须与下拉选项完全匹配
})
```

**分页耗尽：**
```python
# 生成器只能迭代一次
sequences = benchling.dna_sequences.list()
for page in sequences:  # 第一次迭代正常
    pass
for page in sequences:  # 第二次迭代不返回任何内容！
    pass

# 解决方案：创建新的生成器
sequences = benchling.dna_sequences.list()  # 新的生成器
```

## 参考

- **SDK 源代码：** https://github.com/benchling/benchling-sdk
- **SDK 文档：** https://benchling.com/sdk-docs/
- **API 参考：** https://benchling.com/api/reference
- **常见示例：** https://docs.benchling.com/docs/common-sdk-interactions-and-examples
