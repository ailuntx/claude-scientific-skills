# LightningModule - 全面指南

## 概述

`LightningModule` 将 PyTorch 代码组织为六个逻辑部分，无需抽象。代码保持纯 PyTorch 特性，仅优化了组织结构。Trainer 负责设备管理、分布式采样和基础设施，同时保留完整的模型控制权。

## 核心结构

```python
import lightning as L
import torch
import torch.nn.functional as F

class MyModel(L.LightningModule):
    def __init__(self, learning_rate=0.001):
        super().__init__()
        self.save_hyperparameters()  # 保存初始化参数
        self.model = YourNeuralNetwork()

    def training_step(self, batch, batch_idx):
        x, y = batch
        logits = self.model(x)
        loss = F.cross_entropy(logits, y)
        self.log("train_loss", loss)
        return loss

    def validation_step(self, batch, batch_idx):
        x, y = batch
        logits = self.model(x)
        loss = F.cross_entropy(logits, y)
        acc = (logits.argmax(dim=1) == y).float().mean()
        self.log("val_loss", loss)
        self.log("val_acc", acc)

    def test_step(self, batch, batch_idx):
        x, y = batch
        logits = self.model(x)
        loss = F.cross_entropy(logits, y)
        acc = (logits.argmax(dim=1) == y).float().mean()
        self.log("test_loss", loss)
        self.log("test_acc", acc)

    def configure_optimizers(self):
        optimizer = torch.optim.Adam(self.parameters(), lr=self.hparams.learning_rate)
        scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(optimizer, mode='min')
        return {
            "optimizer": optimizer,
            "lr_scheduler": {
                "scheduler": scheduler,
                "monitor": "val_loss"
            }
        }
```

## 核心方法

### 训练流程方法

#### `training_step(batch, batch_idx)`
计算前向传播并返回损失值。在自动优化模式下，Lightning 自动处理反向传播和优化器更新。

**参数：**
- `batch` - 来自 DataLoader 的当前训练批次
- `batch_idx` - 当前批次索引

**返回：** 损失张量（标量）或包含 'loss' 键的字典

**示例：**
```python
def training_step(self, batch, batch_idx):
    x, y = batch
    y_hat = self.model(x)
    loss = F.mse_loss(y_hat, y)

    # 记录训练指标
    self.log("train_loss", loss, on_step=True, on_epoch=True, prog_bar=True)
    self.log("learning_rate", self.optimizers().param_groups[0]['lr'])

    return loss
```

#### `validation_step(batch, batch_idx)`
在验证数据上评估模型。自动禁用梯度并以评估模式运行。

**参数：**
- `batch` - 当前验证批次
- `batch_idx` - 当前批次索引

**返回：** 可选 - 损失值或指标字典

**示例：**
```python
def validation_step(self, batch, batch_idx):
    x, y = batch
    y_hat = self.model(x)
    loss = F.mse_loss(y_hat, y)

    # Lightning 自动聚合验证批次
    self.log("val_loss", loss, prog_bar=True)
    return loss
```

#### `test_step(batch, batch_idx)`
在测试数据上评估模型。仅在显式调用 `trainer.test()` 时运行。通常在训练完成后发布前使用。

**参数：**
- `batch` - 当前测试批次
- `batch_idx` - 当前批次索引

**返回：** 可选 - 损失值或指标字典

#### `predict_step(batch, batch_idx, dataloader_idx=0)`
执行数据推理。通过 `trainer.predict()` 调用。

**参数：**
- `batch` - 当前批次
- `batch_idx` - 当前批次索引
- `dataloader_idx` - 数据加载器索引（多加载器时）

**返回：** 预测结果（任意格式）

**示例：**
```python
def predict_step(self, batch, batch_idx):
    x, y = batch
    return self.model(x)  # 返回原始预测
```

### 配置方法

#### `configure_optimizers()`
返回优化器及可选的学习率调度器。

**返回格式：**

1. **单优化器：**
```python
def configure_optimizers(self):
    return torch.optim.Adam(self.parameters(), lr=0.001)
```

2. **优化器+调度器：**
```python
def configure_optimizers(self):
    optimizer = torch.optim.Adam(self.parameters(), lr=0.001)
    scheduler = torch.optim.lr_scheduler.StepLR(optimizer, step_size=10)
    return [optimizer], [scheduler]
```

3. **带监控的高级配置：**
```python
def configure_optimizers(self):
    optimizer = torch.optim.Adam(self.parameters(), lr=0.001)
    scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(optimizer)
    return {
        "optimizer": optimizer,
        "lr_scheduler": {
            "scheduler": scheduler,
            "monitor": "val_loss",  # 监控指标
            "interval": "epoch",     # 更新时机（epoch/step）
            "frequency": 1,          # 更新频率
            "strict": True           # 监控指标缺失时报错
        }
    }
```

