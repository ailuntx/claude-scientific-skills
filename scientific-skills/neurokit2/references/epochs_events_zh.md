# 事件相关分析与时段划分

## 概述

事件相关分析用于检测与特定刺激或事件时间锁定的生理响应。NeuroKit2 提供了跨所有信号类型的事件检测、时段创建、信号平均及事件相关特征提取工具。

## 事件检测

### events_find()

基于阈值穿越或信号变化自动检测信号中的事件/触发器。

```python
events = nk.events_find(event_channel, threshold=0.5, threshold_keep='above',
                        duration_min=1, inter_min=0)
```

**参数：**
- `threshold`：检测阈值
- `threshold_keep`：保留阈值`'above'`（上方）或`'below'`（下方）的事件
- `duration_min`：保留事件的最小持续时间（采样点数）
- `inter_min`：事件间最小间隔（采样点数）

**返回值：**
- 包含以下键的字典：
  - `'onset'`：事件起始索引
  - `'offset'`：事件结束索引（若适用）
  - `'duration'`：事件持续时间
  - `'label'`：事件标签（多事件类型时）

**典型应用场景：**

**实验TTL触发器：**
```python
# 触发通道：基线0V，事件期间5V脉冲
events = nk.events_find(trigger_channel, threshold=2.5, threshold_keep='above')
```

**按键检测：**
```python
# 检测按键信号上升沿
button_events = nk.events_find(button_signal, threshold=0.5, threshold_keep='above',
                               duration_min=10)  # 消抖处理
```

**状态变化检测：**
```python
# 检测高于/低于阈值的时段
high_arousal = nk.events_find(eda_signal, threshold='auto', duration_min=100)
```

### events_plot()

可视化事件相对于信号的时间分布。

```python
nk.events_plot(events, signal)
```

**显示内容：**
- 信号轨迹
- 事件标记（垂直线或阴影区域）
- 事件标签

**应用场景：**
- 验证事件检测准确性
- 检查事件时间分布
- 时段划分前的质量控制

## 时段创建

### epochs_create()

围绕事件创建数据时段（片段）用于事件相关分析。

```python
epochs = nk.epochs_create(data, events, sampling_rate=1000,
                          epochs_start=-0.5, epochs_end=2.0,
                          event_labels=None, event_conditions=None,
                          baseline_correction=False)
```

**参数：**
- `data`：包含信号的DataFrame或单信号
- `events`：事件索引或`events_find()`返回的字典
- `sampling_rate`：信号采样率（Hz）
- `epochs_start`：事件前起始时间（秒，负值表示事件前）
- `epochs_end`：事件后结束时间（秒，正值表示事件后）
- `event_labels`：事件标签列表（可选）
- `event_conditions`：事件条件名称列表（可选）
- `baseline_correction`：为True时从各时段减去基线均值

**返回值：**
- 按时段划分的DataFrame字典
- 每个DataFrame包含事件时间相关的信号数据（索引0为事件起始点）
- 若提供标签/条件，则包含`'Label'`和`'Condition'`列

**典型时段窗口：**
- **视觉诱发电位(ERP)**：-0.2至1.0秒（200ms基线，1秒刺激后）
- **心脏定向反应**：-1.0至10秒（捕捉预期和响应）
- **肌电惊跳反射**：-0.1至0.5秒（短暂响应）
- **皮电SCR反应**：-1.0至10秒（1-3秒延迟，缓慢恢复）

### 事件标签与条件

按类型和实验条件组织事件：

```python
# 示例：情绪图片实验
event_times = [1000, 2500, 4200, 5800]  # 事件起始采样点
event_labels = ['trial1', 'trial2', 'trial3', 'trial4']
event_conditions = ['positive', 'negative', 'positive', 'neutral']

epochs = nk.epochs_create(signals, events=event_times, sampling_rate=1000,
                          epochs_start=-1, epochs_end=5,
                          event_labels=event_labels,
                          event_conditions=event_conditions)
```

**访问时段：**
```python
# 按编号访问
epoch_1 = epochs['1']

# 按条件筛选
positive_epochs = {k: v for k, v in epochs.items() if v['Condition'][0] == 'positive'}
```

### 基线校正

移除刺激前基线以分离事件相关变化：

**自动校正（创建时段时）：**
```python
epochs = nk.epochs_create(data, events, sampling_rate=1000,
                          epochs_start=-0.5, epochs_end=2.0,
                          baseline_correction=True)  # 减去整个基线均值
```

**手动校正（创建时段后）：**
```python
# 减去基线时段均值
baseline_start = -0.5  # 秒
baseline_end = 0.0     # 秒

for key, epoch in epochs.items():
    baseline_mask = (epoch.index >= baseline_start) & (epoch.index < baseline_end)
    baseline_mean = epoch[baseline_mask].mean()
    epochs[key] = epoch - baseline_mean
```

