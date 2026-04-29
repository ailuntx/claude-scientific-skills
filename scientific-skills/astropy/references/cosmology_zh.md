# 宇宙学计算 (astropy.cosmology)

`astropy.cosmology` 子包提供基于多种宇宙学模型的计算工具。

## 使用内置宇宙学模型

基于 WMAP 和 Planck 观测数据的预置宇宙学模型：

```python
from astropy.cosmology import Planck18, Planck15, Planck13
from astropy.cosmology import WMAP9, WMAP7, WMAP5
from astropy import units as u

# 使用 Planck 2018 宇宙学模型
cosmo = Planck18

# 计算 z=4 处的距离
d = cosmo.luminosity_distance(4)
print(f"红移 z=4 处的光度距离: {d}")

# z=0 时的宇宙年龄
age = cosmo.age(0)
print(f"当前宇宙年龄: {age.to(u.Gyr)}")
```

## 创建自定义宇宙学模型

### FlatLambdaCDM (最常用)

含宇宙学常数的平坦宇宙：

```python
from astropy.cosmology import FlatLambdaCDM

# 定义宇宙学模型
cosmo = FlatLambdaCDM(
    H0=70 * u.km / u.s / u.Mpc,  # z=0 处的哈勃常数
    Om0=0.3,                      # z=0 处的物质密度参数
    Tcmb0=2.725 * u.K             # 宇宙微波背景温度（可选）
)
```

### LambdaCDM (非平坦)

含宇宙学常数的非平坦宇宙：

```python
from astropy.cosmology import LambdaCDM

cosmo = LambdaCDM(
    H0=70 * u.km / u.s / u.Mpc,
    Om0=0.3,
    Ode0=0.7  # 暗能量密度参数
)
```

### wCDM 和 w0wzCDM

含状态方程参数的暗能量模型：

```python
from astropy.cosmology import FlatwCDM, w0wzCDM

# 常数 w
cosmo_w = FlatwCDM(H0=70 * u.km/u.s/u.Mpc, Om0=0.3, w0=-0.9)

# 演化 w(z) = w0 + wz * z
cosmo_wz = w0wzCDM(H0=70 * u.km/u.s/u.Mpc, Om0=0.3, Ode0=0.7,
                   w0=-1.0, wz=0.1)
```

## 距离计算

### 共动距离

视线方向共动距离：

```python
d_c = cosmo.comoving_distance(z)
```

### 光度距离

用于通过观测流量计算光度的距离：

```python
d_L = cosmo.luminosity_distance(z)

# 通过视星等计算绝对星等
M = m - 5*np.log10(d_L.to(u.pc).value) + 5
```

### 角直径距离

用于通过角尺寸计算物理尺寸的距离：

```python
d_A = cosmo.angular_diameter_distance(z)

# 通过角尺寸计算物理尺寸
theta = 10 * u.arcsec  # 角尺寸
physical_size = d_A * theta.to(u.radian).value
```

### 横向共动距离

横向共动距离（在平坦宇宙中等同于共动距离）：

```python
d_M = cosmo.comoving_transverse_distance(z)
```

### 距离模数

```python
dm = cosmo.distmod(z)
# 关联视星等和绝对星等: m - M = dm
```

## 尺度计算

### 每角分千秒差距

给定红移处的物理尺度：

```python
scale = cosmo.kpc_proper_per_arcmin(z)
# 例如："红移 z=1 处每角分对应 50 千秒差距"
```

### 共动体积

用于巡天体积计算的体积元：

```python
vol = cosmo.comoving_volume(z)  # 到红移 z 处的总体积
vol_element = cosmo.differential_comoving_volume(z)  # dV/dz
```

## 时间计算

### 宇宙年龄

给定红移处的年龄：

```python
age = cosmo.age(z)
age_now = cosmo.age(0)  # 当前年龄
age_at_z1 = cosmo.age(1)  # z=1 处的年龄
```

### 回溯时间

光子发射至今的时间：

```python
t_lookback = cosmo.lookback_time(z)
# 红移 z 到 z=0 之间的时间
```

