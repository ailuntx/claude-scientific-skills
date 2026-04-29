# 分布式训练 - 全面指南

## 概述

PyTorch Lightning 提供了多种策略，用于在多个 GPU、节点和机器上高效训练大型模型。根据模型规模和硬件配置选择合适的策略。

## 策略选择指南

### 适用场景

**常规训练（单设备）**
- 模型规模：可放入单个 GPU 显存的任意大小
- 适用场景：原型开发、小型模型、调试

**DDP（分布式数据并行）**
- 模型规模：<5亿参数（例如 ResNet50 ~8000万参数）
- 适用条件：权重、激活值、优化器状态和梯度均可放入 GPU 显存
- 目标：跨多个 GPU 扩展批大小和训练速度
- 最佳场景：大多数标准深度学习模型

**FSDP（全分片数据并行）**
- 模型规模：5亿+参数（例如 BERT-Large、GPT 等大型 Transformer）
- 适用条件：模型无法放入单个 GPU 显存
- 推荐对象：模型并行新手或从 DDP 迁移的用户
- 特性：激活值检查点、CPU 参数卸载

**DeepSpeed**
- 模型规模：5亿+参数
- 适用条件：需要前沿特性或已熟悉 DeepSpeed
- 特性：CPU/磁盘参数卸载、分布式检查点、细粒度控制
- 权衡：配置更复杂

## DDP（分布式数据并行）

### 基础用法

```python
# 单 GPU
trainer = L.Trainer(accelerator="gpu", devices=1)

# 单节点多 GPU（自动 DDP）
trainer = L.Trainer(accelerator="gpu", devices=4)

# 显式 DDP 策略
trainer = L.Trainer(strategy="ddp", accelerator="gpu", devices=4)
```

### 多节点 DDP

```python
# 在每个节点上运行：
trainer = L.Trainer(
    strategy="ddp",
    accelerator="gpu",
    devices=4,  # 每节点 GPU 数量
    num_nodes=4  # 总节点数
)
```

### DDP 配置

```python
from lightning.pytorch.strategies import DDPStrategy

trainer = L.Trainer(
    strategy=DDPStrategy(
        process_group_backend="nccl",  # GPU 用 "nccl"，CPU 用 "gloo"
        find_unused_parameters=False,   # 模型存在未使用参数时设为 True
        gradient_as_bucket_view=True    # 内存效率更高
    ),
    accelerator="gpu",
    devices=4
)
```

### DDP Spawn

当 `ddp` 引发问题时使用（速度较慢但兼容性更好）：

```python
trainer = L.Trainer(strategy="ddp_spawn", accelerator="gpu", devices=4)
```

### DDP 最佳实践

1. **批大小：** 乘以 GPU 数量
   ```python
   # 使用 4 个 GPU 时，有效批大小 = batch_size * 4
   dm = MyDataModule(batch_size=32)  # 32 * 4 = 128 有效批大小
   ```

2. **学习率：** 通常随批大小缩放
   ```python
   # 线性缩放规则
   base_lr = 0.001
   num_gpus = 4
   lr = base_lr * num_gpus
   ```

3. **同步：** 指标计算使用 `sync_dist=True`
   ```python
   self.log("val_loss", loss, sync_dist=True)
   ```

4. **特定秩操作：** 使用装饰器限定主进程
   ```python
   from lightning.pytorch.utilities import rank_zero_only

   @rank_zero_only
   def save_results(self):
       # 仅在主进程（rank 0）运行
       torch.save(self.results, "results.pt")
   ```

## FSDP（全分片数据并行）

### 基础用法

```python
trainer = L.Trainer(
    strategy="fsdp",
    accelerator="gpu",
    devices=4
)
```

### FSDP 配置

