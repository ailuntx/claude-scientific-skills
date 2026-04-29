# 附加功能

## 概述

本文档涵盖 protocols.io API 的附加功能，包括用户档案、近期发布协议、实验记录和通知。

## 基础 URL

所有端点均使用基础 URL：`https://protocols.io/api/v3`

## 用户档案管理

### 获取用户档案

检索已认证用户的档案信息。

**端点：** `GET /profile`

**响应包含：**
- 用户 ID 和用户名
- 全名
- 电子邮箱地址
- 所属机构
- 个人简介
- 档案图片 URL
- 账户创建日期
- 协议数量及统计信息

**请求示例：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://protocols.io/api/v3/profile"
```

### 更新用户档案

更新档案信息。

**端点：** `PATCH /profile`

**请求体：**
- `first_name`: 名字
- `last_name`: 姓氏
- `email`: 电子邮箱地址
- `affiliation`: 机构或组织
- `bio`: 个人简介
- `location`: 地理位置
- `website`: 个人或实验室网站 URL
- `twitter`: Twitter 账号
- `orcid`: ORCID 标识符

**请求示例：**
```bash
curl -X PATCH \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "affiliation": "示例大学生物学系",
    "bio": "专注于CRISPR基因编辑和分子生物学的研究员",
    "orcid": "0000-0001-2345-6789"
  }' \
  "https://protocols.io/api/v3/profile"
```

### 上传档案图片

更新个人头像。

**端点：** `POST /profile/image`

**请求格式：** `multipart/form-data`

**表单参数：**
- `image` (必填): 图片文件 (JPEG, PNG)

**推荐规格：**
- 最小尺寸：200x200 像素
- 宽高比：正方形 (1:1)
- 格式：JPEG 或 PNG
- 最大文件大小：5 MB

## 近期发布协议

### 查询发布协议

发现近期发布的公开协议。

**端点：** `GET /publications`

**查询参数：**
- `key`: 搜索关键词
- `category`: 按类别筛选
  - 示例类别：`molecular-biology`, `cell-biology`, `biochemistry` 等
- `date_from`: 起始日期 (ISO 8601 格式: YYYY-MM-DD)
- `date_to`: 结束日期
- `order_field`: 排序字段 (`published_on`, `title`, `views`)
- `order_dir`: 排序方向 (`desc`, `asc`)
- `page_size`: 每页结果数 (默认: 10, 最大: 50)
- `page_id`: 分页页码

**请求示例：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://protocols.io/api/v3/publications?category=molecular-biology&date_from=2025-01-01&order_field=published_on&order_dir=desc"
```

**应用场景：**
- 发现热门协议
- 监控领域内新发布内容
- 查找特定技术的最新协议
- 追踪值得引用的协议

## 实验记录

### 概述

实验记录允许用户记录协议的单次执行过程，追踪成功/失败情况及所有修改。

### 创建实验记录

记录协议执行过程。

**端点：** `POST /protocols/{protocol_id}/runs`

**路径参数：**
- `protocol_id`: 协议唯一标识符

**请求体：**
- `title` (必填): 实验运行标题
- `date`: 实验执行日期 (ISO 8601 格式)
- `status`: 实验结果状态
  - `success`: 实验成功
  - `partial`: 部分成功
  - `failed`: 实验失败
- `notes`: 实验运行详细说明
- `modifications`: 协议修改或调整
- `results`: 结果摘要
- `attachments`: 数据文件或图片的文件 ID

**请求示例：**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "CRISPR编辑 - HEK293细胞 - 第3次试验",
    "date": "2025-10-20",
    "status": "success",
    "notes": "成功实现87%编辑效率。根据先前试验将sgRNA浓度从100nM提升至150nM",
    "modifications": "步骤3孵育时间从30分钟延长至45分钟",
    "results": "流式细胞术确认72小时后87% GFP阳性细胞。Western blot显示阳性群体完全敲除"
  }' \
  "https://protocols.io/api/v3/protocols/12345/runs"
