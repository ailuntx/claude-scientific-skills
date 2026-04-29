# PufferLib 集成指南

## 概述

PufferLib 提供仿真层，可无缝集成主流强化学习框架，包括 Gymnasium、OpenAI Gym、PettingZoo 及众多专用环境库。该仿真层通过扁平化观测空间与动作空间实现高效向量化，同时保持兼容性。

## Gymnasium 集成

### 基础 Gymnasium 环境

```python
import gymnasium as gym
import pufferlib

# 方法1：直接封装
gym_env = gym.make('CartPole-v1')
puffer_env = pufferlib.emulate(gym_env, num_envs=256)

# 方法2：使用make创建
env = pufferlib.make('gym-CartPole-v1', num_envs=256)

# 方法3：自定义Gymnasium环境
class MyGymEnv(gym.Env):
    def __init__(self):
        self.observation_space = gym.spaces.Box(low=-1, high=1, shape=(4,))
        self.action_space = gym.spaces.Discrete(2)

    def reset(self, seed=None, options=None):
        super().reset(seed=seed)
        return self.observation_space.sample(), {}

    def step(self, action):
        obs = self.observation_space.sample()
        reward = 1.0
        terminated = False
        truncated = False
        info = {}
        return obs, reward, terminated, truncated, info

# 封装自定义环境
puffer_env = pufferlib.emulate(MyGymEnv, num_envs=128)
```

### Atari 环境

```python
import gymnasium as gym
from gymnasium.wrappers import AtariPreprocessing, FrameStack
import pufferlib

# 标准Atari配置
def make_atari_env(env_name='ALE/Pong-v5'):
    env = gym.make(env_name)
    env = AtariPreprocessing(env, frame_skip=4)
    env = FrameStack(env, num_stack=4)
    return env

# 使用PufferLib向量化
env = pufferlib.emulate(make_atari_env, num_envs=256)

# 或使用内置方法
env = pufferlib.make('atari-pong', num_envs=256, frameskip=4, framestack=4)
```

### 复杂观测空间

```python
import gymnasium as gym
from gymnasium.spaces import Dict, Box, Discrete
import pufferlib

class ComplexObsEnv(gym.Env):
    def __init__(self):
        # 字典观测空间
        self.observation_space = Dict({
            'image': Box(low=0, high=255, shape=(84, 84, 3), dtype=np.uint8),
            'vector': Box(low=-np.inf, high=np.inf, shape=(10,), dtype=np.float32),
            'discrete': Discrete(5)
        })
        self.action_space = Discrete(4)

    def reset(self, seed=None, options=None):
        return {
            'image': np.zeros((84, 84, 3), dtype=np.uint8),
            'vector': np.zeros(10, dtype=np.float32),
            'discrete': 0
        }, {}

    def step(self, action):
        obs = {
            'image': np.random.randint(0, 256, (84, 84, 3), dtype=np.uint8),
            'vector': np.random.randn(10).astype(np.float32),
            'discrete': np.random.randint(0, 5)
        }
        return obs, 1.0, False, False, {}

# PufferLib自动扁平化与还原复杂空间
env = pufferlib.emulate(ComplexObsEnv, num_envs=128)
```

## PettingZoo 集成

### 并行环境

```python
from pettingzoo.butterfly import pistonball_v6
import pufferlib

# 封装PettingZoo并行环境
pz_env = pistonball_v6.parallel_env()
puffer_env = pufferlib.emulate(pz_env, num_envs=128)

# 或直接使用make
env = pufferlib.make('pettingzoo-pistonball', num_envs=128)
```

### AEC（智能体环境循环）环境

```python
from pettingzoo.classic import chess_v5
import pufferlib

# 封装AEC环境（PufferLib自动转为并行）
aec_env = chess_v5.env()
puffer_env = pufferlib.emulate(aec_env, num_envs=64)

# 兼容所有PettingZoo AEC环境
env = pufferlib.make('pettingzoo-chess', num_envs=64)
```

