# 模型选择与评估参考指南

## 概述

使用 scikit-learn 模型选择工具评估模型、调优超参数和选择最佳模型的综合指南。

## 训练-测试集划分

### 基础划分

```python
from sklearn.model_selection import train_test_split

# 基础划分（默认75/25）
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, random_state=42)

# 分层抽样（保持类别分布）
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, stratify=y, random_state=42
)

# 三向划分（训练/验证/测试）
X_train, X_temp, y_train, y_temp = train_test_split(X, y, test_size=0.3, random_state=42)
X_val, X_test, y_val, y_test = train_test_split(X_temp, y_temp, test_size=0.5, random_state=42)
```

## 交叉验证

### 交叉验证策略

**KFold**
- 标准k折交叉验证
- 将数据分割为k个连续折叠
```python
from sklearn.model_selection import KFold

kf = KFold(n_splits=5, shuffle=True, random_state=42)
for train_idx, val_idx in kf.split(X):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
```

**StratifiedKFold**
- 保持每折中的类别分布
- 适用于不平衡分类
```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
for train_idx, val_idx in skf.split(X, y):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
```

**TimeSeriesSplit**
- 适用于时间序列数据
- 保持时间顺序
```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
for train_idx, val_idx in tscv.split(X):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
```

**GroupKFold**
- 确保相同组的样本不会同时出现在训练集和验证集
- 适用于样本不独立的情况
```python
from sklearn.model_selection import GroupKFold

gkf = GroupKFold(n_splits=5)
for train_idx, val_idx in gkf.split(X, y, groups=group_ids):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
```

**LeaveOneOut (LOO)**
- 每个样本作为验证集使用一次
- 适用于极小数据集
- 计算开销大
```python
from sklearn.model_selection import LeaveOneOut

loo = LeaveOneOut()
for train_idx, val_idx in loo.split(X):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
```

### 交叉验证函数

**cross_val_score**
- 使用交叉验证评估模型
- 返回分数数组
```python
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=100, random_state=42)
scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')

print(f"分数: {scores}")
print(f"均值: {scores.mean():.3f} (±{scores.std() * 2:.3f})")
```

**cross_validate**
- 比cross_val_score更全面
- 可返回多个指标和拟合时间
```python
from sklearn.model_selection import cross_validate

model = RandomForestClassifier(n_estimators=100, random_state=42)
cv_results = cross_validate(
    model, X, y, cv=5,
    scoring=['accuracy', 'precision', 'recall', 'f1'],
    return_train_score=True,
    return_estimator=True  # 返回拟合的评估器
)

print(f"测试准确率: {cv_results['test_accuracy'].mean():.3f}")
print(f"测试精确率: {cv_results['test_precision'].mean():.3f}")
print(f"拟合时间: {cv_results['fit_time'].mean():.3f}秒")
```

**cross_val_predict**
- 获取每个样本在验证集时的预测
- 适用于错误分析
```python
from sklearn.model_selection import cross_val_predict

model = RandomForestClassifier(n_estimators=100, random_state=42)
y_pred = cross_val_predict(model, X, y, cv=5)

# 可分析预测值与实际值
from sklearn.metrics import confusion_matrix
cm = confusion_matrix(y, y_pred)
```

## 超参数调优

### 网格搜索

**GridSearchCV**
- 在参数网格上进行穷举搜索
- 测试所有组合
```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier

param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [5, 10, 15, None],
    'min_samples_split': [2, 5, 10],
    'min_samples_leaf': [1, 2, 4]
}

model = RandomForestClassifier(random_state=42)
grid_search = GridSearchCV(
    model, param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,  # 使用所有CPU核心
    verbose=1
)

grid_search.fit(X_train, y_train)

print(f"最佳参数: {grid_search.best_params_}")
print(f"最佳交叉验证分数: {grid_search.best_score_:.3f}")
print(f"测试分数: {grid_search.score(X_test, y_test):.3f}")

# 访问最佳模型
best_model = grid_search.best_estimator_

# 查看所有结果
import pandas as pd
results_df = pd.DataFrame(grid_search.cv_results_)
```

### 随机搜索

**RandomizedSearchCV**
- 从参数分布中随机采样组合
- 对大搜索空间更高效
```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform

param_distributions = {
    'n_estimators': randint(50, 300),
    'max_depth': [5, 10, 15, 20, None],
    'min_samples_split': randint(2, 20),
    'min_samples_leaf': randint(1, 10),
    'max_features': uniform(0.1, 0.9)  # 连续分布
}

model = RandomForestClassifier(random_state=42)
random_search = RandomizedSearchCV(
    model, param_distributions,
    n_iter=100,  # 采样的参数设置数量
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    verbose=1,
    random_state=42
)

random_search.fit(X_train, y_train)

print(f"最佳参数: {random_search.best_params_}")
print(f"最佳分数: {random_search.best_score_:.3f}")
```

### 逐次减半法

