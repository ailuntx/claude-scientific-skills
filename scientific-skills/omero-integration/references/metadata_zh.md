# 元数据与注释

本文档介绍如何在OMERO中创建和管理注释，包括标签、键值对、文件附件和评论。

## 注释类型

OMERO支持多种注释类型：

- **标签注释（TagAnnotation）**：用于分类的文本标签
- **映射注释（MapAnnotation）**：用于结构化元数据的键值对
- **文件注释（FileAnnotation）**：文件附件（PDF、CSV、分析结果等）
- **评论注释（CommentAnnotation）**：自由文本评论
- **长整型注释（LongAnnotation）**：整数值
- **双精度注释（DoubleAnnotation）**：浮点数值
- **布尔注释（BooleanAnnotation）**：布尔值
- **时间戳注释（TimestampAnnotation）**：日期/时间戳
- **术语注释（TermAnnotation）**：本体术语

## 标签注释

### 创建并关联标签

```python
import omero.gateway

# 创建新标签
tag_ann = omero.gateway.TagAnnotationWrapper(conn)
tag_ann.setValue("实验2024")
tag_ann.setDescription("此标签的可选描述")
tag_ann.save()

# 将标签关联到对象
project = conn.getObject("Project", project_id)
project.linkAnnotation(tag_ann)
```

### 创建带命名空间的标签

```python
# 创建带自定义命名空间的标签
tag_ann = omero.gateway.TagAnnotationWrapper(conn)
tag_ann.setValue("质量控制")
tag_ann.setNs("mylab.qc.tags")
tag_ann.save()

# 关联到图像
image = conn.getObject("Image", image_id)
image.linkAnnotation(tag_ann)
```

### 复用现有标签

```python
# 查找现有标签
tag_id = 123
tag_ann = conn.getObject("TagAnnotation", tag_id)

# 关联到多个图像
for image in conn.getObjects("Image", [img1, img2, img3]):
    image.linkAnnotation(tag_ann)
```

### 创建标签集（含子标签）

```python
# 创建标签集（父标签）
tag_set = omero.gateway.TagAnnotationWrapper(conn)
tag_set.setValue("细胞类型")
tag_set.save()

# 创建子标签
tags = ["HeLa", "U2OS", "HEK293"]
for tag_value in tags:
    tag = omero.gateway.TagAnnotationWrapper(conn)
    tag.setValue(tag_value)
    tag.save()

    # 将子标签关联到父标签
    tag_set.linkAnnotation(tag)
```

## 映射注释（键值对）

### 创建映射注释

```python
import omero.gateway
import omero.constants.metadata

# 准备键值数据
key_value_data = [
    ["药物名称", "Monastrol"],
    ["浓度", "5 mg/ml"],
    ["处理时间", "24小时"],
    ["温度", "37C"]
]

# 创建映射注释
map_ann = omero.gateway.MapAnnotationWrapper(conn)

# 使用标准客户端命名空间
namespace = omero.constants.metadata.NSCLIENTMAPANNOTATION
map_ann.setNs(namespace)

# 设置数据
map_ann.setValue(key_value_data)
map_ann.save()

# 关联到数据集
dataset = conn.getObject("Dataset", dataset_id)
dataset.linkAnnotation(map_ann)
```

### 映射注释的自定义命名空间

```python
# 为组织特定元数据使用自定义命名空间
key_value_data = [
    ["显微镜", "Zeiss LSM 880"],
    ["物镜", "63x Oil"],
    ["激光功率", "10%"]
]

map_ann = omero.gateway.MapAnnotationWrapper(conn)
map_ann.setNs("mylab.microscopy.settings")
map_ann.setValue(key_value_data)
map_ann.save()

image = conn.getObject("Image", image_id)
image.linkAnnotation(map_ann)
```

### 读取映射注释

```python
# 获取映射注释
image = conn.getObject("Image", image_id)

for ann in image.listAnnotations():
    if isinstance(ann, omero.gateway.MapAnnotationWrapper):
        print(f"映射注释 (ID: {ann.getId()}):")
        print(f"命名空间: {ann.getNs()}")

        # 获取键值对
        for key, value in ann.getValue():
            print(f"  {key}: {value}")
```

## 文件注释

### 上传并关联文件

```python
import os

# 准备文件
file_path = "analysis_results.csv"

# 创建文件注释
namespace = "mylab.analysis.results"
file_ann = conn.createFileAnnfromLocalFile(
    file_path,
    mimetype="text/csv",
    ns=namespace,
    desc="细胞分割结果"
)

# 关联到数据集
dataset = conn.getObject("Dataset", dataset_id)
dataset.linkAnnotation(file_ann)
```

