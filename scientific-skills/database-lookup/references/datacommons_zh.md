# Google Data Commons API

## 基础 URL

```
https://api.datacommons.org
```

## 身份验证

**需提供 API 密钥。** 通过 Google Cloud Console 获取（需启用 Data Commons API）。

通过查询参数传递：`&key=YOUR_KEY`

或通过请求头传递：`X-API-Key: YOUR_KEY`

注意：多数端点支持轻量级无密钥访问，但建议使用密钥确保稳定调用。

## 核心端点

### 1. 获取统计值（单次观测）
```
GET /v2/observation
```
| 参数          | 必填 | 说明                                                |
|---------------|------|----------------------------------------------------|
| key           | 是   | API 密钥                                           |
| entity.dcids  | 是   | 地点 DCID（如 `country/USA`, `geoId/06`）         |
| variable.dcids| 是   | 统计变量 DCID                                      |
| date          | 否   | 指定日期或 `LATEST`（最新值）                     |
| select        | 否   | 返回字段：`entity`, `variable`, `date`, `value`   |

示例：
```
https://api.datacommons.org/v2/observation?key=YOUR_KEY&entity.dcids=country/USA&variable.dcids=Count_Person&date=LATEST&select=entity&select=variable&select=date&select=value
```

### 2. 获取统计时间序列
```
GET /v2/observation
```
使用相同端点，省略 `date` 参数（或设为 `date=''`）可获取完整时间序列。

示例（美国人口时间序列）：
```
https://api.datacommons.org/v2/observation?key=YOUR_KEY&entity.dcids=country/USA&variable.dcids=Count_Person&select=entity&select=variable&select=date&select=value
```

### 3. 节点信息（实体属性值）
```
GET /v2/node
```
| 参数      | 必填 | 说明                                        |
|-----------|------|--------------------------------------------|
| key       | 是   | API 密钥                                    |
| nodes     | 是   | 节点 DCID                                  |
| property  | 是   | 属性表达式：`->prop`（出向）, `<-prop`（入向）|

示例（获取加利福尼亚州属性）：
```
https://api.datacommons.org/v2/node?key=YOUR_KEY&nodes=geoId/06&property=->*
```

示例（获取地点名称）：
```
https://api.datacommons.org/v2/node?key=YOUR_KEY&nodes=geoId/06&property=->name
```

### 4. SPARQL 查询
```
POST /v2/sparql
```
Content-Type: `application/json`

请求体：
```json
{
  "query": "SELECT ?name WHERE { ?state typeOf State . ?state name ?name . ?state containedInPlace country/USA }"
}
```

通过查询参数或请求头传递 API 密钥。

示例（curl）：
```
curl -X POST 'https://api.datacommons.org/v2/sparql?key=YOUR_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"query": "SELECT ?name WHERE { ?place typeOf Country . ?place name ?name } LIMIT 10"}'
```

### 5. 实体解析（将名称/坐标映射为 DCID）
```
GET /v2/resolve
```
| 参数      | 必填 | 说明                                    |
|-----------|------|----------------------------------------|
| key       | 是   | API 密钥                                |
| nodes     | 是   | 待解析的实体标识符                     |
| property  | 是   | `<-description`（名称查询）或基于坐标的解析 |

示例（名称解析）：
```
https://api.datacommons.org/v2/resolve?key=YOUR_KEY&nodes=California&property=<-description->dcid
```

### 6. 搜索统计变量
```
GET /v2/variable/search
```
| 参数  | 必填 | 说明            |
|-------|------|----------------|
| key   | 是   | API 密钥        |
| query | 是   | 搜索关键词      |

示例：
```
https://api.datacommons.org/v2/variable/search?key=YOUR_KEY&query=unemployment+rate
```

## 常用 DCID

### 地点
| DCID              | 说明               |
|-------------------|-------------------|
| country/USA       | 美国              |
| country/GBR       | 英国              |
| country/CHN       | 中国              |
| geoId/06          | 加利福尼亚州      |
| geoId/0667000     | 旧金山市          |
| geoId/06085       | 圣克拉拉县        |

### 统计变量
| DCID                                    | 说明                     |
|-----------------------------------------|--------------------------|
| Count_Person                            | 总人口                   |
| Count_Person_Employed                   | 就业人口                 |
| UnemploymentRate_Person                 | 失业率                   |
| Median_Income_Person                    | 收入中位数               |
| Amount_EconomicActivity_GrossDomesticProduction_Nominal | 名义 GDP |
| Mean_ConsumerPriceIndex                 | 消费者价格指数           |
| Count_Death                             | 死亡人数                 |
| Count_Person_BelowPovertyLevelInThePast12Months | 贫困人口数量 |
| Median_Age_Person                       | 年龄中位数               |

## 响应格式

### 观测值响应
```json
{
  "byVariable": {
    "Count_Person": {
      "byEntity": {
        "country/USA": {
          "orderedFacets": [
            {
              "facetId": "2176550201",
              "observations": [
                {
                  "date": "2020",
                  "value": 331449281
                },
                {
                  "date": "2021",
                  "value": 331893745
                }
              ]
            }
          ]
        }
      }
    }
  },
  "facets": {
    "2176550201": {
      "importName": "CensusACS5YearSurvey",
      "provenanceUrl": "https://www.census.gov/",
      "measurementMethod": "CensusACS5yrSurvey"
    }
  }
}
```

### 节点响应
```json
{
  "data": {
    "geoId/06": {
      "arcs": {
        "name": {
          "nodes": [
            {
              "value": "California"
            }
          ]
        }
      }
    }
  }
}
```

### SPARQL 响应
```json
{
  "header": ["?name"],
  "rows": [
    { "cells": [{ "value": "Alabama" }] },
    { "cells": [{ "value": "Alaska" }] }
  ]
}
```

### 变量搜索响应
```json
{
  "variables": [
    {
      "dcid": "UnemploymentRate_Person",
      "displayName": "Unemployment Rate"
    }
  ]
}
```

## 速率限制

- 无 API 密钥：限制严格（约每分钟数次请求，可能被拦截）
- 使用 API 密钥：未正式公布限制，常规使用通常充足
- 建议客户端限流（推荐 1-2 次请求/秒）
- 大规模分析请通过 Data Commons 数据下载获取批量数据

## 注意事项

- V2 API（路径以 `/v2/` 开头）为当前推荐版本
- 旧版 V1 端点（`/v1/bulk/observations/series`, `/stat/value` 等）仍可用但已弃用
- DCID = Data Commons 唯一标识符，每个实体/统计变量/概念均有唯一 DCID
- 知识图谱整合了美国人口普查局、世界银行、CDC、BLS、FBI 等机构数据
