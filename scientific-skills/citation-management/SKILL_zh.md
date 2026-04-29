---
name: citation-management
description: 学术研究的全面文献管理工具。可搜索Google Scholar和PubMed文献，提取精确元数据，验证引用信息，并生成格式规范的BibTeX条目。当您需要查找论文、验证引用信息、将DOI转换为BibTeX或确保科学写作中的参考文献准确性时，应使用此技能。
allowed-tools: Read Write Edit Bash
license: MIT License
metadata:
    skill-author: K-Dense Inc.
---

# 文献管理

## 概述

在研究写作全流程中系统化管理文献引用。本技能提供搜索学术数据库（Google Scholar、PubMed）、从多源（CrossRef、PubMed、arXiv）提取精确元数据、验证引用信息及生成规范BibTeX条目的工具与策略。

对维护引用准确性、避免参考文献错误及确保研究可复现至关重要。可与文献综述技能无缝集成，实现完整研究流程。

## 使用场景

在以下情况使用本技能：
- 通过Google Scholar或PubMed搜索特定论文
- 将DOI、PMID或arXiv ID转换为规范BibTeX格式
- 提取完整引用元数据（作者、标题、期刊、年份等）
- 验证现有引用的准确性
- 清理并格式化BibTeX文件
- 查找特定领域的高被引论文
- 核验引用信息与实际出版物是否匹配
- 构建稿件或学位论文的参考文献库
- 检查重复引用
- 确保引用格式一致性

## 科学图示增强功能

**使用本技能创建文档时，务必考虑添加科学图表以增强视觉传达效果。**

若文档未包含图示：
- 使用**scientific-schematics**技能生成AI驱动的出版级图表
- 只需用自然语言描述所需图示
- Nano Banana Pro将自动生成、审核并优化图表

**新建文档时：** 默认应生成科学图示，以可视化呈现文本中的核心概念、工作流、架构或关系。

**生成图示方法：**
```bash
python scripts/generate_schematic.py "您的图示描述" -o figures/output.png
```

AI将自动完成：
- 创建出版级图像并规范格式化
- 通过多轮迭代审核优化
- 确保可访问性（色盲友好、高对比度）
- 将输出保存至figures/目录

**添加图示的场景：**
- 文献引用工作流程图
- 文献检索方法论流程图
- 参考文献管理系统架构图
- 引用格式决策树
- 数据库集成示意图
- 任何受益于可视化的复杂概念

详细图示创建指南请参阅scientific-schematics技能文档。

---

## 核心工作流

文献管理遵循系统化流程：

### 阶段1：文献发现与检索

**目标**：通过学术搜索引擎查找相关论文。

#### Google Scholar检索

Google Scholar提供跨学科最全面的文献覆盖。

**基础检索**：
```bash
# 按主题检索论文
python scripts/search_google_scholar.py "CRISPR基因编辑" \
  --limit 50 \
  --output results.json

# 带年份过滤的检索
python scripts/search_google_scholar.py "机器学习蛋白质折叠" \
  --year-start 2020 \
  --year-end 2024 \
  --limit 100 \
  --output ml_proteins.json
```

**高级检索策略**（参见`references/google_scholar_search.md`）：
- 引号精确匹配短语：`"深度学习"`
- 按作者检索：`author:LeCun`
- 标题内检索：`intitle:"神经网络"`
- 排除术语：`机器学习 -综述`
- 使用排序选项查找高被引论文
- 按日期范围筛选获取最新成果

**最佳实践**：
- 使用具体目标检索词
- 包含关键技术术语和缩写
- 快速迭代领域需筛选近年文献
- 通过"被引次数"查找开创性论文
- 导出顶级结果供后续分析

#### PubMed检索

PubMed专注生物医学与生命科学文献（超3500万条引用）。

**基础检索**：
```bash
# PubMed检索
python scripts/search_pubmed.py "阿尔茨海默病治疗" \
  --limit 100 \
  --output alzheimers.json

# 使用MeSH术语和过滤器检索
python scripts/search_pubmed.py \
  --query '"Alzheimer Disease"[MeSH] AND "Drug Therapy"[MeSH]' \
  --date-start 2020 \
  --date-end 2024 \
  --publication-types "Clinical Trial,Review" \
  --output alzheimers_trials.json
```

