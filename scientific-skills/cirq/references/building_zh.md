# 构建量子电路

本指南涵盖 Cirq 中的电路构建，包括量子比特、量子门、操作和电路模式。

## 基础电路构建

### 创建电路

```python
import cirq

# 创建电路
circuit = cirq.Circuit()

# 创建量子比特
q0 = cirq.GridQubit(0, 0)
q1 = cirq.GridQubit(0, 1)
q2 = cirq.LineQubit(0)

# 向电路添加量子门
circuit.append([
    cirq.H(q0),
    cirq.CNOT(q0, q1),
    cirq.measure(q0, q1, key='result')
])
```

### 量子比特类型

**GridQubit**：类硬件布局的二维网格拓扑
```python
qubits = cirq.GridQubit.square(2)  # 2x2网格
qubit = cirq.GridQubit(row=0, col=1)
```

**LineQubit**：一维线性拓扑
```python
qubits = cirq.LineQubit.range(5)  # 5个线性排列的量子比特
qubit = cirq.LineQubit(3)
```

**NamedQubit**：自定义命名的量子比特
```python
qubit = cirq.NamedQubit('my_qubit')
```

## 常用量子门与操作

### 单量子比特门

```python
# 泡利门
cirq.X(qubit)  # NOT门
cirq.Y(qubit)
cirq.Z(qubit)

# 哈达玛门
cirq.H(qubit)

# 旋转门
cirq.rx(angle)(qubit)  # X轴旋转
cirq.ry(angle)(qubit)  # Y轴旋转
cirq.rz(angle)(qubit)  # Z轴旋转

# 相位门
cirq.S(qubit)  # √Z门
cirq.T(qubit)  # ⁴√Z门
```

### 双量子比特门

```python
# CNOT（受控非门）
cirq.CNOT(control, target)
cirq.CX(control, target)  # 别名

# CZ（受控Z门）
cirq.CZ(q0, q1)

# SWAP（交换门）
cirq.SWAP(q0, q1)

# iSWAP
cirq.ISWAP(q0, q1)

# 受控旋转门
cirq.CZPowGate(exponent=0.5)(q0, q1)
```

### 测量操作

```python
# 测量单个量子比特
cirq.measure(qubit, key='m')

# 测量多个量子比特
cirq.measure(q0, q1, q2, key='result')

# 测量电路中所有量子比特
circuit.append(cirq.measure(*qubits, key='final'))
```

## 高级电路构建

### 参数化量子门

```python
import sympy

# 创建符号参数
theta = sympy.Symbol('theta')
phi = sympy.Symbol('phi')

# 在量子门中使用
circuit = cirq.Circuit(
    cirq.rx(theta)(q0),
    cirq.ry(phi)(q1),
    cirq.CNOT(q0, q1)
)

# 后续解析参数
resolved = cirq.resolve_parameters(circuit, {'theta': 0.5, 'phi': 1.2})
```

### 通过酉矩阵创建自定义门

```python
import numpy as np

# 定义酉矩阵
unitary = np.array([
    [1, 0, 0, 0],
    [0, 1, 0, 0],
    [0, 0, 0, 1],
    [0, 0, 1, 0]
]) / np.sqrt(2)

# 从酉矩阵创建量子门
gate = cirq.MatrixGate(unitary)
operation = gate(q0, q1)
```

### 量子门分解

```python
# 定义带分解逻辑的自定义门
class MyGate(cirq.Gate):
    def _num_qubits_(self):
        return 1

    def _decompose_(self, qubits):
        q = qubits[0]
        return [cirq.H(q), cirq.T(q), cirq.H(q)]

    def _circuit_diagram_info_(self, args):
        return 'MyGate'

# 使用自定义门
my_gate = MyGate()
circuit.append(my_gate(q0))
```

## 电路组织结构

### 时序单元

电路按时序单元组织（并行操作）：

```python
# 显式构建时序单元
circuit = cirq.Circuit(
    cirq.Moment([cirq.H(q0), cirq.H(q1)]),
    cirq.Moment([cirq.CNOT(q0, q1)]),
    cirq.Moment([cirq.measure(q0, key='m0'), cirq.measure(q1, key='m1')])
)

# 访问时序单元
for i, moment in enumerate(circuit):
    print(f"时序单元 {i}: {moment}")
```

### 电路操作

```python
# 连接电路
circuit3 = circuit1 + circuit2

# 插入操作
circuit.insert(index, operation)

# 按策略追加操作
circuit.append(operations, strategy=cirq.InsertStrategy.NEW_THEN_INLINE)
```

## 电路模式

### 贝尔态制备

```python
def bell_state_circuit():
    q0, q1 = cirq.LineQubit.range(2)
    return cirq.Circuit(
        cirq.H(q0),
        cirq.CNOT(q0, q1)
    )
```

### GHZ态

```python
def ghz_circuit(qubits):
    circuit = cirq.Circuit()
    circuit.append(cirq.H(qubits[0]))
    for i in range(len(qubits) - 1):
        circuit.append(cirq.CNOT(qubits[i], qubits[i+1]))
    return circuit
```

### 量子傅里叶变换

```python
def qft_circuit(qubits):
    circuit = cirq.Circuit()
    for i, q in enumerate(qubits):
        circuit.append(cirq.H(q))
        for j in range(i + 1, len(qubits)):
            circuit.append(cirq.CZPowGate(exponent=1/2**(j-i))(qubits[j], q))

    # 反转量子比特顺序
    for i in range(len(qubits) // 2):
        circuit.append(cirq.SWAP(qubits[i], qubits[len(qubits) - i - 1]))

    return circuit
```

## 电路导入/导出

### OpenQASM

```python
# 导出为QASM
qasm_str = circuit.to_qasm()

# 从QASM导入
from cirq.contrib.qasm_import import circuit_from_qasm
circuit = circuit_from_qasm(qasm_str)
```

### 电路JSON

```python
import json

# 序列化
json_str = cirq.to_json(circuit)

# 反序列化
circuit = cirq.read_json(json_text=json_str)
```

## 使用量子dit

量子dit是高维量子系统（qutrit、ququart等）：

```python
# 创建qutrit（三能级系统）
qutrit = cirq.LineQid(0, dimension=3)

# 自定义qutrit门
class QutritXGate(cirq.Gate):
    def _qid_shape_(self):
        return (3,)

    def _unitary_(self):
        return np.array([
            [0, 0, 1],
            [1, 0, 0],
            [0, 1, 0]
        ])

gate = QutritXGate()
circuit = cirq.Circuit(gate(qutrit))
```

## 可观测量

从泡利算符创建可观测量：

```python
# 单泡利可观测量
obs = cirq.Z(q0)

# 泡利字符串
obs = cirq.X(q0) * cirq.Y(q1) * cirq.Z(q2)

# 线性组合
from cirq import PauliSum
obs = 0.5 * cirq.X(q0) + 0.3 * cirq.Z(q1)
```

## 最佳实践

1. **选用合适的量子比特类型**：类硬件拓扑用GridQubit，一维问题用LineQubit
2. **保持电路模块化**：构建可复用的电路函数
3. **使用符号参数**：便于参数扫描和优化
4. **清晰标记测量**：为测量结果使用描述性键名
5. **记录自定义门**：包含电路图信息以支持可视化
