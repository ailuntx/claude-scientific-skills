# 引文验证指南

BibTeX 文件中验证引文准确性、完整性和格式的综合指南。

## 概述

引文验证确保：
- 所有引文准确完整
- DOI 可正确解析
- 必填字段齐全
- 无重复条目
- 格式与语法规范
- 链接可访问

应在以下时机执行验证：
- 提取元数据后
- 稿件提交前
- 手动编辑 BibTeX 文件后
- 维护参考文献库时定期执行

## 验证类别

### 1. DOI 验证

**目的**：确保 DOI 有效且可正确解析。

#### 检查内容

**DOI 格式**：
```
有效：   10.1038/s41586-021-03819-2
有效：   10.1126/science.aam9317
无效：   10.1038/invalid
无效：   doi:10.1038/... (BibTeX 中应省略 "doi:" 前缀)
```

**DOI 解析**：
- 应能通过 https://doi.org/ 解析
- 应重定向至实际文章
- 不应返回 404 或错误

**元数据一致性**：
- CrossRef 元数据应与 BibTeX 匹配
- 作者姓名应一致
- 标题应匹配
- 年份应匹配

#### 验证方法

**手动检查**：
1. 从 BibTeX 复制 DOI
2. 访问 https://doi.org/10.1038/nature12345
3. 验证重定向至正确文章
4. 检查元数据是否匹配

**自动检查**（推荐）：
```bash
python scripts/validate_citations.py references.bib --check-dois
```

**流程**：
1. 从 BibTeX 文件提取所有 DOI
2. 向 doi.org 解析器逐个查询
3. 向 CrossRef API 查询元数据
4. 将元数据与 BibTeX 条目对比
5. 报告差异

#### 常见问题

**失效 DOI**：
- DOI 拼写错误
- 出版商更改 DOI（罕见）
- 文章被撤回
- 解决方案：从出版商网站查找正确 DOI

**元数据不匹配**：
- BibTeX 包含过时/错误信息
- 解决方案：从 CrossRef 重新提取元数据

**缺失 DOI**：
- 早期文章可能无 DOI
- 2000 年前出版物可接受
- 改用 URL 或 PMID

### 2. 必填字段

**目的**：确保所有必要信息齐全。

#### 按条目类型要求

**@article**：
```bibtex
author   % 必填
title    % 必填
journal  % 必填
year     % 必填
volume   % 强烈推荐
pages    % 强烈推荐
doi      % 现代论文强烈推荐
```

**@book**：
```bibtex
author 或 editor  % 必填（至少一项）
title            % 必填
publisher        % 必填
year             % 必填
isbn             % 推荐
```

**@inproceedings**：
```bibtex
author     % 必填
title      % 必填
booktitle  % 必填（会议/论文集名称）
year       % 必填
pages      % 推荐
```

**@incollection**（图书章节）：
```bibtex
author     % 必填
title      % 必填（章节标题）
booktitle  % 必填（书名）
publisher  % 必填
year       % 必填
editor     % 推荐
pages      % 推荐
```

**@phdthesis**：
```bibtex
author  % 必填
title   % 必填
school  % 必填
year    % 必填
```

**@misc**（预印本、数据集等）：
```bibtex
author  % 必填
title   % 必填
year    % 必填
howpublished  % 推荐（bioRxiv, Zenodo 等）
doi 或 url    % 至少一项必填
```

#### 验证脚本

```bash
python scripts/validate_citations.py references.bib --check-required-fields
```

**输出**：
```
错误：条目 'Smith2024' 缺失必填字段 'journal'
错误：条目 'Doe2023' 缺失必填字段 'year'
警告：条目 'Jones2022' 缺失推荐字段 'volume'
```

### 3. 作者姓名格式

**目的**：确保作者姓名格式一致且正确。

#### 正确格式

**推荐 BibTeX 格式**：
```bibtex
author = {Last1, First1 and Last2, First2 and Last3, First3}
```

**示例**：
```bibtex
% 正确
author = {Smith, John}
author = {Smith, John A.}
author = {Smith, John Andrew}
author = {Smith, John and Doe, Jane}
author = {Smith, John and Doe, Jane and Johnson, Mary}

% 多位作者
author = {Smith, John and Doe, Jane and others}

% 错误
author = {John Smith}  % 名+姓格式（不推荐）
author = {Smith, J.; Doe, J.}  % 分号分隔符（错误）
author = {Smith J, Doe J}  % 缺失逗号
```

