# TCGA / GDC 数据门户 API

## 基础 URL
```
https://api.gdc.cancer.gov
```

## 认证
公开数据无需认证。仅下载受控访问数据时需要令牌。

## 关键端点

所有搜索端点支持 GET 或 POST 请求（复杂筛选建议使用 POST）。

| 端点 | 描述 |
|----------|-------------|
| `/projects` | 列出/筛选癌症项目（如 TCGA-BRCA） |
| `/cases` | 搜索病例（患者/样本） |
| `/files` | 搜索/筛选文件（BAM、VCF、表达数据） |
| `/genes` | 搜索基因层面数据 |
| `/ssms` | 搜索简单体细胞突变 |
| `/ssm_occurrences` | 跨病例的突变发生情况 |
| `/files/{uuid}` | 通过 UUID 获取文件元数据 |
| `/data/{uuid}` | 通过 UUID 下载文件 |

## 筛选语法（POST 请求体）
```json
{
  "filters": {
    "op": "in",
    "content": {"field": "cases.project.project_id", "value": ["TCGA-BRCA"]}
  },
  "fields": "file_id,file_name,data_type",
  "format": "JSON",
  "size": 10
}
```
操作符：`in`, `=`, `!=`, `>`, `<`, `>=`, `<=`, `is`, `not`, `and`, `or`

## 调用示例
```
# 列出项目
https://api.gdc.cancer.gov/projects?size=5&fields=project_id,name,primary_site

# BRCA1 基因突变 (POST)
curl -X POST https://api.gdc.cancer.gov/ssms \
  -H "Content-Type: application/json" \
  -d '{"filters":{"op":"in","content":{"field":"consequence.transcript.gene.symbol","value":["BRCA1"]}},"fields":"ssm_id,genomic_dna_change","size":5}'

# TCGA-LUAD 项目中的病例
https://api.gdc.cancer.gov/cases?filters=%7B%22op%22%3A%22in%22%2C%22content%22%3A%7B%22field%22%3A%22project.project_id%22%2C%22value%22%3A%5B%22TCGA-LUAD%22%5D%7D%7D&size=3&fields=submitter_id,disease_type
```

## 分页
`from`（起始位置）和 `size`（数量限制，上限 10000）。默认数量为 10。

## 速率限制
元数据查询无严格限制。批量文件下载请使用 GDC Transfer Tool。
