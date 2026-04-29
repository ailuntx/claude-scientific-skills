# Qiskit 可视化工具

Qiskit 提供全面的可视化工具，用于展示量子电路、测量结果和量子态。

## 安装

安装可视化依赖项：

```bash
uv pip install "qiskit[visualization]" matplotlib
```

## 电路可视化

### 文本绘图

```python
from qiskit import QuantumCircuit

qc = QuantumCircuit(3)
qc.h(0)
qc.cx(0, 1)
qc.cx(1, 2)

# 简单文本输出
print(qc.draw())

# 带更多细节的文本
print(qc.draw('text', fold=-1))  # 禁止折叠长电路
```

### Matplotlib 绘图

```python
# 高质量 matplotlib 图形
qc.draw('mpl')

# 保存到文件
fig = qc.draw('mpl')
fig.savefig('circuit.png', dpi=300, bbox_inches='tight')
```

### LaTeX 绘图

```python
# 生成 LaTeX 电路图
qc.draw('latex')

# 保存 LaTeX 源码
latex_source = qc.draw('latex_source')
with open('circuit.tex', 'w') as f:
    f.write(latex_source)
```

## 自定义电路绘图

### 样式选项

```python
from qiskit.visualization import circuit_drawer

# 反转量子比特顺序
qc.draw('mpl', reverse_bits=True)

# 折叠长电路
qc.draw('mpl', fold=20)  # 每20列折叠

# 显示空闲线路
qc.draw('mpl', idle_wires=False)

# 添加初始态
qc.draw('mpl', initial_state=True)
```

### 颜色自定义

```python
style = {
    'displaycolor': {
        'h': ('#FA74A6', '#000000'),     # Hadamard: 粉色
        'cx': ('#A8D0DB', '#000000'),    # CNOT: 浅蓝
        'measure': ('#F7E7B4', '#000000') # 测量: 黄色
    }
}

qc.draw('mpl', style=style)
```

## 结果可视化

### 计数直方图

```python
from qiskit.visualization import plot_histogram
from qiskit.primitives import StatevectorSampler

qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.measure_all()

sampler = StatevectorSampler()
result = sampler.run([qc], shots=1024).result()
counts = result[0].data.meas.get_counts()

# 绘制直方图
plot_histogram(counts)

# 比较多次实验
counts1 = {'00': 500, '11': 524}
counts2 = {'00': 480, '11': 544}
plot_histogram([counts1, counts2], legend=['运行1', '运行2'])

# 保存图形
fig = plot_histogram(counts)
fig.savefig('histogram.png', dpi=300, bbox_inches='tight')
```

### 直方图选项

```python
# 自定义颜色
plot_histogram(counts, color=['#1f77b4', '#ff7f0e'])

# 按值排序
plot_histogram(counts, sort='value')

# 设置柱状图标签
plot_histogram(counts, bar_labels=True)

# 设置目标分布（用于对比）
target = {'00': 0.5, '11': 0.5}
plot_histogram(counts, target=target)
```

## 量子态可视化

### 布洛赫球面

在布洛赫球面上可视化单量子比特态：

```python
from qiskit.visualization import plot_bloch_vector
from qiskit.quantum_info import Statevector
import numpy as np

# 可视化特定态矢量
# |+⟩态: |0⟩和|1⟩的等幅叠加
state = Statevector.from_label('+')
plot_bloch_vector(state.to_bloch())

# 自定义矢量
plot_bloch_vector([0, 1, 0])  # X轴上的|+⟩态
```

### 多量子比特布洛赫球面

```python
from qiskit.visualization import plot_bloch_multivector

qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)

state = Statevector.from_instruction(qc)
plot_bloch_multivector(state)
```

### 态城市图

将态振幅可视化为3D城市：

```python
from qiskit.visualization import plot_state_city
from qiskit.quantum_info import Statevector

qc = QuantumCircuit(3)
qc.h(range(3))
state = Statevector.from_instruction(qc)

plot_state_city(state)

# 自定义
plot_state_city(state, color=['#FF6B6B', '#4ECDC4'])
```

### Q球面

在球面上可视化量子态：

```python
from qiskit.visualization import plot_state_qsphere

state = Statevector.from_instruction(qc)
plot_state_qsphere(state)
```

### 欣顿图

展示态振幅：

```python
from qiskit.visualization import plot_state_hinton

state = Statevector.from_instruction(qc)
plot_state_hinton(state)
```

