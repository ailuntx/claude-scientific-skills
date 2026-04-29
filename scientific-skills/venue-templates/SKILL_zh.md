---
name: 会议模板
description: 获取主要科学出版场所（Nature、Science、PLOS、IEEE、ACM）、学术会议（NeurIPS、ICML、CVPR、CHI）、研究海报以及基金申请书（NSF、NIH、DOE、DARPA）的全面LaTeX模板、格式要求和投稿指南。当需要准备期刊投稿、会议论文、研究海报或基金申请书，并需要特定场所的格式要求和模板时，应使用此技能。
allowed-tools: Read Write Edit Bash
license: MIT许可证
metadata:
    skill-author: K-Dense Inc.
---

# 会议模板

## 概述

获取主要科学出版场所、学术会议、研究海报和基金申请书的全面LaTeX模板、格式要求和投稿指南。该技能为跨学科的成功学术投稿提供现成的模板和详细规范。

在准备期刊投稿、会议论文、研究海报或基金申请书手稿，并需要特定场所的格式要求和模板时，使用此技能。

## 何时使用此技能

此技能应在以下情况下使用：
- 准备向特定期刊（Nature、Science、PLOS、IEEE等）提交手稿
- 撰写具有特定格式要求的会议论文（NeurIPS、ICML、CHI等）
- 为会议创建学术研究海报
- 为联邦机构（NSF、NIH、DOE、DARPA）或私人基金会起草基金申请书
- 检查目标场所的格式要求和页数限制
- 使用作者信息和项目细节自定义模板
- 验证文档是否符合场所规范

## 使用科学示意图增强视觉效果

**使用此技能创建文档时，请始终考虑添加科学图表和示意图以增强视觉传达。**

如果您的文档尚未包含示意图或图表：
- 请使用 **scientific-schematics** 技能生成由人工智能驱动的出版级图表
- 只需用自然语言描述您想要的图表
- Nano Banana Pro 将自动生成、审查和优化示意图

**对于新文档：** 默认应生成科学示意图，以直观地表示文本中描述的关键概念、工作流程、架构或关系。

**如何生成示意图：**
```bash
python scripts/generate_schematic.py "您的图表描述" -o figures/output.png
```

人工智能将自动：
- 创建具有正确格式的出版级图像
- 通过多次迭代进行审查和优化
- 确保可访问性（色盲友好、高对比度）
- 将输出保存到 figures/ 目录

**何时添加示意图：**
- 论文的方法流程图
- 概念框架图
- 系统架构图
- 数据流图
- 实验设计可视化
- 研究工作流图
- 任何受益于可视化的复杂概念

有关创建示意图的详细指导，请参考 scientific-schematics 技能文档。

---

## 核心能力

### 1. 期刊文章模板

获取50多种主要科学期刊（跨学科）的LaTeX模板和格式指南：

**Nature 系列**：
- Nature、Nature Methods、Nature Biotechnology、Nature Machine Intelligence
- Nature Communications、Nature Protocols
- Scientific Reports

**Science 系列**：
- Science、Science Advances、Science Translational Medicine
- Science Immunology、Science Robotics

**PLOS（公共科学图书馆）**：
- PLOS ONE、PLOS Biology、PLOS Computational Biology
- PLOS Medicine、PLOS Genetics

**Cell Press**：
- Cell、Neuron、Immunity、Cell Reports
- Molecular Cell、Developmental Cell

**IEEE 出版物**：
- IEEE Transactions（多个学科）
- IEEE Access、IEEE Journal 模板

**ACM 出版物**：
- ACM Transactions、Communications of the ACM
- ACM 会议论文集

**其他主要出版商**：
- Springer 期刊（多个学科）
- Elsevier 期刊（自定义模板）
- Wiley 期刊
- BMC 期刊
- Frontiers 期刊

### 2. 会议论文模板

针对主要学术会议的特定会议模板，格式正确：

**机器学习和人工智能**：
- NeurIPS（神经信息处理系统大会）
- ICML（国际机器学习大会）
- ICLR（国际学习表征大会）
- CVPR（计算机视觉与模式识别大会）
- AAAI（人工智能促进协会）

**计算机科学**：
- ACM CHI（人机交互）
- SIGKDD（知识发现与数据挖掘）
- EMNLP（自然语言处理经验方法）
- SIGIR（信息检索）
- USENIX 会议

**生物学与生物信息学**：
- ISMB（分子生物学智能系统会议）
- RECOMB（计算分子生物学研究会议）
- PSB（生物计算太平洋研讨会）

