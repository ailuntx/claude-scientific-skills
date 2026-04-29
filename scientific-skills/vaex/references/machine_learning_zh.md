# 机器学习集成

本文档涵盖 Vaex 的机器学习功能，包括转换器、编码器、特征工程、模型集成以及在大型数据集上构建机器学习流水线。

## 概述

Vaex 提供完整的机器学习框架 (`vaex.ml`)，可无缝处理大型数据集。该框架包含：
- 特征缩放和工程转换器
- 分类变量编码器
- 降维技术（PCA）
- 聚类算法
- 与 scikit-learn、XGBoost、LightGBM、CatBoost 和 Keras 的集成
- 生产部署的状态管理

**关键优势：** 所有转换均创建虚拟列，因此预处理不会增加内存占用。

## 特征缩放

### 标准缩放器

```python
import vaex
import vaex.ml

df = vaex.open('data.hdf5')

# 拟合标准缩放器
scaler = vaex.ml.StandardScaler(features=['age', 'income', 'score'])
scaler.fit(df)

# 转换（创建虚拟列）
df = scaler.transform(df)

# 生成缩放列：'standard_scaled_age', 'standard_scaled_income' 等
print(df.column_names)
```

### 最小最大缩放器

```python
# 缩放到 [0, 1] 范围
minmax_scaler = vaex.ml.MinMaxScaler(features=['age', 'income'])
minmax_scaler.fit(df)
df = minmax_scaler.transform(df)

# 自定义范围
minmax_scaler = vaex.ml.MinMaxScaler(
    features=['age'],
    feature_range=(-1, 1)
)
```

### 最大绝对值缩放器

```python
# 按最大绝对值缩放
maxabs_scaler = vaex.ml.MaxAbsScaler(features=['values'])
maxabs_scaler.fit(df)
df = maxabs_scaler.transform(df)
```

### 鲁棒缩放器

```python
# 使用中位数和IQR缩放（抗异常值）
robust_scaler = vaex.ml.RobustScaler(features=['income', 'age'])
robust_scaler.fit(df)
df = robust_scaler.transform(df)
```

## 分类编码

### 标签编码器

```python
# 将分类变量编码为整数
label_encoder = vaex.ml.LabelEncoder(features=['category', 'region'])
label_encoder.fit(df)
df = label_encoder.transform(df)

# 生成：'label_encoded_category', 'label_encoded_region'
```

### 独热编码器

```python
# 为每个类别创建二进制列
onehot = vaex.ml.OneHotEncoder(features=['category'])
onehot.fit(df)
df = onehot.transform(df)

# 生成列如：'category_A', 'category_B', 'category_C'

# 控制前缀
onehot = vaex.ml.OneHotEncoder(
    features=['category'],
    prefix='cat_'
)
```

### 频率编码器

```python
# 按类别频率编码
freq_encoder = vaex.ml.FrequencyEncoder(features=['category'])
freq_encoder.fit(df)
df = freq_encoder.transform(df)

# 每个类别被替换为其在数据集中的频率
```

### 目标编码器（均值编码器）

```python
# 按目标均值编码（用于监督学习）
target_encoder = vaex.ml.TargetEncoder(
    features=['category'],
    target='target_variable'
)
target_encoder.fit(df)
df = target_encoder.transform(df)

# 使用全局均值处理未知类别
```

### 证据权重编码器

```python
# 用于二分类的编码
woe_encoder = vaex.ml.WeightOfEvidenceEncoder(
    features=['category'],
    target='binary_target'
)
woe_encoder.fit(df)
df = woe_encoder.transform(df)
```

## 特征工程

### 分箱/离散化

```python
# 将连续变量分箱为离散区间
binner = vaex.ml.Discretizer(
    features=['age'],
    n_bins=5,
    strategy='uniform'  # 或 'quantile'
)
binner.fit(df)
df = binner.transform(df)
```

### 周期性转换

```python
# 转换周期性特征（小时、星期、月份）
cyclic = vaex.ml.CycleTransformer(
    features=['hour', 'day_of_week'],
    n=[24, 7]  # 各特征的周期
)
cyclic.fit(df)
df = cyclic.transform(df)

# 为每个特征生成正弦和余弦分量
```

