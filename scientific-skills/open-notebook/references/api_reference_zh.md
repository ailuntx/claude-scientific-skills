# Open Notebook API 参考

## 基础 URL

```
http://localhost:5055/api
```

交互式 API 文档可通过 `http://localhost:5055/docs` (Swagger UI) 和 `http://localhost:5055/redoc` (ReDoc) 访问。

## 认证

若配置了 `OPEN_NOTEBOOK_PASSWORD`，请在请求中包含密码。以下路由免认证：`/`, `/health`, `/docs`, `/openapi.json`, `/redoc`, `/api/auth/status`, `/api/config`。

---

## 笔记本

### 列出笔记本

```
GET /api/notebooks
```

**查询参数：**
| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `archived` | boolean | 按归档状态过滤 |
| `order_by` | string | 排序字段（默认：`updated_at`） |

**响应：** 包含 `source_count` 和 `note_count` 的笔记本对象数组。

### 创建笔记本

```
POST /api/notebooks
```

**请求体：**
```json
{
  "name": "我的研究",
  "description": "可选描述"
}
```

### 获取笔记本

```
GET /api/notebooks/{notebook_id}
```

### 更新笔记本

```
PUT /api/notebooks/{notebook_id}
```

**请求体：**
```json
{
  "name": "更新名称",
  "description": "更新描述",
  "archived": false
}
```

### 删除笔记本

```
DELETE /api/notebooks/{notebook_id}
```

**查询参数：**
| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `delete_sources` | boolean | 同时删除专属资源（默认：false） |

### 删除预览

```
GET /api/notebooks/{notebook_id}/delete-preview
```

返回删除操作将影响的笔记和资源数量统计。

### 关联资源到笔记本

```
POST /api/notebooks/{notebook_id}/sources/{source_id}
```

将资源关联到笔记本的幂等操作。

### 解除资源与笔记本关联

```
DELETE /api/notebooks/{notebook_id}/sources/{source_id}
```

---

## 资源

### 列出资源

```
GET /api/sources
```

**查询参数：**
| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `notebook_id` | string | 按笔记本过滤 |
| `limit` | integer | 返回数量 |
| `offset` | integer | 分页偏移量 |
| `order_by` | string | 排序字段 |

### 创建资源

```
POST /api/sources
```

接受文件上传的多部分表单数据，或用于URL/文本资源的JSON。

**表单参数：**
| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `file` | file | 上传文件（PDF, DOCX, 音频, 视频） |
| `url` | string | 待提取的网页URL |
| `text` | string | 原始文本内容 |
| `notebook_id` | string | 关联笔记本 |
| `process_async` | boolean | 异步处理（默认：true） |

### 创建资源（JSON）

```
POST /api/sources/json
```

基于JSON的旧版资源创建端点。

### 获取资源

```
GET /api/sources/{source_id}
```

### 获取资源状态

```
GET /api/sources/{source_id}/status
```

轮询异步提取资源的处理状态。

### 更新资源

```
PUT /api/sources/{source_id}
```

**请求体：**
```json
{
  "title": "更新标题",
  "topic": "更新主题"
}
```

### 删除资源

```
DELETE /api/sources/{source_id}
```

### 下载资源文件

```
GET /api/sources/{source_id}/download
```

返回原始上传文件。

### 检查资源文件

```
HEAD /api/sources/{source_id}/download
```

### 重试失败资源

```
POST /api/sources/{source_id}/retry
```

将失败资源重新加入处理队列。

### 获取资源洞察

```
GET /api/sources/{source_id}/insights
```

获取资源的AI生成洞察。

---

## 笔记

### 列出笔记

```
GET /api/notes
```

**查询参数：**
| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `notebook_id` | string | 按笔记本过滤 |

### 创建笔记

```
POST /api/notes
```

**请求体：**
```json
{
  "title": "我的笔记",
  "content": "笔记内容...",
  "note_type": "human",
  "notebook_id": "notebook:abc123"
}
```

`note_type` 必须为 `"human"` 或 `"ai"`。无标题的AI笔记会自动生成标题。

### 获取笔记

```
GET /api/notes/{note_id}
```

### 更新笔记

```
PUT /api/notes/{note_id}
```

**请求体：**
```json
{
  "title": "更新标题",
  "content": "更新内容",
  "note_type": "human"
}
```

### 删除笔记

```
DELETE /api/notes/{note_id}
```

---

## 聊天

### 列出会话

```
GET /api/chat/sessions
```

**查询参数：**
| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `notebook_id` | string | 按笔记本过滤 |

### 创建会话

```
POST /api/chat/sessions
```

**请求体：**
```json
{
  "notebook_id": "notebook:abc123",
  "title": "讨论主题",
  "model_override": "optional_model_id"
}
```

### 获取会话

```
GET /api/chat/sessions/{session_id}
```

返回包含消息历史的会话详情。

### 更新会话

```
PUT /api/chat/sessions/{session_id}
```

### 删除会话

```
DELETE /api/chat/sessions/{session_id}
```

### 执行聊天

```
POST /api/chat/execute
```

**请求体：**
```json
{
  "session_id": "chat_session:abc123",
  "message": "您的问题",
  "context": {
    "include_sources": true,
    "include_notes": true
  },
  "model_override": "optional_model_id"
}
```

