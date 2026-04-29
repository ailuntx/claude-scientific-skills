# 感兴趣区域（ROIs）

本文档涵盖在OMERO中创建、检索和分析ROI的方法。

## ROI概述

OMERO中的ROI（感兴趣区域）是用于标记图像特定区域的几何形状容器。每个ROI可包含多个形状，且形状可关联到特定Z切片和时间点。

### 支持的形状类型

- **矩形**：矩形区域
- **椭圆**：圆形和椭圆形区域
- **线段**：直线段
- **点**：单点标记
- **多边形**：多点多边形
- **掩膜**：基于像素的掩膜
- **折线**：多段线

## 创建ROI

### 辅助函数

```python
from omero.rtypes import rdouble, rint, rstring
import omero.model

def create_roi(conn, image, shapes):
    """
    创建ROI并关联形状对象

    参数:
        conn: BlitzGateway连接对象
        image: 图像对象
        shapes: 形状对象列表

    返回:
        已保存的ROI对象
    """
    roi = omero.model.RoiI()
    roi.setImage(image._obj)

    for shape in shapes:
        roi.addShape(shape)

    updateService = conn.getUpdateService()
    return updateService.saveAndReturnObject(roi)

def rgba_to_int(red, green, blue, alpha=255):
    """
    将RGBA值(0-255)转换为OMERO的整数编码

    参数:
        red, green, blue, alpha: 颜色值(0-255)

    返回:
        整数格式的颜色值
    """
    return int.from_bytes([red, green, blue, alpha],
                          byteorder='big', signed=True)
```

### 矩形ROI

```python
from omero.rtypes import rdouble, rint, rstring
import omero.model

# 获取图像
image = conn.getObject("Image", image_id)

# 定义位置和尺寸
x, y = 50, 100
width, height = 200, 150
z, t = 0, 0  # Z切片和时间点

# 创建矩形
rect = omero.model.RectangleI()
rect.x = rdouble(x)
rect.y = rdouble(y)
rect.width = rdouble(width)
rect.height = rdouble(height)
rect.theZ = rint(z)
rect.theT = rint(t)

# 设置标签和颜色
rect.textValue = rstring("细胞区域")
rect.fillColor = rint(rgba_to_int(255, 0, 0, 50))    # 红色半透明填充
rect.strokeColor = rint(rgba_to_int(255, 255, 0, 255))  # 黄色边框

# 创建ROI
roi = create_roi(conn, image, [rect])
print(f"已创建ROI ID: {roi.getId().getValue()}")
```

### 椭圆ROI

```python
# 中心位置和半径
center_x, center_y = 250, 250
radius_x, radius_y = 100, 75
z, t = 0, 0

# 创建椭圆
ellipse = omero.model.EllipseI()
ellipse.x = rdouble(center_x)
ellipse.y = rdouble(center_y)
ellipse.radiusX = rdouble(radius_x)
ellipse.radiusY = rdouble(radius_y)
ellipse.theZ = rint(z)
ellipse.theT = rint(t)
ellipse.textValue = rstring("细胞核")
ellipse.fillColor = rint(rgba_to_int(0, 255, 0, 50))

# 创建ROI
roi = create_roi(conn, image, [ellipse])
```

### 线段ROI

```python
# 线段端点
x1, y1 = 100, 100
x2, y2 = 300, 200
z, t = 0, 0

# 创建线段
line = omero.model.LineI()
line.x1 = rdouble(x1)
line.y1 = rdouble(y1)
line.x2 = rdouble(x2)
line.y2 = rdouble(y2)
line.theZ = rint(z)
line.theT = rint(t)
line.textValue = rstring("测量线段")
line.strokeColor = rint(rgba_to_int(0, 0, 255, 255))

# 创建ROI
roi = create_roi(conn, image, [line])
```

### 点ROI

```python
# 点位置
x, y = 150, 150
z, t = 0, 0

# 创建点
point = omero.model.PointI()
point.x = rdouble(x)
point.y = rdouble(y)
point.theZ = rint(z)
point.theT = rint(t)
point.textValue = rstring("特征点")

# 创建ROI
roi = create_roi(conn, image, [point])
```

