# 代码示例 — 美国财政部财政数据

## Python 示例

### 初始化设置

```python
import requests
import pandas as pd

BASE_URL = "https://api.fiscaldata.treasury.gov/services/api/fiscal_service"

def fetch(endpoint, **params):
    resp = requests.get(f"{BASE_URL}{endpoint}", params=params)
    resp.raise_for_status()
    return resp.json()
```

### 国债追踪器

```python
# 当前公共债务总额
result = fetch("/v2/accounting/od/debt_to_penny", 
               sort="-record_date", **{"page[size]": 1})
d = result["data"][0]
debt = float(d["tot_pub_debt_out_amt"])
print(f"截至{d['record_date']}的国债规模: ${debt/1e12:.2f}万亿美元")

# 过去五年债务趋势
result = fetch("/v2/accounting/od/debt_to_penny",
               fields="record_date,tot_pub_debt_out_amt",
               filter="record_date:gte:2020-01-01",
               sort="-record_date", **{"page[size]": 10000})
df = pd.DataFrame(result["data"])
df["date"] = pd.to_datetime(df["record_date"])
df["debt_trillion"] = df["tot_pub_debt_out_amt"].astype(float) / 1e12
df = df.sort_values("date")
print(df[["date", "debt_trillion"]].tail(10))
```

### 联邦汇率查询

```python
# 当前所有财政部汇率
result = fetch("/v1/accounting/od/rates_of_exchange",
               sort="-record_date", **{"page[size]": 300})
df = pd.DataFrame(result["data"])
latest = df[df["record_date"] == df["record_date"].max()]
latest = latest.copy()
latest["exchange_rate"] = latest["exchange_rate"].astype(float)
latest = latest.sort_values("country_currency_desc")
print(latest[["country_currency_desc", "exchange_rate", "record_date"]].to_string(index=False))

# 将美元金额转换为外币
def convert_usd(usd_amount, rates_df):
    rates_df = rates_df.copy()
    rates_df["value_in_foreign"] = usd_amount * rates_df["exchange_rate"].astype(float)
    return rates_df[["country_currency_desc", "value_in_foreign"]]

conversions = convert_usd(1000, latest)
print(conversions.head(10))
```

### 国债拍卖分析

```python
# 近期10年期票据拍卖
result = fetch("/v1/accounting/od/auctions_query",
               filter="security_type:eq:Note,security_term:eq:10-Year",
               sort="-record_date", **{"page[size]": 20})
df = pd.DataFrame(result["data"])
numeric_cols = ["accepted_comp_bid_rate_amt", "bid_to_cover_ratio", 
                "total_accepted_amt", "indirect_bid_pct_accepted"]
for col in numeric_cols:
    if col in df.columns:
        df[col] = pd.to_numeric(df[col], errors="coerce")
print(df[["record_date", "security_term", "accepted_comp_bid_rate_amt", 
         "bid_to_cover_ratio"]].head(10))

# 拍卖收益率趋势：2年期 vs 10年期
def get_auction_yields(term, n=24):
    result = fetch("/v1/accounting/od/auctions_query",
                   fields="record_date,security_term,accepted_comp_bid_rate_amt",
                   filter=f"security_type:eq:Note,security_term:eq:{term}",
                   sort="-record_date", **{"page[size]": n})
    df = pd.DataFrame(result["data"])
    df["yield"] = df["accepted_comp_bid_rate_amt"].astype(float)
    df["date"] = pd.to_datetime(df["record_date"])
    return df[["date", "yield", "security_term"]].sort_values("date")

t2 = get_auction_yields("2-Year")
t10 = get_auction_yields("10-Year")
yield_curve = t2.merge(t10, on="date", suffixes=("_2y", "_10y"), how="inner")
yield_curve["spread"] = yield_curve["yield_10y"] - yield_curve["yield_2y"]
print("收益率曲线利差(10年-2年):")
print(yield_curve[["date", "yield_2y", "yield_10y", "spread"]].tail(10))
```

### 每日财政报表分析

```python
# 近期财政部总账户(TGA)余额
result = fetch("/v1/accounting/dts/operating_cash_balance",
               sort="-record_date", **{"page[size]": 10})
df = pd.DataFrame(result["data"])
print("财政部总账户余额(最近记录):")
for _, row in df.head(5).iterrows():
    bal = float(row["close_today_bal"])
    print(f"  {row['record_date']}: ${bal:,.0f}百万")

# 月度总收支情况
result = fetch("/v1/accounting/dts/deposits_withdrawals_operating_cash",
               fields="record_date,transaction_type,transaction_today_amt",
               filter="record_date:gte:2024-01-01",
               sort="-record_date", **{"page[size]": 10000})
df = pd.DataFrame(result["data"])
df["amount"] = df["transaction_today_amt"].astype(float)
summary = df.groupby(["record_date", "transaction_type"])["amount"].sum().unstack()
print(summary.tail(10))
```

### 月度财政报表(预算)

