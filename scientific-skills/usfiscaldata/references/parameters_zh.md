# 查询参数 — 美国财政部财政数据 API

所有参数均为可选。在 URL 查询字符串中使用 `&` 组合参数。

## `fields=` — 选择列

仅返回指定字段。接受逗号分隔的字段名列表。

```
?fields=record_date,tot_pub_debt_out_amt
?fields=country_currency_desc,exchange_rate,record_date
```

- 若省略则返回所有字段
- 无效字段名将导致错误
- 省略部分字段可能触发**自动聚合**（见下文）

### 聚合 / 自动求和

当 `fields=` 参数排除部分非数值字段时，API 会自动按剩余字段分组并求和数值。

```python
# 返回按 record_date 和 transaction_type 分组后的交易金额总和
params = {
    "fields": "record_date,transaction_type,transaction_today_amt"
}
```

## `filter=` — 筛选记录

通过字段值缩小结果范围。多个字段筛选器需**在单个 `filter=` 参数内用逗号分隔**。

### 筛选语法

```
filter=<字段>:<运算符>:<值>
filter=<字段>:<运算符>:<值>,<字段>:<运算符>:<值>
```

### 运算符

| 运算符 | 含义         | 示例                          |
|--------|--------------|-------------------------------|
| `eq`   | 等于         | `filter=record_date:eq:2024-03-31` |
| `lt`   | 小于         | `filter=exchange_rate:lt:1.5`      |
| `lte`  | 小于或等于   | `filter=record_date:lte:2024-12-31` |
| `gt`   | 大于         | `filter=record_fiscal_year:gt:2010` |
| `gte`  | 大于或等于   | `filter=record_date:gte:2024-01-01` |
| `in`   | 包含在集合中 | `filter=country_currency_desc:in:(Canada-Dollar,Mexico-Peso)` |

### 日期筛选

日期使用 `YYYY-MM-DD` 格式：

```
filter=record_date:gte:2024-01-01
filter=record_date:gte:2023-01-01,record_date:lte:2023-12-31
```

### 多字段筛选

```
filter=country_currency_desc:in:(Canada-Dollar,Mexico-Peso),record_date:gte:2024-01-01
```

### 常用筛选字段

多数端点包含以下标准日期字段：
- `record_date` — 记录日期 (YYYY-MM-DD)
- `record_fiscal_year` — 财政年度 (如 `2024`)
- `record_fiscal_quarter` — 财政季度 (1-4)
- `record_calendar_year` — 日历年
- `record_calendar_month` — 日历月 (01-12)

## `sort=` — 结果排序

按一个或多个字段排序。前缀 `-` 表示降序。

```
?sort=-record_date           # 最近日期优先
?sort=record_date            # 最早日期优先
?sort=-record_fiscal_year,-record_fiscal_quarter  # 嵌套排序
```

**默认：** 按首列排序（通常为 `record_date` 升序）。

## `format=` — 输出格式

```
?format=json    # 默认值
?format=csv     # 逗号分隔值
?format=xml     # XML
```

使用 CSV 或 XML 格式时，响应返回原始文件内容而非 JSON。

## `page[size]=` 与 `page[number]=` — 分页控制

控制每页记录数和返回的页码。

```
?page[size]=100&page[number]=1    # 默认值（100条记录，第1页）
?page[size]=10000                 # 大页数减少请求次数
?page[number]=5&page[size]=50     # 从第5页开始返回50条记录
```

- 默认页大小：**100**
- 默认页码：**1**
- 通过响应中的 `meta.total-pages` 获取总页数
- 通过 `meta.total-count` 获取总记录数

### 获取所有记录

```python
import requests
import pandas as pd

def fetch_all(endpoint, params=None):
    """获取所有分页数据并返回为DataFrame"""
    params = dict(params or {})
    params["page[size]"] = 10000
    params["page[number]"] = 1
    
    base = "https://api.fiscaldata.treasury.gov/services/api/fiscal_service"
    all_data = []
    
    while True:
        resp = requests.get(f"{base}{endpoint}", params=params)
        result = resp.json()
        all_data.extend(result["data"])
        
        meta = result["meta"]
        if params["page[number]"] >= meta["total-pages"]:
            break
        params["page[number]"] += 1
    
    return pd.DataFrame(all_data)
```

## 参数组合示例

```python
params = {
    "fields": "country_currency_desc,exchange_rate,record_date",
    "filter": "country_currency_desc:in:(Canada-Dollar,Euro),record_date:gte:2020-01-01",
    "sort": "-record_date",
    "format": "json",
    "page[size]": 100,
    "page[number]": 1
}
resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v1/accounting/od/rates_of_exchange",
    params=params
)
```
