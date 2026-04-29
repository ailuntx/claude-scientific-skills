# 代谢组学工作台 REST API

## 基础 URL
```
https://www.metabolomicsworkbench.org/rest/
```

## 认证
无需 API 密钥。完全公开。

## URL 结构
```
/rest/{上下文}/{输入项}/{输入值}/{输出项}
```

上下文：`study`, `compound`, `refmet`, `gene`, `protein`, `moverz`, `exactmass`

## 关键端点

### 研究上下文
| URL 模式 | 描述 |
|---|---|
| `/rest/study/study_id/{ST_ID}/summary` | 研究摘要元数据 |
| `/rest/study/study_id/{ST_ID}/metabolites` | 研究中的代谢物 |
| `/rest/study/study_id/{ST_ID}/analysis` | 分析详情 |
| `/rest/study/study_id/{ST_ID}/factors` | 实验因素 |
| `/rest/study/study_id/{ST_ID}/data` | 命名的代谢物数据矩阵 |
| `/rest/study/study_id/{ST_ID}/species` | 物种信息 |
| `/rest/study/study_id/{ST_ID}/disease` | 疾病信息 |
| `/rest/study/study_title/{keyword}/summary` | 按标题关键词搜索研究 |
| `/rest/study/study_type/{type}/summary` | 按研究类型搜索 |
| `/rest/study/analysis_id/{AN_ID}/summary` | 按分析 ID 获取摘要 |

研究 ID：`ST######` (例如 `ST000001`)。分析 ID：`AN######`。

### 化合物上下文
| URL 模式 | 描述 |
|---|---|
| `/rest/compound/name/{NAME}/summary` | 按名称搜索化合物 |
| `/rest/compound/pubchem_cid/{CID}/summary` | 按 PubChem CID 搜索 |
| `/rest/compound/hmdb_id/{HMDB_ID}/summary` | 按 HMDB ID 搜索 |
| `/rest/compound/kegg_id/{KEGG_ID}/summary` | 按 KEGG ID 搜索 |
| `/rest/compound/inchi_key/{KEY}/summary` | 按 InChI 密钥搜索 |
| `/rest/compound/regno/{REGNO}/classification` | 化合物分类 |
| `/rest/compound/regno/{REGNO}/molfile` | MOL 文件（结构） |

### RefMet（标准化命名）
| URL 模式 | 描述 |
|---|---|
| `/rest/refmet/name/{NAME}/all` | 完整 RefMet 记录 |
| `/rest/refmet/match/{NAME}/name` | 匹配名称到标准化 RefMet 名称 |

### 基因/蛋白质上下文
| URL 模式 | 描述 |
|---|---|
| `/rest/gene/gene_symbol/{SYMBOL}/all` | 按符号获取基因信息 |
| `/rest/gene/gene_id/{ID}/all` | 按 Entrez ID 获取基因信息 |
| `/rest/protein/uniprot_id/{ID}/all` | 按 UniProt ID 获取蛋白质信息 |

### 质量搜索（MoverZ / ExactMass）
```
/rest/moverz/mz/{MZ_VALUE}/tol/{TOLERANCE}/mode/{正|负}
/rest/exactmass/mass/{MASS_VALUE}/tol/{TOLERANCE}
```

## 调用示例

```
# 研究摘要
https://www.metabolomicsworkbench.org/rest/study/study_id/ST000001/summary

# 研究中的代谢物
https://www.metabolomicsworkbench.org/rest/study/study_id/ST000001/metabolites

# 按标题搜索研究
https://www.metabolomicsworkbench.org/rest/study/study_title/diabetes/summary

# 按名称搜索化合物
https://www.metabolomicsworkbench.org/rest/compound/name/glucose/summary

# 按 PubChem CID 搜索化合物
https://www.metabolomicsworkbench.org/rest/compound/pubchem_cid/5793/summary

# RefMet 标准化名称匹配
https://www.metabolomicsworkbench.org/rest/refmet/match/alpha-D-Glucose/name

# 正模式下的 m/z 搜索
https://www.metabolomicsworkbench.org/rest/moverz/mz/175.0354/tol/0.005/mode/pos

# 精确质量搜索
https://www.metabolomicsworkbench.org/rest/exactmass/mass/174.0282/tol/0.005
```

## 响应格式
默认为 JSON。`mwtab` 输出返回 MWTab 文本。`molfile` 返回 MOL/SDF 文本。无分页——返回完整结果。

## 速率限制
无公开限制。请合理使用。批量调用时建议添加 0.5-1 秒延迟。
