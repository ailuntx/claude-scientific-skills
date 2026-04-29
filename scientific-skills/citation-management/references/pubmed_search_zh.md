# PubMed 检索指南

生物医学与生命科学文献检索的全面指南，涵盖 MeSH 术语、字段标签、高级检索策略及 E-utilities API 使用。

## 概述

PubMed 是生物医学文献的首要数据库：
- **覆盖范围**：3500 万+ 条引文
- **学科领域**：生物医学与生命科学
- **来源**：MEDLINE、生命科学期刊、在线图书
- **权威性**：由美国国家医学图书馆（NLM）/ NCBI 维护
- **访问方式**：免费开放，无需账户
- **更新频率**：每日新增引文
- **数据质量**：高价值元数据，MeSH 标引

## 基础检索

### 简单关键词检索

PubMed 自动将术语映射至 MeSH 并检索多字段：

```
diabetes
CRISPR gene editing
Alzheimer's disease treatment
cancer immunotherapy
```

**自动功能**：
- 自动 MeSH 映射
- 单复数变体识别
- 缩写扩展
- 拼写检查

### 精确短语检索

使用引号进行精确短语检索：

```
"CRISPR-Cas9"
"systematic review"
"randomized controlled trial"
"machine learning"
```

## MeSH（医学主题词表）

### 什么是 MeSH？

MeSH 是用于标引生物医学文献的受控词表：
- **层级结构**：树状分类体系
- **标引一致性**：相同概念始终使用相同标签
- **全面覆盖**：涵盖疾病、药物、解剖学、技术等
- **专业标引**：由 NLM 标引入员分配 MeSH 术语

### 查找 MeSH 术语

**MeSH 浏览器**：https://meshb.nlm.nih.gov/search

**示例**：
```
检索："heart attack"
MeSH 术语："Myocardial Infarction"
```

**在 PubMed 中操作**：
1. 使用关键词检索
2. 查看左侧边栏的 "MeSH Terms"
3. 选择相关 MeSH 术语
4. 添加到检索式

### 在检索中使用 MeSH

**基础 MeSH 检索**：
```
"Diabetes Mellitus"[MeSH]
"CRISPR-Cas Systems"[MeSH]
"Alzheimer Disease"[MeSH]
"Neoplasms"[MeSH]
```

**带副主题词的 MeSH 检索**：
```
"Diabetes Mellitus/drug therapy"[MeSH]
"Neoplasms/genetics"[MeSH]
"Heart Failure/prevention and control"[MeSH]
```

**常用副主题词**：
- `/drug therapy`：药物治疗
- `/diagnosis`：诊断方面
- `/genetics`：遗传学方面
- `/epidemiology`：发生与分布
- `/prevention and control`：预防方法
- `/etiology`：病因
- `/surgery`：手术治疗
- `/metabolism`：代谢方面

### MeSH 扩展检索

默认 MeSH 检索包含下位词（扩展检索）：

```
"Neoplasms"[MeSH]
# 包含：乳腺癌、肺癌等
```

**禁用扩展**（仅精确术语）：
```
"Neoplasms"[MeSH:NoExp]
```

### MeSH 主要主题

仅检索 MeSH 术语为主要焦点的文献：

```
"Diabetes Mellitus"[MeSH Major Topic]
# 仅包含以糖尿病为核心主题的论文
```

## 字段标签

字段标签指定检索记录的具体部分。

### 常用字段标签

**标题与摘要**：
```
cancer[Title]                    # 仅标题
treatment[Title/Abstract]        # 标题或摘要
"machine learning"[Title/Abstract]
```

**作者**：
```
"Smith J"[Author]
"Doudna JA"[Author]
"Collins FS"[Author]
```

**作者全名**：
```
"Smith, John"[Full Author Name]
```

**期刊**：
```
"Nature"[Journal]
"Science"[Journal]
"New England Journal of Medicine"[Journal]
"Nat Commun"[Journal]           # 缩写形式
```

**出版日期**：
```
2023[Publication Date]
2020:2024[Publication Date]      # 日期范围
2023/01/01:2023/12/31[Publication Date]
```

