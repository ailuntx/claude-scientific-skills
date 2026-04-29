# 欧洲央行统计数据仓库（SDW）REST API参考

## 概述
ECB SDW API提供对欧洲央行统计数据的访问：汇率、货币总量、利率、国际收支、银行业统计数据等。该API遵循SDMX（统计数据和元数据交换）RESTful网络服务标准。

## 基础URL
```
https://data-api.ecb.europa.eu/service
```

注意：旧版URL `https://sdw-wsrest.ecb.europa.eu/service` 仍可使用，但上述URL为当前端点。

## 认证
**无需API密钥。** 该API完全开放公开。

## 速率限制
- 未公布正式速率限制
- ECB要求用户合理使用：避免过度并行请求
- 批量下载时请使用压缩响应（`Accept-Encoding: gzip`）

## 常用请求头
| 请求头 | 值 | 说明 |
|--------|-------|-------------|
| `Accept` | `application/vnd.sdmx.data+json;version=2.0.0` | JSON格式（推荐） |
| `Accept` | `application/vnd.sdmx.data+csv` | CSV格式 |
| `Accept` | `application/vnd.sdmx.data+xml` | SDMX-ML XML（默认） |
| `Accept-Encoding` | `gzip` | 压缩响应 |

---

## 核心端点

### 1. 获取数据（时间序列）

```
GET /data/{flowRef}/{key}?{parameters}
```

| 组件 | 说明 |
|-----------|-------------|
| `flowRef` | 数据流ID（如汇率`EXR`，资产负债表项目`BSI`） |
| `key` | 点分隔的维度值。使用`+`表示OR，`.`跳过维度（通配符） |

**查询参数：**
| 参数 | 必填 | 说明 |
|-----------|----------|-------------|
| `startPeriod` | 否 | 起始日期：`YYYY`、`YYYY-MM`或`YYYY-MM-DD` |
| `endPeriod` | 否 | 结束日期：同上格式 |
| `updatedAfter` | 否 | ISO 8601时间戳；仅返回此时间后更新的数据 |
| `detail` | 否 | `full`（默认）、`dataonly`、`serieskeysonly`、`nodata` |
| `firstNObservations` | 否 | 每序列仅返回前N个观测值 |
| `lastNObservations` | 否 | 每序列仅返回后N个观测值 |
| `dimensionAtObservation` | 否 | 通常为`TIME_PERIOD`（默认） |

**汇率键结构（EXR数据流）：**
`{频率}.{货币}.{基准货币}.{汇率类型}.{汇率后缀}`

| 位置 | 维度 | 常用值 |
|----------|-----------|---------------|
| 1 | 频率 | `D`（日）、`M`（月）、`A`（年） |
| 2 | 货币 | `USD`、`GBP`、`JPY`、`CHF`、`CNY`等 |
| 3 | 基准货币 | `EUR`（通常） |
| 4 | 汇率类型 | `SP00`（即期）、`EN00`（平均） |
| 5 | 汇率后缀 | `A`（平均值）、`E`（期末值） |

**示例——2024年美元/欧元日即期汇率：**
```
GET https://data-api.ecb.europa.eu/service/data/EXR/D.USD.EUR.SP00.A?startPeriod=2024-01-01&endPeriod=2024-12-31
Accept: application/vnd.sdmx.data+json;version=2.0.0
```

**示例——英镑和日元兑欧元月汇率，最近12个观测值：**
```
GET https://data-api.ecb.europa.eu/service/data/EXR/M.GBP+JPY.EUR.SP00.A?lastNObservations=12
Accept: application/vnd.sdmx.data+json;version=2.0.0
```

**示例——特定日期所有日汇率（通配符）：**
```
GET https://data-api.ecb.europa.eu/service/data/EXR/D..EUR.SP00.A?startPeriod=2024-06-01&endPeriod=2024-06-01
Accept: application/vnd.sdmx.data+json;version=2.0.0
```

**JSON响应结构（SDMX-JSON v2.0）：**
```json
{
  "meta": { "schema": "...", "id": "...", "prepared": "2024-11-01T12:00:00Z" },
  "data": {
    "dataSets": [
      {
        "action": "Information",
        "series": {
          "0": {
            "attributes": [0, 0, ...],
            "observations": {
              "0": [1.0856],
              "1": [1.0791],
              "2": [1.0834]
            }
          }
        }
      }
    ],
    "structures": [
      {
        "dimensions": {
          "series": [...],
          "observation": [
            {
              "id": "TIME_PERIOD",
              "values": [
                {"id": "2024-01-02", "name": "2024-01-02"},
                {"id": "2024-01-03", "name": "2024-01-03"}
              ]
            }
          ]
        }
      }
    ]
  }
}
```

注意：观测值为索引数组。需将观测索引与`structures.dimensions.observation`中的`TIME_PERIOD`值匹配。

**CSV响应**（更易解析）：
```
Accept: application/vnd.sdmx.data+csv
```
返回标准CSV，列名：`DATAFLOW`、`FREQ`、`CURRENCY`、`CURRENCY_DENOM`、`EXR_TYPE`、`EXR_SUFFIX`、`TIME_PERIOD`、`OBS_VALUE`等。

---

### 2. 获取数据流定义（可用数据集）

```
GET /dataflow/{agencyID}/{resourceID}/{version}
```

**示例——列出所有ECB数据流：**
```
GET https://data-api.ecb.europa.eu/service/dataflow/ECB
Accept: application/vnd.sdmx.structure+json;version=2.0.0
```

**示例——获取EXR数据流定义：**
```
GET https://data-api.ecb.europa.eu/service/dataflow/ECB/EXR
Accept: application/vnd.sdmx.structure+json;version=2.0.0
```

---

### 3. 获取数据结构定义（维度与代码）

```
GET /datastructure/{agencyID}/{resourceID}/{version}?references=children
```

**示例：**
```
GET https://data-api.ecb.europa.eu/service/datastructure/ECB/ECB_EXR1?references=children
Accept: application/vnd.sdmx.structure+json;version=2.0.0
```

此调用返回所有维度、代码列表及允许值——对构建有效键至关重要。

---

## 常用数据流ID

| 数据流 | 说明 |
|----------|-------------|
| `EXR` | 汇率 |
| `BSI` | 资产负债表项目（货币金融机构） |
| `MIR` | 货币金融机构利率 |
| `ILM` | 内部流动性管理 |
| `SEC` | 证券发行统计 |
| `BOP` | 国际收支 |
| `STP` | 结构性金融指标 |
| `CBD` | 综合银行业数据 |
| `ICP` | 消费者价格指数（HICP） |
| `FM` | 金融市场数据 |
| `YC` | 收益率曲线数据 |

## 注意事项
- SDMX-JSON格式较冗长。简化解析请使用`Accept: application/vnd.sdmx.data+csv`
- 维度未知时可留空（如`D..EUR.SP00.A`）以获取该维度所有值
- 使用`+`可请求单个维度的多个值（如`USD+GBP`）
- `detail=dataonly`参数可忽略属性并减小响应体积
- 历史数据可用性因数据流而异；汇率数据可追溯至1999年（欧元启用）
