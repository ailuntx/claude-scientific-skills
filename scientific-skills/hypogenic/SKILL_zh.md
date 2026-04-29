---
name: hypogenic
description: 基于LLM驱动的表格数据集假设生成与测试自动化。适用于需要系统性地探索实证数据中模式假设的场景（例如，欺骗检测、内容分析）。结合文献洞察与数据驱动的假设检验。手动制定假设请使用 hypothesis-generation；创意构思请使用 scientific-brainstorming。
license: MIT 许可证
metadata:
    skill-author: K-Dense Inc.
---

# Hypogenic

## 概述

Hypogenic 提供使用大型语言模型的自动化假设生成与测试，以加速科学发现。该框架支持三种方法：HypoGeniC（数据驱动假设生成）、HypoRefine（文献与数据协同整合）和 Union 方法（文献与数据驱动假设的机制性结合）。

## 快速开始

几分钟内即可开始使用 Hypogenic：

```bash
# 安装包
uv pip install hypogenic

# 克隆示例数据集
git clone https://github.com/ChicagoHAI/HypoGeniC-datasets.git ./data

# 运行基本假设生成
hypogenic_generation --config ./data/your_task/config.yaml --method hypogenic --num_hypotheses 20

# 对生成的假设进行推理
hypogenic_inference --config ./data/your_task/config.yaml --hypotheses output/hypotheses.json
```

**或使用 Python API：**

```python
from hypogenic import BaseTask

# 使用你的配置创建任务
task = BaseTask(config_path="./data/your_task/config.yaml")

# 生成假设
task.generate_hypotheses(method="hypogenic", num_hypotheses=20)

# 运行推理
results = task.inference(hypothesis_bank="./output/hypotheses.json")
```

## 何时使用此技能

在以下场景中使用此技能：
- 从观测数据集生成科学假设
- 系统性地测试多个竞争性假设
- 将文献洞察与实证模式相结合
- 通过自动化假设构思加速研究发现
- 需要假设驱动分析的领域：欺骗检测、AI生成内容识别、心理健康指标、预测建模或其他实证研究

## 主要特性

**自动化假设生成**
- 在数分钟内从数据中生成10-20+个可检验的假设
- 基于验证性能进行迭代优化
- 支持基于API（OpenAI、Anthropic）和本地LLM

**文献整合**
- 通过PDF处理从研究论文中提取洞察
- 将理论基础与实证模式相结合
- 使用GROBID的系统化文献到假设流水线

**性能优化**
- Redis缓存减少重复实验的API成本
- 并行处理大规模假设测试
- 自适应优化聚焦于具有挑战性的样本

**灵活配置**
- 基于模板的提示工程，支持变量注入
- 针对特定领域任务的自定义标签提取
- 模块化架构，易于扩展

**经过验证的结果**
- 相比少样本基线提升8.97%
- 相比纯文献方法提升15.75%
- 80-84%的假设多样性（非冗余洞察）
- 人类评估者报告显著的决策改进

## 核心能力

### 1. HypoGeniC：数据驱动假设生成

仅通过观测数据生成假设，通过迭代优化实现。

**流程：**
1. 使用一小部分数据子集初始化，生成候选假设
2. 基于性能迭代优化假设
3. 用来自挑战性样本的新假设替换表现不佳的假设

**最佳场景：** 没有现有文献的探索性研究，新颖数据集中的模式发现

### 2. HypoRefine：文献与数据整合

通过代理框架将现有文献与实证数据协同结合起来。

**流程：**
1. 从相关研究论文中提取洞察（通常10篇论文）
2. 从文献生成基于理论的假设
3. 从观测模式生成数据驱动假设
4. 通过迭代改进优化两个假设库

**最佳场景：** 具有既定理论基础的研究，验证或扩展现有理论

### 3. Union 方法

将纯文献假设与框架输出机制性地结合。

**变体：**
- **Literature ∪ HypoGeniC**：将文献假设与数据驱动生成结合
- **Literature ∪ HypoRefine**：将文献假设与整合方法结合

**最佳场景：** 全面的假设覆盖，消除冗余同时保持多样性视角

## 安装

通过 pip 安装：
```bash
uv pip install hypogenic
```

