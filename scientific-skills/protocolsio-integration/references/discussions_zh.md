# 讨论功能 API

## 概述

讨论功能 API 支持在实验方案上进行协作式评论。用户可在方案层级和单个步骤层级添加评论，支持线程式回复、编辑和删除操作。

## 基础 URL

所有讨论接口使用基础 URL：`https://protocols.io/api/v3`

## 方案层级评论

### 获取方案评论列表

检索方案的所有评论。

**接口：** `GET /protocols/{protocol_id}/comments`

**路径参数：**
- `protocol_id`：方案唯一标识符

**查询参数：**
- `page_size`：每页结果数量（默认值：10，最大值：50）
- `page_id`：分页页码（从0开始）

**响应包含：**
- 评论ID及内容
- 作者信息（姓名、所属机构、头像）
- 时间戳（创建与修改时间）
- 回复数量及线程结构

### 创建方案评论

为方案添加新评论。

**接口：** `POST /protocols/{protocol_id}/comments`

**请求体：**
- `body`（必填）：评论文本（支持HTML或Markdown格式）
- `parent_comment_id`（可选）：用于线程回复的父评论ID

**请求示例：**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "body": "该方案在我们的CRISPR实验中效果极佳，编辑效率达到85%"
  }' \
  "https://protocols.io/api/v3/protocols/12345/comments"
```

### 创建线程回复

回复现有评论时需包含父评论ID：

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "body": "请问您使用了哪种细胞类型？",
    "parent_comment_id": 67890
  }' \
  "https://protocols.io/api/v3/protocols/12345/comments"
```

### 更新评论

编辑自己的评论。

**接口：** `PATCH /protocols/{protocol_id}/comments/{comment_id}`

**请求体：**
- `body`（必填）：更新后的评论文本

**权限说明**：仅评论作者可编辑自己的评论

### 删除评论

移除评论。

**接口：** `DELETE /protocols/{protocol_id}/comments/{comment_id}`

**权限说明**：仅评论作者可删除自己的评论

**注意**：删除父评论可能影响整个线程，具体取决于API实现方式

## 步骤层级评论

### 获取步骤评论列表

检索特定方案步骤的所有评论。

**接口：** `GET /protocols/{protocol_id}/steps/{step_id}/comments`

**路径参数：**
- `protocol_id`：方案唯一标识符
- `step_id`：步骤唯一标识符

**查询参数：**
- `page_size`：每页结果数量
- `page_id`：分页页码

### 创建步骤评论

为特定步骤添加评论。

**接口：** `POST /protocols/{protocol_id}/steps/{step_id}/comments`

**请求体：**
- `body`（必填）：评论文本
- `parent_comment_id`（可选）：用于回复的父评论ID

**请求示例：**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "body": "在此步骤中，我们发现将孵育时间延长至2小时可显著改善结果"
  }' \
  "https://protocols.io/api/v3/protocols/12345/steps/67890/comments"
```

### 更新步骤评论

**接口：** `PATCH /protocols/{protocol_id}/steps/{step_id}/comments/{comment_id}`

**请求体：**
- `body`（必填）：更新后的评论文本

### 删除步骤评论

**接口：** `DELETE /protocols/{protocol_id}/steps/{step_id}/comments/{comment_id}`

## 常见应用场景

### 1. 讨论线程分析

分析方案相关讨论：
1. 获取方案评论：`GET /protocols/{id}/comments`
2. 为每个步骤获取步骤专属评论
3. 使用`parent_comment_id`构建讨论线程树
4. 分析反馈模式和常见问题

### 2. 协作优化方案

收集方案反馈：
1. 发布方案
2. 监控新评论：`GET /protocols/{id}/comments`
3. 通过线程回复解答问题
4. 根据反馈更新方案
5. 发布新版本并鸣谢贡献者

### 3. 社区互动

与方案用户互动：
1. 设置方案新评论监控
2. 及时响应问题和疑问
3. 使用步骤评论提供详细说明
4. 为复杂主题创建线程讨论

### 4. 方案问题排查

记录故障排除经验：
1. 识别方案中的问题步骤
2. 添加步骤评论描述具体问题
3. 记录解决方案或替代方法
4. 与遇到同类问题的用户建立讨论线程

## 评论格式

评论支持富文本格式：
- **HTML**：使用标准HTML标签格式化
- **Markdown**：使用Markdown语法简化格式
- **链接**：包含相关资源或文献的URL
- **提及**：引用其他用户（格式可能变化）

**Markdown示例：**
```json
{
  "body": "## 重要提示\n\n以下改进可优化结果：\n\n- 将温度升至37°C\n- 延长孵育时间至2小时\n- 使用新制备的试剂\n\n详见我们的出版物：[doi:10.xxxx/xxxxx](https://doi.org/...)"
}
```

## 最佳实践

1. **具体明确**：评论步骤时引用具体参数或条件
2. **提供背景**：包含相关实验细节（细胞类型、试剂批次、设备）
3. **善用步骤评论**：针对性反馈优先使用步骤层级评论
4. **建设性互动**：及时回应问题和反馈
5. **更新方案**：将验证有效的反馈纳入方案更新
6. **线程化管理**：使用回复功能保持相关评论集中
7. **记录变体**：分享经实践验证的方案修改

## 权限与隐私

- **公开方案**：任何用户均可评论已发布的公开方案
- **私有方案**：仅具有访问权限的协作者可评论
- **评论所有权**：仅评论作者可编辑或删除自己的评论
- **内容审核**：方案作者可能拥有额外管理权限

## 错误处理

常见错误响应：
- `400 Bad Request`：评论格式无效或缺少必填字段
- `401 Unauthorized`：访问令牌缺失或无效
- `403 Forbidden`：权限不足（如尝试编辑他人评论）
- `404 Not Found`：方案、步骤或评论不存在
- `429 Too Many Requests`：超出速率限制

## 通知机制

评论可能触发通知：
- 方案作者会收到新评论通知
- 评论作者会收到回复通知
- 用户可在账户设置中管理通知偏好
