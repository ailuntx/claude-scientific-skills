# Eurostat API 参考文档

## 概述
Eurostat是欧盟的统计机构，为欧盟/欧洲经济区成员国及伙伴国提供经济、人口、贸易、劳动力、环境等领域的统计数据。该API遵循SDMX（统计数据和元数据交换）标准。

## 基础URL
```
https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1
```

另提供旧版JSON-stat端点：
```
https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0
```

## 认证
**无需API密钥**。该API完全开放且免费使用。

## 速率限制
- 无正式文档记录的速率限制
- Eurostat可能限制频繁爬取行为，建议自动化请求频率保持在每秒1-2次
- 大型数据集可能超时，请使用过滤器缩小响应规模

---

## 核心端点（SDMX 2.1 API）

### 1. 获取数据集（观测值）

```
GET /data/{datasetCode}/{filter}
```

| 参数 | 必填 | 说明 |
|-----------|----------|-------------|
| `datasetCode` | 是 | Eurostat数据集代码（如`nama_10_gdp`, `demo_pjan`） |
| `filter` | 否 | 点分隔的维度过滤器。使用`+`表示维度多选，`.`分隔维度，空段表示"全选" |

**查询参数：**
| 参数 | 必填 | 说明 |
|-----------|----------|-------------|
| `format` | 否 | `sdmx+json`（默认）, `sdmx+csv`, `sdmx+xml`, `TSV` |
| `startPeriod` | 否 | 起始年/季/月：`2015`, `2020-Q1`, `2020-01` |
| `endPeriod` | 否 | 结束年/季/月 |
| `detail` | 否 | `full`（默认）, `dataonly`, `serieskeysonly`, `nodata` |
| `lang` | 否 | `en`（默认）, `fr`, `de` |

**示例（德国和法国2018-2023年市场价GDP年度数据）：**
```
https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/nama_10_gdp/A.CP_MEUR.B1GQ.DE+FR?startPeriod=2018&endPeriod=2023
```

**示例（各国年度总人口）：**
```
https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/demo_pjan/A.NR.T.TOTAL.DE+FR+IT+ES?startPeriod=2015&endPeriod=2023
```

**示例（季节性调整月度失业率）：**
```
https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/une_rt_m/M.SA.TOTAL.PC_ACT.T.EA20?startPeriod=2023-01&endPeriod=2024-12
```

**示例（调和消费者物价指数月度通胀率）：**
```
https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/prc_hicp_mmor/M.RCH_A.CP00.DE+FR+IT?startPeriod=2023-01&endPeriod=2024-06&format=sdmx+json
```

### 2. 获取CSV格式数据集

在任何数据请求后附加`?format=sdmx+csv`可获得更易解析的扁平CSV响应。

**示例：**
```
https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/nama_10_gdp/A.CP_MEUR.B1GQ.DE+FR?startPeriod=2018&endPeriod=2023&format=sdmx+csv
```

### 3. 获取数据集结构（维度与代码列表）

```
GET /datastructure/ESTAT/{datasetCode}
```

**示例：**
```
https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/datastructure/ESTAT/nama_10_gdp
```

返回维度名称、位置及代码列表引用。

### 4. 获取代码列表（维度值）

```
GET /codelist/ESTAT/{codelistId}
```

**示例：**
```
https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/codelist/ESTAT/GEO
```

### 5. 搜索/浏览数据集（数据流）

```
GET /dataflow/ESTAT/all
```

返回所有可用Eurostat数据集。添加`?detail=allstubs`获取精简列表。

**示例：**
```
https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/dataflow/ESTAT/all?detail=allstubs
```

---

## JSON-stat API（简化替代方案）

### 基础URL
```
https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data
```

### 获取数据
```
GET /data/{datasetCode}?{dimension_filters}
```

维度通过查询参数传递，使用维度名称作为参数名。

**示例（德国和法国GDP）：**
```
https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/nama_10_gdp?geo=DE&geo=FR&unit=CP_MEUR&na_item=B1GQ&freq=A&time=2020&time=2021&time=2022&lang=en
```

