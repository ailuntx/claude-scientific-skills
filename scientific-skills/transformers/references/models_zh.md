# 模型加载与管理

## 概述

transformers库提供灵活的模型加载功能，支持自动架构检测、设备管理和配置控制。

## 加载模型

### AutoModel类

使用AutoModel类实现自动架构选择：

```python
from transformers import AutoModel, AutoModelForSequenceClassification, AutoModelForCausalLM

# 基础模型（无任务头）
model = AutoModel.from_pretrained("bert-base-uncased")

# 序列分类
model = AutoModelForSequenceClassification.from_pretrained("distilbert-base-uncased")

# 因果语言建模（GPT风格）
model = AutoModelForCausalLM.from_pretrained("gpt2")

# 掩码语言建模（BERT风格）
from transformers import AutoModelForMaskedLM
model = AutoModelForMaskedLM.from_pretrained("bert-base-uncased")

# 序列到序列（T5风格）
from transformers import AutoModelForSeq2SeqLM
model = AutoModelForSeq2SeqLM.from_pretrained("t5-small")
```

### 常用AutoModel类

**NLP任务：**
- `AutoModelForSequenceClassification`: 文本分类，情感分析
- `AutoModelForTokenClassification`: 命名实体识别，词性标注
- `AutoModelForQuestionAnswering`: 抽取式问答
- `AutoModelForCausalLM`: 文本生成（GPT, Llama）
- `AutoModelForMaskedLM`: 掩码语言建模（BERT）
- `AutoModelForSeq2SeqLM`: 翻译，摘要（T5, BART）

**视觉任务：**
- `AutoModelForImageClassification`: 图像分类
- `AutoModelForObjectDetection`: 目标检测
- `AutoModelForImageSegmentation`: 图像分割

**音频任务：**
- `AutoModelForAudioClassification`: 音频分类
- `AutoModelForSpeechSeq2Seq`: 语音识别

**多模态：**
- `AutoModelForVision2Seq`: 图像描述，视觉问答

## 加载参数

### 基础参数

**pretrained_model_name_or_path**: 模型标识符或本地路径
```python
model = AutoModel.from_pretrained("bert-base-uncased")  # 从Hub加载
model = AutoModel.from_pretrained("./local/model/path")  # 从磁盘加载
```

**num_labels**: 分类任务的输出标签数量
```python
model = AutoModelForSequenceClassification.from_pretrained(
    "bert-base-uncased",
    num_labels=3
)
```

**cache_dir**: 自定义缓存位置
```python
model = AutoModel.from_pretrained("model-id", cache_dir="./my_cache")
```

### 设备管理

**device_map**: 大模型自动设备分配
```python
# 自动分配到GPU和CPU
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    device_map="auto"
)

# 顺序分配
model = AutoModelForCausalLM.from_pretrained(
    "model-id",
    device_map="sequential"
)

# 自定义设备映射
device_map = {
    "transformer.layers.0": 0,      # GPU 0
    "transformer.layers.1": 1,      # GPU 1
    "transformer.layers.2": "cpu",  # CPU
}
model = AutoModel.from_pretrained("model-id", device_map=device_map)
```

手动设备分配：
```python
import torch
model = AutoModel.from_pretrained("model-id")
model.to("cuda:0")  # 移动到GPU 0
model.to(torch.device("cuda" if torch.cuda.is_available() else "cpu"))
```

### 精度控制

**torch_dtype**: 设置模型精度
```python
import torch

# Float16（半精度）
model = AutoModel.from_pretrained("model-id", torch_dtype=torch.float16)

# BFloat16（比float16范围更大）
model = AutoModel.from_pretrained("model-id", torch_dtype=torch.bfloat16)

# Auto（使用原始精度）
model = AutoModel.from_pretrained("model-id", torch_dtype="auto")
```

### 注意力实现

**attn_implementation**: 选择注意力机制
```python
# Scaled Dot Product Attention（PyTorch 2.0+，最快）
model = AutoModel.from_pretrained("model-id", attn_implementation="sdpa")

# Flash Attention 2（需安装flash-attn包）
model = AutoModel.from_pretrained("model-id", attn_implementation="flash_attention_2")

# Eager（默认，兼容性最佳）
model = AutoModel.from_pretrained("model-id", attn_implementation="eager")
```

### 内存优化

