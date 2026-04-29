# 数据访问与检索

本文档涵盖导航 OMERO 的层次化数据结构及检索对象的方法。

## OMERO 数据层次结构

### 标准层次结构

```
项目 (Project)
  └─ 数据集 (Dataset)
       └─ 图像 (Image)
```

### 筛选层次结构

```
屏幕 (Screen)
  └─ 板 (Plate)
       └─ 孔 (Well)
            └─ 孔样本 (WellSample)
                 └─ 图像 (Image)
```

## 列出对象

### 列出项目

```python
# 列出当前用户的所有项目
for project in conn.listProjects():
    print(f"项目: {project.getName()} (ID: {project.getId()})")
```

### 带筛选条件的项目列表

```python
# 获取当前用户和组
my_exp_id = conn.getUser().getId()
default_group_id = conn.getEventContext().groupId

# 带筛选条件列出项目
for project in conn.getObjects("Project", opts={
    'owner': my_exp_id,                    # 按所有者筛选
    'group': default_group_id,             # 按组筛选
    'order_by': 'lower(obj.name)',         # 按字母顺序排序
    'limit': 10,                           # 结果数量限制
    'offset': 0                            # 分页偏移量
}):
    print(f"项目: {project.getName()}")
```

### 列出数据集

```python
# 列出所有数据集
for dataset in conn.getObjects("Dataset"):
    print(f"数据集: {dataset.getName()} (ID: {dataset.getId()})")

# 列出孤立数据集（不属于任何项目）
for dataset in conn.getObjects("Dataset", opts={'orphaned': True}):
    print(f"孤立数据集: {dataset.getName()}")
```

### 列出图像

```python
# 列出所有图像
for image in conn.getObjects("Image"):
    print(f"图像: {image.getName()} (ID: {image.getId()})")

# 列出特定数据集中的图像
dataset_id = 123
for image in conn.getObjects("Image", opts={'dataset': dataset_id}):
    print(f"图像: {image.getName()}")

# 列出孤立图像
for image in conn.getObjects("Image", opts={'orphaned': True}):
    print(f"孤立图像: {image.getName()}")
```

## 通过 ID 检索对象

### 获取单个对象

```python
# 通过 ID 获取项目
project = conn.getObject("Project", project_id)
if project:
    print(f"项目: {project.getName()}")
else:
    print("未找到项目")

# 通过 ID 获取数据集
dataset = conn.getObject("Dataset", dataset_id)

# 通过 ID 获取图像
image = conn.getObject("Image", image_id)
```

### 通过 ID 批量获取对象

```python
# 批量获取多个项目
project_ids = [1, 2, 3, 4, 5]
projects = conn.getObjects("Project", project_ids)

for project in projects:
    print(f"项目: {project.getName()}")
```

### 支持的对象类型

`getObject()` 和 `getObjects()` 方法支持：
- `"Project"`
- `"Dataset"`
- `"Image"`
- `"Screen"`
- `"Plate"`
- `"Well"`
- `"Roi"`
- `"Annotation"` (及特定类型：`"TagAnnotation"`, `"FileAnnotation"` 等)
- `"Experimenter"`
- `"ExperimenterGroup"`
- `"Fileset"`

## 按属性查询

### 按名称查询对象

```python
# 查找特定名称的图像
images = conn.getObjects("Image", attributes={"name": "sample_001.tif"})

for image in images:
    print(f"找到图像: {image.getName()} (ID: {image.getId()})")

# 查找特定名称的数据集
datasets = conn.getObjects("Dataset", attributes={"name": "Control Group"})
```

### 按值查询注释

```python
# 查找特定文本值的标签
tags = conn.getObjects("TagAnnotation",
                      attributes={"textValue": "experiment_tag"})

for tag in tags:
    print(f"标签: {tag.getValue()}")

# 查找映射注释
map_anns = conn.getObjects("MapAnnotation",
                          attributes={"ns": "custom.namespace"})
```

## 导航层次结构

### 向下导航（父级到子级）

```python
# 项目 → 数据集 → 图像
project = conn.getObject("Project", project_id)

for dataset in project.listChildren():
    print(f"数据集: {dataset.getName()}")

    for image in dataset.listChildren():
        print(f"  图像: {image.getName()}")
```

### 向上导航（子级到父级）

```python
# 图像 → 数据集 → 项目
image = conn.getObject("Image", image_id)

# 获取父数据集
dataset = image.getParent()
if dataset:
    print(f"数据集: {dataset.getName()}")

    # 获取父项目
    project = dataset.getParent()
    if project:
        print(f"项目: {project.getName()}")
```

### 完整层次遍历