**工程学**：
- IEEE 会议模板（多个学科）
- ASME、AIAA 会议

### 3. 研究海报模板

用于会议展示的学术海报模板：

**标准格式**：
- A0（841 × 1189 毫米 / 33.1 × 46.8 英寸）
- A1（594 × 841 毫米 / 23.4 × 33.1 英寸）
- 36" × 48"（914 × 1219 毫米）——美国常见尺寸
- 42" × 56"（1067 × 1422 毫米）
- 48" × 36"（横向布局）

**模板包**：
- **beamerposter**：经典学术海报模板
- **tikzposter**：现代、色彩丰富的海报设计
- **baposter**：结构化多栏布局

**设计特点**：
- 适合远距离阅读的最佳字体大小
- 配色方案（色盲安全调色板）
- 网格布局和列结构
- 用于补充材料的二维码集成

### 4. 基金申请书模板

主要资助机构的模板和格式要求：

**NSF（美国国家科学基金会）**：
- 完整申请书模板（15页项目描述）
- 项目摘要（1页：概述、智力价值、广泛影响）
- 预算和预算说明
- 个人简历（3页限制）
- 设施、设备和其他资源
- 数据管理计划

**NIH（美国国立卫生研究院）**：
- R01 研究资助（多年期）
- R21 探索/开发资助
- K 奖（职业发展）
- 具体目标页（1页，最关键部分）
- 研究策略（重要性、创新性、方法）
- 个人简历（5页限制）

**DOE（美国能源部）**：
- 科学办公室申请书
- ARPA-E 模板
- 技术准备水平（TRL）描述
- 商业化和影响部分

**DARPA（美国国防高级研究计划局）**：
- BAA（广泛机构公告）响应
- Heilmeier 问答框架
- 技术方法和里程碑
- 转化计划

**私人基金会**：
- 盖茨基金会
- 惠康基金会
- 霍华德·休斯医学研究所（HHMI）
- 陈·扎克伯格倡议（CZI）

## 工作流程：查找和使用模板

### 第一步：确定目标场所

确定具体的出版场所、会议或资助机构：

```
示例查询：
- "我需要向Nature投稿"
- "NeurIPS 2025的要求是什么？"
- "给我看NSF提案的格式"
- "我要为ISMB制作海报"
```

### 第二步：查询模板和要求

获取特定场所的模板和格式指南：

**对于期刊**：
```bash
# 加载期刊格式要求
参考：references/journals_formatting.md
搜索："Nature" 或具体期刊名称

# 获取模板
模板：assets/journals/nature_article.tex
```

**对于会议**：
```bash
# 加载会议格式
参考：references/conferences_formatting.md
搜索："NeurIPS" 或具体会议

# 获取模板
模板：assets/journals/neurips_article.tex
```

**对于海报**：
```bash
# 加载海报指南
参考：references/posters_guidelines.md

# 获取模板
模板：assets/posters/beamerposter_academic.tex
```

**对于基金申请**：
```bash
# 加载基金要求
参考：references/grants_requirements.md
搜索："NSF" 或具体机构

# 获取模板
模板：assets/grants/nsf_proposal_template.tex
```

### 第三步：检查格式要求

在自定义之前检查关键规范：

**需要验证的关键要求**：
- 页数限制（因场所而异）
- 字体大小和系列
- 边距规范
- 行距
- 引用格式（APA、Vancouver、Nature等）
- 图表要求
- 文件格式（PDF、Word、LaTeX源码）
- 匿名化（用于双盲评审）
- 补充材料限制

### 第四步：自定义模板

使用辅助脚本或手动自定义：

**选项1：辅助脚本（推荐）**：
```bash
python scripts/customize_template.py \
  --template assets/journals/nature_article.tex \
  --title "您的论文标题" \
  --authors "第一作者, 第二作者" \
  --affiliations "大学名称" \
  --output my_nature_paper.tex
```

**选项2：手动编辑**：
- 打开模板文件
- 替换占位文本（以注释标记）
- 填写标题、作者、单位、摘要
- 向各个部分添加您的内容

### 第五步：验证格式

检查是否符合场所要求：

```bash
python scripts/validate_format.py \
  --file my_paper.pdf \
  --venue "Nature" \
  --check-all
```

**验证检查**：
- 页数在限制范围内
- 字体大小正确
- 边距符合规范
- 参考文献格式正确
- 图表满足分辨率要求

### 第六步：编译和审查

编译 LaTeX 并审查输出：

