# PyLabRobot 中的物料处理设备

## 概述

PyLabRobot 集成了包括加热振荡器、培养箱、离心机和泵在内的物料处理设备。这些设备实现了环境控制、样品制备以及超越基础液体处理的自动化工作流程。

## 加热振荡器

### Hamilton HeaterShaker

Hamilton HeaterShaker 为微孔板提供温度控制和轨道振荡功能。

#### 设置

```python
from pylabrobot.heating_shaking import HeaterShaker
from pylabrobot.heating_shaking.hamilton import HamiltonHeaterShakerBackend

# 创建加热振荡器
hs = HeaterShaker(
    name="heater_shaker_1",
    backend=HamiltonHeaterShakerBackend(),
    size_x=156.0,
    size_y=156.0,
    size_z=18.0
)

await hs.setup()
```

#### 操作

**温度控制：**

```python
# 设置温度（摄氏度）
await hs.set_temperature(37)

# 获取当前温度
temp = await hs.get_temperature()
print(f"当前温度: {temp}°C")

# 关闭加热
await hs.set_temperature(None)
```

**振荡控制：**

```python
# 开始振荡（RPM）
await hs.set_shake_rate(300)  # 300 RPM

# 停止振荡
await hs.set_shake_rate(0)
```

**微孔板操作：**

```python
# 锁定微孔板
await hs.lock_plate()

# 解锁微孔板
await hs.unlock_plate()
```

#### 与液体处理系统集成

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import STAR
from pylabrobot.resources import STARLetDeck

# 初始化设备
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
hs = HeaterShaker(name="hs", backend=HamiltonHeaterShakerBackend())

await lh.setup()
await hs.setup()

try:
    # 将加热振荡器分配到工作台位置
    lh.deck.assign_child_resource(hs, rails=8)

    # 准备样品
    tip_rack = TIP_CAR_480_A00(name="tips")
    plate = Cos_96_DW_1mL(name="plate")

    lh.deck.assign_child_resource(tip_rack, rails=1)

    # 将微孔板放置到加热振荡器上
    hs.assign_child_resource(plate, location=(0, 0, 0))

    # 向加热振荡器上的微孔板转移试剂
    await lh.pick_up_tips(tip_rack["A1:H1"])
    await lh.transfer(reagent["A1:H1"], plate["A1:H1"], vols=100)
    await lh.drop_tips()

    # 锁定微孔板并开始孵育
    await hs.lock_plate()
    await hs.set_temperature(37)
    await hs.set_shake_rate(300)

    # 孵育
    import asyncio
    await asyncio.sleep(600)  # 10分钟

    # 停止振荡和加热
    await hs.set_shake_rate(0)
    await hs.set_temperature(None)
    await hs.unlock_plate()

finally:
    await lh.stop()
    await hs.stop()
```

#### 多加热振荡器控制

HamiltonHeaterShakerBackend 支持多单元管理：

```python
# 后端自动管理多个加热振荡器
hs1 = HeaterShaker(name="hs1", backend=HamiltonHeaterShakerBackend())
hs2 = HeaterShaker(name="hs2", backend=HamiltonHeaterShakerBackend())

await hs1.setup()
await hs2.setup()

# 分配到不同工作台位置
lh.deck.assign_child_resource(hs1, rails=8)
lh.deck.assign_child_resource(hs2, rails=12)

# 独立控制
await hs1.set_temperature(37)
await hs2.set_temperature(42)
```

### Inheco ThermoShake

Inheco ThermoShake 提供温度控制和振荡功能。

#### 设置

```python
from pylabrobot.heating_shaking import HeaterShaker
from pylabrobot.heating_shaking.inheco import InhecoThermoShakeBackend

hs = HeaterShaker(
    name="thermoshake",
    backend=InhecoThermoShakeBackend(),
    size_x=156.0,
    size_y=156.0,
    size_z=18.0
)

await hs.setup()
```

#### 操作

与 Hamilton HeaterShaker 类似：

```python
# 温度控制
await hs.set_temperature(37)
temp = await hs.get_temperature()

# 振荡控制
await hs.set_shake_rate(300)

# 微孔板锁定
await hs.lock_plate()
await hs.unlock_plate()
```

## 培养箱

### Inheco 培养箱

PyLabRobot 支持多种 Inheco 培养箱型号，用于温控微孔板存储。

#### 支持型号

- Inheco 单板培养箱
- Inheco 多板培养箱
- 其他 Inheco 温控设备

#### 设置

```python
from pylabrobot.temperature_control import TemperatureController
from pylabrobot.temperature_control.inheco import InhecoBackend

# 创建培养箱
incubator = TemperatureController(
    name="incubator",
    backend=InhecoBackend(),
    size_x=156.0,
    size_y=156.0,
    size_z=50.0
)

