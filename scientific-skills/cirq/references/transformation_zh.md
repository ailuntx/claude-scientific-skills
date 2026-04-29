# 电路变换

本指南介绍使用 Cirq 的变换框架进行电路优化、编译和操作。

## 变换器框架

### 基础变换器

```python
import cirq

# 示例电路
qubits = cirq.LineQubit.range(3)
circuit = cirq.Circuit(
    cirq.H(qubits[0]),
    cirq.CNOT(qubits[0], qubits[1]),
    cirq.CNOT(qubits[1], qubits[2])
)

# 应用内置变换器
from cirq.transformers import optimize_for_target_gateset

# 优化到特定门集合
optimized = optimize_for_target_gateset(
    circuit,
    gateset=cirq.SqrtIswapTargetGateset()
)
```

### 合并单量子比特门

```python
from cirq.transformers import merge_single_qubit_gates_to_phxz

# 含多个单量子比特门的电路
circuit = cirq.Circuit(
    cirq.H(q),
    cirq.T(q),
    cirq.S(q),
    cirq.H(q)
)

# 合并为单一操作
merged = merge_single_qubit_gates_to_phxz(circuit)
```

### 丢弃可忽略操作

```python
from cirq.transformers import drop_negligible_operations

# 移除低于阈值的门
circuit_with_small_rotations = cirq.Circuit(
    cirq.rz(1e-10)(q),  # 极小旋转
    cirq.H(q)
)

cleaned = drop_negligible_operations(circuit_with_small_rotations, atol=1e-8)
```

## 自定义变换器

### 变换器装饰器

```python
from cirq.transformers import transformer_api

@transformer_api.transformer
def remove_z_gates(circuit: cirq.Circuit) -> cirq.Circuit:
    """移除电路中所有Z门"""
    new_moments = []
    for moment in circuit:
        new_ops = [op for op in moment if not isinstance(op.gate, cirq.ZPowGate)]
        if new_ops:
            new_moments.append(cirq.Moment(new_ops))
    return cirq.Circuit(new_moments)

# 使用自定义变换器
transformed = remove_z_gates(circuit)
```

### 变换器类

```python
from cirq.transformers import transformer_primitives

class HToRyTransformer(transformer_primitives.Transformer):
    """用Ry(π/2)替换H门"""

    def __call__(self, circuit: cirq.Circuit, *, context=None) -> cirq.Circuit:
        def map_op(op: cirq.Operation, _) -> cirq.OP_TREE:
            if isinstance(op.gate, cirq.HPowGate):
                return cirq.ry(np.pi/2)(op.qubits[0])
            return op

        return transformer_primitives.map_operations(
            circuit,
            map_op,
            deep=True
        ).unfreeze(copy=False)

# 应用变换器
transformer = HToRyTransformer()
result = transformer(circuit)
```

## 门分解

### 分解到目标门集合

```python
from cirq.transformers import optimize_for_target_gateset

# 分解为CZ门和单量子比特旋转
target_gateset = cirq.CZTargetGateset()
decomposed = optimize_for_target_gateset(circuit, gateset=target_gateset)

# 分解为√iSWAP门
sqrt_iswap_gateset = cirq.SqrtIswapTargetGateset()
decomposed = optimize_for_target_gateset(circuit, gateset=sqrt_iswap_gateset)
```

### 自定义门分解

```python
class Toffoli(cirq.Gate):
    def _num_qubits_(self):
        return 3

    def _decompose_(self, qubits):
        """将Toffoli门分解为基础门"""
        c1, c2, t = qubits
        return [
            cirq.H(t),
            cirq.CNOT(c2, t),
            cirq.T(t)**-1,
            cirq.CNOT(c1, t),
            cirq.T(t),
            cirq.CNOT(c2, t),
            cirq.T(t)**-1,
            cirq.CNOT(c1, t),
            cirq.T(c2),
            cirq.T(t),
            cirq.H(t),
            cirq.CNOT(c1, c2),
            cirq.T(c1),
            cirq.T(c2)**-1,
            cirq.CNOT(c1, c2)
        ]

# 使用分解
circuit = cirq.Circuit(Toffoli()(q0, q1, q2))
decomposed = cirq.decompose(circuit)
```

## 电路优化

### 弹出Z门

```python
from cirq.transformers import eject_z

# 将Z门移至电路末端
circuit = cirq.Circuit(
    cirq.H(q0),
    cirq.Z(q0),
    cirq.CNOT(q0, q1)
)

ejected = eject_z(circuit)
```

### 弹出相位门

```python
from cirq.transformers import eject_phased_paulis

# 合并相位门
optimized = eject_phased_paulis(circuit, atol=1e-8)
```

### 丢弃空时序

```python
from cirq.transformers import drop_empty_moments

# 移除无操作的空时序
cleaned = drop_empty_moments(circuit)
```

### 对齐测量

```python
from cirq.transformers import dephase_measurements

# 将测量移至末端并移除后续操作
aligned = dephase_measurements(circuit)
```

## 电路编译

### 硬件编译

```python
import cirq_google

# 获取设备规格
device = cirq_google.Sycamore

# 将电路编译到设备
from cirq.transformers import optimize_for_target_gateset

compiled = optimize_for_target_gateset(
    circuit,
    gateset=cirq_google.SycamoreTargetGateset()
)

# 验证编译后电路
device.validate_circuit(compiled)
```

### 双量子比特门编译

