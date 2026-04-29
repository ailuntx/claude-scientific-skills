# Python API 参考文档

gtars Python 绑定的完整参考手册。

## 安装

```bash
# 安装 gtars Python 包
uv pip install gtars

# 或使用 pip
pip install gtars
```

## 核心类

### RegionSet

管理基因组区间的集合：

```python
import gtars

# 从 BED 文件创建
regions = gtars.RegionSet.from_bed("regions.bed")

# 通过坐标创建
regions = gtars.RegionSet([
    ("chr1", 1000, 2000),
    ("chr1", 3000, 4000),
    ("chr2", 5000, 6000)
])

# 访问区间
for region in regions:
    print(f"{region.chromosome}:{region.start}-{region.end}")

# 获取区间数量
num_regions = len(regions)

# 计算总覆盖度
total_coverage = regions.total_coverage()
```

### 区间操作

对区间集合执行操作：

```python
# 排序区间
sorted_regions = regions.sort()

# 合并重叠区间
merged = regions.merge()

# 按大小筛选
large_regions = regions.filter_by_size(min_size=1000)

# 按染色体筛选
chr1_regions = regions.filter_by_chromosome("chr1")
```

### 集合运算

对基因组区间执行集合运算：

```python
# 加载两个区间集合
set_a = gtars.RegionSet.from_bed("set_a.bed")
set_b = gtars.RegionSet.from_bed("set_b.bed")

# 并集
union = set_a.union(set_b)

# 交集
intersection = set_a.intersect(set_b)

# 差集
difference = set_a.subtract(set_b)

# 对称差集
sym_diff = set_a.symmetric_difference(set_b)
```

## 数据导出

### 写入 BED 文件

将区间导出为 BED 格式：

```python
# 写入 BED 文件
regions.to_bed("output.bed")

# 带分值写入
regions.to_bed("output.bed", scores=score_array)

# 带名称写入
regions.to_bed("output.bed", names=name_list)
```

### 格式转换

格式间相互转换：

```python
# BED 转 JSON
regions = gtars.RegionSet.from_bed("input.bed")
regions.to_json("output.json")

# JSON 转 BED
regions = gtars.RegionSet.from_json("input.json")
regions.to_bed("output.bed")
```

## NumPy 集成

与 NumPy 数组无缝集成：

```python
import numpy as np

# 导出到 NumPy 数组
starts = regions.starts_array()  # 起始位置数组
ends = regions.ends_array()      # 终止位置数组
sizes = regions.sizes_array()    # 区间大小数组

# 从 NumPy 数组创建
chromosomes = ["chr1"] * len(starts)
regions = gtars.RegionSet.from_arrays(chromosomes, starts, ends)
```

## 并行处理

利用并行处理处理大型数据集：

```python
# 启用并行处理
regions = gtars.RegionSet.from_bed("large_file.bed", parallel=True)

# 并行操作
result = regions.parallel_apply(custom_function)
```

## 内存管理

针对大型数据集的高效内存使用：

```python
# 流式读取大型 BED 文件
for chunk in gtars.RegionSet.stream_bed("large_file.bed", chunk_size=10000):
    process_chunk(chunk)

# 内存映射模式
regions = gtars.RegionSet.from_bed("large_file.bed", mmap=True)
```

## 错误处理

处理常见错误：

```python
try:
    regions = gtars.RegionSet.from_bed("file.bed")
except gtars.FileNotFoundError:
    print("文件未找到")
except gtars.InvalidFormatError as e:
    print(f"无效的 BED 格式: {e}")
except gtars.ParseError as e:
    print(f"解析错误 (行 {e.line}): {e.message}")
```

## 配置

配置 gtars 行为：

```python
# 设置全局选项
gtars.set_option("parallel.threads", 4)
gtars.set_option("memory.limit", "4GB")
gtars.set_option("warnings.strict", True)

# 使用上下文管理器临时配置
with gtars.option_context("parallel.threads", 8):
    # 在此代码块内使用 8 线程
    regions = gtars.RegionSet.from_bed("large_file.bed", parallel=True)
```

## 日志记录

启用日志记录用于调试：

```python
import logging

# 启用 gtars 日志
gtars.set_log_level("DEBUG")

# 或使用 Python 日志模块
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger("gtars")
```

## 性能优化建议

- 对大型数据集使用并行处理
- 处理超大文件时启用内存映射模式
- 尽可能使用流式处理降低内存占用
- 适用情况下预先对区间进行排序
- 数值计算优先使用 NumPy 数组
- 缓存频繁访问的数据
