# 晶体学开放数据库（COD）API

## 基础 URL

```
https://www.crystallography.net/cod
```

## 认证

**无需认证**。COD 是完全开放访问的数据库，无需 API 密钥。

## 核心端点

### 按化学式搜索

```
GET /result?formula=Fe2%20O3&format=json
```

化学式格式需在元素间添加空格：`Fe2 O3`、`Si O2`、`C6 H12 O6`。空格需进行 URL 编码为 `%20`。

### 按元素搜索

```
GET /result?el1=Fe&el2=O&format=json
```

使用 `el1`, `el2`, `el3` 等参数过滤元素。通过 `nel=2` 可限定为恰好包含 2 种元素。

### 按晶胞参数搜索

```
GET /result?a_min=5.0&a_max=6.0&b_min=5.0&b_max=6.0&c_min=5.0&c_max=6.0&format=json
```

晶胞参数过滤器：
- `a_min`, `a_max` — a轴长度（埃）
- `b_min`, `b_max` — b轴长度
- `c_min`, `c_max` — c轴长度
- `alpha_min`, `alpha_max` — alpha 角（度）
- `beta_min`, `beta_max` — beta 角
- `gamma_min`, `gamma_max` — gamma 角
- `vol_min`, `vol_max` — 晶胞体积（A^3）

### 按空间群搜索

```
GET /result?sg=F%20m%20-3%20m&format=json
```

### 按文本搜索（作者/期刊/标题）

```
GET /result?text=perovskite&format=json
```

### 组合搜索示例

```
GET /result?el1=Ti&el2=O&nel=2&sg=P%2042/m%20n%20m&format=json
```

### 获取特定 CIF 文件

```
GET /1000000.cif
```

COD ID 为 7 位整数。追加 `.cif` 获取晶体学信息文件，或追加 `.html` 获取网页。

### 获取条目元数据（JSON格式）

```
GET /result?id=1000000&format=json
```

### 输出格式

- `format=json` — 匹配条目的 JSON 数组
- `format=csv` — CSV 格式输出
- `format=lst` — 仅返回 COD ID 列表
- 默认（无格式参数）— HTML 页面

## 响应格式

```json
[
  {
    "file": "1526463",
    "a": "4.759",
    "b": "4.759",
    "c": "12.992",
    "alpha": "90",
    "beta": "90",
    "gamma": "120",
    "vol": "254.94",
    "sg": "R -3 c",
    "formula": "Fe2 O3",
    "title": "Refinement of the crystal structure of ...",
    "journal": "Zeitschrift fuer Kristallographie",
    "year": "1966",
    "authors": "Blake, R.L.; et al."
  }
]
```

`file` 字段为 COD ID。通过此 ID 获取 CIF 文件：`https://www.crystallography.net/cod/{file}.cif`

## 速率限制

- 无官方速率限制说明
- 请保持礼貌：避免快速批量下载数千条条目
- 批量访问建议使用 COD 提供的数据库快照：https://www.crystallography.net/cod/archives/

## 注意事项

- COD 包含约 50 万+ 来自公开文献的晶体结构
- 所有数据均在公共领域/开放许可下开放访问
- 搜索 API 返回元数据；需使用 CIF 端点获取完整结构数据
- 替代访问方式：可通过 MySQL 数据库快照和 SVN 进行批量使用
