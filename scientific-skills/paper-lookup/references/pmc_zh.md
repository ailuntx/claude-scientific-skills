# PMC（PubMed Central）

PMC 是一个生物医学和生命科学文献的**全文存档库**。它与 PubMed 相互独立——PubMed 提供文献引用/摘要，而 PMC 提供全文。并非所有 PubMed 文章都收录于 PMC，反之亦然。

## PMC 的 E-utilities 工具

### 基础 URL

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/
```

与 PubMed 相同的 E-utilities 工具，但需指定 `db=pmc`。

### eSearch —— 检索 PMC

```
GET /esearch.fcgi?db=pmc&term={query}&retmode=json
```

参数与 PubMed eSearch 相同。返回 PMC UID（数字格式，如 `13033346`）。需添加 "PMC" 前缀才能获得 PMCID（例如 `PMC13033346`）。

### eFetch —— 获取全文 XML

```
GET /efetch.fcgi?db=pmc&id={pmcid}&retmode=xml
```

| rettype | retmode | 返回内容 |
|---------|---------|---------|
| *(省略)* | `xml` | **全文 JATS XML**（正文、图表、参考文献） |
| `medline` | `text` | MEDLINE 格式 |

**示例：**
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pmc&id=7029759&retmode=xml
```

XML 采用 JATS（期刊文章标签集）格式：
- `<front>` —— 期刊元数据、文章元数据、作者信息
- `<body>` —— 完整文章文本，含 `<sec>` 章节、`<p>` 段落、`<fig>` 图表
- `<back>` —— 包含所有参考文献的 `<ref-list>`

仅传递数字 ID（非 "PMC7029759"，只需 "7029759"）。

## BioC API —— 结构化全文

另一种获取结构化段落格式全文的替代方案。

### 基础 URL

```
https://www.ncbi.nlm.nih.gov/research/bionlp/RESTful/pmcoa.cgi/
```

### 端点

```
GET /BioC_{format}/{id}/{encoding}
```

| 参数 | 取值 |
|-----------|--------|
| `format` | `json` 或 `xml` |
| `id` | PMID（如 `17299597`）或 PMCID（如 `PMC7029759`） |
| `encoding` | `unicode` 或 `ascii` |

**示例：**
```
https://www.ncbi.nlm.nih.gov/research/bionlp/RESTful/pmcoa.cgi/BioC_json/PMC7029759/unicode
```

**响应结构（JSON）：**
```json
{
  "source": "PMC",
  "documents": [{
    "id": "PMC7029759",
    "infons": {"license": "...", "doi": "..."},
    "passages": [
      {
        "offset": 0,
        "infons": {"section_type": "TITLE"},
        "text": "文章标题..."
      },
      {
        "offset": 42,
        "infons": {"section_type": "ABSTRACT"},
        "text": "摘要文本..."
      },
      {
        "offset": 500,
        "infons": {"section_type": "INTRO"},
        "text": "引言文本..."
      }
    ]
  }]
}
```

章节类型：`TITLE`（标题）、`ABSTRACT`（摘要）、`INTRO`（引言）、`METHODS`（方法）、`RESULTS`（结果）、`DISCUSS`（讨论）、`CONCL`（结论）、`REF`（参考文献）、`SUPPL`（补充材料）、`FIG`（图表）、`TABLE`（表格）

**覆盖范围：** 约 300 万篇来自 PMC 开放获取子集的文献。

## PMC ID 转换器 API

实现 PMID、PMCID、DOI 和稿件 ID 之间的相互转换。

### 基础 URL

```
https://pmc.ncbi.nlm.nih.gov/tools/idconv/api/v1/articles/
```

### 参数

| 参数 | 必填 | 说明 |
|-----------|----------|-------------|
| `ids` | 是 | 最多 200 个逗号分隔的 ID |
| `idtype` | 否 | `pmcid`、`pmid`、`mid`、`doi`（默认自动检测） |
| `format` | 否 | `json`、`xml`、`csv`（默认 xml） |
| `tool` | 推荐 | 您的应用名称 |
| `email` | 推荐 | 您的联系邮箱 |

**示例：**
```
https://pmc.ncbi.nlm.nih.gov/tools/idconv/api/v1/articles/?ids=PMC7029759&format=json
```

**响应：**
```json
{
  "status": "ok",
  "records": [{
    "pmcid": "PMC7029759",
    "pmid": "32117569",
    "doi": "10.12688/f1000research.22211.2"
  }]
}
```

仅返回 PMC 收录文章的结果。若文章在 PubMed 但未收录于 PMC，则不会返回 PMCID。

## 速率限制

| 服务 | 限制 |
|---------|-------|
| E-utilities (`db=pmc`) | 无密钥时 3次/秒，有密钥时 10次/秒 |
| BioC API | 遵循 NCBI 通用策略（无密钥时 3次/秒） |
| ID 转换器 | 遵循 NCBI 通用策略 |

在 E-utility 请求中包含 `tool` 和 `email` 参数。批量任务应在非高峰时段运行（美国东部时间周一至周五 5AM-9PM 以外）。
