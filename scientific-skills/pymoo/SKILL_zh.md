---
name: pymoo
description: 多目标优化框架。提供NSGA-II、NSGA-III、MOEA/D算法，支持帕累托前沿、约束处理、基准测试问题（ZDT, DTLZ），适用于工程设计与优化问题。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# Pymoo - Python多目标优化框架

## 概述

Pymoo是一个专注于多目标优化的综合性Python框架。通过前沿算法（NSGA-II/III, MOEA/D）解决单目标与多目标优化问题，支持基准测试问题（ZDT, DTLZ）、可定制遗传算子及多准则决策方法。擅长为存在目标冲突的问题寻找权衡解（帕累托前沿）。

## 适用场景

本技能适用于：
- 求解单目标或多目标优化问题
- 寻找帕累托最优解并分析权衡关系
- 实现进化算法（GA, DE, PSO, NSGA-II/III）
- 处理带约束的优化问题
- 在标准测试问题（ZDT, DTLZ, WFG）上评估算法性能
- 定制遗传算子（交叉、变异、选择）
- 可视化高维优化结果
- 从多个竞争方案中做出决策
- 处理二进制、离散、连续或混合变量问题

## 核心概念

### 统一接口

Pymoo通过统一的`minimize()`函数处理所有优化任务：

```python
from pymoo.optimize import minimize

result = minimize(
    problem,        # 优化目标
    algorithm,      # 优化算法
    termination,    # 终止条件
    seed=1,
    verbose=True
)
```

**结果对象包含：**
- `result.X`：最优解的决策变量
- `result.F`：最优解的目标值
- `result.G`：约束违反值（带约束时）
- `result.algorithm`：包含历史记录的算法对象

### 问题类型

**单目标：** 最小化/最大化单一目标  
**多目标：** 2-3个冲突目标 → 帕累托前沿  
**超多目标：** 4个以上目标 → 高维帕累托前沿  
**带约束：** 目标函数 + 不等式/等式约束  
**动态问题：** 时变目标或约束  

## 快速入门流程

### 流程1：单目标优化

**适用场景：** 优化单一目标函数

**步骤：**
1. 定义或选择问题
2. 选择单目标算法（GA, DE, PSO, CMA-ES）
3. 配置终止条件
4. 运行优化
5. 提取最优解

**示例：**
```python
from pymoo.algorithms.soo.nonconvex.ga import GA
from pymoo.problems import get_problem
from pymoo.optimize import minimize

# 内置问题
problem = get_problem("rastrigin", n_var=10)

# 配置遗传算法
algorithm = GA(
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

print(f"最优解: {result.X}")
print(f"最优目标值: {result.F[0]}")
```

**完整示例见：** `scripts/single_objective_example.py`

### 流程2：多目标优化（2-3目标）

**适用场景：** 优化2-3个冲突目标，需获取帕累托前沿

**算法选择：** NSGA-II（双/三目标标准算法）

**步骤：**
1. 定义多目标问题
2. 配置NSGA-II
3. 运行优化获取帕累托前沿
4. 可视化权衡关系
5. 应用决策方法（可选）

**示例：**
```python
from pymoo.algorithms.moo.nsga2 import NSGA2
from pymoo.problems import get_problem
from pymoo.optimize import minimize
from pymoo.visualization.scatter import Scatter

# 双目标基准问题
problem = get_problem("zdt1")

# NSGA-II算法
algorithm = NSGA2(pop_size=100)

# 优化
result = minimize(problem, algorithm, ('n_gen', 200), seed=1)

# 可视化帕累托前沿
plot = Scatter()
plot.add(result.F, label="获取前沿")
plot.add(problem.pareto_front(), label="真实前沿", alpha=0.3)
plot.show()

print(f"找到 {len(result.F)} 个帕累托最优解")
```

**完整示例见：** `scripts/multi_objective_example.py`

### 流程3：超多目标优化（4+目标）

**适用场景：** 优化4个及以上目标

**算法选择：** NSGA-III（专为超多目标设计）

**关键区别：** 必须提供参考方向引导种群

**步骤：**
1. 定义超多目标问题
2. 生成参考方向
3. 配置带参考方向的NSGA-III
4. 运行优化
5. 使用平行坐标图可视化

**示例：**
```python
from pymoo.algorithms.moo.nsga3 import NSGA3
from pymoo.problems import get_problem
from pymoo.optimize import minimize
from pymoo.util.ref_dirs import get_reference_directions
from pymoo.visualization.pcp import PCP

# 超多目标问题（5目标）
problem = get_problem("dtlz2", n_obj=5)

# 生成参考方向（NSGA-III必需）
ref_dirs = get_reference_directions("das-dennis", n_dim=5, n_partitions=12)

# 配置NSGA-III
algorithm = NSGA3(ref_dirs=ref_dirs)

# 优化
result = minimize(problem, algorithm, ('n_gen', 300), seed=1)

# 平行坐标可视化
plot = PCP(labels=[f"f{i+1}" for i in range(5)])
plot.add(result.F, alpha=0.3)
plot.show()
```

