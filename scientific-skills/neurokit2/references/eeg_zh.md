# EEG分析与微状态

## 概述

分析脑电图（EEG）信号以获取频带功率、通道质量评估、源定位及微状态识别。NeuroKit2与MNE-Python集成，提供全面的EEG处理流程。

## 核心EEG功能

### eeg_power()

计算指定通道在标准频带上的功率。

```python
power = nk.eeg_power(eeg_data, sampling_rate=250, channels=['Fz', 'Cz', 'Pz'],
                     frequency_bands={'Delta': (0.5, 4),
                                     'Theta': (4, 8),
                                     'Alpha': (8, 13),
                                     'Beta': (13, 30),
                                     'Gamma': (30, 45)})
```

**标准频带：**
- **Delta (0.5-4 Hz)**：深度睡眠、无意识过程
- **Theta (4-8 Hz)**：嗜睡、冥想、记忆编码
- **Alpha (8-13 Hz)**：放松清醒状态、闭眼
- **Beta (13-30 Hz)**：活跃思考、专注、焦虑
- **Gamma (30-45 Hz)**：认知处理、信息整合

**返回：**
- 包含各通道×频带组合功率值的DataFrame
- 列名：`Channel_Band`（如'Fz_Alpha', 'Cz_Beta'）

**应用场景：**
- 静息态分析
- 认知状态分类
- 睡眠分期
- 冥想或神经反馈监测

### eeg_badchannels()

通过统计离群值检测识别问题通道。

```python
bad_channels = nk.eeg_badchannels(eeg_data, sampling_rate=250, bad_threshold=2)
```

**检测方法：**
- 跨通道标准差离群值
- 与其他通道的相关性
- 平坦或失效通道
- 过度噪声通道

**参数：**
- `bad_threshold`：离群值检测的Z分数阈值（默认：2）

**返回：**
- 被识别为问题通道的名称列表

**应用场景：**
- 分析前的质量控制
- 自动坏通道剔除
- 插值或排除决策

### eeg_rereference()

将电压测量值重新表达为不同参考点。

```python
rereferenced = nk.eeg_rereference(eeg_data, reference='average', robust=False)
```

**参考类型：**
- `'average'`：平均参考（所有电极均值）
- `'REST'`：参考电极标准化技术
- `'bipolar'`：电极对间差分记录
- 特定通道名：使用单电极作为参考

**常用参考：**
- **平均参考**：高密度EEG最常用
- **乳突链接**：传统临床EEG
- **顶点(Cz)**：ERP研究中有时使用
- **REST**：近似无限远参考

**返回：**
- 重参考后的EEG数据

### eeg_gfp()

计算全局场功率——各时间点所有电极的标准差。

```python
gfp = nk.eeg_gfp(eeg_data)
```

**解读：**
- 高GFP：跨脑区的强同步神经活动
- 低GFP：弱或去同步化活动
- GFP峰值：稳定地形图时刻，用于微状态检测

**应用场景：**
- 识别稳定地形模式时段
- 选择微状态分析时间点
- 事件相关电位(ERP)可视化

### eeg_diss()

测量电场构型间的地形差异度。

```python
dissimilarity = nk.eeg_diss(eeg_data1, eeg_data2, method='gfp')
```

**方法：**
- 基于GFP：归一化差异
- 空间相关性
- 余弦距离

**应用场景：**
- 不同条件间地形图比较
- 微状态转换分析
- 模板匹配

## 源定位

### eeg_source()

通过头皮记录估计大脑层面活动的源重建。

```python
sources = nk.eeg_source(eeg_data, method='sLORETA')
```

**方法：**
- `'sLORETA'`：标准化低分辨率电磁断层扫描
  - 点源零定位误差
  - 良好空间分辨率
- `'MNE'`：最小范数估计
  - 快速、成熟
  - 偏向浅层源
- `'dSPM'`：动态统计参数映射
  - 归一化MNE
- `'eLORETA'`：精确LORETA
  - 改进定位精度

**要求：**
- 前向模型（导联场矩阵）
- 共配准电极位置
- 头部模型（边界元或球体）

**返回：**
- 源空间活动估计值

### eeg_source_extract()

从特定解剖脑区提取活动。

```python
regional_activity = nk.eeg_source_extract(sources, regions=['PFC', 'MTL', 'Parietal'])
```

**区域选项：**
- 标准图谱：Desikan-Killiany, Destrieux, AAL
- 自定义ROI
- Brodmann分区

**返回：**
- 各区域时间序列
- 体素平均值或主成分

**应用场景：**
- 感兴趣区域分析
- 功能连接性
- 源层面统计

## 微状态分析

