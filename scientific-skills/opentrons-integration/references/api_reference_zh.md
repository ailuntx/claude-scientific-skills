# Opentrons Python 协议 API v2 参考文档

## 协议上下文方法

### 实验器具管理

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `load_labware(name, location, label=None, namespace=None, version=None)` | 将实验器具加载到工作台 | Labware 对象 |
| `load_adapter(name, location, namespace=None, version=None)` | 将适配器加载到工作台 | Labware 对象 |
| `load_labware_from_definition(definition, location, label=None)` | 从 JSON 加载自定义实验器具 | Labware 对象 |
| `load_labware_on_adapter(name, adapter, label=None)` | 在适配器上加载实验器具 | Labware 对象 |
| `load_labware_by_name(name, location, label=None, namespace=None, version=None)` | 替代加载方法 | Labware 对象 |
| `load_lid_stack(load_name, location, quantity=None)` | 加载盖子堆栈（仅限 Flex） | Labware 对象 |

### 仪器管理

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `load_instrument(instrument_name, mount, tip_racks=None, replace=False)` | 加载移液器 | InstrumentContext |

### 模块管理

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `load_module(module_name, location=None, configuration=None)` | 加载硬件模块 | ModuleContext |

### 液体管理

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `define_liquid(name, description=None, display_color=None)` | 定义液体类型 | Liquid 对象 |

### 执行控制

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `pause(msg=None)` | 暂停协议执行 | None |
| `resume()` | 暂停后恢复执行 | None |
| `delay(seconds=0, minutes=0, msg=None)` | 延迟执行 | None |
| `comment(msg)` | 添加注释到协议日志 | None |
| `home()` | 归位所有轴 | None |
| `set_rail_lights(on)` | 控制轨道灯（仅限 Flex） | None |

### 协议属性

| 属性 | 描述 | 类型 |
|----------|-------------|------|
| `deck` | 工作台布局 | Deck 对象 |
| `fixed_trash` | 固定废液槽位置（OT-2） | TrashBin 对象 |
| `loaded_labwares` | 已加载实验器具字典 | Dict |
| `loaded_instruments` | 已加载仪器字典 | Dict |
| `loaded_modules` | 已加载模块字典 | Dict |
| `is_simulating()` | 检查是否在模拟环境 | Bool |
| `bundled_data` | 访问捆绑数据文件 | Dict |
| `params` | 运行时参数 | ParametersContext |

## 仪器上下文（移液器）方法

### 吸头管理

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `pick_up_tip(location=None, presses=None, increment=None)` | 拾取吸头 | InstrumentContext |
| `drop_tip(location=None, home_after=True)` | 丢弃吸头至废液槽 | InstrumentContext |
| `return_tip(home_after=True)` | 将吸头放回架子 | InstrumentContext |
| `reset_tipracks()` | 重置吸头追踪 | None |

### 液体操作 - 基础

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `aspirate(volume=None, location=None, rate=1.0)` | 吸取液体 | InstrumentContext |
| `dispense(volume=None, location=None, rate=1.0, push_out=None)` | 分配液体 | InstrumentContext |
| `blow_out(location=None)` | 吹出残留液体 | InstrumentContext |
| `touch_tip(location=None, radius=1.0, v_offset=-1.0, speed=60.0)` | 去除吸头液滴 | InstrumentContext |
| `mix(repetitions=1, volume=None, location=None, rate=1.0)` | 混合液体 | InstrumentContext |
| `air_gap(volume=None, height=None)` | 创建空气间隙 | InstrumentContext |

### 液体操作 - 复杂

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `transfer(volume, source, dest, **kwargs)` | 转移液体 | InstrumentContext |
| `distribute(volume, source, dest, **kwargs)` | 单源多目标分配 | InstrumentContext |
| `consolidate(volume, source, dest, **kwargs)` | 多源单目标合并 | InstrumentContext |

**transfer(), distribute(), consolidate() 参数:**
- `new_tip`: 'always'（始终更换）、'once'（单次更换）或 'never'（不更换）
- `trash`: 真/假 - 使用后丢弃吸头
- `touch_tip`: 真/假 - 吸取/分配后轻触管壁
- `blow_out`: 真/假 - 分配后吹出
- `mix_before`: (混合次数, 体积) 元组
- `mix_after`: (混合次数, 体积) 元组
- `disposal_volume`: 防污染额外体积
- `carryover`: 真/假 - 启用大体积多次转移
- `gradient`: (起始浓度, 结束浓度) 用于梯度操作

