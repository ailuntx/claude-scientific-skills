# IDC 数字病理学指南

**测试版本：** IDC 数据版本 v23，idc-index 0.11.10

常规 IDC 查询和下载请使用 `idc-index`（参见主 SKILL.md）。本指南涵盖 IDC 数字病理学背景下的玻片显微成像（SM）、显微批量简单标注（ANN）和分割（SEG）。

## 数字病理学索引表

五个专用索引表提供精选元数据，无需使用 BigQuery：

| 表名 | 行粒度 | 描述 |
|-------|-----------------|-------------|
| `sm_index` | 1 行 = 1 个 SM 序列 | 玻片显微序列元数据：容器/玻片 ID、组织类型、解剖结构、诊断、物镜倍数、像素间距、图像尺寸 |
| `sm_instance_index` | 1 行 = 1 个 SM 实例 | 单张玻片图像的实例级（SOPInstanceUID）元数据 |
| `seg_index` | 1 行 = 1 个 SEG 序列 | DICOM 分割元数据：算法、片段数量、源序列引用。同时适用于放射学和病理学——通过源模态筛选病理学专用分割 |
| `ann_index` | 1 行 = 1 个 ANN 序列 | 显微批量简单标注序列元数据；包含链接到标注玻片的 `referenced_SeriesInstanceUID` |
| `ann_group_index` | 1 行 = 1 个标注组 | 标注组详情：`AnnotationGroupLabel`、`GraphicType`、`NumberOfAnnotations`、`AlgorithmName`、属性代码 |

所有表在查询前需执行 `client.fetch_index("table_name")`。使用 `client.indices_overview` 以编程方式检查列结构。

## 玻片显微成像查询

### 基础 SM 元数据

```python
from idc_index import IDCClient
client = IDCClient()

# sm_index 包含详细元数据；与 index 表连接获取 collection_id
client.fetch_index("sm_index")
client.sql_query("""
    SELECT i.collection_id, COUNT(*) as slides,
           MIN(s.min_PixelSpacing_2sf) as min_resolution
    FROM sm_index s
    JOIN index i ON s.SeriesInstanceUID = i.SeriesInstanceUID
    GROUP BY i.collection_id
    ORDER BY slides DESC
""")
```

### 查找特定属性的 SM 序列

```python
# 查找具有特定物镜倍数的高分辨率玻片
client.fetch_index("sm_index")
client.sql_query("""
    SELECT
        i.collection_id,
        i.PatientID,
        s.ObjectiveLensPower,
        s.min_PixelSpacing_2sf
    FROM sm_index s
    JOIN index i ON s.SeriesInstanceUID = i.SeriesInstanceUID
    WHERE s.ObjectiveLensPower >= 40
    ORDER BY s.min_PixelSpacing_2sf
    LIMIT 20
""")
```

### 按样本制备方式筛选

`sm_index` 包含染色、包埋和固定剂元数据。这些列为**数组类型**（例如 H&E 染色表示为 `[苏木精染色, 水溶性伊红染色]`）——使用 `array_to_string()` 配合 `LIKE` 或 `list_contains()` 进行筛选。

```python
# 在集合中查找 H&E 染色玻片
client.fetch_index("sm_index")
client.sql_query("""
    SELECT
        i.PatientID,
        s.staining_usingSubstance_CodeMeaning as staining,
        s.embeddingMedium_CodeMeaning as embedding,
        s.tissueFixative_CodeMeaning as fixative
    FROM sm_index s
    JOIN index i ON s.SeriesInstanceUID = i.SeriesInstanceUID
    WHERE i.collection_id = 'tcga_brca'
      AND array_to_string(s.staining_usingSubstance_CodeMeaning, ', ') LIKE '%hematoxylin%'
    LIMIT 10
""")
```

```python
# 跨集合比较 FFPE 与冷冻玻片
client.sql_query("""
    SELECT
        i.collection_id,
        s.embeddingMedium_CodeMeaning as embedding,
        COUNT(*) as slide_count
    FROM sm_index s
    JOIN index i ON s.SeriesInstanceUID = i.SeriesInstanceUID
    GROUP BY i.collection_id, embedding
    ORDER BY i.collection_id, slide_count DESC
""")
```

