# IDC 的 DICOMweb 指南

IDC 通过 Google Cloud Healthcare API 的 DICOM 存储提供 DICOMweb 访问。本指南涵盖具体实现细节和使用模式。

## 何时使用 DICOMweb

在以下场景中使用 DICOMweb：
- 与 PACS 系统或兼容 DICOMweb 的工具集成
- 流式传输元数据而无需下载完整文件
- 构建自定义查看器或 Web 应用程序
- 使用现有 DICOMweb 客户端库（如 OHIF、dicomweb-client 等）

对于大多数用例，`idc-index` 更简单且推荐使用。仅在明确需要 DICOMweb 协议时使用 DICOMweb。

## 端点

### 公共代理（无需认证）

```
https://proxy.imaging.datacommons.cancer.gov/current/viewer-only-no-downloads-see-tinyurl-dot-com-slash-3j3d9jyp/dicomWeb
```

- **100% 数据覆盖** - 包含所有存储桶中的全部 IDC 数据
- 自动指向最新 IDC 版本
- 新 IDC 版本发布时**立即更新**
- 基于 IP 的每日配额（适合测试和中等使用）
- 无需认证
- 只读访问
- 注：URL 中的 "viewer-only-no-downloads" 是历史命名，无实际功能含义

### Google Healthcare API（需认证）

```
https://healthcare.googleapis.com/v1/projects/nci-idc-data/locations/us-central1/datasets/idc/dicomStores/idc-store-v{VERSION}/dicomWeb
```

将 `{VERSION}` 替换为 IDC 发布版本号。获取当前版本：

```python
from idc_index import IDCClient
client = IDCClient()
print(client.get_idc_version())  # 例如 "23" 表示 v23
```

- **约 96% 数据覆盖** - 仅复制 `idc-open-data` 存储桶的数据（缺失其他存储桶约 4% 的数据）
- IDC 发布后 **1-2 周更新**
- 需认证并提供更高配额
- 性能更优（无代理路由）
- 每个发布版本对应新的版本化存储

