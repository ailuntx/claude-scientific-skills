# NCBI 蛋白质 API 参考

## 概述
通过 NCBI E-utilities 使用 `db=protein` 可访问蛋白质序列记录（RefSeq、GenBank、UniProt 导入数据）。

## 基础 URL
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/
```

## 认证
- **API密钥**（推荐）：在 https://www.ncbi.nlm.nih.gov/account/ 注册后附加 `&api_key=YOUR_KEY`
- 无密钥：每秒 3 次请求。有密钥：每秒 10 次请求
- 需提供 `tool` 和 `email` 参数用于身份识别

## 核心端点

### 1. ESearch -- 搜索蛋白质记录
```
GET esearch.fcgi?db=protein&term=QUERY&retmax=N&retmode=json
```
| 参数 | 说明 |
|-------|-------------|
| `term` | 搜索查询（Entrez语法）。字段：`[Protein Name]`, `[Organism]`, `[Accession]`, `[Gene Name]` |
| `retmax` | 返回最大ID数（默认20，上限100000） |
| `retstart` | 分页偏移量 |
| `usehistory` | 设为 `y` 可在服务器存储结果（适用于大数据集） |

**示例 -- 搜索人胰岛素：**
```
GET esearch.fcgi?db=protein&term=insulin+AND+homo+sapiens[Organism]&retmax=5&retmode=json
```
响应（JSON）：
```json
{
  "esearchresult": {
    "count": "1523",
    "retmax": "5",
    "idlist": ["116734704", "AAA59172.1", "NP_000198.1", ...],
    "querytranslation": "insulin AND \"Homo sapiens\"[Organism]"
  }
}
```

### 2. EFetch -- 获取蛋白质记录
```
GET efetch.fcgi?db=protein&id=IDS&rettype=TYPE&retmode=MODE
```
| rettype | retmode | 输出格式 |
|---------|---------|--------|
| `fasta` | `text` | FASTA序列 |
| `gp` | `text` | GenPept平面文件 |
| `gp` | `xml` | GenPept XML (INSDSeq) |
| `acc` | `text` | 登录号列表 |
| `seqid` | `text` | SeqID列表 |
| `ft` | `text` | 特征表 |

**示例 -- 获取 NP_000198.1（人胰岛素）的FASTA：**
```
GET efetch.fcgi?db=protein&id=NP_000198.1&rettype=fasta&retmode=text
```
响应：
```
>NP_000198.1 insulin preproprotein [Homo sapiens]
MALWMRLLPLLALLALWGPDPAAAFVNQHLCGSHLVEALYLVCGERGFFYTPKTRREAED
LQVGQVELGGGPGAGSLQPLALEGSLQKRGIVEQCCTSICSLYQLENYCN
```

**示例 -- 获取多个ID的GenPept XML：**
```
GET efetch.fcgi?db=protein&id=NP_000198.1,NP_001278826.1&rettype=gp&retmode=xml
```

### 3. ESummary -- 记录摘要
```
GET esummary.fcgi?db=protein&id=IDS&retmode=json
```
返回：登录号、标题、生物体、序列长度、分类信息、创建/更新日期

### 4. ELink -- 查找关联记录
```
GET elink.fcgi?dbfrom=protein&db=gene&id=NP_000198.1
```
关联蛋白质到基因、核苷酸、结构、分类等数据

## 常用搜索模式
```
# 按登录号
term=NP_000198.1[Accession]

# 按基因名+生物体
term=BRCA1[Gene Name] AND human[Organism]

# 仅RefSeq
term=insulin AND srcdb_refseq[Properties]

# 按序列长度范围
term=100:500[Sequence Length] AND kinase[Protein Name]
```

## 速率限制
- 无API密钥：每秒 3 次请求
- 有API密钥：每秒 10 次请求
- 批量下载：使用 `usehistory=y` 配合 `WebEnv`/`query_key`，每次获取500条记录