```bash
# 编译 LaTeX
pdflatex my_paper.tex
bibtex my_paper
pdflatex my_paper.tex
pdflatex my_paper.tex

# 或使用 latexmk 自动编译
latexmk -pdf my_paper.tex
```

审查清单：
- [ ] 所有部分存在且格式正确
- [ ] 引用正确渲染
- [ ] 图表显示并带有正确标题
- [ ] 页数在限制范围内
- [ ] 遵循作者指南
- [ ] 已准备补充材料（如果需要）

## 与其他技能的集成

此技能与其他科学技能无缝协作：

### 科学写作
- 使用 **scientific-writing** 技能获取内容指导（IMRaD结构、清晰性、精确性）
- 应用此技能的特定场所模板进行格式调整
- 结合使用以完成手稿准备

### 文献综述
- 使用 **literature-review** 技能进行系统性文献搜索和综合
- 应用场所要求的适当引用格式
- 根据模板规范格式化参考文献

### 同行评审
- 使用 **peer-review** 技能评估手稿质量
- 使用此技能验证格式合规性
- 确保遵守报告指南（CONSORT、STROBE等）

### 研究基金
- 与 **research-grants** 技能交叉引用以获取内容策略
- 使用此技能获取特定机构的模板和格式
- 结合使用以全面准备基金申请书

### LaTeX 海报
- 此技能提供与场所无关的海报模板
- 用于特定会议的海报要求
- 与可视化技能集成以创建图形

## 模板类别

### 按文档类型

| 类别 | 模板数量 | 常见场所 |
|------|----------|----------|
| **期刊文章** | 30+ | Nature、Science、PLOS、IEEE、ACM、Cell Press |
| **会议论文** | 20+ | NeurIPS、ICML、CVPR、CHI、ISMB |
| **研究海报** | 10+ | A0、A1、36×48、多种包 |
| **基金申请书** | 15+ | NSF、NIH、DOE、DARPA、基金会 |

### 按学科

| 学科 | 支持的场所 |
|------|------------|
| **生命科学** | Nature、Cell Press、PLOS、ISMB、RECOMB |
| **物理科学** | Science、Physical Review、ACS、APS |
| **工程学** | IEEE、ASME、AIAA、ACM |
| **计算机科学** | ACM、IEEE、NeurIPS、ICML、ICLR |
| **医学** | NEJM、Lancet、JAMA、BMJ |
| **跨学科** | PNAS、Nature Communications、Science Advances |

## 辅助脚本

### query_template.py

按场所名称、类型或关键词搜索和获取模板：

```bash
# 查找特定期刊的模板
python scripts/query_template.py --venue "Nature" --type "article"

# 按关键词搜索
python scripts/query_template.py --keyword "machine learning"

# 列出所有可用模板
python scripts/query_template.py --list-all

# 获取场所要求
python scripts/query_template.py --venue "NeurIPS" --requirements
```

### customize_template.py

使用作者和项目信息自定义模板：

```bash
# 基本自定义
python scripts/customize_template.py \
  --template assets/journals/nature_article.tex \
  --output my_paper.tex

# 带作者信息
python scripts/customize_template.py \
  --template assets/journals/nature_article.tex \
  --title "蛋白质折叠的新方法" \
  --authors "张三, 李四, 王五" \
  --affiliations "MIT, 斯坦福, 哈佛" \
  --email "[email protected]" \
  --output my_paper.tex

# 交互模式
python scripts/customize_template.py --interactive
```

### validate_format.py

检查文档是否符合场所要求：

```bash
# 验证已编译的 PDF
python scripts/validate_format.py \
  --file my_paper.pdf \
  --venue "Nature" \
  --check-all

# 检查特定方面
python scripts/validate_format.py \
  --file my_paper.pdf \
  --venue "NeurIPS" \
  --check page-count,margins,fonts

# 生成验证报告
python scripts/validate_format.py \
  --file my_paper.pdf \
  --venue "Science" \
  --report validation_report.txt
```

## 最佳实践

### 模板选择
1. **验证时效性**：检查模板日期并与最新作者指南比较
2. **检查官方来源**：许多期刊提供官方 LaTeX 类
3. **测试编译**：在添加内容之前编译模板
4. **阅读注释**：模板包含有用的内联注释

### 自定义
1. **保留结构**：不要删除必需部分或包
2. **遵循占位符**：系统地替换标记的占位文本
3. **保持格式**：不要覆盖场所特定格式
4. **保留备份**：在自定义之前保存原始模板

### 合规性
1. **检查页数限制**：在最终提交前验证
2. **验证引用**：使用场所的正确引用格式
3. **测试图表**：确保图表满足分辨率要求
4. **审查匿名化**：如果需要，移除标识信息

