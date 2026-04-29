# 贝叶斯统计分析

本文档提供了关于如何实施和解释贝叶斯统计分析的指导，该框架为频率派（经典）统计提供了替代方案。

## 贝叶斯与频率派哲学

### 根本差异

| 方面 | 频率派 | 贝叶斯 |
|--------|-------------|----------|
| **概率解释** | 事件的长期频率 | 信念/不确定性的程度 |
| **参数** | 固定但未知 | 具有分布的随机变量 |
| **推断** | 基于抽样分布 | 基于后验分布 |
| **主要输出** | p值、置信区间 | 后验概率、可信区间 |
| **先验信息** | 不正式纳入 | 通过先验明确纳入 |
| **假设检验** | 拒绝/不拒绝零假设 | 给定数据下假设的概率 |
| **样本量** | 通常需要最小值 | 可处理任意样本量 |
| **解释** | 间接（给定H₀时数据的概率） | 直接（给定数据时假设的概率） |

### 核心问题差异

**频率派**："如果零假设成立，观察到如此极端或更极端数据的概率是多少？"

**贝叶斯**："给定观测数据，该假设成立的概率是多少？"

贝叶斯问题更直观，直接回应研究者想知道的内容。

---

## 贝叶斯定理

**公式**：
```
P(θ|D) = P(D|θ) × P(θ) / P(D)
```

**文字表述**：
```
后验 = 似然 × 先验 / 证据
```

其中：
- **θ (theta)**：目标参数（如均值差、相关性）
- **D**：观测数据
- **P(θ|D)**：后验分布（看到数据后对θ的信念）
- **P(D|θ)**：似然（给定θ时数据的概率）
- **P(θ)**：先验分布（看到数据前对θ的信念）
- **P(D)**：边际似然/证据（归一化常数）

---

## 先验分布

### 先验类型

#### 1. 信息性先验

**使用场景**：当具备充分先验知识时：
- 先前研究
- 专家知识
- 理论依据
- 试点数据

**示例**：元分析显示效应量d≈0.40，标准差=0.15
- 先验：正态分布(0.40, 0.15)

**优势**：
- 整合现有知识
- 更高效（所需样本更小）
- 小数据时可稳定估计

**劣势**：
- 主观性（但主观性可成为优势）
- 需提供合理依据并保持透明
- 若强先验与数据冲突可能引发争议

---

#### 2. 弱信息性先验

**使用场景**：多数应用的默认选择

**特征**：
- 正则化估计（防止极端值）
- 中等数据量时对后验影响最小
- 避免计算问题

**示例先验**：
- 效应量：正态分布(0,1)或柯西分布(0,0.707)
- 方差：半柯西分布(0,1)
- 相关性：均匀分布(-1,1)或贝塔分布(2,2)

**优势**：
- 平衡客观性与正则化
- 计算稳定
- 广泛适用

---

#### 3. 无信息性（平坦/均匀）先验

**使用场景**：试图保持"客观性"时

**示例**：任意值的均匀分布(-∞, ∞)

**⚠️ 注意**：
- 可能导致非正常后验
- 可能产生不合理结果
- 并非真正"无信息"（仍含假设）
- 现代贝叶斯实践中通常不推荐

**更优替代**：使用弱信息性先验

---

### 先验敏感性分析

**必须执行**：测试不同先验下结果的变化

**流程**：
1. 用默认/计划先验拟合模型
2. 用更弥散先验拟合模型
3. 用更集中先验拟合模型
4. 比较后验分布

**报告**：
- 若结果相似：证据稳健
- 若结果显著不同：数据强度不足以覆盖先验

**Python示例**：
```python
import pymc as pm

# 不同先验的模型
priors = [
    ('weakly_informative', pm.Normal.dist(0, 1)),
    ('diffuse', pm.Normal.dist(0, 10)),
    ('informative', pm.Normal.dist(0.5, 0.3))
]

results = {}
for name, prior in priors:
    with pm.Model():
        effect = pm.Normal('effect', mu=prior.mu, sigma=prior.sigma)
        # ... 模型其余部分
        trace = pm.sample()
        results[name] = trace
```

---

## 贝叶斯假设检验

### 贝叶斯因子（BF）

**定义**：两个竞争假设证据的比值

**公式**：
```
BF₁₀ = P(D|H₁) / P(D|H₀)
```

**解释**：

