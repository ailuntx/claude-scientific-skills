# bioRxiv API

bioRxiv 是一个生物学领域的预印本服务器。该 API 提供预印本的元数据，包括标题、作者、摘要、DOI 和发表状态。

**重要提示：** bioRxiv API **不支持关键词检索**。仅支持按日期范围浏览和 DOI 查询。如需检索 bioRxiv 预印本关键词，请改用 Semantic Scholar、OpenAlex 或 CORE。

## 基础 URL

```
https://api.biorxiv.org
```

## 认证

无需认证。完全公开的 API。

## 核心端点

### 1. 内容详情——按日期范围浏览

```
GET /details/biorxiv/{interval}/{cursor}/{format}
```

| 参数       | 取值                  | 描述                          |
|------------|-----------------------|-------------------------------|
| `interval` | `YYYY-MM-DD/YYYY-MM-DD` | 日期范围（包含起止日）。建议保持范围窄（1-3天）避免超时 |
|            | `N` (整数)           | 最新 N 篇预印本               |
|            | `Nd` (整数 + "d")    | 最近 N 天                     |
| `cursor`   | 整数（默认 `0`）      | 分页偏移量（每页 100 条结果） |
| `format`   | `json` (默认), `xml` | 响应格式                      |

可选查询参数：`?category=neuroscience`（按类别筛选，空格用下划线替代）

**示例：**
```
https://api.biorxiv.org/details/biorxiv/2024-01-01/2024-01-31/0
https://api.biorxiv.org/details/biorxiv/5
https://api.biorxiv.org/details/biorxiv/10d
https://api.biorxiv.org/details/biorxiv/2024-01-01/2024-01-31?category=neuroscience
```

### 2. 内容详情——DOI 查询

```
GET /details/biorxiv/{doi}/na/{format}
```

**示例：**
```
https://api.biorxiv.org/details/biorxiv/10.1101/2024.01.16.575895/na/json
```

### 3. 已发表文章链接

```
GET /pubs/biorxiv/{interval}/{cursor}
GET /pubs/biorxiv/{doi}/na
```

将预印本关联至其正式发表的期刊版本。同时接受预印本 DOI 和已发表 DOI。

### 4. 出版商筛选

```
GET /publisher/{prefix}/{interval}/{cursor}
```

按 DOI 前缀查找特定出版商发布的 bioRxiv 论文。

**示例：**
```
https://api.biorxiv.org/publisher/10.15252/2024-01-01/2024-06-01/0
```

## 响应格式

```json
{
  "messages": [{
    "status": "ok",
    "count": 100,
    "total": "1029",
    "cursor": 0
  }],
  "collection": [{
    "title": "论文标题...",
    "authors": "姓氏, A.; 姓氏, B.",
    "author_corresponding": "通讯作者全名",
    "author_corresponding_institution": "机构名称",
    "doi": "10.1101/2024.01.16.575895",
    "date": "2024-01-20",
    "version": "1",
    "type": "new results",
    "license": "cc_no",
    "category": "cancer biology",
    "jatsxml": "https://www.biorxiv.org/content/early/.../source.xml",
    "abstract": "完整摘要文本...",
    "published": "10.1158/2159-8290.CD-24-0187",
    "server": "bioRxiv"
  }]
}
```

- 若尚未在期刊发表，`published` 值为 `"NA"`；若已发表则为对应 DOI
- `type` 取值：`new results`（新发现）, `confirmatory results`（验证性结果）, `contradictory results`（矛盾性结果）

## 分页机制

所有多结果端点均返回**每页 100 条结果**。使用 `cursor` 参数进行分页，`messages` 对象中的 `total` 字段显示总条目数。

## 速率限制

无公开速率限制。无需认证。请合理控制请求频率。

## 学科分类

`animal-behavior-and-cognition`, `biochemistry`, `bioengineering`, `bioinformatics`, `biophysics`, `cancer-biology`, `cell-biology`, `clinical-trials`, `developmental-biology`, `ecology`, `epidemiology`, `evolutionary-biology`, `genetics`, `genomics`, `immunology`, `microbiology`, `molecular-biology`, `neuroscience`, `paleontology`, `pathology`, `pharmacology-and-toxicology`, `physiology`, `plant-biology`, `scientific-communication-and-education`, `synthetic-biology`, `systems-biology`, `zoology`