**创建日期**：
```
2023[Date - Create]              # 添加到 PubMed 的时间
```

**出版物类型**：
```
"Review"[Publication Type]
"Clinical Trial"[Publication Type]
"Meta-Analysis"[Publication Type]
"Randomized Controlled Trial"[Publication Type]
```

**语言**：
```
English[Language]
French[Language]
```

**DOI**：
```
10.1038/nature12345[DOI]
```

**PMID（PubMed ID）**：
```
12345678[PMID]
```

**文章 ID**：
```
PMC1234567[PMC]                  # PubMed Central ID
```

### 不常用但实用的标签

```
humans[MeSH Terms]               # 仅人类研究
animals[MeSH Terms]              # 仅动物研究
"United States"[Place of Publication]
nih[Grant Number]                # NIH 资助研究
"Female"[Sex]                    # 女性受试者
"Aged, 80 and over"[Age]        # 高龄受试者
```

## 布尔运算符

使用布尔逻辑组合检索词。

### AND

必须同时包含两个术语（默认行为）：

```
diabetes AND treatment
"CRISPR-Cas9" AND "gene editing"
cancer AND immunotherapy AND "clinical trial"[Publication Type]
```

### OR

包含任一术语即可：

```
"heart attack" OR "myocardial infarction"
diabetes OR "diabetes mellitus"
CRISPR OR Cas9 OR "gene editing"
```

**应用场景**：同义词和相关术语

### NOT

排除术语：

```
cancer NOT review
diabetes NOT animal
"machine learning" NOT "deep learning"
```

**注意**：可能排除同时提及两个术语的相关文献。

### 组合运算符

使用括号实现复杂逻辑：

```
(diabetes OR "diabetes mellitus") AND (treatment OR therapy)

("CRISPR" OR "gene editing") AND ("therapeutic" OR "therapy") 
  AND 2020:2024[Publication Date]

(cancer OR neoplasm) AND (immunotherapy OR "immune checkpoint inhibitor") 
  AND ("clinical trial"[Publication Type] OR "randomized controlled trial"[Publication Type])
```

## 高级检索构建器

**访问地址**：https://pubmed.ncbi.nlm.nih.gov/advanced/

**功能**：
- 可视化查询构建器
- 支持多查询框
- 下拉菜单选择字段标签
- 使用 AND/OR/NOT 组合
- 结果预览
- 显示最终查询字符串
- 保存查询

**工作流程**：
1. 在独立框中添加检索词
2. 选择字段标签
3. 选择布尔运算符
4. 预览结果
5. 按需优化
6. 复制最终查询字符串
7. 用于脚本或保存

**构建示例**：
```
#1: "Diabetes Mellitus, Type 2"[MeSH]
#2: "Metformin"[MeSH]
#3: "Clinical Trial"[Publication Type]
#4: 2020:2024[Publication Date]
#5: #1 AND #2 AND #3 AND #4
```

## 过滤器与限定条件

### 文章类型

```
"Review"[Publication Type]
"Systematic Review"[Publication Type]
"Meta-Analysis"[Publication Type]
"Clinical Trial"[Publication Type]
"Randomized Controlled Trial"[Publication Type]
"Case Reports"[Publication Type]
"Comparative Study"[Publication Type]
```

### 物种

```
humans[MeSH Terms]
mice[MeSH Terms]
rats[MeSH Terms]
```

### 性别

```
"Female"[MeSH Terms]
"Male"[MeSH Terms]
```

### 年龄组

```
"Infant"[MeSH Terms]
"Child"[MeSH Terms]
"Adolescent"[MeSH Terms]
"Adult"[MeSH Terms]
"Aged"[MeSH Terms]
"Aged, 80 and over"[MeSH Terms]
```

### 文本可获取性

```
free full text[Filter]           # 可获取免费全文
```

### 期刊类别

```
"Journal Article"[Publication Type]
```

## E-utilities API

NCBI 通过 E-utilities（Entrez 编程工具）提供程序化访问接口。

### 概述

**基础 URL**：`https://eutils.ncbi.nlm.nih.gov/entrez/eutils/`

