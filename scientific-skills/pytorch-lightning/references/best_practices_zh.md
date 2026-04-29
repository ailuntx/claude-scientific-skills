# PyTorch Lightning 最佳实践

## 代码组织

### 1. 分离研究与工程代码

**推荐做法:**
```python
class MyModel(L.LightningModule):
    # 研究代码（模型核心逻辑）
    def training_step(self, batch, batch_idx):
        loss = self.compute_loss(batch)
        return loss

# 工程代码（训练配置）- 在Trainer中实现
trainer = L.Trainer(
    max_epochs=100,
    accelerator="gpu",
    devices=4,
    strategy="ddp"
)
```

**不推荐做法:**
```python
# 混合研究与工程逻辑
class MyModel(L.LightningModule):
    def training_step(self, batch, batch_idx):
        loss = self.compute_loss(batch)

        # 避免手动管理设备
        loss = loss.cuda()

        # 避免手动执行优化步骤（除非手动优化模式）
        self.optimizer.zero_grad()
        loss.backward()
        self.optimizer.step()

        return loss
```

### 2. 使用 LightningDataModule

**推荐做法:**
```python
class MyDataModule(L.LightningDataModule):
    def __init__(self, data_dir, batch_size):
        super().__init__()
        self.data_dir = data_dir
        self.batch_size = batch_size

    def prepare_data(self):
        # 单次下载数据
        download_data(self.data_dir)

    def setup(self, stage):
        # 按进程加载数据
        self.train_dataset = MyDataset(self.data_dir, split='train')
        self.val_dataset = MyDataset(self.data_dir, split='val')

    def train_dataloader(self):
        return DataLoader(self.train_dataset, batch_size=self.batch_size, shuffle=True)

# 可复用且可共享
dm = MyDataModule("./data", batch_size=32)
trainer.fit(model, datamodule=dm)
```

**不推荐做法:**
```python
# 分散的数据逻辑
train_dataset = load_data()
val_dataset = load_data()
train_loader = DataLoader(train_dataset, ...)
val_loader = DataLoader(val_dataset, ...)
trainer.fit(model, train_loader, val_loader)
```

### 3. 保持模型模块化

```python
class Encoder(nn.Module):
    def __init__(self):
        super().__init__()
        self.layers = nn.Sequential(...)

    def forward(self, x):
        return self.layers(x)

class Decoder(nn.Module):
    def __init__(self):
        super().__init__()
        self.layers = nn.Sequential(...)

    def forward(self, x):
        return self.layers(x)

class MyModel(L.LightningModule):
    def __init__(self):
        super().__init__()
        self.encoder = Encoder()
        self.decoder = Decoder()

    def forward(self, x):
        z = self.encoder(x)
        return self.decoder(z)
```

## 设备无关性

### 1. 避免显式 CUDA 调用

**不推荐做法:**
```python
x = x.cuda()
model = model.cuda()
torch.cuda.set_device(0)
```

**推荐做法:**
```python
# 在LightningModule内部
x = x.to(self.device)

# 或让Lightning自动处理
def training_step(self, batch, batch_idx):
    x, y = batch  # 已在正确设备上
    return loss
```

### 2. 使用 `self.device` 属性

```python
class MyModel(L.LightningModule):
    def training_step(self, batch, batch_idx):
        # 在正确设备上创建张量
        noise = torch.randn(batch.size(0), 100).to(self.device)

        # 或使用type_as
        noise = torch.randn(batch.size(0), 100).type_as(batch)
```

### 3. 为非参数注册缓冲区

```python
class MyModel(L.LightningModule):
    def __init__(self):
        super().__init__()
        # 注册缓冲区（自动移至正确设备）
        self.register_buffer("running_mean", torch.zeros(100))

    def forward(self, x):
        # self.running_mean 自动位于正确设备
        return x - self.running_mean
```

## 超参数管理

### 1. 始终使用 `save_hyperparameters()`

**推荐做法:**
```python
class MyModel(L.LightningModule):
    def __init__(self, learning_rate, hidden_dim, dropout):
        super().__init__()
        self.save_hyperparameters()  # 保存所有参数

        # 通过self.hparams访问
        self.model = nn.Linear(self.hparams.hidden_dim, 10)

# 从检查点加载保存的超参数
model = MyModel.load_from_checkpoint("checkpoint.ckpt")
print(model.hparams.learning_rate)  # 保留原始值
```

