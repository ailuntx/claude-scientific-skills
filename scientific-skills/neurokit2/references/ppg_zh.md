# 光电容积描记术 (PPG) 分析

## 概述

光电容积描记术 (PPG) 利用光学传感器测量微血管组织中的血容量变化。PPG 技术广泛应用于可穿戴设备、脉搏血氧仪和临床监护仪中，用于监测心率、脉搏特征及心血管评估。

## 主要处理流程

### ppg_process()

自动化 PPG 信号处理流程。

```python
signals, info = nk.ppg_process(ppg_signal, sampling_rate=100, method='elgendi')
```

**处理步骤：**
1. 信号清洗（滤波）
2. 收缩期峰值检测
3. 心率计算
4. 信号质量评估

**返回结果：**
- `signals`：包含以下字段的 DataFrame：
  - `PPG_Clean`：滤波后的 PPG 信号
  - `PPG_Peaks`：收缩期峰值标记
  - `PPG_Rate`：瞬时心率 (BPM)
  - `PPG_Quality`：信号质量指标
- `info`：包含峰值索引和参数的字典

**处理方法：**
- `'elgendi'`：Elgendi 等人 (2013) 算法（默认，鲁棒性强）
- `'nabian2018'`：Nabian 等人 (2018) 方法

## 预处理函数

### ppg_clean()

为峰值检测准备原始 PPG 信号。

```python
cleaned_ppg = nk.ppg_clean(ppg_signal, sampling_rate=100, method='elgendi')
```

**处理方法：**

**1. Elgendi（默认）：**
- 巴特沃斯带通滤波器 (0.5-8 Hz)
- 消除基线漂移和高频噪声
- 针对峰值检测可靠性优化

**2. Nabian2018：**
- 替代滤波方案
- 不同频率特性

**PPG 信号特征：**
- **收缩期峰值**：快速上升支，尖锐峰值（心脏射血）
- **重搏切迹**：次峰（主动脉瓣关闭）
- **基线**：呼吸、运动和灌注引起的慢速漂移

### ppg_peaks()

检测 PPG 信号的收缩期峰值。

```python
peaks, info = nk.ppg_peaks(cleaned_ppg, sampling_rate=100, method='elgendi',
                           correct_artifacts=False)
```

**处理方法：**
- `'elgendi'`：双移动平均动态阈值法
- `'bishop'`：Bishop 算法
- `'nabian2018'`：Nabian 方法
- `'scipy'`：简易 scipy 峰值检测

**伪迹校正：**
- 设置 `correct_artifacts=True` 进行生理合理性检查
- 基于心跳间隔异常值剔除伪峰

**返回结果：**
- 包含 `'PPG_Peaks'` 峰值索引的字典

**典型心跳间隔：**
- 成人静息状态：600-1200 ms (50-100 BPM)
- 运动员：可能更长（心动过缓）
- 压力/运动状态：更短 (<600 ms, >100 BPM)

### ppg_findpeaks()

底层峰值检测与算法比较。

```python
peaks_dict = nk.ppg_findpeaks(cleaned_ppg, sampling_rate=100, method='elgendi')
```

**应用场景：**
- 自定义参数调优
- 算法测试
- 研究方法开发

## 分析函数

### ppg_analyze()

自动选择事件相关或区间相关分析模式。

```python
analysis = nk.ppg_analyze(signals, sampling_rate=100)
```

**模式选择：**
- 时长 < 10 秒 → 事件相关分析
- 时长 ≥ 10 秒 → 区间相关分析

### ppg_eventrelated()

分析离散事件/刺激的 PPG 响应。

```python
results = nk.ppg_eventrelated(epochs)
```

**计算指标（每时段）：**
- `PPG_Rate_Baseline`：事件前心率
- `PPG_Rate_Min/Max`：时段内最小/最大心率
- 跨时段时间窗的心率动态变化

**应用场景：**
- 情绪刺激的心血管响应
- 认知负荷评估
- 应激反应范式

### ppg_intervalrelated()

