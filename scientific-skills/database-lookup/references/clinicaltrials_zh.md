# ClinicalTrials.gov (v2 API)

## 基础 URL
```
https://clinicaltrials.gov/api/v2/
```

## 认证
无需API密钥。完全公开。

## 关键端点

### 搜索研究
```
GET /studies
```

关键参数：
- `query.cond` — 疾病/病症（例如 `breast cancer`）
- `query.intr` — 干预措施/治疗方法（例如 `pembrolizumab`）
- `query.term` — 通用搜索词
- `query.spons` — 申办方
- `query.id` — NCT标识号
- `filter.overallStatus` — 竖线分隔状态：`RECRUITING|COMPLETED|ACTIVE_NOT_RECRUITING|...`
- `filter.phase` — 试验阶段：`PHASE1|PHASE2|PHASE3|PHASE4|NA`
- `filter.geo` — 地理范围：`distance(lat,lon,dist)` 例如 `distance(38.89,-77.03,50mi)`
- `fields` — 逗号分隔的字段列表（用于缩减响应数据）
- `sort` — 排序规则（例如 `LastUpdatePostDate:desc`）
- `pageSize` — 每页结果数（默认10，最大1000）
- `pageToken` — 下一页游标（来自响应中的 `nextPageToken`）
- `countTotal=true` — 包含总计数

示例 — 招募中的乳腺癌III期试验：
```
/studies?query.cond=breast+cancer&filter.overallStatus=RECRUITING&filter.phase=PHASE3&pageSize=5&countTotal=true
```

响应结构：
```json
{
  "totalCount": 1234,
  "studies": [
    {
      "protocolSection": {
        "identificationModule": {"nctId": "NCT05123456", "briefTitle": "..."},
        "statusModule": {"overallStatus": "RECRUITING"},
        "designModule": {"phases": ["PHASE3"], "enrollmentInfo": {"count": 500}},
        "conditionsModule": {"conditions": ["Breast Cancer"]},
        "eligibilityModule": {"minimumAge": "18 Years", "sex": "ALL"}
      }
    }
  ],
  "nextPageToken": "CAYQAg"
}
```

### 通过NCT标识号获取单个研究
```
GET /studies/{nctId}
```
示例：`/studies/NCT05123456`

### 研究计数
```
GET /stats/size?query.cond={condition}&filter.overallStatus=RECRUITING
```

### 字段元数据
```
GET /studies/metadata
```

## 分页
通过 `pageToken` 实现基于游标的分页（非数字偏移）。首次请求需包含 `countTotal=true` 获取总数。

## 速率限制
无需API密钥。请合理使用——每秒不超过数次请求。批量下载：https://clinicaltrials.gov/AllAPIJSON.zip
