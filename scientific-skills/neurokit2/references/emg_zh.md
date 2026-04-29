# 肌电图（EMG）分析

## 概述

肌电图（EMG）测量骨骼肌收缩时产生的电活动。NeuroKit2中的EMG分析聚焦于振幅估计、肌肉激活检测和时间动态特征，服务于心理生理学与运动控制研究。

## 主要处理流程

### emg_process()

自动化EMG信号处理流程。

```python
signals, info = nk.emg_process(emg_signal, sampling_rate=1000)
```

**处理步骤：**
1. 信号清洗（高通滤波、去趋势化）
2. 振幅包络提取
3. 肌肉激活检测
4. 起始点与终止点识别

**返回：**
- `signals`：包含以下字段的DataFrame：
  - `EMG_Clean`：滤波后EMG信号
  - `EMG_Amplitude`：线性包络（平滑整流信号）
  - `EMG_Activity`：二值激活指示器（0/1）
  - `EMG_Onsets`：激活起始标记
  - `EMG_Offsets`：激活终止标记
- `info`：包含激活参数的字典

**典型工作流：**
- 处理原始EMG → 提取振幅 → 检测激活 → 分析特征

## 预处理函数

### emg_clean()

应用滤波去除噪声，为振幅提取做准备。

```python
cleaned_emg = nk.emg_clean(emg_signal, sampling_rate=1000)
```

**滤波方法（BioSPPy方法）：**
- 四阶巴特沃斯高通滤波器（100 Hz）
- 去除低频运动伪影和基线漂移
- 消除直流偏移
- 信号去趋势化

**原理：**
- EMG频率范围：20-500 Hz（主频：50-150 Hz）
- 100 Hz高通滤波可分离肌肉活动
- 消除ECG干扰（尤其在躯干肌肉）
- 消除运动伪影（<20 Hz）

**EMG信号特性：**
- 收缩期呈随机零均值振荡
- 振幅越高表示收缩越强
- 原始EMG：同时存在正负偏转

## 特征提取

### emg_amplitude()

计算代表肌肉收缩强度的线性包络。

```python
amplitude = nk.emg_amplitude(cleaned_emg, sampling_rate=1000)
```

**方法：**
1. 全波整流（取绝对值）
2. 低通滤波（平滑包络）
3. 降采样（可选）

**线性包络：**
- 跟随EMG振幅调制的平滑曲线
- 表征肌肉力量/激活水平
- 适用于后续分析（激活检测、积分）

**典型平滑处理：**
- 低通滤波器：10-20 Hz截止频率
- 移动平均：50-200 ms窗口
- 平衡点：响应性与平滑度

## 激活检测

### emg_activation()

检测肌肉激活时段（起始点与终止点）。

```python
activity, info = nk.emg_activation(emg_amplitude, sampling_rate=1000, method='threshold',
                                   threshold='auto', duration_min=0.05)
```

**方法：**

**1. 基于阈值（默认）：**
```python
activity = nk.emg_activation(amplitude, method='threshold', threshold='auto')
```
- 比较振幅与阈值
- `threshold='auto'`：基于信号统计自动设定（如均值+1标准差）
- `threshold=0.1`：手动绝对阈值
- 简单、快速、广泛使用

**2. 高斯混合模型（GMM）：**
```python
activity = nk.emg_activation(amplitude, method='mixture', n_clusters=2)
```
- 无监督聚类：激活态 vs 静息态
- 自适应信号特性
- 对基线变化更具鲁棒性

**3. 变点检测：**
```python
activity = nk.emg_activation(amplitude, method='changepoint')
```
- 检测信号特性的突变
- 识别激活/失活点
- 适用于复杂时间模式

**4. 双峰法（Silva等, 2013）：**
```python
activity = nk.emg_activation(amplitude, method='bimodal')
```
- 检验双峰分布（激活态 vs 静息态）
- 确定最优分离阈值
- 基于统计原理

**关键参数：**
- `duration_min`：最小激活时长（秒）
  - 过滤短暂伪激活
  - 典型值：50-100 ms
- `threshold`：激活阈值（取决于方法）

**返回：**
- `activity`：二值数组（0=静息, 1=激活）
- `info`：包含起始/终止索引的字典

