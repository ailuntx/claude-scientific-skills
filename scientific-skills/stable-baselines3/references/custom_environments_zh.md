# 为 Stable Baselines3 创建自定义环境

本指南提供创建与 Stable Baselines3 兼容的自定义 Gymnasium 环境的全面信息。

## 环境结构

### 必需方法

每个自定义环境必须继承自 `gymnasium.Env` 并实现：

```python
import gymnasium as gym
from gymnasium import spaces
import numpy as np

class CustomEnv(gym.Env):
    def __init__(self):
        """初始化环境，定义 action_space 和 observation_space"""
        super().__init__()
        self.action_space = spaces.Discrete(4)
        self.observation_space = spaces.Box(low=0, high=1, shape=(4,), dtype=np.float32)

    def reset(self, seed=None, options=None):
        """重置环境至初始状态"""
        super().reset(seed=seed)
        observation = self.observation_space.sample()
        info = {}
        return observation, info

    def step(self, action):
        """执行单步操作"""
        observation = self.observation_space.sample()
        reward = 0.0
        terminated = False  # 自然结束的回合
        truncated = False   # 因时间限制结束的回合
        info = {}
        return observation, reward, terminated, truncated, info

    def render(self):
        """可视化环境（可选）"""
        pass

    def close(self):
        """清理资源（可选）"""
        pass
```

### 方法详解

#### `__init__(self, ...)`

**目的：** 初始化环境并定义空间。

**要求：**
- 必须调用 `super().__init__()`
- 必须定义 `self.action_space`
- 必须定义 `self.observation_space`

**示例：**
```python
def __init__(self, grid_size=10, max_steps=100):
    super().__init__()
    self.grid_size = grid_size
    self.max_steps = max_steps
    self.current_step = 0

    # 定义空间
    self.action_space = spaces.Discrete(4)
    self.observation_space = spaces.Box(
        low=0, high=grid_size-1, shape=(2,), dtype=np.float32
    )
```

#### `reset(self, seed=None, options=None)`

**目的：** 将环境重置至初始状态。

**要求：**
- 必须调用 `super().reset(seed=seed)`
- 必须返回 `(observation, info)` 元组
- 观测值必须匹配 `observation_space`
- 信息必须为字典（可为空）

**示例：**
```python
def reset(self, seed=None, options=None):
    super().reset(seed=seed)

    # 初始化状态
    self.agent_pos = self.np_random.integers(0, self.grid_size, size=2)
    self.goal_pos = self.np_random.integers(0, self.grid_size, size=2)
    self.current_step = 0

    observation = self._get_observation()
    info = {"episode": "started"}

    return observation, info
```

#### `step(self, action)`

**目的：** 在环境中执行单步操作。

**要求：**
- 必须返回5元组：`(observation, reward, terminated, truncated, info)`
- 动作必须符合 `action_space` 要求
- 观测值必须匹配 `observation_space`
- 奖励应为浮点数
- Terminated：True 表示回合自然结束（达成目标、失败等）
- Truncated：True 表示因时间限制结束回合
- 信息必须为字典

**示例：**
```python
def step(self, action):
    # 应用动作
    self.agent_pos += self._action_to_direction(action)
    self.agent_pos = np.clip(self.agent_pos, 0, self.grid_size - 1)
    self.current_step += 1

    # 计算奖励
    distance = np.linalg.norm(self.agent_pos - self.goal_pos)
    goal_reached = distance < 1.0

    if goal_reached:
        reward = 100.0
    else:
        reward = -distance * 0.1

    # 检查终止条件
    terminated = goal_reached
    truncated = self.current_step >= self.max_steps

    observation = self._get_observation()
    info = {"distance": distance, "steps": self.current_step}

    return observation, reward, terminated, truncated, info
```

## 空间类型

### 离散空间 (Discrete)

适用于离散动作（例如 {0, 1, 2, 3}）。

```python
self.action_space = spaces.Discrete(4)  # 4个动作：0, 1, 2, 3
```

**重要：** SB3 不支持 `start != 0` 的 `Discrete` 空间。始终从0开始。

### 连续空间 (Box)

适用于范围内的连续值。

```python
# 1维连续动作 [-1, 1]
self.action_space = spaces.Box(low=-1, high=1, shape=(1,), dtype=np.float32)

# 2维位置观测
self.observation_space = spaces.Box(
    low=0, high=10, shape=(2,), dtype=np.float32
)

# 3维RGB图像（通道优先格式）
self.observation_space = spaces.Box(
    low=0, high=255, shape=(3, 84, 84), dtype=np.uint8
)
```

