---
name: research-lookup
description: 使用 parallel-cli search（主要、快速网络搜索）、Parallel Chat API（深度研究）或 Perplexity sonar-pro-search（学术论文搜索）查找当前研究信息。自动将查询路由到最佳后端。用于查找论文、收集研究数据及验证科学信息。
allowed-tools: Read Write Edit Bash
license: MIT license
compatibility: 需要 parallel-cli（主要）；可选 PARALLEL_API_KEY 和 OPENROUTER_API_KEY 用于深度/学术后端
metadata:
    skill-author: K-Dense Inc.
---

# 研究信息查询

## 概述

此技能提供实时研究信息查询，并具备**智能后端路由**功能：

- **parallel-cli search**（parallel-web 技能）：所有研究查询的**主要和默认后端**。快速、经济的网络搜索，优先学术来源。使用 `parallel-cli search` 配合 `--include-domains` 参数指定学术来源。
- **Parallel Chat API**（`core` 模型）：用于复杂、多源深度研究的次要后端，需要扩展综合（延迟 60 秒至 5 分钟）。仅在明确需要时使用。
- **Perplexity sonar-pro-search**（通过 OpenRouter）：仅在学术论文搜索中需要访问学术数据库时使用。

该技能自动检测查询类型并路由到最优后端。

## 何时使用此技能

在需要以下内容时使用此技能：

- **当前研究信息**：最新的研究、论文和发现
- **文献验证**：根据当前研究核实事实、统计数据或声明
- **背景调研**：为科学写作收集背景和支持证据
- **引用来源**：查找可供引用的相关论文和研究
- **技术文档**：查找规格、协议或方法
- **市场/行业数据**：当前统计数据、趋势、竞争情报
- **最新进展**：新兴趋势、突破性成果、公告

## 使用科学示意图进行可视化增强

**使用此技能创建文档时，始终考虑添加科学图表和示意图以增强视觉传达效果。**

如果您的文档尚未包含示意图或图表：
- 使用 **scientific-schematics** 技能生成 AI 驱动的、符合出版质量的图表
- 只需用自然语言描述您想要的图表

```bash
python scripts/generate_schematic.py "您的图表描述" -o figures/output.png
```

---

## 自动后端选择

该技能根据内容自动将查询路由到最佳后端：

### 路由逻辑

```
查询到达
    |
    +-- 包含学术关键词？（论文、DOI、期刊、同行评审等）
    |       是 --> Perplexity sonar-pro-search（学术搜索模式）
    |
    +-- 需要深度多源综合？（用户说“深度研究”、“详尽无遗”）
    |       是 --> Parallel Chat API（core 模型，60 秒至 5 分钟）
    |
    +-- 其他所有情况（一般研究、市场数据、技术信息、分析）
            --> parallel-cli search（快速、默认）
```

### 默认：parallel-cli search（parallel-web 技能）

**所有标准研究查询的主要后端。** 快速、经济，并支持学术来源优先。

对于科学/技术查询，运行两次搜索以确保学术覆盖：

```bash
# 1. 学术聚焦搜索
parallel-cli search "您的研究查询" -q "关键词1" -q "关键词2" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  --include-domains "scholar.google.com,arxiv.org,pubmed.ncbi.nlm.nih.gov,semanticscholar.org,biorxiv.org,medrxiv.org,ncbi.nlm.nih.gov,nature.com,science.org,ieee.org,acm.org,springer.com,wiley.com,cell.com,pnas.org,nih.gov" \
  -o sources/research_<topic>-academic.json

# 2. 通用搜索（捕获非学术来源）
parallel-cli search "您的研究查询" -q "关键词1" -q "关键词2" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  -o sources/research_<topic>-general.json
```

选项：
- `--after-date YYYY-MM-DD` 用于时间敏感的查询
- `--include-domains domain1.com,domain2.com` 限制到特定来源

合并结果，优先展示学术来源。对于非科学查询，一次通用搜索即可。

默认情况下，所有其他查询都路由至此，包括：

- 一般研究问题
- 市场和行业分析
- 技术信息和文档
- 当前事件和最新发展
- 比较分析
- 统计数据检索
- 事实核查与验证

### 学术关键词（路由到 Perplexity）

包含以下术语的查询将路由到 Perplexity 进行学术聚焦搜索：

- 论文查找：`查找论文`、`查找文章`、`关于...的研究论文`、`已发表的研究`
- 引用：`引用`、`引文`、`doi`、`pubmed`、`pmid`
- 学术来源：`同行评审`、`期刊文章`、`学术`、`arxiv`、`预印本`
- 综述类型：`系统综述`、`荟萃分析`、`文献检索`
- 论文质量：`基础性论文`、`开创性论文`、`里程碑式论文`、`高被引`

