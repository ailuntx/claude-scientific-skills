```markdown
---
name: bioservices
description: 提供统一Python接口访问40多个生物信息服务。适用于在单一工作流中通过一致API查询多个数据库（UniProt、KEGG、ChEMBL、Reactome）。最适合跨数据库分析、跨服务ID映射。快速单数据库查询请使用gget；序列/文件操作请使用biopython。
license: GPLv3 许可证
metadata:
    skill-author: K-Dense Inc.
---

# BioServices

## 概述

BioServices 是一个Python软件包，提供对约40个生物信息学网络服务和数据库的程序化访问。可检索生物数据、执行跨数据库查询、映射标识符、分析序列，并在Python工作流中整合多种生物资源。该包透明处理REST和SOAP/WSDL协议。

## 适用场景

本技能适用于：
- 从UniProt、PDB、Pfam检索蛋白质序列、注释或结构
- 通过KEGG或Reactome分析代谢通路和基因功能
- 在化合物数据库（ChEBI、ChEMBL、PubChem）中搜索化学信息
- 在不同生物数据库间转换标识符（KEGG↔UniProt、化合物ID）
- 运行序列相似性搜索（BLAST、MUSCLE比对）
- 查询基因本体术语（QuickGO、GO注释）
- 访问蛋白质相互作用数据（PSICQUIC、IntactComplex）
- 挖掘基因组数据（BioMart、ArrayExpress、ENA）
- 在单一工作流中整合多个生物信息学资源的数据

## 核心功能

### 1. 蛋白质分析

检索蛋白质信息、序列及功能注释：

```python
from bioservices import UniProt

u = UniProt(verbose=False)

# 按名称搜索蛋白质
results = u.search("ZAP70_HUMAN", frmt="tab", columns="id,genes,organism")

# 获取FASTA序列
sequence = u.retrieve("P43403", "fasta")

# 在数据库间映射标识符
kegg_ids = u.mapping(fr="UniProtKB_AC-ID", to="KEGG", query="P43403")
```

**关键方法：**
- `search()`：使用灵活搜索词查询UniProt
- `retrieve()`：获取多种格式的蛋白质条目（FASTA、XML、表格）
- `mapping()`：在数据库间转换标识符

参考：`references/services_reference.md`获取完整UniProt API详情。

### 2. 通路发现与分析

访问基因和生物体的KEGG通路信息：

```python
from bioservices import KEGG

k = KEGG()
k.organism = "hsa"  # 设置为人类

# 搜索生物体
k.lookfor_organism("droso")  # 查找果蝇物种

# 按名称查找通路
k.lookfor_pathway("B cell")  # 返回匹配的通路ID

# 获取包含特定基因的通路
pathways = k.get_pathway_by_gene("7535", "hsa")  # ZAP70基因

# 检索并解析通路数据
data = k.get("hsa04660")
parsed = k.parse(data)

# 提取通路相互作用
interactions = k.parse_kgml_pathway("hsa04660")
relations = interactions['relations']  # 蛋白质间相互作用

# 转换为简单交互格式
sif_data = k.pathway2sif("hsa04660")
```

**关键方法：**
- `lookfor_organism()`, `lookfor_pathway()`：按名称搜索
- `get_pathway_by_gene()`：查找包含基因的通路
- `parse_kgml_pathway()`：提取结构化通路数据
- `pathway2sif()`：获取蛋白质互作网络

参考：`references/workflow_patterns.md`获取完整通路分析工作流。

### 3. 化合物数据库搜索

跨多个数据库搜索和交叉引用化合物：

```python
from bioservices import KEGG, UniChem

k = KEGG()

# 按名称搜索化合物
results = k.find("compound", "Geldanamycin")  # 返回 cpd:C11222

# 获取含数据库链接的化合物信息
compound_info = k.get("cpd:C11222")  # 包含ChEBI链接

# 使用UniChem进行KEGG→ChEMBL交叉引用
u = UniChem()
chembl_id = u.get_compound_id_from_kegg("C11222")  # 返回 CHEMBL278315
```

**常用工作流：**
1. 在KEGG中按名称搜索化合物
2. 提取KEGG化合物ID
3. 使用UniChem进行KEGG→ChEMBL映射
4. KEGG条目通常提供ChEBI ID

参考：`references/identifier_mapping.md`获取完整跨数据库映射指南。

### 4. 序列分析

运行BLAST搜索和序列比对：

```python
from bioservices import NCBIblast

s = NCBIblast(verbose=False)

# 对UniProtKB运行BLASTP
jobid = s.run(
    program="blastp",
    sequence=protein_sequence,
    stype="protein",
    database="uniprotkb",
    email="your.email@example.com"  # NCBI要求提供
)

# 检查任务状态并获取结果
s.getStatus(jobid)
results = s.getResult(jobid, "out")
```

**注意：** BLAST任务是异步的，获取结果前需检查状态。

### 5. 标识符映射

在不同生物数据库间转换标识符：

```python
from bioservices import UniProt, KEGG

