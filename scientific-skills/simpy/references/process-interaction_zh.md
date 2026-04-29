# SimPy 进程交互

本指南介绍 SimPy 模拟中进程交互与同步的机制。

## 交互机制概述

SimPy 提供三种主要的进程交互方式：

1. **基于事件的挂起/恢复** - 通过共享事件进行信号传递
2. **等待进程终止** - 产出（yield）进程对象
3. **中断** - 强制恢复暂停的进程

## 1. 基于事件的挂起与恢复

进程可通过共享事件协调执行流程。

### 基础信号模式

```python
import simpy

def controller(env, signal_event):
    print(f'控制器：准备于 {env.now}')
    yield env.timeout(5)
    print(f'控制器：发送信号于 {env.now}')
    signal_event.succeed()

def worker(env, signal_event):
    print(f'工作者：等待信号于 {env.now}')
    yield signal_event
    print(f'工作者：收到信号，开始工作于 {env.now}')
    yield env.timeout(3)
    print(f'工作者：工作完成于 {env.now}')

env = simpy.Environment()
signal = env.event()
env.process(controller(env, signal))
env.process(worker(env, signal))
env.run()
```

**适用场景：**
- 协调操作的启动信号
- 完成通知
- 状态变更广播

### 多接收者模式

多个进程可等待同一信号事件。

```python
import simpy

def broadcaster(env, signal):
    yield env.timeout(5)
    print(f'广播信号于 {env.now}')
    signal.succeed(value='开始！')

def listener(env, name, signal):
    print(f'{name}：等待于 {env.now}')
    msg = yield signal
    print(f'{name}：收到"{msg}"于 {env.now}')
    yield env.timeout(2)
    print(f'{name}：完成于 {env.now}')

env = simpy.Environment()
broadcast_signal = env.event()

env.process(broadcaster(env, broadcast_signal))
for i in range(3):
    env.process(listener(env, f'监听器 {i+1}', broadcast_signal))

env.run()
```

### 屏障同步

```python
import simpy

class Barrier:
    def __init__(self, env, n):
        self.env = env
        self.n = n
        self.count = 0
        self.event = env.event()

    def wait(self):
        self.count += 1
        if self.count >= self.n:
            self.event.succeed()
        return self.event

def worker(env, barrier, name, work_time):
    print(f'{name}：工作中于 {env.now}')
    yield env.timeout(work_time)
    print(f'{name}：到达屏障于 {env.now}')
    yield barrier.wait()
    print(f'{name}：通过屏障于 {env.now}')

env = simpy.Environment()
barrier = Barrier(env, 3)

env.process(worker(env, barrier, '工作者 A', 3))
env.process(worker(env, barrier, '工作者 B', 5))
env.process(worker(env, barrier, '工作者 C', 7))

env.run()
```

## 2. 等待进程终止

进程本身即为事件，可通过产出进程对象等待其完成。

### 顺序进程执行

```python
import simpy

def task(env, name, duration):
    print(f'{name}：启动于 {env.now}')
    yield env.timeout(duration)
    print(f'{name}：完成于 {env.now}')
    return f'{name} 结果'

def sequential_coordinator(env):
    # 顺序执行任务
    result1 = yield env.process(task(env, '任务 1', 5))
    print(f'协调器：{result1}')

    result2 = yield env.process(task(env, '任务 2', 3))
    print(f'协调器：{result2}')

    result3 = yield env.process(task(env, '任务 3', 4))
    print(f'协调器：{result3}')

env = simpy.Environment()
env.process(sequential_coordinator(env))
env.run()
```

### 并行进程执行

```python
import simpy

def task(env, name, duration):
    print(f'{name}：启动于 {env.now}')
    yield env.timeout(duration)
    print(f'{name}：完成于 {env.now}')
    return f'{name} 结果'

def parallel_coordinator(env):
    # 启动所有任务
    task1 = env.process(task(env, '任务 1', 5))
    task2 = env.process(task(env, '任务 2', 3))
    task3 = env.process(task(env, '任务 3', 4))

    # 等待全部完成
    results = yield task1 & task2 & task3
    print(f'所有任务完成于 {env.now}')
    print(f'任务1结果：{task1.value}')
    print(f'任务2结果：{task2.value}')
    print(f'任务3结果：{task3.value}')

env = simpy.Environment()
env.process(parallel_coordinator(env))
env.run()
```

### 首完成响应模式

```python
import simpy

def server(env, name, processing_time):
    print(f'{name}：开始处理请求于 {env.now}')
    yield env.timeout(processing_time)
    print(f'{name}：完成于 {env.now}')
    return name

def load_balancer(env):
    # 向多个服务器发送请求
    server1 = env.process(server(env, '服务器 1', 5))
    server2 = env.process(server(env, '服务器 2', 3))
    server3 = env.process(server(env, '服务器 3', 7))

    # 等待首个响应
    result = yield server1 | server2 | server3

    # 获取优先响应者
    winner = list(result.values())[0]
    print(f'负载均衡器：{winner} 率先响应于 {env.now}')

env = simpy.Environment()
env.process(load_balancer(env))
env.run()
```

## 3. 进程中断

可通过 `process.interrupt()` 中断进程，该方法会抛出 `Interrupt` 异常。

### 基础中断

