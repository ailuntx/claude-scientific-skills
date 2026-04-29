# 仿真工作流程

## 标准工作流程

按以下步骤运行 fluidsim 仿真：

### 1. 导入求解器

```python
from fluidsim.solvers.ns2d.solver import Simul

# 或使用动态导入
import fluidsim
Simul = fluidsim.import_simul_class_from_key("ns2d")
```

### 2. 创建默认参数

```python
params = Simul.create_default_params()
```

返回包含所有仿真设置的分层 `Parameters` 对象。

### 3. 配置参数

根据需要修改参数。Parameters 对象通过抛出 `AttributeError` 防止拼写错误：

```python
# 计算域与分辨率
params.oper.nx = 256  # x方向网格点数
params.oper.ny = 256  # y方向网格点数
params.oper.Lx = 2 * pi  # x方向域尺寸
params.oper.Ly = 2 * pi  # y方向域尺寸

# 物理参数
params.nu_2 = 1e-3  # 粘性系数（负拉普拉斯项）

# 时间步进
params.time_stepping.t_end = 10.0  # 终止时间
params.time_stepping.deltat0 = 0.01  # 初始时间步长
params.time_stepping.USE_CFL = True  # 自适应时间步长

# 初始条件
params.init_fields.type = "noise"  # 可选"dipole", "vortex"等

# 输出设置
params.output.periods_save.phys_fields = 1.0  # 每1.0时间单位保存
params.output.periods_save.spectra = 0.5
params.output.periods_save.spatial_means = 0.1
```

### 4. 实例化仿真对象

```python
sim = Simul(params)
```

此步骤初始化：
- 算子（FFT、微分算子）
- 状态变量（速度、涡量等）
- 输出处理器
- 时间步进方案

### 5. 运行仿真

```python
sim.time_stepping.start()
```

仿真运行至 `t_end` 或指定迭代次数。

### 6. 分析结果（运行中/运行后）

```python
# 绘制物理场
sim.output.phys_fields.plot()
sim.output.phys_fields.plot("vorticity")
sim.output.phys_fields.plot("div")

# 绘制空间均值
sim.output.spatial_means.plot()

# 绘制能谱
sim.output.spectra.plot1d()
sim.output.spectra.plot2d()
```

## 加载历史仿真

### 快速加载（仅绘图）

```python
from fluidsim import load_sim_for_plot

sim = load_sim_for_plot("path/to/simulation")
sim.output.phys_fields.plot()
sim.output.spatial_means.plot()
```

无需完整状态初始化，适用于后处理。

### 完整状态加载（用于重启）

```python
from fluidsim import load_state_phys_file

sim = load_state_phys_file("path/to/state_file.h5")
sim.time_stepping.start()  # 继续仿真
```

加载完整状态以延续仿真。

## 重启仿真

从保存状态重启：

```python
params = Simul.create_default_params()
params.init_fields.type = "from_file"
params.init_fields.from_file.path = "path/to/state_file.h5"

# 可选修改延续仿真参数
params.time_stepping.t_end = 20.0  # 延长仿真时间

sim = Simul(params)
sim.time_stepping.start()
```

## 集群运行

FluidSim 集成集群提交系统：

```python
from fluiddyn.clusters.legi import Calcul8 as Cluster

# 配置集群任务
cluster = Cluster()
cluster.submit_script(
    "my_simulation.py",
    name_run="my_job",
    nb_nodes=4,
    nb_cores_per_node=24,
    walltime="24:00:00"
)
```

脚本应包含标准工作流程步骤（导入、配置、运行）。

## 完整示例

```python
from fluidsim.solvers.ns2d.solver import Simul
from math import pi

# 创建并配置参数
params = Simul.create_default_params()
params.oper.nx = params.oper.ny = 256
params.oper.Lx = params.oper.Ly = 2 * pi
params.nu_2 = 1e-3
params.time_stepping.t_end = 10.0
params.init_fields.type = "dipole"
params.output.periods_save.phys_fields = 1.0

# 运行仿真
sim = Simul(params)
sim.time_stepping.start()

# 结果分析
sim.output.phys_fields.plot("vorticity")
sim.output.spatial_means.plot()
```
