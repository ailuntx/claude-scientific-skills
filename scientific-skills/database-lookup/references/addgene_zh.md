# Addgene（质粒库）

## 基础 URL
```
https://www.addgene.org/api/
```

## 认证
需要 API 密钥。在 addgene.org 注册并申请 API 访问权限。
传递方式：`Authorization: Token <your_api_key>`

从 `.env` 文件加载为 `ADDGENE_API_KEY`。

## 关键端点

| 端点 | 描述 |
|----------|-------------|
| `/plasmids/{addgene_id}/` | 通过 ID 获取质粒详情 |
| `/plasmids/search/?q={query}` | 通过关键词搜索质粒 |
| `/depositors/{id}/` | 质粒提交者信息 |
| `/articles/{id}/` | 关联出版物信息 |

## 调用示例
```
# 获取质粒详情（例如 pSpCas9）
GET https://www.addgene.org/api/plasmids/12260/
Authorization: Token YOUR_KEY

# 搜索质粒
GET https://www.addgene.org/api/plasmids/search/?q=GFP
Authorization: Token YOUR_KEY
```

## 响应格式
JSON 格式，包含质粒名称、骨架载体、插入片段、抗性标记、提交者、序列及出版物信息。

## 速率限制
未公布具体限制。需保持合理使用频率。
