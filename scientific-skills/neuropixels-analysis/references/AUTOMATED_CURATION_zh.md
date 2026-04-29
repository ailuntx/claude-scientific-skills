# 自动化筛选参考指南

使用 Bombcell、UnitRefine 等工具进行自动化尖峰分选筛选的指南。

## 为何需要自动化筛选？

手动筛选存在以下问题：
- **耗时**：每个记录会话需要数小时
- **主观性强**：评估者间存在差异
- **不可复现**：难以标准化

自动化工具提供一致且可复现的质量分类。

## 可用工具

| 工具 | 分类方式 | 语言 | 集成性 |
|------|---------------|----------|-------------|
| **Bombcell** | 4类（单细胞/多细胞/噪声/非胞体） | Python/MATLAB | SpikeInterface, Phy |
| **UnitRefine** | 基于机器学习 | Python | SpikeInterface |
| **SpikeInterface QM** | 基于阈值 | Python | 原生支持 |
| **UnitMatch** | 跨会话追踪 | Python/MATLAB | Kilosort, Bombcell |

## Bombcell

### 概述

Bombcell 将单元分为4类：
1. **单胞体单元** - 良好分离的单个神经元
2. **多单元活动(MUA)** - 混合神经元信号
3. **噪声** - 非神经伪迹
4. **非胞体** - 轴突或树突信号

### 安装

```bash
# Python
pip install bombcell

# 或安装开发版
git clone https://github.com/Julie-Fabre/bombcell.git
cd bombcell/py_bombcell
pip install -e .
```

### 基础用法(Python)

```python
import bombcell as bc

# 加载分选数据(Kilosort输出)
kilosort_folder = '/path/to/kilosort/output'
raw_data_path = '/path/to/recording.ap.bin'

# 运行Bombcell
results = bc.run_bombcell(
    kilosort_folder,
    raw_data_path,
    sample_rate=30000,
    n_channels=384
)

# 获取分类结果
unit_labels = results['unit_labels']
# 'good'=单单元, 'mua'=多单元, 'noise'=噪声
```

### 与SpikeInterface集成

```python
import spikeinterface.full as si

# 尖峰分选后
sorting = si.run_sorter('kilosort4', recording, output_folder='ks4/')

# 创建分析器并计算必要扩展
analyzer = si.create_sorting_analyzer(sorting, recording, sparse=True)
analyzer.compute('waveforms')
analyzer.compute('templates')
analyzer.compute('spike_amplitudes')

# 导出为Phy格式(Bombcell可读取)
si.export_to_phy(analyzer, output_folder='phy_export/')

# 在Phy导出上运行Bombcell
import bombcell as bc
results = bc.run_bombcell_phy('phy_export/')
```

### Bombcell指标

Bombcell计算特定分类指标：

| 指标 | 描述 | 用途 |
|--------|-------------|----------|
| `peak_trough_ratio` | 波形形态 | 胞体 vs 非胞体 |
| `spatial_decay` | 跨通道振幅衰减 | 噪声检测 |
| `refractory_period_violations` | 不应期违例 | 单单元 vs 多单元 |
| `presence_ratio` | 时间稳定性 | 单元质量 |
| `waveform_duration` | 峰谷时间差 | 细胞类型 |

### 自定义阈值

```python
# 自定义分类阈值
custom_params = {
    'isi_threshold': 0.01,          # ISI违例阈值
    'presence_threshold': 0.9,       # 最低存在率
    'amplitude_threshold': 20,       # 最小振幅(μV)
    'spatial_decay_threshold': 40,   # 空间衰减(μm)
}

results = bc.run_bombcell(
    kilosort_folder,
    raw_data_path,
    **custom_params
)
```

## SpikeInterface自动筛选

### 基于阈值的筛选

```python
# 计算质量指标
analyzer.compute('quality_metrics')
qm = analyzer.get_extension('quality_metrics').get_data()

# 定义筛选函数
def auto_curate(qm):
    labels = {}
    for unit_id in qm.index:
        row = qm.loc[unit_id]

        # 分类逻辑
        if row['snr'] < 2 or row['presence_ratio'] < 0.5:
            labels[unit_id] = 'noise'
        elif row['isi_violations_ratio'] > 0.1:
            labels[unit_id] = 'mua'
        elif (row['snr'] > 5 and
              row['isi_violations_ratio'] < 0.01 and
              row['presence_ratio'] > 0.9):
            labels[unit_id] = 'good'
        else:
            labels[unit_id] = 'unsorted'

    return labels

unit_labels = auto_curate(qm)

# 按标签过滤
good_unit_ids = [u for u, l in unit_labels.items() if l == 'good']
sorting_curated = sorting.select_units(good_unit_ids)
```

### 使用SpikeInterface筛选模块

```python
from spikeinterface.curation import (
    CurationSorting,
    MergeUnitsSorting,
    SplitUnitSorting
)

# 包装分选结果用于筛选
curation = CurationSorting(sorting)

# 移除噪声单元
noise_units = qm[qm['snr'] < 2].index.tolist()
curation.remove_units(noise_units)

# 合并相似单元(基于模板相似性)
analyzer.compute('template_similarity')
similarity = analyzer.get_extension('template_similarity').get_data()

# 寻找高度相似对
import numpy as np
threshold = 0.9
similar_pairs = np.argwhere(similarity > threshold)
# 合并配对(需谨慎 - 需人工复核)

# 获取筛选后分选
sorting_curated = curation.to_sorting()
```

## UnitMatch：跨会话追踪

