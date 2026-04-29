# 科学代理技能

> **🔔 Claude 科学技能现已更名为科学代理技能。** 相同的技能，更广泛的兼容性 — 现在适用于任何支持开放 [Agent Skills](https://agentskills.io/) 标准的 AI 代理，不仅仅是 Claude。

> **新发布：[K-Dense BYOK](https://github.com/K-Dense-AI/k-dense-byok)** — 一款免费、开源的 AI 合作科学家，在你的桌面上运行，由科学代理技能驱动。带上你自己的 API 密钥，从 40+ 个模型中选择，并获得包含网页搜索、文件处理、100+ 个科学数据库以及本仓库中全部 133 项技能的完整研究工作空间。你的数据保存在你的计算机上，你也可以通过 [Modal](https://modal.com/) 选择扩展到云计算以处理繁重的工作负载。[从这里开始。](https://github.com/K-Dense-AI/k-dense-byok)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE.md)
[![Skills](https://img.shields.io/badge/Skills-133-brightgreen.svg)](#whats-included)
[![Databases](https://img.shields.io/badge/Databases-100%2B-orange.svg)](#whats-included)
[![Agent Skills](https://img.shields.io/badge/Standard-Agent_Skills-blueviolet.svg)](https://agentskills.io/)
[![Works with](https://img.shields.io/badge/Works_with-Cursor_|_Claude_Code_|_Codex-blue.svg)](#getting-started)
[![X](https://img.shields.io/badge/Follow_on_X-%40k__dense__ai-000000?logo=x)](https://x.com/k_dense_ai)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-K--Dense_Inc.-0A66C2?logo=linkedin)](https://www.linkedin.com/company/k-dense-inc)
[![YouTube](https://img.shields.io/badge/YouTube-K--Dense_Inc.-FF0000?logo=youtube)](https://www.youtube.com/@K-Dense-Inc)

一套全面的 **133 项即用型科学与研究技能**（涵盖癌症基因组学、药物-靶点结合、分子动力学、RNA速率、地理空间科学、时间序列预测、78+ 个科学数据库等），适用于任何支持开放 [Agent Skills](https://agentskills.io/) 标准的 AI 代理，由 [K-Dense](https://k-dense.ai) 创建。可与 **Cursor、Claude Code、Codex 等** 协同工作。将你的 AI 代理转变为能够执行跨越生物学、化学、医学等领域的复杂多步骤科学工作流的研究助手。

---

这些技能使你的 AI 代理能够无缝地使用跨多个科学领域的专业科学库、数据库和工具。虽然代理可以自行使用任何 Python 包或 API，但这些明确定义的技能提供了精心整理的文档和示例，使其在以下工作流中显著增强且更可靠：
- 🧬 生物信息学与基因组学 - 序列分析、单细胞 RNA-seq、基因调控网络、变异注释、系统发育分析
- 🧪 化学信息学与药物发现 - 分子性质预测、虚拟筛选、ADMET 分析、分子对接、先导化合物优化
- 🔬 蛋白质组学与质谱分析 - LC-MS/MS 处理、肽段鉴定、谱图匹配、蛋白定量
- 🏥 临床研究与精准医学 - 临床试验、药物基因组学、变异解读、药物安全性、临床决策支持、治疗方案规划
- 🧠 医疗 AI 与临床机器学习 - EHR 分析、生理信号处理、医学影像、临床预测模型
- 🖼️ 医学影像与数字病理学 - DICOM 处理、全切片图像分析、计算病理学、放射学工作流
- 🤖 机器学习与人工智能 - 深度学习、强化学习、时间序列分析、模型可解释性、贝叶斯方法
- 🔮 材料科学与化学 - 晶体结构分析、相图、代谢建模、计算化学
- 🌌 物理学与天文学 - 天文数据分析、坐标转换、宇宙学计算、符号数学、物理计算
- ⚙️ 工程与仿真 - 离散事件仿真、多目标优化、代谢工程、系统建模、流程优化
- 📊 数据分析与可视化 - 统计分析、网络分析、时间序列、出版级图表、大规模数据处理、探索性数据分析
- 🌍 地理空间科学与遥感 - 卫星图像处理、GIS 分析、空间统计、地形分析、地球观测的机器学习
- 🧪 实验室自动化 - 液体处理方案、实验室设备控制、工作流自动化、LIMS 集成
- 📚 科学传播 - 文献综述、同行评审、科学写作、文档处理、海报、幻灯片、示意图、引文管理
- 🔬 多组学与系统生物学 - 多模态数据整合、通路分析、网络生物学、系统级见解
- 🧬 蛋白质工程与设计 - 蛋白质语言模型、结构预测、序列设计、功能注释
- 🎓 研究方法论 - 假设生成、科学头脑风暴、批判性思维、基金申请书撰写、学者评估

**将你的 AI 编码代理转变为桌面上的“AI 科学家”！**

> ⭐ **如果你觉得这个仓库有用**，请考虑给它一颗星！这有助于其他人发现这些工具，并激励我们继续维护和扩展这个集合。

> 🎬 **刚接触科学代理技能？** 观看我们的 [科学代理技能入门](https://youtu.be/ZxbnDaD_FVg) 视频，快速了解。

---

## 📦 包含内容

本仓库提供 **133 项科学与研究技能**，分为以下类别：

- **100+ 科学与金融数据库** - 一个统一的数据库查询技能可直接访问 78 个公共数据库（PubChem、ChEMBL、UniProt、COSMIC、ClinicalTrials.gov、FRED、USPTO 等），还有针对 DepMap、Imaging Data Commons、PrimeKG 和美国财政部财政数据的专用技能。像 BioServices（约 40 个生物信息服务）、BioPython（通过 Entrez 访问 38 个 NCBI 子数据库）和 gget（20+ 基因组学数据库）等多数据库包进一步增加了覆盖范围
- **70+ 优化的 Python 包技能** - 针对 RDKit、Scanpy、PyTorch Lightning、scikit-learn、BioPython、pyzotero、BioServices、PennyLane、Qiskit、OpenMM、MDAnalysis、scVelo、TimesFM 等明确编写的技能，包含精心整理的文档、示例和最佳实践。注意：代理可以使用*任何* Python 包编写代码，不仅限于这些；这些技能仅为所列包提供更强、更可靠的性能
- **9 项科学集成技能** - 针对 Benchling、DNAnexus、LatchBio、OMERO、Protocols.io、Open Notebook 等明确编写的技能。同样，代理并不局限于此——任何可从 Python 访问的 API 或平台都不在话下；这些技能是经过优化和预先记录的路径
- **30+ 分析与传播工具** - 文献综述、科学写作、同行评审、文档处理、海报、幻灯片、示意图、信息图表、Mermaid 图表等
- **10+ 研究与临床工具** - 假设生成、基金申请书撰写、临床决策支持、治疗方案、法规遵从、场景分析

每项技能包含：
- ✅ 全面的文档 (`SKILL.md`)
- ✅ 实用的代码示例
- ✅ 用例和最佳实践
- ✅ 集成指南
- ✅ 参考资料

---

## 📋 目录

- [包含内容](#whats-included)
- [为何使用？](#why-use-this)
- [快速开始](#getting-started)
- [安全免责声明](#-security-disclaimer)
- [支持开源社区](#-support-the-open-source-community)
- [先决条件](#prerequisites)
- [快速示例](#quick-examples)
- [用例](#use-cases)
- [可用技能](#available-skills)
- [贡献](#contributing)
- [故障排除](#troubleshooting)
- [常见问题](#faq)
- [支持](#support)
- [加入我们的社区](#join-our-community)
- [引用](#citation)
- [许可证](#license)

---

## 🚀 为何使用？

### ⚡ **加速你的研究**
- **节省数天的工作** - 跳过 API 文档研究和集成设置
- **生产就绪代码** - 经过测试和验证的示例，遵循科学最佳实践
- **多步骤工作流** - 通过一个简单的提示执行复杂流程

### 🎯 **全面覆盖**
- **133 项技能** - 全面覆盖所有主要科学领域
- **100+ 数据库** - 通过数据库查询统一访问 78+ 个数据库，加上专用的数据访问技能以及 BioServices、BioPython 和 gget 等多数据库包
- **70+ 优化的 Python 包技能** - RDKit、Scanpy、PyTorch Lightning、scikit-learn、BioServices、PennyLane、Qiskit、OpenMM、scVelo、TimesFM 等（代理可以使用任何 Python 包；这些是预先记录且性能更高的路径）

### 🔧 **轻松集成**
- **简单设置** - 将技能复制到你的技能目录即可开始工作
- **自动发现** - 你的代理会自动找到并使用相关技能
- **文档完善** - 每项技能都包含示例、用例和最佳实践

### 🌟 **维护与支持**
- **定期更新** - 由 K-Dense 团队持续维护和扩展
- **社区驱动** - 开源，有活跃的社区贡献
- **企业就绪** - 对于高级需求，提供商业支持

---

## 🎯 快速开始

### 选项 1：npx（所有平台）

通过一条命令安装科学代理技能：

```bash
npx skills add K-Dense-AI/scientific-agent-skills
```

这是在 **所有平台** 上安装 Agent Skills 的官方标准方法，包括 **Claude Code**、**Claude Cowork**、**Codex**、**Gemini CLI**、**Cursor** 以及任何其他支持开放 [Agent Skills](https://agentskills.io/) 标准的代理。

### 选项 2：GitHub CLI (`gh skill`)

如果你使用 [GitHub CLI](https://cli.github.com/)（v2.90.0+），可以通过 [`gh skill`](https://github.blog/changelog/2026-04-16-manage-agent-skills-with-github-cli/) 安装技能：

```bash
# 交互式浏览并安装
gh skill install K-Dense-AI/scientific-agent-skills

# 直接安装特定技能
gh skill install K-Dense-AI/scientific-agent-skills scanpy

# 针对特定代理主机
gh skill install K-Dense-AI/scientific-agent-skills --agent cursor
gh skill install K-Dense-AI/scientific-agent-skills --agent claude-code
gh skill install K-Dense-AI/scientific-agent-skills --agent codex
gh skill install K-Dense-AI/scientific-agent-skills --agent gemini
```

`gh skill` 会自动安装到你的代理主机的正确目录，并记录来源元数据以确保供应链完整性。

#### 版本锁定

固定到特定的发布标签或提交 SHA 以实现可重现的安装：

```bash
# 锁定到发布标签
gh skill install K-Dense-AI/scientific-agent-skills --pin v1.0.0

# 锁定到提交 SHA
gh skill install K-Dense-AI/scientific-agent-skills --pin abc123def
```

#### 保持技能最新

```bash
# 交互式检查更新
gh skill update

# 更新所有已安装的技能
gh skill update --all
```

**这就行了！** 你的 AI 代理将自动发现这些技能，并在你的科学任务中使用它们。你也可以在提示中手动提及技能名称来调用任何技能。

---

## ⚠️ 安全免责声明

> **技能可以执行代码并影响你的编码代理的行为。请仔细审查你安装的内容。**

AI 技能功能强大——它们可以指示你的 AI 代理运行任意代码、安装包、发起网络请求以及修改系统上的文件。恶意或编写不当的技能有可能将你的编码代理引向有害行为。

我们非常重视安全性。所有贡献都经过审查流程，我们使用基于 LLM 的安全扫描（通过 [Cisco AI Defense Skill Scanner](https://github.com/cisco-ai-defense/skill-scanner)）对本仓库中的每项技能进行扫描。然而，作为一个小团队，随着社区贡献的不断增多，我们无法保证每项技能都经过了详尽的审查以排除所有可能的风险。

**最终，你有责任审查你安装的技能，并决定信任哪些。**

我们建议如下：

- **不要一次性安装所有技能。** 仅安装你实际工作所需的技能。虽然当 K-Dense 创建并维护每项技能时，安装整个集合是合理的，但现在仓库包含了大量我们可能没有彻底审查的社区贡献。
- **在安装前阅读 `SKILL.md`。** 每项技能的文档都描述了它的功能、使用的包以及连接的外部服务。如果某些内容看起来可疑，就不要安装。
- **检查贡献历史。** 由 K-Dense（`K-Dense-AI`）编写的技能已经过我们的内部审查流程。社区贡献的技能我们已尽力审查，但资源有限。
- **自行运行安全扫描器。** 在安装第三方技能之前，在本地进行扫描：
  ```bash
  uv pip install cisco-ai-skill-scanner
  skill-scanner scan /path/to/skill --use-behavioral
  ```
- **报告任何可疑内容。** 如果你发现某个技能看起来是恶意或表现出异常行为，请立即 [提交问题](https://github.com/K-Dense-AI/scientific-agent-skills/issues)，以便我们进行调查。

所有技能大约每周扫描一次，[SECURITY.md](SECURITY.md) 会更新最新的结果。我们会在安全问题出现时努力解决。

---

## ❤️ 支持开源社区

科学代理技能由 **50+ 个令人惊叹的开源项目** 提供支持，这些项目由全球敬业的开发者和研究社区维护。像 Biopython、Scanpy、RDKit、scikit-learn、PyTorch Lightning 等众多项目构成了这些技能的基础。

**如果你觉得本仓库有价值，请考虑支持使这一切成为可能的项目：**

- ⭐ 在 GitHub 上 **Star 它们的仓库**
- 💰 通过 GitHub Sponsors 或 NumFOCUS **赞助维护者**
- 📝 在你的出版物中 **引用项目**
- 💻 **贡献** 代码、文档或错误报告

👉 **[查看要支持的完整项目列表](docs/open-source-sponsors.md)**

---

## ⚙️ 先决条件

- **Python**：3.11+（建议 3.12+ 以获得最佳兼容性）
- **uv**：Python 包管理器（安装技能依赖项所必需）
- **客户端**：任何支持 [Agent Skills](https://agentskills.io/) 标准的代理（Cursor、Claude Code、Gemini CLI、Codex 等）
- **系统**：macOS、Linux 或 Windows with WSL2
- **依赖项**：由各项技能自动处理（具体需求请查看 `SKILL.md` 文件）

### 安装 uv

这些技能使用 `uv` 作为 Python 依赖项的包管理器。请根据你的操作系统使用以下说明进行安装：

**macOS 和 Linux：**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows：**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**替代方法（通过 pip）：**
```bash
pip install uv
```

安装后，运行以下命令验证：
```bash
uv --version
```

更多安装选项和详细信息，请访问 [官方 uv 文档](https://docs.astral.sh/uv/)。

---

## 💡 快速示例

安装技能后，你可以要求你的 AI 代理执行复杂的多步骤科学工作流。下面是一些示例提示：

### 🧪 药物发现流程
**目标**：寻找治疗肺癌的新型 EGFR 抑制剂

**提示**：
```
尽可能使用你可用的技能。从 ChEMBL 查询 EGFR 抑制剂（IC50 < 50nM），用 RDKit 分析构效关系，
用 datamol 生成改进的类似物，用 DiffDock 针对 AlphaFold 的 EGFR 结构进行虚拟筛选，
搜索 PubMed 查找耐药机制，检查 COSMIC 的突变，并创建可视化图表和综合报告。
```

**使用的技能**：ChEMBL、RDKit、datamol、DiffDock、AlphaFold DB、PubMed、COSMIC、科学可视化

*需要云端 GPU 并最终获得出版级报告吗？[在 K-Dense Web 上免费运行。](https://k-dense.ai)*

---

### 🔬 单细胞 RNA-seq 分析
**目标**：结合公共数据进行 10X Genomics 数据的综合分析

**提示**：
```
尽可能使用你可用的技能。用 Scanpy 加载 10X 数据集，执行质量控制和双细胞去除，整合 Cellxgene Census 数据，
使用 NCBI Gene 标记识别细胞类型，用 PyDESeq2 运行差异表达分析，用 Arboreto 推断基因调控网络，
通过 Reactome/KEGG 富集通路，并用 Open Targets 识别治疗靶点。
```

**使用的技能**：Scanpy、Cellxgene Census、NCBI Gene、PyDESeq2、Arboreto、Reactome、KEGG、Open Targets

*想要零设置的云端执行和可共享的输出吗？[免费试用 K-Dense Web。](https://k-dense.ai)*

---

### 🧬 多组学生物标志物发现
**目标**：整合 RNA-seq、蛋白质组学和代谢组学以预测患者预后

**提示**：
```
尽可能使用你可用的技能。用 PyDESeq2 分析 RNA-seq，用 pyOpenMS 处理质谱数据，从 HMDB/Metabolomics Workbench 整合代谢物，
将蛋白质映射到通路（UniProt/KEGG），通过 STRING 查找相互作用，用 statsmodels 关联组学层，
用 scikit-learn 构建预测模型，并搜索 ClinicalTrials.gov 查找相关试验。
```

**使用的技能**：PyDESeq2、pyOpenMS、HMDB、Metabolomics Workbench、UniProt、KEGG、STRING、statsmodels、scikit-learn、ClinicalTrials.gov

*这个流程计算量很大。[在 K-Dense Web 上使用云端 GPU 运行，免费起步。](https://k-dense.ai)*

---

### 🎯 虚拟筛选活动
**目标**：发现蛋白质-蛋白质相互作用的变构调节剂

**提示**：
```
尽可能使用你可用的技能。检索 AlphaFold 结构，用 BioPython 识别相互作用界面，搜索 ZINC 数据库中的变构候选物
（MW 300-500，logP 2-4），用 RDKit 过滤，用 DiffDock 进行对接，用 DeepChem 排序，
查询 PubChem 的供应商，搜索 USPTO 专利，并用 MedChem/molfeat 优化先导化合物。
```

**使用的技能**：AlphaFold DB、BioPython、ZINC、RDKit、DiffDock、DeepChem、PubChem、USPTO、MedChem、molfeat

*跳过本地 GPU 瓶颈。[在 K-Dense Web 上免费进行虚拟筛选。](https://k-dense.ai)*

---

### 🏥 临床变异解读
**目标**：分析 VCF 文件以进行遗传性癌症风险评估

**提示**：
```
尽可能使用你可用的技能。用 pysam 解析 VCF，用 Ensembl VEP 注释变异，查询 ClinVar 的致病性，
检查 COSMIC 的癌症突变，从 NCBI Gene 检索基因信息，用 UniProt 分析蛋白质影响，
搜索 PubMed 的病例报告，查看 ClinPGx 的药物基因组学，用文档处理工具生成临床报告，
并在 ClinicalTrials.gov 上寻找匹配的试验。
```

**使用的技能**：pysam、Ensembl、ClinVar、COSMIC、NCBI Gene、UniProt、PubMed、ClinPGx、文档技能、ClinicalTrials.gov

*最终需要一份精美的临床报告，而不仅仅是代码吗？[K-Dense Web 可提供出版级的输出。免费试用。](https://k-dense.ai)*

---

### 🌐 系统生物学网络分析
**目标**：从 RNA-seq 数据分析基因调控网络

**提示**：
```
尽可能使用你可用的技能。查询 NCBI Gene 的注释，从 UniProt 检索序列，通过 STRING 识别相互作用，
映射到 Reactome/KEGG 通路，用 Torch Geometric 分析拓扑结构，用 Arboreto 重建 GRN，
用 Open Targets 评估成药性，用 PyMC 建模，可视化网络，并搜索 GEO 中的相似模式。
```

**使用的技能**：NCBI Gene、UniProt、STRING、Reactome、KEGG、Torch Geometric、Arboreto、Open Targets、PyMC、GEO

*想要端到端流程，可共享输出且无需设置吗？[免费试用 K-Dense Web。](https://k-dense.ai)*

> 📖 **想要更多示例？** 查看 [docs/examples.md](docs/examples.md) 获取全面的工作流示例和所有科学领域的详细用例。

---

## 🚀 想跳过设置，只做科学研究吗？

**你是否遇到这些情况？**

- 你花在配置环境上的时间比运行分析还多
- 你的工作流需要 GPU，但你的本地机器没有
- 你需要的是可共享的、出版级的图表或报告，而不仅仅是一个脚本
- 你想要立即运行一个复杂的多步骤流程，而无需先阅读包文档

如果是这样，**[K-Dense Web](https://k-dense.ai)** 就是为你而构建的。它是一个完整的 AI 合作科学家平台：包含本仓库中的所有内容，外加云端 GPU、200+ 项技能，以及可以直接用于论文或演示的输出。无需任何设置。

| 特性 | 本仓库 | K-Dense Web |
|---------|-----------|-------------|
| 科学技能 | 133 项技能 | **200+ 项技能**（独占访问） |
| 设置 | 手动安装 | **零设置，即刻使用** |
| 计算 | 你的机器 | **包含云端 GPU 和 HPC** |
| 工作流 | 提示和代码 | **端到端研究流程** |
| 输出 | 代码和分析 | **出版级图表、报告和论文** |
| 集成 | 本地工具 | **实验室系统、ELN 和云存储** |

> *“K-Dense Web 在一下午内将我从原始测序数据带到了草稿图表。过去需要三天环境设置和脚本编写的工作现在只需轻松完成。”*
> **计算生物学家，药物发现领域**
>
> **[试用 K-Dense Web](https://k-dense.ai)**

*[k-dense.ai](https://k-dense.ai) | [阅读完整比较](https://k-dense.ai/blog/k-dense-web-vs-scientific-agent-skills)*

---

## 🔬 用例

### 🧪 药物发现与药物化学
- **虚拟筛选**：针对蛋白质靶点从 PubChem/ZINC 筛选数百万化合物
- **先导化合物优化**：用 RDKit 分析构效关系，用 datamol 生成类似物
- **ADMET 预测**：用 DeepChem 预测吸收、分布、代谢、排泄和毒性
- **分子对接**：用 DiffDock 预测结合构象和亲和力
- **生物活性挖掘**：查询 ChEMBL 获取已知抑制剂并分析 SAR 模式

### 🧬 生物信息学与基因组学
- **序列分析**：用 BioPython 和 pysam 处理 DNA/RNA/蛋白质序列
- **单细胞分析**：用 Scanpy 分析 10X Genomics 数据，识别细胞类型，用 Arboreto 推断 GRN
- **变异注释**：用 Ensembl VEP 注释 VCF 文件，查询 ClinVar 的致病性
- **变异数据库管理**：用 TileDB-VCF 构建可扩展的 VCF 数据库，支持增量样本添加、高效群体规模查询和基因组变异数据的压缩存储
- **基因发现**：查询 NCBI Gene、UniProt 和 Ensembl 以获取全面的基因信息
- **网络分析**：通过 STRING 识别蛋白质-蛋白质相互作用，映射到通路（KEGG、Reactome）

### 🏥 临床研究与精准医学
- **临床试验**：搜索 ClinicalTrials.gov 的相关研究，分析资格标准
- **变异解读**：用 ClinVar、COSMIC 和 ClinPGx 注释变异以进行药物基因组学
- **药物安全**：查询 FDA 数据库获取不良事件、药物相互作用和召回信息
- **精准治疗**：将患者变异与靶向治疗和临床试验进行匹配

### 🔬 多组学与系统生物学
- **多组学整合**：结合 RNA-seq、蛋白质组学和代谢组学数据
- **通路分析**：富集 KEGG/Reactome 通路中的差异表达基因
- **网络生物学**：重建基因调控网络，识别 hub 基因
- **生物标志物发现**：整合多组学层以预测患者预后

### 📊 数据分析与可视化
- **统计分析**：执行假设检验、功效分析和实验设计
- **出版级图表**：用 matplotlib 和 seaborn 创建出版质量的可视化
- **网络可视化**：用 NetworkX 可视化生物网络
- **报告生成**：用文档技能生成全面的 PDF 报告

### 🧪 实验室自动化
- **方案设计**：为自动化液体处理创建 Opentrons 方案
- **LIMS 集成**：集成 Benchling 和 LabArchives 以进行数据管理
- **工作流自动化**：自动化多步骤实验室工作流

---

## 📚 可用技能

本仓库包含 **133 项科学与研究技能**，涵盖多个领域。每项技能都为使用科学库、数据库和工具提供了全面的文档、代码示例和最佳实践。

### 技能类别

> **注意：** 下面列出的 Python 包和集成技能是*明确定义的*技能——附有文档、示例和最佳实践，以获得更强、更可靠的性能。它们并非上限：代理可以安装和使用*任何* Python 包或调用*任何* API，即使没有专门的技能。所列出的技能只是让常见工作流更快、更可靠。

#### 🧬 **生物信息学与基因组学** (21+ 项技能)
- 序列分析：BioPython、pysam、scikit-bio、BioServices
- 单细胞分析：Scanpy、AnnData、scvi-tools、scVelo（RNA 速率）、Arboreto、Cellxgene Census
- 基因组工具：gget、geniml、gtars、deepTools、FlowIO、Polars-Bio、Zarr、TileDB-VCF
- 差异表达：PyDESeq2
- 系统发育学：ETE Toolkit、Phylogenetics (MAFFT、IQ-TREE 2、FastTree)

#### 🧪 **化学信息学与药物发现** (10+ 项技能)
- 分子操作：RDKit、Datamol、Molfeat
- 深度学习：DeepChem、TorchDrug
- 对接与筛选：DiffDock
- 分子动力学：OpenMM + MDAnalysis（MD 模拟与轨迹分析）
- 云量子化学：Rowan（pKa、对接、共折叠）
- 类药性：MedChem
- 基准测试：PyTDC

#### 🔬 **蛋白质组学与质谱分析** (2 项技能)
- 谱图处理：matchms、pyOpenMS

#### 🏥 **临床研究与精准医学** (8+ 项技能)
- 临床数据库：通过数据库查询（ClinicalTrials.gov、ClinVar、ClinPGx、COSMIC、FDA、cBioPortal、Monarch 等）
- 癌症基因组学：DepMap（癌症依赖性评分、药物敏感性）
- 癌症影像：Imaging Data Commons（通过 idc-index 获取 NCI 放射学与病理学数据集）
- 医疗 AI：PyHealth、NeuroKit2、临床决策支持
- 临床文档：临床报告、治疗方案

#### 🖼️ **医学影像与数字病理学** (3 项技能)
- DICOM 处理：pydicom
- 全切片成像：histolab、PathML

#### 🧠 **神经科学与电生理学** (1 项技能)
- 神经记录：Neuropixels-Analysis（细胞外 spikes、硅探针、spike sorting）

#### 🤖 **机器学习与人工智能** (16+ 项技能)
- 深度学习：PyTorch Lightning、Transformers、Stable Baselines3、PufferLib
- 经典机器学习：scikit-learn、scikit-survival、SHAP
- 时间序列：aeon、TimesFM（Google 的单变量预测零样本基础模型）
- 贝叶斯方法：PyMC
- 优化：PyMOO
- 图机器学习：Torch Geometric
- 降维：UMAP-learn
- 统计建模：statsmodels

#### 🔮 **材料科学、化学与物理学** (7 项技能)
- 材料：Pymatgen
- 代谢建模：COBRApy
- 天文学：Astropy
- 量子计算：Cirq、PennyLane、Qiskit、QuTiP

#### ⚙️ **工程与仿真** (4 项技能)
- 数值计算：MATLAB/Octave
- 计算流体力学：FluidSim
- 离散事件仿真：SimPy
- 符号数学：SymPy

#### 📊 **数据分析与可视化** (16+ 项技能)
- 可视化：Matplotlib、Seaborn、科学可视化
- 地理空间分析：GeoPandas、GeoMaster（遥感、GIS、卫星图像、空间机器学习、500+ 示例）
- 数据处理：Dask、Polars、Vaex
- 网络分析：NetworkX
- 文档处理：文档技能（PDF、DOCX、PPTX、XLSX）
- 信息图表：信息图表（AI 驱动的专业信息图表创建）
- 图表：Markdown & Mermaid 写作（基于文本的图表作为默认文档标准）
- 探索性数据分析：EDA 工作流
- 统计分析：统计分析工作流

#### 🧪 **实验室自动化** (4 项技能)
- 液体处理：PyLabRobot
- 云实验室：Ginkgo Cloud Lab（无细胞蛋白表达，通过自主 RAC 基础设施实现的荧光像素艺术）
- 方案管理：Protocols.io
- LIMS 集成：Benchling、LabArchives

#### 🔬 **多组学与系统生物学** (4+ 项技能)
- 通路分析：通过数据库查询（KEGG、Reactome、STRING）和 PrimeKG
- 多组学：HypoGeniC
- 数据管理：LaminDB

#### 🧬 **蛋白质工程与设计** (3 项技能)
- 蛋白质语言模型：ESM
- 糖基工程：Glycoengineering（N/O-糖基化预测、治疗性抗体优化）
- 云实验室平台：Adaptyv（自动化蛋白质测试和验证）

#### 📚 **科学传播** (20+ 项技能)
- 文献：论文查询（PubMed、PMC、bioRxiv、medRxiv、arXiv、OpenAlex、Crossref、Semantic Scholar、CORE、Unpaywall），文献综述
- 高级论文搜索：BGPT Paper Search（25+ 个结构化字段——方法、结果、样本量、质量评分——来自全文，不仅是摘要）
- 网络搜索：Parallel Web（带引用的综合摘要）
- 研究笔记本：Open Notebook（自托管的 NotebookLM 替代方案——PDF、视频、音频、网页；16+ AI 提供商；多发言人播客生成）
- 写作：科学写作、同行评审
- 文档处理：XLSX、MarkItDown、文档技能
- 出版：会议模板
- 演示：科学幻灯片、LaTeX 海报、PPTX 海报
- 图表：科学示意图、Markdown & Mermaid 写作
- 信息图表：信息图表（10 种类型、8 种风格、色盲安全调色板）
- 引文：引文管理
- 插图：生成图像（使用 FLUX.2 Pro 和 Gemini 3 Pro (Nano Banana Pro) 的 AI 图像生成）

#### 🔬 **科学数据库与数据访问** (5 项技能 → 总计 100+ 数据库)
> 一个统一的数据库查询技能提供对 78 个跨所有领域的公共数据库的直接 REST API 访问。专用技能涵盖专门的数据平台。像 BioServices（约 40 个生物信息服务）、BioPython（通过 Entrez 访问 38 个 NCBI 子数据库）和 gget（20+ 基因组学数据库）等多数据库包进一步增加了覆盖范围。
- 统一访问：数据库查询（78 个数据库涵盖化学、基因组学、临床、通路、专利、经济学等——PubChem、ChEMBL、UniProt、PDB、AlphaFold、KEGG、Reactome、STRING、ClinVar、COSMIC、ClinicalTrials.gov、FDA、FRED、USPTO、SEC EDGAR 等数十个）
- 癌症基因组学：DepMap（癌细胞系依赖性、药物敏感性、基因效应谱）
- 癌症影像：Imaging Data Commons（通过 idc-index 获取 NCI 放射学与病理学数据集）
- 知识图谱：PrimeKG（精准医学知识图谱——基因、药物、疾病、表型）
- 财政数据：美国财政部财政数据（国债、财政部报表、拍卖、汇率）

#### 🔧 **基础设施与平台** (7+ 项技能)
- 云计算：Modal
- GPU 加速：优化 GPU（CuPy、Numba CUDA、Warp、cuDF、cuML、cuGraph、KvikIO、cuCIM、cuxfilter、cuVS、cuSpatial、RAFT）
- 基因组学平台：DNAnexus、LatchBio
- 显微镜：OMERO
- 自动化：Opentrons
- 资源检测：获取可用资源

#### 🎓 **研究方法论与规划** (12+ 项技能)
- 构思：科学头脑风暴、假设生成
- 批判性分析：科学批判性思维、学者评估
- 场景分析：What-If Oracle（多分支可能性探索、风险分析、战略选择）
- 多视角审议：Consciousness Council（多元专家观点、反对派“魔鬼代言人”分析）
- 认知画像：DHDNA Profiler（从任何文本中提取思维模式和认知特征）
- 资助：研究资助
- 发现：研究查询、论文查询（10 个学术数据库）
- 市场分析：市场研究报告

#### ⚖️ **法规与标准** (1 项技能)
- 医疗器械标准：ISO 13485 认证

> 📖 **有关所有技能的完整详细信息**，请参阅 [docs/scientific-skills.md](docs/scientific-skills.md)

> 💡 **寻找实际示例？** 查看 [docs/examples.md](docs/examples.md) 获取所有科学领域的全面工作流示例。

---

## 🤝 贡献

我们欢迎为扩展和改进本科学技能仓库做出贡献！

### 贡献方式

✨ **添加新技能**
- 为其他科学包或数据库创建技能
- 添加科学平台和工具的集成

📚 **改进现有技能**
- 用更多示例和用例增强文档
- 添加新的工作流和参考资料
- 改进代码示例和脚本
- 修复错误或更新过时信息

🐛 **报告问题**
- 提交包含详细重现步骤的错误报告
- 提出改进建议或新功能

### 如何贡献

1. **Fork** 本仓库
2. **创建** 功能分支 (`git checkout -b feature/amazing-skill`)
3. **遵循** 现有的目录结构和文档模式
4. **确保** 所有新技能都包含全面的 `SKILL.md` 文件
5. **彻底测试** 你的示例和工作流
6. **提交** 你的更改 (`git commit -m '添加 amazing 技能'`)
7. **推送** 到你的分支 (`git push origin feature/amazing-skill`)
8. **提交** 一个合并请求，并清楚地描述你的更改

### 贡献指南

✅ **遵守 [Agent Skills 规范](https://agentskills.io/specification)** — 每项技能必须符合官方规范（有效的 `SKILL.md` frontmatter、命名约定、目录结构）  
✅ 保持与现有技能文档格式的一致性  
✅ 确保所有代码示例经过测试且功能正常  
✅ 在示例和工作流中遵循科学最佳实践  
✅ 在添加新功能时更新相关文档  
✅ 在代码中提供清晰的注释和文档字符串  
✅ 包含对官方文档的引用

### 安全扫描

本仓库中的所有技能使用 [Cisco AI Defense Skill Scanner](https://github.com/cisco-ai-defense/skill-scanner) 进行安全扫描，这是一个开源工具，可检测 Agent Skills 中的提示注入、数据窃取和恶意代码模式。

如果你要贡献新技能，我们建议在提交合并请求前在本地运行扫描器：

```bash
uv pip install cisco-ai-skill-scanner
skill-scanner scan /path/to/your/skill --use-behavioral
```

> **注意：** 干净的扫描结果可减少审查中的噪音，但不能保证技能完全没有风险。贡献的技能在合并前还会接受人工审查。

### 认可

贡献者将在我们的社区中获得认可，并可能出现在：
- 仓库的贡献者名单中
- 发布说明中的特别提及
- K-Dense 社区亮点

你的贡献有助于使科学计算更易用，并使研究人员能够更有效地利用 AI 工具！

### 支持开源

本项目建立在 50+ 个令人惊叹的开源项目之上。如果你发现这些技能有价值，请考虑 [支持我们所依赖的项目](docs/open-source-sponsors.md)。

---

## 🔧 故障排除

### 常见问题

**问题：技能未加载**
- 确认技能文件夹位于正确的目录中（参见 [快速开始](#getting-started)）
- 每个技能文件夹必须包含一个 `SKILL.md` 文件
- 复制技能后重启你的代理/IDE
- 在 Cursor 中，检查 Settings → Rules 以确认技能已被发现

**问题：缺少 Python 依赖项**
- 解决方案：查看特定 `SKILL.md` 文件以获取所需包
- 安装依赖项：`uv pip install package-name`

**问题：API 速率限制**
- 解决方案：许多数据库有速率限制。请查看特定数据库文档
- 考虑实施缓存或批量请求

**问题：认证错误**
- 解决方案：某些服务需要 API 密钥。查看 `SKILL.md` 以获取认证设置
- 验证你的凭据和权限

**问题：示例过时**
- 解决方案：通过 GitHub Issues 报告问题
- 查看官方包文档以获取更新后的语法

---

## ❓ 常见问题

### 一般问题

**问：这是免费使用的吗？**  
答：是的！本仓库采用 MIT 许可证。但是，每项技能在 `SKILL.md` 文件的 `license` 元数据字段中指定了自己的许可证——请务必查看并遵守这些条款。

**问：为什么所有技能都集中在一起，而不是分开的包？**  
答：我们认为 AI 时代的好科学本质上是跨学科的。将所有技能捆绑在一起，让你（和你的代理）可以轻松地跨领域整合——例如，在一个工作流中结合基因组学、化学信息学、临床数据和机器学习——而无需担心要安装或连接哪些单独的技能。

**问：我可以将其用于商业项目吗？**  
答：仓库本身采用 MIT 许可证，允许商业使用。但是，个别技能可能有不同的许可证——请查看每项技能 `SKILL.md` 文件中的 `license` 字段，以确保与你的预期用途相符。

**问：所有技能的许可证都一样吗？**  
答：不。每项技能在 `SKILL.md` 文件的 `license` 元数据字段中指定了自己的许可证。这些许可证可能与仓库的 MIT 许可证不同。用户有责任审查并遵守其所使用的每项技能的许可条款。

**问：更新频率如何？**  
答：我们会定期更新技能以反映包和 API 的最新版本。重大更新会在发布说明中公布。

**问：我可以将其用于其他 AI 模型吗？**  
答：这些技能遵循开放的 [Agent Skills](https://agentskills.io/) 标准，可与任何兼容的代理协同工作，包括 Cursor、Claude Code 和 Codex。

### 安装与设置

**问：我需要安装所有 Python 包吗？**  
答：不需要！只安装你需要的包。每项技能在 `SKILL.md` 文件中指定了其需求。

**问：如果某项技能不起作用怎么办？**  
答：首先查看 [故障排除](#troubleshooting) 部分。如果问题仍然存在，请在 GitHub 上提交包含详细重现步骤的问题。

**问：这些技能可以离线工作吗？**  
答：数据库技能需要互联网访问来查询 API。包技能一旦安装了 Python 依赖项就可以离线工作。

### 贡献

**问：我可以贡献自己的技能吗？**  
答：当然可以！我们欢迎贡献。请参阅 [贡献](#contributing) 部分了解指南和最佳实践。

**问：如何报告错误或建议功能？**  
答：在 GitHub 上打开一个问题并附上清晰的描述。对于错误，请包括重现步骤以及预期与实际行为。

---

## 💬 支持

需要帮助？以下是获取支持的途径：

- 📖 **文档**：查看相关的 `SKILL.md` 和 `references/` 文件夹
- 🐛 **错误报告**：[提交问题](https://github.com/K-Dense-AI/scientific-agent-skills/issues)
- 💡 **功能请求**：[提交功能请求](https://github.com/K-Dense-AI/scientific-agent-skills/issues/new)
- 💼 **企业支持**：联系 [K-Dense](https://k-dense.ai/) 获取商业支持
- 🌐 **社区**：[加入我们的 Slack](https://join.slack.com/t/k-densecommunity/shared_invite/zt-3iajtyls1-EwmkwIZk0g_o74311Tkf5g)

---

## 🎉 加入我们的社区！

**我们非常欢迎你的加入！** 🚀

与其他使用 AI 代理进行科学计算的科学家、研究人员和 AI 爱好者联系。分享你的发现，提出问题，获得项目帮助，并与社区合作！

🌟 **[加入我们的 Slack 社区](https://join.slack.com/t/k-densecommunity/shared_invite/zt-3iajtyls1-EwmkwIZk0g_o74311Tkf5g)** 🌟

无论你是新手还是高级用户，我们的社区都会为你提供支持。我们分享技巧，共同解决问题，展示酷炫项目，并讨论 AI 驱动的科学研究的最新发展。

**期待与你相见！** 💬

---

## 📖 引用

如果你在研究或项目中使用了科学代理技能，请按以下方式引用：

### BibTeX
```bibtex
@software{scientific_agent_skills_2026,
  author = {{K-Dense Inc.}},
  title = {Scientific Agent Skills: A Comprehensive Collection of Scientific Tools for AI Agents},
  year = {2026},
  url = {https://github.com/K-Dense-AI/scientific-agent-skills},
  note = {133 skills covering databases, packages, integrations, and analysis tools}
}
```

### APA
```
K-Dense Inc. (2026). Scientific Agent Skills: A comprehensive collection of scientific tools for AI agents [Computer software]. https://github.com/K-Dense-AI/scientific-agent-skills
```

### MLA
```
K-Dense Inc. Scientific Agent Skills: A Comprehensive Collection of Scientific Tools for AI Agents. 2026, github.com/K-Dense-AI/scientific-agent-skills.
```

### 纯文本
```
Scientific Agent Skills by K-Dense Inc. (2026)
Available at: https://github.com/K-Dense-AI/scientific-agent-skills
```

我们感谢在受益于这些技能的出版物、演示文稿或项目中给予认可！

---

## 📄 许可证

本项目采用 **MIT 许可证**。

**版权所有 © 2026 K-Dense Inc.** ([k-dense.ai](https://k-dense.ai/))

### 要点：
- ✅ **任何用途均免费**（商业和非商业）
- ✅ **开源** - 自由修改、分发和使用
- ✅ **宽松** - 对重复使用的限制最少
- ⚠️ **无担保** - “按原样”提供，不提供任何形式的担保

请参阅 [LICENSE.md](LICENSE.md) 获取完整条款。

### 个别技能许可证

> ⚠️ **重要**：每项技能在 `SKILL.md` 文件的 `license` 元数据字段中指定了自己的许可证。这些许可证可能与仓库的 MIT 许可证不同，并可能包含附加条款或限制。**用户有责任审查并遵守其所使用的每项技能的许可条款。**

## 星标历史

[![Star History Chart](https://api.star-history.com/svg?repos=K-Dense-AI/scientific-agent-skills&type=date&legend=top-left)](https://www.star-history.com/#K-Dense-AI/scientific-agent-skills&type=date&legend=top-left)
