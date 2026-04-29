---
name: fluidsim
description: 使用Python的计算流体动力学模拟框架。适用于运行流体动力学模拟，包括纳维-斯托克斯方程（2D/3D）、浅水方程、分层流，或分析湍流、涡动力学及地球物理流动。提供基于FFT的伪谱方法、高性能计算支持和全面的输出分析。
license: CeCILL FREE SOFTWARE LICENSE AGREEMENT
metadata:
    skill-author: K-Dense Inc.
---

# FluidSim

## 概述

FluidSim 是一个面向对象的高性能计算流体动力学（CFD）Python框架。它通过基于FFT的伪谱方法为周期性域方程提供求解器，在保持Python易用性的同时，提供与Fortran/C++相媲美的性能。

**核心优势**：
- 多求解器：2D/3D纳维-斯托克斯、浅水方程、分层流
- 高性能：Pythran/Transonic编译、MPI并行化
- 完整工作流：参数配置、模拟执行、输出分析
- 交互式分析：基于Python的后处理和可视化

## 核心功能

### 1. 安装与配置

使用uv安装fluidsim并指定功能标志：

```bash
# 基础安装
uv uv pip install fluidsim

# 支持FFT（多数求解器必需）
uv uv pip install "fluidsim[fft]"

# 支持MPI并行计算
uv uv pip install "fluidsim[fft,mpi]"
```

设置输出目录环境变量（可选）：

```bash
export FLUIDSIM_PATH=/path/to/simulation/outputs
export FLUIDDYN_PATH_SCRATCH=/path/to/working/directory
```

无需API密钥或身份验证。

完整安装指南见 `references/installation.md`。

### 2. 运行模拟

标准工作流包含五步：

**步骤1**：导入求解器
```python
from fluidsim.solvers.ns2d.solver import Simul
```

**步骤2**：创建并配置参数
```python
params = Simul.create_default_params()
params.oper.nx = params.oper.ny = 256
params.oper.Lx = params.oper.Ly = 2 * 3.14159
params.nu_2 = 1e-3
params.time_stepping.t_end = 10.0
params.init_fields.type = "noise"
```

**步骤3**：实例化模拟
```python
sim = Simul(params)
```

**步骤4**：执行
```python
sim.time_stepping.start()
```

**步骤5**：分析结果
```python
sim.output.phys_fields.plot("vorticity")
sim.output.spatial_means.plot()
```

完整示例见 `references/simulation_workflow.md`。

### 3. 可用求解器

根据物理问题选择求解器：

**2D纳维-斯托克斯** (`ns2d`)：2D湍流、涡动力学
```python
from fluidsim.solvers.ns2d.solver import Simul
```

**3D纳维-斯托克斯** (`ns3d`)：3D湍流、真实流动
```python
from fluidsim.solvers.ns3d.solver import Simul
```

**分层流** (`ns2d.strat`, `ns3d.strat`)：海洋/大气流动
```python
from fluidsim.solvers.ns2d.strat.solver import Simul
params.N = 1.0  # 布伦特-维赛拉频率
```

**浅水方程** (`sw1l`)：地球物理流动、旋转系统
```python
from fluidsim.solvers.sw1l.solver import Simul
params.f = 1.0  # 科里奥利参数
```

完整求解器列表见 `references/solvers.md`。

### 4. 参数配置

参数通过点号分层访问：

**域与分辨率**：
```python
params.oper.nx = 256  # 网格点数
params.oper.Lx = 2 * pi  # 域尺寸
```

**物理参数**：
```python
params.nu_2 = 1e-3  # 粘性系数
params.nu_4 = 0     # 超粘性系数（可选）
```

**时间步进**：
```python
params.time_stepping.t_end = 10.0
params.time_stepping.USE_CFL = True  # 自适应步长
params.time_stepping.CFL = 0.5
```

**初始条件**：
```python
params.init_fields.type = "noise"  # 可选："dipole", "vortex", "from_file", "in_script"
```

**输出设置**：
```python
params.output.periods_save.phys_fields = 1.0  # 每1.0时间单位保存
params.output.periods_save.spectra = 0.5
params.output.periods_save.spatial_means = 0.1
```

参数对象对拼写错误抛出 `AttributeError`，避免静默配置错误。

完整参数文档见 `references/parameters.md`。

### 5. 输出与分析

模拟自动生成多种输出：

**物理场**：速度、涡量（HDF5格式）
```python
sim.output.phys_fields.plot("vorticity")
sim.output.phys_fields.plot("vx")
```

**空间均值**：体积平均量的时间序列
```python
sim.output.spatial_means.plot()
```

**能谱**：能量和拟能谱
```python
sim.output.spectra.plot1d()
sim.output.spectra.plot2d()
```

**加载历史模拟**：
```python
from fluidsim import load_sim_for_plot
sim = load_sim_for_plot("simulation_dir")
sim.output.phys_fields.plot()
```