**HalvingGridSearchCV / HalvingRandomSearchCV**
- 使用逐次减半迭代选择最佳候选
- 比穷举搜索更高效
```python
from sklearn.experimental import enable_halving_search_cv
from sklearn.model_selection import HalvingGridSearchCV

param_grid = {
    'n_estimators': [50, 100, 200, 300],
    'max_depth': [5, 10, 15, 20, None],
    'min_samples_split': [2, 5, 10, 20]
}

model = RandomForestClassifier(random_state=42)
halving_search = HalvingGridSearchCV(
    model, param_grid,
    cv=5,
    factor=3,  # 每轮淘汰候选的比例
    resource='n_samples',  # 也可用'n_estimators'用于集成模型
    max_resources='auto',
    random_state=42
)

halving_search.fit(X_train, y_train)
print(f"最佳参数: {halving_search.best_params_}")
```

## 分类指标

### 基础指标

```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    balanced_accuracy_score, matthews_corrcoef
)

y_pred = model.predict(X_test)

accuracy = accuracy_score(y_test, y_pred)
precision = precision_score(y_test, y_pred, average='weighted')  # 多分类
recall = recall_score(y_test, y_pred, average='weighted')
f1 = f1_score(y_test, y_pred, average='weighted')
balanced_acc = balanced_accuracy_score(y_test, y_pred)  # 适用于不平衡数据
mcc = matthews_corrcoef(y_test, y_pred)  # 马修斯相关系数

print(f"准确率: {accuracy:.3f}")
print(f"精确率: {precision:.3f}")
print(f"召回率: {recall:.3f}")
print(f"F1分数: {f1:.3f}")
print(f"平衡准确率: {balanced_acc:.3f}")
print(f"MCC: {mcc:.3f}")
```

### 分类报告

```python
from sklearn.metrics import classification_report

print(classification_report(y_test, y_pred, target_names=class_names))
```

### 混淆矩阵

```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

cm = confusion_matrix(y_test, y_pred)
disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=class_names)
disp.plot(cmap='Blues')
plt.show()
```

### ROC与AUC

```python
from sklearn.metrics import roc_auc_score, roc_curve, RocCurveDisplay

# 二分类
y_proba = model.predict_proba(X_test)[:, 1]
auc = roc_auc_score(y_test, y_proba)
print(f"ROC AUC: {auc:.3f}")

# 绘制ROC曲线
fpr, tpr, thresholds = roc_curve(y_test, y_proba)
RocCurveDisplay(fpr=fpr, tpr=tpr, roc_auc=auc).plot()

# 多分类（一对多策略）
auc_ovr = roc_auc_score(y_test, y_proba_multi, multi_class='ovr')
```

### 精确率-召回率曲线

```python
from sklearn.metrics import precision_recall_curve, PrecisionRecallDisplay
from sklearn.metrics import average_precision_score

precision, recall, thresholds = precision_recall_curve(y_test, y_proba)
ap = average_precision_score(y_test, y_proba)

disp = PrecisionRecallDisplay(precision=precision, recall=recall, average_precision=ap)
disp.plot()
```

### 对数损失

```python
from sklearn.metrics import log_loss

y_proba = model.predict_proba(X_test)
logloss = log_loss(y_test, y_proba)
print(f"对数损失: {logloss:.3f}")
```

## 回归指标

```python
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error, r2_score,
    mean_absolute_percentage_error, median_absolute_error
)

y_pred = model.predict(X_test)

mse = mean_squared_error(y_test, y_pred)
rmse = mean_squared_error(y_test, y_pred, squared=False)
mae = mean_absolute_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)
mape = mean_absolute_percentage_error(y_test, y_pred)
median_ae = median_absolute_error(y_test, y_pred)

print(f"均方误差: {mse:.3f}")
print(f"均方根误差: {rmse:.3f}")
print(f"平均绝对误差: {mae:.3f}")
print(f"R²分数: {r2:.3f}")
print(f"平均绝对百分比误差: {mape:.3f}")
print(f"中位数绝对误差: {median_ae:.3f}")
```

## 聚类指标

### 有真实标签时

```python
from sklearn.metrics import (
    adjusted_rand_score, normalized_mutual_info_score,
    adjusted_mutual_info_score, fowlkes_mallows_score,
    homogeneity_score, completeness_score, v_measure_score
)

ari = adjusted_rand_score(y_true, y_pred)
nmi = normalized_mutual_info_score(y_true, y_pred)
ami = adjusted_mutual_info_score(y_true, y_pred)
fmi = fowlkes_mallows_score(y_true, y_pred)
homogeneity = homogeneity_score(y_true, y_pred)
completeness = completeness_score(y_true, y_pred)
v_measure = v_measure_score(y_true, y_pred)
```

### 无真实标签时

```python
from sklearn.metrics import (
    silhouette_score, calinski_harabasz_score, davies_bouldin_score
)

silhouette = silhouette_score(X, labels)  # [-1, 1]，值越大越好
ch_score = calinski_harabasz_score(X, labels)  # 值越大越好
db_score = davies_bouldin_score(X, labels)  # 值越小越好
```

## 自定义评分

### 使用make_scorer

