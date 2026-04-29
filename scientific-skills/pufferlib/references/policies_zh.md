# PufferLib 策略指南

## 概述

PufferLib 策略是标准的 PyTorch 模块，提供可选的观测值处理和 LSTM 集成工具。该框架提供默认架构和工具，同时允许策略设计的完全灵活性。

## 策略架构

### 基础策略结构

```python
import torch
import torch.nn as nn
from pufferlib.pytorch import layer_init

class BasicPolicy(nn.Module):
    def __init__(self, observation_space, action_space):
        super().__init__()

        self.observation_space = observation_space
        self.action_space = action_space

        # 编码器网络
        self.encoder = nn.Sequential(
            layer_init(nn.Linear(observation_space.shape[0], 256)),
            nn.ReLU(),
            layer_init(nn.Linear(256, 256)),
            nn.ReLU()
        )

        # 策略头（执行器）
        self.actor = layer_init(nn.Linear(256, action_space.n), std=0.01)

        # 价值头（评价器）
        self.critic = layer_init(nn.Linear(256, 1), std=1.0)

    def forward(self, observations):
        """前向传播"""
        # 编码观测值
        features = self.encoder(observations)

        # 获取动作逻辑值和状态价值
        logits = self.actor(features)
        value = self.critic(features)

        return logits, value

    def get_action(self, observations, deterministic=False):
        """从策略中采样动作"""
        logits, value = self.forward(observations)

        if deterministic:
            action = logits.argmax(dim=-1)
        else:
            dist = torch.distributions.Categorical(logits=logits)
            action = dist.sample()

        return action, value
```

### 层初始化

PufferLib 提供 `layer_init` 实现正确的权重初始化：

```python
from pufferlib.pytorch import layer_init

# 默认正交初始化
layer = layer_init(nn.Linear(256, 256))

# 自定义标准差
actor_head = layer_init(nn.Linear(256, num_actions), std=0.01)
critic_head = layer_init(nn.Linear(256, 1), std=1.0)

# 适用于任意层类型
conv = layer_init(nn.Conv2d(3, 32, kernel_size=8, stride=4))
```

## CNN 策略

针对基于图像的观测值：

```python
class CNNPolicy(nn.Module):
    def __init__(self, observation_space, action_space):
        super().__init__()

        # 图像编码器
        self.encoder = nn.Sequential(
            layer_init(nn.Conv2d(3, 32, kernel_size=8, stride=4)),
            nn.ReLU(),
            layer_init(nn.Conv2d(32, 64, kernel_size=4, stride=2)),
            nn.ReLU(),
            layer_init(nn.Conv2d(64, 64, kernel_size=3, stride=1)),
            nn.ReLU(),
            nn.Flatten(),
            layer_init(nn.Linear(64 * 7 * 7, 512)),
            nn.ReLU()
        )

        self.actor = layer_init(nn.Linear(512, action_space.n), std=0.01)
        self.critic = layer_init(nn.Linear(512, 1), std=1.0)

    def forward(self, observations):
        # 归一化像素值
        x = observations.float() / 255.0

        features = self.encoder(x)
        logits = self.actor(features)
        value = self.critic(features)

        return logits, value
```

### 高效 CNN 架构

```python
class EfficientCNN(nn.Module):
    """针对Atari风格游戏的优化CNN"""

    def __init__(self, observation_space, action_space):
        super().__init__()

        in_channels = observation_space.shape[0]  # 通常为4（帧堆叠）

        self.network = nn.Sequential(
            layer_init(nn.Conv2d(in_channels, 32, 8, stride=4)),
            nn.ReLU(),
            layer_init(nn.Conv2d(32, 64, 4, stride=2)),
            nn.ReLU(),
            layer_init(nn.Conv2d(64, 64, 3, stride=1)),
            nn.ReLU(),
            nn.Flatten()
        )

        # 计算特征尺寸
        with torch.no_grad():
            sample = torch.zeros(1, *observation_space.shape)
            n_features = self.network(sample).shape[1]

        self.fc = layer_init(nn.Linear(n_features, 512))
        self.actor = layer_init(nn.Linear(512, action_space.n), std=0.01)
        self.critic = layer_init(nn.Linear(512, 1), std=1.0)

    def forward(self, x):
        x = x.float() / 255.0
        x = self.network(x)
        x = torch.relu(self.fc(x))

        return self.actor(x), self.critic(x)
```

## 循环策略 (LSTM)

