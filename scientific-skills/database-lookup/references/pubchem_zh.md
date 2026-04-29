# PubChem PUG REST API

## 基础 URL

```
https://pubchem.ncbi.nlm.nih.gov/rest/pug
```

## URL 模式

```
/{domain}/{namespace}/{identifiers}/{operation}/{output}
```

- **domain**: `compound`, `substance`, `assay`
- **namespace**: `cid`, `name`, `smiles`, `inchi`, `inchikey`, `fastformula`
- **operation**: `record`, `property`, `synonyms`, `description`, `cids`, `xrefs`
- **output**: `JSON`, `XML`, `CSV`, `TXT`, `SDF`, `PNG`

## 关键端点

### 按名称搜索
```
GET /compound/name/{name}/JSON
```
示例：`/compound/name/aspirin/JSON`

### 按 CID 搜索
```
GET /compound/cid/{cid}/JSON
```
示例：`/compound/cid/2244/JSON`

多个 CID：`/compound/cid/2244,5988,3672/JSON`

### 按 SMILES 搜索
```
GET /compound/smiles/{smiles}/JSON
```
含特殊字符的 SMILES 需使用 POST：
```
POST /compound/smiles/JSON
Content-Type: application/x-www-form-urlencoded
smiles=CC(=O)OC1=CC=CC=C1C(=O)O
```

### 按 InChIKey 搜索
```
GET /compound/inchikey/{inchikey}/JSON
```

### 按 InChI 搜索（仅限 POST - InChI 字符串过长不适用于 URL）
```
POST /compound/inchi/JSON
Content-Type: application/x-www-form-urlencoded
inchi=InChI=1S/C9H8O4/...
```

### 按分子式搜索
```
GET /compound/fastformula/{formula}/JSON
```
示例：`/compound/fastformula/C9H8O4/JSON`

### 属性获取
```
GET /compound/{namespace}/{id}/property/{property_list}/JSON
```
属性以逗号分隔。可用属性包括：

`MolecularFormula`, `MolecularWeight`, `CanonicalSMILES`, `IsomericSMILES`, `InChI`, `InChIKey`, `IUPACName`, `XLogP`, `ExactMass`, `MonoisotopicMass`, `TPSA`, `Complexity`, `Charge`, `HBondDonorCount`, `HBondAcceptorCount`, `RotatableBondCount`, `HeavyAtomCount`, `CID`

示例：
```
/compound/cid/2244/property/MolecularFormula,MolecularWeight,CanonicalSMILES,IUPACName/JSON
```

响应：
```json
{
  "PropertyTable": {
    "Properties": [
      {
        "CID": 2244,
        "MolecularFormula": "C9H8O4",
        "MolecularWeight": 180.16,
        "IUPACName": "2-acetyloxybenzoic acid",
        "CanonicalSMILES": "CC(=O)OC1=CC=CC=C1C(O)=O"
      }
    ]
  }
}
```

### 同义词查询
```
GET /compound/{namespace}/{id}/synonyms/JSON
```

### 化合物描述
```
GET /compound/cid/{cid}/description/JSON
```

### 从名称获取 CID
```
GET /compound/name/{name}/cids/JSON
```

### 交叉引用（专利、注册号）
```
GET /compound/cid/{cid}/xrefs/PatentID/JSON
GET /compound/cid/{cid}/xrefs/RegistryID/JSON
```

### 相似性搜索（POST，返回异步检索的 listkey）
```
POST /compound/fastsimilarity_2d/smiles/cids/JSON
smiles=CC(=O)OC1=CC=CC=C1C(=O)O&Threshold=90
```

### 二维结构图像
```
GET /compound/cid/{cid}/PNG
GET /compound/cid/{cid}/PNG?image_size=300x300
```

## 速率限制

- 最高 **每秒 5 次请求**
- 最高 **每分钟 400 次请求**
- GET 请求批量 CID 上限 100 个，POST 约 10,000 个
- 限流错误返回 `PUGREST.ServerBusy` 错误码

## 错误格式

```json
{
  "Fault": {
    "Code": "PUGREST.NotFound",
    "Message": "未找到CID",
    "Details": ["..."]
  }
}
```
