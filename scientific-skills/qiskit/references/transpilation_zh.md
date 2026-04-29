# 电路转译与优化

转译是将量子电路重写以匹配特定量子设备的拓扑结构和门集合的过程，同时针对含噪声量子计算机的执行进行优化。

## 为何需要转译？

**问题**：抽象量子电路可能使用硬件不支持的逻辑门，并假设全连接量子比特拓扑。

**解决方案**：转译通过以下方式改造电路：
1. 仅使用硬件原生门（基础门集）
2. 遵循物理量子比特连接约束
3. 最小化电路深度和门数量
4. 针对含噪声设备优化以减少错误

## 基础转译操作

### 简单转译

```python
from qiskit import QuantumCircuit, transpile

qc = QuantumCircuit(3)
qc.h(0)
qc.cx(0, 1)
qc.cx(1, 2)

# 针对特定后端进行转译
transpiled_qc = transpile(qc, backend=backend)
```

### 优化级别

选择0-3级优化：

```python
# 级别0：无优化（最快）
qc_0 = transpile(qc, backend=backend, optimization_level=0)

# 级别1：轻度优化
qc_1 = transpile(qc, backend=backend, optimization_level=1)

# 级别2：中等优化（默认）
qc_2 = transpile(qc, backend=backend, optimization_level=2)

# 级别3：深度优化（最慢，效果最佳）
qc_3 = transpile(qc, backend=backend, optimization_level=3)
```

**Qiskit SDK v2.2** 提供比竞品 **快83倍的转译速度**。

## 转译阶段流程

转译器流水线包含六个阶段：

### 1. 初始化阶段
- 验证电路指令有效性
- 将多量子比特门转换为标准形式

### 2. 布局阶段
- 将虚拟量子比特映射到物理量子比特
- 考虑量子比特连接性和错误率

```python
from qiskit.transpiler import CouplingMap

# 定义自定义连接图
coupling = CouplingMap([(0, 1), (1, 2), (2, 3)])
qc_transpiled = transpile(qc, coupling_map=coupling)
```

### 3. 路由阶段
- 插入SWAP门以满足连接约束
- 最小化额外SWAP开销

### 4. 翻译阶段
- 将逻辑门转换为硬件基础门集
- 典型基础门集：{RZ, SX, X, CX}

```python
# 指定基础门集
basis_gates = ['cx', 'id', 'rz', 'sx', 'x']
qc_transpiled = transpile(qc, basis_gates=basis_gates)
```

### 5. 优化阶段
- 减少门数量和电路深度
- 应用门消除和对易规则
- 使用**虚拟置换省略**技术（级别2-3）
- 分解可分离操作

### 6. 调度阶段
- 添加时序信息用于脉冲级控制

## 高级优化特性

### 虚拟置换省略

在2-3级优化中，Qiskit通过分析对易结构，跟踪虚拟量子比特置换来消除不必要的SWAP门。

### 门消除

识别并移除相互抵消的门对：
- X-X → I
- H-H → I
- CNOT-CNOT → I

### 数值分解

将可表示为分离单量子比特操作的双量子比特门进行拆分。

## 常用转译参数

### 初始布局

指定使用的物理量子比特：

```python
# 使用特定物理量子比特
initial_layout = [0, 2, 4]  # 将电路量子比特0,1,2映射到物理量子比特0,2,4
qc_transpiled = transpile(qc, backend=backend, initial_layout=initial_layout)
```

### 近似度

在精度与门数量间权衡（0.0=最大近似，1.0=无近似）：

```python
# 允许5%近似误差以减少门数量
qc_transpiled = transpile(qc, backend=backend, approximation_degree=0.95)
```

### 随机种子（确保可复现性）

```python
qc_transpiled = transpile(qc, backend=backend, seed_transpiler=42)
```

### 调度方法

```python
# 添加时序约束
qc_transpiled = transpile(
    qc,
    backend=backend,
    scheduling_method='alap'  # 尽可能晚调度
)
```

## 面向模拟器的转译

即使对模拟器，转译也能优化电路：

```python
from qiskit_aer import AerSimulator

simulator = AerSimulator()
qc_optimized = transpile(qc, backend=simulator, optimization_level=3)

# 比较门数量
print(f"原始电路: {qc.size()} 门")
print(f"优化后: {qc_optimized.size()} 门")
```

## 目标感知转译

使用`Target`对象进行详细后端配置：

```python
from qiskit.transpiler import Target

# 使用目标规范进行转译
qc_transpiled = transpile(qc, target=backend.target)
```

## 转译后电路分析

```python
qc_transpiled = transpile(qc, backend=backend, optimization_level=3)

# 分析结果
print(f"深度: {qc_transpiled.depth()}")
print(f"门数量: {qc_transpiled.size()}")
print(f"操作类型: {qc_transpiled.count_ops()}")

# 检查双量子比特门数量（主要错误源）
two_qubit_gates = qc_transpiled.count_ops().get('cx', 0)
print(f"双量子比特门: {two_qubit_gates}")
```

**Qiskit生成的双量子比特门数量比主流方案少29%**，显著降低错误率。

## 多电路转译

高效转译多个电路：

```python
circuits = [qc1, qc2, qc3]
transpiled_circuits = transpile(
    circuits,
    backend=backend,
    optimization_level=3
)
```

## 转译前最佳实践

### 1. 基于硬件拓扑设计电路

设计时考虑后端连接图：

```python
# 查看后端连接图
print(backend.coupling_map)

# 设计与连接图匹配的电路
```

### 2. 优先使用原生门

某些后端支持{CX, RZ, SX, X}以外的门：

```python
# 检查可用基础门集
print(backend.configuration().basis_gates)
```

### 3. 最小化双量子比特门

双量子比特门错误率显著更高：
- 设计算法时减少CNOT门使用
- 利用门恒等式降低数量

### 4. 先用模拟器测试

```python
from qiskit_aer import AerSimulator

# 本地测试转译
sim_backend = AerSimulator.from_backend(backend)
qc_test = transpile(qc, backend=sim_backend, optimization_level=3)
```

## 不同供应商的转译方案

### IBM Quantum

```python
from qiskit_ibm_runtime import QiskitRuntimeService

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")
qc_transpiled = transpile(qc, backend=backend)
```

### IonQ

```python
# IonQ具有全连接拓扑，基础门集不同
basis_gates = ['gpi', 'gpi2', 'ms']
qc_transpiled = transpile(qc, basis_gates=basis_gates)
```

### Amazon Braket

转译方案取决于具体设备（Rigetti, IonQ等）

## 性能优化技巧

1. **缓存转译结果** - 转译开销大，尽可能复用
2. **选择合适的优化级别** - 级别3速度慢但适合生产环境
3. **利用v2.2速度优势** - 升级至最新Qiskit获得83倍加速
4. **并行化转译** - Qiskit在转译多电路时自动并行处理

## 常见问题与解决方案

### 问题：转译后电路过深
**解决方案**：提高优化级别或重新设计减少电路层数

### 问题：插入过多SWAP门
**解决方案**：调整initial_layout以更好匹配量子比特拓扑

### 问题：转译耗时过长
**解决方案**：降低优化级别或升级至Qiskit v2.2+

### 问题：出现意外门分解
**解决方案**：检查basis_gates并考虑指定自定义分解规则
