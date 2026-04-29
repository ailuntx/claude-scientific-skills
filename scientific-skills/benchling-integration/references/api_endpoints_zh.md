# Benchling REST API 端点参考

## 基础 URL

所有 API 请求均使用以下基础 URL 格式：
```
https://{tenant}.benchling.com/api/v2
```

将 `{tenant}` 替换为您的 Benchling 租户名称。

## API 版本控制

当前 API 版本：`v2` (2025-10-07)

API 版本在 URL 路径中指定。Benchling 在主要版本内保持向后兼容性。

## 身份验证

所有请求均需通过 HTTP 头进行身份验证：

**API 密钥（基础认证）：**
```bash
curl -X GET \
  https://your-tenant.benchling.com/api/v2/dna-sequences \
  -u "your_api_key:"
```

**OAuth Bearer 令牌：**
```bash
curl -X GET \
  https://your-tenant.benchling.com/api/v2/dna-sequences \
  -H "Authorization: Bearer your_access_token"
```

## 通用请求头

```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

## 响应格式

所有响应均遵循一致的 JSON 结构：

**单资源：**
```json
{
  "id": "seq_abc123",
  "name": "My Sequence",
  "bases": "ATCGATCG",
  ...
}
```

**列表响应：**
```json
{
  "results": [
    {"id": "seq_1", "name": "Sequence 1"},
    {"id": "seq_2", "name": "Sequence 2"}
  ],
  "nextToken": "token_for_next_page"
}
```

## 分页

列表端点支持分页：

**查询参数：**
- `pageSize`：每页项目数（默认：50，最大：100）
- `nextToken`：上一页响应中的下一页令牌

**示例：**
```bash
curl -X GET \
  "https://your-tenant.benchling.com/api/v2/dna-sequences?pageSize=50&nextToken=abc123"
```

## 错误响应

**格式：**
```json
{
  "error": {
    "type": "NotFoundError",
    "message": "DNA序列未找到",
    "userMessage": "请求的序列不存在或您无访问权限"
  }
}
```

**常见状态码：**
- `200 OK`：成功
- `201 Created`：资源已创建
- `400 Bad Request`：参数无效
- `401 Unauthorized`：凭证缺失或无效
- `403 Forbidden`：权限不足
- `404 Not Found`：资源不存在
- `422 Unprocessable Entity`：验证错误
- `429 Too Many Requests`：超出速率限制
- `500 Internal Server Error`：服务器错误

## 核心端点

### DNA 序列

**列出 DNA 序列：**
```http
GET /api/v2/dna-sequences

查询参数：
- pageSize: 整数（默认：50，最大：100）
- nextToken: 字符串
- folderId: 字符串
- schemaId: 字符串
- name: 字符串（按名称过滤）
- modifiedAt: 字符串（ISO 8601 日期）
```

**获取 DNA 序列：**
```http
GET /api/v2/dna-sequences/{sequenceId}
```

**创建 DNA 序列：**
```http
POST /api/v2/dna-sequences

请求体：
{
  "name": "我的质粒",
  "bases": "ATCGATCG",
  "isCircular": true,
  "folderId": "fld_abc123",
  "schemaId": "ts_abc123",
  "fields": {
    "gene_name": {"value": "GFP"},
    "resistance": {"value": "卡那霉素"}
  },
  "entityRegistryId": "src_abc123",  // 注册时可选
  "namingStrategy": "NEW_IDS"        // 注册时可选
}
```

**更新 DNA 序列：**
```http
PATCH /api/v2/dna-sequences/{sequenceId}

请求体：
{
  "name": "更新后的质粒",
  "fields": {
    "gene_name": {"value": "mCherry"}
  }
}
```

**归档 DNA 序列：**
```http
POST /api/v2/dna-sequences:archive

请求体：
{
  "dnaSequenceIds": ["seq_abc123"],
  "reason": "已弃用的构建体"
}
```

### RNA 序列

**列出 RNA 序列：**
```http
GET /api/v2/rna-sequences
```

**获取 RNA 序列：**
```http
GET /api/v2/rna-sequences/{sequenceId}
```

**创建 RNA 序列：**
```http
POST /api/v2/rna-sequences