## 识别肿瘤与正常玻片

`sm_index` 表提供两种组织类型识别方式：

| 列名 | 应用场景 |
|--------|----------|
| `primaryAnatomicStructureModifier_CodeMeaning` | 来自 DICOM 样本元数据的结构化组织类型（如 `肿瘤, 原发`、`正常`、`肿瘤`、`肿瘤, 转移性`）。适用于所有含 SM 数据的集合。 |
| `ContainerIdentifier` | 玻片/容器标识符。对于 TCGA 集合，包含 [TCGA 条形码](https://docs.gdc.cancer.gov/Encyclopedia/pages/TCGA_Barcode/)，其中[样本类型代码](https://gdc.cancer.gov/resources-tcga-users/tcga-code-tables/sample-type-codes)（第 14-15 位）编码组织来源：`01`-`09` = 肿瘤，`10`-`19` = 正常。 |

### 使用结构化组织类型元数据

```python
from idc_index import IDCClient
client = IDCClient()
client.fetch_index("sm_index")

# 探索所有 SM 数据的组织类型值
client.sql_query("""
    SELECT
        s.primaryAnatomicStructureModifier_CodeMeaning as tissue_type,
        COUNT(*) as slide_count
    FROM sm_index s
    WHERE s.primaryAnatomicStructureModifier_CodeMeaning IS NOT NULL
    GROUP BY tissue_type
    ORDER BY slide_count DESC
""")
```

#### 示例：TCGA-BRCA 中的肿瘤与正常玻片

```python
# TCGA-BRCA 的组织类型分布
client.sql_query("""
    SELECT
        s.primaryAnatomicStructureModifier_CodeMeaning as tissue_type,
        COUNT(*) as slide_count,
        COUNT(DISTINCT i.PatientID) as patient_count
    FROM sm_index s
    JOIN index i ON s.SeriesInstanceUID = i.SeriesInstanceUID
    WHERE i.collection_id = 'tcga_brca'
    GROUP BY tissue_type
    ORDER BY slide_count DESC
""")
# 返回结果：肿瘤, 原发（2704 张玻片），正常（399 张玻片）
```

### 使用 TCGA 条形码（仅限 TCGA 集合）

对于 TCGA 集合，`ContainerIdentifier` 包含玻片条形码（如 `TCGA-E9-A3X8-01A-03-TSC`）。提取样本类型代码进行组织分类：

```python
# 从 TCGA 条形码解析样本类型
client.sql_query("""
    SELECT
        SUBSTRING(SPLIT_PART(s.ContainerIdentifier, '-', 4), 1, 2) as sample_type_code,
        s.primaryAnatomicStructureModifier_CodeMeaning as tissue_type,
        COUNT(*) as slide_count
    FROM sm_index s
    JOIN index i ON s.SeriesInstanceUID = i.SeriesInstanceUID
    WHERE i.collection_id = 'tcga_brca'
    GROUP BY sample_type_code, tissue_type
    ORDER BY sample_type_code
""")
# 返回结果：01 → 肿瘤, 原发（2704），06 → 无（8），11 → 正常（399）
```

条形码方法可捕获结构化元数据为 NULL 的情况（例如 TCGA-BRCA 中 `06` = 转移性玻片的 `primaryAnatomicStructureModifier_CodeMeaning` 为 NULL）。

## 标注查询（ANN）

DICOM 显微批量简单标注（模态 = 'ANN'）是**基于**玻片显微图像的标注。它们出现在 `ann_index`（序列级）和 `ann_group_index`（组级详情）中。每个 ANN 序列通过 `referenced_SeriesInstanceUID` 引用其标注的玻片。

### 基础标注发现

```python
# 查找标注序列及其引用的图像
client.fetch_index("ann_index")
client.fetch_index("ann_group_index")

client.sql_query("""
    SELECT
        a.SeriesInstanceUID as ann_series,
        a.AnnotationCoordinateType,
        a.referenced_SeriesInstanceUID as source_series
    FROM ann_index a
    LIMIT 10
""")
```

### 标注组统计

```python
# 获取标注组详情（图形类型、数量、算法）
client.sql_query("""
    SELECT
        GraphicType,
        SUM(NumberOfAnnotations) as total_annotations,
        COUNT(*) as group_count
    FROM ann_group_index
    GROUP BY GraphicType
    ORDER BY total_annotations DESC
""")
```

### 查找带源玻片上下文的标注

```python
# 查找标注及其源玻片显微上下文
client.sql_query("""
    SELECT
        i.collection_id,
        g.GraphicType,
        g.AnnotationPropertyType_CodeMeaning,
        g.AlgorithmName,
        g.NumberOfAnnotations
    FROM ann_group_index g
    JOIN ann_index a ON g.SeriesInstanceUID = a.SeriesInstanceUID
    JOIN index i ON a.referenced_SeriesInstanceUID = i.SeriesInstanceUID
    WHERE g.AlgorithmName IS NOT NULL
    LIMIT 10
""")
```

## 玻片显微成像分割

DICOM 分割（模态 = 'SEG'）同时用于放射学（如 CT 器官分割）和病理学（如全玻片图像组织区域分割）。使用 `seg_index.segmented_SeriesInstanceUID` 查找源序列，然后通过源模态筛选以分离病理学分割。

```python
# 查找源为玻片显微图像的分割
client.fetch_index("seg_index")
client.fetch_index("sm_index")
client.sql_query("""
    SELECT
        seg.SeriesInstanceUID as seg_series,
        seg.AlgorithmName,
        seg.total_segments,
        src.collection_id,
        src.Modality as source_modality
    FROM seg_index seg
    JOIN index src ON seg.segmented_SeriesInstanceUID = src.SeriesInstanceUID
    WHERE src.Modality = 'SM'
    LIMIT 20
""")
```

## 查找预计算分析结果

IDC 托管派生数据集（细胞核分割、TIL 图谱、AI 标注），在主 `index` 表中通过 `analysis_result_id` 标识。使用 `analysis_results_index` 发现病理学可用资源。

```python
from idc_index import IDCClient
client = IDCClient()
client.fetch_index("analysis_results_index")

# 查找包含病理学标注或分割的分析结果
client.sql_query("""
    SELECT
        ar.analysis_result_id,
        ar.analysis_result_title,
        ar.Modalities,
        ar.Subjects,
        ar.Collections
    FROM analysis_results_index ar
    WHERE ar.Modalities LIKE '%ANN%' OR ar.Modalities LIKE '%SEG%'
    ORDER BY ar.Subjects DESC
""")
```

### 查找特定玻片的分析结果

```python
# 查找 TCGA-BRCA 玻片的所有派生数据（标注、分割）
client.fetch_index("ann_index")
client.sql_query("""
    SELECT
        i.analysis_result_id,
        i.PatientID,
        a.referenced_SeriesInstanceUID as source_slide,
        g.AnnotationGroupLabel,
        g.NumberOfAnnotations,
        g.AlgorithmName
    FROM ann_group_index g
    JOIN ann_index a ON g.SeriesInstanceUID = a.SeriesInstanceUID
    JOIN index i ON a.SeriesInstanceUID = i.SeriesInstanceUID
    WHERE i.collection_id = 'tcga_brca'
    LIMIT 10
""")
```

标注对象可能包含**每项标注的测量值**（如细胞核面积、偏心率），这些数据存储在 DICOM 文件中但未纳入索引表——下载后需使用 [highdicom](https://github.com/ImagingDataCommons/highdicom) 提取（`ann.get_annotation_groups()`、`group.get_measurements()`）。参见 [microscopy_dicom_ann_intro](https://github.com/ImagingDataCommons/IDC-Tutorials/blob/master/notebooks/pathomics/microscopy_dicom_ann_intro.ipynb) 教程获取包含空间分析和细胞密度计算的完整示例。

## 按 AnnotationGroupLabel 筛选

`AnnotationGroupLabel` 是按名称或语义内容查找标注组的最直接列。使用带通配符的 `LIKE` 进行文本搜索。

### 简单标签筛选

```python
# 按标签查找标注组（例如包含 "blast" 的组）
client.fetch_index("ann_group_index")
client.sql_query("""
    SELECT
        g.SeriesInstanceUID,
        g.AnnotationGroupLabel,
        g.GraphicType,
        g.NumberOfAnnotations,
        g.AlgorithmName
    FROM ann_group_index g
    WHERE LOWER(g.AnnotationGroupLabel) LIKE '%blast%'
    ORDER BY g.NumberOfAnnotations DESC
""")
```

### 带集合上下文的标签筛选

```python
# 在特定集合中查找匹配标签的标注组
client.fetch_index("ann_index")
client.fetch_index("ann_group_index")
client.sql_query("""
    SELECT
        i.collection_id,
        g.AnnotationGroupLabel,
        g.GraphicType,
        g.NumberOfAnnotations,
        g.AnnotationPropertyType_CodeMeaning
    FROM ann_group_index g
    JOIN ann_index a ON g.SeriesInstanceUID = a.SeriesInstanceUID
    JOIN index i ON a.SeriesInstanceUID = i.SeriesInstanceUID
    WHERE i.collection_id = 'your_collection_id'
      AND LOWER(g.AnnotationGroupLabel) LIKE '%keyword%'
    ORDER BY g.NumberOfAnnotations DESC
""")
```

## 玻片显微成像标注（SM + ANN 交叉引用）

查找与玻片显微数据相关的标注时，需同时使用 SM 和 ANN 表。`ann_index.referenced_SeriesInstanceUID` 将每个标注序列链接到其源玻片。

```python
# 在集合中查找玻片显微图像及其标注
client.fetch_index("sm_index")
client.fetch_index("ann_index")
client.fetch_index("ann_group_index")
client.sql_query("""
    SELECT
        i.collection_id,
        s.ObjectiveLensPower,
        g.AnnotationGroupLabel,
        g.NumberOfAnnotations,
        g.GraphicType
    FROM ann_group_index g
    JOIN ann_index a ON g.SeriesInstanceUID = a.SeriesInstanceUID
    JOIN sm_index s ON a.referenced_SeriesInstanceUID = s.SeriesInstanceUID
    JOIN index i ON a.SeriesInstanceUID = i.SeriesInstanceUID
    WHERE i.collection_id = 'your_collection_id'
    ORDER BY g.NumberOfAnnotations DESC
""")
```

## 连接模式

### SM 连接（带集合上下文的玻片显微详情）

```python
client.fetch_index("sm_index")
result = client.sql_query("""
    SELECT i.collection_id, i.PatientID, s.ObjectiveLensPower, s.min_PixelSpacing_2sf
    FROM index i
    JOIN sm_index s ON i.SeriesInstanceUID = s.SeriesInstanceUID
    LIMIT 10
""")
```

### ANN 连接（带集合上下文的标注组）

```python
client.fetch_index("ann_index")
client.fetch_index("ann_group_index")
result = client.sql_query("""
    SELECT
        i.collection_id,
        g.AnnotationGroupLabel,
        g.GraphicType,
        g.NumberOfAnnotations,
        a.referenced_SeriesInstanceUID as source_series
    FROM ann_group_index g
    JOIN ann_index a ON g.SeriesInstanceUID = a.SeriesInstanceUID
    JOIN index i ON a.SeriesInstanceUID = i.SeriesInstanceUID
    LIMIT 10
""")
```

## 相关工具

以下工具支持数字病理工作流的 DICOM 格式：

**Python 库：**
- [highdicom](https://github.com/ImagingDataCommons/highdicom) - Python 高级 DICOM 抽象库。创建和读取病理学/放射学的 DICOM 分割（SEG）、结构化报告（SR）及参数图。由 IDC 开发。
- [wsidicom](https://github.com/imi-bigpicture/wsidicom) - 读取 DICOM WSI 数据集的 Python 包。将元数据解析为易于使用的数据类，用于全玻片图像分析。
- [TIA-Toolbox](https://github.com/TissueImageAnalytics/tiatoolbox) - 端到端计算病理学库，通过 `DICOMWSIReader` 支持 DICOM。提供切片提取、特征提取和预训练深度学习模型。
- [EZ-WSI-DICOMweb](https://github.com/GoogleCloudPlatform/EZ-WSI-DICOMweb) - 通过 DICOMweb 从 DICOM 全玻片图像提取图像块。专为云 DICOM 存储的 AI/ML 工作流设计。

**查看器：**
- [Slim](https://github.com/ImagingDataCommons/slim) - 基于 Web
