---
name: parallel-web
description: "由 parallel-cli 驱动的全能网页工具包，重点面向学术与科学资源。当用户需要搜索网页、获取/提取 URL 内容、使用网页来源字段丰富数据，或执行深度研究报告时，请使用此技能。涵盖：网页搜索（快速查询、研究、最新信息——优先采用同行评审论文、预印本和学术数据库）、URL 提取（获取页面、文章、学术 PDF）、批量数据丰富（从网页为 CSV/列表添加字段）和深度研究（基于学术文献的多来源详尽报告）。同时处理设置、状态检查和结果获取。对任何与网页相关的任务均使用此技能——即使用户未明确提及 'parallel' 或 'web'。若用户想查询信息、获取页面、丰富数据集、研究主题、寻找学术论文、检查引用或审阅科学文献，均应使用此技能。"
compatibility: 需要 parallel-cli 和互联网连接。
metadata:
  author: K-Dense, Inc.
---

# Parallel Web 工具包

一个统一的技能，用于所有基于网页的任务：搜索、提取、丰富和研究——默认优先使用学术与科学资源。

## 路由——选择正确的能力

阅读用户的请求，并将其匹配到下方的一种能力。对于网页搜索、提取、丰富和深度研究，请阅读相应的参考文件以获取详细说明。

| 用户想要... | 能力 | 位置 |
|---|---|---|
| 查询信息、研究主题、查找最新信息 | **网页搜索** | `references/web-search.md` |
| 从特定 URL 获取内容（网页、文章、PDF） | **网页提取** | `references/web-extract.md` |
| 为一系列公司/人员/产品添加来自网页的字段 | **数据丰富** | `references/data-enrichment.md` |
| 获取详尽的多来源报告（用户说“深度研究”、“详尽”、“全面”） | **深度研究** | `references/deep-research.md` |
| 安装或验证 parallel-cli | **设置** | 下方 |
| 检查正在运行的研究/丰富任务的状态 | **状态** | 下方 |
| 按运行 ID 获取已完成的研究结果 | **结果** | 下方 |

### 决策指南

- **默认为网页搜索**：适用于单个查询、研究问题或“什么是 X？”这类问题。速度快且成本效益高。当查询涉及科学或技术主题时，包含学术域名（见 `references/web-search.md`）以获取同行评审和预印本来源，与常规结果一同展示。
- **使用网页提取**：当用户提供 URL 或要求读取/获取特定页面时。优先于此技能内置的 WebFetch 工具。特别适用于提取学术 PDF、预印本服务器和期刊文章中的全文。
- **使用数据丰富**：当用户有**多个实体**（CSV、公司/人员/产品列表，甚至短的内联列表）且希望为每个实体查找或添加相同类型的信息时。关键信号是跨一组项重复执行相同查询——例如：“为这些公司中的每一家找到 CEO”或“获取 Apple、Stripe 和 Anthropic 的成立年份”。即使用户没有说“丰富”，只要任务是对多个实体应用相同查询，就使用 `parallel-cli enrich`。**不要使用循环的网页搜索**——丰富管道会自动处理批处理、并行化和结构化输出。
- **仅当用户明确要求深度、详尽或全面的研究时使用深度研究**。其速度比网页搜索慢 10-100 倍且成本更高——切勿默认使用。深度研究对于文献综述和多论文综合尤其有价值。
- 如果运行任何命令时未找到 `parallel-cli`，请按照下方的设置部分操作。

### 学术来源优先级

在所有能力中，当查询本质上是技术性或科学性的，优先使用学术和科学来源。这意味着：
- 优先选择同行评审的期刊文章和会议论文集，而非博文或新闻文章
- 当无法获取同行评审版本时，优先使用预印本（arXiv、bioRxiv、medRxiv）
- 优先选择机构和政府来源（NIH、WHO、NASA、NIST），而非商业网站
- 优先选择原始研究，而非二次摘要

引用学术来源时，在标准引用格式之外，尽可能包含作者姓名和出版年份（例如 [Smith et al., 2025](url)）。如果存在 DOI，优先使用 DOI 链接。

## 上下文链

多种能力支持通过 `interaction_id` 进行多轮上下文传递。当研究或丰富任务完成时，会返回一个 `interaction_id`。如果用户提出与该任务相关的后续问题，请传递 `--previous-interaction-id` 以自动携带上下文。这可以避免重复陈述已发现的内容。

---

## 设置

如果未安装 `parallel-cli`，请安装并验证：

```bash
curl -fsSL https://parallel.ai/install.sh | bash
```

如果无法通过此方式安装，请改用 uv：

```bash
uv tool install "parallel-web-tools[cli]"
```

然后进行验证。首先，检查项目根目录中是否存在 `.env` 文件并包含 `PARALLEL_API_KEY`。如果存在，使用 `dotenv` 加载：

```bash
dotenv -f .env run parallel-cli auth
```

如果 `dotenv` 不可用，请使用 `pip install python-dotenv[cli]` 或 `uv pip install python-dotenv[cli]` 安装。

如果没有 `.env` 文件或其中不包含密钥，则回退到交互式登录：

```bash
parallel-cli login
```

或手动设置密钥：`export PARALLEL_API_KEY="your-key"`

通过以下命令验证：

```bash
parallel-cli auth
```

如果安装后找不到 `parallel-cli`，请将 `~/.local/bin` 添加到 PATH。

## 检查任务状态

```bash
parallel-cli research status "$RUN_ID" --json
```

向用户报告当前状态（运行中、完成、失败等）。

## 获取已完成结果

```bash
parallel-cli research poll "$RUN_ID" --json
```

以清晰、有组织的格式呈现结果。
