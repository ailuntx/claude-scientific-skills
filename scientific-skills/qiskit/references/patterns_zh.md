# Qiskit 模式：四步工作流

Qiskit 模式提供了一个通用框架，通过四个阶段解决特定领域的量子计算问题：映射（Map）、优化（Optimize）、执行（Execute）和后处理（Post-process）。

## 概述

该模式框架支持量子能力的无缝组合，并兼容异构计算基础设施（CPU/GPU/QPU）。可通过本地执行、云服务或 Qiskit Serverless 实现。

## 四步工作流

```
问题 → [映射] → [优化] → [执行] → [后处理] → 解决方案
```

### 1. 映射
将经典问题转化为量子电路和算子

### 2. 优化
通过转译准备目标硬件适用的电路

### 3. 执行
使用基元在量子硬件上运行电路

### 4. 后处理
通过经典计算提取并优化结果

## 步骤 1：映射

### 目标
将领域特定问题转化为量子表示（电路、算子、哈密顿量）。

### 关键决策

**选择输出类型：**
- **采样器（Sampler）**：用于比特串输出（优化、搜索）
- **估计器（Estimator）**：用于期望值（化学、物理）

**设计电路结构：**
```python
from qiskit import QuantumCircuit
from qiskit.circuit import Parameter
import numpy as np

# 示例：用于VQE的参数化电路
def create_ansatz(num_qubits, depth):
    qc = QuantumCircuit(num_qubits)
    params = []

    for d in range(depth):
        # 旋转层
        for i in range(num_qubits):
            theta = Parameter(f'θ_{d}_{i}')
            params.append(theta)
            qc.ry(theta, i)

        # 纠缠层
        for i in range(num_qubits - 1):
            qc.cx(i, i + 1)

    return qc, params

ansatz, params = create_ansatz(num_qubits=4, depth=2)
```

### 注意事项

- **硬件拓扑**：设计时考虑后端耦合图
- **门效率**：最小化双量子比特门
- **测量基准**：确定所需测量方式

### 领域特定示例

**化学：分子哈密顿量**
```python
from qiskit_nature.second_q.drivers import PySCFDriver
from qiskit_nature.second_q.mappers import JordanWignerMapper

# 定义分子
driver = PySCFDriver(atom='H 0 0 0; H 0 0 0.735', basis='sto3g')
problem = driver.run()

# 映射为量子比特哈密顿量
mapper = JordanWignerMapper()
hamiltonian = mapper.map(problem.hamiltonian)
```

**优化：QAOA 电路**
```python
from qiskit.circuit import QuantumCircuit, Parameter

def qaoa_circuit(graph, p):
    """创建用于MaxCut问题的QAOA电路"""
    num_qubits = len(graph.nodes())
    qc = QuantumCircuit(num_qubits)

    # 初始叠加态
    qc.h(range(num_qubits))

    # 交替层
    betas = [Parameter(f'β_{i}') for i in range(p)]
    gammas = [Parameter(f'γ_{i}') for i in range(p)]

    for i in range(p):
        # 问题哈密顿量
        for edge in graph.edges():
            qc.cx(edge[0], edge[1])
            qc.rz(2 * gammas[i], edge[1])
            qc.cx(edge[0], edge[1])

        # 混合哈密顿量
        qc.rx(2 * betas[i], range(num_qubits))

    return qc
```

## 步骤 2：优化

### 目标
将抽象电路转化为硬件兼容的 ISA（指令集架构）电路。

### 转译过程

```python
from qiskit import transpile

# 基础转译
qc_isa = transpile(qc, backend=backend, optimization_level=3)

# 指定初始布局
qc_isa = transpile(
    qc,
    backend=backend,
    optimization_level=3,
    initial_layout=[0, 2, 4, 6],  # 映射到特定物理量子比特
    seed_transpiler=42  # 确保可复现性
)
```

### 预优化技巧

1. **先使用模拟器测试**：
```python
from qiskit_aer import AerSimulator

sim = AerSimulator.from_backend(backend)
qc_test = transpile(qc, sim, optimization_level=3)
print(f"预估深度: {qc_test.depth()}")
```

2. **分析转译结果**：
```python
print(f"原始门数量: {qc.size()}")
print(f"转译后门数量: {qc_isa.size()}")
print(f"双量子比特门数量: {qc_isa.count_ops().get('cx', 0)}")
```

3. **大型电路考虑电路切割**：
```python
# 当电路规模超过硬件容量时
# 使用电路切割技术拆分为子电路
```

## 步骤 3：执行

