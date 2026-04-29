# SymPy 物理与力学模块

本文档涵盖 SymPy 的物理模块，包括经典力学、量子力学、矢量分析、单位制、光学、连续介质力学和控制系统。

## 矢量分析

### 创建参考系与矢量

```python
from sympy.physics.vector import ReferenceFrame, dynamicsymbols

# 创建参考系
N = ReferenceFrame('N')  # 惯性参考系
B = ReferenceFrame('B')  # 物体参考系

# 创建矢量
v = 3*N.x + 4*N.y + 5*N.z

# 时变量
t = dynamicsymbols._t
x = dynamicsymbols('x')  # 时间函数
v = x.diff(t) * N.x  # 速度矢量
```

### 矢量运算

```python
from sympy.physics.vector import dot, cross

v1 = 3*N.x + 4*N.y
v2 = 1*N.x + 2*N.y + 3*N.z

# 点积
d = dot(v1, v2)

# 叉积
c = cross(v1, v2)

# 模长
mag = v1.magnitude()

# 归一化
v1_norm = v1.normalize()
```

### 参考系定向

```python
# 将B系相对于N系旋转
from sympy import symbols, cos, sin
theta = symbols('theta')

# 绕z轴简单旋转
B.orient(N, 'Axis', [theta, N.z])

# 方向余弦矩阵 (DCM)
dcm = N.dcm(B)

# B系在N系中的角速度
omega = B.ang_vel_in(N)
```

### 点与运动学

```python
from sympy.physics.vector import Point

# 创建点
O = Point('O')  # 原点
P = Point('P')

# 设置位置
P.set_pos(O, 3*N.x + 4*N.y)

# 设置速度
P.set_vel(N, 5*N.x + 2*N.y)

# 获取P点在N系中的速度
v = P.vel(N)

# 获取加速度
a = P.acc(N)
```

## 经典力学

### 拉格朗日力学

```python
from sympy import symbols, Function
from sympy.physics.mechanics import dynamicsymbols, LagrangesMethod

# 定义广义坐标
q = dynamicsymbols('q')
qd = dynamicsymbols('q', 1)  # q点（速度）

# 定义拉格朗日量 (L = T - V)
from sympy import Rational
m, g, l = symbols('m g l')
T = Rational(1, 2) * m * (l * qd)**2  # 动能
V = m * g * l * (1 - cos(q))           # 势能
L = T - V

# 应用拉格朗日方法
LM = LagrangesMethod(L, [q])
LM.form_lagranges_equations()
eqs = LM.rhs()  # 运动方程的右侧
```

### 凯恩方法

```python
from sympy.physics.mechanics import KanesMethod, ReferenceFrame, Point
from sympy.physics.vector import dynamicsymbols

# 定义系统
N = ReferenceFrame('N')
q = dynamicsymbols('q')
u = dynamicsymbols('u')  # 广义速度

# 创建凯恩方程
kd = [u - q.diff()]  # 运动微分方程
KM = KanesMethod(N, [q], [u], kd)

# 定义力与物体
# ... (定义质点、力等)
# KM.kanes_equations(bodies, loads)
```

### 系统物体与惯量

```python
from sympy.physics.mechanics import RigidBody, Inertia, Point, ReferenceFrame
from sympy import symbols

# 质量与惯量参数
m = symbols('m')
Ixx, Iyy, Izz = symbols('I_xx I_yy I_zz')

# 创建参考系与质心
A = ReferenceFrame('A')
P = Point('P')

# 定义惯量张量
I = Inertia(A, Ixx, Iyy, Izz)

# 创建刚体
body = RigidBody('Body', P, A, m, (I, P))
```

### 关节框架

```python
from sympy.physics.mechanics import Body, PinJoint, PrismaticJoint

# 创建物体
parent = Body('P')
child = Body('C')

# 创建旋转关节
pin = PinJoint('pin', parent, child)

# 创建滑动关节
slider = PrismaticJoint('slider', parent, child, axis=parent.frame.z)
```

### 线性化

```python
# 在平衡点附近线性化运动方程
operating_point = {q: 0, u: 0}  # 平衡点
A, B = KM.linearize(q_ind=[q], u_ind=[u],
                     A_and_B=True,
                     op_point=operating_point)
# A: 状态矩阵, B: 输入矩阵
```

## 量子力学

### 态与算符

```python
from sympy.physics.quantum import Ket, Bra, Operator, Dagger

# 定义态
psi = Ket('psi')
phi = Ket('phi')

# 左矢态
bra_psi = Bra('psi')

# 算符
A = Operator('A')
B = Operator('B')

# 厄米共轭
A_dag = Dagger(A)

# 内积
inner = bra_psi * psi
```

