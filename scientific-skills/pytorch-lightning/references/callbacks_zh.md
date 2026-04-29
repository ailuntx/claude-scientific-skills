# 回调函数 - 全面指南

## 概述

回调函数允许在不污染 LightningModule 研究代码的前提下，向训练过程添加任意自包含程序。它们能在训练生命周期的特定钩子中执行自定义逻辑。

## 架构设计

Lightning 将训练逻辑组织为三个组件：
- **Trainer** - 工程基础设施
- **LightningModule** - 研究代码
- **Callbacks** - 非核心功能（监控、检查点保存、自定义行为）

## 创建自定义回调

基础结构：

```python
from lightning.pytorch.callbacks import Callback

class MyCustomCallback(Callback):
    def on_train_start(self, trainer, pl_module):
        print("训练开始！")

    def on_train_end(self, trainer, pl_module):
        print("训练完成！")

# 与Trainer配合使用
trainer = L.Trainer(callbacks=[MyCustomCallback()])
```

## 内置回调函数

### ModelCheckpoint

基于监控指标保存模型。

**关键参数：**
- `dirpath` - 检查点保存目录
- `filename` - 检查点文件名模式
- `monitor` - 监控指标
- `mode` - 监控指标模式："min" 或 "max"
- `save_top_k` - 保留的最佳模型数量
- `save_last` - 保存最后周期检查点
- `every_n_epochs` - 每N个周期保存
- `save_on_train_epoch_end` - 在训练周期结束时保存（而非验证结束时）

**示例：**
```python
from lightning.pytorch.callbacks import ModelCheckpoint

# 基于验证损失保存前三名模型
checkpoint_callback = ModelCheckpoint(
    dirpath="checkpoints/",
    filename="model-{epoch:02d}-{val_loss:.2f}",
    monitor="val_loss",
    mode="min",
    save_top_k=3,
    save_last=True
)

# 每10个周期保存
checkpoint_callback = ModelCheckpoint(
    dirpath="checkpoints/",
    filename="model-{epoch:02d}",
    every_n_epochs=10,
    save_top_k=-1  # 保存全部
)

# 基于准确率保存最佳模型
checkpoint_callback = ModelCheckpoint(
    dirpath="checkpoints/",
    filename="best-model",
    monitor="val_acc",
    mode="max",
    save_top_k=1
)

trainer = L.Trainer(callbacks=[checkpoint_callback])
```

**访问保存的检查点：**
```python
# 获取最佳模型路径
best_model_path = checkpoint_callback.best_model_path

# 获取最后检查点路径
last_checkpoint = checkpoint_callback.last_model_path

# 获取所有检查点路径
all_checkpoints = checkpoint_callback.best_k_models
```

### EarlyStopping

当监控指标停止改善时终止训练。

**关键参数：**
- `monitor` - 监控指标
- `patience` - 指标无改善的容忍周期数
- `mode` - "min" 或 "max" 指标模式
- `min_delta` - 视为改善的最小变化量
- `verbose` - 打印消息
- `strict` - 未找到监控指标时崩溃

**示例：**
```python
from lightning.pytorch.callbacks import EarlyStopping

# 当验证损失停止改善时终止
early_stop = EarlyStopping(
    monitor="val_loss",
    patience=10,
    mode="min",
    verbose=True
)

# 当准确率停滞时终止
early_stop = EarlyStopping(
    monitor="val_acc",
    patience=5,
    mode="max",
    min_delta=0.001  # 至少需改善0.001
)

trainer = L.Trainer(callbacks=[early_stop])
```

### LearningRateMonitor

跟踪调度器的学习率变化。

**关键参数：**
- `logging_interval` - 记录时机："step" 或 "epoch"
- `log_momentum` - 同时记录动量值

**示例：**
```python
from lightning.pytorch.callbacks import LearningRateMonitor

lr_monitor = LearningRateMonitor(logging_interval="step")
trainer = L.Trainer(callbacks=[lr_monitor])

# 自动记录为 "lr-{optimizer_name}"
```

### DeviceStatsMonitor

记录设备性能指标（GPU/CPU/TPU）。

**关键参数：**
- `cpu_stats` - 记录CPU状态

**示例：**
```python
from lightning.pytorch.callbacks import DeviceStatsMonitor

device_stats = DeviceStatsMonitor(cpu_stats=True)
trainer = L.Trainer(callbacks=[device_stats])

# 记录：gpu_utilization, gpu_memory_usage等
```

### ModelSummary / RichModelSummary

显示模型架构和参数量。

**示例：**
```python
from lightning.pytorch.callbacks import ModelSummary, RichModelSummary

# 基础摘要
summary = ModelSummary(max_depth=2)

# 富文本格式摘要（更美观）
rich_summary = RichModelSummary(max_depth=3)

trainer = L.Trainer(callbacks=[rich_summary])
```

### Timer

跟踪并限制训练时长。

**关键参数：**
- `duration` - 最大训练时长（timedelta或字典）
- `interval` - 检查间隔："step", "epoch" 或 "batch"

