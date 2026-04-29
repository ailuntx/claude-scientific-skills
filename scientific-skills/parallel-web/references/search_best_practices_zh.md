# 搜索 API 最佳实践

关于如何从 Parallel 的搜索 API 获得最佳结果的综合指南。

---

## 核心概念

搜索 API 根据自然语言目标返回来自网络资源的、经过 LLM 优化的排名摘要。结果设计为可直接作为模型输入，实现更快的推理和更高质量的生成结果。

### 相对于传统搜索的关键优势

- **面向推理的上下文工程**：结果按推理效用而非用户参与度排序
- **单次请求解决复杂问题**：多主题复合查询单次请求即可解决
- **多跳搜索高效化**：深度研究流程只需更少的工具调用

---

## 构建有效的搜索查询

### 同时提供 `目标` 和 `搜索查询`

`目标`描述宏观任务；`搜索查询`确保优先处理特定关键词。两者结合使用效果显著更佳。

**推荐做法：**
```python
searcher.search(
    objective="我正在撰写关于阿尔茨海默症治疗的文献综述。查找过去两年针对淀粉样蛋白靶向疗法的同行评审研究论文和临床试验结果。",
    search_queries=[
        "淀粉样蛋白临床试验 2024-2025",
        "阿尔茨海默症单克隆抗体治疗结果",
        "lecanemab donanemab 试验结果"
    ],
)
```

**不推荐做法：**
```python
# 过于模糊 - 缺乏意图上下文
searcher.search(objective="阿尔茨海默症治疗")

# 缺少目标 - 无排序依据
searcher.search(search_queries=["阿尔茨海默症药物"])
```

### 目标撰写技巧

1. **说明宏观任务**："我正在撰写关于...的研究论文"、"我正在分析...市场"、"我正在准备关于...的演示"
2. **明确来源偏好**："优先选择政府官网"、"聚焦同行评审期刊"、"来自主流新闻媒体"
3. **包含时效要求**："过去6个月内"、"2024-2025年发布"、"最新可用数据"
4. **指定内容类型**："技术文档"、"临床试验结果"、"市场分析报告"、"产品公告"

### 按用例的目标示例

**学术研究：**
```
"我正在撰写关于CRISPR基因编辑在癌症治疗中应用的文献综述。
查找Nature、Science、Cell等高影响力期刊2023-2025年发表的同行评审论文。
优先选择临床试验结果和系统综述。"
```

**市场情报：**
```
"我正在为金融科技初创公司准备2025年Q1投资者材料。
查找美联储和SEC关于数字资产监管及加密公司银行合作的最新公告。
仅限过去三个月。"
```

**技术文档：**
```
"我正在设计机器学习课程。查找解释Transformer注意力机制的技术文档和API指南，
优先选择PyTorch或Hugging Face等官方框架文档。"
```

**时事追踪：**
```
"我正在追踪AI监管动态。查找欧盟、美国和英国政府过去一个月发布的
官方政策公告、立法行动和监管指南。"
```

---

## 搜索模式

使用`mode`参数优化工作流：

| 模式 | 最佳场景 | 摘要风格 | 延迟 |
|------|----------|---------------|---------|
| `one-shot` (默认) | 直接查询、单次请求工作流 | 全面、较长 | 较低 |
| `agentic` | 多步推理循环、智能体工作流 | 简洁、节省token | 稍高 |
| `fast` | 实时应用、UI自动补全 | 极简、速度优化 | ~1秒 |

### 模式适用场景

**`one-shot`** (默认)：
- 需要全面回答的单一研究问题
- 撰写论文章节需完整上下文
- 文档起草前的背景研究
- 任何只需单次搜索调用的场景

**`agentic`**：
- 多步骤研究流程（搜索→分析→再搜索）
- token效率至关重要的智能体循环
- 研究查询的迭代优化
- 与其他工具集成时（搜索→提取→合成）

**`fast`**：
- 实时自动补全或建议系统
- 写作过程中的快速事实核查
- 实时元数据查询
- 任何延迟敏感型应用

---

## 来源策略

控制结果包含或排除的域名：

```python
searcher.search(
    objective="查找新型癌症免疫治疗药物的临床试验结果",
    search_queries=["检查点抑制剂临床试验 2025"],
    source_policy={
        "allow_domains": ["clinicaltrials.gov", "nejm.org", "thelancet.com", "nature.com"],
        "deny_domains": ["reddit.com", "quora.com"],
        "after_date": "2024-01-01"
    },
)
```

