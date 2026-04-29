# DrugBank API

## 重要提示：DrugBank 的完整 API 为商业服务（需付费许可）

**免费药物数据替代方案：**
- **ChEMBL** —— 丰富的生物活性数据，提供免费 API
- **PubChem** —— 免费化合物数据库
- **OpenFDA** —— 药品标签、不良反应事件数据
- **DGIdb** (https://dgidb.org/api) —— 药物-基因相互作用数据库，免费接口

## 基础 URL（付费 API）
```
https://api.drugbank.com/v1
```

## 认证
需 API 密钥：`Authorization: Bearer <api_key>`

## 核心端点（付费 API）

| 端点路径 | 描述 |
|----------|-------------|
| `/drugs/{drugbank_id}` | 通过 DrugBank ID 获取药物信息 |
| `/drugs?q={query}` | 药物检索 |
| `/drugs/{id}/interactions` | 药物相互作用数据 |
| `/drugs/{id}/targets` | 药物作用靶点 |
| `/drugs/{id}/enzymes` | 代谢酶信息 |
| `/drugs/{id}/pathways` | 相关代谢通路 |
| `/drugs/{id}/adverse_effects` | 不良反应数据 |
| `/drug_interactions?drugbank_id={id1},{id2}` | 检查特定药物相互作用 |

## 调用示例
```
GET /drugs/DB00945  (阿司匹林)
GET /drugs?q=aspirin
GET /drugs/DB00945/interactions
GET /drugs/DB00945/targets
```

## 响应格式
```json
{
  "drugbank_id": "DB00945",
  "name": "Acetylsalicylic acid",
  "cas_number": "50-78-2",
  "groups": ["approved"],
  "targets": [{"name": "Prostaglandin G/H synthase 1", "uniprot_id": "P23219", "gene_name": "PTGS1", "actions": ["inhibitor"]}],
  "external_ids": {"chembl": "CHEMBL25", "pubchem_compound": "2244"}
}
```

## 免费获取途径
- **DrugBank 开放数据**：提供约 2,500 种 FDA 批准药物的 XML/CSV 下载，访问地址：https://go.drugbank.com/releases/latest
- **学术许可**：非商业用途免费，提供数据下载（非 API 接口）