### 提交
1. **遵循说明**：阅读完整的作者指南
2. **包含所有文件**：LaTeX 源码、图表、参考文献
3. **正确生成**：使用推荐的编译方法
4. **检查输出**：验证 PDF 符合预期

## 常见格式要求

### 页数限制（典型）

| 场所类型 | 典型限制 | 注释 |
|----------|----------|------|
| **Nature 文章** | 5页 | 约3000字，不含参考文献 |
| **Science 报告** | 5页 | 图表计入限制 |
| **PLOS ONE** | 无限制 | 无限长度 |
| **NeurIPS** | 8页 | 无限参考文献/附录 |
| **ICML** | 8页 | 无限参考文献/附录 |
| **NSF 申请书** | 15页 | 仅项目描述 |
| **NIH R01** | 12页 | 研究策略 |

### 按场所的引用格式

| 场所 | 引用格式 | 样式 |
|------|----------|------|
| **Nature** | 编号（上标） | Nature 样式 |
| **Science** | 编号（上标） | Science 样式 |
| **PLOS** | 编号（括号） | Vancouver 样式 |
| **Cell Press** | 作者-年份 | Cell 样式 |
| **ACM** | 编号 | ACM 样式 |
| **IEEE** | 编号（括号） | IEEE 样式 |
| **APA 期刊** | 作者-年份 | APA 第7版 |

### 图表要求

| 场所 | 分辨率 | 格式 | 颜色 |
|------|--------|------|------|
| **Nature** | 300+ dpi | TIFF、EPS、PDF | RGB 或 CMYK |
| **Science** | 300+ dpi | TIFF、PDF | RGB |
| **PLOS** | 300-600 dpi | TIFF、EPS | RGB |
| **IEEE** | 300+ dpi | EPS、PDF | RGB 或灰度 |

## 写作风格指南

除了格式之外，此技能还提供全面的**写作风格指南**，描述论文在不同场所应如何**呈现**——而不仅仅是外观。

### 为什么风格很重要

同一项研究为 Nature 写作与为 NeurIPS 写作会有很大不同：
- **Nature/Science**：非专业人士可读，故事驱动，广泛意义
- **Cell Press**：机制深度，全面数据，需要图形摘要
- **医学期刊**：以患者为中心，证据分级，结构化摘要
- **机器学习会议**：贡献要点、消融研究、可重复性重点
- **计算机科学会议**：领域特定惯例，不同评估标准

### 可用的风格指南

| 指南 | 涵盖内容 | 关键主题 |
|------|----------|----------|
| `venue_writing_styles.md` | 总览 | 风格谱系、快速参考 |
| `nature_science_style.md` | Nature、Science、PNAS | 可读性、故事性、广泛影响 |
| `cell_press_style.md` | Cell、Neuron、Immunity | 图形摘要、eTOC、要点 |
| `medical_journal_styles.md` | NEJM、Lancet、JAMA、BMJ | 结构化摘要、证据语言 |
| `ml_conference_style.md` | NeurIPS、ICML、ICLR、CVPR | 贡献要点、消融研究 |
| `cs_conference_style.md` | ACL、EMNLP、CHI、SIGKDD | 领域特定惯例 |
| `reviewer_expectations.md` | 所有场所 | 审稿人关注点、回复技巧 |

### 写作示例

具体示例可在 `assets/examples/` 中找到：
- `nature_abstract_examples.md`：高影响力期刊的流畅段落摘要
- `neurips_introduction_example.md`：带有贡献要点的机器学习会议引言
- `cell_summary_example.md`：Cell Press 的 Summary、Highlights、eTOC 格式
- `medical_structured_abstract.md`：NEJM、Lancet、JAMA 的结构化格式

### 工作流程：适应场所

1. **确定目标场所**并加载相应的风格指南
2. **审查写作惯例**：语气、口吻、摘要格式、结构
3. **查看示例**以获取具体部分指导
4. **审查期望**：该场所的审稿人优先考虑什么？
5. **应用格式**：使用 `assets/` 中的 LaTeX 模板

---

## 资源

### 捆绑资源

**写作风格指南**（在 `references/` 中）：
- `venue_writing_styles.md`：总风格概览和比较
- `nature_science_style.md`：Nature/Science 写作惯例
- `cell_press_style.md`：Cell Press 期刊风格
- `medical_journal_styles.md`：医学期刊写作指南
- `ml_conference_style.md`：机器学习会议写作惯例
- `cs_conference_style.md`：计算机科学会议写作指南
- `reviewer_expectations.md`：不同场所审稿人关注点

