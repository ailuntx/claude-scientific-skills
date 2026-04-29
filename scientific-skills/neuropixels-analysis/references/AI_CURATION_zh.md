# AI辅助筛选参考指南

基于SpikeAgent方法，使用AI视觉分析进行单元筛选的指南。

## 概述

AI辅助筛选利用视觉语言模型分析尖峰排序可视化结果，提供类似人类专家的质量评估。

### 工作流程

```
传统流程：指标 → 阈值 → 标签
AI增强流程：指标 → AI视觉分析 → 置信度评分 → 标签
```

## Claude代码集成

在Claude Code中使用此技能时，Claude可直接分析波形图而无需API设置：

1. 生成单元报告或图表
2. 要求Claude分析可视化结果
3. Claude将提供专家级筛选决策

Claude Code示例工作流：
```python
# 为单元生成图表
npa.plot_unit_summary(analyzer, unit_id=0, output='unit_0_summary.png')

# 询问Claude："请分析此单元的波形和自相关图，
# 判断它是良好隔离的单单元、多单元活动还是噪声"
```

Claude可评估：
- 波形一致性和形状
- 自相关图显示的不应期违规
- 振幅随时间稳定性
- 单元整体隔离质量

## 快速入门

### 生成单元报告

```python
import neuropixels_analysis as npa

# 创建单元可视化报告
report = npa.generate_unit_report(analyzer, unit_id=0, output_dir='reports/')

# 报告包含：
# - 波形、模板、自相关图
# - 振幅时序、ISI直方图
# - 质量指标摘要
# - 用于API的Base64编码图像
```

### AI视觉分析

```python
from anthropic import Anthropic

# 设置API客户端
client = Anthropic()

# 分析单个单元
result = npa.analyze_unit_visually(
    analyzer,
    unit_id=0,
    api_client=client,
    model='claude-opus-4.5',
    task='quality_assessment'
)

print(f"分类结果: {result['classification']}")
print(f"推理依据: {result['reasoning']}")
```

### 批量分析

```python
# 分析所有单元
results = npa.batch_visual_curation(
    analyzer,
    api_client=client,
    output_dir='ai_curation/',
    progress_callback=lambda i, n: print(f"进度: {i}/{n}")
)

# 获取标签
ai_labels = {uid: r['classification'] for uid, r in results.items()}
```

## 交互式筛选会话

人机协作的AI辅助筛选：

```python
# 创建会话
session = npa.CurationSession.create(
    analyzer,
    output_dir='curation_session/',
    sort_by_confidence=True  # 优先显示不确定单元
)

# 处理单元
while True:
    unit = session.current_unit()
    if unit is None:
        break

    print(f"单元 {unit.unit_id}:")
    print(f"  自动分类: {unit.auto_classification} (置信度: {unit.confidence:.2f})")

    # 生成报告
    report = npa.generate_unit_report(analyzer, unit.unit_id)

    # 获取AI意见
    ai_result = npa.analyze_unit_visually(analyzer, unit.unit_id, api_client=client)
    session.set_ai_classification(unit.unit_id, ai_result['classification'])

    # 人工决策
    decision = input("决策 (good/mua/noise/skip): ")
    if decision != 'skip':
        session.set_decision(unit.unit_id, decision)

    session.next_unit()

# 导出结果
labels = session.get_final_labels()
session.export_decisions('final_curation.csv')
```

## 分析任务

### 质量评估（默认）

分析波形形状、不应期、振幅稳定性。

```python
result = npa.analyze_unit_visually(analyzer, uid, task='quality_assessment')
# 返回: 'good', 'mua', 或 'noise'
```

### 合并候选检测

判断两个单元是否应合并。

```python
result = npa.analyze_unit_visually(analyzer, uid, task='merge_candidate')
# 返回: 'merge' 或 'keep_separate'
```

### 漂移评估

评估记录中的运动/漂移。

```python
result = npa.analyze_unit_visually(analyzer, uid, task='drift_assessment')
# 返回漂移幅度和校正建议
```

## 自定义提示

创建定制分析提示：

```python
from neuropixels_analysis.ai_curation import create_curation_prompt

# 获取基础提示
prompt = create_curation_prompt(
    task='quality_assessment',
    additional_context='重点关注波形振幅一致性'
)

# 或完全自定义
custom_prompt = """
分析此单元并判断是否为快速放电中间神经元。

检查要点：
1. 窄波形（峰谷间距 < 0.5ms）
2. 高放电频率
3. 规则ISI分布

分类为：FSI (快速放电中间神经元) 或 OTHER
"""

result = npa.analyze_unit_visually(
    analyzer, uid,
    api_client=client,
    custom_prompt=custom_prompt
)
```

