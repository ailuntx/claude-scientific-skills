# OpenFDA API

## 基础 URL
```
https://api.fda.gov
```

## 认证
可选免费 API 密钥（无密钥限 40 次/分钟，有密钥限 240 次/分钟）。注册地址：https://open.fda.gov/apis/authentication/
传递方式：`?api_key=您的密钥`

## 核心端点

| 端点 | 描述 |
|----------|-------------|
| `/drug/event.json` | 药品不良事件 (FAERS) |
| `/drug/label.json` | 药品标签信息 (SPL) |
| `/drug/ndc.json` | 国家药品编码目录 |
| `/drug/drugsfda.json` | Drugs@FDA（审批信息） |
| `/drug/enforcement.json` | 药品召回信息 |
| `/device/event.json` | 医疗器械不良事件 |
| `/device/510k.json` | 510(k) 上市前通告 |
| `/food/event.json` | 食品安全不良事件 |
| `/food/enforcement.json` | 食品执法行动 |

## 查询参数

- `search` — 使用 OpenFDA 语法查询
- `count` — 统计字段唯一值数量
- `limit` — 单次请求结果数（上限 1000）
- `skip` — 分页偏移量（上限 25000）

### 搜索语法
- 字段搜索：`字段名:"值"`
- 与运算：`字段1:值1+AND+字段2:值2`
- 或运算：`字段1:值1+OR+字段2:值2`
- 日期范围：`字段名:[起始日期+TO+结束日期]`
- 通配符：`字段名:关键词*`
- OpenFDA 标准化字段需添加 `openfda.` 前缀

## 调用示例

```
# 阿司匹林不良事件查询
/drug/event.json?search=patient.drug.openfda.brand_name:"aspirin"&limit=5

# 药品主要不良反应统计
/drug/event.json?search=patient.drug.openfda.generic_name:"metformin"&count=patient.reaction.reactionmeddrapt.exact

# 按通用名查询药品标签
/drug/label.json?search=openfda.generic_name:"ibuprofen"&limit=3

# 日期范围内药品召回查询
/drug/enforcement.json?search=report_date:[20230101+TO+20231231]&limit=10

# 仅查询严重不良事件
/drug/event.json?search=patient.drug.openfda.brand_name:"warfarin"+AND+serious:1&limit=10
```

## 速率限制
| 层级 | 请求/分钟 | 请求/天 |
|------|-------------|-------------|
| 无 API 密钥 | 40 | 1,000 |
| 使用 API 密钥（免费） | 240 | 120,000 |
