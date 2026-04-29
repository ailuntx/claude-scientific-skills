# 心率变异性（HRV）分析

## 概述

心率变异性（HRV）反映了连续心跳间时间间隔的变化，为自主神经系统调节、心血管健康和心理状态提供洞察。NeuroKit2提供跨时域、频域和非线性领域的全面HRV分析。

## 主要函数

### hrv()

一次性计算所有可用领域的HRV指标。

```python
hrv_indices = nk.hrv(peaks, sampling_rate=1000, show=False)
```

**输入:**
- `peaks`: 包含`'ECG_R_Peaks'`键的字典或R峰索引数组
- `sampling_rate`: 信号采样率（Hz）

**返回:**
- 包含所有领域HRV指标的数据框：
  - 时域指标
  - 频域功率谱
  - 非线性复杂度指标

**此为便捷封装函数**，整合了：
- `hrv_time()`
- `hrv_frequency()`
- `hrv_nonlinear()`

## 时域分析

### hrv_time()

基于心跳间期（IBIs）计算时域HRV指标。

```python
hrv_time = nk.hrv_time(peaks, sampling_rate=1000)
```

### 关键指标

**基础间期统计:**
- `HRV_MeanNN`: NN间期均值（ms）
- `HRV_SDNN`: NN间期标准差（ms）
  - 反映整体HRV，捕获所有周期成分
  - 短期需≥5分钟，长期需≥24小时
- `HRV_RMSSD`: 连续差值的均方根（ms）
  - 高频变异性，反映副交感神经活动
  - 短时记录下更稳定

**连续差值度量:**
- `HRV_SDSD`: 连续差值的标准差（ms）
  - 类似RMSSD，与副交感活动相关
- `HRV_pNN50`: NN间期差值>50ms的百分比
  - 副交感神经指标，部分人群可能不敏感
- `HRV_pNN20`: NN间期差值>20ms的百分比
  - pNN50的更敏感替代指标

**范围度量:**
- `HRV_MinNN`, `HRV_MaxNN`: 最小/最大NN间期（ms）
- `HRV_CVNN`: 变异系数（SDNN/MeanNN）
  - 标准化度量，适用于跨对象比较
- `HRV_CVSD`: 连续差值变异系数（RMSSD/MeanNN）

**中位数统计:**
- `HRV_MedianNN`: NN间期中位数（ms）
  - 对异常值稳健
- `HRV_MadNN`: NN间期中位数绝对偏差
  - 稳健的离散度量
- `HRV_MCVNN`: 基于中位数的变异系数

**高级时域指标:**
- `HRV_IQRNN`: NN间期四分位距
- `HRV_pNN10`, `HRV_pNN25`, `HRV_pNN40`: 附加百分位阈值
- `HRV_TINN`: NN间期直方图三角插值
- `HRV_HTI`: HRV三角指数（总NN间期/直方图高度）

### 记录时长要求
- **超短期（<5分钟）**: RMSSD, pNN50最可靠
- **短期（5分钟）**: 临床标准，所有时域指标有效
- **长期（24小时）**: SDNN解读必需，可捕获昼夜节律

## 频域分析

### hrv_frequency()

通过频谱分析计算各频段的HRV功率。

```python
hrv_freq = nk.hrv_frequency(peaks, sampling_rate=1000, ulf=(0, 0.0033), vlf=(0.0033, 0.04),
                            lf=(0.04, 0.15), hf=(0.15, 0.4), vhf=(0.4, 0.5),
                            psd_method='welch', normalize=True)
```

### 频段划分

**超低频（ULF）: 0-0.0033 Hz**
- 需≥24小时记录
- 昼夜节律，体温调节
- 慢速代谢过程

**极低频（VLF）: 0.0033-0.04 Hz**
- 需≥5分钟记录
- 体温调节，激素波动
- 肾素-血管紧张素系统，外周血管舒缩活动

