# cBioPortal API

## 基础 URL
```
https://www.cbioportal.org/api
```

## 认证
公共实例无需认证。私有/机构实例（例如 `genie.cbioportal.org`）需通过 `Authorization: Bearer <token>` 请求头提供数据访问令牌。

## 通用请求头
```
Accept: application/json
Content-Type: application/json
```

## 通用查询参数

多数列表接口支持以下参数：

| 参数 | 类型 | 描述 | 默认值 |
|---|---|---|---|
| `projection` | string | 详情级别：`ID`, `SUMMARY`, `DETAILED`, `META` | `SUMMARY` |
| `pageNumber` | int | 从零开始的页码索引 | `0` |
| `pageSize` | int | 每页结果数量 | `10000000` |
| `sortBy` | string | 排序字段 | 视情况而定 |
| `direction` | string | `ASC` 升序或 `DESC` 降序 | `ASC` |

## 核心接口

### 研究项目

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/studies` | 列出所有癌症研究项目 |
| GET | `/studies/{studyId}` | 获取单个研究项目 |
| POST | `/studies/fetch` | 按ID批量获取研究项目 |

示例：
```
GET https://www.cbioportal.org/api/studies?projection=SUMMARY&pageSize=10
GET https://www.cbioportal.org/api/studies/brca_tcga
```

响应字段：`studyId`, `name`, `description`, `cancerTypeId`, `pmid`, `citation`, `allSampleCount`, `referenceGenome`, `publicStudy`, `importDate`

### 癌症类型

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/cancer-types` | 列出所有癌症类型 |
| GET | `/cancer-types/{cancerTypeId}` | 获取单个癌症类型 |

响应字段：`cancerTypeId`, `name`, `shortName`, `dedicatedColor`, `parent`

### 基因

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/genes` | 分页列出所有基因 |
| GET | `/genes/{geneId}` | 通过Hugo符号或Entrez ID获取基因 |
| GET | `/genes/{geneId}/aliases` | 获取基因别名 |
| POST | `/genes/fetch` | 批量获取基因 |

示例：
```
GET https://www.cbioportal.org/api/genes/TP53
```
响应：`{"entrezGeneId": 7157, "hugoGeneSymbol": "TP53", "type": "protein-coding"}`

### 分子谱

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/molecular-profiles` | 获取所有研究的分子谱 |
| GET | `/studies/{studyId}/molecular-profiles` | 获取某研究的分子谱 |
| GET | `/molecular-profiles/{molecularProfileId}` | 获取单个分子谱 |

分子谱类型 (`molecularAlterationType`)：`MUTATION_EXTENDED`, `COPY_NUMBER_ALTERATION`, `MRNA_EXPRESSION`, `PROTEIN_LEVEL`, `METHYLATION`

示例：
```
GET https://www.cbioportal.org/api/studies/brca_tcga/molecular-profiles
```

### 突变数据

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/molecular-profiles/{profileId}/mutations` | 获取分子谱中的突变 |
| POST | `/molecular-profiles/{profileId}/mutations/fetch` | 筛选突变查询 |
| POST | `/mutations/fetch` | 多分子谱突变批量获取 |

GET参数：
| 参数 | 类型 | 描述 |
|---|---|---|
| `sampleListId` | string | 样本列表ID (如 `brca_tcga_all`) |
| `entrezGeneId` | int | 按基因筛选 |
| `projection` | string | `SUMMARY`, `DETAILED`, `ID`, `META` |

示例 — 查询TCGA乳腺癌中的TP53突变：
```
GET https://www.cbioportal.org/api/molecular-profiles/brca_tcga_mutations/mutations?sampleListId=brca_tcga_all&entrezGeneId=7157&projection=DETAILED
```

批量基因查询的POST请求体：
```json
{
  "sampleListId": "brca_tcga_all",
  "entrezGeneIds": [7157, 672]
}
```

响应字段：`entrezGeneId`, `sampleId`, `patientId`, `proteinChange`, `mutationType`, `mutationStatus`, `chr`, `startPosition`, `endPosition`, `referenceAllele`, `variantAllele`, `variantType`, `ncbiBuild`, `tumorAltCount`, `tumorRefCount`

### 拷贝数变异

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/molecular-profiles/{profileId}/discrete-copy-number` | 获取CNA数据 |
| POST | `/molecular-profiles/{profileId}/discrete-copy-number/fetch` | 筛选CNA查询 |
| POST | `/discrete-copy-number/fetch` | 多分子谱CNA批量获取 |

