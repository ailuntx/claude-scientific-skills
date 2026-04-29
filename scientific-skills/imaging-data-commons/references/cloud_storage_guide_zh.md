# IDC 云存储指南

IDC 将所有 DICOM 文件存储在 Google 云存储（GCS）和 AWS S3 之间镜像的公共云存储桶中。本指南涵盖存储桶组织、文件结构、访问方法和版本控制。

## 何时使用直接云存储访问

在以下场景使用直接存储桶访问：
- 需要并行传输实现最大下载性能
- 与云原生工作流集成（如在云虚拟机上运行分析）
- 通过 s5cmd 或 gsutil 等工具进行程序化访问
- 访问特定文件版本以确保可复现性

对于大多数场景，推荐使用更简单的 `idc-index`——它内部使用 s5cmd 从相同的 S3 存储桶下载，自动处理 UUID 查找。当需要原始文件访问、自定义并行化或构建云原生管道时，请使用直接云存储访问。

## 存储桶组织

IDC 根据许可协议和内容类型将数据组织到多个存储桶中。所有存储桶在 AWS 和 GCS 之间保持内容镜像且文件路径一致。

### 存储桶概览

| 用途 | AWS S3 存储桶 | GCS 存储桶 | 许可协议 | 内容 |
|---------|---------------|------------|---------|---------|
| 主数据 | `idc-open-data` | `idc-open-data` | 无商业限制 | >90% IDC 数据 |
| 头部扫描 | `idc-open-data-two` | `idc-open-idc1` | 无商业限制 | 可能包含头部影像的数据集 |
| 商业限制 | `idc-open-data-cr` | `idc-open-cr` | 商业使用受限 (CC BY-NC) | ~4% 数据 |

**注意：**
- 所有 AWS 存储桶位于 `us-east-1` 区域
- IDC v19 之前 GCS 使用 `public-datasets-idc`（现已被 `idc-open-data` 取代）
- 头部扫描存储桶为未来面部影像数据政策变更预留
- **重要** 使用 `idc-index` 获取许可信息——勿依赖存储桶名称！

### 为何需要多个存储桶？

1. **许可隔离**：具有商业使用限制的数据（CC BY-NC）隔离在 `idc-open-data-cr`/`idc-open-cr` 中，防止意外商业使用
2. **头部扫描处理**：被 TCIA 标记为可能包含头部扫描的数据集存放在独立存储桶（`idc-open-data-two`/`idc-open-idc1`）以备未来政策合规
3. **历史原因**：存储桶结构随 IDC 发展及与不同云项目合作而演变

## 存储桶内文件组织

文件按 CRDC UUID 而非 DICOM UID 组织，实现跨云服务商版本控制的同时保持路径一致性。

### 目录结构

```
<存储桶>/
└── <crdc_series_uuid>/
    ├── <crdc_instance_uuid_1>.dcm
    ├── <crdc_instance_uuid_2>.dcm
    └── ...
```

**示例路径：**
```
s3://idc-open-data/7a6b2389-53c6-4c5b-b07f-6d1ed4a3eed9/0d73f84e-70ae-4eeb-96a0-1c613b5d9229.dcm
```

- `7a6b2389-53c6-4c5b-b07f-6d1ed4a3eed9` = 序列 UUID（文件夹）
- `0d73f84e-70ae-4eeb-96a0-1c613b5d9229.dcm` = 实例 UUID（文件）

### CRDC UUID 与 DICOM UID 对比

| 标识符类型 | 格式 | 变更条件 | 用途 |
|-----------------|--------|--------------|---------|
| DICOM UID (如 SeriesInstanceUID) | 数字格式 (如 `1.3.6.1.4...`) | 永不改变 (包含在 DICOM 元数据中) | 临床识别、DICOMweb 查询 |
| CRDC UUID (如 crdc_series_uuid) | UUID 格式 (如 `e127d258-37c2-...`) | 内容变更时更新 | 文件路径、版本控制、可复现性 |

**核心洞察**：当序列内容变更（实例增删、元数据修正）时，单个 DICOM SeriesInstanceUID 在 IDC 不同版本中可能对应多个 CRDC 序列 UUID。CRDC UUID 唯一标识数据的特定版本。

### 将 DICOM UID 映射到文件路径

使用 `idc-index` 从 DICOM 标识符获取文件 URL：

```python
from idc_index import IDCClient

client = IDCClient()

# 获取序列的所有文件 URL
series_uid = "1.3.6.1.4.1.14519.5.2.1.6450.9002.217441095430480124587725641302"
urls = client.get_series_file_URLs(seriesInstanceUID=series_uid)

for url in urls[:3]:
    print(url)
# 返回 S3 URL 如：s3://idc-open-data/<crdc_series_uuid>/<crdc_instance_uuid>.dcm
```

或直接查询索引的 URL 列：

