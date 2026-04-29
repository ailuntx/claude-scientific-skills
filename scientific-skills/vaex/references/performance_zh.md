# 性能与优化

本文档涵盖 Vaex 的性能特性，包括惰性求值、缓存、内存管理、异步操作以及处理海量数据集的优化策略。

## 理解惰性求值

惰性求值是 Vaex 性能的基石：

### 惰性求值的工作原理

```python
import vaex

df = vaex.open('large_file.hdf5')

# 此处不进行计算 - 仅定义计算逻辑
df['total'] = df.price * df.quantity
df['log_price'] = df.price.log()
mean_expr = df.total.mean()

# 实际计算在此处发生（当需要结果时）
result = mean_expr  # 此时才真正计算均值
```

**核心概念：**
- **表达式是惰性的** - 定义计算但不执行
- **物化操作**在访问结果时发生
- **查询优化**在执行前自动完成

### 求值何时发生？

```python
# 这些操作触发求值：
print(df.x.mean())                    # 访问数值
array = df.x.values                   # 获取 NumPy 数组
pdf = df.to_pandas_df()               # 转换为 pandas
df.export_hdf5('output.hdf5')         # 导出数据

# 这些操作不触发求值：
df['new_col'] = df.x + df.y           # 创建虚拟列
expr = df.x.mean()                    # 创建表达式
df_filtered = df[df.x > 10]           # 创建筛选视图
```

## 使用 delay=True 批量操作

合并执行多个操作以提升性能：

### 基础延迟执行

```python
# 无延迟 - 每个操作都遍历整个数据集
mean_x = df.x.mean()      # 第一次遍历数据
std_x = df.x.std()        # 第二次遍历数据
max_x = df.x.max()        # 第三次遍历数据

# 启用延迟 - 单次遍历数据集
mean_x = df.x.mean(delay=True)
std_x = df.x.std(delay=True)
max_x = df.x.max(delay=True)
results = vaex.execute([mean_x, std_x, max_x])  # 单次遍历完成！

print(results[0])  # 均值
print(results[1])  # 标准差
print(results[2])  # 最大值
```

### 多列延迟执行

```python
# 高效计算多列统计量
stats = {}
delayed_results = []

for column in ['sales', 'quantity', 'profit', 'cost']:
    mean = df[column].mean(delay=True)
    std = df[column].std(delay=True)
    delayed_results.extend([mean, std])

# 一次性执行所有操作
results = vaex.execute(delayed_results)

# 处理结果
for i, column in enumerate(['sales', 'quantity', 'profit', 'cost']):
    stats[column] = {
        'mean': results[i*2],
        'std': results[i*2 + 1]
    }
```

### 何时使用 delay=True

在以下场景使用 `delay=True`：
- 计算多个聚合操作
- 计算多列统计量
- 构建仪表盘或报表
- 任何需要多次遍历数据的场景

```python
# 低效：4次遍历数据集
mean1 = df.col1.mean()
mean2 = df.col2.mean()
mean3 = df.col3.mean()
mean4 = df.col4.mean()

# 高效：单次遍历数据集
results = vaex.execute([
    df.col1.mean(delay=True),
    df.col2.mean(delay=True),
    df.col3.mean(delay=True),
    df.col4.mean(delay=True)
])
```

## 异步操作

使用 async/await 异步处理数据：

### 使用 async/await 实现异步

```python
import vaex
import asyncio

async def compute_statistics(df):
    # 创建异步任务
    mean_task = df.x.mean(delay=True)
    std_task = df.x.std(delay=True)

    # 异步执行
    results = await vaex.async_execute([mean_task, std_task])

    return {'mean': results[0], 'std': results[1]}

# 运行异步函数
async def main():
    df = vaex.open('large_file.hdf5')
    stats = await compute_statistics(df)
    print(stats)

asyncio.run(main())
```

### 使用 Promise/Future 对象

```python
# 获取 future 对象
future = df.x.mean(delay=True)

# 执行其他操作...

# 结果就绪后获取
result = future.get()  # 阻塞直至完成
```

## 虚拟列 vs 物化列

理解差异对性能至关重要：

### 虚拟列（推荐）

```python
# 虚拟列 - 动态计算，零内存占用
df['total'] = df.price * df.quantity
df['log_sales'] = df.sales.log()
df['full_name'] = df.first_name + ' ' + df.last_name

# 检查是否为虚拟列
print(df.is_local('total'))  # False = 虚拟列

# 优势：
# - 零内存开销
# - 源数据变更时自动更新
# - 创建速度快
```

### 物化列

