# 脚本与批量操作

本文档涵盖创建用于服务器端处理和批量操作的 OMERO.scripts。

## OMERO.scripts 概述

OMERO.scripts 是在 OMERO 服务器上运行的 Python 脚本，可通过 OMERO 客户端（网页端、Insight、CLI）调用。它们作为插件扩展 OMERO 功能。

### 核心特性

- **服务器端执行**：脚本在服务器运行，避免数据传输
- **客户端集成**：支持所有 OMERO 客户端调用并自动生成 UI
- **参数处理**：定义带验证的输入参数
- **结果报告**：向客户端返回图像、文件或消息
- **批量处理**：高效处理多幅图像或数据集

## 基础脚本结构

### 最小脚本模板

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import omero
from omero.gateway import BlitzGateway
import omero.scripts as scripts
from omero.rtypes import rlong, rstring, robject

def run_script():
    """
    主脚本函数
    """
    # 脚本定义
    client = scripts.client(
        'Script_Name.py',
        """
        本脚本功能描述
        """,

        # 输入参数
        scripts.String("Data_Type", optional=False, grouping="1",
                      description="选择图像来源",
                      values=[rstring('Dataset'), rstring('Image')],
                      default=rstring('Dataset')),

        scripts.Long("IDs", optional=False, grouping="2",
                    description="数据集或图像ID").ofType(rlong(0)),

        # 输出定义
        namespaces=[omero.constants.namespaces.NSDYNAMIC],
        version="1.0"
    )

    try:
        # 获取连接
        conn = BlitzGateway(client_obj=client)

        # 获取脚本参数
        script_params = client.getInputs(unwrap=True)
        data_type = script_params["Data_Type"]
        ids = script_params["IDs"]

        # 处理数据
        message = process_data(conn, data_type, ids)

        # 返回结果
        client.setOutput("Message", rstring(message))

    finally:
        client.closeSession()

def process_data(conn, data_type, ids):
    """
    基于参数处理图像
    """
    # 实现逻辑
    return "处理完成"

if __name__ == "__main__":
    run_script()
```

## 脚本参数

### 参数类型

```python
# 字符串参数
scripts.String("Name", optional=False,
              description="输入名称")

# 带选项的字符串
scripts.String("Mode", optional=False,
              values=[rstring('Fast'), rstring('Accurate')],
              default=rstring('Fast'))

# 整型参数
scripts.Long("ImageID", optional=False,
            description="待处理图像").ofType(rlong(0))

# 整型列表
scripts.List("ImageIDs", optional=False,
            description="多幅图像").ofType(rlong(0))

# 浮点参数
scripts.Float("Threshold", optional=True,
             description="阈值",
             min=0.0, max=1.0, default=0.5)

# 布尔参数
scripts.Bool("SaveResults", optional=True,
            description="保存结果到OMERO",
            default=True)
```

### 参数分组

```python
# 关联参数分组
scripts.String("Data_Type", grouping="1",
              description="来源类型",
              values=[rstring('Dataset'), rstring('Image')])

scripts.Long("Dataset_ID", grouping="1.1",
            description="数据集ID").ofType(rlong(0))

scripts.List("Image_IDs", grouping="1.2",
            description="图像ID列表").ofType(rlong(0))
```

## 访问输入数据

### 获取脚本参数

```python
# 在 run_script() 内
client = scripts.client(...)

# 获取Python对象格式参数
script_params = client.getInputs(unwrap=True)

# 访问独立参数
data_type = script_params.get("Data_Type", "Image")
image_ids = script_params.get("Image_IDs", [])
threshold = script_params.get("Threshold", 0.5)
save_results = script_params.get("SaveResults", True)
```

### 从参数获取图像

```python
def get_images_from_params(conn, script_params):
    """
    基于脚本参数获取图像对象
    """
    images = []

    data_type = script_params["Data_Type"]

    if data_type == "Dataset":
        dataset_id = script_params["Dataset_ID"]
        dataset = conn.getObject("Dataset", dataset_id)
        if dataset:
            images = list(dataset.listChildren())

    elif data_type == "Image":
        image_ids = script_params["Image_IDs"]
        for image_id in image_ids:
            image = conn.getObject("Image", image_id)
            if image:
                images.append(image)

    return images