### 多边形ROI

```python
from omero.model.enums import UnitsLength

# 定义顶点字符串 "x1,y1 x2,y2 x3,y3 ..."
vertices = "10,20 50,150 200,200 250,75"
z, t = 0, 0

# 创建多边形
polygon = omero.model.PolygonI()
polygon.points = rstring(vertices)
polygon.theZ = rint(z)
polygon.theT = rint(t)
polygon.textValue = rstring("细胞轮廓")

# 设置颜色和线宽
polygon.fillColor = rint(rgba_to_int(255, 0, 255, 50))
polygon.strokeColor = rint(rgba_to_int(255, 255, 0, 255))
polygon.strokeWidth = omero.model.LengthI(2, UnitsLength.PIXEL)

# 创建ROI
roi = create_roi(conn, image, [polygon])
```

### 掩膜ROI

```python
import numpy as np
import struct
import math

def create_mask_bytes(mask_array, bytes_per_pixel=1):
    """
    将二值掩膜数组转换为OMERO所需的位压缩字节

    参数:
        mask_array: 二值numpy数组(0和1)
        bytes_per_pixel: 1或2

    返回:
        OMERO掩膜所需的字节数组
    """
    if bytes_per_pixel == 2:
        divider = 16.0
        format_string = "H"
        byte_factor = 0.5
    elif bytes_per_pixel == 1:
        divider = 8.0
        format_string = "B"
        byte_factor = 1
    else:
        raise ValueError("bytes_per_pixel必须为1或2")

    mask_bytes = mask_array.astype(np.uint8).tobytes()
    steps = math.ceil(len(mask_bytes) / divider)
    packed_mask = []

    for i in range(int(steps)):
        binary = mask_bytes[i * int(divider):
                           i * int(divider) + int(divider)]
        format_str = str(int(byte_factor * len(binary))) + format_string
        binary = struct.unpack(format_str, binary)
        s = "".join(str(bit) for bit in binary)
        packed_mask.append(int(s, 2))

    return bytearray(packed_mask)

# 创建二值掩膜(0和1)
mask_w, mask_h = 100, 100
mask_array = np.fromfunction(
    lambda x, y: ((x - 50)**2 + (y - 50)**2) < 40**2,  # 圆形
    (mask_w, mask_h)
)

# 压缩掩膜
mask_packed = create_mask_bytes(mask_array, bytes_per_pixel=1)

# 掩膜位置
mask_x, mask_y = 50, 50
z, t, c = 0, 0, 0

# 创建掩膜
mask = omero.model.MaskI()
mask.setX(rdouble(mask_x))
mask.setY(rdouble(mask_y))
mask.setWidth(rdouble(mask_w))
mask.setHeight(rdouble(mask_h))
mask.setTheZ(rint(z))
mask.setTheT(rint(t))
mask.setTheC(rint(c))
mask.setBytes(mask_packed)
mask.textValue = rstring("分割掩膜")

# 设置颜色
from omero.gateway import ColorHolder
mask_color = ColorHolder()
mask_color.setRed(255)
mask_color.setGreen(0)
mask_color.setBlue(0)
mask_color.setAlpha(100)
mask.setFillColor(rint(mask_color.getInt()))

# 创建ROI
roi = create_roi(conn, image, [mask])
```

## 单ROI多形状

```python
# 为同一ROI创建多个形状
shapes = []

# 矩形
rect = omero.model.RectangleI()
rect.x = rdouble(100)
rect.y = rdouble(100)
rect.width = rdouble(50)
rect.height = rdouble(50)
rect.theZ = rint(0)
rect.theT = rint(0)
shapes.append(rect)

# 椭圆
ellipse = omero.model.EllipseI()
ellipse.x = rdouble(125)
ellipse.y = rdouble(125)
ellipse.radiusX = rdouble(20)
ellipse.radiusY = rdouble(20)
ellipse.theZ = rint(0)
ellipse.theT =

rect.theZ = rint(0)
    rect.theT = rint(0)
    rect.textValue = rstring("Auto ROI")

    roi = create_roi(conn, image, [rect])
    print(f"Created ROI for image {image.getName()}")
```

