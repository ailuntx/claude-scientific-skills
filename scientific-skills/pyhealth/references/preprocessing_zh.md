# PyHealth 数据预处理与处理器

## 概述

PyHealth 提供全面的数据处理工具，可将原始医疗数据转换为模型就绪格式。处理器负责特征提取、序列处理、信号转换和标签准备。

## 处理器基类

所有处理器均继承自 `Processor` 类，具有标准接口：

**核心方法：**
- `__call__()`：转换输入数据
- `get_input_info()`：返回处理后的输入模式
- `get_output_info()`：返回处理后的输出模式

## 核心处理器类型

### 特征处理器

**特征处理器** (`FeatureProcessor`)
- 特征提取的基类
- 处理词汇表构建
- 嵌入准备
- 特征编码

**常用操作：**
- 医疗代码分词
- 分类编码
- 特征归一化
- 缺失值处理

**用法：**
```python
from pyhealth.data import FeatureProcessor

processor = FeatureProcessor(
    vocabulary="diagnoses",
    min_freq=5,  # 最小代码出现频次
    max_vocab_size=10000
)

processed_features = processor(raw_features)
```

### 序列处理器

**序列处理器** (`SequenceProcessor`)
- 处理临床事件序列
- 保持时序顺序
- 序列填充/截断
- 时间间隔编码

**关键特性：**
- 变长序列处理
- 时序特征提取
- 序列统计计算

**参数：**
- `max_seq_length`：最大序列长度（超长则截断）
- `padding`：填充策略（"pre" 或 "post"）
- `truncating`：截断策略（"pre" 或 "post"）

**用法：**
```python
from pyhealth.data import SequenceProcessor

processor = SequenceProcessor(
    max_seq_length=100,
    padding="post",
    truncating="post"
)

# 处理诊断序列
processed_seq = processor(diagnosis_sequences)
```

**嵌套序列处理器** (`NestedSequenceProcessor`)
- 处理分层序列（如包含事件的就诊记录）
- 两级处理（就诊级和事件级）
- 保留嵌套结构

**适用场景：**
- 包含多次事件的电子健康记录
- 多级时序建模
- 分层注意力模型

**结构：**
```python
# 输入：[[visit1_events], [visit2_events], ...]
# 输出：带填充的已处理嵌套序列
```

### 数值数据处理器

**嵌套浮点处理器** (`NestedFloatsProcessor`)
- 处理嵌套数值数组
- 实验室值、生命体征、测量数据
- 多级数值特征

**操作：**
- 归一化
- 标准化
- 缺失值填补
- 异常值处理

**用法：**
```python
from pyhealth.data import NestedFloatsProcessor

processor = NestedFloatsProcessor(
    normalization="z-score",  # 或 "min-max"
    fill_missing="mean"  # 填补策略
)

processed_labs = processor(lab_values)
```

**张量处理器** (`TensorProcessor`)
- 将数据转换为 PyTorch 张量
- 类型处理（long, float 等）
- 设备分配（CPU/GPU）

**参数：**
- `dtype`：张量数据类型
- `device`：计算设备

### 时序处理器

**时序处理器** (`TimeseriesProcessor`)
- 处理带时间戳的时序数据
- 时间间隔计算
- 时序特征工程
- 不规则采样处理

**提取特征：**
- 距上次事件时间
- 距下次事件时间
- 事件频率
- 时序模式

**用法：**
```python
from pyhealth.data import TimeseriesProcessor

processor = TimeseriesProcessor(
    time_unit="hour",  # "day", "hour", "minute"
    compute_gaps=True,
    compute_frequency=True
)

processed_ts = processor(timestamps, events)
```

**信号处理器** (`SignalProcessor`)
- 生理信号处理
- EEG、ECG、PPG 信号
- 滤波与预处理

**操作：**
- 带通滤波
- 伪迹去除
- 分段处理
- 特征提取（频率、振幅）

**用法：**
```python
from pyhealth.data import SignalProcessor

processor = SignalProcessor(
    sampling_rate=256,  # Hz
    bandpass_filter=(0.5, 50),  # Hz 范围
    segment_length=30  # 秒
)

processed_signal = processor(raw_eeg_signal)
```

### 图像处理器

**图像处理器** (`ImageProcessor`)
- 医学图像预处理
- 归一化与尺寸调整
- 增强支持
- 格式标准化

**操作：**
- 调整至标准尺寸
- 归一化（均值/标准差）
- 窗宽调整（CT/MRI）
- 数据增强

**用法：**
```python
from pyhealth.data import ImageProcessor

processor = ImageProcessor(
    image_size=(224, 224),
    normalization="imagenet",  # 或自定义均值/标准差
    augmentation=True
)

processed_image = processor(raw_image)
```

## 标签处理器

### 二分类

