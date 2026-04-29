---
name: protocolsio-integration
description: 与protocols.io API的集成，用于管理科学实验方案。当需要利用protocols.io进行方案搜索、创建、更新或发布；管理方案步骤和材料；处理讨论与评论；组织工作区；上传管理文件；或将protocols.io功能集成到工作流时，应使用此技能。适用于方案发现、协作方案开发、实验跟踪、实验室方案管理和科学文档记录。
license: Unknown
metadata:
    skill-author: K-Dense Inc.
---

# Protocols.io 集成

## 概述

Protocols.io 是一个用于开发、共享和管理科学实验方案的综合平台。本技能提供与 protocols.io API v3 的完整集成，支持通过编程方式访问方案、工作区、讨论、文件管理和协作功能。

## 适用场景

在以下任何 protocols.io 相关场景中使用本技能：

- **方案发现**：通过关键词、DOI 或类别搜索现有方案
- **方案管理**：创建、更新或发布科学实验方案
- **步骤管理**：添加、编辑或组织方案步骤与流程
- **协作开发**：与团队成员共同开发共享方案
- **工作区组织**：管理实验室或机构的方案存储库
- **讨论与反馈**：添加或回复方案评论
- **文件管理**：为方案上传数据文件、图像或文档
- **实验跟踪**：记录方案执行过程与结果
- **数据导出**：备份或迁移方案集合
- **集成项目**：构建与 protocols.io 交互的工具

## 核心能力

本技能涵盖五大核心能力领域：

### 1. 认证与访问

使用访问令牌和 OAuth 流程管理 API 认证。支持客户端访问令牌（个人内容）和 OAuth 令牌（多用户应用）。

**关键操作：**
- 生成 OAuth 流程授权链接
- 将授权码兑换为访问令牌
- 刷新过期令牌
- 管理速率限制与权限

**参考：** 阅读 `references/authentication.md` 获取详细认证流程、OAuth 实现和安全最佳实践。

### 2. 方案操作

完整的方案生命周期管理，从创建到发布。

**关键操作：**
- 通过关键词、筛选器或 DOI 搜索发现方案
- 获取包含所有步骤的详细方案信息
- 创建带元数据和标签的新方案
- 更新方案信息与设置
- 管理方案步骤（创建、更新、删除、重排序）
- 处理方案材料与试剂
- 发布方案并获取 DOI
- 收藏方案以便快速访问
- 生成方案 PDF

**参考：** 阅读 `references/protocols_api.md` 获取全面的方案管理指南，包括 API 端点、参数、常见工作流和示例。

### 3. 讨论与协作

通过评论和讨论实现社区互动。

**关键操作：**
- 查看方案级和步骤级评论
- 创建新评论及线程式回复
- 编辑或删除个人评论
- 分析讨论模式与反馈
- 响应用户问题与建议

**参考：** 阅读 `references/discussions.md` 获取讨论管理、评论线程和协作工作流指南。

### 4. 工作区管理

在具备角色权限的团队工作区中组织方案。

**关键操作：**
- 列出并访问用户工作区
- 获取工作区详情与成员列表
- 申请加入工作区
- 列出工作区专属方案
- 在工作区内创建方案
- 管理工作区权限与协作

**参考：** 阅读 `references/workspaces.md` 获取工作区组织、权限管理和团队协作模式指南。

### 5. 文件操作

上传、组织和管理与方案关联的文件。

**关键操作：**
- 搜索工作区文件与文件夹
- 上传带元数据和标签的文件
- 下载文件并验证上传
- 按文件夹层级组织文件
- 更新文件元数据
- 删除与恢复文件
- 管理存储与组织结构

**参考：** 阅读 `references/file_manager.md` 获取文件上传流程、组织策略和存储管理指南。

### 6. 附加功能

包括个人资料、通知和导出的补充功能。

**关键操作：**
- 管理用户资料与设置
- 查询最新发布的方案
- 创建并跟踪实验记录
- 接收与管理通知
- 导出机构数据用于归档

**参考：** 阅读 `references/additional_features.md` 获取资料管理、发布方案发现、实验跟踪和数据导出指南。

## 快速入门

### 步骤 1：认证设置

使用 protocols.io API 前需完成：

1. 获取访问令牌（CLIENT_ACCESS_TOKEN 或 OAUTH_ACCESS_TOKEN）
2. 阅读 `references/authentication.md` 了解详细认证流程
3. 安全存储令牌
4. 在所有请求中包含：`Authorization: Bearer YOUR_TOKEN`

