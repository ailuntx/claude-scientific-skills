# 呼吸信号处理

## 概述

NeuroKit2中的呼吸信号处理支持分析呼吸模式、呼吸频率、幅度和变异性。呼吸与心脏活动（呼吸性窦性心律不齐）、情绪状态和认知过程密切相关。

## 主要处理流程

### rsp_process()

通过峰谷检测和特征提取实现呼吸信号的自动化处理。

```python
signals, info = nk.rsp_process(rsp_signal, sampling_rate=100, method='khodadad2018')
```

**处理步骤：**
1. 信号清洗（去噪、滤波）
2. 峰值（呼气）与谷值（吸气）检测
3. 呼吸频率计算
4. 幅度计算
5. 呼吸相位判定（吸气/呼气）
6. 单位时间呼吸容积（RVT）

**返回：**
- `signals`：包含以下字段的DataFrame：
  - `RSP_Clean`：滤波后的呼吸信号
  - `RSP_Peaks`、`RSP_Troughs`：极值标记点
  - `RSP_Rate`：瞬时呼吸频率（次/分钟）
  - `RSP_Amplitude`：逐次呼吸幅度
  - `RSP_Phase`：吸气（0）与呼气（1）相位
  - `RSP_Phase_Completion`：相位完成百分比（0-1）
  - `RSP_RVT`：单位时间呼吸容积
- `info`：包含峰谷索引的字典

**方法：**
- `'khodadad2018'`：Khodadad等人算法（默认，鲁棒性强）
- `'biosppy'`：基于BioSPPy的处理（替代方案）

## 预处理函数

### rsp_clean()

去除噪声并平滑呼吸信号。

```python
cleaned_rsp = nk.rsp_clean(rsp_signal, sampling_rate=100, method='khodadad2018')
```

**方法：**

**1. Khodadad2018（默认）：**
- 巴特沃斯低通滤波
- 去除高频噪声
- 保留呼吸波形特征

**2. BioSPPy：**
- 替代滤波方案
- 性能与Khodadad相当

**3. Hampel滤波器：**
```python
cleaned_rsp = nk.rsp_clean(rsp_signal, sampling_rate=100, method='hampel')
```
- 基于中位数的离群值剔除
- 抗伪影和尖峰干扰
- 保留信号突变特征

**典型呼吸频率：**
- 成人静息状态：12-20次/分钟（0.2-0.33 Hz）
- 儿童：频率更高
- 运动状态：可达40-60次/分钟

### rsp_peaks()

识别呼吸信号中的吸气谷值与呼气峰值。

```python
peaks, info = nk.rsp_peaks(cleaned_rsp, sampling_rate=100, method='khodadad2018')
```

**检测方法：**
- `'khodadad2018'`：针对清洁信号优化
- `'biosppy'`：替代方案
- `'scipy'`：基于Scipy的简易检测

**返回：**
- 包含以下字段的字典：
  - `RSP_Peaks`：呼气峰值索引（最高点）
  - `RSP_Troughs`：吸气谷值索引（最低点）

**呼吸周期定义：**
- **吸气**：谷值→峰值（气流进入，胸腹扩张）
- **呼气**：峰值→谷值（气流排出，胸腹收缩）

### rsp_findpeaks()

支持多算法的底层峰值检测。

```python
peaks_dict = nk.rsp_findpeaks(cleaned_rsp, sampling_rate=100, method='scipy')
```

**方法：**
- `'scipy'`：Scipy的find_peaks函数
- 自定义阈值算法

**适用场景：**
- 精细化峰值检测
- 自定义参数调整
- 算法对比

### rsp_fixpeaks()

修正检测到的峰谷异常（如漏检或误检）。

```python
corrected_peaks = nk.rsp_fixpeaks(peaks, sampling_rate=100)
```

**修正策略：**
- 剔除生理学不可信的间隔
- 插值补全缺失峰值
- 移除伪影相关的假峰

## 特征提取函数

### rsp_rate()

计算瞬时呼吸频率（次/分钟）。

```python
rate = nk.rsp_rate(peaks, sampling_rate=100, desired_length=None)
```

**方法：**
- 根据峰谷时间计算呼吸间隔
- 转换为次/分钟（BPM）
- 插值至信号长度

**典型值：**
- 成人静息：12-20 BPM
- 慢呼吸：<10 BPM（冥想、放松状态）
- 快呼吸：>25 BPM（运动、焦虑状态）

### rsp_amplitude()

计算逐次呼吸幅度（峰谷差值）。

```python
amplitude = nk.rsp_amplitude(cleaned_rsp, peaks)
```

**解读：**
- 幅度增大：呼吸加深（潮气量增加）
- 幅度减小：浅呼吸
- 幅度多变：不规则呼吸模式

**临床意义：**
- 幅度降低：限制性肺病、胸壁僵直
- 幅度升高：代偿性过度通气

### rsp_phase()

判定吸气/呼气相位及完成度百分比。

```python
phase, completion = nk.rsp_phase(cleaned_rsp, peaks, sampling_rate=100)
```

