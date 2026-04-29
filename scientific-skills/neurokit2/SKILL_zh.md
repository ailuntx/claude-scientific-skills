---
name: neurokit2
description: 用于分析生理数据的综合生物信号处理工具包，涵盖心电（ECG）、脑电（EEG）、皮肤电（EDA）、呼吸（RSP）、光电容积（PPG）、肌电（EMG）和眼电（EOG）信号。适用于处理心血管信号、大脑活动、皮肤电反应、呼吸模式、肌肉活动或眼球运动。可用于心率变异性分析、事件相关电位、复杂度测量、自主神经系统评估、心理生理学研究及多模态生理信号整合。
license: MIT 许可证
metadata:
    skill-author: K-Dense Inc.

# NeuroKit2

## 概述

NeuroKit2 是一个用于处理和分析生理信号（生物信号）的综合性 Python 工具包。使用此技能可处理心血管、神经、自主神经、呼吸和肌肉信号，适用于心理生理学研究、临床应用和人机交互研究。

## 适用场景

在以下场景中使用此技能：
- **心脏信号**：心电（ECG）、光电容积（PPG）、心率变异性（HRV）、脉搏分析
- **大脑信号**：脑电（EEG）频段、微状态、复杂度、源定位
- **自主神经信号**：皮肤电活动（EDA/GSR）、皮肤电导反应（SCR）
- **呼吸信号**：呼吸频率、呼吸变异性（RRV）、单位时间容积
- **肌肉信号**：肌电（EMG）振幅、肌肉激活检测
- **眼动追踪**：眼电（EOG）、眨眼检测与分析
- **多模态整合**：同步处理多种生理信号
- **复杂度分析**：熵测量、分形维度、非线性动力学

## 核心功能

### 1. 心脏信号处理（ECG/PPG）

处理心电图和光电容积描记信号以进行心血管分析。详见 `references/ecg_cardiac.md` 中的详细工作流程。

**主要工作流：**
- ECG 处理流程：清洗 → R 峰检测 → 波形解析 → 质量评估
- 时域、频域和非线性域的 HRV 分析
- PPG 脉搏分析与质量评估
- ECG 衍生呼吸信号提取

**关键函数：**
```python
import neurokit2 as nk

# 完整 ECG 处理流程
signals, info = nk.ecg_process(ecg_signal, sampling_rate=1000)

# 分析 ECG 数据（事件相关或区间相关）
analysis = nk.ecg_analyze(signals, sampling_rate=1000)

# 综合 HRV 分析
hrv = nk.hrv(peaks, sampling_rate=1000)  # 时域、频域、非线性域
```

### 2. 心率变异性分析

从心脏信号计算全面的 HRV 指标。所有指标和领域特定分析详见 `references/hrv.md`。

**支持的分析域：**
- **时域**：SDNN、RMSSD、pNN50、SDSD 及衍生指标
- **频域**：ULF、VLF、LF、HF、VHF 功率及比值
- **非线性域**：庞加莱图（SD1/SD2）、熵测量、分形维度
- **专项分析**：呼吸性窦性心律不齐（RSA）、递归量化分析（RQA）

**关键函数：**
```python
# 一次性获取所有 HRV 指标
hrv_indices = nk.hrv(peaks, sampling_rate=1000)

# 领域特定分析
hrv_time = nk.hrv_time(peaks)
hrv_freq = nk.hrv_frequency(peaks, sampling_rate=1000)
hrv_nonlinear = nk.hrv_nonlinear(peaks, sampling_rate=1000)
hrv_rsa = nk.hrv_rsa(peaks, rsp_signal, sampling_rate=1000)
```

### 3. 大脑信号分析（EEG）

分析脑电图信号的频段功率、复杂度和微状态模式。详细工作流程及 MNE 集成见 `references/eeg.md`。

**主要功能：**
- 频段功率分析（Delta、Theta、Alpha、Beta、Gamma）
- 通道质量评估与重参考
- 源定位（sLORETA、MNE）
- 微状态分割与转移动力学
- 全局场功率与相异度测量

