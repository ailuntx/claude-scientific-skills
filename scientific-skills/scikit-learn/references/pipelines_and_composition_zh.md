# 管道与复合估计器参考指南

## 概述

管道将多个处理步骤串联为单一估计器，防止数据泄露并简化代码。它们支持可复现的工作流，并能无缝集成交叉验证与超参数调优。

## 管道基础

### 创建管道

**Pipeline (`sklearn.pipeline.Pipeline`)**
- 将转换器与最终估计器串联
- 所有中间步骤必须实现 fit_transform()
- 最终步骤可为任意估计器（转换器、分类器、回归器、聚类器）
- 示例：
```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.linear_model import LogisticRegression

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA(n_components=10)),
    ('classifier', LogisticRegression())
])

# 训练整个管道
pipeline.fit(X_train, y_train)

# 使用管道预测
y_pred = pipeline.predict(X_test)
y_proba = pipeline.predict_proba(X_test)
```

### 使用 make_pipeline

**make_pipeline**
- 自动生成步骤名称的便捷构造器
- 示例：
```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC

pipeline = make_pipeline(
    StandardScaler(),
    PCA(n_components=10),
    SVC(kernel='rbf')
)

pipeline.fit(X_train, y_train)
```

## 访问管道组件

### 访问步骤

```python
# 通过索引
scaler = pipeline.steps[0][1]

# 通过名称
scaler = pipeline.named_steps['scaler']
pca = pipeline.named_steps['pca']

# 使用索引语法
scaler = pipeline['scaler']
pca = pipeline['pca']

# 获取所有步骤名称
print(pipeline.named_steps.keys())
```

### 设置参数

```python
# 使用双下划线表示法设置参数
pipeline.set_params(
    pca__n_components=15,
    classifier__C=0.1
)

# 或在创建时设置
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA(n_components=10)),
    ('classifier', LogisticRegression(C=1.0))
])
```

### 访问属性

```python
# 访问已训练属性
pca_components = pipeline.named_steps['pca'].components_
explained_variance = pipeline.named_steps['pca'].explained_variance_ratio_

# 访问中间转换结果
X_scaled = pipeline.named_steps['scaler'].transform(X_test)
X_pca = pipeline.named_steps['pca'].transform(X_scaled)
```

## 管道超参数调优

### 管道网格搜索

```python
from sklearn.model_selection import GridSearchCV
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', SVC())
])

param_grid = {
    'classifier__C': [0.1, 1, 10, 100],
    'classifier__gamma': ['scale', 'auto', 0.001, 0.01],
    'classifier__kernel': ['rbf', 'linear']
}

grid_search = GridSearchCV(pipeline, param_grid, cv=5, n_jobs=-1)
grid_search.fit(X_train, y_train)

print(f"最佳参数: {grid_search.best_params_}")
print(f"最佳分数: {grid_search.best_score_:.3f}")
```

### 调优多个管道步骤

```python
param_grid = {
    # PCA参数
    'pca__n_components': [5, 10, 20, 50],

    # 分类器参数
    'classifier__C': [0.1, 1, 10],
    'classifier__kernel': ['rbf', 'linear']
}

grid_search = GridSearchCV(pipeline, param_grid, cv=5)
grid_search.fit(X_train, y_train)
```

## ColumnTransformer

### 基础用法

**ColumnTransformer (`sklearn.compose.ColumnTransformer`)**
- 对不同列应用不同预处理
- 防止交叉验证中的数据泄露
- 示例：
```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer

# 定义列分组
numeric_features = ['age', 'income', 'hours_per_week']
categorical_features = ['gender', 'occupation', 'native_country']

# 创建预处理器
preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), numeric_features),
        ('cat', OneHotEncoder(handle_unknown='ignore'), categorical_features)
    ],
    remainder='passthrough'  # 保留其他列不变
)

X_transformed = preprocessor.fit_transform(X)
```

### 与管道步骤结合

```python
from sklearn.pipeline import Pipeline

numeric_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

categorical_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='constant', fill_value='missing')),
    ('onehot', OneHotEncoder(handle_unknown='ignore'))
])

preprocessor = ColumnTransformer(
    transformers=[
        ('num', numeric_transformer, numeric_features),
        ('cat', categorical_transformer, categorical_features)
    ]
)

# 包含模型的完整管道
full_pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', LogisticRegression())
])

full_pipeline.fit(X_train, y_train)
```

### 使用 make_column_transformer

```python
from sklearn.compose import make_column_transformer

preprocessor = make_column_transformer(
    (StandardScaler(), numeric_features),
    (OneHotEncoder(), categorical_features),
    remainder='passthrough'
)
```

