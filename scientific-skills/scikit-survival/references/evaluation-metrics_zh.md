# 生存模型评估指标

## 概述

评估生存模型需要专门处理删失数据的指标。scikit-survival 提供三大类评估指标：
1. 一致性指数 (C-index)
2. 时间依赖性ROC与AUC
3. Brier分数

## 一致性指数 (C-index)

### 测量内容

一致性指数衡量预测风险评分与观测事件时间之间的秩相关性。它表示对于随机样本对，模型正确排序其生存时间的概率。

**范围**：0 到 1
- 0.5 = 随机预测
- 1.0 = 完全一致
- 典型良好性能：0.7-0.8

### 两种实现

#### Harrell's C-index (concordance_index_censored)

传统估计器，更简单但存在局限性。

**适用场景**：
- 低删失率 (< 40%)
- 开发过程中快速评估
- 相同数据集上的模型比较

**局限性**：
- 高删失率下偏差增大
- 约49%删失率时开始高估性能

```python
from sksurv.metrics import concordance_index_censored

# 计算Harrell's C-index
result = concordance_index_censored(y_test['event'], y_test['time'], risk_scores)
c_index = result[0]
print(f"Harrell's C-index: {c_index:.3f}")
```

#### Uno's C-index (concordance_index_ipcw)

逆概率删失加权(IPCW)估计器，修正删失偏差。

**适用场景**：
- 中高删失率 (> 40%)
- 需要无偏估计
- 跨数据集模型比较
- 发表研究成果（更鲁棒）

**优势**：
- 高删失率下保持稳定
- 估计更可靠
- 偏差更小

```python
from sksurv.metrics import concordance_index_ipcw

# 计算Uno's C-index
# 需要训练数据进行IPCW计算
c_index, concordant, discordant, tied_risk = concordance_index_ipcw(
    y_train, y_test, risk_scores
)
print(f"Uno's C-index: {c_index:.3f}")
```

### 选择指南

**使用Uno's C-index当**：
- 删失率 > 40%
- 需要最高精度估计
- 跨研究比较模型
- 发表研究

**使用Harrell's C-index当**：
- 低删失率
- 开发过程中快速模型比较
- 计算效率至关重要

### 示例比较

```python
from sksurv.metrics import concordance_index_censored, concordance_index_ipcw

# Harrell's C-index
harrell = concordance_index_censored(y_test['event'], y_test['time'], risk_scores)[0]

# Uno's C-index
uno = concordance_index_ipcw(y_train, y_test, risk_scores)[0]

print(f"Harrell's C-index: {harrell:.3f}")
print(f"Uno's C-index: {uno:.3f}")
```

## 时间依赖性ROC与AUC

### 测量内容

时间依赖性AUC评估模型在特定时间点的区分能力。它区分在时间*t*前发生事件的受试者与未发生者。

**回答的问题**："模型预测谁将在时间t前发生事件的能力如何？"

### 适用场景

- 预测特定时间窗内事件发生
- 特定时间点临床决策（如5年生存率）
- 需评估不同时间范围的性能
- 需要区分度和时间信息

### 核心函数：cumulative_dynamic_auc

```python
from sksurv.metrics import cumulative_dynamic_auc

# 定义评估时间点
times = [365, 730, 1095, 1460, 1825]  # 1,2,3,4,5年

# 计算时间依赖性AUC
auc, mean_auc = cumulative_dynamic_auc(
    y_train, y_test, risk_scores, times
)

# 绘制AUC随时间变化
import matplotlib.pyplot as plt
plt.plot(times, auc, marker='o')
plt.xlabel('时间（天）')
plt.ylabel('时间依赖性AUC')
plt.title('模型区分度随时间变化')
plt.show()

print(f"平均AUC: {mean_auc:.3f}")
```

### 解读

- **时间t的AUC**：模型将时间t前发生事件的受试者正确排在未发生者之上的概率
- **AUC随时间变化**：表明模型性能随预测时间范围变化
- **平均AUC**：所有时间点区分度的综合指标

### 示例：模型比较

```python
# 比较两个模型
auc1, mean_auc1 = cumulative_dynamic_auc(y_train, y_test, risk_scores1, times)
auc2, mean_auc2 = cumulative_dynamic_auc(y_train, y_test, risk_scores2, times)

plt.plot(times, auc1, marker='o', label='模型1')
plt.plot(times, auc2, marker='s', label='模型2')
plt.xlabel('时间（天）')
plt.ylabel('时间依赖性AUC')
plt.legend()
plt.show()
```

## Brier分数

### 测量内容

Brier分数将均方误差扩展到含删失的生存数据。同时测量区分度（排序）和校准（预测概率准确性）。

**公式**：**(1/n) Σ (S(t|x_i) - I(T_i > t))²**

其中S(t|x_i)是受试者i在时间t的预测生存概率。

**范围**：0 到 1
- 0 = 完美预测
- 值越低越好
- 典型良好性能：< 0.2

### 适用场景

- 需要校准评估（不仅是排序）
- 需评估预测概率而非风险评分
- 比较输出生存函数的模型
- 需要概率估计的临床应用

### 核心函数

#### brier_score：单时间点

```python
from sksurv.metrics import brier_score

# 在特定时间点计算Brier分数
time_point = 1825  # 5年
surv_probs = model.predict_survival_function(X_test)
# 提取每个受试者在time_point的生存概率
surv_at_t = [fn(time_point) for fn in surv_probs]

bs = brier_score(y_train, y_test, surv_at_t, time_point)[1]
print(f"{time_point}天的Brier分数: {bs:.3f}")
```

