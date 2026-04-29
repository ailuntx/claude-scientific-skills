```markdown
---
name: opentrons-integration
description: 适用于OT-2和Flex机器人的官方Opentrons协议API。编写专为Opentrons硬件设计的协议时使用，可完全访问Protocol API v2功能。最适合生产级Opentrons协议，确保官方API兼容性。如需多供应商自动化或更广泛的设备控制，请使用pylabrobot。
license: Unknown
metadata:
    skill-author: K-Dense Inc.
---

# Opentrons 集成

## 概述

Opentrons是基于Python的实验室自动化平台，支持Flex和OT-2机器人。通过Protocol API v2编写协议，实现液体处理、控制硬件模块（加热振荡器、热循环仪）、管理耗材库，构建自动化移液工作流。

## 适用场景

本技能适用于：
- 使用Python编写Opentrons Protocol API v2协议
- 在Flex或OT-2机器人上自动化液体处理流程
- 控制硬件模块（温控、磁力、加热振荡器、热循环仪）
- 配置耗材库和台面布局
- 实现复杂移液操作（梯度稀释、板复制、PCR体系构建）
- 管理吸头使用并优化协议效率
- 使用多通道移液器进行96孔板操作
- 在机器人执行前模拟测试协议

## 核心功能

### 1. 协议结构与元数据

所有Opentrons协议均遵循标准结构：

```python
from opentrons import protocol_api

# 元数据
metadata = {
    'protocolName': '我的协议',
    'author': '姓名 <email@example.com>',
    'description': '协议描述',
    'apiLevel': '2.19'  # 使用最新可用API版本
}

# 要求（可选）
requirements = {
    'robotType': 'Flex',  # 或 'OT-2'
    'apiLevel': '2.19'
}

# 运行函数
def run(protocol: protocol_api.ProtocolContext):
    # 协议指令写在此处
    pass
```

**关键元素：**
- 从`opentrons`导入`protocol_api`
- 定义包含protocolName/author/description/apiLevel的`metadata`字典
- 可选的`requirements`字典指定机器人类型和API版本
- 实现接收`ProtocolContext`参数的`run()`函数
- 所有协议逻辑置于`run()`函数内

### 2. 加载硬件

**加载仪器（移液器）：**

```python
def run(protocol: protocol_api.ProtocolContext):
    # 在指定挂载点加载移液器
    left_pipette = protocol.load_instrument(
        'p1000_single_flex',  # 仪器名称
        'left',               # 挂载点：'left' 或 'right'
        tip_racks=[tip_rack]  # 吸头架耗材对象列表
    )
```

常用移液器名称：
- Flex: `p50_single_flex`, `p1000_single_flex`, `p50_multi_flex`, `p1000_multi_flex`
- OT-2: `p20_single_gen2`, `p300_single_gen2`, `p1000_single_gen2`, `p20_multi_gen2`, `p300_multi_gen2`

**加载耗材库：**

```python
# 直接在台面加载耗材库
plate = protocol.load_labware(
    'corning_96_wellplate_360ul_flat',  # 耗材库API名称
    'D1',                                # 台面卡槽（Flex: A1-D3, OT-2: 1-11）
    label='样品板'                 # 可选显示标签
)

# 加载吸头架
tip_rack = protocol.load_labware('opentrons_flex_96_tiprack_1000ul', 'C1')

# 通过适配器加载耗材库
adapter = protocol.load_adapter('opentrons_flex_96_tiprack_adapter', 'B1')
tips = adapter.load_labware('opentrons_flex_96_tiprack_200ul')
```

**加载模块：**

```python
# 温控模块
temp_module = protocol.load_module('temperature module gen2', 'D3')
temp_plate = temp_module.load_labware('corning_96_wellplate_360ul_flat')

# 磁力模块
mag_module = protocol.load_module('magnetic module gen2', 'C2')
mag_plate = mag_module.load_labware('nest_96_wellplate_100ul_pcr_full_skirt')

