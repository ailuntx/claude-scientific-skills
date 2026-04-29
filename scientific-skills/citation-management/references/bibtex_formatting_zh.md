# BibTeX 格式指南

全面介绍 BibTeX 条目类型、必填字段、格式规范及最佳实践。

## 概述

BibTeX 是 LaTeX 文档的标准参考文献格式。正确格式化可确保：
- 准确呈现引用
- 格式一致性
- 兼容各类引文样式
- 避免编译错误

本指南涵盖所有常见条目类型及格式规则。

## 条目类型

### @article - 期刊论文

**最常用的条目类型**，适用于同行评审期刊论文。

**必填字段**：
- `author`：作者姓名
- `title`：论文标题
- `journal`：期刊名称
- `year`：出版年份

**可选字段**：
- `volume`：卷号
- `number`：期号
- `pages`：页码范围
- `month`：出版月份
- `doi`：数字对象标识符
- `url`：网址
- `note`：附加说明

**模板**：
```bibtex
@article{CitationKey2024,
  author  = {Last1, First1 and Last2, First2},
  title   = {Article Title Here},
  journal = {Journal Name},
  year    = {2024},
  volume  = {10},
  number  = {3},
  pages   = {123--145},
  doi     = {10.1234/journal.2024.123456},
  month   = jan
}
```

**示例**：
```bibtex
@article{Jumper2021,
  author  = {Jumper, John and Evans, Richard and Pritzel, Alexander and others},
  title   = {Highly Accurate Protein Structure Prediction with {AlphaFold}},
  journal = {Nature},
  year    = {2021},
  volume  = {596},
  number  = {7873},
  pages   = {583--589},
  doi     = {10.1038/s41586-021-03819-2}
}
```

### @book - 书籍

**适用于完整书籍**。

**必填字段**：
- `author` 或 `editor`：作者或编者
- `title`：书名
- `publisher`：出版社名称
- `year`：出版年份

**可选字段**：
- `volume`：卷号（多卷本）
- `series`：丛书名称
- `address`：出版社地点
- `edition`：版次
- `isbn`：ISBN
- `url`：网址

**模板**：
```bibtex
@book{CitationKey2024,
  author    = {Last, First},
  title     = {Book Title},
  publisher = {Publisher Name},
  year      = {2024},
  edition   = {3},
  address   = {City, Country},
  isbn      = {978-0-123-45678-9}
}
```

**示例**：
```bibtex
@book{Kumar2021,
  author    = {Kumar, Vinay and Abbas, Abul K. and Aster, Jon C.},
  title     = {Robbins and Cotran Pathologic Basis of Disease},
  publisher = {Elsevier},
  year      = {2021},
  edition   = {10},
  address   = {Philadelphia, PA},
  isbn      = {978-0-323-53113-9}
}
```

### @inproceedings - 会议论文

**适用于会议论文集中的论文**。

**必填字段**：
- `author`：作者姓名
- `title`：论文标题
- `booktitle`：会议/论文集名称
- `year`：年份

**可选字段**：
- `editor`：论文集编者
- `volume`：卷号
- `series`：丛书名称
- `pages`：页码范围
- `address`：会议地点
- `month`：会议月份
- `organization`：主办机构
- `publisher`：出版社
- `doi`：DOI

**模板**：
```bibtex
@inproceedings{CitationKey2024,
  author    = {Last, First},
  title     = {Paper Title},
  booktitle = {Proceedings of Conference Name},
  year      = {2024},
  pages     = {123--145},
  address   = {City, Country},
  month     = jun
}
```

**示例**：
```bibtex
@inproceedings{Vaswani2017,
  author    = {Vaswani, Ashish and Shazeer, Noam and Parmar, Niki and others},
  title     = {Attention is All You Need},
  booktitle = {Advances in Neural Information Processing Systems 30 (NeurIPS 2017)},
  year      = {2017},
  pages     = {5998--6008},
  address   = {Long Beach, CA}
}
```

**注意**：`@conference` 是 `@inproceedings` 的别名。

### @incollection - 书籍章节

**适用于编辑书籍中的章节**。

**必填字段**：
- `author`：章节作者
- `title`：章节标题
- `booktitle`：书名
- `publisher`：出版社名称
- `year`：出版年份

