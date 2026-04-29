# 后处理与分析参考指南

Neuropixels 排序数据的质量指标、可视化和分析综合指南。

## 排序分析器

`SortingAnalyzer` 是后处理的核心对象。

### 创建分析器
```python
import spikeinterface.full as si

# 创建分析器
analyzer = si.create_sorting_analyzer(
    sorting,
    recording,
    sparse=True,                    # 使用稀疏表示
    format='binary_folder',         # 存储格式
    folder='analyzer_output'        # 保存位置
)
```

### 计算扩展
```python
# 计算所有标准扩展
analyzer.compute('random_spikes')       # 随机尖峰选择
analyzer.compute('waveforms')           # 提取波形
analyzer.compute('templates')           # 计算模板
analyzer.compute('noise_levels')        # 噪声估计
analyzer.compute('principal_components')  # PCA
analyzer.compute('spike_amplitudes')    # 每个尖峰的幅度
analyzer.compute('correlograms')        # 自相关/互相关图
analyzer.compute('unit_locations')      # 单元位置
analyzer.compute('spike_locations')     # 每个尖峰的位置
analyzer.compute('template_similarity') # 模板相似度矩阵
analyzer.compute('quality_metrics')     # 质量指标

# 或同时计算多个
analyzer.compute([
    'random_spikes', 'waveforms', 'templates', 'noise_levels',
    'principal_components', 'spike_amplitudes', 'correlograms',
    'unit_locations', 'quality_metrics'
])
```

### 保存与加载
```python
# 保存
analyzer.save_as(folder='analyzer_saved', format='binary_folder')

# 加载
analyzer = si.load_sorting_analyzer('analyzer_saved')
```

## 质量指标

### 计算指标
```python
analyzer.compute('quality_metrics')
qm = analyzer.get_extension('quality_metrics').get_data()
print(qm)
```

### 可用指标

| 指标 | 描述 | 理想值 |
|--------|-------------|-------------|
| `snr` | 信噪比 | > 5 |
| `isi_violations_ratio` | ISI 违规比例 | < 0.01 (1%) |
| `isi_violations_count` | ISI 违规计数 | 低 |
| `presence_ratio` | 存在尖峰的记录比例 | > 0.9 |
| `firing_rate` | 每秒尖峰数 | 0.1-50 Hz |
| `amplitude_cutoff` | 估计遗漏尖峰比例 | < 0.1 |
| `amplitude_median` | 尖峰幅度中位数 | - |
| `amplitude_cv` | 变异系数 | < 0.5 |
| `drift_ptp` | 峰间漂移量 (um) | < 40 |
| `drift_std` | 漂移标准差 | < 10 |
| `drift_mad` | 漂移中位数绝对偏差 | < 10 |
| `sliding_rp_violation` | 滑动不应期违规 | < 0.05 |
| `sync_spike_2` | 与其他单元同步性 | < 0.5 |
| `isolation_distance` | 马氏距离 | > 20 |
| `l_ratio` | L比率 (隔离度) | < 0.1 |
| `d_prime` | 可区分度 | > 5 |
| `nn_hit_rate` | 最近邻命中率 | > 0.9 |
| `nn_miss_rate` | 最近邻漏检率 | < 0.1 |
| `silhouette_score` | 聚类轮廓系数 | > 0.5 |

### 计算特定指标
```python
analyzer.compute(
    'quality_metrics',
    metric_names=['snr', 'isi_violations_ratio', 'presence_ratio', 'firing_rate']
)
```

### 自定义质量阈值
```python
qm = analyzer.get_extension('quality_metrics').get_data()

# 定义质量标准
quality_criteria = {
    'snr': ('>', 5),
    'isi_violations_ratio': ('<', 0.01),
    'presence_ratio': ('>', 0.9),
    'firing_rate': ('>', 0.1),
    'amplitude_cutoff': ('<', 0.1),
}

# 筛选优质单元
good_units = qm.query(
    "(snr > 5) & (isi_violations_ratio < 0.01) & (presence_ratio > 0.9)"
).index.tolist()

print(f"优质单元: {len(good_units)}/{len(qm)}")
```

## 波形与模板

### 提取波形
```python
analyzer.compute('waveforms', ms_before=1.5, ms_after=2.5, max_spikes_per_unit=500)

# 获取单元波形
waveforms = analyzer.get_extension('waveforms').get_waveforms(unit_id=0)
print(f"形状: {waveforms.shape}")  # (尖峰数, 采样点数, 通道数)
```

### 计算模板
```python
analyzer.compute('templates', operators=['average', 'std', 'median'])

# 获取模板
templates_ext = analyzer.get_extension('templates')
template = templates_ext.get_unit_template(unit_id=0, operator='average')
```

### 模板相似度
```python
analyzer.compute('template_similarity')
sim = analyzer.get_extension('template_similarity').get_data()
# 模板间余弦相似度矩阵
```

## 单元定位

### 计算位置
```python
analyzer.compute('unit_locations', method='monopolar_triangulation')
locations = analyzer.get_extension('unit_locations').get_data()
print(locations)  # 每个单元的 x, y 坐标
```

### 尖峰定位
```python
analyzer.compute('spike_locations', method='center_of_mass')
spike_locs = analyzer.get_extension('spike_locations').get_data()
```

### 定位方法
- `'center_of_mass'` - 快速，精度较低
- `'monopolar_triangulation'` - 精度高，速度慢
- `'grid_convolution'` - 良好平衡

## 相关图

