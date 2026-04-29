# MouseMine（小鼠基因组信息学，基于InterMine）

## 基础URL
```
https://www.mousemine.org/mousemine/service
```

## 认证
多数查询无需认证。保存的列表需使用免费账户令牌。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/search?q={查询词}&format=json` | 跨所有对象的关键词搜索 |
| `/template/results?name={模板}&op1=LOOKUP&value1={值}&format=json` | 运行预构建模板查询 |
| `/query/results` (POST) | 运行自定义PathQuery（XML格式） |
| `/model` | 获取数据模型 |

## 调用示例
```
# Brca1关键词搜索
https://www.mousemine.org/mousemine/service/search?q=Brca1&format=json

# 模板：基因 → GO术语
https://www.mousemine.org/mousemine/service/template/results?name=Gene_GO&op1=LOOKUP&value1=Pax6&format=json
```

## 自定义查询（POST）
```
POST /query/results
Content-Type: application/x-www-form-urlencoded
query=<query model="genomic" view="Gene.symbol Gene.name" sortOrder="Gene.symbol asc"><constraint path="Gene.organism.name" op="=" value="Mus musculus"/></query>&format=json
```

## 响应格式
JSON格式：`{"results": [...], "statusCode": 200}`。通过`format`参数也支持XML、TSV、CSV格式。

## 速率限制
无公开限制。请合理使用。