**示例：**
```python
from lightning.pytorch.callbacks import Timer
from datetime import timedelta

# 限制训练时长为1小时
timer = Timer(duration=timedelta(hours=1))

# 使用字典形式
timer = Timer(duration={"hours": 23, "minutes": 30})

trainer = L.Trainer(callbacks=[timer])
```

### BatchSizeFinder

自动寻找最优批次大小。

**示例：**
```python
from lightning.pytorch.callbacks import BatchSizeFinder

batch_finder = BatchSizeFinder(mode="power", steps_per_trial=3)

trainer = L.Trainer(callbacks=[batch_finder])
trainer.fit(model, datamodule=dm)

# 自动设置最优批次大小
```

### GradientAccumulationScheduler

动态调度梯度累积步数。

**示例：**
```python
from lightning.pytorch.callbacks import GradientAccumulationScheduler

# 前5周期累积4批次，之后累积2批次
accumulator = GradientAccumulationScheduler(scheduling={0: 4, 5: 2})

trainer = L.Trainer(callbacks=[accumulator])
```

### StochasticWeightAveraging (SWA)

应用随机权重平均提升泛化能力。

**示例：**
```python
from lightning.pytorch.callbacks import StochasticWeightAveraging

swa = StochasticWeightAveraging(swa_lrs=1e-2, swa_epoch_start=0.8)

trainer = L.Trainer(callbacks=[swa])
```

## 自定义回调示例

### 简单日志回调

```python
class MetricsLogger(Callback):
    def __init__(self):
        self.metrics = []

    def on_validation_end(self, trainer, pl_module):
        # 访问记录指标
        metrics = trainer.callback_metrics
        self.metrics.append(dict(metrics))
        print(f"验证指标: {metrics}")
```

### 梯度监控回调

```python
class GradientMonitor(Callback):
    def on_after_backward(self, trainer, pl_module):
        # 记录梯度范数
        for name, param in pl_module.named_parameters():
            if param.grad is not None:
                grad_norm = param.grad.norm().item()
                pl_module.log(f"grad_norm/{name}", grad_norm)
```

### 自定义检查点回调

```python
class CustomCheckpoint(Callback):
    def __init__(self, save_dir):
        self.save_dir = save_dir

    def on_train_epoch_end(self, trainer, pl_module):
        epoch = trainer.current_epoch
        if epoch % 5 == 0:  # 每5周期保存
            filepath = f"{self.save_dir}/custom-{epoch}.ckpt"
            trainer.save_checkpoint(filepath)
            print(f"已保存检查点: {filepath}")
```

### 模型冻结回调

```python
class FreezeUnfreeze(Callback):
    def __init__(self, freeze_until_epoch=10):
        self.freeze_until_epoch = freeze_until_epoch

    def on_train_epoch_start(self, trainer, pl_module):
        epoch = trainer.current_epoch

        if epoch < self.freeze_until_epoch:
            # 冻结主干网络
            for param in pl_module.backbone.parameters():
                param.requires_grad = False
        else:
            # 解冻主干网络
            for param in pl_module.backbone.parameters():
                param.requires_grad = True
```

### 学习率查找回调

```python
class LRFinder(Callback):
    def __init__(self, min_lr=1e-5, max_lr=1e-1, num_steps=100):
        self.min_lr = min_lr
        self.max_lr = max_lr
        self.num_steps = num_steps
        self.lrs = []
        self.losses = []

    def on_train_batch_end(self, trainer, pl_module, outputs, batch, batch_idx):
        if batch_idx >= self.num_steps:
            trainer.should_stop = True
            return

        # 指数学习率调度
        lr = self.min_lr * (self.max_lr / self.min_lr) ** (batch_idx / self.num_steps)
        optimizer = trainer.optimizers[0]
        for param_group in optimizer.param_groups:
            param_group['lr'] = lr

        self.lrs.append(lr)
        self.losses.append(outputs['loss'].item())

    def on_train_end(self, trainer, pl_module):
        # 绘制学习率-损失曲线
        import matplotlib.pyplot as plt
        plt.plot(self.lrs, self.losses)
        plt.xscale('log')
        plt.xlabel('学习率')
        plt.ylabel('损失值')
        plt.savefig('lr_finder.png')
```

### 预测保存回调

```python
class PredictionSaver(Callback):
    def __init__(self, save_path):
        self.save_path = save_path
        self.predictions = []

    def on_predict_batch_end(self, trainer, pl_module, outputs, batch, batch_idx):
        self.predictions.append(outputs)

    def on_predict_end(self, trainer, pl_module):
        # 保存所有预测结果
        torch.save(self.predictions, self.save_path)
        print(f"预测结果已保存至 {self.save_path}")
```

## 可用钩子函数

### 初始化和清理
- `setup(trainer, pl_module, stage)` - 在fit/test/predict开始时调用
- `teardown(trainer, pl_module, stage)` - 在fit/test/predict结束时调用

