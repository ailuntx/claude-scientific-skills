# 美联储经济数据 (FRED) API

## 基础URL

```
https://api.stlouisfed.org/fred
```

## 认证

**需要API密钥。** 在 https://fred.stlouisfed.org/docs/api/api_key.html 注册

通过查询参数传递：`&api_key=YOUR_KEY`

## 关键端点

### 获取序列（元数据）
```
GET /series
```
| 参数       | 是否必需 | 描述                          |
|------------|----------|-------------------------------|
| series_id  | 是       | FRED序列ID（例如 `FEDFUNDS`）|
| api_key    | 是       | 您的API密钥                   |
| file_type  | 否       | `json`（默认），`xml`         |

示例：
```
https://api.stlouisfed.org/fred/series?series_id=FEDFUNDS&api_key=YOUR_KEY&file_type=json
```

### 获取序列观测值（实际数据点）
```
GET /series/observations
```
| 参数               | 是否必需 | 描述                                            |
|--------------------|----------|------------------------------------------------|
| series_id          | 是       | FRED序列ID                                     |
| api_key            | 是       | 您的API密钥                                     |
| file_type          | 否       | `json`, `xml`                                  |
| observation_start  | 否       | 起始日期 `YYYY-MM-DD`                          |
| observation_end    | 否       | 结束日期 `YYYY-MM-DD`                          |
| units              | 否       | `lin`（水平值）, `chg`, `ch1`, `pch`, `pc1`, `pca`, `cch`, `cca`, `log` |
| frequency          | 否       | `d`, `w`, `bw`, `m`, `q`, `sa`, `a`（日度至年度）|
| aggregation_method | 否       | `avg`, `sum`, `eop`                            |
| sort_order         | 否       | `asc`（默认）, `desc`                          |
| limit              | 否       | 最大观测数（默认100000）                       |
| offset             | 否       | 分页偏移量                                      |

示例：
```
https://api.stlouisfed.org/fred/series/observations?series_id=FEDFUNDS&api_key=YOUR_KEY&file_type=json&observation_start=2023-01-01&observation_end=2024-01-01
```

### 搜索序列
```
GET /series/search
```
| 参数         | 是否必需 | 描述                                  |
|--------------|----------|---------------------------------------|
| search_text  | 是       | 搜索关键词                            |
| api_key      | 是       | 您的API密钥                           |
| file_type    | 否       | `json`, `xml`                         |
| search_type  | 否       | `full_text`（默认）, `series_id`      |
| limit        | 否       | 最大结果数（默认1000）                |
| offset       | 否       | 分页偏移量                            |
| order_by     | 否       | `search_rank`, `series_id`, `title`, `units`, `frequency`, `seasonal_adjustment`, `realtime_start`, `realtime_end`, `last_updated`, `observation_start`, `observation_end`, `popularity`, `group_popularity` |
| tag_names    | 否       | 分号分隔的标签过滤器                  |

示例：
```
https://api.stlouisfed.org/fred/series/search?search_text=monetary+base&api_key=YOUR_KEY&file_type=json&limit=10
```

### 获取序列分类
```
GET /series/categories
```
示例：
```
https://api.stlouisfed.org/fred/series/categories?series_id=FEDFUNDS&api_key=YOUR_KEY&file_type=json
```

### 浏览分类
```
GET /category
GET /category/children
GET /category/series
```
示例（根分类）：
```
https://api.stlouisfed.org/fred/category?category_id=0&api_key=YOUR_KEY&file_type=json
```

### 获取数据发布
```
GET /releases
GET /release/series
```
示例：
```
https://api.stlouisfed.org/fred/release/series?release_id=10&api_key=YOUR_KEY&file_type=json
```

### 获取标签
```
GET /tags
GET /series/tags
```

## 常用序列ID

| 序列ID       | 描述                              |
|--------------|-----------------------------------|
| FEDFUNDS     | 联邦基金有效利率                  |
| DFF          | 联邦基金利率（日度）              |
| DGS10        | 10年期国债恒定到期收益率          |
| DGS2         | 2年期国债恒定到期收益率           |
| M2SL         | M2货币供应量                      |
| CPIAUCSL     | 消费者价格指数（所有城市消费者）  |
| UNRATE       | 失业率                            |
| GDP          | 国内生产总值                      |
| GDPC1        | 实际GDP                           |
| A191RL1Q225SBEA | 实际GDP增长率（季度）         |
| PAYEMS       | 非农就业人数总计                  |
| T10Y2Y       | 10年-2年国债利差                  |
| MORTGAGE30US | 30年期固定抵押贷款利率            |
| DTWEXBGS     | 贸易加权美元指数                  |
| BOGMBASE     | 基础货币（总量）                  |
| WALCL        | 美联储总资产                      |

## 响应格式

### 序列元数据 (`/series`)
```json
{
  "realtime_start": "2024-01-01",
  "realtime_end": "2024-01-01",
  "seriess": [
    {
      "id": "FEDFUNDS",
      "realtime_start": "2024-01-01",
      "realtime_end": "2024-01-01",
      "title": "Federal Funds Effective Rate",
      "observation_start": "1954-07-01",
      "observation_end": "2024-01-01",
      "frequency": "Monthly",
      "frequency_short": "M",
      "units": "Percent",
      "units_short": "%",
      "seasonal_adjustment": "Not Seasonally Adjusted",
      "seasonal_adjustment_short": "NSA",
      "last_updated": "2024-02-01 15:51:07-06",
      "popularity": 95,
      "notes": "..."
    }
  ]
}
```

### 观测值 (`/series/observations`)
```json
{
  "realtime_start": "2024-01-01",
  "realtime_end": "2024-01-01",
  "observation_start": "2023-01-01",
  "observation_end": "2024-01-01",
  "units": "lin",
  "output_type": 1,
  "file_type": "json",
  "order_by": "observation_date",
  "sort_order": "asc",
  "count": 12,
  "offset": 0,
  "limit": 100000,
  "observations": [
    {
      "realtime_start": "2024-01-01",
      "realtime_end": "2024-01-01",
      "date": "2023-01-01",
      "value": "4.33"
    }
  ]
}
```

注意：`value` 始终为字符串类型。缺失数据显示为 `"."`。

### 搜索结果 (`/series/search`)
```json
{
  "realtime_start": "...",
  "realtime_end": "...",
  "order_by": "search_rank",
  "sort_order": "desc",
  "count": 500,
  "offset": 0,
  "limit": 1000,
  "seriess": [
    {
      "id": "BOGMBASE",
      "title": "Monetary Base; Total",
      "frequency": "Bi-Weekly",
      "units": "Millions of Dollars",
      "popularity": 72,
      "notes": "..."
    }
  ]
}
```

## 速率限制

- 每个API密钥 **每分钟120次请求**
- 未记录每日限制，但过度使用可能被限流
- 响应不含速率限制标头；需在客户端实现限流控制
