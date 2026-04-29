---
name: pymc
description: 使用PyMC进行贝叶斯建模。构建分层模型、MCMC（NUTS）、变分推断、LOO/WAIC比较、后验检验，用于概率编程与推断。
license: Apache License, Version 2.0
metadata:
    skill-author: K-Dense Inc.
---

# PyMC 贝叶斯建模

## 概述

PyMC 是用于贝叶斯建模和概率编程的 Python 库。通过 PyMC 的现代 API（5.x+ 版本）构建、拟合、验证和比较贝叶斯模型，包括分层模型、MCMC 采样（NUTS）、变分推断和模型比较（LOO, WAIC）。

## 使用场景

本技能适用于：
- 构建贝叶斯模型（线性/逻辑回归、分层模型、时间序列等）
- 执行 MCMC 采样或变分推断
- 进行先验/后验预测检验
- 诊断采样问题（发散、收敛性、ESS）
- 使用信息准则（LOO, WAIC）比较多个模型
- 通过贝叶斯方法实现不确定性量化
- 处理分层/多级数据结构
- 以原则性方式处理缺失数据或测量误差

## 标准贝叶斯工作流

遵循以下工作流构建和验证贝叶斯模型：

### 1. 数据准备

```python
import pymc as pm
import arviz as az
import numpy as np

# 加载并准备数据
X = ...  # 预测变量
y = ...  # 结果变量

# 标准化预测变量以提升采样效率
X_mean = X.mean(axis=0)
X_std = X.std(axis=0)
X_scaled = (X - X_mean) / X_std
```

**关键实践：**
- 标准化连续预测变量（提升采样效率）
- 尽可能对结果变量中心化
- 显式处理缺失数据（视为参数）
- 使用 `coords` 命名维度以提高可读性

### 2. 模型构建

```python
coords = {
    'predictors': ['var1', 'var2', 'var3'],
    'obs_id': np.arange(len(y))
}

with pm.Model(coords=coords) as model:
    # 先验分布
    alpha = pm.Normal('alpha', mu=0, sigma=1)
    beta = pm.Normal('beta', mu=0, sigma=1, dims='predictors')
    sigma = pm.HalfNormal('sigma', sigma=1)

    # 线性预测器
    mu = alpha + pm.math.dot(X_scaled, beta)

    # 似然函数
    y_obs = pm.Normal('y_obs', mu=mu, sigma=sigma, observed=y, dims='obs_id')
```

**关键实践：**
- 使用弱信息性先验（避免平坦先验）
- 尺度参数使用 `HalfNormal` 或 `Exponential`
- 尽可能使用命名维度 (`dims`) 替代 `shape`
- 预测时使用 `pm.Data()` 更新值

### 3. 先验预测检验

**拟合前务必验证先验：**

```python
with model:
    prior_pred = pm.sample_prior_predictive(samples=1000, random_seed=42)

# 可视化
az.plot_ppc(prior_pred, group='prior')
```

**检查：**
- 先验预测值是否覆盖合理范围？
- 极端值是否符合领域知识？
- 若先验生成不合理数据，调整后重新检验

### 4. 模型拟合

```python
with model:
    # 可选：使用 ADVI 快速探索
    # approx = pm.fit(n=20000)

    # 完整 MCMC 推断
    idata = pm.sample(
        draws=2000,
        tune=1000,
        chains=4,
        target_accept=0.9,
        random_seed=42,
        idata_kwargs={'log_likelihood': True}  # 用于模型比较
    )
```

**关键参数：**
- `draws=2000`: 每条链的采样数
- `tune=1000`: 预热样本（丢弃）
- `chains=4`: 运行 4 条链检查收敛性
- `target_accept=0.9`: 复杂后验时调高（0.95-0.99）
- 包含 `log_likelihood=True` 用于模型比较

### 5. 诊断检查

**使用诊断脚本：**

```python
from scripts.model_diagnostics import check_diagnostics

results = check_diagnostics(idata, var_names=['alpha', 'beta', 'sigma'])
```

**检查：**
- **R-hat < 1.01**: 链已收敛
- **ESS > 400**: 有效样本量充足
- **无发散**: NUTS 采样成功
- **轨迹图**: 链应充分混合（毛虫状）

