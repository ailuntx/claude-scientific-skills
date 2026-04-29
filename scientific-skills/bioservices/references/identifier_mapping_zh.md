# BioServices：标识符映射指南

本文档提供使用BioServices在不同生物数据库间转换标识符的全面信息。

## 目录

1. [概述](#overview)
2. [UniProt映射服务](#uniprot-mapping-service)
3. [UniChem化合物映射](#unichem-compound-mapping)
4. [KEGG标识符转换](#kegg-identifier-conversions)
5. [常见映射模式](#common-mapping-patterns)
6. [故障排除](#troubleshooting)

---

## 概述

生物数据库使用不同的标识符系统。交叉引用需要在系统间进行映射。BioServices提供多种方法：

1. **UniProt映射**：全面的蛋白质/基因ID转换
2. **UniChem**：化合物ID映射
3. **KEGG**：条目内置交叉引用
4. **PICR**：蛋白质标识符交叉引用服务

---

## UniProt映射服务

UniProt映射服务是最全面的蛋白质和基因标识符转换工具。

### 基本用法

```python
from bioservices import UniProt

u = UniProt()

# 映射单个ID
result = u.mapping(
    fr="UniProtKB_AC-ID",    # 源数据库
    to="KEGG",                # 目标数据库
    query="P43403"            # 待转换标识符
)

print(result)
# 输出: {'P43403': ['hsa:7535']}
```

### 批量映射

```python
# 映射多个ID（逗号分隔）
ids = ["P43403", "P04637", "P53779"]
result = u.mapping(
    fr="UniProtKB_AC-ID",
    to="KEGG",
    query=",".join(ids)
)

for uniprot_id, kegg_ids in result.items():
    print(f"{uniprot_id} → {kegg_ids}")
```

### 支持的数据库对

UniProt支持100+数据库对映射。关键组合包括：

#### 蛋白质/基因数据库

| 源格式 | 代码 | 目标格式 | 代码 |
|---------------|------|---------------|------|
| UniProtKB AC/ID | `UniProtKB_AC-ID` | KEGG | `KEGG` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | Ensembl | `Ensembl` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | Ensembl Protein | `Ensembl_Protein` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | Ensembl Transcript | `Ensembl_Transcript` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | RefSeq Protein | `RefSeq_Protein` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | RefSeq Nucleotide | `RefSeq_Nucleotide` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | GeneID (Entrez) | `GeneID` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | HGNC | `HGNC` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | MGI | `MGI` |
| KEGG | `KEGG` | UniProtKB | `UniProtKB` |
| Ensembl | `Ensembl` | UniProtKB | `UniProtKB` |
| GeneID | `GeneID` | UniProtKB | `UniProtKB` |

#### 结构数据库

| 源 | 代码 | 目标 | 代码 |
|--------|------|--------|------|
| UniProtKB AC/ID | `UniProtKB_AC-ID` | PDB | `PDB` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | Pfam | `Pfam` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | InterPro | `InterPro` |
| PDB | `PDB` | UniProtKB | `UniProtKB` |

#### 表达与蛋白质组学

| 源 | 代码 | 目标 | 代码 |
|--------|------|--------|------|
| UniProtKB AC/ID | `UniProtKB_AC-ID` | PRIDE | `PRIDE` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | ProteomicsDB | `ProteomicsDB` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | PaxDb | `PaxDb` |

#### 物种特异性

| 源 | 代码 | 目标 | 代码 |
|--------|------|--------|------|
| UniProtKB AC/ID | `UniProtKB_AC-ID` | FlyBase | `FlyBase` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | WormBase | `WormBase` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | SGD | `SGD` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | ZFIN | `ZFIN` |

#### 其他实用映射

| 源 | 代码 | 目标 | 代码 |
|--------|------|--------|------|
| UniProtKB AC/ID | `UniProtKB_AC-ID` | GO | `GO` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | Reactome | `Reactome` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | STRING | `STRING` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | BioGRID | `BioGRID` |
| UniProtKB AC/ID | `UniProtKB_AC-ID` | OMA | `OMA` |

### 完整数据库代码列表

获取最新完整列表：

```python
from bioservices import UniProt

u = UniProt()

# 此信息在UniProt REST API文档中
# 常见模式：
# - 源数据库通常以源数据库名称结尾
# - UniProtKB使用"UniProtKB_AC-ID"或"UniProtKB"
# - 大多数其他数据库使用标准缩写
```

### 常用数据库代码参考

**基因/蛋白质标识符：**
- `UniProtKB_AC-ID`：UniProt登录号/ID
- `UniProtKB`：UniProt登录号
- `KEGG`：KEGG基因ID（如hsa:7535）
- `GeneID`：NCBI Gene（Entrez）ID
- `Ensembl`：Ensembl基因ID
- `Ensembl_Protein`：Ensembl蛋白质ID
- `Ensembl_Transcript`：Ensembl转录本ID
- `RefSeq_Protein`：RefSeq蛋白质ID（NP_）
- `RefSeq_Nucleotide`：RefSeq核苷酸ID（NM_）

**基因命名：**
- `HGNC`：人类基因命名委员会
- `MGI`：小鼠基因组信息学
- `RGD`：大鼠基因组数据库
- `SGD`：酿酒酵母基因组数据库
- `FlyBase`：果蝇数据库
- `WormBase`：秀丽隐杆线虫数据库
- `ZFIN`：斑马鱼数据库

**结构：**
- `PDB`：蛋白质数据库
- `Pfam`：蛋白质家族
- `InterPro`：蛋白质结构域
- `SUPFAM`：超家族
- `PROSITE`：蛋白质基序

**通路与网络：**
- `Reactome`：Reactome通路
- `BioCyc`：BioCyc通路
- `PathwayCommons`：通路共享库
- `STRING`：蛋白质-蛋白质网络
- `BioGRID`：相互作用数据库

### 映射示例

#### UniProt → KEGG

```python
from bioservices import UniProt

u = UniProt()

# 单条映射
result = u.mapping(fr="UniProtKB_AC-ID", to="KEGG", query="P43403")
print(result)  # {'P43403': ['hsa:7535']}
```

#### KEGG → UniProt

```python
# 反向映射
result = u.mapping(fr="KEGG", to="UniProtKB", query="hsa:7535")
print(result)  # {'hsa:7535': ['P43403']}
```

#### UniProt → Ensembl

```python
# 转Ensembl基因ID
result = u.mapping(fr="UniProtKB_AC-ID", to="Ensembl", query="P43403")
print(result)  # {'P43403': ['ENSG00000115085']}

# 转Ensembl蛋白质ID
result = u.mapping(fr="UniProtKB_AC-ID", to="Ensembl_Protein", query="P43403")
print(result)  # {'P43403': ['ENSP00000381359']}
```

#### UniProt → PDB

```python
# 查找3D结构
result = u.mapping(fr="UniProtKB_AC-ID", to="PDB", query="P04637")
print(result)  # {'P04637': ['1A1U', '1AIE', '1C26', ...]}
```

#### UniProt → RefSeq

```python
# 获取RefSeq蛋白质ID
result = u.mapping(fr="UniProtKB_AC-ID", to="RefSeq_Protein", query="P43403")
print(result)  # {'P43403': ['NP_001070.2']}
```

#### 基因名 → UniProt（通过搜索再映射）

```python
# 先搜索基因
search_result = u.search("gene:ZAP70 AND organism:9606", frmt="tab", columns="id")
lines = search_result.strip().split("\n")
if len(lines) > 1:
    uniprot_id = lines[1].split("\t")[0]

    # 再映射到其他数据库
    kegg_id = u.mapping(fr="UniProtKB_AC-ID", to="KEGG", query=uniprot_id)
    print(kegg_id)
```

---

## UniChem化合物映射

UniChem专门用于跨数据库映射化合物标识符。

### 源数据库ID

| 源ID | 数据库 |
|-----------|----------|
| 1 | ChEMBL |
| 2 | DrugBank |
| 3 | PDB |
| 4 | IUPHAR/BPS药理学指南 |
| 5 | PubChem |
| 6 | KEGG |
| 7 | ChEBI |
| 8 | NIH临床化合物库 |
| 14 | FDA/SRS |
| 22 | PubChem |

### 基本用法

```python
from bioservices import UniChem

u = UniChem()

# 从KEGG化合物ID获取ChEMBL ID
chembl_id = u.get_compound_id_from_kegg("C11222")
print(chembl_id)  # CHEMBL278315
```

### 获取所有化合物ID

```python
# 获取化合物的所有标识符
# src_compound_id: 化合物ID, src_id: 源数据库ID
all_ids = u.get_all_compound_ids("CHEMBL278315", src_id=1)  # 1 = ChEMBL

for mapping in all_ids:
    src_name = mapping['src_name']
    src_compound_id = mapping['src_compound_id']
    print(f"{src_name}: {src_compound_id}")
```

### 特定数据库转换

```python
# 在特定数据库间转换
# from_src_id=6 (KEGG), to_src_id=1 (ChEMBL)
result = u.get_src_compound_ids("C11222", from_src_id=6, to_src_id=1)
print(result)
```

### 常见化合物映射

#### KEGG → ChEMBL

```python
u = UniChem()
chembl_id = u.get_compound_id_from_kegg("C00031")  # D-葡萄糖
print(f"ChEMBL: {chembl_id}")
```

#### ChEMBL → PubChem

```python
result = u.get_src_compound_ids("CHEMBL278315", from_src_id=1, to_src_id=22)
if result:
    pubchem_id = result[0]['src_compound_id']
    print(f"PubChem: {pubchem_id}")
```

#### ChEBI → DrugBank

```python
result = u.get_src_compound_ids("5292", from_src_id=7, to_src_id=2)
if result:
    drugbank_id = result[0]['src_compound_id']
    print(f"DrugBank: {drugbank_id}")
```

---

## KEGG标识符转换

KEGG条目包含可通过解析提取的交叉引用。

### 从KEGG条目提取数据库链接

```python
from bioservices import KEGG

k = KEGG()

# 获取化合物条目
entry = k.get("cpd:C11222")

# 解析特定数据库
chebi_id = None
uniprot_ids = []

for line in entry.split("\n"):
    if "ChEBI:" in line:
        # 提取ChEBI ID
        parts = line.split("ChEBI:")
        if len(parts) > 1:
            chebi_id = parts[1].strip().split()[0]

# 基因/蛋白质解析
gene_entry = k.get("hsa:7535")
for line in gene_entry.split("\n"):
    if line.startswith("            "):  # 数据库链接部分
        if "UniProt:" in line:
            parts = line.split("UniProt:")
            if len(parts) > 1:
                uniprot_id = parts[1].strip()
                uniprot_ids.append(uniprot_id)
```

### KEGG基因ID组成

KEGG基因ID格式为`物种:基因ID`：

```python
kegg_id = "hsa:7535"
organism, gene_id = kegg_id.split(":")

print(f"物种: {organism}")  # hsa (人类)
print(f"基因ID: {gene_id}")    # 7535
```

### KEGG通路转基因列表

```python
k = KEGG()

# 获取通路条目
pathway = k.get("path:hsa04660")

# 解析基因列表
genes = []
in_gene_section = False

for line in pathway.split("\n"):
    if line.startswith("GENE"):
        in_gene_section = True

    if in_gene_section:
        if line.startswith(" " * 12):  # 基因行
            parts = line.strip().split()
            if parts:
                gene_id = parts[0]
                genes.append(f"hsa:{gene_id}")
        elif not line.startswith(" "):
            break

print(f"找到 {len(genes)} 个基因")
```

---

## 常见映射模式

### 模式1：基因符号 → 多数据库ID

```python
from bioservices import UniProt

def gene_symbol_to_ids(gene_symbol, organism="9606"):
    """将基因符号转换为多数据库ID"""
    u = UniProt()

    # 搜索基因
    query = f"gene:{gene_symbol} AND organism:{organism}"
    result = u.search(query, frmt="tab", columns="id")

    lines = result.strip().split("\n")
    if len(lines) < 2:
        return None

    uniprot_id = lines[1].split("\t")[0]

    # 映射到多个数据库
    ids = {
        'uniprot': uniprot_id,
        'kegg': u.mapping(fr="UniProtKB_AC-ID", to="KEGG", query=uniprot_id),
        'ensembl': u.mapping(fr="UniProtKB_AC-ID", to="Ensembl", query=uniprot_id),
        'refseq': u.mapping(fr="UniProtKB_AC-ID", to="RefSeq_Protein", query=uniprot_id),
        'pdb': u.mapping(fr="UniProtKB_AC-ID", to="PDB", query=uniprot_id)
    }

    return ids

# 使用示例
ids = gene_symbol_to_ids("ZAP70")
print(ids)
```

### 模式2：化合物名称 → 全数据库ID

```python
from bioservices import KEGG, UniChem, ChEBI

def compound_name_to_ids(compound_name):
    """搜索化合物并获取所有数据库ID"""
    k = KEGG()

    # 在KEGG中搜索
    results = k.find("compound", compound_name)
    if not results:
        return None

    # 提取KEGG ID
    kegg_id = results.strip().split("\n")[0].split("\t")[0].replace("cpd:", "")

    # 从KEGG条目获取ChEBI
    entry = k.get(f"cpd:{kegg_id}")
    chebi_id = None
    for line in entry.split("\n"):
        if "ChEBI:" in line:
            parts = line.split("ChEBI:")
            if len(parts) > 1:
                chebi_id = parts[1].strip().split()[0]
                break

    #

# 用法
ids = compound_name_to_ids("Geldanamycin")
print(ids)
```

### 模式三：带错误处理的批量ID转换

```python
from bioservices import UniProt

def safe_batch_mapping(ids, from_db, to_db, chunk_size=100):
    """安全映射ID，包含错误处理和分块机制"""
    u = UniProt()
    all_results = {}

    for i in range(0, len(ids), chunk_size):
        chunk = ids[i:i+chunk_size]
        query = ",".join(chunk)

        try:
            results = u.mapping(fr=from_db, to=to_db, query=query)
            all_results.update(results)
            print(f"✓ 已处理 {min(i+chunk_size, len(ids))}/{len(ids)}")

        except Exception as e:
            print(f"✗ 分块{i}出错: {e}")

            # 尝试处理失败分块中的单个ID
            for single_id in chunk:
                try:
                    result = u.mapping(fr=from_db, to=to_db, query=single_id)
                    all_results.update(result)
                except:
                    all_results[single_id] = None

    return all_results

# 用法
uniprot_ids = ["P43403", "P04637", "P53779", "INVALID123"]
mapping = safe_batch_mapping(uniprot_ids, "UniProtKB_AC-ID", "KEGG")
```

### 模式四：多级映射

有时需要通过中间数据库进行映射：

```python
from bioservices import UniProt

def multi_hop_mapping(gene_symbol, organism="9606"):
    """基因符号 → UniProt → KEGG → 通路"""
    u = UniProt()
    k = KEGG()

    # 步骤1: 基因符号 → UniProt
    query = f"gene:{gene_symbol} AND organism:{organism}"
    result = u.search(query, frmt="tab", columns="id")

    lines = result.strip().split("\n")
    if len(lines) < 2:
        return None

    uniprot_id = lines[1].split("\t")[0]

    # 步骤2: UniProt → KEGG
    kegg_mapping = u.mapping(fr="UniProtKB_AC-ID", to="KEGG", query=uniprot_id)
    if not kegg_mapping or uniprot_id not in kegg_mapping:
        return None

    kegg_id = kegg_mapping[uniprot_id][0]

    # 步骤3: KEGG → 通路
    organism_code, gene_id = kegg_id.split(":")
    pathways = k.get_pathway_by_gene(gene_id, organism_code)

    return {
        'gene': gene_symbol,
        'uniprot': uniprot_id,
        'kegg': kegg_id,
        'pathways': pathways
    }

# 用法
result = multi_hop_mapping("TP53")
print(result)
```

---

## 故障排除

### 问题1：找不到映射

**现象：** 映射返回空值或None

**解决方案：**
1. 确认源ID在源数据库中存在
2. 检查数据库代码拼写
3. 尝试反向映射
4. 某些ID可能无法在所有数据库中找到映射

```python
result = u.mapping(fr="UniProtKB_AC-ID", to="KEGG", query="P43403")

if not result or 'P43403' not in result:
    print("未找到映射。请尝试：")
    print("1. 验证ID是否存在：u.search('P43403')")
    print("2. 检查该蛋白质是否有KEGG注释")
```

### 问题2：批量ID过多

**现象：** 批量映射失败或超时

**解决方案：** 分割成更小的分块

```python
def chunked_mapping(ids, from_db, to_db, chunk_size=50):
    all_results = {}

    for i in range(0, len(ids), chunk_size):
        chunk = ids[i:i+chunk_size]
        result = u.mapping(fr=from_db, to=to_db, query=",".join(chunk))
        all_results.update(result)

    return all_results
```

### 问题3：多个目标ID

**现象：** 一个源ID映射到多个目标ID

**解决方案：** 按列表处理

```python
result = u.mapping(fr="UniProtKB_AC-ID", to="PDB", query="P04637")
# 结果：{'P04637': ['1A1U', '1AIE', '1C26', ...]}

pdb_ids = result['P04637']
print(f"找到{len(pdb_ids)}个PDB结构")

for pdb_id in pdb_ids:
    print(f"  {pdb_id}")
```

### 问题4：物种歧义

**现象：** 基因符号映射到多个物种

**解决方案：** 搜索时始终指定物种

```python
# 错误：存在歧义
result = u.search("gene:TP53")  # 多个物种都有TP53

# 正确：明确指定
result = u.search("gene:TP53 AND organism:9606")  # 仅限人类
```

### 问题5：过期的ID

**现象：** 旧数据库ID无法映射

**解决方案：** 先更新为当前ID

```python
# 检查ID是否有效
entry = u.retrieve("P43403", frmt="txt")

# 查找次级编号
for line in entry.split("\n"):
    if line.startswith("AC"):
        print(line)  # 显示主编号和次级编号
```

---

## 最佳实践

1. **始终验证输入**后再进行批量处理
2. **优雅处理空结果**
3. **使用分块机制**处理大型ID列表（每块50-100个）
4. **缓存结果**用于重复查询
5. **尽可能指定物种**避免歧义
6. **记录失败操作**以便后续重试
7. **批量操作间添加延迟**以遵守API限制

```python
import time

def polite_batch_mapping(ids, from_db, to_db):
    """带速率限制的批量映射"""
    results = {}

    for i in range(0, len(ids), 50):
        chunk = ids[i:i+50]
        result = u.mapping(fr=from_db, to=to_db, query=",".join(chunk))
        results.update(result)

        time.sleep(0.5)  # 遵守API规范

    return results
```

---

完整工作示例请参见：
- `scripts/batch_id_converter.py`：命令行批量转换工具
- `workflow_patterns.md`：集成到大型工作流中的模式
