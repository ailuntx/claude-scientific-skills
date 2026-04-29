# IDC 的 SQL 查询模式

**测试版本：** idc-index 0.11.14（IDC 数据版本 v23）

处理 IDC 数据时常用 SQL 查询模式的快速参考。有关带上下文的详细示例，请参阅主 SKILL.md 中的“核心功能”部分。

## 何时使用本指南

当您需要以下 SQL 模式的快速参考时，请加载本指南：
- 发现可用的过滤值（模态、身体部位、制造商）
- 跨集合查找标注和分割结果
- 查询切片显微图像和标注数据
- 在下载前估算下载大小
- 将影像数据关联到临床数据
- 按 3D 体积几何有效性过滤（volume_geometry_index）
- 查找 RT 结构集和 ROI 元数据（rtstruct_index）

有关表结构、DataFrame 访问和连接列引用，请参见 `references/index_tables_guide.md`。

## 前提条件

```bash
pip install --upgrade idc-index
```

```python
from idc_index import IDCClient
client = IDCClient()
```

## 发现可用的过滤值

```python
# 存在哪些模态？
client.sql_query("SELECT DISTINCT Modality FROM index")

# 特定模态对应的身体部位？
client.sql_query("""
    SELECT DISTINCT BodyPartExamined, COUNT(*) as n
    FROM index WHERE Modality = 'CT' AND BodyPartExamined IS NOT NULL
    GROUP BY BodyPartExamined ORDER BY n DESC
""")

# MR 有哪些制造商？
client.sql_query("""
    SELECT DISTINCT Manufacturer, COUNT(*) as n
    FROM index WHERE Modality = 'MR'
    GROUP BY Manufacturer ORDER BY n DESC
""")
```

## 查找标注和分割结果

**注意：** 并非所有从图像派生的对象都属于分析结果集合。某些标注与原始图像一同存放。请使用 DICOM Modality 或 SOPClassUID 查找所有派生对象，无论其集合类型如何。

```python
# 通过 DICOM Modality 查找所有分割和结构集
# SEG = DICOM 分割，RTSTRUCT = 放射治疗结构集
client.sql_query("""
    SELECT collection_id, Modality, COUNT(*) as series_count
    FROM index
    WHERE Modality IN ('SEG', 'RTSTRUCT')
    GROUP BY collection_id, Modality
    ORDER BY series_count DESC
""")

# 查找特定集合的分割（包括非分析结果项）
client.sql_query("""
    SELECT SeriesInstanceUID, SeriesDescription, analysis_result_id
    FROM index
    WHERE collection_id = 'tcga_luad' AND Modality = 'SEG'
""")

# 列出分析结果集合（精选的派生数据集）
client.fetch_index("analysis_results_index")
client.sql_query("""
    SELECT analysis_result_id, analysis_result_title, Collections, Modalities
    FROM analysis_results_index
""")

# 查找特定源集合的分析结果
client.sql_query("""
    SELECT analysis_result_id, analysis_result_title
    FROM analysis_results_index
    WHERE Collections LIKE '%tcga_luad%'
""")

# 使用 seg_index 获取详细的 DICOM 分割元数据
client.fetch_index("seg_index")

# 按算法获取分割统计信息
client.sql_query("""
    SELECT AlgorithmName, AlgorithmType, COUNT(*) as seg_count
    FROM seg_index
    WHERE AlgorithmName IS NOT NULL
    GROUP BY AlgorithmName, AlgorithmType
    ORDER BY seg_count DESC
    LIMIT 10
""")

# 查找特定源图像的分割（例如胸部 CT）
client.sql_query("""
    SELECT
        s.SeriesInstanceUID as seg_series,
        s.AlgorithmName,
        s.total_segments,
        s.segmented_SeriesInstanceUID as source_series
    FROM seg_index s
    JOIN index src ON s.segmented_SeriesInstanceUID = src.SeriesInstanceUID
    WHERE src.Modality = 'CT' AND src.BodyPartExamined = 'CHEST'
    LIMIT 10
""")

# 查找并附带源图像上下文的 TotalSegmentator 结果
client.sql_query("""
    SELECT
        seg_info.collection_id,
        COUNT(DISTINCT s.SeriesInstanceUID) as seg_count,
        SUM(s.total_segments) as total_segments
    FROM seg_index s
    JOIN index seg_info ON s.SeriesInstanceUID = seg_info.SeriesInstanceUID
    WHERE s.AlgorithmName LIKE '%TotalSegmentator%'
    GROUP BY seg_info.collection_id
    ORDER BY seg_count DESC
""")

# 使用 ann_index 和 ann_group_index 处理显微镜批量简单标注
# ann_group_index 包含 AnnotationGroupLabel、GraphicType、NumberOfAnnotations、AlgorithmName
client.fetch_index("ann_index")
client.fetch_index("ann_group_index")
client.sql_query("""
    SELECT g.AnnotationGroupLabel, g.GraphicType, g.NumberOfAnnotations, i.collection_id
    FROM ann_group_index g
    JOIN ann_index a ON g.SeriesInstanceUID = a.SeriesInstanceUID
    JOIN index i ON a.SeriesInstanceUID = i.SeriesInstanceUID
    WHERE g.AlgorithmName IS NOT NULL
    LIMIT 10
""")
# 关于 AnnotationGroupLabel 过滤、SM+ANN 连接等更多信息，请参见 references/digital_pathology_guide.md
```

## 查询切片显微图像和标注数据

使用 `sm_index` 获取切片显微图像元数据，使用 `ann_index`/`ann_group_index` 获取切片上的标注（DICOM ANN 对象）。通过 `AnnotationGroupLabel` 过滤标注组以按名称查找标注。

