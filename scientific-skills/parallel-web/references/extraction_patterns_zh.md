# 提取模式

使用 Parallel 的 Extract API 将网页转换为简洁、LLM 优化内容的指南。

---

## 概述

Extract API 可将任何公开 URL 转换为简洁的 Markdown 格式。它能处理 JavaScript 密集型页面、PDF 文件以及简单 HTTP 抓取无法解析的复杂布局。输出结果针对 LLM 使用进行了优化。

**核心功能：**
- JavaScript 渲染（单页应用、动态内容）
- PDF 文件转纯净文本
- 根据目标返回聚焦片段
- 完整页面内容提取
- 多 URL 批量处理

---

## Extract 与 Search 的使用场景

| 场景 | 使用 Extract | 使用 Search |
|----------|-------------|------------|
| 您有特定 URL | 是 | 否 |
| 需要获取已知页面内容 | 是 | 否 |
| 需查找某主题的相关页面 | 否 | 是 |
| 需要阅读研究论文 URL | 是 | 否 |
| 需验证特定网站的信息 | 是 | 否 |
| 需要广泛查找信息 | 否 | 是 |
| 已通过搜索获得 URL 并需要完整内容 | 是 | 否 |

**经验法则：** 有 URL 时用 Extract，需查找 URL 时用 Search。

---

## 片段模式 vs 完整内容模式

### 片段模式（默认）

返回与目标对齐的聚焦内容。占用更少 token，相关性更高。

```python
extractor = ParallelExtract()

result = extractor.extract(
    urls=["https://arxiv.org/abs/2301.12345"],
    objective="关键方法和实验结果",
    excerpts=True,     # 默认
    full_content=False  # 默认
)
```

**最佳适用场景：**
- 从长页面提取特定信息
- 高效 token 处理
- 明确查找目标时
- 查找论文中的特定主张或数据点

### 完整内容模式

返回页面的完整 Markdown 内容。

```python
result = extractor.extract(
    urls=["https://docs.example.com/api-reference"],
    objective="完整的 API 文档",
    excerpts=False,
    full_content=True,
)
```

**最佳适用场景：**
- 完整文档页面
- 需要全文进行分析
- 需要所有细节而非片段时
- 网页内容存档或转换

### 混合模式

可同时请求片段和完整内容：

```python
result = extractor.extract(
    urls=["https://example.com/report"],
    objective="执行摘要和关键建议",
    excerpts=True,
    full_content=True,
)

# 使用 excerpts 进行聚焦分析
# 使用 full_content 作为完整参考
```

---

## 提取目标撰写

`objective` 参数使提取聚焦于相关内容，显著提升片段质量。

### 优质目标示例

```python
# 具体且可操作
objective="提取方法部分，包括样本量、统计方法和主要终点"

# 明确需求
objective="查找定价信息、功能对比表和企业版详情"

# 任务导向
objective="从该临床试验中提取关键发现、效应量、置信区间和作者结论"
```

### 劣质目标示例

```python
# 过于模糊
objective="介绍这个页面"

# 未提供目标（仍可工作，但片段聚焦性降低）
extractor.extract(urls=["https://..."])
```

### 按场景推荐的目标模板

**学术论文：**
```python
objective="摘要、关键发现、方法（样本量、设计、统计检验）、含效应量和p值的结果、主要结论"
```

**产品/公司页面：**
```python
objective="公司概况、核心产品/服务、定价、创立时间、管理团队及近期公告"
```

**技术文档：**
```python
objective="API端点、认证方法、请求/响应格式、速率限制和代码示例"
```

**新闻文章：**
```python
objective="主要事件、关键引述、数据点、时间线和具名信源"
```

**政府/政策文件：**
```python
objective="核心政策条款、生效日期、受影响方、合规要求和处罚措施"
```

---

## 批量提取

单次调用处理多个 URL：

```python
result = extractor.extract(
    urls=[
        "https://nature.com/articles/s12345",
        "https://science.org/doi/full/10.1234/science.xyz",
        "https://thelancet.com/journals/lancet/article/PIIS0140-6736(24)12345/fulltext"
    ],
    objective="每项研究的关键发现、样本量和统计结果",
)

# 结果顺序与输入URL一致
for r in result["results"]:
    print(f"=== {r['title']} ===")
    print(f"URL: {r['url']}")
    for excerpt in r["excerpts"]:
        print(excerpt[:500])
```

