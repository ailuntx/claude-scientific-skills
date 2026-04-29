# PyHealth 模型

## 概述

PyHealth 提供 33 种以上用于医疗预测任务的模型，涵盖从简单基线到最先进深度学习架构。模型分为通用架构和医疗专用模型两大类。

## 模型基类

所有模型继承自 `BaseModel`，具备标准 PyTorch 功能：

**关键属性：**
- `dataset`：关联的 SampleDataset
- `feature_keys`：使用的输入特征（如 ["diagnoses", "medications"]）
- `mode`：任务类型（"binary", "multiclass", "multilabel", "regression"）
- `embedding_dim`：特征嵌入维度
- `device`：计算设备（CPU/GPU）

**关键方法：**
- `forward()`：模型前向传播
- `train_step()`：单次训练迭代
- `eval_step()`：单次评估迭代
- `save()`：保存模型检查点
- `load()`：加载模型检查点

## 通用模型

### 基线模型

**逻辑回归** (`LogisticRegression`)
- 带均值池化的线性分类器
- 用于对比的简单基线
- 训练和推理速度快
- 可解释性强

**用法：**
```python
from pyhealth.models import LogisticRegression

model = LogisticRegression(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="binary"
)
```

**多层感知机** (`MLP`)
- 前馈神经网络
- 可配置隐藏层
- 支持均值/求和/最大池化
- 结构化数据的优秀基线

**参数：**
- `hidden_dim`：隐藏层大小
- `num_layers`：隐藏层数量
- `dropout`：丢弃率
- `pooling`：聚合方法（"mean", "sum", "max"）

**用法：**
```python
from pyhealth.models import MLP

model = MLP(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="binary",
    hidden_dim=128,
    num_layers=3,
    dropout=0.5
)
```

### 卷积神经网络

**CNN** (`CNN`)
- 用于模式检测的卷积层
- 对序列和空间数据高效
- 捕捉局部时序模式
- 参数效率高

**架构：**
- 多个一维卷积层
- 最大池化降维
- 全连接输出层

**参数：**
- `num_filters`：卷积滤波器数量
- `kernel_size`：卷积核尺寸
- `num_layers`：卷积层数量
- `dropout`：丢弃率

**用法：**
```python
from pyhealth.models import CNN

model = CNN(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="binary",
    num_filters=64,
    kernel_size=3,
    num_layers=3
)
```

**时序卷积网络** (`TCN`)
- 空洞卷积处理长程依赖
- 因果卷积（避免未来信息泄露）
- 长序列处理高效
- 适用于时间序列预测

**优势：**
- 捕捉长期依赖
- 可并行化（比 RNN 更快）
- 梯度稳定

### 循环神经网络

**RNN** (`RNN`)
- 基础循环架构
- 支持 LSTM、GRU、RNN 变体
- 序列处理能力
- 捕捉时序依赖

**参数：**
- `rnn_type`："LSTM", "GRU" 或 "RNN"
- `hidden_dim`：隐藏状态维度
- `num_layers`：循环层数量
- `dropout`：丢弃率
- `bidirectional`：使用双向 RNN

**用法：**
```python
from pyhealth.models import RNN

model = RNN(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="binary",
    rnn_type="LSTM",
    hidden_dim=128,
    num_layers=2,
    bidirectional=True
)
```

**最佳适用场景：**
- 临床事件序列
- 时序模式学习
- 变长序列处理

### Transformer 模型

**Transformer** (`Transformer`)
- 自注意力机制
- 序列并行处理
- 最先进性能
- 长程依赖处理高效

**架构：**
- 多头自注意力
- 位置嵌入
- 前馈网络
- 层归一化

**参数：**
- `num_heads`：注意力头数量
- `num_layers`：Transformer 层数
- `hidden_dim`：隐藏维度
- `dropout`：丢弃率
- `max_seq_length`：最大序列长度

**用法：**
```python
from pyhealth.models import Transformer

model = Transformer(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="binary",
    num_heads=8,
    num_layers=6,
    hidden_dim=256,
    dropout=0.1
)
```

**TransformersModel** (`TransformersModel`)
- 集成 HuggingFace transformers
- 临床文本预训练语言模型
- 医疗任务微调
- 示例：BERT, RoBERTa, BioClinicalBERT

**用法：**
```python
from pyhealth.models import TransformersModel

model = TransformersModel(
    dataset=sample_dataset,
    feature_keys=["text"],
    mode="multiclass",
    pretrained_model="emilyalsentzer/Bio_ClinicalBERT"
)
```

