# COSMIC（癌症体细胞突变目录）

## 基本URL
```
https://cancer.sanger.ac.uk/cosmic/api/v1/
```

## 认证
**需要注册。** 免费学术账户或付费商业许可证。

登录获取JWT令牌：
```
POST /auth/login
Content-Type: application/json
{"email": "you@example.com", "password": "yourpassword"}
```
令牌传递方式：`Authorization: Bearer <token>`

## 核心端点

### 按基因搜索突变
```
GET /mutations/search?q={gene_symbol}&page=1&page_size=5
```

### 获取基因信息
```
GET /genes/{gene_symbol}
```
示例：`/genes/BRAF`

响应包含：基因符号、基因名称、染色体、癌症普查状态（布尔值）、分级、突变计数、样本计数

### 通过COSMIC ID获取特定突变
```
GET /mutations/{cosmic_mutation_id}
```
示例：`/mutations/COSV56056643`

响应包含：基因、CDS突变、氨基酸突变、突变类型、FATHMM预测、基因组坐标、组织分布

### 癌症基因普查
```
GET /cancer-gene-census?tier=1&page_size=10
```

### 按组织/组织学分类的突变
```
GET /mutations/distribution/{gene_symbol}
```

## 速率限制
未正式公布。批量数据需通过SFTP下载（需许可）。

## 重要提示
- 所有API调用均需COSMIC认证
- 商业用途需要付费许可证
- 大型查询建议通过SFTP访问批量数据而非API
- API结构可能随COSMIC版本变更
