# Semantic Scholar API

Semantic Scholar 通过人工智能功能索引了超过2亿篇涵盖所有学术领域的论文：引用上下文、高影响力引用、摘要（TLDR）和论文推荐。

## 基础URL

```
https://api.semanticscholar.org/graph/v1       (学术图谱)
https://api.semanticscholar.org/recommendations/v1  (推荐系统)
```

## 认证

- **无密钥：** 共享速率池（频繁触发429错误）。可用但不稳定。
- **使用密钥：** 每个密钥每秒1次请求（可申请更高配额）。
- 请求头：`x-api-key: YOUR_KEY`
- 免费获取密钥：https://www.semanticscholar.org/product/api#api-key-form

## `fields` 参数

几乎所有端点都接受 `fields` 参数——这是一个不含空格的逗号分隔字段列表。未指定时仅返回 `paperId` + `title`。

**论文字段：**
`paperId`, `corpusId`, `externalIds`, `url`, `title`, `abstract`, `venue`, `publicationVenue`, `year`, `referenceCount`, `citationCount`, `influentialCitationCount`, `isOpenAccess`, `openAccessPdf`, `fieldsOfStudy`, `s2FieldsOfStudy`, `publicationTypes`, `publicationDate`, `journal`, `authors`, `citations`, `references`, `tldr`, `embedding`

**作者字段：**
`authorId`, `externalIds`, `url`, `name`, `affiliations`, `homepage`, `paperCount`, `citationCount`, `hIndex`, `papers`

## 论文ID格式

`{paper_id}` 参数接受：
- `649def34f8be52c8b66281af98ae884c09aef38b` (S2哈希值)
- `CorpusId:215416146`
- `DOI:10.1038/s41586-021-03819-2`
- `ARXIV:2005.14165`
- `PMID:19872477`
- `PMCID:2323736`
- `ACL:W12-3903`

## 关键端点

### 1. 论文搜索（相关性）

```
GET /graph/v1/paper/search?query={text}&fields={fields}&offset={n}&limit={n}
```

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `query` | 必填 | 纯文本搜索 |
| `fields` | paperId,title | 逗号分隔 |
| `offset` | 0 | 分页起始位置 |
| `limit` | 100 | 最大100 |
| `year` | -- | `2019` 或 `2016-2020` |
| `publicationDateOrYear` | -- | `YYYY-MM-DD:YYYY-MM-DD` |
| `fieldsOfStudy` | -- | 例如 `Computer Science,Medicine` |
| `publicationTypes` | -- | 例如 `JournalArticle,Conference` |
| `openAccessPdf` | -- | 筛选开放获取论文 |
| `minCitationCount` | -- | 最小引用次数 |
| `venue` | -- | 逗号分隔的出版场所 |

通过偏移量最多可访问 **1,000条结果**。

**示例：**
```
https://api.semanticscholar.org/graph/v1/paper/search?query=CRISPR+gene+therapy&fields=title,year,abstract,citationCount,authors,openAccessPdf&limit=10&year=2023-2024
```

### 2. 批量论文搜索（布尔查询，大型结果集）

```
GET /graph/v1/paper/search/bulk?query={text}&fields={fields}&sort={field}:{order}&token={token}
```

- 支持布尔运算符：`+` (AND), `|` (OR), `-` (NOT), `"..."` (短语), `*` (通配符), `()` (分组)
- 基于令牌的分页（最多1000万篇论文）
- 每次返回最多1,000条
- 可排序：`citationCount:desc`, `publicationDate:desc`, `paperId:asc`

### 3. 论文详情（按ID）

```
GET /graph/v1/paper/{paper_id}?fields={fields}
```

**示例：**
```
https://api.semanticscholar.org/graph/v1/paper/DOI:10.1038/s41586-021-03819-2?fields=title,year,abstract,citationCount,referenceCount,isOpenAccess,openAccessPdf,authors,tldr
```

**响应：**
```json
{
  "paperId": "dc32a984b651256a8ec282be52310e6bd33d9815",
  "title": "Highly accurate protein structure prediction with AlphaFold",
  "year": 2021,
  "citationCount": 34260,
  "isOpenAccess": true,
  "openAccessPdf": {"url": "https://...pdf", "status": "HYBRID"},
  "tldr": {"text": "This work develops AlphaFold, a system that..."},
  "authors": [{"authorId": "47921134", "name": "J. Jumper"}, ...]
}
```

### 4. 论文引用

```
GET /graph/v1/paper/{paper_id}/citations?fields={fields}&offset={n}&limit={n}
```

返回引用该论文的文献。`limit` 最大1000。

引用特定字段：`contexts`, `intents`, `isInfluential`

### 5. 论文参考文献

```
GET /graph/v1/paper/{paper_id}/references?fields={fields}&offset={n}&limit={n}
```

返回该论文引用的文献。分页规则同引用接口。

### 6. 论文标题匹配

```
GET /graph/v1/paper/search/match?query={exact title}&fields={fields}
```

返回最佳匹配及`matchScore`。若无匹配则返回404。

### 7. 作者搜索

```
GET /graph/v1/author/search?query={name}&fields={fields}&offset={n}&limit={n}
```

### 8. 作者详情

```
GET /graph/v1/author/{author_id}?fields={fields}
```

### 9. 作者论文列表

```
GET /graph/v1/author/{author_id}/papers?fields={fields}&offset={n}&limit={n}
```

### 10. 论文推荐

```
GET /recommendations/v1/papers/forpaper/{paper_id}?fields={fields}&limit={n}&from={pool}
```

`from`：`recent` (默认) 或 `all-cs`。`limit` 最大500。

### 11. 多论文推荐（POST）

```
POST /recommendations/v1/papers/
Content-Type: application/json

{
  "positivePaperIds": ["paperId1", "paperId2"],
  "negativePaperIds": ["paperId3"]
}
```

### 12. 批量论文查询（POST）

```
POST /graph/v1/paper/batch?fields={fields}
Content-Type: application/json

{"ids": ["DOI:10.1038/nature12373", "ARXIV:2005.14165"]}
```

每次请求最多500个ID。

## 分页规则

| 端点 | 单页最大 | 总量上限 | 方法 |
|----------|-------------|-----------|--------|
| 相关性搜索 | 100 | 1,000 | offset/next |
| 批量搜索 | 1,000 | 10,000,000 | token |
| 引用/参考文献 | 1,000 | 无上限 | offset/next |
| 作者搜索 | 1,000 | -- | offset/next |

## 出版物类型

`Review`, `JournalArticle`, `CaseReport`, `ClinicalTrial`, `Conference`, `Dataset`, `Editorial`, `LettersAndComments`, `MetaAnalysis`, `News`, `Study`, `Book`, `BookSection`

## 研究领域

`Computer Science`, `Medicine`, `Chemistry`, `Biology`, `Materials Science`, `Physics`, `Geology`, `Psychology`, `Art`, `History`, `Geography`, `Sociology`, `Business`, `Political Science`, `Economics`, `Philosophy`, `Mathematics`, `Engineering`, `Environmental Science`, `Agricultural and Food Sciences`, `Education`, `Law`, `Linguistics`

## 错误格式

```json
{"message": "请求过多", "code": "429"}
```

HTTP 404表示未找到，429表示超出速率限制。
