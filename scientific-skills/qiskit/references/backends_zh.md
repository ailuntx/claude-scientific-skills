# 硬件后端与执行

Qiskit 是后端无关的框架，支持在模拟器和多家供应商的真实量子硬件上执行计算。

## 后端类型

### 本地模拟器
- 在本地机器运行
- 无需账户
- 适合开发和测试

### 云端硬件
- IBM Quantum (100+量子位系统)
- IonQ (离子阱技术)
- Amazon Braket (Rigetti, IonQ, Oxford Quantum Circuits)
- 其他供应商通过插件支持

## IBM Quantum 后端

### 连接 IBM Quantum

```python
from qiskit_ibm_runtime import QiskitRuntimeService

# 首次使用：保存凭证
QiskitRuntimeService.save_account(
    channel="ibm_quantum",
    token="YOUR_IBM_QUANTUM_TOKEN"
)

# 后续会话：加载凭证
service = QiskitRuntimeService()
```

### 列出可用后端

```python
# 列出所有可用后端
backends = service.backends()
for backend in backends:
    print(f"{backend.name}: {backend.num_qubits} qubits")

# 按最小量子位数过滤
backends_127q = service.backends(min_num_qubits=127)

# 获取特定后端
backend = service.backend("ibm_brisbane")
backend = service.least_busy()  # 获取最空闲后端
```

### 后端属性

```python
backend = service.backend("ibm_brisbane")

# 基础信息
print(f"名称: {backend.name}")
print(f"量子位数: {backend.num_qubits}")
print(f"版本: {backend.version}")
print(f"状态: {backend.status()}")

# 耦合映射（量子位连接关系）
print(backend.coupling_map)

# 基础门集
print(backend.configuration().basis_gates)

# 量子位属性
print(backend.qubit_properties(0))  # 量子位0的属性
```

### 检查后端状态

```python
status = backend.status()
print(f"运行状态: {status.operational}")
print(f"排队任务数: {status.pending_jobs}")
print(f"状态信息: {status.status_msg}")
```

## 在 IBM Quantum 硬件上运行

### 使用运行时基元

```python
from qiskit import QuantumCircuit, transpile
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

# 创建并转换量子电路
qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.measure_all()

# 针对后端转换电路
transpiled_qc = transpile(qc, backend=backend, optimization_level=3)

# 使用采样器运行
sampler = Sampler(backend)
job = sampler.run([transpiled_qc], shots=1024)

# 获取结果
result = job.result()
counts = result[0].data.meas.get_counts()
print(counts)
```

### 任务管理

```python
# 提交任务
job = sampler.run([qc], shots=1024)

# 获取任务ID（用于后续检索）
job_id = job.job_id()
print(f"任务ID: {job_id}")

# 检查任务状态
print(job.status())

# 等待完成
result = job.result()

# 后续检索任务
service = QiskitRuntimeService()
retrieved_job = service.job(job_id)
result = retrieved_job.result()
```

### 任务排队

```python
# 检查队列位置
job_status = job.status()
print(f"队列位置: {job.queue_position()}")

# 必要时取消任务
job.cancel()
```

## 会话模式

使用会话模式处理迭代算法（如VQE, QAOA）以减少排队时间：

```python
from qiskit_ibm_runtime import Session, SamplerV2 as Sampler

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

with Session(backend=backend) as session:
    sampler = Sampler(session=session)

    # 在同一会话中多次迭代
    for iteration in range(10):
        # 参数化电路
        qc = create_parameterized_circuit(params[iteration])
        job = sampler.run([qc], shots=1024)
        result = job.result()

        # 根据结果更新参数
        params[iteration + 1] = optimize(result)
```

会话优势：
- 减少迭代间排队等待
- 会话期间保证后端可用性
- 更适合变分算法

## 批处理模式

使用批处理模式执行独立并行任务：

```python
from qiskit_ibm_runtime import Batch, SamplerV2 as Sampler

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

with Batch(backend=backend) as batch:
    sampler = Sampler(session=batch)

    # 提交多个独立任务
    jobs = []
    for qc in circuit_list:
        job = sampler.run([qc], shots=1024)
        jobs.append(job)

    # 收集所有结果
    results = [job.result() for job in jobs]
```

## 本地模拟器

### 状态向量采样器（理想模拟）

```python
from qiskit.primitives import StatevectorSampler

sampler = StatevectorSampler()
result = sampler.run([qc], shots=1024).result()
counts = result[0].data.meas.get_counts()
```

### Aer 模拟器（带噪声模拟）

```python
from qiskit_aer import AerSimulator
from qiskit_ibm_runtime import SamplerV2 as Sampler

# 理想模拟
simulator = AerSimulator()

# 使用后端噪声模型模拟
backend = service.backend("ibm_brisbane")
noisy_simulator = AerSimulator.from_backend(backend)

# 运行模拟
transpiled_qc = transpile(qc, simulator)
sampler = Sampler(simulator)
job = sampler.run([transpiled_qc], shots=1024)
result = job.result()
```

