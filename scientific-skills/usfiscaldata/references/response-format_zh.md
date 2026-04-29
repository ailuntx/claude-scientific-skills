# 响应格式 — 美国财政部财政数据 API

## 响应结构 (JSON)

```json
{
  "data": [
    {
      "record_date": "2024-03-31",
      "tot_pub_debt_out_amt": "34589629941.12"
    }
  ],
  "meta": {
    "count": 100,
    "labels": {
      "record_date": "记录日期",
      "tot_pub_debt_out_amt": "公共债务总额"
    },
    "dataTypes": {
      "record_date": "DATE",
      "tot_pub_debt_out_amt": "CURRENCY"
    },
    "dataFormats": {
      "record_date": "YYYY-MM-DD",
      "tot_pub_debt_out_amt": "10.2"
    },
    "total-count": 3790,
    "total-pages": 38
  },
  "links": {
    "self": "&page%5Bnumber%5D=1&page%5Bsize%5D=100",
    "first": "&page%5Bnumber%5D=1&page%5Bsize%5D=100",
    "prev": null,
    "next": "&page%5Bnumber%5D=2&page%5Bsize%5D=100",
    "last": "&page%5Bnumber%5D=38&page%5Bsize%5D=100"
  }
}
```

## `meta` 对象

| 字段 | 描述 |
|-------|-------------|
| `count` | 当前响应页中的记录数 |
| `total-count` | 符合查询条件的总记录数（所有页面） |
| `total-pages` | 当前分页大小下的总页数 |
| `labels` | 人类可读的列标签 |
| `dataTypes` | 逻辑数据类型：`STRING`, `NUMBER`, `DATE`, `CURRENCY`, `INTEGER`, `PERCENTAGE` |
| `dataFormats` | 格式提示：`YYYY-MM-DD`, `10.2`（10位数字，2位小数）, `String` |

## `links` 对象

使用 `links` 对象以编程方式实现分页导航：

| 字段 | 值 |
|-------|-------|
| `self` | 当前页查询参数 |
| `first` | 首页 |
| `prev` | 上一页（若在第一页则为 null） |
| `next` | 下一页（若在最后一页则为 null） |
| `last` | 末页 |

## `data` 对象

行对象数组。所有值均为**字符串**，无论其逻辑类型如何。

## 响应状态码

| 状态码 | 含义 |
|------|---------|
| 200 | 成功 — GET 请求成功 |
| 304 | 未修改 — 缓存响应 |
| 400 | 错误请求 — URL 格式错误或参数无效 |
| 403 | 禁止访问 — API 密钥无效（不适用；无需密钥） |
| 404 | 未找到 — 端点不存在 |
| 405 | 方法不允许 — 非 GET 请求 |
| 429 | 请求过多 — 触发速率限制 |
| 500 | 服务器内部错误 |

## 错误对象

发生错误时，响应将包含错误对象而非 `data`：

```json
{
  "error": "无效查询参数",
  "message": "查询参数 'sorts' 的值 '[-record_date]' 无效。详情请参阅文档。"
}
```

```python
resp = requests.get(url, params=params)
result = resp.json()

if "error" in result:
    print(f"API 错误: {result['error']}")
    print(f"消息: {result['message']}")
elif resp.status_code != 200:
    print(f"HTTP {resp.status_code}: {resp.text}")
else:
    data = result["data"]
```

## 常见错误原因

- `fields=` 参数中包含无效字段名
- 无效的筛选运算符（应使用 `eq`, `gte`, `lte`, `gt`, `lt`, `in`）
- 日期格式错误（必须为 `YYYY-MM-DD`）
- 使用 `/v1/` URL 访问 v2 端点
- 排序字段在端点中不可用

## 解析响应

```python
import requests
import pandas as pd

def api_to_dataframe(endpoint, params=None):
    """获取 API 数据并返回类型化 DataFrame"""
    base = "https://api.fiscaldata.treasury.gov/services/api/fiscal_service"
    resp = requests.get(f"{base}{endpoint}", params=params)
    resp.raise_for_status()
    result = resp.json()
    
    df = pd.DataFrame(result["data"])
    meta = result["meta"]
    
    # 使用元数据应用类型转换
    for col, dtype in meta["dataTypes"].items():
        if col not in df.columns:
            continue
        if dtype in ("NUMBER", "CURRENCY", "PERCENTAGE"):
            df[col] = pd.to_numeric(df[col].replace("null", None), errors="coerce")
        elif dtype == "DATE":
            df[col] = pd.to_datetime(df[col].replace("null", None), errors="coerce")
        elif dtype == "INTEGER":
            df[col] = pd.to_numeric(df[col].replace("null", None), errors="coerce").astype("Int64")
    
    return df, meta

# 使用示例
df, meta = api_to_dataframe(
    "/v2/accounting/od/debt_to_penny",
    params={"sort": "-record_date", "page[size]": 30}
)
print(f"可用总记录数: {meta['total-count']}")
print(df[["record_date", "tot_pub_debt_out_amt"]].head())
```

## CSV 格式响应

当指定 `format=csv` 时，响应体为纯 CSV 文本（非 JSON）：

```python
import io

resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny",
    params={"format": "csv", "sort": "-record_date", "page[size]": 100}
)
df = pd.read_csv(io.StringIO(resp.text))
```

## XML 格式响应

当指定 `format=xml` 时，响应体为 XML：

```python
import xml.etree.ElementTree as ET

resp = requests.get(
    "https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny",
    params={"format": "xml", "page[size]": 10}
)
root = ET.fromstring(resp.text)
```
