# 皮肤电活动 (EDA) 分析

## 概述

皮肤电活动 (EDA)，也称为皮电反应 (GSR) 或皮肤电导 (SC)，通过测量皮肤的电导来反映交感神经系统兴奋和汗腺活动。EDA 广泛应用于心理生理学、情感计算和测谎领域。

## 主要处理流程

### eda_process()

对原始 EDA 信号进行自动化处理，返回紧张性/相位性分解和 SCR 特征。

```python
signals, info = nk.eda_process(eda_signal, sampling_rate=100, method='neurokit')
```

**处理步骤:**
1. 信号清洗（低通滤波）
2. 紧张性-相位性分解
3. 皮肤电导反应 (SCR) 检测
4. SCR 特征提取（起始点、峰值、幅度、上升/恢复时间）

**返回:**
- `signals`: 包含以下字段的 DataFrame:
  - `EDA_Clean`: 滤波后信号
  - `EDA_Tonic`: 慢变基线
  - `EDA_Phasic`: 快变响应
  - `SCR_Onsets`, `SCR_Peaks`, `SCR_Height`: 响应标记
  - `SCR_Amplitude`, `SCR_RiseTime`, `SCR_RecoveryTime`: 响应特征
- `info`: 包含处理参数的字典

**方法:**
- `'neurokit'`: cvxEDA 分解 + neurokit 峰值检测
- `'biosppy'`: 中值平滑 + biosppy 方法

## 预处理函数

### eda_clean()

通过低通滤波去除噪声。

```python
cleaned_eda = nk.eda_clean(eda_signal, sampling_rate=100, method='neurokit')
```

**方法:**
- `'neurokit'`: 低通巴特沃斯滤波器（3 Hz 截止）
- `'biosppy'`: 低通巴特沃斯滤波器（5 Hz 截止）

**自动跳过:**
- 采样率 < 7 Hz 时跳过清洗（已满足低通条件）

**原理:**
- EDA 频率成分通常为 0-3 Hz
- 去除高频噪声和运动伪影
- 保留慢速 SCR（典型上升时间 1-3 秒）

### eda_phasic()

将 EDA 分解为紧张性（慢变基线）和相位性（快速响应）成分。

```python
tonic, phasic = nk.eda_phasic(eda_cleaned, sampling_rate=100, method='cvxeda')
```

**方法:**

**1. cvxEDA（默认推荐）:**
```python
tonic, phasic = nk.eda_phasic(eda_cleaned, sampling_rate=100, method='cvxeda')
```
- 凸优化方法（Greco 等人，2016）
- 稀疏相位驱动模型
- 生理学最准确
- 计算密集但分解效果最优

**2. 中值平滑:**
```python
tonic, phasic = nk.eda_phasic(eda_cleaned, sampling_rate=100, method='smoothmedian')
```
- 可配置窗口的中值滤波
- 快速简单
- 准确性低于 cvxEDA

**3. 高通滤波（Biopac Acqknowledge）:**
```python
tonic, phasic = nk.eda_phasic(eda_cleaned, sampling_rate=100, method='highpass')
```
- 高通滤波器（0.05 Hz）提取相位成分
- 计算快速
- 紧张性通过减法推导

**4. SparsEDA:**
```python
tonic, phasic = nk.eda_phasic(eda_cleaned, sampling_rate=100, method='sparseda')
```
- 稀疏反卷积方法
- 替代优化方案

**返回:**
- `tonic`: 慢变皮肤电导水平 (SCL)
- `phasic`: 快速皮肤电导反应 (SCRs)

**生理学解释:**
- **紧张性 (SCL)**: 基线唤醒水平、整体激活状态、水合作用
- **相位性 (SCR)**: 事件相关反应、定向反射、情绪反应

### eda_peaks()

在相位成分中检测皮肤电导反应 (SCRs)。

```python
peaks, info = nk.eda_peaks(eda_phasic, sampling_rate=100, method='neurokit',
                           amplitude_min=0.1)
```