### Aer GPU 加速

```python
# 使用GPU加速模拟
simulator = AerSimulator(method='statevector', device='GPU')
```

## 第三方供应商

### IonQ

IonQ 提供全连接离子阱量子计算机：

```python
from qiskit_ionq import IonQProvider

provider = IonQProvider("YOUR_IONQ_API_TOKEN")

# 列出IonQ后端
backends = provider.backends()
backend = provider.get_backend("ionq_qpu")

# 运行电路
job = backend.run(qc, shots=1024)
result = job.result()
```

### Amazon Braket

```python
from qiskit_braket_provider import BraketProvider

provider = BraketProvider()

# 列出可用设备
backends = provider.backends()

# 使用特定设备
backend = provider.get_backend("Rigetti")
job = backend.run(qc, shots=1024)
result = job.result()
```

## 错误缓解

### 测量错误缓解

```python
from qiskit_ibm_runtime import SamplerV2 as Sampler, Options

# 配置错误缓解
options = Options()
options.resilience_level = 1  # 0=无缓解, 1=基础, 2=中等, 3=高级

sampler = Sampler(backend, options=options)
job = sampler.run([qc], shots=1024)
result = job.result()
```

### 错误缓解级别

- **级别0**: 无缓解
- **级别1**: 读取错误缓解
- **级别2**: 级别1 + 门错误缓解
- **级别3**: 级别2 + 高级技术

**Qiskit的Samplomatic包**可通过概率错误消除将采样开销降低高达100倍。

### 零噪声外推 (ZNE)

```python
options = Options()
options.resilience_level = 2
options.resilience.zne_mitigation = True

sampler = Sampler(backend, options=options)
```

## 用量与成本监控

### 检查账户用量

```python
# IBM Quantum账户
service = QiskitRuntimeService()

# 查看剩余点数
print(service.usage())
```

### 预估任务成本

```python
from qiskit_ibm_runtime import EstimatorV2 as Estimator

backend = service.backend("ibm_brisbane")

# 预估任务成本
estimator = Estimator(backend)
# 成本取决于电路复杂度和采样次数
```

## 最佳实践

### 1. 运行前务必转换电路

```python
# 错误：未转换直接运行
job = sampler.run([qc], shots=1024)

# 正确：先转换电路
qc_transpiled = transpile(qc, backend=backend, optimization_level=3)
job = sampler.run([qc_transpiled], shots=1024)
```

### 2. 先用模拟器测试

```python
# 上真机前用带噪模拟器测试
noisy_sim = AerSimulator.from_backend(backend)
qc_test = transpile(qc, noisy_sim, optimization_level=3)

# 验证结果合理性
# 再上真机运行
```

### 3. 合理设置采样次数

```python
# 优化算法：较少采样（100-1000次）
# 最终测量：较多采样（10000+次）

# 根据阶段动态调整
shots_optimization = 500
shots_final = 10000
```

### 4. 策略性选择后端

```python
# 测试阶段：选最空闲后端
backend = service.least_busy(min_num_qubits=5)

# 生产环境：选符合需求的后端
backend = service.backend("ibm_brisbane")  # 127量子位
```

### 5. 变分算法使用会话模式

会话模式特别适合VQE、QAOA等迭代算法。

### 6. 监控任务状态

```python
import time

job = sampler.run([qc], shots=1024)

while job.status().name not in ['DONE', 'ERROR', 'CANCELLED']:
    print(f"状态: {job.status().name}")
    time.sleep(10)

result = job.result()
```

## 故障排除

### 问题："找不到后端"
```python
# 列出可用后端
print([b.name for b in service.backends()])
```

### 问题："凭证无效"
```python
# 重新保存凭证
QiskitRuntimeService.save_account(
    channel="ibm_quantum",
    token="YOUR_TOKEN",
    overwrite=True
)
```

### 问题：排队时间过长
```python
# 使用最空闲后端
backend = service.least_busy(min_num_qubits=5)

# 或多任务使用批处理模式
```

### 问题：任务失败提示"电路过大"
```python
# 降低电路复杂度
# 使用更高级别电路转换
qc_opt = transpile(qc, backend=backend, optimization_level=3)
```

## 后端对比

| 供应商 | 连接性 | 门集 | 备注 |
|----------|-------------|----------|--------|
| IBM Quantum | 有限连接 | CX, RZ, SX, X | 100+量子位系统，高质量 |
| IonQ | 全连接 | GPI, GPI2, MS | 离子阱技术，低错误率 |
| Rigetti | 有限连接 | CZ, RZ, RX | 超导量子位 |
| Oxford Quantum Circuits | 有限连接 | ECR, RZ, SX | Coaxmon技术 |
