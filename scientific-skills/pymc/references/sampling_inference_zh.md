# PyMC 采样与推断方法

本参考指南涵盖 PyMC 中用于后验推断的采样算法和推断方法。

## MCMC 采样方法

### 主要采样函数

**`pm.sample(draws=1000, tune=1000, chains=4, **kwargs)`**

PyMC 中 MCMC 采样的主要接口。

**关键参数：**
- `draws`：每条链的采样次数（默认：1000）
- `tune`：调优/预热样本数（默认：1000，最终丢弃）
- `chains`：并行链数量（默认：4）
- `cores`：使用的 CPU 核心数（默认：全部可用）
- `target_accept`：步长调优的目标接受率（默认：0.8，复杂后验可提升至 0.9-0.95）
- `random_seed`：确保可复现性的随机种子
- `return_inferencedata`：返回 ArviZ InferenceData 对象（默认：True）
- `idata_kwargs`：InferenceData 创建的附加参数（如模型比较时使用 `{"log_likelihood": True}`）

**返回：** 包含后验样本、采样统计和诊断的 InferenceData 对象

**示例：**
```python
with pm.Model() as model:
    # ... 定义模型 ...
    idata = pm.sample(draws=2000, tune=1000, chains=4, target_accept=0.9)
```

### 采样算法

PyMC 会根据模型结构自动选择合适采样器，也可手动指定算法。

#### NUTS（No-U-Turn 采样器）

连续参数的**默认算法**，高效的哈密顿蒙特卡洛变体。

- 自动调优步长和质量矩阵
- 自适应：在调优阶段探索后验几何结构
- 最适合平滑连续后验
- 可能在高相关性或多模态时表现不佳

**手动指定：**
```python
with model:
    idata = pm.sample(step=pm.NUTS(target_accept=0.95))
```

**调整建议：**
- 出现发散时增加 `target_accept`（0.9-0.99）
- 快速初始化使用 `init='adapt_diag'`（默认）
- 复杂初始化使用 `init='jitter+adapt_diag'`

#### Metropolis

通用 Metropolis-Hastings 采样器。

- 适用于连续和离散变量
- 对平滑连续后验效率低于 NUTS
- 适用于离散参数或不可微模型
- 需手动调优

**示例：**
```python
with model:
    idata = pm.sample(step=pm.Metropolis())
```

#### 切片采样器

单变量分布的切片采样。

- 无需调优
- 适合复杂单变量后验
- 高维时可能较慢

**示例：**
```python
with model:
    idata = pm.sample(step=pm.Slice())
```

#### 复合步骤

为不同参数组合不同采样器。

**示例：**
```python
with model:
    # 连续参数用 NUTS，离散参数用 Metropolis
    step1 = pm.NUTS([continuous_var1, continuous_var2])
    step2 = pm.Metropolis([discrete_var])
    idata = pm.sample(step=[step1, step2])
```

### 采样诊断

PyMC 自动计算诊断指标，结果可信前需检查：

#### 有效样本量（ESS）

衡量相关样本中的独立信息量。

- **经验准则**：每条链 ESS > 400（4 条链总计 1600）
- 低 ESS 表示高自相关性
- 访问方式：`az.ess(idata)`

#### R-hat（Gelman-Rubin 统计量）

衡量链间收敛性。

- **经验准则**：所有参数 R-hat < 1.01
- R-hat > 1.01 表示未收敛
- 访问方式：`az.rhat(idata)`

#### 发散

指示 NUTS 采样困难的区域。

- **经验准则**：零发散（或极少）
- 发散暗示样本偏差
- **修复**：增加 `target_accept`、重新参数化或使用更强先验
- 访问方式：`idata.sample_stats.diverging.sum()`

#### 能量图

可视化哈密顿蒙特卡洛能量转移。

```python
az.plot_energy(idata)
```

能量分布良好分离表示采样健康。

### 处理采样问题

#### 发散

```python
# 提高目标接受率
idata = pm.sample(target_accept=0.95)

# 或使用非中心化参数化重新参数化
# 不良方案（中心化）：
mu = pm.Normal('mu', 0, 1)
sigma = pm.HalfNormal('sigma', 1)
x = pm.Normal('x', mu, sigma, observed=data)

# 优良方案（非中心化）：
mu = pm.Normal('mu', 0, 1)
sigma = pm.HalfNormal('sigma', 1)
x_offset = pm.Normal('x_offset', 0, 1, observed=(data - mu) / sigma)
```

#### 采样缓慢

```python
# 简单模型减少调优步数
idata = pm.sample(tune=500)

# 增加核心数并行化
idata = pm.sample(cores=8, chains=8)

# 使用变分推断初始化
with model:
    approx = pm.fit()  # 运行 ADVI
    idata = pm.sample(start=approx.sample(return_inferencedata=False)[0])
```

#### 高自相关

```python
# 增加采样次数
idata = pm.sample(draws=5000)

# 重新参数化降低相关性
# 回归模型考虑 QR 分解
```

## 变分推断

适用于大型模型或快速探索的近似推断方法。

### ADVI（自动微分变分推断）

**`pm.fit(n=10000, method='advi', **kwargs)`**

用简单分布（通常为平均场高斯）近似后验。

