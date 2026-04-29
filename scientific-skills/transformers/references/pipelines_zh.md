# 管道 API 参考

## 概述

管道提供了使用预训练模型进行推理的最简方式。它封装了分词、模型加载和后处理过程，为数十种任务提供统一接口。

## 基本用法

通过指定任务创建管道：

```python
from transformers import pipeline

# 自动选择任务的默认模型
pipe = pipeline("text-classification")
result = pipe("这太棒了！")
```

或指定具体模型：

```python
pipe = pipeline("text-classification", model="distilbert-base-uncased-finetuned-sst-2-english")
```

## 支持的任务

### 自然语言处理

**text-generation**：生成文本续写
```python
generator = pipeline("text-generation", model="gpt2")
output = generator("很久很久以前", max_length=50, num_return_sequences=2)
```

**text-classification**：文本分类
```python
classifier = pipeline("text-classification")
result = classifier("我爱这个产品！")  # 返回标签和置信度
```

**token-classification**：标记级标注（命名实体识别，词性标注）
```python
ner = pipeline("token-classification", model="dslim/bert-base-NER")
entities = ner("Hugging Face 总部位于纽约市")
```

**question-answering**：从上下文中提取答案
```python
qa = pipeline("question-answering")
result = qa(question="首都是什么？", context="巴黎是法国的首都。")
```

**fill-mask**：预测掩码标记
```python
unmasker = pipeline("fill-mask", model="bert-base-uncased")
result = unmasker("巴黎是法国的[MASK]")
```

**summarization**：长文本摘要
```python
summarizer = pipeline("summarization", model="facebook/bart-large-cnn")
summary = summarizer("长文章内容...", max_length=130, min_length=30)
```

**translation**：语言翻译
```python
translator = pipeline("translation_en_to_fr", model="Helsinki-NLP/opus-mt-en-fr")
result = translator("你好吗？")
```

**zero-shot-classification**：无需训练数据的分类
```python
classifier = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")
result = classifier(
    "这是一门 Python 编程课程",
    candidate_labels=["教育", "政治", "商业"]
)
```

**sentiment-analysis**：专注于情感分析的文本分类别名
```python
sentiment = pipeline("sentiment-analysis")
result = sentiment("这个产品超出预期！")
```

### 计算机视觉

**image-classification**：图像分类
```python
classifier = pipeline("image-classification", model="google/vit-base-patch16-224")
result = classifier("图像路径.jpg")
# 或使用 PIL 图像/URL
from PIL import Image
result = classifier(Image.open("image.jpg"))
```

**object-detection**：目标检测
```python
detector = pipeline("object-detection", model="facebook/detr-resnet-50")
results = detector("image.jpg")  # 返回边界框和标签
```

**image-segmentation**：图像分割
```python
segmenter = pipeline("image-segmentation", model="facebook/detr-resnet-50-panoptic")
segments = segmenter("image.jpg")
```

**depth-estimation**：深度图估计
```python
depth = pipeline("depth-estimation", model="Intel/dpt-large")
result = depth("image.jpg")
```

**zero-shot-image-classification**：无需训练的图片分类
```python
classifier = pipeline("zero-shot-image-classification", model="openai/clip-vit-base-patch32")
result = classifier("image.jpg", candidate_labels=["猫", "狗", "鸟"])
```

### 音频处理

**automatic-speech-recognition**：语音转文本
```python
asr = pipeline("automatic-speech-recognition", model="openai/whisper-base")
text = asr("audio.mp3")
```

**audio-classification**：音频分类
```python
classifier = pipeline("audio-classification", model="MIT/ast-finetuned-audioset-10-10-0.4593")
result = classifier("audio.wav")
```

**text-to-speech**：文本转语音（需特定模型）
```python
tts = pipeline("text-to-speech", model="microsoft/speecht5_tts")
audio = tts("测试文本")
```

### 多模态

**visual-question-answering**：图像问答
```python
vqa = pipeline("visual-question-answering", model="dandelin/vilt-b32-finetuned-vqa")
result = vqa(image="image.jpg", question="汽车是什么颜色？")
```

**document-question-answering**：文档问答
```python
doc_qa = pipeline("document-question-answering", model="impira/layoutlm-document-qa")
result = doc_qa(image="document.png", question="发票号码是多少？")
```

