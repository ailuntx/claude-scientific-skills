# PyLabRobot 中的硬件后端

## 概述

PyLabRobot 采用后端抽象系统，使得相同的协议代码可在不同液体处理机器人和平台上运行。后端处理设备特定的通信，而 `LiquidHandler` 前端提供统一接口。

## 后端架构

### 后端工作原理

1. **前端**：`LiquidHandler` 类提供高级 API
2. **后端**：设备特定类处理硬件通信
3. **协议**：相同代码适用于不同后端

```python
# 相同的协议代码
await lh.pick_up_tips(tip_rack["A1"])
await lh.aspirate(plate["A1"], vols=100)
await lh.dispense(plate["A2"], vols=100)
await lh.drop_tips()

# 适用于任何后端（STAR、Opentrons、模拟环境等）
```

### 后端接口

所有后端继承自 `LiquidHandlerBackend` 并实现：
- `setup()`：初始化硬件连接
- `stop()`：关闭连接并清理
- 设备特定命令方法（aspirate、dispense 等）

## 支持的后端

### Hamilton STAR（完整支持）

Hamilton STAR 和 STARlet 液体处理机器人拥有完整的 PyLabRobot 支持。

**设置：**

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import STAR
from pylabrobot.resources import STARLetDeck

# 创建 STAR 后端
backend = STAR()

# 初始化液体处理器
lh = LiquidHandler(backend=backend, deck=STARLetDeck())
await lh.setup()
```

**平台支持：**
- Windows ✅
- macOS ✅
- Linux ✅
- Raspberry Pi ✅

**通信方式：**
- 通过 USB 连接机器人
- 直接发送固件指令
- 无需 Hamilton 软件

**功能特性：**
- 完整的液体处理操作
- CO-RE 吸头支持
- 96通道移液头支持（若设备配备）
- 温度控制
- 载架和轨道定位系统

**工作台类型：**
```python
from pylabrobot.resources import STARLetDeck, STARDeck

# STARlet（小型工作台）
deck = STARLetDeck()

# STAR（完整工作台）
deck = STARDeck()
```

**示例：**

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import STAR
from pylabrobot.resources import STARLetDeck, TIP_CAR_480_A00, Cos_96_DW_1mL

# 初始化
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
await lh.setup()

# 定义资源
tip_rack = TIP_CAR_480_A00(name="tips")
plate = Cos_96_DW_1mL(name="plate")

# 分配到轨道
lh.deck.assign_child_resource(tip_rack, rails=1)
lh.deck.assign_child_resource(plate, rails=10)

# 执行协议
await lh.pick_up_tips(tip_rack["A1"])
await lh.transfer(plate["A1"], plate["A2"], vols=100)
await lh.drop_tips()

await lh.stop()
```

### Opentrons OT-2（已支持）

通过 Opentrons HTTP API 支持 OT-2 机器人。

**设置：**

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import OpentronsBackend
from pylabrobot.resources import OTDeck

# 创建 Opentrons 后端（需机器人 IP 地址）
backend = OpentronsBackend(host="192.168.1.100")  # 替换为实际 IP

# 初始化液体处理器
lh = LiquidHandler(backend=backend, deck=OTDeck())
await lh.setup()
```

**平台支持：**
- 任何可访问 OT-2 网络的平台

**通信方式：**
- 基于网络的 HTTP API
- 需要机器人 IP 地址
- 无需 Opentrons 应用

**功能特性：**
- 8通道移液器支持
- 单通道移液器支持
- 标准 OT-2 工作台布局
- 基于坐标的定位系统

**限制：**
- 使用旧版 Opentrons HTTP API
- 部分功能相比 STAR 可能受限

**示例：**

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import OpentronsBackend
from pylabrobot.resources import OTDeck

# 通过 IP 初始化
lh = LiquidHandler(
    backend=OpentronsBackend(host="192.168.1.100"),
    deck=OTDeck()
)
await lh.setup()

# 加载工作台布局
lh.deck = Deck.load_from_json_file("opentrons_layout.json")

# 执行协议
await lh.pick_up_tips(tip_rack["A1"])
await lh.transfer(plate["A1"], plate["A2"], vols=100)
await lh.drop_tips()

await lh.stop()
```

