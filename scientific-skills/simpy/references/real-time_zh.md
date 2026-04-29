# SimPy 实时仿真

本指南介绍 SimPy 的实时仿真功能，其中仿真时间与挂钟时间同步。

## 概述

实时仿真将仿真时间与实际挂钟时间同步，适用于：

- **硬件在环（HIL）**测试
- 人与仿真的**交互操作**
- 实时约束下的**算法行为分析**
- **系统集成**测试
- **演示**场景

## RealtimeEnvironment

将标准 `Environment` 替换为 `simpy.rt.RealtimeEnvironment` 以启用实时同步。

### 基础用法

```python
import simpy.rt

def process(env):
    while True:
        print(f'Tick at {env.now}')
        yield env.timeout(1)

# 1:1 时间映射的实时环境
env = simpy.rt.RealtimeEnvironment(factor=1.0)
env.process(process(env))
env.run(until=5)
```

### 构造函数参数

```python
simpy.rt.RealtimeEnvironment(
    initial_time=0,      # 起始仿真时间
    factor=1.0,          # 单位仿真时间对应的实际时长
    strict=True          # 超时触发错误
)
```

## 时间缩放因子

`factor` 参数控制仿真时间与实际时间的映射关系。

### 因子示例

```python
import simpy.rt
import time

def timed_process(env, label):
    start = time.time()
    print(f'{label}: Starting at {env.now}')
    yield env.timeout(2)
    elapsed = time.time() - start
    print(f'{label}: Completed at {env.now} (real time: {elapsed:.2f}s)')

# Factor = 1.0: 1仿真时间单位 = 1秒
print('Factor = 1.0 (2 sim units = 2 seconds)')
env = simpy.rt.RealtimeEnvironment(factor=1.0)
env.process(timed_process(env, 'Normal speed'))
env.run()

# Factor = 0.5: 1仿真时间单位 = 0.5秒
print('\nFactor = 0.5 (2 sim units = 1 second)')
env = simpy.rt.RealtimeEnvironment(factor=0.5)
env.process(timed_process(env, 'Double speed'))
env.run()

# Factor = 2.0: 1仿真时间单位 = 2秒
print('\nFactor = 2.0 (2 sim units = 4 seconds)')
env = simpy.rt.RealtimeEnvironment(factor=2.0)
env.process(timed_process(env, 'Half speed'))
env.run()
```

**因子解读：**
- `factor=1.0` → 1仿真时间单位耗时1秒
- `factor=0.1` → 1仿真时间单位耗时0.1秒（快10倍）
- `factor=60` → 1仿真时间单位耗时60秒（1分钟）

## 严格模式

### strict=True（默认）

当计算超出分配的实时预算时抛出 `RuntimeError`。

```python
import simpy.rt
import time

def heavy_computation(env):
    print(f'Starting computation at {env.now}')
    yield env.timeout(1)

    # 模拟高负载计算（超出1秒预算）
    time.sleep(1.5)

    print(f'Computation done at {env.now}')

env = simpy.rt.RealtimeEnvironment(factor=1.0, strict=True)
env.process(heavy_computation(env))

try:
    env.run()
except RuntimeError as e:
    print(f'Error: {e}')
```

### strict=False

允许仿真以低于预期的速度运行而不崩溃。

```python
import simpy.rt
import time

def heavy_computation(env):
    print(f'Starting at {env.now}')
    yield env.timeout(1)

    # 高负载计算
    time.sleep(1.5)

    print(f'Done at {env.now}')

env = simpy.rt.RealtimeEnvironment(factor=1.0, strict=False)
env.process(heavy_computation(env))
env.run()

print('Simulation completed (slower than real-time)')
```

**适用 strict=False 的场景：**
- 开发调试阶段
- 计算时间不可预测时
- 可接受低于目标速率的运行
- 分析最坏情况行为

## 硬件在环示例

```python
import simpy.rt

class HardwareInterface:
    """模拟硬件接口"""

    def __init__(self):
        self.sensor_value = 0

    def read_sensor(self):
        """模拟读取硬件传感器"""
        import random
        self.sensor_value = random.uniform(20.0, 30.0)
        return self.sensor_value

    def write_actuator(self, value):
        """模拟写入硬件执行器"""
        print(f'Actuator set to {value:.2f}')

def control_loop(env, hardware, setpoint):
    """实时运行的简单控制回路"""
    while True:
        # 读取传感器
        sensor_value = hardware.read_sensor()
        print(f'[{env.now}] Sensor: {sensor_value:.2f}°C')

        # 简单比例控制
        error = setpoint - sensor_value
        control_output = error * 0.1

        # 写入执行器
        hardware.write_actuator(control_output)

        # 控制回路每0.5秒运行
        yield env.timeout(0.5)

# 实时环境：1仿真单位=1秒
env = simpy.rt.RealtimeEnvironment(factor=1.0, strict=False)
hardware = HardwareInterface()
setpoint = 25.0

env.process(control_loop(env, hardware, setpoint))
env.run(until=5)
```

## 人机交互示例

```python
import simpy.rt

def interactive_process(env):
    """等待模拟用户输入的进程"""
    print('仿真启动。事件将实时发生')

    yield env.timeout(2)
    print(f'[{env.now}] 事件1：系统启动')

    yield env.timeout(3)
    print(f'[{env.now}] 事件2：初始化完成')

    yield env.timeout(2)
    print(f'[{env.now}] 事件3：准备就绪')

# 人机节奏演示的实时环境
env = simpy.rt.RealtimeEnvironment(factor=1.0)
env.process(interactive_process(env))
env.run()
```