**图像处理要点：**
- 必须使用 `dtype=np.uint8` 且范围在 [0, 255]
- 使用**通道优先**格式：(通道数, 高度, 宽度)
- SB3 会自动通过除以255进行归一化
- 若已预归一化，在 policy_kwargs 中设置 `normalize_images=False`

### 多维离散空间 (MultiDiscrete)

适用于多个离散变量。

```python
# 两个离散变量：第一个有3个选项，第二个有4个选项
self.action_space = spaces.MultiDiscrete([3, 4])
```

### 多维二元空间 (MultiBinary)

适用于二元向量。

```python
# 5个二元标志
self.action_space = spaces.MultiBinary(5)  # 例如 [0, 1, 1, 0, 1]
```

### 字典空间 (Dict)

适用于字典形式观测（例如组合图像与传感器数据）。

```python
self.observation_space = spaces.Dict({
    "image": spaces.Box(low=0, high=255, shape=(3, 64, 64), dtype=np.uint8),
    "vector": spaces.Box(low=-10, high=10, shape=(4,), dtype=np.float32),
    "discrete": spaces.Discrete(3),
})
```

**重要：** 使用字典观测时，需使用 `"MultiInputPolicy"` 替代 `"MlpPolicy"`。

```python
model = PPO("MultiInputPolicy", env, verbose=1)
```

### 元组空间 (Tuple)

适用于元组形式观测（较少使用）。

```python
self.observation_space = spaces.Tuple((
    spaces.Box(low=0, high=1, shape=(4,), dtype=np.float32),
    spaces.Discrete(3),
))
```

## 重要约束与最佳实践

### 数据类型

- **观测值：** 连续值使用 `np.float32`
- **图像：** 使用范围 [0, 255] 的 `np.uint8`
- **奖励：** 返回 Python float 或 `np.float32`
- **Terminated/Truncated：** 返回 Python bool

### 随机数生成

始终使用 `self.np_random` 保证可复现性：

```python
def reset(self, seed=None, options=None):
    super().reset(seed=seed)
    # 使用 self.np_random 替代 np.random
    random_pos = self.np_random.integers(0, 10, size=2)
    random_float = self.np_random.random()
```

### 回合终止

- **Terminated：** 自然结束（达成目标、智能体死亡等）
- **Truncated：** 人为结束（时间耗尽、外部中断）

```python
def step(self, action):
    # ... 环境逻辑 ...

    goal_reached = self._check_goal()
    time_limit_exceeded = self.current_step >= self.max_steps

    terminated = goal_reached  # 自然结束
    truncated = time_limit_exceeded  # 时间耗尽

    return observation, reward, terminated, truncated, info
```

### 信息字典

使用 info 字典进行调试和日志记录：

```python
info = {
    "episode_length": self.current_step,
    "distance_to_goal": distance,
    "success": goal_reached,
    "total_reward": self.cumulative_reward,
}
```

**特殊键：**
- `"terminal_observation"`：回合结束时由 VecEnv 自动添加

## 高级功能

### 元数据

提供渲染信息：

```python
class CustomEnv(gym.Env):
    metadata = {
        "render_modes": ["human", "rgb_array"],
        "render_fps": 30,
    }

    def __init__(self, render_mode=None):
        super().__init__()
        self.render_mode = render_mode
        # ...
```

### 渲染模式

```python
def render(self):
    if self.render_mode == "human":
        # 供人工查看的打印或显示
        print(f"Agent at {self.agent_pos}")

    elif self.render_mode == "rgb_array":
        # 返回视频录制用的numpy数组 (高度, 宽度, 3)
        canvas = np.zeros((500, 500, 3), dtype=np.uint8)
        # 在画布上绘制环境
        return canvas
```

### 目标条件环境（用于HER）

后见经验回放需使用特定观测结构：

```python
self.observation_space = spaces.Dict({
    "observation": spaces.Box(low=-10, high=10, shape=(3,), dtype=np.float32),
    "achieved_goal": spaces.Box(low=-10, high=10, shape=(3,), dtype=np.float32),
    "desired_goal": spaces.Box(low=-10, high=10, shape=(3,), dtype=np.float32),
})

def compute_reward(self, achieved_goal, desired_goal, info):
    """HER环境必需方法"""
    distance = np.linalg.norm(achieved_goal - desired_goal)
    return -distance
```

## 环境验证

训练前务必验证环境：

```python
from stable_baselines3.common.env_checker import check_env

env = CustomEnv()
check_env(env, warn=True)
```

**常见验证错误：**

1. **"观测值超出边界"**
   - 检查观测值是否保持在定义空间内
   - 确保正确数据类型（Box空间使用np.float32）

2. **"Reset应返回元组"**
   - 返回 `(observation, info)`，而非单独观测值

3. **"Step应返回5元组"**
   - 返回 `(obs, reward, terminated, truncated, info)`

