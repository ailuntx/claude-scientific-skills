# 参数配置

## 参数对象

`Parameters` 对象采用分层结构并按逻辑分组组织。使用点号访问：

```python
params = Simul.create_default_params()
params.group.subgroup.parameter = value
```

## 关键参数组

### 算子参数 (`params.oper`)

定义计算域和分辨率：

```python
params.oper.nx = 256  # x方向网格点数
params.oper.ny = 256  # y方向网格点数
params.oper.nz = 128  # z方向网格点数（仅3D）

params.oper.Lx = 2 * pi  # x方向域长度
params.oper.Ly = 2 * pi  # y方向域长度
params.oper.Lz = pi      # z方向域长度（仅3D）

params.oper.coef_dealiasing = 2./3.  # 反混淆截断系数（默认2/3）
```

**分辨率建议**：使用2的幂次方以获得最佳FFT性能（128、256、512、1024等）

### 物理参数

#### 粘性系数

```python
params.nu_2 = 1e-3  # 拉普拉斯粘性（负拉普拉斯算子）
params.nu_4 = 0     # 超粘性（可选）
params.nu_8 = 0     # 超超粘性（极高波数阻尼）
```

高阶粘性项（`nu_4`, `nu_8`）可抑制高波数分量而不影响大尺度结构。

#### 层结参数（层化解算器）

```python
params.N = 1.0  # Brunt-Väisälä频率（浮力频率）
```

#### 旋转参数（浅水方程）

```python
params.f = 1.0  # 科里奥利参数
params.c2 = 10.0  # 平方相速度（重力波速）
```

### 时间步进 (`params.time_stepping`)

```python
params.time_stepping.t_end = 10.0  # 模拟结束时间
params.time_stepping.it_end = 100  # 或最大迭代次数

params.time_stepping.deltat0 = 0.01  # 初始时间步长
params.time_stepping.USE_CFL = True  # 启用自适应CFL时间步
params.time_stepping.CFL = 0.5  # CFL数（当USE_CFL=True时）

params.time_stepping.type_time_scheme = "RK4"  # 或"RK2", "Euler"
```

**推荐设置**：使用`USE_CFL=True`配合`CFL=0.5`实现自适应时间步进。

### 初始场设置 (`params.init_fields`)

```python
params.init_fields.type = "noise"  # 初始化方法
```

**可用类型**：
- `"noise"`: 随机噪声
- `"dipole"`: 涡旋偶极子
- `"vortex"`: 单涡旋
- `"taylor_green"`: Taylor-Green涡旋
- `"from_file"`: 从文件加载
- `"in_script"`: 脚本内定义

#### 从文件加载

```python
params.init_fields.type = "from_file"
params.init_fields.from_file.path = "path/to/state_file.h5"
```

#### 脚本内定义

```python
params.init_fields.type = "in_script"

# 创建sim后定义初始化
sim = Simul(params)

# 访问状态场
vx = sim.state.state_phys.get_var("vx")
vy = sim.state.state_phys.get_var("vy")

# 设置场
X, Y = sim.oper.get_XY_loc()
vx[:] = np.sin(X) * np.cos(Y)
vy[:] = -np.cos(X) * np.sin(Y)

# 运行模拟
sim.time_stepping.start()
```

### 输出设置 (`params.output`)

#### 输出目录

```python
params.output.sub_directory = "my_simulation"
```

目录将在`$FLUIDSIM_PATH`或当前目录下创建。

#### 保存周期

```python
params.output.periods_save.phys_fields = 1.0  # 每1.0时间单位保存物理场
params.output.periods_save.spectra = 0.5      # 保存能谱
params.output.periods_save.spatial_means = 0.1  # 保存空间平均值
params.output.periods_save.spect_energy_budg = 0.5  # 谱能量收支
```

设置为`0`可禁用特定输出类型。

#### 打印控制

```python
params.output.periods_print.print_stdout = 0.5  # 每0.5时间单位打印状态
```

#### 在线绘图

```python
params.output.periods_plot.phys_fields = 2.0  # 每2.0时间单位绘图

# 需同时启用输出模块
params.output.ONLINE_PLOT_OK = True
params.output.phys_fields.field_to_plot = "vorticity"  # 或"vx", "vy"等
```

### 强迫项 (`params.forcing`)

添加强迫项以维持能量：

```python
params.forcing.enable = True
params.forcing.type = "tcrandom"  # 时间相关随机强迫

# 强迫参数
params.forcing.nkmax_forcing = 5  # 最大强迫波数
params.forcing.nkmin_forcing = 2  # 最小强迫波数
params.forcing.forcing_rate = 1.0  # 能量注入率
```

**常用强迫类型**：
- `"tcrandom"`: 时间相关随机强迫
- `"proportional"`: 比例强迫（维持特定能谱）
- `"in_script"`: 脚本内自定义强迫

## 参数安全性

当访问不存在的参数时，参数对象会引发`AttributeError`异常：

```python
params.nu_2 = 1e-3  # 正常
params.nu2 = 1e-3   # 错误：AttributeError
```

这能防止文本配置文件中可能被静默忽略的拼写错误。

## 查看所有参数

```python
# 以XML格式打印所有参数
params._print_as_xml()

# 获取参数字典
param_dict = params._make_dict()
```

## 保存参数配置

参数随模拟输出自动保存：

```python
params._save_as_xml("simulation_params.xml")
params._save_as_json("simulation_params.json")
```
