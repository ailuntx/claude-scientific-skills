# Trainer - 全面指南

## 概述

Trainer 在将 PyTorch 代码组织到 LightningModule 后，自动处理训练工作流。它自动管理循环细节、设备分配、回调函数、梯度操作、检查点保存和分布式训练。

## 核心功能

Trainer 负责：
- 自动启用/禁用梯度计算
- 运行训练、验证和测试数据加载器
- 在适当时机调用回调函数
- 将批次数据分配到正确设备
- 协调分布式训练
- 进度条显示与日志记录
- 检查点保存与早停机制

## 主要方法

### `fit(model, train_dataloaders=None, val_dataloaders=None, datamodule=None)`
执行完整训练流程（含可选验证）

**参数：**
- `model` - 待训练的 LightningModule
- `train_dataloaders` - 训练数据加载器
- `val_dataloaders` - 可选验证数据加载器
- `datamodule` - 可选 LightningDataModule（替代数据加载器）

**示例：**
```python
# 使用数据加载器
trainer = L.Trainer(max_epochs=10)
trainer.fit(model, train_loader, val_loader)

# 使用数据模块
trainer.fit(model, datamodule=dm)

# 从检查点继续训练
trainer.fit(model, train_loader, ckpt_path="checkpoint.ckpt")
```

### `validate(model=None, dataloaders=None, datamodule=None)`
不进行训练，仅运行验证循环

**示例：**
```python
trainer = L.Trainer()
trainer.validate(model, val_loader)
```

### `test(model=None, dataloaders=None, datamodule=None)`
运行测试循环（仅限发布结果前使用）

**示例：**
```python
trainer = L.Trainer()
trainer.test(model, test_loader)
```

### `predict(model=None, dataloaders=None, datamodule=None)`
对数据进行推理并返回预测结果

**示例：**
```python
trainer = L.Trainer()
predictions = trainer.predict(model, predict_loader)
```

## 关键参数

### 训练时长

#### `max_epochs` (整数)
最大训练轮数。默认值：1000

```python
trainer = L.Trainer(max_epochs=100)
```

#### `min_epochs` (整数)
最小训练轮数。默认值：无

```python
trainer = L.Trainer(min_epochs=10, max_epochs=100)
```

#### `max_steps` (整数)
最大优化器步数（覆盖 max_epochs）。默认值：-1（无限制）

```python
trainer = L.Trainer(max_steps=10000)
```

#### `max_time` (字符串或字典)
最大训练时长（适用于限时集群）

```python
# 字符串格式
trainer = L.Trainer(max_time="00:12:00:00")  # 12小时

# 字典格式
trainer = L.Trainer(max_time={"days": 1, "hours": 6})
```

### 硬件配置

#### `accelerator` (字符串或 Accelerator)
指定硬件："cpu"、"gpu"、"tpu"、"ipu"、"hpu"、"mps" 或 "auto"。默认值："auto"

```python
trainer = L.Trainer(accelerator="gpu")
trainer = L.Trainer(accelerator="auto")  # 自动检测可用硬件
```

#### `devices` (整数/列表/字符串)
使用的设备索引或数量

```python
# 使用2个GPU
trainer = L.Trainer(devices=2, accelerator="gpu")

# 使用指定GPU
trainer = L.Trainer(devices=[0, 2], accelerator="gpu")

# 使用所有可用设备
trainer = L.Trainer(devices="auto", accelerator="gpu")

# 使用4核CPU
trainer = L.Trainer(devices=4, accelerator="cpu")
```

#### `strategy` (字符串或 Strategy)
分布式训练策略："ddp"、"ddp_spawn"、"fsdp"、"deepspeed" 等。默认值："auto"

```python
# 数据分布式并行
trainer = L.Trainer(strategy="ddp", accelerator="gpu", devices=4)

# 全分片数据并行
trainer = L.Trainer(strategy="fsdp", accelerator="gpu", devices=4)

# DeepSpeed
trainer = L.Trainer(strategy="deepspeed_stage_2", accelerator="gpu", devices=4)
```

