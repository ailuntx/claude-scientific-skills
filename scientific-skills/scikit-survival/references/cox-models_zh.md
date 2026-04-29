# Cox比例风险模型

## 概述

Cox比例风险模型是半参数模型，用于建立协变量与事件发生时间的关联。个体*i*的风险函数表示为：

**h_i(t) = h_0(t) × exp(β^T x_i)**

其中：
- h_0(t) 是基准风险函数（未指定形式）
- β 是系数向量
- x_i 是个体*i*的协变量向量

核心假设是两个个体的风险比随时间保持恒定（比例风险假设）。

## CoxPHSurvivalAnalysis

用于生存分析的基础Cox比例风险模型。

### 适用场景
- 含删失数据的标准生存分析
- 需要可解释的系数（对数风险比）
- 比例风险假设成立
- 数据集特征维度较低

### 关键参数
- `alpha`：正则化参数（默认值：0，无正则化）
- `ties`：处理同时事件的方法（'breslow'或'efron'）
- `n_iter`：优化过程的最大迭代次数

### 使用示例
```python
from sksurv.linear_model import CoxPHSurvivalAnalysis
from sksurv.datasets import load_gbsg2

# 加载数据
X, y = load_gbsg2()

# 拟合Cox模型
estimator = CoxPHSurvivalAnalysis()
estimator.fit(X, y)

# 获取系数（对数风险比）
coefficients = estimator.coef_

# 预测风险评分
risk_scores = estimator.predict(X)
```

## CoxnetSurvivalAnalysis

带弹性网络惩罚的Cox模型，用于特征选择和正则化。

### 适用场景
- 高维数据（特征数量多）
- 需要自动特征选择
- 需处理多重共线性
- 需要稀疏模型

### 惩罚类型
- **岭回归 (L2)**：alpha_min_ratio=1.0, l1_ratio=0
  - 压缩所有系数
  - 适用于所有特征相关的情况
  
- **Lasso (L1)**：l1_ratio=1.0
  - 执行特征选择（将系数归零）
  - 适用于稀疏模型
  
- **弹性网络**：0 < l1_ratio < 1
  - L1与L2惩罚的组合
  - 平衡特征选择与特征分组

### 关键参数
- `l1_ratio`：L1与L2惩罚的平衡（0=岭回归，1=Lasso）
- `alpha_min_ratio`：正则化路径中最小与最大惩罚的比率
- `n_alphas`：正则化路径上的alpha数量
- `fit_baseline_model`：是否拟合无惩罚基准模型

### 使用示例
```python
from sksurv.linear_model import CoxnetSurvivalAnalysis

# 带弹性网络惩罚的拟合
estimator = CoxnetSurvivalAnalysis(l1_ratio=0.5, alpha_min_ratio=0.01)
estimator.fit(X, y)

# 访问正则化路径
alphas = estimator.alphas_
coefficients_path = estimator.coef_path_

# 使用特定alpha预测
risk_scores = estimator.predict(X, alpha=0.1)
```

### Alpha选择的交叉验证
```python
from sklearn.model_selection import GridSearchCV
from sksurv.metrics import concordance_index_censored

# 定义参数网格
param_grid = {'l1_ratio': [0.1, 0.5, 0.9],
              'alpha_min_ratio': [0.01, 0.001]}

# 使用C-index进行网格搜索
cv = GridSearchCV(CoxnetSurvivalAnalysis(),
                  param_grid,
                  scoring='concordance_index_ipcw',
                  cv=5)
cv.fit(X, y)

# 最优参数
best_params = cv.best_params_
```

## IPCRidge

用于加速失效时间模型的逆删失概率加权岭回归。

### 适用场景
- 倾向使用加速失效时间（AFT）框架而非比例风险模型
- 需要建模特征如何加速/减速生存时间
- 高删失率场景
- 需要岭回归正则化

### 与Cox模型的关键区别
AFT模型假设特征将生存时间乘以常数因子，而非乘以风险率。该模型直接预测对数生存时间。

### 使用示例
```python
from sksurv.linear_model import IPCRidge

# 拟合IPCRidge模型
estimator = IPCRidge(alpha=1.0)
estimator.fit(X, y)

# 预测对数生存时间
log_time = estimator.predict(X)
```

## 模型比较与选择

### 模型选择指南

**选用CoxPHSurvivalAnalysis当：**
- 特征数量少到中等
- 需要可解释的风险比
- 标准生存分析场景

**选用CoxnetSurvivalAnalysis当：**
- 高维数据（p >> n）
- 需要特征选择
- 需识别重要预测因子
- 存在多重共线性

**选用IPCRidge当：**
- AFT框架更适用
- 高删失率场景
- 需直接建模时间而非风险

### 验证比例风险假设

应通过以下方法验证比例风险假设：
- Schoenfeld残差
- 对数-对数生存曲线
- 统计检验（可通过lifelines等包实现）

若假设被违反，可考虑：
- 按违反协变量分层
- 使用时变系数
- 替代模型（AFT、参数模型）

## 结果解释

### Cox模型系数
- 正系数：风险增加（生存期缩短）
- 负系数：风险降低（生存期延长）
- 协变量每增加一单位的风险比 = exp(β)
- 示例：β=0.693 → HR=2.0（风险加倍）

### 风险评分
- 高风险评分 = 高事件风险 = 预期生存期短
- 风险评分具有相对性；需用生存函数进行绝对预测