```

### 列出实验记录

检索协议的所有实验记录。

**端点：** `GET /protocols/{protocol_id}/runs`

**查询参数：**
- `status`: 按结果筛选 (`success`, `partial`, `failed`)
- `date_from`: 起始日期
- `date_to`: 结束日期
- `page_size`: 每页结果数
- `page_id`: 分页页码

### 更新实验记录

**端点：** `PATCH /protocols/{protocol_id}/runs/{run_id}`

**请求体**: 与创建参数相同，均为可选

### 删除实验记录

**端点：** `DELETE /protocols/{protocol_id}/runs/{run_id}`

**应用场景：**
- 追踪多次实验的可重复性
- 记录故障排除和优化过程
- 与协作者共享成功修改方案
- 构建机构知识库
- 满足实验记录本要求

## 通知管理

### 获取用户通知

检索已认证用户的通知。

**端点：** `GET /notifications`

**查询参数：**
- `type`: 按通知类型筛选
  - `comment`: 协议收到新评论
  - `mention`: 评论中被提及
  - `protocol_update`: 关注协议已更新
  - `workspace`: 工作区动态
  - `publication`: 协议已发布
- `read`: 按已读状态筛选
  - `true`: 仅已读通知
  - `false`: 仅未读通知
  - 留空则返回所有通知
- `page_size`: 每页结果数 (默认: 20, 最大: 100)
- `page_id`: 分页页码

**响应包含：**
- 通知 ID 和类型
- 消息/描述
- 相关协议/评论/工作区
- 时间戳
- 已读状态

**请求示例：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://protocols.io/api/v3/notifications?read=false&type=comment"
```

### 标记通知为已读

**端点：** `PATCH /notifications/{notification_id}`

**请求体：**
- `read`: 设置为 `true`

### 标记所有通知为已读

**端点：** `POST /notifications/mark-all-read`

### 删除通知

**端点：** `DELETE /notifications/{notification_id}`

## 机构管理

### 导出机构数据

导出机构所有协议和工作区数据。

**端点：** `GET /organizations/{organization_id}/export`

**路径参数：**
- `organization_id`: 机构唯一标识符

**查询参数：**
- `format`: 导出格式
  - `json`: 含完整元数据的 JSON 格式
  - `csv`: 支持电子表格导入的 CSV 格式
  - `xml`: XML 格式
- `include_files`: 包含关联文件 (`true`/`false`)
- `include_comments`: 包含讨论内容 (`true`/`false`)

**响应**: 导出包下载 URL

**应用场景：**
- 机构归档
- 合规与审计要求
- 迁移至其他系统
- 备份与灾难恢复
- 数据分析与报告

**请求示例：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://protocols.io/api/v3/organizations/12345/export?format=json&include_files=true&include_comments=true"
```

## 常见集成模式

### 1. 协议发现与导入

构建协议发现工作流：

```python
# 搜索相关协议
response = requests.get(
    'https://protocols.io/api/v3/publications',
    headers={'Authorization': f'Bearer {token}'},
    params={'key': 'CRISPR', 'category': 'molecular-biology'}
)

# 对每个感兴趣的协议
for protocol in response.json()['items']:
    # 获取完整详情
    details = requests.get(
        f'https://protocols.io/api/v3/protocols/{protocol["id"]}',
        headers={'Authorization': f'Bearer {token}'}
    )
    # 导入本地系统
    import_protocol(details.json())
```

### 2. 实验追踪

追踪所有协议执行：

1. 在实验室执行协议
2. 记录执行过程：`POST /protocols/{id}/runs`
3. 上传结果文件至工作区
4. 在实验记录中关联文件
5. 分析多次运行的成功率

### 3. 通知系统集成

构建自定义通知系统：

1. 轮询新通知：`GET /notifications?read=false`
2. 处理各类通知
3. 发送至内部通讯系统
4. 标记为已读：`PATCH /notifications/{id}`

### 4. 档案同步

跨系统保持档案同步：

1. 获取档案：`GET /profile`
2. 与内部系统对比
3. 更新差异内容
4. 同步头像和元数据

## API 响应格式

### 标准响应结构

多数 API 响应遵循此结构：

```json
{
  "status_code": 0,
  "status_message": "Success",
  "item": { /* 单条数据 */ },
  "items": [ /* 数据数组 */ ],
  "pagination": {
    "current_page": 0,
    "total_pages": 5,
    "page_size": 10,
    "total_items": 42
  }
}
```

### 错误响应结构

```json
{
  "status_code": 400,
  "status_message": "Bad Request",
  "error_message": "缺少必要参数: title",
  "error_details": {
    "field": "title",
    "issue": "required"
  }
}
```

## 最佳实践

1. **档案完整性**
   - 填写所有档案字段
   - 添加 ORCID 以归属研究成果
   - 及时更新所属机构

2. **实验记录**
   - 记录所有协议执行过程
   - 包含成功和失败案例
   - 注明所有修改
   - 附加相关数据文件

3. **通知管理**
   - 定期查看通知
   - 启用相关通知类型
   - 禁用不需要的通知类型
   - 及时回复评论

4. **协议发现**
   - 为研究领域设置定期搜索
   - 关注领域内高产出作者
   - 收藏实用协议
   - 在出版物中引用协议

5. **数据导出**
   - 定期导出机构数据
   - 测试恢复流程
   - 安全存储导出文件
   - 记录导出流程
