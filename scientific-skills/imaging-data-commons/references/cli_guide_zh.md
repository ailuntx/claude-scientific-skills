# idc-index 命令行界面指南

`idc-index` 软件包提供命令行工具，无需编写 Python 代码即可从 NCI 影像数据共享库下载 DICOM 数据。

## 安装

```bash
pip install --upgrade idc-index
```

安装完成后，终端中即可使用 `idc` 命令。

## 可用命令

| 命令 | 用途 |
|---------|---------|
| `idc download` | 通用下载功能，自动检测输入类型 |
| `idc download-from-manifest` | 通过清单文件下载，支持验证和进度跟踪 |
| `idc download-from-selection` | 基于多条件筛选的下载 |

---

## idc download

通用下载命令，智能解析输入内容。自动识别输入是清单文件路径还是标识符列表（collection_id, PatientID, StudyInstanceUID, SeriesInstanceUID, crdc_series_uuid）。

### 用法

```bash
# 下载整个数据集
idc download rider_pilot --download-dir ./data

# 通过 UID 下载特定序列
idc download "1.3.6.1.4.1.9328.50.1.69736" --download-dir ./data

# 下载多个项目（逗号分隔）
idc download "tcga_luad,tcga_lusc" --download-dir ./data

# 通过清单文件下载（根据文件扩展名自动识别）
idc download manifest.txt --download-dir ./data
```

### 选项

| 选项 | 描述 |
|--------|-------------|
| `--download-dir` | 目标目录（默认：当前目录） |
| `--dir-template` | 目录层级模板（默认：`%collection_id/%PatientID/%StudyInstanceUID/%Modality_%SeriesInstanceUID`） |
| `--log-level` | 日志级别：debug, info, warning, error, critical |

### 目录模板变量

在 `--dir-template` 中使用以下变量组织下载内容：

- `%collection_id` - 数据集标识符
- `%PatientID` - 患者标识符
- `%StudyInstanceUID` - 研究实例 UID
- `%SeriesInstanceUID` - 序列实例 UID
- `%Modality` - 影像模态（CT, MR, PT 等）

**示例：**

```bash
# 平铺结构（所有文件存于同一目录）
idc download rider_pilot --download-dir ./data --dir-template ""

# 简化层级
idc download rider_pilot --download-dir ./data --dir-template "%collection_id/%PatientID/%Modality"
```

---

## idc download-from-manifest

专为清单文件下载设计，内置验证、进度跟踪和断点续传功能。

### 用法

```bash
# 基础清单下载
idc download-from-manifest --manifest-file cohort.txt --download-dir ./data

# 带进度条和验证
idc download-from-manifest --manifest-file cohort.txt --download-dir ./data --show-progress-bar

# 使用 s5cmd sync 恢复中断的下载
idc download-from-manifest --manifest-file cohort.txt --download-dir ./data --use-s5cmd-sync
```

### 选项

| 选项 | 描述 |
|--------|-------------|
| `--manifest-file` | **必需。** 包含 S3 URL 的清单文件路径 |
| `--download-dir` | **必需。** 目标目录 |
| `--validate-manifest` | 下载前验证清单（默认启用） |
| `--show-progress-bar` | 显示下载进度条 |
| `--use-s5cmd-sync` | 启用断点续传 - 跳过已下载文件 |
| `--quiet` | 禁止子进程输出 |
| `--dir-template` | 目录层级模板 |
| `--log-level` | 日志详细级别 |

### 清单文件格式

清单文件每行包含一个 S3 URL：

```
s3://idc-open-data/cb09464a-c5cc-4428-9339-d7fa87cfe837/*
s3://idc-open-data/88f3990d-bdef-49cd-9b2b-4787767240f2/*
```

**获取清单文件方法：**

1. **IDC 门户**：导出队列选择为清单
2. **Python 查询**：通过 SQL 结果生成

