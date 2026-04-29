# 世界银行开放数据 API

## 基础 URL

```
https://api.worldbank.org/v2
```

## 认证

**无需 API 密钥。** 该 API 完全开放。

## 关键端点

### 1. 获取国家指标数据
```
GET /country/{country_code}/indicator/{indicator_code}
```
| 参数       | 是否必需 | 描述                                        |
|------------|----------|---------------------------------------------|
| format     | 否       | `json`, `xml` (默认), `jsonP`              |
| date       | 否       | 年份范围: `2010:2023`, 单年: `2020`        |
| page       | 否       | 页码 (默认 1)                               |
| per_page   | 否       | 每页结果数 (默认 50, 最大 32500)            |
| MRV        | 否       | 最近值: 获取最近若干数据点                  |
| gapfill    | 否       | `Y` 表示用最近值填充空缺                   |
| frequency  | 否       | `M` (月度), `Q` (季度), `Y` (年度)         |
| source     | 否       | 数据源编号                                  |

示例 (美国 GDP, 2015-2023):
```
https://api.worldbank.org/v2/country/US/indicator/NY.GDP.MKTP.CD?format=json&date=2015:2023
```

示例 (最近 5 个值):
```
https://api.worldbank.org/v2/country/US/indicator/NY.GDP.MKTP.CD?format=json&MRV=5
```

### 2. 获取多国指标数据
```
GET /country/{code1};{code2};{code3}/indicator/{indicator_code}
```
示例:
```
https://api.worldbank.org/v2/country/US;GB;CN;IN/indicator/SP.POP.TOTL?format=json&date=2020:2023
```

### 3. 获取所有国家指标数据
```
GET /country/all/indicator/{indicator_code}
```
示例:
```
https://api.worldbank.org/v2/country/all/indicator/SI.POV.DDAY?format=json&date=2020&per_page=300
```

### 4. 按区域/收入组获取指标数据
```
GET /country/{aggregate_code}/indicator/{indicator_code}
```
聚合代码: `EAS` (东亚), `ECS` (欧洲与中亚), `LIC` (低收入), `HIC` (高收入), `WLD` (全球) 等。

示例:
```
https://api.worldbank.org/v2/country/WLD/indicator/NY.GDP.MKTP.CD?format=json&date=2020:2023
```

### 5. 列出所有国家
```
GET /country
```
示例:
```
https://api.worldbank.org/v2/country?format=json&per_page=300
```

### 6. 获取国家信息
```
GET /country/{country_code}
```
示例:
```
https://api.worldbank.org/v2/country/US?format=json
```

### 7. 列出所有指标
```
GET /indicator
```
示例:
```
https://api.worldbank.org/v2/indicator?format=json&per_page=100
```

### 8. 搜索指标
```
GET /indicator
```
直接在 URL 路径中使用查询字符串，或按主题/来源过滤。

按主题:
```
https://api.worldbank.org/v2/topic/3/indicator?format=json
```

按来源:
```
https://api.worldbank.org/v2/source/2/indicator?format=json&per_page=50
```

### 9. 列出主题
```
GET /topic
```
示例:
```
https://api.worldbank.org/v2/topic?format=json
```

### 10. 列出数据源
```
GET /source
```
示例:
```
https://api.worldbank.org/v2/source?format=json
```

## 常用指标代码

