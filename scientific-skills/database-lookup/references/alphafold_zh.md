# AlphaFold DB（预测蛋白质结构）

## 基础URL
```
https://alphafold.ebi.ac.uk/api/
```

## 认证
无需认证。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/prediction/{uniprot_accession}` | 通过UniProt ID获取预测元数据 |

## 结构文件URL（直接下载）
```
https://alphafold.ebi.ac.uk/files/AF-{UNIPROT}-F1-model_v4.pdb
https://alphafold.ebi.ac.uk/files/AF-{UNIPROT}-F1-model_v4.cif
https://alphafold.ebi.ac.uk/files/AF-{UNIPROT}-F1-predicted_aligned_error_v4.json
```

## 调用示例
```
# 获取EGFR的预测元数据
https://alphafold.ebi.ac.uk/api/prediction/P00533

# 下载PDB结构文件
https://alphafold.ebi.ac.uk/files/AF-P00533-F1-model_v4.pdb

# 下载PAE（预测对齐误差）
https://alphafold.ebi.ac.uk/files/AF-P00533-F1-predicted_aligned_error_v4.json
```

## 响应格式
元数据为JSON格式。结构文件为PDB/mmCIF格式。PAE为JSON矩阵格式。

## 速率限制
无严格限制。批量下载（约200万+结构）请使用FTP/云服务。
