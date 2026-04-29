# USGS API 参考（地震灾害 + 水文服务）

## A 部分：地震灾害项目

### 基础 URL
```
https://earthquake.usgs.gov/fdsnws/event/1
```

### 认证
**无需认证。** 完全公开，无需 API 密钥。

### 速率限制
- 无文档规定的用户级速率限制，但 USGS 要求用户限制自动化查询以避免服务过载。
- 返回超大结果集（>20,000 条事件）的请求将被拒绝。请使用分页或缩小查询范围。

### 关键端点

#### 1. 查询地震事件
```
GET /query
```
返回符合搜索条件的地震事件。这是主要端点。

**参数：**
| 参数名         | 类型   | 必填 | 默认值     | 描述 |
|--------------|--------|----------|------------|-------------|
| `format`     | 字符串 | 否       | `quakeml`  | `geojson`, `csv`, `quakeml`, `text`, `kml`。JSON 格式请用 `geojson` |
| `starttime`  | 字符串 | 否       | (当前时间 - 30天)| ISO8601 日期，如 `2024-01-01` |
| `endtime`    | 字符串 | 否       | (当前时间) | ISO8601 日期 |
| `minmagnitude`| 浮点数 | 否       | -          | 最小震级（如 `4.5`） |
| `maxmagnitude`| 浮点数 | 否       | -          | 最大震级 |
| `mindepth`   | 浮点数 | 否       | -          | 最小深度（公里） |
| `maxdepth`   | 浮点数 | 否       | -          | 最大深度（公里） |
| `latitude`   | 浮点数 | 否       | -          | 圆形搜索中心纬度（-90 至 90） |
| `longitude`  | 浮点数 | 否       | -          | 圆形搜索中心经度（-180 至 180） |
| `maxradiuskm`| 浮点数 | 否       | -          | 最大半径（公里）（需配合经纬度） |
| `minlatitude`| 浮点数 | 否       | -          | 边界框南侧边界 |
| `maxlatitude`| 浮点数 | 否       | -          | 边界框北侧边界 |
| `minlongitude`| 浮点数 | 否       | -          | 边界框西侧边界 |
| `maxlongitude`| 浮点数 | 否       | -          | 边界框东侧边界 |
| `limit`      | 整数   | 否       | -          | 返回事件上限（最大 20000） |
| `offset`     | 整数   | 否       | 1          | 分页偏移量（从 1 开始） |
| `orderby`    | 字符串 | 否       | `time`     | `time`, `time-asc`, `magnitude`, `magnitude-asc` |
| `alertlevel` | 字符串 | 否       | -          | PAGER 警报级别：`green`, `yellow`, `orange`, `red` |
| `eventtype`  | 字符串 | 否       | -          | 事件类型，如 `earthquake`, `quarry blast` |

**示例——区域内显著地震：**
```
https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&starttime=2024-01-01&endtime=2024-12-31&minmagnitude=5.0&minlatitude=30&maxlatitude=45&minlongitude=-125&maxlongitude=-110&orderby=magnitude
```

**GeoJSON 响应：**
```json
{
  "type": "FeatureCollection",
  "metadata": {
    "generated": 1700000000000,
    "url": "https://earthquake.usgs.gov/fdsnws/event/1/query?...",
    "title": "USGS Earthquakes",
    "status": 200,
    "api": "1.14.1",
    "count": 42
  },
  "features": [
    {
      "type": "Feature",
      "properties": {
        "mag": 6.2,
        "place": "加利福尼亚州里奇克雷斯特东北偏北 15 公里",
        "time": 1700000000000,
        "updated": 1700100000000,
        "tz": null,
        "url": "https://earthquake.usgs.gov/earthquakes/eventpage/ci00000001",
        "detail": "https://earthquake.usgs.gov/fdsnws/event/1/query?eventid=ci00000001&format=geojson",
        "felt": 1500,
        "cdi": 7.1,
        "mmi": 6.5,
        "alert": "yellow",
        "status": "reviewed",
        "tsunami": 0,
        "sig": 800,
        "net": "ci",
        "code": "00000001",
        "type": "earthquake",
        "title": "6.2 级地震 - 加利福尼亚州里奇克雷斯特东北偏北 15 公里"
      },
      "geometry": {
        "type": "Point",
        "coordinates": [-117.5, 35.8, 10.5]
      },
      "id": "ci00000001"
    }
  ]
}
```
注：`geometry.coordinates` 格式为 `[经度, 纬度, 深度_公里]`。

#### 2. 事件详情
```
GET /query?eventid={EVENTID}&format=geojson
```
返回单个事件的详细信息，包括矩张量、震源机制和附近城市。

#### 3. 事件计数
```
GET /count
```
参数与 `/query` 相同，仅返回匹配事件数量。适用于查询前检查结果规模。

**示例：**
```
https://earthquake.usgs.gov/fdsnws/event/1/count?starttime=2024-01-01&endtime=2024-12-31&minmagnitude=4.5
```

#### 4. 实时数据流（无参数）
预构建的 GeoJSON 数据流（每分钟/5分钟/15分钟/每小时更新）：
```
https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_month.geojson
https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/4.5_week.geojson
https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/2.5_day.geojson
https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_hour.geojson
```
模式：`{重要性}_{时间段}.geojson`，其中重要性为 `significant`, `4.5`, `2.5`, `1.0`, `all`，时间段为 `hour`, `day`, `week`, `month`。

---

