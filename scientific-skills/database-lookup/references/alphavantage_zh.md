# Alpha Vantage API 参考文档

## 概述
Alpha Vantage 提供免费 API，涵盖实时和历史股票价格、外汇汇率、加密货币数据、技术指标和基本面数据（财报、资产负债表、利润表）。覆盖全球股票、ETF、共同基金和大宗商品。

## 基础 URL
```
https://www.alphavantage.co/query
```

所有请求均使用单一端点，通过 `function` 参数选择数据类型。

## 认证
- **API 密钥：必需。** 在 https://www.alphavantage.co/support/#api-key 获取免费密钥
- 通过查询参数传递：`&apikey=您的密钥`

## 速率限制
- **免费层级：** 每日 25 次请求，每分钟 5 次调用（截至 2024 年末；此前为 5次/分钟 + 500次/日）。
- 提供**高级层级**支持更高限制（30/75/150+ 次调用/分钟）。
- 超出限制时返回友好 JSON 提示，而非错误代码。

---

## 核心端点（按 `function` 参数分类）

### 1. 股票时间序列

#### 日内数据
```
GET /query?function=TIME_SERIES_INTRADAY&symbol={symbol}&interval={interval}&apikey={key}
```
| 参数 | 必填 | 取值 |
|-----------|----------|--------|
| `symbol` | 是 | 股票代码（如 `AAPL`, `MSFT`） |
| `interval` | 是 | `1min`, `5min`, `15min`, `30min`, `60min` |
| `outputsize` | 否 | `compact`（最近100个点，默认）或 `full`（完整历史） |
| `adjusted` | 否 | `true`（默认）或 `false` |
| `datatype` | 否 | `json`（默认）或 `csv` |

**示例：**
```
https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol=AAPL&interval=5min&apikey=YOUR_KEY
```

#### 日线数据
```
GET /query?function=TIME_SERIES_DAILY&symbol=AAPL&apikey=YOUR_KEY
```

#### 日线数据（复权处理）
```
GET /query?function=TIME_SERIES_DAILY_ADJUSTED&symbol=AAPL&outputsize=full&apikey=YOUR_KEY
```

#### 周线/月线数据
```
GET /query?function=TIME_SERIES_WEEKLY_ADJUSTED&symbol=AAPL&apikey=YOUR_KEY
GET /query?function=TIME_SERIES_MONTHLY_ADJUSTED&symbol=AAPL&apikey=YOUR_KEY
```

**响应示例（日线）：**
```json
{
  "Meta Data": {
    "1. Information": "每日价格（开盘价、最高价、最低价、收盘价）和成交量",
    "2. Symbol": "AAPL",
    "3. Last Refreshed": "2024-11-01",
    "4. Output Size": "Compact",
    "5. Time Zone": "US/Eastern"
  },
  "Time Series (Daily)": {
    "2024-11-01": {
      "1. open": "228.6900",
      "2. high": "229.8600",
      "3. low": "225.8200",
      "4. close": "228.5200",
      "5. volume": "50423432"
    },
    "2024-10-31": {
      "1. open": "229.3400",
      "2. high": "230.2000",
      "3. low": "226.3700",
      "4. close": "227.5500",
      "5. volume": "51235678"
    }
  }
}
```

---

### 2. 股票搜索（代码查询）
```
GET /query?function=SYMBOL_SEARCH&keywords={query}&apikey={key}
```

**示例：**
```
https://www.alphavantage.co/query?function=SYMBOL_SEARCH&keywords=microsoft&apikey=YOUR_KEY
```

**响应：**
```json
{
  "bestMatches": [
    {
      "1. symbol": "MSFT",
      "2. name": "Microsoft Corporation",
      "3. type": "Equity",
      "4. region": "United States",
      "5. marketOpen": "09:30",
      "6. marketClose": "16:00",
      "7. timezone": "UTC-04",
      "8. currency": "USD",
      "9. matchScore": "1.0000"
    }
  ]
}
```

---

### 3. 全局报价（实时价格）
```
GET /query?function=GLOBAL_QUOTE&symbol=AAPL&apikey=YOUR_KEY
```

返回单个股票代码的最新价格、成交量、涨跌幅和涨跌百分比。

---

### 4. 外汇（FX）汇率

#### 实时汇率
```
GET /query?function=CURRENCY_EXCHANGE_RATE&from_currency=USD&to_currency=EUR&apikey=YOUR_KEY
```

#### 外汇时间序列
```
GET /query?function=FX_DAILY&from_symbol=EUR&to_symbol=USD&apikey=YOUR_KEY
GET /query?function=FX_WEEKLY&from_symbol=EUR&to_symbol=USD&apikey=YOUR_KEY
GET /query?function=FX_MONTHLY&from_symbol=EUR&to_symbol=USD&apikey=YOUR_KEY
GET /query?function=FX_INTRADAY&from_symbol=EUR&to_symbol=USD&interval=5min&apikey=YOUR_KEY
```

---

