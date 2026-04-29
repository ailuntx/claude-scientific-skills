# 量子电路构建

## 创建量子电路

使用 `QuantumCircuit` 类创建电路：

```python
from qiskit import QuantumCircuit

# 创建包含3个量子比特的电路
qc = QuantumCircuit(3)

# 创建包含3个量子比特和3个经典比特的电路
qc = QuantumCircuit(3, 3)
```

## 单量子比特门

### 泡利门

```python
qc.x(0)   # 在量子比特0上应用NOT/泡利-X门
qc.y(1)   # 在量子比特1上应用泡利-Y门
qc.z(2)   # 在量子比特2上应用泡利-Z门
```

### 哈达玛门

创建叠加态：

```python
qc.h(0)   # 在量子比特0上应用哈达玛门
```

### 相位门

```python
qc.s(0)   # S门 (√Z)
qc.t(0)   # T门 (√S)
qc.p(π/4, 0)   # 自定义角度的相位门
```

### 旋转门

```python
from math import pi

qc.rx(pi/2, 0)   # 绕X轴旋转
qc.ry(pi/4, 1)   # 绕Y轴旋转
qc.rz(pi/3, 2)   # 绕Z轴旋转
```

## 多量子比特门

### CNOT (受控非门)

```python
qc.cx(0, 1)   # CNOT门，控制位=0，目标位=1
```

### 受控门

```python
qc.cy(0, 1)   # 受控-Y门
qc.cz(0, 1)   # 受控-Z门
qc.ch(0, 1)   # 受控-哈达玛门
```

### SWAP门

```python
qc.swap(0, 1)   # 交换量子比特0和1
```

### 托佛利门 (CCX)

```python
qc.ccx(0, 1, 2)   # 托佛利门，控制位=0,1，目标位=2
```

## 测量操作

添加测量操作读取量子比特状态：

```python
# 测量所有量子比特
qc.measure_all()

# 将特定量子比特测量到特定经典比特
qc.measure(0, 0)   # 测量量子比特0到经典比特0
qc.measure([0, 1], [0, 1])   # 测量量子比特0,1到经典比特0,1
```

## 电路组合

### 电路合并

```python
qc1 = QuantumCircuit(2)
qc1.h(0)

qc2 = QuantumCircuit(2)
qc2.cx(0, 1)

# 组合电路
qc_combined = qc1.compose(qc2)
```

### 张量积

```python
qc1 = QuantumCircuit(1)
qc1.h(0)

qc2 = QuantumCircuit(1)
qc2.x(0)

# 从小电路构建更大电路
qc_tensor = qc1.tensor(qc2)   # 生成2量子比特电路
```

## 屏障与标签

```python
qc.barrier()   # 添加可视化屏障
qc.barrier([0, 1])   # 在特定量子比特上添加屏障

# 添加标签增强可读性
qc.barrier(label="初始化阶段")
```

## 电路属性

```python
print(qc.num_qubits)   # 量子比特数量
print(qc.num_clbits)   # 经典比特数量
print(qc.depth())      # 电路深度
print(qc.size())       # 门操作总数
print(qc.count_ops())  # 门操作计数字典
```

## 示例：贝尔态

创建两个量子比特间的纠缠态：

```python
qc = QuantumCircuit(2)
qc.h(0)           # 在量子比特0上创建叠加态
qc.cx(0, 1)       # 使量子比特0和1纠缠
qc.measure_all()  # 测量两个量子比特
```

## 示例：量子傅里叶变换 (QFT)

```python
from math import pi

def qft(n):
    qc = QuantumCircuit(n)
    for j in range(n):
        qc.h(j)
        for k in range(j+1, n):
            qc.cp(pi/2**(k-j), k, j)
    return qc

# 创建3量子比特QFT电路
qc_qft = qft(3)
```

## 参数化电路

为变分算法创建含参数的电路：

```python
from qiskit.circuit import Parameter

theta = Parameter('θ')
qc = QuantumCircuit(1)
qc.ry(theta, 0)

# 绑定参数值
qc_bound = qc.assign_parameters({theta: pi/4})
```

## 电路操作

```python
# 电路逆操作
qc_inverse = qc.inverse()

# 将门分解为基础门
qc_decomposed = qc.decompose()

# 绘制电路（返回字符串或图表）
print(qc.draw())
print(qc.draw('mpl'))   # Matplotlib图形
```
