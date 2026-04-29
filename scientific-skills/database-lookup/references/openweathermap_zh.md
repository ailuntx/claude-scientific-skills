# OpenWeatherMap API 参考文档

## 基础 URL
```
https://api.openweathermap.org
```

## 认证
- **API密钥：必需。** 在 https://home.openweathermap.org/users/sign_up 注册免费密钥
- 通过查询参数传递：`&appid=YOUR_KEY`
- 免费层级密钥在注册后数小时内激活

## 速率限制（免费层级）
- **每分钟 60 次调用**（部分接口每日 1,000 次）
- **当前天气、5天预报、地理编码：** 免费层级可用
- **One Call 3.0：** 需订阅（绑定信用卡可享每日 1,000 次免费调用）
- **历史天气、空气污染历史数据：** 需付费计划获取扩展范围

---

## 核心接口

### 1. 当前天气
```
GET /data/2.5/weather
```

**参数：**
| 参数       | 类型    | 必填     | 默认值     | 描述 |
|------------|---------|----------|------------|------|
| `q`        | 字符串  | 条件必填 | -          | 城市名（可附加州/国家）：`London`，`London,GB`，`Portland,OR,US` |
| `lat`      | 浮点数  | 条件必填 | -          | 纬度（需配合`lon`使用） |
| `lon`      | 浮点数  | 条件必填 | -          | 经度（需配合`lat`使用） |
| `id`       | 整数    | 条件必填 | -          | 城市ID（来自OWM城市列表） |
| `zip`      | 字符串  | 条件必填 | -          | 邮政编码（含国家）：`90210,US`，`SW1,GB` |
| `units`    | 字符串  | 否       | `standard` | `standard`（开尔文），`metric`（摄氏度），`imperial`（华氏度） |
| `lang`     | 字符串  | 否       | `en`       | 描述语言代码 |
| `appid`    | 字符串  | 是       | -          | API密钥 |

必须提供至少一个位置参数（`q`、`lat`+`lon`、`id`或`zip`）

**示例：**
```
https://api.openweathermap.org/data/2.5/weather?lat=40.7128&lon=-74.0060&units=metric&appid=YOUR_KEY
```

**响应：**
```json
{
  "coord": {"lon": -74.006, "lat": 40.7128},
  "weather": [
    {
      "id": 800,
      "main": "Clear",
      "description": "晴空",
      "icon": "01d"
    }
  ],
  "base": "stations",
  "main": {
    "temp": 22.5,
    "feels_like": 21.8,
    "temp_min": 20.1,
    "temp_max": 24.3,
    "pressure": 1013,
    "humidity": 55,
    "sea_level": 1013,
    "grnd_level": 1010
  },
  "visibility": 10000,
  "wind": {"speed": 3.6, "deg": 220, "gust": 5.1},
  "clouds": {"all": 0},
  "dt": 1700000000,
  "sys": {
    "country": "US",
    "sunrise": 1699960000,
    "sunset": 1699996000
  },
  "timezone": -18000,
  "id": 5128581,
  "name": "New York",
  "cod": 200
}
```

### 2. 5天/3小时预报（免费）
```
GET /data/2.5/forecast
```
返回5天内每3小时间隔的预报数据（共40个数据点）。位置参数与当前天气接口相同。

**附加参数：**
| 参数 | 类型 | 描述 |
|------|------|------|
| `cnt` | 整数 | 返回的3小时步长数量（上限40） |

**示例：**
```
https://api.openweathermap.org/data/2.5/forecast?q=London,GB&units=metric&cnt=8&appid=YOUR_KEY
```

**响应：**
```json
{
  "cod": "200",
  "message": 0,
  "cnt": 8,
  "list": [
    {
      "dt": 1700000000,
      "main": {
        "temp": 10.5,
        "feels_like": 8.2,
        "temp_min": 9.8,
        "temp_max": 10.5,
        "pressure": 1020,
        "humidity": 80
      },
      "weather": [{"id": 802, "main": "Clouds", "description": "疏云", "icon": "03d"}],
      "clouds": {"all": 40},
      "wind": {"speed": 4.1, "deg": 250},
      "visibility": 10000,
      "pop": 0.2,
      "dt_txt": "2024-01-15 12:00:00"
    }
  ],
  "city": {
    "id": 2643743,
    "name": "London",
    "coord": {"lat": 51.5085, "lon": -0.1257},
    "country": "GB",
    "population": 1000000,
    "timezone": 0,
    "sunrise": 1699950000,
    "sunset": 1699982000
  }
}
```
`pop`表示降水概率（0.0至1.0）