```python
from idc_index import IDCClient

client = IDCClient()
results = client.sql_query("""
    SELECT series_aws_url
    FROM index
    WHERE collection_id = 'rider_pilot' AND Modality = 'CT'
""")

with open('ct_manifest.txt', 'w') as f:
    for url in results['series_aws_url']:
        f.write(url + '\n')
```

---

## idc download-from-selection

使用筛选条件下载数据。筛选器按顺序应用。

### 用法

```bash
# 按数据集下载
idc download-from-selection --collection-id rider_pilot --download-dir ./data

# 下载特定序列
idc download-from-selection --series-instance-uid "1.3.6.1.4.1.9328.50.1.69736" --download-dir ./data

# 多重筛选
idc download-from-selection --collection-id nlst --patient-id "100004" --download-dir ./data

# 试运行 - 查看待下载内容而不实际下载
idc download-from-selection --collection-id tcga_luad --dry-run --download-dir ./data
```

### 选项

| 选项 | 描述 |
|--------|-------------|
| `--download-dir` | **必需。** 目标目录 |
| `--collection-id` | 按数据集标识符筛选 |
| `--patient-id` | 按患者标识符筛选 |
| `--study-instance-uid` | 按研究实例 UID 筛选 |
| `--series-instance-uid` | 按序列实例 UID 筛选 |
| `--crdc-series-uuid` | 按 CRDC UUID 筛选 |
| `--dry-run` | 计算队列大小但不下载 |
| `--show-progress-bar` | 显示下载进度条 |
| `--use-s5cmd-sync` | 启用断点续传 |
| `--dir-template` | 目录层级模板 |

### 试运行估算大小

使用 `--dry-run` 在下载前预估数据量：

```bash
idc download-from-selection --collection-id nlst --dry-run --download-dir ./data
```

将显示：
- 符合筛选条件的序列数量
- 总下载大小
- 不会实际下载文件

---

## 常用工作流

### 1. 下载小型测试数据集

```bash
# rider_pilot 约 1GB - 适合测试
idc download rider_pilot --download-dir ./test_data
```

### 2. 带进度条和断点续传的大数据集下载

```bash
# 大文件下载使用 s5cmd sync - 中断后可恢复
idc download-from-selection \
    --collection-id nlst \
    --download-dir ./nlst_data \
    --show-progress-bar \
    --use-s5cmd-sync
```

### 3. 下载前预估大小

```bash
# 先检查大小
idc download-from-selection --collection-id tcga_luad --dry-run --download-dir ./data

# 若大小可接受再下载
idc download-from-selection --collection-id tcga_luad --download-dir ./data
```

### 4. 通过 Python + CLI 下载特定模态数据

```python
# 首先在 Python 中查询序列 UID
from idc_index import IDCClient

client = IDCClient()
results = client.sql_query("""
    SELECT SeriesInstanceUID
    FROM index
    WHERE collection_id = 'nlst'
      AND Modality = 'CT'
      AND BodyPartExamined = 'CHEST'
    LIMIT 50
""")

# 保存为清单
results['SeriesInstanceUID'].to_csv('my_series.csv', index=False, header=False)
```

```bash
# 通过 CLI 下载
idc download my_series.csv --download-dir ./lung_ct
```

---

## 内置安全特性

CLI 包含多项安全特性：

- **磁盘空间检查**：下载前验证空间充足
- **清单验证**：默认验证清单文件格式
- **进度跟踪**：大文件下载可选进度条
- **断点续传**：使用 `--use-s5cmd-sync` 恢复中断下载

---

## 故障排除

### 下载中断

使用 `--use-s5cmd-sync` 恢复：

```bash
idc download-from-manifest --manifest-file cohort.txt --download-dir ./data --use-s5cmd-sync
```

### 连接超时

网络不稳定时，通过 Python 生成多个清单分批下载。

---

## 相关资源

- [idc-index 文档](https://idc-index.readthedocs.io/)
- [IDC 门户](https://portal.imaging.datacommons.cancer.gov/) - 交互式队列构建
- [IDC 教程](https://github.com/ImagingDataCommons/IDC-Tutorials)
