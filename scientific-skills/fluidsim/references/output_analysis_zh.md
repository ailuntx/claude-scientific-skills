# 输出与分析

## 输出类型

FluidSim 在模拟过程中自动保存多种类型的输出数据。

### 物理场

**文件格式**：HDF5 (`.h5`)

**位置**：`simulation_dir/state_phys_t*.h5`

**内容**：特定时刻的速度、涡量及其他物理空间场

**访问方式**：
```python
sim.output.phys_fields.plot()
sim.output.phys_fields.plot("vorticity")
sim.output.phys_fields.plot("vx")
sim.output.phys_fields.plot("div")  # 检查散度

# 手动保存
sim.output.phys_fields.save()

# 获取数据
vorticity = sim.state.state_phys.get_var("rot")
```

### 空间平均值

**文件格式**：文本文件 (`.txt`)

**位置**：`simulation_dir/spatial_means.txt`

**内容**：体积平均量随时间变化（能量、涡量等）

**访问方式**：
```python
sim.output.spatial_means.plot()

# 从文件加载
from fluidsim import load_sim_for_plot
sim = load_sim_for_plot("simulation_dir")
sim.output.spatial_means.load()
spatial_means_data = sim.output.spatial_means
```

### 能谱

**文件格式**：HDF5 (`.h5`)

**位置**：`simulation_dir/spectra_*.h5`

**内容**：能量和涡量随波数的分布

**访问方式**：
```python
sim.output.spectra.plot1d()  # 一维能谱
sim.output.spectra.plot2d()  # 二维能谱

# 加载能谱数据
spectra = sim.output.spectra.load2d_mean()
```

### 谱能量收支

**文件格式**：HDF5 (`.h5`)

**位置**：`simulation_dir/spect_energy_budg_*.h5`

**内容**：不同尺度间的能量传递

**访问方式**：
```python
sim.output.spect_energy_budg.plot()
```

## 后处理

### 加载模拟数据进行分析

#### 快速加载（只读模式）

```python
from fluidsim import load_sim_for_plot

sim = load_sim_for_plot("simulation_dir")

# 访问所有输出类型
sim.output.phys_fields.plot()
sim.output.spatial_means.plot()
sim.output.spectra.plot1d()
```

适用于快速可视化和分析，不会初始化完整模拟状态。

#### 完整状态加载

```python
from fluidsim import load_state_phys_file

sim = load_state_phys_file("simulation_dir/state_phys_t10.000.h5")

# 可继续模拟
sim.time_stepping.start()
```

### 可视化工具

#### 内置绘图功能

FluidSim 通过 matplotlib 提供基础绘图：

```python
# 物理场
sim.output.phys_fields.plot("vorticity")
sim.output.phys_fields.animate("vorticity")

# 时间序列
sim.output.spatial_means.plot()

# 能谱
sim.output.spectra.plot1d()
```

#### 高级可视化

用于出版级或三维可视化：

**ParaView**：直接打开 `.h5` 文件
```bash
paraview simulation_dir/state_phys_t*.h5
```

**VisIt**：类似 ParaView，适用于大型数据集

**自定义 Python 脚本**：
```python
import h5py
import matplotlib.pyplot as plt

# 手动加载场数据
with h5py.File("state_phys_t10.000.h5", "r") as f:
    vx = f["state_phys"]["vx"][:]
    vy = f["state_phys"]["vy"][:]

# 自定义绘图
plt.contourf(vx)
plt.show()
```

## 分析示例

### 能量演化分析

```python
from fluidsim import load_sim_for_plot
import matplotlib.pyplot as plt

sim = load_sim_for_plot("simulation_dir")
df = sim.output.spatial_means.load()

plt.figure()
plt.plot(df["t"], df["E"], label="动能")
plt.xlabel("时间")
plt.ylabel("能量")
plt.legend()
plt.show()
```

### 谱分析

```python
sim = load_sim_for_plot("simulation_dir")

# 绘制能量谱
sim.output.spectra.plot1d(tmin=5.0, tmax=10.0)  # 时间范围内平均

# 获取谱数据
k, E_k = sim.output.spectra.load1d_mean(tmin=5.0, tmax=10.0)

# 检查幂律关系
import numpy as np
log_k = np.log(k)
log_E = np.log(E_k)
# 在惯性区间拟合幂律
```

### 参数化研究分析

运行多个不同参数的模拟时：

```python
import os
import pandas as pd
from fluidsim import load_sim_for_plot

# 收集多个模拟结果
results = []
for sim_dir in os.listdir("simulations"):
    if not os.path.isdir(f"simulations/{sim_dir}"):
        continue

    sim = load_sim_for_plot(f"simulations/{sim_dir}")

    # 提取关键指标
    df = sim.output.spatial_means.load()
    final_energy = df["E"].iloc[-1]

    # 获取参数
    nu = sim.params.nu_2

    results.append({
        "nu": nu,
        "final_energy": final_energy,
        "sim_dir": sim_dir
    })

# 分析结果
results_df = pd.DataFrame(results)
results_df.plot(x="nu", y="final_energy", logx=True)
```

### 场操作

```python
sim = load_sim_for_plot("simulation_dir")

# 加载特定时刻
sim.output.phys_fields.set_of_phys_files.update_times()
times = sim.output.phys_fields.set_of_phys_files.times

# 加载特定时刻场数据
field_file = sim.output.phys_fields.get_field_to_plot(time=5.0)
vorticity = field_file.get_var("rot")

# 计算衍生量
import numpy as np
vorticity_rms = np.sqrt(np.mean(vorticity**2))
vorticity_max = np.max(np.abs(vorticity))
```

## 输出目录结构

```
simulation_dir/
├── params_simul.xml         # 模拟参数
├── stdout.txt               # 标准输出日志
├── state_phys_t*.h5         # 不同时刻的物理场
├── spatial_means.txt        # 空间平均量的时间序列
├── spectra_*.h5            # 谱数据
├── spect_energy_budg_*.h5  # 能量收支数据
└── info_solver.txt         # 求解器信息
```

## 性能监控

```python
# 模拟过程中检查进度
sim.output.print_stdout.complete_timestep()

# 模拟后审查性能
sim.output.print_stdout.plot_deltat()  # 绘制时间步长演化
sim.output.print_stdout.plot_clock_times()  # 绘制计算时间
```

## 数据导出

将 fluidsim 输出转换为其他格式：

```python
import h5py
import numpy as np

# 导出为 numpy 数组
with h5py.File("state_phys_t10.000.h5", "r") as f:
    vx = f["state_phys"]["vx"][:]
    np.save("vx.npy", vx)

# 导出为 CSV
df = sim.output.spatial_means.load()
df.to_csv("spatial_means.csv", index=False)
```
