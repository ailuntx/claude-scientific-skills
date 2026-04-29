---
name: benchling-integration
description: Benchling 研发平台集成。通过 API 访问注册表（DNA、蛋白质）、库存、电子实验笔记本条目和工作流，构建 Benchling 应用，查询数据仓库，实现实验室数据管理自动化。
license: Unknown
compatibility: 需要 Benchling 账户和 API 密钥
metadata:
    skill-author: K-Dense Inc.
---

# Benchling 集成

## 概述

Benchling 是面向生命科学研发的云平台。通过 Python SDK 和 REST API 以编程方式访问注册表实体（DNA、蛋白质）、库存、电子实验笔记本和工作流。

## 何时使用此技能

此技能适用于以下场景：
- 使用 Benchling 的 Python SDK 或 REST API
- 管理生物序列（DNA、RNA、蛋白质）和注册表实体
- 自动化库存操作（样品、容器、位置、转移）
- 创建或查询电子实验笔记本条目
- 构建工作流自动化或 Benchling 应用
- 在 Benchling 与外部系统间同步数据
- 查询 Benchling 数据仓库进行分析
- 通过 AWS EventBridge 设置事件驱动集成

## 核心功能

### 1. 认证与设置

**Python SDK 安装：**
```python
# 稳定版
uv pip install benchling-sdk
# 或使用 Poetry
poetry add benchling-sdk
```

**认证方法：**

API 密钥认证（推荐用于脚本）：
```python
from benchling_sdk.benchling import Benchling
from benchling_sdk.auth.api_key_auth import ApiKeyAuth

benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=ApiKeyAuth("your_api_key")
)
```

OAuth 客户端凭证（适用于应用）：
```python
from benchling_sdk.auth.client_credentials_oauth2 import ClientCredentialsOAuth2

auth_method = ClientCredentialsOAuth2(
    client_id="your_client_id",
    client_secret="your_client_secret"
)
benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=auth_method
)
```

**关键点：**
- API 密钥需从 Benchling 的"个人设置"中获取
- 安全存储凭证（使用环境变量或密码管理器）
- 所有 API 请求必须使用 HTTPS
- 认证权限与用户在 UI 中的权限一致

详细认证信息（包括 OIDC 和安全最佳实践）请参阅 `references/authentication.md`。

### 2. 注册表与实体管理

注册表实体包括 DNA 序列、RNA 序列、氨基酸序列、自定义实体和混合物。SDK 提供类型化类来创建和管理这些实体。

**创建 DNA 序列：**
```python
from benchling_sdk.models import DnaSequenceCreate

sequence = benchling.dna_sequences.create(
    DnaSequenceCreate(
        name="My Plasmid",
        bases="ATCGATCG",
        is_circular=True,
        folder_id="fld_abc123",
        schema_id="ts_abc123",  # 可选
        fields=benchling.models.fields({"gene_name": "GFP"})
    )
)
```

**注册表登记：**

创建时直接注册实体：
```python
sequence = benchling.dna_sequences.create(
    DnaSequenceCreate(
        name="My Plasmid",
        bases="ATCGATCG",
        is_circular=True,
        folder_id="fld_abc123",
        entity_registry_id="src_abc123",  # 目标注册表
        naming_strategy="NEW_IDS"  # 或 "IDS_FROM_NAMES"
    )
)
```

**重要提示：** 只能使用 `entity_registry_id` 或 `naming_strategy` 之一，不可同时使用。

**更新实体：**
```python
from benchling_sdk.models import DnaSequenceUpdate

updated = benchling.dna_sequences.update(
    sequence_id="seq_abc123",
    dna_sequence=DnaSequenceUpdate(
        name="Updated Plasmid Name",
        fields=benchling.models.fields({"gene_name": "mCherry"})
    )
)
```

未指定字段保持不变，支持部分更新。

