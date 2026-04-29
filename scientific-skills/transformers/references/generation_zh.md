# 文本生成

## 概述

使用 `generate()` 方法通过语言模型生成文本。通过生成策略和参数控制输出质量与风格。

## 基础生成

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained("gpt2")
tokenizer = AutoTokenizer.from_pretrained("gpt2")

# 对输入进行分词
inputs = tokenizer("Once upon a time", return_tensors="pt")

# 生成文本
outputs = model.generate(**inputs, max_new_tokens=50)

# 解码输出
text = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(text)
```

## 生成策略

### 贪心解码

每步选择概率最高的词元（确定性输出）：

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=50,
    do_sample=False  # 贪心解码（默认）
)
```

**适用场景**：需要确定性的场景，如事实性文本、翻译任务。

### 随机采样

从概率分布中随机采样：

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=50,
    do_sample=True,
    temperature=0.7,
    top_k=50,
    top_p=0.95
)
```

**适用场景**：创意写作、多样化输出、开放式生成。

### 束搜索

并行探索多个假设路径：

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=50,
    num_beams=5,
    early_stopping=True
)
```

**适用场景**：质量要求高的场景，如翻译、摘要任务。

### 对比搜索

平衡质量与多样性：

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=50,
    penalty_alpha=0.6,
    top_k=4
)
```

**适用场景**：长文本生成，减少重复内容。

## 关键参数

### 长度控制

**max_new_tokens**：最大生成词元数
```python
max_new_tokens=100  # 最多生成100个新词元
```

**max_length**：总长度上限（输入+输出）
```python
max_length=512  # 序列总长度
```

**min_new_tokens**：最小生成词元数
```python
min_new_tokens=50  # 强制至少生成50个词元
```

**min_length**：最小总长度
```python
min_length=100
```

### 温度系数

控制随机性（仅采样时有效）：

```python
temperature=1.0   # 默认值，平衡状态
temperature=0.7   # 更聚焦，随机性更低
temperature=1.5   # 更具创意，随机性更高
```

温度值越低 → 确定性越强  
温度值越高 → 随机性越强

### Top-K采样

仅考虑概率最高的K个词元：

```python
do_sample=True
top_k=50  # 从前50个词元中采样
```

**常用值**：40-100（平衡输出），10-20（聚焦输出）。

### Top-P（核心）采样

考虑累积概率≥P的词元集合：

```python
do_sample=True
top_p=0.95  # 从累积概率95%的最小词元集合中采样
```

**常用值**：0.9-0.95（平衡输出），0.7-0.85（聚焦输出）。

### 重复惩罚

抑制重复内容：

```python
repetition_penalty=1.2  # 惩罚重复词元
```

**取值**：1.0=无惩罚，1.2-1.5=中等惩罚，2.0+=强惩罚。

### 束搜索参数

**num_beams**：束数量
```python
num_beams=5  # 保留5条假设路径
```

**early_stopping**：当num_beams条路径结束时停止
```python
early_stopping=True
```

**no_repeat_ngram_size**：禁止n-gram重复
```python
no_repeat_ngram_size=3  # 禁止任何3-gram重复
```

### 输出控制

**num_return_sequences**：生成多个输出序列
```python
outputs = model.generate(
    **inputs,
    max_new_tokens=50,
    num_beams=5,
    num_return_sequences=3  # 返回3个不同序列
)
```

**pad_token_id**：指定填充词元
```python
pad_token_id=tokenizer.eos_token_id
```

**eos_token_id**：在特定词元处停止生成
```python
eos_token_id=tokenizer.eos_token_id
```

## 高级功能

### 批量生成

多提示词批量生成：

```python
prompts = ["Hello, my name is", "Once upon a time"]
inputs = tokenizer(prompts, return_tensors="pt", padding=True)

outputs = model.generate(**inputs, max_new_tokens=50)

for i, output in enumerate(outputs):
    text = tokenizer.decode(output, skip_special_tokens=True)
    print(f"提示词 {i}: {text}\n")
```

### 流式生成

实时输出生成词元：

```python
from transformers import TextIteratorStreamer
from threading import Thread

streamer = TextIteratorStreamer(tokenizer, skip_special_tokens=True)

generation_kwargs = dict(
    inputs,
    streamer=streamer,
    max_new_tokens=100
)

thread = Thread(target=model.generate, kwargs=generation_kwargs)
thread.start()

for text in streamer:
    print(text, end="", flush=True)

