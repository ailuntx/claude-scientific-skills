---
name: cirq
description: Google量子计算框架。适用于面向Google Quantum AI硬件、设计噪声感知电路或运行量子表征实验的场景。最适合Google硬件、噪声建模和底层电路设计。针对IBM硬件请使用qiskit；支持自动微分的量子机器学习请使用pennylane；物理模拟请使用qutip。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# Cirq - 使用Python进行量子计算

Cirq是Google Quantum AI的开源框架，用于在量子计算机和模拟器上设计、模拟和运行量子电路。

## 安装

```bash
uv pip install cirq
```

硬件集成支持：
```bash
# Google量子引擎
uv pip install cirq-google

# IonQ
uv pip install cirq-ionq

# AQT（阿尔卑斯量子技术）
uv pip install cirq-aqt

# Pasqal
uv pip install cirq-pasqal

# Azure Quantum
uv pip install azure-quantum cirq
```

## 快速入门

### 基础电路

```python
import cirq
import numpy as np

# 创建量子比特
q0, q1 = cirq.LineQubit.range(2)

# 构建电路
circuit = cirq.Circuit(
    cirq.H(q0),              # q0上的Hadamard门
    cirq.CNOT(q0, q1),       # CNOT门（q0控制位，q1目标位）
    cirq.measure(q0, q1, key='result')
)

print(circuit)

# 模拟运行
simulator = cirq.Simulator()
result = simulator.run(circuit, repetitions=1000)

# 显示结果
print(result.histogram(key='result'))
```

### 参数化电路

```python
import sympy

# 定义符号参数
theta = sympy.Symbol('theta')

# 创建参数化电路
circuit = cirq.Circuit(
    cirq.ry(theta)(q0),
    cirq.measure(q0, key='m')
)

# 参数值扫描
sweep = cirq.Linspace('theta', start=0, stop=2*np.pi, length=20)
results = simulator.run_sweep(circuit, params=sweep, repetitions=1000)

# 处理结果
for params, result in zip(sweep, results):
    theta_val = params['theta']
    counts = result.histogram(key='m')
    print(f"θ={theta_val:.2f}: {counts}")
```

## 核心功能

### 电路构建
关于构建量子电路的完整指南（包括量子比特、量子门、操作、自定义门和电路模式），请参阅：
- **[references/building.md](references/building.md)** - 电路构建完整指南

常见主题：
- 量子比特类型（GridQubit, LineQubit, NamedQubit）
- 单量子比特门和双量子比特门
- 参数化门与操作
- 自定义门分解
- 基于时刻（moments）的电路组织
- 标准电路模式（贝尔态、GHZ态、量子傅里叶变换）
- 导入/导出（OpenQASM, JSON）
- 量子比特与可观测量操作

### 模拟仿真
关于量子电路模拟的详细信息（包括精确模拟、噪声模拟、参数扫描和量子虚拟机），请参阅：
- **[references/simulation.md](references/simulation.md)** - 量子模拟完整指南

常见主题：
- 精确模拟（态矢量、密度矩阵）
- 采样与测量
- 参数扫描（单参数与多参数）
- 噪声模拟
- 态直方图与可视化
- 量子虚拟机（QVM）
- 期望值与可观测量
- 性能优化

### 电路转换
关于优化、编译和操作量子电路的信息，请参阅：
- **[references/transformation.md](references/transformation.md)** - 电路转换完整指南

常见主题：
- 转换器框架
- 量子门分解
- 电路优化（合并门、弹出Z门、丢弃可忽略操作）
- 硬件专用电路编译
- 量子比特路由与SWAP门插入
- 自定义转换器
- 转换流水线

### 硬件集成
关于在不同供应商的真实量子硬件上运行电路的信息，请参阅：
- **[references/hardware.md](references/hardware.md)** - 硬件集成完整指南

支持供应商：
- **Google Quantum AI** (cirq-google) - Sycamore, Weber处理器
- **IonQ** (cirq-ionq) - 离子阱量子计算机
- **Azure Quantum** (azure-quantum) - IonQ和Honeywell后端
- **AQT** (cirq-aqt) - 阿尔卑斯量子技术
- **Pasqal** (cirq-pasqal) - 中性原子量子计算机

主题包括设备表示、量子比特选择、认证、任务管理和硬件专用电路优化。

### 噪声建模
关于噪声建模、噪声模拟、表征和误差缓解的信息，请参阅：
- **[references/noise.md](references/noise.md)** - 噪声建模完整指南

常见主题：
- 噪声通道（退极化、振幅阻尼、相位阻尼）
- 噪声模型（恒定噪声、门相关噪声、量子比特相关噪声、热噪声）
- 向电路添加噪声
- 读出噪声
- 噪声表征（随机基准测试、XEB）
- 噪声可视化（热力图）
- 误差缓解技术

### 量子实验
关于实验设计、参数扫描、数据收集和ReCirq框架使用的信息，请参阅：
- **[references/experiments.md](references/experiments.md)** - 量子实验完整指南