### 对易子与反对易子

```python
from sympy.physics.quantum import Commutator, AntiCommutator

# 对易子 [A, B] = AB - BA
comm = Commutator(A, B)
comm.doit()

# 反对易子 {A, B} = AB + BA
anti = AntiCommutator(A, B)
anti.doit()
```

### 量子谐振子

```python
from sympy.physics.quantum.qho_1d import RaisingOp, LoweringOp, NumberOp

# 产生与湮灭算符
a_dag = RaisingOp('a')  # 产生算符
a = LoweringOp('a')      # 湮灭算符
N = NumberOp('N')        # 粒子数算符

# 粒子数态
from sympy.physics.quantum.qho_1d import Ket as QHOKet
n = QHOKet('n')
```

### 自旋系统

```python
from sympy.physics.quantum.spin import (
    JzKet, JxKet, JyKet,  # 自旋态
    Jz, Jx, Jy,            # 自旋算符
    J2                     # 总角动量平方
)

# 自旋-1/2态
from sympy import Rational
psi = JzKet(Rational(1, 2), Rational(1, 2))  # |1/2, 1/2⟩

# 应用算符
result = Jz * psi
```

### 量子门

```python
from sympy.physics.quantum.gate import (
    H,      # Hadamard门
    X, Y, Z,  # Pauli门
    CNOT,    # 受控非门
    SWAP     # 交换门
)

# 对量子态应用门
from sympy.physics.quantum.qubit import Qubit
q = Qubit('01')
result = H(0) * q  # 对量子比特0应用Hadamard门
```

### 量子算法

```python
from sympy.physics.quantum.grover import grover_iteration, OracleGate

# Grover算法组件可用
# from sympy.physics.quantum.shor import <components>
# Shor算法组件可用
```

## 单位与量纲

### 单位操作

```python
from sympy.physics.units import (
    meter, kilogram, second,
    newton, joule, watt,
    convert_to
)

# 定义物理量
distance = 5 * meter
mass = 10 * kilogram
time = 2 * second

# 计算力
force = mass * distance / time**2

# 单位转换
force_in_newtons = convert_to(force, newton)
```

### 单位系统

```python
from sympy.physics.units import SI, gravitational_constant, speed_of_light

# SI单位制
print(SI._base_units)  # SI基本单位

# 物理常数
G = gravitational_constant
c = speed_of_light
```

### 自定义单位

```python
from sympy.physics.units import Quantity, meter, second

# 定义自定义单位
parsec = Quantity('parsec')
parsec.set_global_relative_scale_factor(3.0857e16 * meter, meter)
```

### 量纲分析

```python
from sympy.physics.units import Dimension, length, time, mass

# 检查量纲
from sympy.physics.units import convert_to, meter, second
velocity = 10 * meter / second
print(velocity.dimension)  # 量纲(length/time)
```

## 光学

### 高斯光学

```python
from sympy.physics.optics import (
    BeamParameter,
    FreeSpace,
    FlatRefraction,
    CurvedRefraction,
    ThinLens
)

# 高斯光束参数
q = BeamParameter(wavelen=532e-9, z=0, w=1e-3)

# 自由空间传播
q_new = FreeSpace(1) * q

# 薄透镜
q_focused = ThinLens(f=0.1) * q
```

### 波与偏振

```python
from sympy.physics.optics import TWave

# 平面波
wave = TWave(amplitude=1, frequency=5e14, phase=0)

# 介质属性（折射率等）
from sympy.physics.optics import Medium
medium = Medium('glass', permittivity=2.25)
```

## 连续介质力学

### 梁分析

```python
from sympy.physics.continuum_mechanics.beam import Beam
from sympy import symbols

# 定义梁
E, I = symbols('E I', positive=True)  # 杨氏模量，惯性矩
length = 10

beam = Beam(length, E, I)

# 施加载荷
from sympy.physics.continuum_mechanics.beam import Beam
beam.apply_load(-1000, 5, -1)  # 在x=5处施加-1000的点载荷

# 计算反力
beam.solve_for_reaction_loads()

# 获取剪力、弯矩、挠度
x = symbols('x')
shear = beam.shear_force()
moment = beam.bending_moment()
deflection = beam.deflection()
```

### 桁架分析

