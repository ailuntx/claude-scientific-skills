---
name: qiskit
description: IBM量子计算框架。适用于面向IBM Quantum硬件开发、使用Qiskit Runtime处理生产工作负载或需要IBM优化工具的场景。在IBM硬件执行、量子错误缓解和企业级量子计算方面表现最佳。若使用Google硬件请选择cirq；基于梯度的量子机器学习请用pennylane；开放量子系统模拟请用qutip。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# Qiskit

## 概述

Qiskit是全球最受欢迎的开源量子计算框架，下载量超过1300万次。可构建量子电路、针对硬件优化、在模拟器或真实量子计算机上执行并分析结果。支持IBM Quantum（100+量子位系统）、IonQ、Amazon Braket等提供商。

**核心特性：**
- 比竞品快83倍的量子电路转换速度
- 优化后电路的双量子位门减少29%
- 后端无关执行（本地模拟器或云端硬件）
- 完整的优化/化学/机器学习算法库

## 快速入门

### 安装

```bash
uv pip install qiskit
uv pip install "qiskit[visualization]" matplotlib
```

### 首个量子电路

```python
from qiskit import QuantumCircuit
from qiskit.primitives import StatevectorSampler

# 创建贝尔态（纠缠量子位）
qc = QuantumCircuit(2)
qc.h(0)           # 在量子位0上应用Hadamard门
qc.cx(0, 1)       # 从量子位0到1的CNOT门
qc.measure_all()  # 测量所有量子位

# 本地运行
sampler = StatevectorSampler()
result = sampler.run([qc], shots=1024).result()
counts = result[0].data.meas.get_counts()
print(counts)  # {'00': ~512, '11': ~512}
```

### 可视化

```python
from qiskit.visualization import plot_histogram

qc.draw('mpl')           # 电路图
plot_histogram(counts)   # 结果直方图
```

## 核心功能

### 1. 环境配置
详细安装、认证及IBM Quantum账户设置：
- **参见`references/setup.md`**

涵盖内容：
- 使用uv安装
- Python环境配置
- IBM Quantum账户与API令牌设置
- 本地与云端执行模式

### 2. 构建量子电路
创建含量子门/测量/组合的电路：
- **参见`references/circuits.md`**

涵盖内容：
- 使用QuantumCircuit创建电路
- 单量子位门（H/X/Y/Z/旋转/相位门）
- 多量子位门（CNOT/SWAP/Toffoli）
- 测量与屏障指令
- 电路组合与属性
- 变分算法的参数化电路

### 3. 原语（采样器与估计器）
执行量子电路并计算结果：
- **参见`references/primitives.md`**

涵盖内容：
- **采样器**：获取比特串测量值与概率分布
- **估计器**：计算可观测量期望值
- V2接口（StatevectorSampler, StatevectorEstimator）
- 面向硬件的IBM Quantum Runtime原语
- 会话模式与批处理模式
- 参数绑定

### 4. 电路转换与优化
优化电路并准备硬件执行：
- **参见`references/transpilation.md`**

涵盖内容：
- 为何需要电路转换
- 优化级别（0-3）
- 六阶段转换流程（初始化/布局/路由/翻译/优化/调度）
- 高级功能（虚拟置换消除/门级消减）
- 常用参数（initial_layout/approximation_degree/seed）
- 高效电路最佳实践

### 5. 可视化
展示电路/结果/量子态：
- **参见`references/visualization.md`**

涵盖内容：
- 电路图绘制（文本/matplotlib/LaTeX）
- 结果直方图
- 量子态可视化（Bloch球/态城市图/QSphere）
- 后端拓扑与错误映射
- 自定义样式
- 保存出版级图表

### 6. 硬件后端
在模拟器与真实量子计算机运行：
- **参见`references/backends.md`**

涵盖内容：
- IBM Quantum后端与认证
- 后端属性与状态
- 通过Runtime原语在真实硬件运行
- 任务管理与队列
- 会话模式（迭代算法）
- 批处理模式（并行任务）
- 本地模拟器（StatevectorSampler/Aer）
- 第三方提供商（IonQ/Amazon Braket）
- 错误缓解策略

