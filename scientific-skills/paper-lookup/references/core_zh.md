# CORE API

CORE 聚合了全球 15,000 多个仓库的开放获取研究成果，提供 **3700 万+** 文章的全文内容及 **3.68 亿+** 论文的元数据。

## 基础 URL

```
https://api.core.ac.uk/v3
```

**重要提示：** GET 搜索路径需要**尾部斜杠**（例如 `/v3/search/works/` 而非 `/v3/search/works`）。

## 认证方式

- **请求头：** `Authorization: Bearer YOUR_API_KEY`
- **查询参数：** `?api_key=YOUR_API_KEY`
- 注册地址：https://core.ac.uk/services/api

**无认证状态：** 基础元数据查询可用，但全文内容不可获取（返回"Not available for public API users"提示）。

## 速率限制（基于令牌）

| 用户类型 | 每日令牌数 | 每分钟上限 |
|-----------|-------------|----------------|
| 未认证用户 | 100/天 | 10/分钟 |
| 注册个人用户 | 1,000/天 | 25/分钟 |
| 注册学术用户 | 5,000/天 | 10/分钟 |

简单查询消耗 1 个令牌，下载和滚动分页操作消耗 3-5 个令牌。

## 核心端点

### 1. 搜索研究成果

```
GET /v3/search/works/?q={query}&limit={n}&offset={n}
```

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `q` | 必填 | 搜索语句（支持字段检索、布尔运算符） |
| `limit` | 10 | 每页结果数（上限 100） |
| `offset` | 0 | 分页偏移量 |
| `scroll` | false | 启用滚动分页（获取超过 10,000 条结果） |
| `sort` | relevance | 排序方式：`relevance`（相关度）或 `recency`（时效性） |

**POST 替代方案**（适用于复杂查询）：
```
POST /v3/search/works
Content-Type: application/json

{"q": "machine learning", "limit": 10, "offset": 0}
```

**示例：**
```
https://api.core.ac.uk/v3/search/works/?q=CRISPR+gene+therapy&limit=10
```

### 2. 查询语法

| 运算符 | 示例 | 描述 |
|----------|---------|-------------|
| AND | `title:"AI" AND authors:"Smith"` | 同时满足两个条件 |
| OR | `title:"AI" OR fullText:"Deep Learning"` | 满足任一条件 |
| 分组 | `(title:"AI" OR title:"ML") AND yearPublished>"2020"` | 条件优先级 |
| 字段检索 | `title:"Machine Learning"` | 指定字段搜索 |
| 范围 | `yearPublished>2018` | 数值比较 |
| 存在性 | `_exists_:fullText` | 字段必须存在 |
| 短语 | `title:"Attention is all you need"` | 精确短语匹配 |

**可检索字段：** `abstract`, `arxivId`, `authors`, `contributors`, `createdDate`, `dataProviders`, `depositedDate`, `documentType`, `doi`, `fullText`, `id`, `language`, `license`, `oai`, `title`, `yearPublished`

### 3. 按 ID 获取研究成果

```
GET /v3/works/{id}
```

`id` 为 CORE 研究成果 ID（整数）。示例：`/v3/works/267312`

### 4. 按 ID 获取产出物

```
GET /v3/outputs/{id}
```

### 5. 下载全文

```
GET /v3/outputs/{id}/download
```

返回二进制 PDF 文件，需认证。

```
GET /v3/works/tei/{id}
```

返回 TEI XML 格式数据。

### 6. 搜索产出物

```
GET /v3/search/outputs/?q={query}&limit={n}&offset={n}
```

按 DOI 搜索：`q=doi:10.1038/nature12373`

## 响应格式

### 搜索响应
```json
{
  "totalHits": 2281337,
  "limit": 10,
  "offset": 0,
  "scrollId": null,
  "results": [...]
}
```

### 研究成果对象（关键字段）
```json
{
  "id": 8848131,
  "title": "Attention Is All You Need",
  "authors": [{"name": "Ashish Vaswani"}, ...],
  "abstract": "The dominant sequence...",
  "doi": "10.48550/arXiv.1706.03762",
  "arxivId": "1706.03762",
  "yearPublished": 2017,
  "downloadUrl": "https://core.ac.uk/download/...",
  "fullText": "Full text content (when authenticated)...",
  "language": {"code": "en", "name": "English"},
  "documentType": "research",
  "citationCount": 145678,
  "dataProviders": [{"name": "arXiv"}],
  "links": [{"type": "download", "url": "..."}]
}
```

## 分页机制

- **标准模式：** `offset` + `limit`（上限 10,000 条结果）
- **滚动模式：** 设置 `scroll=true`，响应包含 `scrollId` 用于后续请求（可突破 10,000 条限制，消耗更多令牌）

## 错误处理

高负载时 API 可能返回分片故障提示，此类错误为瞬时状态——请稍后重试。
