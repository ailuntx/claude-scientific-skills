# QuTiP 时间演化与动力学求解器

## 概述

QuTiP 提供多种量子动力学求解器：
- `sesolve` - 薛定谔方程（幺正演化）
- `mesolve` - 主方程（含耗散的开放系统）
- `mcsolve` - 蒙特卡洛（量子轨迹）
- `brmesolve` - Bloch-Redfield 主方程
- `fmmesolve` - Floquet-Markov 主方程
- `ssesolve/smesolve` - 随机薛定谔/主方程

## 薛定谔方程求解器 (sesolve)

用于封闭量子系统的幺正演化。

### 基础用法

```python
from qutip import *
import numpy as np

# 系统设置
N = 10
psi0 = basis(N, 0)  # 初始态
H = num(N)  # 哈密顿量

# 时间点
tlist = np.linspace(0, 10, 100)

# 求解
result = sesolve(H, psi0, tlist)

# 访问结果
states = result.states  # 各时刻态列表
final_state = result.states[-1]
```

### 含期望值计算

```python
# 计算期望值的算符
e_ops = [num(N), destroy(N), create(N)]

result = sesolve(H, psi0, tlist, e_ops=e_ops)

# 访问期望值
n_t = result.expect[0]  # ⟨n⟩(t)
a_t = result.expect[1]  # ⟨a⟩(t)
```

### 含时哈密顿量

```python
# 方法1：字符串形式（更快，需Cython）
H = [num(N), [destroy(N) + create(N), 'cos(w*t)']]
args = {'w': 1.0}
result = sesolve(H, psi0, tlist, args=args)

# 方法2：函数形式
def drive(t, args):
    return np.exp(-t/args['tau']) * np.sin(args['w'] * t)

H = [num(N), [destroy(N) + create(N), drive]]
args = {'w': 1.0, 'tau': 5.0}
result = sesolve(H, psi0, tlist, args=args)

# 方法3：QobjEvo（最灵活）
from qutip import QobjEvo
H_td = QobjEvo([num(N), [destroy(N) + create(N), drive]], args=args)
result = sesolve(H_td, psi0, tlist)
```

## 主方程求解器 (mesolve)

用于含耗散和退相干的开放量子系统。

### 基础用法

```python
# 系统哈密顿量
H = num(N)

# 坍缩算符（Lindblad算符）
kappa = 0.1  # 衰减率
c_ops = [np.sqrt(kappa) * destroy(N)]

# 初始态
psi0 = coherent(N, 2.0)

# 求解
result = mesolve(H, psi0, tlist, c_ops, e_ops=[num(N)])

# 结果为密度矩阵演化
rho_t = result.states  # 密度矩阵列表
n_t = result.expect[0]  # ⟨n⟩(t)
```

### 多重耗散通道

```python
# 光子损耗
kappa = 0.1
# 退相位
gamma = 0.05
# 热激发
nth = 0.5  # 热光子数

c_ops = [
    np.sqrt(kappa * (1 + nth)) * destroy(N),  # 热衰减
    np.sqrt(kappa * nth) * create(N),  # 热激发
    np.sqrt(gamma) * num(N)  # 纯退相位
]

result = mesolve(H, psi0, tlist, c_ops)
```

### 含时耗散

```python
# 含时衰减率
def kappa_t(t, args):
    return args['k0'] * (1 + np.sin(args['w'] * t))

c_ops = [[np.sqrt(1.0) * destroy(N), kappa_t]]
args = {'k0': 0.1, 'w': 1.0}

result = mesolve(H, psi0, tlist, c_ops, args=args)
```

## 蒙特卡洛求解器 (mcsolve)

模拟开放系统的量子轨迹。

### 基础用法

```python
# 与mesolve相同设置
H = num(N)
c_ops = [np.sqrt(0.1) * destroy(N)]
psi0 = coherent(N, 2.0)

# 轨迹数量
ntraj = 500

result = mcsolve(H, psi0, tlist, c_ops, e_ops=[num(N)], ntraj=ntraj)

# 轨迹平均结果
n_avg = result.expect[0]
n_std = result.std_expect[0]  # 标准差

# 单条轨迹（需options.store_states=True）
options = Options(store_states=True)
result = mcsolve(H, psi0, tlist, c_ops, ntraj=ntraj, options=options)
trajectories = result.states  # 轨迹列表
```

### 光子计数

```python
# 跟踪量子跳跃
result = mcsolve(H, psi0, tlist, c_ops, ntraj=ntraj, options=options)

# 访问跳跃时间及对应算符
for traj in result.col_times:
    print(f"跳跃时间: {traj}")

for traj in result.col_which:
    print(f"跳跃算符索引: {traj}")
```

