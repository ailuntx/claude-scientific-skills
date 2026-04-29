---
name: scikit-learn
description: 使用 scikit-learn 在 Python 中进行机器学习。适用于监督学习（分类、回归）、无监督学习（聚类、降维）、模型评估、超参数调优、预处理或构建机器学习流水线。提供算法、预处理技术、流水线和最佳实践的完整参考文档。
license: BSD-3-Clause 许可证
metadata:
    skill-author: K-Dense Inc.
---

# Scikit-learn

## 概述

本技能提供使用 scikit-learn（经典机器学习的行业标准 Python 库）完成机器学习任务的全面指导。适用于分类、回归、聚类、降维、预处理、模型评估以及构建生产级机器学习流水线。

## 安装

```bash
# 使用 uv 安装 scikit-learn
uv uv pip install scikit-learn

# 可选：安装可视化依赖
uv uv pip install matplotlib seaborn

# 常用配套库
uv uv pip install pandas numpy
```

## 使用场景

在以下场景使用 scikit-learn 技能：

- 构建分类或回归模型
- 执行聚类或降维
- 为机器学习预处理和转换数据
- 通过交叉验证评估模型性能
- 使用网格搜索或随机搜索调优超参数
- 为生产流程创建机器学习流水线
- 比较不同算法解决任务的效果
- 处理结构化（表格）和文本数据
- 需要可解释的经典机器学习方法

## 快速入门

### 分类示例

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report

# 分割数据
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