详见下文[内容覆盖差异](#内容覆盖差异)和[认证](#google-healthcare-api-认证)部分。

## 内容覆盖差异

**重要提示：** 两个 DICOMweb 端点的数据覆盖范围不同。IDC 公共代理包含的数据**多于**需认证的 Google Healthcare 端点。

### 覆盖摘要

| 端点 | 覆盖率 | 缺失数据 |
|------|--------|----------|
| **IDC 公共代理** | 100% | 无 |
| **Google Healthcare API** | ~96% | ~4%（两个存储桶未复制） |

### Google Healthcare 缺失哪些数据？

Google Healthcare DICOM 存储**仅复制 `idc-open-data` S3 存储桶的数据**。不包含以下两个额外存储桶的数据：

- `idc-open-data-cr`
- `idc-open-data-two`

这些缺失的存储桶通常每个包含数千个序列，约占 IDC 总数据的 4%。具体数量因 IDC 版本而异。

有关存储桶组织、文件结构和直接访问方法的详细信息，请参阅 `cloud_storage_guide.md`。

### 更新时间

- **IDC 公共代理**：新 IDC 版本发布时立即更新
- **Google Healthcare**：新 IDC 版本发布后 1-2 周更新

在发布间隔期间，两个端点均保持最新状态。1-2 周的延迟仅在新 IDC 版本发布后的过渡期发生。

**IDC 文档警告：** *"Google 托管的 DICOM 存储可能不包含最新版本的 IDC 数据！"* - 请在新版本发布后的几周内注意检查。

### 选择合适端点

**在以下情况使用 IDC 公共代理：**
- 需要完整数据覆盖（100%）
- 需要在新版本发布后立即获取最新数据
- 不想设置 GCP 认证
- 使用量在每 IP 配额范围内（可通过 support@canceridc.dev 申请增加）
- 需要逐帧访问切片显微镜图像

**在以下情况使用 Google Healthcare API：**
- 约 4% 的数据缺失不影响您的用例
- 需要更高配额以满足重度使用
- 需要更优性能（直接访问，无代理路由）

### 检查数据可用性

选择端点前，请验证您的数据是否可能在缺失的存储桶中：

```python
from idc_index import IDCClient

client = IDCClient()

# 检查您的数据集数据所在的存储桶
results = client.sql_query("""
    SELECT series_aws_url, COUNT(*) as series_count
    FROM index
    WHERE collection_id = 'your_collection_id'
    GROUP BY series_aws_url
""")

print(results)

# 查找包含 'idc-open-data-cr' 或 'idc-open-data-two' 的 URL
# 如果存在，则该数据在 Google Healthcare 端点不可用
```

## 实现细节

IDC DICOMweb 通过 Google Cloud Healthcare API 的 DICOM 存储提供。实现遵循 DICOM PS3.18 Web Services，具体特性详见 [Google Healthcare DICOM 一致性声明](https://docs.cloud.google.com/healthcare-api/docs/dicom)。

### 支持的操作

| 服务 | 描述 | 是否支持 |
|------|------|----------|
| QIDO-RS | 搜索 DICOM 对象 | 是 |
| WADO-RS | 检索 DICOM 对象和元数据 | 是 |
| STOW-RS | 存储 DICOM 对象 | 否（IDC 为只读） |

**不支持：** URI 服务、工作列表服务、非患者实例服务、能力事务

### 可搜索的 DICOM 标签（QIDO-RS）

实现支持有限的可搜索标签集：

| 层级 | 可搜索标签 |
|------|------------|
| 研究 | StudyInstanceUID, PatientName, PatientID, AccessionNumber, ReferringPhysicianName, StudyDate |
| 序列 | 所有研究标签 + SeriesInstanceUID, Modality |
| 实例 | 所有序列标签 + SOPInstanceUID |

**重要提示：** 仅支持精确匹配，以下除外：
- StudyDate：支持范围查询
- PatientName：支持模糊匹配

### 查询限制

- 最大结果数：研究/序列搜索为 5,000；实例搜索为 50,000
- 最大偏移量：1,000,000
- 超过约 1 MB 的 DICOM 序列标签不会在元数据中返回（改为提供 BulkDataURI）

## 代码示例

所有示例均使用公共代理端点。如需通过认证访问 Google Healthcare，请参阅[认证部分](#google-healthcare-api-认证)。

### 使用 idc-index 查找 UID

使用 `idc-index` 发现数据，然后使用 DICOMweb 访问元数据：

```python
from idc_index import IDCClient

client = IDCClient()

# 查找感兴趣的研究
results = client.sql_query("""
    SELECT StudyInstanceUID, SeriesInstanceUID, PatientID, Modality
    FROM index
    WHERE collection_id = 'tcga_luad' AND Modality = 'CT'
    LIMIT 5
""")

# 将 UID 用于 DICOMweb
study_uid = results.iloc[0]['StudyInstanceUID']
series_uid = results.iloc[0]['SeriesInstanceUID']
print(f"研究: {study_uid}")
print(f"序列: {series_uid}")
```

### QIDO-RS：按 UID 搜索

```python
import requests

base_url = "https://proxy.imaging.datacommons.cancer.gov/current/viewer-only-no-downloads-see-tinyurl-dot-com-slash-3j3d9jyp/dicomWeb"

# 搜索特定研究
study_uid = "1.3.6.1.4.1.14519.5.2.1.6450.9002.307623500513044641407722230440"
response = requests.get(
    f"{base_url}/studies",
    params={"StudyInstanceUID": study_uid},
    headers={"Accept": "application/dicom+json"}
)

if response.status_code == 200:
    studies = response.json()
    print(f"找到 {len(studies)} 个研究")
```

### QIDO-RS：列出研究中的序列

```python
import requests

base_url = "https://proxy.imaging.datacommons.cancer.gov/current/viewer-only-no-downloads-see-tinyurl-dot-com-slash-3j3d9jyp/dicomWeb"
study_uid = "1.3.6.1.4.1.14519.5.2.1.6450.9002.307623500513044641407722230440"

response = requests.get(
    f"{base_url}/studies/{study_uid}/series",
    headers={"Accept": "application/dicom+json"}
)

if response.status_code == 200:
    series_list = response.json()
    for series in series_list:
        # DICOM 标签以十六进制代码返回
        series_uid = series.get("0020000E", {}).get("Value", [None])[0]
        modality = series.get("00080060", {}).get("Value", [None])[0]
        description = series.get("0008103E", {}).get("Value", [""])[0]
        print(f"{modality}: {description}")
```

### QIDO-RS：列出序列中的实例

```python
import requests

base_url = "https://proxy.imaging.datacommons.cancer.gov/current/viewer-only-no-downloads-see-tinyurl-dot-com-slash-3j3d9jyp/dicomWeb"
study_uid = "1.3.6.1.4.1.14519.5.2.1.6450.9002.307623500513044641407722230440"
series_uid = "1.3.6.1.4.1.14519.5.2.1.6450.9002.217441095430480124587725641302"

response = requests.get(
    f"{base_url}/studies/{study_uid}/series/{series_uid}/instances",
    params={"limit": 10},
    headers={"Accept": "application/dicom+json"}
)

if response.status_code == 200:
    instances = response.json()
    print(f"找到 {len(instances)} 个实例")
    for inst in instances[:3]:
        sop_uid = inst.get("00080018", {}).get("Value", [None])[0]
        print(f"  SOPInstanceUID: {sop_uid}")
```

### WADO-RS：检索序列元数据

```python
import requests

base_url = "https://proxy.imaging.datacommons.cancer.gov/current/viewer-only-no-downloads-see-tinyurl-dot-com-slash-3j3d9jyp/dicomWeb"
study_uid = "1.3.6.1.4.1.14519.5.2.1.6450.9002.307623500513044641407722230440"
series_uid = "1.3.6.1.4.1.14519.5.2.1.6450.9002.217441095430480124587725641302"

response = requests.get(
    f"{base_url}/studies/{study_uid}/series/{series_uid}/metadata",
    headers={"Accept": "application/dicom+json"}
)

if response.status_code == 200:
    instances = response.json()
    print(f"已检索 {len(instances)} 个实例的元数据")

    # 从第一个实例提取图像尺寸
    if instances:
        inst = instances[0]
        rows = inst.get("00280010", {}).get("Value", [None])[0]
        cols = inst.get("00280011", {}).get("Value", [None])[0]
        print(f"图像尺寸: {rows} x {cols}")
```

### 组合工作流：idc-index 发现 + DICOMweb 元数据

```python
from idc_index import IDCClient
import requests

# 使用 idc-index 高效发现数据
idc = IDCClient()
results = idc.sql_query("""
    SELECT StudyInstanceUID, SeriesInstanceUID, Modality, SeriesDescription
    FROM index
    WHERE collection_id = 'nlst' AND Modality = 'CT'
    LIMIT 1
""")

study_uid = results.iloc[0]['StudyInstanceUID']
series_uid = results.iloc[0]['SeriesInstanceUID']
print(f"找到: {results.iloc[0]['SeriesDescription']}")

# 使用 DICOMweb 流式传输元数据而无需下载文件
base_url = "https://proxy.imaging.datacommons.cancer.gov/current/viewer-only-no-downloads-see-tinyurl-dot-com-slash-3j3d9jyp/dicomWeb"

response = requests.get(
    f"{base_url}/studies/{study_uid}/series/{series_uid}/metadata",
    headers={"Accept": "application/dicom+json"}
)

if response.status_code == 200:
    metadata = response.json()
    print(f"已检索 {len(metadata)} 个实例的元数据，无需下载文件")
```

## 常用 DICOM 标签参考

DICOMweb 以十六进制代码返回标签。常用标签：

| 标签 | 名称 | 描述 |
|------|------|------|
| 00080018 | SOPInstanceUID | 实例唯一标识符 |
| 00080020 | StudyDate | 研究执行日期 |
| 00080060 | Modality | 成像模态（CT、MR、PT 等） |
| 0008103E | SeriesDescription | 序列描述 |
| 00100020 | PatientID | 患者标识符 |
| 0020000D | StudyInstanceUID | 研究唯一标识符 |
| 0020000E | SeriesInstanceUID | 序列唯一标识符 |
| 00280010 | Rows | 图像高度（像素） |
| 00280011 | Columns | 图像宽度（像素） |

## Google Healthcare API 认证

使用更高配额的 Google Healthcare 端点：

```python
from google.auth import default
from google.auth.transport.requests import Request
import requests

# 获取凭据（需 gcloud 认证）
credentials, project = default()
credentials.refresh(Request())

# 构建认证请求
base_url = "https://healthcare.googleapis.com/v1/projects/nci-idc-data/locations/us-central1/datasets/idc/dicomStores/idc-store-v23/dicomWeb"

response = requests.get(
    f"{base_url}/studies",
    params={"limit": 5},
    headers={
        "Authorization": f"Bearer {credentials.token}",
        "Accept": "application/dicom+json"
    }
)
```

**前提条件：**
1. 已安装 Google Cloud SDK（`gcloud`）
2. 已认证：`gcloud auth application-default login`
3. 账户具有公共 Google Cloud 数据集访问权限

## 故障排除

### 问题：搜索查询返回 400 Bad Request
- **原因：** 使用了不支持的搜索参数。实现仅支持特定 DICOM 标签进行过滤。
- **解决方案：** 使用基于 UID 的查询（StudyInstanceUID、SeriesInstanceUID）。如需按 Modality 或其他属性过滤，请先用 `idc-index` 发现 UID，再用特定 UID 查询 DICOMweb。

### 问题：Google Healthcare 端点返回 403 Forbidden
- **原因：** 缺少认证或权限不足
- **解决方案：** 运行 `gcloud auth application-default login` 并确保账户具有访问权限

### 问题：429 Too Many Requests
- **原因：** 超出速率限制
- **解决方案：** 增加请求间隔时间、减小 `limit` 值，或使用认证端点获取更高配额

### 问题：有效 UID 返回 204 No Content
- **原因：** UID 可能来自旧版 IDC 数据，或数据位于 Google Healthcare 未复制的存储桶中
- **解决方案：**
  - 先用 `idc-index` 查询验证 UID 是否存在
  - 检查数据是否在 `idc-open-data-cr` 或 `idc-open-data-two` 存储桶中（这些数据在 Google Healthcare 端点不可用）
  - 切换到 IDC 公共代理以获取 100% 覆盖
  - 新版本发布期间，Google Healthcare 可能滞后 1-2 周

### 问题：大型元数据响应解析缓慢
- **原因：** 包含大量实例的序列返回大型 JSON
- **解决方案：** 在实例查询中使用 `limit` 参数，或通过 SOPInstanceUID 查询特定实例

### 问题：响应缺少预期属性
- **原因：** 超过约 1 MB 的 DICOM 序列会从元数据响应中排除
- **解决方案：** 如需所有属性，请使用 WADO-RS 实例检索获取完整 DICOM 实例
