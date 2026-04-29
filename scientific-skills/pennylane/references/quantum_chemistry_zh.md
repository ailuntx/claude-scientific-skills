# 使用 PennyLane 进行量子化学计算

## 目录
1. [分子哈密顿量](#分子哈密顿量)
2. [变分量子本征求解器 (VQE)](#变分量子本征求解器-vqe)
3. [分子结构](#分子结构)
4. [基组与映射](#基组与映射)
5. [激发态](#激发态)
6. [量子化学工作流](#量子化学工作流)

## 分子哈密顿量

### 构建分子哈密顿量

```python
import pennylane as qml
from pennylane import qchem
import numpy as np

# 定义分子
symbols = ['H', 'H']
coordinates = np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.74])  # 埃

# 生成哈密顿量
hamiltonian, n_qubits = qchem.molecular_hamiltonian(
    symbols,
    coordinates,
    charge=0,
    mult=1,  # 自旋多重度
    basis='sto-3g',
    method='dhf'  # Dirac-Hartree-Fock
)

print(f"哈密顿量: {hamiltonian}")
print(f"所需量子比特数: {n_qubits}")
```

### Jordan-Wigner 变换

```python
# 哈密顿量通过 Jordan-Wigner 自动转换为量子比特形式
# 手动转换:
from pennylane import fermi

# 费米子算符
a_0 = fermi.FermiC(0)  # 产生算符
a_1 = fermi.FermiA(1)  # 湮灭算符

# 转换为量子比特
qubit_op = qml.qchem.jordan_wigner(a_0 * a_1)
```

### Bravyi-Kitaev 变换

```python
# 替代映射（对某些系统更高效）
from pennylane.qchem import bravyi_kitaev

# 使用 Bravyi-Kitaev 构建哈密顿量
hamiltonian, n_qubits = qchem.molecular_hamiltonian(
    symbols,
    coordinates,
    mapping='bravyi_kitaev'
)
```

### 自定义哈密顿量

```python
# 从系数和算符构建哈密顿量
coeffs = [0.2, -0.8, 0.5]
obs = [
    qml.PauliZ(0),
    qml.PauliZ(0) @ qml.PauliZ(1),
    qml.PauliX(0) @ qml.PauliX(1)
]

H = qml.Hamiltonian(coeffs, obs)

# 或使用简化语法
H = 0.2 * qml.PauliZ(0) - 0.8 * qml.PauliZ(0) @ qml.PauliZ(1) + 0.5 * qml.PauliX(0) @ qml.PauliX(1)
```

## 变分量子本征求解器 (VQE)

### 基础 VQE 实现

```python
from pennylane import numpy as np

# 定义设备
dev = qml.device('default.qubit', wires=n_qubits)

# Hartree-Fock 态制备
hf_state = qchem.hf_state(electrons=2, orbitals=n_qubits)

def ansatz(params, wires):
    """变分拟设"""
    qml.BasisState(hf_state, wires=wires)

    for i in range(len(wires)):
        qml.RY(params[i], wires=i)

    for i in range(len(wires)-1):
        qml.CNOT(wires=[i, i+1])

@qml.qnode(dev)
def vqe_circuit(params):
    ansatz(params, wires=range(n_qubits))
    return qml.expval(hamiltonian)

# 优化
opt = qml.GradientDescentOptimizer(stepsize=0.4)
params = np.random.normal(0, np.pi, n_qubits, requires_grad=True)

for n in range(100):
    params, energy = opt.step_and_cost(vqe_circuit, params)

    if n % 20 == 0:
        print(f"步骤 {n}: 能量 = {energy:.8f} Ha")

print(f"最终基态能量: {energy:.8f} Ha")
```

### UCCSD 拟设

```python
from pennylane.qchem import UCCSD

# 单激发和双激发
singles, doubles = qchem.excitations(electrons=2, orbitals=n_qubits)

@qml.qnode(dev)
def uccsd_circuit(params):
    # Hartree-Fock 参考态
    qml.BasisState(hf_state, wires=range(n_qubits))

    # UCCSD 拟设
    UCCSD(params, wires=range(n_qubits), s_wires=singles, d_wires=doubles)

    return qml.expval(hamiltonian)

# 初始化参数
n_params = len(singles) + len(doubles)
params = np.zeros(n_params, requires_grad=True)

# 优化
opt = qml.AdamOptimizer(stepsize=0.1)
for n in range(100):
    params, energy = opt.step_and_cost(uccsd_circuit, params)
```

### 自适应 VQE

```python
def adaptive_vqe(hamiltonian, n_qubits, max_gates=10):
    """自适应 VQE：迭代扩展拟设"""
    dev = qml.device('default.qubit', wires=n_qubits)

    # 从 HF 态开始
    operations = []
    params = []

    hf_state = qchem.hf_state(electrons=2, orbitals=n_qubits)

    @qml.qnode(dev)
    def circuit(p):
        qml.BasisState(hf_state, wires=range(n_qubits))

        for op, param in zip(operations, p):
            op(param)

        return qml.expval(hamiltonian)

    # 迭代添加量子门
    for _ in range(max_gates):
        # 寻找最佳添加门
        best_op = None
        best_improvement = 0

        for candidate_op in generate_candidates():
            # 测试添加此操作
            test_ops = operations + [candidate_op]
            test_params = params + [0.0]

            improvement = evaluate_improvement(test_ops, test_params)

            if improvement > best_improvement:
                best_improvement = improvement
                best_op = candidate_op

        if best_improvement < threshold:
            break

        operations.append(best_op)
        params.append(0.0)

        # 优化当前拟设
        opt = qml.AdamOptimizer(stepsize=0.1)
        for _ in range(50):
            params = opt.step(circuit, params)

    return circuit, params
```

## 分子结构

### 定义分子

```python
# 简单双原子分子
h2_symbols = ['H', 'H']
h2_coords = np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.74])

# 水分子
h2o_symbols = ['O', 'H', 'H']
h2o_coords = np.array([
    0.0, 0.0, 0.0,      # O
    0.757, 0.586, 0.0,  # H
   -0.757, 0.586, 0.0   # H
])

# 从 XYZ 格式读取
molecule = qchem.read_structure('molecule.xyz')
symbols, coords = molecule
```

### 几何优化

```python
def optimize_geometry(symbols, initial_coords, basis='sto-3g'):
    """优化分子几何结构"""

    def energy_surface(coords):
        H, n_qubits = qchem.molecular_hamiltonian(
            symbols, coords, basis=basis
        )

        # 运行 VQE 获取能量
        energy = run_vqe(H, n_qubits)
        return energy

    # 经典核坐标优化
    from scipy.optimize import minimize

    result = minimize(
        energy_surface,
        initial_coords,
        method='BFGS',
        options={'gtol': 1e-5}
    )

    return result.x, result.fun

optimized_coords, min_energy = optimize_geometry(h2_symbols, h2_coords)
print(f"优化后几何结构: {optimized_coords}")
print(f"能量: {min_energy} Ha")
```

### 键解离曲线

```python
def dissociation_curve(symbols, axis=2, distances=None):
    """计算势能面"""

    if distances is None:
        distances = np.linspace(0.5, 3.0, 20)

    energies = []

    for d in distances:
        coords = np.zeros(6)
        coords[axis] = d  # 设置键长

        H, n_qubits = qchem.molecular_hamiltonian(
            symbols, coords, basis='sto-3g'
        )

        energy = run_vqe(H, n_qubits)
        energies.append(energy)

        print(f"距离: {d:.2f} Å, 能量: {energy:.6f} Ha")

    return distances, energies

# H2 解离曲线
distances, energies = dissociation_curve(['H', 'H'])

import matplotlib.pyplot as plt
plt.plot(distances, energies)
plt.xlabel('键长 (Å)')
plt.ylabel('能量 (Ha)')
plt.title('H2 解离曲线')
plt.show()
```

## 基组与映射

### 基组选择

```python
# 最小基组（最快，精度最低）
H_sto3g, n_qubits = qchem.molecular_hamiltonian(
    symbols, coords, basis='sto-3g'
)

# 双ζ基组
H_631g, n_qubits = qchem.molecular_hamiltonian(
    symbols, coords, basis='6-31g'
)

# 大基组（较慢，精度更高）
H_ccpvdz, n_qubits = qchem.molecular_hamiltonian(
    symbols, coords, basis='cc-pvdz'
)
```

### 活性空间选择

```python
# 选择活性轨道
active_electrons = 2
active_orbitals = 2

H_active, n_qubits = qchem.molecular_hamiltonian(
    symbols,
    coords,
    active_electrons=active_electrons,
    active_orbitals=active_orbitals
)

print(f"完整系统: {len(symbols)} 个电子")
print(f"活性空间: {active_electrons} 个电子在 {active_orbitals} 个轨道")
print(f"所需量子比特数: {n_qubits}")
```

### 费米子到量子比特的映射

```python
# Jordan-Wigner（默认）
H_jw, n_q_jw = qchem.molecular_hamiltonian(
    symbols, coords, mapping='jordan_wigner'
)

# Bravyi-Kitaev
H_bk, n_q_bk = qchem.molecular_hamiltonian(
    symbols, coords, mapping='bravyi_kitaev'
)

# 宇称映射
H_parity, n_q_parity = qchem.molecular_hamiltonian(
    symbols, coords, mapping='parity'
)

print(f"Jordan-Wigner 项数: {len(H_jw.ops)}")
print(f"Bravyi-Kitaev 项数: {len(H_bk.ops)}")
```

## 激发态

### 量子子空间展开

```python
def quantum_subspace_expansion(hamiltonian, ground_state_params, excitations):
    """通过子空间展开计算激发态"""

    @qml.qnode(dev)
    def ground_state():
        ansatz(ground_state_params, wires=range(n_qubits))
        return qml.state()

    # 获取基态
    psi_0 = ground_state()

    # 生成激发态基矢
    basis = [psi_0]

    for exc in excitations:
        @qml.qnode(dev)
        def excited_state():
            ansatz(ground_state_params, wires=range(n_qubits))
            # 应用激发
            apply_excitation(exc)
            return qml.state()

        psi_exc = excited_state()
        basis.append(psi_exc)

    # 在子空间中构建哈密顿矩阵
    n_basis = len(basis)
    H_matrix = np.zeros((n_basis, n_basis))

    for i in range(n_basis):
        for j in range(n_basis):
            H_matrix[i, j] = np.vdot(basis[i], hamiltonian @ basis[j])

    # 对角化
    eigenvalues, eigenvectors = np.linalg.eigh(H_matrix)

    return eigenvalues, eigenvectors
```

### SSVQE（子空间搜索 VQE）

```python
def ssvqe(hamiltonian, n_states=3):
    """同时计算多个态"""

    def cost_function(params):
        states = []

        for i in range(n_states):
            @qml.qnode(dev)
            def state_i():
                ansatz(params[i], wires=range(n_qubits))
                return qml.state()

            states.append(state_i())

        # 能量期望值
        energies = [np.vdot(s, hamiltonian @ s) for s in states]

        # 正交性惩罚项
        penalty = 0
        for i in range(n_states):
            for j in range(i+1, n_states):
                overlap = np.abs(np.vdot(states[i], states[j]))
                penalty += overlap ** 2

        return sum(energies) + 1000 * penalty

    # 初始化所有态的参数
    params = [np.random.random(n_params) for _ in range(n_states)]

    opt = qml.AdamOptimizer(stepsize=0.01)
    for _ in range(100):
        params = opt.step(cost_function, params)

    return params
```

## 量子化学工作流

### 完整 VQE 工作流

```python
def full_chemistry_workflow(symbols, coords, basis='sto-3g'):
    """完整的量子化学计算"""

    print("1. 构建分子哈密顿量...")
    H, n_qubits = qchem.molecular_hamiltonian(
        symbols, coords, basis=basis
    )

    print(f"   分子: {' '.join(symbols)}")
    print(f"   量子比特数: {n_qubits}")
    print(f"   哈密顿量项数: {len(H.ops)}")

    print("\n2. 准备 Hartree-Fock 态...")
    n_electrons = sum(qchem.atomic_numbers[s] for s in symbols)
    hf_state = qchem.hf_state(n_electrons, n_qubits)

    print("\n3. 运行 VQE...")
    energy, params = run_vqe(H, n_qubits, hf_state)

    print(f"\n4. 结果:")
    print(f"   基态能量: {energy:.8f} Ha")

    print("\n5. 计算性质...")
    dipole = compute_dipole_moment(symbols, coords, params)
    print(f"   偶极矩: {dipole:.4f} D")

    return {
        'energy': energy,
        'params': params,
        'dipole': dipole
    }

results = full_chemistry_workflow(['H', 'H'], h2_coords)
```

### 分子性质计算

```python
def compute_molecular_properties(symbols, coords, vqe_params):
    """从 VQE 解计算分子性质"""

    # 能量
    H, n_qubits = qchem.molecular_hamiltonian(symbols, coords)
    energy = vqe_circuit(vqe_params)

    # 偶极矩
    dipole_obs = qchem.dipole_moment(symbols, coords)

    @qml.qnode(dev)
    def dipole_circuit(axis):
        ansatz(vqe_params, wires=range(n_qubits))
        return qml.expval(dipole_obs[axis])

    dipole = [dipole_circuit(i) for i in range(3)]
    dipole_magnitude = np.linalg.norm(dipole)

    # 粒子数（完整性检查）
    @qml.qnode(dev)
    def particle_number():
        ansatz(vqe_params, wires=range(n_qubits))
        N_op = qchem.particle_number(n_qubits)
        return qml.expval(N_op)

    n_particles = particle_number()

    return {
        'energy': energy,
        'dipole_moment': dipole_magnitude,
        'dipole_vector': dipole,
        'particle_number': n_particles
    }
```

### 反应能计算

```python
def reaction_energy(reactants, products):
    """计算化学反应能量"""
```

# 计算反应物的能量
    E_reactants = 0
    for molecule in reactants:
        symbols, coords = molecule
        H, n_qubits = qchem.molecular_hamiltonian(symbols, coords)
        E_reactants += run_vqe(H, n_qubits)

    # 计算产物的能量
    E_products = 0
    for molecule in products:
        symbols, coords = molecule
        H, n_qubits = qchem.molecular_hamiltonian(symbols, coords)
        E_products += run_vqe(H, n_qubits)

    # 反应能量
    delta_E = E_products - E_reactants

    print(f"反应物能量: {E_reactants:.6f} Ha")
    print(f"产物能量: {E_products:.6f} Ha")
    print(f"反应能量: {delta_E:.6f} Ha ({delta_E * 627.5:.2f} kcal/mol)")

    return delta_E

# 示例：H2解离
reactants = [((['H', 'H'], h2_coords_bonded))]
products = [((['H'], [0, 0, 0]), (['H'], [10, 0, 0]))]  # 分离的原子

delta_E = reaction_energy(reactants, products)
```

## 最佳实践

1. **从小基组开始** - 测试时使用STO-3G，生产环境升级
2. **使用活性空间** - 通过选择相关轨道减少量子比特数
3. **选择合适的映射** - Bravyi-Kitaev通常能减少电路深度
4. **用HF初始化** - 从Hartree-Fock态开始VQE
5. **验证结果** - 与经典方法（FCI, CCSD）比较
6. **考虑对称性** - 利用分子对称性降低复杂度
7. **为精确性使用UCCSD** - UCCSD拟设具有化学动机
8. **监控收敛** - 检查梯度范数和能量方差
9. **考虑关联效应** - 确保拟设能捕捉电子关联
10. **全面基准测试** - 在已知系统上测试后再用于新分子