### 移动与定位

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `move_to(location, force_direct=False, minimum_z_height=None, speed=None)` | 移动到指定位置 | InstrumentContext |
| `home()` | 移液器轴归位 | None |

### 移液器属性

| 属性 | 描述 | 类型 |
|----------|-------------|------|
| `default_speed` | 默认移动速度 | Float |
| `min_volume` | 最小移液体积 | Float |
| `max_volume` | 最大移液体积 | Float |
| `current_volume` | 吸头当前液体体积 | Float |
| `has_tip` | 检查是否装有吸头 | Bool |
| `name` | 移液器名称 | String |
| `model` | 移液器型号 | String |
| `mount` | 安装位置 | String |
| `channels` | 通道数量 | Int |
| `tip_racks` | 关联吸头架 | List |
| `trash_container` | 废液槽位置 | TrashBin 对象 |
| `starting_tip` | 协议起始吸头 | Well 对象 |
| `flow_rate` | 流速设置 | FlowRates 对象 |

### 流速属性

通过 `pipette.flow_rate` 访问：

| 属性 | 描述 | 单位 |
|----------|-------------|-------|
| `aspirate` | 吸取流速 | µL/s |
| `dispense` | 分配流速 | µL/s |
| `blow_out` | 吹出流速 | µL/s |

## 实验器具方法

### 孔位访问

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `wells()` | 获取所有孔位 | List[Well] |
| `wells_by_name()` | 获取孔名字典 | Dict[str, Well] |
| `rows()` | 按行获取孔位 | List[List[Well]] |
| `columns()` | 按列获取孔位 | List[List[Well]] |
| `rows_by_name()` | 获取行名字典 | Dict[str, List[Well]] |
| `columns_by_name()` | 获取列名字典 | Dict[str, List[Well]] |

### 实验器具属性

| 属性 | 描述 | 类型 |
|----------|-------------|------|
| `name` | 实验器具名称 | String |
| `parent` | 父级位置 | Location 对象 |
| `quirks` | 实验器具特性列表 | List |
| `magdeck_engage_height` | 磁力模块高度 | Float |
| `uri` | 实验器具 URI | String |
| `calibrated_offset` | 校准偏移量 | Point |

## 孔位方法与属性

### 液体操作

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `load_liquid(liquid, volume)` | 向孔位加载液体 | None |
| `load_empty()` | 标记孔位为空 | None |
| `from_center_cartesian(x, y, z)` | 从中心获取坐标 | Location |

### 定位方法

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `top(z=0)` | 获取孔位顶部位置 | Location |
| `bottom(z=0)` | 获取孔位底部位置 | Location |
| `center()` | 获取孔位中心位置 | Location |

### 孔位属性

| 属性 | 描述 | 类型 |
|----------|-------------|------|
| `diameter` | 孔径（圆形孔） | Float |
| `length` | 孔长（矩形孔） | Float |
| `width` | 孔宽（矩形孔） | Float |
| `depth` | 孔深 | Float |
| `max_volume` | 最大容积 | Float |
| `display_name` | 显示名称 | String |
| `has_tip` | 检查是否存在吸头 | Bool |

## 模块上下文

### 温度模块

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `set_temperature(celsius)` | 设置目标温度 | None |
| `await_temperature(celsius)` | 等待达到温度 | None |
| `deactivate()` | 关闭温度控制 | None |
| `load_labware(name, label=None, namespace=None, version=None)` | 在模块上加载实验器具 | Labware |

**属性:**
- `temperature`: 当前温度 (°C)
- `target`: 目标温度 (°C)
- `status`: 'idle'（空闲）、'holding'（保持）、'cooling'（降温）或 'heating'（加热）
- `labware`: 已加载实验器具

### 磁力模块

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `engage(height_from_base=None, offset=None, height=None)` | 激活磁铁 | None |
| `disengage()` | 解除磁铁 | None |
| `load_labware(name, label=None, namespace=None, version=None)` | 在模块上加载实验器具 | Labware |

**属性:**
- `status`: 'engaged'（已激活）或 'disengaged'（已解除）
- `labware`: 已加载实验器具

### 加热振荡模块

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `set_target_temperature(celsius)` | 设置加热目标 | None |
| `wait_for_temperature()` | 等待达到温度 | None |
| `set_and_wait_for_temperature(celsius)` | 设置并等待温度 | None |
| `deactivate_heater()` | 关闭加热器 | None |
| `set_and_wait_for_shake_speed(rpm)` | 设置振荡速度 | None |
| `deactivate_shaker()` | 关闭振荡器 | None |
| `open_labware_latch()` | 打开锁扣 | None |
| `close_labware_latch()` | 关闭锁扣 | None |
| `load_labware(name, label=None, namespace=None, version=None)` | 在模块上加载实验器具 | Labware |

