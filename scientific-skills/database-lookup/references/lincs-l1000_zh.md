# LINCS L1000 (Clue.io) API 参考文档

## 概述
可通过 **Connectivity Map (CMap) API** 在 clue.io 访问 LINCS L1000 数据集。

## 基础 URL
```
https://api.clue.io/api
```

## 认证
- **需要 API 密钥**（在 clue.io 免费注册）
- 通过请求头传递：`user_key: YOUR_API_KEY`

## 核心端点

| 端点 | 描述 |
|---|---|
| `GET /perts` | 查询扰动剂（化合物、基因敲除、过表达） |
| `GET /genes` | 查询基因（L1000 标志基因 + 推断基因） |
| `GET /cells` | 查询 L1000 使用的细胞系 |
| `GET /sigs` | 查询连通性特征 |
| `GET /profiles` | 获取表达谱（5级 z 分数） |
| `GET /pcls` | 扰动剂类别 |

## 查询参数
所有端点均支持 Loopback 风格的 JSON 格式 `filter` 参数：
- `where` — 筛选条件
- `fields` — 选择特定字段
- `limit` / `skip` — 分页控制

## 调用示例

```bash
# 按名称搜索化合物扰动剂
curl -H "user_key: YOUR_API_KEY" \
  "https://api.clue.io/api/perts?filter={\"where\":{\"pert_iname\":\"vorinostat\"}}"

# 获取标志基因
curl -H "user_key: YOUR_API_KEY" \
  "https://api.clue.io/api/genes?filter={\"where\":{\"is_lm\":true},\"limit\":10}"

# 查询细胞系
curl -H "user_key: YOUR_API_KEY" \
  "https://api.clue.io/api/cells?filter={\"where\":{\"cell_iname\":\"MCF7\"}}"

# 获取化合物的连通性特征
curl -H "user_key: YOUR_API_KEY" \
  "https://api.clue.io/api/sigs?filter={\"where\":{\"pert_iname\":\"vorinostat\"},\"limit\":5}"
```

## 响应格式
JSON 格式。示例（扰动剂）：
```json
[
  {
    "pert_id": "BRD-K81418486",
    "pert_iname": "vorinostat",
    "pert_type": "trt_cp",
    "moa": ["HDAC inhibitor"],
    "target": ["HDAC1","HDAC2","HDAC3","HDAC6"]
  }
]
```

## 速率限制
- 免费层级：适度的速率限制（具体数值未公开说明）
- 批量数据下载可通过 clue.io 数据门户单独获取