**列表与分页：**
```python
# 列出所有 DNA 序列（返回生成器）
sequences = benchling.dna_sequences.list()
for page in sequences:
    for seq in page:
        print(f"{seq.name} ({seq.id})")

# 检查总数
total = sequences.estimated_count()
```

**关键操作：**
- 创建：`benchling.<实体类型>.create()`
- 读取：`benchling.<实体类型>.get(id)` 或 `.list()`
- 更新：`benchling.<实体类型>.update(id, update_object)`
- 归档：`benchling.<实体类型>.archive(id)`

实体类型：`dna_sequences`, `rna_sequences`, `aa_sequences`, `custom_entities`, `mixtures`

完整 SDK 参考和高级模式请参阅 `references/sdk_reference.md`。

### 3. 库存管理

管理 Benchling 库存系统中的物理样品、容器、盒子和位置。

**创建容器：**
```python
from benchling_sdk.models import ContainerCreate

container = benchling.containers.create(
    ContainerCreate(
        name="Sample Tube 001",
        schema_id="cont_schema_abc123",
        parent_storage_id="box_abc123",  # 可选
        fields=benchling.models.fields({"concentration": "100 ng/μL"})
    )
)
```

**管理盒子：**
```python
from benchling_sdk.models import BoxCreate

box = benchling.boxes.create(
    BoxCreate(
        name="Freezer Box A1",
        schema_id="box_schema_abc123",
        parent_storage_id="loc_abc123"
    )
)
```

**转移物品：**
```python
# 将容器转移到新位置
transfer = benchling.containers.transfer(
    container_id="cont_abc123",
    destination_id="box_xyz789"
)
```

**关键库存操作：**
- 创建容器、盒子、位置、板
- 更新库存项属性
- 在位置间转移物品
- 物品签入/签出
- 批量转移操作

### 4. 实验笔记本与文档

与电子实验笔记本（ELN）条目、实验方案和模板交互。

**创建笔记本条目：**
```python
from benchling_sdk.models import EntryCreate

entry = benchling.entries.create(
    EntryCreate(
        name="Experiment 2025-10-20",
        folder_id="fld_abc123",
        schema_id="entry_schema_abc123",
        fields=benchling.models.fields({"objective": "Test gene expression"})
    )
)
```

**将实体关联到条目：**
```python
# 在条目中添加实体引用
entry_link = benchling.entry_links.create(
    entry_id="entry_abc123",
    entity_id="seq_xyz789"
)
```

**关键笔记本操作：**
- 创建和更新实验笔记本条目
- 管理条目模板
- 关联实体和结果到条目
- 导出条目用于文档记录

### 5. 工作流与自动化

使用 Benchling 工作流系统自动化实验流程。

**创建工作流任务：**
```python
from benchling_sdk.models import WorkflowTaskCreate

task = benchling.workflow_tasks.create(
    WorkflowTaskCreate(
        name="PCR Amplification",
        workflow_id="wf_abc123",
        assignee_id="user_abc123",
        fields=benchling.models.fields({"template": "seq_abc123"})
    )
)
```

**更新任务状态：**
```python
from benchling_sdk.models import WorkflowTaskUpdate

updated_task = benchling.workflow_tasks.update(
    task_id="task_abc123",
    workflow_task=WorkflowTaskUpdate(
        status_id="status_complete_abc123"
    )
)
```

**异步操作：**

部分操作是异步的并返回任务：
```python
# 等待任务完成
from benchling_sdk.helpers.tasks import wait_for_task

result = wait_for_task(
    benchling,
    task_id="task_abc123",
    interval_wait_seconds=2,
    max_wait_seconds=300
)
```

**关键工作流操作：**
- 创建和管理工作流任务
- 更新任务状态和分配
- 异步执行批量操作
- 监控任务进度

### 6. 事件与集成

通过 AWS EventBridge 订阅 Benchling 事件以实现实时集成。

