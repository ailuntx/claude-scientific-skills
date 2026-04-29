# 硬件集成

本指南介绍如何通过 Cirq 的设备接口和服务提供商在真实量子硬件上运行量子电路。

## 设备表示

### 设备类

```python
import cirq

# 定义带连接性的设备
class MyDevice(cirq.Device):
    def __init__(self, qubits, connectivity):
        self.qubits = qubits
        self.connectivity = connectivity

    @property
    def metadata(self):
        return cirq.DeviceMetadata(
            self.qubits,
            self.connectivity
        )

    def validate_operation(self, operation):
        # 检查操作在此设备上是否有效
        if len(operation.qubits) == 2:
            q0, q1 = operation.qubits
            if (q0, q1) not in self.connectivity:
                raise ValueError(f"量子比特 {q0} 和 {q1} 未连接")
```

### 设备约束

```python
# 检查设备元数据
device = cirq_google.Sycamore

# 获取量子比特拓扑
qubits = device.metadata.qubit_set
print(f"可用量子比特数: {len(qubits)}")

# 检查连接性
for q0 in qubits:
    neighbors = device.metadata.nx_graph.neighbors(q0)
    print(f"{q0} 连接到: {list(neighbors)}")

# 根据设备验证电路
try:
    device.validate_circuit(circuit)
    print("电路符合设备要求")
except ValueError as e:
    print(f"无效电路: {e}")
```

## 量子比特选择

### 最优量子比特选择

```python
import cirq_google

# 获取校准指标
processor = cirq_google.get_engine().get_processor('weber')
calibration = processor.get_current_calibration()

# 寻找错误率最低的量子比特
def select_best_qubits(calibration, n_qubits):
    """选择单量子比特门保真度最高的 n 个量子比特"""
    qubit_fidelities = {}

    for qubit in calibration.keys():
        if 'single_qubit_rb_average_error_per_gate' in calibration[qubit]:
            error = calibration[qubit]['single_qubit_rb_average_error_per_gate']
            qubit_fidelities[qubit] = 1 - error

    # 按保真度排序
    best_qubits = sorted(
        qubit_fidelities.items(),
        key=lambda x: x[1],
        reverse=True
    )[:n_qubits]

    return [q for q, _ in best_qubits]

best_qubits = select_best_qubits(calibration, n_qubits=10)
```

### 拓扑感知选择

```python
def select_connected_qubits(device, n_qubits):
    """选择形成路径或网格的连通量子比特"""
    graph = device.metadata.nx_graph

    # 寻找连通子图
    import networkx as nx
    for node in graph.nodes():
        subgraph = nx.ego_graph(graph, node, radius=n_qubits)
        if len(subgraph) >= n_qubits:
            return list(subgraph.nodes())[:n_qubits]

    raise ValueError(f"找不到 {n_qubits} 个连通量子比特")
```

## 服务提供商

### Google Quantum AI (Cirq-Google)

#### 设置

```python
import cirq_google

# 身份验证（需要 Google Cloud 项目）
# 设置环境变量: GOOGLE_CLOUD_PROJECT=your-project-id

# 获取量子引擎
engine = cirq_google.get_engine()

# 列出可用处理器
processors = engine.list_processors()
for processor in processors:
    print(f"处理器: {processor.processor_id}")
```

#### 在 Google 硬件上运行

```python
# 为 Google 设备创建电路
import cirq_google

# 获取处理器
processor = engine.get_processor('weber')
device = processor.get_device()

# 在设备量子比特上创建电路
qubits = sorted(device.metadata.qubit_set)[:5]
circuit = cirq.Circuit(
    cirq.H(qubits[0]),
    cirq.CZ(qubits[0], qubits[1]),
    cirq.measure(*qubits, key='result')
)

# 验证并运行
device.validate_circuit(circuit)
job = processor.run(circuit, repetitions=1000)

# 获取结果
results = job.results()[0]
print(results.histogram(key='result'))
```

### IonQ

#### 设置

```python
import cirq_ionq

# 设置 API 密钥
# 方式1: 环境变量
# export IONQ_API_KEY=your_api_key

# 方式2: 代码内设置
service = cirq_ionq.Service(api_key='your_api_key')
```

