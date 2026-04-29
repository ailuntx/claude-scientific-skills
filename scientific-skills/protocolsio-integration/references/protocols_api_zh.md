# 协议 API

## 概述

协议 API 是 protocols.io 的核心功能，支持从创建到发布的完整协议生命周期。包括搜索、创建、更新、管理步骤、处理材料、书签管理以及生成 PDF。

## 基础 URL

所有协议端点使用的基础 URL：`https://protocols.io/api/v3`

## 内容格式参数

许多端点支持 `content_format` 参数来指定返回内容的格式：

- `json`：Draft.js JSON 格式（默认）
- `html`：HTML 格式
- `markdown`：Markdown 格式

作为查询参数包含：`?content_format=html`

## 列表与搜索操作

### 列出协议

通过筛选和分页检索协议。

**端点：** `GET /protocols`

**查询参数：**
- `filter`：筛选类型
  - `public`：仅公开协议
  - `private`：您的私有协议
  - `shared`：与您共享的协议
  - `user_public`：其他用户的公开协议
- `key`：在协议标题、描述和内容中搜索关键词
- `order_field`：排序字段（`activity`、`created_on`、`modified_on`、`name`、`id`）
- `order_dir`：排序方向（`desc`、`asc`）
- `page_size`：每页结果数（默认：10，最大：50）
- `page_id`：分页页码（从 0 开始）
- `fields`：要返回的字段列表（逗号分隔）
- `content_format`：内容格式（`json`、`html`、`markdown`）

**示例请求：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://protocols.io/api/v3/protocols?filter=public&key=CRISPR&page_size=20&content_format=html"
```

### 通过 DOI 搜索

通过 DOI 检索协议。

**端点：** `GET /protocols/{doi}`

**路径参数：**
- `doi`：协议 DOI（例如 `dx.doi.org/10.17504/protocols.io.xxxxx`）

## 获取协议详情

### 通过 ID 获取协议

**端点：** `GET /protocols/{protocol_id}`

**路径参数：**
- `protocol_id`：协议的唯一标识符

**查询参数：**
- `content_format`：内容格式（`json`、`html`、`markdown`）

**响应包含：**
- 协议元数据（标题、作者、描述、DOI）
- 所有协议步骤及内容
- 材料与试剂
- 指南与警告
- 版本信息
- 发布状态

## 创建与更新协议

### 创建新协议

**端点：** `POST /protocols`

**请求体参数：**
- `title`（必填）：协议标题
- `description`：协议描述
- `tags`：标签字符串数组
- `vendor_name`：供应商/公司名称
- `vendor_link`：供应商网站 URL
- `warning`：警告或安全信息
- `guidelines`：使用指南
- `manuscript_citation`：相关手稿引用
- `link`：相关资源的外部链接

**示例请求：**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "CRISPR 基因编辑协议",
    "description": "CRISPR-Cas9 介导的基因编辑完整协议",
    "tags": ["CRISPR", "基因编辑", "分子生物学"]
  }' \
  "https://protocols.io/api/v3/protocols"
```

### 更新协议

**端点：** `PATCH /protocols/{protocol_id}`

**路径参数：**
- `protocol_id`：协议的唯一标识符

**请求体**：与创建参数相同，均为可选

## 协议步骤管理

### 创建协议步骤

**端点：** `POST /protocols/{protocol_id}/steps`

**请求体参数：**
- `title`（必填）：步骤标题
- `description`：步骤描述（HTML、Markdown 或 Draft.js JSON）
- `duration`：步骤持续时间（秒）
- `temperature`：温度设置
- `components`：使用的材料/试剂数组
- `software`：所需软件或工具
- `commands`：要执行的命令
- `expected_result`：预期结果描述

**示例请求：**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "制备 sgRNA",
    "description": "设计并合成靶向目标基因的单向导 RNA (sgRNA)",
    "duration": 3600,
    "temperature": 25
  }' \
  "https://protocols.io/api/v3/protocols/12345/steps"
```

### 更新协议步骤

**端点：** `PATCH /protocols/{protocol_id}/steps/{step_id}`

**参数**：与创建步骤相同，均为可选

### 删除协议步骤

**端点：** `DELETE /protocols/{protocol_id}/steps/{step_id}`

### 重新排序步骤

**端点：** `POST /protocols/{protocol_id}/steps/reorder`

**请求体：**
- `step_order`：按所需顺序排列的步骤 ID 数组

## 材料与试剂

### 获取协议材料

检索协议中使用的所有材料与试剂。

**端点：** `GET /protocols/{protocol_id}/materials`

**响应包含：**
- 试剂名称与描述
- 目录编号
- 供应商信息
- 浓度与用量
- 产品页面链接

## 发布与 DOI

### 发布协议

分配 DOI 并使协议公开可用。

**端点：** `POST /protocols/{protocol_id}/publish`

**请求体参数：**
- `version_notes`：此版本变更说明
- `publish_type`：发布类型
  - `new`：首次发布
  - `update`：更新现有已发布协议

**重要说明：**
- 发布后协议将获得永久 DOI
- 已发布协议不可删除，只能通过新版本更新
- 已发布协议可公开访问

**示例请求：**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "version_notes": "初始发布",
    "publish_type": "new"
  }' \
  "https://protocols.io/api/v3/protocols/12345/publish"
```

## 书签

### 添加书签

将协议添加到书签以便快速访问。

**端点：** `POST /protocols/{protocol_id}/bookmarks`

### 移除书签

**端点：** `DELETE /protocols/{protocol_id}/bookmarks`

### 列出已加书签的协议

**端点：** `GET /bookmarks`

## PDF 导出

### 生成协议 PDF

生成格式化的协议 PDF 版本。

**端点：** `GET /view/{protocol_uri}.pdf`

**查询参数：**
- `compact`：设置为 `1` 可启用紧凑视图（无大间距）

**速率限制：**
- 已登录用户：每分钟 5 次请求
- 未登录用户：每分钟 3 次请求

**示例：**
```
https://protocols.io/api/v3/view/crispr-protocol-abc123.pdf?compact=1
```

## 常见用例

### 1. 导入现有协议

导入并使用现有协议：

1. 使用关键词或 DOI 搜索协议
2. 通过 `/protocols/{protocol_id}` 获取完整协议详情
3. 提取步骤、材料和元数据供本地使用

### 2. 从零创建新协议

创建新协议：

1. 创建含标题和描述的协议：`POST /protocols`
2. 顺序添加步骤：`POST /protocols/{id}/steps`
3. 审查并测试协议
4. 准备就绪后发布：`POST /protocols/{id}/publish`

### 3. 更新已发布协议

更新已发布的协议：

1. 获取当前版本：`GET /protocols/{protocol_id}`
2. 进行必要更新：`PATCH /protocols/{protocol_id}`
3. 按需更新或添加步骤
4. 发布新版本：`POST /protocols/{protocol_id}/publish` 并指定 `publish_type: "update"`

### 4. 克隆并修改协议

创建现有协议的修改版本：

1. 获取原始协议详情
2. 使用修改后的元数据创建新协议
3. 复制并修改原始协议的步骤
4. 作为新协议发布

## 错误处理

常见错误响应：

- `400 Bad Request`：无效参数或请求格式
- `401 Unauthorized`：访问令牌缺失或无效
- `403 Forbidden`：操作权限不足
- `404 Not Found`：协议或资源未找到
- `429 Too Many Requests`：超出速率限制
- `500 Internal Server Error`：服务器端错误

针对 `429` 和 `500` 错误，实施指数退避的重试逻辑。
