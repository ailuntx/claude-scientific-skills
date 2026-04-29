---
name: 论文查询
description: 通过REST API搜索10个学术论文数据库，查找研究论文、预印本和学术文章。涵盖PubMed、PMC（全文）、bioRxiv、medRxiv、arXiv、OpenAlex、Crossref、Semantic Scholar、CORE、Unpaywall。适用于论文搜索、引用查询、DOI/PMID查找、摘要获取、全文获取、开放获取、预印本、引用图、作者搜索或任何学术文献查询。当用户提及任何支持的数据库或提出类似“查找关于X的论文”或“查找此DOI”的请求时触发。
metadata:
  skill-author: K-Dense Inc.
---

# 论文查询

你可以通过REST API访问10个学术论文数据库。你的任务是找出最适合用户查询的数据库，调用它们，并返回结果。

## 核心工作流

1. **理解查询** -- 用户在找什么？按DOI查找特定论文？某个主题的论文？某位作者的出版物？开放获取PDF？全文？这决定了要查询哪些数据库。

2. **选择数据库** -- 使用下方的数据库选择指南。许多查询适合同时查询多个数据库——例如，先在PubMed搜索论文，再在Unpaywall检查开放获取副本。

3. **阅读参考文件** -- 每个数据库在`references/`中都有一个参考文件，包含端点详情、查询格式和示例调用。在调用API之前先阅读相关文件。

4. **执行API调用** -- 参见下方**执行API调用**部分，了解你的平台上应使用哪个HTTP抓取工具。

5. **返回结果** -- 始终返回：
   - 每个数据库返回的**原始JSON**（或arXiv的解析XML）
   - **查询过的数据库列表**，包括使用的具体端点
   - 如果查询没有返回结果，明确说明，而不是忽略它

## 数据库选择指南

将用户的意图匹配到正确的数据库。

### 按使用场景

| 用户询问的是... | 主要数据库 | 也可考虑 |
|---|---|---|
| 生物医学主题的论文 | PubMed | Semantic Scholar、OpenAlex |
| 生物医学文章的全文 | PMC | CORE |
| 生物学预印本 | bioRxiv | Semantic Scholar、OpenAlex |
| 健康/医学预印本 | medRxiv | Semantic Scholar、OpenAlex |
| 物理、数学或计算机科学预印本 | arXiv | Semantic Scholar、OpenAlex |
| 全学科论文 | OpenAlex | Semantic Scholar、Crossref |
| 按DOI查找特定论文 | Crossref | Unpaywall、Semantic Scholar |
| 论文的开放获取PDF | Unpaywall | CORE、PMC |
| 引用图（谁引用了谁） | Semantic Scholar | OpenAlex |
| 作者的出版物 | Semantic Scholar | OpenAlex |
| 论文推荐 | Semantic Scholar | -- |
| 全文（任何学科） | CORE | PMC（仅生物医学） |
| 期刊/出版商元数据 | Crossref | OpenAlex |
| 资助方信息 | Crossref | OpenAlex |
| 在PMID/PMCID/DOI之间转换 | PMC（ID转换器） | Crossref |
| 按日期查找最新预印本 | bioRxiv、medRxiv | arXiv |

### 跨数据库查询

| 用户询问的是... | 要查询的数据库 |
|---|---|
| 关于一篇论文的所有信息（元数据+引用+OA） | Crossref + Semantic Scholar + Unpaywall |
| 综合文献检索 | PubMed + OpenAlex + Semantic Scholar |
| 查找并阅读一篇论文 | PubMed（查找）+ Unpaywall（OA链接）+ PMC或CORE（全文） |
| 预印本及其发表版本 | bioRxiv/medRxiv + Crossref |
| 作者概览及引用指标 | Semantic Scholar + OpenAlex |

当查询涵盖多个需求时（例如，“查找关于CRISPR的论文并获取PDF”），并行查询相关数据库。

## 常见标识符格式

不同数据库使用不同的标识符系统。如果查询失败，可能是标识符格式错误。

| 标识符 | 格式 | 示例 | 使用方 |
|---|---|---|---|
| DOI | `10.xxxx/xxxxx` | `10.1038/nature12373` | 所有数据库 |
| PMID | 整数 | `34567890` | PubMed、PMC、Semantic Scholar |
| PMCID | `PMC` + 数字 | `PMC7029759` | PMC、Europe PMC |
| arXiv ID | `YYMM.NNNNN` | `2103.15348` | arXiv、Semantic Scholar |
| OpenAlex ID | `W` + 数字 | `W2741809807` | OpenAlex |
| Semantic Scholar ID | 40字符十六进制 | `649def34f8be...` | Semantic Scholar |
| ORCID | `0000-XXXX-XXXX-XXXX` | `0000-0001-6187-6610` | OpenAlex、Crossref |
| ISSN | `XXXX-XXXX` | `0028-0836` | Crossref、OpenAlex |

**交叉引用ID：** Semantic Scholar接受通过前缀传递的DOI、PMID、PMCID和arXiv ID（例如`DOI:10.1038/nature12373`、`PMID:34567890`、`ARXIV:2103.15348`）。OpenAlex接受通过前缀传递的DOI和PMID（`doi:10.1038/...`、`pmid:34567890`）。使用PMC ID转换器在PMID、PMCID和DOI之间进行转换。

## API密钥与访问

这些数据库大多完全开放。部分数据库使用API密钥可获得更高的速率限制。

### 需要或受益于API密钥的数据库