## 结合AI与指标

最佳实践：同时使用AI和量化指标：

```python
def hybrid_curation(analyzer, metrics, api_client):
    """结合指标和AI实现稳健筛选"""
    labels = {}

    for unit_id in metrics.index:
        row = metrics.loc[unit_id]

        # 指标高置信度情况
        if row['snr'] > 10 and row['isi_violations_ratio'] < 0.001:
            labels[unit_id] = 'good'
            continue

        if row['snr'] < 1.5:
            labels[unit_id] = 'noise'
            continue

        # 不确定情况：使用AI
        result = npa.analyze_unit_visually(
            analyzer, unit_id, api_client=api_client
        )
        labels[unit_id] = result['classification']

    return labels
```

## 会话管理

### 恢复会话

```python
# 恢复中断的会话
session = npa.CurationSession.load('curation_session/20250101_120000/')

# 检查进度
summary = session.get_summary()
print(f"进度: {summary['progress_pct']:.1f}%")
print(f"剩余: {summary['remaining']} 个单元")

# 从断点继续
unit = session.current_unit()
```

### 会话导航

```python
# 跳转至特定单元
session.go_to_unit(42)

# 前后导航
session.prev_unit()
session.next_unit()

# 更新决策
session.set_decision(42, 'good', notes='明确的不应期')
```

### 导出结果

```python
# 获取最终标签（优先级：人工 > AI > 自动）
labels = session.get_final_labels()

# 导出详细结果
df = session.export_decisions('curation_results.csv')

# 摘要统计
summary = session.get_summary()
print(f"优质单元: {summary['decisions'].get('good', 0)}")
print(f"多单元活动: {summary['decisions'].get('mua', 0)}")
print(f"噪声: {summary['decisions'].get('noise', 0)}")
```

## 可视化报告组件

生成报告包含6个面板：

| 面板 | 内容 | 检查要点 |
|-------|---------|------------------|
| 波形 | 单个尖峰波形 | 一致性、形状 |
| 模板 | 均值±标准差 | 清晰负峰、生理形态 |
| 自相关图 | 尖峰时序 | 0ms处间隙（不应期） |
| 振幅 | 振幅时序 | 稳定性、无漂移 |
| ISI直方图 | 尖峰间期 | 不应期间隙 < 1.5ms |
| 指标 | 质量数值 | SNR、ISI违规、存在性 |

## API支持

当前支持的API：

| 服务商 | 客户端 | 模型示例 |
|----------|--------|----------------|
| Anthropic | `anthropic.Anthropic()` | claude-opus-4.5 |
| OpenAI | `openai.OpenAI()` | gpt-4-vision-preview |
| Google | `google.generativeai` | gemini-pro-vision |

### Anthropic示例

```python
from anthropic import Anthropic

client = Anthropic(api_key="your-api-key")
result = npa.analyze_unit_visually(analyzer, uid, api_client=client)
```

### OpenAI示例

```python
from openai import OpenAI

client = OpenAI(api_key="your-api-key")
result = npa.analyze_unit_visually(
    analyzer, uid,
    api_client=client,
    model='gpt-4-vision-preview'
)
```

## 最佳实践

1. **AI用于不确定案例** - 避免在明显优质/噪声单元上浪费API调用
2. **结合量化指标** - AI应作为量化指标的补充而非替代
3. **人工监督** - 重要分析需复核AI决策
4. **保存会话** - 始终使用CurationSession跟踪决策
5. **记录推理过程** - 使用注释字段记录决策依据

## 成本优化

```python
# 仅对不确定单元使用AI
uncertain_units = metrics.query("""
    snr > 2 and snr < 8 and
    isi_violations_ratio > 0.001 and isi_violations_ratio < 0.1
""").index.tolist()

# 仅批量处理这些单元
results = npa.batch_visual_curation(
    analyzer,
    unit_ids=uncertain_units,
    api_client=client
)
```

## 参考文献

- [SpikeAgent](https://github.com/SpikeAgent/SpikeAgent) - AI驱动的尖峰排序助手
- [Anthropic视觉API](https://docs.anthropic.com/en/docs/vision)
- [GPT-4视觉](https://platform.openai.com/docs/guides/vision)
