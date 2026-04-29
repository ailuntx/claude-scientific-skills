# QuTiP 高级功能

## Floquet 理论

适用于时间周期性哈密顿量 H(t + T) = H(t)。

### Floquet 模式与准能量

```python
from qutip import *
import numpy as np

# 时间周期性哈密顿量
w_d = 1.0  # 驱动频率
T = 2 * np.pi / w_d  # 周期

H0 = sigmaz()
H1 = sigmax()
H = [H0, [H1, 'cos(w*t)']]
args = {'w': w_d}

# 计算 Floquet 模式和准能量
f_modes, f_energies = floquet_modes(H, T, args)

print("准能量:", f_energies)
print("Floquet 模式:", f_modes)
```

### 时刻 t 的 Floquet 态

```python
# 获取特定时刻的 Floquet 态
t = 1.0
f_states_t = floquet_states(f_modes, f_energies, t)
```

### Floquet 态分解

```python
# 在 Floquet 基中分解初始态
psi0 = basis(2, 0)
f_coeff = floquet_state_decomposition(f_modes, f_energies, psi0)
```

### Floquet-Markov 主方程

```python
# 含耗散的时间演化
c_ops = [np.sqrt(0.1) * sigmam()]
tlist = np.linspace(0, 20, 200)

result = fmmesolve(H, psi0, tlist, c_ops, e_ops=[sigmaz()], T=T, args=args)

# 绘制结果
import matplotlib.pyplot as plt
plt.plot(tlist, result.expect[0])
plt.xlabel('时间')
plt.ylabel('⟨σz⟩')
plt.show()
```

### Floquet 张量

```python
# Floquet 张量（广义 Bloch-Redfield）
A_ops = [[sigmaz(), lambda w: 0.1 * w if w > 0 else 0]]

# 构建 Floquet 张量
R, U = floquet_markov_mesolve(H, psi0, tlist, A_ops, e_ops=[sigmaz()],
                               T=T, args=args)
```

### 有效哈密顿量

```python
# 时间平均有效哈密顿量
H_eff = floquet_master_equation_steadystate(H, c_ops, T, args)
```

## 层级运动方程 (HEOM)

适用于具有强系统-环境耦合的非马尔可夫开放量子系统。

### 基础 HEOM 设置

```python
from qutip import heom

# 系统哈密顿量
H_sys = sigmaz()

# 环境关联函数（指数型）
Q = sigmax()  # 系统-环境耦合算符
ck_real = [0.1]  # 耦合强度
vk_real = [0.5]  # 环境频率

# HEOM 环境
bath = heom.BosonicBath(Q, ck_real, vk_real)

# 初始态
rho0 = basis(2, 0) * basis(2, 0).dag()

# 创建 HEOM 求解器
max_depth = 5
hsolver = heom.HEOMSolver(H_sys, [bath], max_depth=max_depth)

# 时间演化
tlist = np.linspace(0, 10, 100)
result = hsolver.run(rho0, tlist)

# 提取约化系统密度矩阵
rho_sys = [r.extract_state(0) for r in result.states]
```

### 多环境设置

```python
# 定义多个环境
bath1 = heom.BosonicBath(sigmax(), [0.1], [0.5])
bath2 = heom.BosonicBath(sigmay(), [0.05], [1.0])

hsolver = heom.HEOMSolver(H_sys, [bath1, bath2], max_depth=5)
```

### Drude-Lorentz 谱密度

```python
# 凝聚态物理中常见
from qutip.nonmarkov.heom import DrudeLorentzBath

lam = 0.1  # 重组能
gamma = 0.5  # 环境截止频率
T = 1.0  # 温度（能量单位）
Nk = 2  # Matsubara 项数

bath = DrudeLorentzBath(Q, lam, gamma, T, Nk)
```

### HEOM 选项配置

```python
options = heom.HEOMSolver.Options(
    nsteps=2000,
    store_states=True,
    rtol=1e-7,
    atol=1e-9
)

hsolver = heom.HEOMSolver(H_sys, [bath], max_depth=5, options=options)
```

## 置换不变性

适用于全同粒子系统（如自旋系综）。

### Dicke 态

```python
from qutip import dicke

# N 个自旋的 Dicke 态 |j, m⟩
N = 10  # 自旋数
j = N/2  # 总角动量
m = 0   # z分量

psi = dicke(N, j, m)
```

### 置换不变算符

