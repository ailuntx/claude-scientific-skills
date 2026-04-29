# 表格操作 (astropy.table)

`astropy.table` 模块提供了处理表格数据的灵活工具，支持单位、掩码值和多种文件格式。

## 创建表格

### 基础表格创建

```python
from astropy.table import Table, QTable
import astropy.units as u
import numpy as np

# 从列数组创建
a = [1, 4, 5]
b = [2.0, 5.0, 8.2]
c = ['x', 'y', 'z']

t = Table([a, b, c], names=('id', 'flux', 'name'))

# 带单位创建（使用 QTable）
flux = [1.2, 2.3, 3.4] * u.Jy
wavelength = [500, 600, 700] * u.nm
t = QTable([flux, wavelength], names=('flux', 'wavelength'))
```

### 从行列表创建

```python
# 元组列表
rows = [(1, 10.5, 'A'), (2, 11.2, 'B'), (3, 12.3, 'C')]
t = Table(rows=rows, names=('id', 'value', 'name'))

# 字典列表
rows = [{'id': 1, 'value': 10.5}, {'id': 2, 'value': 11.2}]
t = Table(rows)
```

### 从 NumPy 数组创建

```python
# 结构化数组
arr = np.array([(1, 2.0, 'x'), (4, 5.0, 'y')],
               dtype=[('a', 'i4'), ('b', 'f8'), ('c', 'U10')])
t = Table(arr)

# 二维数组带列名
data = np.random.random((100, 3))
t = Table(data, names=['col1', 'col2', 'col3'])
```

### 从 Pandas 数据框创建

```python
import pandas as pd

df = pd.DataFrame({'a': [1, 2, 3], 'b': [4, 5, 6]})
t = Table.from_pandas(df)
```

## 访问表格数据

### 基础访问

```python
# 列访问
ra_col = t['ra']           # 返回列对象
dec_col = t['dec']

# 行访问
first_row = t[0]           # 返回行对象
row_slice = t[10:20]       # 返回新表格

# 单元格访问
value = t['ra'][5]         # 'ra'列的第6个值
value = t[5]['ra']         # 同上

# 多列访问
subset = t['ra', 'dec', 'mag']
```

### 表格属性

```python
len(t)              # 行数
t.colnames          # 列名列表
t.dtype             # 列数据类型
t.info              # 详细信息
t.meta              # 元数据字典
```

### 迭代操作

```python
# 行迭代
for row in t:
    print(row['ra'], row['dec'])

# 列迭代
for colname in t.colnames:
    print(t[colname])
```

## 修改表格

### 添加列

```python
# 添加新列
t['new_col'] = [1, 2, 3, 4, 5]
t['calc'] = t['a'] + t['b']  # 计算列

# 添加带单位的列
t['velocity'] = [10, 20, 30] * u.km / u.s

# 添加空列
from astropy.table import Column
t['empty'] = Column(length=len(t), dtype=float)

# 指定位置插入
t.add_column([7, 8, 9], name='inserted', index=2)
```

### 删除列

```python
# 删除单列
t.remove_column('old_col')

# 删除多列
t.remove_columns(['col1', 'col2'])

# 删除语法
del t['col_name']

# 仅保留指定列
t.keep_columns(['ra', 'dec', 'mag'])
```

### 重命名列

```python
t.rename_column('old_name', 'new_name')

# 批量重命名
t.rename_columns(['old1', 'old2'], ['new1', 'new2'])
```

### 添加行

```python
# 添加单行
t.add_row([1, 2.5, 'new'])

# 以字典形式添加行
t.add_row({'ra': 10.5, 'dec': 41.2, 'mag': 18.5})

# 注意：逐行添加效率较低！
# 建议先收集行再批量创建表格
```

### 修改数据

```python
# 修改整列值
t['flux'] = t['flux'] * gain
t['mag'][t['mag'] < 0] = np.nan

# 修改单个单元格
t['ra'][5] = 10.5

# 修改整行
t[0] = [new_id, new_ra, new_dec]
```

## 排序与筛选

### 排序

```python
# 单列排序
t.sort('mag')

# 降序排序
t.sort('mag', reverse=True)

# 多列排序
t.sort(['priority', 'mag'])

# 获取排序索引（不修改原表）
indices = t.argsort('mag')
sorted_table = t[indices]
```

### 筛选

```python
# 布尔索引
bright = t[t['mag'] < 18]
nearby = t[t['distance'] < 100*u.pc]

# 多条件筛选
selected = t[(t['mag'] < 18) & (t['dec'] > 0)]

# 使用 NumPy 函数
high_snr = t[np.abs(t['flux'] / t['error']) > 5]
```

## 文件读写

### 支持格式

FITS, HDF5, ASCII (CSV, ECSV, IPAC 等), VOTable, Parquet, ASDF

### 读取文件

```python
# 自动检测格式
t = Table.read('catalog.fits')
t = Table.read('data.csv')
t = Table.read('table.vot')

# 显式指定格式
t = Table.read('data.txt', format='ascii')
t = Table.read('catalog.hdf5', path='/dataset/table')

# 读取 FITS 指定 HDU
t = Table.read('file.fits', hdu=2)
```

### 写入文件

```python
# 根据扩展名自动选择格式
t.write('output.fits')
t.write('output.csv')

# 指定格式
t.write('output.txt', format='ascii.csv')
t.write('output.hdf5', path='/data/table', serialize_meta=True)

# 覆盖现有文件
t.write('output.fits', overwrite=True)
```