## 哈勃参数

作为红移函数的哈勃参数：

```python
H_z = cosmo.H(z)  # H(z) 单位 km/s/Mpc
E_z = cosmo.efunc(z)  # E(z) = H(z)/H0
```

## 密度参数

密度参数随红移的演化：

```python
Om_z = cosmo.Om(z)        # z 处的物质密度
Ode_z = cosmo.Ode(z)      # z 处的暗能量密度
Ok_z = cosmo.Ok(z)        # z 处的曲率密度
Ogamma_z = cosmo.Ogamma(z)  # z 处的光子密度
Onu_z = cosmo.Onu(z)      # z 处的中微子密度
```

## 临界密度与特征密度

```python
rho_c = cosmo.critical_density(z)  # z 处的临界密度
rho_m = cosmo.critical_density(z) * cosmo.Om(z)  # 物质密度
```

## 逆向计算

查找特定值对应的红移：

```python
from astropy.cosmology import z_at_value

# 查找特定回溯时间对应的红移
z = z_at_value(cosmo.lookback_time, 10*u.Gyr)

# 查找特定光度距离对应的红移
z = z_at_value(cosmo.luminosity_distance, 1000*u.Mpc)

# 查找特定年龄对应的红移
z = z_at_value(cosmo.age, 1*u.Gyr)
```

## 数组运算

所有方法均支持数组输入：

```python
import numpy as np

z_array = np.linspace(0, 5, 100)
d_L_array = cosmo.luminosity_distance(z_array)
H_array = cosmo.H(z_array)
age_array = cosmo.age(z_array)
```

## 中微子效应

包含有质量中微子：

```python
from astropy.cosmology import FlatLambdaCDM

# 含质量中微子
cosmo = FlatLambdaCDM(
    H0=70 * u.km/u.s/u.Mpc,
    Om0=0.3,
    Tcmb0=2.725 * u.K,
    Neff=3.04,  # 中微子有效种类数
    m_nu=[0., 0., 0.06] * u.eV  # 中微子质量
)
```

注意：含质量中微子会使计算性能降低 3-4 倍，但结果更精确。

## 克隆与修改宇宙学模型

宇宙学对象不可变，需创建修改副本：

```python
# 克隆并修改 H0
cosmo_new = cosmo.clone(H0=72 * u.km/u.s/u.Mpc)

# 克隆并修改名称
cosmo_named = cosmo.clone(name="我的自定义宇宙学模型")
```

## 常见用例

### 计算绝对星等

```python
# 通过视星等和红移计算
z = 1.5
m_app = 24.5  # 视星等
d_L = cosmo.luminosity_distance(z)
M_abs = m_app - cosmo.distmod(z).value
```

### 巡天体积计算

```python
# 两个红移间的体积
z_min, z_max = 0.5, 1.5
volume = cosmo.comoving_volume(z_max) - cosmo.comoving_volume(z_min)

# 转换为 Gpc^3
volume_gpc3 = volume.to(u.Gpc**3)
```

### 通过角尺寸计算物理尺寸

```python
theta = 1 * u.arcsec  # 角尺寸
z = 2.0
d_A = cosmo.angular_diameter_distance(z)
size_kpc = (d_A * theta.to(u.radian)).to(u.kpc)
```

### 大爆炸至今时间

```python
# 特定红移处的年龄
z_formation = 6
age_at_formation = cosmo.age(z_formation)
time_since_formation = cosmo.age(0) - age_at_formation
```

## 宇宙学模型比较

```python
# 比较不同模型
from astropy.cosmology import Planck18, WMAP9

z = 1.0
print(f"Planck18 光度距离: {Planck18.luminosity_distance(z)}")
print(f"WMAP9 光度距离: {WMAP9.luminosity_distance(z)}")
```

## 性能注意事项

- 多数场景下计算速度较快
- 含质量中微子会显著降低速度
- 数组运算已向量化且高效
- 结果在 z < 5000-6000 范围内有效（取决于模型）
