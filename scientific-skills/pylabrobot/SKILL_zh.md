---
name: pylabrobot
description: 供应商无关的实验室自动化框架。适用于控制多种设备类型（Hamilton、Tecan、Opentrons、酶标仪、泵）或需要跨供应商统一编程的场景。最适合复杂工作流、多供应商设置和仿真。若仅需使用Opentrons官方API编写协议，opentrons-integration可能更简单。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# PyLabRobot

## 概述

PyLabRobot 是一个硬件无关的纯Python软件开发套件，用于自动化与自主化实验室。通过此技能，您可以使用统一的Python接口跨平台（Windows、macOS、Linux）控制移液机器人、酶标仪、泵、加热振荡器、培养箱、离心机及其他实验室自动化设备。

## 适用场景

在以下场景使用此技能：
- 编程控制移液机器人（Hamilton STAR/STARlet、Opentrons OT-2、Tecan EVO）
- 自动化涉及移液、样品制备或分析测量的实验室工作流
- 管理台面布局和实验室资源（微孔板、吸头盒、容器、槽板）
- 集成多种实验室设备（移液工作站、酶标仪、加热振荡器、泵）
- 创建具备状态管理的可复现实验室协议
- 在物理硬件上运行前进行协议仿真
- 使用BMG CLARIOstar或其他支持的酶标仪读取微孔板
- 控制温度、振荡、离心或其他物料处理操作
- 使用Python进行实验室自动化

## 核心能力

PyLabRobot通过六大核心能力领域提供全面的实验室自动化解决方案，详情请参阅references/目录：

### 1. 液体处理 (`references/liquid-handling.md`)

控制移液机器人进行吸液、排液和液体转移。关键操作包括：
- **基础操作**：孔间吸液、排液及液体转移
- **吸头管理**：自动拾取、丢弃及追踪移液吸头
- **高级技术**：多通道移液、连续稀释、微孔板复制
- **体积追踪**：自动追踪孔内液体体积
- **硬件支持**：Hamilton STAR/STARlet、Opentrons OT-2、Tecan EVO等

### 2. 资源管理 (`references/resources.md`)

在分层系统中管理实验室资源：
- **资源类型**：微孔板、吸头盒、槽板、试管、载架及定制耗材
- **台面布局**：通过坐标系将资源分配至台面位置
- **状态管理**：追踪吸头状态、液体体积及资源状态
- **序列化**：通过JSON文件保存和加载台面布局与状态
- **资源发现**：通过直观API访问孔位、吸头及容器

### 3. 硬件后端 (`references/hardware-backends.md`)

通过后端抽象连接多样化实验室设备：
- **移液工作站**：Hamilton STAR（全面支持）、Opentrons OT-2、Tecan EVO
- **仿真功能**：ChatterboxBackend实现无硬件协议测试
- **平台支持**：兼容Windows、macOS、Linux及树莓派
- **后端切换**：更换机器人无需重写协议

### 4. 分析设备 (`references/analytical-equipment.md`)

集成酶标仪与分析仪器：
- **酶标仪**：BMG CLARIOstar支持吸光度、化学发光、荧光检测
- **天平**：集成梅特勒托利多进行质量测量
- **集成模式**：将移液工作站与分析设备联动
- **自动化工作流**：设备间自动转移微孔板

### 5. 物料处理 (`references/material-handling.md`)

控制环境与物料处理设备：
- **加热振荡器**：Hamilton HeaterShaker、Inheco ThermoShake
- **培养箱**：Inheco和Thermo Fisher温控培养箱
- **离心机**：安捷伦VSpin支持吊篮定位与转速控制
- **泵**：Cole Parmer Masterflex进行流体泵送操作
- **温控系统**：在协议执行期间设置并监控温度

### 6. 可视化与仿真 (`references/visualization.md`)

可视化与仿真实验室协议：
- **浏览器可视化**：台面状态实时3D展示
- **仿真模式**：无需物理硬件测试协议
- **状态追踪**：可视化监控吸头状态与液体体积
- **台面编辑器**：图形化台面布局设计工具
- **协议验证**：在硬件运行前验证协议

## 快速入门

安装PyLabRobot并初始化移液工作站：

```python
# 安装PyLabRobot
# uv pip install pylabrobot

# 基础液体处理设置
from pylabrobot.liquid_handling import LiquidHandler
from pylabrobot.liquid_handling.backends import STAR
from pylabrobot.resources import STARLetDeck

# 初始化移液工作站
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
await lh.setup()

# 基础操作
await lh.pick_up_tips(tip_rack["A1:H1"])
await lh.aspirate(plate["A1"], vols=100)
await lh.dispense(plate["A2"], vols=100)
await lh.drop_tips()
```

## 参考文档使用指南

本技能将详细信息组织在多个参考文件中。在以下场景加载对应参考：
- **液体处理**：编写移液协议、吸头管理、液体转移时
- **资源管理**：定义台面布局、管理微孔板/吸头、定制耗材时
- **硬件后端**：连接特定机器人、切换平台时
- **分析设备**：集成酶标仪、天平或分析仪器时
- **物料处理**：使用加热振荡器、培养箱、离心机、泵时
- **可视化**：仿真协议、可视化台面状态时

所有参考文件位于`references/`目录，包含完整示例、API使用模式和最佳实践。

## 最佳实践

使用PyLabRobot创建实验室自动化协议时：
1. **从仿真开始**：使用ChatterboxBackend和可视化工具在硬件运行前测试协议
2. **启用追踪**：开启吸头追踪和体积追踪以实现精准状态管理
3. **资源命名**：为所有资源（微孔板、吸头盒、容器）使用清晰描述性名称
4. **状态序列化**：将台面布局和状态保存至JSON确保可复现性
5. **错误处理**：为硬件操作实施完善的异步错误处理
6. **温控策略**：提前设置温度（因加热/冷却需时较长）
7. **模块化协议**：将复杂工作流拆分为可复用函数
8. **文档参考**：访问官方文档 https://docs.pylabrobot.org 获取最新特性

## 典型工作流

### 液体转移协议

```python
# 初始化设置
lh = LiquidHandler(backend=STAR(), deck=STARLetDeck())
await lh.setup()

# 定义资源
tip_rack = TIP_CAR_480_A00(name="tip_rack")
source_plate = Cos_96_DW_1mL(name="source")
dest_plate = Cos_96_DW_1mL(name="dest")

lh.deck.assign_child_resource(tip_rack, rails=1)
lh.deck.assign_child_resource(source_plate, rails=10)
lh.deck.assign_child_resource(dest_plate, rails=15)

# 转移协议
await lh.pick_up_tips(tip_rack["A1:H1"])
await lh.transfer(source_plate["A1:H12"], dest_plate["A1:H12"], vols=100)
await lh.drop_tips()
```

### 微孔板读取工作流

```python
# 初始化酶标仪
from pylabrobot.plate_reading import PlateReader
from pylabrobot.plate_reading.clario_star_backend import CLARIOstarBackend

pr = PlateReader(name="CLARIOstar", backend=CLARIOstarBackend())
await pr.setup()

# 设置温度并读取
await pr.set_temperature(37)
await pr.open()
# （手动或机械臂加载微孔板）
await pr.close()
data = await pr.read_absorbance(wavelength=450)
```

## 附加资源

- **官方文档**：https://docs.pylabrobot.org
- **GitHub仓库**：https://github.com/PyLabRobot/pylabrobot
- **社区论坛**：https://discuss.pylabrobot.org
- **PyPI包**：https://pypi.org/project/PyLabRobot/

具体功能的详细用法请参阅`references/`目录下对应参考文件。