**不推荐做法:**
```python
class MyModel(L.LightningModule):
    def __init__(self, learning_rate, hidden_dim, dropout):
        super().__init__()
        self.learning_rate = learning_rate  # 手动跟踪
        self.hidden_dim = hidden_dim
```

### 2. 忽略特定参数

```python
class MyModel(L.LightningModule):
    def __init__(self, lr, model, dataset):
        super().__init__()
        # 不保存'model'和'dataset'（不可序列化）
        self.save_hyperparameters(ignore=['model', 'dataset'])

        self.model = model
        self.dataset = dataset
```

### 3. 在 `configure_optimizers()` 中使用超参数

```python
def configure_optimizers(self):
    # 使用保存的超参数
    optimizer = torch.optim.Adam(
        self.parameters(),
        lr=self.hparams.learning_rate,
        weight_decay=self.hparams.weight_decay
    )
    return optimizer
```

## 日志最佳实践

### 1. 同时记录步骤和周期指标

```python
def training_step(self, batch, batch_idx):
    loss = self.compute_loss(batch)

    # 记录步骤级指标用于详细监控
    # 记录周期级指标用于聚合视图
    self.log("train_loss", loss, on_step=True, on_epoch=True, prog_bar=True)

    return loss
```

### 2. 使用结构化日志

```python
def training_step(self, batch, batch_idx):
    # 使用前缀组织日志
    self.log("train/loss", loss)
    self.log("train/acc", acc)
    self.log("train/f1", f1)

def validation_step(self, batch, batch_idx):
    self.log("val/loss", loss)
    self.log("val/acc", acc)
    self.log("val/f1", f1)
```

### 3. 分布式训练中的指标同步

```python
def validation_step(self, batch, batch_idx):
    loss = self.compute_loss(batch)

    # 重要：sync_dist=True 确保跨GPU正确聚合
    self.log("val_loss", loss, sync_dist=True)
```

### 4. 监控学习率

```python
from lightning.pytorch.callbacks import LearningRateMonitor

trainer = L.Trainer(
    callbacks=[LearningRateMonitor(logging_interval="step")]
)
```

## 可复现性

### 1. 全局设置随机种子

```python
import lightning as L

# 设置随机种子保证可复现性
L.seed_everything(42, workers=True)

trainer = L.Trainer(
    deterministic=True,  # 使用确定性算法
    benchmark=False      # 禁用cudnn基准测试
)
```

### 2. 避免非确定性操作

```python
# 错误：非确定性
torch.use_deterministic_algorithms(False)

# 正确：确定性
torch.use_deterministic_algorithms(True)
```

### 3. 记录随机状态

```python
def on_save_checkpoint(self, checkpoint):
    # 保存随机状态
    checkpoint['rng_state'] = {
        'torch': torch.get_rng_state(),
        'numpy': np.random.get_state(),
        'python': random.getstate()
    }

def on_load_checkpoint(self, checkpoint):
    # 恢复随机状态
    if 'rng_state' in checkpoint:
        torch.set_rng_state(checkpoint['rng_state']['torch'])
        np.random.set_state(checkpoint['rng_state']['numpy'])
        random.setstate(checkpoint['rng_state']['python'])
```

## 调试技巧

### 1. 使用 `fast_dev_run`

```python
# 完整训练前用单批次测试
trainer = L.Trainer(fast_dev_run=True)
trainer.fit(model, datamodule=dm)
```

### 2. 限制训练数据量

```python
# 仅使用10%数据进行快速迭代
trainer = L.Trainer(
    limit_train_batches=0.1,
    limit_val_batches=0.1
)
```

### 3. 启用异常检测

```python
# 检测梯度中的NaN/Inf
trainer = L.Trainer(detect_anomaly=True)
```

### 4. 小批量过拟合测试

```python
# 用10个批次过拟合以验证模型容量
trainer = L.Trainer(overfit_batches=10)
```

### 5. 代码性能分析