**激活指标：**
- **起始点**：从静息到激活的转变
- **终止点**：从激活到静息的转变
- **持续时间**：起始点与终止点间时长
- **爆发**：单次连续激活时段

## 分析函数

### emg_analyze()

自动选择事件相关或区间相关分析。

```python
analysis = nk.emg_analyze(signals, sampling_rate=1000)
```

**模式选择：**
- 时长 < 10秒 → 事件相关分析
- 时长 ≥ 10秒 → 区间相关分析

### emg_eventrelated()

分析离散事件/刺激引发的EMG响应。

```python
results = nk.emg_eventrelated(epochs)
```

**计算指标（每周期）：**
- `EMG_Activation`：激活存在性（二值）
- `EMG_Amplitude_Mean`：周期内平均振幅
- `EMG_Amplitude_Max`：峰值振幅
- `EMG_Bursts`：激活爆发次数
- `EMG_Onset_Latency`：事件至首次激活的延迟（如适用）

**应用场景：**
- 惊吓反应（眼轮匝肌EMG）
- 情绪刺激下面部EMG（皱眉肌、颧大肌）
- 运动反应延迟
- 肌肉反应范式

### emg_intervalrelated()

分析长时程EMG记录。

```python
results = nk.emg_intervalrelated(signals, sampling_rate=1000)
```

**计算指标：**
- `EMG_Bursts_N`：激活爆发总次数
- `EMG_Amplitude_Mean`：整个区间平均振幅
- `EMG_Activation_Duration`：激活状态总时长
- `EMG_Rest_Duration`：静息状态总时长

**应用场景：**
- 静息肌张力评估
- 慢性疼痛或压力相关肌肉活动
- 持续任务中的疲劳监测
- 姿势肌评估

## 模拟与可视化

### emg_simulate()

生成合成EMG信号用于测试。

```python
synthetic_emg = nk.emg_simulate(duration=10, sampling_rate=1000, burst_number=3,
                                noise=0.1, random_state=42)
```

**参数：**
- `burst_number`：包含的激活爆发次数
- `noise`：背景噪声水平
- `random_state`：可复现性种子

**生成特征：**
- 爆发期随机EMG类振荡
- 符合实际的频率成分
- 可变的爆发时序与振幅

**应用场景：**
- 算法验证
- 检测参数调优
- 教学演示

### emg_plot()

可视化处理后的EMG信号。

```python
nk.emg_plot(signals, info, static=True)
```

**显示内容：**
- 原始与清洗后EMG信号
- 振幅包络
- 检测到的激活时段
- 起始/终止标记

**交互模式**：设置`static=False`启用Plotly可视化

## 实践注意事项

### 采样率建议
- **最低**：500 Hz（250 Hz上限频率的奈奎斯特要求）
- **标准**：1000 Hz（多数研究应用）
- **高分辨率**：2000-4000 Hz（运动单元细节研究）
- **表面EMG**：典型1000-2000 Hz
- **肌内EMG**：单运动单元研究需10000+ Hz

### 记录时长
- **事件相关**：取决于范式（如每试次2-5秒）
- **持续收缩**：数秒至数分钟
- **疲劳研究**：数分钟至数小时
- **长期监测**：数天（可穿戴EMG）

### 电极放置

**表面EMG（最常用）：**
- 双极配置（肌肉腹部两个电极）
- 参考/接地电极置于电中性位点（骨骼）
- 皮肤准备：清洁、打磨、降低阻抗
- 电极间距：10-20 mm（SENIAM标准）

**肌肉特异性指南：**
- 遵循SENIAM（肌肉无创评估表面EMG）建议
- 收缩时触诊定位肌肉腹部
- 电极沿肌纤维方向排列

**心理生理学常用肌肉：**
- **皱眉肌**：皱眉、负性情绪（眉上方）
- **颧大肌**：微笑、正性情绪（面颊）
- **眼轮匝肌**：惊吓反应、恐惧（眼周）
- **咬肌**：咬牙、压力（颌部肌肉）
- **斜方肌**：肩部紧张、压力（上背部）
- **额肌**：前额紧张、惊讶

### 信号质量问题

