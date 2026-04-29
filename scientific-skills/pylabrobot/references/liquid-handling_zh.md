# 使用 PyLabRobot 进行液体处理

## 概述

液体处理模块（`pylabrobot.liquid_handling`）提供了控制液体处理机器人的统一接口。`LiquidHandler` 类作为所有移液操作的主要接口，通过后端抽象可在不同硬件平台上工作。

## 基础设置

### 初始化液体处理器

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import STAR
from pylabrobot.resources import STARLetDeck

# 创建带STAR后端的液体处理器
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
await lh.setup()

# 操作完成后
await lh.stop()
```

### 切换后端

通过更换后端实现不同机器人的切换，无需重写协议：

```python
# Hamilton STAR
from pylabrobot.liquid_handling.backends import STAR
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())

# Opentrons OT-2
from pylabrobot.liquid_handling.backends import OpentronsBackend
lh = LiquidHandler(backend=OpentronsBackend(host="192.168.1.100"), deck=OTDeck())

# 模拟模式（无需硬件）
from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend
lh = LiquidHandler(backend=ChatterboxBackend(), deck=STARLetDeck())
```

## 核心操作

### 吸头管理

吸头的拾取与放置是液体处理的基础操作：

```python
# 从特定位置拾取吸头
await lh.pick_up_tips(tip_rack["A1"])           # 单吸头
await lh.pick_up_tips(tip_rack["A1:H1"])        # 整行8个吸头
await lh.pick_up_tips(tip_rack["A1:A12"])       # 整列12个吸头

# 放置吸头
await lh.drop_tips()                             # 当前位置放置
await lh.drop_tips(waste)                        # 指定位置放置

# 将吸头归位
await lh.return_tips()
```

**吸头追踪**：启用自动吸头追踪以监控使用情况：

```python
from pylabrobot.resources import set_tip_tracking
set_tip_tracking(True)  # 全局启用
```

### 吸取液体

从孔板或容器中吸取液体：

```python
# 基础吸取
await lh.aspirate(plate["A1"], vols=100)         # 从A1吸取100 µL

# 多孔等量吸取
await lh.aspirate(plate["A1:H1"], vols=100)      # 每孔吸取100 µL

# 多孔不等量吸取
await lh.aspirate(
    plate["A1:A3"],
    vols=[100, 150, 200]                          # 不同体积
)

# 高级参数
await lh.aspirate(
    plate["A1"],
    vols=100,
    flow_rate=50,                                 # µL/秒
    liquid_height=5,                              # 距底部高度(mm)
    blow_out_air_volume=10                        # 空气体积(µL)
)
```

### 分配液体

将液体分配到孔板或容器中：

```python
# 基础分配
await lh.dispense(plate["A2"], vols=100)         # 向A2分配100 µL

# 多孔分配
await lh.dispense(plate["A1:H1"], vols=100)      # 每孔分配100 µL

# 不等量分配
await lh.dispense(
    plate["A1:A3"],
    vols=[100, 150, 200]
)

# 高级参数
await lh.dispense(
    plate["A2"],
    vols=100,
    flow_rate=50,                                 # µL/秒
    liquid_height=2,                              # 距底部高度(mm)
    blow_out_air_volume=10                        # 空气体积(µL)
)
```

### 液体转移

单步完成吸取和分配操作：

```python
# 基础转移
await lh.transfer(
    source=source_plate["A1"],
    dest=dest_plate["A1"],
    vols=100
)

# 多孔转移（同组吸头）
await lh.transfer(
    source=source_plate["A1:H1"],
    dest=dest_plate["A1:H1"],
    vols=100
)

# 每孔不同体积
await lh.transfer(
    source=source_plate["A1:A3"],
    dest=dest_plate["B1:B3"],
    vols=[50, 100, 150]
)

# 含吸头管理
await lh.pick_up_tips(tip_rack["A1:H1"])
await lh.transfer(
    source=source_plate["A1:H12"],
    dest=dest_plate["A1:H12"],
    vols=100
)
await lh.drop_tips()
```

## 高级技术

### 连续稀释

在孔板行或列上创建连续稀释：

```python
# 2倍连续稀释
source_vols = [100, 50, 50, 50, 50, 50, 50, 50]
dest_vols = [0, 50, 50, 50, 50, 50, 50, 50]

# 先添加稀释液
await lh.pick_up_tips(tip_rack["A1"])
await lh.transfer(
    source=buffer["A1"],
    dest=plate["A2:A8"],
    vols=50
)
await lh.drop_tips()

# 执行连续稀释
await lh.pick_up_tips(tip_rack["A2"])
for i in range(7):
    await lh.aspirate(plate[f"A{i+1}"], vols=50)
    await lh.dispense(plate[f"A{i+2}"], vols=50)
    # 混合
    await lh.aspirate(plate[f"A{i+2}"], vols=50)
    await lh.dispense(plate[f"A{i+2}"], vols=50)
await lh.drop_tips()
```

### 孔板复制

将整块孔板布局复制到另一块孔板：

```python
# 准备吸头
await lh.pick_up_tips(tip_rack["A1:H1"])

# 复制96孔板（12列）
for col in range(1, 13):
    await lh.transfer(
        source=source_plate[f"A{col}:H{col}"],
        dest=dest_plate[f"A{col}:H{col}"],
        vols=100
    )

