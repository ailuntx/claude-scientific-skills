# 信号处理

## 概述

PyOpenMS 提供处理原始质谱数据的算法，包括平滑、滤波、峰检测、质心化、归一化和解卷积。

## 算法模式

大多数信号处理算法遵循标准模式：

```python
import pyopenms as ms

# 1. 创建算法实例
algo = ms.AlgorithmName()

# 2. 获取并修改参数
params = algo.getParameters()
params.setValue("parameter_name", value)
algo.setParameters(params)

# 3. 应用至数据
algo.filterExperiment(exp)  # 或 filterSpectrum(spec)
```

## 平滑处理

### 高斯滤波器

应用高斯平滑降低噪声：

```python
# 创建高斯滤波器
gaussian = ms.GaussFilter()

# 配置参数
params = gaussian.getParameters()
params.setValue("gaussian_width", 0.2)  # m/z或RT单位宽度
params.setValue("ppm_tolerance", 10.0)  # 用于m/z维度
params.setValue("use_ppm_tolerance", "true")
gaussian.setParameters(params)

# 应用于实验数据
gaussian.filterExperiment(exp)

# 或应用于单张谱图
spec = exp.getSpectrum(0)
gaussian.filterSpectrum(spec)
```

### Savitzky-Golay滤波器

保留峰形的多项式平滑：

```python
# 创建Savitzky-Golay滤波器
sg_filter = ms.SavitzkyGolayFilter()

# 配置参数
params = sg_filter.getParameters()
params.setValue("frame_length", 11)  # 窗口大小(需为奇数)
params.setValue("polynomial_order", 4)  # 多项式阶数
sg_filter.setParameters(params)

# 应用平滑
sg_filter.filterExperiment(exp)
```

## 峰检测与质心化

### 高分辨率峰检测器

检测高分辨率数据中的峰：

```python
# 创建峰检测器
peak_picker = ms.PeakPickerHiRes()

# 配置参数
params = peak_picker.getParameters()
params.setValue("signal_to_noise", 3.0)  # 信噪比阈值
params.setValue("spacing_difference", 1.5)  # 最小峰间距
peak_picker.setParameters(params)

# 检测峰
exp_picked = ms.MSExperiment()
peak_picker.pickExperiment(exp, exp_picked)
```

### 连续小波变换峰检测器

基于连续小波变换的峰检测：

```python
# 创建CWT峰检测器
cwt_picker = ms.PeakPickerCWT()

# 配置参数
params = cwt_picker.getParameters()
params.setValue("signal_to_noise", 1.0)
params.setValue("peak_width", 0.15)  # 预期峰宽
cwt_picker.setParameters(params)

# 检测峰
cwt_picker.pickExperiment(exp, exp_picked)
```

## 归一化

### 归一化器

对谱图内峰强度进行归一化：

```python
# 创建归一化器
normalizer = ms.Normalizer()

# 配置归一化方法
params = normalizer.getParameters()
params.setValue("method", "to_one")  # 选项: "to_one", "to_TIC"
normalizer.setParameters(params)

# 应用归一化
normalizer.filterExperiment(exp)
```

## 峰过滤

### 阈值修剪器

移除低于强度阈值的峰：

```python
# 创建阈值过滤器
mower = ms.ThresholdMower()

# 配置阈值
params = mower.getParameters()
params.setValue("threshold", 1000.0)  # 绝对强度阈值
mower.setParameters(params)

# 应用过滤器
mower.filterExperiment(exp)
```

### 窗口修剪器

在滑动窗口中仅保留最高峰：

```python
# 创建窗口修剪器
window_mower = ms.WindowMower()

# 配置参数
params = window_mower.getParameters()
params.setValue("windowsize", 50.0)  # m/z窗口大小
params.setValue("peakcount", 2)  # 每窗口保留前N个峰
window_mower.setParameters(params)

# 应用过滤器
window_mower.filterExperiment(exp)
```

### N最大峰过滤器

仅保留强度最高的N个峰：

```python
# 创建N最大过滤器
n_largest = ms.NLargest()

# 配置参数
params = n_largest.getParameters()
params.setValue("n", 200)  # 保留200个最强峰
n_largest.setParameters(params)

# 应用过滤器
n_largest.filterExperiment(exp)
```

## 基线校正

### 形态学滤波器

使用形态学操作移除基线：

```python
# 创建形态学滤波器
morph_filter = ms.MorphologicalFilter()

# 配置参数
params = morph_filter.getParameters()
params.setValue("struc_elem_length", 3.0)  # 结构元素尺寸
params.setValue("method", "tophat")  # 方法: "tophat", "bothat", "erosion", "dilation"
morph_filter.setParameters(params)

# 应用滤波器
morph_filter.filterExperiment(exp)
```

## 谱图合并

### 谱图合并器

将多张谱图合并为单张谱图：

```python
# 创建合并器
merger = ms.SpectraMerger()

# 配置参数
params = merger.getParameters()
params.setValue("average_gaussian:spectrum_type", "profile")
params.setValue("average_gaussian:rt_FWHM", 5.0)  # RT窗口
merger.setParameters(params)

# 合并谱图
merger.mergeSpectraBlockWise(exp)
```

