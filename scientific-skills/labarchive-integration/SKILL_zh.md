```markdown
---
name: labarchive-integration
description: 电子实验记录本API集成。通过编程方式访问记录本、管理条目/附件、备份记录本，并与Protocols.io/Jupyter/REDCap集成，实现自动化ELN工作流。
license: Unknown
metadata:
    skill-author: K-Dense Inc.
---

# LabArchives集成

## 概述

LabArchives是用于研究文档和数据管理的电子实验记录本平台。通过REST API以编程方式访问记录本、管理条目和附件、生成报告，并与第三方工具集成。

## 使用场景

本技能适用于以下场景：
- 使用LabArchives REST API实现记录本自动化
- 通过编程方式备份记录本
- 创建或管理记录本条目及附件
- 生成站点报告与分析数据
- 将LabArchives与第三方工具集成（Protocols.io、Jupyter、REDCap）
- 自动化上传数据至电子实验记录本
- 通过编程方式管理用户访问权限

## 核心功能

### 1. 认证与配置

设置LabArchives API集成的访问凭证和区域端点。

**前提条件：**
- 已启用API访问的企业版LabArchives许可
- 来自LabArchives管理员的API访问密钥ID和密码
- 用户认证凭证（邮箱和外部应用专用密码）

**配置设置：**

使用`scripts/setup_config.py`脚本创建配置文件：

```bash
python3 scripts/setup_config.py
```

生成包含以下结构的`config.yaml`文件：

```yaml
api_url: https://api.labarchives.com/api  # 或区域端点
access_key_id: YOUR_ACCESS_KEY_ID
access_password: YOUR_ACCESS_PASSWORD
```

**区域API端点：**
- 美国/国际：`https://api.labarchives.com/api`
- 澳大利亚：`https://auapi.labarchives.com/api`
- 英国：`https://ukapi.labarchives.com/api`

详细认证指南及故障排查请参阅`references/authentication_guide.md`。

### 2. 用户信息获取

获取后续API操作所需的用户ID（UID）及访问信息。

**工作流程：**

1. 使用登录凭证调用`users/user_access_info` API方法
2. 解析XML/JSON响应提取用户ID（UID）
3. 通过`users/user_info_via_id`使用UID获取详细用户信息

**Python封装库示例：**

```python
from labarchivespy.client import Client

# 初始化客户端
client = Client(api_url, access_key_id, access_password)

# 获取用户访问信息
login_params = {'login_or_email': user_email, 'password': auth_token}
response = client.make_call('users', 'user_access_info', params=login_params)

# 从响应中提取UID
import xml.etree.ElementTree as ET
uid = ET.fromstring(response.content)[0].text

# 获取详细用户信息
params = {'uid': uid}
user_info = client.make_call('users', 'user_info_via_id', params=params)
```

### 3. 记录本操作

管理记录本访问、备份及元数据获取。

**关键操作：**
- **列出记录本：** 获取用户可访问的所有记录本
- **备份记录本：** 下载完整记录本数据（可选包含附件）
- **获取记录本ID：** 提取机构定义的记录本标识符（用于与资助/项目管理系统集成）
- **获取记录本成员：** 列出特定记录本的所有访问用户
- **获取记录本设置：** 检索记录本的配置与权限信息

**记录本备份示例：**

使用`scripts/notebook_operations.py`脚本：

```bash
# 带附件备份（默认生成7z压缩包）
python3 scripts/notebook_operations.py backup --uid USER_ID --nbid NOTEBOOK_ID

# 无附件JSON格式备份
python3 scripts/notebook_operations.py backup --uid USER_ID --nbid NOTEBOOK_ID --json --no-attachments
```

**API端点格式：**
```
https://<api_url>/notebooks/notebook_backup?uid=<UID>&nbid=<NOTEBOOK_ID>&json=true&no_attachments=false
```

完整API方法文档请参阅`references/api_reference.md`。

### 4. 条目与附件管理

创建、修改和管理记录本条目及文件附件。

**条目操作：**
- 在记录本中创建新条目
- 向现有条目添加评论
- 创建条目部件/组件
- 上传文件附件至条目

**附件工作流：**

使用`scripts/entry_operations.py`脚本：

