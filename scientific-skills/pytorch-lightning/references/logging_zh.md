# 日志记录 - 全面指南

## 概述

PyTorch Lightning 支持多种日志集成方案，用于实验跟踪和可视化。默认情况下，Lightning 使用 TensorBoard，但您可以轻松切换或组合多个日志记录器。

## 支持的日志记录器

### TensorBoardLogger（默认）

以 TensorBoard 格式记录到本地或远程文件系统。

**安装：**
```bash
pip install tensorboard
```

**用法：**
```python
from lightning.pytorch import loggers as pl_loggers

tb_logger = pl_loggers.TensorBoardLogger(
    save_dir="logs/",
    name="my_model",
    version="version_1",
    default_hp_metric=False
)

trainer = L.Trainer(logger=tb_logger)
```

**查看日志：**
```bash
tensorboard --logdir logs/
```

### WandbLogger

Weights & Biases 集成，用于基于云的实验跟踪。

**安装：**
```bash
pip install wandb
```

**用法：**
```python
from lightning.pytorch import loggers as pl_loggers

wandb_logger = pl_loggers.WandbLogger(
    project="my-project",
    name="experiment-1",
    save_dir="logs/",
    log_model=True  # 将模型检查点记录到 W&B
)

trainer = L.Trainer(logger=wandb_logger)
```

**功能：**
- 基于云的实验跟踪
- 模型版本控制
- 工件管理
- 协作功能
- 超参数扫描

### MLFlowLogger

MLflow 跟踪集成。

**安装：**
```bash
pip install mlflow
```

**用法：**
```python
from lightning.pytorch import loggers as pl_loggers

mlflow_logger = pl_loggers.MLFlowLogger(
    experiment_name="my_experiment",
    tracking_uri="http://localhost:5000",
    run_name="run_1"
)

trainer = L.Trainer(logger=mlflow_logger)
```

### CometLogger

Comet.ml 实验跟踪。

**安装：**
```bash
pip install comet-ml
```

**用法：**
```python
from lightning.pytorch import loggers as pl_loggers

comet_logger = pl_loggers.CometLogger(
    api_key="YOUR_API_KEY",
    project_name="my-project",
    experiment_name="experiment-1"
)

trainer = L.Trainer(logger=comet_logger)
```

### NeptuneLogger

Neptune.ai 集成。

**安装：**
```bash
pip install neptune
```

**用法：**
```python
from lightning.pytorch import loggers as pl_loggers

neptune_logger = pl_loggers.NeptuneLogger(
    api_key="YOUR_API_KEY",
    project="username/project-name",
    name="experiment-1"
)

trainer = L.Trainer(logger=neptune_logger)
```

### CSVLogger

以 YAML 和 CSV 格式记录到本地文件系统。

**用法：**
```python
from lightning.pytorch import loggers as pl_loggers

csv_logger = pl_loggers.CSVLogger(
    save_dir="logs/",
    name="my_model",
    version="1"
)

trainer = L.Trainer(logger=csv_logger)
```

**输出文件：**
- `metrics.csv` - 所有记录的指标
- `hparams.yaml` - 超参数

## 记录指标

### 基础记录

在 LightningModule 中使用 `self.log()`：

```python
class MyModel(L.LightningModule):
    def training_step(self, batch, batch_idx):
        x, y = batch
        y_hat = self.model(x)
        loss = F.cross_entropy(y_hat, y)

        # 记录指标
        self.log("train_loss", loss)

        return loss

    def validation_step(self, batch, batch_idx):
        x, y = batch
        y_hat = self.model(x)
        loss = F.cross_entropy(y_hat, y)
        acc = (y_hat.argmax(dim=1) == y).float().mean()

        # 记录多个指标
        self.log("val_loss", loss)
        self.log("val_acc", acc)
```

### 记录参数

#### `on_step` (布尔值)
在当前步骤记录。默认：在 training_step 中为 True，其他情况为 False。

```python
self.log("loss", loss, on_step=True)
```

#### `on_epoch` (布尔值)
在周期结束时累积并记录。默认：在 training_step 中为 False，其他情况为 True。

```python
self.log("loss", loss, on_epoch=True)
```

#### `prog_bar` (布尔值)
在进度条中显示。默认：False。

```python
self.log("train_loss", loss, prog_bar=True)
```

#### `logger` (布尔值)
发送到日志记录器后端。默认：True。

```python
self.log("internal_metric", value, logger=False)  # 不记录到外部日志记录器
```

#### `reduce_fx` (字符串或可调用对象)
归约函数："mean"、"sum"、"max"、"min"。默认："mean"。

```python
self.log("batch_size", batch.size(0), reduce_fx="sum")
```

#### `sync_dist` (布尔值)
在分布式训练中跨设备同步指标。默认：False。

```python
self.log("loss", loss, sync_dist=True)
```

