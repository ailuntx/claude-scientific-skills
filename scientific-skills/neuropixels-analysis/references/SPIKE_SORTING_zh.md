# 尖峰排序参考指南

Neuropixels 数据尖峰排序综合指南。

## 可用排序工具

| 排序工具 | 需GPU | 速度 | 质量 | 最佳适用场景 |
|--------|--------------|-------|---------|----------|
| **Kilosort4** | 是 (CUDA) | 快 | 极佳 | 生产环境 |
| **Kilosort3** | 是 (CUDA) | 快 | 优秀 | 旧版兼容 |
| **Kilosort2.5** | 是 (CUDA) | 快 | 良好 | 旧版流程 |
| **SpykingCircus2** | 否 | 中等 | 良好 | 纯CPU系统 |
| **Mountainsort5** | 否 | 中等 | 良好 | 小型记录 |
| **Tridesclous2** | 否 | 中等 | 良好 | 交互式排序 |

## Kilosort4 (推荐)

### 安装
```bash
pip install kilosort
```

### 基础用法
```python
import spikeinterface.full as si

# 运行Kilosort4
sorting = si.run_sorter(
    'kilosort4',
    recording,
    output_folder='ks4_output',
    verbose=True
)

print(f"检测到 {len(sorting.unit_ids)} 个单元")
```

### 自定义参数
```python
sorting = si.run_sorter(
    'kilosort4',
    recording,
    output_folder='ks4_output',
    # 检测参数
    Th_universal=9,        # 尖峰检测阈值
    Th_learned=8,          # 学习阈值
    # 模板参数
    dmin=15,               # 模板最小垂直间距(微米)
    dminx=12,              # 模板最小水平间距(微米)
    nblocks=5,             # 非刚性区块数量
    # 聚类参数
    max_channel_distance=None,  # 模板通道最大距离
    # 输出参数
    do_CAR=False,          # 跳过CAR(已在预处理完成)
    skip_kilosort_preprocessing=True,
    save_extra_kwargs=True
)
```

### Kilosort4 完整参数
```python
# 获取所有可用参数
params = si.get_default_sorter_params('kilosort4')
print(params)

# 关键参数:
ks4_params = {
    # 检测参数
    'Th_universal': 9,      # 尖峰检测通用阈值
    'Th_learned': 8,        # 学习模板阈值
    'spkTh': -6,            # 提取阶段尖峰阈值

    # 聚类参数
    'dmin': 15,             # 聚类最小间距(微米)
    'dminx': 12,            # 最小水平间距(微米)
    'nblocks': 5,           # 非刚性漂移校正区块数

    # 模板参数
    'n_templates': 6,       # 每组通用模板数量
    'nt': 61,               # 模板时间采样点数

    # 性能参数
    'batch_size': 60000,    # 样本批处理量
    'nfilt_factor': 8,      # 滤波器数量因子
}
```

## Kilosort3

### 用法
```python
sorting = si.run_sorter(
    'kilosort3',
    recording,
    output_folder='ks3_output',
    # 关键参数
    detect_threshold=6,
    projection_threshold=[9, 9],
    preclust_threshold=8,
    car=False,  # 预处理已完成CAR
    freq_min=300,
)
```

## SpykingCircus2 (纯CPU)

### 安装
```bash
pip install spykingcircus
```

### 用法
```python
sorting = si.run_sorter(
    'spykingcircus2',
    recording,
    output_folder='sc2_output',
    # 参数
    detect_threshold=5,
    selection_method='all',
)
```

## Mountainsort5 (纯CPU)

### 安装
```bash
pip install mountainsort5
```

### 用法
```python
sorting = si.run_sorter(
    'mountainsort5',
    recording,
    output_folder='ms5_output',
    # 参数
    detect_threshold=5.0,
    scheme='2',  # 可选'1','2','3'
)
```

## 运行多个排序工具

### 比较排序结果
```python
# 运行多个排序工具
sorting_ks4 = si.run_sorter('kilosort4', recording, output_folder='ks4/')
sorting_sc2 = si.run_sorter('spykingcircus2', recording, output_folder='sc2/')
sorting_ms5 = si.run_sorter('mountainsort5', recording, output_folder='ms5/')

# 比较结果
comparison = si.compare_multiple_sorters(
    [sorting_ks4, sorting_sc2, sorting_ms5],
    name_list=['KS4', 'SC2', 'MS5']
)

# 获取一致性评分
agreement = comparison.get_agreement_sorting()
```

