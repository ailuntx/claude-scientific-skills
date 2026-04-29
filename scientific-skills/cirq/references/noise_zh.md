# 噪声建模与缓解

本指南涵盖 Cirq 中的噪声模型、含噪模拟、表征和误差缓解技术。

## 噪声通道

### 退极化噪声

```python
import cirq
import numpy as np

# 单量子比特退极化通道
depol_channel = cirq.depolarize(p=0.01)

# 应用于量子比特
q = cirq.LineQubit(0)
noisy_op = depol_channel(q)

# 添加到电路
circuit = cirq.Circuit(
    cirq.H(q),
    depol_channel(q),
    cirq.measure(q, key='m')
)
```

### 振幅阻尼

```python
# 振幅阻尼 (T1衰减)
gamma = 0.1
amp_damp = cirq.amplitude_damp(gamma)

# 在门操作后应用
circuit = cirq.Circuit(
    cirq.X(q),
    amp_damp(q)
)
```

### 相位阻尼

```python
# 相位阻尼 (T2退相干)
gamma = 0.1
phase_damp = cirq.phase_damp(gamma)

circuit = cirq.Circuit(
    cirq.H(q),
    phase_damp(q)
)
```

### 比特翻转噪声

```python
# 比特翻转通道
bit_flip_prob = 0.01
bit_flip = cirq.bit_flip(bit_flip_prob)

circuit = cirq.Circuit(
    cirq.H(q),
    bit_flip(q)
)
```

### 相位翻转噪声

```python
# 相位翻转通道
phase_flip_prob = 0.01
phase_flip = cirq.phase_flip(phase_flip_prob)

circuit = cirq.Circuit(
    cirq.H(q),
    phase_flip(q)
)
```

### 广义振幅阻尼

```python
# 广义振幅阻尼
p = 0.1  # 阻尼概率
gamma = 0.2  # 激发概率
gen_amp_damp = cirq.generalized_amplitude_damp(p=p, gamma=gamma)
```

### 重置通道

```python
# 重置至 |0⟩ 或 |1⟩
reset_to_zero = cirq.reset(q)

# 重置表现为测量后条件翻转
circuit = cirq.Circuit(
    cirq.H(q),
    reset_to_zero
)
```

## 噪声模型

### 恒定噪声模型

```python
# 对所有量子比特应用相同噪声
noise = cirq.ConstantQubitNoiseModel(
    qubit_noise_gate=cirq.depolarize(0.01)
)

# 含噪模拟
simulator = cirq.DensityMatrixSimulator(noise=noise)
result = simulator.run(circuit, repetitions=1000)
```

### 门类型特定噪声

```python
class CustomNoiseModel(cirq.NoiseModel):
    """为不同门类型应用不同噪声"""

    def noisy_operation(self, op):
        # 单量子比特门：退极化噪声
        if len(op.qubits) == 1:
            return [op, cirq.depolarize(0.001)(op.qubits[0])]

        # 双量子比特门：更高强度退极化噪声
        elif len(op.qubits) == 2:
            return [
                op,
                cirq.depolarize(0.01)(op.qubits[0]),
                cirq.depolarize(0.01)(op.qubits[1])
            ]

        return op

# 使用自定义噪声模型
noise_model = CustomNoiseModel()
simulator = cirq.DensityMatrixSimulator(noise=noise_model)
```

### 量子比特特定噪声

```python
class QubitSpecificNoise(cirq.NoiseModel):
    """不同量子比特应用不同噪声"""

    def __init__(self, qubit_noise_map):
        self.qubit_noise_map = qubit_noise_map

    def noisy_operation(self, op):
        noise_ops = [op]
        for qubit in op.qubits:
            if qubit in self.qubit_noise_map:
                noise = self.qubit_noise_map[qubit]
                noise_ops.append(noise(qubit))
        return noise_ops

# 定义量子比特专属噪声
q0, q1, q2 = cirq.LineQubit.range(3)
noise_map = {
    q0: cirq.depolarize(0.001),
    q1: cirq.depolarize(0.005),
    q2: cirq.depolarize(0.002)
}

noise_model = QubitSpecificNoise(noise_map)
```

### 热噪声

```python
class ThermalNoise(cirq.NoiseModel):
    """热弛豫噪声"""

    def __init__(self, T1, T2, gate_time):
        self.T1 = T1  # 振幅阻尼时间
        self.T2 = T2  # 退相位时间
        self.gate_time = gate_time

    def noisy_operation(self, op):
        # 计算概率
        p_amp = 1 - np.exp(-self.gate_time / self.T1)
        p_phase = 1 - np.exp(-self.gate_time / self.T2)

        noise_ops = [op]
        for qubit in op.qubits:
            noise_ops.append(cirq.amplitude_damp(p_amp)(qubit))
            noise_ops.append(cirq.phase_damp(p_phase)(qubit))

        return noise_ops

# 典型超导量子比特参数
T1 = 50e-6  # 50 μs
T2 = 30e-6  # 30 μs
gate_time = 25e-9  # 25 ns

noise_model = ThermalNoise(T1, T2, gate_time)
```

