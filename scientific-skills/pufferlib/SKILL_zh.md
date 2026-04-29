```markdown
---
name: pufferlib
description: 专为速度和规模优化的高性能强化学习框架。适用于需要快速并行训练、向量化环境、多智能体系统或游戏环境集成（Atari、Procgen、NetHack）的场景。相比标准实现可获得2-10倍加速。如需快速原型设计或标准算法实现（含详尽文档），请改用stable-baselines3。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# PufferLib - 高性能强化学习库

## 概述

PufferLib 是专为快速并行环境模拟和训练设计的高性能强化学习库。通过优化的向量化技术、原生多智能体支持及高效的PPO实现（PuffeRL），该库可实现每秒数百万步的训练速度。提供包含20+环境的Ocean套件，并与Gymnasium、PettingZoo及专业RL框架无缝集成。

## 适用场景

在以下场景使用本技能：
- 使用PPO在任意环境（单/多智能体）中**训练RL智能体**
- 通过PufferEnv API**创建自定义环境**
- 为并行环境模拟（向量化）**优化性能**
- **集成现有环境**（Gymnasium、PettingZoo、Atari、Procgen等）
- 使用CNN、LSTM或自定义架构**开发策略**
- 将RL扩展到每秒数百万步以**加速实验**
- 通过原生多智能体环境支持进行**多智能体RL**

## 核心能力

### 1. 高性能训练（PuffeRL）

PuffeRL是PufferLib优化的PPO+LSTM训练算法，可达100万-400万步/秒。

**快速开始训练：**
```bash
# CLI训练
puffer train procgen-coinrun --train.device cuda --train.learning-rate 3e-4

# 分布式训练
torchrun --nproc_per_node=4 train.py
```

**Python训练循环：**
```python
import pufferlib
from pufferlib import PuffeRL

# 创建向量化环境
env = pufferlib.make('procgen-coinrun', num_envs=256)

# 创建训练器
trainer = PuffeRL(
    env=env,
    policy=my_policy,
    device='cuda',
    learning_rate=3e-4,
    batch_size=32768
)

# 训练循环
for iteration in range(num_iterations):
    trainer.evaluate()  # 收集轨迹
    trainer.train()     # 批量训练
    trainer.mean_and_log()  # 记录结果
```

**完整训练指南**请阅读`references/training.md`：
- 完整训练流程与CLI选项
- 使用Protein进行超参数调优
- 分布式多GPU/多节点训练
- 日志集成（Weights & Biases, Neptune）
- 检查点与恢复训练
- 性能优化技巧
- 课程学习模式

### 2. 环境开发（PufferEnv）

通过PufferEnv API创建自定义高性能环境。

**基础环境结构：**
```python
import numpy as np
from pufferlib import PufferEnv

class MyEnvironment(PufferEnv):
    def __init__(self, buf=None):
        super().__init__(buf)

        # 定义空间
        self.observation_space = self.make_space((4,))
        self.action_space = self.make_discrete(4)

        self.reset()

    def reset(self):
        # 重置状态并返回初始观测
        return np.zeros(4, dtype=np.float32)

    def step(self, action):
        # 执行动作，计算奖励，检查终止
        obs = self._get_observation()
        reward = self._compute_reward()
        done = self._is_done()
        info = {}

        return obs, reward, done, info
```

**使用模板脚本：** `scripts/env_template.py`提供完整单/多智能体环境模板，包含：
- 不同观测空间类型（向量、图像、字典）
- 动作空间变体（离散、连续、多离散）
- 多智能体环境结构
- 测试工具

**完整环境开发**请阅读`references/environments.md`：
- PufferEnv API细节与原地操作模式
- 观测/动作空间定义
- 多智能体环境创建
- Ocean套件（20+预置环境）
- 性能优化（Python转C流程）
- 环境包装器与最佳实践
- 调试与验证技术

### 3. 向量化与性能

通过优化并行模拟实现最大吞吐量。

**向量化设置：**
```python
import pufferlib

# 自动向量化
env = pufferlib.make('environment_name', num_envs=256, num_workers=8)

