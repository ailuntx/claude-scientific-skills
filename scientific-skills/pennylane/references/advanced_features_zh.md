# PennyLane 高级特性

## 目录
1. [模板与层](#模板与层)
2. [变换](#变换)
3. [脉冲编程](#脉冲编程)
4. [Catalyst 与即时编译](#catalyst-与jit编译)
5. [自适应电路](#自适应电路)
6. [噪声模型](#噪声模型)
7. [资源估算](#资源估算)

## 模板与层

### 内置模板

```python
import pennylane as qml
from pennylane.templates import *
from pennylane import numpy as np

dev = qml.device('default.qubit', wires=4)

# 强纠缠层
@qml.qnode(dev)
def circuit_sel(weights):
    StronglyEntanglingLayers(weights, wires=range(4))
    return qml.expval(qml.PauliZ(0))

# 生成适当形状的权重
n_layers = 3
n_wires = 4
shape = StronglyEntanglingLayers.shape(n_layers, n_wires)
weights = np.random.random(shape)

result = circuit_sel(weights)
```

### 基础纠缠层

```python
@qml.qnode(dev)
def circuit_bel(weights):
    # 简单纠缠层
    BasicEntanglerLayers(weights, wires=range(4))
    return qml.expval(qml.PauliZ(0))

n_layers = 2
weights = np.random.random((n_layers, 4))
```

### 随机层

```python
@qml.qnode(dev)
def circuit_random(weights):
    # 随机电路结构
    RandomLayers(weights, wires=range(4))
    return qml.expval(qml.PauliZ(0))

n_layers = 5
weights = np.random.random((n_layers, 4))
```

### 简化双设计

```python
@qml.qnode(dev)
def circuit_s2d(weights):
    # 简化双设计
    SimplifiedTwoDesign(initial_layer_weights=weights[0],
                       weights=weights[1:],
                       wires=range(4))
    return qml.expval(qml.PauliZ(0))
```

### 粒子守恒层

```python
@qml.qnode(dev)
def circuit_particle_conserving(weights):
    # 保持粒子数（适用于化学计算）
    ParticleConservingU1(weights, wires=range(4))
    return qml.expval(qml.PauliZ(0))

shape = ParticleConservingU1.shape(n_layers=2, n_wires=4)
weights = np.random.random(shape)
```

### 嵌入模板

```python
# 角度嵌入
@qml.qnode(dev)
def angle_embed(features):
    AngleEmbedding(features, wires=range(4))
    return qml.expval(qml.PauliZ(0))

features = np.array([0.1, 0.2, 0.3, 0.4])

# 振幅嵌入
@qml.qnode(dev)
def amplitude_embed(features):
    AmplitudeEmbedding(features, wires=range(2), normalize=True)
    return qml.expval(qml.PauliZ(0))

features = np.array([0.5, 0.5, 0.5, 0.5])

# IQP嵌入
@qml.qnode(dev)
def iqp_embed(features):
    IQPEmbedding(features, wires=range(4), n_repeats=2)
    return qml.expval(qml.PauliZ(0))
```

### 自定义模板

```python
def custom_layer(weights, wires):
    """定义自定义模板"""
    n_wires = len(wires)

    # 旋转层
    for i, wire in enumerate(wires):
        qml.RY(weights[i], wires=wire)

    # 纠缠模式
    for i in range(0, n_wires-1, 2):
        qml.CNOT(wires=[wires[i], wires[i+1]])

    for i in range(1, n_wires-1, 2):
        qml.CNOT(wires=[wires[i], wires[i+1]])

@qml.qnode(dev)
def circuit_custom(weights, n_layers):
    for i in range(n_layers):
        custom_layer(weights[i], wires=range(4))
    return qml.expval(qml.PauliZ(0))
```

## 变换

### 电路变换

```python
# 取消相邻逆操作
from pennylane import transforms

@transforms.cancel_inverses
@qml.qnode(dev)
def circuit():
    qml.Hadamard(wires=0)
    qml.Hadamard(wires=0)  # 这些操作会相互抵消
    qml.RX(0.5, wires=1)
    return qml.expval(qml.PauliZ(0))

# 合并旋转操作
@transforms.merge_rotations
@qml.qnode(dev)
def circuit():
    qml.RX(0.1, wires=0)
    qml.RX(0.2, wires=0)  # 合并为单个 RX(0.3)
    return qml.expval(qml.PauliZ(0))

# 将测量操作移至末尾
@transforms.commute_controlled
@qml.qnode(dev)
def circuit():
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])
    return qml.expval(qml.PauliZ(0))
```

### 参数广播

```python
# 使用多组参数执行电路
@qml.qnode(dev)
def circuit(x):
    qml.RX(x, wires=0)
    return qml.expval(qml.PauliZ(0))

# 参数广播
params = np.array([0.1, 0.2, 0.3, 0.4])
results = circuit(params)  # 返回结果数组
```

### 度量张量

```python
# 计算量子几何张量
@qml.qnode(dev)
def variational_circuit(params):
    for i, param in enumerate(params):
        qml.RY(param, wires=i % 4)
    for i in range(3):
        qml.CNOT(wires=[i, i+1])
    return qml.expval(qml.PauliZ(0))

params = np.array([0.1, 0.2, 0.3, 0.4], requires_grad=True)

# 获取度量张量（用于量子自然梯度）
metric_tensor = qml.metric_tensor(variational_circuit)(params)
```

### 磁带操作

```python
with qml.tape.QuantumTape() as tape:
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])
    qml.RX(0.5, wires=1)
    qml.expval(qml.PauliZ(0))

# 检查磁带
print("操作序列:", tape.operations)
print("可观测量:", tape.observables)

# 变换磁带
expanded_tape = transforms.expand_tape(tape)
optimized_tape = transforms.cancel_inverses(tape)
```

### 分解操作

```python
# 将操作分解为原生门集合
@qml.qnode(dev)
def circuit():
    qml.U3(0.1, 0.2, 0.3, wires=0)  # 任意单量子比特门
    return qml.expval(qml.PauliZ(0))

# 将 U3 分解为 RZ, RY
decomposed = qml.transforms.decompose(circuit, gate_set={qml.RZ, qml.RY, qml.CNOT})
```

## 脉冲编程

### 脉冲级控制

```python
from pennylane import pulse

# 定义脉冲包络
def gaussian_pulse(t, amplitude, sigma):
    return amplitude * np.exp(-(t**2) / (2 * sigma**2))

# 创建脉冲程序
dev_pulse = qml.device('default.qubit', wires=2)

@qml.qnode(dev_pulse)
def pulse_circuit():
    # 向量子比特施加脉冲
    pulse.drive(
        amplitude=lambda t: gaussian_pulse(t, 1.0, 0.5),
        phase=0.0,
        freq=5.0,
        wires=0,
        duration=2.0
    )

    return qml.expval(qml.PauliZ(0))
```

### 脉冲序列

```python
@qml.qnode(dev_pulse)
def pulse_sequence():
    # 脉冲序列
    duration = 1.0

    # X 脉冲
    pulse.drive(
        amplitude=lambda t: np.sin(np.pi * t / duration),
        phase=0.0,
        freq=5.0,
        wires=0,
        duration=duration
    )

    # Y 脉冲
    pulse.drive(
        amplitude=lambda t: np.sin(np.pi * t / duration),
        phase=np.pi/2,
        freq=5.0,
        wires=0,
        duration=duration
    )

    return qml.expval(qml.PauliZ(0))
```

### 最优控制

```python
def optimize_pulse(target_gate):
    """优化脉冲以实现目标门"""

    def pulse_fn(t, params):
        # 参数化脉冲
        return params[0] * np.sin(params[1] * t + params[2])

    @qml.qnode(dev_pulse)
    def pulse_circuit(params):
        pulse.drive(
            amplitude=lambda t: pulse_fn(t, params),
            phase=0.0,
            freq=5.0,
            wires=0,
            duration=2.0
        )
        return qml.expval(qml.PauliZ(0))

    # 代价函数：与目标的保真度
    def cost(params):
        result_state = pulse_circuit(params)
        target_state = target_gate()
        return 1 - np.abs(np.vdot(result_state, target_state))**2

    # 优化过程
    opt = qml.AdamOptimizer(stepsize=0.01)
    params = np.random.random(3, requires_grad=True)

    for i in range(100):
        params = opt.step(cost, params)

    return params
```

## Catalyst 与即时编译

### 基础即时编译

```python
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

### 编译控制流

```python
@qjit
@qml.qnode(dev)
def circuit_with_loops(n):
    qml.Hadamard(wires=0)

    # 编译 for 循环
    @qml.for_loop(0, n, 1)
    def loop_body(i):
        qml.RX(0.1 * i, wires=0)

    loop_body()

    return qml.expval(qml.PauliZ(0))

result = circuit_with_loops(10)
```

### 编译 while 循环

```python
@qjit
@qml.qnode(dev)
def circuit_while():
    qml.Hadamard(wires=0)

    # 编译 while 循环
    @qml.while_loop(lambda i: i < 10)
    def loop_body(i):
        qml.RX(0.1, wires=0)
        return i + 1

    loop_body(0)

    return qml.expval(qml.PauliZ(0))
```

### 即时编译的自动微分

```python
@qjit
@qml.qnode(dev)
def circuit(params):
    qml.RX(params[0], wires=0)
    qml.RY(params[1], wires=1)
    return qml.expval(qml.PauliZ(0))

# 编译梯度计算
grad_fn = qjit(qml.grad(circuit))

params = np.array([0.1, 0.2])
gradients = grad_fn(params)
```

## 自适应电路

### 带反馈的中途测量

```python
dev = qml.device('default.qubit', wires=3)

@qml.qnode(dev)
def adaptive_circuit():
    # 准备状态
    qml.Hadamard(wires=0)
    qml.CNOT(wires=[0, 1])

    # 中途测量
    m0 = qml.measure(0)

    # 基于测量的条件操作
    qml.cond(m0, qml.PauliX)(wires=2)

    # 再次测量
    m1 = qml.measure(1)

    # 更复杂的条件操作
    qml.cond(m0 & m1, qml.Hadamard)(wires=2)

    return qml.expval(qml.PauliZ(2))
```

### 动态电路深度

```python
@qml.qnode(dev)
def dynamic_depth_circuit(max_depth):
    qml.Hadamard(wires=0)

    converged = False
    depth = 0

    while not converged and depth < max_depth:
        # 应用层
        qml.RX(0.1 * depth, wires=0)

        # 通过测量检查收敛性
        m = qml.measure(0, reset=True)

        if m == 1:
            converged = True

        depth += 1

    return qml.expval(qml.PauliZ(0))
```

### 量子纠错

```python
def bit_flip_code():
    """3量子比特比特翻转纠错码"""

    @qml.qnode(dev)
    def circuit():
        # 编码逻辑量子比特
        qml.CNOT(wires=[0, 1])
        qml.CNOT(wires=[0, 2])

        # 模拟错误
        qml.PauliX(wires=1)  # 量子比特1上的比特翻转

        # 症状测量
        qml.CNOT(wires=[0, 3])
        qml.CNOT(wires=[1, 3])
        s1 = qml.measure(3)

        qml.CNOT(wires=[1, 4])
        qml.CNOT(wires=[2, 4])
        s2 = qml.measure(4)

        # 纠错操作
        qml.cond(s1 & ~s2, qml.PauliX)(wires=0)
        qml.cond(s1 & s2, qml.PauliX)(wires=1)
        qml.cond(~s1 & s2, qml.PauliX)(wires=2)

        return qml.expval(qml.PauliZ(0))

    return circuit()
```

## 噪声模型

### 内置噪声通道

```python
dev_noisy = qml.device('default.mixed', wires=2)

@qml.qnode(dev_noisy)
def noisy_circuit():
    qml.Hadamard(wires=0)

    # 退极化噪声
    qml.DepolarizingChannel(0.1, wires=0)

    qml.CNOT(wires=[0, 1])

    # 振幅阻尼（能量损失）
    qml.AmplitudeDamping(0.05, wires=0)

    # 相位阻尼（退相干）
    qml.PhaseDamping(0.05, wires=1)

    # 比特翻转错误
    qml.BitFlip(0.01, wires=0)

    # 相位翻转错误
    qml.PhaseFlip(0.01, wires=1)

    return qml.expval(qml.PauliZ(0))
```

### 自定义噪声模型

```python
def custom_noise(p):
    """自定义噪声通道"""
    #

```markdown
qml.QubitChannel(custom_noise(0.1), wires=0)

    return qml.expval(qml.PauliZ(0))
```

### 噪声感知训练

```python
def train_with_noise(circuit, params, noise_level):
    """在考虑硬件噪声的情况下进行训练"""

    dev_ideal = qml.device('default.qubit', wires=4)
    dev_noisy = qml.device('default.mixed', wires=4)

    @qml.qnode(dev_noisy)
    def noisy_circuit(p):
        circuit(p)

        # 在每个量子门后添加噪声
        for wire in range(4):
            qml.DepolarizingChannel(noise_level, wires=wire)

        return qml.expval(qml.PauliZ(0))

    # 优化含噪电路
    opt = qml.AdamOptimizer(stepsize=0.01)

    for i in range(100):
        params = opt.step(noisy_circuit, params)

    return params
```

## 资源估算

### 操作计数

```python
@qml.qnode(dev)
def circuit(params):
    for i, param in enumerate(params):
        qml.RY(param, wires=i % 4)
    for i in range(3):
        qml.CNOT(wires=[i, i+1])
    return qml.expval(qml.PauliZ(0))

params = np.random.random(10)

# 获取资源信息
specs = qml.specs(circuit)(params)

print(f"总门数: {specs['num_operations']}")
print(f"电路深度: {specs['depth']}")
print(f"门类型: {specs['gate_types']}")
print(f"门尺寸: {specs['gate_sizes']}")
print(f"可训练参数: {specs['num_trainable_params']}")
```

### 估算执行时间

```python
import time

def estimate_runtime(circuit, params, n_runs=10):
    """估算电路执行时间"""

    times = []
    for _ in range(n_runs):
        start = time.time()
        result = circuit(params)
        times.append(time.time() - start)

    mean_time = np.mean(times)
    std_time = np.std(times)

    print(f"平均执行时间: {mean_time*1000:.2f} 毫秒")
    print(f"标准差: {std_time*1000:.2f} 毫秒")

    return mean_time
```

### 资源需求估算

```python
def estimate_resources(n_qubits, depth):
    """估算计算资源需求"""

    # 经典模拟成本
    state_vector_size = 2**n_qubits * 16  # 字节 (complex128)

    # 操作数量
    n_operations = depth * n_qubits

    print(f"量子比特数: {n_qubits}")
    print(f"电路深度: {depth}")
    print(f"态向量大小: {state_vector_size / 1e9:.2f} GB")
    print(f"操作总数: {n_operations}")

    # 近似模拟时间（粗略估计）
    gate_time = 1e-6  # 每门操作时间（秒，因设备而异）
    total_time = n_operations * gate_time * 2**n_qubits

    print(f"预估模拟时间: {total_time:.4f} 秒")

    return {
        'memory': state_vector_size,
        'operations': n_operations,
        'time': total_time
    }

estimate_resources(n_qubits=20, depth=100)
```

## 最佳实践

1. **使用模板** - 利用内置模板实现通用模式
2. **应用变换** - 执行前使用变换优化电路
3. **JIT编译** - 性能关键代码使用Catalyst编译
4. **考虑噪声** - 包含噪声模型以模拟真实硬件
5. **估算资源** - 在硬件运行前分析电路资源
6. **使用自适应电路** - 通过中途测量实现灵活性
7. **优化脉冲** - 微调控制脉冲参数
8. **缓存编译结果** - 复用已编译电路
9. **监控性能** - 跟踪执行时间和资源使用
10. **全面测试** - 硬件部署前在模拟器验证
```
