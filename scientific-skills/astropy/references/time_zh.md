# 时间处理（astropy.time）

`astropy.time` 模块提供了强大的工具来处理时间和日期，支持多种时间尺度和格式。

## 创建时间对象

### 基础创建

```python
from astropy.time import Time
import astropy.units as u

# ISO格式（自动检测）
t = Time('2023-01-15 12:30:45')
t = Time('2023-01-15T12:30:45')

# 显式指定格式
t = Time('2023-01-15 12:30:45', format='iso', scale='utc')

# 儒略日
t = Time(2460000.0, format='jd')

# 简化儒略日
t = Time(59945.0, format='mjd')

# Unix时间（1970-01-01起的秒数）
t = Time(1673785845.0, format='unix')
```

### 时间数组

```python
# 多个时间点
times = Time(['2023-01-01', '2023-06-01', '2023-12-31'])

# 从数组创建
import numpy as np
jd_array = np.linspace(2460000, 2460100, 100)
times = Time(jd_array, format='jd')
```

## 时间格式

### 支持的格式

```python
# ISO 8601
t = Time('2023-01-15 12:30:45', format='iso')
t = Time('2023-01-15T12:30:45.123', format='isot')

# 儒略日
t = Time(2460000.0, format='jd')          # 儒略日
t = Time(59945.0, format='mjd')           # 简化儒略日

# 十进制年份
t = Time(2023.5, format='decimalyear')
t = Time(2023.5, format='jyear')          # 儒略年
t = Time(2023.5, format='byear')          # 贝塞尔年

# 年份与年积日
t = Time('2023:046', format='yday')       # 2023年第46天

# FITS格式
t = Time('2023-01-15T12:30:45', format='fits')

# GPS秒数
t = Time(1000000000.0, format='gps')

# Unix时间
t = Time(1673785845.0, format='unix')

# Matplotlib日期
t = Time(738521.0, format='plot_date')

# datetime对象
from datetime import datetime
dt = datetime(2023, 1, 15, 12, 30, 45)
t = Time(dt)
```

## 时间尺度

### 可用时间尺度

```python
# UTC - 协调世界时（默认）
t = Time('2023-01-15 12:00:00', scale='utc')

# TAI - 国际原子时
t = Time('2023-01-15 12:00:00', scale='tai')

# TT - 地球时
t = Time('2023-01-15 12:00:00', scale='tt')

# TDB - 质心力学时
t = Time('2023-01-15 12:00:00', scale='tdb')

# TCG - 地心坐标时
t = Time('2023-01-15 12:00:00', scale='tcg')

# TCB - 质心坐标时
t = Time('2023-01-15 12:00:00', scale='tcb')

# UT1 - 世界时
t = Time('2023-01-15 12:00:00', scale='ut1')
```

### 时间尺度转换

```python
t = Time('2023-01-15 12:00:00', scale='utc')

# 转换为不同尺度
t_tai = t.tai
t_tt = t.tt
t_tdb = t.tdb
t_ut1 = t.ut1

# 检查偏移量
print(f"TAI - UTC = {(t.tai - t.utc).sec} 秒")
# TAI - UTC = 37 秒（闰秒）
```

## 格式转换

### 更改输出格式

```python
t = Time('2023-01-15 12:30:45')

# 以不同格式访问
print(t.jd)           # 儒略日
print(t.mjd)          # 简化儒略日
print(t.iso)          # ISO格式
print(t.isot)         # 带'T'的ISO格式
print(t.unix)         # Unix时间
print(t.decimalyear)  # 十进制年份

# 更改默认格式
t.format = 'mjd'
print(t)  # 显示为简化儒略日
```

### 高精度输出

```python
# 使用subfmt控制精度
t.to_value('mjd', subfmt='float')    # 标准浮点数
t.to_value('mjd', subfmt='long')     # 扩展精度
t.to_value('mjd', subfmt='decimal')  # 十进制（最高精度）
t.to_value('mjd', subfmt='str')      # 字符串表示
```

## 时间运算

### TimeDelta对象

```python
from astropy.time import TimeDelta

# 创建时间差
dt = TimeDelta(1.0, format='jd')      # 1天
dt = TimeDelta(3600.0, format='sec')  # 1小时

# 时间相减
t1 = Time('2023-01-15')
t2 = Time('2023-02-15')
dt = t2 - t1
print(dt.jd)   # 31天
print(dt.sec)  # 2678400秒
```

### 时间加减

```python
t = Time('2023-01-15 12:00:00')

# 添加TimeDelta
t_future = t + TimeDelta(7, format='jd')  # 增加7天

# 添加Quantity
t_future = t + 1*u.hour
t_future = t + 30*u.day
t_future = t + 1*u.year

# 减去
t_past = t - 1*u.week
```

### 时间范围

```python
# 创建时间范围
start = Time('2023-01-01')
end = Time('2023-12-31')
times = start + np.linspace(0, 365, 100) * u.day

# 或使用TimeDelta
times = start + TimeDelta(np.linspace(0, 365, 100), format='jd')
```

## 观测相关功能

### 恒星时

