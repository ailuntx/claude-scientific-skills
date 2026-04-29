# GTEx（基因型-组织表达）API参考

## 概述
GTEx收录了来自遗体捐赠者的人类各组织基因表达水平数据，支持研究组织特异性基因调控和eQTL（表达数量性状位点）。

## 基础URL
`https://gtexportal.org/api/v2`

## 认证
无需认证（公开访问）。

## 响应格式
JSON格式。多数端点返回分页结果，结构如下：
```json
{
  "data": [ ... ],
  "paging_info": {
    "numberOfPages": 10,
    "page": 0,
    "maxItemsPerPage": 250
  }
}
```

## 分页参数（通用多数端点）
- `page` -- 页码（从0开始，默认值：0）
- `itemsPerPage` -- 每页结果数（默认值：250，最大值：250）

## 关键端点

### 基因表达（组织间中位数）
```
GET /expression/medianGeneExpression?gencodeId=ENSG00000139618.17&datasetId=gtex_v8
```
参数：
- `gencodeId` -- 带版本号的Ensembl基因ID（必需）
- `datasetId` -- `gtex_v8`（必需）
- `tissueSiteDetailId` -- 筛选特定组织（可选）

返回该基因在各组织的TPM中位值。

### 基因表达（特定组织所有数据）
```
GET /expression/medianGeneExpression?tissueSiteDetailId=Liver&datasetId=gtex_v8
```

### 单组织eQTL分析
```
GET /association/singleTissueEqtl?gencodeId=ENSG00000139618.17&tissueSiteDetailId=Whole_Blood&datasetId=gtex_v8
```
参数：
- `gencodeId` -- 带版本号的Ensembl基因ID（必需）
- `tissueSiteDetailId` -- 组织ID（必需）
- `datasetId` -- `gtex_v8`（必需）

### 多组织eQTL分析
```
GET /association/multiTissueEqtl?gencodeId=ENSG00000139618.17&datasetId=gtex_v8
```

### 基因检索
```
GET /reference/gene?geneId=BRCA2&gencodeVersion=v26&genomeBuild=GRCh38/hg38
```
参数：
- `geneId` -- 基因符号或Ensembl ID
- `gencodeVersion` -- GTEx v8对应`v26`
- `genomeBuild` -- `GRCh38/hg38`

### 组织列表
```
GET /dataset/tissueSiteDetail?datasetId=gtex_v8
```
返回所有组织站点ID、名称、颜色编码及样本数量。

### 外显子表达
```
GET /expression/medianExonExpression?gencodeId=ENSG00000139618.17&datasetId=gtex_v8
```

### 转录本表达
```
GET /expression/medianTranscriptExpression?gencodeId=ENSG00000139618.17&datasetId=gtex_v8
```

### 组织内高表达基因
```
GET /expression/topExpressedGene?tissueSiteDetailId=Brain_Cortex&datasetId=gtex_v8&filterMtGene=true
```

### 位点变异分析（二元）
```
GET /association/dyneqtl?variantId=chr1_1000000_A_G_b38&gencodeId=ENSG00000139618.17&tissueSiteDetailId=Whole_Blood&datasetId=gtex_v8
```

## 组织ID示例
请严格使用下划线分隔的名称：
- `Whole_Blood`（全血）, `Liver`（肝脏）, `Brain_Cortex`（大脑皮层）, `Heart_Left_Ventricle`（左心室）
- `Muscle_Skeletal`（骨骼肌）, `Adipose_Subcutaneous`（皮下脂肪）, `Lung`（肺）, `Skin_Sun_Exposed_Lower_leg`（小腿曝光皮肤）

## 响应示例（基因表达中位值）
```json
{
  "data": [
    {
      "datasetId": "gtex_v8",
      "gencodeId": "ENSG00000139618.17",
      "geneSymbol": "BRCA2",
      "median": 4.523,
      "tissueSiteDetailId": "Whole_Blood",
      "unit": "TPM"
    },
    {
      "datasetId": "gtex_v8",
      "gencodeId": "ENSG00000139618.17",
      "geneSymbol": "BRCA2",
      "median": 12.87,
      "tissueSiteDetailId": "Testis",
      "unit": "TPM"
    }
  ],
  "paging_info": { "numberOfPages": 1, "page": 0, "maxItemsPerPage": 250 }
}
```

## 速率限制
- 未公布明确限制
- 建议合理控制请求频率（约1-2次/秒）
- 批量分析请从GTEx门户下载完整数据集

## 注意事项
- GTEx v8是主数据集，请始终指定`datasetId=gtex_v8`
- 基因ID必须为带版本号的GENCODE ID（如ENSG00000139618.17）
- 使用基因检索端点将符号解析为带版本号的GENCODE ID
- `gencodeVersion=v26`对应GTEx v8版本
