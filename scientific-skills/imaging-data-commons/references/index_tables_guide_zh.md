# IDC 索引表指南

**测试版本：** idc-index 0.11.14（IDC 数据版本 v23）

本指南涵盖 IDC 索引表的结构和访问模式：程序化模式发现、DataFrame 访问和连接列引用。有关可用表及其用途的概述，请参见主 SKILL.md 中的“索引表”章节。

**完整索引表文档：** https://idc-index.readthedocs.io/en/latest/indices_reference.html

## 何时使用本指南

在以下场景下加载本指南：
- 需要通过编程方式发现表模式和列类型
- 需要以 pandas DataFrame（而非 SQL）的形式访问索引表
- 需要了解关键列及表之间的连接关系

有关 SQL 查询示例（过滤器发现、查找注释、大小估算），请参见 `references/sql_patterns.md`。

## 前置条件

```bash
pip install --upgrade idc-index
```

## 访问索引表

### 通过 SQL（推荐用于过滤/聚合）

```python
from idc_index import IDCClient
client = IDCClient()

# 查询主索引（始终可用）
results = client.sql_query("SELECT * FROM index WHERE Modality = 'CT' LIMIT 10")

# 获取并查询其他索引
client.fetch_index("collections_index")
collections = client.sql_query("SELECT collection_id, CancerTypes, TumorLocations FROM collections_index")

client.fetch_index("analysis_results_index")
analysis = client.sql_query("SELECT * FROM analysis_results_index LIMIT 5")
```

### 以 pandas DataFrame 形式（直接访问）

```python
# 主索引（客户端初始化后始终可用）
df = client.index

# 获取并按需访问索引
client.fetch_index("sm_index")
sm_df = client.sm_index
```

## 发现表模式

`indices_overview` 字典包含所有表的完整模式信息。**编写查询或探索数据结构时请始终参考它。**

**DICOM 属性映射：** 许多列直接从源文件中的 DICOM 属性填充。列描述中会指示某列是否对应 DICOM 属性（例如，“DICOM Modality 属性”或引用 DICOM 标签）。这使得在查询时可以充分利用 DICOM 知识——标准的 DICOM 属性名称如 `PatientID`、`StudyInstanceUID`、`Modality`、`BodyPartExamined` 等按预期工作。

```python
from idc_index import IDCClient
client = IDCClient()

# 列出所有可用索引及其描述
for name, info in client.indices_overview.items():
    print(f"\n{name}:")
    print(f"  已安装: {info['installed']}")
    print(f"  描述: {info['description']}")

# 获取特定索引的完整模式（列、类型、描述）
schema = client.indices_overview["index"]["schema"]
print(f"\n表: {schema['table_description']}")
print("\n列:")
for col in schema['columns']:
    desc = col.get('description', '无描述')
    # 描述指示该列是否来自 DICOM 属性
    print(f"  {col['name']} ({col['type']}): {desc}")

# 查找 DICOM 属性列（检查描述中是否引用“DICOM”）
dicom_cols = [c['name'] for c in schema['columns'] if 'DICOM' in c.get('description', '').upper()]
print(f"\nDICOM 来源的列: {dicom_cols}")
```

**另一种选择：使用 `get_index_schema()` 方法：**
```python
schema = client.get_index_schema("index")
# 返回相同的模式字典：{'table_description': ..., 'columns': [...]}
```

## 关键列参考

主 `index` 表中最常用的列（使用 `indices_overview` 获取完整列表和描述）：

