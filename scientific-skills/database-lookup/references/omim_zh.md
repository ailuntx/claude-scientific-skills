# OMIM（在线人类孟德尔遗传）API 参考文档

## 基础 URL
```
https://api.omim.org/api
```

## 认证
**必须提供 API 密钥**。请通过 https://omim.org/api 申请（学术/非商业用途免费）。
- 通过查询参数传递：`?apiKey=您的API密钥`
- 所有请求均需密钥；未认证请求将被拒绝

## 速率限制
未公开详细说明。请根据服务条款保持合理使用频率。

## 响应格式
支持 JSON（使用 `&format=json`）或 XML（默认格式）。始终附加 `&format=json` 获取 JSON 响应。

## 核心端点

### 1. 条目查询（通过 MIM 编号）
```
GET https://api.omim.org/api/entry?mimNumber={mim_number}&apiKey={key}&format=json
```
示例：
```
GET https://api.omim.org/api/entry?mimNumber=141900&apiKey=YOUR_KEY&format=json
```
返回条目信息：标题、文本、基因图谱、等位基因变异、参考文献。

### 2. 包含特定字段的条目
```
GET https://api.omim.org/api/entry?mimNumber=141900&include=text&include=allelicVariantList&include=geneMap&apiKey={key}&format=json
```
可选字段：`text`, `clinicalSynopsis`, `geneMap`, `allelicVariantList`, `referenceList`, `existFlags`, `externalLinks`。

### 3. 条目搜索
```
GET https://api.omim.org/api/entry/search?search={query}&apiKey={key}&format=json
```
示例——搜索 "马凡综合征"：
```
GET https://api.omim.org/api/entry/search?search=marfan+syndrome&apiKey=YOUR_KEY&format=json&start=0&limit=10
```

### 4. 带筛选条件的搜索
```
GET https://api.omim.org/api/entry/search?search={query}&filter=gene&apiKey={key}&format=json
```
筛选选项：`gene`, `phenotype`, `clinical_synopsis` 等。

### 5. 基因图谱查询
```
GET https://api.omim.org/api/geneMap?chromosome={chrom}&apiKey={key}&format=json
```
示例：
```
GET https://api.omim.org/api/geneMap?chromosome=17&apiKey=YOUR_KEY&format=json&start=0&limit=10
```

### 6. 基因图谱搜索
```
GET https://api.omim.org/api/geneMap/search?search={query}&apiKey={key}&format=json
```

### 7. 临床概要搜索
```
GET https://api.omim.org/api/clinicalSynopsis/search?search={query}&apiKey={key}&format=json
```

## 响应结构
```json
{
  "omim": {
    "version": "1.0",
    "entryList": [
      {
        "entry": {
          "mimNumber": 141900,
          "status": "live",
          "titles": {
            "preferredTitle": "HEMOGLOBIN S; HBS",
            "alternativeTitles": "SICKLE CELL ANEMIA"
          },
          "textSectionList": [...],
          "geneMap": {
            "chromosome": "11",
            "cytoLocation": "11p15.4",
            "geneSymbols": "HBB"
          }
        }
      }
    ]
  }
}
```

## 分页机制
使用 `start` 和 `limit` 查询参数：
```
&start=0&limit=20
```

## MIM 编号类型标识
- **星号 (*)**：基因条目
- **加号 (+)**：已知表型的基因
- **井号 (#)**：分子机制明确的表型
- **百分号 (%)**：分子机制不明的表型
- **无符号**：其他条目类型

## 注意事项
- OMIM 数据受版权保护；学术用途 API 免费但需注册
- 不支持批量下载；请通过 OMIM 下载页面签署独立协议
- 建议联合 ClinVar、NCBI Gene 和 HPO 数据库交叉引用 MIM 编号进行疾病综合分析
