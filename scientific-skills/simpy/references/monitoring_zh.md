# SimPy 监控与数据收集指南

本指南涵盖在 SimPy 中收集数据和监控仿真行为的技术。

## 监控策略

实施监控前需明确三点：

1. **监控对象**：进程、资源、事件或系统状态
2. **监控时机**：变更时、固定间隔或特定事件发生时
3. **数据存储方式**：列表、文件、数据库或实时输出

## 1. 进程监控

### 状态变量追踪

通过记录变量变更实现进程状态追踪。

```python
import simpy

def customer(env, name, service_time, log):
    arrival_time = env.now
    log.append(('arrival', name, arrival_time))

    yield env.timeout(service_time)

    departure_time = env.now
    log.append(('departure', name, departure_time))

    wait_time = departure_time - arrival_time
    log.append(('wait_time', name, wait_time))

env = simpy.Environment()
log = []

env.process(customer(env, 'Customer 1', 5, log))
env.process(customer(env, 'Customer 2', 3, log))
env.run()

print('仿真日志:')
for entry in log:
    print(entry)
```

### 时间序列数据收集

```python
import simpy

def system_monitor(env, system_state, data_log, interval):
    while True:
        data_log.append((env.now, system_state['queue_length'], system_state['utilization']))
        yield env.timeout(interval)

def process(env, system_state):
    while True:
        system_state['queue_length'] += 1
        yield env.timeout(2)
        system_state['queue_length'] -= 1
        system_state['utilization'] = system_state['queue_length'] / 10
        yield env.timeout(3)

env = simpy.Environment()
system_state = {'queue_length': 0, 'utilization': 0.0}
data_log = []

env.process(system_monitor(env, system_state, data_log, interval=1))
env.process(process(env, system_state))
env.run(until=20)

print('时间序列数据:')
for time, queue, util in data_log:
    print(f'时间 {time}: 队列长度={queue}, 利用率={util:.2f}')
```

### 多变量追踪

```python
import simpy

class SimulationData:
    def __init__(self):
        self.timestamps = []
        self.queue_lengths = []
        self.processing_times = []
        self.utilizations = []

    def record(self, timestamp, queue_length, processing_time, utilization):
        self.timestamps.append(timestamp)
        self.queue_lengths.append(queue_length)
        self.processing_times.append(processing_time)
        self.utilizations.append(utilization)

def monitored_process(env, data):
    queue_length = 0
    processing_time = 0
    utilization = 0.0

    for i in range(5):
        queue_length = i % 3
        processing_time = 2 + i
        utilization = queue_length / 10

        data.record(env.now, queue_length, processing_time, utilization)
        yield env.timeout(2)

env = simpy.Environment()
data = SimulationData()
env.process(monitored_process(env, data))
env.run()

print(f'收集到 {len(data.timestamps)} 个数据点')
```

## 2. 资源监控

### 资源猴子补丁

通过拦截资源方法记录操作日志。

```python
import simpy

def patch_resource(resource, data_log):
    """为资源添加请求和释放操作的日志记录"""

    # 保存原始方法
    original_request = resource.request
    original_release = resource.release

    # 创建请求包装器
    def logged_request(*args, **kwargs):
        req = original_request(*args, **kwargs)
        data_log.append(('request', resource._env.now, len(resource.queue)))
        return req

    # 创建释放包装器
    def logged_release(*args, **kwargs):
        result = original_release(*args, **kwargs)
        data_log.append(('release', resource._env.now, len(resource.queue)))
        return result

    # 替换方法
    resource.request = logged_request
    resource.release = logged_release

def user(env, name, resource):
    with resource.request() as req:
        yield req
        print(f'{name} 在 {env.now} 使用资源')
        yield env.timeout(3)
        print(f'{name} 在 {env.now} 释放资源')

env = simpy.Environment()
resource = simpy.Resource(env, capacity=1)
log = []

patch_resource(resource, log)

env.process(user(env, '用户1', resource))
env.process(user(env, '用户2', resource))
env.run()

print('\n资源日志:')
for entry in log:
    print(entry)
```