```python
# 将虚拟列物化
df['total_materialized'] = df['total'].values

# 或使用 materialize 方法
df = df.materialize(df['total'], inplace=True)

# 检查是否为物化列
print(df.is_local('total_materialized'))  # True = 物化列

# 物化场景：
# - 需重复计算的列（分摊成本）
# - 复杂表达式被多次使用
# - 需要导出数据
```

### 决策：虚拟列 vs 物化列

```python
# 适合虚拟列的场景：
# - 简单计算（x + y, x * 2 等）
# - 列使用频率低
# - 内存受限

# 适合物化的场景：
# - 复杂计算（多步操作）
# - 在聚合中重复使用
# - 拖慢其他操作

# 示例：多次使用的复杂计算
df['complex'] = (df.x.log() * df.y.sqrt() + df.z ** 2).values  # 物化
```

## 缓存策略

Vaex 自动缓存部分操作，但可进一步优化：

### 自动缓存

```python
# 首次调用计算并缓存
mean1 = df.x.mean()  # 执行计算

# 后续调用使用缓存
mean2 = df.x.mean()  # 从缓存读取（瞬时完成）

# DataFrame 变更时缓存失效
df['new_col'] = df.x + 1
mean3 = df.x.mean()  # 重新计算
```

### 状态管理

```python
# 保存 DataFrame 状态（含虚拟列）
df.state_write('state.json')

# 后续加载状态
df_new = vaex.open('data.hdf5')
df_new.state_load('state.json')  # 恢复虚拟列和筛选状态
```

### 检查点模式

```python
# 为复杂流程保存中间结果
df['processed'] = complex_calculation(df)

# 保存检查点
df.export_hdf5('checkpoint.hdf5')

# 从检查点恢复
df = vaex.open('checkpoint.hdf5')
# 继续处理...
```

## 内存管理

优化海量数据集的内存使用：

### 内存映射文件

```python
# HDF5 和 Arrow 支持内存映射（最优方案）
df = vaex.open('data.hdf5')  # 访问前不占用内存

# 文件保留在磁盘，仅加载访问部分到内存
mean = df.x.mean()  # 流式处理数据，最小化内存占用
```

### 分块处理

```python
# 分块处理大型 DataFrame
chunk_size = 1_000_000

for i1, i2, chunk in df.to_pandas_df(chunk_size=chunk_size):
    # 处理分块（注意：违背 Vaex 设计初衷）
    process_chunk(chunk)

# 更优方案：直接使用 Vaex 操作（无需分块）
result = df.x.mean()  # 自动处理大数据
```

### 监控内存使用

```python
# 检查 DataFrame 内存占用
print(df.byte_size())  # 物化列占用的字节数

# 检查列内存
for col in df.get_column_names():
    if df.is_local(col):
        print(f"{col}: {df[col].nbytes / 1e9:.2f} GB")

# 操作性能分析
import vaex.profiler
with vaex.profiler():
    result = df.x.mean()
```

## 并行计算

Vaex 自动并行化操作：

### 多线程

```python
# Vaex 默认使用所有 CPU 核心
import vaex

# 查看/设置线程数
print(vaex.multithreading.thread_count_default)
vaex.multithreading.thread_count_default = 8  # 使用 8 线程

# 操作自动并行化
mean = df.x.mean()  # 使用全部线程
```

### 基于 Dask 的分布式计算

```python
# 转换为 Dask 实现分布式处理
import vaex
import dask.dataframe as dd

# 创建 Vaex DataFrame
df_vaex = vaex.open('large_file.hdf5')

# 转换为 Dask
df_dask = df_vaex.to_dask_dataframe()

# 使用 Dask 处理
result = df_dask.groupby('category')['value'].sum().compute()
```

## JIT 编译

Vaex 支持使用即时编译处理自定义操作：

### 使用 Numba

```python
import vaex
import numba

# 定义 JIT 编译函数
@numba.jit
def custom_calculation(x, y):
    return x ** 2 + y ** 2

# 应用到 DataFrame
df['custom'] = df.apply(custom_calculation,
                        arguments=[df.x, df.y],
                        vectorize=True)
```

### 自定义聚合

```python
@numba.jit
def custom_sum(a):
    total = 0
    for val in a:
        total += val * 2  # 自定义逻辑
    return total

# 用于聚合操作
result = df.x.custom_agg(custom_sum)
```

## 优化策略

### 策略一：最小化物化操作

```python
# 低效：创建多个物化列
df['a'] = (df.x + df.y).values
df['b'] = (df.a * 2).values
df['c'] = (df.b + df.z).values

# 高效：保持虚拟状态直至最终导出
df['a'] = df.x + df.y
df['b'] = df.a * 2
df['c'] = df.b + df.z
# 仅在导出时物化：
# df.export_hdf5('output.hdf5')
```

