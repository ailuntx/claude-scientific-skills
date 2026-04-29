# 生存分析集成模型

## 随机生存森林

### 概述

随机生存森林将随机森林算法扩展到包含删失数据的生存分析中。该方法通过在自助采样样本上构建多棵决策树并聚合预测结果来实现。

### 工作原理

1. **自助采样**：每棵树基于训练数据的不同自助采样样本构建
2. **特征随机性**：在每个节点仅考虑随机特征子集进行分裂
3. **生存函数估计**：在终端节点使用Kaplan-Meier和Nelson-Aalen估计器计算生存函数
4. **集成聚合**：最终预测通过平均所有树的生存函数获得

### 适用场景

- 特征与生存之间存在复杂非线性关系
- 无需假设函数形式
- 需要最小调参的稳健预测
- 需要特征重要性评估
- 具备足够样本量（通常 n > 100）

### 关键参数

- `n_estimators`：树的数量（默认：100）
  - 树越多预测越稳定但速度越慢
  - 典型范围：100-1000

- `max_depth`：树的最大深度
  - 控制树复杂度
  - None = 节点持续分裂直至纯净或达到min_samples_split

- `min_samples_split`：节点分裂最小样本数（默认：6）
  - 值越大正则化越强

- `min_samples_leaf`：叶节点最小样本数（默认：3）
  - 防止对小群体过拟合

- `max_features`：每次分裂考虑的特征数
  - 'sqrt'：sqrt(n_features) - 良好默认值
  - 'log2'：log2(n_features)
  - None：所有特征

- `n_jobs`：并行任务数（-1使用所有处理器）

### 使用示例

```python
from sksurv.ensemble import RandomSurvivalForest
from sksurv.datasets import load_breast_cancer

# 加载数据
X, y = load_breast_cancer()

# 拟合随机生存森林
rsf = RandomSurvivalForest(n_estimators=1000,
                           min_samples_split=10,
                           min_samples_leaf=15,
                           max_features="sqrt",
                           n_jobs=-1,
                           random_state=42)
rsf.fit(X, y)

# 预测风险评分
risk_scores = rsf.predict(X)

# 预测生存函数
surv_funcs = rsf.predict_survival_function(X)

# 预测累积风险函数
chf_funcs = rsf.predict_cumulative_hazard_function(X)
```

### 特征重要性

**重要提示**：基于分裂不纯度的内置特征重要性对生存数据不可靠。应使用基于置换的特征重要性。

```python
from sklearn.inspection import permutation_importance
from sksurv.metrics import concordance_index_censored

# 定义评分函数
def score_survival_model(model, X, y):
    prediction = model.predict(X)
    result = concordance_index_censored(y['event'], y['time'], prediction)
    return result[0]

# 计算置换重要性
perm_importance = permutation_importance(
    rsf, X, y,
    n_repeats=10,
    random_state=42,
    scoring=score_survival_model
)

# 获取特征重要性
feature_importance = perm_importance.importances_mean
```

## 梯度提升生存分析

### 概述

梯度提升通过顺序添加弱学习器来修正先前学习器的误差构建集成模型。模型公式：**f(x) = Σ β_m g(x; θ_m)**

### 模型类型

#### GradientBoostingSurvivalAnalysis

使用回归树作为基础学习器。可捕捉复杂非线性关系。

**适用场景：**
- 需建模复杂非线性关系
- 追求高预测性能
- 有足够数据避免过拟合
- 可精细调参

#### ComponentwiseGradientBoostingSurvivalAnalysis

使用分量最小二乘作为基础学习器。生成带自动特征选择的线性模型。

**适用场景：**
- 需要可解释线性模型
- 需自动特征选择（类似Lasso）
- 处理高维数据
- 偏好稀疏模型

### 损失函数

#### Cox部分似然（默认）

保持比例风险框架，但将线性模型替换为加性集成模型。

**适用于：**
- 标准生存分析场景
- 比例风险假设合理时
- 多数使用场景

#### 加速失效时间（AFT）

假设特征以恒定因子加速或减速生存时间。损失函数：**(1/n) Σ ω_i (log y_i - f(x_i))²**

**适用于：**
- 优先选择AFT框架而非比例风险
- 需直接建模时间
- 需解释对生存时间的影响

### 正则化策略

三种主要技术防止过拟合：

1. **学习率** (`learning_rate < 1`)
   - 缩减每个基础学习器的贡献
   - 较小值需更多迭代但泛化更好
   - 典型范围：0.01 - 0.1

2. **丢弃法** (`dropout_rate > 0`)
   - 训练中随机丢弃先前学习器
   - 强制学习器更鲁棒
   - 典型范围：0.01 - 0.2

