# SDSS SkyServer API

## 基础 URL

```
https://skyserver.sdss.org/dr18/SkyServerWS
```

将 `dr18` 替换为所需的数据发布版本（例如 `dr17`、`dr16`）。

## 认证

无需 API 密钥。所有端点均为公开访问。

## 关键端点

### 1. SQL 搜索（CasJobs 风格自由格式 SQL）

```
GET /SearchTools/SqlSearch
```

| 参数      | 类型   | 描述                          |
|-----------|--------|-----------------------------|
| `cmd`     | string | **必填**。针对 SDSS CasJobs 模式的 SQL 查询。 |
| `format`  | string | `json`、`xml`、`csv`、`html`、`votable`。默认值：`html`。 |

**示例 — 查询 10 个星系：**
```
https://skyserver.sdss.org/dr18/SkyServerWS/SearchTools/SqlSearch?cmd=SELECT TOP 10 objid,ra,dec,u,g,r,i,z FROM PhotoObj WHERE type=3&format=json
```

类型代码：`3` = 星系，`6` = 恒星。

**响应（JSON）：**
```json
[
  {"Rows": [
    {"objid": 1237645941825863680, "ra": 195.123, "dec": 2.456, "u": 22.1, "g": 20.8, "r": 19.5, "i": 19.1, "z": 18.9}
  ]}
]
```

### 2. 径向搜索

```
GET /SearchTools/RadialSearch
```

| 参数        | 类型   | 描述                          |
|-------------|--------|-----------------------------|
| `ra`        | float  | **必填**。赤经（度）。          |
| `dec`       | float  | **必填**。赤纬（度）。          |
| `radius`    | float  | 搜索半径（角分）。默认值：1。    |
| `format`    | string | `json`、`xml`、`csv`。         |
| `limit`     | int    | 最大结果数。                   |
| `objtype`   | string | 筛选条件：`star`、`galaxy` 或留空表示全部。 |

**示例 — 在 RA=180, Dec=+0.5 周围 2 角分内的天体：**
```
https://skyserver.sdss.org/dr18/SkyServerWS/SearchTools/RadialSearch?ra=180&dec=0.5&radius=2&format=json&limit=10
```

### 3. 矩形区域搜索

```
GET /SearchTools/RectangularSearch
```

| 参数      | 类型   | 描述                  |
|-----------|--------|---------------------|
| `min_ra`  | float  | 最小赤经（度）。      |
| `max_ra`  | float  | 最大赤经（度）。      |
| `min_dec` | float  | 最小赤纬（度）。      |
| `max_dec` | float  | 最大赤纬（度）。      |
| `format`  | string | `json`、`xml`、`csv`。 |
| `limit`   | int    | 最大结果数。         |

### 4. 通过 ObjID 查找天体

```
GET /SearchTools/SqlSearch?cmd=SELECT * FROM PhotoObj WHERE objid={objid}&format=json
```

### 5. 通过 Plate-MJD-Fiber 搜索光谱

```
GET /SearchTools/SqlSearch?cmd=SELECT * FROM SpecObj WHERE plate={plate} AND mjd={mjd} AND fiberid={fiberid}&format=json
```

### 6. 图像截取服务

```
GET /ImgCutout/getjpeg
```

| 参数      | 类型   | 描述                      |
|-----------|--------|-------------------------|
| `ra`      | float  | **必填**。赤经（度）。    |
| `dec`     | float  | **必填**。赤纬（度）。    |
| `scale`   | float  | 角秒/像素。默认值：0.396127。 |
| `width`   | int    | 图像宽度（像素）。默认值：512。 |
| `height`  | int    | 图像高度（像素）。默认值：512。 |

**示例：**
```
https://skyserver.sdss.org/dr18/SkyServerWS/ImgCutout/getjpeg?ra=180.0&dec=0.5&scale=0.4&width=256&height=256
```

返回 JPEG 图像数据。

### 7. 光谱图/数据

光谱 FITS 文件可从科学档案服务器获取：
```
https://data.sdss.org/sas/dr18/spectro/sdss/redux/{run2d}/spectra/{plate}/spec-{plate}-{mjd}-{fiberid}.fits
```

## 重要 SQL 表

| 表名        | 描述                     |
|-------------|------------------------|
| `PhotoObj`  | 测光测量（位置、星等）。   |
| `SpecObj`   | 光谱测量（红移、分类）。   |
| `Galaxy`    | 筛选为星系的 PhotoObj 视图。 |
| `Star`      | 筛选为恒星的 PhotoObj 视图。 |

## 速率限制

无正式文档化的速率限制。返回超大结果集的查询可能超时。在 SQL 查询中使用 `TOP N` 限制结果数量。批量数据请使用 CasJobs (https://skyserver.sdss.org/CasJobs/) 并注册免费账户。