```

## 处理图像

### 批量图像处理

```python
def process_images(conn, images, threshold):
    """
    处理多幅图像
    """
    results = []

    for image in images:
        print(f"处理中: {image.getName()}")

        # 获取像素数据
        pixels = image.getPrimaryPixels()
        size_z = image.getSizeZ()
        size_c = image.getSizeC()
        size_t = image.getSizeT()

        # 处理每个平面
        for z in range(size_z):
            for c in range(size_c):
                for t in range(size_t):
                    plane = pixels.getPlane(z, c, t)

                    # 应用阈值
                    binary = (plane > threshold).astype(np.uint8)

                    # 特征计数
                    feature_count = count_features(binary)

                    results.append({
                        'image_id': image.getId(),
                        'image_name': image.getName(),
                        'z': z, 'c': c, 't': t,
                        'feature_count': feature_count
                    })

    return results
```

## 生成输出

### 返回消息

```python
# 简单消息
message = "成功处理10幅图像"
client.setOutput("Message", rstring(message))

# 详细消息
message = "结果:\n"
for result in results:
    message += f"图像 {result['image_id']}: {result['count']} 个细胞\n"
client.setOutput("Message", rstring(message))
```

### 返回图像

```python
# 返回新建图像
new_image = conn.createImageFromNumpySeq(...)
client.setOutput("New_Image", robject(new_image._obj))
```

### 返回文件

```python
# 创建并返回文件注释
file_ann = conn.createFileAnnfromLocalFile(
    output_file_path,
    mimetype="text/csv",
    ns="analysis.results"
)

client.setOutput("Result_File", robject(file_ann._obj))
```

### 返回表格

```python
# 创建OMERO表格并返回
resources = conn.c.sf.sharedResources()
table = create_results_table(resources, results)
orig_file = table.getOriginalFile()
table.close()

# 创建文件注释
file_ann = omero.model.FileAnnotationI()
file_ann.setFile(orig_file)
file_ann = conn.getUpdateService().saveAndReturnObject(file_ann)

client.setOutput("Results_Table", robject(file_ann._obj))
```

## 完整示例脚本

### 示例1：最大强度投影

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import omero
from omero.gateway import BlitzGateway
import omero.scripts as scripts
from omero.rtypes import rlong, rstring, robject
import numpy as np

def run_script():
    client = scripts.client(
        'Maximum_Intensity_Projection.py',
        """
        从Z轴堆栈图像创建最大强度投影
        """,

        scripts.String("Data_Type", optional=False, grouping="1",
                      description="处理来源",
                      values=[rstring('Dataset'), rstring('Image')],
                      default=rstring('Image')),

        scripts.List("IDs", optional=False, grouping="2",
                    description="数据集或图像ID列表").ofType(rlong(0)),

        scripts.Bool("Link_to_Source", optional=True, grouping="3",
                    description="将结果链接到源数据集",
                    default=True),

        version="1.0"
    )

    try:
        conn = BlitzGateway(client_obj=client)
        script_params = client.getInputs(unwrap=True)

        # 获取图像
        images = get_images(conn, script_params)
        created_images = []

        for image in images:
            print(f"处理中: {image.getName()}")

            # 创建MIP
            mip_image = create_mip(conn, image)
            if mip_image:
                created_images.append(mip_image)

        # 报告结果
        if created_images:
            message = f"已创建 {len(created_images)} 幅MIP图像"
            # 返回首幅图像用于显示
            client.setOutput("Message", rstring(message))
            client.setOutput("Result", robject(created_images[0]._obj))
        else:
            client.setOutput("Message", rstring("未创建图像"))

    finally:
        client.closeSession()

def get_images(conn, script_params):
    """从脚本参数获取图像"""
    images = []
    data_type = script_params["Data_Type"]
    ids = script_params["IDs"]

    if data_type == "Dataset":
        for dataset_id in ids:
            dataset = conn.getObject("Dataset", dataset_id)
            if dataset:
                images.extend(list(dataset.listChildren()))
    else:
        for image_id in ids:
            image = conn.getObject("Image", image_id)
            if image:
                images.append(image)

    return images

def create_mip(conn, source_image):
    """创建最大强度投影"""
    pixels = source_image.getPrimaryPixels()
    size_z = source_image.getSizeZ()
    size_c = source_image.getSizeC()
    size_t = source_image.getSizeT()

    if size_z == 1:
        print("  跳过（单Z层）")
        return None

    def plane_gen():
        for c in range(size_c):
            for t in range(size_t):
                # 获取Z轴堆栈
                z_stack = []
                for z in range(size_z):
                    plane = pixels.getPlane(z, c, t)
                    z_stack.append(plane)

                # 最大投影
                max_proj = np.max(z_stack, axis=0)
                yield max_proj

    # 创建新图像
    new_image = conn.createImageFromNumpySeq(
        plane_gen(),
        f"{source_image.getName()}_MIP",
        1, size_c, size_t,
        description="最大强度投影",
        dataset=source_image.getParent()
    )

    return new_image

if __name__ == "__main__":
    run_script()
```

