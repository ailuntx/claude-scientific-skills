# 数据集与基准测试

Aeon 提供了用于加载数据集和基准测试时间序列算法的全面工具。

## 数据集加载

### 任务特定加载器

**分类数据集**：
```python
from aeon.datasets import load_classification

# 加载训练/测试集
X_train, y_train = load_classification("GunPoint", split="train")
X_test, y_test = load_classification("GunPoint", split="test")

# 加载整个数据集
X, y = load_classification("GunPoint")
```

**回归数据集**：
```python
from aeon.datasets import load_regression

X_train, y_train = load_regression("Covid3Month", split="train")
X_test, y_test = load_regression("Covid3Month", split="test")

# 批量下载
from aeon.datasets import download_all_regression
download_all_regression()  # 下载 Monash TSER 存档
```

**预测数据集**：
```python
from aeon.datasets import load_forecasting

# 从 forecastingdata.org 加载
y, X = load_forecasting("airline", return_X_y=True)
```

**异常检测数据集**：
```python
from aeon.datasets import load_anomaly_detection

X, y = load_anomaly_detection("NAB_realKnownCause")
```

### 文件格式加载器

**从 .ts 文件加载**：
```python
from aeon.datasets import load_from_ts_file

X, y = load_from_ts_file("path/to/data.ts")
```

**从 .tsf 文件加载**：
```python
from aeon.datasets import load_from_tsf_file

df, metadata = load_from_tsf_file("path/to/data.tsf")
```

**从 ARFF 文件加载**：
```python
from aeon.datasets import load_from_arff_file

X, y = load_from_arff_file("path/to/data.arff")
```

**从 TSV 文件加载**：
```python
from aeon.datasets import load_from_tsv_file

data = load_from_tsv_file("path/to/data.tsv")
```

**加载 TimeEval CSV**：
```python
from aeon.datasets import load_from_timeeval_csv_file

X, y = load_from_timeeval_csv_file("path/to/timeeval.csv")
```

### 写入数据集

**写入 .ts 格式**：
```python
from aeon.datasets import write_to_ts_file

write_to_ts_file(X, "output.ts", y=y, problem_name="MyDataset")
```

**写入 ARFF 格式**：
```python
from aeon.datasets import write_to_arff_file

write_to_arff_file(X, "output.arff", y=y)
```

## 内置数据集

Aeon 包含多个用于快速测试的基准数据集：

### 分类
- `ArrowHead` - 形状分类
- `GunPoint` - 手势识别
- `ItalyPowerDemand` - 能源需求
- `BasicMotions` - 运动分类
- 以及来自 UCR/UEA 档案的 100+ 数据集

### 回归
- `Covid3Month` - COVID 预测
- 来自 Monash TSER 档案的各种数据集

### 分割
- 时间序列分割数据集
- 人类活动数据
- 传感器数据集合

### 特殊集合
- `RehabPile` - 康复数据（分类与回归）

## 数据集元数据

获取数据集信息：

```python
from aeon.datasets import get_dataset_meta_data

metadata = get_dataset_meta_data("GunPoint")
print(metadata)
# {'n_train': 50, 'n_test': 150, 'length': 150, 'n_classes': 2, ...}
```

## 基准测试工具

### 加载已发布结果

访问预计算的基准测试结果：

```python
from aeon.benchmarking import get_estimator_results

# 获取特定算法在数据集上的结果
results = get_estimator_results(
    estimator_name="ROCKET",
    dataset_name="GunPoint"
)

# 获取数据集上所有可用的估计器
estimators = get_available_estimators("GunPoint")
```

### 重采样策略

创建可复现的训练/测试划分：

```python
from aeon.benchmarking import stratified_resample

# 分层重采样，保持类别分布
X_train, X_test, y_train, y_test = stratified_resample(
    X, y,
    random_state=42,
    test_size=0.3
)
```

### 性能指标

针对时间序列任务的专用指标：

**异常检测指标**：
```python
from aeon.benchmarking.metrics.anomaly_detection import (
    range_precision,
    range_recall,
    range_f_score,
    range_roc_auc_score
)

# 用于窗口检测的基于范围的指标
precision = range_precision(y_true, y_pred, alpha=0.5)
recall = range_recall(y_true, y_pred, alpha=0.5)
f1 = range_f_score(y_true, y_pred, alpha=0.5)
auc = range_roc_auc_score(y_true, y_scores)
```

**聚类指标**：
```python
from aeon.benchmarking.metrics.clustering import clustering_accuracy

# 带标签匹配的聚类准确率
accuracy = clustering_accuracy(y_true, y_pred)
```

