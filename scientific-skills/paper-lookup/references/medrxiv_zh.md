# medRxiv 接口

medRxiv 是健康科学领域的预印本服务器。该接口与 bioRxiv 的接口完全相同——相同的端点、相同的响应格式——只需将 `medrxiv` 作为服务器参数即可。

**重要提示：** 与 bioRxiv 相同，**不支持关键词搜索**。如需对 medRxiv 内容进行关键词搜索，请使用 Semantic Scholar、OpenAlex 或 PubMed。

## 基础 URL

```
https://api.biorxiv.org
```

（与 bioRxiv 的基础 URL 相同——服务器在路径中指定。）

## 认证

无需认证。完全公开的接口。

## 核心端点

### 1. 内容详情——按日期范围浏览

```
GET /details/medrxiv/{interval}/{cursor}/{format}
```

| 参数 | 取值 | 描述 |
|-----------|--------|-------------|
| `interval` | `YYYY-MM-DD/YYYY-MM-DD` | 日期范围（包含起止日期） |
| | `N` (整数) | 最新的 N 篇预印本 |
| | `Nd` (整数 + "d") | 最近 N 天 |
| `cursor` | 整数（默认 `0`） | 分页偏移量（每页 100 条） |
| `format` | `json`（默认）, `xml` | 响应格式 |

可选参数：`?category=cardiovascular%20medicine`（空格需进行 URL 编码）

**示例：**
```
https://api.biorxiv.org/details/medrxiv/2024-01-01/2024-01-31/0
https://api.biorxiv.org/details/medrxiv/5
https://api.biorxiv.org/details/medrxiv/10d
```

### 2. 内容详情——DOI 查询

```
GET /details/medrxiv/{doi}/na/{format}
```

**示例：**
```
https://api.biorxiv.org/details/medrxiv/10.1101/2021.04.29.21256344/na/json
```

### 3. 已发表文章链接

```
GET /pubs/medrxiv/{interval}/{cursor}
GET /pubs/medrxiv/{doi}/na
```

将预印本链接至其正式发表的期刊版本。同时接受预印本 DOI 和已发表 DOI。

## 响应格式

与 bioRxiv 相同：

```json
{
  "messages": [{
    "status": "ok",
    "count": 100,
    "total": "502",
    "cursor": 0
  }],
  "collection": [{
    "title": "论文标题...",
    "authors": "姓氏, A.; 姓氏, B.",
    "author_corresponding": "通讯作者全名",
    "author_corresponding_institution": "机构",
    "doi": "10.1101/2021.04.29.21256344",
    "date": "2021-05-03",
    "version": "1",
    "type": "PUBLISHAHEADOFPRINT",
    "license": "cc_by_nc_nd",
    "category": "cardiovascular medicine",
    "abstract": "完整摘要文本...",
    "published": "10.1371/journal.pone.0256482",
    "server": "medRxiv"
  }]
}
```

## 分页

每页 100 条结果。使用 `cursor` 参数进行分页。

## 速率限制

无文档化速率限制。无需认证。

## 学科分类

`addiction-medicine`, `allergy-and-immunology`, `anesthesia`, `cardiovascular-medicine`, `dentistry-and-oral-medicine`, `dermatology`, `emergency-medicine`, `endocrinology`, `epidemiology`, `forensic-medicine`, `gastroenterology`, `genetic-and-genomic-medicine`, `geriatric-medicine`, `health-economics`, `health-informatics`, `health-policy`, `health-systems-and-quality-improvement`, `hematology`, `hiv-aids`, `infectious-diseases`, `intensive-care-and-critical-care-medicine`, `medical-education`, `medical-ethics`, `nephrology`, `neurology`, `nursing`, `nutrition`, `obstetrics-and-gynecology`, `occupational-and-environmental-health`, `oncology`, `ophthalmology`, `orthopedics`, `otolaryngology`, `pain-medicine`, `palliative-medicine`, `pathology`, `pediatrics`, `pharmacology-and-therapeutics`, `primary-care-research`, `psychiatry-and-clinical-psychology`, `public-and-global-health`, `radiology-and-imaging`, `rehabilitation-medicine-and-physical-therapy`, `respiratory-medicine`, `rheumatology`, `sexual-and-reproductive-health`, `sports-medicine`, `surgery`, `toxicology`, `transplantation`, `urology`
