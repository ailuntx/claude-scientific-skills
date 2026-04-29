# NCBI基因（E-utilities）

## 基础URL
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/
```

## 认证
API密钥可选但推荐使用。无密钥：每秒3次请求。有密钥：每秒10次请求。
免费密钥获取地址：https://www.ncbi.nlm.nih.gov/account/settings/
传递方式：`&api_key=YOUR_KEY`

## 关键端点

### eSearch — 搜索基因ID
```
GET /esearch.fcgi?db=gene&term={query}&retmode=json&retmax={n}
```

参数：
- `db=gene`（必需）
- `term` — 搜索查询（例如 `BRCA1[gene]+AND+human[orgn]`）
- `retmode=json`
- `retmax` — 最大结果数（默认20）
- `retstart` — 分页起始位置

示例：
```
/esearch.fcgi?db=gene&term=BRCA1[gene]+AND+human[orgn]&retmode=json&retmax=5
```

### eSummary — 获取基因元数据
```
GET /esummary.fcgi?db=gene&id={gene_ids}&retmode=json
```

关键响应字段：`name`, `description`, `chromosome`, `maplocation`, `otheraliases`, `nomenclaturesymbol`, `organism`

示例：
```
/esummary.fcgi?db=gene&id=672&retmode=json
```

### eFetch — 完整基因记录（仅XML/文本，无JSON）
```
GET /efetch.fcgi?db=gene&id={gene_ids}&rettype=gene_table&retmode=text
```

### eLink — 跨数据库链接（基因到通路、PubMed、OMIM）
```
GET /elink.fcgi?dbfrom=gene&db={target_db}&id={gene_id}&retmode=json
```

目标数据库：`biosystems`（通路）、`pubmed`、`omim`、`nuccore`、`protein`

示例 — 基因到通路：
```
/elink.fcgi?dbfrom=gene&db=biosystems&id=672&retmode=json
```

## 速率限制
- 无API密钥：每秒3次请求
- 有API密钥：每秒10次请求
- 批量操作：在eSearch中使用`usehistory=y`，然后通过`query_key`和`WebEnv`检索
