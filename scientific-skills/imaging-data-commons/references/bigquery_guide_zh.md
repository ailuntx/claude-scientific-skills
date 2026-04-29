# IDC BigQuery 指南

**测试版本：** IDC 数据版本 v23

对于大多数查询和下载，请使用 `idc-index`（参见主 SKILL.md）。本指南涵盖 BigQuery，适用于需要完整 DICOM 元数据或复杂连接的高级用例。

## 前提条件

**要求：**
1. Google 账户
2. 已启用结算功能的 Google Cloud 项目（前 1 TB/月免费）
3. `google-cloud-bigquery` Python 包或 BigQuery 控制台访问权限

**身份验证设置：**
```bash
# 安装 Google Cloud SDK，然后运行：
gcloud auth application-default login
```

## 何时使用 BigQuery

当你需要以下内容时，请使用 BigQuery 而不是 `idc-index`：
- 完整的 DICOM 元数据（全部 4000+ 个标签，不仅仅是 idc-index 中的约 50 个）
- 跨临床数据表的复杂连接
- DICOM 序列属性（嵌套结构）
- 对 idc-index 迷你索引中不存在的字段进行查询
- 私有 DICOM 元素（OtherElements 列中特定于供应商的标签）
- **来自 DICOM 分割对象的逐段详细信息** — `idc-index` 的 `seg_index` 提供系列级元数据，但不提供单个段的解剖代码；请使用 `segmentations` BigQuery 表按结构名称进行查询
- **来自 DICOM 结构化报告的定量测量** — 无需下载和解析 SR 文件即可获取影像组学特征（体积、直径、形状描述符）；idc-index 中没有对应项
- **来自 DICOM 结构化报告的定性测量** — 无需解析 SR 文件即可获取编码评估（恶性评级、纹理、边缘）；idc-index 中没有对应项

## 在 BigQuery 中访问 IDC

### 数据集结构

所有 IDC 表都位于 `bigquery-public-data` BigQuery 项目中。

**当前版本（推荐用于探索）：**
- `bigquery-public-data.idc_current.*`
- `bigquery-public-data.idc_current_clinical.*`

**版本化数据集（推荐用于可重现性）：**

- `bigquery-public-data.idc_v{IDC 版本}.*`
- `bigquery-public-data.idc_v{IDC 版本}_clinical.*`

始终使用版本化数据集进行可重现的研究！

## 关键表

