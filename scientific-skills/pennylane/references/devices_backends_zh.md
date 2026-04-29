# PennyLane 中的设备与后端

## 目录
1. [内置模拟器](#内置模拟器)
2. [硬件插件](#硬件插件)
3. [设备选择](#设备选择)
4. [设备配置](#设备配置)
5. [自定义设备](#自定义设备)
6. [性能优化](#性能优化)

## 内置模拟器

### default.qubit

通用态矢量模拟器：

```python
import pennylane as qml

# 基础初始化
dev = qml.device('default.qubit', wires=4)

# 带采样次数（采样模式）
dev = qml.device('default.qubit', wires=4, shots=1000)

# 指定量子线标签
dev = qml.device('default.qubit', wires=['a', 'b', 'c', 'd'])
```

### default.mixed

用于含噪声量子系统的混合态模拟器：

```python
# 支持密度矩阵模拟
dev = qml.device('default.mixed', wires=2)

@qml.qnode(dev)
def noisy_circuit():
    qml.Hadamard(wires=0)

    # 施加噪声
    qml.DepolarizingChannel(0.1, wires=0)

    qml.CNOT(wires=[0, 1])

    # 振幅阻尼
    qml.AmplitudeDamping(0.05, wires=1)

    return qml.expval(qml.PauliZ(0))
```

### default.qubit.torch, default.qubit.tf, default.qubit.jax

框架专用模拟器，集成度更高：

```python
# PyTorch
dev = qml.device('default.qubit.torch', wires=4)

# TensorFlow
dev = qml.device('default.qubit.tf', wires=4)

# JAX
dev = qml.device('default.qubit.jax', wires=4)
```

### lightning.qubit

高性能 C++ 模拟器：

```python
# 比 default.qubit 更快
dev = qml.device('lightning.qubit', wires=20)

# 高效支持更大系统
@qml.qnode(dev)
def large_circuit():
    for i in range(20):
        qml.Hadamard(wires=i)

    for i in range(19):
        qml.CNOT(wires=[i, i+1])

    return qml.expval(qml.PauliZ(0))
```

### default.clifford

高效 Clifford 电路模拟器：

```python
# 仅支持 Clifford 门 (H, S, CNOT 等)
dev = qml.device('default.clifford', wires=100)

@qml.qnode(dev)
def clifford_circuit():
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])
    qml.S(wires=1)
    # 不能使用 RX, RY, RZ 等门

    return qml.expval(qml.PauliZ(0))
```

## 硬件插件

### IBM Quantum (Qiskit)

```bash
# 安装插件
uv pip install pennylane-qiskit
```

```python
import pennylane as qml

# 使用 IBM 模拟器
dev = qml.device('qiskit.aer', wires=2)

# 使用 IBM 量子硬件
dev = qml.device(
    'qiskit.ibmq',
    wires=2,
    backend='ibmq_manila',  # 指定后端
    shots=1024
)

# 带 API 令牌
dev = qml.device(
    'qiskit.ibmq',
    wires=2,
    backend='ibmq_manila',
    ibmqx_token='YOUR_API_TOKEN'
)

@qml.qnode(dev)
def circuit():
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])
    return qml.expval(qml.PauliZ(0))
```

### Amazon Braket

```bash
# 安装插件
uv pip install amazon-braket-pennylane-plugin
```

```python
# 使用 Braket 模拟器
dev = qml.device(
    'braket.local.qubit',
    wires=2
)

# 使用 AWS 模拟器
dev = qml.device(
    'braket.aws.qubit',
    device_arn='arn:aws:braket:::device/quantum-simulator/amazon/sv1',
    wires=4,
    s3_destination_folder=('amazon-braket-outputs', 'outputs')
)

# 使用量子硬件 (IonQ, Rigetti 等)
dev = qml.device(
    'braket.aws.qubit',
    device_arn='arn:aws:braket:us-east-1::device/qpu/ionq/Harmony',
    wires=11,
    shots=1000,
    s3_destination_folder=('amazon-braket-outputs', 'outputs')
)
```

### Google Cirq

```bash
# 安装插件
uv pip install pennylane-cirq
```

```python
# 使用 Cirq 模拟器
dev = qml.device('cirq.simulator', wires=2)

# 使用带 qsim 的 Cirq（更快）
dev = qml.device('cirq.qsim', wires=20)

# 使用 Google 量子硬件（需有访问权限）
dev = qml.device(
    'cirq.pasqal',
    wires=2,
    device='rainbow',
    shots=1000
)
```

### Rigetti Forest

```bash
# 安装插件
uv pip install pennylane-rigetti
```

```python
# 使用 QVM（量子虚拟机）
dev = qml.device('rigetti.qvm', device='4q-qvm', shots=1000)

# 使用 Rigetti QPU
dev = qml.device('rigetti.qpu', device='Aspen-M-3', shots=1000)
```

### Microsoft Azure Quantum

```bash
# 安装插件
uv pip install pennylane-azure
```

```python
# 使用 Azure 模拟器
dev = qml.device(
    'azure.simulator',
    wires=4,
    workspace='your-workspace',
    resource_group='your-resource-group',
    subscription_id='your-subscription-id'
)

# 使用 Azure 上的 IonQ
dev = qml.device(
    'azure.ionq',
    wires=11,
    workspace='your-workspace',
    resource_group='your-resource-group',
    subscription_id='your-subscription-id',
    shots=500
)
```

### IonQ

```bash
# 安装插件
uv pip install pennylane-ionq
```

```python
# 使用 IonQ 硬件
dev = qml.device(
    'ionq.simulator',  # 或 'ionq.qpu'
    wires=11,
    shots=1024,
    api_key='your_api_key'
)
```

### Xanadu 硬件 (Borealis)

```python
# 光子量子计算机
dev = qml.device(
    'strawberryfields.remote',
    backend='borealis',
    shots=10000
)
```

## 设备选择

### 选择合适的设备

```python
def select_device(n_qubits, use_hardware=False, noise_model=None):
    """根据需求选择合适设备"""

    if use_hardware:
        # 使用真实量子硬件
        if n_qubits <= 11:
            return qml.device('ionq.qpu', wires=n_qubits, shots=1000)
        elif n_qubits <= 127:
            return qml.device('qiskit.ibmq', wires=n_qubits, shots=1024)
        else:
            raise ValueError(f"无支持 {n_qubits} 量子位的硬件")

    elif noise_model:
        # 使用含噪声模拟器
        return qml.device('default.mixed', wires=n_qubits)

    else:
        # 使用理想模拟器
        if n_qubits <= 20:
            return qml.device('lightning.qubit', wires=n_qubits)
        else:
            return qml.device('default.qubit', wires=n_qubits)

# 用法示例
dev = select_device(n_qubits=10, use_hardware=False)
```

### 设备能力查询

```python
# 检查设备能力
dev = qml.device('default.qubit', wires=4)

print("设备名称:", dev.name)
print("量子线数量:", dev.num_wires)
print("支持采样:", dev.shots is not None)

# 检查支持的操作
print("支持的门操作:", dev.operations)

# 检查支持的观测量
print("支持的观测量:", dev.observables)
```

## 设备配置

### 设置采样次数

```python
# 精确模拟（无采样）
dev = qml.device('default.qubit', wires=2)

@qml.qnode(dev)
def exact_circuit():
    qml.Hadamard(wires=0)
    return qml.expval(qml.PauliZ(0))

result = exact_circuit()  # 返回精确期望值

# 采样模式（带采样次数）
dev_sampled = qml.device('default.qubit', wires=2, shots=1000)

@qml.qnode(dev_sampled)
def sampled_circuit():
    qml.Hadamard(wires=0)
    return qml.expval(qml.PauliZ(0))

result = sampled_circuit()  # 基于采样估算
```

### 动态采样次数

```python
# 每次执行时更改采样次数
dev = qml.device('default.qubit', wires=2)

@qml.qnode(dev)
def circuit():
    qml.Hadamard(wires=0)
    return qml.expval(qml.PauliZ(0))

# 不同采样次数
result_100 = circuit(shots=100)
result_1000 = circuit(shots=1000)
result_exact = circuit(shots=None)  # 精确计算
```

### 解析模式 vs 有限采样

```python
# 比较解析解与采样结果
dev_analytic = qml.device('default.qubit', wires=2)
dev_sampled = qml.device('default.qubit', wires=2, shots=1000)

@qml.qnode(dev_analytic)
def circuit_analytic(x):
    qml.RX(x, wires=0)
    return qml.expval(qml.PauliZ(0))

@qml.qnode(dev_sampled)
def circuit_sampled(x):
    qml.RX(x, wires=0)
    return qml.expval(qml.PauliZ(0))

import numpy as np
x = np.pi / 4

print(f"解析解: {circuit_analytic(x)}")
print(f"采样解: {circuit_sampled(x)}")
print(f"精确值: {np.cos(x)}")
```

### 设置随机种子保证可复现性

```python
# 设置随机种子
dev = qml.device('default.qubit', wires=2, shots=1000, seed=42)

@qml.qnode(dev)
def circuit():
    qml.Hadamard(wires=0)
    return qml.sample(qml.PauliZ(0))

# 可复现的结果
samples1 = circuit()
samples2 = circuit()  # 若设置种子则与 samples1 相同
```

## 自定义设备

### 创建自定义设备

```python
from pennylane.devices import DefaultQubit

class CustomDevice(DefaultQubit):
    """带附加功能的自定义量子设备"""

    name = 'Custom device'
    short_name = 'custom'
    pennylane_requires = '>=0.30.0'
    version = '0.1.0'
    author = 'Your Name'

    def __init__(self, wires, shots=None, **kwargs):
        super().__init__(wires=wires, shots=shots)
        # 自定义初始化

    def apply(self, operations, **kwargs):
        """带自定义逻辑的应用操作"""
        # 自定义操作处理
        for op in operations:
            # 记录或修改操作
            print(f"正在应用: {op.name}")

        # 调用父类实现
        super().apply(operations, **kwargs)

# 使用自定义设备
dev = CustomDevice(wires=4)
```

### 插件开发

```python
# 定义自定义插件操作
class CustomGate(qml.operation.Operation):
    """自定义量子门"""

    num_wires = 1
    num_params = 1
    par_domain = 'R'

    def decomposition(self):
        """分解为标准门序列"""
        theta = self.parameters[0]
        wires = self.wires

        return [
            qml.RY(theta / 2, wires=wires),
            qml.RZ(theta, wires=wires),
            qml.RY(-theta / 2, wires=wires)
        ]

# 向设备注册
qml.ops.CustomGate = CustomGate
```

## 性能优化

### 批量执行

```python
# 高效执行多组参数
dev = qml.device('default.qubit', wires=2)

@qml.qnode(dev)
def circuit(params):
    qml.RX(params[0], wires=0)
    qml.RY(params[1], wires=1)
    qml.CNOT(wires=[0, 1])
    return qml.expval(qml.PauliZ(0))

# 批量参数
params_batch = np.random.random((100, 2))

# 向量化执行（更快）
results = [circuit(p) for p in params_batch]
```

### 设备缓存

```python
# 缓存设备以供复用
_device_cache = {}

def get_device(n_qubits, device_type='default.qubit'):
    """获取或创建缓存设备"""
    key = (device_type, n_qubits)

    if key not in _device_cache:
        _device_cache[key] = qml.device(device_type, wires=n_qubits)

    return _device_cache[key]

# 复用设备
dev1 = get_device(4)
dev2 = get_device(4)  # 返回相同设备
```

### 使用 Catalyst 进行 JIT 编译

```python
# 安装 Catalyst
# uv pip install pennylane-catalyst

import pennylane as qml
from catalyst import qjit

dev = qml.device('lightning.qubit', wires=4)

@qjit  # 即时编译
@qml.qnode(dev)
def compiled_circuit(x):
    qml.RX(x, wires=0)
    qml.Hadamard(wires=1)
    qml.CNOT(wires=[0, 1])
    return qml.expval(qml.PauliZ(0))

# 首次调用编译，后续调用快速执行
result = compiled_circuit(0.5)
```

### 并行执行

```python
from multiprocessing import Pool

def run_circuit(params):
    """用给定参数运行电路"""
    dev = qml.device('default.qubit', wires=4)

    @qml.qnode(dev)
    def circuit(p):
        # 电路定义
        return qml.expval(qml.PauliZ(0))

    return circuit(params)

# 并行执行
param_list = [np.random.random(10) for _ in range(100)]

with Pool(processes=4) as pool:
    results = pool.map(run_circuit, param_list)
```

### GPU 加速

```python
# 使用 GPU 加速设备（如果可用）
try:
    dev = qml.device('lightning.gpu', wires=20)
except:
    dev = qml.device('lightning.qubit', wires=20)

@qml.qnode(dev)
def gpu_circuit():
    # 大型电路受益于 GPU
    for i in range(20):
        qml.Hadamard(wires=i)

    for i in range(19):
        qml.CNOT(wires=[i, i+1])

    return [qml.expval(qml.PauliZ(i)) for i in range(20)]
```

## 最佳实践

1. **从模拟器开始** - 在硬件上运行前先用 `default.qubit` 测试
2. **使用 lightning 提速** - 大型电路切换至 `lightning.qubit`
3. **匹配设备与任务** - 噪声研究使用 `default.mixed`
4. **缓存设备** - 复用设备对象避免初始化开销
5. **设置合适采样次数** - 平衡精度与速度
6. **检查设备能力** - 确认设备支持所需操作
7. **处理硬件错误** - 实现重试和错误缓解
8. **监控成本** - 跟踪硬件使用情况和费用
9. **尽可能使用 JIT** - 用 Catalyst 编译电路提速
10. **先本地测试** - 提交硬件前在模拟器验证

## 设备对比

| 设备 | 类型 | 最大量子位数 | 速度 | 噪声 | 适用场景 |
|------|------|------------|------|------|----------

| lightning.qubit | 模拟器 | ~30 | 快速 | 无 | 大型电路 |
| default.mixed | 模拟器 | ~15 | 慢速 | 有 | 噪声研究 |
| default.clifford | 模拟器 | 100+ | 极快 | 无 | Clifford电路 |
| IBM Quantum | 硬件 | 127 | 慢速 | 有 | 真实实验 |
| IonQ | 硬件 | 11 | 慢速 | 低 | 高保真度 |
| Rigetti | 硬件 | 80 | 慢速 | 有 | 研究 |
| Borealis | 硬件 | 216 | 慢速 | 有 | 光子量子计算 |
