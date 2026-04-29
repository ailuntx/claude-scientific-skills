# 图像处理与渲染

本文档涵盖在 OMERO 中访问原始像素数据、图像渲染以及创建新图像的相关内容。

## 访问原始像素数据

### 获取单层平面

```python
# 获取图像
image = conn.getObject("Image", image_id)

# 获取维度
size_z = image.getSizeZ()
size_c = image.getSizeC()
size_t = image.getSizeT()

# 获取像素对象
pixels = image.getPrimaryPixels()

# 获取单层平面（返回NumPy数组）
z, c, t = 0, 0, 0  # 第一层Z切面、通道和时间点
plane = pixels.getPlane(z, c, t)

print(f"形状: {plane.shape}")
print(f"数据类型: {plane.dtype.name}")
print(f"最小值: {plane.min()}, 最大值: {plane.max()}")
```

### 获取多层平面

```python
import numpy as np

# 获取特定通道和时间点的Z堆栈
pixels = image.getPrimaryPixels()
c, t = 0, 0  # 第一通道和时间点

# 构建(z, c, t)坐标列表
zct_list = [(z, c, t) for z in range(size_z)]

# 一次性获取所有平面
planes = pixels.getPlanes(zct_list)

# 堆叠成3D数组
z_stack = np.array([p for p in planes])
print(f"Z堆栈形状: {z_stack.shape}")
```

### 获取超立方体（5D数据子集）

```python
# 获取5D数据子集(Z, C, T)
zct_list = []
for z in range(size_z // 2, size_z):  # Z轴后半部分
    for c in range(size_c):           # 所有通道
        for t in range(size_t):       # 所有时间点
            zct_list.append((z, c, t))

# 获取平面
planes = pixels.getPlanes(zct_list)

# 处理每个平面
for i, plane in enumerate(planes):
    z, c, t = zct_list[i]
    print(f"平面 Z={z}, C={c}, T={t}: 最小值={plane.min()}, 最大值={plane.max()}")
```

### 获取图块（感兴趣区域）

```python
# 定义图块坐标
x, y = 50, 50          # 左上角坐标
width, height = 100, 100  # 图块尺寸
tile = (x, y, width, height)

# 获取特定Z,C,T的图块
z, c, t = 0, 0, 0
zct_list = [(z, c, t, tile)]

tiles = pixels.getTiles(zct_list)
tile_data = tiles[0]

print(f"图块形状: {tile_data.shape}")  # 应为(height, width)
```

### 获取多个图块

```python
# 从Z堆栈获取图块
c, t = 0, 0
tile = (50, 50, 100, 100)  # x, y, width, height

# 构建带图块的列表
zct_list = [(z, c, t, tile) for z in range(size_z)]

tiles = pixels.getTiles(zct_list)

for i, tile_data in enumerate(tiles):
    print(f"图块 Z={i}: {tile_data.shape}, 最小值={tile_data.min()}")
```

## 图像直方图

### 获取直方图

```python
# 获取第一通道直方图
channel_index = 0
num_bins = 256
z, t = 0, 0

histogram = image.getHistogram([channel_index], num_bins, False, z, t)
print(f"直方图分箱数: {len(histogram)}")
print(f"前10个分箱: {histogram[:10]}")
```

### 多通道直方图

```python
# 获取所有通道直方图
channels = list(range(image.getSizeC()))
histograms = image.getHistogram(channels, 256, False, 0, 0)

for c, hist in enumerate(histograms):
    print(f"通道 {c}: 总像素数 = {sum(hist)}")
```

## 图像渲染

### 使用当前设置渲染图像

```python
from PIL import Image
from io import BytesIO

# 获取图像
image = conn.getObject("Image", image_id)

# 在特定Z和T渲染
z = image.getSizeZ() // 2  # 中间Z切面
t = 0

rendered_image = image.renderImage(z, t)
# rendered_image是PIL图像对象
rendered_image.save("rendered_image.jpg")
```

### 获取缩略图

```python
from PIL import Image
from io import BytesIO

# 获取缩略图（使用当前渲染设置）
thumbnail_data = image.getThumbnail()

# 转换为PIL图像
thumbnail = Image.open(BytesIO(thumbnail_data))
thumbnail.save("thumbnail.jpg")

# 获取特定尺寸缩略图
thumbnail_data = image.getThumbnail(size=(96, 96))
thumbnail = Image.open(BytesIO(thumbnail_data))
```

