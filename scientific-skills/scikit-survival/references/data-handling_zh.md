# 数据处理与预处理

## 理解生存数据

### Surv对象

scikit-survival中的生存数据使用结构化数组表示，包含两个字段：
- **event**：布尔值，指示事件是否发生（True）或数据被删失（False）
- **time**：事件发生时间或删失时间

```python
from sksurv.util import Surv

# 从独立数组创建生存结果
event = np.array([True, False, True, False, True])
time = np.array([5.2, 10.1, 3.7, 8.9, 6.3])

y = Surv.from_arrays(event=event, time=time)
print(y.dtype)  # [('event', '?'), ('time', '<f8')]
```

### 删失类型

**右删失**（最常见）：
- 研究对象在研究结束时未发生事件
- 研究对象失访
- 研究对象退出研究

**左删失**：
- 事件在观察开始前已发生
- 实践中罕见

**区间删失**：
- 事件发生在已知时间区间内
- 需要特殊处理方法

scikit-survival主要处理右删失数据。

## 加载数据

### 内置数据集

```python
from sksurv.datasets import (
    load_aids,
    load_breast_cancer,
    load_gbsg2,
    load_veterans_lung_cancer,
    load_whas500
)

# 加载数据集
X, y = load_breast_cancer()

# X是包含特征的pandas DataFrame
# y是包含'event'和'time'的结构化数组
print(f"特征维度: {X.shape}")
print(f"事件数量: {y['event'].sum()}")
print(f"删失率: {1 - y['event'].mean():.2%}")
```

### 加载自定义数据

#### 从Pandas DataFrame加载

```python
import pandas as pd
from sksurv.util import Surv

# 加载数据
df = pd.read_csv('survival_data.csv')

# 分离特征和结果
X = df.drop(['time', 'event'], axis=1)
y = Surv.from_dataframe('event', 'time', df)
```

#### 使用Surv.from_arrays从CSV加载

```python
import numpy as np
import pandas as pd
from sksurv.util import Surv

# 加载数据
df = pd.read_csv('survival_data.csv')

# 创建特征矩阵
X = df.drop(['time', 'event'], axis=1)

# 创建生存结果
y = Surv.from_arrays(
    event=df['event'].astype(bool),
    time=df['time'].astype(float)
)
```

### 加载ARFF文件

```python
from sksurv.io import loadarff

# 加载ARFF格式（Weka格式）
data = loadarff('survival_data.arff')

# 提取X和y
X = data[0]  # pandas DataFrame
y = data[1]  # 结构化数组
```

## 数据预处理

### 处理分类变量

#### 方法1：OneHotEncoder（scikit-survival）

```python
from sksurv.preprocessing import OneHotEncoder
import pandas as pd

# 识别分类列
categorical_cols = ['gender', 'race', 'treatment']

# 独热编码
encoder = OneHotEncoder()
X_encoded = encoder.fit_transform(X[categorical_cols])

# 与数值特征合并
numerical_cols = [col for col in X.columns if col not in categorical_cols]
X_processed = pd.concat([X[numerical_cols], X_encoded], axis=1)
```

#### 方法2：encode_categorical

```python
from sksurv.preprocessing import encode_categorical

# 自动编码所有分类列
X_encoded = encode_categorical(X)
```

#### 方法3：Pandas get_dummies

```python
import pandas as pd

# 对分类变量进行独热编码
X_encoded = pd.get_dummies(X, drop_first=True)
```

### 标准化

标准化对以下模型至关重要：
- 带正则化的Cox模型
- 支持向量机
- 对特征尺度敏感的模型

```python
from sklearn.preprocessing import StandardScaler

# 标准化特征
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 转换回DataFrame
X_scaled = pd.DataFrame(X_scaled, columns=X.columns, index=X.index)
```

### 处理缺失值

#### 检查缺失值

