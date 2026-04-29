# Scikit-learn 快速参考指南

## 常用导入模式

```python
# 核心 scikit-learn
import sklearn

# 数据拆分与交叉验证
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV

# 数据预处理
from sklearn.preprocessing import StandardScaler, MinMaxScaler, OneHotEncoder
from sklearn.impute import SimpleImputer

# 特征选择
from sklearn.feature_selection import SelectKBest, RFE

# 监督学习
from sklearn.linear_model import LogisticRegression, Ridge, Lasso
from sklearn.ensemble import RandomForestClassifier, GradientBoostingRegressor
from sklearn.svm import SVC, SVR
from sklearn.tree import DecisionTreeClassifier

# 无监督学习
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.decomposition import PCA, NMF

# 评估指标
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    mean_squared_error, r2_score, confusion_matrix, classification_report
)

# 管道工具
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.compose import ColumnTransformer, make_column_transformer

# 实用工具
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

## 安装指南

```bash
# 使用 uv（推荐）
uv pip install scikit-learn

# 可选依赖项
uv pip install scikit-learn[plots]  # 绘图工具
uv pip install pandas numpy matplotlib seaborn  # 常用配套库
```

## 快速工作流模板

### 分类流程

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, confusion_matrix

# 拆分数据
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)

# 预处理
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 训练模型
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train_scaled, y_train)

# 模型评估
y_pred = model.predict(X_test_scaled)
print(classification_report(y_test, y_pred))
print(confusion_matrix(y_test, y_pred))
```

### 回归流程

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.metrics import mean_squared_error, r2_score

# 数据拆分
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 预处理与训练
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

model = GradientBoostingRegressor(n_estimators=100, random_state=42)
model.fit(X_train_scaled, y_train)

# 模型评估
y_pred = model.predict(X_test_scaled)
print(f"RMSE: {mean_squared_error(y_test, y_pred, squared=False):.3f}")
print(f"R² 分数: {r2_score(y_test, y_pred):.3f}")
```

### 交叉验证

```python
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=100, random_state=42)
scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')
print(f"交叉验证准确率: {scores.mean():.3f} (± {scores.std() * 2:.3f})")
```

### 混合数据类型的完整管道

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import RandomForestClassifier

# 定义特征类型
numeric_features = ['age', 'income']
categorical_features = ['gender', 'occupation']

# 创建预处理管道
numeric_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

categorical_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='most_frequent')),
    ('onehot', OneHotEncoder(handle_unknown='ignore'))
])

# 组合转换器
preprocessor = ColumnTransformer([
    ('num', numeric_transformer, numeric_features),
    ('cat', categorical_transformer, categorical_features)
])

# 完整管道
model = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier(n_estimators=100, random_state=42))
])

# 训练与预测
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

### 超参数调优

```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier

param_grid = {
    'n_estimators': [100, 200, 300],
    'max_depth': [10, 20, None],
    'min_samples_split': [2, 5, 10]
}

model = RandomForestClassifier(random_state=42)
grid_search = GridSearchCV(
    model, param_grid, cv=5, scoring='accuracy', n_jobs=-1
)

grid_search.fit(X_train, y_train)
print(f"最佳参数: {grid_search.best_params_}")
print(f"最佳分数: {grid_search.best_score_:.3f}")

# 使用最优模型
best_model = grid_search.best_estimator_
```

## 常用模式

### 数据加载

```python
# 从 scikit-learn 数据集加载
from sklearn.datasets import load_iris, load_digits, make_classification

# 内置数据集
iris = load_iris()
X, y = iris.data, iris.target

# 合成数据
X, y = make_classification(
    n_samples=1000, n_features=20, n_classes=2, random_state=42
)

# 从 pandas 加载
import pandas as pd
df = pd.read_csv('data.csv')
X = df.drop('target', axis=1)
y = df['target']
```

### 处理不平衡数据

```python
from sklearn.ensemble import RandomForestClassifier

# 使用 class_weight 参数
model = RandomForestClassifier(class_weight='balanced', random_state=42)
model.fit(X_train, y_train)

# 或使用合适指标
from sklearn.metrics import balanced_accuracy_score, f1_score
print(f"平衡准确率: {balanced_accuracy_score(y_test, y_pred):.3f}")
print(f"F1 分数: {f1_score(y_test, y_pred):.3f}")
```

### 特征重要性分析

```python
from sklearn.ensemble import RandomForestClassifier
import pandas as pd

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# 获取特征重要性
importances = pd.DataFrame({
    'feature': feature_names,
    'importance': model.feature_importances_
}).sort_values('importance', ascending=False)

print(importances.head(10))
```

### 聚类分析

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

# 先缩放数据
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 训练 K-Means
kmeans = KMeans(n_clusters=3, random_state=42)
labels = kmeans.fit_predict(X_scaled)

# 评估模型
from sklearn.metrics import silhouette_score
score = silhouette_score(X_scaled, labels)
print(f"轮廓系数: {score:.3f}")
```