### 5. 加密货币

#### 实时汇率
```
GET /query?function=CURRENCY_EXCHANGE_RATE&from_currency=BTC&to_currency=USD&apikey=YOUR_KEY
```

#### 加密货币时间序列
```
GET /query?function=DIGITAL_CURRENCY_DAILY&symbol=BTC&market=USD&apikey=YOUR_KEY
GET /query?function=DIGITAL_CURRENCY_WEEKLY&symbol=BTC&market=USD&apikey=YOUR_KEY
GET /query?function=DIGITAL_CURRENCY_MONTHLY&symbol=BTC&market=USD&apikey=YOUR_KEY
```

---

### 6. 技术指标
```
GET /query?function={INDICATOR}&symbol={symbol}&interval={interval}&time_period={n}&series_type={type}&apikey={key}
```

| 参数 | 必填 | 说明 |
|-----------|----------|-------------|
| `function` | 是 | 指标名称（见下方列表） |
| `symbol` | 是 | 股票代码 |
| `interval` | 是 | `1min`, `5min`, `15min`, `30min`, `60min`, `daily`, `weekly`, `monthly` |
| `time_period` | 是* | 计算使用的数据点数（如 RSI 取 14） |
| `series_type` | 是* | `close`, `open`, `high`, `low` |

*大多数指标必需；部分指标（如 MACD, BBANDS）有额外参数。

**常用指标函数：**
`SMA`, `EMA`, `WMA`, `DEMA`, `TEMA`, `VWAP`, `RSI`, `MACD`, `STOCH`, `ADX`, `CCI`, `AROON`, `BBANDS`, `AD`, `OBV`, `ATR`, `WILLR`, `MOM`

**示例——RSI（14日）：**
```
https://www.alphavantage.co/query?function=RSI&symbol=AAPL&interval=daily&time_period=14&series_type=close&apikey=YOUR_KEY
```

**示例——MACD：**
```
https://www.alphavantage.co/query?function=MACD&symbol=AAPL&interval=daily&series_type=close&apikey=YOUR_KEY
```

---

### 7. 基本面数据

#### 公司概览
```
GET /query?function=OVERVIEW&symbol=AAPL&apikey=YOUR_KEY
```
返回：市值、市盈率、每股收益、股息收益率、52周高/低价、行业分类、公司描述等约60个字段。

#### 利润表
```
GET /query?function=INCOME_STATEMENT&symbol=AAPL&apikey=YOUR_KEY
```

#### 资产负债表
```
GET /query?function=BALANCE_SHEET&symbol=AAPL&apikey=YOUR_KEY
```

#### 现金流量表
```
GET /query?function=CASH_FLOW&symbol=AAPL&apikey=YOUR_KEY
```

#### 财报数据
```
GET /query?function=EARNINGS&symbol=AAPL&apikey=YOUR_KEY
```

返回年度和季度收益数据（每股收益、预期每股收益、预期差）。

---

### 8. 大宗商品与经济指标
```
GET /query?function=WTI&interval=monthly&apikey=YOUR_KEY
GET /query?function=BRENT&interval=monthly&apikey=YOUR_KEY
GET /query?function=NATURAL_GAS&interval=monthly&apikey=YOUR_KEY
GET /query?function=COPPER&interval=monthly&apikey=YOUR_KEY
GET /query?function=ALUMINUM&interval=monthly&apikey=YOUR_KEY
GET /query?function=WHEAT&interval=monthly&apikey=YOUR_KEY
GET /query?function=CORN&interval=monthly&apikey=YOUR_KEY
GET /query?function=COTTON&interval=monthly&apikey=YOUR_KEY
GET /query?function=SUGAR&interval=monthly&apikey=YOUR_KEY
GET /query?function=COFFEE&interval=monthly&apikey=YOUR_KEY
```

经济指标：
```
GET /query?function=REAL_GDP&interval=quarterly&apikey=YOUR_KEY
GET /query?function=CPI&interval=monthly&apikey=YOUR_KEY
GET /query?function=INFLATION&apikey=YOUR_KEY
GET /query?function=RETAIL_SALES&apikey=YOUR_KEY
GET /query?function=UNEMPLOYMENT&apikey=YOUR_KEY
GET /query?function=FEDERAL_FUNDS_RATE&interval=monthly&apikey=YOUR_KEY
GET /query?function=TREASURY_YIELD&interval=monthly&maturity=10year&apikey=YOUR_KEY
```

---

## 注意事项
- 所有返回值在 JSON 中均为字符串格式
- JSON 键名使用数字前缀（如 `"1. open"`, `"2. high"`）
- 时间序列数据按日期/时间戳字符串索引，非数组格式
- 触发速率限制时返回：`{"Note": "感谢使用 Alpha Vantage！..."}`
- 使用 `outputsize=full` 时，日线数据可回溯 20 年以上
- `datatype=csv` 选项可为任意端点返回简化 CSV 格式
- 免费层级限制严格（25次/日），生产环境建议使用高级密钥
