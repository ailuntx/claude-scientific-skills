# Stable Baselines3 回调系统

本文档提供关于 Stable Baselines3 中用于监控和控制训练的回调系统的全面信息。

## 概述

回调是在训练过程中特定节点调用的函数，用于：
- 监控训练指标
- 保存检查点
- 实现早停机制
- 记录自定义指标
- 动态调整超参数
- 触发评估过程

## 内置回调

### EvalCallback

定期评估智能体并保存最佳模型。

```python
from stable_baselines3.common.callbacks import EvalCallback

eval_callback = EvalCallback(
    eval_env,                                    # 独立的评估环境
    best_model_save_path="./logs/best_model/",  # 最佳模型保存路径
    log_path="./logs/eval/",                    # 评估日志保存路径
    eval_freq=10000,                            # 每N步评估一次
    n_eval_episodes=5,                          # 每次评估的回合数
    deterministic=True,                         # 使用确定性动作
    render=False,                               # 评估期间是否渲染
    verbose=1,
    warn=True,
)

model.learn(total_timesteps=100000, callback=eval_callback)
```

**主要特性：**
- 基于平均奖励自动保存最佳模型
- 将评估指标记录到 TensorBoard
- 达到奖励阈值时可停止训练

**重要提示：** 使用向量化训练环境时需调整 `eval_freq`：
```python
# 使用4个并行环境时，将eval_freq除以n_envs
eval_freq = 10000 // 4  # 每10000个总环境步数评估一次
```

### CheckpointCallback

定期保存模型检查点。

```python
from stable_baselines3.common.callbacks import CheckpointCallback

checkpoint_callback = CheckpointCallback(
    save_freq=10000,                     # 每N步保存一次
    save_path="./logs/checkpoints/",     # 检查点保存目录
    name_prefix="rl_model",              # 检查点文件前缀
    save_replay_buffer=True,             # 保存回放缓冲区（仅限离策略算法）
    save_vecnormalize=True,              # 保存VecNormalize统计信息
    verbose=2,
)

model.learn(total_timesteps=100000, callback=checkpoint_callback)
```

**输出文件：**
- `rl_model_10000_steps.zip` - 10k步时的模型
- `rl_model_20000_steps.zip` - 20k步时的模型
- 以此类推

**重要提示：** 向量化环境中需调整 `save_freq`（除以 n_envs）。

### StopTrainingOnRewardThreshold

当平均奖励超过阈值时停止训练。

```python
from stable_baselines3.common.callbacks import StopTrainingOnRewardThreshold

stop_callback = StopTrainingOnRewardThreshold(
    reward_threshold=200,  # 当平均奖励≥200时停止
    verbose=1,
)

# 必须与EvalCallback配合使用
eval_callback = EvalCallback(
    eval_env,
    callback_on_new_best=stop_callback,  # 发现新最佳模型时触发
    eval_freq=10000,
    n_eval_episodes=5,
)

model.learn(total_timesteps=1000000, callback=eval_callback)
```

### StopTrainingOnNoModelImprovement

如果模型连续N次评估未改进则停止训练。

```python
from stable_baselines3.common.callbacks import StopTrainingOnNoModelImprovement

stop_callback = StopTrainingOnNoModelImprovement(
    max_no_improvement_evals=10,  # 连续10次评估无改进则停止
    min_evals=20,                 # 触发停止的最小评估次数
    verbose=1,
)

# 与EvalCallback配合使用
eval_callback = EvalCallback(
    eval_env,
    callback_after_eval=stop_callback,
    eval_freq=10000,
)

model.learn(total_timesteps=1000000, callback=eval_callback)
```

### StopTrainingOnMaxEpisodes

达到最大回合数后停止训练。

```python
from stable_baselines3.common.callbacks import StopTrainingOnMaxEpisodes

stop_callback = StopTrainingOnMaxEpisodes(
    max_episodes=1000,  # 1000回合后停止
    verbose=1,
)

model.learn(total_timesteps=1000000, callback=stop_callback)
```

### ProgressBarCallback

训练期间显示进度条（需安装 tqdm）。

```python
from stable_baselines3.common.callbacks import ProgressBarCallback

progress_callback = ProgressBarCallback()

model.learn(total_timesteps=100000, callback=progress_callback)
```

**输出：**
```
100%|██████████| 100000/100000 [05:23<00:00, 309.31it/s]
```

## 创建自定义回调

### BaseCallback 结构