**高级PubMed查询**（参见`references/pubmed_search.md`）：
- MeSH术语：`"Diabetes Mellitus"[MeSH]`
- 字段标签：`"cancer"[Title]`, `"Smith J"[Author]`
- 布尔运算符：`AND`, `OR`, `NOT`
- 日期过滤：`2020:2024[Publication Date]`
- 出版物类型：`"Review"[Publication Type]`
- 结合E-utilities API实现自动化

**最佳实践**：
- 使用MeSH浏览器查找标准术语
- 先在PubMed高级检索构建器中构造复杂查询
- 用OR包含多个同义词
- 获取PMID便于元数据提取
- 导出JSON或直接转为BibTeX

### 阶段2：元数据提取

**目标**：将文献标识符（DOI、PMID、arXiv ID）转换为完整准确的元数据。

#### 快速DOI转BibTeX

单条DOI转换工具：

```bash
# 转换单条DOI
python scripts/doi_to_bibtex.py 10.1038/s41586-021-03819-2

# 从文件批量转换DOI
python scripts/doi_to_bibtex.py --input dois.txt --output references.bib

# 不同输出格式
python scripts/doi_to_bibtex.py 10.1038/nature12345 --format json
```

#### 综合元数据提取

支持DOI、PMID、arXiv ID或URL：

```bash
# 从DOI提取
python scripts/extract_metadata.py --doi 10.1038/s41586-021-03819-2

# 从PMID提取
python scripts/extract_metadata.py --pmid 34265844

# 从arXiv ID提取
python scripts/extract_metadata.py --arxiv 2103.14030

# 从URL提取
python scripts/extract_metadata.py --url "https://www.nature.com/articles/s41586-021-03819-2"

# 从文件批量提取（混合标识符）
python scripts/extract_metadata.py --input identifiers.txt --output citations.bib
```

**元数据来源**（参见`references/metadata_extraction.md`）：

1. **CrossRef API**：DOI主要来源
   - 期刊文章完整元数据
   - 出版商提供信息
   - 包含作者、标题、期刊、卷号、页码、日期
   - 免费，无需API密钥

2. **PubMed E-utilities**：生物医学文献
   - NCBI官方元数据
   - 包含MeSH术语、摘要
   - PMID和PMCID标识符
   - 免费，高频率调用建议使用API密钥

3. **arXiv API**：物理、数学、计算机、定量生物学预印本
   - 预印本完整元数据
   - 版本追踪
   - 作者所属机构
   - 免费开放获取

4. **DataCite API**：研究数据集、软件等资源
   - 非传统学术成果元数据
   - 数据集和代码的DOI
   - 免费访问

**提取内容**：
- **必填字段**：作者、标题、年份
- **期刊文章**：期刊名称、卷号、期号、页码、DOI
- **书籍**：出版商、ISBN、版次
- **会议论文**：书名、会议地点、页码
- **预印本**：存储库（arXiv, bioRxiv）、预印本ID
- **附加项**：摘要、关键词、URL

### 阶段3：BibTeX格式化

**目标**：生成整洁规范的BibTeX条目。

#### BibTeX条目类型说明

完整指南参见`references/bibtex_formatting.md`。

**常用条目类型**：
- `@article`：期刊文章（最常用）
- `@book`：书籍
- `@inproceedings`：会议论文
- `@incollection`：书籍章节
- `@phdthesis`：学位论文
- `@misc`：预印本、软件、数据集

**类型必填字段**：

```bibtex
@article{引用键,
  author  = {姓1, 名1 and 姓2, 名2},
  title   = {文章标题},
  journal = {期刊名称},
  year    = {2024},
  volume  = {10},
  number  = {3},
  pages   = {123--145},
  doi     = {10.1234/example}
}

@inproceedings{引用键,
  author    = {姓, 名},
  title     = {论文标题},
  booktitle = {会议名称},
  year      = {2024},
  pages     = {1--10}
}

@book{引用键,
  author    = {姓, 名},
  title     = {书籍标题},
  publisher = {出版商名称},
  year      = {2024}
}
```

