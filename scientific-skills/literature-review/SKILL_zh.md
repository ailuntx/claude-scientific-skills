---
name: 文献综述
description: 使用多个学术数据库（PubMed、arXiv、bioRxiv、Semantic Scholar等）进行系统、全面的文献综述。当需要开展系统文献综述、荟萃分析、研究综合或在生物医学、科学和技术领域进行综合文献检索时，应使用此技能。可生成专业格式的Markdown文档和PDF，并提供多种引用格式（APA、Nature、Vancouver等）的已验证引文。
allowed-tools: Read Write Edit Bash
license: MIT 许可证
metadata:
    skill-author: K-Dense Inc.
---

# 文献综述

## 概述

按照严谨的学术方法开展系统、全面的文献综述。检索多个文献数据库，按主题综合研究成果，验证所有引文的准确性，并生成Markdown和PDF格式的专业输出文档。

此技能使用 **parallel-web 技能**（`parallel-cli search`）作为主要的网络搜索工具进行广泛的学术文献发现，并以专门的数据库访问技能（gget、bioservices、datacommons-client）作为补充。它还提供用于引文验证、结果聚合和文档生成的专用工具。

## 何时使用此技能

在以下情况下使用此技能：
- 为研究或发表开展系统文献综述
- 综合多个来源关于特定主题的当前知识
- 进行荟萃分析或范围综述
- 撰写研究论文或学位论文的文献综述部分
- 调查某一研究领域的最新进展
- 识别研究空白和未来方向
- 需要经过验证的引文和专业格式

## 科学示意图的视觉增强

**⚠️ 必须执行：每篇文献综述必须至少包含1-2张使用 scientific-schematics 技能生成的AI图形。**

这是非可选的。没有视觉元素的文献综述是不完整的。在定稿任何文档之前：
1. 至少生成一张示意图或图表（例如，系统综述的PRISMA流程图）
2. 对于综合性综述，最好使用2-3张图形（检索策略流程图、主题综合图、概念框架）

**如何生成图形：**
- 使用 **scientific-schematics** 技能生成AI驱动的出版级图表
- 用自然语言简单描述你想要的图表
- Nano Banana Pro 将自动生成、审查和完善示意图

**如何生成示意图：**
```bash
python scripts/generate_schematic.py "你的图表描述" -o figures/output.png
```

AI将自动：
- 创建具有适当格式的出版级图像
- 通过多次迭代进行审查和完善
- 确保无障碍（色盲友好、高对比度）
- 将输出保存到 figures/ 目录

**何时添加示意图：**
- 系统综述的PRISMA流程图
- 文献检索策略流程图
- 主题综合图
- 研究空白可视化图
- 引文网络图
- 概念框架插图
- 任何适合可视化的复杂概念

有关创建示意图的详细指南，请参阅 scientific-schematics 技能文档。

---

## 核心工作流程

文献综述遵循结构化的多阶段工作流程：

### 第一阶段：规划与范围界定

1. **定义研究问题**：使用PICO框架（Population, Intervention, Comparison, Outcome）进行临床/生物医学综述
   - 示例：“与标准治疗（C）相比，CRISPR-Cas9（I）治疗镰状细胞病（P）的疗效（O）如何？”

2. **确定范围与目标**：
   - 定义清晰、具体的研究问题
   - 确定综述类型（叙述性、系统性、范围性、荟萃分析）
   - 设定边界（时间段、地理范围、研究类型）

3. **制定检索策略**：
   - 从研究问题中识别2-4个主要概念
   - 列出每个概念的同义词、缩写和相关术语
   - 规划布尔运算符（AND、OR、NOT）以组合术语
   - 至少选择3个互补的数据库
   - **使用 parallel-web 技能（`parallel-cli search`）进行初步范围界定**，以便在正式数据库检索前快速了解整体情况

4. **设定纳入/排除标准**：
   - 日期范围（例如，最近10年：2015-2024）
   - 语言（通常为英文，或指定多语言）
   - 出版物类型（同行评审、预印本、综述）
   - 研究设计（RCT、观察性、体外等）
   - 清晰记录所有标准

