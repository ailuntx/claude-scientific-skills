# 期刊格式要求

各学科主要科学期刊的全面格式要求与投稿指南。

**最后更新**：2024年

---

## Nature系列期刊

### Nature

**期刊类型**：顶级多学科科学期刊  
**出版商**：Nature Publishing Group  
**影响因子**：约64（每年浮动）

**格式要求**：
- **篇幅**：正文约3,000词（方法、参考文献、图注除外）
- **结构**：标题、作者、单位、摘要（≤200词）、正文、方法、参考文献、致谢、作者贡献、利益冲突声明、图注
- **排版**：单栏提交（最终出版为双栏）
- **字体**：标准字体（Times, Arial, Helvetica），12磅
- **行距**：双倍行距
- **页边距**：四边均2.5厘米（1英寸）
- **页码**：所有页面均需编号
- **引文**：上标数字顺序编号¹'²'³
- **参考文献**：Nature格式（期刊名缩写）
  - 格式：作者A. A., 作者B. B. & 作者C. C. 文章标题. *期刊缩写* **卷号**, 页码 (年份).
  - 示例：Watson, J. D. & Crick, F. H. C. Molecular structure of nucleic acids. *Nature* **171**, 737–738 (1953).
- **图片**：
  - 格式：TIFF, EPS, PDF（矢量图优先）
  - 分辨率：照片300-600 dpi，线稿1000 dpi
  - 色彩模式：RGB或CMYK
  - 尺寸：适配单栏（89毫米）或双栏（183毫米）
  - 图注：单独提供，不嵌入图片
- **表格**：可编辑格式（Word, Excel），禁止图片形式
- **补充材料**：无数量限制，建议PDF格式

**LaTeX模板**：`assets/journals/nature_article.tex`

**作者指南**：https://www.nature.com/nature/for-authors

---

### Nature Communications

**期刊类型**：开放获取多学科期刊  
**出版商**：Nature Publishing Group

**格式要求**：
- **篇幅**：无严格限制（通常5,000-8,000词）
- **结构**：同Nature（标题、摘要、正文、方法等）
- **排版**：单栏
- **字体**：Times New Roman或Arial等，12磅
- **行距**：双倍行距
- **页边距**：四边均2.5厘米
- **引文**：上标数字顺序编号
- **参考文献**：Nature格式（同Nature）
- **图片**：要求同Nature
- **表格**：要求同Nature
- **开放获取**：所有文章开放获取（需支付文章处理费）

**LaTeX模板**：`assets/journals/nature_communications.tex`

---

### Nature Methods, Nature Biotechnology, Nature Machine Intelligence

**格式**：同Nature Communications（Nature系列期刊格式相似）

**学科特定说明**：
- **Nature Methods**：强调方法创新与验证
- **Nature Biotechnology**：聚焦生物技术应用与转化
- **Nature Machine Intelligence**：跨学科人工智能/机器学习应用

---

## Science系列期刊

### Science

**期刊类型**：顶级多学科科学期刊  
**出版商**：美国科学促进会（AAAS）

**格式要求**：
- **篇幅**：
  - 研究论文：2,500词（仅正文，不含参考文献/图表）
  - 报告：上限2,500词
- **结构**：标题、作者、单位、摘要（≤125词）、正文、材料与方法、参考文献、致谢、补充材料
- **排版**：单栏提交
- **字体**：Times New Roman, 12磅
- **行距**：双倍行距
- **页边距**：四边均1英寸
- **引文**：括号内顺序编号(1, 2, 3)
- **参考文献**：Science格式（主文献不含文章标题，移至补充材料）
  - 格式：A. 作者, B. 作者, *期刊缩写* **卷号**, 页码 (年份).
  - 示例：J. D. Watson, F. H. C. Crick, *Nature* **171**, 737 (1953).
- **图片**：
  - 格式：PDF, EPS, TIFF
  - 分辨率：最低300 dpi
  - 色彩模式：RGB
  - 尺寸：最大宽度9厘米（单栏）或18.3厘米（双栏）
  - 图片计入篇幅限制
- **表格**：可置于正文或单独文件
- **补充材料**：允许大量补充内容

**LaTeX模板**：`assets/journals/science_article.tex`

**作者指南**：https://www.science.org/content/page/instructions-authors

---

### Science Advances

**期刊类型**：开放获取多学科期刊  
**出版商**：AAAS

