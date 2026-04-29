# PyLabRobot 中的资源管理

## 概述

PyLabRobot 中的资源代表实验室设备、耗材或协议中使用的组件。资源系统通过分层结构管理板、吸头盒、槽、试管、载架等耗材，提供精确的空间定位和状态跟踪功能。

## 资源基础

### 什么是资源？

资源可表示：
- 实验室耗材（微孔板、吸头盒、槽、试管）
- 设备（液体处理器、读板器）
- 耗材组件（孔、吸头）
- 耗材容器（工作台面、载架）

所有资源均继承自基础类 `Resource`，并通过父子关系形成树状结构。

### 资源属性

每个资源必须包含：
- **name**：唯一标识符
- **size_x, size_y, size_z**：毫米为单位的尺寸（长方体表示）
- **location**：相对于父级原点的坐标（可选，在分配时设置）

```python
from pylabrobot.resources import Resource

# 创建基础资源
resource = Resource(
    name="my_resource",
    size_x=127.76,  # mm
    size_y=85.48,   # mm
    size_z=14.5     # mm
)
```

## 资源类型

### 微孔板

带有液体容器的孔板：

```python
from pylabrobot.resources import (
    Cos_96_DW_1mL,      # 96孔深孔板，1mL容量
    Cos_96_DW_500ul,    # 96孔深孔板，500µL容量
    Plate_384_Sq,       # 384孔方孔板
    Cos_96_PCR          # 96孔PCR板
)

# 创建孔板
plate = Cos_96_DW_1mL(name="sample_plate")

# 访问孔位
well_a1 = plate["A1"]                  # 单孔
row_a = plate["A1:H1"]                 # 整行 (A1-H1)
col_1 = plate["A1:A12"]                # 整列 (A1-A12)
range_wells = plate["A1:C3"]           # 孔位范围
all_wells = plate.children             # 所有孔位列表
```

### 吸头盒

存放移液器吸头的容器：

```python
from pylabrobot.resources import (
    TIP_CAR_480_A00,    # 96标准吸头
    HTF_L,              # Hamilton 过滤吸头
    TipRack             # 通用吸头盒
)

# 创建吸头盒
tip_rack = TIP_CAR_480_A00(name="tips")

# 访问吸头
tip_a1 = tip_rack["A1"]                # 单吸头位置
tips_row = tip_rack["A1:H1"]           # 吸头行
tips_col = tip_rack["A1:A12"]          # 吸头列

# 检查吸头存在状态（需启用吸头追踪）
from pylabrobot.resources import set_tip_tracking
set_tip_tracking(True)

has_tip = tip_rack["A1"].tracker.has_tip
```

### 储液槽

试剂储液容器：

```python
from pylabrobot.resources import Trough_100ml

# 创建储液槽
trough = Trough_100ml(name="buffer")

# 访问通道
channel_1 = trough["channel_1"]
all_channels = trough.children
```

### 试管

独立试管或试管架：

```python
from pylabrobot.resources import Tube, TubeRack

# 创建试管架
tube_rack = TubeRack(name="samples")

# 访问试管
tube_a1 = tube_rack["A1"]
```

### 载架

用于承载板、吸头盒等耗材的平台：

```python
from pylabrobot.resources import (
    PlateCarrier,
    TipCarrier,
    MFXCarrier
)

# 载架提供耗材放置位
carrier = PlateCarrier(name="plate_carrier")

# 将孔板分配至载架
plate = Cos_96_DW_1mL(name="plate")
carrier.assign_child_resource(plate, location=(0, 0, 0))
```

## 工作台面管理

### 操作工作台面

工作台面代表机器人的工作区域：

```python
from pylabrobot.resources import STARLetDeck, OTDeck

# Hamilton STARlet 工作台面
deck = STARLetDeck()

# Opentrons OT-2 工作台面
deck = OTDeck()
```

### 分配资源至工作台面

通过轨道或坐标将资源分配到指定位置：

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.resources import STARLetDeck, TIP_CAR_480_A00, Cos_96_DW_1mL

lh = LiquidHandler(backend=backend, deck=STARLetDeck())

# 使用轨道位置分配 (Hamilton STAR)
tip_rack = TIP_CAR_480_A00(name="tips")
source_plate = Cos_96_DW_1mL(name="source")
dest_plate = Cos_96_DW_1mL(name="dest")

