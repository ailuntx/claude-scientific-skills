---
name: qutip
description: 用于开放量子系统模拟的量子物理库。适用于研究主方程、林德布拉德动力学、退相干、量子光学或腔量子电动力学。最适合物理研究、开放系统动力学和教育模拟。不适用于基于电路的量子计算——量子算法和硬件执行请使用qiskit、cirq或pennylane。
license: BSD-3-Clause license
metadata:
    skill-author: K-Dense Inc.
---

# QuTiP：Python量子工具箱

## 概述

QuTiP提供全面的量子力学系统模拟与分析工具。它支持封闭（幺正）和开放（耗散）量子系统，提供针对不同场景优化的多种求解器。

## 安装

```bash
uv pip install qutip
```

扩展功能可选包：

```bash
# 量子信息处理（电路、门操作）
uv pip install qutip-qip

# 量子轨迹可视化工具
uv pip install qutip-qtrl
```

## 快速入门

```python
from qutip import *
import numpy as np
import matplotlib.pyplot as plt

# 创建量子态
psi = basis(2, 0)  # |0⟩态

# 创建算符
H = sigmaz()  # 哈密顿量

# 时间演化
tlist = np.linspace(0, 10, 100)
result = sesolve(H, psi, tlist, e_ops=[sigmaz()])

# 绘制结果
plt.plot(tlist, result.expect[0])
plt.xlabel('时间')
plt.ylabel('⟨σz⟩')
plt.show()
```

## 核心功能

### 1. 量子对象与态

创建和操作量子态与算符：

```python
# 量子态
psi = basis(N, n)  # 福克态 |n⟩
psi = coherent(N, alpha)  # 相干态 |α⟩
rho = thermal_dm(N, n_avg)  # 热密度矩阵

# 算符
a = destroy(N)  # 湮灭算符
H = num(N)  # 粒子数算符
sx, sy, sz = sigmax(), sigmay(), sigmaz()  # 泡利矩阵

# 复合系统
psi_AB = tensor(psi_A, psi_B)  # 张量积
```

**参见** `references/core_concepts.md` 获取量子对象、态、算符和张量积的完整说明。

### 2. 时间演化与动力学

多种场景求解器：

```python
# 封闭系统（幺正演化）
result = sesolve(H, psi0, tlist, e_ops=[num(N)])

# 开放系统（耗散）
c_ops = [np.sqrt(0.1) * destroy(N)]  # 坍缩算符
result = mesolve(H, psi0, tlist, c_ops, e_ops=[num(N)])

# 量子轨迹（蒙特卡洛）
result = mcsolve(H, psi0, tlist, c_ops, ntraj=500, e_ops=[num(N)])
```

**求解器选择指南：**
- `sesolve`：纯态，幺正演化
- `mesolve`：混合态，耗散，通用开放系统
- `mcsolve`：量子跳跃，光子计数，单条轨迹
- `brmesolve`：弱系统-环境耦合
- `fmmesolve`：时间周期哈密顿量（Floquet）

**参见** `references/time_evolution.md` 获取求解器文档、含时哈密顿量和高级选项的详细说明。

### 3. 分析与测量

计算物理量：

```python
# 期望值
n_avg = expect(num(N), psi)

# 熵度量
S = entropy_vn(rho)  # 冯·诺依曼熵
C = concurrence(rho)  # 纠缠度（双量子比特）

# 保真度与距离
F = fidelity(psi1, psi2)
D = tracedist(rho1, rho2)

# 关联函数
corr = correlation_2op_1t(H, rho0, taulist, c_ops, A, B)
w, S = spectrum_correlation_fft(taulist, corr)

# 稳态
rho_ss = steadystate(H, c_ops)
```

**参见** `references/analysis.md` 获取熵、保真度、测量、关联函数和稳态计算的完整说明。

### 4. 可视化

量子态与动力学可视化：

```python
# 布洛赫球
b = Bloch()
b.add_states(psi)
b.show()

# 维格纳函数（相空间）
xvec = np.linspace(-5, 5, 200)
W = wigner(psi, xvec, xvec)
plt.contourf(xvec, xvec, W, 100, cmap='RdBu')

# 福克分布
plot_fock_distribution(psi)

# 矩阵可视化
hinton(rho)  # 欣顿图
matrix_histogram(H.full())  # 3D柱状图
```

**参见** `references/visualization.md` 获取布洛赫球动画、维格纳函数、Q函数和矩阵可视化方法。

### 5. 高级方法

复杂场景专用技术：

```python
# Floquet理论（周期哈密顿量）
T = 2 * np.pi / w_drive
f_modes, f_energies = floquet_modes(H, T, args)
result = fmmesolve(H, psi0, tlist, c_ops, T=T, args=args)

# HEOM（非马尔可夫，强耦合）
from qutip.nonmarkov.heom import HEOMSolver, BosonicBath
bath = BosonicBath(Q, ck_real, vk_real)
hsolver = HEOMSolver(H_sys, [bath], max_depth=5)
result = hsolver.run(rho0, tlist)

# 置换不变性（全同粒子）
psi = dicke(N, j, m)  # 迪克态
Jz = jspin(N, 'z')  # 集体算符
```