### Tecan EVO（开发中）

Tecan EVO 液体处理机器人的支持正在开发中。

**当前状态：**
- 开发进行中
- 基础命令可能可用
- 请查阅文档获取最新功能支持

**设置（可用时）：**

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import TecanBackend
from pylabrobot.resources import TecanDeck

backend = TecanBackend()
lh = LiquidHandler(backend=backend, deck=TecanDeck())
```

### Hamilton Vantage（基本支持）

Hamilton Vantage 拥有"基本完整"的支持。

**设置：**

```python
from pylabrobot.liquid_handling.backends import Vantage
from pylabrobot.resources import VantageDeck

lh = LiquidHandler(backend=Vantage(), deck=VantageDeck())
```

**功能特性：**
- 类似 STAR 的支持
- 部分高级功能可能受限

## 模拟后端

### ChatterboxBackend（模拟环境）

无需物理硬件即可测试协议。

**设置：**

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend
from pylabrobot.resources import STARLetDeck

# 创建模拟后端
backend = ChatterboxBackend(num_channels=8)

# 初始化液体处理器
lh = LiquidHandler(backend=backend, deck=STARLetDeck())
await lh.setup()
```

**功能特性：**
- 无需硬件设备
- 模拟所有液体处理操作
- 支持可视化实时反馈
- 验证协议逻辑
- 追踪吸头和液体体积

**使用场景：**
- 协议开发与测试
- 培训教学
- CI/CD 流水线测试
- 无硬件环境调试

**示例：**

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend
from pylabrobot.resources import STARLetDeck, TIP_CAR_480_A00, Cos_96_DW_1mL
from pylabrobot.resources import set_tip_tracking, set_volume_tracking

# 启用模拟追踪
set_tip_tracking(True)
set_volume_tracking(True)

# 使用模拟后端初始化
lh = LiquidHandler(
    backend=ChatterboxBackend(num_channels=8),
    deck=STARLetDeck()
)
await lh.setup()

# 定义资源
tip_rack = TIP_CAR_480_A00(name="tips")
plate = Cos_96_DW_1mL(name="plate")

lh.deck.assign_child_resource(tip_rack, rails=1)
lh.deck.assign_child_resource(plate, rails=10)

# 设置初始体积
for well in plate.children:
    well.tracker.set_liquids([(None, 200)])

# 运行模拟协议
await lh.pick_up_tips(tip_rack["A1:H1"])
await lh.transfer(plate["A1:H1"], plate["A2:H2"], vols=100)
await lh.drop_tips()

# 查看结果
print(f"A1 体积: {plate['A1'].tracker.get_volume()} µL")  # 100 µL
print(f"A2 体积: {plate['A2'].tracker.get_volume()} µL")  # 100 µL

await lh.stop()
```

## 后端切换

### 后端无关协议

编写适用于任何后端的协议：

```python
def get_backend(robot_type: str):
    """工厂函数创建对应后端"""
    if robot_type == "star":
        from pylabrobot.liquid_handling.backends import STAR
        return STAR()
    elif robot_type == "opentrons":
        from pylabrobot.liquid_handling.backends import OpentronsBackend
        return OpentronsBackend(host="192.168.1.100")
    elif robot_type == "simulation":
        from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend
        return ChatterboxBackend()
    else:
        raise ValueError(f"未知机器人类型: {robot_type}")

def get_deck(robot_type: str):
    """工厂函数创建对应工作台"""
    if robot_type == "star":
        from pylabrobot.resources import STARLetDeck
        return STARLetDeck()
    elif robot_type == "opentrons":
        from pylabrobot.resources import OTDeck
        return OTDeck()
    elif robot_type == "simulation":
        from pylabrobot.resources import STARLetDeck
        return STARLetDeck()
    else:
        raise ValueError(f"未知机器人类型: {robot_type}")

# 在协议中使用
robot_type = "simulation"  # 按需改为 "star" 或 "opentrons"
backend = get_backend(robot_type)
deck = get_deck(robot_type)

lh = LiquidHandler(backend=backend, deck=deck)
await lh.setup()