### 支持的MIME类型

常用MIME类型：
- 文本：`"text/plain"`, `"text/csv"`, `"text/tab-separated-values"`
- 文档：`"application/pdf"`, `"application/vnd.ms-excel"`
- 图像：`"image/png"`, `"image/jpeg"`
- 数据：`"application/json"`, `"application/xml"`
- 归档：`"application/zip"`, `"application/gzip"`

### 上传多个文件

```python
files = ["figure1.pdf", "figure2.pdf", "table1.csv"]
namespace = "publication.supplementary"

dataset = conn.getObject("Dataset", dataset_id)

for file_path in files:
    file_ann = conn.createFileAnnfromLocalFile(
        file_path,
        mimetype="application/octet-stream",
        ns=namespace,
        desc=f"补充文件: {os.path.basename(file_path)}"
    )
    dataset.linkAnnotation(file_ann)
```

### 下载文件注释

```python
import os

# 获取带文件注释的对象
image = conn.getObject("Image", image_id)

# 下载目录
download_path = "./downloads"
os.makedirs(download_path, exist_ok=True)

# 按命名空间过滤
namespace = "mylab.analysis.results"

for ann in image.listAnnotations(ns=namespace):
    if isinstance(ann, omero.gateway.FileAnnotationWrapper):
        file_name = ann.getFile().getName()
        file_path = os.path.join(download_path, file_name)

        print(f"下载中: {file_name}")

        # 分块下载文件
        with open(file_path, 'wb') as f:
            for chunk in ann.getFileInChunks():
                f.write(chunk)

        print(f"保存至: {file_path}")
```

### 获取文件注释元数据

```python
for ann in dataset.listAnnotations():
    if isinstance(ann, omero.gateway.FileAnnotationWrapper):
        orig_file = ann.getFile()

        print(f"文件注释 ID: {ann.getId()}")
        print(f"  文件名: {orig_file.getName()}")
        print(f"  文件大小: {orig_file.getSize()} 字节")
        print(f"  MIME类型: {orig_file.getMimetype()}")
        print(f"  命名空间: {ann.getNs()}")
        print(f"  描述: {ann.getDescription()}")
```

## 评论注释

### 添加评论

```python
# 创建评论
comment = omero.gateway.CommentAnnotationWrapper(conn)
comment.setValue("此图像显示出色的染色质量")
comment.save()

# 关联到图像
image = conn.getObject("Image", image_id)
image.linkAnnotation(comment)
```

### 添加带命名空间的评论

```python
comment = omero.gateway.CommentAnnotationWrapper(conn)
comment.setValue("已批准发表")
comment.setNs("mylab.publication.status")
comment.save()

dataset = conn.getObject("Dataset", dataset_id)
dataset.linkAnnotation(comment)
```

## 数值注释

### 长整型注释（整数）

```python
# 创建长整型注释
long_ann = omero.gateway.LongAnnotationWrapper(conn)
long_ann.setValue(42)
long_ann.setNs("mylab.cell.count")
long_ann.save()

image = conn.getObject("Image", image_id)
image.linkAnnotation(long_ann)
```

### 双精度注释（浮点数）

```python
# 创建双精度注释
double_ann = omero.gateway.DoubleAnnotationWrapper(conn)
double_ann.setValue(3.14159)
double_ann.setNs("mylab.fluorescence.intensity")
double_ann.save()

image = conn.getObject("Image", image_id)
image.linkAnnotation(double_ann)
```

## 列出注释

### 列出对象的所有注释

```python
import omero.model

# 获取对象
project = conn.getObject("Project", project_id)

# 列出所有注释
for ann in project.listAnnotations():
    print(f"注释 ID: {ann.getId()}")
    print(f"  类型: {ann.OMERO_TYPE}")
    print(f"  添加者: {ann.link.getDetails().getOwner().getOmeName()}")

    # 类型特定处理
    if ann.OMERO_TYPE == omero.model.TagAnnotationI:
        print(f"  标签值: {ann.getTextValue()}")

    elif isinstance(ann, omero.gateway.MapAnnotationWrapper):
        print(f"  映射数据: {ann.getValue()}")

    elif isinstance(ann, omero.gateway.FileAnnotationWrapper):
        print(f"  文件: {ann.getFile().getName()}")

    elif isinstance(ann, omero.gateway.CommentAnnotationWrapper):
        print(f"  评论: {ann.getValue()}")

    print()
```

### 按命名空间过滤注释