**问题处理：**
- 发散 → 提高 `target_accept=0.95`，使用非中心化参数化
- 低 ESS → 增加采样数，重新参数化降低相关性
- 高 R-hat → 延长采样，检查多模态性

### 6. 后验预测检验

**验证模型拟合：**

```python
with model:
    pm.sample_posterior_predictive(idata, extend_inferencedata=True, random_seed=42)

# 可视化
az.plot_ppc(idata)
```

**检查：**
- 后验预测是否捕捉观测数据模式？
- 是否存在系统性偏差（模型误设）？
- 拟合不佳时考虑替代模型

### 7. 结果分析

```python
# 统计摘要
print(az.summary(idata, var_names=['alpha', 'beta', 'sigma']))

# 后验分布
az.plot_posterior(idata, var_names=['alpha', 'beta', 'sigma'])

# 系数估计
az.plot_forest(idata, var_names=['beta'], combined=True)
```

### 8. 预测生成

```python
X_new = ...  # 新预测变量值
X_new_scaled = (X_new - X_mean) / X_std

with model:
    pm.set_data({'X_scaled': X_new_scaled})
    post_pred = pm.sample_posterior_predictive(
        idata.posterior,
        var_names=['y_obs'],
        random_seed=42
    )

# 提取预测区间
y_pred_mean = post_pred.posterior_predictive['y_obs'].mean(dim=['chain', 'draw'])
y_pred_hdi = az.hdi(post_pred.posterior_predictive, var_names=['y_obs'])
```

## 常用模型模式

### 线性回归

连续结果变量的线性关系：

```python
with pm.Model() as linear_model:
    alpha = pm.Normal('alpha', mu=0, sigma=10)
    beta = pm.Normal('beta', mu=0, sigma=10, shape=n_predictors)
    sigma = pm.HalfNormal('sigma', sigma=1)

    mu = alpha + pm.math.dot(X, beta)
    y = pm.Normal('y', mu=mu, sigma=sigma, observed=y_obs)
```

**模板路径：** `assets/linear_regression_template.py`

### 逻辑回归

二分类结果变量：

```python
with pm.Model() as logistic_model:
    alpha = pm.Normal('alpha', mu=0, sigma=10)
    beta = pm.Normal('beta', mu=0, sigma=10, shape=n_predictors)

    logit_p = alpha + pm.math.dot(X, beta)
    y = pm.Bernoulli('y', logit_p=logit_p, observed=y_obs)
```

### 分层模型

分组数据（使用非中心化参数化）：

```python
with pm.Model(coords={'groups': group_names}) as hierarchical_model:
    # 超先验
    mu_alpha = pm.Normal('mu_alpha', mu=0, sigma=10)
    sigma_alpha = pm.HalfNormal('sigma_alpha', sigma=1)

    # 组级别（非中心化）
    alpha_offset = pm.Normal('alpha_offset', mu=0, sigma=1, dims='groups')
    alpha = pm.Deterministic('alpha', mu_alpha + sigma_alpha * alpha_offset, dims='groups')

    # 观测级别
    mu = alpha[group_idx]
    sigma = pm.HalfNormal('sigma', sigma=1)
    y = pm.Normal('y', mu=mu, sigma=sigma, observed=y_obs)
```

**模板路径：** `assets/hierarchical_model_template.py`

**关键：** 分层模型务必使用非中心化参数化避免发散。

### 泊松回归

计数数据：

```python
with pm.Model() as poisson_model:
    alpha = pm.Normal('alpha', mu=0, sigma=10)
    beta = pm.Normal('beta', mu=0, sigma=10, shape=n_predictors)

    log_lambda = alpha + pm.math.dot(X, beta)
    y = pm.Poisson('y', mu=pm.math.exp(log_lambda), observed=y_obs)
```

过度离散计数数据使用 `NegativeBinomial`。

### 时间序列

自回归过程：

```python
with pm.Model() as ar_model:
    sigma = pm.HalfNormal('sigma', sigma=1)
    rho = pm.Normal('rho', mu=0, sigma=0.5, shape=ar_order)
    init_dist = pm.Normal.dist(mu=0, sigma=sigma)

    y = pm.AR('y', rho=rho, sigma=sigma, init_dist=init_dist, observed=y_obs)
```

