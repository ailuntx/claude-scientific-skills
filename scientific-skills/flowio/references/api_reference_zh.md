# FlowIO API 参考文档

## 概述

FlowIO 是一个用于读写流式细胞术标准（FCS）文件的 Python 库。它支持 FCS 2.0、3.0 和 3.1 版本，且依赖项极少。

## 安装

```bash
pip install flowio
```

支持 Python 3.9 及更高版本。

## 核心类

### FlowData

处理 FCS 文件的主类。

#### 构造函数

```python
FlowData(fcs_file,
         ignore_offset_error=False,
         ignore_offset_discrepancy=False,
         use_header_offsets=False,
         only_text=False,
         nextdata_offset=None,
         null_channel_list=None)
```

**参数：**
- `fcs_file`：文件路径（字符串）、Path 对象或文件句柄
- `ignore_offset_error` (布尔值)：忽略偏移错误（默认：False）
- `ignore_offset_discrepancy` (布尔值)：忽略 HEADER 和 TEXT 段之间的偏移差异（默认：False）
- `use_header_offsets` (布尔值)：使用 HEADER 段偏移而非 TEXT 段（默认：False）
- `only_text` (布尔值)：仅解析 TEXT 段，跳过 DATA 和 ANALYSIS（默认：False）
- `nextdata_offset` (整型)：读取多数据集文件的字节偏移量
- `null_channel_list` (列表)：要排除的空通道 PnN 标签列表

#### 属性

**文件信息：**
- `name`：FCS 文件名
- `file_size`：文件大小（字节）
- `version`：FCS 版本（如 '3.0', '3.1'）
- `header`：包含 HEADER 段信息的字典
- `data_type`：数据格式类型（'I', 'F', 'D', 'A'）

**通道信息：**
- `channel_count`：数据集中的通道数量
- `channels`：通道号到通道信息的映射字典
- `pnn_labels`：PnN（通道短名称）标签列表
- `pns_labels`：PnS（染色描述名称）标签列表
- `pnr_values`：各通道 PnR（量程）值列表
- `fluoro_indices`：荧光通道索引列表
- `scatter_indices`：散射通道索引列表
- `time_index`：时间通道索引（若无则为 None）
- `null_channels`：空通道索引列表

**事件数据：**
- `event_count`：数据集中的事件数（行数）
- `events`：原始事件数据（字节格式）

**元数据：**
- `text`：TEXT 段键值对字典
- `analysis`：ANALYSIS 段键值对字典（如果存在）

#### 方法

##### as_array()

```python
as_array(preprocess=True)
```

将事件数据作为二维 NumPy 数组返回。

**参数：**
- `preprocess` (布尔值)：应用增益、对数及时间缩放转换（默认：True）

**返回值：**
- 形状为 (event_count, channel_count) 的 NumPy ndarray

**示例：**
```python
flow_data = FlowData('sample.fcs')
events_array = flow_data.as_array()  # 预处理后数据
raw_array = flow_data.as_array(preprocess=False)  # 原始数据
```

##### write_fcs()

```python
write_fcs(filename, metadata=None)
```

将 FlowData 实例导出为新的 FCS 文件。

**参数：**
- `filename` (字符串)：输出文件路径
- `metadata` (字典)：可选的 TEXT 段关键词字典（用于添加/更新）

**示例：**
```python
flow_data = FlowData('sample.fcs')
flow_data.write_fcs('output.fcs', metadata={'$SRC': '修改后数据'})
```

**注意：** 导出为 FCS 3.1 格式的单精度浮点数据。

## 实用函数

### read_multiple_data_sets()

```python
read_multiple_data_sets(fcs_file,
                        ignore_offset_error=False,
                        ignore_offset_discrepancy=False,
                        use_header_offsets=False)
```

读取包含多个数据集的 FCS 文件中的所有数据集。

**参数：**
- 与 FlowData 构造函数相同（除 `nextdata_offset` 外）

**返回值：**
- FlowData 实例列表（每个数据集对应一个实例）

**示例：**
```python
from flowio import read_multiple_data_sets

datasets = read_multiple_data_sets('multi_dataset.fcs')
print(f"找到 {len(datasets)} 个数据集")
for i, dataset in enumerate(datasets):
    print(f"数据集 {i}: {dataset.event_count} 个事件")
```

### create_fcs()

```python
create_fcs(filename,
           event_data,
           channel_names,
           opt_channel_names=None,
           metadata=None)
```

从事件数据创建新的 FCS 文件。

**参数：**
- `filename` (字符串)：输出文件路径
- `event_data` (ndarray)：事件数据的二维 NumPy 数组（行=事件，列=通道）
- `channel_names` (列表)：PnN（短名称）通道名称列表
- `opt_channel_names` (列表)：可选的 PnS（描述性）通道名称列表
- `metadata` (字典)：可选的 TEXT 段关键词字典

**示例：**
```python
import numpy as np
from flowio import create_fcs

# 创建合成数据
events = np.random.rand(10000, 5)
channels = ['FSC-A', 'SSC-A', 'FL1-A', 'FL2-A', 'Time']
opt_channels = ['Forward Scatter', 'Side Scatter', 'FITC', 'PE', 'Time']

create_fcs('synthetic.fcs',
           events,
           channels,
           opt_channel_names=opt_channels,
           metadata={'$SRC': '合成数据'})
```

## 异常类

### FlowIOWarning

非关键问题的通用警告类。

### PnEWarning

创建 FCS 文件时 PnE 值无效引发的警告。

### FlowIOException

FlowIO 错误的基异常类。

### FCSParsingError

解析 FCS 文件时出现问题的异常。

### DataOffsetDiscrepancyError