### 集成排序
```python
# 创建共识排序
sorting_ensemble = si.create_ensemble_sorting(
    [sorting_ks4, sorting_sc2, sorting_ms5],
    voting_method='agreement',
    min_agreement=2  # 至少被两个工具检测到的单元
)
```

## Docker/Singularity环境排序

### Docker使用
```python
sorting = si.run_sorter(
    'kilosort3',
    recording,
    output_folder='ks3_docker/',
    docker_image='spikeinterface/kilosort3-compiled-base:latest',
    verbose=True
)
```

### Singularity使用
```python
sorting = si.run_sorter(
    'kilosort3',
    recording,
    output_folder='ks3_singularity/',
    singularity_image='/path/to/kilosort3.sif',
    verbose=True
)
```

## 长时程记录策略

### 合并记录数据
```python
# 多个记录文件
recordings = [
    si.read_spikeglx(f'/path/to/recording_{i}', stream_id='imec0.ap')
    for i in range(3)
]

# 合并记录
recording_concat = si.concatenate_recordings(recordings)

# 排序
sorting = si.run_sorter('kilosort4', recording_concat, output_folder='ks4/')

# 按原始记录拆分结果
sortings_split = si.split_sorting(sorting, recording_concat)
```

### 分段排序
```python
# 超长记录分段排序
from pathlib import Path

segments_output = Path('sorting_segments')
sortings = []

for i, segment in enumerate(recording.split_by_times([0, 3600, 7200, 10800])):
    sorting_seg = si.run_sorter(
        'kilosort4',
        segment,
        output_folder=segments_output / f'segment_{i}'
    )
    sortings.append(sorting_seg)
```

## 排序后处理

### Phy手动校正
```python
# 导出至Phy格式
analyzer = si.create_sorting_analyzer(sorting, recording)
analyzer.compute(['random_spikes', 'waveforms', 'templates'])
si.export_to_phy(analyzer, output_folder='phy_export/')

# 启动Phy
# 终端执行: phy template-gui phy_export/params.py
```

### 加载Phy校正结果
```python
# Phy手动校正后
sorting_curated = si.read_phy('phy_export/')

# 或应用Phy标签
sorting_curated = si.apply_phy_curation(sorting, 'phy_export/')
```

### 自动校正
```python
# 过滤低质量单元
analyzer = si.create_sorting_analyzer(sorting, recording)
analyzer.compute('quality_metrics')

qm = analyzer.get_extension('quality_metrics').get_data()

# 定义质量标准
query = "(snr > 5) & (isi_violations_ratio < 0.01) & (presence_ratio > 0.9)"
good_unit_ids = qm.query(query).index.tolist()

sorting_clean = sorting.select_units(good_unit_ids)
print(f"保留 {len(good_unit_ids)}/{len(sorting.unit_ids)} 个单元")
```

## 排序指标分析

### 检查输出结果
```python
# 基础统计
print(f"检测单元数: {len(sorting.unit_ids)}")
print(f"总尖峰数: {sorting.get_total_num_spikes()}")

# 单元尖峰计数
for unit_id in sorting.unit_ids[:10]:
    n_spikes = len(sorting.get_unit_spike_train(unit_id))
    print(f"单元 {unit_id}: {n_spikes} 个尖峰")
```

### 放电频率
```python
# 计算放电频率
duration = recording.get_total_duration()
for unit_id in sorting.unit_ids:
    n_spikes = len(sorting.get_unit_spike_train(unit_id))
    fr = n_spikes / duration
    print(f"单元 {unit_id}: {fr:.2f} Hz")
```

## 故障排除

### 常见问题

**GPU内存不足**
```python
# 减小批处理量
sorting = si.run_sorter(
    'kilosort4',
    recording,
    output_folder='ks4/',
    batch_size=30000  # 较小批次
)
```

**检测单元过少**
```python
# 降低检测阈值
sorting = si.run_sorter(
    'kilosort4',
    recording,
    output_folder='ks4/',
    Th_universal=7,  # 默认值9下调
    Th_learned=6
)
```

**单元过多(过分割)**
```python
# 增大模板最小间距
sorting = si.run_sorter(
    'kilosort4',
    recording,
    output_folder='ks4/',
    dmin=20,   # 从15上调
    dminx=16   # 从12上调
)
```

**检查GPU状态**
```python
import torch
print(f"CUDA可用: {torch.cuda.is_available()}")
print(f"GPU型号: {torch.cuda.get_device_name(0)}")
```