| 数据库 | 环境变量 | 是否必需？ | 注册地址 |
|---|---|---|---|
| NCBI（PubMed、PMC） | `NCBI_API_KEY` | 否（无密钥3 req/s，有密钥10 req/s） | https://www.ncbi.nlm.nih.gov/account/settings/ |
| CORE | `CORE_API_KEY` | 全文需要 | https://core.ac.uk/services/api |
| Semantic Scholar | `S2_API_KEY` | 否（无密钥使用共享池） | https://www.semanticscholar.org/product/api#api-key-form |
| OpenAlex | `OPENALEX_API_KEY` | 推荐 | https://openalex.org/settings/api |

### 完全开放的数据库（无需密钥）

| 数据库 | 备注 |
|---|---|
| bioRxiv / medRxiv | 无需认证，无文档记录的速率限制 |
| arXiv | 无需认证，最大每3秒1个请求 |
| Crossref | 无需认证；添加`mailto`参数可进入礼貌池（速率限制加倍） |
| Unpaywall | 无需认证；需要`email`参数 |

### 加载API密钥

1. **先检查环境** -- 密钥可能已经被导出（例如`$NCBI_API_KEY`）。
2. **回退到`.env`** -- 检查当前工作目录下的`.env`文件。
3. **无密钥继续运行** -- 大多数API仍可在较低速率限制下工作。告知用户缺少哪个密钥以及如何获取。

## 执行API调用

使用你所在环境的HTTP抓取工具调用REST端点：

| 平台 | HTTP抓取工具 | 备选方案 |
|---|---|---|
| Claude Code | `WebFetch` | 通过Bash使用`curl` |
| Gemini CLI | `web_fetch` | 通过shell使用`curl` |
| Windsurf | `read_url_content` | 通过terminal使用`curl` |
| Cursor | 无专用抓取工具 | 通过`run_terminal_cmd`使用`curl` |
| Codex CLI | 无专用抓取工具 | 通过`shell`使用`curl` |
| Cline | 无专用抓取工具 | 通过`execute_command`使用`curl` |

如果抓取工具失败，通过可用的shell工具回退到`curl`。

### 特殊情况

- **arXiv返回Atom XML**，而非JSON。解析它或使用`curl`提取相关字段。如果可用，考虑通过简单解析器进行管道处理。
- **PMC eFetch返回JATS XML**作为全文。这是预期的——全文文章为XML格式。
- **Crossref和Unpaywall**受益于包含`mailto`参数或电子邮件地址，以便进入礼貌/快速池。

### 请求指南

- 对于**NCBI API**（PubMed、PMC）：无密钥时最多每秒3个请求，有密钥时每秒10个。按顺序发起请求。
- 对于**arXiv**：每3秒最多1个请求。请耐心等待。
- 对于**Crossref**：每秒5个请求（公共池），每秒10个请求（礼貌池，带`mailto`）。
- 对于其他没有严格限制的API，可以并行查询多个数据库。
- 如果收到HTTP 429（速率限制），稍等片刻并重试一次。

### 错误恢复

1. **检查标识符格式** -- 使用常见标识符格式表。PMID在arXiv中无效，arXiv ID在PubMed中无法直接使用。
2. **尝试其他标识符** -- 如果DOI在某数据库中失败，尝试使用标题或PMID。
3. **尝试另一个数据库** -- 如果PubMed对一篇CS论文没有返回结果，尝试Semantic Scholar或OpenAlex。
4. **报告失败** -- 告知用户哪个数据库失败、错误信息以及你尝试了哪些替代方案。

## 输出格式

按如下结构组织你的响应：

```
## 查询的数据库
- **PubMed** -- esearch + esummary，查询“CRISPR gene therapy”
- **Unpaywall** -- DOI查找，10.1038/...

## 结果

### PubMed
[原始JSON响应或格式化结果]

### Unpaywall
[原始JSON响应]
```

如果结果非常庞大，展示最相关的部分并注明有更多数据可用。但默认情况下应展示完整的原始JSON——用户正是需要它。

## 可用的数据库

在调用任何API之前，先阅读相关的参考文件。

### 生物医学文献
| 数据库 | 参考文件 | 覆盖内容 |
|---|---|---|
| PubMed | `references/pubmed.md` | 3700万+生物医学引文、摘要、MeSH术语 |
| PMC | `references/pmc.md` | 1000万+全文生物医学文章（JATS XML）、ID转换 |

### 预印本服务器
| 数据库 | 参考文件 | 覆盖内容 |
|---|---|---|
| bioRxiv | `references/biorxiv.md` | 生物学预印本（按日期/DOI浏览，无关键词搜索） |
| medRxiv | `references/medrxiv.md` | 健康科学预印本（按日期/DOI浏览，无关键词搜索） |
| arXiv | `references/arxiv.md` | 物理、数学、计算机科学、生物学、经济学预印本（关键词搜索，Atom XML） |

### 多学科索引
| 数据库 | 参考文件 | 覆盖内容 |
|---|---|---|
| OpenAlex | `references/openalex.md` | 2.5亿+作品、作者、机构、主题、引用数据 |
| Crossref | `references/crossref.md` | 1.5亿+DOI元数据、期刊、资助方、参考文献 |
| Semantic Scholar | `references/semantic-scholar.md` | 2亿+论文、引用图、AI生成的TLDR、推荐 |

### 开放获取与全文
| 数据库 | 参考文件 | 覆盖内容 |
|---|---|---|
| CORE | `references/core.md` | 3700万+来自全球OA存储库的全文 |
| Unpaywall | `references/unpaywall.md` | 任意DOI的开放获取状态和PDF链接 |