PufferLib 提供优化的 LSTM 集成与自动循环处理：

```python
from pufferlib.pytorch import LSTMWrapper

class RecurrentPolicy(nn.Module):
    def __init__(self, observation_space, action_space, hidden_size=256):
        super().__init__()

        # 观测值编码器
        self.encoder = nn.Sequential(
            layer_init(nn.Linear(observation_space.shape[0], 128)),
            nn.ReLU()
        )

        # LSTM层
        self.lstm = nn.LSTM(128, hidden_size, num_layers=1)

        # 策略头与价值头
        self.actor = layer_init(nn.Linear(hidden_size, action_space.n), std=0.01)
        self.critic = layer_init(nn.Linear(hidden_size, 1), std=1.0)

        # 隐藏状态
        self.hidden_size = hidden_size

    def forward(self, observations, state=None):
        """
        参数:
            observations: (批次大小, 观测维度)
            state: 可选的LSTM状态元组(h, c)

        返回:
            逻辑值, 状态价值, 新状态
        """
        batch_size = observations.shape[0]

        # 编码观测值
        features = self.encoder(observations)

        # 初始化隐藏状态
        if state is None:
            h = torch.zeros(1, batch_size, self.hidden_size, device=features.device)
            c = torch.zeros(1, batch_size, self.hidden_size, device=features.device)
            state = (h, c)

        # LSTM前向传播
        features = features.unsqueeze(0)  # 添加序列维度
        lstm_out, new_state = self.lstm(features, state)
        lstm_out = lstm_out.squeeze(0)

        # 获取输出
        logits = self.actor(lstm_out)
        value = self.critic(lstm_out)

        return logits, value, new_state
```

### LSTM 优化

PufferLib 的 LSTM 优化在推演时使用 LSTMCell，训练时使用 LSTM，实现高达3倍的推理加速：

```python
class OptimizedLSTMPolicy(nn.Module):
    def __init__(self, observation_space, action_space, hidden_size=256):
        super().__init__()

        self.encoder = nn.Sequential(
            layer_init(nn.Linear(observation_space.shape[0], 128)),
            nn.ReLU()
        )

        # 使用LSTMCell进行逐步推理
        self.lstm_cell = nn.LSTMCell(128, hidden_size)

        # 使用LSTM进行批量训练
        self.lstm = nn.LSTM(128, hidden_size, num_layers=1)

        self.actor = layer_init(nn.Linear(hidden_size, action_space.n), std=0.01)
        self.critic = layer_init(nn.Linear(hidden_size, 1), std=1.0)

        self.hidden_size = hidden_size

    def encode_observations(self, observations, state):
        """使用LSTMCell进行快速推理"""
        features = self.encoder(observations)

        if state is None:
            h = torch.zeros(observations.shape[0], self.hidden_size, device=features.device)
            c = torch.zeros(observations.shape[0], self.hidden_size, device=features.device)
        else:
            h, c = state

        # 使用LSTMCell逐步处理（推理更快）
        h, c = self.lstm_cell(features, (h, c))

        logits = self.actor(h)
        value = self.critic(h)

        return logits, value, (h, c)

    def decode_actions(self, observations, actions, state):
        """使用LSTM进行批量训练"""
        seq_len, batch_size = observations.shape[:2]

        # 为LSTM重塑张量
        obs_flat = observations.reshape(seq_len * batch_size, -1)
        features = self.encoder(obs_flat)
        features = features.reshape(seq_len, batch_size, -1)

        if state is None:
            h = torch.zeros(1, batch_size, self.hidden_size, device=features.device)
            c = torch.zeros(1, batch_size, self.hidden_size, device=features.device)
            state = (h, c)

        # 使用LSTM批量处理（训练更快）
        lstm_out, new_state = self.lstm(features, state)

        # 展平回原始形状
        lstm_out = lstm_out.reshape(seq_len * batch_size, -1)

        logits = self.actor(lstm_out)
        value = self.critic(lstm_out)

        return logits, value, new_state
```

## 多输入策略

针对多种观测类型的场景：

