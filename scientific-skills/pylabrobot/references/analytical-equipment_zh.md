# PyLabRobot 中的分析设备

## 概述

PyLabRobot 集成了包括酶标仪、天平和其它测量设备在内的分析设备。这使得自动化工作流程能够将液体处理与分析测量相结合。

## 酶标仪

### BMG CLARIOstar (Plus)

BMG Labtech CLARIOstar 和 CLARIOstar Plus 是用于测量吸光度、发光和荧光的微孔板读数仪。

#### 硬件设置

**物理连接：**
1. IEC C13 电源线连接主电源
2. USB-B 线连接计算机（设备端带安全螺丝）
3. 可选：RS-232 端口用于板堆叠单元

**通信：**
- 通过 FTDI/USB-A 进行固件级串行连接
- 跨平台支持（Windows, macOS, Linux）

#### 软件设置

```python
from pylabrobot.plate_reading import PlateReader
from pylabrobot.plate_reading.clario_star_backend import CLARIOstarBackend

# 创建后端
backend = CLARIOstarBackend()

# 初始化酶标仪
pr = PlateReader(
    name="CLARIOstar",
    backend=backend,
    size_x=0.0,    # 物理尺寸对酶标仪不关键
    size_y=0.0,
    size_z=0.0
)

# 设置（初始化设备）
await pr.setup()

# 使用完毕时
await pr.stop()
```

#### 基本操作

**开启与关闭：**

```python
# 打开载板托盘
await pr.open()

# (手动或通过机器人加载板)

# 关闭载板托盘
await pr.close()
```

**温度控制：**

```python
# 设置温度（摄氏度）
await pr.set_temperature(37)

# 注意：达到设定温度较慢
# 建议在实验流程早期设置温度
```

**读取测量值：**

```python
# 吸光度读取
data = await pr.read_absorbance(wavelength=450)  # 单位 nm

# 发光读取
data = await pr.read_luminescence()

# 荧光读取
data = await pr.read_fluorescence(
    excitation_wavelength=485,  # 单位 nm
    emission_wavelength=535     # 单位 nm
)
```

#### 数据格式

酶标仪方法返回数组数据：

```python
import numpy as np

# 读取吸光度
data = await pr.read_absorbance(wavelength=450)

# data 通常是二维数组（96孔板为8x12）
print(f"数据形状: {data.shape}")
print(f"A1孔: {data[0][0]}")
print(f"H12孔: {data[7][11]}")

# 转换为DataFrame便于处理
import pandas as pd
df = pd.DataFrame(data)
```

#### 与液体处理系统集成

结合酶标仪与液体处理：

```python
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import STAR
from pylabrobot.resources import STARLetDeck
from pylabrobot.plate_reading import PlateReader
from pylabrobot.plate_reading.clario_star_backend import CLARIOstarBackend

# 初始化液体处理器
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
await lh.setup()

# 初始化酶标仪
pr = PlateReader(name="CLARIOstar", backend=CLARIOstarBackend())
await pr.setup()

# 提前设置温度
await pr.set_temperature(37)

try:
    # 使用液体处理器制备样品
    tip_rack = TIP_CAR_480_A00(name="tips")
    reagent_plate = Cos_96_DW_1mL(name="reagents")
    assay_plate = Cos_96_DW_1mL(name="assay")

    lh.deck.assign_child_resource(tip_rack, rails=1)
    lh.deck.assign_child_resource(reagent_plate, rails=10)
    lh.deck.assign_child_resource(assay_plate, rails=15)

    # 转移样品
    await lh.pick_up_tips(tip_rack["A1:H1"])
    await lh.transfer(
        reagent_plate["A1:H12"],
        assay_plate["A1:H12"],
        vols=100
    )
    await lh.drop_tips()

    # 将板移至读数仪（手动或机械臂）
    print("将检测板移至酶标仪")
    input("板加载完成后按回车...")

    # 读板
    await pr.open()
    # (此处加载板)
    await pr.close()

    data = await pr.read_absorbance(wavelength=450)
    print(f"吸光度数据: {data}")

finally:
    await lh.stop()
    await pr.stop()
```

