# 监督学习参考指南

## 概述

监督学习算法通过带标签的训练数据进行学习，从而对新数据做出预测。Scikit-learn 为分类和回归任务提供了全面的实现方案。

## 线性模型

### 回归

**线性回归 (`sklearn.linear_model.LinearRegression`)**
- 普通最小二乘回归
- 速度快、可解释性强、无需超参数
- 适用场景：线性关系、可解释性要求高
- 示例：
```python
from sklearn.linear_model import LinearRegression

model = LinearRegression()
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```

**岭回归 (`sklearn.linear_model.Ridge`)**
- L2正则化防止过拟合
- 关键参数：`alpha`（正则化强度，默认=1.0）
- 适用场景：存在多重共线性、需要正则化
- 示例：
```python
from sklearn.linear_model import Ridge

model = Ridge(alpha=1.0)
model.fit(X_train, y_train)
```

**Lasso回归 (`sklearn.linear_model.Lasso`)**
- L1正则化配合特征选择
- 关键参数：`alpha`（正则化强度）
- 适用场景：需要稀疏模型、特征选择
- 可将部分系数压缩至零
- 示例：
```python
from sklearn.linear_model import Lasso

model = Lasso(alpha=0.1)
model.fit(X_train, y_train)
# 检查被选中的特征
print(f"非零系数数量: {sum(model.coef_ != 0)}")
```

**弹性网络 (`sklearn.linear_model.ElasticNet`)**
- 结合L1和L2正则化
- 关键参数：`alpha`, `l1_ratio` (0=岭回归, 1=Lasso)
- 适用场景：同时需要特征选择和正则化
- 示例：
```python
from sklearn.linear_model import ElasticNet

model = ElasticNet(alpha=0.1, l1_ratio=0.5)
model.fit(X_train, y_train)
```

### 分类

**逻辑回归 (`sklearn.linear_model.LogisticRegression`)**
- 支持二分类与多分类
- 关键参数：`C`（正则化倒数）, `penalty` ('l1', 'l2', 'elasticnet')
- 返回概率估计值
- 适用场景：需要概率预测、可解释性
- 示例：
```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(C=1.0, max_iter=1000)
model.fit(X_train, y_train)
probas = model.predict_proba(X_test)
```

**随机梯度下降 (SGD)**
- `SGDClassifier`, `SGDRegressor`
- 适用于大规模学习
- 关键参数：`loss`, `penalty`, `alpha`, `learning_rate`
- 适用场景：超大数据集 (>10^4样本)
- 示例：
```python
from sklearn.linear_model import SGDClassifier

model = SGDClassifier(loss='log_loss', max_iter=1000, tol=1e-3)
model.fit(X_train, y_train)
```

## 支持向量机

**SVC (`sklearn.svm.SVC`)**
- 基于核方法的分类器
- 关键参数：`C`, `kernel` ('linear', 'rbf', 'poly'), `gamma`
- 适用场景：中小型数据集、复杂决策边界
- 注意：不适用于超大规模数据集
- 示例：
```python
from sklearn.svm import SVC

# 线性核处理线性可分数据
model_linear = SVC(kernel='linear', C=1.0)

# RBF核处理非线性数据
model_rbf = SVC(kernel='rbf', C=1.0, gamma='scale')
model_rbf.fit(X_train, y_train)
```

**SVR (`sklearn.svm.SVR`)**
- 基于核方法的回归器
- 参数与SVC类似
- 额外参数：`epsilon`（间隔带宽度）
- 示例：
```python
from sklearn.svm import SVR

model = SVR(kernel='rbf', C=1.0, epsilon=0.1)
model.fit(X_train, y_train)
```

## 决策树

**决策树分类器/回归器**
- 基于决策规则的非参数模型
- 关键参数：
  - `max_depth`：最大树深度（防止过拟合）
  - `min_samples_split`：节点分裂最小样本数
  - `min_samples_leaf`：叶节点最小样本数
  - `criterion`：分类用'gini'/'entropy'；回归用'squared_error'/'absolute_error'
- 适用场景：需可解释模型、非线性关系、混合特征类型
- 易过拟合 - 建议使用集成方法或剪枝
- 示例：
```python
from sklearn.tree import DecisionTreeClassifier

model = DecisionTreeClassifier(
    max_depth=5,
    min_samples_split=20,
    min_samples_leaf=10,
    criterion='gini'
)
model.fit(X_train, y_train)

# 可视化决策树
from sklearn.tree import plot_tree
plot_tree(model, feature_names=feature_names, class_names=class_names)
```

## 集成方法

### 随机森林

**随机森林分类器/回归器**
- 基于装袋法的决策树集成
- 关键参数：
  - `n_estimators`：树的数量（默认=100）
  - `max_depth`：最大树深度
  - `max_features`：分裂考虑的特征数 ('sqrt', 'log2', 或整数)
  - `min_samples_split`, `min_samples_leaf`：控制树生长
- 适用场景：需高精度、计算资源充足
- 提供特征重要性评估
- 示例：
```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(
    n_estimators=100,
    max_depth=10,
    max_features='sqrt',
    n_jobs=-1  # 使用所有CPU核心
)
model.fit(X_train, y_train)

# 特征重要性
importances = model.feature_importances_
```

### 梯度提升

**梯度提升分类器/回归器**
- 基于残差顺序构建树的集成方法
- 关键参数：
  - `n_estimators`：提升阶段数量
  - `learning_rate`：收缩每棵树的贡献
  - `max_depth`：单棵树深度（通常3-5）
  - `subsample`：每棵树训练样本比例
