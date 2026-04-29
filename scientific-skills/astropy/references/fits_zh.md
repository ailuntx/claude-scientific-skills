# FITS 文件处理 (astropy.io.fits)

`astropy.io.fits` 模块提供了一套完整的工具，用于读取、写入和操作 FITS（灵活图像传输系统）文件。

## 打开 FITS 文件

### 基本文件打开方式

```python
from astropy.io import fits

# 打开文件（返回 HDUList - HDU 列表）
hdul = fits.open('filename.fits')

# 操作完成后务必关闭
hdul.close()

# 更佳方式：使用上下文管理器（自动关闭）
with fits.open('filename.fits') as hdul:
    hdul.info()  # 显示文件结构
    data = hdul[0].data
```

### 文件打开模式

```python
fits.open('file.fits', mode='readonly')   # 只读模式（默认）
fits.open('file.fits', mode='update')     # 读写模式
fits.open('file.fits', mode='append')     # 追加模式（添加新 HDU）
```

### 内存映射

处理大文件时使用内存映射（默认行为）：

```python
hdul = fits.open('large_file.fits', memmap=True)
# 仅按需加载数据块
```

### 远程文件

访问云端托管的 FITS 文件：

```python
uri = "s3://bucket-name/image.fits"
with fits.open(uri, use_fsspec=True, fsspec_kwargs={"anon": True}) as hdul:
    # 使用 .section 获取切片而无需下载整个文件
    cutout = hdul[1].section[100:200, 100:200]
```

## HDU 结构

FITS 文件包含头数据单元（HDU）：
- **主 HDU** (`hdul[0]`)：首个 HDU，始终存在
- **扩展 HDU** (`hdul[1:]`)：图像或表格扩展

```python
hdul.info()  # 显示所有 HDU
# 输出示例：
# No.    Name      Ver    Type      Cards   Dimensions   Format
#  0  PRIMARY       1 PrimaryHDU     220   ()
#  1  SCI           1 ImageHDU       140   (1014, 1014)   float32
#  2  ERR           1 ImageHDU        51   (1014, 1014)   float32
```

## 访问 HDU

```python
# 通过索引访问
primary = hdul[0]
extension1 = hdul[1]

# 通过名称访问
sci = hdul['SCI']

# 通过名称和版本号访问
sci2 = hdul['SCI', 2]  # 第二个 SCI 扩展
```

## 处理头信息

### 读取头信息值

```python
hdu = hdul[0]
header = hdu.header

# 获取关键字值（不区分大小写）
observer = header['OBSERVER']
exptime = header['EXPTIME']

# 获取值（若缺失则返回默认值）
filter_name = header.get('FILTER', 'Unknown')

# 通过索引访问
value = header[7]  # 第 8 个卡的值
```

### 修改头信息

```python
# 更新现有关键字
header['OBSERVER'] = 'Edwin Hubble'

# 添加/更新带注释的关键字
header['OBSERVER'] = ('Edwin Hubble', '观测者姓名')

# 在指定位置插入关键字
header.insert(5, ('NEWKEY', 'value', '注释'))

# 添加 HISTORY 和 COMMENT
header['HISTORY'] = '文件处理于 2025-01-15'
header['COMMENT'] = '数据相关说明'

# 删除关键字
del header['OLDKEY']
```

### 头信息卡片

每个关键字存储为 "卡片"（80 字符记录）：

```python
# 访问完整卡片
card = header.cards[0]
print(f"{card.keyword} = {card.value} / {card.comment}")

# 遍历所有卡片
for card in header.cards:
    print(f"{card.keyword}: {card.value}")
```

## 处理图像数据

### 读取图像数据

```python
# 从 HDU 获取数据
data = hdul[1].data  # 返回 NumPy 数组

# 数据属性
print(data.shape)      # 例如 (1024, 1024)
print(data.dtype)      # 例如 float32
print(data.min(), data.max())

# 访问特定像素
pixel_value = data[100, 200]
region = data[100:200, 300:400]
```

### 数据操作

数据是 NumPy 数组，可使用标准 NumPy 操作：

```python
import numpy as np

# 统计计算
mean = np.mean(data)
median = np.median(data)
std = np.std(data)

# 修改数据
data[data < 0] = 0  # 裁剪负值
data = data * gain + bias  # 校准

# 数学运算
log_data = np.log10(data)
smoothed = scipy.ndimage.gaussian_filter(data, sigma=2)
```

### 切片与区域提取

无需加载整个数组即可提取区域：

```python
# 切片表示法 [y起始:y结束, x起始:x结束]
cutout = hdul[1].section[500:600, 700:800]
```

## 创建新 FITS 文件

### 简单图像文件

```python
# 创建数据
data = np.random.random((100, 100))

# 创建 HDU
hdu = fits.PrimaryHDU(data=data)

# 添加头信息关键字
hdu.header['OBJECT'] = '测试图像'
hdu.header['EXPTIME'] = 300.0

# 写入文件
hdu.writeto('new_image.fits')

# 覆盖已存在文件
hdu.writeto('new_image.fits', overwrite=True)
```

