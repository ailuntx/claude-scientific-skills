# API 基础 — 美国财政部财政数据

## 概述

- RESTful API — 仅接受 HTTP GET 请求
- 默认返回 JSON 格式（也支持 CSV、XML）
- 无需 API 密钥、身份验证或注册
- 开放数据，可免费用于商业和非商业用途
- 当前版本：v1 和 v2（具体版本请查看各数据集页面）

## URL 结构

```
基础URL + 端点路径 + 参数

基础URL：https://api.fiscaldata.treasury.gov/services/api/fiscal_service
端点路径：/v2/accounting/od/debt_to_penny
参数：    ?fields=record_date,tot_pub_debt_out_amt&sort=-record_date&page[size]=5

完整URL：
https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny?fields=record_date,tot_pub_debt_out_amt&sort=-record_date&page[size]=5
```

- 端点路径组件使用小写字母 + 下划线
- 端点名称采用单数形式

## API 版本管理

- **v1**：早期数据集（DTS、MTS 及部分债务表格）
- **v2**：新增或更新的数据集（Debt to Penny、TROR、平均利率）
- 请访问 `fiscaldata.treasury.gov/datasets/` 确认具体数据集版本

## 数据类型

响应中所有字段值均为**字符串类型**（带引号），无论其逻辑类型如何。

| 逻辑类型   | 数据类型值    | 示例值              | 转换方法             |
|------------|---------------|---------------------|----------------------|
| 字符串     | `STRING`      | `"Canada-Dollar"`   | 无需转换             |
| 数值       | `NUMBER`      | `"36123456789012.34"` | `float(value)`       |
| 日期       | `DATE`        | `"2024-03-31"`      | `pd.to_datetime(value)` |
| 货币       | `CURRENCY`    | `"1234567.89"`      | `float(value)`       |
| 整数       | `INTEGER`     | `"42"`              | `int(value)`         |
| 百分比     | `PERCENTAGE`  | `"4.25"`            | `float(value)`       |

**空值**显示为字符串 `"null"`（非 Python 的 `None` 或 JSON 的 `null`）。

```python
# 安全处理空值的数值转换函数
def safe_float(val):
    return float(val) if val and val != "null" else None
```

## HTTP 方法

- **仅支持 GET 方法**
- POST、PUT、DELETE 请求将返回 HTTP 405

## 速率限制

- 触发限流时返回 HTTP 429
- 无固定速率限制文档说明；批量请求需实现退避重试机制

```python
import time
import requests

def get_with_retry(url, params, retries=3):
    for attempt in range(retries):
        resp = requests.get(url, params=params)
        if resp.status_code == 429:
            time.sleep(2 ** attempt)
            continue
        resp.raise_for_status()
        return resp.json()
    raise Exception("重试后仍被限流")
```

## 缓存机制

- 可能返回 HTTP 304（未修改）响应缓存
- 可安全缓存响应；多数数据集按日/月/季度更新

## 数据注册中心

[财政服务数据注册中心](https://fiscal.treasury.gov/data-registry/index.html)包含联邦政府数据的字段定义、权威数据源、数据类型及格式规范。
