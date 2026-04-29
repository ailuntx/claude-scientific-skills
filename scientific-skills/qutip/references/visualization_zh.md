# QuTiP 可视化

## 布洛赫球面

在布洛赫球面上可视化量子比特状态。

### 基础用法

```python
from qutip import *
import matplotlib.pyplot as plt

# 创建布洛赫球面
b = Bloch()

# 添加量子态
psi = (basis(2, 0) + basis(2, 1)).unit()
b.add_states(psi)

# 添加向量
b.add_vectors([1, 0, 0])  # X轴

# 显示
b.show()
```

### 多量子态可视化

```python
# 添加多个量子态
states = [(basis(2, 0) + basis(2, 1)).unit(),
          (basis(2, 0) + 1j*basis(2, 1)).unit()]
b.add_states(states)

# 添加数据点
b.add_points([[0, 1, 0], [0, -1, 0]])

# 自定义颜色
b.point_color = ['r', 'g']
b.point_marker = ['o', 's']
b.point_size = [20, 20]

b.show()
```

### 动态演化

```python
# 量子态演化动画
states = result.states  # 来自sesolve/mesolve

b = Bloch()
b.vector_color = ['r']
b.view = [-40, 30]  # 视角设置

# 创建动画
from matplotlib.animation import FuncAnimation

def animate(i):
    b.clear()
    b.add_states(states[i])
    b.make_sphere()
    return b.axes

anim = FuncAnimation(b.fig, animate, frames=len(states),
                      interval=50, blit=False, repeat=True)
plt.show()
```

### 自定义设置

```python
b = Bloch()

# 球面外观
b.sphere_color = '#FFDDDD'
b.sphere_alpha = 0.1
b.frame_alpha = 0.1

# 坐标轴标签
b.xlabel = ['$|+\\\\rangle$', '$|-\\\\rangle$']
b.ylabel = ['$|+i\\\\rangle$', '$|-i\\\\rangle$']
b.zlabel = ['$|0\\\\rangle$', '$|1\\\\rangle$']

# 字体设置
b.font_size = 20
b.font_color = 'black'

# 视角角度
b.view = [-60, 30]

# 保存图像
b.save('bloch.png')
```

## 维格纳函数

相空间准概率分布。

### 基础计算

```python
# 创建量子态
psi = coherent(N, alpha)

# 计算维格纳函数
xvec = np.linspace(-5, 5, 200)
W = wigner(psi, xvec, xvec)

# 绘图
fig, ax = plt.subplots(1, 1, figsize=(6, 6))
cont = ax.contourf(xvec, xvec, W, 100, cmap='RdBu')
ax.set_xlabel('Re(α)')
ax.set_ylabel('Im(α)')
plt.colorbar(cont, ax=ax)
plt.show()
```

### 特殊配色方案

```python
# 突出负值的维格纳配色方案
from qutip import wigner_cmap

W = wigner(psi, xvec, xvec)

fig, ax = plt.subplots()
cont = ax.contourf(xvec, xvec, W, 100, cmap=wigner_cmap(W))
ax.set_title('维格纳函数')
plt.colorbar(cont)
plt.show()
```

### 三维曲面图

```python
from mpl_toolkits.mplot3d import Axes3D

X, Y = np.meshgrid(xvec, xvec)

fig = plt.figure(figsize=(8, 6))
ax = fig.add_subplot(111, projection='3d')
ax.plot_surface(X, Y, W, cmap='RdBu', alpha=0.8)
ax.set_xlabel('Re(α)')
ax.set_ylabel('Im(α)')
ax.set_zlabel('W(α)')
plt.show()
```

### 量子态对比

```python
# 比较不同量子态
states = [coherent(N, 2), fock(N, 2), thermal_dm(N, 2)]
titles = ['相干态', '福克态', '热态']

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

for i, (state, title) in enumerate(zip(states, titles)):
    W = wigner(state, xvec, xvec)
    cont = axes[i].contourf(xvec, xvec, W, 100, cmap='RdBu')
    axes[i].set_title(title)
    axes[i].set_xlabel('Re(α)')
    if i == 0:
        axes[i].set_ylabel('Im(α)')

plt.tight_layout()
plt.show()
```

## Q函数（Husimi函数）

平滑相空间分布（恒为正）。

### 基础用法

```python
from qutip import qfunc

Q = qfunc(psi, xvec, xvec)

fig, ax = plt.subplots()
cont = ax.contourf(xvec, xvec, Q, 100, cmap='viridis')
ax.set_xlabel('Re(α)')
ax.set_ylabel('Im(α)')
ax.set_title('Q函数')
plt.colorbar(cont)
plt.show()
```

### 高效批量计算

```python
from qutip import QFunc

# 用于多点计算Q函数
qf = QFunc(rho)
Q = qf.eval(xvec, xvec)
```

## 福克态概率分布

可视化光子数分布。

### 基础直方图

```python
from qutip import plot_fock_distribution

# 单量子态
psi = coherent(N, 2)
fig, ax = plot_fock_distribution(psi)
ax.set_title('相干态')
plt.show()
```

### 分布对比

