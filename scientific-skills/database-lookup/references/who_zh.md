# WHO 全球健康观察站（GHO）API 参考

## 概述
WHO 全球健康观察站（GHO）OData API 提供对 194 个 WHO 成员国健康统计数据的访问。涵盖 2000 多项指标，包括预期寿命、疾病负担、死亡率、免疫覆盖率、卫生人力、空气污染、水/卫生设施以及可持续发展目标（SDG）健康指标。

## 基础 URL
```
https://ghoapi.azureedge.net/api
```

## 认证
**无需 API 密钥。** 该 API 完全开放且免费使用。

## 速率限制
- 无正式速率限制文档
- API 通过 Azure CDN 提供服务，可良好处理中等负载
- 自动化请求需保持合理频率；建议每秒 1-2 次

---

## 核心端点

API 遵循 OData v4 协议。支持标准 OData 查询参数：`$filter`、`$select`、`$orderby`、`$top`、`$skip`、`$count`。

### 1. 列出所有指标

```
GET /Indicator
```

**示例：**
```
https://ghoapi.azureedge.net/api/Indicator
```

**响应：**
```json
{
  "@odata.context": "...",
  "value": [
    {
      "IndicatorCode": "WHOSIS_000001",
      "IndicatorName": "Life expectancy at birth (years)",
      "Language": "EN"
    },
    {
      "IndicatorCode": "WHOSIS_000002",
      "IndicatorName": "Healthy life expectancy (HALE) at birth (years)",
      "Language": "EN"
    },
    {
      "IndicatorCode": "WHS4_100",
      "IndicatorName": "Measles (MCV1) immunization coverage among 1-year-olds (%)",
      "Language": "EN"
    }
  ]
}
```

### 2. 获取特定指标数据

```
GET /{IndicatorCode}
```

**示例（出生时预期寿命）：**
```
https://ghoapi.azureedge.net/api/WHOSIS_000001
```

**响应：**
```json
{
  "@odata.context": "...",
  "value": [
    {
      "Id": 12345,
      "IndicatorCode": "WHOSIS_000001",
      "SpatialDim": "USA",
      "SpatialDimType": "COUNTRY",
      "TimeDim": 2019,
      "TimeDimType": "YEAR",
      "Dim1": "SEX",
      "Dim1Type": "BTSX",
      "Dim2": null,
      "Dim2Type": null,
      "Dim3": null,
      "Dim3Type": null,
      "DataSourceDim": null,
      "Value": "78.5",
      "NumericValue": 78.5,
      "Low": 78.2,
      "High": 78.8,
      "Comments": "",
      "Date": "2024-01-15T00:00:00+00:00",
      "TimeDimensionValue": "2019",
      "TimeDimensionBegin": "2019-01-01T00:00:00+00:00",
      "TimeDimensionEnd": "2019-12-31T00:00:00+00:00"
    }
  ]
}
```

### 3. 按国家筛选

使用 OData `$filter` 按国家（SpatialDim）筛选结果。

**示例（仅限美国的预期寿命）：**
```
https://ghoapi.azureedge.net/api/WHOSIS_000001?$filter=SpatialDim eq 'USA'
```

**示例（多国预期寿命）：**
```
https://ghoapi.azureedge.net/api/WHOSIS_000001?$filter=SpatialDim eq 'USA' or SpatialDim eq 'GBR' or SpatialDim eq 'JPN'
```

### 4. 按年份筛选

**示例（2019年预期寿命）：**
```
https://ghoapi.azureedge.net/api/WHOSIS_000001?$filter=TimeDim eq 2019
```

**示例（2015年起美国预期寿命）：**
```
https://ghoapi.azureedge.net/api/WHOSIS_000001?$filter=SpatialDim eq 'USA' and TimeDim ge 2015
```

### 5. 按性别/维度筛选

**示例（两性预期寿命，美国，2015年+）：**
```
https://ghoapi.azureedge.net/api/WHOSIS_000001?$filter=SpatialDim eq 'USA' and TimeDim ge 2015 and Dim1 eq 'BTSX'
```

Dim1 性别取值：`BTSX`（两性）、`MLE`（男性）、`FMLE`（女性）。

### 6. 分页与限制

**示例（前10条结果）：**
```
https://ghoapi.azureedge.net/api/WHOSIS_000001?$top=10
```

**示例（跳过前100条，获取后续50条）：**
```
https://ghoapi.azureedge.net/api/WHOSIS_000001?$top=50&$skip=100
```

### 7. 选择特定字段

```
https://ghoapi.azureedge.net/api/WHOSIS_000001?$filter=SpatialDim eq 'USA'&$select=SpatialDim,TimeDim,NumericValue,Dim1
```

