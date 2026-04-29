# 欧洲核苷酸档案（ENA）API参考

## 概述
ENA是欧洲主要的核苷酸序列存储库，隶属于国际核苷酸序列数据库协作组织（INSDC），与NCBI GenBank和DDBJ并列。它存储原始测序读数、组装序列、基因组组装体及相关元数据。ENA提供五种互补API以满足不同访问需求。

## 1. ENA门户API（高级搜索）

### 基础URL
```
https://www.ebi.ac.uk/ena/portal/api
```

无需认证。所有端点均为公开访问。

### 关键端点

#### 搜索记录
```
GET /search?result={result_type}&query={query}&fields={fields}&limit={N}&format={format}
```

| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `result` | 字符串 | **必填**。搜索的数据类型。参见下方结果类型。 |
| `query` | 字符串 | 使用ENA查询语法的搜索语句。 |
| `fields` | 字符串 | 返回字段的逗号分隔列表。使用`/returnFields`查看各结果类型可用字段。 |
| `limit` | 整型 | 返回结果上限（默认100000）。 |
| `offset` | 整型 | 分页偏移量。 |
| `format` | 字符串 | `json`（默认）或`tsv`。 |

**查询语法示例：**
```
tax_id=9606 AND description="*hemoglobin*"
tax_id=9606 AND library_strategy="RNA-Seq"
accession="PRJEB40665"
scientific_name="Escherichia coli" AND dataclass="STD"
country="United Kingdom" AND first_public>2024-01-01
```

运算符：`=`, `!=`, `>`, `<`, `>=`, `<=`。逻辑符：`AND`, `OR`, `NOT`。通配符：`*`。含空格的数值需用双引号包裹。

**示例——搜索人类RNA-Seq测序运行：**
```
https://www.ebi.ac.uk/ena/portal/api/search?result=read_run&query=tax_id%3D9606%20AND%20library_strategy%3D%22RNA-Seq%22&fields=run_accession,experiment_accession,sample_accession,study_accession,instrument_platform,library_strategy,read_count,base_count&limit=5&format=json
```

**示例——按生物体搜索核苷酸序列：**
```
https://www.ebi.ac.uk/ena/portal/api/search?result=sequence&query=tax_id%3D9606%20AND%20description%3D%22*hemoglobin*%22&fields=accession,description,tax_id,scientific_name,base_count&limit=5&format=json
```

**响应示例：**
```json
[
  {
    "accession": "AA126503",
    "description": "zk94h05.s1 Soares_pregnant_uterus_NbHPU Homo sapiens cDNA clone ...",
    "tax_id": "9606"
  }
]
```

#### 记录计数
```
GET /count?result={result_type}&query={query}
```

返回纯整数计数。

**示例：**
```
https://www.ebi.ac.uk/ena/portal/api/count?result=read_run&query=tax_id%3D9606%20AND%20library_strategy%3D%22RNA-Seq%22
```

#### 列出可用结果类型
```
GET /results?format=json
```

#### 列出结果类型的可搜索字段
```
GET /searchFields?result={result_type}
```

#### 列出结果类型的可返回字段
```
GET /returnFields?result={result_type}
```

### 结果类型

| 结果类型 | 描述 |
|-------------|-------------|
| `sequence` | 核苷酸序列 |
| `coding` | 编码序列（CDS） |
| `noncoding` | 非编码序列 |
| `read_run` | 原始测序读数（运行） |
| `read_experiment` | 测序实验 |
| `read_study` | 原始读数相关研究 |
| `analysis` | 分析数据 |
| `analysis_study` | 分析相关研究 |
| `assembly` | 基因组组装 |
| `sample` | 样本 |
| `study` | 研究项目 |
| `taxon` | 分类学分类 |
| `wgs_set` | 全基因组鸟枪法组装重叠群集（WGS） |
| `tsa_set` | 转录组组装重叠群集（TSA） |
| `tls_set` | 靶向基因座研究重叠群集（TLS） |

---

## 2. ENA浏览器API（记录检索）

### 基础URL
```
https://www.ebi.ac.uk/ena/browser/api
```

通过登录号直接检索记录时使用此API。

### 关键端点

#### 以XML格式检索记录
```
GET /xml/{accession}
```

**示例：**
```
https://www.ebi.ac.uk/ena/browser/api/xml/PRJEB40665
https://www.ebi.ac.uk/ena/browser/api/xml/SRR12345678
https://www.ebi.ac.uk/ena/browser/api/xml/ERS1234567
```

#### 以EMBL平面文件格式检索记录
```
GET /embl/{accession}
```

**示例：**
```
https://www.ebi.ac.uk/ena/browser/api/embl/AY585947
```

支持`?lineLimit=N`参数截断长记录。

#### 以FASTA格式检索序列
```
GET /fasta/{accession}
```

**示例：**
```
https://www.ebi.ac.uk/ena/browser/api/fasta/AY585947
```

### 响应格式

| 端点 | 格式 | 使用场景 |
|----------|--------|----------|
| `/xml/{accession}` | XML | 研究、样本、实验、运行的完整结构化元数据 |
| `/embl/{accession}` | EMBL平面文件 | 带特征注释的序列 |
| `/fasta/{accession}` | FASTA | 原始核苷酸/蛋白质序列 |

### 登录号类型