### 在Z轴堆栈上创建ROI

```python
image = conn.getObject("Image", image_id)
size_z = image.getSizeZ()

# 在每个Z切片上创建矩形
shapes = []
for z in range(size_z):
    rect = omero.model.RectangleI()
    rect.x = rdouble(100)
    rect.y = rdouble(100)
    rect.width = rdouble(50)
    rect.height = rdouble(50)
    rect.theZ = rint(z)
    rect.theT = rint(0)
    shapes.append(rect)

# 创建跨Z切片的单个ROI
roi = create_roi(conn, image, shapes)
```

## 完整示例

```python
from omero.gateway import BlitzGateway
from omero.rtypes import rdouble, rint, rstring
import omero.model

HOST = 'omero.example.com'
PORT = 4064
USERNAME = 'user'
PASSWORD = 'pass'

def rgba_to_int(r, g, b, a=255):
    return int.from_bytes([r, g, b, a], byteorder='big', signed=True)

with BlitzGateway(USERNAME, PASSWORD, host=HOST, port=PORT) as conn:
    # 获取图像
    image = conn.getObject("Image", image_id)
    print(f"Processing: {image.getName()}")

    # 创建多个ROI
    updateService = conn.getUpdateService()

    # ROI 1: 矩形
    roi1 = omero.model.RoiI()
    roi1.setImage(image._obj)

    rect = omero.model.RectangleI()
    rect.x = rdouble(50)
    rect.y = rdouble(50)
    rect.width = rdouble(100)
    rect.height = rdouble(100)
    rect.theZ = rint(0)
    rect.theT = rint(0)
    rect.textValue = rstring("Cell 1")
    rect.strokeColor = rint(rgba_to_int(255, 0, 0, 255))

    roi1.addShape(rect)
    roi1 = updateService.saveAndReturnObject(roi1)
    print(f"Created ROI 1: {roi1.getId().getValue()}")

    # ROI 2: 椭圆
    roi2 = omero.model.RoiI()
    roi2.setImage(image._obj)

    ellipse = omero.model.EllipseI()
    ellipse.x = rdouble(200)
    ellipse.y = rdouble(150)
    ellipse.radiusX = rdouble(40)
    ellipse.radiusY = rdouble(30)
    ellipse.theZ = rint(0)
    ellipse.theT = rint(0)
    ellipse.textValue = rstring("Cell 2")
    ellipse.strokeColor = rint(rgba_to_int(0, 255, 0, 255))

    roi2.addShape(ellipse)
    roi2 = updateService.saveAndReturnObject(roi2)
    print(f"Created ROI 2: {roi2.getId().getValue()}")

    # 检索并分析
    roi_service = conn.getRoiService()
    result = roi_service.findByImage(image_id, None)

    shape_ids = []
    for roi in result.rois:
        for shape in roi.copyShapes():
            shape_ids.append(shape.id.val)

    # 获取统计信息
    stats = roi_service.getShapeStatsRestricted(shape_ids, 0, 0, [0])

    for i, stat in enumerate(stats):
        print(f"Shape {shape_ids[i]}:")
        print(f"  Mean intensity: {stat.mean[0]:.2f}")
```

## 最佳实践

1. **组织图形**：将相关图形分组到单个ROI中
2. **标记图形**：使用textValue进行标识
3. **设置Z和T**：始终指定Z切片和时间点
4. **颜色编码**：为图形类型使用一致的颜色
5. **验证坐标**：确保图形在图像边界内
6. **批量创建**：尽可能在单个事务中创建多个ROI
7. **删除未使用的**：移除临时或测试用的ROI
8. **导出数据**：将ROI统计信息存储在表格中以供后续分析
9. **版本控制**：在注释中记录ROI创建方法
10. **性能优化**：使用图形统计服务而非手动提取像素