```python
states = {
    '相干态': coherent(20, 2),
    '热态': thermal_dm(20, 2),
    '福克态': fock(20, 2)
}

fig, axes = plt.subplots(1, 3, figsize=(15, 4))

for ax, (title, state) in zip(axes, states.items()):
    plot_fock_distribution(state, fig=fig, ax=ax)
    ax.set_title(title)
    ax.set_ylim([0, 0.3])

plt.tight_layout()
plt.show()
```

### 时间演化

```python
# 展示光子数分布演化
result = mesolve(H, psi0, tlist, c_ops)

# 在不同时间点绘图
times_to_plot = [0, 5, 10, 15]
fig, axes = plt.subplots(1, 4, figsize=(16, 4))

for ax, t_idx in zip(axes, times_to_plot):
    plot_fock_distribution(result.states[t_idx], fig=fig, ax=ax)
    ax.set_title(f't = {tlist[t_idx]:.1f}')
    ax.set_ylim([0, 1])

plt.tight_layout()
plt.show()
```

## 矩阵可视化

### 汉顿图

通过加权方块可视化矩阵结构。

```python
from qutip import hinton

# 密度矩阵
rho = bell_state('00').proj()

hinton(rho)
plt.title('贝尔态密度矩阵')
plt.show()
```

### 矩阵直方图

矩阵元素的三维柱状图。

```python
from qutip import matrix_histogram

# 显示实部与虚部
H = sigmaz()

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

matrix_histogram(H.full(), xlabels=['0', '1'], ylabels=['0', '1'],
                 fig=fig, ax=axes[0])
axes[0].set_title('实部')

matrix_histogram(H.full(), bar_type='imag', xlabels=['0', '1'],
                 ylabels=['0', '1'], fig=fig, ax=axes[1])
axes[1].set_title('虚部')

plt.tight_layout()
plt.show()
```

### 复相位图

```python
# 可视化复矩阵元素
rho = coherent_dm(10, 2)

# 绘制复元素
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# 模值
matrix_histogram(rho.full(), bar_type='abs', fig=fig, ax=axes[0])
axes[0].set_title('模值')

# 相位
matrix_histogram(rho.full(), bar_type='phase', fig=fig, ax=axes[1])
axes[1].set_title('相位')

plt.tight_layout()
plt.show()
```

## 能级图

```python
# 可视化能量本征值
H = num(N) + 0.1 * (create(N) + destroy(N))**2

# 获取本征值和本征态
evals, ekets = H.eigenstates()

# 绘制能级
fig, ax = plt.subplots(figsize=(8, 6))

for i, E in enumerate(evals[:10]):
    ax.hlines(E, 0, 1, linewidth=2)
    ax.text(1.1, E, f'|{i}⟩', va='center')

ax.set_ylabel('能量')
ax.set_xlim([-0.2, 1.5])
ax.set_xticks([])
ax.set_title('能谱')
plt.show()
```

## 量子过程层析

可视化量子通道/门操作。

```python
from qutip.qip.operations import cnot
from qutip_qip.tomography import qpt, qpt_plot_combined

# 定义量子过程（如CNOT门）
U = cnot()

# 执行量子过程层析
chi = qpt(U, method='choicm')

# 可视化
fig = qpt_plot_combined(chi)
plt.show()
```

## 期望值随时间演化

```python
# 期望值标准绘图
result = mesolve(H, psi0, tlist, c_ops, e_ops=[num(N)])

fig, ax = plt.subplots()
ax.plot(tlist, result.expect[0])
ax.set_xlabel('时间')
ax.set_ylabel('⟨n⟩')
ax.set_title('光子数演化')
ax.grid(True)
plt.show()
```

### 多观测量

```python
# 绘制多个期望值
e_ops = [a.dag() * a, a + a.dag(), 1j * (a - a.dag())]
labels = ['⟨n⟩', '⟨X⟩', '⟨P⟩']

result = mesolve(H, psi0, tlist, c_ops, e_ops=e_ops)

fig, axes = plt.subplots(3, 1, figsize=(8, 9))

for i, (ax, label) in enumerate(zip(axes, labels)):
    ax.plot(tlist, result.expect[i])
    ax.set_ylabel(label)
    ax.grid(True)

axes[-1].set_xlabel('时间')
plt.tight_layout()
plt.show()
```

## 关联函数与频谱

```python
# 双时关联函数
taulist = np.linspace(0, 10, 200)
corr = correlation_2op_1t(H, rho0, taulist, c_ops, a.dag(), a)

# 绘制关联函数
fig, ax = plt.subplots()
ax.plot(taulist, np.real(corr))
ax.set_xlabel('τ')
ax.set_ylabel('⟨a†(τ)a(0)⟩')
ax.set_title('关联函数')
plt.show()

# 功率谱
from qutip import spectrum_correlation_fft

w, S = spectrum_correlation_fft(taulist, corr)

fig, ax = plt.subplots()
ax.plot(w, S)
ax.set_xlabel('频率')
ax.set_ylabel('S(ω)')
ax.set_title('功率谱')
plt.show()
```

## 图像保存

```python
# 高分辨率保存
fig.savefig('my_plot.png', dpi=300, bbox_inches='tight')
fig.savefig('my_plot.pdf', bbox_inches='tight')
fig.savefig('my_plot.svg', bbox_inches='tight')
```