```python
# 定位性能瓶颈
trainer = L.Trainer(profiler="simple")  # 或 "advanced"
```

## 内存优化

### 1. 使用混合精度

```python
# FP16/BF16混合精度节省内存并加速
trainer = L.Trainer(
    precision="16-mixed",   # V100, T4
    # 或
    precision="bf16-mixed"  # A100, H100
)
```

### 2. 梯度累积

```python
# 模拟更大批次尺寸而不增加内存
trainer = L.Trainer(
    accumulate_grad_batches=4  # 累积4个批次
)
```

### 3. 梯度检查点

```python
class MyModel(L.LightningModule):
    def __init__(self):
        super().__init__()
        self.model = transformers.AutoModel.from_pretrained("bert-base")

        # 启用梯度检查点
        self.model.gradient_checkpointing_enable()
```

### 4. 清理缓存

```python
def on_train_epoch_end(self):
    # 清理收集的输出以释放内存
    self.training_step_outputs.clear()

    # 按需清理CUDA缓存
    if torch.cuda.is_available():
        torch.cuda.empty_cache()
```

### 5. 使用高效数据类型

```python
# 选择合适的精度
# FP32保证稳定性，FP16/BF16提升速度/内存效率

class MyModel(L.LightningModule):
    def __init__(self):
        super().__init__()
        # 使用bfloat16获得比fp16更好的数值稳定性
        self.model = MyTransformer().to(torch.bfloat16)
```

## 训练稳定性

### 1. 梯度裁剪

```python
# 防止梯度爆炸
trainer = L.Trainer(
    gradient_clip_val=1.0,
    gradient_clip_algorithm="norm"  # 或 "value"
)
```

### 2. 学习率预热

```python
def configure_optimizers(self):
    optimizer = torch.optim.Adam(self.parameters(), lr=1e-3)

    scheduler = torch.optim.lr_scheduler.OneCycleLR(
        optimizer,
        max_lr=1e-2,
        total_steps=self.trainer.estimated_stepping_batches,
        pct_start=0.1  # 10%预热
    )

    return {
        "optimizer": optimizer,
        "lr_scheduler": {
            "scheduler": scheduler,
            "interval": "step"
        }
    }
```

### 3. 监控梯度

```python
class MyModel(L.LightningModule):
    def on_after_backward(self):
        # 记录梯度范数
        for name, param in self.named_parameters():
            if param.grad is not None:
                self.log(f"grad_norm/{name}", param.grad.norm())
```

### 4. 使用早停策略

```python
from lightning.pytorch.callbacks import EarlyStopping

early_stop = EarlyStopping(
    monitor="val_loss",
    patience=10,
    mode="min",
    verbose=True
)

trainer = L.Trainer(callbacks=[early_stop])
```

## 检查点管理

### 1. 保存Top-K和最终模型

```python
from lightning.pytorch.callbacks import ModelCheckpoint

checkpoint_callback = ModelCheckpoint(
    dirpath="checkpoints/",
    filename="{epoch}-{val_loss:.2f}",
    monitor="val_loss",
    mode="min",
    save_top_k=3,    # 保留最优3个
    save_last=True   # 始终保存最终模型用于恢复
)

trainer = L.Trainer(callbacks=[checkpoint_callback])
```

### 2. 恢复训练

```python
# 从最终检查点恢复
trainer.fit(model, datamodule=dm, ckpt_path="last.ckpt")

# 从特定检查点恢复
trainer.fit(model, datamodule=dm, ckpt_path="epoch=10-val_loss=0.23.ckpt")
```

### 3. 自定义检查点状态

```python
def on_save_checkpoint(self, checkpoint):
    # 添加自定义状态
    checkpoint['custom_data'] = self.custom_data
    checkpoint['epoch_metrics'] = self.metrics

def on_load_checkpoint(self, checkpoint):
    # 恢复自定义状态
    self.custom_data = checkpoint.get('custom_data', {})
    self.metrics = checkpoint.get('epoch_metrics', [])
```

## 测试规范

### 1. 分离训练与测试

```python
# 训练阶段
trainer = L.Trainer(max_epochs=100)
trainer.fit(model, datamodule=dm)

# 发布前仅执行一次测试
trainer.test(model, datamodule=dm)
```

