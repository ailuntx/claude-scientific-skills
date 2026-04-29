# 人类蛋白质图谱（HPA）

## 基础 URL
```
https://www.proteinatlas.org
```

## 认证
无需 API 密钥。

## 关键端点

| 用途 | URL 模式 |
|---|---|
| 通过 Ensembl ID 获取基因数据 | `/{ENSEMBL_ID}.json` |
| 通过基因符号获取基因数据 | `/{GENE_NAME}.json` |
| 搜索（JSON格式） | `/search/{QUERY}?format=json` |
| 搜索（XML格式） | `/search/{QUERY}?format=xml` |

## 调用示例

```
# 通过 Ensembl ID 获取基因数据
https://www.proteinatlas.org/ENSG00000141510.json

# 通过基因符号获取基因数据
https://www.proteinatlas.org/TP53.json

# 搜索
https://www.proteinatlas.org/search/TP53?format=json
```

## 响应格式（JSON，基因端点）
```json
{
  "Gene": "TP53",
  "Gene synonym": ["p53", "LFS1"],
  "Ensembl": "ENSG00000141510",
  "Gene description": "肿瘤蛋白 p53",
  "Uniprot": ["P04637"],
  "Chromosome": "17",
  "Protein class": ["转录因子"],
  "RNA tissue specificity": "组织特异性低",
  "Subcellular location": ["核质"],
  "Pathology prognostics": [...]
}
```

## 批量下载
如需大规模处理，请使用 https://www.proteinatlas.org/about/download 提供的 TSV 文件：
- `normal_tissue.tsv` — 免疫组化组织表达数据
- `rna_tissue_consensus.tsv` — RNA 一致性数据
- `subcellular_location.tsv` — 亚细胞定位数据
- `pathology.tsv` — 癌症预后数据

## 速率限制
未公布具体限制。请合理使用。大规模查询建议优先采用批量下载方式。