### 自相关图
```python
analyzer.compute('correlograms', window_ms=50, bin_ms=1)
correlograms, bins = analyzer.get_extension('correlograms').get_data()

# 相关图形状: (单元数, 单元数, 分箱数)
# 单元 i 的自相关图: correlograms[i, i, :]
# 单元 i,j 的互相关图: correlograms[i, j, :]
```

## 可视化

### 探针布局图
```python
si.plot_probe_map(recording, with_channel_ids=True)
```

### 单元模板
```python
# 所有单元
si.plot_unit_templates(analyzer)

# 特定单元
si.plot_unit_templates(analyzer, unit_ids=[0, 1, 2])
```

### 波形图
```python
# 带模板的波形图
si.plot_unit_waveforms(analyzer, unit_ids=[0])

# 波形密度图
si.plot_unit_waveforms_density_map(analyzer, unit_id=0)
```

### 点阵图
```python
si.plot_rasters(sorting, time_range=(0, 10))  # 前10秒
```

### 幅度图
```python
analyzer.compute('spike_amplitudes')
si.plot_amplitudes(analyzer)

# 分布图
si.plot_all_amplitudes_distributions(analyzer)
```

### 相关图
```python
# 自相关图
si.plot_autocorrelograms(analyzer, unit_ids=[0, 1, 2])

# 互相关图
si.plot_crosscorrelograms(analyzer, unit_ids=[0, 1])
```

### 质量指标
```python
# 概览图
si.plot_quality_metrics(analyzer)

# 特定指标分布
import matplotlib.pyplot as plt
qm = analyzer.get_extension('quality_metrics').get_data()
plt.hist(qm['snr'], bins=50)
plt.xlabel('信噪比')
plt.ylabel('计数')
```

### 探针单元位置
```python
si.plot_unit_locations(analyzer)
```

### 漂移图
```python
si.plot_drift_raster(sorting, recording)
```

### 单元概览图
```python
# 综合单元概览
si.plot_unit_summary(analyzer, unit_id=0)
```

## LFP分析

### 加载LFP数据
```python
lfp = si.read_spikeglx('/path/to/data', stream_id='imec0.lf')
print(f"LFP采样率: {lfp.get_sampling_frequency()} Hz")
```

### 基础LFP处理
```python
# 降采样（如需要）
lfp_ds = si.resample(lfp, resample_rate=1000)

# 公共平均参考
lfp_car = si.common_reference(lfp_ds, reference='global', operator='median')
```

### 提取LFP轨迹
```python
import numpy as np

# 获取轨迹 (通道 x 采样点)
traces = lfp.get_traces(start_frame=0, end_frame=30000)

# 特定通道
traces = lfp.get_traces(channel_ids=[0, 1, 2])
```

### 频谱分析
```python
from scipy import signal
import matplotlib.pyplot as plt

# 获取单通道
trace = lfp.get_traces(channel_ids=[0]).flatten()
fs = lfp.get_sampling_frequency()

# 功率谱
freqs, psd = signal.welch(trace, fs, nperseg=4096)
plt.semilogy(freqs, psd)
plt.xlabel('频率 (Hz)')
plt.ylabel('功率')
plt.xlim(0, 100)
```

### 频谱图
```python
f, t, Sxx = signal.spectrogram(trace, fs, nperseg=2048, noverlap=1024)
plt.pcolormesh(t, f, 10*np.log10(Sxx), shading='gouraud')
plt.ylabel('频率 (Hz)')
plt.xlabel('时间 (秒)')
plt.ylim(0, 100)
plt.colorbar(label='功率 (dB)')
```

## 导出格式

### 导出至Phy
```python
si.export_to_phy(
    analyzer,
    output_folder='phy_export',
    compute_pc_features=True,
    compute_amplitudes=True,
    copy_binary=True
)
# 然后运行: phy template-gui phy_export/params.py
```

### 导出至NWB
```python
from spikeinterface.exporters import export_to_nwb

export_to_nwb(
    recording,
    sorting,
    'output.nwb',
    metadata=dict(
        session_description='Neuropixels记录',
        experimenter='姓名',
        lab='实验室名称',
        institution='机构名称'
    )
)
```

### 导出报告
```python
si.export_report(
    analyzer,
    output_folder='report',
    remove_if_exists=True,
    format='html'
)
```

## 完整分析流程

```python
import spikeinterface.full as si

def analyze_sorting(recording, sorting, output_dir):
    """完整的后处理流程"""

    # 创建分析器
    analyzer = si.create_sorting_analyzer(
        sorting, recording,
        sparse=True,
        folder=f'{output_dir}/analyzer'
    )

    # 计算所有扩展
    print("计算扩展中...")
    analyzer.compute(['random_spikes', 'waveforms', 'templates', 'noise_levels'])
    analyzer.compute(['principal_components', 'spike_amplitudes'])
    analyzer.compute(['correlograms', 'unit_locations', 'template_similarity'])
    analyzer.compute('quality_metrics')

    # 获取质量指标
    qm = analyzer.get_extension('quality_metrics').get_data()

    # 筛选优质单元
    good_units = qm.query(
        "(snr > 5) & (isi_violations_ratio < 0.01) & (presence_ratio > 0.9)"
    ).index.tolist()

    print(f"质量筛选: {len(good_units)}/{len(qm)} 个单元通过")

    # 导出
    si.export_to_phy(analyzer, f'{output_dir}/phy')
    si.export_report(analyzer, f'{output_dir}/report')

    # 保存指标
    qm.to_csv(f'{output_dir}/quality_metrics.csv')

    return analyzer, qm, good_units

# 使用示例
analyzer, qm, good_units = analyze_sorting(recording, sorting, 'output/')
```
