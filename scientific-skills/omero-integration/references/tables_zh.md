# OMERO 表格

本文档介绍如何使用 OMERO.tables 在 OMERO 中创建和管理结构化表格数据。

## OMERO.tables 概述

OMERO.tables 提供了一种存储与 OMERO 对象关联的结构化表格数据的方法。表格以 HDF5 文件形式存储，可高效查询。常见用例包括：

- 存储图像的量化测量结果
- 记录分析结果
- 跟踪实验元数据
- 将测量结果关联到特定图像或 ROI

## 列类型

OMERO.tables 支持多种列类型：

- **LongColumn**：整数值（64位）
- **DoubleColumn**：浮点数值
- **StringColumn**：文本数据（固定最大长度）
- **BoolColumn**：布尔值
- **LongArrayColumn**：整数数组
- **DoubleArrayColumn**：浮点数数组
- **FileColumn**：指向 OMERO 文件的引用
- **ImageColumn**：指向 OMERO 图像的引用
- **RoiColumn**：指向 OMERO ROI 的引用
- **WellColumn**：指向 OMERO 孔板的引用

## 创建表格

### 基础表格创建

```python
from random import random
import omero.grid

# 创建唯一表名
table_name = f"MyAnalysisTable_{random()}"

# 定义列（初始化空数据）
col1 = omero.grid.LongColumn('ImageID', '图像标识符', [])
col2 = omero.grid.DoubleColumn('MeanIntensity', '平均像素强度', [])
col3 = omero.grid.StringColumn('Category', '分类', 64, [])

columns = [col1, col2, col3]

# 获取资源并创建表格
resources = conn.c.sf.sharedResources()
repository_id = resources.repositories().descriptions[0].getId().getValue()
table = resources.newTable(repository_id, table_name)

# 用列定义初始化表格
table.initialize(columns)
```

### 向表格添加数据

```python
# 准备数据
image_ids = [1, 2, 3, 4, 5]
intensities = [123.4, 145.2, 98.7, 156.3, 132.8]
categories = ["Good", "Good", "Poor", "Excellent", "Good"]

# 创建数据列
data_col1 = omero.grid.LongColumn('ImageID', '图像标识符', image_ids)
data_col2 = omero.grid.DoubleColumn('MeanIntensity', '平均像素强度', intensities)
data_col3 = omero.grid.StringColumn('Category', '分类', 64, categories)

data = [data_col1, data_col2, data_col3]

# 向表格添加数据
table.addData(data)

# 获取文件引用
orig_file = table.getOriginalFile()
table.close()  # 操作完成后始终关闭表格
```

### 将表格关联到数据集

```python
# 从表格创建文件注释
orig_file_id = orig_file.id.val
file_ann = omero.model.FileAnnotationI()
file_ann.setFile(omero.model.OriginalFileI(orig_file_id, False))
file_ann = conn.getUpdateService().saveAndReturnObject(file_ann)

# 关联到数据集
link = omero.model.DatasetAnnotationLinkI()
link.setParent(omero.model.DatasetI(dataset_id, False))
link.setChild(omero.model.FileAnnotationI(file_ann.getId().getValue(), False))
conn.getUpdateService().saveAndReturnObject(link)

print(f"已将表格关联到数据集 {dataset_id}")
```

## 列类型详解

### 长整型列（整数）

```python
# 存储整数值的列
image_ids = [101, 102, 103, 104, 105]
col = omero.grid.LongColumn('ImageID', '图像标识符', image_ids)
```

### 双精度列（浮点数）

```python
# 存储浮点数值的列
measurements = [12.34, 56.78, 90.12, 34.56, 78.90]
col = omero.grid.DoubleColumn('Measurement', '以微米为单位的值', measurements)
```

### 字符串列（文本）

```python
# 存储文本的列（需指定最大长度）
labels = ["Control", "Treatment A", "Treatment B", "Control", "Treatment A"]
col = omero.grid.StringColumn('Condition', '实验条件', 64, labels)
```

### 布尔列

```python
# 存储布尔值的列
flags = [True, False, True, True, False]
col = omero.grid.BoolColumn('QualityPass', '通过质量控制', flags)
```

### 图像列（指向图像的引用）

```python
# 关联到 OMERO 图像的列
image_ids = [101, 102, 103, 104, 105]
col = omero.grid.ImageColumn('Image', '源图像', image_ids)
```

### ROI 列（指向 ROI 的引用）

```python
# 关联到 OMERO ROI 的列
roi_ids = [201, 202, 203, 204, 205]
col = omero.grid.RoiColumn('ROI', '关联的 ROI', roi_ids)
```

### 数组列

```python
# 存储双精度数组的列
histogram_data = [
    [10, 20, 30, 40],
    [15, 25, 35, 45],
    [12, 22, 32, 42]
]
col = omero.grid.DoubleArrayColumn('Histogram', '强度直方图', histogram_data)

# 存储长整型数组的列
bin_counts = [[5, 10, 15], [8, 12, 16], [6, 11, 14]]
col = omero.grid.LongArrayColumn('Bins', '直方图分箱', bin_counts)
```

## 读取表格数据

