# PyMC工作流程与常见模式

本参考提供了在PyMC中构建、验证和分析贝叶斯模型的标准工作流程与模式。

## 标准贝叶斯工作流程

### 完整工作流模板

```python
import pymc as pm
import arviz as az
import numpy as np
import matplotlib.pyplot as plt

# 1. 准备数据
# ===============
X = ...  # 预测变量
y = ...  # 观测结果

# 标准化预测变量以获得更好的采样效果
X_scaled = (X - X.mean(axis=0)) / X.std(axis=0)

# 2. 构建模型
# ==============
with pm.Model() as model:
    # 为命名维度定义坐标
    coords = {
        'predictors': ['var1', 'var2', 'var3'],
        'obs_id': np.arange(len(y))
    }

    # 先验分布
    alpha = pm.Normal('alpha', mu=0, sigma=1)
    beta = pm.Normal('beta', mu=0, sigma=1, dims='predictors')
    sigma = pm.HalfNormal('sigma', sigma=1)

    # 线性预测器
    mu = alpha + pm.math.dot(X_scaled, beta)

    # 似然函数
    y_obs = pm.Normal('y_obs', mu=mu, sigma=sigma, observed=y, dims='obs_id')

# 3. 先验预测检验
# ==========================
with model:
    prior_pred = pm.sample_prior_predictive(samples=1000, random_seed=42)

# 可视化先验预测
az.plot_ppc(prior_pred, group='prior', num_pp_samples=100)
plt.title('先验预测检验')
plt.show()

# 4. 拟合模型
# ============
with model:
    # 快速变分推断探索（可选）
    approx = pm.fit(n=20000, random_seed=42)

    # 完整MCMC推断
    idata = pm.sample(
        draws=2000,
        tune=1000,
        chains=4,
        target_accept=0.9,
        random_seed=42,
        idata_kwargs={'log_likelihood': True}  # 用于模型比较
    )

# 5. 诊断检查
# ====================
# 汇总统计量
print(az.summary(idata, var_names=['alpha', 'beta', 'sigma']))

# R-hat和ESS
summary = az.summary(idata)
if (summary['r_hat'] > 1.01).any():
    print("警告：部分R-hat值>1.01，链可能未收敛")

if (summary['ess_bulk'] < 400).any():
    print("警告：部分ESS值<400，建议增加样本量")

# 检查发散
divergences = idata.sample_stats.diverging.sum().item()
print(f"发散次数：{divergences}")

# 轨迹图
az.plot_trace(idata, var_names=['alpha', 'beta', 'sigma'])
plt.tight_layout()
plt.show()

# 6. 后验预测检验
# ==============================
with model:
    pm.sample_posterior_predictive(idata, extend_inferencedata=True, random_seed=42)

# 可视化拟合效果
az.plot_ppc(idata, num_pp_samples=100)
plt.title('后验预测检验')
plt.show()

# 7. 结果分析
# ==================
# 后验分布
az.plot_posterior(idata, var_names=['alpha', 'beta', 'sigma'])
plt.tight_layout()
plt.show()

# 系数森林图
az.plot_forest(idata, var_names=['beta'], combined=True)
plt.title('系数估计')
plt.show()

# 8. 新数据预测
# ============================
X_new = ...  # 新预测变量值
X_new_scaled = (X_new - X.mean(axis=0)) / X.std(axis=0)

with model:
    # 更新数据
    pm.set_data({'X': X_new_scaled})

    # 采样预测
    post_pred = pm.sample_posterior_predictive(
        idata.posterior,
        var_names=['y_obs'],
        random_seed=42
    )

# 预测区间
y_pred_mean = post_pred.posterior_predictive['y_obs'].mean(dim=['chain', 'draw'])
y_pred_hdi = az.hdi(post_pred.posterior_predictive, var_names=['y_obs'])

# 9. 保存结果
# ===============
idata.to_netcdf('model_results.nc')  # 保存供后续使用
```