#### `precision` (字符串或整数)
浮点精度："32-true"、"16-mixed"、"bf16-mixed"、"64-true" 等

```python
# 混合精度 (FP16)
trainer = L.Trainer(precision="16-mixed")

# BFloat16混合精度
trainer = L.Trainer(precision="bf16-mixed")

# 全精度
trainer = L.Trainer(precision="32-true")
```

### 优化配置

#### `gradient_clip_val` (浮点数)
梯度裁剪阈值。默认值：无

```python
# 按范数裁剪梯度
trainer = L.Trainer(gradient_clip_val=0.5)
```

#### `gradient_clip_algorithm` (字符串)
梯度裁剪算法："norm" 或 "value"。默认值："norm"

```python
trainer = L.Trainer(gradient_clip_val=0.5, gradient_clip_algorithm="norm")
```

#### `accumulate_grad_batches` (整数或字典)
执行优化器步骤前累积梯度的批次数

```python
# 累积4个批次
trainer = L.Trainer(accumulate_grad_batches=4)

# 分阶段累积
trainer = L.Trainer(accumulate_grad_batches={0: 4, 5: 2, 10: 1})
```

### 验证配置

#### `check_val_every_n_epoch` (整数)
每N轮执行验证。默认值：1

```python
trainer = L.Trainer(check_val_every_n_epoch=10)
```

#### `val_check_interval` (整数或浮点数)
训练轮次内的验证频率

```python
# 每完成25%训练轮次验证
trainer = L.Trainer(val_check_interval=0.25)

# 每100个训练批次验证
trainer = L.Trainer(val_check_interval=100)
```

#### `limit_val_batches` (整数或浮点数)
限制验证批次数量

```python
# 仅使用10%验证数据
trainer = L.Trainer(limit_val_batches=0.1)

# 仅使用50个验证批次
trainer = L.Trainer(limit_val_batches=50)

# 禁用验证
trainer = L.Trainer(limit_val_batches=0)
```

#### `num_sanity_val_steps` (整数)
训练开始前的验证批次数量。默认值：2

```python
# 跳过完整性检查
trainer = L.Trainer(num_sanity_val_steps=0)

# 执行5步完整性验证
trainer = L.Trainer(num_sanity_val_steps=5)
```

### 日志与进度

#### `logger` (Logger/列表/布尔值)
实验跟踪使用的日志记录器

```python
from lightning.pytorch import loggers as pl_loggers

# TensorBoard日志
tb_logger = pl_loggers.TensorBoardLogger("logs/")
trainer = L.Trainer(logger=tb_logger)

# 多日志记录器
wandb_logger = pl_loggers.WandbLogger(project="my-project")
trainer = L.Trainer(logger=[tb_logger, wandb_logger])

# 禁用日志
trainer = L.Trainer(logger=False)
```

#### `log_every_n_steps` (整数)
训练步骤内的日志频率。默认值：50

```python
trainer = L.Trainer(log_every_n_steps=10)
```

#### `enable_progress_bar` (布尔值)
显示进度条。默认值：True

```python
trainer = L.Trainer(enable_progress_bar=False)
```

### 回调函数

#### `callbacks` (列表)
训练期间使用的回调函数列表

```python
from lightning.pytorch.callbacks import ModelCheckpoint, EarlyStopping

checkpoint_callback = ModelCheckpoint(
    monitor="val_loss",
    save_top_k=3,
    mode="min"
)

early_stop_callback = EarlyStopping(
    monitor="val_loss",
    patience=5,
    mode="min"
)

trainer = L.Trainer(callbacks=[checkpoint_callback, early_stop_callback])
```

### 检查点

#### `default_root_dir` (字符串)
日志和检查点的默认目录。默认值：当前工作目录