```python
class MultiInputPolicy(nn.Module):
    def __init__(self, observation_space, action_space):
        super().__init__()

        # 不同观测类型的独立编码器
        self.image_encoder = nn.Sequential(
            layer_init(nn.Conv2d(3, 32, 8, stride=4)),
            nn.ReLU(),
            layer_init(nn.Conv2d(32, 64, 4, stride=2)),
            nn.ReLU(),
            nn.Flatten()
        )

        self.vector_encoder = nn.Sequential(
            layer_init(nn.Linear(observation_space['vector'].shape[0], 128)),
            nn.ReLU()
        )

        # 合并特征
        combined_size = 64 * 9 * 9 + 128  # 图像特征 + 向量特征
        self.combiner = nn.Sequential(
            layer_init(nn.Linear(combined_size, 512)),
            nn.ReLU()
        )

        self.actor = layer_init(nn.Linear(512, action_space.n), std=0.01)
        self.critic = layer_init(nn.Linear(512, 1), std=1.0)

    def forward(self, observations):
        # 处理每种观测类型
        image_features = self.image_encoder(observations['image'].float() / 255.0)
        vector_features = self.vector_encoder(observations['vector'])

        # 合并特征
        combined = torch.cat([image_features, vector_features], dim=-1)
        features = self.combiner(combined)

        return self.actor(features), self.critic(features)
```

## 连续动作策略

针对连续控制任务：

```python
class ContinuousPolicy(nn.Module):
    def __init__(self, observation_space, action_space):
        super().__init__()

        self.encoder = nn.Sequential(
            layer_init(nn.Linear(observation_space.shape[0], 256)),
            nn.ReLU(),
            layer_init(nn.Linear(256, 256)),
            nn.ReLU()
        )

        # 动作分布均值
        self.actor_mean = layer_init(nn.Linear(256, action_space.shape[0]), std=0.01)

        # 动作分布对数标准差
        self.actor_logstd = nn.Parameter(torch.zeros(1, action_space.shape[0]))

        # 价值头
        self.critic = layer_init(nn.Linear(256, 1), std=1.0)

    def forward(self, observations):
        features = self.encoder(observations)

        action_mean = self.actor_mean(features)
        action_std = torch.exp(self.actor_logstd)

        value = self.critic(features)

        return action_mean, action_std, value

    def get_action(self, observations, deterministic=False):
        action_mean, action_std, value = self.forward(observations)

        if deterministic:
            return action_mean, value
        else:
            dist = torch.distributions.Normal(action_mean, action_std)
            action = dist.sample()
            return torch.tanh(action), value  # 将动作限制在[-1, 1]区间
```

## 观测值处理

PufferLib 提供观测值解压工具：

```python
from pufferlib.pytorch import unflatten_observations

class PolicyWithUnflatten(nn.Module):
    def __init__(self, observation_space, action_space):
        super().__init__()

        self.observation_space = observation_space

        # 为每个观测组件定义编码器
        self.encoders = nn.ModuleDict({
            'image': self._make_image_encoder(),
            'vector': self._make_vector_encoder()
        })

        # ... 策略其余部分 ...

    def forward(self, flat_observations):
        # 将展平的观测值解压为结构化格式
        observations = unflatten_observations(
            flat_observations,
            self.observation_space
        )

        # 处理每个组件
        image_features = self.encoders['image'](observations['image'])
        vector_features = self.encoders['vector'](observations['vector'])

        # 合并并继续处理...
```

## 多智能体策略

### 共享参数

所有智能体使用相同策略：

```python
class SharedMultiAgentPolicy(nn.Module):
    def __init__(self, observation_space, action_space, num_agents):
        super().__init__()

        self.num_agents = num_agents

        # 所有智能体共享单一策略
        self.encoder = nn.Sequential(
            layer_init(nn.Linear(observation_space.shape[0], 256)),
            nn.ReLU()
        )

        self.actor = layer_init(nn.Linear(256, action_space.n), std=0.01)
        self.critic = layer_init(nn.Linear(256, 1), std=1.0)

    def forward(self, observations):
        """
        参数:
            observations: (批次大小 * 智能体数量, 观测维度)
        返回:
            逻辑值: (批次大小 * 智能体数量, 动作数量)
            状态价值: (批次大小 * 智能体数量, 1)
        """
        features = self.encoder(observations)
        return self.actor(features), self.critic(features)
```

### 独立参数

每个智能体拥有独立策略：

