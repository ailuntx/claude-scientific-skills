# SymPy 高级主题

本文档涵盖 SymPy 的高级数学功能，包括几何、数论、组合数学、逻辑与集合、统计学、多项式和特殊函数。

## 几何

### 二维几何

```python
from sympy.geometry import Point, Line, Circle, Triangle, Polygon

# 点
p1 = Point(0, 0)
p2 = Point(1, 1)
p3 = Point(1, 0)

# 点间距离
dist = p1.distance(p2)

# 直线
line = Line(p1, p2)
line_from_eq = Line(Point(0, 0), slope=2)

# 直线属性
line.slope       # 斜率
line.equation()  # 直线方程
line.length      # oo (直线无限长)

# 线段
from sympy.geometry import Segment
seg = Segment(p1, p2)
seg.length       # 有限长度
seg.midpoint     # 中点

# 交点
line2 = Line(Point(0, 1), Point(1, 0))
intersection = line.intersection(line2)  # [Point(1/2, 1/2)]

# 圆
circle = Circle(Point(0, 0), 5)  # 圆心, 半径
circle.area           # 25*pi
circle.circumference  # 10*pi

# 三角形
tri = Triangle(p1, p2, p3)
tri.area       # 面积
tri.perimeter  # 周长
tri.angles     # 角度字典
tri.vertices   # 顶点元组

# 多边形
poly = Polygon(Point(0, 0), Point(1, 0), Point(1, 1), Point(0, 1))
poly.area
poly.perimeter
poly.vertices
```

### 几何查询

```python
# 检查点是否在直线/曲线上
point = Point(0.5, 0.5)
line.contains(point)

# 检查平行/垂直
line1 = Line(Point(0, 0), Point(1, 1))
line2 = Line(Point(0, 1), Point(1, 2))
line1.is_parallel(line2)  # True
line1.is_perpendicular(line2)  # False

# 切线
from sympy.geometry import Circle, Point
circle = Circle(Point(0, 0), 5)
point = Point(5, 0)
tangents = circle.tangent_lines(point)
```

### 三维几何

```python
from sympy.geometry import Point3D, Line3D, Plane

# 三维点
p1 = Point3D(0, 0, 0)
p2 = Point3D(1, 1, 1)
p3 = Point3D(1, 0, 0)

# 三维直线
line = Line3D(p1, p2)

# 平面
plane = Plane(p1, p2, p3)  # 由三点定义
plane = Plane(Point3D(0, 0, 0), normal_vector=(1, 0, 0))  # 由点和法向量定义

# 平面方程
plane.equation()

# 点到平面距离
point = Point3D(2, 3, 4)
dist = plane.distance(point)

# 平面与直线交点
intersection = plane.intersection(line)
```

### 曲线与椭圆

```python
from sympy.geometry import Ellipse, Curve
from sympy import sin, cos, pi

# 椭圆
ellipse = Ellipse(Point(0, 0), hradius=3, vradius=2)
ellipse.area          # 6*pi
ellipse.eccentricity  # 离心率

# 参数曲线
from sympy.abc import t
curve = Curve((cos(t), sin(t)), (t, 0, 2*pi))  # 圆
```

## 数论

### 质数

```python
from sympy.ntheory import isprime, primerange, prime, nextprime, prevprime

# 检查质数
isprime(7)    # True
isprime(10)   # False

# 生成范围内的质数
list(primerange(10, 50))  # [11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]

# 第n个质数
prime(10)     # 29 (第10个质数)

# 下一个/上一个质数
nextprime(10)  # 11
prevprime(10)  # 7
```

### 质因数分解

```python
from sympy import factorint, primefactors, divisors

# 质因数分解
factorint(60)  # {2: 2, 3: 1, 5: 1} 表示 2^2 * 3^1 * 5^1

# 质因数列表
primefactors(60)  # [2, 3, 5]

# 所有约数
divisors(60)  # [1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30, 60]
```

### 最大公约数与最小公倍数

```python
from sympy import gcd, lcm, igcd, ilcm

# 最大公约数
gcd(60, 48)   # 12
igcd(60, 48)  # 12 (整数版本)

# 最小公倍数
lcm(60, 48)   # 240
ilcm(60, 48)  # 240 (整数版本)

# 多参数
gcd(60, 48, 36)  # 12
```

### 模运算

```python
from sympy.ntheory import mod_inverse, totient, is_primitive_root

# 模逆元 (求x使得 a*x ≡ 1 (mod m))
mod_inverse(3, 7)  # 5 (因为 3*5 = 15 ≡ 1 (mod 7))

# 欧拉函数
totient(10)  # 4 (小于10且与10互质的数: 1,3,7,9)

# 原根
is_primitive_root(2, 5)  # True
```

