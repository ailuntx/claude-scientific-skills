# 多信号整合（生物模块）

## 概述

生物模块提供统一功能，用于同时处理和分析多种生理信号。它作为协调器封装了信号专用处理函数，并支持集成多模态分析。

## 多信号处理

### bio_process()

通过单次函数调用同时处理多种生理信号。

```python
bio_signals, bio_info = nk.bio_process(ecg=None, rsp=None, eda=None, emg=None,
                                       ppg=None, eog=None, sampling_rate=1000)
```

**参数：**
- `ecg`：ECG信号数组（可选）
- `rsp`：呼吸信号数组（可选）
- `eda`：EDA信号数组（可选）
- `emg`：EMG信号数组（可选）
- `ppg`：PPG信号数组（可选）
- `eog`：EOG信号数组（可选）
- `sampling_rate`：采样率（Hz）（所有信号需一致或单独指定）

**返回值：**
- `bio_signals`：包含所有处理信号的统一DataFrame，列包括：
  - 信号特定特征（如`ECG_Clean`, `ECG_Rate`, `EDA_Phasic`, `RSP_Rate`）
  - 所有检测到的事件/峰值
  - 衍生指标
- `bio_info`：包含信号特定信息的字典（峰值位置、参数）

**示例：**
```python
# 同时处理ECG、呼吸和EDA信号
bio_signals, bio_info = nk.bio_process(
    ecg=ecg_signal,
    rsp=rsp_signal,
    eda=eda_signal,
    sampling_rate=1000
)

# 访问处理后的信号
ecg_clean = bio_signals['ECG_Clean']
rsp_rate = bio_signals['RSP_Rate']
eda_phasic = bio_signals['EDA_Phasic']

# 访问检测到的峰值
ecg_peaks = bio_info['ECG']['ECG_R_Peaks']
rsp_peaks = bio_info['RSP']['RSP_Peaks']
```

**内部工作流程：**
1. 每个信号通过专用处理函数处理：
   - `ecg_process()`处理ECG
   - `rsp_process()`处理呼吸
   - `eda_process()`处理EDA
   - `emg_process()`处理EMG
   - `ppg_process()`处理PPG
   - `eog_process()`处理EOG
2. 结果合并至统一DataFrame
3. 计算跨信号特征（如同时存在ECG和RSP时计算RSA）

**优势：**
- 简化多模态记录的API
- 统一所有信号的时间基准
- 自动计算跨信号特征
- 一致的输出格式

## 多信号分析

### bio_analyze()

对处理后的多模态信号执行综合分析。

```python
bio_results = nk.bio_analyze(bio_signals, sampling_rate=1000)
```

**参数：**
- `bio_signals`：来自`bio_process()`的DataFrame或自定义处理信号
- `sampling_rate`：采样率（Hz）

**返回值：**
- 包含所有检测信号类型分析结果的DataFrame：
  - 时长≥10秒时计算区间相关指标
  - 时长<10秒时计算事件相关指标
  - 跨信号指数（如ECG+RSP可用时计算RSA）

**各信号计算指标：**
- **ECG**：心率统计、HRV指数（时域、频域、非线性域）
- **RSP**：呼吸率统计、RRV、振幅测量
- **EDA**：SCR计数、振幅、紧张水平、交感神经指数
- **EMG**：激活计数、振幅统计
- **PPG**：类似ECG（心率、HRV）
- **EOG**：眨眼计数、眨眼频率

**跨信号指标：**
- **RSA（呼吸性窦性心律不齐）**：ECG+RSP同时存在时计算
- **心肺耦合**：相位同步指数
- **多模态唤醒**：组合自主神经指数

**示例：**
```python
# 分析处理后的信号
results = nk.bio_analyze(bio_signals, sampling_rate=1000)

# 访问结果
heart_rate_mean = results['ECG_Rate_Mean']
hrv_rmssd = results['HRV_RMSSD']
breathing_rate = results['RSP_Rate_Mean']
scr_count = results['SCR_Peaks_N']
rsa_value = results['RSA']  # ECG和RSP同时存在时可用
```

## 跨信号特征

当同时处理多个信号时，NeuroKit2可计算集成特征：

### 呼吸性窦性心律不齐（RSA）

ECG和呼吸信号同时存在时自动计算。

```python
bio_signals, bio_info = nk.bio_process(ecg=ecg, rsp=rsp, sampling_rate=1000)
results = nk.bio_analyze(bio_signals, sampling_rate=1000)

rsa = results['RSA']  # 自动包含
```