请求体：
{
  "name": "gRNA-001",
  "bases": "AUCGAUCG",
  "folderId": "fld_abc123",
  "fields": {
    "target_gene": {"value": "TP53"}
  }
}
```

**更新 RNA 序列：**
```http
PATCH /api/v2/rna-sequences/{sequenceId}
```

**归档 RNA 序列：**
```http
POST /api/v2/rna-sequences:archive
```

### 氨基酸（蛋白质）序列

**列出氨基酸序列：**
```http
GET /api/v2/aa-sequences
```

**获取氨基酸序列：**
```http
GET /api/v2/aa-sequences/{sequenceId}
```

**创建氨基酸序列：**
```http
POST /api/v2/aa-sequences

请求体：
{
  "name": "GFP蛋白",
  "aminoAcids": "MSKGEELFTGVVPILVELDGDVNGHKF",
  "folderId": "fld_abc123"
}
```

### 自定义实体

**列出自定义实体：**
```http
GET /api/v2/custom-entities

查询参数：
- schemaId: 字符串（按类型过滤时必需）
- pageSize: 整数
- nextToken: 字符串
```

**获取自定义实体：**
```http
GET /api/v2/custom-entities/{entityId}
```

**创建自定义实体：**
```http
POST /api/v2/custom-entities

请求体：
{
  "name": "HEK293T-Clone5",
  "schemaId": "ts_cellline_abc",
  "folderId": "fld_abc123",
  "fields": {
    "passage_number": {"value": "15"},
    "mycoplasma_test": {"value": "阴性"}
  }
}
```

**更新自定义实体：**
```http
PATCH /api/v2/custom-entities/{entityId}

请求体：
{
  "fields": {
    "passage_number": {"value": "16"}
  }
}
```

### 混合物

**列出混合物：**
```http
GET /api/v2/mixtures
```

**创建混合物：**
```http
POST /api/v2/mixtures

请求体：
{
  "name": "LB-Amp培养基",
  "folderId": "fld_abc123",
  "schemaId": "ts_mixture_abc",
  "ingredients": [
    {
      "componentEntityId": "ent_lb_base",
      "amount": {"value": "1000", "units": "mL"}
    },
    {
      "componentEntityId": "ent_ampicillin",
      "amount": {"value": "100", "units": "mg"}
    }
  ]
}
```

### 容器

**列出容器：**
```http
GET /api/v2/containers

查询参数：
- parentStorageId: 字符串（按位置/盒子过滤）
- schemaId: 字符串
- barcode: 字符串
```

**获取容器：**
```http
GET /api/v2/containers/{containerId}
```

**创建容器：**
```http
POST /api/v2/containers

请求体：
{
  "name": "样品-001",
  "schemaId": "cont_schema_abc",
  "barcode": "CONT001",
  "parentStorageId": "box_abc123",
  "fields": {
    "concentration": {"value": "100 ng/μL"},
    "volume": {"value": "50 μL"}
  }
}
```

**更新容器：**
```http
PATCH /api/v2/containers/{containerId}

请求体：
{
  "fields": {
    "volume": {"value": "45 μL"}
  }
}
```

**转移容器：**
```http
POST /api/v2/containers:transfer

请求体：
{
  "containerIds": ["cont_abc123"],
  "destinationStorageId": "box_xyz789"
}
```

**签出容器：**
```http
POST /api/v2/containers:checkout

请求体：
{
  "containerIds": ["cont_abc123"],
  "comment": "带至实验台"
}
```

**签入容器：**
```http
POST /api/v2/containers:checkin

请求体：
{
  "containerIds": ["cont_abc123"],
  "locationId": "bench_loc_abc"
}
```

### 盒子

**列出盒子：**
```http
GET /api/v2/boxes

查询参数：
- parentStorageId: 字符串
- schemaId: 字符串
```

**获取盒子：**
```http
GET /api/v2/boxes/{boxId}
```

**创建盒子：**
```http
POST /api/v2/boxes