### 训练生命周期
- `on_fit_start(trainer, pl_module)` - fit开始时调用
- `on_fit_end(trainer, pl_module)` - fit结束时调用
- `on_train_start(trainer, pl_module)` - 训练开始时调用
- `on_train_end(trainer, pl_module)` - 训练结束时调用

### 周期边界
- `on_train_epoch_start(trainer, pl_module)` - 训练周期开始时调用
- `on_train_epoch_end(trainer, pl_module)` - 训练周期结束时调用
- `on_validation_epoch_start(trainer, pl_module)` - 验证开始时调用
- `on_validation_epoch_end(trainer, pl_module)` - 验证结束时调用
- `on_test_epoch_start(trainer, pl_module)` - 测试开始时调用
- `on_test_epoch_end(trainer, pl_module)` - 测试结束时调用

### 批次边界
- `on_train_batch_start(trainer, pl_module, batch, batch_idx)` - 训练批次前调用
- `on_train_batch_end(trainer, pl_module, outputs, batch, batch_idx)` - 训练批次后调用
- `on_validation_batch_start(trainer, pl_module, batch, batch_idx)` - 验证批次前调用
- `on_validation_batch_end(trainer, pl_module, outputs, batch, batch_idx)` - 验证批次后调用

### 梯度事件
- `on_before_backward(trainer, pl_module, loss)` - loss.backward()前调用
- `on_after_backward(trainer, pl_module)` - loss.backward()后调用
- `on_before_optimizer_step(trainer, pl_module, optimizer)` - optimizer.step()前调用

### 检查点事件
- `on_save_checkpoint(trainer, pl_module, checkpoint)` - 保存检查点时调用
- `on_load_checkpoint(trainer, pl_module, checkpoint)` - 加载检查点时调用

### 异常处理
- `on_exception(trainer, pl_module, exception)` - 发生异常时调用

## 状态管理

需要跨检查点持久化的回调：

```python
class StatefulCallback(Callback):
    def __init__(self):
        self.counter = 0

    def on_train_batch_end(self, trainer, pl_module, outputs, batch, batch_idx):
        self.counter += 1

    def state_dict(self):
        return {"counter": self.counter}

    def load_state_dict(self, state_dict):
        self.counter = state_dict["counter"]

    @property
    def state_key(self):
        # 回调的唯一标识符
        return "my_stateful_callback"
```

## 最佳实践

### 1. 保持回调隔离性
每个回调应自包含且独立：

```python
# 良好：自包含
class MyCallback(Callback):
    def __init__(self):
        self.data = []

    def on_train_batch_end(self, trainer, pl_module, outputs, batch, batch_idx):
        self.data.append(outputs['loss'].item())

# 不良：依赖外部状态
global_data = []

class BadCallback(Callback):
    def on_train_batch_end(self, trainer, pl_module, outputs, batch, batch_idx):
        global_data.append(outputs['loss'].item())  # 外部依赖
```

### 2. 避免回调间依赖
回调不应依赖其他回调：

```python
# 不良：回调B依赖回调A
class CallbackA(Callback):
    def __init__(self):
        self.value = 0

class CallbackB(Callback):
    def __init__(self, callback_a):
        self.callback_a = callback_a  # 紧耦合

# 良好：独立回调
class CallbackA(Callback):
    def __init__(self):
        self.value = 0

class CallbackB(Callback):
    def on_train_batch_end(self, trainer, pl_module, outputs, batch, batch_idx):
        # 通过trainer状态访问
        value = trainer.callback_metrics.get('metric')
```

### 3. 切勿手动调用回调方法
由Lightning自动调用回调：

```python
# 不良：手动调用
callback = MyCallback()
callback.on_train_start(trainer, model)  # 禁止操作

# 良好：由Trainer处理
trainer = L.Trainer(callbacks=[MyCallback()])
```

### 4. 设计适应任意执行顺序
回调可能以任意顺序执行，避免依赖特定顺序：

```python
# 良好：顺序无关
class GoodCallback(Callback):
    def on_train_epoch_end(self, trainer, pl_module):
        # 使用trainer状态而非其他回调
        metrics = trainer.callback_metrics
        self.log_metrics(metrics)
```

### 5. 将非核心逻辑放入回调
核心研究代码保留在LightningModule中，辅助功能使用回调：

```python
# 良好分离
class MyModel(L.LightningModule):
    # 核心研究逻辑
    def training_step(self, batch, batch_idx):
        return loss

# 非核心监控放入回调
class MonitorCallback(Callback):
    def on_validation_end(self, trainer, pl_module):
        # 监控逻辑
        pass
```

## 常用模式

### 组合多个回调

```python
from lightning.pytorch.callbacks import (
    ModelCheckpoint,
    EarlyStopping,
    LearningRateMonitor,
    DeviceStatsMonitor
)

callbacks = [
    ModelCheckpoint(monitor="val_loss", mode="min", save_top_k=3),
    EarlyStopping(monitor="val_loss", patience=10, mode="min"),
    LearningRateMonitor(logging_interval="step"),
    DeviceStatsMonitor()
]

trainer = L.Trainer(callbacks=call
