# DailyMed（NIH/NLM 药品标签）

## 基础 URL
```
https://dailymed.nlm.nih.gov/dailymed/services/
```

## 认证
无需API密钥。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `v2/spls.json?drug_name={name}` | 按名称搜索药品标签 |
| `v2/spls/{setid}.json` | 通过SetID获取标签元数据 |
| `v2/spls/{setid}/ndcs.json` | 获取标签的NDC代码 |
| `v2/spls/{setid}/media.json` | 获取标签的图片/媒体 |
| `v2/drugnames.json?drug_name={prefix}` | 药品名称自动补全 |
| `v2/drugclasses.json?drug_class_name={name}` | 按药理学分类搜索 |
| `v2/rxcuis.json?drug_name={name}` | 获取药品的RxNorm CUI |
| `v2/ndc/{ndc_code}/spls.json` | 通过NDC代码查找标签 |

## `/v2/spls.json` 的附加筛选条件
- `drug_class` — 药理学分类
- `labeler` — 制造商名称
- `page` / `pagesize` — 分页（最大100）

## 调用示例

```
# 搜索二甲双胍的标签
https://dailymed.nlm.nih.gov/dailymed/services/v2/spls.json?drug_name=metformin

# 药品名称自动补全
https://dailymed.nlm.nih.gov/dailymed/services/v2/drugnames.json?drug_name=ator

# 按药理学分类搜索
https://dailymed.nlm.nih.gov/dailymed/services/v2/spls.json?drug_class=HMG-CoA+Reductase+Inhibitor

# 完整标签XML（包含章节的SPL内容）
https://dailymed.nlm.nih.gov/dailymed/services/v2/spls/{setid}/packaging.xml
```

## 响应格式
```json
{
  "metadata": {
    "total_elements": 12,
    "elements_per_page": 10,
    "current_page": 1,
    "total_pages": 2
  },
  "data": [
    {
      "published_date": "2024-01-15",
      "title": "METFORMIN HYDROCHLORIDE tablet",
      "setid": "b03f295f-..."
    }
  ]
}
```

## 速率限制
无公开限制。请合理使用。