```python
import simpy

def worker(env):
    try:
        print(f'工作者：开始长任务于 {env.now}')
        yield env.timeout(10)
        print(f'工作者：任务完成于 {env.now}')
    except simpy.Interrupt as interrupt:
        print(f'工作者：中断于 {env.now}')
        print(f'中断原因：{interrupt.cause}')

def interrupter(env, target_process):
    yield env.timeout(5)
    print(f'中断器：中断工作者于 {env.now}')
    target_process.interrupt(cause='高优先级任务')

env = simpy.Environment()
worker_process = env.process(worker(env))
env.process(interrupter(env, worker_process))
env.run()
```

### 可恢复中断

进程中断后可重新产出相同事件以继续等待。

```python
import simpy

def resumable_worker(env):
    work_left = 10

    while work_left > 0:
        try:
            print(f'工作者：工作中（剩余 {work_left} 单位）于 {env.now}')
            start = env.now
            yield env.timeout(work_left)
            work_left = 0
            print(f'工作者：完成于 {env.now}')
        except simpy.Interrupt:
            work_left -= (env.now - start)
            print(f'工作者：已中断！剩余 {work_left} 单位于 {env.now}')

def interrupter(env, worker_proc):
    yield env.timeout(3)
    worker_proc.interrupt()
    yield env.timeout(2)
    worker_proc.interrupt()

env = simpy.Environment()
worker_proc = env.process(resumable_worker(env))
env.process(interrupter(env, worker_proc))
env.run()
```

### 带自定义原因的中断

```python
import simpy

def machine(env, name):
    while True:
        try:
            print(f'{name}：运行中于 {env.now}')
            yield env.timeout(5)
        except simpy.Interrupt as interrupt:
            if interrupt.cause == 'maintenance':
                print(f'{name}：需维护于 {env.now}')
                yield env.timeout(2)
                print(f'{name}：维护完成于 {env.now}')
            elif interrupt.cause == 'emergency':
                print(f'{name}：紧急停止于 {env.now}')
                break

def maintenance_scheduler(env, machine_proc):
    yield env.timeout(7)
    machine_proc.interrupt(cause='maintenance')
    yield env.timeout(10)
    machine_proc.interrupt(cause='emergency')

env = simpy.Environment()
machine_proc = env.process(machine(env, '机器 1'))
env.process(maintenance_scheduler(env, machine_proc))
env.run()
```

### 带中断的抢占式资源

```python
import simpy

def user(env, name, resource, priority, duration):
    with resource.request(priority=priority) as req:
        try:
            yield req
            print(f'{name} (优先级 {priority})：获取资源于 {env.now}')
            yield env.timeout(duration)
            print(f'{name}：完成于 {env.now}')
        except simpy.Interrupt:
            print(f'{name}：被抢占于 {env.now}')

env = simpy.Environment()
resource = simpy.PreemptiveResource(env, capacity=1)

env.process(user(env, '低优先级用户', resource, priority=10, duration=10))
env.process(user(env, '高优先级用户', resource, priority=1, duration=5))
env.run()
```

## 高级模式

### 带信号的生产者-消费者模型

```python
import simpy

class Buffer:
    def __init__(self, env, capacity):
        self.env = env
        self.capacity = capacity
        self.items = []
        self.item_available = env.event()

    def put(self, item):
        if len(self.items) < self.capacity:
            self.items.append(item)
            if not self.item_available.triggered:
                self.item_available.succeed()
            return True
        return False

    def get(self):
        if self.items:
            return self.items.pop(0)
        return None

def producer(env, buffer):
    item_id = 0
    while True:
        yield env.timeout(2)
        item = f'物品 {item_id}'
        if buffer.put(item):
            print(f'生产者：添加 {item} 于 {env.now}')
            item_id += 1

def consumer(env, buffer):
    while True:
        if buffer.items:
            item = buffer.get()
            print(f'消费者：获取 {item} 于 {env.now}')
            yield env.timeout(3)
        else:
            print(f'消费者：等待物品于 {env.now}')
            yield buffer.item_available
            buffer.item_available = env.event()

env = simpy.Environment()
buffer = Buffer(env, capacity=5)
env.process(producer(env, buffer))
env.process(consumer(env, buffer))
env.run(until=20)
```

### 握手协议

```python
import simpy

def sender(env, request_event, acknowledge_event):
    for i in range(3):
        print(f'发送方：发送请求 {i} 于 {env.now}')
        request_event.succeed(value=f'请求 {i}')
        yield acknowledge_event
        print(f'发送方：收到确认于 {env.now}')

        # 重置事件用于下一轮
        request_event = env.event()
        acknowledge_event = env.event()
        yield env.timeout(1)

def receiver(env, request_event, acknowledge_event):
    for i in range(3):
        request = yield request_event
        print(f'接收方：收到 {request} 于 {env.now}')
        yield env.timeout(2)  # 处理请求
        acknowledge_event.succeed()
        print(f'接收方：发送确认于 {env.now}')

        # 重置用于下一轮
        request_event = env.event()
        acknowledge_event = env.event()

env = simpy.Environment()
request = env.event()
ack = env.event()
env.process(sender(env, request, ack))
env.process(receiver(env, request, ack))
env.run()
```

## 最佳实践

1. **选择合适机制**：
   - 信号传递使用事件
   - 顺序/并行工作流使用进程产出
   - 抢占和紧急处理使用中断

2. **异常处理**：易中断代码始终使用 try-except 块包裹

3. **事件生命周期**：事件仅能触发一次，重复信号需创建新事件

4. **进程引用**：需后续中断的进程应存储其对象引用

5. **原因信息**：通过中断原因传递中断信息

6. **可恢复模式**：记录进度以实现中断后恢复

7. **避免死锁**：确保任意时刻至少有一个进程可推进
