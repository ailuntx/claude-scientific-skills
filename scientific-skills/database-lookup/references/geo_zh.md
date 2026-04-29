# NCBI GEO（基因表达综合数据库）通过E-utilities访问

## 基础URL

| 用途 | URL |
|---|---|
| E-utilities | `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/` |
| GEO直接查询 | `https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi` |

## 重要提示：数据库名称为`gds`

GEO的Entrez数据库是`gds`（而非`geo`）。它包含所有GEO记录类型：GDS数据集、GSE系列、GPL平台、GSM样本。在搜索词中使用`[ETYP]`按类型筛选。

## 关键端点

### eSearch — 搜索GEO

```
GET /esearch.fcgi?db=gds&term={query}&retmode=json&retmax={n}
```

参数：
- `db=gds`（必需）
- `term` — 带字段标签的搜索查询
- `retmax` — 最大结果数（默认20）
- `retstart` — 分页偏移量
- `retmode=json` — 获取JSON响应
- `usehistory=y` — 大型查询时在服务器端存储结果
- `api_key` — NCBI API密钥（可选，可提高速率限制）

#### 条目类型筛选器（`[ETYP]`）
- `gds[ETYP]` — 精选GEO数据集
- `gse[ETYP]` — GEO系列（最常用，默认选此项）
- `gpl[ETYP]` — 平台
- `gsm[ETYP]` — 样本

#### 其他字段标签
- `[Organism]` — 例如`"Homo sapiens"[Organism]`
- `[PDAT]` — 发布日期
- `[Title]` — 标题搜索
- 布尔运算符：`AND`、`OR`、`NOT`（大写）

示例 — 人类癌症GSE系列：
```
/esearch.fcgi?db=gds&term=cancer+AND+gse[ETYP]+AND+"Homo+sapiens"[Organism]&retmax=10&retmode=json
```

响应：
```json
{
  "esearchresult": {
    "count": "15432",
    "retmax": "10",
    "idlist": ["200012345", "200067890"],
    "querytranslation": "cancer AND gse[ETYP]"
  }
}
```

返回的ID是数字UID（非登录号）。对于GSE记录：UID = 200000000 + GSE编号。

### eSummary — 获取UID元数据

```
GET /esummary.fcgi?db=gds&id={uid_list}&retmode=json
```

每条记录的关键响应字段：
- `Accession` — 例如"GSE12345"
- `title`、`summary`
- `taxon` — 生物体
- `entrytype` — "GDS"、"GSE"、"GPL"、"GSM"
- `gdstype` — 例如"Expression profiling by array"
- `n_samples` — 样本计数
- `pubmedids` — 关联的PubMed ID
- `PDAT` — 发布日期
- `Samples` — 样本对象数组
- `FTPLink` — 数据下载路径

示例：
```
/esummary.fcgi?db=gds&id=200012345&retmode=json
```

### GEO直接查询 — 按登录号获取完整记录

```
GET https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc={accession}&form={format}&view={detail}
```

参数：
- `acc` — GEO登录号（GSE12345、GDS1234、GPL570、GSM12345）
- `targ` — `self`、`gsm`（样本）、`gpl`（平台）、`gse`（系列）
- `form` — `text`（SOFT格式）、`xml`（MINiML）、`html`
- `view` — `quick`、`brief`、`full`、`data`

示例 — SOFT格式的系列元数据：
```
acc.cgi?acc=GSE53757&targ=self&form=text&view=brief
```

注意：acc.cgi不返回JSON。需要JSON结果时请使用eSearch+eSummary。需要完整SOFT/MINiML记录时使用acc.cgi。

### eLink — 与其他NCBI数据库交叉引用

```
GET /elink.fcgi?dbfrom=gds&db=pubmed&id={uid}&retmode=json
```

## 实用工作流程

多数查询采用两步法：

1. **eSearch** 查找匹配查询的UID
2. **eSummary** 获取这些UID的元数据

全程使用JSON格式。

## 重要注意事项

- GDS记录基本冻结 — NCBI已停止维护新GDS。全面检索请使用`gse[ETYP]`
- eFetch对`gds`数据库支持有限。获取元数据用eSummary，完整记录用acc.cgi
- URL编码规则：空格转为`+`，引号转为`%22`

## 速率限制

- **无API密钥**：每秒3次请求
- **有API密钥**：每秒10次请求（免费注册：ncbi.nlm.nih.gov/account/settings）
- 建议添加`&email=user@example.com`作为礼貌声明
- 大型结果集请使用历史服务器（`usehistory=y`后传递`WebEnv`和`query_key`至eSummary）
