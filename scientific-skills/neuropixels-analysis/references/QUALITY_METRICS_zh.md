# 质量指标参考指南

使用 SpikeInterface 指标和 Allen/IBL 标准进行单元质量评估的全面指南。

## 概述

质量指标评估排序单元的三个方面：

| 类别 | 核心问题 | 关键指标 |
|----------|----------|-------------|
| **污染度** (I类错误) | 是否包含多个神经元的脉冲？ | ISI 违例率、信噪比 |
| **完整度** (II类错误) | 是否遗漏了脉冲？ | 振幅截断值、存在比率 |
| **稳定性** | 单元是否随时间保持稳定？ | 漂移指标、振幅变异系数 |

## 计算质量指标

```python
import spikeinterface.full as si

# 创建分析器并计算波形
analyzer = si.create_sorting_analyzer(sorting, recording, sparse=True)
analyzer.compute('random_spikes', max_spikes_per_unit=500)
analyzer.compute('waveforms', ms_before=1.5, ms_after=2.0)
analyzer.compute('templates')
analyzer.compute('noise_levels')
analyzer.compute('spike_amplitudes')
analyzer.compute('principal_components', n_components=5)

# 计算所有质量指标
analyzer.compute('quality_metrics')

# 或计算特定指标
analyzer.compute('quality_metrics', metric_names=[
    'firing_rate', 'snr', 'isi_violations_ratio',
    'presence_ratio', 'amplitude_cutoff'
])

# 获取结果
qm = analyzer.get_extension('quality_metrics').get_data()
print(qm.columns.tolist())  # 可用指标列表
```

## 指标定义与阈值

### 污染度指标

#### ISI 违例比率
违反不应期的脉冲比例。所有神经元均有约1.5ms的不应期。

```python
# 使用自定义不应期计算
analyzer.compute('quality_metrics',
                 metric_names=['isi_violations_ratio'],
                 isi_threshold_ms=1.5,
                 min_isi_ms=0.0)
```

| 数值 | 解释 |
|-------|---------------|
| < 0.01 | 优秀（良好隔离的单单元） |
| 0.01 - 0.1 | 良好（轻微污染） |
| 0.1 - 0.5 | 中等（可能为多单元活动） |
| > 0.5 | 差（可能为多单元） |

**参考文献：** Hill et al. (2011) J Neurosci 31:8699-8705

#### 信噪比 (SNR)
峰值波形振幅与背景噪声的比值。

```python
analyzer.compute('quality_metrics', metric_names=['snr'])
```

| 数值 | 解释 |
|-------|---------------|
| > 10 | 优秀 |
| 5 - 10 | 良好 |
| 2 - 5 | 可接受 |
| < 2 | 差（可能为噪声） |

#### 隔离距离
PCA空间中到最近簇的马氏距离。

```python
analyzer.compute('quality_metrics',
                 metric_names=['isolation_distance'],
                 n_neighbors=4)
```

| 数值 | 解释 |
|-------|---------------|
| > 50 | 良好隔离 |
| 20 - 50 | 中等隔离 |
| < 20 | 隔离性差 |

#### L-比率
基于马氏距离的污染度量。

| 数值 | 解释 |
|-------|---------------|
| < 0.05 | 良好隔离 |
| 0.05 - 0.1 | 可接受 |
| > 0.1 | 存在污染 |

#### D-prime
单元与最近邻域的可区分度。

| 数值 | 解释 |
|-------|---------------|
| > 8 | 分离度极佳 |
| 5 - 8 | 分离度良好 |
| < 5 | 分离度差 |

### 完整度指标

#### 振幅截断值
估计低于检测阈值的脉冲比例。

```python
analyzer.compute('quality_metrics',
                 metric_names=['amplitude_cutoff'],
                 peak_sign='neg')  # 'neg'、'pos' 或 'both'
```

| 数值 | 解释 |
|-------|---------------|
| < 0.01 | 优秀（几乎完整） |
| 0.01 - 0.1 | 良好 |
| 0.1 - 0.2 | 中等（部分脉冲遗漏） |
| > 0.2 | 差（大量脉冲遗漏） |

**精确时序分析要求：** < 0.01

#### 存在比率
检测到脉冲的录制时间占比。

```python
analyzer.compute('quality_metrics',
                 metric_names=['presence_ratio'],
                 bin_duration_s=60)  # 1分钟分箱
```

| 数值 | 解释 |
|-------|---------------|
| > 0.99 | 优秀 |
| 0.9 - 0.99 | 良好 |
| 0.8 - 0.9 | 可接受 |
| < 0.8 | 单元可能已漂移消失 |

### 稳定性指标

#### 漂移指标
测量单元随时间的位置移动。

```python
analyzer.compute('quality_metrics',
                 metric_names=['drift_ptp', 'drift_std', 'drift_mad'])
```

| 指标 | 描述 | 良好值 |
|--------|-------------|------------|
| `drift_ptp` | 峰峰值漂移 (μm) | < 40 |
| `drift_std` | 漂移标准差 | < 10 |
| `drift_mad` | 漂移中位数绝对偏差 | < 10 |

#### 振幅变异系数
脉冲振幅的变异系数。

| 数值 | 解释 |
|-------|---------------|
| < 0.25 | 非常稳定 |
| 0.25 - 0.5 | 可接受 |
| > 0.5 | 不稳定（漂移或污染） |

### 聚类质量指标

