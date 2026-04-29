# 单位与物理量 (astropy.units)

`astropy.units` 模块用于定义物理量、进行单位转换以及执行带单位的算术运算。

## 创建物理量

通过数值与内置单位相乘或相除创建 Quantity 对象：

```python
from astropy import units as u
import numpy as np

# 标量物理量
distance = 42.0 * u.meter
velocity = 100 * u.km / u.s

# 数组物理量
distances = np.array([1., 2., 3.]) * u.m
wavelengths = [500, 600, 700] * u.nm
```

通过 `.value` 和 `.unit` 属性访问数值和单位：
```python
distance.value  # 42.0
distance.unit   # Unit("m")
```

## 单位转换

使用 `.to()` 方法进行转换：

```python
distance = 1.0 * u.parsec
distance.to(u.km)  # <Quantity 30856775814671.914 km>

wavelength = 500 * u.nm
wavelength.to(u.angstrom)  # <Quantity 5000. Angstrom>
```

## 算术运算

物理量支持自动单位管理的标准算术运算：

```python
# 基础运算
speed = 15.1 * u.meter / (32.0 * u.second)  # <Quantity 0.471875 m / s>
area = (5 * u.m) * (3 * u.m)  # <Quantity 15. m2>

# 单位自动约简
ratio = (10 * u.m) / (5 * u.m)  # <Quantity 2. (dimensionless)>

# 分解复合单位
time = (3.0 * u.kilometer / (130.51 * u.meter / u.second))
time.decompose()  # <Quantity 22.986744310780782 s>
```

## 单位系统

支持主要单位系统间转换：

```python
# SI 转 CGS
pressure = 1.0 * u.Pa
pressure.cgs  # <Quantity 10. Ba>

# 查找等效表示
(u.s ** -1).compose()  # [Unit("Bq"), Unit("Hz"), ...]
```

## 等效关系

特定领域的转换需要等效关系参数：

```python
# 光谱等效（波长 ↔ 频率）
wavelength = 1000 * u.nm
wavelength.to(u.Hz, equivalencies=u.spectral())
# <Quantity 2.99792458e+14 Hz>

# 多普勒等效
velocity = 1000 * u.km / u.s
velocity.to(u.Hz, equivalencies=u.doppler_optical(500*u.nm))

# 其他等效关系
u.brightness_temperature(500*u.GHz)
u.doppler_radio(1.4*u.GHz)
u.mass_energy()
u.parallax()
```

## 对数单位

支持星等、分贝和 dex 等特殊对数单位：

```python
# 星等
flux = -2.5 * u.mag(u.ct / u.s)

# 分贝
power_ratio = 3 * u.dB(u.W)

# Dex（以10为底的对数）
abundance = 8.5 * u.dex(u.cm**-3)
```

## 常用单位

### 长度
`u.m, u.km, u.cm, u.mm, u.micron, u.angstrom, u.au, u.pc, u.kpc, u.Mpc, u.lyr`

### 时间
`u.s, u.min, u.hour, u.day, u.year, u.Myr, u.Gyr`

### 质量
`u.kg, u.g, u.M_sun, u.M_earth, u.M_jup`

### 温度
`u.K, u.deg_C`

### 角度
`u.deg, u.arcmin, u.arcsec, u.rad, u.hourangle, u.mas`

### 能量/功率
`u.J, u.erg, u.eV, u.keV, u.MeV, u.GeV, u.W, u.L_sun`

### 频率
`u.Hz, u.kHz, u.MHz, u.GHz`

### 流量
`u.Jy, u.mJy, u.erg / u.s / u.cm**2`

## 性能优化

对数组运算预计算复合单位：

```python
# 慢速（生成中间物理量）
result = array * u.m / u.s / u.kg / u.sr

# 快速（预计算复合单位）
UNIT_COMPOSITE = u.m / u.s / u.kg / u.sr
result = array * UNIT_COMPOSITE

# 极速（使用 << 避免复制）
result = array << UNIT_COMPOSITE  # 速度提升10000倍
```

## 字符串格式化

使用标准 Python 语法格式化物理量：

```python
velocity = 15.1 * u.meter / (32.0 * u.second)
f"{velocity:0.03f}"     # '0.472 m / s'
f"{velocity:.2e}"       # '4.72e-01 m / s'
f"{velocity.unit:FITS}" # 'm s-1'
```

## 定义自定义单位

```python
# 创建新单位
bakers_fortnight = u.def_unit('bakers_fortnight', 13 * u.day)

# 启用字符串解析
u.add_enabled_units([bakers_fortnight])
```

## 物理常量

获取带单位的物理常量：

```python
from astropy.constants import c, G, M_sun, h, k_B

speed_of_light = c.to(u.km/u.s)
gravitational_constant = G.to(u.m**3 / u.kg / u.s**2)
```
