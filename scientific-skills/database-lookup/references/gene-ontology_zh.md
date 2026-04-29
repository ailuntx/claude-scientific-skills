# 基因本体论（GO）API 参考

## 基础 URL
- **QuickGO (EBI, 推荐)**: `https://www.ebi.ac.uk/QuickGO/services` — 最可靠的端点
- **GO API**: `https://api.geneontology.org/api` — 可能返回403；建议使用QuickGO作为备选
- **AmiGO / GOlr (基于Solr)**: `http://golr-aux.geneontology.org/solr`

## 认证
无需认证。所有端点均为公开接口。

## 速率限制
无官方硬性限制。QuickGO建议合理使用。

---

## GO API (api.geneontology.org)

### 1. GO 术语查询
```
GET https://api.geneontology.org/api/ontology/term/{go_id}
```
示例：
```
GET https://api.geneontology.org/api/ontology/term/GO%3A0008150
```
返回包含术语名称、定义、命名空间（生物过程/分子功能/细胞组分）及同义词的JSON。

### 2. 基因/蛋白质注释（生物实体）
```
GET https://api.geneontology.org/api/bioentity/gene/{gene_id}/function
```
示例 — UniProt蛋白质的GO注释：
```
GET https://api.geneontology.org/api/bioentity/gene/UniProtKB%3AP04637/function
```
返回包含证据代码、限定符和参考文献的GO注释。

### 3. 关联GO术语的基因
```
GET https://api.geneontology.org/api/bioentity/function/{go_id}/genes
```
示例：
```
GET https://api.geneontology.org/api/bioentity/function/GO%3A0006915/genes?rows=20
```
返回关联该GO术语的基因/蛋白质。

### 4. 搜索实体
```
GET https://api.geneontology.org/api/search/entity/{query}
```
示例：
```
GET https://api.geneontology.org/api/search/entity/apoptosis?rows=10
```

### 5. 本体祖先/后代
```
GET https://api.geneontology.org/api/ontology/term/{go_id}/graph
```

---

## QuickGO API (EBI — 推荐用于稳定注释查询)

### 1. GO术语详情
```
GET https://www.ebi.ac.uk/QuickGO/services/ontology/go/terms/{go_ids}
```
示例：
```
GET https://www.ebi.ac.uk/QuickGO/services/ontology/go/terms/GO:0008150
```
支持逗号分隔的ID（最多25个）。

### 2. 搜索注释
```
GET https://www.ebi.ac.uk/QuickGO/services/annotation/search?geneProductId={uniprot_id}
```
示例 — TP53的注释：
```
GET https://www.ebi.ac.uk/QuickGO/services/annotation/search?geneProductId=P04637&limit=25
```

### 3. 按GO术语查询注释
```
GET https://www.ebi.ac.uk/QuickGO/services/annotation/search?goId=GO:0006915&taxonId=9606&limit=25
```

### 4. 按证据筛选注释
```
GET https://www.ebi.ac.uk/QuickGO/services/annotation/search?geneProductId=P04637&goUsage=descendants&evidenceCode=ECO:0000269&limit=25
```

### 5. GO术语子节点
```
GET https://www.ebi.ac.uk/QuickGO/services/ontology/go/terms/GO:0008150/children
```

### 6. GO术语祖先图谱
```
GET https://www.ebi.ac.uk/QuickGO/services/ontology/go/terms/GO:0006915/ancestors?relations=is_a,part_of
```

### 7. 按名称搜索GO术语
```
GET https://www.ebi.ac.uk/QuickGO/services/ontology/go/search?query=apoptosis&limit=10
```

## QuickGO注释搜索参数
| 参数 | 描述 |
|-----------|-------------|
| `geneProductId` | UniProt编号 (如P04637) |
| `goId` | GO术语 (如GO:0006915) |
| `goUsage` | `exact` 或 `descendants` (包含子术语) |
| `taxonId` | NCBI分类ID (9606=人类) |
| `evidenceCode` | ECO代码 (如ECO:0000269=实验证据) |
| `aspect` | `biological_process`, `molecular_function`, `cellular_component` |
| `limit` | 每页结果数 (上限100) |
| `page` | 页码 (从1开始) |

## QuickGO响应格式
```json
{
  "numberOfHits": 1234,
  "results": [
    {
      "geneProductId": "P04637",
      "symbol": "TP53",
      "goId": "GO:0006915",
      "goName": "凋亡过程",
      "evidenceCode": "ECO:0000269",
      "goAspect": "biological_process",
      "taxonId": 9606,
      "reference": "PMID:12345678",
      "assignedBy": "UniProt"
    }
  ]
}
```

## 注意事项
- QuickGO (EBI) 在注释查询方面通常更稳定且文档更完善
- GO API (geneontology.org) 更适合本体结构遍历
- GO ID在路径中使用时需URL编码 (如`GO%3A0008150`对应`GO:0008150`)
- 三个GO命名空间：生物过程(BP)、分子功能(MF)、细胞组分(CC)
- 证据代码示例：IDA（直接实验）、IMP（突变表型）、IGI（遗传互作）、IEA（电子注释）等
