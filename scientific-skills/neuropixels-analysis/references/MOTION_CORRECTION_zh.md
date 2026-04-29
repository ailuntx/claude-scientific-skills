# 运动/漂移校正参考指南

急性探针插入过程中的机械漂移是Neuropixels记录面临的主要挑战。本指南涵盖运动伪影的检测、估计与校正方法。

## 为何运动校正至关重要

- Neuropixels探针在记录过程中可能漂移10-100+微米
- 未校正的漂移会导致：
  - 记录中途单元信号出现/消失
  - 波形幅度变化
  - 错误的尖峰-单元分配
  - 单元产出率降低

## 检测：排序前检查

**运行尖峰排序前务必可视化漂移情况！**

```python
import spikeinterface.full as si
from spikeinterface.sortingcomponents.peak_detection import detect_peaks
from spikeinterface.sortingcomponents.peak_localization import localize_peaks

# 首先预处理（不要白化——会影响峰值定位）
rec = si.highpass_filter(recording, freq_min=400.)
rec = si.common_reference(rec, operator='median', reference='global')

# 检测峰值
noise_levels = si.get_noise_levels(rec, return_in_uV=False)
peaks = detect_peaks(
    rec,
    method='locally_exclusive',
    noise_levels=noise_levels,
    detect_threshold=5,
    radius_um=50.,
    n_jobs=8,
    chunk_duration='1s',
    progress_bar=True
)

# 定位峰值
peak_locations = localize_peaks(
    rec, peaks,
    method='center_of_mass',
    n_jobs=8,
    chunk_duration='1s'
)

# 可视化漂移
si.plot_drift_raster_map(
    peaks=peaks,
    peak_locations=peak_locations,
    recording=rec,
    clim=(-200, 0)  # 调整颜色范围
)
```

### 漂移图解读指南

| 模式 | 含义 | 处理方案 |
|---------|---------------|--------|
| 水平稳定带 | 无显著漂移 | 跳过校正 |
| 对角线带（缓慢） | 渐进沉降漂移 | 使用运动校正 |
| 急剧跳跃 | 脑搏动或位移 | 使用非刚性校正 |
| 混沌模式 | 严重不稳定 | 考虑丢弃该片段 |

## 运动校正方法

### 快速校正（推荐起点）

```python
# 使用预设的单行命令
rec_corrected = si.correct_motion(
    recording=rec,
    preset='nonrigid_fast_and_accurate'
)
```

### 可用预设方案

| 预设 | 速度 | 精度 | 最佳适用场景 |
|--------|-------|----------|----------|
| `rigid_fast` | 快 | 低 | 快速检查，小漂移 |
| `kilosort_like` | 中 | 良好 | 兼容Kilosort的结果 |
| `nonrigid_accurate` | 慢 | 高 | 发表级质量 |
| `nonrigid_fast_and_accurate` | 中 | 高 | **推荐默认方案** |
| `dredge` | 慢 | 最高 | 最佳效果，复杂漂移 |
| `dredge_fast` | 中 | 高 | 计算量更少的DREDge |

### 全流程控制

```python
from spikeinterface.sortingcomponents.motion import (
    estimate_motion,
    interpolate_motion
)

# 步骤1：估计运动
motion, temporal_bins, spatial_bins = estimate_motion(
    rec,
    peaks,
    peak_locations,
    method='decentralized',
    direction='y',
    rigid=False,          # Neuropixels使用非刚性
    win_step_um=50,       # 空间窗口步长
    win_sigma_um=150,     # 空间平滑
    bin_s=2.0,            # 时间分箱大小
    progress_bar=True
)

# 步骤2：可视化运动估计
si.plot_motion(
    motion,
    temporal_bins,
    spatial_bins,
    recording=rec
)

# 步骤3：通过插值应用校正
rec_corrected = interpolate_motion(
    recording=rec,
    motion=motion,
    temporal_bins=temporal_bins,
    spatial_bins=spatial_bins,
    border_mode='force_extrapolate'
)
```

### 保存运动估计

```python
# 保存供后续使用
import numpy as np
np.savez('motion_estimate.npz',
         motion=motion,
         temporal_bins=temporal_bins,
         spatial_bins=spatial_bins)

# 后续加载
data = np.load('motion_estimate.npz')
motion = data['motion']
temporal_bins = data['temporal_bins']
spatial_bins = data['spatial_bins']
```

## DREDge：前沿校正方法

DREDge（电生理数据分布式配准）是当前性能最优的运动校正方法。

### 使用DREDge预设

```python
# AP频段运动估计
rec_corrected = si.correct_motion(rec, preset='dredge')

# 或显式计算
motion, motion_info = si.compute_motion(
    rec,
    preset='dredge',
    output_motion_info=True,
    folder='motion_output/',
    **job_kwargs
)
```