#### `rank_zero_only` (布尔值)
仅从 rank 0 进程记录。默认：False。

```python
self.log("debug_metric", value, rank_zero_only=True)
```

### 完整示例

```python
def training_step(self, batch, batch_idx):
    loss = self.compute_loss(batch)

    # 记录每步和每周期指标，在进度条显示
    self.log("train_loss", loss, on_step=True, on_epoch=True, prog_bar=True)

    return loss

def validation_step(self, batch, batch_idx):
    loss = self.compute_loss(batch)
    acc = self.compute_accuracy(batch)

    # 记录周期级指标
    self.log("val_loss", loss, on_epoch=True)
    self.log("val_acc", acc, on_epoch=True, prog_bar=True)
```

### 记录多个指标

使用 `log_dict()` 一次性记录多个指标：

```python
def training_step(self, batch, batch_idx):
    loss, acc, f1 = self.compute_metrics(batch)

    metrics = {
        "train_loss": loss,
        "train_acc": acc,
        "train_f1": f1
    }

    self.log_dict(metrics, on_step=True, on_epoch=True)

    return loss
```

## 记录超参数

### 自动超参数记录

在模型中使用 `save_hyperparameters()`：

```python
class MyModel(L.LightningModule):
    def __init__(self, learning_rate, hidden_dim, dropout):
        super().__init__()
        # 自动保存并记录超参数
        self.save_hyperparameters()
```

### 手动超参数记录

```python
# 在 LightningModule 中
class MyModel(L.LightningModule):
    def __init__(self, learning_rate):
        super().__init__()
        self.save_hyperparameters()

# 或通过日志记录器手动记录
trainer.logger.log_hyperparams({
    "learning_rate": 0.001,
    "batch_size": 32
})
```

## 记录频率

默认情况下，Lightning 每 50 个训练步骤记录一次。可通过 `log_every_n_steps` 调整：

```python
trainer = L.Trainer(log_every_n_steps=10)
```

## 多日志记录器

同时使用多个日志记录器：

```python
from lightning.pytorch import loggers as pl_loggers

tb_logger = pl_loggers.TensorBoardLogger("logs/")
wandb_logger = pl_loggers.WandbLogger(project="my-project")
csv_logger = pl_loggers.CSVLogger("logs/")

trainer = L.Trainer(logger=[tb_logger, wandb_logger, csv_logger])
```

## 高级记录

### 记录图像

```python
import torchvision

def validation_step(self, batch, batch_idx):
    x, y = batch
    y_hat = self.model(x)

    # 每周期记录第一批图像
    if batch_idx == 0:
        # 创建图像网格
        grid = torchvision.utils.make_grid(x[:8])

        # 记录到 TensorBoard
        self.logger.experiment.add_image("val_images", grid, self.current_epoch)

        # 记录到 Wandb
        if isinstance(self.logger, pl_loggers.WandbLogger):
            import wandb
            self.logger.experiment.log({
                "val_images": [wandb.Image(img) for img in x[:8]]
            })
```

### 记录直方图

```python
def on_train_epoch_end(self):
    # 记录参数直方图
    for name, param in self.named_parameters():
        self.logger.experiment.add_histogram(name, param, self.current_epoch)

        if param.grad is not None:
            self.logger.experiment.add_histogram(
                f"{name}_grad", param.grad, self.current_epoch
            )
```

### 记录模型图

```python
def on_train_start(self):
    # 记录模型架构
    sample_input = torch.randn(1, 3, 224, 224).to(self.device)
    self.logger.experiment.add_graph(self.model, sample_input)
```

### 记录自定义图表

```python
import matplotlib.pyplot as plt

def on_validation_epoch_end(self):
    # 创建自定义图表
    fig, ax = plt.subplots()
    ax.plot(self.validation_losses)
    ax.set_xlabel("周期")
    ax.set_ylabel("损失值")

    # 记录到 TensorBoard
    self.logger.experiment.add_figure("loss_curve", fig, self.current_epoch)

    plt.close(fig)
```

### 记录文本

```python
def validation_step(self, batch, batch_idx):
    # 生成预测
    predictions = self.generate_text(batch)

    # 记录到 TensorBoard
    self.logger.experiment.add_text(
        "predictions",
        f"批次 {batch_idx}: {predictions}",
        self.current_epoch
    )
```

### 记录音频

```python
def validation_step(self, batch, batch_idx):
    audio = self.generate_audio(batch)

    # 记录到 TensorBoard（音频为形状 [1, 采样数] 的张量）
    self.logger.experiment.add_audio(
        "generated_audio",
        audio,
        self.current_epoch,
        sample_rate=22050
    )
```

## 在 LightningModule 中访问日志记录器

