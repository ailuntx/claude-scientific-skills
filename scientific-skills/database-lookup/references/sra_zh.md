# SRA（序列读段存档）API参考

## 概述
测序运行元数据：实验、样本、研究和运行。可通过E-utilities访问，使用`db=sra`参数。返回描述测序实验、平台、文库策略和样本属性的XML元数据。

## 基础URL
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/
```

## 认证
- **API密钥**（推荐）：附加`&api_key=您的密钥`
- 无密钥：每秒3次请求。有密钥：每秒10次请求
- 需提供`tool`和`email`参数

## 核心端点

### 1. ESearch —— 搜索SRA记录
```
GET esearch.fcgi?db=sra&term=查询词&retmax=N&retmode=json
```

**示例 —— 搜索人类RNA-seq实验：**
```
GET esearch.fcgi?db=sra&term=RNA-seq[Strategy] AND Homo sapiens[Organism]&retmax=5&retmode=json
```
响应：
```json
{
  "esearchresult": {
    "count": "584231",
    "retmax": "5",
    "idlist": ["28574913", "28574912", "28574911", ...]
  }
}
```

**示例 —— 按登录号搜索：**
```
GET esearch.fcgi?db=sra&term=SRP123456[Accession] OR SRR123456[Accession]&retmode=json
```

### 2. EFetch —— 获取完整SRA元数据（仅XML格式）
```
GET efetch.fcgi?db=sra&id=标识符&rettype=full&retmode=xml
```

**示例 —— 获取SRA记录的元数据：**
```
GET efetch.fcgi?db=sra&id=28574913&rettype=full&retmode=xml
```
响应（简略XML）：
```xml
<EXPERIMENT_PACKAGE_SET>
  <EXPERIMENT_PACKAGE>
    <EXPERIMENT accession="SRX12345" alias="...">
      <TITLE>人类肝脏组织RNA测序</TITLE>
      <STUDY_REF accession="SRP12345"/>
      <DESIGN>
        <LIBRARY_DESCRIPTOR>
          <LIBRARY_STRATEGY>RNA-Seq</LIBRARY_STRATEGY>
          <LIBRARY_SOURCE>转录组</LIBRARY_SOURCE>
          <LIBRARY_SELECTION>cDNA</LIBRARY_SELECTION>
          <LIBRARY_LAYOUT><PAIRED/></LIBRARY_LAYOUT>
        </LIBRARY_DESCRIPTOR>
      </DESIGN>
      <PLATFORM>
        <ILLUMINA><INSTRUMENT_MODEL>Illumina NovaSeq 6000</INSTRUMENT_MODEL></ILLUMINA>
      </PLATFORM>
    </EXPERIMENT>
    <SUBMISSION accession="SRA12345" center_name="GEO"/>
    <Organization><Name>某研究所</Name></Organization>
    <STUDY accession="SRP12345">
      <DESCRIPTOR>
        <STUDY_TITLE>人类组织转录组分析</STUDY_TITLE>
        <STUDY_TYPE existing_study_type="转录组分析"/>
      </DESCRIPTOR>
    </STUDY>
    <SAMPLE accession="SRS12345">
      <TITLE>人类肝脏RNA</TITLE>
      <SAMPLE_ATTRIBUTES>
        <SAMPLE_ATTRIBUTE><TAG>组织</TAG><VALUE>肝脏</VALUE></SAMPLE_ATTRIBUTE>
        <SAMPLE_ATTRIBUTE><TAG>细胞类型</TAG><VALUE>肝细胞</VALUE></SAMPLE_ATTRIBUTE>
      </SAMPLE_ATTRIBUTES>
    </SAMPLE>
    <RUN_SET>
      <RUN accession="SRR12345" total_spots="45000000" total_bases="9000000000">
        <Statistics nreads="2">
          <Read average="150" count="45000000"/>
        </Statistics>
      </RUN>
    </RUN_SET>
  </EXPERIMENT_PACKAGE>
</EXPERIMENT_PACKAGE_SET>
```

### 3. ESummary —— 获取SRA摘要信息
```
GET esummary.fcgi?db=sra&id=标识符&retmode=json
```
返回：实验标题、平台、总运行数/读段数/碱基数、创建日期、研究/样本登录号，以XML字符串形式存储在`expxml`和`runs`字段中

### 4. ELink —— 跨NCBI数据库链接
```
GET elink.fcgi?dbfrom=sra&db=biosample&id=SRA_UID
GET elink.fcgi?dbfrom=sra&db=gds&id=SRA_UID
```

## SRA登录号类型
| 前缀 | 实体 |
|--------|--------|
| `SRP` / `ERP` / `DRP` | 研究 |
| `SRX` / `ERX` / `DRX` | 实验 |
| `SRS` / `ERS` / `DRS` | 样本 |
| `SRR` / `ERR` / `DRR` | 运行 |
| `SRA` | 提交 |

## 常用搜索模式
```
# 按生物体和策略
term=Mus musculus[Organism] AND WGS[Strategy]

# 按平台
term=Illumina[Platform] AND ATAC-seq[Strategy] AND human[Organism]

# 按研究登录号
term=SRP123456[Accession]

# 按BioProject
term=PRJNA123456[BioProject]

# 按日期范围
term=("2024/01/01"[Publication Date] : "2024/12/31"[Publication Date])

# 按文库来源
term=基因组[Source] AND ChIP-Seq[Strategy] AND 癌症[Text Word]

# 按读段数范围
term=10000000:100000000[ReadLength]

# 组合复杂查询
term=(RNA-Seq[Strategy] AND 双端[Layout] AND Homo sapiens[Organism] AND Illumina[Platform])
```

## 速率限制
- 无API密钥：每秒3次请求
- 有API密钥：每秒10次请求
- 批量元数据获取：使用`usehistory=y`配合`WebEnv`/`query_key`参数，分批次获取
- 实际序列数据（FASTQ）无法通过E-utilities获取；请使用SRA工具包（`fastq-dump`/`fasterq-dump`）或XML中提供的SRA云URL

## 故障排除
- **HTTP 429**：请求过多。请降低请求频率
- **HTTP 414**：URI过长。长查询请改用POST方法
- **HTTP 400**：错误请求。请检查参数
- **空结果**：尝试简化查询或检查拼写
- **XML解析错误**：确保已设置`retmode=xml`参数
