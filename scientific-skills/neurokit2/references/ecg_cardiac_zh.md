# 心电图与心电信号处理

## 概述

处理心电图（ECG）和光电容积脉搏波（PPG）信号用于心血管分析。本模块提供R波峰值检测、波形描绘、质量评估和心率分析的综合工具。

## 主处理流程

### ecg_process()

完整的自动化ECG处理流程，协调多个处理步骤。

```python
signals, info = nk.ecg_process(ecg_signal, sampling_rate=1000, method='neurokit')
```

**处理步骤：**
1. 信号清洗（噪声去除）
2. R波峰值检测
3. 心率计算
4. 质量评估
5. QRS波群描绘（P波、Q波、S波、T波）
6. 心动周期相位判定

**返回：**
- `signals`：包含清洗后ECG信号、峰值标记、心率、质量指标和心动相位的DataFrame
- `info`：包含R波位置和处理参数的字典

**常用方法：**
- `'neurokit'`（默认）：NeuroKit2综合流程
- `'biosppy'`：基于BioSPPy的处理
- `'pantompkins1985'`：Pan-Tompkins算法
- `'hamilton2002'`、`'elgendi2010'`、`'engzeemod2012'`：替代方法

## 预处理函数

### ecg_clean()

使用特定方法过滤去除原始ECG信号中的噪声。

```python
cleaned_ecg = nk.ecg_clean(ecg_signal, sampling_rate=1000, method='neurokit')
```

**方法：**
- `'neurokit'`：高通巴特沃斯滤波器（0.5 Hz）+ 工频滤波
- `'biosppy'`：0.67-45 Hz FIR滤波
- `'pantompkins1985'`：5-15 Hz带通滤波 + 基于导数的处理
- `'hamilton2002'`：8-16 Hz带通滤波
- `'elgendi2010'`：8-20 Hz带通滤波
- `'engzeemod2012'`：0.5-40 Hz FIR带通滤波

**关键参数：**
- `powerline`：去除50或60 Hz工频噪声（默认：50）

### ecg_peaks()

检测ECG信号中的R波峰值，支持伪差校正。

```python
peaks_dict, info = nk.ecg_peaks(cleaned_ecg, sampling_rate=1000, method='neurokit', correct_artifacts=False)
```

**可用方法（13+种算法）：**
- `'neurokit'`：优化可靠性的混合方法
- `'pantompkins1985'`：经典Pan-Tompkins算法
- `'hamilton2002'`：Hamilton自适应阈值法
- `'christov2004'`：Christov自适应方法
- `'gamboa2008'`：Gamboa方法
- `'elgendi2010'`：Elgendi双移动平均法
- `'engzeemod2012'`：改进的Engelse-Zeelenberg法
- `'kalidas2017'`：基于XQRS的方法
- `'martinez2004'`、`'rodrigues2021'`、`'koka2022'`、`'promac'`：高级方法

**伪差校正：**
设置`correct_artifacts=True`应用Lipponen & Tarvainen (2019)校正：
- 检测异位搏动、长短间期、漏搏
- 使用可配置参数的阈值检测

**返回：**
- 包含`'ECG_R_Peaks'`键的字典（存储峰值索引）

### ecg_delineate()

识别P波、Q波、S波、T波及其起始/终止点。

```python
waves, waves_peak = nk.ecg_delineate(cleaned_ecg, rpeaks, sampling_rate=1000, method='dwt')
```

**方法：**
- `'dwt'`（默认）：基于离散小波变换的检测
- `'peak'`：R波峰附近的简单峰值检测
- `'cwt'`：连续小波变换（Martinez等，2004）

**检测组件：**
- P波：`ECG_P_Peaks`、`ECG_P_Onsets`、`ECG_P_Offsets`
- Q波：`ECG_Q_Peaks`
- S波：`ECG_S_Peaks`
- T波：`ECG_T_Peaks`、`ECG_T_Onsets`、`ECG_T_Offsets`
- QRS波群：起始与终止点

