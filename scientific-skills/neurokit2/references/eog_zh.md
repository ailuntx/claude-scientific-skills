# 眼电图（EOG）分析

## 概述

眼电图（EOG）通过检测眼球位置变化产生的电位差来测量眼球运动和眨眼。EOG应用于睡眠研究、注意力研究、阅读分析以及EEG伪迹校正。

## 主要处理流程

### eog_process()

自动化EOG信号处理流程。

```python
signals, info = nk.eog_process(eog_signal, sampling_rate=500, method='neurokit')
```

**处理步骤：**
1. 信号清洗（滤波）
2. 眨眼检测
3. 眨眼频率计算

**返回：**
- `signals`：包含以下字段的DataFrame：
  - `EOG_Clean`：滤波后的EOG信号
  - `EOG_Blinks`：二进制眨眼标记（0/1）
  - `EOG_Rate`：瞬时眨眼频率（次/分钟）
- `info`：包含眨眼索引和参数的字典

**方法：**
- `'neurokit'`：NeuroKit2优化方法（默认）
- `'agarwal2019'`：Agarwal等人(2019)算法
- `'mne'`：MNE-Python方法
- `'brainstorm'`：Brainstorm工具箱方法
- `'kong1998'`：Kong等人(1998)方法

## 预处理函数

### eog_clean()

为眨眼检测准备原始EOG信号。

```python
cleaned_eog = nk.eog_clean(eog_signal, sampling_rate=500, method='neurokit')
```

**方法：**
- `'neurokit'`：针对EOG优化的巴特沃斯滤波
- `'agarwal2019'`：替代滤波方法
- `'mne'`：MNE-Python预处理
- `'brainstorm'`：Brainstorm方法
- `'kong1998'`：Kong方法

**典型滤波：**
- 低通：10-20 Hz（去除高频噪声）
- 高通：0.1-1 Hz（去除直流漂移）
- 保留眨眼波形（典型持续时间100-400 ms）

**EOG信号特征：**
- **眨眼**：大幅值，固定波形（200-400 ms）
- **扫视**：快速阶跃式偏转（20-80 ms）
- **平滑追踪**：缓慢斜坡式变化
- **基线**：眼球固定时保持稳定

## 眨眼检测

### eog_peaks()

检测EOG信号中的眨眼。

```python
blinks, info = nk.eog_peaks(cleaned_eog, sampling_rate=500, method='neurokit',
                            threshold=0.33)
```

**方法：**
- `'neurokit'`：幅值和持续时间标准（默认）
- `'mne'`：MNE-Python眨眼检测
- `'brainstorm'`：Brainstorm方法
- `'blinker'`：BLINKER算法（Kleifges等人，2017）

**关键参数：**
- `threshold`：幅值阈值（最大幅值的比例）
  - 典型值：0.2-0.5
  - 较低值：更敏感（可能包含假阳性）
  - 较高值：更保守（可能遗漏小幅眨眼）

**返回：**
- 包含眨眼峰值索引的字典（键为`'EOG_Blinks'`）

**眨眼特征：**
- **频率**：15-20次/分钟（静息舒适状态）
- **持续时间**：100-400 ms（平均约200 ms）
- **幅值**：随电极位置和个体差异变化
- **波形**：双相或三相

### eog_findpeaks()

支持多算法的底层眨眼检测。

```python
blinks_dict = nk.eog_findpeaks(cleaned_eog, sampling_rate=500, method='neurokit')
```

**应用场景：**
- 自定义参数调整
- 算法比较
- 研究方法开发

## 特征提取

### eog_features()

提取单次眨眼特征。

```python
features = nk.eog_features(signals, sampling_rate=500)
```

**计算特征：**
- **幅值速度比（AVR）**：峰值速度/幅值
  - 区分眨眼与伪迹
- **眨眼幅值比**：眨眼幅值一致性
- **持续时间指标**：眨眼持续时间统计量（均值，标准差）
- **峰值幅值**：最大偏转量
- **峰值速度**：最大变化率

**应用场景：**
- 眨眼质量评估
- 困倦检测（困倦时眨眼持续时间增加）
- 神经学评估（疾病中的眨眼动态变化）

### eog_rate()

计算眨眼频率（次/分钟）。

```python
blink_rate = nk.eog_rate(blinks, sampling_rate=500, desired_length=None)
```

**方法：**
- 计算眨眼间隔
- 转换为次/分钟
- 插值匹配信号长度