**格式要求**：
- **篇幅**：无严格限制（鼓励简洁行文）
- **结构**：类似Science（更灵活）
- **排版**：单栏
- **字体**：Times New Roman, 12磅
- **引文**：括号内编号
- **参考文献**：Science格式
- **图片**：要求同Science
- **开放获取**：所有文章开放获取

**LaTeX模板**：`assets/journals/science_advances.tex`

---

## PLOS（公共科学图书馆）

### PLOS ONE

**期刊类型**：开放获取多学科期刊  
**出版商**：Public Library of Science

**格式要求**：
- **篇幅**：无上限
- **结构**：标题、作者、单位、摘要、引言、材料与方法、结果、讨论、结论（可选）、参考文献、支持信息
- **排版**：可编辑文件（LaTeX, Word, RTF）
- **字体**：Times, Arial或Helvetica, 10-12磅
- **行距**：双倍行距
- **页边距**：四边均1英寸（2.54厘米）
- **页码**：必需
- **引文**：温哥华格式，方括号编号[1], [2], [3]
- **参考文献**：温哥华/NLM格式
  - 格式：作者AA, 作者BB, 作者CC. 文章标题. 期刊缩写. 年份;卷(期):页码. doi:xx.xxxx
  - 示例：Watson JD, Crick FHC. Molecular structure of nucleic acids. Nature. 1953;171(4356):737-738.
- **图片**：
  - 格式：TIFF, EPS, PDF, PNG
  - 分辨率：300-600 dpi
  - 色彩模式：RGB
  - 图注：置于参考文献后的正文中
- **表格**：可编辑格式，每页一表
- **数据可用性**：需提供声明
- **开放获取**：所有文章开放获取（需支付文章处理费）

**LaTeX模板**：`assets/journals/plos_one.tex`

**作者指南**：https://journals.plos.org/plosone/s/submission-guidelines

---

### PLOS Biology, PLOS Computational Biology等

**格式**：类似PLOS ONE，含学科特定调整

**关键差异**：
- PLOS Biology：遴选更严格，强调广泛意义
- PLOS Comp Bio：侧重计算方法与模型

---

## Cell Press

### Cell

**期刊类型**：顶级生物学期刊  
**出版商**：Cell Press（Elsevier）

**格式要求**：
- **篇幅**：
  - 论文：约5,000词（方法、参考文献除外）
  - 短文：约2,500词
- **结构**：摘要（≤150词）、关键词、引言、结果、讨论、实验步骤、致谢、作者贡献、利益声明、参考文献
- **排版**：双倍行距
- **字体**：12磅
- **页边距**：四边均1英寸
- **引文**：作者-年份格式（Smith et al., 2023）
- **参考文献**：Cell格式
  - 格式：作者, A.A., and 作者, B.B. (年份). 标题. *期刊* 卷, 页码.
  - 示例：Watson, J.D., and Crick, F.H. (1953). Molecular structure of nucleic acids. *Nature* 171, 737-738.
- **图片**：
  - 格式：优先TIFF, EPS
  - 分辨率：照片300 dpi，线稿1000 dpi
  - 色彩模式：RGB或CMYK
  - 多面板图常见
- **表格**：可编辑格式
- **电子目录摘要**：需30-50词概要
- **图文摘要**：必需

**LaTeX模板**：`assets/journals/cell_article.tex`

**作者指南**：https://www.cell.com/cell/authors

---

### Neuron, Immunity, Molecular Cell, Developmental Cell

**格式**：类似Cell，含学科特定要求

---

## IEEE汇刊

### IEEE Transactions on [各主题]

**期刊类型**：工程与计算机科学期刊  
**出版商**：电气电子工程师学会

**格式要求**：
- **篇幅**：各汇刊不同（最终版通常8-12页）
- **结构**：摘要、索引词、引言、[主体章节]、结论、致谢、参考文献、作者简介
- **排版**：双栏
- **字体**：Times New Roman, 10磅
- **栏间距**：0.17英寸（4.23毫米）
- **页边距**：
  - 上：19毫米（0.75英寸）
  - 下：25毫米（1英寸）
  - 侧：17毫米（0.67英寸）
- **引文**：方括号编号[1], [2], [3]
- **参考文献**：IEEE格式
  - 格式：[1] A. A. 作者, "文章标题," *期刊缩写*, 卷x, 期x, 页xxx-xxx, 月 年.
  - 示例：[1] J. D. Watson and F. H. C. Crick, "Molecular structure of nucleic acids," *Nature*, vol. 171, pp. 737-738, Apr. 1953.