### 打开现有表格

```python
# 按名称获取表格文件
orig_table_file = conn.getObject("OriginalFile",
                                 attributes={'name': table_name})

# 打开表格
resources = conn.c.sf.sharedResources()
table = resources.openTable(orig_table_file._obj)

print(f"已打开表格: {table.getOriginalFile().getName().getValue()}")
print(f"行数: {table.getNumberOfRows()}")
```

### 读取全部数据

```python
# 获取列标题
print("列信息:")
for col in table.getHeaders():
    print(f"  {col.name}: {col.description}")

# 读取全部数据
row_count = table.getNumberOfRows()
data = table.readCoordinates(range(row_count))

# 显示数据
for col in data.columns:
    print(f"\n列名: {col.name}")
    for value in col.values:
        print(f"  {value}")

table.close()
```

### 读取特定行

```python
# 读取第10-20行
start = 10
stop = 20
data = table.read(list(range(table.getHeaders().__len__())), start, stop)

for col in data.columns:
    print(f"列名: {col.name}")
    for value in col.values:
        print(f"  {value}")
```

### 读取特定列

```python
# 仅读取第0列和第2列
column_indices = [0, 2]
start = 0
stop = table.getNumberOfRows()

data = table.read(column_indices, start, stop)

for col in data.columns:
    print(f"列名: {col.name}")
    print(f"值: {col.values}")
```

## 表格查询

### 条件查询

```python
# 查询 MeanIntensity > 100 的行
row_count = table.getNumberOfRows()

query_rows = table.getWhereList(
    "(MeanIntensity > 100)",
    variables={},
    start=0,
    stop=row_count,
    step=0
)

print(f"找到 {len(query_rows)} 条匹配行")

# 读取匹配行
data = table.readCoordinates(query_rows)

for col in data.columns:
    print(f"\n{col.name}:")
    for value in col.values:
        print(f"  {value}")
```

### 复杂查询

```python
# 使用 AND 的多条件查询
query_rows = table.getWhereList(
    "(MeanIntensity > 100) & (MeanIntensity < 150)",
    variables={},
    start=0,
    stop=row_count,
    step=0
)

# 使用 OR 的多条件查询
query_rows = table.getWhereList(
    "(Category == 'Good') | (Category == 'Excellent')",
    variables={},
    start=0,
    stop=row_count,
    step=0
)

# 字符串匹配查询
query_rows = table.getWhereList(
    "(Category == 'Good')",
    variables={},
    start=0,
    stop=row_count,
    step=0
)
```

## 完整示例：图像分析结果

```python
from omero.gateway import BlitzGateway
import omero.grid
import omero.model
import numpy as np

HOST = 'omero.example.com'
PORT = 4064
USERNAME = 'user'
PASSWORD = 'pass'

with BlitzGateway(USERNAME, PASSWORD, host=HOST, port=PORT) as conn:
    # 获取数据集
    dataset = conn.getObject("Dataset", dataset_id)
    print(f"正在分析数据集: {dataset.getName()}")

    # 从图像收集测量值
    image_ids = []
    mean_intensities = []
    max_intensities = []
    cell_counts = []

    for image in dataset.listChildren():
        image_ids.append(image.getId())

        # 获取像素数据
        pixels = image.getPrimaryPixels()
        plane = pixels.getPlane(0, 0, 0)  # Z=0, C=0, T=0

        # 计算统计量
        mean_intensities.append(float(np.mean(plane)))
        max_intensities.append(float(np.max(plane)))

        # 模拟细胞计数（实际应来自分析）
        cell_counts.append(np.random.randint(50, 200))

    # 创建表格
    table_name = f"Analysis_Results_{dataset.getId()}"

    # 定义列
    col1 = omero.grid.ImageColumn('Image', '源图像', [])
    col2 = omero.grid.DoubleColumn('MeanIntensity', '平均像素值', [])
    col3 = omero.grid.DoubleColumn('MaxIntensity', '最大像素值', [])
    col4 = omero.grid.LongColumn('CellCount', '检测到的细胞数', [])

    # 初始化表格
    resources = conn.c.sf.sharedResources()
    repository_id = resources.repositories().descriptions[0].getId().getValue()
    table = resources.newTable(repository_id, table_name)
    table.initialize([col1, col2, col3, col4])

    # 添加数据
    data_col1 = omero.grid.ImageColumn('Image', '源图像', image_ids)
    data_col2 = omero.grid.DoubleColumn('MeanIntensity', '平均像素值',
                                        mean_intensities)
    data_col3 = omero.grid.DoubleColumn('MaxIntensity', '最大像素值',
                                        max_intensities)
    data_col4 = omero.grid.LongColumn('CellCount', '检测到的细胞数',
                                      cell_counts)

    table.addData([data_col1, data_col2, data_col3, data_col4])

    # 获取文件并关闭表格
    orig_file = table.getOriginalFile()
    table.close()

    # 关联到数据集
    orig_file_id = orig_file.id.val
    file_ann = omero.model.FileAnnotationI()
    file_ann.setFile(omero.model.OriginalFileI(orig_file_id, False))
    file_ann = conn.getUpdateService().saveAndReturnObject(file_ann)

    link = omero.model.DatasetAnnotationLinkI()
    link.setParent(omero.model.DatasetI(dataset_id, False))
    link.setChild(omero.model.FileAnnotationI(file_ann.getId().getValue(), False))
    conn.getUpdateService().saveAndReturnObject(link)

    print(f"已创建并关联包含 {len(image_ids)} 行的表格")

    # 查询结果
    table = resources.openTable(orig_file)

    high_cell_count_rows = table.getWhereList(
        "(CellCount > 100)",
        variables={},
        start=0,
        stop=table.getNumberOfRows(),
        step=0
    )

    print(f"细胞数>100的图像: {len(high_cell_count_rows)}")

    # 读取这些行
    data = table.readCoordinates(high_cell_count_rows)
    for i in range(len(high_cell_count_rows)):
        img_id = data.columns[0].values[i]
        count = data.columns[3].values[i]
        print(f"  图像 {img_id}: {count} 个细胞")

    table.close()
```