await incubator.setup()
```

#### 操作

```python
# 设置温度
await incubator.set_temperature(37)

# 获取温度
temp = await incubator.get_temperature()
print(f"培养箱温度: {temp}°C")

# 关闭
await incubator.set_temperature(None)
```

### Thermo Fisher Cytomat 培养箱

Cytomat 培养箱提供带温度和 CO2 控制的自动化微孔板存储。

#### 设置

```python
from pylabrobot.incubation import Incubator
from pylabrobot.incubation.cytomat_backend import CytomatBackend

incubator = Incubator(
    name="cytomat",
    backend=CytomatBackend()
)

await incubator.setup()
```

#### 操作

```python
# 存储微孔板
await incubator.store_plate(plate_id="plate_001", position=1)

# 取出微孔板
await incubator.retrieve_plate(position=1)

# 设置环境条件
await incubator.set_temperature(37)
await incubator.set_co2(5.0)  # 5% CO2
```

## 离心机

### Agilent VSpin

Agilent VSpin 是用于微孔板处理的真空辅助离心机。

#### 设置

```python
from pylabrobot.centrifuge import Centrifuge
from pylabrobot.centrifuge.vspin import VSpinBackend

centrifuge = Centrifuge(
    name="vspin",
    backend=VSpinBackend()
)

await centrifuge.setup()
```

#### 操作

**门控制：**

```python
# 开门
await centrifuge.open_door()

# 关门
await centrifuge.close_door()

# 锁门
await centrifuge.lock_door()

# 解锁门
await centrifuge.unlock_door()
```

**吊篮定位：**

```python
# 移动吊篮至装载位置
await centrifuge.move_bucket_to_loading()

# 移动吊篮至初始位置
await centrifuge.move_bucket_to_home()
```

**离心：**

```python
# 运行离心机
await centrifuge.spin(
    speed=2000,      # RPM
    duration=300     # 秒
)

# 停止离心
await centrifuge.stop_spin()
```

#### 集成示例

```python
async def centrifuge_workflow():
    """完整离心工作流程"""

    lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
    centrifuge = Centrifuge(name="vspin", backend=VSpinBackend())

    await lh.setup()
    await centrifuge.setup()

    try:
        # 准备样品
        await lh.pick_up_tips(tip_rack["A1:H1"])
        await lh.transfer(samples["A1:H12"], plate["A1:H12"], vols=200)
        await lh.drop_tips()

        # 装载至离心机
        print("将微孔板移至离心机")
        await centrifuge.open_door()
        await centrifuge.move_bucket_to_loading()
        input("微孔板装载完成后按回车...")

        await centrifuge.move_bucket_to_home()
        await centrifuge.close_door()
        await centrifuge.lock_door()

        # 离心
        await centrifuge.spin(speed=2000, duration=300)

        # 卸载
        await centrifuge.unlock_door()
        await centrifuge.open_door()
        await centrifuge.move_bucket_to_loading()
        input("微孔板移除后按回车...")

        await centrifuge.move_bucket_to_home()
        await centrifuge.close_door()

    finally:
        await lh.stop()
        await centrifuge.stop()
```

## 泵

### Cole Parmer Masterflex

PyLabRobot 支持 Cole Parmer Masterflex 蠕动泵进行流体传输。

#### 设置

```python
from pylabrobot.pumps import Pump
from pylabrobot.pumps.cole_parmer import ColeParmerMasterflexBackend

pump = Pump(
    name="masterflex",
    backend=ColeParmerMasterflexBackend()
)

await pump.setup()
```

#### 操作

**运行泵：**

```python
# 按持续时间运行
await pump.run_for_duration(
    duration=10,      # 秒
    speed=50          # 最大速度百分比
)

# 持续运行
await pump.start(speed=50)

# 停止泵
await pump.stop()
```

**体积定量传输：**

```python
# 泵送指定体积（需校准）
await pump.pump_volume(
    volume=10,        # mL
    speed=50          # 最大速度百分比
)
```

#### 校准

```python
# 校准泵体积精度
# (需已知体积测量值)
await pump.run_for_duration(duration=60, speed=50)
actual_volume = 25.3  # 实测 mL

pump.calibrate(duration=60, speed=50, volume=actual_volume)
```

### Agrowtek 泵阵列

支持 Agrowtek 泵阵列实现多通道同步流体传输。

#### 设置

```python
from pylabrobot.pumps import PumpArray
from pylabrobot.pumps.agrowtek import AgrowtekBackend

pump_array = PumpArray(
    name="agrowtek",
    backend=AgrowtekBackend(),
    num_pumps=8
)

await pump_array.setup()
```

#### 操作

```python
# 运行指定泵
await pump_array.run_pump(
    pump_number=1,
    duration=10,
    speed=50
)

