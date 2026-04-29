# 引文质量核对清单

使用此清单确保您的引文在最终提交前准确、完整且格式规范。

## 提交前核对清单

### ✓ 元数据准确性

- [ ] 所有作者姓名正确且格式规范
- [ ] 文章标题与实际出版物一致
- [ ] 期刊/会议名称完整（除非要求缩写）
- [ ] 出版年份准确
- [ ] 卷号和期号正确
- [ ] 页码范围准确

### ✓ 必填字段

- [ ] 所有@article条目包含：作者、标题、期刊、年份
- [ ] 所有@book条目包含：作者/编辑、标题、出版商、年份
- [ ] 所有@inproceedings条目包含：作者、标题、会议名称、年份
- [ ] 现代论文（2000年后）包含可用DOI
- [ ] 所有条目具有唯一引用键

### ✓ DOI验证

- [ ] 所有DOI格式规范（10.XXXX/...）
- [ ] DOI能正确解析到对应文章
- [ ] BibTeX字段不含DOI前缀（无"doi:"或"https://doi/"）
- [ ] CrossRef元数据与BibTeX条目匹配
- [ ] 运行：`python scripts/validate_citations.py references.bib --check-dois`

### ✓ 格式一致性

- [ ] 页码范围使用双连字符（--）而非单连字符（-）
- [ ] pages字段不含"pp."前缀
- [ ] 作者姓名使用"and"分隔符（非分号或&符号）
- [ ] 标题中大写受保护（{AlphaFold}、{CRISPR}等）
- [ ] 月份名称使用标准缩写（若包含）
- [ ] 引用键遵循统一格式

### ✓ 重复检测

- [ ] 参考文献中无重复DOI
- [ ] 无重复引用键
- [ ] 无近似重复标题
- [ ] 预印本已更新为正式发表版本（若可用）
- [ ] 运行：`python scripts/validate_citations.py references.bib`

### ✓ 特殊字符

