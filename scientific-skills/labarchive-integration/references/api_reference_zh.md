# LabArchives API 参考指南

## API 结构

所有 LabArchives API 调用均遵循以下 URL 模式：

```
https://<base_url>/api/<api_class>/<api_method>?<authentication_parameters>&<method_parameters>
```

## 区域 API 端点

| 区域 | 基础 URL |
|--------|----------|
| 美国/国际 | `https://api.labarchives.com/api` |
| 澳大利亚 | `https://auapi.labarchives.com/api` |
| 英国 | `https://ukapi.labarchives.com/api` |

## 身份验证

所有 API 调用均需以下身份验证参数：

- `access_key_id`：由 LabArchives 管理员提供
- `access_password`：由 LabArchives 管理员提供
- 特定操作可能需要额外的用户凭证

## API 类与方法

### 用户 API 类

#### `users/user_access_info`

获取用户 ID 和笔记本访问信息。

**参数：**
- `login_or_email`（必填）：用户邮箱或登录用户名
- `password`（必填）：用户外部应用密码（非常规登录密码）

**返回：** XML 或 JSON 响应，包含：
- 用户 ID (uid)
- 可访问笔记本列表及 ID (nbid)
- 账户状态与权限

**示例：**
```python
params = {
    'login_or_email': 'researcher@university.edu',
    'password': 'external_app_password'
}
response = client.make_call('users', 'user_access_info', params=params)
```

#### `users/user_info_via_id`

通过用户 ID 获取详细用户信息。

**参数：**
- `uid`（必填）：从 user_access_info 获取的用户 ID

**返回：** 用户档案信息，包括：
- 姓名与邮箱
- 账户创建日期
- 所属机构
- 角色与权限
- 存储配额及使用情况

**示例：**
```python
params = {'uid': '12345'}
response = client.make_call('users', 'user_info_via_id', params=params)
```

### 笔记本 API 类

#### `notebooks/notebook_backup`

下载完整笔记本数据（含条目、附件和元数据）。

**参数：**
- `uid`（必填）：用户 ID
- `nbid`（必填）：笔记本 ID
- `json`（可选，默认：false）：返回 JSON 格式数据（替代 XML）
- `no_attachments`（可选，默认：false）：备份中排除附件

**返回：**
- 当 `no_attachments=false`：包含所有笔记本数据的 7z 压缩包
- 当 `no_attachments=true`：包含条目内容的 XML 或 JSON 结构化数据

**文件格式：**
返回的压缩包包含：
- HTML 格式的条目文本内容
- 原始格式的文件附件
- 含时间戳、作者和版本历史的元数据 XML 文件
- 评论线程与批注

**示例：**
```python
# 含附件的完整备份
params = {
    'uid': '12345',
    'nbid': '67890',
    'json': 'false',
    'no_attachments': 'false'
}
response = client.make_call('notebooks', 'notebook_backup', params=params)

# 写入文件
with open('notebook_backup.7z', 'wb') as f:
    f.write(response.content)
```

```python
# 仅元数据备份（JSON格式，不含附件）
params = {
    'uid': '12345',
    'nbid': '67890',
    'json': 'true',
    'no_attachments': 'true'
}
response = client.make_call('notebooks', 'notebook_backup', params=params)
import json
notebook_data = json.loads(response.content)
```

#### `notebooks/list_notebooks`

获取用户可访问的所有笔记本（方法名可能因 API 版本而异）。

**参数：**
- `uid`（必填）：用户 ID

**返回：** 笔记本列表，包含：
- 笔记本 ID (nbid)
- 笔记本名称
- 创建与修改日期
- 访问级别（所有者、编辑者、查看者）
- 成员数量

### 条目 API 类

#### `entries/create_entry`

在笔记本中创建新条目。

**参数：**
- `uid`（必填）：用户 ID
- `nbid`（必填）：笔记本 ID
- `title`（必填）：条目标题
- `content`（可选）：HTML 格式的条目内容
- `date`（可选）：条目日期（默认为当前日期）

**返回：** 条目 ID 及创建确认信息

**示例：**
```python
params = {
    'uid': '12345',
    'nbid': '67890',
    'title': '2025-10-20 实验记录',
    'content': '<p>对目标基因进行 PCR 扩增...</p>',
    'date': '2025-10-20'
}
response = client.make_call('entries', 'create_entry', params=params)
```

#### `entries/create_comment`

向现有条目添加评论。

**参数：**
- `uid`（必填）：用户 ID
- `nbid`（必填）：笔记本 ID
- `entry_id`（必填）：目标条目 ID
- `comment`（必填）：评论文本（支持 HTML）