#### 格式化与清理

使用格式化工具标准化BibTeX文件：

```bash
# 格式化清理BibTeX文件
python scripts/format_bibtex.py references.bib \
  --output formatted_references.bib

# 按引用键排序条目
python scripts/format_bibtex.py references.bib \
  --sort key \
  --output sorted_references.bib

# 按年份排序（新到旧）
python scripts/format_bibtex.py references.bib \
  --sort year \
  --descending \
  --output sorted_references.bib

# 去重处理
python scripts/format_bibtex.py references.bib \
  --deduplicate \
  --output clean_references.bib

# 验证并生成问题报告
python scripts/format_bibtex.py references.bib \
  --validate \
  --report validation_report.txt
```

**格式化操作**：
- 标准化字段顺序
- 统一缩进与间距
- 标题正确大小写（用{}保护）
- 标准化作者姓名格式
- 统一引用键格式
- 移除冗余字段
- 修复常见错误（缺失逗号、括号）

### 阶段4：引用验证

**目标**：核验所有引用准确完整。

#### 综合验证

```bash
# 验证BibTeX文件
python scripts/validate_citations.py references.bib

# 验证并自动修复常见问题
python scripts/validate_citations.py references.bib \
  --auto-fix \
  --output validated_references.bib

# 生成详细验证报告
python scripts/validate_citations.py references.bib \
  --report validation_report.json \
  --verbose
```

**验证检查项**（参见`references/citation_validation.md`）：

1. **DOI核验**：
   - 通过doi.org正确解析DOI
   - BibTeX与CrossRef元数据匹配
   - 无失效或无效DOI

2. **必填字段**：
   - 条目类型所有必填字段完整
   - 无空缺或缺失关键信息
   - 作者姓名格式规范

3. **数据一致性**：
   - 年份有效（4位数，合理范围）
   - 卷号/期号为数字
   - 页码格式正确（如123--145）
   - URL可访问

4. **重复检测**：
   - 相同DOI重复使用
   - 相似标题（可能重复）
   - 相同作者/年份/标题组合

5. **格式合规**：
   - 有效BibTeX语法
   - 括号和引号使用规范
   - 引用键唯一
   - 特殊字符正确处理

**验证输出示例**：
```json
{
  "total_entries": 150,
  "valid_entries": 145,
  "errors": [
    {
      "citation_key": "Smith2023",
      "error_type": "missing_field",
      "field": "journal",
      "severity": "high"
    },
    {
      "citation_key": "Jones2022",
      "error_type": "invalid_doi",
      "doi": "10.1234/broken",
      "severity": "high"
    }
  ],
  "warnings": [
    {
      "citation_key": "Brown2021",
      "warning_type": "possible_duplicate",
      "duplicate_of": "Brown2021a",
      "severity": "medium"
    }
  ]
}
```

### 阶段5：写作流程集成

#### 构建稿件参考文献库

创建参考文献库的完整流程：

```bash
# 1. 按主题检索论文
python scripts/search_pubmed.py \
  '"CRISPR-Cas Systems"[MeSH] AND "Gene Editing"[MeSH]' \
  --date-start 2020 \
  --limit 200 \
  --output crispr_papers.json

# 2. 从检索结果提取DOI并转为BibTeX
python scripts/extract_metadata.py \
  --input crispr_papers.json \
  --output crispr_refs.bib

# 3. 通过DOI添加特定论文
python scripts/doi_to_bibtex.py 10.1038/nature12345 >> crispr_refs.bib
python scripts/doi_to_bibtex.py 10.1126/science.abcd1234 >> crispr_refs.bib

# 4. 格式化清理BibTeX文件
python scripts/format_bibtex.py crispr_refs.bib \
  --deduplicate \
  --sort year \
  --descending \
  --output references.bib

# 5. 验证所有引用
python scripts/validate_citations.py references.bib \
  --auto-fix \
  --report validation.json \
  --output final_references.bib

# 6. 审查验证报告并修复遗留问题
cat validation.json

# 7. 在LaTeX文档中使用
# \bibliography{final_references}
```