### 丢番图方程

```python
from sympy.solvers.diophantine import diophantine
from sympy.abc import x, y, z

# 线性丢番图方程: ax + by = c
diophantine(3*x + 4*y - 5)  # {(4*t_0 - 5, -3*t_0 + 5)}

# 二次型
diophantine(x**2 + y**2 - 25)  # 毕达哥拉斯型方程

# 更复杂的方程
diophantine(x**2 - 4*x*y + 8*y**2 - 3*x + 7*y - 5)
```

### 连分数

```python
from sympy import nsimplify, continued_fraction_iterator
from sympy import Rational, pi

# 转换为连分数
cf = continued_fraction_iterator(Rational(415, 93))
list(cf)  # [4, 2, 6, 7]

# 近似无理数
cf_pi = continued_fraction_iterator(pi.evalf(20))
```

## 组合数学

### 排列与组合

```python
from sympy import factorial, binomial, factorial2
from sympy.functions.combinatorial.numbers import nC, nP

# 阶乘
factorial(5)  # 120

# 二项式系数 (n选k)
binomial(5, 2)  # 10

# 排列 nPk = n!/(n-k)!
nP(5, 2)  # 20

# 组合 nCk = n!/(k!(n-k)!)
nC(5, 2)  # 10

# 双阶乘 n!!
factorial2(5)  # 15 (5*3*1)
factorial2(6)  # 48 (6*4*2)
```

### 置换对象

```python
from sympy.combinatorics import Permutation

# 创建置换 (循环表示法)
p = Permutation([1, 2, 0, 3])  # 映射 0->1, 1->2, 2->0, 3->3
p = Permutation(0, 1, 2)(3)    # 循环表示: (0 1 2)(3)

# 置换操作
p.order()       # 置换阶数
p.is_even       # 是否为偶置换
p.inversions()  # 逆序数

# 复合置换
q = Permutation([2, 0, 1, 3])
r = p * q       # 复合运算
```

### 整数划分

```python
from sympy.utilities.iterables import partitions
from sympy.functions.combinatorial.numbers import partition

# 整数划分数
partition(5)  # 7 (5, 4+1, 3+2, 3+1+1, 2+2+1, 2+1+1+1, 1+1+1+1+1)

# 生成所有划分
list(partitions(4))
# {4: 1}, {3: 1, 1: 1}, {2: 2}, {2: 1, 1: 2}, {1: 4}
```

### 卡特兰数与斐波那契数

```python
from sympy import catalan, fibonacci, lucas

# 卡特兰数
catalan(5)  # 42

# 斐波那契数
fibonacci(10)  # 55
lucas(10)      # 123 (卢卡斯数)
```

### 群论

```python
from sympy.combinatorics import PermutationGroup, Permutation

# 创建置换群
p1 = Permutation([1, 0, 2])
p2 = Permutation([0, 2, 1])
G = PermutationGroup(p1, p2)

# 群属性
G.order()        # 群阶
G.is_abelian     # 是否阿贝尔群
G.is_cyclic()    # 是否循环群
G.elements       # 所有群元素
```

## 逻辑与集合

### 布尔逻辑

```python
from sympy import symbols, And, Or, Not, Xor, Implies, Equivalent
from sympy.logic.boolalg import truth_table, simplify_logic

# 定义布尔变量
x, y, z = symbols('x y z', bool=True)

# 逻辑运算
expr = And(x, Or(y, Not(z)))
expr = Implies(x, y)  # x -> y
expr = Equivalent(x, y)  # x <-> y
expr = Xor(x, y)  # 异或

# 化简
expr = (x & y) | (x & ~y)
simplified = simplify_logic(expr)  # 返回 x

# 真值表
expr = Implies(x, y)
print(truth_table(expr, [x, y]))
```

### 集合

```python
from sympy import FiniteSet, Interval, Union, Intersection, Complement
from sympy import S  # 特殊集合

# 有限集
A = FiniteSet(1, 2, 3, 4)
B = FiniteSet(3, 4, 5, 6)

# 集合运算
union = Union(A, B)              # {1, 2, 3, 4, 5, 6}
intersection = Intersection(A, B)  # {3, 4}
difference = Complement(A, B)     # {1, 2}

# 区间
I = Interval(0, 1)              # [0, 1]
I_open = Interval.open(0, 1)    # (0, 1)
I_lopen = Interval.Lopen(0, 1)  # (0, 1]
I_ropen = Interval.Ropen(0, 1)  # [0, 1)

# 特殊集合
S.Reals        # 所有实数
S.Integers     # 所有整数
S.Naturals     # 自然数
S.EmptySet     # 空集
S.Complexes    # 复数

# 集合成员
3 in A  # True
7 in A  # False

# 子集与超集
A.is_subset(B)    # False
A.is_superset(B)  # False
```

