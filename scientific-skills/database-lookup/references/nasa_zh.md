# NASA API

## 基础URL

```
https://api.nasa.gov
```

## 认证

所有端点都需要通过 `api_key` 查询参数传递 API 密钥。
- 在此获取免费密钥：https://api.nasa.gov/#signUp
- 演示密钥：`DEMO_KEY`（限速：每个IP 30次/小时，50次/天）
- 已注册密钥：1,000次/小时

## 关键端点

### 1. APOD（每日天文图片）

```
GET /planetary/apod
```

**参数：**

| 参数         | 类型    | 描述 |
|--------------|---------|------|
| `api_key`    | string  | **必填。** API 密钥。 |
| `date`       | string  | YYYY-MM-DD。默认：当天。 |
| `start_date` | string  | 日期范围起始（YYYY-MM-DD）。 |
| `end_date`   | string  | 日期范围结束（YYYY-MM-DD）。 |
| `count`      | int     | 返回 N 张随机图片（不可与日期/范围参数共用）。 |
| `thumbs`     | bool    | 为视频类条目返回缩略图 URL。 |

**示例：**
```
https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY&date=2024-01-15
```

**响应（JSON）：**
```json
{
  "date": "2024-01-15",
  "title": "...",
  "explanation": "...",
  "url": "https://apod.nasa.gov/apod/image/...",
  "hdurl": "https://apod.nasa.gov/apod/image/...",
  "media_type": "image",
  "copyright": "..."
}
```

### 2. NEO —— 近地天体（小行星 NeoWs）

```
GET /neo/rest/v1/feed
```

**参数：**

| 参数         | 类型   | 描述 |
|--------------|--------|------|
| `api_key`    | string | **必填。** |
| `start_date` | string | YYYY-MM-DD。默认：当天。 |
| `end_date`   | string | YYYY-MM-DD。与起始日期间隔不超过 7 天。 |

**示例：**
```
https://api.nasa.gov/neo/rest/v1/feed?start_date=2024-01-01&end_date=2024-01-03&api_key=DEMO_KEY
```

**通过小行星 ID 查询：**
```
GET /neo/rest/v1/neo/{asteroid_id}?api_key=DEMO_KEY
```

**浏览全部：**
```
GET /neo/rest/v1/neo/browse?api_key=DEMO_KEY
```

**响应结构：** 按日期键控的 `near_earth_objects` 字段，每个日期包含对象数组，对象含 `name`、`nasa_jpl_url`、`estimated_diameter`、`close_approach_data`、`is_potentially_hazardous_asteroid` 等属性。

### 3. 火星车照片

```
GET /mars-photos/api/v1/rovers/{rover}/photos
```

火星车类型：`curiosity`, `opportunity`, `spirit`, `perseverance`

**参数：**

| 参数        | 类型   | 描述 |
|-------------|--------|------|
| `api_key`   | string | **必填。** |
| `sol`       | int    | 火星日（Sol）。`sol` 与 `earth_date` 不可共用。 |
| `earth_date`| string | YYYY-MM-DD。 |
| `camera`    | string | 相机筛选：`FHAZ`, `RHAZ`, `MAST`, `CHEMCAM`, `NAVCAM` 等。 |
| `page`      | int    | 每页 25 条结果。 |

**示例：**
```
https://api.nasa.gov/mars-photos/api/v1/rovers/curiosity/photos?sol=1000&camera=NAVCAM&api_key=DEMO_KEY
```

**火星车任务清单（元数据）：**
```
GET /mars-photos/api/v1/manifests/{rover}?api_key=DEMO_KEY
```

**响应：** `photos` 数组，每个元素含 `id`、`sol`、`camera`（含 `full_name`）、`img_src`、`earth_date`、`rover` 等字段。

## 速率限制

| 密钥类型     | 每小时限制 | 每日限制 |
|--------------|------------|----------|
| `DEMO_KEY`   | 30/小时    | 50/天    |
| 已注册密钥   | 1,000/小时 | 无限制   |

速率限制响应头：`X-RateLimit-Limit`、`X-RateLimit-Remaining`。