微状态是短暂（80-120毫秒）的稳定脑地形图时段，代表协调的神经网络。通常包含4-7个微状态类别（常标记为A,B,C,D），具有不同功能。

### microstates_segment()

使用聚类算法识别并提取微状态。

```python
microstates = nk.microstates_segment(eeg_data, n_microstates=4, sampling_rate=250,
                                      method='kmod', normalize=True)
```

**方法：**
- `'kmod'`（默认）：针对EEG地形优化的改进k均值
  - 极性不变聚类
  - 微状态文献最常用
- `'kmeans'`：标准k均值聚类
- `'kmedoids'`：K中心点（抗离群值更强）
- `'pca'`：主成分分析
- `'ica'`：独立成分分析
- `'aahc'`：原子化聚合层次聚类

**参数：**
- `n_microstates`：微状态类别数（通常4-7）
- `normalize`：地形图归一化（建议：True）
- `n_inits`：随机初始化次数（增加可提升稳定性）

**返回：**
- 包含以下内容的字典：
  - `'maps'`：微状态模板地形图
  - `'labels'`：各时间点微状态标签
  - `'gfp'`：全局场功率
  - `'gev'`：全局解释方差

### microstates_findnumber()

估计最优微状态数量。

```python
optimal_k = nk.microstates_findnumber(eeg_data, show=True)
```

**标准：**
- **全局解释方差(GEV)**：解释方差百分比
  - 肘部法：寻找GEV曲线拐点
  - 通常达到70-80% GEV
- **Krzanowski-Lai(KL)准则**：平衡拟合度与简约性的统计量
  - KL最大值指示最优k值

**典型范围：** 4-7个微状态
- 4微状态：经典A,B,C,D状态
- 5-7微状态：更精细分解

### microstates_classify()

根据前后及左右通道值重排微状态。

```python
classified = nk.microstates_classify(microstates)
```

**目的：**
- 跨被试标准化微状态标签
- 匹配传统A,B,C,D地形图：
  - **A**：左右走向，顶枕区
  - **B**：右左走向，额颞区
  - **C**：前后走向，额中央区
  - **D**：额中央区，前后走向（与C相反）

**返回：**
- 重排序的微状态地图与标签

### microstates_clean()

为微状态提取预处理EEG数据。

```python
cleaned_eeg = nk.microstates_clean(eeg_data, sampling_rate=250)
```

**预处理步骤：**
- 带通滤波（通常2-20 Hz）
- 伪迹剔除
- 坏通道插值
- 平均重参考

**原理：**
- 微状态反映大尺度网络活动
- 高频/低频伪迹会扭曲地形图

### microstates_peaks()

识别微状态分析的GFP峰值点。

```python
peak_indices = nk.microstates_peaks(eeg_data, sampling_rate=250)
```

**目的：**
- 通常在GFP峰值分析微状态
- 峰值代表最大稳定地形活动时刻
- 降低计算负荷与噪声敏感性

**返回：**
- GFP局部极大值索引

### microstates_static()

计算单个微状态的时间特性。

```python
static_metrics = nk.microstates_static(microstates)
```

**指标：**
- **持续时间(ms)**：各微状态平均停留时间
  - 典型值：80-120 ms
  - 反映稳定性与持续性
- **出现频率(次/秒)**：微状态出现次数
  - 各状态进入频次
- **覆盖率(%)**：各微状态占总时间百分比
  - 相对主导性
- **全局解释方差(GEV)**：各类别解释方差
  - 模板拟合质量

**返回：**
- 包含各微状态类别指标的DataFrame

**解读：**
- 持续时间变化：网络稳定性改变
- 出现频率变化：状态动态转换
- 覆盖率变化：特定网络主导性变化

### microstates_dynamic()

分析微状态间转换模式。

```python
dynamic_metrics = nk.microstates_dynamic(microstates)
```

**指标：**
- **转移矩阵**：从状态i转移到状态j的概率
  - 揭示优先序列
- **转移速率**：总体转移频率
  - 高速率：更快切换
- **熵值**：转移随机性
  - 高熵：不可预测切换
  - 低熵：固定序列
- **马尔可夫检验**：转移是否依赖历史？

**返回：**
- 包含转移统计量的字典

**应用场景：**
- 识别临床群体异常微状态序列
- 网络动态与灵活性
- 状态依赖信息处理

### microstates_plot()

可视化微状态地形图与时间进程。

```python
nk.microstates_plot(microstates, eeg_data)
```

**显示内容：**
- 各微状态类别地形图
- 带微状态标签的GFP轨迹
- 状态序列转移图
- 统计摘要

## MNE集成工具

### mne_data()

