# PufferLib 训练指南

## 概述

PuffeRL 是 PufferLib 基于 CleanRL 的 PPO with LSTMs 实现的高性能训练算法，通过专有研究改进进行了增强。它通过优化的向量化和高效实现，达到每秒数百万步的训练速度。

## 训练工作流

### 基础训练循环

PuffeRL 训练器提供三个核心方法：

```python
# 收集环境交互数据
rollout_data = trainer.evaluate()

# 在收集的批次上训练
train_metrics = trainer.train()

# 聚合并记录结果
trainer.mean_and_log()
```

### 命令行训练

通过命令行快速开始训练：

```bash
# 基础训练
puffer train environment_name --train.device cuda --train.learning-rate 0.001

# 自定义配置
puffer train environment_name \
    --train.device cuda \
    --train.batch-size 32768 \
    --train.learning-rate 0.0003 \
    --train.num-iterations 10000
```

### Python 训练脚本

```python
import pufferlib
from pufferlib import PuffeRL

# 初始化环境
env = pufferlib.make('environment_name', num_envs=256)

# 创建训练器
trainer = PuffeRL(
    env=env,
    policy=my_policy,
    device='cuda',
    learning_rate=3e-4,
    batch_size=32768,
    n_epochs=4,
    gamma=0.99,
    gae_lambda=0.95,
    clip_coef=0.2,
    ent_coef=0.01,
    vf_coef=0.5,
    max_grad_norm=0.5
)

# 训练循环
for iteration in range(num_iterations):
    # 收集轨迹数据
    rollout_data = trainer.evaluate()

    # 在批次上训练
    train_metrics = trainer.train()

    # 记录结果
    trainer.mean_and_log()
```

## 关键训练参数

### 核心超参数

- **learning_rate**: 优化器学习率 (默认: 3e-4)
- **batch_size**: 每训练批次的步数 (默认: 32768)
- **n_epochs**: 每批次的训练轮数 (默认: 4)
- **num_envs**: 并行环境数量 (默认: 256)
- **num_steps**: 每次滚动的每环境步数 (默认: 128)

### PPO 参数

- **gamma**: 折扣因子 (默认: 0.99)
- **gae_lambda**: GAE 计算的 lambda 值 (默认: 0.95)
- **clip_coef**: PPO 裁剪系数 (默认: 0.2)
- **ent_coef**: 探索的熵系数 (默认: 0.01)
- **vf_coef**: 价值函数损失系数 (默认: 0.5)
- **max_grad_norm**: 梯度裁剪的最大范数 (默认: 0.5)

### 性能参数

- **device**: 计算设备 ('cuda' 或 'cpu')
- **compile**: 使用 torch.compile 加速训练 (默认: True)
- **num_workers**: 向量化工作进程数 (默认: auto)

## 分布式训练

### 多 GPU 训练

使用 torchrun 在多个 GPU 上进行分布式训练：

```bash
torchrun --nproc_per_node=4 train.py \
    --train.device cuda \
    --train.batch-size 131072
```

### 多节点训练

跨多个节点的分布式训练：

```bash
# 主节点 (rank 0)
torchrun --nproc_per_node=8 \
    --nnodes=4 \
    --node_rank=0 \
    --master_addr=MASTER_IP \
    --master_port=29500 \
    train.py

# 工作节点 (rank 1, 2, 3)
torchrun --nproc_per_node=8 \
    --nnodes=4 \
    --node_rank=NODE_RANK \
    --master_addr=MASTER_IP \
    --master_port=29500 \
    train.py
```

## 监控与日志

### 日志集成

PufferLib 支持多种日志后端：

#### Weights & Biases

```python
from pufferlib import WandbLogger

logger = WandbLogger(
    project='my_project',
    entity='my_team',
    name='experiment_name',
    config=trainer_config
)

trainer = PuffeRL(env, policy, logger=logger)
```

#### Neptune

```python
from pufferlib import NeptuneLogger

logger = NeptuneLogger(
    project='my_team/my_project',
    name='experiment_name',
    api_token='YOUR_TOKEN'
)

trainer = PuffeRL(env, policy, logger=logger)
```

#### 无日志

```python
from pufferlib import NoLogger

trainer = PuffeRL(env, policy, logger=NoLogger())
```

### 关键指标

训练日志包含：

- **性能指标**:
  - 每秒步数 (SPS)
  - 训练吞吐量
  - 每次迭代的挂钟时间