**事件类型：**
- 实体创建、更新、归档
- 库存转移
- 工作流任务状态变更
- 条目创建和更新
- 结果登记

**集成模式：**
1. 在 Benchling 设置中配置到 AWS EventBridge 的事件路由
2. 创建 EventBridge 规则过滤事件
3. 将事件路由到 Lambda 函数或其他目标
4. 处理事件并更新外部系统

**用例：**
- 将 Benchling 数据同步到外部数据库
- 工作流完成时触发下游流程
- 实体变更时发送通知
- 审计追踪记录

事件模式和配置请参阅 Benchling 事件文档。

### 7. 数据仓库与分析

通过数据仓库使用 SQL 查询历史 Benchling 数据。

**访问方法：**
Benchling 数据仓库提供 SQL 访问接口用于分析和报告。使用标准 SQL 客户端连接并输入提供的凭证。

**常用查询：**
- 聚合实验结果
- 分析库存趋势
- 生成合规报告
- 导出数据用于外部分析

**分析工具集成：**
- Jupyter 笔记本用于交互式分析
- BI 工具（Tableau, Looker, PowerBI）
- 自定义仪表板

## 最佳实践

### 错误处理

SDK 自动重试失败请求：
```python
# 自动重试 429、502、503、504 状态码
# 最多重试 5 次，采用指数退避策略
# 如需自定义重试行为
from benchling_sdk.retry import RetryStrategy

benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=ApiKeyAuth("your_api_key"),
    retry_strategy=RetryStrategy(max_retries=3)
)
```

### 分页效率

使用生成器实现内存高效分页：
```python
# 基于生成器的迭代
for page in benchling.dna_sequences.list():
    for sequence in page:
        process(sequence)

# 不加载所有页面即可估算总数
total = benchling.dna_sequences.list().estimated_count()
```

### 模式字段辅助工具

使用 `fields()` 辅助函数处理自定义模式字段：
```python
# 将字典转换为 Fields 对象
custom_fields = benchling.models.fields({
    "concentration": "100 ng/μL",
    "date_prepared": "2025-10-20",
    "notes": "High quality prep"
})
```

### 前向兼容性

SDK 能优雅处理未知枚举值和类型：
- 保留未知枚举值
- 无法识别的多态类型返回 `UnknownType`
- 支持使用更新的 API 版本

### 安全注意事项

- 切勿将 API 密钥提交到版本控制系统
- 使用环境变量存储凭证
- 密钥泄露时立即轮换
- 为应用授予最小必要权限
- 多用户场景使用 OAuth

## 资源

### references/

详细参考文档提供深度信息：

- **authentication.md** - 完整认证指南（含 OIDC、安全最佳实践和凭证管理）
- **sdk_reference.md** - 详细 Python SDK 参考（含高级模式、示例和所有实体类型）
- **api_endpoints.md** - REST API 端点参考（无需 SDK 直接发起 HTTP 调用）

根据具体集成需求加载这些参考文档。

### scripts/

此技能包含示例脚本，可移除或替换为针对特定 Benchling 工作流的自定义自动化脚本。

## 常见用例

**1. 批量实体导入：**
```python
# 从 FASTA 文件导入多个序列
from Bio import SeqIO

for record in SeqIO.parse("sequences.fasta", "fasta"):
    benchling.dna_sequences.create(
        DnaSequenceCreate(
            name=record.id,
            bases=str(record.seq),
            is_circular=False,
            folder_id="fld_abc123"
        )
    )
```

**2. 库存审计：**
```python
# 列出特定位置的所有容器
containers = benchling.containers.list(
    parent_storage_id="box_abc123"
)

for page in containers:
    for container in page:
        print(f"{container.name}: {container.barcode}")
```

**3. 工作流自动化：**
```python
# 更新工作流的所有待处理任务
tasks = benchling.workflow_tasks.list(
    workflow_id="wf_abc123",
    status="pending"
)

for page in tasks:
    for task in page:
        #