## 渲染设置

### 查看当前设置

```python
# 显示渲染设置
print("当前渲染设置:")
print(f"灰度模式: {image.isGreyscaleRenderingModel()}")
print(f"默认Z: {image.getDefaultZ()}")
print(f"默认T: {image.getDefaultT()}")
print()

# 通道设置
print("通道设置:")
for idx, channel in enumerate(image.getChannels()):
    print(f"通道 {idx + 1}:")
    print(f"  标签: {channel.getLabel()}")
    print(f"  颜色: {channel.getColor().getHtml()}")
    print(f"  激活状态: {channel.isActive()}")
    print(f"  窗口: {channel.getWindowStart()} - {channel.getWindowEnd()}")
    print(f"  最小/最大值: {channel.getWindowMin()} - {channel.getWindowMax()}")
```

### 设置渲染模式

```python
# 切换到灰度渲染
image.setGreyscaleRenderingModel()

# 切换到彩色渲染
image.setColorRenderingModel()
```

### 设置激活通道

```python
# 激活特定通道（1起始索引）
image.setActiveChannels([1, 3])  # 仅通道1和3

# 激活所有通道
all_channels = list(range(1, image.getSizeC() + 1))
image.setActiveChannels(all_channels)

# 激活单个通道
image.setActiveChannels([2])
```

### 设置通道颜色

```python
# 设置通道颜色（十六进制格式）
channels = [1, 2, 3]
colors = ['FF0000', '00FF00', '0000FF']  # 红,绿,蓝

image.setActiveChannels(channels, colors=colors)

# 使用None保留现有颜色
colors = ['FF0000', None, '0000FF']  # 保留通道2的颜色
image.setActiveChannels(channels, colors=colors)
```

### 设置通道窗口（强度范围）

```python
# 设置通道强度窗口
channels = [1, 2]
windows = [
    [100.0, 500.0],  # 通道1: 100-500
    [50.0, 300.0]    # 通道2: 50-300
]

image.setActiveChannels(channels, windows=windows)

# 使用None保留现有窗口
windows = [[100.0, 500.0], [None, None]]
image.setActiveChannels(channels, windows=windows)
```

### 设置默认Z和T

```python
# 设置默认Z切面和时间点
image.setDefaultZ(5)
image.setDefaultT(0)

# 使用默认值渲染
rendered_image = image.renderImage(z=None, t=None)
rendered_image.save("default_rendering.jpg")
```

## 渲染独立通道

### 分别渲染每个通道

```python
# 设置灰度模式
image.setGreyscaleRenderingModel()

z = image.getSizeZ() // 2
t = 0

# 渲染每个通道
for c in range(1, image.getSizeC() + 1):
    image.setActiveChannels([c])
    rendered = image.renderImage(z, t)
    rendered.save(f"channel_{c}.jpg")
```

### 渲染多通道合成图

```python
# 前3通道的彩色合成
image.setColorRenderingModel()
channels = [1, 2, 3]
colors = ['FF0000', '00FF00', '0000FF']  # RGB

image.setActiveChannels(channels, colors=colors)
rendered = image.renderImage(z, t)
rendered.save("rgb_composite.jpg")
```

## 图像投影

### 最大强度投影

```python
# 设置投影类型
image.setProjection('intmax')

# 渲染（跨所有Z投影）
z, t = 0, 0  # Z值在投影中被忽略
rendered = image.renderImage(z, t)
rendered.save("max_projection.jpg")

# 重置为普通渲染
image.setProjection('normal')
```

### 平均强度投影

```python
image.setProjection('intmean')
rendered = image.renderImage(z, t)
rendered.save("mean_projection.jpg")
image.setProjection('normal')
```

### 可用投影类型

- `'normal'`: 无投影（默认）
- `'intmax'`: 最大强度投影
- `'intmean'`: 平均强度投影
- `'intmin'`: 最小强度投影（如支持）

## 保存与重置渲染设置

### 保存当前设置为默认值

```python
# 修改渲染设置
image.setActiveChannels([1, 2])
image.setDefaultZ(5)

# 保存为新默认值
image.saveDefaults()
```

### 重置为导入设置

```python
# 重置为原始导入设置
image.resetDefaults(save=True)
```

## 从NumPy数组创建图像

### 创建简单图像