### 第二阶段：系统文献检索

1. **多数据库检索**：

   选择适合该领域的数据库。**始终从 parallel-web 开始进行广泛的学术覆盖**，然后补充特定领域的数据库。

   **基于网络的学术检索（parallel-web 技能——从此处开始）：**
   - 使用 `parallel-cli search` 配合学术域过滤，进行广泛的学术覆盖
   - 运行两次检索：学术聚焦 + 通用，以捕获所有相关来源
   ```bash
   # 在学术来源中聚焦学术检索
   parallel-cli search "你的研究主题" -q "关键词1" -q "关键词2" \
     --json --max-results 10 --excerpt-max-chars-total 27000 \
     --include-domains "scholar.google.com,arxiv.org,pubmed.ncbi.nlm.nih.gov,semanticscholar.org,biorxiv.org,medrxiv.org,ncbi.nlm.nih.gov,nature.com,science.org,ieee.org,acm.org,springer.com,wiley.com,cell.com,pnas.org,nih.gov" \
     -o sources/litreview_<主题>-academic.json

   # 通用检索以补充来源
   parallel-cli search "你的研究主题" -q "关键词1" -q "关键词2" \
     --json --max-results 10 --excerpt-max-chars-total 27000 \
     -o sources/litreview_<主题>-general.json
   ```
   - 使用 `parallel-cli extract` 从检索结果中获取特定论文URL或PDF的完整内容
   ```bash
   parallel-cli extract "https://arxiv.org/abs/XXXX.XXXXX" --json
   ```

   **生物医学与生命科学：**
   - 使用 `gget` 技能：`gget search pubmed "检索词"` 检索 PubMed/PMC
   - 使用 `gget` 技能：`gget search biorxiv "检索词"` 检索预印本
   - 使用 `bioservices` 技能检索 ChEMBL、KEGG、UniProt 等

   **通用科学文献：**
   - 通过直接API检索arXiv（物理学、数学、计算机科学、定量生物学预印本）
   - 通过API检索Semantic Scholar（2亿+论文，跨学科）
   - 使用Google Scholar进行全面覆盖（手动或谨慎爬取）

   **专业数据库：**
   - 使用 `gget alphafold` 检索蛋白质结构
   - 使用 `gget cosmic` 检索癌症基因组学
   - 使用 `datacommons-client` 检索人口/统计数据
   - 根据领域需要酌情使用专业数据库

2. **记录检索参数**：
   ```markdown
   ## 检索策略

   ### 数据库：PubMed
   - **检索日期**：2024-10-25
   - **日期范围**：2015-01-01 至 2024-10-25
   - **检索字符串**：
     ```
     ("CRISPR"[Title] OR "Cas9"[Title])
     AND ("sickle cell"[MeSH] OR "SCD"[Title/Abstract])
     AND 2015:2024[Publication Date]
     ```
   - **结果**：247 篇文章
   ```

   对每个检索的数据库重复此步骤。

3. **导出并聚合结果**：
   - 从每个数据库以JSON格式导出结果
   - 将所有结果合并到一个文件中
   - 使用 `scripts/search_databases.py` 进行后处理：
     ```bash
     python search_databases.py combined_results.json \
       --deduplicate \
       --format markdown \
       --output aggregated_results.md
     ```

### 第三阶段：筛选与选择

1. **去重**：
   ```bash
   python search_databases.py results.json --deduplicate --output unique_results.json
   ```
   - 按DOI（主要）或标题（备用）去除重复项
   - 记录去除的重复项数量

2. **标题筛选**：
   - 根据纳入/排除标准审查所有标题
   - 排除明显不相关的研究
   - 记录此阶段排除的数量

3. **摘要筛选**：
   - 阅读剩余研究的摘要
   - 严格应用纳入/排除标准
   - 记录排除原因

