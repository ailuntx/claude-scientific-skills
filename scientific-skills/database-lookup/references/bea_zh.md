# BEA（经济分析局）API参考

## 概述
经济分析局API提供美国经济账户数据访问，包括GDP（国民收入和生产账户——NIPA）、个人收入、国际贸易、行业账户和区域经济数据。采用单一端点结构，支持数据集特定参数。

## 基础URL
```
https://apps.bea.gov/api/data
```

## 认证
- **API密钥：必需**。在 https://apps.bea.gov/API/signup/ 注册
- 通过查询参数传递：`&UserID=您的API密钥`

## 速率限制
- 每个API密钥**每分钟100次请求**
- 每个API密钥**每分钟100 MB数据量**
- **每分钟30次错误**——超出将触发临时锁定
- 未正式公布每日/每月限制，但BEA可能对重度使用进行限流

## 通用参数（所有请求）
| 参数          | 类型   | 必填 | 描述               |
|---------------|--------|------|--------------------|
| `UserID`      | 字符串 | 是   | 您的BEA API密钥    |
| `method`      | 字符串 | 是   | API方法（见下文）  |
| `ResultFormat`| 字符串 | 否   | `JSON`（默认）或`XML` |

---

## 方法

### 1. GetDataSetList
列出所有可用数据集。

#### `GET /api/data?method=GetDataSetList&UserID=您的密钥&ResultFormat=JSON`

**示例：**
```
https://apps.bea.gov/api/data?method=GetDataSetList&UserID=您的密钥&ResultFormat=JSON
```

**响应：**
```json
（保持原JSON不变）
```

---

### 2. GetParameterList
列出特定数据集的参数。

#### `GET /api/data?method=GetParameterList&DatasetName={数据集}&UserID=您的密钥&ResultFormat=JSON`

**示例：**
```
https://apps.bea.gov/api/data?method=GetParameterList&DatasetName=NIPA&UserID=您的密钥&ResultFormat=JSON
```

**响应：**
```json
（保持原JSON不变）
```

---

### 3. GetParameterValues
列出参数的有效值。

#### `GET /api/data?method=GetParameterValues&DatasetName={数据集}&ParameterName={参数}&UserID=您的密钥&ResultFormat=JSON`

**示例（列出NIPA表）：**
```
https://apps.bea.gov/api/data?method=GetParameterValues&DatasetName=NIPA&ParameterName=TableName&UserID=您的密钥&ResultFormat=JSON
```

**响应（节选）：**
```json
（保持原JSON不变）
```

---

### 4. GetData
主要数据检索方法。参数因数据集而异。

#### `GET /api/data?method=GetData&DatasetName={数据集}&{参数}&UserID=您的密钥&ResultFormat=JSON`

---

## 数据集特定参数与示例

### A. NIPA（国民收入和生产账户）

**参数：**
| 参数        | 类型   | 必填 | 描述                          |
|-------------|--------|------|-------------------------------|
| `TableName` | 字符串 | 是   | NIPA表标识符（如`T10101`）    |
| `Frequency` | 字符串 | 是   | `A`（年度）、`Q`（季度）、`M`（月度） |
| `Year`      | 字符串 | 是   | 逗号分隔年份，或`ALL`，或`X`表示最新 |

**示例（实际GDP百分比变化，季度，2022-2024）：**
```
https://apps.bea.gov/api/data?method=GetData&DatasetName=NIPA&TableName=T10101&Frequency=Q&Year=2022,2023,2024&UserID=您的密钥&ResultFormat=JSON
```

**示例（GDP水平，年度，所有年份）：**
```
https://apps.bea.gov/api/data?method=GetData&DatasetName=NIPA&TableName=T10105&Frequency=A&Year=ALL&UserID=您的密钥&ResultFormat=JSON
```

**响应：**
```json
（保持原JSON不变）
```

---

### B. Regional（州、县、都会区数据）

**参数：**
| 参数        | 类型   | 必填 | 描述                                     |
|-------------|--------|------|------------------------------------------|
| `TableName` | 字符串 | 是   | 区域表（如州GDP用`CAGDP1`）              |
| `LineCode`  | 整数   | 是   | 表内行号（指定数据系列）                 |
| `GeoFips`   | 字符串 | 是   | FIPS代码：`STATE`（全州）、`COUNTY`（全县）、`MSA`（全都会区）或具体代码（如加州`06000`） |
| `Year`      | 字符串 | 是   | 逗号分隔年份或`ALL`或`LAST5`             |

**常用区域表：**
| 表名     | 描述               |
|----------|--------------------|
| `CAGDP1` | 各州GDP摘要        |
| `CAGDP2` | 按成分划分的州GDP  |
| `CAGDP9` | 各州实际GDP        |
| `CAINC1` | 各州个人收入摘要   |
| `CAINC4` | 各州个人收入与就业 |
| `CAINC5N`| 按类型划分的州个人收入 |
| `SAINC1` | 州年度个人收入     |
| `SQINC1` | 州季度个人收入     |

**示例（各州GDP，全州，2020-2023）：**
```
https://apps.bea.gov/api/data?method=GetData&DatasetName=Regional&TableName=CAGDP1&LineCode=1&GeoFips=STATE&Year=2020,2021,2022,2023&UserID=您的密钥&ResultFormat=JSON
```

**示例（加利福尼亚州个人收入）：**
```
https://apps.bea.gov/api/data?method=GetData&DatasetName=Regional&TableName=CAINC1&LineCode=1&GeoFips=06000&Year=LAST5&UserID=您的密钥&ResultFormat=JSON
```

**响应：**
```json
（保持原JSON不变）
```

---

### C. ITA（国际交易账户/贸易）

