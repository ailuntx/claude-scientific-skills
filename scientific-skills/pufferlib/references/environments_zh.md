# PufferLib 环境指南

## 概述

PufferLib 提供用于创建高性能自定义环境的 PufferEnv API，以及包含 20+ 预构建环境的 Ocean 套件。所有环境均原生支持单智能体与多智能体场景的向量化操作。

## PufferEnv API

### 核心特性

PufferEnv 通过原地操作实现高性能设计：
- 观测值、动作和奖励从共享缓冲区对象初始化
- 所有操作均为原地执行，避免创建和复制数组
- 原生支持单智能体与多智能体环境
- 扁平化的观测/动作空间以实现高效向量化

### 创建 PufferEnv

```python
import numpy as np
import pufferlib
from pufferlib import PufferEnv

class MyEnvironment(PufferEnv):
    def __init__(self, buf=None):
        super().__init__(buf)

        # 定义观测空间和动作空间
        self.observation_space = self.make_space({
            'image': (84, 84, 3),
            'vector': (10,)
        })

        self.action_space = self.make_discrete(4)  # 4个离散动作

        # 初始化状态
        self.reset()

    def reset(self):
        """重置环境到初始状态"""
        # 重置内部状态
        self.agent_pos = np.array([0, 0])
        self.step_count = 0

        # 返回初始观测值
        obs = {
            'image': np.zeros((84, 84, 3), dtype=np.uint8),
            'vector': np.zeros(10, dtype=np.float32)
        }

        return obs

    def step(self, action):
        """执行单步环境操作"""
        # 根据动作更新状态
        self.step_count += 1

        # 计算奖励
        reward = self._compute_reward()

        # 检查回合是否结束
        done = self.step_count >= 1000

        # 生成观测值
        obs = self._get_observation()

        # 附加信息
        info = {'episode': {'r': reward, 'l': self.step_count}} if done else {}

        return obs, reward, done, info

    def _compute_reward(self):
        """计算当前状态奖励"""
        return 1.0

    def _get_observation(self):
        """从当前状态生成观测值"""
        return {
            'image': np.random.randint(0, 256, (84, 84, 3), dtype=np.uint8),
            'vector': np.random.randn(10).astype(np.float32)
        }
```

### 观测空间

#### 离散空间

```python
# 单离散值
self.observation_space = self.make_discrete(10)  # 取值范围0-9

# 包含离散值的字典
self.observation_space = self.make_space({
    'position': (1,),  # 连续值
    'type': self.make_discrete(5)  # 离散值
})
```

#### 连续空间

```python
# Box空间（连续）
self.observation_space = self.make_space({
    'image': (84, 84, 3),      # 图像
    'vector': (10,),            # 向量
    'scalar': (1,)              # 单值
})
```

#### 多离散空间

```python
# 多离散值
self.observation_space = self.make_multi_discrete([3, 5, 2])  # 3个值/5个值/2个值
```

### 动作空间

```python
# 离散动作
self.action_space = self.make_discrete(4)  # 4个动作: 0,1,2,3

# 连续动作
self.action_space = self.make_space((3,))  # 3维连续动作

# 多离散动作
self.action_space = self.make_multi_discrete([3, 3])  # 两个3选1离散选择
```

## 多智能体环境

PufferLib 原生支持多智能体环境，统一处理单智能体和多智能体场景。

### 多智能体 PufferEnv

```python
class MultiAgentEnv(PufferEnv):
    def __init__(self, num_agents=4, buf=None):
        super().__init__(buf)

        self.num_agents = num_agents

        # 单智能体观测空间
        self.single_observation_space = self.make_space({
            'position': (2,),
            'velocity': (2,),
            'global': (10,)
        })

        # 单智能体动作空间
        self.single_action_space = self.make_discrete(5)

        self.reset()

    def reset(self):
        """重置所有智能体"""
        self.agents = {f'agent_{i}': Agent(i) for i in range(self.num_agents)}

        # 返回所有智能体的观测值
        return {
            agent_id: self._get_obs(agent)
            for agent_id, agent in self.agents.items()
        }

    def step(self, actions):
        """执行所有智能体的动作"""
        # actions是字典: {agent_id: action}
        observations = {}
        rewards = {}
        dones = {}
        infos = {}

        for agent_id, action in actions.items():
            agent = self.agents[agent_id]

            # 更新智能体
            agent.update(action)

            # 生成结果
            observations[agent_id] = self._get_obs(agent)
            rewards[agent_id] = self._compute_reward(agent)
            dones[agent_id] = agent.is_done()
            infos[agent_id] = {}

        # 检查全局终止条件
        dones['__all__'] = all(dones.values())

        return observations, rewards, dones, infos
```

