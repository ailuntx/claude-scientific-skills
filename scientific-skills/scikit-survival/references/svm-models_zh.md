# 生存支持向量机

## 概述

生存支持向量机（SVMs）将传统SVM框架适配于含删失数据的生存分析。该方法通过优化排序目标函数，确保生存时间的正确排序。

### 核心思想

生存分析中的支持向量机学习一个生成风险评分函数f(x)，其优化过程保证生存时间较短的受试者获得比生存时间较长者更高的风险评分。

## 适用场景

**适用情况：**
- 中等规模数据集（通常100-10,000样本）
- 需要非线性决策边界（核SVM）
- 期望基于间隔的正则化学习
- 特征空间定义明确

**不适用情况：**
- 超大规模数据集（>100,000样本）——集成方法可能更快
- 需要可解释系数——改用Cox模型
- 需要生存函数估计——使用随机生存森林
- 超高维数据——使用正则化Cox或梯度提升

## 模型类型

### FastSurvivalSVM

采用坐标下降优化的线性生存SVM，速度优先。

**适用场景：**
- 预期存在线性关系
- 大型数据集且速度敏感
- 需要快速训练和预测

**关键参数：**
- `alpha`：正则化参数（默认：1.0）
  - 值越高正则化越强
- `rank_ratio`：排序与回归的权衡参数（默认：1.0）
- `max_iter`：最大迭代次数（默认：20）
- `tol`：停止条件容差（默认：1e-5）

```python
from sksurv.svm import FastSurvivalSVM

# 拟合线性生存SVM
estimator = FastSurvivalSVM(alpha=1.0, max_iter=100, tol=1e-5, random_state=42)
estimator.fit(X, y)

# 预测风险评分
risk_scores = estimator.predict(X_test)
```

### FastKernelSurvivalSVM

处理非线性关系的核生存SVM。

**适用场景：**
- 特征与生存存在非线性关系
- 中等规模数据集
- 可接受更长训练时间换取更好性能

**核函数选项：**
- `'linear'`：线性核（等同于FastSurvivalSVM）
- `'poly'`：多项式核
- `'rbf'`：径向基函数（高斯）核——最常用
- `'sigmoid'`：Sigmoid核
- 自定义核函数

**关键参数：**
- `alpha`：正则化参数（默认：1.0）
- `kernel`：核函数（默认：'rbf'）
- `gamma`：rbf/poly/sigmoid的核系数
- `degree`：多项式核阶数
- `coef0`：poly/sigmoid的独立项
- `rank_ratio`：权衡参数（默认：1.0）
- `max_iter`：最大迭代次数（默认：20）

```python
from sksurv.svm import FastKernelSurvivalSVM

# 拟合RBF核生存SVM
estimator = FastKernelSurvivalSVM(
    alpha=1.0,
    kernel='rbf',
    gamma='scale',
    max_iter=50,
    random_state=42
)
estimator.fit(X, y)

# 预测风险评分
risk_scores = estimator.predict(X_test)
```

### HingeLossSurvivalSVM

采用合页损失的生存SVM，更接近分类SVM。

**适用场景：**
- 需使用合页损失替代平方合页损失
- 期望稀疏解
- 需要类似分类SVM的行为

**关键参数：**
- `alpha`：正则化参数
- `fit_intercept`：是否拟合截距项（默认：False）

```python
from sksurv.svm import HingeLossSurvivalSVM

# 拟合合页损失SVM
estimator = HingeLossSurvivalSVM(alpha=1.0, fit_intercept=False, random_state=42)
estimator.fit(X, y)

# 预测风险评分
risk_scores = estimator.predict(X_test)
```

### NaiveSurvivalSVM

基于二次规划的原始生存SVM实现。

**适用场景：**
- 小型数据集
- 研究/基准测试
- 其他方法不收敛时

**局限性：**
- 比Fast系列更慢
- 可扩展性差