**low_cpu_mem_usage**: 减少加载时的CPU内存占用
```python
model = AutoModelForCausalLM.from_pretrained(
    "large-model-id",
    low_cpu_mem_usage=True,
    device_map="auto"
)
```

**load_in_8bit**: 8位量化（需bitsandbytes）
```python
model = AutoModelForCausalLM.from_pretrained(
    "model-id",
    load_in_8bit=True,
    device_map="auto"
)
```

**load_in_4bit**: 4位量化
```python
from transformers import BitsAndBytesConfig

quantization_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype=torch.float16
)

model = AutoModelForCausalLM.from_pretrained(
    "model-id",
    quantization_config=quantization_config,
    device_map="auto"
)
```

## 模型配置

### 加载自定义配置

```python
from transformers import AutoConfig, AutoModel

# 加载并修改配置
config = AutoConfig.from_pretrained("bert-base-uncased")
config.hidden_dropout_prob = 0.2
config.attention_probs_dropout_prob = 0.2

# 使用自定义配置初始化模型
model = AutoModel.from_pretrained("bert-base-uncased", config=config)
```

### 仅从配置初始化

```python
config = AutoConfig.from_pretrained("gpt2")
model = AutoModelForCausalLM.from_config(config)  # 随机权重
```

## 模型模式

### 训练模式与评估模式

模型默认加载为评估模式：

```python
model = AutoModel.from_pretrained("model-id")
print(model.training)  # False

# 切换到训练模式
model.train()

# 切换回评估模式
model.eval()
```

评估模式会禁用dropout并使用批归一化统计量。

## 保存模型

### 本地保存

```python
model.save_pretrained("./my_model")
```

将创建：
- `config.json`: 模型配置
- `pytorch_model.bin` 或 `model.safetensors`: 模型权重

### 保存到Hugging Face Hub

```python
model.push_to_hub("username/model-name")

# 带自定义提交信息
model.push_to_hub("username/model-name", commit_message="更新模型")

# 私有仓库
model.push_to_hub("username/model-name", private=True)
```

## 模型检查

### 参数量统计

```python
# 总参数量
total_params = model.num_parameters()

# 仅可训练参数
trainable_params = model.num_parameters(only_trainable=True)

print(f"总计: {total_params:,}")
print(f"可训练: {trainable_params:,}")
```

### 内存占用

```python
memory_bytes = model.get_memory_footprint()
memory_mb = memory_bytes / 1024**2
print(f"内存: {memory_mb:.2f} MB")
```

### 模型架构

```python
print(model)  # 打印完整架构

# 访问特定组件
print(model.config)
print(model.base_model)
```

## 前向传播

基础推理：

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("model-id")
model = AutoModelForSequenceClassification.from_pretrained("model-id")

inputs = tokenizer("示例文本", return_tensors="pt")
outputs = model(**inputs)

logits = outputs.logits
predictions = logits.argmax(dim=-1)
```

## 模型格式

### SafeTensors vs PyTorch

SafeTensors更快更安全：

```python
# 保存为safetensors（推荐）
model.save_pretrained("./model", safe_serialization=True)

# 自动加载任意格式
model = AutoModel.from_pretrained("./model")
```

### ONNX导出

导出用于优化推理：

```python
from transformers.onnx import export

# 导出到ONNX
export(
    tokenizer=tokenizer,
    model=model,
    config=config,
    output=Path("model.onnx")
)
```

## 最佳实践

1. **使用AutoModel类**：自动检测架构
2. **显式指定dtype**：控制精度和内存
3. **使用device_map="auto"**：适用于大模型
4. **启用low_cpu_mem_usage**：加载大模型时
5. **使用safetensors格式**：更快更安全的序列化
6. **检查model.training**：确保任务模式正确
7. **考虑量化**：资源受限设备部署
8. **本地缓存模型**：设置TRANSFORMERS_CACHE环境变量

## 常见问题

**CUDA内存不足：**
```python
# 使用更低精度
model = AutoModel.from_pretrained("model-id", torch_dtype=torch.float16)

# 或使用量化
model = AutoModel.from_pretrained("model-id", load_in_8bit=True)

# 或使用CPU
model = AutoModel.from_pretrained("model-id", device_map="cpu")
```

**加载缓慢：**
```python
# 启用低CPU内存模式
model = AutoModel.from_pretrained("model-id", low_cpu_mem_usage=True)
```

**找不到模型：**
```python
# 在hub.co验证模型ID
# 私有模型需检查认证
from huggingface_hub import login
login()
```