#### 特殊情况

**后缀（Jr., III 等）**：
```bibtex
author = {King, Jr., Martin Luther}
```

**复姓（带连字符）**：
```bibtex
author = {Smith-Jones, Mary}
```

**Van, von, de 等**：
```bibtex
author = {van der Waals, Johannes}
author = {de Broglie, Louis}
```

**机构作者**：
```bibtex
author = {{World Health Organization}}
% 双花括号视为单个作者
```

#### 验证检查

**自动验证**：
```bash
python scripts/validate_citations.py references.bib --check-authors
```

**检查项**：
- 正确分隔符（and，而非 &, ; 等）
- 逗号位置
- 空作者字段
- 格式错误姓名

### 4. 数据一致性

**目的**：确保所有字段包含有效合理值。

#### 年份验证

**有效年份**：
```bibtex
year = {2024}    % 当前/近期
year = {1953}    % Watson & Crick DNA 结构（历史）
year = {1665}    % Hooke 的《显微图谱》（早期）
```

**无效年份**：
```bibtex
year = {24}      % 两位数（歧义）
year = {202}     % 笔误
year = {2025}    % 未来年份（除非已录用/印刷中）
year = {0}       % 明显错误
```

**检查项**：
- 四位数年份
- 合理范围（1600-当前年份+1）
- 非全零值

#### 卷号/期号验证

```bibtex
volume = {123}      % 数字
volume = {12}       % 有效
number = {3}        % 有效
number = {S1}       % 增刊（有效）
```

**无效**：
```bibtex
volume = {Vol. 123}  % 应仅为数字
number = {Issue 3}   % 应仅为数字
```

#### 页码范围验证

**正确格式**：
```bibtex
pages = {123--145}    % 短破折号（两个连字符）
pages = {e0123456}    % PLOS 风格文章 ID
pages = {123}         % 单页
```

**错误格式**：
```bibtex
pages = {123-145}     % 单连字符（应使用 --）
pages = {pp. 123-145} % 删除 "pp."
pages = {123–145}     % Unicode 短破折号（可能导致问题）
```

#### URL 验证

**检查项**：
- URL 可访问（返回 200 状态）
- 优先使用 HTTPS
- 无明显拼写错误
- 永久链接（非临时）

**有效**：
```bibtex
url = {https://www.nature.com/articles/nature12345}
url = {https://arxiv.org/abs/2103.14030}
```

**存疑**：
```bibtex
url = {http://...}  % HTTP 而非 HTTPS
url = {file:///...} % 本地文件路径
url = {bit.ly/...}  % URL 短链接（非永久）
```

### 5. 重复项检测

**目的**：查找并删除重复条目。

#### 重复类型

**完全重复**（相同 DOI）：
```bibtex
@article{Smith2024a,
  doi = {10.1038/nature12345},
  ...
}

@article{Smith2024b,
  doi = {10.1038/nature12345},  % 相同 DOI！
  ...
}
```

**近似重复**（相似标题/作者）：
```bibtex
@article{Smith2024,
  title = {Machine Learning for Drug Discovery},
  ...
}

@article{Smith2024method,
  title = {Machine learning for drug discovery},  % 相同，大小写不同
  ...
}
```

**预印本+已发表**：
```bibtex
@misc{Smith2023arxiv,
  title = {AlphaFold Results},
  howpublished = {arXiv},
  ...
}

@article{Smith2024,
  title = {AlphaFold Results},  % 同一论文，现已发表
  journal = {Nature},
  ...
}
% 仅保留已发表版本
```

#### 检测方法

**按 DOI**（最可靠）：
- 相同 DOI = 完全重复
- 保留一项，删除其余

**按标题相似度**：
- 标准化：小写化，删除标点
- 计算相似度（如 Levenshtein 距离）
- >90% 相似则标记

**按作者-年份-标题**：
- 相同第一作者 + 年份 + 相似标题
- 可能为重复项

**自动检测**：
```bash
python scripts/validate_citations.py references.bib --check-duplicates
```

