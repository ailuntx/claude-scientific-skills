# PRIDE 归档 REST API 参考

## 概述
EMBL-EBI 的 PRIDE（蛋白质组学鉴定数据库）提供完整的公共 REST API，用于查询蛋白质组学数据集、蛋白质、肽段和谱图。

## 基础 URL
```
https://www.ebi.ac.uk/pride/ws/archive/v2
```
（旧版 v1 仍存在，但 v2 为当前版本）

## 认证
- 读取访问**无需认证**
- 开放且免费使用

## 关键端点

| 端点 | 描述 |
|---|---|
| `GET /projects` | 搜索/列出蛋白质组学项目 |
| `GET /projects/{accession}` | 通过 PXD 编号获取特定项目 |
| `GET /projects/{accession}/files` | 列出项目的文件 |
| `GET /spectra` | 搜索谱图 |
| `GET /peptideevidences` | 搜索肽段证据 |
| `GET /proteinevidences` | 搜索蛋白质证据 |
| `GET /stats` | 数据库统计信息 |

## 查询参数
- `keyword` — 自由文本搜索
- `filter` — 字段特定筛选器（例如：物种、仪器、修饰）
- `pageSize` — 每页结果数（默认 10，最大 100）
- `page` — 页码（从0开始）
- `sortDirection` — 升序 ASC 或降序 DESC
- `sortFields` — 排序字段

## 调用示例

```bash
# 通过关键词搜索项目
curl "https://www.ebi.ac.uk/pride/ws/archive/v2/projects?keyword=alzheimer&pageSize=5"

# 获取特定项目
curl "https://www.ebi.ac.uk/pride/ws/archive/v2/projects/PXD010000"

# 列出项目的文件
curl "https://www.ebi.ac.uk/pride/ws/archive/v2/projects/PXD010000/files?pageSize=10"

# 按物种搜索（人类 = 9606）
curl "https://www.ebi.ac.uk/pride/ws/archive/v2/projects?filter=organisms_facet==9606&pageSize=5"

# 获取数据库统计信息
curl "https://www.ebi.ac.uk/pride/ws/archive/v2/stats"
```

## 响应格式
JSON 格式。示例（项目）：
```json
{
  "accession": "PXD010000",
  "title": "项目标题",
  "projectDescription": "...",
  "organisms": [{"accession": "9606", "name": "智人"}],
  "instruments": [{"name": "Q Exactive"}],
  "submissionDate": "2018-05-01",
  "publicationDate": "2018-09-01",
  "numAssays": 12,
  "references": [{"pubmedId": 12345678}]
}
```

## 速率限制
- 无严格公布的速率限制，但需遵守标准 EBI 公平使用政策
- 建议：每秒限制少量请求
- 批量数据可通过 FTP/Aspera 获取：ftp.pride.ebi.ac.uk