```python
class IndependentMultiAgentPolicy(nn.Module):
    def __init__(self, observation_space, action_space, num_agents):
        super().__init__()

        self.num_agents = num_agents

        # 为每个智能体创建独立策略
        self.policies = nn.ModuleList([
            self._make_policy(observation_space, action_space)
            for _ in range(num_agents)
        ])

    def _make_policy(self, observation_space, action_space):
        return nn.Sequential(
            layer_init(nn.Linear(observation_space.shape[0], 256)),
            nn.ReLU(),
            layer_init(nn.Linear(256, 256)),
            nn.ReLU()
        )

    def forward(self, observations, agent_ids):
        """
        参数:
            observations: (批次大小, 观测维度)
            agent_ids: (批次大小,) 每个观测值所属的智能体ID
        """
        outputs = []
        for agent_id in range(self.num_agents):
            mask = agent_ids == agent_id
            if mask.any():
                agent_obs = observations[mask]
                agent_out = self.policies[agent_id](agent_obs)
                outputs.append(agent_out)

        return torch.cat(outputs, dim=0)
```

## 高级架构

### 基于注意力的策略

```python
class AttentionPolicy(nn.Module):
    def __init__(self, observation_space, action_space, d_model=

```python
return x + self.block(x)

class ResidualPolicy(nn.Module):
    def __init__(self, observation_space, action_space, num_blocks=4):
        super().__init__()

        dim = 256

        self.encoder = layer_init(nn.Linear(observation_space.shape[0], dim))

        self.blocks = nn.Sequential(
            *[ResidualBlock(dim) for _ in range(num_blocks)]
        )

        self.actor = layer_init(nn.Linear(dim, action_space.n), std=0.01)
        self.critic = layer_init(nn.Linear(dim, 1), std=1.0)

    def forward(self, observations):
        x = torch.relu(self.encoder(observations))
        x = self.blocks(x)
        return self.actor(x), self.critic(x)
```

## 策略最佳实践

### 初始化

```python
# 始终使用 layer_init 进行正确初始化
good_layer = layer_init(nn.Linear(256, 256))

# 对执行器头部使用小标准差（训练初期更稳定）
actor = layer_init(nn.Linear(256, num_actions), std=0.01)

# 对评价器头部使用 std=1.0
critic = layer_init(nn.Linear(256, 1), std=1.0)
```

### 观测值归一化

```python
class NormalizedPolicy(nn.Module):
    def __init__(self, observation_space, action_space):
        super().__init__()

        # 用于归一化的运行统计量
        self.obs_mean = nn.Parameter(torch.zeros(observation_space.shape[0]), requires_grad=False)
        self.obs_std = nn.Parameter(torch.ones(observation_space.shape[0]), requires_grad=False)

        # ... 策略其余部分 ...

    def forward(self, observations):
        # 归一化观测值
        normalized_obs = (observations - self.obs_mean) / (self.obs_std + 1e-8)

        # 使用归一化观测值继续处理
        return self.policy(normalized_obs)

    def update_normalization(self, observations):
        """更新运行统计量"""
        self.obs_mean.data = observations.mean(dim=0)
        self.obs_std.data = observations.std(dim=0)
```

### 梯度裁剪

```python
# PufferLib 训练器自动处理梯度裁剪
trainer = PuffeRL(
    env=env,
    policy=policy,
    max_grad_norm=0.5  # 将梯度裁剪至此范数
)
```

### 模型编译

```python
# 启用 torch.compile 加速训练（PyTorch 2.0+）
policy = MyPolicy(observation_space, action_space)

# 编译模型
policy = torch.compile(policy, mode='reduce-overhead')

# 与训练器配合使用
trainer = PuffeRL(env=env, policy=policy, compile=True)
```

## 策略调试

### 检查输出形状

```python
def test_policy_shapes(policy, observation_space, batch_size=32):
    """验证策略输出形状"""
    # 创建虚拟观测值
    obs = torch.randn(batch_size, *observation_space.shape)

    # 前向传播
    logits, value = policy(obs)

    # 检查形状
    assert logits.shape == (batch_size, policy.action_space.n)
    assert value.shape == (batch_size, 1)

    print("✓ 策略形状正确")
```

### 验证梯度

```python
def check_gradients(policy, observation_space):
    """检查梯度是否正常流动"""
    obs = torch.randn(1, *observation_space.shape, requires_grad=True)

    logits, value = policy(obs)

    # 反向传播
    loss = logits.sum() + value.sum()
    loss.backward()

    # 检查梯度存在性
    for name, param in policy.named_parameters():
        if param.grad is None:
            print(f"⚠ {name} 无梯度")
        elif torch.isnan(param.grad).any():
            print(f"⚠ {name} 存在 NaN 梯度")
        else:
            print(f"✓ {name} 梯度正常")
```
