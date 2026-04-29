# IDC 临床数据指南

**测试版本：** idc-index 0.11.7（IDC 数据版本 v23）

临床数据（人口统计学、诊断、治疗、实验室检查、分期）伴随许多 IDC 影像合集。本指南介绍如何使用 `idc-index` 发现、访问临床数据并将其与影像数据集成。

## 何时使用本指南

当您需要以下内容时，请使用本指南：
- 查找某个合集有哪些临床元数据可用
- 根据临床标准（如癌症分期、治疗史）筛选患者
- 将临床属性与影像数据连接以选择队列
- 理解并解码临床表中的编码值

有关基本临床数据访问，请参阅主 SKILL.md 中的“临床数据访问”部分。本指南提供了详细的工作流和高级模式。

## 前提条件

```bash
pip install --upgrade idc-index
```

无需 BigQuery 凭证——临床数据已随 `idc-index` 一同打包。

## 了解 IDC 中的临床数据

### 什么是临床数据？

临床数据是指伴随医学影像的非影像信息：
- 患者人口统计学信息（年龄、性别、种族）
- 临床病史（诊断、手术、治疗）
- 实验室检查和病理结果
- 癌症分期（临床和病理）
- 治疗结果

### 数据组织

IDC 中的临床数据来自数据提交者提供的特定于合集的电子表格。IDC 将这些数据解析为可通过 `idc-index` 查询的表格。

**重要特性：**
- 临床数据**未跨合集统一**（术语和格式各不相同）
- 并非所有合集都有临床数据（请先检查可用性）
- 所有数据均已**匿名化**——`dicom_patient_id` 用于关联影像

### clinical_index 表

`clinical_index` 充当所有可用临床数据的目录/字典：

| 列 | 用途 | 用于 |
|--------|---------|---------|
| `collection_id` | 合集标识符 | 按合集筛选 |
| `table_name` | 完整的 BigQuery 表引用 | BigQuery 查询（如需） |
| `short_table_name` | 短名称 | `get_clinical_table()` 方法 |
| `column` | 表中的列名 | 选择数据列 |
| `column_label` | 人类可读的描述 | 搜索概念 |
| `values` | 该列观察到的属性值 | 解释编码值 |

### `values` 列

`values` 列包含 `column` 字段定义的列中观察到的属性值数组。每个条目包含：
- **option_code**：该列中实际观察到的值
- **option_description**：该值的人类可读描述（来自数据字典，如有；否则为 `None`）

对于 ACRIN 合集，值描述来自提供的数据字典。对于其他合集，则通过检查实际数据值得出。

**注意：** 对于具有 >20 个唯一值的列，`values` 数组为空（`[]`）以保持简洁。

## 核心工作流

### 步骤 1：获取临床索引

```python
from idc_index import IDCClient

client = IDCClient()
client.fetch_index('clinical_index')

# 查看可用列
print(client.clinical_index.columns.tolist())
```

### 步骤 2：发现可用的临床数据

```python
# 列出所有包含临床数据的合集
collections_with_clinical = client.clinical_index["collection_id"].unique().tolist()
print(f"{len(collections_with_clinical)} 个合集包含临床数据")

# 查找特定合集的临床属性
nlst_columns = client.clinical_index[client.clinical_index['collection_id']=='nlst']
nlst_columns[['short_table_name', 'column', 'column_label', 'values']]
```

### 步骤 3：搜索特定属性

```python
# 按关键字在 column_label 中搜索（不区分大小写）
stage_attrs = client.clinical_index[
    client.clinical_index["column_label"].str.contains("[Ss]tage", na=False)
]
stage_attrs[["collection_id", "short_table_name", "column", "column_label"]]
```

### 步骤 4：加载临床表

```python
# 使用 short_table_name 加载表
nlst_canc_df = client.get_clinical_table("nlst_canc")

# 检查结构
print(f"行数：{len(nlst_canc_df)}，列数：{len(nlst_canc_df.columns)}")
nlst_canc_df.head()
```

### 步骤 5：将编码值映射为描述