### 步骤 2：确定使用场景

根据需求选择对应能力领域：

- **处理方案？** → 阅读 `references/protocols_api.md`
- **管理团队方案？** → 阅读 `references/workspaces.md`
- **处理评论/反馈？** → 阅读 `references/discussions.md`
- **上传文件/数据？** → 阅读 `references/file_manager.md`
- **跟踪实验或资料？** → 阅读 `references/additional_features.md`

### 步骤 3：实施集成

遵循相关参考文件指南：

- 每个参考文件包含详细端点文档
- 明确 API 参数及请求/响应格式
- 提供常见用例和工作流示例
- 包含最佳实践与错误处理指南

## 基础 URL 与请求格式

所有 API 请求使用基础 URL：
```
https://protocols.io/api/v3
```

所有请求需包含 Authorization 请求头：
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

多数端点支持 JSON 请求/响应格式，需设置 `Content-Type: application/json`。

## 内容格式选项

多个端点支持 `content_format` 参数控制方案内容返回格式：

- `json`：Draft.js JSON 格式（默认）
- `html`：HTML 格式
- `markdown`：Markdown 格式

通过查询参数指定：`?content_format=html`

## 速率限制

注意 API 速率限制：

- **标准端点**：每用户每分钟 100 次请求
- **PDF 端点**：登录用户 5 次/分钟，未登录用户 3 次/分钟

遇到速率限制错误（HTTP 429）时实施指数退避策略。

## 常见工作流

### 工作流 1：导入与分析方案

分析 protocols.io 现有方案：

1. **搜索**：使用 `GET /protocols` 关键词查找相关方案
2. **获取**：通过 `GET /protocols/{protocol_id}` 获取完整详情
3. **提取**：解析步骤、材料和元数据用于分析
4. **审查讨论**：通过 `GET /protocols/{id}/comments` 查看用户反馈
5. **导出**：如需离线参考可生成 PDF

**参考文件**：`protocols_api.md`, `discussions.md`

### 工作流 2：创建与发布方案

创建新方案并发布 DOI：

1. **认证**：确保持有有效访问令牌（见 `authentication.md`）
2. **创建**：使用 `POST /protocols` 提交标题和描述
3. **添加步骤**：对每个步骤使用 `POST /protocols/{id}/steps`
4. **添加材料**：在步骤组件中记录试剂
5. **审查**：验证内容完整准确
6. **发布**：通过 `POST /protocols/{id}/publish` 获取 DOI

**参考文件**：`protocols_api.md`, `authentication.md`

### 工作流 3：协作实验室工作区

建立团队方案管理：

1. **创建/加入工作区**：访问或申请工作区成员资格（见 `workspaces.md`）
2. **组织结构**：创建实验室方案文件夹层级（见 `file_manager.md`）
3. **创建方案**：使用 `POST /workspaces/{id}/protocols` 创建团队方案
4. **上传文件**：添加实验数据与图像
5. **启用讨论**：团队成员可评论并提供反馈
6. **跟踪实验**：通过实验记录文档化方案执行

**参考文件**：`workspaces.md`, `file_manager.md`, `protocols_api.md`, `discussions.md`, `additional_features.md`

### 工作流 4：实验文档记录

跟踪方案执行与结果：

1. **执行方案**：在实验室完成方案操作
2. **上传数据**：使用文件管理 API 上传结果（见 `file_manager.md`）
3. **创建记录**：通过 `POST /protocols/{id}/runs` 文档化执行过程
4. **关联文件**：在实验记录中引用上传的数据文件
5. **记录修改**：文档化方案偏差或优化
6. **分析**：评估多次执行的复现性

**参考文件**：`additional_features.md`, `file_manager.md`, `protocols_api.md`

### 工作流 5：方案发现与引用

在研究中发现并引用方案：

1. **搜索**：通过 `GET /publications` 查询已发布方案
2. **筛选**：使用类别和关键词筛选相关方案
3. **审查**：阅读方案详情及社区评论
4. **收藏**：通过 `POST /protocols/{id}/bookmarks` 保存有用方案
5. **引用**：在出版物中使用方案 DOI（规范署名）
6. **导出 PDF**：生成格式化 PDF 用于离线参考

**参考文件**：`protocols_api.md`, `additional_features.md`

## Python 请求示例