#### 与文献综述技能集成

本技能与`literature-review`技能互补：

**文献综述技能** → 系统性检索与综合  
**文献管理技能** → 技术性引用处理  

**组合工作流**：
1. 使用`literature-review`进行跨库综合检索
2. 使用`citation-management`提取验证所有引用
3. 使用`literature-review`按主题综合研究发现
4. 使用`citation-management`核验最终参考文献准确性

```bash
# 完成文献综述后
# 验证综述文档所有引用
python scripts/validate_citations.py my_review_references.bib --report review_validation.json

# 按需格式化特定引用风格
python scripts/format_bibtex.py my_review_references.bib \
  --style nature \
  --output formatted_refs.bib
```

## 检索策略

### Google Scholar最佳实践

**查找开创性与高影响力论文**（关键）：

始终依据被引量、期刊质量和作者声誉优先筛选论文：

**被引量阈值参考：**
| 论文年限 | 被引量 | 分类 |
|-----------|-----------|----------------|
| 0-3年 | 20+ | 值得关注 |
| 0-3年 | 100+ | 高影响力 |
| 3-7年 | 100+ | 重要成果 |
| 3-7年 | 500+ | 里程碑论文 |
| 7年以上 | 500+ | 开创性工作 |
| 7年以上 | 1000+ | 奠基性研究 |

**期刊质量分级：**
- **一级（优先）：** Nature、Science、Cell、NEJM、Lancet、JAMA、PNAS
- **二级（重点）：** 影响因子>10，顶级会议（NeurIPS、ICML、ICLR）
- **三级（优良）：** 专业期刊（IF 5-10）
- **四级（谨慎）：** 低影响力同行评审期刊

**作者声誉指标：**
- h指数>40的资深研究者
- 在顶级期刊多次发表
- 知名机构领导职务
- 获奖及编辑职位

**高影响力论文检索策略：**
- 按被引量排序（从高到低）
- 查找一级期刊综述文章获取领域概览
- 通过"被引"功能评估影响力及追踪后续研究
- 设置引用提醒追踪关键论文新引用
- 使用`source:Nature`或`source:Science`筛选顶级期刊
- 用`author:LastName`检索领域权威论文

**高级运算符**（完整列表见`references/google_scholar_search.md`）：
```
"精确短语"           # 精确短语匹配
author:姓氏          # 按作者检索
intitle:关键词        # 仅标题内检索
source:期刊名         # 检索特定期刊
-排除词              # 排除术语
OR                   # 替代术语
2020..202

"CRISPR" intitle:review 2023..2024

# 按特定作者查找主题文献
author:Church "synthetic biology"

# 查找高被引基础性工作
"deep learning" 2012..2015 sort:citations

# 排除综述并聚焦方法研究
"protein folding" -survey -review intitle:method
```

### PubMed 最佳实践

**使用 MeSH 术语**：
MeSH（医学主题词表）提供精确检索的受控词汇。

1. **查找 MeSH 术语**：访问 https://meshb.nlm.nih.gov/search
2. **在查询中使用**：`"Diabetes Mellitus, Type 2"[MeSH]`
3. **结合关键词**实现全面覆盖

**字段标签**：
```
[Title]              # 仅搜索标题
[Title/Abstract]     # 搜索标题或摘要
[Author]             # 按作者姓名搜索
[Journal]            # 搜索特定期刊
[Publication Date]   # 日期范围
[Publication Type]   # 文章类型
[MeSH]               # MeSH 术语
```

