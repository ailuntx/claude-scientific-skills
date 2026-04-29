# Dask Bags

## 概述

Dask Bag 实现了对通用 Python 对象的函数式操作，包括 `map`、`filter`、`fold` 和 `groupby`。它通过 Python 迭代器在并行处理数据的同时保持较小的内存占用。Bag 可视为 "PyToolz 的并行版本或 PySpark RDD 的 Pythonic 实现"。

## 核心概念

Dask Bag 是分布在多个分区的 Python 对象集合：
- 每个分区包含通用 Python 对象
- 操作采用函数式编程模式
- 处理使用流式/迭代器实现内存高效
- 适用于非结构化或半结构化数据

## 关键能力

### 函数式操作
- `map`: 转换每个元素
- `filter`: 基于条件筛选元素
- `fold`: 使用组合函数归约元素
- `groupby`: 按键分组元素
- `pluck`: 从记录中提取字段
- `flatten`: 展平嵌套结构

### 使用场景
- 文本处理和日志分析
- JSON 记录处理
- 非结构化数据的 ETL
- 结构化分析前的数据清洗

## 何时使用 Dask Bags

**适用场景**：
- 处理需要灵活计算的通用 Python 对象
- 数据不符合结构化数组或表格格式
- 处理文本、JSON 或自定义 Python 对象
- 需要初始数据清洗和 ETL
- 内存高效的流式处理很重要

**其他集合适用场景**：
- 结构化数据（改用 DataFrame）
- 数值计算（改用 Array）
- 需要复杂分组或洗牌的操作（改用 DataFrame）

**关键建议**：使用 Bag 清洗处理数据，然后将其转换为数组或 DataFrame，再进行需要洗牌步骤的复杂操作。

## 重要限制

Bags 为通用性牺牲性能：
- 依赖多进程调度（非线程）
- 保持不可变性（需创建新 bag 进行更改）
- 比数组/DataFrame 等效操作慢
- `groupby` 效率低（尽可能使用 `foldby`）
- 需要大量工作节点通信的操作较慢

## 创建 Bags

### 从序列创建
```python
import dask.bag as db

# 从 Python 列表创建
bag = db.from_sequence([1, 2, 3, 4, 5], partition_size=2)

# 从 range 创建
bag = db.from_sequence(range(10000), partition_size=1000)
```

### 从文本文件创建
```python
# 单个文件
bag = db.read_text('data.txt')

# 使用通配符匹配多个文件
bag = db.read_text('data/*.txt')

# 指定编码
bag = db.read_text('data/*.txt', encoding='utf-8')

# 自定义行处理
bag = db.read_text('logs/*.log', blocksize='64MB')
```

### 从延迟对象创建
```python
import dask

@dask.delayed
def load_data(filename):
    with open(filename) as f:
        return [line.strip() for line in f]

files = ['file1.txt', 'file2.txt', 'file3.txt']
partitions = [load_data(f) for f in files]
bag = db.from_delayed(partitions)
```

### 从自定义源创建
```python
# 从任意生成迭代器的函数创建
def read_json_files():
    import json
    for filename in glob.glob('data/*.json'):
        with open(filename) as f:
            yield json.load(f)

# 从生成器创建 bag
bag = db.from_sequence(read_json_files(), partition_size=10)
```

## 常用操作

### Map (转换)
```python
import dask.bag as db

bag = db.read_text('data/*.json')

# 解析 JSON
import json
parsed = bag.map(json.loads)

# 提取字段
values = parsed.map(lambda x: x['value'])

# 复杂转换
def process_record(record):
    return {
        'id': record['id'],
        'value': record['value'] * 2,
        'category': record.get('category', 'unknown')
    }

processed = parsed.map(process_record)
```

### Filter (筛选)
```python
# 按条件筛选
valid = parsed.filter(lambda x: x['status'] == 'valid')

# 多条件筛选
filtered = parsed.filter(lambda x: x['value'] > 100 and x['year'] == 2024)

# 使用自定义函数筛选
def is_valid_record(record):
    return record.get('status') == 'valid' and record.get('value') is not None

valid_records = parsed.filter(is_valid_record)
```

### Pluck (提取字段)
```python
# 提取单个字段
ids = parsed.pluck('id')

# 提取多个字段（生成元组）
key_pairs = parsed.pluck(['id', 'value'])
```

### Flatten (展平)
```python
# 展平嵌套列表
nested = db.from_sequence([[1, 2], [3, 4], [5, 6]])
flat = nested.flatten()  # [1, 2, 3, 4, 5, 6]

# 在 map 后展平
bag = db.read_text('data/*.txt')
words = bag.map(str.split).flatten()  # 所有文件的所有单词
```

### GroupBy (高开销)
```python
# 按键分组（需要洗牌）
grouped = parsed.groupby(lambda x: x['category'])

# 分组后聚合
counts = grouped.map(lambda key_items: (key_items[0], len(list(key_items[1]))))
result = counts.compute()
```

### FoldBy (聚合首选)
```python
# 对于聚合操作，foldby 比 groupby 更高效
def add(acc, item):
    return acc + item['value']

def combine(acc1, acc2):
    return acc1 + acc2

# 按类别求和
sums = parsed.foldby(
    key='category',
    binop=add,
    initial=0,
    combine=combine
)

result = sums.compute()
```

### 归约操作
```python
# 元素计数
count = bag.count().compute()

# 获取所有唯一值（需内存）
distinct = bag.distinct().compute()

# 获取前 n 个元素
first_ten = bag.take(10)

# 折叠/归约
total = bag.fold(
    lambda acc, x: acc + x['value'],
    initial=0,
    combine=lambda a, b: a + b
).compute()
```

