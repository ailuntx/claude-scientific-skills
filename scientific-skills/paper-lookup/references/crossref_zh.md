# Crossref API

Crossref 是学术内容的 DOI 注册机构，提供超过 1.5 亿条作品的元数据，涵盖期刊文章、书籍、会议论文、数据集和预印本。

## 基础 URL

```
https://api.crossref.org
```

## 认证

无需认证。添加 `mailto=you@example.com` 可进入**礼貌池**（速率限制提升 2 倍）。

## 速率限制

| 池类型 | 速率 | 并发数 |
|------|------|-------------|
| 公共池 (无 mailto) | 5 次/秒 | 1 并发 |
| 礼貌池 (含 mailto) | 10 次/秒 | 3 并发 |

HTTP 429 状态码表示临时封禁。

## 核心端点

### 1. 检索作品

```
GET /works?query={text}&rows={n}&mailto=you@example.com
```

| 参数 | 默认值 | 说明 |
|-----------|---------|-------------|
| `query` | -- | 全字段自由文本检索 |
| `query.author` | -- | 作者姓名检索 |
| `query.bibliographic` | -- | 标题/作者/ISSN/年份检索 |
| `query.affiliation` | -- | 机构隶属关系检索 |
| `query.container-title` | -- | 期刊名称检索 |
| `filter` | -- | 逗号分隔的 `name:value` 键值对 |
| `sort` | `score` | 排序字段：`score`, `published`, `issued`, `deposited`, `updated`, `is-referenced-by-count`, `references-count` |
| `order` | `desc` | 排序方向：`asc` 或 `desc` |
| `rows` | 20 | 每页结果数（上限 1000） |
| `offset` | 0 | 跳过结果数（上限 10,000） |
| `cursor` | -- | 使用 `*` 进行游标式深度分页 |
| `select` | -- | 逗号分隔的返回字段名 |
| `facet` | -- | 分面统计，如 `type-name:10` |
| `sample` | -- | 返回随机条目数（上限 100） |

**示例：**
```
https://api.crossref.org/works?query=CRISPR+gene+therapy&filter=from-pub-date:2024-01-01,type:journal-article,has-abstract:true&rows=5&sort=published&order=desc&mailto=you@example.com
```

### 2. 按 DOI 获取作品

```
GET /works/{doi}?mailto=you@example.com
```

需对 DOI 进行 URL 编码：`10.1038/nature12373` 转为 `10.1038%2Fnature12373`

**示例：**
```
https://api.crossref.org/works/10.1038%2Fnature12373?mailto=you@example.com
```

### 3. 期刊

```
GET /journals?query={name}&rows={n}
GET /journals/{issn}
GET /journals/{issn}/works?query={text}&rows={n}
```

### 4. 资助机构

```
GET /funders?query={name}
GET /funders/{id}
GET /funders/{id}/works?rows={n}
```

资助机构 ID 来自资助者注册库（如 NSF 对应 `100000001`）。

### 5. 会员（出版商）

```
GET /members?query={name}
GET /members/{id}/works?rows={n}
```

## 核心筛选器

### 日期筛选器（支持 `YYYY`, `YYYY-MM`, `YYYY-MM-DD`）
| 筛选器 | 说明 |
|--------|-------------|
| `from-pub-date` / `until-pub-date` | 出版日期 |
| `from-print-pub-date` / `until-print-pub-date` | 印刷出版日期 |
| `from-online-pub-date` / `until-online-pub-date` | 在线出版日期 |
| `from-posted-date` / `until-posted-date` | 预印本发布日期 |

### 布尔筛选器
| 筛选器 | 说明 |
|--------|-------------|
| `has-abstract` | 含摘要 |
| `has-orcid` | 含 ORCID 标识符 |
| `has-funder` | 含资助信息 |
| `has-full-text` | 含全文链接 |
| `has-references` | 含参考文献列表 |
| `has-license` | 含许可信息 |

### 值筛选器
| 筛选器 | 说明 |
|--------|-------------|
| `type` | 类型：`journal-article`, `posted-content`, `book-chapter`, `proceedings-article` 等 |
| `issn` | 期刊 ISSN |
| `doi` | 特定 DOI |
| `orcid` | 贡献者 ORCID |
| `funder` | 资助机构注册 ID |
| `member` | Crossref 会员 ID |
| `prefix` | DOI 前缀 |
| `license.url` | 许可协议 URL |
| `update-type` | 更新类型：`correction`, `retraction` |

**语法：** `filter=name1:value1,name2:value2`

## 分页机制

### 偏移分页（上限 10,000）
```
/works?query=cancer&rows=100&offset=200
```

### 游标分页（无上限）
1. 首次请求：`?cursor=*&rows=100`
2. 响应含 `next-cursor` 字段
3. 后续请求：`?cursor={next-cursor-value}&rows=100`
4. 游标有效期 5 分钟

## 响应格式

### 列表响应
```json
{
  "status": "ok",
  "message-type": "work-list",
  "message": {
    "total-results": 2779116,
    "items-per-page": 20,
    "next-cursor": "...",
    "items": [...]
  }
}
```

### 作品对象（核心字段）
```json
{
  "DOI": "10.1038/nature12373",
  "title": ["Nanometre-scale thermometry in a living cell"],
  "author": [{"given": "G.", "family": "Kucsko", "sequence": "first"}],
  "publisher": "Springer Science and Business Media LLC",
  "type": "journal-article",
  "published": {"date-parts": [[2013, 7, 31]]},
  "container-title": ["Nature"],
  "ISSN": ["0028-0836", "1476-4687"],
  "volume": "500",
  "issue": "7460",
  "page": "54-58",
  "is-referenced-by-count": 1745,
  "references-count": 30,
  "abstract": "<p>Abstract text with HTML tags...</p>",
  "license": [{"URL": "...", "content-version": "vor"}],
  "link": [{"URL": "...", "content-type": "application/pdf"}],
  "reference": [{"key": "...", "doi-asserted-by": "crossref", "DOI": "..."}],
  "subject": ["Multidisciplinary"],
  "language": "en"
}
```

注：`title` 和 `container-title` 为数组格式；`published.date-parts` 格式为 `[[年, 月, 日]]`；摘要可能包含 HTML 标签。
