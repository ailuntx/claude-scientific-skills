# ChEMBL REST API

## 基础 URL
```
https://www.ebi.ac.uk/chembl/api/data
```

## 认证
无需 API 密钥。完全开放且免费。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/molecule/{chembl_id}` | 通过 ChEMBL ID 获取分子信息 |
| `/molecule/search?q={query}` | 分子自由文本搜索 |
| `/target/{chembl_id}` | 通过 ChEMBL ID 获取靶点信息 |
| `/target/search?q={query}` | 靶点自由文本搜索 |
| `/activity?molecule_chembl_id={id}` | 分子相关活性数据 |
| `/activity?target_chembl_id={id}` | 靶点相关活性数据 |
| `/mechanism?molecule_chembl_id={id}` | 作用机制 |
| `/drug_indication?molecule_chembl_id={id}` | 药物适应症 |
| `/similarity/{smiles}/{threshold}` | 相似性搜索（阈值 40-100） |
| `/substructure/{smiles}` | 子结构搜索 |

## 通用参数

- `format=json` — 响应格式（默认 json）
- `limit` — 每页结果数（默认 20，最大 1000）
- `offset` — 分页偏移量
- `order_by` — 排序字段（前缀 `-` 表示降序）
- `only` — 仅返回指定字段（逗号分隔）

### 过滤运算符（附加在字段名后）
`__exact`, `__icontains`, `__gt`, `__gte`, `__lt`, `__lte`, `__in`, `__isnull`, `__startswith`, `__range`, `__regex`

## 调用示例

```
# 通过 ID 获取分子
/molecule/CHEMBL25.json

# 按名称搜索分子
/molecule/search?q=aspirin&format=json

# 靶点活性数据（含效价过滤）
/activity?target_chembl_id=CHEMBL240&pchembl_value__gte=6&format=json&limit=100

# 相似性搜索（80% 阈值）
/similarity/CC(%3DO)Oc1ccccc1C(%3DO)O/80.json

# 仅获取获批药物
/molecule?max_phase=4&format=json

# 作用机制查询
/mechanism?molecule_chembl_id=CHEMBL25&format=json
```

## 响应格式（分子）
```json
{
  "page_meta": {"limit": 20, "offset": 0, "total_count": 150},
  "molecules": [{
    "molecule_chembl_id": "CHEMBL25",
    "pref_name": "ASPIRIN",
    "max_phase": 4,
    "molecule_properties": {
      "full_mwt": 180.16, "full_molformula": "C9H8O4",
      "alogp": 1.31, "hba": 3, "hbd": 1, "psa": 63.60
    },
    "molecule_structures": {
      "canonical_smiles": "CC(=O)Oc1ccccc1C(=O)O",
      "standard_inchi_key": "BSYNRYMUTXBXSQ-UHFFFAOYSA-N"
    }
  }]
}
```

## 速率限制
无严格限制。建议每秒请求不超过 10 次。无需认证。