分析长时程 PPG 记录。

```python
results = nk.ppg_intervalrelated(signals, sampling_rate=100)
```

**计算指标：**
- `PPG_Rate_Mean`：平均心率
- 心率变异性 (HRV) 指标
  - 调用 `hrv()` 函数
  - 时域、频域及非线性域分析

**记录时长要求：**
- 基础心率：至少 60 秒
- HRV 分析：建议 2-5 分钟

**应用场景：**
- 静息态心血管评估
- 可穿戴设备数据分析
- 长时程心率监测

## 质量评估

### ppg_quality()

评估信号质量与可靠性。

```python
quality = nk.ppg_quality(ppg_signal, sampling_rate=100, method='averageQRS')
```

**评估方法：**

**1. averageQRS（默认）：**
- 模板匹配法
- 计算各脉冲与平均模板的相关系数
- 返回每搏质量评分 (0-1)
- 阈值：>0.6 为可接受质量

**2. dissimilarity：**
- 形态差异度测量
- 检测形态学变化

**应用场景：**
- 识别受损信号段
- 分析前过滤低质量数据
- 验证峰值检测准确性

**常见质量问题：**
- 运动伪影：信号突变
- 传感器接触不良：低幅值，高噪声
- 血管收缩：信号幅值降低（寒冷、压力）

## 实用函数

### ppg_segment()

提取单脉冲进行形态学分析。

```python
pulses = nk.ppg_segment(cleaned_ppg, peaks, sampling_rate=100)
```

**返回结果：**
- 以收缩期峰值为中心的脉冲时段字典
- 支持脉冲间对比
- 跨条件形态学分析

**应用场景：**
- 脉搏波分析
- 动脉僵硬度评估
- 血管老化研究

### ppg_methods()

记录分析中使用的预处理方法。

```python
methods_info = nk.ppg_methods(method='elgendi')
```

**返回结果：**
- 描述处理流程的字符串
- 适用于学术论文方法章节

## 模拟与可视化

### ppg_simulate()

生成合成 PPG 信号用于测试。

```python
synthetic_ppg = nk.ppg_simulate(duration=60, sampling_rate=100, heart_rate=70,
                                noise=0.1, random_state=42)
```

**参数说明：**
- `heart_rate`：平均心率 (默认: 70 BPM)
- `heart_rate_std`：HRV 幅度
- `noise`：高斯噪声水平
- `random_state`：随机种子

**应用场景：**
- 算法验证
- 参数优化
- 教学演示

### ppg_plot()

可视化处理后的 PPG 信号。

```python
nk.ppg_plot(signals, info, static=True)
```

**显示内容：**
- 原始与清洗后的 PPG 信号
- 检测到的收缩期峰值
- 瞬时心率轨迹
- 信号质量指标

## 实际考量

### 采样率建议
- **最低要求**：20 Hz（基础心率）
- **标准范围**：50-100 Hz（多数可穿戴设备）
- **高分辨率**：200-500 Hz（科研，脉搏波分析）
- **过高采样**：>1000 Hz（PPG 无需此精度）

### 记录时长
- **心率检测**：≥10 秒（少量心跳）
- **HRV 分析**：至少 2-5 分钟
- **长时监测**：数小时至数天（可穿戴设备）

### 传感器放置

**常用部位：**
- **指尖**：信号质量最佳，最常用
- **耳垂**：运动伪影少，临床常用
- **手腕**：可穿戴设备（智能手表）
- **前额**：反射模式，医疗监护

**透射式 vs 反射式：**
- **透射式**：光线穿透组织（指尖、耳垂）
  - 信号质量更高
  - 运动伪影更少
- **反射式**：光线从组织反射（手腕、前额）
  - 更易受噪声干扰
  - 可穿戴设备更便捷

### 常见问题与解决方案

**信号幅值过低：**
- 灌注不足：温暖双手，促进血流
- 传感器接触：调整位置，清洁皮肤
- 血管收缩：环境温度，压力因素

