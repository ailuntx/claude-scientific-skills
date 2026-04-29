# 美国劳工统计局（BLS）公共数据API

## 基础URL

```
https://api.bls.gov/publicAPI/v2
```

版本1（无密钥）：`https://api.bls.gov/publicAPI/v1`

## 认证

**API密钥可选但强烈推荐。** 在 https://data.bls.gov/registrationEngine/ 注册

- **V1（无密钥）：** 每日限25次请求，10年日期范围，每次查询25个序列
- **V2（带密钥）：** 每日500次请求，20年日期范围，每次查询50个序列，额外提供目录数据与计算功能

## 核心端点

### 1. 获取序列数据（POST -- 主要方法）
```
POST /timeseries/data/
```
内容类型：`application/json`

**请求体：**
```json
{
  "seriesid": ["CUUR0000SA0", "LNS14000000"],
  "startyear": "2020",
  "endyear": "2024",
  "registrationkey": "YOUR_KEY",
  "catalog": true,
  "calculations": true,
  "annualaverage": true,
  "aspects": true
}
```

| 字段             | 必需     | V1  | V2  | 说明                                               |
|------------------|----------|-----|-----|---------------------------------------------------|
| seriesid         | 是       | 是  | 是  | 序列ID数组（V1上限25个/V2上限50个）               |
| startyear        | 是       | 是  | 是  | 4位数起始年份                                     |
| endyear          | 是       | 是  | 是  | 4位数结束年份                                     |
| registrationkey  | 否       | 否  | 是  | API密钥（V2功能必需）                             |
| catalog          | 否       | 否  | 是  | `true`包含序列元数据                              |
| calculations     | 否       | 否  | 是  | `true`包含净变化/百分比变化                       |
| annualaverage    | 否       | 否  | 是  | `true`包含年度平均值                              |
| aspects          | 否       | 否  | 是  | `true`包含脚注与附加属性                          |

### 2. 获取单序列数据（GET -- 便捷方式）
```
GET /timeseries/data/{seriesID}
```
示例：
```
https://api.bls.gov/publicAPI/v2/timeseries/data/CUUR0000SA0?registrationkey=YOUR_KEY&startyear=2022&endyear=2024
```

### 3. 最新数据（GET -- 无日期范围）
```
GET /timeseries/data/{seriesID}
```
不指定起止年份时，返回最近3年数据

示例：
```
https://api.bls.gov/publicAPI/v2/timeseries/data/LNS14000000?registrationkey=YOUR_KEY
```

## 常用序列ID

### 消费者价格指数（CPI）
| 序列ID         | 说明                                               |
|----------------|---------------------------------------------------|
| CUUR0000SA0    | CPI-U所有项目，美国城市平均，未季调                |
| CUSR0000SA0    | CPI-U所有项目，美国城市平均，季调                  |
| CUUR0000SAF1   | CPI-U食品，美国城市平均                           |
| CUUR0000SETB01 | CPI-U汽油（所有类型）                             |
| CUUR0000SAH1   | CPI-U居住成本                                     |
| CUUR0000SAM    | CPI-U医疗护理                                     |

CPI序列ID结构：`CU` + `U/S`（未调整/调整） + `R/S`（修订版） + 区域代码 + 项目代码

### 就业/失业（当前人口调查）
| 序列ID         | 说明                                               |
|----------------|---------------------------------------------------|
| LNS14000000    | 失业率（季调）                                    |
| LNS11000000    | 民用劳动力总量                                    |
| LNS12000000    | 就业总量                                          |
| LNS13000000    | 失业总量                                          |
| LNS14000006    | 黑人或非裔美国人失业率                            |
| LNS14000009    | 西班牙裔或拉丁裔失业率                            |

### 就业（当前就业统计/非农就业）
| 序列ID           | 说明                                               |
|------------------|---------------------------------------------------|
| CES0000000001    | 非农就业总量（季调）                              |
| CES0500000003    | 私营部门平均时薪                                  |
| CES0500000002    | 私营部门平均周工时                                |

### 生产者价格指数（PPI）
| 序列ID       | 说明                                               |
|-------------|---------------------------------------------------|
| WPSFD4       | PPI最终需求                                       |
| WPUFD49104   | PPI扣除食品与能源的最终需求                        |