```python
from qutip.piqs import jspin

# 集体自旋算符
N = 10
Jx = jspin(N, 'x')
Jy = jspin(N, 'y')
Jz = jspin(N, 'z')
Jp = jspin(N, '+')
Jm = jspin(N, '-')
```

### PIQS 动力学

```python
from qutip.piqs import Dicke

# 建立 Dicke 模型
N = 10
emission = 1.0
dephasing = 0.5
pumping = 0.0
collective_emission = 0.0

system = Dicke(N=N, emission=emission, dephasing=dephasing,
               pumping=pumping, collective_emission=collective_emission)

# 初始态
psi0 = dicke(N, N/2, N/2)  # 所有自旋向上

# 时间演化
tlist = np.linspace(0, 10, 100)
result = system.solve(psi0, tlist, e_ops=[Jz])
```

## 非马尔可夫蒙特卡洛

具有记忆效应的量子轨迹方法。

```python
from qutip import nm_mcsolve

# 非马尔可夫环境关联
def bath_correlation(t1, t2):
    tau = abs(t2 - t1)
    return np.exp(-tau / 2.0) * np.cos(tau)

# 系统设置
H = sigmaz()
c_ops = [sigmax()]
psi0 = basis(2, 0)
tlist = np.linspace(0, 10, 100)

# 含记忆求解
result = nm_mcsolve(H, psi0, tlist, c_ops, sc_ops=[],
                     bath_corr=bath_correlation, ntraj=500,
                     e_ops=[sigmaz()])
```

## 含测量的随机求解器

### 连续测量

```python
# 零差探测
sc_ops = [np.sqrt(0.1) * destroy(N)]  # 测量算符

result = ssesolve(H, psi0, tlist, sc_ops=sc_ops,
                   e_ops=[num(N)], ntraj=100,
                   noise=11)  # 11 表示零差

# 外差探测
result = ssesolve(H, psi0, tlist, sc_ops=sc_ops,
                   e_ops=[num(N)], ntraj=100,
                   noise=12)  # 12 表示外差
```

### 光子计数

```python
# 量子跳跃时间
result = mcsolve(H, psi0, tlist, c_ops, ntraj=50,
                 options=Options(store_states=True))

# 提取测量时间
for i, jump_times in enumerate(result.col_times):
    print(f"轨迹 {i} 跳跃时间: {jump_times}")
    print(f"作用算符: {result.col_which[i]}")
```

## Krylov 子空间方法

适用于大型系统的高效求解。

```python
from qutip import krylovsolve

# 使用 Krylov 求解器
result = krylovsolve(H, psi0, tlist, krylov_dim=10, e_ops=[num(N)])
```

## Bloch-Redfield 主方程

适用于弱系统-环境耦合。

```python
# 环境谱密度
def ohmic_spectrum(w):
    if w >= 0:
        return 0.1 * w  # Ohmic 谱
    else:
        return 0

# 耦合算符与谱函数
a_ops = [[sigmax(), ohmic_spectrum]]

# 求解
result = brmesolve(H, psi0, tlist, a_ops, e_ops=[sigmaz()])
```

### 温度相关环境

```python
def thermal_spectrum(w):
    # 玻色-爱因斯坦分布
    T = 1.0  # 温度
    if abs(w) < 1e-10:
        return 0.1 * T
    n_th = 1 / (np.exp(abs(w)/T) - 1)
    if w >= 0:
        return 0.1 * w * (n_th + 1)
    else:
        return 0.1 * abs(w) * n_th

a_ops = [[sigmax(), thermal_spectrum]]
result = brmesolve(H, psi0, tlist, a_ops, e_ops=[sigmaz()])
```

## 超算符与量子通道

### 超算符表示

```python
# Liouvillian 超算符
L = liouvillian(H, c_ops)

# 表示形式转换
from qutip import (spre, spost, sprepost,
                    super_to_choi, choi_to_super,
                    super_to_kraus, kraus_to_super)

# 超算符形式
L_spre = spre(H)  # 左乘
L_spost = spost(H)  # 右乘
L_sprepost = sprepost(H, H.dag())

# Choi 矩阵
choi = super_to_choi(L)

# Kraus 算符
kraus = super_to_kraus(L)
```

### 量子通道

```python
# 去极化通道
p = 0.1  # 错误概率
K0 = np.sqrt(1 - 3*p/4) * qeye(2)
K1 = np.sqrt(p/4) * sigmax()
K2 = np.sqrt(p/4) * sigmay()
K3 = np.sqrt(p/4) * sigmaz()

kraus_ops = [K0, K1, K2, K3]
E = kraus_to_super(kraus_ops)

# 应用通道
rho_out = E * operator_to_vector(rho_in)
rho_out = vector_to_operator(rho_out)
```

