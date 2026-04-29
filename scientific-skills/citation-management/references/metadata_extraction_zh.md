# 元数据提取指南

全面指南：使用多种API和服务从DOI、PMID、arXiv ID和URL中提取准确的引文元数据。

## 概述

准确的元数据对规范引文至关重要。本指南涵盖：
- 识别论文标识符（DOI、PMID、arXiv ID）
- 查询元数据API（CrossRef、PubMed、arXiv、DataCite）
- 各条目类型所需的BibTeX字段
- 处理边缘案例和特殊情况
- 验证提取的元数据

## 论文标识符

### DOI（数字对象标识符）

**格式**：`10.XXXX/suffix`

**示例**：
```
10.1038/s41586-021-03819-2    # Nature文章
10.1126/science.aam9317       # Science文章
10.1016/j.cell.2023.01.001    # Cell文章
10.1371/journal.pone.0123456  # PLOS ONE文章
```

**特性**：
- 永久标识符
- 元数据最可靠
- 解析至最新位置
- 出版商分配

**查找位置**：
- 文章首页
- 文章网页
- CrossRef、Google Scholar、PubMed
- 通常在出版商网站显著位置

### PMID（PubMed ID）

**格式**：8位数字（通常）

**示例**：
```
34265844
28445112
35476778
```

**特性**：
- 专用于PubMed数据库
- 仅限生物医学文献
- 由NCBI分配
- 永久标识符

**查找位置**：
- PubMed搜索结果
- PubMed文章页面
- 常出现在文章PDF页脚
- PMC（PubMed Central）页面

### PMCID（PubMed Central ID）

**格式**：PMC后接数字

**示例**：
```
PMC8287551
PMC7456789
```

**特性**：
- PMC中的免费全文文章
- PubMed文章的子集
- 开放获取或作者手稿

### arXiv ID

**格式**：YYMM.NNNNN 或 archive/YYMMNNN

**示例**：
```
2103.14030        # 新格式（2007年后）
2401.12345        # 2024年提交
arXiv:hep-th/9901001  # 旧格式
```

**特性**：
- 预印本（未经同行评审）
- 物理、数学、计算机科学、定量生物学等
- 版本追踪（v1, v2等）
- 免费开放获取

**查找位置**：
- arXiv.org
- 发表前常被引用
- 论文PDF页眉

### 其他标识符

**ISBN**（图书）：
```
978-0-12-345678-9
0-123-45678-9
```

**arXiv分类**：
```
cs.LG    # 计算机科学-机器学习
q-bio.QM # 定量生物学-定量方法
math.ST  # 数学-统计学
```

## 元数据API

### CrossRef API

**DOI的主要来源** - 期刊文章最全面的元数据。

**基础URL**：`https://api.crossref.org/works/`

**无需API密钥**，但建议使用礼貌池：
- 在User-Agent中添加邮箱
- 获得更优服务
- 无速率限制

#### 基础DOI查询

**请求**：
```
GET https://api.crossref.org/works/10.1038/s41586-021-03819-2
```

**响应**（简化）：
```json
{
  "message": {
    "DOI": "10.1038/s41586-021-03819-2",
    "title": ["文章标题"],
    "author": [
      {"given": "John", "family": "Smith"},
      {"given": "Jane", "family": "Doe"}
    ],
    "container-title": ["Nature"],
    "volume": "595",
    "issue": "7865",
    "page": "123-128",
    "published-print": {"date-parts": [[2021, 7, 1]]},
    "publisher": "Springer Nature",
    "type": "journal-article",
    "ISSN": ["0028-0836"]
  }
}
```

#### 可用字段

**始终存在**：
- `DOI`：数字对象标识符
- `title`：文章标题（数组）
- `type`：内容类型（期刊文章、图书章节等）

**通常存在**：
- `author`：作者对象数组
- `container-title`：期刊/图书标题
- `published-print`或`published-online`：发布日期
- `volume`、`issue`、`page`：出版详情
- `publisher`：出版商名称

**有时存在**：
- `abstract`：文章摘要
- `subject`：主题分类
- `ISSN`：期刊ISSN
- `ISBN`：图书ISBN
- `reference`：参考文献列表
- `is-referenced-by-count`：引用计数