跨记录日追踪相同神经元。

### 安装

```bash
pip install unitmatch
# 或从源码安装
git clone https://github.com/EnnyvanBeest/UnitMatch.git
```

### 用法

```python
# 在多个会话运行Bombcell后
session_folders = [
    '/path/to/session1/kilosort/',
    '/path/to/session2/kilosort/',
    '/path/to/session3/kilosort/',
]

from unitmatch import UnitMatch

# 运行UnitMatch
um = UnitMatch(session_folders)
um.run()

# 获取匹配结果
matches = um.get_matches()
# 返回跨会话匹配的单元ID数据框

# 分配唯一ID
unique_ids = um.get_unique_ids()
```

### 工作流集成

```python
# 典型工作流：
# 1. 分选每个会话
# 2. 运行Bombcell进行质控
# 3. 运行UnitMatch进行跨会话追踪

# 会话1
sorting1 = si.run_sorter('kilosort4', rec1, output_folder='session1/ks4/')
# 运行Bombcell
labels1 = bc.run_bombcell('session1/ks4/', raw1_path)

# 会话2
sorting2 = si.run_sorter('kilosort4', rec2, output_folder='session2/ks4/')
labels2 = bc.run_bombcell('session2/ks4/', raw2_path)

# 跨会话追踪单元
um = UnitMatch(['session1/ks4/', 'session2/ks4/'])
matches = um.get_matches()
```

## 半自动化工作流

结合自动化与手动筛选：

```python
# 步骤1：自动分类
analyzer.compute('quality_metrics')
qm = analyzer.get_extension('quality_metrics').get_data()

# 自动标记明确案例
auto_labels = {}
for unit_id in qm.index:
    row = qm.loc[unit_id]
    if row['snr'] < 1.5:
        auto_labels[unit_id] = 'noise'
    elif row['snr'] > 8 and row['isi_violations_ratio'] < 0.005:
        auto_labels[unit_id] = 'good'
    else:
        auto_labels[unit_id] = '待复核'

# 步骤2：导出不确定单元供人工复核
needs_review = [u for u, l in auto_labels.items() if l == '待复核']

# 仅导出不确定单元到Phy
sorting_review = sorting.select_units(needs_review)
analyzer_review = si.create_sorting_analyzer(sorting_review, recording)
analyzer_review.compute('waveforms')
analyzer_review.compute('templates')
si.export_to_phy(analyzer_review, output_folder='phy_review/')

# 在Phy中人工复核: phy template-gui phy_review/params.py

# 步骤3：加载人工标签并合并
manual_labels = si.read_phy('phy_review/').get_property('quality')
# 合并自动+人工标签获得最终结果
```

## 方法对比

| 方法 | 优点 | 缺点 |
|--------|------|------|
| **手动(Phy)** | 黄金标准，灵活 | 耗时，主观性强 |
| **SpikeInterface QM** | 快速，可复现 | 仅支持简单阈值 |
| **Bombcell** | 多类别，经验证 | 需提取波形 |
| **UnitRefine** | 基于机器学习，可学习 | 需要训练数据 |

## 最佳实践

1. **始终可视化** - 勿盲目信任自动化结果
2. **记录阈值** - 保存使用的精确参数
3. **验证** - 在子集上比较自动与手动结果
4. **保守处理** - 存疑时排除单元
5. **报告方法** - 在出版物中包含筛选标准

## 流程示例

```python
def curate_sorting(sorting, recording, output_dir):
    """完整筛选流程"""

    # 创建分析器
    analyzer = si.create_sorting_analyzer(sorting, recording, sparse=True,
                                          folder=f'{output_dir}/analyzer')

    # 计算必要扩展
    analyzer.compute('random_spikes', max_spikes_per_unit=500)
    analyzer.compute('waveforms')
    analyzer.compute('templates')
    analyzer.compute('noise_levels')
    analyzer.compute('spike_amplitudes')
    analyzer.compute('quality_metrics')

    qm = analyzer.get_extension('quality_metrics').get_data()

    # 自动分类
    labels = {}
    for unit_id in qm.index:
        row = qm.loc[unit_id]

        if row['snr'] < 2:
            labels[unit_id] = 'noise'
        elif row['isi_violations_ratio'] > 0.1 or row['presence_ratio'] < 0.8:
            labels[unit_id] = 'mua'
        elif (row['snr'] > 5 and
              row['isi_violations_ratio'] < 0.01 and
              row['presence_ratio'] > 0.9 and
              row['amplitude_cutoff'] < 0.1):
            labels[unit_id] = 'good'
        else:
            labels[unit_id] = 'unsorted'

    # 结果统计
    from collections import Counter
    print("分类统计:")
    print(Counter(labels.values()))

    # 保存标签
    import json
    with open(f'{output_dir}/unit_labels.json', 'w') as f:
        json.dump(labels, f)

    # 返回优质单元
    good_ids = [u for u, l in labels.items() if l == 'good']
    return sorting.select_units(good_ids), labels

# 使用示例
sorting_curated, labels = curate_sorting(sorting, recording, 'output/')
```

## 参考文献

- [Bombcell GitHub](https://github.com/Julie-Fabre/bombcell)
- [UnitMatch GitHub](https://github.com/EnnyvanBeest/UnitMatch)
- [SpikeInterface筛选文档](https://spikeinterface.readthedocs.io/en/stable/modules/curation.html)
- Fabre et al. (2023) "Bombcell: automated curation and cell classification"
- van Beest et al. (2024) "UnitMatch: tracking neurons across days with high-density probes"
