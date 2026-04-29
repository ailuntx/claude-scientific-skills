# FRED（联邦储备经济数据）API 参考文档

## 概述
FRED API 由圣路易斯联邦储备银行提供，可访问来自 100 多个来源的超过 800,000 个经济时间序列。涵盖 GDP、就业、通货膨胀、利率、货币供应量、贸易、住房等众多领域。

## 基础 URL
```
https://api.stlouisfed.org/fred
```

## 认证
- **API 密钥：必需。** 在 https://fred.stlouisfed.org/docs/api/api_key.html 注册
- 通过查询参数传递：`&api_key=YOUR_KEY`

## 速率限制
- 每个 API 密钥**每分钟 120 个请求**
- 未记录每日限制，但过度使用可能触发节流

## 通用参数（适用于多数端点）
| 参数            | 类型     | 是否必需   | 默认值     | 描述                  |
|----------------|--------|----------|-----------|-----------------------|
| `api_key`      | 字符串  | 是        | -         | 您的 FRED API 密钥    |
| `file_type`    | 字符串  | 否        | `xml`     | 响应格式：`xml` 或 `json` |
| `realtime_start` | 字符串 | 否       | 当天日期   | 实时周期开始日期 `YYYY-MM-DD` |
| `realtime_end`   | 字符串 | 否       | 当天日期   | 实时周期结束日期 `YYYY-MM-DD` |

---

## 关键端点

### 1. 序列观测值（时间序列数据）

#### `GET /fred/series/observations`
返回经济时间序列的数据值

**参数：**
| 参数                  | 类型     | 是否必需   | 默认值        | 描述                  |
|----------------------|--------|----------|--------------|-----------------------|
| `series_id`          | 字符串  | 是        | -            | FRED 序列 ID（如 `GDP`, `UNRATE`, `CPIAUCSL`） |
| `observation_start`  | 字符串  | 否        | `1776-07-04` | 开始日期 `YYYY-MM-DD` |
| `observation_end`    | 字符串  | 否        | `9999-12-31` | 结束日期 `YYYY-MM-DD` |
| `units`              | 字符串  | 否        | `lin`        | 数据转换：`lin`（原始值），`chg`（变化量），`ch1`（与上年同期相比的变化量），`pch`（百分比变化），`pc1`（与上年同期相比的百分比变化），`pca`（复合年化百分比变化），`cch`（连续复合变化率），`cca`（连续复合年化变化率），`log`（自然对数） |
| `frequency`          | 字符串  | 否        | (原始频率)    | 聚合频率：`d`, `w`, `bw`, `m`, `q`, `sa`, `a`（日度至年度） |
| `aggregation_method` | 字符串  | 否        | `avg`        | `avg`（平均），`sum`（求和），`eop`（期末值） |
| `sort_order`         | 字符串  | 否        | `asc`        | `asc`（升序）或 `desc`（降序） |
| `limit`              | 整数    | 否        | 100000       | 返回的最大观测值数量（上限 100000） |
| `offset`             | 整数    | 否        | 0            | 分页偏移量            |

**示例：**
```
https://api.stlouisfed.org/fred/series/observations?series_id=GDP&api_key=YOUR_KEY&file_type=json&observation_start=2020-01-01&observation_end=2024-12-31&units=pch&frequency=q
```

**响应：**
```json
{
  "realtime_start": "2024-11-01",
  "realtime_end": "2024-11-01",
  "observation_start": "2020-01-01",
  "observation_end": "2024-12-31",
  "units": "Percent Change",
  "output_type": 1,
  "file_type": "json",
  "order_by": "observation_date",
  "sort_order": "asc",
  "count": 20,
  "offset": 0,
  "limit": 100000,
  "observations": [
    {
      "realtime_start": "2024-11-01",
      "realtime_end": "2024-11-01",
      "date": "2020-01-01",
      "value": "-1.3"
    },
    {
      "realtime_start": "2024-11-01",
      "realtime_end": "2024-11-01",
      "date": "2020-04-01",
      "value": "-8.4"
    }
  ]
}
```

注意：`value` 始终为字符串。缺失值显示为 `"."`

---

### 2. 序列信息（元数据）

#### `GET /fred/series`
返回序列的元数据

**参数：**
| 参数         | 类型     | 是否必需   | 描述          |
|------------|--------|----------|---------------|
| `series_id`| 字符串  | 是        | FRED 序列 ID  |