```python
# 检查缺失值
missing = X.isnull().sum()
print(missing[missing > 0])

# 可视化缺失数据
import seaborn as sns
sns.heatmap(X.isnull(), cbar=False)
```

#### 填补策略

```python
from sklearn.impute import SimpleImputer

# 数值特征均值填补
num_imputer = SimpleImputer(strategy='mean')
X_num = X.select_dtypes(include=[np.number])
X_num_imputed = num_imputer.fit_transform(X_num)

# 分类特征使用最频繁值填补
cat_imputer = SimpleImputer(strategy='most_frequent')
X_cat = X.select_dtypes(include=['object', 'category'])
X_cat_imputed = cat_imputer.fit_transform(X_cat)
```

#### 高级填补

```python
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer

# 迭代填补
imputer = IterativeImputer(random_state=42)
X_imputed = imputer.fit_transform(X)
```

### 特征选择

#### 方差阈值

```python
from sklearn.feature_selection import VarianceThreshold

# 移除低方差特征
selector = VarianceThreshold(threshold=0.01)
X_selected = selector.fit_transform(X)

# 获取选择的特征名称
selected_features = X.columns[selector.get_support()]
```

#### 单变量特征选择

```python
from sklearn.feature_selection import SelectKBest
from sksurv.util import Surv

# 选择top k特征
selector = SelectKBest(k=10)
X_selected = selector.fit_transform(X, y)

# 获取选择的特征
selected_features = X.columns[selector.get_support()]
```

## 完整预处理流程

### 使用sklearn Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.impute import SimpleImputer
from sksurv.linear_model import CoxPHSurvivalAnalysis

# 创建预处理和建模流程
pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='mean')),
    ('scaler', StandardScaler()),
    ('model', CoxPHSurvivalAnalysis())
])

# 拟合流程
pipeline.fit(X, y)

# 预测
predictions = pipeline.predict(X_test)
```

### 自定义预处理函数

```python
def preprocess_survival_data(X, y=None, scaler=None, encoder=None):
    """
    生存数据完整预处理流程

    参数:
    -----------
    X : DataFrame
        特征矩阵
    y : 结构化数组, 可选
        生存结果（用于过滤无效样本）
    scaler : StandardScaler, 可选
        已拟合的标准化器（用于测试数据）
    encoder : OneHotEncoder, 可选
        已拟合的编码器（用于测试数据）

    返回:
    --------
    X_processed : DataFrame
        处理后的特征
    scaler : StandardScaler
        已拟合的标准化器
    encoder : OneHotEncoder
        已拟合的编码器
    """
    from sklearn.preprocessing import StandardScaler
    from sksurv.preprocessing import encode_categorical

    # 1. 处理缺失值
    # 移除结果缺失的行
    if y is not None:
        mask = np.isfinite(y['time']) & (y['time'] > 0)
        X = X[mask]
        y = y[mask]

    # 填补缺失特征
    X = X.fillna(X.median())

    # 2. 编码分类变量
    if encoder is None:
        X_processed = encode_categorical(X)
        encoder = None  # encode_categorical不返回编码器
    else:
        X_processed = encode_categorical(X)

    # 3. 标准化数值特征
    if scaler is None:
        scaler = StandardScaler()
        X_processed = pd.DataFrame(
            scaler.fit_transform(X_processed),
            columns=X_processed.columns,
            index=X_processed.index
        )
    else:
        X_processed = pd.DataFrame(
            scaler.transform(X_processed),
            columns=X_processed.columns,
            index=X_processed.index
        )

    if y is not None:
        return X_processed, y, scaler, encoder
    else:
        return X_processed, scaler, encoder