```python
import numpy as np

# 创建样本数据
size_x, size_y = 512, 512
size_z, size_c, size_t = 10, 2, 1

# 生成平面
def plane_generator():
    """生成器函数，逐层产生平面"""
    for z in range(size_z):
        for c in range(size_c):
            for t in range(size_t):
                # 创建合成数据
                plane = np.random.randint(0, 255, (size_y, size_x), dtype=np.uint8)
                yield plane

# 创建图像
image = conn.createImageFromNumpySeq(
    plane_generator(),
    "测试图像",
    size_z, size_c, size_t,
    description="从NumPy数组创建的图像",
    dataset=None
)

print(f"已创建图像ID: {image.getId()}")
```

### 从硬编码数组创建图像

```python
from numpy import array, int8

# 定义维度
size_x, size_y = 5, 4
size_z, size_c, size_t = 1, 2, 1

# 创建平面
plane1 = array(
    [[0, 1, 2, 3, 4],
     [5, 6, 7, 8, 9],
     [0, 1, 2, 3, 4],
     [5, 6, 7, 8, 9]],
    dtype=int8
)

plane2 = array(
    [[5, 6, 7, 8, 9],
     [0, 1, 2, 3, 4],
     [5, 6, 7, 8, 9],
     [0, 1, 2, 3, 4]],
    dtype=int8
)

planes = [plane1, plane2]

def plane_gen():
    for p in planes:
        yield p

# 创建图像
desc = "从硬编码数组创建的图像"
image = conn.createImageFromNumpySeq(
    plane_gen(),
    "numpy图像",
    size_z, size_c, size_t,
    description=desc,
    dataset=None
)

print(f"已创建图像: {image.getName()} (ID: {image.getId()})")
```

### 在数据集中创建图像

```python
# 获取目标数据集
dataset = conn.getObject("Dataset", dataset_id)

# 创建图像
image = conn.createImageFromNumpySeq(
    plane_generator(),
    "新分析结果",
    size_z, size_c, size_t,
    description="分析流程结果",
    dataset=dataset  # 添加到数据集
)
```

### 创建衍生图像

```python
# 获取源图像
source = conn.getObject("Image", source_image_id)
size_z = source.getSizeZ()
size_c = source.getSizeC()
size_t = source.getSizeT()
dataset = source.getParent()

pixels = source.getPrimaryPixels()
new_size_c = 1  # 平均通道数

def plane_gen():
    """通道平均处理"""
    for z in range(size_z):
        for c in range(new_size_c):
            for t in range(size_t):
                # 获取多个通道
                channel0 = pixels.getPlane(z, 0, t)
                channel1 = pixels.getPlane(z, 1, t)

                # 合并通道
                new_plane = (channel0.astype(float) + channel1.astype(float)) / 2
                new_plane = new_plane.astype(channel0.dtype)

                yield new_plane

# 创建新图像
desc = "源图像通道平均结果"
derived = conn.createImageFromNumpySeq(
    plane_gen(),
    f"{source.getName()}_平均",
    size_z, new_size_c, size_t,
    description=desc,
    dataset=dataset
)

print(f"已创建衍生图像: {derived.getId()}")
```

## 设置物理尺寸

### 设置带单位的像素尺寸

```python
from omero.model.enums import UnitsLength
import omero.model

# 获取图像
image = conn.getObject("Image", image_id)

# 创建单位对象
pixel_size_x = omero.model.LengthI(0.325, UnitsLength.MICROMETER)
pixel_size_y = omero.model.LengthI(0.325, UnitsLength.MICROMETER)
pixel_size_z = omero.model.LengthI(1.0, UnitsLength.MICROMETER)

# 获取像素对象
pixels = image.getPrimaryPixels()._obj

# 设置物理尺寸
pixels.setPhysicalSizeX(pixel_size_x)
pixels.setPhysicalSizeY(pixel_size_y)
pixels.setPhysicalSizeZ(pixel_size_z)

# 保存更改
conn.getUpdateService().saveObject(pixels)

print("已更新像素尺寸")
```

### 可用长度单位

来自`omero.model.enums.UnitsLength`:
- `ANGSTROM`
- `NANOMETER`
- `MICROMETER`
- `MILLIMETER`
- `CENTIMETER`
- `METER`
- `PIXEL`

### 设置新图像的像素尺寸

