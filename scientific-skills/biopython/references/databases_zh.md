# 使用 Bio.Entrez 访问数据库

## 概述

Bio.Entrez 提供了对 NCBI Entrez 数据库（包括 PubMed、GenBank、Gene、Protein、Nucleotide 等）的程序化访问。它处理了所有 API 调用、速率限制和数据解析的复杂性。

## 设置与配置

### 电子邮箱（必需）

NCBI 要求提供电子邮箱以跟踪使用情况并在出现问题时联系用户：

```python
from Bio import Entrez

# 务必设置您的邮箱
Entrez.email = "your.email@example.com"
```

### API 密钥（推荐）

使用 API 密钥可将速率限制从每秒 3 次提升至 10 次请求：

```python
# 从以下网址获取 API 密钥：https://www.ncbi.nlm.nih.gov/account/settings/
Entrez.api_key = "your_api_key_here"
```

### 速率限制

Biopython 自动遵守 NCBI 速率限制：
- **无 API 密钥**：每秒 3 次请求
- **有 API 密钥**：每秒 10 次请求

该模块会自动处理此限制，因此无需在请求间添加延迟。

## 核心 Entrez 功能

### EInfo - 数据库信息

获取可用数据库及其统计信息：

```python
# 列出所有数据库
handle = Entrez.einfo()
result = Entrez.read(handle)
print(result["DbList"])

# 获取特定数据库信息
handle = Entrez.einfo(db="pubmed")
result = Entrez.read(handle)
print(result["DbInfo"]["Description"])
print(result["DbInfo"]["Count"])  # 记录数量
```

### ESearch - 数据库搜索

搜索记录并获取其 ID：

```python
# 在 PubMed 中搜索
handle = Entrez.esearch(db="pubmed", term="biopython")
result = Entrez.read(handle)
handle.close()

id_list = result["IdList"]
count = result["Count"]
print(f"找到 {count} 条结果")
print(f"获取的 ID：{id_list}")
```

### 高级 ESearch 参数

```python
# 使用附加参数搜索
handle = Entrez.esearch(
    db="pubmed",
    term="biopython[Title]",
    retmax=100,           # 最多返回 100 个 ID
    sort="relevance",     # 按相关性排序
    reldate=365,          # 仅限过去一年的结果
    datetype="pdat"       # 使用发布日期
)
result = Entrez.read(handle)
handle.close()
```

### ESummary - 获取记录摘要

检索 ID 列表的摘要信息：

```python
# 获取多条记录的摘要
handle = Entrez.esummary(db="pubmed", id="19304878,18606172")
results = Entrez.read(handle)
handle.close()

for record in results:
    print(f"标题：{record['Title']}")
    print(f"作者：{record['AuthorList']}")
    print(f"期刊：{record['Source']}")
    print()
```

### EFetch - 检索完整记录

以多种格式获取完整记录：

```python
# 获取 GenBank 记录
handle = Entrez.efetch(db="nucleotide", id="EU490707", rettype="gb", retmode="text")
record_text = handle.read()
handle.close()

# 使用 SeqIO 解析
from Bio import SeqIO
handle = Entrez.efetch(db="nucleotide", id="EU490707", rettype="gb", retmode="text")
record = SeqIO.read(handle, "genbank")
handle.close()
print(record.description)
```

### EFetch 返回类型

不同数据库支持不同的返回类型：

**核酸/蛋白质：**
- `rettype="fasta"` - FASTA 格式
- `rettype="gb"` 或 `"genbank"` - GenBank 格式
- `rettype="gp"` - GenPept 格式（蛋白质）

**PubMed：**
- `rettype="medline"` - MEDLINE 格式
- `rettype="abstract"` - 摘要文本

**常用模式：**
- `retmode="text"` - 纯文本
- `retmode="xml"` - XML 格式

### ELink - 查找相关记录

查找不同数据库间的关联记录：

```python
# 查找与核酸记录关联的蛋白质记录
handle = Entrez.elink(dbfrom="nucleotide", db="protein", id="EU490707")
result = Entrez.read(handle)
handle.close()

# 提取关联 ID
for linkset in result[0]["LinkSetDb"]:
    if linkset["LinkName"] == "nucleotide_protein":
        protein_ids = [link["Id"] for link in linkset["Link"]]
        print(f"关联蛋白质 ID：{protein_ids}")
```

