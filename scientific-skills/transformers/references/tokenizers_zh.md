# 分词器

## 概述

分词器将文本转换为模型可处理的数值表示（标记）。它们处理特殊标记、填充、截断和注意力掩码。

## 加载分词器

### AutoTokenizer

自动加载适合模型的分词器：

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
```

从本地路径加载：
```python
tokenizer = AutoTokenizer.from_pretrained("./local/tokenizer/path")
```

## 基础分词

### 编码文本

```python
# 简单编码
text = "Hello, how are you?"
tokens = tokenizer.encode(text)
print(tokens)  # [101, 7592, 1010, 2129, 2024, 2017, 1029, 102]

# 文本分词
tokens = tokenizer.tokenize(text)
print(tokens)  # ['hello', ',', 'how', 'are', 'you', '?']
```

### 解码标记

```python
token_ids = [101, 7592, 1010, 2129, 2024, 2017, 1029, 102]
text = tokenizer.decode(token_ids)
print(text)  # "hello, how are you?"

# 跳过特殊标记
text = tokenizer.decode(token_ids, skip_special_tokens=True)
print(text)  # "hello, how are you?"
```

## `__call__` 方法

主要分词接口：

```python
# 单文本
inputs = tokenizer("Hello, how are you?")

# 返回包含 input_ids 和 attention_mask 的字典
print(inputs)
# {
#   'input_ids': [101, 7592, 1010, 2129, 2024, 2017, 1029, 102],
#   'attention_mask': [1, 1, 1, 1, 1, 1, 1, 1]
# }
```

多文本处理：
```python
texts = ["Hello", "How are you?"]
inputs = tokenizer(texts, padding=True, truncation=True)
```

## 关键参数

### 返回张量

**return_tensors**: 输出格式 ("pt", "tf", "np")
```python
# PyTorch 张量
inputs = tokenizer("text", return_tensors="pt")

# TensorFlow 张量
inputs = tokenizer("text", return_tensors="tf")

# NumPy 数组
inputs = tokenizer("text", return_tensors="np")
```

### 填充

**padding**: 将序列填充至相同长度
```python
# 按批次中最长序列填充
inputs = tokenizer(texts, padding=True)

# 填充至指定长度
inputs = tokenizer(texts, padding="max_length", max_length=128)

# 不填充
inputs = tokenizer(texts, padding=False)
```

**pad_to_multiple_of**: 填充至指定值的倍数
```python
inputs = tokenizer(texts, padding=True, pad_to_multiple_of=8)
```

### 截断

**truncation**: 限制序列长度
```python
# 截断至 max_length
inputs = tokenizer(text, truncation=True, max_length=512)

# 截断句对中的首句
inputs = tokenizer(text1, text2, truncation="only_first")

# 截断第二句
inputs = tokenizer(text1, text2, truncation="only_second")

# 优先截断较长句（句对默认）
inputs = tokenizer(text1, text2, truncation="longest_first", max_length=512)
```

### 最大长度

**max_length**: 最大序列长度
```python
inputs = tokenizer(text, max_length=512, truncation=True)
```

### 附加输出

**return_attention_mask**: 包含注意力掩码（默认 True）
```python
inputs = tokenizer(text, return_attention_mask=True)
```

**return_token_type_ids**: 句对的分段ID
```python
inputs = tokenizer(text1, text2, return_token_type_ids=True)
```

**return_offsets_mapping**: 字符位置映射（仅限快速分词器）
```python
inputs = tokenizer(text, return_offsets_mapping=True)
```

**return_length**: 包含序列长度
```python
inputs = tokenizer(texts, padding=True, return_length=True)
```

## 特殊标记

### 预定义特殊标记

访问特殊标记：
```python
print(tokenizer.cls_token)      # [CLS] 或 <s>
print(tokenizer.sep_token)      # [SEP] 或 </s>
print(tokenizer.pad_token)      # [PAD]
print(tokenizer.unk_token)      # [UNK]
print(tokenizer.mask_token)     # [MASK]
print(tokenizer.eos_token)      # 序列结束符
print(tokenizer.bos_token)      # 序列起始符

# 获取ID
print(tokenizer.cls_token_id)
print(tokenizer.sep_token_id)
```

### 添加特殊标记

手动控制：
```python
# 自动添加特殊标记（默认 True）
inputs = tokenizer(text, add_special_tokens=True)

# 跳过特殊标记
inputs = tokenizer(text, add_special_tokens=False)
```

### 自定义特殊标记

```python
special_tokens_dict = {
    "additional_special_tokens": ["<CUSTOM>", "<SPECIAL>"]
}

num_added = tokenizer.add_special_tokens(special_tokens_dict)
print(f"添加了 {num_added} 个标记")

# 添加标记后调整模型嵌入层
model.resize_token_embeddings(len(tokenizer))
```

## 句对处理

处理文本对：

```python
text1 = "What is the capital of France?"
text2 = "Paris is the capital of France."

# 自动处理分隔
inputs = tokenizer(text1, text2, padding=True, truncation=True)

# 生成格式: [CLS] text1 [SEP] text2 [SEP]
```

## 批量编码

处理多文本：

```python
texts = ["First text", "Second text", "Third text"]

