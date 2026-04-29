# 美国人口普查局 API 参考

## 概述
美国人口普查局 API 提供数百个数据集的访问权限，包括美国社区调查（ACS）、十年一度的人口普查、经济普查、人口估算等。它是美国人口统计、社会、经济和住房数据的主要来源。

## 基础 URL
```
https://api.census.gov/data
```

## 认证
- **API 密钥：必需（免费）。** 在 https://api.census.gov/data/key_signup.html 注册
- 作为查询参数传递：`&key=您的密钥`
- 无密钥请求限制为约 500 次/天。使用密钥后限制大幅提高。

## 速率限制
- 无密钥：约每天 500 次请求。
- 有密钥：文档标注的软限制为每天每 IP 500 次，但实际使用中密钥提供更高限额。
- 未记录正式每分钟速率限制；自动化请求建议控制在每秒数次以内。

---

## 核心数据集与 URL 模式

通用 URL 模式：
```
https://api.census.gov/data/{year}/{dataset}?get={variables}&for={geography}&key=您的密钥
```

### 主要数据集路径

| 数据集 | 路径片段 | 描述 |
|---------|-------------|-------------|
| ACS 5年详细表 | `acs/acs5` | 5年估算值，覆盖多数地理区域（2009年至今） |
| ACS 1年详细表 | `acs/acs1` | 1年估算值，仅限人口6.5万+区域（2005年至今） |
| ACS 5年主题表 | `acs/acs5/subject` | 预计算主题表格 |
| ACS 5年数据概况 | `acs/acs5/profile` | 社会/经济/住房概况 |
| 十年人口普查（2020） | `dec/dhc` | 人口与住房特征 |
| 十年人口普查（2020 PL） | `dec/pl` | 选区重划数据（PL 94-171） |
| 十年人口普查（2010） | `dec/sf1` | 摘要文件1 |
| 人口估算 | `pep/population` | 年度人口估算 |
| 经济普查 | `ecnbasic` | 经济普查（2017年、2022年） |
| 县商业模式 | `cbp` | 商业机构数量统计 |
| 年度商业调查 | `abscs` | 商业特征统计 |

---

## 核心端点

### 1. ACS 5年估算（最常用）

```
GET /data/{year}/acs/acs5?get={variables}&for={geography}&key=您的密钥
```

| 参数 | 必填 | 描述 |
|-----------|----------|-------------|
| `get`     | 是 | 逗号分隔的变量名（如 `NAME,B01001_001E`） |
| `for`     | 是 | 目标地理区域（如 `state:*`, `county:*`, `tract:*`） |
| `in`      | 部分场景 | 州以下层级的上级地理区域 |
| `key`     | 是 | 您的 API 密钥 |

**示例（2022年ACS 5年数据中所有州的总人口）：**
```
https://api.census.gov/data/2022/acs/acs5?get=NAME,B01001_001E&for=state:*&key=YOUR_KEY
```

**示例（加利福尼亚州所有县的家庭收入中位数）：**
```
https://api.census.gov/data/2022/acs/acs5?get=NAME,B19013_001E&for=county:*&in=state:06&key=YOUR_KEY
```

**示例（特定普查区的种族人口分布）：**
```
https://api.census.gov/data/2022/acs/acs5?get=NAME,B02001_001E,B02001_002E,B02001_003E&for=tract:000100&in=state:06&in=county:075&key=YOUR_KEY
```

**响应（JSON数组的数组，首行为表头）：**
```json
[
  ["NAME", "B01001_001E", "state"],
  ["Alabama", "5024279", "01"],
  ["Alaska", "733391", "02"],
  ["Arizona", "7151502", "04"]
]
```

### 2. ACS 1年估算

```
GET /data/{year}/acs/acs1?get={variables}&for={geography}&key=YOUR_KEY
```
参数同ACS 5年。仅适用于人口6.5万+区域。

**示例（所有州的贫困率）：**
```
https://api.census.gov/data/2022/acs/acs1?get=NAME,B17001_001E,B17001_002E&for=state:*&key=YOUR_KEY
```

### 3. ACS 数据概况

```
GET /data/{year}/acs/acs5/profile?get={variables}&for={geography}&key=YOUR_KEY
```
使用带预计算百分比的`DP`前缀变量。

**示例（教育程度概况）：**
```
https://api.census.gov/data/2022/acs/acs5/profile?get=NAME,DP02_0068PE&for=state:*&key=YOUR_KEY
```

### 4. 2020年十年人口普查

```
GET /data/2020/dec/dhc?get={variables}&for={geography}&key=YOUR_KEY
```

**示例（2020年人口普查各州总人口）：**
```
https://api.census.gov/data/2020/dec/dhc?get=NAME,P1_001N&for=state:*&key=YOUR_KEY
```

### 5. 2010年十年人口普查

```
GET /data/2010/dec/sf1?get={variables}&for={geography}&key=YOUR_KEY
```

**示例：**
```
https://api.census.gov/data/2010/dec/sf1?get=NAME,P001001&for=state:*&key=YOUR_KEY
```

### 6. 发现可用变量

```
GET /data/{year}/{dataset}/variables.json
```

