# 配置

## 概述

polars-bio 使用基于 `set_option` 和 `get_option` 的全局配置系统来控制执行行为、坐标系、并行性和流处理模式。

## set_option / get_option

```python
import polars_bio as pb

# 设置配置选项
pb.set_option("datafusion.execution.target_partitions", 8)

# 获取当前值
value = pb.get_option("datafusion.execution.target_partitions")
```

## 并行性

### DataFusion 目标分区数

控制并行执行分区的数量。默认为 1（单线程）。

```python
import os
import polars_bio as pb

# 使用所有可用的 CPU 核心
pb.set_option("datafusion.execution.target_partitions", os.cpu_count())

# 设置特定分区数
pb.set_option("datafusion.execution.target_partitions", 8)
```

**何时增加并行性：**
- 处理大型文件（>1GB）
- 在数百万个区间上运行区间操作
- 批量处理多个染色体

**何时保持默认值（1）：**
- 小型数据集
- 内存受限环境
- 调试（确定性执行）

## 坐标系

polars-bio 默认使用 1-based 坐标系（标准基因组惯例）。

### 全局坐标系

```python
import polars_bio as pb

# 切换到 0-based 半开坐标系
pb.set_option("coordinate_system", "0-based")

# 切换回 1-based（默认）
pb.set_option("coordinate_system", "1-based")

# 检查当前设置
print(pb.get_option("coordinate_system"))
```

### 通过 I/O 函数实现按文件覆盖

I/O 函数接受 `use_zero_based` 参数，用于在结果 DataFrame 上设置坐标系元数据：

```python
# 读取时显式指定 0-based 元数据
df = pb.read_bed("regions.bed", use_zero_based=True)
```

**注意：** 区间操作（如重叠、最近邻等）**不**接受 `use_zero_based` 参数。它们从 DataFrame 中读取坐标系元数据，这些元数据由 I/O 函数或全局选项设置。当使用手动构建的 DataFrame 时，polars-bio 会警告元数据缺失并回退到全局设置。

### 在手动构建的 DataFrame 上设置元数据

```python
import polars_bio as pb

# 在手动创建的 DataFrame 上设置坐标系元数据
pb.set_source_metadata(df, format="bed", path="")
```

### 文件格式约定

| 格式 | 原生坐标系 | polars-bio 转换方式 |
|------|------------|----------------------|
| BED | 0-based half-open | 读取时转换为配置的坐标系 |
| VCF | 1-based | 读取时转换为配置的坐标系 |
| GFF/GTF | 1-based | 读取时转换为配置的坐标系 |
| BAM/SAM | 0-based | 读取时转换为配置的坐标系 |

## 流处理执行模式

polars-bio 支持两种流处理模式，用于核外处理：

### DataFusion 流处理

默认在区间操作中启用。通过 DataFusion 执行引擎分批处理数据。

```python
# 区间操作自动启用 DataFusion 流处理
result = pb.overlap(lf1, lf2)  # 输入为 LazyFrame 时启用流处理
```

### Polars 流处理

使用 Polars 原生流处理进行后处理操作：

```python
# 使用 Polars 流处理收集结果
result = lf.collect(streaming=True)
```

### 组合使用

```python
import polars_bio as pb

# 惰性扫描文件（DataFusion 流处理用于 I/O）
lf1 = pb.scan_bed("large1.bed")
lf2 = pb.scan_bed("large2.bed")

# 区间操作（DataFusion 流处理）
result_lf = pb.overlap(lf1, lf2)

# 使用 Polars 流处理进行最终物化
result = result_lf.collect(streaming=True)
```

## 日志

控制日志详细程度以进行调试：

```python
import polars_bio as pb

# 设置日志级别
pb.set_loglevel("debug")   # 详细执行信息
pb.set_loglevel("info")    # 标准消息
pb.set_loglevel("warn")    # 仅警告（默认）
```

**注意：** 只有 `"debug"`、`"info"` 和 `"warn"` 是有效的日志级别。

## 元数据管理

polars-bio 会将坐标系和源元数据附加到由 I/O 函数生成的 DataFrame 上。区间操作使用此元数据来确定坐标系。

```python
import polars_bio as pb

# 检查 DataFrame 上的元数据
metadata = pb.get_metadata(df)

# 打印元数据摘要
pb.print_metadata_summary(df)

# 以 JSON 格式打印元数据
pb.print_metadata_json(df)

# 在手动创建的 DataFrame 上设置元数据
pb.set_source_metadata(df, format="bed", path="regions.bed")

# 将 DataFrame 注册为 SQL 表
pb.from_polars("my_table", df)
```

## 完整配置参考

| 选项 | 默认值 | 描述 |
|------|--------|------|
| `datafusion.execution.target_partitions` | `1` | 并行执行分区的数量 |
| `coordinate_system` | `"1-based"` | 默认坐标系（`"0-based"` 或 `"1-based"`） |
