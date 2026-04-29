# 运行量子实验

本指南涵盖设计和执行量子实验，包括参数扫描、数据收集以及使用ReCirq框架。

## 实验设计

### 基本实验结构

```python
import cirq
import numpy as np
import pandas as pd

class QuantumExperiment:
    """量子实验基类"""

    def __init__(self, qubits, simulator=None):
        self.qubits = qubits
        self.simulator = simulator or cirq.Simulator()
        self.results = []

    def build_circuit(self, **params):
        """使用给定参数构建电路"""
        raise NotImplementedError

    def run(self, params_list, repetitions=1000):
        """执行参数扫描实验"""
        for params in params_list:
            circuit = self.build_circuit(**params)
            result = self.simulator.run(circuit, repetitions=repetitions)
            self.results.append({
                'params': params,
                'result': result
            })
        return self.results

    def analyze(self):
        """分析实验结果"""
        raise NotImplementedError
```

### 参数扫描

```python
import sympy

# 定义参数
theta = sympy.Symbol('theta')
phi = sympy.Symbol('phi')

# 创建参数化电路
def parameterized_circuit(qubits, theta, phi):
    return cirq.Circuit(
        cirq.ry(theta)(qubits[0]),
        cirq.rz(phi)(qubits[1]),
        cirq.CNOT(qubits[0], qubits[1]),
        cirq.measure(*qubits, key='result')
    )

# 定义扫描范围
sweep = cirq.Product(
    cirq.Linspace('theta', 0, np.pi, 20),
    cirq.Linspace('phi', 0, 2*np.pi, 20)
)

# 执行扫描
circuit = parameterized_circuit(cirq.LineQubit.range(2), theta, phi)
results = cirq.Simulator().run_sweep(circuit, params=sweep, repetitions=1000)
```

### 数据收集

```python
def collect_experiment_data(circuit, sweep, simulator, repetitions=1000):
    """收集并组织实验数据"""

    data = []
    results = simulator.run_sweep(circuit, params=sweep, repetitions=repetitions)

    for params, result in zip(sweep, results):
        # 提取参数
        param_dict = {k: v for k, v in params.param_dict.items()}

        # 提取测量结果
        counts = result.histogram(key='result')

        # 结构化存储
        data.append({
            **param_dict,
            'counts': counts,
            'total': repetitions
        })

    return pd.DataFrame(data)

# 收集数据
df = collect_experiment_data(circuit, sweep, cirq.Simulator())

# 保存到文件
df.to_csv('experiment_results.csv', index=False)
```

## ReCirq框架

ReCirq为可复现的量子实验提供结构化框架。

### ReCirq实验结构

```python
"""
标准ReCirq实验结构：

experiment_name/
├── __init__.py
├── experiment.py        # 主实验代码
├── tasks.py            # 数据生成任务
├── data_collection.py  # 并行数据收集
├── analysis.py         # 数据分析
└── plots.py           # 可视化
"""
```

### 基于任务的数据收集

```python
from dataclasses import dataclass
from typing import List
import cirq

@dataclass
class ExperimentTask:
    """参数扫描中的单个任务"""
    theta: float
    phi: float
    repetitions: int = 1000

    def build_circuit(self, qubits):
        """构建本任务电路"""
        return cirq.Circuit(
            cirq.ry(self.theta)(qubits[0]),
            cirq.rz(self.phi)(qubits[1]),
            cirq.CNOT(qubits[0], qubits[1]),
            cirq.measure(*qubits, key='result')
        )

    def run(self, qubits, simulator):
        """执行任务"""
        circuit = self.build_circuit(qubits)
        result = simulator.run(circuit, repetitions=self.repetitions)
        return {
            'theta': self.theta,
            'phi': self.phi,
            'result': result
        }

# 创建任务
tasks = [
    ExperimentTask(theta=t, phi=p)
    for t in np.linspace(0, np.pi, 10)
    for p in np.linspace(0, 2*np.pi, 10)
]

# 执行任务
qubits = cirq.LineQubit.range(2)
simulator = cirq.Simulator()
results = [task.run(qubits, simulator) for task in tasks]
```

### 并行数据收集

```python
from multiprocessing import Pool
import functools

def run_task_parallel(task, qubits, simulator):
    """执行单个任务（用于并行）"""
    return task.run(qubits, simulator)

def collect_data_parallel(tasks, qubits, simulator, n_workers=4):
    """使用并行处理收集数据"""

    # 创建固定参数的偏函数
    run_func = functools.partial(
        run_task_parallel,
        qubits=qubits,
        simulator=simulator
    )

    # 并行执行
    with Pool(n_workers) as pool:
        results = pool.map(run_func, tasks)

    return results

# 使用并行收集
results = collect_data_parallel(tasks, qubits, cirq.Simulator(), n_workers=8)
```

## 常见量子算法

### 变分量子本征求解器 (VQE)

