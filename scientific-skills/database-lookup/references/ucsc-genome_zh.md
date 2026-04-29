# UCSC 基因组浏览器 REST API 参考

## 概述
提供对 UCSC 基因组浏览器数据库中基因组注释、基因轨道、序列数据及其他资源的程序化访问。

## 基础 URL
`https://api.genome.ucsc.edu`

## 认证
无需认证（公开访问）。

## 响应格式
所有接口均返回 JSON 格式数据。

## 关键接口

### 获取可用基因组列表
```
GET /list/ucscGenomes
```
返回所有基因组组装版本（如 hg38, mm39 等）及其描述信息。

### 获取基因组轨道列表
```
GET /list/tracks?genome=hg38
```
返回指定组装版本的所有注释轨道。

### 获取染色体/重叠群列表
```
GET /list/chromosomes?genome=hg38
```
可选参数：添加 `&track=<轨道名称>` 可限定为包含该轨道数据的染色体。

### 获取轨道内数据表
```
GET /list/schema?genome=hg38&track=knownGene
```
返回表结构信息，包含字段名称、类型及 SQL 建表语句。

### 获取轨道数据（注释）
```
GET /getData/track?genome=hg38&track=knownGene&chrom=chr1&start=11873&end=14409
```
参数说明：
- `genome` -- 组装版本名称（必需）
- `track` -- 轨道名称（必需）
- `chrom` -- 染色体（可选，限定单条染色体）
- `start`, `end` -- 0基半开区间坐标（可选，需配合 chrom 使用）
- `maxItemsOutput` -- 返回条目数上限（部分轨道默认 1000 条）

### 获取序列
```
GET /getData/sequence?genome=hg38&chrom=chr1&start=11873&end=11893
```
返回指定区域的 DNA 序列。坐标采用 0基半开区间。

### 关键词搜索
```
GET /search?search=BRCA1&genome=hg38
```
返回跨轨道匹配位置（基因名、序列号等）。

### 获取中心基因组数据
```
GET /list/hubGenomes?hubUrl=<hubURL>
```
列出轨道中心内可用的基因组。

## 调用示例

### 获取区域内的 RefSeq 基因注释
```
GET https://api.genome.ucsc.edu/getData/track?genome=hg38&track=ncbiRefSeq&chrom=chr17&start=43044295&end=43125483
```

### 获取 DNA 序列
```
GET https://api.genome.ucsc.edu/getData/sequence?genome=hg38&chrom=chr7&start=117119148&end=117119178
```

### 响应示例（序列）
```json
{
  "genome": "hg38",
  "chrom": "chr7",
  "start": 117119148,
  "end": 117119178,
  "dna": "atgcagatatcagcgatgcagatcgatcg..."
}
```

### 响应示例（轨道数据）
```json
{
  "genome": "hg38",
  "track": "ncbiRefSeq",
  "chrom": "chr17",
  "start": 43044295,
  "end": 43125483,
  "ncbiRefSeq": [
    {
      "chrom": "chr17",
      "chromStart": 43044295,
      "chromEnd": 43125483,
      "name": "NM_007294.4",
      "strand": "-",
      "name2": "BRCA1",
      "exonCount": 23,
      "exonStarts": "43044295,43047642,...",
      "exonEnds": "43045802,43047703,..."
    }
  ]
}
```

## 坐标系统
所有坐标均采用 **0基半开区间**（标准 BED 格式），即 `start` 包含而 `end` 不包含。

## 速率限制
- 未公布硬性速率限制
- API 适用于中等规模程序化调用；批量下载应使用 MySQL 公共服务器（genome-mysql.soe.ucsc.edu）或 BigBed/BigWig 文件下载
- 返回超大结果集的请求可能通过 `maxItemsOutput` 截断

## 常用基因组标识
- `hg38` -- 人类 GRCh38（当前版本）
- `hg19` -- 人类 GRCh37
- `mm39` -- 小鼠 GRCm39
- `mm10` -- 小鼠 GRCm38
- `dm6` -- 果蝇
- `danRer11` -- 斑马鱼
- `sacCer3` -- 酵母
