# 变换

Aeon 提供了广泛的变换能力，用于时间序列数据的预处理、特征提取和表示学习。

## 变换类型

Aeon 区分两类变换：
- **集合变换器（CollectionTransformers）**：处理多个时间序列（集合）
- **序列变换器（SeriesTransformers）**：处理单个时间序列

## 集合变换器

### 基于卷积的特征提取

使用随机核函数实现快速、可扩展的特征生成：
- `RocketTransformer` - 随机卷积核
- `MiniRocketTransformer` - 简化版 ROCKET 加速计算
- `MultiRocketTransformer` - 增强型 ROCKET 变体
- `HydraTransformer` - 多分辨率膨胀卷积
- `MultiRocketHydraTransformer` - ROCKET 与 Hydra 组合
- `ROCKETGPU` - GPU 加速版本

**适用场景**：需要为任何机器学习算法提供快速可扩展特征，具备强基线性能时。

### 统计特征提取

基于时间序列特性的领域无关特征：
- `Catch22` - 22 个经典时间序列特征
- `TSFresh` - 全面自动化特征提取（100+ 特征）
- `TSFreshRelevant` - 带相关性筛选的特征提取
- `SevenNumberSummary` - 描述性统计（均值、标准差、分位数）

**适用场景**：需要可解释特征、领域无关方法或传统机器学习输入时。

### 基于字典的表示

用于离散表示的符号化近似：
- `SAX` - 符号聚合近似
- `PAA` - 分段聚合近似
- `SFA` - 符号傅里叶近似
- `SFAFast` - 优化版 SFA
- `SFAWhole` - 全序列 SFA（无窗口分割）
- `BORF` - 感受野词袋模型

**适用场景**：需要离散/符号表示、降维或可解释性时。

### 基于 Shapelet 的特征

判别性子序列提取：
- `RandomShapeletTransform` - 随机判别 shapelet
- `RandomDilatedShapeletTransform` - 多尺度膨胀 shapelet
- `SAST` - 可扩展精确子序列变换
- `RSAST` - 随机化 SAST

**适用场景**：需要可解释判别模式或相位不变特征时。

### 基于区间的特征

时间区间的统计摘要：
- `RandomIntervals` - 随机区间特征
- `SupervisedIntervals` - 监督式区间选择
- `QUANTTransformer` - 基于分位数的区间特征

**适用场景**：预测模式集中在特定时间窗口时。

### 预处理变换

数据准备与标准化：
- `MinMaxScaler` - 缩放到 [0, 1] 范围
- `Normalizer` - Z 标准化（零均值，单位方差）
- `Centerer` - 零均值中心化
- `SimpleImputer` - 填充缺失值
- `DownsampleTransformer` - 降低时间分辨率
- `Tabularizer` - 时间序列转表格格式

**适用场景**：需要标准化、缺失值处理或格式转换时。

### 专用变换

高级分析方法：
- `MatrixProfile` - 模式发现的距离剖面计算
- `DWTTransformer` - 离散小波变换
- `AutocorrelationFunctionTransformer` - 自相关函数计算
- `Dobin` - 基于邻居的距离离群基准
- `SignatureTransformer` - 路径签名方法
- `PLATransformer` - 分段线性近似

### 类别不平衡处理
- `ADASYN` - 自适应合成采样
- `SMOTE` - 合成少数类过采样
- `OHIT` - 高度不平衡时间序列过采样

**适用场景**：处理类别不平衡的分类任务时。

### 流水线组合
- `CollectionTransformerPipeline` - 多变换器链式组合

## 序列变换器

处理单个时间序列（如预测任务中的预处理）。

### 统计分析
- `AutoCorrelationSeriesTransformer` - 自相关
- `StatsModelsACF` - statsmodels 自相关函数
- `StatsModelsPACF` - 偏自相关函数

