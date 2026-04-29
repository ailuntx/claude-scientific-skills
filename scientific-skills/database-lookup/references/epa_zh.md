# EPA Envirofacts API 参考文档

## 基础 URL
```
https://data.epa.gov/efservice
```

注意：旧版 URL `https://enviro.epa.gov/enviro/efservice` 可能重定向，请使用上方当前基础 URL。

## 认证
**无需认证。** 完全公开，无需 API 密钥。

## 速率限制
- 无文档规定的用户级速率限制。
- 大型结果集可能超时，请使用行数限制和分页。

## URL 模式
Envirofacts 使用基于 URL 的 RESTful 查询模式：
```
https://data.epa.gov/efservice/{表名}/{列名}/{运算符}/{值}/.../rows/{起始}:{结束}/{格式}
```

- **{表名}**：数据库表名称（如 `TRI_FACILITY`, `AQS_SITES`）。
- **{列名}/{运算符}/{值}**：筛选条件，在 URL 路径中链式拼接。
- **运算符**：`=`（隐式，直接使用 `/{列名}/{值}`）、`>`、`<`、`!=`、`BEGINNING`（开头匹配）、`CONTAINING`（包含）。
- **rows/{起始}:{结束}**：分页行范围（从0开始计数）。
- **{格式}**：`JSON`、`XML`、`CSV`、`EXCEL`，作为最后路径段添加。

**多重筛选条件**在 URL 路径中顺序链式拼接。

---

## 核心数据库与数据表

### 1. 有毒物质排放清单 (TRI)

追踪工业设施的有毒化学品排放情况。

**核心数据表：**
| 表名 | 描述 |
|-------|-------------|
| `TRI_FACILITY` | 设施信息（名称、地址、坐标）。 |
| `TRI_REPORTING_FORM` | 年度报告表单数据。 |
| `TRI_RELEASE_QTY` | 按介质分类的排放量（空气、水、土壤）。 |
| `TRI_TRANSFER_QTY` | 场外转移量。 |
| `TRI_CHEM_INFO` | 化学品信息。 |

**示例——北卡罗来纳州的 TRI 设施：**
```
https://data.epa.gov/efservice/TRI_FACILITY/STATE_ABBR/NC/rows/0:9/JSON
```

**响应示例：**
```json
[
  {
    "TRI_FACILITY_ID": "27601MPLNT501WE",
    "FACILITY_NAME": "EXAMPLE MANUFACTURING PLANT",
    "STREET_ADDRESS": "501 WEST MAIN ST",
    "CITY_NAME": "RALEIGH",
    "COUNTY_NAME": "WAKE",
    "STATE_ABBR": "NC",
    "ZIP_CODE": "27601",
    "LATITUDE": 35.7796,
    "LONGITUDE": -78.6382,
    "FEDERAL_FACILITY_FLAG": "NO",
    "INDUSTRY_SECTOR_CODE": "325",
    "PRIMARY_SIC_CODE": "2819",
    "PRIMARY_NAICS_CODE": "325180"
  }
]
```

**示例——某州特定化学品的 TRI 排放量（2022年）：**
```
https://data.epa.gov/efservice/TRI_RELEASE_QTY/STATE_ABBR/TX/REPORTING_YEAR/2022/CHEM_NAME/CONTAINING/BENZENE/rows/0:24/JSON
```

### 2. 空气质量系统 (AQS)

来自国家监测网络的空气质量监测数据。

**核心数据表：**
| 表名 | 描述 |
|-------|-------------|
| `AQS_SITES` | 监测站点元数据。 |
| `AQS_MONITORS` | 监测器级别信息（测量参数）。 |
| `AQS_ANNUAL_SUMMARY` | 监测器年度统计摘要。 |
| `AQS_DAILY_SUMMARY` | 每日观测摘要。 |

**示例——加利福尼亚州的 AQS 监测站点：**
```
https://data.epa.gov/efservice/AQS_SITES/STATE_CODE/06/rows/0:9/JSON
```

**示例——某县臭氧年度摘要：**
```
https://data.epa.gov/efservice/AQS_ANNUAL_SUMMARY/STATE_CODE/06/COUNTY_CODE/037/PARAMETER_CODE/44201/rows/0:9/JSON
```