### 目标
使用基元在量子硬件上运行 ISA 电路。

### 使用采样器

```python
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

# 先转译
qc_isa = transpile(qc, backend=backend, optimization_level=3)

# 执行
sampler = Sampler(backend)
job = sampler.run([qc_isa], shots=10000)
result = job.result()
counts = result[0].data.meas.get_counts()
```

### 使用估计器

```python
from qiskit_ibm_runtime import EstimatorV2 as Estimator
from qiskit.quantum_info import SparsePauliOp

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

# 转译
qc_isa = transpile(qc, backend=backend, optimization_level=3)

# 定义可观测量
observable = SparsePauliOp(["ZZZZ", "XXXX"])

# 执行
estimator = Estimator(backend)
job = estimator.run([(qc_isa, observable)])
result = job.result()
expectation_value = result[0].data.evs
```

### 执行模式

**会话模式（迭代）：**
```python
from qiskit_ibm_runtime import Session

with Session(backend=backend) as session:
    sampler = Sampler(session=session)

    # 多轮迭代
    for iteration in range(max_iterations):
        qc_iteration = update_circuit(params[iteration])
        qc_isa = transpile(qc_iteration, backend=backend)

        job = sampler.run([qc_isa], shots=1000)
        result = job.result()

        # 更新参数
        params[iteration + 1] = optimize_params(result)
```

**批处理模式（并行）：**
```python
from qiskit_ibm_runtime import Batch

with Batch(backend=backend) as batch:
    sampler = Sampler(session=batch)

    # 批量提交任务
    jobs = []
    for qc in circuit_list:
        qc_isa = transpile(qc, backend=backend)
        job = sampler.run([qc_isa], shots=1000)
        jobs.append(job)

    # 收集结果
    results = [job.result() for job in jobs]
```

### 错误缓解

```python
from qiskit_ibm_runtime import Options

options = Options()
options.resilience_level = 2  # 0=无, 1=轻度, 2=中度, 3=重度
options.optimization_level = 3

sampler = Sampler(backend, options=options)
```

## 步骤 4：后处理

### 目标
通过经典计算从量子测量中提取有意义的结果。

### 结果处理

**采样器（比特串）：**
```python
counts = result[0].data.meas.get_counts()

# 转换为概率
total_shots = sum(counts.values())
probabilities = {state: count/total_shots for state, count in counts.items()}

# 寻找最高概率态
max_state = max(counts, key=counts.get)
print(f"最高概率态: {max_state} ({counts[max_state]}/{total_shots})")
```

**估计器（期望值）：**
```python
expectation_value = result[0].data.evs
std_dev = result[0].data.stds  # 标准差

print(f"能量值: {expectation_value} ± {std_dev}")
```

### 领域特定后处理

**化学：基态能量**
```python
def post_process_chemistry(result, nuclear_repulsion):
    """提取基态能量"""
    electronic_energy = result[0].data.evs
    total_energy = electronic_energy + nuclear_repulsion
    return total_energy
```

**优化：MaxCut 解决方案**
```python
def post_process_maxcut(counts, graph):
    """从测量结果寻找最优切割"""
    def compute_cut_value(bitstring, graph):
        cut_value = 0
        for edge in graph.edges():
            if bitstring[edge[0]] != bitstring[edge[1]]:
                cut_value += 1
        return cut_value

    # 寻找最大切割的比特串
    best_cut = 0
    best_string = None

    for bitstring, count in counts.items():
        cut = compute_cut_value(bitstring, graph)
        if cut > best_cut:
            best_cut = cut
            best_string = bitstring

    return best_string, best_cut
```

### 高级后处理

**错误缓解后处理：**
```python
# 应用经典错误缓解技术
from qiskit.result import marginal_counts

# 提取相关量子比特
relevant_qubits = [0, 1, 2]
marginal = marginal_counts(counts, indices=relevant_qubits)
```

**统计分析：**
```python
import numpy as np

def analyze_results(results_list):
    """分析多次运行的统计结果"""
    energies = [r[0].data.evs for r in results_list]

    mean_energy = np.mean(energies)
    std_energy = np.std(energies)
    confidence_interval = 1.96 * std_energy / np.sqrt(len(energies))

    return {
        '平均值': mean_energy,
        '标准差': std_energy,
        '95%置信区间': (mean_energy - confidence_interval, mean_energy + confidence_interval)
    }
```

**可视化：**
```python
from qiskit.visualization import plot_histogram
import matplotlib.pyplot as plt

# 结果可视化
plot_histogram(counts, figsize=(12, 6))
plt.title("测量结果分布")
plt.show()
```