### 深度研究（路由到 Parallel Chat API）

仅在用户明确要求深度、详尽或全面研究时使用。比 parallel-cli search 慢得多且更昂贵。

### 手动覆盖

您可以强制使用特定后端：

```bash
# 强制使用 parallel-cli search（快速网络搜索）
parallel-cli search "您的查询" -q "关键词" --json --max-results 10 -o sources/research_<topic>.json

# 强制使用 Parallel 深度研究（慢速、详尽）
python research_lookup.py "您的查询" --force-backend parallel

# 强制使用 Perplexity 学术搜索
python research_lookup.py "您的查询" --force-backend perplexity
```

---

## 核心能力

### 1. 一般研究查询（parallel-cli search — 默认）

**主要后端。** 快速、经济的网络搜索，通过 parallel-web 技能优先学术来源。

```
查询示例：
- "CRISPR 基因编辑的最新进展 2025"
- "比较 mRNA 疫苗与传统疫苗在癌症治疗中的应用"
- "医疗行业中 AI 采用的统计数据"
- "全球可再生能源市场趋势与预测"
- "解释肠道微生物组与抑郁症之间关联的机制"
```

```bash
# 示例：关于 CRISPR 进展的研究
parallel-cli search "CRISPR 基因编辑的最新进展 2025" \
  -q "CRISPR" -q "gene editing" -q "2025" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  --include-domains "scholar.google.com,arxiv.org,pubmed.ncbi.nlm.nih.gov,nature.com,science.org,cell.com,pnas.org,nih.gov" \
  -o sources/research_crispr_advances-academic.json

parallel-cli search "CRISPR 基因编辑的最新进展 2025" \
  -q "CRISPR" -q "gene editing" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  -o sources/research_crispr_advances-general.json
```

**响应包括：**
- 综合发现，内嵌来自搜索结果的引用
- 优先显示学术来源（同行评审、预印本）
- 具体事实、数字和日期
- 来源部分，按类型列出所有引用的 URL

### 2. 学术论文搜索（Perplexity sonar-pro-search）

**用于学术特定查询。** 优先考虑学术数据库和同行评审来源。当查询明确要求论文、引用或 DOI 时使用。

```
查询示例：
- "在 NeurIPS 2024 中查找关于 Transformer 注意力机制的论文"
- "关于量子纠错的基础性论文"
- "非小细胞肺癌免疫治疗的系统综述"
- "引用原始 BERT 论文及其最具影响力的后续工作"
- "关于 CRISPR 临床试验中脱靶效应的已发表研究"
```

**响应包括：**
- 学术文献关键发现总结
- 5-8 个高质量引用，包含作者、标题、期刊、年份、DOI
- 引用次数和期刊等级指标
- 关键统计数据和方**法亮点
- 研究空白与未来方向

### 3. 深度研究（Parallel Chat API — 仅按请求提供）

**仅在用户明确要求深度/详尽研究时使用。** 通过 Chat API（`core` 模型）提供全面的多源综合。延迟 60 秒至 5 分钟。

```
查询示例：
- "深度研究：量子计算纠错的当前状态"
- "详尽分析：用于癌症免疫治疗的 mRNA 疫苗平台"
```

### 4. 技术与方法信息

使用 parallel-cli search（默认）进行快速查找：

```bash
parallel-cli search "蛋白质检测的 Western blot 方案" \
  -q "western blot" -q "protocol" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  -o sources/research_western_blot.json
```

### 5. 统计与市场数据

使用 parallel-cli search（默认）获取当前数据：

```bash
parallel-cli search "全球 AI 市场规模与增长预测 2025" \
  -q "AI market" -q "statistics" -q "growth" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  --after-date 2024-01-01 \
  -o sources/research_ai_market.json
```

---

## 论文质量与流行度优先级

**关键**：在搜索论文时，始终优先选择高质量、有影响力的论文。

### 基于引用次数的排名

| 论文年龄 | 引用阈值 | 分类 |
|----------|----------|------|
| 0-3 年 | 20+ 次引用 | 值得关注 |
| 0-3 年 | 100+ 次引用 | 高度影响 |
| 3-7 年 | 100+ 次引用 | 重要 |
| 3-7 年 | 500+ 次引用 | 里程碑论文 |
| 7+ 年 | 500+ 次引用 | 开创性工作 |
| 7+ 年 | 1000+ 次引用 | 基础性论文 |

### 期刊/会议质量等级

