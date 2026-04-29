# 通用信号处理

## 概述

NeuroKit2 提供适用于任何时间序列数据的全面信号处理工具。这些函数支持跨所有信号类型的滤波、变换、峰值检测、分解和分析操作。

## 预处理函数

### signal_filter()

应用频域滤波以去除噪声或分离频段。

```python
filtered = nk.signal_filter(signal, sampling_rate=1000, lowcut=None, highcut=None,
                            method='butterworth', order=5)
```

**滤波器类型（通过 lowcut/highcut 组合）：**

**低通滤波**（仅 highcut）：
```python
lowpass = nk.signal_filter(signal, sampling_rate=1000, highcut=50)
```
- 去除高于 highcut 的频率
- 平滑信号，消除高频噪声

**高通滤波**（仅 lowcut）：
```python
highpass = nk.signal_filter(signal, sampling_rate=1000, lowcut=0.5)
```
- 去除低于 lowcut 的频率
- 消除基线漂移、直流偏移

**带通滤波**（同时指定 lowcut 和 highcut）：
```python
bandpass = nk.signal_filter(signal, sampling_rate=1000, lowcut=0.5, highcut=50)
```
- 保留 lowcut 与 highcut 之间的频率
- 分离特定频段

**带阻/陷波滤波**（电源线干扰消除）：
```python
notch = nk.signal_filter(signal, sampling_rate=1000, method='powerline', powerline=50)
```
- 消除 50 或 60 Hz 电源线噪声
- 窄带陷波滤波器

**方法：**
- `'butterworth'`（默认）：平滑频率响应，平坦通带
- `'bessel'`：线性相位，最小振铃效应
- `'chebyshev1'`：陡峭滚降，通带波纹
- `'chebyshev2'`：陡峭滚降，阻带波纹
- `'elliptic'`：最陡滚降，通带阻带均有波纹
- `'powerline'`：50/60 Hz 陷波滤波器

**阶数参数：**
- 高阶：更陡峭过渡，更多振铃
- 低阶：更平缓过渡，较少振铃
- 生理信号典型值：2-5阶

### signal_sanitize()

清除无效值（NaN, inf）并可选插值。

```python
clean_signal = nk.signal_sanitize(signal, interpolate=True)
```

**使用场景：**
- 处理缺失数据点
- 清除标记为 NaN 的伪迹
- 为需要连续数据的算法准备信号

### signal_resample()

改变信号采样率（上采样或下采样）。

```python
resampled = nk.signal_resample(signal, sampling_rate=1000, desired_sampling_rate=500,
                               method='interpolation')
```

**方法：**
- `'interpolation'`：三次样条插值
- `'FFT'`：频域重采样
- `'poly'`：多相滤波（最适合下采样）

**使用场景：**
- 统一多模态记录的采样率
- 缩减数据量（下采样）
- 提升时间分辨率（上采样）

### signal_fillmissing()

插值填补缺失或无效数据点。

```python
filled = nk.signal_fillmissing(signal, method='linear')
```

**方法：**
- `'linear'`：线性插值
- `'nearest'`：最近邻插值
- `'pad'`：前向/后向填充
- `'cubic'`：三次样条插值
- `'polynomial'`：多项式拟合

## 变换函数

### signal_detrend()

消除信号中的慢速趋势。

```python
detrended = nk.signal_detrend(signal, method='polynomial', order=1)
```

**方法：**
- `'polynomial'`：拟合并减去多项式（1阶=线性）
- `'loess'`：局部加权回归
- `'tarvainen2002'`：平滑先验去趋势法

**使用场景：**
- 消除基线漂移
- 分析前稳定均值
- 为假设平稳性的算法做准备

### signal_decompose()

将信号分解为组成成分。

```python
components = nk.signal_decompose(signal, sampling_rate=1000, method='emd')
```

**方法：**

**经验模态分解（EMD）：**
```python
components = nk.signal_decompose(signal, sampling_rate=1000, method='emd')
```
- 数据自适应分解为本征模态函数（IMF）
- 每个IMF代表不同频率成分（高频到低频）
- 无预定义基函数