thread.join()
```

### 约束生成

强制特定词元序列：

```python
# 强制生成以特定词元开头
force_words = ["Paris", "France"]
force_words_ids = [tokenizer.encode(word, add_special_tokens=False) for word in force_words]

outputs = model.generate(
    **inputs,
    force_words_ids=force_words_ids,
    num_beams=5
)
```

### 引导控制

**屏蔽不良词汇：**
```python
bad_words = ["offensive", "inappropriate"]
bad_words_ids = [tokenizer.encode(word, add_special_tokens=False) for word in bad_words]

outputs = model.generate(
    **inputs,
    bad_words_ids=bad_words_ids
)
```

### 生成配置

保存并复用生成参数：

```python
from transformers import GenerationConfig

# 创建配置
generation_config = GenerationConfig(
    max_new_tokens=100,
    temperature=0.7,
    top_k=50,
    top_p=0.95,
    do_sample=True
)

# 保存配置
generation_config.save_pretrained("./my_generation_config")

# 加载并使用
generation_config = GenerationConfig.from_pretrained("./my_generation_config")
outputs = model.generate(**inputs, generation_config=generation_config)
```

## 模型特定生成

### 对话模型

使用对话模板：

```python
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is the capital of France?"}
]

input_text = tokenizer.apply_chat_template(messages, tokenize=False)
inputs = tokenizer(input_text, return_tensors="pt")

outputs = model.generate(**inputs, max_new_tokens=100)
response = tokenizer.decode(outputs[0], skip_special_tokens=True)
```

### 编码器-解码器模型

适用于T5、BART等：

```python
from transformers import AutoModelForSeq2SeqLM, AutoTokenizer

model = AutoModelForSeq2SeqLM.from_pretrained("t5-small")
tokenizer = AutoTokenizer.from_pretrained("t5-small")

# T5使用任务前缀
input_text = "translate English to French: Hello, how are you?"
inputs = tokenizer(input_text, return_tensors="pt")

outputs = model.generate(**inputs, max_new_tokens=50)
translation = tokenizer.decode(outputs[0], skip_special_tokens=True)
```

## 性能优化

### 缓存机制

启用KV缓存加速生成：

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=100,
    use_cache=True  # 默认开启，加速生成
)
```

### 静态缓存

适用于固定序列长度：

```python
from transformers import StaticCache

cache = StaticCache(model.config, max_batch_size=1, max_cache_len=1024, device="cuda")

outputs = model.generate(
    **inputs,
    max_new_tokens=100,
    past_key_values=cache
)
```

### 注意力实现

使用Flash Attention加速：

```python
model = AutoModelForCausalLM.from_pretrained(
    "model-id",
    attn_implementation="flash_attention_2"
)
```

## 生成方案示例

### 创意写作

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=200,
    do_sample=True,
    temperature=0.8,
    top_k=50,
    top_p=0.95,
    repetition_penalty=1.2
)
```

### 事实性生成

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=100,
    do_sample=False,  # 贪心解码
    repetition_penalty=1.1
)
```

### 多样化输出

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=100,
    num_beams=5,
    num_return_sequences=5,
    temperature=1.5,
    do_sample=True
)
```

### 长文本生成

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=1000,
    penalty_alpha=0.6,  # 对比搜索
    top_k=4,
    repetition_penalty=1.2
)
```

### 翻译/摘要

```python
outputs = model.generate(
    **inputs,
    max_new_tokens=100,
    num_beams=5,
    early_stopping=True,
    no_repeat_ngram_size=3
)
```

## 常见问题

**输出重复：**
- 增加repetition_penalty（1.2-1.5）
- 使用no_repeat_ngram_size（2-3）
- 尝试对比搜索
- 降低温度值

**质量不佳：**
- 使用束搜索（num_beams=5）
- 降低温度值
- 调整top_k/top_p

**过于确定：**
- 启用采样（do_sample=True）
- 提高温度值（0.7-1.0）
- 调整top_k/top_p

**生成缓慢：**
- 减小批次大小
- 启用use_cache=True
- 使用Flash Attention
- 减少max_new_tokens

## 最佳实践

1. **从默认值开始**：根据输出结果逐步调整
2. **选择合适策略**：事实性内容用贪心，创意内容用采样
3. **设置max_new_tokens**：避免不必要长文本
4. **启用缓存**：加速连续生成
5. **调节温度值**：采样中最关键参数
6. **谨慎使用束搜索**：速度较慢但质量更高
7. **测试不同随机种子**：保证采样可复现性
8. **监控内存**：大束宽会显著增加内存占用
