```markdown
---
name: perplexity-search
description: 通过LiteLLM和OpenRouter使用Perplexity模型执行AI驱动的实时网络搜索。该技能适用于检索实时信息、查找最新科学文献、获取带来源引用的可靠答案或访问超出模型知识截止日期的信息。通过单一OpenRouter API密钥即可访问多个Perplexity模型，包括Sonar Pro、Sonar Pro Search（高级代理搜索）和Sonar Reasoning Pro。
license: MIT许可证
compatibility: 使用Perplexity搜索需提供OpenRouter API密钥
metadata:
    skill-author: K-Dense Inc.
---

# Perplexity搜索

## 概述

通过LiteLLM和OpenRouter使用Perplexity模型执行AI驱动的网络搜索。Perplexity提供带来源引用的实时网络锚定答案，非常适合查找当前信息、最新科学文献以及超出模型训练数据截止日期的事实。

本技能通过OpenRouter提供所有Perplexity模型的访问权限，仅需单一API密钥（无需单独注册Perplexity账户）。

## 使用场景

在以下情况使用本技能：
- 搜索当前信息或最新进展（2024年及以后）
- 查找最新科学出版物和研究
- 获取基于网络来源的实时答案
- 通过来源引用验证事实
- 跨多个领域进行文献检索
- 访问超出模型知识截止日期的信息
- 执行特定领域研究（生物医学、技术、临床）
- 比较当前方法或技术

**请勿用于**：
- 简单计算或逻辑问题（请直接使用模型）
- 需要代码执行的任务（请使用标准工具）
- 模型训练数据范围内的问题（除非需要验证）

## 快速入门

### 设置（一次性）

1. **获取OpenRouter API密钥**：
   - 访问 https://openrouter.ai/keys
   - 创建账户并生成API密钥
   - 为账户充值（建议最低$5）

2. **配置环境**：
   ```bash
   # 设置API密钥
   export OPENROUTER_API_KEY='sk-or-v1-your-key-here'

   # 或使用安装脚本
   python scripts/setup_env.py --api-key sk-or-v1-your-key-here
   ```

3. **安装依赖项**：
   ```bash
   uv pip install litellm
   ```

4. **验证设置**：
   ```bash
   python scripts/perplexity_search.py --check-setup
   ```

详细设置说明、故障排除和安全实践请参阅 `references/openrouter_setup.md`。

### 基础用法

**简单搜索：**
```bash
python scripts/perplexity_search.py "CRISPR基因编辑领域的最新进展有哪些？"
```

**保存结果：**
```bash
python scripts/perplexity_search.py "近期CAR-T疗法临床试验" --output results.json
```

**指定模型：**
```bash
python scripts/perplexity_search.py "比较mRNA疫苗与病毒载体疫苗" --model sonar-pro-search
```

**详细输出：**
```bash
python scripts/perplexity_search.py "量子计算在药物发现中的应用" --verbose
```

## 可用模型

通过 `--model` 参数访问模型：

- **sonar-pro**（默认）：通用搜索，成本与质量的最佳平衡
- **sonar-pro-search**：具备多步推理能力的高级代理搜索
- **sonar**：基础模型，简单查询最具成本效益
- **sonar-reasoning-pro**：带逐步分析的高级推理
- **sonar-reasoning**：基础推理能力

**模型选择指南：**
- 默认查询 → `sonar-pro`
- 复杂多步分析 → `sonar-pro-search`
- 需要显式推理 → `sonar-reasoning-pro`
- 简单事实查询 → `sonar`
- 成本敏感的批量查询 → `sonar`

详细模型对比、用例、定价和性能特征请参阅 `references/model_comparison.md`。

## 构建高效查询

### 保持具体详细

**优秀示例：**
- "2024年发表的关于CAR-T细胞疗法治疗B细胞淋巴瘤的最新临床试验结果有哪些？"
- "比较mRNA疫苗与病毒载体疫苗在COVID-19中的有效性和安全性"
- "根据2023-2024年研究数据，解释AlphaFold3相比AlphaFold2在准确度指标上的改进"

**不良示例：**
- "告诉我癌症治疗信息"（过于宽泛）
- "CRISPR"（过于模糊）
- "疫苗"（缺乏具体性）

### 包含时间限制

Perplexity搜索实时网络数据：
- "Nature Medicine期刊2024年发表了哪些关于长期新冠的论文？"
- "过去6个月大型语言模型效率领域的最新进展是什么？"
- "NeurIPS 2023会议关于AI安全发布了哪些公告？"

### 指定领域和来源

为获得高质量结果，可注明来源偏好：
- "根据高影响力期刊的同行评审出版物..."
- "基于FDA批准的治疗方案..."
- "来自clinicaltrials.gov等临床试验注册库..."

### 结构化复杂查询

将复杂问题拆分为清晰组件：
1. **主题**：核心研究对象
2. **范围**：关注的具体方面
3. **背景**：时间范围、领域、限制条件
4. **输出**：期望的答案格式或类型

**示例：**
"根据2023至2024年发表的研究，AlphaFold3在蛋白质结构预测方面相比AlphaFold2有哪些改进？需包含具体准确度指标和基准测试数据。"

完整查询设计指南、领域特定模式和高级技巧请参阅 `references/search_strategies.md`。

## 常见用例

### 科学文献检索

```bash
python scripts/perplexity_search.py \
  "近期研究（2023-2024）关于肠道微生物组在帕金森病中的作用有哪些结论？聚焦同行评审研究并注明已识别的特定菌种。" \
  --model sonar-pro