**计算方式：**
- 呼吸对高频HRV的调制
- 需同步的ECG R峰和呼吸信号
- 方法：Porges-Bohrer法或峰谷法

**解读：**
- RSA值越高：副交感（迷走）神经影响越强
- 心肺耦合的标志物
- 健康状态和情绪调节能力的指标

### ECG衍生呼吸（EDR）

若呼吸信号不可用，NeuroKit2可从ECG估计：

```python
ecg_signals, ecg_info = nk.ecg_process(ecg, sampling_rate=1000)

# 提取EDR
edr = nk.ecg_rsp(ecg_signals['ECG_Clean'], sampling_rate=1000)
```

**应用场景：**
- 直接测量不可用时估计呼吸
- 交叉验证呼吸测量结果

### 心电-皮电整合

同步心电活动与皮肤电活动：

```python
bio_signals, bio_info = nk.bio_process(ecg=ecg, eda=eda, sampling_rate=1000)

# 两种信号可用于集成分析
ecg_rate = bio_signals['ECG_Rate']
eda_phasic = bio_signals['EDA_Phasic']

# 计算相关性或耦合指标
correlation = ecg_rate.corr(eda_phasic)
```

## 实践工作流

### 完整多模态记录分析

```python
import neurokit2 as nk
import pandas as pd

# 1. 加载多模态生理数据
ecg = load_ecg()        # 自定义数据加载函数
rsp = load_rsp()
eda = load_eda()
emg = load_emg()

# 2. 同步处理所有信号
bio_signals, bio_info = nk.bio_process(
    ecg=ecg,
    rsp=rsp,
    eda=eda,
    emg=emg,
    sampling_rate=1000
)

# 3. 可视化处理后的信号
import matplotlib.pyplot as plt

fig, axes = plt.subplots(4, 1, figsize=(15, 12), sharex=True)

# ECG
axes[0].plot(bio_signals.index / 1000, bio_signals['ECG_Clean'])
axes[0].set_ylabel('ECG')
axes[0].set_title('多模态生理记录')

# 呼吸
axes[1].plot(bio_signals.index / 1000, bio_signals['RSP_Clean'])
axes[1].set_ylabel('呼吸')

# EDA
axes[2].plot(bio_signals.index / 1000, bio_signals['EDA_Phasic'])
axes[2].set_ylabel('EDA（相位）')

# EMG
axes[3].plot(bio_signals.index / 1000, bio_signals['EMG_Amplitude'])
axes[3].set_ylabel('EMG振幅')
axes[3].set_xlabel('时间（秒）')

plt.tight_layout()
plt.show()

# 4. 分析所有信号
results = nk.bio_analyze(bio_signals, sampling_rate=1000)

# 5. 提取关键指标
print("平均心率:", results['ECG_Rate_Mean'])
print("HRV (RMSSD):", results['HRV_RMSSD'])
print("呼吸频率:", results['RSP_Rate_Mean'])
print("SCR计数:", results['SCR_Peaks_N'])
print("RSA:", results['RSA'])
```

### 事件相关多模态分析

```python
# 1. 处理信号
bio_signals, bio_info = nk.bio_process(ecg=ecg, rsp=rsp, eda=eda, sampling_rate=1000)

# 2. 检测事件
events = nk.events_find(trigger_channel, threshold=0.5)

# 3. 为所有信号创建事件片段
epochs = nk.epochs_create(bio_signals, events, sampling_rate=1000,
                          epochs_start=-1.0, epochs_end=10.0,
                          event_labels=event_labels,
                          event_conditions=event_conditions)

# 4. 信号特定的事件相关分析
ecg_eventrelated = nk.ecg_eventrelated(epochs)
rsp_eventrelated = nk.rsp_eventrelated(epochs)
eda_eventrelated = nk.eda_eventrelated(epochs)

# 5. 合并结果
all_results = pd.merge(ecg_eventrelated, rsp_eventrelated,
                       left_index=True, right_index=True)
all_results = pd.merge(all_results, eda_eventrelated,
                       left_index=True, right_index=True)

# 6. 按条件进行统计比较
all_results['Condition'] = event_conditions
condition_means = all_results.groupby('Condition').mean()
```

### 不同采样率处理

处理具有不同原生采样率的信号：

```python
# ECG为1000Hz，EDA为100Hz
bio_signals, bio_info = nk.bio_process(
    ecg=ecg_1000hz,
    eda=eda_100hz,
    sampling_rate=1000  # 目标采样率
)
# EDA将在内部自动重采样至1000Hz
```

或分别处理后合并：