**构建复杂查询**：
```bash
# 近期发表的糖尿病治疗临床试验
"Diabetes Mellitus, Type 2"[MeSH] AND "Drug Therapy"[MeSH] 
AND "Clinical Trial"[Publication Type] AND 2020:2024[Publication Date]

# 特定期刊的CRISPR综述
"CRISPR-Cas Systems"[MeSH] AND "Nature"[Journal] AND "Review"[Publication Type]

# 特定作者近期工作
"Smith AB"[Author] AND cancer[Title/Abstract] AND 2022:2024[Publication Date]
```

**自动化 E-utilities**：
脚本使用 NCBI E-utilities API 实现程序化访问：
- **ESearch**：检索 PMID
- **EFetch**：获取完整元数据
- **ESummary**：获取摘要信息
- **ELink**：查找相关文献

完整 API 文档见 `references/pubmed_search.md`。

## 工具与脚本

### search_google_scholar.py

搜索 Google Scholar 并导出结果。

**功能**：
- 带速率限制的自动化搜索
- 分页支持
- 年份范围过滤
- 导出为 JSON 或 BibTeX
- 引用计数信息

**用法**：
```bash
# 基础搜索
python scripts/search_google_scholar.py "quantum computing"

# 带过滤的高级搜索
python scripts/search_google_scholar.py "quantum computing" \
  --year-start 2020 \
  --year-end 2024 \
  --limit 100 \
  --sort-by citations \
  --output quantum_papers.json

# 直接导出为 BibTeX
python scripts/search_google_scholar.py "machine learning" \
  --limit 50 \
  --format bibtex \
  --output ml_papers.bib
```

### search_pubmed.py

使用 E-utilities API 搜索 PubMed。

**功能**：
- 支持复杂查询（MeSH、字段标签、布尔逻辑）
- 日期范围过滤
- 文献类型过滤
- 批量元数据获取
- 导出为 JSON 或 BibTeX

**用法**：
```bash
# 简单关键词搜索
python scripts/search_pubmed.py "CRISPR gene editing"

# 带过滤的复杂查询
python scripts/search_pubmed.py \
  --query '"CRISPR-Cas Systems"[MeSH] AND "therapeutic"[Title/Abstract]' \
  --date-start 2020-01-01 \
  --date-end 2024-12-31 \
  --publication-types "Clinical Trial,Review" \
  --limit 200 \
  --output crispr_therapeutic.json

# 导出为 BibTeX
python scripts/search_pubmed.py "Alzheimer's disease" \
  --limit 100 \
  --format bibtex \
  --output alzheimers.bib
```

### extract_metadata.py

从文献标识符提取完整元数据。

**功能**：
- 支持 DOI、PMID、arXiv ID、URL
- 查询 CrossRef、PubMed、arXiv API
- 批量处理
- 多格式输出

**用法**：
```bash
# 单条 DOI
python scripts/extract_metadata.py --doi 10.1038/s41586-021-03819-2

# 单条 PMID
python scripts/extract_metadata.py --pmid 34265844

# 单条 arXiv ID
python scripts/extract_metadata.py --arxiv 2103.14030

# 通过 URL
python scripts/extract_metadata.py \
  --url "https://www.nature.com/articles/s41586-021-03819-2"

# 批量处理（每行一个标识符的文件）
python scripts/extract_metadata.py \
  --input paper_ids.txt \
  --output references.bib

# 不同输出格式
python scripts/extract_metadata.py \
  --doi 10.1038/nature12345 \
  --format json  # 或 bibtex, yaml
```

### validate_citations.py

验证 BibTeX 条目的准确性和完整性。

**功能**：
- 通过 doi.org 和 CrossRef 验证 DOI
- 必填字段检查
- 重复项检测
- 格式验证
- 自动修复常见问题
- 详细报告

**用法**：
```bash
# 基础验证
python scripts/validate_citations.py references.bib

# 自动修复
python scripts/validate_citations.py references.bib \
  --auto-fix \
  --output fixed_references.bib

# 详细验证报告
python scripts/validate_citations.py references.bib \
  --report validation_report.json \
  --verbose

# 仅检查 DOI
python scripts/validate_citations.py references.bib \
  --check-dois-only
```

### format_bibtex.py

格式化并清理 BibTeX 文件。

