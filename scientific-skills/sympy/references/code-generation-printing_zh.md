# SymPy 代码生成与打印

本文档涵盖 SymPy 在生成多种语言可执行代码、将表达式转换为不同输出格式以及自定义打印行为方面的功能。

## 代码生成

### 转换为 NumPy 函数

```python
from sympy import symbols, sin, cos, lambdify
import numpy as np

x, y = symbols('x y')
expr = sin(x) + cos(y)

# 创建 NumPy 函数
f = lambdify((x, y), expr, 'numpy')

# 与 NumPy 数组配合使用
x_vals = np.linspace(0, 2*np.pi, 100)
y_vals = np.linspace(0, 2*np.pi, 100)
result = f(x_vals, y_vals)
```

### Lambdify 选项

```python
from sympy import lambdify, exp, sqrt

# 不同后端
f_numpy = lambdify(x, expr, 'numpy')      # NumPy
f_scipy = lambdify(x, expr, 'scipy')      # SciPy
f_mpmath = lambdify(x, expr, 'mpmath')    # mpmath (任意精度)
f_math = lambdify(x, expr, 'math')        # Python math 模块

# 自定义函数映射
custom_funcs = {'sin': lambda x: x}  # 用恒等函数替换 sin
f = lambdify(x, sin(x), modules=[custom_funcs, 'numpy'])

# 多个表达式
exprs = [x**2, x**3, x**4]
f = lambdify(x, exprs, 'numpy')
# 返回结果元组
```

### 生成 C/C++ 代码

```python
from sympy.utilities.codegen import codegen
from sympy import symbols

x, y = symbols('x y')
expr = x**2 + y**2

# 生成 C 代码
[(c_name, c_code), (h_name, h_header)] = codegen(
    ('distance_squared', expr),
    'C',
    header=False,
    empty=False
)

print(c_code)
# 输出有效的 C 函数
```

### 生成 Fortran 代码

```python
from sympy.utilities.codegen import codegen

[(f_name, f_code), (h_name, h_interface)] = codegen(
    ('my_function', expr),
    'F95',  # Fortran 95
    header=False
)

print(f_code)
```

### 高级代码生成

```python
from sympy.utilities.codegen import CCodeGen, make_routine
from sympy import MatrixSymbol, Matrix

# 矩阵运算
A = MatrixSymbol('A', 3, 3)
expr = A + A.T

# 创建例程
routine = make_routine('matrix_sum', expr)

# 生成代码
gen = CCodeGen()
code = gen.write([routine], prefix='my_module')
```

### 代码打印机

```python
from sympy.printing.c import C99CodePrinter, C89CodePrinter
from sympy.printing.fortran import FCodePrinter
from sympy.printing.cxx import CXX11CodePrinter

# C 代码
c_printer = C99CodePrinter()
c_code = c_printer.doprint(expr)

# Fortran 代码
f_printer = FCodePrinter()
f_code = f_printer.doprint(expr)

# C++ 代码
cxx_printer = CXX11CodePrinter()
cxx_code = cxx_printer.doprint(expr)
```

## 打印与输出格式

### 美观打印

```python
from sympy import init_printing, pprint, pretty, symbols
from sympy import Integral, sqrt, pi

# 初始化美观打印（适用于 Jupyter 笔记本和终端）
init_printing()

x = symbols('x')
expr = Integral(sqrt(1/x), (x, 0, pi))

# 在终端美观打印
pprint(expr)
#   π
#   ⌠
#   ⎮   1
#   ⎮  ───  dx
#   ⎮  √x
#   ⌡
#   0

# 获取美观字符串
s = pretty(expr)
print(s)
```

### LaTeX 输出

```python
from sympy import latex, symbols, Integral, sin, sqrt

x, y = symbols('x y')
expr = Integral(sin(x)**2, (x, 0, pi))

# 转换为 LaTeX
latex_str = latex(expr)
print(latex_str)
# \int\limits_{0}^{\pi} \sin^{2}{\left(x \right)}\, dx

# 自定义 LaTeX 格式
latex_str = latex(expr, mode='equation')  # 包裹在方程环境中
latex_str = latex(expr, mode='inline')    # 行内数学公式

# 矩阵处理
from sympy import Matrix
M = Matrix([[1, 2], [3, 4]])
latex(M)  # \left[\begin{matrix}1 & 2\\3 & 4\end{matrix}\right]
```

### MathML 输出

```python
from sympy.printing.mathml import mathml, print_mathml
from sympy import sin, pi

expr = sin(pi/4)

# 内容 MathML
mathml_str = mathml(expr)

# 呈现 MathML
mathml_str = mathml(expr, printer='presentation')

# 打印到控制台
print_mathml(expr)
```