## 模型比较

### 模型对比

使用 LOO 或 WAIC 比较模型：

```python
from scripts.model_comparison import compare_models, check_loo_reliability

# 拟合含对数似然的模型
models = {
    'Model1': idata1,
    'Model2': idata2,
    'Model3': idata3
}

# LOO 比较
comparison = compare_models(models, ic='loo')

# 检查可靠性
check_loo_reliability(models)
```

**解读：**
- **Δloo < 2**: 模型相似，选择更简单模型
- **2 < Δloo < 4**: 支持更优模型的弱证据
- **4 < Δloo < 10**: 中等证据
- **Δloo > 10**: 支持更优模型的强证据

**检查 Pareto-k 值：**
- k < 0.7: LOO 可靠
- k > 0.7: 考虑 WAIC 或 k 折交叉验证

### 模型平均

模型相似时平均预测结果：

```python
from scripts.model_comparison import model_averaging

averaged_pred, weights = model_averaging(models, var_name='y_obs')
```

## 分布选择指南

### 先验分布

**尺度参数** (σ, τ):
- `pm.HalfNormal('sigma', sigma=1)` - 默认选择
- `pm.Exponential('sigma', lam=1)` - 替代方案
- `pm.Gamma('sigma', alpha=2, beta=1)` - 信息性更强

**无界参数**:
- `pm.Normal('theta', mu=0, sigma=1)` - 标准化数据
- `pm.StudentT('theta', nu=3, mu=0, sigma=1)` - 抗异常值

**正数参数**:
- `pm.LogNormal('theta', mu=0, sigma=1)`
- `pm.Gamma('theta', alpha=2, beta=1)`

**概率参数**:
- `pm.Beta('p', alpha=2, beta=2)` - 弱信息性
- `pm.Uniform('p', lower=0, upper=1)` - 无信息性（慎用）

**相关矩阵**:
- `pm.LKJCorr('corr', n=n_vars, eta=2)` - eta=1 均匀分布，eta>1 偏好单位矩阵

### 似然分布

**连续结果变量**:
- `pm.Normal('y', mu=mu, sigma=sigma)` - 连续数据默认
- `pm.StudentT('y', nu=nu, mu=mu, sigma=sigma)` - 抗异常值

**计数数据**:
- `pm.Poisson('y', mu=lambda)` - 等离散计数
- `pm.NegativeBinomial('y', mu=mu, alpha=alpha)` - 过度离散计数
- `pm.ZeroInflatedPoisson('y', psi=psi, mu=mu)` - 零膨胀数据

**二分类结果变量**:
- `pm.Bernoulli('y', p=p)` 或 `pm.Bernoulli('y', logit_p=logit_p)`

**多分类结果变量**:
- `pm.Categorical('y', p=probs)`

**参考：** `references/distributions.md` 获取完整分布参考

## 采样与推断

### NUTS MCMC

默认推荐用于多数模型：

```python
idata = pm.sample(
    draws=2000,
    tune=1000,
    chains=4,
    target_accept=0.9,
    random_seed=42
)
```

**调整建议：**
- 发散 → `target_accept=0.95` 或更高
- 采样慢 → 使用 ADVI 初始化
- 离散参数 → 对离散变量使用 `pm.Metropolis()`

### 变分推断

用于快速探索或初始化：

```python
with model:
    approx = pm.fit(n=20000, method='advi')

    # 用于初始化
    start = approx.sample(return_inferencedata=False)[0]
    idata = pm.sample(start=start)
```

**权衡：**
- 比 MCMC 快得多
- 近似解（可能低估不确定性）
- 适用于大型模型或快速探索

**参考：** `references/sampling_inference.md` 获取详细采样指南

## 诊断脚本

### 综合诊断

```python
from scripts.model_diagnostics import create_diagnostic_report

create_diagnostic_report(
    idata,
    var_names=['alpha', 'beta', 'sigma'],
    output_dir='diagnostics/'
)
```

生成：
- 轨迹图
- 秩图（混合检查）
- 自相关图
- 能量图
- ESS 演化图
- 统计摘要 CSV

### 快速诊断检查

