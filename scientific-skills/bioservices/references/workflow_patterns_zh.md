# BioServices：常用工作流模式

本文档详细描述了使用BioServices执行常见生物信息学任务的多步骤工作流。

## 目录

1. [完整蛋白质分析流程](#complete-protein-analysis-pipeline)
2. [通路发现与网络分析](#pathway-discovery-and-network-analysis)
3. [化合物多数据库检索](#compound-multi-database-search)
4. [批量标识符转换](#batch-identifier-conversion)
5. [基因功能注释](#gene-functional-annotation)
6. [蛋白质互作网络构建](#protein-interaction-network-construction)
7. [多物种比较分析](#multi-organism-comparative-analysis)

---

## 完整蛋白质分析流程

**目标：** 给定蛋白质名称，获取序列、寻找同源物、识别通路并发现互作关系。

**示例：** 分析人类ZAP70蛋白

### 步骤1：UniProt检索与标识符获取

```python
from bioservices import UniProt

u = UniProt(verbose=False)

# 通过名称搜索蛋白质
query = "ZAP70_HUMAN"
results = u.search(query, frmt="tab", columns="id,genes,organism,length")

# 解析结果
lines = results.strip().split("\n")
if len(lines) > 1:
    header = lines[0]
    data = lines[1].split("\t")
    uniprot_id = data[0]  # 例如 P43403
    gene_names = data[1]   # 例如 ZAP70

print(f"UniProt ID: {uniprot_id}")
print(f"Gene names: {gene_names}")
```

**输出：**
- UniProt登录号：P43403
- 基因名称：ZAP70

### 步骤2：序列获取

```python
# 获取FASTA序列
sequence = u.retrieve(uniprot_id, frmt="fasta")
print(sequence)

# 提取纯序列字符串（移除头部）
seq_lines = sequence.split("\n")
sequence_only = "".join(seq_lines[1:])  # 跳过FASTA头部
```

**输出：** FASTA格式的完整蛋白质序列

### 步骤3：BLAST相似性搜索

```python
from bioservices import NCBIblast
import time

s = NCBIblast(verbose=False)

# 提交BLAST任务
jobid = s.run(
    program="blastp",
    sequence=sequence_only,
    stype="protein",
    database="uniprotkb",
    email="your.email@example.com"
)

print(f"BLAST任务ID: {jobid}")

# 等待完成
while True:
    status = s.getStatus(jobid)
    print(f"状态: {status}")
    if status == "FINISHED":
        break
    elif status == "ERROR":
        print("BLAST任务失败")
        break
    time.sleep(5)

# 获取结果
if status == "FINISHED":
    blast_results = s.getResult(jobid, "out")
    print(blast_results[:500])  # 打印前500个字符
```

**输出：** 显示相似蛋白质的BLAST比对结果

### 步骤4：KEGG通路发现

```python
from bioservices import KEGG

k = KEGG()

# 通过UniProt映射获取KEGG基因ID
kegg_mapping = u.mapping(fr="UniProtKB_AC-ID", to="KEGG", query=uniprot_id)
print(f"KEGG映射: {kegg_mapping}")

# 提取KEGG基因ID（例如 hsa:7535）
if kegg_mapping:
    kegg_gene_id = kegg_mapping[uniprot_id][0] if uniprot_id in kegg_mapping else None

    if kegg_gene_id:
        # 查找包含该基因的通路
        organism = kegg_gene_id.split(":")[0]  # 例如 "hsa"
        gene_id = kegg_gene_id.split(":")[1]   # 例如 "7535"

        pathways = k.get_pathway_by_gene(gene_id, organism)
        print(f"发现{len(pathways)}条通路:")

        # 获取通路名称
        for pathway_id in pathways:
            pathway_info = k.get(pathway_id)
            # 解析NAME行
            for line in pathway_info.split("\n"):
                if line.startswith("NAME"):
                    pathway_name = line.replace("NAME", "").strip()
                    print(f"  {pathway_id}: {pathway_name}")
                    break
```

**输出：**
- path:hsa04064 - NF-κB信号通路
- path:hsa04650 - 自然杀伤细胞介导的细胞毒性
- path:hsa04660 - T细胞受体信号通路
- path:hsa04662 - B细胞受体信号通路

### 步骤5：蛋白质-蛋白质互作

```python
from bioservices import PSICQUIC

p = PSICQUIC()

# 在MINT数据库中查询人类（分类号:9606）互作
query = f"ZAP70 AND species:9606"
interactions = p.query("mint", query)

# 解析PSI-MI TAB格式结果
if interactions:
    interaction_lines = interactions.strip().split("\n")
    print(f"发现{len(interaction_lines)}个互作关系")

    # 打印前几个互作
    for line in interaction_lines[:5]:
        fields = line.split("\t")
        protein_a = fields[0]
        protein_b = fields[1]
        interaction_type = fields[11]
        print(f"  {protein_a} - {protein_b}: {interaction_type}")
```

**输出：** 与ZAP70互作的蛋白质列表

### 步骤6：基因本体注释

```python
from bioservices import QuickGO

g = QuickGO()

# 获取蛋白质的GO注释
annotations = g.Annotation(protein=uniprot_id, format="tsv")

if annotations:
    # 解析TSV结果
    lines = annotations.strip().split("\n")
    print(f"发现{len(lines)-1}个GO注释")

    # 显示前几个注释
    for line in lines[1:6]:  # 跳过头部
        fields = line.split("\t")
        go_id = fields[6]
        go_term = fields[7]
        go_aspect = fields[8]
        print(f"  {go_id}: {go_term} [{go_aspect}]")
```

**输出：** 注释ZAP70功能、过程和位置的GO术语

### 完整流程总结

**输入：** 蛋白质名称（例如"ZAP70_HUMAN"）

**输出：**
1. UniProt登录号和基因名称
2. 蛋白质序列（FASTA）
3. 相似蛋白质（BLAST结果）
4. 生物通路（KEGG）
5. 互作伙伴（PSICQUIC）
6. 功能注释（GO术语）

**脚本：** `scripts/protein_analysis_workflow.py` 自动化此完整流程。

---

## 通路发现与网络分析

**目标：** 分析生物体的所有通路并提取蛋白质互作网络。

**示例：** 人类（hsa）通路分析

### 步骤1：获取生物体所有通路

```python
from bioservices import KEGG

k = KEGG()
k.organism = "hsa"

# 获取所有通路ID
pathway_ids = k.pathwayIds
print(f"为{k.organism}发现{len(pathway_ids)}条通路")

# 显示前几个
for pid in pathway_ids[:10]:
    print(f"  {pid}")
```

**输出：** 约300条人类通路列表

### 步骤2：解析通路互作关系

```python
# 分析特定通路
pathway_id = "hsa04660"  # T细胞受体信号通路

# 获取KGML数据
kgml_data = k.parse_kgml_pathway(pathway_id)

# 提取条目（基因/蛋白质）
entries = kgml_data['entries']
print(f"通路包含{len(entries)}个条目")

# 提取关系（互作）
relations = kgml_data['relations']
print(f"发现{len(relations)}个关系")

# 分析关系类型
relation_types = {}
for rel in relations:
    rel_type = rel.get('name', 'unknown')
    relation_types[rel_type] = relation_types.get(rel_type, 0) + 1

print("\n关系类型分布:")
for rel_type, count in sorted(relation_types.items()):
    print(f"  {rel_type}: {count}")
```

**输出：**
- 条目计数（通路中的基因/蛋白质）
- 关系计数（互作）
- 互作类型分布（激活、抑制、结合等）

### 步骤3：提取蛋白质-蛋白质互作

```python
# 筛选特定互作类型
pprel_interactions = [
    rel for rel in relations
    if rel.get('link') == 'PPrel'  # 蛋白质-蛋白质关系
]

print(f"发现{len(pprel_interactions)}个蛋白质-蛋白质互作")

# 提取互作详情
for rel in pprel_interactions[:10]:
    entry1 = rel['entry1']
    entry2 = rel['entry2']
    interaction_type = rel.get('name', 'unknown')

    print(f"  {entry1} -> {entry2}: {interaction_type}")
```

**输出：** 带类型的定向蛋白质-蛋白质互作

### 步骤4：转换为网络格式（SIF）

```python
# 获取简单互作格式（筛选关键互作）
sif_data = k.pathway2sif(pathway_id)

# SIF格式：源节点, 互作类型, 目标节点
print("\n简单互作格式:")
for interaction in sif_data[:10]:
    print(f"  {interaction}")
```

**输出：** 适用于Cytoscape或NetworkX的网络边

### 步骤5：批量分析所有通路

```python
import pandas as pd

# 分析所有通路（耗时操作！）
all_results = []

for pathway_id in pathway_ids[:50]:  # 示例限制数量
    try:
        kgml = k.parse_kgml_pathway(pathway_id)

        result = {
            'pathway_id': pathway_id,
            'num_entries': len(kgml.get('entries', [])),
            'num_relations': len(kgml.get('relations', []))
        }

        all_results.append(result)

    except Exception as e:
        print(f"解析{pathway_id}时出错: {e}")

# 创建数据框
df = pd.DataFrame(all_results)
print(df.describe())

# 查找最大通路
print("\n最大通路:")
print(df.nlargest(10, 'num_entries')[['pathway_id', 'num_entries', 'num_relations']])
```

**输出：** 通路规模和互作密度的统计摘要

**脚本：** `scripts/pathway_analysis.py` 实现此工作流并支持导出选项。

---

## 化合物多数据库检索

**目标：** 通过名称搜索化合物并在KEGG、ChEBI和ChEMBL中获取标识符。

**示例：** 格尔德霉素（抗生素）

### 步骤1：检索KEGG化合物数据库

```python
from bioservices import KEGG

k = KEGG()

# 通过化合物名称搜索
compound_name = "Geldanamycin"
results = k.find("compound", compound_name)

print(f"'{compound_name}'的KEGG搜索结果:")
print(results)

# 提取化合物ID
if results:
    lines = results.strip().split("\n")
    if lines:
        kegg_id = lines[0].split("\t")[0]  # 例如 cpd:C11222
        kegg_id_clean = kegg_id.replace("cpd:", "")  # C11222
        print(f"\nKEGG化合物ID: {kegg_id_clean}")
```

**输出：** KEGG ID（例如 C11222）

### 步骤2：获取带数据库链接的KEGG条目

```python
# 获取化合物条目
compound_entry = k.get(kegg_id)

# 解析条目中的数据库链接
chebi_id = None
for line in compound_entry.split("\n"):
    if "ChEBI:" in line:
        # 提取ChEBI ID
        parts = line.split("ChEBI:")
        if len(parts) > 1:
            chebi_id = parts[1].strip().split()[0]
            print(f"ChEBI ID: {chebi_id}")
            break

# 显示条目片段
print("\nKEGG条目（前500字符）:")
print(compound_entry[:500])
```

**输出：** ChEBI ID（例如 5292）和化合物信息

### 步骤3：通过UniChem交叉引用ChEMBL

```python
from bioservices import UniChem

u = UniChem()

# KEGG → ChEMBL转换
try:
    chembl_id = u.get_compound_id_from_kegg(kegg_id_clean)
    print(f"ChEMBL ID: {chembl_id}")
except Exception as e:
    print(f"UniChem查询失败: {e}")
    chembl_id = None
```

**输出：** ChEMBL ID（例如 CHEMBL278315）

### 步骤4：获取详细信息

```python
# 获取ChEBI信息
if chebi_id:
    from bioservices import ChEBI
    c = ChEBI()

    try:
        chebi_entity = c.getCompleteEntity(f"CHEBI:{chebi_id}")
        print(f"\nChEBI分子式: {chebi_entity.Formulae}")
        print(f"ChEBI名称: {chebi_entity.chebiAsciiName}")
    except Exception as e:
        print(f"ChEBI查询失败: {e}")

# 获取ChEMBL信息
if chembl_id:
    from bioservices import ChEMBL
    chembl = ChEMBL()

    try:
        chembl_compound = chembl.get_compound_by_chemblId(chembl_id)
        print(f"\nChEMBL分子量: {chembl_compound['molecule_properties']['full_mwt']}")
        print(f"ChEMBL SMILES: {chembl_compound['molecule_structures']['canonical_smiles']}")
    except Exception as e:
        print(f"ChEMBL查询失败: {e}")
```

**输出：** 来自多个数据库的化学属性

### 完整化合物工作流总结

**输入：** 化合物名称（例如"Geldanamycin"）

**输出：**
- KEGG ID: C11222
- ChEBI ID: 5292
- ChEMBL ID: CHEMBL278315
- 化学分子式
- 分子量
- SMILES结构

**脚本：** `scripts/compound_cross_reference.py` 自动化此工作流。

---

## 批量标识符转换

**目标：** 在数据库间高效转换多个标识符。

### 批量UniProt → KEGG映射

```python
from bioservices import UniProt

u = UniProt()

# UniProt ID列表
uniprot_ids = ["P43403", "P04637", "P53779", "Q9Y6K9"]

# 批量映射（逗号分隔）
query_string = ",".join(uniprot_ids)
results = u.mapping(fr="UniProtKB_AC-ID", to="KEGG", query=query_string)

print("UniProt → KEGG映射:")
for uniprot_id, kegg_ids in results.items():
    print(f"  {uniprot_id} → {kegg_ids}")
```

**输出：** 将每个UniProt ID映射到KEGG基因ID的字典

### 批量文件处理

```python
import csv

# 从文件读取标识符
def read_ids_from_file(filename):
    with open(filename, 'r') as f:
        ids = [line.strip() for line in f if line.strip()]
    return ids

# 分块处理（考虑API限制）
def batch_convert(ids, from_db, to_db, chunk_size=100):
    u = UniProt()
    all_results = {}

    for i in range(0, len(ids), chunk_size):
        chunk = ids[i:i+chunk_size]
        query = ",".join(chunk)

        try:
            results = u.mapping(fr=from_db, to=to_db, query=query)
            all_results.update(results)
            print(f"已处理 {min(i+chunk_size, len(ids))}/{len(ids)}")
        except Exception as e:
            print(f"处理分块{i}时出错: {e}")

    return all_results

# 将映射结果写入CSV
def write_mapping_to_csv(mapping, output_file):
    with open(output_file, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['源标识符', '目标标识符'])

        for source_id, target_ids in mapping.items():
            target_str = ";".join(target_ids) if target_ids else "无映射"
            writer.writerow([source_id, target_str])

# 使用示例
input_ids = read_ids_from_file("uniprot_ids.txt")
mapping = batch_convert(input_ids, "UniProtKB_AC-ID", "KEGG", chunk_size=50)
```

```python
write_mapping_to_csv(mapping, "uniprot_to_kegg_mapping.csv")
```

**脚本：** `scripts/batch_id_converter.py` 提供命令行批量转换功能。

---

## 基因功能注释

**目标：** 获取基因的全面功能信息。

### 工作流程

```python
from bioservices import UniProt, KEGG, QuickGO

# 目标基因
gene_symbol = "TP53"

# 1. 查找UniProt条目
u = UniProt()
search_results = u.search(f"gene:{gene_symbol} AND organism:9606",
                          frmt="tab",
                          columns="id,genes,protein names")

# 提取UniProt ID
lines = search_results.strip().split("\n")
if len(lines) > 1:
    uniprot_id = lines[1].split("\t")[0]
    protein_name = lines[1].split("\t")[2]
    print(f"蛋白质: {protein_name}")
    print(f"UniProt ID: {uniprot_id}")

# 2. 获取KEGG通路
kegg_mapping = u.mapping(fr="UniProtKB_AC-ID", to="KEGG", query=uniprot_id)
if uniprot_id in kegg_mapping:
    kegg_id = kegg_mapping[uniprot_id][0]

    k = KEGG()
    organism, gene_id = kegg_id.split(":")
    pathways = k.get_pathway_by_gene(gene_id, organism)

    print(f"\n通路 ({len(pathways)}条):")
    for pathway_id in pathways[:5]:
        print(f"  {pathway_id}")

# 3. 获取GO注释
g = QuickGO()
go_annotations = g.Annotation(protein=uniprot_id, format="tsv")

if go_annotations:
    lines = go_annotations.strip().split("\n")
    print(f"\nGO注释 (共{len(lines)-1}条):")

    # 按类别分组
    aspects = {"P": [], "F": [], "C": []}
    for line in lines[1:]:
        fields = line.split("\t")
        go_aspect = fields[8]  # P, F 或 C
        go_term = fields[7]
        aspects[go_aspect].append(go_term)

    print(f"  生物过程: {len(aspects['P'])}条术语")
    print(f"  分子功能: {len(aspects['F'])}条术语")
    print(f"  细胞组分: {len(aspects['C'])}条术语")

# 4. 获取蛋白质序列特征
full_entry = u.retrieve(uniprot_id, frmt="txt")
print("\n蛋白质特征:")
for line in full_entry.split("\n"):
    if line.startswith("FT   DOMAIN"):
        print(f"  {line}")
```

**输出：** 包含名称、通路、GO术语和特征的全面注释。

---

## 蛋白质互作网络构建

**目标：** 为蛋白质集合构建蛋白质互作网络。

### 工作流程

```python
from bioservices import PSICQUIC
import networkx as nx

# 目标蛋白质
proteins = ["ZAP70", "LCK", "LAT", "SLP76", "PLCg1"]

# 初始化PSICQUIC
p = PSICQUIC()

# 构建网络
G = nx.Graph()

for protein in proteins:
    # 查询人类互作
    query = f"{protein} AND species:9606"

    try:
        results = p.query("intact", query)

        if results:
            lines = results.strip().split("\n")

            for line in lines:
                fields = line.split("\t")
                # 提取蛋白质名称（简化）
                protein_a = fields[4].split(":")[1] if ":" in fields[4] else fields[4]
                protein_b = fields[5].split(":")[1] if ":" in fields[5] else fields[5]

                # 添加边
                G.add_edge(protein_a, protein_b)

    except Exception as e:
        print(f"查询{protein}时出错: {e}")

print(f"网络: {G.number_of_nodes()}个节点, {G.number_of_edges()}条边")

# 分析网络
print("\n节点度:")
for node in proteins:
    if node in G:
        print(f"  {node}: {G.degree(node)}个互作")

# 导出可视化
nx.write_gml(G, "protein_network.gml")
print("\n网络已导出至protein_network.gml")
```

**输出：** 导出为GML格式的NetworkX图，用于Cytoscape可视化。

---

## 多物种比较分析

**目标：** 比较不同物种间的通路或基因存在情况。

### 工作流程

```python
from bioservices import KEGG

k = KEGG()

# 待比较物种
organisms = ["hsa", "mmu", "dme", "sce"]  # 人类、小鼠、果蝇、酵母
organism_names = {
    "hsa": "人类",
    "mmu": "小鼠",
    "dme": "果蝇",
    "sce": "酵母"
}

# 目标通路
pathway_name = "cell cycle"

print(f"在以下物种中搜索'{pathway_name}'通路:\n")

for org in organisms:
    k.organism = org

    # 搜索通路
    results = k.lookfor_pathway(pathway_name)

    print(f"{organism_names[org]} ({org}):")
    if results:
        for pathway in results[:3]:  # 显示前3条
            print(f"  {pathway}")
    else:
        print("  未找到匹配项")
    print()
```

**输出：** 跨物种的通路存在/缺失情况。

---

## 工作流最佳实践

### 1. 错误处理

始终封装服务调用：
```python
try:
    result = service.method(params)
    if result:
        # 处理结果
        pass
except Exception as e:
    print(f"错误: {e}")
```

### 2. 速率限制

批处理时添加延迟：
```python
import time

for item in items:
    result = service.query(item)
    time.sleep(0.5)  # 500毫秒延迟
```

### 3. 结果验证

检查空值或异常结果：
```python
if result and len(result) > 0:
    # 处理结果
    pass
else:
    print("未返回结果")
```

### 4. 进度报告

长工作流中：
```python
total = len(items)
for i, item in enumerate(items):
    # 处理条目
    if (i + 1) % 10 == 0:
        print(f"已处理 {i+1}/{total}")
```

### 5. 数据导出

保存中间结果：
```python
import json

with open("results.json", "w") as f:
    json.dump(results, f, indent=2)
```

---

## 与其他工具集成

### BioPython集成

```python
from bioservices import UniProt
from Bio import SeqIO
from io import StringIO

u = UniProt()
fasta_data = u.retrieve("P43403", "fasta")

# 使用BioPython解析
fasta_io = StringIO(fasta_data)
record = SeqIO.read(fasta_io, "fasta")

print(f"序列长度: {len(record.seq)}")
print(f"描述: {record.description}")
```

### Pandas集成

```python
from bioservices import UniProt
import pandas as pd
from io import StringIO

u = UniProt()
results = u.search("zap70", frmt="tab", columns="id,genes,length,organism")

# 加载到DataFrame
df = pd.read_csv(StringIO(results), sep="\t")
print(df.head())
print(df.describe())
```

### NetworkX集成

参见蛋白质互作网络构建部分。

---

完整示例代码请参见 `scripts/` 目录中的脚本。
