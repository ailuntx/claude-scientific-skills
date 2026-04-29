# 文件管理器 API

## 概述

文件管理器 API 支持在 protocols.io 工作区内执行文件操作，包括上传文件、组织文件夹、搜索内容和管理文件生命周期。该功能适用于将数据文件、图像、文档和其他资源附加到实验方案中。

## 基础 URL

所有文件管理器端点均使用基础 URL：`https://protocols.io/api/v3`

## 搜索与浏览

### 搜索工作区文件

在工作区内搜索文件和文件夹。

**端点：** `GET /workspaces/{workspace_id}/files/search`

**路径参数：**
- `workspace_id`：工作区的唯一标识符

**查询参数：**
- `query`：搜索关键词（匹配文件名和元数据）
- `type`：按类型筛选
  - `file`：仅文件
  - `folder`：仅文件夹
  - `all`：文件和文件夹（默认值）
- `folder_id`：限定在特定文件夹内搜索
- `page_size`：每页结果数量（默认：20，最大值：100）
- `page_id`：分页页码（从0开始）

**响应包含：**
- 文件/文件夹ID和名称
- 文件大小和类型
- 创建与修改日期
- 工作区内文件路径
- 下载URL（仅文件）

**请求示例：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://protocols.io/api/v3/workspaces/12345/files/search?query=microscopy&type=file"
```

### 列出文件夹内容

浏览特定文件夹内的文件和子文件夹。

**端点：** `GET /workspaces/{workspace_id}/folders/{folder_id}`

**路径参数：**
- `workspace_id`：工作区的唯一标识符
- `folder_id`：文件夹的唯一标识符（使用`root`表示工作区根目录）

**查询参数：**
- `order_by`：排序字段（`name`, `size`, `created`, `modified`）
- `order_dir`：排序方向（`asc`, `desc`）
- `page_size`：每页结果数量
- `page_id`：分页页码

**请求示例：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://protocols.io/api/v3/workspaces/12345/folders/root?order_by=modified&order_dir=desc"
```

## 文件上传

### 上传文件

将文件上传至工作区文件夹。

**端点：** `POST /workspaces/{workspace_id}/files/upload`

**请求格式：** `multipart/form-data`

**表单参数：**
- `file`（必填）：待上传文件
- `folder_id`：目标文件夹ID（省略或使用`root`表示工作区根目录）
- `name`：自定义文件名（可选，省略则使用原始文件名）
- `description`：文件描述
- `tags`：逗号分隔的标签

**请求示例：**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file=@/path/to/local/data.xlsx" \
  -F "folder_id=67890" \
  -F "description=实验三的试验结果" \
  -F "tags=实验,数据,2025" \
  "https://protocols.io/api/v3/workspaces/12345/files/upload"
```

### 上传验证

上传后验证文件是否被正确处理。

**端点：** `GET /workspaces/{workspace_id}/files/{file_id}/status`

**响应包含：**
- 上传状态（`processing`, `complete`, `failed`）
- 文件元数据
- 处理错误信息

## 文件操作

### 下载文件

从工作区下载文件。

**端点：** `GET /workspaces/{workspace_id}/files/{file_id}/download`

**路径参数：**
- `workspace_id`：工作区的唯一标识符
- `file_id`：文件的唯一标识符

**响应：** 二进制文件数据及对应的Content-Type头

**请求示例：**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  -o "downloaded_file.xlsx" \
  "https://protocols.io/api/v3/workspaces/12345/files/67890/download"
```

### 获取文件元数据

在不下载文件的情况下检索文件信息。

**端点：** `GET /workspaces/{workspace_id}/files/{file_id}`

**响应包含：**
- 文件名、大小和类型
- 上传日期和作者
- 描述和标签
- 文件路径和位置
- 下载URL
- 共享权限

### 更新文件元数据

修改文件描述、标签或其他元数据。

**端点：** `PATCH /workspaces/{workspace_id}/files/{file_id}`

**请求体：**
- `name`：新文件名
- `description`：更新描述
- `tags`：更新标签（逗号分隔）
- `folder_id`：移动至其他文件夹

**请求示例：**
```bash
curl -X PATCH \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "实验三的试验结果 - 修订版",
    "tags": "实验,数据,2025,修订版"
  }' \
  "https://protocols.io/api/v3/workspaces/12345/files/67890"
```

### 删除文件

将文件移至回收站（软删除）。

**端点：** `DELETE /workspaces/{workspace_id}/files/{file_id}`

**注意：** 已删除文件可在限定时间内从回收站恢复

### 恢复文件

从回收站恢复已删除文件。

**端点：** `POST /workspaces/{workspace_id}/files/{file_id}/restore`

## 文件夹操作

### 创建文件夹

在工作区中新建文件夹。

**端点：** `POST /workspaces/{workspace_id}/folders`

**请求体：**
- `name`（必填）：文件夹名称
- `parent_folder_id`：父文件夹ID（省略表示工作区根目录）
- `description`：文件夹描述

**请求示例：**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "2025年实验",
    "parent_folder_id": "root",
    "description": "2025年所有实验数据"
  }' \
  "https://protocols.io/api/v3/workspaces/12345/folders"
