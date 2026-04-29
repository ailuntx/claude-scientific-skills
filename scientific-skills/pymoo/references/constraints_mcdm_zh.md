# Pymoo 约束处理与决策参考

关于 pymoo 中约束处理和多准则决策的参考指南。

## 约束处理

### 定义约束

约束在问题定义中指定：

```python
from pymoo.core.problem import ElementwiseProblem
import numpy as np

class ConstrainedProblem(ElementwiseProblem):
    def __init__(self):
        super().__init__(
            n_var=2,
            n_obj=2,
            n_ieq_constr=2,    # 不等式约束数量
            n_eq_constr=1,      # 等式约束数量
            xl=np.array([0, 0]),
            xu=np.array([5, 5])
        )

    def _evaluate(self, x, out, *args, **kwargs):
        # 目标函数
        f1 = x[0]**2 + x[1]**2
        f2 = (x[0]-1)**2 + (x[1]-1)**2

        out["F"] = [f1, f2]

        # 不等式约束 (需满足 g(x) <= 0)
        g1 = x[0] + x[1] - 5  # x[0] + x[1] >= 5 → -(x[0] + x[1] - 5) <= 0
        g2 = x[0]**2 + x[1]**2 - 25  # x[0]^2 + x[1]^2 <= 25

        out["G"] = [g1, g2]

        # 等式约束 (需满足 h(x) = 0)
        h1 = x[0] - 2*x[1]

        out["H"] = [h1]
```

**约束公式规则：**
- 不等式：`g(x) <= 0`（负值或零时可行）
- 等式：`h(x) = 0`（等于零时可行）
- 将 `g(x) >= 0` 转换为 `-g(x) <= 0`

### 约束处理技术

#### 1. 可行性优先（默认）
**机制：** 始终优先选择可行解
**比较规则：**
1. 两者可行 → 按目标值比较
2. 一可行一不可行 → 可行解胜出
3. 两者不可行 → 按约束违反程度比较

**用法：**
```python
from pymoo.algorithms.moo.nsga2 import NSGA2

# 可行性优先是大多数算法的默认设置
algorithm = NSGA2(pop_size=100)
```

**优点：**
- 适用于任何基于排序的算法
- 简单有效
- 无需参数调整

**缺点：**
- 在可行域较小时可能表现不佳
- 可能忽略优质的不可行解

#### 2. 罚函数法
**机制：** 根据约束违反程度添加目标惩罚项
**公式：** `F_penalized = F + penalty_factor * violation`

**用法：**
```python
from pymoo.algorithms.soo.nonconvex.ga import GA
from pymoo.constraints.as_penalty import ConstraintsAsPenalty

# 使用罚函数包装问题
problem_with_penalty = ConstraintsAsPenalty(problem, penalty=1e6)

algorithm = GA(pop_size=100)
```

**参数：**
- `penalty`：惩罚系数（需根据问题规模调整）

**优点：**
- 将约束问题转换为无约束问题
- 适用于任何优化算法

**缺点：**
- 惩罚参数敏感
- 可能需要针对问题专门调整

#### 3. 约束转目标
**机制：** 将约束违反视为额外目标
**结果：** 多目标问题变为 M+1 个目标（M 个原始目标 + 约束目标）

**用法：**
```python
from pymoo.algorithms.moo.nsga2 import NSGA2
from pymoo.constraints.as_obj import ConstraintsAsObjective

# 添加约束违反作为目标
problem_with_cv_obj = ConstraintsAsObjective(problem)

algorithm = NSGA2(pop_size=100)
```

**优点：**
- 无需参数调整
- 保留可能有用的不可行解
- 在可行域较小时表现良好

**缺点：**
- 增加问题维度
- 帕累托前沿分析更复杂

#### 4. ε约束处理
**机制：** 动态可行性阈值
**概念：** 随迭代代数逐步收紧约束容忍度

**优点：**
- 平滑过渡到可行域
- 适用于复杂约束场景

**缺点：**
- 需特定算法实现
- 需要参数调整

#### 5. 修复算子
**机制：** 修改不可行解使其满足约束
**应用：** 在交叉/变异后修复后代

**用法：**
```python
from pymoo.core.repair import Repair

class MyRepair(Repair):
    def _do(self, problem, X, **kwargs):
        # 将X投影到可行域
        # 示例：边界裁剪
        X = np.clip(X, problem.xl, problem.xu)
        return X

from pymoo.algorithms.soo.nonconvex.ga import GA

algorithm = GA(pop_size=100, repair=MyRepair())
```

**优点：**
- 在整个优化过程中保持可行性
- 可编码领域知识

**缺点：**
- 需要针对问题实现
- 可能限制搜索空间

### 约束处理算法

部分算法内置约束处理机制：

#### SRES（随机排序进化策略）
**用途：** 单目标约束优化
**机制：** 随机排序平衡目标与约束

**用法：**
```python
from pymoo.algorithms.soo.nonconvex.sres import SRES

algorithm = SRES()
```

#### ISRES（改进型SRES）
**用途：** 增强型约束优化
**改进：** 更好的参数自适应

**用法：**
```python
from pymoo.algorithms.soo.nonconvex.isres import ISRES

algorithm = ISRES()
```

### 约束处理指南

**根据问题特性选择技术：**

| 问题特征 | 推荐技术 |
|------------------------|----------------------|
| 可行域较大 | 可行性优先 |
| 可行域较小 | 约束转目标、修复算子 |
| 强约束问题 | SRES/ISRES、ε约束 |
| 线性约束 | 修复算子（投影法） |
| 非线性约束 | 可行性优先、罚函数法 |
| 已知可行解 | 偏置初始化 |

