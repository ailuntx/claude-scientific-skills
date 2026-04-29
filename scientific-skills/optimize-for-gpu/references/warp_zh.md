# NVIDIA Warp 参考文档 — GPU 模拟与空间计算

NVIDIA Warp 是一个用于编写高性能模拟与图形代码的 Python 框架。它通过将装饰有 `@wp.kernel` 的 Python 函数即时编译（JIT）为高效的 C++/CUDA 代码，可在 CPU 或 GPU 上运行。Warp 专为空间计算设计——物理模拟、机器人学、几何处理与可微分编程——内置丰富的类型（向量、矩阵、四元数、变换）和空间原语（网格、体素、哈希网格、BVH）。

与 Numba CUDA（提供原始线程/块控制）或 CuPy（替代 NumPy 操作）不同，Warp 提供更高层次的编程模型，内置支持可微分模拟、空间查询和基于分块的协作操作。

## 目录

1. [安装](#安装)
2. [何时使用 Warp 与其他库](#何时使用-warp-与其他库)
3. [内核与启动](#内核与启动)
4. [数组](#数组)
5. [数据类型](#数据类型)
6. [空间计算原语](#空间计算原语)
7. [基于分块的编程](#基于分块的编程)
8. [可微分性](#可微分性)
9. [流、事件与 CUDA 图](#流事件与-cuda-图)
10. [随机数生成](#随机数生成)
11. [互操作性](#互操作性)
12. [性能优化](#性能优化)
13. [常见模式](#常见模式)
14. [常见陷阱](#常见陷阱)

---

## 安装

```bash
uv add warp-lang              # CUDA 12 运行时（最常用）
# uv add warp-lang[examples]  # 包含 USD 和示例依赖
```

要求 CUDA 驱动版本 >= 525.60.13（Linux）或 528.33（Windows）。

验证安装：

```python
import warp as wp
wp.init()
# 打印设备信息、CUDA 版本、内核缓存位置
```

---

## 何时使用 Warp 与其他库

| 使用场景 | 最佳选择 | 原因 |
|----------|------------|-----|
| 物理模拟（粒子、布料、流体） | **Warp** | 内置空间原语、可微分、面向模拟优化 |
| 几何处理（网格、光线投射、SDF） | **Warp** | 原生网格/体素/BVH 类型、空间查询 |
| 用于 ML 训练的可微分模拟 | **Warp** | 自动前向/反向 AD、PyTorch/JAX 集成 |
| 机器人学（运动学、动力学、控制） | **Warp** | 内置变换、四元数、空间向量 |
| NumPy 数组数学（FFT、线性代数、排序） | **CuPy** | 直接替代 NumPy、封装 cuBLAS/cuFFT |
| 需原始线程控制的自定义 CUDA 内核 | **Numba** | 直接 CUDA 编程模型、共享内存 |
| 表格数据清洗/ETL | **cuDF** | GPU 上的 pandas API |
| ML 训练（sklearn 风格） | **cuML** | GPU 上的 scikit-learn API |

Warp 和 Numba 均将 Python 编译为 CUDA，但定位不同：
- **Warp** 擅长模拟/空间计算，具备丰富类型系统（vec3、quat、transform、mesh、volume）和自动微分
- **Numba** 擅长原始 CUDA 编程，支持显式线程/块控制、共享内存管理和任意数据的原子操作

---

## 内核与启动

### 定义内核

```python
import warp as wp

@wp.kernel
def compute_forces(positions: wp.array(dtype=wp.vec3),
                   velocities: wp.array(dtype=wp.vec3),
                   forces: wp.array(dtype=wp.vec3),
                   dt: float):
    tid = wp.tid()

    pos = positions[tid]
    vel = velocities[tid]

    # 重力
    force = wp.vec3(0.0, -9.81, 0.0)

    forces[tid] = force
```

### 启动内核

```python
# 一维启动
wp.launch(kernel=compute_forces,
          dim=num_particles,
          inputs=[positions, velocities, forces, 0.01],
          device="cuda")

# 二维启动（如图像处理）
wp.launch(kernel=compute_image, dim=(1024, 1024), inputs=[img], device="cuda")

# 三维启动
wp.launch(kernel=compute_field, dim=(nx, ny, nz), inputs=[field], device="cuda")
```

在 2D/3D 内核中获取索引：

```python
i, j = wp.tid()       # 二维
i, j, k = wp.tid()    # 三维
```

### 用户函数

```python
@wp.func
def spring_force(x0: wp.vec3, x1: wp.vec3, rest_length: float, stiffness: float):
    delta = x1 - x0
    length = wp.length(delta)
    direction = delta / length
    return stiffness * (length - rest_length) * direction
```

函数可从内核调用，支持重载，并可返回多个值。

### 用户结构体

```python
@wp.struct
class Particle:
    pos: wp.vec3
    vel: wp.vec3
    mass: float
    active: int
```

---

## 数组

Warp 数组是类型化、设备感知的容器（1D 至 4D）：

```python
# 分配
positions = wp.zeros(n, dtype=wp.vec3, device="cuda")
grid = wp.empty(shape=(nx, ny, nz), dtype=float, device="cuda")

# 从 NumPy 导入
import numpy as np
data = np.random.rand(1000, 3).astype(np.float32)
wp_data = wp.from_numpy(data, dtype=wp.vec3, device="cuda")

# 转回 NumPy（自动同步 GPU）
np_data = wp_data.numpy()

# 数组数学运算符
c = 2.0 * a + b   # 逐元素 GPU 加速
c *= 10.0          # 原地操作
```

内核签名的类型别名：`wp.array2d`、`wp.array3d`、`wp.array4d`。

---

## 数据类型

### 标量
`bool`、`int8`、`uint8`、`int16`、`uint16`、`int32`（别名：`int`）、`uint32`、`int64`、`uint64`、`float16`、`float32`（别名：`float`）、`float64`

### 向量
`vec2`、`vec3`、`vec4` — 默认为 float32。支持所有标量类型的变体：`vec3f`、`vec3d`、`vec3h`、`vec3i` 等。

```python
v = wp.vec3(1.0, 2.0, 3.0)
length = wp.length(v)
normalized = wp.normalize(v)
d = wp.dot(a, b)
c = wp.cross(a, b)
```

### 矩阵
`mat22`、`mat33`、`mat44` — 行优先。变体：`mat33f`、`mat33d`、`mat33h`。

```python
m = wp.mat33(1.0, 0.0, 0.0,
             0.0, 1.0, 0.0,
             0.0, 0.0, 1.0)
inv = wp.inverse(m)
det = wp.determinant(m)
result = m * v  # 矩阵-向量乘法
```

### 四元数
`quat`（i, j, k, w 布局，w 为实部）

```python
q = wp.quat_from_axis_angle(wp.vec3(0.0, 1.0, 0.0), 3.14159 / 2.0)
rotated = wp.quat_rotate(q, wp.vec3(1.0, 0.0, 0.0))
q_combined = wp.mul(q1, q2)  # 组合旋转
```

### 变换
`transform` — 7D（位置 vec3 + 四元数）

```python
t = wp.transform(wp.vec3(1.0, 2.0, 3.0), wp.quat_identity())
world_point = wp.transform_point(t, local_point)
world_dir = wp.transform_vector(t, local_dir)
```

### 空间向量与矩阵
`spatial_vector`（6D）、`spatial_matrix`（6x6）— 用于刚体动力学。

---

## 空间计算原语

### 网格（`wp.Mesh`）

带 BVH 的三角网格，支持快速光线投射和最近点查询：

```python
# 从顶点和三角形索引创建网格
mesh = wp.Mesh(points=vertices,     # wp.array(dtype=wp.vec3)
               indices=triangles)   # wp.array(dtype=int), 展平格式 (v0,v1,v2,...)

# 在内核中查询
@wp.kernel
def raycast(mesh_id: wp.uint64, origins: wp.array(dtype=wp.vec3),
            directions: wp.array(dtype=wp.vec3), hits: wp.array(dtype=float)):
    tid = wp.tid()
    query = wp.mesh_query_ray(mesh_id, origins[tid], directions[tid], 1000.0)
    if query.result:
        hits[tid] = query.t  # 命中距离

wp.launch(raycast, dim=n, inputs=[mesh.id, origins, dirs, hits])

# 更新顶点位置（拓扑固定）
mesh.points = new_positions
mesh.refit()  # 重建 BVH
```

### 哈希网格（`wp.HashGrid`）

空间哈希用于粒子邻居查询（DEM、SPH）：

```python
grid = wp.HashGrid(dim_x=128, dim_y=128, dim_z=128, device="cuda")
grid.build(points=particle_positions, radius=search_radius)

@wp.kernel
def find_neighbors(grid_id: wp.uint64, positions: wp.array(dtype=wp.vec3)):
    tid = wp.tid()
    pos = positions[tid]

    query = wp.hash_grid_query(grid_id, pos, search_radius)
    index = int(0)
    while wp.hash_grid_query_next(query, index):
        neighbor_pos = positions[index]
        dist = wp.length(pos - neighbor_pos)
        if dist < search_radius:
            # 处理邻居粒子
            ...
```

### 体素（`wp.Volume`）

基于 NanoVDB 的稀疏体素网格（SDF、速度场、烟雾）：

```python
# 从 NanoVDB 文件加载
volume = wp.Volume.load_from_nvdb("field.nvdb")

# 从 NumPy 创建（稠密→稀疏）
volume = wp.Volume.load_from_numpy(numpy_3d_array, bg_value=0.0)

# 在内核中采样
@wp.kernel
def sample_sdf(volume_id: wp.uint64, points: wp.array(dtype=wp.vec3),
               distances: wp.array(dtype=float)):
    tid = wp.tid()
    # 世界空间中的三线性插值
    uvw = wp.volume_world_to_index(volume_id, points[tid])
    distances[tid] = wp.volume_sample(volume_id, uvw, wp.Volume.LINEAR)
```

### BVH（`wp.Bvh`）

用于光线和 AABB 相交查询的包围体层次结构：

```python
bvh = wp.Bvh(lowers=box_mins, uppers=box_maxs)

# 光线查询
query = wp.bvh_query_ray(bvh.id, ray_origin, ray_dir)
# AABB 重叠查询
query = wp.bvh_query_aabb(bvh.id, aabb_min, aabb_max)
```

### Marching Cubes

从 3D 标量场提取等值面：

```python
mc = wp.MarchingCubes(nx=128, ny=128, nz=128, device="cuda")
mc.surface(field=sdf_array, threshold=0.0)

vertices = mc.verts    # wp.array(dtype=wp.vec3)
triangles = mc.indices # wp.array(dtype=int)
```

---

## 基于分块的编程

Warp 的分块 API 支持协作式块级操作（类似 Triton），使用共享内存和张量核心：

```python
TILE_M = wp.constant(16)
TILE_N = wp.constant(16)
TILE_K = wp.constant(16)
TILE_THREADS = 64

@wp.kernel
def tile_gemm(A: wp.array2d(dtype=float), B: wp.array2d(dtype=float),
              C: wp.array2d(dtype=float)):
    i, j = wp.tid()

    sum = wp.tile_zeros(shape=(TILE_M, TILE_N), dtype=wp.float32)
    count = int(A.shape[1] / TILE_K)

    for k in range(count):
        a = wp.tile_load(A, shape=(TILE_M, TILE_K), offset=(i * TILE_M, k * TILE_K))
        b = wp.tile_load(B, shape=(TILE_K, TILE_N), offset=(k * TILE_K, j * TILE_N))
        wp.tile_matmul(a, b, sum)

    wp.tile_store(C, sum, offset=(i * TILE_M, j * TILE_N))

wp.launch_tiled(tile_gemm, dim=(M // TILE_M, N // TILE_N),
                inputs=[A, B, C], block_dim=TILE_THREADS)
```

关键分块操作：
- **构造**：`tile_zeros`、`tile_ones`、`tile_load`、`tile_from_thread`
- **数学**：`tile_matmul`、`tile_fft`、`tile_ifft`、`tile_cholesky`、`tile_cholesky_solve`
- **归约**：`tile_sum`、`tile_min`、`tile_max`、`tile_reduce`
- **IO**：`tile_load`、`tile_store`、`tile_atomic_add`
- **算术**：支持 `+`、`-`、`*`、`/` 运算符
- **空间查询**：`tile_bvh_query_aabb`、`tile_mesh_query_aabb`

SIMT↔分块桥接：`wp.tile(scalar_value)` 从线程值创建分块；`wp.untile(tile)` 将分块解压回线程值。

---

## 可微分性

Warp 自动生成前向和反向（伴随）内核，支持基于梯度的优化和 ML 集成：

```python
# 参与梯度的数组需设置 requires_grad=True
a = wp.zeros(1024, dtype=wp.vec3, device="cuda", requires_grad=True)

# 记录前向传播
tape = wp.Tape()
with tape:
    wp.launch(kernel=compute1, inputs=[a, b], device="cuda")
    wp.launch(kernel=compute2, inputs=[c, d], device="cuda")
    wp.launch(kernel=loss_fn, inputs=[d, loss], device="cuda")

# 反向传播
tape.backward(loss)

# 访问梯度
grad_a = tape.gradients[a]
```

关键特性：
- 所有内核自动生成伴随代码
- `wp.Tape` 记录并回放计算图
- 与 PyTorch autograd 和 JAX JIT 集成
- 通过 `@wp.func_grad` 支持自定义梯度函数
- 提供雅可比矩阵计算支持

---

## 流、事件与 CUDA 图

```python
# 流支持并发执行
stream1 = wp.Stream("cuda:0")
stream2 = wp.Stream("cuda:0")
wp.launch(kernel1, ..., stream=stream1)
wp.launch(kernel2, ..., stream=stream2)

# CUDA 图捕获（消除 Python 启动开销）
with wp.ScopedCapture() as capture:
    wp.launch(kernel1, ...)
    wp.launch(kernel2, ...)
    wp.launch(kernel3, ...)

# 以接近零 CPU 开销重放图
for _ in range(1000):
    wp.capture_launch(capture.graph)
```

---

## 随机数生成

使用 PCG（置换同余生成器）— 按线程初始化：

```python
@wp.kernel
def monte_carlo(s

```markdown
cupy_arr = cp.asarray(warp_array)  # 零拷贝

### JAX（通过DLPack实现零拷贝）
```python
jax_array = wp.to_jax(warp_array)
warp_array = wp.from_jax(jax_array)
# @warp.jax_experimental.jax_kernel() 用于JAX原语集成
```

### DLPack（通用零拷贝）
```python
# 从任意DLPack框架导入
warp_array = wp.from_dlpack(external_array)
# 导出
external = framework.from_dlpack(warp_array)
```

---

## 性能优化

### 1. 使用CUDA图实现重复启动

若需多次启动相同内核序列（如模拟循环），CUDA图捕获可消除Python开销：

```python
with wp.ScopedCapture() as capture:
    for _ in range(substeps):
        wp.launch(integrate, ...)
        wp.launch(collide, ...)

for frame in range(num_frames):
    wp.capture_launch(capture.graph)
```

### 2. 最小化主机-设备传输

保持数据在GPU上。使用设备端`wp.array`，避免在内部循环中使用`.numpy()`。

### 3. 使用分块操作实现归约和GEMM

基于分块的归约比逐线程原子操作快50倍以上。使用`wp.tile()` + `wp.tile_sum()` + `wp.tile_atomic_add()`替代`wp.atomic_add()`。

### 4. 优先使用float32而非float64

GPU的float32吞吐量是float64的2-32倍。

### 5. 内核缓存

Warp在运行间缓存已编译内核。首次启动触发编译（可能需数秒）；后续运行从缓存加载（毫秒级）。

### 6. 对象生命周期

当空间图元（Mesh/HashGrid/Volume/BVH）的`.id`仍在使用时，保持其Python引用存活。若Python对象被垃圾回收而内核仍持有其ID，将导致未定义行为。

---

## 常用模式

### 粒子模拟
```python
@wp.kernel
def integrate_particles(positions: wp.array(dtype=wp.vec3),
                        velocities: wp.array(dtype=wp.vec3),
                        forces: wp.array(dtype=wp.vec3),
                        dt: float):
    tid = wp.tid()
    vel = velocities[tid] + forces[tid] * dt
    pos = positions[tid] + vel * dt

    velocities[tid] = vel
    positions[tid] = pos
```

### 网格光线投射
```python
@wp.kernel
def cast_rays(mesh_id: wp.uint64,
              ray_origins: wp.array(dtype=wp.vec3),
              ray_dirs: wp.array(dtype=wp.vec3),
              hit_points: wp.array(dtype=wp.vec3)):
    tid = wp.tid()
    query = wp.mesh_query_ray(mesh_id, ray_origins[tid], ray_dirs[tid], 1e6)
    if query.result:
        hit_points[tid] = ray_origins[tid] + ray_dirs[tid] * query.t
```

### 使用PyTorch实现可微分模拟
```python
import torch
import warp as wp

# Warp模拟内核
@wp.kernel
def simulate(state: wp.array(dtype=wp.vec3), params: wp.array(dtype=float),
             output: wp.array(dtype=wp.vec3)):
    tid = wp.tid()
    # ...物理计算...

# PyTorch训练循环
optimizer = torch.optim.Adam([torch_params], lr=1e-3)

for epoch in range(100):
    wp_params = wp.from_torch(torch_params)
    tape = wp.Tape()
    with tape:
        wp.launch(simulate, dim=n, inputs=[state, wp_params, output])
        wp.launch(loss_kernel, dim=1, inputs=[output, target, loss])

    tape.backward(loss)
    grad = wp.to_torch(tape.gradients[wp_params])
    torch_params.grad = grad
    optimizer.step()
```

### 基于哈希网格的SPH流体
```python
grid = wp.HashGrid(128, 128, 128, device="cuda")

@wp.kernel
def compute_density(grid_id: wp.uint64, positions: wp.array(dtype=wp.vec3),
                    densities: wp.array(dtype=float), radius: float):
    tid = wp.tid()
    pos = positions[tid]
    density = float(0.0)

    query = wp.hash_grid_query(grid_id, pos, radius)
    index = int(0)
    while wp.hash_grid_query_next(query, index):
        dist = wp.length(pos - positions[index])
        if dist < radius:
            # SPH核函数
            q = dist / radius
            density += (1.0 - q) * (1.0 - q) * (1.0 - q)

    densities[tid] = density

# 每时间步：
grid.build(points=positions, radius=h)
wp.launch(compute_density, dim=n, inputs=[grid.id, positions, densities, h])
```

---

## 常见陷阱

1. **遗漏类型注解**——所有内核参数必须显式类型标注。Warp通过注解而非运行时值推断类型。

2. **在内核中使用Python数据结构**——禁止使用list/dict/set。应使用`wp.array`/`wp.vec3`/`@wp.struct`。

3. **在用户函数中调用`wp.tid()`**——`wp.tid()`仅在内核中有效。需将线程索引作为参数传递给`@wp.func`函数。

4. **对象生命周期问题**——空间图元（Mesh/HashGrid/Volume/BVH）必须在其`.id`被内核使用时保持存活（Python引用）。若Python对象被垃圾回收将导致崩溃。

5. **期望原地操作可微分**——Warp自动微分不支持数组原地修改。需写入独立输出数组以计算梯度。

6. **未使用`requires_grad=True`**——参与梯度计算的数组创建时必须设置`requires_grad=True`。

7. **设备不一致启动**——数组和内核启动必须在相同设备。需统一使用`device="cuda"`。

8. **首次启动编译时间**——首次内核启动触发JIT编译（可能需数秒）。后续运行使用缓存。避免在首次运行时进行基准测试。

9. **使用元组替代Warp类型**——内核作用域中`(1.0, 2.0, 3.0)`无效。应使用`wp.vec3(1.0, 2.0, 3.0)`。

10. **CPU上的块大小**——CPU分块操作使用`block_dim=1`会改变行为。设计跨设备内核时应保持块大小无关性。
```
