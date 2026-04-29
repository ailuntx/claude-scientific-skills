# Stable Baselines3 中的向量化环境

本文档提供关于 Stable Baselines3 中向量化环境的全面信息，用于实现高效的并行训练。

## 概述

向量化环境将多个独立的环境实例堆叠成单个环境，以批处理方式处理动作和观测值。您不再每次与一个环境交互，而是同时与 `n` 个环境交互。

**优势：**
- **速度：** 并行执行显著加速训练
- **样本效率：** 更快收集多样化经验
- **必需场景：** 帧堆叠和标准化封装器
- **更适用：** 同策略算法（PPO, A2C）

## 向量化环境类型

### DummyVecEnv

在当前 Python 进程中顺序执行环境。

```python
from stable_baselines3.common.vec_env import DummyVecEnv

# 方法1：使用 make_vec_env
from stable_baselines3.common.env_util import make_vec_env

env = make_vec_env("CartPole-v1", n_envs=4, vec_env_cls=DummyVecEnv)

# 方法2：手动创建
def make_env():
    def _init():
        return gym.make("CartPole-v1")
    return _init

env = DummyVecEnv([make_env() for _ in range(4)])
```

**适用场景：**
- 轻量级环境（CartPole, 简单网格）
- 当多进程开销 > 计算时间时
- 调试（更易追踪错误）
- 单线程环境

**性能：** 无实际并行（顺序执行）

### SubprocVecEnv

在独立进程中执行每个环境，实现真正并行。

```python
from stable_baselines3.common.vec_env import SubprocVecEnv
from stable_baselines3.common.env_util import make_vec_env

env = make_vec_env("CartPole-v1", n_envs=8, vec_env_cls=SubprocVecEnv)
```

**适用场景：**
- 计算密集型环境（物理模拟, 3D游戏）
- 当环境计算时间可抵消多进程开销时
- 需要真正并行执行时

**重要：** 使用 forkserver 或 spawn 时需将代码包裹在 `if __name__ == "__main__":` 中：

```python
if __name__ == "__main__":
    env = make_vec_env("CartPole-v1", n_envs=8, vec_env_cls=SubprocVecEnv)
    model = PPO("MlpPolicy", env)
    model.learn(total_timesteps=100000)
```

**性能：** 跨 CPU 核心的真正并行

## 使用 make_vec_env 快速配置

创建向量化环境的最简方式：

```python
from stable_baselines3.common.env_util import make_vec_env
from stable_baselines3.common.vec_env import SubprocVecEnv

# 基础用法
env = make_vec_env("CartPole-v1", n_envs=4)

# 使用 SubprocVecEnv
env = make_vec_env("CartPole-v1", n_envs=8, vec_env_cls=SubprocVecEnv)

# 自定义环境参数
env = make_vec_env(
    "MyEnv-v0",
    n_envs=4,
    env_kwargs={"difficulty": "hard", "max_steps": 500}
)

# 自定义随机种子
env = make_vec_env("CartPole-v1", n_envs=4, seed=42)
```

## 与标准 Gym API 的差异

向量化环境 API 与标准 Gym 环境不同：

### reset()

**标准 Gym：**
```python
obs, info = env.reset()
```

**向量化环境：**
```python
obs = env.reset()  # 仅返回观测值（numpy数组）
# 通过 env.reset_infos 访问信息
infos = env.reset_infos
```

### step()

**标准 Gym：**
```python
obs, reward, terminated, truncated, info = env.step(action)
```

**向量化环境：**
```python
obs, rewards, dones, infos = env.step(actions)
# 返回4元组而非5元组
# dones = terminated | truncated
# actions 形状为 (n_envs,) 或 (n_envs, action_dim)
```

### 自动重置

**向量化环境在回合结束时自动重置：**

```python
obs = env.reset()  # 形状: (n_envs, obs_dim)
for _ in range(1000):
    actions = env.action_space.sample()  # 形状: (n_envs,)
    obs, rewards, dones, infos = env.step(actions)
    # 若 dones[i] 为 True，环境 i 已自动重置
    # 重置前的最终观测值可在 infos[i]["terminal_observation"] 获取
```

### 终止状态观测值

回合结束时访问真实最终观测值：

```python
obs, rewards, dones, infos = env.step(actions)

for i, done in enumerate(dones):
    if done:
        # obs[i] 已是重置后的观测值
        # 真实终止观测值在 info 中
        terminal_obs = infos[i]["terminal_observation"]
        print(f"回合终止观测值: {terminal_obs}")
```

## 使用向量化环境训练

### 同策略算法（PPO, A2C）

同策略算法显著受益于向量化：

```python
from stable_baselines3 import PPO
from stable_baselines3.common.env_util import make_vec_env
from stable_baselines3.common.vec_env import SubprocVecEnv

# 创建向量化环境
env = make_vec_env("CartPole-v1", n_envs=8, vec_env_cls=SubprocVecEnv)

# 训练
model = PPO("MlpPolicy", env, verbose=1, n_steps=128)
model.learn(total_timesteps=100000)

# 当 n_envs=8 且 n_steps=128 时：
# - 每轮收集 8*128=1024 步
# - 每1024步更新一次
```