## B 部分：水文服务

### 基础 URL
```
https://waterservices.usgs.gov/nwis
```

### 认证
**无需认证。** 完全公开，无需 API 密钥。

### 速率限制
- 无严格用户级限制，但 USGS 建议限制自动化请求。大型查询可能超时。

### 关键端点

#### 1. 瞬时值（实时数据）
```
GET /iv/
```
返回最新传感器读数（通常为 15 分钟间隔）。

**参数：**
| 参数名         | 类型   | 必填 | 默认值 | 描述 |
|----------------|--------|----------|---------|-------------|
| `format`       | 字符串 | 否       | `wml`   | `json`, `xml`, `wml,1.1`, `wml,2.0`, `rdb`。JSON 格式请用 `json` |
| `sites`        | 字符串 | 条件必填 | -       | 逗号分隔的 USGS 站点编号（如 `01646500`） |
| `stateCd`      | 字符串 | 条件必填 | -       | 2 字母州代码（如 `NY`） |
| `huc`          | 字符串 | 条件必填 | -       | 水文单元代码 |
| `bBox`         | 字符串 | 条件必填 | -       | 边界框：`西经,南纬,东经,北纬`（十进制度） |
| `countyCd`     | 字符串 | 条件必填 | -       | 5 位 FIPS 县代码 |
| `parameterCd`  | 字符串 | 否       | `00060` | 参数代码。`00060`=流量, `00065`=水位, `00010`=水温 |
| `period`       | 字符串 | 否       | -       | ISO8601 时长，如 `P7D`（过去 7 天） |
| `startDT`      | 字符串 | 否       | -       | 起始时间（ISO8601） |
| `endDT`        | 字符串 | 否       | -       | 结束时间（ISO8601） |
| `siteType`     | 字符串 | 否       | -       | 站点类型，如 `ST`（河流）, `GW`（地下水）, `LK`（湖泊） |
| `siteStatus`   | 字符串 | 否       | `all`   | `active`, `inactive`, `all` |

至少需提供一项位置参数（`sites`, `stateCd`, `huc`, `bBox` 或 `countyCd`）。

**示例——站点实时流量：**
```
https://waterservices.usgs.gov/nwis/iv/?format=json&sites=01646500&parameterCd=00060&period=P1D
```

**JSON 响应（节选）：**
```json
{
  "name": "ns1:timeSeriesResponseType",
  "declaredType": "org.cuahsi.waterml.TimeSeriesResponseType",
  "value": {
    "timeSeries": [
      {
        "sourceInfo": {
          "siteName": "波托马克河近华盛顿特区小瀑布泵站",
          "siteCode": [{"value": "01646500", "agencyCode": "USGS"}],
          "geoLocation": {
            "geogLocation": {"latitude": 38.94977778, "longitude": -77.12763889}
          }
        },
        "variable": {
          "variableCode": [{"value": "00060"}],
          "variableName": "流量，立方英尺/秒",
          "unit": {"unitCode": "ft3/s"}
        },
        "values": [
          {
            "value": [
              {"value": "5280", "dateTime": "2024-01-15T00:00:00.000-05:00"},
              {"value": "5310", "dateTime": "2024-01-15T00:15:00.000-05:00"}
            ]
          }
        ]
      }
    ]
  }
}
```

#### 2. 日值（历史聚合数据）
```
GET /dv/
```
返回日统计值（均值、最大值、最小值）。位置参数与 `/iv/` 相同。

**附加参数：**
| 参数名  | 类型   | 描述 |
|-----------|--------|-------------|
| `statCd`  | 字符串 | 统计代码：`00001`=最大值, `00002`=最小值, `00003`=均值, `00006`=总和。默认 `00003` |

**示例——站点年日均流量：**
```
https://waterservices.usgs.gov/nwis/dv/?format=json&sites=01646500&parameterCd=00060&statCd=00003&startDT=2023-01-01&endDT=2023-12-31
```

#### 3. 站点信息
```
GET /site/
```
返回监测站点元数据。适用相同位置参数。

**示例——弗吉尼亚州活跃河流站点：**
```
https://waterservices.usgs.gov/nwis/site/?format=rdb&stateCd=VA&siteType=ST&siteStatus=active&hasDataTypeCd=iv
```

#### 4. 统计值（预计算）
```
GET /stat/
```
返回预计算的日值统计（百分位、均值、中位数），用于比较当前状态与历史常态。

**示例：**
```
https://waterservices.usgs.gov/nwis/stat/?format=rdb&sites=01646500&parameterCd=00060&statReportType=daily&statTypeCd=mean,p05,p25,p50,p75,p95
```

### 常用参数代码
| 代码    | 描述 |
|---------|-------------|
| `00060` | 流量（立方英尺/秒） |
| `00065` | 水位（英尺） |
| `00010` | 水温（摄氏度） |
| `00045` | 降水量（英寸） |
| `00400` | pH 值 |
| `00300` | 溶解氧（毫克/升） |
| `00095` | 电导率（微西门子/厘米） |
| `72019` | 地下水位深度（地表下英尺） |

## 注意事项
- 地震 API 返回坐标格式为 `[经度, 纬度, 深度]`（注意：经度在前）
- 水文服务 JSON 采用类 WaterML 的冗长结构。`rdb`（制表符分隔）格式更适用于表格数据
- USGS 站点编号通常为 8 位（地表水）或 15 位（地下水）
- 所有 API 均为免费公开服务，无需认证