### 平滑与滤波
- `ExponentialSmoothing` - 指数加权移动平均
- `MovingAverage` - 简单/加权移动平均
- `SavitzkyGolayFilter` - 多项式平滑
- `GaussianFilter` - 高斯核平滑
- `BKFilter` - Baxter-King 带通滤波器
- `DiscreteFourierApproximation` - 傅里叶基滤波

**适用场景**：需要降噪、趋势提取或频率滤波时。

### 降维
- `PCASeriesTransformer` - 主成分分析
- `PlASeriesTransformer` - 分段线性近似

### 变换操作
- `BoxCoxTransformer` - 方差稳定化
- `LogTransformer` - 对数缩放
- `ClaSPTransformer` - 分类分数剖面

### 流水线组合
- `SeriesTransformerPipeline` - 序列变换器链式组合

## 快速开始：特征提取

```python
from aeon.transformations.collection.convolution_based import RocketTransformer
from aeon.classification.sklearn import RotationForest
from aeon.datasets import load_classification

# 加载数据
X_train, y_train = load_classification("GunPoint", split="train")
X_test, y_test = load_classification("GunPoint", split="test")

# 提取 ROCKET 特征
rocket = RocketTransformer()
X_train_features = rocket.fit_transform(X_train)
X_test_features = rocket.transform(X_test)

# 搭配任意 sklearn 分类器
clf = RotationForest()
clf.fit(X_train_features, y_train)
accuracy = clf.score(X_test_features, y_test)
```

## 快速开始：预处理流水线

```python
from aeon.transformations.collection import (
    MinMaxScaler,
    SimpleImputer,
    CollectionTransformerPipeline
)

# 构建预处理流水线
pipeline = CollectionTransformerPipeline([
    ('imputer', SimpleImputer(strategy='mean')),
    ('scaler', MinMaxScaler())
])

X_transformed = pipeline.fit_transform(X_train)
```

## 快速开始：序列平滑

```python
from aeon.transformations.series import MovingAverage

# 平滑单个时间序列
smoother = MovingAverage(window_size=5)
y_smoothed = smoother.fit_transform(y)
```

## 算法选择

### 特征提取场景：
- **速度+性能**：MiniRocketTransformer
- **可解释性**：Catch22, TSFresh
- **降维需求**：PAA, SAX, PCA
- **判别模式提取**：Shapelet 变换
- **全面特征**：TSFresh（需更长运行时间）

### 预处理场景：
- **标准化**：Normalizer, MinMaxScaler
- **平滑处理**：MovingAverage, SavitzkyGolayFilter
- **缺失值处理**：SimpleImputer
- **频率分析**：DWTTransformer, 傅里叶方法

### 符号表示场景：
- **快速近似**：PAA
- **基于字母表**：SAX
- **基于频率**：SFA, SFAFast

## 最佳实践

1. **仅在训练数据上拟合**：避免数据泄露
   ```python
   transformer.fit(X_train)
   X_train_tf = transformer.transform(X_train)
   X_test_tf = transformer.transform(X_test)
   ```

2. **流水线组合**：复杂工作流链式组合变换器
   ```python
   pipeline = CollectionTransformerPipeline([
       ('imputer', SimpleImputer()),
       ('scaler', Normalizer()),
       ('features', RocketTransformer())
   ])
   ```

3. **特征选择**：TSFresh 可能生成过多特征，建议筛选
   ```python
   from sklearn.feature_selection import SelectKBest
   selector = SelectKBest(k=100)
   X_selected = selector.fit_transform(X_features, y)
   ```

4. **内存考量**：部分变换器在大数据集上内存消耗高
   - 追求速度时用 MiniRocket 替代 ROCKET
   - 超长序列考虑降采样
   - 使用 ROCKETGPU 实现 GPU 加速

5. **领域知识引导**：选择匹配领域的变换方法：
   - 周期性数据：傅里叶基方法
   - 噪声数据：平滑滤波器
   - 峰值检测：小波变换