访问MNE-Python的示例数据集。

```python
raw = nk.mne_data(dataset='sample', directory=None)
```

**可用数据集：**
- `'sample'`：多模态(MEG/EEG)示例
- `'ssvep'`：稳态视觉诱发电位
- `'eegbci'`：运动想象BCI数据集

### mne_to_df() / mne_to_dict()

将MNE对象转换为NeuroKit兼容格式。

```python
df = nk.mne_to_df(raw)
data_dict = nk.mne_to_dict(epochs)
```

**应用场景：**
- 在NeuroKit2中使用MNE处理数据
- 格式转换以进行分析

### mne_channel_add() / mne_channel_extract()

管理MNE对象中的单个通道。

```python
# 提取特定通道
subset = nk.mne_channel_extract(raw, ['Fz', 'Cz', 'Pz'])

# 添加衍生通道
raw_with_eog = nk.mne_channel_add(raw, new_channel_data, ch_name='EOG')
```

### mne_crop()

按时间或样本裁剪记录。

```python
cropped = nk.mne_crop(raw, tmin=10, tmax=100)
```

### mne_templateMRI()

为源定位提供模板解剖结构。

```python
subjects_dir = nk.mne_templateMRI()
```

**应用场景：**
- 无需个体MRI的源分析
- 组水平源定位
- fsaverage模板大脑

### eeg_simulate()

生成测试用合成EEG信号。

```python
synthetic_eeg = nk.eeg_simulate(duration=60, sampling_rate=250, n_channels=32)
```

## 实践考量

### 采样率建议
- **最低**：基础功率分析需100 Hz
- **标准**：多数应用需250-500 Hz
- **高分辨率**：精细时间动态需1000+ Hz

### 记录时长
- **功率分析**：≥2分钟以获稳定估计
- **微状态**：≥2-5分钟，更长更佳
- **静息态**：通常3-10分钟
- **事件相关**：取决于试次数（≥30次/条件）

### 伪迹管理
- **眨眼伪迹**：用ICA或回归剔除
- **肌肉伪迹**：高通滤波(≥1 Hz)或手动剔除
- **坏通道**：分析前检测并插值
- **工频噪声**：50/60 Hz陷波滤波

### 最佳实践

**功率分析流程：**
```python
# 1. 数据清洗
cleaned = nk.signal_filter(eeg_data, sampling_rate=250, lowcut=0.5, highcut=45)

# 2. 识别并插值坏通道
bad = nk.eeg_badchannels(cleaned, sampling_rate=250)
# 使用MNE插值坏通道

# 3. 重参考
rereferenced = nk.eeg_rereference(cleaned, reference='average')

# 4. 计算功率
power = nk.eeg_power(rereferenced, sampling_rate=250, channels=channel_list)
```

**微状态工作流：**
```python
# 1. 预处理
cleaned = nk.microstates_clean(eeg_data, sampling_rate=250)

# 2. 确定最优状态数
optimal_k = nk.microstates_findnumber(cleaned, show=True)

# 3. 分割微状态
microstates = nk.microstates_segment(cleaned, n_microstates=optimal_k,
                                     sampling_rate=250, method='kmod')

# 4. 按标准标签分类
microstates = nk.microstates_classify(microstates)

# 5. 计算时间指标
static = nk.microstates_static(microstates)
dynamic = nk.microstates_dynamic(microstates)

# 6. 可视化
nk.microstates_plot(microstates, cleaned)
```

## 临床与科研应用

**认知神经科学：**
- 注意力、工作记忆、执行功能
- 语言处理
- 感官知觉

**临床群体：**
- 癫痫：发作检测、定位
- 阿尔茨海默症：EEG减慢、微状态改变
- 精神分裂症：微状态异常（尤其C状态）
- ADHD：theta/beta比率升高
- 抑郁症：额叶alpha不对称性

**意识研究：**
- 麻醉监测
- 意识障碍
- 睡眠分期

**神经反馈：**
- 实时频带训练
- alpha增强促进放松
- beta增强提升专注力

## 参考文献

- Michel, C. M., & Koenig, T. (2018). EEG微状态作为研究全脑神经元网络时间动态的工具：综述。Neuroimage, 180, 577-593.
- Pascual-Marqui, R. D., Michel, C. M., & Lehmann, D. (1995). 脑电活动分割为微状态：模型估计与验证。IEEE生物医学工程汇刊, 42(7), 658-665.
- Gramfort, A., Luessi, M., Larson, E., Engemann, D. A., Strohmeier, D., Brodbeck, C., ... & Hämäläinen, M. (2013). 使用MNE-Python分析MEG和EEG数据。神经科学前沿, 7, 267.