### 基于LFP的运动估计

适用于快速漂移或AP频段估计失败时：

```python
# 加载LFP数据流
lfp = si.read_spikeglx('/path/to/data', stream_name='imec0.lf')

# 从LFP估计运动（更快，处理快速漂移）
motion_lfp, motion_info = si.compute_motion(
    lfp,
    preset='dredge_lfp',
    output_motion_info=True
)

# 应用于AP记录
rec_corrected = interpolate_motion(
    recording=rec,  # AP记录
    motion=motion_lfp,
    temporal_bins=motion_info['temporal_bins'],
    spatial_bins=motion_info['spatial_bins']
)
```

## 与尖峰排序的集成

### 方案1：预校正（推荐）

```python
# 排序前校正
rec_corrected = si.correct_motion(rec, preset='nonrigid_fast_and_accurate')

# 保存校正后记录
rec_corrected = rec_corrected.save(folder='preprocessed_motion_corrected/',
                                    format='binary', n_jobs=8)

# 在校正数据上运行尖峰排序
sorting = si.run_sorter('kilosort4', rec_corrected, output_folder='ks4/')
```

### 方案2：由Kilosort处理

Kilosort 2.5+内置漂移校正：

```python
sorting = si.run_sorter(
    'kilosort4',
    rec,  # 未运动校正
    output_folder='ks4/',
    nblocks=5,  # 非刚性区块用于漂移校正
    do_correction=True  # 启用Kilosort漂移校正
)
```

### 方案3：事后校正

```python
# 先排序
sorting = si.run_sorter('kilosort4', rec, output_folder='ks4/')

# 从排序尖峰估计运动（更准确，使用实际尖峰时间）
from spikeinterface.sortingcomponents.motion import estimate_motion_from_sorting

motion = estimate_motion_from_sorting(sorting, rec)
```

## 参数详解

### 峰值检测

```python
peaks = detect_peaks(
    rec,
    method='locally_exclusive',  # 密集探针最佳方案
    noise_levels=noise_levels,
    detect_threshold=5,          # 值越低=峰值越多（估计噪声更大）
    radius_um=50.,               # 排除半径
    exclude_sweep_ms=0.1,        # 时间排除窗口
)
```

### 运动估计

```python
motion = estimate_motion(
    rec, peaks, peak_locations,
    method='decentralized',      # 'decentralized' 或 'iterative_template'
    direction='y',               # 沿探针轴向
    rigid=False,                 # 非刚性设为False
    bin_s=2.0,                   # 时间分辨率（秒）
    win_step_um=50,              # 空间窗口步长
    win_sigma_um=150,            # 空间平滑系数
    margin_um=0,                 # 探针边缘余量
    win_scale_um=150,            # 权重计算窗口尺度
)
```

## 故障排除

### 过校正（波浪状模式）

```python
# 增加时间平滑
motion = estimate_motion(..., bin_s=5.0)  # 增大分箱

# 或对小漂移使用刚性校正
motion = estimate_motion(..., rigid=True)
```

### 校正不足（残留漂移）

```python
# 减小空间窗口以获取精细非刚性估计
motion = estimate_motion(..., win_step_um=25, win_sigma_um=75)

# 使用更多峰值
peaks = detect_peaks(..., detect_threshold=4)  # 降低阈值
```

### 边缘伪影

```python
rec_corrected = interpolate_motion(
    rec, motion, temporal_bins, spatial_bins,
    border_mode='force_extrapolate',  # 或 'remove_channels'
    spatial_interpolation_method='kriging'
)
```

## 验证

校正后重新可视化确认：

```python
# 在校正记录上重新检测峰值
peaks_corrected = detect_peaks(rec_corrected, ...)
peak_locations_corrected = localize_peaks(rec_corrected, peaks_corrected, ...)

# 绘制前后对比图
fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 校正前
si.plot_drift_raster_map(peaks, peak_locations, rec, ax=axes[0])
axes[0].set_title('校正前')

# 校正后
si.plot_drift_raster_map(peaks_corrected, peak_locations_corrected,
                         rec_corrected, ax=axes[1])
axes[1].set_title('校正后')
```

## 参考文献

- [SpikeInterface运动校正文档](https://spikeinterface.readthedocs.io/en/stable/modules/motion_correction.html)
- [漂移处理教程](https://spikeinterface.readthedocs.io/en/stable/how_to/handle_drift.html)
- [DREDge GitHub](https://github.com/evarol/DREDge)
- Windolf等 (2023) "DREDge：高密度电生理记录的鲁棒运动校正"