- 适用场景：需高精度、训练时间充足
- 通常达到最佳性能
- 示例：
```python
from sklearn.ensemble import GradientBoostingClassifier

model = GradientBoostingClassifier(
    n_estimators=100,
    learning_rate=0.1,
    max_depth=3,
    subsample=0.8
)
model.fit(X_train, y_train)
```

**直方图梯度提升分类器/回归器**
- 基于直方图算法的快速梯度提升
- 原生支持缺失值和类别特征
- 参数与梯度提升类似
- 适用场景：大型数据集、需快速训练
- 示例：
```python
from sklearn.ensemble import HistGradientBoostingClassifier

model = HistGradientBoostingClassifier(
    max_iter=100,
    learning_rate=0.1,
    max_depth=None,  # 默认无深度限制
    categorical_features='from_dtype'  # 自动检测类别特征
)
model.fit(X_train, y_train)
```

### 其他集成方法

**AdaBoost**
- 聚焦误分类样本的自适应提升
- 关键参数：`n_estimators`, `learning_rate`, `estimator`（基学习器）
- 适用场景：需要简单提升策略
- 示例：
```python
from sklearn.ensemble import AdaBoostClassifier

model = AdaBoostClassifier(n_estimators=50, learning_rate=1.0)
model.fit(X_train, y_train)
```

**投票分类器/回归器**
- 组合多个模型的预测结果
- 类型：'hard'（多数表决）或'soft'（概率平均）
- 适用场景：需集成不同模型类型
- 示例：
```python
from sklearn.ensemble import VotingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.svm import SVC

model = VotingClassifier(
    estimators=[
        ('lr', LogisticRegression()),
        ('dt', DecisionTreeClassifier()),
        ('svc', SVC(probability=True))
    ],
    voting='soft'
)
model.fit(X_train, y_train)
```

**堆叠分类器/回归器**
- 在基模型预测上训练元模型
- 比投票法更复杂
- 关键参数：`final_estimator`（元学习器）
- 示例：
```python
from sklearn.ensemble import StackingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.svm import SVC

model = StackingClassifier(
    estimators=[
        ('dt', DecisionTreeClassifier()),
        ('svc', SVC())
    ],
    final_estimator=LogisticRegression()
)
model.fit(X_train, y_train)
```

## K近邻算法

**K近邻分类器/回归器**
- 基于距离的非参数方法
- 关键参数：
  - `n_neighbors`：邻居数量（默认=5）
  - `weights`：'uniform' 或 'distance'
  - `metric`：距离度量 ('euclidean', 'manhattan'等)
- 适用场景：小型数据集、需简单基线模型
- 大型数据集预测速度慢
- 示例：
```python
from sklearn.neighbors import KNeighborsClassifier

model = KNeighborsClassifier(n_neighbors=5, weights='distance')
model.fit(X_train, y_train)
```

## 朴素贝叶斯

**高斯/多项式/伯努利朴素贝叶斯**
- 基于贝叶斯定理的概率分类器
- 训练和预测速度快
- GaussianNB：连续特征（假设高斯分布）
- MultinomialNB：计数特征（文本分类）
- BernoulliNB：二值特征
- 适用场景：文本分类、快速基线、概率预测
- 示例：
```python
from sklearn.naive_bayes import GaussianNB, MultinomialNB

# 连续特征
model_gaussian = GaussianNB()

# 文本/计数数据
model_multinomial = MultinomialNB(alpha=1.0)  # alpha为平滑参数
model_multinomial.fit(X_train, y_train)
```

## 神经网络

**多层感知器分类器/回归器**
- 多层感知器（前馈神经网络）
- 关键参数：
  - `hidden_layer_sizes`：隐藏层尺寸元组，如(100, 50)
  - `activation`：'relu', 'tanh', 'logistic'
  - `solver`：'adam', 'sgd', 'lbfgs'
  - `alpha`：L2正则化参数
  - `learning_rate`：'constant', 'adaptive'
- 适用场景：复杂非线性模式、大型数据集
- 需特征缩放
- 示例：
```python
from sklearn.neural_network import MLPClassifier
from sklearn.preprocessing import StandardScaler

# 先缩放特征
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)

model = MLPClassifier(
    hidden_layer_sizes=(100, 50),
    activation='relu',
    solver='adam',
    alpha=0.0001,
    max_iter=1000
)
model.fit(X_train_scaled, y_train)
```

## 算法选择指南

### 选择依据

**数据集规模：**
- 小型 (<1k样本)：KNN, SVM, 决策树
- 中型 (1k-100k)：随机森林, 梯度提升, 线性模型
- 大型 (>100k)：SGD, 线性模型, 直方图梯度提升

**可解释性：**
- 高：线性模型, 决策树
- 中：随机森林（特征重要性）
- 低：RBF核SVM, 神经网络

**精度 vs 速度：**
- 训练快：朴素贝叶斯, 线性模型, KNN
- 精度高：梯度提升, 随机森林, 堆叠法
- 预测快：线性模型, 朴素贝叶斯
- 预测慢：KNN（大型数据集）, SVM

**特征类型：**
- 连续型：多数算法适用
- 类别型：决策树, 直方图梯度提升（原生支持）
- 混合型：决策树, 梯度提升
- 文本型：朴素贝叶斯, 结合TF-IDF的线性模型

**常用起点：**
1. 逻辑回归（分类）/线性回归（回归） - 快速基线
2. 随机森林 - 优质默认选择
3. 梯度提升 - 追求最佳精度