**示例：**
```
https://api.census.gov/data/2022/acs/acs5/variables.json
```
返回包含所有变量标签和概念的JSON对象。

### 7. 发现可用地理区域

```
GET /data/{year}/{dataset}/geography.json
```

**示例：**
```
https://api.census.gov/data/2022/acs/acs5/geography.json
```

### 8. 列出可用数据集

```
GET /data.json
```
返回所有可用数据集及其标题、年份和API端点。

---

## 地理区域语法

| 层级 | `for` 语法 | `in` 要求 |
|-------|-------------|-----------------|
| 国家 | `us:1` 或 `us:*` | 无 |
| 州 | `state:06` 或 `state:*` | 无 |
| 县 | `county:075` 或 `county:*` | `in=state:06`（查询全部时可省略） |
| 县分区 | `county subdivision:*` | `in=state:XX&in=county:YYY` |
| 普查区 | `tract:*` | `in=state:XX&in=county:YYY` |
| 区块组 | `block group:*` | `in=state:XX&in=county:YYY&in=tract:ZZZZZZ` |
| 地方（城市） | `place:*` | `in=state:XX` |
| 都会区（CBSA） | `metropolitan statistical area/micropolitan statistical area:*` | 无 |
| 邮编制表区 | `zip code tabulation area:*` | 无 |

州 FIPS 代码：`01`=AL, `02`=AK, `04`=AZ, `05`=AR, `06`=CA, `08`=CO, `09`=CT, `10`=DE, `11`=DC, `12`=FL, `13`=GA, `15`=HI, `16`=ID, `17`=IL, `18`=IN, `19`=IA, `20`=KS, `21`=KY, `22`=LA, `23`=ME, `24`=MD, `25`=MA, `26`=MI, `27`=MN, `28`=MS, `29`=MO, `30`=MT, `31`=NE, `32`=NV, `33`=NH, `34`=NJ, `35`=NM, `36`=NY, `37`=NC, `38`=ND, `39`=OH, `40`=OK, `41`=OR, `42`=PA, `44`=RI, `45`=SC, `46`=SD, `47`=TN, `48`=TX, `49`=UT, `50`=VT, `51`=VA, `53`=WA, `54`=WV, `55`=WI, `56`=WY

---

## 常用变量代码

### ACS 详细表（B表）
| 变量 | 描述 |
|----------|-------------|
| `B01001_001E` | 总人口 |
| `B01002_001E` | 年龄中位数 |
| `B02001_001E` | 总人口（种族） |
| `B02001_002E` | 仅白人 |
| `B02001_003E` | 仅黑人或非裔美国人 |
| `B03001_003E` | 西班牙裔或拉丁裔 |
| `B19013_001E` | 家庭收入中位数 |
| `B19001_001E` | 家庭收入（总计，用于分布） |
| `B25077_001E` | 房屋价值中位数 |
| `B25064_001E` | 租金中位数 |
| `B17001_001E` | 贫困状态（总计） |
| `B17001_002E` | 贫困状态（低于贫困线） |
| `B15003_022E` | 学士学位 |
| `B15003_023E` | 硕士学位 |
| `B15003_025E` | 博士学位 |
| `B23025_005E` | 失业人口（民用劳动力） |
| `B25001_001E` | 住房单元总数 |
| `B08301_001E` | 通勤方式（总计） |

变量命名：`B{表}_{序列}E`为估算值，`B{表}_{序列}M`为误差范围。

### ACS 数据概况变量（DP表）
| 变量 | 描述 |
|----------|-------------|
| `DP02_0068PE` | 拥有学士学位或更高比例 |
| `DP03_0062E` | 家庭收入中位数 |
| `DP03_0128PE` | 低于贫困线比例 |
| `DP04_0089E` | 房屋价值中位数 |
| `DP05_0001E` | 总人口 |

### 2020年十年普查（DHC）
| 变量 | 描述 |
|----------|-------------|
| `P1_001N` | 总人口 |
| `P1_003N` | 仅白人 |
| `P1_004N` | 仅黑人或非裔美国人 |
| `H1_001N` | 住房单元总数 |
| `H1_002N` | 已占用住房单元 |

---

## 响应格式
所有数据响应均为**JSON数组的数组**。首数组始终为列标题，后续数组为数据行。

```json
[
  ["NAME", "B01001_001E", "B19013_001E", "state", "county"],
  ["Los Angeles County, California", "10014009", "73538", "06", "037"],
  ["San Diego County, California", "3298634", "85750", "06", "073"]
]
```

- 值为字符串格式（包括数值）。
- 缺失数据可能显示为 `null`、`"-"` 或 `"N"`。
- 注释值：`"-"`（样本不足），`"N"`（不可用），`"(X)"`（不适用）。

## 注意事项
- 始终在`get`参数中包含`NAME`以获取可读地理标签。
- `E`后缀表示"估算值"；`M`后缀表示误差范围（如`B19013_001M`）。
- 变量发现：访问 https://api.census.gov/data/{year}/acs/acs5/variables.html 浏览可搜索表格。
- ACS 5年估算覆盖全地理区域但时效性较低；1年估算仅限大区域但更新更快。
- 组端点：`?get=group(B01001)` 可获取表组内所有变量。