```python
import scipy.optimize

def vqe_experiment(hamiltonian, ansatz_func, initial_params):
    """运行VQE寻找基态能量"""

    def cost_function(params):
        """能量期望值计算"""
        circuit = ansatz_func(params)

        # 测量哈密顿量期望值
        simulator = cirq.Simulator()
        result = simulator.simulate(circuit)
        energy = hamiltonian.expectation_from_state_vector(
            result.final_state_vector,
            qubit_map={q: i for i, q in enumerate(circuit.all_qubits())}
        )
        return energy.real

    # 优化参数
    result = scipy.optimize.minimize(
        cost_function,
        initial_params,
        method='COBYLA'
    )

    return result

# 示例：H2分子
def h2_ansatz(params, qubits):
    """H2的UCC拟设"""
    theta = params[0]
    return cirq.Circuit(
        cirq.X(qubits[1]),
        cirq.ry(theta)(qubits[0]),
        cirq.CNOT(qubits[0], qubits[1])
    )

# 定义哈密顿量（简化版）
qubits = cirq.LineQubit.range(2)
hamiltonian = cirq.PauliSum.from_pauli_strings([
    cirq.PauliString({qubits[0]: cirq.Z}),
    cirq.PauliString({qubits[1]: cirq.Z}),
    cirq.PauliString({qubits[0]: cirq.Z, qubits[1]: cirq.Z})
])

# 运行VQE
result = vqe_experiment(
    hamiltonian,
    lambda p: h2_ansatz(p, qubits),
    initial_params=[0.0]
)

print(f"基态能量: {result.fun}")
print(f"最优参数: {result.x}")
```

### 量子近似优化算法 (QAOA)

```python
def qaoa_circuit(graph, params, p_layers):
    """MaxCut问题的QAOA电路"""

    qubits = cirq.LineQubit.range(graph.number_of_nodes())
    circuit = cirq.Circuit()

    # 初始叠加态
    circuit.append(cirq.H(q) for q in qubits)

    # QAOA层
    for layer in range(p_layers):
        gamma = params[layer]
        beta = params[p_layers + layer]

        # 问题哈密顿量（代价）
        for edge in graph.edges():
            i, j = edge
            circuit.append(cirq.ZZPowGate(exponent=gamma)(qubits[i], qubits[j]))

        # 混合哈密顿量
        circuit.append(cirq.rx(2 * beta)(q) for q in qubits)

    circuit.append(cirq.measure(*qubits, key='result'))
    return circuit

# 运行QAOA
import networkx as nx

graph = nx.cycle_graph(4)
p_layers = 2

def qaoa_cost(params):
    """评估QAOA代价函数"""
    circuit = qaoa_circuit(graph, params, p_layers)
    simulator = cirq.Simulator()
    result = simulator.run(circuit, repetitions=1000)

    # 计算MaxCut目标值
    total_cost = 0
    counts = result.histogram(key='result')

    for bitstring, count in counts.items():
        cost = 0
        bits = [(bitstring >> i) & 1 for i in range(graph.number_of_nodes())]
        for edge in graph.edges():
            i, j = edge
            if bits[i] != bits[j]:
                cost += 1
        total_cost += cost * count

    return -total_cost / 1000  # 最大化切割数

# 优化
initial_params = np.random.random(2 * p_layers) * np.pi
result = scipy.optimize.minimize(qaoa_cost, initial_params, method='COBYLA')

print(f"最优代价: {-result.fun}")
print(f"最优参数: {result.x}")
```

### 量子相位估计

```python
def qpe_circuit(unitary, eigenstate_prep, n_counting_qubits):
    """量子相位估计电路"""

    counting_qubits = cirq.LineQubit.range(n_counting_qubits)
    target_qubit = cirq.LineQubit(n_counting_qubits)

    circuit = cirq.Circuit()

    # 准备本征态
    circuit.append(eigenstate_prep(target_qubit))

    # 计数比特应用Hadamard门
    circuit.append(cirq.H(q) for q in counting_qubits)

    # 受控酉操作
    for i, q in enumerate(counting_qubits):
        power = 2 ** (n_counting_qubits - 1 - i)
        # 应用受控U^power
        for _ in range(power):
            circuit.append(cirq.ControlledGate(unitary)(q, target_qubit))

    # 计数比特上应用逆QFT
    circuit.append(inverse_qft(counting_qubits))

    # 测量计数比特
    circuit.append(cirq.measure(*counting_qubits, key='phase'))

    return circuit

def inverse_qft(qubits):
    """逆量子傅里叶变换"""
    n = len(qubits)
    ops = []

    for i in range(n // 2):
        ops.append(cirq.SWAP(qubits[i], qubits[n - i - 1]))

    for i in range(n):
        for j in range(i):
            ops.append(cirq.CZPowGate(exponent=-1/2**(i-j))(qubits[j], qubits[i]))
        ops.append(cirq.H(qubits[i]))

    return ops
```

## 数据分析

### 统计分析