**典型眨眼频率：**
- **静息状态**：15-20次/分钟
- **阅读/视觉任务**：5-10次/分钟（受抑制）
- **对话状态**：20-30次/分钟
- **压力/眼干**：>30次/分钟
- **困倦状态**：频率不定，眨眼时间延长

## 分析函数

### eog_analyze()

自动选择事件相关或区间相关分析。

```python
analysis = nk.eog_analyze(signals, sampling_rate=500)
```

**模式选择：**
- 时长<10秒 → 事件相关分析
- 时长≥10秒 → 区间相关分析

### eog_eventrelated()

分析特定事件相关的眨眼模式。

```python
results = nk.eog_eventrelated(epochs)
```

**计算指标（每时段）：**
- `EOG_Blinks_N`：时段内眨眼次数
- `EOG_Rate_Mean`：平均眨眼频率
- `EOG_Blink_Presence`：二元值（是否发生眨眼）
- 时段内眨眼时间分布

**应用场景：**
- 眨眼锁定的ERP污染评估
- 刺激期间的注意力和参与度
- 视觉任务难度（高要求任务中眨眼受抑制）
- 刺激结束后的自发眨眼

### eog_intervalrelated()

分析长时程眨眼模式。

```python
results = nk.eog_intervalrelated(signals, sampling_rate=500)
```

**计算指标：**
- `EOG_Blinks_N`：总眨眼次数
- `EOG_Rate_Mean`：平均眨眼频率（次/分钟）
- `EOG_Rate_SD`：眨眼频率变异性
- `EOG_Duration_Mean`：平均眨眼持续时间（如可用）
- `EOG_Amplitude_Mean`：平均眨眼幅值（如可用）

**应用场景：**
- 静息态眨眼模式
- 困倦或疲劳监测（持续时间增加）
- 持续注意力任务（频率受抑制）
- 干眼症评估（频率增加，不完全眨眼）

## 模拟与可视化

### eog_plot()

可视化处理后的EOG信号及检测到的眨眼。

```python
nk.eog_plot(signals, info)
```

**显示内容：**
- 原始与清洗后的EOG信号
- 检测到的眨眼标记
- 眨眼频率时间序列

## 实践注意事项

### 采样率建议
- **最低**：100 Hz（基础眨眼检测）
- **标准**：250-500 Hz（研究应用）
- **高分辨率**：1000 Hz（详细波形分析，扫视）
- **睡眠研究**：典型200-250 Hz

### 记录时长
- **眨眼检测**：任意时长（≥1次眨眼）
- **眨眼频率估计**：≥60秒以获得稳定估计
- **事件相关**：取决于实验范式（单次试验秒数）
- **睡眠EOG**：小时级（整夜）

### 电极放置

**标准配置：**

**水平EOG（HEOG）：**
- 双电极：左右眼外眦（外眼角）
- 测量水平眼球运动（扫视，平滑追踪）
- 双极记录（左-右）

**垂直EOG（VEOG）：**
- 双电极：单眼（通常右眼）上下方
- 测量垂直眼球运动和眨眼
- 双极记录（上-下）

**睡眠EOG：**
- 常采用不同位置（颞区）
- E1：左眼外眦外侧1cm，下方1cm
- E2：右眼外眦外侧1cm，上方1cm
- 同时捕获水平和垂直运动

**EEG污染去除：**
- 额叶电极（Fp1, Fp2）可作为EOG代理
- EEG预处理中常用基于ICA的EOG伪迹去除

### 常见问题与解决方案

**电极问题：**
- 接触不良：低幅值，噪声
- 皮肤准备：清洁，轻度打磨
- 导电膏：确保良好接触

**伪迹：**
- 肌肉活动（尤其额肌）：高频噪声
- 运动：线缆伪迹，头部移动
- 电噪声：50/60 Hz交流声（正确接地）

**饱和：**
- 大幅扫视可能导致放大器饱和
- 调整增益或电压范围
- 低分辨率系统更常见

### 最佳实践

**标准工作流：**
```python
# 1. 清洗信号
cleaned = nk.eog_clean(eog_raw, sampling_rate=500, method='neurokit')

# 2. 检测眨眼
blinks, info = nk.eog_peaks(cleaned, sampling_rate=500, method='neurokit')

# 3. 提取特征
features = nk.eog_features(signals, sampling_rate=500)

# 4. 综合处理（替代方案）
signals, info = nk.eog_process(eog_raw, sampling_rate=500)

# 5. 分析
analysis = nk.eog_analyze(signals, sampling_rate=500)
```

