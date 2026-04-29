# 时间序列分割

Aeon 提供将时间序列划分为具有不同特征区域的算法，用于识别变更点和边界。

## 分割算法

### 二分分割
- `BinSegmenter` - 递归二分分割
  - 在统计显著性最高的变更点迭代分割序列
  - 参数：`n_segments`, `cost_function`
  - **适用场景**：已知分段数量、存在层次结构

### 基于分类
- `ClaSPSegmenter` - 分类得分剖面法
  - 利用分类性能识别边界
  - 发现分类器能区分相邻数据的区段
  - **适用场景**：分段具有不同时序模式

### 快速模式识别
- `FLUSSSegmenter` - 快速低成本单能语义分割
  - 利用弧线交叉实现高效语义分割
  - 基于矩阵剖面技术
  - **适用场景**：大型时间序列、需快速模式发现

### 信息论方法
- `InformationGainSegmenter` - 信息增益最大化
  - 寻找使信息增益最大的边界点
  - **适用场景**：分段间存在统计差异

### 高斯建模
- `GreedyGaussianSegmenter` - 贪婪高斯近似
  - 将分段建模为高斯分布
  - 增量式添加变更点
  - **适用场景**：分段符合高斯分布

### 层次凝聚法
- `EAggloSegmenter` - 自底向上合并策略
  - 通过凝聚过程估计变更点
  - **适用场景**：需要层次化分割结构

### 隐马尔可夫模型
- `HMMSegmenter` - 维特比解码的隐马尔可夫模型
  - 基于概率状态的分割
  - **适用场景**：分段代表隐藏状态

### 维度分析法
- `HidalgoSegmenter` - 异构本征维度算法
  - 检测局部维度变化
  - **适用场景**：分段间存在维度偏移

### 基准方法
- `RandomSegmenter` - 随机变更点生成
  - **适用场景**：需要零假设基准

## 快速入门

```python
from aeon.segmentation import ClaSPSegmenter
import numpy as np

# 创建具有状态变化的时间序列
y = np.concatenate([
    np.sin(np.linspace(0, 10, 100)),      # 片段1
    np.cos(np.linspace(0, 10, 100)),      # 片段2
    np.sin(2 * np.linspace(0, 10, 100))   # 片段3
])

# 分割时间序列
segmenter = ClaSPSegmenter()
change_points = segmenter.fit_predict(y)

print(f"检测到的变更点: {change_points}")
```

## 输出格式

分割器返回变更点索引：

```python
# change_points = [100, 200]  # 分段边界
# 将序列划分为: [0:100], [100:200], [200:end]
```

## 算法选择指南

- **速度优先**：FLUSSSegmenter, BinSegmenter
- **精度优先**：ClaSPSegmenter, HMMSegmenter
- **已知分段数量**：使用n_segments参数的BinSegmenter
- **未知分段数量**：ClaSPSegmenter, InformationGainSegmenter
- **模式变化检测**：FLUSSSegmenter, ClaSPSegmenter
- **统计差异检测**：InformationGainSegmenter, GreedyGaussianSegmenter
- **状态转换识别**：HMMSegmenter

## 典型应用场景

### 状态变化检测
识别时间序列行为根本性变化：

```python
from aeon.segmentation import InformationGainSegmenter

segmenter = InformationGainSegmenter(k=3)  # 最多3个变更点
change_points = segmenter.fit_predict(stock_prices)
```

### 活动分段
划分传感器数据为不同活动：

```python
from aeon.segmentation import ClaSPSegmenter

segmenter = ClaSPSegmenter()
boundaries = segmenter.fit_predict(accelerometer_data)
```

### 季节边界识别
发现时间序列中的季节转换：

```python
from aeon.segmentation import HMMSegmenter

segmenter = HMMSegmenter(n_states=4)  # 4个季节状态
segments = segmenter.fit_predict(temperature_data)
```

## 评估指标

使用分割质量评估指标：

```python
from aeon.benchmarking.metrics.segmentation import (
    count_error,
    hausdorff_error
)

# 数量误差：变更点数量差异
count_err = count_error(y_true, y_pred)

# 豪斯多夫距离：预测点与真实点间最大距离
hausdorff_err = hausdorff_error(y_true, y_pred)
```

## 最佳实践

1. **数据归一化**：避免尺度差异主导变更检测
2. **选择合适指标**：不同算法优化不同标准
3. **验证分段结果**：可视化确认边界有效性
4. **噪声处理**：分割前考虑平滑处理
5. **利用领域知识**：已知分段数量时优先使用
6. **参数调优**：调整敏感度参数（阈值/惩罚项）

## 可视化

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(12, 4))
plt.plot(y, label='时间序列')
for cp in change_points:
    plt.axvline(cp, color='r', linestyle='--', label='变更点')
plt.legend()
plt.show()
```