**可选字段**：
- `editor`：书籍编者
- `volume`：卷号
- `series`：丛书名称
- `type`：章节类型（如"chapter"）
- `chapter`：章节编号
- `pages`：页码范围
- `address`：出版社地点
- `edition`：版次
- `month`：月份

**模板**：
```bibtex
@incollection{CitationKey2024,
  author    = {Last, First},
  title     = {Chapter Title},
  booktitle = {Book Title},
  editor    = {Editor, Last and Editor2, Last},
  publisher = {Publisher Name},
  year      = {2024},
  pages     = {123--145},
  chapter   = {5}
}
```

**示例**：
```bibtex
@incollection{Brown2020,
  author    = {Brown, Peter O. and Botstein, David},
  title     = {Exploring the New World of the Genome with {DNA} Microarrays},
  booktitle = {DNA Microarrays: A Molecular Cloning Manual},
  editor    = {Eisen, Michael B. and Brown, Patrick O.},
  publisher = {Cold Spring Harbor Laboratory Press},
  year      = {2020},
  pages     = {1--45},
  address   = {Cold Spring Harbor, NY}
}
```

### @phdthesis - 博士论文

**适用于博士论文**。

**必填字段**：
- `author`：作者姓名
- `title`：论文标题
- `school`：机构名称
- `year`：年份

**可选字段**：
- `type`：类型（如"PhD dissertation"）
- `address`：机构地点
- `month`：月份
- `url`：网址
- `note`：附加说明

**模板**：
```bibtex
@phdthesis{CitationKey2024,
  author = {Last, First},
  title  = {Dissertation Title},
  school = {University Name},
  year   = {2024},
  type   = {{PhD} dissertation},
  address = {City, State}
}
```

**示例**：
```bibtex
@phdthesis{Johnson2023,
  author  = {Johnson, Mary L.},
  title   = {Novel Approaches to Cancer Immunotherapy Using {CRISPR} Technology},
  school  = {Stanford University},
  year    = {2023},
  type    = {{PhD} dissertation},
  address = {Stanford, CA}
}
```

**注意**：`@mastersthesis` 适用于硕士论文。

### @mastersthesis - 硕士论文

**适用于硕士论文**。

**必填字段**：
- `author`：作者姓名
- `title`：论文标题
- `school`：机构名称
- `year`：年份

**模板**：
```bibtex
@mastersthesis{CitationKey2024,
  author = {Last, First},
  title  = {Thesis Title},
  school = {University Name},
  year   = {2024}
}
```

### @misc - 其他类型

**适用于其他类别不匹配的项目**（预印本、数据集、软件、网站等）。

**必填字段**：
- `author`（如已知）
- `title`
- `year`

**可选字段**：
- `howpublished`：存储库/网站/格式
- `url`：网址
- `doi`：DOI
- `note`：附加信息
- `month`：月份

**预印本模板**：
```bibtex
@misc{CitationKey2024,
  author       = {Last, First},
  title        = {Preprint Title},
  year         = {2024},
  howpublished = {bioRxiv},
  doi          = {10.1101/2024.01.01.123456},
  note         = {Preprint}
}
```

**数据集模板**：
```bibtex
@misc{DatasetName2024,
  author       = {Last, First},
  title        = {Dataset Title},
  year         = {2024},
  howpublished = {Zenodo},
  doi          = {10.5281/zenodo.123456},
  note         = {Version 1.2}
}
```

**软件模板**：
```bibtex
@misc{SoftwareName2024,
  author       = {Last, First},
  title        = {Software Name},
  year         = {2024},
  howpublished = {GitHub},
  url          = {https://github.com/user/repo},
  note         = {Version 2.0}
}
```

### @techreport - 技术报告

**适用于技术报告**。

**必填字段**：
- `author`：作者姓名
- `title`：报告标题
- `institution`：机构名称
- `year`：年份

**可选字段**：
- `type`：报告类型
- `number`：报告编号
- `address`：机构地点
- `month`：月份

**模板**：
```bibtex
@techreport{CitationKey2024,
  author      = {Last, First},
  title       = {Report Title},
  institution = {Institution Name},
  year        = {2024},
  type        = {Technical Report},
  number      = {TR-2024-01}
}
```

### @unpublished - 未发表作品

**适用于未发表作品**（预印本请使用 @misc）。

**必填字段**：
- `author`：作者姓名
- `title`：作品标题
- `note`：描述说明