# 协议代码适用于任何后端
await lh.pick_up_tips(tip_rack["A1"])
await lh.transfer(plate["A1"], plate["A2"], vols=100)
await lh.drop_tips()
```

### 开发工作流

1. **开发**：使用 ChatterboxBackend 编写协议
2. **测试**：通过可视化验证逻辑
3. **验证**：在模拟环境中测试真实工作台布局
4. **部署**：切换到硬件后端（STAR、Opentrons）

```python
# 开发阶段
lh = LiquidHandler(backend=ChatterboxBackend(), deck=STARLetDeck())

# ... 开发协议 ...

# 生产环境（仅需更换后端）
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
```

## 后端配置

### 自定义后端参数

部分后端支持配置参数：

```python
# Opentrons 自定义参数
backend = OpentronsBackend(
    host="192.168.1.100",
    port=31950  # 默认 Opentrons API 端口
)

# ChatterboxBackend 自定义通道数
backend = ChatterboxBackend(
    num_channels=8  # 8通道模拟
)
```

### 连接故障排除

**Hamilton STAR：**
- 确保 USB 线已连接
- 检查无其他软件占用设备
- 验证固件版本为最新
- macOS/Linux 系统可能需要 USB 权限

**Opentrons OT-2：**
- 确认机器人 IP 地址正确
- 检查网络连通性（ping 测试）
- 确保机器人电源开启
- 确认 Opentrons 应用未阻塞 API 访问

**通用建议：**
- 使用 `await lh.setup()` 测试连接
- 根据错误信息排查具体问题
- 确保设备访问权限正确

## 后端特定功能

### Hamilton STAR 专属

```python
# 直接访问后端获取硬件特定功能
star_backend = lh.backend

# Hamilton 专属命令（如需要）
# 多数操作应通过 LiquidHandler 接口
```

### Opentrons 专属

```python
# Opentrons 特定配置
ot_backend = lh.backend

# 直接访问 OT-2 API（高级用法）
# 多数操作应通过 LiquidHandler 接口
```

## 最佳实践

1. **硬件抽象**：尽可能编写后端无关协议
2. **模拟测试**：始终先用 ChatterboxBackend 测试
3. **工厂模式**：使用工厂函数创建后端
4. **错误处理**：优雅处理连接错误
5. **文档说明**：记录协议支持的后端类型
6. **配置管理**：使用配置文件管理后端参数
7. **版本控制**：跟踪后端版本与兼容性
8. **资源清理**：始终调用 `await lh.stop()` 释放硬件
9. **单一连接**：确保单程序独占硬件连接
10. **平台测试**：部署前在目标平台测试

## 常用模式

### 多后端支持

```python
import asyncio
from typing import Literal

async def run_protocol(
    robot_type: Literal["star", "opentrons", "simulation"],
    visualize: bool = False
):
    """在指定后端运行协议"""

    # 创建后端
    if robot_type == "star":
        from pylabrobot.liquid_handling.backends import STAR
        backend = STAR()
        deck = STARLetDeck()
    elif robot_type == "opentrons":
        from pylabrobot.liquid_handling.backends import OpentronsBackend
        backend = OpentronsBackend(host="192.168.1.100")
        deck = OTDeck()
    elif robot_type == "simulation":
        from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend
        backend = ChatterboxBackend()
        deck = STARLetDeck()

    # 初始化
    lh = LiquidHandler(backend=backend, deck=deck)
    await lh.setup()

    try:
        # 加载工作台布局（后端无关）
        # lh.deck = Deck.load_from_json_file(f"{robot_type}_layout.json")

        # 执行协议（后端无关）
        await lh.pick_up_tips(tip_rack["A1"])
        await lh.transfer(plate["A1"], plate["A2"], vols=100)
        await lh.drop_tips()

        print("协议执行成功！")

    finally:
        await lh.stop()

# 在不同后端运行
await run_protocol("simulation")      # 模拟环境测试
await run_protocol("star")            # Hamilton STAR 运行
await run_protocol("opentrons")       # Opentrons OT-2 运行
```

## 附加资源

- 后端文档：https://docs.pylabrobot.org/user_guide/backends.html
- 支持设备：https://docs.pylabrobot.org/user_guide/machines.html
- API 参考：https://docs.pylabrobot.org/api/pylabrobot.liquid_handling.backends.html
- GitHub 示例：https://github.com/PyLabRobot/pylabrobot/tree/main/examples
