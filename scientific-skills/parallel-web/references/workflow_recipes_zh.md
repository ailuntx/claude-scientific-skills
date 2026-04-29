# 工作流配方

结合Parallel的搜索、提取和深度研究API，用于科学写作任务的常见多步骤模式。

---

## 配方索引

| 配方 | 使用的API | 时间 | 使用场景 |
|--------|-----------|------|----------|
| [章节研究流程](#recipe-1-section-research-pipeline) | 研究 + 搜索 | 2-5分钟 | 撰写论文章节 |
| [文献引用验证](#recipe-2-citation-verification) | 搜索 + 提取 | 1-2分钟 | 验证论文元数据 |
| [文献综述](#recipe-3-literature-survey) | 研究 + 搜索 + 提取 | 5-15分钟 | 全面文献回顾 |
| [市场情报报告](#recipe-4-market-intelligence-report) | 研究（多阶段） | 10-30分钟 | 市场/行业分析 |
| [竞争分析](#recipe-5-competitive-analysis) | 搜索 + 提取 + 研究 | 5-10分钟 | 公司/产品对比 |
| [事实核查流程](#recipe-6-fact-check-pipeline) | 搜索 + 提取 | 1-3分钟 | 验证声明真实性 |
| [时事简报](#recipe-7-current-events-briefing) | 搜索 + 研究 | 3-5分钟 | 新闻综合 |
| [技术文档收集](#recipe-8-technical-documentation-gathering) | 搜索 + 提取 | 2-5分钟 | API/框架文档 |
| [基金背景研究](#recipe-9-grant-background-research) | 研究 + 搜索 | 5-10分钟 | 基金申请背景 |

---

## 配方1：章节研究流程

**目标：** 为科学论文的单个章节收集研究资料和引用文献。

**API：** 深度研究（pro-fast）+ 搜索

```bash
# 步骤1：深度研究获取全面背景
python scripts/parallel_web.py research \
  "医疗AI中联邦学习的最新进展（2023-2025），聚焦隐私保护训练方法、实际部署和监管考量" \
  --processor pro-fast -o sources/section_background.md

# 步骤2：针对性搜索特定引用文献
python scripts/parallel_web.py search \
  "查找医院环境中联邦学习的同行评审论文" \
  --queries "联邦学习临床部署" "隐私保护机器学习医疗" \
  --max-results 10 -o sources/section_citations.txt
```

**Python版本：**
```python
from parallel_web import ParallelDeepResearch, ParallelSearch

researcher = ParallelDeepResearch()
searcher = ParallelSearch()

# 步骤1：深度背景研究
background = researcher.research(
    query="医疗AI中联邦学习的最新进展（2023-2025）："
          "隐私保护方法、实际部署、监管环境",
    processor="pro-fast",
    description="按结构组织：(1) 关键方法 (2) 临床部署 "
                "(3) 监管考量 (4) 开放挑战。包含统计数据"
)

# 步骤2：查找待引用的具体论文
papers = searcher.search(
    objective="查找医院环境中部署联邦学习的最新同行评审论文",
    search_queries=[
        "联邦学习医院临床研究 2024",
        "隐私保护机器学习医疗部署"
    ],
    source_policy={"allow_domains": ["nature.com", "thelancet.com", "arxiv.org", "pubmed.ncbi.nlm.nih.gov"]},
)

# 整合：使用背景资料写作，论文用于引用
```

**使用场景：** 在撰写研究论文、文献综述或基金申请的主要章节前使用。

---

## 配方2：文献引用验证

**目标：** 验证引用文献的真实性并获取完整元数据（DOI、卷号、页码、年份）。

**API：** 搜索 + 提取

```bash
# 方案A：搜索论文
python scripts/parallel_web.py search \
  "Vaswani等2017年NeurIPS论文《Attention is All You Need》" \
  --queries "Attention is All You Need DOI" --max-results 5

# 方案B：从DOI提取元数据
python scripts/parallel_web.py extract \
  "https://doi.org/10.48550/arXiv.1706.03762" \
  --objective "完整引用信息：作者、标题、出处、年份、页码、DOI"
```

**Python版本：**
```python
from parallel_web import ParallelSearch, ParallelExtract

searcher = ParallelSearch()
extractor = ParallelExtract()

# 步骤1：查找论文
result = searcher.search(
    objective="查找Vaswani等《Attention Is All You Need》论文的完整引用详情",
    search_queries=["Attention is All You Need Vaswani 2017 NeurIPS DOI"],
    max_results=5,
)

# 步骤2：从论文页面提取完整元数据
paper_url = result["results"][0]["url"]
metadata = extractor.extract(
    urls=[paper_url],
    objective="完整BibTeX引用：所有作者、标题、会议/期刊、年份、页码、DOI、卷号",
)
```

**使用场景：** 完成章节写作后，验证references.bib中每个引用的元数据是否准确完整。

---

## 配方3：文献综述

**目标：** 全面调研研究领域，识别关键论文、主题和空白。

**API：** 深度研究 + 搜索 + 提取

```python
from parallel_web import ParallelDeepResearch, ParallelSearch, ParallelExtract

researcher = ParallelDeepResearch()
searcher = ParallelSearch()
extractor = ParallelExtract()

topic = "基于CRISPR的传染病诊断技术"

# 阶段1：广泛研究概览
overview = researcher.research(
    query=f"{topic}全面综述：关键进展、临床应用、"
          f"监管现状、商业产品和未来方向（2020-2025）",
    processor="ultra-fast",
    description="按文献综述结构组织：(1) 历史发展 "
                "(2) 当前技术 (3) 临床应用 "
                "(4) 监管环境 (5) 商业产品 "
                "(6) 局限性与未来方向。包含关键统计数据和里程碑"
)

# 阶段2：查找具体里程碑论文
key_papers = searcher.search(
    objective=f"查找Nature、Science、Cell、NEJM中关于{topic}的高被引影响力论文",
    search_queries=[
        "CRISPR诊断 SHERLOCK DETECTR Nature",
        "CRISPR即时检测临床研究",
        "核酸检测CRISPR综述"
    ],
    source_policy={
        "allow_domains": ["nature.com", "science.org", "cell.com", "nejm.org", "thelancet.com"],
    },
    max_results=15,
)

# 阶段3：提取前5篇论文的详细内容
top_urls = [r["url"] for r in key_papers["results"][:5]]
detailed = extractor.extract(
    urls=top_urls,
    objective="研究设计、关键结果、灵敏度/特异性数据和临床意义",
)
```

**使用场景：** 启动文献综述、系统评价或全面背景章节时使用。

---

## 配方4：市场情报报告

**目标：** 生成关于行业或产品类别的全面市场研究报告。

**API：** 深度研究（多阶段）

```python
researcher = ParallelDeepResearch()

industry = "AI驱动的药物发现"

# 阶段1：市场概览（ultra-fast获取最大深度）
market_overview = researcher.research(
    query=f"{industry}全面市场分析：市场规模、增长率、"
          f"关键细分领域、地域分布及2030年前预测",
    processor="ultra-fast",
    description="包含具体金额数据、复合年增长率百分比和数据来源。"
                "按细分领域和地域分解"
)

# 阶段2：竞争格局
competitors = researcher.research_structured(
    query=f"{industry}领域前10公司：收入、融资、核心产品、合作伙伴和市场地位",
    processor="pro-fast",
)

# 阶段3：技术与创新趋势
tech_trends = researcher.research(
    query=f"{industry}技术趋势与创新格局："
          f"新兴方法、突破性技术、专利格局和研发投入",
    processor="pro-fast",
    description="聚焦具体技术，量化研发支出，识别新兴领导者"
)

# 阶段4：监管与风险分析
regulatory = researcher.research(
    query=f"{industry}监管环境与风险因素："
          f"FDA指南、EMA要求、合规挑战和市场风险",
    processor="pro-fast",
)
```

**使用场景：** 创建市场研究报告、投资者演示文稿或战略分析文档。

---

## 配方5：竞争分析

**目标：** 并排比较多家公司、产品或技术。

**API：** 搜索 + 提取 + 研究

```python
searcher = ParallelSearch()
extractor = ParallelExtract()
researcher = ParallelDeepResearch()

companies = ["OpenAI", "Anthropic", "Google DeepMind"]

# 步骤1：搜索各公司最新数据
for company in companies:
    result = searcher.search(
        objective=f"{company}2025年最新产品发布、融资、团队规模和战略",
        search_queries=[f"{company}产品发布2025", f"{company}融资估值"],
        source_policy={"after_date": "2024-06-01"},
    )

# 步骤2：从公司页面提取信息
company_pages = [
    "https://openai.com/about",
    "https://anthropic.com/company",
    "https://deepmind.google/about/",
]
company_data = extractor.extract(
    urls=company_pages,
    objective="使命、核心产品、团队规模、创立日期和近期里程碑",
)

# 步骤3：深度研究进行综合
comparison = researcher.research(
    query=f"{'、'.join(companies)}详细对比："
          f"产品、定价、技术路线、市场地位、优势劣势",
    processor="pro-fast",
    description="创建结构化对比，涵盖："
                "(1) 产品组合 (2) 技术路线 (3) 定价策略 "
                "(4) 市场地位 (5) 优势/劣势 (6) 未来展望。"
                "包含总结对比表"
)
```

---

## 配方6：事实核查流程

**目标：** 在文档中引用前验证具体声明或统计数据。

**API：** 搜索 + 提取

```python
searcher = ParallelSearch()
extractor = ParallelExtract()

claim = "全球AI市场预计2030年将达到1.8万亿美元"

# 步骤1：搜索佐证来源
result = searcher.search(
    objective=f"验证该声明：'{claim}'。查找证实或反驳该数据的权威来源",
    search_queries=["全球AI市场规模2030预测", "人工智能市场预测万亿美元"],
    max_results=8,
)

# 步骤2：从顶级来源提取具体数据
source_urls = [r["url"] for r in result["results"][:3]]
details = extractor.extract(
    urls=source_urls,
    objective="具体市场规模数据、预测年份、复合年增长率及预测方法",
)

# 分析：多个权威来源是否一致？
```

**使用场景：** 在论文或报告中引用具体统计数据、市场数据或事实声明前使用。

---

## 配方7：时事简报

**目标：** 获取某主题近期发展的最新综合摘要。

**API：** 搜索 + 研究

```python
searcher = ParallelSearch()
researcher = ParallelDeepResearch()

topic = "欧盟AI法案实施"

# 步骤1：查找最新新闻
latest = searcher.search(
    objective=f"过去一个月关于{topic}的最新新闻与进展",
    search_queries=[f"{topic} 2025", f"{topic} 最新动态"],
    source_policy={"after_date": "2025-01-15"},
    max_results=15,
)

# 步骤2：综合生成简报
briefing = researcher.research(
    query=f"截至2025年2月{topic}的最新进展摘要："
          f"关键里程碑、合规期限、行业反应和影响",
    processor="pro-fast",
    description="撰写500字行政简报，包含关键事件时间线"
)
```

---

## 配方8：技术文档收集

**目标：** 收集并综合框架或API的技术文档。

**API：** 搜索 + 提取

```python
searcher = ParallelSearch()
extractor = ParallelExtract()

# 步骤1：查找文档页面
docs = searcher.search(
    objective="查找PyTorch实现自定义注意力机制的官方文档",
    search_queries=["PyTorch注意力机制教程", "PyTorch MultiheadAttention文档"],
    source_policy={"allow_domains": ["pytorch.org", "github.com/pytorch"]},
)

# 步骤2：从文档页面提取完整内容
doc_urls = [r["url"] for r in docs["results"][:3]]
full_docs = extractor.extract(
    urls=doc_urls,
    objective="完整API参考、参数、用法示例和代码片段",
    full_content=True,
)
```

---

## 配方9：基金背景研究

**目标：** 为基金申请书构建包含验证数据的全面背景章节。

**API：** 深度研究 + 搜索

```python
researcher = ParallelDeepResearch()
searcher = ParallelSearch()

research_area = "AI引导的抗生素发现以应对抗菌素耐药性"

# 步骤1：疾病负担与意义
significance = researcher.research(
    query=f"抗菌素耐药性负担：死亡率统计、经济影响、"
          f"WHO重点病原体和预测。包含具体数字",
    processor="pro-fast",
    description="聚焦适用于NIH意义部分的统计数据："
                "年死亡人数、经济成本、耐药趋势和紧迫性"
)

# 步骤2：创新格局
innovation = researcher.research(
    query=f"{research_area}的当前方法：成功案例（halicin等）、"
          f"现有方法的局限性和我们方法的新颖性",
    processor="pro-fast",
    description="聚焦创新部分：已尝试方法、现存空白和新兴方法"
)

# 步骤3：查找初步数据背景的特定论文
papers = searcher.search(
    objective="查找AI发现抗生素和药物发现ML方法的里程碑论文",
    search_queries=[
        "halicin AI抗生素发现 Nature",
        "机器学习抗生素耐药性预测",
        "深度学习药物发现抗生素"
    ],
    source_policy={"allow_domains": ["nature.com", "science.org", "cell.com", "pnas.org"]},
)
```

**使用场景：** 撰写NIH、NSF等基金申请的意义、创新或背景章节时使用。

---

## 与其他技能结合

### 结合`research-lookup`（学术论文）

```python
# 使用parallel-web进行常规研究
researcher.research("量子计算应用现状")

# 使用research-lookup搜索学术论文（自动路由至Perplexity）
# python research_lookup.py "查找Nature和Science中量子纠错相关论文"
```

### 结合`citation-management`（BibTeX）

```python
# 步骤1：通过parallel搜索查找论文
result = searcher.search(objective="Vaswani等Attention Is All You Need论文")

# 步骤2：从结果获取DOI
doi = "10.48550/arXiv.1706.03762"

# 步骤3：使用citation-management技能转为BibTeX
# python scripts/doi_to_bibtex.py 10.48550/arXiv.1706.03762
```

### 结合`scientific-schematics`（图表生成）

```python
# 步骤1：研究某个流程
result = researcher.research("CRISPR-Cas9基因编辑机制的分步工作原理")

# 步骤2：利用研究生成示意图
# python scripts/generate_schematic.py "CRISPR-Cas9基因编辑流程：向导RNA设计→Cas9结合→DNA切割→修复路径" -o figures/crispr_mechanism.png
```

---

## 性能速查表

| 任务 | 处理器 | 预计时间 | 近似成本 |
|------|-----------|---------------|------------------|
| 快速事实查询 | `base-fast` | 15-50秒 | $0.01 |
| 章节背景研究 | `pro-fast` | 30秒-5分钟 | $0.10 |
| 综合报告 | `ultra-fast` | 1-10分钟 | $0.30 |
| 网络搜索（10条结果） | 搜索API | 1-3秒 | $0.005

| URL提取（5个URL） | 提取API | 5-30秒 | 0.005美元 |

---

## 另请参阅

- [API参考](api_reference.md) - 完整的API参数参考
- [搜索最佳实践](search_best_practices.md) - 高效搜索查询
- [深度研究指南](deep_research_guide.md) - 处理器选择与输出格式
- [提取模式](extraction_patterns.md) - URL内容提取