# 评估
y_pred = model.predict(X_test_scaled)
print(classification_report(y_test, y_pred))
```

### 混合数据完整流水线

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import GradientBoostingClassifier

# 定义特征类型
numeric_features = ['age', 'income']
categorical_features = ['gender', 'occupation']

# 创建预处理流水线
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

# 完整流水线
model = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', GradientBoostingClassifier(random_state=42))
])

# 训练与预测
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

## 核心能力

### 1. 监督学习

提供分类和回归任务的完整算法集。

**关键算法：**
- **线性模型**：逻辑回归、线性回归、岭回归、Lasso、弹性网络
- **树模型**：决策树、随机森林、梯度提升
- **支持向量机**：SVC、SVR（支持多种核函数）
- **集成方法**：AdaBoost、投票法、堆叠法
- **神经网络**：MLPClassifier、MLPRegressor
- **其他**：朴素贝叶斯、K近邻

**适用场景：**
- 分类：预测离散类别（垃圾邮件检测、图像分类、欺诈检测）
- 回归：预测连续值（价格预测、需求预测）

**详见：** `references/supervised_learning.md` 获取算法文档、参数说明和用法示例。

### 2. 无监督学习

通过聚类和降维发现未标记数据的模式。

**聚类算法：**
- **划分式**：K-Means、MiniBatchKMeans
- **密度式**：DBSCAN、HDBSCAN、OPTICS
- **层次式**：AgglomerativeClustering
- **概率式**：高斯混合模型
- **其他**：MeanShift、SpectralClustering、BIRCH

**降维方法：**
- **线性**：PCA、截断SVD、非负矩阵分解
- **流形学习**：t-SNE、UMAP、Isomap、局部线性嵌入
- **特征提取**：FastICA、潜在狄利克雷分布

**适用场景：**
- 客户分群、异常检测、数据可视化
- 降低特征维度、探索性数据分析
- 主题建模、图像压缩

**详见：** `references/unsupervised_learning.md` 获取完整文档。

### 3. 模型评估与选择

提供稳健的模型评估、交叉验证和超参数调优工具。

**交叉验证策略：**
- K折交叉、分层K折交叉（分类）
- 时间序列分割（时序数据）
- 分组K折交叉（分组样本）

**超参数调优：**
- 网格搜索（穷举搜索）
- 随机搜索（随机采样）
- 减半网格搜索（逐次减半）

**评估指标：**
- **分类**：准确率、精确率、召回率、F1值、ROC AUC、混淆矩阵
- **回归**：均方误差、均方根误差、平均绝对误差、R²、平均绝对百分比误差
- **聚类**：轮廓系数、Calinski-Harabasz指数、Davies-Bouldin指数

**适用场景：**
- 客观比较模型性能
- 寻找最优超参数
- 通过交叉验证防止过拟合
- 利用学习曲线理解模型行为

**详见：** `references/model_evaluation.md` 获取完整指标和调优策略。

### 4. 数据预处理

将原始数据转换为适合机器学习的格式。

**缩放与归一化：**
- 标准缩放器（零均值单位方差）
- 最小最大缩放器（限定范围）
- 鲁棒缩放器（抗异常值）
- 归一化器（样本级归一化）

**类别变量编码：**
- 独热编码（无序类别）
- 序数编码（有序类别）
- 标签编码（目标编码）

**缺失值处理：**
- 简单填充器（均值、中位数、众数）
- K近邻填充器
- 迭代填充器（多变量填充）

**特征工程：**
- 多项式特征（交互项）
- 分箱离散化
- 特征选择（递归特征消除、SelectKBest、SelectFromModel）

**适用场景：**
- 训练需要特征缩放的算法前（SVM、KNN、神经网络）
- 将类别变量转为数值格式
- 系统化处理缺失数据
- 为线性模型创建非线性特征

**详见：** `references/preprocessing.md` 获取详细预处理技术。

### 5. 流水线与组合

构建可复现的生产级机器学习工作流。

**核心组件：**
- **流水线**：顺序链接转换器与评估器
- **列转换器**：对不同列应用不同预处理
- **特征联合**：并行组合多个转换器
- **目标转换回归器**：转换目标变量

**优势：**
- 防止交叉验证中的数据泄露
- 简化代码并提高可维护性
- 支持联合超参数调优
- 确保训练与预测的一致性

**适用场景：**
- 生产工作流务必使用流水线
- 混合数值与类别特征时（使用列转换器）
- 包含预处理步骤的交叉验证
- 超参数调优包含预处理参数时

**详见：** `references/pipelines_and_composition.md` 获取完整流水线模式。

## 示例脚本

### 分类流水线

运行包含预处理、模型比较、超参数调优和评估的完整分类工作流：

```bash
python scripts/classification_pipeline.py
```

该脚本演示：
- 混合数据类型处理（数值与类别）
- 使用交叉验证进行模型比较
- 网格搜索超参数调优
- 多指标综合评估
- 特征重要性分析

### 聚类分析

执行包含算法比较和可视化的聚类分析：

```bash
python scripts/clustering_analysis.py
```

该脚本演示：
- 寻找最优聚类数（肘部法则、轮廓分析）
- 比较多种聚类算法（K-Means、DBSCAN、层次聚类、高斯混合）
- 无真实标签的聚类质量评估
- 通过PCA投影可视化结果

## 参考文档

本技能包含特定主题的深度参考文件：

### 速查指南
**文件：** `references/quick_reference.md`
- 常用导入模式与安装说明
- 常见任务快速工作流模板
- 算法选择速查表
- 常见模式与陷阱
- 性能优化技巧

### 监督学习
**文件：** `references/supervised_learning.md`
- 线性模型（回归与分类）
- 支持向量机
- 决策树与集成方法
- K近邻、朴素贝叶斯、神经网络
- 算法选择指南

### 无监督学习
**文件：** `references/unsupervised_learning.md`
- 所有聚类算法参数与用例
- 降维技术
- 异常值检测
- 高斯混合模型
- 方法选择指南

### 模型评估
**文件：** `references/model_evaluation.md`
- 交叉验证策略
- 超参数调优方法
- 分类/回归/聚类评估指标
- 学习曲线与验证曲线
- 模型选择最佳实践

### 预处理
**文件：** `references/preprocessing.md`
- 特征缩放与归一化
- 类别变量编码
- 缺失值填充
- 特征工程技术
- 自定义转换器

### 流水线与组合
**文件：** `references/pipelines_and_composition.md`
- 流水线构建与使用
- 混合数据类型的列转换器
- 并行转换的特征联合
- 完整端到端示例
- 最佳实践

## 常用工作流

### 构建分类模型

1. **加载与探索数据**
   ```python
   import pandas as pd
   df = pd.read_csv('data.csv')
   X = df.drop('target', axis=1)
   y = df['target']
   ```

2. **分层分割数据**
   ```python
   from sklearn.model_selection import train_test_split
   X_train, X_test, y_train, y_test = train_test_split(
       X, y, test_size=0.2, stratify=y, random_state=42
   )
   ```

3. **创建预处理流水线**
   ```python
   from sklearn.pipeline import Pipeline
   from sklearn.preprocessing import StandardScaler
   from sklearn.compose import ColumnTransformer

   # 分别处理数值与类别特征
   preprocessor = ColumnTransformer([
       ('num', StandardScaler(), numeric_features),
       ('cat', OneHotEncoder(), categorical_features)
   ])
   ```

4. **构建完整流水线**
   ```python
   model = Pipeline([
       ('preprocessor', preprocessor),
       ('classifier', RandomForestClassifier(random_state=42))
   ])
   ```

5. **调优超参数**
   ```python
   from sklearn.model_selection import GridSearchCV

   param_grid = {
       'classifier__n_estimators': [100, 200],
       'classifier__max_depth': [10, 20, None]
   }

   grid_search = GridSearchCV(model, param_grid, cv=5)
   grid_search.fit(X_train, y_train)
   ```

6. **测试集评估**
   ```python
   from sklearn.metrics import classification_report

   best_model = grid_search.best_estimator_
   y_pred = best_model.predict(X_test)
   print(classification_report(y_test, y_pred))
   ```

### 执行聚类分析

1. **预处理数据**
   ```python
   from sklearn.preprocessing import StandardScaler

   scaler = StandardScaler()
   X_scaled = scaler.fit_transform(X)
   ```

2. **寻找最优聚类数**
   ```python
   from sklearn.cluster import KMeans
   from sklearn.metrics import silhouette_score

   scores = []
   for k in range(2, 11):
       kmeans = KMeans(n_clusters=k, random_state=42)
       labels = kmeans.fit_predict(X_scaled)
       scores.append(silhouette_score(X_scaled, labels))

   optimal_k = range(2, 11)[np.argmax(scores)]
   ```

3. **应用聚类**
   ```python
   model = KMeans(n_clusters=optimal_k, random_state=42)
   labels = model.fit_predict(X_scaled)
   ```

4. **降维可视化**
   ```python
   from sklearn.decomposition import PCA

   pca = PCA(n_components=2)
   X_2d = pca.fit_transform(X_scaled)

   plt.scatter(X_2d[:, 0], X_2d[:, 1], c=labels, cmap='viridis')
   ```

## 最佳实践

### 始终使用流水线
流水线防止数据泄露并确保一致性：
```python
# 正确：流水线内预处理
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression())
])

