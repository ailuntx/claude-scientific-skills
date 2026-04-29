# SEC EDGAR API 参考文档

## 概述
美国证券交易委员会（SEC）的电子化数据收集、分析与检索系统。免费提供公司申报文件、企业数据和XBRL财务数据访问。无需API密钥，但必须携带标识用户身份的User-Agent请求头。

## 基础URL
- **EFTS（全文搜索）：** `https://efts.sec.gov/LATEST`
- **公司/申报数据：** `https://data.sec.gov`
- **EDGAR网站/档案库：** `https://www.sec.gov`
- **XBRL API：** `https://data.sec.gov/api/xbrl`

## 认证
- **API密钥：** 无需。
- **User-Agent请求头：** 每次请求必需。必须包含公司/个人名称和邮箱。
  ```
  User-Agent: 我的公司 admin@mycompany.com
  ```
  未携带合规User-Agent的请求将被拦截（403错误）。

## 速率限制
- 每个源IP **每秒10次请求**。
- 超出限制将触发基于IP的临时限流（HTTP 429）。
- SEC建议用户尽可能在非交易时段（美东时间晚9点至早6点）执行批量下载操作。

---

## 核心端点

### 1. 全文搜索（EFTS）

#### `GET https://efts.sec.gov/LATEST/search-index`
在所有EDGAR申报文件中执行全文检索。

**参数：**
| 参数         | 类型   | 必需     | 说明 |
|-------------|--------|----------|-------------|
| `q`         | 字符串 | 是       | 搜索查询文本。支持布尔运算符（`AND`, `OR`, `NOT`）和引号包裹的精确短语 |
| `dateRange` | 字符串 | 否       | 设为`custom`启用日期筛选 |
| `startdt`   | 字符串 | 否       | 起始日期 `YYYY-MM-DD` |
| `enddt`     | 字符串 | 否       | 结束日期 `YYYY-MM-DD` |
| `forms`     | 字符串 | 否       | 逗号分隔的表格类型，如 `10-K,10-Q,8-K` |
| `from`      | 整型   | 否       | 分页偏移量（默认0） |
| `size`      | 整型   | 否       | 每页结果数（默认10，最大值浮动） |

**示例：**
```
https://efts.sec.gov/LATEST/search-index?q=%22artificial+intelligence%22&forms=10-K&startdt=2024-01-01&enddt=2024-12-31
```

**响应：**
```json
{
  "hits": {
    "hits": [
      {
        "_id": "0001234567-24-000123:filing.htm",
        "_source": {
          "file_date": "2024-03-15",
          "display_date_filed": "2024-03-15",
          "entity_name": "示例公司",
          "file_num": "001-12345",
          "form_type": "10-K",
          "file_description": "年度报告",
          "period_of_report": "2023-12-31"
        }
      }
    ],
    "total": { "value": 150 }
  }
}
```

### 2. EDGAR全文搜索（推荐新端点）

#### `GET https://efts.sec.gov/LATEST/search-index`（也可通过下方URL访问）

#### `GET https://efts.sec.gov/LATEST/search-index?q=...`

注：EDGAR全文搜索也通过更简洁的URL提供：

#### `GET https://efts.sec.gov/LATEST/search-index`

此为规范端点。部分文档提及的EDGAR搜索界面也调用相同后端。

---

### 3. 公司股票代码与CIK查询

#### `GET https://www.sec.gov/cgi-bin/browse-edgar`
传统EDGAR公司搜索接口。

**参数：**
| 参数      | 类型   | 必需     | 说明 |
|-----------|--------|----------|-------------|
| `company` | 字符串 | 否       | 公司名称搜索 |
| `CIK`     | 字符串 | 否       | CIK编号或股票代码 |
| `type`    | 字符串 | 否       | 申报类型筛选（如`10-K`） |
| `dateb`   | 字符串 | 否       | 申报截止日期 `YYYY-MM-DD` |
| `owner`   | 字符串 | 否       | `include`, `exclude` 或 `only` |
| `count`   | 整型   | 否       | 结果数量（上限100） |
| `action`  | 字符串 | 是       | 公司搜索需设为`getcompany` |
| `output`  | 字符串 | 否       | 设为`atom`获取XML/Atom格式 |

**示例：**
```
https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=AAPL&type=10-K&dateb=&owner=include&count=10&output=atom
```

#### `GET https://www.sec.gov/files/company_tickers.json`
返回所有公司股票代码与CIK编号的JSON映射表。

**响应：**
```json
{
  "0": {"cik_str": 320193, "ticker": "AAPL", "title": "苹果公司"},
  "1": {"cik_str": 789019, "ticker": "MSFT", "title": "微软公司"},
  ...
}
```

#### `GET https://www.sec.gov/files/company_tickers_exchange.json`
包含各股票代码的交易所信息。

---

### 4. 公司申报文件与提交记录

#### `GET https://data.sec.gov/submissions/CIK{cik_padded}.json`
返回指定CIK（补零至10位）的公司元数据与近期申报记录。

**示例：**
```
https://data.sec.gov/submissions/CIK0000320193.json
```

**响应：**
```json
{
  "cik": "320193",
  "entityType": "operating",
  "sic": "3571",
  "sicDescription": "电子计算机",
  "name": "苹果公司",
  "tickers": ["AAPL"],
  "exchanges": ["纳斯达克"],
  "filings": {
    "recent": {
      "accessionNumber": ["0000320193-24-000123", ...],
      "filingDate": ["2024-11-01", ...],
      "reportDate": ["2024-09-28", ...],
      "form": ["10-K", ...],
      "primaryDocument": ["aapl-20240928.htm", ...],
      "primaryDocDescription": ["10-K", ...]
    },
    "files": [
      {"name": "CIK0000320193-submissions-001.json", "filingCount": 1000}
    ]
  }
}
```