## 模型构建模式

### 线性回归

```python
with pm.Model() as linear_model:
    # 先验
    alpha = pm.Normal('alpha', mu=0, sigma=10)
    beta = pm.Normal('beta', mu=0, sigma=10, shape=n_predictors)
    sigma = pm.HalfNormal('sigma', sigma=1)

    # 线性预测器
    mu = alpha + pm.math.dot(X, beta)

    # 似然函数
    y = pm.Normal('y', mu=mu, sigma=sigma, observed=y_obs)
```

### 逻辑回归

```python
with pm.Model() as logistic_model:
    # 先验
    alpha = pm.Normal('alpha', mu=0, sigma=10)
    beta = pm.Normal('beta', mu=0, sigma=10, shape=n_predictors)

    # 线性预测器
    logit_p = alpha + pm.math.dot(X, beta)

    # 似然函数
    y = pm.Bernoulli('y', logit_p=logit_p, observed=y_obs)
```

### 分层/多级模型

```python
with pm.Model(coords={'group': group_names, 'obs': np.arange(n_obs)}) as hierarchical_model:
    # 超先验
    mu_alpha = pm.Normal('mu_alpha', mu=0, sigma=10)
    sigma_alpha = pm.HalfNormal('sigma_alpha', sigma=1)

    mu_beta = pm.Normal('mu_beta', mu=0, sigma=10)
    sigma_beta = pm.HalfNormal('sigma_beta', sigma=1)

    # 组级参数（非中心化）
    alpha_offset = pm.Normal('alpha_offset', mu=0, sigma=1, dims='group')
    alpha = pm.Deterministic('alpha', mu_alpha + sigma_alpha * alpha_offset, dims='group')

    beta_offset = pm.Normal('beta_offset', mu=0, sigma=1, dims='group')
    beta = pm.Deterministic('beta', mu_beta + sigma_beta * beta_offset, dims='group')

    # 观测级模型
    mu = alpha[group_idx] + beta[group_idx] * X

    sigma = pm.HalfNormal('sigma', sigma=1)
    y = pm.Normal('y', mu=mu, sigma=sigma, observed=y_obs, dims='obs')
```

### 泊松回归（计数数据）

```python
with pm.Model() as poisson_model:
    # 先验
    alpha = pm.Normal('alpha', mu=0, sigma=10)
    beta = pm.Normal('beta', mu=0, sigma=10, shape=n_predictors)

    # 对数尺度线性预测器
    log_lambda = alpha + pm.math.dot(X, beta)

    # 似然函数
    y = pm.Poisson('y', mu=pm.math.exp(log_lambda), observed=y_obs)
```

### 时间序列（自回归）

```python
with pm.Model() as ar_model:
    # 创新标准差
    sigma = pm.HalfNormal('sigma', sigma=1)

    # AR系数
    rho = pm.Normal('rho', mu=0, sigma=0.5, shape=ar_order)

    # 初始分布
    init_dist = pm.Normal.dist(mu=0, sigma=sigma)

    # AR过程
    y = pm.AR('y', rho=rho, sigma=sigma, init_dist=init_dist, observed=y_obs)
```

### 混合模型

```python
with pm.Model() as mixture_model:
    # 组分权重
    w = pm.Dirichlet('w', a=np.ones(n_components))

    # 组分参数
    mu = pm.Normal('mu', mu=0, sigma=10, shape=n_components)
    sigma = pm.HalfNormal('sigma', sigma=1, shape=n_components)

    # 混合
    components = [pm.Normal.dist(mu=mu[i], sigma=sigma[i]) for i in range(n_components)]
    y = pm.Mixture('y', w=w, comp_dists=components, observed=y_obs)
```

## 数据准备最佳实践

### 标准化

标准化连续预测变量以改善采样效果：