**返回：**
- `RSP_Phase`：二值化（0=吸气，1=呼气）
- `RSP_Phase_Completion`：0-1连续值表示相位进度

**应用场景：**
- 呼吸门控刺激呈现
- 相位锁定平均分析
- 呼吸-心脏耦合研究

### rsp_symmetry()

分析呼吸对称模式（峰谷平衡性，升降时间）。

```python
symmetry = nk.rsp_symmetry(cleaned_rsp, peaks)
```

**指标：**
- 峰谷对称性：峰值与谷值是否等距？
- 升降对称性：吸气时间是否等于呼气时间？

**解读：**
- 对称：正常放松呼吸
- 不对称：费力呼吸，气道阻塞

## 高级分析函数

### rsp_rrv()

呼吸频率变异性——类比心率变异性。

```python
rrv_indices = nk.rsp_rrv(peaks, sampling_rate=100)
```

**时域指标：**
- `RRV_SDBB`：呼吸间隔标准差
- `RRV_RMSSD`：连续差值的均方根
- `RRV_MeanBB`：平均呼吸间隔

**频域指标：**
- 频段功率（如适用）

**解读：**
- RRV升高：灵活自适应的呼吸调控
- RRV降低：僵化的呼吸模式
- RRV异常：焦虑、呼吸系统疾病、自主神经功能障碍

**记录时长要求：**
- 最低：2-3分钟
- 最优：5-10分钟（稳定估计）

### rsp_rvt()

单位时间呼吸容积——fMRI混杂回归因子。

```python
rvt = nk.rsp_rvt(cleaned_rsp, peaks, sampling_rate=100)
```

**计算原理：**
- 呼吸信号的导数
- 捕捉容积变化速率
- 与BOLD信号波动相关

**应用场景：**
- fMRI伪影校正
- 神经影像预处理
- 呼吸混杂因素回归

**参考文献：**
- Birn, R. M., et al. (2008). Separating respiratory-variation-related fluctuations from neuronal-activity-related fluctuations in fMRI. NeuroImage, 31(4), 1536-1548.

### rsp_rav()

呼吸幅度变异性指标。

```python
rav = nk.rsp_rav(amplitude, sampling_rate=100)
```

**指标：**
- 幅度标准差
- 变异系数
- 幅度范围

**解读：**
- RAV高：呼吸深度不规则（叹息、觉醒变化）
- RAV低：稳定可控的呼吸

## 分析函数

### rsp_analyze()

自动选择事件相关或区间相关分析模式。

```python
analysis = nk.rsp_analyze(signals, sampling_rate=100)
```

**模式选择：**
- 时长<10秒 → 事件相关分析
- 时长≥10秒 → 区间相关分析

### rsp_eventrelated()

分析特定事件/刺激引发的呼吸响应。

```python
results = nk.rsp_eventrelated(epochs)
```

**计算指标（每周期）：**
- `RSP_Rate_Mean`：周期内平均呼吸频率
- `RSP_Rate_Min/Max`：最小/最大频率
- `RSP_Amplitude_Mean`：平均呼吸深度
- `RSP_Phase`：事件起始时的呼吸相位
- 周期内频率与幅度的动态变化

**应用场景：**
- 情绪刺激下的呼吸变化
- 任务事件前的预期性呼吸调整
- 屏息或过度通气实验范式

### rsp_intervalrelated()

分析长时程呼吸记录。

```python
results = nk.rsp_intervalrelated(signals, sampling_rate=100)
```

**计算指标：**
- `RSP_Rate_Mean`：平均呼吸频率
- `RSP_Rate_SD`：频率变异性
- `RSP_Amplitude_Mean`：平均呼吸深度
- RRV指标（数据充足时）
- RAV指标

**记录时长要求：**
- 最低：60秒
- 最优：5-10分钟

**应用场景：**
- 静息态呼吸模式
- 基线呼吸评估
- 压力或放松状态监测

## 模拟与可视化

### rsp_simulate()

生成测试用合成呼吸信号。

```python
synthetic_rsp = nk.rsp_simulate(duration=60, sampling_rate=100, respiratory_rate=15,
                                method='sinusoidal', noise=0.1, random_state=42)
```

**方法：**
- `'sinusoidal'`：简易正弦振荡（快速）
- `'breathmetrics'`：高级真实呼吸模型（较慢，更精确）

**参数：**
- `respiratory_rate`：呼吸频率（次/分钟，默认15）
- `noise`：高斯噪声水平
- `random_state`：随机种子（可复现性）

**应用场景：**
- 算法验证
- 参数调优
- 教学演示

### rsp_plot()

可视化处理后的呼吸信号。

```python
nk.rsp_plot(signals, info, static=True)
```

**显示内容：**
- 原始与清洗后的呼吸信号
- 检测到的峰谷位置
- 瞬时呼吸频率
- 相位标记

**交互模式：** 设置`static=False`启用Plotly交互可视化

## 实践注意事项

### 采样率建议
- **最低**：10 Hz（满足频率估计）
- **标准**：50-100 Hz（研究级）
- **高分辨率**：1000 Hz（通常冗余）