# 加热振荡器模块
hs_module = protocol.load_module('heaterShakerModuleV1', 'D1')
hs_plate = hs_module.load_labware('corning_96_wellplate_360ul_flat')

# 热循环仪模块（自动占用特定卡槽）
tc_module = protocol.load_module('thermocyclerModuleV2')
tc_plate = tc_module.load_labware('nest_96_wellplate_100ul_pcr_full_skirt')
```

### 3. 液体处理操作

**基础操作：**

```python
# 拾取吸头
pipette.pick_up_tip()

# 吸液
pipette.aspirate(
    volume=100,           # 体积(µL)
    location=source['A1'] # 孔位或位置对象
)

# 排液
pipette.dispense(
    volume=100,
    location=dest['B1']
)

# 丢弃吸头
pipette.drop_tip()

# 归还吸头
pipette.return_tip()
```

**复合操作：**

```python
# 转移（整合拾取/吸液/排液/丢弃）
pipette.transfer(
    volume=100,
    source=source_plate['A1'],
    dest=dest_plate['B1'],
    new_tip='always'  # 'always'/'once'/'never'
)

# 分配（单源到多目标）
pipette.distribute(
    volume=50,
    source=reservoir['A1'],
    dest=[plate['A1'], plate['A2'], plate['A3']],
    new_tip='once'
)

# 合并（多源到单目标）
pipette.consolidate(
    volume=50,
    source=[plate['A1'], plate['A2'], plate['A3']],
    dest=reservoir['A1'],
    new_tip='once'
)
```

**高级技巧：**

```python
# 混匀（同位置反复吸排）
pipette.mix(
    repetitions=3,
    volume=50,
    location=plate['A1']
)

# 气隙（防滴液）
pipette.aspirate(100, source['A1'])
pipette.air_gap(20)  # 20µL气隙
pipette.dispense(120, dest['A1'])

# 吹出（排尽残留液体）
pipette.blow_out(location=dest['A1'].top())

# 碰壁（去除外壁液滴）
pipette.touch_tip(location=plate['A1'])
```

**流速控制：**

```python
# 设置流速(µL/s)
pipette.flow_rate.aspirate = 150
pipette.flow_rate.dispense = 300
pipette.flow_rate.blow_out = 400
```

### 4. 孔位与位置访问

**孔位访问方法：**

```python
# 按名称
well_a1 = plate['A1']

# 按索引
first_well = plate.wells()[0]

# 所有孔位
all_wells = plate.wells()  # 返回列表

# 按行
rows = plate.rows()  # 返回列表的列表
row_a = plate.rows()[0]  # A行所有孔位

# 按列
columns = plate.columns()  # 返回列表的列表
column_1 = plate.columns()[0]  # 第1列所有孔位

# 按名称字典
wells_dict = plate.wells_by_name()  # {'A1': Well, 'A2': Well, ...}
```

**定位方法：**

```python
# 孔顶部（默认：顶部下方1mm）
pipette.aspirate(100, well.top())
pipette.aspirate(100, well.top(z=5))  # 顶部上方5mm

# 孔底部（默认：底部上方1mm）
pipette.aspirate(100, well.bottom())
pipette.aspirate(100, well.bottom(z=2))  # 底部上方2mm

# 孔中心
pipette.aspirate(100, well.center())
```

### 5. 硬件模块控制

**温控模块：**

```python
# 设置温度
temp_module.set_temperature(celsius=4)

# 等待温度稳定
temp_module.await_temperature(celsius=4)

# 停用
temp_module.deactivate()

# 查看状态
current_temp = temp_module.temperature  # 当前温度
target_temp = temp_module.target  # 目标温度
```

**磁力模块：**

```python
# 磁棒升起
mag_module.engage(height_from_base=10)  # 距耗材库底部高度(mm)

# 磁棒降下
mag_module.disengage()

