# PubMed (NCBI E-utilities)

PubMed 提供超过 3700 万篇生物医学与生命科学文献的引文、摘要和元数据。它不包含全文——如需全文请使用 PMC。

## 基础 URL

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/
```

## 认证

- **API 密钥可选**但推荐使用。无密钥：每秒 3 次请求。有密钥：每秒 10 次请求。
- 传递方式：`&api_key=YOUR_KEY`
- 所有请求需包含 `&tool=your_app_name&email=your@email.com`。

## 核心端点

### 1. eSearch —— 检索并获取 PMID

```
GET /esearch.fcgi?db=pubmed&term={query}&retmode=json
```

| 参数 | 必填 | 默认值 | 描述 |
|-----------|----------|---------|-------------|
| `db` | 是 | -- | `pubmed` |
| `term` | 是 | -- | 检索语句。支持 PubMed 语法：字段标签 `[AU]`、`[TI]`、`[TA]`、`[MH]`（MeSH），布尔运算符 AND/OR/NOT |
| `retmax` | 否 | 20 | 返回 PMID 最大数量（上限 10,000） |
| `retstart` | 否 | 0 | 分页偏移量 |
| `retmode` | 否 | `xml` | `json` 或 `xml` |
| `rettype` | 否 | `uilist` | `uilist`（ID列表）或 `count`（仅计数） |
| `sort` | 否 | `relevance` | `relevance`（相关度）、`pub_date`（发表日期）、`Author`（作者）、`JournalName`（期刊名） |
| `datetype` | 否 | -- | `pdat`（出版日期）、`mdat`（修改日期）、`edat`（入库日期） |
| `mindate` / `maxdate` | 否 | -- | 日期范围 `YYYY/MM/DD` |
| `reldate` | 否 | -- | 最近 N 天的条目 |
| `usehistory` | 否 | -- | `y` 表示将大型结果集存入历史服务器 |

**示例：**
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=CRISPR+gene+therapy&retmode=json&retmax=5&sort=pub_date
```

**响应：**
```json
{
  "esearchresult": {
    "count": "224107",
    "retmax": "5",
    "retstart": "0",
    "idlist": ["39984857", "39984678", "39984543", "39984210", "39983901"]
  }
}
```

### 2. eSummary —— 获取文献摘要

```
GET /esummary.fcgi?db=pubmed&id={pmids}&retmode=json
```

| 参数 | 必填 | 描述 |
|-----------|----------|-------------|
| `db` | 是 | `pubmed` |
| `id` | 是 | 逗号分隔的 PMID（上限 10,000） |
| `retmode` | 否 | `json` 或 `xml` |

**示例：**
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&id=39984857,39984678&retmode=json
```

**响应字段：** `uid`、`pubdate`、`source`（期刊）、`authors`、`title`、`volume`、`issue`、`pages`、`fulljournalname`、`elocationid`（DOI）、`articleids`（PMC/DOI等）、`pubtype`、`pmcrefcount`

### 3. eFetch —— 获取完整记录（摘要、MEDLINE格式）

```
GET /efetch.fcgi?db=pubmed&id={pmids}&rettype={type}&retmode={mode}
```

| rettype | retmode | 返回内容 |
|---------|---------|---------|
| *(省略)* | `xml` | 完整 PubMed XML（引文+摘要） |
| `medline` | `text` | MEDLINE 格式 |
| `abstract` | `text` | 纯文本摘要 |
| `uilist` | `text` | PMID 列表 |

**示例——获取 XML 格式摘要：**
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=39984857&retmode=xml
```

XML 包含 `<PubmedArticle>` 节点，内含 `<MedlineCitation>`（标题/摘要/MeSH词/作者）和 `<PubmedData>`（文献ID/出版历史）。

### 4. eLink —— 查找相关文献

```
GET /elink.fcgi?dbfrom=pubmed&db=pubmed&id={pmid}&cmd=neighbor_score&retmode=json
```

返回带相关度评分的关联 PMID。

## 检索语法技巧

- **字段标签：** `aspirin[TI]`（标题）、`Smith J[AU]`（作者）、`Nature[TA]`（期刊）、`neoplasms[MH]`（MeSH主题词）
- **布尔运算：** `CRISPR AND (therapy OR treatment)`
- **日期范围：** `2020/01/01:2024/12/31[PDAT]`
- **文献类型：** `review[PT]`（综述）、`clinical trial[PT]`（临床试验）
- **生物体：** `humans[MH]`（人类）、`mice[MH]`（小鼠）

## 速率限制

- **无 API 密钥**：每秒 3 次请求
- **有 API 密钥**：每秒 10 次请求
- 每次请求需包含 `tool` 和 `email` 参数
- 大型批处理任务应在非高峰时段运行（美国东部时间周一至周五 5AM-9PM）

## 错误格式

```json
{"error": "API 速率限制已超出", "count": "11"}
```

HTTP 400 表示错误请求，429 表示超出速率限制。