4. **全文筛选**：
   - 获取剩余研究的全文
   - 根据所有标准进行详细审查
   - 记录具体的排除原因
   - 记录最终纳入研究的数量

5. **创建PRISMA流程图**：
   ```
   初始检索：n = X
   ├─ 去重后：n = Y
   ├─ 标题筛选后：n = Z
   ├─ 摘要筛选后：n = A
   └─ 纳入综述：n = B
   ```

### 第四阶段：数据提取与质量评估

1. **从每篇纳入研究中提取关键数据**：
   - 研究元数据（作者、年份、期刊、DOI）
   - 研究设计与方法
   - 样本量与人群特征
   - 主要发现与结果
   - 作者指出的局限性
   - 资助来源与利益冲突

2. **评估研究质量**：
   - **对于RCT**：使用Cochrane偏倚风险工具
   - **对于观察性研究**：使用纽卡斯尔-渥太华量表
   - **对于系统综述**：使用AMSTAR 2
   - 对每项研究评级：高、中、低或非常低质量
   - 考虑排除非常低质量的研究

3. **按主题组织**：
   - 识别研究中的3-5个主要主题
   - 按主题分组研究（一项研究可能出现在多个主题中）
   - 注意模式、共识和争议

### 第五阶段：综合与分析

1. **从模板创建综述文档**：
   ```bash
   cp assets/review_template.md my_literature_review.md
   ```

2. **撰写主题综合**（而非逐项总结）：
   - 按主题或研究问题组织结果部分
   - 综合每个主题中多项研究的发现
   - 比较和对比不同的方法和结果
   - 识别一致领域和争议点
   - 强调最强证据

   示例结构：
   ```markdown
   #### 3.3.1 主题：CRISPR递送方法

   已研究多种递送方法用于治疗性基因编辑。病毒载体（AAV）在15项研究¹⁻¹⁵中被使用，显示出高转导效率（65-85%），但引发了免疫原性问题³，⁷，¹²。相比之下，脂质纳米颗粒效率较低（40-60%），但安全性更佳¹⁶⁻²³。
   ```

3. **批判性分析**：
   - 评估跨研究的方法学优势和局限性
   - 评估证据的质量和一致性
   - 识别知识空白和方法学空白
   - 指出需要未来研究的领域

4. **撰写讨论**：
   - 在更广泛的背景下解释发现
   - 讨论临床、实践或研究意义
   - 承认综述本身的局限性
   - 如适用，与以往综述进行比较
   - 提出具体的未来研究方向

### 第六阶段：引文验证

**关键**：在最终提交前，必须验证所有引文的准确性。

1. **验证所有DOI**：
   ```bash
   python scripts/verify_citations.py my_literature_review.md
   ```

   此脚本：
   - 从文档中提取所有DOI
   - 验证每个DOI是否正确解析
   - 从CrossRef检索元数据
   - 生成验证报告
   - 输出格式正确的引文

2. **审查验证报告**：
   - 检查任何失败的DOI
   - 验证作者姓名、标题和出版详情是否匹配
   - 纠正原始文档中的任何错误
   - 重新运行验证，直到所有引文通过

3. **统一格式引文**：
   - 选择一种引文风格并在整个文档中保持一致（参见 `references/citation_styles.md`）
   - 常见风格：APA、Nature、Vancouver、Chicago、IEEE
   - 使用验证脚本输出来正确格式化引文
   - 确保文中引文与参考文献列表格式匹配

### 第七阶段：文档生成

1. **生成PDF**：
   ```bash
   python scripts/generate_pdf.py my_literature_review.md \
     --citation-style apa \
     --output my_review.pdf
   ```

   选项：
   - `--citation-style`: apa, nature, chicago, vancouver, ieee
   - `--no-toc`: 禁用目录
   - `--no-numbers`: 禁用章节编号
   - `--check-deps`: 检查是否安装了pandoc/xelatex