**可选依赖项：**
- **Redis 服务器**（端口 6832）：启用 LLM 响应的缓存，显著降低迭代假设生成期间的 API 成本
- **s2orc-doc2json**：在 HypoRefine 工作流中处理文献 PDF 所需
- **GROBID**：PDF 预处理所需（见文献处理部分）

**克隆示例数据集：**
```bash
# 针对 HypoGeniC 示例
git clone https://github.com/ChicagoHAI/HypoGeniC-datasets.git ./data

# 针对 HypoRefine/Union 示例
git clone https://github.com/ChicagoHAI/Hypothesis-agent-datasets.git ./data
```

## 数据集格式

数据集必须遵循 HuggingFace 数据集格式，并具有特定的命名约定：

**必需文件：**
- `<TASK>_train.json`：训练数据
- `<TASK>_val.json`：验证数据
- `<TASK>_test.json`：测试数据

**JSON 中必需的键：**
- `text_features_1` 至 `text_features_n`：包含特征值的字符串列表
- `label`：包含真实标签的字符串列表

**示例（标题点击预测）：**
```json
{
  "headline_1": [
    "What Up, Comet? You Just Got *PROBED*",
    "Scientists Made a Breakthrough in Quantum Computing"
  ],
  "headline_2": [
    "Scientists Everywhere Were Holding Their Breath Today. Here's Why.",
    "New Quantum Computer Achieves Milestone"
  ],
  "label": [
    "Headline 2 has more clicks than Headline 1",
    "Headline 1 has more clicks than Headline 2"
  ]
}
```

**重要说明：**
- 所有列表长度必须相同
- 标签格式必须与你的 `extract_label()` 函数输出格式匹配
- 特征键可以根据你的领域自定义（例如，`review_text`、`post_content` 等）

## 配置

每个任务需要一个 `config.yaml` 文件，指定以下内容：

**必需元素：**
- 数据集路径（训练/验证/测试）
- 提示模板，用于：
  - 观察生成
  - 批量假设生成
  - 假设推理
  - 相关性检查
  - 自适应方法（用于 HypoRefine）

**模板能力：**
- 数据集占位符，用于动态变量注入（例如，`${text_features_1}`、`${num_hypotheses}`）
- 用于领域特定解析的自定义标签提取函数
- 基于角色的提示结构（系统、用户、助手角色）

**配置结构：**
```yaml
task_name: your_task_name

train_data_path: ./your_task_train.json
val_data_path: ./your_task_val.json
test_data_path: ./your_task_test.json

prompt_templates:
  # 额外键，用于可重用的提示组件
  observations: |
    Feature 1: ${text_features_1}
    Feature 2: ${text_features_2}
    Observation: ${label}
  
  # 必需模板
  batched_generation:
    system: "你的系统提示在此"
    user: "你的用户提示，包含 ${num_hypotheses} 占位符"
  
  inference:
    system: "你的推理系统提示"
    user: "你的推理用户提示"
  
  # 高级功能的可选模板
  few_shot_baseline: {...}
  is_relevant: {...}
  adaptive_inference: {...}
  adaptive_selection: {...}
```

有关完整示例配置，请参考 `references/config_template.yaml`。

## 文献处理（HypoRefine/Union 方法）

要使用基于文献的假设生成，必须预处理 PDF 论文。