```python
# 编译到特定双量子比特门
from cirq import two_qubit_to_cz

# 将所有双量子比特门转换为CZ门
cz_circuit = cirq.Circuit()
for moment in circuit:
    for op in moment:
        if len(op.qubits) == 2:
            cz_circuit.append(two_qubit_to_cz(op))
        else:
            cz_circuit.append(op)
```

## 量子比特路由

### 路由到设备拓扑

```python
from cirq.transformers import route_circuit

# 定义设备连接性
device_graph = cirq.NamedTopology(
    {
        (0, 0): [(0, 1), (1, 0)],
        (0, 1): [(0, 0), (1, 1)],
        (1, 0): [(0, 0), (1, 1)],
        (1, 1): [(0, 1), (1, 0)]
    }
)

# 将逻辑量子比特路由到物理量子比特
routed_circuit = route_circuit(
    circuit,
    device_graph=device_graph,
    routing_algo=cirq.RouteCQC(device_graph)
)
```

### 插入SWAP网络

```python
# 手动插入SWAP门进行路由
def insert_swaps(circuit, swap_locations):
    """在指定位置插入SWAP门"""
    new_circuit = cirq.Circuit()
    moment_idx = 0

    for i, moment in enumerate(circuit):
        if i in swap_locations:
            q0, q1 = swap_locations[i]
            new_circuit.append(cirq.SWAP(q0, q1))
        new_circuit.append(moment)

    return new_circuit
```

## 高级变换

### 酉矩阵编译

```python
import scipy.linalg

# 将任意酉矩阵编译为门序列
def compile_unitary(unitary, qubits):
    """使用KAK分解编译2x2酉矩阵"""
    from cirq.linalg import kak_decomposition

    decomp = kak_decomposition(unitary)
    operations = []

    # 添加前置单量子比特门
    operations.append(cirq.MatrixGate(decomp.single_qubit_operations_before[0])(qubits[0]))
    operations.append(cirq.MatrixGate(decomp.single_qubit_operations_before[1])(qubits[1]))

    # 添加交互（双量子比特）部分
    x, y, z = decomp.interaction_coefficients
    operations.append(cirq.XXPowGate(exponent=x/np.pi)(qubits[0], qubits[1]))
    operations.append(cirq.YYPowGate(exponent=y/np.pi)(qubits[0], qubits[1]))
    operations.append(cirq.ZZPowGate(exponent=z/np.pi)(qubits[0], qubits[1]))

    # 添加后置单量子比特门
    operations.append(cirq.MatrixGate(decomp.single_qubit_operations_after[0])(qubits[0]))
    operations.append(cirq.MatrixGate(decomp.single_qubit_operations_after[1])(qubits[1]))

    return operations
```

### 电路简化

```python
from cirq.transformers import (
    merge_k_qubit_unitaries,
    merge_single_qubit_gates_to_phxz
)

# 合并相邻单量子比特门
simplified = merge_single_qubit_gates_to_phxz(circuit)

# 合并相邻k量子比特酉操作
simplified = merge_k_qubit_unitaries(circuit, k=2)
```

### 基于交换律的优化

```python
# 使Z门通过CNOT门交换
def commute_z_through_cnot(circuit):
    """使Z门通过CNOT门交换"""
    new_moments = []

    for moment in circuit:
        ops = list(moment)
        # 找出CNOT前的Z门
        z_ops = [op for op in ops if isinstance(op.gate, cirq.ZPowGate)]
        cnot_ops = [op for op in ops if isinstance(op.gate, cirq.CXPowGate)]

        # 应用交换规则
        # 控制位上的Z可交换，目标位上的Z反对易
        # (此处为简化逻辑)

        new_moments.append(cirq.Moment(ops))

    return cirq.Circuit(new_moments)
```

## 变换流水线

### 组合多个变换器

```python
from cirq.transformers import transformer_api

# 构建变换流水线
@transformer_api.transformer
def optimization_pipeline(circuit: cirq.Circuit) -> cirq.Circuit:
    # 步骤1：合并单量子比特门
    circuit = merge_single_qubit_gates_to_phxz(circuit)

    # 步骤2：丢弃可忽略操作
    circuit = drop_negligible_operations(circuit)

    # 步骤3：弹出Z门
    circuit = eject_z(circuit)

    # 步骤4：丢弃空时序
    circuit = drop_empty_moments(circuit)

    return circuit

# 应用流水线
optimized = optimization_pipeline(circuit)
```

## 验证与分析

### 电路深度缩减

```python
# 测量优化前后的电路深度
print(f"原始深度: {len(circuit)}")
optimized = optimization_pipeline(circuit)
print(f"优化后深度: {len(optimized)}")
```

### 门数量分析

```python
def count_gates(circuit):
    """按类型统计门数量"""
    counts = {}
    for moment in circuit:
        for op in moment:
            gate_type = type(op.gate).__name__
            counts[gate_type] = counts.get(gate_type, 0) + 1
    return counts

original_counts = count_gates(circuit)
optimized_counts = count_gates(optimized)
print(f"原始: {original_counts}")
print(f"优化后: {optimized_counts}")
```

## 最佳实践

1. **从高级变换器开始**：在编写自定义变换器前优先使用内置变换器
2. **链式变换器**：按顺序应用多个优化
3. **变换后验证**：确保电路正确性和设备兼容性
4. **量化改进**：跟踪深度和门数量缩减
5. **使用合适门集合**：匹配目标硬件能力
6. **考虑交换律**：利用门交换律进行优化
7. **先在小电路测试**：扩展前验证变换器正确性
