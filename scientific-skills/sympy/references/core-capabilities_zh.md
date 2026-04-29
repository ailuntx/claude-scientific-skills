# SymPy 核心功能

本文档涵盖 SymPy 的基础操作：符号计算基础、代数、微积分、化简和方程求解。

## 创建符号与基础操作

### 符号创建

**单个符号：**
```python
from sympy import symbols, Symbol
x = Symbol('x')
# 或更常用：
x, y, z = symbols('x y z')
```

**带假设条件：**
```python
x = symbols('x', real=True, positive=True)
n = symbols('n', integer=True)
```

常用假设：`real`, `positive`, `negative`, `integer`, `rational`, `prime`, `even`, `odd`, `complex`

### 基础算术

SymPy 支持标准 Python 运算符处理符号表达式：
- 加法：`x + y`
- 减法：`x - y`
- 乘法：`x * y`
- 除法：`x / y`
- 幂运算：`x**y`

**重要提示：** 使用 `sympy.Rational()` 或 `S()` 处理精确有理数：
```python
from sympy import Rational, S
expr = Rational(1, 2) * x  # 正确：精确 1/2
expr = S(1)/2 * x          # 正确：精确 1/2
expr = 0.5 * x             # 创建浮点数近似值
```

### 替换与求值

**值替换：**
```python
expr = x**2 + 2*x + 1
expr.subs(x, 3)  # 返回 16
expr.subs({x: 2, y: 3})  # 多重替换
```

**数值求值：**
```python
from sympy import pi, sqrt
expr = sqrt(8)
expr.evalf()      # 2.82842712474619
expr.evalf(20)    # 2.8284271247461900976 (20位精度)
pi.evalf(100)     # pi的100位精度
```

## 化简

SymPy 提供多种化简函数，采用不同策略：

### 通用化简

```python
from sympy import simplify, expand, factor, collect, cancel, trigsimp

# 通用化简（尝试多种方法）
simplify(sin(x)**2 + cos(x)**2)  # 返回 1

# 展开乘积与幂
expand((x + 1)**3)  # x**3 + 3*x**2 + 3*x + 1

# 多项式因式分解
factor(x**3 - x**2 + x - 1)  # (x - 1)*(x**2 + 1)

# 按变量合并项
collect(x*y + x - 3 + 2*x**2 - z*x**2 + x**3, x)

# 有理式约分
cancel((x**2 + 2*x + 1)/(x**2 + x))  # (x + 1)/x
```

### 三角化简

```python
from sympy import sin, cos, tan, trigsimp, expand_trig

# 三角表达式化简
trigsimp(sin(x)**2 + cos(x)**2)  # 1
trigsimp(sin(x)/cos(x))          # tan(x)

# 三角函数展开
expand_trig(sin(x + y))  # sin(x)*cos(y) + sin(y)*cos(x)
```

### 幂与对数化简

```python
from sympy import powsimp, powdenest, log, expand_log, logcombine

# 幂化简
powsimp(x**a * x**b)  # x**(a + b)

# 对数展开
expand_log(log(x*y))  # log(x) + log(y)

# 对数合并
logcombine(log(x) + log(y))  # log(x*y)
```

## 微积分

### 导数

```python
from sympy import diff, Derivative

# 一阶导数
diff(x**2, x)  # 2*x

# 高阶导数
diff(x**4, x, x, x)  # 24*x (三阶导数)
diff(x**4, x, 3)     # 24*x (同上)

# 偏导数
diff(x**2*y**3, x, y)  # 6*x*y**2

# 未求值导数（用于显示）
d = Derivative(x**2, x)
d.doit()  # 求值得 2*x
```

### 积分

**不定积分：**
```python
from sympy import integrate

integrate(x**2, x)           # x**3/3
integrate(exp(x)*sin(x), x)  # exp(x)*sin(x)/2 - exp(x)*cos(x)/2
integrate(1/x, x)            # log(x)
```

**注意：** SymPy 不包含积分常数。如需请手动添加 `+ C`。

**定积分：**
```python
from sympy import oo, pi, exp, sin

integrate(x**2, (x, 0, 1))    # 1/3
integrate(exp(-x), (x, 0, oo)) # 1
integrate(sin(x), (x, 0, pi))  # 2
```

**多重积分：**
```python
integrate(x*y, (x, 0, 1), (y, 0, x))  # 1/12
```

**数值积分（符号方法失效时）：**
```python
integrate(x**x, (x, 0, 1)).evalf()  # 0.783430510712134
```

### 极限

