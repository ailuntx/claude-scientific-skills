# ENCODE（DNA元素百科全书）

## 基础URL
```
https://www.encodeproject.org
```

## 认证
无需认证。附加 `?format=json` 或设置 `Accept: application/json` 请求头。

## 所有门户URL在携带正确请求头时均返回JSON

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/search/?type=Experiment&format=json` | 搜索实验 |
| `/experiments/{accession}/?format=json` | 特定实验 |
| `/files/{accession}/?format=json` | 文件元数据 |
| `/biosamples/{accession}/?format=json` | 生物样本信息 |
| `/annotations/?format=json` | 搜索注释 |

## 搜索参数
- `type` — 实验(Experiment)、文件(File)、生物样本(Biosample)、注释(Annotation)等
- `assay_title` — ChIP-seq、RNA-seq、ATAC-seq等
- `target.label` — 靶蛋白（如CTCF、H3K27ac）
- `biosample_ontology.term_name` — 细胞类型
- `limit` — 每页结果数
- `field` — 指定返回字段

## 调用示例
```
# CTCF的ChIP-seq实验
https://www.encodeproject.org/search/?type=Experiment&assay_title=ChIP-seq&target.label=CTCF&format=json&limit=5

# 特定实验
https://www.encodeproject.org/experiments/ENCSR000AAA/?format=json

# 某实验的文件
https://www.encodeproject.org/search/?type=File&dataset=/experiments/ENCSR000AAA/&format=json
```

## 响应格式
JSON-LD。搜索响应包含：`@graph`数组 + `total`总数 + `facets`分面。使用`frame=object`或`frame=embedded`控制数据深度。

## 速率限制
无公开限制。建议使用`limit=`和`field=`参数减少响应负载。