**输出**：
```
警告：可能存在重复条目：
  - Smith2024a (DOI: 10.1038/nature12345)
  - Smith2024b (DOI: 10.1038/nature12345)
  建议：保留一项，删除其余。
```

### 6. 格式与语法

**目的**：确保 BibTeX 语法有效。

#### 常见语法错误

**缺失逗号**：
```bibtex
@article{Smith2024,
  author = {Smith, John}   % 缺失逗号！
  title = {Title}
}
% 应为：
  author = {Smith, John},  % 每个字段后加逗号
```

**花括号未闭合**：
```bibtex
title = {Title with {Protected} Text  % 缺失闭合花括号
% 应为：
title = {Title with {Protected} Text}
```

**条目缺失闭合花括号**：
```bibtex
@article{Smith2024,
  author = {Smith, John},
  title = {Title}
  % 缺失闭合花括号！
% 应以 } 结尾
}
```

**键名含无效字符**：
```bibtex
@article{Smith&Doe2024,  % 键名不允许 &
  ...
}
% 应改为：
@article{SmithDoe2024,
  ...
}
```

#### BibTeX 语法规则

**条目结构**：
```bibtex
@TYPE{citationkey,
  field1 = {value1},
  field2 = {value2},
  ...
  fieldN = {valueN}
}
```

**引用键**：
- 字母数字及部分标点（-, _, ., :）
- 无空格
- 大小写敏感
- 文件内唯一

**字段值**：
- 用 {花括号} 或 "引号" 包裹
- 复杂文本建议用花括号
- 数字可不加引号：`year = 2024`