**主要工具**：
- **ESearch**：检索 PMID
- **EFetch**：获取完整记录
- **ESummary**：获取文档摘要
- **ELink**：查找相关文献
- **EInfo**：数据库统计

**无需 API 密钥**，但建议使用：
- 更高请求频率（10次/秒 vs 3次/秒）
- 更优性能
- 项目标识

**获取 API 密钥**：https://www.ncbi.nlm.nih.gov/account/

### ESearch - 检索 PubMed

获取查询结果的 PMID。

**端点**：`/esearch.fcgi`

**参数**：
- `db`：数据库（pubmed）
- `term`：检索式
- `retmax`：最大结果数（默认20，上限10000）
- `retstart`：起始位置（分页用）
- `sort`：排序方式（relevance, pub_date, author）
- `api_key`：API 密钥（可选但推荐）

**示例 URL**：
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?
  db=pubmed&
  term=diabetes+AND+treatment&
  retmax=100&
  retmode=json&
  api_key=YOUR_API_KEY
```

**响应**：
```json
{
  "esearchresult": {
    "count": "250000",
    "retmax": "100",
    "idlist": ["12345678", "12345679", ...]
  }
}
```

### EFetch - 获取记录

获取 PMID 的完整元数据。

**端点**：`/efetch.fcgi`

**参数**：
- `db`：数据库（pubmed）
- `id`：逗号分隔的 PMID
- `retmode`：格式（xml, json, text）
- `rettype`：类型（abstract, medline, full）
- `api_key`：API 密钥

**示例 URL**：
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?
  db=pubmed&
  id=12345678,12345679&
  retmode=xml&
  api_key=YOUR_API_KEY
```

**响应**：包含完整元数据的 XML，包括：
- 标题
- 作者（含所属机构）
- 摘要
- 期刊
- 出版日期
- DOI
- PMID, PMCID
- MeSH 术语
- 关键词

### ESummary - 获取摘要

比 EFetch 更轻量化的替代方案。

**示例**：
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?
  db=pubmed&
  id=12345678&
  retmode=json&
  api_key=YOUR_API_KEY
```

**返回**：不含完整摘要的关键元数据。

### ELink - 查找相关文献

查找相关文献或跨数据库链接。

**示例**：
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/elink.fcgi?
  dbfrom=pubmed&
  db=pubmed&
  id=12345678&
  linkname=pubmed_pubmed_citedin
```

**链接类型**：
- `pubmed_pubmed`：相关文献
- `pubmed_pubmed_citedin`：引用本文的文献
- `pubmed_pmc`：PMC 全文版本
- `pubmed_protein`：相关蛋白质记录

### 频率限制

**无 API 密钥**：
- 每秒 3 次请求
- 超限将被阻断

**使用 API 密钥**：
- 每秒 10 次请求
- 更适合程序化访问

**最佳实践**：
```python
import time
time.sleep(0.34)  # 约每秒3次请求
# 或
time.sleep(0.11)  # 使用API密钥时约每秒10次请求
```

### API 密钥使用

**获取 API 密钥**：
1. 创建 NCBI 账户：https://www.ncbi.nlm.nih.gov/account/
2. 设置 → API 密钥管理
3. 创建新 API 密钥
4. 复制密钥

**在请求中使用**：
```
&api_key=YOUR_API_KEY_HERE
```

**安全存储**：
```bash
# 环境变量存储
export NCBI_API_KEY="your_key_here"

# 脚本中调用
import os
api_key = os.getenv('NCBI_API_KEY')
```

## 检索策略

### 系统性全面检索

适用于系统综述和荟萃分析：

```
# 1. 确定核心概念
概念1：糖尿病
概念2：治疗
概念3：结局指标

# 2. 查找 MeSH 术语及同义词
概念1："Diabetes Mellitus"[MeSH] OR diabetes OR diabetic
概念2："Drug Therapy"[MeSH] OR treatment OR therapy OR medication
概念3："Treatment Outcome"[MeSH] OR outcome OR efficacy OR effectiveness

# 3. 使用 AND 组合
("Diabetes Mellitus"[MeSH] OR diabetes OR diabetic) 
  AND ("Drug Therapy"[MeSH] OR treatment OR therapy OR medication)
  AND ("Treatment Outcome"[MeSH] OR outcome OR efficacy OR effectiveness)

# 4. 添加过滤器
AND 2015:2024[Publication Date]
AND ("Clinical Trial"[Publication Type] OR "Randomized Controlled Trial"[Publication Type])
AND English[Language]
AND humans[MeSH Terms]
```