**奇异谱分析（SSA）：**
```python
components = nk.signal_decompose(signal, method='ssa')
```
- 分解为趋势项、振荡项和噪声
- 基于轨迹矩阵的特征值分解

**小波分解：**
- 时频表示
- 时间和频率局部化

**返回：**
- 包含成分信号的字典
- 趋势项、振荡成分、残差项

**使用场景：**
- 分离生理节律
- 区分信号与噪声
- 多尺度分析

### signal_recompose()

从分解成分重建信号。

```python
reconstructed = nk.signal_recompose(components, indices=[1, 2, 3])
```

**使用场景：**
- 分解后的选择性重建
- 移除特定IMF或成分
- 自适应滤波

### signal_binarize()

基于阈值将连续信号二值化（0/1）。

```python
binary = nk.signal_binarize(signal, method='threshold', threshold=0.5)
```

**方法：**
- `'threshold'`：简单阈值法
- `'median'`：基于中位数
- `'mean'`：基于均值
- `'quantile'`：基于分位数

**使用场景：**
- 从连续信号检测事件
- 提取触发点
- 状态分类

### signal_distort()

添加可控噪声或伪迹用于测试。

```python
distorted = nk.signal_distort(signal, sampling_rate=1000, noise_amplitude=0.1,
                              noise_frequency=50, artifacts_amplitude=0.5)
```

**参数：**
- `noise_amplitude`：高斯噪声强度
- `noise_frequency`：正弦干扰（如电源线）
- `artifacts_amplitude`：随机尖峰伪迹强度
- `artifacts_number`：添加的伪迹数量

**使用场景：**
- 算法鲁棒性测试
- 预处理方法评估
- 真实数据模拟

### signal_interpolate()

在新时间点插值或填补间隙。

```python
interpolated = nk.signal_interpolate(x_values, y_values, x_new=None, method='quadratic')
```

**方法：**
- `'linear'`、`'quadratic'`、`'cubic'`：多项式插值
- `'nearest'`：最近邻插值
- `'monotone_cubic'`：保持单调性

**使用场景：**
- 将不规则采样转为规则网格
- 可视化上采样
- 对齐不同时间基准的信号

### signal_merge()

合并不同采样率的多个信号。

```python
merged = nk.signal_merge(signal1, signal2, time1=None, time2=None, sampling_rate=None)
```

**使用场景：**
- 多模态信号整合
- 合并不同设备数据
- 基于时间戳同步

### signal_flatline()

识别信号恒定段（伪迹或传感器故障）。

```python
flatline_mask = nk.signal_flatline(signal, duration=5.0, sampling_rate=1000)
```

**返回：**
- 二值掩码（True表示平直线段）
- 持续时间阈值防止正常稳定段的误判

### signal_noise()

向信号添加各类噪声。

```python
noisy = nk.signal_noise(signal, sampling_rate=1000, noise_type='gaussian',
                        amplitude=0.1)
```

**噪声类型：**
- `'gaussian'`：白噪声
- `'pink'`：1/f噪声（生理信号常见）
- `'brown'`：布朗噪声（随机游走）
- `'powerline'`：正弦干扰（50/60 Hz）

### signal_surrogate()

生成保留特定特性的替代信号。

```python
surrogate = nk.signal_surrogate(signal, method='IAAFT')
```

**方法：**
- `'IAAFT'`：迭代幅度调整傅里叶变换
  - 保留幅度分布和功率谱
- `'random_shuffle'`：随机排列（零假设检验）

**使用场景：**
- 非线性检验
- 统计检验的零假设生成

## 峰值检测与校正

### signal_findpeaks()

检测信号中的局部极大值（峰值）。

```python
peaks_dict = nk.signal_findpeaks(signal, height_min=None, height_max=None,
                                 relative_height_min=None, relative_height_max=None)
```

**关键参数：**
- `height_min/max`：绝对幅度阈值
- `relative_height_min/max`：相对于信号范围（0-1）
- `threshold`：最小显著度
- `distance`：峰值间最小样本数