许多临床属性使用编码值。`clinical_index` 中的 `values` 列包含观察到的值及其描述的数组（如有）。

```python
# 获取 NLST 的 clinical_index 行
nlst_clinical_columns = client.clinical_index[client.clinical_index['collection_id']=='nlst']

# 获取特定列的观察值
# 筛选到 'clinical_stag' 的行并提取 values 数组
clinical_stag_values = nlst_clinical_columns[
    nlst_clinical_columns['column']=='clinical_stag'
]['values'].values[0]

# 查看观察值及其描述
print(clinical_stag_values)
# 输出：array([{'option_code': '.M', 'option_description': '缺失'},
#                {'option_code': '110', 'option_description': 'IA 期'},
#                {'option_code': '120', 'option_description': 'IB 期'}, ...])

# 创建从编码到描述的映射字典
mapping_dict = {item['option_code']: item['option_description'] for item in clinical_stag_values}

# 应用于 DataFrame——先将列转换为字符串以确保匹配一致
nlst_canc_df['clinical_stag_meaning'] = nlst_canc_df['clinical_stag'].astype(str).map(mapping_dict)
```

### 步骤 6：与影像数据连接

`dicom_patient_id` 列将临床数据与影像关联。它与影像索引中的 `PatientID` 列匹配。

```python
# Pandas 合并方法
import pandas as pd

# 获取 NLST CT 影像数据
nlst_imaging = client.index[(client.index['collection_id']=='nlst') & (client.index['Modality']=='CT')]

# 与临床数据连接
merged = pd.merge(
    nlst_imaging[['PatientID', 'StudyInstanceUID']].drop_duplicates(),
    nlst_canc_df[['dicom_patient_id', 'clinical_stag', 'clinical_stag_meaning']],
    left_on='PatientID',
    right_on='dicom_patient_id',
    how='inner'
)
```

```python
# SQL 连接方法
# 通过 get_clinical_table() 加载的临床表不会自动
# 注册到 DuckDB 中。请在连接前手动注册 DataFrame。
nlst_canc_df = client.get_clinical_table("nlst_canc")
client._duckdb_conn.register("nlst_canc", nlst_canc_df)

query = """
SELECT
  index.PatientID,
  index.StudyInstanceUID,
  index.Modality,
  nlst_canc.clinical_stag
FROM index
JOIN nlst_canc ON index.PatientID = nlst_canc.dicom_patient_id
WHERE index.collection_id = 'nlst' AND index.Modality = 'CT'
"""
results = client.sql_query(query)
```

## 常见用例

### 用例 1：按癌症分期选择患者

```python
from idc_index import IDCClient
import pandas as pd

client = IDCClient()
client.fetch_index('clinical_index')

# 加载临床表
nlst_canc = client.get_clinical_table("nlst_canc")

# 选择 IV 期患者（编码 '400'）
stage_iv_patients = nlst_canc[nlst_canc['clinical_stag'] == '400']['dicom_patient_id']

# 获取这些患者的 CT 影像检查
stage_iv_studies = pd.merge(
    client.index[(client.index['collection_id']=='nlst') & (client.index['Modality']=='CT')],
    stage_iv_patients,
    left_on='PatientID',
    right_on='dicom_patient_id',
    how='inner'
)['StudyInstanceUID'].drop_duplicates()

print(f"找到 {len(stage_iv_studies)} 项 IV 期患者的 CT 检查")
```

### 用例 2：查找具有特定临床属性的合集

```python
# 查找包含化疗信息的合集
chemo_collections = client.clinical_index[
    client.clinical_index["column_label"].str.contains("[Cc]hemotherapy", na=False)
]["collection_id"].unique()

print(f"包含化疗数据的合集：{list(chemo_collections)}")
```

### 用例 3：检查临床属性的观察值

```python
# 查找特定属性已观察到的值
chemotherapy_rows = client.clinical_index[
    (client.clinical_index["collection_id"] == "hcc_tace_seg") &
    (client.clinical_index["column"] == "chemotherapy")
]

# 获取观察值数组
values_list = chemotherapy_rows["values"].tolist()
print(values_list)
# 输出：[[{'option_code': 'Cisplastin', 'option_description': None},
#           {'option_code': 'Cisplatin, Mitomycin-C', 'option_description': None}, ...]]
```