```bash
# 上传附件至条目
python3 scripts/entry_operations.py upload --uid USER_ID --nbid NOTEBOOK_ID --entry-id ENTRY_ID --file /path/to/file.pdf

# 创建含文本内容的新条目
python3 scripts/entry_operations.py create --uid USER_ID --nbid NOTEBOOK_ID --title "实验结果" --content "今日实验的结果..."
```

**支持的文件类型：**
- 文档（PDF、DOCX、TXT）
- 图像（PNG、JPG、TIFF）
- 数据文件（CSV、XLSX、HDF5）
- 科学格式（CIF、MOL、PDB）
- 压缩包（ZIP、7Z）

### 5. 站点报告与分析

生成记录本使用情况、活动及合规性报告（企业版功能）。

**可用报告：**
- 详细使用报告：用户活动指标与参与度统计
- 详细记录本报告：记录本元数据、成员列表及设置
- PDF/离线记录本生成报告：合规性导出追踪
- 记录本成员报告：访问控制与协作分析
- 记录本设置报告：配置与权限审计

**报告生成示例：**

```python
# 生成详细使用报告
response = client.make_call('site_reports', 'detailed_usage_report',
                           params={'start_date': '2025-01-01', 'end_date': '2025-10-20'})
```

### 6. 第三方集成

LabArchives与众多科研软件平台集成。本技能提供编程利用这些集成的指导。

**支持的集成：**
- **Protocols.io：** 将实验方案直接导出至LabArchives记录本
- **GraphPad Prism：** 导出分析与图表（8.0+版本）
- **SnapGene：** 分子生物学工作流直接集成
- **Geneious：** 生物信息学分析导出
- **Jupyter：** 将Jupyter笔记本嵌入为条目
- **REDCap：** 临床数据采集集成
- **Qeios：** 科研出版平台
- **SciSpace：** 文献管理工具

**OAuth认证：**
LabArchives所有新集成均采用OAuth认证。旧版集成可能仍使用API密钥认证。

详细集成设置指南及用例请参阅`references/integrations.md`。

## 常用工作流

### 完整记录本备份流程

1. 认证并获取用户ID
2. 列出所有可访问记录本
3. 遍历记录本并逐个备份
4. 存储带时间戳元数据的备份

```bash
# 完整备份脚本
python3 scripts/notebook_operations.py backup-all --email user@example.edu --password AUTH_TOKEN
```

### 自动化数据上传流程

1. 通过LabArchives API认证
2. 确定目标记录本和条目
3. 上传实验数据文件
4. 向条目添加元数据注释
5. 生成活动报告

### 集成工作流示例（Jupyter → LabArchives）

1. 将Jupyter笔记本导出为HTML或PDF
2. 使用entry_operations.py上传至LabArchives
3. 添加含执行时间戳和环境信息的注释
4. 为条目添加标签便于检索

## Python包安装

安装`labarchives-py`封装库简化API访问：

```bash
git clone https://github.com/mcmero/labarchives-py
cd labarchives-py
uv pip install .
```

也可通过Python的`requests`库直接发起HTTP请求实现自定义方案。

## 最佳实践

1. **速率限制：** 在API调用间添加适当延迟避免限流
2. **错误处理：** 始终在try-except块中封装API调用并记录日志
3. **认证安全：** 将凭证存储在环境变量或安全配置文件中（切勿在代码中明文存储）
4. **备份验证：** 备份后检查文件完整性与完备性
5. **增量操作：** 大型记录本操作使用分页和批处理
6. **区域端点：** 使用正确的区域API端点优化性能

## 故障排查

**常见问题：**

- **401未认证：** 确认访问密钥ID和密码正确；检查账户是否启用API访问
- **404未找到：** 验证记录本ID（nbid）存在且用户具有访问权限
- **403禁止访问：** 检查用户是否具有操作权限
- **空响应：** 确保必填参数（uid、nbid）正确提供
- **附件上传失败：** 验证文件大小限制与格式兼容性

更多支持请联系LabArchives：support@labarchives.com。

## 资源

本技能包含支持LabArchives API集成的捆绑资源：

### scripts/

- `setup_config.py`：交互式API凭证配置文件生成器
- `notebook_operations.py`：记录本列表、备份及管理工具
- `entry_operations.py`：创建条目和上传附件工具

### references/

- `api_reference.md`：含参数示例的完整API端点文档
- `authentication_guide.md`：详细认证设置与配置指南
- `integrations.md`：第三方集成设置指南及用例
```
