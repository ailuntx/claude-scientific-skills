# 统计假设与诊断流程

本文档为各类分析中统计假设的检验与验证提供全面指导。

## 基本原则

1. **在解读检验结果前务必验证假设**
2. **采用多种诊断方法**（可视化 + 正式检验）
3. **考虑稳健性**：某些检验在特定条件下对假设违背具有稳健性
4. **在分析报告中记录所有假设检验过程**
5. **报告违背情况及采取的补救措施**

## 通用假设条件

### 1. 观测值独立性

**含义**：各观测值相互独立；一个受试对象的测量结果不影响其他受试对象的测量结果。

**检验方法**：
- 审查研究设计和数据收集流程
- 时间序列数据：检验自相关性（ACF/PACF图、Durbin-Watson检验）
- 聚类数据：考虑组内相关系数（ICC）

**违背处理**：
- 聚类/分层数据使用混合效应模型
- 时间相关数据使用时序分析方法
- 相关数据使用广义估计方程（GEE）

**严重程度**：高——违背会显著增加I类错误概率

---

### 2. 正态性

**含义**：数据或残差服从正态（高斯）分布。

**适用场景**：
- t检验（小样本需满足；当n>30/组时具有稳健性）
- 方差分析（小样本需满足；当n>30/组时具有稳健性）
- 线性回归（残差需满足）
- 部分相关检验（Pearson）

**检验方法**：

**可视化方法**（主要）：
- Q-Q（分位数）图：数据点应落在对角线上
- 叠加正态曲线的直方图
- 核密度图

**正式检验**（辅助）：
- Shapiro-Wilk检验（推荐n<50时使用）
- Kolmogorov-Smirnov检验
- Anderson-Darling检验

**Python实现**：
```python
from scipy import stats
import matplotlib.pyplot as plt

# Shapiro-Wilk检验
statistic, p_value = stats.shapiro(data)

# Q-Q图
stats.probplot(data, dist="norm", plot=plt)
```

**解读指南**：
- n<30：需同时重视可视化和正式检验
- 30≤n<100：以可视化为主，正式检验为辅
- n≥100：正式检验过于敏感，应依赖可视化
- 重点关注严重偏态、离群值或双峰分布

**违背处理**：
- **轻度违背**（轻微偏态）：当n>30/组时可继续分析
- **中度违背**：使用非参数替代方法（Mann-Whitney、Kruskal-Wallis、Wilcoxon）
- **严重违背**：
  - 数据转换（对数、平方根、Box-Cox）
  - 采用非参数方法
  - 使用稳健回归方法
  - 考虑自助法（bootstrap）

**严重程度**：中——当样本量充足时，参数检验对轻度违背通常具有稳健性

---

### 3. 方差齐性（同方差性）

**含义**：各组间或预测变量范围内的方差保持恒定。

**适用场景**：
- 独立样本t检验
- 方差分析
- 线性回归（残差需满足恒定方差）

**检验方法**：

**可视化方法**（主要）：
- 分组箱线图（用于t检验/方差分析）
- 残差-拟合值散点图（回归分析）——应呈现随机分布
- 尺度-位置图（标准化残差平方根 vs 拟合值）

**正式检验**（辅助）：
- Levene检验（对非正态性稳健）
- Bartlett检验（对非正态性敏感，不推荐）
- Brown-Forsythe检验（Levene检验的中位数版本）
- Breusch-Pagan检验（回归分析专用）

**Python实现**：
```python
from scipy import stats
import pingouin as pg

# Levene检验
statistic, p_value = stats.levene(group1, group2, group3)

# 回归分析
# Breusch-Pagan检验
from statsmodels.stats.diagnostic import het_breuschpagan
_, p_value, _, _ = het_breuschpagan(residuals, exog)
```

**解读指南**：
- 方差比（最大/最小）<2-3：通常可接受
- 方差分析：当组间样本量相同时检验具有稳健性
- 回归分析：关注残差图中的漏斗形模式

**违背处理**：
- **t检验**：采用Welch t检验（不要求方差齐性）
- **方差分析**：使用Welch方差分析或Brown-Forsythe方差分析
- **回归分析**：
  - 转换因变量（对数、平方根）
  - 使用加权最小二乘法（WLS）
  - 采用稳健标准误（HC3）
  - 使用带合适方差函数的广义线性模型（GLM）

**严重程度**：中——当样本量相同时检验具有稳健性

---

## 特定检验的假设条件

### T检验

**假设条件**：
1. 观测值独立性
2. 正态性（独立t检验要求各组正态；配对t检验要求差值正态）
3. 方差齐性（仅独立t检验要求）

**诊断流程**：
```python
import scipy.stats as stats
import pingouin as pg

# 检验各组正态性
stats.shapiro(group1)
stats.shapiro(group2)

# 检验方差齐性
stats.levene(group1, group2)

# 若假设违背：
# 方案1：Welch t检验（方差不齐）
pg.ttest(group1, group2, correction=False)  # Welch法

# 方案2：非参数替代
pg.mwu(group1, group2)  # Mann-Whitney U检验
```

---

### 方差分析

**假设条件**：
1. 组内与组间观测值独立
2. 各组数据正态分布
3. 组间方差齐性

**附加考量**：
- 重复测量方差分析需满足球形假设（Mauchly检验）

