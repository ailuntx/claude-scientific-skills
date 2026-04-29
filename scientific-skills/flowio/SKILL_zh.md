---
name: flowio
description: 解析 FCS（流式细胞术标准）文件 v2.0-3.1。将事件提取为 NumPy 数组，读取元数据/通道，转换为 CSV/DataFrame，用于流式细胞术数据预处理。
license: BSD-3-Clause license
metadata:
    skill-author: K-Dense Inc.
---

# FlowIO：流式细胞术标准文件处理器

## 概述

FlowIO 是一个轻量级 Python 库，用于读写流式细胞术标准（FCS）文件。可解析 FCS 元数据、提取事件数据并创建新的 FCS 文件，依赖项极少。该库支持 FCS 2.0、3.0 和 3.1 版本，是后端服务、数据管道和基础细胞术文件操作的理想选择。

## 使用场景

该技能适用于以下场景：

- 需要解析或提取元数据的 FCS 文件
- 需将流式细胞术数据转换为 NumPy 数组
- 需将事件数据导出为 FCS 格式
- 需拆分多数据集 FCS 文件
- 提取通道信息（散射、荧光、时间）
- 细胞术文件验证或检查
- 高级分析前的预处理工作流

**相关工具：** 如需进行补偿、设门和 FlowJo/GatingML 支持等高级流式细胞术分析，推荐将 FlowKit 库作为 FlowIO 的配套工具。

## 安装

```bash
uv pip install flowio
```

需要 Python 3.9 或更高版本。

## 快速入门

### 基础文件读取

```python
from flowio import FlowData

# 读取 FCS 文件
flow_data = FlowData('experiment.fcs')

# 访问基础信息
print(f"FCS 版本: {flow_data.version}")
print(f"事件数: {flow_data.event_count}")
print(f"通道: {flow_data.pnn_labels}")

# 获取 NumPy 数组形式的事件数据
events = flow_data.as_array()  # 形状: (事件数, 通道数)
```

### 创建 FCS 文件

```python
import numpy as np
from flowio import create_fcs

# 准备数据
data = np.array([[100, 200, 50], [150, 180, 60]])  # 2 个事件, 3 个通道
channels = ['FSC-A', 'SSC-A', 'FL1-A']

# 创建 FCS 文件
create_fcs('output.fcs', data, channels)
```

## 核心工作流

### 读取和解析 FCS 文件

FlowData 类提供读取 FCS 文件的主要接口。

**标准读取：**

```python
from flowio import FlowData

# 基础读取
flow = FlowData('sample.fcs')

# 访问属性
version = flow.version              # '3.0', '3.1' 等
event_count = flow.event_count      # 事件数量
channel_count = flow.channel_count  # 通道数量
pnn_labels = flow.pnn_labels        # 通道短名称
pns_labels = flow.pns_labels        # 描述性染色名称

# 获取事件数据
events = flow.as_array()            # 预处理后数据（应用增益、对数缩放）
raw_events = flow.as_array(preprocess=False)  # 原始数据
```

**内存高效元数据读取：**

当仅需元数据时（无需事件数据）：

```python
# 仅解析 TEXT 段，跳过 DATA 和 ANALYSIS
flow = FlowData('sample.fcs', only_text=True)

# 访问元数据
metadata = flow.text  # TEXT 段关键词字典
print(metadata.get('$DATE'))  # 采集日期
print(metadata.get('$CYT'))   # 仪器名称
```

**处理问题文件：**

某些 FCS 文件存在偏移量差异或错误：

```python
# 忽略 HEADER 和 TEXT 段间的偏移量差异
flow = FlowData('problematic.fcs', ignore_offset_discrepancy=True)

# 使用 HEADER 偏移量替代 TEXT 偏移量
flow = FlowData('problematic.fcs', use_header_offsets=True)

# 完全忽略偏移量错误
flow = FlowData('problematic.fcs', ignore_offset_error=True)
```

**排除空通道：**

```python
# 解析时排除特定通道
flow = FlowData('sample.fcs', null_channel_list=['Time', 'Null'])
```

### 提取元数据和通道信息

FCS 文件在 TEXT 段包含丰富的元数据。

**常用元数据关键词：**

