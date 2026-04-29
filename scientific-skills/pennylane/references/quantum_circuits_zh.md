# PennyLane中的量子电路

## 目录
1. [基础门与操作](#basic-gates-and-operations)
2. [多量子比特门](#multi-qubit-gates)
3. [受控操作](#controlled-operations)
4. [测量操作](#measurements)
5. [电路构建模式](#circuit-construction-patterns)
6. [动态电路](#dynamic-circuits)
7. [电路检查](#circuit-inspection)

## 基础门与操作

### 单量子比特门

```python
import pennylane as qml

# 泡利门
qml.PauliX(wires=0)  # X门（比特翻转）
qml.PauliY(wires=0)  # Y门
qml.PauliZ(wires=0)  # Z门（相位翻转）

# Hadamard门（叠加态）
qml.Hadamard(wires=0)

# 相位门
qml.S(wires=0)       # S门（π/2相位）
qml.T(wires=0)       # T门（π/4相位）
qml.PhaseShift(phi, wires=0)  # 任意相位

# 旋转门（参数化）
qml.RX(theta, wires=0)  # X轴旋转
qml.RY(theta, wires=0)  # Y轴旋转
qml.RZ(theta, wires=0)  # Z轴旋转

# 通用单量子比特旋转
qml.Rot(phi, theta, omega, wires=0)

# 通用门（任意单量子比特酉变换）
qml.U3(theta, phi, delta, wires=0)
```

### 基态制备

```python
# 计算基态
qml.BasisState([1, 0, 1], wires=[0, 1, 2])  # |101⟩

# 振幅编码
amplitudes = [0.5, 0.5, 0.5, 0.5]  # 需归一化
qml.MottonenStatePreparation(amplitudes, wires=[0, 1])
```

## 多量子比特门

### 双量子比特门

```python
# CNOT（受控非门）
qml.CNOT(wires=[0, 1])  # 控制位=0, 目标位=1

# CZ（受控Z门）
qml.CZ(wires=[0, 1])

# SWAP门
qml.SWAP(wires=[0, 1])

# 受控旋转门
qml.CRX(theta, wires=[0, 1])
qml.CRY(theta, wires=[0, 1])
qml.CRZ(theta, wires=[0, 1])

# Ising耦合门
qml.IsingXX(phi, wires=[0, 1])
qml.IsingYY(phi, wires=[0, 1])
qml.IsingZZ(phi, wires=[0, 1])
```

### 多量子比特门

```python
# Toffoli门（CCNOT）
qml.Toffoli(wires=[0, 1, 2])  # 控制位=0,1, 目标位=2

# 多控X门
qml.MultiControlledX(control_wires=[0, 1, 2], wires=3)

# 多量子比特泡利旋转
qml.MultiRZ(theta, wires=[0, 1, 2])
```

## 受控操作

### 通用受控操作

```python
# 应用任意操作的受控版本
qml.ctrl(qml.RX(0.5, wires=1), control=0)

# 多控制量子比特
qml.ctrl(qml.RY(0.3, wires=2), control=[0, 1])

# 负控制（当控制位为|0⟩时激活）
qml.ctrl(qml.Hadamard(wires=2), control=0, control_values=[0])
```

### 条件操作

```python
@qml.qnode(dev)
def conditional_circuit():
    qml.Hadamard(wires=0)

    # 电路中途测量
    m = qml.measure(0)

    # 条件应用门操作
    qml.cond(m, qml.PauliX)(wires=1)

    return qml.expval(qml.PauliZ(1))
```

## 测量操作

### 期望值

```python
@qml.qnode(dev)
def measure_expectation():
    qml.Hadamard(wires=0)

    # 单观测值
    return qml.expval(qml.PauliZ(0))

@qml.qnode(dev)
def measure_tensor():
    qml.Hadamard(wires=0)
    qml.Hadamard(wires=1)

    # 观测值张量积
    return qml.expval(qml.PauliZ(0) @ qml.PauliZ(1))
```

### 概率分布

```python
@qml.qnode(dev)
def measure_probabilities():
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])

    # 所有基态概率
    return qml.probs(wires=[0, 1])  # 返回 [p(|00⟩), p(|01⟩), p(|10⟩), p(|11⟩)]
```

### 采样与计数

```python
@qml.qnode(dev)
def measure_samples(shots=1000):
    qml.Hadamard(wires=0)

    # 原始采样
    return qml.sample(qml.PauliZ(0))

@qml.qnode(dev)
def measure_counts(shots=1000):
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])

    # 出现次数统计
    return qml.counts(wires=[0, 1])
```

### 方差

```python
@qml.qnode(dev)
def measure_variance():
    qml.RX(0.5, wires=0)

    # 观测值方差
    return qml.var(qml.PauliZ(0))
```

### 电路中途测量

```python
@qml.qnode(dev)
def mid_circuit_measure():
    qml.Hadamard(wires=0)

    # 电路中途测量量子比特0
    m0 = qml.measure(0)

    # 使用测量结果
    qml.cond(m0, qml.PauliX)(wires=1)

    # 最终测量
    return qml.expval(qml.PauliZ(1))
```

## 电路构建模式

### 分层构建

```python
def layer(weights, wires):
    """参数化门的单层结构"""
    for i, wire in enumerate(wires):
        qml.RY(weights[i], wires=wire)

    for wire in wires[:-1]:
        qml.CNOT(wires=[wire, wire+1])

@qml.qnode(dev)
def layered_circuit(weights):
    n_layers = len(weights)
    wires = range(4)

    for i in range(n_layers):
        layer(weights[i], wires)

    return qml.expval(qml.PauliZ(0))
```

### 数据编码

```python
def angle_encoding(x, wires):
    """将经典数据编码为旋转角度"""
    for i, wire in enumerate(wires):
        qml.RX(x[i], wires=wire)

def amplitude_encoding(x, wires):
    """将数据编码为量子态振幅"""
    qml.MottonenStatePreparation(x, wires=wires)

def basis_encoding(x, wires):
    """在计算基中编码二进制数据"""
    for i, val in enumerate(x):
        if val:
            qml.PauliX(wires=i)
```

### 拟设模式

```python
# 硬件高效拟设
def hardware_efficient_ansatz(weights, wires):
    n_layers = len(weights) // len(wires)

    for layer in range(n_layers):
        # 旋转层
        for i, wire in enumerate(wires):
            qml.RY(weights[layer * len(wires) + i], wires=wire)

        # 纠缠层
        for wire in wires[:-1]:
            qml.CNOT(wires=[wire, wire+1])

# 交替分层拟设
def alternating_ansatz(weights, wires):
    for w in weights:
        for wire in wires:
            qml.RX(w[wire], wires=wire)
        for wire in wires[:-1]:
            qml.CNOT(wires=[wire, wire+1])
```

## 动态电路

### For循环

```python
@qml.qnode(dev)
def dynamic_for_loop(n_iterations):
    qml.Hadamard(wires=0)

    # 动态for循环
    for i in range(n_iterations):
        qml.RX(0.1 * i, wires=0)

    return qml.expval(qml.PauliZ(0))
```

### While循环（使用Catalyst）

```python
@qml.qjit  # 即时编译
@qml.qnode(dev)
def dynamic_while_loop():
    qml.Hadamard(wires=0)

    # 动态while循环
    @qml.while_loop(lambda i: i < 5)
    def loop(i):
        qml.RX(0.1, wires=0)
        return i + 1

    loop(0)
    return qml.expval(qml.PauliZ(0))
```

### 自适应电路

```python
@qml.qnode(dev)
def adaptive_circuit():
    qml.Hadamard(wires=0)

    # 测量并调整
    m = qml.measure(0)

    # 基于测量的不同路径
    if m:
        qml.RX(0.5, wires=1)
    else:
        qml.RY(0.5, wires=1)

    return qml.expval(qml.PauliZ(1))
```

## 电路检查

### 绘制电路

```python
# 文本表示
print(qml.draw(circuit)(params))

# ASCII图形
print(qml.draw(circuit, wire_order=[0,1,2])(params))

# Matplotlib可视化
fig, ax = qml.draw_mpl(circuit)(params)
```

### 分析电路结构

```python
# 获取电路规格
specs = qml.specs(circuit)(params)
print(f"门数量: {specs['gate_sizes']}")
print(f"深度: {specs['depth']}")
print(f"参数: {specs['num_trainable_params']}")

# 资源估算
resources = qml.resource.resource_estimation(circuit)(params)
print(f"总门数: {resources['num_gates']}")
```

### 磁带检查

```python
# 记录操作
with qml.tape.QuantumTape() as tape:
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])
    qml.expval(qml.PauliZ(0))

# 检查磁带内容
print("操作序列:", tape.operations)
print("测量操作:", tape.measurements)
print("使用的量子线:", tape.wires)
```

### 电路变换

```python
# 展开复合操作
expanded = qml.transforms.expand_tape(tape)

# 取消相邻逆操作
optimized = qml.transforms.cancel_inverses(tape)

# 将测量移至末端
commuted = qml.transforms.commute_controlled(tape)
```

## 最佳实践

1. **使用原生门** - 优先选择目标设备支持的门
2. **最小化电路深度** - 减少退相干效应
3. **高效编码** - 选择匹配数据结构的编码方式
4. **复用电路** - 尽可能缓存编译后的电路
5. **验证测量** - 确保观测量为厄米算符
6. **检查量子比特数** - 确认设备有足够量子线
7. **性能分析** - 使用`qml.specs()`分析复杂度

## 常用模式

### Bell态制备

```python
@qml.qnode(dev)
def bell_state():
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])
    return qml.state()  # 返回 |Φ+⟩ = (|00⟩ + |11⟩)/√2
```

### GHZ态

```python
@qml.qnode(dev)
def ghz_state(n_qubits):
    qml.Hadamard(wires=0)
    for i in range(n_qubits-1):
        qml.CNOT(wires=[0, i+1])
    return qml.state()
```

### 量子傅里叶变换

```python
def qft(wires):
    """量子傅里叶变换"""
    n_wires = len(wires)
    for i in range(n_wires):
        qml.Hadamard(wires=wires[i])
        for j in range(i+1, n_wires):
            qml.CRZ(np.pi / (2**(j-i)), wires=[wires[j], wires[i]])
```

### 逆量子傅里叶变换

```python
def inverse_qft(wires):
    """逆量子傅里叶变换"""
    n_wires = len(wires)
    for i in range(n_wires-1, -1, -1):
        for j in range(n_wires-1, i, -1):
            qml.CRZ(-np.pi / (2**(j-i)), wires=[wires[j], wires[i]])
        qml.Hadamard(wires=wires[i])
```
