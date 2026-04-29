# Ensembl REST API

## 基础 URL

```
https://rest.ensembl.org
```

适用于 Ensembl Genomes（植物、真菌、细菌、原生生物、后生动物）：
```
https://rest.ensembl.org
```
（相同基础 URL；Ensembl Genomes 已合并至主 REST API）

适用于 GRCh37 (hg19) 归档版本：
```
https://grch37.rest.ensembl.org
```

## 认证

无需 API 密钥。所有端点均为公开访问。

## 通用请求头

所有请求应包含：
```
Content-Type: application/json
```

该 API 使用内容协商机制。对于 GET 请求，可在 URL 后附加 `?content-type=application/json` 或设置 `Accept` 请求头。

## 核心端点

### 1. 通过基因符号查询基因

```
GET /lookup/symbol/{species}/{symbol}?content-type=application/json
```

| 参数       | 类型     | 说明 |
|------------|--------|-------------|
| `species`   | 字符串 | **必填**。物种名称（如 `homo_sapiens`, `mus_musculus`） |
| `symbol`    | 字符串 | **必填**。基因符号（如 `TP53`, `BRCA1`） |
| `expand`    | 整数    | 设为 `1` 可包含转录本、翻译产物、外显子信息 |

**示例：**
```
https://rest.ensembl.org/lookup/symbol/homo_sapiens/TP53?content-type=application/json
https://rest.ensembl.org/lookup/symbol/homo_sapiens/BRCA1?content-type=application/json;expand=1
```

**响应：**
```json
{
  "id": "ENSG00000141510",
  "display_name": "TP53",
  "description": "肿瘤蛋白 p53 [来源:HGNC Symbol;编号:HGNC:11998]",
  "species": "homo_sapiens",
  "object_type": "Gene",
  "biotype": "protein_coding",
  "assembly_name": "GRCh38",
  "seq_region_name": "17",
  "start": 7661779,
  "end": 7687538,
  "strand": -1,
  "source": "ensembl_havana",
  "logic_name": "ensembl_havana_gene_homo_sapiens",
  "version": 16,
  "Transcript": [...]
}
```

---

### 2. 通过 Ensembl ID 查询基因/特征

```
GET /lookup/id/{id}?content-type=application/json
```

| 参数     | 类型     | 说明 |
|-----------|--------|-------------|
| `id`      | 字符串 | **必填**。Ensembl 稳定 ID（基因、转录本、蛋白质、外显子） |
| `expand`  | 整数    | 设为 `1` 可包含子对象（如基因的转录本等） |
| `db_type` | 字符串 | 数据库类型：`core`, `otherfeatures`, `cdna`, `rnaseq` |

**示例：**
```
https://rest.ensembl.org/lookup/id/ENSG00000141510?content-type=application/json;expand=1
https://rest.ensembl.org/lookup/id/ENST00000269305?content-type=application/json
https://rest.ensembl.org/lookup/id/ENSP00000269305?content-type=application/json
```

---

### 3. 批量查询（POST，最多 1000 个 ID）

```
POST /lookup/id
Content-Type: application/json

{ "ids": ["ENSG00000141510", "ENSG00000012048", "ENSG00000157764"] }
```

**响应示例：** 返回按 ID 索引的映射
```json
{
  "ENSG00000141510": {
    "id": "ENSG00000141510",
    "display_name": "TP53",
    ...
  },
  "ENSG00000012048": { ... }
}
```

---

### 4. 序列获取

```
GET /sequence/id/{id}?content-type=application/json
```

| 参数           | 类型     | 说明 |
|--------------|--------|-------------|
| `id`          | 字符串 | Ensembl 稳定 ID（基因、转录本或蛋白质） |
| `type`        | 字符串 | `genomic`（基因组）, `cdna`（cDNA）, `cds`（编码序列）, `protein`（蛋白质）。默认值因对象类型而异 |
| `format`      | 字符串 | `json` 或 `fasta` |
| `expand_3prime` | 整数  | 向 3' 端扩展 N 个碱基 |
| `expand_5prime` | 整数  | 向 5' 端扩展 N 个碱基 |
| `mask`        | 字符串 | `soft`（小写标记重复序列）或 `hard`（N 屏蔽重复序列） |

**示例：**

蛋白质序列：
```
https://rest.ensembl.org/sequence/id/ENSP00000269305?content-type=application/json
```

编码序列（CDS）：
```
https://rest.ensembl.org/sequence/id/ENST00000269305?type=cds&content-type=application/json
```

含侧翼序列的基因组序列：
```
https://rest.ensembl.org/sequence/id/ENSG00000141510?type=genomic&expand_5prime=1000&expand_3prime=500&content-type=application/json
```

