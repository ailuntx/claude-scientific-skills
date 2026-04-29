# IDC 直接 Parquet 访问指南

**测试版本：** idc-index-data 23.10.1，DuckDB 1.x

所有 idc-index 元数据表都以 Parquet 文件形式发布到公共 GCS 存储桶，并具有不受限制的 CORS 访问。这使得无需安装 idc-index 即可使用 DuckDB 或 pandas 进行元数据查询——适用于快速探索或无法使用 `pip install` 的环境。

**限制：** 下载辅助函数（`download_from_selection()`）、查看器 URL（`get_viewer_URL()`）和引用生成需要 idc-index 客户端，从原始 Parquet 文件中无法使用。

## 何时使用本指南

当您需要以下操作时，请参考本指南：
- 在不安装 idc-index 的情况下查询 IDC 元数据
- 对最新索引文件执行即席 DuckDB 查询
- 访问 `volume_geometry_index` 或 `rtstruct_index` 以进行几何验证或 RT 结构查询

如需完整的 API 访问（下载、查看器、引用），请使用主文档 SKILL.md 中描述的 idc-index。

## URL 模式

```
https://storage.googleapis.com/idc-index-data-artifacts/current/release_artifacts/{filename}.parquet
```

`current/` 始终解析为最新数据版本。要锁定特定版本，请将 `current` 替换为数据版本号（例如 `23.10.1`）。

## 可用文件

| 文件 | 大约大小 | 描述 |
|------|---------|------|
| `idc_index.parquet` | ~70 MB | 主索引（所有 DICOM 系列元数据） |
| `volume_geometry_index.parquet` | ~5 MB | CT/MR/PT 系列的 3D 几何验证 |
| `rtstruct_index.parquet` | ~2 MB | RT 结构集 ROI 元数据 |
| `seg_index.parquet` | ~6 MB | DICOM 分割交叉引用 |
| `sm_index.parquet` | ~2 MB | 切片显微镜系列元数据 |
| `contrast_index.parquet` | ~1 MB | 造影剂元数据 |
| `ann_index.parquet` | ~0.2 MB | 显微镜注释系列元数据 |
| `ann_group_index.parquet` | ~0.5 MB | 注释组元数据 |
| `collections_index.parquet` | — | 集合级元数据 |
| `analysis_results_index.parquet` | — | 派生数据集元数据 |
| `clinical_index.parquet` | ~0.2 MB | 临床数据列字典 |
| `prior_versions_index.parquet` | — | 先前 IDC 版本的系列 |

**注意：** 主索引文件名为 `idc_index.parquet`，而不是 `index.parquet`。在 SQL 查询中使用别名引用（例如 `FROM read_parquet(...) AS index`）。

## 前提条件

```bash
pip install duckdb
# 或： uv add duckdb
```

DuckDB 通过 HTTP 范围请求直接从 HTTPS URL 读取 Parquet——无需 GCS 客户端库或身份验证。

## 基本查询

```python
import duckdb

BASE = "https://storage.googleapis.com/idc-index-data-artifacts/current/release_artifacts"

# 发现模态和系列数
duckdb.sql(f"""
    SELECT Modality, COUNT(*) as series_count, ROUND(SUM(series_size_MB)/1000, 1) as size_GB
    FROM read_parquet('{BASE}/idc_index.parquet')
    GROUP BY Modality
    ORDER BY series_count DESC
""").df()

# 包含 CT 数据的集合，按大小排序
duckdb.sql(f"""
    SELECT collection_id,
           COUNT(DISTINCT PatientID) as patients,
           COUNT(*) as series,
           ROUND(SUM(series_size_MB)/1000, 1) as size_GB
    FROM read_parquet('{BASE}/idc_index.parquet')
    WHERE Modality = 'CT'
    GROUP BY collection_id
    ORDER BY size_GB DESC
    LIMIT 10
""").df()
```

## 体积几何验证

`volume_geometry_index` 涵盖单帧 CT、MR 和 PT 系列。每行包含关于方向、间距、尺寸和切片位置的布尔检查，以及一个复合标志 `regularly_spaced_3d_volume`。