### 3. 地理编码
```
GET /geo/1.0/direct
GET /geo/1.0/reverse
GET /geo/1.0/zip
```

**正向地理编码（城市名转坐标）：**
```
https://api.openweathermap.org/geo/1.0/direct?q=London,GB&limit=5&appid=YOUR_KEY
```
返回数组格式：`{name, lat, lon, country, state}`

**反向地理编码（坐标转城市名）：**
```
https://api.openweathermap.org/geo/1.0/reverse?lat=51.5085&lon=-0.1257&limit=1&appid=YOUR_KEY
```

**邮政编码地理编码：**
```
https://api.openweathermap.org/geo/1.0/zip?zip=90210,US&appid=YOUR_KEY
```

### 4. One Call API 3.0（需订阅）
```
GET /data/3.0/onecall
```
综合接口，单次调用返回当前天气、分钟级（1小时）、小时级（48小时）、每日（8天）预报及警报

**参数：**
| 参数     | 类型    | 必填 | 描述 |
|----------|---------|------|------|
| `lat`    | 浮点数  | 是   | 纬度 |
| `lon`    | 浮点数  | 是   | 经度 |
| `exclude`| 字符串  | 否   | 需排除的模块（逗号分隔）：`current`, `minutely`, `hourly`, `daily`, `alerts` |
| `units`  | 字符串  | 否   | `standard`, `metric`, `imperial` |
| `appid`  | 字符串  | 是   | API密钥 |

**示例：**
```
https://api.openweathermap.org/data/3.0/onecall?lat=40.7128&lon=-74.006&exclude=minutely,alerts&units=metric&appid=YOUR_KEY
```

### 5. 空气污染
```
GET /data/2.5/air_pollution
GET /data/2.5/air_pollution/forecast
GET /data/2.5/air_pollution/history
```

**参数：** `lat`, `lon`, `appid`（必填）。历史数据需：`start`和`end`（Unix时间戳）

**示例：**
```
https://api.openweathermap.org/data/2.5/air_pollution?lat=40.7128&lon=-74.006&appid=YOUR_KEY
```

**响应：**
```json
{
  "coord": {"lon": -74.006, "lat": 40.7128},
  "list": [
    {
      "main": {"aqi": 2},
      "components": {
        "co": 230.31,
        "no": 0.5,
        "no2": 15.0,
        "o3": 68.0,
        "so2": 2.5,
        "pm2_5": 8.1,
        "pm10": 12.3,
        "nh3": 1.0
      },
      "dt": 1700000000
    }
  ]
}
```
AQI等级：1=优，2=良，3=中，4=差，5=极差。污染物单位为ug/m3

---

## 天气状态码
| 范围   | 类别 |
|--------|------|
| 2xx    | 雷暴 |
| 3xx    | 细雨 |
| 5xx    | 降雨 |
| 6xx    | 降雪 |
| 7xx    | 大气现象（雾/薄雾/霾） |
| 800    | 晴 |
| 80x    | 多云 |

天气图标：`https://openweathermap.org/img/wn/{icon}@2x.png`

## 免费版与付费版对比
| 接口 | 免费版 | 订阅版 |
|------|--------|--------|
| 当前天气 | ✓ | ✓ |
| 5天/3小时预报 | ✓ | ✓ |
| 地理编码 | ✓ | ✓ |
| 空气污染（当前） | ✓ | ✓ |
| One Call 3.0 | 每日1,000次（需绑卡） | ✓ |
| 历史天气 | ✗ | ✓ |
| 16天每日预报 | ✗ | ✓ |
| 30天气候预报 | ✗ | ✓ |

## 注意事项
- 所有时间戳（`dt`, `sunrise`, `sunset`）均为 **Unix纪元秒数（UTC）**
- `timezone`字段为UTC偏移秒数（例：-18000 = UTC-5）
- 默认温度单位为开尔文。建议始终指定`units=metric`或`units=imperial`
- 城市名查询（`q=`）可能存在歧义。建议优先使用`lat`+`lon`精确定位，可先通过地理编码获取坐标
- 天气图标URL格式：`https://openweathermap.org/img/wn/{icon}@2x.png`（例：`01d`表示晴天）
- 错误响应示例：`{"cod": 401, "message": "无效API密钥"}`