```python
from scripts.model_diagnostics import check_diagnostics

results = check_diagnostics(idata)
```

检查 R-hat、ESS、发散和树深度。

## 常见问题与解决方案

### 发散问题

**症状：** `idata.sample_stats.diverging.sum() > 0`

**解决方案：**
1. 提高 `target_accept=0.95` 或 `0.99`
2. 使用非中心化参数化（分层模型）
3. 添加更强先验约束参数
4. 检查模型误设

### 低有效样本量

**症状：** `ESS < 400`

**解决方案：**
1. 增加采样数：`draws=5000`
2. 重新参数化降低后验相关性
3. 相关预测变量回归使用 QR 分解

### 高 R-hat

**症状：** `R-hat > 1.01`

**解决方案：**
1. 延长链：`tune=2000, draws=5000`
2. 检查多模态性
3. 使用 ADVI 改进初始化

### 采样缓慢

**解决方案：**
1. 使用 ADVI 初始化
2. 降低模型复杂度
3. 增加并行化：`cores=8, chains=8`
4. 适用时使用变分推断

## 最佳实践

### 模型构建

1. **始终标准化预测变量** 提升采样效率
2. **使用弱信息性先验**（避免平坦先验）
3. **使用命名维度** (`dims`) 提高可读性
4. **分层模型采用非中心化参数化**
5. **拟合前执行先验预测检验**

### 采样

1. **运行多条链**（至少 4 条）检查收敛性
2. **基线设置 `target_accept=0.9`**（需要时调高）
3. **包含 `log_likelihood=True`** 用于模型比较
4. **设置随机种子** 保证可复现性

### 验证

1. **结果解释前检查诊断**（R-hat, ESS, 发散）
2. **后验预测检验** 验证模型
3. **适时比较多个模型**
4. **报告不确定性**（HDI 区间，非仅点估计）

### 工作流

1. 从简单模型开始，逐步增加复杂度
2. 流程：先验预测检验 → 拟合 → 诊断 → 后验预测检验
3. 根据检验结果迭代优化模型
4. 记录假设和先验选择

## 资源

本技能包含：

### 参考资料 (`references/`)

- **`distributions.md`**: 按类别（连续、离散、多元、混合、时间序列）整理的 PyMC 分布全集。用于选择先验或似然分布。

- **`sampling_inference.md`**: 采样算法（NUTS, Metropolis, SMC）、变分推断（ADVI, SVGD）及采样问题处理的详细指南。用于解决

- **`model_diagnostics.py`**：自动化诊断检查与报告生成。功能：`check_diagnostics()`用于快速检查，`create_diagnostic_report()`生成带图表的综合分析报告。

- **`model_comparison.py`**：使用LOO/WAIC的模型比较工具。功能：`compare_models()`、`check_loo_reliability()`、`model_averaging()`。

### 模板 (`assets/`)

- **`linear_regression_template.py`**：完整贝叶斯线性回归模板，包含全流程（数据预处理、先验检查、拟合、诊断、预测）。

- **`hierarchical_model_template.py`**：完整分层/多级模型模板，包含非中心化参数化与组别分析。

## 速查指南

### 模型构建
```python
with pm.Model(coords={'var': names}) as model:
    # 先验分布
    param = pm.Normal('param', mu=0, sigma=1, dims='var')
    # 似然函数
    y = pm.Normal('y', mu=..., sigma=..., observed=data)
```

### 采样
```python
idata = pm.sample(draws=2000, tune=1000, chains=4, target_accept=0.9)
```

### 诊断
```python
from scripts.model_diagnostics import check_diagnostics
check_diagnostics(idata)
```

### 模型比较
```python
from scripts.model_comparison import compare_models
compare_models({'m1': idata1, 'm2': idata2}, ic='loo')
```

### 预测
```python
with model:
    pm.set_data({'X': X_new})
    pred = pm.sample_posterior_predictive(idata.posterior)
```

## 补充说明

- PyMC集成ArviZ实现可视化与诊断
- 使用`pm.model_to_graphviz(model)`可视化模型结构
- 通过`idata.to_netcdf('results.nc')`保存结果
- 通过`az.from_netcdf('results.nc')`加载结果
- 对于超大模型，建议使用小批量ADVI或数据子采样