**返回：** 评论 ID 及时间戳

#### `entries/create_part`

向条目添加组件（如文本段落、表格、图片）。

**参数：**
- `uid`（必填）：用户 ID
- `nbid`（必填）：笔记本 ID
- `entry_id`（必填）：目标条目 ID
- `part_type`（必填）：组件类型（文本、表格、图片等）
- `content`（必填）：对应格式的组件内容

**返回：** 组件 ID 及创建确认

#### `entries/upload_attachment`

向条目上传文件附件。

**参数：**
- `uid`（必填）：用户 ID
- `nbid`（必填）：笔记本 ID
- `entry_id`（必填）：目标条目 ID
- `file`（必填）：文件数据（multipart/form-data）
- `filename`（必填）：原始文件名

**返回：** 附件 ID 及上传确认

**使用 requests 库的示例：**
```python
import requests

url = f'{api_url}/entries/upload_attachment'
files = {'file': open('/path/to/data.csv', 'rb')}
params = {
    'uid': '12345',
    'nbid': '67890',
    'entry_id': '11111',
    'filename': 'data.csv',
    'access_key_id': access_key_id,
    'access_password': access_password
}
response = requests.post(url, files=files, data=params)
```

### 站点报告 API 类

企业版专属功能，用于机构报告与分析。

#### `site_reports/detailed_usage_report`

生成机构综合使用统计报告。

**参数：**
- `start_date`（必填）：报告起始日期（YYYY-MM-DD）
- `end_date`（必填）：报告结束日期（YYYY-MM-DD）
- `format`（可选）：输出格式（csv、json、xml）

**返回：** 使用指标，包括：
- 用户登录频率
- 条目创建数量
- 存储利用率
- 协作统计数据
- 基于时间的活动模式

#### `site_reports/detailed_notebook_report`

生成机构所有笔记本的详细报告。

**参数：**
- `include_settings`（可选，默认：false）：包含笔记本设置
- `include_members`（可选，默认：false）：包含成员列表

**返回：** 笔记本清单，包含：
- 笔记本名称与 ID
- 所有者信息
- 创建与最后修改日期
- 成员数量与访问级别
- 存储大小
- 设置信息（如请求）

#### `site_reports/pdf_offline_generation_report`

跟踪 PDF 导出记录用于合规与审计。

**参数：**
- `start_date`（必填）：报告起始日期
- `end_date`（必填）：报告结束日期

**返回：** 导出活动日志，包含：
- 生成 PDF 的用户
- 导出的笔记本与条目
- 导出时间戳
- IP 地址

### 工具 API 类

#### `utilities/institutional_login_urls`

获取机构登录 URL 用于单点登录集成。

**参数：** 无需参数（使用访问密钥认证）

**返回：** 机构登录端点列表

## 响应格式

### XML 响应示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<response>
    <uid>12345</uid>
    <email>researcher@university.edu</email>
    <notebooks>
        <notebook>
            <nbid>67890</nbid>
            <name>2025 实验室笔记本</name>
            <role>owner</role>
        </notebook>
    </notebooks>
</response>
```

### JSON 响应示例

```json
{
    "uid": "12345",
    "email": "researcher@university.edu",
    "notebooks": [
        {
            "nbid": "67890",
            "name": "2025 实验室笔记本",
            "role": "owner"
        }
    ]
}
```

## 错误代码

| 代码 | 消息 | 含义 | 解决方案 |
|------|---------|---------|----------|
| 401 | 未授权 | 凭证无效 | 验证 access_key_id 和 access_password |
| 403 | 禁止访问 | 权限不足 | 检查用户角色和笔记本访问权限 |
| 404 | 未找到 | 资源不存在 | 确认 uid、nbid 或 entry_id 正确 |
| 429 | 请求过多 | 超出速率限制 | 实施指数退避策略 |
| 500 | 服务器内部错误 | 服务端问题 | 重试请求或联系技术支持 |

## 速率限制

LabArchives 实施速率限制以确保服务稳定性：

- **推荐值：** 每个 API 密钥每分钟最多 60 次请求
- **突发容限：** 短时突发最多允许 100 次请求
- **最佳实践：** 批量操作时在请求间设置 1-2 秒延迟

## API 版本管理

LabArchives API 保持向后兼容性。新增方法不会破坏现有实现。请关注 LabArchives 公告获取新功能信息。

## 支持与文档

如需 API 访问申请、技术问题或功能请求：
- 邮箱：support@labarchives.com
- 请附上机构名称和具体使用场景以加速处理