2. **审查最终输出**：
   - 检查PDF格式和布局
   - 验证所有部分是否都存在
   - 确保引文正确渲染
   - 检查图表/表格是否正确显示
   - 验证目录是否准确

3. **质量检查清单**：
   - [ ] 所有DOI已通过verify_citations.py验证
   - [ ] 引文格式一致
   - [ ] 包含PRISMA流程图（对于系统综述）
   - [ ] 检索方法已完全记录
   - [ ] 纳入/排除标准已明确说明
   - [ ] 结果按主题组织（而非逐项研究）
   - [ ] 质量评估已完成
   - [ ] 局限性已承认
   - [ ] 参考文献完整且准确
   - [ ] PDF生成无错误

## 特定数据库检索指南

### PubMed / PubMed Central

通过 `gget` 技能访问：
```bash
# 检索 PubMed
gget search pubmed "CRISPR基因编辑" -l 100

# 带过滤器检索
# 使用PubMed高级检索构建器构建复杂查询
# 然后通过gget或直接Entrez API执行
```

**检索技巧**：
- 使用MeSH术语：`"sickle cell disease"[MeSH]`
- 字段标签：`[Title]`, `[Title/Abstract]`, `[Author]`
- 日期过滤器：`2020:2024[Publication Date]`
- 布尔运算符：AND, OR, NOT
- 参见MeSH浏览器：https://meshb.nlm.nih.gov/search

### bioRxiv / medRxiv

通过 `gget` 技能访问：
```bash
gget search biorxiv "CRISPR镰状细胞" -l 50
```

**重要考虑因素**：
- 预印本未经同行评审
- 谨慎验证发现
- 检查预印本是否已发表（CrossRef）
- 注意预印本版本和日期

### arXiv

通过直接API或WebFetch访问：
```python
# 示例检索类别：
# q-bio.QM（定量方法）
# q-bio.GN（基因组学）
# q-bio.MN（分子网络）
# cs.LG（机器学习）
# stat.ML（机器学习统计）

# 检索格式：类别 AND 术语
search_query = "cat:q-bio.QM AND ti:\"单细胞测序\""
```

### Semantic Scholar

通过直接API访问（需要API密钥，或使用免费层级）：
- 涵盖所有领域的2亿+篇论文
- 非常适合跨学科检索
- 提供引文图和论文推荐
- 用于查找高影响力论文

### 专业生物医学数据库

使用适当的技能：
- **ChEMBL**：`bioservices` 技能用于化学活性
- **UniProt**：`gget` 或 `bioservices` 技能用于蛋白质信息
- **KEGG**：`bioservices` 技能用于通路和基因
- **COSMIC**：`gget` 技能用于癌症突变
- **AlphaFold**：`gget alphafold` 用于蛋白质结构
- **PDB**：`gget` 或直接API用于实验结构

### 引文链式扩展

通过引文网络扩展检索：

1. **前向引文**（引用关键论文的论文）：
   - 使用 `parallel-cli search` 查找引用特定工作的论文：
     ```bash
     parallel-cli search "引用[作者等人 年份] [论文标题]的论文" \
       -q "引用" -q "[关键作者]" \
       --json --max-results 10 --excerpt-max-chars-total 27000 \
       --include-domains "scholar.google.com,semanticscholar.org,arxiv.org,pubmed.ncbi.nlm.nih.gov" \
       -o sources/litreview_forward_citations.json
     ```
   - 使用Google Scholar的“被引用次数”
   - 使用Semantic Scholar或OpenAlex API
   - 识别建立在开创性工作基础上的较新研究

2. **后向引文**（关键论文的参考文献）：
   - 使用 `parallel-cli extract` 获取关键论文的全文并提取其参考文献列表：
     ```bash
     parallel-cli extract "https://doi.org/10.xxxx/yyyy" --json
     ```
   - 从纳入的研究中提取参考文献
   - 识别高引用的基础工作
   - 查找被多项纳入研究引用的论文

## 引文风格指南

详细格式化指南见 `references/citation_styles.md`。快速参考：

