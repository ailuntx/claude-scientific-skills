# EMDB（电子显微镜数据库）

## 基础URL
```
https://www.ebi.ac.uk/emdb/api/
```

## 认证
无需认证。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/entry/{emdb_id}` | 完整条目元数据（例如EMD-1234） |
| `/entry/map/{emdb_id}` | 图谱/体积元数据 |
| `/entry/experiment/{emdb_id}` | 实验细节 |
| `/entry/fitted/{emdb_id}` | 拟合的PDB模型 |
| `/search/{query}?rows={n}` | 通过关键词搜索条目 |

## 调用示例
```
# 条目元数据
https://www.ebi.ac.uk/emdb/api/entry/EMD-1234

# 搜索核糖体条目
https://www.ebi.ac.uk/emdb/api/search/ribosome?rows=5

# 实验细节
https://www.ebi.ac.uk/emdb/api/entry/experiment/EMD-1234
```

## 响应格式
JSON格式。搜索结果包含分页信息和匹配条目数组。

## 速率限制
遵循EBI合理使用政策。可通过FTP批量获取图谱文件（MRC/CCP4格式）。