**关键函数：**
```python
# 跨频段功率分析
power = nk.eeg_power(eeg_data, sampling_rate=250, channels=['Fz', 'Cz', 'Pz'])

# 微状态分析
microstates = nk.microstates_segment(eeg_data, n_microstates=4, method='kmod')
static = nk.microstates_static(microstates)
dynamic = nk.microstates_dynamic(microstates)
```

### 4. 皮肤电活动（EDA）

处理皮肤电导信号以评估自主神经系统。详细工作流程见 `references/eda.md`。

**主要工作流：**
- 信号分解为紧张性与相位性成分
- 皮肤电导反应（SCR）检测与分析
- 交感神经系统指数计算
- 自相关与变点检测

**关键函数：**
```python
# 完整 EDA 处理
signals, info = nk.eda_process(eda_signal, sampling_rate=100)

# 分析 EDA 数据
analysis = nk.eda_analyze(signals, sampling_rate=100)

# 交感神经系统活动
sympathetic = nk.eda_sympathetic(signals, sampling_rate=100)
```

### 5. 呼吸信号处理（RSP）

分析呼吸模式与呼吸变异性。详细工作流程见 `references/rsp.md`。

**主要功能：**
- 呼吸频率计算与变异性分析
- 呼吸幅度与对称性评估
- 单位时间呼吸容积（fMRI 应用）
- 呼吸幅度变异性（RAV）

**关键函数：**
```python
# 完整 RSP 处理
signals, info = nk.rsp_process(rsp_signal, sampling_rate=100)

# 呼吸频率变异性
rrv = nk.rsp_rrv(signals, sampling_rate=100)

# 单位时间呼吸容积
rvt = nk.rsp_rvt(signals, sampling_rate=100)
```

### 6. 肌电图（EMG）

处理肌肉活动信号以进行激活检测与幅度分析。工作流程见 `references/emg.md`。

**关键函数：**
```python
# 完整 EMG 处理
signals, info = nk.emg_process(emg_signal, sampling_rate=1000)

# 肌肉激活检测
activation = nk.emg_activation(signals, sampling_rate=1000, method='threshold')
```

### 7. 眼电图（EOG）

分析眼球运动与眨眼模式。工作流程见 `references/eog.md`。

**关键函数：**
```python
# 完整 EOG 处理
signals, info = nk.eog_process(eog_signal, sampling_rate=500)

# 提取眨眼特征
features = nk.eog_features(signals, sampling_rate=500)
```

### 8. 通用信号处理

对任意信号应用滤波、分解与变换操作。完整工具集见 `references/signal_processing.md`。

**关键操作：**
- 滤波（低通、高通、带通、带阻）
- 分解（EMD、SSA、小波）
- 峰值检测与校正
- 功率谱密度估计
- 信号插值与重采样
- 自相关与同步性分析

**关键函数：**
```python
# 滤波
filtered = nk.signal_filter(signal, sampling_rate=1000, lowcut=0.5, highcut=40)

# 峰值检测
peaks = nk.signal_findpeaks(signal)

# 功率谱密度
psd = nk.signal_psd(signal, sampling_rate=1000)
```

### 9. 复杂度与熵分析

计算非线性动力学、分形维度和信息论指标。所有可用指标见 `references/complexity.md`。

**可用测量方法：**
- **熵**：香农熵、近似熵、样本熵、排列熵、谱熵、模糊熵、多尺度熵
- **分形维度**：Katz 维度、Higuchi 维度、Petrosian 维度、Sevcik 维度、相关维度
- **非线性动力学**：李雅普诺夫指数、Lempel-Ziv 复杂度、递归量化
- **DFA**：去趋势波动分析、多重分形 DFA
- **信息论**：费雪信息、互信息

**关键函数：**
```python
# 一次性获取多种复杂度指标
complexity_indices = nk.complexity(signal, sampling_rate=1000)

# 特定测量方法
apen = nk.entropy_approximate(signal)
dfa = nk.fractal_dfa(signal)
lyap = nk.complexity_lyapunov(signal, sampling_rate=1000)
```

### 10. 事件相关分析

围绕刺激事件创建时段并分析生理响应。工作流程见 `references/epochs_events.md`。

