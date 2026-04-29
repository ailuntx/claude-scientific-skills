# GWAS目录（EBI）

## 基础URL
```
https://www.ebi.ac.uk/gwas/rest/api
```

## 认证
无需API密钥。

## 注意：响应使用HAL+JSON格式，包含`_links`和`_embedded`键。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/studies/{accession}` | 单个研究（例如GCST001633） |
| `/studies/search/findByPubmedId?pubmedId={id}` | 通过PubMed ID查找研究 |
| `/singleNucleotidePolymorphisms/{rsId}` | SNP详情 |
| `/singleNucleotidePolymorphisms/{rsId}/associations` | SNP关联性数据 |
| `/singleNucleotidePolymorphisms/search/findByRsId?rsId={rsId}` | 通过rsID搜索 |
| `/associations` | 关联性列表 |
| `/associations/{id}` | 单个关联性数据 |
| `/efoTraits` | EFO性状列表 |
| `/efoTraits/search/findByEfoTrait?trait={name}` | 性状搜索 |

## 分页
`?page=0&size=20`（从零开始计数，最大约500条）

## 调用示例
```
# 获取研究
https://www.ebi.ac.uk/gwas/rest/api/studies/GCST001633

# 获取SNP关联性
https://www.ebi.ac.uk/gwas/rest/api/singleNucleotidePolymorphisms/rs7329174/associations

# 搜索性状
https://www.ebi.ac.uk/gwas/rest/api/efoTraits/search/findByEfoTrait?trait=diabetes&page=0&size=5
```

## 响应格式
HAL+JSON。结果位于`_embedded.studies[]`或`_embedded.associations[]`中。关键字段：`pvalue`、`riskFrequency`、`orPerCopyNum`、`betaNum`。

## 速率限制
未公布限制。可通过FTP获取批量数据：ftp.ebi.ac.uk/pub/databases/gwas/