```python
# 获取序列级 URL（指向文件夹）
result = client.sql_query("""
    SELECT SeriesInstanceUID, series_aws_url
    FROM index
    WHERE collection_id = 'rider_pilot' AND Modality = 'CT'
    LIMIT 3
""")

print(result[['SeriesInstanceUID', 'series_aws_url']])
```

**索引中的可用 URL 列：**
- `series_aws_url`：序列文件夹的 S3 URL（如 `s3://idc-open-data/uuid/*`）

GCS URL 遵循相同路径结构——将 `s3://` 替换为 `gs://`（如 `gs://idc-open-data/uuid/*`）。使用 `idc-index` 下载方法时，GCS 访问由内部处理。

## 访问云存储

通过与 AWS 开放数据和 Google 公共数据项目合作，所有 IDC 存储桶支持免费出口（无下载费用）。无需身份验证。

### AWS S3 访问

**使用 AWS CLI（无需账户）：**
```bash
# 列出存储桶内容
aws s3 ls --no-sign-request s3://idc-open-data/

# 列出序列文件夹中的文件
aws s3 ls --no-sign-request s3://idc-open-data/7a6b2389-53c6-4c5b-b07f-6d1ed4a3eed9/

# 下载单个文件
aws s3 cp --no-sign-request \
    s3://idc-open-data/7a6b2389-53c6-4c5b-b07f-6d1ed4a3eed9/0d73f84e-70ae-4eeb-96a0-1c613b5d9229.dcm \
    ./local_file.dcm

# 下载整个序列文件夹
aws s3 cp --no-sign-request --recursive \
    s3://idc-open-data/7a6b2389-53c6-4c5b-b07f-6d1ed4a3eed9/ \
    ./series_folder/
```

**使用 s5cmd（批量下载更快）：**
```bash
# 安装 s5cmd
# macOS：brew install s5cmd
# Linux：从 https://github.com/peak/s5cmd/releases 下载

# 下载特定序列
s5cmd --no-sign-request cp 's3://idc-open-data/7a6b2389-53c6-4c5b-b07f-6d1ed4a3eed9/*' ./local_folder/

# 通过清单文件下载
s5cmd --no-sign-request run manifest.txt
```

**s5cmd 清单格式**：`s5cmd run` 命令要求每行一个 s5cmd 指令，而非仅 URL：
```
cp s3://idc-open-data/uuid1/instance1.dcm ./local_folder/
cp s3://idc-open-data/uuid1/instance2.dcm ./local_folder/
cp s3://idc-open-data/uuid2/instance3.dcm ./local_folder/
```

IDC 门户以此格式导出清单。编程创建清单时，应使用 `idc-index` 下载方法（内部处理此逻辑），而非手动构建清单。

### GCS 访问

**使用 gsutil：**
```bash
# 列出存储桶内容
gsutil ls gs://idc-open-data/

# 下载序列文件夹
gsutil -m cp -r gs://idc-open-data/7a6b2389-53c6-4c5b-b07f-6d1ed4a3eed9/ ./local_folder/
```

**使用 gcloud storage（新版 CLI）：**
```bash
gcloud storage cp -r gs://idc-open-data/7a6b2389-53c6-4c5b-b07f-6d1ed4a3eed9/ ./local_folder/
```

### Python 直接访问

```python
import s3fs
import gcsfs
from idc_index import IDCClient

# 首先从 idc-index 获取文件 URL
client = IDCClient()
result = client.sql_query("""
    SELECT series_aws_url
    FROM index
    WHERE collection_id = 'rider_pilot' AND Modality = 'CT'
    LIMIT 1
""")
# series_aws_url 格式如：s3://idc-open-data/<uuid>/*
series_url = result['series_aws_url'].iloc[0]
series_path = series_url.replace('s3://', '').rstrip('/*')  # 例如 "idc-open-data/<uuid>"

# AWS S3 访问
s3 = s3fs.S3FileSystem(anon=True)
files = s3.ls(series_path)
with s3.open(files[0], 'rb') as f:
    data = f.read()

# GCS 访问（路径结构与 AWS 相同）
gcs = gcsfs.GCSFileSystem(token='anon')
files = gcs.ls(series_path)
with gcs.open(files[0], 'rb') as f:
    data = f.read()
```

## 版本控制与可复现性

IDC 每 2-4 个月发布新数据版本。版本控制系统通过保留所有历史数据确保可复现性。

### 版本控制机制

1. **快照**：每个 IDC 版本（v1, v2, ..., v23 等）代表发布时的完整数据快照
2. **基于 UUID**：数据变更时分配新 CRDC UUID；旧 UUID 保持可访问
3. **累积存储桶**：所有版本共存于相同存储桶——旧序列文件夹永久保留

