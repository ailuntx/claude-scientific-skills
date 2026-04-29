# KEGG REST API

## 基础 URL
```
https://rest.kegg.jp
```

## 认证
无需 API 密钥。学术用途免费，商业用途需授权许可。

## 重要提示：KEGG 返回制表符分隔文本和平面文件格式，而非 JSON。

## 核心操作（基于 URL 路径，无查询参数）

| URL 模式 | 描述 |
|-------------|-------------|
| `/list/{database}` | 列出所有条目 |
| `/list/{database}/{organism}` | 列出特定生物体的条目 |
| `/get/{dbentries}` | 获取条目数据（平面文件） |
| `/get/{dbentries}/image` | 通路图像（PNG 格式） |
| `/get/{dbentries}/kgml` | 通路 KGML XML 格式 |
| `/find/{database}/{query}` | 关键词搜索 |
| `/find/{database}/{query}/formula` | 分子式搜索 |
| `/find/{database}/{value}/exact_mass` | 精确质量搜索 |
| `/link/{target_db}/{source_db}` | 查找跨数据库关联条目 |
| `/link/{target_db}/{dbentries}` | 特定 ID 的关联关系 |
| `/conv/{target_db}/{dbentries}` | 跨数据库 ID 转换 |
| `/ddi/{dbentries}` | 药物相互作用 |

## 数据库代码

| 代码 | 数据库 | 示例 ID |
|------|----------|------------|
| `pathway` | 通路 | `hsa00010` |
| `compound` | 化合物 | `C00001` |
| `drug` | 药物 | `D00001` |
| `enzyme` | 酶 | `ec:1.1.1.1` |
| `genes`/`hsa` | 基因 | `hsa:10458` |
| `disease` | 疾病 | `H00001` |
| `reaction` | 反应 | `R00001` |
| `ko` | KO 直系同源 | `K00001` |

## 调用示例

```
# 列出人类通路
https://rest.kegg.jp/list/pathway/hsa

# 获取通路条目
https://rest.kegg.jp/get/hsa00010

# 按名称搜索化合物
https://rest.kegg.jp/find/compound/aspirin

# 按分子式搜索
https://rest.kegg.jp/find/compound/C9H8O4/formula

# 查找基因关联通路
https://rest.kegg.jp/link/pathway/hsa:10458

# 查找基因关联疾病
https://rest.kegg.jp/link/disease/hsa:672

# KEGG 转 PubChem ID
https://rest.kegg.jp/conv/pubchem/C00001

# 获取多个条目（最多10个，用+连接）
https://rest.kegg.jp/get/C00001+C00002+C00003

# 药物相互作用
https://rest.kegg.jp/ddi/D00564+D00110
```

## 响应格式
list/find/link/conv 返回制表符分隔文本，get 返回平面文件文本。**不支持 JSON**。

## 速率限制
无公开限制。建议每秒不超过数次请求。使用 `+` 在 `/get` 中批量处理最多 10 个 ID。请求过多可能返回 HTTP 403。