```python
flow = FlowData('sample.fcs')

# 文件级元数据
text_dict = flow.text
acquisition_date = text_dict.get('$DATE', 'Unknown')
instrument = text_dict.get('$CYT', 'Unknown')
data_type = flow.data_type  # 'I', 'F', 'D', 'A'

# 通道元数据
for i in range(flow.channel_count):
    pnn = flow.pnn_labels[i]      # 短名称 (如 'FSC-A')
    pns = flow.pns_labels[i]      # 描述性名称 (如 'Forward Scatter')
    pnr = flow.pnr_values[i]      # 范围/最大值
    print(f"通道 {i}: {pnn} ({pns}), 范围: {pnr}")
```

**通道类型识别：**

FlowIO 自动分类通道：

```python
# 按通道类型获取索引
scatter_idx = flow.scatter_indices    # FSC/SSC 对应 [0, 1]
fluoro_idx = flow.fluoro_indices      # FL 通道对应 [2, 3, 4]
time_idx = flow.time_index            # 时间通道索引 (若无则为 None)

# 访问特定通道类型
events = flow.as_array()
scatter_data = events[:, scatter_idx]
fluorescence_data = events[:, fluoro_idx]
```

**ANALYSIS 段：**

若存在，可访问处理结果：

```python
if flow.analysis:
    analysis_keywords = flow.analysis  # ANALYSIS 关键词字典
    print(analysis_keywords)
```

### 创建新 FCS 文件

从 NumPy 数组或其他数据源生成 FCS 文件。

**基础创建：**

```python
import numpy as np
from flowio import create_fcs

# 创建事件数据 (行=事件, 列=通道)
events = np.random.rand(10000, 5) * 1000

# 定义通道名称
channel_names = ['FSC-A', 'SSC-A', 'FL1-A', 'FL2-A', 'Time']

# 创建 FCS 文件
create_fcs('output.fcs', events, channel_names)
```

**添加描述性通道名称：**

```python
# 添加可选描述性名称 (PnS)
channel_names = ['FSC-A', 'SSC-A', 'FL1-A', 'FL2-A', 'Time']
descriptive_names = ['Forward Scatter', 'Side Scatter', 'FITC', 'PE', 'Time']

create_fcs('output.fcs',
           events,
           channel_names,
           opt_channel_names=descriptive_names)
```

**添加自定义元数据：**

```python
# 添加 TEXT 段元数据
metadata = {
    '$SRC': 'Python 脚本',
    '$DATE': '19-OCT-2025',
    '$CYT': '合成仪器',
    '$INST': '实验室 A'
}

create_fcs('output.fcs',
           events,
           channel_names,
           opt_channel_names=descriptive_names,
           metadata=metadata)
```

**注意：** FlowIO 以 FCS 3.1 格式导出单精度浮点数据。

### 导出修改后的数据

修改现有 FCS 文件并重新导出。

**方法 1：使用 write_fcs() 方法：**

```python
from flowio import FlowData

# 读取原始文件
flow = FlowData('original.fcs')

# 使用更新后的元数据写入
flow.write_fcs('modified.fcs', metadata={'$SRC': '修改后数据'})
```

**方法 2：提取、修改并重建：**

用于修改事件数据：

```python
from flowio import FlowData, create_fcs

# 读取并提取数据
flow = FlowData('original.fcs')
events = flow.as_array(preprocess=False)

# 修改事件数据
events[:, 0] = events[:, 0] * 1.5  # 缩放第一通道

# 使用修改后数据创建新 FCS 文件
create_fcs('modified.fcs',
           events,
           flow.pnn_labels,
           opt_channel_names=flow.pns_labels,
           metadata=flow.text)
```

### 处理多数据集 FCS 文件

某些 FCS 文件在单个文件中包含多个数据集。

**检测多数据集文件：**

```python
from flowio import FlowData, MultipleDataSetsError

try:
    flow = FlowData('sample.fcs')
except MultipleDataSetsError:
    print("文件包含多个数据集")
    # 改用 read_multiple_data_sets()
```

**读取所有数据集：**

```python
from flowio import read_multiple_data_sets

# 从文件读取所有数据集
datasets = read_multiple_data_sets('multi_dataset.fcs')

print(f"发现 {len(datasets)} 个数据集")

# 处理每个数据集
for i, dataset in enumerate(datasets):
    print(f"\n数据集 {i}:")
    print(f"  事件数: {dataset.event_count}")
    print(f"  通道: {dataset.pnn_labels}")

    # 获取该数据集的事件数据
    events = dataset.as_array()
    print(f"  形状: {events.shape}")
    print(f"  均值: {events.mean(axis=0)}")
```