### 构建上下文

```
POST /api/chat/context
```

为聊天会话从资源和笔记构建上下文数据。

---

## 搜索

### 搜索知识库

```
POST /api/search
```

**请求体：**
```json
{
  "query": "搜索词",
  "search_type": "vector",
  "limit": 10,
  "source_ids": [],
  "note_ids": [],
  "min_similarity": 0.7
}
```

`search_type` 可为 `"vector"`（需嵌入模型）或 `"text"`（关键词匹配）。

### 流式问答

```
POST /api/search/ask
```

返回基于知识库内容的AI生成答案的服务器发送事件（SSE）。

### 简易问答

```
POST /api/search/ask/simple
```

返回完整响应的非流式版本。

---

## 播客

### 生成播客

```
POST /api/podcasts/generate
```

**请求体：**
```json
{
  "notebook_id": "notebook:abc123",
  "episode_profile_id": "episode_profile:xyz",
  "speaker_profile_ids": ["speaker:a", "speaker:b"]
}
```

返回用于跟踪生成进度的 `job_id`。

### 获取任务状态

```
GET /api/podcasts/jobs/{job_id}
```

### 列出剧集

```
GET /api/podcasts/episodes
```

### 获取剧集

```
GET /api/podcasts/episodes/{episode_id}
```

### 获取剧集音频

```
GET /api/podcasts/episodes/{episode_id}/audio
```

流式传输播客音频文件。

### 重试失败剧集

```
POST /api/podcasts/episodes/{episode_id}/retry
```

### 删除剧集

```
DELETE /api/podcasts/episodes/{episode_id}
```

---

## 转换

### 列出转换

```
GET /api/transformations
```

### 创建转换

```
POST /api/transformations
```

**请求体：**
```json
{
  "name": "summarize",
  "title": "内容摘要",
  "description": "生成简明摘要",
  "prompt": "总结以下文本...",
  "apply_default": false
}
```

### 执行转换

```
POST /api/transformations/execute
```

**请求体：**
```json
{
  "transformation_id": "transformation:abc",
  "input_text": "待转换文本...",
  "model_id": "model:xyz"
}
```

### 获取默认提示

```
GET /api/transformations/default-prompt
```

### 更新默认提示

```
PUT /api/transformations/default-prompt
```

### 获取转换

```
GET /api/transformations/{transformation_id}
```

### 更新转换

```
PUT /api/transformations/{transformation_id}
```

### 删除转换

```
DELETE /api/transformations/{transformation_id}
```

---

## 模型

### 列出模型

```
GET /api/models
```

**查询参数：**
| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `model_type` | string | 按类型过滤（llm, embedding, stt, tts） |

### 创建模型

```
POST /api/models
```

### 删除模型

```
DELETE /api/models/{model_id}
```

### 测试模型

```
POST /api/models/{model_id}/test
```

### 获取默认模型

```
GET /api/models/defaults
```

返回七个服务槽位的默认模型分配：chat（聊天）, transformation（转换）, embedding（嵌入）, speech-to-text（语音转文本）, text-to-speech（文本转语音）, podcast（播客）, summary（摘要）。

### 更新默认模型

```
PUT /api/models/defaults
```

### 获取提供商

```
GET /api/models/providers
```

### 发现模型

```
GET /api/models/discover/{provider}
```

### 同步模型（单提供商）

```
POST /api/models/sync/{provider}
```

### 同步所有模型

```
POST /api/models/sync
```

### 自动分配默认值

```
POST /api/models/auto-assign
```

根据提供商优先级自动填充空默认模型槽位。

### 获取模型数量

```
GET /api/models/count/{provider}
```

### 按提供商获取模型

```
GET /api/models/by-provider/{provider}
```

---

## 凭证

### 获取状态

```
GET /api/credentials/status
```

### 获取环境状态

```
GET /api/credentials/env-status
```

### 列出凭证

```
GET /api/credentials
```

**查询参数：**
| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `provider` | string | 按提供商过滤 |

### 按提供商列出

```
GET /api/credentials/by-provider/{provider}
```

### 创建凭证

```
POST /api/credentials
```

**请求体：**
```json
{
  "provider": "openai",
  "name": "我的OpenAI密钥",
  "api_key": "sk-...",
  "base_url": null
}
```

### 获取凭证

```
GET /api/credentials/{credential_id}
```

注：API密钥值永不返回。

### 更新凭证

```
PUT /api/credentials/{credential_id}
```

### 删除凭证

```
DELETE /api/credentials/{credential_id}
```

### 测试凭证

```
POST /api/credentials/{credential_id}/test
```

### 通过凭证发现模型

```
POST /api/credentials/{credential_id}/discover
```

### 通过凭证注册模型

```
POST /api/credentials/{credential_id}/register-models
```

---

## 错误响应

API返回标准HTTP状态码及JSON错误体：

| 状态码 | 含义 |
|--------|---------|
| 400 | 无效输入 |
| 401 | 需要认证 |
| 404 | 资源未找到 |
| 422 | 配置错误 |
| 429 | 速率限制 |
| 500 | 服务器内部错误 |
| 502 | 外部服务错误 |

**错误响应格式：**
```json
{
  "detail": "错误描述"
}
```
