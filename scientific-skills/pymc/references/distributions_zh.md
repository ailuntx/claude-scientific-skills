# PyMC 分布参考手册

本参考手册提供了 PyMC 中可用概率分布的完整目录，按类别组织。在构建贝叶斯模型时，可用于选择合适的先验分布和似然分布。

## 连续分布

连续分布定义了实值域上的概率密度。

### 常用连续分布

**`pm.Normal(name, mu, sigma)`**
- 正态（高斯）分布
- 参数：`mu`（均值），`sigma`（标准差）
- 支撑集：(-∞, ∞)
- 常见用途：无界参数的默认先验，带加性噪声的连续数据似然

**`pm.HalfNormal(name, sigma)`**
- 半正态分布（正态分布的正半部分）
- 参数：`sigma`（标准差）
- 支撑集：[0, ∞)
- 常见用途：尺度/标准差参数的先验

**`pm.Uniform(name, lower, upper)`**
- 均匀分布
- 参数：`lower`, `upper`（边界）
- 支撑集：[lower, upper]
- 常见用途：参数必须受限时的弱信息先验

**`pm.Beta(name, alpha, beta)`**
- Beta 分布
- 参数：`alpha`, `beta`（形状参数）
- 支撑集：[0, 1]
- 常见用途：概率和比例的先验

**`pm.Gamma(name, alpha, beta)`**
- Gamma 分布
- 参数：`alpha`（形状），`beta`（速率）
- 支撑集：(0, ∞)
- 常见用途：正参数、速率参数的先验

**`pm.Exponential(name, lam)`**
- 指数分布
- 参数：`lam`（速率参数）
- 支撑集：[0, ∞)
- 常见用途：尺度参数、等待时间的先验

**`pm.LogNormal(name, mu, sigma)`**
- 对数正态分布
- 参数：`mu`, `sigma`（基础正态分布的参数）
- 支撑集：(0, ∞)
- 常见用途：具有乘性效应的正参数先验

**`pm.StudentT(name, nu, mu, sigma)`**
- 学生 t 分布
- 参数：`nu`（自由度），`mu`（位置），`sigma`（尺度）
- 支撑集：(-∞, ∞)
- 常见用途：正态分布的稳健替代，用于抗异常值模型

**`pm.Cauchy(name, alpha, beta)`**
- 柯西分布
- 参数：`alpha`（位置），`beta`（尺度）
- 支撑集：(-∞, ∞)
- 常见用途：正态分布的重尾替代

### 专用连续分布

**`pm.Laplace(name, mu, b)`** - 拉普拉斯（双指数）分布  
**`pm.AsymmetricLaplace(name, kappa, mu, b)`** - 非对称拉普拉斯分布  
**`pm.InverseGamma(name, alpha, beta)`** - 逆 Gamma 分布  
**`pm.Weibull(name, alpha, beta)`** - 可靠性分析中的威布尔分布  
**`pm.Logistic(name, mu, s)`** - 逻辑分布  
**`pm.LogitNormal(name, mu, sigma)`** - (0,1) 支撑集的对数正态分布  
**`pm.Pareto(name, alpha, m)`** - 幂律现象的帕累托分布  
**`pm.ChiSquared(name, nu)`** - 卡方分布  
**`pm.ExGaussian(name, mu, sigma, nu)`** - 指数修正高斯分布  
**`pm.VonMises(name, mu, kappa)`** - 冯·米塞斯（环形正态）分布  
**`pm.SkewNormal(name, mu, sigma, alpha)`** - 偏态正态分布  
**`pm.Triangular(name, lower, c, upper)`** - 三角分布  
**`pm.Gumbel(name, mu, beta)`** - 极值分析中的冈贝尔分布  
**`pm.Rice(name, nu, sigma)`** - 莱斯（Rician）分布  
**`pm.Moyal(name, mu, sigma)`** - 莫亚尔分布  
**`pm.Kumaraswamy(name, a, b)`** - Kumaraswamy 分布（Beta 替代）  
**`pm.Interpolated(name, x_points, pdf_points)`** - 插值自定义分布  

## 离散分布

