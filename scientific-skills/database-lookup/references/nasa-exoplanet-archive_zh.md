# NASA系外行星档案库API

## 基础URL

```
https://exoplanetarchive.ipac.caltech.edu
```

## 认证

无需API密钥。所有端点均为公开访问。

## 核心端点

### 1. TAP服务（推荐——当前方法）

```
GET /TAP/sync?query={ADQL}&format={format}
```

| 参数      | 类型   | 描述                  |
|-----------|--------|-----------------------|
| `query`   | string | **必填**。ADQL查询语句 |
| `format`  | string | `json`, `csv`, `votable`, `tsv`, `ipac`。默认值：`votable` |

**示例——含关键参数的已确认行星：**
```
https://exoplanetarchive.ipac.caltech.edu/TAP/sync?query=SELECT pl_name,hostname,sy_dist,pl_orbper,pl_rade,pl_bmasse,disc_year,discoverymethod FROM ps WHERE default_flag=1 ORDER BY disc_year DESC&format=json
```

**示例——宜居带行星（粗略估计）：**
```
https://exoplanetarchive.ipac.caltech.edu/TAP/sync?query=SELECT TOP 50 pl_name,hostname,pl_orbsmax,st_teff,pl_rade FROM ps WHERE default_flag=1 AND pl_orbsmax BETWEEN 0.8 AND 1.5 AND st_teff BETWEEN 4000 AND 7000&format=json
```

**示例——TESS发现的行星：**
```
https://exoplanetarchive.ipac.caltech.edu/TAP/sync?query=SELECT pl_name,pl_rade,pl_orbper,disc_year FROM ps WHERE default_flag=1 AND disc_facility='Transiting Exoplanet Survey Satellite (TESS)'&format=json
```

**示例——按发现方法统计行星数量：**
```
https://exoplanetarchive.ipac.caltech.edu/TAP/sync?query=SELECT discoverymethod, COUNT(*) as cnt FROM ps WHERE default_flag=1 GROUP BY discoverymethod ORDER BY cnt DESC&format=json
```

### 2. 旧版API（较旧，但仍可用）

```
GET /cgi-bin/nstedAPI/nph-nstedAPI?table={table}&format={format}&where={conditions}&select={columns}
```

**示例：**
```
https://exoplanetarchive.ipac.caltech.edu/cgi-bin/nstedAPI/nph-nstedAPI?table=ps&select=pl_name,pl_orbper,pl_rade&where=disc_year=2023&format=json
```

注意：旧版API已弃用，推荐使用TAP。新应用请使用TAP。

## 核心TAP数据表

| 表名          | 描述 |
|---------------|------|
| `ps`          | **行星系统**——每个参考文献对应每颗行星占一行。使用`default_flag=1`获取默认/最佳参数集 |
| `pscomppars`  | **行星系统复合参数**——每颗行星占一行，包含多篇文献的最佳拟合值 |
| `stellarhosts`| 宿主恒星的恒星属性 |
| `td`          | 时间序列数据（凌星曲线、径向速度曲线） |
| `keplernames` | 开普勒目标交叉引用 |
| `k2names`     | K2观测任务交叉引用 |
| `toi`         | TESS关注目标 |

## 核心字段（ps表）

| 字段             | 描述 |
|------------------|------|
| `pl_name`        | 行星名称（如"Kepler-22 b"） |
| `hostname`       | 宿主恒星名称 |
| `default_flag`   | 1表示该行星的默认参数集 |
| `disc_year`      | 发现年份 |
| `discoverymethod`| `凌星法`, `径向速度法`, `直接成像法`, `微引力透镜法`等 |
| `pl_orbper`      | 轨道周期（天） |
| `pl_orbsmax`     | 轨道半长轴（天文单位） |
| `pl_rade`        | 行星半径（地球半径） |
| `pl_bmasse`      | 行星质量（地球质量） |
| `pl_eqt`         | 平衡温度（开尔文） |
| `sy_dist`        | 系统距离（秒差距） |
| `st_teff`        | 恒星有效温度（开尔文） |
| `st_rad`         | 恒星半径（太阳半径） |
| `st_mass`        | 恒星质量（太阳质量） |
| `disc_facility`  | 发现设备名称 |

## 响应格式（TAP JSON）

```json
{
  "metadata": [
    {"name": "pl_name", "datatype": "char"},
    {"name": "pl_orbper", "datatype": "double"}
  ],
  "data": [
    ["Kepler-22 b", 289.8623]
  ]
}
```

## 速率限制

无需API密钥或认证。未规定正式速率限制，但档案库要求用户避免过度自动化查询。大型结果集可能导致超时；建议在ADQL中使用`TOP N`或通过`OFFSET`和`MAXREC`分页。

如需批量下载，请使用以下接口：
```
https://exoplanetarchive.ipac.caltech.edu/cgi-bin/TblView/nph-tblView?app=ExoTbls&config=PS
```