```python
def analyze_measurement_statistics(results):
    """分析测量统计特性"""

    counts = results.histogram(key='result')
    total = sum(counts.values())

    # 计算概率
    probabilities = {state: count/total for state, count in counts.items()}

    # 香农熵
    entropy = -sum(p * np.log2(p) for p in probabilities.values() if p > 0)

    # 最可能结果
    most_likely = max(counts.items(), key=lambda x: x[1])

    return {
        'probabilities': probabilities,
        'entropy': entropy,
        'most_likely_state': most_likely[0],
        'most_likely_probability': most_likely[1] / total
    }
```

### 期望值计算

```python
def calculate_expectation_value(circuit, observable, simulator):
    """计算可观测量期望值"""

    # 移除测量操作
    circuit_no_measure = cirq.Circuit(
        m for m in circuit if not isinstance(m, cirq.MeasurementGate)
    )

    result = simulator.simulate(circuit_no_measure)
    state_vector = result.final_state_vector

    # 计算⟨ψ|O|ψ⟩
    expectation = observable.expectation_from_state_vector(
        state_vector,
        qubit_map={q: i for i, q in enumerate(circuit.all_qubits())}
    )

    return expectation.real
```

### 保真度估计

```python
def state_fidelity(state1, state2):
    """计算两态间保真度"""
    return np.abs(np.vdot(state1, state2)) ** 2

def process_fidelity(result1, result2):
    """根据测量结果计算过程保真度"""

    counts1 = result1.histogram(key='result')
    counts2 = result2.histogram(key='result')

    # 归一化为概率
    total1 = sum(counts1.values())
    total2 = sum(counts2.values())

    probs1 = {k: v/total1 for k, v in counts1.items()}
    probs2 = {k: v/total2 for k, v in counts2.items()}

    # 经典保真度（Bhattacharyya系数）
    all_states = set(probs1.keys()) | set(probs2.keys())
    fidelity = sum(np.sqrt(probs1.get(s, 0) * probs2.get(s, 0))
                   for s in all_states) ** 2

    return fidelity
```

## 可视化

### 绘制参数空间

```python
import matplotlib.pyplot as plt

def plot_parameter_landscape(theta_vals, phi_vals, energies):
    """绘制二维参数空间"""

    plt.figure(figsize=(10, 8))
    plt.contourf(theta_vals, phi_vals, energies, levels=50, cmap='viridis')
    plt.colorbar(label='能量')
    plt.xlabel('θ')
    plt.ylabel('φ')
    plt.title('能量空间分布')
    plt.show()
```

### 绘制收敛曲线

```python
def plot_optimization_convergence(optimization_history):
    """绘制优化收敛曲线"""

    iterations = range(len(optimization_history))
    energies = [result['energy'] for result in optimization_history]

    plt.figure(figsize=(10, 6))
    plt.plot(iterations, energies, 'b-', linewidth=2)
    plt.xlabel('迭代次数')
    plt.ylabel('能量')
    plt.title('优化收敛过程')
    plt.grid(True)
    plt.show()
```

### 绘制测量分布

```python
def plot_measurement_distribution(results):
    """绘制测量结果分布"""

    counts = results.histogram(key='result')

    plt.figure(figsize=(12, 6))
    plt.bar(counts.keys(), counts.values())
    plt.xlabel('测量结果')
    plt.ylabel('计数')
    plt.title('测量分布')
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()
```

## 最佳实践

1. **清晰构建实验结构**：使用ReCirq模式确保可复现性
2. **分离任务**：划分数据生成、收集和分析阶段
3. **使用参数扫描**：系统化探索参数空间
4. **保存中间结果**：避免丢失昂贵计算结果
5. **并行化处理**：对独立任务使用多进程
6. **记录元数据**：保存实验条件、时间戳和版本信息
7. **在模拟器验证**：上硬件前测试实验代码
8. **实现错误处理**：长时运行实验需健壮代码
9. **数据版本控制**：实验数据与代码同步追踪
10. **完整文档记录**：确保可复现性的清晰文档

## 示例：完整实验流程

```python
# 完整实验工作流
class VQEExperiment(QuantumExperiment):
    """完整VQE实验"""

    def __init__(self, hamiltonian, ansatz, qubits):
        super().__init__(qubits)
        self.hamiltonian = hamiltonian
        self.ansatz = ansatz
        self.history = []

    def build_circuit(self, params):
        return self.ansatz(params, self.qubits)

    def cost_function(self, params):
        circuit = self.build_circuit(params)
        result = self.simulator.simulate(circuit)
        energy = self.hamiltonian.ex

```python
options={'maxiter': 100}
        )
        return result

    def analyze(self):
        # 绘制收敛曲线
        energies = [h['energy'] for h in self.history]
        plt.plot(energies)
        plt.xlabel('迭代次数')
        plt.ylabel('能量')
        plt.title('VQE 收敛图')
        plt.show()

        return {
            'final_energy': self.history[-1]['energy'],
            'optimal_params': self.history[-1]['params'],
            'num_iterations': len(self.history)
        }

# 运行实验
experiment = VQEExperiment(hamiltonian, h2_ansatz, qubits)
result = experiment.run(initial_params=[0.0])
analysis = experiment.analyze()
```