### 多智能体训练

```python
import pufferlib
from pufferlib import PuffeRL

# 创建多智能体环境
env = pufferlib.make('pettingzoo-knights-archers-zombies', num_envs=128)

# 所有智能体共享策略
policy = create_policy(env.observation_space, env.action_space)

# 训练
trainer = PuffeRL(env=env, policy=policy)

for iteration in range(num_iterations):
    # 观测为字典格式：{agent_id: batch_obs}
    rollout = trainer.evaluate()

    # 基于多智能体数据训练
    trainer.train()
    trainer.mean_and_log()
```

## 第三方环境

### Procgen

```python
import pufferlib

# Procgen环境
env = pufferlib.make('procgen-coinrun', num_envs=256, distribution_mode='easy')

# 自定义配置
env = pufferlib.make(
    'procgen-coinrun',
    num_envs=256,
    num_levels=200,  # 唯一关卡数量
    start_level=0,   # 起始关卡种子
    distribution_mode='hard'
)
```

### NetHack

```python
import pufferlib

# NetHack学习环境
env = pufferlib.make('nethack', num_envs=128)

# MiniHack变体
env = pufferlib.make('minihack-corridor', num_envs=128)
env = pufferlib.make('minihack-room', num_envs=128)
```

### Minigrid

```python
import pufferlib

# Minigrid环境
env = pufferlib.make('minigrid-empty-8x8', num_envs=256)
env = pufferlib.make('minigrid-doorkey-8x8', num_envs=256)
env = pufferlib.make('minigrid-multiroom', num_envs=256)
```

### Neural MMO

```python
import pufferlib

# 大规模多智能体环境
env = pufferlib.make(
    'neuralmmo',
    num_envs=64,
    num_agents=128,  # 每环境智能体数
    map_size=128
)
```

### Crafter

```python
import pufferlib

# 开放式制作环境
env = pufferlib.make('crafter', num_envs=128)
```

### GPUDrive

```python
import pufferlib

# GPU加速驾驶模拟器
env = pufferlib.make(
    'gpudrive',
    num_envs=1024,  # 支持GPU上运行大量环境
    num_vehicles=8
)
```

### MicroRTS

```python
import pufferlib

# 即时战略游戏
env = pufferlib.make(
    'microrts',
    num_envs=128,
    map_size=16,
    max_steps=2000
)
```

### Griddly

```python
import pufferlib

# 网格类游戏
env = pufferlib.make('griddly-clusters', num_envs=256)
env = pufferlib.make('griddly-sokoban', num_envs=256)
```

## 自定义封装器

### 观测封装器

```python
import numpy as np
import pufferlib
from pufferlib import PufferEnv

class NormalizeObservations(pufferlib.Wrapper):
    """将观测归一化为零均值单位方差"""

    def __init__(self, env):
        super().__init__(env)
        self.obs_mean = np.zeros(env.observation_space.shape)
        self.obs_std = np.ones(env.observation_space.shape)
        self.count = 0

    def reset(self):
        obs = self.env.reset()
        return self._normalize(obs)

    def step(self, action):
        obs, reward, done, info = self.env.step(action)
        return self._normalize(obs), reward, done, info

    def _normalize(self, obs):
        # 更新运行统计量
        self.count += 1
        delta = obs - self.obs_mean
        self.obs_mean += delta / self.count
        self.obs_std = np.sqrt(((self.count - 1) * self.obs_std ** 2 + delta * (obs - self.obs_mean)) / self.count)

        # 归一化处理
        return (obs - self.obs_mean) / (self.obs_std + 1e-8)
```

### 奖励封装器