**示例：**
```
https://api.stlouisfed.org/fred/series?series_id=UNRATE&api_key=YOUR_KEY&file_type=json
```

**响应：**
```json
{
  "realtime_start": "2024-11-01",
  "realtime_end": "2024-11-01",
  "seriess": [
    {
      "id": "UNRATE",
      "title": "Unemployment Rate",
      "observation_start": "1948-01-01",
      "observation_end": "2024-10-01",
      "frequency": "Monthly",
      "frequency_short": "M",
      "units": "Percent",
      "units_short": "%",
      "seasonal_adjustment": "Seasonally Adjusted",
      "seasonal_adjustment_short": "SA",
      "last_updated": "2024-11-01 07:41:02-05",
      "popularity": 95,
      "notes": "The unemployment rate represents..."
    }
  ]
}
```

---

### 3. 序列搜索

#### `GET /fred/series/search`
通过关键词搜索序列

**参数：**
| 参数               | 类型     | 是否必需   | 默认值         | 描述                  |
|-------------------|--------|----------|---------------|-----------------------|
| `search_text`     | 字符串  | 是        | -             | 搜索关键词            |
| `search_type`     | 字符串  | 否        | `full_text`   | `full_text`（全文）或 `series_id`（序列ID） |
| `order_by`        | 字符串  | 否        | `search_rank` | 排序依据：`search_rank`, `series_id`, `title`, `units`, `frequency`, `seasonal_adjustment`, `realtime_start`, `realtime_end`, `last_updated`, `observation_start`, `observation_end`, `popularity`, `group_popularity` |
| `sort_order`      | 字符串  | 否        | `asc`         | `asc`（升序）或 `desc`（降序） |
| `limit`           | 整数    | 否        | 1000          | 最大结果数（上限 1000） |
| `offset`          | 整数    | 否        | 0             | 分页偏移量            |
| `filter_variable` | 字符串  | 否        | -             | 筛选字段：`frequency`, `units`, `seasonal_adjustment` |
| `filter_value`    | 字符串  | 否        | -             | 筛选值（如 `Monthly`） |
| `tag_names`       | 字符串  | 否        | -             | 分号分隔的标签筛选（如 `gdp;quarterly`） |

**示例：**
```
https://api.stlouisfed.org/fred/series/search?search_text=consumer+price+index&api_key=YOUR_KEY&file_type=json&limit=5
```

**响应：**
```json
{
  "realtime_start": "2024-11-01",
  "realtime_end": "2024-11-01",
  "order_by": "search_rank",
  "sort_order": "asc",
  "count": 1256,
  "offset": 0,
  "limit": 5,
  "seriess": [
    {
      "id": "CPIAUCSL",
      "title": "Consumer Price Index for All Urban Consumers: All Items in U.S. City Average",
      "observation_start": "1947-01-01",
      "observation_end": "2024-09-01",
      "frequency": "Monthly",
      "units": "Index 1982-1984=100",
      "seasonal_adjustment": "Seasonally Adjusted",
      "popularity": 95
    }
  ]
}
```

---

### 4. 分类查询

#### `GET /fred/category`
获取特定分类信息

**参数：**
| 参数          | 类型   | 是否必需   | 描述          |
|-------------|------|----------|---------------|
| `category_id`| 整数  | 是        | 分类 ID（0 = 根分类） |

**示例：**
```
https://api.stlouisfed.org/fred/category?category_id=0&api_key=YOUR_KEY&file_type=json
```

#### `GET /fred/category/children`
获取子分类

**示例：**
```
https://api.stlouisfed.org/fred/category/children?category_id=0&api_key=YOUR_KEY&file_type=json
```

#### `GET /fred/category/series`
获取分类下所有序列

**参数：**
| 参数          | 类型   | 是否必需   | 描述          |
|-------------|------|----------|---------------|
| `category_id`| 整数  | 是        | 分类 ID       |
| `limit`     | 整数  | 否        | 最大结果数（上限 1000） |
| `offset`    | 整数  | 否        | 分页偏移量      |

**示例：**
```
https://api.stlouisfed.org/fred/category/series?category_id=125&api_key=YOUR_KEY&file_type=json
```

---

### 5. 数据发布

#### `GET /fred/releases`
获取所有经济数据发布

**示例
