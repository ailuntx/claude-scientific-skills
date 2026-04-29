# BioGRID API 参考

## 基础 URL
```
https://webservice.thebiogrid.org/interactions
```

## 认证
**需要 API 密钥。** 在 https://webservice.thebiogrid.org/ 免费注册获取访问密钥。
- 通过查询参数传递：`?accesskey=您的访问密钥`

## 速率限制
未正式公布。预期合理使用。

## 响应格式
JSON（使用 `&format=json`）、制表符分隔格式（`&format=tab2`）或 XML。默认为 tab2。

## 关键端点

### 1. 按基因搜索相互作用
```
GET https://webservice.thebiogrid.org/interactions?accesskey={key}&format=json&searchNames=true&geneList={gene_symbol}&taxId={taxon_id}
```
示例 — 获取人类 TP53 的相互作用：
```
GET https://webservice.thebiogrid.org/interactions?accesskey=YOUR_KEY&format=json&searchNames=true&geneList=TP53&taxId=9606&max=50
```

### 2. 多基因搜索
```
GET https://webservice.thebiogrid.org/interactions?accesskey={key}&format=json&geneList=BRCA1|BRCA2&taxId=9606&max=100
```
使用 `|`（竖线）分隔基因名称。

### 3. 按证据类型过滤
```
GET https://webservice.thebiogrid.org/interactions?accesskey={key}&format=json&geneList=TP53&taxId=9606&evidenceList=physical&max=50
```
证据类型：`physical`（物理）、`genetic`（遗传）。

### 4. 按实验系统过滤
```
GET https://webservice.thebiogrid.org/interactions?accesskey={key}&format=json&geneList=TP53&taxId=9606&experimentalSystemList=Two-hybrid&max=50
```
系统包括：`Two-hybrid`（双杂交）、`Affinity Capture-MS`（亲和捕获质谱）、`Co-fractionation`（共分离）、`Reconstituted Complex`（重构复合物）、`Synthetic Lethality`（合成致死）、`Dosage Rescue`（剂量拯救）等。

### 5. 按 BioGRID 相互作用 ID 搜索
```
GET https://webservice.thebiogrid.org/interactions/{interaction_id}?accesskey={key}&format=json
```

### 6. 按 PubMed ID 搜索
```
GET https://webservice.thebiogrid.org/interactions?accesskey={key}&format=json&pubmedList=12345678
```

### 7. 跨物种相互作用
```
GET https://webservice.thebiogrid.org/interactions?accesskey={key}&format=json&geneList=TP53&taxId=9606&interSpeciesExcluded=false
```

### 8. 包含相互作用因子注释
```
GET https://webservice.thebiogrid.org/interactions?accesskey={key}&format=json&geneList=TP53&taxId=9606&includeInteractors=true&max=50
```

## 常用查询参数
| 参数 | 描述 |
|-----------|-------------|
| `geneList` | 基因符号，使用竖线分隔 |
| `taxId` | NCBI 分类学 ID（9606=人类，10090=小鼠，559292=酵母） |
| `max` | 返回结果上限（默认 10000） |
| `start` | 分页起始偏移量 |
| `format` | `json`、`tab2`、`extendedTab2`、`count` |
| `searchNames` | `true` 表示匹配官方符号 |
| `selfInteractionsExcluded` | `true` 表示排除自身相互作用 |
| `evidenceList` | `physical` 或 `genetic` |
| `throughputTag` | `low` 或 `high` |

## JSON 响应结构
```json
{
  "12345": {
    "BIOGRID_INTERACTION_ID": 12345,
    "ENTREZ_GENE_A": "7157",
    "ENTREZ_GENE_B": "672",
    "OFFICIAL_SYMBOL_A": "TP53",
    "OFFICIAL_SYMBOL_B": "BRCA1",
    "EXPERIMENTAL_SYSTEM": "Two-hybrid",
    "EXPERIMENTAL_SYSTEM_TYPE": "physical",
    "PUBMED_ID": "9482880",
    "ORGANISM_A": 9606,
    "ORGANISM_B": 9606,
    "THROUGHPUT": "Low Throughput",
    "SCORE": "-"
  }
}
```

## 仅计数查询
```
GET https://webservice.thebiogrid.org/interactions?accesskey={key}&format=count&geneList=TP53&taxId=9606
```
仅返回整数计数。

## 说明
- BioGRID 汇总了文献中经过整理的相互作用数据
- 涵盖物理（蛋白质-蛋白质）和遗传相互作用
- 批量数据请使用 BioGRID 下载（制表符分隔文件）：https://downloads.thebiogrid.org/
- 可与 STRING 交叉参考以获取组合的相互作用证据