**低频（LF）: 0.04-0.15 Hz**
- 交感与副交感神经混合影响
- 压力感受器反射活动
- 血压调节（10秒节律）

**高频（HF）: 0.15-0.4 Hz**
- 副交感（迷走）神经活动
- 呼吸性窦性心律不齐
- 与呼吸同步（呼吸频率范围）

**超高频（VHF）: 0.4-0.5 Hz**
- 极少使用，可能反映测量噪声
- 需谨慎解读

### 关键指标

**绝对功率（ms²）:**
- `HRV_ULF`, `HRV_VLF`, `HRV_LF`, `HRV_HF`, `HRV_VHF`: 各频段功率
- `HRV_TP`: 总功率（NN间期方差）
- `HRV_LFHF`: LF/HF比值（交感-迷走平衡）

**归一化功率:**
- `HRV_LFn`: LF功率/(LF+HF) - 归一化LF
- `HRV_HFn`: HF功率/(LF+HF) - 归一化HF
- `HRV_LnHF`: HF自然对数（对数正态分布）

**峰值频率:**
- `HRV_LFpeak`, `HRV_HFpeak`: 各频段最大功率频率
- 用于识别主导振荡

### 功率谱密度方法

**Welch法（默认）:**
```python
hrv_freq = nk.hrv_frequency(peaks, sampling_rate=1000, psd_method='welch')
```
- 带重叠的窗口化FFT
- 频谱更平滑，方差更低
- 标准HRV分析首选

**Lomb-Scargle周期图:**
```python
hrv_freq = nk.hrv_frequency(peaks, sampling_rate=1000, psd_method='lomb')
```
- 处理非均匀采样数据
- 无需插值
- 更适用于含噪声或伪影数据

**多锥形法:**
```python
hrv_freq = nk.hrv_frequency(peaks, sampling_rate=1000, psd_method='multitapers')
```
- 卓越的频谱估计
- 最小偏差下的低方差
- 计算密集

**Burg自回归:**
```python
hrv_freq = nk.hrv_frequency(peaks, sampling_rate=1000, psd_method='burg', order=16)
```
- 参数化方法
- 峰值清晰的平滑频谱
- 需选择模型阶数

### 解读指南

**LF/HF比值:**
- 传统解读为交感-迷走平衡
- **注意**: 新证据质疑此解释
- LF反映交感与副交感双重影响
- 情境依赖：控制呼吸影响HF

**HF功率:**
- 可靠的副交感神经指标
- 休息/放松/深呼吸时升高
- 压力/焦虑/交感激活时降低

**记录要求:**
- **最低**: 60秒（LF/HF估算）
- **推荐**: 2-5分钟（短期HRV）
- **最优**: 5分钟（工作组标准）
- **长期**: 24小时（ULF分析）

## 非线性域分析

### hrv_nonlinear()

计算反映自主神经动态的复杂度、熵和分形指标。

```python
hrv_nonlinear = nk.hrv_nonlinear(peaks, sampling_rate=1000)
```

### Poincaré图指标

**Poincaré图**: NN(i+1)与NN(i)的散点图几何特征

- `HRV_SD1`: 垂直于恒等线的标准差（ms）
  - 短期HRV，快节奏逐搏变异性
  - 反映副交感活动
  - 数学关联：SD1 ≈ RMSSD/√2

- `HRV_SD2`: 沿恒等线的标准差（ms）
  - 长期HRV，慢变异性
  - 反映交感与副交感活动
  - 与SDNN相关

- `HRV_SD1SD2`: SD1/SD2比值
  - 短/长期变异性平衡
  - <1：以长期变异性为主

- `HRV_SD2SD1`: SD2/SD1比值
  - SD1SD2的倒数

- `HRV_S`: 椭圆面积（π × SD1 × SD2）
  - HRV总强度

- `HRV_CSI`: 心脏交感指数（SD2/SD1）
  - 建议的交感神经指标

- `HRV_CVI`: 心脏迷走指数（log10(SD1 × SD2)）
  - 建议的副交感神经指标

