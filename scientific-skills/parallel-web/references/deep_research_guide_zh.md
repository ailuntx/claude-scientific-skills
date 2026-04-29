# 深度研究指南

全面指南：使用 Parallel 的任务 API 进行深度研究，包括处理器选择、输出格式、结构化模式与高级模式。

---

## 概述

深度研究将自然语言研究查询转化为全面的情报报告。不同于简单搜索，它通过多步骤网络探索权威来源，并整合带行内引用和置信度的研究结果。

**核心特性：**
- 多步骤、多来源研究
- 自动引用与来源标注
- 结构化或文本输出格式
- 异步处理（30秒至25分钟以上）
- 每条结论附带研究依据与置信度

---

## 处理器选择

选择合适处理器是最关键的决策，直接影响研究深度、速度与成本。

### 决策矩阵

| 场景 | 推荐处理器 | 原因 |
|----------|----------------------|-----|
| 论文章节快速背景调研 | `pro-fast` | 速度快、深度适中、成本低 |
| 全面市场研究报告 | `ultra-fast` | 深度多源整合 |
| 简单事实查询或元数据 | `base-fast` | 快速、低成本 |
| 竞争格局分析 | `pro-fast` | 深度与速度的平衡 |
| 项目申请书背景调研 | `pro-fast` | 全面且及时 |
| 领域前沿技术综述 | `ultra-fast` | 最大化来源覆盖 |
| 写作中快速问题解答 | `core-fast` | 响应时间<2分钟 |
| 突发新闻或近期事件 | `pro` (标准版) | 优先获取最新数据 |
| 大规模数据增强 | `base-fast` | 高性价比批量处理 |

### 处理器层级详解

**`pro-fast`** (默认推荐，适用于多数任务):
- 延迟：30秒至5分钟
- 深度：探索10-20+个网络来源
- 最佳场景：章节级研究、背景收集、对比分析
- 成本：$0.10/查询

**`ultra-fast`** (深度研究专用):
- 延迟：1至10分钟
- 深度：探索20-50+个网络来源，多步推理
- 最佳场景：完整报告、市场分析、复杂多维度问题
- 成本：$0.30/查询

**`core-fast`** (快速交叉验证答案):
- 延迟：15秒至100秒
- 深度：交叉验证5-10个来源
- 最佳场景：中等复杂度问题、验证任务
- 成本：$0.025/查询

**`base-fast`** (简单数据增强):
- 延迟：15至50秒
- 深度：标准网络查询，3-5个来源
- 最佳场景：简单事实查询、元数据增强
- 成本：$0.01/查询

### 标准版 vs 快速版

- **快速处理器** (`-fast`): 速度快2-5倍，数据时效性高，适合交互式使用
- **标准处理器** (无后缀): 数据时效性最高，适合后台任务

**经验法则：** 除非需要最新数据（突发新闻、实时金融数据、即时事件），否则始终使用`-fast`版本。

---

## 输出格式

### 文本模式 (Markdown报告)

生成带行内引用的Markdown报告，适合人工阅读与文档整合。

```python
researcher = ParallelDeepResearch()

result = researcher.research(
    query="mRNA疫苗技术平台及其在COVID-19以外应用的全面分析",
    processor="pro-fast",
    description="重点包含临床试验、获批应用、研发管线进展及核心企业。需包含市场规模数据"
)

# result["output"] 包含完整Markdown报告
# result["citations"] 包含来源URL及摘录
```

**文本模式适用场景：**
- 撰写科学文档（论文、综述、报告）
- 主题背景研究
- 为读者创建摘要
- 需要流畅叙述而非结构化数据时

**通过`description`引导文本输出：**

`description`参数可控制报告内容：

```python
# 聚焦特定方面
result = researcher.research(
    query="电动汽车电池技术格局",
    description="重点包含：(1)固态电池进展 (2)充电速度改进 (3)每kWh成本趋势 (4)关键专利与知识产权。按结构化报告格式分章节呈现"
)

# 控制长度与深度
result = researcher.research(
    query="AI在药物发现中的应用",
    description="提供500字执行摘要，涵盖关键应用、标志性成果、领先企业及市场预测"
)
```

### 自动模式 (结构化JSON)

由处理器自动确定最佳输出结构，返回带字段级引用的结构化JSON。

```python
result = researcher.research_structured(
    query="云计算Top5企业：营收、市场份额、核心产品及近期动态",
    processor="pro-fast",
)

# result["content"] 包含结构化数据(dict)
# result["basis"] 包含字段级引用与置信度
```

**自动模式适用场景：**
- 数据提取与增强
- 特定字段的对比分析
- 需要编程访问独立数据点时
- 数据库或电子表格集成

### 自定义JSON模式

精确定义返回字段：

```python
schema = {
    "type": "object",
    "properties": {
        "market_size_2024": {
            "type": "string",
            "description": "2024年全球市场规模（十亿美元）。需包含来源"
        },
        "growth_rate": {
            "type": "string",
            "description": "2024-2030预测期复合年增长率百分比"
        },
        "top_companies": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {"type": "string", "description": "企业名称"},
                    "market_share": {"type": "string", "description": "近似市场份额百分比"},
                    "revenue": {"type": "string", "description": "最近年度营收"}
                },
                "required": ["name", "market_share", "revenue"]
            },
            "description": "按市场份额排序的Top5企业"
        },
        "key_trends": {
            "type": "array",
            "items": {"type": "string"},
            "description": "驱动增长的3-5个行业趋势"
        }
    },
    "required": ["market_size_2024", "growth_rate", "top_companies", "key_trends"],
    "additionalProperties": False
}

result = researcher.research_structured(
    query="全球网络安全市场分析",
    output_schema=schema,
)
```