# 同时运行多个泵
await pump_array.run_pumps(
    pump_numbers=[1, 2, 3],
    duration=10,
    speed=50
)
```

## 多设备协议

### 复杂工作流程示例

```python
async def complex_workflow():
    """多设备自动化工作流程"""

    # 初始化所有设备
    lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
    hs = HeaterShaker(name="hs", backend=HamiltonHeaterShakerBackend())
    centrifuge = Centrifuge(name="vspin", backend=VSpinBackend())
    pump = Pump(name="pump", backend=ColeParmerMasterflexBackend())

    await lh.setup()
    await hs.setup()
    await centrifuge.setup()
    await pump.setup()

    try:
        # 1. 样品制备
        await lh.pick_up_tips(tip_rack["A1:H1"])
        await lh.transfer(samples["A1:H12"], plate["A1:H12"], vols=100)
        await lh.drop_tips()

        # 2. 通过泵添加试剂
        await pump.pump_volume(volume=50, speed=50)

        # 3. 在加热振荡器上混合
        await hs.lock_plate()
        await hs.set_temperature(37)
        await hs.set_shake_rate(300)
        await asyncio.sleep(600)  # 10分钟孵育
        await hs.set_shake_rate(0)
        await hs.set_temperature(None)
        await hs.unlock_plate()

        # 4. 离心
        await centrifuge.open_door()
        # (装载微孔板)
        await centrifuge.close_door()
        await centrifuge.spin(speed=2000, duration=180)
        await centrifuge.open_door()
        # (卸载微孔板)

        # 5. 转移上清液
        await lh.pick_up_tips(tip_rack["A2:H2"])
        await lh.transfer(
            plate["A1:H12"],
            output_plate["A1:H12"],
            vols=80
        )
        await lh.drop_tips()

    finally:
        await lh.stop()
        await hs.stop()
        await centrifuge.stop()
        await pump.stop()
```

## 最佳实践

1. **设备初始化**：在协议开始时设置所有设备
2. **顺序操作**：物料处理通常需要顺序步骤
3. **安全性**：手动操作前始终解锁/打开设备门
4. **温度平衡**：预留设备达到设定温度的时间
5. **错误处理**：使用 try/finally 优雅处理设备错误
6. **状态验证**：操作前检查设备状态
7. **时间控制**：考虑设备特有延迟（加热、离心）
8. **维护保养**：遵循制造商维护计划
9. **校准**：定期校准泵和温控设备
10. **文档记录**：记录所有设备设置和参数

## 常用模式

### 温控孵育

```python
async def incubate_with_shaking(
    plate,
    temperature: float,
    shake_rate: int,
    duration: int
):
    """带温度和振荡的微孔板孵育"""

    hs = HeaterShaker(name="hs", backend=HamiltonHeaterShakerBackend())
    await hs.setup()

    try:
        # 将微孔板分配到加热振荡器
        hs.assign_child_resource(plate, location=(0, 0, 0))

        # 开始孵育
        await hs.lock_plate()
        await hs.set_temperature(temperature)
        await hs.set_shake_rate(shake_rate)

        # 等待
        await asyncio.sleep(duration)

        # 停止
        await hs.set_shake_rate(0)
        await hs.set_temperature(None)
        await hs.unlock_plate()

    finally:
        await hs.stop()

# 在协议中使用
await incubate_with_shaking(
    plate=assay_plate,
    temperature=37,
    shake_rate=300,
    duration=600  # 10分钟
)
```

### 自动化微孔板处理

```python
async def process_plates(plate_list: list):
    """多微孔板工作流程处理"""

    lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
    hs = HeaterShaker(name="hs", backend=HamiltonHeaterShakerBackend())

    await lh.setup()
    await hs.setup()

    try:
        for i, plate in enumerate(plate_list):
            print(f"处理微孔板 {i+1}/{len(plate_list)}")

            # 转移样品
            await lh.pick_up_tips(tip_rack[f"A{i+1}:H{i+1}"])
            await lh.transfer(
                source[f"A{i+1}:H{i+1}"],
                plate["A1:H1"],
                vols=100
            )
            await lh.drop_tips()

            # 孵育
            hs.assign_child_resource(plate, location=(0, 0, 0))
            await hs.lock_plate()
            await hs.set_temperature(37)
            await hs.set_shake_rate(300)
            await asyncio.sleep(300)  # 5分钟
            await hs.set_shake_rate(0)
            await hs.set_temperature(None)
            await hs.unlock_plate()
            hs.unassign_child_resource(plate)

    finally:
        await lh.stop()
        await hs.stop()
```

## 附加资源

- 物料处理文档：https://docs.pylabrobot.org/user_guide/01_material-handling/
- 加热振荡器：https://docs.pylabrobot.org/user_guide/01_material-handling/heating_shaking/
- API 参考：https://docs.pylabrobot.org/api/
- 支持设备列表：https://docs.pylabrobot.org/user_guide/machines.html
