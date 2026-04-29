# SimPy 事件系统

本指南涵盖 SimPy 中的事件系统，它是离散事件仿真的基础。

## 事件基础

事件是控制仿真流程的核心机制。进程通过生成事件并在事件触发时恢复执行。

### 事件生命周期

事件经历三种状态：

1. **未触发** - 作为内存对象的初始状态
2. **已触发** - 被调度到事件队列中；`triggered`属性为`True`
3. **已处理** - 从队列中移除并执行回调；`processed`属性为`True`

```python
import simpy

env = simpy.Environment()

# 创建事件
event = env.event()
print(f'Triggered: {event.triggered}, Processed: {event.processed}')  # 均为False

# 触发事件
event.succeed(value='事件结果')
print(f'Triggered: {event.triggered}, Processed: {event.processed}')  # True, False

# 运行以处理事件
env.run()
print(f'Triggered: {event.triggered}, Processed: {event.processed}')  # True, True
print(f'Value: {event.value}')  # '事件结果'
```

## 核心事件类型

### 超时事件

控制仿真中的时间推进。最常见的事件类型。

```python
import simpy

def process(env):
    print(f'开始于 {env.now}')
    yield env.timeout(5)
    print(f'恢复于 {env.now}')

    # 带返回值的超时
    result = yield env.timeout(3, value='完成')
    print(f'结果: {result} 于 {env.now}')

env = simpy.Environment()
env.process(process(env))
env.run()
```

**用法：**
- `env.timeout(delay)` - 等待指定时间
- `env.timeout(delay, value=val)` - 等待并返回值

### 进程事件

进程本身可作为事件，允许进程等待其他进程完成。

```python
import simpy

def worker(env, name, duration):
    print(f'{name} 开始于 {env.now}')
    yield env.timeout(duration)
    print(f'{name} 结束于 {env.now}')
    return f'{name} 结果'

def coordinator(env):
    # 启动工作进程
    worker1 = env.process(worker(env, '工人1', 5))
    worker2 = env.process(worker(env, '工人2', 3))

    # 等待worker1完成
    result = yield worker1
    print(f'协调器收到: {result}')

    # 等待worker2
    result = yield worker2
    print(f'协调器收到: {result}')

env = simpy.Environment()
env.process(coordinator(env))
env.run()
```

### 通用事件

可手动触发的通用事件。

```python
import simpy

def waiter(env, event):
    print(f'等待事件于 {env.now}')
    value = yield event
    print(f'收到事件值: {value} 于 {env.now}')

def triggerer(env, event):
    yield env.timeout(5)
    print(f'触发事件于 {env.now}')
    event.succeed(value='你好!')

env = simpy.Environment()
event = env.event()
env.process(waiter(env, event))
env.process(triggerer(env, event))
env.run()
```

## 复合事件

### AllOf - 等待多个事件

当所有指定事件发生时触发。

```python
import simpy

def process(env):
    # 启动多个任务
    task1 = env.timeout(3, value='任务1完成')
    task2 = env.timeout(5, value='任务2完成')
    task3 = env.timeout(4, value='任务3完成')

    # 等待全部完成
    results = yield simpy.AllOf(env, [task1, task2, task3])
    print(f'所有任务完成于 {env.now}')
    print(f'结果: {results}')

    # 使用&运算符的替代语法
    task4 = env.timeout(2)
    task5 = env.timeout(3)
    yield task4 & task5
    print(f'任务4和5完成于 {env.now}')

env = simpy.Environment()
env.process(process(env))
env.run()
```

**返回：** 事件到其值的映射字典

**使用场景：**
- 并行任务完成
- 屏障同步
- 等待多个资源

### AnyOf - 等待任一事件

当任意指定事件发生时触发。

```python
import simpy

def process(env):
    # 启动不同时长的任务
    fast_task = env.timeout(2, value='快速')
    slow_task = env.timeout(10, value='慢速')

    # 等待首个完成
    results = yield simpy.AnyOf(env, [fast_task, slow_task])
    print(f'首个任务完成于 {env.now}')
    print(f'结果: {results}')

    # 使用|运算符的替代语法
    task1 = env.timeout(5)
    task2 = env.timeout(3)
    yield task1 | task2
    print(f'任一任务完成于 {env.now}')

env = simpy.Environment()
env.process(process(env))
env.run()
```