lh.deck.assign_child_resource(tip_rack, rails=1)
lh.deck.assign_child_resource(source_plate, rails=10)
lh.deck.assign_child_resource(dest_plate, rails=15)

# 使用坐标分配 (x, y, z 单位毫米)
lh.deck.assign_child_resource(
    resource=tip_rack,
    location=(100, 200, 0)
)
```

### 解除资源分配

从工作台面移除资源：

```python
# 解除特定资源
lh.deck.unassign_child_resource(tip_rack)

# 访问已分配资源
all_resources = lh.deck.children
resource_names = [r.name for r in lh.deck.children]
```

## 坐标系

PyLabRobot 采用右手笛卡尔坐标系：
- **X轴**：从左到右（向右递增）
- **Y轴**：从前到后（向后递增）
- **Z轴**：从下到上（向上递增）
- **原点**：父级资源的左下前角

### 位置计算

```python
# 获取绝对位置（相对于工作台面/根节点）
absolute_loc = plate.get_absolute_location()

# 获取相对于其他资源的位置
relative_loc = well.get_location_wrt(deck)

# 获取相对于父级的位置
parent_relative = plate.location
```

## 状态管理

### 液体体积追踪

跟踪孔位和容器中的液体体积：

```python
from pylabrobot.resources import set_volume_tracking

# 全局启用体积追踪
set_volume_tracking(True)

# 设置孔位液体
plate["A1"].tracker.set_liquids([
    (None, 200)  # (液体类型, 体积_微升)
])

# 多种液体
plate["A2"].tracker.set_liquids([
    ("water", 100),
    ("ethanol", 50)
])

# 获取当前体积
volume = plate["A1"].tracker.get_volume()  # 返回总体积

# 获取液体信息
liquids = plate["A1"].tracker.get_liquids()  # 返回 (类型, 体积) 元组列表
```

### 吸头存在状态追踪

跟踪吸头盒中的吸头状态：

```python
from pylabrobot.resources import set_tip_tracking

# 全局启用吸头追踪
set_tip_tracking(True)

# 检查吸头是否存在
has_tip = tip_rack["A1"].tracker.has_tip

# 使用 pick_up_tips/drop_tips 时自动更新状态
await lh.pick_up_tips(tip_rack["A1"])  # 标记吸头为缺失
await lh.return_tips()                  # 标记吸头为存在
```

## 序列化

### 保存与加载资源

将资源定义和状态保存为 JSON：

```python
# 保存资源定义
plate.save("plate_definition.json")

# 从 JSON 加载资源
from pylabrobot.resources import Plate
plate = Plate.load_from_json_file("plate_definition.json")

# 保存工作台面布局
lh.deck.save("deck_layout.json")

# 加载工作台面布局
from pylabrobot.resources import Deck
deck = Deck.load_from_json_file("deck_layout.json")
```

### 状态序列化

独立保存和恢复资源状态：

```python
# 保存状态（吸头存在性、液体体积）
state = plate.serialize_state()
with open("plate_state.json", "w") as f:
    json.dump(state, f)

# 加载状态
with open("plate_state.json", "r") as f:
    state = json.load(f)
plate.load_state(state)

# 保存层级中所有状态
all_states = lh.deck.serialize_all_state()

# 加载所有状态
lh.deck.load_all_state(all_states)
```

## 自定义资源

### 定义自定义耗材

当内置资源不匹配设备时创建自定义耗材：

```python
from pylabrobot.resources import Plate, Well

# 定义自定义孔板
class CustomPlate(Plate):
    def __init__(self, name: str):
        super().__init__(
            name=name,
            size_x=127.76,
            size_y=85.48,
            size_z=14.5,
            num_items_x=12,  # 12列
            num_items_y=8,   # 8行
            dx=9.0,          # 孔位X间距
            dy=9.0,          # 孔位Y间距
            dz=0.0,          # 孔位Z间距（通常为0）
            item_dx=9.0,     # 孔中心X间距
            item_dy=9.0      # 孔中心Y间距
        )

# 使用自定义孔板
custom_plate = CustomPlate(name="my_custom_plate")
```

### 自定义孔位

定义特殊几何形状的孔位：

```python
from pylabrobot.resources import Well