```python
# 获取特定命名空间的注释
namespace = "mylab.qc.tags"

for ann in image.listAnnotations(ns=namespace):
    print(f"找到注释: {ann.getId()}")

    if isinstance(ann, omero.gateway.MapAnnotationWrapper):
        for key, value in ann.getValue():
            print(f"  {key}: {value}")
```

### 获取命名空间的第一个注释

```python
# 通过命名空间获取单个注释
namespace = "mylab.analysis.results"
ann = dataset.getAnnotation(namespace)

if ann:
    print(f"找到带命名空间的注释: {ann.getNs()}")
else:
    print("未找到该命名空间的注释")
```

### 跨多个对象查询注释

```python
# 获取关联到图像ID的所有标签注释
image_ids = [1, 2, 3, 4, 5]

for link in conn.getAnnotationLinks('Image', parent_ids=image_ids):
    ann = link.getChild()

    if isinstance(ann._obj, omero.model.TagAnnotationI):
        print(f"图像 {link.getParent().getId()}: 标签 '{ann.getTextValue()}'")
```

## 注释计数

```python
# 统计项目的注释数量
project_id = 123
count = conn.countAnnotations('Project', [project_id])
print(f"项目拥有 {count[project_id]} 条注释")

# 统计多个图像的注释数量
image_ids = [1, 2, 3]
counts = conn.countAnnotations('Image', image_ids)

for image_id, count in counts.items():
    print(f"图像 {image_id}: {count} 条注释")
```

## 注释关联

### 手动创建注释关联

```python
# 获取注释和图像
tag = conn.getObject("TagAnnotation", tag_id)
image = conn.getObject("Image", image_id)

# 创建关联
link = omero.model.ImageAnnotationLinkI()
link.setParent(omero.model.ImageI(image.getId(), False))
link.setChild(omero.model.TagAnnotationI(tag.getId(), False))

# 保存关联
conn.getUpdateService().saveAndReturnObject(link)
```

### 更新注释关联

```python
# 获取现有关联
annotation_ids = [1, 2, 3]
new_tag_id = 5

for link in conn.getAnnotationLinks('Image', ann_ids=annotation_ids):
    print(f"图像 ID: {link.getParent().id}")

    # 更改关联的注释
    link._obj.child = omero.model.TagAnnotationI(new_tag_id, False)
    link.save()
```

## 删除注释

### 删除注释

```python
# 获取图像
image = conn.getObject("Image", image_id)

# 收集待删除的注释ID
to_delete = []
namespace = "mylab.temp.annotations"

for ann in image.listAnnotations(ns=namespace):
    to_delete.append(ann.getId())

# 删除注释
if to_delete:
    conn.deleteObjects('Annotation', to_delete, wait=True)
    print(f"已删除 {len(to_delete)} 条注释")
```

### 解除注释关联（保留注释，移除关联）

```python
# 获取图像
image = conn.getObject("Image", image_id)

# 收集待删除的关联ID
to_delete = []

for ann in image.listAnnotations():
    if isinstance(ann, omero.gateway.TagAnnotationWrapper):
        to_delete.append(ann.link.getId())

# 删除关联（注释保留在数据库中）
if to_delete:
    conn.deleteObjects("ImageAnnotationLink", to_delete, wait=True)
    print(f"已解除 {len(to_delete)} 条注释的关联")
```

### 删除特定类型的注释

```python
import omero.gateway

# 仅删除映射注释
image = conn.getObject("Image", image_id)
to_delete = []

for ann in image.listAnnotations():
    if isinstance(ann, omero.gateway.MapAnnotationWrapper):
        to_delete.append(ann.getId())

conn.deleteObjects('Annotation', to_delete, wait=True)
```

## 注释所有权

### 设置注释所有者（仅管理员）

```python
import omero.model

# 创建带指定所有者的标签
tag_ann = omero.gateway.TagAnnotationWrapper(conn)
tag_ann.setValue("管理员标签")

# 设置所有者（需要管理员权限）
user_id = 5
tag_ann._obj.details.owner = omero.model.ExperimenterI(user_id, False)
tag_ann.save()
```

### 以其他用户身份创建注释（仅管理员）

```python
# 管理员连接
admin_conn = BlitzGateway(admin_user, admin_pass, host=host, port=4064)
admin_conn.connect()

# 获取目标用户
user_id = 10
user = admin_conn.getObject("Experimenter", user_id).getName()

# 以用户身份创建连接
user_conn = admin_conn.suConn(user)

# 以该用户身份创建注释
map_ann = omero.gateway.MapAnnotationWrapper(user_conn)
map_ann.setNs("mylab.metadata")
map_ann.setValue([["key", "value"]])
map_ann.save()

# 关联到项目
project = admin_conn.getObject("Project", project_id)
project.linkAnnotation(map_ann)

# 关闭连接
user_conn.close()
admin_conn.close()
```

