---
name: simpy
description: 基于进程的Python离散事件仿真框架。当需要构建涉及流程、队列、资源和基于时间事件的系统仿真时使用此技能，例如制造系统、服务运营、网络流量、物流或任何实体随时间与共享资源交互的系统。
license: MIT 许可证
metadata:
    skill-author: K-Dense Inc.
---

# SimPy - 离散事件仿真

## 概述

SimPy 是基于标准 Python 的进程驱动离散事件仿真框架。使用 SimPy 对实体（客户、车辆、数据包等）随时间相互交互并竞争共享资源（服务器、机器、带宽等）的系统进行建模。

**核心能力：**
- 使用 Python 生成器函数进行流程建模
- 共享资源管理（服务器、容器、仓库）
- 事件驱动调度与同步
- 与挂钟时间同步的实时仿真
- 全面监控与数据收集

## 适用场景

在以下情况使用 SimPy 技能：

1. **离散事件系统建模** - 事件在非固定间隔发生的系统
2. **资源竞争** - 实体竞争有限资源（服务器、机器、人员）
3. **队列分析** - 研究等待队列、服务时间和吞吐量
4. **流程优化** - 分析制造、物流或服务流程
5. **网络仿真** - 数据包路由、带宽分配、延迟分析
6. **容量规划** - 确定满足性能需求的最佳资源水平
7. **系统验证** - 在实施前测试系统行为

**不适用于：**
- 固定时间步长的连续仿真（考虑 SciPy ODE 求解器）
- 无资源共享的独立流程
- 纯数学优化（考虑 SciPy 优化模块）

## 快速入门

### 基础仿真结构

```python
import simpy

def process(env, name):
    """等待并打印的简单流程"""
    print(f'{name} 开始于 {env.now}')
    yield env.timeout(5)
    print(f'{name} 结束于 {env.now}')

# 创建环境
env = simpy.Environment()

# 启动流程
env.process(process(env, '流程1'))
env.process(process(env, '流程2'))

# 运行仿真
env.run(until=10)
```

### 资源使用模式

```python
import simpy

def customer(env, name, resource):
    """客户请求资源，使用后释放"""
    with resource.request() as req:
        yield req  # 等待资源
        print(f'{name} 于 {env.now} 获取资源')
        yield env.timeout(3)  # 使用资源
        print(f'{name} 于 {env.now} 释放资源')

env = simpy.Environment()
server = simpy.Resource(env, capacity=1)

env.process(customer(env, '客户1', server))
env.process(customer(env, '客户2', server))
env.run()
```

## 核心概念

### 1. 环境

仿真环境管理时间并调度事件。

```python
import simpy

# 标准环境（全速运行）
env = simpy.Environment(initial_time=0)

# 实时环境（与挂钟同步）
import simpy.rt
env_rt = simpy.rt.RealtimeEnvironment(factor=1.0)

# 运行仿真
env.run(until=100)  # 运行至时间100
env.run()  # 运行至无事件剩余
```

### 2. 进程

进程通过 Python 生成器函数（含 `yield` 语句的函数）定义。

```python
def my_process(env, param1, param2):
    """通过 yield 暂停执行的进程"""
    print(f'开始于 {env.now}')

    # 等待时间流逝
    yield env.timeout(5)

    print(f'恢复于 {env.now}')

    # 等待其他事件
    yield env.timeout(3)

    print(f'结束于 {env.now}')
    return '结果'

# 启动进程
env.process(my_process(env, '值1', '值2'))
```

### 3. 事件

事件是进程同步的基础机制。进程通过 yield 挂起，在事件触发时恢复。

**常用事件类型：**
- `env.timeout(delay)` - 等待时间流逝
- `resource.request()` - 请求资源
- `env.event()` - 创建自定义事件
- `env.process(func())` - 将进程作为事件
- `event1 & event2` - 等待所有事件完成（AllOf）
- `event1 | event2` - 等待任一事件完成（AnyOf）

## 资源

SimPy 为不同场景提供多种资源类型。完整说明见 `references/resources.md`。

### 资源类型概览

| 资源类型         | 适用场景                     |
|------------------|------------------------------|
| Resource         | 容量受限资源（服务器、机器） |
| PriorityResource | 优先级队列                   |
| PreemptiveResource | 高优先级可抢占低优先级     |
| Container        | 大宗物料（燃料、水）         |
| Store            | Python 对象存储（FIFO）      |
| FilterStore      | 选择性物品检索               |
| PriorityStore    | 优先级排序物品               |

### 速查示例

```python
import simpy

env = simpy.Environment()

# 基础资源（如服务器）
resource = simpy.Resource(env, capacity=2)

# 优先级资源
priority_resource = simpy.PriorityResource(env, capacity=1)

# 容器（如油箱）
fuel_tank = simpy.Container(env, capacity=100, init=50)

# 仓库（如存储区）
warehouse = simpy.Store(env, capacity=10)
```

## 常用仿真模式

### 模式1：客户-服务器队列

```python
import simpy
import random

def customer(env, name, server):
    arrival = env.now
    with server.request() as req:
        yield req
        wait = env.now - arrival
        print(f'{name} 等待 {wait:.2f}，于 {env.now} 接受服务')
        yield env.timeout(random.uniform(2, 4))

def customer_generator(env, server):
    i = 0
    while True:
        yield env.timeout(random.uniform(1, 3))
        i += 1
        env.process(customer(env, f'客户 {i}', server))

env = simpy.Environment()
server = simpy.Resource(env, capacity=2)
env.process(customer_generator(env, server))
env.run(until=20)
```

### 模式2：生产者-消费者

