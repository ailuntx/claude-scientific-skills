# 证券与储蓄债券数据集 — 美国财政部财政数据

## 国债拍卖数据

**端点：** `/v1/accounting/od/auctions_query`  
**更新频率：** 按需  
**日期范围：** 1979年11月至今

包含国债拍卖历史数据，涵盖国库券、中期票据、长期债券、通胀保值债券（TIPS）及浮动利率票据（FRN）。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | DATE | 拍卖日期 |
| `security_type` | STRING | 证券类型（国库券/中期票据/长期债券/TIPS/FRN） |
| `security_term` | STRING | 期限（如"4周"、"2年"、"10年"） |
| `cusip` | STRING | CUSIP证券识别码 |
| `offering_amt` | CURRENCY | 发行总额 |
| `accepted_comp_bid_rate_amt` | PERCENTAGE | 最高中标竞争性投标利率 |
| `bid_to_cover_ratio` | NUMBER | 投标覆盖率 |
| `total_accepted_amt` | CURRENCY | 中标总额 |
| `indirect_bid_pct_accepted` | PERCENTAGE | 间接投标者中标比例 |
| `issue_date` | DATE | 发行/结算日期 |
| `maturity_date` | DATE | 到期日 |

```python
# 获取近期10年期国债拍卖数据
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/od/auctions_query",
    params={
        "filter": "security_type:eq:Note,security_term:eq:10-Year",
        "sort": "-record_date",
        "page[size]": 10
    }
)
df = pd.DataFrame(resp.json()["data"])

# 获取2024年所有拍卖数据
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/od/auctions_query",
    params={
        "filter": "record_date:gte:2024-01-01,record_date:lte:2024-12-31",
        "sort": "-record_date",
        "page[size]": 10000
    }
)
```

## 国债即将拍卖数据

**端点：** `/v1/accounting/od/upcoming_auctions`  
**更新频率：** 按需  
**日期范围：** 2024年3月至今

已公布但尚未结算的拍卖计划。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `auction_date` | DATE | 预定拍卖日期 |
| `security_type` | STRING | 证券类型 |
| `security_term` | STRING | 期限 |
| `offering_amt` | CURRENCY | 公布发行额 |

```python
# 获取即将举行的拍卖
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/od/upcoming_auctions",
    params={"sort": "auction_date"}
)
upcoming = pd.DataFrame(resp.json()["data"])
print(upcoming[["auction_date", "security_type", "security_term", "offering_amt"]])
```

## 创纪录国债拍卖数据

**更新频率：** 按需

追踪各类证券期限的拍卖记录（最大规模、最高利率、最低利率等）。

## 国债回购数据

**更新频率：** 按需（2张数据表）  
**日期范围：** 2000年3月至今

国债二级市场回购操作数据。自2024年计划重启后持续更新。

---

## I系列债券利率

**端点：** `/v2/accounting/od/i_bond_interest_rates`  
**更新频率：** 半年（5月与11月）  
**日期范围：** 1998年9月至今

I系列储蓄债券综合利率数据，包含固定利率与通胀率成分。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `effective_date` | DATE | 利率生效日期 |
| `announcement_date` | DATE | 公布日期 |
| `fixed_rate` | PERCENTAGE | 固定利率成分 |
| `semiannual_inflation_rate` | PERCENTAGE | 半年期CPI-U通胀率 |
| `earnings_rate_i_bonds` | PERCENTAGE | 综合收益率 |

```python
# 当前I债券利率
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/i_bond_interest_rates",
    params={"sort": "-effective_date", "page[size]": 5}
)
df = pd.DataFrame(resp.json()["data"])
latest = df.iloc[0]
print(f"当前I债券利率: {latest['earnings_rate_i_bonds']}%")
print(f"  固定利率: {latest['fixed_rate']}%")
print(f"  通胀成分: {latest['semiannual_inflation_rate']}%")
```

## 美国储蓄债券：发行、赎回与到期

**端点：** `/v1/accounting/od/sb_issues_redemptions`（3张表）  
**更新频率：** 月度  
**日期范围：** 1998年9月至今

EE系列、I系列及HH系列储蓄债券的月度统计数据，涵盖流通量、发行量与赎回量。

**关键字段：**
| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | DATE | 月末日期 |
| `series_cd` | STRING | 债券系列（EE/I/HH） |
| `issued_amt` | CURRENCY | 发行金额 |
| `redeemed_amt` | CURRENCY | 赎回金额 |
| `matured_amt` | CURRENCY | 到期金额 |
| `outstanding_amt` | CURRENCY | 流通总额 |

## 储蓄债券价值文件

**更新频率：** 半年  
**日期范围：** 1992年5月至今

用于计算储蓄债券当前赎回价值的文件。

## 累积型储蓄债券赎回表（已停用）

**端点：** `/v2/accounting/od/redemption_tables`  
**更新频率：** 已停用（最后更新于2022年）  
**日期范围：** 1999年3月 – 2023年5月

历史储蓄债券月度赎回价值表。

## 已售储蓄债券（已停用）

**更新频率：** 已停用  
**日期范围：** 1998年10月 – 2022年6月

---

## 州及地方政府证券（SLGS）

**端点：** `/v1/accounting/od/slgs_statistics`  
**更新频率：** 每日  
**日期范围：** 1998年10月至今

SLGS证券流通数据——面向州及地方政府发行的非市场化特殊用途证券。

## 月度州及地方政府证券（SLGS）计划

**更新频率：** 月度  
**日期范围：** 2014年3月至今

SLGS计划的月度统计数据。

---

## 电子证券交易

**更新频率：** 月度（8张数据表）  
**日期范围：** 2000年1月至今

TRADES系统（国债/储备自动债务录入系统）中的国债电子簿记交易数据。

---

## 联邦投资计划

### 基金利息成本
**更新频率：** 月度  
**日期范围：** 2001年10月至今

政府信托基金投资联邦资金的月度利息成本。

### 未偿本金
**更新频率：** 月度（2张表）  
**日期范围：** 2017年10月至今

### 账户报表
**更新频率：** 月度（3张表）  
**日期范围：** 2011年11月至今

---

## 联邦借贷计划

### 分配与交易数据
**更新频率：** 每日（2张表）  
**日期范围：** 2000年9月至今

### 未投资资金利息
**更新频率：** 季度  
**日期范围：** 2016年12月至今

### 总账余额摘要报告
**更新频率：** 月度（2张表）  
**日期范围：** 2005年10月至今