## 解卷积

### 电荷解卷积

确定电荷态并转换为中性质量：

```python
# 创建特征解卷积器
deconvoluter = ms.FeatureDeconvolution()

# 配置参数
params = deconvoluter.getParameters()
params.setValue("charge_min", 1)
params.setValue("charge_max", 4)
params.setValue("potential_charge_states", "1,2,3,4")
deconvoluter.setParameters(params)

# 应用解卷积
feature_map_out = ms.FeatureMap()
deconvoluter.compute(exp, feature_map, feature_map_out, ms.ConsensusMap())
```

### 同位素解卷积

移除同位素模式：

```python
# 创建同位素小波变换器
isotope_wavelet = ms.IsotopeWaveletTransform()

# 配置参数
params = isotope_wavelet.getParameters()
params.setValue("max_charge", 3)
params.setValue("intensity_threshold", 10.0)
isotope_wavelet.setParameters(params)

# 应用变换
isotope_wavelet.transform(exp)
```

## 保留时间对齐

### 图谱对齐

跨多个实验进行保留时间对齐：

```python
# 创建图谱对齐器
aligner = ms.MapAlignmentAlgorithmPoseClustering()

# 加载多个实验
exp1 = ms.MSExperiment()
exp2 = ms.MSExperiment()
ms.MzMLFile().load("run1.mzML", exp1)
ms.MzMLFile().load("run2.mzML", exp2)

# 创建参考
reference = ms.MSExperiment()

# 对齐实验
transformations = []
aligner.align(exp1, exp2, transformations)

# 应用变换
transformer = ms.MapAlignmentTransformer()
transformer.transformRetentionTimes(exp2, transformations[0])
```

## 质量校准

### 内标校准

使用已知参考质量进行质量轴校准：

```python
# 创建内标校准器
calibration = ms.InternalCalibration()

# 设置参考质量
reference_masses = [500.0, 1000.0, 1500.0]  # 已知m/z值

# 执行校准
calibration.calibrate(exp, reference_masses)
```

## 质量控制

### 谱图统计

计算质量指标：

```python
# 获取谱图
spec = exp.getSpectrum(0)

# 计算统计量
mz, intensity = spec.get_peaks()

# 总离子流
tic = sum(intensity)

# 基峰
base_peak_intensity = max(intensity)
base_peak_mz = mz[intensity.argmax()]

print(f"TIC: {tic}")
print(f"基峰: {base_peak_mz} m/z 强度 {base_peak_intensity}")
```

## 谱图预处理流程

### 完整预处理示例

```python
import pyopenms as ms

def preprocess_experiment(input_file, output_file):
    """完整预处理流程"""

    # 加载数据
    exp = ms.MSExperiment()
    ms.MzMLFile().load(input_file, exp)

    # 1. 高斯平滑
    gaussian = ms.GaussFilter()
    gaussian.filterExperiment(exp)

    # 2. 峰检测
    picker = ms.PeakPickerHiRes()
    exp_picked = ms.MSExperiment()
    picker.pickExperiment(exp, exp_picked)

    # 3. 强度归一化
    normalizer = ms.Normalizer()
    params = normalizer.getParameters()
    params.setValue("method", "to_TIC")
    normalizer.setParameters(params)
    normalizer.filterExperiment(exp_picked)

    # 4. 低强度峰过滤
    mower = ms.ThresholdMower()
    params = mower.getParameters()
    params.setValue("threshold", 10.0)
    mower.setParameters(params)
    mower.filterExperiment(exp_picked)

    # 保存处理数据
    ms.MzMLFile().store(output_file, exp_picked)

    return exp_picked

# 执行流程
exp_processed = preprocess_experiment("raw_data.mzML", "processed_data.mzML")
```

## 最佳实践

### 参数优化

在代表性数据上测试参数：

```python
# 尝试不同高斯宽度
widths = [0.1, 0.2, 0.5]

for width in widths:
    exp_test = ms.MSExperiment()
    ms.MzMLFile().load("test_data.mzML", exp_test)

    gaussian = ms.GaussFilter()
    params = gaussian.getParameters()
    params.setValue("gaussian_width", width)
    gaussian.setParameters(params)
    gaussian.filterExperiment(exp_test)

    # 评估质量
    # ... 添加评估代码 ...
```

### 保留原始数据

保留原始数据用于比较：

```python
# 加载原始数据
exp_original = ms.MSExperiment()
ms.MzMLFile().load("data.mzML", exp_original)

# 创建处理副本
exp_processed = ms.MSExperiment(exp_original)

# 处理副本
gaussian = ms.GaussFilter()
gaussian.filterExperiment(exp_processed)

# 原始数据保持不变
```

### 轮廓图与质心图数据

处理前检查数据类型：

```python
# 检查谱图是否质心化
spec = exp.getSpectrum(0)

if spec.isSorted():
    # 可能为质心数据
    print("质心数据")
else:
    # 可能为轮廓图数据
    print("轮廓图数据 - 需应用峰检测")
```