| 前缀 | 实体 | 示例 |
|--------|--------|---------|
| `PRJEB` / `PRJNA` / `PRJDB` | 研究/项目 | `PRJEB40665` |
| `ERX` / `SRX` / `DRX` | 实验 | `ERX1234567` |
| `ERS` / `SRS` / `DRS` | 样本 | `ERS1234567` |
| `ERR` / `SRR` / `DRR` | 运行 | `ERR1234567` |
| `GCA` | 基因组组装 | `GCA_000001405.29` |
| 标准INSDC | 序列 | `AY585947`, `M10051` |

---

## 3. ENA分类学REST API

### 基础URL
```
https://www.ebi.ac.uk/ena/taxonomy/rest
```

### 关键端点

#### 通过分类ID查询
```
GET /tax-id/{taxId}
```

**示例：**
```
https://www.ebi.ac.uk/ena/taxonomy/rest/tax-id/9606
```

**响应示例：**
```json
{
  "taxId": 9606,
  "scientificName": "Homo sapiens",
  "commonName": "human",
  "formalName": true,
  "rank": "species",
  "division": "HUM",
  "lineage": "Eukaryota; Metazoa; Chordata; Craniata; Vertebrata; ...; Homo; ",
  "geneticCode": "1",
  "mitochondrialGeneticCode": "2",
  "submittable": true,
  "binomial": true,
  "metagenome": false,
  "otherNames": [
    {"nameClass": "authority", "name": "Linnaeus, 1758"},
    {"nameClass": "genbank common name", "name": "human"}
  ]
}
```

#### 按学名搜索
```
GET /scientific-name/{name}
```

**示例：**
```
https://www.ebi.ac.uk/ena/taxonomy/rest/scientific-name/Homo%20sapiens
```

返回匹配的分类记录数组。

#### 按通用名搜索
```
GET /any-name/{name}
```

**示例：**
```
https://www.ebi.ac.uk/ena/taxonomy/rest/any-name/human
```

#### 名称建议（自动补全）
```
GET /suggest-for-submission/{partialName}
```

**示例：**
```
https://www.ebi.ac.uk/ena/taxonomy/rest/suggest-for-submission/Homo%20sap
```

---

## 4. ENA交叉引用服务

### 基础URL
```
https://www.ebi.ac.uk/ena/xref/rest
```

检索ENA记录与外部数据库（UniProt、PDB、PubMed等）的关联。

### 关键端点

#### 按登录号搜索交叉引用
```
GET /json/search?accession={accession}
```

| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `accession` | 字符串 | 需查询交叉引用的ENA登录号。 |
| `source` | 字符串 | 按源数据库筛选（如`UniProtKB`）。 |
| `target` | 字符串 | 按目标类型筛选。 |
| `limit` | 整型 | 结果上限。 |
| `offset` | 整型 | 分页偏移量。 |

**示例：**
```
https://www.ebi.ac.uk/ena/xref/rest/json/search?accession=A00145
```

**响应示例：**
```json
[
  {
    "Source": "EuropePMC",
    "Source Primary Accession": "PMC12345",
    "Source Secondary Accession": "",
    "Source URL": "https://europepmc.org/...",
    "Source Secondary URL": "",
    "Target": "sequence",
    "Target Primary Accession": "A00145",
    "Target Secondary Accession": "",
    "Target URL": "https://www.ebi.ac.uk/ena/...",
    "Has Inferred": "N",
    "Inferred From": ""
  }
]
```

---

## 5. CRAM参考序列注册库

### 基础URL
```
https://www.ebi.ac.uk/ena/cram
```

检索CRAM文件压缩使用的参考序列。

### 关键端点

#### 通过MD5校验值查询
```
GET /md5/{md5}
```

**示例：**
```
https://www.ebi.ac.uk/ena/cram/md5/b1eba5b6e4440e22e1e02f7e0febd2da
```

#### 通过SHA1校验值查询
```
GET /sha1/{sha1}
```

返回FASTA格式的参考序列。

---

## 常用搜索模式

```
# 某物种的全部RNA-Seq运行
result=read_run&query=tax_id=9606 AND library_strategy="RNA-Seq"

# 某生物体的WGS组装
result=assembly&query=tax_id=562 AND assembly_type="primary metagenome"

# 按研究登录号搜索序列
result=sequence&query=study_accession="PRJEB40665"

# 特定国家采集日期后的样本
result=sample&query=country="Germany" AND collection_date>=2024-01-01

# 某基因关键词的编码序列
result=coding&query=description="*BRCA1*" AND tax_id=9606

# 统计可用数据集
/count?result=read_run&query=tax_id=9606

# 获取结果类型的元数据字段
/returnFields?result=read_run
/searchFields?result=read_run
```

## 速率限制

- 无需认证
- 无官方公布的速率限制，但请保持合理请求频率：避免超过约5个并发请求
- 大型结果集：使用`limit`和`offset`进行分页
- 批量下载序列数据（FASTQ等）时，请使用ENA的FTP/Aspera服务而非REST API

## 使用技巧

- **ENA与SRA关系**：ENA与NCBI SRA互为镜像（同属INSDC成员）。ENA登录号（ERR/ERX/ERS/PRJEB）与NCBI登录号（SRR/SRX/SRS/PRJNA）可交叉引用。根据查询需求选择API。
- **门户API用于搜索，浏览器API用于检索**：需跨记录筛选时使用门户API，已知登录号获取完整记录时使用浏览器API。
- **JSON与XML选择**：门户API返回JSON或TSV，浏览器API主要返回XML、EMBL或FASTA。
- **字段发现**：始终通过`/returnFields?result={type}`和`/searchFields?result={type}`查看各结果类型的可用字段——不同结果类型的字段存在差异。
- **交叉引用**：使用交叉引用服务查找ENA记录与UniProt、PDB、PubMed等数据库的关联。