4. **多优化器（用于GAN等）：**
```python
def configure_optimizers(self):
    opt_g = torch.optim.Adam(self.generator.parameters(), lr=0.0002)
    opt_d = torch.optim.Adam(self.discriminator.parameters(), lr=0.0002)
    return [opt_g, opt_d]
```

#### `forward(*args, **kwargs)`
标准 PyTorch 前向方法。用于推理或作为 training_step 的组成部分。

**示例：**
```python
def forward(self, x):
    return self.model(x)

def training_step(self, batch, batch_idx):
    x, y = batch
    y_hat = self(x)  # 调用 forward()
    return F.mse_loss(y_hat, y)
```

### 日志与指标

#### `log(name, value, **kwargs)`
记录指标并自动跨设备进行周期级聚合。

**关键参数：**
- `name` - 指标名称（字符串）
- `value` - 指标值（张量或数值）
- `on_step` - 当前步骤记录（默认：training_step 中为 True，其他为 False）
- `on_epoch` - 周期结束时记录（默认：training_step 中为 False，其他为 True）
- `prog_bar` - 在进度条显示（默认：False）
- `logger` - 发送到日志后端（默认：True）
- `reduce_fx` - 聚合函数："mean", "sum", "max", "min"（默认："mean"）
- `sync_dist` - 分布式训练中跨设备同步（默认：False）

**示例：**
```python
# 简单日志
self.log("train_loss", loss)

# 进度条显示
self.log("accuracy", acc, prog_bar=True)

# 按步骤和周期记录
self.log("loss", loss, on_step=True, on_epoch=True)

# 分布式训练自定义聚合
self.log("batch_size", batch.size(0), reduce_fx="sum", sync_dist=True)
```

#### `log_dict(dictionary, **kwargs)`
同时记录多个指标。

**示例：**
```python
metrics = {"train_loss": loss, "train_acc": acc, "learning_rate": lr}
self.log_dict(metrics, on_step=True, on_epoch=True)
```

#### `save_hyperparameters(*args, **kwargs)`
存储初始化参数以实现可复现性和检查点恢复。在 `__init__()` 中调用。

**示例：**
```python
def __init__(self, learning_rate, hidden_dim, dropout):
    super().__init__()
    self.save_hyperparameters()  # 保存所有初始化参数
    # 通过 self.hparams.learning_rate 等访问
```

## 关键属性

| 属性 | 描述 |
|----------|-------------|
| `self.current_epoch` | 当前周期数（0起始） |
| `self.global_step` | 跨所有周期的总优化步数 |
| `self.device` | 当前设备（cuda:0, cpu 等） |
| `self.global_rank` | 分布式训练进程等级（主进程为0） |
| `self.local_rank` | 当前节点GPU等级 |
| `self.hparams` | 保存的超参数（通过 save_hyperparameters） |
| `self.trainer` | 父 Trainer 实例引用 |
| `self.automatic_optimization` | 是否启用自动优化（默认：True） |

## 手动优化

高级场景（GAN、强化学习、多优化器）需禁用自动优化：

```python
class GANModel(L.LightningModule):
    def __init__(self):
        super().__init__()
        self.automatic_optimization = False
        self.generator = Generator()
        self.discriminator = Discriminator()

    def training_step(self, batch, batch_idx):
        opt_g, opt_d = self.optimizers()

        # 训练生成器
        opt_g.zero_grad()
        g_loss = self.compute_generator_loss(batch)
        self.manual_backward(g_loss)
        opt_g.step()

        # 训练判别器
        opt_d.zero_grad()
        d_loss = self.compute_discriminator_loss(batch)
        self.manual_backward(d_loss)
        opt_d.step()

        self.log_dict({"g_loss": g_loss, "d_loss": d_loss})

    def configure_optimizers(self):
        opt_g = torch.optim.Adam(self.generator.parameters(), lr=0.0002)
        opt_d = torch.optim.Adam(self.discriminator.parameters(), lr=0.0002)
        return [opt_g, opt_d]
```

## 重要生命周期钩子

### 初始化和清理

#### `setup(stage)`
在 fit/validate/test/predict 开始时调用。用于阶段特定初始化。

**参数：**
- `stage` - 'fit', 'validate', 'test' 或 'predict'

**示例：**
```python
def setup(self, stage):
    if stage == 'fit':
        # 初始化训练组件
        self.train_dataset = load_train_data()
    elif stage == 'test':
        # 初始化测试组件
        self.test_dataset = load_test_data()
```