### 列选择方法

```python
# 按列名（当X为DataFrame时）
preprocessor = ColumnTransformer([
    ('num', StandardScaler(), ['age', 'income']),
    ('cat', OneHotEncoder(), ['gender', 'occupation'])
])

# 按列索引
preprocessor = ColumnTransformer([
    ('num', StandardScaler(), [0, 1, 2]),
    ('cat', OneHotEncoder(), [3, 4])
])

# 按布尔掩码
numeric_mask = [True, True, True, False, False]
categorical_mask = [False, False, False, True, True]

preprocessor = ColumnTransformer([
    ('num', StandardScaler(), numeric_mask),
    ('cat', OneHotEncoder(), categorical_mask)
])

# 通过可调用函数
def is_numeric(X):
    return X.select_dtypes(include=['number']).columns.tolist()

preprocessor = ColumnTransformer([
    ('num', StandardScaler(), is_numeric)
])
```

### 获取特征名称

```python
# 获取输出特征名
feature_names = preprocessor.get_feature_names_out()

# 训练后获取
preprocessor.fit(X_train)
output_features = preprocessor.get_feature_names_out()
print(f"输入特征: {X_train.columns.tolist()}")
print(f"输出特征: {output_features}")
```

### 剩余列处理

```python
# 丢弃未指定列（默认）
preprocessor = ColumnTransformer([...], remainder='drop')

# 原样保留
preprocessor = ColumnTransformer([...], remainder='passthrough')

# 对剩余列应用转换器
preprocessor = ColumnTransformer([...], remainder=StandardScaler())
```

## FeatureUnion

### 基础用法

**FeatureUnion (`sklearn.pipeline.FeatureUnion`)**
- 拼接多个转换器的结果
- 转换器并行执行
- 示例：
```python
from sklearn.pipeline import FeatureUnion
from sklearn.decomposition import PCA
from sklearn.feature_selection import SelectKBest

# 组合PCA与特征选择
feature_union = FeatureUnion([
    ('pca', PCA(n_components=10)),
    ('select_best', SelectKBest(k=20))
])

X_combined = feature_union.fit_transform(X_train, y_train)
print(f"组合特征数: {X_combined.shape[1]}")  # 10 + 20 = 30
```

### 与管道结合

```python
from sklearn.pipeline import Pipeline, FeatureUnion
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA, TruncatedSVD

# 创建特征联合
feature_union = FeatureUnion([
    ('pca', PCA(n_components=10)),
    ('svd', TruncatedSVD(n_components=10))
])

# 完整管道
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('features', feature_union),
    ('classifier', LogisticRegression())
])

pipeline.fit(X_train, y_train)
```

### 加权特征联合

```python
# 为转换器分配权重
feature_union = FeatureUnion(
    transformer_list=[
        ('pca', PCA(n_components=10)),
        ('select_best', SelectKBest(k=20))
    ],
    transformer_weights={
        'pca': 2.0,  # PCA特征权重加倍
        'select_best': 1.0
    }
)
```

## 高级管道模式

### 缓存管道步骤

```python
from sklearn.pipeline import Pipeline
from tempfile import mkdtemp
from shutil import rmtree

# 缓存中间结果
cachedir = mkdtemp()
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA(n_components=50)),
    ('classifier', LogisticRegression())
], memory=cachedir)

pipeline.fit(X_train, y_train)

# 清理缓存
rmtree(cachedir)
```

### 嵌套管道

```python
from sklearn.pipeline import Pipeline

# 文本处理内部管道
text_pipeline = Pipeline([
    ('vect', CountVectorizer()),
    ('tfidf', TfidfTransformer())
])

# 组合文本与数值特征的完整管道
full_pipeline = Pipeline([
    ('features', FeatureUnion([
        ('text', text_pipeline),
        ('numeric', StandardScaler())
    ])),
    ('classifier', LogisticRegression())
])
```

### 管道中的自定义转换器

```python
from sklearn.base import BaseEstimator, TransformerMixin

class TextLengthExtractor(BaseEstimator, TransformerMixin):
    def fit(self, X, y=None):
        return self

    def transform(self, X):
        return [[len(text)] for text in X]

pipeline = Pipeline([
    ('length', TextLengthExtractor()),
    ('scaler', StandardScaler()),
    ('classifier', LogisticRegression())
])
```

### 管道切片

```python
# 获取子管道
sub_pipeline = pipeline[:2]  # 前两个步骤

# 获取特定范围
middle_steps = pipeline[1:3]
```

## TransformedTargetRegressor

### 基础用法

