# 高级功能

## 自定义驱动

### 驱动类型

FluidSim 支持多种驱动机制以维持湍流或驱动特定动力学。

#### 时间关联随机驱动

维持湍流最常用的方法：

```python
params.forcing.enable = True
params.forcing.type = "tcrandom"
params.forcing.nkmin_forcing = 2  # 最小驱动波数
params.forcing.nkmax_forcing = 5  # 最大驱动波数
params.forcing.forcing_rate = 1.0  # 能量注入率
params.forcing.tcrandom_time_correlation = 1.0  # 关联时间
```

#### 比例驱动

维持特定能量分布：

```python
params.forcing.type = "proportional"
params.forcing.forcing_rate = 1.0
```

#### 脚本内自定义驱动

直接在启动脚本中定义驱动：

```python
params.forcing.enable = True
params.forcing.type = "in_script"

sim = Simul(params)

# 定义自定义驱动函数
def compute_forcing_fft(sim):
    """在傅里叶空间中计算驱动"""
    forcing_vx_fft = sim.oper.create_arrayK(value=0.)
    forcing_vy_fft = sim.oper.create_arrayK(value=0.)

    # 添加自定义驱动逻辑
    # 示例：驱动特定模式
    forcing_vx_fft[10, 10] = 1.0 + 0.5j

    return forcing_vx_fft, forcing_vy_fft

# 重写驱动方法
sim.forcing.forcing_maker.compute_forcing_fft = lambda: compute_forcing_fft(sim)

# 运行模拟
sim.time_stepping.start()
```

## 自定义初始条件

### 脚本内初始化

完全控制初始场：

```python
from math import pi
import numpy as np

params = Simul.create_default_params()
params.oper.nx = params.oper.ny = 256
params.oper.Lx = params.oper.Ly = 2 * pi

params.init_fields.type = "in_script"

sim = Simul(params)

# 获取坐标数组
X, Y = sim.oper.get_XY_loc()

# 定义速度场
vx = sim.state.state_phys.get_var("vx")
vy = sim.state.state_phys.get_var("vy")

# Taylor-Green涡旋
vx[:] = np.sin(X) * np.cos(Y)
vy[:] = -np.cos(X) * np.sin(Y)

# 在傅里叶空间初始化状态
sim.state.statephys_from_statespect()

# 运行模拟
sim.time_stepping.start()
```

### 分层初始化（层流）

设置密度分层：

```python
from fluidsim.solvers.ns2d.strat.solver import Simul

params = Simul.create_default_params()
params.N = 1.0  # 分层强度
params.init_fields.type = "in_script"

sim = Simul(params)

# 定义致密层
X, Y = sim.oper.get_XY_loc()
b = sim.state.state_phys.get_var("b")  # 浮力场

# 高斯密度异常
x0, y0 = pi, pi
sigma = 0.5
b[:] = np.exp(-((X - x0)**2 + (Y - y0)**2) / (2 * sigma**2))

sim.state.statephys_from_statespect()
sim.time_stepping.start()
```

## MPI并行计算

### 运行MPI模拟

安装MPI支持：
```bash
uv pip install "fluidsim[fft,mpi]"
```

使用MPI运行：
```bash
mpirun -np 8 python simulation_script.py
```

FluidSim自动检测MPI并分配计算任务。

### MPI特定参数

```python
# 无需特殊参数
# FluidSim自动处理域分解

# 用于超大型3D模拟
params.oper.nx = 512
params.oper.ny = 512
params.oper.nz = 512

# 运行命令：mpirun -np 64 python script.py
```

### MPI输出

输出文件由0号处理器写入。分析脚本在串行和MPI运行中功能相同。

## 参数化研究

### 运行多组模拟

生成并运行多组参数组合的脚本：

```python
from fluidsim.solvers.ns2d.solver import Simul
import numpy as np

# 参数范围
viscosities = [1e-3, 5e-4, 1e-4, 5e-5]
resolutions = [128, 256, 512]

for nu in viscosities:
    for nx in resolutions:
        params = Simul.create_default_params()

        # 配置模拟
        params.oper.nx = params.oper.ny = nx
        params.nu_2 = nu
        params.time_stepping.t_end = 10.0

        # 唯一输出目录
        params.output.sub_directory = f"nu{nu}_nx{nx}"

        # 运行模拟
        sim = Simul(params)
        sim.time_stepping.start()
```

### 集群任务提交

向集群提交多个任务：

