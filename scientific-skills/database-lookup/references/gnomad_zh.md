# gnomAD（基因组聚合数据库）API参考

## 概述
gnomAD通过聚合外显子组和基因组测序数据，提供跨不同人群的等位基因频率和变异注释。

## API类型：GraphQL
- **端点**：`https://gnomad.broadinstitute.org/api`
- **方法**：POST请求附带包含GraphQL查询的JSON主体
- **认证**：无需认证（公开访问）
- **响应格式**：JSON（包含GraphQL结构的`data`包装器）

## 关键查询

### 通过变异ID查询变异
变异ID格式：`{染色体}-{位置}-{参考碱基}-{替代碱基}`（GRCh37或GRCh38）。

```
POST https://gnomad.broadinstitute.org/api
Content-Type: application/json

{
  "query": "{ variant(variantId: \"1-55516888-G-A\", dataset: gnomad_r4) { variant_id rsids chrom pos ref alt exome { ac an af } genome { ac an af } } }"
}
```

### 基因查询
```json
{
  "query": "{ gene(gene_symbol: \"BRCA1\", reference_genome: GRCh38) { gene_id symbol chrom start stop strand } }"
}
```

### 基因内变异查询
```json
{
  "query": "{ gene(gene_symbol: \"PCSK9\", reference_genome: GRCh38) { variants(dataset: gnomad_r4) { variant_id consequence rsids exome { ac an af } genome { ac an af } } } }"
}
```

### 区域内变异查询
```json
{
  "query": "{ region(chrom: \"1\", start: 55505222, stop: 55530526, reference_genome: GRCh38) { variants(dataset: gnomad_r4) { variant_id rsids consequence exome { ac af } genome { ac af } } } }"
}
```

### 转录本查询
```json
{
  "query": "{ transcript(transcript_id: \"ENST00000357654\", reference_genome: GRCh38) { transcript_id gene_id chrom start stop strand } }"
}
```

## 数据集取值
- `gnomad_r4` -- gnomAD v4（GRCh38，最新主版本）
- `gnomad_r3` -- gnomAD v3.1.2（GRCh38，仅基因组）
- `gnomad_r2_1` -- gnomAD v2.1.1（GRCh37，外显子组+基因组）

## 人群频率字段
在`exome`或`genome`对象中，可通过`populations { id ac an af }`获取人群特异性频率，其中`id`取值包括：`afr`, `amr`, `asj`, `eas`, `fin`, `mid`, `nfe`, `oth`, `sas`。

## 响应示例（变异）
```json
{
  "data": {
    "variant": {
      "variant_id": "1-55516888-G-A",
      "rsids": ["rs11591147"],
      "chrom": "1",
      "pos": 55516888,
      "ref": "G",
      "alt": "A",
      "exome": { "ac": 1234, "an": 250000, "af": 0.004936 },
      "genome": { "ac": 456, "an": 150000, "af": 0.00304 }
    }
  }
}
```

## 速率限制
- 未公布明确速率限制，但高频查询会被限流
- 建议采用合理请求间隔（约1次/秒）
- 批量下载请使用gnomAD在Google Cloud的Hail表或直接下载VCF文件

## 注意事项
- GraphQL模式不单独进行版本控制，其与gnomAD网站界面保持同步
- 可通过浏览器网络检查器访问gnomad.broadinstitute.org以发现更多查询字段和结构
- 结构变异（SV）使用独立查询结构（`structural_variant`）
- 约束性指标（pLI, LOEUF）可通过基因查询中的`gnomad_constraint`字段获取
