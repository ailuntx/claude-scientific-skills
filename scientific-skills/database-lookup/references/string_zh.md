# STRING REST API

## 基础 URL

```
https://string-db.org/api
```

## URL 模式

```
/api/{输出格式}/{方法}
```

- **输出格式**: `json`, `tsv`, `tsv-no-header`, `image`, `svg`（非所有端点都支持全部格式）
- **方法**: 端点名称（见下文）

## 认证

无需 API 密钥。所有端点均为公开接口。

## 核心端点

### 1. 解析蛋白质标识符

将蛋白质名称/标识符映射为 STRING 内部 ID。建议首先执行此操作以获取标准 STRING ID。

```
GET /api/json/resolve?identifier={查询词}&species={物种分类ID}
```

| 参数         | 类型   | 说明 |
|-------------|--------|------|
| `identifier` | 字符串 | **必需**。蛋白质名称、基因符号或外部 ID。 |
| `species`    | 整数   | NCBI 物种分类 ID（9606=人类, 10090=小鼠）。推荐使用以避免歧义。 |

**示例：**
```
https://string-db.org/api/json/resolve?identifier=TP53&species=9606
```

**响应：**
```json
[
  {
    "stringId": "9606.ENSP00000269305",
    "preferredName": "TP53",
    "ncbiTaxonId": 9606,
    "taxonName": "Homo sapiens",
    "annotation": "细胞肿瘤抗原 p53; ..."
  }
]
```

---

### 2. 获取互作伙伴（网络）

```
GET /api/json/interaction_partners?identifiers={蛋白质列表}&species={物种分类ID}
```

| 参数               | 类型   | 说明 |
|--------------------|--------|------|
| `identifiers`       | 字符串 | **必需**。蛋白质名称列表，使用 `%0d`（换行符编码）分隔多个名称。 |
| `species`           | 整数   | NCBI 物种分类 ID。 |
| `limit`             | 整数   | 返回的互作伙伴最大数量（每个输入蛋白质）。 |
| `required_score`    | 整数   | 最低综合评分（0-1000）。默认值：400。常用阈值：400（中）, 700（高）, 900（最高）。 |
| `network_type`      | 字符串 | `functional`（默认，所有关联）或 `physical`（仅物理结合）。 |

**示例：**
```
https://string-db.org/api/json/interaction_partners?identifiers=TP53&species=9606&limit=10&required_score=900
```

**响应：**
```json
[
  {
    "stringId_A": "9606.ENSP00000269305",
    "stringId_B": "9606.ENSP00000261842",
    "preferredName_A": "TP53",
    "preferredName_B": "MDM2",
    "ncbiTaxonId": 9606,
    "score": 0.999,
    "nscore": 0,
    "fscore": 0,
    "pscore": 0,
    "ascore": 0.93,
    "escore": 0.994,
    "dscore": 0.9,
    "tscore": 0.981
  }
]
```

评分通道：`nscore`（邻近关系）, `fscore`（基因融合）, `pscore`（系统发育共现）, `ascore`（共表达）, `escore`（实验证据）, `dscore`（数据库/人工标注）, `tscore`（文本挖掘）。

---

### 3. 获取蛋白质集合间的网络互作

```
GET /api/json/network?identifiers={蛋白质列表}&species={物种分类ID}
```

| 参数             | 类型   | 说明 |
|------------------|--------|------|
| `identifiers`     | 字符串 | **必需**。蛋白质名称列表，使用 `%0d`（换行符编码）分隔。 |
| `species`         | 整数   | NCBI 物种分类 ID。 |
| `required_score`  | 整数   | 最低综合评分（0-1000）。 |
| `network_type`    | 字符串 | `functional` 或 `physical`。 |
| `add_nodes`       | 整数   | 需添加的额外互作蛋白数量（扩展网络）。 |

**示例 — 蛋白质集合间的网络：**
```
https://string-db.org/api/json/network?identifiers=TP53%0dBRCA1%0dATM%0dCHEK2%0dMDM2&species=9606&required_score=700
```

返回输入集合中所有成对互作关系。

---

### 4. 网络图像

```
GET /api/image/network?identifiers={蛋白质列表}&species={物种分类ID}
GET /api/svg/network?identifiers={蛋白质列表}&species={物种分类ID}
```

返回互作网络的 PNG 图像或 SVG 矢量图。

**示例：**
```
https://string-db.org/api/image/network?identifiers=TP53%0dBRCA1%0dMDM2&species=9606
```

---

### 5. 功能富集分析

对蛋白质集合进行基因本体（GO）、KEGG 通路等富集分析。

```
GET /api/json/enrichment?identifiers={蛋白质列表}&species={物种分类ID}
```

