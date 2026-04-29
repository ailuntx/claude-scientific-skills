# PufferLib 向量化指南

## 概述

PufferLib 的向量化系统通过受 EnvPool 启发的优化实现，支持高性能并行环境模拟，每秒可处理数百万步操作。该系统以最小开销同时支持同步和异步向量化。

## 向量化架构

### 关键优化

1. **共享内存缓冲区**：跨所有环境使用单一统一缓冲区（与 Gymnasium 的每环境缓冲区不同）
2. **忙等待标志**：工作线程在未锁定标志上忙等待，而非使用管道/队列
3. **零拷贝批处理**：连续工作子集返回观测值无需复制
4. **超额环境**：模拟数量超过批次大小的环境以实现异步返回
5. **多环境共工作线程**：优化轻量级环境性能

### 性能特征

- **纯 Python 环境**：10万-50万步/秒
- **C 语言环境**：1亿+步/秒
- **结合训练**：40万-400万总步/秒
- **向量化开销**：优化配置下<5%

## 创建向量化环境

### 基础向量化

```python
import pufferlib

# 自动向量化
env = pufferlib.make('environment_name', num_envs=256)

# 显式配置
env = pufferlib.make(
    'environment_name',
    num_envs=256,
    num_workers=8,
    envs_per_worker=32
)
```

### 手动向量化

```python
from pufferlib import PufferEnv
from pufferlib.vectorization import Serial, Multiprocessing

# 串行向量化（单进程）
vec_env = Serial(
    env_creator=lambda: MyEnvironment(),
    num_envs=16
)

# 多进程向量化
vec_env = Multiprocessing(
    env_creator=lambda: MyEnvironment(),
    num_envs=256,
    num_workers=8
)
```

## 向量化模式

### 串行向量化

适用于调试和轻量级环境：

```python
from pufferlib.vectorization import Serial

vec_env = Serial(
    env_creator=env_creator_fn,
    num_envs=16
)

# 所有环境在主进程运行
# 无多进程开销
# 使用标准工具更易调试
```

**适用场景：**
- 开发与调试
- 极快环境（<1μs/步）
- 少量环境（<32）
- 单线程性能分析

### 多进程向量化

适用于多数生产场景：

```python
from pufferlib.vectorization import Multiprocessing

vec_env = Multiprocessing(
    env_creator=env_creator_fn,
    num_envs=256,
    num_workers=8,
    envs_per_worker=32
)

# 跨工作线程并行执行
# 实现CPU密集型环境真正并行
# 可扩展至数百环境
```

**适用场景：**
- 生产环境训练
- CPU密集型环境
- 大规模并行模拟
- 最大化吞吐量

### 异步向量化

适用于步长时间可变的环境：

```python
vec_env = Multiprocessing(
    env_creator=env_creator_fn,
    num_envs=256,
    num_workers=8,
    mode='async',
    surplus_envs=32  # 模拟额外环境
)

# 就绪后立即返回批次
# 提升GPU利用率
# 处理环境速度差异
```

**适用场景：**
- 环境步长时间可变
- 最大化GPU利用率
- 基于网络的环境
- 外部模拟器

## 优化向量化性能

### 工作线程配置

```python
import multiprocessing

# 计算最优工作线程数
num_cpus = multiprocessing.cpu_count()

# 保守策略（为训练预留资源）
num_workers = num_cpus - 2

# 激进策略（最大化环境吞吐）
num_workers = num_cpus

# 启用超线程时
num_workers = num_cpus // 2  # 仅使用物理核心
```

### 每工作线程环境数

```python
# 快速环境（<10μs/步）
envs_per_worker = 64  # 增加每线程环境数

# 中等环境（10-100μs/步）
envs_per_worker = 32  # 平衡配置

# 慢速环境（>100μs/步）
envs_per_worker = 16  # 减少每线程环境数

# 根据目标批次大小计算
batch_size = 32768
num_workers = 8
envs_per_worker = batch_size // num_workers
```

### 批次大小调优

```python
# 小批次（<8k）：适合快速迭代
batch_size = 4096
num_envs = 256
steps_per_env = batch_size // num_envs  # 16步

# 中等批次（8k-32k）：平衡选择
batch_size = 16384
num_envs = 512
steps_per_env = 32

# 大批次（>32k）：最大化吞吐
batch_size = 65536
num_envs = 1024
steps_per_env = 64
```

