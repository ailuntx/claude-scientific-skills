# arXiv API

arXiv 是一个预印本服务器，涵盖物理学、数学、计算机科学、定量生物学、定量金融学、统计学、电气工程和经济学领域。

**重要提示：** arXiv API 返回 **Atom XML** 而非 JSON。不提供 JSON 格式选项。

## 基础 URL

```
https://export.arxiv.org/api/query
```

## 认证

无需认证。完全公开。

## 查询参数

```
GET https://export.arxiv.org/api/query?search_query={query}&start={n}&max_results={n}
```

| 参数 | 必填 | 默认值 | 描述 |
|-----------|----------|---------|-------------|
| `search_query` | 是* | -- | 使用字段前缀 + 布尔运算符搜索 |
| `id_list` | 是* | -- | 逗号分隔的 arXiv ID（例如 `2103.15348,2005.14165`） |
| `start` | 否 | 0 | 分页偏移量（从0开始） |
| `max_results` | 否 | 10 | 每请求结果数（上限2000；绝对上限30000） |
| `sortBy` | 否 | `relevance` | `relevance`（相关性）、`lastUpdatedDate`（最后更新日期）、`submittedDate`（提交日期） |
| `sortOrder` | 否 | `descending` | `ascending`（升序）或 `descending`（降序） |

*必须至少提供 `search_query` 或 `id_list` 中的一个参数。两者可组合使用（取交集）。

## 搜索字段前缀

| 前缀 | 搜索范围 |
|--------|----------|
| `ti:` | 标题 |
| `au:` | 作者 |
| `abs:` | 摘要 |
| `co:` | 评论 |
| `jr:` | 期刊引用 |
| `cat:` | 学科分类 |
| `rn:` | 报告编号 |
| `all:` | 全部字段 |

## 布尔运算符

- `AND` -- 同时满足两个条件
- `OR` -- 满足任一条件
- `ANDNOT` -- 排除条件
- 括号分组（URL编码为 `%28` / `%29`）
- 引号包裹短语（URL编码为 `%22`）

## 查询示例

**全字段搜索：**
```
https://export.arxiv.org/api/query?search_query=all:transformer+attention&max_results=5
```

**作者 + 分类：**
```
https://export.arxiv.org/api/query?search_query=au:hinton+AND+cat:cs.LG&max_results=10
```

**标题搜索：**
```
https://export.arxiv.org/api/query?search_query=ti:%22attention+is+all+you+need%22
```

**按 ID 查询：**
```
https://export.arxiv.org/api/query?id_list=2103.15348
```

**多个 ID：**
```
https://export.arxiv.org/api/query?id_list=2103.15348,2005.14165,1706.03762
```

**日期范围：**
```
https://export.arxiv.org/api/query?search_query=cat:cs.AI+AND+submittedDate:[202401010000+TO+202412312359]
```

## 响应格式（Atom XML）

```xml
<feed xmlns="http://www.w3.org/2005/Atom">
  <opensearch:totalResults>1234</opensearch:totalResults>
  <opensearch:startIndex>0</opensearch:startIndex>
  <opensearch:itemsPerPage>10</opensearch:itemsPerPage>

  <entry>
    <id>http://arxiv.org/abs/1706.03762v7</id>
    <title>Attention Is All You Need</title>
    <summary>The dominant sequence transduction models are based on...</summary>
    <published>2017-06-12T17:57:34Z</published>
    <updated>2023-08-02T00:00:12Z</updated>
    <author><name>Ashish Vaswani</name></author>
    <author><name>Noam Shazeer</name></author>
    <!-- 更多作者 -->
    <category term="cs.CL" scheme="http://arxiv.org/schemas/atom"/>
    <arxiv:primary_category term="cs.CL"/>
    <link rel="alternate" href="http://arxiv.org/abs/1706.03762v7"/>
    <link rel="related" type="application/pdf" href="http://arxiv.org/pdf/1706.03762v7"/>
    <arxiv:doi>10.48550/arXiv.1706.03762</arxiv:doi>
    <arxiv:comment>15 pages, 5 figures</arxiv:comment>
    <arxiv:journal_ref>Advances in Neural Information Processing Systems 30 (NIPS 2017)</arxiv:journal_ref>
  </entry>
</feed>
```

### 条目关键 XML 元素

| 元素 | 描述 |
|---------|-------------|
| `<id>` | arXiv URL：`http://arxiv.org/abs/{id}` |
| `<title>` | 论文标题 |
| `<summary>` | 摘要 |
| `<published>` | 原始提交日期（ISO 8601格式） |
| `<updated>` | 最新版本日期 |
| `<author><name>` | 作者姓名（每作者一个标签） |
| `<category term="...">` | 学科分类 |
| `<arxiv:primary_category>` | 主分类 |
| `<link rel="alternate">` | 摘要页面 URL |
| `<link rel="related" title="pdf">` | PDF 文件 URL |
| `<arxiv:doi>` | DOI（若存在） |
| `<arxiv:comment>` | 作者评论 |
| `<arxiv:journal_ref>` | 期刊引用信息 |

## 解析建议

由于 arXiv 返回 XML 格式数据，需进行解析。使用 `curl` 时可通过管道输出并提取所需内容。XML 命名空间为 `http://www.w3.org/2005/Atom`，arXiv 扩展位于 `http://arxiv.org/schemas/atom`。

实际提取时，关键数据位于 `<entry>` 元素中。每个条目的 `<id>` 包含 URL 路径中的 arXiv ID。

## 常用分类

| 分类 | 领域 |
|----------|-------|
| `cs.AI` | 人工智能 |
| `cs.CL` | 计算与语言（自然语言处理） |
| `cs.CV` | 计算机视觉 |
| `cs.LG` | 机器学习 |
| `stat.ML` | 机器学习（统计学） |
| `q-bio` | 定量生物学 |
| `physics` | 物理学（全部分支） |
| `math` | 数学（全部分支） |
| `econ` | 经济学 |
| `eess` | 电气工程与系统科学 |

完整列表：https://arxiv.org/category_taxonomy

## 速率限制

- **每 3 秒仅限 1 次请求**（硬性限制）
- 仅允许单连接
- 搜索结果每日缓存——相同查询 24 小时内不显示新结果
- 批量数据请改用 OAI-PMH 接口