**基线校正适用场景：**
- **诱发电位(ERP)**：必需（分离事件相关变化）
- **心脏/皮电信号**：通常需要（消除个体基线差异）
- **绝对值测量**：有时不需要（如分析绝对振幅）

## 时段分析与可视化

### epochs_plot()

可视化单个或平均时段。

```python
nk.epochs_plot(epochs, column='ECG_Rate', condition=None, show=True)
```

**参数：**
- `column`：待绘制的信号列名
- `condition`：仅绘制特定条件（可选）

**显示内容：**
- 半透明个体时段轨迹
- 时段平均曲线（加粗线）
- 可选：阴影误差带（SEM或SD）

**应用场景：**
- 可视化事件相关响应
- 比较不同条件
- 识别异常时段

### epochs_average()

计算跨时段的总平均及统计量。

```python
average_epochs = nk.epochs_average(epochs, output='dict')
```

**参数：**
- `output`：`'dict'`（默认）或`'df'`（DataFrame格式）

**返回值：**
- 包含以下内容的字典或DataFrame：
  - `'Mean'`：各时间点跨时段均值
  - `'SD'`：标准差
  - `'SE'`：均值标准误
  - `'CI_lower'`，`'CI_upper'`：95%置信区间

**应用场景：**
- 计算事件相关电位(ERP)
- 心脏/皮电/肌电响应总平均
- 组水平分析

**条件特异性平均：**
```python
# 按条件分别平均
positive_epochs = {k: v for k, v in epochs.items() if v['Condition'][0] == 'positive'}
negative_epochs = {k: v for k, v in epochs.items() if v['Condition'][0] == 'negative'}

avg_positive = nk.epochs_average(positive_epochs)
avg_negative = nk.epochs_average(negative_epochs)
```

### epochs_to_df()

将时段字典转换为统一DataFrame。

```python
epochs_df = nk.epochs_to_df(epochs)
```

**返回值：**
- 包含所有时段数据的单一DataFrame
- 含`'Epoch'`，`'Time'`，`'Label'`，`'Condition'`列
- 便于使用pandas/seaborn进行统计分析和绘图

**应用场景：**
- 准备混合效应模型数据
- 使用seaborn/plotly绘图
- 导出至R或其他统计软件

### epochs_to_array()

将时段转换为3D NumPy数组。

```python
epochs_array = nk.epochs_to_array(epochs, column='ECG_Rate')
```

**返回值：**
- 3D数组：（时段数，时间点数，信号列数）

**应用场景：**
- 机器学习输入（时段特征）
- 自定义数组分析
- 基于数组的统计检验

## 信号特异性事件相关分析

NeuroKit2为各类信号提供专用事件相关分析：

### 心电事件相关
```python
ecg_epochs = nk.epochs_create(ecg_signals, events, sampling_rate=1000,
                              epochs_start=-1, epochs_end=10)
ecg_results = nk.ecg_eventrelated(ecg_epochs)
```

**计算指标：**
- `ECG_Rate_Baseline`：事件前心率
- `ECG_Rate_Min/Max`：时段内最小/最大心率
- `ECG_Phase_*`：事件起始时的心动周期相位
- 时间窗内心率动态变化

### 皮电事件相关
```python
eda_epochs = nk.epochs_create(eda_signals, events, sampling_rate=100,
                              epochs_start=-1, epochs_end=10)
eda_results = nk.eda_eventrelated(eda_epochs)
```

**计算指标：**
- `EDA_SCR`：皮电反应存在性（二值）
- `SCR_Amplitude`：最大SCR振幅
- `SCR_Latency`：SCR起始延迟
- `SCR_RiseTime`，`SCR_RecoveryTime`：上升/恢复时间
- `EDA_Tonic`：平均紧张水平

### 呼吸事件相关
```python
rsp_epochs = nk.epochs_create(rsp_signals, events, sampling_rate=100,
                              epochs_start=-0.5, epochs_end=5)
rsp_results = nk.rsp_eventrelated(rsp_epochs)
```

**计算指标：**
- `RSP_Rate_Mean`：平均呼吸频率
- `RSP_Amplitude_Mean`：平均呼吸深度
- `RSP_Phase`：事件起始时的呼吸相位
- 频率/振幅动态变化

### 肌电事件相关
```python
emg_epochs = nk.epochs_create(emg_signals, events, sampling_rate=1000,
                              epochs_start=-0.1, epochs_end=1.0)
emg_results = nk.emg_eventrelated(emg_epochs)
```

**计算指标：**
- `EMG_Activation`：激活存在性
- `EMG_Amplitude_Mean/Max`：振幅统计量
- `EMG_Onset_Latency`：激活起始延迟
- `EMG_Bursts`：激活爆发次数

### 眼电事件相关
```python
eog_epochs = nk.epochs_create(eog_signals, events, sampling_rate=500,
                              epochs_start=-0.5, epochs_end=2.0)
eog_results = nk.eog_eventrelated(eog_epochs)
```