## 共享内存优化

### 缓冲区管理

PufferLib 使用共享内存实现零拷贝观测传递：

```python
import numpy as np
from multiprocessing import shared_memory

class OptimizedEnv(PufferEnv):
    def __init__(self, buf=None):
        super().__init__(buf)

        # 环境使用提供的共享缓冲区
        self.observation_space = self.make_space({'obs': (84, 84, 3)})

        # 观测值直接写入共享内存
        self._obs_buffer = None

    def reset(self):
        # 原地写入共享内存
        if self._obs_buffer is None:
            self._obs_buffer = np.zeros((84, 84, 3), dtype=np.uint8)

        self._render_to_buffer(self._obs_buffer)
        return {'obs': self._obs_buffer}

    def step(self, action):
        # 仅进行原地更新
        self._update_state(action)
        self._render_to_buffer(self._obs_buffer)

        return {'obs': self._obs_buffer}, reward, done, info
```

### 零拷贝模式

```python
# 错误：产生拷贝
def get_observation(self):
    obs = np.zeros((84, 84, 3))
    # ... 填充观测值 ...
    return obs.copy()  # 不必要的拷贝！

# 正确：重用缓冲区
def get_observation(self):
    # 使用预分配缓冲区
    self._render_to_buffer(self._obs_buffer)
    return self._obs_buffer  # 无拷贝

# 错误：分配新数组
def step(self, action):
    new_state = self.state + action  # 分配内存
    self.state = new_state
    return obs, reward, done, info

# 正确：原地操作
def step(self, action):
    self.state += action  # 原地操作
    return obs, reward, done, info
```

## 高级向量化

### 自定义向量化

```python
from pufferlib.vectorization import VectorEnv

class CustomVectorEnv(VectorEnv):
    """自定义向量化实现"""

    def __init__(self, env_creator, num_envs, **kwargs):
        super().__init__()

        self.envs = [env_creator() for _ in range(num_envs)]
        self.num_envs = num_envs

    def reset(self):
        """重置所有环境"""
        observations = [env.reset() for env in self.envs]
        return self._stack_obs(observations)

    def step(self, actions):
        """推进所有环境"""
        results = [env.step(action) for env, action in zip(self.envs, actions)]

        obs, rewards, dones, infos = zip(*results)

        return (
            self._stack_obs(obs),
            np.array(rewards),
            np.array(dones),
            list(infos)
        )

    def _stack_obs(self, observations):
        """将观测值堆叠为批次"""
        return np.stack(observations, axis=0)
```

### 分层向量化

适用于超大规模并行：

```python
# 外层：多进程向量化（8工作线程）
# 内层：每个工作线程运行串行向量化（32环境）
# 总计：256并行环境

def create_serial_vec_env():
    return Serial(
        env_creator=lambda: MyEnvironment(),
        num_envs=32
    )

outer_vec_env = Multiprocessing(
    env_creator=create_serial_vec_env,
    num_envs=8,  # 8个串行向量环境
    num_workers=8
)

# 总环境数：8 * 32 = 256
```

## 多智能体向量化

### 原生多智能体支持

PufferLib 将多智能体环境视为一等公民：

```python
# 自动向量化多智能体环境
env = pufferlib.make(
    'pettingzoo-knights-archers-zombies',
    num_envs=128,
    num_agents=4
)

# 观测值：每个智能体的{agent_id: [batch_obs]}
# 动作：每个智能体的{agent_id: [batch_actions]}
# 奖励：每个智能体的{agent_id: [batch_rewards]}
```

### 自定义多智能体向量化

```python
class MultiAgentVectorEnv(VectorEnv):
    def step(self, actions):
        """
        参数：
            actions: 动作字典 {agent_id: [batch_actions]}

        返回：
            observations: 观测字典 {agent_id: [batch_obs]}
            rewards: 奖励字典 {agent_id: [batch_rewards]}
            dones: 终止字典 {agent_id: [batch_dones]}
            infos: 信息字典列表
        """
        # 将动作分配到各环境
        env_actions = self._distribute_actions(actions)

        # 推进每个环境
        results = [env.step(act) for env, act in zip(self.envs, env_actions)]

        # 收集并批处理结果
        return self._batch_results(results)
```

## 性能监控