# UniProt映射（支持多种数据库对）
u = UniProt()
results = u.mapping(
    fr="UniProtKB_AC-ID",  # 源数据库
    to="KEGG",              # 目标数据库
    query="P43403"          # 待转换标识符
)

# KEGG基因ID→UniProt
kegg_to_uniprot = u.mapping(fr="KEGG", to="UniProtKB_AC-ID", query="hsa:7535")

# 化合物映射使用UniChem
from bioservices import UniChem
u = UniChem()
chembl_from_kegg = u.get_compound_id_from_kegg("C11222")
```

**支持的映射（UniProt）：**
- UniProtKB ↔ KEGG
- UniProtKB ↔ Ensembl
- UniProtKB ↔ PDB
- UniProtKB ↔ RefSeq
- 更多映射（见`references/identifier_mapping.md`）

### 6. 基因本体查询

访问GO术语及注释：

```python
from bioservices import QuickGO

g = QuickGO(verbose=False)

# 获取GO术语信息
term_info = g.Term("GO:0003824", frmt="obo")

# 搜索注释
annotations = g.Annotation(protein="P43403", format="tsv")
```

### 7. 蛋白质相互作用

通过PSICQUIC查询相互作用数据库：

```python
from bioservices import PSICQUIC

s = PSICQUIC(verbose=False)

# 查询特定数据库（如MINT）
interactions = s.query("mint", "ZAP70 AND species:9606")

# 列出可用相互作用数据库
databases = s.activeDBs
```

**可用数据库：** MINT、IntAct、BioGRID、DIP等30余个。

## 多服务集成工作流

BioServices擅长整合多个服务进行综合分析，常见集成模式：

### 完整蛋白质分析流程

执行全面的蛋白质表征工作流：

```bash
python scripts/protein_analysis_workflow.py ZAP70_HUMAN your.email@example.com
```

该脚本演示：
1. UniProt搜索蛋白质条目
2. 获取FASTA序列
3. BLAST相似性搜索
4. KEGG通路发现
5. PSICQUIC相互作用映射

### 通路网络分析

分析生物体的所有通路：

```bash
python scripts/pathway_analysis.py hsa output_directory/
```

提取并分析：
- 生物体的所有通路ID
- 每条通路的蛋白质相互作用
- 相互作用类型分布
- 导出为CSV/SIF格式

### 跨数据库化合物搜索

跨数据库映射化合物标识符：

```bash
python scripts/compound_cross_reference.py Geldanamycin
```

检索内容：
- KEGG化合物ID
- ChEBI标识符
- ChEMBL标识符
- 基本化合物属性

### 批量标识符转换

一次性转换多个标识符：

```bash
python scripts/batch_id_converter.py input_ids.txt --from UniProtKB_AC-ID --to KEGG
```

## 最佳实践

### 输出格式处理

不同服务返回多种数据格式：
- **XML**：使用BeautifulSoup解析（多数SOAP服务）
- **制表符分隔（TSV）**：用Pandas DataFrame处理表格数据
- **字典/JSON**：直接Python操作
- **FASTA**：集成BioPython进行序列分析

### 速率限制与日志控制

调整API请求行为：

```python
from bioservices import KEGG

k = KEGG(verbose=False)  # 隐藏HTTP请求详情
k.TIMEOUT = 30  # 为慢速连接调整超时
```

### 错误处理

使用try-except块包裹服务调用：

```python
try:
    results = u.search("ambiguous_query")
    if results:
        # 处理结果
        pass
except Exception as e:
    print(f"搜索失败: {e}")
```

### 生物体代码

使用标准生物体缩写：
- `hsa`：智人（人类）
- `mmu`：小家鼠（小鼠）
- `dme`：黑腹果蝇
- `sce`：酿酒酵母

列出所有生物体：`k.list("organism")` 或 `k.organismIds`

### 与其他工具集成

BioServices可与以下工具协同工作：
- **BioPython**：对检索的FASTA数据进行序列分析
- **Pandas**：表格数据处理
- **PyMOL**：3D结构可视化（检索PDB ID）
- **NetworkX**：通路相互作用的网络分析
- **Galaxy**：工作流平台的自定义工具封装

## 资源

### scripts/

演示完整工作流的可执行Python脚本：

- `protein_analysis_workflow.py`：端到端蛋白质表征
- `pathway_analysis.py`：KEGG通路发现与网络提取
- `compound_cross_reference.py`：多数据库化合物搜索
- `batch_id_converter.py`：批量标识符转换工具

脚本可直接执行或根据用例调整。

### references/

按需加载的详细文档：

- `services_reference.md`：所有40+服务的完整方法列表
- `workflow_patterns.md`：多步骤分析工作流详解
- `identifier_mapping.md`：跨数据库ID转换完整指南

使用特定服务或复杂集成任务时加载参考文档。

## 安装

```bash
uv pip install bioservices
```

依赖项自动管理。包在Python 3.9-3.12测试通过。

## 附加信息

详细API文档和高级功能参考：
- 官方文档：https://bioservices.readthedocs.io/
- 源代码：https://github.com/cokelaer/bioservices
- 服务特定参考：`references/services_reference.md`
```