### 记录时长
- **频率估计**：≥10秒（数次呼吸）
- **RRV分析**：≥2-3分钟
- **静息态**：5-10分钟
- **昼夜节律**：数小时至数天

### 信号采集方法

**应变仪/压电传感器腰带：**
- 胸腹扩张测量
- 最常用方案
- 舒适无创

**热敏电阻/热电偶：**
- 鼻腔/口腔气流温度监测
- 直接气流测量
- 可能造成不适

**二氧化碳浓度监测：**
- 潮气末CO₂测量
- 生理学金标准
- 成本高，适用于临床

**阻抗式呼吸描记：**
- 源自心电电极
- 多模态记录便捷
- 精度低于专用传感器

### 常见问题与解决方案

**呼吸不规则：**
- 清醒静息状态属正常现象
- 叹息、哈欠、说话、吞咽导致变异
- 剔除伪影或建模为独立事件

**浅呼吸：**
- 信号幅度过低
- 检查传感器位置与松紧度
- 增加增益（若支持）

**运动伪影：**
- 尖峰或信号中断
- 减少受试者移动
- 采用鲁棒峰值检测（Hampel滤波）

**说话/咳嗽：**
- 破坏自然呼吸模式
- 标注并排除分析
- 或建模为独立事件类型

### 最佳实践

**标准工作流：**
```python
# 1. 清洗信号
cleaned = nk.rsp_clean(rsp_raw, sampling_rate=100, method='khodadad2018')

# 2. 检测峰谷
peaks, info = nk.rsp_peaks(cleaned, sampling_rate=100)

# 3. 提取特征
rate = nk.rsp_rate(peaks, sampling_rate=100, desired_length=len(cleaned))
amplitude = nk.rsp_amplitude(cleaned, peaks)
phase = nk.rsp_phase(cleaned, peaks, sampling_rate=100)

# 4. 综合处理（替代方案）
signals, info = nk.rsp_process(rsp_raw, sampling_rate=100)

# 5. 分析
analysis = nk.rsp_analyze(signals, sampling_rate=100)
```

**呼吸-心脏信号整合：**
```python
# 并行处理双信号
ecg_signals, ecg_info = nk.ecg_process(ecg, sampling_rate=1000)
rsp_signals, rsp_info = nk.rsp_process(rsp, sampling_rate=100)

# 呼吸性窦性心律不齐（RSA）
rsa = nk.hrv_rsa(ecg_info['ECG_R_Peaks'], rsp_signals['RSP_Clean'], sampling_rate=1000)

# 或使用bio_process实现多信号整合
bio_signals, bio_info = nk.bio_process(ecg=ecg, rsp=rsp, sampling_rate=1000)
```

## 临床与科研应用

**心理生理学：**
- 情绪与唤醒（压力下的快速浅呼吸）
- 放松干预（缓慢深呼吸）
- 呼吸生物反馈

**焦虑与惊恐障碍：**
- 惊恐发作时的过度通气
- 异常呼吸模式
- 呼吸再训练疗法效果评估

**睡眠医学：**
- 睡眠呼吸暂停检测
- 呼吸模式异常
- 睡眠阶段相关性

**心肺耦合：**
- 呼吸性窦性心律不齐（呼吸对HRV的调制）
- 心肺交互作用
- 自主神经系统评估

**神经影像：**
- fMRI伪影校正（RVT回归因子）
- BOLD信号混杂去除
- 呼吸相关脑活动研究

**冥想与正念：**
- 呼吸觉知训练
- 慢呼吸练习（共振频率约6次/分钟）
- 放松状态的生理标志物

**运动表现：**
- 呼吸效率优化
- 训练适应性评估
- 恢复状态监测

## 解读指南

**呼吸频率：**
- **正常**：12-20 BPM（成人静息）
- **过缓**：<10 BPM（放松、冥想、睡眠）
- **过速**：>25 BPM（运动、焦虑、疼痛、发热）

**呼吸幅度：**
- 静息潮气量通常400-600 mL
- 深呼吸：2-3 L
- 浅呼吸：<300 mL

**呼吸模式：**
- **正常**：平滑规则的正弦波
- **潮式呼吸**：渐强渐弱伴呼吸暂停（临床病理）
- **共济失调式**：完全无规律（脑干损伤）

## 参考文献

- Khodadad, D., Nordebo, S., Müller, B., Waldmann, A., Yerworth, R., Becher, T., ... & Bayford, R. (2018). A review of tissue substitutes for ultrasound imaging. Ultrasound in medicine & biology, 44(9), 1807-1823.
- Grossman, P., & Taylor, E. W. (2007). Toward understanding respiratory sinus arrhythmia: Relations to cardiac vagal tone, evolution and biobehavioral functions. Biological psychology, 74(2), 263-285.
- Birn, R. M., Diamond, J. B., Smith, M. A., & Bandettini, P. A. (2006). Separating respiratory-variation-related fluctuations from neuronal-activity-related fluctuations in fMRI. NeuroImage, 31(4), 1536-1548.