**image-to-text**：图像描述生成
```python
captioner = pipeline("image-to-text", model="Salesforce/blip-image-captioning-base")
caption = captioner("image.jpg")
```

## 管道参数

### 通用参数

**model**：模型标识符或路径
```python
pipe = pipeline("task", model="model-id")
```

**device**：GPU 设备索引（-1 表示 CPU，0+ 表示 GPU）
```python
pipe = pipeline("task", device=0)  # 使用第一块 GPU
```

**device_map**：大模型自动设备分配
```python
pipe = pipeline("task", model="large-model", device_map="auto")
```

**dtype**：模型精度（减少内存占用）
```python
import torch
pipe = pipeline("task", torch_dtype=torch.float16)
```

**batch_size**：批量处理输入
```python
pipe = pipeline("task", batch_size=8)
results = pipe(["文本1", "文本2", "文本3"])
```

**framework**：选择 PyTorch 或 TensorFlow
```python
pipe = pipeline("task", framework="pt")  # 或 "tf"
```

## 批量处理

高效处理多个输入：

```python
classifier = pipeline("text-classification")
texts = ["好产品！", "糟糕体验", "一般般"]
results = classifier(texts)
```

大型数据集使用生成器或 KeyDataset：

```python
from transformers.pipelines.pt_utils import KeyDataset
import datasets

dataset = datasets.load_dataset("dataset-name", split="test")
pipe = pipeline("task", device=0)

for output in pipe(KeyDataset(dataset, "text")):
    print(output)
```

## 性能优化

### GPU 加速

始终指定设备以使用 GPU：
```python
pipe = pipeline("task", device=0)
```

### 混合精度

在支持 GPU 上使用 float16 实现 2 倍加速：
```python
import torch
pipe = pipeline("task", torch_dtype=torch.float16, device=0)
```

### 批处理指南

- **CPU**：通常跳过批处理
- **变长序列 GPU**：可能降低效率
- **等长序列 GPU**：显著加速
- **实时应用**：跳过批处理（降低延迟）

```python
# 高吞吐量场景适用
pipe = pipeline("task", batch_size=32, device=0)
results = pipe(文本列表)
```

### 流式输出

文本生成时实时输出标记：

```python
from transformers import TextStreamer

generator = pipeline("text-generation", model="gpt2", streamer=TextStreamer())
generator("AI 的未来", max_length=100)
```

## 自定义管道配置

单独指定分词器和模型：

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification

tokenizer = AutoTokenizer.from_pretrained("model-id")
model = AutoModelForSequenceClassification.from_pretrained("model-id")
pipe = pipeline("text-classification", model=model, tokenizer=tokenizer)
```

使用自定义管道类：

```python
from transformers import TextClassificationPipeline

class CustomPipeline(TextClassificationPipeline):
    def postprocess(self, model_outputs, **kwargs):
        # 自定义后处理
        return super().postprocess(model_outputs, **kwargs)

pipe = pipeline("text-classification", model="model-id", pipeline_class=CustomPipeline)
```

## 输入格式

管道支持多种输入类型：

**文本任务**：字符串或字符串列表
```python
pipe("单个文本")
pipe(["文本1", "文本2"])
```

**图像任务**：URL、文件路径、PIL 图像或 numpy 数组
```python
pipe("https://example.com/image.jpg")
pipe("本地路径/image.png")
pipe(PIL.Image.open("image.jpg"))
pipe(numpy_array)
```

**音频任务**：文件路径、numpy 数组或原始波形
```python
pipe("audio.mp3")
pipe(audio_array)
```

## 错误处理

处理常见问题：

```python
try:
    result = pipe(input_data)
except Exception as e:
    if "CUDA out of memory" in str(e):
        # 减小批大小或使用 CPU
        pipe = pipeline("task", device=-1)
    elif "does not appear to have a file named" in str(e):
        # 模型未找到
        print("检查模型标识符")
    else:
        raise
```

## 最佳实践

1. **原型设计使用管道**：无需样板代码快速迭代
2. **显式指定模型**：默认模型可能变更
3. **启用可用 GPU**：显著加速
4. **高吞吐量使用批处理**：处理大量输入时
5. **考虑内存占用**：大批次使用 float16 或小模型
6. **本地缓存模型**：避免重复下载
