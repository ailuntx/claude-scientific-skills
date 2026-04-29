# QuTiP 分析与测量

## 期望值

### 基本期望值

```python
from qutip import *
import numpy as np

# 单算符
psi = coherent(N, 2)
n_avg = expect(num(N), psi)

# 多算符
ops = [num(N), destroy(N), create(N)]
results = expect(ops, psi)  # 返回列表
```

### 密度矩阵的期望值

```python
# 适用于纯态和密度矩阵
rho = thermal_dm(N, 2)
n_avg = expect(num(N), rho)
```

### 方差

```python
# 计算可观测量方差
var_n = variance(num(N), psi)

# 手动计算
var_n = expect(num(N)**2, psi) - expect(num(N), psi)**2
```

### 含时期望值

```python
# 时间演化过程中
result = mesolve(H, psi0, tlist, c_ops, e_ops=[num(N)])
n_t = result.expect[0]  # 各时刻 ⟨n⟩ 的数组
```

## 熵度量

### 冯·诺依曼熵

```python
from qutip import entropy_vn

# 密度矩阵熵
rho = thermal_dm(N, 2)
S = entropy_vn(rho)  # 返回 S = -Tr(ρ log₂ ρ)
```

### 线性熵

```python
from qutip import entropy_linear

# 线性熵 S_L = 1 - Tr(ρ²)
S_L = entropy_linear(rho)
```

### 纠缠熵

```python
# 二分系统
psi = bell_state('00')
rho = psi.proj()

# 对子系统B求迹得约化密度矩阵
rho_A = ptrace(rho, 0)

# 纠缠熵
S_ent = entropy_vn(rho_A)
```

### 互信息

```python
from qutip import entropy_mutual

# 二分态 ρ_AB
I = entropy_mutual(rho, [0, 1])  # I(A:B) = S(A) + S(B) - S(AB)
```

### 条件熵

```python
from qutip import entropy_conditional

# S(A|B) = S(AB) - S(B)
S_cond = entropy_conditional(rho, 0)  # 给定子系统1时子系统0的熵
```

## 保真度与距离度量

### 态保真度

```python
from qutip import fidelity

# 两态间保真度
psi1 = coherent(N, 2)
psi2 = coherent(N, 2.1)

F = fidelity(psi1, psi2)  # 返回 [0,1] 区间值
```

### 过程保真度

```python
from qutip import process_fidelity

# 两过程（超算符）间保真度
U_ideal = (-1j * H * t).expm()
U_actual = mesolve(H, basis(N, 0), [0, t], c_ops).states[-1]

F_proc = process_fidelity(U_ideal, U_actual)
```

### 迹距离

```python
from qutip import tracedist

# 迹距离 D = (1/2) Tr|ρ₁ - ρ₂|
rho1 = coherent_dm(N, 2)
rho2 = thermal_dm(N, 2)

D = tracedist(rho1, rho2)  # 返回 [0,1] 区间值
```

### 希尔伯特-施密特距离

```python
from qutip import hilbert_dist

# 希尔伯特-施密特距离
D_HS = hilbert_dist(rho1, rho2)
```

### Bures距离

```python
from qutip import bures_dist

# Bures距离
D_B = bures_dist(rho1, rho2)
```

### Bures角

```python
from qutip import bures_angle

# Bures角
angle = bures_angle(rho1, rho2)
```

## 纠缠度量

### 并发度

```python
from qutip import concurrence

# 双量子比特态
psi = bell_state('00')
rho = psi.proj()

C = concurrence(rho)  # 最大纠缠态时 C=1
```

### 负度

```python
from qutip import negativity

# 负度（偏转置判据）
N_ent = negativity(rho, 0)  # 对子系统0偏转置

# 对数负度
from qutip import logarithmic_negativity
E_N = logarithmic_negativity(rho, 0)
```

### 纠缠能力

```python
from qutip import entangling_power

# 酉门纠缠能力
U = cnot()
E_pow = entangling_power(U)
```

## 纯度度量

### 纯度

```python
# 纯度 P = Tr(ρ²)
P = (rho * rho).tr()

# 纯态时: P = 1
# 最大混合态: P = 1/d
```

### 态属性检查

```python
# 是否为纯态?
is_pure = abs((rho * rho).tr() - 1.0) < 1e-10

# 算符是否厄米?
H.isherm

# 算符是否酉?
U.check_isunitary()
```

## 测量

### 投影测量

```python
from qutip import measurement

# 计算基测量
psi = (basis(2, 0) + basis(2, 1)).unit()

# 执行测量
result, state_after = measurement.measure(psi, None)  # 随机结果

# 特定测量算符
M = basis(2, 0).proj()
prob = measurement.measure_povm(psi, [M, qeye(2) - M])
```

### 测量统计

```python
from qutip import measurement_statistics

# 获取所有可能结果及概率
outcomes, probabilities = measurement_statistics(psi, [M0, M1])
```

### 可观测量测量

```python
from qutip import measure_observable

# 测量可观测量并获取结果+坍缩态
result, state_collapsed = measure_observable(psi, sigmaz())
```

### POVM测量

```python
from qutip import measure_povm

# 正算子值测度
E_0 = Qobj([[0.8, 0], [0, 0.2]])
E_1 = Qobj([[0.2, 0], [0, 0.8]])

result, state_after = measure_povm(psi, [E_0, E_1])
```