```python
from stable_baselines3.common.callbacks import BaseCallback

class CustomCallback(BaseCallback):
    """
    自定义回调模板
    """

    def __init__(self, verbose=0):
        super().__init__(verbose)
        # 自定义初始化

    def _init_callback(self) -> None:
        """
        训练开始时调用一次
        适用于需要访问模型/环境的初始化
        """
        pass

    def _on_training_start(self) -> None:
        """
        首次rollout开始前调用
        """
        pass

    def _on_rollout_start(self) -> None:
        """
        收集新样本前调用（同策略算法）
        """
        pass

    def _on_step(self) -> bool:
        """
        环境每步执行后调用

        返回值：
            bool: 返回False则停止训练
        """
        return True  # 继续训练

    def _on_rollout_end(self) -> None:
        """
        rollout结束后调用（同策略算法）
        """
        pass

    def _on_training_end(self) -> None:
        """
        训练结束时调用
        """
        pass
```

### 可用属性

回调函数内可访问：

- **`self.model`**: RL算法实例
- **`self.training_env`**: 训练环境
- **`self.n_calls`**: `_on_step()`调用次数
- **`self.num_timesteps`**: 环境步数总量
- **`self.locals`**: 算法局部变量（因算法而异）
- **`self.globals`**: 算法全局变量
- **`self.logger`**: TensorBoard/CSV日志记录器
- **`self.parent`**: 父回调（在CallbackList中使用时）

## 自定义回调示例

### 示例1：记录自定义指标

```python
class LogCustomMetricsCallback(BaseCallback):
    """
    将自定义指标记录到TensorBoard
    """

    def __init__(self, verbose=0):
        super().__init__(verbose)
        self.episode_rewards = []

    def _on_step(self) -> bool:
        # 检查回合是否结束
        if self.locals["dones"][0]:
            # 记录回合奖励
            episode_reward = self.locals["infos"][0].get("episode", {}).get("r", 0)
            self.episode_rewards.append(episode_reward)

            # 记录到TensorBoard
            self.logger.record("custom/episode_reward", episode_reward)
            self.logger.record("custom/mean_reward_last_100",
                             np.mean(self.episode_rewards[-100:]))

        return True
```

### 示例2：调整学习率

```python
class LinearScheduleCallback(BaseCallback):
    """
    训练期间线性降低学习率
    """

    def __init__(self, initial_lr=3e-4, final_lr=3e-5, verbose=0):
        super().__init__(verbose)
        self.initial_lr = initial_lr
        self.final_lr = final_lr

    def _on_step(self) -> bool:
        # 计算进度（0到1）
        progress = self.num_timesteps / self.locals["total_timesteps"]

        # 线性插值
        new_lr = self.initial_lr + (self.final_lr - self.initial_lr) * progress

        # 更新学习率
        for param_group in self.model.policy.optimizer.param_groups:
            param_group["lr"] = new_lr

        # 记录学习率
        self.logger.record("train/learning_rate", new_lr)

        return True
```

### 示例3：基于移动平均早停

```python
class EarlyStoppingCallback(BaseCallback):
    """
    当奖励移动平均无改进时停止训练
    """

    def __init__(self, check_freq=10000, min_reward=200, window=100, verbose=0):
        super().__init__(verbose)
        self.check_freq = check_freq
        self.min_reward = min_reward
        self.window = window
        self.rewards = []

    def _on_step(self) -> bool:
        # 收集回合奖励
        if self.locals["dones"][0]:
            reward = self.locals["infos"][0].get("episode", {}).get("r", 0)
            self.rewards.append(reward)

        # 每check_freq步检查一次
        if self.n_calls % self.check_freq == 0 and len(self.rewards) >= self.window:
            mean_reward = np.mean(self.rewards[-self.window:])
            if self.verbose > 0:
                print(f"平均奖励: {mean_reward:.2f}")

            if mean_reward >= self.min_reward:
                if self.verbose > 0:
                    print(f"停止：达到奖励阈值！")
                return False  # 停止训练

        return True  # 继续训练
```

### 示例4：按自定义指标保存最佳模型

```python
class SaveBestModelCallback(BaseCallback):
    """
    当自定义指标最优时保存模型
    """

    def __init__(self, check_freq=1000, save_path="./best_model/", verbose=0):
        super().__init__(verbose)
        self.check_freq = check_freq
        self.save_path = save_path
        self.best_score = -np.inf

    def _init_callback(self) -> None:
        if self.save_path is not None:
            os.makedirs(self.save_path, exist_ok=True)

    def _on_step(self) -> bool:
        if self.n_calls % self.check_freq == 0:
            # 计算自定义指标（示例：策略熵）
            custom_metric = self.locals.get("entropy_losses", [0])[-1]

            if custom_metric > self.best_score:
                self.best_score = custom_metric
                if self.verbose > 0:
                    print(f"新最佳记录！保存模型至 {self.save_path}")
                self.model.save(os.path.join(self.save_path, "best_model"))

        return True
```