**高级可视化**：在ParaView或VisIt中打开 `.h5` 文件进行3D可视化。

完整分析流程见 `references/output_analysis.md`。

### 6. 高级功能

**自定义驱动力**：维持湍流或驱动特定动力学
```python
params.forcing.enable = True
params.forcing.type = "tcrandom"  # 时间相关随机驱动力
params.forcing.forcing_rate = 1.0
```

**自定义初始条件**：脚本定义场
```python
params.init_fields.type = "in_script"
sim = Simul(params)
X, Y = sim.oper.get_XY_loc()
vx = sim.state.state_phys.get_var("vx")
vx[:] = sin(X) * cos(Y)
sim.time_stepping.start()
```

**MPI并行化**：多处理器运行
```bash
mpirun -np 8 python simulation_script.py
```

**参数化研究**：多参数批量模拟
```python
for nu in [1e-3, 5e-4, 1e-4]:
    params = Simul.create_default_params()
    params.nu_2 = nu
    params.output.sub_directory = f"nu{nu}"
    sim = Simul(params)
    sim.time_stepping.start()
```

完整指南见 `references/advanced_features.md`。

## 典型用例

### 2D湍流研究

```python
from fluidsim.solvers.ns2d.solver import Simul
from math import pi

params = Simul.create_default_params()
params.oper.nx = params.oper.ny = 512
params.oper.Lx = params.oper.Ly = 2 * pi
params.nu_2 = 1e-4
params.time_stepping.t_end = 50.0
params.time_stepping.USE_CFL = True
params.init_fields.type = "noise"
params.output.periods_save.phys_fields = 5.0
params.output.periods_save.spectra = 1.0

sim = Simul(params)
sim.time_stepping.start()

# 分析能量级联
sim.output.spectra.plot1d(tmin=30.0, tmax=50.0)
```

### 分层流模拟

```python
from fluidsim.solvers.ns2d.strat.solver import Simul

params = Simul.create_default_params()
params.oper.nx = params.oper.ny = 256
params.N = 2.0  # 分层强度
params.nu_2 = 5e-4
params.time_stepping.t_end = 20.0

# 初始化密度层
params.init_fields.type = "in_script"
sim = Simul(params)
X, Y = sim.oper.get_XY_loc()
b = sim.state.state_phys.get_var("b")
b[:] = exp(-((X - 3.14)**2 + (Y - 3.14)**2) / 0.5)
sim.state.statephys_from_statespect()

sim.time_stepping.start()
sim.output.phys_fields.plot("b")
```

### MPI高分辨率3D模拟

```python
from fluidsim.solvers.ns3d.solver import Simul

params = Simul.create_default_params()
params.oper.nx = params.oper.ny = params.oper.nz = 512
params.nu_2 = 1e-5
params.time_stepping.t_end = 10.0
params.init_fields.type = "noise"

sim = Simul(params)
sim.time_stepping.start()
```

运行命令：
```bash
mpirun -np 64 python script.py
```

### 泰勒-格林涡验证

```python
from fluidsim.solvers.ns2d.solver import Simul
import numpy as np
from math import pi

params = Simul.create_default_params()
params.oper.nx = params.oper.ny = 128
params.oper.Lx = params.oper.Ly = 2 * pi
params.nu_2 = 1e-3
params.time_stepping.t_end = 10.0
params.init_fields.type = "in_script"

sim = Simul(params)
X, Y = sim.oper.get_XY_loc()
vx = sim.state.state_phys.get_var("vx")
vy = sim.state.state_phys.get_var("vy")
vx[:] = np.sin(X) * np.cos(Y)
vy[:] = -np.cos(X) * np.sin(Y)
sim.state.statephys_from_statespect()

sim.time_stepping.start()

# 验证能量衰减
df = sim.output.spatial_means.load()
# 与解析解对比
```

## 速查指南

**导入求解器**：`from fluidsim.solvers.ns2d.solver import Simul`

**创建参数**：`params = Simul.create_default_params()`

**设置分辨率**：`params.oper.nx = params.oper.ny = 256`

**设置粘性**：`params.nu_2 = 1e-3`

**设置结束时间**：`params.time_stepping.t_end = 10.0`

**运行模拟**：`sim = Simul(params); sim.time_stepping.start()`

**绘制结果**：`sim.output.phys_fields.plot("vorticity")`

**加载模拟**：`sim = load_sim_for_plot("path/to/sim")`

## 资源

**文档**：https://fluidsim.readthedocs.io/

**参考文件**：
- `references/installation.md`：完整安装指南
- `references/solvers.md`：求解器列表与选择指南
- `references/simulation_workflow.md`：详细工作流示例
- `references/parameters.md`：参数文档
- `references/output_analysis.md`：输出类型与分析方法
- `references/advanced_features.md`：驱动力、MPI、参数化研究、自定义求解器
