---
name: pytorch-lightning
description: 深度学习框架（PyTorch Lightning）。将PyTorch代码组织为LightningModule，配置多GPU/TPU训练器，实现数据管道、回调函数、日志记录（W&B、TensorBoard）和分布式训练（DDP、FSDP、DeepSpeed），用于可扩展的神经网络训练。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# PyTorch Lightning

## 概述

PyTorch Lightning 是一个深度学习框架，通过组织 PyTorch 代码消除样板代码，同时保持完全灵活性。自动化训练流程、多设备编排，并实现跨多GPU/TPU的神经网络训练与扩展最佳实践。

## 适用场景

在以下场景应使用此技能：
- 使用 PyTorch Lightning 构建、训练或部署神经网络
- 将 PyTorch 代码组织为 LightningModule
- 配置多 GPU/TPU 训练器
- 通过 LightningDataModule 实现数据管道
- 使用回调函数、日志记录和分布式训练策略（DDP、FSDP、DeepSpeed）
- 专业构建深度学习项目

## 核心功能

### 1. LightningModule - 模型定义

将 PyTorch 模型组织为六个逻辑部分：

1. **初始化** - `__init__()` 和 `setup()`
2. **训练循环** - `training_step(batch, batch_idx)`
3. **验证循环** - `validation_step(batch, batch_idx)`
4. **测试循环** - `test_step(batch, batch_idx)`
5. **预测** - `predict_step(batch, batch_idx)`
6. **优化器配置** - `configure_optimizers()`

**快速模板参考：** 完整样板代码见 `scripts/template_lightning_module.py`

**详细文档：** 阅读 `references/lightning_module.md` 获取完整方法文档、钩子函数、属性和最佳实践

### 2. Trainer - 训练自动化

Trainer 自动处理训练循环、设备管理、梯度操作和回调函数。核心特性包括：
- 多 GPU/TPU 支持及策略选择（DDP、FSDP、DeepSpeed）
- 自动混合精度训练
- 梯度累积与裁剪
- 检查点保存与早停机制
- 进度条与日志记录

**快速配置参考：** 常见 Trainer 配置示例见 `scripts/quick_trainer_setup.py`

**详细文档：** 阅读 `references/trainer.md` 获取所有参数、方法和配置选项

### 3. LightningDataModule - 数据管道组织

在可复用类中封装数据处理流程：
1. `prepare_data()` - 下载和处理数据（单进程）
2. `setup()` - 创建数据集并应用转换（每 GPU）
3. `train_dataloader()` - 返回训练 DataLoader
4. `val_dataloader()` - 返回验证 DataLoader
5. `test_dataloader()` - 返回测试 DataLoader

**快速模板参考：** 完整样板代码见 `scripts/template_datamodule.py`

**详细文档：** 阅读 `references/data_module.md` 获取方法细节和使用模式

### 4. Callbacks - 可扩展训练逻辑

在不修改 LightningModule 的情况下，通过特定训练钩子添加自定义功能。内置回调包括：
- **ModelCheckpoint** - 保存最佳/最新模型
- **EarlyStopping** - 指标停滞时停止训练
- **LearningRateMonitor** - 跟踪学习率调度变化
- **BatchSizeFinder** - 自动确定最佳批大小

**详细文档：** 阅读 `references/callbacks.md` 获取内置回调及自定义回调创建指南

### 5. Logging - 实验跟踪

集成多平台日志记录：
- TensorBoard（默认）
- Weights & Biases（WandbLogger）
- MLflow（MLFlowLogger）
- Neptune（NeptuneLogger）
- Comet（CometLogger）
- CSV（CSVLogger）

在任意 LightningModule 方法中使用 `self.log("metric_name", value)` 记录指标

**详细文档：** 阅读 `references/logging.md` 获取日志器设置与配置

### 6. 分布式训练 - 多设备扩展

根据模型规模选择策略：
- **DDP** - 适用于 <5亿参数模型（ResNet、小型Transformer）
- **FSDP** - 适用于 5亿+ 参数模型（大型Transformer，Lightning用户推荐）
- **DeepSpeed** - 适用于前沿特性和精细控制

配置示例：`Trainer(strategy="ddp", accelerator="gpu", devices=4)`

**详细文档：** 阅读 `references/distributed_training.md` 获取策略对比与配置指南

### 7. 最佳实践

- 设备无关代码 - 使用 `self.device` 替代 `.cuda()`
- 超参数保存 - 在 `__init__()` 中使用 `self.save_hyperparameters()`
- 指标记录 - 使用 `self.log()` 实现跨设备自动聚合
- 可复现性 - 使用 `seed_everything()` 和 `Trainer(deterministic=True)`
- 调试 - 使用 `Trainer(fast_dev_run=True)` 单批次测试

**详细文档：** 阅读 `references/best_practices.md` 获取常见模式与陷阱

## 快速工作流

1. **定义模型：**
   ```python
   class MyModel(L.LightningModule):
       def __init__(self):
           super().__init__()
           self.save_hyperparameters()
           self.model = YourNetwork()

       def training_step(self, batch, batch_idx):
           x, y = batch
           loss = F.cross_entropy(self.model(x), y)
           self.log("train_loss", loss)
           return loss

       def configure_optimizers(self):
           return torch.optim.Adam(self.parameters())
   ```

2. **准备数据：**
   ```python
   # 方案1：直接使用DataLoader
   train_loader = DataLoader(train_dataset, batch_size=32)

   # 方案2：LightningDataModule（推荐复用）
   dm = MyDataModule(batch_size=32)
   ```

3. **训练：**
   ```python
   trainer = L.Trainer(max_epochs=10, accelerator="gpu", devices=2)
   trainer.fit(model, train_loader)  # 或 trainer.fit(model, datamodule=dm)
   ```

## 资源

### scripts/
可执行Python模板，包含常见PyTorch Lightning模式：
- `template_lightning_module.py` - 完整LightningModule样板
- `template_datamodule.py` - 完整LightningDataModule样板
- `quick_trainer_setup.py` - 常用Trainer配置示例

### references/
各PyTorch Lightning组件的详细文档：
- `lightning_module.md` - LightningModule综合指南（方法、钩子、属性）
- `trainer.md` - Trainer配置与参数说明
- `data_module.md` - LightningDataModule模式与方法
- `callbacks.md` - 内置及自定义回调函数
- `logging.md` - 日志器集成与使用
- `distributed_training.md` - DDP/FSDP/DeepSpeed对比与设置
- `best_practices.md` - 常见模式、技巧与陷阱