## 实时性能监控

```python
import simpy.rt
import time

class RealTimeMonitor:
    def __init__(self):
        self.step_times = []
        self.drift_values = []

    def record_step(self, sim_time, real_time, expected_real_time):
        self.step_times.append(sim_time)
        drift = real_time - expected_real_time
        self.drift_values.append(drift)

    def report(self):
        if self.drift_values:
            avg_drift = sum(self.drift_values) / len(self.drift_values)
            max_drift = max(abs(d) for d in self.drift_values)
            print(f'\n实时性能报告：')
            print(f'平均偏移：{avg_drift*1000:.2f} 毫秒')
            print(f'最大偏移：{max_drift*1000:.2f} 毫秒')

def monitored_process(env, monitor, start_time, factor):
    for i in range(5):
        step_start = time.time()
        yield env.timeout(1)

        real_elapsed = time.time() - start_time
        expected_elapsed = env.now * factor
        monitor.record_step(env.now, real_elapsed, expected_elapsed)

        print(f'仿真时间：{env.now}, 实际耗时：{real_elapsed:.2f}秒, ' +
              f'预期耗时：{expected_elapsed:.2f}秒')

start = time.time()
factor = 1.0
env = simpy.rt.RealtimeEnvironment(factor=factor, strict=False)
monitor = RealTimeMonitor()

env.process(monitored_process(env, monitor, start, factor))
env.run()
monitor.report()
```

## 混合实时与快速仿真

```python
import simpy.rt

def background_simulation(env):
    """快速后台仿真"""
    for i in range(100):
        yield env.timeout(0.01)
    print(f'后台仿真完成于 {env.now}')

def real_time_display(env):
    """实时显示更新"""
    for i in range(5):
        print(f'显示更新于 {env.now}')
        yield env.timeout(1)

# 注意：此为概念演示 - SimPy不直接支持混合模式
# 可考虑独立运行仿真或使用strict=False
env = simpy.rt.RealtimeEnvironment(factor=1.0, strict=False)
env.process(background_simulation(env))
env.process(real_time_display(env))
env.run()
```

## 标准仿真转实时

标准仿真转实时操作简便：

```python
import simpy
import simpy.rt

def process(env):
    print(f'事件发生于 {env.now}')
    yield env.timeout(1)
    print(f'事件发生于 {env.now}')
    yield env.timeout(1)
    print(f'事件发生于 {env.now}')

# 标准仿真（瞬时完成）
print('标准仿真：')
env = simpy.Environment()
env.process(process(env))
env.run()

# 实时仿真（耗时2秒）
print('\n实时仿真：')
env_rt = simpy.rt.RealtimeEnvironment(factor=1.0)
env_rt.process(process(env_rt))
env_rt.run()
```

## 最佳实践

1. **因子选择**：根据硬件/人员约束选择因子
   - 人机交互：`factor=1.0`（1:1时间映射）
   - 高速硬件：`factor=0.01`（快100倍）
   - 慢速流程：`factor=60`（1仿真单位=1分钟）

2. **严格模式使用**：
   - 时序验证用 `strict=True`
   - 开发及可变负载用 `strict=False`

3. **计算预算**：确保进程逻辑执行时间小于超时周期

4. **错误处理**：用 try-except 捕获实时运行超时异常

5. **测试策略**：
   - 用标准Environment开发（快速迭代）
   - 用RealtimeEnvironment测试（验证）
   - 按需配置因子和严格模式部署

6. **性能监控**：跟踪仿真与实际的时序偏移

7. **优雅降级**：时序保障非关键时使用 `strict=False`

## 常用模式

### 周期性实时任务

```python
import simpy.rt

def periodic_task(env, name, period, duration):
    """实时周期性任务"""
    while True:
        start = env.now
        print(f'{name}：启动于 {start}')

        # 模拟工作
        yield env.timeout(duration)

        print(f'{name}：完成于 {env.now}')

        # 等待下一周期
        elapsed = env.now - start
        wait_time = period - elapsed
        if wait_time > 0:
            yield env.timeout(wait_time)

env = simpy.rt.RealtimeEnvironment(factor=1.0)
env.process(periodic_task(env, '任务', period=2.0, duration=0.5))
env.run(until=6)
```

### 多设备同步控制

```python
import simpy.rt

def device_controller(env, device_id, update_rate):
    """单设备控制回路"""
    while True:
        print(f'设备 {device_id}：更新于 {env.now}')
        yield env.timeout(update_rate)

# 所有设备实时同步
env = simpy.rt.RealtimeEnvironment(factor=1.0)

# 不同设备采用不同更新频率
env.process(device_controller(env, 'A', 1.0))
env.process(device_controller(env, 'B', 0.5))
env.process(device_controller(env, 'C', 2.0))

env.run(until=5)
```

## 局限性

1. **性能**：实时仿真增加开销，不适用于高频事件
2. **同步**：单线程运行，所有进程共享同一时间基准
3. **精度**：受Python时间精度和系统调度限制
4. **严格模式**：计算密集型进程易触发超时错误
5. **平台依赖**：不同操作系统时序精度存在差异