| BF₁₀ | 证据强度 |
|------|----------|
| >100 | 决定性支持H₁ |
| 30-100 | 极强支持H₁ |
| 10-30 | 强支持H₁ |
| 3-10 | 中等支持H₁ |
| 1-3 | 微弱支持H₁ |
| 1 | 无证据 |
| 1/3-1 | 微弱支持H₀ |
| 1/10-1/3 | 中等支持H₀ |
| 1/30-1/10 | 强支持H₀ |
| 1/100-1/30 | 极强支持H₀ |
| <1/100 | 决定性支持H₀ |

**相对于p值的优势**：
1. 可为零假设提供证据
2. 不依赖抽样意图（无"偷看"问题）
3. 直接量化证据强度
4. 可随新数据更新

**Python计算**：
```python
import pingouin as pg

# 注意：Python中BF支持有限
# 更佳选择：R包(BayesFactor)、JASP软件

# 从t统计量近似计算BF
# 使用Jeffreys-Zellner-Siow先验
from scipy import stats

def bf_from_t(t, n1, n2, r_scale=0.707):
    """
    从t统计量近似贝叶斯因子
    r_scale: 柯西先验尺度（默认0.707对应中等效应）
    """
    # 此为简化版；精确计算需用专用包
    df = n1 + n2 - 2
    # 实现需数值积分
    pass
```

---

### 实际等价区域（ROPE）

**目的**：定义可忽略效应量的范围

**流程**：
1. 定义ROPE（如d∈[-0.1,0.1]表示可忽略效应）
2. 计算后验位于ROPE内的百分比
3. 决策：
   - >95%在ROPE内：接受实际等价
   - >95%在ROPE外：拒绝等价
   - 否则：不确定

**优势**：直接检验实际显著性

**Python示例**：
```python
# 定义ROPE
rope_lower, rope_upper = -0.1, 0.1

# 计算后验在ROPE内的比例
in_rope = np.mean((posterior_samples > rope_lower) &
                  (posterior_samples < rope_upper))

print(f"{in_rope*100:.1f}% 后验位于ROPE内")
```

---

## 贝叶斯估计

### 可信区间

**定义**：以X%概率包含参数的区间

**95%可信区间解释**：
> "真实参数位于此区间内的概率为95%"

**这正是人们误认为置信区间的含义**（但频率派框架中并非如此）

**类型**：

#### 等尾区间（ETI）
- 2.5%至97.5%分位数
- 计算简单
- 偏态分布可能不包含众数

#### 最高密度区间（HDI）
- 包含95%分布的最窄区间
- 始终包含众数
- 更适用于偏态分布

**Python计算**：
```python
import arviz as az

# 等尾区间
eti = np.percentile(posterior_samples, [2.5, 97.5])

# HDI
hdi = az.hdi(posterior_samples, hdi_prob=0.95)
```

---

### 后验分布

**后验分布解释**：

1. **集中趋势**：
   - 均值：后验平均值
   - 中位数：50%分位数
   - 众数：最可能值（MAP-最大后验估计）

2. **不确定性**：
   - 标准差：后验离散度
   - 可信区间：量化不确定性

3. **形态**：
   - 对称：类似正态分布
   - 偏态：非对称不确定性
   - 多峰：多个合理取值

**可视化**：
```python
import matplotlib.pyplot as plt
import arviz as az

# 带HDI的后验图
az.plot_posterior(trace, hdi_prob=0.95)

# 轨迹图（检查收敛性）
az.plot_trace(trace)

# 森林图（多参数）
az.plot_forest(trace)
```

---

## 常见贝叶斯分析

### 贝叶斯T检验

**目的**：比较两组差异（t检验的贝叶斯替代）

**输出**：
1. 均值差的后验分布
2. 95%可信区间
3. 贝叶斯因子（BF₁₀）
4. 方向性假设概率（如P(μ₁ > μ₂)）

**Python实现**：
```python
import pymc as pm
import arviz as az

# 贝叶斯独立样本t检验
with pm.Model() as model:
    # 组均值先验
    mu1 = pm.Normal('mu1', mu=0, sigma=10)
    mu2 = pm.Normal('mu2', mu=0, sigma=10)

    # 合并标准差先验
    sigma = pm.HalfNormal('sigma', sigma=10)

    # 似然函数
    y1 = pm.Normal('y1', mu=mu1, sigma=sigma, observed=group1)
    y2 = pm.Normal('y2', mu=mu2, sigma=sigma, observed=group2)

    # 派生量：均值差
    diff = pm.Deterministic('diff', mu1 - mu2)

    # 抽样后验
    trace = pm.sample(2000, tune=1000, return_inferencedata=True)

# 分析结果
print(az.summary(trace, var_names=['mu1', 'mu2', 'diff']))

# 组1>组2的概率
prob_greater = np.mean(trace.posterior['diff'].values > 0)
print(f"P(μ₁ > μ₂) = {prob_greater:.3f}")

# 绘制后验
az.plot_posterior(trace, var_names=['diff'], ref_val=0)
```