**方法:**
- `'neurokit'`: 可靠性优化，阈值可配置
- `'gamboa2008'`: Gamboa 算法
- `'kim2004'`: Kim 方法
- `'vanhalem2020'`: Van Halem 方法
- `'nabian2018'`: Nabian 算法

**关键参数:**
- `amplitude_min`: SCR 最小幅度（默认：0.1 µS）
  - 过低：噪声导致假阳性
  - 过高：遗漏有效小响应
- `rise_time_max`: 最大上升时间（默认：2 秒）
- `rise_time_min`: 最小上升时间（默认：0.01 秒）

**返回:**
- 包含以下字段的字典:
  - `SCR_Onsets`: SCR 起始点索引
  - `SCR_Peaks`: 峰值幅度索引
  - `SCR_Height`: 基线以上峰值高度
  - `SCR_Amplitude`: 起始点到峰值幅度
  - `SCR_RiseTime`: 起始点到峰值持续时间
  - `SCR_RecoveryTime`: 峰值到恢复持续时间（50% 衰减）

**SCR 时间特性:**
- **潜伏期**: 刺激后 1-3 秒（典型）
- **上升时间**: 0.5-3 秒
- **恢复时间**: 2-10 秒（至 50% 恢复）
- **最小幅度**: 0.01-0.05 µS（检测阈值）

### eda_fixpeaks()

修正检测到的 SCR 峰值（当前为 EDA 占位函数）。

```python
corrected_peaks = nk.eda_fixpeaks(peaks)
```

**注意:** 因动态较慢，对 EDA 的重要性低于心电信号。

## 分析函数

### eda_analyze()

根据数据时长自动选择分析类型。

```python
analysis = nk.eda_analyze(signals, sampling_rate=100)
```

**模式选择:**
- 时长 < 10 秒 → `eda_eventrelated()`
- 时长 ≥ 10 秒 → `eda_intervalrelated()`

**返回:**
- 包含适合分析模式的 EDA 指标的 DataFrame

### eda_eventrelated()

分析刺激锁定的 EDA 时段，获取事件相关响应。

```python
results = nk.eda_eventrelated(epochs)
```

**计算指标（每时段）:**
- `EDA_SCR`: SCR 存在性（二元：0 或 1）
- `SCR_Amplitude`: 时段内最大 SCR 幅度
- `SCR_Magnitude`: 相位活动均值
- `SCR_Peak_Amplitude`: 起始点到峰值幅度
- `SCR_RiseTime`: 起始点到峰值时间
- `SCR_RecoveryTime`: 50% 恢复时间
- `SCR_Latency`: 刺激到 SCR 起始的延迟
- `EDA_Tonic`: 时段内紧张性水平均值

**典型参数:**
- 时段长度：刺激后 0-10 秒
- 基线：刺激前 -1 至 0 秒
- 预期 SCR 潜伏期：1-3 秒

**应用场景:**
- 情绪刺激处理（图像、声音）
- 认知负荷评估（心算任务）
- 预期与预测误差
- 定向反应

### eda_intervalrelated()

分析长时程 EDA 记录的整体唤醒和激活模式。

```python
results = nk.eda_intervalrelated(signals, sampling_rate=100)
```

**计算指标:**
- `SCR_Peaks_N`: 检测到的 SCR 数量
- `SCR_Peaks_Amplitude_Mean`: SCR 平均幅度
- `EDA_Tonic_Mean`, `EDA_Tonic_SD`: 紧张性水平统计量
- `EDA_Sympathetic`: 交感神经系统指数
- `EDA_SympatheticN`: 标准化交感指数
- `EDA_Autocorrelation`: 时间结构（4 秒滞后）
- `EDA_Phasic_*`: 相位成分的均值、标准差、最小值、最大值

**记录时长:**
- **最低要求**: 10 秒
- **推荐**: 60+ 秒以获得稳定 SCR 率
- **交感指数**: 需 ≥64 秒

**应用场景:**
- 静息态唤醒评估
- 压力水平监测
- 基线交感活动
- 长期情感状态

## 专项分析函数

### eda_sympathetic()

从频带（0.045-0.25 Hz）推导交感神经系统活动。