**完整示例见：** `scripts/many_objective_example.py`

### 流程4：自定义问题定义

**适用场景：** 求解领域特定优化问题

**步骤：**
1. 继承`ElementwiseProblem`类
2. 在`__init__`中定义问题维度与边界
3. 在`_evaluate`中实现目标（及约束）计算
4. 与任意算法配合使用

**无约束示例：**
```python
from pymoo.core.problem import ElementwiseProblem
import numpy as np

class MyProblem(ElementwiseProblem):
    def __init__(self):
        super().__init__(
            n_var=2,              # 变量数量
            n_obj=2,              # 目标数量
            xl=np.array([0, 0]),  # 下界
            xu=np.array([5, 5])   # 上界
        )

    def _evaluate(self, x, out, *args, **kwargs):
        # 定义目标函数
        f1 = x[0]**2 + x[1]**2
        f2 = (x[0]-1)**2 + (x[1]-1)**2

        out["F"] = [f1, f2]
```

**带约束示例：**
```python
class ConstrainedProblem(ElementwiseProblem):
    def __init__(self):
        super().__init__(
            n_var=2,
            n_obj=2,
            n_ieq_constr=2,        # 不等式约束数量
            n_eq_constr=1,         # 等式约束数量
            xl=np.array([0, 0]),
            xu=np.array([5, 5])
        )

    def _evaluate(self, x, out, *args, **kwargs):
        # 目标函数
        out["F"] = [f1, f2]

        # 不等式约束 (g <= 0)
        out["G"] = [g1, g2]

        # 等式约束 (h = 0)
        out["H"] = [h1]
```

**约束公式规则：**
- 不等式：表达为 `g(x) <= 0`（≤0时为可行解）
- 等式：表达为 `h(x) = 0`（=0时为可行解）
- 将 `g(x) >= b` 转换为 `-(g(x) - b) <= 0`

**完整示例见：** `scripts/custom_problem_example.py`

### 流程5：约束处理

**适用场景：** 问题包含可行性约束

**处理方法选项：**

**1. 可行性优先（默认推荐）**
```python
from pymoo.algorithms.moo.nsga2 import NSGA2

# 自动处理带约束问题
algorithm = NSGA2(pop_size=100)
result = minimize(problem, algorithm, termination)

# 检查可行性
feasible = result.CV[:, 0] == 0  # CV = 约束违反值
print(f"可行解数量: {np.sum(feasible)}")
```

**2. 罚函数法**
```python
from pymoo.constraints.as_penalty import ConstraintsAsPenalty

# 将约束转换为罚函数
problem_penalized = ConstraintsAsPenalty(problem, penalty=1e6)
```

**3. 约束转目标**
```python
from pymoo.constraints.as_obj import ConstraintsAsObjective

# 将约束违反值作为额外目标
problem_with_cv = ConstraintsAsObjective(problem)
```

**4. 专用算法**
```python
from pymoo.algorithms.soo.nonconvex.sres import SRES

# SRES内置约束处理
algorithm = SRES()
```

**完整约束处理指南见：** `references/constraints_mcdm.md`

### 流程6：帕累托前沿决策

**适用场景：** 需从帕累托前沿选择偏好解

**步骤：**
1. 运行多目标优化
2. 将目标归一化至[0,1]
3. 定义偏好权重
4. 应用多准则决策方法
5. 可视化选定解

**伪权重法示例：**
```python
from pymoo.mcdm.pseudo_weights import PseudoWeights
import numpy as np

# 获取多目标优化结果后
# 目标归一化
F_norm = (result.F - result.F.min(axis=0)) / (result.F.max(axis=0) - result.F.min(axis=0))

# 定义偏好（总和为1）
weights = np.array([0.3, 0.7])  # 30% f1, 70% f2

# 应用决策方法
dm = PseudoWeights(weights)
selected_idx = dm.do(F_norm)

# 获取选定解
best_solution = result.X[selected_idx]
best_objectives = result.F[selected_idx]

print(f"选定解: {best_solution}")
print(f"目标值: {best_objectives}")
```

**其他决策方法：**
- 折衷规划：选择最接近理想点的解
- 拐点法：寻找平衡的权衡解
- 超体积贡献：选择最具多样性的子集

**完整示例见：**
- `scripts/decision_making_example.py`
- `references/constraints_mcdm.md`（详细决策方法）

### 流程7：结果可视化

**根据目标数量选择可视化方式：**

**2目标：散点图**
```python
from pymoo.visualization.scatter import Scatter

plot = Scatter(title="双目标结果")
plot.add(result.F, color="blue", alpha=0.7)
plot.show()
```

**3目标：3D散点图**
```python
plot = Scatter(title="三目标结果")
plot.add(result.F)  # 自动3D渲染
plot.show()
```

**4+目标：平行坐标图**
```python
from pymoo.visualization.pcp import PCP

plot = PCP(
    labels=[f"f{i+1}" for i in range(n_obj)],
    normalize_each_axis=True
)
plot.add(result.F, alpha=0.3)
plot.show()
```

