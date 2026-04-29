# NOAA 气候数据在线 (CDO) API 参考

## 基础 URL
```
https://www.ncdc.noaa.gov/cdo-web/api/v2
```

## 认证
- **API 令牌：必需。** 在 https://www.ncdc.noaa.gov/cdo-web/token 申请免费令牌
- 通过 HTTP 头传递：`Token: YOUR_TOKEN`

## 速率限制
- 每个令牌**每秒 5 个请求**
- 每个令牌**每天 10,000 个请求**
- 每次查询最多返回 **1,000 条结果**（使用 `offset` 分页）
- `/data` 端点每次请求的日期范围限制为 **1 年**

## 通用参数（适用于多数端点）
| 参数         | 类型   | 是否必需 | 默认值    | 描述 |
|--------------|--------|----------|-----------|------|
| `datasetid`  | string | 视情况   | -         | 数据集 ID（如 `GHCND`, `GSOM`） |
| `datatypeid` | string | 否       | -         | 数据类型过滤（如 `TMAX`, `PRCP`） |
| `locationid` | string | 否       | -         | 位置 ID（如 `FIPS:37`, `ZIP:28801`, `CITY:US390029`） |
| `stationid`  | string | 否       | -         | 站点 ID（如 `GHCND:USW00013874`） |
| `startdate`  | string | 视情况   | -         | ISO 日期 `YYYY-MM-DD` |
| `enddate`    | string | 视情况   | -         | ISO 日期 `YYYY-MM-DD` |
| `units`      | string | 否       | `standard` | `standard` 或 `metric` |
| `limit`      | int    | 否       | 25        | 每页结果数（最大 1000） |
| `offset`     | int    | 否       | 1         | 分页偏移量（从 1 开始） |
| `sortfield`  | string | 否       | -         | 排序字段（如 `date`, `name`） |
| `sortorder`  | string | 否       | `asc`     | `asc` 或 `desc` |

---

## 关键端点

### 1. 数据（观测值）
```
GET /data
```
返回实际观测数据。这是主要的数据检索端点。

**必需参数：** `datasetid`, `startdate`, `enddate`

**示例——获取站点每日最高温度：**
```bash
curl -H "Token: YOUR_TOKEN" \
  "https://www.ncdc.noaa.gov/cdo-web/api/v2/data?datasetid=GHCND&datatypeid=TMAX&stationid=GHCND:USW00013874&startdate=2024-01-01&enddate=2024-01-31&units=metric&limit=31"
```

**响应：**
```json
{
  "metadata": {
    "resultset": {
      "offset": 1,
      "count": 31,
      "limit": 31
    }
  },
  "results": [
    {
      "date": "2024-01-01T00:00:00",
      "datatype": "TMAX",
      "station": "GHCND:USW00013874",
      "attributes": ",,W,2400",
      "value": 12.2
    },
    {
      "date": "2024-01-02T00:00:00",
      "datatype": "TMAX",
      "station": "GHCND:USW00013874",
      "attributes": ",,W,2400",
      "value": 8.9
    }
  ]
}
```
注意：当 `units=standard` 时，GHCND 温度值单位为摄氏度的十分之一；使用 `units=metric` 时会转换为摄氏度。

### 2. 数据集
```
GET /datasets
GET /datasets/{id}
```
列出可用数据集或获取单个数据集详情。

**示例：**
```bash
curl -H "Token: YOUR_TOKEN" \
  "https://www.ncdc.noaa.gov/cdo-web/api/v2/datasets?limit=10"
```

**关键数据集 ID：**
| ID          | 名称 | 描述 |
|-------------|------|------|
| `GHCND`     | 每日摘要 | 全球每日站点观测数据（TMAX, TMIN, PRCP, SNOW 等） |
| `GSOM`      | 全球月摘要 | 月度聚合数据 |
| `GSOY`      | 全球年摘要 | 年度聚合数据 |
| `NORMAL_DLY`| 日气候常态 | 30 年日气候常态 |
| `NORMAL_MLY`| 月气候常态 | 30 年月气候常态 |
| `PRECIP_15` | 15 分钟降水 | 亚小时级降水数据 |
| `PRECIP_HLY`| 小时降水 | 小时级降水数据 |