```

### 技术文档查询

```bash
python scripts/perplexity_search.py \
  "如何使用Python实现Kafka到PostgreSQL的实时数据流传输？需包含处理背压和确保精确一次语义的注意事项。" \
  --model sonar-reasoning-pro
```

### 对比分析

```bash
python scripts/perplexity_search.py \
  "比较PyTorch与TensorFlow在实现Transformer模型时的易用性、性能和生态系统支持。需包含近期研究的基准测试数据。" \
  --model sonar-pro-search
```

### 临床研究

```bash
python scripts/perplexity_search.py \
  "间歇性禁食管理成人2型糖尿病的证据有哪些？聚焦随机对照试验并报告HbA1c变化和减重结果。" \
  --model sonar-pro
```

### 趋势分析

```bash
python scripts/perplexity_search.py \
  "过去5年单细胞RNA测序技术的主要趋势是什么？重点说明通量、成本和分辨率的改进，并给出具体案例。" \
  --model sonar-pro
```

## 结果处理

### 程序化访问

将 `perplexity_search.py` 作为模块使用：

```python
from scripts.perplexity_search import search_with_perplexity

result = search_with_perplexity(
    query="CRISPR领域最新进展有哪些？",
    model="openrouter/perplexity/sonar-pro",
    max_tokens=4000,
    temperature=0.2,
    verbose=False
)

if result["success"]:
    print(result["answer"])
    print(f"使用token数: {result['usage']['total_tokens']}")
else:
    print(f"错误: {result['error']}")
```

### 保存与处理结果

```bash
# 保存为JSON
python scripts/perplexity_search.py "查询内容" --output results.json

# 使用jq处理
cat results.json | jq '.answer'
cat results.json | jq '.usage'
```

### 批量处理

创建多查询脚本：

```bash
#!/bin/bash
queries=(
  "2024年CRISPR进展"
  "mRNA疫苗技术进步"
  "AlphaFold3准确度改进"
)

for query in "${queries[@]}"; do
  echo "搜索: $query"
  python scripts/perplexity_search.py "$query" --output "results_$(echo $query | tr ' ' '_').json"
  sleep 2  # 限速控制