**方案对比：花瓣图**
```python
from pymoo.visualization.petal import Petal

plot = Petal(
    bounds=[result.F.min(axis=0), result.F.max(axis=0)],
    labels=["成本", "重量", "效率"]
)
plot.add(solution_A, label="方案A")
plot.add(solution_B, label="方案B")
plot.show()
```

**所有可视化类型见：** `references/visualization.md`

## 算法选择指南

### 单目标问题

| 算法 | 适用场景 | 核心特性 |
|------|----------|----------|
| **GA** | 通用场景 | 灵活可定制的算子 |
| **DE** | 连续优化 | 全局搜索能力强 |
| **PSO** | 平滑地形 | 快速收敛 |
| **CMA-ES** | 复杂/噪声问题 | 自适应机制 |

### 多目标问题（2-3目标）

| 算法 | 适用场景 | 核心特性 |
|------|----------|----------|
| **NSGA-II** | 标准基准 | 快速可靠，久经考验 |
| **R-NSGA-II** | 偏好区域 | 参考点引导 |
| **MOEA/D** | 可分解问题 | 标量化方法 |

### 超多目标问题（4+目标）

| 算法 | 适用场景 | 核心特性 |
|------|----------|----------|
| **NSGA-III** | 4-15目标 | 基于参考方向 |
| **RVEA** | 自适应搜索 | 参考向量进化 |
| **AGE-MOEA** | 复杂地形 | 自适应几何结构 |

### 带约束问题

| 方法 | 算法 | 适用场景 |
|------|------|----------|
| 可行性优先 | 任意算法 | 可行域较大时 |
| 专用算法 | SRES, ISRES | 强约束场景 |
| 罚函数法 | GA + 罚函数 | 算法兼容性要求 |

**完整算法参考见：** `references/algorithms.md`

## 基准测试问题

### 快速调用问题：
```python
from pymoo.problems import get_problem

# 单目标
problem = get_problem("rastrigin", n_var=10)
problem = get_problem("rosenbrock", n_var=10)

# 多目标
problem = get_problem("zdt1")        # 凸前沿
problem = get_problem("zdt2")        # 非凸前沿
problem = get_problem("zdt3")        # 非连续前沿

# 超多目标
problem = get_problem("dtlz2", n_obj=5, n_var=12)
problem = get_problem("dtlz7", n_obj=4)
```

**完整测试问题参考见：** `references/problems.md`

## 遗传算子定制

### 标准算子配置：
```python
from pymoo.algorithms.soo.nonconvex.ga import GA
from pymoo.operators.crossover.sbx import SBX
from pymoo.operators.mutation.pm import PM

algorithm = GA(
    pop_size=100,
    crossover=SBX(prob=0.9, eta=15),
    mutation=PM(eta=20),
    eliminate_duplicates=True
)
```

### 按变量类型选择算子：

**连续变量：**
- 交叉：SBX（模拟二进制交叉）
- 变异：PM（多项式变异）

**二进制变量：**
- 交叉：双点交叉、均匀交叉
- 变异：位翻转变异

**排列问题（TSP, 调度）：**
- 交叉：顺序交叉（OX）
- 变异：逆转变异

**完整算子参考见：** `references/operators.md`

## 性能与问题排查

### 常见问题及解决方案：

**问题：算法不收敛**
- 增大种群规模
- 增加迭代代数
- 检查是否为多峰问题（尝试不同算法）
- 验证约束公式正确性

**问题：帕累托前沿分布不均**
-

- 算法详情：`grep -r "NSGA-II\|NSGA-III\|MOEA/D" references/`
- 约束方法：`grep -r "Feasibility First\|Penalty\|Repair" references/`
- 可视化类型：`grep -r "Scatter\|PCP\|Petal" references/`

### scripts/
可执行示例展示常见工作流：

- **single_objective_example.py**：使用遗传算法进行基础单目标优化
- **multi_objective_example.py**：使用NSGA-II进行多目标优化及可视化
- **many_objective_example.py**：使用NSGA-III进行高维多目标优化（含参考方向）
- **custom_problem_example.py**：定义自定义问题（含约束与无约束）
- **decision_making_example.py**：基于不同偏好的多准则决策

**运行示例：**
```bash
python3 scripts/single_objective_example.py
python3 scripts/multi_objective_example.py
python3 scripts/many_objective_example.py
python3 scripts/custom_problem_example.py
python3 scripts/decision_making_example.py
```

## 补充说明

**安装：**
```bash
uv pip install pymoo
```

**依赖项：** NumPy, SciPy, matplotlib, autograd（梯度计算可选）

**文档：** https://pymoo.org/

**版本：** 本技能基于 pymoo 0.6.x

**常用模式：**
- 自定义问题始终使用 `ElementwiseProblem`
- 约束条件定义为 `g(x) <= 0` 和 `h(x) = 0`
- NSGA-III 需要参考方向
- 多准则决策前需归一化目标函数
- 使用适当终止条件：`('n_gen', N)` 或 `get_termination("f_tol", tol=0.001)`