```python
client.fetch_index("sm_index")
client.fetch_index("ann_index")
client.fetch_index("ann_group_index")

# 示例：在集合中按标签查找标注组
client.sql_query("""
    SELECT g.AnnotationGroupLabel, g.GraphicType, g.NumberOfAnnotations
    FROM ann_group_index g
    JOIN index i ON g.SeriesInstanceUID = i.SeriesInstanceUID
    WHERE i.collection_id = 'your_collection_id'
      AND LOWER(g.AnnotationGroupLabel) LIKE '%keyword%'
""")
```

有关 SM 查询、ANN 过滤模式、SM+ANN 交叉引用和连接示例，请参见 `references/digital_pathology_guide.md`。

## 估算下载大小

```python
# 特定条件的下载大小
client.sql_query("""
    SELECT SUM(series_size_MB) as total_mb, COUNT(*) as series_count
    FROM index
    WHERE collection_id = 'nlst' AND Modality = 'CT'
""")
```

## 链接临床数据

```python
client.fetch_index("clinical_index")

# 查找具有临床数据的集合及其表
client.sql_query("""
    SELECT collection_id, table_name, COUNT(DISTINCT column_label) as columns
    FROM clinical_index
    GROUP BY collection_id, table_name
    ORDER BY collection_id
""")
```

有关完整模式（包括值映射和患者队列选择），请参见 `references/clinical_data_guide.md`。

## 故障排除

**问题：** 查询返回错误“table not found”
- **原因：** 查询前未获取索引
- **解决方案：** 在使用主 `index` 以外的表之前，调用 `client.fetch_index("table_name")`

**问题：** LIKE 模式未匹配到预期结果
- **原因：** 大小写敏感或空白字符
- **解决方案：** 使用 `LOWER(column)` 进行大小写不敏感匹配，使用 `TRIM()` 处理空白

**问题：** JOIN 返回的行数少于预期
- **原因：** 连接列中的 NULL 值或没有匹配记录
- **解决方案：** 使用 `LEFT JOIN` 包含无匹配的行，使用 `IS NOT NULL` 检查 NULL

## 体积几何验证

`volume_geometry_index` 涵盖单帧 CT、MR 和 PT 序列。查询前请先获取该索引。

```python
client.fetch_index("volume_geometry_index")

# 构成规则间隔 3D 体积的序列（无需重采样）
client.sql_query("""
    SELECT i.collection_id, i.SeriesInstanceUID, i.BodyPartExamined,
           v.obliquity_degrees
    FROM index i
    JOIN volume_geometry_index v ON i.SeriesInstanceUID = v.SeriesInstanceUID
    WHERE i.Modality = 'CT'
      AND v.regularly_spaced_3d_volume = TRUE
    LIMIT 10
""")

# 每个集合中 3D 有效的 CT 占比
client.sql_query("""
    SELECT i.collection_id,
           COUNT(*) as total_ct,
           SUM(CASE WHEN v.regularly_spaced_3d_volume THEN 1 ELSE 0 END) as valid_3d,
           ROUND(100.0 * SUM(CASE WHEN v.regularly_spaced_3d_volume THEN 1 ELSE 0 END) / COUNT(*), 1) as pct_valid
    FROM index i
    JOIN volume_geometry_index v ON i.SeriesInstanceUID = v.SeriesInstanceUID
    WHERE i.Modality = 'CT'
    GROUP BY i.collection_id
    ORDER BY total_ct DESC
    LIMIT 10
""")
```

关键列：`regularly_spaced_3d_volume`（复合标志），`obliquity_degrees`（0 为纯轴位/矢状位/冠状位），以及各个布尔检查：`single_orientation`、`orthogonal_orientation`、`unique_slice_positions`、`consistent_pixel_spacing`、`consistent_image_dimensions`、`uniform_slice_spacing`。

## RT 结构集

`rtstruct_index` 每个 RTSTRUCT 序列对应一行。数组列（`ROINames`、`ROIGenerationAlgorithms`、`RTROIInterpretedTypes`）以字符串形式存储。

```python
client.fetch_index("rtstruct_index")

# RTSTRUCT 序列及其 ROI 计数和名称
client.sql_query("""
    SELECT i.collection_id, i.SeriesInstanceUID,
           r.total_rois, r.ROINames, r.RTROIInterpretedTypes,
           r.referenced_SeriesInstanceUID
    FROM index i
    JOIN rtstruct_index r ON i.SeriesInstanceUID = r.SeriesInstanceUID
    LIMIT 10
""")

# 包含最多 RTSTRUCT 序列的集合
client.sql_query("""
    SELECT i.collection_id,
           COUNT(*) as rtstruct_series,
           ROUND(AVG(r.total_rois), 1) as avg_rois
    FROM index i
    JOIN rtstruct_index r ON i.SeriesInstanceUID = r.SeriesInstanceUID
    GROUP BY i.collection_id
    ORDER BY rtstruct_series DESC
    LIMIT 10
""")

# 查找给定 RTSTRUCT 的源 CT 序列
client.sql_query("""
    SELECT r.SeriesInstanceUID as rtstruct_uid,
           r.total_rois, r.ROINames,
           src.SeriesInstanceUID as source_ct_uid,
           src.collection_id, src.BodyPartExamined
    FROM rtstruct_index r
    JOIN index src ON r.referenced_SeriesInstanceUID = src.SeriesInstanceUID
    LIMIT 10
""")
```

## 资源

- `references/index_tables_guide.md`：表结构、DataFrame 访问和连接列引用
- `references/clinical_data_guide.md`：临床数据模式和值映射
- `references/digital_pathology_guide.md`：病理学特定查询
- `references/bigquery_guide.md`：需要完整 DICOM 元数据的高级查询
- `references/parquet_access_guide.md`：无需安装 idc-index 即可直接查询 Parquet