| 参数         | 类型   | 说明 |
|--------------|--------|------|
| `identifiers` | 字符串 | **必需**。换行符分隔（`%0d`）的蛋白质名称。 |
| `species`     | 整数   | NCBI 物种分类 ID。 |

**示例：**
```
https://string-db.org/api/json/enrichment?identifiers=TP53%0dBRCA1%0dATM%0dCHEK2%0dCDK2%0dCDKN1A&species=9606
```

**响应：**
```json
[
  {
    "category": "Process",
    "term": "GO:0006974",
    "description": "细胞对DNA损伤刺激的响应",
    "number_of_genes": 6,
    "number_of_genes_in_background": 781,
    "ncbiTaxonId": 9606,
    "inputGenes": "TP53,BRCA1,ATM,CHEK2,CDK2,CDKN1A",
    "preferredNames": "TP53,BRCA1,ATM,CHEK2,CDK2,CDKN1A",
    "p_value": 1.2e-12,
    "fdr": 5.6e-10
  }
]
```

分析类别包括：`Process`（GO生物过程）, `Function`（GO分子功能）, `Component`（GO细胞组分）, `KEGG`, `Pfam`, `InterPro`, `SMART`, `Keyword`（UniProt）, `Reactome`, `WikiPathways`, `HPO`（人类表型本体）。

---

### 6. 获取蛋白质注释/信息

```
GET /api/json/get_string_ids?identifiers={蛋白质列表}&species={物种分类ID}
```

将任意名称映射为带注释文本的 STRING ID。

**示例：**
```
https://string-db.org/api/json/get_string_ids?identifiers=CDK2%0dp53&species=9606
```

**响应：**
```json
[
  {
    "queryIndex": 0,
    "queryItem": "CDK2",
    "stringId": "9606.ENSP00000266970",
    "ncbiTaxonId": 9606,
    "taxonName": "Homo sapiens",
    "preferredName": "CDK2",
    "annotation": "细胞周期蛋白依赖性激酶2; ..."
  }
]
```

---

### 7. 获取同源蛋白 / 跨物种最佳匹配

```
GET /api/json/homology?identifiers={蛋白质列表}&species={源物种分类ID}&species_b={目标物种分类ID}
```

| 参数         | 类型   | 说明 |
|------------|--------|------|
| `identifiers` | 字符串 | 源蛋白质列表。 |
| `species`     | 整数   | 源物种分类 ID。 |
| `species_b`   | 整数   | 同源蛋白查找的目标物种分类 ID。 |

**示例：**
```
https://string-db.org/api/json/homology?identifiers=TP53&species=9606&species_b=10090
```

---

### 8. PPI 富集分析（检测蛋白质集合互作是否显著）

```
GET /api/json/ppi_enrichment?identifiers={蛋白质列表}&species={物种分类ID}
```

**示例：**
```
https://string-db.org/api/json/ppi_enrichment?identifiers=TP53%0dBRCA1%0dATM%0dCHEK2&species=9606
```

**响应：**
```json
[
  {
    "number_of_nodes": 4,
    "number_of_edges": 6,
    "average_node_degree": 3.0,
    "local_clustering_coefficient": 1.0,
    "expected_number_of_edges": 1,
    "p_value": 0.000123
  }
]
```

---

## 常用物种分类 ID

| 物种 | 分类 ID |
|---------|----------|
| 智人（Homo sapiens） | 9606 |
| 小家鼠（Mus musculus） | 10090 |
| 褐家鼠（Rattus norvegicus） | 10116 |
| 黑腹果蝇（Drosophila melanogaster） | 7227 |
| 酿酒酵母（Saccharomyces cerevisiae） | 4932 |
| 秀丽隐杆线虫（Caenorhabditis elegans） | 6239 |
| 斑马鱼（Danio rerio） | 7955 |
| 大肠杆菌 K12（Escherichia coli K12） | 511145 |
| 拟南芥（Arabidopsis thaliana） | 3702 |

## 速率限制

- 无公开硬性限制，但 API 设计用于中等频率的程序化访问。
- 建议：**每秒最多 1 次请求**。
- 大规模数据下载请使用 STRING 网站的平面文件下载功能。
- 过量请求可能导致 HTTP 429 错误或临时封禁。
- 强烈建议使用单次多标识符请求，而非多次单标识符请求。

## 错误处理

- 错误请求返回 HTTP 400。
- 无匹配蛋白质时返回 HTTP 404。
- 有效查询但无结果时返回空 JSON 数组 `[]`（例如无超过阈值的互作）。
- 尽可能包含 `species` 参数以避免标识符解析歧义。