---

## 构建高效研究查询

### 查询构建框架

按此结构组织查询：**[主题] + [具体方向] + [范围/时间] + [输出要求]**

**优质查询示例：**
```
"全球锂离子电池回收市场综合分析，包含市场规模、核心企业、政策驱动因素及技术路线。聚焦2023-2025年进展"

"基于近期临床试验数据，对比GLP-1受体激动剂（司美格鲁肽、替尔泊肽、利拉鲁肽）治疗2型糖尿病的疗效、安全性及成本效益"

"2023-2025年医疗AI联邦学习方法综述，涵盖隐私保护技术、实际部署案例、法规合规性及性能基准"
```

**低效查询示例：**
```
"告诉我电池信息"          # 过于模糊
"人工智能"                # 缺乏具体方向
"有什么新消息？"           # 未指定主题
"量子计算有史以来所有内容"  # 范围过广
```

### 效果优化技巧

1. **明确需求**："市场规模"优于"介绍市场"
2. **限定时间范围**："2024-2025"缩小数据范围
3. **命名实体**："司美格鲁肽 vs 替尔泊肽"优于"糖尿病药物"
4. **指定输出要求**："包含统计数据、核心企业及增长预测"
5. **控制在15,000字符内**：简洁查询效果更佳

---

## 研究依据处理

每个深度研究结果均包含**依据**——每条结论的引用来源、推理过程及置信度。

### 文本模式依据

```python
result = researcher.research(query="...", processor="pro-fast")

# 引用经过去重处理，包含URL与摘录
for citation in result["citations"]:
    print(f"来源：{citation['title']}")
    print(f"URL：{citation['url']}")
    if citation.get("excerpts"):
        print(f"摘录：{citation['excerpts'][0][:200]}")
```

### 结构化模式依据

```python
result = researcher.research_structured(query="...", processor="pro-fast")

for basis_entry in result["basis"]:
    print(f"字段：{basis_entry['field']}")
    print(f"置信度：{basis_entry['confidence']}")
    print(f"推理：{basis_entry['reasoning']}")
    for cit in basis_entry["citations"]:
        print(f"  来源：{cit['url']}")
```

### 置信度等级

| 等级 | 含义 | 处理建议 |
|-------|---------|--------|
| `high` | 多个权威来源一致 | 可直接使用 |
| `medium` | 部分证据支持，存在轻微不确定性 | 需附加说明使用 |
| `low` | 证据有限，存在显著不确定性 | 需独立验证 |

---

## 高级模式

### 多阶段研究

分阶段使用不同处理器实现渐进式深度研究：

```python
# 阶段1：base-fast快速概览
overview = researcher.research(
    query="量子纠错主要方法有哪些？",
    processor="base-fast",
)

# 阶段2：对最优方法深度研究
deep_dive = researcher.research(
    query=f"表面码量子纠错技术深度分析：近期突破、实施挑战及领先研究团队。"
          f"背景：{overview['output'][:500]}",
    processor="pro-fast",
)
```

### 对比研究

```python
result = researcher.research(
    query="对比三大主流大语言模型架构：GPT-4、Claude与Gemini。"
          "涵盖架构差异、基准性能、定价策略、上下文窗口及独特能力。需包含具体基准分数",
    processor="pro-fast",
    description="创建结构化对比报告含摘要表格。需包含具体数值与基准测试结果"
)
```

### 研究后内容提取

```python
# 步骤1：研究获取相关来源
research_result = researcher.research(
    query="2024年注意力机制领域最具影响力论文",
    processor="pro-fast",
)

# 步骤2：从核心来源提取全文
from parallel_web import ParallelExtract
extractor = ParallelExtract()

key_urls = [c["url"] for c in research_result["citations"][:5]]
for url in key_urls:
    extracted = extractor.extract(
        urls=[url],
        objective="核心方法论、结果与结论",
    )
```

---

## 性能优化

### 降低延迟

1. **使用`-fast`处理器**：比标准版快2-5倍
2. **中等查询用`core-fast`**：多数问题响应<2分钟
3. **查询需具体**：模糊查询增加探索时间
4. **设置合理超时**：避免过度等待

### 降低成本

1. **从`base-fast`开始**：深度不足时再升级
2. **中等复杂度用`core-fast`**：$0.025 vs $0.10(pro)
3. **批量处理关联查询**：单次精准查询优于多次简单查询
4. **缓存结果**：跨章节复用研究输出

### 提升质量

1. **使用`pro-fast`或`ultra-fast`**：来源越多整合质量越高
2. **提供上下文**："我正在为Nature Medicine撰写关于..."
3. **善用`description`参数**：引导输出结构与重点
4. **关键结论需验证**：通过搜索API或提取功能交叉验证

---

## 常见错误

| 错误 | 影响 | 解决方案 |
|---------|--------|-----|
| 查询过于模糊 | 结果分散不聚焦 | 增加具体方向与时间限定 |
| 查询过长(>15K字符) | API拒绝或结果降级 | 精简上下文，聚焦核心问题 |
| 选错处理器 | 速度过慢或深度不足 | 参考上文决策矩阵 |
| 未使用`description` | 报告结构不符合需求 | 添加描述引导输出 |
| 忽略置信度 | 将低置信数据当作事实 | 引用前检查依据置信度 |
| 未验证引用 | 存在过时或错误归因风险 | 通过提取功能交叉验证关键引用 |

---

## 相关文档

- [API参考](api_reference.md) - 完整API参数说明
- [搜索最佳实践](search_best_practices.md) - 快速网络搜索指南
- [提取模式](extraction_patterns.md) - 特定URL内容读取
- [工作流方案](workflow_recipes.md) - 常见多步骤模式