```python
# 遍历完整项目层次结构
for project in conn.getObjects("Project", opts={'order_by': 'lower(obj.name)'}):
    print(f"项目: {project.getName()} (ID: {project.getId()})")

    for dataset in project.listChildren():
        image_count = dataset.countChildren()
        print(f"  数据集: {dataset.getName()} ({image_count} 张图像)")

        for image in dataset.listChildren():
            print(f"    图像: {image.getName()}")
            print(f"      尺寸: {image.getSizeX()} x {image.getSizeY()}")
            print(f"      通道数: {image.getSizeC()}")
```

## 筛选数据访问

### 列出屏幕和板

```python
# 列出所有屏幕
for screen in conn.getObjects("Screen"):
    print(f"屏幕: {screen.getName()} (ID: {screen.getId()})")

    # 列出屏幕中的板
    for plate in screen.listChildren():
        print(f"  板: {plate.getName()} (ID: {plate.getId()})")
```

### 访问板孔位

```python
# 获取板对象
plate = conn.getObject("Plate", plate_id)

# 板元数据
print(f"板: {plate.getName()}")
print(f"网格尺寸: {plate.getGridSize()}")  # 例如 96 孔板为 (8, 12)
print(f"视野数量: {plate.getNumberOfFields()}")

# 遍历孔位
for well in plate.listChildren():
    print(f"位于行 {well.row}, 列 {well.column} 的孔")

    # 统计孔内图像（视野）
    field_count = well.countWellSample()
    print(f"  视野数量: {field_count}")

    # 访问孔内图像
    for index in range(field_count):
        image = well.getImage(index)
        print(f"    视野 {index}: {image.getName()}")
```

### 直接孔位访问

```python
# 通过行列获取特定孔
well = plate.getWell(row=0, column=0)  # 左上角孔

# 从孔获取图像
if well.countWellSample() > 0:
    image = well.getImage(0)  # 第一个视野
    print(f"图像: {image.getName()}")
```

### 孔样本访问

```python
# 直接访问孔样本
for well in plate.listChildren():
    for ws in well.listChildren():  # ws = 孔样本
        image = ws.getImage()
        print(f"孔样本 {ws.getId()}: {image.getName()}")
```

## 图像属性

### 基本维度

```python
image = conn.getObject("Image", image_id)

# 像素维度
print(f"X: {image.getSizeX()}")
print(f"Y: {image.getSizeY()}")
print(f"Z: {image.getSizeZ()} (Z 切片)")
print(f"C: {image.getSizeC()} (通道)")
print(f"T: {image.getSizeT()} (时间点)")

# 图像类型
print(f"类型: {image.getPixelsType()}")  # 例如 'uint16', 'uint8'
```

### 物理维度

```python
# 获取带单位的像素尺寸 (OMERO 5.1.0+)
size_x_obj = image.getPixelSizeX(units=True)
size_y_obj = image.getPixelSizeY(units=True)
size_z_obj = image.getPixelSizeZ(units=True)

print(f"X 像素尺寸: {size_x_obj.getValue()} {size_x_obj.getSymbol()}")
print(f"Y 像素尺寸: {size_y_obj.getValue()} {size_y_obj.getSymbol()}")
print(f"Z 像素尺寸: {size_z_obj.getValue()} {size_z_obj.getSymbol()}")

# 获取浮点值 (微米)
size_x = image.getPixelSizeX()  # 返回 µm 单位的浮点数
size_y = image.getPixelSizeY()
size_z = image.getPixelSizeZ()
```

### 通道信息

```python
# 遍历通道
for channel in image.getChannels():
    print(f"通道 {channel.getLabel()}:")
    print(f"  颜色: {channel.getColor().getRGB()}")
    print(f"  查找表: {channel.getLut()}")
    print(f"  波长: {channel.getEmissionWave()}")
```

### 图像元数据

```python
# 采集日期
acquired = image.getAcquisitionDate()
if acquired:
    print(f"采集时间: {acquired}")

# 描述
description = image.getDescription()
if description:
    print(f"描述: {description}")

# 所有者和组
details = image.getDetails()
print(f"所有者: {details.getOwner().getFullName()}")
print(f"用户名: {details.getOwner().getOmeName()}")
print(f"组: {details.getGroup().getName()}")
print(f"创建时间: {details.getCreationEvent().getTime()}")
```

## 对象所有权与权限

### 获取所有者信息

```python
# 获取对象所有者
obj = conn.getObject("Dataset", dataset_id)
owner = obj.getDetails().getOwner()

print(f"所有者 ID: {owner.getId()}")
print(f"用户名: {owner.getOmeName()}")
print(f"全名: {owner.getFullName()}")
print(f"邮箱: {owner.getEmail()}")
```

### 获取组信息

```python
# 获取对象所属组
obj = conn.getObject("Image", image_id)
group = obj.getDetails().getGroup()

print(f"组: {group.getName()} (ID: {group.getId()})")
```

### 按所有者筛选

```python
# 获取特定用户的对象
user_id = 5
datasets = conn.getObjects("Dataset", opts={'owner': user_id})

for dataset in datasets:
    print(f"数据集: {dataset.getName()}")
```

