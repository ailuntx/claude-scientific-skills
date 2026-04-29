# 量子算法与应用

Qiskit 支持广泛的量子算法，涵盖优化、化学、机器学习和物理模拟等领域。

## 目录

1. [优化算法](#optimization-algorithms)
2. [化学与材料科学](#chemistry-and-materials-science)
3. [机器学习](#machine-learning)
4. [算法库](#algorithm-libraries)

## 优化算法

### 变分量子本征求解器 (VQE)

VQE 采用混合量子-经典方法寻找哈密顿量的最小本征值。

**应用场景:**
- 分子基态能量计算
- 组合优化问题
- 材料模拟

**实现:**
```python
from qiskit import QuantumCircuit, transpile
from qiskit_ibm_runtime import QiskitRuntimeService, EstimatorV2 as Estimator, Session
from qiskit.quantum_info import SparsePauliOp
from scipy.optimize import minimize
import numpy as np

def vqe_algorithm(hamiltonian, ansatz, backend, initial_params):
    """
    运行 VQE 算法

    参数:
        hamiltonian: 可观测量 (SparsePauliOp)
        ansatz: 参数化量子电路
        backend: 量子后端
        initial_params: 初始参数值
    """

    with Session(backend=backend) as session:
        estimator = Estimator(session=session)

        def cost_function(params):
            # 将参数绑定到电路
            bound_circuit = ansatz.assign_parameters(params)

            # 为硬件进行转译
            qc_isa = transpile(bound_circuit, backend=backend, optimization_level=3)

            # 计算期望值
            job = estimator.run([(qc_isa, hamiltonian)])
            result = job.result()
            energy = result[0].data.evs

            return energy

        # 经典优化
        result = minimize(
            cost_function,
            initial_params,
            method='COBYLA',
            options={'maxiter': 100}
        )

    return result.fun, result.x

# 示例：H2 分子哈密顿量
hamiltonian = SparsePauliOp(
    ["IIII", "ZZII", "IIZZ", "ZZZI", "IZZI"],
    coeffs=[-0.8, 0.17, 0.17, -0.24, 0.17]
)

# 创建 ansatz
qc = QuantumCircuit(4)
# ... 定义 ansatz 结构 ...

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

energy, params = vqe_algorithm(hamiltonian, qc, backend, np.random.rand(10))
print(f"基态能量: {energy}")
```

### 量子近似优化算法 (QAOA)

QAOA 解决组合优化问题，如 MaxCut、旅行商问题和图着色问题。

**应用场景:**
- MaxCut 问题
- 投资组合优化
- 车辆路径规划
- 调度问题

**实现:**
```python
from qiskit import QuantumCircuit
from qiskit.circuit import Parameter
import networkx as nx

def qaoa_maxcut(graph, p, backend):
    """
    QAOA 解决 MaxCut 问题

    参数:
        graph: NetworkX 图
        p: QAOA 层数
        backend: 量子后端
    """
    num_qubits = len(graph.nodes())
    qc = QuantumCircuit(num_qubits)

    # 初始叠加态
    qc.h(range(num_qubits))

    # 交替层
    betas = [Parameter(f'β_{i}') for i in range(p)]
    gammas = [Parameter(f'γ_{i}') for i in range(p)]

    for i in range(p):
        # 问题哈密顿量 (MaxCut)
        for edge in graph.edges():
            u, v = edge
            qc.cx(u, v)
            qc.rz(2 * gammas[i], v)
            qc.cx(u, v)

        # 混合哈密顿量
        for qubit in range(num_qubits):
            qc.rx(2 * betas[i], qubit)

    qc.measure_all()
    return qc, betas + gammas

# 示例：4节点图的 MaxCut
G = nx.Graph()
G.add_edges_from([(0, 1), (1, 2), (2, 3), (3, 0)])

qaoa_circuit, params = qaoa_maxcut(G, p=2, backend=backend)

# 使用 Sampler 运行并优化参数
# ... (类似 VQE 优化循环)
```

### Grover 算法

量子搜索算法，为无结构搜索提供二次加速。

**应用场景:**
- 数据库搜索
- SAT 问题求解
- 标记项查找

**实现:**
```python
from qiskit import QuantumCircuit

def grover_oracle(marked_states):
    """创建标记目标态的神谕"""
    num_qubits = len(marked_states[0])
    qc = QuantumCircuit(num_qubits)

    for target in marked_states:
        # 翻转目标态相位
        for i, bit in enumerate(target):
            if bit == '0':
                qc.x(i)

        # 多控制 Z 门
        qc.h(num_qubits - 1)
        qc.mcx(list(range(num_qubits - 1)), num_qubits - 1)
        qc.h(num_qubits - 1)

        for i, bit in enumerate(target):
            if bit == '0':
                qc.x(i)

    return qc

def grover_diffusion(num_qubits):
    """创建 Grover 扩散算子"""
    qc = QuantumCircuit(num_qubits)

    qc.h(range(num_qubits))
    qc.x(range(num_qubits))

    qc.h(num_qubits - 1)
    qc.mcx(list(range(num_qubits - 1)), num_qubits - 1)
    qc.h(num_qubits - 1)

    qc.x(range(num_qubits))
    qc.h(range(num_qubits))

    return qc

def grover_algorithm(marked_states, num_iterations):
    """完整 Grover 算法"""
    num_qubits = len(marked_states[0])
    qc = QuantumCircuit(num_qubits)

    # 初始化叠加态
    qc.h(range(num_qubits))

    # Grover 迭代
    oracle = grover_oracle(marked_states)
    diffusion = grover_diffusion(num_qubits)

    for _ in range(num_iterations):
        qc.compose(oracle, inplace=True)
        qc.compose(diffusion, inplace=True)

    qc.measure_all()
    return qc

# 在 3 量子比特空间搜索 |101⟩ 态
marked = ['101']
iterations = int(np.pi/4 * np.sqrt(2**3))  # 最优迭代次数
qc_grover = grover_algorithm(marked, iterations)
```

## 化学与材料科学

### 分子基态能量

**安装 Qiskit Nature:**
```bash
uv pip install qiskit-nature qiskit-nature-pyscf
```

**示例：H2 分子**
```python
from qiskit_nature.second_q.drivers import PySCFDriver
from qiskit_nature.second_q.mappers import JordanWignerMapper, ParityMapper
from qiskit_nature.second_q.circuit.library import UCCSD, HartreeFock

# 定义分子
driver = PySCFDriver(
    atom="H 0 0 0; H 0 0 0.735",
    basis="sto3g",
    charge=0,
    spin=0
)

# 获取电子结构问题
problem = driver.run()

# 将费米子算符映射到量子比特
mapper = JordanWignerMapper()
hamiltonian = mapper.map(problem.hamiltonian.second_q_op())

# 创建初始态
num_particles = problem.num_particles
num_spatial_orbitals = problem.num_spatial_orbitals

init_state = HartreeFock(
    num_spatial_orbitals,
    num_particles,
    mapper
)

# 创建 ansatz
ansatz = UCCSD(
    num_spatial_orbitals,
    num_particles,
    mapper,
    initial_state=init_state
)

# 运行 VQE
energy, params = vqe_algorithm(
    hamiltonian,
    ansatz,
    backend,
    np.zeros(ansatz.num_parameters)
)

# 添加核排斥能
total_energy = energy + problem.nuclear_repulsion_energy
print(f"基态能量: {total_energy} Ha")
```

### 不同分子映射方法

```python
# Jordan-Wigner 映射
jw_mapper = JordanWignerMapper()
ham_jw = jw_mapper.map(problem.hamiltonian.second_q_op())

# 宇称映射 (通常更高效)
parity_mapper = ParityMapper()
ham_parity = parity_mapper.map(problem.hamiltonian.second_q_op())

# Bravyi-Kitaev 映射
from qiskit_nature.second_q.mappers import BravyiKitaevMapper
bk_mapper = BravyiKitaevMapper()
ham_bk = bk_mapper.map(problem.hamiltonian.second_q_op())
```

### 激发态计算

```python
from qiskit_nature.second_q.algorithms import QEOM

# 激发态的量子运动方程
qeom = QEOM(estimator, ansatz, 'sd')  # 单双激发
excited_states = qeom.solve(problem)
```

## 机器学习

### 量子核方法

量子计算机可计算机器学习中的核函数。

**安装 Qiskit Machine Learning:**
```bash
uv pip install qiskit-machine-learning
```

**示例：量子核分类**
```python
from qiskit_machine_learning.kernels import FidelityQuantumKernel
from qiskit_algorithms.state_fidelities import ComputeUncompute
from qiskit.circuit.library import ZZFeatureMap
from sklearn.svm import SVC
import numpy as np

# 创建特征映射
num_features = 2
feature_map = ZZFeatureMap(feature_dimension=num_features, reps=2)

# 创建量子核
fidelity = ComputeUncompute(sampler=sampler)
qkernel = FidelityQuantumKernel(fidelity=fidelity, feature_map=feature_map)

# 与 scikit-learn 结合使用
X_train = np.random.rand(50, 2)
y_train = np.random.choice([0, 1], 50)

# 计算核矩阵
kernel_matrix = qkernel.evaluate(X_train)

# 训练量子核 SVM
svc = SVC(kernel='precomputed')
svc.fit(kernel_matrix, y_train)

# 预测
X_test = np.random.rand(10, 2)
kernel_test = qkernel.evaluate(X_test, X_train)
predictions = svc.predict(kernel_test)
```

### 变分量子分类器 (VQC)

```python
from qiskit_machine_learning.algorithms import VQC
from qiskit.circuit.library import RealAmplitudes

# 创建特征映射和 ansatz
feature_map = ZZFeatureMap(2)
ansatz = RealAmplitudes(2, reps=1)

# 创建 VQC
vqc = VQC(
    sampler=sampler,
    feature_map=feature_map,
    ansatz=ansatz,
    optimizer='COBYLA'
)

# 训练
vqc.fit(X_train, y_train)

# 预测
predictions = vqc.predict(X_test)
accuracy = vqc.score(X_test, y_test)
```

### 量子神经网络 (QNN)

```python
from qiskit_machine_learning.neural_networks import SamplerQNN
from qiskit.circuit import QuantumCircuit, Parameter

# 创建参数化电路
qc = QuantumCircuit(2)
params = [Parameter(f'θ_{i}') for i in range(4)]

# 网络结构
for i, param in enumerate(params[:2]):
    qc.ry(param, i)

qc.cx(0, 1)

for i, param in enumerate(params[2:]):
    qc.ry(param, i)

qc.measure_all()

# 创建 QNN
qnn = SamplerQNN(
    circuit=qc,
    sampler=sampler,
    input_params=[],  # 此示例无输入参数
    weight_params=params
)

# 可与 PyTorch 或 TensorFlow 结合训练
```

## 算法库

### Qiskit 算法库

量子算法的标准实现：

```bash
uv pip install qiskit-algorithms
```

**可用算法:**
- 振幅估计
- 相位估计
- Shor 算法
- 量子傅里叶变换
- HHL (线性方程组求解)

**示例：量子相位估计**
```python
from qiskit.circuit.library import QFT
from qiskit_algorithms import PhaseEstimation

# 创建酉算子
num_qubits = 3
unitary = QuantumCircuit(num_qubits)
# ... 定义酉算子 ...

# 相位估计
pe = PhaseEstimation(num_evaluation_qubits=3, quantum_instance=backend)
result = pe.estimate(unitary=unitary, state_preparation=initial_state)
```

### Qiskit 优化库

优化问题求解器：

```bash
uv pip install qiskit-optimization
```

**支持问题类型:**
- 二次规划
- 整数规划
- 线性规划
- 约束满足问题

**示例：投资

- [Qiskit 优化文档](https://qiskit.org/ecosystem/optimization)

**研究论文：**
- 变分量子本征求解器 (VQE)：Peruzzo 等人，《自然·通讯》(2014)
- 量子近似优化算法 (QAOA)：Farhi 等人，arXiv:1411.4028
- 量子核方法：Havlíček 等人，《自然》(2019)