**返回：**
- 包含以下内容的字典：
  - `'Peaks'`：峰值索引
  - `'Height'`：峰值幅度
  - `'Distance'`：峰间间隔

**使用场景：**
- 通用信号峰值检测
- R波峰、呼吸峰、脉搏峰
- 事件检测

### signal_fixpeaks()

校正检测到的伪迹和异常峰值。

```python
corrected = nk.signal_fixpeaks(peaks, sampling_rate=1000, iterative=True,
                               method='Kubios', interval_min=None, interval_max=None)
```

**方法：**
- `'Kubios'`：Kubios HRV软件方法（默认）
- `'Malik1996'`：1996年工作组标准
- `'Kamath1993'`：Kamath方法

**校正内容：**
- 移除生理学不可信的间隔
- 插值填补缺失峰值
- 移除额外检测峰值（重复项）

**使用场景：**
- R-R间期伪迹校正
- 提升HRV分析质量
- 呼吸或脉搏峰值校正

## 分析函数

### signal_rate()

根据事件发生点（峰值）计算瞬时频率。

```python
rate = nk.signal_rate(peaks, sampling_rate=1000, desired_length=None)
```

**方法：**
- 计算事件间间隔
- 转换为每分钟事件数
- 插值至目标长度

**使用场景：**
- 通过R波峰计算心率
- 通过呼吸峰计算呼吸频率
- 任何周期性事件频率

### signal_period()

检测信号中的主导周期/频率。

```python
period = nk.signal_period(signal, sampling_rate=1000, method='autocorrelation',
                          show=False)
```

**方法：**
- `'autocorrelation'`：自相关函数峰值
- `'powerspectraldensity'`：频谱峰值

**返回：**
- 周期（样本数或秒数）
- 频率（1/周期，单位Hz）

**使用场景：**
- 检测主导节律
- 估计基频
- 呼吸频率、心率估计

### signal_phase()

计算信号的瞬时相位。

```python
phase = nk.signal_phase(signal, method='hilbert')
```

**方法：**
- `'hilbert'`：希尔伯特变换（解析信号）
- `'wavelet'`：基于小波的相位

**返回：**
- 相位（弧度制：-π 至 π，或归一化 0 至 1）

**使用场景：**
- 锁相分析
- 同步性测量
- 相位-幅度耦合

### signal_psd()

计算功率谱密度。

```python
psd, freqs = nk.signal_psd(signal, sampling_rate=1000, method='welch',
                           max_frequency=None, show=False)
```

**方法：**
- `'welch'`：Welch周期图（加窗FFT，默认）
- `'multitapers'`：多锥形法（更优频谱估计）
- `'lomb'`：Lomb-Scargle（非均匀采样数据）
- `'burg'`：自回归模型（参数法）

**返回：**
- `psd`：各频率点功率（单位²/Hz）
- `freqs`：频率分档（Hz）

**使用场景：**
- 频率成分分析
- HRV频域分析
- 频谱特征识别

### signal_power()

计算特定频段的功率。

```python
power_dict = nk.signal_power(signal, sampling_rate=1000, frequency_bands={
    'VLF': (0.003, 0.04),
    'LF': (0.04, 0.15),
    'HF': (0.15, 0.4)
}, method='welch')
```

**返回：**
- 包含各频段绝对/相对功率的字典
- 峰值频率

**使用场景：**
- HRV频段分析
- EEG频带功率
- 节律量化

### signal_autocor()

计算自相关函数。

```python
autocorr = nk.signal_autocor(signal, lag=1000, show=False)
```

**解读：**
- 滞后点高自相关：信号每滞后个样本重复
- 周期信号：在周期倍数处出现峰值
- 随机信号：快速衰减至零

**使用场景：**
- 检测周期性
- 评估时间结构
- 信号记忆性分析

### signal_zerocrossings()

统计过零点（符号变化）。

```python
n_crossings = nk.signal_zerocrossings(signal)
```

**解读：**
- 过零点越多：高频成分越丰富
- 与主导频率相关（粗略估计）

