# 复杂性与熵分析

## 概述

复杂性度量用于量化时间序列信号的不规则性、不可预测性和多尺度结构。NeuroKit2 提供全面的熵、分形维度和非线性动力学指标，用于评估生理信号的复杂性。

## 主要函数

### complexity()

同步计算多个复杂性指标以进行探索性分析。

```python
complexity_indices = nk.complexity(signal, sampling_rate=1000, show=False)
```

**返回值：**
- 包含跨类别多种复杂性指标的 DataFrame：
  - 熵指数
  - 分形维度
  - 非线性动力学指标
  - 信息论度量

**应用场景：**
- 识别相关指标的探索性分析
- 全面信号表征
- 跨信号比较研究

## 参数优化

计算复杂性指标前需确定最优嵌入参数：

### complexity_delay()

确定相空间重构的最优时间延迟 (τ)。

```python
optimal_tau = nk.complexity_delay(signal, delay_max=100, method='fraser1986', show=False)
```

**方法：**
- `'fraser1986'`: 互信息首次最小值
- `'theiler1990'`: 自相关首次过零点
- `'casdagli1991'`: Cao 方法

**用途：** 熵计算、吸引子重构的嵌入延迟

### complexity_dimension()

确定最优嵌入维度 (m)。

```python
optimal_m = nk.complexity_dimension(signal, delay=None, dimension_max=20,
                                    method='afn', show=False)
```

**方法：**
- `'afn'`: 平均虚假最近邻
- `'fnn'`: 虚假最近邻
- `'correlation'`: 相关维度饱和法

**用途：** 熵计算、相空间重构

### complexity_tolerance()

确定熵度量的最优容差 (r)。

```python
optimal_r = nk.complexity_tolerance(signal, method='sd', show=False)
```

**方法：**
- `'sd'`: 基于标准差（典型值 0.1-0.25 × SD）
- `'maxApEn'`: 最大化近似熵
- `'recurrence'`: 基于递归率

**用途：** 近似熵、样本熵

### complexity_k()

确定 Higuchi 分形维度的最优 k 参数。

```python
optimal_k = nk.complexity_k(signal, k_max=20, show=False)
```

**用途：** Higuchi 分形维度计算

## 熵度量

熵量化随机性、不可预测性和信息量。

### entropy_shannon()

香农熵 - 经典信息论度量。

```python
shannon_entropy = nk.entropy_shannon(signal)
```

**解读：**
- 值越高：更随机，更不可预测
- 值越低：更规则，可预测
- 单位：比特（信息量）

**应用场景：**
- 通用随机性评估
- 信息量分析
- 信号不规则性

### entropy_approximate()

近似熵 (ApEn) - 模式规则性度量。

```python
apen = nk.entropy_approximate(signal, delay=1, dimension=2, tolerance='sd')
```

**参数：**
- `delay`: 时间延迟 (τ)
- `dimension`: 嵌入维度 (m)
- `tolerance`: 相似性阈值 (r)

**解读：**
- 低 ApEn：更规则，自相似模式
- 高 ApEn：更复杂，不规则
- 对信号长度敏感（建议 ≥100-300 点）

**生理应用：**
- 心率变异性 (HRV)：心脏病患者 ApEn 降低
- 脑电图 (EEG)：神经系统疾病中 ApEn 改变

### entropy_sample()

样本熵 (SampEn) - 改进的近似熵。

```python
sampen = nk.entropy_sample(signal, delay=1, dimension=2, tolerance='sd')
```

**优于 ApEn 的特点：**
- 对信号长度依赖性更低
- 跨记录一致性更高
- 无自匹配偏差

**解读：**
- 与 ApEn 相同但更可靠
- 多数应用中优先使用

**典型值范围：**
- HRV：0.5-2.5（依赖场景）
- EEG：0.3-1.5

### entropy_multiscale()

多尺度熵 (MSE) - 跨时间尺度的复杂性。

```python
mse = nk.entropy_multiscale(signal, scale=20, dimension=2, tolerance='sd',
                            method='MSEn', show=False)
```

**方法：**
- `'MSEn'`: 多尺度样本熵
- `'MSApEn'`: 多尺度近似熵
- `'CMSE'`: 复合多尺度熵
- `'RCMSE'`: 精细化复合多尺度熵