```python
# 标准化
X_mean = X.mean(axis=0)
X_std = X.std(axis=0)
X_scaled = (X - X_mean) / X_std

# 使用标准化数据建模
with pm.Model() as model:
    beta_scaled = pm.Normal('beta_scaled', 0, 1)
    # ... 模型其余部分 ...

# 转换回原始尺度
beta_original = beta_scaled / X_std
alpha_original = alpha - (beta_scaled * X_mean / X_std).sum()
```

### 处理缺失数据

将缺失值视为参数：

```python
# 识别缺失值
missing_idx = np.isnan(X)
X_observed = np.where(missing_idx, 0, X)  # 占位符

with pm.Model() as model:
    # 缺失值先验
    X_missing = pm.Normal('X_missing', mu=0, sigma=1, shape=missing_idx.sum())

    # 合并观测值与插补值
    X_complete = pm.math.switch(missing_idx.flatten(), X_missing, X_observed.flatten())

    # ... 使用X_complete构建模型其余部分 ...
```

### 中心化与标准化

回归模型中中心化预测变量和结果变量：

```python
# 中心化
X_centered = X - X.mean(axis=0)
y_centered = y - y.mean()

with pm.Model() as model:
    # 简化截距先验
    alpha = pm.Normal('alpha', mu=0, sigma=1)  # 中心化时截距接近0
    beta = pm.Normal('beta', mu=0, sigma=1, shape=n_predictors)

    mu = alpha + pm.math.dot(X_centered, beta)
    sigma = pm.HalfNormal('sigma', sigma=1)

    y_obs = pm.Normal('y_obs', mu=mu, sigma=sigma, observed=y_centered)
```

## 先验选择指南

### 弱信息先验

在缺乏先验知识时使用：

```python
# 标准化预测变量
beta = pm.Normal('beta', mu=0, sigma=1)

# 尺度参数
sigma = pm.HalfNormal('sigma', sigma=1)

# 概率参数
p = pm.Beta('p', alpha=2, beta=2)  # 轻微偏好中间值
```

### 信息先验

利用领域知识：

```python
# 文献效应量：Cohen's d ≈ 0.3
beta = pm.Normal('beta', mu=0.3, sigma=0.1)

# 物理约束：概率在0.7-0.9之间
p = pm.Beta('p', alpha=8, beta=2)  # 需通过先验预测检验验证！
```

### 先验预测检验

始终验证先验分布：

```python
with model:
    prior_pred = pm.sample_prior_predictive(samples=1000)

# 检查预测是否合理
print(f"先验预测范围：{prior_pred.prior_predictive['y'].min():.2f} 至 {prior_pred.prior_predictive['y'].max():.2f}")
print(f"观测范围：{y_obs.min():.2f} 至 {y_obs.max():.2f}")

# 可视化
az.plot_ppc(prior_pred, group='prior')
```

## 模型比较工作流

### 多模型比较

```python
import arviz as az

# 拟合多个模型
models = {}
idatas = {}

# 模型1：简单线性
with pm.Model() as models['linear']:
    # ... 定义模型 ...
    idatas['linear'] = pm.sample(idata_kwargs={'log_likelihood': True})

# 模型2：含交互项
with pm.Model() as models['interaction']:
    # ... 定义模型 ...
    idatas['interaction'] = pm.sample(idata_kwargs={'log_likelihood': True})

# 模型3：分层模型
with pm.Model() as models['hierarchical']:
    # ... 定义模型 ...
    idatas['hierarchical'] = pm.sample(idata_kwargs={'log_likelihood': True})

# 使用LOO比较
comparison = az.compare(idatas, ic='loo')
print(comparison)

# 可视化比较结果
az.plot_compare(comparison)
plt.show()

# 检查LOO可靠性
for name, idata in idatas.items():
    loo = az.loo(idata, pointwise=True)
    high_pareto_k = (loo.pareto_k > 0.7).sum().item()
    if high_pareto_k > 0:
        print(f"警告：{name}有{high_pareto_k}个观测值存在高Pareto-k值")
```

