---
name: open-notebook
description: 谷歌NotebookLM的开源自托管替代方案，用于AI驱动的研究和文档分析。适用于将研究材料整理成笔记本、摄取多样化内容源（PDF、视频、音频、网页、Office文档）、生成AI驱动的笔记和摘要、从研究材料创建多发言人播客、通过上下文感知AI与文档对话、使用全文和向量搜索跨材料检索，或运行自定义内容转换。支持16+家AI供应商（包括OpenAI、Anthropic、Google、Ollama、Groq和Mistral），通过自托管实现完全数据隐私。
license: MIT
metadata:
    skill-author: K-Dense Inc.
---

# Open Notebook

## 概述

Open Notebook是谷歌NotebookLM的开源自托管替代方案，帮助研究人员整理材料、生成AI驱动的洞察、创建播客，并与文档进行上下文感知对话——同时保持完全的数据隐私。

与谷歌Notebook LM（其企业版外无公开API）不同，Open Notebook提供完整的REST API，支持16+家AI供应商，并完全运行在您自己的基础设施上。

**相比NotebookLM的核心优势：**
- 完整的REST API支持程序化访问和自动化
- 可选用16+家AI供应商（不受限于谷歌模型）
- 支持1-4个可定制发言人的多发言人播客生成（突破2人限制）
- 通过自托管实现完全数据主权
- 开源且高度可扩展（MIT许可证）

**代码仓库：** https://github.com/lfnovo/open-notebook

## 快速入门

### 先决条件

- 已安装Docker Desktop
- 至少一个AI供应商的API密钥（或使用本地Ollama进行免费推理）

### 安装

使用Docker Compose部署Open Notebook：

```bash
# 下载docker-compose文件
curl -o docker-compose.yml https://raw.githubusercontent.com/lfnovo/open-notebook/main/docker-compose.yml

# 设置必需的加密密钥
export OPEN_NOTEBOOK_ENCRYPTION_KEY="your-secret-key-here"

# 启动服务
docker-compose up -d
```

访问应用：
- **前端界面：** http://localhost:8502
- **REST API：** http://localhost:5055
- **API文档：** http://localhost:5055/docs

### 配置AI供应商

启动后配置至少一个AI供应商：

1. 在UI中进入**设置 > API密钥**
2. 添加首选供应商（OpenAI、Anthropic等）的凭证
3. 测试连接并发现可用模型
4. 注册模型供全平台使用

或通过REST API配置：

```python
import requests

BASE_URL = "http://localhost:5055/api"

# 添加AI供应商凭证
response = requests.post(f"{BASE_URL}/credentials", json={
    "provider": "openai",
    "name": "My OpenAI Key",
    "api_key": "sk-..."
})
credential = response.json()

# 发现可用模型
response = requests.post(
    f"{BASE_URL}/credentials/{credential['id']}/discover"
)
discovered = response.json()

# 注册发现的模型
requests.post(
    f"{BASE_URL}/credentials/{credential['id']}/register-models",
    json={"model_ids": [m["id"] for m in discovered["models"]]}
)
```

## 核心功能

### 笔记本
将研究整理到独立笔记本中，每个笔记本包含来源、笔记和对话会话。

```python
import requests

BASE_URL = "http://localhost:5055/api"

# 创建笔记本
response = requests.post(f"{BASE_URL}/notebooks", json={
    "name": "癌症基因组学研究",
    "description": "肿瘤突变负荷文献综述"
})
notebook = response.json()
notebook_id = notebook["id"]
```

### 来源
摄取多样化内容类型，包括PDF、视频、音频文件、网页和Office文档。来源内容经过处理支持全文和向量搜索。

```python
# 添加网页URL来源
response = requests.post(f"{BASE_URL}/sources", data={
    "url": "https://arxiv.org/abs/2301.00001",
    "notebook_id": notebook_id,
    "process_async": "true"
})
source = response.json()

# 上传PDF文件
with open("paper.pdf", "rb") as f:
    response = requests.post(
        f"{BASE_URL}/sources",
        data={"notebook_id": notebook_id},
        files={"file": ("paper.pdf", f, "application/pdf")}
    )
```

### 笔记
创建和管理与笔记本关联的笔记（人工或AI生成）。

```python
# 创建人工笔记
response = requests.post(f"{BASE_URL}/notes", json={
    "title": "关键发现",
    "content": "在非小细胞肺癌中，TMB与免疫治疗反应相关...",
    "note_type": "human",
    "notebook_id": notebook_id
})
```

### 上下文感知对话
通过可引用来源的AI与您的研究材料对话。

```python
# 创建对话会话
session = requests.post(f"{BASE_URL}/chat/sessions", json={
    "notebook_id": notebook_id,
    "title": "TMB讨论"
}).json()

# 发送包含来源上下文的消息
response = requests.post(f"{BASE_URL}/chat/execute", json={
    "session_id": session["id"],
    "message": "免疫治疗反应的关键生物标志物有哪些？",
    "context": {"include_sources": True, "include_notes": True}
})
```