**参见** `references/advanced.md` 获取Floquet理论、HEOM、置换不变性、随机求解器、超算符和性能优化的完整说明。

## 典型工作流

### 阻尼谐振子模拟

```python
# 系统参数
N = 20  # 希尔伯特空间维度
omega = 1.0  # 振荡频率
kappa = 0.1  # 衰减率

# 哈密顿量与坍缩算符
H = omega * num(N)
c_ops = [np.sqrt(kappa) * destroy(N)]

# 初始态
psi0 = coherent(N, 3.0)

# 时间演化
tlist = np.linspace(0, 50, 200)
result = mesolve(H, psi0, tlist, c_ops, e_ops=[num(N)])

# 可视化
plt.plot(tlist, result.expect[0])
plt.xlabel('时间')
plt.ylabel('⟨n⟩')
plt.title('光子数衰减')
plt.show()
```

### 双量子比特纠缠动力学

```python
# 创建贝尔态
psi0 = bell_state('00')

# 每个量子比特的局域退相位
gamma = 0.1
c_ops = [
    np.sqrt(gamma) * tensor(sigmaz(), qeye(2)),
    np.sqrt(gamma) * tensor(qeye(2), sigmaz())
]

# 追踪纠缠
def compute_concurrence(t, psi):
    rho = ket2dm(psi) if psi.isket else psi
    return concurrence(rho)

tlist = np.linspace(0, 10, 100)
result = mesolve(qeye([2, 2]), psi0, tlist, c_ops)

# 计算各态纠缠度
C_t = [concurrence(state.proj()) for state in result.states]

plt.plot(tlist, C_t)
plt.xlabel('时间')
plt.ylabel('纠缠度')
plt.title('纠缠衰减')
plt.show()
```

### Jaynes-Cummings模型

```python
# 系统参数
N = 10  # 腔福克空间
wc = 1.0  # 腔频率
wa = 1.0  # 原子频率
g = 0.05  # 耦合强度

# 算符
a = tensor(destroy(N), qeye(2))  # 腔场
sm = tensor(qeye(N), sigmam())  # 原子

# 哈密顿量（RWA）
H = wc * a.dag() * a + wa * sm.dag() * sm + g * (a.dag() * sm + a * sm.dag())

# 初始态：腔相干态，原子基态
psi0 = tensor(coherent(N, 2), basis(2, 0))

# 耗散
kappa = 0.1  # 腔衰减
gamma = 0.05  # 原子衰减
c_ops = [np.sqrt(kappa) * a, np.sqrt(gamma) * sm]

# 可观测量
n_cav = a.dag() * a
n_atom = sm.dag() * sm

# 演化
tlist = np.linspace(0, 50, 200)
result = mesolve(H, psi0, tlist, c_ops, e_ops=[n_cav, n_atom])

# 绘图
fig, axes = plt.subplots(2, 1, figsize=(8, 6), sharex=True)
axes[0].plot(tlist, result.expect[0])
axes[0].set_ylabel('⟨n_cavity⟩')
axes[1].plot(tlist, result.expect[1])
axes[1].set_ylabel('⟨n_atom⟩')
axes[1].set_xlabel('时间')
plt.tight_layout()
plt.show()
```

## 高效模拟技巧

1. **截断希尔伯特空间**：使用能捕捉动力学的最小维度
2. **选择合适求解器**：纯态用`sesolve`比`mesolve`更快
3. **含时项处理**：字符串格式（如`'cos(w*t)'`）速度最快
4. **仅存储必要数据**：用`e_ops`替代保存全部态
5. **调整容差**：通过`Options`平衡精度与计算时间
6. **并行轨迹**：`mcsolve`自动使用多CPU
7. **验证收敛性**：调整`ntraj`、希尔伯特空间大小和容差

## 故障排除

**内存问题**：降低希尔伯特空间维度，使用`store_final_state`选项，或尝试Krylov方法

**模拟缓慢**：使用基于字符串的含时项，略微增大容差，或对刚性问题尝试`method='bdf'`

**数值不稳定**：减小时间步长（`nsteps`选项），增大容差，或检查哈密顿量/算符定义

**导入错误**：确保正确安装QuTiP；量子门操作需`qutip-qip`包

## 参考文档

本技能包含详细参考文档：

- **`references/core_concepts.md`**：量子对象、态、算符、张量积、复合系统
- **`references/time_evolution.md`**：所有求解器（sesolve/mesolve/mcsolve/brmesolve等）、含时哈密顿量、求解器选项
- **`references/visualization.md`**：布洛赫球、维格纳函数、Q函数、福克分布、矩阵绘图
- **`references/analysis.md`**：期望值、熵、保真度、纠缠度量、关联函数、稳态
- **`references/advanced.md`**：Floquet理论、HEOM、置换不变性、随机方法、超算符、性能优化

## 外部资源

- 文档：https://qutip.readthedocs.io/
- 教程：https://qutip.org/qutip-tutorials/
- API参考：https://qutip.readthedocs.io/en/stable/apidoc/apidoc.html
- GitHub：https://github.com/qutip/qutip