# 基础批量编码
batch = tokenizer(texts, padding=True, truncation=True, return_tensors="pt")

# 访问单个编码
for i in range(len(texts)):
    input_ids = batch["input_ids"][i]
    attention_mask = batch["attention_mask"][i]
```

## 快速分词器

使用基于 Rust 的分词器提升速度：

```python
from transformers import AutoTokenizer

# 自动加载快速版本（若可用）
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

# 检查是否快速
print(tokenizer.is_fast)  # True

# 强制使用快速分词器
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased", use_fast=True)

# 强制使用慢速（Python）分词器
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased", use_fast=False)
```

### 快速分词器特性

**偏移映射**（字符位置）：
```python
inputs = tokenizer("Hello world", return_offsets_mapping=True)
print(inputs["offset_mapping"])
# [(0, 0), (0, 5), (6, 11), (0, 0)]  # [CLS], "Hello", "world", [SEP]
```

**标记到单词映射**：
```python
encoding = tokenizer("Hello world")
word_ids = encoding.word_ids()
print(word_ids)  # [None, 0, 1, None]  # [CLS]=None, "Hello"=0, "world"=1, [SEP]=None
```

## 保存分词器

保存至本地：
```python
tokenizer.save_pretrained("./my_tokenizer")
```

推送至 Hub：
```python
tokenizer.push_to_hub("username/my-tokenizer")
```

## 高级用法

### 词汇表

访问词汇表：
```python
vocab = tokenizer.get_vocab()
vocab_size = len(vocab)

# 通过ID获取标记
token = tokenizer.convert_ids_to_tokens(100)

# 通过标记获取ID
token_id = tokenizer.convert_tokens_to_ids("hello")
```

### 编码详情

获取详细编码信息：

```python
encoding = tokenizer("Hello world", return_tensors="pt")

# 原始方法仍可用
tokens = encoding.tokens()
word_ids = encoding.word_ids()
sequence_ids = encoding.sequence_ids()
```

### 自定义预处理

子类化实现自定义行为：

```python
class CustomTokenizer(AutoTokenizer):
    def __call__(self, text, **kwargs):
        # 自定义预处理
        text = text.lower().strip()
        return super().__call__(text, **kwargs)
```

## 对话模板

用于对话模型：

```python
messages = [
    {"role": "system", "content": "You are helpful."},
    {"role": "user", "content": "Hello!"},
    {"role": "assistant", "content": "Hi there!"},
    {"role": "user", "content": "How are you?"}
]

# 应用对话模板
text = tokenizer.apply_chat_template(messages, tokenize=False)
print(text)

# 直接分词
inputs = tokenizer.apply_chat_template(messages, tokenize=True, return_tensors="pt")
```

## 常用模式

### 模式1：简单文本分类

```python
texts = ["I love this!", "I hate this!"]
labels = [1, 0]

inputs = tokenizer(
    texts,
    padding=True,
    truncation=True,
    max_length=512,
    return_tensors="pt"
)

# 配合模型使用
outputs = model(**inputs, labels=torch.tensor(labels))
```

### 模式2：问答系统

```python
question = "What is the capital?"
context = "Paris is the capital of France."

inputs = tokenizer(
    question,
    context,
    padding=True,
    truncation=True,
    max_length=384,
    return_tensors="pt"
)
```

### 模式3：文本生成

```python
prompt = "Once upon a time"

inputs = tokenizer(prompt, return_tensors="pt")

# 生成文本
outputs = model.generate(
    inputs["input_ids"],
    max_new_tokens=50,
    pad_token_id=tokenizer.eos_token_id
)

# 解码
text = tokenizer.decode(outputs[0], skip_special_tokens=True)
```

### 模式4：数据集分词

```python
def tokenize_function(examples):
    return tokenizer(
        examples["text"],
        padding="max_length",
        truncation=True,
        max_length=512
    )

# 应用于数据集
tokenized_dataset = dataset.map(tokenize_function, batched=True)
```

## 最佳实践

1. **始终指定 return_tensors**：用于模型输入
2. **使用填充和截断**：用于批处理
3. **显式设置 max_length**：避免内存问题
4. **优先使用快速分词器**：提升处理速度
5. **处理填充标记**：生成任务中若未设置则用 eos_token
6. **添加特殊标记**：除非特殊需求，否则保持启用
7. **调整嵌入层大小**：添加自定义标记后执行
8. **解码时跳过特殊标记**：获得更清晰输出
9. **使用批处理**：提升数据集处理效率
10. **与模型同时保存分词器**：确保兼容性

## 常见问题

**未设置填充标记：**
```python
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token
```

**序列过长：**
```python
# 启用截断
inputs = tokenizer(text, truncation=True, max_length=512)
```

**词汇表不匹配：**
```python
# 始终从相同检查点加载分词器和模型
tokenizer = AutoTokenizer.from_pretrained("model-id")
model = AutoModel.from_pretrained("model-id")
```

**注意力掩码问题：**
```python
# 确保传递 attention_mask
outputs = model(
    input_ids=inputs["input_ids"],
    attention_mask=inputs["attention_mask"]
)
```