**二分类标签处理器** (`BinaryLabelProcessor`)
- 二分类标签处理（0/1）
- 处理正负类别
- 类别加权处理不平衡

**用法：**
```python
from pyhealth.data import BinaryLabelProcessor

processor = BinaryLabelProcessor(
    positive_class=1,
    class_weight="balanced"
)

processed_labels = processor(raw_labels)
```

### 多分类

**多分类标签处理器** (`MultiClassLabelProcessor`)
- 多分类处理（互斥类别）
- 标签编码
- 类别平衡

**参数：**
- `num_classes`：类别数量
- `class_weight`：加权策略

**用法：**
```python
from pyhealth.data import MultiClassLabelProcessor

processor = MultiClassLabelProcessor(
    num_classes=5,  # 例如睡眠分期：W, N1, N2, N3, REM
    class_weight="balanced"
)

processed_labels = processor(raw_labels)
```

### 多标签分类

**多标签处理器** (`MultiLabelProcessor`)
- 多标签分类（单样本多标签）
- 各标签二进制编码
- 标签共现处理

**适用场景：**
- 药物推荐（多药物）
- ICD 编码（多诊断）
- 共病预测

**用法：**
```python
from pyhealth.data import MultiLabelProcessor

processor = MultiLabelProcessor(
    num_labels=100,  # 总可能标签数
    threshold=0.5  # 预测阈值
)

processed_labels = processor(raw_label_sets)
```

### 回归

**回归标签处理器** (`RegressionLabelProcessor`)
- 连续值预测
- 目标缩放与归一化
- 异常值处理

**适用场景：**
- 住院时长预测
- 实验室值预测
- 风险评分估计

**用法：**
```python
from pyhealth.data import RegressionLabelProcessor

processor = RegressionLabelProcessor(
    normalization="z-score",  # 或 "min-max"
    clip_outliers=True,
    outlier_std=3  # 3个标准差处截断
)

processed_targets = processor(raw_values)
```

## 专用处理器

### 文本处理

**文本处理器** (`TextProcessor`)
- 临床文本预处理
- 分词
- 词汇表构建
- 序列编码

**操作：**
- 小写转换
- 标点移除
- 医学术语缩写处理
- 词频过滤

**用法：**
```python
from pyhealth.data import TextProcessor

processor = TextProcessor(
    tokenizer="word",  # 或 "sentencepiece", "bpe"
    lowercase=True,
    max_vocab_size=50000,
    min_freq=5
)

processed_text = processor(clinical_notes)
```

### 模型专用处理器

**StageNet处理器** (`StageNetProcessor`)
- StageNet 模型专用预处理
- 分块序列处理
- 阶段感知特征提取

**用法：**
```python
from pyhealth.data import StageNetProcessor

processor = StageNetProcessor(
    chunk_size=128,
    num_stages=3
)

processed_data = processor(sequential_data)
```

**StageNet张量处理器** (`StageNetTensorProcessor`)
- StageNet 张量转换
- 批处理与填充
- 阶段掩码生成

### 原始数据处理

**原始处理器** (`RawProcessor`)
- 最小化预处理
- 预处理的直通处理
- 自定义预处理场景

**用法：**
```python
from pyhealth.data import RawProcessor

processor = RawProcessor()
processed_data = processor(data)  # 最小转换
```

## 样本级处理

**样本处理器** (`SampleProcessor`)
- 处理完整样本（输入+输出）
- 协调多个处理器
- 端到端预处理流水线

**工作流：**
1. 应用输入处理器处理特征
2. 应用输出处理器处理标签
3. 组合为模型就绪样本

**用法：**
```python
from pyhealth.data import SampleProcessor

processor = SampleProcessor(
    input_processors={
        "diagnoses": SequenceProcessor(max_seq_length=50),
        "medications": SequenceProcessor(max_seq_length=30),
        "labs": NestedFloatsProcessor(normalization="z-score")
    },
    output_processor=BinaryLabelProcessor()
)

processed_sample = processor(raw_sample)
```

## 数据集级处理

**数据集处理器** (`DatasetProcessor`)
- 处理完整数据集
- 批处理
- 并行处理支持
- 缓存优化效率

**操作：**
- 对所有样本应用处理器
- 从数据集生成词汇表
- 计算数据集统计量
- 保存处理后的数据

**用法：**
```python
from pyhealth.data import DatasetProcessor

processor = DatasetProcessor(
    sample_processor=sample_processor,
    num_workers=4,  # 并行处理
    cache_dir="/path/to/cache"
)

processed_dataset = processor(raw_dataset)
```

## 常用预处理流程

### 流程1：电子健康记录死亡率预测