### EPost - 上传 ID 列表

将大型 ID 列表上传至服务器供后续使用：

```python
# 将 ID 发布到服务器
id_list = ["19304878", "18606172", "16403221"]
handle = Entrez.epost(db="pubmed", id=",".join(id_list))
result = Entrez.read(handle)
handle.close()

# 获取后续使用的 query_key 和 WebEnv
query_key = result["QueryKey"]
webenv = result["WebEnv"]

# 在后续查询中使用
handle = Entrez.efetch(
    db="pubmed",
    query_key=query_key,
    WebEnv=webenv,
    rettype="medline",
    retmode="text"
)
```

### EGQuery - 全局查询

一次性在所有 Entrez 数据库中搜索：

```python
handle = Entrez.egquery(term="biopython")
result = Entrez.read(handle)
handle.close()

for row in result["eGQueryResult"]:
    print(f"{row['DbName']}: {row['Count']} 条结果")
```

### ESpell - 拼写建议

获取搜索词的拼写建议：

```python
handle = Entrez.espell(db="pubmed", term="biopythn")
result = Entrez.read(handle)
handle.close()

print(f"原始查询：{result['Query']}")
print(f"建议修正：{result['CorrectedQuery']}")
```

## 操作不同数据库

### PubMed

```python
# 搜索文章
handle = Entrez.esearch(db="pubmed", term="cancer genomics", retmax=10)
result = Entrez.read(handle)
handle.close()

# 获取摘要
handle = Entrez.efetch(
    db="pubmed",
    id=result["IdList"],
    rettype="medline",
    retmode="text"
)
records = handle.read()
handle.close()
print(records)
```

### GenBank / 核酸数据库

```python
# 搜索序列
handle = Entrez.esearch(db="nucleotide", term="Cypripedioideae[Orgn] AND matK[Gene]")
result = Entrez.read(handle)
handle.close()

# 获取序列
if result["IdList"]:
    handle = Entrez.efetch(
        db="nucleotide",
        id=result["IdList"][:5],
        rettype="fasta",
        retmode="text"
    )
    sequences = handle.read()
    handle.close()
```

### 蛋白质数据库

```python
# 搜索蛋白质序列
handle = Entrez.esearch(db="protein", term="human insulin")
result = Entrez.read(handle)
handle.close()

# 获取蛋白质记录
from Bio import SeqIO
handle = Entrez.efetch(
    db="protein",
    id=result["IdList"][:5],
    rettype="gp",
    retmode="text"
)
records = SeqIO.parse(handle, "genbank")
for record in records:
    print(f"{record.id}: {record.description}")
handle.close()
```

### 基因数据库

```python
# 搜索基因记录
handle = Entrez.esearch(db="gene", term="BRCA1[Gene] AND human[Organism]")
result = Entrez.read(handle)
handle.close()

# 获取基因信息
handle = Entrez.efetch(db="gene", id=result["IdList"][0], retmode="xml")
record = Entrez.read(handle)
handle.close()
```

### 分类数据库

```python
# 搜索生物体
handle = Entrez.esearch(db="taxonomy", term="Homo sapiens")
result = Entrez.read(handle)
handle.close()

# 获取分类信息
handle = Entrez.efetch(db="taxonomy", id=result["IdList"][0], retmode="xml")
records = Entrez.read(handle)
handle.close()

for record in records:
    print(f"分类 ID：{record['TaxId']}")
    print(f"学名：{record['ScientificName']}")
    print(f"谱系：{record['Lineage']}")
```

## 解析 Entrez 结果

### 读取 XML 结果

```python
# 大多数结果可通过 Entrez.read() 解析
handle = Entrez.efetch(db="pubmed", id="19304878", retmode="xml")
records = Entrez.read(handle)
handle.close()

# 访问解析后的数据
article = records['PubmedArticle'][0]['MedlineCitation']['Article']
print(article['ArticleTitle'])
```

### 处理大型结果集