#### `teardown(stage)`
在 fit/validate/test/predict 结束时调用。清理资源。

### 周期边界

#### `on_train_epoch_start()` / `on_train_epoch_end()`
每个训练周期开始/结束时调用。

**示例：**
```python
def on_train_epoch_end(self):
    # 计算周期级指标
    all_preds = torch.cat(self.training_step_outputs)
    epoch_metric = compute_custom_metric(all_preds)
    self.log("epoch_metric", epoch_metric)
    self.training_step_outputs.clear()  # 释放内存
```

#### `on_validation_epoch_start()` / `on_validation_epoch_end()`
验证周期开始/结束时调用。

#### `on_test_epoch_start()` / `on_test_epoch_end()`
测试周期开始/结束时调用。

### 梯度钩子

#### `on_before_backward(loss)`
在 loss.backward() 前调用。

#### `on_after_backward()`
在 loss.backward() 后但优化器更新前调用。

**示例 - 梯度检查：**
```python
def on_after_backward(self):
    # 记录梯度范数
    grad_norm = torch.nn.utils.clip_grad_norm_(self.parameters(), max_norm=1.0)
    self.log("grad_norm", grad_norm)
```

### 检查点钩子

#### `on_save_checkpoint(checkpoint)`
自定义检查点保存。添加额外状态。

**示例：**
```python
def on_save_checkpoint(self, checkpoint):
    checkpoint['custom_state'] = self.custom_data
```

#### `on_load_checkpoint(checkpoint)`
自定义检查点加载。恢复额外状态。

**示例：**
```python
def on_load_checkpoint(self, checkpoint):
    self.custom_data = checkpoint.get('custom_state', default_value)
```

## 最佳实践

### 1. 设备无关性
避免显式调用 `.cuda()` 或 `.cpu()`。Lightning 自动处理设备分配。

**错误示范：**
```python
x = x.cuda()
model = model.cuda()
```

**正确示范：**
```python
x = x.to(self.device)  # 在 LightningModule 内部
# 或交由 Lightning 自动处理
```

### 2. 分布式训练安全
勿手动创建 `DistributedSampler`。Lightning 自动处理。

**错误示范：**
```python
sampler = DistributedSampler(dataset)
DataLoader(dataset, sampler=sampler)
```

**正确示范：**
```python
DataLoader(dataset, shuffle=True)  # Lightning 自动转换为 DistributedSampler
```

### 3. 指标聚合
使用 `self.log()` 实现自动跨设备聚合，避免手动收集。

**错误示范：**
```python
self.validation_outputs.append(loss)

def on_validation_epoch_end(self):
    avg_loss = torch.stack(self.validation_outputs).mean()
```

**正确示范：**
```python
self.log("val_loss", loss)  # 自动聚合
```

### 4. 超参数追踪
始终使用 `self.save_hyperparameters()` 以便模型重载。

**示例：**
```python
def __init__(self, learning_rate, hidden_dim):
    super().__init__()
    self.save_hyperparameters()

# 后续：从检查点加载
model = MyModel.load_from_checkpoint("checkpoint.ckpt")
print(model.hparams.learning_rate)
```

### 5. 验证放置
在单设备上运行验证以确保样本精确评估一次。Lightning 通过策略配置自动处理。

## 从检查点加载

```python
# 加载含超参数的模型
model = MyModel.load_from_checkpoint("path/to/checkpoint.ckpt")

# 按需覆盖超参数
model = MyModel.load_from_checkpoint(
    "path/to/checkpoint.ckpt",
    learning_rate=0.0001  # 覆盖保存值
)

# 用于推理
model.eval()
predictions = model(data)
```

## 常用模式

### 梯度累积
交由 Lightning 处理：

```python
trainer = L.Trainer(accumulate_grad_batches=4)
```

### 梯度裁剪
在 Trainer 中配置：

```python
trainer = L.Trainer(gradient_clip_val=0.5, gradient_clip_algorithm="norm")
```

### 混合精度训练
在 Trainer 中配置精度：

```python
trainer = L.Trainer(precision="16-mixed")  # 或 "bf16-mixed", "32-true"
```

### 学习率预热
在 configure_optimizers 实现：

```python
def configure_optimizers(self):
    optimizer = torch.optim.Adam(self.parameters(), lr=0.001)
    scheduler = {
        "scheduler": torch.optim.lr_scheduler.OneCycleLR(
            optimizer,
            max_lr=0.01,
            total_steps=self.trainer.estimated_stepping_batches
        ),
        "interval": "step"
    }
    return [optimizer], [scheduler]
```