```python
from omero.model.enums import UnitsLength
import omero.model

# 创建图像
image = conn.createImageFromNumpySeq(
    plane_generator(),
    "带尺寸的新图像",
    size_z, size_c, size_t
)

# 设置像素尺寸
pixel_size = omero.model.LengthI(0.5, UnitsLength.MICROMETER)
pixels = image.getPrimaryPixels()._obj
pixels.setPhysicalSizeX(pixel_size)
pixels.setPhysicalSizeY(pixel_size)

z_size = omero.model.LengthI(2.0, UnitsLength.MICROMETER)
pixels.setPhysicalSizeZ(z_size)

conn.getUpdateService().saveObject(pixels)
```

## 完整示例：图像处理流程

```python
from omero.gateway import BlitzGateway
import numpy as np

HOST = 'omero.example.com'
PORT = 4064
USERNAME = 'user'
PASSWORD = 'pass'

with BlitzGateway(USERNAME, PASSWORD, host=HOST, port=PORT) as conn:
    # 获取源图像
    source = conn.getObject("Image", source_image_id)
    print(f"处理中: {source.getName()}")

    # 获取维度
    size_x = source.getSizeX()
    size_y = source.getSizeY()
    size_z = source.getSizeZ()
    size_c = source.getSizeC()
    size_t = source.getSizeT()

    pixels = source.getPrimaryPixels()

    # 处理：Z轴最大强度投影
    def plane_gen():
        for c in range(size_c):
            for t in range(size_t):
                # 获取当前C,T的所有Z平面
                z_stack = []
                for z in range(size_z):
                    plane = pixels.getPlane(z, c, t)
                    z_stack.append(plane)

                # 最大投影
                max_proj = np.max(z_stack, axis=0)
                yield max_proj

    # 创建结果图像（单Z切面）
    result = conn

```markdown
1, size_c, size_t,  # 投影时Z=1
        description="最大强度投影",
        dataset=source.getParent()
    )

    print(f"创建MIP图像: {result.getId()}")

    # 复制像素尺寸（仅X和Y方向，投影不需要Z）
    from omero.model.enums import UnitsLength
    import omero.model

    source_pixels = source.getPrimaryPixels()._obj
    result_pixels = result.getPrimaryPixels()._obj

    result_pixels.setPhysicalSizeX(source_pixels.getPhysicalSizeX())
    result_pixels.setPhysicalSizeY(source_pixels.getPhysicalSizeY())

    conn.getUpdateService().saveObject(result_pixels)

    print("处理完成")
```

## 处理不同的数据类型

### 处理各种像素类型

```python
# 获取像素类型
pixel_type = image.getPixelsType()
print(f"像素类型: {pixel_type}")

# 常见类型：uint8, uint16, uint32, int8, int16, int32, float, double

# 获取正确数据类型的平面
plane = pixels.getPlane(z, c, t)
print(f"NumPy数据类型: {plane.dtype}")

# 处理时按需转换
if plane.dtype == np.uint16:
    # 转换为浮点数以便处理
    plane_float = plane.astype(np.float32)
    # 处理...
    # 转换回原类型
    result = plane_float.astype(np.uint16)
```

### 处理大图像

```python
# 以分块方式处理大图像以节省内存
tile_size = 512
size_x = image.getSizeX()
size_y = image.getSizeY()

for y in range(0, size_y, tile_size):
    for x in range(0, size_x, tile_size):
        # 获取分块尺寸
        w = min(tile_size, size_x - x)
        h = min(tile_size, size_y - y)
        tile = (x, y, w, h)

        # 获取分块数据
        zct_list = [(z, c, t, tile)]
        tile_data = pixels.getTiles(zct_list)[0]

        # 处理分块
        # ...
```

## 最佳实践

1. **使用生成器**：创建图像时使用生成器，避免将所有数据加载到内存
2. **指定数据类型**：将NumPy数据类型与OMERO像素类型匹配
3. **设置物理尺寸**：始终为新图像设置像素尺寸
4. **分块处理大图像**：以分块方式处理大图像以管理内存
5. **关闭连接**：完成后始终关闭连接
6. **渲染效率**：渲染多张图像时缓存渲染设置
7. **通道索引**：注意：通道在渲染时是1索引，在像素访问时是0索引
8. **保存设置**：如果渲染设置需要永久保存，则进行保存
9. **压缩**：在renderImage()中使用压缩参数以加快传输速度
10. **错误处理**：检查None返回值并处理异常
```
