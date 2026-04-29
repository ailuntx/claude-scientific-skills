# PyLabRobot 中的可视化与仿真

## 概述

PyLabRobot 提供可视化与仿真工具，用于在没有物理硬件的情况下开发、测试和验证实验室协议。可视化器提供实验台状态的实时 3D 可视化，而仿真后端支持协议测试与验证。

## 可视化器

### 什么是可视化器？

PyLabRobot 可视化器是基于浏览器的工具，具备以下功能：
- 展示实验台布局的 3D 可视化
- 实时显示吸头状态与液体体积
- 兼容仿真机器人和物理机器人
- 提供交互式实验台状态检查
- 支持可视化协议验证

### 启动可视化器

可视化器作为 Web 服务器运行并在浏览器中显示：

```python
from pylabrobot.visualizer import Visualizer

# 创建可视化器
vis = Visualizer()

# 启动 Web 服务器（自动打开浏览器）
await vis.start()

# 停止可视化器
await vis.stop()
```

**默认设置：**
- 端口：1234 (http://localhost:1234)
- 启动时自动打开浏览器

### 连接移液处理器到可视化器

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend
from pylabrobot.resources import STARLetDeck
from pylabrobot.visualizer import Visualizer

# 创建可视化器
vis = Visualizer()
await vis.start()

# 创建带仿真后端的移液处理器
lh = LiquidHandler(
    backend=ChatterboxBackend(num_channels=8),
    deck=STARLetDeck()
)

# 连接移液处理器到可视化器
lh.visualizer = vis

await lh.setup()

# 所有操作将实时可视化
await lh.pick_up_tips(tip_rack["A1:H1"])
await lh.aspirate(plate["A1:H1"], vols=100)
await lh.dispense(plate["A2:H2"], vols=100)
await lh.drop_tips()
```

### 追踪功能

#### 启用追踪

需启用追踪功能以显示吸头和液体状态：

```python
from pylabrobot.resources import set_tip_tracking, set_volume_tracking

# 全局启用（在创建资源前设置）
set_tip_tracking(True)
set_volume_tracking(True)
```

#### 设置初始液体

定义初始液体内容用于可视化：

```python
# 设置单个孔位液体
plate["A1"].tracker.set_liquids([
    (None, 200)  # (液体类型, 体积/µL)
])

# 设置单孔混合液体
plate["A2"].tracker.set_liquids([
    ("water", 100),
    ("ethanol", 50)
])

# 设置多孔液体
for well in plate["A1:H1"]:
    well.tracker.set_liquids([(None, 200)])

# 设置整板液体
for well in plate.children:
    well.tracker.set_liquids([("sample", 150)])
```

#### 可视化吸头状态

```python
# 使用 pick_up/drop 操作时自动追踪吸头
await lh.pick_up_tips(tip_rack["A1:H1"])  # 可视化器中显示吸头缺失
await lh.return_tips()                     # 可视化器中显示吸头存在
```

### 完整可视化器示例

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend
from pylabrobot.resources import (
    STARLetDeck,
    TIP_CAR_480_A00,
    Cos_96_DW_1mL,
    set_tip_tracking,
    set_volume_tracking
)
from pylabrobot.visualizer import Visualizer

# 启用追踪
set_tip_tracking(True)
set_volume_tracking(True)

# 创建可视化器
vis = Visualizer()
await vis.start()

# 创建移液处理器
lh = LiquidHandler(
    backend=ChatterboxBackend(num_channels=8),
    deck=STARLetDeck()
)
lh.visualizer = vis
await lh.setup()

# 定义资源
tip_rack = TIP_CAR_480_A00(name="tips")
source_plate = Cos_96_DW_1mL(name="source")
dest_plate = Cos_96_DW_1mL(name="dest")

# 分配到实验台
lh.deck.assign_child_resource(tip_rack, rails=1)
lh.deck.assign_child_resource(source_plate, rails=10)
lh.deck.assign_child_resource(dest_plate, rails=15)

# 设置初始体积
for well in source_plate.children:
    well.tracker.set_liquids([("sample", 200)])

# 执行带可视化的协议
await lh.pick_up_tips(tip_rack["A1:H1"])
await lh.transfer(
    source_plate["A1:H12"],
    dest_plate["A1:H12"],
    vols=100
)
await lh.drop_tips()

# 保持可视化器开启以检查最终状态
input("按 Enter 键关闭可视化器...")

# 清理
await lh.stop()
await vis.stop()
```

## 实验台布局编辑器

### 使用实验台编辑器

PyLabRobot 包含图形化实验台布局编辑器：

**功能：**
- 可视化实验台设计界面
- 拖放式资源放置
- 编辑初始液体状态
- 设置吸头状态
- 保存/加载 JSON 布局文件

**使用方法：**
- 通过可视化器界面访问
- 图形化创建布局替代代码编写
- 导出 JSON 文件用于协议

### 加载实验台布局

```python
from pylabrobot.resources import Deck

# 从 JSON 文件加载实验台
deck = Deck.load_from_json_file("my_deck_layout.json")

# 用于移液处理器
lh = LiquidHandler(backend=backend, deck=deck)
await lh.setup()

# 资源已预分配
source = deck.get_resource("source")
dest = deck.get_resource("dest")
tip_rack = deck.get_resource("tips")
```

## 仿真

### ChatterboxBackend

ChatterboxBackend 仿真移液操作：

**功能：**
- 无需物理硬件
- 验证协议逻辑
- 追踪吸头和液体体积
- 支持所有移液操作
- 兼容可视化器

**设置方法：**

```python
from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend

# 创建仿真后端
backend = ChatterboxBackend(
    num_channels=8  # 仿真 8 通道移液器
)

# 用于移液处理器
lh = LiquidHandler(backend=backend, deck=STARLetDeck())
```

### 仿真应用场景

#### 协议开发

```python
async def develop_protocol():
    """使用仿真开发协议"""

    # 使用仿真进行开发
    lh = LiquidHandler(
        backend=ChatterboxBackend(),
        deck=STARLetDeck()
    )

    # 连接可视化器
    vis = Visualizer()
    await vis.start()
    lh.visualizer = vis

    await lh.setup()

    try:
        # 开发并测试协议
        await lh.pick_up_tips(tip_rack["A1"])
        await lh.transfer(plate["A1"], plate["A2"], vols=100)
        await lh.drop_tips()

        print("协议开发完成！")

    finally:
        await lh.stop()
        await vis.stop()
```

#### 协议验证

```python
async def validate_protocol():
    """无硬件验证协议逻辑"""

    set_tip_tracking(True)
    set_volume_tracking(True)

    lh = LiquidHandler(
        backend=ChatterboxBackend(),
        deck=STARLetDeck()
    )
    await lh.setup()

    try:
        # 设置资源
        tip_rack = TIP_CAR_480_A00(name="tips")
        plate = Cos_96_DW_1mL(name="plate")

        lh.deck.assign_child_resource(tip_rack, rails=1)
        lh.deck.assign_child_resource(plate, rails=10)

        # 设置初始状态
        for well in plate.children:
            well.tracker.set_liquids([(None, 200)])

        # 执行协议
        await lh.pick_up_tips(tip_rack["A1:H1"])

        # 测试不同体积
        test_volumes = [50, 100, 150]
        for i, vol in enumerate(test_volumes):
            await lh.transfer(
                plate[f"A{i+1}:H{i+1}"],
                plate[f"A{i+4}:H{i+4}"],
                vols=vol
            )

        await lh.drop_tips()

        # 验证体积
        for i, vol in enumerate(test_volumes):
            for row in "ABCDEFGH":
                well = plate[f"{row}{i+4}"]
                actual_vol = well.tracker.get_volume()
                assert actual_vol == vol, f"{well.name} 孔体积不匹配"

        print("✓ 协议验证通过！")

    finally:
        await lh.stop()
```

#### 测试边界条件

```python
async def test_edge_cases():
    """在仿真中测试协议边界条件"""

    lh = LiquidHandler(
        backend=ChatterboxBackend(),
        deck=STARLetDeck()
    )
    await lh.setup()

    try:
        # 测试 1：空孔吸液
        try:
            await lh.aspirate(empty_plate["A1"], vols=100)
            print("✗ 本应触发空孔错误")
        except Exception as e:
            print(f"✓ 正确触发错误: {e}")

        # 测试 2：孔位溢液
        try:
            await lh.dispense(small_well, vols=1000)  # 过量
            print("✗ 本应触发溢液错误")
        except Exception as e:
            print(f"✓ 正确触发错误: {e}")

        # 测试 3：吸头容量
        try:
            await lh.aspirate(large_volume_well, vols=2000)  # 超出吸头容量
            print("✗ 本应触发吸头容量错误")
        except Exception as e:
            print(f"✓ 正确触发错误: {e}")

    finally:
        await lh.stop()
```

### CI/CD 集成

使用仿真进行自动化测试：

```python
# test_protocols.py
import pytest
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend

@pytest.mark.asyncio
async def test_transfer_protocol():
    """测试液体转移协议"""

    lh = LiquidHandler(
        backend=ChatterboxBackend(),
        deck=STARLetDeck()
    )
    await lh.setup()

    try:
        # 设置
        tip_rack = TIP_CAR_480_A00(name="tips")
        plate = Cos_96_DW_1mL(name="plate")

        lh.deck.assign_child_resource(tip_rack, rails=1)
        lh.deck.assign_child_resource(plate, rails=10)

        # 设置初始体积
        plate["A1"].tracker.set_liquids([(None, 200)])

        # 执行
        await lh.pick_up_tips(tip_rack["A1"])
        await lh.transfer(plate["A1"], plate["A2"], vols=100)
        await lh.drop_tips()

        # 断言
        assert plate["A1"].tracker.get_volume() == 100
        assert plate["A2"].tracker.get_volume() == 100

    finally:
        await lh.stop()
```

## 最佳实践

1. **优先使用仿真**：在硬件运行前通过仿真开发和测试协议
2. **启用追踪**：开启吸头和体积追踪以获得准确可视化
3. **设置初始状态**：定义初始液体体积实现真实仿真
4. **视觉检查**：使用可视化器验证实验台布局和协议执行
5. **逻辑验证**：在仿真中测试边界条件和错误场景
6. **自动化测试**：将仿真集成到 CI/CD 流程
7. **保存布局**：使用 JSON 保存和共享实验台布局
8. **状态记录**：记录初始状态确保可复现性
9. **交互式开发**：开发期间保持可视化器开启
10. **协议优化**：硬件运行前在仿真中迭代优化

## 常用模式

### 开发到生产工作流

```python
import os

# 配置
USE_HARDWARE = os.getenv("USE_HARDWARE", "false").lower() == "true"

# 创建对应后端
if USE_HARDWARE:
    from pylabrobot.liquid_handling.backends import STAR
    backend = STAR()
    print("运行于 Hamilton STAR 硬件")
else:
    from pylabrobot.liquid_handling.backends.simulation import ChatterboxBackend
    backend = ChatterboxBackend()
    print("运行于仿真模式")

# 其余协议代码完全一致
lh = LiquidHandler(backend=backend, deck=STARLetDeck())

if not USE_HARDWARE:
    # 为仿真启用可视化器
    vis = Visualizer()
    await vis.start()
    lh.visualizer = vis

await lh.setup()

# 协议执行
# ...（硬件与仿真使用相同代码）

# 运行命令: USE_HARDWARE=false python protocol.py  # 仿真模式
# 运行命令: USE_HARDWARE=true python protocol.py   # 硬件模式
```

### 可视化协议验证

```python
async def visual_verification():
    """带可视化验证暂停的协议执行"""

    vis = Visualizer()
    await vis.start()

    lh = LiquidHandler(
        backend=ChatterboxBackend(),
        deck=STARLetDeck()
    )
    lh.visualizer = vis
    await lh.setup()

    try:
        # 步骤 1
        await lh.pick_up_tips(tip_rack["A1:H1"])
        input("按 Enter 键继续...")

        # 步骤 2
        await lh.aspirate(source["A1:H1"], vols=100)
        input("按 Enter 键继续...")

        # 步骤 3
        await lh.dispense(dest["A1:H1"], vols=100)
        input("按 Enter 键继续...")

        # 步骤 4
        await lh.drop_tips()
        input("按 Enter 键结束...")

    finally:
        await lh.stop()
        await vis.stop()
```

## 故障排除

### 可视化器未更新

- 确保在执行操作前设置 `lh.visualizer = vis`
- 检查是否全局启用了追踪功能
- 确认可视化器正在运行 (`vis.start()`)
- 连接中断时刷新浏览器

### 追踪功能失效

```python
# 必须在创建资源前启用追踪
set_tip_tracking(True)
set_volume_tracking(True)

# 然后创建资源
tip_rack = TIP_CAR_480_A00(name="tips")
plate = Cos_96_DW_1mL(name="plate")
```

### 仿真错误

- 仿真会验证操作（例如不能从空孔吸液）
- 使用 try/except 处理验证错误
- 检查初始状态设置是否正确
- 确认体积未超过容量限制

## 附加资源

- 可视化器文档：https://docs.pylabrobot.org/user_guide/using-the-visualizer.html (如可用)
- 仿真指南：https://docs.pylabrobot.org/user_guide/simulation.html (如可用)
- API 参考：https://docs.pylabrobot.org/api/pylabrobot.visualizer.html
- GitHub 示例：https://github.com/PyLabRobot/pylabrobot/tree/main/examples