### 振幅阻尼

```python
# T1 衰减
gamma = 0.1
K0 = Qobj([[1, 0], [0, np.sqrt(1 - gamma)]])
K1 = Qobj([[0, np.sqrt(gamma)], [0, 0]])

E_damping = kraus_to_super([K0, K1])
```

### 相位阻尼

```python
# T2 退相位
gamma = 0.1
K0 = Qobj([[1, 0], [0, np.sqrt(1 - gamma/2)]])
K1 = Qobj([[0, 0], [0, np.sqrt(gamma/2)]])

E_dephasing = kraus_to_super([K0, K1])
```

## 量子轨迹分析

### 提取单条轨迹

```python
options = Options(store_states=True, store_final_state=False)
result = mcsolve(H, psi0, tlist, c_ops, ntraj=100, options=options)

# 访问单条轨迹
for i in range(len(result.states)):
    trajectory = result.states[i]  # 轨迹 i 的状态序列
    # 分析轨迹
```

### 轨迹统计

```python
# 均值与标准差
result = mcsolve(H, psi0, tlist, c_ops, e_ops=[num(N)], ntraj=500)

n_mean = result.expect[0]
n_std = result.std_expect[0]

# 最终时刻光子数分布
final_states = [result.states[i][-1] for i in range(len(result.states))]
```

## 时间依赖项进阶

### QobjEvo 对象

```python
from qutip import QobjEvo

# 使用 QobjEvo 的时间依赖哈密顿量
def drive(t, args):
    return args['A'] * np.exp(-t/args['tau']) * np.sin(args['w'] * t)

H0 = num(N)
H1 = destroy(N) + create(N)
args = {'A': 1.0, 'w': 1.0, 'tau': 5.0}

H_td = QobjEvo([H0, [H1, drive]], args=args)

# 无需重建即可更新参数
H_td.arguments({'A': 2.0, 'w': 1.5, 'tau': 10.0})
```

### 编译时间依赖项

```python
# 最快方法（需 Cython）
H = [num(N), [destroy(N) + create(N), 'A * exp(-t/tau) * sin(w*t)']]
args = {'A': 1.0, 'w': 1.0, 'tau': 5.0}

# QuTiP 将编译此表达式以加速
result = sesolve(H, psi0, tlist, args=args)
```

### 回调函数

```python
# 高级控制
def time_dependent_coeff(t, args):
    # 按需访问求解器状态
    return complex_function(t, args)

H = [H0, [H1, time_dependent_coeff]]
```

## 并行处理

### 并行映射

```python
from qutip import parallel_map

# 定义任务
def simulate(gamma):
    c_ops = [np.sqrt(gamma) * destroy(N)]
    result = mesolve(H, psi0, tlist, c_ops, e_ops=[num(N)])
    return result.expect[0]

# 并行执行
gamma_values = np.linspace(0, 1, 20)
results = parallel_map(simulate, gamma_values, num_cpus=4)
```

### 串行映射（调试用）

```python
from qutip import serial_map

# 相同接口但串行执行
results = serial_map(simulate, gamma_values)
```

## 文件输入输出

### 保存/加载量子对象

```python
# 保存
H.save('hamiltonian.qu')
psi.save('state.qu')

# 加载
H_loaded = qload('hamiltonian.qu')
psi_loaded = qload('state.qu')
```

### 保存/加载计算结果

```python
# 保存模拟结果
result = mesolve(H, psi0, tlist, c_ops, e_ops=[num(N)])
result.save('simulation.dat')

# 加载结果
from qutip import Result
loaded_result = Result.load('simulation.dat')
```

### 导出至 MATLAB

```python
# 导出为 .mat 文件
H.matlab_export('hamiltonian.mat', 'H')
```

## 求解器选项

### 微调解算器

```python
options = Options()

# 积分参数
options.nsteps = 10000  # 最大内部步数
options.rtol = 1e-8     # 相对容差
options.atol = 1e-10    # 绝对容差

# 方法选择
options.method = 'adams'  # 非刚性（默认）
# options.method = 'bdf'  # 刚性系统

# 存储选项
options.store_states = True
options.store_final_state = True

# 进度条
options.progress_bar = True

# 随机数种子（确保可复现性）
options.seeds = 12345

result = mesolve(H, psi0, tlist, c_ops, options=options)
```

### 调试模式

```python
#