```python
from astropy.coordinates import EarthLocation

# 定义观测者位置
location = EarthLocation(lat=40*u.deg, lon=-120*u.deg, height=1000*u.m)

# 创建带位置的时间
t = Time('2023-06-15 23:00:00', location=location)

# 计算恒星时
lst_apparent = t.sidereal_time('apparent')
lst_mean = t.sidereal_time('mean')

print(f"地方恒星时: {lst_apparent}")
```

### 光行时校正

```python
from astropy.coordinates import SkyCoord, EarthLocation

# 定义目标和观测者
target = SkyCoord(ra=10*u.deg, dec=20*u.deg)
location = EarthLocation.of_site('Keck Observatory')

# 观测时间
times = Time(['2023-01-01', '2023-06-01', '2023-12-31'],
             location=location)

# 计算至太阳系质心的光行时
ltt_bary = times.light_travel_time(target, kind='barycentric')
ltt_helio = times.light_travel_time(target, kind='heliocentric')

# 应用校正
times_barycentric = times.tdb + ltt_bary
```

### 地球自转角

```python
# 地球自转角（用于天球到地球坐标转换）
era = t.earth_rotation_angle()
```

## 处理缺失或无效时间

### 掩码时间

```python
import numpy as np

# 创建含缺失值的时间
times = Time(['2023-01-01', '2023-06-01', '2023-12-31'])
times[1] = np.ma.masked  # 标记为缺失

# 检查掩码
print(times.mask)  # [False True False]

# 获取未掩码版本
times_clean = times.unmasked

# 填充掩码值
times_filled = times.filled(Time('2000-01-01'))
```

## 时间精度与表示

### 内部表示

时间对象使用两个64位浮点数（jd1, jd2）实现高精度：

```python
t = Time('2023-01-15 12:30:45.123456789', format='iso', scale='utc')

# 访问内部表示
print(t.jd1, t.jd2)  # 整数和小数部分

# 可在天文时间尺度上保持亚纳秒级精度
```

### 精度

```python
# 长时间跨度的高精度处理
t1 = Time('1900-01-01')
t2 = Time('2100-01-01')
dt = t2 - t1
print(f"时间跨度: {dt.sec / (365.25 * 86400)} 年")
# 全程保持精度
```

## 时间格式化

### 自定义字符串格式

```python
t = Time('2023-01-15 12:30:45')

# Strftime风格格式化
t.strftime('%Y-%m-%d %H:%M:%S')  # '2023-01-15 12:30:45'
t.strftime('%B %d, %Y')          # 'January 15, 2023'

# ISO格式子类型
t.iso                    # '2023-01-15 12:30:45.000'
t.isot                   # '2023-01-15T12:30:45.000'
t.to_value('iso', subfmt='date_hms')  # '2023-01-15 12:30:45.000'
```

## 常见用例

### 格式间转换

```python
# 简化儒略日转ISO
t_mjd = Time(59945.0, format='mjd')
iso_string = t_mjd.iso

# ISO转儒略日
t_iso = Time('2023-01-15 12:00:00')
jd_value = t_iso.jd

# Unix时间转ISO
t_unix = Time(1673785845.0, format='unix')
iso_string = t_unix.iso
```

### 不同单位的时间差

```python
t1 = Time('2023-01-01')
t2 = Time('2023-12-31')

dt = t2 - t1
print(f"天数: {dt.to(u.day)}")
print(f"小时数: {dt.to(u.hour)}")
print(f"秒数: {dt.sec}")
print(f"年数: {dt.to(u.year)}")
```

### 创建规则时间序列

```python
# 全年每日观测
start = Time('2023-01-01')
times = start + np.arange(365) * u.day

# 单日每小时观测
start = Time('2023-01-15 00:00:00')
times = start + np.arange(24) * u.hour

# 每30秒观测
start = Time('2023-01-15 12:00:00')
times = start + np.arange(1000) * 30 * u.second
```

### 时区处理

```python
# UTC转本地时间（需datetime）
t = Time('2023-01-15 12:00:00', scale='utc')
dt_utc = t.to_datetime()

# 使用pytz转换特定时区
import pytz
eastern = pytz.timezone('US/Eastern')
dt_eastern = dt_utc.replace(tzinfo=pytz.utc).astimezone(eastern)
```

### 质心校正示例

```python
from astropy.coordinates import SkyCoord, EarthLocation

# 目标坐标
target = SkyCoord(ra='23h23m08.55s', dec='+18d24m59.3s')

# 观测台位置
location = EarthLocation.of_site('Keck Observatory')

# 观测时间（需包含位置）
times = Time(['2023-01-15 08:30:00', '2023-01-16 08:30:00'],
             location=location)

# 计算质心校正
ltt_bary = times.light_travel_time(target, kind='barycentric')

# 应用校正获取质心时间
times_bary = times.tdb + ltt_bary

# 用于径向速度校正
rv_correction = ltt_bary.to(u.km, equivalencies=u.dimensionless_angles())
```

## 性能考量

1. **数组操作高效**：支持批量处理多个时间点
2. **格式转换缓存**：重复访问效率高
3. **尺度转换需IERS数据**：自动下载
4. **保持高精度**：在天文时间尺度上保持亚纳秒级精度
