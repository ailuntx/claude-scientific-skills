# Neuropixels 预处理参考指南

Neuropixels 神经信号记录的全面预处理技术。

## 标准预处理流程

```python
import spikeinterface.full as si

# 加载原始数据
recording = si.read_spikeglx('/path/to/data', stream_id='imec0.ap')

# 1. 相位校正（适用于 Neuropixels 1.0）
rec = si.phase_shift(recording)

# 2. 用于尖峰检测的带通滤波
rec = si.bandpass_filter(rec, freq_min=300, freq_max=6000)

# 3. 公共中值参考（消除相关噪声）
rec = si.common_reference(rec, reference='global', operator='median')

# 4. 移除坏通道（可选）
rec = si.remove_bad_channels(rec, bad_channel_ids=bad_channels)
```

## 滤波选项

### 带通滤波
```python
# 标准动作电位频带
rec = si.bandpass_filter(recording, freq_min=300, freq_max=6000)

# 宽频带（保留更多波形特征）
rec = si.bandpass_filter(recording, freq_min=150, freq_max=7500)

# 滤波参数设置
rec = si.bandpass_filter(
    recording,
    freq_min=300,
    freq_max=6000,
    filter_order=5,
    ftype='butter',  # 'butter'、'bessel' 或 'cheby1'
    margin_ms=5.0    # 防止边缘伪影
)
```

### 仅高通滤波
```python
rec = si.highpass_filter(recording, freq_min=300)
```

### 陷波滤波（消除工频噪声）
```python
# 消除60Hz及其谐波
rec = si.notch_filter(recording, freq=60, q=30)
rec = si.notch_filter(rec, freq=120, q=30)
rec = si.notch_filter(rec, freq=180, q=30)
```

## 参考方案

### 公共中值参考（推荐）
```python
# 全局中值参考
rec = si.common_reference(recording, reference='global', operator='median')

# 分片参考（多探针片场景）
rec = si.common_reference(recording, reference='global', operator='median',
                          groups=recording.get_channel_groups())
```

### 公共平均参考
```python
rec = si.common_reference(recording, reference='global', operator='average')
```

### 局部参考
```python
# 按通道局部组参考
rec = si.common_reference(recording, reference='local', local_radius=(30, 100))
```

## 坏通道检测与移除

### 自动检测
```python
# 检测坏通道
bad_channel_ids, channel_labels = si.detect_bad_channels(
    recording,
    method='coherence+psd',
    dead_channel_threshold=-0.5,
    noisy_channel_threshold=1.0,
    outside_channel_threshold=-0.3,
    n_neighbors=11
)

print(f"坏通道: {bad_channel_ids}")
print(f"标签: {dict(zip(bad_channel_ids, channel_labels))}")
```

### 移除坏通道
```python
rec_clean = si.remove_bad_channels(recording, bad_channel_ids=bad_channel_ids)
```

### 插值坏通道
```python
rec_interp = si.interpolate_bad_channels(recording, bad_channel_ids=bad_channel_ids)
```

## 运动校正

### 运动估计
```python
# 估计运动（漂移）
motion, temporal_bins, spatial_bins = si.estimate_motion(
    recording,
    method='decentralized',
    rigid=False,              # 非刚性运动估计
    win_step_um=50,           # 空间窗口步长
    win_sigma_um=150,         # 空间窗口标准差
    progress_bar=True
)
```

### 应用运动校正
```python
rec_corrected = si.correct_motion(
    recording,
    motion,
    temporal_bins,
    spatial_bins,
    interpolate_motion_border=True
)
```

### 运动可视化
```python
si.plot_motion(motion, temporal_bins, spatial_bins)
```

## 探针特定处理

### Neuropixels 1.0
```python
# 相位校正（每通道独立ADC）
rec = si.phase_shift(recording)

# 标准流程
rec = si.bandpass_filter(rec, freq_min=300, freq_max=6000)
rec = si.common_reference(rec, reference='global', operator='median')
```

### Neuropixels 2.0
```python
# 无需相位校正（单ADC）
rec = si.bandpass_filter(recording, freq_min=300, freq_max=6000)
rec = si.common_reference(rec, reference='global', operator='median')
```

### 多探针片（Neuropixels 2.0 四片式）
```python
# 按探针片分组参考
groups = recording.get_channel_groups()  # 获取探针片分组
rec = si.common_reference(recording, reference='global', operator='median', groups=groups)
```

## 白化处理

```python
# 白化数据（通道解相关）
rec_whitened = si.whiten(recording, mode='local', local_radius_um=100)

# 全局白化
rec_whitened = si.whiten(recording, mode='global')
```

## 伪迹消除

### 消除刺激伪迹
```python
# 定义伪迹时间点（采样点单位）
triggers = [10000, 20000, 30000]  # 采样索引

rec = si.remove_artifacts(
    recording,
    triggers,
    ms_before=0.5,
    ms_after=3.0,
    mode='cubic'  # 'zeros'、'linear'、'cubic'
)
```

### 消除饱和区段
```python
rec = si.blank_staturation(recording, threshold=0.95, fill_value=0)
```

## 保存预处理数据

### 二进制格式（推荐）
```python
rec_preprocessed.save(folder='preprocessed/', format='binary', n_jobs=4)
```

### Zarr格式（压缩存储）
```python
rec_preprocessed.save(folder='preprocessed.zarr', format='zarr')
```

### 保存为记录提取器
```python
# 保存供后续使用
rec_preprocessed.save(folder='preprocessed/', format='binary')

# 后续加载
rec_loaded = si.load_extractor('preprocessed/')
```

## 完整流程示例

```python
import spikeinterface.full as si

def preprocess_neuropixels(data_path, output_path):
    """标准 Neuropixels 预处理流程"""

    # 加载数据
    recording = si.read_spikeglx(data_path, stream_id='imec0.ap')
    print(f"已加载: {recording.get_num_channels()} 通道, "
          f"{recording.get_total_duration():.1f}秒")

    # 相位校正（仅限 NP 1.0）
    rec = si.phase_shift(recording)

    # 滤波
    rec = si.bandpass_filter(rec, freq_min=300, freq_max=6000)

    # 检测并移除坏通道
    bad_ids, _ = si.detect_bad_channels(rec)
    if len(bad_ids) > 0:
        print(f"正在移除 {len(bad_ids)} 个坏通道: {bad_ids}")
        rec = si.interpolate_bad_channels(rec, bad_ids)

    # 公共参考
    rec = si.common_reference(rec, reference='global', operator='median')

    # 保存
    rec.save(folder=output_path, format='binary', n_jobs=4)
    print(f"已保存至: {output_path}")

    return rec

# 使用示例
rec_preprocessed = preprocess_neuropixels(
    '/path/to/spikeglx/data',
    '/path/to/preprocessed'
)
```

## 性能优化建议

```python
# 使用并行处理
rec.save(folder='output/', n_jobs=-1)  # 使用所有核心

# 任务参数内存管理
job_kwargs = dict(n_jobs=8, chunk_duration='1s', progress_bar=True)
rec.save(folder='output/', **job_kwargs)

# 设置全局任务参数
si.set_global_job_kwargs(n_jobs=8, chunk_duration='1s')
```