### ASCII 格式选项

```python
# 自定义分隔符的 CSV
t.write('output.csv', format='ascii.csv', delimiter='|')

# 固定宽度格式
t.write('output.txt', format='ascii.fixed_width')

# IPAC 格式
t.write('output.tbl', format='ascii.ipac')

# LaTeX 表格
t.write('table.tex', format='ascii.latex')
```

## 表格操作

### 垂直堆叠表格

```python
from astropy.table import vstack

# 垂直拼接表格
t1 = Table([[1, 2], [3, 4]], names=('a', 'b'))
t2 = Table([[5, 6], [7, 8]], names=('a', 'b'))
t_combined = vstack([t1, t2])
```

### 水平连接表格

```python
from astropy.table import hstack

# 水平拼接表格
t1 = Table([[1, 2]], names=['a'])
t2 = Table([[3, 4]], names=['b'])
t_combined = hstack([t1, t2])
```

### 数据库式连接

```python
from astropy.table import join

# 公共列内连接
t1 = Table([[1, 2, 3], ['a', 'b', 'c']], names=('id', 'data1'))
t2 = Table([[1, 2, 4], ['x', 'y', 'z']], names=('id', 'data2'))
t_joined = join(t1, t2, keys='id')

# 左/右/外连接
t_joined = join(t1, t2, join_type='left')
t_joined = join(t1, t2, join_type='outer')
```

### 分组与聚合

```python
# 按列分组
g = t.group_by('filter')

# 聚合分组
means = g.groups.aggregate(np.mean)

# 遍历分组
for group in g.groups:
    print(f"Filter: {group['filter'][0]}")
    print(f"Mean mag: {np.mean(group['mag'])}")
```

### 唯一行

```python
# 获取唯一行
t_unique = t.unique('id')

# 多列唯一值
t_unique = t.unique(['ra', 'dec'])
```

## 单位与量值

使用 QTable 进行单位感知操作：

```python
from astropy.table import QTable

# 创建带单位表格
t = QTable()
t['flux'] = [1.2, 2.3, 3.4] * u.Jy
t['wavelength'] = [500, 600, 700] * u.nm

# 单位转换
t['flux'].to(u.mJy)
t['wavelength'].to(u.angstrom)

# 计算保留单位
t['freq'] = t['wavelength'].to(u.Hz, equivalencies=u.spectral())
```

## 缺失数据掩码

```python
from astropy.table import MaskedColumn
import numpy as np

# 创建掩码列
flux = MaskedColumn([1.2, np.nan, 3.4], mask=[False, True, False])
t = Table([flux], names=['flux'])

# 操作自动处理掩码
mean_flux = np.ma.mean(t['flux'])

# 填充掩码值
t['flux'].filled(0)  # 用0替换掩码值
```

## 快速查找索引

创建索引实现快速行检索：

```python
# 为列添加索引
t.add_index('id')

# 通过索引快速查找
row = t.loc[12345]  # 查找 id=12345 的行

# 范围查询
subset = t.loc[100:200]
```

## 表格元数据

```python
# 设置表格级元数据
t.meta['TELESCOPE'] = 'HST'
t.meta['FILTER'] = 'F814W'
t.meta['EXPTIME'] = 300.0

# 设置列级元数据
t['ra'].meta['unit'] = 'deg'
t['ra'].meta['description'] = '赤经'
t['ra'].description = '赤经'  # 快捷方式
```

## 性能优化技巧

### 快速构建表格

```python
# 慢：逐行添加
t = Table(names=['a', 'b'])
for i in range(1000):
    t.add_row([i, i**2])

# 快：从列表批量构建
rows = [(i, i**2) for i in range(1000)]
t = Table(rows=rows, names=['a', 'b'])
```

### 内存映射 FITS 表格

```python
# 不将整个表加载到内存
t = Table.read('huge_catalog.fits', memmap=True)

# 仅在访问时加载数据
subset = t[10000:10100]  # 高效方式
```

### 复制与视图

```python
# 创建视图（共享数据，快速）
t_view = t['ra', 'dec']

# 创建副本（独立数据）
t_copy = t['ra', 'dec'].copy()
```

## 表格展示

```python
# 控制台打印
print(t)

# 在浏览器中显示
t.show_in_browser()
t.show_in_browser(jsviewer=True)  # 交互式排序/筛选

# 分页查看
t.more()

# 自定义格式化
t['flux'].format = '%.3f'
t['ra'].format = '{:.6f}'
```

## 格式转换

```python
# 转为 NumPy 数组
arr = np.array(t)

# 转为 Pandas 数据框
df = t.to_pandas()

# 转为字典
d = {name: t[name] for name in t.colnames}
```

## 常见用例

### 星表交叉匹配

```python
from astropy.coordinates import SkyCoord, match_coordinates_sky

# 从表格列创建坐标对象
coords1 = SkyCoord(t1['ra'], t1['dec'], unit='deg')
coords2 = SkyCoord(t2['ra'], t2['dec'], unit='deg')

# 查找匹配
idx, sep, _ = coords1.match_to_catalog_sky(coords2)

# 按角距筛选
max_sep = 1 * u.arcsec
matches = sep < max_sep
t1_matched = t1[matches]
t2_matched = t2[idx[matches]]
```

### 数据分箱

```python
from astropy.table import Table
import numpy as np

# 按星等分箱
mag_bins = np.arange(10, 20, 0.5)
binned = t.group_by(np.digitize(t['mag'], mag_bins))
counts = binned.groups.aggregate(len)
```