**常用 AQS 参数代码：**
| 代码  | 污染物 |
|-------|-----------|
| `44201` | 臭氧 |
| `42401` | 二氧化硫 |
| `42101` | 一氧化碳 |
| `42602` | 二氧化氮 |
| `81102` | 可吸入颗粒物（PM10） |
| `88101` | 细颗粒物（PM2.5，FRM标准） |
| `88502` | 细颗粒物（PM2.5，非FRM标准） |
| `14129` | 铅（Pb） |

### 3. 设施注册服务 (FRS)

EPA 监管设施中央注册库。

**核心数据表：**
| 表名 | 描述 |
|-------|-------------|
| `FRS_FACILITY_SITE` | 设施位置与标识符。 |
| `FRS_PROGRAM_FACILITY` | 设施与 EPA 项目关联。 |
| `FRS_NAICS` | 设施 NAICS 代码。 |
| `FRS_SIC` | 设施 SIC 代码。 |

**示例——按邮编查询 EPA 监管设施：**
```
https://data.epa.gov/efservice/FRS_FACILITY_SITE/POSTAL_CODE/90210/rows/0:9/JSON
```

### 4. 安全饮用水系统 (SDWIS)

公共饮用水系统数据。

**核心数据表：**
| 表名 | 描述 |
|-------|-------------|
| `WATER_SYSTEM` | 供水系统信息。 |
| `VIOLATION` | 饮用水违规记录。 |
| `LCR_SAMPLE_RESULT` | 铅铜规则采样结果。 |

**示例——某州饮用水违规记录：**
```
https://data.epa.gov/efservice/VIOLATION/PWSID/BEGINNING/OH/rows/0:19/JSON
```

### 5. 温室气体报告 (GHG)

设施级温室气体排放数据。

**核心数据表：**
| 表名 | 描述 |
|-------|-------------|
| `PUB_DIM_FACILITY` | GHG 报告设施信息。 |
| `PUB_FACTS_SECTOR_GHG_EMISSION` | 按行业分类的排放量。 |

**示例——某州 GHG 设施：**
```
https://data.epa.gov/efservice/PUB_DIM_FACILITY/STATE/TX/rows/0:9/JSON
```

---

## 查询模式

### 运算符筛选
```
# 精确匹配（隐式=）
/TABLE/COLUMN/VALUE/JSON

# 大于
/TABLE/COLUMN/>/VALUE/JSON

# 小于
/TABLE/COLUMN/</VALUE/JSON

# 不等于
/TABLE/COLUMN/!=/VALUE/JSON

# 开头匹配
/TABLE/COLUMN/BEGINNING/VALUE/JSON

# 包含匹配
/TABLE/COLUMN/CONTAINING/VALUE/JSON
```

### 组合筛选
链式拼接多组列/值对：
```
/TABLE/COLUMN1/VALUE1/COLUMN2/VALUE2/JSON
```

### 分页
使用 `rows/{起始}:{结束}`（从0开始计数，包含边界）：
```
/TABLE/rows/0:99/JSON       # 前100行
/TABLE/rows/100:199/JSON    # 后续100行
```
未指定 `rows` 时默认返回前10,000行。

### 输出格式
将格式作为最后路径段添加：
```
/TABLE/.../JSON
/TABLE/.../XML
/TABLE/.../CSV
/TABLE/.../EXCEL
```

---

## AQS 数据 API（独立系统）

如需更细粒度空气质量数据，EPA 另提供 AQS 数据 API：
```
https://aqs.epa.gov/data/api
```

- **要求：** 通过 https://aqs.epa.gov/data/api/signup?email=您的邮箱 注册免费账户
- **认证：** 通过查询参数传递 `email` 和 `key`。
- 核心端点：`/dailyData/byState`, `/annualData/byState`, `/sampleData/bySite`, `/monitors/byState`。

**示例：**
```
https://aqs.epa.gov/data/api/dailyData/byState?email=您的邮箱&key=您的密钥&param=44201&bdate=20240101&edate=20240131&state=06
```

## 注意事项
- URL 中的表名和列名不区分大小写。
- Envirofacts API 返回表的所有列，无法选择特定列。
- 跨表关联数据需分别请求，并在客户端使用共享键（如 `TRI_FACILITY_ID`, `REGISTRY_ID`）进行关联。
- 部分表数据量极大，请始终使用 `rows/` 限制结果并分页。
- EPA 数据更新频率因项目而异：TRI 为年度更新，AQS 为每日/年度更新，SDWIS 为季度更新。