# 使用示例
X_train_processed, y_train_processed, scaler, encoder = preprocess_survival_data(X_train, y_train)
X_test_processed, _, _ = preprocess_survival_data(X_test, scaler=scaler, encoder=encoder)
```

## 数据质量检查

### 验证生存数据

```python
def validate_survival_data(y):
    """检查生存数据质量"""

    # 检查负时间
    if np.any(y['time'] <= 0):
        print("警告：发现非正生存时间")
        print(f"负时间数量: {np.sum(y['time'] <= 0)}")

    # 检查缺失值
    if np.any(~np.isfinite(y['time'])):
        print("警告：发现缺失生存时间")
        print(f"缺失时间数量: {np.sum(~np.isfinite(y['time']))}")

    # 删失率
    censor_rate = 1 - y['event'].mean()
    print(f"删失率: {censor_rate:.2%}")

    if censor_rate > 0.7:
        print("警告：高删失率(>70%)")
        print("建议使用Uno's C-index替代Harrell's C-index")

    # 事件率
    print(f"事件数量: {y['event'].sum()}")
    print(f"删失数量: {(~y['event']).sum()}")

    # 时间统计
    print(f"中位时间: {np.median(y['time']):.2f}")
    print(f"时间范围: [{np.min(y['time']):.2f}, {np.max(y['time']):.2f}]")

# 使用验证
validate_survival_data(y)
```

### 检查事件充足性

```python
def check_events_per_feature(X, y, min_events_per_feature=10):
    """
    检查每个特征是否有足够事件。
    经验法则：Cox模型每个特征至少需要10个事件。
    """
    n_events = y['event'].sum()
    n_features = X.shape[1]
    events_per_feature = n_events / n_features

    print(f"事件数量: {n_events}")
    print(f"特征数量: {n_features}")
    print(f"每个特征的事件数: {events_per_feature:.1f}")

    if events_per_feature < min_events_per_feature:
        print(f"警告：每个特征事件数过低(<{min_events_per_feature})")
        print("建议:")
        print("  - 特征选择")
        print("  - 正则化(CoxnetSurvivalAnalysis)")
        print("  - 收集更多数据")

    return events_per_feature

# 使用检查
check_events_per_feature(X, y)
```

## 训练-测试集划分

### 随机划分

```python
from sklearn.model_selection import train_test_split

# 划分数据
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
```

### 分层划分

确保相似的删失率和时间分布：

```python
from sklearn.model_selection import train_test_split

# 创建分层标签
# 按事件状态和时间四分位数分层
time_quartiles = pd.qcut(y['time'], q=4, labels=False)
strat_labels = y['event'].astype(int) * 10 + time_quartiles

# 分层划分
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=strat_labels, random_state=42
)

# 验证相似分布
print("训练集:")
print(f"  删失率: {1 - y_train['event'].mean():.2%}")
print(f"  中位时间: {np.median(y_train['time']):.2f}")

print("测试集:")
print(f"  删失率: {1 - y_test['event'].mean():.2%}")
print(f"  中位时间: {np.median(y_test['time']):.2f}")
```

## 处理时变协变量

注意：scikit-survival不直接支持时变协变量。处理此类数据可考虑：
1. 时间分层分析
2. Landmarking方法
3. 使用其他包（如lifelines）

## 总结：完整数据准备流程

```python
from sksurv.util import Surv
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sksurv.preprocessing import encode_categorical
import pandas as pd
import numpy as np

# 1. 加载数据
df = pd.read_csv('data.csv')

# 2. 创建生存结果
y = Surv.from_dataframe('event', 'time', df)

# 3. 准备特征
X = df.drop(['event', 'time'], axis=1)

# 4. 验证数据
validate_survival_data(y)
check_events_per_feature(X, y)

# 5. 处理缺失值
X = X.fillna(X.median())

# 6. 划分数据
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 7. 编码分类变量
X_train = encode_categorical(X_train)
X_test = encode_categorical(X_test)

# 8. 标准化
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 转换回DataFrame
X_train_scaled = pd.DataFrame(X_train_scaled, columns=X_train.columns)
X_test_scaled = pd.DataFrame(X_test_scaled, columns=X_test.columns)

# 现在可以开始建模了！
```