---

### 贝叶斯方差分析

**目的**：比较三组及以上差异

**模型**：
```python
import pymc as pm

with pm.Model() as anova_model:
    # 超先验
    mu_global = pm.Normal('mu_global', mu=0, sigma=10)
    sigma_between = pm.HalfNormal('sigma_between', sigma=5)
    sigma_within = pm.HalfNormal('sigma_within', sigma=5)

    # 组均值（分层）
    group_means = pm.Normal('group_means',
                            mu=mu_global,
                            sigma=sigma_between,
                            shape=n_groups)

    # 似然函数
    y = pm.Normal('y',
                  mu=group_means[group_idx],
                  sigma=sigma_within,
                  observed=data)

    trace = pm.sample(2000, tune=1000, return_inferencedata=True)

# 后验对比
contrast_1_2 = trace.posterior['group_means'][:,:,0] - trace.posterior['group_means'][:,:,1]
```

---

### 贝叶斯相关性

**目的**：估计两变量间相关性

**优势**：提供相关值的分布

**Python实现**：
```python
import pymc as pm

with pm.Model() as corr_model:
    # 相关性先验
    rho = pm.Uniform('rho', lower=-1, upper=1)

    # 转换为协方差矩阵
    cov_matrix = pm.math.stack([[1, rho],
                                [rho, 1]])

    # 似然函数（二元正态）
    obs = pm.MvNormal('obs',
                     mu=[0, 0],
                     cov=cov_matrix,
                     observed=np.column_stack([x, y]))

    trace = pm.sample(2000, tune=1000, return_inferencedata=True)

# 相关性摘要
print(az.summary(trace, var_names=['rho']))

# 相关性为正的概率
prob_positive = np.mean(trace.posterior['rho'].values > 0)
```

---

### 贝叶斯线性回归

**目的**：建模预测变量与结果的关系

**优势**：
- 所有参数的不确定性
- 自然正则化（通过先验）
- 可整合先验知识
- 预测值的可信区间

**Python实现**：
```python
import pymc as pm

with pm.Model() as regression_model:
    # 系数先验
    alpha = pm.Normal('alpha', mu=0, sigma=10)  # 截距
    beta = pm.Normal('beta', mu=0, sigma=10, shape=n_predictors)
    sigma = pm.HalfNormal('sigma', sigma=10)

    # 期望值
    mu = alpha + pm.math.dot(X, beta)

    # 似然函数
    y_obs = pm.Normal('y_obs', mu=mu, sigma=sigma, observed=y)

    trace = pm.sample(2000, tune=1000, return_inferencedata=True)

# 后验预测检验
with regression_model:
    ppc = pm.sample_posterior_predictive(trace)

az.plot_ppc(ppc)

# 含不确定性的预测
with regression_model:
    pm.set_data({'X': X_new})
    posterior_pred = pm.sample_posterior_predictive(trace)
```

---

## 分层（多水平）模型

**使用场景**：
- 嵌套/聚类数据（学校内的学生）
- 重复测量
- 元分析
- 跨组别变化效应

**核心概念**：部分池化
- 完全池化：忽略组别（有偏）
- 无池化：独立分析组别（高方差）
- 部分池化：跨组别借力（贝叶斯方法）

**示例：变化截距**：
```python
with pm.Model() as hierarchical_model:
    # 超先验
    mu_global = pm.Normal('mu_global', mu=0, sigma=10)
    sigma_between = pm.HalfNormal('sigma_between', sigma=5)
    sigma_within = pm.HalfNormal('sigma_within', sigma=5)

    # 组别截距
    alpha = pm.Normal('alpha',
                     mu=mu_global,
                     sigma=sigma_between,
                     shape=n_groups)

    # 似然函数
    y_obs = pm.Normal('y_obs',
                     mu=alpha[group_idx],
                     sigma=sigma_within,
                     observed=y)

    trace = pm.sample()
```

---

## 模型比较

### 方法

#### 1. 贝叶斯因子
- 直接比较模型证据
- 对先验设定敏感
- 计算可能复杂

