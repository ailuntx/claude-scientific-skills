# Cirq 中的模拟

本指南涵盖量子电路模拟，包括精确模拟和含噪声模拟、参数扫描以及量子虚拟机（QVM）。

## 精确模拟

### 基础模拟

```python
import cirq
import numpy as np

# 创建电路
q0, q1 = cirq.LineQubit.range(2)
circuit = cirq.Circuit(
    cirq.H(q0),
    cirq.CNOT(q0, q1),
    cirq.measure(q0, q1, key='result')
)

# 模拟
simulator = cirq.Simulator()
result = simulator.run(circuit, repetitions=1000)

# 获取测量结果
print(result.histogram(key='result'))
```

### 态矢量模拟

```python
# 无测量模拟以获取最终态
simulator = cirq.Simulator()
result = simulator.simulate(circuit_without_measurement)

# 访问态矢量
state_vector = result.final_state_vector
print(f"态矢量: {state_vector}")

# 获取振幅
print(f"|00⟩ 振幅: {state_vector[0]}")
print(f"|11⟩ 振幅: {state_vector[3]}")
```

### 密度矩阵模拟

```python
# 使用密度矩阵模拟器处理混合态
simulator = cirq.DensityMatrixSimulator()
result = simulator.simulate(circuit)

# 访问密度矩阵
density_matrix = result.final_density_matrix
print(f"密度矩阵形状: {density_matrix.shape}")
```

### 分步模拟

```python
# 逐时刻模拟
simulator = cirq.Simulator()
for step in simulator.simulate_moment_steps(circuit):
    print(f"时刻 {step.moment} 后的状态: {step.state_vector()}")
```

## 采样与测量

### 多次运行

```python
# 多次运行电路
result = simulator.run(circuit, repetitions=10000)

# 获取测量计数
counts = result.histogram(key='result')
print(f"测量计数: {counts}")

# 获取原始测量数据
measurements = result.measurements['result']
print(f"形状: {measurements.shape}")  # (重复次数, 量子位数)
```

### 期望值

```python
# 测量可观测量期望值
from cirq import PauliString

observable = PauliString({q0: cirq.Z, q1: cirq.Z})
result = simulator.simulate_expectation_values(
    circuit,
    observables=[observable]
)
print(f"⟨ZZ⟩ = {result[0]}")
```

## 参数扫描

### 参数扫描

```python
import sympy

# 创建参数化电路
theta = sympy.Symbol('theta')
q = cirq.LineQubit(0)
circuit = cirq.Circuit(
    cirq.ry(theta)(q),
    cirq.measure(q, key='m')
)

# 定义参数扫描
sweep = cirq.Linspace(key='theta', start=0, stop=2*np.pi, length=50)

# 执行扫描
simulator = cirq.Simulator()
results = simulator.run_sweep(circuit, params=sweep, repetitions=1000)

# 处理结果
for params, result in zip(sweep, results):
    theta_val = params['theta']
    counts = result.histogram(key='m')
    print(f"θ={theta_val:.2f}: {counts}")
```

### 多参数扫描

```python
# 多参数扫描
theta = sympy.Symbol('theta')
phi = sympy.Symbol('phi')

circuit = cirq.Circuit(
    cirq.ry(theta)(q0),
    cirq.rz(phi)(q1)
)

# 乘积扫描（所有组合）
sweep = cirq.Product(
    cirq.Linspace('theta', 0, np.pi, 10),
    cirq.Linspace('phi', 0, 2*np.pi, 10)
)

results = simulator.run_sweep(circuit, params=sweep, repetitions=100)
```

### 配对参数扫描

```python
# 参数配对扫描
sweep = cirq.Zip(
    cirq.Linspace('theta', 0, np.pi, 20),
    cirq.Linspace('phi', 0, 2*np.pi, 20)
)

results = simulator.run_sweep(circuit, params=sweep, repetitions=100)
```

## 含噪声模拟

### 添加噪声通道

```python
# 创建含噪声电路
noisy_circuit = circuit.with_noise(cirq.depolarize(p=0.01))

# 模拟含噪声电路
simulator = cirq.DensityMatrixSimulator()
result = simulator.run(noisy_circuit, repetitions=1000)
```

### 自定义噪声模型

```python
# 为不同门应用不同噪声
noise_model = cirq.NoiseModel.from_noise_model_like(
    cirq.ConstantQubitNoiseModel(cirq.depolarize(0.01))
)

# 使用噪声模型模拟
result = cirq.DensityMatrixSimulator(noise=noise_model).run(
    circuit, repetitions=1000
)
```

