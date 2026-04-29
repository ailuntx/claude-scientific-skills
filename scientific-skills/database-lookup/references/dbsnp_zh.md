# dbSNP API 参考文档

## 概述
SNP 和变异数据。可通过两个 API 访问：NCBI E-utilities (`db=snp`) 用于搜索/元数据，以及 NCBI Variation Services REST API 用于获取详细的变异注释。

## 基础 URL
```
E-utilities:  https://eutils.ncbi.nlm.nih.gov/entrez/eutils/
Variation API: https://api.ncbi.nlm.nih.gov/variation/v0/
```

## 认证
- **E-utilities**：推荐使用 API 密钥（`&api_key=KEY`）。无密钥时每秒 3 次请求，使用密钥时每秒 10 次请求。
- **Variation API**：无需认证。存在速率限制（未公开；请合理使用，建议每秒约 1-2 次请求）。

---

## E-utilities 端点 (db=snp)

### 1. ESearch —— 搜索 SNP
```
GET esearch.fcgi?db=snp&term=QUERY&retmax=N&retmode=json
```

**示例 —— 在 BRCA1 基因中搜索 SNP：**
```
GET esearch.fcgi?db=snp&term=BRCA1[Gene Name] AND homo sapiens[Organism]&retmax=5&retmode=json
```
响应：
```json
{
  "esearchresult": {
    "count": "12847",
    "idlist": ["80357713", "80357508", ...]
  }
}
```
注意：返回的 ID 是 rs 编号，但不包含 "rs" 前缀。

### 2. ESummary —— SNP 摘要
```
GET esummary.fcgi?db=snp&id=IDS&retmode=json
```

**示例 —— 获取 rs334（镰状细胞变异）摘要：**
```
GET esummary.fcgi?db=snp&id=334&retmode=json
```
响应包含：`snp_id`、`chr`、`chrpos`、`genes`、`clinical_significance`、`global_mafs`、`docsum`。

### 3. EFetch —— 获取 SNP 详情（仅 XML）
```
GET efetch.fcgi?db=snp&id=IDS&rettype=json&retmode=text
```
注意：dbSNP 的 EFetch 需使用 `rettype=json` 返回 JSON。也支持通过 `retmode=xml` 获取 XML。

---

## Variation Services API

### 1. 通过 rsID 查询变异
```
GET /variation/v0/refsnp/{rsid}
```

**示例：**
```
GET https://api.ncbi.nlm.nih.gov/variation/v0/refsnp/334
```
响应（JSON，节选）：
```json
{
  "refsnp_id": "334",
  "create_date": "2000/09/19",
  "primary_snapshot_data": {
    "placements_with_allele": [...],
    "allele_annotations": [...],
    "support": [...]
  },
  "present_obs_movements": [
    {
      "component_ids": [{"type": "clinvar", "value": "..."}],
      "observation": {
        "seq_id": "NC_000011.10",
        "position": 5227002,
        "deleted_sequence": "T",
        "inserted_sequence": "A"
      }
    }
  ]
}
```

### 2. 通过 SPDI 表示法查询变异
```
GET /variation/v0/spdi/{spdi}/rsids
```
SPDI 格式：`SeqID:Position:Deletion:Insertion`

**示例：**
```
GET https://api.ncbi.nlm.nih.gov/variation/v0/spdi/NC_000011.10:5227002:T:A/rsids
```

### 3. 通过 HGVS 查询变异
```
GET /variation/v0/hgvs/{hgvs}/contextuals
```

**示例：**
```
GET https://api.ncbi.nlm.nih.gov/variation/v0/hgvs/NC_000011.10:g.5227003T>A/contextuals
```

### 4. 批量 rsID 查询 (POST)
```
POST /variation/v0/refsnp/batch
Content-Type: application/json

{"refsnp_ids": ["334", "1805007", "7412"]}
```

## 常用 E-utilities 搜索模式
```
# 按 rs 编号
term=334[RS ID]

# 临床意义
term=pathogenic[Clinical Significance] AND BRCA1[Gene Name]

# 按染色体位置 (GRCh38)
term=11[Chromosome] AND 5227002:5227002[Base Position]

# 按变异类型
term=missense[Function Class] AND TP53[Gene Name]

# 按全局次要等位基因频率
term=0.01:0.05[Global MAF]
```

## 速率限制
- E-utilities：每秒 3 次请求（无密钥），每秒 10 次请求（使用密钥）
- Variation Services API：无公开限制；建议每秒 1-2 次请求
