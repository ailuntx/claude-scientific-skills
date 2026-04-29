# NIST 数据接口

## 概述

美国国家标准与技术研究院（NIST）提供多个科学数据库。各数据集的 REST API 可用性存在差异。

## 1. NIST CODATA 基本物理常数

### 基础 URL
```
https://physics.nist.gov/cgi-bin/cuu
```

**无正式 REST API**。数据通过返回 HTML 的 CGI 脚本提供。可通过结构化 URL 编程访问常数，但响应为 HTML 而非 JSON/XML。

**替代方案——机器可读 ASCII：**
```
https://physics.nist.gov/cuu/Constants/Table/allascii.txt
```
返回包含所有基本常数及其数值、不确定度和单位的制表符分隔文本文件。

**单常数查询：**
```
https://physics.nist.gov/cgi-bin/cuu/Value?{constant_key}
```
示例键名：`bohrrada0`（玻尔半径）、`c`（光速）、`h`（普朗克常数）、`e`（电子电荷）、`me`（电子质量）、`na`（阿伏伽德罗常数）、`k`（玻尔兹曼常数）。

示例：
```
https://physics.nist.gov/cgi-bin/cuu/Value?h
```
返回 HTML 页面，需从页面内容解析数值。

**无需 API 密钥。未记录速率限制。**

## 2. NIST 原子光谱数据库 (ASD)

### 基础 URL
```
https://physics.nist.gov/cgi-bin/ASD
```

**无正式 REST API**。查询基于 CGI 返回 HTML，但可通过特定参数获取机器可读输出。

**谱线查询：**
```
https://physics.nist.gov/cgi-bin/ASD/lines1.pl?spectra={element}&low_w={min_wavelength}&upp_w={max_wavelength}&unit={unit}&format={format}
```

| 参数         | 类型   | 说明                          |
|--------------|--------|-------------------------------|
| `spectra`    | 字符串 | 元素符号或离子态（如 `H`, `Fe`, `He+I`, `O+II`） |
| `low_w`      | 浮点数 | 波长下限                      |
| `upp_w`      | 浮点数 | 波长上限                      |
| `unit`       | 整数   | `0`=埃, `1`=纳米, `2`=微米     |
| `format`     | 整数   | `0`=HTML, `1`=ASCII, `2`=CSV, `3`=制表符分隔 |
| `line_out`   | 整数   | `0`=全部, `1`=仅观测值, `2`=仅里兹值 |
| `show_obs_wl`| 整数   | `1`=显示观测波长              |
| `show_calc_wl`| 整数  | `1`=显示里兹计算波长          |
| `A_out`      | 整数   | `1`=包含跃迁概率              |

**示例——氢元素 3000-7000 埃谱线的 CSV 输出：**
```
https://physics.nist.gov/cgi-bin/ASD/lines1.pl?spectra=H&low_w=3000&upp_w=7000&unit=0&format=2&line_out=0&show_obs_wl=1&A_out=1
```

**能级查询：**
```
https://physics.nist.gov/cgi-bin/ASD/energy1.pl?spectra={element}&units={units}&format={format}
```

**无需 API 密钥。无正式速率限制，但不鼓励自动化批量查询。**

## 3. NIST 化学网络手册

### 基础 URL
```
https://webbook.nist.gov/cgi/cbook.cgi
```

**无正式 REST API**。基于 CGI 返回 HTML，可使用结构化 URL。

**名称搜索：**
```
https://webbook.nist.gov/cgi/cbook.cgi?Name={compound}&Units=SI
```

**CAS 编号搜索：**
```
https://webbook.nist.gov/cgi/cbook.cgi?ID={cas_number}&Units=SI
```

**分子式搜索：**
```
https://webbook.nist.gov/cgi/cbook.cgi?Formula={formula}&Units=SI
```

**JCAMP-DX 光谱（机器可读）：**
```
https://webbook.nist.gov/cgi/cbook.cgi?ID={cas_number}&Type=IR-Spec&Index=0&JCAMP=C{cas_no_dashes}
```

## 总结

NIST 数据库通常**不提供**现代 REST/JSON 接口。数据访问主要通过返回 HTML 或分隔文本的 CGI 端点实现。对于编程使用，ASD 的 ASCII/CSV 输出选项最为实用。所有 NIST 端点均无需认证。