- `HRV_CSI_Modified`: 修正CSI（SD2²/(SD1 × SD2)）

### 心率不对称性

分析心率加速与减速对HRV的差异性贡献。

- `HRV_GI`: Guzik指数 - 短期变异不对称性
- `HRV_SI`: 斜率指数 - 长期变异不对称性
- `HRV_AI`: 面积指数 - 整体不对称性
- `HRV_PI`: Porta指数 - 减速百分比
- `HRV_C1d`, `HRV_C2d`: 减速贡献
- `HRV_C1a`, `HRV_C2a`: 加速贡献
- `HRV_SD1d`, `HRV_SD1a`: 减速/加速的Poincaré SD1
- `HRV_SD2d`, `HRV_SD2a`: 减速/加速的Poincaré SD2

**解读:**
- 健康人群：存在不对称性（更多/更大减速）
- 临床人群：不对称性降低
- 反映加速与减速的差异性自主神经控制

### 熵度量

**近似熵（ApEn）:**
- `HRV_ApEn`: 规律性度量，值越低越规律/可预测
- 对数据长度、阶数m、容差r敏感

**样本熵（SampEn）:**
- `HRV_SampEn`: 改进版ApEn，降低数据长度依赖性
- 短时记录下更稳定
- 值越低表示模式越规律

**多尺度熵（MSE）:**
- `HRV_MSE`: 跨时间尺度的复杂度
- 区分真实复杂度与随机性

**模糊熵:**
- `HRV_FuzzyEn`: 模式匹配的模糊隶属函数
- 短数据下更稳定

**香农熵:**
- `HRV_ShanEn`: 信息论随机性度量

### 分形度量

**去趋势波动分析（DFA）:**
- `HRV_DFA_alpha1`: 短期分形标度指数（4-11搏动）
  - α1 > 1：相关性（心脏病中降低）
  - α1 ≈ 1：粉红噪声（健康状态）
  - α1 < 0.5：反相关性

- `HRV_DFA_alpha2`: 长期分形标度指数（>11搏动）
  - 反映长程相关性

- `HRV_DFA_alpha1alpha2`: α1/α2比值

**相关维度:**
- `HRV_CorDim`: 相空间吸引子维度
- 指示系统复杂度

**Higuchi分形维度:**
- `HRV_HFD`: 复杂度与自相似性
- 值越高越复杂/不规则

**Petrosian分形维度:**
- `HRV_PFD`: 替代性复杂度度量
- 计算高效

**Katz分形维度:**
- `HRV_KFD`: 波形复杂度

### 心率碎片化

量化反映自主神经失调的异常短期波动。

- `HRV_PIP`: 拐点百分比
  - 正常：~50%，碎片化：>70%
- `HRV_IALS`: 加速/减速段平均长度的倒数
- `HRV_PSS`: 短片段百分比（<3搏动）
- `HRV_PAS`: 交替片段中的NN间期百分比

**临床意义:**
- 碎片化增加与心血管风险相关
- 超越传统HRV指标的独立预测因子

### 其他非线性指标

- `HRV_Hurst`: Hurst指数（长程依赖性）
- `HRV_LZC`: Lempel-Ziv复杂度（算法复杂度）
- `HRV_MFDFA`: 多重分形DFA指标

## 专用HRV函数

### hrv_rsa()

呼吸性窦性心律不齐 - 呼吸对心率的调制。

```python
rsa = nk.hrv_rsa(peaks, rsp_signal, sampling_rate=1000, method='porges1980')
```

**方法:**
- `'porges1980'`: Porges-Bohrer法（呼吸频率带通滤波）
- `'harrison2021'`: 峰谷RSA（每呼吸周期最大-最小心率）

**要求:**
- 同步的ECG与呼吸信号
- 至少包含多个呼吸周期

**返回:**
- `RSA`: RSA幅度（单位取决于方法）

### hrv_rqa()

递归量化分析 - 通过相空间重构分析非线性动力学。