```python
import simpy

def producer(env, store):
    item_id = 0
    while True:
        yield env.timeout(2)
        item = f'物品 {item_id}'
        yield store.put(item)
        print(f'于 {env.now} 生产 {item}')
        item_id += 1

def consumer(env, store):
    while True:
        item = yield store.get()
        print(f'于 {env.now} 消费 {item}')
        yield env.timeout(3)

env = simpy.Environment()
store = simpy.Store(env, capacity=10)
env.process(producer(env, store))
env.process(consumer(env, store))
env.run(until=20)
```

### 模式3：并行任务执行

```python
import simpy

def task(env, name, duration):
    print(f'{name} 于 {env.now} 启动')
    yield env.timeout(duration)
    print(f'{name} 于 {env.now} 完成')
    return f'{name} 结果'

def coordinator(env):
    # 并行启动任务
    task1 = env.process(task(env, '任务1', 5))
    task2 = env.process(task(env, '任务2', 3))
    task3 = env.process(task(env, '任务3', 4))

    # 等待全部完成
    results = yield task1 & task2 & task3
    print(f'全部完成于 {env.now}')

env = simpy.Environment()
env.process(coordinator(env))
env.run()
```

## 工作流指南

### 步骤1：定义系统

识别：
- **实体**：系统中流动的对象？（客户、零件、数据包）
- **资源**：约束条件是什么？（服务器、机器、带宽）
- **流程**：关键活动有哪些？（到达、服务、离开）
- **指标**：需测量什么？（等待时间、利用率、吞吐量）

### 步骤2：实现流程函数

为每类流程创建生成器函数：

```python
def entity_process(env, name, resources, parameters):
    # 到达逻辑
    arrival_time = env.now

    # 请求资源
    with resource.request() as req:
        yield req

        # 服务逻辑
        service_time = calculate_service_time(parameters)
        yield env.timeout(service_time)

    # 离开逻辑
    collect_statistics(env.now - arrival_time)
```

### 步骤3：设置监控

使用监控工具收集数据。完整技术见 `references/monitoring.md`。

```python
from scripts.resource_monitor import ResourceMonitor

# 创建并监控资源
resource = simpy.Resource(env, capacity=2)
monitor = ResourceMonitor(env, resource, "服务器")

# 仿真结束后
monitor.report()
```

### 步骤4：运行与分析

```python
# 运行仿真
env.run(until=simulation_time)

# 生成报告
monitor.report()
stats.report()

# 导出数据供后续分析
monitor.export_csv('results.csv')
```

## 高级功能

### 进程交互

进程可通过事件、进程挂起和中断进行交互。详细模式见 `references/process-interaction.md`。

**关键机制：**
- **事件信号**：通过共享事件协调
- **进程挂起**：等待其他进程完成
- **中断**：强制恢复进程以实现抢占

### 实时仿真

通过挂钟时间同步仿真，适用于硬件在环或交互式应用。见 `references/real-time.md`。

```python
import simpy.rt

env = simpy.rt.RealtimeEnvironment(factor=1.0)  # 1:1 时间映射
# factor=0.5 表示 1 个仿真单位 = 0.5 秒（2倍速）
```

### 全面监控

监控进程、资源和事件。技术包括（详见 `references/monitoring.md`）：
- 状态变量追踪
- 资源猴子补丁
- 事件追踪
- 统计收集

## 脚本与模板

### basic_simulation_template.py

队列仿真的完整模板，包含：
- 可配置参数
- 统计收集
- 客户生成
- 资源使用
- 报告生成

**用法：**
```python
from scripts.basic_simulation_template import SimulationConfig, run_simulation

config = SimulationConfig()
config.num_resources = 2
config.sim_time = 100
stats = run_simulation(config)
stats.report()
```

### resource_monitor.py

可复用的监控工具：
- `ResourceMonitor` - 追踪单一资源
- `MultiResourceMonitor` - 监控多个资源
- `ContainerMonitor` - 追踪容器液位
- 自动统计计算
- CSV 导出功能

**用法：**
```python
from scripts.resource_monitor import ResourceMonitor

monitor = ResourceMonitor(env, resource, "我的资源")
# ...运行仿真...
monitor.report()
monitor.export_csv('data.csv')
```

## 参考文档

专题详细指南：
- **`references/resources.md`** - 所有资源类型及示例
- **`references/events.md`** - 事件系统与模式
- **`references/process-interaction.md`** - 进程同步
- **`references/monitoring.md`** - 数据收集技术
- **`references/real-time.md`** - 实时仿真设置

## 最佳实践

1. **生成器函数**：进程函数始终使用 `yield`
2. **资源上下文管理器**：使用 `with resource.request() as req:` 自动清理
3. **结果可复现**：设置 `random.seed()` 保持结果一致
4. **监控**：在仿真全程收集数据，而非仅结束时
5. **验证**：将简单案例与解析解对比
6. **文档化**：注释流程逻辑和参数选择
7. **模块化设计**：分离流程逻辑、统计和配置

## 常见陷阱

1. **遗漏 yield**：进程必须 yield 事件才能暂停
2. **事件重用**：事件只能触发一次
3. **资源泄漏**：使用上下文管理器或确保释放
4. **阻塞操作**：避免在进程中使用 Python 阻塞调用
5. **时间单位**：保持时间单位解释的一致性
6. **死锁**：确保至少一个进程能推进

## 应用案例

- **制造业**：机器调度、生产线、库存管理
- **医疗**：急诊室仿真、患者流、人员配置
- **电信**：网络流量、数据包路由、带宽分配
- **交通**：车流模拟、物流、车辆路径规划
- **服务运营**：呼叫中心、零售收银、预约调度
- **计算机系统**：CPU调度、内存管理、I/O操作