**经验法则：** 同策略方法使用 4-16 个并行环境

### 异策略算法（SAC, TD3, DQN）

异策略算法可使用向量化但受益较小：

```python
from stable_baselines3 import SAC
from stable_baselines3.common.env_util import make_vec_env

# 使用较少环境（1-4个）
env = make_vec_env("Pendulum-v1", n_envs=4)

# 设置 gradient_steps=-1 提升效率
model = SAC(
    "MlpPolicy",
    env,
    verbose=1,
    train_freq=1,
    gradient_steps=-1,  # 每环境步执行1次梯度更新（4环境共4次）
)
model.learn(total_timesteps=50000)
```

**经验法则：** 异策略方法使用 1-4 个并行环境

## 向量化环境封装器

### VecNormalize

使用运行统计量标准化观测值和奖励

```python
from stable_baselines3.common.vec_env import VecNormalize

env = make_vec_env("Pendulum-v1", n_envs=4)

# 添加标准化封装
env = VecNormalize(
    env,
    norm_obs=True,        # 标准化观测值
    norm_reward=True,     # 标准化奖励
    clip_obs=10.0,        # 截断标准化观测值
    clip_reward=10.0,     # 截断标准化奖励
    gamma=0.99,           # 奖励标准化的折扣因子
)

# 训练
model = PPO("MlpPolicy", env)
model.learn(total_timesteps=50000)

# 保存模型及标准化统计量
model.save("ppo_pendulum")
env.save("vec_normalize.pkl")

# 评估时加载
env = make_vec_env("Pendulum-v1", n_envs=1)
env = VecNormalize.load("vec_normalize.pkl", env)
env.training = False  # 评估期间不更新统计量
env.norm_reward = False  # 评估期间不标准化奖励

model = PPO.load("ppo_pendulum", env=env)
```

**适用场景：**
- 连续控制任务（特别是 MuJoCo）
- 观测值尺度差异大时
- 奖励方差较高时

**重要提示：**
- 统计量不与模型一起保存 - 需单独保存
- 评估时禁用训练和奖励标准化

### VecFrameStack

堆叠连续多帧的观测值

```python
from stable_baselines3.common.vec_env import VecFrameStack

env = make_vec_env("PongNoFrameskip-v4", n_envs=8)

# 堆叠4帧
env = VecFrameStack(env, n_stack=4)

# 观测值形状变为: (n_envs, n_stack, height, width)
model = PPO("CnnPolicy", env)
model.learn(total_timesteps=1000000)
```

**适用场景：**
- Atari 游戏（堆叠4帧）
- 需要速度信息的场景
- 部分可观测性问题

### VecVideoRecorder

录制智能体行为视频

```python
from stable_baselines3.common.vec_env import VecVideoRecorder

env = make_vec_env("CartPole-v1", n_envs=1)

# 录制视频
env = VecVideoRecorder(
    env,
    video_folder="./videos/",
    record_video_trigger=lambda x: x % 2000 == 0,  # 每2000步录制
    video_length=200,  # 最大视频长度
    name_prefix="training"
)

model = PPO("MlpPolicy", env)
model.learn(total_timesteps=10000)
```

**输出：** `./videos/` 目录下的 MP4 视频

### VecCheckNan

检查观测值和奖励中的 NaN 或无限值

```python
from stable_baselines3.common.vec_env import VecCheckNan

env = make_vec_env("CustomEnv-v0", n_envs=4)

# 添加NaN检查（调试用）
env = VecCheckNan(env, raise_exception=True, warn_once=True)

model = PPO("MlpPolicy", env)
model.learn(total_timesteps=10000)
```

**适用场景：**
- 调试自定义环境
- 捕获数值不稳定问题
- 验证环境实现

### VecTransposeImage

将图像观测值从 (高度, 宽度, 通道) 转置为 (通道, 高度, 宽度)

```python
from stable_baselines3.common.vec_env import VecTransposeImage

env = make_vec_env("PongNoFrameskip-v4", n_envs=4)

# 转换 HWC 为 CHW 格式
env = VecTransposeImage(env)

model = PPO("CnnPolicy", env)
```

**适用场景：**
- 环境返回 HWC 格式图像时
- SB3 的 CNN 策略要求 CHW 格式

## 高级用法

### 自定义向量化环境

创建自定义向量化环境：

```python
from stable_baselines3.common.vec_env import DummyVecEnv
import gymnasium as gym

class CustomVecEnv(DummyVecEnv):
    def step_wait(self):
        # 步进前后的自定义逻辑
        obs, rewards, dones, infos = super().step_wait()
        # 修改观测值/奖励等
        return obs, rewards, dones, infos
```

### 环境方法调用

调用封装环境的方法：

```python
env = make_vec_env("MyEnv-v0", n_envs=4)

# 调用所有环境的同名方法
env.env_method("set_difficulty", "hard")

# 调用特定环境的方法
env.env_method("reset_level", indices=[0, 2])

# 获取所有环境的属性
levels = env.get_attr("current_level")
```