4. **"动作超出边界"**
   - 确认 action_space 定义与预期动作匹配

5. **"观测/动作数据类型不匹配"**
   - 确保观测值匹配空间数据类型（通常为np.float32）

## 环境注册

向 Gymnasium 注册环境：

```python
import gymnasium as gym
from gymnasium.envs.registration import register

register(
    id="MyCustomEnv-v0",
    entry_point="my_module:CustomEnv",
    max_episode_steps=200,
    kwargs={"grid_size": 10},  # 默认参数
)

# 现在可通过gym.make使用
env = gym.make("MyCustomEnv-v0")
```

## 测试自定义环境

### 基础测试

```python
def test_environment(env, n_episodes=5):
    """使用随机动作测试环境"""
    for episode in range(n_episodes):
        obs, info = env.reset()
        episode_reward = 0
        done = False
        steps = 0

        while not done:
            action = env.action_space.sample()
            obs, reward, terminated, truncated, info = env.step(action)
            episode_reward += reward
            steps += 1
            done = terminated or truncated

        print(f"回合 {episode+1}: 奖励={episode_reward:.2f}, 步数={steps}")
```

### 训练测试

```python
from stable_baselines3 import PPO

def train_test(env, timesteps=10000):
    """快速训练测试"""
    model = PPO("MlpPolicy", env, verbose=1)
    model.learn(total_timesteps=timesteps)

    # 评估
    obs, info = env.reset()
    for _ in range(100):
        action, _states = model.predict(obs, deterministic=True)
        obs, reward, terminated, truncated, info = env.step(action)
        if terminated or truncated:
            break
```

## 常见模式

### 网格世界

```python
class GridWorldEnv(gym.Env):
    def __init__(self, size=10):
        super().__init__()
        self.size = size
        self.action_space = spaces.Discrete(4)  # 上,下,左,右
        self.observation_space = spaces.Box(0, size-1, shape=(2,), dtype=np.float32)
```

### 连续控制

```python
class ContinuousEnv(gym.Env):
    def __init__(self):
        super().__init__()
        self.action_space = spaces.Box(low=-1, high=1, shape=(2,), dtype=np.float32)
        self.observation_space = spaces.Box(low=-np.inf, high=np.inf, shape=(8,), dtype=np.float32)
```

### 基于图像的环境

```python
class VisionEnv(gym.Env):
    def __init__(self):
        super().__init__()
        self.action_space = spaces.Discrete(4)
        # 通道优先: (通道数, 高度, 宽度)
        self.observation_space = spaces.Box(
            low=0, high=255, shape=(3, 84, 84), dtype=np.uint8
        )
```

### 多模态环境

```python
class MultiModalEnv(gym.Env):
    def __init__(self):
        super().__init__()
        self.action_space = spaces.Discrete(4)
        self.observation_space = spaces.Dict({
            "image": spaces.Box(0, 255, shape=(3, 64, 64), dtype=np.uint8),
            "sensors": spaces.Box(-10, 10, shape=(4,), dtype=np.float32),
        })
```

## 性能考量

### 高效观测生成

```python
# 预分配数组
def __init__(self):
    # ...
    self._obs_buffer = np.zeros(self.observation_space.shape, dtype=np.float32)

def _get_observation(self):
    # 重用缓冲区而非分配新数组
    self._obs_buffer[0] = self.agent_x
    self._obs_buffer[1] = self.agent_y
    return self._obs_buffer
```

### 向量化

使环境操作可向量化：

```python
# 推荐：使用numpy操作
def step(self, action):
    direction = np.array([[0,1], [0,-1], [1,0], [-1,0]])[action]
    self.pos = np.clip(self.pos + direction, 0, self.size-1)

# 避免：尽可能减少Python循环
# for i in range(len(self.agents)):
#     self.agents[i].update()
```

## 故障排除

### "观测值超出边界"
- 检查所有观测值是否在定义空间内
- 验证数据类型（np.float32 与 np.float64）

### "观测/奖励中出现NaN或Inf"
- 添加检查：`assert np.isfinite(reward)`
- 使用 `VecCheckNan` 包装器捕获问题

### "策略未学习"
- 检查奖励缩放（归一化奖励）
- 验证观测值归一化
- 确保奖励信号有意义
- 检查探索是否充分

### "训练崩溃"
- 使用 `check_env()` 验证环境
- 检查自定义环境中的竞态条件
- 确认动作/观测空间一致性

## 附加资源

- 模板：参见 `scripts/custom_env_template.py`
- Gymnasium 文档：https://gymnasium.farama.org/
- SB3 自定义环境指南：https://stable-baselines3.readthedocs.io/en/master/guide/custom_env.html