离散分布定义了整数值域上的概率。

### 常用离散分布

**`pm.Bernoulli(name, p)`**
- 伯努利分布（二元结果）
- 参数：`p`（成功概率）
- 支撑集：{0, 1}
- 常见用途：二元分类，抛硬币

**`pm.Binomial(name, n, p)`**
- 二项分布
- 参数：`n`（试验次数），`p`（成功概率）
- 支撑集：{0, 1, ..., n}
- 常见用途：固定试验中的成功次数

**`pm.Poisson(name, mu)`**
- 泊松分布
- 参数：`mu`（速率参数）
- 支撑集：{0, 1, 2, ...}
- 常见用途：计数数据，发生率，事件发生

**`pm.Categorical(name, p)`**
- 分类分布
- 参数：`p`（概率向量）
- 支撑集：{0, 1, ..., K-1}
- 常见用途：多类别分类

**`pm.DiscreteUniform(name, lower, upper)`**
- 离散均匀分布
- 参数：`lower`, `upper`（边界）
- 支撑集：{lower, ..., upper}
- 常见用途：有限整数上的均匀先验

**`pm.NegativeBinomial(name, mu, alpha)`**
- 负二项分布
- 参数：`mu`（均值），`alpha`（离散度）
- 支撑集：{0, 1, 2, ...}
- 常见用途：过离散计数数据

**`pm.Geometric(name, p)`**
- 几何分布
- 参数：`p`（成功概率）
- 支撑集：{0, 1, 2, ...}
- 常见用途：首次成功前的失败次数

### 专用离散分布

**`pm.BetaBinomial(name, alpha, beta, n)`** - Beta-二项分布（过离散二项）  
**`pm.HyperGeometric(name, N, k, n)`** - 超几何分布  
**`pm.DiscreteWeibull(name, q, beta)`** - 离散威布尔分布  
**`pm.OrderedLogistic(name, eta, cutpoints)`** - 有序逻辑分布（用于序数数据）  
**`pm.OrderedProbit(name, eta, cutpoints)`** - 有序概率单位分布（用于序数数据）  

## 多元分布

多元分布定义了向量值随机变量的联合概率分布。

### 常用多元分布

**`pm.MvNormal(name, mu, cov)`**
- 多元正态分布
- 参数：`mu`（均值向量），`cov`（协方差矩阵）
- 常见用途：相关连续变量，高斯过程

**`pm.Dirichlet(name, a)`**
- 狄利克雷分布
- 参数：`a`（集中度参数）
- 支撑集：单纯形（和为1）
- 常见用途：概率向量先验，主题建模

**`pm.Multinomial(name, n, p)`**
- 多项分布
- 参数：`n`（试验次数），`p`（概率向量）
- 常见用途：多类别计数数据

**`pm.MvStudentT(name, nu, mu, cov)`**
- 多元学生 t 分布
- 参数：`nu`（自由度），`mu`（位置），`cov`（尺度矩阵）
- 常见用途：稳健多元建模

### 专用多元分布

**`pm.LKJCorr(name, n, eta)`** - LKJ 相关矩阵先验（用于相关矩阵）  
**`pm.LKJCholeskyCov(name, n, eta, sd_dist)`** - 带 Cholesky 分解的 LKJ 先验  
**`pm.Wishart(name, nu, V)`** - Wishart 分布（用于协方差矩阵）  
**`pm.InverseWishart(name, nu, V)`** - 逆 Wishart 分布  
**`pm.MatrixNormal(name, mu, rowcov, colcov)`** - 矩阵正态分布  
**`pm.KroneckerNormal(name, mu, covs, sigma)`** - Kronecker 结构正态分布  
**`pm.CAR(name, mu, W, alpha, tau)`** - 条件自回归（空间）  
**`pm.ICAR(name, W, sigma)`** - 内在条件自回归（空间）  

## 混合分布

混合分布组合了多个组分分布。

**`pm.Mixture(name, w, comp_dists)`**
- 通用混合分布
- 参数：`w`（权重），`comp_dists`（组分分布）
- 常见用途：聚类，多模态数据