#### integrated_brier_score：时间综合指标

```python
from sksurv.metrics import integrated_brier_score

# 计算综合Brier分数
times = [365, 730, 1095, 1460, 1825]
surv_probs = model.predict_survival_function(X_test)

ibs = integrated_brier_score(y_train, y_test, surv_probs, times)
print(f"综合Brier分数: {ibs:.3f}")
```

### 解读

- **时间t的Brier分数**：预测与真实生存概率在时间t的期望平方差
- **综合Brier分数**：各时间点Brier分数的加权平均
- **值越低 = 预测越好**

### 与零模型比较

始终与基线模型比较（如Kaplan-Meier）：

```python
from sksurv.nonparametric import kaplan_meier_estimator

# 计算Kaplan-Meier基线
time_km, surv_km = kaplan_meier_estimator(y_train['event'], y_train['time'])

# 为测试集生成KM预测
surv_km_test = [surv_km[time_km <= time_point][-1] if any(time_km <= time_point) else 1.0
                for _ in range(len(X_test))]

bs_km = brier_score(y_train, y_test, surv_km_test, time_point)[1]
bs_model = brier_score(y_train, y_test, surv_at_t, time_point)[1]

print(f"Kaplan-Meier Brier分数: {bs_km:.3f}")
print(f"模型Brier分数: {bs_model:.3f}")
print(f"提升率: {(bs_km - bs_model) / bs_km * 100:.1f}%")
```

## 交叉验证中的指标应用

### 一致性指数评分器

```python
from sklearn.model_selection import cross_val_score
from sksurv.metrics import as_concordance_index_ipcw_scorer

# 创建评分器
scorer = as_concordance_index_ipcw_scorer()

# 执行交叉验证
scores = cross_val_score(model, X, y, cv=5, scoring=scorer)
print(f"平均C-index: {scores.mean():.3f} (±{scores.std():.3f})")
```

### 综合Brier分数评分器

```python
from sksurv.metrics import as_integrated_brier_score_scorer

# 定义评估时间点
times = np.percentile(y['time'][y['event']], [25, 50, 75])

# 创建评分器
scorer = as_integrated_brier_score_scorer(times)

# 执行交叉验证
scores = cross_val_score(model, X, y, cv=5, scoring=scorer)
print(f"平均IBS: {scores.mean():.3f} (±{scores.std():.3f})")
```

## 使用GridSearchCV进行模型选择

```python
from sklearn.model_selection import GridSearchCV
from sksurv.ensemble import RandomSurvivalForest
from sksurv.metrics import as_concordance_index_ipcw_scorer

# 定义参数网格
param_grid = {
    'n_estimators': [100, 200, 300],
    'min_samples_split': [10, 20, 30],
    'max_depth': [None, 10, 20]
}

# 创建评分器
scorer = as_concordance_index_ipcw_scorer()

# 执行网格搜索
cv = GridSearchCV(
    RandomSurvivalForest(random_state=42),
    param_grid,
    scoring=scorer,
    cv=5,
    n_jobs=-1
)
cv.fit(X, y)

print(f"最优参数: {cv.best_params_}")
print(f"最优C-index: {cv.best_score_:.3f}")
```

## 综合模型评估

### 推荐评估流程

```python
from sksurv.metrics import (
    concordance_index_censored,
    concordance_index_ipcw,
    cumulative_dynamic_auc,
    integrated_brier_score
)

def evaluate_survival_model(model, X_train, X_test, y_train, y_test):
    """生存模型综合评估"""

    # 获取预测
    risk_scores = model.predict(X_test)
    surv_funcs = model.predict_survival_function(X_test)

    # 1. 一致性指数（双版本）
    c_harrell = concordance_index_censored(y_test['event'], y_test['time'], risk_scores)[0]
    c_uno = concordance_index_ipcw(y_train, y_test, risk_scores)[0]

    # 2. 时间依赖性AUC
    times = np.percentile(y_test['time'][y_test['event']], [25, 50, 75])
    auc, mean_auc = cumulative_dynamic_auc(y_train, y_test, risk_scores, times)

    # 3. 综合Brier分数
    ibs = integrated_brier_score(y_train, y_test, surv_funcs, times)

    # 输出结果
    print("=" * 50)
    print("模型评估结果")
    print("=" * 50)
    print(f"Harrell's C-index:  {c_harrell:.3f}")
    print(f"Uno's C-index:      {c_uno:.3f}")
    print(f"平均AUC:           {mean_auc:.3f}")
    print(f"综合Brier分数:   {ibs:.3f}")
    print("=" * 50)

    return {
        'c_harrell': c_harrell,
        'c_uno': c_uno,
        'mean_auc': mean_auc,
        'ibs': ibs,
        'time_auc': dict(zip(times, auc))
    }

# 使用评估函数
results = evaluate_survival_model(model, X_train, X_test, y_train, y_test)
```

## 选择合适指标

### 决策指南

**使用C-index(Uno's)当**：
- 主要目标是排序/区分度
- 不需要校准概率
- 标准生存分析场景
- 最常见选择

**使用时间依赖性AUC当**：
- 需要特定时间点的区分度
- 特定时间范围的临床决策
- 需了解性能随时间变化

**使用Brier分数当**：
- 需要校准的概率估计
- 区分度与校准同等重要
- 需要概率的临床决策
- 需要全面评估

**最佳实践**：报告多指标进行全面评估。至少报告：
- Uno's C-index（区分度）
- 综合Brier分数（区分度+校准）
- 临床相关时间点的时间依赖性AUC