### APA（第7版）
- 文中：(Smith et al., 2023)
- 参考文献：Smith, J. D., Johnson, M. L., & Williams, K. R. (2023). Title. *Journal*, *22*(4), 301-318. https://doi.org/10.xxx/yyy

### Nature
- 文中：上标数字¹，²
- 参考文献：Smith, J. D., Johnson, M. L. & Williams, K. R. Title. *Nat. Rev. Drug Discov.* **22**, 301-318 (2023).

### Vancouver
- 文中：上标数字¹，²
- 参考文献：Smith JD, Johnson ML, Williams KR. Title. Nat Rev Drug Discov. 2023;22(4):301-18.

**在定稿前务必使用 verify_citations.py 验证引文。**

### 优先考虑高影响力论文（关键）

**始终优先选择来自知名作者和顶级刊物的高影响力、高被引论文。** 在文献综述中，质量比数量更重要。

#### 被引次数阈值

使用被引次数识别最具影响力的论文：

| 论文年龄 | 被引次数阈值 | 分类 |
|-----------|-------------------|----------------|
| 0-3年 | 20+ | 值得注意 |
| 0-3年 | 100+ | 高影响力 |
| 3-7年 | 100+ | 重要 |
| 3-7年 | 500+ | 里程碑论文 |
| 7年以上 | 500+ | 开创性工作 |
| 7年以上 | 1000+ | 基础性工作 |

#### 期刊与会议级别

优先选择来自更高级别刊物的论文：

- **第一级（始终优先）：** Nature, Science, Cell, NEJM, Lancet, JAMA, PNAS, Nature Medicine, Nature Biotechnology
- **第二级（强烈优先）：** 高影响力专业期刊（IF>10），顶级会议（NeurIPS, ICML用于机器学习/人工智能）
- **第三级（相关时纳入）：** 受尊敬的专业期刊（IF 5-10）
- **第四级（谨慎使用）：** 较低影响力的同行评审刊物

#### 作者声誉评估

优先选择：
- **资深研究人员**（在成熟领域h指数>40）
- **知名机构的领先研究团队**（哈佛、斯坦福、麻省理工、牛津等）
- **在相关领域有多篇第一级刊物的作者**
- **具有公认专业知识的作者**（奖项、编辑职务、学会会士）

#### 识别开创性论文

对于任何主题，通过以下方式确定基础工作：
1. **高被引次数**（通常5年以上的论文500+次）
2. **经常被其他纳入研究引用**（出现在许多参考文献列表中）
3. **发表于第一级刊物**（Nature, Science, Cell 系列）
4. **领域先驱撰写**（通常被认为是建立概念的人）

## 最佳实践

### 检索策略
1. **从 parallel-web 开始**：在查询专业数据库之前，先使用 `parallel-cli search` 配合学术域进行广泛的初步覆盖
2. **使用多个数据库**（至少3个）：确保全面覆盖——parallel-web 算一个来源
3. **包含预印本服务器**：捕获最新的未发表发现
4. **记录一切**：为可重复性记录检索字符串、日期、结果数量——将所有 parallel-cli 输出保存到 `sources/`
5. **测试并优化**：运行试检索，审查结果，调整检索词
6. **按被引次数排序**：可用时，按被引次数对结果排序，优先显示有影响力的工作
7. **使用 parallel-cli extract**：从检索中发现的有前景的URL获取完整内容，在全文筛选前验证相关性

### 筛选与选择
1. **使用多个数据库**（至少3个）：确保全面覆盖
2. **包含预印本服务器**：捕获最新的未发表发现
3. **记录一切**：为可重复性记录检索字符串、日期、结果数量
4. **测试并优化**：运行试检索，审查结果，调整检索词

### 筛选与选择
1. **使用清晰的标准**：在筛选前记录纳入/排除标准
2. **系统筛选**：标题 → 摘要 → 全文
3. **记录排除**：记录排除研究的原因
4. **考虑双人筛选**：对于系统综述，由两名评审员独立筛选

