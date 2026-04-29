---
name: sympy
description: 当在Python中进行符号数学运算时使用此技能。该技能适用于符号计算任务，包括代数解方程、执行微积分运算（导数、积分、极限）、操作代数表达式、符号化处理矩阵、物理计算、数论问题、几何计算以及从数学表达式生成可执行代码。当用户需要精确的符号结果而非数值近似值，或处理包含变量和参数的数学公式时，请应用此技能。
license: https://github.com/sympy/sympy/blob/master/LICENSE
metadata:
    skill-author: K-Dense Inc.
---

# SymPy - Python中的符号数学库

## 概述

SymPy是一个用于符号数学的Python库，支持使用数学符号进行精确计算而非数值近似。本技能提供使用SymPy执行符号代数、微积分、线性代数、方程求解、物理计算和代码生成的全面指南。

## 何时使用本技能

在以下场景使用本技能：
- 符号化求解方程（代数方程、微分方程、方程组）
- 执行微积分运算（导数、积分、极限、级数）
- 操作和简化代数表达式
- 符号化处理矩阵和线性代数
- 进行物理计算（力学、量子力学、向量分析）
- 数论计算（质数、因式分解、模运算）
- 几何计算（2D/3D几何、解析几何）
- 将数学表达式转换为可执行代码（Python、C、Fortran）
- 生成LaTeX或其他格式化数学输出
- 需要精确数学结果（例如保留`sqrt(2)`而非`1.414...`）

## 核心能力

### 1. 符号计算基础

**创建符号和表达式：**
```python
from sympy import symbols, Symbol
x, y, z = symbols('x y z')
expr = x**2 + 2*x + 1

# 带假设条件
x = symbols('x', real=True, positive=True)
n = symbols('n', integer=True)
```

**简化和操作：**
```python
from sympy import simplify, expand, factor, cancel
simplify(sin(x)**2 + cos(x)**2)  # 返回1
expand((x + 1)**3)  # x**3 + 3*x**2 + 3*x + 1
factor(x**2 - 1)    # (x - 1)*(x + 1)
```

**详细基础操作：** 参见`references/core-capabilities.md`

### 2. 微积分

**导数：**
```python
from sympy import diff
diff(x**2, x)        # 2*x
diff(x**4, x, 3)     # 24*x (三阶导数)
diff(x**2*y**3, x, y)  # 6*x*y**2 (偏导数)
```

**积分：**
```python
from sympy import integrate, oo
integrate(x**2, x)              # x**3/3 (不定积分)
integrate(x**2, (x, 0, 1))      # 1/3 (定积分)
integrate(exp(-x), (x, 0, oo))  # 1 (反常积分)
```

**极限和级数：**
```python
from sympy import limit, series
limit(sin(x)/x, x, 0)  # 1
series(exp(x), x, 0, 6)  # 1 + x + x**2/2 + x**3/6 + x**4/24 + x**5/120 + O(x**6)
```

**详细微积分操作：** 参见`references/core-capabilities.md`

### 3. 方程求解

**代数方程：**
```python
from sympy import solveset, solve, Eq
solveset(x**2 - 4, x)  # {-2, 2}
solve(Eq(x**2, 4), x)  # [-2, 2]
```

**方程组：**
```python
from sympy import linsolve, nonlinsolve
linsolve([x + y - 2, x - y], x, y)  # {(1, 1)} (线性)
nonlinsolve([x**2 + y - 2, x + y**2 - 3], x, y)  # (非线性)
```

**微分方程：**
```python
from sympy import Function, dsolve, Derivative
f = symbols('f', cls=Function)
dsolve(Derivative(f(x), x) - f(x), f(x))  # Eq(f(x), C1*exp(x))
```

**详细求解方法：** 参见`references/core-capabilities.md`

### 4. 矩阵与线性代数

**矩阵创建和操作：**
```python
from sympy import Matrix, eye, zeros
M = Matrix([[1, 2], [3, 4]])
M_inv = M**-1  # 逆矩阵
M.det()        # 行列式
M.T            # 转置
```

**特征值和特征向量：**
```python
eigenvals = M.eigenvals()  # {特征值: 重数}
eigenvects = M.eigenvects()  # [(特征值, 重数, [特征向量])]
P, D = M.diagonalize()  # M = P*D*P^-1
```

**求解线性系统：**
```python
A = Matrix([[1, 2], [3, 4]])
b = Matrix([5, 6])
x = A.solve(b)  # 求解 Ax = b
```

**完整线性代数：** 参见`references/matrices-linear-algebra.md`

### 5. 物理与力学