**批量限制：**
- 单次请求无URL数量上限
- 每个URL按单次提取计费
- 大批量处理耗时可能增加
- 失败URL在`errors`字段报告，不影响成功项

---

## 处理不同内容类型

### 网页（HTML）

标准提取模式。支持JavaScript渲染，适用于单页应用和动态内容。

```python
# 标准网页
result = extractor.extract(
    urls=["https://example.com/article"],
    objective="文章主体内容",
)
```

### PDF文件

自动检测PDF并转换为文本。

```python
# PDF提取
result = extractor.extract(
    urls=["https://example.com/whitepaper.pdf"],
    objective="执行摘要和关键建议",
)
```

### 文档站点

完整渲染单页应用和文档框架（Docusaurus, GitBook, ReadTheDocs）。

```python
result = extractor.extract(
    urls=["https://docs.example.com/getting-started"],
    objective="安装说明和快速入门指南",
    full_content=True,
)
```

---

## 常用提取模式

### 模式1：先搜索后提取

通过Search查找相关页面，再从最佳结果中提取完整内容。

```python
from parallel_web import ParallelSearch, ParallelExtract

searcher = ParallelSearch()
extractor = ParallelExtract()

# 步骤1：查找相关页面
search_result = searcher.search(
    objective="查找原始Transformer论文及其关键后续研究",
    search_queries=["attention is all you need paper", "transformer architecture paper"],
)

# 步骤2：从顶部结果提取详细内容
top_urls = [r["url"] for r in search_result["results"][:3]]
extract_result = extractor.extract(
    urls=top_urls,
    objective="摘要、架构描述、关键结果和消融研究",
)
```

### 模式2：DOI解析与论文阅读

```python
# 从DOI链接提取内容
result = extractor.extract(
    urls=["https://doi.org/10.1038/s41586-024-07487-w"],
    objective="研究设计、患者群体、主要终点、疗效结果和安全性数据",
)
```

### 模式3：公司页面竞争情报

```python
companies = [
    "https://openai.com/about",
    "https://anthropic.com/company",
    "https://deepmind.google/about/",
]

result = extractor.extract(
    urls=companies,
    objective="公司使命、团队规模、核心产品、近期公告和融资信息",
)
```

### 模式4：文档提取参考

```python
result = extractor.extract(
    urls=["https://docs.parallel.ai/search/search-quickstart"],
    objective="完整API使用指南（含请求格式、响应格式和代码示例）",
    full_content=True,
)
```

### 模式5：元数据验证

```python
# 验证特定论文的引用元数据
result = extractor.extract(
    urls=["https://doi.org/10.1234/example-doi"],
    objective="完整引用元数据：作者、标题、期刊、卷号、页码、年份、DOI",
)
```

---

## 错误处理

### 常见错误

| 错误 | 原因 | 解决方案 |
|-------|-------|----------|
| URL不可访问 | 页面需认证/付费墙/服务中断 | 更换URL或改用Search |
| 超时 | 页面渲染时间过长 | 重试或使用更简单URL |
| 空内容 | 动态加载方式无法渲染 | 尝试完整内容模式或改用Search |
| 速率限制 | 请求过多 | 等待后重试或减少批量 |

### 错误检查

```python
result = extractor.extract(urls=["https://example.com/page"])

if not result["success"]:
    print(f"提取失败: {result['error']}")
elif result.get("errors"):
    print(f"部分URL失败: {result['errors']}")
else:
    print(f"成功提取 {len(result['results'])} 个页面")
```

---

## 技巧与最佳实践

1. **始终提供目标**：即使通用目标也能显著提升片段质量
2. **默认使用片段模式**：仅在确实需要所有内容时启用完整模式
3. **批量关联URL**：单次调用处理5个URL优于5次独立调用
4. **检查错误**：并非所有URL都可提取（付费墙/认证等）
5. **结合Search使用**：Search查找URL，Extract深度读取
6. **用于DOI解析**：Extract自动处理DOI重定向
7. **优先选用Extract**：优于手动抓取（支持JS/PDF/复杂布局）

---

## 另请参阅

- [API参考](api_reference.md) - 完整API参数说明
- [搜索最佳实践](search_best_practices.md) - 查找待提取URL
- [深度研究指南](deep_research_guide.md) - 综合研究任务
- [工作流配方](workflow_recipes.md) - 常见多步骤模式