## Bloch-Redfield 求解器 (brmesolve)

用于弱系统-环境耦合的旋波近似。

```python
# 系统哈密顿量
H = sigmaz()

# 耦合算符与谱密度
a_ops = [[sigmax(), lambda w: 0.1 * w if w > 0 else 0]]  # Ohmic浴

psi0 = basis(2, 0)
result = brmesolve(H, psi0, tlist, a_ops, e_ops=[sigmaz(), sigmax()])
```

## Floquet 求解器 (fmmesolve)

用于含周期驱动哈密顿量。

```python
# 含时周期哈密顿量
w_d = 1.0  # 驱动频率
H0 = sigmaz()
H1 = sigmax()
H = [H0, [H1, 'cos(w*t)']]
args = {'w': w_d}

# Floquet模式与准能量
T = 2 * np.pi / w_d  # 周期
f_modes, f_energies = floquet_modes(H, T, args)

# Floquet基下的初始态
psi0 = basis(2, 0)

# Floquet基下的耗散
c_ops = [np.sqrt(0.1) * sigmam()]

result = fmmesolve(H, psi0, tlist, c_ops, e_ops=[num(2)], T=T, args=args)
```

## 随机求解器

### 随机薛定谔方程 (ssesolve)

```python
# 扩散算符
sc_ops = [np.sqrt(0.1) * destroy(N)]

# 外差探测
result = ssesolve(H, psi0, tlist, sc_ops=sc_ops, e_ops=[num(N)],
                   ntraj=500, noise=1)  # noise=1 表示外差探测
```

### 随机主方程 (smesolve)

```python
result = smesolve(H, psi0, tlist, c_ops=[], sc_ops=sc_ops,
                   e_ops=[num(N)], ntraj=500)
```

## 传播子

### 时间演化算符

```python
# 演化算符 U(t) 满足 ψ(t) = U(t)ψ(0)
U = (-1j * H * t).expm()
psi_t = U * psi0

# 主方程情形（超算符传播子）
L = liouvillian(H, c_ops)
U_super = (L * t).expm()
rho_t = vector_to_operator(U_super * operator_to_vector(rho0))
```

### 传播子函数

```python
# 生成多时刻传播子
U_list = propagator(H, tlist, c_ops)

# 应用于量子态
psi_t = [U_list[i] * psi0 for i in range(len(tlist))]
```

## 稳态解

### 直接稳态求解

```python
# 求Liouvillian的稳态
rho_ss = steadystate(H, c_ops)

# 验证稳态性
L = liouvillian(H, c_ops)
assert (L * operator_to_vector(rho_ss)).norm() < 1e-10
```

### 伪逆方法

```python
# 用于简并稳态
rho_ss = steadystate(H, c_ops, method='direct')
# 或 'eigen', 'svd', 'power'
```

## 关联函数

### 双时关联

```python
# ⟨A(t+τ)B(t)⟩
A = destroy(N)
B = create(N)

# 发射谱
taulist = np.linspace(0, 10, 200)
corr = correlation_2op_1t(H, None, taulist, c_ops, A, B)

# 功率谱
w, S = spectrum_correlation_fft(taulist, corr)
```

### 多时关联

```python
# ⟨A(t3)B(t2)C(t1)⟩
corr = correlation_3op_1t(H, None, taulist, c_ops, A, B, C)
```

## 求解器选项

```python
from qutip import Options

options = Options()
options.nsteps = 10000  # 最大内部步数
options.atol = 1e-8  # 绝对容差
options.rtol = 1e-6  # 相对容差
options.method = 'adams'  # 或 'bdf' 处理刚性问题
options.store_states = True  # 存储所有态
options.store_final_state = True  # 仅存储终态

result = mesolve(H, psi0, tlist, c_ops, options=options)
```

### 进度条

```python
options.progress_bar = True
result = mesolve(H, psi0, tlist, c_ops, options=options)
```

## 结果保存与加载

```python
# 保存结果
result.save("my_simulation.dat")

# 加载结果
from qutip import Result
loaded_result = Result.load("my_simulation.dat")
```

## 高效模拟技巧

1. **稀疏矩阵**：QuTiP 自动使用稀疏矩阵
2. **小希尔伯特空间**：尽可能截断
3. **含时项**：字符串格式最快（需编译）
4. **并行轨迹**：mcsolve 自动并行化
5. **收敛性**：通过调整 `ntraj`, `nsteps`, 容差验证
6. **求解器选择**：
   - 纯态：用 `sesolve`（更快）
   - 混合态/耗散：用 `mesolve`
   - 噪声/测量：用 `mcsolve`
   - 弱耦合：用 `brmesolve`
   - 周期驱动：用 Floquet 方法
