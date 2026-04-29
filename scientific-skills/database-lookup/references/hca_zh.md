# 人类细胞图谱（HCA）

## 基础 URL
```
https://service.azul.data.humancellatlas.org/
```

## 认证
无需认证。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/index/projects?size={n}&catalog=dcp2` | 列出/搜索项目 |
| `/index/samples?size={n}&catalog=dcp2` | 列出/搜索样本 |
| `/index/files?size={n}&catalog=dcp2` | 列出/搜索文件 |
| `/index/summary?catalog=dcp2` | 汇总统计 |

## 调用示例
```
# 列出项目
https://service.azul.data.humancellatlas.org/index/projects?size=5&catalog=dcp2

# 汇总统计
https://service.azul.data.humancellatlas.org/index/summary?catalog=dcp2
```

支持通过JSON参数过滤器官、物种、文库构建方式等。

## 响应格式
JSON格式。包含项目/样本/文件元数据的`hits`数组及分页信息。

## 速率限制
未公布具体限制。请合理使用。