- **学习指标**:
  - 回合奖励 (均值/最小值/最大值)
  - 回合长度
  - 价值函数损失
  - 策略损失
  - 熵值
  - 解释方差
  - 裁剪比例

- **环境指标**:
  - 环境特定奖励
  - 成功率
  - 自定义指标

### 终端仪表盘

PufferLib 提供实时终端仪表盘显示：
- 训练进度
- 当前 SPS
- 回合统计数据
- 损失值
- GPU 利用率

## 检查点

### 保存检查点

```python
# 保存检查点
trainer.save_checkpoint('checkpoint.pt')

# 保存额外元数据
trainer.save_checkpoint(
    'checkpoint.pt',
    metadata={'iteration': iteration, 'best_reward': best_reward}
)
```

### 加载检查点

```python
# 加载检查点
trainer.load_checkpoint('checkpoint.pt')

# 恢复训练
for iteration in range(resume_iteration, num_iterations):
    trainer.evaluate()
    trainer.train()
    trainer.mean_and_log()
```

## 使用 Protein 进行超参数调优

Protein 系统支持自动超参数和奖励调优：

```python
from pufferlib import Protein

# 定义搜索空间
search_space = {
    'learning_rate': [1e-4, 3e-4, 1e-3],
    'batch_size': [16384, 32768, 65536],
    'ent_coef': [0.001, 0.01, 0.1],
    'clip_coef': [0.1, 0.2, 0.3]
}

# 运行超参数搜索
protein = Protein(
    env_name='environment_name',
    search_space=search_space,
    num_trials=100,
    metric='mean_reward'
)

best_config = protein.optimize()
```

## 性能优化技巧

### 最大化吞吐量

1. **批次大小**: 增加 batch_size 以充分利用 GPU
2. **环境数量**: 平衡 CPU 和 GPU 利用率
3. **编译**: 启用 torch.compile 获得 10-20% 加速
4. **工作进程**: 根据环境复杂度调整 num_workers
5. **设备**: 神经网络训练始终使用 'cuda'

### 环境速度

- 纯 Python 环境: ~100k-500k SPS
- C 语言环境: ~4M SPS
- 含训练开销: ~1M-4M 总 SPS

### 内存管理

- GPU 内存不足时减少 batch_size
- CPU 内存不足时减少 num_envs
- 大有效批次时使用梯度累积

## 常见训练模式

### 课程学习

```python
# 从简单任务开始，逐步增加难度
difficulty_levels = [0.1, 0.3, 0.5, 0.7, 1.0]

for difficulty in difficulty_levels:
    env = pufferlib.make('environment_name', difficulty=difficulty)
    trainer = PuffeRL(env, policy)

    for iteration in range(iterations_per_level):
        trainer.evaluate()
        trainer.train()
        trainer.mean_and_log()
```

### 奖励塑形

```python
# 使用自定义奖励塑形封装环境
class RewardShapedEnv(pufferlib.PufferEnv):
    def step(self, actions):
        obs, rewards, dones, infos = super().step(actions)

        # 添加塑形奖励
        shaped_rewards = rewards + 0.1 * proximity_bonus

        return obs, shaped_rewards, dones, infos
```

### 多阶段训练

```python
# 使用不同配置进行多阶段训练
stages = [
    {'learning_rate': 1e-3, 'iterations': 1000},   # 探索阶段
    {'learning_rate': 3e-4, 'iterations': 5000},   # 主训练阶段
    {'learning_rate': 1e-4, 'iterations': 2000}    # 微调阶段
]

for stage in stages:
    trainer.learning_rate = stage['learning_rate']
    for iteration in range(stage['iterations']):
        trainer.evaluate()
        trainer.train()
        trainer.mean_and_log()
```

## 故障排除

### 性能低下

- 检查环境是否正确向量化
- 使用 `nvidia-smi` 验证 GPU 利用率
- 增加 batch_size 以饱和 GPU
- 启用编译模式
- 使用 `torch.profiler` 进行性能分析

### 训练不稳定

- 降低 learning_rate
- 减小 batch_size
- 增加 num_envs 获取更多样样本
- 添加熵系数增强探索
- 检查奖励缩放比例

### 内存问题

- 减少 batch_size 或 num_envs
- 使用梯度累积
- 如遇 OOM 错误则禁用编译模式
- 检查自定义环境中的内存泄漏