### 查找临床试验

```
# 特定疾病 + 临床试验
"Alzheimer Disease"[MeSH] 
  AND ("Clinical Trial"[Publication Type] 
       OR "Randomized Controlled Trial"[Publication Type])
  AND 2020:2024[Publication Date]

# 特定药物试验
"Metformin"[MeSH] 
  AND "Diabetes Mellitus, Type 2"[MeSH]
  AND "Randomized Controlled Trial"[Publication Type]
```

### 查找综述文献

```
# 主题系统综述
"CRISPR-Cas Systems"[MeSH] 
  AND ("Systematic Review"[Publication Type] OR "Meta-Analysis"[Publication Type])

# 高影响力期刊综述
cancer immunotherapy 
  AND "Review"[Publication Type]
  AND ("Nature"[Journal] OR "Science"[Journal] OR "Cell"[Journal])
```

### 查找近期论文

```
# 过去一年的论文
"machine learning"[Title/Abstract] 
  AND "drug discovery"[Title/Abstract]
  AND 2024[Publication Date]

# 特定期刊近期论文
"CRISPR"[Title/Abstract] 
  AND "Nature"[Journal]
  AND 2023:2024[Publication Date]
```

### 作者追踪

```
# 特定作者近期成果
"Doudna JA"[Author] AND 2020:2024[Publication Date]

# 作者 + 主题
"Church GM"[Author] AND "synthetic biology"[Title/Abstract]
```

### 高质量证据检索

```
# 荟萃分析与系统综述
(diabetes OR "diabetes mellitus") 
  AND (treatment OR therapy)
  AND ("Meta-Analysis"[Publication Type] OR "Systematic Review"[Publication Type])

# 仅随机对照试验
cancer immunotherapy 
  AND "Randomized Controlled Trial"[Publication Type]
  AND 2020:2024[Publication Date]
```

## 脚本集成

### search_pubmed.py 使用说明

**基础检索**：
```bash
python scripts/search_pubmed.py "diabetes treatment"
```

**使用 MeSH 术语**：
```bash
python scripts/search_pubmed.py \
  --query '"Diabetes Mellitus"[MeSH] AND "Drug Therapy"[MeSH]'
```

**日期范围过滤**：
```bash
python scripts/search_pubmed.py "CRISPR" \
  --date-start 2020-01-01 \
  --date-end 2024-12-31 \
  --limit 200
```

**出版物类型过滤**：
```bash
python scripts/search_pubmed.py "cancer immunotherapy" \
  --publication-types "Clinical Trial,Randomized Controlled Trial" \
  --limit 100
```

**导出为 BibTeX**：
```bash
python scripts/search_pubmed.py "Alzheimer's disease" \
  --limit 100 \
  --format bibtex \
  --output alzheimers.bib
```

**从文件读取复杂查询**：
```bash
# 将复杂查询保存至 query.txt
```

```bash
cat > query.txt << 'EOF'
("2型糖尿病"[MeSH] OR "diabetes"[标题/摘要])
AND ("二甲双胍"[MeSH] OR "metformin"[标题/摘要])
AND "随机对照试验"[文献类型]
AND 2015:2024[出版日期]
AND English[语言]
EOF

# 执行搜索
python scripts/search_pubmed.py --query-file query.txt --limit 500
```

### 批量搜索

```bash
# 搜索多个主题
TOPICS=("diabetes treatment" "cancer immunotherapy" "CRISPR gene editing")

for topic in "${TOPICS[@]}"; do
  python scripts/search_pubmed.py "$topic" \
    --limit 100 \
    --output "${topic// /_}.json"
  sleep 1
done
```

### 提取元数据

```bash
# 搜索返回PMID
python scripts/search_pubmed.py "topic" --output results.json

# 提取完整元数据
python scripts/extract_metadata.py \
  --input results.json \
  --output references.bib
```