## 高级查询

### 分页处理

```python
# 分页处理大型结果集
page_size = 50
offset = 0

while True:
    images = list(conn.getObjects("Image", opts={
        'limit': page_size,
        'offset': offset,
        'order_by': 'obj.id'
    }))

    if not images:
        break

    for image in images:
        print(f"图像: {image.getName()}")

    offset += page_size
```

### 结果排序

```python
# 按名称排序 (不区分大小写)
projects = conn.getObjects("Project", opts={
    'order_by': 'lower(obj.name)'
})

# 按 ID 排序 (升序)
datasets = conn.getObjects("Dataset", opts={
    'order_by': 'obj.id'
})

# 按名称排序 (降序)
images = conn.getObjects("Image", opts={
    'order_by': 'lower(obj.name) desc'
})
```

### 组合筛选条件

```python
# 包含多个筛选条件的复杂查询
my_exp_id = conn.getUser().getId()
default_group_id = conn.getEventContext().groupId

images = conn.getObjects("Image", opts={
    'owner': my_exp_id,
    'group': default_group_id,
    'dataset': dataset_id,
    'order_by': 'lower(obj.name)',
    'limit': 100,
    'offset': 0
})
```

## 对象计数

### 统计子对象数量

```python
# 统计数据集中的图像数量
dataset = conn.getObject("Dataset", dataset_id)
image_count = dataset.countChildren()
print(f"数据集包含 {image_count} 张图像")

# 统计项目中的数据集数量
project = conn.getObject("Project", project_id)
dataset_count = project.countChildren()
print(f"项目包含 {dataset_count} 个数据集")
```

### 统计注释数量

```python
# 统计对象的注释数量
image = conn.getObject("Image", image_id)
annotation_count = image.countAnnotations()
print(f"图像包含 {annotation_count} 条注释")
```

## 孤立对象

### 查找孤立数据集

```python
# 未链接到任何项目的数据集
orphaned_datasets = conn.getObjects("Dataset", opts={'orphaned': True})

print("孤立数据集:")
for dataset in orphaned_datasets:
    print(f"  {dataset.getName()} (ID: {dataset.getId()})")
    print(f"    所有者: {dataset.getDetails().getOwner().getOmeName()}")
    print(f"    图像数: {dataset.countChildren()}")
```

### 查找孤立图像

```python
# 不属于任何数据集的图像
orphaned_images = conn.getObjects("Image", opts={'orphaned': True})

print("孤立图像:")
for image in orphaned_images:
    print(f"  {image.getName()} (ID: {image.getId()})")
```

### 查找孤立板

```python
# 不属于任何屏幕的板
orphaned_plates = conn.getObjects("Plate", opts={'orphaned': True})

for plate in orphaned_plates:
    print(f"孤立板: {plate.getName()}")
```

## 完整示例

```python
from omero.gateway import BlitzGateway

# 连接参数
HOST = 'omero.example.com'
PORT = 4064
USERNAME = 'user'
PASSWORD = 'pass'

# 连接并查询数据
with BlitzGateway(USERNAME, PASSWORD, host=HOST, port=PORT) as conn:
    # 获取用户上下文
    user = conn.getUser()
    group = conn.getGroupFromContext()

    print(f"已连接用户: {user.getName()}，所属组: {group.getName()}")
    print()

    # 列出项目及其数据集和图像
    for project in conn.getObjects("Project", opts={'limit': 5}):
        print(f"项目: {project.getName()} (ID: {project.getId()})")

        for dataset in project.listChildren():
            image_count = dataset.countChildren()
            print(f"  数据集: {dataset.getName()} ({image_count} 张图像)")

            # 显示前 3 张图像
            for idx, image in enumerate(dataset.listChildren()):
                if idx >= 3:
                    print(f"    ... 及另外 {image_count - 3} 张")
                    break
                print(f"    图像: {image.getName()}")
                print(f"      尺寸: {image.getSizeX()}x{image.getSizeY()}")
                print(f"      通道数: {image.getSizeC()}，Z 切片: {image.getSizeZ()}")

        print()
```

## 最佳实践

1. **使用上下文管理器**：始终使用 `with` 语句确保自动清理连接
2. **限制结果集**：处理大型数据集时使用 `limit` 和 `offset`
3. **尽早筛选**：应用筛选条件减少数据传输量
4. **空值检查**：使用 `getObject()` 后始终检查返回值是否为 None
5. **高效遍历**：优先使用 `listChildren()` 而非单独查询
6. **预先计数**：使用 `countChildren()` 判断是否需要加载数据
7. **组上下文**：跨组查询前设置正确的组上下文
8. **分页机制**：为大型结果集实现分页处理
9. **对象复用**：缓存频繁访问的对象以减少查询次数
10. **错误处理**：使用 try-except 块包裹查询以增强健壮性
