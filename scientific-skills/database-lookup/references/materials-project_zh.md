# Materials Project API

## 基础URL

```
https://api.materialsproject.org
```

## 认证

需要免费API密钥。在 https://materialsproject.org 注册（免费账户）。

| 环境变量 | 请求头 |
|---|---|
| `MP_API_KEY` | `X-API-KEY: your_key_here` |

所有请求必须包含API密钥请求头。

## API版本

当前API为**v2**（基于`mp-api` Python客户端和新的MAPI端点）。位于`https://www.materialsproject.org/rest/v2/`的旧版v1 REST API已被弃用。

## 核心端点

### 通过化学式或元素搜索材料

```
GET /materials/summary/?formula=Fe2O3&_fields=material_id,formula_pretty,band_gap,formation_energy_per_atom
```

```
GET /materials/summary/?elements=Si,O&_fields=material_id,formula_pretty,band_gap
```

查询参数：
- `formula` — 精确化学式（如`Fe2O3`、`SiO2`）
- `chemsys` — 化学体系，短横线分隔（如`Fe-O`、`Li-Fe-P-O`）
- `elements` — 必须存在的元素，逗号分隔
- `band_gap_min` / `band_gap_max` — 按带隙（eV）过滤
- `is_stable` — 设为`true`仅返回热力学稳定相
- `_fields` — 返回字段列表，逗号分隔
- `_limit` — 最大结果数（默认10，上限1000）
- `_skip` — 分页偏移量

### 通过ID获取材料

```
GET /materials/summary/mp-149?_fields=material_id,formula_pretty,band_gap,formation_energy_per_atom,symmetry
```

材料ID格式为`mp-NNNNN`（如硅的ID为`mp-149`）。

### 可用字段（摘要）

`material_id`, `formula_pretty`, `formula_anonymous`, `chemsys`, `volume`, `density`, `density_atomic`, `symmetry`, `band_gap`, `cbm`, `vbm`, `is_gap_direct`, `is_metal`, `is_magnetic`, `ordering`, `total_magnetization`, `formation_energy_per_atom`, `energy_above_hull`, `is_stable`, `equilibrium_reaction_energy_per_atom`, `nsites`, `elements`, `nelements`, `composition`, `structure`

### 晶体结构

```
GET /materials/summary/mp-149?_fields=structure
```

返回结构体为pymatgen兼容的JSON字典，包含晶格参数和原子位置。

### 弹性性质

```
GET /materials/elasticity/?material_id=mp-149&_fields=material_id,bulk_modulus,shear_modulus,elastic_tensor
```

### 电子结构（能带结构/态密度）

```
GET /materials/electronic_structure/bandstructure/mp-149
GET /materials/electronic_structure/dos/mp-149
```

### 热力学性质

```
GET /materials/thermo/?formula=Fe2O3&_fields=material_id,formation_energy_per_atom,energy_above_hull
```

### 示例：寻找带隙大于2 eV的稳定氧化物

```
GET /materials/summary/?elements=O&band_gap_min=2&is_stable=true&_fields=material_id,formula_pretty,band_gap,formation_energy_per_atom&_limit=10
```

## 响应格式

```json
{
  "data": [
    {
      "material_id": "mp-149",
      "formula_pretty": "Si",
      "band_gap": 0.6105,
      "formation_energy_per_atom": 0.0
    }
  ],
  "meta": {
    "total_doc": 1
  }
}
```

## 速率限制

- 已认证：约50次请求/分钟（根据服务器负载变化）
- 推荐批量请求替代多次单独调用
- 使用`_fields`减小负载大小以提升性能
- Python客户端`mp-api`会自动处理分页和重试

## 错误格式

```json
{
  "detail": "Not authenticated"
}
```

HTTP 401 = 缺少或无效API密钥。HTTP 404 = 未找到材料。HTTP 429 = 超出速率限制。