### 资源子类化

创建内置监控的自定义资源类。

```python
import simpy

class MonitoredResource(simpy.Resource):
    def __init__(self, env, capacity):
        super().__init__(env, capacity)
        self.data = []
        self.utilization_data = []

    def request(self, *args, **kwargs):
        req = super().request(*args, **kwargs)
        queue_length = len(self.queue)
        utilization = self.count / self.capacity
        self.data.append(('request', self._env.now, queue_length, utilization))
        self.utilization_data.append((self._env.now, utilization))
        return req

    def release(self, *args, **kwargs):
        result = super().release(*args, **kwargs)
        queue_length = len(self.queue)
        utilization = self.count / self.capacity
        self.data.append(('release', self._env.now, queue_length, utilization))
        self.utilization_data.append((self._env.now, utilization))
        return result

    def average_utilization(self):
        if not self.utilization_data:
            return 0.0
        return sum(u for _, u in self.utilization_data) / len(self.utilization_data)

def user(env, name, resource):
    with resource.request() as req:
        yield req
        print(f'{name} 在 {env.now} 使用资源')
        yield env.timeout(2)

env = simpy.Environment()
resource = MonitoredResource(env, capacity=2)

for i in range(5):
    env.process(user(env, f'用户 {i+1}', resource))

env.run()

print(f'\n平均利用率: {resource.average_utilization():.2%}')
print(f'总操作数: {len(resource.data)}')
```

### 容器级别监控

```python
import simpy

class MonitoredContainer(simpy.Container):
    def __init__(self, env, capacity, init=0):
        super().__init__(env, capacity, init)
        self.level_data = [(0, init)]

    def put(self, amount):
        result = super().put(amount)
        self.level_data.append((self._env.now, self.level))
        return result

    def get(self, amount):
        result = super().get(amount)
        self.level_data.append((self._env.now, self.level))
        return result

def producer(env, container, amount, interval):
    while True:
        yield env.timeout(interval)
        yield container.put(amount)
        print(f'生产 {amount}。当前容量: {container.level}，时间: {env.now}')

def consumer(env, container, amount, interval):
    while True:
        yield env.timeout(interval)
        yield container.get(amount)
        print(f'消耗 {amount}。当前容量: {container.level}，时间: {env.now}')

env = simpy.Environment()
container = MonitoredContainer(env, capacity=100, init=50)

env.process(producer(env, container, 20, 3))
env.process(consumer(env, container, 15, 4))
env.run(until=20)

print('\n容量历史记录:')
for time, level in container.level_data:
    print(f'时间 {time}: 容量={level}')
```

## 3. 事件追踪

### 环境步骤监控

通过修补环境的 step 函数监控所有事件。

```python
import simpy

def trace(env, callback):
    """追踪环境处理的所有事件"""

    def _trace_step():
        # 在事件处理前获取下一个事件
        if env._queue:
            time, priority, event_id, event = env._queue[0]
            callback(time, priority, event_id, event)

        # 调用原始步骤函数
        return original_step()

    original_step = env.step
    env.step = _trace_step

def event_callback(time, priority, event_id, event):
    print(f'事件: 时间={time}, 优先级={priority}, ID={event_id}, 类型={type(event).__name__}')

def process(env, name):
    print(f'{name}: 开始于 {env.now}')
    yield env.timeout(5)
    print(f'{name}: 结束于 {env.now}')

env = simpy.Environment()
trace(env, event_callback)

env.process(process(env, '进程1'))
env.process(process(env, '进程2'))
env.run()
```

### 事件调度监控

追踪事件调度情况。