```python
# 联邦预算收支(MTS表1)
result = fetch("/v1/accounting/mts/mts_table_1",
               filter="record_fiscal_year:eq:2024",
               sort="record_date", **{"page[size]": 1000})
df = pd.DataFrame(result["data"])

# 获取总收入行(行代码可变; 按描述筛选)
receipts = df[df["classification_desc"].str.contains("Total Receipts", na=False, case=False)]
outlays = df[df["classification_desc"].str.contains("Total Outlays", na=False, case=False)]
print("2024财年月度摘要:")
print(receipts[["record_date", "current_month_gross_rcpt_amt"]].head(12))
```

### 利率分析

```python
# 所有可交易国债的平均利率
result = fetch("/v2/accounting/od/avg_interest_rates",
               filter="security_type_desc:eq:Marketable,record_date:gte:2015-01-01",
               sort="-record_date", **{"page[size]": 10000})
df = pd.DataFrame(result["data"])
df["date"] = pd.to_datetime(df["record_date"])
df["rate"] = df["avg_interest_rate_amt"].astype(float)

# 按证券类型对比利率
pivot = df.pivot_table(index="date", columns="security_desc", values="rate")
print(pivot.tail(5))

# I债券利率历史
result = fetch("/v2/accounting/od/i_bond_interest_rates",
               sort="-effective_date", **{"page[size]": 20})
df = pd.DataFrame(result["data"])
df["total_rate"] = df["earnings_rate_i_bonds"].astype(float)
df["fixed_rate"] = df["fixed_rate"].astype(float)
print("I债券利率历史:")
print(df[["effective_date", "fixed_rate", "total_rate"]].head(10))
```

### 财年摘要

```python
def get_fiscal_year_summary(fy: int):
    """获取指定财年的关键财政指标"""
    
    # 财年末总债务
    fy_end = f"{fy}-09-30"
    result = fetch("/v2/accounting/od/debt_to_penny",
                   filter=f"record_date:lte:{fy_end}",
                   sort="-record_date", **{"page[size]": 1})
    debt = float(result["data"][0]["tot_pub_debt_out_amt"]) / 1e12

    # 财年利息支出
    result = fetch("/v2/accounting/od/interest_expense",
                   fields="record_date,expense_net_amt",
                   filter=f"record_fiscal_year:eq:{fy}",
                   **{"page[size]": 10000})
    interest_df = pd.DataFrame(result["data"])
    if not interest_df.empty:
        total_interest = interest_df["expense_net_amt"].astype(float).sum() / 1e9
    else:
        total_interest = None
    
    return {
        "fiscal_year": fy,
        "total_debt_trillion": round(debt, 2),
        "interest_expense_billion": round(total_interest, 1) if total_interest else None
    }

for fy in [2021, 2022, 2023, 2024]:
    summary = get_fiscal_year_summary(fy)
    print(f"FY{fy}: 债务=${summary['total_debt_trillion']}万亿美元, "
          f"利息=${summary['interest_expense_billion']}十亿美元")
```

---

## R 示例

```r
library(httr)
library(jsonlite)

BASE_URL <- "https://api.fiscaldata.treasury.gov/services/api/fiscal_service"

# 国债查询
response <- GET(paste0(BASE_URL, "/v2/accounting/od/debt_to_penny"),
                query = list(sort = "-record_date", `page[size]` = 1))
data <- fromJSON(rawToChar(response$content))$data
cat(sprintf("国债总额: $%.2f万亿美元\n", 
            as.numeric(data$tot_pub_debt_out_amt) / 1e12))

# 汇率查询
response <- GET(paste0(BASE_URL, "/v1/accounting/od/rates_of_exchange"),
                query = list(
                    fields = "country_currency_desc,exchange_rate,record_date",
                    filter = "record_date:gte:2024-01-01",
                    sort = "-record_date",
                    `page[size]` = 200
                ))
rates <- fromJSON(rawToChar(response$content))$data
rates$exchange_rate <- as.numeric(rates$exchange_rate)
head(rates)

# MTS表9：最新总收入
response <- GET(paste0(BASE_URL, "/v1/accounting/mts/mts_table_9"),
                query = list(
                    filter = "line_code_nbr:eq:120",
                    sort = "-record_date",
                    `page[size]` = 1
                ))
mts_data <- fromJSON(rawToChar(response$content))$data
cat("最新总收入行:", mts_data$current_month_gross_rcpt_amt, "\n")
```

---

## 发现可用字段

要查找任何端点的可用字段，可请求小样本并检查`meta.labels`和`meta.dataTypes`：

```python
result = fetch("/v2/accounting/od/debt_to_penny", **{"page[size]": 1})
meta = result["meta"]
for field, label in meta["labels"].items():
    dtype = meta["dataTypes"].get(field, "?")
    fmt = meta["dataFormats"].get(field, "?")
    print(f"{field:40s} | {dtype:12s} | {label}")
```

## 查找数据集

浏览全部54个数据集和182个端点：
- `https://fiscaldata.treasury.gov/datasets/` — 可搜索的数据集目录
- `https://fiscaldata.treasury.gov/api-documentation/#list-of-endpoints-table` — 完整端点列表表