**格式要求**（在 `references/` 中）：
- `journals_formatting.md`：全面的期刊格式要求
- `conferences_formatting.md`：会议论文规范
- `posters_guidelines.md`：研究海报设计和尺寸
- `grants_requirements.md`：按机构的基金申请书要求

**写作示例**（在 `assets/examples/` 中）：
- `nature_abstract_examples.md`：高影响力期刊摘要示例
- `neurips_introduction_example.md`：机器学习会议引言格式
- `cell_summary_example.md`：Cell Press Summary/Highlights/eTOC
- `medical_structured_abstract.md`：NEJM/Lancet/JAMA 摘要格式

**模板**（在 `assets/` 中）：
- `journals/`：期刊文章 LaTeX 模板
- `posters/`：研究海报模板
- `grants/`：基金申请书模板

**脚本**（在 `scripts/` 中）：
- `query_template.py`：搜索和获取模板
- `customize_template.py`：使用作者信息自定义
- `validate_format.py`：检查是否符合要求

### 外部资源

**期刊作者指南**：
- Nature：https://www.nature.com/nature/for-authors
- Science：https://www.science.org/content/page/instructions-authors
- PLOS：https://plos.org/resources/for-authors/
- Cell Press：https://www.cell.com/author-guidelines

**会议格式**：
- NeurIPS：https://neurips.cc/Conferences/2024/CallForPapers
- ICML：https://icml.cc/Conferences/2024/StyleAuthorInstructions
- CVPR：https://cvpr.thecvf.com/Conferences/2024/AuthorGuidelines

**基金资源**：
- NSF PAPPG：https://www.nsf.gov/publications/pub_summ.jsp?ods_key=pappg
- NIH Forms：https://grants.nih.gov/grants/how-to-apply-application-guide.html
- DOE Funding：https://science.osti.gov/grants

## 使用示例

### 示例1：Nature 文章投稿

```markdown
用户："我需要准备一份手稿提交给 Nature"

响应：
1. 加载 Nature 格式要求（references/journals_formatting.md）
2. 获取 Nature 模板（assets/journals/nature_article.tex）
3. 关键要求：
   - 长度：约3000字，约5页
   - 格式：单栏，12pt 字体
   - 引用：编号上标（Nature 样式）
   - 图表：300+ dpi，RGB 颜色
4. 使用您的标题、作者、单位自定义模板
5. 编译并根据要求验证
```

### 示例2：NeurIPS 会议论文

```markdown
用户："NeurIPS 的格式要求是什么？"

响应：
来自 references/conferences_formatting.md：
- 页数限制：8页正文 + 无限参考文献/附录
- 格式：双栏，10pt 字体
- 模板：neurips_2024.sty（官方样式文件）
- 匿名化：初次提交时需要
- 引用：编号（括号）
- 图表：高分辨率，建议色盲安全
```

### 示例3：NSF 基金申请书

```markdown
用户："帮我格式化一份 NSF 基金申请书"

响应：
NSF 要求（来自 references/grants_requirements.md）：
- 项目描述：最多15页
- 项目摘要：1页（概述、智力价值、广泛影响）
- 预算说明：3-5页
- 个人简历：每位高级人员3页
- 字体：最小10pt，Times Roman 或类似字体
- 边距：四面各1英寸
模板：assets/grants/nsf_proposal_template.tex
```

### 示例4：会议海报

```markdown
用户："我需要为 ISMB 制作一张海报"

响应：
ISMB 海报规格：
- 尺寸：通常为 A0 竖向（33.1 × 46.8 英寸）
- 推荐模板：beamerposter 或 tikzposter
- 字体大小：标题60-85pt，标题36-48pt，正文24-32pt
- 包含：论文/补充材料的二维码
可用模板：
- assets/posters/beamerposter_academic.tex
```

## 更新与维护

**模板时效性**：
- 模板每年更新，或当场所发布新指南时更新
- 最后更新：2024年
- 请查看官方场所网站获取最新要求

**报告问题**：
- 模板编译错误
- 过时的格式要求
- 缺少场所模板
- 不正确的规范

## 总结

venue-templates 技能提供以下方面的全面访问：

1. **跨学科的50多个出版场所模板**
2. **期刊、会议、海报、基金申请书的详细格式要求**
3. **用于模板发现、自定义和验证的辅助脚本**
4. **与其他科学写作技能的集成**
5. **成功学术投稿的最佳实践**

每当你需要特定场所的格式指导或学术出版模板时，请使用此技能。