**功能**：
- 标准化格式
- 排序条目（按引用键、年份、作者）
- 去重
- 语法验证
- 修复常见错误
- 强制引用键规范

**用法**：
```bash
# 基础格式化
python scripts/format_bibtex.py references.bib

# 按年份排序（最新优先）
python scripts/format_bibtex.py references.bib \
  --sort year \
  --descending \
  --output sorted_refs.bib

# 去重
python scripts/format_bibtex.py references.bib \
  --deduplicate \
  --output clean_refs.bib

# 完整清理
python scripts/format_bibtex.py references.bib \
  --deduplicate \
  --sort year \
  --validate \
  --auto-fix \
  --output final_refs.bib
```

### doi_to_bibtex.py

快速将 DOI 转换为 BibTeX。

**功能**：
- 快速单条 DOI 转换
- 批量处理
- 多格式输出
- 剪贴板支持

**用法**：
```bash
# 单条 DOI
python scripts/doi_to_bibtex.py 10.1038/s41586-021-03819-2

# 多条 DOI
python scripts/doi_to_bibtex.py \
  10.1038/nature12345 \
  10.1126/science.abc1234 \
  10.1016/j.cell.2023.01.001

# 从文件读取（每行一个 DOI）
python scripts/doi_to_bibtex.py --input dois.txt --output references.bib

# 复制到剪贴板
python scripts/doi_to_bibtex.py 10.1038/nature12345 --clipboard
```

## 最佳实践

### 检索策略

1. **先宽后窄**：
   - 从通用术语开始了解领域
   - 用特定关键词和过滤器细化
   - 使用同义词和相关术语

2. **多源检索**：
   - Google Scholar 全面覆盖
   - PubMed 专注生物医学
   - arXiv 获取预印本
   - 合并结果确保完整性

3. **利用引用关系**：
   - 通过"被引用"查找开创性论文
   - 查阅关键论文的参考文献
   - 使用引文网络发现相关研究

4. **记录检索过程**：
   - 保存检索式和日期
   - 记录结果数量
   - 注明使用的过滤条件

### 元数据提取

1. **优先使用 DOI**：
   - 最可靠的标识符
   - 文献永久链接
   - 通过 CrossRef 获取最佳元数据

2. **验证提取结果**：
   - 核对作者姓名
   - 确认期刊/会议名称
   - 验证出版年份
   - 检查页码和卷号

3. **处理特殊情况**：
   - 预印本：包含存储库和 ID
   - 已发表预印本：使用期刊版本
   - 会议论文：包含会议名称和地点
   - 书籍章节：包含书名和编者

4. **保持一致性**：
   - 统一作者姓名格式
   - 标准化期刊缩写
   - 统一 DOI 格式（推荐 URL 形式）

### BibTeX 质量

1. **遵循规范**：
   - 使用有意义的引用键（FirstAuthor2024keyword）
   - 用{}保护标题中的大写字母
   - 页码范围使用 --（非单连字符）
   - 现代文献必须包含 DOI 字段

2. **保持简洁**：
   - 删除不必要字段
   - 避免冗余信息
   - 统一格式化
   - 定期验证语法

3. **系统化组织**：
   - 按年份或主题排序
   - 分组相关论文
   - 不同项目使用独立文件
   - 合并时避免重复

### 验证

1. **及早且频繁验证**：
   - 添加文献时即时验证
   - 提交前验证完整参考文献
   - 手动修改后重新验证

2. **及时修复问题**：
   - 失效 DOI：查找正确标识符
   - 缺失字段：从原始来源提取
   - 重复项：保留最佳版本
   - 格式错误：安全时使用自动修复

3. **关键文献人工复核**：
   - 验证核心文献引用准确性
   - 核对作者姓名与出版物一致
   - 确认页码和卷号
   - 确保 URL 有效

## 常见陷阱与规避

1. **单一来源偏差**：仅使用 Google Scholar 或 PubMed
   - **解决方案**：多数据库检索确保全面性

2. **盲目接受元数据**：未验证提取信息
   - **解决方案**：对照原始来源抽查元数据

