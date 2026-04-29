# UniProt REST API

## 基础 URL

```
https://rest.uniprot.org
```

## 认证

无需 API 密钥。所有端点均为公开访问。

## 核心端点

### 1. 蛋白质检索

```
GET /uniprotkb/search
```

**参数：**

| 参数名     | 类型   | 描述 |
|-----------|--------|-------------|
| `query`   | string | **必填**。使用 UniProt 查询语法（字段:值 对，布尔运算符） |
| `format`  | string | `json`（默认）、`tsv`、`fasta`、`xml`、`list`、`xlsx`、`obo` |
| `fields`  | string | 返回字段列表（逗号分隔）。关键字段：`accession`、`id`、`protein_name`、`gene_names`、`organism_name`、`organism_id`、`length`、`sequence`、`cc_function`、`go_id`、`go`、`xref_pdb`、`reviewed`、`ec`、`cc_subcellular_location`、`ft_domain`、`lineage` |
| `size`    | int    | 每页结果数（最大 500，默认 25） |
| `cursor`  | string | 分页游标（通过响应头 `Link` 返回） |
| `sort`    | string | 排序字段及方向，如 `gene asc`、`length desc`、`annotation_score desc` |

**调用示例：**

检索已审核的人类 TP53：
```
https://rest.uniprot.org/uniprotkb/search?query=(gene:TP53) AND (organism_id:9606) AND (reviewed:true)&format=json&fields=accession,protein_name,gene_names,organism_name,length,cc_function&size=10
```

通过蛋白质名称关键词检索：
```
https://rest.uniprot.org/uniprotkb/search?query=(protein_name:insulin) AND (reviewed:true)&format=json&size=5
```

通过 EC 编号（酶分类）检索：
```
https://rest.uniprot.org/uniprotkb/search?query=(ec:2.7.11.1) AND (organism_id:9606)&format=json&size=25
```

通过基因本体论检索：
```
https://rest.uniprot.org/uniprotkb/search?query=(go:0006915) AND (organism_id:9606) AND (reviewed:true)&format=json&size=25
```

**响应（JSON）：**
```json
{
  "results": [
    {
      "entryType": "UniProtKB 已审核（Swiss-Prot）",
      "primaryAccession": "P04637",
      "uniProtkbId": "P53_HUMAN",
      "organism": {
        "scientificName": "Homo sapiens",
        "taxonId": 9606
      },
      "proteinDescription": {
        "recommendedName": {
          "fullName": { "value": "细胞肿瘤抗原 p53" }
        }
      },
      "genes": [
        {
          "geneName": { "value": "TP53" },
          "synonyms": [{ "value": "P53" }]
        }
      ],
      "sequence": {
        "value": "MEEPQSDP...",
        "length": 393,
        "molWeight": 43653,
        "crc64": "..."
      },
      "comments": [...],
      "features": [...],
      "references": [...]
    }
  ]
}
```

**分页机制：** 响应头 `Link` 包含带游标参数的下一页 URL。通过该 URL 获取后续页面。

---

### 2. 按编号获取单条记录

```
GET /uniprotkb/{accession}
```

**参数：**

| 参数名    | 类型   | 描述 |
|-----------|--------|-------------|
| `format`  | string | `json`、`tsv`、`fasta`、`xml`、`gff` |

**调用示例：**

```
https://rest.uniprot.org/uniprotkb/P04637?format=json
https://rest.uniprot.org/uniprotkb/P04637.fasta
```

---

### 3. FASTA 序列获取

在编号后追加 `.fasta` 或使用 `format=fasta`：

```
https://rest.uniprot.org/uniprotkb/P04637.fasta
```

批量获取 FASTA（通过检索）：
```
https://rest.uniprot.org/uniprotkb/search?query=(gene:BRCA1) AND (organism_id:9606) AND (reviewed:true)&format=fasta
```

---

### 4. ID 映射（跨标识符类型转换）