**解读：**
- 不同粗粒度尺度下的熵值
- 健康/复杂系统：在多个尺度保持高熵
- 疾病/简化系统：熵值降低（尤其大尺度）

**应用场景：**
- 区分真实复杂性与随机性
- 白噪声：跨尺度恒定
- 粉红噪声/复杂性：跨尺度结构化变化

### entropy_fuzzy()

模糊熵 - 采用模糊隶属函数。

```python
fuzzen = nk.entropy_fuzzy(signal, delay=1, dimension=2, tolerance='sd', r=0.2)
```

**优势：**
- 对含噪信号更稳定
- 模式匹配采用模糊边界
- 短信号表现更佳

### entropy_permutation()

排列熵 - 基于序数模式。

```python
perment = nk.entropy_permutation(signal, delay=1, dimension=3)
```

**方法：**
- 将信号编码为序数模式（排列）
- 统计模式频率
- 抗噪声和非平稳性强

**解读：**
- 值越低：序数结构更规则
- 值越高：排序更随机

**应用场景：**
- EEG 分析
- 麻醉深度监测
- 快速计算

### entropy_spectral()

谱熵 - 基于功率谱。

```python
spec_ent = nk.entropy_spectral(signal, sampling_rate=1000, bands=None)
```

**方法：**
- 功率谱的归一化香农熵
- 量化频率分布规则性

**解读：**
- 0：单一频率（纯音）
- 1：白噪声（平坦频谱）

**应用场景：**
- EEG：谱分布随状态变化
- 麻醉监测

### entropy_svd()

奇异值分解熵。

```python
svd_ent = nk.entropy_svd(signal, delay=1, dimension=2)
```

**方法：**
- 轨迹矩阵的 SVD 分解
- 奇异值分布的熵

**应用场景：**
- 吸引子复杂性
- 确定性 vs 随机动力学

### entropy_differential()

微分熵 - 香农熵的连续模拟。

```python
diff_ent = nk.entropy_differential(signal)
```

**用途：** 连续概率分布

### 其他熵度量

**Tsallis 熵：**
```python
tsallis = nk.entropy_tsallis(signal, q=2)
```
- 含参数 q 的广义熵
- q=1 时退化为香农熵

**Rényi 熵：**
```python
renyi = nk.entropy_renyi(signal, alpha=2)
```
- 含参数 α 的广义熵

**其他专用熵：**
- `entropy_attention()`: 注意力熵
- `entropy_grid()`: 网格熵
- `entropy_increment()`: 增量熵
- `entropy_slope()`: 斜率熵
- `entropy_dispersion()`: 散布熵
- `entropy_symbolicdynamic()`: 符号动力学熵
- `entropy_range()`: 范围熵
- `entropy_phase()`: 相位熵
- `entropy_quadratic()`, `entropy_cumulative_residual()`, `entropy_rate()`: 专用变体

## 分形维度度量

分形维度表征自相似性与粗糙度。

### fractal_katz()

Katz 分形维度 - 波形复杂性。

```python
kfd = nk.fractal_katz(signal)
```

**解读：**
- 1：直线
- >1：粗糙度与复杂性递增
- 典型范围：1.0-2.0

**优势：**
- 简单快速计算
- 无需参数调优

### fractal_higuchi()

Higuchi 分形维度 - 自相似性。

```python
hfd = nk.fractal_higuchi(signal, k_max=10)
```

**方法：**
- 从原始信号构造 k 个新时间序列
- 通过长度-尺度关系估计维度

**解读：**
- 高 HFD：更复杂、不规则
- 低 HFD：更平滑、规则

**应用场景：**
- EEG 复杂性
- HRV 分析
- 癫痫检测

### fractal_petrosian()

Petrosian 分形维度 - 快速估计。

```python
pfd = nk.fractal_petrosian(signal)
```

**优势：**
- 计算快速
- 直接计算（无需曲线拟合）

### fractal_sevcik()

Sevcik 分形维度 - 归一化波形复杂性。

```python
sfd = nk.fractal_sevcik(signal)
```

### fractal_nld()

归一化长度密度 - 基于曲线长度的度量。

```python
nld = nk.fractal_nld(signal)
```

### fractal_psdslope()

功率谱密度斜率 - 频域分形度量。

```python
slope = nk.fractal_psdslope(signal, sampling_rate=1000)
```

**方法：**
- 对数功率谱的线性拟合
- 斜率 β 关联分形维度