### 来源策略参数

| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `allow_domains` | 字符串列表 | 仅包含这些域名的结果 |
| `deny_domains` | 字符串列表 | 排除这些域名的结果 |
| `after_date` | 字符串 (YYYY-MM-DD) | 仅包含此日期后发布的内容 |

### 按用例的域名列表

**学术研究：**
```python
allow_domains = [
    "nature.com", "science.org", "cell.com", "thelancet.com",
    "nejm.org", "bmj.com", "pnas.org", "arxiv.org",
    "pubmed.ncbi.nlm.nih.gov", "scholar.google.com"
]
```

**技术/AI：**
```python
allow_domains = [
    "arxiv.org", "openai.com", "anthropic.com", "deepmind.google",
    "huggingface.co", "pytorch.org", "tensorflow.org",
    "proceedings.neurips.cc", "proceedings.mlr.press"
]
```

**市场情报：**
```python
deny_domains = [
    "reddit.com", "quora.com", "medium.com",
    "wikipedia.org"  # 适用于事实核查，不适用于市场数据
]
```

**政府/政策：**
```python
allow_domains = [
    "gov", "europa.eu", "who.int", "worldbank.org",
    "imf.org", "oecd.org", "un.org"
]
```

---

## 控制结果数量

### `max_results` 参数

- 范围：1-20 (默认：10)
- 结果越多=覆盖越广但需处理更多token
- 结果越少=越聚焦但可能遗漏相关来源

**建议：**
- 快速事实核查：`max_results=3`
- 标准研究：`max_results=10` (默认)
- 全面调研：`max_results=20`

### 摘要长度控制

```python
searcher.search(
    objective="...",
    max_chars_per_result=10000,  # 默认：10000
)
```

- **短摘要 (1000-3000字符)**：快速摘要、元数据提取
- **中摘要 (5000-10000字符)**：标准研究、深度适中
- **长摘要 (10000-50000字符)**：完整文章内容、深度分析

---

## 常见模式

### 模式1：写作前研究

```python
# 撰写每个章节前搜索相关信息
result = searcher.search(
    objective="为NeurIPS论文引言查找Transformer注意力机制的最新进展",
    search_queries=["注意力机制创新 2024", "高效transformer"],
    max_results=10,
)

# 提取章节关键发现
for r in result["results"]:
    print(f"来源：{r['title']} ({r['url']})")
    # 使用摘要辅助写作
```

### 模式2：事实核查

```python
# 快速验证特定声明
result = searcher.search(
    objective="验证：GPT-4是否在MMLU基准测试中获得86.4%分数？",
    search_queries=["GPT-4 MMLU基准分数"],
    max_results=5,
)
```

### 模式3：竞争情报

```python
result = searcher.search(
    objective="查找2025年AI编程助手的最新产品发布和融资公告",
    search_queries=[
        "AI编程助手融资 2025",
        "代码生成工具发布",
        "AI开发者工具新产品"
    ],
    source_policy={"after_date": "2025-01-01"},
    max_results=15,
)
```

### 模式4：多语言研究

```python
# 搜索自动包含多语言结果
result = searcher.search(
    objective="查找关于AI监管的全球视角，包括欧盟、中国和美国方案",
    search_queries=[
        "欧盟AI法案实施 2025",
        "中国AI监管政策",
        "美国AI行政令更新"
    ],
)
```

---

## 故障排除

### 结果过少或无结果

- **扩展目标范围**：移除过度具体的限制条件
- **增加搜索查询**：同一概念的不同表述方式
- **放宽来源策略**：域名限制可能过严
- **检查日期筛选**：`after_date`可能设置过近

### 结果不相关

- **细化目标描述**：增加任务上下文信息
- **使用来源策略**：仅允许权威域名
- **添加排除条件**："排除[无关主题]"
- **优化搜索查询**：使用更精确的关键词

### 结果token过多

- **减少`max_results`**：从10降至5或3
- **缩短摘要长度**：降低`max_chars_per_result`
- **使用`agentic`模式**：摘要更简洁
- **使用`fast`模式**：极简摘要

---

## 另请参阅

- [API参考](api_reference.md) - 完整API参数说明
- [深度研究指南](deep_research_guide.md) - 综合研究任务指南
- [提取模式](extraction_patterns.md) - 特定URL内容提取
- [工作流方案](workflow_recipes.md) - 常见多步骤模式