### 降维处理

```python
from sklearn.decomposition import PCA
import matplotlib.pyplot as plt

# 训练 PCA
pca = PCA(n_components=2)
X_reduced = pca.fit_transform(X)

# 可视化
plt.scatter(X_reduced[:, 0], X_reduced[:, 1], c=y, cmap='viridis')
plt.xlabel('主成分1')
plt.ylabel('主成分2')
plt.title(f'PCA (解释方差: {pca.explained_variance_ratio_.sum():.2%})')
```

### 模型持久化

```python
import joblib

# 保存模型
joblib.dump(model, 'model.pkl')

# 加载模型
loaded_model = joblib.load('model.pkl')
predictions = loaded_model.predict(X_new)
```

## 常见陷阱与解决方案

### 数据泄露问题
```python
# 错误：在整个数据集上拟合缩放器
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
X_train, X_test = train_test_split(X_scaled)

# 正确：仅在训练数据上拟合
X_train, X_test = train_test_split(X)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 最佳：使用管道
from sklearn.pipeline import Pipeline
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression())
])
pipeline.fit(X_train, y_train)  # 避免泄露！
```

### 分类问题的分层抽样
```python
# 分类问题始终使用分层抽样
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
```

### 设置随机种子保证可复现性
```python
# 设置 random_state 确保结果可复现
model = RandomForestClassifier(n_estimators=100, random_state=42)
```

### 处理未知类别
```python
# 在 OneHotEncoder 中使用 handle_unknown='ignore'
encoder = OneHotEncoder(handle_unknown='ignore')
```

### 管道中的特征名称
```python
# 获取转换后的特征名称
preprocessor.fit(X_train)
feature_names = preprocessor.get_feature_names_out()
```

## 算法选择速查表

### 分类算法

| 问题类型 | 算法 | 适用场景 |
|---------|-----------|-------------|
| 二分类/多分类 | 逻辑回归 | 快速基线，可解释性强 |
| 二分类/多分类 | 随机森林 | 稳健的默认选择 |
| 二分类/多分类 | 梯度提升 | 追求最高精度，可调参 |
| 二分类/多分类 | 支持向量机 | 小数据集，复杂边界 |
| 二分类/多分类 | 朴素贝叶斯 | 文本分类，速度快 |
| 高维特征 | 线性SVM或逻辑回归 | 文本数据，特征众多 |

### 回归算法

| 问题类型 | 算法 | 适用场景 |
|---------|-----------|-------------|
| 连续目标 | 线性回归 | 快速基线，可解释性强 |
| 连续目标 | 岭回归/Lasso | 需要正则化 |
| 连续目标 | 随机森林 | 非线性关系默认选择 |
| 连续目标 | 梯度提升 | 追求最高精度 |
| 连续目标 | 支持向量回归 | 小数据集，非线性 |

### 聚类算法

| 问题类型 | 算法 | 适用场景 |
|---------|-----------|-------------|
| 已知K值，球形分布 | K-Means | 快速简单 |
| 未知K值，任意形状 | DBSCAN | 存在噪声/离群点 |
| 层次结构 | 凝聚聚类 | 需要树状图 |
| 软聚类 | 高斯混合模型 | 需要概率估计 |

### 降维算法

| 问题类型 | 算法 | 适用场景 |
|---------|-----------|-------------|
| 线性降维 | 主成分分析 | 方差解释 |
| 可视化 | t-SNE | 2D/3D绘图 |
| 非负数据 | 非负矩阵分解 | 图像，文本 |
| 稀疏数据 | 截断SVD | 文本，推荐系统 |

## 性能优化技巧

### 加速训练
```python
# 使用 n_jobs=-1 进行并行处理
model = RandomForestClassifier(n_estimators=100, n_jobs=-1)

# 使用 warm_start 增量训练
model = RandomForestClassifier(n_estimators=100, warm_start=True)
model.fit(X, y)
model.n_estimators += 50
model.fit(X, y)  # 增加50棵树

# 使用 partial_fit 在线学习
from sklearn.linear_model import SGDClassifier
model = SGDClassifier()
for X_batch, y_batch in batches:
    model.partial_fit(X_batch, y_batch, classes=np.unique(y))
```

### 内存优化
```python
# 使用稀疏矩阵
from scipy.sparse import csr_matrix
X_sparse = csr_matrix(X)

# 大数据集使用 MiniBatchKMeans
from sklearn.cluster import MiniBatchKMeans
model = MiniBatchKMeans(n_clusters=8, batch_size=100)
```

## 版本检查

```python
import sklearn
print(f"scikit-learn 版本: {sklearn.__version__}")
```

## 实用资源

- 官方文档: https://scikit-learn.org/stable/
- 用户指南: https://scikit-learn.org/stable/user_guide.html
- API参考: https://scikit-learn.org/stable/api/index.html
- 示例库: https://scikit-learn.org/stable/auto_examples/index.html
- 教程: https://scikit-learn.org/stable/tutorial/index.html
