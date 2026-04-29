# 债务数据集 — 美国财政部财政数据

## 精确到美分的债务数据

**端点：** `/v2/accounting/od/debt_to_penny`  
**更新频率：** 每日  
**日期范围：** 1993-04-01 至今  

追踪每个工作日的精确公共债务总额。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | 日期 | 记录日期 |
| `debt_held_public_amt` | 货币 | 公众持有的债务 |
| `intragov_hold_amt` | 货币 | 政府内部持有额 |
| `tot_pub_debt_out_amt` | 货币 | **公共债务总额** |

```python
# 当前国债
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny",
    params={"sort": "-record_date", "page[size]": 1}
)
latest = resp.json()["data"][0]
print(f"As of {latest['record_date']}: ${float(latest['tot_pub_debt_out_amt']):,.2f}")

# 过去一年债务变化
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny",
    params={
        "fields": "record_date,tot_pub_debt_out_amt",
        "filter": "record_date:gte:2024-01-01",
        "sort": "-record_date"
    }
)
df = pd.DataFrame(resp.json()["data"])
df["tot_pub_debt_out_amt"] = df["tot_pub_debt_out_amt"].astype(float)
```

## 历史债务总额

**端点：** `/v2/accounting/od/historical_debt_outstanding`  
**更新频率：** 年度  
**日期范围：** 1790 年至今  

追溯至美国建国时期的年度国债记录。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | 日期 | 年末日期 |
| `debt_outstanding_amt` | 货币 | 债务总额 |

```python
# 完整历史债务序列
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/historical_debt_outstanding",
    params={"sort": "-record_date", "page[size]": 10000}
)
df = pd.DataFrame(resp.json()["data"])
```

## 联邦债务分类表

**端点：** `/v1/accounting/od/schedules_fed_debt`  
**更新频率：** 月度  
**日期范围：** 2005年10月至今  

按证券类型和成分划分的月度联邦债务明细。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | 日期 | 月末日期 |
| `security_type_desc` | 字符串 | 证券类型 |
| `security_class_desc` | 字符串 | 证券类别 |
| `debt_outstanding_amt` | 货币 | 未偿债务 |

## 每日联邦债务分类表

**端点：** `/v1/accounting/od/schedules_fed_debt_daily`  
**更新频率：** 每日  
**日期范围：** 2006年9月至今  

包含两个数据表的每日版联邦债务分类表。

## 应收账款国库报告 (TROR)

**端点：** `/v2/debt/tror`  
**更新频率：** 季度  
**日期范围：** 2016年12月至今  

联邦机构合规性与应收账款数据。同时包含：
- `/v2/debt/tror/data_act_compliance` — 120天逾期债务转介合规报告

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | 日期 | 季度末日期 |
| `funding_type_desc` | 字符串 | 资金类型 |
| `total_receivables_delinquent_amt` | 货币 | 逾期金额 |

```python
# 按资金类型排序的TROR数据
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/debt/tror",
    params={"sort": "funding_type_id"}
)
```

## 减少公共债务的捐赠

**端点：** `/v2/accounting/od/gift_contributions`  
**更新频率：** 月度  
**日期范围：** 1996年9月至今  

记录公众为减少国债自愿捐赠的款项。

## 公共债务利息支出

**端点：** `/v2/accounting/od/interest_expense`  
**更新频率：** 月度  
**日期范围：** 2010年5月至今  

按证券类型划分的月度利息支出明细。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | 日期 | 月末日期 |
| `security_type_desc` | 字符串 | 证券类型 |
| `expense_net_amt` | 货币 | 净利息支出 |
| `expense_gross_amt` | 货币 | 毛利息支出 |

```python
# 获取月度利息支出总额
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/interest_expense",
    params={
        "fields": "record_date,expense_net_amt",
        "filter": "record_date:gte:2020-01-01",
        "sort": "-record_date"
    }
)
df = pd.DataFrame(resp.json()["data"])
df["expense_net_amt"] = df["expense_net_amt"].astype(float)
```

## 州失业保险基金预付款（第十二编）

**端点：** `/v2/accounting/od/title_xii`  
**更新频率：** 每日  
**日期范围：** 2016年10月至今  

各州及地区从联邦失业保险信托基金借款的记录。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | 日期 | 记录日期 |
| `state_nm` | 字符串 | 州名 |
| `debt_outstanding_amt` | 货币 | 未偿还预付款金额 |