## Ocean 环境套件

PufferLib 提供包含 20+ 预构建环境的 Ocean 套件：

### 可用环境

#### 街机游戏
- **Atari**: 通过 Arcade Learning Environment 的经典 Atari 2600 游戏
- **Procgen**: 用于泛化测试的程序生成游戏

#### 网格世界
- **Minigrid**: 部分可观测的网格世界环境
- **Crafter**: 开放式生存制作游戏
- **NetHack**: 经典 Roguelike 地牢探索游戏
- **MiniHack**: 简化版 NetHack 变体

#### 多智能体
- **PettingZoo**: 多智能体环境套件（含 Butterfly）
- **MAgent**: 大规模多智能体场景
- **Neural MMO**: 海量多智能体生存游戏

#### 专项环境
- **Pokemon Red**: 经典口袋妖怪游戏环境
- **GPUDrive**: 高性能驾驶模拟器
- **Griddly**: 基于网格的游戏引擎
- **MicroRTS**: 即时战略游戏

### 使用 Ocean 环境

```python
import pufferlib

# 创建环境
env = pufferlib.make('procgen-coinrun', num_envs=256)

# 自定义配置
env = pufferlib.make(
    'atari-pong',
    num_envs=128,
    frameskip=4,
    framestack=4
)

# 多智能体环境
env = pufferlib.make('pettingzoo-knights-archers-zombies', num_agents=4)
```

## 自定义环境开发

### 开发流程

1. **Python 原型开发**: 从纯 Python PufferEnv 开始
2. **关键路径优化**: 识别性能瓶颈
3. **C 语言实现**: 用 C 重写性能关键代码
4. **创建绑定**: 使用 Python C API
5. **编译**: 构建扩展模块
6. **注册**: 添加到 Ocean 套件

### 性能基准

- **纯 Python**: 10万-50万步/秒
- **C 实现**: 1亿+步/秒
- **Python 环境训练**: 约40万总步数/秒
- **C 环境训练**: 约400万总步数/秒

### Python 优化技巧

```python
# 使用 NumPy 操作替代 Python 循环
# 错误示范
for i in range(len(array)):
    array[i] = array[i] * 2

# 正确示范
array *= 2

# 预分配数组而非追加
# 错误示范
observations = []
for i in range(n):
    observations.append(generate_obs())

# 正确示范
observations = np.empty((n, obs_shape), dtype=np.float32)
for i in range(n):
    observations[i] = generate_obs()

# 使用原地操作
# 错误示范
new_state = state + delta

# 正确示范
state += delta
```

### C 扩展示例

```c
// my_env.c
#include <Python.h>
#include <numpy/arrayobject.h>

// 快速环境步进实现
static PyObject* fast_step(PyObject* self, PyObject* args) {
    PyArrayObject* state;
    int action;

    if (!PyArg_ParseTuple(args, "O!i", &PyArray_Type, &state, &action)) {
        return NULL;
    }

    // 高性能 C 实现
    // ...

    return Py_BuildValue("Ofi", obs, reward, done);
}

static PyMethodDef methods[] = {
    {"fast_step", fast_step, METH_VARARGS, "快速环境步进"},
    {NULL, NULL, 0, NULL}
};

static struct PyModuleDef module = {
    PyModuleDef_HEAD_INIT,
    "my_env_c",
    NULL,
    -1,
    methods
};

PyMODINIT_FUNC PyInit_my_env_c(void) {
    import_array();
    return PyModule_Create(&module);
}
```

## 第三方环境集成

