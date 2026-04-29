# 使用 PennyLane 进行量子机器学习

## 目录
1. [混合量子-经典模型](#混合量子-经典模型)
2. [框架集成](#框架集成)
3. [量子神经网络](#量子神经网络)
4. [变分分类器](#变分分类器)
5. [训练与优化](#训练与优化)
6. [数据编码策略](#数据编码策略)
7. [迁移学习](#迁移学习)

## 混合量子-经典模型

### 基础混合模型

```python
import pennylane as qml
import numpy as np

dev = qml.device('default.qubit', wires=4)

@qml.qnode(dev)
def quantum_layer(inputs, weights):
    # 编码经典数据
    for i, inp in enumerate(inputs):
        qml.RY(inp, wires=i)

    # 参数化量子电路
    for wire in range(4):
        qml.RX(weights[wire], wires=wire)

    for wire in range(3):
        qml.CNOT(wires=[wire, wire+1])

    # 测量
    return [qml.expval(qml.PauliZ(i)) for i in range(4)]

# 在经典工作流中使用
inputs = np.array([0.1, 0.2, 0.3, 0.4])
weights = np.random.random(4)
output = quantum_layer(inputs, weights)
```

### 量子-经典流水线

```python
def hybrid_model(x, quantum_weights, classical_weights):
    # 经典预处理
    x_preprocessed = np.tanh(classical_weights['pre'] @ x)

    # 量子层
    quantum_out = quantum_layer(x_preprocessed, quantum_weights)

    # 经典后处理
    output = classical_weights['post'] @ quantum_out

    return output
```

## 框架集成

### PyTorch 集成

```python
import torch
import pennylane as qml

dev = qml.device('default.qubit', wires=2)

@qml.qnode(dev, interface='torch')
def quantum_circuit(inputs, weights):
    qml.RY(inputs[0], wires=0)
    qml.RY(inputs[1], wires=1)
    qml.RX(weights[0], wires=0)
    qml.RX(weights[1], wires=1)
    qml.CNOT(wires=[0, 1])
    return qml.expval(qml.PauliZ(0))

# 创建 PyTorch 层
class QuantumLayer(torch.nn.Module):
    def __init__(self, n_qubits):
        super().__init__()
        self.n_qubits = n_qubits
        self.weights = torch.nn.Parameter(torch.randn(n_qubits))

    def forward(self, x):
        return torch.stack([quantum_circuit(xi, self.weights) for xi in x])

# 在 PyTorch 模型中使用
class HybridModel(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.classical_1 = torch.nn.Linear(10, 2)
        self.quantum = QuantumLayer(2)
        self.classical_2 = torch.nn.Linear(1, 2)

    def forward(self, x):
        x = torch.relu(self.classical_1(x))
        x = self.quantum(x)
        x = self.classical_2(x.unsqueeze(1))
        return x

# 训练循环
model = HybridModel()
optimizer = torch.optim.Adam(model.parameters(), lr=0.01)
criterion = torch.nn.CrossEntropyLoss()

for epoch in range(100):
    optimizer.zero_grad()
    outputs = model(inputs)
    loss = criterion(outputs, labels)
    loss.backward()
    optimizer.step()
```

### JAX 集成

```python
import jax
import jax.numpy as jnp
import pennylane as qml

dev = qml.device('default.qubit', wires=2)

@qml.qnode(dev, interface='jax')
def quantum_circuit(inputs, weights):
    qml.RY(inputs[0], wires=0)
    qml.RY(inputs[1], wires=1)
    qml.RX(weights[0], wires=0)
    qml.RX(weights[1], wires=1)
    qml.CNOT(wires=[0, 1])
    return qml.expval(qml.PauliZ(0))

# JAX 兼容训练
@jax.jit
def loss_fn(weights, x, y):
    predictions = quantum_circuit(x, weights)
    return jnp.mean((predictions - y) ** 2)

# 使用 JAX 计算梯度
grad_fn = jax.grad(loss_fn)

# 训练
weights = jnp.array([0.1, 0.2])
for i in range(100):
    grads = grad_fn(weights, x_train, y_train)
    weights = weights - 0.01 * grads
```

### TensorFlow 集成

```python
import tensorflow as tf
import pennylane as qml

dev = qml.device('default.qubit', wires=2)

@qml.qnode(dev, interface='tf')
def quantum_circuit(inputs, weights):
    qml.RY(inputs[0], wires=0)
    qml.RY(inputs[1], wires=1)
    qml.RX(weights[0], wires=0)
    qml.RX(weights[1], wires=1)
    qml.CNOT(wires=[0, 1])
    return qml.expval(qml.PauliZ(0))

# Keras 层
class QuantumLayer(tf.keras.layers.Layer):
    def __init__(self, n_qubits):
        super().__init__()
        self.n_qubits = n_qubits
        weight_init = tf.random_uniform_initializer()
        self.weights = tf.Variable(
            initial_value=weight_init(shape=(n_qubits,), dtype=tf.float32),
            trainable=True
        )

    def call(self, inputs):
        return tf.stack([quantum_circuit(x, self.weights) for x in inputs])

# Keras 模型
model = tf.keras.Sequential([
    tf.keras.layers.Dense(2, activation='relu'),
    QuantumLayer(2),
    tf.keras.layers.Dense(2, activation='softmax')
])

model.compile(
    optimizer=tf.keras.optimizers.Adam(0.01),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

model.fit(x_train, y_train, epochs=100, batch_size=32)
```

## 量子神经网络

### 变分量子电路 (VQC)

```python
from pennylane import numpy as np

dev = qml.device('default.qubit', wires=4)

def variational_block(weights, wires):
    """变分电路的单层结构"""
    for i, wire in enumerate(wires):
        qml.RY(weights[i, 0], wires=wire)
        qml.RZ(weights[i, 1], wires=wire)

    for i in range(len(wires)-1):
        qml.CNOT(wires=[wires[i], wires[i+1]])

@qml.qnode(dev)
def quantum_neural_network(inputs, weights):
    # 编码输入
    for i, inp in enumerate(inputs):
        qml.RY(inp, wires=i)

    # 应用变分层
    n_layers = len(weights)
    for layer_weights in weights:
        variational_block(layer_weights, wires=range(4))

    return qml.expval(qml.PauliZ(0))

# 初始化权重
n_layers = 3
n_wires = 4
weights_shape = (n_layers, n_wires, 2)
weights = np.random.random(weights_shape, requires_grad=True)
```

### 量子卷积神经网络

```python
def conv_layer(weights, wires):
    """量子卷积层"""
    n_wires = len(wires)

    # 应用局部酉变换
    for i in range(n_wires):
        qml.RY(weights[i], wires=wires[i])

    # 最近邻纠缠
    for i in range(0, n_wires-1, 2):
        qml.CNOT(wires=[wires[i], wires[i+1]])

def pooling_layer(wires):
    """量子池化层（测量并丢弃）"""
    measurements = []
    for i in range(0, len(wires), 2):
        measurements.append(qml.measure(wires[i]))
    return measurements

@qml.qnode(dev)
def qcnn(inputs, weights):
    # 编码图像数据
    for i, pixel in enumerate(inputs):
        qml.RY(pixel, wires=i)

    # 卷积层
    conv_layer(weights[0], wires=range(8))
    pooling_layer(wires=range(0, 8, 2))

    conv_layer(weights[1], wires=range(1, 8, 2))
    pooling_layer(wires=range(1, 8, 4))

    return qml.expval(qml.PauliZ(1))
```

### 量子循环神经网络

```python
def qrnn_cell(x, hidden, weights):
    """单个 QRNN 单元"""
    @qml.qnode(dev)
    def cell(x, h, w):
        # 编码输入和隐藏状态
        qml.RY(x, wires=0)
        qml.RY(h, wires=1)

        # 应用循环变换
        qml.RX(w[0], wires=0)
        qml.RX(w[1], wires=1)
        qml.CNOT(wires=[0, 1])
        qml.RY(w[2], wires=1)

        return qml.expval(qml.PauliZ(1))

    return cell(x, hidden, weights)

def qrnn_sequence(sequence, weights):
    """使用 QRNN 处理序列"""
    hidden = 0.0
    outputs = []

    for x in sequence:
        hidden = qrnn_cell(x, hidden, weights)
        outputs.append(hidden)

    return outputs
```

## 变分分类器

### 二分类

```python
dev = qml.device('default.qubit', wires=2)

@qml.qnode(dev)
def variational_classifier(x, weights):
    # 特征映射
    qml.RY(x[0], wires=0)
    qml.RY(x[1], wires=1)

    # 变分层
    for w in weights:
        qml.RX(w[0], wires=0)
        qml.RX(w[1], wires=1)
        qml.CNOT(wires=[0, 1])
        qml.RY(w[2], wires=0)
        qml.RY(w[3], wires=1)

    return qml.expval(qml.PauliZ(0))

def cost_function(weights, X, y):
    """二元交叉熵损失"""
    predictions = np.array([variational_classifier(x, weights) for x in X])
    predictions = (predictions + 1) / 2  # 将[-1,1]映射到[0,1]
    return -np.mean(y * np.log(predictions) + (1 - y) * np.log(1 - predictions))

# 训练
n_layers = 2
n_params_per_layer = 4
weights = np.random.random((n_layers, n_params_per_layer), requires_grad=True)

opt = qml.GradientDescentOptimizer(stepsize=0.1)
for i in range(100):
    weights = opt.step(lambda w: cost_function(w, X_train, y_train), weights)
```

### 多类分类

```python
@qml.qnode(dev)
def multiclass_circuit(x, weights):
    # 编码输入
    for i, val in enumerate(x):
        qml.RY(val, wires=i)

    # 变分电路
    for layer_weights in weights:
        for i, w in enumerate(layer_weights):
            qml.RY(w, wires=i)
        for i in range(len(x)-1):
            qml.CNOT(wires=[i, i+1])

    # 多类输出
    return [qml.expval(qml.PauliZ(i)) for i in range(3)]

def softmax(x):
    exp_x = np.exp(x - np.max(x))
    return exp_x / exp_x.sum()

def predict_class(x, weights):
    logits = multiclass_circuit(x, weights)
    return softmax(logits)
```

## 训练与优化

### 基于梯度的训练

```python
# 自动微分
@qml.qnode(dev, diff_method='backprop')
def circuit_backprop(x, weights):
    # ... 电路定义
    return qml.expval(qml.PauliZ(0))

# 参数平移规则
@qml.qnode(dev, diff_method='parameter-shift')
def circuit_param_shift(x, weights):
    # ... 电路定义
    return qml.expval(qml.PauliZ(0))

# 有限差分法
@qml.qnode(dev, diff_method='finite-diff')
def circuit_finite_diff(x, weights):
    # ... 电路定义
    return qml.expval(qml.PauliZ(0))
```

### 小批量训练

```python
def batch_cost(weights, X_batch, y_batch):
    predictions = np.array([variational_classifier(x, weights) for x in X_batch])
    return np.mean((predictions - y_batch) ** 2)

# 小批量训练
batch_size = 32
n_epochs = 100

for epoch in range(n_epochs):
    for i in range(0, len(X_train), batch_size):
        X_batch = X_train[i:i+batch_size]
        y_batch = y_train[i:i+batch_size]

        weights = opt.step(lambda w: batch_cost(w, X_batch, y_batch), weights)
```

### 学习率调度

```python
def train_with_schedule(weights, X, y, n_epochs):
    initial_lr = 0.1
    decay = 0.95

    for epoch in range(n_epochs):
        lr = initial_lr * (decay ** epoch)
        opt = qml.GradientDescentOptimizer(stepsize=lr)

        weights = opt.step(lambda w: cost_function(w, X, y), weights)

        if epoch % 10 == 0:
            print(f"Epoch {epoch}, Loss: {cost_function(weights, X, y)}")

    return weights
```

## 数据编码策略

### 角度编码

```python
def angle_encoding(x, wires):
    """将特征编码为旋转角度"""
    for i, feature in enumerate(x):
        qml.RY(feature, wires=wires[i])
```

### 振幅编码

```python
def amplitude_encoding(x, wires):
    """将特征编码为态振幅"""
    # 归一化
    x_norm = x / np.linalg.norm(x)
    qml.MottonenStatePreparation(x_norm, wires=wires)
```

### 基态编码

```python
def basis_encoding(x, wires):
    """在计算基中编码二进制特征"""
    for i, bit in enumerate(x):
        if bit:
            qml.PauliX(wires=wires[i])
```

### IQP 编码

```python
def iqp_encoding(x, wires):
    """瞬时量子多项式编码"""
    # Hadamard 层
    for wire in wires:
        qml.Hadamard(wires=wire)

    # 编码特征
    for i, feature in enumerate(x):
        qml.RZ(feature, wires=wires[i])

    # 纠缠
    for i in range(len(wires)-1):
        qml.IsingZZ(x[i] * x[i+1], wires=[wires[i], wires[i+1]])
```

### 哈密顿量编码

```python
def hamiltonian_encoding(x, wires, time=1.0):
    """通过哈密顿演化编码"""
    # 从特征构建哈密顿量
    coeffs = x
    obs = [qml.PauliZ(i) for i in wires]

    H = qml.Hamiltonian(coeffs, obs)

    # 应用时间演化
    qml.ApproxTimeEvolution(H, time, n=10)
```

## 迁移学习

### 预训练量子模型

```python
# 在大数据集上训练
pretrained_weights = train_quantum_model(large_dataset)

# 在特定任务上微调
def fine_tune(pretrained_weights, small_dataset, n_epochs=50):
    # 冻结早期层
    frozen_weights = pretrained_weights[:-1]  # 除最后一层外
    trainable_weights = pretrained_weights[-1:]  # 仅最后一层

    @qml.qnode(dev)
    def transfer_circuit(x, trainable):
        # 应用冻结层
        for layer_w in frozen_weights:
            variational_block(layer_w, wires=range(4))

        # 应用可训练层
        variational_block(trainable, wires=range(4))

        return qml.expval(qml.Pauli

# 仅训练最后一层

```python
    opt = qml.AdamOptimizer(stepsize=0.01)
    for epoch in range(n_epochs):
        trainable_weights = opt.step(
            lambda w: cost_function(w, small_dataset),
            trainable_weights
        )

    return np.concatenate([frozen_weights, trainable_weights])
```

### 经典到量子迁移

```python
# 使用经典网络进行特征提取
import torch.nn as nn

classical_extractor = nn.Sequential(
    nn.Conv2d(3, 16, 3),
    nn.ReLU(),
    nn.MaxPool2d(2),
    nn.Flatten(),
    nn.Linear(16*13*13, 4)  # 输出4个特征供量子电路使用
)

# 量子分类器
@qml.qnode(dev)
def quantum_classifier(features, weights):
    angle_encoding(features, wires=range(4))
    variational_block(weights, wires=range(4))
    return qml.expval(qml.PauliZ(0))

# 混合模型
def hybrid_transfer_model(image, classical_weights, quantum_weights):
    features = classical_extractor(image)
    return quantum_classifier(features, quantum_weights)
```

## 最佳实践

1. **从简单开始** - 先使用小型电路再逐步扩展
2. **明智选择编码方式** - 使编码匹配数据结构
3. **使用合适接口** - 选择匹配机器学习框架的接口
4. **监控梯度** - 检查梯度消失/爆炸问题（贫瘠高原现象）
5. **正则化** - 添加L2正则化防止过拟合
6. **验证硬件兼容性** - 先在模拟器测试再上硬件
7. **高效批处理** - 尽可能使用向量化
8. **缓存编译结果** - 推理时复用已编译电路
