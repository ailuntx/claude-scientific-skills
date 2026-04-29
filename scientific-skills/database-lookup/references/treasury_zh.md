# 美国财政部财政数据 API 参考

## 概述
美国财政部财政数据 API 提供对联邦财务数据的机器可读访问：包括国债、国库证券、利率、收益率曲线、财政收入、支出等。由财政服务局维护。

## 基础 URL
```
https://api.fiscaldata.treasury.gov/services/api/fiscal_service
```

## 认证
**无需 API 密钥。** 该 API 完全开放且公开。

## 速率限制
- **未公布正式速率限制。**
- 预期合理使用；无认证或节流机制记录。
- 批量数据请使用大分页尺寸进行分页。

---

## 关键端点

### URL 模式
所有数据集端点遵循：
```
GET /services/api/fiscal_service/{endpoint}?{parameters}
```

### 通用查询参数（适用于所有端点）
| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `fields` | 字符串 | 返回字段的逗号分隔列表 |
| `filter` | 字符串 | 过滤表达式：`字段:操作符:值` (例：`record_date:gte:2024-01-01`) |
| `sort` | 字符串 | 排序字段：`字段` (升序) 或 `-字段` (降序)；逗号分隔 |
| `page[number]` | 整数 | 页码 (默认 1) |
| `page[size]` | 整数 | 每页结果数 (默认 100，最大 10000) |
| `format` | 字符串 | `json` (默认) 或 `csv` |

**过滤操作符：**
`eq` (等于), `lt`, `lte`, `gt`, `gte`, `in` (逗号分隔值)

---

### 1. 国债收益率曲线利率（每日）
```
GET /v2/accounting/od/avg_interest_rates
```

**更优的收益率曲线端点：**
```
GET /v1/accounting/od/rates_of_exchange
```

**每日国债票面收益率曲线利率：**
注意：每日收益率曲线利率发布于 `https://home.treasury.gov/resource-center/data-chart-center/interest-rates/` 并通过 TreasuryDirect API 提供。如需通过财政数据以编程方式访问：

```
GET /v2/accounting/od/avg_interest_rates
```

**示例——国债证券平均利率：**
```
https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/avg_interest_rates?filter=record_date:gte:2024-01-01&sort=-record_date&page[size]=100
```

**响应：**
```json
{
  "data": [
    {
      "record_date": "2024-10-31",
      "security_type_desc": "Treasury Bills",
      "security_desc": "Treasury Bills",
      "avg_interest_rate_amt": "5.223",
      "src_line_nbr": "1",
      "record_fiscal_year": "2025",
      "record_fiscal_quarter": "1",
      "record_calendar_year": "2024",
      "record_calendar_quarter": "4",
      "record_calendar_month": "10",
      "record_calendar_day": "31"
    }
  ],
  "meta": {
    "count": 100,
    "labels": { ... },
    "dataTypes": { ... },
    "dataFormats": { ... },
    "total-count": 1234,
    "total-pages": 13
  },
  "links": {
    "self": "&page%5Bnumber%5D=1&page%5Bsize%5D=100",
    "first": "&page%5Bnumber%5D=1&page%5Bsize%5D=100",
    "prev": null,
    "next": "&page%5Bnumber%5D=2&page%5Bsize%5D=100",
    "last": "&page%5Bnumber%5D=13&page%5Bsize%5D=100"
  }
}
```

---

### 2. 精确到美分的债务（每日国债）
```
GET /v2/accounting/od/debt_to_penny
```

**示例——2024年以来的债务：**
```
https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny?filter=record_date:gte:2024-01-01&sort=-record_date&page[size]=10
```

**关键字段：** `record_date`, `tot_pub_debt_out_amt`, `intragov_hold_amt`, `debt_held_public_amt`

---

### 3. 国库证券拍卖
```
GET /v1/accounting/od/auctions_query
```

**示例——近期短期国库券拍卖：**
```
https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/od/auctions_query?filter=security_type:eq:Bill&sort=-auction_date&page[size]=10
```

**关键字段：** `cusip`, `security_type`, `security_term`, `auction_date`, `issue_date`, `maturity_date`, `high_yield`, `high_discount_rate`, `bid_to_cover_ratio`, `total_accepted`

---

### 4. 月度财政报表（收入与支出）
```
GET /v1/accounting/mts/mts_table_5
```

**示例——联邦收入/支出：**
```
https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/mts/mts_table_5?filter=record_date:gte:2024-01-01&sort=-record_date&page[size]=50
```

---

### 5. 按类别划分的联邦支出
```
GET /v1/accounting/mts/mts_table_9
```

**示例：**
```
https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/mts/mts_table_9?filter=record_date:gte:2024-01-01&sort=-record_date
```

---

### 6. 财政部报告汇率
```
GET /v1/accounting/od/rates_of_exchange
```

**示例——季度汇率：**
```
https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/od/rates_of_exchange?filter=record_date:eq:2024-09-30&page[size]=200
```

**关键字段：** `country_currency_desc`, `exchange_rate`, `record_date`, `effective_date`

---

### 7. 债务利息支出
```
GET /v2/accounting/od/interest_expense
```

**示例：**
```
https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/interest_expense?filter=record_fiscal_year:eq:2024&sort=-record_date
```

---

### 8. 储蓄债券利率
```
GET /v2/accounting/od/sb_value
```

---

## 常用端点路径

| 端点 | 描述 |
|----------|-------------|
| `v2/accounting/od/debt_to_penny` | 每日未偿公共债务总额 |
| `v2/accounting/od/avg_interest_rates` | 国债证券平均利率 |
| `v1/accounting/od/auctions_query` | 国库证券拍卖结果 |
| `v1/accounting/od/rates_of_exchange` | 财政部报告汇率 |
| `v2/accounting/od/interest_expense` | 公共债务利息支出 |
| `v1/accounting/mts/mts_table_5` | 月度财政报表：收入/支出 |
| `v1/accounting/mts/mts_table_9` | 月度财政报表：按功能划分的支出 |
| `v2/accounting/od/statement_net_cost` | 净成本报表 |
| `v2/accounting/od/debt_outstanding` | 历史未偿债务（年度） |

## 响应格式
所有 JSON 响应共享相同结构：
- `data`：结果对象数组
- `meta`：包含 `count`, `total-count`, `total-pages`, 字段标签及数据类型
- `links`：分页链接 (`self`, `first`, `prev`, `next`, `last`)

## 注意事项
- 所有货币金额均以字符串形式返回以保持精度。
- 日期在 `record_date` 字段中使用 `YYYY-MM-DD` 格式。
- `filter` 参数支持链式操作：`filter=字段1:eq:值1,字段2:gte:值2`。
- 使用 `fields=` 通过仅请求所需列来减小响应体积。
- API 文档和数据集浏览器位于：https://fiscaldata.treasury.gov/api-documentation/
- 对于国债收益率曲线利率，FRED 序列 `DGS1`, `DGS2`, `DGS5`, `DGS10`, `DGS30` 可能更为便捷。
