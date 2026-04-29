# BioServices：完整服务参考手册

本文档提供 BioServices 中所有主要服务的全面参考，包括核心方法、参数和使用场景。

## 蛋白质与基因资源

### UniProt

蛋白质序列与功能信息数据库。

**初始化：**
```python
from bioservices import UniProt
u = UniProt(verbose=False)
```

**核心方法：**

- `search(query, frmt="tab", columns=None, limit=None, sort=None, compress=False, include=False, **kwargs)`
  - 使用灵活查询语法搜索 UniProt
  - `frmt`：支持 "tab"、"fasta"、"xml"、"rdf"、"gff"、"txt"
  - `columns`：逗号分隔字段列表（如 "id,genes,organism,length"）
  - 返回值：请求格式的字符串

- `retrieve(uniprot_id, frmt="txt")`
  - 获取特定 UniProt 条目
  - `frmt`：支持 "txt"、"fasta"、"xml"、"rdf"、"gff"
  - 返回值：请求格式的条目数据

- `mapping(fr="UniProtKB_AC-ID", to="KEGG", query="P43403")`
  - 跨数据库标识符转换
  - `fr`/`to`：数据库标识符（参见 identifier_mapping.md）
  - `query`：单个 ID 或逗号分隔列表
  - 返回值：输入到输出 ID 的映射字典

- `searchUniProtId(pattern, columns="entry name,length,organism", limit=100)`
  - 基于 ID 的便捷搜索方法
  - 返回值：制表符分隔值

**常用字段：** id, entry name, genes, organism, protein names, length, sequence, go-id, ec, pathway, interactor

**使用场景：**
- 获取 BLAST 所需的蛋白质序列
- 功能注释查询
- 跨数据库标识符映射
- 批量蛋白质信息获取

---

### KEGG（京都基因与基因组百科全书）

代谢通路、基因与生物体数据库。

**初始化：**
```python
from bioservices import KEGG
k = KEGG()
k.organism = "hsa"  # 设置默认生物体
```

**核心方法：**

- `list(database)`
  - 列出 KEGG 数据库条目
  - `database`：支持 "organism"、"pathway"、"module"、"disease"、"drug"、"compound"
  - 返回值：多行条目字符串

- `find(database, query)`
  - 关键词搜索数据库
  - 返回值：匹配条目 ID 列表

- `get(entry_id)`
  - 按 ID 获取条目
  - 支持基因、通路、化合物等
  - 返回值：原始条目文本

- `parse(data)`
  - 将 KEGG 条目解析为字典
  - 返回值：结构化数据字典

- `lookfor_organism(name)`
  - 按名称模式搜索生物体
  - 返回值：匹配生物体代码列表

- `lookfor_pathway(name)`
  - 按名称搜索通路
  - 返回值：通路 ID 列表

- `get_pathway_by_gene(gene_id, organism)`
  - 查找包含基因的通路
  - 返回值：通路 ID 列表

- `parse_kgml_pathway(pathway_id)`
  - 解析通路 KGML 获取相互作用
  - 返回值：包含 "entries" 和 "relations" 的字典

- `pathway2sif(pathway_id)`
  - 提取简单相互作用格式数据
  - 筛选激活/抑制关系
  - 返回值：相互作用元组列表

**生物体代码：**
- hsa：智人（人类）
- mmu：小家鼠（小鼠）
- dme：黑腹果蝇
- sce：酿酒酵母
- eco：大肠杆菌

**使用场景：**
- 通路分析与可视化
- 基因功能注释
- 代谢网络重建
- 蛋白质相互作用提取

---

### HGNC（人类基因命名委员会）

官方人类基因命名机构。

**初始化：**
```python
from bioservices import HGNC
h = HGNC()
```

**核心方法：**
- `search(query)`：搜索基因符号/名称
- `fetch(format, query)`：获取基因信息

**使用场景：**
- 标准化人类基因名称
- 查询官方基因符号

---

### MyGeneInfo

基因注释与查询服务。

**初始化：**
```python
from bioservices import MyGeneInfo
m = MyGeneInfo()
```

**核心方法：**
- `querymany(ids, scopes, fields, species)`：批量基因查询
- `getgene(geneid)`：获取基因注释

**使用场景：**
- 批量基因注释获取
- 基因 ID 转换

---

## 化合物资源

### ChEBI（生物活性化学实体词典）

分子实体字典。

**初始化：**
```python
from bioservices import ChEBI
c = ChEBI()
```