```

### 重命名文件夹

**端点：** `PATCH /workspaces/{workspace_id}/folders/{folder_id}`

**请求体：**
- `name`：新文件夹名称
- `description`：更新描述

### 删除文件夹

删除文件夹及其可选内容。

**端点：** `DELETE /workspaces/{workspace_id}/folders/{folder_id}`

**查询参数：**
- `recursive`：设为`true`可删除文件夹及所有内容（默认：`false`）

**警告：** 递归删除操作不可逆

## 常见用例

### 1. 实验方案数据附件

将实验数据文件附加到实验方案：
1. 上传数据文件：`POST /workspaces/{id}/files/upload`
2. 验证上传完成状态
3. 在实验方案步骤中引用文件ID
4. 在实验方案描述中包含下载链接

**工作流示例：**
```bash
# 上传数据文件
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file=@results.csv" \
  -F "description=实验方案执行结果" \
  "https://protocols.io/api/v3/workspaces/12345/files/upload"

# 记录响应中的file_id，随后在实验方案中引用
```

### 2. 工作区组织

将文件按逻辑文件夹结构组织：
1. 创建文件夹层级：`POST /workspaces/{id}/folders`
2. 将文件上传至对应文件夹
3. 使用一致的命名规范
4. 为文件添加标签便于搜索

**结构示例：**
```
工作区根目录
├── 实验方案
│   ├── 已发布
│   └── 草稿
├── 数据
│   ├── 原始数据
│   └── 处理数据
├── 图像
│   ├── 显微图像
│   └── 凝胶图像
└── 文档
    ├── 论文
    └── 演示文稿
```

### 3. 文件搜索与发现

跨工作区查找文件：
1. 关键词搜索：`GET /workspaces/{id}/files/search?query=关键词`
2. 按类型和日期筛选
3. 下载相关文件
4. 更新元数据优化组织

### 4. 批量文件上传

上传多个关联文件：
1. 创建目标文件夹
2. 对每个文件执行：
   - 上传文件
   - 验证上传状态
   - 添加统一标签
3. 创建索引或清单文件记录所有上传项

### 5. 数据备份与导出

导出工作区文件用于备份：
1. 列出所有文件夹：`GET /workspaces/{id}/folders/root`
2. 遍历每个文件夹并列出文件
3. 下载所有文件：`GET /workspaces/{id}/files/{file_id}/download`
4. 本地保留文件夹结构
5. 单独存储元数据以便恢复

### 6. 文件版本管理

手动管理文件版本：
1. 上传带版本号的新文件（如`data_v2.csv`）
2. 更新旧版本元数据标记为已替代
3. 在文件夹结构中维护版本历史
4. 在实验方案中引用特定版本

## 支持的文件类型

Protocols.io 支持多种文件类型：

**数据文件：**
- 电子表格：`.xlsx`, `.xls`, `.csv`, `.tsv`
- 统计数据：`.rds`, `.rdata`, `.sav`, `.dta`
- 纯文本：`.txt`, `.log`, `.json`, `.xml`

**图像：**
- 通用格式：`.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.tif`, `.tiff`
- 科研专用：`.czi`, `.nd2`, `.lsm`（可能需要特殊处理）

**文档：**
- PDF：`.pdf`
- Word：`.docx`, `.doc`
- PowerPoint：`.pptx`, `.ppt`

**代码与脚本：**
- Python：`.py`, `.ipynb`
- R：`.r`, `.rmd`
- Shell：`.sh`, `.bash`

**多媒体：**
- 视频：`.mp4`, `.avi`, `.mov`
- 音频：`.mp3`, `.wav`

**归档文件：**
- 压缩格式：`.zip`, `.tar.gz`, `.7z`

**文件大小限制：**
- 标准文件：查看工作区限制（通常100 MB - 1 GB）
- 大文件：可能需要分块上传或特殊处理

## 最佳实践

1. **文件命名**
   - 采用描述性、一致的命名规范
   - 包含ISO格式日期（YYYY-MM-DD）
   - 避免特殊字符和空格（使用下划线）
   - 示例：`experiment_results_2025-10-26.csv`

2. **组织管理**
   - 创建逻辑文件夹层级
   - 将相关文件集中存放
   - 原始数据与处理结果分离
   - 实验方案专属文件存入独立文件夹

3. **元数据**
   - 添加详细描述
   - 统一标签体系
   - 包含版本信息
   - 记录处理步骤

4. **存储管理**
   - 定期审查归档旧文件
   - 删除不必要的重复项
   - 压缩大型数据集
   - 监控工作区存储限制

5. **协作规范**
   - 采用团队成员易懂的文件名
   - 在描述中说明文件用途
   - 保持一致的文件夹结构
   - 重大结构调整时及时沟通

6. **安全性**
   - 未经授权勿上传敏感数据
   - 注意工作区可见性设置
   - 使用适当的访问控制
   - 定期审计文件访问记录

## 错误处理

常见错误响应：

- `400 Bad Request`：无效文件格式或参数
- `401 Unauthorized`：访问令牌缺失或无效
- `403 Forbidden`：工作区权限不足
- `404 Not Found`：文件或文件夹不存在
- `413 Payload Too Large`：文件超出大小限制
- `422 Unprocessable Entity`：文件验证失败
- `429 Too Many Requests`：超出速率限制
- `507 Insufficient Storage`：工作区存储空间不足

## 性能考量

1. **大文件处理**
   - 超过100 MB的文件建议分块上传
   - 对大型数据集使用压缩
   - 尽量在非高峰时段上传

2. **批量操作**
   - 对失败上传实施重试机制
   - 速率限制时采用指数退避策略
   - 尽可能并行处理上传任务

3. **下载优化**
   - 本地缓存频繁访问的文件
   - 大文件下载使用流式传输
   - 为中断下载实现续传功能
