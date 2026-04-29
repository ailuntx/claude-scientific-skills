# QuTiP 核心概念

## 量子对象 (Qobj)

QuTiP 中的所有量子对象都由 `Qobj` 类表示：

```python
from qutip import *

# 创建量子对象
psi = basis(2, 0)  # 二能级系统的基态
rho = fock_dm(5, 2)  # n=2 Fock 态的密度矩阵
H = sigmaz()  # 泡利 Z 算符
```

关键属性：
- `.dims` - 维度结构
- `.shape` - 矩阵维度
- `.type` - 类型 (ket, bra, oper, super)
- `.isherm` - 检查是否厄米
- `.dag()` - 厄米共轭
- `.tr()` - 迹
- `.norm()` - 范数

## 量子态

### 基态

```python
# Fock（粒子数）态
n = 2  # 激发能级
N = 10  # 希尔伯特空间维度
psi = basis(N, n)  # 或 fock(N, n)

# 相干态
alpha = 1 + 1j
coherent(N, alpha)

# 热态（密度矩阵）
n_avg = 2.0  # 平均光子数
thermal_dm(N, n_avg)
```

### 自旋态

```python
# 自旋-1/2 态
spin_state(1/2, 1/2)  # 自旋上态
spin_coherent(1/2, theta, phi)  # 相干自旋态

# 多量子比特计算基
basis([2,2,2], [0,1,0])  # 三量子比特的 |010⟩
```

### 复合态

```python
# 张量积
psi1 = basis(2, 0)
psi2 = basis(2, 1)
tensor(psi1, psi2)  # |01⟩

# 贝尔态
bell_state('00')  # (|00⟩ + |11⟩)/√2
maximally_mixed_dm(2)  # 最大混合态
```

## 算符

### 产生/湮灭算符

```python
N = 10
a = destroy(N)  # 湮灭算符
a_dag = create(N)  # 产生算符
num = num(N)  # 粒子数算符 (a†a)
```

### 泡利矩阵

```python
sigmax()  # σx
sigmay()  # σy
sigmaz()  # σz
sigmap()  # σ+ = (σx + iσy)/2
sigmam()  # σ- = (σx - iσy)/2
```

### 角动量算符

```python
# 任意 j 的自旋算符
j = 1  # 自旋-1
jmat(j, 'x')  # Jx
jmat(j, 'y')  # Jy
jmat(j, 'z')  # Jz
jmat(j, '+')  # J+
jmat(j, '-')  # J-
```

### 平移与压缩算符

```python
alpha = 1 + 1j
displace(N, alpha)  # 平移算符 D(α)

z = 0.5  # 压缩参数
squeeze(N, z)  # 压缩算符 S(z)
```

## 张量积与复合系统

### 构建复合系统

```python
# 算符张量积
H1 = sigmaz()
H2 = sigmax()
H_total = tensor(H1, H2)

# 单位算符
qeye([2, 2])  # 双量子比特单位算符

# 部分应用
# 三量子比特系统的 σz ⊗ I
tensor(sigmaz(), qeye(2), qeye(2))
```

### 偏迹

```python
# 复合系统态
rho = bell_state('00').proj()  # |Φ+⟩⟨Φ+|

# 对子系统求迹
rho_A = ptrace(rho, 0)  # 对子系统 0 求迹
rho_B = ptrace(rho, 1)  # 对子系统 1 求迹
```

## 期望值与测量

```python
# 期望值
psi = coherent(N, alpha)
expect(num, psi)  # ⟨n⟩

# 多算符计算
ops = [a, a_dag, num]
expect(ops, psi)  # 返回列表

# 方差
variance(num, psi)  # Var(n) = ⟨n²⟩ - ⟨n⟩²
```

## 超算符与刘维尔算符

### Lindblad 形式

```python
# 系统哈密顿量
H = num

# 坍缩算符（耗散）
c_ops = [np.sqrt(0.1) * a]  # 衰减率 0.1

# 刘维尔超算符
L = liouvillian(H, c_ops)

# 显式形式
L = -1j * (spre(H) - spost(H)) + lindblad_dissipator(a, a)
```

### 超算符表示

```python
# Kraus 表示
kraus_to_super(kraus_ops)

# Choi 矩阵
choi_to_super(choi_matrix)

# Chi（过程）矩阵
chi_to_super(chi_matrix)

# 转换函数
super_to_choi(L)
choi_to_kraus(choi_matrix)
```

## 量子门 (需 qutip-qip)

```python
from qutip_qip.operations import *

# 单量子比特门
hadamard_transform()  # Hadamard
rx(np.pi/2)  # X 旋转
ry(np.pi/2)  # Y 旋转
rz(np.pi/2)  # Z 旋转
phasegate(np.pi/4)  # 相位门
snot()  # Hadamard (替代实现)

# 双量子比特门
cnot()  # CNOT
swap()  # SWAP
iswap()  # iSWAP
sqrtswap()  # √SWAP
berkeley()  # Berkeley 门
swapalpha(alpha)  # SWAP^α

# 三量子比特门
fredkin()  # 受控 SWAP
toffoli()  # 受控 CNOT

# 扩展到多量子比特系统
N = 3  # 总量子比特数
target = 1
controls = [0, 2]
gate_expand_2toN(cnot(), N, [controls[0], target])
```

## 常用哈密顿量

### Jaynes-Cummings 模型

```python
# 腔模
N = 10
a = tensor(destroy(N), qeye(2))

# 原子
sm = tensor(qeye(N), sigmam())

# 哈密顿量
wc = 1.0  # 腔频率
wa = 1.0  # 原子频率
g = 0.05  # 耦合强度
H = wc * a.dag() * a + wa * sm.dag() * sm + g * (a.dag() * sm + a * sm.dag())
```

### 驱动系统

```python
# 含时哈密顿量
H0 = sigmaz()
H1 = sigmax()

def drive(t, args):
    return np.sin(args['w'] * t)

H = [H0, [H1, drive]]
args = {'w': 1.0}
```

### 自旋链

```python
# 海森堡链
N_spins = 5
J = 1.0  # 交换耦合

# 构建哈密顿量
H = 0
for i in range(N_spins - 1):
    # σᵢˣσᵢ₊₁ˣ + σᵢʸσᵢ₊₁ʸ + σᵢᶻσᵢ₊₁ᶻ
    H += J * (
        tensor_at([sigmax()], i, N_spins) * tensor_at([sigmax()], i+1, N_spins) +
        tensor_at([sigmay()], i, N_spins) * tensor_at([sigmay()], i+1, N_spins) +
        tensor_at([sigmaz()], i, N_spins) * tensor_at([sigmaz()], i+1, N_spins)
    )
```

## 实用工具函数

```python
# 生成随机量子对象
rand_ket(N)  # 随机态矢
rand_dm(N)  # 随机密度矩阵
rand_herm(N)  # 随机厄米算符
rand_unitary(N)  # 随机酉算符

# 对易与反对易子
commutator(A, B)  # [A, B]
anti_commutator(A, B)  # {A, B}

# 矩阵指数
(-1j * H * t).expm()  # e^(-iHt)

# 本征值与本征态
H.eigenstates()  # 返回 (本征值, 本征态)
H.eigenenergies()  # 仅返回本征值
H.groundstate()  # 基态能量与态矢
```