```python
# 大型搜索的批量处理
search_term = "cancer[Title]"
handle = Entrez.esearch(db="pubmed", term=search_term, retmax=0)
result = Entrez.read(handle)
handle.close()

total_count = int(result["Count"])
batch_size = 500

for start in range(0, total_count, batch_size):
    # 获取批次数据
    handle = Entrez.esearch(
        db="pubmed",
        term=search_term,
        retstart=start,
        retmax=batch_size
    )
    result = Entrez.read(handle)
    handle.close()

    # 处理 ID
    id_list = result["IdList"]
    print(f"正在处理 ID {start} 至 {start + len(id_list)}")
```

## 高级模式

### 使用 WebEnv 的搜索历史

```python
# 执行搜索并存储在服务器
handle = Entrez.esearch(
    db="pubmed",
    term="biopython",
    usehistory="y"
)
result = Entrez.read(handle)
handle.close()

webenv = result["WebEnv"]
query_key = result["QueryKey"]
count = int(result["Count"])

# 使用历史记录分批获取结果
batch_size = 100
for start in range(0, count, batch_size):
    handle = Entrez.efetch(
        db="pubmed",
        retstart=start,
        retmax=batch_size,
        rettype="medline",
        retmode="text",
        webenv=webenv,
        query_key=query_key
    )
    data = handle.read()
    handle.close()
    # 处理数据
```

### 组合搜索

```python
# 使用布尔运算符
complex_search = "(cancer[Title]) AND (genomics[Title]) AND 2020:2025[PDAT]"
handle = Entrez.esearch(db="pubmed", term=complex_search, retmax=100)
result = Entrez.read(handle)
handle.close()
```

## 最佳实践

1. **始终设置 Entrez.email** - NCBI 强制要求
2. **使用 API 密钥**提高速率限制（10 次/秒 vs 3 次/秒）
3. **读取后关闭句柄**释放资源
4. **批量处理大型请求** - 使用 retstart 和 retmax 分页
5. **大型下载使用 WebEnv** - 将结果存储在服务器
6. **本地缓存** - 下载后保存避免重复请求
7. **优雅处理错误** - 可能出现网络问题和 API 限制
8. **遵守 NCBI 指南** - 不要过度使用服务
9. **选择合适的返回类型** - 匹配需求格式
10. **谨慎解析 XML** - 结构因数据库和记录类型而异

## 错误处理

```python
from urllib.error import HTTPError
from Bio import Entrez

Entrez.email = "your.email@example.com"

try:
    handle = Entrez.efetch(db="nucleotide", id="invalid_id", rettype="gb")
    record = handle.read()
    handle.close()
except HTTPError as e:
    print(f"HTTP 错误：{e.code} - {e.reason}")
except Exception as e:
    print(f"错误：{e}")
```

## 常见用例

### 下载 GenBank 记录

```python
from Bio import Entrez, SeqIO

Entrez.email = "your.email@example.com"

# 登录号列表
accessions = ["EU490707", "EU490708", "EU490709"]

for acc in accessions:
    handle = Entrez.efetch(db="nucleotide", id=acc, rettype="gb", retmode="text")
    record = SeqIO.read(handle, "genbank")
    handle.close()

    # 保存到文件
    SeqIO.write(record, f"{acc}.gb", "genbank")
```

### 搜索并下载论文

```python
# 在 PubMed 搜索
handle = Entrez.esearch(db="pubmed", term="machine learning bioinformatics", retmax=20)
result = Entrez.read(handle)
handle.close()

# 获取详情
handle = Entrez.efetch(db="pubmed", id=result["IdList"], retmode="xml")
papers = Entrez.read(handle)
handle.close()

# 提取信息
for paper in papers['PubmedArticle']:
    article = paper['MedlineCitation']['Article']
    print(f"标题：{article['ArticleTitle']}")
    print(f"期刊：{article['Journal']['Title']}")
    print()
```

### 查找相关序列

```python
# 从单条序列开始
handle = Entrez.efetch(db="nucleotide", id="EU490707", rettype="gb", retmode="text")
record = SeqIO.read(handle, "genbank")
handle.close()

# 查找相似序列
handle = Entrez.elink(dbfrom="nucleotide", db="nucleotide", id="EU490707")
result = Entrez.read(handle)
handle.close()

# 获取相关 ID
related_ids = []
for linkset in result[0]["LinkSetDb"]:
    for link in linkset["Link"]:
        related_ids.append(link["Id"])
```