3. **子采样** (`subsample < 1`)
   - 每次迭代使用随机数据子集
   - 增加随机性并减少过拟合
   - 典型范围：0.5 - 0.9

**建议**：结合小学习率与早停法获得最佳性能。

### 关键参数

- `loss`：损失函数（'coxph'或'ipcwls'）
- `learning_rate`：缩减每棵树的贡献（默认：0.1）
- `n_estimators`：提升迭代次数（默认：100）
- `subsample`：每次迭代样本比例（默认：1.0）
- `dropout_rate`：学习器丢弃率（默认：0.0）
- `max_depth`：树的最大深度（默认：3）
- `min_samples_split`：节点分裂最小样本数（默认：2）
- `min_samples_leaf`：叶节点最小样本数（默认：1）
- `max_features`：每次分裂考虑的特征数

### 使用示例

```python
from sksurv.ensemble import GradientBoostingSurvivalAnalysis
from sklearn.model_selection import train_test_split

# 划分数据
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 拟合梯度提升模型
gbs = GradientBoostingSurvivalAnalysis(
    loss='coxph',
    learning_rate=0.05,
    n_estimators=200,
    subsample=0.8,
    dropout_rate=0.1,
    max_depth=3,
    random_state=42
)
gbs.fit(X_train, y_train)

# 预测风险评分
risk_scores = gbs.predict(X_test)

# 预测生存函数
surv_funcs = gbs.predict_survival_function(X_test)

# 预测累积风险函数
chf_funcs = gbs.predict_cumulative_hazard_function(X_test)
```

### 早停法

使用验证集防止过拟合：

```python
from sklearn.model_selection import train_test_split

# 创建训练/验证划分
X_tr, X_val, y_tr, y_val = train_test_split(X_train, y_train, test_size=0.2, random_state=42)

# 带早停法的拟合
gbs = GradientBoostingSurvivalAnalysis(
    n_estimators=1000,
    learning_rate=0.01,
    max_depth=3,
    validation_fraction=0.2,
    n_iter_no_change=10,
    random_state=42
)
gbs.fit(X_tr, y_tr)

# 实际迭代次数
print(f"实际迭代次数: {gbs.n_estimators_}")
```

### 超参数调优

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    'learning_rate': [0.01, 0.05, 0.1],
    'n_estimators': [100, 200, 300],
    'max_depth': [3, 5, 7],
    'subsample': [0.8, 1.0]
}

cv = GridSearchCV(
    GradientBoostingSurvivalAnalysis(),
    param_grid,
    scoring='concordance_index_ipcw',
    cv=5,
    n_jobs=-1
)
cv.fit(X, y)

best_model = cv.best_estimator_
```

## ComponentwiseGradientBoostingSurvivalAnalysis

### 概述

使用分量最小二乘法，生成类似Lasso的稀疏线性模型并自动选择特征。

### 适用场景

- 需要可解释线性模型
- 需自动特征选择
- 处理含大量无关特征的高维数据
- 偏好基于系数的解释

### 使用示例

```python
from sksurv.ensemble import ComponentwiseGradientBoostingSurvivalAnalysis

# 拟合分量梯度提升
cgbs = ComponentwiseGradientBoostingSurvivalAnalysis(
    loss='coxph',
    learning_rate=0.1,
    n_estimators=100
)
cgbs.fit(X, y)

# 获取选定特征及系数
coef = cgbs.coef_
selected_features = [i for i, c in enumerate(coef) if c != 0]
```

## ExtraSurvivalTrees

极端随机化生存树 - 类似随机生存森林，但在分裂选择中增加额外随机性。

### 适用场景

- 需要比随机生存森林更强的正则化
- 数据量有限
- 需要更快训练速度

### 关键区别

不再为选定特征寻找最优分裂点，而是随机选择分裂点，增加集成的多样性。

```python
from sksurv.ensemble import ExtraSurvivalTrees

est = ExtraSurvivalTrees(n_estimators=100, random_state=42)
est.fit(X, y)
```

## 模型比较

| 模型 | 复杂度 | 可解释性 | 性能 | 速度 |
|-------|-----------|------------------|-------------|-------|
| 随机生存森林 | 中等 | 低 | 高 | 中等 |
| GradientBoostingSurvivalAnalysis | 高 | 低 | 最高 | 慢 |
| ComponentwiseGradientBoostingSurvivalAnalysis | 低 | 高 | 中等 | 快 |
| ExtraSurvivalTrees | 中等 | 低 | 中高 | 快 |

**通用建议：**
- **最佳整体性能**：调优后的GradientBoostingSurvivalAnalysis
- **最佳平衡**：RandomSurvivalForest
- **最佳可解释性**：ComponentwiseGradientBoostingSurvivalAnalysis
- **最快训练**：ExtraSurvivalTrees