**经典力学：**
```python
from sympy.physics.mechanics import dynamicsymbols, LagrangesMethod
from sympy import symbols

# 定义系统
q = dynamicsymbols('q')
m, g, l = symbols('m g l')

# 拉格朗日量 (T - V)
L = m*(l*q.diff())**2/2 - m*g*l*(1 - cos(q))

# 应用拉格朗日方法
LM = LagrangesMethod(L, [q])
```

**向量分析：**
```python
from sympy.physics.vector import ReferenceFrame, dot, cross
N = ReferenceFrame('N')
v1 = 3*N.x + 4*N.y
v2 = 1*N.x + 2*N.z
dot(v1, v2)  # 点积
cross(v1, v2)  # 叉积
```

**量子力学：**
```python
from sympy.physics.quantum import Ket, Bra, Commutator
psi = Ket('psi')
A = Operator('A')
comm = Commutator(A, B).doit()
```

**详细物理能力：** 参见`references/physics-mechanics.md`

### 6. 高等数学

本技能全面支持：

- **几何：** 2D/3D解析几何、点、线、圆、多边形、变换
- **数论：** 质数、因式分解、最大公约数/最小公倍数、模运算、丢番图方程
- **组合数学：** 排列、组合、划分、群论
- **逻辑与集合：** 布尔逻辑、集合论、有限集与无限集
- **统计：** 概率分布、随机变量、期望、方差
- **特殊函数：** Gamma函数、贝塞尔函数、正交多项式、超几何函数
- **多项式：** 多项式代数、求根、因式分解、Gröbner基

**详细高等主题：** 参见`references/advanced-topics.md`

### 7. 代码生成与输出

**转换为可执行函数：**
```python
from sympy import lambdify
import numpy as np

expr = x**2 + 2*x + 1
f = lambdify(x, expr, 'numpy')  # 创建NumPy函数
x_vals = np.linspace(0, 10, 100)
y_vals = f(x_vals)  # 快速数值计算
```

**生成C/Fortran代码：**
```python
from sympy.utilities.codegen import codegen
[(c_name, c_code), (h_name, h_header)] = codegen(
    ('my_func', expr), 'C'
)
```

**LaTeX输出：**
```python
from sympy import latex
latex_str = latex(expr)  # 转换为LaTeX文档格式
```

**完整代码生成：** 参见`references/code-generation-printing.md`

## SymPy使用最佳实践

### 1. 始终先定义符号

```python
from sympy import symbols
x, y, z = symbols('x y z')
# 现在x, y, z可用于表达式
```

### 2. 使用假设优化简化

```python
x = symbols('x', positive=True, real=True)
sqrt(x**2)  # 返回x（而非Abs(x)），因设定了positive假设
```

常用假设：`real`, `positive`, `negative`, `integer`, `rational`, `complex`, `even`, `odd`

### 3. 使用精确算术

```python
from sympy import Rational, S
# 正确（精确）：
expr = Rational(1, 2) * x
expr = S(1)/2 * x

# 错误（浮点数）：
expr = 0.5 * x  # 创建近似值
```

### 4. 按需数值计算

```python
from sympy import pi, sqrt
result = sqrt(8) + pi
result.evalf()    # 5.96371554103586
result.evalf(50)  # 50位精度
```

### 5. 转换为NumPy提升性能

```python
# 多次计算效率低：
for x_val in range(1000):
    result = expr.subs(x, x_val).evalf()

# 高效方式：
f = lambdify(x, expr, 'numpy')
results = f(np.arange(1000))
```

### 6. 选用合适的求解器

- `solveset`: 代数方程（首选）
- `linsolve`: 线性系统
- `nonlinsolve`: 非线性系统
- `dsolve`: 微分方程
- `solve`: 通用求解器（传统但灵活）

## 参考文件结构

本技能采用模块化参考文件组织不同能力：

1. **`core-capabilities.md`**: 符号、代数、微积分、简化、方程求解
   - 适用场景：基础符号计算、微积分或方程求解

2. **`matrices-linear-algebra.md`**: 矩阵操作、特征值、线性系统
   - 适用场景：矩阵或线性代数问题

3. **`physics-mechanics.md`**: 经典力学、量子力学、向量、单位制
   - 适用场景：物理计算或力学问题

4. **`advanced-topics.md`**: 几何、数论、组合数学、逻辑、统计
   - 适用场景：超出基础代数和微积分的高等数学主题

5. **`code-generation-printing.md`**: Lambdify、codegen、LaTeX输出、打印
   - 适用场景：表达式转代码或生成格式化输出

## 常用模式案例

### 模式1：求解与验证

```python
from sympy import symbols, solve, simplify
x = symbols('x')

# 求解方程
equation = x**2 - 5*x + 6
solutions = solve(equation, x)  # [2, 3]

# 验证解
for sol in solutions:
    result = simplify(equation.subs(x, sol))
    assert result == 0
```

### 模式2：符号到数值的转换流程