### 综合
1. **按主题组织**：按主题分组，而非逐项研究
2. **跨研究综合**：比较、对比、识别模式
3. **批判性**：评估证据的质量和一致性
4. **识别空白**：指出缺失或研究不足的内容

### 质量与可重复性
1. **评估研究质量**：使用适当的质量评估工具
2. **验证所有引文**：运行 verify_citations.py 脚本
3. **记录方法学**：提供足够的细节以便他人重复
4. **遵循指南**：系统综述使用PRISMA

### 写作
1. **客观**：公平呈现证据，承认局限性
2. **系统**：遵循结构化模板
3. **具体**：包含数字、统计数据、效应量（如有）
4. **清晰**：使用清晰的标题、逻辑流程、主题组织

## 常见陷阱避免

1. **单一数据库检索**：遗漏相关论文；务必检索多个数据库
2. **无检索文档**：使综述不可重复；文档化所有检索
3. **逐项研究总结**：缺乏综合；改为按主题组织
4. **未验证引文**：导致错误；始终运行 verify_citations.py
5. **检索范围过宽**：产生数千条不相关结果；使用特定词细化
6. **检索范围过窄**：遗漏相关论文；包含同义词和相关术语
7. **忽略预印本**：遗漏最新发现；包含 bioRxiv, medRxiv, arXiv
8. **无质量评估**：平等对待所有证据；评估并报告质量
9. **发表偏倚**：只发表阳性结果；注意潜在偏倚
10. **检索过时**：领域快速演进；明确说明检索日期

## 示例工作流程

生物医学文献综述的完整工作流程：

```bash
# 1. 从模板创建综述文档
cp assets/review_template.md crispr_sickle_cell_review.md

# 2. 从 parallel-web 开始广泛的学术检索
parallel-cli search "CRISPR Cas9 sickle cell disease gene therapy efficacy" \
  -q "CRISPR" -q "sickle cell" -q "gene therapy" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  --include-domains "scholar.google.com,arxiv.org,pubmed.ncbi.nlm.nih.gov,semanticscholar.org,biorxiv.org,nature.com,science.org,cell.com,pnas.org,nih.gov" \
  -o sources/litreview_crispr_scd-academic.json

parallel-cli search "CRISPR sickle cell disease clinical trials treatment" \
  -q "CRISPR" -q "sickle cell" \
  --json --max-results 10 --excerpt-max-chars-total 27000 \
  -o sources/litreview_crispr_scd-general.json

# 3. 使用适当技能检索专业数据库
# - 使用 gget 技能检索 PubMed、bioRxiv
# - 使用直接API访问 arXiv、Semantic Scholar
# - 以JSON格式导出结果

# 4. 聚合并处理结果（合并 parallel-cli + 数据库结果）
python scripts/search_databases.py combined_results.json \
  --deduplicate \
  --rank citations \
  --year-start 2015 \
  --year-end 2024 \
  --format markdown \
  --output search_results.md \
  --summary

# 5. 筛选结果并提取数据
# - 使用 parallel-cli extract 从有前景的URL获取完整内容
# - 手动筛选标题、摘要、全文
# - 将关键数据提取到综述文档中
# - 按主题组织

# 6. 按照模板结构撰写综述
# - 引言，包含明确目标
# - 详细的方法学部分
# - 按主题组织的结果
# - 批判性讨论
# - 清晰结论

# 7. 验证所有引文
python scripts/verify_citations.py crispr_sickle_cell_review.md

# 审查引文报告
cat crispr_sickle_cell_review_citation_report.json

# 修复任何失败的引文并重新验证
python scripts/verify_citations.py crispr_sickle_cell_review.md

# 8. 生成专业PDF
python scripts/generate_pdf.py crispr_sickle_cell_review.md \
  --citation-style nature \
  --output crispr_sickle_cell_review.pdf

# 9. 审查最终PDF和Markdown输出
```

## 与其他技能的集成

此技能可与其他科学技能无缝协作：