FASTA 格式：
```
https://rest.ensembl.org/sequence/id/ENSP00000269305?content-type=text/x-fasta
```

**响应（JSON）：**
```json
{
  "id": "ENSP00000269305",
  "seq": "MEEPQSDPSVEPPLSQETFSDL...",
  "molecule": "protein",
  "desc": "染色体:GRCh38:17:7661779:7687538:-1"
}
```

---

### 5. 通过区域获取序列

```
GET /sequence/region/{species}/{region}?content-type=application/json
```

区域格式：`染色体:起始..终止` 或 `染色体:起始..终止:链方向`

**示例：**
```
https://rest.ensembl.org/sequence/region/homo_sapiens/17:7661779..7662000:1?content-type=application/json
```

---

### 6. 变异注释（VEP - 变异效应预测器）

**通过 HGVS 命名法：**
```
GET /vep/{species}/hgvs/{hgvs_notation}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/vep/homo_sapiens/hgvs/ENST00000269305.9:c.817C>T?content-type=application/json
https://rest.ensembl.org/vep/homo_sapiens/hgvs/17:g.7674220G>A?content-type=application/json
```

**通过基因组区域：**
```
GET /vep/{species}/region/{region}/{allele}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/vep/homo_sapiens/region/17:7674220-7674220:1/A?content-type=application/json
```

**通过 rsID：**
```
GET /vep/{species}/id/{rsid}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/vep/homo_sapiens/id/rs699?content-type=application/json
```

**VEP 响应：**
```json
[
  {
    "input": "17:g.7674220G>A",
    "assembly_name": "GRCh38",
    "seq_region_name": "17",
    "start": 7674220,
    "end": 7674220,
    "strand": 1,
    "allele_string": "G/A",
    "most_severe_consequence": "错义变异",
    "transcript_consequences": [
      {
        "gene_id": "ENSG00000141510",
        "gene_symbol": "TP53",
        "transcript_id": "ENST00000269305",
        "biotype": "protein_coding",
        "consequence_terms": ["错义变异"],
        "impact": "中度",
        "amino_acids": "R/H",
        "codons": "cGc/cAc",
        "protein_start": 248,
        "polyphen_prediction": "可能有害",
        "polyphen_score": 1.0,
        "sift_prediction": "有害",
        "sift_score": 0.0,
        "cadd_phred": 35.0
      }
    ],
    "colocated_variants": [
      {
        "id": "rs28934578",
        "frequencies": { ... },
        "clin_sig": ["致病性"]
      }
    ]
  }
]
```

**批量 VEP（POST，最多 200 个变异）：**
```
POST /vep/homo_sapiens/region
Content-Type: application/json

{ "variants": ["17 7674220 7674220 G/A 1", "7 140753336 140753336 A/T 1"] }
```

---

### 7. 变异（通过 rsID 查询已知变异）

```
GET /variation/{species}/{rsid}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/variation/homo_sapiens/rs699?content-type=application/json
```

**响应：**
```json
{
  "name": "rs699",
  "source": "从 dbSNP 导入的变异（包括 SNP 和 indel）",
  "mappings": [
    {
      "seq_region_name": "1",
      "start": 230710048,
      "end": 230710048,
      "strand": 1,
      "allele_string": "A/G",
      "assembly_name": "GRCh38",
      "location": "1:230710048-230710048"
    }
  ],
  "MAF": 0.35,
  "minor_allele": "G",
  "clinical_significance": [],
  "synonyms": [],
  "ancestral_allele": "A"
}
```

---

### 8. 区域重叠/特征查询

```
GET /overlap/region/{species}/{region}?feature={type}&content-type=application/json
```

| 参数     | 类型     | 说明 |
|-----------|--------|-------------|
| `region`  | 字符串 | 格式：`染色体:起始-终止` |
| `feature` | 字符串 | 可单选或多选：`gene`, `transcript`, `cds`, `exon`, `repeat`, `simple`, `misc`, `variation`, `somatic_variation`, `structural_variation`, `regulatory`, `motif`, `chipseq`, `constrained`。可重复参数查询多类型 |

**示例 - 获取区域内所有基因：**
```
https://rest.ensembl.org/overlap/region/homo_sapiens/17:7660000-7690000?feature=gene&content-type=application/json
```

**示例 - 获取调控特征：**
```
https://rest.ensembl.org/overlap/region/homo_sapiens/17:7660000-7690000?feature=regulatory&content-type=application/json
```

---

### 9. 交叉引用（Xrefs）

```
GET /xrefs/id/{id}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/xrefs/id/ENSG00000141510?content-type=application/json
```

返回外部数据库链接（HGNC、UniProt、NCBI Gene、RefSeq 等）。