完整噪声建模细节参见 `noise.md`。

## 状态直方图

### 结果可视化

```python
import matplotlib.pyplot as plt

# 获取直方图
result = simulator.run(circuit, repetitions=1000)
counts = result.histogram(key='result')

# 绘图
plt.bar(counts.keys(), counts.values())
plt.xlabel('状态')
plt.ylabel('计数')
plt.title('测量结果')
plt.show()
```

### 状态概率分布

```python
# 获取态矢量
result = simulator.simulate(circuit_without_measurement)
state_vector = result.final_state_vector

# 计算概率
probabilities = np.abs(state_vector) ** 2

# 绘图
plt.bar(range(len(probabilities)), probabilities)
plt.xlabel('基态索引')
plt.ylabel('概率')
plt.show()
```

## 量子虚拟机 (QVM)

QVM 模拟真实量子硬件，包含设备特定约束和噪声。

### 使用虚拟设备

```python
# 使用谷歌虚拟设备
import cirq_google

# 获取虚拟设备
device = cirq_google.Sycamore

# 在设备上创建电路
qubits = device.metadata.qubit_set
circuit = cirq.Circuit(device=device)

# 添加符合设备约束的操作
circuit.append(cirq.CZ(qubits[0], qubits[1]))

# 根据设备验证电路
device.validate_circuit(circuit)
```

### 含噪声虚拟硬件

```python
# 使用设备噪声模拟
processor = cirq_google.get_engine().get_processor('weber')
noise_props = processor.get_device_specification()

# 创建真实噪声模拟器
noisy_sim = cirq.DensityMatrixSimulator(
    noise=cirq_google.NoiseModelFromGoogleNoiseProperties(noise_props)
)

result = noisy_sim.run(circuit, repetitions=1000)
```

## 高级模拟技术

### 自定义初始态

```python
# 从自定义态开始
initial_state = np.array([1, 0, 0, 1]) / np.sqrt(2)  # |00⟩ + |11⟩

simulator = cirq.Simulator()
result = simulator.simulate(circuit, initial_state=initial_state)
```

### 部分迹

```python
# 追踪子系统
result = simulator.simulate(circuit)
full_state = result.final_state_vector

# 计算首个量子比特的约化密度矩阵
from cirq import partial_trace
reduced_dm = partial_trace(result.final_density_matrix, keep_indices=[0])
```

### 中间态访问

```python
# 获取特定时刻状态
simulator = cirq.Simulator()
for i, step in enumerate(simulator.simulate_moment_steps(circuit)):
    if i == 5:  # 第5个时刻后
        state = step.state_vector()
        print(f"第5时刻后状态: {state}")
        break
```

## 模拟性能

### 大型模拟优化

1. **纯态使用态矢量**：比密度矩阵更快
2. **避免密度矩阵**：指数级开销
3. **批量参数扫描**：比单次运行更高效
4. **合理设置重复次数**：平衡精度与计算时间

```python
# 高效：单次扫描
results = simulator.run_sweep(circuit, params=sweep, repetitions=100)

# 低效：多次独立运行
results = [simulator.run(circuit, param_resolver=p, repetitions=100)
           for p in sweep]
```

### 内存考量

```python
# 大系统需监控态矢量大小
n_qubits = 20
state_size = 2**n_qubits * 16  # 字节 (complex128)
print(f"态矢量大小: {state_size / 1e9:.2f} GB")
```

## 稳定子模拟

对于仅含 Clifford 门的电路，使用高效稳定子模拟：

```python
# Clifford 电路 (H, S, CNOT)
circuit = cirq.Circuit(
    cirq.H(q0),
    cirq.S(q1),
    cirq.CNOT(q0, q1)
)

# 使用稳定子模拟器（指数级加速）
simulator = cirq.CliffordSimulator()
result = simulator.run(circuit, repetitions=1000)
```

## 最佳实践

1. **选择合适模拟器**：纯态用 Simulator，混合态用 DensityMatrixSimulator
2. **使用参数扫描**：比单电路运行更高效
3. **验证电路**：长时模拟前检查电路有效性
4. **监控资源使用**：大规模模拟时跟踪内存
5. **使用稳定子模拟**：当电路仅含 Clifford 门时
6. **保存中间结果**：用于长参数扫描或优化运行
