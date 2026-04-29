---
name: pennylane
description: 硬件无关的量子机器学习框架，支持自动微分。适用于通过梯度训练量子电路、构建量子-经典混合模型，或需要跨IBM/Google/Rigetti/IonQ设备移植的场景。最适用于变分算法（VQE、QAOA）、量子神经网络及与PyTorch/JAX/TensorFlow的集成。硬件特定优化请使用qiskit（IBM）或cirq（Google）；开放量子系统请使用qutip。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# PennyLane

## 概述

PennyLane是一个量子计算库，可将量子计算机像神经网络一样训练。它提供量子电路的自动微分、设备无关的编程能力，以及与经典机器学习框架的无缝集成。

## 安装

使用uv安装：

```bash
uv pip install pennylane
```

访问量子硬件需安装设备插件：

```bash
# IBM Quantum
uv pip install pennylane-qiskit

# Amazon Braket
uv pip install amazon-braket-pennylane-plugin

# Google Cirq
uv pip install pennylane-cirq

# Rigetti Forest
uv pip install pennylane-rigetti

# IonQ
uv pip install pennylane-ionq
```

## 快速入门

构建量子电路并优化参数：

```python
import pennylane as qml
from pennylane import numpy as np

# 创建设备
dev = qml.device('default.qubit', wires=2)

# 定义量子电路
@qml.qnode(dev)
def circuit(params):
    qml.RX(params[0], wires=0)
    qml.RY(params[1], wires=1)
    qml.CNOT(wires=[0, 1])
    return qml.expval(qml.PauliZ(0))

# 优化参数
opt = qml.GradientDescentOptimizer(stepsize=0.1)
params = np.array([0.1, 0.2], requires_grad=True)

for i in range(100):
    params = opt.step(circuit, params)
```

## 核心功能

### 1. 量子电路构建

通过门操作、测量和态制备构建电路。详见 `references/quantum_circuits.md`：
- 单量子位与多量子位门
- 受控操作与条件逻辑
- 电路中间测量与自适应电路
- 多种测量类型（期望值、概率、采样）
- 电路检查与调试

### 2. 量子机器学习

创建量子-经典混合模型。详见 `references/quantum_ml.md`：
- 与PyTorch/JAX/TensorFlow集成
- 量子神经网络与变分分类器
- 数据编码策略（角度、振幅、基态、IQP）
- 通过反向传播训练混合模型
- 量子电路迁移学习

### 3. 量子化学

模拟分子并计算基态能量。详见 `references/quantum_chemistry.md`：
- 分子哈密顿量生成
- 变分量子本征求解器（VQE）
- 化学UCCSD拟设
- 几何优化与解离曲线
- 分子属性计算

### 4. 设备管理

在模拟器或量子硬件上执行。详见 `references/devices_backends.md`：
- 内置模拟器（default.qubit, lightning.qubit, default.mixed）
- 硬件插件（IBM, Amazon Braket, Google, Rigetti, IonQ）
- 设备选择与配置
- 性能优化与缓存
- GPU加速与JIT编译

### 5. 优化

使用多种优化器训练量子电路。详见 `references/optimization.md`：
- 内置优化器（Adam、梯度下降、动量法、RMSProp）
- 梯度计算方法（反向传播、参数平移、伴随法）
- 变分算法（VQE、QAOA）
- 训练策略（学习率调度、小批量）
- 应对贫瘠高原与局部极小值

### 6. 高级功能

利用模板、变换与编译。详见 `references/advanced_features.md`：
- 电路模板与层结构
- 变换与电路优化
- 脉冲级编程
- Catalyst即时编译
- 噪声模型与误差缓解
- 资源估算

## 典型工作流

### 训练变分分类器

```python
# 1. 定义拟设
@qml.qnode(dev)
def classifier(x, weights):
    # 数据编码
    qml.AngleEmbedding(x, wires=range(4))

    # 变分层
    qml.StronglyEntanglingLayers(weights, wires=range(4))

    return qml.expval(qml.PauliZ(0))

# 2. 训练
opt = qml.AdamOptimizer(stepsize=0.01)
weights = np.random.random((3, 4, 3))  # 3层, 4量子位

for epoch in range(100):
    for x, y in zip(X_train, y_train):
        weights = opt.step(lambda w: (classifier(x, w) - y)**2, weights)
```

### 运行分子基态VQE

```python
from pennylane import qchem

# 1. 构建哈密顿量
symbols = ['H', 'H']
coords = np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.74])
H, n_qubits = qchem.molecular_hamiltonian(symbols, coords)

# 2. 定义拟设
@qml.qnode(dev)
def vqe_circuit(params):
    qml.BasisState(qchem.hf_state(2, n_qubits), wires=range(n_qubits))
    qml.UCCSD(params, wires=range(n_qubits))
    return qml.expval(H)

# 3. 优化
opt = qml.AdamOptimizer(stepsize=0.1)
params = np.zeros(10, requires_grad=True)

for i in range(100):
    params, energy = opt.step_and_cost(vqe_circuit, params)
    print(f"步骤 {i}: 能量 = {energy:.6f} Ha")
```

### 设备切换

```python
# 相同电路，不同后端
circuit_def = lambda dev: qml.qnode(dev)(circuit_function)

# 模拟器测试
dev_sim = qml.device('default.qubit', wires=4)
result_sim = circuit_def(dev_sim)(params)

# 量子硬件运行
dev_hw = qml.device('qiskit.ibmq', wires=4, backend='ibmq_manila')
result_hw = circuit_def(dev_hw)(params)
```

## 详细文档

特定主题的完整文档请查阅参考文件：

- **入门指南**：`references/getting_started.md` - 安装、基础概念、第一步操作
- **量子电路**：`references/quantum_circuits.md` - 门操作、测量、电路模式
- **量子机器学习**：`references/quantum_ml.md` - 混合模型、框架集成、量子神经网络
- **量子化学**：`references/quantum_chemistry.md` - VQE、分子哈密顿量、化学工作流
- **设备管理**：`references/devices_backends.md` - 模拟器、硬件插件、设备配置
- **优化方法**：`references/optimization.md` - 优化器、梯度计算、变分算法
- **高级功能**：`references/advanced_features.md` - 模板、变换、即时编译、噪声

## 最佳实践

1. **从模拟器开始** - 硬件部署前先在 `default.qubit` 测试
2. **硬件使用参数平移法** - 反向传播仅适用于模拟器
3. **选择合适编码方式** - 根据问题结构匹配数据编码
4. **谨慎初始化** - 使用小随机值避免贫瘠高原
5. **监控梯度** - 检查深层电路中的梯度消失
6. **缓存设备对象** - 复用设备以减少初始化开销
7. **分析电路性能** - 使用 `qml.specs()` 评估电路复杂度
8. **本地测试** - 提交硬件前在模拟器验证
9. **使用模板** - 利用内置模板实现通用电路模式
10. **适时编译** - 性能关键代码使用Catalyst即时编译

## 资源

- 官方文档：https://docs.pennylane.ai
- 代码手册（教程）：https://pennylane.ai/codebook
- 量子机器学习示例：https://pennylane.ai/qml/demonstrations
- 社区论坛：https://discuss.pennylane.ai
- GitHub仓库：https://github.com/PennyLaneAI/pennylane