**响应：**
```json
[
  {
    "primary_id": "11998",
    "display_id": "TP53",
    "dbname": "HGNC",
    "db_display_name": "HGNC 符号"
  },
  {
    "primary_id": "P04637",
    "display_id": "P53_HUMAN",
    "dbname": "Uniprot/SWISSPROT"
  },
  {
    "primary_id": "7157",
    "display_id": "TP53",
    "dbname": "EntrezGene"
  }
]
```

**通过符号查询交叉引用：**
```
GET /xrefs/symbol/{species}/{symbol}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/xrefs/symbol/homo_sapiens/TP53?content-type=application/json
```

---

### 10. 比较基因组学 - 同源关系

```
GET /homology/id/{id}?content-type=application/json
```

| 参数             | 类型     | 说明 |
|--------------|--------|-------------|
| `id`          | 字符串 | Ensembl 基因 ID |
| `type`        | 字符串 | `orthologues`（直系同源）, `paralogues`（旁系同源）, `projections`, `all` |
| `target_species` | 字符串 | 筛选特定物种（如 `mus_musculus`） |
| `target_taxon`   | 整数    | 筛选 NCBI 分类 ID |
| `sequence`    | 字符串 | `none`, `cdna`, `protein`。包含比对序列 |

**示例 - 获取人类 TP53 的小鼠直系同源基因：**
```
https://rest.ensembl.org/homology/id/ENSG00000141510?type=orthologues&target_species=mus_musculus&content-type=application/json
```

**响应：**
```json
{
  "data": [
    {
      "id": "ENSG00000141510",
      "homologies": [
        {
          "type": "ortholog_one2one",
          "target": {
            "id": "ENSMUSG00000059552",
            "species": "mus_musculus",
            "protein_id": "ENSMUSP00000073359",
            "perc_id": 77.8,
            "perc_pos": 86.0
          },
          "source": {
            "id": "ENSG00000141510",
            "species": "homo_sapiens",
            "protein_id": "ENSP00000269305"
          },
          "method_link_type": "ENSEMBL_ORTHOLOGUES",
          "dn_ds": 0.15
        }
      ]
    }
  ]
}
```

**通过符号查询同源关系：**
```
GET /homology/symbol/{species}/{symbol}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/homology/symbol/homo_sapiens/TP53?type=orthologues&target_species=mus_musculus&content-type=application/json
```

---

### 11. 调控特征

```
GET /regulatory/species/{species}/id/{id}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/regulatory/species/homo_sapiens/id/ENSR00000000163?content-type=application/json
```

---

### 12. 物种信息

```
GET /info/species?content-type=application/json
```

返回所有可用物种及其组装信息。

---

### 13. 组装信息

```
GET /info/assembly/{species}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/info/assembly/homo_sapiens?content-type=application/json
```

返回染色体名称、长度、组装名称（GRCh38）、坐标系统等信息。

---

### 14. 基因相关表型

```
GET /phenotype/gene/{species}/{gene}?content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/phenotype/gene/homo_sapiens/TP53?content-type=application/json
```

---

### 15. LD（连锁不平衡）

```
GET /ld/{species}/pairwise/{rsid1}/{rsid2}?population_name={pop}&content-type=application/json
```

**示例：**
```
https://rest.ensembl.org/ld/homo_sapiens/pairwise/rs699/rs4762?population_name=1000GENOMES:phase_3:CEU&content-type=application/json

| 狗 | `canis_lupus_familiaris` |
| 猪 | `sus_scrofa` |

## 速率限制

- 普通用户（无API密钥）**每秒15个请求**。
- 注册API密钥（可选）可获得更高限制。
- 超出限制的请求将收到HTTP 429响应及`Retry-After`头部。
- 批量端点（POST）按单个请求计数——可减少调用次数。
- `/lookup/id`批量POST最多支持1000个ID。
- VEP批量POST最多支持200个变异体。
- 返回速率限制头部：`X-RateLimit-Limit`、`X-RateLimit-Remaining`、`X-RateLimit-Reset`。

## 错误格式

```json
{
  "error": "未找到ID 'ENSG999'"
}
```
HTTP 400表示错误请求，404表示未找到，429表示速率限制，503表示服务不可用。

## 使用技巧

- GET请求需附加`?content-type=application/json`（或设置Accept头部）——默认返回XML/HTML格式。
- 如需hg19坐标，请使用GRCh37基础URL（`grch37.rest.ensembl.org`）。
- `/lookup/symbol`端点是基因符号转Ensembl ID的最快途径。
- VEP服务中：单变异体推荐HGVS端点；批量处理推荐region POST端点。
- 结合基因ID使用`/xrefs/id`，可交叉引用UniProt、NCBI Gene、HGNC等数据库。