# 查看状态
is_engaged = mag_module.status  # 'engaged' 或 'disengaged'
```

**加热振荡器模块：**

```python
# 设置温度
hs_module.set_target_temperature(celsius=37)

# 等待温度
hs_module.wait_for_temperature()

# 设置振荡速度
hs_module.set_and_wait_for_shake_speed(rpm=500)

# 关闭耗材锁扣
hs_module.close_labware_latch()

# 打开耗材锁扣
hs_module.open_labware_latch()

# 停用加热
hs_module.deactivate_heater()

# 停用振荡
hs_module.deactivate_shaker()
```

**热循环仪模块：**

```python
# 打开盖板
tc_module.open_lid()

# 关闭盖板
tc_module.close_lid()

# 设置盖板温度
tc_module.set_lid_temperature(celsius=105)

# 设置模块温度
tc_module.set_block_temperature(
    temperature=95,
    hold_time_seconds=30,
    hold_time_minutes=0.5,
    block_max_volume=50  # 每孔体积(µL)
)

# 执行程序（PCR循环）
profile = [
    {'temperature': 95, 'hold_time_seconds': 30},
    {'temperature': 57, 'hold_time_seconds': 30},
    {'temperature': 72, 'hold_time_seconds': 60}
]
tc_module.execute_profile(
    steps=profile,
    repetitions=30,
    block_max_volume=50
)

# 停用
tc_module.deactivate_lid()
tc_module.deactivate_block()
```

**酶标仪：**

```python
# 初始化并读取
result = plate_reader.read(wavelengths=[450, 650])

# 获取读数
absorbance_data = result  # 含波长键的字典
```

### 6. 液体追踪与标记

**定义液体：**

```python
# 定义液体类型
water = protocol.define_liquid(
    name='纯水',
    description='超纯水',
    display_color='#0000FF'  # HEX颜色码
)

sample = protocol.define_liquid(
    name='样品',
    description='细胞裂解液',
    display_color='#FF0000'
)
```

**向孔位加载液体：**

```python
# 向特定孔位加载液体
reservoir['A1'].load_liquid(liquid=water, volume=50000)  # µL
plate['A1'].load_liquid(liquid=sample, volume=100)

# 标记孔位为空
plate['B1'].load_empty()
```

### 7. 协议控制与工具

**执行控制：**

```python
# 暂停协议
protocol.pause(msg='更换吸头盒后继续')

# 延时
protocol.delay(seconds=60)
protocol.delay(minutes=5)

# 日志注释
protocol.comment('开始梯度稀释')

# 归位机器人
protocol.home()
```

**条件逻辑：**

```python
# 检查是否模拟运行
if protocol.is_simulating():
    protocol.comment('模拟模式运行中')
else:
    protocol.comment('实际机器人运行中')
```

**轨道灯（仅Flex）：**

```python
# 开灯
protocol.set_rail_lights(on=True)

# 关灯
protocol.set_rail_lights(on=False)
```

### 8. 多通道与8通道移液

使用多通道移液器时：

```python
# 加载8通道移液器
multi_pipette = protocol.load_instrument(
    'p300_multi_gen2',
    'left',
    tip_racks=[tips]
)

# 通过单孔引用整列操作
multi_pipette.transfer(
    volume=100,
    source=source_plate['A1'],  # 操作整列1
    dest=dest_plate['A1']       # 分配至整列1
)

# 使用rows()进行行操作
for row in plate.rows():
    multi_pipette.transfer(100, reservoir['A1'], row[0])