**读取特定数据集：**

```python
from flowio import FlowData

# 读取第一个数据集 (nextdata_offset=0)
first_dataset = FlowData('multi.fcs', nextdata_offset=0)

# 使用第一个数据集的 NEXTDATA 偏移量读取第二个数据集
next_offset = int(first_dataset.text['$NEXTDATA'])
if next_offset > 0:
    second_dataset = FlowData('multi.fcs', nextdata_offset=next_offset)
```

## 数据预处理

当 `preprocess=True` 时，FlowIO 应用标准 FCS 预处理转换。

**预处理步骤：**

1. **增益缩放：** 乘以 PnG（增益）关键词
2. **对数转换：** 若存在 PnE 指数转换则应用
   - 公式：`value = a * 10^(b * raw_value)`，其中 PnE = "a,b"
3. **时间缩放：** 转换时间值为适当单位

**控制预处理：**

```python
# 预处理数据 (默认)
preprocessed = flow.as_array(preprocess=True)

# 原始数据 (无转换)
raw = flow.as_array(preprocess=False)
```

## 错误处理

适当处理常见 FlowIO 异常。

```python
from flowio import (
    FlowData,
    FCSParsingError,
    DataOffsetDiscrepancyError,
    MultipleDataSetsError
)

try:
    flow = FlowData('sample.fcs')
    events = flow.as_array()

except FCSParsingError as e:
    print(f"解析 FCS 文件失败: {e}")
    # 尝试宽松解析
    flow = FlowData('sample.fcs', ignore_offset_error=True)

except DataOffsetDiscrepancyError as e:
    print(f"检测到偏移量差异: {e}")
    # 使用 ignore_offset_discrepancy 参数
    flow = FlowData('sample.fcs', ignore_offset_discrepancy=True)

except MultipleDataSetsError as e:
    print(f"检测到多数据集: {e}")
    # 改用 read_multiple_data_sets
    from flowio import read_multiple_data_sets
    datasets = read_multiple_data_sets('sample.fcs')

except Exception as e:
    print(f"意外错误: {e}")
```

## 常见用例

### 检查 FCS 文件内容

快速探索 FCS 文件结构：

```python
from flowio import FlowData

flow = FlowData('unknown.fcs')

print("=" * 50)
print(f"文件: {flow.name}")
print(f"版本: {flow.version}")
print(f"大小: {flow.file_size:,} 字节")
print("=" * 50)

print(f"\n事件数: {flow.event_count:,}")
print(f"通道数: {flow.channel_count}")

print("\n通道信息:")
for i, (pnn, pns) in enumerate(zip(flow.pnn_labels, flow.pns_labels)):
    ch_type = "散射" if i in flow.scatter_indices else \
              "荧光" if i in flow.fluoro_indices else \
              "时间" if i == flow.time_index else "其他"
    print(f"  [{i}] {pnn:10s} | {pns:30s} | {ch_type}")

print("\n关键元数据:")
for key in ['$DATE', '$BTIM', '$ETIM', '$CYT', '$INST', '$SRC']:
    value = flow.text.get(key, 'N/A')
    print(f"  {key:15s}: {value}")
```

### 批量处理多个文件

处理目录中的 FCS 文件：

```python
from pathlib import Path
from flowio import FlowData
import pandas as pd

# 查找所有 FCS 文件
fcs_files = list(Path('data/').glob('*.fcs'))

# 提取摘要信息
summaries = []
for fcs_path in fcs_files:
    try:
        flow = FlowData(str(fcs_path), only_text=True)
        summaries.append({
            'filename': fcs_path.name,
            'version': flow.version,
            'events': flow.event_count,
            'channels': flow.channel_count,
            'date': flow.text.get('$DATE', 'N/A')
        })
    except Exception as e:
        print(f"处理 {fcs_path.name} 时出错: {e}")

# 创建摘要 DataFrame
df = pd.DataFrame(summaries)
print(df)
```

### 将 FCS 转换为 CSV

将事件数据导出为 CSV 格式：

```python
from flowio import FlowData
import pandas as pd

# 读取 FCS 文件
flow = FlowData('sample.fcs')

# 转换为 DataFrame
df = pd.DataFrame(
    flow.as_array(),
    columns=flow.pnn_labels
)

# 添加元数据作为属性
df.attrs['fcs_version'] = flow.version
df.attrs['instrument'] = flow.text.get('$CYT', 'Unknown')

# 导出为 CSV
df.to_csv('output.csv', index=False)
print(f"已导出 {len(df)} 个事件到 CSV")
```

