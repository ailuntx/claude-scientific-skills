# QuickGO (EBI GO 注释浏览器)

## 基础 URL
```
https://www.ebi.ac.uk/QuickGO/services/
```

## 认证
无需认证。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/ontology/go/terms/{goId}` | GO 术语详情 |
| `/ontology/go/terms/{goId}/children` | 子级术语 |
| `/ontology/go/terms/{goId}/ancestors` | 祖先术语 |
| `/ontology/go/search?query={term}` | 通过关键词搜索 GO 术语 |
| `/annotation/search` | 通过基因/分类群/GO 术语搜索注释 |

## 注释搜索参数
- `goId` — GO 术语 (如 GO:0003723)
- `taxonId` — NCBI 分类编号 (如 9606 表示人类)
- `geneProductId` — UniProt 登录号
- `evidence` — 证据代码 (如 ECO:0000269)
- `aspect` — 生物过程、分子功能、细胞组分
- `limit`, `page` — 分页参数

## 调用示例
```
# 获取 GO 术语详情
https://www.ebi.ac.uk/QuickGO/services/ontology/go/terms/GO:0003723

# 人类基因的 RNA 结合注释
https://www.ebi.ac.uk/QuickGO/services/annotation/search?goId=GO:0003723&taxonId=9606&limit=10

# 通过关键词搜索术语
https://www.ebi.ac.uk/QuickGO/services/ontology/go/search?query=apoptosis&limit=5
```

## 响应格式
JSON 格式。注释数据采用分页结果，包含基因产物、GO 术语、证据和限定符。

## 速率限制
遵循 EBI 合理使用政策。获取大型结果集请使用下载端点。
