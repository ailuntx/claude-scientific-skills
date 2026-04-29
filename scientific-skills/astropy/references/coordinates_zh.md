# 天文坐标 (astropy.coordinates)

`astropy.coordinates` 包提供了表示天体坐标和在坐标系间转换的工具。

## 使用 SkyCoord 创建坐标

推荐使用高级 `SkyCoord` 类接口：

```python
from astropy import units as u
from astropy.coordinates import SkyCoord

# 十进制角度
c = SkyCoord(ra=10.625*u.degree, dec=41.2*u.degree, frame='icrs')

# 六十进制字符串
c = SkyCoord(ra='00h42m30s', dec='+41d12m00s', frame='icrs')

# 混合格式
c = SkyCoord('00h42.5m +41d12m', unit=(u.hourangle, u.deg))

# 银道坐标
c = SkyCoord(l=120.5*u.degree, b=-23.4*u.degree, frame='galactic')
```

## 数组坐标

高效处理多个坐标：

```python
# 创建坐标数组
coords = SkyCoord(ra=[10, 11, 12]*u.degree,
                  dec=[41, -5, 42]*u.degree)

# 访问单个元素
coords[0]
coords[1:3]

# 数组操作
coords.shape
len(coords)
```

## 访问坐标分量

```python
c = SkyCoord(ra=10.68*u.degree, dec=41.27*u.degree, frame='icrs')

# 访问坐标
c.ra        # <Longitude 10.68 deg>
c.dec       # <Latitude 41.27 deg>
c.ra.hour   # 转换为小时制
c.ra.hms    # 时/分/秒元组
c.dec.dms   # 度/角分/角秒元组
```

## 字符串格式化

```python
c.to_string('decimal')      # '10.68 41.27'
c.to_string('dms')          # '10d40m48s 41d16m12s'
c.to_string('hmsdms')       # '00h42m43.2s +41d16m12s'

# 自定义格式
c.ra.to_string(unit=u.hour, sep=':', precision=2)
```

## 坐标转换

在参考系间转换：

```python
c_icrs = SkyCoord(ra=10.68*u.degree, dec=41.27*u.degree, frame='icrs')

# 简单转换（属性方式）
c_galactic = c_icrs.galactic
c_fk5 = c_icrs.fk5
c_fk4 = c_icrs.fk4

# 显式转换
c_icrs.transform_to('galactic')
c_icrs.transform_to(FK5(equinox='J1975'))  # 自定义框架参数
```

## 常用坐标系框架

### 天球框架
- **ICRS**: 国际天球参考系（默认最常用）
- **FK5**: 第五基本星表（默认历元 J2000.0）
- **FK4**: 第四基本星表（较旧，需指定历元）
- **GCRS**: 地心天球参考系
- **CIRS**: 天球中间参考系

### 银道框架
- **Galactic**: IAU 1958 银道坐标
- **Supergalactic**: 德沃古勒超银道坐标
- **Galactocentric**: 银心三维坐标

### 地平框架
- **AltAz**: 高度-方位角（观测者相关）
- **HADec**: 时角-赤纬

### 黄道框架
- **GeocentricMeanEcliptic**: 地心平黄道
- **BarycentricMeanEcliptic**: 质心平黄道
- **HeliocentricMeanEcliptic**: 日心平黄道

## 观测者相关转换

高度-方位角坐标需指定观测时间和位置：

```python
from astropy.time import Time
from astropy.coordinates import EarthLocation, AltAz

# 定义观测位置
observing_location = EarthLocation(lat=40.8*u.deg, lon=-121.5*u.deg, height=1060*u.m)
# 或使用命名天文台
observing_location = EarthLocation.of_site('Apache Point Observatory')

# 定义观测时间
observing_time = Time('2023-01-15 23:00:00')

# 转换到高度-方位角
aa_frame = AltAz(obstime=observing_time, location=observing_location)
aa = c_icrs.transform_to(aa_frame)

print(f"高度: {aa.alt}")
print(f"方位角: {aa.az}")
```

## 距离处理

为三维坐标添加距离信息：