### 8. 结果排序

```
https://ghoapi.azureedge.net/api/WHOSIS_000001?$filter=SpatialDim eq 'USA'&$orderby=TimeDim desc
```

### 9. 列出维度值

```
GET /DIMENSION/{DimensionType}/DimensionValues
```

**示例（列出所有国家）：**
```
https://ghoapi.azureedge.net/api/DIMENSION/COUNTRY/DimensionValues
```

**示例（列出所有区域）：**
```
https://ghoapi.azureedge.net/api/DIMENSION/REGION/DimensionValues
```

**示例（列出性别维度值）：**
```
https://ghoapi.azureedge.net/api/DIMENSION/SEX/DimensionValues
```

---

## 常用指标代码

### 预期寿命与死亡率
| 代码 | 描述 |
|------|-------------|
| `WHOSIS_000001` | 出生时预期寿命（岁） |
| `WHOSIS_000002` | 出生时健康预期寿命（HALE）（岁） |
| `WHOSIS_000004` | 新生儿死亡率（每1000例活产） |
| `MDG_0000000001` | 婴儿死亡率（每1000例活产） |
| `MDG_0000000007` | 五岁以下儿童死亡率（每1000例活产） |
| `MORT_MATERNALNUM` | 孕产妇死亡人数 |
| `MDG_0000000026` | 孕产妇死亡率（每10万例活产） |
| `NCDMORT3070` | 30-70岁非传染性疾病死亡概率 |
| `LIFE_0000000029` | 成人死亡率（15-60岁死亡概率） |

### 传染性疾病
| 代码 | 描述 |
|------|-------------|
| `WHS3_49` | HIV感染率（15-49岁人口占比） |
| `MDG_0000000029` | 结核病发病率（每10万人） |
| `MALARIA_EST_INCIDENCE` | 疟疾发病率（每1000风险人口） |
| `WHS3_62` | 新发HIV感染（每1000未感染人口） |

### 免疫接种
| 代码 | 描述 |
|------|-------------|
| `WHS4_100` | 麻疹（MCV1）接种率（1岁儿童占比） |
| `WHS4_117` | 百白破三联疫苗（DTP3）接种率（1岁儿童占比） |
| `WHS4_129` | 乙肝疫苗（HepB3）接种率（%） |
| `WHS4_543` | 脊髓灰质炎（Pol3）接种率（1岁儿童占比） |

### 非传染性疾病与风险因素
| 代码 | 描述 |
|------|-------------|
| `NCD_BMI_30A` | 肥胖患病率（BMI≥30，年龄标准化） |
| `NCD_HYP_PREVALENCE_A` | 高血压患病率 |
| `NCD_GLUC_04` | 糖尿病患病率（人口占比） |
| `M_Est_smk_curr_std` | 当前烟草使用流行率 |
| `SA_0000001462` | 人均酒精消费总量（升） |

### 卫生系统
| 代码 | 描述 |
|------|-------------|
| `HWF_0001` | 医生数（每万人口） |
| `HWF_0006` | 护理和助产人员（每万人口） |
| `WHS7_104` | 医院床位（每万人口） |
| `GHED_CHE_pc_US_SHA2011` | 人均卫生支出（美元） |
| `UHC_INDEX_REPORTED` | 全民健康覆盖服务指数 |

### 环境卫生
| 代码 | 描述 |
|------|-------------|
| `SDGPM25` | PM2.5空气污染年均暴露量（微克/立方米） |
| `WSH_SANITATION_SAFELY_MANAGED` | 安全管理的卫生设施服务（%） |
| `WSH_WATER_SAFELY_MANAGED` | 安全管理的饮用水服务（%） |

---

## 国家代码（ISO 3166-1 alpha-3）

GHO API 在 `SpatialDim` 字段使用 **ISO 三字母代码** 表示国家。

`USA`（美国）、`GBR`（英国）、`DEU`（德国）、`FRA`（法国）、`JPN`（日本）、`CHN`（中国）、`IND`（印度）、`BRA`（巴西）、`ZAF`（南非）、`NGA`（尼日利亚）、`AUS`（澳大利亚）、`CAN`（加拿大）、`KOR`（韩国）、`MEX`（墨西哥）、`RUS`（俄罗斯）

WHO区域：`AFR`（非洲）、`AMR`（美洲）、`SEAR`（东南亚）、`EUR`（欧洲）、`EMR`（东地中海）、`WPR`（西太平洋）、`GLOBAL`（全球）

---

## 响应格式
所有响应均为遵循 OData v
