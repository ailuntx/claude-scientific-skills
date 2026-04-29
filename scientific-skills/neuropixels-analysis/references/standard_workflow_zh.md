# 标准 Neuropixels 分析工作流

从原始数据到筛选单元的完整 Neuropixels 记录分析逐步指南。

## 概述

本文档记录完整的分析流程：

```
原始记录 → 预处理 → 运动校正 → 尖峰排序 →
后处理 → 质量指标 → 筛选 → 导出
```

## 1. 数据加载

### 支持格式

```python
import spikeinterface.full as si
import neuropixels_analysis as npa

# SpikeGLX (最常用)
recording = si.read_spikeglx('/path/to/run/', stream_id='imec0.ap')

# Open Ephys
recording = si.read_openephys('/path/to/experiment/')

# NWB 格式
recording = si.read_nwb('/path/to/file.nwb')

# 或使用便捷封装
recording = npa.load_recording('/path/to/data/', format='spikeglx')
```

### 验证记录属性

```python
# 基本属性
print(f"通道数: {recording.get_num_channels()}")
print(f"时长: {recording.get_total_duration():.1f}s")
print(f"采样率: {recording.get_sampling_frequency()}Hz")

# 探针几何结构
print(f"探针型号: {recording.get_probe().name}")

# 通道位置
locations = recording.get_channel_locations()
```

## 2. 预处理

### 标准预处理流程

```python
# 选项1：完整流程（推荐）
rec_preprocessed = npa.preprocess(recording)

# 选项2：分步控制
rec = si.bandpass_filter(recording, freq_min=300, freq_max=6000)
rec = si.phase_shift(rec)  # 校正ADC相位
bad_channels = si.detect_bad_channels(rec)
rec = rec.remove_channels(bad_channels)
rec = si.common_reference(rec, operator='median')
rec_preprocessed = rec
```

### IBL 风格去条纹处理

针对强伪影记录：

```python
from ibldsp.voltage import decompress_destripe_cbin

# IBL去条纹（效果显著）
rec = si.highpass_filter(recording, freq_min=400)
rec = si.phase_shift(rec)
rec = si.highpass_spatial_filter(rec)  # 去条纹
rec = si.common_reference(rec, reference='global', operator='median')
```

### 保存预处理数据

```python
# 保存以便复用（加速迭代）
rec_preprocessed.save(folder='preprocessed/', n_jobs=4)
```

## 3. 运动/漂移校正

### 检查校正需求

```python
# 估计运动
motion_info = npa.estimate_motion(rec_preprocessed, preset='kilosort_like')

# 可视化漂移
npa.plot_drift(rec_preprocessed, motion_info, output='drift_map.png')

# 检查幅度
if motion_info['motion'].max() > 10:  # 微米
    print("检测到显著漂移 - 建议校正")
```

### 应用校正

```python
# 基于DREDge的校正（默认）
rec_corrected = npa.correct_motion(
    rec_preprocessed,
    preset='nonrigid_accurate',  # 或'kilosort_like'加速处理
)

# 或完全手动控制
from spikeinterface.preprocessing import correct_motion

rec_corrected = correct_motion(
    rec_preprocessed,
    preset='nonrigid_accurate',
    folder='motion_output/',
    output_motion=True,
)
```

## 4. 尖峰排序

### 推荐：Kilosort4

```python
# 运行Kilosort4（需GPU）
sorting = npa.run_sorting(
    rec_corrected,
    sorter='kilosort4',
    output_folder='sorting_KS4/',
)

# 自定义参数
sorting = npa.run_sorting(
    rec_corrected,
    sorter='kilosort4',
    output_folder='sorting_KS4/',
    sorter_params={
        'batch_size': 30000,
        'nblocks': 5,  # 非刚性漂移校正
        'Th_learned': 8,  # 检测阈值
    },
)
```

### 备选排序器

```python
# SpykingCircus2（基于CPU）
sorting = npa.run_sorting(rec_corrected, sorter='spykingcircus2')

# Mountainsort5（快速，适合短时记录）
sorting = npa.run_sorting(rec_corrected, sorter='mountainsort5')
```

### 多排序器比较

```python
# 运行多个排序器
sortings = {}
for sorter in ['kilosort4', 'spykingcircus2']:
    sortings[sorter] = npa.run_sorting(rec_corrected, sorter=sorter)

# 比较结果
comparison = npa.compare_sorters(list(sortings.values()))
agreement_matrix = comparison.get_agreement_matrix()
```

## 5. 后处理

### 创建分析器

```python
# 创建排序分析器（所有后处理的核心对象）
analyzer = npa.create_analyzer(
    sorting,
    rec_corrected,
    output_folder='analyzer/',
)

# 计算所有标准扩展
analyzer = npa.postprocess(
    sorting,
    rec_corrected,
    output_folder='analyzer/',
    compute_all=True,  # 波形、模板、指标等
)
```