常见主题：
- 实验设计模式
- 参数扫描与数据收集
- ReCirq框架结构
- 常用算法（VQE, QAOA, QPE）
- 数据分析与可视化
- 统计分析与保真度估计
- 并行数据收集

## 常用模式

### 变分算法模板

```python
import scipy.optimize

def variational_algorithm(ansatz, cost_function, initial_params):
    """变分量子算法模板"""

    def objective(params):
        circuit = ansatz(params)
        simulator = cirq.Simulator()
        result = simulator.simulate(circuit)
        return cost_function(result)

    # 优化过程
    result = scipy.optimize.minimize(
        objective,
        initial_params,
        method='COBYLA'
    )

    return result

# 定义ansatz电路
def my_ansatz(params):
    q = cirq.LineQubit(0)
    return cirq.Circuit(
        cirq.ry(params[0])(q),
        cirq.rz(params[1])(q)
    )

# 定义损失函数
def my_cost(result):
    state = result.final_state_vector
    # 基于量子态计算损失值
    return np.real(state[0])

# 执行优化
result = variational_algorithm(my_ansatz, my_cost, [0.0, 0.0])
```

### 硬件执行模板

```python
def run_on_hardware(circuit, provider='google', device_name='weber', repetitions=1000):
    """量子硬件执行模板"""

    if provider == 'google':
        import cirq_google
        engine = cirq_google.get_engine()
        processor = engine.get_processor(device_name)
        job = processor.run(circuit, repetitions=repetitions)
        return job.results()[0]

    elif provider == 'ionq':
        import cirq_ionq
        service = cirq_ionq.Service()
        result = service.run(circuit, repetitions=repetitions, target='qpu')
        return result

    elif provider == 'azure':
        from azure.quantum.cirq import AzureQuantumService
        # 工作区配置...
        service = AzureQuantumService(workspace)
        result = service.run(circuit, repetitions=repetitions, target='ionq.qpu')
        return result

    else:
        raise ValueError(f"未知供应商: {provider}")
```

### 噪声研究模板

```python
def noise_comparison_study(circuit, noise_levels):
    """比较不同噪声水平下的电路性能"""

    results = {}

    for noise_level in noise_levels:
        # 创建含噪声电路
        noisy_circuit = circuit.with_noise(cirq.depolarize(p=noise_level))

        # 模拟运行
        simulator = cirq.DensityMatrixSimulator()
        result = simulator.run(noisy_circuit, repetitions=1000)

        # 结果分析
        results[noise_level] = {
            'histogram': result.histogram(key='result'),
            'dominant_state': max(
                result.histogram(key='result').items(),
                key=lambda x: x[1]
            )
        }

    return results

# 执行研究
noise_levels = [0.0, 0.001, 0.01, 0.05, 0.1]
results = noise_comparison_study(circuit, noise_levels)
```

## 最佳实践

1. **电路设计**
   - 根据拓扑结构选用合适的量子比特类型
   - 保持电路模块化和可复用性
   - 使用描述性键名标记测量操作
   - 执行前根据设备约束验证电路

2. **模拟仿真**
   - 纯态场景使用态矢量模拟（效率更高）
   - 仅在混合态/噪声场景使用密度矩阵模拟
   - 优先采用参数扫描而非单次运行
   - 监控大系统内存使用（2^n增长迅速）

3. **硬件执行**
   - 始终先在模拟器测试
   - 根据校准数据选择最优量子比特
   - 针对目标硬件门集优化电路
   - 生产环境实施误差缓解
   - 立即存储昂贵的硬件运行结果

4. **电路优化**
   - 从高级内置转换器开始
   - 按顺序链接多个优化步骤
   - 跟踪深度和量子门数量缩减
   - 转换后验证正确性

5. **噪声建模**
   - 采用校准数据的真实噪声模型
   - 包含所有误差源（门误差、退相干、读出误差）
   - 先表征后缓解
   - 保持电路浅层以最小化噪声累积

6. **实验设计**
   - 明确分离实验阶段（数据生成、收集、分析）
   - 使用ReCirq模式确保可复现性
   - 频繁保存中间结果
   - 并行化独立任务
   - 使用元数据完整记录

## 附加资源

- **官方文档**: https://quantumai.google/cirq
- **API参考**: https://quantumai.google/reference/python/cirq
- **教程**: https://quantumai.google/cirq/tutorials
- **示例**: https://github.com/quantumlib/Cirq/tree/master/examples
- **ReCirq**: https://github.com/quantumlib/ReCirq

## 常见问题

**电路深度超出硬件限制：**
- 使用电路优化转换器降低深度
- 参见`transformation.md`获取优化技术

**模拟内存不足：**
- 从密度矩阵模拟切换至态矢量模拟
- 减少量子比特数量或对Clifford电路使用稳定子模拟器

**设备验证错误：**
- 通过device.metadata.nx_graph检查量子比特连通性
- 将量子门分解为设备原生门集
- 参见`hardware.md`获取设备专用编译指南

**噪声模拟速度过慢：**
- 密度矩阵模拟复杂度O(2^2n) - 考虑减少量子比特
- 仅在关键操作上选择性应用噪声模型
- 参见`simulation.md`获取性能优化方案