### 搜索
使用全文或向量（语义）搜索跨所有材料检索。

```python
# 在知识库中进行向量搜索
results = requests.post(f"{BASE_URL}/search", json={
    "query": "肿瘤突变负荷 免疫治疗",
    "search_type": "vector",
    "limit": 10
}).json()

# 通过AI获取问题答案
answer = requests.post(f"{BASE_URL}/search/ask/simple", json={
    "query": "TMB如何预测检查点抑制剂反应？"
}).json()
```

### 播客生成
从研究材料生成专业的1-4发言人定制播客。

```python
# 生成播客单集
job = requests.post(f"{BASE_URL}/podcasts/generate", json={
    "notebook_id": notebook_id,
    "episode_profile_id": episode_profile_id,
    "speaker_profile_ids": [speaker1_id, speaker2_id]
}).json()

# 检查生成状态
status = requests.get(f"{BASE_URL}/podcasts/jobs/{job['job_id']}").json()

# 完成后下载音频
audio = requests.get(
    f"{BASE_URL}/podcasts/episodes/{status['episode_id']}/audio"
)
```

### 内容转换
应用自定义AI驱动转换实现内容摘要、提取和分析。

```python
# 创建自定义转换
transform = requests.post(f"{BASE_URL}/transformations", json={
    "name": "extract_methods",
    "title": "提取方法",
    "description": "从论文中提取方法论细节",
    "prompt": "提取并总结方法论部分...",
    "apply_default": False
}).json()

# 对文本执行转换
result = requests.post(f"{BASE_URL}/transformations/execute", json={
    "transformation_id": transform["id"],
    "input_text": "...",
    "model_id": "model_id_here"
}).json()
```

## 支持的AI供应商

通过Esperanto库支持16+家AI供应商：

| 供应商 | 大语言模型 | 嵌入模型 | 语音转文本 | 文本转语音 |
|----------|-----|-----------|----------------|----------------|
| OpenAI | 是 | 是 | 是 | 是 |
| Anthropic | 是 | 否 | 否 | 否 |
| Google GenAI | 是 | 是 | 否 | 是 |
| Vertex AI | 是 | 是 | 否 | 是 |
| Ollama | 是 | 是 | 否 | 否 |
| Groq | 是 | 否 | 是 | 否 |
| Mistral | 是 | 是 | 否 | 否 |
| Azure OpenAI | 是 | 是 | 否 | 否 |
| DeepSeek | 是 | 否 | 否 | 否 |
| xAI | 是 | 否 | 否 | 否 |
| OpenRouter | 是 | 否 | 否 | 否 |
| ElevenLabs | 否 | 否 | 是 | 是 |
| Perplexity | 是 | 否 | 否 | 否 |
| Voyage | 否 | 是 | 否 | 否 |

## 环境变量

Docker部署关键配置变量：

| 变量 | 描述 | 默认值 |
|----------|-------------|---------|
| `OPEN_NOTEBOOK_ENCRYPTION_KEY` | **必需**。加密存储凭证的密钥 | 无 |
| `SURREAL_URL` | SurrealDB连接URL | `ws://surrealdb:8000/rpc` |
| `SURREAL_NAMESPACE` | 数据库命名空间 | `open_notebook` |
| `SURREAL_DATABASE` | 数据库名称 | `open_notebook` |
| `OPEN_NOTEBOOK_PASSWORD` | UI的密码保护（可选） | 无 |

## API参考

REST API地址：`http://localhost:5055/api`，交互式文档位于`/docs`。

核心端点组：
- `/api/notebooks` - 笔记本CRUD及来源关联
- `/api/sources` - 来源摄取、处理与检索
- `/api/notes` - 笔记管理
- `/api/chat/sessions` - 对话会话管理
- `/api/chat/execute` - 对话消息执行
- `/api/search` - 全文与向量搜索
- `/api/podcasts` - 播客生成与管理
- `/api/transformations` - 内容转换流水线
- `/api/models` - AI模型配置与发现
- `/api/credentials` - 供应商凭证管理

完整API参考（含所有端点和请求/响应格式）见`references/api_reference.md`。

## 架构

采用现代技术栈：
- **后端：** Python + FastAPI
- **数据库：** SurrealDB（文档+关系型）
- **AI集成：** LangChain + Esperanto多供应商库
- **前端：** Next.js + React
- **部署：** Docker Compose + 持久化存储卷

## 重要提示

- 部署需依赖Docker
- AI功能需配置至少一个供应商
- 免费本地推理推荐使用Ollama（无需API成本）
- `OPEN_NOTEBOOK_ENCRYPTION_KEY`必须在首次启动前设置且重启时保持一致
- 所有数据存储在Docker卷中实现完全数据主权