## 向电路添加噪声

### with_noise 方法

```python
# 向所有操作添加噪声
noisy_circuit = circuit.with_noise(cirq.depolarize(p=0.01))

# 模拟含噪电路
simulator = cirq.DensityMatrixSimulator()
result = simulator.run(noisy_circuit, repetitions=1000)
```

### insert_into_circuit 方法

```python
# 手动噪声注入
def add_noise_to_circuit(circuit, noise_model):
    noisy_moments = []
    for moment in circuit:
        ops = []
        for op in moment:
            ops.extend(noise_model.noisy_operation(op))
        noisy_moments.append(cirq.Moment(ops))
    return cirq.Circuit(noisy_moments)
```

## 读出噪声

### 测量误差模型

```python
class ReadoutNoiseModel(cirq.NoiseModel):
    """建模读出/测量误差"""

    def __init__(self, p0_given_1, p1_given_0):
        # p0_given_1: 状态为1时测得0的概率
        # p1_given_0: 状态为0时测得1的概率
        self.p0_given_1 = p0_given_1
        self.p1_given_0 = p1_given_0

    def noisy_operation(self, op):
        if isinstance(op.gate, cirq.MeasurementGate):
            # 测量前应用比特翻转
            noise_ops = []
            for qubit in op.qubits:
                # 平均读出误差
                p_error = (self.p0_given_1 + self.p1_given_0) / 2
                noise_ops.append(cirq.bit_flip(p_error)(qubit))
            noise_ops.append(op)
            return noise_ops
        return op

# 典型读出误差
readout_noise = ReadoutNoiseModel(p0_given_1=0.02, p1_given_0=0.01)
```

## 噪声表征

### 随机基准测试

```python
import cirq

def generate_rb_circuit(qubits, depth):
    """生成随机基准测试电路"""
    # 随机Clifford门
    clifford_gates = [cirq.X, cirq.Y, cirq.Z, cirq.H, cirq.S]

    circuit = cirq.Circuit()
    for _ in range(depth):
        for qubit in qubits:
            gate = np.random.choice(clifford_gates)
            circuit.append(gate(qubit))

    # 添加逆操作返回初始态（理想情况）
    # （简化版 - 完整RB需跟踪整个序列）

    circuit.append(cirq.measure(*qubits, key='result'))
    return circuit

# 运行RB实验
def run_rb_experiment(qubits, depths, repetitions=1000):
    """在不同深度运行随机基准测试"""
    simulator = cirq.DensityMatrixSimulator(
        noise=cirq.ConstantQubitNoiseModel(cirq.depolarize(0.01))
    )

    survival_probs = []
    for depth in depths:
        circuits = [generate_rb_circuit(qubits, depth) for _ in range(20)]

        total_survival = 0
        for circuit in circuits:
            result = simulator.run(circuit, repetitions=repetitions)
            # 计算存活概率（返回|0⟩态）
            counts = result.histogram(key='result')
            survival = counts.get(0, 0) / repetitions
            total_survival += survival

        avg_survival = total_survival / len(circuits)
        survival_probs.append(avg_survival)

    return survival_probs

# 拟合提取错误率
# p_survival = A * p^depth + B
# 每门错误率 ≈ (1 - p) / 2
```

### 交叉熵基准测试 (XEB)

```python
def xeb_fidelity(circuit, simulator, ideal_probs, repetitions=10000):
    """计算XEB保真度"""

    # 运行含噪模拟
    result = simulator.run(circuit, repetitions=repetitions)
    measured_probs = result.histogram(key='result')

    # 归一化
    for key in measured_probs:
        measured_probs[key] /= repetitions

    # 计算交叉熵
    cross_entropy = 0
    for bitstring, prob in measured_probs.items():
        if bitstring in ideal_probs:
            cross_entropy += prob * np.log2(ideal_probs[bitstring])

    # 转换为保真度
    n_qubits = len(circuit.all_qubits())
    fidelity = (2**n_qubits * cross_entropy + 1) / (2**n_qubits - 1)

    return fidelity
```

## 噪声可视化

### 热力图可视化