**特殊字符**：
- `{` 和 `}` 用于分组
- `\` 用于 LaTeX 命令
- 保护大写：`{AlphaFold}`
- 重音符号：`{\"u}`, `{\'e}`, `{\aa}`

#### 验证

```bash
python scripts/validate_citations.py references.bib --check-syntax
```

**检查项**：
- 有效 BibTeX 结构
- 花括号闭合
- 逗号使用正确
- 有效条目类型
- 唯一引用键

## 验证工作流

### 步骤 1：基础验证

执行全面验证：

```bash
python scripts/validate_citations.py references.bib
```

**检查所有项**：
- DOI 解析
- 必填字段
- 作者格式
- 数据一致性
- 重复项
- 语法

### 步骤 2：审查报告

检查验证报告：

```json
{
  "total_entries": 150,
  "valid_entries": 140,
  "errors": [
    {
      "entry": "Smith2024",
      "error": "missing_required_field",
      "field": "journal",
      "severity": "high"
    },
    {
      "entry": "Doe2023",
      "error": "invalid_doi",
      "doi": "10.1038/broken",
      "severity": "high"
    }
  ],
  "warnings": [
    {
      "entry": "Jones2022",
      "warning": "missing_recommended_field",
      "field": "volume",
      "severity": "medium"
    }
  ],
  "duplicates": [
    {
      "entries": ["Smith2024a", "Smith2024b"],
      "reason": "same_doi",
      "doi": "10.1038/nature12345"
    }
  ]
}
```

### 步骤 3：修复问题

**高优先级**（错误）：
1. 补充缺失必填字段
2. 修复失效 DOI
3. 删除重复项
4. 修正语法错误

**中优先级**（警告）：
1. 补充推荐字段
2. 改进作者格式
3. 修正页码范围

**低优先级**：
1. 标准化格式
2. 添加可访问 URL

### 步骤 4：自动修复

对安全修正使用自动修复：

```bash
python scripts/validate_citations.py references.bib \
  --auto-fix \
  --output fixed_references.bib
```

**自动修复可处理**：
- 修正页码范围格式（- 改为 --）
- 删除 pages 中的 "pp."
- 标准化作者分隔符
- 修复常见语法错误
- 规范化字段顺序

**自动修复无法处理**：
- 补充缺失信息
- 查找正确 DOI
- 决定保留哪个重复项
- 修复语义错误

### 步骤 5：人工审查

审查自动修复后的文件：
```bash
# 检查变更内容
diff references.bib fixed_references.bib

# 审查含错误的条目
grep -A 10 "Smith2024" fixed_references.bib
```

### 步骤 6：重新验证

修复后重新验证：

```bash
python scripts/validate_citations.py fixed_references.bib --verbose
```

应显示：
```
✓ 所有 DOI 有效
✓ 所有必填字段齐全
✓ 未发现重复项
✓ 语法有效
✓ 150/150 条目有效
```

## 验证清单

最终提交前使用此清单：

### DOI 验证
- [ ] 所有 DOI 解析正确
- [ ] BibTeX 与 CrossRef 元数据匹配
- [ ] 无失效或无效 DOI

### 完整性
- [ ] 所有条目含必填字段
- [ ] 现代论文（2000+）含 DOI
- [ ] 作者格式正确
- [ ] 期刊/会议名称规范

### 一致性
- [ ] 年份为 4 位数字
- [ ] 页码范围使用 -- 而非 -
- [ ] 卷号/期号为数字
- [ ] URL 可访问

### 重复项
- [ ] 无相同 DOI 条目
- [ ] 无近似重复标题
- [ ] 预印本已更新为发表版本

### 格式
- [ ] BibTeX 语法有效
- [ ] 花括号闭合
- [ ] 逗号使用正确
- [ ] 引用键唯一

### 最终检查
- [ ] 参考文献库编译无报错
- [ ] 正文所有引用在参考文献中列出
- [ ] 参考文献所有条目在正文中被引用
- [ ] 引文格式符合期刊要求

## 最佳实践

### 1. 尽早且频繁验证

```bash
# 提取元数据后
python scripts/extract_metadata.py --doi ... --output refs.bib
python scripts/validate

### 4. 优先处理高优先级问题

**优先级顺序**：
1. 语法错误（阻止编译）
2. 缺失必填字段（引用不完整）
3. 失效的DOI（链接损坏）
4. 重复项（导致混淆和空间浪费）
5. 缺失推荐字段
6. 格式不一致

### 5. 记录例外情况

对于无法修复的条目：

```bibtex
@article{Old1950,
  author = {Smith, John},
  title = {标题},
  journal = {冷门期刊},
  year = {1950},
  volume = {12},
  pages = {34--56},
  note = {2000年前出版物无DOI可用}
}
```

### 6. 根据期刊要求验证

不同期刊有不同要求：
- 引用格式（编号制、作者-年份制）
- 缩写规则（期刊名称）
- 最大参考文献数量
- 格式要求（BibTeX、EndNote、手动）

务必查阅期刊作者指南！

## 常见验证问题

### 问题1：元数据不匹配

**现象**：BibTeX显示2023年，CrossRef显示2024年。

**原因**：
- 在线优先与印刷版差异
- 修正/更新版本
- 提取错误

**解决方案**：
1. 核对原始文章
2. 采用最新/准确日期
3. 更新BibTeX条目
4. 重新验证

### 问题2：特殊字符

**现象**：LaTeX编译因特殊字符失败。

**原因**：
- 带重音字符（é, ü, ñ）
- 化学式（H₂O）
- 数学符号（α, β, ±）

**解决方案**：
```bibtex
% 使用LaTeX命令
author = {M{\"u}ller, Hans}  % Müller
title = {Study of H\textsubscript{2}O}  % H₂O
% 或配合LaTeX包使用UTF-8编码
```

### 问题3：提取不完整

**现象**：提取的元数据缺失字段。

**原因**：
- 数据源未提供完整元数据
- 提取过程出错
- 记录本身不完整

**解决方案**：
1. 核对原始文章
2. 手动添加缺失字段
3. 使用替代数据源（如PubMed与CrossRef）

### 问题4：无法识别重复项

**现象**：同一论文重复出现但未被检测。

**原因**：
- 不同DOI（应属罕见）
- 标题差异（缩写、拼写错误）
- 不同引用键

**解决方案**：
- 手动搜索作者+年份
- 检查相似标题
- 手动删除重复项

## 总结

验证确保引用质量：

✓ **准确性**：DOI可解析，元数据正确  
✓ **完整性**：所有必填字段齐全  
✓ **一致性**：全文格式统一  
✓ **无重复**：每篇论文仅引用一次  
✓ **语法有效**：BibTeX编译无报错  

**最终提交前务必验证！**

推荐自动化工具：
```bash
python scripts/validate_citations.py references.bib
```

标准工作流：
1. 提取元数据
2. 验证
3. 修复错误
4. 重新验证
5. 提交