#### 内容类型

CrossRef的`type`字段值：
- `journal-article`：期刊文章
- `book-chapter`：图书章节
- `book`：图书
- `proceedings-article`：会议论文
- `posted-content`：预印本
- `dataset`：研究数据集
- `report`：技术报告
- `dissertation`：学位论文

### PubMed E-utilities API

**专用于生物医学文献** - 含MeSH术语的精选元数据。

**基础URL**：`https://eutils.ncbi.nlm.nih.gov/entrez/eutils/`

**推荐使用API密钥**（免费）：
- 更高速率限制
- 更优性能

#### PMID转元数据

**步骤1：EFetch获取完整记录**

```
GET https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?
  db=pubmed&
  id=34265844&
  retmode=xml&
  api_key=YOUR_KEY
```

**响应**：包含全面元数据的XML

**步骤2：解析XML**

关键字段：
```xml
<PubmedArticle>
  <MedlineCitation>
    <PMID>34265844</PMID>
    <Article>
      <ArticleTitle>标题</ArticleTitle>
      <AuthorList>
        <Author><LastName>Smith</LastName><ForeName>John</ForeName></Author>
      </AuthorList>
      <Journal>
        <Title>Nature</Title>
        <JournalIssue>
          <Volume>595</Volume>
          <Issue>7865</Issue>
          <PubDate><Year>2021</Year></PubDate>
        </JournalIssue>
      </Journal>
      <Pagination><MedlinePgn>123-128</MedlinePgn></Pagination>
      <Abstract><AbstractText>摘要文本</AbstractText></Abstract>
    </Article>
  </MedlineCitation>
  <PubmedData>
    <ArticleIdList>
      <ArticleId IdType="doi">10.1038/s41586-021-03819-2</ArticleId>
      <ArticleId IdType="pmc">PMC8287551</ArticleId>
    </ArticleIdList>
  </PubmedData>
</PubmedArticle>
```

#### 特有PubMed字段

**MeSH术语**：受控词汇表
```xml
<MeshHeadingList>
  <MeshHeading>
    <DescriptorName UI="D003920">Diabetes Mellitus</DescriptorName>
  </MeshHeading>
</MeshHeadingList>
```

**出版物类型**：
```xml
<PublicationTypeList>
  <PublicationType UI="D016428">Journal Article</PublicationType>
  <PublicationType UI="D016449">Randomized Controlled Trial</PublicationType>
</PublicationTypeList>
```

**资助信息**：
```xml
<GrantList>
  <Grant>
    <GrantID>R01-123456</GrantID>
    <Agency>NIAID NIH HHS</Agency>
    <Country>United States</Country>
  </Grant>
</GrantList>
```

### arXiv API

**物理、数学、计算机科学、定量生物学领域的预印本** - 免费开放获取。

**基础URL**：`http://export.arxiv.org/api/query`

**无需API密钥**

#### arXiv ID转元数据

**请求**：
```
GET http://export.arxiv.org/api/query?id_list=2103.14030
```

**响应**：Atom XML

```xml
<entry>
  <id>http://arxiv.org/abs/2103.14030v2</id>
  <title>高精度蛋白质结构预测AlphaFold</title>
  <author><name>John Jumper</name></author>
  <author><name>Richard Evans</name></author>
  <published>2021-03-26T17:47:17Z</published>
  <updated>2021-07-01T16:51:46Z</updated>
  <summary>摘要文本...</summary>
  <arxiv:doi>10.1038/s41586-021-03819-2</arxiv:doi>
  <category term="q-bio.BM" scheme="http://arxiv.org/schemas/atom"/>
  <category term="cs.LG" scheme="http://arxiv.org/schemas/atom"/>
</entry>
```

#### 关键字段

- `id`：arXiv URL
- `title`：预印本标题
- `author`：作者列表
- `published`：初版日期
- `updated`：最新版本日期
- `summary`：摘要
- `arxiv:doi`：已发表文章的DOI
- `arxiv:journal_ref`：已发表文章的期刊引用
- `category`：arXiv分类

