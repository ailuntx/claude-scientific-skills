# FluidSim 求解器

FluidSim 提供了多种针对不同流体动力学方程的求解器。所有求解器均采用基于快速傅里叶变换（FFT）的伪谱方法，在周期性计算域上运行。

## 可用求解器

### 二维不可压纳维-斯托克斯方程

**求解器标识符**：`ns2d`

**导入方式**：
```python
from fluidsim.solvers.ns2d.solver import Simul
# 或动态导入
Simul = fluidsim.import_simul_class_from_key("ns2d")
```

**适用场景**：二维湍流研究、涡旋动力学、基础流体流动模拟

**核心特性**：能量与拟能级串、涡量动力学

### 三维不可压纳维-斯托克斯方程

**求解器标识符**：`ns3d`

**导入方式**：
```python
from fluidsim.solvers.ns3d.solver import Simul
```

**适用场景**：三维湍流、真实流体流动模拟、高分辨率直接数值模拟（DNS）

**核心特性**：完整三维湍流动力学、并行计算支持

### 分层流（2D/3D）

**求解器标识符**：`ns2d.strat`, `ns3d.strat`

**导入方式**：
```python
from fluidsim.solvers.ns2d.strat.solver import Simul  # 二维
from fluidsim.solvers.ns3d.strat.solver import Simul  # 三维
```

**适用场景**：海洋与大气流动、密度驱动流

**核心特性**：布辛涅斯克近似、浮力效应、恒定布伦特-维赛拉频率

**参数设置**：通过`params.N`设置分层强度（布伦特-维赛拉频率）

### 浅水方程

**求解器标识符**：`sw1l`（单层模型）

**导入方式**：
```python
from fluidsim.solvers.sw1l.solver import Simul
```

**适用场景**：地球物理流动、海啸模拟、旋转流

**核心特性**：旋转坐标系支持、地转平衡

**参数设置**：通过`params.f`设置旋转（科里奥利参数）

### 费普尔-冯·卡门方程

**求解器标识符**：`fvk`（弹性板方程）

**导入方式**：
```python
from fluidsim.solvers.fvk.solver import Simul
```

**适用场景**：弹性板动力学、流固耦合研究

## 求解器选择指南

根据物理问题选择求解器：

1. **二维湍流、快速测试**：使用`ns2d`
2. **三维流动、真实场景模拟**：使用`ns3d`
3. **密度分层流动**：使用`ns2d.strat`或`ns3d.strat`
4. **地球物理流动、旋转系统**：使用`sw1l`
5. **弹性板问题**：使用`fvk`

## 修改版本

多数求解器提供包含附加物理特性的修改版本：
- 外力源项
- 不同边界条件
- 附加标量场

完整列表请查阅`fluidsim.solvers`模块。
