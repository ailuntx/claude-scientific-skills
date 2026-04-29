# 数据预处理与特征工程参考指南

## 概述

数据预处理将原始数据转换为适合机器学习模型的格式，包括缩放、编码、缺失值处理和特征工程。

## 特征缩放与归一化

### StandardScaler

**StandardScaler (`sklearn.preprocessing.StandardScaler`)**
- 将特征标准化为零均值和单位方差
- 公式：z = (x - 均值) / 标准差
- 适用场景：特征尺度不同，算法假设数据服从正态分布
- 必需算法：SVM、KNN、神经网络、PCA、带正则化的线性回归
- 示例：
```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)  # 使用与训练相同的参数

# 访问学习到的参数
print(f"均值: {scaler.mean_}")
print(f"标准差: {scaler.scale_}")
```

### MinMaxScaler

**MinMaxScaler (`sklearn.preprocessing.MinMaxScaler`)**
- 将特征缩放到指定范围（默认[0,1]）
- 公式：X_scaled = (X - X.min) / (X.max - X.min)
- 适用场景：需要限定数值范围，数据不服从正态分布
- 对异常值敏感
- 示例：
```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler(feature_range=(0, 1))
X_scaled = scaler.fit_transform(X_train)

# 自定义范围
scaler = MinMaxScaler(feature_range=(-1, 1))
X_scaled = scaler.fit_transform(X_train)
```

### RobustScaler

**RobustScaler (`sklearn.preprocessing.RobustScaler`)**
- 使用中位数和四分位距(IQR)进行缩放
- 公式：X_scaled = (X - 中位数) / IQR
- 适用场景：数据包含异常值
- 对异常值稳健
- 示例：
```python
from sklearn.preprocessing import RobustScaler

scaler = RobustScaler()
X_scaled = scaler.fit_transform(X_train)
```

### Normalizer

**Normalizer (`sklearn.preprocessing.Normalizer`)**
- 将每个样本独立归一化为单位范数
- 常用范数：'l1'、'l2'、'max'
- 适用场景：需要独立归一化每个样本（如文本特征）
- 示例：
```python
from sklearn.preprocessing import Normalizer

normalizer = Normalizer(norm='l2')  # 欧几里得范数
X_normalized = normalizer.fit_transform(X)
```

### MaxAbsScaler

**MaxAbsScaler (`sklearn.preprocessing.MaxAbsScaler`)**
- 按最大绝对值缩放
- 范围：[-1, 1]
- 不偏移/居中数据（保留稀疏性）
- 适用场景：数据已居中或稀疏
- 示例：
```python
from sklearn.preprocessing import MaxAbsScaler

scaler = MaxAbsScaler()
X_scaled = scaler.fit_transform(X_sparse)
```

## 类别型变量编码

### OneHotEncoder

**OneHotEncoder (`sklearn.preprocessing.OneHotEncoder`)**
- 为每个类别创建二进制列
- 适用场景：名义类别（无序）、树模型或线性模型
- 示例：
```python
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(sparse_output=False, handle_unknown='ignore')
X_encoded = encoder.fit_transform(X_categorical)

# 获取特征名称
feature_names = encoder.get_feature_names_out(['color', 'size'])

# 转换时处理未知类别
X_test_encoded = encoder.transform(X_test_categorical)
```

### OrdinalEncoder

**OrdinalEncoder (`sklearn.preprocessing.OrdinalEncoder`)**
- 将类别编码为整数
- 适用场景：有序类别、树模型
- 示例：
```python
from sklearn.preprocessing import OrdinalEncoder

# 自然顺序
encoder = OrdinalEncoder()
X_encoded = encoder.fit_transform(X_categorical)

# 自定义顺序
encoder = OrdinalEncoder(categories=[['small', 'medium', 'large']])
X_encoded = encoder.fit_transform(X_categorical)
```

### LabelEncoder

**LabelEncoder (`sklearn.preprocessing.LabelEncoder`)**
- 将目标标签(y)编码为整数
- 用途：目标变量编码
- 示例：
```python
from sklearn.preprocessing import LabelEncoder

le = LabelEncoder()
y_encoded = le.fit_transform(y)

# 解码还原
y_decoded = le.inverse_transform(y_encoded)
print(f"类别: {le.classes_}")
```

### 目标编码（使用category_encoders）

```python
# 安装: uv pip install category-encoders
from category_encoders import TargetEncoder

encoder = TargetEncoder()
X_train_encoded = encoder.fit_transform(X_train_categorical, y_train)
X_test_encoded = encoder.transform(X_test_categorical)
```

## 非线性变换

### 幂变换

**PowerTransformer**
- 使数据更接近高斯分布
- 方法：'yeo-johnson'（支持负值）、'box-cox'（仅正值）
- 适用场景：数据偏斜，算法假设正态分布
- 示例：
```python
from sklearn.preprocessing import PowerTransformer

# Yeo-Johnson（处理负值）
pt = PowerTransformer(method='yeo-johnson', standardize=True)
X_transformed = pt.fit_transform(X)

# Box-Cox（仅正值）
pt = PowerTransformer(method='box-cox', standardize=True)
X_transformed = pt.fit_transform(X)
```