### PCA（主成分分析）

```python
# 降维处理
pca = vaex.ml.PCA(
    features=['feature1', 'feature2', 'feature3', 'feature4'],
    n_components=2
)
pca.fit(df)
df = pca.transform(df)

# 生成：'PCA_0', 'PCA_1'

# 查看解释方差
print(pca.explained_variance_ratio_)
```

### 随机投影

```python
# 快速降维
projector = vaex.ml.RandomProjection(
    features=['x1', 'x2', 'x3', 'x4', 'x5'],
    n_components=3
)
projector.fit(df)
df = projector.transform(df)
```

## 聚类

### K均值

```python
# 数据聚类
kmeans = vaex.ml.KMeans(
    features=['feature1', 'feature2', 'feature3'],
    n_clusters=5,
    max_iter=100
)
kmeans.fit(df)
df = kmeans.transform(df)

# 生成包含聚类标签的 'prediction' 列

# 查看聚类中心
print(kmeans.cluster_centers_)
```

## 外部库集成

### Scikit-Learn

```python
from sklearn.ensemble import RandomForestClassifier
import vaex.ml

# 准备数据
train_df = df[df.split == 'train']
test_df = df[df.split == 'test']

# 特征和目标
features = ['feature1', 'feature2', 'feature3']
target = 'target'

# 训练scikit-learn模型
model = RandomForestClassifier(n_estimators=100)

# 使用Vaex数据拟合
sklearn_model = vaex.ml.sklearn.Predictor(
    features=features,
    target=target,
    model=model,
    prediction_name='rf_prediction'
)
sklearn_model.fit(train_df)

# 预测（创建虚拟列）
test_df = sklearn_model.transform(test_df)

# 获取预测结果
predictions = test_df.rf_prediction.values
```

### XGBoost

```python
import xgboost as xgb
import vaex.ml

# 创建XGBoost模型
booster = vaex.ml.xgboost.XGBoostModel(
    features=features,
    target=target,
    prediction_name='xgb_pred'
)

# 配置参数
params = {
    'max_depth': 6,
    'eta': 0.1,
    'objective': 'reg:squarederror',
    'eval_metric': 'rmse'
}

# 训练
booster.fit(
    df=train_df,
    params=params,
    num_boost_round=100
)

# 预测
test_df = booster.transform(test_df)
```

### LightGBM

```python
import lightgbm as lgb
import vaex.ml

# 创建LightGBM模型
lgb_model = vaex.ml.lightgbm.LightGBMModel(
    features=features,
    target=target,
    prediction_name='lgb_pred'
)

# 参数配置
params = {
    'objective': 'binary',
    'metric': 'auc',
    'num_leaves': 31,
    'learning_rate': 0.05
}

# 训练
lgb_model.fit(
    df=train_df,
    params=params,
    num_boost_round=100
)

# 预测
test_df = lgb_model.transform(test_df)
```

### CatBoost

```python
from catboost import CatBoostClassifier
import vaex.ml

# 创建CatBoost模型
catboost_model = vaex.ml.catboost.CatBoostModel(
    features=features,
    target=target,
    prediction_name='catboost_pred'
)

# 参数配置
params = {
    'iterations': 100,
    'depth': 6,
    'learning_rate': 0.1,
    'loss_function': 'Logloss'
}

# 训练
catboost_model.fit(train_df, **params)

# 预测
test_df = catboost_model.transform(test_df)
```

### Keras/TensorFlow

```python
from tensorflow import keras
import vaex.ml

# 定义Keras模型
def create_model(input_dim):
    model = keras.Sequential([
        keras.layers.Dense(64, activation='relu', input_shape=(input_dim,)),
        keras.layers.Dense(32, activation='relu'),
        keras.layers.Dense(1, activation='sigmoid')
    ])
    model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
    return model

# 封装到Vaex
keras_model = vaex.ml.keras.KerasModel(
    features=features,
    target=target,
    model=create_model(len(features)),
    prediction_name='keras_pred'
)

# 训练
keras_model.fit(
    train_df,
    epochs=10,
    batch_size=10000
)

# 预测
test_df = keras_model.transform(test_df)
```

## 构建机器学习流水线

### 顺序流水线