| 列名 | 类型 | DICOM | 描述 |
|--------|------|-------|-------------|
| `collection_id` | STRING | 否 | IDC 集合标识符 |
| `analysis_result_id` | STRING | 否 | 如果适用，指示给定系列属于哪个分析结果集合 |
| `source_DOI` | STRING | 否 | 链接到数据集详细信息的 DOI；用于了解更多内容以及归属（见下方引用） |
| `PatientID` | STRING | 是 | 患者标识符 |
| `StudyInstanceUID` | STRING | 是 | DICOM 研究 UID |
| `SeriesInstanceUID` | STRING | 是 | DICOM 系列 UID — 用于下载/查看 |
| `Modality` | STRING | 是 | 成像模态（CT、MR、PT、SM、SEG、ANN、RTSTRUCT 等） |
| `BodyPartExamined` | STRING | 是 | 解剖区域 |
| `SeriesDescription` | STRING | 是 | 系列描述 |
| `Manufacturer` | STRING | 是 | 设备制造商 |
| `StudyDate` | STRING | 是 | 检查日期 |
| `PatientSex` | STRING | 是 | 患者性别 |
| `PatientAge` | STRING | 是 | 检查时患者年龄 |
| `license_short_name` | STRING | 否 | 许可证类型（CC BY 4.0、CC BY-NC 4.0 等） |
| `series_size_MB` | FLOAT | 否 | 系列大小（兆字节） |
| `instanceCount` | INTEGER | 否 | 系列中 DICOM 实例数量 |
| `SOPClassUID` | STRING | 是 | DICOM SOP 类 UID（标识对象/服务类别，例如 CT 图像存储） |
| `TransferSyntaxUID` | STRING | 是 | DICOM 传输语法 UID（编码/压缩方式） |

**DICOM = 是**：列值从同名 DICOM 属性中提取。有关数值标签映射，请参阅 [DICOM 标准](https://dicom.nema.org/medical/dicom/current/output/chtml/part06/chapter_6.html)。使用标准 DICOM 知识来理解预期值和格式。

## 连接列参考

使用此表确定索引表之间的连接列。在 SQL 中使用表之前，请始终调用 `client.fetch_index("table_name")`。

| 表 A | 表 B | 连接条件 |
|---------|---------|----------------|
| `index` | `collections_index` | `index.collection_id = collections_index.collection_id` |
| `index` | `sm_index` | `index.SeriesInstanceUID = sm_index.SeriesInstanceUID` |
| `index` | `seg_index` | `index.SeriesInstanceUID = seg_index.segmented_SeriesInstanceUID` |
| `index` | `ann_index` | `index.SeriesInstanceUID = ann_index.SeriesInstanceUID` |
| `ann_index` | `ann_group_index` | `ann_index.SeriesInstanceUID = ann_group_index.SeriesInstanceUID` |
| `index` | `clinical_index` | `index.collection_id = clinical_index.collection_id`（然后按患者过滤） |
| `index` | `contrast_index` | `index.SeriesInstanceUID = contrast_index.SeriesInstanceUID` |
| `index` | `volume_geometry_index` | `index.SeriesInstanceUID = volume_geometry_index.SeriesInstanceUID` |
| `index` | `rtstruct_index` | `index.SeriesInstanceUID = rtstruct_index.SeriesInstanceUID` |
| `rtstruct_index` | `index`（源图像） | `rtstruct_index.referenced_SeriesInstanceUID = index.SeriesInstanceUID` |

有关使用这些连接的完整查询示例，请参见 `references/sql_patterns.md`。

## 故障排除

**问题：** 表中未找到列
- **原因：** 列名拼写错误或该表中不存在该列
- **解决方法：** 使用 `client.indices_overview["table_name"]["schema"]["columns"]` 列出可用列

**问题：** DataFrame 访问返回 None
- **原因：** 未获取索引或属性名称不正确
- **解决方法：** 先使用 `client.fetch_index()` 获取，然后通过与索引名称匹配的属性访问

## 资源

- 完整索引表文档：https://idc-index.readthedocs.io/en/latest/indices_reference.html
- `references/sql_patterns.md` 包含使用这些表的查询示例
- `references/clinical_data_guide.md` 用于临床数据工作流
- `references/digital_pathology_guide.md` 用于病理学特定索引
