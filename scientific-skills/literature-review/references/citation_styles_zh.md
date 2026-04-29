# 引文格式参考

本文档详细说明了文献综述中常用各类学术引文格式的规范。

## APA格式（第7版）

### 期刊文章

**格式**：作者, A. A., 作者, B. B., & 作者, C. C. (年份). 文章标题. *期刊名称*, *卷号*(期号), 页码范围. https://doi.org/xx.xxx/yyyy

**示例**：Smith, J. D., Johnson, M. L., & Williams, K. R. (2023). Machine learning approaches in drug discovery. *Nature Reviews Drug Discovery*, *22*(4), 301-318. https://doi.org/10.1038/nrd.2023.001

### 书籍

**格式**：作者, A. A. (年份). *著作标题：副标题首字母大写*. 出版社名称. https://doi.org/xxxx

**示例**：Kumar, V., Abbas, A. K., & Aster, J. C. (2021). *Robbins and Cotran pathologic basis of disease* (10th ed.). Elsevier.

### 书籍章节

**格式**：作者, A. A., & 作者, B. B. (年份). 章节标题. 见 E. E. 编者 & F. F. 编者 (编), *书籍标题* (pp. xx-xx). 出版社.

**示例**：Brown, P. O., & Botstein, D. (2020). Exploring the new world of the genome with DNA microarrays. In M. B. Eisen & P. O. Brown (Eds.), *DNA microarrays: A molecular cloning manual* (pp. 1-45). Cold Spring Harbor Laboratory Press.

### 预印本

**格式**：作者, A. A., & 作者, B. B. (年份). 预印本标题. *存储库名称*. https://doi.org/xxxx

**示例**：Zhang, Y., Chen, L., & Wang, H. (2024). Novel therapeutic targets in Alzheimer's disease. *bioRxiv*. https://doi.org/10.1101/2024.01.001

### 会议论文

**格式**：作者, A. A. (年份, 月 日-日). 论文标题. 见 E. E. 编者 (编), *会议论文集标题* (pp. xx-xx). 出版社. https://doi.org/xxxx

---

## Nature格式

### 期刊文章

**格式**：作者, A. A., 作者, B. B. & 作者, C. C. 文章标题. *期刊缩写* **卷号**, 页码范围 (年份).

**示例**：Smith, J. D., Johnson, M. L. & Williams, K. R. Machine learning approaches in drug discovery. *Nat. Rev. Drug Discov.* **22**, 301-318 (2023).

### 书籍

**格式**：作者, A. A. & 作者, B. B. *书籍标题* (出版社, 年份).

**示例**：Kumar, V., Abbas, A. K. & Aster, J. C. *Robbins and Cotran Pathologic Basis of Disease* 10th edn (Elsevier, 2021).

### 多位作者

- 1-2位作者：列出全部
- 3位及以上作者：列出第一作者后加"et al."

**示例**：Zhang, Y. et al. Novel therapeutic targets in Alzheimer's disease. *bioRxiv* https://doi.org/10.1101/2024.01.001 (2024).

---

## 芝加哥格式（作者-日期）

### 期刊文章

**格式**：作者, 名 中间名首字母. 年份. "文章标题." *期刊名称* 卷号, 期号 (月份): 页码范围. https://doi.org/xxxx.

**示例**：Smith, John D., Mary L. Johnson, and Karen R. Williams. 2023. "Machine Learning Approaches in Drug Discovery." *Nature Reviews Drug Discovery* 22, no. 4 (April): 301-318. https://doi.org/10.1038/nrd.2023.001.

### 书籍

**格式**：作者, 名 中间名首字母. 年份. *书籍标题：副标题*. 版次. 出版地: 出版社.

**示例**：Kumar, Vinay, Abul K. Abbas, and Jon C. Aster. 2021. *Robbins and Cotran Pathologic Basis of Disease*. 10th ed. Philadelphia: Elsevier.

---

## 温哥华格式（编号制）

### 期刊文章

**格式**：作者 AA, 作者 BB, 作者 CC. 文章标题. 期刊缩写. 年份;卷号(期号):页码范围.

**示例**：Smith JD, Johnson ML, Williams KR. Machine learning approaches in drug discovery. Nat Rev Drug Discov. 2023;22(4):301-18.

### 书籍

**格式**：作者 AA, 作者 BB. 书籍标题. 版次. 出版地: 出版社; 年份.

**示例**：Kumar V, Abbas AK, Aster JC. Robbins and Cotran pathologic basis of disease. 10th ed. Philadelphia: Elsevier; 2021.

### 文内引用

使用上标数字按出现顺序："近期研究^1,2^表明..."

---

## IEEE格式

### 期刊文章

**格式**：[#] A. A. 作者, B. B. 作者, and C. C. 作者, "文章标题," *期刊缩写*, vol. x, no. x, pp. xxx-xxx, 月份 年份.

**示例**：[1] J. D. Smith, M. L. Johnson, and K. R. Williams, "Machine learning approaches in drug discovery," *Nat. Rev. Drug Discov.*, vol. 22, no. 4, pp. 301-318, Apr. 2023.

### 书籍

**格式**：[#] A. A. 作者, *书籍标题*, x版. 城市, 州: 出版社, 年份.

**示例**：[2] V. Kumar, A. K. Abbas, and J. C. Aster, *Robbins and Cotran Pathologic Basis of Disease*, 10th ed. Philadelphia, PA: Elsevier, 2021.

---

## 期刊名称常用缩写

- Nature: Nat.
- Science: Science
- Cell: Cell
- Nature Reviews Drug Discovery: Nat. Rev. Drug Discov.
- Journal of the American Chemical Society: J. Am. Chem. Soc.
- Proceedings of the National Academy of Sciences: Proc. Natl. Acad. Sci. U.S.A.
- PLOS ONE: PLoS ONE
- Bioinformatics: Bioinformatics
- Nucleic Acids Research: Nucleic Acids Res.

---

## DOI最佳实践

1. **始终验证DOI**：使用verify_citations.py脚本检查所有DOI
2. **格式化为URL**：https://doi.org/10.xxxx/yyyy（优于doi:10.xxxx/yyyy）
3. **DOI后不加句点**：DOI应为最后元素且不带结尾标点
4. **检查重定向**：确认DOI解析至正确文章

---

## 文内引用规范

### APA格式
- (Smith et al., 2023)
- Smith et al. (2023) 证实...
- 多重引用：(Brown, 2022; Smith et al., 2023; Zhang, 2024)

### Nature格式
- 上标数字：近期研究^1,2^表明...
- 或：近期研究（参考文献1,2）表明...

### 芝加哥格式
- (Smith, Johnson, and Williams 2023)
- Smith, Johnson, and Williams (2023) 发现...

---

## 参考文献列表组织

### 按引文格式
- **APA, 芝加哥**：按第一作者姓氏字母排序
- **Nature, 温哥华, IEEE**：按文中首次出现顺序编号

### 悬挂缩进
多数格式采用悬挂缩进：首行顶格，后续行缩进

### 一致性
保持全篇格式统一：
- 大小写（标题式与句子式）
- 期刊名称缩写
- DOI呈现方式
- 作者姓名格式