```python
import simpy

class MonitoredEnvironment(simpy.Environment):
    def __init__(self):
        super().__init__()
        self.scheduled_events = []

    def schedule(self, event, priority=simpy.core.NORMAL, delay=0):
        super().schedule(event, priority, delay)
        scheduled_time = self.now + delay
        self.scheduled_events.append((scheduled_time, priority, type(event).__name__))

def process(env, name, delay):
    print(f'{name}: 在 {env.now} 调度 {delay} 秒超时')
    yield env.timeout(delay)
    print(f'{name}: 在 {env.now} 恢复执行')

env = MonitoredEnvironment()
env.process(process(env, '进程1', 5))
env.process(process(env, '进程2', 3))
env.run()

print('\n已调度事件:')
for time, priority, event_type in env.scheduled_events:
    print(f'时间 {time}, 优先级 {priority}, 类型 {event_type}')
```

## 4. 统计监控

### 队列统计

```python
import simpy

class QueueStatistics:
    def __init__(self):
        self.arrival_times = []
        self.departure_times = []
        self.queue_lengths = []
        self.wait_times = []

    def record_arrival(self, time, queue_length):
        self.arrival_times.append(time)
        self.queue_lengths.append(queue_length)

    def record_departure(self, arrival_time, departure_time):
        self.departure_times.append(departure_time)
        self.wait_times.append(departure_time - arrival_time)

    def average_wait_time(self):
        return sum(self.wait_times) / len(self.wait_times) if self.wait_times else 0

    def average_queue_length(self):
        return sum(self.queue_lengths) / len(self.queue_lengths) if self.queue_lengths else 0

def customer(env, resource, stats):
    arrival_time = env.now
    stats.record_arrival(arrival_time, len(resource.queue))

    with resource.request() as req:
        yield req
        departure_time = env.now
        stats.record_departure(arrival_time, departure_time)
        yield env.timeout(2)

env = simpy.Environment()
resource = simpy.Resource(env, capacity=1)
stats = QueueStatistics()

for i in range(5):
    env.process(customer(env, resource, stats))

env.run()

print(f'平均等待时间: {stats.average_wait_time():.2f}')
print(f'平均队列长度: {stats.average_queue_length():.2f}')
```

## 5. 数据导出

### CSV 导出

```python
import simpy
import csv

def export_to_csv(data, filename):
    with open(filename, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['时间', '指标', '数值'])
        writer.writerows(data)

def monitored_simulation(env, data_log):
    for i in range(10):
        data_log.append((env.now, 'queue_length', i % 3))
        data_log.append((env.now, 'utilization', (i % 3) / 10))
        yield env.timeout(1)

env = simpy.Environment()
data = []
env.process(monitored_simulation(env, data))
env.run()

export_to_csv(data, 'simulation_data.csv')
print('数据已导出至 simulation_data.csv')
```

### 实时绘图（需 matplotlib）

```python
import simpy
import matplotlib.pyplot as plt

class RealTimePlotter:
    def __init__(self):
        self.times = []
        self.values = []

    def update(self, time, value):
        self.times.append(time)
        self.values.append(value)

    def plot(self, title='仿真结果'):
        plt.figure(figsize=(10, 6))
        plt.plot(self.times, self.values)
        plt.xlabel('时间')
        plt.ylabel('数值')
        plt.title(title)
        plt.grid(True)
        plt.show()

def monitored_process(env, plotter):
    value = 0
    for i in range(20):
        value = value * 0.9 + (i % 5)
        plotter.update(env.now, value)
        yield env.timeout(1)

env = simpy.Environment()
plotter = RealTimePlotter()
env.process(monitored_process(env, plotter))
env.run()

plotter.plot('进程数值随时间变化')
```

## 最佳实践

1. **最小化开销**：仅监控必要内容，过度日志记录会降低仿真速度
2. **结构化数据**：对复杂数据点使用类或命名元组
3. **时间戳标记**：始终为监控数据添加时间戳
4. **数据聚合**：长时仿真中应聚合数据而非存储每个事件
5. **延迟计算**：考虑收集原始数据并在仿真后计算统计量
6. **内存管理**：超长仿真中定期将数据转存至磁盘
7. **验证机制**：确保监控代码不影响仿真行为
8. **关注点分离**：保持监控代码与仿真逻辑独立
9. **可复用组件**：创建可在不同仿真中复用的通用监控类