```python
from lightning.pytorch.strategies import FSDPStrategy
import torch.nn as nn

trainer = L.Trainer(
    strategy=FSDPStrategy(
        # 分片策略
        sharding_strategy="FULL_SHARD",  # 可选 "SHARD_GRAD_OP", "NO_SHARD", "HYBRID_SHARD"

        # 激活值检查点（节省内存）
        activation_checkpointing_policy={nn.TransformerEncoderLayer},

        # CPU 卸载（节省 GPU 显存，速度较慢）
        cpu_offload=False,

        # 混合精度
        mixed_precision=True,

        # 封装策略（自动封装层）
        auto_wrap_policy=None
    ),
    accelerator="gpu",
    devices=8,
    precision="bf16-mixed"
)
```

### 分片策略

**FULL_SHARD（默认）**
- 分片优化器状态、梯度和参数
- 最大内存节省
- 通信开销更高

**SHARD_GRAD_OP**
- 仅分片优化器状态和梯度
- 参数保留在所有设备
- 内存节省较少但速度更快

**NO_SHARD**
- 不分片（等效于 DDP）
- 用于对比或无需分片场景

**HYBRID_SHARD**
- 节点内使用 FULL_SHARD，跨节点使用 NO_SHARD
- 适合多节点设置

### 激活值检查点

用计算量换取内存：

```python
from lightning.pytorch.strategies import FSDPStrategy
import torch.nn as nn

# 检查点特定层类型
trainer = L.Trainer(
    strategy=FSDPStrategy(
        activation_checkpointing_policy={
            nn.TransformerEncoderLayer,
            nn.TransformerDecoderLayer
        }
    )
)
```

### CPU 卸载

将未使用参数卸载到 CPU：

```python
trainer = L.Trainer(
    strategy=FSDPStrategy(
        cpu_offload=True  # 速度较慢但节省 GPU 显存
    ),
    accelerator="gpu",
    devices=4
)
```

### 大型模型 FSDP 应用

```python
from lightning.pytorch.strategies import FSDPStrategy
import torch.nn as nn

class LargeTransformer(L.LightningModule):
    def __init__(self):
        super().__init__()
        self.transformer = nn.TransformerEncoder(
            nn.TransformerEncoderLayer(d_model=4096, nhead=32),
            num_layers=48
        )

    def configure_sharded_model(self):
        # FSDP 调用此方法封装模型
        pass

# 训练
trainer = L.Trainer(
    strategy=FSDPStrategy(
        activation_checkpointing_policy={nn.TransformerEncoderLayer},
        cpu_offload=False,
        sharding_strategy="FULL_SHARD"
    ),
    accelerator="gpu",
    devices=8,
    precision="bf16-mixed",
    max_epochs=10
)

model = LargeTransformer()
trainer.fit(model, datamodule=dm)
```

## DeepSpeed

### 安装

```bash
pip install deepspeed
```

### 基础用法

```python
trainer = L.Trainer(
    strategy="deepspeed_stage_2",  # 或 "deepspeed_stage_3"
    accelerator="gpu",
    devices=4,
    precision="16-mixed"
)
```

### DeepSpeed 阶段

**阶段 1：优化器状态分片**
- 分片优化器状态
- 中等内存节省

```python
trainer = L.Trainer(strategy="deepspeed_stage_1")
```

**阶段 2：优化器+梯度分片**
- 分片优化器状态和梯度
- 良好内存节省

```python
trainer = L.Trainer(strategy="deepspeed_stage_2")
```

**阶段 3：全模型分片（ZeRO-3）**
- 分片优化器状态、梯度和模型参数
- 最大内存节省
- 可训练超大型模型

```python
trainer = L.Trainer(strategy="deepspeed_stage_3")
```

**带卸载的阶段 2**
- 卸载到 CPU 或 NVMe

```python
trainer = L.Trainer(strategy="deepspeed_stage_2_offload")
trainer = L.Trainer(strategy="deepspeed_stage_3_offload")
```

### DeepSpeed 配置文件

细粒度控制配置：