### 用例 4：为所选患者生成查看器 URL

```python
import random

# 获取某位 IV 期患者的检查
sample_patient = stage_iv_patients.iloc[0]
studies = client.index[client.index['PatientID'] == sample_patient]['StudyInstanceUID'].unique()

# 生成查看器 URL
if len(studies) > 0:
    viewer_url = client.get_viewer_URL(studyInstanceUID=studies[0])
    print(viewer_url)
```

## 关键概念

### column 与 column_label 的区别

- **column**：用于从表中选择数据（程序化访问）
- **column_label**：用于搜索/理解数据的含义（人类可读）

某些合集（如 `c4kc_kits`）的 column 和 column_label 相同。其他合集（如 ACRIN 合集）的列名可能晦涩难懂，但标签具有描述性。

### option_code 与 option_description 的区别

`values` 数组包含观察到的属性值：
- **option_code**：列中实际观察到的值（用于筛选条件）
- **option_description**：人类可读描述（来自数据字典，如有；否则为 `None`）

### dicom_patient_id

每个临床表都包含 `dicom_patient_id`，它与影像索引中的 `PatientID` 列匹配。这是连接临床数据和影像数据的关键。

## 故障排除

### 问题：找不到临床表

**原因：** 表名错误或该合集不存在该表

**解决方法：** 首先查询 clinical_index 以查找可用表：
```python
client.clinical_index[client.clinical_index['collection_id']=='your_collection']['short_table_name'].unique()
```

### 问题：values 数组为空

**原因：** 当某列具有 >20 个唯一值时，`values` 数组会保持为空

**解决方法：** 直接加载临床表并检查唯一值：
```python
clinical_df = client.get_clinical_table("table_name")
clinical_df['column_name'].unique()
```

### 问题：编码值未在映射中

**原因：** 某些值可能缺失于字典中（例如空字符串、特殊代码如 `.M` 表示缺失）

**解决方法：** 优雅地处理未映射的值：
```python
df['meaning'] = df['code'].astype(str).map(mapping_dict).fillna('未知/缺失')
```

### 问题：连接时找不到匹配的患者

**原因：** 临床数据可能包含没有影像的患者，反之亦然

**解决方法：** 在连接前验证患者重叠情况：
```python
imaging_patients = set(client.index[client.index['collection_id']=='nlst']['PatientID'].unique())
clinical_patients = set(clinical_df['dicom_patient_id'].unique())
overlap = imaging_patients & clinical_patients
print(f"同时具有影像和临床数据的患者数：{len(overlap)}")
```

## 资源

**IDC 文档：**
- [临床数据组织](https://learn.canceridc.dev/data/organization-of-data/clinical) - 临床数据在 IDC 中的组织方式
- [临床数据仪表板](https://datastudio.google.com/u/0/reporting/04cf5976-4ea0-4fee-a749-8bfd162f2e87/page/p_s7mk6eybqc) - 可用临床数据的可视化摘要
- [idc-index clinical_index 文档](https://idc-index.readthedocs.io/en/latest/column_descriptions.html#clinical-index)

**相关指南：**
- `bigquery_guide.md` - 通过 BigQuery 进行高级临床查询
- 主 SKILL.md - IDC 核心工作流

**IDC 教程：**
- [clinical_data_intro.ipynb](https://github.com/ImagingDataCommons/IDC-Tutorials/blob/master/notebooks/advanced_topics/clinical_data_intro.ipynb)
- [exploring_clinical_data.ipynb](https://github.com/ImagingDataCommons/IDC-Tutorials/blob/master/notebooks/getting_started/exploring_clinical_data.ipynb)
- [nlst_clinical_data.ipynb](https://github.com/ImagingDataCommons/IDC-Tutorials/blob/master/notebooks/collections_demos/nlst_clinical_data.ipynb)