### 过滤事件并重新导出

应用过滤器并保存过滤后数据：

```python
from flowio import FlowData, create_fcs
import numpy as np

# 读取原始文件
flow = FlowData('sample.fcs')
events = flow.as_array(preprocess=False)

# 应用过滤 (示例: 第一通道阈值)
fsc_idx = 0
threshold = 500
mask = events[:, fsc_idx] > threshold
filtered_events = events[mask]

print(f"原始事件数: {len(events)}")
print(f"过滤后事件数: {len(filtered_events)}")

# 使用过滤后数据创建新 FCS 文件
create_fcs('filtered.fcs',
           filtered_events,
           flow.pnn_labels,
           opt_channel_names=flow.pns_labels,
           metadata={**flow.text, '$SRC': '过滤后数据'})
```

### 提取特定通道

提取并处理特定通道：

```python
from flowio import FlowData
import numpy as np

flow = FlowData('sample.fcs')
events = flow.as_array()

# 仅提取荧光通道
fluoro_indices = flow.fluoro_indices
fluoro_data = events[:, fluoro_indices]
fluoro_names = [flow.pnn_labels[i] for i in fluoro_indices]

print(f"荧光通道: {fluoro_names}")
print(f"形状: {fluoro_data.shape}")

# 计算各通道统计量
for i, name in enumerate(fluoro_names):
    channel_data = fluoro_data[:, i]
    print(f"\n{name}:")
    print(f"  均值: {channel_data.mean():.2f}")
    print(f"  中位数: {np.median(channel_data):.2f}")
    print(f"  标准差: {channel_data.std():.2f}")
```

## 最佳实践

1. **内存效率：** 无需事件数据时使用 `only_text=True`
2. **错误处理：** 用 try-except 块包装文件操作以保证代码健壮性
3. **多数据集检测：** 检查 MultipleDataSetsError 并使用对应函数
4. **预处理控制：** 根据分析需求显式设置 `preprocess` 参数
5. **偏移量问题：** 若解析失败

- `flow.text` - TEXT 段关键词
- `flow.events` - DATA 段（字节形式）
- `flow.analysis` - ANALYSIS 段关键词（若存在）

### 详细 API 参考

有关包含所有参数、方法、异常和 FCS 关键词参考的完整 API 文档，请查阅详细参考文件：

**阅读：** `references/api_reference.md`

该参考包含：
- 完整的 FlowData 类文档
- 所有实用函数（read_multiple_data_sets, create_fcs）
- 异常类与处理方式
- FCS 文件结构详情
- 常见 TEXT 段关键词
- 扩展示例工作流

处理复杂 FCS 操作或遇到非常规文件格式时，请加载此参考获取详细指导。

## 集成说明

**NumPy 数组：** 所有事件数据均以形状为（事件数, 通道数）的 NumPy ndarrays 形式返回

**Pandas 数据帧：** 可轻松转换为数据帧进行分析：
```python
import pandas as pd
df = pd.DataFrame(flow.as_array(), columns=flow.pnn_labels)
```

**FlowKit 集成：** 进行高级分析（补偿、设门、FlowJo 支持）时，请使用基于 FlowIO 解析能力构建的 FlowKit 库

**Web 应用程序：** FlowIO 的极简依赖使其成为处理 FCS 上传的 Web 后端服务的理想选择

## 故障排除

**问题：** "偏移量差异错误"  
**解决方案：** 使用 `ignore_offset_discrepancy=True` 参数

**问题：** "多数据集错误"  
**解决方案：** 使用 `read_multiple_data_sets()` 函数替代 FlowData 构造函数

**问题：** 大文件导致内存不足  
**解决方案：** 元数据操作使用 `only_text=True`，或分块处理事件数据

**问题：** 通道数量异常  
**解决方案：** 检查空通道；使用 `null_channel_list` 参数排除

**问题：** 无法原地修改事件数据  
**解决方案：** FlowIO 不支持直接修改；需提取数据→修改→使用 `create_fcs()` 保存

## 总结

FlowIO 为流式细胞术工作流程提供核心 FCS 文件处理能力，适用于解析、元数据提取和文件创建。简单文件操作和数据提取使用 FlowIO 即可满足需求。涉及补偿和设门等复杂分析时，请集成 FlowKit 或其他专业工具。