### 网络搜索与提取（parallel-web 技能——主要）
- **parallel-cli search**：广泛的学术和通用网络搜索，带域过滤——用于初步范围界定、查找论文、引文链式扩展和补充检索
- **parallel-cli extract**：从论文URL、期刊网站和预印本服务器获取完整内容——用于阅读摘要、提取参考文献列表和验证论文详情
- **parallel-cli search --include-domains**：跨学术域（arxiv.org、pubmed、nature.com等）的学术聚焦搜索

### 数据库访问技能
- **gget**：PubMed、bioRxiv、COSMIC、AlphaFold、Ensembl、UniProt
- **bioservices**：ChEMBL、KEGG、Reactome、UniProt、PubChem
- **datacommons-client**：人口统计、经济、健康统计数据

### 分析技能
- **pydeseq2**：RNA-seq差异表达（用于方法部分）
- **scanpy**：单细胞分析（用于方法部分）
- **anndata**：单细胞数据（用于方法部分）
- **biopython**：序列分析（用于背景部分）

### 可视化技能
- **matplotlib**：为综述生成图表和绘图
- **seaborn**：统计可视化

### 写作技能
- **brand-guidelines**：为PDF应用机构品牌标识
- **internal-comms**：为不同受众调整综述

## 资源

### 捆绑资源

**脚本：**
- `scripts/verify_citations.py`：验证DOI并生成格式正确的引文
- `scripts/generate_pdf.py`：将Markdown转换为专业PDF
- `scripts/search_databases.py`：处理、去重并格式化检索结果

**参考文献：**
- `references/citation_styles.md`：详细的引文格式指南（APA、Nature、Vancouver、Chicago、IEEE）
- `references/database_strategies.md`：全面的数据库检索策略

**资产：**
- `assets/review_template.md`：包含所有部分的完整文献综述模板

### 外部资源

**指南：**
- PRISMA（系统综述）：http://www.prisma-statement.org/
- Cochrane手册：https://training.cochrane.org/handbook
- AMSTAR 2（综述质量）：https://amstar.ca/

**工具：**
- MeSH浏览器：https://meshb.nlm.nih.gov/search
- PubMed高级检索：https://pubmed.ncbi.nlm.nih.gov/advanced/
- 布尔检索指南：https://www.ncbi.nlm.nih.gov/books/NBK3827/

**引文风格：**
- APA风格：https://apastyle.apa.org/
- Nature Portfolio：https://www.nature.com/nature-portfolio/editorial-policies/reporting-standards
- NLM/Vancouver：https://www.nlm.nih.gov/bsd/uniform_requirements.html

## 依赖项

### 必需的CLI工具
```bash
# parallel-cli（主要——用于网络搜索和URL提取）
curl -fsSL https://parallel.ai/install.sh | bash
# 或者：uv tool install "parallel-web-tools[cli]"
# 认证：parallel-cli auth
```

### 必需的Python包
```bash
pip install requests  # 用于引文验证
```

### 必需的系统工具
```bash
# 用于生成PDF
brew install pandoc  # macOS
apt-get install pandoc  # Linux

# 用于LaTeX（PDF生成）
brew install --cask mactex  # macOS
apt-get install texlive-xetex  # Linux
```

检查依赖项：
```bash
python scripts/generate_pdf.py --check-deps
```

## 总结

此文献综述技能提供：

1. **系统的方法学**，遵循学术最佳实践
2. **基于parallel-web的检索**，使用 `parallel-cli search` 进行快速、广泛的学术文献发现，并具有学术域过滤功能
3. **多数据库集成**，通过现有科学技能（gget、bioservices、datacommons-client）
4. **引文验证**，确保准确性和可信度
5. **专业输出**，包括Markdown和PDF格式
6. **全面指导**，涵盖整个综述过程
7. **质量保证**，通过验证和确认工具
8. **可重复性**，通过详细的文档要求

开展符合学术标准、全面综合当前知识的文献综述，适用于任何领域。
