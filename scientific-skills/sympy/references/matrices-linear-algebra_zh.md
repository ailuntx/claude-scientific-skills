# SymPy 矩阵与线性代数

本文档涵盖 SymPy 的矩阵操作、线性代数功能以及线性方程组的求解。

## 矩阵创建

### 基础矩阵构建

```python
from sympy import Matrix, eye, zeros, ones, diag

# 从行列表创建
M = Matrix([[1, 2], [3, 4]])
M = Matrix([
    [1, 2, 3],
    [4, 5, 6]
])

# 列向量
v = Matrix([1, 2, 3])

# 行向量
v = Matrix([[1, 2, 3]])
```

### 特殊矩阵

```python
# 单位矩阵
I = eye(3)  # 3x3单位矩阵
# [[1, 0, 0],
#  [0, 1, 0],
#  [0, 0, 1]]

# 零矩阵
Z = zeros(2, 3)  # 2行3列的零矩阵

# 全一矩阵
O = ones(3, 2)   # 3行2列的全一矩阵

# 对角矩阵
D = diag(1, 2, 3)
# [[1, 0, 0],
#  [0, 2, 0],
#  [0, 0, 3]]

# 块对角矩阵
from sympy import BlockDiagMatrix
A = Matrix([[1, 2], [3, 4]])
B = Matrix([[5, 6], [7, 8]])
BD = BlockDiagMatrix(A, B)
```

## 矩阵属性与访问

### 形状与维度

```python
M = Matrix([[1, 2, 3], [4, 5, 6]])

M.shape  # (2, 3) - 返回元组(行数, 列数)
M.rows   # 2
M.cols   # 3
```

### 元素访问

```python
M = Matrix([[1, 2, 3], [4, 5, 6]])

# 单个元素
M[0, 0]  # 1 (从0开始索引)
M[1, 2]  # 6

# 行访问
M[0, :]   # Matrix([[1, 2, 3]])
M.row(0)  # 同上

# 列访问
M[:, 1]   # Matrix([[2], [5]])
M.col(1)  # 同上

# 切片
M[0:2, 0:2]  # 左上角2x2子矩阵
```

### 修改操作

```python
M = Matrix([[1, 2], [3, 4]])

# 插入行
M = M.row_insert(1, Matrix([[5, 6]]))
# [[1, 2],
#  [5, 6],
#  [3, 4]]

# 插入列
M = M.col_insert(1, Matrix([7, 8]))

# 删除行
M = M.row_del(0)

# 删除列
M = M.col_del(1)
```

## 基础矩阵运算

### 算术运算

```python
from sympy import Matrix

A = Matrix([[1, 2], [3, 4]])
B = Matrix([[5, 6], [7, 8]])

# 加法
C = A + B

# 减法
C = A - B

# 标量乘法
C = 2 * A

# 矩阵乘法
C = A * B

# 逐元素乘法（Hadamard积）
C = A.multiply_elementwise(B)

# 幂运算
C = A**2  # 等价于 A * A
C = A**3  # A * A * A
```

### 转置与共轭

```python
M = Matrix([[1, 2], [3, 4]])

# 转置
M.T
# [[1, 3],
#  [2, 4]]

# 共轭（针对复矩阵）
M.conjugate()

# 共轭转置（Hermitian转置）
M.H  # 等价于 M.conjugate().T
```

### 逆矩阵

```python
M = Matrix([[1, 2], [3, 4]])

# 逆矩阵
M_inv = M**-1
M_inv = M.inv()

# 验证
M * M_inv  # 返回单位矩阵

# 检查是否可逆
M.is_invertible()  # True 或 False
```

## 高级线性代数

### 行列式

```python
M = Matrix([[1, 2], [3, 4]])
M.det()  # -2

# 符号矩阵示例
from sympy import symbols
a, b, c, d = symbols('a b c d')
M = Matrix([[a, b], [c, d]])
M.det()  # a*d - b*c
```

### 迹

```python
M = Matrix([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
M.trace()  # 1 + 5 + 9 = 15
```