```python
from sksurv.svm import NaiveSurvivalSVM

# 拟合原始SVM（较慢）
estimator = NaiveSurvivalSVM(alpha=1.0, random_state=42)
estimator.fit(X, y)

# 预测
risk_scores = estimator.predict(X_test)
```

### MinlipSurvivalAnalysis

采用最小化Lipschitz常数方法的生存分析模型。

**适用场景：**
- 需要不同优化目标
- 研究应用
- 作为标准生存SVM的替代方案

```python
from sksurv.svm import MinlipSurvivalAnalysis

# 拟合Minlip模型
estimator = MinlipSurvivalAnalysis(alpha=1.0, random_state=42)
estimator.fit(X, y)

# 预测
risk_scores = estimator.predict(X_test)
```

## 超参数调优

### 调整Alpha（正则化）

```python
from sklearn.model_selection import GridSearchCV
from sksurv.metrics import as_concordance_index_ipcw_scorer

# 定义参数网格
param_grid = {
    'alpha': [0.1, 0.5, 1.0, 5.0, 10.0, 50.0]
}

# 网格搜索
cv = GridSearchCV(
    FastSurvivalSVM(),
    param_grid,
    scoring=as_concordance_index_ipcw_scorer(),
    cv=5,
    n_jobs=-1
)
cv.fit(X, y)

print(f"最佳alpha: {cv.best_params_['alpha']}")
print(f"最佳C指数: {cv.best_score_:.3f}")
```

### 调整核参数

```python
from sklearn.model_selection import GridSearchCV

# 定义核SVM参数网格
param_grid = {
    'alpha': [0.1, 1.0, 10.0],
    'gamma': ['scale', 'auto', 0.001, 0.01, 0.1, 1.0]
}

# 网格搜索
cv = GridSearchCV(
    FastKernelSurvivalSVM(kernel='rbf'),
    param_grid,
    scoring=as_concordance_index_ipcw_scorer(),
    cv=5,
    n_jobs=-1
)
cv.fit(X, y)

print(f"最佳参数: {cv.best_params_}")
print(f"最佳C指数: {cv.best_score_:.3f}")
```

## 临床核变换

### ClinicalKernelTransform

专为医疗应用设计的特殊核函数，整合临床特征与分子数据以提升预测效果。

**使用场景：**
- 同时包含临床变量（年龄、分期等）和高维分子数据（基因表达、基因组学）
- 临床特征需差异化加权
- 需整合异构数据类型

**关键参数：**
- `fit_once`：是否在交叉验证中仅拟合一次核（默认：False）
- 临床特征应与分子特征分开传递

```python
from sksurv.kernels import ClinicalKernelTransform
from sksurv.svm import FastKernelSurvivalSVM
from sklearn.pipeline import make_pipeline

# 分离临床与分子特征
clinical_features = ['age', 'stage', 'grade']
X_clinical = X[clinical_features]
X_molecular = X.drop(clinical_features, axis=1)

# 创建含临床核的流水线
estimator = make_pipeline(
    ClinicalKernelTransform(),
    FastKernelSurvivalSVM()
)

# 拟合模型
# ClinicalKernelTransform要求输入元组(临床特征, 分子特征)
X_combined = list(zip(X_clinical.values, X_molecular.values))
estimator.fit(X_combined, y)
```

## 实践案例

### 案例1：带交叉验证的线性SVM

```python
from sksurv.svm import FastSurvivalSVM
from sklearn.model_selection import cross_val_score
from sksurv.metrics import as_concordance_index_ipcw_scorer
from sklearn.preprocessing import StandardScaler

# 特征标准化（对SVM至关重要！）
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 创建模型
svm = FastSurvivalSVM(alpha=1.0, max_iter=100, random_state=42)

# 交叉验证
scores = cross_val_score(
    svm, X_scaled, y,
    cv=5,
    scoring=as_concordance_index_ipcw_scorer(),
    n_jobs=-1
)

print(f"平均C指数: {scores.mean():.3f} (±{scores.std():.3f})")
```