### 字符串表示

```python
from sympy import symbols, sin, pi, srepr, sstr

x = symbols('x')
expr = sin(x)**2

# 标准字符串（Python 中显示的形式）
str(expr)  # 'sin(x)**2'

# 字符串表示（更美观）
sstr(expr)  # 'sin(x)**2'

# 可重现的表示
srepr(expr)  # "Pow(sin(Symbol('x')), Integer(2))"
# 可通过 eval() 重建表达式
```

### 自定义打印

```python
from sympy.printing.str import StrPrinter

class MyPrinter(StrPrinter):
    def _print_Symbol(self, expr):
        return f"<{expr.name}>"

    def _print_Add(self, expr):
        return " PLUS ".join(self._print(arg) for arg in expr.args)

printer = MyPrinter()
x, y = symbols('x y')
print(printer.doprint(x + y))  # "<x> PLUS <y>"
```

## Python 代码生成

### autowrap - 编译与导入

```python
from sympy.utilities.autowrap import autowrap
from sympy import symbols

x, y = symbols('x y')
expr = x**2 + y**2

# 自动编译 C 代码并创建 Python 包装器
f = autowrap(expr, backend='cython')
# 或使用 backend='f2py' 生成 Fortran 代码

# 像常规函数一样使用
result = f(3, 4)  # 25
```

### ufuncify - 创建 NumPy 通用函数

```python
from sympy.utilities.autowrap import ufuncify
import numpy as np

x, y = symbols('x y')
expr = x**2 + y**2

# 创建通用函数
f = ufuncify((x, y), expr)

# 支持 NumPy 广播
x_arr = np.array([1, 2, 3])
y_arr = np.array([4, 5, 6])
result = f(x_arr, y_arr)  # [17, 29, 45]
```

## 表达式树操作

### 遍历表达式树

```python
from sympy import symbols, sin, cos, preorder_traversal, postorder_traversal

x, y = symbols('x y')
expr = sin(x) + cos(y)

# 前序遍历（父节点先于子节点）
for arg in preorder_traversal(expr):
    print(arg)

# 后序遍历（子节点先于父节点）
for arg in postorder_traversal(expr):
    print(arg)

# 获取所有子表达式
subexprs = list(preorder_traversal(expr))
```

### 表达式树中的替换

```python
from sympy import Wild, symbols, sin, cos

x, y = symbols('x y')
a = Wild('a')

expr = sin(x) + cos(y)

# 模式匹配与替换
new_expr = expr.replace(sin(a), a**2)  # sin(x) -> x**2
```

## Jupyter 笔记本集成

### 数学公式显示

```python
from sympy import init_printing, display
from IPython.display import display as ipy_display

# 为 Jupyter 初始化打印
init_printing(use_latex='mathjax')  # 或 'png', 'svg'

# 美观地显示表达式
expr = Integral(sin(x)**2, x)
display(expr)  # 在笔记本中渲染为 LaTeX

# 多输出显示
ipy_display(expr1, expr2, expr3)
```

### 交互式控件

```python
from sympy import symbols, sin
from IPython.display import display
from ipywidgets import interact, FloatSlider
import matplotlib.pyplot as plt
import numpy as np

x = symbols('x')
expr = sin(x)

@interact(a=FloatSlider(min=0, max=10, step=0.1, value=1))
def plot_expr(a):
    f = lambdify(x, a * expr, 'numpy')
    x_vals = np.linspace(-np.pi, np.pi, 100)
    plt.plot(x_vals, f(x_vals))
    plt.show()
```

## 表示形式转换

### 字符串转 SymPy

```python
from sympy.parsing.sympy_parser import parse_expr
from sympy import symbols

x, y = symbols('x y')

# 解析字符串为表达式
expr = parse_expr('x**2 + 2*x + 1')
expr = parse_expr('sin(x) + cos(y)')

# 使用转换规则
from sympy.parsing.sympy_parser import (
    standard_transformations,
    implicit_multiplication_application
)

transformations = standard_transformations + (implicit_multiplication_application,)
expr = parse_expr('2x', transformations=transformations)  # 将 '2x' 视为 2*x
```

### LaTeX 转 SymPy

```python
from sympy.parsing.latex import parse_latex

# 解析 LaTeX
expr = parse_latex(r'\frac{x^2}{y}')
# 返回: x**2/y

expr = parse_latex(r'\int_0^\pi \sin(x) dx')
```

### Mathematica 转 SymPy

```python
from sympy.parsing.mathematica import parse_mathematica

# 解析 Mathematica 代码
expr = parse_mathematica('Sin[x]^2 + Cos[y]^2')
# 返回 SymPy 表达式
```

## 结果导出