**返回：**
- `waves`：包含所有波形索引的字典
- `waves_peak`：包含峰值振幅的字典

### ecg_quality()

评估ECG信号的完整性与质量。

```python
quality = nk.ecg_quality(ecg_signal, rpeaks=None, sampling_rate=1000, method='averageQRS')
```

**方法：**
- `'averageQRS'`（默认）：模板匹配相关法（Zhao & Zhang, 2018）
  - 返回每个心跳的质量评分（0-1）
  - 阈值：>0.6 = 良好质量
- `'zhao2018'`：使用峰度与功率谱分布的多指标方法

**应用场景：**
- 识别低质量信号段
- 分析前过滤噪声干扰的心跳
- 验证R波峰值检测准确性

## 分析函数

### ecg_analyze()

根据信号时长自动选择事件相关或区间相关模式的高级分析。

```python
analysis = nk.ecg_analyze(signals, sampling_rate=1000, method='auto')
```

**模式选择：**
- 时长 < 10秒 → 事件相关分析
- 时长 ≥ 10秒 → 区间相关分析

**返回：**
根据分析模式输出对应心脏指标的DataFrame。

### ecg_eventrelated()

分析刺激锁定的ECG片段，获取事件相关响应。

```python
results = nk.ecg_eventrelated(epochs)
```

**计算指标：**
- `ECG_Rate_Baseline`：刺激前平均心率
- `ECG_Rate_Min/Max`：片段内最小/最大心率
- `ECG_Phase_Atrial/Ventricular`：刺激起始时的心动相位
- 跨片段时间窗的心率动态变化

**应用场景：**
离散试验的实验范式（如刺激呈现、任务事件）。

### ecg_intervalrelated()

分析连续ECG记录，适用于静息态或长时程监测。

```python
results = nk.ecg_intervalrelated(signals, sampling_rate=1000)
```

**计算指标：**
- `ECG_Rate_Mean`：区间平均心率
- 综合心率变异性（HRV）指标（调用`hrv()`函数）
  - 时域：SDNN、RMSSD、pNN50等
  - 频域：LF、HF、LF/HF比值
  - 非线性：庞加莱图、熵值、分形度量

**最低时长要求：**
- 基础心率：任意时长
- HRV频域指标：建议≥60秒，1-5分钟为佳

## 实用函数

### ecg_rate()

根据R波间期计算瞬时心率。

```python
heart_rate = nk.ecg_rate(peaks, sampling_rate=1000, desired_length=None)
```

**方法：**
- 计算连续R波间的搏动间期（IBIs）
- 转换为每分钟心跳数（BPM）：60 / IBI
- 若指定`desired_length`则插值至信号长度

**返回：**
- 瞬时心率值数组

### ecg_phase()

判定心房与心室收缩/舒张相位。

```python
cardiac_phase = nk.ecg_phase(ecg_cleaned, rpeaks, delineate_info)
```

**计算相位：**
- `ECG_Phase_Atrial`：心房收缩期（1） vs. 舒张期（0）
- `ECG_Phase_Ventricular`：心室收缩期（1） vs. 舒张期（0）
- `ECG_Phase_Completion_Atrial/Ventricular`：相位完成百分比（0-1）

**应用场景：**
- 心动周期锁定的刺激呈现
- 心理生理学实验中事件与心动周期的时序对齐

### ecg_segment()

提取单次心跳用于形态学分析。

```python
heartbeats = nk.ecg_segment(ecg_cleaned, rpeaks, sampling_rate=1000)
```

**返回：**
- 包含各心跳片段的字典
- 以R波峰为中心，可配置前后时间窗
- 适用于逐跳形态比较

### ecg_invert()

自动检测并校正倒置的ECG信号。

```python
corrected_ecg, is_inverted = nk.ecg_invert(ecg_signal, sampling_rate=1000)
```

**方法：**
- 分析QRS波群极性
- 若主波为负向则翻转信号
- 返回校正后信号及倒置状态

### ecg_rsp()

提取ECG衍生呼吸（EDR）信号作为呼吸代用指标。