### 案例2：不同核函数的核SVM比较

```python
from sksurv.svm import FastKernelSurvivalSVM
from sklearn.model_selection import train_test_split
from sksurv.metrics import concordance_index_ipcw

# 划分数据
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 标准化
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 比较不同核函数
kernels = ['linear', 'poly', 'rbf', 'sigmoid']
results = {}

for kernel in kernels:
    # 拟合模型
    svm = FastKernelSurvivalSVM(kernel=kernel, alpha=1.0, random_state=42)
    svm.fit(X_train_scaled, y_train)

    # 预测
    risk_scores = svm.predict(X_test_scaled)

    # 评估
    c_index = concordance_index_ipcw(y_train, y_test, risk_scores)[0]
    results[kernel] = c_index

    print(f"{kernel:10s}: C指数 = {c_index:.3f}")

# 最优核函数
best_kernel = max(results, key=results.get)
print(f"\n最优核函数: {best_kernel} (C指数 = {results[best_kernel]:.3f})")
```

### 案例3：含超参数调优的完整流水线

```python
from sksurv.svm import FastKernelSurvivalSVM
from sklearn.model_selection import GridSearchCV, train_test_split
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sksurv.metrics import as_concordance_index_ipcw_scorer

# 划分数据
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 创建流水线
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', FastKernelSurvivalSVM(kernel='rbf'))
])

# 定义参数网格
param_grid = {
    'svm__alpha': [0.1, 1.0, 10.0],
    'svm__gamma': ['scale', 0.01, 0.1, 1.0]
}

# 网格搜索
cv = GridSearchCV(
    pipeline,
    param_grid,
    scoring=as_concordance_index_ipcw_scorer(),
    cv=5,
    n_jobs=-1,
    verbose=1
)
cv.fit(X_train, y_train)

# 最优模型
best_model = cv.best_estimator_
print(f"最佳参数: {cv.best_params_}")
print(f"最佳交叉验证C指数: {cv.best_score_:.3f}")

# 测试集评估
risk_scores = best_model.predict(X_test)
c_index = concordance_index_ipcw(y_train, y_test, risk_scores)[0]
print(f"测试集C指数: {c_index:.3f}")
```

## 重要注意事项

### 特征缩放

**关键提示**：使用SVM前必须标准化特征！

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

### 计算复杂度

- **FastSurvivalSVM**：每次迭代O(n×p)——快速
- **FastKernelSurvivalSVM**：O(n²×p)——较慢，二次方扩展
- **NaiveSurvivalSVM**：O(n³)——大型数据集极慢

对于大型数据集（>10,000样本），优先选择：
- FastSurvivalSVM（线性）
- 梯度提升
- 随机生存森林

### SVM可能非最优的场景

- **超大数据集**：集成方法更快
- **需要生存函数**：使用随机生存森林或Cox模型
- **需要可解释性**：使用Cox模型
- **超高维数据**：使用惩罚Cox（Coxnet）或带特征选择的梯度提升

## 模型选择指南

| 模型 | 速度 | 非线性能力 | 可扩展性 | 可解释性 |
|-------|-------|---------------|-------------|------------------|
| FastSurvivalSVM | 快 | 无 | 高 | 中等 |
| FastKernelSurvivalSVM | 中等 | 有 | 中等 | 低 |
| HingeLossSurvivalSVM | 快 | 无 | 高 | 中等 |
| NaiveSurvivalSVM | 慢 | 无 | 低 | 中等 |

**通用建议：**
- 基线模型首选**FastSurvivalSVM**
- 预期非线性时尝试**FastKernelSurvivalSVM+RBF核**
- 使用网格搜索调整alpha和gamma
- 始终标准化特征
- 与随机生存森林和梯度提升进行对比
