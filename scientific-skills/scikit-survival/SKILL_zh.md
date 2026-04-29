```markdown
---
name: scikit-survival
description: 基于scikit-survival的Python生存分析和事件时间建模综合工具包。适用于处理删失生存数据、进行事件时间分析、拟合Cox模型、随机生存森林、梯度提升模型或生存支持向量机，使用一致性指数或Brier评分评估生存预测，处理竞争风险，或实现任何使用scikit-survival库的生存分析工作流。
license: GPL-3.0 license
metadata:
    skill-author: K-Dense Inc.
---

# scikit-survival：Python中的生存分析

## 概述

scikit-survival是基于scikit-learn构建的Python生存分析库。它提供专门工具处理事件时间分析，解决删失数据（部分观测值仅部分已知）这一独特挑战。

生存分析旨在建立协变量与事件发生时间之间的关联，同时考虑删失记录（特别是来自研究期间未经历事件的参与者的右删失数据）。

## 适用场景

在以下场景使用本技能：
- 执行生存分析或事件时间建模
- 处理删失数据（右删失、左删失或区间删失）
- 拟合Cox比例风险模型（标准或惩罚化）
- 构建集成生存模型（随机生存森林、梯度提升）
- 训练生存支持向量机
- 评估生存模型性能（一致性指数、Brier评分、时间依赖性AUC）
- 估计Kaplan-Meier或Nelson-Aalen曲线
- 分析竞争风险
- 预处理生存数据或处理生存数据集缺失值
- 使用scikit-survival库进行任何分析

## 核心功能

### 1. 模型类型与选择

scikit-survival提供多种模型系列，分别适用于不同场景：

#### Cox比例风险模型
**适用场景**：具有可解释系数的标准生存分析
- `CoxPHSurvivalAnalysis`：基础Cox模型
- `CoxnetSurvivalAnalysis`：针对高维数据的弹性网络惩罚Cox模型
- `IPCRidge`：加速失效时间模型的岭回归

**参考**：`references/cox-models.md`获取Cox模型、正则化和解释的详细指南

#### 集成方法
**适用场景**：处理复杂非线性关系的高预测性能
- `RandomSurvivalForest`：鲁棒的非参数集成方法
- `GradientBoostingSurvivalAnalysis`：基于树的提升方法实现最佳性能
- `ComponentwiseGradientBoostingSurvivalAnalysis`：带特征选择的线性提升
- `ExtraSurvivalTrees`：额外正则化的极端随机树

**参考**：`references/ensemble-models.md`获取集成方法、超参数调优及模型选择的完整指南

#### 生存支持向量机
**适用场景**：中等规模数据集的边界学习
- `FastSurvivalSVM`：速度优化的线性SVM
- `FastKernelSurvivalSVM`：处理非线性关系的核SVM
- `HingeLossSurvivalSVM`：使用铰链损失的SVM
- `ClinicalKernelTransform`：针对临床+分子数据的专用核函数

**参考**：`references/svm-models.md`获取SVM指南、核选择及超参数调优详情

#### 模型选择决策树

```
开始
├─ 高维数据 (p > n)?
│  ├─ 是 → CoxnetSurvivalAnalysis (弹性网络)
│  └─ 否 → 继续
│
├─ 需要可解释系数？
│  ├─ 是 → CoxPHSurvivalAnalysis 或 ComponentwiseGradientBoostingSurvivalAnalysis
│  └─ 否 → 继续
│
├─ 预期存在复杂非线性关系？
│  ├─ 是
│  │  ├─ 大数据集 (n > 1000) → GradientBoostingSurvivalAnalysis
│  │  ├─ 中等数据集 → RandomSurvivalForest 或 FastKernelSurvivalSVM
│  │  └─ 小数据集 → RandomSurvivalForest
│  └─ 否 → CoxPHSurvivalAnalysis 或 FastSurvivalSVM
│
└─ 追求极致性能 → 尝试多种模型并比较
```

### 2. 数据准备与预处理

建模前需正确准备生存数据：

#### 创建生存结果
```python
from sksurv.util import Surv

# 从独立数组创建
y = Surv.from_arrays(event=事件数组, time=时间数组)

# 从DataFrame创建
y = Surv.from_dataframe('事件列', '时间列', 数据框)
```

#### 基本预处理步骤
1. **处理缺失值**：特征插补策略
2. **编码分类变量**：独热编码或标签编码
3. **标准化特征**：对SVM和正则化Cox模型至关重要
4. **验证数据质量**：检查负时间值、每个特征足够的事件数
5. **训练-测试集划分**：保持不同分组的删失率相似

**参考**：`references/data-handling.md`获取完整预处理流程、数据验证和最佳实践

### 3. 模型评估

正确评估对生存模型至关重要。使用考虑删失的适当指标：

#### 一致性指数 (C-index)
排序/区分的主要指标：
- **Harrell's C-index**：适用于低删失率 (<40%)
- **Uno's C-index**：适用于中高删失率 (>40%) —— 更鲁棒

```python
from sksurv.metrics import concordance_index_censored, concordance_index_ipcw