**`pm.NormalMixture(name, w, mu, sigma)`**
- 正态混合分布
- 常见用途：高斯混合聚类

### 零膨胀与障碍模型

**`pm.ZeroInflatedPoisson(name, psi, mu)`** - 计数数据中的过量零  
**`pm.ZeroInflatedBinomial(name, psi, n, p)`** - 零膨胀二项分布  
**`pm.ZeroInflatedNegativeBinomial(name, psi, mu, alpha)`** - 零膨胀负二项分布  
**`pm.HurdlePoisson(name, psi, mu)`** - 障碍泊松分布（两部分模型）  
**`pm.HurdleGamma(name, psi, alpha, beta)`** - 障碍 Gamma 分布  
**`pm.HurdleLogNormal(name, psi, mu, sigma)`** - 障碍对数正态分布  

## 时间序列分布

专为时序数据和序列建模设计的分布。

**`pm.AR(name, rho, sigma, init_dist)`**
- 自回归过程
- 参数：`rho`（AR 系数），`sigma`（创新标准差），`init_dist`（初始分布）
- 常见用途：时间序列建模，序列数据

**`pm.GaussianRandomWalk(name, mu, sigma, init_dist)`**
- 高斯随机游走
- 参数：`mu`（漂移），`sigma`（步长），`init_dist`（初始值）
- 常见用途：累积过程，随机游走先验

**`pm.MvGaussianRandomWalk(name, mu, cov, init_dist)`** - 多元高斯随机游走  
**`pm.GARCH11(name, omega, alpha_1, beta_1)`**
- GARCH(1,1) 波动率模型
- 常见用途：金融时间序列，波动率建模

**`pm.EulerMaruyama(name, dt, sde_fn, sde_pars, init_dist)`**
- 通过 Euler-Maruyama 离散化的随机微分方程
- 常见用途：连续时间过程

## 特殊分布

**`pm.Deterministic(name, var)`**
- 确定性变换（非随机变量）
- 用于从其他变量派生的计算量

**`pm.Potential(name, logp)`**
- 添加任意对数概率贡献
- 用于自定义似然组件或约束

**`pm.Flat(name)`**
- 非正常平坦先验（恒定密度）
- 谨慎使用；可能导致采样问题

**`pm.HalfFlat(name)`**
- 正实数上的非正常平坦先验
- 谨慎使用；可能导致采样问题

## 分布修饰器

**`pm.Truncated(name, dist, lower, upper)`** - 将任意分布截断至指定边界  
**`pm.Censored(name, dist, lower, upper)`** - 处理删失观测（观测边界而非精确值）  
**`pm.CustomDist(name, ..., logp, random)`** - 通过用户指定的对数概率和随机采样函数定义自定义分布  
**`pm.Simulator(name, fn, params, ...)`** - 通过模拟的自定义分布（用于无似然推断）  

## 使用技巧

### 选择先验

1. **尺度参数** (σ, τ)：使用 `HalfNormal`、`HalfCauchy`、`Exponential` 或 `Gamma`  
2. **概率**：使用 `Beta` 或 `Uniform(0, 1)`  
3. **无界参数**：使用 `Normal` 或 `StudentT`（增强稳健性）  
4. **正参数**：使用 `LogNormal`、`Gamma` 或 `Exponential`  
5. **相关矩阵**：使用 `LKJCorr`  
6. **计数数据**：使用 `Poisson` 或 `NegativeBinomial`（处理过离散）  

### 形状广播

PyMC 分布支持 NumPy 风格的广播。使用 `shape` 参数创建随机变量的向量或数组：

```python
# 5 个独立正态分布的向量
beta = pm.Normal('beta', mu=0, sigma=1, shape=5)

# 3x4 独立 Gamma 分布的矩阵
tau = pm.Gamma('tau', alpha=2, beta=1, shape=(3, 4))
```

### 使用 dims 命名维度

使用 `dims` 替代 `shape` 使模型更易读：

```python
with pm.Model(coords={'predictors': ['age', 'income', 'education']}) as model:
    beta = pm.Normal('beta', mu=0, sigma=1, dims='predictors')
```