```python
edr_signal = nk.ecg_rsp(ecg_cleaned, sampling_rate=1000, method='vangent2019')
```

**方法：**
- `'vangent2019'`：0.1-0.4 Hz带通滤波
- `'charlton2016'`：0.15-0.4 Hz带通滤波
- `'soni2019'`：0.08-0.5 Hz带通滤波

**应用场景：**
- 无直接呼吸信号时的呼吸估计
- 多模态生理信号分析

## 仿真与可视化

### ecg_simulate()

生成合成ECG信号用于测试验证。

```python
synthetic_ecg = nk.ecg_simulate(duration=10, sampling_rate=1000, heart_rate=70, method='ecgsyn', noise=0.01)
```

**方法：**
- `'ecgsyn'`：真实动力学模型（McSharry等，2003）
  - 模拟P-QRS-T复合波形态
  - 生理合理的波形
- `'simple'`：快速小波近似法
  - 类高斯QRS复合波
  - 计算高效但真实性较低

**关键参数：**
- `heart_rate`：平均BPM（默认：70）
- `heart_rate_std`：心率变异性幅度（默认：1）
- `noise`：高斯噪声水平（默认：0.01）
- `random_state`：随机种子

### ecg_plot()

可视化处理后的ECG信号及检测结果。

```python
nk.ecg_plot(signals, info)
```

**显示内容：**
- 原始与清洗后的ECG信号
- 叠加显示的R波峰值
- 心率轨迹
- 信号质量指示器

## ECG处理注意事项

### 采样率建议
- **最低要求**：基础R波检测需250 Hz
- **推荐值**：波形描绘需500-1000 Hz
- **高分辨率**：详细形态分析需2000+ Hz

### 信号时长要求
- **R波检测**：任意时长（≥2次心跳）
- **基础心率**：≥10秒
- **HRV时域分析**：≥60秒
- **HRV频域分析**：1-5分钟（最佳）
- **超低频HRV**：≥24小时

### 常见问题与解决方案

**R波检测不佳：**
- 尝试不同方法：`method='pantompkins1985'`通常稳健
- 确保足够采样率（≥250 Hz）
- 检查信号是否倒置：使用`ecg_invert()`
- 应用伪差校正：`correct_artifacts=True`

**信号噪声大：**
- 根据噪声类型选用适当清洗方法
- 非欧美地区需调整工频参数
- 分析前进行信号质量评估

**波形成分缺失：**
- 提升采样率（描绘需≥500 Hz）
- 尝试不同描绘方法（`'dwt'`、`'peak'`、`'cwt'`）
- 用`ecg_quality()`验证信号质量

## 多信号集成

### ECG + RSP：呼吸性窦性心律不齐
```python
# 并行处理双信号
ecg_signals, ecg_info = nk.ecg_process(ecg, sampling_rate=1000)
rsp_signals, rsp_info = nk.rsp_process(rsp, sampling_rate=1000)

# 计算RSA
rsa = nk.hrv_rsa(ecg_info['ECG_R_Peaks'], rsp_signals['RSP_Clean'], sampling_rate=1000)
```

### 多模态集成
```python
# 批量处理多信号
bio_signals, bio_info = nk.bio_process(
    ecg=ecg_signal,
    rsp=rsp_signal,
    eda=eda_signal,
    sampling_rate=1000
)
```

## 参考文献

- Pan, J., & Tompkins, W. J. (1985). A real-time QRS detection algorithm. IEEE transactions on biomedical engineering, 32(3), 230-236.
- Hamilton, P. (2002). Open source ECG analysis. Computers in cardiology, 101-104.
- Martinez, J. P., Almeida, R., Olmos, S., Rocha, A. P., & Laguna, P. (2004). A wavelet-based ECG delineator: evaluation on standard databases. IEEE Transactions on biomedical engineering, 51(4), 570-581.
- Lipponen, J. A., & Tarvainen, M. P. (2019). A robust algorithm for heart rate variability time series artefact correction using novel beat classification. Journal of medical engineering & technology, 43(3), 173-181.
