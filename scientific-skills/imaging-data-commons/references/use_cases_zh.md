# IDC 常见使用场景

**测试版本：** idc-index 0.11.9 (IDC 数据版本 v23)

本指南提供了常见 IDC 使用场景的完整端到端工作流示例。每个用例都展示了从查询到下载的最佳实践完整流程。

## 何时使用本指南

当您需要以下内容时请查阅本指南：
- 创建训练数据集的完整端到端工作流示例
- 多步骤数据选择和下载工作流模式
- 商业用途的许可证感知数据处理示例
- 下载前数据预览的可视化工作流

关于核心 API 模式（查询、下载、可视化、引用），请参阅主 SKILL.md 中的"核心功能"部分。

## 先决条件

```bash
pip install --upgrade idc-index
```

## 用例 1：查找并下载用于深度学习的肺部 CT 扫描

**目标：** 从 NLST 集合构建肺部 CT 扫描训练数据集

**步骤：**
```python
from idc_index import IDCClient

client = IDCClient()

# 1. 使用特定条件查询肺部 CT 扫描
query = """
SELECT
  PatientID,
  SeriesInstanceUID,
  SeriesDescription
FROM index
WHERE collection_id = 'nlst'
  AND Modality = 'CT'
  AND BodyPartExamined = 'CHEST'
  AND license_short_name = 'CC BY 4.0'
ORDER BY PatientID
LIMIT 100
"""

results = client.sql_query(query)
print(f"找到 {len(results)} 个序列，来自 {results['PatientID'].nunique()} 名患者")

# 2. 按患者组织下载数据
client.download_from_selection(
    seriesInstanceUID=list(results['SeriesInstanceUID'].values),
    downloadDir="./training_data",
    dirTemplate="%collection_id/%PatientID/%SeriesInstanceUID"
)

# 3. 保存清单以确保可复现性
results.to_csv('training_manifest.csv', index=False)
```

## 用例 2：按制造商查询脑部 MRI 进行质量研究

**目标：** 比较不同 MRI 扫描仪制造商的图像质量

**步骤：**
```python
from idc_index import IDCClient
import pandas as pd

client = IDCClient()

# 按制造商分组查询脑部 MRI
query = """
SELECT
  Manufacturer,
  ManufacturerModelName,
  COUNT(DISTINCT SeriesInstanceUID) as num_series,
  COUNT(DISTINCT PatientID) as num_patients
FROM index
WHERE Modality = 'MR'
  AND BodyPartExamined LIKE '%BRAIN%'
GROUP BY Manufacturer, ManufacturerModelName
HAVING num_series >= 10
ORDER BY num_series DESC
"""

manufacturers = client.sql_query(query)
print(manufacturers)

# 从每个制造商下载样本进行比较
for _, row in manufacturers.head(3).iterrows():
    mfr = row['Manufacturer']
    model = row['ManufacturerModelName']

    query = f"""
    SELECT SeriesInstanceUID
    FROM index
    WHERE Manufacturer = '{mfr}'
      AND ManufacturerModelName = '{model}'
      AND Modality = 'MR'
      AND BodyPartExamined LIKE '%BRAIN%'
    LIMIT 5
    """

    series = client.sql_query(query)
    client.download_from_selection(
        seriesInstanceUID=list(series['SeriesInstanceUID'].values),
        downloadDir=f"./quality_study/{mfr.replace(' ', '_')}"
    )
```

## 用例 3：无需下载即可可视化序列

**目标：** 在决定下载前预览影像数据

```python
from idc_index import IDCClient
import webbrowser

client = IDCClient()

series_list = client.sql_query("""
    SELECT SeriesInstanceUID, PatientID, SeriesDescription
    FROM index
    WHERE collection_id = 'acrin_nsclc_fdg_pet' AND Modality = 'PT'
    LIMIT 10
""")

# 在浏览器中预览每个序列
for _, row in series_list.iterrows():
    viewer_url = client.get_viewer_URL(seriesInstanceUID=row['SeriesInstanceUID'])
    print(f"患者 {row['PatientID']}: {row['SeriesDescription']}")
    print(f"  查看地址: {viewer_url}")
    # webbrowser.open(viewer_url)  # 取消注释可自动打开
```

更多可视化选项，请参阅 [IDC 门户入门指南](https://learn.canceridc.dev/portal/getting-started)或用于 3D Slicer 集成的 [SlicerIDCBrowser](https://github.com/ImagingDataCommons/SlicerIDCBrowser)。

## 用例 4：面向商业用途的许可证感知批量下载

**目标：** 仅下载适用于商业应用的 CC-BY 许可数据

**步骤：**
```python
from idc_index import IDCClient

client = IDCClient()

# 仅查询 CC BY 许可数据（允许带归属的商业使用）
query = """
SELECT
  SeriesInstanceUID,
  collection_id,
  PatientID,
  Modality
FROM index
WHERE license_short_name LIKE 'CC BY%'
  AND license_short_name NOT LIKE '%NC%'
  AND Modality IN ('CT', 'MR')
  AND BodyPartExamined IN ('CHEST', 'BRAIN', 'ABDOMEN')
LIMIT 200
"""

cc_by_data = client.sql_query(query)

print(f"找到 {len(cc_by_data)} 个 CC BY 许可序列")
print(f"集合: {cc_by_data['collection_id'].unique()}")

# 带许可证验证的下载
client.download_from_selection(
    seriesInstanceUID=list(cc_by_data['SeriesInstanceUID'].values),
    downloadDir="./commercial_dataset",
    dirTemplate="%collection_id/%Modality/%PatientID/%SeriesInstanceUID"
)

# 保存许可证信息
cc_by_data.to_csv('commercial_dataset_manifest_CC-BY_ONLY.csv', index=False)
```

## 资源

- 主 SKILL.md 获取核心 API 模式（查询、下载、可视化）
- `references/clinical_data_guide.md` 获取临床数据集成工作流
- `references/sql_patterns.md` 获取更多 SQL 查询模式
- `references/index_tables_guide.md` 获取复杂连接模式
