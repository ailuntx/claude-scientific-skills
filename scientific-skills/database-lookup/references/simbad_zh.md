# SIMBAD 天文数据库（CDS 斯特拉斯堡）

## 基础 URL

**TAP 端点（推荐）：**
```
https://simbad.cds.unistra.fr/simbad/sim-tap/sync
```

**传统脚本接口：**
```
https://simbad.cds.unistra.fr/simbad/sim-script
```

**简单查询端点：**
```
https://simbad.cds.unistra.fr/simbad/sim-id
https://simbad.cds.unistra.fr/simbad/sim-coo
```

## 认证

无需 API 密钥。所有端点均为公开访问。

## 关键端点

### 1. TAP 查询（ADQL — 推荐用于程序化调用）

```
GET /simbad/sim-tap/sync?request=doQuery&lang=adql&format={format}&query={ADQL}
```

| 参数       | 类型   | 说明          |
|------------|--------|---------------|
| `request`  | string | `doQuery`     |
| `lang`     | string | `adql`        |
| `format`   | string | `json`, `votable`, `csv`, `tsv` |
| `query`    | string | **必填** ADQL 查询语句 |

**示例 — 按名称查询天体：**
```
https://simbad.cds.unistra.fr/simbad/sim-tap/sync?request=doQuery&lang=adql&format=json&query=SELECT basic.OID, ra, dec, main_id, otype FROM basic JOIN ident ON oid = ident.oidref WHERE id = 'M31'
```

**示例 — 锥形搜索（坐标周围 5 角分内的天体）：**
```
https://simbad.cds.unistra.fr/simbad/sim-tap/sync?request=doQuery&lang=adql&format=json&query=SELECT TOP 50 main_id, ra, dec, otype FROM basic WHERE CONTAINS(POINT('ICRS', ra, dec), CIRCLE('ICRS', 10.684, 41.269, 0.083)) = 1
```
注：CIRCLE 中的半径单位为度（5 角分 = 0.083 度）。

**示例 — 按类型查询天体（如所有脉冲星）：**
```
https://simbad.cds.unistra.fr/simbad/sim-tap/sync?request=doQuery&lang=adql&format=json&query=SELECT TOP 100 main_id, ra, dec, otype FROM basic WHERE otype = 'Pulsar'
```

### 2. 标识符查询（简单检索）

```
GET /simbad/sim-id?Ident={name}&output.format=votable
```

| 参数             | 类型   | 说明                          |
|------------------|--------|-------------------------------|
| `Ident`          | string | **必填** 天体名称（如 `M31`, `Sirius`, `NGC 1275`） |
| `output.format`  | string | `votable`, `html`             |

**示例：**
```
https://simbad.cds.unistra.fr/simbad/sim-id?Ident=M31&output.format=votable
```

### 3. 坐标查询

```
GET /simbad/sim-coo?Coord={coords}&Radius={radius}&Radius.unit={unit}&output.format=votable
```

| 参数            | 类型   | 说明                                      |
|-----------------|--------|-------------------------------------------|
| `Coord`         | string | **必填** 坐标（如 `10.684 +41.269` 或 `00 42 44 +41 16 09`） |
| `Radius`        | float  | 搜索半径（默认值：2）                     |
| `Radius.unit`   | string | `arcmin`, `arcsec`, `deg`（默认：`arcmin`） |
| `output.format` | string | `votable`, `html`                         |

**示例：**
```
https://simbad.cds.unistra.fr/simbad/sim-coo?Coord=10.684+%2B41.269&Radius=5&Radius.unit=arcmin&output.format=votable
```

### 4. 脚本接口（用于多命令查询）

```
POST /simbad/sim-script
Content-Type: application/x-www-form-urlencoded
script=format+object+"%MAIN_ID+|+%RA+|+%DEC+|+%OTYPE"\nquery+id+M31
```

## 关键 TAP 数据表

| 表名            | 说明                     |
|-----------------|--------------------------|
| `basic`         | 核心数据：坐标、主标识符、天体类型 |
| `ident`         | 天体的所有已知标识符       |
| `flux`          | 流量/星等测量值           |
| `mesVelocities` | 径向速度测量值            |
| `mesDistance`   | 距离测量值                |
| `otypedef`      | 天体类型定义/标签         |
| `allfluxes`     | 合并所有流量数据          |

## 常见天体类型（otype）

`恒星`, `星系`, `脉冲星`, `类星体`, `星云`, `球状星团`, `射电源`, `X射线源`, `超新星遗迹`

## 响应格式（TAP JSON）

```json
{
  "metadata": [
    {"name": "main_id", "datatype": "char"},
    {"name": "ra", "datatype": "double"},
    {"name": "dec", "datatype": "double"}
  ],
  "data": [
    ["M 31", 10.6847, 41.2687]
  ]
}
```

## 速率限制

无正式速率限制规定。SIMBAD 要求自动化脚本在查询间保持合理延迟。大型 TAP 查询可能超时，建议使用 `TOP N` 限制结果数量或切换至异步 TAP 端点 `/simbad/sim-tap/async`。
