# 工作区 API

## 概述

protocols.io 中的工作区通过组织协议、管理成员和控制访问权限实现团队协作。工作区 API 允许您列出工作区、管理成员关系以及访问特定工作区的协议。

## 基础 URL

所有工作区端点均使用基础 URL：`https://protocols.io/api/v3`

## 工作区操作

### 列出用户工作区

获取认证用户有权访问的所有工作区。

**端点：** `GET /workspaces`

**查询参数：**
- `page_size`：每页结果数量（默认值：10，最大值：50）
- `page_id`：分页页码（从 0 开始）

**响应包含：**
- 工作区 ID 和名称
- 工作区类型（个人、小组、机构）
- 成员数量
- 访问级别（所有者、管理员、成员、查看者）
- 创建日期

**请求示例：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://protocols.io/api/v3/workspaces"
```

### 获取工作区详情

获取特定工作区的详细信息。

**端点：** `GET /workspaces/{workspace_id}`

**路径参数：**
- `workspace_id`：工作区的唯一标识符

**响应包含：**
- 完整的工作区元数据
- 带角色的成员列表
- 工作区设置和权限
- 协议数量与分类

## 工作区成员关系

### 列出工作区成员

获取工作区的所有成员。

**端点：** `GET /workspaces/{workspace_id}/members`

**查询参数：**
- `page_size`：每页结果数量
- `page_id`：分页页码

**响应包含：**
- 成员姓名和邮箱
- 角色（所有者、管理员、成员、查看者）
- 加入日期
- 活跃状态

### 请求加入工作区

申请加入工作区。

**端点：** `POST /workspaces/{workspace_id}/join-request`

**请求体：**
- `message`（可选）：向管理员说明申请原因的消息

**请求示例：**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "我正在与 Smith 博士合作 CRISPR 项目，希望访问共享协议。"
  }' \
  "https://protocols.io/api/v3/workspaces/12345/join-request"
```

### 加入公开工作区

无需审批直接加入公开工作区。

**端点：** `POST /workspaces/{workspace_id}/join`

**注意**：仅适用于允许公开加入的工作区

## 工作区协议

### 列出工作区协议

获取工作区中的所有协议。

**端点：** `GET /workspaces/{workspace_id}/protocols`

**查询参数：**
- `filter`：协议筛选条件
  - `all`：工作区中所有协议
  - `own`：仅您创建的协议
  - `shared`：与您共享的协议
- `key`：关键词搜索
- `order_field`：排序字段（`activity`, `created_on`, `modified_on`, `name`）
- `order_dir`：排序方向（`desc`, `asc`）
- `page_size`：每页结果数量
- `page_id`：分页页码
- `content_format`：内容格式（`json`, `html`, `markdown`）

**请求示例：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://protocols.io/api/v3/workspaces/12345/protocols?filter=all&order_field=modified_on&order_dir=desc"
```

### 在工作区创建协议

在指定工作区内创建新协议。

**端点：** `POST /workspaces/{workspace_id}/protocols`

**请求体**：与标准协议创建参数相同（参见 protocols_api.md）

**注意**：协议将在工作区内创建并继承工作区权限

## 工作区类型与权限

### 工作区类型

1. **个人工作区**
   - 用户的默认工作区
   - 默认私有
   - 可共享特定协议

2. **小组工作区**
   - 团队协作空间
   - 成员共享访问权限
   - 基于角色的权限控制

3. **机构工作区**
   - 组织级工作区
   - 通常包含品牌标识
   - 集中式协议管理

### 权限级别

1. **所有者**
   - 完全控制工作区
   - 管理成员和权限
   - 可删除工作区

2. **管理员**
   - 管理协议和成员
   - 配置工作区设置
   - 不可删除工作区

3. **成员**
   - 创建和编辑协议
   - 查看所有工作区协议
   - 评论和协作

4. **查看者**
   - 仅查看权限
   - 可评论协议
   - 不可创建或编辑

## 常见用例

### 1. 实验室协议库

在共享工作区组织实验室协议：

1. 创建或加入实验室工作区：`GET /workspaces`
2. 列出现有协议：`GET /workspaces/{id}/protocols`
3. 创建新协议：`POST /workspaces/{id}/protocols`
4. 邀请实验室成员：分享工作区邀请
5. 按分类或标签组织

### 2. 协作协议开发

与团队成员共同开发协议：

1. 确定目标工作区：`GET /workspaces`
2. 在工作区创建协议草案
3. 通过工作区自动共享给团队成员
4. 通过评论收集反馈
5. 迭代并发布最终版本

### 3. 跨机构协作

与外部协作者合作：

1. 创建或识别共享工作区
2. 申请访问权限：`POST /workspaces/{id}/join-request`
3. 获批后访问共享协议
4. 贡献新协议或更新
5. 在个人工作区保留机构协议副本

### 4. 协议迁移

在工作区间移动协议：

1. 列出源工作区协议：`GET /workspaces/{source_id}/protocols`
2. 获取每个协议的完整详情
3. 在目标工作区创建协议：`POST /workspaces/{target_id}/protocols`
4. 复制所有步骤和元数据
5. 更新引用和链接

### 5. 工作区审计

审查工作区活动与内容：

1. 列出所有工作区：`GET /workspaces`
2. 获取每个工作区的成员列表
3. 检索带活动日期的协议列表
4. 识别非活跃或过时协议
5. 生成活动报告

## 工作区管理最佳实践

1. **组织规范**
   - 使用统一命名规则
   - 按项目或分类标记协议
   - 维护工作区目录或索引

2. **访问控制**
   - 定期审查成员列表
   - 分配适当权限级别
   - 移除非活跃成员

3. **协议标准**
   - 建立工作区级协议模板
   - 定义必需元数据字段
   - 实施质量审查流程

4. **协作机制**
   - 向成员传达工作区指南
   - 鼓励协议文档化
   - 促进知识共享

5. **备份与归档**
   - 定期导出工作区协议
   - 维护协议版本历史
   - 归档已完成项目

## 机构与工作区

机构是包含多个工作区的高层级实体。

### 导出机构数据

**端点：** `GET /organizations/{org_id}/export`

**用例**：批量导出所有协议和工作区数据用于机构存档或备份

## 通知与活动

工作区活动可能触发通知：

- 新增协议至工作区
- 团队成员更新协议
- 工作区协议的新评论
- 成员加入或离开工作区
- 权限变更

在账户设置中配置通知偏好。

## 错误处理

常见错误响应：

- `400 Bad Request`：无效工作区 ID 或参数
- `401 Unauthorized`：访问令牌缺失或无效
- `403 Forbidden`：工作区权限不足
- `404 Not Found`：工作区不存在或无访问权限
- `429 Too Many Requests`：超出速率限制

## 集成注意事项

集成工作区功能时：

1. **缓存工作区列表**：避免重复调用
2. **遵守权限**：执行操作前检查用户角色
3. **处理加入请求**：实现工作区访问审批流程
4. **定期同步**：周期性更新本地工作区数据
5. **支持离线访问**：缓存协议并在重连时同步