### 分子数据（表达量、甲基化）

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/molecular-profiles/{profileId}/molecular-data` | 获取表达量/甲基化数据 |
| POST | `/molecular-data/fetch` | 多分子谱分子数据批量获取 |

### 临床数据

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/studies/{studyId}/clinical-data` | 获取研究的临床数据 |
| POST | `/clinical-data/fetch` | 多研究临床数据批量获取 |
| GET | `/studies/{studyId}/clinical-attributes` | 获取可用临床属性 |

GET参数：
| 参数 | 类型 | 描述 |
|---|---|---|
| `clinicalDataType` | string | `PATIENT` 患者级或 `SAMPLE` 样本级 |
| `attributeId` | string | 如 `OS_STATUS`, `OS_MONTHS`, `CANCER_TYPE` |

示例：
```
GET https://www.cbioportal.org/api/studies/brca_tcga/clinical-data?clinicalDataType=PATIENT&attributeId=OS_STATUS&projection=SUMMARY
```

### 患者与样本

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/studies/{studyId}/patients` | 获取研究中的患者 |
| GET | `/studies/{studyId}/samples` | 获取研究中的样本 |
| POST | `/patients/fetch` | 多研究患者批量获取 |
| POST | `/samples/fetch` | 多研究样本批量获取 |

### 样本列表

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/studies/{studyId}/sample-lists` | 获取预定义样本组 |
| GET | `/sample-lists/{sampleListId}` | 获取单个样本列表 |

### 基因面板

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/gene-panels` | 获取所有基因面板 |
| GET | `/gene-panels/{genePanelId}` | 获取含基因列表的面板详情 |
| POST | `/gene-panel-data/fetch` | 查询样本覆盖的基因面板 |

### 治疗方案

| 方法 | 接口路径 | 描述 |
|---|---|---|
| POST | `/treatments/patient` | 患者级治疗数据 |
| POST | `/treatments/sample` | 样本级治疗数据 |

### 系统状态

| 方法 | 接口路径 | 描述 |
|---|---|---|
| GET | `/health` | 服务健康检查 |
| GET | `/info` | 获取门户版本和数据库架构版本 |

## 典型工作流

1. **查找研究**：`GET /studies` — 浏览可用癌症研究，获取 `studyId`
2. **获取分子谱**：`GET /studies/{studyId}/molecular-profiles` — 查找分子谱ID (如 `brca_tcga_mutations`, `brca_tcga_gistic`)
3. **获取样本列表**：`GET /studies/{studyId}/sample-lists` — 查找样本列表ID (如 `brca_tcga_all`, `brca_tcga_sequenced`)
4. **查询数据**：使用分子谱ID和样本列表ID获取突变、CNA、表达量或临床数据

## 速率限制

未公开速率限制。请合理使用 — 避免高并发请求。如需批量数据，cBioPortal提供可下载数据集：https://docs.cbioportal.org/downloads/

## 使用技巧

- **研究ID** 遵循格式：`{癌症类型}_{数据源}` (如 `brca_tcga`, `luad_tcga`, `prad_mskcc_2017`)
- **分子谱ID** 扩展自研究ID：`{studyId}_mutations`, `{studyId}_gistic`, `{studyId}_rna_seq_v2_mrna`
- 使用 `projection=DETAILED` 获取包含嵌套对象的完整响应
- POST `/fetch` 接口支持跨多研究、多基因或多样本的批量查询 — 这是最灵活的查询方式
- 基因查询同时支持Hugo符号 (`TP53`) 和Entrez ID (`7157`)
- 交互式文档请访问Swagger UI：https://www.cbioportal.org/api/swagger-ui/index.html