### 3. 数据类型
```
GET /datatypes
GET /datatypes/{id}
```
列出可用数据类型，可按数据集筛选。

**示例：**
```bash
curl -H "Token: YOUR_TOKEN" \
  "https://www.ncdc.noaa.gov/cdo-web/api/v2/datatypes?datasetid=GHCND&limit=50"
```

**常用 GHCND 数据类型：**
| ID    | 描述 |
|-------|------|
| `TMAX`| 最高温度 |
| `TMIN`| 最低温度 |
| `TAVG`| 平均温度 |
| `PRCP`| 降水量 |
| `SNOW`| 降雪量 |
| `SNWD`| 积雪深度 |
| `AWND`| 平均风速 |
| `WSF2`| 最快 2 分钟风速 |

### 4. 站点
```
GET /stations
GET /stations/{id}
```
查找气象站点，可按位置、数据集或范围筛选。

**附加参数：**
| 参数     | 类型   | 描述 |
|----------|--------|------|
| `extent` | string | 边界框：`南纬,西经,北纬,东经` |

**示例——北卡罗来纳州阿什维尔附近提供日数据的站点：**
```bash
curl -H "Token: YOUR_TOKEN" \
  "https://www.ncdc.noaa.gov/cdo-web/api/v2/stations?datasetid=GHCND&locationid=ZIP:28801&limit=10"
```

**响应：**
```json
{
  "metadata": {"resultset": {"offset": 1, "count": 5, "limit": 10}},
  "results": [
    {
      "elevation": 661.1,
      "mindate": "1893-01-01",
      "maxdate": "2024-11-15",
      "latitude": 35.5951,
      "name": "ASHEVILLE REGIONAL AIRPORT, NC US",
      "datacoverage": 1,
      "id": "GHCND:USW00013874",
      "elevationUnit": "METERS",
      "longitude": -82.5572
    }
  ]
}
```

### 5. 位置和位置类别
```
GET /locations
GET /locations/{id}
GET /locationcategories
GET /locationcategories/{id}
```
浏览位置层级（国家、州、城市、邮编、气候区）。

**示例：**
```bash
curl -H "Token: YOUR_TOKEN" \
  "https://www.ncdc.noaa.gov/cdo-web/api/v2/locations?locationcategoryid=ST&limit=52"
```

位置类别 ID：`CITY`, `CLIM_DIV`, `CLIM_REG`, `CNTRY`, `CNTY`, `HYD_ACC`, `HYD_CAT`, `HYD_REG`, `HYD_SUB`, `ST`, `ZIP`

---

## 工作流程：查找与查询数据

1. **查找数据集：** `GET /datasets` 列出可用数据集
2. **查找站点：** `GET /stations?datasetid=GHCND&locationid=ZIP:28801` 查找附近站点
3. **检查可用数据类型：** `GET /datatypes?datasetid=GHCND&stationid=GHCND:USW00013874`
4. **查询数据：** `GET /data?datasetid=GHCND&stationid=GHCND:USW00013874&datatypeid=TMAX,TMIN&startdate=2024-01-01&enddate=2024-12-31&units=metric&limit=1000`

## 注意事项
- `/data` 端点强制要求**每次请求日期范围不超过 1 年**。多年度查询需分次请求
- 分页：`offset` 从 1 开始计数。循环查询直到 `offset + limit > count`（来自元数据）
- 站点 ID 包含数据集前缀（如 `GHCND:USW00013874`）
- 数据结果中的 `attributes` 字段包含质量标志（逗号分隔）。标志含义请查阅数据集文档
- 令牌需通过 HTTP 头传递，不可作为查询参数