**EEG伪迹校正工作流：**
```python
# 方案1：基于回归的去除
# 从清洗后的EOG信号识别EOG成分
# 从EEG通道回归去除EOG

# 方案2：基于ICA的去除（推荐）
# 1. 对含EOG通道的EEG数据运行ICA
# 2. 识别与EOG相关的ICA成分
# 3. 从EEG数据中移除EOG成分
# NeuroKit2可与MNE集成实现此流程
```

## 临床与研究应用

**EEG伪迹校正：**
- 眨眼污染额叶EEG通道
- ICA或回归法去除EOG伪迹
- ERP研究必备

**睡眠分期：**
- REM睡眠期的快速眼动（REMs）
- 困倦期的缓慢滚动眼动
- 睡眠起始与阶段转换

**注意力与认知负荷：**
- 高要求任务中眨眼频率受抑制
- 眨眼聚集在任务边界（自然断点）
- 自发眨眼作为注意力转移指标

**疲劳与困倦监测：**
- 困倦时眨眼持续时间增加
- 眼睑闭合速度变慢
- 部分或不完全眨眼
- 驾驶员监测应用

**阅读与视觉处理：**
- 阅读时眨眼受抑制
- 扫视期间的眼球运动（换行）
- 疲劳对阅读效率的影响

**神经疾病：**
- **帕金森病**：自发眨眼频率降低
- **精神分裂症**：眨眼频率增加
- **图雷特综合征**：过度眨眼（抽动）
- **干眼综合征**：频率增加，不完全眨眼

**情感与社会认知：**
- 社交互动中的眨眼同步性
- 眨眼频率的情绪调节
- ERP中的眨眼相关电位

**人机交互：**
- 视线追踪预处理
- 注意力监测
- 用户参与度评估

## EOG可检测的眼球运动类型

**眨眼：**
- 大幅值，短时程（100-400 ms）
- NeuroKit2主要关注点
- 垂直EOG最敏感

**扫视：**
- 快速弹道式眼球运动（20-80 ms）
- 阶跃式电压偏转
- 水平或垂直方向
- 详细分析需更高采样率

**平滑追踪：**
- 对移动物体的缓慢跟踪
- 斜坡式电压变化
- 幅值低于扫视

**注视：**
- 稳定凝视
- 基线EOG伴小幅振荡
- 持续时间不定（阅读中典型200-600 ms）

**注意：** 详细的扫视/注视分析通常需要眼动仪（红外，视频式）。EOG适用于眨眼和宏观眼球运动。

## 解读指南

**眨眼频率：**
- **正常静息**：15-20次/分钟
- **<10次/分钟**：视觉任务投入，专注状态
- **>30次/分钟**：压力，眼干，疲劳
- **依赖情境**：任务需求，光照，屏幕使用

**眨眼持续时间：**
- **正常**：100-400 ms（平均约200 ms）
- **延长**：困倦，疲劳（>500 ms）
- **缩短**：正常警觉状态

**眨眼幅值：**
- 随电极位置和个体差异变化
- 个体内比较最可靠
- 不完全眨眼：幅值降低（眼干，疲劳）

**时间模式：**
- **聚集眨眼**：任务或认知状态转换期
- **受抑眨眼**：主动视觉处理，持续注意
- **刺激后眨眼**：视觉处理完成后

## 参考文献

- Kleifges, K., Bigdely-Shamlo, N., Kerick, S. E., & Robbins, K. A. (2017). BLINKER: Automated extraction of ocular indices from EEG enabling large-scale analysis. Frontiers in Neuroscience, 11, 12.
- Agarwal, M., & Sivakumar, R. (2019). Blink: A fully automated unsupervised algorithm for eye-blink detection in EEG signals. In 2019 57th Annual Allerton Conference on Communication, Control, and Computing (pp. 1113-1121). IEEE.
- Kong, X., & Wilson, G. F. (1998). A new EOG-based eyeblink detection algorithm. Behavior Research Methods, Instruments, & Computers, 30(4), 713-719.
- Schleicher, R., Galley, N., Briest, S., & Galley, L. (2008). Blinks and saccades as indicators of fatigue in sleepiness warnings: Looking tired? Ergonomics, 51(7), 982-1010.