**版本变更场景：**
| 变更类型 | DICOM UID | CRDC UUID | 影响 |
|-------------|-----------|-----------|--------|
| 新增序列 | 新 | 新 | 存储桶中新增文件夹 |
| 序列新增实例 | 相同 | 新序列 UUID | 新建文件夹，实例可能重复 |
| 元数据修正 | 相同或新 | 新 | 新建包含更新文件的文件夹 |
| 序列移除 | 不适用 | 不适用 | 旧文件夹保留，但不出现在当前索引 |

**数据移除说明**：极少数情况（如数据所有者要求、PHI 事件），数据可能从 IDC 完全移除，包括所有历史版本。

**BigQuery 版本化数据集（仅元数据，非文件存储）：**

查询特定版本元数据时，BigQuery 提供版本化表。详见 `bigquery_guide.md`：
- `bigquery-public-data.idc_current` — 指向最新版本的别名
- `bigquery-public-data.idc_v23` — 特定版本（将 23 替换为目标版本）

### 复现历史分析

确保可复现性的最简单方法是保存分析时使用的 `crdc_series_uuid` 值：

```python
from idc_index import IDCClient
import json

client = IDCClient()

# 选择分析数据
selection = client.sql_query("""
    SELECT crdc_series_uuid
    FROM index
    WHERE collection_id = 'tcga_luad'
      AND Modality = 'CT'
    LIMIT 10
""")
series_uuids = list(selection['crdc_series_uuid'])

# 下载数据
client.download_from_selection(seriesInstanceUID=series_uuids, downloadDir="./data")

# 保存复现清单
manifest = {
    "crdc_series_uuids": series_uuids,
    "download_date": "2024-01-15",
    "idc_version": client.get_idc_version(),
    "description": "肺癌分析的 CT 扫描"
}
with open("analysis_manifest.json", "w") as f:
    json.dump(manifest, f, indent=2)

# 后续复现相同数据集：
with open("analysis_manifest.json") as f:
    manifest = json.load(f)
client.download_from_selection(
    seriesInstanceUID=manifest["crdc_series_uuids"],
    downloadDir="./reproduced_data"
)
```

由于 `crdc_series_uuid` 标识每个序列的不可变版本，保存这些 UUID 可保证后续能检索完全相同的文件。

## 存储桶、版本与其他访问方法的关系

### 数据覆盖范围对比

| 访问方法 | 包含存储桶 | 覆盖率 | 版本支持 |
|---------------|------------------|----------|----------|
| 直接存储桶访问 | 全部 3 个存储桶 | 100% | 所有历史版本 |
| `idc-index` 下载 | 全部 3 个存储桶 | 100% | 当前版本 + 历史索引 |
| IDC 门户 | 全部 3 个存储桶 | 100% | 仅当前版本 |
| DICOMweb 公共代理 | 全部 3 个存储桶 | 100% | 仅当前版本 |
| Google Healthcare DICOM | 仅 `idc-open-data` | ~96% | 仅当前版本 |

**重要**：Google Healthcare API DICOM 存储仅复制 `idc-open-data` 的数据。`idc-open-data-two` 和 `idc-open-data-cr` 中的数据（约占总量的 4%）无法通过 Google Healthcare DICOMweb 端点访问。

## 最佳实践

- **使用 `idc-index` 进行发现**：先查询元数据，再通过已知 UUID 访问存储桶
- **下载默认使用 AWS 存储桶**：需要时请求 GCS
- **保存清单**：存储 `series_aws_url` 或 `crdc_series_uuid` 值以确保可复现性
- **检查许可协议**：商业使用前查询 `license_short_name`；CC-NC 数据要求非商业用途
- **非复现场景使用当前版本**：`index` 表包含当前数据；仅需精确复现时使用 `prior_versions_index`

## 故障排除

### 问题：访问存储桶时提示"Access Denied"
- **原因**：使用了签名请求或错误的存储桶名称
- **解决方案**：AWS CLI 使用 `--no-sign-request` 标志，Python 库使用 `anon=True`

### 问题：预期路径找不到文件
- **原因**：使用了 DICOM UID 而非 CRDC UUID，或数据在新版本中已变更
- **解决方案**：通过 `idc-index` 查询当前 `series_aws_url`，或检查 `prior_versions_index` 获取历史路径

### 问题：下载文件与预期序列不匹配
- **原因**：序列在新版 IDC 中已修订
- **解决方案**：使用 `prior_versions_index` 查找所需确切版本；对比 `crdc_series_uuid` 值

### 问题：Google Healthcare DICOMweb 缺失部分数据
- **原因**：Google Healthcare 仅镜像 `idc-open-data` 存储桶（约 96% 数据）
- **解决方案**：使用 IDC 公共代理实现 100% 覆盖，或直接访问存储桶

## 资源

**IDC 文档：**
- [文件与元数据](https://learn.canceridc.dev/data/organization-of-data/files-and-metadata) - 存储桶组织详情
- [数据版本控制](https://learn.canceridc.dev/data/data-versioning) - 版本控制方案说明
- [解析 GUID 与 UUID](https://learn.canceridc.dev/data/organization-of-data