**可选字段**：
- `month`：月份
- `year`：年份

**模板**：
```bibtex
@unpublished{CitationKey2024,
  author = {Last, First},
  title  = {Work Title},
  note   = {Unpublished manuscript},
  year   = {2024}
}
```

### @online/@electronic - 在线资源

**适用于网页及纯在线内容**。

**注意**：非标准 BibTeX 格式，但多数文献包（如 biblatex）支持。

**必填字段**：
- `author` 或 `organization`
- `title`
- `url`
- `year`

**模板**：
```bibtex
@online{CitationKey2024,
  author = {{Organization Name}},
  title  = {Page Title},
  url    = {https://example.com/page},
  year   = {2024},
  note   = {Accessed: 2024-01-15}
}
```

## 格式规则

### 引用键

**命名惯例**：`第一作者年份关键词`

**示例**：
```bibtex
Smith2024protein
Doe2023machine
JohnsonWilliams2024cancer  % 多位作者，无空格
NatureEditorial2024        % 无作者时用出版物名称
WHO2024guidelines          % 机构作者
```

**规则**：
- 仅含字母数字及：`-`、`_`、`.`、`:`
- 禁止空格
- 大小写敏感
- 文件内唯一
- 具有描述性

**避免**：
- 特殊字符：`@`、`#`、`&`、`%`、`$`
- 空格：改用驼峰式或下划线
- 数字开头：`2024Smith`（部分系统不支持）

### 作者姓名

**推荐格式**：`姓, 名 中间名`

**单作者**：
```bibtex
author = {Smith, John}
author = {Smith, John A.}
author = {Smith, John Andrew}
```

**多作者** - 用 `and` 分隔：
```bibtex
author = {Smith, John and Doe, Jane}
author = {Smith, John A. and Doe, Jane M. and Johnson, Mary L.}
```

**大量作者**（10+）：
```bibtex
author = {Smith, John and Doe, Jane and Johnson, Mary and others}
```

**特殊情况**：
```bibtex
% 后缀（Jr.、III 等）
author = {King, Jr., Martin Luther}

% 机构作者
author = {{World Health Organization}}
% 注意：双花括号保持整体性

% 复合姓氏
author = {Garc{\'i}a-Mart{\'i}nez, Jos{\'e}}

% 冠词（van、von、de 等）
author = {van der Waals, Johannes}
author = {de Broglie, Louis}
```

**错误格式**（请勿使用）：
```bibtex
author = {Smith, J.; Doe, J.}  % 分号（错误）
author = {Smith, J., Doe, J.}  % 逗号（错误）
author = {Smith, J. & Doe, J.} % &符号（错误）
author = {Smith J}             % 无逗号
```

### 标题大小写

**用花括号保护特定大写**：

```bibtex
% 专有名词、缩写、公式
title = {{AlphaFold}: Protein Structure Prediction}
title = {Machine Learning for {DNA} Sequencing}
title = {The {Ising} Model in Statistical Physics}
title = {{CRISPR-Cas9} Gene Editing Technology}
```

**原因**：引文样式可能修改大小写，花括号提供保护。

**示例**：
```bibtex
% 正确示例
title = {Advances in {COVID-19} Treatment}
title = {Using {Python} for Data Analysis}
title = {The {AlphaFold} Protein Structure Database}

% 在标题大小写样式中会转为小写
title = {Advances in COVID-19 Treatment}  % covid-19
title = {Using Python for Data Analysis}  % python
```

**全标题保护**（极少使用）：
```bibtex
title = {{This Entire Title Keeps Its Capitalization}}
```

### 页码范围

**使用短破折号**（双连字符 `--`）：

```bibtex
pages = {123--145}     % 正确
pages = {1234--1256}   % 正确
pages = {e0123456}     % 文章ID（PLOS等）
pages = {123}          % 单页
```

**错误**：
```bibtex
pages = {123-145}      % 单连字符（勿用）
pages = {pp. 123-145}  % 无需"pp."
pages = {123–145}      % Unicode短破折号（可能导致问题）
```

### 月份名称

**使用三字母缩写**（不加引号）：

```bibtex
month = jan
month = feb
month = mar
month = apr
month = may
month = jun
month = jul
month = aug
month = sep
month = oct
month = nov
month = dec
```