## 批量注释操作

### 标记多个图像

```python
# 创建或获取标签
tag = omero.gateway.TagAnnotationWrapper(conn)
tag.setValue("已验证")
tag.save()

# 获取待标记的图像
dataset = conn.getObject("Dataset", dataset_id)

# 标记数据集中的所有图像
for image in dataset.listChildren():
    image.linkAnnotation(tag)
    print(f"已标记图像: {image.getName()}")
```

### 批量添加映射注释

```python
# 为多个图像准备元数据
image_metadata = {
    101: [["质量", "良好"], ["已审核", "是"]],
    102: [["质量", "优秀"], ["已审核", "是"]],
    103: [["质量", "差"], ["已审核", "否"]]
}

# 添加注释
for image_id, kv_data in image_metadata.items():
    image = conn.getObject("Image", image_id)

    if image:
        map_ann = omero.gateway.MapAnnotationWrapper(conn)
        map_ann.setNs("mylab.qc")
        map_ann.setValue(kv_data)
        map_ann.save()

        image.linkAnnotation(map_ann)
        print(f"已注释图像 {image_id}")
```

## 命名空间

### 标准OMERO命名空间

```python
import omero.constants.metadata as omero_ns

# 客户端映射注释命名空间
omero_ns.NSCLIENTMAPANNOTATION
# "openmicroscopy.org/omero/client/mapAnnotation"

# 批量注释命名空间
omero_ns.NSBULKANNOTATIONS
# "openmicroscopy.org/omero/bulk_annotations"
```

### 自定义命名空间

自定义命名空间最佳实践：
- 使用反向域名表示法：`"org.mylab.category.subcategory"`
- 保持明确性：`"com.company.project.analysis.v1"`
- 模式可能变更时包含版本号：`"mylab.metadata.v2"`

```python
# 定义命名空间
NS_QC = "org.mylab.quality_control"
NS_ANALYSIS = "org

print(f"文件注释ID: {ann.getId().getValue()}")
    print(f"  文件: {ann.getFile().getName().getValue()}")
    print(f"  大小: {ann.getFile().getSize().getValue()} 字节")
```

## 完整示例

```python
from omero.gateway import BlitzGateway
import omero.gateway
import omero.constants.metadata

HOST = 'omero.example.com'
PORT = 4064
USERNAME = 'user'
PASSWORD = 'pass'

with BlitzGateway(USERNAME, PASSWORD, host=HOST, port=PORT) as conn:
    # 获取数据集
    dataset = conn.getObject("Dataset", dataset_id)

    # 添加标签
    tag = omero.gateway.TagAnnotationWrapper(conn)
    tag.setValue("Analysis Complete")
    tag.save()
    dataset.linkAnnotation(tag)

    # 添加包含元数据的映射注释
    metadata = [
        ["分析日期", "2024-10-20"],
        ["软件", "CellProfiler 4.2"],
        ["流程", "cell_segmentation_v3"]
    ]
    map_ann = omero.gateway.MapAnnotationWrapper(conn)
    map_ann.setNs(omero.constants.metadata.NSCLIENTMAPANNOTATION)
    map_ann.setValue(metadata)
    map_ann.save()
    dataset.linkAnnotation(map_ann)

    # 添加文件注释
    file_ann = conn.createFileAnnfromLocalFile(
        "analysis_summary.pdf",
        mimetype="application/pdf",
        ns="mylab.reports",
        desc="分析总结报告"
    )
    dataset.linkAnnotation(file_ann)

    # 添加评论
    comment = omero.gateway.CommentAnnotationWrapper(conn)
    comment.setValue("数据集已准备就绪，待审核")
    comment.save()
    dataset.linkAnnotation(comment)

    print(f"向数据集 {dataset.getName()} 添加了4个注释")
```

## 最佳实践

1. **使用命名空间**：始终使用命名空间组织注释
2. **描述性标签**：采用清晰、一致的标签命名
3. **结构化元数据**：结构化数据优先使用映射注释而非评论
4. **文件组织**：使用描述性文件名和正确MIME类型
5. **链接复用**：重用现有标签避免重复创建
6. **批量操作**：通过循环处理多个对象提升效率
7. **错误处理**：链接前确认保存操作成功
8. **清理机制**：及时移除不再需要的临时注释
9. **文档记录**：为自定义命名空间编写说明文档
10. **权限管理**：在协作工作流中考虑注释所有权问题