| 指标代码             | 描述                                     |
|----------------------|------------------------------------------|
| NY.GDP.MKTP.CD      | GDP (现价美元)                           |
| NY.GDP.MKTP.KD.ZG   | GDP 增长率 (年度百分比)                  |
| NY.GDP.PCAP.CD      | 人均 GDP (现价美元)                      |
| NY.GDP.PCAP.PP.CD   | 人均 GDP, PPP (现价国际元)               |
| SP.POP.TOTL         | 总人口                                   |
| SP.POP.GROW         | 人口增长率 (年度百分比)                  |
| SP.DYN.LE00.IN      | 出生时预期寿命 (年)                      |
| SP.DYN.TFRT.IN      | 生育率 (每位妇女生育数)                  |
| SL.UEM.TOTL.ZS      | 失业率 (占总劳动力百分比)                |
| FP.CPI.TOTL.ZG      | 通货膨胀率 (消费者价格年度百分比)        |
| SI.POV.DDAY         | 每日 2.15 美元贫困人口占比               |
| SI.POV.GINI         | 基尼指数                                 |
| BX.KLT.DINV.CD.WD   | 外国直接投资净流入 (国际收支平衡表, 美元)|
| NE.EXP.GNFS.ZS      | 商品与服务出口 (占 GDP 百分比)           |
| EN.ATM.CO2E.PC      | 人均二氧化碳排放量 (公吨)                |
| SE.ADT.LITR.ZS      | 成人识字率 (15 岁以上人群百分比)         |
| SH.XPD.CHEX.PC.CD   | 人均医疗支出 (美元)                      |
| IT.NET.USER.ZS      | 互联网用户比例 (占总人口百分比)          |

## 常用国家代码 (ISO 3166-1 alpha-2)

`US` (美国), `GB` (英国), `CN` (中国), `IN` (印度), `JP` (日本), `DE` (德国), `FR` (法国), `BR` (巴西), `ZA` (南非), `NG` (尼日利亚), `AU` (澳大利亚), `CA` (加拿大)

## 响应格式

**重要提示:** JSON 响应为**双元素数组**。首元素为分页元数据，次元素为数据数组。

### 指标观测值
```json
[
  {
    "page": 1,
    "pages": 1,
    "per_page": 50,
    "total": 9,
    "sourceid": "2",
    "lastupdated": "2024-03-28"
  },
  [
    {
      "indicator": {
        "id": "NY.GDP.MKTP.CD",
        "value": "GDP (current US$)"
      },
      "country": {
        "id": "US",
        "value": "United States"
      },
      "countryiso3code": "USA",
      "date": "2023",
      "value": 27360935000000,
      "unit": "",
      "obs_status": "",
      "decimal": 0
    },
    {
      "indicator": { "id": "NY.GDP.MKTP.CD", "value": "GDP (current US$)" },
      "country": { "id": "US", "value": "United States" },
      "countryiso3code": "USA",
      "date": "2022",
      "value": 25462700000000,
      "unit": "",
      "obs_status": "",
      "decimal": 0
    }
  ]
]
```

注: 当某年数据不可用时，`value` 为 `null`

### 国家信息
```json
[
  { "page": 1, "pages": 1, "per_page": 50, "total": 1 },
  [
    {
      "id": "US",
      "iso2Code": "US",
      "name": "United States",
      "region": { "id": "NAC", "iso2code": "XU", "value": "North America" },
      "adminregion": { "id": "", "iso2code": "", "value": "" },
      "incomeLevel": { "id": "HIC", "iso2code": "XD", "value": "High income" },
      "lendingType": { "id": "LNX", "iso2code": "XX", "value": "Not classified" },
      "capitalCity": "Washington D.C.",
      "longitude": "-77.032",
      "latitude": "38.8895"
    }
  ]
]
```

## 速率限制

- 未公布正式速率限制；API 开放且宽松
- 批量下载时使用 `per_page=32500` 以减少请求次数
- 保持合理频率：自动化脚本建议 1-2 次请求/秒
- 超大数据集请使用世界银行批量下载工具

## 注意事项

- 始终包含 `format=json` —— 默认返回 XML
- 结果默认按**降序**日期排列
- `null` 值常见于未发布数据的年份或覆盖稀疏的指标
- 分页：检查元数据中的 `pages` 字段；通过 `page=1`, `page=2` 等迭代
- URL 路径中的国家代码遵循 ISO 3166-1 alpha-2 (双字母)。响应同时包含 `countryiso3code`