```python
import vaex.ml

# 创建预处理流水线
pipeline = []

# 步骤1：分类编码
label_enc = vaex.ml.LabelEncoder(features=['category'])
pipeline.append(label_enc)

# 步骤2：特征缩放
scaler = vaex.ml.StandardScaler(features=['age', 'income'])
pipeline.append(scaler)

# 步骤3：PCA
pca = vaex.ml.PCA(features=['age', 'income'], n_components=2)
pipeline.append(pca)

# 拟合流水线
for step in pipeline:
    step.fit(df)
    df = step.transform(df)

# 或使用fit_transform
for step in pipeline:
    df = step.fit_transform(df)
```

### 完整机器学习流水线

```python
import vaex
import vaex.ml
from sklearn.ensemble import RandomForestClassifier

# 加载数据
df = vaex.open('data.hdf5')

# 数据拆分
train_df = df[df.year < 2020]
test_df = df[df.year >= 2020]

# 定义流水线
# 1. 分类编码
cat_encoder = vaex.ml.LabelEncoder(features=['category', 'region'])

# 2. 特征缩放
scaler = vaex.ml.StandardScaler(features=['age', 'income', 'score'])

# 3. 模型
features = ['label_encoded_category', 'label_encoded_region',
            'standard_scaled_age', 'standard_scaled_income', 'standard_scaled_score']
model = vaex.ml.sklearn.Predictor(
    features=features,
    target='target',
    model=RandomForestClassifier(n_estimators=100),
    prediction_name='prediction'
)

# 拟合流水线
train_df = cat_encoder.fit_transform(train_df)
train_df = scaler.fit_transform(train_df)
model.fit(train_df)

# 应用于测试集
test_df = cat_encoder.transform(test_df)
test_df = scaler.transform(test_df)
test_df = model.transform(test_df)

# 评估
accuracy = (test_df.prediction == test_df.target).mean()
print(f"准确率: {accuracy:.4f}")
```

## 状态管理与部署

### 保存流水线状态

```python
# 完成所有转换器和模型拟合后
# 保存整个流水线状态
train_df.state_write('pipeline_state.json')

# 生产环境：加载新数据并应用转换
prod_df = vaex.open('new_data.hdf5')
prod_df.state_load('pipeline_state.json')

# 所有转换和模型已应用
predictions = prod_df.prediction.values
```

### 在数据帧间转移状态

```python
# 在训练数据上拟合
train_df = cat_encoder.fit_transform(train_df)
train_df = scaler.fit_transform(train_df)
model.fit(train_df)

# 保存状态
train_df.state_write('model_state.json')

# 应用于测试数据
test_df.state_load('model_state.json')

# 应用于验证数据
val_df.state_load('model_state.json')
```

### 带转换导出

```python
# 导出包含所有物化虚拟列的数据帧
df_with_features = df.copy()
df_with_features = df_with_features.materialize()
df_with_features.export_hdf5('processed_data.hdf5')
```

## 模型评估

### 分类指标

```python
# 二分类评估
from sklearn.metrics import accuracy_score, roc_auc_score, f1_score

y_true = test_df.target.values
y_pred = test_df.prediction.values
y_proba = test_df.prediction_proba.values if hasattr(test_df, 'prediction_proba') else None

accuracy = accuracy_score(y_true, y_pred)
f1 = f1_score(y_true, y_pred)
if y_proba is not None:
    auc = roc_auc_score(y_true, y_proba)

print(f"准确率: {accuracy:.4f}")
print(f"F1分数: {f1:.4f}")
if y_proba is not None:
    print(f"AUC-ROC: {auc:.4f}")
```

### 回归指标

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score

y_true = test_df.target.values
y_pred = test_df.prediction.values

mse = mean_squared_error(y_true, y_pred)
rmse = np.sqrt(mse)
mae = mean_absolute_error(y_true, y_pred)
r2 = r2_score(y_true, y_pred)

print(f"RMSE: {rmse:.4f}")
print(f"MAE: {mae:.4f}")
print(f"R²: {r2:.4f}")
```

### 交叉验证

```python
# 手动K折交叉验证
import numpy as np

# 创建折数索引
df['fold'] = np.random.randint(0, 5, len(df))