请求体：
{
  "name": "冷冻柜A-盒子01",
  "schemaId": "box_schema_abc",
  "parentStorageId": "loc_freezer_a",
  "barcode": "BOX001"
}
```

### 位置

**列出位置：**
```http
GET /api/v2/locations
```

**获取位置：**
```http
GET /api/v2/locations/{locationId}
```

**创建位置：**
```http
POST /api/v2/locations

请求体：
{
  "name": "冷冻柜A - 第2层",
  "parentStorageId": "loc_freezer_a",
  "barcode": "LOC-A-S2"
}
```

### 微孔板

**列出微孔板：**
```http
GET /api/v2/plates
```

**获取微孔板：**
```http
GET /api/v2/plates/{plateId}
```

**创建微孔板：**
```http
POST /api/v2/plates

请求体：
{
  "name": "PCR板-001",
  "schemaId": "plate_schema_abc",
  "barcode": "PLATE001",
  "wells": [
    {"position": "A1", "entityId": "ent_abc"},
    {"position": "A2", "entityId": "ent_xyz"}
  ]
}
```

### 条目（实验记录）

**列出条目：**
```http
GET /api/v2/entries

查询参数：
- folderId: 字符串
- schemaId: 字符串
- modifiedAt: 字符串
```

**获取条目：**
```http
GET /api/v2/entries/{entryId}
```

**创建条目：**
```http
POST /api/v2/entries

请求体：
{
  "name": "实验2025-10-20",
  "folderId": "fld_abc123",
  "schemaId": "entry_schema_abc",
  "fields": {
    "objective": {"value": "测试基因表达"},
    "date": {"value": "2025-10-20"}
  }
}
```

**更新条目：**
```http
PATCH /api/v2/entries/{entryId}

请求体：
{
  "fields": {
    "results": {"value": "表达成功"}
  }
}
```

### 工作流任务

**列出工作流任务：**
```http
GET /api/v2/tasks

查询参数：
- workflowId: 字符串
- statusIds: 字符串数组（逗号分隔）
- assigneeId: 字符串
```

**获取任务：**
```http
GET /api/v2/tasks/{taskId}
```

**创建任务：**
```http
POST /api/v2/tasks

请求体：
{
  "name": "PCR扩增",
  "workflowId": "wf_abc123",
  "assigneeId": "user_abc123",
  "schemaId": "task_schema_abc",
  "fields": {
    "template": {"value": "seq_abc123"},
    "priority": {"value": "高"}
  }
}
```

**更新任务：**
```http
PATCH /api/v2/tasks/{taskId}

请求体：
{
  "statusId": "status_complete_abc",
  "fields": {
    "completion_date": {"value": "2025-10-20"}
  }
}
```

### 文件夹

**列出文件夹：**
```http
GET /api/v2/folders

查询参数：
- projectId: 字符串
- parentFolderId: 字符串
```

**获取文件夹：**
```http
GET /api/v2/folders/{folderId}
```

**创建文件夹：**
```http
POST /api/v2/folders

请求体：
{
  "name": "2025年实验",
  "parentFolderId": "fld_parent_abc",
  "projectId": "proj_abc123"
}
```

### 项目

**列出项目：**
```http
GET /api/v2/projects
```

**获取项目：**
```http
GET /api/v2/projects/{projectId}
```

### 用户

**获取当前用户：**
```http
GET /api/v2/users/me
```

**列出用户：**
```http
GET /api/v2/users
```

**获取用户：**
```http
GET /api/v2/users/{userId}
```

### 团队

**列出团队：**
```http
GET /api/v2/teams
```

**获取团队：**
```http
GET /api/v2/teams/{teamId}
```

### 模式

**列出模式：**
```http
GET /api/v2/schemas

查询参数：
- entityType: 字符串（如 "dna_sequence", "custom_entity"）
```

**获取模式：**
```http
GET /api/v2/schemas/{schemaId}
```

### 注册表

**列出注册表：**
```http
GET /api/v2/registries
```

**获取注册表：**
```http
GET /api/v2/registries/{registryId}
```

## 批量操作

### 批量归档

**归档多个实体：**
```http
POST /api/v2/{entity-type}:archive