```python
class RewardShaping(pufferlib.Wrapper):
    """为环境添加塑形奖励"""

    def __init__(self, env, shaping_fn):
        super().__init__(env)
        self.shaping_fn = shaping_fn

    def step(self, action):
        obs, reward, done, info = self.env.step(action)

        # 添加塑形奖励
        shaped_reward = reward + self.shaping_fn(obs, action)

        return obs, shaped_reward, done, info

# 使用示例
def proximity_shaping(obs, action):
    """智能体接近目标时给予奖励"""
    goal_pos = np.array([10, 10])
    agent_pos = obs[:2]
    distance = np.linalg.norm(goal_pos - agent_pos)
    return -0.1 * distance

env = pufferlib.make('myenv', num_envs=128)
env = RewardShaping(env, proximity_shaping)
```

### 帧堆叠

```python
class FrameStack(pufferlib.Wrapper):
    """为时序上下文堆叠帧"""

    def __init__(self, env, num_stack=4):
        super().__init__(env)
        self.num_stack = num_stack
        self.frames = None

    def reset(self):
        obs = self.env.reset()

        # 初始化帧堆栈
        self.frames = np.repeat(obs[np.newaxis], self.num_stack, axis=0)

        return self._get_obs()

    def step(self, action):
        obs, reward, done, info = self.env.step(action)

        # 更新帧堆栈
        self.frames = np.roll(self.frames, shift=-1, axis=0)
        self.frames[-1] = obs

        if done:
            self.frames = None

        return self._get_obs(), reward, done, info

    def _get_obs(self):
        return self.frames
```

### 动作重复

```python
class ActionRepeat(pufferlib.Wrapper):
    """重复执行动作多步"""

    def __init__(self, env, repeat=4):
        super().__init__(env)
        self.repeat = repeat

    def step(self, action):
        total_reward = 0.0
        done = False

        for _ in range(self.repeat):
            obs, reward, done, info = self.env.step(action)
            total_reward += reward

            if done:
                break

        return obs, total_reward, done, info
```

## 空间转换

### 扁平化空间

PufferLib 自动扁平化复杂观测/动作空间：

```python
from gymnasium.spaces import Dict, Box, Discrete
import pufferlib

# 复杂空间
original_space = Dict({
    'image': Box(0, 255, (84, 84, 3), dtype=np.uint8),
    'vector': Box(-np.inf, np.inf, (10,), dtype=np.float32),
    'discrete': Discrete(5)
})

# PufferLib自动扁平化
# 观测以扁平数组形式呈现以提升处理效率
# 策略处理时可还原为原始结构
```

### 策略层空间还原

```python
from pufferlib.pytorch import unflatten_observations

class PolicyWithUnflatten(nn.Module):
    def __init__(self, observation_space, action_space):
        super().__init__()
        self.observation_space = observation_space
        # ... 策略架构 ...

    def forward(self, flat_observations):
        # 还原为原始结构
        observations = unflatten_observations(
            flat_observations,
            self.observation_space
        )

        # 此时observations为包含'image','vector','discrete'的字典
        image_features = self.image_encoder(observations['image'])
        vector_features = self.vector_encoder(observations['vector'])
        # ...
```

## 环境注册

### 注册自定义环境

```python
import pufferlib

# 注册环境以便快捷访问
pufferlib.register(
    id='my-custom-env',
    entry_point='my_package.envs:MyEnvironment',
    kwargs={'param1': 'value1'}
)

# 现在可通过make使用
env = pufferlib.make('my-custom-env', num_envs=256)
```

### 在 Ocean 套件中注册

将环境添加至 Ocean：

```python
# 在ocean/environment.py中
OCEAN_REGISTRY = {
    'my-env': {
        'entry_point': 'my_package.envs:MyEnvironment',
        'kwargs': {
            'default_param': 'default_value'
        }
    }
}
```

## 兼容性模式

### Gymnasium 转 PufferLib