**第一等级 - 顶级期刊**（始终首选）：
- **综合科学**：Nature, Science, Cell, PNAS
- **医学**：NEJM, Lancet, JAMA, BMJ
- **领域特定**：Nature Medicine, Nature Biotechnology, Nature Methods
- **顶级计算机科学/AI**：NeurIPS, ICML, ICLR, ACL, CVPR

**第二等级 - 高影响专业期刊**（强烈偏好）：
- 影响因子 > 10 的期刊
- 子领域顶级会议（EMNLP, NAACL, ECCV, MICCAI）

**第三等级 - 受尊重的专业期刊**（相关时包含）：
- 影响因子 5-10 的期刊

---

## 技术集成

### 前置条件

```bash
# 主要后端（parallel-cli）— 必需
# 如果尚未安装，请安装 parallel-cli：
curl -fsSL https://parallel.ai/install.sh | bash
# 或：uv tool install "parallel-web-tools[cli]"

# 认证：
parallel-cli auth
# 或：export PARALLEL_API_KEY="您的_parallel_api_key"
```

### 环境变量

```bash
# 主要后端（parallel-cli search）— 必需
export PARALLEL_API_KEY="您的_parallel_api_key"

# 深度研究后端（Parallel Chat API）— 可选，仅用于深度研究
# 使用相同的 PARALLEL_API_KEY

# 学术搜索后端（Perplexity）— 可选，用于学术论文查询
export OPENROUTER_API_KEY="您的_openrouter_api_key"
```

### API 规格

**parallel-cli search（主要）：**
- 命令：`parallel-cli search` 配合 `--json` 输出
- 延迟：2-10 秒（快速）
- 输出：包含标题、URL、发布日期、摘录的 JSON
- 学术域名：使用 `--include-domains` 限定学术来源
- 保存结果：`-o filename.json` 用于后续追踪和可重复性

**Parallel Chat API（仅深度研究）：**
- 端点：`https://api.parallel.ai`（兼容 OpenAI SDK）
- 模型：`core`（延迟 60 秒至 5 分钟，复杂多源综合）
- 输出：带内联引用的 Markdown 文本
- 引用：研究依据，含 URL、推理过程及置信度
- 速率限制：300 req/min
- Python 包：`openai`

**Perplexity sonar-pro-search（仅学术）：**
- 模型：`perplexity/sonar-pro-search`（通过 OpenRouter）
- 搜索模式：学术（优先同行评审来源）
- 搜索上下文：高（全面研究）
- 响应时间：5-15 秒

### 命令行使用

```bash
# 通过 parallel-cli 进行快速网络搜索（默认—推荐）— 始终保存到 sources/ 目录
parallel-cli search "您的查询" -q "关键词1" -q "关键词2" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  -o sources/research_<topic>.json

# 通过 parallel-cli 进行学术聚焦搜索 — 始终保存到 sources/
parallel-cli search "您的查询" -q "关键词1" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  --include-domains "scholar.google.com,arxiv.org,pubmed.ncbi.nlm.nih.gov,semanticscholar.org,biorxiv.org,medrxiv.org,nature.com,science.org,cell.com,pnas.org,nih.gov" \
  -o sources/research_<topic>-academic.json

# 通过 parallel-cli 进行时间敏感搜索
parallel-cli search "您的查询" -q "关键词" \
  --json --max-results 10 --after-date 2024-01-01 \
  -o sources/research_<topic>.json

# 提取特定 URL 的完整内容（使用 parallel-web extract）
parallel-cli extract "https://example.com/paper" --json

# 强制使用 Parallel 深度研究（慢速、详尽）— 通过 research_lookup.py
python research_lookup.py "您的查询" --force-backend parallel -o sources/research_<topic>.md

# 强制使用 Perplexity 学术搜索 — 通过 research_lookup.py
python research_lookup.py "您的查询" --force-backend perplexity -o sources/papers_<topic>.md

# 通过 research_lookup.py 自动路由（旧版）— 始终保存到 sources/
python research_lookup.py "您的查询" -o sources/research_YYYYMMDD_HHMMSS_<topic>.md

# 通过 research_lookup.py 批量查询 — 始终保存到 sources/
python research_lookup.py --batch "查询1" "查询2" "查询3" -o sources/batch_research_<topic>.md
```

---

## 强制要求：将所有结果保存到 Sources 文件夹

**每个 research-lookup 结果必须保存到项目的 `sources/` 文件夹中。**

这是不可妥协的。研究成果获取成本高，且对可重复性至关重要。

### 保存规则

