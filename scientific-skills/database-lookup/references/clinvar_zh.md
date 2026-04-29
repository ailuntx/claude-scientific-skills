# ClinVar API 参考文档

## 基础 URL
- **NCBI E-utilities**：`https://eutils.ncbi.nlm.nih.gov/entrez/eutils`
- **ClinVar 网络 API (VCV)**：`https://www.ncbi.nlm.nih.gov/clinvar`
- **NCBI 变异服务**：`https://api.ncbi.nlm.nih.gov/variation/v0`

## 认证
- E-utilities：无需密钥，但**强烈建议注册**。访问 https://www.ncbi.nlm.nih.gov/account/ 获取 `api_key`
- 无密钥：3 次请求/秒。有密钥：10 次请求/秒
- 在所有 E-utility 请求后附加 `&api_key=YOUR_KEY`

## 速率限制
- 无 API 密钥：3 次请求/秒
- 有 API 密钥：10 次请求/秒

## 核心端点

### 1. 检索 ClinVar (esearch)
```
GET https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=clinvar&term={query}&retmode=json
```
示例 — 检索 BRCA1 致病性变异：
```
GET https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=clinvar&term=BRCA1[gene]+AND+pathogenic[clinical_significance]&retmode=json&retmax=10
```
返回包含 ClinVar 变异 ID 列表 `idlist` 的 JSON

### 2. 获取 ClinVar 记录 (esummary)
```
GET https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=clinvar&id={id_list}&retmode=json
```
示例：
```
GET https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=clinvar&id=37088,37087&retmode=json
```
返回包含临床意义、变异名称、基因、疾病、审核状态的 JSON

### 3. 完整记录 (efetch)
```
GET https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=clinvar&id={id}&rettype=vcv&is_variationid&retmode=xml
```
注意：ClinVar efetch 仅返回 **XML 格式**（不支持 JSON）

### 4. 变异服务 API — SPDI/HGVS 查询
```
GET https://api.ncbi.nlm.nih.gov/variation/v0/spdi/{spdi_expression}/clinvar
GET https://api.ncbi.nlm.nih.gov/variation/v0/hgvs/{hgvs_expression}/clinvar
```
示例：
```
GET https://api.ncbi.nlm.nih.gov/variation/v0/hgvs/NM_007294.4%3Ac.5266dupC/clinvar
```

### 5. ClinVar VCV/RCV 直接访问
```
GET https://www.ncbi.nlm.nih.gov/clinvar/variation/{variation_id}/?redir=vcv
```
此接口返回 HTML。程序化访问请使用 E-utilities 或变异服务 API

## 常用检索限定符
- `[gene]` — 基因符号 (例：`BRCA1[gene]`)
- `[clinical_significance]` — 致病性、可能致病性、良性、意义未明
- `[molecular_consequence]` — 错义、无义、移码等
- `[review_status]` — 单一提交者提供证据、专家小组审核等
- `[condition]` — 疾病名称

## 响应格式
- esearch/esummary：JSON (需指定 `retmode=json`)
- efetch：ClinVar 仅支持 XML
- 变异服务：JSON

## esummary 响应关键字段
```json
{
  "result": {
    "37088": {
      "uid": "37088",
      "title": "NM_007294.4(BRCA1):c.5266dupC (p.Gln1756Profs*74)",
      "clinical_significance": { "description": "Pathogenic" },
      "genes": [{"symbol": "BRCA1", "geneid": 672}],
      "variation_set": [...],
      "trait_set": [{"trait_name": "遗传性乳腺癌卵巢癌综合征"}]
    }
  }
}
```

## 注意事项
- 组合使用 esearch + esummary 实现"检索-获取"工作流
- 批量下载请使用 ClinVar FTP：https://ftp.ncbi.nlm.nih.gov/pub/clinvar/