### 图神经网络

**GNN** (`GNN`)
- 基于图的学习
- 建模实体间关系
- 支持 GAT（图注意力）和 GCN（图卷积）

**适用场景：**
- 药物相互作用
- 患者相似性网络
- 知识图谱集成
- 共病关系建模

**参数：**
- `gnn_type`："GAT" 或 "GCN"
- `hidden_dim`：隐藏维度
- `num_layers`：GNN 层数
- `dropout`：丢弃率
- `num_heads`：注意力头数（GAT 专用）

**用法：**
```python
from pyhealth.models import GNN

model = GNN(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="multilabel",
    gnn_type="GAT",
    hidden_dim=128,
    num_layers=3,
    num_heads=4
)
```

## 医疗专用模型

### 可解释临床模型

**RETAIN** (`RETAIN`)
- 逆时序注意力机制
- 高可解释性预测
- 就诊级和事件级注意力
- 识别关键临床事件

**核心特性：**
- 双级注意力（就诊与特征）
- 时序衰减建模
- 临床可解释性
- 发表于 NeurIPS 2016

**用法：**
```python
from pyhealth.models import RETAIN

model = RETAIN(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="binary",
    hidden_dim=128
)

# 获取注意力权重用于解释
outputs = model(batch)
visit_attention = outputs["visit_attention"]
feature_attention = outputs["feature_attention"]
```

**最佳适用场景：**
- 死亡率预测
- 再入院预测
- 临床风险评分
- 可解释预测

**AdaCare** (`AdaCare`)
- 带特征校准的自适应护理模型
- 疾病特异性注意力
- 处理不规则时间间隔
- 可解释特征重要性

**ConCare** (`ConCare`)
- 跨就诊卷积注意力
- 时序卷积特征提取
- 多级注意力机制
- 适用于纵向 EHR 建模

### 用药推荐模型

**GAMENet** (`GAMENet`)
- 基于图的用药推荐
- 药物相互作用建模
- 患者历史记忆网络
- 多跳推理

**架构：**
- 药物知识图谱
- 记忆增强神经网络
- DDI 感知预测

**用法：**
```python
from pyhealth.models import GAMENet

model = GAMENet(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="multilabel",
    embedding_dim=128,
    ddi_adj_path="/path/to/ddi_adjacency_matrix.pkl"
)
```

**MICRON** (`MICRON`)
- 带 DDI 约束的用药推荐
- 相互作用感知预测
- 安全导向药物选择

**SafeDrug** (`SafeDrug`)
- 安全感知药物推荐
- 分子结构集成
- DDI 约束优化
- 平衡疗效与安全性

**核心特性：**
- 分子图编码
- DDI 图神经网络
- 强化学习保障安全
- 发表于 KDD 2021

**用法：**
```python
from pyhealth.models import SafeDrug

model = SafeDrug(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="multilabel",
    ddi_adj_path="/path/to/ddi_matrix.pkl",
    molecule_path="/path/to/molecule_graphs.pkl"
)
```

**MoleRec** (`MoleRec`)
- 分子级药物推荐
- 子结构推理
- 细粒度药物选择

### 疾病进展模型

**StageNet** (`StageNet`)
- 疾病分期感知预测
- 自动学习临床分期
- 分期自适应特征提取
- 慢性病监测高效模型

**架构：**
- 分期感知 LSTM
- 动态分期转换
- 时间衰减机制

**用法：**
```python
from pyhealth.models import StageNet

model = StageNet(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="binary",
    hidden_dim=128,
    num_stages=3,
    chunk_size=128
)
```

**最佳适用场景：**
- ICU 死亡率预测
- 慢性病进展
- 时变风险评估

**Deepr** (`Deepr`)
- 深度循环架构
- 医疗概念嵌入
- 时序模式学习
- 发表于 JAMIA

### 高级序列模型

**Agent** (`Agent`)
- 基于强化学习
- 治疗推荐
- 动作价值优化
- 序列决策策略学习

**GRASP** (`GRASP`)
- 基于图的序列模式
- 结构化事件关系
- 分层表示学习

**SparcNet** (`SparcNet`)
- 稀疏临床网络
- 高效特征选择
- 降低计算成本
- 可解释预测

**ContraWR** (`ContraWR`)
- 对比学习方法
- 自监督预训练
- 鲁棒表示学习
- 小样本场景适用

### 医疗实体链接