#### 在 IonQ 上运行

```python
import cirq_ionq

# 创建服务
service = cirq_ionq.Service(api_key='your_api_key')

# 创建电路（IonQ 使用通用量子比特）
qubits = cirq.LineQubit.range(3)
circuit = cirq.Circuit(
    cirq.H(qubits[0]),
    cirq.CNOT(qubits[0], qubits[1]),
    cirq.CNOT(qubits[1], qubits[2]),
    cirq.measure(*qubits, key='result')
)

# 在模拟器上运行
result = service.run(
    circuit=circuit,
    repetitions=1000,
    target='simulator'
)
print(result.histogram(key='result'))

# 在硬件上运行
result = service.run(
    circuit=circuit,
    repetitions=1000,
    target='qpu'
)
```

#### IonQ 作业管理

```python
# 创建作业
job = service.create_job(circuit, repetitions=1000, target='qpu')

# 检查作业状态
status = job.status()
print(f"作业状态: {status}")

# 等待完成
job.wait_until_complete()

# 获取结果
results = job.results()
```

#### IonQ 校准数据

```python
# 获取当前校准
calibration = service.get_current_calibration()

# 访问指标
print(f"保真度: {calibration['fidelity']}")
print(f"时序: {calibration['timing']}")
```

### Azure Quantum

#### 设置

```python
from azure.quantum import Workspace
from azure.quantum.cirq import AzureQuantumService

# 创建工作区连接
workspace = Workspace(
    resource_id="/subscriptions/.../resourceGroups/.../providers/Microsoft.Quantum/Workspaces/...",
    location="eastus"
)

# 创建 Cirq 服务
service = AzureQuantumService(workspace)
```

#### 在 Azure Quantum 上运行 (IonQ 后端)

```python
# 列出可用目标
targets = service.targets()
for target in targets:
    print(f"目标: {target.name}")

# 在 IonQ 模拟器上运行
result = service.run(
    circuit=circuit,
    repetitions=1000,
    target='ionq.simulator'
)

# 在 IonQ QPU 上运行
result = service.run(
    circuit=circuit,
    repetitions=1000,
    target='ionq.qpu'
)
```

#### 在 Azure Quantum 上运行 (Honeywell 后端)

```python
# 在 Honeywell System Model H1 上运行
result = service.run(
    circuit=circuit,
    repetitions=1000,
    target='honeywell.hqs-lt-s1'
)

# 检查 Honeywell 特定选项
target_info = service.get_target('honeywell.hqs-lt-s1')
print(f"目标信息: {target_info}")
```

### AQT (Alpine Quantum Technologies)

#### 设置

```python
import cirq_aqt

# 设置 API 令牌
# export AQT_TOKEN=your_token

# 创建服务
service = cirq_aqt.AQTSampler(
    remote_host='https://gateway.aqt.eu',
    access_token='your_token'
)
```

#### 在 AQT 上运行

```python
# 创建电路
qubits = cirq.LineQubit.range(3)
circuit = cirq.Circuit(
    cirq.H(qubits[0]),
    cirq.CNOT(qubits[0], qubits[1]),
    cirq.measure(*qubits, key='result')
)

# 在模拟器上运行
result = service.run(
    circuit,
    repetitions=1000,
    target='simulator'
)

# 在设备上运行
result = service.run(
    circuit,
    repetitions=1000,
    target='device'
)
```

### Pasqal

#### 设置

```python
import cirq_pasqal

# 创建 Pasqal 设备
device = cirq_pasqal.PasqalDevice(qubits=cirq.LineQubit.range(10))
```

#### 在 Pasqal 上运行

```python
# 创建采样器
sampler = cirq_pasqal.PasqalSampler(
    remote_host='https://api.pasqal.cloud',
    access_token='your_token',
    device=device
)

# 运行电路
result = sampler.run(circuit, repetitions=1000)
```

## 硬件最佳实践

### 硬件电路优化