```python
from pyhealth.data import (
    SequenceProcessor,
    BinaryLabelProcessor,
    SampleProcessor
)

# 定义处理器
input_processors = {
    "diagnoses": SequenceProcessor(max_seq_length=50),
    "medications": SequenceProcessor(max_seq_length=30),
    "procedures": SequenceProcessor(max_seq_length=20)
}

output_processor = BinaryLabelProcessor(class_weight="balanced")

# 组合样本处理器
sample_processor = SampleProcessor(
    input_processors=input_processors,
    output_processor=output_processor
)

# 处理数据集
processed_samples = [sample_processor(s) for s in raw_samples]
```

### 流程2：基于EEG的睡眠分期

```python
from pyhealth.data import (
    SignalProcessor,
    MultiClassLabelProcessor,
    SampleProcessor
)

# 信号预处理
signal_processor = SignalProcessor(
    sampling_rate=100,
    bandpass_filter=(0.3, 35),  # EEG频率范围
    segment_length=30  # 30秒时段
)

# 标签处理
label_processor = MultiClassLabelProcessor(
    num_classes=5,  # W, N1, N2, N3, REM
    class_weight="balanced"
)

# 组合
sample_processor = SampleProcessor(
    input_processors={"signal": signal_processor},
    output_processor=label_processor
)
```

### 流程3：药物推荐

```python
from pyhealth.data import (
    SequenceProcessor,
    MultiLabelProcessor,
    SampleProcessor
)

# 输入处理
input_processors = {
    "diagnoses": SequenceProcessor(max_seq_length=50),
    "previous_medications": SequenceProcessor(max_seq_length=40)
}

# 多标签输出（多药物）
output_processor = MultiLabelProcessor(
    num_labels=150,  # 可能药物总数
    threshold=0.5
)

sample_processor = SampleProcessor(
    input_processors=input_processors,
    output_processor=output_processor
)
```

### 流程4：住院时长预测

```python
from pyhealth.data import (
    SequenceProcessor,
    NestedFloatsProcessor,
    RegressionLabelProcessor,
    SampleProcessor
)

# 处理不同特征类型
input_processors = {
    "diagnoses": SequenceProcessor(max_seq_length=30),
    "procedures": SequenceProcessor(max_seq_length=20),
    "labs": NestedFloatsProcessor(
        normalization="z-score",
        fill_missing="mean"
    )
}

# 回归目标
output_processor = RegressionLabelProcessor(
    normalization="log",  # 住院时长对数转换
    clip_outliers=True
)

sample_processor = SampleProcessor(
    input_processors=input_processors,
    output_processor=output_processor
)
```

## 最佳实践

### 序列处理

1. **选择合适 max_seq_length**：平衡上下文与计算量
   - 短序列 (20-50)：快速但上下文少
   - 中序列 (50-100)：良好平衡
   - 长序列 (100+)：上下文丰富但速度慢

2. **截断策略**：
   - "post"：保留近期事件（临床预测推荐）
   - "pre"：保留早期事件

3. **填充策略**：
   - "post"：末端填充（标准）
   - "pre"：前端填充

### 特征编码

1. **词汇表大小**：限制为高频代码
   - `min_freq=5`：包含出现≥5次的代码
   - `max_vocab_size=10000`：限制总词汇量

2. **处理稀有代码**：归入 "unknown" 类别

3. **缺失值处理**：
   - 填补（均值、中位数、前向填充）
   - 指示变量
   - 特殊标记

### 归一化

1. **数值特征**：始终归一化
   - Z-score：标准缩放（均值=0，标准差=1）
   - Min-max：范围缩放 [0, 1]

2. **仅用训练集计算统计量**：防止数据泄露

3. **对验证/测试集应用相同归一化**

### 类别不平衡

1. **使用类别加权**：`class_weight="balanced"`

2. **考虑过采样**：针对极稀有正例

3. **使用合适评估指标**：AUROC、AUPRC、F1

### 性能优化

1. **缓存处理数据**：保存预处理结果

2. **并行处理**：DataLoader 使用 `num_workers`

3. **批处理**：单次处理多样本

4. **特征选择**：移除低信息量特征

### 验证

1. **检查处理后的维度**：确保正确形状

2. **验证值范围**：归一化后检查

3. **人工检查样本**：审查处理后的数据

4. **监控内存使用**：尤其大型数据集

## 故障排除

### 常见问题

**内存错误：**
- 减小 `max_seq_length`
- 使用更小批次
- 分块处理数据
- 启用磁盘缓存

**处理缓慢：**
- 启用并行处理 (`num_workers`)
- 缓存预处理数据
- 降低特征维度
- 使用高效数据类型

**维度不匹配：**
- 检查序列长度
- 验证填充配置
- 确保处理器设置一致

**NaN值：**
- 显式处理缺失数据
- 检查归一化参数
- 验证填补策略

**类别不平衡：**
- 使用类别加权
- 考虑过采样
- 调整决策阈值
- 使用合适评估指标
