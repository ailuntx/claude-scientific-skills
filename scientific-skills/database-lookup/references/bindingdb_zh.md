# BindingDB REST API

## 基础URL
```
https://bindingdb.org/rest/
https://bindingdb.org/axis2/services/BDBService/
```

## 认证
无需API密钥。完全开放且免费。

## 响应格式
默认返回XML。在任何端点后附加`&response=application/json`可获取JSON格式。

## 核心端点

| 端点 | 描述 |
|----------|-------------|
| `/rest/getLigandsByUniprot` | 获取单个蛋白质靶点的配体 |
| `/rest/getLigandsByUniprots` | 获取多个蛋白质靶点的配体 |
| `/rest/getLigandsByPDBs` | 通过PDB结构ID获取配体 |
| `/rest/getTargetByCompound` | 获取化合物的靶点（基于SMILES相似性） |

## 端点详情

### 获取单个靶点的配体
```
GET https://bindingdb.org/rest/getLigandsByUniprot?uniprot={UNIPROT_ID};{IC50_cutoff_nM}&response=application/json
```
- `uniprot` — UniProt ID后接`;`及纳摩尔级亲和力阈值
- 返回monomerID、SMILES、亲和力类型（IC50/Ki/Kd）及数值
- 若UniProt ID不存在则返回空字符串

示例：
```
https://bindingdb.org/rest/getLigandsByUniprot?uniprot=P35355;100&response=application/json
```

### 获取多个靶点的配体
```
GET https://bindingdb.org/rest/getLigandsByUniprots?uniprot={IDs}&cutoff={nM}&response=application/json
```
- `uniprot` — 逗号分隔的UniProt ID列表
- `cutoff` — 纳摩尔级亲和力阈值
- 若无匹配ID则返回空字符串

示例：
```
https://bindingdb.org/rest/getLigandsByUniprots?uniprot=P00176,P00183&cutoff=10000&response=application/json
```

### 通过PDB结构获取配体
```
GET https://bindingdb.org/rest/getLigandsByPDBs?pdb={PDBs}&cutoff={nM}&identity={percent}&response=application/json
```
- `pdb` — 逗号分隔的PDB ID列表
- `cutoff` — 纳摩尔级亲和力阈值
- `identity` — 序列一致性阈值（百分比，如92）

示例：
```
https://bindingdb.org/rest/getLigandsByPDBs?pdb=1Q0L,3ANM&cutoff=100&identity=92&response=application/json
```

### 查找化合物的靶点（相似性搜索）
```
GET https://bindingdb.org/rest/getTargetByCompound?smiles={SMILES}&cutoff={similarity}&response=application/json
```
- `smiles` — 化合物SMILES（需URL编码）
- `cutoff` — Tanimoto相似度阈值（小数，如0.85）
- 返回相似化合物及其蛋白靶点与亲和力数据

示例：
```
https://bindingdb.org/rest/getTargetByCompound?smiles=CCC%5BN%2B%5D%28C%29%28C%29CCn1nncc1COc1cc%28%3DO%29n%28C%29c2ccccc12&cutoff=0.85&response=application/json
```

## 速率限制
无文档化限制。建议保持每秒约1次请求以维护服务友好性。

## 注意事项
- API接口精简（仅4个端点），专注于结合亲和力数据
- 化合物名称搜索需先通过PubChem解析为SMILES，再使用`getTargetByCompound`
- 批量数据访问请使用https://www.bindingdb.org/bind/chemsearch/marvin/Download.jsp提供的TSV/SDF下载文件
- 包含约320万条结合测量数据，覆盖140万种化合物和1.14万个靶点