**返回：** 包含已完成事件及其值的字典

**使用场景：**
- 竞态条件
- 超时机制
- 首个响应场景

## 事件触发方法

事件可通过三种方式触发：

### succeed(value=None)

标记事件为成功。

```python
event = env.event()
event.succeed(value='成功!')
```

### fail(exception)

标记事件为失败并携带异常。

```python
def process(env):
    event = env.event()
    event.fail(ValueError('出错了'))

    try:
        yield event
    except ValueError as e:
        print(f'捕获异常: {e}')

env = simpy.Environment()
env.process(process(env))
env.run()
```

### trigger(event)

复制另一个事件的结果。

```python
event1 = env.event()
event1.succeed(value='原始值')

event2 = env.event()
event2.trigger(event1)  # event2现在与event1结果相同
```

## 回调函数

附加在事件触发时执行的函数。

```python
import simpy

def callback(event):
    print(f'回调执行! 事件值: {event.value}')

def process(env):
    event = env.timeout(5, value='完成')
    event.callbacks.append(callback)
    yield event

env = simpy.Environment()
env.process(process(env))
env.run()
```

**注意：** 进程生成事件时会自动将进程的恢复方法添加为回调。

## 事件共享

多个进程可等待同一事件。

```python
import simpy

def waiter(env, name, event):
    print(f'{name} 等待于 {env.now}')
    value = yield event
    print(f'{name} 恢复执行 值:{value} 于 {env.now}')

def trigger_event(env, event):
    yield env.timeout(5)
    event.succeed(value='开始!')

env = simpy.Environment()
shared_event = env.event()

env.process(waiter(env, '进程1', shared_event))
env.process(waiter(env, '进程2', shared_event))
env.process(waiter(env, '进程3', shared_event))
env.process(trigger_event(env, shared_event))

env.run()
```

**使用场景：**
- 广播信号
- 屏障同步
- 协调进程恢复

## 高级事件模式

### 带取消的超时

```python
import simpy

def process_with_timeout(env):
    work = env.timeout(10, value='工作完成')
    timeout = env.timeout(5, value='超时!')

    # 工作与超时竞速
    result = yield work | timeout

    if work in result:
        print(f'工作完成: {result[work]}')
    else:
        print(f'已超时: {result[timeout]}')

env = simpy.Environment()
env.process(process_with_timeout(env))
env.run()
```

### 事件链

```python
import simpy

def event_chain(env):
    # 创建依赖事件链
    event1 = env.event()
    event2 = env.event()
    event3 = env.event()

    def trigger_sequence(env):
        yield env.timeout(2)
        event1.succeed(value='步骤1')
        yield env.timeout(2)
        event2.succeed(value='步骤2')
        yield env.timeout(2)
        event3.succeed(value='步骤3')

    env.process(trigger_sequence(env))

    # 等待序列
    val1 = yield event1
    print(f'{val1} 于 {env.now}')
    val2 = yield event2
    print(f'{val2} 于 {env.now}')
    val3 = yield event3
    print(f'{val3} 于 {env.now}')

env = simpy.Environment()
env.process(event_chain(env))
env.run()
```

### 条件事件

```python
import simpy

def conditional_process(env):
    temperature = 20

    if temperature > 25:
        yield env.timeout(5)  # 需要冷却
        print('系统已冷却')
    else:
        yield env.timeout(1)  # 无需冷却
        print('温度正常')

env = simpy.Environment()
env.process(conditional_process(env))
env.run()
```

## 最佳实践

1. **始终生成事件**：进程必须生成事件才能暂停执行
2. **勿重复触发事件**：事件只能触发一次
3. **处理失败**：对可能失败的事件使用try-except
4. **复合事件实现并行**：使用AllOf/AnyOf处理并发操作
5. **共享事件实现广播**：多个进程可等待同一事件
6. **事件值传递数据**：通过事件值在进程间传递结果