```python
import gymnasium as gym
import pufferlib

# 标准Gymnasium环境
class GymEnv(gym.Env):
    def reset(self, seed=None, options=None):
        return observation, info

    def step(self, action):
        return observation, reward, terminated, truncated, info

# 转为PufferEnv
puffer_env = pufferlib.emulate(GymEnv, num_envs=128)
```

### PettingZoo 转 PufferLib

```python
from pettingzoo import ParallelEnv
import pufferlib

# PettingZoo并行环境
class PZEnv(ParallelEnv):
    def reset(self, seed=None, options=None):
        return {agent: obs for agent, obs in ...}, {agent: info for agent in ...}

    def step(self, actions):
        return observations, rewards, terminations, truncations, infos

# 转为PufferEnv
puffer_env = pufferlib.emulate(PZEnv, num_envs=128)
```

### 旧版 Gym (v0.21) 转 PufferLib

```python
import gym  # 旧版gym
import pufferlib

# 旧版gym环境（返回done而非terminated/truncated）
class LegacyEnv(gym.Env):
    def reset(self):
        return observation

    def step(self, action):
        return observation, reward, done, info

# PufferLib自动处理旧版格式
puffer_env = pufferlib.emulate(LegacyEnv, num_envs=128)
```

## 性能考量

### 高效集成

```python
# 快速：优先使用内置集成
env = pufferlib.make('procgen-coinrun', num_envs=256)

# 较慢：通用封装器（仍有开销）
import gymnasium as gym
gym_env = gym.make('CartPole-v1')
env = pufferlib.emulate(gym_env, num_envs=256)

# 最慢：嵌套封装增加开销
import gymnasium as gym
gym_env = gym.make('CartPole-v1')
gym_env = SomeWrapper(gym_env)
gym_env = AnotherWrapper(gym_env)
env = pufferlib.emulate(gym_env, num_envs=256)
```

### 最小化封装开销

```python
# 错误做法：过多封装器
env = gym.make('CartPole-v1')
env = Wrapper1(env)
env = Wrapper2(env)
env = Wrapper3(env)
puffer_env = pufferlib.emulate(env, num_envs=256)

# 正确做法：合并封装逻辑
class CombinedWrapper(gym.Wrapper):
    def step(self, action):
        obs, reward, done, truncated, info = self.env.step(action)
        # 一次性应用所有转换
        obs = self._transform_obs(obs)
        reward = self._transform_reward(reward)
        return obs, reward, done, truncated, info

env = gym.make('CartPole-v1')
env = CombinedWrapper(env)
puffer_env = pufferlib.emulate(env, num_envs=256)
```

## 集成调试

### 验证环境兼容性

```python
def test_environment(env, num_steps=100):
    """检测环境常见问题"""

assert env.observation_space.contains(obs), "无效的观测值"
        assert isinstance(reward, (int, float)), "无效的奖励类型"
        assert isinstance(done, bool), "无效的 done 类型"
        assert isinstance(info, dict), "无效的 info 类型"

        if done:
            obs = env.reset()

    print("✓ 环境通过兼容性测试")

# 矢量化前测试
test_environment(MyEnvironment())
```

### 比较输出

```python
# 验证 PufferLib 模拟是否与原始环境一致
import gymnasium as gym
import pufferlib
import numpy as np

gym_env = gym.make('CartPole-v1')
puffer_env = pufferlib.emulate(lambda: gym.make('CartPole-v1'), num_envs=1)

# 使用相同种子测试
gym_env.reset(seed=42)
puffer_obs = puffer_env.reset()

for _ in range(100):
    action = gym_env.action_space.sample()

    gym_obs, gym_reward, gym_done, gym_truncated, gym_info = gym_env.step(action)
    puffer_obs, puffer_reward, puffer_done, puffer_info = puffer_env.step(np.array([action]))

    # 比较输出（考虑批处理维度）
    assert np.allclose(gym_obs, puffer_obs[0])
    assert gym_reward == puffer_reward[0]
    assert gym_done == puffer_done[0]
```