```python
sympathetic = nk.eda_sympathetic(signals, sampling_rate=100, method='posada',
                                  show=False)
```

**方法:**
- `'posada'`: Posada-Quintero 方法（2016）
  - 0.045-0.25 Hz 频带谱功率
  - 经其他自主神经测量验证
- `'ghiasi'`: Ghiasi 方法（2018）
  - 替代性频率分析方法

**要求:**
- **最低时长**: 64 秒
- 满足目标频带的频率分辨率要求

**返回:**
- `EDA_Sympathetic`: 交感指数（绝对值）
- `EDA_SympatheticN`: 标准化交感指数（0-1）

**解释:**
- 值越高：交感神经兴奋增强
- 反映紧张性交感活动，非相位响应
- 作为 SCR 分析的补充

**应用场景:**
- 压力评估
- 随时间变化的唤醒监测
- 认知负荷测量
- 作为 HRV 的补充用于自主神经平衡分析

### eda_autocor()

通过自相关评估 EDA 信号的时间结构。

```python
autocorr = nk.eda_autocor(eda_phasic, sampling_rate=100, lag=4)
```

**参数:**
- `lag`: 时间滞后（秒）（默认：4 秒）

**解释:**
- 高自相关：持续缓慢变化的信号
- 低自相关：快速不相关波动
- 反映 SCR 的时间规律性

**应用场景:**
- 评估信号质量
- 表征响应模式
- 区分持续性与瞬时唤醒

### eda_changepoints()

检测 EDA 信号均值和方差的突变点。

```python
changepoints = nk.eda_changepoints(eda_phasic, penalty=10000, show=False)
```

**方法:**
- 基于惩罚的分段
- 识别状态间转换

**参数:**
- `penalty`: 控制敏感度（默认：10,000）
  - 惩罚值越高：突变点越少越稳健
  - 惩罚值越低：对小变化更敏感

**返回:**
- 检测到的突变点索引
- 可选分段可视化

**应用场景:**
- 连续监测中的状态转换识别
- 按唤醒水平分段数据
- 实验中检测阶段变化
- 自动化时段定义

## 可视化

### eda_plot()

创建处理后的 EDA 静态或交互式可视化。

```python
nk.eda_plot(signals, info, static=True)
```

**显示内容:**
- 原始与清洗后的 EDA 信号
- 紧张性与相位成分
- 检测到的 SCR 起始点、峰值和恢复点
- 交感指数时间序列（如已计算）

**交互模式 (`static=False`):**
- 基于 Plotly 的交互探索
- 缩放、平移、悬停查看详情
- 导出为图像格式

## 模拟与测试

### eda_simulate()

生成参数可配置的合成 EDA 信号。

```python
synthetic_eda = nk.eda_simulate(duration=10, sampling_rate=100, scr_number=3,
                                noise=0.01, drift=0.01)
```

**参数:**
- `duration`: 信号时长（秒）
- `sampling_rate`: 采样频率（Hz）
- `scr_number`: 包含的 SCR 数量
- `noise`: 高斯噪声水平
- `drift`: 慢基线漂移幅度
- `random_state`: 可重现性种子

**返回:**
- 具有真实 SCR 形态的合成 EDA 信号

**应用场景:**
- 算法测试与验证
- 教学演示
- 方法比较

## 实践注意事项

### 采样率建议
- **最低**: 10 Hz（满足慢速 SCR）
- **标准**: 20-100 Hz（多数商用系统）
- **高分辨率**: 1000 Hz（研究级，过采样）

### 记录时长

- 波萨达-金特罗, H. F., 弗洛里安, J. P., 奥尔胡埃拉-卡尼翁, A. D., 阿尔哈马-科拉莱斯, T., 查尔斯顿-比利亚洛沃斯, S., & 钟, K. H. (2016). 皮肤电活动功率谱密度分析用于交感神经功能评估. 《生物医学工程年鉴》, 44(10), 3124-3135.
- 道森, M. E., 谢尔, A. M., & 菲利翁, D. L. (2017). 皮肤电反应系统. 载于《心理生理学手册》(第217–243页). 剑桥大学出版社.
