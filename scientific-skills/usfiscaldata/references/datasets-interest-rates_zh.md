# 利率与汇率数据集 — 美国财政部财政数据

## 美国国债平均利率

**端点：** `/v2/accounting/od/avg_interest_rates`  
**更新频率：** 月度  
**时间范围：** 2001年1月至今

按证券类型分类的可流通与不可流通国债平均利率。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | DATE | 月末日期 |
| `security_desc` | STRING | 证券描述（如"国库券"） |
| `security_type_desc` | STRING | "可流通"或"不可流通" |
| `avg_interest_rate_amt` | PERCENTAGE | 平均利率（%） |

```python
# 获取所有可流通证券最新月度平均利率
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/avg_interest_rates",
    params={
        "filter": "security_type_desc:eq:Marketable",
        "sort": "-record_date",
        "page[size]": 50
    }
)
df = pd.DataFrame(resp.json()["data"])
latest = df[df["record_date"] == df["record_date"].max()]
print(latest[["security_desc", "avg_interest_rate_amt"]])

# 获取特定证券类型历史利率
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/avg_interest_rates",
    params={
        "fields": "record_date,avg_interest_rate_amt",
        "filter": "security_desc:eq:Treasury Notes,record_date:gte:2010-01-01",
        "sort": "-record_date"
    }
)
```

**常见证券描述：**
- `国库券`
- `中期国债`
- `长期国债`
- `通胀保值国债 (TIPS)`
- `浮动利率票据 (FRN)`
- `联邦融资银行`
- `美国储蓄证券`
- `政府账户系列`
- `可流通证券总计`
- `不可流通证券总计`
- `计息债务总额`

---

## 财政部报告汇率

**端点：** `/v1/accounting/od/rates_of_exchange`  
**更新频率：** 季度  
**时间范围：** 2001年3月至今

联邦机构用于报告目的的官方财政部外汇汇率。每季度更新（3月31日、6月30日、9月30日、12月31日）。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | DATE | 季度末日期 |
| `country` | STRING | 国家名称 |
| `currency` | STRING | 货币名称 |
| `country_currency_desc` | STRING | "国家-货币"组合（如"加拿大-加元"） |
| `exchange_rate` | NUMBER | 每1美元兑换外币单位数 |
| `effective_date` | DATE | 汇率生效日期 |

```python
# 获取所有当前汇率（最新季度）
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/od/rates_of_exchange",
    params={"sort": "-record_date", "page[size]": 200}
)
df = pd.DataFrame(resp.json()["data"])
latest_date = df["record_date"].max()
current_rates = df[df["record_date"] == latest_date].copy()
current_rates["exchange_rate"] = current_rates["exchange_rate"].astype(float)
print(current_rates[["country_currency_desc", "exchange_rate"]].to_string())

# 欧元汇率历史数据
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/od/rates_of_exchange",
    params={
        "fields": "record_date,exchange_rate",
        "filter": "country_currency_desc:eq:Euro Zone-Euro",
        "sort": "-record_date",
        "page[size]": 100
    }
)
euro_df = pd.DataFrame(resp.json()["data"])
euro_df["exchange_rate"] = euro_df["exchange_rate"].astype(float)
euro_df["record_date"] = pd.to_datetime(euro_df["record_date"])
```

---

## TIPS与CPI数据

**端点：** `/v1/accounting/od/tips_cpi_data`  
**更新频率：** 月度  
**时间范围：** 1998年4月至今（2个数据表）

通胀保值国债（TIPS）参考CPI数据及用于计算TIPS价值的指数比率。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | DATE | 记录日期 |
| `index_ratio` | NUMBER | TIPS调整指数比率 |
| `ref_cpi` | NUMBER | 参考CPI值 |

---

## FRN每日指数

**端点：** `/v1/accounting/od/frn_daily_indexes`  
**更新频率：** 每日  
**时间范围：** 2024年4月至今

浮动利率票据（FRN）每日指数值。该利率基于13周国库券拍卖利率。

---

## 财政部认证利率

包含四个认证周期，每个周期有独立端点：

### 年度认证
**更新频率：** 年度  
**时间范围：** 2006年10月至今（9个数据表）

### 月度认证  
**更新频率：** 月度  
**时间范围：** 2006年10月至今（6个数据表）

### 季度认证
**更新频率：** 季度  
**时间范围：** 2006年10月至今（4个数据表）

### 半年度认证
**更新频率：** 半年度  
**时间范围：** 2008年1月至今（1个数据表）

这些认证利率用于联邦贷款、融资计划及其他需要财政部官方认证利率的场景。

---

## 联邦信贷相似期限利率

**端点：** `/v1/accounting/od/fed_credit_similar_maturity_rates`  
**更新频率：** 年度  
**时间范围：** 1992年9月至今

根据《联邦信贷改革法案》用于评估联邦信贷计划（贷款及贷款担保）的利率。

---

## 历史合格税收抵免债券利率

**更新频率：** 每日（已停更）  
**时间范围：** 2009年3月 – 2018年1月

合格税收抵免债券（QTCB）历史利率数据，已停止更新。

---

## 州及地方政府系列（SLGS）每日利率表

**端点：** `/v1/accounting/od/slgs_savings_bonds`（2个表）  
**更新频率：** 每日  
**时间范围：** 1992年6月至今

州及地方政府系列证券每日利率，供州和地方发行机构遵守联邦税法套利限制规定使用。