### 多扩展文件

```python
# 创建主 HDU（可无数据）
primary = fits.PrimaryHDU()
primary.header['TELESCOP'] = 'HST'

# 创建图像扩展
sci_data = np.ones((100, 100))
sci = fits.ImageHDU(data=sci_data, name='SCI')

err_data = np.ones((100, 100)) * 0.1
err = fits.ImageHDU(data=err_data, name='ERR')

# 组合为 HDUList
hdul = fits.HDUList([primary, sci, err])

# 写入文件
hdul.writeto('multi_extension.fits')
```

## 处理表格数据

### 读取表格

```python
# 打开表格
with fits.open('table.fits') as hdul:
    table = hdul[1].data  # BinTableHDU 或 TableHDU

    # 访问列数据
    ra = table['RA']
    dec = table['DEC']
    mag = table['MAG']

    # 访问行数据
    first_row = table[0]
    subset = table[10:20]

    # 列信息
    cols = hdul[1].columns
    print(cols.names)
    cols.info()
```

### 创建表格

```python
# 定义列
col1 = fits.Column(name='ID', format='K', array=[1, 2, 3, 4])
col2 = fits.Column(name='RA', format='D', array=[10.5, 11.2, 12.3, 13.1])
col3 = fits.Column(name='DEC', format='D', array=[41.2, 42.1, 43.5, 44.2])
col4 = fits.Column(name='Name', format='20A',
                   array=['Star1', 'Star2', 'Star3', 'Star4'])

# 创建表格 HDU
table_hdu = fits.BinTableHDU.from_columns([col1, col2, col3, col4])
table_hdu.name = 'CATALOG'

# 写入文件
table_hdu.writeto('catalog.fits', overwrite=True)
```

### 列格式

常用 FITS 表格列格式：
- `'A'`：字符串（例如 '20A' 表示 20 字符）
- `'L'`：逻辑值（布尔型）
- `'B'`：无符号字节
- `'I'`：16 位整数
- `'J'`：32 位整数
- `'K'`：64 位整数
- `'E'`：32 位浮点数
- `'D'`：64 位浮点数

## 修改现有文件

### 更新模式

```python
with fits.open('file.fits', mode='update') as hdul:
    # 修改头信息
    hdul[0].header['NEWKEY'] = 'value'

    # 修改数据
    hdul[1].data[100, 100] = 999

    # 上下文退出时自动保存更改
```

### 追加模式

```python
# 向现有文件添加新扩展
new_data = np.random.random((50, 50))
new_hdu = fits.ImageHDU(data=new_data, name='NEW_EXT')

with fits.open('file.fits', mode='append') as hdul:
    hdul.append(new_hdu)
```

## 便捷函数

无需管理 HDU 列表的快速操作：

```python
# 仅获取数据
data = fits.getdata('file.fits', ext=1)

# 仅获取头信息
header = fits.getheader('file.fits', ext=0)

# 同时获取数据和头信息
data, header = fits.getdata('file.fits', ext=1, header=True)

# 获取单个关键字值
exptime = fits.getval('file.fits', 'EXPTIME', ext=0)

# 设置关键字值
fits.setval('file.fits', 'NEWKEY', value='newvalue', ext=0)

# 写入简单文件
fits.writeto('output.fits', data, header, overwrite=True)

# 追加到文件
fits.append('file.fits', data, header)

# 显示文件信息
fits.info('file.fits')
```

## 比较 FITS 文件

```python
# 打印两个文件的差异
fits.printdiff('file1.fits', 'file2.fits')

# 编程方式比较
diff = fits.FITSDiff('file1.fits', 'file2.fits')
print(diff.report())
```

## 格式转换

### FITS 与 Astropy 表格互转

```python
from astropy.table import Table

# FITS 转表格
table = Table.read('catalog.fits')

# 表格转 FITS
table.write('output.fits', format='fits', overwrite=True)
```

## 最佳实践

1. **始终使用上下文管理器**（`with` 语句）确保文件安全处理
2. **避免修改结构关键字**（SIMPLE, BITPIX, NAXIS 等）
3. **大文件使用内存映射**以节省内存
4. **远程文件使用 .section**避免完整下载
5. **访问数据前用 `.info()` 检查 HDU 结构**
6. **操作前验证数据类型**避免意外行为
7. **简单操作使用便捷函数**

## 常见问题

### 处理非标准 FITS

处理不符合 FITS 标准的文件：

```python
# 忽略验证警告
hdul = fits.open('bad_file.fits', ignore_missing_end=True)

# 修复非标准文件
hdul = fits.open('bad_file.fits')
hdul.verify('fix')  # 尝试修复问题
hdul.writeto('fixed_file.fits')
```

### 大文件性能优化

```python
# 使用内存映射（默认）
hdul = fits.open('huge_file.fits', memmap=True)

# 大数组写入操作使用 Dask
import dask.array as da
large_array = da.random.random((10000, 10000))
fits.writeto('output.fits', large_array)
```