**ECG干扰：**
- 常见于躯干及近端肌肉
- 高通滤波（>100 Hz）通常有效
- 持续干扰时：模板减法、ICA

**运动伪影：**
- 低频干扰
- 电极导线移动
- 固定电极，减少导线运动

**电极问题：**
- 接触不良：高阻抗、低振幅
- 汗液：振幅渐增、不稳定
- 毛发：清洁或剃除区域

**串扰：**
- 邻近肌肉活动渗入记录
- 谨慎放置电极
- 缩小电极间距

### 最佳实践

**标准工作流：**
```python
# 1. 清洗信号（高通滤波、去趋势）
cleaned = nk.emg_clean(emg_raw, sampling_rate=1000)

# 2. 提取振幅包络
amplitude = nk.emg_amplitude(cleaned, sampling_rate=1000)

# 3. 检测激活时段
activity, info = nk.emg_activation(amplitude, sampling_rate=1000,
                                   method='threshold', threshold='auto')

# 4. 综合处理（替代方案）
signals, info = nk.emg_process(emg_raw, sampling_rate=1000)

# 5. 分析
analysis = nk.emg_analyze(signals, sampling_rate=1000)
```

**归一化：**
```python
# 最大自主收缩（MVC）归一化
mvc_amplitude = np.max(mvc_emg_amplitude)  # 来自独立MVC试验
normalized_emg = (amplitude / mvc_amplitude) * 100  # 以% MVC表示

# 常用于工效学、运动生理学
# 支持跨个体和跨会话比较
```

## 临床与研究应用

**心理生理学：**
- **面部EMG**：情绪效价（微笑 vs 皱眉）
- **惊吓反应**：恐惧、惊讶、防御性反应
- **压力**：慢性肌紧张（斜方肌、咬肌）

**运动控制与康复：**
- 步态分析
- 运动障碍（震颤、肌张力障碍）
- 中风康复（肌肉再激活）
- 假肢控制（肌电）

**工效学与职业健康：**
- 职业性肌肉骨骼疾病
- 姿势评估
- 重复性劳损风险

**运动科学：**
- 运动中的肌肉激活模式
- 疲劳评估（中位频率偏移）
- 训练优化

**生物反馈：**
- 放松训练（降低肌紧张）
- 神经肌肉再教育
- 慢性疼痛管理

**睡眠医学：**
- 颏肌EMG检测REM睡眠失张力
- 周期性肢体运动
- 磨牙症

## 高级EMG分析（超越NeuroKit2基础功能）

**频域分析：**
- 疲劳期的中位频率偏移
- 功率谱分析
- 需较长片段（≥1秒/分析窗口）

**运动单元识别：**
- 肌内EMG
- 尖峰检测与分类
- 放电频率分析
- 需高采样率（10+ kHz）

**肌肉协调：**
- 共同收缩指数
- 协同作用分析
- 多肌肉整合

## 解读指南

**振幅（线性包络）：**
- 振幅越高 ≈ 收缩越强（非线性关系）
- 与力量关系：受多因素影响的S型曲线
- 个体内比较最可靠

**激活阈值：**
- 自动阈值：便捷但需视觉验证
- 手动阈值：非标准肌肉可能需要
- 静息基线：应接近零（否则检查电极）

**爆发特征：**
- **相位性**：短暂爆发（惊吓、快速运动）
- **紧张性**：持续激活（姿势、持续抓握）
- **节律性**：重复爆发（震颤、行走）

## 参考文献

- Fridlund, A. J., & Cacioppo, J. T. (1986). 人类肌电图研究指南. Psychophysiology, 23(5), 567-589.
- Hermens, H. J., Freriks, B., Disselhorst-Klug, C., & Rau, G. (2000). SEMG传感器及放置程序建议的制定. Journal of electromyography and Kinesiology, 10(5), 361-374.
- Silva, H., Scherer, R., Sousa, J., & Londral, A. (2013). 提升肌电接口可用性. Journal of Oral Rehabilitation, 40(6), 456-465.
- Tassinary, L. G., Cacioppo, J. T., & Vanman, E. J. (2017). 骨骼肌系统：表面肌电图. 见《心理生理学手册》(pp. 267-299). Cambridge University Press.