#### 版本追踪

arXiv追踪版本：
- `v1`：初始提交
- `v2`、`v3`等：修订版

**务必检查**预印本是否已在期刊发表（优先使用DOI）。

### DataCite API

**研究数据集、软件及其他成果** - 为非传统学术成果分配DOI。

**基础URL**：`https://api.datacite.org/dois/`

**类似CrossRef**，但面向数据集、软件、代码等。

**请求**：
```
GET https://api.datacite.org/dois/10.5281/zenodo.1234567
```

**响应**：包含数据集/软件元数据的JSON

## 必备BibTeX字段

### @article（期刊文章）

**必需**：
- `author`：作者姓名
- `title`：文章标题
- `journal`：期刊名称
- `year`：出版年份

**可选但推荐**：
- `volume`：卷号
- `number`：期号
- `pages`：页码范围（如123--145）
- `doi`：数字对象标识符
- `url`：无DOI时的URL
- `month`：出版月份

**示例**：
```bibtex
@article{Smith2024,
  author  = {Smith, John and Doe, Jane},
  title   = {蛋白质折叠新方法},
  journal = {Nature},
  year    = {2024},
  volume  = {625},
  number  = {8001},
  pages   = {123--145},
  doi     = {10.1038/nature12345}
}
```

### @book（图书）

**必需**：
- `author`或`editor`：作者/编辑
- `title`：书名
- `publisher`：出版商
- `year`：出版年份

**可选但推荐**：
- `edition`：版次（非首版时）
- `address`：出版商地址
- `isbn`：ISBN
- `url`：URL
- `series`：丛书名

**示例**：
```bibtex
@book{Kumar2021,
  author    = {Kumar, Vinay and Abbas, Abul K. and Aster, Jon C.},
  title     = {罗宾斯病理学基础},
  publisher = {Elsevier},
  year      = {2021},
  edition   = {10},
  isbn      = {978-0-323-53113-9}
}
```

### @inproceedings（会议论文）

**必需**：
- `author`：作者
- `title`：论文标题
- `booktitle`：会议/论文集名称
- `year`：年份

**可选但推荐**：
- `pages`：页码范围
- `organization`：主办机构
- `publisher`：出版商
- `address`：会议地点
- `month`：会议月份
- `doi`：DOI（如有）

**示例**：
```bibtex
@inproceedings{Vaswani2017,
  author    = {Vaswani, Ashish and Shazeer, Noam and others},
  title     = {注意力机制即一切},
  booktitle = {神经信息处理系统进展},
  year      = {2017},
  pages     = {5998--6008},
  volume    = {30}
}
```

### @incollection（图书章节）

**必需**：
- `author`：章节作者
- `title`：章节标题
- `booktitle`：书名
- `publisher`：出版商
- `year`：出版年份

**可选但推荐**：
- `editor`：图书编辑
- `pages`：章节页码范围
- `chapter`：章节编号
- `edition`：版次
- `address`：出版商地址

**示例**：
```bibtex
@incollection{Brown2020,
  author    = {Brown, Peter O. and Botstein, David},
  title     = {用DNA微阵列探索基因组新世界},
  booktitle = {DNA微阵列：分子克隆手册},
  editor    = {Eisen, Michael B. and Brown, Patrick O.},
  publisher = {冷泉港实验室出版社},
  year      = {2020},
  pages     = {1--45}
}
```

### @phdthesis（学位论文）

**必需**：
- `author`：作者
- `title`：论文标题
- `school`：机构
- `year`：年份

**可选**：
- `type`：类型（如"博士论文"）
- `address`：机构地址
- `month`：月份
- `url`：URL

**示例**：
```bibtex
@phdthesis{Johnson2023,
  author = {Johnson, Mary L.},
  title  = {癌症免疫治疗新方法},
  school = {斯坦福大学},
  year   = {2023},
  type   = {{PhD} dissertation}
}
```

### @misc（预印本、软件、数据集）

**必需**：
- `author`：作者
- `title`：标题
- `year`：年份

**预印本需添加**：
- `howpublished`：存储库（如"bioRxiv"）
- `doi`：预印本DOI
- `note`：预印本ID