## 相干性度量

### l1范数相干性

```python
from qutip import coherence_l1norm

# 非对角元素的l1范数
C_l1 = coherence_l1norm(rho)
```

## 关联函数

### 双时关联

```python
from qutip import correlation_2op_1t, correlation_2op_2t

# 单时关联 ⟨A(t+τ)B(t)⟩
A = destroy(N)
B = create(N)
taulist = np.linspace(0, 10, 200)

corr = correlation_2op_1t(H, rho0, taulist, c_ops, A, B)

# 双时关联 ⟨A(t)B(τ)⟩
tlist = np.linspace(0, 10, 100)
corr_2t = correlation_2op_2t(H, rho0, tlist, taulist, c_ops, A, B)
```

### 三算符关联

```python
from qutip import correlation_3op_1t

# ⟨A(t)B(t+τ)C(t)⟩
C_op = num(N)
corr_3 = correlation_3op_1t(H, rho0, taulist, c_ops, A, B, C_op)
```

### 四算符关联

```python
from qutip import correlation_4op_1t

# ⟨A(0)B(τ)C(τ)D(0)⟩
D_op = create(N)
corr_4 = correlation_4op_1t(H, rho0, taulist, c_ops, A, B, C_op, D_op)
```

## 谱分析

### FFT谱

```python
from qutip import spectrum_correlation_fft

# 通过关联函数计算功率谱
w, S = spectrum_correlation_fft(taulist, corr)
```

### 直接谱计算

```python
from qutip import spectrum

# 发射/吸收谱
wlist = np.linspace(0, 2, 200)
spec = spectrum(H, wlist, c_ops, A, B)
```

### 赝模方法

```python
from qutip import spectrum_pi

# 赝模分解谱
spec_pi = spectrum_pi(H, rho0, wlist, c_ops, A, B)
```

## 稳态分析

### 求解稳态

```python
from qutip import steadystate

# 求解 ∂ρ/∂t = 0 的稳态
rho_ss = steadystate(H, c_ops)

# 不同方法
rho_ss = steadystate(H, c_ops, method='direct')  # 默认
rho_ss = steadystate(H, c_ops, method='eigen')   # 本征值法
rho_ss = steadystate(H, c_ops, method='svd')     # SVD法
rho_ss = steadystate(H, c_ops, method='power')   # 幂法
```

### 稳态属性

```python
# 验证稳态
L = liouvillian(H, c_ops)
assert (L * operator_to_vector(rho_ss)).norm() < 1e-10

# 计算稳态期望值
n_ss = expect(num(N), rho_ss)
```

## 量子Fisher信息

```python
from qutip import qfisher

# 量子Fisher信息
F_Q = qfisher(rho, num(N))  # 关于生成元 num(N)
```

## 矩阵分析

### 本征分析

```python
# 本征值和本征矢
evals, ekets = H.eigenstates()

# 仅本征值
evals = H.eigenenergies()

# 基态
E0, psi0 = H.groundstate()
```

### 矩阵函数

```python
# 矩阵指数
U = (H * t).expm()

# 矩阵对数
log_rho = rho.logm()

# 矩阵平方根
sqrt_rho = rho.sqrtm()

# 矩阵幂
rho_squared = rho ** 2
```

### 奇异值分解

```python
# 算符的SVD
U, S, Vh = H.svd()
```

### 置换操作

```python
from qutip import permute

# 子系统置换
rho_permuted = permute(rho, [1, 0])  # 交换子系统
```

## 部分操作

### 部分迹

```python
# 约化到子系统
rho_A = ptrace(rho_AB, 0)  # 保留子系统0
rho_B = ptrace(rho_AB, 1)  # 保留子系统1

# 保留多个子系统
rho_AC = ptrace(rho_ABC, [0, 2])  # 保留0和2，对1求迹
```

### 部分转置

```python
from qutip import partial_transpose

# 部分转置（用于纠缠检测）
rho_pt = partial_transpose(rho, [0, 1])  # 转置子系统0

# 检测纠缠（PPT判据）
evals = rho_pt.eigenenergies()
is_entangled = any(evals < -1e-10)
```

## 量子态层析

### 态重构

```python
from qutip_qip.tomography import state_tomography

# 准备测量结果
# measurements = ... (实验数据)

# 重构密度矩阵
rho_reconstructed = state_tomography(measurements, basis='Pauli')
```

### 过程层析

```python
from qutip_qip.tomography import qpt

# 表征量子过程
chi = qpt(U_gate, method='lstsq')  # Chi矩阵表示
```

## 随机量子对象

用于测试和蒙特卡洛模拟。

```python
# 随机态矢量
psi_rand = rand_ket(N)

# 随机密度矩阵
rho_rand = rand_dm(N)

# 随机厄米算符
H_rand = rand_herm(N)

# 随机酉算符
U_rand = rand_unitary(N)

# 特定属性
rho_rank2 = rand_dm(N, rank=2)  # 秩2密度矩阵
H_sparse = rand_herm(N, density=0.1)  # 10%非零元素
```

## 实用检查

```python
# 检查算符是否厄米
H.isherm

# 检查态是否归一化
abs(psi.norm() - 1.0) < 1e-10

# 检查密度矩阵是否物理
rho.tr() ≈ 1 and all(rho.eigenenergies() >= 0)

# 检查算符是否对易
commutator(A, B).norm() < 1e-10
```