### 向量化性能分析

```python
import time

def profile_vectorization(vec_env, num_steps=10000):
    """分析向量化性能"""
    start = time.time()

    vec_env.reset()

    for _ in range(num_steps):
        actions = vec_env.action_space.sample()
        vec_env.step(actions)

    elapsed = time.time() - start
    sps = (num_steps * vec_env.num_envs) / elapsed

    print(f"每秒步数: {sps:,.0f}")
    print(f"单步耗时: {elapsed/num_steps*1000:.2f}毫秒")

    return sps
```

### 瓶颈分析

```python
import cProfile
import pstats

def analyze_bottlenecks(vec_env):
    """识别向量化瓶颈"""
    profiler = cProfile.Profile()

    profiler.enable()

    vec_env.reset()
    for _ in range(1000):
        actions = vec_env.action_space.sample()
        vec_env.step(actions)

    profiler.disable()

    stats = pstats.Stats(profiler)
    stats.sort_stats('cumulative')
    stats.print_stats(20)
```

### 实时监控

```python
class MonitoredVectorEnv(VectorEnv):
    """带性能监控的向量环境"""

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self.step_times = []
        self.step_count = 0

    def step(self, actions):
        start = time.perf_counter()

        result = super().step(actions)

        elapsed = time.perf_counter() - start
        self.step_times.append(elapsed)
        self.step_count += 1

        # 每1000步记录
        if self.step_count % 1000 == 0:
            mean_time = np.mean(self.step_times[-1000:])
            sps = self.num_envs / mean_time
            print(f"步/秒: {sps:,.0f} | 单步耗时: {mean_time*1000:.2f}毫秒")

        return result
```

## 故障排除

### 低吞吐量

```python
# 检查配置
print(f"环境数: {vec_env.num_envs}")
print(f"工作线程数: {vec_env.num_workers}")
print(f"每线程环境数: {vec_env.num_envs // vec_env.num_workers}")

# 分析单环境性能
single_env = MyEnvironment()
single_sps = profile_single_env(single_env)
print(f"单环境步/秒: {single_sps:,.0f}")

# 对比向量化性能
vec_sps = profile_vectorization(vec_env)
print(f"向量化步/秒: {vec_sps:,.0f}")
print(f"加速比: {vec_sps / single_sps:.1f}倍")
```

### 内存问题

```python
# 减少环境数量
num_envs = 128  # 替代256

# 减少每线程环境数
envs_per_worker = 16  # 替代32

# 使用串行模式调试
vec_env = Serial(env_creator, num_envs=16)
```

### 同步问题

```python
# 确保线程安全操作
import threading

class ThreadSafeEnv(PufferEnv):
    def __init__(self, buf=None):
        super().__init__(buf)
        self.lock = threading.Lock()

    def step(self, action):
        with self.lock:
            return super().step(action)
```

## 最佳实践

### 配置指南

```python
# 初始保守配置
config = {
    'num_envs': 64,
    'num_workers': 4,
    'envs_per_worker': 16
}

# 迭代扩展
config = {
    'num_envs': 256,     # 4倍增长
    'num_workers': 8,     # 2倍增长
    'envs_per_worker': 32 # 2倍增长
}

# 监控并调整
if sps < target_sps:
    # 尝试增加环境数或工作线程数
    pass
if memory_usage > threshold:
    # 减少环境数或每线程环境数
    pass
```

### 环境设计

```python
# 最小化每步内存分配
class EfficientEnv(PufferEnv):
    def __init__(self, buf=None):
        super().__init__(buf)

        # 预分配所有缓冲区
        self._obs = np.zeros((84, 84, 3), dtype=np.uint8)
        self._state = np.zeros(10, dtype=np.float32)

    def step(self, action):
        # 使用预分配缓冲区
        self._update_state_inplace(action)
        self._render_to_obs()

        return self._obs, reward, done, info
```

### 测试验证

```python
# 验证向量化与串行结果一致
serial_env = Serial(env_creator, num_envs=4)
vec_env = Multiprocessing(env_creator, num_envs=4, num_workers=2)

# 并行运行并验证结果匹配
serial_env.seed(42)
vec_env.seed(42)

serial_obs = serial_env.reset()
vec_obs = vec_env.reset()

assert np.allclose(serial_obs, vec_obs), "向量化结果不匹配！"
```
