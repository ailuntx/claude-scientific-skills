# USPTO 公共 API

## 1. PatentsView API（主要专利搜索）

推荐使用基于 Elasticsearch 的新版 API 端点。

### 基础 URL

```
https://search.patentsview.org/api/v1/
```

**需 API 密钥** — 在 `https://patentsview.org/apis/keyrequest` 注册

通过查询参数传递：`?api_key=YOUR_KEY`

### 关键端点

#### 搜索专利
```
GET or POST /patent/
```

查询参数 `q` 接受 JSON 查询对象。

运算符：`_eq`, `_neq`, `_gt`, `_gte`, `_lt`, `_lte`, `_begins`, `_contains`, `_text_any`, `_text_all`, `_text_phrase`, `_and`, `_or`, `_not`

参数：
- `q` — JSON 查询
- `f` — 返回字段（JSON 数组）
- `o` — 选项：`{"size": 25}` 用于分页
- `s` — 排序：`[{"patent_date": "desc"}]`

#### 关键词搜索
```
GET /patent/?q={"_text_any":{"patent_abstract":"autonomous vehicle"}}&f=["patent_id","patent_title","patent_date"]&o={"size":5}&api_key=KEY
```

#### 发明人搜索
```
GET /patent/?q={"inventors.inventor_name_last":"Tesla"}&f=["patent_id","patent_title","patent_date"]&api_key=KEY
```

#### 受让人搜索
```
GET /patent/?q={"assignees.assignee_organization":"Google LLC"}&f=["patent_id","patent_title","patent_date","assignees"]&api_key=KEY
```

#### 专利号查询
```
GET /patent/{patent_number}/?api_key=KEY
```

#### 其他实体端点
```
/inventor/
/assignee/
/cpc_group/
```

### 响应结构

```json
{
  "patents": [
    {
      "patent_id": "11234567",
      "patent_title": "...",
      "patent_date": "2022-03-15",
      "patent_abstract": "...",
      "assignees": [{"assignee_organization": "..."}],
      "inventors": [{"inventor_name_first": "...", "inventor_name_last": "..."}]
    }
  ],
  "count": 1,
  "total_hits": 8923
}
```

### 速率限制

每个 API 密钥每分钟约 45 次请求。

### 重要提示

用户必须持有 PatentsView API 密钥。若未持有，请告知其前往 `https://patentsview.org/apis/keyrequest` 注册。密钥需通过 `.env` 加载为 `PATENTSVIEW_API_KEY`。

**注意：** 旧版 API `api.patentsview.org` 已停用（返回 410 Gone）。仅上述新版 API 有效。

## 3. PEDS — 专利审查数据系统

**URL**: `https://ped.uspto.gov/api/queries`

**方法**: POST

用于专利审查数据（申请状态、提交日期、审查员信息）。

```json
{
  "searchText": "applicationNumberText:16123456",
  "fl": "*",
  "mm": "100%",
  "df": "patentTitle",
  "facet": "false",
  "sort": "applId asc",
  "start": 0
}
```

无需 API 密钥但严格限流。服务可用性不稳定。

## 4. TSDR — 商标状态与文件检索

通过序列号或注册号查询商标（非全文搜索）。

```
GET https://tsdr.uspto.gov/documentxml/status/{serial_number}
GET https://tsdr.uspto.gov/documentxml/status/rn{registration_number}
```

返回包含商标详情、状态、持有人、商品/服务及审查历史的 XML。

无 API 密钥。严格限流。无 JSON 端点 — 仅返回 XML。

## 5. 限制

- **无商标全文搜索公共 REST API**（TESS 仅限网页版）
- PatentsView 新版 API 需注册获取密钥
- PEDS 服务可用性不稳定
- TSDR 需预先知晓序列号/注册号