**关键参数：**
- `n`：迭代次数（默认：10000）
- `method`：VI 算法（'advi', 'fullrank_advi', 'svgd'）
- `random_seed`：随机种子

**返回：** 用于采样和分析的近似对象

**示例：**
```python
with model:
    approx = pm.fit(n=50000)
    # 从近似分布抽取样本
    idata = approx.sample(1000)
    # 或采样用于 MCMC 初始化
    start = approx.sample(return_inferencedata=False)[0]
```

**权衡：**
- **优点**：比 MCMC 快得多，可扩展至大数据
- **缺点**：近似解，可能遗漏后验结构，低估不确定性

### 全秩 ADVI

捕捉参数间相关性。

```python
with model:
    approx = pm.fit(method='fullrank_advi')
```

比平均场更精确但更慢。

### SVGD（Stein 变分梯度下降）

非参数化变分推断。

```python
with model:
    approx = pm.fit(method='svgd', n=20000)
```

更好捕捉多模态但计算成本更高。

## 先验与后验预测采样

### 先验预测采样

从先验分布采样（观测数据前）。

**`pm.sample_prior_predictive(samples=500, **kwargs)`**

**目的：**
- 验证先验合理性
- 拟合前检查隐含预测
- 确保模型生成合理数据

**示例：**
```python
with model:
    prior_pred = pm.sample_prior_predictive(samples=1000)

# 可视化先验预测
az.plot_ppc(prior_pred, group='prior')
```

### 后验预测采样

从后验预测分布采样（拟合后）。

**`pm.sample_posterior_predictive(trace, **kwargs)`**

**目的：**
- 通过后验预测检查验证模型
- 为新数据生成预测
- 评估拟合优度

**示例：**
```python
with model:
    # 采样后
    idata = pm.sample()

    # 添加后验预测样本
    pm.sample_posterior_predictive(idata, extend_inferencedata=True)

# 后验预测检查
az.plot_ppc(idata)
```

### 新数据预测

更新数据并采样预测分布：

```python
with model:
    # 原始模型拟合
    idata = pm.sample()

    # 更新新预测值
    pm.set_data({'X': X_new})

    # 采样预测
    post_pred_new = pm.sample_posterior_predictive(
        idata.posterior,
        var_names=['y_pred']
    )
```

## 最大后验估计（MAP）

寻找后验众数（点估计）。

**`pm.find_MAP(start=None, method='L-BFGS-B', **kwargs)`**

**适用场景：**
- 快速点估计
- MCMC 初始化
- 无需完整后验时

**示例：**
```python
with model:
    map_estimate = pm.find_MAP()
    print(map_estimate)
```

**局限性：**
- 无法量化不确定性
- 多模态后验中可能找到局部最优
- 对先验设定敏感

## 推断建议

### 标准工作流

1. **ADVI 快速探索**：
   ```python
   approx = pm.fit(n=20000)
   ```

2. **MCMC 完整推断**：
   ```python
   idata = pm.sample(draws=2000, tune=1000)
   ```

3. **检查诊断**：
   ```python
   az.summary(idata, var_names=['~mu_log__'])  # 排除转换变量
   ```

4. **后验预测采样**：
   ```python
   pm.sample_posterior_predictive(idata, extend_inferencedata=True)
   ```

### 推断方法选择

| 场景 | 推荐方法 |
|------|----------|
| 中小模型，需完整不确定性 | NUTS MCMC |
| 大型模型，初步探索 | ADVI |
| 离散参数 | Metropolis 或边缘化 |
| 分层模型发散 | 非中心化参数化 + NUTS |
| 超大数据 | 小批量 ADVI |
| 快速点估计 | MAP 或 ADVI |

### 参数化技巧

**分层模型非中心化参数化**：

```python
# 中心化（可能导致发散）：
mu = pm.Normal('mu', 0, 10)
sigma = pm.HalfNormal('sigma', 1)
theta = pm.Normal('theta', mu, sigma, shape=n_groups)

# 非中心化（采样更优）：
mu = pm.Normal('mu', 0, 10)
sigma = pm.HalfNormal('sigma', 1)
theta_offset = pm.Normal('theta_offset', 0, 1, shape=n_groups)
theta = pm.Deterministic('theta', mu + sigma * theta_offset)
```

**相关预测变量的 QR 分解**：

```python
import numpy as np

# QR 分解
Q, R = np.linalg.qr(X)

with pm.Model():
    # 不相关系数
    beta_tilde = pm.Normal('beta_tilde', 0, 1, shape=p)

    # 转换回原始尺度
    beta = pm.Deterministic('beta', pm.math.solve(R, beta_tilde))

    mu = pm.math.dot(Q, beta_tilde)
    sigma = pm.HalfNormal('sigma', 1)
    y = pm.Normal('y', mu, sigma, observed=y_obs)
```

## 高级采样

### 序贯蒙特卡洛（SMC）

适用于复杂后验或模型证据估计：

```python
with model:
    idata = pm.sample_smc(draws=2000, chains=4)
```

对多模态后验或 NUTS 困难时效果良好。

### 自定义初始化

提供初始值：

```python
start = {'mu': 0, 'sigma': 1}
with model:
    idata = pm.sample(start=start)
```

或使用 MAP 估计：

```python
with model:
    start = pm.find_MAP()
    idata = pm.sample(start=start)
```