**响应（JSON-stat格式）：**
```json
{
  "version": "2.0",
  "label": "GDP及主要构成（产出、支出和收入）",
  "id": ["freq", "unit", "na_item", "geo", "time"],
  "size": [1, 1, 1, 2, 3],
  "dimension": {
    "geo": {
      "label": "地理实体",
      "category": {
        "index": {"DE": 0, "FR": 1},
        "label": {"DE": "德国", "FR": "法国"}
      }
    },
    "time": {
      "label": "时间",
      "category": {
        "index": {"2020": 0, "2021": 1, "2022": 2}
      }
    }
  },
  "value": {3336010.0, 3601750.0, 3876810.0, 2310420.0, 2500870.0, 2639090.0},
  "status": {}
}
```

值为扁平数组，需根据维度尺寸重构。

---

## 常用数据集代码

| 代码 | 说明 |
|------|-------------|
| `nama_10_gdp` | GDP及主要构成 |
| `nama_10_pc` | 人均GDP |
| `namq_10_gdp` | 季度GDP |
| `demo_pjan` | 1月1日人口数 |
| `demo_gind` | 人口变动（出生、死亡、迁移） |
| `une_rt_m` | 失业率（月度） |
| `une_rt_a` | 失业率（年度） |
| `lfsi_emp_a` | 就业率（年度） |
| `prc_hicp_manr` | 调和消费者物价指数通胀（月度同比） |
| `prc_hicp_mmor` | 调和消费者物价指数通胀（月度环比） |
| `ext_lt_maineu` | 欧盟贸易（主要伙伴） |
| `ext_st_27_2020sitc` | 按SITC分类的国际贸易 |
| `bop_c6_q` | 国际收支（季度） |
| `gov_10dd_edpt1` | 政府赤字/盈余 |
| `gov_10a_main` | 政府收入、支出及主要总量 |
| `sts_inpr_m` | 工业生产（月度） |
| `tour_occ_nim` | 旅游（住宿过夜数） |
| `env_air_gge` | 温室气体排放 |
| `tec00114` | GDP增长率（百分比变化） |
| `tec00118` | 政府债务占GDP百分比 |

## 常用国家代码（ISO双字母，Eurostat使用大写）

`AT` 奥地利, `BE` 比利时, `BG` 保加利亚, `CY` 塞浦路斯, `CZ` 捷克, `DE` 德国, `DK` 丹麦, `EE` 爱沙尼亚, `EL` 希腊, `ES` 西班牙, `FI` 芬兰, `FR` 法国, `HR` 克罗地亚, `HU` 匈牙利, `IE` 爱尔兰, `IT` 意大利, `LT` 立陶宛, `LU` 卢森堡, `LV` 拉脱维亚, `MT` 马耳他, `NL` 荷兰, `PL` 波兰, `PT` 葡萄牙, `RO` 罗马尼亚, `SE` 瑞典, `SI` 斯洛文尼亚, `SK` 斯洛伐克

聚合区域：`EU27_2020`（欧盟27国）, `EA20`（欧元区20国）, `EA19`（欧元区19国）, `EEA30_2007`（欧洲经济区）

**注意：** 希腊在Eurostat中使用`EL`（而非`GR`）。

## SDMX JSON响应格式

```json
{
  "header": {
    "id": "...",
    "prepared": "2024-01-15T10:00:00"
  },
  "dataSets": [
    {
      "series": {
        "0:0:0:0": {
          "observations": {
            "0": [3336010.0],
            "1": [3601750.0]
          }
        }
      }
    }
  ],
  "structure": {
    "dimensions": {
      "series": [...],
      "observation": [...]
    }
  }
}
```

在SDMX+JSON中，维度值被编码为整数索引。`structure.dimensions`部分将索引映射到代码和标签。此格式紧凑但需索引查找。

## 注意事项
- 过滤器路径中的维度顺序取决于数据集结构，请先检查`/datastructure/ESTAT/{code}`
- 使用`+`选择单个维度的多个值（如`DE+FR+IT`）
- 留空维度段（连续点`..`）表示全选
- 推荐使用CSV格式（`?format=sdmx+csv`）便于解析——返回带标签列的扁平行数据
- JSON-stat API适合快速查询，而SDMX API功能更强大完整
- 数据集代码可通过https://ec.europa.eu/eurostat/databrowser/按主题浏览获取
- 大型无限制查询可能超时，请始终按国家和时间段进行过滤