**使用场景：**
- 简单频率估计
- 信号规律性评估

### signal_changepoints()

检测信号特性突变点（均值、方差）。

```python
changepoints = nk.signal_changepoints(signal, penalty=10, method='pelt', show=False)
```

**方法：**
- `'pelt'`：剪枝精确线性时间法（快速，精确）
- `'binseg'`：二分分割法（更快，近似）

**参数：**
- `penalty`：控制敏感度（值越高突变点越少）

**返回：**
- 检测到的突变点索引
- 突变点间的分段

**使用场景：**
- 将信号分割为不同状态
- 检测状态转换（如睡眠阶段、唤醒状态）
- 自动分段定义

### signal_synchrony()

评估两个信号的同步性。

```python
sync = nk.signal_synchrony(signal1, signal2, method='correlation')
```

**方法：**
- `'correlation'`：皮尔逊相关系数
- `'coherence'`：频域相干性
- `'mutual_information'`：互信息度量
- `'phase'`：相位锁定值

**使用场景：**
- 心脑耦合分析
- 脑间同步性
- 多通道协调性

### signal_smooth()

应用平滑处理以降低噪声。

```python
smoothed = nk.signal_smooth(signal, method='convolution', kernel='boxzen', size=10)
```

**方法：**
- `'convolution'`：应用核函数（矩形窗、高斯窗等）
- `'median'`：中值滤波（抗离群值）
- `'savgol'`：Savitzky-Golay滤波（保留峰值）
- `'loess'`：局部加权回归

**核函数类型（用于卷积）：**
- `'boxcar'`：简单移动平均
- `'gaussian'`：高斯加权平均
- `'hann'`、`'hamming'`、`'blackman'`：窗函数

**使用场景：**
- 降噪处理
- 趋势提取
- 可视化增强

### signal_timefrequency()

时频表示（频谱图）。

```python
tf, time, freq = nk.signal_timefrequency(signal, sampling_rate=1000, method='stft',
                                        max_frequency=50, show=False)
```

**方法：**
- `'stft'`：短时傅里叶变换
- `'cwt'`：连续小波变换

**返回：**
- `tf`：时频矩阵（各时频点功率）
- `time`：时间分档
- `freq`：频率分档

**使用场景：**
- 非平稳信号分析
- 时变频率成分
- EEG/MEG时频分析

## 模拟

### signal_simulate()

生成多种合成信号用于测试。

```python
signal = nk.signal_simulate(duration=10, sampling_rate=1000, frequency=[5, 10],
                            amplitude=[1.0, 0.5], noise=0.1)
```

**信号类型

- 滤波会在信号边缘引入伪影
- 处理前先填充信号，完成后裁剪
- 或舍弃首尾数秒数据

**处理数据间隙：**
- 小间隙：用`signal_fillmissing()`进行插值填补
- 大间隙：分割信号后分别分析
- 将间隙标记为NaN，谨慎使用插值

**操作流程整合：**
```python
# 典型预处理流程
signal = nk.signal_sanitize(raw_signal)  # 移除无效值
signal = nk.signal_filter(signal, sampling_rate=1000, lowcut=0.5, highcut=40)  # 带通滤波
signal = nk.signal_detrend(signal, method='polynomial', order=1)  # 去除线性趋势
```

**性能优化要点：**
- 滤波：FFT方法对长信号处理更快
- 重采样：在流程早期降采样可提速
- 大型数据集：内存受限时分块处理

## 参考文献

- Virtanen, P., et al. (2020). SciPy 1.0: fundamental algorithms for scientific computing in Python. Nature methods, 17(3), 261-272.
- Tarvainen, M. P., Ranta-aho, P. O., & Karjalainen, P. A. (2002). An advanced detrending method with application to HRV analysis. IEEE Transactions on Biomedical Engineering, 49(2), 172-175.
- Huang, N. E., et al. (1998). The empirical mode decomposition and the Hilbert spectrum for nonlinear and non-stationary time series analysis. Proceedings of the Royal Society of London A, 454(1971), 903-995.
