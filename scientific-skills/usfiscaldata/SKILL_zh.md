---
name: usfiscaldata
description: 查询美国财政部财政数据API，获取联邦财务数据，包括国债、政府支出、收入、利率、汇率和储蓄债券。无需API密钥即可访问54个数据集和182个数据表。适用于处理美国联邦财政数据、国债追踪（Debt to the Penny）、每日财政报告、月度财政报告、国债拍卖、国债利率、外汇汇率、储蓄债券或任何美国政府财务统计数据。
license: MIT
metadata:
    skill-author: K-Dense Inc.
---

# 美国财政部财政数据API

美国财政部提供的免费开放REST API，用于获取联邦财务数据。无需API密钥或注册。

**基础URL:** `https://api.fiscaldata.treasury.gov/services/api/fiscal_service`

## 快速入门

```python
import requests
import pandas as pd

BASE_URL = "https://api.fiscaldata.treasury.gov/services/api/fiscal_service"

# 获取当前国债数据（Debt to the Penny）
resp = requests.get(f"{BASE_URL}/v2/accounting/od/debt_to_penny", params={
    "sort": "-record_date",
    "page[size]": 1
})
data = resp.json()["data"][0]
print(f"截至{data['record_date']}的公共债务总额: ${float(data['tot_pub_debt_out_amt']):,.0f}")
```

```python
# 获取近期季度国债汇率
resp = requests.get(f"{BASE_URL}/v1/accounting/od/rates_of_exchange", params={
    "fields": "country_currency_desc,exchange_rate,record_date",
    "filter": "record_date:gte:2024-01-01",
    "sort": "-record_date",
    "page[size]": 100
})
df = pd.DataFrame(resp.json()["data"])
```

## 认证

无需认证。该API完全开放且免费。

## 核心参数

| 参数 | 示例 | 描述 |
|-----------|---------|-------------|
| `fields=` | `fields=record_date,tot_pub_debt_out_amt` | 选择特定列 |
| `filter=` | `filter=record_date:gte:2024-01-01` | 筛选记录 |
| `sort=` | `sort=-record_date` | 排序（前缀`-`表示降序） |
| `format=` | `format=json` | 输出格式：`json`、`csv`、`xml` |
| `page[size]=` | `page[size]=100` | 每页记录数（默认100） |
| `page[number]=` | `page[number]=2` | 页码索引（从1开始） |

**筛选运算符:** `lt`（小于）、`lte`（小于等于）、`gt`（大于）、`gte`（大于等于）、`eq`（等于）、`in`（包含）

```python
# 多个筛选条件用逗号分隔
"filter=country_currency_desc:in:(Canada-Dollar,Mexico-Peso),record_date:gte:2024-01-01"
```

## 关键数据集与端点

### 债务

| 数据集 | 端点 | 更新频率 |
|---------|----------|-----------|
| Debt to the Penny | `/v2/accounting/od/debt_to_penny` | 每日 |
| 历史未偿债务 | `/v2/accounting/od/historical_debt_outstanding` | 年度 |
| 联邦债务计划表 | `/v1/accounting/od/schedules_fed_debt` | 月度 |

### 每日与月度报告

| 数据集 | 端点 | 更新频率 |
|---------|----------|-----------|
| DTS运营现金余额 | `/v1/accounting/dts/operating_cash_balance` | 每日 |
| DTS存款与提款 | `/v1/accounting/dts/deposits_withdrawals_operating_cash` | 每日 |
| 月度财政报告（MTS） | `/v1/accounting/mts/mts_table_1`（16个表格） | 月度 |

### 利率与汇率

| 数据集 | 端点 | 更新频率 |
|---------|----------|-----------|
| 国债平均利率 | `/v2/accounting/od/avg_interest_rates` | 月度 |
| 财政部汇率报告 | `/v1/accounting/od/rates_of_exchange` | 季度 |
| 公共债务利息支出 | `/v2/accounting/od/interest_expense` | 月度 |

### 证券与拍卖

| 数据集 | 端点 | 更新频率 |
|---------|----------|-----------|
| 国债拍卖数据 | `/v1/accounting/od/auctions_query` | 按需 |
| 即将举行的国债拍卖 | `/v1/accounting/od/upcoming_auctions` | 按需 |
| 平均利率 | `/v2/accounting/od/avg_interest_rates` | 月度 |

### 储蓄债券

| 数据集 | 端点 | 更新频率 |
|---------|----------|-----------|
| I债券利率 | `/v2/accounting/od/i_bond_interest_rates` | 半年度 |
| 美国财政部储蓄债券：发行、赎回与到期 | `/v1/accounting/od/sb_issues_redemptions` | 月度 |

## 响应结构

```json
{
  "data": [...],
  "meta": {
    "count": 100,
    "total-count": 3790,
    "total-pages": 38,
    "labels": {"field_name": "可读标签"},
    "dataTypes": {"field_name": "STRING|NUMBER|DATE|CURRENCY"},
    "dataFormats": {"field_name": "String|10.2|YYYY-MM-DD"}
  },
  "links": {"self": "...", "first": "...", "prev": null, "next": "...", "last": "..."}
}
```

**注意：** 所有值均以字符串形式返回。需按需转换（如`float()`、`pd.to_datetime()`）。空值显示为字符串`"null"`。

## 常用模式

### 将所有页面加载到DataFrame

```python
def fetch_all_pages(endpoint, params=None):
    params = params or {}
    params["page[size]"] = 10000  # 最大化页面尺寸以减少请求次数
    resp = requests.get(f"{BASE_URL}{endpoint}", params=params)
    result = resp.json()
    df = pd.DataFrame(result["data"])
    return df
```

### 自动聚合（自动求和）

省略分组字段将触发自动聚合：

```python
# 按记录日期和交易类型汇总所有存款/提款
resp = requests.get(f"{BASE_URL}/v1/accounting/dts/deposits_withdrawals_operating_cash", params={
    "fields": "record_date,transaction_type,transaction_today_amt"
})
```

## 参考文件

- **[api-basics.md](references/api-basics.md)** — URL结构、HTTP方法、版本控制、数据类型
- **[parameters.md](references/parameters.md)** — 所有参数详解及示例
- **[datasets-debt.md](references/datasets-debt.md)** — 债务数据集：Debt to the Penny、历史债务、联邦债务计划表
- **[datasets-fiscal.md](references/datasets-fiscal.md)** — 每日财政报告、月度财政报告、收入与支出
- **[datasets-interest-rates.md](references/datasets-interest-rates.md)** — 平均利率、汇率、TIPS/CPI、核定利率
- **[datasets-securities.md](references/datasets-securities.md)** — 国债拍卖、储蓄债券、SLGS、回购
- **[response-format.md](references/response-format.md)** — 响应对象、错误处理、分页、响应代码
- **[examples.md](references/examples.md)** — 常见用例的Python/R/pandas代码示例