```python
trainer = L.Trainer(default_root_dir="./experiments/")
```

#### `enable_checkpointing` (布尔值)
启用自动检查点保存。默认值：True

```python
trainer = L.Trainer(enable_checkpointing=True)
```

### 调试

#### `fast_dev_run` (布尔值或整数)
运行单批次（或N批次）训练/验证/测试进行调试

```python
# 运行1批次训练/验证/测试
trainer = L.Trainer(fast_dev_run=True)

# 运行5批次训练/验证/测试
trainer = L.Trainer(fast_dev_run=5)
```

#### `limit_train_batches` (整数或浮点数)
限制训练批次数量

```python
# 仅使用25%训练数据
trainer = L.Trainer(limit_train_batches=0.25)

# 仅使用100个训练批次
trainer = L.Trainer(limit_train_batches=100)
```

#### `limit_test_batches` (整数或浮点数)
限制测试批次数量

```python
trainer = L.Trainer(limit_test_batches=0.5)
```

#### `overfit_batches` (整数或浮点数)
在数据子集上过拟合（用于调试）

```python
# 在10个批次上过拟合
trainer = L.Trainer(overfit_batches=10)

# 在1%数据上过拟合
trainer = L.Trainer(overfit_batches=0.01)
```

#### `detect_anomaly` (布尔值)
启用PyTorch异常检测（调试NaN值）。默认值：False

```python
trainer = L.Trainer(detect_anomaly=True)
```

### 可复现性

#### `deterministic` (布尔值或字符串)
控制确定性行为。默认值：False

```python
import lightning as L

# 设置全局随机种子
L.seed_everything(42, workers=True)

# 完全确定性（可能影响性能）
trainer = L.Trainer(deterministic=True)

# 检测到非确定性操作时警告
trainer = L.Trainer(deterministic="warn")
```

#### `benchmark` (布尔值)
启用cudnn基准测试提升性能。默认值：False

```python
trainer = L.Trainer(benchmark=True)
```

### 其他配置

#### `enable_model_summary` (布尔值)
训练前打印模型摘要。默认值：True

```python
trainer = L.Trainer(enable_model_summary=False)
```

#### `inference_mode` (布尔值)
验证/测试时使用 torch.inference_mode() 替代 torch.no_grad()。默认值：True

```python
trainer = L.Trainer(inference_mode=True)
```

#### `profiler` (字符串或 Profiler)
性能分析器："simple"、"advanced" 或自定义分析器

```python
# 简单分析器
trainer = L.Trainer(profiler="simple")

# 高级分析器
trainer = L.Trainer(profiler="advanced")
```

## 常用配置

### 基础训练
```python
trainer = L.Trainer(
    max_epochs=100,
    accelerator="auto",
    devices="auto"
)
trainer.fit(model, train_loader, val_loader)
```

### 多GPU训练
```python
trainer = L.Trainer(
    max_epochs=100,
    accelerator="gpu",
    devices=4,
    strategy="ddp",
    precision="16-mixed"
)
trainer.fit(model, datamodule=dm)
```

### 生产级训练（含检查点）
```python
from lightning.pytorch.callbacks import ModelCheckpoint, EarlyStopping, LearningRateMonitor

checkpoint_callback = ModelCheckpoint(
    dirpath="checkpoints/",
    filename="{epoch}-{val_loss:.2f}",
    monitor="val_loss",
    mode="min",
    save_top_k=3,
    save_last=True
)

early_stop = EarlyStopping(
    monitor="val_loss",
    patience=10,
    mode="min"
)

lr_monitor = LearningRateMonitor(logging_interval="step")

trainer = L.Trainer(
    max_epochs=100,
    accelerator="gpu",
    devices=2,
    strategy="ddp",
    precision="16-mixed",
    callbacks=[checkpoint_callback, early_stop, lr_monitor],
    log_every_n_steps=10,
    gradient_clip_val=1.0
)

trainer.fit(model, datamodule=dm)
```