### dicom_all
主表，将完整的 DICOM 元数据与 IDC 特定列（collection_id、gcs_url、license）连接起来。包含来自 `dicom_metadata` 的所有 DICOM 标签以及集合和管理元数据。请参阅 [dicom_all.sql](https://github.com/ImagingDataCommons/etl_flow/blob/master/bq/generate_tables_and_views/derived_tables/BQ_Table_Building/derived_data_views/sql/dicom_all.sql) 了解具体推导过程。

```sql
SELECT 
  collection_id,
  PatientID,
  StudyInstanceUID, 
  SeriesInstanceUID,
  Modality,
  BodyPartExamined,
  SeriesDescription,
  gcs_url,
  license_short_name
FROM `bigquery-public-data.idc_current.dicom_all`
WHERE Modality = 'CT'
  AND BodyPartExamined = 'CHEST'
LIMIT 10
```

### 派生表

这些表是从 DICOM 对象（分割和结构化报告）中派生的，在 **idc-index 中没有对应项**。使用它们无需下载 DICOM 文件即可查询每段解剖结构、影像组学特征和定性评估。

**segmentations** — DICOM SEG 对象中每个段一行。允许按解剖结构名称或 DICOM 编码概念进行搜索。`idc-index` 的 `seg_index` 提供系列级元数据；此表提供每段详细信息。

**measurement_groups** — 每个 SR TID1500 测量组一行。定量和定性测量的父分组；将测量与分段和源图像相关联。

**quantitative_measurements** — SR TID1500 组中每个数值测量一行。包含从 DICOM SR 中提取的影像组学特征（体积、直径、形状描述符、纹理），无需下载或解析 SR 文件。

**qualitative_measurements** — SR TID1500 组中每个编码评估一行。包含使用编码概念值评估的发现（恶性可能性、纹理、边缘类型）。

参见下文[派生表：详细文档](#派生表详细文档)部分，了解模式、列描述和查询示例。

### 集合元数据

**original_collections_metadata** — 集合级描述

```sql
SELECT
  collection_id,
  CancerTypes,
  TumorLocations,
  Subjects,
  src.source_doi,
  src.ImageTypes,
  src.license.license_short_name
FROM `bigquery-public-data.idc_current.original_collections_metadata`,
UNNEST(Sources) AS src
WHERE CancerTypes LIKE '%Lung%'
```

## 常见查询模式

### 按条件查找集合

```sql
SELECT 
  collection_id,
  COUNT(DISTINCT PatientID) as patient_count,
  COUNT(DISTINCT SeriesInstanceUID) as series_count,
  ARRAY_AGG(DISTINCT Modality) as modalities
FROM `bigquery-public-data.idc_current.dicom_all`
WHERE BodyPartExamined LIKE '%BRAIN%'
GROUP BY collection_id
HAVING patient_count > 50
ORDER BY patient_count DESC
```

### 获取下载 URL

```sql
SELECT
  SeriesInstanceUID,
  gcs_url
FROM `bigquery-public-data.idc_current.dicom_all`
WHERE collection_id = 'rider_pilot'
  AND Modality = 'CT'
```

### 查找包含多种模态的研究

```sql
SELECT
  StudyInstanceUID,
  ARRAY_AGG(DISTINCT Modality) as modalities,
  COUNT(DISTINCT SeriesInstanceUID) as series_count
FROM `bigquery-public-data.idc_current.dicom_all`
GROUP BY StudyInstanceUID
HAVING ARRAY_LENGTH(ARRAY_AGG(DISTINCT Modality)) > 1
LIMIT 100
```

### 许可证过滤

```sql
SELECT
  collection_id,
  license_short_name,
  COUNT(*) as instance_count
FROM `bigquery-public-data.idc_current.dicom_all`
WHERE license_short_name = 'CC BY 4.0'
GROUP BY collection_id, license_short_name
```

### 查找带有源图像的分割

```sql
SELECT
  src.collection_id,
  seg.SeriesInstanceUID as seg_series,
  seg.SegmentedPropertyType,
  src.SeriesInstanceUID as source_series,
  src.Modality as source_modality
FROM `bigquery-public-data.idc_current.segmentations` seg
JOIN `bigquery-public-data.idc_current.dicom_all` src
  ON seg.segmented_SeriesInstanceUID = src.SeriesInstanceUID
WHERE src.collection_id = 'qin_prostate_repeatability'
LIMIT 10
```

## 派生表：详细文档

### segmentations

DICOM 分割对象中每个段一行。与 `idc-index` 的 `seg_index`（每个 SEG 系列一行）不同，此表暴露每个标记区域，以便你可以按解剖结构或发现类型进行搜索。

**关键列：**

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `SeriesInstanceUID` | STRING | SEG 系列 UID |
| `SOPInstanceUID` | STRING | SEG 实例 UID |
| `PatientID` | STRING | 患者标识符 |
| `StudyInstanceUID` | STRING | 研究 UID |
| `SegmentNumber` | INTEGER | SEG 中的段索引（从 1 开始） |
| `SegmentedPropertyCategory` | RECORD | 编码类别（例如，“解剖结构”、“形态改变的结构”） |
| `SegmentedPropertyType` | RECORD | 特定结构（例如，“肝脏”、“肾脏”、“肿瘤”） |
| `AnatomicRegion` | RECORD | 可选的解剖区域修饰符 |
| `SegmentAlgorithmType` | STRING | AUTOMATIC、SEMIAUTOMATIC 或 MANUAL |
| `SegmentAlgorithmName` | STRING (REPEATED) | 算法名称数组（例如，["TotalSegmentator"]） |
| `TrackingUID` | STRING | 将段链接到 SR 测量值 |
| `TrackingID` | STRING | 人类可读的跟踪标签 |
| `segmented_SeriesInstanceUID` | STRING | 源图像系列 UID — 连接到 `dicom_all` 以获取集合/模态 |
| `viewer_url` | STRING | SEG 的直接 IDC 查看器链接 |

`SegmentedPropertyCategory` 和 `SegmentedPropertyType` 是 RECORD 类型，具有子字段 `CodeValue`、`CodingSchemeDesignator` 和 `CodeMeaning`。使用 `.CodeMeaning` 进行人类可读的过滤。

**idc-index 缺口：** idc-index 中的 `seg_index` 有 `total_segments`、`AlgorithmName` 和聚合代码，但不暴露每行单个段解剖信息。使用此 BigQuery 表查找包含特定结构的 SEG 系列（例如，所有包含“肝脏”段的系列）。

**发现 IDC 中哪些结构被分割：**

```sql
SELECT
  SegmentedPropertyCategory.CodeMeaning AS category,
  SegmentedPropertyType.CodeMeaning AS structure,
  SegmentAlgorithmType,
  COUNT(DISTINCT SeriesInstanceUID) AS seg_series_count
FROM `bigquery-public-data.idc_current.segmentations`
GROUP BY 1, 2, 3
ORDER BY seg_series_count DESC
LIMIT 20
```

**查找包含特定结构的所有 SEG 系列，以及源图像上下文：**

```sql
SELECT
  seg.SeriesInstanceUID AS seg_series,
  seg.SegmentNumber,
  seg.SegmentedPropertyType.CodeMeaning AS structure,
  seg.SegmentAlgorithmType,
  seg.SegmentAlgorithmName,
  img.collection_id,
  img.PatientID,
  img.Modality,
  seg.viewer_url
FROM `bigquery-public-data.idc_current.segmentations` seg
JOIN `bigquery-public-data.idc_current.dicom_all` img
  ON seg.segmented_SeriesInstanceUID = img.SeriesInstanceUID
WHERE seg.SegmentedPropertyType.CodeMeaning = 'Liver'
  AND seg.SegmentAlgorithmType = 'AUTOMATIC'
LIMIT 20
```

**查找集合中存在所有段类型：**

```sql
SELECT
  seg.SegmentedPropertyType.CodeMeaning AS structure,
  seg.SegmentAlgorithmType,
  COUNT(DISTINCT seg.SeriesInstanceUID) AS seg_series_count
FROM `bigquery-public-data.idc_current.segmentations` seg
JOIN `bigquery-public-data.idc_current.dicom_all` img
  ON seg.segmented_SeriesInstanceUID = img.SeriesInstanceUID
WHERE img.collection_id = 'nlst'
GROUP BY 1, 2
ORDER BY seg_series_count DESC
```

**使用 TrackingUID 将段链接到 SR 测量值：**

```sql
-- 查找具有相应 SR 测量值的段
SELECT
  seg.SeriesInstanceUID AS seg_series,
  seg.SegmentNumber,
  seg.SegmentedPropertyType.CodeMeaning AS structure,
  qm.Quantity.CodeMeaning AS measurement,
  ROUND(CAST(qm.Value AS FLOAT64), 2) AS value,
  qm.Units.CodeMeaning AS units
FROM `bigquery-public-data.idc_current.segmentations` seg
JOIN `bigquery-public-data.idc_current.quantitative_measurements` qm
  ON seg.SeriesInstanceUID = qm.segmentationSeriesUID
  AND seg.SegmentNumber = qm.segmentationSegmentNumber
WHERE seg.SegmentedPropertyType.CodeMeaning = 'Neoplasm'
  AND qm.Quantity.CodeMeaning = 'Volume from Voxel Summation'
LIMIT 10
```

---

### quantitative_measurements

DICOM SR TID1500 测量报告中每个数值测量一行。包含影像组学特征（形状、强度、纹理）和临床测量值（体积、直径、SUV）。这些测量值是从 SR 中预先提取的——无需下载或 DICOM 解析。

**idc-index 中没有对应项。** 此表只能通过 BigQuery 访问。

**关键列：**

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `SOPInstanceUID` | STRING | SR 实例 UID |
| `SeriesInstanceUID` | STRING | SR 系列 UID — 连接到 `dicom_all` 以获取集合/模态 |
| `SeriesDescription` | STRING | SR 系列描述（例如，“TotalSegmentator(v1.5.6) shape Measurements”） |
| `PatientID` | STRING | 患者标识符 |
| `measurementGroup_number` | INTEGER | SR 中的组索引（从 0 开始）；与 `measurement_groups` 和 `qualitative_measurements` 的连接键 |
| `Quantity` | RECORD | 测量的内容 — `CodeValue`、`CodingSchemeDesignator`、`CodeMeaning`（例如，“Volume from Voxel Summation”） |
| `Value` | NUMERIC | 数值测量值 |
| `Units` | RECORD | 单位 — `CodeMeaning`（例如，“cubic millimeter”、“no units”、“Hounsfield Unit”） |
| `derivationModifier` | RECORD | 值的推导方式（例如，“Mean”、“Minimum”、“Maximum”） |
| `lateralityModifier` | RECORD | 左右侧修饰符 |
| `finding` | RECORD | 测量的发现 — `CodeMeaning`（例如，“Nodule”、“Organ”、“Anatomical Structure”） |
| `findingSite` | RECORD | 发现的位置 — `CodeMeaning`（例如，“Liver”、“Esophagus”、“Lung”） |
| `trackingIdentifier` | STRING | 人类可读的跟踪标签（例如，“Nodule 1”、“Measurements group 26”） |
| `trackingUniqueIdentifier` | STRING | 跟踪 UID — 链接回 `segmentations.TrackingUID` |
| `segmentationInstanceUID` | STRING | 引用的 SEG 对象的 SOPInstanceUID |
| `segmentationSeriesUID` | STRING | 引用的 SEG 对象的 SeriesInstanceUID |
| `segmentationSegmentNumber` | INTEGER | SEG 中的段编号 — 连接到 `segmentations.SegmentNumber` |
| `sourceSegmentedSeriesUID` | STRING | 源图像系列 — 连接到 `dicom_all.SeriesInstanceUID` |

**发现可用的测量类型：**

```sql
SELECT
  Quantity.CodeMeaning AS measurement,
  Units.CodeMeaning AS units,
  COUNT(*) AS measurement_count,
  COUNT(DISTINCT SeriesInstanceUID) AS sr_series_count
FROM `bigquery-public-data.idc_current.quantitative_measurements`
GROUP BY 1, 2
ORDER BY measurement_count DESC
LIMIT 20
```

**查询特定结构的测量值（例如，跨集合的肝脏体积）：**

```sql
SELECT
  qm.PatientID,
  ROUND(CAST(qm.Value AS FLOAT64) / 1000, 1) AS volume_cm3,
  img.collection_id,
  qm.segmentationSeriesUID
FROM `bigquery-public-data.idc_current.quantitative_measurements` qm
JOIN `bigquery-public-data.idc_current.dicom_all` img
  ON qm.sourceSegmentedSeriesUID = img.SeriesInstanceUID
WHERE qm.Quantity.CodeMeaning = 'Volume from Voxel Summation'
  AND qm.findingSite.CodeMeaning = 'Liver'
ORDER BY volume_cm3 DESC
LIMIT 20
```

**检索特定患者和发现的所有测量值：**

```sql
SELECT
  qm.measurementGroup_number,
  qm.finding.CodeMeaning AS finding,
  qm.findingSite.CodeMeaning AS finding_site,
  qm.lateralityModifier.CodeMeaning AS laterality,
  qm.Quantity.CodeMeaning AS feature,
  ROUND(CAST(qm.Value AS FLOAT64), 3) AS value,
  qm.Units.CodeMeaning AS units
FROM `bigquery-public-data.idc_current.quantitative_measurements` qm
WHERE qm.PatientID = 'LIDC-IDRI-0001'
  AND qm.finding.CodeMeaning = 'Nodule'
ORDER BY qm.measurementGroup_number, qm.Quantity.CodeMeaning
```

---

### qualitative_measurements

DICOM SR TID1500 测量报告中每个编码评估一行。这些记录的不是数值，而是使用编码概念对（例如，数量="Malignancy"，值="4 out of 5 (Moderately Suspicious for Cancer)"）进行评估的特征。

**idc-index 中没有对应项。** 此表只能通过 BigQuery 访问。

**关键列：**

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| `SOPInstanceUID` | STRING | SR 实例 UID |
| `SeriesInstanceUID` | STRING | SR 系列 UID — 连接到 `dicom_all` 以获取集合/模态 |
| `PatientID` | STRING | 患者标识符 |
| `measurementGroup_number` | INTEGER | SR 中的组索引 — 与 `quantitative_measurements` 的连接键 |
| `Quantity` | RECORD | 评估的内容 — `CodeMeaning`（例如，“Malignancy”、“Calcification”、“Texture”） |
| `Value` | RECORD | 编码的答案 — `CodeMeaning`（例如，“4 out of 5 (Moderately Suspicious for Cancer)”） |
| `finding` | RECORD | 评估的发现 — `CodeMeaning`（例如，“Nodule”） |
| `findingSite` | RECORD | 解剖部位 — `CodeMeaning`（例如，“Lung”） |
| `trackingIdentifier` | STRING | 人类可读的跟踪标签 |
| `segmentationInstanceUID` | STRING | 引用的 SEG 对象的 SOPInstanceUID |
| `segmentationSeriesUID` | STRING | 引用的 SEG 对象的 SeriesInstanceUID |
| `segmentationSegmentNumber` | INTEGER | 引用的 SEG 中的段编号 |
| `sourceSegmentedSeriesUID` | STRING | 源图像系列 — 连接到 `dicom_all.SeriesInstanceUID` |

**发现可用的定性特征及其值：**

```sql
SELECT
  Quantity.CodeMeaning AS feature,
  Value.CodeMeaning AS assessed_value,
  finding.CodeMeaning AS finding,
  COUNT(*) AS count
FROM `bigquery-public-data.idc_current.qualitative_measurements`
GROUP BY 1, 2, 3
ORDER BY count DESC
LIMIT 20
```

**查找具有特定恶性评级的所有结节：**

```sql
SELECT
  qm.PatientID,
  qm.trackingIdentifier AS nodule_id,
  qm.Value.CodeMeaning AS malignancy_rating,
  img.collection_id
FROM `bigquery-public-data.idc_current.qualitative_measurements` qm
JOIN `bigquery-public-data.idc_current.dicom_all` img
  ON qm.SeriesInstanceUID = img.SeriesInstanceUID
WHERE qm.Quantity.CodeMeaning = 'Malignancy'
  AND qm.Value.CodeMeaning LIKE '%Suspicious%'
ORDER BY qm.PatientID
LIMIT 20
```

---

### measurement_groups

TID1500 测量组的父表。每行代表 SR 中的一个测量组，包含对分割和源图像的引用，但不包含单个测量值。当你需要枚举组或检查跟踪的内容时，使用此表，而无需拉取所有测量值。

**关键列：** `SOPInstanceUID`、`SeriesInstanceUID`、`PatientID`、`measurementGroup_number`、`trackingIdentifier`、`trackingUniqueIdentifier`、`finding`、`findingSite`、`segmentationInstanceUID`、`segmentationSeriesUID`、`segmentationSegmentNumber`、`sourceSegmentedSeriesUID`、`contentSequence`（原始 SR 内容序列）。

在大多数工作流中，直接使用 `SOPInstanceUID` + `measurementGroup_number` 连接 `quantitative_measurements` 和 `qualitative_measurements`，而无需通过 `measurement_groups`。

---

### 组合定量和定性测量值

需要两个表的主要用例：将数值特征（体积、直径）与同一发现的编码评估（恶性、纹理）相关联。通过 `SOPInstanceUID` + `measurementGroup_number` 连接。

**示例：LIDC-IDRI 肺结节分析 — 恶性评级与体积和直径：**

```sql
SELECT
  qual.PatientID,
  qual.trackingIdentifier AS nodule_id,
  qual.Value.CodeMeaning AS malignancy_rating,
  ROUND(CAST(vol.Value AS FLOAT64), 1) AS volume_mm3,
  ROUND(CAST(diam.Value AS FLOAT64), 1) AS diameter_mm
FROM `bigquery-public-data.idc_current.qualitative_measurements` qual
JOIN `bigquery-public-data.idc_current.quantitative_measurements` vol
  ON qual.SOPInstanceUID = vol.SOPInstanceUID
  AND qual.measurementGroup_number = vol.measurementGroup_number
JOIN `bigquery-public-data.idc_current.quantitative_measurements` diam
  ON qual.SOPInstanceUID = diam.SOPInstanceUID
  AND qual.measurementGroup_number = diam.measurementGroup_number
WHERE qual.Quantity.CodeMeaning = 'Malignancy'
  AND vol.Quantity.CodeMeaning = 'Volume'
  AND diam.Quantity.CodeMeaning = 'Diameter'
ORDER BY qual.PatientID, qual.trackingIdentifier
LIMIT 20
```

**连接所有三个派生表以获取完整的段上下文：**

```sql
SELECT
  seg.SegmentedPropertyType.CodeMeaning AS structure,
  qual.Quantity.CodeMeaning AS qualitative_feature,
  qual.Value.CodeMeaning AS qualitative_value,
  qm.Quantity.CodeMeaning AS quantitative_feature,
  ROUND(CAST(qm.Value AS FLOAT64), 3) AS numeric_value,
  qm.Units.CodeMeaning AS units,
  img.collection_id
FROM `bigquery-public-data.idc_current.segmentations` seg
JOIN `bigquery-public-data.idc_current.qualitative_measurements` qual
  ON seg.SeriesInstanceUID = qual.segmentationSeriesUID
  AND seg.SegmentNumber = qual.segmentationSegmentNumber
JOIN `bigquery-public-data.idc_current.quantitative_measurements` qm
  ON qual.SOPInstanceUID = qm.SOPInstanceUID
  AND qual.measurementGroup_number = qm.measurementGroup_number
JOIN `bigquery-public-data.idc_current.dicom_all` img
  ON seg.segmented_SeriesInstanceUID = img.SeriesInstanceUID
WHERE seg.SegmentedPropertyType.CodeMeaning = 'Neoplasm'
LIMIT 10
```

## 私有 DICOM 元素

私有 DICOM 元素是供应商特定的属性，未在 DICOM 标准中定义。它们通常包含对图像解释和分析至关重要的采集参数（如扩散 b 值、梯度方向或扫描仪特定设置）。

### 理解私有元素

**私有元素的工作原理：**
- 私有元素使用奇数编号的组号（例如，0019、0043、2001）
- 每个供应商通过位置（gggg,0010-00FF）的私有创建者标识符保留 256 个元素的块
- 例如，GE 使用私有创建者 "GEMS_PARM_01" 在（0043,0010）处保留元素（0043,1000-10FF）

**标准与私有标签：** 某些参数以两种形式存在：
| 参数 | 标准标签 | GE | Siemens | Philips |
|-----------|--------------|-----|---------|---------|
| 扩散 b 值 | (0018,9087) | (0043,1039) | (0019,100C) | (2001,1003) |
| 私有创建者 | - | GEMS_PARM_01 | SIEMENS CSA HEADER | Philips Imaging |

较旧的扫描仪通常只填充私有标签；较新的扫描仪可能使用标准标签。请始终同时检查两者。

**私有元素的挑战：**
- 需要制造商 DICOM 符合性声明来解释
- 标签含义可能在软件版本之间发生变化
- 在去标识化过程中可能会被删除（以符合 HIPAA）
- 值编码各异（字符串与数值，不同单位）

### 在 BigQuery 中访问私有元素

私有元素存储在 `dicom_all` 的 `OtherElements` 列中，作为包含 `Tag` 和 `Data` 字段的结构体数组。

**标签表示法：** DICOM 表示法 (0043,1039) 变为 BigQuery 格式 `Tag_00431039`。

### 私有元素查询模式

#### 发现可用的私有标签

列出集合中所有非空的私有标签：

```sql
SELECT
  other_elements.Tag,
  COUNT(*) AS instance_count,
  ARRAY_AGG(DISTINCT other_elements.Data[SAFE_OFFSET(0)] IGNORE NULLS LIMIT 5) AS sample_values
FROM `bigquery-public-data.idc_current.dicom_all`,
  UNNEST(OtherElements) AS other_elements
WHERE collection_id = 'qin_prostate_repeatability'
  AND Modality = 'MR'
  AND ARRAY_LENGTH(other_elements.Data) > 0
  AND other_elements.Data[SAFE_OFFSET(0)] IS NOT NULL
  AND other_elements.Data[SAFE_OFFSET(0)] != ''
GROUP BY other_elements.Tag
ORDER BY instance_count DESC
```

针对特定系列：

```sql
SELECT
  other_elements.Tag,
  ARRAY_AGG(DISTINCT other_elements.Data[SAFE_OFFSET(0)] IGNORE NULLS) AS values
FROM `bigquery-public-data.idc_current.dicom_all`,
  UNNEST(OtherElements) AS other_elements
WHERE SeriesInstanceUID = '1.3.6.1.4.1.14519.5.2.1.7311.5101.206828891270520544417996275680'
  AND ARRAY_LENGTH(other_elements.Data) > 0
  AND other_elements.Data[SAFE_OFFSET(0)] IS NOT NULL
  AND other_elements.Data[SAFE_OFFSET(0)] != ''
GROUP BY other_elements.Tag
```

要识别标签的私有创建者，请在同一个组中查找预留元素。例如，如果你找到 `Tag_00431039`，则私有创建者位于 `Tag_00430010`（在组 0043 中预留块 10xx 的标签）。

#### 识别设备制造商

确定生成数据的设备，以找到正确的 DICOM 符合性声明：

```sql
SELECT DISTINCT Manufacturer, ManufacturerModelName
FROM `bigquery-public-data.idc_current.dicom_all`
WHERE collection_id = 'qin_prostate_repeatability'
  AND Modality = 'MR'
```

#### 访问私有元素值

使用 `UNNEST` 访问单个私有元素：

```sql
SELECT
  SeriesInstanceUID,
  SeriesDescription,
  other_elements.Data[SAFE_OFFSET(0)] AS b_value
FROM `bigquery-public-data.idc_current.dicom_all`,
  UNNEST(OtherElements) AS other_elements
WHERE collection_id = 'qin_prostate_repeatability'
  AND other_elements.Tag = 'Tag_00431039'
LIMIT 10
```

#### 按系列聚合值

收集系列中所有切片的唯一值：

```sql
SELECT
  SeriesInstanceUID,
  ANY_VALUE(SeriesDescription) AS SeriesDescription,
  ARRAY_AGG(DISTINCT other_elements.Data[SAFE_OFFSET(0)]) AS b_values
FROM `bigquery-public-data.idc_current.dicom_all`,
  UNNEST(OtherElements) AS other_elements
WHERE collection_id = 'qin_prostate_repeatability'
  AND other_elements.Tag = 'Tag_00431039'
GROUP BY SeriesInstanceUID
```

#### 组合标准和私有过滤器

同时使用标准 DICOM 属性和私有元素值进行过滤：

```sql
SELECT
  PatientID,
  SeriesInstanceUID,
  ANY_VALUE(SeriesDescription) AS SeriesDescription,
  ARRAY_AGG(DISTINCT other_elements.Data[SAFE_OFFSET(0)]) AS b_values,
  COUNT(DISTINCT SOPInstanceUID) AS n_slices
FROM `bigquery-public-data.idc_current.dicom_all`,
  UNNEST(OtherElements) AS other_elements
WHERE collection_id = 'qin_prostate_repeatability'
  AND Modality = 'MR'
  AND other_elements.Tag = 'Tag_00431039'
  AND ImageType[SAFE_OFFSET(0)] = 'ORIGINAL'
  AND other_elements.Data[SAFE_OFFSET(0)] = '1400'
GROUP BY PatientID, SeriesInstanceUID
ORDER BY PatientID
```

#### 跨集合分析

调查私有标签在所有 IDC 集合中的使用情况：

```sql
SELECT
  collection_id,
  ARRAY_TO_STRING(ARRAY_AGG(DISTINCT other_elements.Data[SAFE_OFFSET(0)] IGNORE NULLS), ', ') AS values_found,
  ARRAY_AGG(DISTINCT Manufacturer IGNORE NULLS) AS manufacturers
FROM `bigquery-public-data.idc_current.dicom_all`,
  UNNEST(OtherElements) AS other_elements
WHERE other_elements.Tag = 'Tag_00431039'
  AND other_elements.Data[SAFE_OFFSET(0)] IS NOT NULL
  AND other_elements.Data[SAFE_OFFSET(0)] != ''
GROUP BY collection_id
ORDER BY collection_id
```

### 工作流：查找和使用私有标签

1. **发现集合中可用的私有标签**，使用上面的发现查询
2. **识别制造商**，以知道要查阅哪个符合性声明
3. **从制造商的网站查找 DICOM 符合性声明**（参见下面的资源）
4. **在符合性声明中搜索**所需的参数（例如，“b_value”、“gradient”），以了解每个标签包含什么
5. **将标签转换为 BigQuery 格式：** (gggg,eeee) → `Tag_ggggeeee`
6. **查询并验证**结果，在 IDC 查看器中可视化

### 数据质量说明

- 某些集合显示不现实的值（例如，b 值“1000000600”），表明编码问题或不同的约定
- IDC 数据是去标识化的；包含 PHI 的私有标签可能已被删除或修改
- 相同标签在不同软件版本中可能有不同含义
- 在进行大规模分析之前，始终使用 [IDC 查看器](https://viewer.imaging.datacommons.cancer.gov/) 可视化验证查询结果

### 私有元素资源

**制造商 DICOM 符合性声明：**
- [GE Healthcare MR](https://www.gehealthcare.com/products/interoperability/dicom/magnetic-resonance-imaging-dicom-conformance-statements)
- [Siemens MR](https://www.siemens-healthineers.com/services/it-standards/dicom-conformance-statements-magnetic-resonance)
- [Siemens CT](https://www.siemens-healthineers.com/services/it-standards/dicom-conformance-statements-computed-tomography)

**DICOM 标准：**
- [第 5 部分第 7.8 节 — 私有数据元素](https://dicom.nema.org/medical/dicom/current/output/chtml/part05/sect_7.8.html)
- [第 15 部分附录 E — 去标识化配置文件](https://dicom.nema.org/medical/dicom/current/output/chtml/part15/chapter_e.html)

**社区资源：**
- [NAMIC Wiki: DWI/DTI DICOM](https://www.na-mic.org/wiki/NAMIC_Wiki:DTI:DICOM_for_DWI_and_DTI) — 扩散成像的全面供应商比较
- [StandardizeBValue](https://github.com/nslay/StandardizeBValue) — 将供应商 b 值提取到标准标签的工具

## 使用查询结果与 idc-index

将 BigQuery 用于复杂查询，idc-index 用于下载（下载不需要 GCP 身份验证）：

```python
from google.cloud import bigquery
from idc_index import IDCClient

# 初始化 BigQuery 客户端
# 需要：pip install google-cloud-bigquery
# 身份验证：gcloud auth application-default login
# 项目：即使对于公共数据集，结算也需要项目（免费层适用）
bq_client = bigquery.Client(project="your-gcp-project-id")

# 查询具有特定条件的系列
query = """
SELECT DISTINCT SeriesInstanceUID
FROM `bigquery-public-data.idc_current.dicom_all`
WHERE collection_id = 'tcga_luad'
  AND Modality = 'CT'
  AND Manufacturer = 'GE MEDICAL SYSTEMS'
LIMIT 100
"""

df = bq_client.query(query).to_dataframe()
print(f"找到 {len(df)} 个 GE CT 系列")

# 使用 idc-index 下载（无需 GCP 身份验证）
idc_client = IDCClient()
idc_client.download_from_selection(
    seriesInstanceUID=list(df['SeriesInstanceUID'].values),
    downloadDir="./tcga_luad_thin_ct"
)
```

## 成本与优化

**定价：** 每扫描 1 TB 收费 5 美元（前 1 TB/月免费）。大多数用户保持在免费层内。

**最小化扫描的数据量：**
- 仅选择所需的列（不要使用 `SELECT *`）
- 使用 `WHERE` 子句尽早过滤
- 测试时使用 `LIMIT`
- 尽可能使用 `dicom_all` 而不是 `dicom_metadata`（更小）
- 在 BQ 控制台中预览查询（免费，显示要扫描的字节数）

**运行前检查成本：**
```python
query_job = client.query(query, job_config=bigquery.QueryJobConfig(dry_run=True))
print(f"查询将扫描 {query_job.total_bytes_processed / 1e9:.2f} GB")
```

**使用物化表：** IDC 同时提供视图（`table_name_view`）和物化表（`table_name`）。始终使用物化表（更快，成本更低）。

## 临床数据

临床数据位于单独的数据集中，具有特定于集合的表。所有通过 `idc-index` 可用的临床数据也都在 BigQuery 中可用，内容和结构相同。当你需要复杂的跨集合查询或无法使用本地 `idc-index` 表进行的连接时，请使用 BigQuery。

**数据集：**
- `bigquery-public-data.idc_current_clinical` — 当前版本（用于探索）
- `bigquery-public-data.idc_v{version}_clinical` — 版本化数据集（用于可重现性）

目前约有 130 个临床表，代表约 70 个集合。并非所有集合都有临床数据（从 IDC v11 开始）。

### 临床表命名

大多数集合使用单个表：`<collection_id>_clinical`

**例外：** ACRIN 集合对不同的数据类型使用多个表（例如，`acrin_6698_A0`、`acrin_6698_A1` 等）。

### 元数据表

两个元数据表帮助导航临床数据：

**table_metadata** — 集合级信息：
```sql
SELECT
  collection_id,
  table_name,
  table_description
FROM `bigquery-public-data.idc_current_clinical.table_metadata`
WHERE collection_id = 'nlst'
```

**column_metadata** — 属性级详细信息，包含值映射：
```sql
SELECT
  collection_id,
  table_name,
  column,
  column_label,
  data_type,
  values
FROM `bigquery-public-data.idc_current_clinical.column_metadata`
WHERE collection_id = 'nlst'
  AND column_label LIKE '%stage%'
```

`values` 字段包含观察到的属性值及其描述（与 `idc-index` 的 clinical_index 相同）。

### 常见临床查询

**列出可用的临床表：**
```sql
SELECT table_name
FROM `bigquery-public-data.idc_current_clinical.INFORMATION_SCHEMA.TABLES`
WHERE table_name NOT IN ('table_metadata', 'column_metadata')
```

**查找具有特定临床属性的集合：**
```sql
SELECT DISTINCT collection_id, table_name, column, column_label
FROM `bigquery-public-data.idc_current_clinical.column_metadata`
WHERE LOWER(column_label) LIKE '%chemotherapy%'
```

**查询集合的临床数据：**
```sql
-- 示例：NLST 癌症分期数据
SELECT
  dicom_patient_id,
  clinical_stag,
  path_stag,
  de_stag
FROM `bigquery-public-data.idc_current_clinical.nlst_canc`
WHERE clinical_stag IS NOT NULL
LIMIT 10
```

**将临床数据与影像数据连接：**
```sql
SELECT
  d.PatientID,
  d.StudyInstanceUID,
  d.Modality,
  c.clinical_stag,
  c.path_stag
FROM `bigquery-public-data.idc_current.dicom_all` d
JOIN `bigquery-public-data.idc_current_clinical.nlst_canc` c
  ON d.PatientID = c.dicom_patient_id
WHERE d.collection_id = 'nlst'
  AND d.Modality = 'CT'
  AND c.clinical_stag = '400'  -- Stage IV
LIMIT 20
```

**跨集合临床搜索：**
```sql
-- 查找所有具有分期信息的集合
SELECT
  cm.collection_id,
  cm.table_name,
  cm.column,
  cm.column_label
FROM `bigquery-public-data.idc_current_clinical.column_metadata` cm
WHERE LOWER(cm.column_label) LIKE '%stage%'
ORDER BY cm.collection_id
```

### 关键列：dicom_patient_id

每个临床表都包含 `dicom_patient_id`，它与影像表中的 DICOM `PatientID` 属性匹配。这是临床数据和影像数据之间的连接键。

**注意：** 临床表模式因集合而异。始终首先检查可用的列：
```sql
SELECT column_name, data_type
FROM `bigquery-public-data.idc_current_clinical.INFORMATION_SCHEMA.COLUMNS`
WHERE table_name = 'nlst_canc'
```

请参阅 `references/clinical_data_guide.md` 了解使用 `idc-index` 的详细工作流，它提供相同的临床数据，无需 BigQuery 身份验证。

## 重要说明

- 表是只读的（公共数据集）
- 模式在 IDC 版本之间会发生变化
- 使用版本化数据集以实现可重现性
- 某些深度超过 15 层的 DICOM 序列未提取
- 非常大的序列（>1MB）可能会被截断
- 在使用前始终检查数据许可证

## 常见错误

**问题：必须启用结算**
- 原因：BigQuery 需要一个启用结算的 GCP 项目
- 解决方案：在 Google Cloud 控制台中启用结算，或改用 idc-index 迷你索引

**问题：查询超出资源限制**
- 原因：查询扫描的数据太多或过于复杂
- 解决方案：添加更具体的 WHERE 过滤器，使用 LIMIT，分解为更小的查询

**问题：找不到列**
- 原因：字段名拼写错误或未在所选表中
- 解决方案：首先使用 `INFORMATION_SCHEMA.COLUMNS` 检查表模式

**问题：权限被拒绝**
- 原因：未向 Google Cloud 进行身份验证
- 解决方案：运行 `gcloud auth application-default login` 或设置 GOOGLE_APPLICATION_CREDENTIALS

## 资源

- [理解 BigQuery DICOM 模式](https://docs.cloud.google.com/healthcare-api/docs/how-tos/dicom-bigquery-schema)
- [BigQuery 查询语法](https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax)
- [Kaggle SQL 入门](https://www.kaggle.com/learn/intro-to-sql)
- [IDC 数据的 BigQuery 示例查询](https://github.com/ImagingDataCommons/idc-bigquery-cookbook)