#### 轮廓系数
聚类内聚度与分离度 (-1 到 1)。

| 数值 | 解释 |
|-------|---------------|
| > 0.5 | 聚类定义清晰 |
| 0.25 - 0.5 | 中等 |
| < 0.25 | 聚类重叠 |

#### 最近邻指标

```python
analyzer.compute('quality_metrics',
                 metric_names=['nn_hit_rate', 'nn_miss_rate'],
                 n_neighbors=4)
```

| 指标 | 描述 | 良好值 |
|--------|-------------|------------|
| `nn_hit_rate` | 同单元邻近点的脉冲比例 | > 0.9 |
| `nn_miss_rate` | 异单元邻近点的脉冲比例 | < 0.1 |

## 标准筛选条件

### Allen研究所默认值

```python
# Allen视觉编码/行为默认标准
allen_query = """
    presence_ratio > 0.95 and
    isi_violations_ratio < 0.5 and
    amplitude_cutoff < 0.1
"""
good_units = qm.query(allen_query).index.tolist()
```

### IBL标准

```python
# IBL可重复电生理标准
ibl_query = """
    presence_ratio > 0.9 and
    isi_violations_ratio < 0.1 and
    amplitude_cutoff < 0.1 and
    firing_rate > 0.1
"""
good_units = qm.query(ibl_query).index.tolist()
```

### 严格单单元标准

```python
# 适用于精确时序/脉冲时序分析
strict_query = """
    snr > 5 and
    presence_ratio > 0.99 and
    isi_violations_ratio < 0.01 and
    amplitude_cutoff < 0.01 and
    isolation_distance > 20 and
    drift_ptp < 40
"""
single_units = qm.query(strict_query).index.tolist()
```

### 多单元活动 (MUA)

```python
# 包含多单元活动
mua_query = """
    snr > 2 and
    presence_ratio > 0.5 and
    isi_violations_ratio < 1.0
"""
all_units = qm.query(mua_query).index.tolist()
```

## 可视化

### 质量指标概览

```python
# 绘制所有指标
si.plot_quality_metrics(analyzer)
```

### 单指标分布

```python
import matplotlib.pyplot as plt

fig, axes = plt.subplots(2, 3, figsize=(15, 10))

metrics = ['snr', 'isi_violations_ratio', 'presence_ratio',
           'amplitude_cutoff', 'firing_rate', 'drift_ptp']

for ax, metric in zip(axes.flat, metrics):
    ax.hist(qm[metric].dropna(), bins=50, edgecolor='black')
    ax.set_xlabel(metric)
    ax.set_ylabel('计数')
    # 添加阈值线
    if metric == 'snr':
        ax.axvline(5, color='r', linestyle='--', label='阈值')
    elif metric == 'isi_violations_ratio':
        ax.axvline(0.01, color='r', linestyle='--')
    elif metric == 'presence_ratio':
        ax.axvline(0.9, color='r', linestyle='--')

plt.tight_layout()
```

### 单元质量摘要

```python
# 综合单元摘要图
si.plot_unit_summary(analyzer, unit_id=0)
```

### 质量 vs 发放率

```python
fig, ax = plt.subplots()
scatter = ax.scatter(qm['firing_rate'], qm['snr'],
                     c=qm['isi_violations_ratio'],
                     cmap='RdYlGn_r', alpha=0.6)
ax.set_xlabel('发放率 (Hz)')
ax.set_ylabel('信噪比')
plt.colorbar(scatter, label='ISI违例率')
ax.set_xscale('log')
```

## 批量计算所有指标

```python
# 完整质量指标计算
all_metric_names = [
    # 发放特性
    'firing_rate', 'presence_ratio',
    # 波形
    'snr', 'amplitude_cutoff', 'amplitude_cv_median', 'amplitude_cv_range',
    # ISI
    'isi_violations_ratio', 'isi_violations_count',
    # 漂移
    'drift_ptp', 'drift_std', 'drift_mad',
    # 隔离度（需PCA）
    'isolation_distance', 'l_ratio', 'd_prime',
    # 最近邻（需PCA）
    'nn_hit_rate', 'nn_miss_rate',
    # 聚类质量
    'silhouette_score',
    # 同步性
    'sync_spike_2', 'sync_spike_4', 'sync_spike_8',
]

# 先计算PCA（部分指标必需）
analyzer.compute('principal_components', n_components=5)

# 计算指标
analyzer.compute('quality_metrics', metric_names=all_metric_names)
qm = analyzer.get_extension('quality_metrics').get_data()

# 保存为CSV
qm.to_csv('quality_metrics.csv')
```

## 自定义指标

```python
from spikeinterface.qualitymetrics import compute_firing_rates, compute_snrs

# 计算独立指标
firing_rates = compute_firing_rates(sorting)
snrs = compute_snrs(analyzer)

# 添加自定义指标到DataFrame
qm['custom_score'] = qm['snr'] * qm['presence_ratio'] / (qm['isi_violations_ratio'] + 0.001)
```

## 参考文献

- [SpikeInterface质量指标文档](https://spikeinterface.readthedocs.io/en/latest/modules/qualitymetrics.html)
- [Allen研究所电生理质量指标](https://allensdk.readthedocs.io/en/latest/_static/examples/nb/ecephys_quality_metrics.html)
- Hill et al. (2011) "Quality metrics to accompany spike sorting of extracellular signals"
- Siegle et al. (2021) "Survey of spiking in the mouse visual system reveals functional hierarchy"