```

### 9. 常用协议模板

**梯度稀释：**

```python
def run(protocol: protocol_api.ProtocolContext):
    # 加载耗材库
    tips = protocol.load_labware('opentrons_flex_96_tiprack_200ul', 'D1')
    reservoir = protocol.load_labware('nest_12_reservoir_15ml', 'D2')
    plate = protocol.load_labware('corning_96_wellplate_360ul_flat', 'D3')

    # 加载移液器
    p300 = protocol.load_instrument('p300_single_flex', 'left', tip_racks=[tips])

    # 向首孔外所有孔添加稀释液
    p300.transfer(100, reservoir['A1'], plate.rows()[0][1:])

    # 行内梯度稀释
    p300.transfer(
        100,
        plate.rows()[0][:11],  # 源孔：0-10
        plate.rows()[0][1:],   # 目标孔：1-11
        mix_after=(3, 50),     # 排液后混匀3次(50µL)
        new_tip='always'
    )
```

**板复制：**

```python
def run(protocol: protocol_api.ProtocolContext):
    # 加载耗材库
    tips = protocol.load_labware('opentrons_flex_96_tiprack_1000ul', 'C1')
    source = protocol.load_labware('corning_96_wellplate_360ul_flat', 'D1')
    dest = protocol.load_labware('corning_96_wellplate_360ul_flat', 'D2')

    # 加载移液器
    p1000 = protocol.load_instrument('p1000_single_flex', 'left', tip_racks=[tips])

    # 全板复制
    p1000.transfer(
        100,
        source.wells(),
        dest.wells(),
        new_tip='always'
    )
```

**PCR体系构建：**

```python
def run(protocol: protocol_api.ProtocolContext):
    # 加载热循环仪
    tc_mod = protocol.load_module('thermocyclerModuleV2')
    tc_plate = tc_mod.load_labware('nest_96_wellplate_100ul_pcr_full_skirt')

    # 加载吸头与试剂
    tips = protocol.load_labware('opentrons_flex_96_tiprack_200ul', 'C1')
    reagents = protocol.load_labware('opentrons_24_tuberack_nest_1.5ml_snapcap', 'D1')

    # 加载移液器
    p300 = protocol.load_instrument('p300_single_flex', 'left', tip_racks=[tips])

    # 打开热循环仪盖板
    tc_mod.open_lid()

    # 分配预混液
    p300.distribute(
        20,
        reagents['A1'],
        tc_plate.wells(),
        new_tip='once'
    )

    # 添加样品（示例：前8孔）
    for i, well in enumerate(tc_plate.wells()[:8]):
        p300.transfer(5, reagents.wells()[i+1], well, new_tip='always')

    # 运行PCR
    tc_mod.close_lid()
    tc_mod.set_lid_temperature(105)

    # PCR程序
    tc_mod.set_block_temperature(95, hold_time_seconds=180)

    profile = [
        {'temperature': 95, 'hold_time_seconds': 15},
        {'temperature': 60, 'hold_time_

2. **使用有意义的标签**：为实验器具添加标签，便于在日志中识别  
3. **检查吸头余量**：确保有足够吸头完成实验流程  
4. **添加注释**：使用 `protocol.comment()` 进行调试和日志记录  
5. **先模拟运行**：在机器人上执行前务必进行模拟测试  
6. **优雅处理错误**：需要时添加暂停以进行人工干预  
7. **考虑时间因素**：当流程需要孵育期时添加延迟  
8. **追踪液体**：使用液体追踪功能优化实验设置验证  
9. **优化吸头使用**：适时采用 `new_tip='once'` 节省吸头  
10. **控制流速**：针对粘性或挥发性液体调整流速  

## 故障排除  

**常见问题：**  

- **吸头耗尽**：确认吸头盒容量符合流程需求  
- **实验器具碰撞**：检查台面布局是否存在空间冲突  
- **容量错误**：确保液体体积未超过孔位或移液器容量  
- **模块无响应**：验证模块连接状态并更新固件  
- **移液不准**：校准移液器并检查气泡问题  
- **模拟运行失败**：检查API版本兼容性及实验器具定义  

## 资源  

详细API文档请查阅技能目录中的 `references/api_reference.md`。  

示例流程模板请参见 `scripts/` 目录。
