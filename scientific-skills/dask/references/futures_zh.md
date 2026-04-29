# Dask Futures

## 概述

Dask Futures 扩展了 Python 的 `concurrent.futures` 接口，支持即时（非惰性）任务执行。与延迟计算（用于 DataFrames、Arrays 和 Bags）不同，futures 在计算可能随时间演变或需要动态构建工作流的场景中提供了更大的灵活性。

## 核心概念

Futures 代表实时任务执行：
- 任务提交后立即执行（非惰性）
- 每个 future 代表一个远程计算结果
- 自动追踪 futures 间的依赖关系
- 支持动态演进的工作流
- 直接控制任务调度和数据放置

## 关键能力

### 实时执行
- 任务提交后立即运行
- 无需显式调用 `.compute()`
- 通过 `.result()` 方法获取结果

### 自动依赖管理
当使用 future 作为输入提交任务时，Dask 自动处理依赖追踪。所有输入 futures 完成后，任务将被分配到单个工作节点进行高效计算。

### 动态工作流
构建基于中间结果演变的计算：
- 根据先前结果提交新任务
- 条件执行路径
- 结构可变的迭代算法

## 使用场景

**适用 Futures 的情况**：
- 构建动态演进的工作流
- 需要即时任务执行（非惰性）
- 计算依赖运行时条件
- 需要精细控制任务放置
- 实现自定义并行算法
- 需要状态计算（使用 actors）

**适用其他集合的情况**：
- 静态预定义计算图（使用 delayed、DataFrames、Arrays）
- 大型集合的简单数据并行（使用 Bags、DataFrames）
- 标准数组/数据框操作已满足需求

## 客户端设置

Futures 需要分布式客户端：

```python
from dask.distributed import Client

# 本地集群（单机）
client = Client()

# 或指定资源
client = Client(n_workers=4, threads_per_worker=2)

# 或连接现有集群
client = Client('scheduler-address:8786')
```

## 任务提交

### 基础提交
```python
from dask.distributed import Client

client = Client()

# 提交单个任务
def add(x, y):
    return x + y

future = client.submit(add, 1, 2)

# 获取结果
result = future.result()  # 阻塞直至完成
print(result)  # 3
```

### 多任务处理
```python
# 提交多个独立任务
futures = []
for i in range(10):
    future = client.submit(add, i, i)
    futures.append(future)

# 收集结果
results = client.gather(futures)  # 高效并行收集
```

### 批量映射
```python
# 对多个输入应用函数
def square(x):
    return x ** 2

# 提交批量任务
futures = client.map(square, range(100))

# 收集结果
results = client.gather(futures)
```

**注意**：每个任务约 1ms 开销，因此 `map` 不适合数百万个微型任务。处理海量数据时请改用 Bags 或 DataFrames。

## Futures 操作

### 状态检查
```python
future = client.submit(expensive_function, arg)

# 检查是否完成
print(future.done())  # False 或 True

# 检查状态
print(future.status)  # 'pending'（等待中）、'running'（运行中）、'finished'（已完成）或 'error'（错误）
```

### 非阻塞结果获取
```python
# 非阻塞检查
if future.done():
    result = future.result()
else:
    print("仍在计算中...")

# 或使用回调
def handle_result(future):
    print(f"结果: {future.result()}")

future.add_done_callback(handle_result)
```

### 错误处理
```python
def might_fail(x):
    if x < 0:
        raise ValueError("负值无效")
    return x ** 2

future = client.submit(might_fail, -5)

try:
    result = future.result()
except ValueError as e:
    print(f"任务失败: {e}")
```

## 任务依赖

### 自动依赖追踪
```python
# 提交任务
future1 = client.submit(add, 1, 2)

# 使用 future 作为输入（创建依赖）
future2 = client.submit(add, future1, 10)  # 依赖 future1

# 链式依赖
future3 = client.submit(add, future2, 100)  # 依赖 future2

# 获取最终结果
result = future3.result()  # 113
```

### 复杂依赖
```python
# 多重依赖
a = client.submit(func1, x)
b = client.submit(func2, y)
c = client.submit(func3, a, b)  # 同时依赖 a 和 b

result = c.result()
```

## 数据移动优化

### 预分发数据
预分发重要数据避免重复传输：

```python
# 单次上传数据到集群
large_dataset = client.scatter(big_data)  # 返回 future

# 在多个任务中使用预分发数据
futures = [client.submit(process, large_dataset, i) for i in range(100)]

# 每个任务复用预分发数据无需重复传输
results = client.gather(futures)
```

### 高效收集
使用 `client.gather()` 进行并发结果收集：

```python
# 推荐：批量收集（并行）
results = client.gather(futures)

# 不推荐：顺序获取结果
results = [f.result() for f in futures]
```

## 即发即弃

适用于不需要结果的副作用任务：

```python
from dask.distributed import fire_and_forget

def log_to_database(data):
    # 写入数据库，无需返回值
    database.write(data)

# 提交但不保留引用
future = client.submit(log_to_database, data)
fire_and_forget(future)

# 即使没有活动 future 引用，Dask 也不会放弃此计算
```

## 性能特征

### 任务开销
- 每任务约 1ms 开销
- 适用场景：数千个任务
- 不适用场景：数百万个微型任务

### 工作节点间通信
- 直接的工作节点间数据传输
- 往返延迟：约 1ms
- 高效处理任务依赖

### 内存管理
Dask 在本地追踪活动 futures。当本地 Python 会话垃圾回收 future 时，Dask 将释放对应数据。