- **图片**：
  - 格式：EPS, PDF（矢量）, TIFF（位图）
  - 分辨率：线稿600-1200 dpi，灰度/彩色300 dpi
  - 色彩模式：在线用RGB，印刷需CMYK
  - 位置：栏顶或栏底
- **表格**：LaTeX表格环境，置于栏顶/栏底
- **公式**：连续编号

**LaTeX模板**：`assets/journals/ieee_trans.tex`

**作者指南**：https://journals.ieeeauthorcenter.ieee.org/

---

### IEEE Access

**期刊类型**：开放获取多学科工程期刊  
**出版商**：IEEE

**格式**：类似IEEE汇刊
- **篇幅**：无页数限制
- **开放获取**：所有文章开放获取
- **快速出版**：审稿快于汇刊

**LaTeX模板**：`assets/journals/ieee_access.tex`

---

## ACM出版物

### ACM汇刊

**期刊类型**：计算机科学汇刊  
**出版商**：计算机协会

**格式要求**：
- **篇幅**：无严格限制
- **结构**：摘要、CCS概念、关键词、ACM参考文献格式、引言、[主体]、结论、致谢、参考文献
- **排版**：最终版双栏，单栏提交可接受
- **字体**：依模板而定（通常9-10磅）
- **文档类**：使用`acmart` LaTeX文档类
- **引文**：编号[1]或作者-年份（取决于会议）
- **参考文献**：ACM格式
  - 格式：作者. 年份. 标题. 期刊 卷, 期 (年份), 页码. DOI
  - 示例：James D. Watson and Francis H. C. Crick. 1953. Molecular structure of nucleic acids. Nature 171, 4356 (1953), 737-738. https://doi.org/10.1038/171737a0
- **图片**：EPS, PDF（优先矢量图），高分辨率位图
- **CCS概念**：必需（ACM计算分类系统）
- **关键词**：必需

**LaTeX模板**：`assets/journals/acm_article.tex`

**作者指南**：https://www.acm.org/publications/authors

---

## Springer期刊

### Springer通用期刊

**出版商**：Springer Nature

**格式要求**：
- **篇幅**：各期刊不同（需具体查询）
- **排版**：单栏提交（LaTeX或Word）
- **字体**：10-12磅
- **行距**：双倍或1.5倍
- **引文**：编号或作者-年份（因刊而异）
- **参考文献**：Springer格式（类似温哥华或作者-年份）
  - 编号：作者AA, 作者BB (年份) 标题. 期刊 卷:页码
  - 作者-年份：作者AA, 作者BB (年份) 标题. 期刊 卷:页码
- **图片**：TIFF, EPS, PDF；300+ dpi
- **表格**：可编辑格式
- **文档类**：多数期刊使用`svjour3`

**LaTeX模板**：`assets/journals/springer_article.tex`

**作者指南**：各期刊不同

---

## Elsevier期刊

### Elsevier通用期刊

**出版商**：Elsevier

**格式要求**：
- **篇幅**：各期刊差异大
- **排版**：单栏（LaTeX或Word）
- **字体**：12磅
- **行距**：双倍行距
- **引文**：编号或作者-年份（需查期刊指南）
- **参考文献**：格式因刊而异（哈佛、温哥华、编号）
  - 需查阅具体期刊的"作者指南"
- **图片**：TIFF, EPS；300+ dpi
- **表格**：可编辑格式
- **文档类**：`elsarticle` LaTeX类

**LaTeX模板**：`assets/journals/elsevier_article.tex`

**作者指南**：https://www.elsevier.com/authors（选择具体期刊）

---

## BMC期刊

### BMC Biology, BMC Bioinformatics等

**出版商**：BioMed Central（Springer Nature）

**格式要求**：
- **篇幅**：无上限
- **结构**：摘要（结构化）、关键词、背景、[方法/结果/讨论]、结论、缩写词、声明（伦理、同意书、可用性、利益冲突、基金、作者贡献、致谢）、参考文献
- **排版**：单栏
- **字体**：Arial或Times, 12磅
- **行距**：双倍
- **引文**：温哥华格式，方括号编号[1]
- **参考文献**：温哥华/NLM格式
- **图片**：TIFF, EPS, PNG；300+ dpi
- **表格**：可编辑
- **开放获取**：所有BMC期刊开放获取
- **数据可用性**：需提供声明

**LaTeX模板**：`assets/journals/bmc_article.tex`