## 密度矩阵可视化

```python
from qiskit.visualization import plot_state_density
from qiskit.quantum_info import DensityMatrix

qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)

state = DensityMatrix.from_instruction(qc)
plot_state_density(state)
```

## 门映射可视化

可视化后端耦合映射：

```python
from qiskit.visualization import plot_gate_map
from qiskit_ibm_runtime import QiskitRuntimeService

service = QiskitRuntimeService()
backend = service.backend("ibm_brisbane")

# 显示量子比特连接性
plot_gate_map(backend)

# 显示含错误率
plot_gate_map(backend, plot_error_rates=True)
```

## 错误映射可视化

展示后端错误率：

```python
from qiskit.visualization import plot_error_map

plot_error_map(backend)
```

## 电路布局展示

```python
from qiskit.visualization import plot_circuit_layout

# 显示电路到物理量子比特的映射
transpiled_qc = transpile(qc, backend=backend)
plot_circuit_layout(transpiled_qc, backend)
```

## 脉冲可视化

用于脉冲级控制：

```python
from qiskit import pulse
from qiskit.visualization import pulse_drawer

# 创建脉冲调度
with pulse.build(backend) as schedule:
    pulse.play(pulse.Gaussian(duration=160, amp=0.1, sigma=40), pulse.drive_channel(0))

# 可视化
schedule.draw()
```

## 交互式组件 (Jupyter)

### 电路编辑器组件

```python
from qiskit.tools.jupyter import QuantumCircuitComposer

composer = QuantumCircuitComposer()
composer.show()
```

### 交互式态可视化

```python
from qiskit.visualization import plot_histogram
import matplotlib.pyplot as plt

# 启用交互模式
plt.ion()
plot_histogram(counts)
plt.show()
```

## 对比图表

### 多重直方图

```python
# 比较不同后端的结果
counts_sim = {'00': 500, '11': 524}
counts_hw = {'00': 480, '01': 20, '10': 24, '11': 500}

plot_histogram(
    [counts_sim, counts_hw],
    legend=['模拟器', '硬件'],
    figsize=(12, 6)
)
```

### 转译前后对比

```python
import matplotlib.pyplot as plt

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 4))

# 原始电路
qc.draw('mpl', ax=ax1)
ax1.set_title('原始电路')

# 转译后电路
qc_transpiled = transpile(qc, backend=backend, optimization_level=3)
qc_transpiled.draw('mpl', ax=ax2)
ax2.set_title('转译后电路')

plt.tight_layout()
plt.show()
```

## 保存可视化结果

### 保存为多种格式

```python
# PNG
fig = qc.draw('mpl')
fig.savefig('circuit.png', dpi=300, bbox_inches='tight')

# PDF
fig.savefig('circuit.pdf', bbox_inches='tight')

# SVG (矢量图)
fig.savefig('circuit.svg', bbox_inches='tight')

# 直方图
hist_fig = plot_histogram(counts)
hist_fig.savefig('results.png', dpi=300, bbox_inches='tight')
```

## 样式最佳实践

### 出版物级图形

```python
import matplotlib.pyplot as plt

# 设置 matplotlib 样式
plt.rcParams['figure.dpi'] = 300
plt.rcParams['font.size'] = 12
plt.rcParams['font.family'] = 'sans-serif'

# 创建高质量可视化
fig = qc.draw('mpl', style='iqp')
fig.savefig('publication_circuit.png', dpi=600, bbox_inches='tight')
```

### 可用样式

```python
# 默认样式
qc.draw('mpl')

# IQP 样式 (IBM Quantum)
qc.draw('mpl', style='iqp')

# 色盲友好模式
qc.draw('mpl', style='bw')  # 黑白样式
```

## 可视化故障排除

### 常见问题

**问题**："No module named 'matplotlib'"
```bash
uv pip install matplotlib
```

**问题**：电路过大无法显示
```python
# 使用折叠功能
qc.draw('mpl', fold=50)

# 或导出到文件而非直接显示
fig = qc.draw('mpl')
fig.savefig('large_circuit.png', dpi=150, bbox_inches='tight')
```

**问题**：Jupyter 笔记本不显示图表
```python
# 在笔记本开头添加魔法命令
%matplotlib inline
```

**问题**：LaTeX 可视化失效
```bash
# 安装 LaTeX 支持
uv pip install pylatexenc
```