**保留引用**：
```python
# 保留引用防止数据删除
important_result = client.submit(expensive_calc, data)

# 多次复用结果
future1 = client.submit(process1, important_result)
future2 = client.submit(process2, important_result)
```

## 高级协调

### 分布式原语

**队列**：
```python
from dask.distributed import Queue

queue = Queue()

def producer():
    for i in range(10):
        queue.put(i)

def consumer():
    results = []
    for _ in range(10):
        results.append(queue.get())
    return results

# 提交任务
client.submit(producer)
result_future = client.submit(consumer)
results = result_future.result()
```

**锁**：
```python
from dask.distributed import Lock

lock = Lock()

def critical_section():
    with lock:
        # 同一时间仅一个任务执行此代码
        shared_resource.update()
```

**事件**：
```python
from dask.distributed import Event

event = Event()

def waiter():
    event.wait()  # 阻塞直至事件触发
    return "事件已触发"

def setter():
    time.sleep(5)
    event.set()

# 启动双任务
wait_future = client.submit(waiter)
set_future = client.submit(setter)

result = wait_future.result()  # 等待 setter 完成
```

**变量**：
```python
from dask.distributed import Variable

var = Variable('my-var')

# 设置值
var.set(42)

# 从任务中获取值
def reader():
    return var.get()

future = client.submit(reader)
print(future.result())  # 42
```

## Actors

对于有状态、快速变化的工作流，actors 能实现约 1ms 的工作节点间往返延迟，同时绕过调度器协调。

### 创建 Actors
```python
from dask.distributed import Client

client = Client()

class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        return self.count

    def get_count(self):
        return self.count

# 在工作节点创建 actor
counter = client.submit(Counter, actor=True).result()

# 调用方法
future1 = counter.increment()
future2 = counter.increment()
result = counter.get_count().result()
print(result)  # 2
```

### Actor 适用场景
- 有状态服务（数据库、缓存）
- 快速变化的状态
- 复杂协调模式
- 实时流处理应用

## 常用模式

### 易并行任务
```python
from dask.distributed import Client

client = Client()

def process_item(item):
    # 独立计算
    return expensive_computation(item)

# 并行处理多个项目
items = range(1000)
futures = client.map(process_item, items)

# 收集所有结果
results = client.gather(futures)
```

### 动态任务提交
```python
def recursive_compute(data, depth):
    if depth == 0:
        return process(data)

    # 分割并递归
    left, right = split(data)
    left_future = client.submit(recursive_compute, left, depth - 1)
    right_future = client.submit(recursive_compute, right, depth - 1)

    # 合并结果
    return combine(left_future.result(), right_future.result())

# 启动计算
result_future = client.submit(recursive_compute, initial_data, 5)
result = result_future.result()
```

### 参数扫描
```python
from itertools import product

def run_simulation(param1, param2, param3):
    # 使用参数运行模拟
    return simulate(param1, param2, param3)

# 生成参数组合
params = product(range(10), range(10), range(10))

# 提交所有组合
futures = [client.submit(run_simulation, p1, p2, p3) for p1, p2, p3 in params]

# 按完成顺序收集结果
from dask.distributed import as_completed

for future in as_completed(futures):
    result = future.result()
    process_result(result)
```

### 依赖管道
```python
# 阶段1：加载数据
load_futures = [client.submit(load_data, file) for file in files]

# 阶段2：处理（依赖阶段1）
process_futures = [client.submit(process, f) for f in load_futures]

# 阶段3：聚合（依赖阶段2）
agg_future = client.submit(aggregate, process_futures)

# 获取最终结果
result = agg_future.result()
```

### 迭代算法
```python
# 初始化
state = client.scatter(initial_state)

# 迭代
for iteration in range(num_iterations):
    # 基于当前状态计算更新
    state = client.submit(update_function, state)

    # 检查收敛性
    converged = client.submit(check_convergence, state)
    if converged.result():
        break

# 获取最终状态
final_state = state.result()
```

## 最佳实践

### 1. 预分发大型数据
```python
# 单次上传，多次复用
large_data = client.scatter(big_dataset)
futures = [client.submit(process, large_data, i) for i in range(100)]
```

### 2. 批量收集结果
```python
# 高效：并行收集
results = client.gather(futures)

# 低效：顺序收集
results = [f.result() for f in futures]
```

### 3. 通过引用管理内存
```python
# 保留重要 futures
important = client.submit(expensive_calc, data)

# 多次复用
f1 = client.submit(use_result, important)
f2 = client.submit(use_result, important)

# 完成后清理
del important
```

### 4. 合理处理错误
```python
futures = client.map(might_fail, inputs)

# 错误检查
results = []
errors = []
for future in as_completed(futures):
    try:
        results.append(future.result())
    except Exception as e:
        errors.append(e)
```

### 5. 使用 as_completed 渐进处理
```python
from dask.distributed import as_completed

futures = client.map(process, items)

# 按完成顺序处理结果
for future in as_completed(futures):
    result = future.result()
    handle_result(result)
```

## 调试技巧

### 监控仪表盘
通过 Dask 仪表盘查看：
- 任务进度
- 工作节点利用率
- 内存使用
- 任务依赖关系

### 检查任务状态
```python
# 检查 future
print(future.status)
print(future.done())

# 出错时获取回溯
try:
    future.result()
except Exception:
    print(future.traceback())
```

### 任务性能分析
```python
# 获取性能数据
client.profile(filename='profile.html')
```