```python
from sympy.physics.continuum_mechanics.truss import Truss

# 创建桁架
truss = Truss()

# 添加节点
truss.add_node(('A', 0, 0), ('B', 4, 0), ('C', 2, 3))

# 添加构件
truss.add_member(('AB', 'A', 'B'), ('BC', 'B', 'C'))

# 施加载荷
truss.apply_load(('C', 1000, 270))  # 在节点C施加1000N（270°方向）

# 求解
truss.solve()
```

### 索分析

```python
from sympy.physics.continuum_mechanics.cable import Cable

# 创建索
cable = Cable(('A', 0, 10), ('B', 10, 10))

# 施加载荷
cable.apply_load(-1, 5)  # 分布载荷

# 求解张力与形状
cable.solve()
```

## 控制系统

### 传递函数与状态空间

```python
from sympy.physics.control import TransferFunction, StateSpace
from sympy.abc import s

# 传递函数
tf = TransferFunction(s + 1, s**2 + 2*s + 1, s)

# 状态空间表示
A = [[0, 1], [-1, -2]]
B = [[0], [1]]
C = [[1, 0]]
D = [[0]]

ss = StateSpace(A, B, C, D)

# 表示形式转换
ss_from_tf = tf.to_statespace()
tf_from_ss = ss.to_TransferFunction()
```

### 系统分析

```python
# 极点与零点
poles = tf.poles()
zeros = tf.zeros()

# 稳定性
is_stable = tf.is_stable()

# 阶跃响应、脉冲响应等
# (通常需要数值计算)
```

## 生物力学

### 肌肉肌腱模型

```python
from sympy.physics.biomechanics import (
    MusculotendonDeGroote2016,
    FirstOrderActivationDeGroote2016
)

# 创建肌肉肌腱模型
mt = MusculotendonDeGroote2016('muscle')

# 激活动力学
activation = FirstOrderActivationDeGroote2016('muscle_activation')
```

## 高能物理

### 粒子物理

```python
# Gamma矩阵与狄拉克方程
from sympy.physics.hep.gamma_matrices import GammaMatrix

gamma0 = GammaMatrix(0)
gamma1 = GammaMatrix(1)
```

## 常用物理模式

### 模式1：建立力学问题

```python
from sympy.physics.mechanics import dynamicsymbols, ReferenceFrame, Point
from sympy import symbols

# 1. 定义参考系
N = ReferenceFrame('N')

# 2. 定义广义坐标
q = dynamicsymbols('q')
q_dot = dynamicsymbols('q', 1)

# 3. 定义点与矢量
O = Point('O')
P = Point('P')

# 4. 设置运动学
P.set_pos(O, length * N.x)
P.set_vel(N, length * q_dot * N.x)

# 5. 定义力并应用拉格朗日或凯恩方法
```

### 模式2：量子态操作

```python
from sympy.physics.quantum import Ket, Operator, qapply

# 定义态
psi = Ket('psi')

# 定义算符
H = Operator('H')  # 哈密顿量

# 应用算符
result = qapply(H * psi)
```

### 模式3：单位转换流程

```python
from sympy.physics.units import convert_to, meter, foot, second, minute

# 定义带单位的物理量
distance = 100 * meter
time = 5 * minute

# 执行计算
speed = distance / time

# 转换为目标单位
speed_m_per_s = convert_to(speed, meter/second)
speed_ft_per_min = convert_to(speed, foot/minute)
```

### 模式4：梁挠度分析

```python
from sympy.physics.continuum_mechanics.beam import Beam
from sympy import symbols

E, I = symbols('E I', positive=True, real=True)
beam = Beam(10, E, I)

# 施加边界条件
beam.apply_support(0, 'pin')
beam.apply_support(10, 'roller')

# 施加载荷
beam.apply_load(-1000, 5, -1)  # 点载荷
beam.apply_load(-50, 0, 0, 10)  # 分布载荷

# 求解
beam.solve_for_reaction_loads()

# 获取特定位置结果
x = 5
deflection_at_mid = beam.deflection().subs(symbols('x'), x)
```

## 重要说明

1. **时变量：** 在力学问题中使用 `dynamicsymbols()` 表示时变量。

2. **单位：** 物理计算中始终使用 `sympy.physics.units` 模块显式指定单位。

3. **参考系：** 明确定义参考系及其相对方向以进行矢量分析。

4. **数值计算：** 许多物理计算需要数值求解。使用 `evalf()` 或转换为 NumPy 进行数值处理。

5. **假设条件：** 为符号设置适当假设（如 `positive=True`, `real=True`）以帮助 SymPy 正确简化物理表达式。