#### 高级功能

**开发状态：**

部分 CLARIOstar 功能正在开发中：
- 光谱扫描
- 注射器针头控制
- 详细测量参数配置
- 孔位特异性读取模式

请查阅最新文档获取功能支持状态。

#### 最佳实践

1. **温度控制**：提前设置温度，升温过程缓慢
2. **板加载**：确保板正确放置后再关闭
3. **测量选择**：为实验选择合适波长
4. **数据验证**：检查测量质量和预期范围
5. **错误处理**：处理超时和通信错误
6. **维护**：按制造商指南保持光学元件清洁

#### 示例：完整读板工作流程

```python
async def run_plate_reading_assay():
    """包含样品制备和读数的完整工作流程"""

    # 初始化设备
    lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
    pr = PlateReader(name="CLARIOstar", backend=CLARIOstarBackend())

    await lh.setup()
    await pr.setup()

    # 设置酶标仪温度
    await pr.set_temperature(37)

    try:
        # 定义资源
        tip_rack = TIP_CAR_480_A00(name="tips")
        samples = Cos_96_DW_1mL(name="samples")
        assay_plate = Cos_96_DW_1mL(name="assay")
        substrate = Trough_100ml(name="substrate")

        lh.deck.assign_child_resource(tip_rack, rails=1)
        lh.deck.assign_child_resource(substrate, rails=5)
        lh.deck.assign_child_resource(samples, rails=10)
        lh.deck.assign_child_resource(assay_plate, rails=15)

        # 转移样品
        await lh.pick_up_tips(tip_rack["A1:H1"])
        await lh.transfer(
            samples["A1:H12"],
            assay_plate["A1:H12"],
            vols=50
        )
        await lh.drop_tips()

        # 添加底物
        await lh.pick_up_tips(tip_rack["A2:H2"])
        for col in range(1, 13):
            await lh.transfer(
                substrate["channel_1"],
                assay_plate[f"A{col}:H{col}"],
                vols=50
            )
        await lh.drop_tips()

        # 孵育（如需要）
        # await asyncio.sleep(300)  # 5分钟

        # 移至酶标仪
        print("将检测板转移至CLARIOstar")
        input("准备就绪后按回车...")

        await pr.open()
        input("板加载完成后按回车...")
        await pr.close()

        # 读取吸光度
        data = await pr.read_absorbance(wavelength=450)

        # 处理结果
        import pandas as pd
        df = pd.DataFrame(
            data,
            index=[f"{r}" for r in "ABCDEFGH"],
            columns=[f"{c}" for c in range(1, 13)]
        )

        print("吸光度结果:")
        print(df)

        # 保存结果
        df.to_csv("plate_reading_results.csv")

        return df

    finally:
        await lh.stop()
        await pr.stop()

# 运行检测
results = await run_plate_reading_assay()
```

## 天平

### 梅特勒托利多天平

PyLabRobot 支持梅特勒托利多天平进行质量测量。

#### 设置

```python
from pylabrobot.scales import Scale
from pylabrobot.scales.mettler_toledo_backend import MettlerToledoBackend

# 创建天平
scale = Scale(
    name="analytical_scale",
    backend=MettlerToledoBackend()
)

await scale.setup()
```

#### 操作

```python
# 获取重量测量值
weight = await scale.get_weight()  # 返回克为单位的重量
print(f"重量: {weight} g")

# 天平去皮（归零）
await scale.tare()

# 获取多次测量
weights = []
for i in range(5):
    w = await scale.get_weight()
    weights.append(w)
    await asyncio.sleep(1)

average_weight = sum(weights) / len(weights)
print(f"平均重量: {average_weight} g")
```

#### 与液体处理系统集成