### 计算独立扩展

```python
# 波形
analyzer.compute('waveforms', ms_before=1.0, ms_after=2.0, max_spikes_per_unit=500)

# 模板
analyzer.compute('templates', operators=['average', 'std'])

# 尖峰幅度
analyzer.compute('spike_amplitudes')

# 自相关图
analyzer.compute('correlograms', window_ms=50.0, bin_ms=1.0)

# 单元位置
analyzer.compute('unit_locations', method='monopolar_triangulation')

# 尖峰位置
analyzer.compute('spike_locations', method='center_of_mass')
```

## 6. 质量指标

### 计算全部指标

```python
# 计算综合指标
metrics = npa.compute_quality_metrics(
    analyzer,
    metric_names=[
        'snr',
        'isi_violations_ratio',
        'presence_ratio',
        'amplitude_cutoff',
        'firing_rate',
        'amplitude_cv',
        'sliding_rp_violation',
        'd_prime',
        'nearest_neighbor',
    ],
)

# 查看指标
print(metrics.head())
```

### 关键指标说明

| 指标 | 理想值 | 描述 |
|--------|------------|-------------|
| `snr` | > 5 | 信噪比 |
| `isi_violations_ratio` | < 0.01 | 不应期违规率 |
| `presence_ratio` | > 0.9 | 存在尖峰的时间占比 |
| `amplitude_cutoff` | < 0.1 | 估计遗漏尖峰比例 |
| `firing_rate` | > 0.1 Hz | 平均放电频率 |

## 7. 单元筛选

### 自动筛选

```python
# Allen研究所标准
labels = npa.curate(metrics, method='allen')

# IBL标准
labels = npa.curate(metrics, method='ibl')

# 自定义阈值
labels = npa.curate(
    metrics,
    snr_threshold=5,
    isi_violations_threshold=0.01,
    presence_threshold=0.9,
)
```

### AI辅助筛选

```python
from anthropic import Anthropic

# 设置API
client = Anthropic()

# 对不确定单元进行可视化分析
uncertain = metrics.query('snr > 3 and snr < 8').index.tolist()

for unit_id in uncertain:
    result = npa.analyze_unit_visually(analyzer, unit_id, api_client=client)
    labels[unit_id] = result['classification']
```

### 交互式筛选会话

```python
# 创建会话
session = npa.CurationSession.create(analyzer, output_dir='curation/')

# 审查单元
while session.current_unit():
    unit = session.current_unit()
    report = npa.generate_unit_report(analyzer, unit.unit_id)

    # 人工决策
    decision = input(f"单元 {unit.unit_id}: ")
    session.set_decision(unit.unit_id, decision)
    session.next_unit()

# 导出
labels = session.get_final_labels()
```

## 8. 结果导出

### 导出至Phy

```python
from spikeinterface.exporters import export_to_phy

export_to_phy(
    analyzer,
    output_folder='phy_export/',
    copy_binary=True,
)
```

### 导出至NWB

```python
from spikeinterface.exporters import export_to_nwb

export_to_nwb(
    analyzer,
    nwbfile_path='results.nwb',
    metadata={
        'session_description': 'Neuropixels记录',
        'experimenter': '实验室名称',
    },
)
```

### 保存质量摘要

```python
# 保存指标CSV
metrics.to_csv('quality_metrics.csv')

# 保存标签
import json
with open('curation_labels.json', 'w') as f:
    json.dump(labels, f, indent=2)

# 生成摘要报告
npa.plot_quality_metrics(analyzer, metrics, output='quality_summary.png')
```

## 完整流程示例

```python
import neuropixels_analysis as npa

# 加载
recording = npa.load_recording('/data/experiment/', format='spikeglx')

# 预处理
rec = npa.preprocess(recording)

# 运动校正
rec = npa.correct_motion(rec)

# 排序
sorting = npa.run_sorting(rec, sorter='kilosort4')

# 后处理
analyzer, metrics = npa.postprocess(sorting, rec)

# 筛选
labels = npa.curate(metrics, method='allen')

# 导出优质单元
good_units = [uid for uid, label in labels.items() if label == 'good']
print(f"优质单元: {len(good_units)}/{len(labels)}")
```

## 成功要点

1. **始终可视化漂移**后再决定运动校正
2. **保存预处理数据**避免重复计算
3. **关键实验需比较多种排序器**
4. **人工复核不确定单元** - 勿盲目信任自动筛选
5. **记录参数**确保可复现性
6. **使用GPU运行Kilosort4**（比CPU方案快10-50倍）
