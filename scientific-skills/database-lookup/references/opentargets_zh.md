# Open Targets 平台 API

## 基础 URL

**GraphQL API（主要推荐）：**
```
https://api.platform.opentargets.org/api/v4/graphql
```

**重要提示：** GraphQL 端点需要通过 HTTP POST 请求并设置 `Content-Type: application/json`。仅支持 GET 的 WebFetch 无法使用——请改用 shell 中的 `curl`：
```bash
curl -s -X POST -H "Content-Type: application/json" \
  -d '{"query":"{ target(ensemblId: \"ENSG00000157764\") { approvedSymbol approvedName } }"}' \
  https://api.platform.opentargets.org/api/v4/graphql
```

**REST API（简单查询）：**
```
https://api.platform.opentargets.org/api/v4
```

## 认证

无需 API 密钥。所有端点均为公开访问。

## GraphQL API

所有 GraphQL 查询均通过 POST 请求发送至 GraphQL 端点。

```
POST https://api.platform.opentargets.org/api/v4/graphql
Content-Type: application/json

{
  "query": "...",
  "variables": { ... }
}
```

### 1. 靶点信息（通过 Ensembl 基因 ID）

```graphql
query TargetInfo($ensemblId: String!) {
  target(ensemblId: $ensemblId) {
    id
    approvedSymbol
    approvedName
    biotype
    proteinIds {
      id
      source
    }
    tractability {
      label
      modality
      value
    }
    safetyLiabilities {
      event
      effects {
        direction
        dosing
      }
    }
    pathways {
      pathway
      pathwayId
    }
    functionDescriptions
    subcellularLocations {
      location
    }
  }
}
```

**变量：** `{ "ensemblId": "ENSG00000141510" }`

**URL 示例（简单查询也支持 GET）：**
```
https://api.platform.opentargets.org/api/v4/graphql?query={target(ensemblId:"ENSG00000141510"){id approvedSymbol approvedName biotype functionDescriptions}}
```

---

### 2. 疾病信息（通过 EFO ID）

```graphql
query DiseaseInfo($efoId: String!) {
  disease(efoId: $efoId) {
    id
    name
    description
    therapeuticAreas {
      id
      name
    }
    synonyms {
      terms
    }
  }
}
```

**变量：** `{ "efoId": "EFO_0000311" }`（癌症）

**URL 示例：**
```
https://api.platform.opentargets.org/api/v4/graphql?query={disease(efoId:"EFO_0000311"){id name description therapeuticAreas{id name}}}
```

---

### 3. 靶点-疾病关联关系

```graphql
query Associations($ensemblId: String!, $page: Pagination!) {
  target(ensemblId: $ensemblId) {
    approvedSymbol
    associatedDiseases(page: $page) {
      count
      rows {
        disease {
          id
          name
        }
        score
        datasourceScores {
          id
          score
        }
      }
    }
  }
}
```

**变量：**
```json
{
  "ensemblId": "ENSG00000141510",
  "page": { "index": 0, "size": 10 }
}
```

**URL 示例：**
```
https://api.platform.opentargets.org/api/v4/graphql?query={target(ensemblId:"ENSG00000141510"){approvedSymbol associatedDiseases(page:{index:0,size:5}){count rows{disease{id name}score}}}}
```

---

### 4. 疾病-靶点关联关系（从疾病角度）

```graphql
query DiseaseAssociations($efoId: String!, $page: Pagination!) {
  disease(efoId: $efoId) {
    name
    associatedTargets(page: $page) {
      count
      rows {
        target {
          id
          approvedSymbol
        }
        score
        datasourceScores {
          id
          score
        }
      }
    }
  }
}
```

**变量：**
```json
{
  "efoId": "EFO_0000311",
  "page": { "index": 0, "size": 10 }
}
```

---

### 5. 靶点-疾病对的证据

```graphql
query Evidence($ensemblId: String!, $efoId: String!, $size: Int!) {
  disease(efoId: $efoId) {
    evidences(ensemblIds: [$ensemblId], size: $size) {
      count
      rows {
        id
        score
        datasourceId
        datatypeId
        literature
        diseaseFromSource
        targetFromSourceId
        resourceScore
        urls {
          niceName
          url
        }
      }
    }
  }
}
```

**变量：**
```json
{
  "ensemblId": "ENSG00000141510",
  "efoId": "EFO_0000311",
  "size": 10
}
```

---

### 6. 药物/分子信息

```graphql
query DrugInfo($chemblId: String!) {
  drug(chemblId: $chemblId) {
    id
    name
    drugType
    maximumClinicalTrialPhase
    hasBeenWithdrawn
    mechanismsOfAction {
      rows {
        mechanismOfAction
        targets {
          id
          approvedSymbol
        }
      }
    }
    indications {
      rows {
        disease {
          id
          name
        }
        maxPhaseForIndication
      }
    }
    linkedDiseases {
      count
      rows {
        id
        name
      }
    }
    linkedTargets {
      count
      rows {
        id
        approvedSymbol
      }
    }
  }
}
```