**示例（预印本）**：
```bibtex
@misc{Zhang2024,
  author       = {Zhang, Yi and Chen, Li and Wang, Hui},
  title        = {阿尔茨海默病新治疗靶点},
  year         = {2024},
  howpublished = {bioRxiv},
  doi          = {10.1101/2024.01.001},
  note         = {预印本}
}
```

**示例（软件）**：
```bibtex
@misc{AlphaFold2021,
  author       = {DeepMind},
  title        = {{AlphaFold}蛋白质结构数据库},
  year         = {2021},
  howpublished = {软件},
  url          = {https://alphafold.ebi.ac.uk/},
  doi          = {10.5281/zenodo.5123456}
}
```

## 提取工作流

### 从DOI提取

**最佳实践**

**流程**：
1. 使用 PMID 查询 PubMed EFetch
2. 解析 XML 响应
3. 提取元数据（包括 MeSH 术语）
4. 检查响应中是否存在 DOI
5. 若存在 DOI，可选查询 CrossRef 获取补充元数据
6. 格式化为 BibTeX

### 通过 arXiv ID

**预印本处理**：

```bash
python scripts/extract_metadata.py --arxiv 2103.14030
```

**流程**：
1. 使用 ID 查询 arXiv API
2. 解析 Atom XML 响应
3. 检查是否已发表（响应中的 DOI）
4. 若已发表：使用 DOI 和 CrossRef
5. 若未发表：使用预印本元数据
6. 格式化为带预印本说明的 @misc 条目

**重要提示**：始终检查预印本是否已发表！

### 通过 URL

**仅知 URL 时**：

```bash
python scripts/extract_metadata.py \
  --url "https://www.nature.com/articles/s41586-021-03819-2"
```

**流程**：
1. 解析 URL 提取标识符
2. 识别类型（DOI/PMID/arXiv）
3. 从 URL 提取标识符
4. 查询对应 API
5. 格式化为 BibTeX

**URL 模式**：
```
# DOI URL
https://doi.org/10.1038/nature12345
https://dx.doi.org/10.1126/science.abc123
https://www.nature.com/articles/s41586-021-03819-2

# PubMed URL
https://pubmed.ncbi.nlm.nih.gov/34265844/
https://www.ncbi.nlm.nih.gov/pubmed/34265844

# arXiv URL
https://arxiv.org/abs/2103.14030
https://arxiv.org/pdf/2103.14030.pdf
```

### 批量处理

**混合标识符文件处理**：

```bash
# 创建每行一个标识符的文件
# identifiers.txt:
#   10.1038/nature12345
#   34265844
#   2103.14030
#   https://doi.org/10.1126/science.abc123

python scripts/extract_metadata.py \
  --input identifiers.txt \
  --output references.bib
```

**流程**：
- 脚本自动识别标识符类型
- 查询对应 API
- 合并为单一 BibTeX 文件
- 优雅处理错误

## 特殊案例与边界情况

### 预印本后续发表

**问题**：引用了预印本，但期刊版本已发布。

**解决方案**：
1. 检查 arXiv 元数据的 DOI 字段
2. 若存在 DOI，采用期刊版本
3. 更新为期刊文章引用
4. 必要时在注释中说明预印本版本

**示例**：
```bibtex
% 原始：arXiv:2103.14030
% 已发表为：
@article{Jumper2021,
  author  = {Jumper, John and Evans, Richard and others},
  title   = {Highly Accurate Protein Structure Prediction with {AlphaFold}},
  journal = {Nature},
  year    = {2021},
  volume  = {596},
  pages   = {583--589},
  doi     = {10.1038/s41586-021-03819-2}
}
```

### 多位作者（et al.）

**问题**：作者数量过多（10+）。

**BibTeX 实践**：
- 少于 10 位时列出全部
- 10 位以上使用 "and others"
- 或全部列出（期刊要求各异）

**示例**：
```bibtex
@article{LargeCollaboration2024,
  author = {First, Author and Second, Author and Third, Author and others},
  ...
}
```

### 作者姓名变体

**问题**：作者使用不同姓名格式发表。