| 后端 | `-o` 标志目标 | 文件名模式 |
|------|--------------|------------|
| parallel-cli search（默认） | `sources/research_<topic>.json` | `research_<简要主题>.json` 或 `research_<简要主题>-academic.json` |
| Parallel 深度研究 | `sources/research_<topic>.md` | `research_YYYYMMDD_HHMMSS_<简要主题>.md` |
| Perplexity（学术） | `sources/papers_<topic>.md` | `papers_YYYYMMDD_HHMMSS_<简要主题>.md` |
| 批量查询 | `sources/batch_<topic>.md` | `batch_research_YYYYMMDD_HHMMSS_<简要主题>.md` |

### 如何保存

**关键：每次搜索必须使用 `-o` 标志将结果保存到 `sources/` 文件夹中。**

**关键：保存的文件必须保留所有引用、源 URL 和 DOI。**

```bash
# parallel-cli search（默认）— 将 JSON 保存到 sources/
parallel-cli search "CRISPR 基因编辑的最新进展 2025" \
  -q "CRISPR" -q "gene editing" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  --include-domains "scholar.google.com,arxiv.org,pubmed.ncbi.nlm.nih.gov,nature.com,science.org,cell.com,pnas.org,nih.gov" \
  -o sources/research_crispr_advances-academic.json

parallel-cli search "CRISPR 基因编辑的最新进展 2025" \
  -q "CRISPR" -q "gene editing" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  -o sources/research_crispr_advances-general.json

# 通过 Perplexity 进行学术论文搜索 — 保存到 sources/
python research_lookup.py "在 NeurIPS 2024 中查找关于 Transformer 注意力机制的论文" \
  -o sources/papers_20250217_143500_transformer_attention.md

# 通过 Parallel Chat API 进行深度研究 — 保存到 sources/
python research_lookup.py "AI 监管格局" --force-backend parallel \
  -o sources/research_20250217_144000_ai_regulation.md

# 批量查询 — 保存到 sources/
python research_lookup.py --batch "mRNA 疫苗效力" "mRNA 疫苗安全性" \
  -o sources/batch_research_20250217_144500_mrna_vaccines.md
```

### 保存文件中的引用保留

每种输出格式以不同方式保留引用：

| 格式 | 包含的引用 | 何时使用 |
|------|------------|----------|
| parallel-cli JSON（默认） | 完整结果对象：`title`, `url`, `publish_date`, `excerpts` | 标准使用 — 结构化、可解析、快速 |
| 文本（research_lookup.py） | `Sources (N):` 部分，包含 `[标题] (日期) + URL` + `Additional References (N):` 含 DOI 和学术 URL | 深度研究 / Perplexity — 人类可读 |
| JSON（`--json` 通过 research_lookup.py） | 完整引用对象：`url`, `title`, `date`, `snippet`, `doi`, `type` | 需要从深度研究中获取最大引用元数据时 |

**对于 parallel-cli search**，保存的 JSON 文件包含：完整搜索结果，每个结果包含标题、URL、发布日期和内容摘录。
**对于 Parallel Chat API 后端**，保存的文件包含：研究报告 + 来源列表（标题、URL）+ 额外参考文献（DOI、学术 URL）。
**对于 Perplexity 后端**，保存的文件包含：学术摘要 + 来源列表（标题、日期、URL、摘录）+ 额外参考文献（DOI、学术 URL）。

**在以下情况下使用 `--json`：**
- 需要编程式解析引用元数据
- 需要保留完整的 DOI 和 URL 数据以生成 BibTeX
- 需要维护结构化引用对象以进行交叉引用

### 为什么保存所有内容

1. **可重复性**：每个引用和声明都可以追溯到其原始研究来源
2. **上下文窗口恢复**：如果上下文被压缩，可以重新读取保存的结果而无需重新查询
3. **审计轨迹**：`sources/` 文件夹记录了所有研究信息的收集方式
4. **跨部分复用**：多个部分可以引用相同的保存研究，无需重复查询
5. **成本效益**：在进行新 API 调用前，先检查 `sources/` 中是否已有相关结果
6. **同行评审支持**：评审者可以验证每个引用背后的研究

### 在发出新查询之前，先检查 Sources

在调用 `research_lookup.py` 之前，检查是否已存在相关结果：

```bash
ls sources/  # 检查已保存的结果
```

如果之前的查询覆盖了相同主题，则重新读取已保存的文件，而不是进行新的 API 调用。

### 日志记录

保存研究结果时，始终记录：

```
[HH:MM:SS] 已保存：研究查询至 sources/research_20250217_143000_crispr_advances.md（3,800 字，8 条引用）
[HH:MM:SS] 已保存：论文搜索至 sources/papers_20250217_143500_transformer_attention.md（找到 6 篇论文）
```