### 示例5：记录环境特定信息

```python
class EnvironmentInfoCallback(BaseCallback):
    """
    记录环境中的自定义信息
    """

    def _on_step(self) -> bool:
        # 访问环境信息字典
        info = self.locals["infos"][0]

        # 记录环境自定义指标
        if "distance_to_goal" in info:
            self.logger.record("env/distance_to_goal", info["distance_to_goal"])

        if "success" in info:
            self.logger.record("env/success_rate", info["success"])

        return True
```

## 链式调用多个回调

使用 `CallbackList` 组合多个回调：

```python
from stable_baselines3.common.callbacks import CallbackList

callback_list = CallbackList([
    eval_callback,
    checkpoint_callback,
    progress_callback,
    custom_callback,
])

model.learn(total_timesteps=100000, callback=callback_list)
```

或直接传递列表：

```python
model.learn(
    total_timesteps=100000,
    callback=[eval_callback, checkpoint_callback, custom_callback]
)
```

## 基于事件的回调

回调可在特定事件触发其他回调：

```python
from stable_baselines3.common.callbacks import EventCallback

# 达到奖励阈值时停止训练
stop_callback = StopTrainingOnRewardThreshold(reward_threshold=200)

# 定期评估并在发现新最佳模型时触发停止回调
eval_callback = EvalCallback(
    eval_env,
    callback_on_new_best=stop_callback,  # 发现新最佳模型时触发
    eval_freq=10000,
)
```

## 记录到 TensorBoard

使用 `self.logger.record()` 记录指标：

```python
class TensorBoardCallback(BaseCallback):
    def _on_step(self) -> bool:
        # 记录标量
        self.logger.record("custom/my_metric", value)

        # 记录多个指标
        self.logger.record("custom/metric1", value1)
        self.logger.record("custom/metric2", value2)

        # 日志记录器自动写入TensorBoard
        return True
```

**在TensorBoard中查看：**
```bash
tensorboard --logdir ./logs/
```

## 高级模式

### 课程学习

```python
class CurriculumCallback(BaseCallback):
    """
    随时间增加任务难度
    """

    def __init__(self, difficulty_schedule, verbose=0):
        super().__init__(verbose)
        self.difficulty_schedule = difficulty_schedule

    def _on_step(self) -> bool:
        # 根据进度更新环境难度
        progress = self.num_timesteps / self.locals["total_timesteps"]

        for threshold, difficulty in self.difficulty_schedule:
            if progress >= threshold:
                self.training_env.env_method("set_difficulty", difficulty)

        return True
```

### 基于群体的训练

```python
class PopulationBasedCallback(BaseCallback):
    """
    根据性能调整超参数
    """

    def __init__(self, check_freq=10000, verbose=0):
        super().__init__(verbose)
        self.check_freq = check_freq
        self.performance_history = []

    def _on_step(self) -> bool:
        if self.n_calls % self.check_freq == 0:
            # 评估性能
            perf = self._evaluate_performance()
            self.performance_history.append(perf)

            # 性能停滞时调整超参数
            if len(self.performance_history) >= 3:
                recent = self.performance_history[-3:]
                if max(recent) - min(recent) < 0.01:  # 检测到停滞
                    self._adjust_hyperparameters()

        return True

    def _adjust_hyperparameters(self):
        # 示例：提高学习率
        for param_group in self.model.policy.optimizer.param_groups:
            param_group["lr"] *= 1.2
```

## 调试技巧

### 打印可用属性

```python
class DebugCallback(BaseCallback):
    def _on_step(self) -> bool:
        if self.n_calls == 1:
            print("self.locals中的可用属性：")
            for key in self.locals.keys():
                print(f"  {key}: {type(self.locals[key])}")
        return True
```

### 常见问题

1. **回调未被调用：**
   - 确保回调已传递给 `model.learn()`
   - 检查 `_on_step()` 是否返回 `True`

2. **回调中的属性错误：**
   - 并非所有属性在所有回调中都可用
   - 使用 `self.locals.get("key", default)` 确保安全

3. **内存泄漏：**
   - 不要在回调状态中存储大型数组
   - 定期清理缓冲区

4. **性能影响：**
   - 最小化 `_on_step()` 中的计算（每步调用）
   - 使用 `check_freq` 限制高开销操作

## 最佳实践

1. **选择合适的回调时机：**
   - `_on_step()`：用于每步变化的指标
   - `_on_rollout_end()`：用于基于rollout计算的指标
   - `_init_callback()`：用于一次性初始化

2. **高效记录日志：**
   - 避免每步记录（影响性能）
   - 聚合指标并定期记录

3. **处理向量化环境：**
   - 注意 `dones`、`inf