**分割指标**：
```python
from aeon.benchmarking.metrics.segmentation import (
    count_error,
    hausdorff_error
)

# 变化点数量的差异
count_err = count_error(y_true, y_pred)

# 预测变化点与真实变化点之间的最大距离
hausdorff_err = hausdorff_error(y_true, y_pred)
```

### 统计检验

算法比较的事后分析：

```python
from aeon.benchmarking import (
    nemenyi_test,
    wilcoxon_test
)

# 针对多个算法的 Nemenyi 检验
results = nemenyi_test(scores_matrix, alpha=0.05)

# 成对 Wilcoxon 符号秩检验
stat, p_value = wilcoxon_test(scores_alg1, scores_alg2)
```

## 基准测试集合

### UCR/UEA 时间序列档案库

访问全面的基准测试仓库：

```python
# 分类：112 个单变量 + 30 个多变量数据集
X_train, y_train = load_classification("Chinatown", split="train")

# 自动从 timeseriesclassification.com 下载
```

### Monash 预测档案库

```python
# 加载预测数据集
y = load_forecasting("nn5_daily", return_X_y=False)
```

### 已发布的基准测试结果

来自主要竞赛的预计算结果：

- 2017 单变量竞赛
- 2021 多变量分类
- 2023 单变量竞赛

## 工作流示例

完整的基准测试工作流：

```python
from aeon.datasets import load_classification
from aeon.classification.convolution_based import RocketClassifier
from aeon.benchmarking import get_estimator_results
from sklearn.metrics import accuracy_score
import numpy as np

# 加载数据集
dataset_name = "GunPoint"
X_train, y_train = load_classification(dataset_name, split="train")
X_test, y_test = load_classification(dataset_name, split="test")

# 训练模型
clf = RocketClassifier(n_kernels=10000, random_state=42)
clf.fit(X_train, y_train)
y_pred = clf.predict(X_test)

# 评估
accuracy = accuracy_score(y_test, y_pred)
print(f"准确率: {accuracy:.4f}")

# 与已发布结果比较
published = get_estimator_results("ROCKET", dataset_name)
print(f"已发布 ROCKET 准确率: {published['accuracy']:.4f}")
```

## 最佳实践

### 1. 使用标准划分

为保证可复现性，使用提供的训练/测试划分：

```python
# 良好：使用标准划分
X_train, y_train = load_classification("GunPoint", split="train")
X_test, y_test = load_classification("GunPoint", split="test")

# 避免：创建自定义划分
X, y = load_classification("GunPoint")
X_train, X_test, y_train, y_test = train_test_split(X, y)
```

### 2. 设置随机种子

确保可复现性：

```python
clf = RocketClassifier(random_state=42)
results = stratified_resample(X, y, random_state=42)
```

### 3. 报告多个指标

不要依赖单一指标：

```python
from sklearn.metrics import accuracy_score, f1_score, precision_score

accuracy = accuracy_score(y_test, y_pred)
f1 = f1_score(y_test, y_pred, average='weighted')
precision = precision_score(y_test, y_pred, average='weighted')
```

### 4. 交叉验证

用于在小数据集上进行鲁棒评估：

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(
    clf, X_train, y_train,
    cv=5,
    scoring='accuracy'
)
print(f"交叉验证准确率: {scores.mean():.4f} (+/- {scores.std():.4f})")
```

### 5. 与基线比较

始终与简单基线比较：

```python
from aeon.classification.distance_based import KNeighborsTimeSeriesClassifier

# 简单基线：使用欧氏距离的 1-NN
baseline = KNeighborsTimeSeriesClassifier(n_neighbors=1, distance="euclidean")
baseline.fit(X_train, y_train)
baseline_acc = baseline.score(X_test, y_test)

print(f"基线: {baseline_acc:.4f}")
print(f"您的模型: {accuracy:.4f}")
```

### 6. 统计显著性

测试改进是否具有统计显著性：

```python
from aeon.benchmarking import wilcoxon_test

# 在多个数据集上运行
accuracies_alg1 = [0.85, 0.92, 0.78, 0.88]
accuracies_alg2 = [0.83, 0.90, 0.76, 0.86]

stat, p_value = wilcoxon_test(accuracies_alg1, accuracies_alg2)
if p_value < 0.05:
    print("差异具有统计显著性")
```

## 数据集发现

查找符合条件的数据集：

```python
# 列出所有可用的分类数据集
from aeon.datasets import get_available_datasets

datasets = get_available_datasets("classification")
print(f"找到 {len(datasets)} 个分类数据集")

# 按属性过滤
univariate_datasets = [
    d for d in datasets
    if get_dataset_meta_data(d)['n_channels'] == 1
]
```