```python
from lightning.pytorch.strategies import DeepSpeedStrategy

# 创建配置文件：ds_config.json
config = {
    "zero_optimization": {
        "stage": 3,
        "offload_optimizer": {
            "device": "cpu",
            "pin_memory": True
        },
        "offload_param": {
            "device": "cpu",
            "pin_memory": True
        },
        "overlap_comm": True,
        "contiguous_gradients": True,
        "sub_group_size": 1e9,
        "reduce_bucket_size": "auto",
        "stage3_prefetch_bucket_size": "auto",
        "stage3_param_persistence_threshold": "auto",
        "stage3_max_live_parameters": 1e9,
        "stage3_max_reuse_distance": 1e9
    },
    "fp16": {
        "enabled": True,
        "loss_scale": 0,
        "initial_scale_power": 16,
        "loss_scale_window": 1000,
        "hysteresis": 2,
        "min_loss_scale": 1
    },
    "gradient_clipping": 1.0,
    "train_batch_size": "auto",
    "train_micro_batch_size_per_gpu": "auto"
}

trainer = L.Trainer(
    strategy=DeepSpeedStrategy(config=config),
    accelerator="gpu",
    devices=8,
    precision="16-mixed"
)
```

### DeepSpeed 最佳实践

1. **<100亿参数模型使用阶段 2**
2. **>100亿参数模型使用阶段 3**
3. **GPU 显存不足时启用卸载**
4. **调整 `reduce_bucket_size` 提升通信效率**

## 对比表格

| 特性 | DDP | FSDP | DeepSpeed |
|------|-----|------|-----------|
| 模型规模 | <5亿参数 | 5亿+参数 | 5亿+参数 |
| 内存效率 | 低 | 高 | 极高 |
| 速度 | 最快 | 快 | 快 |
| 配置复杂度 | 简单 | 中等 | 复杂 |
| 卸载支持 | 无 | CPU | CPU + 磁盘 |
| 最佳场景 | 标准模型 | 大型模型 | 超大型模型 |
| 配置需求 | 最小 | 中等 | 复杂 |

## 混合精度训练

使用混合精度加速训练并节省内存：

```python
# FP16 混合精度
trainer = L.Trainer(precision="16-mixed")

# BFloat16 混合精度（A100, H100）
trainer = L.Trainer(precision="bf16-mixed")

# 全精度（默认）
trainer = L.Trainer(precision="32-true")

# 双精度
trainer = L.Trainer(precision="64-true")
```

### 不同策略的混合精度应用

```python
# DDP + FP16
trainer = L.Trainer(
    strategy="ddp",
    accelerator="gpu",
    devices=4,
    precision="16-mixed"
)

# FSDP + BFloat16
trainer = L.Trainer(
    strategy="fsdp",
    accelerator="gpu",
    devices=8,
    precision="bf16-mixed"
)

# DeepSpeed + FP16
trainer = L.Trainer(
    strategy="deepspeed_stage_2",
    accelerator="gpu",
    devices=4,
    precision="16-mixed"
)
```

## 多节点训练

### SLURM 集群

```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --gpus-per-node=4
#SBATCH --time=24:00:00

srun python train.py
```

```python
# train.py
trainer = L.Trainer(
    strategy="ddp",
    accelerator="gpu",
    devices=4,
    num_nodes=4
)
```

### 手动多节点配置

节点 0（主节点）：
```bash
python train.py --num_nodes=2 --node_rank=0 --master_addr=192.168.1.1 --master_port=12345
```

节点 1：
```bash
python train.py --num_nodes=2 --node_rank=1 --master_addr=192.168.1.1 --master_port=12345
```

```python
# train.py
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--num_nodes", type=int, default=1)
parser.add_argument("--node_rank", type=int, default=0)
parser.add_argument("--master_addr", type=str, default="localhost")
parser.add_argument("--master_port", type=int, default=12345)
args = parser.parse_args()

trainer = L.Trainer(
    strategy="ddp",
    accelerator="gpu",
    devices=4,
    num_nodes=args.num_nodes
)
```