### 分位数变换

**QuantileTransformer**
- 将特征变换为服从均匀或正态分布
- 对异常值稳健
- 适用场景：需要降低异常值影响
- 示例：
```python
from sklearn.preprocessing import QuantileTransformer

# 变换为均匀分布
qt = QuantileTransformer(output_distribution='uniform', random_state=42)
X_transformed = qt.fit_transform(X)

# 变换为正态分布
qt = QuantileTransformer(output_distribution='normal', random_state=42)
X_transformed = qt.fit_transform(X)
```

### 对数变换

```python
import numpy as np

# Log1p (log(1 + x)) - 处理零值
X_log = np.log1p(X)

# 或使用FunctionTransformer
from sklearn.preprocessing import FunctionTransformer

log_transformer = FunctionTransformer(np.log1p, inverse_func=np.expm1)
X_log = log_transformer.fit_transform(X)
```

## 缺失值填补

### SimpleImputer

**SimpleImputer (`sklearn.impute.SimpleImputer`)**
- 基础填补策略
- 策略：'mean'、'median'、'most_frequent'、'constant'
- 示例：
```python
from sklearn.impute import SimpleImputer

# 数值特征
imputer = SimpleImputer(strategy='mean')
X_imputed = imputer.fit_transform(X)

# 类别特征
imputer = SimpleImputer(strategy='most_frequent')
X_imputed = imputer.fit_transform(X_categorical)

# 常量填充
imputer = SimpleImputer(strategy='constant', fill_value=0)
X_imputed = imputer.fit_transform(X)
```

### 迭代填补

**IterativeImputer**
- 将含缺失值的特征建模为其他特征的函数
- 比SimpleImputer更复杂
- 示例：
```python
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer

imputer = IterativeImputer(max_iter=10, random_state=42)
X_imputed = imputer.fit_transform(X)
```

### K近邻填补

**KNNImputer**
- 使用K近邻进行填补
- 适用场景：特征间存在相关性
- 示例：
```python
from sklearn.impute import KNNImputer

imputer = KNNImputer(n_neighbors=5)
X_imputed = imputer.fit_transform(X)
```

## 特征工程

### 多项式特征

**PolynomialFeatures**
- 创建多项式和交互特征
- 适用场景：线性模型需要非线性特征
- 示例：
```python
from sklearn.preprocessing import PolynomialFeatures

# 二次项：包含x1, x2, x1^2, x2^2, x1*x2
poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(X)

# 获取特征名称
feature_names = poly.get_feature_names_out(['x1', 'x2'])

# 仅交互项（无幂次）
poly = PolynomialFeatures(degree=2, interaction_only=True, include_bias=False)
X_interactions = poly.fit_transform(X)
```

### 分箱/离散化

**KBinsDiscretizer**
- 将连续特征分箱为离散区间
- 策略：'uniform'、'quantile'、'kmeans'
- 编码：'onehot'、'ordinal'、'onehot-dense'
- 示例：
```python
from sklearn.preprocessing import KBinsDiscretizer

# 等宽分箱
binner = KBinsDiscretizer(n_bins=5, encode='ordinal', strategy='uniform')
X_binned = binner.fit_transform(X)

# 等频分箱（基于分位数）
binner = KBinsDiscretizer(n_bins=5, encode='onehot', strategy='quantile')
X_binned = binner.fit_transform(X)
```

### 二值化

**Binarizer**
- 基于阈值将特征转换为二进制值（0或1）
- 示例：
```python
from sklearn.preprocessing import Binarizer

binarizer = Binarizer(threshold=0.5)
X_binary = binarizer.fit_transform(X)
```

### 样条特征

**SplineTransformer**
- 创建样条基函数
- 适用于捕捉非线性关系
- 示例：
```python
from sklearn.preprocessing import SplineTransformer

spline = SplineTransformer(n_knots=5, degree=3)
X_splines = spline.fit_transform(X)
```

## 文本特征提取

### CountVectorizer

**CountVectorizer (`sklearn.feature_extraction.text.CountVectorizer`)**
- 将文本转换为词频矩阵
- 用途：词袋表示
- 示例：
```python
from sklearn.feature_extraction.text import CountVectorizer

vectorizer = CountVectorizer(
    max_features=5000,  # 保留前5000个特征
    min_df=2,  # 忽略出现少于2次的词
    max_df=0.8,  # 忽略出现在80%以上文档中的词
    ngram_range=(1, 2)  # 一元和二元词组
)

X_counts = vectorizer.fit_transform(documents)
feature_names = vectorizer.get_feature_names_out()
```

### TfidfVectorizer

**TfidfVectorizer**
- TF-IDF（词频-逆文档频率）变换
- 在多数任务中优于CountVectorizer
- 示例：
```python
from sklearn.feature_extraction.text import TfidfVectorizer

vectorizer = TfidfVectorizer(
    max_features=5000,
    min_df=2,
    max_df=0.8,
    ngram_range=(1, 2),
    stop_words='english'  # 移除英文停用词
)

X_tfidf = vectorizer.fit_transform(documents)
```

