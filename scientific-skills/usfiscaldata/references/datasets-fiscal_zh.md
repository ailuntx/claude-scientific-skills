# 财政报表数据集 — 美国财政部财政数据

## 每日财政报表 (DTS)

DTS数据集包含**9个数据表**，均位于`/v1/accounting/dts/`路径下。每日更新（工作日）。

**日期范围：** 2005年10月至今

### DTS数据表

| 表名称 | 端点路径 | 描述 |
|-------|----------|-------------|
| 运营现金余额 | `/v1/accounting/dts/operating_cash_balance` | 财政部总账户余额 |
| 存款与提款 | `/v1/accounting/dts/deposits_withdrawals_operating_cash` | 财政部总账户变动 |
| 公共债务交易 | `/v1/accounting/dts/public_debt_transactions` | 证券发行与赎回 |
| 公共债务调整 | `/v1/accounting/dts/adjustment_public_debt_transactions_cash_basis` | 现金制调整 |
| 法定限额内债务 | `/v1/accounting/dts/debt_subject_to_limit` | 债务与法定上限对比 |
| 跨机构税收转移 | `/v1/accounting/dts/inter_agency_tax_transfers` | 政府内部税收转移 |
| 联邦税收存款 | `/v1/accounting/dts/federal_tax_deposits` | 税收存款活动 |
| 短期现金投资 | `/v1/accounting/dts/short_term_cash_investments` | 现金投资活动 |
| 已发放所得税退税 | `/v1/accounting/dts/income_tax_refunds_issued` | 退税发放情况 |

### 通用DTS字段

| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | DATE | 业务日期 |
| `account_type` | STRING | 账户/余额类型 |
| `open_today_bal` | CURRENCY | 期初余额 |
| `open_month_bal` | CURRENCY | 月初余额 |
| `open_fiscal_year_bal` | CURRENCY | 财年初余额 |
| `close_today_bal` | CURRENCY | 期末余额 |
| `transaction_today_amt` | CURRENCY | 当日交易金额 |
| `transaction_mtd_amt` | CURRENCY | 当月累计金额 |
| `transaction_fytd_amt` | CURRENCY | 财年累计金额 |

```python
# 获取当前财政部总账户(TGA)余额
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/dts/operating_cash_balance",
    params={"sort": "-record_date", "page[size]": 5}
)
for row in resp.json()["data"]:
    print(f"{row['record_date']}: ${float(row['close_today_bal']):,.0f}M (期末余额)")

# 获取特定期间存款与提款数据
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/dts/deposits_withdrawals_operating_cash",
    params={
        "filter": "record_date:gte:2024-01-01,record_date:lte:2024-01-31",
        "sort": "record_date",
        "page[size]": 1000
    }
)
```

### 聚合示例 (DTS)

```python
# 按交易类型汇总当日交易金额
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/dts/deposits_withdrawals_operating_cash",
    params={
        "fields": "record_date,transaction_type,transaction_today_amt",
        "filter": "record_date:eq:2024-01-15"
    }
)
```

---

## 月度财政报表 (MTS)

MTS数据集包含**16个数据表**，均位于`/v1/accounting/mts/`路径下。每月更新。

**日期范围：** 1980年10月至今

### MTS数据表

| 表名称 | 端点路径 | 描述 |
|-------|----------|-------------|
| MTS表1 | `/v1/accounting/mts/mts_table_1` | 收支概要 |
| MTS表2 | `/v1/accounting/mts/mts_table_2` | 收入来源分类 |
| MTS表3 | `/v1/accounting/mts/mts_table_3` | 按功能分类支出 |
| MTS表4 | `/v1/accounting/mts/mts_table_4` | 按机构分类支出 |
| MTS表5 | `/v1/accounting/mts/mts_table_5` | 按类别分类支出 |
| MTS表6 | `/v1/accounting/mts/mts_table_6` | 融资方式 |
| MTS表7 | `/v1/accounting/mts/mts_table_7` | 收入来源分类（季度） |
| MTS表8 | `/v1/accounting/mts/mts_table_8` | 按功能分类支出（季度） |
| MTS表9 | `/v1/accounting/mts/mts_table_9` | 收入：对比概要 |
| MTS表10 | `/v1/accounting/mts/mts_table_10` | 支出：对比概要 |
| MTS表11 | `/v1/accounting/mts/mts_table_11` | 收入补充明细 |
| MTS表12 | `/v1/accounting/mts/mts_table_12` | 支出补充明细 |
| MTS表13 | `/v1/accounting/mts/mts_table_13` | 联邦借贷与债务 |
| MTS表14 | `/v1/accounting/mts/mts_table_14` | 融资方式：联邦 |
| MTS表15 | `/v1/accounting/mts/mts_table_15` | 联邦信托基金概要 |
| MTS表16 | `/v1/accounting/mts/mts_table_16` | 融资方式：预算外 |

### 通用MTS字段

| 字段 | 类型 | 描述 |
|-------|------|-------------|
| `record_date` | DATE | 月末日期 |
| `record_fiscal_year` | STRING | 财年（10月-9月） |
| `record_fiscal_quarter` | STRING | 财季（1-4） |
| `classification_desc` | STRING | 科目描述 |
| `classification_id` | STRING | 科目代码 |
| `parent_id` | STRING | 上级分类ID |
| `current_month_gross_rcpt_amt` | CURRENCY | 当月总收入 |
| `current_fytd_gross_rcpt_amt` | CURRENCY | 财年累计总收入 |
| `prior_fytd_gross_rcpt_amt` | CURRENCY | 上年同期财年累计 |

```python
# MTS表1：收支概要
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/mts/mts_table_1",
    params={
        "filter": "record_fiscal_year:eq:2024",
        "sort": "record_date"
    }
)
df = pd.DataFrame(resp.json()["data"])

# MTS表9：获取最近期间120行（总收入）
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/mts/mts_table_9",
    params={
        "filter": "line_code_nbr:eq:120",
        "sort": "-record_date",
        "page[size]": 1
    }
)
```

---

## 美国政府收入征收

**端点路径：** `/v1/accounting/od/rev_collections`  
**更新频率：** 每日  
**日期范围：** 2004年10月至今

每日税收与非税收入征收数据。

---

## 美国政府财务报告

**端点路径：** (8个表)  
**更新频率：** 年度  
**日期范围：** 1995年9月至今（FY2024最新）

年度审计财务报表。包含：
- 资产负债表
- 净成本表
- 运营报表
- 净资产变动表

---

## 月度财政支出

**更新频率：** 月度  
**日期范围：** 2013年10月至今

月度联邦支出数据。

---

## 按部门分类收入

**端点路径：** `/v2/accounting/od/receipts_by_dept`  
**更新频率：** 年度  
**日期范围：** 2015年9月至今

按部门分类的联邦收入年度明细。

---

## 财政部管理账户

**更新频率：** 季度  
**日期范围：** 2022年12月至今（3个数据表）

财政部管理的信托及专项基金账户数据。

---

## 财政部公报

**更新频率：** 季度  
**日期范围：** 2021年3月至今（13个表）

涵盖政府财政、公共债务、储蓄债券等的季度财务报告。

**端点前缀：** `/v1/accounting/od/treasury_bulletin_`