```python
import duckdb

BASE = "https://storage.googleapis.com/idc-index-data-artifacts/current/release_artifacts"

# 构成有效 3D 体积的 CT 系列（无需重采样即可加载）
duckdb.sql(f"""
    SELECT i.collection_id, i.SeriesInstanceUID, i.BodyPartExamined,
           v.obliquity_degrees, v.regularly_spaced_3d_volume
    FROM read_parquet('{BASE}/idc_index.parquet') i
    JOIN read_parquet('{BASE}/volume_geometry_index.parquet') v
        ON i.SeriesInstanceUID = v.SeriesInstanceUID
    WHERE i.Modality = 'CT'
      AND v.regularly_spaced_3d_volume = TRUE
    LIMIT 10
""").df()

# 每个集合和模态中 3D 有效系列的比例
duckdb.sql(f"""
    SELECT i.collection_id, i.Modality,
           COUNT(*) as total,
           SUM(CASE WHEN v.regularly_spaced_3d_volume THEN 1 ELSE 0 END) as valid_3d,
           ROUND(100.0 * SUM(CASE WHEN v.regularly_spaced_3d_volume THEN 1 ELSE 0 END) / COUNT(*), 1) as pct_valid
    FROM read_parquet('{BASE}/idc_index.parquet') i
    JOIN read_parquet('{BASE}/volume_geometry_index.parquet') v
        ON i.SeriesInstanceUID = v.SeriesInstanceUID
    WHERE i.Modality IN ('CT', 'MR', 'PT')
    GROUP BY i.collection_id, i.Modality
    ORDER BY total DESC
    LIMIT 10
""").df()
```

`volume_geometry_index` 中的关键列：

| 列 | 类型 | 描述 |
|--------|------|-------------|
| `SeriesInstanceUID` | STRING | 连接键 |
| `single_orientation` | BOOLEAN | 所有实例共享相同的 ImageOrientationPatient |
| `orthogonal_orientation` | BOOLEAN | 方向方向余弦正交 |
| `unique_slice_positions` | BOOLEAN | 无重复或重叠切片 |
| `consistent_pixel_spacing` | BOOLEAN | 所有实例共享相同的 PixelSpacing |
| `consistent_image_dimensions` | BOOLEAN | 所有实例共享相同的 Rows 和 Columns |
| `uniform_slice_spacing` | BOOLEAN | 连续切片之间的间距恒定 |
| `obliquity_degrees` | FLOAT | 切片法线与最近主轴之间的角度（0 = 纯轴向/矢状/冠状） |
| `regularly_spaced_3d_volume` | BOOLEAN | 复合标志：如果所有检查均通过则为 TRUE |

## RT 结构集

`rtstruct_index` 每个 RTSTRUCT 系列一行，包含聚合的 ROI 元数据。

```python
import duckdb

BASE = "https://storage.googleapis.com/idc-index-data-artifacts/current/release_artifacts"

# 带有 ROI 详情的 RTSTRUCT 系列
duckdb.sql(f"""
    SELECT i.collection_id, i.SeriesInstanceUID,
           r.total_rois, r.ROINames, r.RTROIInterpretedTypes,
           r.referenced_SeriesInstanceUID
    FROM read_parquet('{BASE}/idc_index.parquet') i
    JOIN read_parquet('{BASE}/rtstruct_index.parquet') r
        ON i.SeriesInstanceUID = r.SeriesInstanceUID
    WHERE i.Modality = 'RTSTRUCT'
    LIMIT 5
""").df()

# 拥有最多 RTSTRUCT 系列的集合
duckdb.sql(f"""
    SELECT i.collection_id,
           COUNT(*) as rtstruct_series,
           ROUND(AVG(r.total_rois), 1) as avg_rois_per_struct
    FROM read_parquet('{BASE}/idc_index.parquet') i
    JOIN read_parquet('{BASE}/rtstruct_index.parquet') r
        ON i.SeriesInstanceUID = r.SeriesInstanceUID
    GROUP BY i.collection_id
    ORDER BY rtstruct_series DESC
    LIMIT 10
""").df()
```

`rtstruct_index` 中的关键列：

| 列 | 类型 | 描述 |
|--------|------|-------------|
| `SeriesInstanceUID` | STRING | 连接键（即 RTSTRUCT 系列） |
| `total_rois` | INTEGER | 结构集中的 ROI 数量 |
| `ROINames` | STRING (array) | 不同的 ROI 名称（例如 `["GTV", "Heart", "PTV"]`） |
| `ROIGenerationAlgorithms` | STRING (array) | 不同的生成算法（例如 `["AUTOMATIC", "MANUAL"]`） |
| `RTROIInterpretedTypes` | STRING (array) | 不同的 ROI 类型（例如 `["GTV", "ORGAN", "PTV"]`） |
| `referenced_SeriesInstanceUID` | STRING | 所引用源图像系列的 SeriesInstanceUID |

## 固定到特定版本

```python
import duckdb

# 使用特定数据版本而非 'current'
VERSION = "23.10.1"
BASE = f"https://storage.googleapis.com/idc-index-data-artifacts/{VERSION}/release_artifacts"

duckdb.sql(f"SELECT COUNT(*) FROM read_parquet('{BASE}/idc_index.parquet')").df()
```

## 资源

- idc-index-data 发布：https://github.com/ImagingDataCommons/idc-index-data/releases
- idc-index 文档：https://idc-index.readthedocs.io/
- IDC 门户：https://portal.imaging.datacommons.cancer.gov/
