---
name: transformers
description: 该技能适用于使用预训练Transformer模型处理自然语言处理、计算机视觉、音频或多模态任务。可用于文本生成、分类、问答、翻译、摘要、图像分类、目标检测、语音识别以及在自定义数据集上微调模型。
license: Apache-2.0 许可证
compatibility: 部分功能需要Huggingface令牌
metadata:
    skill-author: K-Dense Inc.
---

# Transformers

## 概述

Hugging Face Transformers库提供数千个预训练模型，覆盖NLP、计算机视觉、音频和多模态领域。使用此技能可加载模型、执行推理并在自定义数据上微调。

## 安装

安装transformers及核心依赖：

```bash
uv pip install torch transformers datasets evaluate accelerate
```

视觉任务需额外安装：
```bash
uv pip install timm pillow
```

音频任务需额外安装：
```bash
uv pip install librosa soundfile
```

## 认证

Hugging Face Hub上的多数模型需要认证。设置访问权限：

```python
from huggingface_hub import login
login()  # 按提示输入令牌
```

或设置环境变量：
```bash
export HUGGINGFACE_TOKEN="your_token_here"
```

获取令牌：https://huggingface.co/settings/tokens

## 快速开始

使用Pipeline API无需手动配置即可快速推理：

```python
from transformers import pipeline

# 文本生成
generator = pipeline("text-generation", model="gpt2")
result = generator("AI的未来是", max_length=50)

# 文本分类
classifier = pipeline("text-classification")
result = classifier("这部电影太精彩了！")

# 问答系统
qa = pipeline("question-answering")
result = qa(question="什么是AI？", context="AI即人工智能...")
```

## 核心功能

### 1. 快速推理管道

为多种任务提供简单优化的推理方案。支持文本生成、分类、命名实体识别、问答、摘要、翻译、图像分类、目标检测、音频分类等。

**适用场景**：快速原型设计、简单推理任务、无需自定义预处理。

完整任务覆盖及优化详见 `references/pipelines.md`。

### 2. 模型加载与管理

通过细粒度配置控制加载预训练模型，包括设备分配和精度设置。

**适用场景**：自定义模型初始化、高级设备管理、模型检查。

加载模式及最佳实践详见 `references/models.md`。

### 3. 文本生成

使用多种解码策略（贪婪搜索、束搜索、采样）和控制参数（温度、top-k、top-p）生成文本。

**适用场景**：创意文本生成、代码生成、对话式AI、文本补全。

生成策略及参数详解见 `references/generation.md`。

### 4. 训练与微调

通过Trainer API在自定义数据集上微调预训练模型，支持自动混合精度、分布式训练和日志记录。

**适用场景**：任务特定模型适配、领域适应、提升模型性能。

训练流程及最佳实践详见 `references/training.md`。

### 5. 分词处理

将文本转换为词元及词元ID，支持填充、截断和特殊词元处理。

**适用场景**：自定义预处理流程、理解模型输入、批处理。

分词细节详见 `references/tokenizers.md`。

## 常用模式

### 模式1：简单推理
基础任务直接使用管道：
```python
pipe = pipeline("任务名称", model="模型ID")
output = pipe(输入数据)
```

### 模式2：自定义模型调用
需精细控制时单独加载模型和分词器：
```python
from transformers import AutoModelForCausalLM, AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("模型ID")
model = AutoModelForCausalLM.from_pretrained("模型ID", device_map="auto")

inputs = tokenizer("文本", return_tensors="pt")
outputs = model.generate(**inputs, max_new_tokens=100)
result = tokenizer.decode(outputs[0])
```

### 模式3：模型微调
任务适配使用Trainer：
```python
from transformers import Trainer, TrainingArguments

training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=3,
    per_device_train_batch_size=8,
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
)

trainer.train()
```

## 参考文档

各组件详细说明：
- **管道系统**：`references/pipelines.md` - 支持任务及优化方案
- **模型管理**：`references/models.md` - 加载/保存及配置
- **文本生成**：`references/generation.md` - 生成策略与参数
- **训练流程**：`references/training.md` - Trainer API微调
- **分词处理**：`references/tokenizers.md` - 分词与预处理