**运动伪影：**
- 可穿戴设备主要问题
- 自适应滤波，加速度计校正
- 模板匹配，异常值剔除

**基线漂移：**
- 呼吸调制（正常现象）
- 运动或压力变化
- 高通滤波或去趋势处理

**峰值漏检：**
- 低质量信号：检查传感器接触
- 算法参数：调整检测阈值
- 尝试替代检测方法

### 最佳实践

**标准工作流：**
```python
# 1. 信号清洗
cleaned = nk.ppg_clean(ppg_raw, sampling_rate=100, method='elgendi')

# 2. 峰值检测（含伪迹校正）
peaks, info = nk.ppg_peaks(cleaned, sampling_rate=100, correct_artifacts=True)

# 3. 质量评估
quality = nk.ppg_quality(cleaned, sampling_rate=100)

# 4. 综合处理（替代方案）
signals, info = nk.ppg_process(ppg_raw, sampling_rate=100)

# 5. 分析
analysis = nk.ppg_analyze(signals, sampling_rate=100)
```

**PPG 衍生 HRV：**
```python
# 处理 PPG 信号
signals, info = nk.ppg_process(ppg_raw, sampling_rate=100)

# 提取峰值并计算 HRV
hrv_indices = nk.hrv(info['PPG_Peaks'], sampling_rate=100)

# PPG 衍生的 HRV 有效，但与 ECG 衍生 HRV 存在细微差异
# 差异源于脉搏传导时间及血管特性
```

## 临床与研究应用

**可穿戴健康监测：**
- 消费级智能手表与健身追踪器
- 连续心率监测
- 睡眠追踪与活动评估

**临床监护：**
- 脉搏血氧监测 (SpO₂ + 心率)
- 围手术期监护
- 重症监护心率评估

**心血管评估：**
- 脉搏波分析
- 动脉僵硬度评估（脉搏传导时间）
- 血管老化指标

**自主神经功能：**
- PPG 衍生的 HRV (PPG-HRV)
- 压力与恢复监测
- 脑力负荷评估

**远程患者监护：**
- 远程医疗应用
- 家庭健康追踪
- 慢性病管理

**情感计算：**
- 生理信号情绪识别
- 用户体验研究
- 人机交互

## PPG 与 ECG 对比

**PPG 优势：**
- 无创，无需电极
- 适合长时程监测
- 低成本，可微型化
- 适用于可穿戴设备

**PPG 局限：**
- 更易受运动伪影影响
- 低灌注条件下信号质量差
- 脉搏传导时间延迟
- 无法评估心电活动

**HRV 对比：**
- PPG-HRV 在时域/频域分析中基本有效
- 因脉搏传导时间变异性存在细微差异
- 临床 HRV 优先选择 ECG
- 科研和消费级应用中 PPG 可接受

## 解读指南

**PPG 心率：**
- 解读规则与 ECG 心率相同
- 脉搏传导时间延迟对心率计算影响可忽略
- 运动伪影更常见：需结合信号质量验证

**脉搏幅值：**
- 反映外周灌注状态
- 升高：血管舒张，温暖环境
- 降低：血管收缩，寒冷，压力，接触不良

**脉搏形态：**
- 收缩期峰值：心脏射血
- 重搏切迹：主动脉瓣关闭，动脉顺应性
- 老化/僵化：重搏切迹更早更显著

## 参考文献

- Elgendi, M. (2012). 指尖光电容积描记信号分析. 当代心脏病学评论, 8(1), 14-25.
- Elgendi, M., Norton, I., Brearley, M., Abbott, D., & Schuurmans, D. (2013). 热带环境下急救人员加速度光电容积图的收缩期峰值检测. PLoS 综合, 8(10), e76585.
- Allen, J. (2007). 光电容积描记术及其在临床生理测量中的应用. 生理测量, 28(3), R1.
- Tamura, T., Maeda, Y., Sekine, M., & Yoshida, M. (2014). 可穿戴光电容积传感器——过去与现在. 电子学, 3(2), 282-302.