## 技巧与最佳实践

### 搜索构建

1. **从MeSH术语开始**：
   - 使用MeSH浏览器查找正确术语
   - 比关键词搜索更精确
   - 可捕获所有相关论文（无论术语差异）

2. **包含文本词变体**：
   ```
   # 更广覆盖范围
   ("糖尿病"[MeSH] OR diabetes OR diabetic)
   ```

3. **合理使用字段标签**：
   - `[MeSH]` 用于标准化概念
   - `[标题/摘要]` 用于特定术语
   - `[作者]` 用于已知作者
   - `[期刊]` 用于特定出版物

4. **逐步构建**：
   ```
   # 步骤1：基础搜索
   diabetes
   
   # 步骤2：增加特异性
   "2型糖尿病"[MeSH]
   
   # 步骤3：添加治疗方式
   "2型糖尿病"[MeSH] AND "二甲双胍"[MeSH]
   
   # 步骤4：添加研究类型
   "2型糖尿病"[MeSH] AND "二甲双胍"[MeSH] 
     AND "临床试验"[文献类型]
   
   # 步骤5：添加日期范围
   ... AND 2020:2024[出版日期]
   ```

### 优化结果

1. **结果过多时**：添加筛选条件
   - 限制文献类型
   - 缩小日期范围
   - 使用更具体的MeSH术语
   - 使用主要主题：`[MeSH主要主题]`

2. **结果过少时**：扩大搜索
   - 移除限制性筛选条件
   - 使用OR连接同义词
   - 扩展日期范围
   - 使用MeSH扩展检索（默认启用）

3. **结果不相关时**：优化术语
   - 使用更具体的MeSH术语
   - 用NOT排除无关项
   - 使用标题字段而非全字段
   - 添加MeSH副主题词

### 质量控制

1. **记录搜索策略**：
   - 保存精确查询语句
   - 记录搜索日期
   - 备注结果数量
   - 保存使用的筛选条件

2. **系统化导出**：
   - 使用统一文件名
   - 导出为JSON保持灵活性
   - 按需转换为BibTeX
   - 保留原始搜索结果

3. **验证引用文献**：
   ```bash
   python scripts/validate_citations.py pubmed_results.bib
   ```

### 跟踪更新

1. **设置搜索提醒**：
   - PubMed → 保存搜索
   - 接收邮件更新
   - 每日/每周/每月推送

2. **追踪特定期刊**：
   ```
   "Nature"[期刊] AND CRISPR[标题]
   ```

3. **关注关键作者**：
   ```
   "Church GM"[作者]
   ```

## 常见问题与解决方案

### 问题：未找到MeSH术语

**解决方案**：
- 检查拼写
- 使用MeSH浏览器
- 尝试相关术语
- 使用文本词搜索作为备选

### 问题：零结果

**解决方案**：
- 移除筛选条件
- 检查查询语法
- 使用OR扩大搜索
- 尝试同义词

### 问题：结果质量差

**解决方案**：
- 添加文献类型筛选
- 限制近年发表
- 使用MeSH主要主题
- 按期刊质量筛选

### 问题：不同来源的重复项

**解决方案**：
```bash
python scripts/format_bibtex.py results.bib \
  --deduplicate \
  --output clean.bib
```

### 问题：API速率限制

**解决方案**：
- 获取API密钥（提升至10次/秒）
- 脚本中添加延迟
- 分批处理
- 在非高峰时段操作

## 总结

PubMed提供权威生物医学文献检索：

✓ **精选内容**：MeSH索引，质量控制  
✓ **精准搜索**：字段标签，MeSH术语，筛选器  
✓ **程序化访问**：E-utilities API  
✓ **免费开放**：无需订阅  
✓ **全面覆盖**：3500万+引文，每日更新  

关键策略：
- 使用MeSH术语实现精准搜索
- 结合文本词确保全面覆盖
- 应用合适的字段标签
- 按文献类型和日期筛选
- 通过E-utilities API实现自动化
- 记录搜索策略保证可复现性

如需跨学科扩展检索，可配合使用Google Scholar。