# 性能基准：
# - 纯Python环境：10万-50万步/秒
# - C语言环境：1亿+步/秒
# - 含训练：40万-400万总步/秒
```

**关键优化：**
- 共享内存缓冲区实现零拷贝观测传递
- 忙等待标志替代管道/队列
- 备用环境实现异步返回
- 单工作进程多环境处理

**向量化优化**请阅读`references/vectorization.md`：
- 架构与性能特征
- 工作进程与批量大小配置
- 串行/多进程/异步模式对比
- 共享内存与零拷贝模式
- 大规模分层向量化
- 多智能体向量化策略
- 性能分析与故障排除

### 4. 策略开发

构建标准PyTorch模块策略，含可选工具。

**基础策略结构：**
```python
import torch.nn as nn
from pufferlib.pytorch import layer_init

class Policy(nn.Module):
    def __init__(self, observation_space, action_space):
        super().__init__()

        # 编码器
        self.encoder = nn.Sequential(
            layer_init(nn.Linear(obs_dim, 256)),
            nn.ReLU(),
            layer_init(nn.Linear(256, 256)),
            nn.ReLU()
        )

        # 执行者与评估者头
        self.actor = layer_init(nn.Linear(256, num_actions), std=0.01)
        self.critic = layer_init(nn.Linear(256, 1), std=1.0)

    def forward(self, observations):
        features = self.encoder(observations)
        return self.actor(features), self.critic(features)
```

**完整策略开发**请阅读`references/policies.md`：
- 图像观测的CNN策略
- 优化LSTM的循环策略（推理速度提升3倍）
- 复杂观测的多输入策略
- 连续动作策略
- 多智能体策略（共享/独立参数）
- 高级架构（注意力、残差）
- 观测归一化与梯度裁剪
- 策略调试与测试

### 5. 环境集成

无缝集成主流RL框架环境。

**Gymnasium集成：**
```python
import gymnasium as gym
import pufferlib

# 包装Gymnasium环境
gym_env = gym.make('CartPole-v1')
env = pufferlib.emulate(gym_env, num_envs=256)