```python
# 1. 定义符号问题
x, y = symbols('x y')
expr = sin(x) + cos(y)

# 2. 符号化处理
simplified = simplify(expr)
derivative = diff(simplified, x)

# 3. 转换为数值函数
f = lambdify((x, y), derivative, 'numpy')

# 4. 数值计算
results = f(x_data, y_data)
```

### 模式3：生成数学结果文档

```python
# 符号化计算结果
integral_expr = Integral(x**2, (x, 0, 1))
result = integral_expr.doit()

# 生成文档
print(f"LaTeX: {latex(integral_expr)} = {latex(result)}")
print(f"Pretty: {pretty(integral_expr)} = {pretty(result)}")
print(f"数值结果: {result.evalf()}")
```

## 科学工作流集成

### 与NumPy集成

```python
import numpy as np
from sympy import symbols, lambdify

x = symbols('x')
expr = x**2 + 2*x + 1

f = lambdify(x, expr, 'numpy')
x_array = np.linspace(-5, 5, 100)
y_array = f(x_array)
```

### 与Matplotlib集成

```python
import matplotlib.pyplot as plt
import numpy as np
from sympy import symbols, lambdify, sin

x = symbols('x')
expr = sin(x) / x

f = lambdify(x, expr, 'numpy')
x_vals = np.linspace(-10, 10, 1000)
y_vals = f(x_vals)

plt.plot(x_vals, y_vals)
plt.show()
```

### 与SciPy集成

```python
from scipy.optimize import fsolve
from sympy import symbols, lambdify

# 符号化定义方程
x = symbols('x')
equation = x**3 - 2*x - 5

# 转换为数值函数
f = lambdify(x, equation, 'numpy')

# 带初始猜测数值求解
solution = fsolve(f, 2)
```

## 速查表：最常用函数

```python
# 符号
from sympy import symbols, Symbol
x, y = symbols('x y')

# 基础运算
from sympy import simplify, expand, factor, collect, cancel
from sympy import sqrt, exp, log, sin, cos, tan, pi, E, I, oo

# 微积分
from sympy import diff, integrate, limit, series, Derivative, Integral

# 求解
from sympy import solve, solveset, linsolve, nonlinsolve, dsolve

# 矩阵
from sympy import Matrix, eye, zeros, ones, diag

# 逻辑与集合
from sympy import And, Or, Not, Implies, FiniteSet, Interval, Union

# 输出
from sympy import latex, pprint, lambdify, init_printing

# 工具
from sympy import evalf, N, nsimplify
```

## 入门示例

### 示例1：求解二次方程
```python
from sympy import symbols, solve, sqrt
x = symbols('x')
solution = solve(x**2 - 5*x + 6, x)
# [2, 3]
```

### 示例2：计算导数
```python
from sympy import symbols, diff, sin
x = symbols('x')
f = sin(x**2)
df_dx = diff(f, x)
# 2*x*cos(x**2)
```

### 示例3：计算积分
```python
from sympy import symbols, integrate, exp
x = symbols('x')
integral = integrate(x * exp(-x**2), (x, 0, oo))
# 1/2
```

### 示例4：矩阵特征值
```python
from sympy import Matrix
M = Matrix([[1, 2], [2, 1]])
eigenvals = M.eigenvals()
# {3: 1, -1: 1}
```

### 示例5：生成Python函数
```python
from sympy import symbols, lambdify
import numpy as np
x = symbols('x')
expr = x**2 + 2*x + 1
f = lambdify(x, expr, 'numpy')
f(np.array([1, 2, 3]))
# array([ 4,  9, 16])
```

## 常见问题排查

1. **"NameError: name 'x' is not defined"**
   - 解决方案：使用前务必通过`symbols()`定义符号

2. **意外数值结果**
   - 问题：使用`0.5`等浮点数而非`Rational(1, 2)`
   - 解决方案：使用`Rational()`或`S()`进行精确算术

3. **循环中性能低下**
   - 问题：重复使用`subs()`和`evalf()`
   - 解决方案：使用`lambdify()`创建快速数值函数

4. **"无法求解此方程"**
   - 尝试不同求解器：`solve`, `solveset`, `nsolve`（数值）
   - 检查方程是否可代数求解
   - 若无闭式解则使用数值方法

5. **简化未达预期效果**
   - 尝试不同简化函数：`simplify`, `factor`, `expand`, `trigsimp`

- 为符号添加假设（例如，`positive=True`）
   - 使用 `simplify(expr, force=True)` 进行激进简化

## 附加资源

- 官方文档：https://docs.sympy.org/
- 教程：https://docs.sympy.org/latest/tutorials/intro-tutorial/index.html
- API 参考：https://docs.sympy.org/latest/reference/index.html
- 示例：https://github.com/sympy/sympy/tree/master/examples
