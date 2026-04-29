# Unpaywall API

Unpaywall 可查询学术论文是否存在合法免费副本。通过输入 DOI，该服务将返回开放获取状态、PDF 链接及位置详情。

## 基础 URL

```
https://api.unpaywall.org/v2
```

## 认证

无需 API 密钥。必须在查询参数中包含**电子邮箱地址**：`?email=you@example.com`

**重要提示：** 请使用真实邮箱。Unpaywall 会拒绝 `test@example.com` 等占位邮箱并返回 HTTP 422 错误。

## 速率限制

每日 100,000 次调用。如需更高用量，请下载数据库快照。

## 核心端点

### 1. DOI 查询

```
GET /v2/{doi}?email=you@example.com
```

**示例：**
```
https://api.unpaywall.org/v2/10.1038/nature12373?email=you@example.com
```

### 2. 搜索功能（不可靠）

```
GET /v2/search?query={text}&email=you@example.com
```

**警告：** 截至 2026 年 3 月，搜索端点持续返回 HTTP 500 错误。该功能可能已弃用或间歇性失效。建议改用 DOI 查询——先通过 PubMed/OpenAlex/Semantic Scholar 查找论文，再逐篇检查开放获取状态。

| 参数      | 描述                          |
|-----------|-----------------------------|
| `query`   | 搜索文本。支持引号短语、`OR`、`-` 排除 |
| `is_oa`   | `true` 或 `false` —— 按开放获取状态过滤 |
| `page`    | 页码（从1开始），每页50条结果        |

## 响应格式

### DOI 查询响应
```json
{
  "doi": "10.1038/nature12373",
  "doi_url": "https://doi.org/10.1038/nature12373",
  "title": "Nanometre-scale thermometry in a living cell",
  "year": 2013,
  "published_date": "2013-07-31",
  "genre": "journal-article",
  "publisher": "Springer Nature",
  "is_oa": true,
  "oa_status": "green",
  "best_oa_location": {
    "url": "https://dash.harvard.edu/bitstream/1/...",
    "url_for_pdf": "https://dash.harvard.edu/bitstream/1/...pdf",
    "url_for_landing_page": "https://dash.harvard.edu/handle/...",
    "host_type": "repository",
    "version": "acceptedVersion",
    "license": "cc-by",
    "is_best": true,
    "oa_date": "2016-01-01"
  },
  "first_oa_location": {...},
  "oa_locations": [...],
  "has_repository_copy": true,
  "journal_name": "Nature",
  "journal_issns": "0028-0836,1476-4687",
  "journal_issn_l": "0028-0836",
  "journal_is_oa": false,
  "journal_is_in_doaj": false,
  "z_authors": [
    {"raw_author_name": "G. Kucsko", "author_position": "first"},
    {"raw_author_name": "P. C. Maurer", "author_position": "middle"}
  ]
}
```

### 开放获取状态值
| 状态      | 含义                                     |
|-----------|----------------------------------------|
| `gold`    | 发表于完全开放获取期刊                         |
| `hybrid`  | 订阅期刊中的开放获取内容（由出版商托管）               |
| `bronze`  | 出版商网站免费阅读但无开放获取许可                   |
| `green`   | 通过知识库获取（如机构库、预印本库）                 |
| `closed`  | 未找到合法免费副本                           |

### 开放获取位置字段
| 字段                   | 描述                                     |
|------------------------|----------------------------------------|
| `url`                  | 最佳链接（优先PDF，其次落地页）                  |
| `url_for_pdf`          | 直接PDF链接（无PDF时为null）                 |
| `url_for_landing_page` | 落地页链接                                 |
| `host_type`            | `publisher`（出版商）或 `repository`（知识库） |
| `version`              | `submittedVersion`, `acceptedVersion`, `publishedVersion` |
| `license`              | 如 `cc-by`, `cc-by-nc`, `implied-oa` 或 null |
| `is_best`              | 是否为 `best_oa_location`                 |
| `oa_date`              | 首次在此位置可获取的日期                       |

### 搜索响应
```json
{
  "results": [
    {
      "response": {...},
      "score": 42.5,
      "snippet": "...text with <b>highlighted</b> matches..."
    }
  ]
}
```

## 典型工作流

1. 从 PubMed、Crossref 等来源获取 DOI
2. 通过 DOI 调用 Unpaywall API
3. 检查 `is_oa` —— 若为 true，则使用 `best_oa_location.url_for_pdf` 获取免费 PDF
4. 检查 `oa_status` 了解开放获取类型
5. 若状态为 closed，`oa_locations` 将为空 —— 该文章需要订阅访问