### 策略二：使用选择器替代筛选

```python
# 低效：创建新 DataFrame
df_high = df[df.value > 100]
df_low = df[df.value <= 100]
mean_high = df_high.value.mean()
mean_low = df_low.value.mean()

# 高效：使用选择器
df.select(df.value > 100, name='high')
df.select(df.value <= 100, name='low')
mean_high = df.value.mean(selection='high')
mean_low = df.value.mean(selection='low')
```

### 策略三：批量聚合操作

```python
# 低效：多次遍历
stats = {
    'mean': df.x.mean(),
    'std': df.x.std(),
    'min': df.x.min(),
    'max': df.x.max()
}

# 高效：单次遍历
delayed = [
    df.x.mean(delay=True),
    df.x.std(delay=True),
    df.x.min(delay=True),
    df.x.max(delay=True)
]
results = vaex.execute(delayed)
stats = dict(zip(['mean', 'std', 'min', 'max'], results))
```

### 策略四：选择最优文件格式

```python
# 低速：大型 CSV
df = vaex.from_csv('huge.csv')  # 可能耗时数分钟

# 高速：HDF5 或 Arrow
df = vaex.open('huge.hdf5')     # 瞬时加载
df = vaex.open('huge.arrow')    # 瞬时加载

# 单次转换
df = vaex.from_csv('huge.csv', convert='huge.hdf5')
# 后续加载：vaex.open('huge.hdf5')
```

### 策略五：优化表达式

```python
# 低效：重复计算
df['result'] = df.x.log() + df.x.log() * 2

# 高效：复用计算结果
df['log_x'] = df.x.log()
df['result'] = df.log_x + df.log_x * 2

# 更优方案：合并操作
df['result'] = df.x.log() * 3  # 简化数学运算
```

## 性能分析

### 基础性能分析

```python
import time
import vaex

df = vaex.open('large_file.hdf5')

# 计时操作
start = time.time()
result = df.x.mean()
elapsed = time.time() - start
print(f"计算耗时 {elapsed:.2f} 秒")
```

### 详细性能分析

```python
# 使用上下文管理器分析
with vaex.profiler():
    result = df.groupby('category').agg({'value': 'sum'})
# 打印详细时间信息
```

### 基准测试模式

```python
# 比较不同策略
def benchmark_operation(operation, name):
    start = time.time()
    result = operation()
    elapsed = time.time() - start
    print(f"{name}: {elapsed:.3f}s")
    return result

# 测试不同方法
benchmark_operation(lambda: df.x.mean(), "直接求均值")
benchmark_operation(lambda: df[df.x > 0].x.mean(), "筛选后求均值")
benchmark_operation(lambda: df.x.mean(selection='positive'), "选择器求均值")
```

## 常见性能问题与解决方案

### 问题：聚合操作缓慢

```python
# 问题：多个独立聚合操作
for col in df.column_names:
    print(f"{col}: {df[col].mean()}")

# 解决方案：使用 delay=True 批量处理
delayed = [df[col].mean(delay=True) for col in df.column_names]
results = vaex.execute(delayed)
for col, result in zip(df.column_names, results):
    print(f"{col}: {result}")
```

### 问题：内存占用过高

```python
# 问题：物化大型虚拟列
df['large_col'] = (complex_expression).values

# 解决方案：保持虚拟状态，或物化后导出
df['large_col'] = complex_expression  # 保持虚拟
# 或：df.export_hdf5('with_new_col.hdf5')
```

### 问题：导出速度慢

```python
# 问题：导出含大量虚拟列的数据
df.export_csv('output.csv')  # 虚拟列过多时缓慢

# 解决方案：导出为 HDF5 或 Arrow（更快）
df.export_hdf5('output.hdf5')
df.export_arrow('output.arrow')

# 或先物化再导出 CSV
df_materialized = df.materialize()
df_materialized.export_csv('output.csv')
```

### 问题：重复复杂计算

```python
# 问题：复杂虚拟列被重复使用
df['complex'] = df.x.log() * df.y.sqrt() + df.z ** 3
result1 = df.groupby('cat1').agg({'complex': 'mean'})
result2 = df.groupby('cat2').agg({'complex': 'sum'})
result3 = df.complex.std()

# 解决方案：单次物化
df['complex'] = (df.x.log() * df.y.sqrt() + df.z ** 3).values
# 或：df = df.materialize('complex')
```

## 性能最佳实践总结

1. **使用 HDF5 或 Arrow 格式** - 比 CSV 快数个数量级
2. **利用惰性求值** - 非必要不触发计算
3. **使用 delay=True 批量操作** -