await lh.drop_tips()
```

### 多通道移液

使用多通道同时进行并行操作：

```python
# 8通道转移（整行）
await lh.pick_up_tips(tip_rack["A1:H1"])
await lh.transfer(
    source=source_plate["A1:H1"],
    dest=dest_plate["A1:H1"],
    vols=100
)
await lh.drop_tips()

# 8通道处理整板
for col in range(1, 13):
    await lh.pick_up_tips(tip_rack[f"A{col}:H{col}"])
    await lh.transfer(
        source=source_plate[f"A{col}:H{col}"],
        dest=dest_plate[f"A{col}:H{col}"],
        vols=100
    )
    await lh.drop_tips()
```

### 液体混合

通过反复吸取分配实现混合：

```python
# 通过吸取/分配混合
await lh.pick_up_tips(tip_rack["A1"])

# 混合5次
for _ in range(5):
    await lh.aspirate(plate["A1"], vols=80)
    await lh.dispense(plate["A1"], vols=80)

await lh.drop_tips()
```

## 体积追踪

自动追踪孔内液体体积：

```python
from pylabrobot.resources import set_volume_tracking

# 全局启用体积追踪
set_volume_tracking(True)

# 设置初始体积
plate["A1"].tracker.set_liquids([(None, 200)])  # 200 µL

# 吸取100 µL后
await lh.aspirate(plate["A1"], vols=100)
print(plate["A1"].tracker.get_volume())  # 100 µL

# 检查剩余体积
remaining = plate["A1"].tracker.get_volume()
```

## 液体类别

定义液体属性以实现最佳移液效果：

```python
# 液体类别控制吸取/分配参数
from pylabrobot.liquid_handling import LiquidClass

# 创建自定义液体类别
water = LiquidClass(
    name="Water",
    aspiration_flow_rate=100,
    dispense_flow_rate=150,
    aspiration_mix_flow_rate=100,
    dispense_mix_flow_rate=100,
    air_transport_retract_dist=10
)

# 在操作中使用
await lh.aspirate(
    plate["A1"],
    vols=100,
    liquid_class=water
)
```

## 错误处理

处理液体操作中的异常：

```python
try:
    await lh.setup()
    await lh.pick_up_tips(tip_rack["A1"])
    await lh.transfer(source["A1"], dest["A1"], vols=100)
    await lh.drop_tips()
except Exception as e:
    print(f"液体处理出错: {e}")
    # 尝试丢弃当前吸头
    try:
        await lh.drop_tips()
    except:
        pass
finally:
    await lh.stop()
```

## 最佳实践

1. **始终初始化和停止**：操作前调用 `await lh.setup()`，完成后调用 `await lh.stop()`
2. **启用追踪**：使用吸头和体积追踪确保状态准确性
3. **吸头管理**：吸取前拾取吸头，完成后及时丢弃
4. **流速控制**：根据液体粘度和容器类型调整流速
5. **液体高度**：设置合适的吸取/分配高度避免飞溅
6. **错误处理**：使用 try/finally 块确保资源清理
7. **模拟测试**：使用 ChatterboxBackend 在硬件运行前测试协议
8. **体积限制**：遵守吸头容量和孔板容积限制
9. **混合操作**：分配粘性液体或精度要求高时进行混合
10. **文档记录**：记录液体类别和自定义参数确保可复现性

## 常用模式

### 完整液体处理协议

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import STAR
from pylabrobot.resources import STARLetDeck, TIP_CAR_480_A00, Cos_96_DW_1mL
from pylabrobot.resources import set_tip_tracking, set_volume_tracking

# 启用追踪
set_tip_tracking(True)
set_volume_tracking(True)

# 初始化
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
await lh.setup()

try:
    # 定义资源
    tip_rack = TIP_CAR_480_A00(name="tips")
    source = Cos_96_DW_1mL(name="source")
    dest = Cos_96_DW_1mL(name="dest")

    # 分配到台面
    lh.deck.assign_child_resource(tip_rack, rails=1)
    lh.deck.assign_child_resource(source, rails=10)
    lh.deck.assign_child_resource(dest, rails=15)

    # 设置初始体积
    for well in source.children:
        well.tracker.set_liquids([(None, 200)])

    # 执行协议
    await lh.pick_up_tips(tip_rack["A1:H1"])
    await lh.transfer(
        source=source["A1:H12"],
        dest=dest["A1:H12"],
        vols=100
    )
    await lh.drop_tips()

finally:
    await lh.stop()
```

## 硬件特定说明

### Hamilton STAR

- 支持完整液体处理功能
- 通过USB连接通信
- 直接执行固件指令
- 支持CO-RE（压缩O型圈扩展）吸头

### Opentrons OT-2

- 需要IP地址进行网络连接
- 使用HTTP API通信
- 仅支持8通道和单通道移液器
- 台面布局比STAR简单

### Tecan EVO

- 支持功能开发中
- 能力与Hamilton STAR相似
- 请查阅文档确认当前兼容性

## 附加资源

- 官方液体处理指南：https://docs.pylabrobot.org/user_guide/basic.html
- API参考文档：https://docs.pylabrobot.org/api/pylabrobot.liquid_handling.html
- 协议示例：https://github.com/PyLabRobot/pylabrobot/tree/main/examples