---

## 与科学写作的集成

此技能通过以下方式增强科学写作：

1. **文献综述支持**：为引言和讨论收集当前研究 — **保存到 `sources/`**
2. **方法验证**：根据当前标准验证实验方案 — **保存到 `sources/`**
3. **结果背景化**：将研究结果与近期类似研究进行比较 — **保存到 `sources/`**
4. **讨论增强**：用最新证据支持论点 — **保存到 `sources/`**
5. **引用管理**：提供格式正确的引用 — **保存到 `sources/`**

## 互补工具

| 任务 | 工具 |
|------|------|
| 通用网络搜索（快速） | `parallel-cli search`（内置于此技能） |
| 学术聚焦网络搜索 | `parallel-cli search --include-domains`（内置于此技能） |
| URL 内容提取 | `parallel-cli extract`（parallel-web 技能） |
| 深度研究（详尽） | `research-lookup` 通过 Parallel Chat API 或 `parallel-web` 深度研究 |
| 学术论文搜索 | `research-lookup`（自动路由到 Perplexity） |
| Google Scholar 搜索 | `citation-management` 技能 |
| PubMed 搜索 | `citation-management` 技能 |
| DOI 转 BibTeX | `citation-management` 技能 |
| 元数据验证 | `parallel-cli extract`（parallel-web 技能） |

---

## 错误处理与限制

**已知限制：**
- parallel-cli search：需要安装并认证 `parallel-cli`
- Parallel Chat API（core 模型）：复杂查询可能需要长达 5 分钟
- Perplexity：信息截止日期，可能无法访问付费内容
- 所有后端：无法访问专有或受限数据库

**回退行为：**
- 若找不到 `parallel-cli`，请使用 `curl -fsSL https://parallel.ai/install.sh | bash` 或 `uv tool install "parallel-web-tools[cli]"` 安装
- 若 parallel-cli search 返回结果不足，回退到 Perplexity 或 Parallel Chat API
- 若所选后端的 API 密钥缺失，尝试其他后端
- 若所有后端均失败，返回结构化错误响应
- 若初始响应不足，改写查询以获得更佳结果

---

## 使用示例

### 示例 1：一般研究（路由到 parallel-cli search）

**查询**："Transformer 注意力机制的最新进展 2025"

**后端**：parallel-cli search（默认、快速）

**命令**：
```bash
parallel-cli search "Transformer 注意力机制的最新进展 2025" \
  -q "transformer" -q "attention" -q "2025" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  --include-domains "arxiv.org,semanticscholar.org,nature.com,science.org,ieee.org,acm.org" \
  -o sources/research_transformer_attention-academic.json

parallel-cli search "Transformer 注意力机制的最新进展 2025" \
  -q "transformer" -q "attention" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  -o sources/research_transformer_attention-general.json
```

**响应**：综合发现，内嵌来自学术和通用来源的引用，涵盖近期论文、关键创新和性能基准。

### 示例 2：学术论文搜索（路由到 Perplexity）

**查询**："查找关于 CRISPR 临床试验中脱靶效应的论文"

**后端**：Perplexity sonar-pro-search（学术模式）

**响应**：包含 5-8 篇高影响力论文的精选列表，含完整引用、DOI、引用次数和期刊等级指标。

### 示例 3：比较分析（路由到 parallel-cli search）

**查询**："比较 mRNA 疫苗与传统疫苗在癌症治疗中的异同"

**后端**：parallel-cli search（默认、快速）

**响应**：基于多个网络来源的综合比较，内嵌引用，结构化分析，证据质量说明。

### 示例 4：市场数据（路由到 parallel-cli search）

**查询**："2025 年全球 AI 在医疗中的采用统计数据"

**后端**：parallel-cli search（默认、快速）

```bash
parallel-cli search "2025 年全球 AI 在医疗中的采用统计数据" \
  -q "AI healthcare" -q "adoption statistics" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  --after-date 2024-01-01 \
  -o sources/research_ai_healthcare_adoption.json
```

**响应**：当前市场数据、采用率、增长预测和区域分析，附有来源引用。

---

## 总结

此技能作为主要研究接口，具备智能三后端路由：

- **parallel-cli search**（默认）：快速、经济的网络搜索，通过 parallel-web 技能优先学术来源
- **Parallel Chat API**（`core` 模型）：深度、详尽的多源综合（仅按明确请求）
- **Perplexity sonar-pro-search**：仅用于学术论文搜索
- **自动路由**：检测查询类型并路由到最优后端
- **手动覆盖**：需要时强制使用任何后端
- **学术优先**：双搜索模式确保科学查询的学术来源浮现
