# HPO（人类表型本体）

## 基础 URL
```
https://ontology.jax.org/api/hp
```

## 认证
无需 API 密钥。

## 重要提示：对 HP ID 中的冒号进行 URL 编码 — `HP:0001250` 需转换为 `HP%3A0001250`

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/hpo/search?q={query}&max={n}` | 按名称搜索 HPO 术语 |
| `/hpo/term/{id}` | 术语详情 |
| `/hpo/term/{id}/genes` | 与表型相关的基因 |
| `/hpo/term/{id}/diseases` | 与表型相关的疾病 |
| `/hpo/term/{id}/children` | 层级中的子术语 |
| `/hpo/term/{id}/parents` | 父术语 |
| `/hpo/gene/{gene_id}` | 基因的表型（Entrez ID） |
| `/hpo/disease/{disease_id}` | 疾病的表型（OMIM/ORPHA） |

## 示例调用
```
# 搜索 "seizure"
https://ontology.jax.org/api/hp/hpo/search?q=seizure&max=5

# Seizure 的术语详情
https://ontology.jax.org/api/hp/hpo/term/HP%3A0001250

# 与 Seizure 相关的基因
https://ontology.jax.org/api/hp/hpo/term/HP%3A0001250/genes

# Seizure 相关的疾病
https://ontology.jax.org/api/hp/hpo/term/HP%3A0001250/diseases

# SCN1A（Entrez 6323）的表型
https://ontology.jax.org/api/hp/hpo/gene/6323
```

## 响应格式
JSON。术语字段：`id`、`name`、`definition`、`synonyms`。基因关联：`genes[]`，包含 `geneId` 和 `geneSymbol`。疾病：`diseases[]`，包含 `diseaseId` 和 `diseaseName`。

## 速率限制
无公开限制。批量注释文件位于 https://hpo.jax.org/data/annotations