## 转换为其他集合

### 转为 DataFrame
```python
import dask.bag as db
import dask.dataframe as dd

# 字典组成的 bag
bag = db.read_text('data/*.json').map(json.loads)

# 转换为 DataFrame
ddf = bag.to_dataframe()

# 指定列元数据
ddf = bag.to_dataframe(meta={'id': int, 'value': float, 'category': str})
```

### 转为列表/计算
```python
# 计算为 Python 列表（加载到内存）
result = bag.compute()

# 获取样本
sample = bag.take(100)
```

## 常用模式

### JSON 处理
```python
import dask.bag as db
import json

# 读取并解析 JSON 文件
bag = db.read_text('logs/*.json')
parsed = bag.map(json.loads)

# 筛选有效记录
valid = parsed.filter(lambda x: x.get('status') == 'success')

# 提取相关字段
processed = valid.map(lambda x: {
    'user_id': x['user']['id'],
    'timestamp': x['timestamp'],
    'value': x['metrics']['value']
})

# 转换为 DataFrame 进行分析
ddf = processed.to_dataframe()

# 分析
summary = ddf.groupby('user_id')['value'].mean().compute()
```

### 日志分析
```python
# 读取日志文件
logs = db.read_text('logs/*.log')

# 解析日志行
def parse_log_line(line):
    parts = line.split(' ')
    return {
        'timestamp': parts[0],
        'level': parts[1],
        'message': ' '.join(parts[2:])
    }

parsed_logs = logs.map(parse_log_line)

# 筛选错误
errors = parsed_logs.filter(lambda x: x['level'] == 'ERROR')

# 按消息模式计数
error_counts = errors.foldby(
    key='message',
    binop=lambda acc, x: acc + 1,
    initial=0,
    combine=lambda a, b: a + b
)

result = error_counts.compute()
```

### 文本处理
```python
# 读取文本文件
text = db.read_text('documents/*.txt')

# 拆分为单词
words = text.map(str.lower).map(str.split).flatten()

# 词频统计
def increment(acc, word):
    return acc + 1

def combine_counts(a, b):
    return a + b

word_counts = words.foldby(
    key=lambda word: word,
    binop=increment,
    initial=0,
    combine=combine_counts
)

# 获取高频词
top_words = word_counts.compute()
sorted_words = sorted(top_words, key=lambda x: x[1], reverse=True)[:100]
```

### 数据清洗流程
```python
import dask.bag as db
import json

# 读取原始数据
raw = db.read_text('raw_data/*.json').map(json.loads)

# 验证函数
def is_valid(record):
    required_fields = ['id', 'timestamp', 'value']
    return all(field in record for field in required_fields)

# 清洗函数
def clean_record(record):
    return {
        'id': int(record['id']),
        'timestamp': record['timestamp'],
        'value': float(record['value']),
        'category': record.get('category', 'unknown'),
        'tags': record.get('tags', [])
    }

# 处理流程
cleaned = (raw
    .filter(is_valid)
    .map(clean_record)
    .filter(lambda x: x['value'] > 0)
)

# 转换为 DataFrame
ddf = cleaned.to_dataframe()

# 保存清洗后数据
ddf.to_parquet('cleaned_data/')
```

## 性能考量

### 高效操作
- Map、filter、pluck：非常高效（流式）
- Flatten：高效
- FoldBy（键分布良好时）：合理
- Take 和 head：高效（仅处理所需分区）

### 高开销操作
- GroupBy：需要洗牌，可能较慢
- Distinct：需要收集所有唯一值
- 需要完全数据物化的操作

### 优化技巧

**1. 优先使用 FoldBy 替代 GroupBy**
```python
# 更优：使用 foldby 进行聚合
result = bag.foldby(key='category', binop=add, initial=0, combine=sum)

# 较差：GroupBy 后归约
result = bag.groupby('category').map(lambda x: (x[0], sum(x[1])))
```

**2. 尽早转换为 DataFrame**
```python
# 结构化操作尽早转 DataFrame
bag = db.read_text('data/*.json').map(json.loads)
bag = bag.filter(lambda x: x['status'] == 'valid')
ddf = bag.to_dataframe()  # 使用高效的 DataFrame 操作
```

**3. 控制分区大小**
```python
# 平衡分区数量
bag = db.read_text('data/*.txt', blocksize='64MB')  # 合理分区大小
```

**4. 利用惰性求值**
```python
# 链式操作后再计算
result = (bag
    .map(process1)
    .filter(condition)
    .map(process2)
    .compute()  # 最后统一计算
)
```

## 调试技巧

### 检查分区
```python
# 获取分区数量
print(bag.npartitions)

# 获取样本
sample = bag.take(10)
print(sample)
```

### 小数据验证
```python
# 在子集测试逻辑
small_bag = db.from_sequence(sample_data, partition_size=10)
result = process_pipeline(small_bag).compute()
# 验证结果后再扩展
```

### 检查中间结果
```python
# 计算中间步骤进行调试
step1 = bag.map(parse).take(5)
print("解析后:", step1)

step2 = bag.map(parse).filter(validate).take(5)
print("筛选后:", step2)
```

## 内存管理

Bags 专为内存高效处理设计：

```python
# 流式处理 - 不加载全部数据到内存
bag = db.read_text('huge_file.txt')  # 惰性
processed = bag.map(process_line)     # 仍为惰性
result = processed.compute()          # 分块处理
```

对于超大结果，避免直接计算到内存：

```python
# 不要将巨大结果计算到内存
# result = bag.compute()  # 可能导致内存溢出

# 应转换为 DataFrame 并保存到磁盘
ddf = bag.to_dataframe()
ddf.to_parquet('output/')
```
