# LightningDataModule - 全面指南

## 概述

LightningDataModule 是 PyTorch Lightning 中一个可复用、可共享的类，它封装了所有数据处理步骤。通过标准化数据集的管理和跨项目共享方式，它解决了数据准备逻辑分散的问题。

## 核心解决的问题

在传统 PyTorch 工作流中，数据处理逻辑分散在多个文件中，导致难以回答以下问题：
- "你使用了哪些数据划分？"
- "应用了哪些数据变换？"
- "数据是如何准备的？"

DataModule 将这些信息集中管理，确保可复现性和可重用性。

## 五个处理步骤

DataModule 将数据处理组织为五个阶段：

1. **下载/分词/处理** - 初始数据获取
2. **清理并保存** - 将处理后的数据持久化到磁盘
3. **加载到 Dataset** - 创建 PyTorch Dataset 对象
4. **应用变换** - 数据增强、归一化等操作
5. **封装到 DataLoader** - 配置批处理和加载方式

## 主要方法

### `prepare_data()`
下载并处理数据。仅在单个进程上运行一次（非分布式）。

**适用场景：**
- 下载数据集
- 文本分词
- 将处理后的数据保存到磁盘

**重要：** 不要在此设置状态（如 self.x = y）。状态不会传递到其他进程。

**示例：**
```python
def prepare_data(self):
    # 下载数据（仅运行一次）
    download_dataset("http://example.com/data.zip", "data/")

    # 分词并保存（仅运行一次）
    tokenize_and_save("data/raw/", "data/processed/")
```

### `setup(stage)`
创建数据集并应用变换。在分布式训练的每个进程上运行。

**参数：**
- `stage` - 'fit'（训练）、'validate'（验证）、'test'（测试）或 'predict'（预测）

**适用场景：**
- 创建训练/验证/测试划分
- 构建 Dataset 对象
- 应用变换
- 设置状态（self.train_dataset = ...）

**示例：**
```python
def setup(self, stage):
    if stage == 'fit':
        full_dataset = MyDataset("data/processed/")
        self.train_dataset, self.val_dataset = random_split(
            full_dataset, [0.8, 0.2]
        )

    if stage == 'test':
        self.test_dataset = MyDataset("data/processed/test/")

    if stage == 'predict':
        self.predict_dataset = MyDataset("data/processed/predict/")
```

### `train_dataloader()`
返回训练 DataLoader。

**示例：**
```python
def train_dataloader(self):
    return DataLoader(
        self.train_dataset,
        batch_size=self.batch_size,
        shuffle=True,
        num_workers=self.num_workers,
        pin_memory=True
    )
```

### `val_dataloader()`
返回验证 DataLoader（可多个）。

**示例：**
```python
def val_dataloader(self):
    return DataLoader(
        self.val_dataset,
        batch_size=self.batch_size,
        shuffle=False,
        num_workers=self.num_workers,
        pin_memory=True
    )
```

### `test_dataloader()`
返回测试 DataLoader（可多个）。

**示例：**
```python
def test_dataloader(self):
    return DataLoader(
        self.test_dataset,
        batch_size=self.batch_size,
        shuffle=False,
        num_workers=self.num_workers
    )
```

### `predict_dataloader()`
返回预测 DataLoader（可多个）。

**示例：**
```python
def predict_dataloader(self):
    return DataLoader(
        self.predict_dataset,
        batch_size=self.batch_size,
        shuffle=False,
        num_workers=self.num_workers
    )
```

## 完整示例