**主要功能：**
- 根据事件标记创建时段
- 事件相关平均与可视化
- 基线校正选项
- 带置信区间的总平均计算

**关键函数：**
```python
# 在信号中查找事件
events = nk.events_find(trigger_signal, threshold=0.5)

# 围绕事件创建时段
epochs = nk.epochs_create(signals, events, sampling_rate=1000,
                          epochs_start=-0.5, epochs_end=2.0)

# 跨时段平均
grand_average = nk.epochs_average(epochs)
```

### 11. 多信号整合

同步处理多种生理信号并输出统一结果。整合工作流程见 `references/bio_module.md`。

**关键函数：**
```python
# 同步处理多信号
bio_signals, bio_info = nk.bio_process(
    ecg=ecg_signal,
    rsp=rsp_signal,
    eda=eda_signal,
    emg=emg_signal,
    sampling_rate=1000
)

# 分析所有处理后的信号
bio_analysis = nk.bio_analyze(bio_signals, sampling_rate=1000)
```

## 分析模式

NeuroKit2 根据数据时长自动选择两种分析模式：

**事件相关分析**（< 10 秒）：
- 分析刺激锁定的响应
- 基于时段的分割
- 适用于离散试次的实验范式

**区间相关分析**（≥ 10 秒）：
- 描述长时间生理模式
- 静息态或连续活动
- 适用于基线测量与长期监测

大多数 `*_analyze()` 函数会自动选择合适的模式。

## 安装

```bash
uv pip install neurokit2
```

开发版安装：
```bash
uv pip install https://github.com/neuropsychology/NeuroKit/zipball/dev
```

## 常用工作流

### 快速入门：ECG 分析
```python
import neurokit2 as nk

# 加载示例数据
ecg = nk.ecg_simulate(duration=60, sampling_rate=1000)

# 处理 ECG
signals, info = nk.ecg_process(ecg, sampling_rate=1000)

# 分析 HRV
hrv = nk.hrv(info['ECG_R_Peaks'], sampling_rate=1000)

# 可视化
nk.ecg_plot(signals, info)
```

### 多模态分析
```python
# 处理多信号
bio_signals, bio_info = nk.bio_process(
    ecg=ecg_signal,
    rsp=rsp_signal,
    eda=eda_signal,
    sampling_rate=1000
)

# 分析所有信号
results = nk.bio_analyze(bio_signals, sampling_rate=1000)
```

### 事件相关电位
```python
# 查找事件
events = nk.events_find(trigger_channel, threshold=0.5)

# 创建时段
epochs = nk.epochs_create(processed_signals, events,
                          sampling_rate=1000,
                          epochs_start=-0.5, epochs_end=2.0)

# 各信号类型的事件相关分析
ecg_epochs = nk.ecg_eventrelated(epochs)
eda_epochs = nk.eda_eventrelated(epochs)
```

## 参考文献

本技能包含按信号类型和分析方法组织的完整参考文档：

- **ecg_cardiac.md**：ECG/PPG 处理、R 峰检测、波形解析、质量评估
- **hrv.md**：全领域心率变异性指标
- **eeg.md**：EEG 分析、频段、微状态、源定位
- **eda.md**：皮肤电活动处理与 SCR 分析
- **rsp.md**：呼吸信号处理与变异性
- **ppg.md**：光电容积描记信号分析
- **emg.md**：肌电图处理与激活检测
- **eog.md**：眼电图与眨眼分析
- **signal_processing.md**：通用信号工具与变换
- **complexity.md**：熵、分形及非线性测量
- **epochs_events.md**：事件相关分析与时段创建
- **bio_module.md**：多信号整合工作流

使用 Read 工具加载特定参考文件以获取详细函数文档和参数说明。

## 附加资源

- 官方文档：https://neuropsychology.github.io/NeuroKit/
- GitHub 仓库：https://github.com/neuropsychology/NeuroKit
- 出版物：Makowski et al. (2021). NeuroKit2: A Python toolbox for neurophysiological signal processing. Behavior Research Methods. https://doi.org/10.3758/s13428-020-01516-y