**解读：**
- β ≈ 0：白噪声（随机）
- β ≈ -1：粉红噪声 (1/f，复杂)
- β ≈ -2：布朗噪声（布朗运动）

### fractal_hurst()

Hurst 指数 - 长程依赖性。

```python
hurst = nk.fractal_hurst(signal, show=False)
```

**解读：**
- H < 0.5：反持续性（均值回归）
- H = 0.5：随机游走（白噪声）
- H > 0.5：持续性（趋势性，长记忆）

**应用场景：**
- 评估长期相关性
- 金融时间序列
- HRV 分析

### fractal_correlation()

相关维度 - 吸引子维度。

```python
corr_dim = nk.fractal_correlation(signal, delay=1, dimension=10, radius=64)
```

**方法：**
- Grassberger-Procaccia 算法
- 估计相空间吸引子维度

**解读：**
- 低维度：确定性低维混沌
- 高维度：高维混沌或噪声

### fractal_dfa()

去趋势波动分析 - 标度指数。

```python
dfa_alpha = nk.fractal_dfa(signal, multifractal=False, q=2, show=False)
```

**解读：**
- α < 0.5：反相关
- α = 0.5：不相关（白噪声）
- α = 1.0：1/f 噪声（粉红噪声，健康复杂性）
- α = 1.5：布朗噪声
- α > 1.0：持续性长程相关

**HRV 应用：**
- α1（短期，4-11 次心跳）：反映自主神经调节
- α2（长期，>11 次心跳）：长程相关性
- α1 降低：心脏病理状态

### fractal_mfdfa()

多重分形 DFA - 多尺度分形特性。

```python
mfdfa_results = nk.fractal_mfdfa(signal, q=None, show=False)
```

**方法：**
- 将 DFA 扩展至多 q 阶
- 表征多重分形谱

**返回值：**
- 广义 Hurst 指数 h(q)
- 多重分形谱 f(α)
- 宽度指示多重分形强度

**应用场景：**
- 检测多重分形结构
- 健康 vs 疾病的 HRV 多重分形性
- EEG 多尺度动力学

### fractal_tmf()

多重分形非线性 - 偏离单分形的程度。

```python
tmf = nk.fractal_tmf(signal)
```

**解读：**
- 量化偏离简单标度的程度
- 值越高：多重分形结构越显著

### fractal_density()

密度分形维度。

```python
density_fd = nk.fractal_density(signal)
```

### fractal_linelength()

线长度 - 总变差度量。

```python
linelength = nk.fractal_linelength(signal)
```

**应用场景：**
- 简单复杂性代理指标
- EEG 癫痫发作检测

## 非线性动力学

### complexity_lyapunov()

最大 Lyapunov 指数 - 混沌与发散性。

```python
lyap = nk.complexity_lyapunov(signal, delay=None, dimension=None,
                              sampling_rate=1000, show=False)
```

**解读：**
- λ < 0：稳定不动点
- λ = 0：周期轨道
- λ > 0：混沌（相邻轨迹指数发散）

**应用场景：**
- 检测生理信号混沌
- HRV：正 Lyapunov 指数提示非线性动力学
- EEG：癫痫检测（发作前 λ 降低）

### complexity_lempelziv()

Lempel-Ziv 复杂度 - 算法复杂度。

```python
lz = nk.complexity_lempelziv(signal, symbolize='median')
```

**方法：**
- 统计独特模式数量
- 随机性的粗粒度度量

**解读：**
- 值越低：重复性、可预测模式
- 值越高：多样性、不可预测模式

**应用场景：**
- EEG：意识水平、麻醉深度
- HRV：自主神经复杂性

### complexity_rqa()

递归量化分析 - 相空间递归性。

```python
rqa_indices = nk.complexity_rqa(signal, delay=1, dimension=3, tolerance='sd')
```

**指标：**
- **递归率 (RR)**：递归状态百分比
- **确定性 (DET)**：线段中递归点比例
- **层流性 (LAM)**：垂直结构（层流状态）比例
- **陷获时间 (TT)**：平均垂直线长度
- **最长对角线/垂直线**：系统可预测性
- **熵 (ENTR)**：线段长度分布的香农熵

**解读：**
- 高 DET：确定性动力学
- 高 LAM：系统陷于特定状态
- 低 RR：随机、非递归动力学