**变量：** `{ "chemblId": "CHEMBL25" }`（阿司匹林）

**URL 示例：**
```
https://api.platform.opentargets.org/api/v4/graphql?query={drug(chemblId:"CHEMBL25"){id name drugType maximumClinicalTrialPhase mechanismsOfAction{rows{mechanismOfAction targets{id approvedSymbol}}}}}
```

---

### 7. 跨靶点、疾病和药物搜索

```graphql
query Search($queryString: String!, $entityNames: [String!], $page: Pagination!) {
  search(queryString: $queryString, entityNames: $entityNames, page: $page) {
    total
    hits {
      id
      entity
      name
      description
      score
    }
  }
}
```

**变量：**
```json
{
  "queryString": "BRAF melanoma",
  "entityNames": ["target", "disease", "drug"],
  "page": { "index": 0, "size": 10 }
}
```

**URL 示例：**
```
https://api.platform.opentargets.org/api/v4/graphql?query={search(queryString:"BRAF",entityNames:["target"],page:{index:0,size:5}){total hits{id entity name description}}}
```

---

### 8. 靶点的已知药物

```graphql
query KnownDrugs($ensemblId: String!, $size: Int!) {
  target(ensemblId: $ensemblId) {
    approvedSymbol
    knownDrugs(size: $size) {
      count
      rows {
        drug {
          id
          name
          drugType
          maximumClinicalTrialPhase
        }
        disease {
          id
          name
        }
        phase
        status
        mechanismOfAction
        urls {
          niceName
          url
        }
      }
    }
  }
}
```

**变量：**
```json
{
  "ensemblId": "ENSG00000157764",
  "size": 10
}
```

(ENSG00000157764 = BRAF)

---

### 9. 成药性评估

包含在靶点查询中（见上文端点1）。干预模式包括：
- `SM`（小分子）
- `AB`（抗体）
- `PR`（PROTAC）
- `OC`（其他临床阶段方式）

---

## REST API 端点

这些是常见操作的简化替代方案。

### 搜索

```
GET /api/v4/search?q={query}&page=0&size=10
```

**示例：**
```
https://api.platform.opentargets.org/api/v4/search?q=TP53&size=5
```

**响应：**
```json
{
  "total": 15,
  "data": [
    {
      "id": "ENSG00000141510",
      "entity": "target",
      "name": "TP53",
      "description": "细胞肿瘤抗原 p53",
      "score": 142.5
    }
  ]
}
```

---

## 关键标识符

| 实体    | ID 格式          | 示例                     |
|---------|------------------|--------------------------|
| 靶点    | Ensembl 基因 ID  | `ENSG00000141510` (TP53) |
| 疾病    | EFO/Mondo/HP/Orphanet | `EFO_0000311` (癌症), `MONDO_0007254` |
| 药物    | ChEMBL ID        | `CHEMBL25` (阿司匹林)    |

## 数据源 ID（用于证据过滤）

- `ot_genetics_portal` -- Open Targets 遗传学平台
- `eva` -- ClinVar（通过 EVA）
- `cancer_gene_census` -- COSMIC 癌症基因普查
- `chembl` -- ChEMBL（临床试验）
- `europepmc` -- 文献挖掘
- `expression_atlas` -- 表达图谱
- `gene2phenotype` -- Gene2Phenotype
- `genomics_england` -- Genomics England PanelApp
- `intogen` -- IntOGen（癌症驱动基因）
- `ot_crispr` -- Open Targets CRISPR 筛选
- `progeny` -- PROGENy（通路活性）
- `reactome` -- Reactome 通路
- `slapenrich` -- SLAPenrich
- `sysbio` -- 系统生物学
- `uniprot_literature` -- UniProt 文献

## 分页机制

GraphQL 使用 `page: { index: Int, size: Int }`（索引从0开始）。
REST 使用 `page` 和 `size` 查询参数。

## 速率限制

- 无需 API 密钥。
- 适用公平使用速率限制，无公开硬性限制。
- 批量数据请使用 Open Targets 数据下载（GCS/FTP 上的 Parquet 文件），而非 API。
- 遵守 HTTP 429 状态码及 `Retry-After` 响应头。

## 错误格式

GraphQL 错误：
```json
{
  "errors": [
    {
      "message": "变量 '$ensemblId' 预期类型为 'String!' 但实际值为: null",
      "locations": [{"line": 1, "column": 7}]
    }
  ]
}
```

REST 错误返回对应的 HTTP 状态码及 JSON 错误体。

## 使用技巧

- 使用 GraphQL API 获得最大灵活性——仅请求所需字段。
- GraphQL 的 GET 方法适用于简单查询，但带变量的复杂查询需用 POST。
- 组合靶点+疾病查询可获取带证据细分的关联评分。
- 在关联查询中使用 `datasourceScores` 查看主要贡献证据源。
- Open Targets 平台 Web 界面 `https://platform.opentargets.org` 提供 GraphQL 交互式查询界面用于测试。
