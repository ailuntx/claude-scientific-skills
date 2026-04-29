# SimPy 共享资源

本指南涵盖 SimPy 中用于建模拥塞点和资源分配的所有资源类型。

## 资源类型概览

SimPy 提供三大类共享资源：

1. **资源** - 容量有限的资源（如加油站泵、服务器）
2. **容器** - 同质散装物料（如油箱、筒仓）
3. **存储** - Python 对象存储（如物品队列、仓库）

## 1. 资源

建模可被有限数量进程同时使用的资源。

### 基础资源（Resource）

基础资源是具有指定容量的信号量。

```python
import simpy

env = simpy.Environment()
resource = simpy.Resource(env, capacity=2)

def process(env, resource, name):
    with resource.request() as req:
        yield req
        print(f'{name} 在 {env.now} 时刻占用资源')
        yield env.timeout(5)
        print(f'{name} 在 {env.now} 时刻释放资源')

env.process(process(env, resource, '进程 1'))
env.process(process(env, resource, '进程 2'))
env.process(process(env, resource, '进程 3'))
env.run()
```

**关键属性：**
- `capacity` - 最大并发用户数（默认：1）
- `count` - 当前用户数
- `queue` - 排队请求列表

### 优先级资源（PriorityResource）

在基础资源上增加优先级（数值越小优先级越高）。

```python
import simpy

env = simpy.Environment()
resource = simpy.PriorityResource(env, capacity=1)

def process(env, resource, name, priority):
    with resource.request(priority=priority) as req:
        yield req
        print(f'{name} (优先级 {priority}) 在 {env.now} 时刻占用资源')
        yield env.timeout(5)

env.process(process(env, resource, '低优先级', priority=10))
env.process(process(env, resource, '高优先级', priority=1))
env.run()
```

**适用场景：**
- 紧急服务（救护车优先于普通车辆）
- VIP客户队列
- 带优先级的任务调度

### 抢占式资源（PreemptiveResource）

允许高优先级请求中断低优先级用户。

```python
import simpy

env = simpy.Environment()
resource = simpy.PreemptiveResource(env, capacity=1)

def process(env, resource, name, priority):
    with resource.request(priority=priority) as req:
        try:
            yield req
            print(f'{name} 在 {env.now} 时刻获取资源')
            yield env.timeout(10)
            print(f'{name} 在 {env.now} 时刻完成')
        except simpy.Interrupt:
            print(f'{name} 在 {env.now} 时刻被抢占')

env.process(process(env, resource, '低优先级', priority=10))
env.process(process(env, resource, '高优先级', priority=1))
env.run()
```

**适用场景：**
- 操作系统CPU调度
- 急诊室分诊
- 网络数据包优先级排序

## 2. 容器

建模同质散装物料的生产与消耗（连续或离散）。

```python
import simpy

env = simpy.Environment()
container = simpy.Container(env, capacity=100, init=50)

def producer(env, container):
    while True:
        yield env.timeout(5)
        yield container.put(20)
        print(f'生产 20。当前存量: {container.level}')

def consumer(env, container):
    while True:
        yield env.timeout(7)
        yield container.get(15)
        print(f'消耗 15。当前存量: {container.level}')

env.process(producer(env, container))
env.process(consumer(env, container))
env.run(until=50)
```

**关键属性：**
- `capacity` - 最大容量（默认：float('inf')）
- `level` - 当前存量
- `init` - 初始存量（默认：0）

**操作：**
- `put(amount)` - 添加物料（满时阻塞）
- `get(amount)` - 取出物料（不足时阻塞）

**适用场景：**
- 加油站储油罐
- 制造缓冲区
- 水库水位
- 电池电量

## 3. 存储

建模 Python 对象的生产与消耗。

### 基础存储（Store）

通用先进先出（FIFO）对象存储。

```python
import simpy

env = simpy.Environment()
store = simpy.Store(env, capacity=2)

def producer(env, store):
    for i in range(5):
        yield env.timeout(2)
        item = f'物品 {i}'
        yield store.put(item)
        print(f'在 {env.now} 时刻生产 {item}')

def consumer(env, store):
    while True:
        yield env.timeout(3)
        item = yield store.get()
        print(f'在 {env.now} 时刻消耗 {item}')

env.process(producer(env, store))
env.process(consumer(env, store))
env.run()
```

**关键属性：**
- `capacity` - 最大物品数（默认：float('inf')）
- `items` - 存储物品列表

**操作：**
- `put(item)` - 添加物品（满时阻塞）
- `get()` - 取出物品（空时阻塞）

### 过滤存储（FilterStore）

支持通过过滤函数检索特定对象。

```python
import simpy

env = simpy.Environment()
store = simpy.FilterStore(env, capacity=10)

def producer(env, store):
    for color in ['红', '蓝', '绿', '红', '蓝']:
        yield env.timeout(1)
        yield store.put({'color': color, 'time': env.now})
        print(f'在 {env.now} 时刻生产 {color} 物品')

def consumer(env, store, color):
    while True:
        yield env.timeout(2)
        item = yield store.get(lambda x: x['color'] == color)
        print(f'{color} 消费者在 {env.now} 时刻获取 {item["time"]} 生产的物品')

env.process(producer(env, store))
env.process(consumer(env, store, '红'))
env.process(consumer(env, store, '蓝'))
env.run(until=15)
```

**适用场景：**
- 仓库拣货（特定SKU）
- 技能匹配的任务队列
- 按目的地的数据包路由

### 优先级存储（PriorityStore）

按优先级顺序检索物品（数值越小优先级越高）。

```python
import simpy

class PriorityItem:
    def __init__(self, priority, data):
        self.priority = priority
        self.data = data

    def __lt__(self, other):
        return self.priority < other.priority

env = simpy.Environment()
store = simpy.PriorityStore(env, capacity=10)

def producer(env, store):
    items = [(10, '低'), (1, '高'), (5, '中')]
    for priority, name in items:
        yield env.timeout(1)
        yield store.put(PriorityItem(priority, name))
        print(f'生产 {name} 优先级物品')

def consumer(env, store):
    while True:
        yield env.timeout(5)
        item = yield store.get()
        print(f'检索到 {item.data} 优先级物品')

env.process(producer(env, store))
env.process(consumer(env, store))
env.run()
```

**适用场景：**
- 任务调度
- 打印作业队列
- 消息优先级排序

## 选择正确的资源类型

| 场景 | 资源类型 |
|------|----------|
| 有限服务器/机器 | Resource |
| 基于优先级的排队 | PriorityResource |
| 抢占式调度 | PreemptiveResource |
| 燃料、水等散装物料 | Container |
| 通用物品队列（FIFO） | Store |
| 选择性物品检索 | FilterStore |
| 优先级排序物品 | PriorityStore |

## 最佳实践

1. **容量规划**：根据系统约束设置合理容量
2. **请求模式**：使用上下文管理器（`with resource.request()`）自动清理
3. **错误处理**：用 try-except 包裹抢占式资源以处理中断
4. **监控**：跟踪队列长度和利用率（参见 monitoring.md）
5. **性能**：FilterStore 和 PriorityStore 的检索时间为 O(n)；大型存储需谨慎使用
