# ChEBI（生物相关化学实体）API 参考

## 基础 URL
- **OLS（本体查询服务）API**：`https://www.ebi.ac.uk/ols4/api`
- **ChEBI Web Services (SOAP)**：`https://www.ebi.ac.uk/webservices/chebi/2.0/test`（仅支持 SOAP/XML）
- **ChEBI LibChebi REST（功能有限）**：实体页面位于 `https://www.ebi.ac.uk/chebi`

## 认证
无需认证。所有端点均为公开访问。

## 速率限制
未公布硬性限制。EBI 通用建议：合理使用。

## 重要提示
ChEBI 的主要网络服务基于 **SOAP（XML）** 而非 REST。如需 REST 风格的 JSON 访问，请使用 **EBI OLS4 API**（该服务将 ChEBI 作为本体索引）。

---

## OLS4 API 端点（推荐用于 REST/JSON）

### 1. 搜索 ChEBI 术语
```
GET https://www.ebi.ac.uk/ols4/api/search?q={query}&ontology=chebi
```
示例：
```
GET https://www.ebi.ac.uk/ols4/api/search?q=aspirin&ontology=chebi
```
返回包含匹配 ChEBI 术语、ID、定义、同义词的 JSON。

### 2. 通过 ChEBI ID 查询
```
GET https://www.ebi.ac.uk/ols4/api/ontologies/chebi/terms?iri=http://purl.obolibrary.org/obo/CHEBI_{id}
```
示例：
```
GET https://www.ebi.ac.uk/ols4/api/ontologies/chebi/terms?iri=http://purl.obolibrary.org/obo/CHEBI_15365
```
返回完整术语详情：名称、定义、同义词、交叉引用、关联关系。

### 3. 通过短格式获取术语
```
GET https://www.ebi.ac.uk/ols4/api/ontologies/chebi/terms/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_{id}
```
（路径中为双重编码的 IRI）

### 4. 术语层级 - 父项
```
GET https://www.ebi.ac.uk/ols4/api/ontologies/chebi/terms/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_{id}/parents
```

### 5. 术语层级 - 子项
```
GET https://www.ebi.ac.uk/ols4/api/ontologies/chebi/terms/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_{id}/children
```

### 6. 本体元数据
```
GET https://www.ebi.ac.uk/ols4/api/ontologies/chebi
```

## OLS 搜索响应格式
```json
{
  "response": {
    "numFound": 5,
    "docs": [
      {
        "id": "chebi:15365",
        "iri": "http://purl.obolibrary.org/obo/CHEBI_15365",
        "label": "aspirin",
        "description": ["A member of the class of benzoic acids..."],
        "short_form": "CHEBI_15365",
        "obo_id": "CHEBI:15365",
        "ontology_name": "chebi",
        "type": "class"
      }
    ]
  }
}
```

## ChEBI SOAP 网络服务（替代方案）
如需化学特性数据（分子式、质量、结构、InChI），请使用 SOAP 服务：
- WSDL：`https://www.ebi.ac.uk/webservices/chebi/2.0/webservice?wsdl`
- 操作：`getCompleteEntity`, `getLiteEntity`, `getStructureSearch`, `getOntologyChildren`, `getOntologyParents`
- 仅返回 XML 格式。

`getCompleteEntity` 的 SOAP 请求示例：
```xml
<soapenv:Body>
  <chebi:getCompleteEntity>
    <chebi:chebiId>CHEBI:15365</chebi:chebiId>
  </chebi:getCompleteEntity>
</soapenv:Body>
```
返回：分子式、质量、电荷、InChI、InChIKey、SMILES、同义词、数据库链接、本体父项/子项。

## 注意事项
- 编程式 REST 访问首选 OLS4 方案
- 化学结构搜索（通过 InChI、SMILES、子结构）必须使用 SOAP 服务
- ChEBI ID 为数字形式（如 15365），但在 OBO 格式中引用为 "CHEBI:15365"
- PubChem 和 UniChem 可将 ChEBI ID 与其他化学数据库交叉引用