### 行阶梯形式

```python
M = Matrix([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# 简化行阶梯形式
rref_M, pivot_cols = M.rref()
# rref_M 是RREF矩阵
# pivot_cols 是主元列索引元组
```

### 秩

```python
M = Matrix([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
M.rank()  # 2 (该矩阵秩亏)
```

### 零空间与列空间

```python
M = Matrix([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# 零空间（核）
null = M.nullspace()
# 返回零空间基向量列表

# 列空间
col = M.columnspace()
# 返回列空间基向量列表

# 行空间
row = M.rowspace()
# 返回行空间基向量列表
```

### 正交化

```python
# Gram-Schmidt正交化
vectors = [Matrix([1, 2, 3]), Matrix([4, 5, 6])]
ortho = Matrix.orthogonalize(*vectors)

# 带归一化
ortho_norm = Matrix.orthogonalize(*vectors, normalize=True)
```

## 特征值与特征向量

### 计算特征值

```python
M = Matrix([[1, 2], [2, 1]])

# 带重数的特征值
eigenvals = M.eigenvals()
# 返回字典：{特征值: 重数}
# 示例：{3: 1, -1: 1}

# 仅特征值列表
eigs = list(M.eigenvals().keys())
```

### 计算特征向量

```python
M = Matrix([[1, 2], [2, 1]])

# 带特征值的特征向量
eigenvects = M.eigenvects()
# 返回元组列表：(特征值, 重数, [特征向量])
# 示例：[(3, 1, [Matrix([1, 1])]), (-1, 1, [Matrix([1, -1])])]

# 访问单个特征向量
for eigenval, multiplicity, eigenvecs in M.eigenvects():
    print(f"特征值: {eigenval}")
    print(f"特征向量: {eigenvecs}")
```

### 对角化

```python
M = Matrix([[1, 2], [2, 1]])

# 检查是否可对角化
M.is_diagonalizable()  # True 或 False

# 对角化 (M = P*D*P^-1)
P, D = M.diagonalize()
# P: 特征向量矩阵
# D: 特征值对角矩阵

# 验证
P * D * P**-1 == M  # True
```

### 特征多项式

```python
from sympy import symbols
lam = symbols('lambda')

M = Matrix([[1, 2], [2, 1]])
charpoly = M.charpoly(lam)
# 返回特征多项式
```

### Jordan标准形

```python
M = Matrix([[2, 1, 0], [0, 2, 1], [0, 0, 2]])

# Jordan形式（针对不可对角化矩阵）
P, J = M.jordan_form()
# J 是Jordan标准形
# P 是变换矩阵
```

## 矩阵分解

### LU分解

```python
M = Matrix([[1, 2, 3], [4, 5, 6], [7, 8, 10]])

# LU分解
L, U, perm = M.LUdecomposition()
# L: 下三角矩阵
# U: 上三角矩阵
# perm: 置换索引
```

### QR分解

```python
M = Matrix([[1, 2], [3, 4], [5, 6]])

# QR分解
Q, R = M.QRdecomposition()
# Q: 正交矩阵
# R: 上三角矩阵
```

### Cholesky分解

```python
# 针对正定对称矩阵
M = Matrix([[4, 2], [2, 3]])

L = M.cholesky()
# L 是下三角矩阵，满足 M = L*L.T
```

### 奇异值分解 (SVD)

```python
M = Matrix([[1, 2], [3, 4], [5, 6]])

# SVD（注意：可能需要数值计算）
U, S, V = M.singular_value_decomposition()
# M = U * S * V
```

## 求解线性系统

### 使用矩阵方程

```python
# 求解 Ax = b
A = Matrix([[1, 2], [3, 4]])
b = Matrix([5, 6])

# 解向量
x = A.solve(b)  # 或 A**-1 * b

# 最小二乘法（针对超定系统）
x = A.solve_least_squares(b)
```

### 使用 linsolve