```python
rqa = nk.hrv_rqa(peaks, sampling_rate=1000)
```

**指标:**
- `RQA_RR`: 递归率 - 系统可预测性
- `RQA_DET`: 确定性 - 形成直线的递归点占比
- `RQA_LMean`, `RQA_LMax`: 对角线平均/最大长度
- `RQA_ENTR`: 线长香农熵 - 复杂度
- `RQA_LAM`: 层流性 - 系统陷于特定状态
- `RQA_TT`: 陷获时间 - 层流状态持续时间

**应用场景:**
- 检测生理状态转变
- 评估系统确定性与随机性

## 间期处理

### intervals_process()

HRV分析前的RR间期预处理。

```python
processed_intervals = nk.intervals_process(rr_intervals, interpolate=False,
                                           interpolate_sampling_rate=1000)
```

**操作:**
- 移除生理学不可信间期
- 可选：插值为均匀采样
- 可选：去趋势消除慢变趋势

**应用场景:**
- 处理预提取的RR间期
- 清理外部设备间期数据
- 频域分析数据准备

### intervals_to_peaks()

将间期数据（RR, NN）转换为HRV分析所需的峰值索引。

```python
peaks_dict = nk.intervals_to_peaks(rr_intervals, sampling_rate=1000)
```

**应用场景:**
- 导入外部HRV设备数据
- 处理商业系统间期数据
- 间期与峰值表示转换

## 实践考量

### 最小记录时长

| 分析类型 | 最短时长 | 最优时长 |
|----------|-----------------|------------------|
| RMSSD, pNN50 | 30秒 | 5分钟 |
| SDNN | 5分钟 | 5分钟（短时），24小时（长时） |
| LF, HF功率 | 2分钟 | 5分钟 |
| VLF功率 | 5分钟 | 10+分钟 |
| ULF功率 | 24小时 | 24小时 |
| 非线性（ApEn, SampEn） | 100-300搏动 | 500+搏动 |
| DFA | 300搏动 | 1000+搏动 |

### 伪影管理

**预处理:**
```python
# 带伪影校正的R峰检测
peaks, info = nk.ecg_peaks(cleaned_ecg, sampling_rate=1000, correct_artifacts=True)

# 或手动处理间期
processed = nk.intervals_process(rr_intervals, interpolate=False)
```

**质量检查:**
- 心动图目视检查（NN间期时序）
- 识别生理学不可信间期（<300 ms或>2000 ms）
- 检查突变或漏搏
- 分析前评估信号质量

### 标准化与比较

**工作组标准（1996）:**
- 短时分析采用5分钟记录
- 推荐仰卧位控制呼吸
- 长时评估需24小时

**归一化考量:**
-

- 过度训练：HRV降低
- 恢复评估

**神经科学：**
- 情绪调节研究
- 认知负荷评估
- 脑-心轴研究

**衰老：**
- HRV随年龄增长而下降
- 复杂性指标衰退
- 需要基线参考

## 参考文献

- Task Force of the European Society of Cardiology. (1996). Heart rate variability: standards of measurement, physiological interpretation and clinical use. Circulation, 93(5), 1043-1065.
- Shaffer, F., & Ginsberg, J. P. (2017). An overview of heart rate variability metrics and norms. Frontiers in public health, 5, 258.
- Peng, C. K., Havlin, S., Stanley, H. E., & Goldberger, A. L. (1995). Quantification of scaling exponents and crossover phenomena in nonstationary heartbeat time series. Chaos, 5(1), 82-87.
- Guzik, P., Piskorski, J., Krauze, T., Wykretowicz, A., & Wysocki, H. (2006). Heart rate asymmetry by Poincaré plots of RR intervals. Biomedizinische Technik/Biomedical Engineering, 51(4), 272-275.
- Costa, M., Goldberger, A. L., & Peng, C. K. (2005). Multiscale entropy analysis of biological signals. Physical review E, 71(2), 021906.