```python
class MyModel(L.LightningModule):
    def training_step(self, batch, batch_idx):
        # 访问日志记录器实验对象
        logger = self.logger.experiment

        # TensorBoard 专用
        if isinstance(self.logger, pl_loggers.TensorBoardLogger):
            logger.add_scalar("custom_metric", value, self.global_step)

        # Wandb 专用
        if isinstance(self.logger, pl_loggers.WandbLogger):
            logger.log({"custom_metric": value})

        # MLflow 专用
        if isinstance(self.logger, pl_loggers.MLFlowLogger):
            logger.log_metric("custom_metric", value)
```

## 自定义日志记录器

通过继承 `Logger` 创建自定义日志记录器：

```python
from lightning.pytorch.loggers import Logger
from lightning.pytorch.utilities import rank_zero_only

class MyCustomLogger(Logger):
    def __init__(self, save_dir):
        super().__init__()
        self.save_dir = save_dir
        self._name = "my_logger"
        self._version = "0.1"

    @property
    def name(self):
        return self._name

    @property
    def version(self):
        return self._version

    @rank_zero_only
    def log_metrics(self, metrics, step):
        # 将指标记录到您的后端
        print(f"步骤 {step}: {metrics}")

    @rank_zero_only
    def log_hyperparams(self, params):
        # 记录超参数
        print(f"超参数: {params}")

    @rank_zero_only
    def save(self):
        # 保存日志记录器状态
        pass

    @rank_zero_only
    def finalize(self, status):
        # 训练结束时清理
        pass

# 使用示例
custom_logger = MyCustomLogger(save_dir="logs/")
trainer = L.Trainer(logger=custom_logger)
```

## 最佳实践

### 1. 同时记录步骤和周期指标

```python
# 良好实践：同时跟踪细粒度和聚合指标
self.log("train_loss", loss, on_step=True, on_epoch=True)
```

### 2. 关键指标使用进度条显示

```python
# 在进度条显示重要指标
self.log("val_acc", acc, prog_bar=True)
```

### 3. 分布式训练中同步指标

```python
# 确保跨 GPU 正确聚合
self.log("val_loss", loss, sync_dist=True)
```

### 4. 记录学习率

```python
from lightning.pytorch.callbacks import LearningRateMonitor

trainer = L.Trainer(callbacks=[LearningRateMonitor(logging_interval="step")])
```

### 5. 记录梯度范数

```python
def on_after_backward(self):
    # 监控梯度流
    grad_norm = torch.nn.utils.clip_grad_norm_(self.parameters(), max_norm=float('inf'))
    self.log("grad_norm", grad_norm)
```

### 6. 使用描述性指标名称

```python
# 良好实践：清晰的命名约定
self.log("train/loss", loss)
self.log("train/accuracy", acc)
self.log("val/loss", val_loss)
self.log("val/accuracy", val_acc)
```

### 7. 记录超参数

```python
# 始终保存超参数以确保可复现性
class MyModel(L.LightningModule):
    def __init__(self, **kwargs):
        super().__init__()
        self.save_hyperparameters()
```

### 8. 避免过于频繁记录

```python
# 避免对高开销操作每步记录
if batch_idx % 100 == 0:
    self.log_images(batch)
```

## 常见模式

### 结构化记录

```python
def training_step(self, batch, batch_idx):
    loss, metrics = self.compute_loss_and_metrics(batch)

    # 使用前缀组织日志
    self.log("train/loss", loss)
    self.log_dict({f"train/{k}": v for k, v in metrics.items()})

    return loss

def validation_step(self, batch, batch_idx):
    loss, metrics = self.compute_loss_and_metrics(batch)

    self.log("val/loss", loss)
    self.log_dict({f"val/{k}": v for k, v in metrics.items()})
```

### 条件记录

```python
def training_step(self, batch, batch_idx):
    loss = self.compute_loss(batch)

    # 减少高开销指标的记录频率
    if self.global_step % 100 == 0:
        expensive_metric = self.compute_expensive_metric(batch)
        self.log("expensive_metric", expensive_metric)

    self.log("train_loss", loss)
    return loss
```

### 多任务记录

```python
def training_step(self, batch, batch_idx):
    x, y_task1, y_task2 = batch

    loss_task1 = self.compute_task1_loss(x, y_task1)
    loss_task2 = self.compute_task2_loss(x, y_task2)
    total_loss = loss_task1 + loss_task2

    # 记录每任务指标
    self.log_dict({
        "train/loss_task1": loss_task1,
        "train/loss_task2": loss_task2,
        "train/loss_total": total_loss
    })

    return total_loss
```

## 故障排除

### 指标未找到错误

如果调度器出现"指标未找到"错误：

```python
# 确保指标以 logger=True 记录
self.log("val_loss", loss, logger=True)

# 并配置调度器监控该指标
def configure_optimizers(self):
    optimizer = torch.optim.Adam(self.parameters