**属性:**
- `temperature`: 当前温度 (°C)
- `target_temperature`: 目标温度 (°C)
- `current_speed`: 当前振荡速度 (rpm)
- `target_speed`: 目标振荡速度 (rpm)
- `labware_latch_status`: 'idle_open'（空闲开启）、'idle_closed'（空闲关闭）、'opening'（开启中）、'closing'（关闭中）
- `status`: 模块状态
- `labware`: 已加载实验器具

### 热循环模块

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `open_lid()` | 打开盖子 | None |
| `close_lid()` | 关闭盖子 | None |
| `set_lid_temperature(celsius)` | 设置盖子温度 | None |
| `deactivate_lid()` | 关闭盖子加热 | None |
| `set_block_temperature(temperature, hold_time_seconds=0, hold_time_minutes=0, ramp_rate=None, block_max_volume=None)` | 设置模块温度 | None |
| `deactivate_block()` | 关闭模块 | None |
| `execute_profile(steps, repetitions, block_max_volume=None)` | 运行温度曲线 | None |
| `load_labware(name, label=None, namespace=None, version=None)` | 在模块上加载实验器具 | Labware |

**温度曲线步骤格式:**
```python
{'temperature': 95, 'hold_time_seconds': 30, 'hold_time_minutes': 0}
```

**属性:**
- `block_temperature`: 当前模块温度 (°C)
- `block_target_temperature`: 目标模块温度 (°C)
- `lid_temperature`: 当前盖子温度 (°C)
- `lid_target_temperature`: 目标盖子温度 (°C)
- `lid_position`: 'open'（开启）、'closed'（关闭）、'in_between'（中间状态）
- `ramp_rate`: 模块温度变化速率 (°C/s)
- `status`: 模块状态
- `labware`: 已加载实验器具

### 吸光度板阅读器模块

| 方法 | 描述 | 返回值 |
|--------|-------------|---------|
| `initialize(mode, wavelengths)` | 初始化阅读器 | None |
| `read(export_filename=None)` | 读取板数据 | Dict |
| `close_lid()` | 关闭盖子 | None |
| `open_lid()` | 打开盖子 | None |
| `load_labware(name, label=None, namespace=None, version=None)` | 在模块上加载实验器具 | Labware |

**读取模式:**
- `'single'`: 单波长
- `'multi'`: 多波长

**属性:**
- `is_lid_on`: 盖子状态
- `labware`: 已加载实验器具

## 常用实验器具 API 名称

### 微孔板

- `corning_96_wellplate_360ul_flat`
- `nest_96_wellplate_100ul_pcr_full_skirt`
- `nest_96_wellplate_200ul_flat`
- `biorad_96_wellplate_200ul_pcr`
- `appliedbiosystems_384_wellplate_40ul`

### 储液槽

- `nest_12_reservoir_15ml`
- `nest_1_reservoir_195ml`
- `usascientific_12_reservoir_22ml`

### 吸头架

**Flex:**
- `opentrons_flex_96_tiprack_50ul`
- `opentrons_flex_96_tiprack_200ul`
- `opentrons_flex_96_tiprack_1000ul`

**OT-2:**
- `opentrons_96_tiprack_20ul`
- `opentrons_96_tiprack_300ul`
- `opentrons_96_tiprack_1000ul`

### 试管架

- `opentrons_10_tuberack_falcon_4x50ml_6x15ml_conical`
- `opentrons_24_tuberack_nest_1.5ml_snapcap`
- `opentrons_24_tuberack_nest_1.5ml_screwcap`
- `opentrons_15_tuberack_falcon_15ml_conical`

### 适配器

- `opentrons_flex_96_tiprack_adapter`
- `opentrons_96_deep_well_adapter`
- `opentrons_aluminum_flat_bottom_plate`

## 错误处理

常见异常：

- `OutOfTipsError`: 无可用吸头
- `LabwareNotLoadedError`: 实验器具未加载到工作台
- `InvalidContainerError`: 无效实验器具规格
- `InstrumentNotLoadedError`: 移液器未加载
- `InvalidVolumeError`: 体积超出范围

## 模拟与调试

检查模拟状态：
```python
if protocol.is_simulating():
    protocol.comment('运行在模拟环境中')
```

访问捆绑数据文件：
```python
data_file = protocol.bundled_data['data.csv']
with open(data_file) as f:
    data = f.read()
```

## 版本兼容性

API 级别兼容性：

| API 级别 | 功能特性 |
|-----------|----------