### 设置属性

```python
# 设置所有环境的属性
env.set_attr("difficulty", "hard")

# 设置特定环境的属性
env.set_attr("max_steps", 1000, indices=[1, 3])
```

## 性能优化

### 选择环境数量

**同策略算法（PPO, A2C）：**
```python
# 通用规则：4-16个环境
# 环境越多 = 数据收集越快
n_envs = 8
env = make_vec_env("CartPole-v1", n_envs=n_envs)

# 调整 n_steps 保持相同轮次长度
# 每轮总步数 = n_envs * n_steps
model = PPO("MlpPolicy", env, n_steps=128)  # 8*128 = 1024步/轮次
```

**异策略算法（SAC, TD3, DQN）：**
```python
# 通用规则：1-4个环境
# 增加环境收益有限（经验回放池提供多样性）
n_envs = 4
env = make_vec_env("Pendulum-v1", n_envs=n_envs)

model = SAC("MlpPolicy", env, gradient_steps=-1)  # 每环境步执行1次梯度更新
```

### CPU 核心利用率

```python
import multiprocessing

# 使用总核心数减一（保留一个给Python主进程）
n_cpus = multiprocessing.cpu_count() - 1
env = make_vec_env("MyEnv-v0", n_envs=n_cpus, vec_env_cls=SubprocVecEnv)
```

### 内存考量

```python
# 大型回放池 + 多环境 = 高内存占用
# 内存受限时减小缓冲区
model = SAC(
    "MlpPolicy",
    env,
    buffer_size=100_000,  # 从1M缩减
)
```

## 常见问题

### 问题："Can't pickle local object"

**原因：** SubprocVecEnv 要求环境可序列化

**解决方案：** 在类/函数外定义环境创建：

```python
# 错误方式
def train():
    def make_env():
        return gym.make("CartPole-v1")
    env = SubprocVecEnv([make_env for _ in range(4)])

# 正确方式
def make_env():
    return gym.make("CartPole-v1")

if __name__ == "__main__":
    env = SubprocVecEnv([make_env for _ in range(4)])
```

### 问题：单环境与向量化环境行为不一致

**原因：** 向量化环境的自动重置机制

**解决方案：** 正确处理终止观测值：

```python
obs, rewards, dones, infos = env.step(actions)
for i, done in enumerate(dones):
    if done:
        terminal_obs = infos[i]["terminal_observation"]
        # 如需处理终止观测值
```

### 问题：SubprocVecEnv 比 DummyVecEnv 更慢

**原因：** 环境过于轻量（多进程开销 > 计算量）

**解决方案：** 简单环境使用 DummyVecEnv：

```python
# CartPole 使用 DummyVecEnv
env = make_vec_env("CartPole-v1", n_envs=8, vec_env_cls=DummyVecEnv)
```

### 问题：使用 SubprocVecEnv 时训练崩溃

**原因：** 环境未完全隔离或存在共享状态

**解决方案：**
- 确保环境无共享全局状态
- 用 `if __name__ == "__main__":` 包裹代码
- 调试时使用 DummyVecEnv

## 最佳实践

1. **选用合适的向量化环境类型：**
   - DummyVecEnv：简单环境（CartPole, 基础网格）
   - SubprocVecEnv：复杂环境（MuJoCo, Unity, 3D游戏）

2. **为向量化调整超参数：**
   - 回调函数中将 `eval_freq`, `save_freq` 除以 `n_envs`
   - 同策略算法保持相同 `n_steps * n_envs`

3. **保存标准化统计量：**
   - 始终与模型一起保存 VecNormalize 统计量
   - 评估时禁用训练

4. **监控内存使用：**
   - 环境越多 = 内存占用越高
   - 必要时减小缓冲区

5. **先用 DummyVecEnv 测试：**
   - 更易调试
   - 确保环境工作正常后再并行化

## 示例

### 基础训练循环

```python
from stable_baselines3 import PPO
from stable_baselines3.common.env_util import make_vec_env
from stable_baselines3.common.vec_env import SubprocVecEnv

# 创建向量化环境
env = make_vec_env("CartPole-v1", n_envs=8, vec_env_cls=SubprocVecEnv)

# 训练
model = PPO("MlpPolicy", env, verbose=1)
model.learn(total_timesteps=100000)

# 评估
obs = env.reset()
for _ in range(1000):
    action, _states = model.predict(obs, deterministic=True)
    obs, rewards, dones, infos = env.step(action)
```

### 带标准化

```python
from stable_baselines3 import PPO
from stable_baselines3.common.env_util import make_vec_env
from stable_baselines3.common.vec_env import VecNormalize

# 创建并标准化
env = make_vec_env("P

- SB3官方VecEnv指南：https://stable-baselines3.readthedocs.io/en/master/guide/vec_envs.html
- VecEnv API参考文档：https://stable-baselines3.readthedocs.io/en/master/common/vec_env.html
- 多进程最佳实践：https://docs.python.org/3/library/multiprocessing.html
