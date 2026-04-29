# Parallel Web Systems API 快速参考

**完整文档：** https://docs.parallel.ai  
**API密钥：** https://platform.parallel.ai  
**Python SDK：** `pip install parallel-web`  
**环境变量：** `PARALLEL_API_KEY`

---

## 搜索API（测试版）

**端点：** `POST https://api.parallel.ai/v1beta/search`  
**请求头：** `parallel-beta: search-extract-2025-10-10`

### 请求

```json
{
  "objective": "自然语言搜索目标（最多5000字符）",
  "search_queries": ["关键词查询1", "关键词查询2"],
  "max_results": 10,
  "excerpts": {
    "max_chars_per_result": 10000,
    "max_chars_total": 50000
  },
  "source_policy": {
    "allow_domains": ["example.com"],
    "deny_domains": ["spam.com"],
    "after_date": "2024-01-01"
  }
}
```

### 响应

```json
{
  "search_id": "search_...",
  "results": [
    {
      "url": "https://...",
      "title": "页面标题",
      "publish_date": "2025-01-15",
      "excerpts": ["相关内容..."]
    }
  ]
}
```

### Python SDK

```python
from parallel import Parallel
client = Parallel(api_key="...")
result = client.beta.search(
    objective="...",
    search_queries=["..."],
    max_results=10,
    excerpts={"max_chars_per_result": 10000},
)
```

**费用：** 每1000次请求5美元（默认每次10个结果）  
**速率限制：** 600次请求/分钟

---

## 提取API（测试版）

**端点：** `POST https://api.parallel.ai/v1beta/extract`  
**请求头：** `parallel-beta: search-extract-2025-10-10`

### 请求

```json
{
  "urls": ["https://example.com/page"],
  "objective": "需要聚焦的内容",
  "excerpts": true,
  "full_content": false
}
```

### 响应

```json
{
  "extract_id": "extract_...",
  "results": [
    {
      "url": "https://...",
      "title": "页面标题",
      "excerpts": ["聚焦内容..."],
      "full_content": null
    }
  ],
  "errors": []
}
```

### Python SDK

```python
result = client.beta.extract(
    urls=["https://..."],
    objective="...",
    excerpts=True,
    full_content=False,
)
```

**费用：** 每1000个URL 1美元  
**速率限制：** 600次请求/分钟

---

## 任务API（深度研究）

**端点：** `POST https://api.parallel.ai/v1/tasks/runs`

### 创建任务运行

```json
{
  "input": "研究问题（最多15,000字符）",
  "processor": "pro-fast",
  "task_spec": {
    "output_schema": {
      "type": "text"
    }
  }
}
```

### 响应（即时）

```json
{
  "run_id": "trun_...",
  "status": "queued"
}
```

### 获取结果（阻塞式）

**端点：** `GET https://api.parallel.ai/v1/tasks/runs/{run_id}/result`

### Python SDK

```python
# 文本输出（带引用的Markdown报告）
from parallel.types import TaskSpecParam
task_run = client.task_run.create(
    input="研究问题",
    processor="pro-fast",
    task_spec=TaskSpecParam(output_schema={"type": "text"}),
)
result = client.task_run.result(task_run.run_id, api_timeout=3600)
print(result.output.content)

# 自动模式输出（结构化JSON）
task_run = client.task_run.create(
    input="研究问题",
    processor="pro-fast",
)
result = client.task_run.result(task_run.run_id, api_timeout=3600)
print(result.output.content)  # 结构化字典
print(result.output.basis)    # 字段引用依据
```

### 处理器

| 处理器 | 延迟 | 费用/1000 | 最佳适用场景 |
|-----------|---------|-----------|----------|
| `lite-fast` | 10-20秒 | 5美元 | 基础元数据 |
| `base-fast` | 15-50秒 | 10美元 | 标准增强 |
| `core-fast` | 15秒-100秒 | 25美元 | 交叉引用 |
| `core2x-fast` | 15秒-3分钟 | 50美元 | 高复杂度 |
| **`pro-fast`** | **30秒-5分钟** | **100美元** | **默认：探索性研究** |
| `ultra-fast` | 1-10分钟 | 300美元 | 多源深度研究 |
| `ultra2x-fast` | 1-20分钟 | 600美元 | 困难研究 |
| `ultra4x-fast` | 1-40分钟 | 1200美元 | 极困难研究 |
| `ultra8x-fast` | 1小时 | 2400美元 | 最高难度研究 |

标准版（非fast）处理器费用相同，但延迟更高且使用最新数据。

---

## 聊天API（测试版）

**端点：** `POST https://api.parallel.ai/chat/completions`  
**兼容OpenAI SDK。**

### 模型

| 模型 | 延迟（TTFT） | 费用/1000 | 使用场景 |
|-------|----------------|-----------|----------|
| `speed` | ~3秒 | 5美元 | 低延迟聊天 |
| `lite` | 10-60秒 | 5美元 | 带依据的简单查询 |
| `base` | 15-100秒 | 10美元 | 带依据的标准研究 |
| `core` | 1-5分钟 | 25美元 | 带依据的复杂研究 |

### Python SDK（兼容OpenAI）

```python
from openai import OpenAI
client = OpenAI(
    api_key="PARALLEL_API_KEY",
    base_url="https://api.parallel.ai",
)
response = client.chat.completions.create(
    model="speed",
    messages=[{"role": "user", "content": "什么是Parallel Web Systems？"}],
)
```

---

## 速率限制

| API | 默认限制 |
|-----|---------------|
| 搜索 | 600次/分钟 |
| 提取 | 600次/分钟 |
| 聊天 | 300次/分钟 |
| 任务 | 随处理器变化 |

---

## 来源策略

控制搜索使用的来源：

```json
{
  "source_policy": {
    "allow_domains": ["nature.com", "science.org"],
    "deny_domains": ["unreliable-source.com"],
    "after_date": "2024-01-01"
  }
}
```

适用于搜索API，可聚焦特定权威域名结果。
