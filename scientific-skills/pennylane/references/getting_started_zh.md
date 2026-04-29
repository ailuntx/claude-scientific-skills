# PennyLane 入门指南

## 什么是 PennyLane？

PennyLane 是一个跨平台的 Python 库，用于量子计算、量子机器学习和量子化学。它通过自动微分技术实现量子计算机的类神经网络训练，并能无缝集成经典机器学习框架。

## 安装

使用 uv 安装 PennyLane：

```bash
uv pip install pennylane
```

安装特定设备插件（IBM、Amazon Braket、Google、Rigetti 等）：

```bash
# IBM Qiskit
uv pip install pennylane-qiskit

# Amazon Braket
uv pip install amazon-braket-pennylane-plugin

# Google Cirq
uv pip install pennylane-cirq

# Rigetti
uv pip install pennylane-rigetti
```

## 核心概念

### 量子节点（QNodes）

QNode 是可运行在量子设备上的量子函数，它将量子电路定义与设备结合：

```python
import pennylane as qml

# 定义设备
dev = qml.device('default.qubit', wires=2)

# 创建 QNode
@qml.qnode(dev)
def circuit(params):
    qml.RX(params[0], wires=0)
    qml.RY(params[1], wires=1)
    qml.CNOT(wires=[0, 1])
    return qml.expval(qml.PauliZ(0))
```

### 设备

设备用于执行量子电路，PennyLane 支持：
- **模拟器**：`default.qubit`, `default.mixed`, `lightning.qubit`
- **硬件**：通过插件访问（IBM、Amazon Braket、Rigetti 等）

```python
# 本地模拟器
dev = qml.device('default.qubit', wires=4)

# 高性能 Lightning 模拟器
dev = qml.device('lightning.qubit', wires=10)
```

### 测量

PennyLane 支持多种测量类型：

```python
@qml.qnode(dev)
def measure_circuit():
    qml.Hadamard(wires=0)
    # 期望值
    return qml.expval(qml.PauliZ(0))

@qml.qnode(dev)
def measure_probs():
    qml.Hadamard(wires=0)
    # 概率分布
    return qml.probs(wires=[0, 1])

@qml.qnode(dev)
def measure_samples():
    qml.Hadamard(wires=0)
    # 采样测量
    return qml.sample(qml.PauliZ(0))
```

## 基础工作流

### 1. 构建电路

```python
import pennylane as qml
import numpy as np

dev = qml.device('default.qubit', wires=3)

@qml.qnode(dev)
def quantum_circuit(weights):
    # 应用量子门
    qml.RX(weights[0], wires=0)
    qml.RY(weights[1], wires=1)
    qml.CNOT(wires=[0, 1])
    qml.RZ(weights[2], wires=2)

    # 测量
    return qml.expval(qml.PauliZ(0) @ qml.PauliZ(1))
```

### 2. 计算梯度

```python
# 自动微分
grad_fn = qml.grad(quantum_circuit)
weights = np.array([0.1, 0.2, 0.3])
gradients = grad_fn(weights)
```

### 3. 优化参数

```python
from pennylane import numpy as np

# 定义优化器
opt = qml.GradientDescentOptimizer(stepsize=0.1)

# 优化循环
weights = np.array([0.1, 0.2, 0.3], requires_grad=True)
for i in range(100):
    weights = opt.step(quantum_circuit, weights)
    if i % 20 == 0:
        print(f"步骤 {i}: 损失值 = {quantum_circuit(weights)}")
```

## 设备无关编程

一次编写，随处运行：

```python
# 相同电路，不同后端
@qml.qnode(qml.device('default.qubit', wires=2))
def circuit_simulator(x):
    qml.RX(x, wires=0)
    return qml.expval(qml.PauliZ(0))

# 切换至硬件（若可用）
@qml.qnode(qml.device('qiskit.ibmq', wires=2))
def circuit_hardware(x):
    qml.RX(x, wires=0)
    return qml.expval(qml.PauliZ(0))
```

## 常用模式

### 参数化电路

```python
@qml.qnode(dev)
def parameterized_circuit(params, x):
    # 数据编码
    qml.RX(x, wires=0)

    # 应用参数化层
    for param in params:
        qml.RY(param, wires=0)
        qml.CNOT(wires=[0, 1])

    return qml.expval(qml.PauliZ(0))
```

### 电路模板

使用内置模板实现通用模式：

```python
from pennylane.templates import StronglyEntanglingLayers

@qml.qnode(dev)
def template_circuit(weights):
    StronglyEntanglingLayers(weights, wires=range(3))
    return qml.expval(qml.PauliZ(0))

# 为模板生成随机权重
n_layers = 2
n_wires = 3
shape = StronglyEntanglingLayers.shape(n_layers, n_wires)
weights = np.random.random(shape)
```

## 调试与可视化

### 打印电路结构

```python
print(qml.draw(circuit)(params))
print(qml.draw_mpl(circuit)(params))  # Matplotlib 可视化
```

### 检查操作

```python
with qml.tape.QuantumTape() as tape:
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])

print(tape.operations)
print(tape.measurements)
```

## 后续步骤

各专题详细指南：
- **构建电路**：参见 `references/quantum_circuits.md`
- **量子机器学习**：参见 `references/quantum_ml.md`
- **化学应用**：参见 `references/quantum_chemistry.md`
- **设备管理**：参见 `references/devices_backends.md`
- **优化方法**：参见 `references/optimization.md`
- **高级功能**：参见 `references/advanced_features.md`

## 资源

- 官方文档：https://docs.pennylane.ai
- 代码手册：https://pennylane.ai/codebook
- QML 示例：https://pennylane.ai/qml/demonstrations
- 社区论坛：https://discuss.pennylane.ai