## 完整示例：化学 VQE 计算

```python
from qiskit import QuantumCircuit, transpile
from qiskit_ibm_runtime import QiskitRuntimeService, EstimatorV2 as Estimator, Session
from qiskit.quantum_info import SparsePauliOp
from scipy.optimize import minimize
import numpy as np

# 1. 映射：创建参数化电路
def create_ansatz(num_qubits):
    qc = QuantumCircuit(num_qubits)
    params = []

    for i in range(num_qubits):
        theta = f'θ_{i}'
        params.append(theta)
        qc.ry(theta, i)

    for i in range(num_qubits - 1):
        qc.cx(i, i + 1)

    return qc, params

# 定义哈密顿量（示例：H2分子）
hamiltonian = SparsePauliOp(["IIZZ", "ZZII", "XXII", "IIXX"], coeffs=[0.3, 0.3, 0.1, 0.1])

# 2. 优化：连接并准备
service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

ansatz, param_names = create_ansatz(num_qubits=4)

# 3. 执行：运行VQE
def cost_function(params):
    # 绑定参数
    bound_circuit = ansatz.assign_parameters({param_names[i]: params[i] for i in range(len(params))})

    # 转译
    qc_isa = transpile(bound_circuit, backend=backend, optimization_level=3)

    # 执行
    job = estimator.run([(qc_isa, hamiltonian)])
    result = job.result()
    energy = result[0].data.evs

    return energy

with Session(backend=backend) as session:
    estimator = Estimator(session=session)

    # 经典优化循环
    initial_params = np.random.random(len(param_names)) * 2 * np.pi
    result = minimize(cost_function, initial_params, method='COBYLA')

# 4. 后处理：提取基态能量
ground_state_energy = result.fun
optimized_params = result.x

print(f"基态能量: {ground_state_energy}")
print(f"优化参数: {optimized_params}")
```

## 最佳实践

### 1. 先在本地迭代
使用硬件前用模拟器测试完整流程：
```python
from qiskit.primitives import StatevectorEstimator

estimator = StatevectorEstimator()
# 本地测试工作流
```

### 2. 迭代算法使用会话
VQE、QAOA 等变分算法受益于会话模式。

### 3. 选择合适的测量次数
- 开发/测试：100-1000 次测量
- 生产环境：10,000+ 次测量

### 4. 监控收敛情况
```python
energies = []

def cost_function_with_tracking(params):
    energy = cost_function(params)
    energies.append(energy)
    print(f"迭代 {len(energies)}: 能量值 = {energy}")
    return energy
```

### 5. 保存结果
```python
import json

results_data = {
    '能量值': float(ground_state_energy),
    '参数': optimized_params.tolist(),
    '迭代次数': len(energies),
    '后端': backend.name
}

with open('vqe_results.json', 'w') as f:
    json.dump(results_data, f, indent=2)
```

## Qiskit Serverless

大规模工作流使用 Qiskit Serverless 进行分布式计算：

```python
from qiskit_serverless import ServerlessClient, QiskitFunction

client = ServerlessClient()

# 定义无服务器函数
@QiskitFunction()
def run_vqe_serverless(hamiltonian, ansatz):
    # 您的VQE实现
    pass

# 远程执行
job = run_vqe_serverless(hamiltonian, ansatz)
result = job.result()
```

## 常见工作流模式

### 模式 1：参数扫描
```python
# 映射 → 单次优化 → 多次执行 → 后处理
qc_isa = transpile(parameterized_circuit, backend=backend)

with Batch(backend=backend) as batch:
    sampler = Sampler(session=batch)
    results = []

    for param_set in parameter_sweep:
        bound_qc = qc_isa.assign_parameters(param_set)
        job = sampler.run([bound_qc], shots=1000)
        results.append(job.result())
```

### 模式 2：迭代优化
```python
# 映射 → (优化 → 执行 → 后处理) 循环
with Session(backend=backend) as session:
    estimator = Estimator(session=session)

    for iteration in range(max_iter):
        qc = update_circuit(params)
        qc_isa = transpile(qc, backend=backend)

        result = estimator.run([(qc_isa, observable)]).result()
        params = update_params(result)
```

### 模式 3：集成测量
```python
# 映射 → 优化 → 多组可观测量执行 → 后处理
qc_isa = transpile(qc, backend=backend)

observables = [obs1, obs2, obs3, obs4]
jobs = [(qc_isa, obs) for obs in observables]

estimator = Estimator(backend)
result = estimator.run(jobs).result()
expectation_values = [r.data.evs for r in result]
```