```python
def optimize_for_hardware(circuit, device):
    """针对特定硬件优化电路"""
    from cirq.transformers import (
        optimize_for_target_gateset,
        merge_single_qubit_gates_to_phxz,
        drop_negligible_operations
    )

    # 获取设备门集
    if hasattr(device, 'gateset'):
        gateset = device.gateset
    else:
        gateset = cirq.CZTargetGateset()  # 默认

    # 优化步骤
    circuit = merge_single_qubit_gates_to_phxz(circuit)
    circuit = drop_negligible_operations(circuit)
    circuit = optimize_for_target_gateset(circuit, gateset=gateset)

    return circuit
```

### 错误缓解

```python
def run_with_readout_error_mitigation(circuit, sampler, repetitions):
    """使用校准缓解读出错误"""

    # 测量读出错误
    cal_circuits = []
    for state in range(2**len(circuit.qubits)):
        cal_circuit = cirq.Circuit()
        for i, q in enumerate(circuit.qubits):
            if state & (1 << i):
                cal_circuit.append(cirq.X(q))
        cal_circuit.append(cirq.measure(*circuit.qubits, key='m'))
        cal_circuits.append(cal_circuit)

    # 运行校准
    cal_results = [sampler.run(c, repetitions=1000) for c in cal_circuits]

    # 构建混淆矩阵
    # ... (实现细节)

    # 运行实际电路
    result = sampler.run(circuit, repetitions=repetitions)

    # 应用校正
    # ... (应用混淆矩阵的逆)

    return result
```

### 作业管理

```python
def submit_jobs_in_batches(circuits, sampler, batch_size=10):
    """批量提交多个电路"""
    jobs = []

    for i in range(0, len(circuits), batch_size):
        batch = circuits[i:i+batch_size]
        job_ids = []

        for circuit in batch:
            job = sampler.run_async(circuit, repetitions=1000)
            job_ids.append(job)

        jobs.extend(job_ids)

    # 等待所有作业
    results = [job.result() for job in jobs]
    return results
```

## 设备规格

### 检查设备能力

```python
def print_device_info(device):
    """打印设备能力和约束"""

    print(f"设备: {device}")
    print(f"量子比特数量: {len(device.metadata.qubit_set)}")

    # 门支持
    print("\n支持的门:")
    if hasattr(device, 'gateset'):
        for gate in device.gateset.gates:
            print(f"  - {gate}")

    # 连接性
    print("\n连接性:")
    graph = device.metadata.nx_graph
    print(f"  边数: {graph.number_of_edges()}")
    print(f"  平均度数: {sum(dict(graph.degree()).values()) / graph.number_of_nodes():.2f}")

    # 时长约束
    if hasattr(device, 'gate_durations'):
        print("\n门操作时长:")
        for gate, duration in device.gate_durations.items():
            print(f"  {gate}: {duration}")
```

## 认证与访问

### 设置凭证

**Google Cloud:**
```bash
# 安装 gcloud CLI
# 访问: https://cloud.google.com/sdk/docs/install

# 身份验证
gcloud auth application-default login

# 设置项目
export GOOGLE_CLOUD_PROJECT=your-project-id
```

**IonQ:**
```bash
# 设置 API 密钥
export IONQ_API_KEY=your_api_key
```

**Azure Quantum:**
```python
# 使用 Azure CLI 或工作区连接字符串
# 参考: https://docs.microsoft.com/azure/quantum/
```

**AQT:**
```bash
# 向 AQT 申请访问令牌
export AQT_TOKEN=your_token
```

**Pasqal:**
```bash
# 向 Pasqal 申请 API 访问
export PASQAL_TOKEN=your_token
```

## 最佳实践

1. **提交前验证电路**: 使用 device.validate_circuit()
2. **针对目标硬件优化**: 分解为原生门操作
3. **选择最优量子比特**: 利用校准数据进行量子比特选择
4. **监控作业状态**: 获取结果前检查作业完成状态
5. **实施错误缓解**: 使用读出错误校正
6. **高效批量作业**: 同时提交多个电路
7. **遵守速率限制**: 遵循提供商特定的 API 限制
8. **存储结果**: 立即保存昂贵的硬件结果
9. **先在模拟器测试**: 硬件运行前在模拟器验证
10. **保持电路浅层**: 硬件相干时间有限