3. **忽略 DOI 错误**：参考文献中存在失效 DOI
   - **解决方案**：最终提交前运行验证

4. **格式不一致**：混合的引用键风格
   - **解决方案**：使用 format_bibtex.py 标准化

5. **重复条目**：同一文献不同引用键
   - **解决方案**：验证时启用重复检测

6. **缺失必填字段**：不完整的 BibTeX 条目
   - **解决方案**：验证并补全必填字段

7. **过时预印本**：引用预印本但期刊版已存在
   - **解决方案**：检查预印本是否已发表并更新

8. **特殊字符问题**：非常规字符导致 LaTeX 编译失败
   - **解决方案**：BibTeX 中正确转义或使用 Unicode

9. **提交前未验证**：带着引用错误提交
   - **解决方案**：最终检查必须运行验证

10. **手动录入 BibTeX**：手工输入条目
    - **解决方案**：始终通过脚本从元数据源提取

## 工作流示例

### 示例1：构建论文参考文献库

```bash
# 步骤1：查找主题核心文献
python scripts/search_google_scholar.py "transformer neural networks" \
  --year-start 2017 \
  --limit 50 \
  --output transformers_gs.json

python scripts/search_pubmed.py "deep learning medical imaging" \
  --date-start 2020 \
  --limit 50 \
  --output medical_dl_pm.json

# 步骤2：从搜索结果提取元数据
python scripts/extract_metadata.py \
  --input transformers_gs.json \
  --output transformers.bib

python scripts/extract_metadata.py \
  --input medical_dl_pm.json \
  --output medical.bib

# 步骤3：添加已知特定文献
python scripts/doi_to_bibtex.py 10.1038/s41586-021-03819-2 >> specific.bib
python scripts/doi_to_bibtex.py 10.1126/science.aam9317 >> specific.bib

# 步骤4：合并所有 BibTeX 文件
cat transformers.bib medical.bib specific.bib > combined.bib

# 步骤5：格式化并去重
python scripts/format_bibtex.py combined.bib \
  --deduplicate \
  --sort year \
  --descending \
  --output formatted.bib

# 步骤6：验证
python scripts/validate_citations.py formatted.bib \
  --auto-fix \
  --report validation.json \
  --output final_references.bib

# 步骤7：复查问题
cat validation.json | grep -A 3 '"errors"'

# 步骤8：在 LaTeX 中使用
# \bibliography{final_references}
```

### 示例2：转换 DOI 列表

```bash
# 文本文件包含 DOI（每行一个）
# dois.txt 内容：
# 10.1038/s41586-021-03819-2
# 10.1126/science.aam9317
# 10.1016/j.cell.2023.01.001

# 全部转换为 BibTeX
python scripts/doi_to_bibtex.py --input dois.txt --output references.bib

# 验证结果
python scripts/validate_citations.py references.bib --verbose
```

### 示例3：清理现有 BibTeX 文件

```bash
# 清理来自不同来源的杂乱 BibTeX 文件

# 步骤1：格式化标准化
python scripts/format_bibtex.py messy_references.bib \
  --output step1_formatted.bib

# 步骤2：去重
python scripts/format_bibtex.py step1_formatted.bib \
  --deduplicate \
  --output step2_deduplicated.bib

# 步骤3：验证并自动修复
python scripts/validate_citations.py step2_deduplicated.bib \
  --auto-fix \
  --output step3_validated.bib

# 步骤4：按年份排序
python scripts/format_bibtex.py step3_validated.bib \
  --sort year \
  --descending \
  --output clean_references.bib

# 步骤5：最终验证报告
python scripts/validate_citations.py clean_references.bib \
  --report final_validation.json \
  --verbose

# 查看报告
cat final_validation.json
```

### 示例4：查找并引用开创性论文

