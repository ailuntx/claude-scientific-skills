```markdown
---
name: stable-baselines3
description: 生产级强化学习算法（PPO、SAC、DQN、TD3、DDPG、A2C），提供类似 scikit-learn 的 API。适用于标准 RL 实验、快速原型设计和文档完善的算法实现。最适合 Gymnasium 环境的单智能体 RL。如需高性能并行训练、多智能体系统或自定义向量化环境，请改用 pufferlib。
license: MIT 许可证
metadata:
    skill-author: K-Dense 公司
---

# Stable Baselines3

## 概述

Stable Baselines3（SB3）是基于 PyTorch 的强化学习算法库，提供可靠的算法实现。本技能提供使用 SB3 统一 API 训练 RL 智能体、创建自定义环境、实现回调函数和优化训练流程的全面指导。

## 核心功能

### 1. 训练 RL 智能体

**基础训练模式：**

```python
import gymnasium as gym
from stable_baselines3 import PPO

# 创建环境
env = gym.make("CartPole-v1")

# 初始化智能体
model = PPO("MlpPolicy", env, verbose=1)

# 训练智能体
model.learn(total_timesteps=10000)

# 保存模型
model.save("ppo_cartpole")

# 加载模型（无需预先实例化）
model = PPO.load("ppo_cartpole", env=env)
```

**重要说明：**
- `total_timesteps` 为最低训练步数，实际训练可能因批次收集而超出
- 使用 `model.load()` 作为静态方法，而非在现有实例上调用
- 为节省空间，回放缓冲区不与模型一起保存

**算法选择：**
参考 `references/algorithms.md` 获取详细算法特性与选择指南。速查表：
- **PPO/A2C**：通用型，支持所有动作空间类型，适合多进程
- **SAC/TD3**：连续控制，离策略，样本高效
- **DQN**：离散动作，离策略
- **HER**：目标导向型任务

完整训练模板与最佳实践见 `scripts/train_rl_agent.py`。

### 2. 自定义环境

**要求：**
自定义环境必须继承 `gymnasium.Env` 并实现：
- `__init__()`：定义 action_space 和 observation_space
- `reset(seed, options)`：返回初始观测值和信息字典
- `step(action)`：返回观测值、奖励、终止标志、截断标志和信息
- `render()`：可视化（可选）
- `close()`：资源清理

**关键约束：**
- 图像观测值必须为 `np.uint8` 类型且范围在 [0, 255]
- 尽可能使用通道优先格式（通道数, 高度, 宽度）
- SB3 自动通过除以 255 归一化图像
- 若已预归一化，在 policy_kwargs 中设置 `normalize_images=False`
- SB3 不支持 `start!=0` 的 `Discrete` 或 `MultiDiscrete` 空间

**环境验证：**
```python
from stable_baselines3.common.env_checker import check_env

check_env(env, warn=True)
```

完整自定义环境模板见 `scripts/custom_env_template.py`，综合指南见 `references/custom_environments.md`。

### 3. 向量化环境

**目的：**
向量化环境并行运行多个环境实例，加速训练并支持特定包装器（帧堆叠、归一化）。

**类型：**
- **DummyVecEnv**：当前进程顺序执行（轻量级环境）
- **SubprocVecEnv**：跨进程并行执行（计算密集型环境）

**快速设置：**
```python
from stable_baselines3.common.env_util import make_vec_env

# 创建 4 个并行环境
env = make_vec_env("CartPole-v1", n_envs=4, vec_env_cls=SubprocVecEnv)

model = PPO("MlpPolicy", env, verbose=1)
model.learn(total_timesteps=25000)
```

**离策略算法优化：**
使用多环境运行离策略算法（SAC、TD3、DQN）时，设置 `gradient_steps=-1` 实现每环境步执行一次梯度更新，平衡训练时间与样本效率。

**API 差异：**
- `reset()` 仅返回观测值（信息在 `vec_env.reset_infos` 中）
- `step()` 返回四元组 `(obs, rewards, dones, infos)` 而非五元组
- 环境在回合结束后自动重置
- 终止状态观测值通过 `infos[env_idx]["terminal_observation"]` 获取

包装器与高级用法详见 `references/vectorized_envs.md`。

### 4. 监控与控制回调

**目的：**
回调函数支持监控指标、保存检查点、实现早停和自定义训练逻辑，无需修改核心算法。

**常用回调：**
- **EvalCallback**：定期评估并保存最佳模型
- **CheckpointCallback**：间隔保存模型检查点
- **StopTrainingOnRewardThreshold**：达到目标奖励时停止训练
- **ProgressBarCallback**：显示带计时器的训练进度条

**自定义回调结构：**
```python
from stable_baselines3.common.callbacks import BaseCallback