```python
from sympy import linsolve, symbols

x, y = symbols('x y')

# 方法1：方程列表
eqs = [x + y - 5, 2*x - y - 1]
sol = linsolve(eqs, [x, y])
# {(2, 3)}

# 方法2：增广矩阵
M = Matrix([[1, 1, 5], [2, -1, 1]])
sol = linsolve(M, [x, y])

# 方法3：A*x = b 形式
A = Matrix([[1, 1], [2, -1]])
b = Matrix([5, 1])
sol = linsolve((A, b), [x, y])
```

### 欠定与超定系统

```python
# 欠定系统（无穷解）
A = Matrix([[1, 2, 3]])
b = Matrix([6])
sol = A.solve(b)  # 返回参数化解

# 超定系统（最小二乘解）
A = Matrix([[1, 2], [3, 4], [5, 6]])
b = Matrix([1, 2, 3])
sol = A.solve_least_squares(b)
```

## 符号矩阵

### 符号元素操作

```python
from sympy import symbols, Matrix

a, b, c, d = symbols('a b c d')
M = Matrix([[a, b], [c, d]])

# 所有操作均支持符号计算
M.det()  # a*d - b*c
M.inv()  # Matrix([[d/(a*d - b*c), -b/(a*d - b*c)], ...])
M.eigenvals()  # 符号特征值
```

### 矩阵函数

```python
from sympy import exp, sin, cos, Matrix

M = Matrix([[0, 1], [-1, 0]])

# 矩阵指数
exp(M)

# 三角函数
sin(M)
cos(M)
```

## 可变与不可变矩阵

```python
from sympy import Matrix, ImmutableMatrix

# 可变矩阵（默认）
M = Matrix([[1, 2], [3, 4]])
M[0, 0] = 5  # 允许操作

# 不可变矩阵（可用作字典键等）
I = ImmutableMatrix([[1, 2], [3, 4]])
# I[0, 0] = 5  # 错误：ImmutableMatrix不可修改
```

## 稀疏矩阵

针对含大量零元素的大型矩阵：

```python
from sympy import SparseMatrix

# 创建稀疏矩阵
S = SparseMatrix(1000, 1000, {(0, 0): 1, (100, 100): 2})
# 仅存储非零元素

# 稠密矩阵转稀疏矩阵
M = Matrix([[1, 0, 0], [0, 2, 0]])
S = SparseMatrix(M)
```

## 常见线性代数模式

### 模式1：求解多个b向量的Ax=b

```python
A = Matrix([[1, 2], [3, 4]])
A_inv = A.inv()

b1 = Matrix([5, 6])
b2 = Matrix([7, 8])

x1 = A_inv * b1
x2 = A_inv * b2
```

### 模式2：基变换

```python
# 将旧基下的向量转换到新基
old_basis = [Matrix([1, 0]), Matrix([0, 1])]
new_basis = [Matrix([1, 1]), Matrix([1, -1])]

# 基变换矩阵
P = Matrix.hstack(*new_basis)
P_inv = P.inv()

# 将向量v从旧基转换到新基
v = Matrix([3, 4])
v_new = P_inv * v
```

### 模式3：矩阵条件数

```python
# 估计条件数（最大奇异值与最小奇异值之比）
M = Matrix([[1, 2], [3, 4]])
eigenvals = M.eigenvals()
cond = max(eigenvals.keys()) / min(eigenvals.keys())
```

### 模式4：投影矩阵

```python
# 投影到A的列空间
A = Matrix([[1, 0], [0, 1], [1, 1]])
P = A * (A.T * A).inv() * A.T
# P 是投影到A列空间的投影矩阵
```

## 重要注意事项

1. **零值检测：** SymPy的符号零值检测可能影响精度。数值计算建议使用`evalf()`或数值计算库。

2. **性能：** 大型数值矩阵建议使用`lambdify`转换为NumPy或直接使用数值线性代数库。

3. **符号计算：** 含符号元素的大型矩阵运算可能计算代价高昂。

4. **假设条件：** 使用符号假设（如`real=True`, `positive=True`）可帮助SymPy正确简化矩阵表达式。