### 导出到文件

```python
from sympy import symbols, sin
import json

x = symbols('x')
expr = sin(x)**2

# 导出为 LaTeX 文件
with open('output.tex', 'w') as f:
    f.write(latex(expr))

# 导出为字符串
with open('output.txt', 'w') as f:
    f.write(str(expr))

# 导出为 Python 代码
with open('output.py', 'w') as f:
    f.write(f"from numpy import sin\n")
    f.write(f"def f(x):\n")
    f.write(f"    return {lambdify(x, expr, 'numpy')}\n")
```

### 序列化 SymPy 对象

```python
import pickle
from sympy import symbols, sin

x = symbols('x')
expr = sin(x)**2 + x

# 保存
with open('expr.pkl', 'wb') as f:
    pickle.dump(expr, f)

# 加载
with open('expr.pkl', 'rb') as f:
    loaded_expr = pickle.load(f)
```

## 数值计算与精度

### 高精度计算

```python
from sympy import symbols, pi, sqrt, E, exp, sin
from mpmath import mp

x = symbols('x')

# 标准精度
pi.evalf()  # 3.14159265358979

# 高精度（1000 位）
pi.evalf(1000)

# 使用 mpmath 设置全局精度
mp.dps = 50  # 50 位小数
expr = exp(pi * sqrt(163))
float(expr.evalf())

# 表达式计算
result = (sqrt(2) + sqrt(3)).evalf(100)
```

### 数值替换

```python
from sympy import symbols, sin, cos

x, y = symbols('x y')
expr = sin(x) + cos(y)

# 数值计算
result = expr.evalf(subs={x: 1.5, y: 2.3})

# 带单位的计算
from sympy.physics.units import meter, second
distance = 100 * meter
time = 10 * second
speed = distance / time
speed.evalf()
```

## 常用模式

### 模式 1：生成并执行代码

```python
from sympy import symbols, lambdify
import numpy as np

# 1. 定义符号表达式
x, y = symbols('x y')
expr = x**2 + y**2

# 2. 生成函数
f = lambdify((x, y), expr, 'numpy')

# 3. 使用数值数据执行
data_x = np.random.rand(1000)
data_y = np.random.rand(1000)
results = f(data_x, data_y)
```

### 模式 2：创建 LaTeX 文档

```python
from sympy import symbols, Integral, latex
from sympy.abc import x

# 定义数学内容
expr = Integral(x**2, (x, 0, 1))
result = expr.doit()

# 生成 LaTeX 文档
latex_doc = f"""
\\documentclass{{article}}
\\usepackage{{amsmath}}
\\begin{{document}}

计算积分：
\\begin{{equation}}
{latex(expr)} = {latex(result)}
\\end{{equation}}

\\end{{document}}
"""

with open('document.tex', 'w') as f:
    f.write(latex_doc)
```

### 模式 3：交互式计算

```python
from sympy import symbols, simplify, expand
from sympy.parsing.sympy_parser import parse_expr

x, y = symbols('x y')

# 交互式输入
user_input = input("输入表达式: ")
expr = parse_expr(user_input)

# 处理
simplified = simplify(expr)
expanded = expand(expr)

# 显示
print(f"简化形式: {simplified}")
print(f"展开形式: {expanded}")
print(f"LaTeX: {latex(expr)}")
```

### 模式 4：批量代码生成

```python
from sympy import symbols, lambdify
from sympy.utilities.codegen import codegen

# 多个函数
x = symbols('x')
functions = {
    'f1': x**2,
    'f2': x**3,
    'f3': x**4
}

# 为所有函数生成 C 代码
for name, expr in functions.items():
    [(c_name, c_code), _] = codegen((name, expr), 'C')
    with open(f'{name}.c', 'w') as f:
        f.write(c_code)
```

### 模式 5：性能优化

```python
from sympy import symbols, sin, cos, cse
import numpy as np

x, y = symbols('x y')

# 包含重复子表达式的复杂表达式
expr = sin(x + y)**2 + cos(x + y)**2 + sin(x + y)

# 公共子表达式消除
replacements, reduced = cse(expr)
# replacements: [(x0, sin(x + y)), (x1, cos(x + y))]
# reduced: [x0**2 + x1**2 + x0]

# 生成优化代码
for var, subexpr in replacements:
    print(f"{var} = {subexpr}")
print(f"result = {reduced[0]}")
```

## 重要注意事项

1. **NumPy 兼容性**：使用 `lambdify` 配合 NumPy 时，确保表达式使用 NumPy 中可用的函数。

2. **性能**：进行数值计算时，始终使用 `lambdify` 或代码生成，避免在循环中使用 `subs()` + `evalf()`。

3. **精度**：需要任意精度