```python
# 在流程中称量样品
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
scale = Scale(name="scale", backend=MettlerToledoBackend())

await lh.setup()
await scale.setup()

try:
    # 天平去皮
    await scale.tare()

    # 分配液体
    await lh.pick_up_tips(tip_rack["A1"])
    await lh.aspirate(reagent["A1"], vols=1000)

    # (移至天平位置)

    # 分配并称重
    await lh.dispense(container, vols=1000)
    weight = await scale.get_weight()

    print(f"分配重量: {weight} g")

    # 计算实际体积（假设水密度=1 g/mL）
    actual_volume = weight * 1000  # 克转微升
    print(f"实际体积: {actual_volume} µL")

    await lh.drop_tips()

finally:
    await lh.stop()
    await scale.stop()
```

## 其它分析设备

### 流式细胞仪

部分流式细胞仪集成正在开发中。请查阅最新文档获取支持状态。

### 分光光度计

可能支持更多分光光度计型号。请查阅文档获取当前设备兼容性信息。

## 多设备工作流程

### 协调多设备

```python
async def multi_device_workflow():
    """协调液体处理器、酶标仪和天平"""

    # 初始化所有设备
    lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
    pr = PlateReader(name="CLARIOstar", backend=CLARIOstarBackend())
    scale = Scale(name="scale", backend=MettlerToledoBackend())

    await lh.setup()
    await pr.setup()
    await scale.setup()

    try:
        # 1. 称量试剂
        await scale.tare()
        # (将容器放置在天平上)
        reagent_weight = await scale.get_weight()

        # 2. 用液体处理器制备样品
        await lh.pick_up_tips(tip_rack["A1:H1"])
        await lh.transfer(source["A1:H12"], dest["A1:H12"], vols=100)
        await lh.drop_tips()

        # 3. 读板
        await pr.open()
        # (加载板)
        await pr.close()
        data = await pr.read_absorbance(wavelength=450)

        return {
            "reagent_weight": reagent_weight,
            "absorbance_data": data
        }

    finally:
        await lh.stop()
        await pr.stop()
        await scale.stop()
```

## 最佳实践

1. **设备初始化**：在流程开始时设置所有设备
2. **错误处理**：优雅处理通信错误
3. **清理**：始终对所有设备调用 `stop()`
4. **时序**：考虑设备特定时间（温度平衡、测量时间）
5. **校准**：遵循制造商校准流程
6. **数据验证**：验证测量值在预期范围内
7. **文档记录**：记录设备设置和参数
8. **集成测试**：全面测试多设备工作流程
9. **并发操作**：尽可能使用异步实现操作重叠
10. **数据存储**：保存带元数据的原始数据（时间戳、设置）

## 常用模式

### 动力学读板

```python
async def kinetic_reading(num_reads: int, interval: int):
    """执行动力学读板"""

    pr = PlateReader(name="CLARIOstar", backend=CLARIOstarBackend())
    await pr.setup()

    try:
        await pr.set_temperature(37)
        await pr.open()
        # (加载板)
        await pr.close()

        results = []
        for i in range(num_reads):
            data = await pr.read_absorbance(wavelength=450)
            timestamp = time.time()
            results.append({
                "read_number": i + 1,
                "timestamp": timestamp,
                "data": data
            })

            if i < num_reads - 1:
                await asyncio.sleep(interval)

        return results

    finally:
        await pr.stop()

# 每30秒读取一次，持续10分钟
results = await kinetic_reading(num_reads=20, interval=30)
```

## 附加资源

- 读板文档：https://docs.pylabrobot.org/user_guide/02_analytical/
- BMG CLARIOstar 指南：https://docs.pylabrobot.org/user_guide/02_analytical/plate-reading/bmg-clariostar.html
- API 参考：https://docs.pylabrobot.org/api/pylabrobot.plate_reading.html
- 支持设备列表：https://docs.pylabrobot.org/user_guide/machines.html