# 或直接使用make
env = pufferlib.make('gym-CartPole-v1', num_envs=256)
```

**PettingZoo多智能体：**
```python
# 多智能体环境
env = pufferlib.make('pettingzoo-knights-archers-zombies', num_envs=128)
```

**支持框架：**
- Gymnasium / OpenAI Gym
- PettingZoo（并行与AEC）
- Atari (ALE)
- Procgen
- NetHack / MiniHack
- Minigrid
- Neural MMO
- Crafter
- GPUDrive
- MicroRTS
- Griddly
- 及其他...

**集成细节**请阅读`references/integration.md`：
- 各框架完整集成示例
- 自定义包装器（观测、奖励、帧堆叠、动作重复）
- 空间扁平化与还原
- 环境注册
- 兼容性模式
- 性能考量
- 集成调试

## 快速入门流程

### 训练现有环境

1. 从Ocean套件或兼容框架选择环境
2. 以`scripts/train_template.py`为起点
3. 配置任务超参数
4. 通过CLI或Python脚本运行训练
5. 使用Weights & Biases或Neptune监控
6. 参考`references/training.md`进行优化

### 创建自定义环境

1. 从`scripts/env_template.py`开始
2. 定义观测与动作空间
3. 实现`reset()`和`step()`方法
4. 本地测试环境
5. 通过`pufferlib.emulate()`或`make()`向量化
6. 参考`references/environments.md`获取高级模式
7. 按需使用`references/vectorization.md`优化

### 策略开发

1. 根据观测类型选择架构：
   - 向量观测 → MLP策略
   - 图像观测 → CNN策略
   - 序列任务 → LSTM策略
   - 复杂观测 → 多输入策略
2. 使用`layer_init`进行权重初始化
3. 遵循`references/policies.md`模式
4. 训练前在环境中测试

### 性能优化

1. 分析当前吞吐量（步/秒）
2. 检查向量化配置（num_envs, num_workers）
3. 优化环境代码（原地操作、numpy向量化）
4. 关键路径考虑C语言实现
5. 使用`references/vectorization.md`系统优化

## 资源

### scripts/

**train_template.py** - 完整训练脚本模板：
- 环境创建与配置
- 策略初始化
- 日志集成（WandB, Neptune）
- 含检查点的训练循环
- 命令行参数解析
- 多GPU分布式训练设置

**env_template.py** - 环境实现模板：
- 单智能体PufferEnv示例（网格世界）
- 多智能体PufferEnv示例（协作导航）
- 多观测/动作空间模式
- 测试工具

### references/

**training.md** - 综合训练指南：
- 训练流程与CLI选项
- 超参数配置
- 分布式训练（多GPU/多节点）
- 监控与日志
- 检查点
- Protein超参数调优
- 性能优化
- 常见训练模式
- 故障排除

**environments.md** - 环境开发指南：
- PufferEnv API与特性
- 观测/动作空间
- 多智能体环境
- Ocean套件环境
- 自定义环境开发流程
- Python转C优化路径
- 第三方环境集成
- 包装器与最佳实践
- 调试

**vectorization.md** - 向量化优化：
- 架构与关键优化
- 向量化模式（串行/多进程/异步）
- 工作进程与批量配置
- 共享内存与零拷贝模式
- 高级向量化（分层/自定义）
- 多智能体向量化
- 性能监控与分析
- 故障排除与最佳实践

**policies.md** - 策略架构指南：
- 基础策略结构
- 图像CNN策略
- 优化LSTM策略
- 多输入策略
- 连续动作策略
- 多智能体策略
- 高级架构（注意力/残差）
- 观测处理与还原
- 初始化与归一化
- 调试与测试

**integration.md** - 框架集成指南：
- Gymnasium集成
- PettingZoo集成（并行与AEC）
- 第三方环境（Procgen/NetHack/Minigrid等）
- 自定义包装器（观测/奖励/帧堆叠等）
- 空间转换与还原
- 环境注册
- 兼容性模式
- 性能考量
- 集成调试

## 成功要诀

1. **从简开始**：先使用Ocean环境或Gymnasium集成，再创建自定义环境
2. **及早分析**：初始阶段即测量步/秒以识别瓶颈
3. **善用模板**：`scripts/train_template.py`和`scripts/env_template.py`提供坚实基础
4. **按需查阅文档**：每个参考文件专注特定能力且自成体系
5. **渐进优化**：从Python开始，分析性能，再对关键路径进行C优化
6. **发挥向量化优势**：PufferLib向量化是实现高吞吐的关键
7. **监控训练**：使用WandB或Neptune跟踪实验并及早发现问题
8. **测试环境**：扩展训练前验证环境逻辑
9. **检查现有环境**：Ocean套件提供20+预置环境
10. **正确初始化**：策略始终使用`pufferlib.pytorch`的`layer_init`

## 典型用例

### 标准基准训练
```python
# Atari
env = pufferlib.make('atari-pong', num_envs=256)

# Procgen
env = pufferlib.make('procgen-coinrun', num_envs=256)

# Minigrid
env = pufferlib.make('minigrid-empty-8x8', num_envs=256)
```

### 多智能体学习
```python
# PettingZoo
env = pufferlib.make('pettingzoo-pistonball', num_envs=128)

# 所有智能体共享策略
policy = create_policy(env.observation_space, env.action_space)
trainer = PuffeRL(env=env, policy=policy)
```

### 自定义任务开发
```python
# 创建自定义环境
class MyTask(PufferEnv):
    # ... 实现环境 ...

# 向量化并训练
env = pufferlib.emulate(MyTask, num_envs=256)
trainer = PuffeRL(env=env, policy=my_policy)
```

### 高性能优化
```python
# 最大化吞吐量
env = pufferlib.make(
    'my-env',
    num_envs=1024,      # 大批量
    num_workers=16,     # 多工作进程
    envs_per_worker=64  # 单工作进程优化
)
```

## 安装

```bash
uv pip install pufferlib
```

## 文档

- 官方文档：https://puffer.ai/docs.html
- GitHub：https://github.com/PufferAI/PufferLib
- Discord：提供社区支持
```
