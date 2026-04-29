# 深度学习网络

Aeon 提供了专为时间序列任务设计的神经网络架构。这些网络可作为分类、回归、聚类和预测任务的基础构建模块。

## 核心网络架构

### 卷积网络

**FCNNetwork** - 全卷积网络
- 三个带批量归一化的卷积块
- 全局平均池化用于降维
- **适用场景**：需要简单有效的CNN基线模型

**ResNetNetwork** - 残差网络
- 带跳跃连接的残差块
- 防止深层网络梯度消失
- **适用场景**：需深层网络且训练稳定性重要时

**InceptionNetwork** - Inception模块
- 并行卷积实现多尺度特征提取
- 不同卷积核尺寸捕获多尺度模式
- **适用场景**：存在多时间尺度模式时

**TimeCNNNetwork** - 标准CNN
- 基础卷积架构
- **适用场景**：简单CNN足够且重视可解释性时

**DisjointCNNNetwork** - 分离路径
- 独立卷积路径
- **适用场景**：需不同特征提取策略时

**DCNNNetwork** - 空洞卷积网络
- 空洞卷积扩大感受野
- **适用场景**：需长程依赖但不宜堆叠过多层时

### 循环网络

**RecurrentNetwork** - RNN/LSTM/GRU
- 可配置单元类型（RNN/LSTM/GRU）
- 时序依赖关系的序列建模
- **适用场景**：序列依赖关键且处理变长序列时

### 时序卷积网络

**TCNNetwork** - 时序卷积网络
- 空洞因果卷积
- 无需循环结构的大感受野
- **适用场景**：长序列且需并行化架构时

### 多层感知机

**MLPNetwork** - 基础前馈网络
- 简单全连接层
- 处理前将时间序列展平
- **适用场景**：需基线模型/计算受限/简单模式时

## 编码器架构

专为表征学习和聚类设计的网络。

### 自编码器变体

**EncoderNetwork** - 通用编码器
- 灵活编码结构
- **适用场景**：需自定义编码时

**AEFCNNetwork** - FCN自编码器
- 全卷积编码器-解码器
- **适用场景**：需卷积表征学习时

**AEResNetNetwork** - ResNet自编码器
- 编码器-解码器含残差块
- **适用场景**：需带跳跃连接的深度自编码时

**AEDCNNNetwork** - 空洞CNN自编码器
- 空洞卷积实现压缩
- **适用场景**：自编码器需大感受野时

**AEDRNNNetwork** - 空洞RNN自编码器
- 空洞循环连接
- **适用场景**：含长程依赖的序列模式时

**AEBiGRUNetwork** - 双向GRU
- 双向循环编码
- **适用场景**：需双向上下文信息时

**AEAttentionBiGRUNetwork** - 注意力+双向GRU
- 在BiGRU输出添加注意力机制
- **适用场景**：需聚焦关键时间步时

## 专用架构

**LITENetwork** - 轻量级Inception时间集成
- 高效Inception架构
- LITEMV变体支持多元序列
- **适用场景**：需高效且强性能时

**DeepARNetwork** - 概率预测
- 自回归RNN用于预测
- 生成概率预测结果
- **适用场景**：需预测不确定性量化时

## 与评估器配合使用

网络通常通过评估器间接使用：

```python
from aeon.classification.deep_learning import FCNClassifier
from aeon.regression.deep_learning import ResNetRegressor
from aeon.clustering.deep_learning import AEFCNClusterer

# FCN分类
clf = FCNClassifier(n_epochs=100, batch_size=16)
clf.fit(X_train, y_train)

# ResNet回归
reg = ResNetRegressor(n_epochs=100)
reg.fit(X_train, y_train)

# 自编码器聚类
clusterer = AEFCNClusterer(n_clusters=3, n_epochs=100)
labels = clusterer.fit_predict(X_train)
```

## 自定义网络配置

多数网络支持配置参数：

```python
# 配置FCN层
clf = FCNClassifier(
    n_epochs=200,
    batch_size=32,
    kernel_size=[7, 5, 3],  # 各层卷积核尺寸
    n_filters=[128, 256, 128],  # 每层滤波器数
    learning_rate=0.001
)
```

