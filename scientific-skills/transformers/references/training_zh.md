# 训练与微调

## 概述

使用Trainer API在自定义数据集上微调预训练模型。Trainer负责处理训练循环、梯度累积、混合精度训练、日志记录和检查点保存。

## 基础微调流程

### 步骤1：加载与预处理数据

```python
from datasets import load_dataset

# 加载数据集
dataset = load_dataset("yelp_review_full")
train_dataset = dataset["train"]
eval_dataset = dataset["test"]

# 分词处理
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

def tokenize_function(examples):
    return tokenizer(
        examples["text"],
        padding="max_length",
        truncation=True,
        max_length=512
    )

train_dataset = train_dataset.map(tokenize_function, batched=True)
eval_dataset = eval_dataset.map(tokenize_function, batched=True)
```

### 步骤2：加载模型

```python
from transformers import AutoModelForSequenceClassification

model = AutoModelForSequenceClassification.from_pretrained(
    "bert-base-uncased",
    num_labels=5  # 类别数量
)
```

### 步骤3：定义评估指标

```python
import evaluate
import numpy as np

metric = evaluate.load("accuracy")

def compute_metrics(eval_pred):
    logits, labels = eval_pred
    predictions = np.argmax(logits, axis=-1)
    return metric.compute(predictions=predictions, references=labels)
```

### 步骤4：配置训练参数

```python
from transformers import TrainingArguments

training_args = TrainingArguments(
    output_dir="./results",
    eval_strategy="epoch",
    save_strategy="epoch",
    learning_rate=2e-5,
    per_device_train_batch_size=8,
    per_device_eval_batch_size=8,
    num_train_epochs=3,
    weight_decay=0.01,
    logging_dir="./logs",
    logging_steps=10,
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
)
```

### 步骤5：创建Trainer并训练

```python
from transformers import Trainer

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset,
    compute_metrics=compute_metrics,
)

# 开始训练
trainer.train()

# 评估模型
results = trainer.evaluate()
print(results)
```

### 步骤6：保存模型

```python
trainer.save_model("./fine_tuned_model")
tokenizer.save_pretrained("./fine_tuned_model")

# 或推送至Hugging Face Hub
trainer.push_to_hub("username/my-finetuned-model")
```

## TrainingArguments参数详解

### 核心参数

**output_dir**: 检查点和日志的保存目录
```python
output_dir="./results"
```

**num_train_epochs**: 训练轮数
```python
num_train_epochs=3
```

**per_device_train_batch_size**: 单设备批处理大小
```python
per_device_train_batch_size=8
```

**learning_rate**: 优化器学习率
```python
learning_rate=2e-5  # BERT类模型常用值
learning_rate=5e-5  # 小型模型常用值
```

**weight_decay**: L2正则化强度
```python
weight_decay=0.01
```

### 评估与保存

**eval_strategy**: 评估时机（"no", "steps", "epoch"）
```python
eval_strategy="epoch"  # 每轮结束后评估
eval_strategy="steps"  # 按指定步数评估
```

**save_strategy**: 检查点保存策略
```python
save_strategy="epoch"
save_strategy="steps"
save_steps=500
```

**load_best_model_at_end**: 训练结束后加载最佳模型
```python
load_best_model_at_end=True
metric_for_best_model="accuracy"  # 模型比较指标
```

### 优化配置

**gradient_accumulation_steps**: 梯度累积步数
```python
gradient_accumulation_steps=4  # 有效批大小 = 批大小 * 4
```

**fp16**: 启用混合精度（NVIDIA GPU）
```python
fp16=True
```

**bf16**: 启用bfloat16（新型GPU）
```python
bf16=True
```

**gradient_checkpointing**: 内存优化技术
```python
gradient_checkpointing=True  # 速度较慢但节省内存
```

**optim**: 优化器选择
```python
optim="adamw_torch"  # 默认优化器
optim="adamw_8bit"    # 8位Adam（需bitsandbytes）
optim="adafactor"     # 内存高效替代方案
```

### 学习率调度

**lr_scheduler_type**: 学习率调度策略
```python
lr_scheduler_type="linear"       # 线性衰减
lr_scheduler_type="cosine"       # 余弦退火
lr_scheduler_type="constant"     # 恒定学习率
lr_scheduler_type="constant_with_warmup"
```

**warmup_steps** 或 **warmup_ratio**: 预热步数/比例
```python
warmup_steps=500
# 或
warmup_ratio=0.1  # 总步数的10%
```

### 日志记录

**logging_dir**: TensorBoard日志目录
```python
logging_dir="./logs"
```

**logging_steps**: 日志记录频率
```python
logging_steps=10
```

**report_to**: 日志集成平台
```python
report_to=["tensorboard"]
report_to=["wandb"]
report_to=["tensorboard", "wandb"]
```

### 分布式训练

**ddp_backend**: 分布式后端
```python
ddp_backend="nccl"  # 多GPU配置
```

**deepspeed**: DeepSpeed配置文件
```python
deepspeed="ds_config.json"
```

## 数据整理器

处理动态填充和特殊预处理：

### DataCollatorWithPadding

