# Reactome 内容服务 REST API

## 基础 URL

```
https://reactome.org/ContentService
```

无需认证。默认返回 JSON 格式。

## 关键端点

### 搜索（跨通路、反应、蛋白质的全文检索）

```
GET /search/query?query={term}
```

参数：
- `query`（必需）— 搜索词（如 "apoptosis", "TP53", "R-HSA-109581"）
- `species` — 按物种过滤（如 "Homo sapiens"）
- `types` — 按类型过滤：`Pathway`, `Reaction`, `Protein`, `Complex`, `SmallMolecule`
- `cluster` — 布尔值，是否聚类结果（默认 true）
- `rows` — 每页结果数
- `Start row` — 分页偏移量

示例：
```
/search/query?query=apoptosis&species=Homo+sapiens&types=Pathway
```

响应：
```json
{
  "results": [
    {
      "typeName": "Pathway",
      "rows": [
        {
          "dbId": 109581,
          "stId": "R-HSA-109581",
          "name": "Apoptosis",
          "species": ["Homo sapiens"],
          "summation": ["..."]
        }
      ]
    }
  ],
  "found": 42
}
```

### 自动补全
```
GET /search/suggest?query={partial_term}
```

### 物种顶级通路
```
GET /data/pathways/top/{species}
```
示例：`/data/pathways/top/Homo+sapiens`

### 通路详情
```
GET /data/query/{id}
```
`{id}` 为稳定 ID（如 `R-HSA-109581`）或数字 dbId。

### 通路包含的事件
```
GET /data/pathway/{id}/containedEvents
```

### 反应的参与者
```
GET /data/event/{id}/participants
```

### 事件的祖先路径
```
GET /data/event/{id}/ancestors
```

### 外部 ID 到通路的映射（如 UniProt 到 Reactome 通路）
```
GET /data/mapping/{resource}/{id}/pathways
```
示例 — 查询 TP53（UniProt P04637）相关通路：
```
/data/mapping/UniProt/P04637/pathways
```

### 外部 ID 到反应的映射
```
GET /data/mapping/{resource}/{id}/reactions
```

### 通用实体查询
```
GET /data/query/{id}
```

### 事件的参考实体
```
GET /data/participants/{id}/referenceEntities
```

### 所有物种
```
GET /data/species/all
```

### 物种事件层级（大型响应）
```
GET /data/eventsHierarchy/{species}
```

## 稳定 ID 格式

`R-{物种代码}-{数字}`

| 代码 | 物种 |
|---|---|
| HSA | 智人（Homo sapiens） |
| MMU | 小家鼠（Mus musculus） |
| RNO | 褐家鼠（Rattus norvegicus） |
| DME | 黑腹果蝇（Drosophila melanogaster） |
| CEL | 秀丽隐杆线虫（C. elegans） |
| SCE | 酿酒酵母（S. cerevisiae） |

## 映射支持的外部资源

`UniProt`, `ChEBI`, `ENSEMBL`, `miRBase`, `GeneCards`, `NCBI`

多参数值处理：重复参数（如 `types=Pathway&types=Reaction`）。

## 速率限制

无需 API 密钥。无官方速率限制，但需合理使用——避免数百并发请求。批量数据请使用 Reactome 可下载转储（MySQL, Neo4j, BioPAX, SBML）。