### 示例2：批量ROI分析

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import omero
from omero.gateway import BlitzGateway
import omero.scripts as scripts
from omero.rtypes import rlong, rstring, robject
import omero.grid

def run_script():
    client = scripts.client(
        'Batch_ROI_Analysis.py',
        """
        分析多幅图像的ROI并创建结果表格
        """,

        scripts.Long("Dataset_ID", optional=False,
                    description="包含图像和ROI的数据集").ofType(rlong(0)),

        scripts.Long("Channel_Index", optional=True,
                    description="分析通道（0起始）",
                    default=0, min=0),

        version="1.0"
    )

    try:
        conn = BlitzGateway(client_obj=client)
        script_params = client.getInputs(unwrap=True)

        dataset_id = script_params["Dataset_ID"]
        channel_index = script_params["Channel_Index"]

        # 获取数据集
        dataset = conn.getObject("Dataset", dataset_id)
        if not dataset:
            client.setOutput("Message", rstring("未找到数据集"))
            return

        # 分析ROI
        results = analyze_rois(conn, dataset, channel_index)

        # 创建表格
        table_file = create_results_table(conn, dataset, results)

        # 报告
        message = f"已分析 {len(results)} 个ROI（来自 {dataset.getName()}）"
        client.setOutput("Message", rstring(message))
        client.setOutput("Results_Table", robject(table_file._obj))

    finally:
        client.closeSession()

def analyze_rois(conn, dataset, channel_index):
    """分析数据集图像中的所有ROI"""
    roi_service = conn.getRoiService()
    results = []

    for image in dataset.listChildren():
        result = roi_service.findByImage(image.getId(), None)

        if not result.rois:
            continue

        # 获取形状ID
        shape_ids = []
        for roi in result.rois:
            for shape in roi.copyShapes():
                shape_ids.append(shape.id.val)

        # 获取统计信息
        stats = roi_service.getShapeStatsRestricted(
            shape_ids, 0, 0, [channel_index]
        )

        # 存储结果
        for i, stat in enumerate(stats):
            results.append({
                'image_id': image.getId(),
                'image_name': image.getName(),
                'shape_id': shape_ids[i],
                'mean': stat.mean[channel_index],
                'min': stat.min[channel_index],
                'max': stat.max[channel_index],
                'sum': stat.sum[channel_index],
                'area': stat.pointsCount[channel_index]
            })

    return results