```python
from sympy import limit, oo, sin

# 基础极限
limit(sin(x)/x, x, 0)  # 1
limit(1/x, x, oo)      # 0

# 单侧极限
limit(1/x, x, 0, '+')  # oo
limit(1/x, x, 0, '-')  # -oo

# 奇点处使用 limit()（而非 subs()）
limit((x**2 - 1)/(x - 1), x, 1)  # 2
```

**重要：** 在奇点处使用 `limit()` 而非 `subs()`，因为无穷大对象无法可靠追踪增长速率。

### 级数展开

```python
from sympy import series, sin, exp, cos

# 泰勒级数展开
expr = sin(x)
expr.series(x, 0, 6)  # x - x**3/6 + x**5/120 + O(x**6)

# 定点展开
exp(x).series(x, 1, 4)  # 在 x=1 处展开

# 移除 O() 项
series(exp(x), x, 0, 4).removeO()  # 1 + x + x**2/2 + x**3/6
```

### 有限差分（数值导数）

```python
from sympy import Function, differentiate_finite
f = Function('f')

# 有限差分近似导数
differentiate_finite(f(x), x)
f(x).as_finite_difference()
```

## 方程求解

### 代数方程 - solveset

**主函数：** `solveset(equation, variable, domain)`

```python
from sympy import solveset, Eq, S

# 基础求解（默认 equation = 0）
solveset(x**2 - 1, x)  # {-1, 1}
solveset(x**2 + 1, x)  # {-I, I} (复数解)

# 显式方程
solveset(Eq(x**2, 4), x)  # {-2, 2}

# 指定定义域
solveset(x**2 - 1, x, domain=S.Reals)  # {-1, 1}
solveset(x**2 + 1, x, domain=S.Reals)  # EmptySet (无实数解)
```

**返回类型：** 有限集、区间或像集

### 方程组

**线性系统 - linsolve：**
```python
from sympy import linsolve, Matrix

# 方程形式
linsolve([x + y - 2, x - y], x, y)  # {(1, 1)}

# 增广矩阵形式
linsolve(Matrix([[1, 1, 2], [1, -1, 0]]), x, y)

# A*x = b 形式
A = Matrix([[1, 1], [1, -1]])
b = Matrix([2, 0])
linsolve((A, b), x, y)
```

**非线性系统 - nonlinsolve：**
```python
from sympy import nonlinsolve

nonlinsolve([x**2 + y - 2, x + y**2 - 3], x, y)
```

**注意：** 当前 nonlinsolve 不返回 LambertW 形式的解。

### 多项式求根

```python
from sympy import roots, solve

# 获取根及重数
roots(x**3 - 6*x**2 + 9*x, x)  # {0: 1, 3: 2}
# 表示 x=0 (重数1), x=3 (重数2)
```

### 通用求解器 - solve

超越方程求解的灵活方案：
```python
from sympy import solve, exp, log

solve(exp(x) - 3, x)     # [log(3)]
solve(x**2 - 4, x)       # [-2, 2]
solve([x + y - 1, x - y + 1], [x, y])  # {x: 0, y: 1}
```

### 微分方程 - dsolve

```python
from sympy import Function, dsolve, Derivative, Eq

# 定义函数
f = symbols('f', cls=Function)

# 解常微分方程
dsolve(Derivative(f(x), x) - f(x), f(x))
# 返回：Eq(f(x), C1*exp(x))

# 带初始条件
dsolve(Derivative(f(x), x) - f(x), f(x), ics={f(0): 1})
# 返回：Eq(f(x), exp(x))

# 二阶常微分方程
dsolve(Derivative(f(x), x, 2) + f(x), f(x))
# 返回：Eq(f(x), C1*sin(x) + C2*cos(x))
```

## 常用模式与最佳实践

### 模式1：增量构建复杂表达式
```python
from sympy import *
x, y = symbols('x y')

# 逐步构建
expr = x**2
expr = expr + 2*x + 1
expr = simplify(expr)
```

### 模式2：使用假设条件
```python
# 定义带物理约束的符号
x = symbols('x', positive=True, real=True)
y = symbols('y', real=True)

# SymPy 可利用假设简化
sqrt(x**2)  # 返回 x (非 Abs(x))，因 positive 假设
```

### 模式3：转换为数值函数
```python
from sympy import lambdify
import numpy as np

expr = x**2 + 2*x + 1
f = lambdify(x, expr, 'numpy')

# 可配合 numpy 数组使用
x_vals = np.linspace(0, 10, 100)
y_vals = f(x_vals)
```

### 模式4：美观打印
```python
from sympy import init_printing, pprint
init_printing()  # 在终端/笔记本启用美观打印

expr = Integral(sqrt(1/x), x)
pprint(expr)  # 显示格式化输出
```