#### 2. 信息准则

**WAIC（广泛适用信息准则）**：
- AIC的贝叶斯版本
- 值越低越好
- 考虑有效参数数量

**LOO（留一交叉验证）**：
- 估计样本外预测误差
- 值越低越好
- 比WAIC更稳健

**Python计算**：
```python
import arviz as az

# 计算WAIC和LOO
waic = az.waic(trace)
loo = az.loo(trace)

print(f"WAIC: {waic.elpd_waic:.2f}")
print(f"LOO: {loo.elpd_loo:.2f}")

# 多模型比较
comparison = az.compare({
    'model1': trace1,
    'model2': trace2,
    'model3': trace3
})
print(comparison)
```

---

## 贝叶斯模型检验

### 1. 收敛诊断

**R-hat（Gelman-Rubin统计量）**：
- 比较链内与链间方差
- 接近1.0的值表明收敛

- R-hat < 1.01：良好
- R-hat > 1.05：收敛性差

**有效样本量 (ESS)**：
- 独立样本的数量
- 数值越高越好
- 建议每条链的 ESS > 400

**轨迹图**：
- 应呈现"毛虫状"形态
- 无趋势性波动，无停滞链

**Python 检查**：
```python
# 自动生成诊断摘要
print(az.summary(trace, var_names=['parameter']))

# 可视化诊断
az.plot_trace(trace)
az.plot_rank(trace)  # 秩次图
```

---

### 2. 后验预测检验

**目的**：模型生成的数据是否与观测数据相似？

**流程**：
1. 从后验分布生成预测值
2. 与真实数据对比
3. 检查系统性差异

**Python 实现**：
```python
with model:
    ppc = pm.sample_posterior_predictive(trace)

# 可视化检验
az.plot_ppc(ppc, num_pp_samples=100)

# 定量检验
obs_mean = np.mean(observed_data)
pred_means = [np.mean(sample) for sample in ppc.posterior_predictive['y_obs']]
p_value = np.mean(pred_means >= obs_mean)  # 贝叶斯p值
```

---

## 贝叶斯结果报告

### 独立样本T检验报告示例

> "采用贝叶斯独立样本T检验比较组A与组B。使用弱信息先验：均值差服从Normal(0,1)，合并标准差服从Half-Cauchy(0,1)。均值差的后验分布均值为5.2（95%置信区间[2.3,8.1]），表明组A得分显著高于组B。贝叶斯因子BF₁₀=23.5为组间差异提供了强有力证据，组A均值超过组B均值的概率达99.7%。"

### 回归分析报告示例

> "采用弱信息先验（系数服从Normal(0,10)，残差标准差服从Half-Cauchy(0,5)）拟合贝叶斯线性回归模型。模型解释力显著（R²=0.47，95%置信区间[0.38,0.55]）。学习时长（β=0.52，95%置信区间[0.38,0.66]）与既往GPA（β=0.31，95%置信区间[0.17,0.45]）均为可信预测因子（95%置信区间不包含零）。后验预测检验显示模型拟合良好，收敛诊断指标满意（所有R-hat<1.01，ESS>1000）。"

---

## 优势与局限

### 优势

1. **直观解释**：直接获得参数的概率陈述
2. **整合先验知识**：充分利用现有信息
3. **灵活性**：轻松处理复杂模型
4. **规避p值操纵**：支持实时数据观测
5. **量化不确定性**：提供完整后验分布
6. **小样本适用**：不受样本量限制

### 局限

1. **计算要求**：依赖MCMC采样（可能耗时）
2. **先验设定**：需充分论证合理性
3. **复杂性**：学习曲线陡峭
4. **软件生态**：工具少于频率学派方法
5. **沟通成本**：需向审稿人/读者普及知识

---

## 核心Python工具包

- **PyMC**：完整贝叶斯建模框架
- **ArviZ**：可视化与诊断工具
- **Bambi**：回归模型高层接口
- **PyStan**：Stan的Python接口
- **TensorFlow Probability**：基于TensorFlow的贝叶斯推断

---

## 贝叶斯方法适用场景

**推荐使用贝叶斯方法当**：
- 需整合先验信息
- 需要直接概率陈述
- 样本量较小
- 模型复杂（分层结构/缺失数据等）
- 需实时更新分析结果

**频率学派方法可能足够当**：
- 大样本标准分析
- 无先验信息可用
- 计算资源有限
- 审稿人不熟悉贝叶斯方法