**核心方法：**
- `getCompleteEntity(chebi_id)`：完整化合物信息
- `getLiteEntity(chebi_id)`：基础信息
- `getCompleteEntityByList(chebi_ids)`：批量获取

**使用场景：**
- 小分子信息查询
- 化学结构数据
- 化合物属性查找

---

### ChEMBL

类药生物活性化合物数据库。

**初始化：**
```python
from bioservices import ChEMBL
c = ChEMBL()
```

**核心方法：**
- `get_molecule_form(chembl_id)`：化合物详情
- `get_target(chembl_id)`：靶点信息
- `get_similarity(chembl_id)`：获取相似化合物
- `get_assays()`：生物测定数据

**使用场景：**
- 药物发现数据
- 查找相似化合物
- 生物活性信息
- 靶点-化合物关系分析

---

### UniChem

化学标识符映射服务。

**初始化：**
```python
from bioservices import UniChem
u = UniChem()
```

**核心方法：**
- `get_compound_id_from_kegg(kegg_id)`：KEGG → ChEMBL
- `get_all_compound_ids(src_compound_id, src_id)`：获取所有 ID
- `get_src_compound_ids(src_compound_id, from_src_id, to_src_id)`：转换 ID

**来源 ID：**
- 1：ChEMBL
- 2：DrugBank
- 3：PDB
- 6：KEGG
- 7：ChEBI
- 22：PubChem

**使用场景：**
- 跨数据库化合物 ID 映射
- 化学数据库关联

---

### PubChem

NIH 化学化合物数据库。

**初始化：**
```python
from bioservices import PubChem
p = PubChem()
```

**核心方法：**
- `get_compounds(identifier, namespace)`：获取化合物
- `get_properties(properties, identifier, namespace)`：获取属性

**使用场景：**
- 化学结构获取
- 化合物属性信息

---

## 序列分析工具

### NCBIblast

序列相似性搜索。

**初始化：**
```python
from bioservices import NCBIblast
s = NCBIblast(verbose=False)
```

**核心方法：**
- `run(program, sequence, stype, database, email, **params)`
  - 提交 BLAST 任务
  - `program`：支持 "blastp"、"blastn"、"blastx"、"tblastn"、"tblastx"
  - `stype`："protein" 或 "dna"
  - `database`："uniprotkb"、"pdb"、"refseq_protein" 等
  - `email`：NCBI 要求提供
  - 返回值：任务 ID

- `getStatus(jobid)`
  - 检查任务状态
  - 返回值："RUNNING"、"FINISHED"、"ERROR"

- `getResult(jobid, result_type)`
  - 获取结果
  - `result_type`："out"（默认）、"ids"、"xml"

**重要提示：** BLAST 任务为异步操作，获取结果前需检查状态。

**使用场景：**
- 蛋白质同源性搜索
- 序列相似性分析
- 基于同源性的功能注释

---

## 通路与相互作用资源

### Reactome

通路数据库。

**初始化：**
```python
from bioservices import Reactome
r = Reactome()
```

**核心方法：**
- `get_pathway_by_id(pathway_id)`：通路详情
- `search_pathway(query)`：搜索通路

**使用场景：**
- 人类通路分析
- 生物过程注释

---

### PSICQUIC

蛋白质相互作用查询服务（集成 30+ 数据库）。

**初始化：**
```python
from bioservices import PSICQUIC
s = PSICQUIC()
```

**核心方法：**
- `query(database, query_string)`
  - 查询特定相互作用数据库
  - 返回值：PSI-MI TAB 格式数据

- `activeDBs`
  - 列出可用数据库的属性
  - 返回值：数据库名称列表

**可用数据库：** MINT, IntAct, BioGRID, DIP, InnateDB, MatrixDB, MPIDB, UniProt 等 30+ 数据库

**查询语法：** 支持 AND、OR、物种筛选
- 示例："ZAP70 AND species:9606"

**使用场景：**
- 蛋白质相互作用发现
- 网络分析
- 相互作用组图谱构建

---

### IntactComplex

蛋白质复合物数据库。

**初始化：**
```python
from bioservices import IntactComplex
i = IntactComplex()
```

**核心方法：**
- `search(query)`：搜索复合物
- `details(complex_ac)`：复合物详情

**使用场景：**
- 蛋白质复合物组成分析
- 多蛋白组装研究

---

### OmniPath

集成信号通路数据库。

**初始化：**
```python
from bioservices import OmniPath
o = OmniPath()
```

**核心方法：**
- `interactions(datasets, organisms)`：获取相互作用
- `ptms(datasets, organisms)`：翻译后修饰信息