# 错误：外部预处理（可能导致信息泄露）
X_scaled = StandardScaler().fit_transform(X)
```

### 仅在训练数据上拟合
切勿在测试数据上拟合：
```python
# 正确
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)  # 仅转换

# 错误
scaler = StandardScaler()
X_all_scaled = scaler.fit_transform(np.vstack([X_train, X_test]))
```

### 分类任务使用分层分割
保持类别分布：
```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
```

### 设置随机种子确保可复现性
```python
model = RandomForestClassifier(n_estimators=100, random_state=42)
```

### 选择合适评估指标
- 平衡数据：准确率、F1值
- 不平衡数据：精确率、召回率、ROC AUC、平衡准确率
- 成本敏感：定义自定义评分器

### 按需缩放特征
需要特征缩放的算法：
- SVM、KNN、神经网络
- PCA、带正则化的线性/逻辑回归
- K-Means聚类

无需缩放的算法：
- 树模型（决策树、随机森林、梯度提升）
- 朴素贝叶斯

## 常见问题排查

### 收敛警告
**问题：** 模型未收敛
**解决：** 增加 `max_iter` 或缩放特征
```python
model = LogisticRegression(max_iter=1000)
```

### 测试集性能差
**问题：** 过拟合
**解决：** 使用正则化、交叉验证或简化模型
```python
# 添加正则化
model = Ridge(alpha=1.0)

# 使用交叉验证
scores = cross_val_score(model, X, y, cv=5)
```

### 大数据集内存错误
**解决：** 使用专为大设计的算法
```python
# 大型数据集使用 SGD
from sklearn.linear_model import SGDClassifier
model = SGDClassifier()

# 聚类使用 MiniBatchKMeans
from sklearn.cluster import MiniBatchKMeans
model = MiniBatchKMeans(n_clusters=8, batch_size=100)
```

## 附加资源

- 官方文档：https://scikit-learn.org/stable/
- 用户指南：https://scikit-learn.org/stable/user_guide.html
- API 参考：https://scikit-learn.org/stable/api/index.html
- 示例库：https://scikit-learn.org/stable/auto_examples/index.html