```python
# 含距离的坐标
c = SkyCoord(ra=10*u.degree, dec=9*u.degree, distance=770*u.kpc, frame='icrs')

# 访问三维笛卡尔坐标
c.cartesian.x
c.cartesian.y
c.cartesian.z

# 到原点的距离
c.distance

# 三维分离
c1 = SkyCoord(ra=10*u.degree, dec=9*u.degree, distance=10*u.pc)
c2 = SkyCoord(ra=11*u.degree, dec=10*u.degree, distance=11.5*u.pc)
sep_3d = c1.separation_3d(c2)  # 三维距离
```

## 角距离计算

计算天球投影分离：

```python
c1 = SkyCoord(ra=10*u.degree, dec=9*u.degree, frame='icrs')
c2 = SkyCoord(ra=11*u.degree, dec=10*u.degree, frame='fk5')

# 角距离（自动处理框架转换）
sep = c1.separation(c2)
print(f"分离角: {sep.arcsec} 角秒")

# 位置角
pa = c1.position_angle(c2)
```

## 星表匹配

匹配坐标到星表源：

```python
# 单目标匹配
catalog = SkyCoord(ra=ra_array*u.degree, dec=dec_array*u.degree)
target = SkyCoord(ra=10.5*u.degree, dec=41.2*u.degree)

# 查找最近匹配
idx, sep2d, dist3d = target.match_to_catalog_sky(catalog)
matched_coord = catalog[idx]

# 带最大分离约束的匹配
matches = target.separation(catalog) < 1*u.arcsec
```

## 命名天体

从在线星表获取坐标：

```python
# 按名称查询（需联网）
m31 = SkyCoord.from_name("M31")
crab = SkyCoord.from_name("Crab Nebula")
psr = SkyCoord.from_name("PSR J1012+5307")
```

## 地球位置

定义观测位置：

```python
# 通过坐标定义
location = EarthLocation(lat=40*u.deg, lon=-120*u.deg, height=1000*u.m)

# 通过命名天文台
keck = EarthLocation.of_site('Keck Observatory')
vlt = EarthLocation.of_site('Paranal Observatory')

# 通过地址（需联网）
location = EarthLocation.of_address('1002 Holy Grail Court, St. Louis, MO')

# 列出可用天文台
EarthLocation.get_site_names()
```

## 速度信息

包含自行和径向速度：

```python
# 自行
c = SkyCoord(ra=10*u.degree, dec=41*u.degree,
             pm_ra_cosdec=15*u.mas/u.yr,
             pm_dec=5*u.mas/u.yr,
             distance=150*u.pc)

# 径向速度
c = SkyCoord(ra=10*u.degree, dec=41*u.degree,
             radial_velocity=20*u.km/u.s)

# 同时包含
c = SkyCoord(ra=10*u.degree, dec=41*u.degree, distance=150*u.pc,
             pm_ra_cosdec=15*u.mas/u.yr, pm_dec=5*u.mas/u.yr,
             radial_velocity=20*u.km/u.s)
```

## 表示类型

切换坐标表示形式：

```python
# 笛卡尔表示
c = SkyCoord(x=1*u.kpc, y=2*u.kpc, z=3*u.kpc,
             representation_type='cartesian', frame='icrs')

# 更改表示类型
c.representation_type = 'cylindrical'
c.rho  # 柱坐标半径
c.phi  # 方位角
c.z    # 高度

# 球坐标（多数框架默认）
c.representation_type = 'spherical'
```

## 性能优化

1. **使用数组而非循环**：将多个坐标作为单数组处理
2. **预计算框架**：复用框架对象进行多次转换
3. **利用广播机制**：高效处理多位置跨多时间的转换
4. **启用插值**：密集时间采样时使用 ErfaAstromInterpolator

```python
# 快速方法
coords = SkyCoord(ra=ra_array*u.degree, dec=dec_array*u.degree)
coords_transformed = coords.transform_to('galactic')

# 慢速方法（避免）
for ra, dec in zip(ra_array, dec_array):
    c = SkyCoord(ra=ra*u.degree, dec=dec*u.degree)
    c_transformed = c.transform_to('galactic')
```