### 模型权重

```python
# 获取模型权重（伪BMA）
weights = comparison['weight'].values

print("模型概率：")
for name, weight in zip(comparison.index, weights):
    print(f"  {name}: {weight:.2%}")

# 模型平均（加权预测）
def weighted_predictions(idatas, weights):
    preds = []
    for (name, idata), weight in zip(idatas.items(), weights):
        pred = idata.posterior_predictive['y_obs'].mean(dim=['chain', 'draw'])
        preds.append(weight * pred)
    return sum(preds)

averaged_pred = weighted_predictions(idatas, weights)
```

## 诊断与故障排除

### 采样问题诊断

```python
def diagnose_sampling(idata, var_names=None):
    """综合采样诊断"""

    # 检查收敛性
    summary = az.summary(idata, var_names=var_names)

    print("=== 收敛性诊断 ===")
    bad_rhat = summary[summary['r_hat'] > 1.01]
    if len(bad_rhat) > 0:
        print(f"⚠️  {len(bad_rhat)}个变量的R-hat值>1.01")
        print(bad_rhat[['r_hat']])
    else:
        print("✓ 所有R-hat值<1.01")

    # 检查有效样本量
    print("\n=== 有效样本量(ESS) ===")
    low_ess = summary[summary['ess_bulk'] < 400]
    if len(low_ess) > 0:
        print(f"⚠️  {len(low_ess)}个变量的ESS<400")
        print(low_ess[['ess_bulk', 'ess_tail']])
    else:
        print("✓ 所有ESS值>400")

    # 检查发散
    print("\n=== 发散诊断 ===")
    divergences = idata.sample_stats.diverging.sum().item()
    if divergences > 0:
        print(f"⚠️  {divergences}次发散转移")
        print("   建议：增加target_accept、重新参数化或使用更强先验")
    else:
        print("✓ 无发散")

    # 检查树深度
    print("\n=== NUTS统计量 ===")
    max_treedepth = idata.sample_stats.tree_depth.max().item()
    hits_max = (idata.sample_stats.tree_depth == max_treedepth).sum().item()
    if hits_max > 0:
        print(f"⚠️  达到最大树深度{hits_max}次")
        print("   建议：重新参数化或增加max_treedepth")
    else:
        print(f"✓ 无最大树深度问题（最大值：{max_treedepth}）")

    return summary

# 使用示例
diagnose_sampling(idata, var_names=['alpha', 'beta', 'sigma'])
```

### 常见问题解决方案

| 问题 | 解决方案 |
|---------|----------|
| 发散 | 增加`target_accept=0.95`，使用非中心化参数化 |
| 低ESS | 增加采样次数，重新参数化以减少相关性 |
| 高R-hat | 延长链运行时间，检查多模态，改进初始化 |
| 采样缓慢 | 使用ADVI初始化，重新参数化，降低模型复杂度 |
| 后验偏差 | 检查先验预测，确保似

alpha = pm.Normal('alpha', mu=0, sigma=1, dims='groups')
    y = pm.Normal('y', mu=0, sigma=1, dims=['groups', 'time'], observed=data)

# 采样后维度得以保留
idata = pm.sample()

# 简便的子集选择
beta_age = idata.posterior['beta'].sel(predictors='age')
group_A = idata.posterior['alpha'].sel(groups='A')
```

## 保存与加载结果

```python
# 保存 InferenceData
idata.to_netcdf('results.nc')

# 加载 InferenceData
loaded_idata = az.from_netcdf('results.nc')

# 保存模型以供后续预测
import pickle

with open('model.pkl', 'wb') as f:
    pickle.dump({'model': model, 'idata': idata}, f)

# 加载模型
with open('model.pkl', 'rb') as f:
    saved = pickle.load(f)
    model = saved['model']
    idata = saved['idata']
```