`filings.recent`对象包含最近约1000条申报记录。历史申报文件存储于`filings.files`引用的分页文件中。

---

### 5. 公司概念（XBRL数据）

#### `GET https://data.sec.gov/api/xbrl/companyconcept/CIK{cik}/{taxonomy}/{tag}.json`
返回某公司在所有申报文件中针对特定XBRL标签报告的全部数值。

**路径参数：**
| 参数       | 说明 |
|------------|-------------|
| `cik`      | 补零的CIK（10位数字） |
| `taxonomy` | XBRL分类标准：`us-gaap`, `ifrs-full`, `dei`, `srt` |
| `tag`      | XBRL概念标签，如`Revenue`, `Assets`, `AccountsPayableCurrent` |

**示例：**
```
https://data.sec.gov/api/xbrl/companyconcept/CIK0000320193/us-gaap/Revenue.json
```

**响应：**
```json
{
  "cik": 320193,
  "taxonomy": "us-gaap",
  "tag": "Revenue",
  "label": "营业收入",
  "description": "确认的收入金额...",
  "entityName": "苹果公司",
  "units": {
    "USD": [
      {
        "start": "2023-10-01",
        "end": "2024-09-28",
        "val": 391035000000,
        "accn": "0000320193-24-000123",
        "fy": 2024,
        "fp": "FY",
        "form": "10-K",
        "filed": "2024-11-01"
      }
    ]
  }
}
```

---

### 6. 公司事实（单公司全量XBRL）

#### `GET https://data.sec.gov/api/xbrl/companyfacts/CIK{cik}.json`
返回某公司在所有申报文件中报告的全部XBRL概念。

**示例：**
```
https://data.sec.gov/api/xbrl/companyfacts/CIK0000320193.json
```

**响应：** 结构与companyconcept相同，但包含嵌套在`facts.us-gaap`等分类下的所有标签。

```json
{
  "cik": 320193,
  "entityName": "苹果公司",
  "facts": {
    "dei": {
      "EntityCommonStockSharesOutstanding": { "units": { "shares": [...] } }
    },
    "us-gaap": {
      "Revenue": { "units": { "USD": [...] } },
      "Assets": { "units": { "USD": [...] } }
    }
  }
}
```

---

### 7. 框架数据（跨公司XBRL期间数据）

#### `GET https://data.sec.gov/api/xbrl/frames/{taxonomy}/{tag}/{unit}/{period}.json`
返回特定报告期内所有公司针对某XBRL概念的报告值。

**路径参数：**
| 参数       | 说明 |
|------------|-------------|
| `taxonomy` | `us-gaap`, `ifrs-full`, `dei`, `srt` |
| `tag`      | XBRL标签，如`Assets` |
| `unit`     | `USD`, `shares`, `pure`等 |
| `period`   | 时点值：`CY2023Q4I`；期间值：`CY2023`, `CY2023Q1` |

**期间格式：**
- `CY2023` = 2023日历年（全年期间）
- `CY2023Q1` = 2023年第一季度期间
- `CY2023Q4I` = 2023年第四季度末时点值（资产负债表项目）

**示例：**
```
https://data.sec.gov/api/xbrl/frames/us-gaap/Assets/USD/CY2023Q4I.json
```

**响应：**
```json
{
  "taxonomy": "us-gaap",
  "tag": "Assets",
  "ccp": "CY2023Q4I",
  "uom": "USD",
  "label": "资产总额",
  "description": "各项资产账面价值总和...",
  "pts": 8500,
  "data": [
    {"accn": "0000320193-24-000123", "cik": 320193, "entityName": "苹果公司", "loc": "US-CA", "end": "2023-12-30", "val": 352583000000}
  ]
}
```

---

### 8. 申报档案库（直接文档访问）

#### `GET https://www.sec.gov/Archives/edgar/data/{cik}/{accession_number_no_dashes}/{filename}`
直接访问任意申报文档。

**示例：**
```
https://www.sec.gov/Archives/edgar/data/320193/000032019324000123/aapl-20240928.htm
```

URL中的登记号需去除短横线：`0000320193-24-000123` 转为 `000032019324000123`。

---

## 常用XBRL标签参考
| 标签 | 说明 |
|-----|-------------|
| `Revenue` / `Revenues` | 营业收入总额 |
| `NetIncomeLoss` | 净收益 |
| `Assets` | 资产总额 |
| `Liabilities` | 负债总额 |
| `StockholdersEquity` | 股东权益总额 |
| `EarningsPerShareBasic` | 基本每股收益 |
| `EarningsPerShareDiluted` | 稀释每股收益 |
| `OperatingIncomeLoss` | 营业利润 |
| `CashAndCashEquivalentsAtCarryingValue` | 现金及现金等价物 |
| `LongTermDebt` | 长期债务 |
| `CommonStockSharesOutstanding` | 流通普通股数量 |

## 注意事项
- 在`data.sec.gov`的URL中，CIK编号必须补零至10位
- EFTS全文搜索仅索引申报文件文本内容，不包含XBRL数据
- 批量下载可使用SEC提供的索引文件：`https://www.sec.gov/Archives/edgar/full-index/`
- 除申报文档可能为HTML/XML/纯文本外，所有响应均为JSON格式