## 通用模式

### DDP 梯度累积

```python
# 模拟更大批大小
trainer = L.Trainer(
    strategy="ddp",
    accelerator="gpu",
    devices=4,
    accumulate_grad_batches=4  # 有效批大小 = batch_size * devices * 4
)
```

### 分布式训练模型检查点

```python
from lightning.pytorch.callbacks import ModelCheckpoint

checkpoint_callback = ModelCheckpoint(
    monitor="val_loss",
    save_top_k=3,
    mode="min"
)

trainer = L.Trainer(
    strategy="ddp",
    accelerator="gpu",
    devices=4,
    callbacks=[checkpoint_callback]
)
```

### 分布式训练可复现性

```python
import lightning as L

L.seed_everything(42, workers=True)

trainer = L.Trainer(
    strategy="ddp",
    accelerator="gpu",
    devices=4,
    deterministic=True
)
```

## 故障排除

### NCCL 超时

为慢速网络增加超时时间：

```python
import os
os.environ["NCCL_TIMEOUT"] = "3600"  # 1 小时

trainer = L.Trainer(strategy="ddp", accelerator="gpu", devices=4)
```

### CUDA 内存不足

解决方案：
1. 启用梯度检查点
2. 减小批大小
3. 使用 FSDP 或 DeepSpeed
4. 启用 CPU 卸载
5. 使用混合精度

```python
# 方案 1：梯度检查点
class MyModel(L.LightningModule):
    def __init__(self):
        super().__init__()
        self.model = MyTransformer()
        self.model.gradient_checkpointing_enable()

# 方案 2：减小批大小
dm = MyDataModule(batch_size=16)  # 从 32 减小

# 方案 3：带卸载的 FSDP
trainer = L.Trainer(
    strategy=FSDPStrategy(cpu_offload=True),
    precision="bf16-mixed"
)

# 方案 4：梯度累积
trainer = L.Trainer(accumulate_grad_batches=4)
```

### 分布式采样器问题

Lightning 自动处理 DistributedSampler：

```python
# 避免手动操作
from torch.utils.data import DistributedSampler
sampler = DistributedSampler(dataset)  # Lightning 已自动处理

# 只需设置 shuffle
train_loader = DataLoader(dataset, batch_size=32, shuffle=True)
```

### 通信开销

通过 `find_unused_parameters` 减少通信：

```python
trainer = L.Trainer(
    strategy=DDPStrategy(find_unused_parameters=False),
    accelerator="gpu",
    devices=4
)
```

## 最佳实践

### 1. 从单 GPU 开始
扩展前先在单 GPU 测试代码：

```python
# 单 GPU 调试
trainer = L.Trainer(accelerator="gpu", devices=1, fast_dev_run=True)

# 再扩展到多 GPU
trainer = L.Trainer(accelerator="gpu", devices=4, strategy="ddp")
```

### 2. 选择合适策略
- <5亿参数：使用 DDP
- 5亿-100亿参数：使用 FSDP
- >100亿参数：使用 DeepSpeed 阶段 3

### 3. 启用混合精度
现代 GPU 始终使用混合精度：

```python
trainer = L.Trainer(precision="bf16-mixed")  # A100, H100
trainer = L.Trainer(precision="16-mixed")    # V100, T4
```

### 4. 缩放超参数
扩展时调整学习率和批大小：

```python
# 线性缩放规则
lr = base_lr * num_gpus
```

### 5. 同步指标
分布式训练始终同步指标：

```python
self.log("val_loss", loss, sync_dist=True)
```

### 6. 使用 Rank

保存检查点以便从故障中恢复：

```python
checkpoint_callback = ModelCheckpoint(
    save_top_k=3,
    save_last=True,  # 始终保存最后一个以便恢复
    every_n_epochs=5
)
```
