---
name: imaging-data-commons
description: 使用 idc-index 查询并下载来自美国国家癌症研究所影像数据共享中心（NCI Imaging Data Commons）的公开癌症影像数据。用于访问大规模放射学（CT、MR、PET）和病理学数据集，以进行 AI 训练或研究。无需身份验证。通过元数据进行查询，在浏览器中可视化，检查许可。
license: 本技能基于 MIT 许可证提供。IDC 数据本身有各自的许可（主要为 CC-BY，部分为 CC-NC），在使用数据时必须遵守。
metadata:
    version: 1.4.0
    skill-author: Andrey Fedorov, @fedorov
    idc-index: "0.11.14"
    idc-data-version: "v23"
    repository: https://github.com/ImagingDataCommons/idc-claude-skill
---

# 影像数据共享中心 (Imaging Data Commons)

## 概述

使用 `idc-index` Python 包查询并下载来自美国国家癌症研究所影像数据共享中心（NCI Imaging Data Commons）的公开癌症影像数据。数据访问无需身份验证。

**当前 IDC 数据版本：v23**（始终通过 `IDCClient().get_idc_version()` 进行验证）

**主要工具：** `idc-index`（[GitHub](https://github.com/imagingdatacommons/idc-index)）

**关键——检查包版本并在需要时升级（请先运行此代码）：**

```python
import idc_index

REQUIRED_VERSION = "0.11.14"  # 必须与本文件的 metadata.idc-index 一致
installed = idc_index.__version__

if installed < REQUIRED_VERSION:
    print(f"正在将 idc-index 从 {installed} 升级到 {REQUIRED_VERSION}...")
    import subprocess
    subprocess.run(["pip3", "install", "--upgrade", "--break-system-packages", "idc-index"], check=True)
    print("升级完成。请重启 Python 以使用新版本。")
else:
    print(f"idc-index {installed} 满足要求（{REQUIRED_VERSION}）")
```

**验证 IDC 数据版本并检查当前数据规模：**

```python
from idc_index import IDCClient
client = IDCClient()

# 验证 IDC 数据版本（应为 "v23"）
print(f"IDC 数据版本：{client.get_idc_version()}")

# 获取集合数量和总序列数
stats = client.sql_query("""
    SELECT
        COUNT(DISTINCT collection_id) as collections,
        COUNT(DISTINCT analysis_result_id) as analysis_results,
        COUNT(DISTINCT PatientID) as patients,
        COUNT(DISTINCT StudyInstanceUID) as studies,
        COUNT(DISTINCT SeriesInstanceUID) as series,
        SUM(instanceCount) as instances,
        SUM(series_size_MB)/1000000 as size_TB
    FROM index
""")
print(stats)
```

**核心工作流程：**
1. 查询元数据 → `client.sql_query()`
2. 下载 DICOM 文件 → `client.download_from_selection()`
3. 在浏览器中可视化 → `client.get_viewer_URL(seriesInstanceUID=...)`

## 何时使用本技能

- 查找公开可用的放射学（CT、MR、PET）或病理学（切片显微镜）图像
- 按癌症类型、模态、解剖部位或其他元数据选择图像子集
- 从 IDC 下载 DICOM 数据
- 在用于研究或商业应用之前检查数据许可
- 在浏览器中可视化医学图像，无需本地 DICOM 查看器软件

## 快速导航

**核心章节（内联）：**
- IDC 数据模型 - 集合和分析结果层次结构
- 索引表 - 可用表和连接模式
- 安装 - 包设置和版本验证
- 核心功能 - 基本 API 模式（查询、下载、可视化、许可、引用、批处理）
- 最佳实践 - 使用指南
- 故障排除 - 常见问题和解决方案

**参考指南（按需加载）：**

| 指南 | 何时加载 |
|------|----------|
| `index_tables_guide.md` | 复杂 JOIN、模式发现、DataFrame 访问 |
| `use_cases.md` | 端到端工作流程示例（训练数据集、批量下载） |
| `sql_patterns.md` | 用于过滤器发现、注释、大小估算的快速 SQL 模式 |
| `clinical_data_guide.md` | 临床/表格数据、影像+临床连接、值映射 |
| `cloud_storage_guide.md` | 直接 S3/GCS 访问、版本控制、UUID 映射 |
| `dicomweb_guide.md` | DICOMweb 端点、PACS 集成 |
| `digital_pathology_guide.md` | 切片显微镜（SM）、注释（ANN）、病理学工作流程 |
| `bigquery_guide.md` | 完整 DICOM 元数据、私有元素（需要 GCP） |
| `cli_guide.md` | 命令行工具（`idc download`、清单文件） |
| `parquet_access_guide.md` | 通过 GCS 直接查询 Parquet（无需安装 idc-index） |

## IDC 数据模型

IDC 在标准 DICOM 层次结构（患者 → 检查 → 序列 → 实例）之上增加了两个分组级别：

- **collection_id**：按疾病、模态或研究重点对患者进行分组（例如 `tcga_luad`、`nlst`）。一个患者恰好属于一个集合。
- **analysis_result_id**：标识跨一个或多个原始集合的派生对象（分割、注释、影像组学特征）。

使用 `collection_id` 查找原始影像数据，可能包含与图像一起提交的注释；使用 `analysis_result_id` 查找 AI 生成或专家注释。

**查询的关键标识符：**
| 标识符 | 范围 | 用于 |
|--------|------|------|
| `collection_id` | 数据集分组 | 按项目/研究筛选 |
| `PatientID` | 患者 | 按患者分组图像 |
| `StudyInstanceUID` | DICOM 检查 | 相关序列的分组、可视化 |
| `SeriesInstanceUID` | DICOM 序列 | 相关序列的分组、可视化 |

## 索引表

`idc-index` 包提供多个元数据索引表，可通过 SQL 或 pandas DataFrame 访问。

**完整的索引表文档：** 使用 https://idc-index.readthedocs.io/en/latest/indices_reference.html 快速查看可用表和列，无需执行任何代码。

**重要提示：** 使用 `client.indices_overview` 获取当前表描述和列模式。这是可用列及其类型的权威来源——在编写 SQL 或探索数据结构时，请始终查询它。

### 可用表

| 表名 | 行粒度 | 加载方式 | 描述 |
|------|--------|----------|------|
| `index` | 1 行 = 1 个 DICOM 序列 | 自动 | 所有当前 IDC 数据的主要元数据 |
| `prior_versions_index` | 1 行 = 1 个 DICOM 序列 | 自动 | 先前 IDC 版本中的序列；用于下载已弃用的数据 |
| `collections_index` | 1 行 = 1 个集合 | fetch_index() | 集合级元数据和描述 |
| `analysis_results_index` | 1 行 = 1 个分析结果集合 | fetch_index() | 派生数据集（注释、分割）的元数据 |
| `clinical_index` | 1 行 = 1 个临床数据列 | fetch_index() | 将临床表列映射到集合的字典 |
| `sm_index` | 1 行 = 1 个切片显微镜序列 | fetch_index() | 切片显微镜（病理学）序列元数据 |
| `sm_instance_index` | 1 行 = 1 个切片显微镜实例 | fetch_index() | 切片显微镜的实例级（SOPInstanceUID）元数据 |
| `seg_index` | 1 行 = 1 个 DICOM 分割序列 | fetch_index() | 分割元数据：算法、分割数量、源图像序列引用 |
| `ann_index` | 1 行 = 1 个 DICOM ANN 序列 | fetch_index() | 显微镜批量简单注释序列元数据；引用注释的图像序列 |
| `ann_group_index` | 1 行 = 1 个注释组 | fetch_index() | 详细注释组元数据：图形类型、注释数量、属性代码、算法 |
| `contrast_index` | 1 行 = 1 个含对比剂信息的序列 | fetch_index() | 对比剂元数据：剂名称、成分、给药途径（CT、MR、PT、XA、RF） |
| `volume_geometry_index` | 1 行 = 1 个 CT/MR/PT 序列 | fetch_index() | 单帧 CT、MR 和 PT 序列的 3D 体积几何验证；方向、间距、尺寸和切片位置的布尔检查；复合标志 `regularly_spaced_3d_volume` |
| `rtstruct_index` | 1 行 = 1 个 RTSTRUCT 序列 | fetch_index() | RT 结构集元数据：总 ROI 数量、ROI 名称、生成算法、解释类型以及引用的图像序列 UID |

**自动** = 实例化 `IDCClient()` 时自动加载
**fetch_index()** = 需要调用 `client.fetch_index("table_name")` 加载

### 连接表

**关键列未明确标记，以下是可以用于连接的部分列。**

| 连接列 | 表 | 用例 |
|--------|-----|------|
| `collection_id` | index、prior_versions_index、collections_index、clinical_index | 将序列链接到集合元数据或临床数据 |
| `SeriesInstanceUID` | index、prior_versions_index、sm_index、sm_instance_index | 跨表链接序列；连接到切片显微镜详细信息 |
| `StudyInstanceUID` | index、prior_versions_index | 跨当前和历史数据链接检查 |
| `PatientID` | index、prior_versions_index | 跨当前和历史数据链接患者 |
| `analysis_result_id` | index、analysis_results_index | 将序列链接到分析结果元数据（注释、分割） |
| `source_DOI` | index、analysis_results_index | 按出版物 DOI 链接 |
| `crdc_series_uuid` | index、prior_versions_index | 按 CRDC 唯一标识符链接 |
| `Modality` | index、prior_versions_index | 按影像模态筛选 |
| `SeriesInstanceUID` | index、seg_index、ann_index、ann_group_index、contrast_index | 将分割/注释/对比剂序列链接到其索引元数据 |
| `segmented_SeriesInstanceUID` | seg_index → index | 将分割链接到其源图像序列（连接 seg_index.segmented_SeriesInstanceUID = index.SeriesInstanceUID） |
| `referenced_SeriesInstanceUID` | ann_index → index | 将注释链接到其源图像序列（连接 ann_index.referenced_SeriesInstanceUID = index.SeriesInstanceUID） |
| `SeriesInstanceUID` | index、volume_geometry_index | 将序列链接到其 3D 几何验证结果（连接 index.SeriesInstanceUID = volume_geometry_index.SeriesInstanceUID） |
| `SeriesInstanceUID` / `referenced_SeriesInstanceUID` | index、rtstruct_index | 将 RTSTRUCT 序列链接到其元数据（index.SeriesInstanceUID = rtstruct_index.SeriesInstanceUID）；使用 rtstruct_index.referenced_SeriesInstanceUID 查找源图像序列 |

**注意：** `Subjects`、`Updated` 和 `Description` 在多个表中出现，但含义不同（计数与标识符，不同的更新上下文）。

有关详细的连接示例、模式发现模式、关键列参考和 DataFrame 访问，请参阅 `references/index_tables_guide.md`。

### 临床数据访问

```python
# 获取临床索引（同时下载临床数据表）
client.fetch_index("clinical_index")

# 查询临床索引以查找可用的表及其列
tables = client.sql_query("SELECT DISTINCT table_name, column_label FROM clinical_index")

# 将特定临床表加载为 DataFrame
clinical_df = client.get_clinical_table("table_name")
```

有关详细工作流程（包括值映射模式以及将临床数据与影像连接），请参阅 `references/clinical_data_guide.md`。

## 数据访问选项

| 方法 | 需要认证 | 最适合 |
|------|----------|--------|
| `idc-index` | 否 | 关键查询和下载（推荐） |
| 直接 Parquet (GCS) | 否 | 无需安装 idc-index 的快速查询；始终使用最新数据 |
| IDC Portal | 否 | 交互式探索、手动选择、基于浏览器的下载 |
| BigQuery | 是（GCP 账户） | 复杂查询、完整 DICOM 元数据 |
| DICOMweb 代理 | 否 | 通过 DICOMweb API 进行工具集成 |
| 云存储 (S3/GCS) | 否 | 直接文件访问、批量下载、自定义管道 |

**云存储组织方式**

IDC 将所有 DICOM 文件存放在公共云存储桶中，在 AWS S3 和 Google Cloud Storage 之间进行镜像。文件按 CRDC UUID（而非 DICOM UID）组织以支持版本控制。

| 桶 (AWS / GCS) | 许可 | 内容 |
|----------------|------|------|
| `idc-open-data` / `idc-open-data` | 无商业限制 | >90% 的 IDC 数据 |
| `idc-open-data-two` / `idc-open-idc1` | 无商业限制 | 可能包含头部扫描的集合 |
| `idc-open-data-cr` / `idc-open-cr` | 商业使用受限 (CC BY-NC) | 约 4% 的数据 |

文件存储为 `<crdc_series_uuid>/<crdc_instance_uuid>.dcm`。可通过 AWS CLI、gsutil 或 s5cmd 以匿名访问方式免费访问（无出站费用）。S3 URL 使用索引中的 `series_aws_url` 列；GCS 使用相同的路径结构。

有关桶详细信息、访问命令、UUID 映射和版本控制，请参阅 `references/cloud_storage_guide.md`。

**DICOMweb 访问**

IDC 数据通过 DICOMweb 接口（Google Cloud Healthcare API 实现）提供，用于与 PACS 系统和兼容 DICOMweb 的工具集成。

| 端点 | 认证 | 用例 |
|------|------|------|
| 公共代理 | 否 | 测试、适度查询、每日配额 |
| Google Healthcare | 是 (GCP) | 生产使用、更高配额 |

有关端点 URL、代码示例、支持的操作和实现细节，请参阅 `references/dicomweb_guide.md`。

**直接 Parquet 访问**

所有 idc-index 元数据表都作为 Parquet 文件发布到公共 GCS 桶 (`idc-index-data-artifacts`)，具有不受限制的 CORS。这使得无需安装 idc-index 即可使用 DuckDB 或 pandas 进行查询，包括跨表连接以及针对 `volume_geometry_index` 和 `rtstruct_index` 的查询。

有关 URL 模式、可用文件和 DuckDB 查询示例，请参阅 `references/parquet_access_guide.md`。

## 安装和设置

**必需（基本访问）：**
```bash
pip install --upgrade idc-index
```

**重要提示：** 新的 IDC 数据发布将始终触发新版本的 `idc-index`。除非需要旧版本以实现可重复性，否则安装时务必使用 `--upgrade` 标志。

**重要提示：** 当前 IDC 数据版本为 v23。请始终验证您的版本：
```python
print(client.get_idc_version())  # 应返回 "v23"
```
如果您看到旧版本，请使用以下命令升级：`pip install --upgrade idc-index`

**已测试版本：** idc-index 0.11.14（IDC 数据版本 v23）

**可选（用于数据分析）：**
```bash
pip install pandas numpy pydicom
```

## 核心功能

### 1. 数据发现和探索

发现 IDC 中可用的影像集合和数据：

```python
from idc_index import IDCClient

client = IDCClient()

# 从主要索引获取汇总统计信息
query = """
SELECT
  collection_id,
  COUNT(DISTINCT PatientID) as patients,
  COUNT(DISTINCT SeriesInstanceUID) as series,
  SUM(series_size_MB) as size_mb
FROM index
GROUP BY collection_id
ORDER BY patients DESC
"""
collections_summary = client.sql_query(query)

# 获取更丰富的集合元数据，使用 collections_index
client.fetch_index("collections_index")
collections_info = client.sql_query("""
    SELECT collection_id, CancerTypes, TumorLocations, Species, Subjects, SupportingData
    FROM collections_index
""")

# 获取分析结果（注释、分割），使用 analysis_results_index
client.fetch_index("analysis_results_index")
analysis_info = client.sql_query("""
    SELECT analysis_result_id, analysis_result_title, Subjects, Collections, Modalities
    FROM analysis_results_index
""")
```

**`collections_index`** 提供每个集合的精选元数据：癌症类型、肿瘤位置、物种、受试者数量和支持数据类型——无需从主要索引进行聚合。

**`analysis_results_index`** 列出派生数据集（AI 分割、专家注释、影像组学特征）及其源集合和模态。

### 2. 使用 SQL 查询元数据

使用 SQL 查询 IDC 迷你索引以查找特定数据集。

**首先，探索可用筛选列的值：**
```python
from idc_index import IDCClient

client = IDCClient()

# 检查存在哪些 Modality 值
modalities = client.sql_query("""
    SELECT DISTINCT Modality, COUNT(*) as series_count
    FROM index
    GROUP BY Modality
    ORDER BY series_count DESC
""")
print(modalities)

# 检查 MR 模态存在哪些 BodyPartExamined 值
body_parts = client.sql_query("""
    SELECT DISTINCT BodyPartExamined, COUNT(*) as series_count
    FROM index
    WHERE Modality = 'MR' AND BodyPartExamined IS NOT NULL
    GROUP BY BodyPartExamined
    ORDER BY series_count DESC
    LIMIT 20
""")
print(body_parts)
```

**然后使用验证过的筛选值进行查询：**
```python
# 查找乳腺 MRI 扫描（使用上述探索中的实际值）
results = client.sql_query("""
    SELECT
      collection_id,
      PatientID,
      SeriesInstanceUID,
      Modality,
      SeriesDescription,
      license_short_name
    FROM index
    WHERE Modality = 'MR'
      AND BodyPartExamined = 'BREAST'
    LIMIT 20
""")

# 将结果作为 pandas DataFrame 访问
for idx, row in results.iterrows():
    print(f"患者：{row['PatientID']}，序列：{row['SeriesInstanceUID']}")
```

**要按癌症类型筛选，请与 `collections_index` 连接：**
```python
client.fetch_index("collections_index")
results = client.sql_query("""
    SELECT i.collection_id, i.PatientID, i.SeriesInstanceUID, i.Modality
    FROM index i
    JOIN collections_index c ON i.collection_id = c.collection_id
    WHERE c.CancerTypes LIKE '%Breast%'
      AND i.Modality = 'MR'
    LIMIT 20
""")
```

**可用的元数据字段**（使用 `client.indices_overview` 获取完整列表）：
- 标识符：collection_id、PatientID、StudyInstanceUID、SeriesInstanceUID
- 影像：Modality、BodyPartExamined、Manufacturer、ManufacturerModelName
- 临床：PatientAge、PatientSex、StudyDate
- 描述：StudyDescription、SeriesDescription
- 许可：license_short_name

**注意：** 癌症类型位于 `collections_index.CancerTypes` 中，不在主 `index` 表中。

### 3. 下载 DICOM 文件

高效地从 IDC 云存储下载影像数据：

**下载整个集合：**
```python
from idc_index import IDCClient

client = IDCClient()

# 下载小集合 (RIDER Pilot ~1GB)
client.download_from_selection(
    collection_id="rider_pilot",
    downloadDir="./data/rider"
)
```

**下载特定序列：**
```python
# 首先查询序列 UID
series_df = client.sql_query("""
    SELECT SeriesInstanceUID
    FROM index
    WHERE Modality = 'CT'
      AND BodyPartExamined = 'CHEST'
      AND collection_id = 'nlst'
    LIMIT 5
""")

# 仅下载这些序列
client.download_from_selection(
    seriesInstanceUID=list(series_df['SeriesInstanceUID'].values),
    downloadDir="./data/lung_ct"
)
```

**自定义目录结构：**

默认 `dirTemplate`：`%collection_id/%PatientID/%StudyInstanceUID/%Modality_%SeriesInstanceUID`

```python
# 简化层次结构（省略 StudyInstanceUID 级别）
client.download_from_selection(
    collection_id="tcga_luad",
    downloadDir="./data",
    dirTemplate="%collection_id/%PatientID/%Modality"
)
# 结果：./data/tcga_luad/TCGA-05-4244/CT/

# 扁平结构（所有文件在一个目录中）
client.download_from_selection(
    seriesInstanceUID=list(series_df['SeriesInstanceUID'].values),
    downloadDir="./data/flat",
    dirTemplate=""
)
# 结果：./data/flat/*.dcm
```

**下载的文件名：**

单个 DICOM 文件使用其 CRDC 实例 UUID 命名：`<crdc_instance_uuid>.dcm`（例如 `0d73f84e-70ae-4eeb-96a0-1c613b5d9229.dcm`）。这种基于 UUID 的命名：
- 支持版本跟踪（文件内容更改时 UUID 会更改）
- 匹配云存储组织方式（`s3://idc-open-data/<crdc_series_uuid>/<crdc_instance_uuid>.dcm`）
- 与 DICOM UID (SOPInstanceUID) 不同，后者保留在文件元数据中

要识别文件，请在查询中使用 `crdc_instance_uuid` 列或从文件中读取 DICOM 元数据 (SOPInstanceUID)。

### 命令行下载

`idc download` 命令提供命令行访问下载功能，无需编写 Python 代码。安装 `idc-index` 后即可使用。

**自动检测输入类型：** 清单文件路径或标识符（collection_id、PatientID、StudyInstanceUID、SeriesInstanceUID、crdc_series_uuid）。

```bash
# 下载整个集合
idc download rider_pilot --download-dir ./data

# 按 UID 下载特定序列
idc download "1.3.6.1.4.1.9328.50.1.69736" --download-dir ./data

# 下载多个项目（逗号分隔）
idc download "tcga_luad,tcga_lusc" --download-dir ./data

# 从清单文件下载（自动检测）
idc download manifest.txt --download-dir ./data
```

**选项：**

| 选项 | 描述 |
|------|------|
| `--download-dir` | 输出目录（默认：当前目录） |
| `--dir-template` | 目录层次结构模板（默认：`%collection_id/%PatientID/%StudyInstanceUID/%Modality_%SeriesInstanceUID`） |
| `--log-level` | 详细程度：debug、info、warning、error、critical |

**清单文件：**

清单文件包含 S3 URL（每行一个），可以：
- 从 IDC Portal 群组选择后导出
- 由协作者共享以实现可重复的数据访问
- 通过编程方式根据查询结果生成

格式（每行一个 S3 URL）：
```
s3://idc-open-data/cb09464a-c5cc-4428-9339-d7fa87cfe837/*
s3://idc-open-data/88f3990d-bdef-49cd-9b2b-4787767240f2/*
```

**示例：从 Python 查询生成清单：**

```python
from idc_index import IDCClient

client = IDCClient()

# 查询序列 URL
results = client.sql_query("""
    SELECT series_aws_url
    FROM index
    WHERE collection_id = 'rider_pilot' AND Modality = 'CT'
""")

# 保存为清单文件
with open('ct_manifest.txt', 'w') as f:
    for url in results['series_aws_url']:
        f.write(url + '\n')
```

然后下载：
```bash
idc download ct_manifest.txt --download-dir ./ct_data
```

### 4. 可视化 IDC 图像

在浏览器中查看 DICOM 数据，无需下载：

```python
from idc_index import IDCClient
import webbrowser

client = IDCClient()

# 首先查询获取有效的 UID
results = client.sql_query("""
    SELECT SeriesInstanceUID, StudyInstanceUID
    FROM index
    WHERE collection_id = 'rider_pilot' AND Modality = 'CT'
    LIMIT 1
""")

# 查看单个序列
viewer_url = client.get_viewer_URL(seriesInstanceUID=results.iloc[0]['SeriesInstanceUID'])
webbrowser.open(viewer_url)

# 查看检查中的所有序列（对于多序列检查如 MRI 协议很有用）
viewer_url = client.get_viewer_URL(studyInstanceUID=results.iloc[0]['StudyInstanceUID'])
webbrowser.open(viewer_url)
```

该方法自动为放射学选择 OHIF v3，为切片显微镜选择 SLIM。按检查查看在 DICOM 检查包含多个序列（例如单次 MRI 会话中的 T1、T2、DWI 序列）时非常有用。

### 5. 理解并检查许可

使用前检查数据许可（对于商业应用至关重要）：

```python
from idc_index import IDCClient

client = IDCClient()

# 检查所有集合的许可
query = """
SELECT DISTINCT
  collection_id,
  license_short_name,
  COUNT(DISTINCT SeriesInstanceUID) as series_count
FROM index
GROUP BY collection_id, license_short_name
ORDER BY collection_id
"""

licenses = client.sql_query(query)
print(licenses)
```

**IDC 中的许可类型：**
- **CC BY 4.0** / **CC BY 3.0**（约 97% 的数据）——允许商业使用，需注明出处
- **CC BY-NC 4.0** / **CC BY-NC 3.0**（约 3% 的数据）——仅限非商业使用
- **自定义许可**（罕见）——某些集合有特定条款（例如 NLM 条款和条件）

**重要提示：** 在出版物或商业应用中使用 IDC 数据前，请务必检查许可。每个 DICOM 文件都在元数据中标记了其特定许可。

### 生成引用以注明出处

`source_DOI` 列包含指向描述数据生成方式的出版物的 DOI。要满足署名要求，请使用 `citations_from_selection()` 生成格式正确的引用：

```python
from idc_index import IDCClient

client = IDCClient()

# 获取集合的引用（默认 APA 格式）
citations = client.citations_from_selection(collection_id="rider_pilot")
for citation in citations:
    print(citation)

# 获取特定序列的引用
results = client.sql_query("""
    SELECT SeriesInstanceUID FROM index
    WHERE collection_id = 'tcga_luad' LIMIT 5
""")
citations = client.citations_from_selection(
    seriesInstanceUID=list(results['SeriesInstanceUID'].values)
)

# 替代格式：BibTeX（用于 LaTeX 文档）
bibtex_citations = client.citations_from_selection(
    collection_id="tcga_luad",
    citation_format=IDCClient.CITATION_FORMAT_BIBTEX
)
```

**参数：**
- `collection_id`：按集合筛选
- `patientId`：按患者 ID 筛选
- `studyInstanceUID`：按检查 UID 筛选
- `seriesInstanceUID`：按序列 UID 筛选
- `citation_format`：使用 `IDCClient.CITATION_FORMAT_*` 常量：
  - `CITATION_FORMAT_APA`（默认）——APA 格式
  - `CITATION_FORMAT_BIBTEX`——用于 LaTeX 的 BibTeX
  - `CITATION_FORMAT_JSON`——CSL JSON
  - `CITATION_FORMAT_TURTLE`——RDF Turtle

**最佳实践：** 在发布使用 IDC 数据的结果时，请包含生成的引用，以正确注明数据来源并满足许可要求。

### 6. 批处理与筛选

高效处理大型数据集并应用筛选：

```python
from idc_index import IDCClient
import pandas as pd

client = IDCClient()

# 查找来自 GE 扫描仪的胸部 CT 扫描
query = """
SELECT
  SeriesInstanceUID,
  PatientID,
  collection_id,
  ManufacturerModelName
FROM index
WHERE Modality = 'CT'
  AND BodyPartExamined = 'CHEST'
  AND Manufacturer = 'GE MEDICAL SYSTEMS'
  AND license_short_name = 'CC BY 4.0'
LIMIT 100
"""

results = client.sql_query(query)

# 保存清单以供后续使用
results.to_csv('lung_ct_manifest.csv', index=False)

# 分批下载以避免超时
batch_size = 10
for i in range(0, len(results), batch_size):
    batch = results.iloc[i:i+batch_size]
    client.download_from_selection(
        seriesInstanceUID=list(batch['SeriesInstanceUID'].values),
        downloadDir=f"./data/batch_{i//batch_size}"
    )
```

### 7. 使用 BigQuery 进行高级查询

对于需要完整 DICOM 元数据、复杂 JOIN、临床数据表或私有 DICOM 元素的查询，请使用 Google BigQuery。需要启用计费的 GCP 账户。

**快速参考：**
- 数据集：`bigquery-public-data.idc_current.*`
- 主表：`dicom_all`（组合元数据）
- 完整元数据：`dicom_metadata`（所有 DICOM 标签）
- 私有元素：`OtherElements` 列（供应商特定标签，如扩散 b 值）

有关设置、表模式、查询模式、私有元素访问和成本优化，请参阅 `references/bigquery_guide.md`。

**在使用 BigQuery 之前**，请始终检查专门的索引表是否已包含所需的元数据：
1. 使用 `client.indices_overview` 或 [idc-index 索引参考](https://idc-index.readthedocs.io/en/latest/indices_reference.html) 发现所有可用表及其列
2. 获取相关索引：`client.fetch_index("table_name")`
3. 使用 `client.sql_query()` 本地查询（免费，无需 GCP 账户）

常见的专门索引：`seg_index`（分割）、`ann_index` / `ann_group_index`（显微镜注释）、`sm_index`（切片显微镜）、`collections_index`（集合元数据）。仅当需要任何索引中都不存在的私有 DICOM 元素或属性时，才使用 BigQuery。

**需要 BigQuery 的用例（无 idc-index 等效项）：**
- **按分割段解剖结构搜索**——`seg_index` 提供序列级 SEG 元数据，但 BigQuery 的 `segmentations` 表单独公开每个分段及其 DICOM 编码结构名称（例如，查找所有包含"肝脏"或"肿瘤"分段的 SEG 序列）
- **来自 SR 的定量测量**——BigQuery 的 `quantitative_measurements` 表包含从 DICOM SR TID1500 对象中预先提取的影像组学特征（体积、直径、形状描述符、纹理、强度统计）；无 idc-index 等效项
- **来自 SR 的定性测量**——BigQuery 的 `qualitative_measurements` 表包含来自 DICOM SR TID1500 的编码评估（恶性分级、钙化、纹理、边缘）；无 idc-index 等效项

有关这些表的模式、列描述和查询示例，请参阅 `references/bigquery_guide.md`。

### 8. 工具选择指南

| 任务 | 工具 | 参考 |
|------|------|------|
| 编程查询和下载 | `idc-index` | 本文档 |
| 交互式探索 | IDC Portal | https://portal.imaging.datacommons.cancer.gov/ |
| 复杂元数据查询 | BigQuery | `references/bigquery_guide.md` |
| 3D 可视化和分析 | SlicerIDCBrowser | https://github.com/ImagingDataCommons/SlicerIDCBrowser |

**默认选择：** 大多数任务使用 `idc-index`（无需认证、API 简单、支持批量下载）。

### 9. 与分析管道集成

将 IDC 数据集成到影像分析工作流程中：

**读取已下载的 DICOM 文件：**
```python
import pydicom
import os

# 从下载的序列目录读取 DICOM 文件
series_dir = "./data/rider/rider_pilot/RIDER-1007893286/CT_1.3.6.1..."

dicom_files = [os.path.join(series_dir, f) for f in os.listdir(series_dir)
               if f.endswith('.dcm')]

# 加载第一张图像
ds = pydicom.dcmread(dicom_files[0])
print(f"患者 ID：{ds.PatientID}")
print(f"模态：{ds.Modality}")
print(f"图像形状：{ds.pixel_array.shape}")
```

**从 CT 序列构建 3D 体积：**
```python
import pydicom
import numpy as np
from pathlib import Path

def load_ct_series(series_path):
    """将 CT 序列加载为 3D numpy 数组"""
    files = sorted(Path(series_path).glob('*.dcm'))
    slices = [pydicom.dcmread(str(f)) for f in files]

    # 按切片位置排序
    slices.sort(key=lambda x: float(x.ImagePositionPatient[2]))

    # 堆叠为 3D 数组
    volume = np.stack([s.pixel_array for s in slices])

    return volume, slices[0]  # 返回体积和第一张切片以获取元数据

volume, metadata = load_ct_series("./data/lung_ct/series_dir")
print(f"体积形状：{volume.shape}")  # (z, y, x)
```

**与 SimpleITK 集成：**
```python
import SimpleITK as sitk
from pathlib import Path

# 读取 DICOM 序列
series_path = "./data/ct_series"
reader = sitk.ImageSeriesReader()
dicom_names = reader.GetGDCMSeriesFileNames(series_path)
reader.SetFileNames(dicom_names)
image = reader.Execute()

# 应用处理
smoothed = sitk.CurvatureFlow(image1=image, timeStep=0.125, numberOfIterations=5)

# 保存为 NIfTI
sitk.WriteImage(smoothed, "processed_volume.nii.gz")
```

## 常见用例

有关完整的端到端工作流程示例，请参阅 `references/use_cases.md`，包括：
- 从肺部 CT 扫描构建深度学习训练数据集
- 比较不同扫描仪制造商的图像质量
- 在浏览器中预览数据后再下载
- 为商业使用进行许可感知的批量下载

## 最佳实践

- **在生成响应前验证 IDC 版本** - 在会话开始时始终调用 `client.get_idc_version()`，以确认您使用的是预期的数据版本（当前为 v23）。如果使用旧版本，建议执行 `pip install --upgrade idc-index`
- **使用前检查许可** - 始终查询 `license_short_name` 字段并遵守许可条款（CC BY 与 CC BY-NC）
- **生成引用以注明出处** - 使用 `citations_from_selection()` 从 `source_DOI` 值获取格式正确的引用；在出版物中包含这些引用
- **从小查询开始** - 探索时使用 `LIMIT` 子句，以避免长时间下载并理解数据结构
- **简单查询使用迷你索引** - 仅当需要全面元数据或复杂 JOIN 时才使用 BigQuery
- **使用 dirTemplate 组织下载** - 使用有意义的目录结构，如 `%collection_id/%PatientID/%Modality`
- **缓存查询结果** - 将 DataFrame 保存为 CSV 文件，以避免重复查询并确保可重复性
- **先估算大小** - 下载前检查集合大小——某些集合大小可达数 TB！
- **保存清单** - 始终保存包含序列 UID 的查询结果，以确保可重复性和数据溯源
- **阅读文档** - IDC 数据结构和元数据字段记录在 https://learn.canceridc.dev/
- **使用 IDC 论坛** - 在 https://discourse.canceridc.dev/ 上搜索问题/答案并向 IDC 维护者和用户提问

## 故障排除

**问题：`ModuleNotFoundError: No module named 'idc_index'`**
- **原因：** 未安装 idc-index 包
- **解决方案：** 使用 `pip install --upgrade idc-index` 安装

**问题：下载因连接超时而失败**
- **原因：** 网络不稳定或下载规模过大
- **解决方案：**
  - 下载较小的批次（例如一次 10-20 个序列）
  - 检查网络连接
  - 使用 `dirTemplate` 按批次组织下载
  - 实现带延迟的重试逻辑

**问题：`BigQuery 配额超出` 或计费错误**
- **原因：** BigQuery 需要启用计费的 GCP 项目
- **解决方案：** 简单查询使用 idc-index 迷你索引（无需计费），或参阅 `references/bigquery_guide.md` 了解成本优化技巧

**问题：未找到序列 UID 或未返回数据**
- **原因：** UID 拼写错误、数据不在当前 IDC 版本中、或字段名错误
- **解决方案：**
  - 检查数据是否在当前 IDC 版本中（某些旧数据可能已弃用）
  - 首先使用 `LIMIT 5` 测试查询
  - 对照元数据模式文档检查字段名

**问题：下载的 DICOM 文件无法打开**
- **原因：** 下载损坏或查看器不兼容
- **解决方案：**
  - 检查 DICOM 对象类型（Modality 和 SOPClassUID 属性）——某些对象类型需要专用工具
  - 验证文件完整性（检查文件大小）
  - 使用 pydicom 验证：`pydicom.dcmread(file, force=True)`
  - 尝试不同的 DICOM 查看器（3D Slicer、Horos、RadiAnt、QuPath）
  - 重新下载序列

## 常见 SQL 查询模式

有关快速参考的 SQL 模式，请参阅 `references/sql_patterns.md`，包括：
- 筛选值发现（模态、身体部位、制造商）
- 注释和分割查询（包括 seg_index、ann_index 连接）
- 切片显微镜查询（sm_index 模式）
- 下载大小估算
- 临床数据链接

有关分割和注释的详细信息，另请参阅 `references/digital_pathology_guide.md`。

## 相关技能

以下技能补充了 IDC 工作流程，用于下游分析和可视化：

### DICOM 处理
- **pydicom** - 读取、写入和操作下载的 DICOM 文件。用于提取像素数据、读取元数据、匿名化和格式转换。处理 IDC 放射学数据（CT、MR、PET）时必不可少。

### 病理学和切片显微镜
有关兼容 DICOM 的工具（highdicom、wsidicom、TIA-Toolbox、Slim viewer），请参阅 `references/digital_pathology_guide.md`。

### 元数据可视化
- **matplotlib** - 低级绘图，用于完全自定义。用于创建总结 IDC 查询结果的静态图表（模态条形图、序列数量直方图等）。
- **seaborn** - 与 pandas 集成的统计可视化。用于快速探索 IDC 元数据分布、变量间关系以及具有美观默认设置的分类别比较。
- **plotly** - 交互式可视化。当需要悬停信息、缩放和平移来探索 IDC 元数据，或创建可嵌入网页的集合统计仪表板时使用。

### 数据探索
- ** exploratory-data-analysis** - 对科学数据文件进行全面的探索性数据分析。下载 IDC 数据后使用，以在分析前了解文件结构、质量和特征。

## 资源

### 模式参考（主要来源）

**始终使用 `client.indices_overview` 获取当前列模式。** 这可确保与已安装的 idc-index 版本保持一致：

```python
# 获取任何表的所有列名和类型
schema = client.indices_overview["index"]["schema"]
columns = [(c['name'], c['type'], c.get('description', '')) for c in schema['columns']]
```

### 参考文档

请参阅顶部的快速导航部分，了解完整的参考指南列表及其决策触发条件。

- **[indices_reference](https://idc-index.readthedocs.io/en/latest/indices_reference.html)** - 索引表的外部文档（可能领先于已安装的版本）

### 外部链接

- **IDC Portal**: https://portal.imaging.datacommons.cancer.gov/explore/
- **文档**: https://learn.canceridc.dev/
- **教程**: https://github.com/ImagingDataCommons/IDC-Tutorials
- **用户论坛**: https://discourse.canceridc.dev/
- **idc-index GitHub**: https://github.com/ImagingDataCommons/idc-index
- **引用**: Fedorov, A., et al. "National Cancer Institute Imaging Data Commons: Toward Transparency, Reproducibility, and Scalability in Imaging Artificial Intelligence." RadioGraphics 43.12 (2023). https://doi.org/10.1148/rg.230180

### 技能更新

本技能版本可在技能元数据中找到。要检查更新：
- 访问 [发布页面](https://github.com/ImagingDataCommons/idc-claude-skill/releases)
- 在 GitHub 上关注该仓库（关注 → 自定义 → 发布）