**计算指标：**
- `EOG_Blinks_N`：时段内眨眼次数
- `EOG_Rate_Mean`：平均眨眼频率
- 眨眼时间分布

### 光电容积脉搏波事件相关
```python
ppg_epochs = nk.epochs_create(ppg_signals, events, sampling_rate=100,
                              epochs_start=-1, epochs_end=10)
ppg_results = nk.ppg_eventrelated(ppg_epochs)
```

**计算指标：**
- 类似心电：心率动态、相位信息

## 实践工作流

### 完整事件分析流程

```python
import neurokit2 as nk

# 1. 生理信号预处理
ecg_signals, ecg_info = nk.ecg_process(ecg, sampling_rate=1000)
eda_signals, eda_info = nk.eda_process(eda, sampling_rate=100)

# 2. 对齐采样率（如需要）
eda_signals_resampled = nk.signal_resample(eda_signals, sampling_rate=100,
                                           desired_sampling_rate=1000)

# 3. 合并信号至单一DataFrame
signals = pd.concat([ecg_signals, eda_signals_resampled], axis=1)

# 4. 事件检测
events = nk.events_find(trigger_channel, threshold=0.5)

# 5. 添加事件标签与条件
event_labels = ['trial1', 'trial2', 'trial3', ...]
event_conditions = ['condition_A', 'condition_B', 'condition_A', ...]

# 6. 创建时段
epochs = nk.epochs_create(signals, events, sampling_rate=1000,
                          epochs_start=-1.0, epochs_end=5.0,
                          event_labels=event_labels,
                          event_conditions=event_conditions,
                          baseline_correction=True)

# 7. 信号特异性事件分析
ecg_results = nk.ecg_eventrelated(epochs)
eda_results = nk.eda_eventrelated(epochs)

# 8. 合并结果
results = pd.merge(ecg_results, eda_results, left_index=True, right_index=True)

# 9. 按条件统计分析
results['Condition'] = event_conditions
condition_comparison = results.groupby('Condition').mean()
```

### 多事件类型处理

```python
# 不同标记通道的事件类型
event_type1 = nk.events_find(trigger_ch1, threshold=0.5)
event_type2 = nk.events_find(trigger_ch2, threshold=0.5)

# 带标签合并事件
all_events = np.concatenate([event_type1['onset'], event_type2['onset']])
event_labels = ['type1'] * len(event_type1['onset']) + ['type2'] * len(event_type2['onset'])

# 按时间排序
sort_idx = np.argsort(all_events)
all_events = all_events[sort_idx]
event_labels = [event_labels[i] for i in sort_idx]

# 创建时段
epochs = nk.epochs_create(signals, all_events, sampling_rate=1000,
                          epochs_start=-0.5, epochs_end=3.0,
                          event_labels=event_labels)

# 按类型分离
type1_epochs = {k: v for k, v in epochs.items() if v['Label'][0] == 'type1'}
type2_epochs = {k: v for k, v in epochs.items() if v['Label'][0] == 'type2'}
```

### 质量控制与伪迹剔除

```python
# 剔除噪声或伪迹过大的时段
clean_epochs = {}
for key, epoch in epochs.items():
    # 示例：皮电振幅过高（运动伪迹）则剔除
    if epoch['EDA_Phasic'].abs().max() < 5.0:  # 阈值
        # 示例：心率变化过大（无效数据）则剔除
        if epoch['ECG_Rate'].max() - epoch['ECG_Rate'].min() < 50:
            clean_epochs[key] = epoch

print(f"保留时段: {len(clean_epochs)}/{len(epochs)}")

# 分析有效时段
results = nk.ecg_eventrelated(clean_epochs)
```

## 统计考量

### 样本量
- **ERP/平均分析**：每条件至少20-30+试次
- **单试次分析**：混合效应模型处理可变试次数
- **组间比较**：通过预实验进行功效分析

### 时间窗选择
- **先验假设**：基于文献预先注册时间窗
- **探索性分析**：使用完整时段，需多重比较校正
- **避免**：根据观测数据选择时间窗（循环分析）

### 基线时段
- 应无预期效应干扰
- 足够时长确保稳定估计（典型500-1000毫秒）
- 快速动态响应可缩短（如惊跳反射：100毫秒足够）

### 条件比较
- 重复测量方差分析（被试内设计）
- 混合效应模型（非平衡数据）
- 置换检验（非参数比较）
- 多重比较校正（时间点/信号维度）

## 典型应用场景

**认知心理学：**
- P300诱发电位分析
- 错误相关负波(ERN)
- 注意瞬脱
- 工作记忆负荷效应

**情感神经科学：**
- 情绪图片观看（皮电、心率、面部肌电）
- 恐惧条件反射（心率减速、皮电反应）
- 效价/唤醒维度分析

**临床研究：**
- 惊跳反射（眼轮匝肌肌电）
- 定向反应（心率减速）
- 预期与预测误差

**心理生理学：**
- 心脏防御