def create_results_table(conn, dataset, results):
    """从结果创建OMERO表格"""
    # 准备数据
    image_ids = [r['image_id'] for r in results]
    shape_ids = [r['shape_id'] for r in results]
    means = [r['mean'] for r in results]
    mins = [r['min'] for r in results]
    maxs = [r['max'] for r in results]
    sums = [r['sum'] for r in results]
    areas = [r['area'] for r in results]

    # 创建表格
    resources = conn.c.sf.sharedResources()
    repository_id = resources.repositories().descriptions[0].getId().getValue()
    table = resources.newTable(repository_id, f"ROI分析_{dataset.getId()}")

    # 定义列
    columns = [
        omero.grid.ImageColumn('Image', '源图像', []),
        omero.grid.LongColumn('ShapeID', 'ROI形状ID', []),
        omero.grid.DoubleColumn('Mean', '平均强度', []),
        omero.grid.DoubleColumn('Min', '最小强度', []),
        omero.grid.DoubleColumn('Max', '最大强度', []),
        omero.grid.DoubleColumn('Sum', '积分光密度', []),
        omero.grid.LongColumn('Area', '像素面积', [])
    ]
    table.initialize(columns)

    # 添加数据
    data = [
        omero.grid.ImageColumn('Image', '源图像', image_ids),
        omero.grid.LongColumn('ShapeID', 'ROI形状ID', shape_ids),
        omero.grid.DoubleColumn('Mean', '平均强度', means),
        omero.grid.DoubleColumn('Min', '最小强度', mins),
        omero.grid.DoubleColumn('Max', '最大强度', maxs),
        omero.grid.DoubleColumn('Sum', '积分光密度', sums),
        omero.grid.LongColumn('Area', '像素面积', areas)
    ]
    table.addData(data)

    orig_file = table.getOriginalFile()
    table.close()

    # 链接到数据集
    file_ann = omero.model.FileAnnotationI()
    file_ann.setFile(orig_file)
    file_ann = conn.getUpdateService().saveAndReturnObject(file_ann)

    link = omero.model.DatasetAnnotationLinkI()
    link.setParent(dataset._obj)
    link.setChild(file_ann)
    conn.getUpdateService().saveAndReturnObject(link)

    return file_ann

if __name__ == "__main__":
    run_script()
```

## 脚本部署

### 安装位置

脚本应放置在 OMERO 服务器的脚本目录：
```
OMERO_DIR/lib/scripts/
```

### 推荐结构

```
lib/scripts/
├── analysis/
│   ├── Cell_Counter.py
│   └── ROI_Analyzer.py
├── export/
│   ├── Export_Images.py
│   └── Export_ROIs.py
└── util/
    └── Helper_Functions.py
```

### 测试脚本

```bash
# 测试脚本语法
python Script_Name.py

# 上传到OMERO
omero script upload Script_Name.py

# 列出脚本
omero script list

# 从CLI运行脚本
omero script launch Script_ID Dataset_ID=123
```

## 最佳实践

1. **错误处理**：始终使用 try-finally 关闭会话  
2. **进度更新**：对长时间操作打印状态消息  
3. **参数验证**：处理前检查参数  
4. **内存管理**：分批处理大型数据集  
5. **文档规范**：包含清晰的描述和参数文档  
6. **版本控制**：在脚本中包含版本号  
7. **命名空间**：为输出使用适当的命名空间  
8. **返回对象**：返回创建的对象供客户端显示  
9. **日志记录**：使用 print() 记录服务器日志  
10. **测试要求**：使用多种输入组合测试  

## 常用模式  

### 进度报告  

```python
total = len(images)
for idx, image in enumerate(images):
    print(f"Processing {idx + 1}/{total}: {image.getName()}")
    # 处理图像
```

### 错误收集  

```python
errors = []
for image in images:
    try:
        process_image(image)
    except Exception as e:
        errors.append(f"{image.getName()}: {str(e)}")

if errors:
    message = "处理完成，但出现以下错误：\n" + "\n".join(errors)
else:
    message = "所有图像已成功处理"
```

### 资源清理  

```python
try:
    # 脚本处理
    pass
finally:
    # 清理临时文件
    if os.path.exists(temp_file):
        os.remove(temp_file)
    client.closeSession()
```