### 7. Qiskit工作流模式
实现四步量子计算工作流：
- **参见`references/patterns.md`**

涵盖内容：
- **映射**：将问题转化为量子电路
- **优化**：针对硬件转换电路
- **执行**：通过原语运行
- **后处理**：提取分析结果
- 完整VQE示例
- 会话模式 vs 批处理执行
- 常见工作流模式

### 8. 量子算法与应用
实现特定量子算法：
- **参见`references/algorithms.md`**

涵盖内容：
- **优化算法**：VQE/QAOA/Grover算法
- **化学计算**：分子基态/激发态/哈密顿量
- **机器学习**：量子核/VQC/QNN
- **算法库**：Qiskit Nature/Qiskit ML/Qiskit Optimization
- 物理模拟与基准测试

## 工作流决策指南

**若需：**

- 安装Qiskit或设置IBM Quantum账户 → `references/setup.md`
- 构建新量子电路 → `references/circuits.md`
- 理解量子门与电路操作 → `references/circuits.md`
- 运行电路获取测量结果 → `references/primitives.md`
- 计算期望值 → `references/primitives.md`
- 为硬件优化电路 → `references/transpilation.md`
- 可视化电路或结果 → `references/visualization.md`
- 在IBM Quantum硬件执行 → `references/backends.md`
- 连接第三方提供商 → `references/backends.md`
- 实现端到端量子工作流 → `references/patterns.md`
- 构建特定算法（VQE/QAOA等） → `references/algorithms.md`
- 解决化学或优化问题 → `references/algorithms.md`

## 最佳实践

### 开发工作流

1. **从模拟器开始**：使用硬件前本地测试
   ```python
   from qiskit.primitives import StatevectorSampler
   sampler = StatevectorSampler()
   ```

2. **始终转换电路**：执行前优化电路
   ```python
   from qiskit import transpile
   qc_optimized = transpile(qc, backend=backend, optimization_level=3)
   ```

3. **选用合适原语**：
   - 采样器：获取比特串（优化算法）
   - 估计器：计算期望值（化学/物理）

4. **选择执行模式**：
   - 会话模式：迭代算法（VQE/QAOA）
   - 批处理：独立并行任务
   - 单任务：一次性实验

### 性能优化

- 生产环境使用optimization_level=3
- 最小化双量子位门（主要错误源）
- 硬件执行前用含噪声模拟器测试
- 保存并复用转换后电路
- 监控变分算法收敛性

### 硬件执行

- 提交前检查后端状态
- 测试时使用least_busy()
- 保存任务ID供后续检索
- 应用错误缓解（resilience_level）
- 初始减少测量次数，最终运行增加

## 常用模式

### 模式1：基础电路执行

```python
from qiskit import QuantumCircuit, transpile
from qiskit.primitives import StatevectorSampler

qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.measure_all()

sampler = StatevectorSampler()
result = sampler.run([qc], shots=1024).result()
counts = result[0].data.meas.get_counts()
```

### 模式2：含电路转换的硬件执行

```python
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler
from qiskit import transpile

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

qc_optimized = transpile(qc, backend=backend, optimization_level=3)

sampler = Sampler(backend)
job = sampler.run([qc_optimized], shots=1024)
result = job.result()
```

### 模式3：变分算法（VQE）

```python
from qiskit_ibm_runtime import Session, EstimatorV2 as Estimator
from scipy.optimize import minimize

with Session(backend=backend) as session:
    estimator = Estimator(session=session)

    def cost_function(params):
        bound_qc = ansatz.assign_parameters(params)
        qc_isa = transpile(bound_qc, backend=backend)
        result = estimator.run([(qc_isa, hamiltonian)]).result()
        return result[0].data.evs

    result = minimize(cost_function, initial_params, method='COBYLA')
```

## 扩展资源

- **官方文档**: https://quantum.ibm.com/docs
- **Qiskit教程**: https://qiskit.org/learn
- **API参考**: https://docs.quantum.ibm.com/api/qiskit
- **模式指南**: https://quantum.cloud.ibm.com/docs/en/guides/intro-to-patterns
