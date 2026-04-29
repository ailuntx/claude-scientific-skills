# JASPAR（转录因子结合谱）

## 基础 URL
```
https://jaspar.elixir.no/api/v1/
```

## 认证
无需认证。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/matrix/` | 列出所有转录因子结合谱 |
| `/matrix/{matrix_id}/` | 特定谱（例如CTCF的MA0139.1） |
| `/matrix/?tax_id={id}&collection=CORE` | 按物种+集合筛选 |
| `/matrix/{id}/?format=jaspar` | JASPAR格式的谱 |
| `/matrix/{id}/?format=meme` | MEME格式的谱 |
| `/matrix/{id}/?format=transfac` | TRANSFAC格式的谱 |
| `/taxon/` | 列出分类群组 |
| `/collection/` | 列出集合（CORE、CNE等） |

## 筛选参数
- `tax_id` — NCBI分类学ID（9606代表人类）
- `collection` — CORE、CNE、PHYLOFACTS等
- `tf_class` — 转录因子结构分类
- `name` — 转录因子名称搜索
- `page`, `page_size` — 分页参数

## 调用示例
```
# 获取CTCF结合谱
https://jaspar.elixir.no/api/v1/matrix/MA0139.1/

# 人类CORE转录因子谱
https://jaspar.elixir.no/api/v1/matrix/?tax_id=9606&collection=CORE&page_size=10

# 以MEME格式获取谱
https://jaspar.elixir.no/api/v1/matrix/MA0139.1/?format=meme
```

## 响应格式
JSON。谱信息包含：`matrix_id`、`name`、`pfm`（位置频率矩阵，以A/C/G/T字典表示）、`sequence_logo` URL、`species`、`class`、`family`。

## API文档
Swagger文档位于 https://jaspar.elixir.no/api/v1/docs/

## 速率限制
未公布限制。请合理使用。