**MedLink** (`MedLink`)
- 医疗实体链接到知识库
- 临床概念标准化
- UMLS 集成
- 实体消歧

### 生成模型

**GAN** (`GAN`)
- 生成对抗网络
- 合成 EHR 数据生成
- 隐私保护数据共享
- 罕见病症数据增强

**VAE** (`VAE`)
- 变分自编码器
- 患者表示学习
- 异常检测
- 隐空间探索

### 健康社会决定因素

**SDOH** (`SDOH`)
- 健康社会决定因素集成
- 多模态预测
- 解决健康差异
- 结合临床与社会数据

## 模型选择指南

### 按任务类型

**二分类任务**（死亡率、再入院）
- 初始选择：逻辑回归（基线）
- 标准方案：RNN, Transformer
- 可解释方案：RETAIN, AdaCare
- 高级方案：StageNet

**多标签分类**（药物推荐）
- 标准方案：CNN, RNN
- 医疗专用：GAMENet, SafeDrug, MICRON, MoleRec
- 图基础方案：GNN

**回归任务**（住院时长）
- 初始选择：MLP（基线）
- 序列方案：RNN, TCN
- 高级方案：Transformer

**多分类任务**（医疗编码、专科分类）
- 标准方案：CNN, RNN, Transformer
- 文本方案：TransformersModel（BERT 变体）

### 按数据类型

**序列事件**（诊断、用药、手术）
- RNN, LSTM, GRU
- Transformer
- RETAIN, AdaCare, ConCare

**时序信号**（EEG, ECG）
- CNN, TCN
- RNN
- Transformer

**文本数据**（临床记录）
- TransformersModel（ClinicalBERT, BioBERT）
- CNN（短文本）
- RNN（序列文本）

**图数据**（药物相互作用、患者网络）
- GNN（GAT, GCN）
- GAMENet, SafeDrug

**影像数据**（X光、CT）
- CNN（通过 TransformersModel 集成 ResNet, DenseNet）
- Vision Transformers

### 按可解释性需求

**高可解释性要求：**
- 逻辑回归
- RETAIN
- AdaCare
- SparcNet

**中等可解释性：**
- CNN（滤波器可视化）
- Transformer（注意力可视化）
- GNN（图注意力）

**可接受黑盒：**
- 深度 RNN 模型
- 复杂集成模型

## 训练注意事项

### 超参数调优

**嵌入维度：**
- 小数据集：64-128
- 大数据集：128-256
- 复杂任务：256-512

**隐藏维度：**
- 与 embedding_dim 成比例
- 通常为 embedding_dim 的 1-2 倍

**层数设置：**
- 初始 2-3 层
- 复杂模式增加层数
- 警惕过拟合

**丢弃率：**
- 初始值 0.5
- 欠拟合时降低（0.1-0.3）
- 过拟合时提高（0.5-0.7）

### 计算需求

**显存（GPU）：**
- CNN：低至中等
- RNN：中等（依赖序列长度）
- Transformer：高（序列长度平方级）
- GNN：中至高（依赖图规模）

**训练速度：**
- 最快：逻辑回归、MLP、CNN
- 中等：RNN、GNN
- 较慢：Transformer（但可并行）

### 最佳实践

1. **从简单基线开始**（逻辑回归、MLP）
2. **根据数据可用性选择特征键**
3. **任务输出与模式匹配**（二分类/多分类/多标签/回归）
4. **临床部署考虑可解释性需求**
5. **保留测试集验证**确保性能可靠
6. **监控过拟合**（尤其复杂模型）
7. **优先使用预训练模型**（TransformersModel）
8. **部署时考虑计算限制**

## 示例工作流

```python
from pyhealth.datasets import MIMIC4Dataset
from pyhealth.tasks import mortality_prediction_mimic4_fn
from pyhealth.models import Transformer
from pyhealth.trainer import Trainer

# 1. 准备数据
dataset = MIMIC4Dataset(root="/path/to/data")
sample_dataset = dataset.set_task(mortality_prediction_mimic4_fn)

# 2. 初始化模型
model = Transformer(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications", "procedures"],
    mode="binary",
    embedding_dim=128,
    num_heads=8,
    num_layers=3,
    dropout=0.3
)

# 3. 训练模型
trainer = Trainer(model=model)
trainer.train(
    train_dataloader=train_loader,
    val_dataloader=val_loader,
    epochs=50,
    monitor="pr_auc_score",
    monitor_criterion="max"
)

# 4. 评估
results = trainer.evaluate(test_loader)
print(results)
```