### Gymnasium 环境

```python
import gymnasium as gym
import pufferlib

# 封装 Gymnasium 环境
gym_env = gym.make('CartPole-v1')
puffer_env = pufferlib.emulate(gym_env, num_envs=256)

# 或直接使用 make
env = pufferlib.make('gym-CartPole-v1', num_envs=256)
```

### PettingZoo 环境

```python
from pettingzoo.butterfly import pistonball_v6
import pufferlib

# 封装 PettingZoo 环境
pz_env = pistonball_v6.env()
puffer_env = pufferlib.emulate(pz_env, num_envs=128)

# 或直接使用 make
env = pufferlib.make('pettingzoo-pistonball', num_envs=128)
```

### 自定义封装器

```python
class CustomWrapper(pufferlib.PufferEnv):
    """修改环境行为的封装器"""

    def __init__(self, base_env, buf=None):
        super().__init__(buf)
        self.base_env = base_env
        self.observation_space = base_env.observation_space
        self.action_space = base_env.action_space

    def reset(self):
        obs = self.base_env.reset()
        # 修改观测值
        return self._process_obs(obs)

    def step(self, action):
        # 修改动作
        modified_action = self._process_action(action)

        obs, reward, done, info = self.base_env.step(modified_action)

        # 修改输出
        obs = self._process_obs(obs)
        reward = self._process_reward(reward)

        return obs, reward, done, info
```

## 环境最佳实践

### 状态管理

```python
# 存储最小状态，按需计算
class EfficientEnv(PufferEnv):
    def __init__(self, buf=None):
        super().__init__(buf)
        self.agent_pos = np.zeros(2)  # 最小状态

    def _get_observation(self):
        # 按需计算完整观测值
        observation = np.zeros((84, 84, 3), dtype=np.uint8)
        self._render_scene(observation, self.agent_pos)
        return observation
```

### 奖励缩放

```python
# 将奖励归一化到合理范围
def step(self, action):
    # ... 环境逻辑 ...

    # 缩放高额奖励
    raw_reward = compute_raw_reward()
    reward = np.clip(raw_reward / 100.0, -10, 10)

    return obs, reward, done, info
```

### 回合终止

```python
def step(self, action):
    # ... 环境逻辑 ...

    # 多重终止条件
    timeout = self.step_count >= self.max_steps
    success = self._check_success()
    failure = self._check_failure()

    done = timeout or success or failure

    info = {
        'TimeLimit.truncated': timeout,
        'success': success
    }

    return obs, reward, done, info
```

### 内存效率

```python
# 复用缓冲区而非新建
class MemoryEfficientEnv(PufferEnv):
    def __init__(self, buf=None):
        super().__init__(buf)

        # 预分配观测缓冲区
        self._obs_buffer = np.zeros((84, 84, 3), dtype=np.uint8)

    def _get_observation(self):
        # 复用缓冲区，原地修改
        self._render_scene(self._obs_buffer)
        return self._obs_buffer  # 返回视图而非副本
```

## 环境调试

### 验证检查

```python
# 添加断言捕获错误
def step(self, action):
    assert self.action_space.contains(action), f"无效动作: {action}"

    obs, reward, done, info = self._step_impl(action)

    assert self.observation_space.contains(obs), "无效观测值"
    assert np.isfinite(reward), "非有限奖励值"

    return obs, reward, done, info
```

### 渲染功能

```python
class DebuggableEnv(PufferEnv):
    def __init__(self, buf=None, render_mode=None):
        super().__init__(buf)
        self.render_mode = render_mode

    def render(self):
        """调试用环境渲染"""
        if self.render_mode == 'human':
            # 屏幕显示
            self._display_scene()
        elif self.render_mode == 'rgb_array':
            # 返回图像
            return self._render_to_array()
```

### 日志记录

```python
import logging

logger = logging.getLogger(__name__)

def step(self, action):
    logger.debug(f"步进 {self.step_count}: 动作={action}")

    obs, reward, done, info = self._step_impl(action)

    if done:
        logger.info(f"回合结束: 奖励={self.total_reward}")

    return obs, reward, done, info
```