class CustomCallback(BaseCallback):
    def _on_training_start(self):
        # 首次环境交互前调用
        pass

    def _on_step(self):
        # 每次环境步后调用
        # 返回 False 可停止训练
        return True

    def _on_rollout_end(self):
        # 环境交互结束时调用
        pass
```

**可用属性：**
- `self.model`：RL 算法实例
- `self.num_timesteps`：总环境步数
- `self.training_env`：训练环境

**回调链：**
```python
from stable_baselines3.common.callbacks import CallbackList

callback = CallbackList([eval_callback, checkpoint_callback, custom_callback])
model.learn(total_timesteps=10000, callback=callback)
```

完整回调文档见 `references/callbacks.md`。

### 5. 模型持久化与检查

**保存与加载：**
```python
# 保存模型
model.save("model_name")

# 保存归一化统计量（使用 VecNormalize 时）
vec_env.save("vec_normalize.pkl")

# 加载模型
model = PPO.load("model_name", env=env)

# 加载归一化统计量
vec_env = VecNormalize.load("vec_normalize.pkl", vec_env)
```

**参数访问：**
```python
# 获取参数
params = model.get_parameters()

# 设置参数
model.set_parameters(params)

# 访问 PyTorch 状态字典
state_dict = model.policy.state_dict()
```

### 6. 评估与记录

**评估：**
```python
from stable_baselines3.common.evaluation import evaluate_policy

mean_reward, std_reward = evaluate_policy(
    model,
    env,
    n_eval_episodes=10,
    deterministic=True
)
```

**视频录制：**
```python
from stable_baselines3.common.vec_env import VecVideoRecorder

# 用视频记录器包装环境
env = VecVideoRecorder(
    env,
    "videos/",
    record_video_trigger=lambda x: x % 2000 == 0,
    video_length=200
)
```

完整评估与录制模板见 `scripts/evaluate_agent.py`。

### 7. 高级功能

**学习率调度：**
```python
def linear_schedule(initial_value):
    def func(progress_remaining):
        # progress_remaining 从 1 递减至 0
        return progress_remaining * initial_value
    return func

model = PPO("MlpPolicy", env, learning_rate=linear_schedule(0.001))
```

**多输入策略（字典观测）：**
```python
model = PPO("MultiInputPolicy", env, verbose=1)
```
适用于字典类型观测（如图像与传感器数据结合）。

**事后经验回放：**
```python
from stable_baselines3 import SAC, HerReplayBuffer

model = SAC(
    "MultiInputPolicy",
    env,
    replay_buffer_class=HerReplayBuffer,
    replay_buffer_kwargs=dict(
        n_sampled_goal=4,
        goal_selection_strategy="future",
    ),
)
```

**TensorBoard 集成：**
```python
model = PPO("MlpPolicy", env, tensorboard_log="./tensorboard/")
model.learn(total_timesteps=10000)
```

## 工作流指南

**启动新 RL 项目：**

1. **问题定义**：明确观测空间、动作空间和奖励结构
2. **选择算法**：参考 `references/algorithms.md`
3. **创建/适配环境**：按需使用 `scripts/custom_env_template.py`
4. **环境验证**：训练前务必运行 `check_env()`
5. **训练配置**：以 `scripts/train_rl_agent.py` 为起点
6. **添加监控**：实现评估与检查点回调
7. **性能优化**：考虑使用向量化环境加速
8. **评估迭代**：使用 `scripts/evaluate_agent.py` 进行评估

**常见问题：**

- **内存错误**：减少离策略算法的 `buffer_size` 或减少并行环境数
- **训练缓慢**：考虑使用 SubprocVecEnv 并行环境
- **训练不稳定**：尝试不同算法、调参或检查奖励缩放
- **导入错误**：确保安装 `stable_baselines3`：`uv pip install stable-baselines3[extra]`

## 资源

### scripts/
- `train_rl_agent.py`：包含最佳实践的完整训练脚本模板
- `evaluate_agent.py`：智能体评估与视频录制模板
- `custom_env_template.py`：自定义 Gym 环境模板

### references/
- `algorithms.md`：详细算法对比与选择指南
- `custom_environments.md`：自定义环境创建综合指南
- `callbacks.md`：完整回调系统参考
- `vectorized_envs.md`：向量化环境用法与包装器

## 安装

```bash
# 基础安装
uv pip install stable-baselines3

# 含额外依赖（Tensorboard 等）
uv pip install stable-baselines3[extra]
```