```bash
# 查找主题高被引文献
python scripts/search_google_scholar.py "AlphaFold protein structure" \
  --year-start 2020 \
  --year-end 2024 \
  --sort-by citations \
  --limit 20 \
  --output alphafold_seminal.json

# 提取被引量前10的文献
# (JSON 中已包含被引量)

# 转换为 BibTeX
python scripts/extract_metadata.py \
  --input alphafold_seminal.json \
  --output alphafold_refs.bib

# 获得最具影响力论文的 BibTeX
```

## 与其他技能集成

### 文献综述技能

**文献管理**为**文献综述**提供技术基础：
- **文献综述**：多数据库系统检索与综合
- **文献管理**：元数据提取与验证

**组合工作流**：
1. 使用 literature-review 进行系统检索
2. 使用 citation-management 提取验证文献
3. 使用 literature-review 综合发现
4. 使用 citation-management 确保参考文献准确

### 科技写作技能

**文献管理**确保**科技写作**的准确引用：
- 导出验证后的 BibTeX 用于 LaTeX 稿件
- 验证引用符合出版标准
- 按期刊要求格式化参考文献

### 会议模板技能

**文献管理**配合**会议模板**生成可投稿稿件：
- 不同会议需要不同引文格式
- 生成规范格式的参考文献
- 验证引用满足会议要求

## 资源

### 内置资源

**参考文献**（位于 `references/`）：
- `google_scholar_search.md`：Google Scholar 完整检索指南
- `pubmed_search.md`：PubMed 和 E-utilities API 文档
- `metadata_extraction.md`：元数据来源与字段要求
- `citation_validation.md`：验证标准与质量检查
- `bibtex_formatting.md`：BibTeX 条目类型与格式规则

**脚本**（位于 `scripts/`）：
- `search_google_scholar.py`：Google Scholar 自动化检索
- `search_pub

- `format_bibtex.py`：BibTeX 格式化与清理工具
- `doi_to_bibtex.py`：快速 DOI 转 BibTeX 转换器

**资源文件**（位于 `assets/` 目录）：
- `bibtex_template.bib`：全类型 BibTeX 条目示例
- `citation_checklist.md`：质量保证检查清单

### 外部资源

**学术搜索引擎**：
- Google Scholar：https://scholar.google.com/
- PubMed：https://pubmed.ncbi.nlm.nih.gov/
- PubMed 高级检索：https://pubmed.ncbi.nlm.nih.gov/advanced/

**元数据接口**：
- CrossRef API：https://api.crossref.org/
- PubMed E-utilities：https://www.ncbi.nlm.nih.gov/books/NBK25501/
- arXiv API：https://arxiv.org/help/api/
- DataCite API：https://api.datacite.org/

**工具与验证器**：
- MeSH 主题词浏览器：https://meshb.nlm.nih.gov/search
- DOI 解析器：https://doi.org/
- BibTeX 格式指南：http://www.bibtex.org/Format/

**引文格式**：
- BibTeX 文档：http://www.bibtex.org/
- LaTeX 文献管理：https://www.overleaf.com/learn/latex/Bibliography_management

## 依赖项

### 必需 Python 包

```bash
# 核心依赖
pip install requests  # 用于 API 的 HTTP 请求
pip install bibtexparser  # BibTeX 解析与格式化
pip install biopython  # PubMed E-utilities 访问

# 可选（用于 Google Scholar）
pip install scholarly  # Google Scholar API 封装
# 或
pip install selenium  # 更稳定的学术搜索爬取
```

### 可选工具

```bash
# 高级验证工具
pip install crossref-commons  # 增强型 CrossRef API 访问
pip install pylatexenc  # LaTeX 特殊字符处理
```

## 功能概述

文献管理工具提供：

1. **全面检索能力**：支持 Google Scholar 和 PubMed
2. **自动化元数据提取**：支持 DOI、PMID、arXiv ID、URL 等来源
3. **引文验证**：包含 DOI 校验与完整性检查
4. **BibTeX 格式化**：标准化与清理工具
5. **质量保证**：通过验证与报告机制
6. **无缝集成**：科学写作流程整合
7. **可复现性**：文档化的检索与提取方法

使用本工具可在研究过程中维护精准完整的文献引用，确保生成符合出版要求的参考文献目录。