**应用场景：**
- 检测系统动力学转变
- 生理状态变化
- 非线性时间序列分析

### complexity_hjorth()

Hjorth 参数 - 时域复杂性。

```python
hjorth = nk.complexity_hjorth(signal)
```

**指标：**
- **活跃度 (Activity)**：信号方差
- **迁移率 (Mobility)**：导数标准差与信号标准差之比
- **复杂性 (Complexity)**：导数引起的迁移率变化

**应用场景：**
- EEG 特征提取
- 癫痫发作检测
- 信号表征

### complexity_decorrelation()

去相关时间 - 记忆持续时间。

```python
decorr_time = nk.complexity_decorrelation(signal, show=False)
```

**解读：**
- 自相关降至阈值以下的时间滞后
- 值越小：快速波动，短记忆
- 值越大：慢速波动，长记忆

### complexity_relativeroughness()

相对粗糙度 - 平滑度度量。

```python
roughness = nk.complexity_relativeroughness(signal)
```

## 信息论

### fisher_information()

Fisher 信息 - 有序性度量。

```python
fisher = nk.fisher_information(signal, delay=1, dimension=2)
```

**解读：**
- 值越高：有序、结构化
- 值越低：无序、随机

**应用场景：**
- 与香农熵结合（Fisher-Shannon 平面）
- 表征系统复杂性

### fishershannon_information()

Fisher-Shannon 信息积。

```python
fs = nk.fishershannon_information(signal)
```

**方法：**
- Fisher 信息与香农熵的乘积
- 表征有序-无序平衡

### mutual_information()

互信息 - 变量间共享信息量。

```python
mi = nk.mutual_in

| DFA | 500 | 1000+ |
| Lyapunov | 1000 | 5000+ |
| 相关维数 | 1000 | 5000+ |

### 参数选择

**通用准则：**
- 优先使用参数优化函数
- 或采用常规默认值：
  - 延迟（τ）：心率变异性（HRV）取1，脑电图（EEG）取自相关函数首个极小值
  - 维度（m）：通常取2-3
  - 容差（r）：常用0.2×标准差（SD）

**敏感性：**
- 结果可能对参数敏感
- 需报告所用参数
- 建议进行敏感性分析

### 标准化与预处理

**标准化处理：**
- 多数指标对信号幅值敏感
- 通常推荐Z分数标准化
- 可能需要进行去趋势处理

**平稳性：**
- 部分指标假设信号平稳
- 需通过统计检验验证（如ADF检验）
- 对非平稳信号进行分段处理

### 结果解读

**情境依赖性：**
- 不存在普适的"好"或"坏"复杂度
- 应在受试者内或组间进行比较
- 需结合生理学背景分析

**复杂度与随机性：**
- 最大熵 ≠ 最大复杂度
- 真实复杂度：结构化变异
- 白噪声：高熵但低复杂度（多尺度熵可区分）

## 应用场景

**心血管领域：**
- 心率变异性（HRV）复杂度：心脏病及衰老过程中降低
- DFA α1：心肌梗死后预后标志物

**神经科学：**
- 脑电图（EEG）复杂度：反映意识状态、麻醉深度
- 熵：阿尔茨海默症、癫痫、睡眠分期评估
- 排列熵：麻醉状态监测

**心理学：**
- 抑郁焦虑伴随复杂度下降
- 压力状态下规律性增强

**衰老研究：**
- 多系统呈现"复杂度衰减"现象
- 多尺度复杂度降低

**临界转变：**
- 状态转换前出现复杂度变化
- 临界减速可作为早期预警信号

## 参考文献

- Pincus, S. M. (1991). 近似熵作为系统复杂度的度量指标.《美国国家科学院院刊》,88(6),2297-2301.
- Richman, J. S., & Moorman, J. R. (2000). 基于近似熵与样本熵的生理时间序列分析.《美国生理学杂志-心脏与循环生理》,278(6),H2039-H2049.
- Peng, C. K., et al. (1995). 非平稳心跳时间序列标度指数与交叉现象的量化.《混沌》,5(1),82-87.
- Costa, M., Goldberger, A. L., & Peng, C. K. (2005). 生物信号的多尺度熵分析.《物理评论E》,71(2),021906.
- Grassberger, P., & Procaccia, I. (1983). 奇异吸引子的奇异度测量.《非线性现象物理D辑》,9(1-2),189-208.
