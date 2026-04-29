# DisGeNET（基因-疾病关联）

## 基础 URL
```
https://www.disgenet.org/api
```

## 认证
**需提供 API 密钥。** 在 disgenet.org 注册后，通过以下方式认证：
```bash
curl -X POST https://www.disgenet.org/api/auth/ \
  -d 'email=you@example.com&password=yourpassword'
# 返回：{"token": "abc123..."}
```
在请求头中以 `Authorization: Bearer <token>` 形式传递。

从 `.env` 文件加载令牌，变量名为 `DISGENET_API_KEY`。

## 核心端点

| 端点 | 描述 |
|----------|-------------|
| `/gda/gene/{gene_id}` | 基因-疾病关联（NCBI 基因 ID） |
| `/gda/disease/{disease_id}` | 基因-疾病关联（UMLS CUI） |
| `/gda/evidences/gene/{gene_id}` | 证据层级数据 |
| `/vda/gene/{gene_id}` | 基因的变异-疾病关联 |
| `/vda/variant/{rsid}` | 变异-疾病关联（dbSNP rsID） |

## 参数
- `source` — `CURATED`（人工筛选）, `BEFREE`, `ALL`（全部）
- `min_score` — GDA 分数阈值（0-1）
- `min_ei` — 证据指数阈值
- `format` — `json` 或 `tsv`
- `limit`, `offset` — 分页参数

## 调用示例
```
# TP53 基因（ID 7157）的基因-疾病关联
/gda/gene/7157?source=CURATED&min_score=0.3&limit=10&format=json

# 乳腺癌（UMLS CUI C0006142）的疾病-基因关联
/gda/disease/C0006142?limit=10

# rs1042522 变异的变异-疾病关联
/vda/variant/rs1042522
```

## 速率限制
免费学术层级：约每日数百次请求。提供付费层级。

## 免费替代方案
若无 API 密钥：使用 **Open Targets** 获取疾病-基因关联数据。