**使用场景：**
- 细胞信号分析
- 调控网络图谱构建

---

## 基因本体

### QuickGO

基因本体注释服务。

**初始化：**
```python
from bioservices import QuickGO
g = QuickGO()
```

**核心方法：**
- `Term(go_id, frmt="obo")`
  - 获取 GO 术语信息
  - 返回值：术语定义与元数据

- `Annotation(protein=None, goid=None, format="tsv")`
  - 获取 GO 注释
  - 返回值：请求格式的注释数据

**GO 分类：**
- 生物过程 (BP)
- 分子功能 (MF)
- 细胞组分 (CC)

**使用场景：**
- 功能注释
- 富集分析
- GO 术语查询

---

## 基因组资源

### BioMart

基因组数据挖掘工具。

**初始化：**
```python
from bioservices import BioMart
b = BioMart()
```

**核心方法：**
- `datasets(dataset)`：列出可用数据集
- `attributes(dataset)`：列出属性字段
- `query(query_xml)`：执行 BioMart 查询

**使用场景：**
- 批量基因组数据获取
- 定制基因组注释
- SNP 信息查询

---

### ArrayExpress

基因表达数据库。

**初始化：**
```python
from bioservices import ArrayExpress
a = ArrayExpress()
```

**核心方法：**
- `queryExperiments(keywords)`：搜索实验
- `retrieveExperiment(accession)`：获取实验数据

**使用场景：**
- 基因表达数据
- 微阵列分析
- RNA-seq 数据获取

---

### ENA（欧洲核苷酸档案库）

核苷酸序列数据库。

**初始化：**
```python
from bioservices import ENA
e = ENA()
```

**核心方法：**
- `search_data(query)`：搜索序列
- `retrieve_data(accession)`：获取序列

**使用场景：**
- 核苷酸序列获取
- 基因组组装访问

---

## 结构生物学

### PDB（蛋白质数据库）

蛋白质三维结构数据库。

**初始化：**
```python
from bioservices import PDB
p = PDB()
```

**核心方法：**
- `get_file(pdb_id, file_format)`：下载结构文件
- `search(query)`：搜索结构

**文件格式：** pdb, cif, xml

**使用场景：**
- 三维结构获取
- 基于结构的分析
- PyMOL 可视化

---

### Pfam

蛋白质家族数据库。

**初始化：**
```python
from bioservices import Pfam
p = Pfam()
```

**核心方法：**
- `searchSequence(sequence)`：序列结构域识别
- `getPfamEntry(pfam_id)`：结构域信息

**使用场景：**
- 蛋白质结构域鉴定
- 家族分类
- 功能基序发现

---

## 专业资源

### BioModels

系统生物学模型库。

**初始化：**
```python
from bioservices import BioModels
b = BioModels()
```

**核心方法：**
- `get_model_by_id(model_id)`：获取 SBML 模型

**使用场景：**
- 系统生物学建模
- SBML 模型获取

---

### COG（直系同源基因簇）

直系同源基因分类。

**初始化：**
```python
from bioservices import COG
c = COG()
```

**使用场景：**
- 直系同源分析
- 功能分类

---

### BiGG Models

代谢网络模型。

**初始化：**
```python
from bioservices import BiGG
b = BiGG()
```

**核心方法：**
- `list_models()`：可用模型列表
- `get_model(model_id)`：模型详情

**使用场景：**
- 代谢网络分析
- 通量平衡分析

---

## 通用模式

### 错误处理

所有服务可能抛出异常，建议使用 try-except 包裹调用：

```python
try:
    result = service.method(params)
    if result:
        # 处理结果
        pass
except Exception as e:
    print(f"错误: {e}")
```

### 日志控制

多数服务支持 `verbose` 参数：
```python
service = Service(verbose=False)  # 关闭 HTTP 日志
```

### 速率限制

服务存在超时和速率限制：
```python
service.TIMEOUT = 30  # 调整超时时间
service.DELAY = 1     # 请求间延迟（如支持）
```

### 输出格式

常用格式参数：
- `frmt`："xml"、"json"、"tab"、"txt"、"fasta"
- `format`：服务特定变体

### 缓存机制

部分服务支持结果缓存：
```python
service.CACHE = True  # 启用缓存
service.clear_cache() # 清除缓存
```

## 附加资源

详细 API 文档：
- 官方文档：https://bioservices.readthedocs.io/
- 各服务文档见主页链接
- 源代码：https://github.com/cokelaer/bioservices