**或数字格式**：
```bibtex
month = {1}   % 一月
month = {12}  % 十二月
```

**或完整名称加花括号**：
```bibtex
month = {January}
```

**标准缩写无需引号**，因 BibTeX 已预定义。

### 期刊名称

**使用完整名称**（勿缩写）：

```bibtex
journal = {Nature}
journal = {Science}
journal = {Cell}
journal = {Proceedings of the National Academy of Sciences}
journal = {Journal of the American Chemical Society}
```

**文献样式**会在需要时自动处理缩写。

**避免手动缩写**：
```bibtex
% 勿在BibTeX文件中使用
journal = {Proc. Natl. Acad. Sci. U.S.A.}

% 应使用完整名称
journal =

### DOI 格式规范

**URL 格式**（推荐）：

```bibtex
doi = {10.1038/s41586-021-03819-2}
```

**避免**：
```bibtex
doi = {https://doi.org/10.1038/s41586-021-03819-2}  % 不要包含URL
doi = {doi:10.1038/s41586-021-03819-2}              % 不要添加前缀
```

**LaTeX** 会自动将其格式化为 URL。

**注意**：DOI 字段后不要加句点！

### URL 格式规范

```bibtex
url = {https://www.example.com/article}
```

**适用场景**：
- 无 DOI 时
- 网页引用
- 补充材料

**避免重复**：
```bibtex
% 当 DOI URL 与 url 相同时不要同时包含
doi = {10.1038/nature12345}
url = {https://doi.org/10.1038/nature12345}  % 冗余！
```

### 特殊字符处理

**重音符号与变音符号**：
```bibtex
author = {M{\"u}ller, Hans}        % ü
author = {Garc{\'i}a, Jos{\'e}}    % í, é
author = {Erd{\H{o}}s, Paul}       % ő
author = {Schr{\"o}dinger, Erwin}  % ö
```

**或使用 UTF-8 编码**（需正确配置 LaTeX）：
```bibtex
author = {Müller, Hans}
author = {García, José}
```

**数学符号**：
```bibtex
title = {The $\alpha$-helix Structure}
title = {$\beta$-sheet Prediction}
```

**化学式**：
```bibtex
title = {H$_2$O Molecular Dynamics}
% 或使用 chemformula 宏包：
title = {\ce{H2O} Molecular Dynamics}
```

### 字段顺序

**推荐顺序**（增强可读性）：

```bibtex
@article{Key,
  author  = {},
  title   = {},
  journal = {},
  year    = {},
  volume  = {},
  number  = {},
  pages   = {},
  doi     = {},
  url     = {},
  note    = {}
}
```

**原则**：
- 重要字段优先
- 条目间保持统一
- 使用格式化工具标准化

## 最佳实践

### 1. 格式统一性
保持全库格式一致：
- 作者姓名格式
- 标题大小写
- 期刊名称
- 引用键风格

### 2. 必填字段
始终包含：
- 条目类型的所有必填字段
- 现代文献（2000年后）的 DOI
- 文章的卷号和页码
- 书籍的出版商信息

### 3. 保护大小写
使用花括号包裹：
- 专有名词：`{AlphaFold}`
- 缩写词：`{DNA}`, `{CRISPR}`
- 化学式：`{H2O}`
- 名称：`{Python}`, `{R}`

### 4. 完整作者列表
尽可能列出所有作者：
- 少于10位作者时全列出
- 超过10位时用 "and others"
- 不要手动缩写为 "et al."

### 5. 使用标准条目类型
选择正确条目类型：
- 期刊文章 → `@article`
- 书籍 → `@book`
- 会议论文 → `@inproceedings`
- 预印本 → `@misc`

### 6. 语法验证
检查：
- 成对的花括号
- 字段后的逗号
- 唯一引用键
- 有效条目类型

### 7. 使用格式化工具
使用自动化工具：
```bash
python scripts/format_bibtex.py references.bib
```

优势：
- 统一格式
- 捕获语法错误
- 标准化字段顺序
- 修复常见问题

## 常见错误

### 1. 错误的分隔符
**错误**：
```bibtex
author = {Smith, J.; Doe, J.}    % 分号
author = {Smith, J., Doe, J.}    % 逗号
author = {Smith, J. & Doe, J.}   % &符号
```

**正确**：
```bibtex
author = {Smith, John and Doe, Jane}
```

### 2. 缺失逗号
**错误**：
```bibtex
@article{Smith2024,
  author = {Smith, John}    % 缺少逗号！
  title = {Title}
}
```

**正确**：
```bibtex
@article{Smith2024,
  author = {Smith, John},   % 每个字段后加逗号
  title = {Title}
}
```

### 3. 未保护的大小写
**错误**：
```bibtex
title = {Machine Learning with Python}
% "Python" 在标题格式中会变成 "python"
```

**正确**：
```bibtex
title = {Machine Learning with {Python}}
```

### 4. 页码使用单连字符
**错误**：
```bibtex
pages = {123-145}   % 单连字符
```

**正确**：
```bibtex
pages = {123--145}  % 双连字符（短破折号）
```

### 5. 冗余的 "pp." 标注
**错误**：
```bibtex
pages = {pp. 123--145}
```

**正确**：
```bibtex
pages = {123--145}
```

### 6. 带前缀的 DOI
**错误**：
```bibtex
doi = {https://doi.org/10.1038/nature12345}
doi = {doi:10.1038/nature12345}
```

**正确**：
```bibtex
doi = {10.1038/nature12345}
```

## 完整参考文献示例

```bibtex
% 期刊文章
@article{Jumper2021,
  author  = {Jumper, John and Evans, Richard and Pritzel, Alexander and others},
  title   = {Highly Accurate Protein Structure Prediction with {AlphaFold}},
  journal = {Nature},
  year    = {2021},
  volume  = {596},
  number  = {7873},
  pages   = {583--589},
  doi     = {10.1038/s41586-021-03819-2}
}

% 书籍
@book{Kumar2021,
  author    = {Kumar, Vinay and Abbas, Abul K. and Aster, Jon C.},
  title     = {Robbins and Cotran Pathologic Basis of Disease},
  publisher = {Elsevier},
  year      = {2021},
  edition   = {10},
  address   = {Philadelphia, PA},
  isbn      = {978-0-323-53113-9}
}

% 会议论文
@inproceedings{Vaswani2017,
  author    = {Vaswani, Ashish and Shazeer, Noam and Parmar, Niki and others},
  title     = {Attention is All You Need},
  booktitle = {Advances in Neural Information Processing Systems 30 (NeurIPS 2017)},
  year      = {2017},
  pages     = {5998--6008}
}

% 书籍章节
@incollection{Brown2020,
  author    = {Brown, Peter O. and Botstein, David},
  title     = {Exploring the New World of the Genome with {DNA} Microarrays},
  booktitle = {DNA Microarrays: A Molecular Cloning Manual},
  editor    = {Eisen, Michael B. and Brown, Patrick O.},
  publisher = {Cold Spring Harbor Laboratory Press},
  year      = {2020},
  pages     = {1--45}
}

% 博士论文
@phdthesis{Johnson2023,
  author  = {Johnson, Mary L.},
  title   = {Novel Approaches to Cancer Immunotherapy},
  school  = {Stanford University},
  year    = {2023},
  type    = {{PhD} dissertation}
}

% 预印本
@misc{Zhang2024,
  author       = {Zhang, Yi and Chen, Li and Wang, Hui},
  title        = {Novel Therapeutic Targets in {Alzheimer}'s Disease},
  year         = {2024},
  howpublished = {bioRxiv},
  doi          = {10.1101/2024.01.001},
  note         = {Preprint}
}

% 数据集
@misc{AlphaFoldDB2021,
  author       = {{DeepMind} and {EMBL-EBI}},
  title        = {{AlphaFold} Protein Structure Database},
  year         = {2021},
  howpublished = {Database},
  url          = {https://alphafold.ebi.ac.uk/},
  doi          = {10.1093/nar/gkab1061}
}
```

## 总结

BibTeX 格式核心要点：

✓ **选择正确条目类型** (@article, @book 等)  
✓ **包含所有必填字段**  
✓ **多位作者用 `and` 连接**  
✓ **用花括号保护大小写**  
✓ **页码范围用 `--`**  
✓ **现代文献包含 DOI**  
✓ **编译前验证语法**  

使用格式化工具确保一致性：
```bash
python scripts/format_bibtex.py references.bib
```

规范的 BibTeX 格式能确保所有文献样式中的引用准确统一！