```python
import matplotlib.pyplot as plt

def plot_noise_heatmap(device, noise_metric):
    """在二维网格设备上绘制噪声特性"""

    # 获取设备量子比特（假设为GridQubit）
    qubits = sorted(device.metadata.qubit_set)
    rows = max(q.row for q in qubits) + 1
    cols = max(q.col for q in qubits) + 1

    # 创建热力图数据
    heatmap = np.full((rows, cols), np.nan)

    for qubit in qubits:
        if isinstance(qubit, cirq.GridQubit):
            value = noise_metric.get(qubit, 0)
            heatmap[qubit.row, qubit.col] = value

    # 绘图
    plt.figure(figsize=(10, 8))
    plt.imshow(heatmap, cmap='RdYlGn_r', interpolation='nearest')
    plt.colorbar(label='错误率')
    plt.title('量子比特错误率分布')
    plt.xlabel('列')
    plt.ylabel('行')
    plt.show()

# 使用示例
noise_metric = {q: np.random.random() * 0.01 for q in device.metadata.qubit_set}
plot_noise_heatmap(device, noise_metric)
```

### 门保真度可视化

```python
def plot_gate_fidelities(calibration_data):
    """绘制单/双量子比特门保真度"""

    sq_fidelities = []
    tq_fidelities = []

    for qubit, metrics in calibration_data.items():
        if 'single_qubit_rb_fidelity' in metrics:
            sq_fidelities.append(metrics['single_qubit_rb_fidelity'])
        if 'two_qubit_rb_fidelity' in metrics:
            tq_fidelities.append(metrics['two_qubit_rb_fidelity'])

    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))

    ax1.hist(sq_fidelities, bins=20)
    ax1.set_xlabel('单量子比特门保真度')
    ax1.set_ylabel('计数')
    ax1.set_title('单量子比特门保真度分布')

    ax2.hist(tq_fidelities, bins=20)
    ax2.set_xlabel('双量子比特门保真度')
    ax2.set_ylabel('计数')
    ax2.set_title('双量子比特门保真度分布')

    plt.tight_layout()
    plt.show()
```

## 误差缓解技术

### 零噪声外推

```python
def zero_noise_extrapolation(circuit, noise_levels, simulator):
    """外推至零噪声极限"""

    expectation_values = []

    for noise_level in noise_levels:
        # 缩放噪声
        noisy_circuit = circuit.with_noise(
            cirq.depolarize(p=noise_level)
        )

        # 测量期望值
        result = simulator.simulate(noisy_circuit)
        # ... 计算期望值
        exp_val = calculate_expectation(result)
        expectation_values.append(exp_val)

    # 外推至零噪声
    from scipy.optimize import curve_fit

    def exponential_fit(x, a, b, c):
        return a * np.exp(-b * x) + c

    popt, _ = curve_fit(exponential_fit, noise_levels, expectation_values)
    zero_noise_value = popt[2]

    return zero_noise_value
```

### 概率误差消除

```python
def quasi_probability_decomposition(noisy_gate, ideal_gate, noise_model):
    """将含噪门分解为准概率分布"""

    # 分解含噪门：N = ideal + error
    # 反演：ideal = (N - error) / (1 - error_rate)

    # 这将创建准概率分布
    # （部分概率可能为负值）

    # 具体实现取决于噪声模型
    pass
```

### 读出误差缓解

```python
def mitigate_readout_errors(results, confusion_matrix):
    """使用混淆矩阵缓解读出误差"""

    # 求逆混淆矩阵
    inv_confusion = np.linalg.inv(confusion_matrix)

    # 获取测量计数
    counts = results.histogram(key='result')

    # 转换为概率向量
    total_counts = sum(counts.values())
    measured_probs = np.array([counts.get(i, 0) / total_counts
                               for i in range(len(confusion_matrix))])

    # 应用逆矩阵
    corrected_probs = inv_confusion @ measured_probs

    # 转换回计数
    corrected_counts = {i: int(p * total_counts)
                       for i, p in enumerate(corrected_probs) if p > 0}

    return corrected_counts
```

## 基于硬件的噪声模型

### 来自谷歌校准数据

```python
import cirq_google

# 获取校准数据
processor = cirq_google.get_engine().get_processor('weber')
noise_props = processor.get_device_specification()

# 从校准创建噪声模型
noise_model = cirq_google.NoiseModelFromGoogleNoiseProperties(noise_props)

# 使用真实噪声模拟
simulator = cirq.DensityMatrixSimulator(noise=noise_model)
result = simulator.run(circuit, repetitions=1000)
```

## 最佳实践

1. **使用密度矩阵模拟器进行含噪模拟**：状态向量模拟器无法建模混合态
2. **匹配硬件噪声模型**：优先使用校准数据
3. **包含所有误差源**：门误差、退相干、读出误差
4. **先表征后缓解**：实施缓解前理解噪声特性
5. **考虑误差传播**：噪声随电路深度累积
6. **选用合适基准测试**：RB用于门误差，XEB用于全电路保真度
7. **可视化噪声模式**：识别问题量子比特和门操作
8. **针对性应用缓解**：聚焦主要误差源
9. **验证缓解效果**：确认缓解措施改善结果
10. **保持电路浅层**：最小化噪声累积