**标准化建议**：
```
# 常见变体
John Smith
John A. Smith
John Andrew Smith
J. A. Smith
Smith, J.
Smith, J. A.

# BibTeX 推荐格式
author = {Smith, John A.}
```

**提取优先级**：
1. 优先使用全名
2. 包含中间名首字母
3. 格式：姓, 名 中间名

### 无 DOI 可用

**问题**：早期论文或书籍无 DOI。

**解决方案**：
1. 生物医学文献使用 PMID
2. 书籍使用 ISBN
3. 提供稳定来源 URL
4. 包含完整出版信息

**示例**：
```bibtex
@article{OldPaper1995,
  author  = {Author, Name},
  title   = {Title Here},
  journal = {Journal Name},
  year    = {1995},
  volume  = {123},
  pages   = {45--67},
  url     = {https://stable-url-here},
  note    = {PMID: 12345678}
}
```

### 会议论文 vs 期刊文章

**问题**：同一成果在会议和期刊发表。

**最佳实践**：
- 两者皆有时优先引用期刊版
- 期刊版为存档版本
- 会议版适用于时效性需求

**引用会议版**：
```bibtex
@inproceedings{Smith2024conf,
  author    = {Smith, John},
  title     = {Title},
  booktitle = {Proceedings of NeurIPS 2024},
  year      = {2024}
}
```

**引用期刊版**：
```bibtex
@article{Smith2024journal,
  author  = {Smith, John},
  title   = {Title},
  journal = {Journal of Machine Learning Research},
  year    = {2024}
}
```

### 书籍章节 vs 编辑文集

**正确提取**：
- 章节：使用 `@incollection`
- 整书：使用 `@book`
- 书籍编辑：填入 `editor` 字段
- 章节作者：填入 `author` 字段

### 数据集与软件

**使用 @misc** 并添加对应字段：

```bibtex
@misc{DatasetName2024,
  author       = {Author, Name},
  title        = {Dataset Title},
  year         = {2024},
  howpublished = {Zenodo},
  doi          = {10.5281/zenodo.123456},
  note         = {Version 1.2}
}
```

## 提取后验证

始终验证提取的元数据：

```bash
python scripts/validate_citations.py extracted_refs.bib
```

**检查项**：
- 必备字段完整
- DOI 可正常解析
- 作者姓名格式统一
- 年份合理（4位数）
- 期刊/出版社名称正确
- 页码范围使用 -- 而非 -
- 特殊字符处理无误

## 最佳实践

### 1. 优先使用 DOI

DOI 提供：
- 永久标识符
- 最佳元数据源
- 出版商验证信息
- 可解析链接

### 2. 验证自动提取的元数据

重点检查：
- 作者姓名与出版物一致
- 标题匹配（包括大小写）
- 年份正确
- 期刊名称完整

### 3. 处理特殊字符

**LaTeX 特殊字符**：
- 保护大写：`{AlphaFold}`
- 处理变音符号：`M{\"u}ller` 或 Unicode
- 化学式：`H$_2$O` 或 `\ce{H2O}`

### 4. 统一引用键格式

**惯例**：`第一作者年份关键词`
```
Smith2024protein
Doe2023machine
Johnson2024cancer
```

### 5. 现代文献必含 DOI

2000年后发表的论文应含DOI：
```bibtex
doi = {10.1038/nature12345}
```

### 6. 标注来源

非标准来源需添加说明：
```bibtex
note = {Preprint, not peer-reviewed}
note = {Technical report}
note = {Dataset accompanying [citation]}
```

## 总结

元数据提取流程：

1. **识别**：确定标识符类型（DOI/PMID/arXiv/URL）
2. **查询**：调用对应 API（CrossRef/PubMed/arXiv）
3. **提取**：解析响应获取必备字段
4. **格式化**：生成标准 BibTeX 条目
5. **验证**：检查完整性与准确性
6. **复核**：关键引用人工抽查

**自动化脚本**：
- `extract_metadata.py`：通用提取器
- `doi_to_bibtex.py`：快速 DOI 转换
- `validate_citations.py`：准确性验证

**最终提交前务必验证元数据！**