**TransformedTargetRegressor**
- 训练前转换目标变量
- 自动反向转换预测结果
- 示例：
```python
from sklearn.compose import TransformedTargetRegressor
from sklearn.preprocessing import QuantileTransformer
from sklearn.linear_model import LinearRegression

model = TransformedTargetRegressor(
    regressor=LinearRegression(),
    transformer=QuantileTransformer(output_distribution='normal')
)

model.fit(X_train, y_train)
y_pred = model.predict(X_test)  # 自动反向转换
```

### 使用函数

```python
import numpy as np

model = TransformedTargetRegressor(
    regressor=LinearRegression(),
    func=np.log1p,
    inverse_func=np.expm1
)

model.fit(X_train, y_train)
```

## 完整示例：端到端管道

```python
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.decomposition import PCA
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import GridSearchCV

# 定义特征类型
numeric_features = ['age', 'income', 'hours_per_week']
categorical_features = ['gender', 'occupation', 'education']

# 数值预处理管道
numeric_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

# 类别预处理管道
categorical_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='constant', fill_value='missing')),
    ('onehot', OneHotEncoder(handle_unknown='ignore', sparse_output=False))
])

# 组合预处理器
preprocessor = ColumnTransformer(
    transformers=[
        ('num', numeric_transformer, numeric_features),
        ('cat', categorical_transformer, categorical_features)
    ]
)

# 完整管道
pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('pca', PCA(n_components=0.95)),  # 保留95%方差
    ('classifier', RandomForestClassifier(random_state=42))
])

# 超参数调优
param_grid = {
    'preprocessor__num__imputer__strategy': ['mean', 'median'],
    'pca__n_components': [0.90, 0.95, 0.99],
    'classifier__n_estimators': [100, 200],
    'classifier__max_depth': [10, 20, None]
}

grid_search = GridSearchCV(
    pipeline, param_grid,
    cv=5, scoring='accuracy',
    n_jobs=-1, verbose=1
)

grid_search.fit(X_train, y_train)

print(f"最佳参数: {grid_search.best_params_}")
print(f"最佳CV分数: {grid_search.best_score_:.3f}")
print(f"测试分数: {grid_search.score(X_test, y_test):.3f}")

# 进行预测
best_pipeline = grid_search.best_estimator_
y_pred = best_pipeline.predict(X_test)
y_proba = best_pipeline.predict_proba(X_test)
```

## 可视化

### 显示管道结构

```python
# 在Jupyter中管道显示为图表
from sklearn import set_config
set_config(display='diagram')

pipeline  # 显示可视化图表
```

### 文本表示

```python
# 打印管道结构
print(pipeline)

# 获取详细参数
print(pipeline.get_params())
```

## 最佳实践

### 始终使用管道
- 防止数据泄露
- 确保训练与预测的一致性
- 提高代码可维护性
- 便于超参数调优

### 正确构建管道
```python
# 正确：预处理在管道内
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression())
])
pipeline.fit(X_train, y_train)

# 错误：管道外预处理（可能导致泄露）
X_train_scaled = StandardScaler().fit_transform(X_train)
model = LogisticRegression()
model.fit(X_train_scaled, y_train)
```

### 混合数据使用ColumnTransformer
当同时存在数值和类别特征时始终使用ColumnTransformer：
```python
preprocessor = ColumnTransformer([
    ('num', StandardScaler(), numeric_features),
    ('cat', OneHotEncoder(), categorical_features)
])
```

### 为步骤命名需有意义
```python
# 正确
pipeline = Pipeline([
    ('imputer', SimpleImputer()),
    ('scaler', StandardScaler()),
    ('pca', PCA(n_components=10)),
    ('rf_classifier', RandomForestClassifier())
])

# 错误
pipeline = Pipeline([
    ('step1', SimpleImputer()),
    ('step2', StandardScaler()),
    ('step3', PCA(n_components=10)),
    ('step4', RandomForestClassifier())
])
```

### 缓存高开销转换
在重复训练时（如网格搜索），缓存高开销步骤：
```python
from tempfile import mkdtemp

cachedir = mkdtemp()
pipeline = Pipeline([
    ('expensive_preprocessing', ExpensiveTransformer()),
    ('classifier', LogisticRegression())
], memory=cachedir)
```

### 测试管道兼容性
确保所有步骤兼容：
- 所有中间步骤需实现 fit() 和 transform()
- 最终步骤需实现 fit() 和 predict()（或 transform()）
- 使用 set_output(transform='pandas') 获取DataFrame输出
```python
pipeline.set_output(transform='pandas')
X_transformed = pipeline.transform(X)  # 返回DataFrame
```