ID 映射为异步两步流程。

**步骤 1：提交任务**
```
POST /idmapping/run
Content-Type: application/x-www-form-urlencoded

from={dbFrom}&to={dbTo}&ids={comma-separated-ids}
```

常用数据库名 `from`/`to`：
- `UniProtKB_AC-ID`（UniProt 编号）
- `Gene_Name`
- `GeneID`（NCBI Gene / Entrez Gene）
- `Ensembl`、`Ensembl_Genomes`
- `RefSeq_Protein`
- `PDB`
- `ChEMBL`
- `EMBL-GenBank-DDBJ`
- `STRING`

返回：
```json
{ "jobId": "abc123def456" }
```

**步骤 2：轮询并获取结果**
```
GET /idmapping/status/{jobId}
```
完成后重定向至：
```
GET /idmapping/results/{jobId}?format=json&size=500
```

**示例：**

将 Ensembl 基因 ID 映射至 UniProt 编号：
```
POST /idmapping/run
from=Ensembl&to=UniProtKB_AC-ID&ids=ENSG00000141510,ENSG00000012048
```

将 UniProt 映射至 PDB：
```
POST /idmapping/run
from=UniProtKB_AC-ID&to=PDB&ids=P04637,P38398
```

**响应（结果）：**
```json
{
  "results": [
    {
      "from": "ENSG00000141510",
      "to": {
        "primaryAccession": "P04637",
        "uniProtkbId": "P53_HUMAN",
        ...
      }
    }
  ]
}
```

---

### 5. UniRef（序列聚类）

```
GET /uniref/search?query={query}&format=json
GET /uniref/{id}
```

聚类 ID：`UniRef100_P04637`、`UniRef90_P04637`、`UniRef50_P04637`

---

### 6. UniParc（序列归档）

```
GET /uniparc/search?query={query}&format=json
GET /uniparc/{upi}
```

---

### 7. 蛋白质组

```
GET /proteomes/search?query=(organism_id:9606)&format=json
GET /proteomes/{upid}
```

示例——人类参考蛋白质组：
```
https://rest.uniprot.org/proteomes/UP000005640?format=json
```

---

### 8. 分类学

```
GET /taxonomy/search?query={query}&format=json
GET /taxonomy/{taxonId}
```

---

## 查询语法

UniProt 检索查询支持字段:值语法及布尔运算符：

- `(gene:TP53)` -- 基因名称
- `(organism_id:9606)` -- NCBI 分类学 ID（9606=人类，10090=小鼠）
- `(organism_name:"Homo sapiens")` -- 生物体名称
- `(reviewed:true)` -- 仅 Swiss-Prot（人工审核）
- `(protein_name:kinase)` -- 蛋白质名称含关键词
- `(ec:2.7.11.1)` -- 酶分类编号
- `(go:0006915)` -- 基因本体术语 ID
- `(xref:pdb-P04637)` -- 交叉引用
- `(length:[100 TO 300])` -- 序列长度范围
- `(cc_disease:cancer)` -- 疾病关联
- `(ft_domain:SH2)` -- 结构域注释
- `(cc_subcellular_location:nucleus)` -- 亚细胞定位
- `(date_modified:[2024-01-01 TO *])` -- 修改日期

通过 `AND`、`OR`、`NOT` 组合：
```
(gene:BRCA1) AND (organism_id:9606) AND (reviewed:true)
```

## 速率限制

- 无官方硬性限制，但过量请求将被限流
- 使用分页参数（`size` + `cursor`）分批获取结果
- 批量提交 ID 映射任务替代单次查询
- 大型下载请使用流式端点或 FTP 站点
- 收到 HTTP 429 时遵守 `Retry-After` 响应头

## 错误格式

```json
{
  "url": "https://rest.uniprot.org/...",
  "messages": ["错误信息"]
}
```
HTTP 400 表示无效查询，404 表示未找到，429 表示速率限制，500 表示服务器错误。