- [ ] 带重音字符格式规范（如{\"u}表示ü）
- [ ] 数学符号使用LaTeX命令
- [ ] 化学式格式规范
- [ ] 无未转义特殊字符（%、&、$、#等）

### ✓ BibTeX语法

- [ ] 所有条目具有成对大括号{}
- [ ] 字段间用逗号分隔
- [ ] 每个条目末字段后无逗号
- [ ] 有效条目类型（@article、@book等）
- [ ] 运行：`python scripts/validate_citations.py references.bib`

### ✓ 文件组织

- [ ] 参考文献按逻辑顺序排列（按年份、作者或键）
- [ ] 全文格式统一
- [ ] 条目间无格式不一致
- [ ] 运行：`python scripts/format_bibtex.py references.bib --sort year`

## 自动化验证流程

### 步骤1：格式化清理

```bash
python scripts/format_bibtex.py references.bib \
  --deduplicate \
  --sort year \
  --descending \
  --output clean_references.bib
```

**功能说明**：
- 移除重复项
- 标准化格式
- 修复常见问题（页码范围、DOI格式等）
- 按年份降序排列（最新优先）

### 步骤2：验证

```bash
python scripts/validate_citations.py clean_references.bib \
  --check-dois \
  --report validation_report.json \
  --verbose
```

**功能说明**：
- 检查必填字段
- 验证DOI可解析
- 检测重复项
- 校验语法
- 生成详细报告

### 步骤3：审查报告

```bash
cat validation_report.json
```

**需处理项**：
- **错误**：必须修复（缺失字段、无效DOI、语法错误）
- **警告**：建议修复（缺失推荐字段、格式问题）
- **重复项**：移除或合并

### 步骤4：最终检查

```bash
python scripts/validate_citations.py clean_references.bib --verbose
```

**目标**：零错误，最少警告

## 人工审查清单

### 关键引文（前10-20篇重要文献）

对最重要引文进行人工验证：

- [ ] 访问DOI链接确认文章正确
- [ ] 对照实际出版物核对作者姓名
- [ ] 验证年份与出版日期匹配
- [ ] 确认期刊/会议名称正确
- [ ] 核对卷号/页码是否匹配

### 常见问题排查

**信息缺失**：
- [ ] 2000年后论文无DOI
- [ ] 期刊文章缺失卷号或页码
- [ ] 书籍缺失出版商信息
- [ ] 会议录缺失会议地点

**格式错误**：
- [ ] 页码范围用单连字符（123-145 → 123--145）
- [ ] 作者列表含&符号（Smith & Jones → Smith and Jones）
- [ ] 标题中未保护缩写（DNA → {DNA}）
- [ ] DOI含URL前缀（https://doi.org/10.xxx → 10.xxx）

**元数据不匹配**：
- [ ] 作者姓名与出版物不符
- [ ] 年份使用在线优先而非印刷版
- [ ] 期刊名称应完整却缩写
- [ ] 卷号/期号位置颠倒

**重复项**：
- [ ] 相同论文使用不同引用键
- [ ] 同时引用预印本和正式版
- [ ] 同时引用会议论文和期刊版

## 领域专项检查

### 生物医学

- [ ] 包含PubMed Central ID（PMCID）（若可用）
- [ ] MeSH术语恰当（若使用）
- [ ] 包含临床试验注册号（若适用）
- [ ] 所有治疗/药物引用准确

### 计算机科学

- [ ] 预印本包含arXiv ID
- [ ] 会议录引用规范（非仅"NeurIPS"）
- [ ] 软件/数据集引用含版本号
- [ ] GitHub链接稳定永久

### 通用科学

- [ ] 数据可用性声明引用规范
- [ ] 已撤稿论文已识别并移除
- [ ] 预印本已检查正式发表版
- [ ] 关键性补充材料已引用

## 最终提交前步骤

### 提交前1周

- [ ] 运行含DOI检查的完整验证
- [ ] 修复所有错误和严重警告
- [ ] 人工验证前10-20篇关键引文
- [ ] 检查是否存在撤稿论文

### 提交前3天

- [ ] 人工编辑后重新验证
- [ ] 确保所有正文引用对应参考文献条目
- [ ] 确保所有参考文献条目在正文中被引用
- [ ] 核对引用格式是否符合期刊要求

### 提交前1天

- [ ] 最终验证检查
- [ ] LaTeX编译成功无警告
- [ ] PDF正确渲染所有引文
- [ ] 参考文献格式正确
- [ ] 无占位引用（Smith et al. XXXX）

### 提交当日

- [ ] 最后一次验证运行
- [ ] 最终修改后必须重新验证
- [ ] 提交包中包含参考文献文件
- [ ] 正文引用的图表与参考文献匹配

## 质量评级标准

### 优秀参考文献

- ✓ 现代论文100%含DOI
- ✓ 零验证错误
- ✓ 零缺失必填字段
- ✓ 零无效DOI
- ✓ 零重复项
- ✓ 全文格式统一
- ✓ 所有引文经人工抽检

### 合格参考文献

- ✓ 90%以上现代条目含DOI
- ✓ 零严重错误
- ✓ 仅轻微警告（如缺失推荐字段）
- ✓ 关键引文经人工验证
- ✓ 编译无错误通过

### 需改进参考文献

- ✗ 近期论文缺失DOI
- ✗ 存在严重验证错误
- ✗ DOI无效或错误
- ✗ 存在重复条目
- ✗ 格式不一致
- ✗ 编译出现警告或错误

## 紧急修复指南

### DOI无效

```bash
# 查找正确DOI
# 方法1：检索CrossRef
# https://www.crossref.org/

# 方法2：出版商网站检索
# 方法3：Google学术搜索

# 重新提取元数据
python scripts/extract_metadata.py --doi CORRECT_DOI
```

### 信息缺失

```bash
# 通过DOI提取
python scripts/extract_metadata.py --doi 10.xxxx/yyyy

# 或通过PMID（生物医学）
python scripts/extract_metadata.py --pmid 12345678

# 或通过arXiv
python scripts/extract_metadata.py --arxiv 2103.12345
```

### 重复条目

```bash
# 自动去重
python scripts/format_bibtex.py references.bib \
  --deduplicate \
  --output fixed_references.bib
```

### 格式错误

```bash
# 自动修复常见问题
python scripts/format_bibtex.py references.bib \
  --output fixed_references.bib

# 重新验证
python scripts/validate_citations.py fixed_references.bib
```

## 长期最佳实践

### 研究过程中

- [ ] 发现文献时即时添加至参考文献文件
- [ ] 立即通过DOI提取元数据
- [ ] 每添加10-20条后验证
- [ ] 参考文献文件纳入版本控制

### 写作过程中

- [ ] 边写边引用
- [ ] 使用统一引用键
- [ ] 勿延迟添加参考文献
- [ ] 每周验证

### 提交前准备

- [ ] 预留2-3天清理引文
- [ ] 勿拖延至最后一天
- [ ] 尽可能自动化处理
- [ ] 人工验证关键引文

## 工具速查手册

### 元数据提取

```bash
# 通过DOI提取
python scripts/doi_to_bibtex.py 10.1038/nature12345

# 多源提取
python scripts/extract_metadata.py \
  --doi 10.1038/nature12345 \
  --pmid 12345678 \
  --arxiv 2103.12345 \
  --output references.bib
```

### 验证操作

```bash
# 基础验证
python scripts/validate_citations.py references.bib

# DOI检查（耗时但彻底）
python scripts/validate_citations.py references.bib --check-dois

# 生成报告
python scripts/validate_citations.py references.bib \
  --report validation.json \
  --verbose
```

### 格式化清理

```bash
# 格式化修复问题
python scripts/format_bibtex.py references.bib

# 去重排序
python scripts/format_bibtex.py references.bib \
  --deduplicate \
  --sort year \
  --descending \
  --output clean_refs.bib
```

## 执行摘要

**最低要求**：
1. 运行`format_bibtex.py --deduplicate`
2. 运行`validate_citations.py`
3. 修复所有错误
4. 成功编译

**推荐流程**：
1. 格式化、去重、排序
2. 使用`--check-dois`验证
3. 修复所有错误和警告
4. 人工验证关键引文
5. 修复后重新验证

**最佳实践**：
1. 研究全程持续验证
2. 坚持使用自动化工具
3. 保持参考文献整洁有序
4. 记录特殊案例
5. 提交前1-3天完成最终验证

**重要提示**：引文错误将影响学术声誉，投入时间确保准确性绝对值得！
