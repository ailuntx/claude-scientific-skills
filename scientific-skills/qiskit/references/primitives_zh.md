# Qiskit 原语

原语是执行量子电路的基础构建模块。Qiskit 提供两种主要原语：**采样器**（用于测量比特串）和**估计器**（用于计算期望值）。

## 原语类型

### 采样器
计算量子电路的比特串概率或准概率。适用于需要：
- 测量结果
- 输出概率分布
- 从量子态中采样

### 估计器
计算量子电路可观测量的期望值。适用于需要：
- 能量计算
- 可观测量的测量
- 变分算法优化

## V2 接口（当前标准）

Qiskit 使用 V2 原语（BaseSamplerV2, BaseEstimatorV2）作为当前标准。V1 原语为旧版。

## 采样器原语

### StatevectorSampler（本地模拟）

```python
from qiskit import QuantumCircuit
from qiskit.primitives import StatevectorSampler

# 创建电路
qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.measure_all()

# 使用采样器运行
sampler = StatevectorSampler()
result = sampler.run([qc], shots=1024).result()

# 获取结果
counts = result[0].data.meas.get_counts()
print(counts)  # 例如 {'00': 523, '11': 501}
```

### 多电路处理

```python
qc1 = QuantumCircuit(2)
qc1.h(0)
qc1.measure_all()

qc2 = QuantumCircuit(2)
qc2.x(0)
qc2.measure_all()

# 运行多个电路
sampler = StatevectorSampler()
job = sampler.run([qc1, qc2], shots=1000)
results = job.result()

# 获取独立结果
counts1 = results[0].data.meas.get_counts()
counts2 = results[1].data.meas.get_counts()
```

### 使用参数

```python
from qiskit.circuit import Parameter

theta = Parameter('θ')
qc = QuantumCircuit(1)
qc.ry(theta, 0)
qc.measure_all()

# 带参数值运行
sampler = StatevectorSampler()
param_values = [[0], [np.pi/4], [np.pi/2]]
result = sampler.run([(qc, param_values)], shots=1024).result()
```

## 估计器原语

### StatevectorEstimator（本地模拟）

```python
from qiskit import QuantumCircuit
from qiskit.primitives import StatevectorEstimator
from qiskit.quantum_info import SparsePauliOp

# 创建无测量电路
qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)

# 定义可观测算符
observable = SparsePauliOp(["ZZ", "XX"])

# 运行估计器
estimator = StatevectorEstimator()
result = estimator.run([(qc, observable)]).result()

# 获取期望值
exp_value = result[0].data.evs
print(f"期望值: {exp_value}")
```

### 多可观测算符

```python
from qiskit.quantum_info import SparsePauliOp

qc = QuantumCircuit(2)
qc.h(0)

obs1 = SparsePauliOp(["ZZ"])
obs2 = SparsePauliOp(["XX"])

estimator = StatevectorEstimator()
result = estimator.run([(qc, obs1), (qc, obs2)]).result()

ev1 = result[0].data.evs
ev2 = result[1].data.evs
```

### 参数化估计器

```python
from qiskit.circuit import Parameter
import numpy as np

theta = Parameter('θ')
qc = QuantumCircuit(1)
qc.ry(theta, 0)

observable = SparsePauliOp(["Z"])

# 多参数值运行
estimator = StatevectorEstimator()
param_values = [[0], [np.pi/4], [np.pi/2], [np.pi]]
result = estimator.run([(qc, observable, param_values)]).result()
```

## IBM Quantum Runtime 原语

在真实硬件上运行时使用：

### Runtime 采样器

```python
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.measure_all()

# 在真实硬件上运行
sampler = Sampler(backend)
job = sampler.run([qc], shots=1024)
result = job.result()
counts = result[0].data.meas.get_counts()
```

### Runtime 估计器

```python
from qiskit_ibm_runtime import QiskitRuntimeService, EstimatorV2 as Estimator
from qiskit.quantum_info import SparsePauliOp

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)

observable = SparsePauliOp(["ZZ"])

# 在真实硬件上运行
estimator = Estimator(backend)
job = estimator.run([(qc, observable)])
result = job.result()
exp_value = result[0].data.evs
```

## 会话处理迭代任务

会话可将多个任务分组以减少队列等待时间：

```python
from qiskit_ibm_runtime import Session

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

with Session(backend=backend) as session:
    sampler = Sampler(session=session)

    # 在会话中运行多个任务
    job1 = sampler.run([qc1], shots=1024)
    result1 = job1.result()

    job2 = sampler.run([qc2], shots=1024)
    result2 = job2.result()
```

## 批处理模式并行任务

批处理模式可并行运行独立任务：

```python
from qiskit_ibm_runtime import Batch

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

with Batch(backend=backend) as batch:
    sampler = Sampler(session=batch)

    # 提交多个独立任务
    job1 = sampler.run([qc1], shots=1024)
    job2 = sampler.run([qc2], shots=1024)

    # 就绪后获取结果
    result1 = job1.result()
    result2 = job2.result()
```

## 结果处理

### 采样器结果

```python
result = sampler.run([qc], shots=1024).result()

# 获取计数
counts = result[0].data.meas.get_counts()

# 计算概率
probs = {k: v/1024 for k, v in counts.items()}

# 获取元数据
metadata = result[0].metadata
```

### 估计器结果

```python
result = estimator.run([(qc, observable)]).result()

# 期望值
exp_val = result[0].data.evs

# 标准差（若可用）
std_dev = result[0].data.stds

# 元数据
metadata = result[0].metadata
```

## V1 原语差异

**V2 改进：**
- 更灵活的参数绑定
- 更优的结果结构
- 性能提升
- 更简洁的 API 设计

**从 V1 迁移：**
- 使用 `StatevectorSampler` 替代 `Sampler`
- 使用 `StatevectorEstimator` 替代 `Estimator`
- 结果访问方式从 `.result().quasi_dists[0]` 改为 `.result()[0].data.meas.get_counts()`