# Harrell's C-index
c_harrell = concordance_index_censored(y_test['事件'], y_test['时间'], 风险分数)[0]

# Uno's C-index (推荐)
c_uno = concordance_index_ipcw(y_train, y_test, 风险分数)[0]
```

#### 时间依赖性AUC
评估特定时间点的区分能力：

```python
from sksurv.metrics import cumulative_dynamic_auc

times = [365, 730, 1095]  # 1年、2年、3年
auc, mean_auc = cumulative_dynamic_auc(y_train, y_test, 风险分数, times)
```

#### Brier评分
同时评估区分能力和校准能力：

```python
from sksurv.metrics import integrated_brier_score

ibs = integrated_brier_score(y_train, y_test, 生存函数, times)
```

**参考**：`references/evaluation-metrics.md`获取完整评估指南、指标选择及交叉验证评分器使用

### 4. 竞争风险分析

处理存在多种互斥事件类型的情况：

```python
from sksurv.nonparametric import cumulative_incidence_competing_risks

# 估计每种事件类型的累积发生率
时间点, 事件1累积发生率, 事件2累积发生率 = cumulative_incidence_competing_risks(y)
```

**使用竞争风险分析当**：
- 存在多种互斥事件类型（如不同原因导致的死亡）
- 一种事件的发生会阻止其他事件
- 需要特定事件类型的概率估计

**参考**：`references/competing-risks.md`获取竞争风险方法、特定原因风险模型及解释详情

### 5. 非参数估计

无需参数假设估计生存函数：

#### Kaplan-Meier估计器
```python
from sksurv.nonparametric import kaplan_meier_estimator

时间, 生存概率 = kaplan_meier_estimator(y['事件'], y['时间'])
```

#### Nelson-Aalen估计器
```python
from sksurv.nonparametric import nelson_aalen_estimator

时间, 累积风险 = nelson_aalen_estimator(y['事件'], y['时间'])
```

## 典型工作流

### 工作流1：标准生存分析

```python
from sksurv.datasets import load_breast_cancer
from sksurv.linear_model import CoxPHSurvivalAnalysis
from sksurv.metrics import concordance_index_ipcw
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# 1. 加载并准备数据
X, y = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 2. 预处理
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 3. 拟合模型
estimator = CoxPHSurvivalAnalysis()
estimator.fit(X_train_scaled, y_train)

# 4. 预测
风险分数 = estimator.predict(X_test_scaled)

# 5. 评估
c_index = concordance_index_ipcw(y_train, y_test, 风险分数)[0]
print(f"C-index: {c_index:.3f}")
```

### 工作流2：高维数据特征选择

```python
from sksurv.linear_model import CoxnetSurvivalAnalysis
from sklearn.model_selection import GridSearchCV
from sksurv.metrics import as_concordance_index_ipcw_scorer

# 1. 使用惩罚Cox进行特征选择
estimator = CoxnetSurvivalAnalysis(l1_ratio=0.9)  # 类Lasso

# 2. 通过交叉验证调整正则化
param_grid = {'alpha_min_ratio': [0.01, 0.001]}
cv = GridSearchCV(estimator, param_grid,
                  scoring=as_concordance_index_ipcw_scorer(), cv=5)
cv.fit(X, y)

# 3. 识别选定特征
best_model = cv.best_estimator_
selected_features = np.where(best_model.coef_ != 0)[0]
```

### 工作流3：追求极致性能的集成方法

```python
from sksurv.ensemble import GradientBoostingSurvivalAnalysis
from sklearn.model_selection import GridSearchCV

# 1. 定义参数网格
param_grid = {
    'learning_rate': [0.01, 0.05, 0.1],
    'n_estimators': [100, 200, 300],
    'max_depth': [3, 5, 7]
}

# 2. 网格搜索
gbs = GradientBoostingSurvivalAnalysis()
cv = GridSearchCV(gbs, param_grid, cv=5,
                  scoring=as_concordance_index_ipcw_scorer(), n_jobs=-1)
cv.fit(X_train, y_train)

# 3. 评估最佳模型
best_model = cv.best_estimator_
风险分数 = best_model.predict(X_test)
c_index = concordance_index_ipcw(y_train, y_test, 风险分数)[0]
```

### 工作流4：综合模型比较

```python
from sksurv.linear_model import CoxPHSurvivalAnalysis
from sksurv.ensemble import RandomSurvivalForest, GradientBoostingSurvivalAnalysis
from sksurv.svm import FastSurvivalSVM
from sksurv.metrics import concordance_index_ipcw, integrated_brier_score