```python
import lightning as L
from torch.utils.data import DataLoader, Dataset, random_split
import torch

class MyDataset(Dataset):
    def __init__(self, data_path, transform=None):
        self.data_path = data_path
        self.transform = transform
        self.data = self._load_data()

    def _load_data(self):
        # 在此处加载数据
        return torch.randn(1000, 3, 224, 224)

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        sample = self.data[idx]
        if self.transform:
            sample = self.transform(sample)
        return sample

class MyDataModule(L.LightningDataModule):
    def __init__(self, data_dir="./data", batch_size=32, num_workers=4):
        super().__init__()
        self.data_dir = data_dir
        self.batch_size = batch_size
        self.num_workers = num_workers

        # 变换
        self.train_transform = self._get_train_transforms()
        self.test_transform = self._get_test_transforms()

    def _get_train_transforms(self):
        # 定义训练变换
        return lambda x: x  # 占位符

    def _get_test_transforms(self):
        # 定义测试/验证变换
        return lambda x: x  # 占位符

    def prepare_data(self):
        # 下载数据（在单个进程上运行一次）
        # download_data(self.data_dir)
        pass

    def setup(self, stage=None):
        # 创建数据集（在每个进程上运行）
        if stage == 'fit' or stage is None:
            full_dataset = MyDataset(
                self.data_dir,
                transform=self.train_transform
            )
            train_size = int(0.8 * len(full_dataset))
            val_size = len(full_dataset) - train_size
            self.train_dataset, self.val_dataset = random_split(
                full_dataset, [train_size, val_size]
            )

        if stage == 'test' or stage is None:
            self.test_dataset = MyDataset(
                self.data_dir,
                transform=self.test_transform
            )

        if stage == 'predict':
            self.predict_dataset = MyDataset(
                self.data_dir,
                transform=self.test_transform
            )

    def train_dataloader(self):
        return DataLoader(
            self.train_dataset,
            batch_size=self.batch_size,
            shuffle=True,
            num_workers=self.num_workers,
            pin_memory=True,
            persistent_workers=True if self.num_workers > 0 else False
        )

    def val_dataloader(self):
        return DataLoader(
            self.val_dataset,
            batch_size=self.batch_size,
            shuffle=False,
            num_workers=self.num_workers,
            pin_memory=True,
            persistent_workers=True if self.num_workers > 0 else False
        )

    def test_dataloader(self):
        return DataLoader(
            self.test_dataset,
            batch_size=self.batch_size,
            shuffle=False,
            num_workers=self.num_workers
        )

    def predict_dataloader(self):
        return DataLoader(
            self.predict_dataset,
            batch_size=self.batch_size,
            shuffle=False,
            num_workers=self.num_workers
        )
```

## 使用方式

```python
# 创建 DataModule
dm = MyDataModule(data_dir="./data", batch_size=64, num_workers=8)

# 与 Trainer 配合使用
trainer = L.Trainer(max_epochs=10)
trainer.fit(model, datamodule=dm)

# 测试
trainer.test(model, datamodule=dm)

# 预测
predictions = trainer.predict(model, datamodule=dm)

# 或在 PyTorch 中独立使用
dm.prepare_data()
dm.setup(stage='fit')
train_loader = dm.train_dataloader()

for batch in train_loader:
    # 训练代码
    pass
```

## 附加钩子

### `transfer_batch_to_device(batch, device, dataloader_idx)`
自定义将批次数据移动到设备的逻辑。

**示例：**
```python
def transfer_batch_to_device(self, batch, device, dataloader_idx):
    # 自定义转移逻辑
    if isinstance(batch, dict):
        return {k: v.to(device) for k, v in batch.items()}
    return super().transfer_batch_to_device(batch, device, dataloader_idx)
```

### `on_before_batch_transfer(batch, dataloader_idx)`
在将批次数据转移到设备前进行增强或修改（在 CPU 上运行）。

**示例：**
```python
def on_before_batch_transfer(self, batch, dataloader_idx):
    # 应用基于 CPU 的增强
    batch['image'] = apply_augmentation(batch['image'])
    return batch
```

### `on_after_batch_transfer(batch, dataloader_idx)`
在将批次数据转移到设备后进行增强或修改（在 GPU 上运行）。

**示例：**
```python
def on_after_batch_transfer(self, batch, dataloader_idx):
    # 应用基于 GPU 的增强
    batch['image'] = gpu_augmentation(batch['image'])
    return batch
```

### `state_dict()` / `load_state_dict(state_dict)`
保存和恢复 DataModule 状态以实现检查点功能。

**示例：**
```python
def state_dict(self):
    return {"current_fold": self.current_fold}

def load_state_dict(self, state_dict):
    self.current_fold = state_dict["current_fold"]
```

### `teardown(stage)`
训练/测试/预测后的清理操作。

**示例：**
```python
def teardown(self, stage):
    # 清理资源
    if stage == 'fit':
        self.train_dataset = None
        self.val_dataset = None
```

## 高级模式

### 多验证/测试 DataLoader

返回 DataLoader 列表或字典：

```python
def val_dataloader(self):
    return [
        DataLoader(self.val_dataset_1, batch_size=32),
        DataLoader(self.val_dataset_2, batch_size=32)
    ]

# 或使用命名（用于日志记录）
def val_dataloader(self):
    return {
        "val_easy": DataLoader(self.val_easy, batch_size=32),
        "val_hard": DataLoader(self.val_hard, batch_size=32)
    }

# 在 LightningModule 中
def validation_step(self, batch, batch_idx, dataloader_idx=0):
    if dataloader_idx == 0:
        # 处理 val_dataset_1
        pass
    else:
        # 处理 val_dataset_2
        pass
```

### 交叉验证