```python
from sklearn.metrics import make_scorer

def custom_metric(y_true, y_pred):
    # 自定义逻辑
    return score

custom_scorer = make_scorer(custom_metric, greater_is_better=True)

# 在交叉验证或网格搜索中使用
scores = cross_val_score(model, X, y, cv=5, scoring=custom_scorer)
```

### 网格搜索中的多指标

```python
from sklearn.model_selection import GridSearchCV

scoring = {
    'accuracy': 'accuracy',
    'precision': 'precision_weighted',
    'recall': 'recall_weighted',
    'f1': 'f1_weighted'
}

grid_search = GridSearchCV(
    model, param_grid,
    cv=5,
    scoring=scoring,
    refit='f1',  # 按最佳f1分数重新拟合
    return_train_score=True
)

grid_search.fit(X_train, y_train)
```

## 验证曲线

### 学习曲线

```python
from sklearn.model_selection import learning_curve
import matplotlib.pyplot as plt
import numpy as np

train_sizes, train_scores, val_scores = learning_curve(
    model, X, y,
    cv=5,
    train_sizes=np.linspace(0.1, 1.0, 10),
    scoring='accuracy',
    n_jobs=-1
)

train_mean = train_scores.mean(axis=1)
train_std = train_scores.std(axis=1)
val_mean = val_scores.mean(axis=1)
val_std = val_scores.std(axis=1)

plt.figure(figsize=(10, 6))
plt.plot(train_sizes, train_mean, label='训练分数')
plt.plot(train_sizes, val_mean, label='验证分数')
plt.fill_between(train_sizes, train_mean - train_std, train_mean + train_std, alpha=0.1)
plt.fill_between(train_sizes, val_mean - val_std, val_mean + val_std, alpha=0.1)
plt.xlabel('训练集大小')
plt.ylabel('分数')
plt.title('学习曲线')
plt.legend()
plt.grid(True)
```

### 验证曲线

```python
from sklearn.model_selection import validation_curve

param_range = [1, 10, 50, 100, 200, 500]
train_scores, val_scores = validation_curve(
    model, X, y,
    param_name='n_estimators',
    param_range=param_range,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)

train_mean = train_scores.mean(axis=1)
val_mean = val_scores.mean(axis=1)

plt.figure(figsize=(10, 6))
plt.plot(param_range, train_mean, label='训练分数')
plt.plot(param_range, val_mean, label='验证分数')
plt.xlabel('n_estimators')
plt.ylabel('分数')
plt.title('验证曲线')
plt.legend()
plt.grid(True)
```

## 模型持久化

### 保存与加载模型

```python
import joblib

# 保存模型
joblib.dump(model, 'model.pkl')

# 加载模型
loaded_model = joblib.load('model.pkl')

# 同样适用于流水线
joblib.dump(pipeline, 'pipeline.pkl')
```

### 使用pickle

```python
import pickle

# 保存
with open('model.pkl', 'wb') as f:
    pickle.dump(model, f)

# 加载
with open('model.pkl', 'rb') as f:
    loaded_model = pickle.load(f)
```

## 不平衡数据处理策略

### 类别加权

```python
from sklearn.ensemble import RandomForestClassifier

# 自动平衡类别
model = RandomForestClassifier(class_weight='balanced',

# 自定义权重
class_weights = {0: 1, 1: 10}  # 给类别1分配更高权重
model = RandomForestClassifier(class_weight=class_weights, random_state=42)
```

### 重采样（使用 imbalanced-learn）

```python
# 安装：uv pip install imbalanced-learn
from imblearn.over_sampling import SMOTE
from imblearn.under_sampling import RandomUnderSampler
from imblearn.pipeline import Pipeline as ImbPipeline

# SMOTE过采样
smote = SMOTE(random_state=42)
X_resampled, y_resampled = smote.fit_resample(X_train, y_train)

# 组合方法
pipeline = ImbPipeline([
    ('over', SMOTE(sampling_strategy=0.5)),
    ('under', RandomUnderSampler(sampling_strategy=0.8)),
    ('model', RandomForestClassifier())
])
```

## 最佳实践

### 分层划分
分类问题始终使用分层划分：
```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
```

### 合适的评估指标
- **平衡数据**: 准确率（Accuracy）、F1分数（F1-score）
- **不平衡数据**: 精确率（Precision）、召回率（Recall）、F1分数、ROC AUC、平衡准确率（Balanced Accuracy）
- **代价敏感**: 基于代价定义自定义评分器
- **排序任务**: ROC AUC、平均精确率（Average Precision）

### 交叉验证
- 多数情况使用5折或10折交叉验证
- 分类问题使用StratifiedKFold
- 时间序列使用TimeSeriesSplit
- 分组样本使用GroupKFold

### 嵌套交叉验证
调参时用于无偏性能估计：
```python
from sklearn.model_selection import cross_val_score, GridSearchCV

# 内循环：超参数调优
grid_search = GridSearchCV(model, param_grid, cv=5)

# 外循环：性能评估
scores = cross_val_score(grid_search, X, y, cv=5)
print(f"嵌套交叉验证得分：{scores.mean():.3f} (+/- {scores.std() * 2:.3f})")
```