### 2. 使用验证集进行模型选择

```python
# 使用验证集进行超参数调优
checkpoint_callback = ModelCheckpoint(monitor="val_loss", mode="min")
trainer = L.Trainer(callbacks=[checkpoint_callback])
trainer.fit(model, datamodule=dm)

# 加载最优模型
best_model = MyModel.load_from_checkpoint(checkpoint_callback.best_model_path)

# 仅用最优模型测试一次
trainer.test(best_model, datamodule=dm)
```

## 代码质量

### 1. 类型注解

```python
from typing import Any, Dict, Tuple
import torch
from torch import Tensor

class MyModel(L.LightningModule):
    def training_step(self, batch: Tuple[Tensor, Tensor], batch_idx: int) -> Tensor:
        x, y = batch
        loss = self.compute_loss(x, y)
        return loss

    def configure_optimizers(self) -> Dict[str, Any]:
        optimizer = torch.optim.Adam(self.parameters())
        return {"optimizer": optimizer}
```

### 2. 文档字符串

```python
class MyModel(L.LightningModule):
    """
    用于图像分类的卓越模型。

    参数:
        num_classes: 输出类别数量
        learning_rate: 优化器学习率
        hidden_dim: 隐藏层维度大小
    """

    def __init__(self, num_classes: int, learning_rate: float, hidden_dim: int):
        super().__init__()
        self.save_hyperparameters()
```

### 3. 属性方法

```python
class MyModel(L.LightningModule):
    @property
    def learning_rate(self) -> float:
        """当前学习率"""
        return self.hparams.learning_rate

    @property
    def num_parameters(self) -> int:
        """参数量总数"""
        return sum(p.numel() for p in self.parameters())
```

## 常见陷阱

### 1. 忘记返回损失值

**错误:**
```python
def training_step(self, batch, batch_idx):
    loss = self.compute_loss(batch)
    self.log("train_loss", loss)
    # 忘记返回损失值！
```

**正确:**
```python
def training_step(self, batch, batch_idx):
    loss = self.compute_loss(batch)
    self.log("train_loss", loss)
    return loss  # 必须返回损失值
```

### 2. DDP模式下未同步指标

**错误:**
```python
def validation_step(self, batch, batch_idx):
    self.log("val_acc", acc)  # 多GPU环境下数值错误！
```

**正确:**
```python
def validation_step(self, batch, batch_idx):
    self.log("val_acc", acc, sync_dist=True)  # 正确聚合
```

### 3. 手动管理设备

**错误:**
```python
def training_step(self, batch, batch_idx):
    x = x.cuda()  # 禁止此操作
    y = y.cuda()
```

**正确:**
```

### 5. 原地修改批次数据

**错误做法：**
```python
def training_step(self, batch, batch_idx):
    x, y = batch
    x[:] = self.augment(x)  # 原地修改可能导致问题
```

**正确做法：**
```python
def training_step(self, batch, batch_idx):
    x, y = batch
    x = self.augment(x)  # 创建新的张量
```

## 性能优化建议

### 1. 使用 DataLoader 多进程

```python
def train_dataloader(self):
    return DataLoader(
        self.train_dataset,
        batch_size=32,
        num_workers=4,           # 使用多个工作进程
        pin_memory=True,         # 加速 GPU 数据传输
        persistent_workers=True  # 保持工作进程存活
    )
```

### 2. 启用基准测试模式（当输入尺寸固定时）

```python
trainer = L.Trainer(benchmark=True)
```

### 3. 使用自动批次大小调整

```python
from lightning.pytorch.tuner import Tuner

trainer = L.Trainer()
tuner = Tuner(trainer)

# 寻找最佳批次大小
tuner.scale_batch_size(model, datamodule=dm, mode="power")

# 然后开始训练
trainer.fit(model, datamodule=dm)
```

### 4. 优化数据加载

```python
# 使用更快的图像解码
import torch
import torchvision.transforms as T

transforms = T.Compose([
    T.ToTensor(),
    T.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

# 使用 PIL-SIMD 加速图像加载
# pip install pillow-simd
```
