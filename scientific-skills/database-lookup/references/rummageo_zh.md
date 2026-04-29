# RummaGEO（GEO基因集富集搜索）

## 基础URL
```
https://rummageo.com/
```

## 认证
无需认证。

## 关键端点

| 端点 | 方法 | 描述 |
|----------|--------|-------------|
| `/api/enrich` | POST | 提交基因集进行GEO特征富集分析 |
| `/api/table` | GET | 分页展示已索引的GEO特征表 |

## 调用示例
```bash
curl -X POST "https://rummageo.com/api/enrich" \
  -H "Content-Type: application/json" \
  -d '{"genes": ["BRCA1","TP53","EGFR","MYC","PTEN"]}'
```

## 响应格式
JSON格式。包含匹配的GEO特征排序列表，附带重叠统计、p值及原始研究链接。

## 注意
POST端点——请通过shell使用`curl`调用，勿用WebFetch。

## 速率限制
未公布限制。设计用于交互式/程序化调用。