# 定义模型
models = {
    'Cox': CoxPHSurvivalAnalysis(),
    'RSF': RandomSurvivalForest(n_estimators=100, random_state=42),
    'GBS': GradientBoostingSurvivalAnalysis(random_state=42),
    'SVM': FastSurvivalSVM(random_state=42)
}

# 评估每个模型
results = {}
for name, model in models.items():
    model.fit(X_train_scaled, y_train)
    风险分数 = model.predict(X_test_scaled)
    c_index = concordance_index_ipcw(y_train, y_test, 风险分数)[0]
    results[name] = c_index
    print(f"{name}: C-index = {c_index:.3f}")

# 选择最佳模型
best_model_name = max(results, key=results.get)
print(f"\n最佳模型: {best_model_name}")
```

## 与scikit-learn集成

scikit-survival完全兼容scikit-learn生态系统：

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import cross_val_score, GridSearchCV

# 使用管道
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', CoxPHSurvivalAnalysis())
])

# 使用交叉验证
scores = cross_val_score(pipeline, X, y, cv=5,
                         scoring=as_concordance_index_ipcw_scorer())

# 使用网格搜索
param_grid = {'model__alpha': [0.1, 1.0, 10.0]}
cv = GridSearchCV(pipeline, param_grid, cv=5)
cv.fit(X, y)
```

## 最佳实践

1. **始终标准化特征**：适用于SVM和正则化Cox模型
2. **使用Uno's C-index**：当删失率>40%时替代Harrell's
3. **报告多维度量指标**：C-index、综合Brier评分、时间依赖性AUC
4. **检查比例风险假设**：针对Cox模型
5. **使用交叉验证**：配合适当评分器进行超参数调优
6. **建模前验证数据质量**：检查负时间值、每个特征足够的事件数
7. **比较多种模型类型**：寻找最佳性能
8. **使用置换重要性**：针对随机生存森林（非内置重要性）
9. **考虑竞争风险**：当存在多种事件类型时
10. **记录删失机制和比率**：在分析中明确说明

## 常见陷阱规避

1. **高删失率下使用Harrell's C-index** → 改用Uno's C-index
2. **未对SVM特征标准化** → 始终标准化
3. **忘记向concordance_index_ipcw传递y_train** → IPCW计算必需
4. **将竞争事件视为删失** → 使用竞争风险方法
5. **未检查每个特征足够的事件数** → 经验法则：每个特征≥10个事件
6. **使用RSF内置特征重要性** → 改用置换重要性
7. **忽略比例风险假设** → 验证或使用替代模型
8. **交叉验证未使用适当评分器** → 使用as_concordance_index_ipcw_scorer()

## 参考文件

本技能包含针对特定主题的详细参考文件：

- **`references/cox-models.md`**：Cox比例风险模型完整指南，惩罚化Cox (CoxNet)、IPCRidge、正则化策略及解释
- **`references/ensemble-models.md`**：随机生存森林、梯度提升、超参数调优、特征重要性和模型选择
- **`references/evaluation-metrics.md`**：一致性指数 (Harrell's vs Uno's)、时间依赖性AUC、Brier评分、完整评估流程
- **`references/data-handling.md`**：数据加载、预处理流程、缺失值处理、特征编码、验证检查
- **`references/svm-models.md`**：生存支持向量机、核选择、临床核变换、超参数调优
- **`references/competing-risks.md`**：竞争风险分析、累积发生率函数、特定原因风险模型

需要特定任务详细信息时请加载这些参考文件。

## 附加资源

- **官方文档**：https://scikit-survival.readthedocs.io/
- **GitHub仓库**：https://github.com/sebp/scikit-survival
- **内置数据集**：使用`sksurv.datasets`获取练习数据集 (GBSG2, WHAS500, veterans肺癌等)
- **API参考**：完整类和函数列表：https://scikit-survival.readthedocs.io/en/stable/api/index.html

## 速查表：关键导入

```python
# 模型
from sksurv.linear_model import CoxPHSurvivalAnalysis, CoxnetSurvivalAnalysis, IPCRidge
from sksurv.ensemble import RandomSurvivalForest, GradientBoostingSurvivalAnalysis
from sksurv.svm import FastSurvivalSVM, FastKernelSurvivalSVM
from sksurv.tree import SurvivalTree

# 评估指标
from sksurv.metrics import (
    concordance_index_censored,
    concordance_index_ipcw,
    cumulative_dynamic_auc,
    brier_score,
    integrated_brier_score,
    as_concordance_index_ipcw_scorer,
    as_integrated_brier_score_scorer
)

# 非参数估计
from sksurv.nonparametric import (
    kaplan_meier_estimator,
    nelson_aalen_estimator,
    cumulative_incidence_competing_risks
)

# 数据处理
from sksurv.util import Surv
from sksurv.preprocessing import OneHotEncoder, encode_categorical
from sksurv.datasets import load_gbsg2, load_breast_cancer, load_veterans_lung_cancer

# 核函数
from sksurv.kernels import ClinicalKernelTransform
```