请求体：
{
  "{entity}Ids": ["id1", "id2", "id3"],
  "reason": "清理"
}
```

### 批量转移

**转移多个容器：**
```http
POST /api/v2/containers:bulk-transfer

请求体：
{
  "transfers": [
    {"containerId": "cont_1", "destinationId": "box_a"},
    {"containerId": "cont_2", "destinationId": "box_b"}
  ]
}
```

## 异步操作

部分操作返回任务 ID 用于异步处理：

**响应：**
```json
{
  "taskId": "task_abc123"
}
```

**检查任务状态：**
```http
GET /api/v2/tasks/{taskId}

响应：
{
  "id": "task_abc123",
  "status": "运行中", // 或 "成功", "失败"
  "message": "处理中...",
  "response": {...}  // 状态为"成功"时可用
}
```

## 字段值格式

自定义模式字段使用特定格式：

**简单值：**
```json
{
  "field_name": {
    "value": "字段值"
  }
}
```

**下拉菜单：**
```json
{
  "dropdown_field": {
    "value": "

- `X-RateLimit-Remaining`: 剩余请求次数
  - `X-RateLimit-Reset`: 限制重置的 Unix 时间戳

**处理 429 响应:**
```json
{
  "error": {
    "type": "RateLimitError",
    "message": "超出速率限制",
    "retryAfter": 5  // 需等待的秒数
  }
}
```

## 过滤与搜索

**常用查询参数:**
- `name`: 部分名称匹配
- `modifiedAt`: ISO 8601 时间格式
- `createdAt`: ISO 8601 时间格式
- `schemaId`: 按模式筛选
- `folderId`: 按文件夹筛选
- `archived`: 布尔值（是否包含归档项）

**示例:**
```bash
curl -X GET \
  "https://tenant.benchling.com/api/v2/dna-sequences?name=plasmid&folderId=fld_abc&archived=false"
```

## 最佳实践

### 请求效率

1. **使用合适的分页大小:**
   - 默认值: 50 项
   - 最大值: 100 项
   - 根据需求调整

2. **服务端过滤:**
   - 优先使用查询参数而非客户端过滤
   - 减少数据传输和处理开销

3. **批量操作:**
   - 尽可能使用批量端点
   - 单次请求内归档/转移多项内容

### 错误处理

```javascript
// 错误处理示例
async function fetchSequence(id) {
  try {
    const response = await fetch(
      `https://tenant.benchling.com/api/v2/dna-sequences/${id}`,
      {
        headers: {
          'Authorization': `Bearer ${token}`,
          'Accept': 'application/json'
        }
      }
    );

    if (!response.ok) {
      if (response.status === 429) {
        // 速率限制 - 采用退避策略重试
        const retryAfter = response.headers.get('Retry-After');
        await sleep(retryAfter * 1000);
        return fetchSequence(id);
      } else if (response.status === 404) {
        return null;  // 未找到
      } else {
        throw new Error(`API 错误: ${response.status}`);
      }
    }

    return await response.json();
  } catch (error) {
    console.error('请求失败:', error);
    throw error;
  }
}
```

### 分页循环

```javascript
async function getAllSequences() {
  let allSequences = [];
  let nextToken = null;

  do {
    const url = new URL('https://tenant.benchling.com/api/v2/dna-sequences');
    if (nextToken) {
      url.searchParams.set('nextToken', nextToken);
    }
    url.searchParams.set('pageSize', '100');

    const response = await fetch(url, {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Accept': 'application/json'
      }
    });

    const data = await response.json();
    allSequences = allSequences.concat(data.results);
    nextToken = data.nextToken;
  } while (nextToken);

  return allSequences;
}
```

## 参考资料

- **API 文档:** https://benchling.com/api/reference
- **交互式 API 浏览器:** https://your-tenant.benchling.com/api/reference (需认证)
- **更新日志:** https://docs.benchling.com/changelog