```python
# 按原生采样率处理
ecg_signals, ecg_info = nk.ecg_process(ecg, sampling_rate=1000)
eda_signals, eda_info = nk.eda_process(eda, sampling_rate=100)

# 重采样至统一频率
eda_resampled = nk.signal_resample(eda_signals, sampling_rate=100,
                                   desired_sampling_rate=1000)

# 手动合并
bio_signals = pd.concat([ecg_signals, eda_resampled], axis=1)
```

## 应用场景

### 综合心理生理学研究

捕捉生理唤醒的多维度特征：

- **心电**：定向反应、注意力、情绪效价
- **呼吸**：唤醒度、压力、情绪调节
- **皮电**：交感神经唤醒、情绪强度
- **肌电**：肌肉紧张度、面部表情、惊跳反射

**示例：情绪图片观看**
- ECG：图片观看时心率减速（注意力）
- EDA：SCR反映情绪唤醒强度
- RSP：屏息或变化反映情绪投入度
- 面部EMG：皱眉肌（负面）、颧大肌（正面）反映情绪效价

### 压力与放松评估

多模态标记提供聚合证据：

- **压力升高**：↑心率、↓HRV、↑EDA、↑呼吸频率、↑肌肉紧张度
- **放松状态**：↓心率、↑HRV、↓EDA、↓呼吸频率、呼吸放缓、↓肌肉紧张度

**干预效果评估：**
- 比较干预前后的多模态指数
- 识别特定技术响应的模态

### 临床评估

**焦虑障碍：**
- 基线EDA、心率升高
- 对压力源的过度反应
- HRV和呼吸变异性降低

**抑郁症：**
- 自主神经平衡改变（↓HRV）
- EDA反应钝化
- 不规则呼吸模式

**创伤后应激障碍（PTSD）：**
- 过度唤醒（↑基线心率、↑EDA）
- 惊跳反射增强（EMG）
- RSA异常

### 人机交互

无干扰的用户状态评估：

- **认知负荷**：↓HRV、↑EDA、眨眼抑制
- **挫败感**：↑心率、↑EDA、↑肌肉紧张度
- **投入度**：适度唤醒、同步化响应
- **厌倦感**：低唤醒、不规则模式

### 运动表现与恢复

监测训练负荷与恢复状态：

- **静息HRV**：每日监测过度训练
- **EDA**：交感神经激活与压力
- **呼吸**：运动/恢复期间的呼吸模式
- **多模态整合**：综合恢复评估

## 多模态记录优势

**聚合效度：**
- 多指标聚焦同一构念（如唤醒度）
- 比单一测量更稳健

**区分效度：**
- 不同信号在特定条件下分离
- ECG同时反映交感和副交感神经
- EDA主要反映交感神经

**系统整合：**
- 理解全身生理协调机制
- 跨信号耦合指标（RSA、相干性）

**冗余性与鲁棒性：**
- 单信号质量差时其他信号可用
- 跨模态交叉验证结果

**丰富解释维度：**
- 心率减速+SCR上升=伴随唤醒的定向反应
- 心率加速+无SCR=无交感唤醒的心血管反应

## 注意事项

### 硬件与同步

- **同设备**：信号固有同步
- **多设备**：需共同触发器/时间戳
  - 使用硬件触发器标记同步事件
  - 基于事件标记的软件对齐
  - 验证同步质量（交叉关联冗余信号）

### 跨模态信号质量

- 不同信号质量可能不均等
- 根据研究问题确定优先级
- 记录各信号质量问题

### 计算成本

- 处理多信号增加计算时间
- 大数据集考虑分批处理
- 适当降采样减轻负荷

### 分析复杂度

- 信号越多=变量越多=统计比较越复杂
- 未校正时I类错误（假阳性）风险增加
- 采用多变量方法或预注册分析

### 结果解读

- 避免过度解读复杂多模态模式
- 以生理学理论为基础
- 重要发现需重复验证

## 参考文献

- Berntson, G. G., Cacioppo, J. T., & Quigley, K. S. (1993). Respiratory sinus arrhythmia: autonomic origins, physiological mechanisms, and psychophysiological implications. Psychophysiology, 30(2), 183-196.
- Cacioppo, J. T., Tassinary, L. G., & Berntson, G. (Eds.). (2017). Handbook of psychophysiology (4th ed.). Cambridge University Press.
- Kreibig, S. D. (2010). Autonomic nervous system activity in emotion: A review. Biological psychology, 84(3), 394-421.
- Laborde, S., Mosley, E., & Thayer, J. F. (2017). Heart rate variability and cardiac vagal tone in psychophysiological research–recommendations for experiment planning, data analysis, and data reporting. Frontiers in psychology, 8, 213.