### 调试配置
```python
trainer = L.Trainer(
    fast_dev_run=True,          # 运行单批次
    accelerator="cpu",
    enable_progress_bar=True,
    log_every_n_steps=1,
    detect_anomaly=True
)
trainer.fit(model, train_loader, val_loader)
```

### 研究配置（可复现性）
```python
import lightning as L

L.seed_everything(42, workers=True)

trainer = L.Trainer(
    max_epochs=100,
    accelerator="gpu",
    devices=1,
    deterministic=True,
    benchmark=False,
    precision="32-true"
)
trainer.fit(model, datamodule=dm)
```

### 限时训练（集群环境）
```python
trainer = L.Trainer(
    max_time={"hours": 23, "minutes": 30},  # SLURM时间限制
    max_epochs=1000,
    callbacks=[ModelCheckpoint(save_last=True)]
)
trainer.fit(model, datamodule=dm)

# 从检查点恢复
trainer.fit(model, datamodule=dm, ckpt_path="last.ckpt")
```

### 大模型训练（FSDP）
```python
from lightning.pytorch.strategies import FSDPStrategy

trainer = L.Trainer(
    max_epochs=100,
    accelerator="gpu",
    devices=8,
    strategy=FSDPStrategy(
        activation_checkpointing_policy={nn.TransformerEncoderLayer},
        cpu_offload=False
    ),
    precision="bf16-mixed",
    accumulate_grad_batches=4
)
trainer.fit(model, datamodule=dm)
```

## 恢复训练

### 从检查点恢复
```python
# 从特定检查点恢复
trainer.fit(model, datamodule=dm, ckpt_path="epoch=10-val_loss=0.23.ckpt")

# 从最后检查点恢复
trainer.fit(model, datamodule=dm, ckpt_path="last.ckpt")
```

### 查找最后检查点
```python
from lightning.pytorch.callbacks import ModelCheckpoint

checkpoint_callback = ModelCheckpoint(save_last=True)
trainer = L.Trainer(callbacks=[checkpoint_callback])
trainer.fit(model, datamodule=dm)

# 获取最后检查点路径
last_checkpoint = checkpoint_callback.last_model_path
```

## 在LightningModule中访问Trainer

通过 `self.trainer` 访问 Trainer 实例：

```python
class MyModel(L.LightningModule):
    def training_step(self, batch, batch_idx):
        # 访问trainer属性
        current_epoch = self.trainer.current_epoch
        global_step = self.trainer.global_step
        max_epochs = self.trainer.max_epochs

        # 访问回调函数
        for callback in self.trainer.callbacks:
            if isinstance(callback, ModelCheckpoint):
                print(f"最佳模型: {callback.best_model_path}")

        # 访问日志记录器
        self.trainer.logger.log_metrics({"custom": value})
```

## Trainer 属性

| 属性 | 描述 |
|-----------|-------------|
| `trainer.current_epoch` | 当前轮次（0起始） |
| `trainer.global_step` | 总优化器步数 |
| `trainer.max_epochs` | 配置的最大轮次 |
| `trainer.max_steps` | 配置的最大步数 |
| `trainer.callbacks` | 回调函数列表 |
| `trainer.logger` | 日志记录器实例 |
| `trainer.strategy` | 训练策略 |
| `trainer.estimated_stepping_batches` | 训练预估总步数 |

## 最佳实践

### 1. 从快速开发运行

trainer = L.Trainer(callbacks=[LearningRateMonitor(logging_interval="step")])
```

### 6. 使用 DataModule 确保可复现性
将数据逻辑封装在 DataModule 中：

```python
# 比直接传递 DataLoaders 更好
trainer.fit(model, datamodule=dm)
```

### 7. 为研究设置确定性
确保发表成果的可复现性：

```python
L.seed_everything(42, workers=True)
trainer = L.Trainer(deterministic=True)
```