**作者指南**：https://www.biomedcentral.com/getpublished

---

## Frontiers期刊

### Frontiers in [各主题]

**出版商**：Frontiers Media

**格式要求**：
- **篇幅**：因文章类型而异（研究论文约12页，简报约4页）
- **结构**：摘要、关键词、引言、材料与方法、结果、讨论、结论、数据可用性声明、伦理声明、作者贡献、基金、致谢、利益冲突、参考文献
- **排版**：单栏
- **字体**：Times New Roman, 12磅
- **行距**：双倍
- **引文**：编号（Frontiers格式）
- **参考文献**：Frontiers格式
  - 格式：作者A., 作者B., 作者C. (年份). 标题. *期刊缩写* 卷:页码. doi
  - 示例：Watson J. D., Crick F. H. C. (195

### PNAS（美国国家科学院院刊）

**格式要求**：
- **篇幅**：6页（含正文、图表）
- **摘要**：不超过250词
- **意义声明**：不超过120词（必需）
- **结构**：摘要、意义、正文、材料与方法、致谢、参考文献
- **格式**：单栏
- **引用**：编号制
- **参考文献**：PNAS格式
- **LaTeX文档类**：`pnas-new`

**LaTeX模板**：`assets/journals/pnas_article.tex`

---

### 物理评论快报（PRL）

**出版商**：美国物理学会

**格式要求**：
- **篇幅**：4页（含图表和参考文献）
- **格式**：双栏（REVTeX 4.2）
- **摘要**：不超过600字符
- **引用**：编号制
- **参考文献**：APS格式
- **文档类**：`revtex4-2`

**LaTeX模板**：`assets/journals/prl_article.tex`

---

### 新英格兰医学杂志（NEJM）

**格式要求**：
- **篇幅**：原创文章约3,000词
- **结构**：摘要（结构化，250词）、引言、方法、结果、讨论、参考文献
- **格式**：双倍行距
- **引用**：编号制
- **参考文献**：NEJM格式（修订版温哥华格式）
- **图表**：高分辨率专业品质
- **建议提交Word文档**（较少使用LaTeX）

---

### 柳叶刀

**格式要求**：
- **篇幅**：文章约3,000词
- **摘要**：结构化，300词
- **结构**：要点框、引言、方法、结果、讨论、参考文献
- **引用**：编号制
- **参考文献**：柳叶刀格式（修订版温哥华格式）
- **建议提交Word文档**

---

## 快速参考表

| 期刊 | 最大篇幅 | 格式 | 引用格式 | 模板 |
|---------|-----------|--------|-----------|----------|
| **Nature** | ~3,000词 | 单栏 | 上标 | `nature_article.tex` |
| **Science** | 2,500词 | 单栏 | (1)括号 | `science_article.tex` |
| **PLOS ONE** | 无限制 | 单栏 | [1]温哥华 | `plos_one.tex` |
| **Cell** | ~5,000词 | 双倍行距 | (作者,年份) | `cell_article.tex` |
| **IEEE Trans** | 8-12页 | 双栏 | [1]IEEE | `ieee_trans.tex` |
| **ACM Trans** | 可变 | 双栏 | [1]或作者-年份 | `acm_article.tex` |
| **Springer** | 可变 | 单栏 | 编号/作者-年份 | `springer_article.tex` |
| **BMC** | 无限制 | 单栏 | [1]温哥华 | `bmc_article.tex` |
| **Frontiers** | ~12页 | 单栏 | 编号 | `frontiers_article.tex` |

---

## 注意事项

1. **务必查阅官方指南**：期刊要求可能变更，提交前请核实
2. **模板时效性**：本模板定期更新，但可能滞后于官方变更
3. **补充材料**：多数期刊允许提交扩展补充材料
4. **预印本政策**：确认期刊预印本政策（多数允许arXiv、bioRxiv）
5. **开放获取选项**：多数订阅期刊提供付费开放获取服务
6. **LaTeX与Word**：多数期刊两者皆可，数学密集型内容建议LaTeX

## 获取官方模板

多数期刊提供官方LaTeX模板：
- **Nature**：期刊官网下载
- **IEEE**：IEEEtran文档类（广泛可用）
- **ACM**：acmart文档类（CTAN）
- **Elsevier**：elsarticle文档类（CTAN）
- **Springer**：svjour3文档类（期刊官网）

请访问期刊"作者指南"或"投稿"页面获取最新模板。