## 从对象检索表格

### 查找关联到数据集的表格

```python
# 获取数据集
dataset = conn.getObject("Dataset", dataset_id)

# 列出文件注释
for ann in dataset.listAnnotations():
    if isinstance(ann, omero.gateway.FileAnnotationWrapper):
        file_obj = ann.getFile()
        file_name = file_obj.getName()

        # 检查是否为表格（可能有特定命名模式）
        if "Table" in file_name or file_name.endswith(".h5"):
            print(f"找到表格: {file_name} (ID: {file_obj.getId()})")

            # 打开并检查
            resources = conn.c.sf.sharedResources()
            table = resources.openTable(file_obj._obj)

            print(f"  行数: {table.getNumberOfRows()}")
            print(f"  列信息:")
            for col in table.getHeaders():
                print(f"    {col.name}")

            table.close()
```

## 更新表格

### 追加行

```python
# 打开现有表格
resources = conn.c.sf.sharedResources()
table = resources.openTable(orig_file._obj)

# 准备新数据
new_image_ids = [106, 107]
new_intensities = [88.9, 92.3]
new_categories = ["Good", "Excellent"]

# 创建数据列
data_col1 = omero.grid.LongColumn('ImageID', '', new_image_ids)
data_col2 = omero.grid.DoubleColumn('MeanIntensity', '', new_intensities)
data_col3 = omero.grid.StringColumn('Category', '', 64, new_categories)

# 追加数据
table.addData([data_col1, data_col2, data_col3])

print(f"新行数: {table.getNumberOfRows()}")
table.close()
```

## 删除表格

### 删除表格文件

```python
# 获取文件对象
orig_file = conn.getObject("OriginalFile", file_id)

# 删除文件（同时删除表格）
conn.deleteObjects("OriginalFile", [file_id], wait=True)
print(f"已删除表格文件 {file_id}")
```

### 解除对象关联

```python
# 查找注释关联
dataset = conn.getObject("Dataset", dataset_id)

for ann in dataset.listAnnotations():
    if isinstance(ann, omero.gateway.FileAnnotationWrapper):
        if "Table" in ann.getFile().getName():
            # 删除关联（保留表格，移除关联关系）
            conn.deleteObjects("DatasetAnnotationLink",
                             [ann.link.getId()],
                             wait=True)
            print(f"已解除表格与数据集的关联")
```

## 最佳实践

1. **描述性命名**：使用有意义的表格和列名
2. **关闭表格**：使用后始终关闭表格
3. **字符串长度**：为字符串列设置合适的最大长度
4. **关联对象**：将表格关联到相关数据集或项目
5. **使用引用**：使用 ImageColumn、RoiColumn 进行对象引用
6. **高效查询**：使用 getWhereList() 替代读取全部数据
7. **文档记录**：为列添加描述信息
8. **版本控制**：在表名或元数据中包含版本信息
9. **批量操作**：批量添加数据以提高性能
10. **错误处理**：检查 None 返回值并处理异常

## 常用模式

### ROI 测量表

```python
# ROI 测量表结构
columns = [
    omero.grid.ImageColumn('Image', '源图像', []),
    omero.grid.RoiColumn('ROI', '测量的 ROI', []),
    omero.grid.LongColumn('ChannelIndex', '通道编号', []),
    omero.grid.DoubleColumn('Area', 'ROI 面积（像素）', []),
    omero.grid.DoubleColumn('MeanIntensity', '平均强度', []),
    omero.grid.DoubleColumn('IntegratedDensity', '强度总和', []),
    omero.grid.StringColumn('CellType', '细胞分类', 32, [])
]
```

### 时间序列数据表

```python
# 时间序列测量表结构
columns = [
    omero.grid.ImageColumn('Image', '时间序列图像',

omero.grid.LongColumn('FieldIndex', '视野编号', []),
    omero.grid.DoubleColumn('CellCount', '细胞数量', []),
    omero.grid.DoubleColumn('Viability', '存活百分比', []),
    omero.grid.StringColumn('Phenotype', '观察到的表型', 128, []),
    omero.grid.BoolColumn('Hit', '筛选中的命中', [])
]
```
