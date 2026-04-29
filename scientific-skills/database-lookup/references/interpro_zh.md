# InterPro API 参考文档

## 基础 URL
```
https://www.ebi.ac.uk/interpro/api
```

## 认证
无需认证。完全公开的 API。

## 速率限制
未公布硬性限制。EBI 通用建议：合理使用，大数据集请使用批量下载。

## 响应格式
默认 JSON。部分端点支持显式指定 `?format=json`。

## 核心端点

### 1. 条目查询（按编号）
```
GET https://www.ebi.ac.uk/interpro/api/entry/interpro/{accession}
```
示例：
```
GET https://www.ebi.ac.uk/interpro/api/entry/interpro/IPR000504
```
返回包含条目名称、类型（家族/结构域/位点等）、描述、GO 术语、文献引用的 JSON 数据。

### 2. 按成员数据库查询条目
```
GET https://www.ebi.ac.uk/interpro/api/entry/pfam/{pfam_accession}
GET https://www.ebi.ac.uk/interpro/api/entry/smart/{smart_accession}
GET https://www.ebi.ac.uk/interpro/api/entry/prosite/{prosite_accession}
```
示例：
```
GET https://www.ebi.ac.uk/interpro/api/entry/pfam/PF00076
```

### 3. 条目搜索/列表
```
GET https://www.ebi.ac.uk/interpro/api/entry/interpro?search={query}
```
示例：
```
GET https://www.ebi.ac.uk/interpro/api/entry/interpro?search=kinase
```
返回匹配 InterPro 条目的分页列表。

### 4. 蛋白质注释 — 获取蛋白质的 InterPro 条目
```
GET https://www.ebi.ac.uk/interpro/api/entry/interpro/protein/uniprot/{uniprot_accession}
```
示例：
```
GET https://www.ebi.ac.uk/interpro/api/entry/interpro/protein/uniprot/P12345
```
返回注释该蛋白质的所有 InterPro 条目。

### 5. 含特定条目的蛋白质
```
GET https://www.ebi.ac.uk/interpro/api/protein/uniprot/entry/interpro/{accession}
```
示例：
```
GET https://www.ebi.ac.uk/interpro/api/protein/uniprot/entry/interpro/IPR000504
```
返回被该条目注释的 UniProt 蛋白质分页列表。

### 6. 结构映射
```
GET https://www.ebi.ac.uk/interpro/api/structure/pdb/entry/interpro/{accession}
```
示例：
```
GET https://www.ebi.ac.uk/interpro/api/structure/pdb/entry/interpro/IPR000504
```

### 7. 按类型筛选条目
```
GET https://www.ebi.ac.uk/interpro/api/entry/interpro?type=domain
GET https://www.ebi.ac.uk/interpro/api/entry/interpro?type=family
GET https://www.ebi.ac.uk/interpro/api/entry/interpro?type=homologous_superfamily
```

### 8. 分类学交叉引用
```
GET https://www.ebi.ac.uk/interpro/api/taxonomy/uniprot/entry/interpro/{accession}
```

## 分页机制
响应包含 `next` 和 `previous` URL：
```json
{
  "count": 1234,
  "next": "https://www.ebi.ac.uk/interpro/api/entry/interpro?cursor=...&page_size=20",
  "previous": null,
  "results": [...]
}
```
使用 `?page_size=N` 控制分页大小（默认 20）。

## 条目响应关键字段
```json
{
  "metadata": {
    "accession": "IPR000504",
    "name": "RNA recognition motif domain",
    "type": "domain",
    "source_database": "interpro",
    "member_databases": {"pfam": {"PF00076": "RRM_1"}},
    "go_terms": [{"identifier": "GO:0003723", "name": "RNA binding"}],
    "description": ["<p>The RNA recognition motif...</p>"]
  }
}
```

## 注意事项
- API 采用可组合 URL 模式：组合实体类型（entry, protein, structure, taxonomy）可创建交叉查询
- 成员数据库：pfam, smart, prosite, prints, panther, cdd, hamap, tigrfam, pirsf, sfld, ncbifam