当 HEADER 和 TEXT 段提供的数据段字节偏移不一致时引发。

**解决方法：** 创建 FlowData 实例时使用 `ignore_offset_discrepancy=True` 参数。

### MultipleDataSetsError

尝试使用标准 FlowData 构造函数读取多数据集文件时引发。

**解决方案：** 改用 `read_multiple_data_sets()` 函数。

## FCS 文件结构参考

FCS 文件包含四个段：

1. **HEADER**：包含 FCS 版本和其他段的字节位置
2. **TEXT**：键值对元数据（分隔格式）
3. **DATA**：原始事件数据（二进制、浮点或 ASCII）
4. **ANALYSIS**（可选）：数据处理结果

### 常见 TEXT 段关键词

- `$BEGINDATA`, `$ENDDATA`：DATA 段的字节偏移
- `$BEGINANALYSIS`, `$ENDANALYSIS`：ANALYSIS 段的字节偏移
- `$BYTEORD`：字节序（1,2,3,4 表示小端序；4,3,2,1 表示大端序）
- `$DATATYPE`：数据类型（'I'=整型, 'F'=浮点, 'D'=双精度, 'A'=ASCII）
- `$MODE`：数据模式（'L'=列表模式，最常见）
- `$NEXTDATA`：下一数据集的偏移量（单数据集时为 0）
- `$PAR`：参数（通道）数量
- `$TOT`：事件总数
- `PnN`：参数 n 的短名称
- `PnS`：参数 n 的染色描述名称
- `PnR`：参数 n 的量程（最大值）
- `PnE`：参数 n 的放大指数（格式："a,b"，计算值 = a * 10^(b*x)）
- `PnG`：参数 n 的放大增益

## 通道类型

FlowIO 自动分类通道：

- **散射通道**：FSC（前向散射）、SSC（侧向散射）
- **荧光通道**：FL1、FL2、FITC、PE 等
- **时间通道**：通常标记为 "Time"

通过以下属性访问索引：
- `flow_data.scatter_indices`
- `flow_data.fluoro_indices`
- `flow_data.time_index`

## 数据预处理

调用 `as_array(preprocess=True)` 时，FlowIO 会应用：

1. **增益缩放**：乘以 PnG 值
2. **对数转换**：若存在 PnE 则应用指数转换
3. **时间缩放**：将时间值转换为适当单位

访问原始未处理数据：`as_array(preprocess=False)`

## 最佳实践

1. **内存效率**：仅需元数据时使用 `only_text=True`
2. **错误处理**：用 try-except 块包裹文件操作以捕获 FCSParsingError
3. **多数据集文件**：不确定数据集数量时始终使用 `read_multiple_data_sets()`
4. **偏移问题**：遇到偏移错误时尝试 `ignore_offset_discrepancy=True`
5. **通道选择**：解析时使用 null_channel_list 排除不需要的通道

## 与 FlowKit 集成

如需进行补偿、设门和 GatingML 支持等高级流式分析，建议将 FlowKit 库与 FlowIO 配合使用。FlowKit 在 FlowIO 文件解析能力基础上提供了更高级的抽象层。

## 示例工作流

### 基础文件读取

```python
from flowio import FlowData

# 读取 FCS 文件
flow = FlowData('experiment.fcs')

# 打印基础信息
print(f"版本: {flow.version}")
print(f"事件数: {flow.event_count}")
print(f"通道数: {flow.channel_count}")
print(f"通道名称: {flow.pnn_labels}")

# 获取事件数据
events = flow.as_array()
print(f"数据形状: {events.shape}")
```

### 元数据提取

```python
from flowio import FlowData

flow = FlowData('sample.fcs', only_text=True)

# 访问元数据
print(f"采集日期: {flow.text.get('$DATE', 'N/A')}")
print(f"仪器: {flow.text.get('$CYT', 'N/A')}")

# 通道信息
for i, (pnn, pns) in enumerate(zip(flow.pnn_labels, flow.pns_labels)):
    print(f"通道 {i}: {pnn} ({pns})")
```

### 创建新 FCS 文件

```python
import numpy as np
from flowio import create_fcs

# 生成或处理数据
data = np.random.rand(5000, 3) * 1000

# 定义通道
channels = ['FSC-A', 'SSC-A', 'FL1-A']
stains = ['Forward Scatter', 'Side Scatter', 'GFP']

# 创建 FCS 文件
create_fcs('output.fcs',
           data,
           channels,
           opt_channel_names=stains,
           metadata={
               '$SRC': 'Python 脚本',
               '$DATE': '2025年10月19日'
           })
```

### 处理多数据集文件

```python
from flowio import read_multiple_data_sets

# 读取所有数据集
datasets = read_multiple_data_sets('multi.fcs')

# 处理每个数据集
for i, dataset in enumerate(datasets):
    print(f"\n数据集 {i}:")
    print(f"  事件数: {dataset.event_count}")
    print(f"  通道: {dataset.pnn_labels}")

    # 获取数据数组
    events = dataset.as_array()
    mean_values = events.mean(axis=0)
    print(f"  均值: {mean_values}")
```

### 修改并重新导出

```python
from flowio import FlowData

# 读取原始文件
flow = FlowData('original.fcs')

# 获取事件数据
events = flow.as_array(preprocess=False)

# 修改数据（示例：应用自定义转换）
events[:, 0] = events[:, 0] * 1.5  # 缩放第一通道

# 注意：当前 FlowIO 不支持直接修改事件数据
# 修改数据请改用 create_fcs()：
from flowio import create_fcs

create_fcs('modified.fcs',
           events,
           flow.pnn_labels,
           opt_channel_names=flow.pns_labels,
           metadata=flow.text)
```