> **注意：** 以下命令在克隆的 [HypoGenic 仓库](https://github.com/ChicagoHAI/hypothesis-generation) 内运行，而非从此技能目录运行。

**第 1 步：安装 GROBID**（仅首次）
```bash
bash ./modules/setup_grobid.sh
```

**第 2 步：添加 PDF 文件**
将研究论文放入 `literature/YOUR_TASK_NAME/raw/`

**第 3 步：处理 PDF**
```bash
# 启动 GROBID 服务
bash ./modules/run_grobid.sh

# 处理你的任务的 PDF
cd examples
python pdf_preprocess.py --task_name YOUR_TASK_NAME
```

这将 PDF 转换为假设提取的结构化格式。未来版本将支持自动化文献搜索。

## CLI 使用

### 假设生成

```bash
hypogenic_generation --help
```

**关键参数：**
- 任务配置文件路径
- 模型选择（基于 API 或本地）
- 生成方法（HypoGeniC、HypoRefine 或 Union）
- 生成的假设数量
- 假设库的输出目录

### 假设推理

```bash
hypogenic_inference --help
```

**关键参数：**
- 任务配置文件路径
- 假设库文件路径
- 测试数据集路径
- 推理方法（默认或多假设）
- 结果输出文件

## Python API 使用

对于程序化控制和自定义工作流，直接在 Python 代码中使用 Hypogenic：

### 基本 HypoGeniC 生成

```python
from hypogenic import BaseTask

# 首先克隆示例数据集
# git clone https://github.com/ChicagoHAI/HypoGeniC-datasets.git ./data

# 加载你的任务，并传入自定义的 extract_label 函数
task = BaseTask(
    config_path="./data/your_task/config.yaml",
    extract_label=lambda text: extract_your_label(text)
)

# 生成假设
task.generate_hypotheses(
    method="hypogenic",
    num_hypotheses=20,
    output_path="./output/hypotheses.json"
)

# 运行推理
results = task.inference(
    hypothesis_bank="./output/hypotheses.json",
    test_data="./data/your_task/your_task_test.json"
)
```

### HypoRefine/Union 方法

```python
# 对于文献整合的方法
# git clone https://github.com/ChicagoHAI/Hypothesis-agent-datasets.git ./data

# 使用 HypoRefine 生成
task.generate_hypotheses(
    method="hyporefine",
    num_hypotheses=15,
    literature_path="./literature/your_task/",
    output_path="./output/"
)
# 这生成 3 个假设库：
# - HypoRefine（整合方法）
# - 纯文献假设
# - Literature∪HypoRefine（联合）
```

### 多假设推理

```python
from examples.multi_hyp_inference import run_multi_hypothesis_inference

# 同时测试多个假设
results = run_multi_hypothesis_inference(
    config_path="./data/your_task/config.yaml",
    hypothesis_bank="./output/hypotheses.json",
    test_data="./data/your_task/your_task_test.json"
)
```

### 自定义标签提取

`extract_label()` 函数对于解析 LLM 输出至关重要。根据你的任务实现：

```python
def extract_label(llm_output: str) -> str:
    """从 LLM 推理文本提取预测标签。
    
    默认行为：搜索 'final answer:\s+(.*)' 模式。
    根据你的领域特定输出格式进行自定义。
    """
    import re
    match = re.search(r'final answer:\s+(.*)', llm_output, re.IGNORECASE)
    if match:
        return match.group(1).strip()
    return llm_output.strip()
```

**重要：** 提取的标签必须与数据集中 `label` 值的格式匹配，以便正确计算准确率。

## 工作流示例

### 示例 1：数据驱动假设生成（HypoGeniC）

**场景：** 在没有先验理论框架的情况下检测 AI 生成内容

**步骤：**
1. 准备包含文本样本和标签（人类 vs. AI 生成）的数据集
2. 创建带有适当提示模板的 `config.yaml`
3. 运行假设生成：
   ```bash
   hypogenic_generation --config config.yaml --method hypogenic --num_hypotheses 20
   ```
4. 在测试集上运行推理：
   ```bash
   hypogenic_inference --config config.yaml --hypotheses output/hypotheses.json --test_data data/test.json
   ```
5. 分析形式性、语法精确性和语气差异等模式的结果

### 示例 2：文献通知的假设测试（HypoRefine）

**场景：** 基于现有研究构建酒店评论中的欺骗检测

**步骤：**
1. 收集 10 篇关于语言欺骗线索的相关论文
2. 准备包含真实和欺诈评论的数据集
3. 配置包含文献处理和数据生成模板的 `config.yaml`
4. 运行 HypoRefine：
   ```bash
   hypogenic_generation --config config.yaml --method hyporefine --papers papers/ --num_hypotheses 15
   ```
5. 测试检查代词频率、细节特异性和其他语言模式的假设
6. 比较基于文献和数据驱动的假设性能

### 示例 3：全面假设覆盖（Union 方法）

**场景：** 精神压力检测，最大化假设多样性

**步骤：**
1. 从心理健康研究论文生成文献假设
2. 从社交媒体帖子生成数据驱动假设
3. 运行 Union 方法合并并去重：
   ```bash
   hypogenic_generation --config config.yaml --method union --literature_hypotheses lit_hyp.json
   ```
4. 推理捕获理论构造（发帖行为变化）和数据模式（情感语言转变）

## 性能优化

**缓存：** 启用 Redis 缓存以减少重复 LLM 调用的 API 成本和计算时间

**并行处理：** 利用多个工作线程进行大规模假设生成和测试

**自适应优化：** 使用挑战性样本迭代提高假设质量

## 预期结果

使用 hypogenic 的研究已证明：
- AI 内容检测任务准确率提升 14.19%
- 欺骗检测任务准确率提升 7.44%
- 80-84% 的假设对提供独特、非冗余的洞察
- 人类评估者在多个研究领域给予高有用性评分

## 故障排查

**问题：** 生成的假设过于通用
**解决方案：** 在 `config.yaml` 中优化提示模板，要求更具体、可检验的假设

**问题：** 推理性能差
**解决方案：** 确保数据集有足够的训练样本，调整假设生成参数，或增加假设数量

**问题：** 标签提取失败
**解决方案：** 为领域特定输出解析实现自定义 `extract_label()` 函数

**问题：** GROBID PDF 处理失败
**解决方案：** 确保 GROBID 服务正在运行（从克隆的仓库运行 `bash ./modules/run_grobid.sh`），且 PDF 是有效的研究论文

## 创建自定义任务

要向 Hypogenic 添加新任务或数据集：

### 第 1 步：准备数据集

按照所需格式创建三个 JSON 文件：
- `your_task_train.json`
- `your_task_val.json`
- `your_task_test.json`

每个文件必须包含文本特征的键（`text_features_1` 等）和 `label`。

### 第 2 步：创建 config.yaml

定义你的任务配置，包含：
- 任务名称和数据集路径
- 用于观察、生成、推理的提示模板
- 可重用提示组件的任何额外键
- 占位符变量（例如，`${text_features_1}`、`${num_hypotheses}`）

### 第 3 步：实现 extract_label 函数

创建解析 LLM 输出的自定义标签提取函数，适用于你的领域：

```python
from hypogenic import BaseTask

def extract_my_label(llm_output: str) -> str:
    """为你的任务自定义标签提取。
    
    必须返回与数据集 'label' 字段相同格式的标签。
    """
    # 示例：从特定格式提取
    if "Final prediction:" in llm_output:
        return llm_output.split("Final prediction:")[-1].strip()
    
    # 回退到默认模式
    import re
    match = re.search(r'final answer:\s+(.*)', llm_output, re.IGNORECASE)
    return match.group(1).strip() if match else llm_output.strip()

# 使用你的自定义任务
task = BaseTask(
    config_path="./your_task/config.yaml",
    extract_label=extract_my_label
)
```

### 第 4 步：（可选）处理文献

对于 HypoRefine/Union 方法：
1. 创建 `literature/your_task_name/raw/` 目录
2. 添加相关研究论文 PDF
3. 运行 GROBID 预处理
4. 使用 `pdf_preprocess.py` 处理

### 第 5 步：生成和测试

使用 CLI 或 Python API 运行假设生成和推理：

```bash
# CLI 方式
hypogenic_generation --config your_task/config.yaml --method hypogenic --num_hypotheses 20
hypogenic_inference --config your_task/config.yaml --hypotheses output/hypotheses.json

# 或使用 Python API（参见 Python API 使用部分）
```

## 仓库结构

了解仓库布局：

```
hypothesis-generation/
├── hypogenic/              # 核心包代码
├── hypogenic_cmd/          # CLI 入口点
├── hypothesis_agent/       # HypoRefine 代理框架
├── literature/            # 文献处理工具
├── modules/               # GROBID 和预处理模块
├── examples/              # 示例脚本
│   ├── generation.py      # 基本 HypoGeniC 生成
│   ├── union_generation.py # HypoRefine/Union 生成
│   ├── inference.py       # 单假设推理
│   ├── multi_hyp_inference.py # 多假设推理
│   └── pdf_preprocess.py  # 文献 PDF 处理
├── data/                  # 示例数据集（单独克隆）
├── tests/                 # 单元测试
└── IO_prompting/          # 提示模板和实验
```

**关键目录：**
- **hypogenic/**：包含 BaseTask 和生成逻辑的主包
- **examples/**：常见工作流的参考实现
- **literature/**：PDF 处理和文献提取的工具
- **modules/**：外部工具集成（GROBID 等）

## 相关出版物

### HypoBench (2025)

Liu, H., Huang, S., Hu, J., Zhou, Y., & Tan, C. (2025). HypoBench: Towards Systematic and Principled Benchmarking for Hypothesis Generation. arXiv preprint arXiv:2504.11524.

- **论文：** https://arxiv.org/abs/2504.11524
- **描述：** 用于假设生成方法系统性评估的基准框架

**BibTeX：**
```bibtex
@misc{liu2025hypobenchsystematicprincipledbenchmarking,
      title={HypoBench: Towards Systematic and Principled Benchmarking for Hypothesis Generation}, 
      author={Haokun Liu and Sicong Huang and Jingyu Hu and Yangqiaoyu Zhou and Chenhao Tan},
      year={2025},
      eprint={2504.11524},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={https://arxiv.org/abs/2504.11524}, 
}
```

### Literature Meets Data (2024)

Liu, H., Zhou, Y., Li, M., Yuan, C., & Tan, C. (2024). Literature Meets Data: A Synergistic Approach to Hypothesis Generation. arXiv preprint arXiv:2410.17309.

- **论文：** https://arxiv.org/abs/2410.17309
- **代码：** https://github.com/ChicagoHAI/hypothesis-generation
- **描述：** 引入 HypoRefine，并展示了基于文献和数据驱动假设生成的协同组合

**BibTeX：**
```bibtex
@misc{liu2024literaturemeetsdatasynergistic,
      title={Literature Meets Data: A Synergistic Approach to Hypothesis Generation}, 
      author={Haokun Liu and Yangqiaoyu Zhou and Mingxuan Li and Chenfei Yuan and Chenhao Tan},
      year={2024},
      eprint={2410.17309},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={https://arxiv.org/abs/2410.17309}, 
}
```

### Hypothesis Generation with Large Language Models (2024)

Zhou, Y., Liu, H., Srivastava, T., Mei, H., & Tan, C. (2024). Hypothesis Generation with Large Language Models. In Proceedings of EMNLP Workshop of NLP for Science.

- **论文：** https://aclanthology.org/2024.nlp4science-1.10/
- **描述：** 用于数据驱动假设生成的原始 HypoGeniC 框架

**BibTeX：**
```bibtex
@inproceedings{zhou2024hypothesisgenerationlargelanguage,
      title={Hypothesis Generation with Large Language Models}, 
      author={Yangqiaoyu Zhou and Haokun Liu and Tejes Srivastava and Hongyuan Mei and Chenhao Tan},
      booktitle = {Proceedings of EMNLP Workshop of NLP for Science},
      year={2024},
      url={https://aclanthology.org/2024.nlp4science-1.10/},
}
```

## 其他资源

### 官方链接

- **GitHub 仓库：** https://github.com/ChicagoHAI/hypothesis-generation
- **PyPI 包：** https://pypi.org/project/hypogenic/
- **许可证：** MIT 许可证
- **问题与支持：** https://github.com/ChicagoHAI/hypothesis-generation/issues

### 示例数据集

克隆这些仓库以获取即用示例：

```bash
# HypoGeniC 示例（仅数据驱动）
git clone https://github.com/ChicagoHAI/HypoGeniC-datasets.git ./data

# HypoRefine/Union 示例（文献+数据）
git clone https://github.com/ChicagoHAI/Hypothesis-agent-datasets.git ./data
```

### 社区与贡献

- **贡献者：** 7+ 活跃贡献者
- **星标：** GitHub 上 89+ 星
- **主题：** research-tool, interpretability, hypothesis-generation, scientific-discovery, llm-application

如有贡献或问题，请访问 GitHub 仓库并查看问题页面。

## 本地资源

### references/

`config_template.yaml` - 包含所有必需提示模板和参数的完整示例配置文件。包括：
- 任务配置的完整 YAML 结构
- 所有方法的示例提示模板
- 占位符变量文档
- 基于角色的提示示例

### scripts/

脚本目录可用于：
- 自定义数据准备工具
- 格式转换工具
- 分析和评估脚本
- 与外部工具的集成

### assets/

资产目录可用于：
- 示例数据集和模板
- 示例假设库
- 可视化输出
- 文档补充