### 基础方案搜索

```python
import requests

token = "YOUR_ACCESS_TOKEN"
headers = {"Authorization": f"Bearer {token}"}

# 搜索 CRISPR 方案
response = requests.get(
    "https://protocols.io/api/v3/protocols",
    headers=headers,
    params={
        "filter": "public",
        "key": "CRISPR",
        "page_size": 10,
        "content_format": "html"
    }
)

protocols = response.json()
for protocol in protocols["items"]:
    print(f"{protocol['title']} - {protocol['doi']}")
```

### 创建新方案

```python
import requests

token = "YOUR_ACCESS_TOKEN"
headers = {
    "Authorization": f"Bearer {token}",
    "Content-Type": "application/json"
}

# 创建方案
data = {
    "title": "CRISPR-Cas9 基因编辑方案",
    "description": "CRISPR 基因编辑综合方案",
    "tags": ["CRISPR", "gene editing", "molecular biology"]
}

response = requests.post(
    "https://protocols.io/api/v3/protocols",
    headers=headers,
    json=data
)

protocol_id = response.json()["item"]["id"]
print(f"已创建方案：{protocol_id}")
```

### 上传文件至工作区

```python
import requests

token = "YOUR_ACCESS_TOKEN"
headers = {"Authorization": f"Bearer {token}"}

# 上传文件
with open("data.csv", "rb") as f:
    files = {"file": f}
    data = {
        "folder_id": "root",
        "description": "实验结果",
        "tags": "experiment,data,2025"
    }

    response = requests.post(
        "https://protocols.io/api/v3/workspaces/12345/files/upload",
        headers=headers,
        files=files,
        data=data
    )

file_id = response.json()["item"]["id"]
print(f"已上传文件：{file_id}")
```

## 错误处理

为 API 请求实施健壮的错误处理：

```python
import requests
import time

def make_request_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.get(url, headers=headers)

            if response.status_code == 200:
                return response.json()
            elif response.status_code == 429:  # 速率限制
                retry_after = int(response.headers.get('Retry-After', 60))
                time.sleep(retry_after)
                continue
            elif response.status_code >= 500:  # 服务器错误
                time.sleep(2 ** attempt)  # 指数退避
                continue
            else:
                response.raise_for_status()

        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)

    raise Exception("超过最大重试次数")
```

## 参考文件

根据任务加载对应参考文件：

- **`authentication.md`**：OAuth 流程、令牌管理、速率限制
- **`protocols_api.md`**：方案增删改查、步骤、材料、发布、PDF
- **`discussions.md`**：评论、回复、协作
- **`workspaces.md`**：团队工作区、权限、组织
- **`file_manager.md`**：文件上传、文件夹、存储管理
- **`additional_features.md`**：资料、发布方案、实验、通知

在需要特定功能时，从 `references/` 目录读取对应参考文件。

## 最佳实践

1. **认证**：安全存储令牌，切勿置于代码或版本控制中
2. **速率限制**：实施指数退避并遵守速率限制
3. **错误处理**：妥善处理所有 HTTP 错误码
4. **数据验证**：API 调用前验证输入
5. **文档化**：完整记录方案步骤
6. **协作**：使用评论和讨论进行团队沟通
7. **组织**：保持一致的命名和标签规范
8. **版本控制**：更新方案时跟踪版本
9. **署名**：使用 DOI 规范引用方案
10. **备份**：定期导出重要方案和工作区数据

## 附加资源

- **官方 API 文档**：https://apidoc.protocols.io/
- **Protocols.io 平台**：https://www.protocols.io/
- **支持**：联系 protocols.io 支持获取 API 访问和技术问题协助
- **社区**：参与 protocols.io 社区交流最佳实践

## 故障排除

**认证问题：**
- 验证令牌有效性及是否过期
- 检查 Authorization 请求头格式：`Bearer YOUR_TOKEN`
- 确认令牌类型正确（CLIENT 与 OAUTH）

**速率限制：**
- 对 429 错误实施指数退避
- 监控请求频率
- 考虑缓存高频请求

**权限错误：**
- 验证工作区/方案访问权限
- 检查用户在工作区中的角色
- 确保非私有方案在无权限时也可访问

**文件上传失败：**
- 检查文件大小是否超出工作区限制
- 验证文件类型受支持
- 确保 multipart/form-data 编码正确

详细故障排除指南请参考各能力领域对应的参考文件。