### HashingVectorizer

**HashingVectorizer**
- 使用哈希技巧提高内存效率
- 无需拟合，无法逆向转换
- 适用场景：超大词汇表、流式数据
- 示例：
```python
from sklearn.feature_extraction.text import HashingVectorizer

vectorizer = HashingVectorizer(n_features=2**18)
X_hashed = vectorizer.transform(documents)  # 无需拟合
```

## 特征选择

### 过滤式方法

**方差阈值**
- 移除低方差特征
- 示例：
```python
from sklearn.feature_selection import VarianceThreshold

selector = VarianceThreshold(threshold=0.01)
X_selected = selector.fit_transform(X)
```

**SelectKBest / SelectPercentile**
- 基于统计检验选择特征
- 检验方法：f_classif、chi2、mutual_info_classif
- 示例：
```python
from sklearn.feature_selection import SelectKBest, f_classif

# 选择前10个特征
selector = SelectKBest(score_func=f_classif, k=10)
X_selected = selector.fit_transform(X_train, y_train)

# 获取选中特征索引
selected_indices = selector.get_support(indices=True)
```

### 包裹式方法

**递归特征消除(RFE)**
- 递归移除特征
- 使用模型特征重要性
- 示例：
```python
from sklearn.feature_selection import RFE
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=100, random_state=42)
rfe = RFE(estimator=model, n_features_to_select=10, step=1)
X_selected = rfe.fit_transform(X_train, y_train)

# 获取选中特征
selected_features = rfe.support_
feature_ranking = rfe.ranking_
```

**RFECV（带交叉验证）**
- 通过交叉验证确定最优特征数量
- 示例：
```python
from sklearn.feature_selection import RFECV

model = RandomForestClassifier(n_estimators=100, random_state=42)
rfecv = RFECV(estimator=model, cv=5, scoring='accuracy')
X_selected = rfecv.fit_transform(X_train, y_train)

print(f"最优特征数量: {rfecv.n_features_}")
```

### 嵌入式方法

**SelectFromModel**
- 基于模型系数/重要性选择特征
- 适用模型：线性模型（L1）、树模型
- 示例：
```python
from sklearn.feature_selection import SelectFromModel
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=100, random_state=42)
selector = SelectFromModel(model, threshold='median')
selector.fit(X_train, y_train)
X_selected = selector.transform(X_train)

# 获取选中特征
selected_features = selector.get_support()
```

**基于L1的特征选择**
```python
from sklearn.linear_model import LogisticRegression
from sklearn.feature_selection import SelectFromModel

model = LogisticRegression(penalty='l1', solver='liblinear', C=0.1)
selector = SelectFromModel(model)
selector.fit(X_train, y_train)
X_selected = selector.transform(X_train)
```

## 异常值处理

### IQR方法

```python
import numpy as np

Q1 = np.percentile(X, 25, axis=0)
Q3 = np.percentile(X, 75, axis=0)
IQR = Q3 - Q1

# 定义异常值边界
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# 移除异常值
mask = np.all((X >= lower_bound) & (X <= upper_bound), axis=1)
X_no_outliers = X[mask]
```

### 缩尾处理

```python
from scipy.stats import mstats

# 在5%和95%分位数处截断异常值
X_winsorized = mstats.winsorize(X, limits=[0.05, 0.05], axis=0)
```

## 自定义转换器

### 使用FunctionTransformer

```python
from sklearn.preprocessing import FunctionTransformer
import numpy as np

def log_transform(X):
    return np.log1p(X)

transformer = FunctionTransformer(log_transform, inverse_func=np.expm1)
X_transformed = transformer.fit_transform(X)
```

### 创建自定义转换器

```python
from sklearn.base import BaseEstimator, TransformerMixin

class CustomTransformer(BaseEstimator, TransformerMixin):
    def __init__(self, parameter=1):
        self.parameter = parameter

    def fit(self, X, y=None):
        # 从X学习参数（如需要）
        return self

    def transform(self, X):
        # 转换X
        return X * self.parameter

transformer = CustomTransformer(parameter=2)
X_transformed = transformer.fit_transform(X)
```

## 最佳实践

### 仅在训练数据上拟合
始终仅在训练数据上拟合转换器：
```python
# 正确做法
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 错误做法 - 导致数据泄露
scaler = StandardScaler()
X_all_scaled = scaler.fit_transform(np.vstack([X_train, X_test]))
```

### 使用管道
将预处理与模型结合：
```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', LogisticRegression())
])

pipeline.fit(X_train, y_train)
```

### 分别处理类别和数值特征
使用ColumnTransformer：
```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

numeric_features = ['age', 'income']
categorical_features = ['gender', 'occupation']

preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), numeric_features),
        ('cat', OneHotEncoder(), categorical_features)
    ]
)

X_transformed = preprocessor.fit_transform(X)
```

###

- K-Means 聚类

**无需缩放处理：**
- 基于树的模型（决策树、随机森林、梯度提升）
- 朴素贝叶斯

**编码要求：**
- 线性模型、支持向量机、K近邻算法：名义特征需采用独热编码
- 基于树的模型：可直接处理有序编码