results = []
for fold in range(5):
    train = df[df.fold != fold]
    val = df[df.fold == fold]

    # 拟合流水线
    train = encoder.fit_transform(train)
    train = scaler.fit_transform(train)
    model.fit(train)

    # 验证
    val = encoder.transform(val)
    val = scaler.transform(val)
    val = model.transform(val)

    accuracy = (val.prediction == val.target).mean()
    results.append(accuracy)

print(f"交叉验证准确率: {np.mean(results):.4f} ± {np.std(results):.4f}")
```

## 特征选择

### 基于相关性

```python
# 计算与目标的相关性
correlations = {}
for feature in features:
    corr = df.correlation(df[feature], df.target)
    correlations[feature] = abs(corr)

# 按相关性排序
sorted_features = sorted(correlations.items(), key=lambda x: x[1], reverse=True)
top_features = [f[0] for f in sorted_features[:10]]

print("Top 10特征:", top_features)
```

### 基于方差

```python
# 移除低方差特征
feature_variances = {}
for feature in features:
    var = df[feature].std() ** 2
    feature_variances[feature] = var

# 保留方差高于阈值的特征
threshold = 0.01
selected_features = [f for f, v in feature_variances.items() if v > threshold]
```

## 处理不平衡数据

### 类别权重

```python
# 计算类别权重
class_counts = df.groupby('target', agg='count')
total = len(df)
weights = {
    0: total / (2 * class_counts[0]),
    1: total / (2 * class_counts[1])
}

# 在模型中使用
model = RandomForestClassifier(class_weight=weights)
```

### 欠采样

```python
# 对多数类欠采样
minority_count = df[df.target == 1].count()

# 从多数类抽样
majority_sampled = df[df.target == 0].sample(n=minority_count)
minority_all = df[df.target == 1]

# 合并
df_balanced = vaex.concat([majority_sampled, minority_all])
```

### 过采样（SMOTE替代方案）

```python
# 复制少数类样本
minority = df[df.target == 1

```markdown
train = cat_enc.fit_transform(train)

# 特征缩放
scaler = vaex.ml.StandardScaler(features=['num1', 'num2', 'num3'])
train = scaler.fit_transform(train)

# 模型训练
features = ['label_encoded_cat1', 'label_encoded_cat2',
            'standard_scaled_num1', 'standard_scaled_num2', 'standard_scaled_num3']
model = vaex.ml.sklearn.Predictor(
    features=features,
    target='target',
    model=RandomForestClassifier(n_estimators=100)
)
model.fit(train)

# 保存状态
train.state_write('production_pipeline.json')

# 应用到测试集
test.state_load('production_pipeline.json')

# 评估
accuracy = (test.prediction == test.target).mean()
print(f"测试集准确率: {accuracy:.4f}")
```

### 模式：特征工程流水线

```python
# 创建丰富特征
df['age_squared'] = df.age ** 2
df['income_log'] = df.income.log()
df['age_income_interaction'] = df.age * df.income

# 分箱处理
df['age_bin'] = df.age.digitize([0, 18, 30, 50, 65, 100])

# 周期特征
df['hour_sin'] = (2 * np.pi * df.hour / 24).sin()
df['hour_cos'] = (2 * np.pi * df.hour / 24).cos()

# 聚合特征
avg_by_category = df.groupby('category').agg({'income': 'mean'})
# 回连创建特征
df = df.join(avg_by_category, on='category', rsuffix='_category_mean')
```

## 最佳实践

1. **使用虚拟列** - 转换器创建虚拟列（无内存开销）
2. **保存状态文件** - 便于部署和复现
3. **批量操作** - 计算多个特征时使用 `delay=True`
4. **特征缩放** - 在PCA或基于距离的算法前必须缩放特征
5. **类别编码** - 使用合适的编码器（标签编码、独热编码、目标编码）
6. **交叉验证** - 始终在保留数据集上验证
7. **监控内存** - 使用 `df.byte_size()` 检查内存占用
8. **导出检查点** - 在长流水线中保存中间结果

## 相关资源

- 数据预处理：参见 `data_processing.md`
- 性能优化：参见 `performance.md`
- DataFrame操作：参见 `core_dataframes.md`
```
