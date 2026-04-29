# OpenAlex API

OpenAlex 是一个包含 2.5 亿多篇学术成果、作者、机构、来源和主题的综合索引库，是该领域覆盖范围最广的多学科数据库。

## 基础 URL

```
https://api.openalex.org
```

## 认证

- **推荐使用 API 密钥**（免费）。访问 https://openalex.org/settings/api 获取
- 传递方式：`?api_key=YOUR_KEY`
- 传统礼貌池仍有效：添加 `?mailto=you@example.com` 可获得更宽松的速率限制

## 速率限制

- 最高 **100 次请求/秒**
- 基于使用量的计费模式，含每日 1 美元免费额度
- 通过 ID/DOI 的单实体查询免费（无限制）
- 列表+筛选查询：约 0.0001 美元/次（每日约 10,000 次免费）
- 搜索查询：约 0.001 美元/次（每日约 1,000 次免费）

## 核心端点

### 1. 获取单篇成果

```
GET /works/{id}
```

支持多种 ID 格式：
```
/works/W2741809807                              (OpenAlex ID)
/works/doi:10.7717/peerj.4375                  (DOI)
/works/pmid:29456894                            (PMID)
/works/https://doi.org/10.7717/peerj.4375      (完整 DOI URL)
```

### 2. 搜索成果

```
GET /works?search={query}&per_page={n}&page={n}
```

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `search` | -- | 全文搜索（标题、摘要、全文）。支持布尔运算符：`AND`、`OR`、`NOT`（大写） |
| `search.exact` | -- | 禁用词干提取 |
| `search.semantic` | -- | AI 嵌入搜索（测试版，1 次请求/秒，最多 50 条结果） |
| `filter` | -- | 逗号分隔的 `字段:值` 组合 |
| `sort` | relevance | `cited_by_count:desc`、`publication_date:desc`、`relevance_score:desc` |
| `per_page` | 25 | 每页结果数（最多 100） |
| `page` | 1 | 页码（最大 `页码 * 每页数量` = 10,000） |
| `cursor` | -- | 深度分页时首页使用 `*` |
| `select` | -- | 逗号分隔的返回字段 |
| `group_by` | -- | 按字段聚合 |

**高级搜索：** 支持通配符（`machin*`）、模糊匹配（`machin~1`）、邻近搜索（`"climate change"~5`）、布尔分组。

**示例：**
```
https://api.openalex.org/works?search=CRISPR+gene+therapy&filter=from_publication_date:2023-01-01&sort=cited_by_count:desc&per_page=10
```

### 3. 筛选成果

```
GET /works?filter={filters}
```

关键筛选字段：
| 筛选器 | 示例 | 描述 |
|--------|---------|-------------|
| `from_publication_date` | `2023-01-01` | 发布日期之后 |
| `to_publication_date` | `2024-12-31` | 发布日期之前 |
| `publication_year` | `2024` | 精确年份 |
| `type` | `article` | 成果类型 |
| `cited_by_count` | `>100` | 引用次数阈值 |
| `is_oa` | `true` | 仅开放获取 |
| `has_abstract` | `true` | 包含摘要 |
| `authorships.author.id` | `A5048491430` | 按作者 ID |
| `primary_location.source.id` | `S137773608` | 按期刊/来源 |
| `institutions.country_code` | `us` | 按国家 |
| `concepts.id` | `C41008148` | 按概念/主题 |
| `doi` | `10.1038/nature12373` | 按 DOI |

**运算符：** `>`、`<`、`!`（取反）、`|`（筛选器内 OR 逻辑）

**示例：**
```
https://api.openalex.org/works?filter=from_publication_date:2024-01-01,type:article,is_oa:true,cited_by_count:>50
```

### 4. 其他实体

```
GET /authors?search={name}
GET /authors/{id}
GET /sources?search={name}          (期刊、知识库)
GET /sources/{id}
GET /institutions?search={name}
GET /institutions/{id}
GET /topics/{id}
```

作者和机构端点支持类似的筛选/排序/分页参数。

### 5. 游标分页（超过 10,000 条结果）

```
GET /works?filter=publication_year:2024&cursor=*&per_page=100
```

响应包含 `meta.next_cursor`。在后续请求中作为 `cursor={value}` 传递。当 `next_cursor` 为 null 时停止。

## 响应格式

### 成果对象（关键字段）

```json
{
  "id": "https://openalex.org/W2741809807",
  "doi": "https://doi.org/10.7717/peerj.4375",
  "title": "The state of OA",
  "publication_year": 2018,
  "publication_date": "2018-02-13",
  "type": "article",
  "language": "en",
  "is_retracted": false,
  "cited_by_count": 1169,
  "open_access": {
    "is_oa": true,
    "oa_status": "gold",
    "oa_url": "https://doi.org/10.7717/peerj.4375"
  },
  "authorships": [{
    "author": {"id": "https://openalex.org/A5048491430", "display_name": "Heather Piwowar"},
    "institutions": [{"display_name": "Impactstory"}]
  }],
  "primary_location": {
    "source": {"display_name": "PeerJ", "issn_l": "2167-8359"}
  },
  "abstract_inverted_index": {"Despite": [0], "growing": [1], "interest": [2], ...},
  "referenced_works": ["https://openalex.org/W123...", ...],
  "ids": {"openalex": "...", "doi": "...", "pmid": "..."}
}
```

### 摘要倒排索引

摘要以 `{单词: [位置]}` 格式存储。重构方法：
```python
def reconstruct(inverted_index):
    positions = {}
    for word, indices in inverted_index.items():
        for idx in indices:
            positions[idx] = word
    return ' '.join(positions[i] for i in sorted(positions.keys()))
```

### 列表响应

```json
{
  "meta": {"count": 3771834, "page": 1, "per_page": 10},
  "results": [...]
}
```

## 错误格式

HTTP 403 表示无效 API 密钥，429 表示超出速率限制。错误响应包含 message 字段。