### 集合论运算

```python
from sympy import ImageSet, Lambda
from sympy.abc import x

# 像集 (函数值集合)
squares = ImageSet(Lambda(x, x**2), S.Integers)
# {x^2 | x ∈ ℤ}

# 幂集
from sympy.sets import FiniteSet
A = FiniteSet(1, 2, 3)
# 注意: SymPy 不直接支持幂集，但可生成
```

## 多项式

### 多项式操作

```python
from sympy import Poly, symbols, factor, expand, roots
x, y = symbols('x y')

# 创建多项式
p = Poly(x**2 + 2*x + 1, x)

# 多项式属性
p.degree()       # 次数
p.coeffs()       # 系数列表
p.as_expr()      # 转回表达式

# 算术运算
p1 = Poly(x**2 + 1, x)
p2 = Poly(x + 1, x)
p3 = p1 + p2
p4 = p1 * p2
q, r = div(p1, p2)  # 商和余式
```

### 多项式根

```python
from sympy import roots, real_roots, count_roots

p = Poly(x**3 - 6*x**2 + 11*x - 6, x)

# 所有根
r = roots(p)  # {1: 1, 2: 1, 3: 1}

# 仅实根
r = real_roots(p)

# 区间内根计数
count_roots(p, a, b)  # [a, b] 区间内的根数
```

### 多项式的最大公约数与因式分解

```python
from sympy import gcd, lcm, factor, factor_list

p1 = Poly(x**2 - 1, x)
p2 = Poly(x**2 - 2*x + 1, x)

# 最大公约数与最小公倍数
g = gcd(p1, p2)
l = lcm(p1, p2

```markdown
X = Normal('X', 0, 1)
Y = Normal('Y', 0, 1)

# 联合概率
P((X > 0) & (Y > 0))  # 1/4

# 协方差
from sympy.stats import covariance
covariance(X, Y)  # 0 (独立)
```

## 特殊函数

### 常用特殊函数

```python
from sympy import (
    gamma,      # Gamma函数
    beta,       # Beta函数
    erf,        # 误差函数
    besselj,    # 第一类贝塞尔函数
    bessely,    # 第二类贝塞尔函数
    hermite,    # 埃尔米特多项式
    legendre,   # 勒让德多项式
    laguerre,   # 拉盖尔多项式
    chebyshevt, # 切比雪夫多项式（第一类）
    zeta        # 黎曼zeta函数
)

# Gamma函数
gamma(5)  # 24 (等价于4!)
gamma(1/2)  # sqrt(pi)

# 贝塞尔函数
besselj(0, x)  # J_0(x)
bessely(1, x)  # Y_1(x)

# 正交多项式
hermite(3, x)    # 8*x**3 - 12*x
legendre(2, x)   # (3*x**2 - 1)/2
laguerre(2, x)   # x**2/2 - 2*x + 1
chebyshevt(3, x) # 4*x**3 - 3*x
```

### 超几何函数

```python
from sympy import hyper, meijerg

# 超几何函数
hyper([1, 2], [3], x)

# Meijer G函数
meijerg([[1, 1], []], [[1], [0]], x)
```

## 常用模式

### 模式1：符号几何问题

```python
from sympy.geometry import Point, Triangle
from sympy import symbols

# 定义符号三角形
a, b = symbols('a b', positive=True)
tri = Triangle(Point(0, 0), Point(a, 0), Point(0, b))

# 符号化计算属性
area = tri.area  # a*b/2
perimeter = tri.perimeter  # a + b + sqrt(a**2 + b**2)
```

### 模式2：数论计算

```python
from sympy.ntheory import factorint, totient, isprime

# 因式分解与分析
n = 12345
factors = factorint(n)
phi = totient(n)
is_prime = isprime(n)
```

### 模式3：组合生成

```python
from sympy.utilities.iterables import multiset_permutations, combinations

# 生成全排列
perms = list(multiset_permutations([1, 2, 3]))

# 生成组合
combs = list(combinations([1, 2, 3, 4], 2))
```

### 模式4：概率计算

```python
from sympy.stats import Normal, P, E, variance

X = Normal('X', mu, sigma)

# 计算统计量
mean = E(X)
var = variance(X)
prob = P(X > a)
```

## 重要说明

1. **假设条件**：许多操作受益于符号假设（如 `positive=True`, `integer=True`）。

2. **符号与数值**：这些操作均为符号计算。需使用 `evalf()` 获取数值结果。

3. **性能**：复杂符号运算可能较慢。大规模计算建议考虑数值方法。

4. **精确算术**：SymPy 保持精确表示（如使用 `sqrt(2)` 而非 `1.414...`）。
```