## 基础类

- `BaseDeepLearningNetwork` - 所有网络抽象基类
- `BaseDeepRegressor` - 深度回归基类
- `BaseDeepClassifier` - 深度分类基类
- `BaseDeepForecaster` - 深度预测基类

可通过扩展这些类实现自定义架构。

## 训练注意事项

### 超参数

关键调优超参数：
- `n_epochs` - 训练轮次（通常50-200）
- `batch_size` - 批次样本量（通常16-64）
- `learning_rate` - 学习率（0.0001-0.01）
- 网络特定参数：层数/滤波器数/卷积核尺寸

### 回调函数

多数网络支持训练监控回调：

```python
from tensorflow.keras.callbacks import EarlyStopping, ReduceLROnPlateau

clf = FCNClassifier(
    n_epochs=200,
    callbacks=[
        EarlyStopping(patience=20, restore_best_weights=True),
        ReduceLROnPlateau(patience=10, factor=0.5)
    ]
)
```

### GPU加速

深度学习网络受益于GPU：

```python
import os
os.environ['CUDA_VISIBLE_DEVICES'] = '0'  # 使用首块GPU

# 网络在GPU可用时自动启用
clf = InceptionTimeClassifier(n_epochs=100)
clf.fit(X_train, y_train)
```

## 架构选择指南

### 按任务类型：

**分类**：InceptionNetwork, ResNetNetwork, FCNNetwork  
**回归**：InceptionNetwork, ResNetNetwork, TCNNetwork  
**预测**：TCNNetwork, DeepARNetwork, RecurrentNetwork  
**聚类**：AEFCNNetwork, AEResNetNetwork, AEAttentionBiGRUNetwork  

### 按数据特征：

**长序列**：TCNNetwork, DCNNNetwork（空洞卷积）  
**短序列**：MLPNetwork, FCNNetwork  
**多元序列**：InceptionNetwork, FCNNetwork, LITENetwork  
**变长序列**：带掩码的RecurrentNetwork  
**多尺度模式**：InceptionNetwork  

### 按计算资源：

**有限算力**：MLPNetwork, LITENetwork  
**中等算力**：FCNNetwork, TimeCNNNetwork  
**充足算力**：InceptionNetwork, ResNetNetwork  
**GPU可用**：任意深度网络（显著加速）  

## 最佳实践

### 1. 数据预处理

标准化输入数据：

```python
from aeon.transformations.collection import Normalizer

normalizer = Normalizer()
X_train_norm = normalizer.fit_transform(X_train)
X_test_norm = normalizer.transform(X_test)
```

### 2. 训练/验证集划分

使用验证集进行早停：

```python
from sklearn.model_selection import train_test_split

X_train_fit, X_val, y_train_fit, y_val = train_test_split(
    X_train, y_train, test_size=0.2, stratify=y_train
)

clf = FCNClassifier(n_epochs=200)
clf.fit(X_train_fit, y_train_fit, validation_data=(X_val, y_val))
```

### 3. 从简开始

先尝试简单架构再过渡到复杂模型：
1. 先试MLPNetwork或FCNNetwork
2. 效果不足时尝试ResNetNetwork或InceptionNetwork
3. 单模型不足时可考虑集成方法

### 4. 超参数调优

使用网格搜索或随机搜索：

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    'n_epochs': [100, 200],
    'batch_size': [16, 32],
    'learning_rate': [0.001, 0.0001]
}

clf = FCNClassifier()
grid = GridSearchCV(clf, param_grid, cv=3)
grid.fit(X_train, y_train)
```

### 5. 正则化

防止过拟合：
- 使用Dropout（若网络支持）
- 早停法
- 数据增强（若可用）
- 降低模型复杂度

### 6. 可复现性

设置随机种子：

```python
import numpy as np
import random
import tensorflow as tf

seed = 42
np.random.seed(seed)
random.seed(seed)
tf.random.set_seed(seed)
```