# 创建自定义孔位
well = Well(
    name="custom_well",
    size_x=8.0,
    size_y=8.0,
    size_z=10.5,
    max_volume=200,      # µL
    bottom_shape="flat"  # 或 "v", "u"
)
```

## 资源发现

### 查找资源

在资源层级中导航：

```python
# 获取孔板所有孔位
wells = plate.children

# 按名称查找资源
resource = lh.deck.get_resource("plate_name")

# 遍历资源
for resource in lh.deck.children:
    print(f"{resource.name}: {resource.get_absolute_location()}")

# 按模式获取孔位
wells_a = [w for w in plate.children if w.name.startswith("A")]
```

### 资源元数据

访问资源信息：

```python
# 资源属性
print(f"名称: {plate.name}")
print(f"尺寸: {plate.size_x} x {plate.size_y} x {plate.size_z} mm")
print(f"位置: {plate.get_absolute_location()}")
print(f"父级: {plate.parent.name if plate.parent else None}")
print(f"子级数量: {len(plate.children)}")

# 类型检查
from pylabrobot.resources import Plate, TipRack
if isinstance(resource, Plate):
    print("这是孔板")
elif isinstance(resource, TipRack):
    print("这是吸头盒")
```

## 最佳实践

1. **唯一命名**：为所有资源使用描述性唯一名称
2. **启用追踪**：开启吸头和体积追踪确保状态准确
3. **坐标验证**：确保工作台面资源位置无重叠
4. **状态序列化**：保存工作台面布局和状态实现协议可复现
5. **资源清理**：不再需要时解除资源分配
6. **自定义资源**：内置选项不匹配时定义自定义耗材
7. **文档记录**：记录自定义资源的尺寸和属性
8. **类型检查**：操作前使用 isinstance() 验证资源类型
9. **层级导航**：通过父子关系遍历资源树
10. **JSON存储**：使用 JSON 存储工作台面布局便于版本控制和共享

## 常用模式

### 完整工作台面设置

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import STAR
from pylabrobot.resources import (
    STARLetDeck,
    TIP_CAR_480_A00,
    Cos_96_DW_1mL,
    Trough_100ml,
    set_tip_tracking,
    set_volume_tracking
)

# 启用追踪
set_tip_tracking(True)
set_volume_tracking(True)

# 初始化液体处理器
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
await lh.setup()

# 定义资源
tip_rack_1 = TIP_CAR_480_A00(name="tips_1")
tip_rack_2 = TIP_CAR_480_A00(name="tips_2")
source_plate = Cos_96_DW_1mL(name="source")
dest_plate = Cos_96_DW_1mL(name="dest")
buffer = Trough_100ml(name="buffer")

# 分配至工作台面
lh.deck.assign_child_resource(tip_rack_1, rails=1)
lh.deck.assign_child_resource(tip_rack_2, rails=2)
lh.deck.assign_child_resource(buffer, rails=5)
lh.deck.assign_child_resource(source_plate, rails=10)
lh.deck.assign_child_resource(dest_plate, rails=15)

# 设置初始体积
for well in source_plate.children:
    well.tracker.set_liquids([(None, 200)])

buffer["channel_1"].tracker.set_liquids([(None, 50000)])  # 50 mL

# 保存工作台面布局
lh.deck.save("my_protocol_deck.json")

# 保存初始状态
import json
with open("initial_state.json", "w") as f:
    json.dump(lh.deck.serialize_all_state(), f)
```

### 加载已保存的工作台面

```python
from pylabrobot.resources import Deck

# 从文件加载工作台面
deck = Deck.load_from_json_file("my_protocol_deck.json")

# 加载状态
import json
with open("initial_state.json", "r") as f:
    state = json.load(f)
deck.load_all_state(state)

# 与液体处理器配合使用
lh = LiquidHandler(backend=STAR(), deck=deck)
await lh.setup()

# 按名称访问资源
source_plate = deck.get_resource("source")
dest_plate = deck.get_resource("dest")
```

## 附加资源

- 资源文档：https://docs.pylabrobot.org/resources/introduction.html
- 自定义资源指南：https://docs.pylabrobot.org/resources/custom-resources.html
- API 参考：https://docs.pylabrobot.org/api/pylabrobot.resources.html
- 工作台面布局：https://github.com/PyLabRobot/pylabrobot/tree/main/pylabrobot/resources/deck