```python
class CrossValidationDataModule(L.LightningDataModule):
    def __init__(self, data_dir, batch_size, num_folds=5):
        super().__init__()
        self.data_dir = data_dir
        self.batch_size = batch_size
        self.num_folds = num_folds
        self.current_fold = 0

    def setup(self, stage=None):
        full_dataset = MyDataset(self.data_dir)
        fold_size = len(full_dataset) // self.num_folds

        # 创建折叠索引
        indices = list(range(len(full_dataset)))
        val_start = self.current_fold * fold_size
        val_end = val_start + fold_size

        val_indices = indices[val_start:val_end]
        train_indices = indices[:val_start] + indices[val_end:]

        self.train_dataset = Subset(full_dataset, train_indices)
        self.val_dataset = Subset(full_dataset, val_indices)

    def set_fold(self, fold):
        self.current_fold = fold

    def state_dict(self):
        return {"current_fold": self.current_fold}

    def load_state_dict(self, state_dict):
        self.current_fold = state_dict["current_fold"]

# 使用方式
dm = CrossValidationDataModule("./data", batch_size=32, num_folds=5)

for fold in range(5):
    dm.set_fold(fold)
    trainer = L.Trainer(max_epochs=10)
    trainer.fit(model, datamodule=dm)
```

### 超参数保存

```python
class MyDataModule(L.LightningDataModule):
    def __init__(self, data_dir, batch_size=32, num_workers=4):
        super().__init__()
        # 保存超参数
        self.save_hyperparameters()

    def setup(self, stage=None):
        # 通过 self.hparams 访问
        print(f"Batch size: {self.hparams.batch_size}")
```

## 最佳实践

### 1. 分离 prepare_data 和 setup
- `prepare_data()` - 下载/处理（单进程，无状态）
- `setup()` - 创建数据集（所有进程，设置状态）

### 2. 使用 stage 参数
在 `setup()` 中检查 stage 避免不必要操作：

```python
def setup(self, stage):
    if stage == 'fit':
        # 仅在训练时加载训练/验证数据
        self.train_dataset = ...
        self.val_dataset = ...
    elif stage == 'test':
        # 仅在测试时加载测试数据
        self.test_dataset = ...
```

### 3. GPU 训练启用内存固定
在 DataLoader 中启用 `pin_memory=True` 加速 GPU 数据传输：

```python
def train_dataloader(self):
    return DataLoader(..., pin_memory=True)
```

### 4. 使用持久化工作进程
避免 epoch 间重启工作进程：

```python
def train_dataloader(self):
    return DataLoader(
        ...,
        num_workers=4,
        persistent_workers=True
    )
```

### 5. 验证/测试中禁止打乱顺序
切勿打乱验证或测试数据顺序：

```python
def val_dataloader(self):
    return DataLoader(..., shuffle=False)  # 永远不要设为 True
```

### 6. 确保 DataModule 可复用
在 `__init__` 中接收配置参数：

```python
class MyDataModule(L.LightningDataModule):
    def __init__(self, data_dir, batch_size, num_workers, augment=True):
        super().__init__()
        self.save_hyperparameters()
```

### 7. 文档化数据结构
添加文档字符串说明数据格式和预期：

```python
class MyDataModule(L.LightningDataModule):
    """
    XYZ 数据集专用 DataModule。

    数据格式: (图像, 标签) 元组
    - 图像: 形状为 (C, H, W) 的 torch.Tensor
    - 标签: 范围在 [0, num_classes) 的整数

    参数:
        data_dir: 数据目录路径
        batch_size: 数据加载器的批大小
        num_workers: 数据加载工作进程数
    """
```

## 常见陷阱

### 1. 在 prepare_data 中设置状态
**错误：**
```python
def prepare_data(self):
    self.dataset = load_data()  # 状态不会传递到其他进程！
```

**正确：**
```python
def prepare_data(self):
    download_data()  # 仅下载，不设置状态

def setup(self, stage):
    self.dataset = load_data()  # 在此设置状态
```

### 2. 未使用 stage 参数
**低效：**
```python
def setup(self, stage):
    self.train_dataset = load_train()
    self.val_dataset = load_val()
    self.test_dataset = load_test()  # 即使仅训练也会加载
```

**高效：**
```python
def setup(self, stage):
    if stage == 'fit':
        self.train_dataset = load_train()
        self.val_dataset = load_val()
    elif stage == 'test':
        self.test_dataset = load_test()
```

### 3. 忘记返回 DataLoader
**错误：**
```python
def train_dataloader(self):
    DataLoader(self.train_dataset, ...)  # 忘记返回！
```

**正确：**
```python
def train_dataloader(self):
    return DataLoader(self.train_dataset, ...)
```

## 与 Trainer 集成

```python
# 初始化 DataModule
dm = MyDataModule(data_dir="./data", batch_size=64)

# 所有数据加载由 DataModule 处理
trainer = L.Trainer(max_epochs=10)
trainer.fit(model, datamodule=dm)

# DataModule 也处理验证
trainer.validate(model, datamodule=dm)

# 以及测试
trainer.test(model, datamodule=dm)

# 和预测
predictions = trainer.predict(model, datamodule=dm)
```
