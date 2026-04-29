# Monarch Initiative API

## 基础 URL
```
https://api.monarchinitiative.org/v3/api
```

## 认证
无需 API 密钥。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/search?q={query}` | 全文搜索所有实体 |
| `/autocomplete?q={prefix}` | 自动补全实体名称 |
| `/entity/{id}` | 实体详情（基因、疾病、表型） |
| `/entity/{id}/associations` | 实体关联信息 |
| `/entity/{id}/associations?category={cat}` | 筛选后的关联信息 |

## 实体 ID 前缀
- `MONDO:` — 疾病（例如 `MONDO:0007947`）
- `HP:` — 表型（例如 `HP:0001250`）
- `HGNC:` — 基因（例如 `HGNC:3603`）
- `NCBIGene:` — 基因（例如 `NCBIGene:7157`）

## 关联类别
`biolink:GeneToPhenotypicFeatureAssociation`, `biolink:DiseaseToPhenotypicFeatureAssociation`, `biolink:GeneToDiseaseAssociation`

## 示例调用
```
# 搜索马凡综合征
https://api.monarchinitiative.org/v3/api/search?q=Marfan+syndrome&limit=5

# 疾病实体详情
https://api.monarchinitiative.org/v3/api/entity/MONDO:0007947

# FBN1基因的表型关联
https://api.monarchinitiative.org/v3/api/entity/HGNC:3603/associations?category=biolink:GeneToPhenotypicFeatureAssociation&limit=10
```

## 响应格式
JSON 格式。搜索：`items[]` 包含 `id`、`name`、`category`。关联：`items[]` 包含 `subject`、`predicate`、`object`、`publications`。

## 速率限制
无公开限制。请合理使用。