**参数：**
| 参数            | 类型   | 必填 | 描述                          |
|-----------------|--------|------|-------------------------------|
| `Indicator`     | 字符串 | 是   | 指标代码（如商品余额`BalGds`）|
| `AreaOrCountry` | 字符串 | 是   | 国家代码：`AllCountries`、`China`、`Japan`等，或`All` |
| `Frequency`     | 字符串 | 是   | `A`、`Q`、`M`                 |
| `Year`          | 字符串 | 是   | 逗号分隔年份或`ALL`           |

**常用ITA指标：**
| 代码           | 描述               |
|----------------|--------------------|
| `BalGds`       | 商品余额           |
| `BalServ`      | 服务余额           |
| `BalGdsServ`   | 商品和服务余额     |
| `BalCurAcct`   | 经常账户余额       |
| `ExpGds`       | 商品出口           |
| `ImpGds`       | 商品进口           |
| `ExpServ`      | 服务出口           |
| `ImpServ`      | 服务进口           |

**示例（中美商品贸易余额，季度）：**
```
https://apps.bea.gov/api/data?method=GetData&DatasetName=ITA&Indicator=BalGds&AreaOrCountry=China&Frequency=Q&Year=2022,2023,2024&UserID=您的密钥&ResultFormat=JSON
```

---

### D. GDPbyIndustry（按行业划分的GDP）

**参数：**
| 参数        | 类型   | 必填 | 描述                          |
|-------------|--------|------|-------------------------------|
| `TableID`   | 整数   | 是   | 表编号（1-15）                |
| `Industry`  | 字符串 | 是   | 行业代码：`ALL`或具体代码（如农业`11`） |
| `Frequency` | 字符串 | 是   | `A`或`Q`                      |
| `Year`      | 字符串 | 是   | 逗号分隔年份或`ALL`           |

**常用表ID：**
| ID | 描述               |
|----|--------------------|
| 1  | 按行业划分的增加值 |
| 5  | 行业增加值占GDP百分比 |
| 6  | 按行业划分的实际增加值 |
| 7  | 行业实际增加值百分比变化 |

**示例（全行业增加值，年度）：**
```
https://apps.bea.gov/api/data?method=GetData&DatasetName=GDPbyIndustry&TableID=1&Industry=ALL&Frequency=A&Year=2020,2021,2022,2023&UserID=您的密钥&ResultFormat=JSON
```

---

### E. IIP（国际投资头寸）

**参数：**
| 参数               | 类型   | 必填 | 描述               |
|--------------------|--------|------|--------------------|
| `TypeOfInvestment` | 字符串 | 是   | `ALL`、`FinAssetsExclFinDeriv`等 |
| `Component`        | 字符串 | 是   | `ALL`或具体成分    |
| `Frequency`        | 字符串 | 是   | `A`或`Q`           |
| `Year`             | 字符串 | 是   | 逗号分隔年份或`ALL`|

**示例：**
```
https://apps.bea.gov/api/data?method=GetData&DatasetName=IIP&TypeOfInvestment=ALL&Component=ALL&Frequency=A&Year=2020,2021,2022,2023&UserID=您的密钥&ResultFormat=JSON
```

---

### F. FixedAssets（固定资产）

**参数：**
| 参数        | 类型   | 必填 | 描述               |
|-------------|--------|------|--------------------|
| `TableName` | 字符串 | 是   | 固定资产表ID       |
| `Year`      | 字符串 | 是   | 逗号分隔年份或`ALL`|

**示例：**
```
https://apps.bea.gov/api/data?method=GetData&DatasetName=FixedAssets&TableName=FAAt101&Year=ALL&UserID=您的密钥&ResultFormat=JSON
```

---

## 关键NIPA表参考

| 表名     | 描述                     |
|----------|--------------------------|
| `T10101` | 实际GDP百分比变化        |
| `T10105` | GDP（现价美元）          |
| `T10106` | 实际GDP（2017年链式美元）|
| `T10107` | GDP价格指数（百分比变化）|
| `T10110` | GDP价格平减指数          |
| `T20100` | 个人收入及其支配         |
| `T20301` | 按类型划分的个人消费支出 |
| `T20600` | 个人收入与支出           |
| `T30100` | 政府当期收支             |
| `T40100` | 国民账户中的涉外交易     |
| `T50100` | 按部门划分的储蓄与投资   |
| `T50105` | 储蓄与投资（实际值）     |
| `T60100` | 企业利润                 |
| `T70100` | 按主要产品类型划分的GDP  |
| `T11000` | 实际GDP（扩展明细）      |
| `T11200` | 对GDP增长的贡献          |

## GeoFips参考（常用）
| FIPS   | 州         |
|--------|------------|
| `00000`| 美国全国   |

| `01000` | 阿拉巴马州 |
| `06000` | 加利福尼亚州 |
| `12000` | 佛罗里达州 |
| `36000` | 纽约州 |
| `48000` | 得克萨斯州 |
| `STATE` | 所有州 |
| `COUNTY` | 所有县 |
| `MSA` | 所有大都市统计区 |

## 说明
- 响应中的DataValue是字符串类型，可能包含逗号（例如`"3,220,965,123"`），解析时需移除逗号。
- `Year=X`仅返回最新可用年份数据。
- `Year=LAST5`返回最近5年数据。
- 对于NIPA表格，结果包含每个表格的多个行项目（不同GDP组成部分对应不同行号）。
- `GetParameterValues`方法对发现各数据集的有效表名、行代码和指标代码至关重要。
- BEA在https://apps.bea.gov/iTable/提供批量下载文件用于交互操作。
- 季度数据的时间周期采用`2024Q1`、`2024Q2`等格式。
- 除非特别说明，所有货币值均以美元计。单位信息通过`CL_UNIT`和`UNIT_MULT`字段标识。
