# NCBI 分类学 API 参考

## 概述
提供 NCBI 数据库中所有生物的分类学数据（名称、谱系、分类等级）。可通过 E-utilities 使用 `db=taxonomy` 参数访问。

## 基础 URL
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/
```

## 认证
- **API 密钥**（推荐）：附加 `&api_key=您的密钥`（在 ncbi.nlm.nih.gov/account 注册）
- 无密钥：每秒 3 次请求；有密钥：每秒 10 次请求
- 需提供 `tool` 和 `email` 参数

## 核心端点

### 1. ESearch —— 按名称搜索分类
```
GET esearch.fcgi?db=taxonomy&term=QUERY&retmode=json
```
| 参数 | 说明 |
|-------|-------------|
| `term` | 生物名称、俗名或分类标识符。支持字段：`[Scientific Name]`, `[Common Name]`, `[All Names]`, `[Rank]` |
| `retmax` | 返回的最大 ID 数（默认 20） |

**示例 —— 按学名搜索：**
```
GET esearch.fcgi?db=taxonomy&term=Homo+sapiens[Scientific Name]&retmode=json
```
响应：
```json
{
  "esearchresult": {
    "count": "1",
    "idlist": ["9606"]
  }
}
```

**示例 —— 按俗名搜索：**
```
GET esearch.fcgi?db=taxonomy&term=dog[Common Name]&retmode=json
```

### 2. EFetch —— 获取完整分类记录
```
GET efetch.fcgi?db=taxonomy&id=TAXIDS&retmode=xml
```
注：分类学 EFetch 仅支持 XML 输出格式

**示例 —— 获取人类分类数据（分类标识符 9606）：**
```
GET efetch.fcgi?db=taxonomy&id=9606&retmode=xml
```
响应（简化 XML）：
```xml
<TaxaSet>
  <Taxon>
    <TaxId>9606</TaxId>
    <ScientificName>Homo sapiens</ScientificName>
    <OtherNames>
      <CommonName>human</CommonName>
    </OtherNames>
    <Rank>species</Rank>
    <Division>Primates</Division>
    <GeneticCode><GCId>1</GCId><GCName>Standard</GCName></GeneticCode>
    <MitoGeneticCode><MGCId>2</MGCId><MGCName>Vertebrate Mitochondrial</MGCName></MitoGeneticCode>
    <Lineage>cellular organisms; Eukaryota; Opisthokonta; Metazoa; ... ; Hominidae; Homo</Lineage>
    <LineageEx>
      <Taxon><TaxId>131567</TaxId><ScientificName>cellular organisms</ScientificName><Rank>no rank</Rank></Taxon>
      <Taxon><TaxId>2759</TaxId><ScientificName>Eukaryota</ScientificName><Rank>superkingdom</Rank></Taxon>
      <!-- ... 每个祖先节点 ... -->
    </LineageEx>
  </Taxon>
</TaxaSet>
```

### 3. ESummary —— 获取分类摘要
```
GET esummary.fcgi?db=taxonomy&id=TAXIDS&retmode=json
```
**示例 —— 多分类单元查询：**
```
GET esummary.fcgi?db=taxonomy&id=9606,10090,7227&retmode=json
```
响应包含：`ScientificName`, `CommonName`, `Rank`, `Division`, `TaxId`, `Genus`, `Species`

### 4. ELink —— 跨数据库链接分类信息
```
GET elink.fcgi?dbfrom=taxonomy&db=protein&id=9606&term=insulin
```
查找指定分类标识符对应的所有蛋白质记录，支持关键词过滤

## 常用搜索模式
```
# 获取属下的所有物种
term=Drosophila[Next Level] AND species[Rank]

# 直接按分类标识符搜索
term=txid9606[Organism:exp]

# 按分类等级搜索
term=Mammalia[Scientific Name] AND class[Rank]

# 子树搜索（所有后代）
term=txid9606[Organism:exp]
```

## 实用交叉引用
| 链接 | 说明 |
|------|-------------|
| `taxonomy_protein` | 某分类单元的所有蛋白质 |
| `taxonomy_gene` | 某分类单元的所有基因 |
| `taxonomy_nuccore` | 某分类单元的所有核苷酸记录 |
| `taxonomy_genome` | 某分类单元的基因组组装 |

## 速率限制
- 无 API 密钥：每秒 3 次请求
- 有 API 密钥：每秒 10 次请求
- EFetch 支持单次调用多个分类标识符（逗号分隔）