## 多准则决策（MCDM）

获得帕累托前沿后，MCDM 帮助选择偏好解。

### 决策背景

**帕累托前沿特征：**
- 多个非支配解
- 每个解代表不同权衡
- 不存在客观"最优"解
- 需要决策者偏好

### Pymoo 中的 MCDM 方法

#### 1. 伪权重法
**概念：** 为各目标分配权重，选择加权和最小的解
**公式：** `score = w1*f1 + w2*f2 + ... + wM*fM`

**用法：**
```python
from pymoo.mcdm.pseudo_weights import PseudoWeights

# 定义权重（总和为1）
weights = np.array([0.3, 0.7])  # f1权重30%，f2权重70%

dm = PseudoWeights(weights)
best_idx = dm.do(result.F)
best_solution = result.X[best_idx]
```

**适用场景：**
- 偏好表达明确
- 目标可公度
- 可接受线性权衡

**局限：**
- 需指定权重
- 线性假设可能不符合偏好
- 对目标缩放敏感

#### 2. 折衷规划
**概念：** 选择最接近理想点的解
**度量：** 到理想点的距离（如欧氏距离、切比雪夫距离）

**用法：**
```python
from pymoo.mcdm.compromise_programming import CompromiseProgramming

dm = CompromiseProgramming()
best_idx = dm.do(result.F, ideal=ideal_point, nadir=nadir_point)
```

**适用场景：**
- 已知或可估计理想目标值
- 需平衡考虑所有目标
- 无明确权重偏好

#### 3. 交互式决策
**概念：** 迭代式偏好优化
**流程：**
1. 向决策者展示代表性解
2. 收集偏好反馈
3. 聚焦搜索偏好区域
4. 重复直至获得满意解

**方法：**
- 参考点法
- 权衡分析法
- 渐进式偏好表达

### 决策流程

**步骤1：目标归一化**
```python
# 归一化至[0,1]区间
F_norm = (result.F - result.F.min(axis=0)) / (result.F.max(axis=0) - result.F.min(axis=0))
```

**步骤2：分析权衡关系**
```python
from pymoo.visualization.scatter import Scatter

plot = Scatter()
plot.add(result.F)
plot.show()

# 识别拐点、极端解
```

**步骤3：应用MCDM方法**
```python
from pymoo.mcdm.pseudo_weights import PseudoWeights

weights = np.array([0.4, 0.6])  # 基于偏好
dm = PseudoWeights(weights)
selected = dm.do(F_norm)
```

**步骤4：验证选择**
```python
# 可视化选定解
from pymoo.visualization.petal import Petal

plot = Petal()
plot.add(result.F[selected], label="选定解")
# 添加其他候选解对比
plot.show()
```

### 高级MCDM技术

#### 拐点检测
**概念：** 微小目标改进会导致其他目标大幅恶化的解

**用法：**
```python
from pymoo.mcdm.knee import KneePoint

km = KneePoint()
knee_idx = km.do(result.F)
knee_solutions = result.X[knee_idx]
```

**适用场景：**
- 无明确偏好
- 需要平衡权衡
- 凸帕累托前沿

#### 超体积贡献度
**概念：** 选择对超体积贡献最大的解
**用例：** 维护解的多样性子集

**用法：**
```python
from pymoo.indicators.hv import HV

hv = HV(ref_point=reference_point)
hv_contributions = hv.calc_contributions(result.F)

# 选择贡献度最高的解
top_k = 5
top_indices = np.argsort(hv_contributions)[-top_k:]
selected_solutions = result.X[top_indices]
```

### 决策指南

**当决策者具备：**

| 偏好信息 | 推荐方法 |
|------------------------|-------------------|
| 明确目标权重 | 伪权重法 |
| 理想目标值 | 折衷规划 |
| 无先验偏好 | 拐点检测、可视化检查 |
| 冲突准则 | 交互式方法 |
| 需要多样子集 | 超体积贡献度 |

**最佳实践：**
1. **目标归一化**后再应用MCDM
2. **可视化帕累托前沿**理解权衡关系
3. **结合多种方法**确保选择鲁棒性
4. 与领域专家**验证结果**
5. **记录假设**和偏好来源
6. 对权重/参数进行**敏感性分析**

### 完整集成示例

包含约束处理和决策的完整工作流：

```python
from pymoo.algorithms.moo.nsga2 import NSGA2
from pymoo.optimize import minimize
from pymoo.mcdm.pseudo_weights import PseudoWeights
import numpy as np

# 定义约束问题
problem = MyConstrainedProblem()

# 配置算法（默认可行性优先）
algorithm = NSGA2(
    pop_size=100,
    eliminate_duplicates=True
)

# 优化
result = minimize(
    problem,
    algorithm,
    ('n_gen', 200),
    seed=1,
    verbose=True
)

# 仅筛选可行解
feasible_mask = result.CV[:, 0] == 0  # 约束违反=0
F_feasible = result.F[feasible_mask]
X_feasible = result.X[feasible_mask]

# 目标归一化
F_norm = (F_feasible - F_feasible.min(axis=0)) / (F_feasible.max(axis=0) - F_feasible.min(axis=0))

# 应用MCDM
weights = np.array([0.5, 0.5])
dm = PseudoWeights(weights)
best_idx = dm.do(F_norm)

# 获取最终解
best_solution = X_feasible[best_idx]
best_objectives = F_feasible[best_idx]

print(f"选定解: {best_solution}")
print(f"目标值: {best_objectives}")
```
