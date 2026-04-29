# 网络搜索

搜索网络：$ARGUMENTS

## 命令

根据查询内容选择一个简短、描述性的文件名（例如 `ai-chip-news`、`react-vs-vue`）。使用小写字母和连字符，不要有空格。

```bash
parallel-cli search "$ARGUMENTS" -q "<关键词1>" -q "<关键词2>" --json --max-results 10 --excerpt-max-chars-total 27000 -o "$FILENAME.json"
```

第一个参数是**目标**——即你正在寻找的内容的自然语言描述。它用一个调用代替多个关键词搜索，适用于宽泛或复杂的查询。添加 `-q` 标志用于特定关键词查询，以补充目标。`-o` 标志将完整结果保存到 JSON 文件中，以便后续提问。

可选选项：
- `--after-date YYYY-MM-DD` 用于时效性查询
- `--include-domains domain1.com,domain2.com` 限制特定来源

## 学术源策略

对于科学或技术查询，进行**两次搜索**，以确保学术来源与一般结果同时出现：

1. **面向学术的搜索**——附加 `--include-domains` 并指定学术域名：
   ```bash
   parallel-cli search "$ARGUMENTS" -q "<关键词1>" --json --max-results 10 --excerpt-max-chars-total 27000 --include-domains "scholar.google.com,arxiv.org,pubmed.ncbi.nlm.nih.gov,semanticscholar.org,biorxiv.org,medrxiv.org,ncbi.nlm.nih.gov,nature.com,science.org,ieee.org,acm.org,springer.com,wiley.com,cell.com,pnas.org,nih.gov" -o "$FILENAME-academic.json"
   ```

2. **一般搜索**——标准的无域名限制命令，用于捕获相关的非学术来源。

合并结果，以学术来源优先。如果只有一次搜索是实际的（例如明显非科学查询），则跳过面向学术的搜索。

**何时使用两次搜索模式：** 任何涉及科学主张、医疗信息、研究发现、技术机制、统计数据，或任何一手文献比二手报道更可靠的内容。

## 解析结果

不要在命令执行中设置 `max_output_tokens`——输出已由 `--max-results` 和 `--excerpt-max-chars-total` 限制。限制输出 token 会截断 JSON 并破坏解析。

从标准输出解析 JSON。对每个结果，提取：
- 标题、URL、发布日期
- 摘录中的有用内容（跳过导航噪音，如菜单、页脚、“跳到内容”）

## 响应格式

**关键：每个声明都必须有内联引用。** 使用仅来自 JSON 输出的 Markdown 链接。切勿编造或猜测 URL。

对于学术来源，在有元数据的情况下使用作者-年份引用风格：
- 学术：[Smith et al., 2025](url) 或 [Smith & Jones, 2024](url)
- 非学术：[来源标题](url)

综合响应，要求：
- 尽可能以同行评审或预印本来源的发现开头
- 清晰区分由一手研究支持的声明与次要报告
- 包含具体事实、名称、数字、日期
- 内联引用每个事实——不要留下任何未引用的声明
- 如果涉及多个主题，按主题组织
- 注明证据质量（例如“一项随机对照试验发现...” vs “一篇博客文章报道...”）

**以“来源”部分结尾**，列出每个引用的 URL，按类型分组：

```
来源：

学术/同行评审：
- [Smith et al., 2025 — 论文标题](https://doi.org/...) (Nature, 2025)
- [Jones & Lee, 2024 — 论文标题](https://arxiv.org/...) (arXiv 预印本)

其他：
- [来源标题](https://example.com/article) (2026年2月)
```

此“来源”部分为必选项，不可省略。如果未找到学术来源，需注明并解释原因（例如主题太新、尚未研究或本质上非学术）。

在“来源”部分之后，提及输出文件路径（`$FILENAME.json`），以便用户知道该文件可用于后续提问。