```python
from fluiddyn.clusters.legi import Calcul8 as Cluster

cluster = Cluster()

for nu in viscosities:
    for nx in resolutions:
        script_content = f"""
from fluidsim.solvers.ns2d.solver import Simul

params = Simul.create_default_params()
params.oper.nx = params.oper.ny = {nx}
params.nu_2 = {nu}
params.time_stepping.t_end = 10.0
params.output.sub_directory = "nu{nu}_nx{nx}"

sim = Simul(params)
sim.time_stepping.start()
"""

        with open(f"job_nu{nu}_nx{nx}.py", "w") as f:
            f.write(script_content)

        cluster.submit_script(
            f"job_nu{nu}_nx{nx}.py",
            name_run=f"sim_nu{nu}_nx{nx}",
            nb_nodes=1,
            nb_cores_per_node=24,
            walltime="12:00:00"
        )
```

### 参数化研究分析

```python
import os
import pandas as pd
from fluidsim import load_sim_for_plot
import matplotlib.pyplot as plt

results = []

# 收集所有模拟数据
for sim_dir in os.listdir("simulations"):
    sim_path = f"simulations/{sim_dir}"
    if not os.path.isdir(sim_path):
        continue

    try:
        sim = load_sim_for_plot(sim_path)

        # 提取参数
        nu = sim.params.nu_2
        nx = sim.params.oper.nx

        # 提取结果
        df = sim.output.spatial_means.load()
        final_energy = df["E"].iloc[-1]
        mean_energy = df["E"].mean()

        results.append({
            "nu": nu,
            "nx": nx,
            "final_energy": final_energy,
            "mean_energy": mean_energy
        })
    except Exception as e:
        print(f"加载{sim_dir}出错: {e}")

# 分析结果
results_df = pd.DataFrame(results)

# 绘制结果
plt.figure(figsize=(10, 6))
for nx in results_df["nx"].unique():
    subset = results_df[results_df["nx"] == nx]
    plt.plot(subset["nu"], subset["mean_energy"],
             marker="o", label=f"nx={nx}")

plt.xlabel("粘度")
plt.ylabel("平均能量")
plt.xscale("log")
plt.legend()
plt.savefig("parametric_study_results.png")
```

## 自定义求解器

### 扩展现有求解器

通过继承现有求解器创建新求解器：

```python
from fluidsim.solvers.ns2d.solver import Simul as SimulNS2D
from fluidsim.base.setofvariables import SetOfVariables

class SimulCustom(SimulNS2D):
    """包含额外物理过程的自定义求解器"""

    @staticmethod
    def _complete_params_with_default(params):
        """添加自定义参数"""
        SimulNS2D._complete_params_with_default(params)
        params._set_child("custom", {"param1": 0.0})

    def __init__(self, params):
        super().__init__(params)
        # 自定义初始化

    def tendencies_nonlin(self, state_spect=None):
        """重写以添加自定义趋势项"""
        tendencies = super().tendencies_nonlin(state_spect)

        # 添加自定义项
        # tendencies.vx_fft += custom_term_vx
        # tendencies.vy_fft += custom_term_vy

        return tendencies
```

使用自定义求解器：
```python
params = SimulCustom.create_default_params()
# 配置参数...
sim = SimulCustom(params)
sim.time_stepping.start()
```

## 在线可视化

模拟过程中实时显示场：

```python
params.output.ONLINE_PLOT_OK = True
params.output.periods_plot.phys_fields = 1.0  # 每1.0时间单位绘图
params.output.phys_fields.field_to_plot = "vorticity"

sim = Simul(params)
sim.time_stepping.start()
```

执行过程中实时显示绘图。

## 检查点与重启

### 自动检查点

```python
params.output.periods_save.phys_fields = 1.0  # 每1.0时间单位保存
```

模拟过程中自动保存场数据。

### 手动检查点

```python
# 模拟过程中
sim.output.phys_fields.save()
```

### 从检查点重启

```python
params = Simul.create_default_params()
params.init_fields.type = "from_file"
params.init_fields.from_file.path = "simulation_dir/state_phys_t5.000.h5"
params.time_stepping.t_end = 20.0  # 延长模拟时间

sim = Simul(params)
sim.time_stepping.start()
```

## 内存与性能优化

### 减少内存占用

```python
# 禁用非必要输出
params.output.periods_save.spectra = 0  # 禁用能谱保存
params.output.periods_save.spect_energy_budg = 0  # 禁用能量收支保存

# 减少空间场保存频率
params.output.periods_save.phys_fields = 10.0  # 降低保存频率
```

### 优化FFT性能

```python
import os

# 选择FFT库
os.environ["FLUIDSIM_TYPE_FFT2D"] = "fft2d.with_fftw"
os.environ["FLUIDSIM_TYPE_FFT3D"] = "fft3d.with_fftw"

# 或使用MKL（如果可用）
# os.environ["FLUIDSIM_TYPE_FFT2D"] = "fft2d.with_mkl"
```

### 时间步长优化

```python
# 使用自适应时间步长
params.time_stepping.USE_CFL = True
params.time_stepping.CFL = 0.8  # 稍大的CFL值加速运行

# 使用高效时间方案
params.time_stepping.type_time_scheme = "RK4"  # 四阶龙格-库塔法
```