按批次最长序列填充：
```python
from transformers import DataCollatorWithPadding

data_collator = DataCollatorWithPadding(tokenizer=tokenizer)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    data_collator=data_collator,
)
```

### DataCollatorForLanguageModeling

用于掩码语言建模：
```python
from transformers import DataCollatorForLanguageModeling

data_collator = DataCollatorForLanguageModeling(
    tokenizer=tokenizer,
    mlm=True,
    mlm_probability=0.15
)
```

### DataCollatorForSeq2Seq

用于序列到序列任务：
```python
from transformers import DataCollatorForSeq2Seq

data_collator = DataCollatorForSeq2Seq(
    tokenizer=tokenizer,
    model=model,
    padding=True
)
```

## 自定义训练

### 自定义Trainer

重写方法实现定制行为：

```python
from transformers import Trainer

class CustomTrainer(Trainer):
    def compute_loss(self, model, inputs, return_outputs=False):
        labels = inputs.pop("labels")
        outputs = model(**inputs)
        logits = outputs.logits

        # 自定义损失计算
        loss_fct = torch.nn.CrossEntropyLoss(weight=class_weights)
        loss = loss_fct(logits.view(-1, self.model.config.num_labels), labels.view(-1))

        return (loss, outputs) if return_outputs else loss
```

### 自定义回调

监控和控制训练过程：

```python
from transformers import TrainerCallback

class CustomCallback(TrainerCallback):
    def on_epoch_end(self, args, state, control, **kwargs):
        print(f"Epoch {state.epoch} 已完成")
        # 自定义逻辑
        return control

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    callbacks=[CustomCallback],
)
```

## 高级训练技术

### 参数高效微调（PEFT）

使用LoRA实现高效微调：
```python
from peft import LoraConfig, get_peft_model

lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["query", "value"],
    lora_dropout=0.05,
    bias="none",
    task_type="SEQ_CLS"
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()  # 显示可训练参数数量

# 使用Trainer正常训练
trainer = Trainer(model=model, args=training_args, ...)
trainer.train()
```

### 梯度检查点

牺牲速度换取内存优化：
```python
model.gradient_checkpointing_enable()

training_args = TrainingArguments(
    gradient_checkpointing=True,
    ...
)
```

### 混合精度训练
```python
training_args = TrainingArguments(
    fp16=True,  # 支持Tensor Core的NVIDIA GPU
    # 或
    bf16=True,  # 新型GPU（A100, H100）
    ...
)
```

### DeepSpeed集成

适用于超大规模模型：
```python
# ds_config.json
{
  "train_batch_size": 16,
  "gradient_accumulation_steps": 1,
  "optimizer": {
    "type": "AdamW",
    "params": {
      "lr": 2e-5
    }
  },
  "fp16": {
    "enabled": true
  },
  "zero_optimization": {
    "stage": 2
  }
}
```

```python
training_args = TrainingArguments(
    deepspeed="ds_config.json",
    ...
)
```

## 训练技巧

### 超参数调优

常用初始值：
- **学习率**：BERT类模型建议2e-5至5e-5，小型模型建议1e-4至1e-3
- **批大小**：根据GPU内存选择8-32
- **训练轮数**：微调建议2-4轮，领域适应需更多轮次
- **预热步数**：占总步数10%

使用Optuna进行超参数搜索：
```python
def model_init():
    return AutoModelForSequenceClassification.from_pretrained(
        "bert-base-uncased",
        num_labels=5
    )

def optuna_hp_space(trial):
    return {
        "learning_rate": trial.suggest_float("learning_rate", 1e-5, 5e-5, log=True),
        "per_device_train_batch_size": trial.suggest_categorical("per_device_train_batch_size", [8, 16, 32]),
        "num_train_epochs": trial.suggest_int("num_train_epochs", 2, 5),
    }

trainer = Trainer(model_init=model_init, args=training_args, ...)
best_trial = trainer.hyperparameter_search(
    direction="maximize",
    backend="optuna",
    hp_space=optuna_hp_space,
    n_trials=10,
)
```

### 训练监控

使用TensorBoard：
```bash
tensorboard --logdir ./logs
```

或Weights & Biases：
```python
import wandb
wandb.init(project="my-project")

training_args = TrainingArguments(
    report_to=["wandb"],
    ...
)
```

### 恢复训练

从检查点恢复：
```python
trainer.train(resume_from_checkpoint="./results/checkpoint-1000")
```

## 常见问题

**CUDA内存不足：**
- 减小批大小
- 启用梯度检查点
- 使用梯度累积
- 采用8位优化器

**过拟合：**
- 增加权重衰减
- 添加Dropout层
- 使用早停策略
- 减小模型规模或训练轮数

**训练速度慢：**
- 增大批大小
- 启用混合精度（fp16/bf16）
- 使用多GPU并行
- 优化数据加载流程

## 最佳实践

1. **从小规模开始**：先在数据子集测试
2. **持续评估**：监控验证集指标
3. **保存检查点**：启用save_strategy
4. **详细记录日志**：使用TensorBoard或W&B
5. **尝试不同学习率**：从2e-5开始
6. **使用预热策略**：提升训练稳定性
7. **启用混合精度**：加速训练过程
8. **考虑PEFT技术**：资源有限时处理大模型