done
```

## 成本管理

Perplexity模型采用分级定价：

**单次查询近似成本：**
- Sonar: $0.001-0.002（最具成本效益）
- Sonar Pro: $0.002-0.005（推荐默认）
- Sonar Reasoning Pro: $0.005-0.010
- Sonar Pro Search: $0.020-0.050+（功能最全面）

**成本优化策略：**
1. 简单事实查询使用 `sonar`
2. 多数查询默认使用 `sonar-pro`
3. 复杂分析保留使用 `sonar-pro-search`
4. 设置 `--max-tokens` 限制响应长度
5. 通过 https://openrouter.ai/activity 监控用量
6. 在OpenRouter仪表板设置消费限额

## 故障排除

### API密钥未设置

**错误**："未配置OpenRouter API密钥"

**解决方案**：
```bash
export OPENROUTER_API_KEY='sk-or-v1-your-key-here'
# 或运行安装脚本
python scripts/setup_env.py --api-key sk-or-v1-your-key-here
```

### LiteLLM未安装

**错误**："未安装LiteLLM"

**解决方案**：
```bash
uv pip install litellm
```

### 速率限制

**错误**："超出速率限制"

**解决方案**：
- 重试前等待数秒
- 通过 https://openrouter.ai/keys 提高速率限制
- 在批量处理中添加请求间隔

### 余额不足

**错误**："余额不足"

**解决方案**：
- 通过 https://openrouter.ai/account 充值
- 启用自动充值避免服务中断

完整故障排除指南请参阅 `references/openrouter_setup.md`。

## 与其他技能集成

本技能可与其他科研技能互补使用：

### 文献综述

与 `literature-review` 技能配合：
1. 使用Perplexity查找近期论文和预印本
2. 通过实时网络结果补充PubMed检索
3. 验证引用并发现相关研究
4. 获取数据库索引后的最新进展

### 科研写作

与 `scientific-writing` 技能配合：
1. 为引言/讨论部分查找最新参考文献
2. 验证当前技术发展水平
3. 检查最新术语和规范
4. 识别近期竞争性方法

### 假设生成

与 `hypothesis-generation` 技能配合：
1. 搜索最新研究发现
2. 识别当前知识空白
3. 发现近期方法论进展
4. 探索新兴研究方向

### 批判性思维

与 `scientific-critical-thinking` 技能配合：
1. 寻找支持/反对假设的证据
2. 定位方法论批判
3. 识别领域争议点
4. 通过当前证据验证主张

## 最佳实践

### 查询设计

1. **保持具体**：包含领域、时间范围和限制条件
2. **使用术语**：领域相关的关键词和短语
3. **指定来源**：注明偏好出版物类型或期刊
4. **结构化问题**：明确背景的清晰组件
5. **迭代优化**：根据初步结果改进查询

### 模型选择

1. **从sonar-pro开始**：多数查询的良好默认选择
2. **复杂查询升级**：多步分析使用sonar-pro-search
3. **简单查询降级**：基础事实查询使用sonar
4. **使用推理模型**：需要逐步分析时选用

### 成本优化

1. **选择合适模型**：根据查询复杂度匹配模型
2. **设置token限制**：使用 `--max-tokens` 控制成本
3. **监控用量**：定期检查OpenRouter仪表板
4. **高效批量处理**：尽可能合并相关简单查询
5. **缓存结果**：保存并复用重复查询结果

### 安全性

1. **保护API密钥**：切勿提交至版本控制系统
2. **使用环境变量**：密钥与代码分离存储
3. **设置消费限额**：在OpenRouter仪表板配置
4. **监控使用情况**：关注异常活动
5. **定期轮换密钥**：周期性更换密钥

## 资源

### 内置资源

**脚本：**
- `scripts/perplexity_search.py`：带CLI接口的主搜索脚本
- `scripts/setup_env.py`：环境设置与验证工具

**参考文档：**
- `references/search_strategies.md`：完整查询设计指南
- `references/model_comparison.md`：详细模型对比与选择指南
- `references/openrouter_setup.md`：完整设置、故障排除与安全指南

**资源文件：**
- `assets/.env.example`：环境文件示例模板

### 外部资源

**OpenRouter：**
- 仪表板： https://openrouter.ai/account
- API密钥： https://openrouter.ai/keys
- Perplexity模型： https://openrouter.ai/perplexity
- 用量监控： https://openrouter.ai/activity
- 文档： https://openrouter.ai/docs

**LiteLLM：**
- 文档： https://docs.litellm.ai/
- OpenRouter供应商： https://docs.litellm.ai/docs/providers/openrouter
- GitHub： https://github.com/BerriAI/litellm

**Perplexity：**
- 官方文档： https://docs.perplexity.ai/

## 依赖项

### 必需项

```bash
# 用于API访问的LiteLLM
uv pip install litellm
```

### 可选项

```bash
# 支持.env文件
uv pip install python-dotenv

# JSON处理工具（通常预装）
uv pip install jq
```

### 环境变量

必需项：
- `OPENROUTER_API_KEY`：您的OpenRouter API密钥

可选项：
- `DEFAULT_MODEL`：默认使用模型（默认值：sonar-pro）
- `DEFAULT_MAX_TOKENS`：默认最大token数（默认值：4000）
- `DEFAULT_TEMPERATURE`：默认温度参数（默认值：0.2）

## 总结

本技能提供：

1. **实时网络搜索**：访问超出训练数据截止日期的当前信息
2. **多模型支持**：从高性价比Sonar到高级Sonar Pro Search
3. **简单设置**：单一OpenRouter API密钥，无需单独Perplexity账户
4. **完整指导**：详细的查询设计和模型选择参考
5. **成本效益**：用量监控的按量付费模式
6. **科研导向**：针对研究、文献检索和技术查询优化
7. **轻松集成**：与其他科研技能无缝协作

执行AI驱动的网络搜索，获取带来源引用的实时信息、最新研究和可靠答案。
```