**诊断流程**：
```python
import pingouin as pg

# 检验各组正态性
for group in df['group'].unique():
    data = df[df['group'] == group]['value']
    stats.shapiro(data)

# 检验方差齐性
pg.homoscedasticity(df, dv='value', group='group')

# 重复测量：检验球形假设
# pingouin的rm_anova会自动检验
```

**球形假设违背处理**（重复测量）：
- Greenhouse-Geisser校正（ε < 0.75时）
- Huynh-Feldt校正（ε > 0.75时）
- 采用多元方法（MANOVA）

---

### 线性回归

**假设条件**：
1. **线性关系**：X与Y呈线性关联
2. **独立性**：残差相互独立
3. **同方差性**：残差具有恒定方差
4. **正态性**：残差服从正态分布
5. **无多重共线性**：预测变量间无高度相关（多元回归）

**诊断流程**：

**1. 线性关系**：
```python
import matplotlib.pyplot as plt
import seaborn as sns

# 绘制Y与每个X的散点图
# 残差-拟合值图（应呈随机分布）
plt.scatter(fitted_values, residuals)
plt.axhline(y=0, color='r', linestyle='--')
```

**2. 独立性**：
```python
from statsmodels.stats.stattools import durbin_watson

# Durbin-Watson检验（时序数据）
dw_statistic = durbin_watson(residuals)
# 1.5-2.5区间值表明独立性
```

**3. 同方差性**：
```python
# Breusch-Pagan检验
from statsmodels.stats.diagnostic import het_breuschpagan
_, p_value, _, _ = het_breuschpagan(residuals, exog)

# 可视化：尺度-位置图
plt.scatter(fitted_values, np.sqrt(np.abs(std_residuals)))
```

**4. 残差正态性**：
```python
# 残差Q-Q图
stats.probplot(residuals, dist="norm", plot=plt)

# Shapiro-Wilk检验
stats.shapiro(residuals)
```

**5. 多重共线性**：
```python
from statsmodels.stats.outliers_influence import variance_inflation_factor

# 计算各预测变量的VIF
vif_data = pd.DataFrame()
vif_data["feature"] = X.columns
vif_data["VIF"] = [variance_inflation_factor(X.values, i) for i in range(len(X.columns))]

# VIF>10表明严重多重共线性
# VIF>5表明中度多重共线性
```

**违背处理**：
- **非线性**：添加多项式项、使用GAM或变量转换
- **异方差性**：转换Y变量、采用WLS或稳健标准误
- **残差非正态**：转换Y变量、使用稳健方法、检查离群值
- **多重共线性**：移除相关预测变量、使用主成分分析或岭回归

---

### 逻辑回归

**假设条件**：
1. **独立性**：观测值相互独立
2. **线性关系**：连续预测变量与logit值呈线性关联
3. **无完全多重共线性**：预测变量无完全相关
4. **大样本量**：每个预测变量至少对应10-20个事件

**诊断流程**：

**1. Logit线性**：
```python
# Box-Tidwell检验：添加连续预测变量的对数交互项
# 若交互项显著，则违背线性假设
```

**2. 多重共线性**：
```python
# 采用与线性回归相同的VIF检验
```

**3. 强影响点**：
```python
# Cook距离、DFBetas、杠杆值
from statsmodels.stats.outliers_influence import OLSInfluence

influence = OLSInfluence(model)
cooks_d = influence.cooks_distance
```

**4. 模型拟合**：
```python
# Hosmer-Lemeshow检验
# 伪R方
# 分类指标（准确率、AUC-ROC）
```

---

## 离群值检测

**方法**：
1. **可视化**：箱线图、散点图
2. **统计方法**：
   - Z分数法：|z|>3提示离群值
   - IQR法：< Q1-1.5×IQR 或 > Q3+1.5×IQR的值
   - 中位数绝对偏差修正Z分数法（对离群值稳健）

**回归分析专用**：
- **杠杆值**：高杠杆点（帽子值）
- **影响力**：Cook距离>4/n提示强影响点
- **离群值**：学生化残差>±3

**处理建议**：
1. 核查数据录入错误
2. 评估离群值是否为有效观测
3. 报告敏感性分析（包含/排除离群值的结果）
4. 若离群值合理，采用稳健方法

---

## 样本量考量

### 最小样本量（经验法则）

- **t检验**：每组n≥30可保证对非正态性的稳健性
- **方差分析**：每组n≥30
- **相关分析**：n≥30以保证足够功效
- **简单回归**：n≥50
- **多元回归**：每个预测变量n≥10-20（最低10+k个预测变量）
- **逻辑回归**：每个预测变量n≥10-20个事件

### 小样本注意事项

小样本分析时：
- 假设条件更为关键
- 优先使用精确检验（Fisher精确检验、精确逻辑回归）
- 考虑非参数替代方法
- 采用置换检验或自助法
- 结果解读需保持保守

---

## 假设检验的报告规范

分析报告中需包含：

1. **检验假设声明**：列出所有验证的假设
2. **使用方法**：说明采用的可视化和正式检验方法
3. **诊断结果**：报告检验统计量和p值
4. **评估结论**：说明假设是否满足
5. **处理措施**：若违背假设，描述补救方案（变量转换、替代检验、稳健方法）

**报告范例**：
> "采用Shapiro-Wilk检验和Q-Q图评估正态性。A组（W=0.97, p=.18）与B组（W=0.96, p=.12）数据均未显著偏离正态分布。通过Levene检验评估方差齐性，结果不显著（F(1,58)=1.23, p=.27），表明组间方差齐性成立。因此独立样本t检验的假设条件均满足。"