### 就业成本指数（ECI）
| 序列ID             | 说明                                               |
|-------------------|---------------------------------------------------|
| CIU1010000000000A | ECI总薪酬，全体平民                                |

### 职业就业与工资统计（OEWS）
| 序列ID模式           | 说明                                               |
|---------------------|---------------------------------------------------|
| OEUM003342000000011-0000 | 示例：特定职业/区域组合                      |

OEWS序列ID结构复杂，请使用BLS序列ID查找工具：https://data.bls.gov/cgi-bin/srgate

## 序列ID结构

BLS序列ID编码了调查类型、季节调整、区域、行业和项目信息。核心调查前缀：

| 前缀 | 调查类型                                         |
|------|------------------------------------------------|
| CU   | 消费者价格指数                                  |
| LN   | 当前人口调查（劳动力）                          |
| CE   | 当前就业统计                                    |
| WP   | 生产者价格指数                                  |
| EI   | 就业成本指数/国家薪酬                           |
| OE   | 职业就业与工资统计                              |
| LA   | 地方区域失业统计                                |
| SM   | 州及都会区就业（CES）                           |
| JT   | 职位空缺与劳动力流动调查（JOLTS）               |

## 响应格式

### 标准响应
```json
{
  "status": "REQUEST_SUCCEEDED",
  "responseTime": 85,
  "message": [],
  "Results": {
    "series": [
      {
        "seriesID": "CUUR0000SA0",
        "catalog": {
          "series_title": "All items in U.S. city average, all urban consumers, not seasonally adjusted",
          "series_id": "CUUR0000SA0",
          "seasonality": "Not Seasonally Adjusted",
          "survey_name": "Consumer Price Index - All Urban Consumers",
          "survey_abbreviation": "CU",
          "measure_data_type": "All items",
          "area": "U.S. city average",
          "item": "All items"
        },
        "data": [
          {
            "year": "2024",
            "period": "M01",
            "periodName": "January",
            "latest": "true",
            "value": "308.417",
            "footnotes": [{}],
            "calculations": {
              "net_changes": {
                "1": "0.5",
                "3": "1.2",
                "6": "2.1",
                "12": "3.1"
              },
              "pct_changes": {
                "1": "0.2",
                "3": "0.4",
                "6": "0.7",
                "12": "3.1"
              }
            }
          },
          {
            "year": "2023",
            "period": "M12",
            "periodName": "December",
            "value": "306.746",
            "footnotes": [{}]
          }
        ]
      }
    ]
  }
}
```

### 数据对象关键字段
- `year`: 4位数年份字符串
- `period`: `M01`-`M12`（月度）, `Q01`-`Q05`（季度）, `A01`（年度）, `S01`-`S03`（半年度）
- `periodName`: 人类可读周期名称
- `value`: 字符串（计算时需转为浮点数）
- `latest`: 仅最新数据点标记为`"true"`
- `calculations`: 仅当请求中设置`calculations: true`时出现（V2）。包含1/3/6/12个月跨度的`net_changes`（净变化）和`pct_changes`（百分比变化）
- `footnotes`: 脚注对象数组

### 错误响应
```json
{
  "status": "REQUEST_NOT_PROCESSED",
  "responseTime": 10,
  "message": ["No data available for the given series and date range."],
  "Results": {
    "series": []
  }
}
```

## 速率限制

| 特性               | V1（无密钥）     | V2（带密钥）      |
|--------------------|-----------------|------------------|
| 每日查询限制       | 25次请求        | 500次请求        |
| 每次查询序列数     | 25              | 50               |
| 每次查询年份跨度   | 10年            | 20年             |
| 目录数据           | 否              | 是               |
| 计算功能           | 否              | 是               |
| 年度平均值         | 否              | 是               |
| 净变化/百分比变化  | 否              | 是               |

## 注意事项

- BLS强烈建议使用POST请求获取数据，GET端点为便捷封装
- 周期`M13`代表年度平均值（仅当`annualaverage: true`时出现）
- 所有`value`字段均为字符串。缺失数据通常被省略（该观测点不会出现）
- 计算CPI百分比变化（通胀率）时，可从原始指数值计算，或使用V2的`calculations`功能获取预计算的12个月百分比变化
- BLS官网提供序列ID构造工具：https://data.bls.gov/cgi-bin/srgate
- 批量数据下载地址：https://download.bls.gov/pub/time.series/ 按调查前缀分类
